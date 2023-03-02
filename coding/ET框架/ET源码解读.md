<!-- markdownlint-disable MD033 -->
# 源码解读

## 启动流程

### 服务端

`Server/Server.App/Program.cs` 主函数入口

`Server/Server.Hotfix/AppStart_Init.cs` 启动事件监听 `EventType.AppStart`

### 读取启动配置

`Server/Server.Hotfix/AppStart_Init.cs`

```c#
private async ETTask RunAsync(EventType.AppStart args)
{
  Game.Scene.AddComponent<ConfigComponent>(); //=> ConfigAwakeSystem.Awake()
  await ConfigComponent.Instance.LoadAsync(); //执行配置文件异步加载
  ...
}
```

### 主线程while内容

* `Server/Server.App/Program.cs` 服务端启动类

  ```c#
  ...
  while (true)
    {
      try
      {
        Thread.Sleep(1);
        Game.Update(); //逻辑驱动
        Game.LateUpdate();
        Game.FrameFinish();
      }
      catch (Exception e)
      {
        Log.Error(e);
      }
    }
  ...
  ```

* `Server/Server.Model/Core/Entity/Game.cs` Game.Scene 根节点, 所有组件,实体都在此基础之上
  
  ```c#
  public static void Update()
  {
      ThreadSynchronizationContext.Update(); //网络线程, 在接收到
      TimeInfo.Update(); //更新系统时间
      EventSystem.Update(); //驱动事件系统
  }
  ```

## ETTask

* `ETTask.Coroutine()` 启动协程执行逻辑, 则不等待逻辑结束会接挂起await相关的操作

### 核心

* ETTask : Awaiter
* ETTaskCompleted : Awaiter
  
  为默认完成状态, 主要是用于配合需要返回 ` ETTask` 的函数使用, `await ETTask.CompletedTask;`

## 默认组件

### ThreadSynchronizationContext 网络异步线程队列

NetThreadComponent 网络线程组件

* `Server\Server.Model\Core\ThreadSynchronizationContext.cs`

    ```c#
    // 线程同步队列,发送接收socket回调都放到该队列,由poll线程统一执行
    private readonly ConcurrentQueue<Action> queue = new ConcurrentQueue<Action>();

    //如果调用线程为主线程就立即执行, 否则先将操作压入队列
    public void Post(Action action){}
    //将操作压入队列
    public void PostNext(Action action)
    ```

发送接收socket回调都放到该队列

### Entity 实体组件的挂载

* `Server/Server.Model/Core/Object/Entity`

```c#
public K AddComponent<K>(bool isFromPool = false) where K : Entity, IAwake, new()
{
    Type type = typeof (K);
    if (this.components != null && this.components.ContainsKey(type))
    {
        throw new Exception($"entity already has component: {type.FullName}");
    }
    Entity component = Create(type, isFromPool);
    component.Id = this.Id;
    component.ComponentParent = this; //为组件关联父实体
    EventSystem.Instance.Awake(component); //调用组件 Awake, 执行唤醒操作
    
    if (this is IAddComponent)
    {
        EventSystem.Instance.AddComponent(this, component);
    }
    return component as K;
}
```

`ET.Entity.ComponentParent`

```c#
private Entity ComponentParent
{
  set
  {
    ...
    this.parent = value;
    this.IsComponent = true;
    this.parent.AddToComponents(this); //将当前组件添加到父实体中
    this.Domain = this.parent.domain;
  }
}
```

> 将当前组件添加到父实体中, 即`ET.Entity.components`的字典容器中`private Dictionary<Type, Entity> components;`

### TService TCP网络服务

