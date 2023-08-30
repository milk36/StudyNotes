<!-- markdownlint-disable MD024 -->
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

### 匹配

* `LSConstValue.MatchCount` 匹配成功人数
* `C2G_MatchHandler` -> `G2Match_MatchHandler` -> `MatchComponentSystem.Match` 匹配创建房间逻辑

  -> `Match2Map_GetRoomHandler` Map创建匹配房间; Room运行在Map节点里
* `Match2G_NotifyMatchSuccess` 匹配成功，通知客户端切换场景

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

## 客户端

### 战斗房间业务

* `Match2G_NotifyMatchSuccessHandler` 通知匹配成功协议逻辑
* `OneFrameInputsHandler` 处理接收到服务端推送的帧数据

### UI

* `LSOperaComponentUpdateSystem` 监听键盘输入

## 导出数据

* 配置表json导出目录:`Unity\Assets\Res\Editor\ET\Luban`