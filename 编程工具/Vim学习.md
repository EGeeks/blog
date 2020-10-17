# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [An automatic configuration program for vim](https://github.com/chxuan/vimplus)
> [所需即所获：像 IDE 一样使用 vim](https://github.com/yangyangwithgnu/use_vim_as_ide)
> [手把手教你配置linux下C++开发工具——vim+ycm(YouCompleteMe)，支持基于语义的自动补全和第三方库补全（史上最简单、史上最透彻、史上最全的终极解决方案）](https://blog.csdn.net/lianshaohua/article/details/108225916)
> [VIM环境配置](https://blog.csdn.net/song1361478501/article/details/66968774)
> [VimAwesome](https://vimawesome.com/)
> [VIM使用(二) 浏览内核源代码](https://www.cnblogs.com/linux-sir/p/4675919.html)
> [VIM安装YCM插件](https://www.cnblogs.com/yangxinrui/p/10046639.html)
> [关于vim在插入模式中Backspace键无法删除的问题[转]](https://blog.csdn.net/u011475134/article/details/76216145)
> [vim打开多个文件、同时显示多个文件、在文件之间切换](https://www.cnblogs.com/sunsky303/p/10998654.html)
> [vim学习笔记-tags用法](https://blog.csdn.net/paopaohehe/article/details/106977758)
> [vim之tags](https://www.cnblogs.com/pangchol/p/3455662.html)
> [Vim插件之属性目录NERDTree](https://www.cnblogs.com/littlewrong/p/6535728.html)

# 效果
ubuntu18.04 LTS， Vim8.2，和Source Insight的用户体验还是没法比，熟悉中，工作三年，才学Vim，一直Si，现在学习Vim纯粹是为了在部门年轻同事面前装B，别无他用！没人在的时候我就切出Vim，有人在的时候立刻打开Si！！！不能因为正事耽误装B。
![2020-10-07 23-40-26](https://img-blog.csdnimg.cn/20201007234514722.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)

# 安装
安装`ctags`，`cscope`用于阅读内核源代码，
```bash
$ sudo apt install ctags cscope
```
我的Ubuntu18.04默认没有`.vim`和`.vimrc`，通过命令`vim --version`查看，版本为8.0，默认支持python，cscope，不支持ruby，lua，perl，打开语言设置，发现默认使用ibus不是fcitx，
```bash
$ vim --version
VIM - Vi IMproved 8.0 (2016 Sep 12, compiled Mar 18 2020 18:29:15)
包含补丁: 1-1453
修改者 pkg-vim-maintainers@lists.alioth.debian.org
编译者 pkg-vim-maintainers@lists.alioth.debian.org
巨型版本 无图形界面。  可使用(+)与不可使用(-)的功能:
+acl               +farsi             +mouse_sgr         -tag_any_white
+arabic            +file_in_path      -mouse_sysmouse    -tcl
+autocmd           +find_in_path      +mouse_urxvt       +termguicolors
-autoservername    +float             +mouse_xterm       +terminal
-balloon_eval      +folding           +multi_byte        +terminfo
+balloon_eval_term -footer            +multi_lang        +termresponse
-browse            +fork()            -mzscheme          +textobjects
++builtin_terms    +gettext           +netbeans_intg     +timers
+byte_offset       -hangul_input      +num64             +title
+channel           +iconv             +packages          -toolbar
+cindent           +insert_expand     +path_extra        +user_commands
-clientserver      +job               -perl              +vertsplit
-clipboard         +jumplist          +persistent_undo   +virtualedit
+cmdline_compl     +keymap            +postscript        +visual
+cmdline_hist      +lambda            +printer           +visualextra
+cmdline_info      +langmap           +profile           +viminfo
+comments          +libcall           -python            +vreplace
+conceal           +linebreak         +python3           +wildignore
+cryptv            +lispindent        +quickfix          +wildmenu
+cscope            +listcmds          +reltime           +windows
+cursorbind        +localmap          +rightleft         +writebackup
+cursorshape       -lua               -ruby              -X11
+dialog_con        +menu              +scrollbind        -xfontset
+diff              +mksession         +signs             -xim
+digraphs          +modify_fname      +smartindent       -xpm
-dnd               +mouse             +startuptime       -xsmp
-ebcdic            -mouseshape        +statusline        -xterm_clipboard
+emacs_tags        +mouse_dec         -sun_workshop      -xterm_save
+eval              +mouse_gpm         +syntax            
+ex_extra          -mouse_jsbterm     +tag_binary        
+extra_search      +mouse_netterm     +tag_old_static
```
安装vundle，或者手动下载解压，
```bash
$ git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```
编辑`.vimrc`，安装插件，删除插件，更新插件，
```bash
:PluginInstall
:PluginClean
:PluginUpdate
```
安装插件时发现，YCM要求Vim版本8.1.2269+，重新安装Vim，到github上下载最新的release版8.2.1787，
```bash
$ vim .vimrc 
YouCompleteMe unavailable: requires Vim 8.1.2269+.
socket file of fcitx not found, fcitx.vim not loaded.
请按 ENTER 或其它命令继续
$ sudo apt-get install -y libncurses5-dev libncurses5 libgnome2-dev libgnomeui-dev libgtk2.0-dev libatk1.0-dev libbonoboui2-dev libcairo2-dev libx11-dev libxpm-dev libxt-dev python-dev python3-dev ruby-dev lua5.1 lua5.1-dev
# 基于python2.7构建
$ ./configure --with-features=huge --enable-multibyte --enable-rubyinterp --enable-pythoninterp --with-python-config-dir=/usr/lib/python2.7/config-x86_64-linux-gnu --enable-perlinterp --enable-luainterp --enable-tclinterp --enable-gui=gtk2 --enable-cscope --prefix=/usr
# 基于python3.6构建
$ ./configure --with-features=huge --enable-multibyte --enable-rubyinterp --enable-python3interp --with-python3-config-dir=/usr/lib/python3.6/config-3.6m-x86_64-linux-gnu --enable-perlinterp --enable-luainterp --enable-tclinterp --enable-gui=gtk2 --enable-cscope --prefix=/usr
$ make
$ sudo make install
```
安装YCM，会自动下载`libclang`来编译，
```bash
# clang
sudo apt install cmake
# js/ts
sudo apt install npm
$ ./install.py --clang-completer --ts-completer
Searching Python 3.6 libraries...
...
-- Found PythonLibs: /usr/lib/python3.6/config-3.6m-x86_64-linux-gnu/libpython3.6.so (found suitable version "3.6.9", minimum required is "3.5") 
-- Downloading libclang 10.0.0 from https://dl.bintray.com/ycm-core/libclang/libclang-10.0.0-x86_64-unknown-linux-gnu.tar.bz2
...
```
配置输入法，打开语言设置，选择fcitx，
```bash
$ sudo apt-get install fcitx fcitx-pinyin
```
如果`solarized`主题报错，需安装[pathogen](https://github.com/tpope/vim-pathogen)，支持插件管理器，或者将其他主题放入`.vim/colors`即可，
```bash
$ vim
处理 /home/qe/.vimrc 时发生错误:
第  119 行:
E185: Cannot find color scheme 'solarized'
请按 ENTER 或其它命令继续
```

# 使用
## 工作目录
执行vim所在的目录即为工作目录，NERDTree显示当前目录

## NERDTree
 NERDTree快捷键，
- 通过h j k l移动光标定位
- 打开关闭文件或者目录，如果是文件的话，光标出现在打开的文件中
- go 效果同上，不过光标保持在文件目录里，类似预览文件内容的功能
- i和s可以水平分割或纵向分割窗口打开文件，前面加g类似go的功能
- t 在标签页中打开
- T 在后台标签页中打开
- p 到上层目录
- P 到根目录
- K 到同目录第一个节点
- J 到同目录最后一个节点
- m 显示文件系统菜单（添加、删除、移动操作）
- ? 帮助
- q 关闭


## 系统粘贴板

## 多文件编辑
实测快捷键无法工作，
```bash
map <C-Tab> :MBEbn<cr>
map <C-S-Tab> :MBEbp<cr>
```
采用，
```bash
Ctrl+6 下一个文件 
:bn 下一个文件 
:bp 上一个文件
:e 文档名        这是在进入vim后，不离开 vim 的情形下打开其他文档
:open file 可以再打开一个文件，并且此时vim里会显示出file文件的内容
:ls 显示缓存
:b num 切换文件，num为buffer list编号
```

## ctags
使用ctags生成标签，
```bash
$ ctags -R *
# vim
:set tags+=/workspace/tags
# .vimrc
" ctags警告关闭
let g:indexer_disableCtagsWarning=1
```
vim会自动在当前目录查找tags，找不到会一直往上一级目录搜索，
```bash
# .vimrc
set tags=./tags,tags;
# or 
set tags=tags;
set autochdir
```
或者使用indexer插件。使用`c+]`建立tag堆栈，使用`:tnext、:tprevious`切换标签，使用`c+t`返回，实测不适合内核这样的大规模工程，每次打开卡半天，
```bash
$ cat .indexer_files
[linux-aspeed]
/opt/openbmc/openbmc/build/workspace/sources/linux-aspeed
[u-boot-aspeed]
/opt/openbmc/openbmc/build/workspace/sources/u-boot-aspeed
```

## cscope
生成索引文件，
```bash
$ cscope -Rbkq
# vim
:cs add /path/to/cscope.out /your/work/dir
:cs find c func
:cw -- 打开quickfix窗口查看结果
```
索引文件有三个，cscope.out，cscope.in.out cscope.po.out 是使用q参数才会有，这两个文件可以加速cscope，定义的跳转通过ctags, 需要查找的时候才会用`cs find`，

## 快捷键
初学者的手忙脚乱：快捷键是直接按的，不是在命令框里输入的

## map
map是映射，
- imap只在insert模式下生效
- vmap只在visual模式下生效
- nmap只在normal模式下生效

noremap是不递归的映射，
- inoremap只在insert模式下生效
- vnoremap只在visual模式下生效
- nnoremap只在normal模式下生效

## YCM


# 问题
## YouCompleteMe unavailable: requires Vim compiled with Python (3.6.0+) support
需要基于python3来编译vim

## The ycmd server SHUT DOWN (restart with ':YcmRestartServer'). YCM core library not detected; you need to compile YCM before using it. Follow the instructions in the documentation.
需要到YCM目录执行`install.py`

## 插入模式下无法用Backspace删除
在`.vimrc`中添加` set backspace=indent,eol,start`

## vim终端中输出>4;2m
vim输出乱码，注释下面几行
```bash
" Plugin 'vim-scripts/indexer.tar.gz'
" Plugin 'vim-scripts/DfrankUtil'
" Plugin 'vim-scripts/vimprj'

" source $VIMRUNTIME/ftplugin/man.vim
" nmap <Leader>man :Man 3 <cword><CR>
```

## powerline输出>4;2m
加载powerline插件之后，界面最下方输出乱码，滚动文件，乱码消失，使用vim8.2后问题消失！

## 光标出现在
```bash
set virtualedit=block,onemore   " 允许光标出现在最后一个字符的后面
```

