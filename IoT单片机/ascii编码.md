# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [Ctrl 组合键的ASCII码值的浅析](https://blog.csdn.net/wangyx1995/article/details/23789869)

# ascii
![ascii](https://img-blog.csdnimg.cn/20190818144111250.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
Linux下`showkey -a`可以得到任意你想要的键或组合键的ASCII码，依次是`Enter A Ctrl+A B Ctrl+B Backspace`， ASCII码值`1~26`被设定为`Ctrl+A~Z`组合键的ASCII码值，
```shell
$ showkey -a

Press any keys - Ctrl-D will terminate this program

^M 	 13 0015 0x0d
a 	 97 0141 0x61
^A 	  1 0001 0x01
b 	 98 0142 0x62
^B 	  2 0002 0x02
^? 	127 0177 0x7f
```

