---
layout: post
title: Vim-Command
tag: Vim
category: lession
---
vim commands

### 一些基本的Vim命令：

#### 进入输入模式命令：

    i--insert,              I--行首insert
    a--append,              A--行尾append
    o--next line open,      O--up line open
    s--替换，               x--字母剪切

    home， end
    page up， page down
    del
#### 删除和撤销操作
    dd, dw, d3。
    u, U, ctrl + R
#### 复制操作
    yy， yw， y$, y^
    #yy 复制几行
    #yw 复制几个单词
#### 查找和替换
    / word, ? word
    % s/old/new #用new替换整篇old
    #，# s/old/new #几行到几行替换
    s/old/new #当行首次出现替换
    s/old/new/g #当行全部替换

#### 其他

##### 根据不同文件的空格设置
    set smarttab
    set tabstop=4
    set shiftwidth=4
    set expandtab

    autocmd BufNewFile,BufRead *.html,*.htm,*.css,*.js set expandtab tabstop=2 shiftwidth=2


[vim经典配置参考](https://github.com/wklken/k-vim "vim")
[vim键盘映射参考](http://www.the5fire.com/vim-copy-to-clipboard.html)
[vim常见配置参考](http://www.the5fire.com/vim-copy-to-clipboard.html)

#### 我的vimrc配置:

	" 设定文件浏览器目录为当前目录
	set bsdir=buffer
	set autochdir
	" 设置编码
	set enc=utf-8
	" 设置文件编码
	set fenc=utf-8
	" 设置语法高亮度
	set syn=cpp
	"显示行号
	set nu!
	" 指定菜单语言
	set langmenu=zh_CN.UTF-8


	nmap <leader>w :w!<cr>
	nmap <leader>f :find<cr>
	map <C-c> y
	map <C-X> d
	map <C-A> <Esc>ggVG
	" 映射全选+复制 ctrl+a
	map <C-A> ggVGY
	map! <C-A> <Esc>ggVGY
	" 选中状态下 Ctrl+c 复制
	vmap <C-c> "+y
	"去空行
	nnoremap <F2> :g/^\s*$/d<CR>
	"比较文件
	nnoremap <C-F2> :vert diffsplit
	"新建标签
	map <M-F2> :tabnew<CR>
	"列出当前目录文件
	map <F3> :tabnew .<CR>
	"打开树状文件目录
	map <C-F3> \be
	"C，C++ 按F5编译运行
	map <F5> :call CompileRunGcc()<CR>
	func! CompileRunGcc()
	    exec "w"
	    if &filetype == 'c'
		exec "!g++ % -o %<"
		exec "! ./%<"
	    elseif &filetype == 'cpp'
		exec "!g++ % -o %<"
		exec "! ./%<"
	    elseif &filetype == 'java'
		exec "!javac %"
		exec "!java %<"
	    elseif &filetype == 'python'
		exec '!python %'
	    elseif &filetype == 'ruby'
		exec '!ruby %'
	    endif
	endfunc
	"C,C++的调试
	map <F8> :call Rungdb()<CR>
	func! Rungdb()
	    exec "w"
	    exec "!g++ % -g -o %<"
	    exec "!gdb ./%<"
	endfunc

	" 设置当文件被改动时自动载入
	set autoread
	" quickfix模式
	autocmd FileType c,cpp map <buffer> <leader><space> :w<cr>:make<cr>

	autocmd BufNewFile,BufRead *.html,*.htm,*.css,*.js set expandtab tabstop=2 shiftwidth=2

	" CTRL-X and SHIFT-Del are Cut
	vnoremap <C-X> "+x
	vnoremap <S-Del> "+x

	" CTRL-C and CTRL-Insert are Copy
	vnoremap <C-C> "+y
	vnoremap <C-Insert> "+y
	" For CTRL-V to work autoselect must be off.
	" On Unix we have two selections, autoselect can be used.
	if !has("unix")
	  set guioptions-=a
	endif

	" CTRL-V
	noremap <C-V>        "+gP
	inoremap<C-V>        "+gP
	cnoremap<C-V>        "+gP
	onoremap<C-V>        "+gP
	vmap<C-V>            "+gP

	cmap <C-V>        <C-R>+
	cmap <S-Insert>        <C-R>+
	exe 'inoremap <script> <C-V>' paste#paste_cmd['i']
	exe 'vnoremap <script> <C-V>' paste#paste_cmd['v']

	imap <S-Insert>        <C-V>
	vmap <S-Insert>        <C-V>



	" Use CTRL-S for saving, also in Insert mode
	noremap <C-S>        :update<CR>
	vnoremap <C-S>        <C-C>:update<CR>
	inoremap <C-S>        <C-O>:update<CR>

	" CTRL-Z is Undo; not in cmdline though
	noremap <C-Z> u
	inoremap <C-Z> <C-O>u

	" CTRL-A is Select all
	noremap <C-A> gggH<C-O>G
	inoremap <C-A> <C-O>gg<C-O>gH<C-O>G
	cnoremap <C-A> <C-C>gggH<C-O>G
	onoremap <C-A> <C-C>gggH<C-O>G
	snoremap <C-A> <C-C>gggH<C-O>G
	xnoremap <C-A> <C-C>ggVG

	" CTRL-Tab is Next window
	noremap <C-Tab> <C-W>w
	inoremap <C-Tab> <C-O><C-W>w
	cnoremap <C-Tab> <C-C><C-W>w
	onoremap <C-Tab> <C-C><C-W>w


	set smarttab
	set tabstop=4
	set shiftwidth=4
	set expandtab
	set autoindent
	set wrapscan
	set hlsearch " 打开高亮显示查找的文本
	set ignorecase " 忽略大小写

	autocmd BufNewFile,BufRead *.html,*.htm,*.css,*.js,*.rb,*.erb set expandtab tabstop=2 shiftwidth=2

	map <F7> :NERDTreeToggle<CR>
	imap <F7> <ESC>:NERDTreeToggle<CR>
