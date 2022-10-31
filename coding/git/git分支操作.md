git分支操作
---
1. 从 `master` 合并到 `main`; 并移除 `master`分支
    ```sh
    #查看所有分支
    $ git branch -a
    #查看远程分支
    $ git branch -r
    # 新建一个分支，并切换到该分支
    $ git checkout -b main
    # 合并指定(master)分支内容到当前分支
    $ git merge master
    # 删除远程分支
    $ git push origin --delete master
    # 删除本地分支(master)
    $ git branch -d master
    ```