# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# UNCALLED SEGMENT,IGNORED FOR OVERLAY PROCESS
解决无用代码占用ROM空间和去除编译告警，
![3](https://img-blog.csdnimg.cn/20200813224911820.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)
然后，
![4](https://img-blog.csdnimg.cn/20200813225228313.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)
前后对比，
```bash
Program Size: data=21.6 xdata=3413 code=58108
Program Size: data=21.6 xdata=2763 const=0 code=43872
```


# 数据类型
在keil C51或者iar for c8051编译器下，
- int 占两个字节 范围:-32768~+32767
- long占四个字节 范围:-2147483648~+2147483647
- float占四个字节 范围:3.40E+38 ~ +3.40E+38
- double占8个字节 范围:-1.79E+308 ~ +1.79E+308

# printf
STC8A单片机，Keil4，下面的printf会错乱，
```c
printf("int[%d]:%d\r\n", i, vali);
printf("float[%d]:%f\r\n", f, valf);
```
只有一个参数不会，
```c
printf("int:%d\r\n", vali);
printf("float:%f\r\n", valf);
```
int用d，long使用ld来打印，如果把数据类型由int升级到long，**要记得把d改成ld**，否则打印出错。

# sprintf
格式和数据类型必须一一对应，否则乱码，
```c
%d  -> int
%lu -> unsigned long
```

# Keil 0xfd问题
Keil编辑器设置Ascii编码，字符串中输入中文直接显示，发现汉字待和过乱码，这两个字均包含0xfd，在字符串中添加0xfd即可，
```c
  "待\xfd机            ", 
```
后续发现这是Keil的bug，需要打[补丁](https://blog.csdn.net/u010443760/article/details/80441183)。

# 数据溢出
这样的写法`ccpCount1[0] = ((ccpCfCnt - ccpCfCntBase[0]) << 16) + (CCAP0H << 8) + CCAP0L;`数据会溢出，
```c
//ccpCount1[0] = ((ccpCfCnt - ccpCfCntBase[0]) << 16) + (CCAP0H << 8) + CCAP0L;
ccpCount1[0] = (ccpCfCnt - ccpCfCntBase[0]);
ccpCount1[0] <<= 8;
ccpCount1[0] += CCAP0H;
ccpCount1[0] <<= 8;
ccpCount1[0] += CCAP0L;
```

