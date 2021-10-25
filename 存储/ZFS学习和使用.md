# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [github openzfs](https://github.com/openzfs)
> [Building ZFS](https://openzfs.github.io/openzfs-docs/Developer%20Resources/Building%20ZFS.html)
> [初学者指南：ZFS 是什么，为什么要使用 ZFS？ | Linux 中国](https://blog.csdn.net/F8qG7f9YD02Pe/article/details/82836037)
> [wiki openzfs](https://openzfs.org/wiki/Main_Page)
> [ZIL（ZFS Intent Log）](http://blog.chinaunix.net/uid-24395800-id-4314226.html)
> [ZIL不是ZFS的写缓存！](https://zhuanlan.zhihu.com/p/63991068)
> [ZFS: 文件系统空间使用率的疑惑](https://postgres.fun/20140709170043.html)
> [参考文档：关闭ZFS数据去重功能](https://zhuanlan.zhihu.com/p/370254633)
> [开始使用zfs](https://blog.endaosi.com/uncategorized/zfs-start.html)
> [ZFS最佳实践指南](http://blog.sina.com.cn/s/blog_6838386a0101drox.html)
> [OpenZFS开源文件系统2.0+：持久化L2ARC读缓存、ZIL写缓存提速](https://www.sohu.com/a/439393476_314773)
> [用FreeBSD10搭建基于ZFS的iSCSI服务](https://blog.csdn.net/Raptor/article/details/28254155)
> [提升ZFS性能的10个简便方法](https://blog.51cto.com/zfs114/814105)
> [centos7从源码编译openzfs 2.1 rpm](https://developer.aliyun.com/article/788382)

# 原理
## ZIL
ZIL是ZFS的写入日志，即使没有添加独立高速ZIL，它也存在于储存池内。ARC和L2ARC才是缓存，读和写的缓存。性能上，在不同步时有没有ZIL几乎无影响；强制同步时即使有傲腾900P做独立ZIL，随机写入速度也惨不忍睹。ZIL并不需要上百GB的空间，因此在一块大固态硬盘上划分一小块做ZIL，其余作为L2ARC是更合理的选择。

## 去重
去重非常耗费内存，按照官方的建议，通常1TB存储空间需要额外提供5GB的内存。当启用Deduplication时，向数据集写入的数据越多，所需的内存就越多。如果内存过小，容量不足以保存DDT（重复数据删除表），系统会将DDT存储在磁盘上，这将会导致系统性能一落千丈。

## RAID
raidz，没有硬raid5的断电bug，错误检查可以定位到文件。

## 内存 ARC L2ARC
ZFS官方文档建议的内存需求是1G/TB数据。并且内存是需要ECC的。
- 推荐使用1GB以上的内存。
- 装载每个ZFS文件系统约耗费64KB的内存空间。在一个存在上千个ZFS文件系统的系统上，我们建议您为包括快照在内的每1000个所装载的文件系统多分配1GB额外内存。并对为其所带来的更长的引导时间有所准备。
- 因为ZFS在内核保留内存中缓存数据，所以内核的大小很可能会比使用其它文件系统时要大。您可以配置额外的基于磁盘的交换空间（Swap Space）来解决系统内存限制的这一问题。您可以使用物理内存的大小作为额外所需的交换空间的大小的上限。但无论如何，您都应该监控交换空间的使用情况 来确定是否交换正在进行。
- 如条件允许，请不要将交换空间与ZFS文件系统所使用分区（slice）放在同一磁盘上。保证交换区域同ZFS文件系统分开。最佳策略是提供足够多的内存使您的系统不常使用交换设备。
- ZFS的自适应置换高速缓存（Adaptive Replacement Cache，ARC）试图使用最多的系统可用内存来缓存文件系统数据。默认是使用除了1GB以上的所有的物理内存。当内存压迫性增长时，ARC会放弃内存。
- ZFS默认使用内存来作为读取缓存，但是也可以把SSD作为二级缓存（L2ARC）来进行使用，逻辑上来讲自然是设置二级缓存是好的，但是二级缓存需要依赖更大的内存来发挥作用，根据官方建议，针对小于32G的内存场景下，不推荐开启L2ARC缓存，并且L2ARC的缓存大小一般不能超过内存的10倍。

# OpenZFS
## 安装
`META`文件记录的支持的内核版本号，`3.10~5.9`，
```bash
Meta:          1
Name:          zfs
Branch:        1.0
Version:       2.0.0
Release:       rc1
Release-Tags:  relext
License:       CDDL
Author:        OpenZFS
Linux-Maximum: 5.9
Linux-Minimum: 3.10
```

## 卷
`-V`表示创建ZFS卷，`-s`表示不在创建时分配空间，不加此参数则会创建一个实际占用指定容量的卷。`-b`指定块大小，即传统意义上的扇区大小，一般用4096或512。`-V`参数才能在/dev/zvol下创建相应的块设备供iSCSI之用。
```bash
zfs create -s -V 4G -b 4k tank/testtarget
```

