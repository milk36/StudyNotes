# idea vimrc 配置

## 目录

- [idea vimrc 配置](#idea-vimrc-配置)
  - [目录](#目录)
  - [idea 配置](#idea-配置)
    - [配置1](#配置1)
    - [配置2](#配置2)
    - [IdeaVim-EasyMotion](#ideavim-easymotion)

## idea 配置

- [ideavim-github](https://github.com/JetBrains/ideavim)
- [参考配置方案](https://cloud.tencent.com/developer/article/2158218)
  - `:actionlist` 查看 Idea 所有快捷键对应的指令
  - [actionlist 文档](https://gist.github.com/matt-snider/3b51f1c56b55324e6c05ec3d93ca4679)
- [另一个方案:需要 which-key 插件](https://github.com/LintaoAmons/CoolStuffes/blob/main/ideavim/.ideavimrc)
- 手动重新加载配置:`:source ~/.ideavimrc`
- 快速确认映射键位的字符

  在普通模式下，你可以输入 `:` 然后按 `Ctrl+k` 后跟任意键，vim 会告诉你如何引用它。例如，在正常模式下输入 `:` 然后按 `Ctrl+k` 并按 Backspace 会输出：`:<BS>`
- [ideavim nerdtree 快捷键](https://raw.githubusercontent.com/wiki/JetBrains/ideavim/NERDTree-support.md)

### 配置1

  ```vim
  " <leader>键设置（本配置为空格键
  let mapleader=' '
  "--将搜索匹配的文本高亮显示
  set hlsearch
  "--递增搜索功能：在执行搜索（使用 / 或 ? 命令）时，
  "Vim 会在您输入搜索模式的过程中逐步匹配并高亮显示匹配的文本。
  set incsearch
  "--在搜索时忽略大小写
  set ignorecase
  set smartcase
  set showmode
  "--设置相对行号
  set number
  "--当前行的绝对行号
  " set relativenumber
  "设置在光标距离窗口顶部或底部一定行数时，开始滚动屏幕内容的行为
  set scrolloff=3
  set history=100000
  set clipboard=unnamed
  "--设置返回normal模式时回到英文输入法
  set keep-english-in-normal
  " clear the highlighted search result
  " (清除高亮)
  nnoremap <Leader>sc :nohlsearch<CR>
  nnoremap <Leader>fs :w<CR>
  " Quit normal mode
  " (保存关闭)
  nnoremap <Leader>q  :q<CR>
  nnoremap <Leader>Q  :qa!<CR>
  " Move half page faster
  " (上下翻页)
  nnoremap <Leader>d  <C-d>
  nnoremap <Leader>u  <C-u>
  " 快速进入vim模式
  inoremap jj <Esc>
  inoremap jk <Esc>
  inoremap kk <Esc>
  " Quit visual mode
  vnoremap v <Esc>
  " Redo
  nnoremap U <C-r>
  " 以下是一些vim使用方法 -----------------
  " viw  vaw  ciw caw diw daw vi' va' ci' ca' di' da' vi( va( ci( ca( di( da( ...
  " ###### vim宏：normal模式下 qr 带表给r标记宏 然后vim操作。  @r 重复一次宏 10@r重复10次宏 注意（idea的提示功能干扰，可以在字符串里面先写好然后在复制黏贴）
  " ###### 7.2 替换
  "           :s/old/new - 用new替换当前行第一个old。
  "           :s/old/new/g - 用new替换当前行所有的old。
  "           :n1,n2s/old/new/g - 用new替换文件n1行到n2行所有的old。
  "           :%s/old/new/g - 用new替换文件中所有的old。
  "           :%s/^/xxx/g - 在每一行的行首插入xxx，^表示行首。
  "           :%s/$/xxx/g - 在每一行的行尾插入xxx，$表示行尾。
  "           有替换命令末尾加上c，每个替换都将需要用户确认。 如：%s/old/new/gc，加上i则忽略大小写(ignore)。
  "           有一种比替换更灵活的方式，它是匹配到某个模式后执行某种命令，
  "           法为 :[range]g/pattern/command
  "           如 :%g/^ xyz/normal dd。
  "           示对于以一个空格和xyz开头的行执行normal模式下的dd命令。
  "           于range的规定为：
  "           果不指定range，则表示当前行。
  "           ,n: 从m行到n行。
  "           : 最开始一行（可能是这样）。
  "           $: 最后一行
  "           .: 当前行
  "           %: 所有行
  " 以上是一些vim使用方法 -----------------
  " 以下是自定义快捷键-----------------
  " 复制单个单词到寄存器a并标记到o (使用快捷键 空格+y)
  " 注释:（mo => 标记o）,(+yiw =>复制当前单词到系统剪切板),("a => 寄存器a) ,("ayiw => 复制当前单词到寄存器a)
  nnoremap <Leader>y mo"+yiw"ayiw
  " 剪切单个单词到寄存器a并标记到o (使用快捷键 空格+x)
  " 注释:（mo => 标记o[标记为的是使用''来回跳标记]）,(+yiw =>复制当前单词到系统剪切板),("a => 寄存器a) ,("ayiw => 复制当前单词到寄存器a),(diw =>删除当前单词)
  nnoremap <Leader>x mo"+yiw"ayiwdiw
  " 删除单个字符串并黏贴寄存器a的内容并来回标记o和p  (使用快捷键 空格+v)
  " 注释：（mp => 标记p）,(viw => 选中当前单词),（"a => 寄存器a）,(p => 将寄存器a内容黏贴到选中的单词),（'o => 跳回标记o）,('p =>跳回标记p[标记为的是使用''来回跳标记])
  nnoremap <Leader>v mpviw"ap'o'p
  " idea内置快捷键alt+F1 (使用快捷键 空格+m)
  nnoremap <Leader>m :action SelectIn<CR>
  " idea内置快捷键control+E (使用快捷键 空格+e)
  nnoremap <Leader>e :action RecentFiles<CR>
  "生成get set方法  (使用快捷键 空格+cc)
  nnoremap <Leader>cc :action Generate<CR>
  " shift+h--zz(向上翻页)(使用快捷键 空格+h)
  nnoremap <Leader>h <S-h>zz
  " shift+l--zz(向下翻页)(使用快捷键 空格+l)
  nnoremap <Leader>l <S-l>zz
  " 以上是自定义快捷键-----------------
  " quit ==> close current window
  nnoremap <Leader>q <C-W>w
  " Window operation
  " (关于窗口操作)
  nnoremap <Leader>ww <C-W>w
  nnoremap <Leader>wd <C-W>c
  nnoremap <Leader>wj <C-W>j
  nnoremap <Leader>wk <C-W>k
  nnoremap <Leader>wh <C-W>h
  nnoremap <Leader>wl <C-W>l
  nnoremap <Leader>ws <C-W>s
  nnoremap <Leader>w- <C-W>s
  nnoremap <Leader>wv <C-W>v
  nnoremap <Leader>w\| <C-W>v
  " Tab operation
  " (切换标签)
  nnoremap tn gt
  nnoremap tp gT
  " ==================================================
  " Show all the provided actions via `:actionlist`
  " ==================================================
  " project search
  "相当于IDEA的两次shift按钮
  nnoremap <Leader>se :action SearchEverywhere<CR>
  "查找用法
  nnoremap <Leader>fu :action FindUsages<CR>
  "打断点
  nnoremap <Leader>bb :action ToggleLineBreakpoint<CR>
  "查看所有断点
  nnoremap <Leader>br :action ViewBreakpoints<CR>
  "DUG启动
  nnoremap <Leader>cd :action ChooseDebugConfiguration<CR>
  "跳转到Action
  nnoremap <Leader>ga :action GotoAction<CR>
  "跳转到实体类
  nnoremap <Leader>gc :action GotoClass<CR>
  "跳转到声明
  nnoremap <Leader>gd :action GotoDeclaration<CR>
  "跳转到文件
  nnoremap <Leader>gf :action GotoFile<CR>
  "跳转到实现类
  nnoremap <Leader>gi :action GotoImplementation<CR>
  "跳转到测试类(没有则自动建立)
  nnoremap <Leader>gt :action GotoTest<CR>
  "展开文件结构树
  nnoremap <Leader>gm :action FileStructurePopup<CR>
  "显示当前文件路径
  nnoremap <Leader>fp :action ShowFilePath<CR>
  "激活maven窗口
  "nnoremap <Leader>mv :action ActivateMavenProjectsToolWindow<CR>
  "修改所有的关联名字
  nnoremap <Leader>re :action RenameElement<CR>
  "修改当前文件的文件名
  nnoremap <Leader>rf :action RenameFile<CR>
  "显示用法
  nnoremap <Leader>su :action ShowUsages<CR>
  "关闭活动显示板
  nnoremap <Leader>tc :action CloseActiveTab<CR>
  "以下是不常用
  "打开命令管理器
  nnoremap <Leader>tl Vy<CR>:action ActivateTerminalToolWindow<CR>
  vnoremap <Leader>tl y<CR>:action ActivateTerminalToolWindow<CR>
  " built in search looks better
  " (查找字符串)
  nnoremap / :action Find<CR>
  " but preserve ideavim search
  " (vim自带的搜索)
  nnoremap <Leader>/ /
  "添加注释
  nnoremap <Leader>;; :action CommentByLineComment<CR>
  "改变视图
  nnoremap <Leader>cv :action ChangeView<CR>
  "跳转到标致
  nnoremap <Leader>gs :action GotoSymbol<CR>
  "
  nnoremap <Leader>ic :action InspectCode<CR>
  nnoremap <Leader>oi :action OptimizeImports<CR>
  nnoremap <Leader>pm :action ShowPopupMenu<CR>
  "正常启动工程
  nnoremap <Leader>rc :action ChooseRunConfiguration<CR>
  ```

### 配置2

- [参考](https://blog.csdn.net/Leivzy/article/details/132001375)

  ```vim
  " " =============================================
  " 插件
  " =============================================

  "Plug 'preservim/nerdtree'

  " =============================================
  " 基础配置
  " =============================================

  "--将搜索匹配的文本高亮显示
  set hlsearch

  "--递增搜索功能：在执行搜索（使用 / 或 ? 命令）时，
  "Vim 会在您输入搜索模式的过程中逐步匹配并高亮显示匹配的文本。
  set incsearch

  "--在搜索时忽略大小写
  set ignorecase
  set smartcase
  set showmode

  "--当前行的绝对行号
  "set number

  "--设置相对行号
  set relativenumber

  "设置在光标距离窗口顶部或底部一定行数时，开始滚动屏幕内容的行为
  set scrolloff=10
  set history=100000

  " 系统剪贴板
  set clipboard=unnamed

  "--设置返回normal模式时回到英文输入法
  set keep-english-in-normal

  " =============================================
  " 非 Leader 键设置
  " =============================================

  " 保存文件 Ctrl+s
  inoremap <c-s> <Esc>:w<cr>

  " 恢复
  nmap U <c-r>

  "--普通模式下使用回车键，向下/向上 增加一行
  nmap <CR> o<Esc>
  nmap <S-Enter> O<Esc>

  "--将 jj 和 jk 映射为 <Esc>, 快速退出插入模式
  imap jj <Esc>
  imap jk <Esc>

  "将Ctrl + s 映射为保存文档(也可以在VIM设置里将此快捷键设置为IDEA的快捷键)
  nmap <C-S> :action SaveAll<cr>
  imap <C-S> <Esc>:action　SaveAll<cr>

  " =============================================
  " Leader 键设置
  " =============================================

  " <leader>键设置（本配置为空格键)
  let mapleader=' '

  " ===================== Extract/提取 ========================

  " extract method/function 将代码提取为一个独立的方法(Ctrl + Alt + M)
  vmap <leader>em :action ExtractMethod<cr>

  " extract constant 引入常量(Ctrl + Alt + C)
  vmap <leader>ec :action IntroduceConstant<cr>

  " extract field （引入字段）的重构操作:将选中的代码片段转化为一个新的字段，并自动将选中的代码片段替换为对该字段的引用(Ctrl + Alt + F)
  vmap <leader>ef :action IntroduceField<cr>

  " extract variable （引入变量）的重构操作:将选中的代码片段抽取为一个新的变量，并自动替换选中的代码片段为新的变量引用(Ctrl + Alt + V)
  vmap <leader>ev :action IntroduceVariable<cr>

  " ===================== Goto/跳转 ========================

  " <c-F12> 文件结构
  nnoremap <c-BS> :action FileStructurePopup<CR>

  " idea内置快捷键alt+F1 (使用快捷键 空格+m)
  nnoremap <Leader>m :action SelectIn<CR>

  " idea内置快捷键control+E (使用快捷键 空格+e)
  nnoremap <Leader>e :action RecentFiles<CR>

  "相当于IDEA的两次shift按钮
  nnoremap <Leader>ga :action SearchEverywhere<CR>

  " 跳转到Action
  nnoremap <Leader>ac :action GotoAction<CR>

  " 跳转到实体类
  nnoremap <Leader>gc :action GotoClass<CR>

  " 跳转到文件
  nnoremap <Leader>gf :action GotoFile<CR>

  " F2 跳转到下一个错误或警告
  nnoremap ge :action GotoNextError<cr>

  " <A-C-Right> 向前跳转
  nnoremap <Leader>. :action Forward<CR>

  " <A-C-Left> 向后跳转
  nnoremap <Leader>, :action Back<CR>

  " 切换标签页
  " <A-Right>
  nmap <leader>l :action NextTab<cr>

  " <A-Left>
  nmap <leader>h :action PreviousTab<cr>

  " ===================== c ========================

  " <C-U> 跳转到上级方法
  nnoremap cu :action GotoSuperMethod<cr>

  " <C-B> 跳转到声明
  nnoremap cb :action GotoDeclaration<CR>

  " <A-C-B> 跳转到声明/跳转到实现类
  nnoremap <leader>cb :action GotoImplementation<CR>

  " <c-f> 查找
  nnoremap cf :action Find<CR>

  " <c-r> 替换
  nnoremap cr :action Replace<CR>

  " <C-A> 全选
  "nnoremap ca :action $SelectAll<cr>
  nnoremap ca gg V G<cr>

  " <c-w> 选中单词
  "nnoremap <leader>cw :action EditorSelectWord<CR>

  " ===================== 搜索 ========================

  " <C-S-F> 跳转到文件
  nnoremap <Leader>ff :action FindInPath<CR>


  " (vim自带的搜索)
  nnoremap / /

  " ===================== easymotion 设置 ========================
  " easymotion 设置
  set easymotion

  " easymotion 设置 两字符搜索
  nmap <leader><leader>s <Plug>(easymotion-s2)

  " ===================== Run/运行 ========================

  " <C-S-F10> 运行当前类
  nnoremap <Leader>rr :action RunClass<CR>

  " <S-F10> 运行当前文件
  nnoremap <Leader>ru :action Run<CR>
  ```

### IdeaVim-EasyMotion

[IdeaVim-EasyMotion](https://github.com/AlexPl292/IdeaVim-EasyMotion)

模拟的是`Vim-easymotion`插件

需要事先安装`IdeaVim-EasyMotion`和`AceJump`这两个Idea的插件.

使用方法：`<leader><leader>s` 两次`<leader>` 然后输入`s` 即可
