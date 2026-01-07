
# svn 操作

## 目录

- [svn 操作](#svn-操作)
  - [目录](#目录)
  - [svn 忽略文件](#svn-忽略文件)
    - [svn:ignore](#svnignore)

## svn 忽略文件

### svn:ignore

1. 在项目根目录下创建一个名为 `.svnignore` 的文件，把需要忽略的文件名或文件夹名写入此文件中，一行一个。
2. 在命令行中执行 `svn propset svn:ignore .svnignore .`，将 `.svnignore` 文件添加到版本控制中。
3. 查看当前目录的忽略规则:`svn propget svn:ignore .` 
4. 测试命令:`svn status` , 将不再显示被忽略的文件或文件夹
5. `.svnignore`配置文件示例:

    ```txt
    Library
    Temp
    obj
    Logs
    Builds
    HybridCLRData
    AssetBundles
    UserSettings

    .idea
    .vscode
    .DS_Store
    *.log
    ```
