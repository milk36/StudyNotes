# github相关配置操作

* [linux配置GitHub并简单的操作git](https://blog.csdn.net/Magic_Ninja/article/details/80640902)

  1. `ssh-keygen` 现在linux生成私钥和公钥

    ```sh
    ssh-keygen -f github_key
    ```

  1. 复制公钥文件中的内容到github

      `/root/.ssh/id_rsa.pub` -> github `settings > SSH and GPG keys > new SSH key`
  1. linux git配置全局 `user.name`和`user.email`
  
      ```sh
      git config --global user.name "your name"
      git config --global user.email "your email"
      ```

## 建库

### 新建远程仓库,并关联

1. 新建库关联
  
    ```sh
    touch README.md
    git init
    git add README.md
    git commit -m "first commit"
    git remote add origin http://119.29.18.237:3688/git/root/dubbo-samples.git #添加远程仓库
    git push -u origin master #推送到 origin 远程仓库 master分支
    ```

1. 已有仓库关联

    ```shell
    git remote add origin http://119.29.18.237:3688/git/root/dubbo-samples.git #添加远程仓库
    git push -u origin master
    ```

## 远程覆盖本地git仓库

### git clean 清理本地目录

git clean 命令用来从你的工作目录中删除所有没有 `tracked` 过的文件。

```sh
git clean -n #是一次clean的演习, 告诉你哪些文件会被删除. 记住他不会真正的删除文件, 只是一个提醒

git clean -f #删除当前目录下所有没有track过的文件. 他不会删除.gitignore文件里面指定的文件夹和文件, 不管这些文件有没有被track过

git clean -f <path> #删除指定路径下的没有被track过的文件
```

### git强制覆盖

```sh
git fetch --all #拉取所有更新，不同步；
git reset --hard origin/master #本地代码同步线上最新版本(会覆盖本地所有与远程仓库上同名的文件)；
git pull #再更新一次（其实也可以不用，第二步命令做过了其实）
```
