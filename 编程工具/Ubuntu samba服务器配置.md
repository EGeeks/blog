# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [Linux samba的配置和使用](https://blog.csdn.net/weixin_40806910/article/details/81917077)
> [解决 samba不允许一个用户使用一个以上用户名与一个服务器或共享资源的多重连接](https://www.cnblogs.com/senior-engineer/p/4528378.html)

# 安装
```shell
qe@ubuntu:/var/www/html$ sudo apt-get install samba
```

# 配置文件
/etc/samba/smb.conf
```shell
[share]
comment = share folder
browseable = yes
path = /home/zc
create mask = 0777
directory mask = 0777
#security = share
security = user
#valid users = share
#force user = nobody
#force group = nogroup
#public = yes
read only = no
writable = yes
available = yes
```

# 添加用户
```shell
zc@ubuntu:~/project/fdk/mwm178$ sudo smbpasswd -a zc
New SMB password:
Retype new SMB password:
Added user zc.
```

# 重启
```shell
zc@ubuntu:~/project/fdk/mwm178$  sudo /etc/init.d/samba restart
[ ok ] Restarting nmbd (via systemctl): nmbd.service.
[ ok ] Restarting smbd (via systemctl): smbd.service.
[ ok ] Restarting samba-ad-dc (via systemctl): samba-ad-dc.service.
```

# windows
映射网络驱动器，不是添加网络位置，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190221202804578.PNG)
完成时，弹出用户登录的对话框，如果没有出现，则断开网络驱动器，勾选下面的框，重新使用smb用户登录，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190221202857389.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

# 网络位置不存在
直接重新添加网络位置，覆盖Z盘，重启之后就可以了。

# Failed to restart samba.service: Unit samba.service is masked
使用`sudo service samba restart`或`sudo systemctl restart samba`更换命令为`sudo systemctl restart smbd.service`即可。

# 不允许一个用户使用一个以上用户名与一个服务器或共享资源的多重连接
事实上这个不是samba的限制。是Windows的限制。始终要用public＝yes的话，上面的方法都不能有效解决，因为：
在打开存在public＝yes的samba服务器时，如果首先点击了有public＝yes的共享资源的时候，widows会用默认的用户名去连接服务器，一般就是windows的登录名（可以在服务器端查看到的），这时候，再去点击没有public＝yes的共享资源，由于使用了user级别，服务器就会要求验证，这时，之前的默认登录已经存在，就出现了楼主的故障了。即使注销连接后如果没有采用正确的顺序访问共享资源，还是会陷入这个泥潭中。因此，最好办法就是不用public＝yes，给公共帐号建立一个共用的账户并公示出来。这样处理，其实权限更清晰一些。使用以下命令解决，
```bash
net use * /del /y
```

