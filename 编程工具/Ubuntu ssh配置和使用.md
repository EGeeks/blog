# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [Ubuntu16.04 搭建ssh](https://blog.csdn.net/swiftfake/article/details/79861320)
> [Ubuntu配置SSH服务器](https://www.cnblogs.com/supernalsnow/p/5567206.html)
> [SSH Client连接Ubuntu Server失败解法](https://segmentfault.com/a/1190000005709819)
> [ssh安全优化](https://blog.csdn.net/bwlab/article/details/51249254)

# 方法
使用命令`sudo apt install openssh-server`只安装服务器时，报错`Package ssh-server is a virtual package provided by...`，提示让你确认选择安装哪一个包，使用命令`sudo apt install ssh`就成功安装了，且同时把服务器和客户端都安装了，记住版本号`1:7.2p2-4ubuntu2.8`，下次可以仅安装服务端。
```bash
qe@ubuntu:~$ sudo apt install ssh
Reading package lists... Done
Building dependency tree       
Reading state information... Done
ssh is already the newest version (1:7.2p2-4ubuntu2.8).
0 upgraded, 0 newly installed, 0 to remove and 320 not upgraded.
qe@ubuntu:~$ 
qe@ubuntu:~$ 
qe@ubuntu:~$ sudo apt install openssh-server
Reading package lists... Done
Building dependency tree       
Reading state information... Done
openssh-server is already the newest version (1:7.2p2-4ubuntu2.8).
openssh-server set to manually installed.
0 upgraded, 0 newly installed, 0 to remove and 320 not upgraded.
```
安装完启动，
```bash
$ sudo service ssh start  #启动ssh
$ sudo service ssh stop   #关闭ssh
$ sudo service ssh restart  #重启ssh
```
ssh配置文件`/etc/ssh/sshd_config`，允许root登录，把`PermitRootLogin prohibit-password`换成`PermitRootLogin yes`
```bash
qe@ubuntu:~$ cat /etc/ssh/sshd_config
# Package generated configuration file
# See the sshd_config(5) manpage for details

# What ports, IPs and protocols we listen for
Port 22
# Use these options to restrict which interfaces/protocols sshd will bind to
#ListenAddress ::
#ListenAddress 0.0.0.0
Protocol 2
# HostKeys for protocol version 2
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_dsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key
#Privilege Separation is turned on for security
UsePrivilegeSeparation yes

# Lifetime and size of ephemeral version 1 server key
KeyRegenerationInterval 3600
ServerKeyBits 1024

# Logging
SyslogFacility AUTH
LogLevel INFO

# Authentication:
LoginGraceTime 120
PermitRootLogin prohibit-password
StrictModes yes

RSAAuthentication yes
PubkeyAuthentication yes
#AuthorizedKeysFile	%h/.ssh/authorized_keys

# Don't read the user's ~/.rhosts and ~/.shosts files
IgnoreRhosts yes
# For this to work you will also need host keys in /etc/ssh_known_hosts
RhostsRSAAuthentication no
# similar for protocol version 2
HostbasedAuthentication no
# Uncomment if you don't trust ~/.ssh/known_hosts for RhostsRSAAuthentication
#IgnoreUserKnownHosts yes

# To enable empty passwords, change to yes (NOT RECOMMENDED)
PermitEmptyPasswords no

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
ChallengeResponseAuthentication no

# Change to no to disable tunnelled clear text passwords
#PasswordAuthentication yes

# Kerberos options
#KerberosAuthentication no
#KerberosGetAFSToken no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes

# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes

X11Forwarding yes
X11DisplayOffset 10
PrintMotd no
PrintLastLog yes
TCPKeepAlive yes
#UseLogin no

#MaxStartups 10:30:60
#Banner /etc/issue.net

# Allow client to pass locale environment variables
AcceptEnv LANG LC_*

Subsystem sftp /usr/lib/openssh/sftp-server

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the ChallengeResponseAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via ChallengeResponseAuthentication may bypass
# the setting of "PermitRootLogin without-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and ChallengeResponseAuthentication to 'no'.
UsePAM yes
```
在最后方添加Ciphers配置，兼容所有ssh客户端，避免`Algorithm negotiation failed`，比如我的老版本SecureCRT就无法ssh登录板卡，
```bash
KexAlgorithms curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256,diffie-hellman-group14-sha1,diffie-hellman-group-exchange-sha1,diffie-hellman-group1-sha1
```
```bash
Ciphers aes128-cbc,aes192-cbc,aes256-cbc,aes128-ctr,aes192-ctr,aes256-ctr,3des-cbc,arcfour128,arcfour256,arcfour,blowfish-cbc,cast128-cbc
MACs hmac-md5,hmac-sha1,umac-64@openssh.com,hmac-ripemd160,hmac-sha1-96,hmac-md5-96
KexAlgorithms diffie-hellman-group1-sha1,diffie-hellman-group14-sha1,diffie-hellman-group-exchange-sha1,diffie-hellman-group-exchange-sha256,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group1-sha1,curve25519-sha256@libssh.org
```
Ciphers
指定SSH-2允许使用的加密算法。多个算法之间使用逗号分隔。可以使用的算法如下：
"aes128-cbc", "aes192-cbc", "aes256-cbc", "aes128-ctr", "aes192-ctr", "aes256-ctr",
"3des-cbc", "arcfour128", "arcfour256", "arcfour", "blowfish-cbc", "cast128-cbc"
默认值是可以使用上述所有算法。

Ciphers 默认使用这些
aes128-ctr,aes192-ctr,aes256-ctr,arcfour256,arcfour128,aes128-cbc,3des-cbc
漏洞提示arcfour,arcfour128,arcfour256都是不安全的,那就删掉
变成这样aes128-ctr,aes192-ctr,aes256-ctr,aes128-cbc,3des-cbc


# SFTP
远程命令`cd`，本地命令`lcd`，
```bash
$ ssh root@***.20.77.109
$ sftp root@***.20.77.109
```

