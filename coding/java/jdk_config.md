JDK配置事项
---
1. JDK升级中碰到的问题
    * [1.8 -> 11 Java环境变量配置不生效](https://zhuanlan.zhihu.com/p/376054382)
    
        `Path中删除C:\Program Files\Common Files\Oracle\Java\javapath;即可。`
1. [jdk11（Java11）之后没有 jre（Java Runtime Environment）环境用户变量配置解决方法](https://bbs.huaweicloud.com/blogs/242257)
    ```sh
    $ jlink.exe --module-path jmods --add-modules java.desktop --output jre
    ```