* `Server\Server.Model\Module\NetworkTCP\TService.cs`

  1. `TService` 创建Socket,等待新连接建立
  
      ```c#
      //用于服务端等待连接进入
      public TService(ThreadSynchronizationContext threadSynchronizationContext, IPEndPoint ipEndPoint, ServiceType serviceType)
      {
        //初始化Socket
        this.acceptor = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
        this.acceptor.SetSocketOption(SocketOptionLevel.Socket, SocketOptionName.ReuseAddress, true);
        this.innArgs.Completed += this.OnComplete;
        this.acceptor.Bind(ipEndPoint);
        this.acceptor.Listen(1000);
        this.ThreadSynchronizationContext.PostNext(this.AcceptAsync);  
      }      
      //用于新连接
      public TChannel(long id, Socket socket, TService service)
      {

        // 下一帧再开始读写
        this.Service.ThreadSynchronizationContext.PostNext(() =>
        {
          this.StartRecv(); 
          this.StartSend();
        });
      }

      //处理 SocketAsyncEventArgs.Completed 网络事件
      private void OnComplete(object sender, SocketAsyncEventArgs e)
      {
        switch (e.LastOperation)
        {
          case SocketAsyncOperation.Connect:OnConnectComplete(e)
          case SocketAsyncOperation.Receive:OnRecvComplete(e)
          case SocketAsyncOperation.Send:OnSendComplete(e)
          case SocketAsyncOperation.Disconnect:OnDisconnectComplete(e)
        }
      }
      ```

      1. `TService.AcceptAsync()` //等待新连接的建立

          ```c#
          this.innArgs.AcceptSocket = null;
          if (this.acceptor.AcceptAsync(this.innArgs))
          {
            return;
          }
          OnAcceptComplete(this.innArgs.SocketError, this.innArgs.AcceptSocket);
          ```

      1. `TService.OnAcceptComplete`

          ```c#
          //创建TChannel
          long id = this.CreateAcceptChannelId(0);
          TChannel channel = new TChannel(id, acceptSocket, this); //新连接构造函数
          this.idChannels.Add(channel.Id, channel);
          long channelId = channel.Id;      
          this.OnAccept(channelId, channel.RemoteAddress); //ET.NetKcpComponentSystem.OnAccept
          // 开始新的accept
          this.AcceptAsync(); //继续等待下一个连接
          ```

          `ET.AService.OnAccept -> ET.AService.OnAccept -> ET.NetKcpComponentSystem.OnAccept` 创建Session 并挂载Session实体

            ```c#
            public static void OnAccept(this NetKcpComponent self, long channelId, IPEndPoint ipEndPoint)
            {
                Session session = self.AddChildWithId<Session, AService>(channelId, self.Service);
                session.RemoteAddress = ipEndPoint;

                // 挂上这个组件，5秒就会删除session，所以客户端验证完成要删除这个组件。该组件的作用就是防止外挂一直连接不发消息也不进行权限验证
                session.AddComponent<SessionAcceptTimeoutComponent>();
                // 客户端连接，2秒检查一次recv消息，10秒没有消息则断开
                session.AddComponent<SessionIdleCheckerComponent, int>(NetThreadComponent.checkInteral);
            }
            ```
  
  1. 客户端调用的构造函数 <a id="TService2"></a>

      ```c#
      //用于客户端后续连接指定服务端口
      public TService(ThreadSynchronizationContext threadSynchronizationContext, ServiceType serviceType){}       
      ```

      `ET.AppStart_Init.RunAsync()` 初始网络组件

      ```c#
      Scene zoneScene = SceneFactory.CreateZoneScene(1, "Game", Game.Scene);
      ```

      `ET.SceneFactory.CreateZoneScene()`

      ```c#
      zoneScene.AddComponent<NetKcpComponent, int>(SessionStreamDispatcherType.SessionStreamDispatcherClientOuter); //Scene添加NetKcpComponent组件
      ```

      `ET.NetKcpComponentAwakeSystem.Awake()` [调用到客户端构造函数](#TService2)

      ```c#
      self.Service = new TService(NetThreadComponent.Instance.ThreadSynchronizationContext, ServiceType.Outer);
      ```

      `ET.LoginHelper.Login()` 确定连接地址

      ```c#
      accountSession = zoneScene.GetComponent<NetKcpComponent>().Create(NetworkHelper.ToIPEndPoint(address));
      ```

      `ET.NetKcpComponentAwakeSystem.Create -> ET.AService.GetOrCreate -> ET.TService.Get -> ET.TService.Create`

      ```c#
      private TChannel Create(IPEndPoint ipEndPoint, long id)
      {
        TChannel channel = new TChannel(id, ipEndPoint, this); //创建Channel
        this.idChannels.Add(channel.Id, channel);
        return channel;
      }
      ```

### TChannel 封装TCP Socket

`Server\Server.Model\Module\NetworkTCP\TChannel.cs` 网络数据收发

   ```c#
   //异步套接字操作
   private SocketAsyncEventArgs innArgs
   private SocketAsyncEventArgs outArgs
   //新连接建立 创建Channel
   public TChannel(long id, IPEndPoint ipEndPoint, TService service)
   {
      this.parser = new PacketParser(this.recvBuffer, this.Service); //解包器
      ...//关联Socket等信息
      // 下一帧再开始读写
      this.Service.ThreadSynchronizationContext.PostNext(() =>
      {
        this.StartRecv();
        this.StartSend();
      });
   }
   //开始接收
   private void StartRecv()
   {
    while(true)
    {
      if (this.socket.ReceiveAsync(this.innArgs)) //开始一个异步请求以便从连接的 Socket 对象中接收数据。 如果 I/O 操作同步完成，则为 false
      {
        return;
      }
      this.HandleRecv(this.innArgs); //处理接收的数据
    }
   }
   //处理结束
   private void HandleRecv(object o)
   {
    while(true)
    {
      bool ret = this.parser.Parse(); //解包
      if (!ret)
      {
       break;
      }

      this.OnRead(this.parser.MemoryStream); //读取数据
    }
   }
   ```

   `TChannel.OnRead -> AService.OnRead -> ET.NetKcpComponentSystem.OnRead`

   ```c#
   public static void OnRead(this NetKcpComponent self, long channelId, MemoryStream memoryStream)
   {
       //消息分发
       Session session = self.GetChild<Session>(channelId);
       if (session == null)
       {
           return;
       }
       session.LastRecvTime = TimeHelper.ClientNow();
       SessionStreamDispatcher.Instance.Dispatch(self.SessionStreamDispatcherType, session, memoryStream);
   }
   ```

### NetKcpComponent 服务端外网通讯

* `Server/Server.Hotfix/Module/Message/NetKcpComponentSystem.cs`

  1. AwakeSystem 建立TCP连接,关联网络线程池

     ```c#
     self.Service = new TService(NetThreadComponent.Instance.ThreadSynchronizationContext, address, ServiceType.Outer);
     ```

* `Server\Server.Model\Module\NetworkTCP\TChannel.cs`

### NetInnerComponent 服务器内网通讯

`ET.AppStart_Init.RunAsync` 根据`StartProcessConfig@s.xlsx`配置启动节点内网通讯服务端口:

```c#
Game.Scene.AddComponent<NetInnerComponent, IPEndPoint, int>(processConfig.InnerIPPort, SessionStreamDispatcherType.SessionStreamDispatcherServerInner);
```

### TimerComponent 定时器组件


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

  每帧 `Update` 逻辑 `TimerComponentSystem.TimerComponentUpdateSystem`
  
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

### ConsoleComponent 控制台组件

以实现Dll重载与Config重载功能

Server启动项添加配置项: `--Console=1`
