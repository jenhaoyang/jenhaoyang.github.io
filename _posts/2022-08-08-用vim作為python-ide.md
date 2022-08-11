---
layout: post
title: 用vim作為Python IDE
date: 2022-08-08 12:24 +0800
categories: [IDE]
tags: [ide]  
---
vim版本:8.2

# 安裝套件管理員Vundle

1. 下載Vundle plugin manager並且放到VIM套件資料夾
```
git clone https://github.com/gmarik/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

2. 建立套件管理檔
```
touch ~/.vimrc
```

3. 將下面指令複製到~/.vimrc檔案裡面
```
set nocompatible              " required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()

" alternatively, pass a path where Vundle should install plugins
"call vundle#begin('~/some/path/here')

" let Vundle manage Vundle, required
Plugin 'gmarik/Vundle.vim'

" add all your plugins here (note older versions of Vundle
" used Bundle instead of Plugin)

" ...

" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
```

4. 安裝套件
    1. 先開啟vim
    2. 輸入:
    3. PluginInstall


# 操作
* 分割畫面
    1. .vimrc 加入下面幾行
    ```
    "split navigations
    nnoremap <C-J> <C-W><C-J>
    nnoremap <C-K> <C-W><C-K>
    nnoremap <C-L> <C-W><C-L>
    nnoremap <C-H> <C-W><C-H>
    ```
    2. ctrl加上vim的方向鍵(J下, K上, L右, H左)就可以移動到不同地方的split

* Buffers
    * :ls可以看到buffer清單
    * 可以在輸入:ls後直接:b <buffer number>，就可以不用記下buffer number

* 程式碼收折
    * 在.vimrc加入下面幾行
    ```
    " Enable folding
    set foldmethod=indent
    set foldlevel=99

    " Enable folding with the spacebar
    nnoremap <space> za
    ```
    * 之後就可用空白鍵收折程式碼

    * 如果想要看到收折的程式碼的docstrings 可以在.vimrc加入這行
    ```
    let g:SimpylFold_docstring_preview=1
    ```


參考:  
vim 快捷鍵 : http://stackoverflow.com/a/5400978/1799408  


https://www.openvim.com/  
https://realpython.com/vim-and-python-a-match-made-in-heaven/

breakpoints : https://github.com/puremourning/vimspector