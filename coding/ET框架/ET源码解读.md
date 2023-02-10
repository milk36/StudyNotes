# 源码解读

## 启动流程

### 服务端

`Server\Server.App\Program.cs` 主函数入口

`Server\Server.Hotfix\AppStart_Init.cs` 启动事件监听 `EventType.AppStart`

## 默认组件

### TimerComponent 定时组件?

* 任务类型
  
  ```c#
  public enum TimerClass
  {
      None,
      OnceTimer, //一次
      OnceWaitTimer, //等待执行一次
      RepeatedTimer, //重复执行
  }  
  ```

* 添加定时任务

  ```c#
  private static void AddTimer(this TimerComponent self, long tillTime, TimerAction timer)
  {
      if (timer.TimerClass == TimerClass.RepeatedTimer && timer.Time == 0)
      {
          self.everyFrameTimer.Enqueue(timer.Id); //添加需要重复执行的任务 
          return;
      }
      // TimeId 类型为:MultiMap<T,K> : SortedDictionary<T,List<K>> ,有序字典,同一时间可以有多个任务
      self.TimeId.Add(tillTime, timer.Id); //添加到等待队列, 默认升序, OnceTimer|OnceWaitTimer, 剩余执行时间 与 任务id
      if (tillTime < self.minTime)
      {
          self.minTime = tillTime; //记录最小时间，不用每次都去MultiMap取第一个值, 让需要更早执行的任务插到前面
      }
  }
  ```

  展开:`self.TimeId.ForEachFunc(self.foreachFunc);`

  判断 `MultiMap<long,long> TimeId` 中已经超时需要执行的任务,并添加到超时任务队列中(定时器不能可能绝对准确)

  ```c#
  self.foreachFunc = (k, v) =>
  {
      if (k > self.timeNow) //还没到执行时间
      {
          self.minTime = k; //刷新 minTime
          return false; //跳出TimeId遍历
      }
      self.timeOutTime.Enqueue(k); //将超时timeId添加到队列 ,后续逻辑运行任务
      return true;
  };
  //ET.ForeachHelper.ForEachFunc<T,K> 静态扩展 MultiMap 类型添加 ForEachFunc方法
  public static void ForEachFunc<T, K>(this MultiMap<T, K> multiMap, Func<T, List<K>, bool> func)
  {
      foreach (var kv in multiMap)
      {
          if (!func(kv.Key, kv.Value))
          {
              break;
          }
      }
  }
  ```

  每帧 `Update` 逻辑
  
  ```c#
  public override void Update(TimerComponent self)
  {
    //RepeatedTimer 重复执行的任务
    #region 每帧执行的timer，不用foreach TimeId，减少GC
    int count = self.everyFrameTimer.Count;
    for (int i = 0; i < count; ++i)
    {
        long timerId = self.everyFrameTimer.Dequeue();
        TimerAction timerAction = self.GetChild<TimerAction>(timerId);
        if (timerAction == null)
        {
            continue;
        }
        self.Run(timerAction);
    }
    #endregion
    if (self.TimeId.Count == 0)
    {
        return; //如果没有其他任务, 这一帧结束
    }
    self.timeNow = TimeHelper.ServerNow(); //重置当前时间
    if (self.timeNow < self.minTime)//对应到 不用每次都去MultiMap取第一个值
    {
        return; //队列中最小时间 也没有达到执行条件, 这一帧结束
    }
    //到这里说明队列中因该有任务以超时,可以执行了
    self.TimeId.ForEachFunc(self.foreachFunc); //把TimeId再遍历一遍, 得到超时任务, 判断是否要刷新minTime
    while (self.timeOutTime.Count > 0)
    {
        long time = self.timeOutTime.Dequeue();
        var list = self.TimeId[time]; //同一时间可能有多个任务需要执行
        for (int i = 0; i < list.Count; ++i)
        {
            long timerId = list[i];
            self.timeOutTimerIds.Enqueue(timerId); //添加到可执行任务队列
        }
        self.TimeId.Remove(time); //从等待队列中移除
    }
    while (self.timeOutTimerIds.Count > 0)
    {
        long timerId = self.timeOutTimerIds.Dequeue(); //逐个弹出任务id
        TimerAction timerAction = self.GetChild<TimerAction>(timerId);
        if (timerAction == null)
        {
            continue;
        }
        self.Run(timerAction); //执行任务
    }
  }
  ```
