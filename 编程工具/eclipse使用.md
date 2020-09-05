# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# eclipse cdt工程拷贝后不识别头文件
进入workspace目录，打开文件
```shell
D:\program\sdk2015.2\.metadata\.plugins\org.eclipse.cdt.make.core
```
修改路径为sdk的安装路径
```shell
<includePath path="c:\xilinx\sdk\2015.2\gnu\arm\nt\bin\../lib/gcc/arm-xilinx-linux-gnueabi/4.9.1/include"/>
<includePath path="c:\xilinx\sdk\2015.2\gnu\arm\nt\bin\../lib/gcc/arm-xilinx-linux-gnueabi/4.9.1/include-fixed"/>
<includePath path="c:\xilinx\sdk\2015.2\gnu\arm\nt\bin\../lib/gcc/arm-xilinx-linux-gnueabi/4.9.1/../../../../arm-xilinx-linux-gnueabi/include"/>
<includePath path="c:\xilinx\sdk\2015.2\gnu\arm\nt\bin\../arm-xilinx-linux-gnueabi/libc/usr/include"/>
```

# 显示行号和空格替代TAB
在General和C/C++下都有关于Editor的设置。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190227105816128.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)


