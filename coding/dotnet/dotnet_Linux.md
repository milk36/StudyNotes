# Dotnet 与 Linux

* [yum 包管理器安装](https://learn.microsoft.com/zh-cn/dotnet/core/install/linux-centos#centos-linux-7)
  * 添加 Microsoft 包存储库 `sudo rpm -Uvh https://packages.microsoft.com/config/centos/7/packages-microsoft-prod.rpm`
  * 安装SDK `sudo yum install -y dotnet-sdk-6.0`
* [dotnet-install 安装脚本](https://learn.microsoft.com/zh-cn/dotnet/core/tools/dotnet-install-script)
  * 将最新的长期支持 (LTS 目前为6.0) 版本安装到默认位置

    ```sh
    ./dotnet-install.sh --channel LTS
    ```
