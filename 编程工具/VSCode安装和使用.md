# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)


# 参考
> [vscode 的tab与空格设置](https://www.cnblogs.com/xuanmanstein/p/9181592.html)

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
搜索Market，安装`Verilog-HDL/SystemVerilog/Bluespec SystemVerilog`，这个下载量最大。。。

## C/C++
安装`C/C++`，自动推荐安装的，完成`C/C++ IntelliSense, debugging, and code browsing`功能。对于驱动开发，配置头文件检索路径，
```bash
${workspaceFolder}/**
/usr/src/linux-headers-5.3.0-62-generic/include
/usr/src/linux-headers-5.3.0-62-generic/include/uapi
/usr/src/linux-headers-5.3.0-62-generic/arch/x86/include
```

