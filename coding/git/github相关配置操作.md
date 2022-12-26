github相关配置操作
---
* [linux配置GitHub并简单的操作git](https://blog.csdn.net/Magic_Ninja/article/details/80640902)

  1. `ssh-keygen` 现在linux生成私钥和公钥
      ```
      ssh-keygen -f github_key
      ```
  1. 复制公钥文件中的内容到github

      `/root/.ssh/id_rsa.pub` -> github `settings > SSH and GPG keys > new SSH key`
  1. linux git配置全局 `user.name`和`user.email`
      ```sh
      git config --global user.name "your name"
      git config --global user.email "your email"
      ```
### 新建远程仓库,并关联

  1. 新建库关联
      ```shell
      touch README.md
      git init
      git add README.md
      git commit -m "first commit"
      git remote add origin http://119.29.18.237:3688/git/root/dubbo-samples.git
      git push -u origin master
      ```

  1. 已有仓库关联
      ```shell
      git remote add origin http://119.29.18.237:3688/git/root/dubbo-samples.git
      git push -u origin master
      ```