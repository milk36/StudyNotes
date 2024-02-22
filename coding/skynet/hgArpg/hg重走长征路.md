# Hg重走长征路

## 目录

- [Hg重走长征路](#hg重走长征路)
  - [目录](#目录)
  - [环境配置](#环境配置)
    - [ubuntu 配置](#ubuntu-配置)
    - [Openssl 1.1.1安装](#openssl-111安装)
    - [创建启动脚本](#创建启动脚本)
    - [初始化配置](#初始化配置)
  - [配置](#配置)
    - [测试mongodb端口](#测试mongodb端口)
    - [设置 sh 脚本执行权限](#设置-sh-脚本执行权限)

## 环境配置

### ubuntu 配置

ubuntu 22.04 LTS

```sh
sudo apt install -y protobuf-compiler protobuf-c-compiler libreadline-dev autoconf git subversion telnet netcat libcurl4-openssl-dev libssl-dev gawk

sudo apt-get install build-essential libssl-dev libcurl4-gnutls-dev libexpat1-dev gettext unzip

sudo apt install build-essential
sudo apt install python3
sudo ln -s /bin/python3 /bin/python
```

### [Openssl 1.1.1安装](https://www.cnblogs.com/tinywan/p/13961228.html)

Ubuntu 22.04 LTS Openssl为 3.0, 因此 `skynet` 编译无法通过

### 创建启动脚本

`create_server.sh gameserver_lyh`

### 初始化配置

```sh
python3 tools/script/import_servers.py --appid=arpg --config=tools/script/servers.dev.config --loginserver="127.0.0.1:8885" --globalid="1" #将配置文件写入登陆服数据库

python3 tools/script/generate_gameserver_config.py --config=tools/script/servers.dev.config --out=./gameserver/src/app/config --globalid="1" #输出配置文件到gameserver目录
  
```

## 配置

### 测试mongodb端口

`nc -v 127.0.0.1 28028`

### 设置 sh 脚本执行权限

```sh
find . -name "*.sh" -maxdepth 1 -exec chmod 777 {} \;
find gameserver/ -name "*.sh" -exec chmod 777 {} \;
find loginserver/ -name "*.sh" -exec chmod 777 {} \;
find tools/ -name "*.sh" -exec chmod 777 {} \;
```