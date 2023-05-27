<!-- markdownlint-disable MD033 -->
# Dotnet GC 内构

## GC in .NET

* 应用类型
  * Workstation 工作站; UI应用程序等APP应用;占用资源少
  * Server 服务器;大吞吐量,多线程GC
  * `System.GC.Server` 默认:工作站垃圾回收. 它等效于将值设置为 false.
  * `server` 模式可以处理比 `workstation` 模式更多更大的托管堆，拥有更大的吞吐量
* GC 的模式
  * `Non-Concurrent Workstation` 非并发工作站, 整个停顿 `stop the world`
    * 最理想的, 没人打扰
  * `Background Server` 后台并发GC,GC部分与应用程序并行执行
    * 几乎不发生停顿
    * (目前)无压缩;缺点之一
  * `Foreground garbage collection` 前台GC : 在后台垃圾收集期间对 `临时段` (G0 和 G1) 的收集称为 *前台垃圾收集*。

### Background GC 后台GC

[参考:Tame Your GC -- 驯服GC](https://prodotnetmemory.com/slides/TameYourGC)  

1. 下图显示对独立专用线程执行的 工作站 后台GC:

   <img src="./img/background-workstation-garbage-collection.png" width="60%">
1. 下图显示对独立专用线程执行的 服务器 后台GC:

   <img src="./img/background-server-garbage-collection.png" width="60%">

### HeapHardLimit 堆限制 与 堆限制百分比

[在小容器场景中使用服务器GC运行 - 0](https://devblogs.microsoft.com/dotnet/running-with-server-gc-in-a-small-container-scenario-part-0/)

[在小容器场景中使用服务器GC运行 - 1 HardLimit](https://devblogs.microsoft.com/dotnet/running-with-server-gc-in-a-small-container-scenario-part-1-hard-limit-for-the-gc-heap/)

为了支持容器场景，添加了2个 `HardLimit` 配置-

`GCHeapHardLimit` -指定GC堆的硬限制

`GCHeapHardLimitPercent` -指定允许此进程使用的物理内存百分比

如果两者都指定，则首先检查 `GCHeapHardLimit` ，并且仅在未指定时会检查 `GCHeapHardLimitPercent` 。

如果两者都未指定，但进程正在具有内存的容器内运行 `limit` 指定，将其作为硬限制：max（20mb，容器内存限制的75%）

如果指定了其中一个 `HardLimit` 配置，并且进程在具有内存限制的容器中运行，则GC堆使用量不会超过 `HardLimit`，

但总内存仍然是容器上的内存限制，因此当我们计算内存负载时，它是基于容器内存限制的。

* [堆限制](https://learn.microsoft.com/zh-cn/dotnet/core/runtime-config/garbage-collector#heap-limit): 指定 GC 堆和 GC 簿记的最大提交大小（以字节为单位）默认:20 MB
  * `runtimeconfig.json` -> `System.GC.HeapHardLimit`
* [堆限制百分比](https://learn.microsoft.com/zh-cn/dotnet/core/runtime-config/garbage-collector#heap-limit-percent): 指定允许的 GC 堆使用量占总物理内存的百分比。
  * `runtimeconfig.json` -> `System.GC.HeapHardLimitPercent` 默认: 75%

### CPU 超过 64 个的处理器

* [关联](https://learn.microsoft.com/zh-cn/dotnet/core/runtime-config/garbage-collector#affinitize-mask): `runtimeconfig.json` ->	`System.GC.NoAffinitize`

   默认：将垃圾回收线程与处理器关联。 它等效于将值设置为 false.
* [关联掩码](https://learn.microsoft.com/zh-cn/dotnet/core/runtime-config/garbage-collector#affinitize-mask) `runtimeconfig.json` -> `System.GC.HeapAffinitizeMask`

   指定垃圾回收器线程应使用的确切处理器数。
* [关联范围](https://learn.microsoft.com/zh-cn/dotnet/core/runtime-config/garbage-collector#affinitize-mask) `runtimeconfig.json` -> `System.GC.HeapAffinitizeRanges`

   指定用于垃圾回收器线程的处理器列表。

## GC调优

* [runtimeconfig.json 配置文件](https://learn.microsoft.com/zh-cn/dotnet/core/runtime-config/#runtimeconfigjson)
  
  构建项目时，将在输出目录中生成 [appname].runtimeconfig.json 文件。 如果项目文件所在的文件夹中存在 runtimeconfig.template.json 文件，它包含的任何配置选项都将插入到 [appname].runtimeconfig.json 文件中。
* [用于垃圾回收的运行时配置选项](https://learn.microsoft.com/zh-cn/dotnet/core/runtime-config/garbage-collector)
* [调试 .NET Core 中的内存泄漏](https://learn.microsoft.com/zh-cn/dotnet/core/diagnostics/debug-memory-leak)
* [分析.net core在linux下内存占用过高问题](https://www.cnblogs.com/zhenglisai/p/14751677.html)
* [.net 相关分析工具](https://www.yuque.com/et-xd/docs/ikuilq#ExIo0)

### PerfView 使用

[PerfView：终极 .NET 性能工具 -- youtube](https://www.youtube.com/watch?v=qGEeZZBwVp4&ab_channel=InfoQ)

[Perfview的使用](https://www.cnblogs.com/lwhkdash/p/9969215.html)

[dotnet-gcdump 配合Perfview的使用](https://youtu.be/FchQ2GUx5lY)

### PerfView 如何分析 Linux 跟踪文件

[使用 PerfCollect 跟踪 .NET 应用程序](https://learn.microsoft.com/zh-cn/dotnet/core/diagnostics/trace-perfcollect-lttng)

1. 安装 `perfcollect`

   ```sh
   curl -OL https://aka.ms/perfcollect #下载perfcollect

   chmod +x perfcollect #授权可执行
   sudo ./perfcollect install #安装 trace 组件
   ```

1. `perfcollect` 收集跟踪文件

   * 启动

      ```sh
      sudo ./perfcollect collect sampleTrace #启动收集
      ```

      预期输出:`Collection started.  Press CTRL+C to stop.`

   * 配置环境变量:

      ```sh
      export DOTNET_PerfMapEnabled=1
      export DOTNET_EnableEventLog=1
      ```

   * 运行应用: `dotnet run ...`
   * 停止收集 - 按 `CTRL+C`
   * 下载 `xxx.trace.zip`, 使用 `PerfView` 进行分析
   * 打开跟踪文件: `PerfView.exe <path to trace.zip file>`
