# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 自动给命令输入
```bash
echo "123456" | <cmd>
```

# 获取IP MAC
> [如何在 Linux 上检查网卡信息](https://zhuanlan.zhihu.com/p/137332390)

```bash
qe@ubuntu:~$ sudo ./mac.sh 
ens33:
Permanent address: 00:0c:29:9e:f5:ce

ubuntu
-------------
ens33:
192.168.91.150/24
00:0c:29:9e:f5:ce
	Speed: 1000Mb/s
qe@ubuntu:~$ 
qe@ubuntu:~$ 
qe@ubuntu:~$ cat mac.sh 
#!/bin/sh
ip a |awk '/state UP/{print $2}' | sed 's/://' | while read output;
do
  echo $output:
  ethtool -P $output
done

echo ""

hostname
echo "-------------"
for iname in $(ip a |awk '/state UP/{print $2}')
do
  echo "$iname"
  ip a | grep -A2 $iname | awk '/inet/{print $2}'
  ip a | grep -A2 $iname | awk '/link/{print $2}'
  ethtool $iname |grep "Speed:"
done
```

# sh -c
> [sh -c的必要性](https://blog.csdn.net/bobchill/article/details/84647575)

可以让bash将一个字串作为完整的命令来执行，这样就可以将sudo的影响范围扩展到整条命令。
```bash
$ sudo sh -c 'echo "dtoverlay=mcp2517fd" >> /boot/config.txt'
```

# find
查找文件，
```bash
$ find /usr/src/linux-headers-5.3.0-62 -name time_types.h
```

# 查看文件夹大小
使用du命令，
```bash
$ du -h --max-depth=1 *
```

# 查找域名对应的ip
使用`nslookup`，
```bash
$ nslookup www.baidu.com
Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
www.baidu.com	canonical name = www.a.shifen.com.
Name:	www.a.shifen.com
Address: 183.232.231.172
Name:	www.a.shifen.com
Address: 183.232.231.174
```

# patch
> [四种常用的打Patch方法 ](http://cxy7.com/articles/2018/06/11/1528724449109.html)
> [打patch的方法](https://blog.csdn.net/miss_lazygoat/article/details/50056407)

```bash
$ patch [-R] {-p(n)} [--dry-run] < patch_file_name
```
- p：为path的缩写。
- n：指将patch文件中的path第n条"/"及其左边部分取消，生成的补丁中, 路径信息包含了你的Linux源码根目录的名称, 但其他人的源码根目录可能是其它名字, 所以, 打补丁时, 要进入你的Linux源码根目录, 并且告诉patch工具, 请忽略补丁中的路径的第一级目录(参数-p1).
- -R：卸载patch包。
- --dry-run：尝试patch软件，并不真正修改软件。

# 正则表达式
> [正则表达式 - 教程](https://www.runoob.com/regexp/regexp-tutorial.html)

# 加减乘除
> [shell 加法计算](https://www.cnblogs.com/sea-stream/p/9883491.html)

```bash
val=$(expr 10 + 20)
```

# 操作16进制
> [linux echo 写二进制文件](https://blog.csdn.net/whatday/article/details/96875933)
> [linux shell 下各种进制数据转换。](https://blog.csdn.net/hejinjing_tom_com/article/details/12650417)

`$ echo -e -n "\x11\x22" > test`，`-e`表示使能反斜杠转义，这样遇到`\`就会转义为二进制，`-n`不添加行尾换行标识，因为echo默认会在末尾添加`0x0A`。

# 文件比较
- cmp 以字节为单位
- diff 以行为单位
- patch 以diff为基础

# rar文件
```bash
$ sudo apt-get install rar unrar
$ rar x a.rar
```
# 获取IP地址
> [Shell脚本中获取本机ip地址的3个方法](https://blog.csdn.net/yangchunlu0101/article/details/78067282)
```bash
IP=$(ifconfig eth0 | grep "inet addr" | awk '{ print $2}' | awk -F: '{print $2}')
```

# pushd popd
完成一个`cd dir | do something | cd old_dir`功能，
```bash
pushd $(OSDRV_DIR)/pub/$(PUB_IMAGE);./mkboot.sh $(UBOOT_REG_BIN) $(UBOOT);popd
```

# 输出重定向
标准输入的文件描述符是0，标准输出的文件描述符是1，标准错误的文件描述符是2，在`>`符号前明确指定标准错误的文件描述符，即使用`2>`对标准错误进行重定向,
- `>`：打开file文件时会先清空文件，然后添加输出信息。
- `>>`：打开file文件时不清空文件，直接在file文件结尾处添加输出信息。

使用`&>`符号，命令生成的输出都发送到同一位置，`2>&1`表示标准错误合并到标准输出，`./test > a.txt 2>&1`，表示标准错误合并到标准输出后再写入文件，先合并是为了数据有序。

# ((: i<: syntax error: operand expected (error token is "<")
代码中忘记对channelPairs赋值，导致错误，
```shell
for ((i=0; i<$channelPairs; i++))
do
  rm -f data/output_datafile${i}_4K.bin
  echo "Info: DMA setup to read from c2h channel $i. Waiting on write data to channel $i."
  ./dma_from_device -d /dev/xdma0_c2h_${i} -f data/output_datafile${i}_4K.bin -s $transferSize -c $transferCount &
done
```

# Bad substitution
```shell
sudo dpkg-reconfigure dash
```
然后弹出选择框,选择no,就可以把默认shell改成bash。

# shift: can't shift that many
注释掉一个shift
```shell
	-b)
		fdk_work="build_workspace"
		fdk_build=$2
		# fdk_resv_arg=$3
		shift
		shift
		# shift
		;;
```

# 终端出现>怎么退出
Ctrl+d

# /bin/sh^M: bad interpreter
```shell
vim test.sh
:set ff?
如果出现fileforma＝dos那么就基本可以确定是这个问题了。
:set fileformat=unix
:wq
```

# chown chgrp
使用chown命令来改变文件所有者。chown命令是change owner（改变拥有者）的缩写。需要要注意的是，用户必须是已经存在系统中的，也就是只能改变为在 /etc/passwd这个文件中有记录的用户名称才可以。
chown命令的用途很多，还可以顺便直接修改用户组的名称。此外，如果要连目录下的所有子目录或文件同时更改文件拥有者的话，直接加上 -R的参数即可。
基本语法：
chown [-R] 账号名称 文件或目录
chown [-R] 账号名称:用户组名称 文件或目录
使用chgrp命令来改变文件所属用户组，该命令就是change group（改变用户组）的缩写。需要注意的是要改变成为的用户组名称，必须在 /etc/group里存在，否则就会显示错误。
基本语法：
chgrp [-R] 用户组名称 dirname/filename ...

# sed
> [linux中sed引用shell变量](https://www.jianshu.com/p/78522b72bdd8)
利用sed变量替换，下面的脚本sed后面必须用双引号括起来，单引号不行，
```shell
cur_date_time=$(date +"%Y-%m-%d %H:%M:%S")
echo "current time: ${cur_date_time}"
#cur_date_time=$(echo $cur_date_time)
sed -i "s/2017-08-08 00:00:00/${cur_date_time}/g" rootfs/etc/init.d/mwmstart.sh
```
同样的语句，在arm的嵌入式平台用单引号却可以，
```shell
for mtdline in `ls /sys/class/mtd | grep 'mtd[0-9]*[0-9]$'`
do
	mtdlabel=`cat /sys/class/mtd/${mtdline}/name`
	if [ "${mtdlabel}" = "bootenv" ] && [ "${mtdline}" != "mtd1" ]; then
		sed -i 's/mtd1/'$mtdline'/g' /etc/fw_env.config
	elif [ "${mtdlabel}" = "bootenvredund" ] && [ "${mtdline}" != "mtd2" ]; then
		sed -i 's/mtd2/'$mtdline'/g' /etc/fw_env.config
	fi
done
```
行首行尾批量添加
```shell
1、^代表行首
2、$代表行尾
3、所有行首增加sed -i 's/^/ABC/' a.txt
4、所有行尾添加sed -i 's/$/XYZ/' a.txt
5、删除首行sed -i '1d' d.txt
6、删除末行sed -i '$d' d.txt
7、第5行添加sed -i '5 r 5.txt' a.txt
8、删除空行sed -i '/^$/d' a.txt
9、剔除空格sed -i 's/[ ]*//g' ~/vip1.txt
10、删除回车符sed -i 's/^M//g' a.txt
11、从fromstart这行下面添加内容sed -i '/fromstart/r 4.txt' 5.txt
12、第一列排序存文件awk '{print $1}' vip1.txt |sort -n > vip2.txt
在646到650的行尾添加内容
sed -i '646,650s/$/" >> $its_path/g' fdk.sh
在619到689的行首添加内容
sed -i '619,689s/^/echo "/g' fdk.sh
```
sed在指定行添加文本，换行用\n，制表用\t。这里i为在上一行添加，下一行添加用a。
```shell
sed -i '33i\\treserved-memory' $dt_path/system-top.dts
```
sed通过正则表达式在指定行添加，在行首匹配处添加字符串。这里i为在匹配处上一行添加，下一行添加用a。
```shell
sed -i '/^};/i\\treserved-memory' $dt_path/system-top.dts
```
sed在第n次匹配后添加，
> [ 文本编辑的一点心得--sed篇 ](http://bbs.chinaunix.net/forum.php?mod=viewthread&tid=1762006)

## sed: -e expression #1, char 126: unknown option to `s'
这是sed替换字符串中包含变量，而这个变量是一个路径，包含了`/`，和sed的分隔符一样了，将分隔符换成`#`，
```shell
sed -i "1225{s#.*#\.\/configure $CONFIGURE_FLAGS --disable-shared#g}" $cur_path/objs/Makefile
```

# basename
> 参考[shell脚本学习笔记（四） —— expr、basename、shift](https://blog.csdn.net/Robot__Man/article/details/53102421)

basename返回一个字符串参数的基本文件名称。用法： 
basename [string] [suffix] 
basename 命令读取 String 参数，删除以 /(斜杠) 结尾的前缀以及任何指定的 Suffix 参数，并将剩余的基本文件名称写至标准输出。

# 字符串

## 字符串比较
> [Shell数值、字符串比较](https://www.cnblogs.com/happyhotty/articles/2065412.html)

`=`等于，如`if [ "$a" = "$b" ] `
`==`等于，如`if [ "$a" == "$b" ]`，与=等价 
注意`==`的功能在`[[]]`和`[]`中的行为是不同的，如下，
1 `[[ $a == z* ]]`    # 如果$a以"z"开头(模式匹配)那么将为true 
2 `[[ $a == "z*" ]]` # 如果$a等于z*(字符匹配),那么结果为true 
3 `[ $a == z* ] `     # File globbing 和word splitting将会发生 
4 `[ "$a" == "z*" ]` # 如果`$a`等于`z*`(字符匹配),那么结果为true 
一点解释,关于File globbing是一种关于文件的速记法,比如`"*.c"`就是,再如`~`也是，但是file globbing并不是严格的正则表达式，虽然绝大多数情况下结构比较像。
`!=`不等于,如:`if [ "$a" != "$b" ] `这个操作符将在`[[]]`结构中使用模式匹配. 
大于，在ASCII字母顺序下，如`if [[ "$a" > "$b" ]] `，`if [ "$a" \> "$b" ] `，注意在`[]`结构中`">"`需要被转义。 
`-z`字符串为null，就是长度为0。
`-n`字符串不为null。


## 字符串截取
> [shell脚本字符串截取的8种方法](https://www.cnblogs.com/zwgblog/p/6031256.html)

`#`截取功能，删除最左匹配项的左边字符，保留右边字符。`##`表示删除最右匹配项的左边字符。
`%`截取功能，删除最右匹配项的右边字符，保留左边字符。`%%`表示删除最左匹配项的右边字符。
```
echo ${var#*/}
echo ${var##*a}
echo ${var%c*}
echo ${var%%b*}
```

# grep
linux用grep从文件中查找匹配字符串，注意第二种里面查找的字符串里有一个横线，这就fuck了，一直报错，不能带小横线啊，
```shell
grep -rn "elf32ppclinux" *
grep -rn "-elf32ppclinux" *
grep: invalid max count
```

# tar
> [tar遇到错误不停止](https://zhidao.baidu.com/question/622336900884939492.html?qbl=relate_question_5&word=tar%D3%F6%B5%BD%B4%ED%CE%F3%B2%BB%CD%CB%B3%F6)

linux下tar命令解压到指定的目录 ：
```shell
tar zxvf /bbs.tar.zip -C /zzz/bbs 
```
和cp命令有点不同，cp命令如果不存在这个目录就会自动创建这个目录！
将当前目录下的zzz文件打包到根目录下并命名为zzz.tar.gz
```shell
tar zcvf /zzz.tar.gz ./zzz
```
遇到错误不停止，
```shell
#错误信息重定向
tar zcf file.tgz file 2>/dev/null
#不管啥信息都不要
tar zcf file.tgz file 2>&1 > /dev/null

zc@ubuntu:~/project$ tar -zcvf project-fdk.tar.gz fdk 2>&1 > /dev/null
tar: fdk/mwm197/t2080_rootfs/etc/gshadow: Cannot open: Permission denied
tar: Exiting with failure status due to previous errors
zc@ubuntu:~/project$ ls
fdk  petalinux  project-fdk.tar.gz
```
虽然有错误，但是不影响使用。

# 获取当前登录终端名
tty命令查看当前登录终端，who命令查看所有登录终端

# cp
a1为原文件夹，a2为目标文件夹
```shell
cp -Rf a1 a2 #cp覆盖目标文件
awk 'BEGIN { cmd="cp -ri a1/* a2/"; print "n" |cmd; }' #cp不覆盖目标文件
```

# find
批量复制更改文件名
```shell
file_list=$(find $cur_path/t2080-$FSL_PPC64E6500_TOOLCHAIN_VER/bin -type f -exec basename {} \;)
for bin_file in $file_list
do
	new_bin_file=${bin_file##*-}
	cp -vf $cur_path/t2080-$FSL_PPC64E6500_TOOLCHAIN_VER/bin/$bin_file $package_path/app/bin/$new_bin_file
done
```
只搜索当前文件夹，不递归搜索，
```bash
$ find . -maxdepth 1 -regex "./\w.*"
```

