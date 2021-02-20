---
title: "小白都能快速上手的Vim配置"
date: 2021-02-20T09:56:31+08:00
draft: false
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Desc Text."
disableHLJS: true
disableShare: true
disableHLJS: false
---

# 首先把所有的vim相关的都删除。

```
cd

rm -rf .vim*
```

# 创建自己的.vimrc

```
vim .vimrc
```

# 一些基本的设置

在.vimrc中添加下面的代码

```
" basic set
set number
set noswapfile
set encoding=utf-8
set fileencodings=utf-8,gb18030
set backspace=eol,start,indent
set laststatus=2
set colorcolumn=80
set cursorline
set linebreak
set autoindent
set ignorecase
set smartcase
set ruler
set diffopt+=internal,indent-heuristic,algorithm:patience
set showcmd
set clipboard^=unnamed,unnamedplus
set showmode
set mouse=a
set tabstop=2
set shiftwidth=4
set expandtab
set softtabstop=2
set showmatch
set incsearch
set nobackup
set autoread
set wildmenu
set wildmode=longest:list,full
set nofoldenable

filetype plugin indent on
syntax on

```

有了上面的设置，会让你的vim更好用一些。

每个参数的含义，可以看下阮一峰写的[vim配置入门](https://www.ruanyifeng.com/blog/2018/09/vimrc.html)

# 安装vim-plug

```
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

我是用vim-plug来管理vim的插件配置的，用起来比较简单。

它的思路是这样的，把github的vim配置clone下来，然后加载。

# 安装一些插件

在.vimrc中继续添加下面的代码

```
" Plugs
call plug#begin()
Plug 'luochen1990/rainbow'
Plug 'jiangmiao/auto-pairs'
Plug 'mechatroner/rainbow_csv'
Plug 'liuchengxu/space-vim-theme'
Plug 'lvht/mru'
Plug 'preservim/tagbar'
Plug 'preservim/nerdtree'
Plug 'Xuyuanp/nerdtree-git-plugin'
Plug 'ryanoasis/vim-devicons'
Plug 'liuchengxu/nerdtree-dash'
Plug 'fatih/vim-go', { 'do': ':GoUpdateBinaries' }
Plug 'godlygeek/tabular'
Plug 'plasticboy/vim-markdown'
Plug 'liuchengxu/eleline.vim'
Plug 'tpope/vim-fugitive'
call plug#end()
```

这些是我比较常用的插件

- rainbow 是一个每个括号都用不同颜色区分，增加代码的可读性
- auto-pairs 是自动补全括号
- rainbow_csv 是打开csv文件更好看一些的插件
- space-vim-theme 是vim的一个主题
- mru 是最近最常使用的文件
- tagbar 显示代码结构的
- nerdtree + nerdtree-git-plugin + vim-devicons + nerdtree-dash 这几个搭配起来，展示一个更好看的文件目录
- vim-go 写go必备
- tabular + vim-markdown 写markdown必备的
- eleline 状态栏更好看一些
- vim-fugitive vim的git插件


# 插件的自定义设置

安装了这么多插件，一般可能会自定义一下，有些插件都提供了一些变量，我们可以通过let g:xxx的方式去自定义

这些设置也是在vimrc中，要在插件安装的下面

```
" plug settings
let g:rainbow_active=1
```

# keymap的设置

我们可以设置一些快捷键加快操作。

vim 有一个leader键，这个键的作用是按下之后，再按别的键，触发一些命令。 之所以有这个leader键，就是为了防止用户自己的快捷键，覆盖了默认的。 vim默认的leader键是|,也就是enter上面那个中竖线。

```
" key map 
nnoremap <silent> <c-m> :Mru<cr>
nnoremap <silent> <c-p> :call fzf#Open()<cr>
nnoremap <silent> <leader>t :TagbarToggle<cr>
nnoremap <silent> <leader>e :NERDTreeToggle<cr>
nnoremap <silent> <leader>f :NERDTreeFind<cr>
nnoremap <silent> <leader>c :call lv#Term()<cr>
```

# 如果你想和我的一样

参考这个github项目：https://github.com/zhenfeng-zhu/vim

```
git clone --recursive https://github.com/zhenfeng-zhu/vim.git ~/.vim

ln -s ~/.vim/init.vim ~/.vimrc
```

然后就可以愉快的自己定制了。
