---
layout: post
title: "Win7下Vim设置Python开发环境"
date: 2012-10-11 19:52
comments: true
categories: 
- Vim
- Python
---

最近用python写代码，条件有限，懒得用eclipse，遂用vim来做。网上搜了下vim支持python开发的方法，一步一步的来有点麻烦，想起以前有个脚本，稍加修改就OK了，在此与大家分享，功能上不只是支持python,常用的都支持，支持taglist以及minibuffer外加nerdtree（树形显示目录），应该说相当完善了，整个过程差不多10分钟搞定。

<!--more-->

>About:
>OS:Win 7
>2012年10月11日，第一版。

##下载Vim7
前往[Vim下载](http://www.vim.org/ "Vim下载")下载最新的Vim7.

##下载IDE脚本
前往[Vim-IDE](http://code.google.com/p/vimide/)下载vimide for windows，按照里面操作执行，只是简单的复制过程。

##更新Vim安装目录下的_vimrc文件
{% codeblock _vimrc - awesome.rb %}

"设置文件编码
set fileencodings=utf-8,cp936,ucs-bom,gbk 
set helplang=cn

"ColorScheme设置色彩主题
colorscheme desert

set shiftwidth=2
set tabstop=2
set expandtab
set nobackup
set noswapfile
set nowb
set backspace=start,indent,eol
set nu
set autoindent
set smartindent
set wrap

"设置鼠标
set mouse=a

set noerrorbells
set novisualbell

"语法高亮
syntax on

"设置自动补全
filetype plugin on
filetype indent on
set completeopt=longest,menu

"自动补全命令时候使用菜单式匹配列表  
set wildmenu
autocmd FileType ruby,eruby set omnifunc=rubycomplete#Complete
autocmd FileType python set omnifunc=pythoncomplete#Complete
autocmd FileType javascript set omnifunc=javascriptcomplete#CompleteJS
autocmd FileType html set omnifunc=htmlcomplete#CompleteTags
autocmd FileType css set omnifunc=csscomplete#CompleteCSS
autocmd FileType xml set omnifunc=xmlcomplete#CompleteTags
autocmd FileType java set omnifunc=javacomplete#Complet

"NERDTree可以树形显示目录,Taglist显示Tags列表
"Ctrl+w：打开文件浏览 Ctrl+t：打开Taglist
map <C-w> :NERDTree<cr>
map <C-t> :TlistToggle<cr>
vmap <C-c> "+y

let Tlist_Ctags_Cmd='ctags.exe'
let Tlist_Show_One_File = 1
let Tlist_Exit_OnlyWindow=1

"存放tags的目录，ctags -R后生成的tags所在目录
set tags=C:\Users\lusky\quantum

let g:miniBufExplMapWindowNavVim = 1 
let g:miniBufExplMapWindowNavArrows = 1 
let g:miniBufExplMapCTabSwitchBufs = 1 
let g:miniBufExplModSelTarget = 1

"PHP支持,不需要的话可以不要
inoremap <C-P> <ESC>:call PhpDocSingle()<CR>i 
nnoremap <C-P> :call PhpDocSingle()<CR> 
vnoremap <C-P> :call PhpDocRange()<CR> 

{% endcodeblock %}

##下载更适合Python的自动补全插件
下载pydiction，解压后有4个文件，拷贝python_pydiction.vim和complete-dict到ftplugin目录，修改_vimrc
{% codeblock _vimrc - awesome.rb %}

"Pydiction插件设置
let g:pydiction_location = 'D:\Program Files\Vim\vim73\ftplugin\complete-dict'

{% endcodeblock %}

##最后,给一张图吧:
{% img /images/2012-10/2012-10-11-setup-vim-for-python.jpg %}


也可以到我的github博客的source分支上下载。位于`blog/attachments/2012-10/_vimrc`下载.

