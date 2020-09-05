# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [ubuntu 搭建TFTP服务器之最佳方案分析](http://blog.chinaunix.net/uid-30031530-id-5098813.html)

# 安装
可见默认的ubuntu 16.04这3个软件都没有安装。
```shell
j2@j2-pc:~$ sudo apt-get install tftp-hpa tftpd-hpa xinetd
正在读取软件包列表... 完成
正在分析软件包的依赖关系树       
正在读取状态信息... 完成       
建议安装：
  pxelinux
下列【新】软件包将被安装：
  tftp-hpa tftpd-hpa xinetd
升级了 0 个软件包，新安装了 3 个软件包，要卸载 0 个软件包，有 423 个软件包未被升级。
需要下载 164 kB 的归档。
解压缩后会消耗 469 kB 的额外空间。
获取:1 http://mirrors.aliyun.com/ubuntu xenial-updates/main amd64 tftp-hpa amd64 5.2+20150808-1ubuntu1.16.04.1 [18.0 kB]
获取:2 http://mirrors.aliyun.com/ubuntu xenial-updates/main amd64 tftpd-hpa amd64 5.2+20150808-1ubuntu1.16.04.1 [39.1 kB]
获取:3 http://mirrors.aliyun.com/ubuntu xenial/main amd64 xinetd amd64 1:2.3.15-6 [107 kB]
已下载 164 kB，耗时 0秒 (260 kB/s)
正在预设定软件包 ...
正在选中未选择的软件包 tftp-hpa。
(正在读取数据库 ... 系统当前共安装有 178705 个文件和目录。)
正准备解包 .../tftp-hpa_5.2+20150808-1ubuntu1.16.04.1_amd64.deb  ...
正在解包 tftp-hpa (5.2+20150808-1ubuntu1.16.04.1) ...
正在选中未选择的软件包 tftpd-hpa。
正准备解包 .../tftpd-hpa_5.2+20150808-1ubuntu1.16.04.1_amd64.deb  ...
正在解包 tftpd-hpa (5.2+20150808-1ubuntu1.16.04.1) ...
正在选中未选择的软件包 xinetd。
正准备解包 .../xinetd_1%3a2.3.15-6_amd64.deb  ...
正在解包 xinetd (1:2.3.15-6) ...
正在处理用于 man-db (2.7.5-1) 的触发器 ...
正在处理用于 ureadahead (0.100.0-19) 的触发器 ...
ureadahead will be reprofiled on next reboot
正在处理用于 systemd (229-4ubuntu21.1) 的触发器 ...
正在处理用于 doc-base (0.10.7) 的触发器 ...
Processing 33 changed doc-base files, 1 added doc-base file...
正在设置 tftp-hpa (5.2+20150808-1ubuntu1.16.04.1) ...
正在设置 tftpd-hpa (5.2+20150808-1ubuntu1.16.04.1) ...
正在设置 xinetd (1:2.3.15-6) ...
正在处理用于 systemd (229-4ubuntu21.1) 的触发器 ...
正在处理用于 ureadahead (0.100.0-19) 的触发器 ...
```

# 配置
如果只安装 tftpd-hpa，也可以开启tftp服务，但是每次开机都需要手动执行一次service  tftpd-hpa restart（注意是restart，不是start）。而安装xinetd后，每次开机，只要不执行service tftpd-hpa restart，都会以/etc/xinetd.d中的配置目录为准。一旦执行service tftpd-hpa restart，就会以/etc/default/tftpd-hpa配置为准。
编辑
```shell
j2@j2-pc:~$ sudo gedit /etc/default/tftpd-hpa
```
默认是
```shell
# /etc/default/tftpd-hpa

TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/var/lib/tftpboot"
TFTP_ADDRESS=":69"
TFTP_OPTIONS="--secure"
```
修改为，-c表示允许上传，可以不加
```shell
# /etc/default/tftpd-hpa

TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/tftpboot"
TFTP_ADDRESS="0.0.0.0:69"
TFTP_OPTIONS="-l -c -s"
```
编辑
```shell
j2@j2-pc:~$ sudo gedit /etc/xinetd.d/tftp
```
添加
```shell
service tftp
{
socket_type = dgram
wait = yes
disable = no
user = root
protocol = udp
server = /usr/sbin/in.tftpd
server_args = -s /tftpboot
#log_on_success += PID HOST DURATION
#log_on_failure += HOST
per_source = 11
cps = 100 2
flags = IPv4
}
```
重启服务
```shell
j2@j2-pc:~$ sudo service tftpd-hpa restart
j2@j2-pc:~$ sudo /etc/init.d/xinetd reload
j2@j2-pc:~$ sudo /etc/init.d/xinetd restart
```

# 上传失败
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181226171454747.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
```shell
j2@j2-pc:/tftpboot$ ls -l
总用量 8
-rw-rw-r-- 1 j2   j2   12 12月 26 14:58 a.txt
-rw-rw-rw- 1 tftp tftp 12 12月 26 15:01 b.txt

```

