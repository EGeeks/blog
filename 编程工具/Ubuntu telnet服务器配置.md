# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [ubuntu 下安装配置 telnet server服务](https://blog.csdn.net/jirryzhang/article/details/70163012)
> [Ubuntu下安装建立Telnet 服务器](https://www.linuxidc.com/Linux/2010-03/25150.htm)

# 安装
```shell
j2@j2-pc:~$ sudo apt-get install telnetd xinetd
```

# 配置
编辑
```shell
sudo gedit /etc/xinetd.conf
```
添加
```shell
instances = 60
log_type = SYSLOG authpriv
log_on_success = HOST PID
log_on_failure = HOST
cps = 25 30
```
编辑
```shell
j2@j2-pc:~$ sudo gedit /etc/xinetd.d/telnet
```
添加
```shell
# default: on
# description: The telnet server serves telnet sessions; it uses
# unencrypted username/password pairs for authentication.

service telnet
{
disable = no
flags = REUSE
socket_type = stream
wait = no
user = root
server = /usr/sbin/in.telnetd
log_on_failure += USERID
}
```
重启
```shell
j2@j2-pc:~$ sudo /etc/init.d/xinetd restart
```

