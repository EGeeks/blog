# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [vscode 的tab与空格设置](https://www.cnblogs.com/xuanmanstein/p/9181592.html)
> [用VSCode连接远程Linux服务器实时修改代码的方法](https://www.cnblogs.com/zbzhm3728/articles/13710964.html)
> [VsCode SFTP插件详细使用介绍](https://blog.csdn.net/iamlujingtao/article/details/102501845)
> [工具篇-vscode sftp代码同步](https://zhuanlan.zhihu.com/p/73218983)
> [win10下vscode配置sftp](https://www.cnblogs.com/raind/p/8975978.html)
> [vscode设置ssh进行远程编辑](https://blog.csdn.net/sqlquan/article/details/111918019)
> [玩转VSCode插件之Remote-SSH](https://www.cnblogs.com/liyufeia/p/11405779.html)

# 安装
## Windows
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190223223004221.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
如果忘记添加到右键菜单，新建`reg`文件，双击执行，
```bash
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\*\shell\VSCode]
@="Open with Code"
"Icon"="C:\\Users\\we\\AppData\\Local\\Programs\\Microsoft VS Code\\Code.exe"

[HKEY_CLASSES_ROOT\*\shell\VSCode\command]
@="\"C:\\Users\\we\\AppData\\Local\\Programs\\Microsoft VS Code\\Code.exe\" \"%1\""

Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\Directory\shell\VSCode]
@="Open with Code"
"Icon"="C:\\Users\\we\\AppData\\Local\\Programs\\Microsoft VS Code\\Code.exe"

[HKEY_CLASSES_ROOT\Directory\shell\VSCode\command]
@="\"C:\\Users\\we\\AppData\\Local\\Programs\\Microsoft VS Code\\Code.exe\" \"%V\""

Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\Directory\Background\shell\VSCode]
@="Open with Code"
"Icon"="C:\\Users\\we\\AppData\\Local\\Programs\\Microsoft VS Code\\Code.exe"

[HKEY_CLASSES_ROOT\Directory\Background\shell\VSCode\command]
@="\"C:\\Users\\we\\AppData\\Local\\Programs\\Microsoft VS Code\\Code.exe\" \"%V\""
```

## Ubuntu
官网下载deb安装包，
```bash
$ sudo apt install ./code_1.50.1-1602600906_amd64.deb
```

# 快捷键
Ctrl+~ 打开命令行输入页面，默认为Powershell，选中文本，点击右键复制，在cmd中，enter为复制。

# 更改默认tab宽度
> [vscode 的tab与空格设置](https://www.cnblogs.com/xuanmanstein/p/9181592.html)

```shell
File > Preferences > Settings
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190227212157955.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 插件
## verilog
搜索Market，安装`Verilog-HDL/SystemVerilog/Bluespec SystemVerilog`，功能比较丰富，
Icarus Verilog - iverilog
Vivado Logical Simulation - xvlog
Modelsim - modelsim
Verilator - verilator

## C/C++
安装`C/C++`，自动推荐安装的，完成`C/C++ IntelliSense, debugging, and code browsing`功能。对于驱动开发，配置头文件检索路径，
```bash
${workspaceFolder}/**
/usr/src/linux-headers-5.3.0-62-generic/include
/usr/src/linux-headers-5.3.0-62-generic/include/uapi
/usr/src/linux-headers-5.3.0-62-generic/arch/x86/include
```

## SSH
通过SSH修改远程服务器上的代码，安装Remote SSH插件，`Settings > Extensions`，设置config file路径，`C:\Users\qe\.ssh`，按`ctrl+shift+p`，搜索`SFTP:Config`，配置，

## SFTP
搜索安装SFTP，按`ctrl+shift+p`，搜索`SFTP:Config`，配置，
```bash
{
    "name": "Ubuntu16.04 VM",
    "host": "192.168.91.150",
    "protocol": "sftp",
    "port": 22,
    "username": "qe",
    "password": "qe",
    "remotePath": "/home/qe/fdk_develop/package/hw",
    "uploadOnSave": true,
    "ignore": [
        "**/.vscode/**",
        "**/.git/**",
        "**/obj/**",
        "**/lib/**",
        "**/*.o",
        "**/*.a",
        "**/*static*"
    ],
    "watcher": {
        "files": "*",
        "autoUpload": false,
        "autoDelete": false
    }
}
```
![411](https://img-blog.csdnimg.cn/20210526171922854.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)


