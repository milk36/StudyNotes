gradle学习笔记
---
## 版本特性
### Gralde 4.10 移除compile等
* [找不到参数 Gradle 的方法 compile()](https://stackoverflow.com/questions/23796404/could-not-find-method-compile-for-arguments-gradle)

  请注意，Java 插件引入的 `compile, runtime, testCompile, 和 testRuntime` 等配置到**Gradle 4.10**以后版本已被弃用, 并最终被移除于**Gradle 7.0**

  上述配置应替换为 `implementation, runtimeOnly, testImplementation, 和 testRuntimeOnly`

### implementation和api的区别
* `api`: 跟 2.x 版本的 `compile`完全相同
* `implementation`: 使用了该命令编译的依赖，它仅仅对当前的`Module`提供接口
* [官方文档 -- 用于声明依赖项的配置](https://docs.gradle.org/current/userguide/java_library_plugin.html#sec:java_library_configurations_graph)