# ET7 笔记

## ET7 服务端启动加载顺序

`Program.GameAppMain()` -> `Init.Start()` -> `CodeLoader.Start`

```c#
//-> Init.Start()
Game.AddSingleton<CodeLoader>().Start();
```

`CodeLoader.Start`

```c#
IStaticMethod start = new StaticMethod(this.model, "ET.Entry", "Start");
start.Run();
```

`Entry.Start -> Entry.StartAsync`

```c#
//-> Entry.Start -> Entry.StartAsync

//设置系统时钟精度为1毫秒,主要作用于 windows下
WinPeriod.Init();

//EntryEvent1_InitShare 共享组件初始化
await EventSystem.Instance.PublishAsync(Root.Instance.Scene, new EventType.EntryEvent1());
//EntryEvent2_InitServer 服务端组件初始化
await EventSystem.Instance.PublishAsync(Root.Instance.Scene, new EventType.EntryEvent2());
//EntryEvent3_InitClient 客户端组件初始化
await EventSystem.Instance.PublishAsync(Root.Instance.Scene, new EventType.EntryEvent3());
```
