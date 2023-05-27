<!-- markdownlint-disable MD033 -->
# dotnet 命令行工具

## dotnet

### --list-sdks

输出已安装 .NET SDK 的列表。

`dotnet --list-sdks`

## dotnet-counters

[dotnet-counters 官方文档](https://learn.microsoft.com/zh-cn/dotnet/core/diagnostics/dotnet-counters)

### 安装工具

```sh
dotnet tool install --global dotnet-counters
```

### dotnet-counters ps

列出可由 dotnet-counters 监视的 dotnet 进程。 dotnet-counters 版本 6.0.320703 及更高版本还显示每个进程的启动命令行参数（如果可用）

### dotnet-counters collect

定期收集所选计数器的值，并将它们导出为指定的文件格式以进行后续处理