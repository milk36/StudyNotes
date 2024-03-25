# neovim 配置

## 目录

- [neovim 配置](#neovim-配置)
  - [目录](#目录)
  - [资源](#资源)
  - [配置](#配置)
  - [lunarVim](#lunarvim)
    - [lunarVim-默认快捷键](#lunarvim-默认快捷键)
    - [使用neovide 启动 lunarVim](#使用neovide-启动-lunarvim)
    - [配置参考](#配置参考)
    - [自定义配置](#自定义配置)
    - [常用快捷键](#常用快捷键)
    - [mason 目录](#mason-目录)
    - [jdtls 的安装](#jdtls-的安装)
  - [lunarVim 安装中遇到问题/修复方式](#lunarvim-安装中遇到问题修复方式)
    - [修复 monospace 字体问题](#修复-monospace-字体问题)
    - [Mason jdtls 指向本地资源](#mason-jdtls-指向本地资源)
    - [nvim-tree `\zf` 问题](#nvim-tree-zf-问题)

## 资源

- [neovim 官网](https://neovim.io/)
- [neovide 官网](https://neovide.dev/)
- [NvChad 官网](https://nvchad.com/)
- [Neovim 配置实战：从 0 到 1 打造自己的 IDE](https://github.com/nshen/learn-neovim-lua/tree/main)
- [Neovim Java, Rust, C/C++, JavaScript 等编程语言开发环境](https://github.com/JavaHello/nvim)

## 配置

- [配置右键打开](https://blog.csdn.net/jaray/article/details/86490375)
- windows 配置目录

  `C:\Users\[username]\AppData\Local\nvim`
- [`vim-plug` init.lua 配置方式](https://gitlab.com/sultanahamer/dotfiles/-/blob/master/nvim/lua/plugins.lua?ref_type=heads)
- [winget 配置](https://www.sujx.net/2023/06/30/powershell-winget/index.html)

  开始使用本地代理：spp

  停止使用本地代理：upp

  - [windows 先执行安装指令](https://github.com/junegunn/vim-plug?tab=readme-ov-file#windows-powershell-1)
  
    ```sh
    iwr -useb https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim |`
    ni "$(@($env:XDG_DATA_HOME, $env:LOCALAPPDATA)[$null -eq $env:XDG_DATA_HOME])/nvim-data/site/autoload/plug.vim" -Force
    ```

  - 配置 `plugins.lua`

    ```lua
    local vim = vim
    local Plug = vim.fn['plug#']
    vim.call('plug#begin', '~/.config/nvim/plugged')

    -- 启动页
    Plug 'mhinz/vim-startify'

    vim.call('plug#end')
    ```

## lunarVim

- windows默认安装目录: `C:\Users\<your-user>\AppData\Roaming\lunarvim`

### [lunarVim-默认快捷键](https://www.lunarvim.org/docs/beginners-guide/keybinds-overview)

- 提示： `<M>` 在 Windows/Linux 上为 `alt` ，在 macOS 上为 `option`

### [使用neovide 启动 lunarVim](https://github.com/neovide/neovide/issues/1219)

- [参考:How do I use LunarVim in Neovide?](https://www.lunarvim.org/docs/faq#how-do-i-use-lunarvim-in-neovide)
- 复制默认启动脚本:`c:/users/<your-user>/.local/bin/lvim.ps1`
- 修改脚本名为 `lvide.ps1`，修改启动脚本内容
  
  ```sh
  #Windows:
  #before:

  nvim -u "$env:LUNARVIM_BASE_DIR\init.lua" @args 
  #after:

  neovide -- -u "$env:LUNARVIM_BASE_DIR\init.lua" @args
  ```

- 配置文件目录 `C:\Users\<your-user>\AppData\Local\lvim`

### 配置参考

- [QingYunA lvim config](https://github.com/QingYunA/lvim-conifg/tree/main)
- [Linux上配置LunarVim](https://www.mintimate.cn/2023/01/10/guideForLunarvim/)

### 自定义配置

- [插件配置示例](https://www.lunarvim.org/docs/configuration/plugins/example-configurations)

### 常用快捷键

- `<Space>sk` 快捷键映射列表
- `<Space>ls` 文件结构
- 搜索文件
  - `<Space>;` 仪表盘
  - `<Space>sf` 搜索文件
  - `<Space>sr` 搜索最近开启的文件
  - `<Space>e` 文件树
- 主题
  - `<Space>sc` 切换主题
  - `<Space>sp` 预览切换主题
- 终端
  - `<C-\>` 切换终端
  - `<A-1>` 水平终端
  - `<A-2>` 垂直终端
  - `<A-3>` 浮动终端

### mason 目录

`C:\Users\<your-user>\AppData\Roaming\nvim-data\mason\packages`

### jdtls 的安装

- 使用 `nvim-jdtls` 插件安装 Lsp

## lunarVim 安装中遇到问题/修复方式

- 重装清理目录:
  - `C:\Users\<your-user>\AppData\Roaming\nvim-data`
  - `C:\Users\<your-user>\AppData\Local\nvim-data`

### 修复 monospace 字体问题

`lunarVim` 运行时目录 :`C:\Users\<your-user>\AppData\Roaming\lunarvim`

- [参考](https://github.com/LunarVim/LunarVim/pull/4447/files)

  `lua/lvim/config/settings.lua` 文件中内容注释掉即可

  ```lua
  -- guifont = "monospace:h17", 
  ```

### Mason jdtls 指向本地资源

- 参考: [Mason-jdtls 指向本地资源](https://github.com/williamboman/mason.nvim/issues/1508)
- `C:\Users\<your-user>\AppData\Roaming\nvim-data\mason\registries\github\mason-org\mason-registry\registry.json`

  ```json
  {
          "target": "win",
          "files": {
            "jdtls.tar.gz": "D:/download/1-download-Zip/jdt-language-server-1.33.0-202402151717.tar.gz",
            "lombok.jar": "https://projectlombok.org/downloads/lombok.jar"
          },
          "config": "config_win/"
  }
  ```

### nvim-tree `\zf` 问题

> 最后直接用 NeoTree 替代了

`C:\Users\<your-user>\AppData\Roaming\lunarvim\site\pack\lazy\opt\nvim-tree.lua\lua\nvim-tree\explorer\watch.lua`

```lua
local function is_folder_ignored(path)
  if utils.is_windows then
    path = string.gsub(path, "\\", "/")
  end
  -- print(path)
  for _, ignore_dir in ipairs(M.ignore_dirs) do
    if utils.is_windows then
      ignore_dir = string.gsub(ignore_dir, "\\", "/")
    end
    -- print("ignore_dir",ignore_dir)
    if vim.fn.match(path, ignore_dir) ~= -1 then
      return true
    end
  end

  return false
end
```
