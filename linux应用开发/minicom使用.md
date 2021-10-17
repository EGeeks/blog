# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [在minicom中自动换行](https://blog.csdn.net/hens007/article/details/7255477)
> [Ubuntu下安装和使用lrzsz，实现串口传输文件&&minicom](https://blog.csdn.net/xiao628945/article/details/8259063)

# 安装
```bash
sudo apt-get install minicom
```
- Ctrl+A，再按z，可以获取到minicom CTRL-A命令的帮助信息。
- S键：发送文件到目标系统中；
- W键：自动卷屏。当显示的内容超过一行之后，自动将后面的内容换行。这个功能在查看内核的启动信息时很有用；
- C键：清除屏幕的显示内容；
- B键：浏览Minicom的历史显示；
- X键：退出Minicom，会提示确认退出。

# minicom无法输入
Hardware Flow Contorl选项改为No。

# minicom中自动换行
在minicom中自动换行：Ctrl+A，再按Z，再按W。

# 收发文件
需要使用到另一个软件包lrzsz，` sudo apt-get install lrzsz`，通过S键，使用xmodem协议发送文件。这时候就可以正常地用minicom通过串口烧写内核了。

# FT4232
xilinx zcu104开发板在linux下有4个串口`ttyUSB0~ttyUSB3`，zu终端在`ttyUSB1`。
