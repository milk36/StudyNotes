JDK配置事项
---
1. JDK升级中碰到的问题
    * [1.8 -> 11 Java环境变量配置不生效](https://zhuanlan.zhihu.com/p/376054382)
    
        `Path中删除C:\Program Files\Common Files\Oracle\Java\javapath;即可。`
1. [jdk11（Java11）之后没有 jre（Java Runtime Environment）环境用户变量配置解决方法](https://bbs.huaweicloud.com/blogs/242257)
    ```sh
    $ jlink.exe --module-path jmods --add-modules java.desktop --output jre
    ```
### JVMS 多版本JDK配置
* [JVMS--用于管理 Windows 计算机上的多个 JDK 安装](https://github.com/ystyle/jvms)
    1. 解压zip并将`jvms.exe`复制到你想要的路径
    1. 以管理员身份运行 `cmd` 或 `powershell`
    1. cd到`jvms.exe`所在的文件夹
    1. 运行 `jvms.exe init` , 设置完成！切换并安装 jdk 参见用法部分

    > jvms代理设置: `jvms.exe proxy 127.0.0.1:10809`

    > 在powershell 设置代理 `$env:http_proxy="http://127.0.0.1:10809"`

    1. `jvms.exe rls` 列表可用JDK版本可下载

    添加本地jdk版本

    1. 将 jdk 主文件夹复制到 `jvms/store`
    1. 将文件夹重命名为 17.0.1
    1. `jvms list` 检查这个
    1. `jvms switch 17.0.1`
    1. `java -version` 查看jdk版本