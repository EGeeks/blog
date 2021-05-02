# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [【转】NFS各个版本之间的比较](https://blog.csdn.net/qigaoqiang/article/details/86233225)
> [（NFS移植到arm上）编译portmap和nfs-utils](https://blog.csdn.net/maopig/article/details/77184209)
> [嵌入式Linux平台的NFS移植](https://www.linuxidc.com/Linux/2011-04/34311.htm)
> [搭建 nfs服务器及客户端（Ubuntu/ARM）](https://www.cnblogs.com/jalynfang/p/7573085.html)
> [NFS服务器搭建与配置](https://blog.csdn.net/qq_38265137/article/details/83146421)
> [rpcbind结合nfs实现文件共享](https://blog.csdn.net/m0_46674735/article/details/110038148)
> [Poky NFS Root](https://wiki.yoctoproject.org/wiki/Poky_NFS_Root)
> [zcu102的nfs挂载问题](https://forums.xilinx.com/t5/%E5%B5%8C%E5%85%A5%E5%BC%8F-%E5%B7%A5%E5%85%B7-%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/zcu102%E7%9A%84nfs%E6%8C%82%E8%BD%BD%E9%97%AE%E9%A2%98/m-p/901262#M567)
> [rpc portmap rpcbind vxi11](https://blog.csdn.net/z1026544682/article/details/100942221)
> [网络安全NFS协议及v2,v3,v4版本搭建](https://blog.csdn.net/qq_28201689/article/details/103594051)

# nfs server
rpcbind是用来替代portmap的（rhel6 和centos6.2使用rpcbind替代了portmap）。在nfs共享时候负责通知客户端，服务器的nfs端口号的。简单理解rpc就是一个中介服务，功能和portmap类似。


# 问题
最近把Ubuntu16.04升级为Ubuntu18.04，升级完成后，导致使用正常的开发板挂载nfs网络文件系统失败，终端中报错：VFS: Unable to mount root fs via NFS, trying floppy。
查找资料发现从Ubuntu17.04开始，nfs默认只支持协议3和协议4，而kernel中默认支持协议2，所以才会出现挂载失败的情况，现有两种方法可以解决该问题：
1. 设置Ubuntu18.04系统中的nfs服务支持协议2，修改nfs配置文件 /etc/default/nfs-kernel-server,在文件末尾加入一句：RPCNFSDOPTS="--nfs-version 2,3,4 --debug --syslog"。
2. 如果kernel版本较高支持nfs协议3的话，可以在Uboot传到Kernel的bootargs参数中加入'nfsvers=3',使kernel使用nfs协议3。
