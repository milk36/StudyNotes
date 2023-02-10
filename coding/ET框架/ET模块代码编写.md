<!-- markdownlint-disable MD033 -->
# ET模块代码编写笔记

## ECS模式

* 实体与组件的定义方式: _实体即组件,组件即实体_

  1. Entity实体定义 `Unity/Codes/Model/Demo/Computer/Computer.cs`
  
      ```c#
      namespace ET
      {
        public class Computer:Entity,IAwake,IUpdate,IDestroy //这里必须继承Entity,实现IAwake 等生命周期接口
        {        
        }
      }
      ```

  1. Component组件定义 `Unity/Codes/Model/Demo/Computer/PCCaseComponent.cs`

      ```c#
      namespace ET
      {
        [ComponentOf(typeof(Computer))] //关联组件挂载的实体
        public class PCCaseComponent:Entity,IAwake //定义组件和实体一样都需要继承Entity,实现IAwake
        {        
        }
      }
      namespace ET
      {
        [ComponentOf(typeof(Computer))]
        public class MonitorsComponent:Entity,IAwake
        {
          
        }
      }
      ...
      ```

* System定义

  ComputerSystem 的定义:`Unity/Codes/Hotfix/Demo/Computer/ComputerSystem.cs`
  
  ```c#
  namespace ET
  {
    public static class ComputerSystem
    {
      public static void Start(this Computer self) //静态扩展 Computer添加 Start 启动方法
      {
        Log.Info("Computer Start =======");
        self.GetComponent<PCCaseComponent>().StartPower(); //调用子组件
        self.GetComponent<MonitorsComponent>().Display();
      }
    }
    /// <summary>
    /// 生命周期Awake
    /// </summary>
    public class ComputerAwakSystem: AwakeSystem<Computer>
    {
      public override void Awake(Computer self)
      {
        Log.Debug("Computer Awake ====");
      }
    }

    /// <summary>
    /// 生命周期Update
    /// </summary>
    public class ComputerUpdateSystem: UpdateSystem<Computer>
    {
      public override void Update(Computer self)
      {
        // Log.Debug("Computer Update ===");
      }
    }

    /// <summary>
    /// 生命周期Destroy
    /// </summary>
    public class ComputerDestorySystem: DestroySystem<Computer>
    {
      public override void Destroy(Computer self)
      {
        Log.Debug("Computer Destory ====");      
      }
    }
  }
  ```

  其他system的定义:`Unity/Codes/Hotfix/Demo/Computer/PCCaseComponentSystem.cs`

  ```c#
  namespace ET
  {
    public static class PCCaseComponentSystem
    {
      public static void StartPower(this PCCaseComponent self) //静态扩展 PCCaseComponent 添加 StartPower 启动方法
      {
        Log.Info("PCCaseComponent start =======");
      }
    }
  }
  ```

## 网络

### ET项目中Demo的登陆流程

<img src="./img/ET_Demo_Login_1.png" width="50%"/>

### 消息类型

||普通消息|Actor|ActorLocation|
|-|:-|-|-|
|消息|单向发送:<br/>IMessage<br/>请求与响应:<br/>IRequest<br/>IResponse |单向发送:<br/>IActorMessage<br/>请求与响应:<br/>IActorRequest<br/>IActorResponse|单向发送:<br/>IActorLocationMessage<br/>请求与响应:<br/>IActorLocationRequest<br/>IActorLocationResponse|

### Protobuf 简易通讯协议的编写

* `OuterMessage.proto` 文件路径: `ET\Proto\OuterMessage.proto`
  
  以下 `C->Client 客户端` , `R->Realm 负载服务` 添加协议内容:

  ```pb  
  //ResponseType R2C_LoginTest                //定义返回的消息类型
  message C2R_LoginTest // IRequest           //请求类型协议
  {
    int32 RpcId = 90;                         //RpcId 必须要定义, 因为IRequest和IResponse两个接口都定义了RpcId
    string Account = 1;
    string Password = 2;
  }

  message R2C_LoginTest // IResponse          //响应类型协议
  {
    int32 RpcId = 90;                         //RpcId 必须要定义, 因为IRequest和IResponse两个接口都定义了RpcId
    int32 Error = 91;
    string Message = 92;
    string GateAddress = 1;
    string Key = 2;
  }

  message C2R_SayHello // IMessage            //IMessage单向协议,不关心返回协议
  {
    string TestMsg = 1;
  }

  message R2C_SayGoodBye // IMessage          //IMessage单向协议,不关心返回协议
  {
    string GoodBye = 1;
  }
  ```

  * `RpcId` 因该是用于协议消息上下文的处理

