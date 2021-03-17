# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [GitHub Help](https://help.github.com/)
> [Git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)
> [使用Gitee](https://www.liaoxuefeng.com/wiki/896043488029600/1163625339727712)
> [Git冲突：commit your changes or stash them before you can merge.](https://blog.csdn.net/lincyang/article/details/21519333)
> [Git多账户切换配置](https://blog.csdn.net/Nathan1987_/article/details/98546095)
> [git 配置多个账户](https://www.jianshu.com/p/8895d239b8cc)
> [玩转GIT系列之【git submodule update出错提示子模组未对路径注册】](https://blog.csdn.net/LEON1741/article/details/90259836)
> [Git修改提交的仓库地址](https://blog.csdn.net/csdn_flyyoung/article/details/87905077)
> [git 撤销远程的上次提交（撤销远程服务器如GitHub gitlab上的提交）](https://blog.csdn.net/oLiZuoZuo12/article/details/108468552)

# Git
## 安装
百度下载Git Windows安装包，安装即可,安装完打开Git Bash，配置Git。
```shell
zc@DESKTOP-KVKC06A MINGW64 ~
$ git config --global user.name "q***sina"

zc@DESKTOP-KVKC06A MINGW64 ~
$ git config --global user.email "z**apple@me.com"
```
注意git config命令的--global参数，用了这个参数，表示你这台机器上所有的Git仓库都会使用这个配置，当然也可以对某个仓库指定不同的用户名和Email地址。

## 创建本地仓库
在Git Bash中进入代码工程目录，使用cd命令。执行git init创建版本库。
```shell
zc@DESKTOP-KVKC06A MINGW64 ~
$ cd /c/project/****/

zc@DESKTOP-KVKC06A MINGW64 /c/project/****
$ ls
****/  ****.sdf

zc@DESKTOP-KVKC06A MINGW64 /c/project/****
$ git init
Initialized empty Git repository in C:/project/****/.git/
```

## 提交到本地仓库
git status查看文件状态，git add添加文件，git commit提交到本地仓库。
```shell
zc@DESKTOP-KVKC06A MINGW64 /c/project/**** (master)
$ git add *.h *.c *.cpp *.rc *.inf *.md sources
warning: LF will be replaced by CRLF in README.md.
The file will have its original line endings in your working directory.

zc@DESKTOP-KVKC06A MINGW64 /c/project/**** (master)
$ git status
On branch master
...
Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
...
Untracked files:
  (use "git add <file>..." to include in what will be committed)
...

zc@DESKTOP-KVKC06A MINGW64 /c/project/**** (master)
$ git commit -m "Initial commit"
```

## 撤销操作
参考[GitHub撤销修改](http://lib.csdn.net/article/git/12522)，`git reset –soft/--hard head_id`，使用`git reset`，`–soft`（两个`-`）表示只是改变了HEAD的指向，本地代码不会变化，`–hard`直接回改变本地源码，一般直接使用hard。 
```shell
aj@aj-PC MINGW64 /f/xilinxlinux/shall (master)
$ git reflog
785d7a7 HEAD@{0}: reset: moving to 785d7a7b66ff6adcb5a2cf0cba8d5df8ca6ee10e
785d7a7 HEAD@{1}: commit: add cmem
f73755d HEAD@{2}: commit: add cmem
869bf6c HEAD@{3}: commit: add build-image.sh add cmem
1a67ab0 HEAD@{4}: commit: add hsi dt
7e773fe HEAD@{5}: commit: mmc ext4 rootfs
31b5ef9 HEAD@{6}: commit: add mwmstart.h
a81f4b3 HEAD@{7}: pull: Fast-forward
078f961 HEAD@{8}: reset: moving to HEAD
078f961 HEAD@{9}: pull: Fast-forward
982a90a HEAD@{10}: commit: update build-image.sh
46b4433 HEAD@{11}: commit (merge): nothing
391f18a HEAD@{12}: commit: update build-image.sh
343eabb HEAD@{13}: clone: from git@github.com:zhuzhu2009/shall.git

aj@aj-PC MINGW64 /f/xilinxlinux/shall (master)
$ git reset -soft HEAD-1
error: did you mean `--soft` (with two dashes ?)

aj@aj-PC MINGW64 /f/xilinxlinux/shall (master)
$ git reset --soft HEAD-1
fatal: ambiguous argument 'HEAD-1': unknown revision or path not in the working tree.
Use '--' to separate paths from revisions, like this:
'git <command> [<revision>...] -- [<file>...]'

aj@aj-PC MINGW64 /f/xilinxlinux/shall (master)
$ git reset --soft HEAD~1
```

# GitHub
## 新建GitHub仓库
我这里用的GitHub，GitHub首页自带英文教程，读一下指导，
![这里写图片描述](https://img-blog.csdn.net/20180614195154298?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
进入Guide界面，
![这里写图片描述](https://img-blog.csdn.net/20180615000959475?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
点击GitHub Help，
![这里写图片描述](https://img-blog.csdn.net/20180615001116373?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
有了这些帮助文档就好办了，点击Start a Project开始工程，或者右上角的加号。
![这里写图片描述](https://img-blog.csdn.net/20180616161025973?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
选择了README或license这样创建的仓库就不是空的了，有初始化文件。


## 配置
本地Git仓库和GitHub之间的传输是通过SSH加密的，所以需要设置SSH key，如果不设置则报错`Warning: Permanently added the RSA host key for IP address '13.229.188.59' to the list of known hosts.`，一路回车默认值。
```shell
zc@DESKTOP-KVKC06A MINGW64 /c/project/**** (master)
$ ssh-keygen -t rsa -C "z**apple@me.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/zc/.ssh/id_rsa):
Created directory '/c/Users/zc/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/zc/.ssh/id_rsa.
Your public key has been saved in /c/Users/zc/.ssh/id_rsa.pub.
...
```
在用户主目录里找到.ssh目录，里面id_rsa是私钥，id_rsa.pub是公钥，在GitHub打开Account settings，SSH Keys页面，点Add SSH Key，填上任意Title，在Key文本框里粘贴id_rsa.pub文件的内容。
![这里写图片描述](https://img-blog.csdn.net/20180616170158599?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
操作完之后。
![这里写图片描述](https://img-blog.csdn.net/20180616165851743?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 测试一下，
```bash
$ ssh -T git@github.com
Hi zhuzhu2009! You've successfully authenticated, but GitHub does not provide shell access.
```

## 同步git到本地
使用git clone
```shell
aj@aj-PC MINGW64 /f/xilinxlinux
$ git clone git@github.com:xxxx/xxxx.git
Cloning into 'xxxx'...
remote: Counting objects: 12, done.
remote: Compressing objects: 100% (9/9), done.
remote: Total 12 (delta 3), reused 12 (delta 3), pack-reused 0
Receiving objects: 100% (12/12), 4.39 KiB | 0 bytes/s, done.
Resolving deltas: 100% (3/3), done.
```
对于有submodule的，
```bash
$ git submodule update --init --recursive
```

## 上传到GitHub
git三连：添加文件，提交文件，同步到github，
 
 1. git status
 2. git add xxx
 3. git commit -m "xxx"
 4. git push -u origin master/git push origin master
 
```shell
aj@aj-PC MINGW64 /f/qt(master)
$ git status
...
aj@aj-PC MINGW64 /f/qt(master)
$ git add i* m*
...
aj@aj-PC MINGW64 /f/qt(master)
$ git commit -m "list widget modify"
...
aj@aj-PC MINGW64 /f/qt(master)
$ git push -u origin master
...
```

## 撤销操作
如果本地没有执行任何操作，使用，
```bash
$ git revert HEAD
$ git push origin master
```
$ 如果本地已经reset了，并重新commit了，使用，
```bash
git push origin master -f
```

## 删除GitHub仓库
点击进入Repositories（仓库），点击Settings，拖到网页最下方，点击删除，删除过程中会让你输入仓库名字确认删除。

## Raw配置
需手动配置raw IP，打开网站[ipaddress](https://www.ipaddress.com/)，搜索`raw.githubusercontent.com`，配置host文件，
```bash
# C:\Windows\System32\drivers\etc\hosts
# GitHub raw & imag
199.232.68.133 raw.githubusercontent.com
```
在Raw按钮上邮件链接另存为即可下载单个文件。

## 修改仓储地址
修改.git目录下的config文件。

# Gitee
## 配置
添加SSH，`C:\Users\**\.ssh\id_isa.pub`，
![150](https://img-blog.csdnimg.cn/20200607225338242.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)新建仓库，选择SSH模式，不是HTTPS，
```bash
Git 全局设置:

git config --global user.name "Mr.Bang***"
git config --global user.email "zhu***@me.com"
创建 git 仓库:

mkdir recorder
cd recorder
git init
touch README.md
git add README.md
git commit -m "first commit"
git remote add origin git@gitee.com:qingermaker/recorder.git
git push -u origin master
已有仓库?

cd existing_git_repo
git remote add origin git@gitee.com:qing***/re***.git
git push -u origin master
```
clone需要用户名和密码，
```bash
$ git clone https://gitee.com/qing***/re***.git
正克隆到 'recorder'...
Username for 'https://gitee.com': qing***
Password for 'https://qing***@gitee.com': 
remote: Enumerating objects: 408, done.
remote: Counting objects: 100% (408/408), done.
remote: Compressing objects: 100% (403/403), done.
remote: Total 408 (delta 184), reused 123 (delta 4), pack-reused 0
接收对象中: 100% (408/408), 7.62 MiB | 1.07 MiB/s, 完成.
处理 delta 中: 100% (184/184), 完成.
```

# 多账户设置
同时使用Github和Gitee的时候，有多个账号和邮箱，参考[Git多账户切换配置](https://blog.csdn.net/Nathan1987_/article/details/98546095)，

# 常见错误
## Please commit your changes or stash them before you merge.
> 参考博客[Git冲突](https://blog.csdn.net/lincyang/article/details/21519333)
> 参考博客[Git:代码冲突常见解决方法](https://blog.csdn.net/iefreer/article/details/7679631)

## warning: LF will be replaced by CRLF
备份博客时发现所有文字显示成一行了。
> [git提示“warning: LF will be replaced by CRLF”的解决办法](https://blog.csdn.net/u012757419/article/details/105614028/)

## git submodule update出错提示子模组未对路径注册或者请确认您有正确的访问权限并且仓库存在
> [github克隆项目中的子模块submodule时遇到的问题](https://blog.k-res.net/archives/1595.html)

`vi .gitmodules`，统一url格式是`https`还是`git`，然后执行`git submodule sync`再继续之前的操作。

## fatal: refusing to merge unrelated histories
由于新建GitHub仓库的时候，仓库不是空的，此时若是把本地仓库提交到GitHub就会有冲突，所以先把远程仓库同步到本地。中间出错参考[git无法pull仓库refusing to merge unrelated histories](https://blog.csdn.net/lindexi_gd/article/details/52554159)和[多人协作](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013760174128707b935b0be6fc4fc6ace66c4f15618f8d000)。出现错误：fatal: refusing to merge unrelated histories，执行git pull --allow-unrelated-histories，提交的时候出现vi编辑器，输入comment。
```shell
zc@DESKTOP-KVKC06A MINGW64 /c/project/**** (master)
$ git remote add origin git@github.com:****.git

zc@DESKTOP-KVKC06A MINGW64 /c/project/**** (master)
$ git pull
The authenticity of host 'github.com (13.250.177.223)' can't be established.
...
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'github.com,13.250.177.223' (RSA) to the list of known hosts.
warning: no common commits
remote: Counting objects: 3, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), done.
...
 * [new branch]      master     -> origin/master
There is no tracking information for the current branch.
Please specify which branch you want to merge with.
See git-pull(1) for details.

    git pull <remote> <branch>

If you wish to set tracking information for this branch you can do so with:

    git branch --set-upstream-to=origin/<branch> master


zc@DESKTOP-KVKC06A MINGW64 /c/project/**** (master)
$ git pull origin master
Warning: Permanently added the RSA host key for IP address '52.74.223.119' to the list of known hosts.
...
 * branch            master     -> FETCH_HEAD
fatal: refusing to merge unrelated histories

zc@DESKTOP-KVKC06A MINGW64 /c/project/**** (master)
$ git branch --set-upstream-to=origin/master master
Branch master set up to track remote branch master from origin.

zc@DESKTOP-KVKC06A MINGW64 /c/project/**** (master)
$ git pull
fatal: refusing to merge unrelated histories

zc@DESKTOP-KVKC06A MINGW64 /c/project/**** (master)
$ git pull --allow-unrelated-histories
Merge made by the 'recursive' strategy.
 LICENSE | 674 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 1 file changed, 674 insertions(+)
 create mode 100644 LICENSE

zc@DESKTOP-KVKC06A MINGW64 /c/project/**** (master)
$ git push -u origin master
Counting objects: 23, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (23/23), done.
Writing objects: 100% (23/23), 36.07 KiB | 0 bytes/s, done.
Total 23 (delta 2), reused 0 (delta 0)
remote: Resolving deltas: 100% (2/2), done.
...

zc@DESKTOP-KVKC06A MINGW64 /c/project/**** (master)
$ git status
...

zc@DESKTOP-KVKC06A MINGW64 /c/project/**** (master)
$ git add *.md
...

zc@DESKTOP-KVKC06A MINGW64 /c/project/**** (master)
$ git commit -m "change README.md"
...

zc@DESKTOP-KVKC06A MINGW64 /c/project/**** (master)
$ git push origin master
...
```

## commit your changes or stash them before you can merge
首先关闭占用的文件，强制更新到本地，之后可以把stash清空。
```bash
$ git stash
$ git pull
```

- git stash: 备份当前的工作区的内容，从最近的一次提交中读取相关内容，让工作区保证和上次提交的内容一致。同时，将当前的工作区内容保存到Git栈中。
- git stash pop: 从Git栈中读取最近一次保存的内容，恢复工作区的相关内容。由于可能存在多个Stash的内容，所以用栈来管理，pop会从最近的一个stash中读取内容并恢复。
- git stash list: 显示Git栈内的所有备份，可以利用这个列表来决定从那个地方恢复。
- git stash clear: 清空Git栈。此时使用gitg等图形化工具会发现，原来stash的哪些节点都消失了。

