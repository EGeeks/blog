# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考

> [ubuntu 下如何下载linux内核源码](https://blog.csdn.net/xq723310/article/details/49201331)
> [编译自己的Ubuntu内核](https://www.cnblogs.com/arnoldlu/p/6227843.html)
> [Ubuntu Linux内核版本升级或降级到指定版本（基于ubuntu 18.04示例）](https://blog.csdn.net/weixin_42915431/article/details/106614841)
> [Ubuntu 设置内核版本的GRUB默认启动](https://www.cnblogs.com/open-skill/p/8295234.html)

# 更新内核到指定版本
查看当前版本，
```bash
$ uname -a
Linux test-pc 5.4.0-54-generic #60~18.04.1-Ubuntu SMP Fri Nov 6 17:25:16 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
#更新到，其实是回退版本，
$ uname -a
Linux qe-pc 5.3.0-62-generic #56~18.04.1-Ubuntu SMP Wed Jun 24 16:17:03 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
```
查看已安装kernel，
```bash
$ dpkg -l linux*
```
搜索并安装想安装的版本，还推荐安装一些软件，暂时不安装，
```bash
$ apt search linux | grep 5.3.0-62
$ sudo apt install linux-headers-5.3.0-62-generic linux-image-5.3.0-62-generic linux-modules-extra-5.3.0-62-generic
[sudo] test 的密码： 
正在读取软件包列表... 完成
正在分析软件包的依赖关系树       
正在读取状态信息... 完成       
将会同时安装下列软件：
  linux-headers-5.3.0-62 linux-modules-5.3.0-62-generic
建议安装：
  fdutils linux-hwe-doc-5.3.0 | linux-hwe-source-5.3.0 linux-hwe-tools
下列【新】软件包将被安装：
  linux-headers-5.3.0-62 linux-headers-5.3.0-62-generic linux-image-5.3.0-62-generic linux-modules-5.3.0-62-generic
升级了 0 个软件包，新安装了 4 个软件包，要卸载 0 个软件包，有 345 个软件包未被升级。
```
查看内核启动顺序，默认启动最新版内核，
```bash
$ grep menuentry /boot/grub/grub.cfg
if [ x"${feature_menuentry_id}" = xy ]; then
  menuentry_id_option="--id"
  menuentry_id_option=""
export menuentry_id_option
menuentry 'Ubuntu' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-simple-58d2260d-9ce3-4893-b6bd-f66a8f883f69' {
submenu 'Ubuntu 高级选项' $menuentry_id_option 'gnulinux-advanced-58d2260d-9ce3-4893-b6bd-f66a8f883f69' {
	menuentry 'Ubuntu，Linux 5.4.0-54-generic' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.4.0-54-generic-advanced-58d2260d-9ce3-4893-b6bd-f66a8f883f69' {
	menuentry 'Ubuntu, with Linux 5.4.0-54-generic (recovery mode)' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.4.0-54-generic-recovery-58d2260d-9ce3-4893-b6bd-f66a8f883f69' {
	menuentry 'Ubuntu，Linux 5.3.0-62-generic' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.3.0-62-generic-advanced-58d2260d-9ce3-4893-b6bd-f66a8f883f69' {
	menuentry 'Ubuntu, with Linux 5.3.0-62-generic (recovery mode)' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.3.0-62-generic-recovery-58d2260d-9ce3-4893-b6bd-f66a8f883f69' {
	menuentry 'Ubuntu，Linux 5.3.0-28-generic' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.3.0-28-generic-advanced-58d2260d-9ce3-4893-b6bd-f66a8f883f69' {
	menuentry 'Ubuntu, with Linux 5.3.0-28-generic (recovery mode)' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.3.0-28-generic-recovery-58d2260d-9ce3-4893-b6bd-f66a8f883f69' {
menuentry 'System setup' $menuentry_id_option 'uefi-firmware' {
```
修改GRUB_DEFAULT，从
```bash
$ cat /etc/default/grub 
# If you change this file, run 'update-grub' afterwards to update
# /boot/grub/grub.cfg.
# For full documentation of the options in this file, see:
#   info -f grub -n 'Simple configuration'

GRUB_DEFAULT="1>2"
```
更新grub，
```bash
$ sudo update-grub
```
重启验证。

# 下载编译内核源代码
ubuntu 16.04.4为4.13版本，所以我们下载4.13的内核。
```shell
j2@j2-pc:~$ apt-cache search linux-source
linux-source - Linux kernel source with Ubuntu patches
linux-source-4.4.0 - Linux kernel source for version 4.4.0 with Ubuntu patches
linux-source-4.10.0 - Linux kernel source for version 4.10.0 with Ubuntu patches
linux-source-4.11.0 - Linux kernel source for version 4.11.0 with Ubuntu patches
linux-source-4.13.0 - Linux kernel source for version 4.13.0 with Ubuntu patches
linux-source-4.15.0 - Linux kernel source for version 4.15.0 with Ubuntu patches
linux-source-4.8.0 - Linux kernel source for version 4.8.0 with Ubuntu patches
j2@j2-pc:~$ uname -a
Linux j2-pc 4.13.0-36-generic #40~16.04.1-Ubuntu SMP Fri Feb 16 23:25:58 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
# 该方法下载内核不太匹配，版本为4.15
j2@j2-pc:~$ apt-get source linux-image-$(uname -r)
j2@j2-pc:~$ ls -l
总用量 308224
drwxrwxr-x 30 j2 j2      4096 1月   4 10:46 linux-hwe-4.15.0
-rw-r--r--  1 j2 j2  10432140 12月  8 17:43 linux-hwe_4.15.0-43.46~16.04.1.diff.gz
-rw-r--r--  1 j2 j2      6805 12月  8 17:43 linux-hwe_4.15.0-43.46~16.04.1.dsc
-rw-r--r--  1 j2 j2 157656459 6月  18  2018 linux-hwe_4.15.0.orig.tar.gz
j2@j2-pc:~$ sudo apt-get install linux-source-4.13.0
j2@j2-pc:~$ cd linux-source-4.13.0/
j2@j2-pc:~/linux-source-4.13.0$ make menuconfig
# save config
j2@j2-pc:~/linux-source-4.13.0$ make -j8

```

