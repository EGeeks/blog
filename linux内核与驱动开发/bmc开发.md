# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [OpenBMC](https://www.jianshu.com/p/12139db32e49)
> [服务器BMC技术调研](https://www.jianshu.com/p/e18de3800686)
> [openBMC（todo）](https://www.cnblogs.com/soul-stone/p/7327879.html)
> [BMC学习记录-smartfusion硬件架构](https://blog.csdn.net/zhaoxinfan/article/details/81089093)
> [IPMI和BMC 通信的过程](https://blog.csdn.net/tiantao2012/article/details/72864286)
> [OpenBmc开发2：构建开发环境](https://blog.csdn.net/qq_34160841/article/details/104841318)
> [OpenBmc开发7：创建新layer（其他方法）](https://blog.csdn.net/qq_34160841/article/details/106203086)
> [使用 IPMI 远程为服务器安装操作系统教程](https://blog.csdn.net/shida_csdn/article/details/86134733)
> [产品知识中心：SOL（Serial Over LAN）](http://server.it168.com/a2009/0930/750/000000750632.shtml)

# 使用
```bash
qe@ubuntu:~/program$ sudo apt-get install -y git build-essential libsdl1.2-dev texinfo gawk chrpath diffstat
qe@ubuntu:~/program$ git clone git@github.com:openbmc/openbmc.git
Cloning into 'openbmc'...
ssh: connect to host github.com port 22: Connection refused
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
qe@ubuntu:~/program$ git clone https://github.com/openbmc/openbmc.git
Cloning into 'openbmc'...
remote: Enumerating objects: 480, done.
remote: Counting objects: 100% (480/480), done.
remote: Compressing objects: 100% (306/306), done.
remote: Total 121605 (delta 242), reused 385 (delta 147), pack-reused 121125
Receiving objects: 100% (121605/121605), 53.98 MiB | 920.00 KiB/s, done.
Resolving deltas: 100% (64453/64453), done.
Checking connectivity... done.
qe@ubuntu:~/program$ git clone https://github.com/openbmc/docs.git
Cloning into 'docs'...
remote: Enumerating objects: 1418, done.
remote: Total 1418 (delta 0), reused 0 (delta 0), pack-reused 1418
Receiving objects: 100% (1418/1418), 694.88 KiB | 362.00 KiB/s, done.
Resolving deltas: 100% (823/823), done.
Checking connectivity... done.
qe@ubuntu:~/program$ cd openbmc/
qe@ubuntu:~/program/openbmc$ find meta-* -name local.conf.sample
meta-aspeed/conf/local.conf.sample
meta-evb/meta-evb-nuvoton/meta-evb-npcm750/conf/local.conf.sample
meta-evb/meta-evb-aspeed/meta-evb-ast2500/conf/local.conf.sample
meta-evb/meta-evb-enclustra/meta-evb-zx3-pm3/conf/local.conf.sample
meta-evb/meta-evb-raspberrypi/conf/local.conf.sample
meta-facebook/meta-tiogapass/conf/local.conf.sample
meta-facebook/meta-yosemitev2/conf/local.conf.sample
meta-hxt/meta-stardragon4800-rep2/conf/local.conf.sample
meta-ibm/meta-romulus/conf/local.conf.sample
meta-ibm/meta-palmetto/conf/local.conf.sample
meta-ibm/meta-witherspoon/conf/local.conf.sample
meta-ingrasys/meta-zaius/conf/local.conf.sample
meta-inspur/meta-fp5280g2/conf/local.conf.sample
meta-inspur/meta-on5263m5/conf/local.conf.sample
meta-intel/meta-s2600wf/conf/local.conf.sample
meta-inventec/meta-lanyang/conf/local.conf.sample
meta-lenovo/meta-hr630/conf/local.conf.sample
meta-lenovo/meta-hr855xg2/conf/local.conf.sample
meta-mellanox/meta-msn/conf/local.conf.sample
meta-microsoft/meta-olympus/conf/local.conf.sample
meta-phosphor/conf/local.conf.sample
meta-portwell/meta-neptune/conf/local.conf.sample
meta-qualcomm/meta-centriq2400-rep/conf/local.conf.sample
meta-quanta/meta-gsj/conf/local.conf.sample
meta-quanta/meta-f0b/conf/local.conf.sample
meta-quanta/meta-olympus-nuvoton/conf/local.conf.sample
meta-quanta/meta-q71l/conf/local.conf.sample
meta-yadro/meta-vesnin/conf/local.conf.sample
meta-yadro/meta-nicole/conf/local.conf.sample
```
选个ast2400，demo的`meta-ibm/meta-romulus/conf`是ast2500，
```bash
qe@ubuntu:~/program/openbmc$ export TEMPLATECONF=meta-ibm/meta-palmetto/conf
qe@ubuntu:~/program/openbmc$ . openbmc-env
### Initializing OE build env ###
You had no conf/local.conf file. This configuration file has therefore been
created for you with some default values. You may wish to edit it to, for
example, select a different MACHINE (target hardware). See conf/local.conf
for more information as common configuration options are commented.

You had no conf/bblayers.conf file. This configuration file has therefore been
created for you with some default values. To add additional metadata layers
into your configuration please add entries to conf/bblayers.conf.

The Yocto Project has extensive documentation about OE including a reference
manual which can be found at:
    http://yoctoproject.org/documentation

For more information about OpenEmbedded see their website:
    http://www.openembedded.org/

Common targets are:
     obmc-phosphor-image
qe@ubuntu:~/program/openbmc/build$ bitbake obmc-phosphor-image
qe@ubuntu:~/program/docs$ sudo apt-get install pandoc texlive-xetex
qe@ubuntu:~/program/docs$ fc-list #查看安装的字体
qe@ubuntu:~/program/docs$ cat ./userguide/userguide.tex 
\documentclass[]{article}

% fonts: libertine for roman text, inconsolata for monospace
\usepackage{fontspec}
% \setmainfont{Linux Libertine O}
\setmainfont{Ubuntu Condensed}
\setmonofont{Ubuntu Mono}
...
qe@ubuntu:~/program/docs$ make
```

# facebook
| board | machine      |
|:--------|:-------------|
| Wedge | ast1250 |
| Wedge100 | ast1250 |
| Wedge400 | ast2520 |
| Yosemite | ast1250 |
| lightning | ast1250 |
| yamp | ast2520 |


