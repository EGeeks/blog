# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [ubuntu下的apt-get内网本地源的搭建](https://www.cnblogs.com/liangqihui/p/7150066.html)
> [Ubuntu的apt-get本地源搭配——根据需要自己添加软件作源](https://www.cnblogs.com/myitroad/p/4970416.html)
> [ubuntu16.04 自建源](https://www.cnblogs.com/dribs/p/8507525.html)
> [ubuntu局域网apt-get源搭建](https://blog.csdn.net/toronto2016/article/details/49147167)
> [ubuntu下安装程序的三种方法](https://www.cnblogs.com/xwdreamer/p/3623454.html)
> [Ubuntu 清除缓存 apt-get命令参数](https://www.cnblogs.com/jiu0821/p/10760296.html)
> [Ubuntu 镜像](https://developer.aliyun.com/mirror/ubuntu?spm=a2c6h.13651102.0.0.3e221b11s0bwoN)

# dpkg命令
查看已安装的软件包`dpkg -l`，
```bash
dpkg --info "软件包名" --列出软件包解包后的包名称.
dpkg -l --列出当前系统中所有的包.可以和参数less一起使用在分屏查看. (类似于rpm -qa)
dpkg -l |grep -i "软件包名" --查看系统中与"软件包名"相关联的包.
dpkg -s 查询已安装的包的详细信息.
dpkg -L 查询系统中已安装的软件包所安装的位置. (类似于rpm -ql)
dpkg -S 查询系统中某个文件属于哪个软件包. (类似于rpm -qf)
dpkg -I 查询deb包的详细信息,在一个软件包下载到本地之后看看用不用安装(看一下呗).
dpkg -i 手动安装软件包(这个命令并不能解决软件包之前的依赖性问题),如果在安装某一个软件包的时候遇到了软件依赖的问题,可以用apt-get -f install在解决信赖性这个问题.
dpkg -r 卸载软件包.不是完全的卸载,它的配置文件还存在.
dpkg -P 全部卸载(但是还是不能解决软件包的依赖性的问题)
dpkg -reconfigure 重新配置
```

# apt命令
常用的APT命令，
```bash
apt-cache search package 搜索包
apt-cache show package 获取包的相关信息，如说明、大小、版本等
sudo apt-get install package 安装包
sudo apt-get install package - - reinstall 重新安装包
sudo apt-get -f install 修复安装"-f = ——fix-missing"
sudo apt-get remove package 删除包
sudo apt-get remove package - - purge 删除包，包括删除配置文件等
sudo apt-get update 更新源
sudo apt-get upgrade 更新已安装的包
sudo apt-get dist-upgrade 升级系统
sudo apt-get dselect-upgrade 使用 dselect 升级
apt-cache depends package 了解使用依赖
apt-cache rdepends package 是查看该包被哪些包依赖
sudo apt-get build-dep package 安装相关的编译环境
apt-get source package 下载该包的源代码
sudo apt-get clean 清除/var/cache/apt下的包缓存
sudo apt-get autoclean 清理无用的包
sudo apt-get check 检查是否有损坏的依赖
```

# 手动安装deb包
安装和安装所有软件包，
```shell
# sudo dpkg -i package.deb
# sudo dpkg -i *.deb
```

# 配置阿里源
阿里巴巴官方介绍的十分清楚，参考文章[Ubuntu 镜像](https://developer.aliyun.com/mirror/ubuntu?spm=a2c6h.13651102.0.0.3e221b11s0bwoN)，ubuntu16.04，
```shell
jj@jj-ProLiant-DL388-Gen9:~$ sudo mv /etc/apt/sources.list /etc/apt/sources.list.bak
jj@jj-ProLiant-DL388-Gen9:~$ lsb_release -c
Codename:	xenial
jj@jj-ProLiant-DL388-Gen9:~$ sudo gedit /etc/apt/sources.list

# deb cdrom:[Ubuntu 16.04 LTS _Xenial Xerus_ - Release amd64 (20160420.1)]/ xenial main restricted
#deb-src http://archive.ubuntu.com/ubuntu xenial main restricted #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb http://mirrors.aliyun.com/ubuntu/ xenial multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse #Added by software-properties
#deb http://archive.canonical.com/ubuntu xenial partner
#deb-src http://archive.canonical.com/ubuntu xenial partner
deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial-security universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-security multiverse
	
jj@jj-ProLiant-DL388-Gen9:~$ sudo apt-get update
```
ubuntu18.04，
```shell
qe@qe-pc:~$ cat /etc/apt/sources.list
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted multiverse universe
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted multiverse universe
deb http://mirrors.aliyun.com/ubuntu/ bionic universe
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates universe
deb http://mirrors.aliyun.com/ubuntu/ bionic multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted multiverse universe
deb http://mirrors.aliyun.com/ubuntu/ bionic-security universe
deb http://mirrors.aliyun.com/ubuntu/ bionic-security multiverse
qe@qe-pc:~$ sudo apt-get update
```
ubuntu20.04，
```shell
qe@qe-pc:~$ cat /etc/apt/sources.list
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted multiverse universe
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted multiverse universe
deb http://mirrors.aliyun.com/ubuntu/ focal universe
deb http://mirrors.aliyun.com/ubuntu/ focal-updates universe
deb http://mirrors.aliyun.com/ubuntu/ focal multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted multiverse universe
deb http://mirrors.aliyun.com/ubuntu/ focal-security universe
deb http://mirrors.aliyun.com/ubuntu/ focal-security multiverse
qe@qe-pc:~$ sudo apt-get update
```

# 利用apt-get缓存建立本地源
安装dpkg-dev，apache2，查看本地源,
```shell
j2@j2-pc:~$ sudo apt-get install dpkg-dev apache2
j2@j2-pc:~$ ls /var/cache/apt/archives
apache2_2.4.18-2ubuntu3.9_amd64.deb        libaprutil1_1.5.4-1build1_amd64.deb              partial
apache2-bin_2.4.18-2ubuntu3.9_amd64.deb    libaprutil1-dbd-sqlite3_1.5.4-1build1_amd64.deb  tftpd-hpa_5.2+20150808-1ubuntu1.16.04.1_amd64.deb
apache2-data_2.4.18-2ubuntu3.9_all.deb     libaprutil1-ldap_1.5.4-1build1_amd64.deb         tftp-hpa_5.2+20150808-1ubuntu1.16.04.1_amd64.deb
apache2-utils_2.4.18-2ubuntu3.9_amd64.deb  libdpkg-perl_1.18.4ubuntu1.5_all.deb             xinetd_1%3a2.3.15-6_amd64.deb
dpkg-dev_1.18.4ubuntu1.5_all.deb           liblua5.1-0_5.1.5-8ubuntu1_amd64.deb
libapr1_1.5.2-3_amd64.deb                  lock
```
建立源，
```shell
j2@j2-pc:~$ sudo mkdir /var/www/html/soft
j2@j2-pc:~$ sudo mkdir /var/www/html/dists
j2@j2-pc:~$ sudo mkdir /var/www/html/dists/zc
j2@j2-pc:~$ sudo mkdir /var/www/html/dists/zc/main
j2@j2-pc:~$ sudo mkdir /var/www/html/dists/zc/main/binary-i386
j2@j2-pc:~$ sudo mkdir /var/www/html/dists/zc/main/binary-amd64

j2@j2-pc:~$ cd /var/www/html
j2@j2-pc:/var/www/html$ sudo chmod -Rf a+w dists
j2@j2-pc:/var/www/html$ sudo chmod -Rf a+w soft 

j2@j2-pc:/var/www/html$ cp /var/cache/apt/archives/*.deb /var/www/html/soft 
j2@j2-pc:/var/www/html$ awk 'BEGIN { cmd="cp -vri /var/cache/apt/archives/*.deb /var/www/html/soft"; print "n" |cmd; }'

j2@j2-pc:/var/www/html$ dpkg-scanpackages /var/www/html/soft/ /dev/null | gzip > /var/www/html/dists/zc/main/binary-i386/Packages.gz
dpkg-scanpackages: 警告: Packages in archive but missing from override file:
dpkg-scanpackages: 警告:   apache2 apache2-bin apache2-data apache2-utils dpkg-dev libapr1 libaprutil1 libaprutil1-dbd-sqlite3 libaprutil1-ldap libdpkg-perl liblua5.1-0 tftp-hpa tftpd-hpa xinetd
dpkg-scanpackages: info: Wrote 14 entries to output Packages file.
j2@j2-pc:/var/www/html$ dpkg-scanpackages /var/www/html/soft/ /dev/null | gzip > /var/www/html/dists/zc/main/binary-amd64/Packages.gz
dpkg-scanpackages: 警告: Packages in archive but missing from override file:
dpkg-scanpackages: 警告:   apache2 apache2-bin apache2-data apache2-utils dpkg-dev libapr1 libaprutil1 libaprutil1-dbd-sqlite3 libaprutil1-ldap libdpkg-perl liblua5.1-0 tftp-hpa tftpd-hpa xinetd
dpkg-scanpackages: info: Wrote 14 entries to output Packages file.

j2@j2-pc:~/develop$ cat update_deb.sh 
#!/bin/sh

awk 'BEGIN { cmd="cp -vri /var/cache/apt/archives/*.deb /var/www/html/soft/"; print "n" |cmd; }'
dpkg-scanpackages /var/www/html/soft/ /dev/null | gzip > /var/www/html/dists/zc/main/binary-i386/Packages.gz
dpkg-scanpackages /var/www/html/soft/ /dev/null | gzip > /var/www/html/dists/zc/main/binary-amd64/Packages.gz

dpkg-scanpackages packages/ | gzip > packages/Packages.gz
```
不用压缩，压缩了ubuntu16.04反而不识别，一定是这个目录/var/www/html，不用麻烦，
```shell
j2@j2-pc:/var/www/html$ sudo mkdir packages
j2@j2-pc:/var/www/html$ sudo mv soft/* packages/
j2@j2-pc:/var/www/html$ dpkg-scanpackages packages > packages/Packages
dpkg-scanpackages: info: Wrote 183 entries to output Packages file.
j2@j2-pc:~$ cat develop/update_deb.sh 
#!/bin/sh

cur_path=$(pwd)
awk 'BEGIN { cmd="cp -vri /var/cache/apt/archives/*.deb /var/www/html/packages"; print "n" |cmd; }'
cd /var/www/html
dpkg-scanpackages packages > packages/Packages
dpkg-scanpackages packages | gzip > packages/Packages.gz
cd $cur_path
```

# 客户端配置
编辑/etc/apt/sources.list，
```shell
deb http://192.168.6.81 packages/
deb http://127.0.0.1 packages/
```
```shell
jj@jj-ProLiant-DL388-Gen9:~$ sudo apt-get update
```
# E: Could not get lock /var/lib/dpkg/lock-frontend
```bash
$ sudo rm /var/lib/dpkg/lock-frontend
```
# E: Could not get lock /var/lib/dpkg/lock
```bash
$ sudo rm /var/lib/dpkg/lock
```
# E: Could not get lock /var/cache/apt/archives/lock
```bash
$ sudo rm /var/cache/apt/archives/lock
```

# 0% [正在等待报头]
上一次update没有完才用Ctrl+C结束，再次update出现这个错误，执行clean（`sudo rm -rf /var/cache/apt/archives/*`）
```bash
sudo apt-get clean
```

