# linux vimrc 配置

## 目录

- [linux vimrc 配置](#linux-vimrc-配置)
  - [目录](#目录)
  - [vimrc 配置](#vimrc-配置)

## vimrc 配置

```vim
" 插件设置，这里使用了vim-plug
call plug#begin('~/ . vim/plugged')

" 启动页
Plug 'mhinz/vim-startify'
" catppuccin 主题
Plug 'catppuccin/nvim', { 'as': 'catppuccin' }
" 配色
Plug 'altercation/vim-colors-solarized'
Plug 'morhetz/gruvbox'
" 状态栏
Plug 'vim-airline/vim-airline'
Plug 'vim-airline/vim-airline-themes'
" 缩进线
Plug 'Yggdroot/indentLine'
" 文件树
Plug 'preservim/nerdtree'
" 文件搜索
Plug 'ctrlpvim/ctrlp.vim'
" 光标快速定位
Plug 'easymotion/vim-easymotion'
" 快速编辑括号等内容 
Plug 'tpope/vim-surround'
" fzf模糊搜索
Plug 'junegunn/fzf', { 'do': { -> fzf#install() } }
Plug 'junegunn/fzf.vim'
" far模糊替换
Plug 'brooth/far.vim'
" vim go
Plug 'fatih/vim-go', { 'do': ':GoUpdateBinaries' }


call plug#end()

" ===================================================

" 常用设置
" 设置行号
set number
" 高亮搜索
set hlsearch
" 系统剪贴板
set clipboard=unnamed
" 相对当前行行数显示
set relativenumber

" 主题配色设置
colorscheme catppuccin-frappe " catppuccin-latte, catppuccin-frappe, catppuccin-macchiato, catppuccin-mocha
"set background=dark
"colorscheme gruvbox 
" 一些方便的映射
" 设置leader键为空格
let mapleader=' ' 
" 使用jj进入normal模式
inoremap jj <Esc>`^
" 空格+w 保存文件
inoremap <leader>w <Esc>:w<cr>
" 空格+q 保存并退出文件
inoremap <leader>q <Esc>:wq<cr>

" NERDTree 设置
" 开启/关闭文件树
nnoremap <leader>g :NERDTreeToggle<CR>
" 当前文件显示在文件树中
nnoremap <leader>v :NERDTreeFind<CR>
" set width
let NERDTreeWinSize=31
" show hidden files
let NERDTreeShowHidden=1
" not show files listed
let NERDTreeIgnore= [
             \ '\.git$', '\.hg$', '\.svn$', '\.stversions$', '\.pyc$', '\.pyo$', '\.swp$',
             \ '\.DS_Store$', '\.sass-cache$', '__pycache__$', '\.egg-info$', '\.ropeproject$',
             \ ]

" CtrlP 设置
let g:ctrlp_map='<c-p>'
let g:ctrlp_cmd='CtrlP'
" easymotion 设置 两字符搜索
nmap <leader><leader>s <Plug>(easymotion-s2)

" =====================================================================
" 测试映射设置
nmap  <leader>t :echo "test key"


" 快速切换窗口
nnoremap <C-j> <C-w>j<cr>
nnoremap <C-h> <C-w>h<cr>
nnoremap <C-k> <C-w>k<cr>
nnoremap <C-l> <C-w>l<cr>
" 反撤销
nnoremap U <c-r><cr>
" 当前目录模糊搜索文件
nnoremap <leader>p :Files  
" 全局模糊搜索字符串
nnoremap <leader>s :Ag
```