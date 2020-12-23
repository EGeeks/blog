# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 创建Linux工程
点击`File > New > Application Project`，
![357](https://img-blog.csdnimg.cn/20201222103747741.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
自动弹出下面的窗口，按如下配置，
![358](https://img-blog.csdnimg.cn/20201222104013913.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
选择工程模板，两个都可以选，区别是Hello World工程会自动添加一个C文件（该文件完成向终端输出Hello World的功能），Empty工程没有添加任何文件，后面需要手动创建，
![359](https://img-blog.csdnimg.cn/20201222104043636.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
后面开发工程的时候，采用下面的方法继续添加代码文件，一般都使用C语言开发，
![360](https://img-blog.csdnimg.cn/20201222104409204.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
SDK会自动编译生成应用程序，从下方的窗口可获取应用程序的相关信息，应用程序路径是`<project dir>\Debug\*.elf`，
![361](https://img-blog.csdnimg.cn/2020122210454822.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

# 调试Linux工程
设置TCF参数，右键Edit，
![362](https://img-blog.csdnimg.cn/20201222171136240.png)
将IP改为板卡的IP，这时可以点击Test Connection测试网络是否通畅，
![363](https://img-blog.csdnimg.cn/20201222171319694.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
在工程上配置调试选项，
![364](https://img-blog.csdnimg.cn/20201222171545769.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
按下图新建一个调试项并配置，
![365](https://img-blog.csdnimg.cn/20201222171714192.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
点击Debug开始调试，进入调试视图，可以看到下方Console检测到了打印输出，可以设置断点，右边也有变量窗口和断点窗口。
![366](https://img-blog.csdnimg.cn/2020122217202172.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

# 创建裸机工程
待续。。。

# 调试裸机工程
待续。。。


