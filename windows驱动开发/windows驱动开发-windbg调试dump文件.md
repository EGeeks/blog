# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 设置符号路径
对于WinDbg图形界面，可以点击File | Symbol File Path设置符号路径，多个路径用分号隔开。实际上只需要设置自己的路径，不需要微软的符号表，毕竟出错的是自己的代码，不是微软的代码。
```shell
srv*DownstreamStore*http://msdl.microsoft.com/download/symbols;<your path>
```

# 设置源代码路径
File | Source File Path选择源文件路径，多个路径用分号隔开。

# 分析dump文件
File | Open Crash Dump选择dump文件路径，可在c盘搜索*.dmp。点击输出窗口中的!analyze –v，WinDbg就自动定位错误所在的源代码处。
