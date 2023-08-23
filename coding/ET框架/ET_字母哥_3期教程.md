<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD024 -->
# ET_字母哥_3期教程笔记

## 服务端应用程序的发布

* 发布指令: `dotnet publish -r linux-x64 --no-self-contained --no-dependencies -c Release`

  将 `linux-x64` 替换成 `win-x64` 即可发布windows 应用程序
* `Rdier` 打包发布
  
  <img src="./img/ET6_publish_1.png" width="30%"/>

  <img src="./img/ET6_publish_2.png" width="30%"/>

## ET框架多进程部署

### 相关配置

* `StartMachineConfig` 机器内网/外网地址/守护进程端口(多机器配置)
* `StartProcessConfig` 进程ID配置/部署机器包含的进程配置
  
  需要添加 :
  |程序名|
  |-|
  |AppName|
  |string|

* `StartSceneConfig` Gate/Map 等场景配置对应进程配置
* `StartZoneConfig` 区服ID/数据库配置

### 监听进程

 Watcher 监控本机其他进程是否奔溃报错邮件/重启等操作

### 正式部署时关闭输出流(Console)日志选项

关闭对命令行的日志输出 `NLog.config`:

`<logger ruleName="ErrorConsole" name="Server" minlevel="Warn" maxlevel="Error" writeTo="ErrorConsole" enabled="false"/>`

## 热重载

* 重载Hotfix代码;添加启动参数:`--Console=1`, 使用 R 指令
* 重载配置表;添加启动参数:`--Console=1`, 使用 C 配置表名: `C UnitConfig`

### ET7 C2R_ReLoadDllHandler  Hotfix 代码

```c#
xxx Run()
{
  await Game.WaitFrameFinish(); //加上这一行,确保 Hotfix 层的热更逻辑能正确执行
  CodeLoader.Instance.LoadHotfix();                    
  EventSystem.Instance.Load();
}
```

### ReloadConfigConsoleHandler 热重载配置表

配置表数据直接热重载无需调用 `await Game.WaitFrameFinish();`

直接使用`ConfigComponent.Instance.Load();` 即可重载所有配置数据

## Robot

### 测试用例

用于测试网络消息

ET6 需要独立启动Robot进程,启动参数: `--Process=2 --Console=1`

ET7 可以在同一个进程中启动Robot,启动参数: `--Console=1`

命令行

```sh
> Robot 
> Run 1 #测试单个用例
> RunAll #测试全部用例
```

* `RobotConsoleHandler` 解析机器人启动命令

### 机器人

* `CreateRobotConsoleHandler`

启动机器人逻辑,命令行

```sh
> CreateRobot --Num 10
> CreateRobot --Num=10
```
