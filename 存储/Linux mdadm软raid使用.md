# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [mdadm详细使用手册](https://blog.csdn.net/a7320760/article/details/10442715)
> [创建RAID5重启系统，出现md127，磁盘阵列永久挂载](https://blog.csdn.net/java_java_cl/article/details/103423552)
> [mdadm 创建md 删除md步骤](http://t.zoukankan.com/ariclee-p-6421064.html)
> [centos5.4上soft raid之/etc/mdadm.conf学习笔记](http://blog.itpub.net/9240380/viewspace-630895/)
> [RAID superblock formats](https://raid.wiki.kernel.org/index.php/RAID_superblock_formats)
> [Initial Array Creation](https://raid.wiki.kernel.org/index.php/Initial_Array_Creation)
> [Linux Software RAID的rebuild速度。](https://blog.csdn.net/weixin_34379433/article/details/92932702)
> [linux mdadm raid阵列重建加速---bitmaps文件](https://www.cnblogs.com/xinyuyuanm/archive/2013/04/13/3019586.html)

# 创建md
```bash
$ sudo mdadm -v -C /dev/md5 -l 5 -n 4 -c64 /dev/nvme0n1 /dev/nvme1n1 /dev/nvme2n1 /dev/nvme3n1
$ sudo mdadm -v -C /dev/md5 -l 5 -n 4 -c64 /dev/nvme0n1 /dev/nvme1n1 /dev/nvme2n1 /dev/nvme3n1
```
保存到配置文件，否则会有`/dev/md127`的问题，
```bash
# sudo mdadm -Ds >> /etc/mdadm.conf
$ sudo mdadm --detail --scan >> /etc/mdadm.conf
# 注意格式，有以下几种/dev/md0 /dev/md/0 /dev/md/md0
$ sudo vim /etc/mdadm.conf
# ubuntu
$ sudo update-initramfs -u
# or centos
$ sudo dracut --force
```

# 查看md
```bash
$ sudo cat /proc/mdstat
$ sudo mdadm --detail /dev/md0
$ sudo mdadm -D /dev/md0
$ sudo watch -n1 'cat /proc/mdstat'
```

# 删除md
```bash
# 停止md
$ sudo mdadm -S /dev/md127
# 依次清除md的各个盘的raid数据，sync，这个时候重启md127会消失
$ sudo mdadm --zero-superblock /dev/nvme[0123]n1
# 删除/etc/mdadm/mdadm.conf文件中添加的DEVICE行和ARRAY行
```

# 重建
```bash
$ sudo mdadm --examine /dev/nvme0n1
$ sudo mdadm --assemble --scan /dev/md0
$ sudo mdadm --wait /dev/mdX # wait for rebuild to finish
$ sudo mdadm /dev/mdX --assemble --force <list of devices>
$ sudo mdadm --action=check /dev/mdX
```
设置限速，
```bash
$ sudo sysctl dev.raid.speed_limit_min
$ sudo sysctl -w dev.raid.speed_limit_min=50000
```

# bitmap
```bash
# 开启内部bitmap
$ sudo mdadm --grow /dev/md5 --bitmap=internal
# 外部bitmap
$ sudo mdadm --grow /dev/md5 --bitmap=<file>
```

