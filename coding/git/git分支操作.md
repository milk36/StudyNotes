# git分支操作

- main/master 属于项目

  主分支要尽量简洁
- my-feature 分支则属于开发者

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

1. 切换分支
  
    ```sh
    git checkout <branch>
    ```

1. 将master/main分支内容合并到当前分支

    ```sh
    # 使用 rebase 比 merge 更安全
    git rebase master
    # 或者
    git rebase main
    # 需要 使用 push -f 强制推送
    git push -f origin my-feature
    ```

1. 主干合并分支内容

    应该使用 `squash and merge`