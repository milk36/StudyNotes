# GameDevelopmentKit 中的帧同步笔记

* [GameDevelopmentKit](https://github.com/XuToWei/GameDevelopmentKit)

## 启动

### 客户端入口

* `GlobalComponent` 进入帧同步demo

  ```c#
  public void Awake()
  {
      // AppType = AppType.Demo;
      AppType = AppType.LockStep;
  }
  ```

## 帧同步服务端

### 战斗房间业务

* `Room`

  ```c#
  LSConstValue.UpdateInterval=50;//固定帧间隔50毫秒
  ```
  
* `FrameMessageHandler` 处理输入帧数据
* `LSServerUpdaterUpdateSystem` 广播帧数据

  ```c#
  protected override void Update(LSServerUpdater self)
  {
      Room room = self.GetParent<Room>();
      long timeNow = TimeHelper.ServerFrameTime();
  
  
      int frame = room.AuthorityFrame + 1;
      if (timeNow < room.FixedTimeCounter.FrameTime(frame))//判断固定帧间隔执行时间
      {
          return;
      }

      OneFrameInputs oneFrameInputs = self.GetOneFrameMessage(frame);
      ++room.AuthorityFrame;

      OneFrameInputs sendInput = new();
      oneFrameInputs.CopyTo(sendInput);

      RoomMessageHelper.BroadCast(room, sendInput);//广播帧数据
  
      room.Update(oneFrameInputs);//继续驱动 update
  }
  ```
