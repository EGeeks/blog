# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [[Win32]ReadFile/WriteFile 的文件同步读写](https://blog.csdn.net/zuishikonghuan/article/details/46926787)
> [[Win32] 直接读写磁盘扇区（磁盘绝对读写）](https://blog.csdn.net/zuishikonghuan/article/details/50380313)

# 读写
使用ReadFile，WriteFile，读写之前把盘格式化了，不能挂载，否则只能写boot sectors。
```bash
A write on a volume handle will succeed if the volume does not have a mounted file system, or if one of the following conditions is true:
The sectors to be written to are boot sectors.
The sectors to be written to reside outside of file system space.
You have explicitly locked or dismounted the volume by using FSCTL_LOCK_VOLUME or FSCTL_DISMOUNT_VOLUME.
The volume has no actual file system. (In other words, it has a RAW file system mounted.)
A write on a disk handle will succeed if one of the following conditions is true:
The sectors to be written to do not fall within a volume's extents.
The sectors to be written to fall within a mounted volume, but you have explicitly locked or dismounted the volume by using FSCTL_LOCK_VOLUME or FSCTL_DISMOUNT_VOLUME.
The sectors to be written to fall within a volume that has no mounted file system other than RAW.
```

