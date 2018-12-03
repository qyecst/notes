<!--
{
    "title": "vim相关",
    "create": "2018-05-16 15:02:26",
    "modify": "2018-12-02 19:40:55",
    "tag": [
        "vim"
    ],
    "info": []
}
-->

## vim配置

```vim
"高亮当前行 ctermbg/背景色 ctermfg/前景色
"取值red/红 white/白 black/黑 green/绿 yellow/黄 blue/蓝 cyan/青色；可添加dark/light前缀
set cursorline
hi CursorLine cterm=NONE ctermbg=233 ctermfg=NONE
"set cursorcolumn
"hi CursorColumn cterm=NONE ctermbg=darkgray ctermfg=white
"-------------------------------------------------------------------------------
"显示行号
set number
"-------------------------------------------------------------------------------
"语法高亮
syntax on
"-------------------------------------------------------------------------------
"显示标尺
set ruler
"-------------------------------------------------------------------------------
"去掉有关vi一致性模式，避免以前版本的一些bug和局限
set nocompatible
"-------------------------------------------------------------------------------
"设置Encoding为UTF-8
set encoding=utf-8
"-------------------------------------------------------------------------------
"新建.c,.h,.sh,.java文件，自动插入文件头
autocmd BufNewFile *.cpp,*.[ch],*.sh,*.java exec ":call SetTitle()"
"定义函数SetTitle，自动插入文件头
func SetTitle()
    "如果文件类型为.sh文件
    if &filetype == 'sh'
        call setline(1,"\#########################################################################")
        call append(line("."), "\# File Name: ".expand("%"))
        call append(line(".")+1, "\# Author: qyecst")
        call append(line(".")+2, "\# Version: 0.1")
        call append(line(".")+3, "\# Created Time: ".strftime("%Y-%m-%d %H:%M:%S"))
        call append(line(".")+4, "\# Modified Time: ".strftime("%Y-%m-%d %H:%M:%S"))
        call append(line(".")+5, "\#########################################################################")
        call append(line(".")+6, "\#!/bin/bash")
        call append(line(".")+7, "")
    else
        call setline(1, "/*************************************************************************")
        call append(line("."), "    > File Name: ".expand("%"))
        call append(line(".")+1, "    > Author: qyecst")
        call append(line(".")+2, "    >Version: 0.1")
        call append(line(".")+3, "    > Created Time: ".strftime("%Y-%m-%d %H:%M:%S"))
        call append(line(".")+4, "    > Modified Time: ".strftime("%Y-%m-%d %H:%M:%S"))
        call append(line(".")+5, " ************************************************************************/")
        call append(line(".")+6, "")
    endif
    if &filetype == 'cpp'
        call append(line(".")+7, "#include<iostream>")
        call append(line(".")+8, "using namespace std;")
        call append(line(".")+9, "")
    endif
    if &filetype == 'c'
        call append(line(".")+7, "#include<stdio.h>")
        call append(line(".")+8, "")
    endif
endfunc
"新建文件后，自动定位到文件末尾
autocmd BufNewFile * normal G
"-------------------------------------------------------------------------------
"新建标签,以Ctrl+N触发
map <C-n> :tabnew<CR>
"-------------------------------------------------------------------------------
"列出当前目录文件,以Ctrl+M触发
map <C-m> :tabnew .<CR>
"-------------------------------------------------------------------------------
"自动缩进
set autoindent
"-------------------------------------------------------------------------------
"使用空格代替制表符
set expandtab
"Tab键的宽度
set tabstop=8
"统一缩进为8
set softtabstop=8
set shiftwidth=8
"-------------------------------------------------------------------------------
"禁止生成临时文件
set nobackup
set noswapfile
"-------------------------------------------------------------------------------
"我的状态行显示的内容,包括文件类型和解码
"set statusline=%F%m%r%h%w\ [FORMAT=%{&ff}]\ [TYPE=%Y]\ [POS=%l,%v][%p%%]\ %{strftime(\"%d/%m/%y\ -\ %H:%M\")}
set statusline=[%F]\ %y%r%m%*%=[Line:%l/%L,Column:%c]\ [%p%%]\ [%{strftime(\"%d/%m/%y\-\%H:%M\")}]
"总是显示状态行
set laststatus=2
"-------------------------------------------------------------------------------
"高亮显示匹配的括号
set showmatch
"-------------------------------------------------------------------------------
iab xdate <c-r>=strftime("%Y-%m-%d %H:%M:%S")<cr>
```

## 配置

```vim
"Date: 2018-12-02 20:11:39

set cursorline
hi CursorLine cterm=NONE ctermbg=233 ctermfg=NONE
"set cursorcolumn
"hi CursorColumn cterm=NONE ctermbg=darkgray ctermfg=white
set number
syntax on
set ruler
set nocompatible
set encoding=utf-8
set nobackup
set noswapfile
set showmatch
set autoindent
set shiftwidth=4
set tabstop=4
set expandtab
set softtabstop=4
```