* 服务端协议Handler

  `Server\Server.Hotfix\Demo\Login\C2R_LoginTestHandler.cs` 处理有请求和响应协议逻辑: 接收`C2R_LoginTest` 返回 `R2C_LoginTest`

  ```c#
  namespace ET
  {
    [MessageHandler] //标记为消息处理器
    public class C2R_LoginTestHandler :AMRpcHandler<C2R_LoginTest,R2C_LoginTest> //接收 C2R_LoginTest, 返回 R2C_LoginTest
    {
      protected override async ETTask Run(Session session, C2R_LoginTest request, R2C_LoginTest response, Action reply)
      {
        Log.Debug($"C2R_LoginTest recvice:{request.Account}"); //接收到 C2R_LoginTest 协议
        response.Key = "999999";
        response.Message = "TestMsg";
        reply();//返回消息 最终调用:session.Reply(response);
        await ETTask.CompletedTask; //配合 async ETTask 使用
      }
    }
  }
  ```

  `Server\Server.Hotfix\Demo\Login\C2R_SayHelloHandler.cs` 处理单项协议逻辑, 只接收指定协议

  ```c#
  namespace ET
  {
    [MessageHandler] //标记为消息处理器
    public class C2R_SayHelloHandler : AMHandler<C2R_SayHello> //接收 C2R_SayHello 消息
    {
      protected override void Run(Session session, C2R_SayHello message)
      {
        Log.Debug($"C2R_SayHello:{message.TestMsg}");//处理接收数据
        session.Send(new R2C_SayGoodBye(){GoodBye = "byby"}); //服务端主动推送数据, 只要能拿到session对象, 理论上可以再任意模块中推送消息, 和 C2R_SayHelloHandler 没有强绑定关系
      }
    }
  }
  ```

* 客户端协议Handler

  `Client\Unity.Hotfix\Codes\Hotfix\Demo\Login\LoginHelper.cs` HotfixView显示层 UI 调用,向服务端发送协议逻辑

  ```c#
  public static async ETTask LoginTest(Scene zoneScene, string address)
  {
      try
      {
          Session session = null;
          R2C_LoginTest r2CLoginTest = null;
          try
          {
              //创建以session, 建立与server的连接
              session = zoneScene.GetComponent<NetKcpComponent>().Create(NetworkHelper.ToIPEndPoint(address));
              {
                  r2CLoginTest = (R2C_LoginTest)await session.Call(new C2R_LoginTest() { Account = "yoyo", Password = "" }); //创建 C2R_LoginTest 消息, 并发送, await等待返回数据
                  //因为使用了 await 异步等待响应的形式, 所以没有定义 R2C_LoginTest 对应的 Handler
                  Log.Debug($"R2C_LoginTest key:{r2CLoginTest.Key} msg:{r2CLoginTest.Error}"); //打印返回数据
                  session.Send(new C2R_SayHello(){TestMsg = "kkk"}); //推送 C2R_SayHello 到服务端, 没有返回数据
                  await TimerComponent.Instance.WaitAsync(100); //这里是为了等待服务端推送 R2C_SayGoodBye协议的返回
              }
          }
          finally
          {
              session?.Dispose();//?. Null条件运算符, 关闭连接
          }
      }
      catch (Exception ex)
      {
          Log.Error(ex.ToString());
      }
  }
  ```
  
  `Client\Unity.Hotfix\Codes\Hotfix\Demo\Login\R2C_SayGoodByeHandler.cs` 接收服务端主动推送 R2C_SayGoodBye 消息

  ```c#
  namespace ET
  {
    [MessageHandler] //标记为消息处理器
    public class R2C_SayGoodByeHandler:AMHandler<R2C_SayGoodBye> //接收 R2C_SayGoodBye 消息
    {
      protected override void Run(Session session, R2C_SayGoodBye message)
      {
        Log.Debug($"receive:{message.GoodBye}"); //打印接收的消息
      }
    }
  }
  ```
