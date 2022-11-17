github相关配置操作
---
* [linux配置GitHub并简单的操作git](https://blog.csdn.net/Magic_Ninja/article/details/80640902)
  
  1. `ssh-keygen` 现在linux生成私钥和公钥
  1. 复制公钥文件中的内容到github

      `/root/.ssh/id_rsa.pub` -> github `settings > SSH and GPG keys > new SSH key`
  1. linux git配置全局 `user.name`和`user.email`
      ```sh    
      git config --global user.name "your name"
      git config --global user.email "your email"
      ```