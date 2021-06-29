# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# Ubuntu
```bash
$ sudo apt install build-essential cmake qt5-default qtcreator
```

# centos

```bash
$ sudo yum install qt5-qtbase qt5-qtbase-devel qt5-qtquickcontrols2-devel qtcreator
```

# 问题
## QStandardPaths: XDG_RUNTIME_DIR not set, defaulting to '/tmp/runtime-root'
从命令行运行Qt，报错，
```bash
QStandardPaths: XDG_RUNTIME_DIR not set, defaulting to '/tmp/runtime-root'
```
`/etc/profile`，
```bash
export XDG_RUNTIME_DIR=/usr/lib/
export RUNLEVEL=3
```

