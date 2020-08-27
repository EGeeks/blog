# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [学习笔记之－51单片机IO口详解](https://blog.csdn.net/gjxman1314/article/details/54985734)

# 准双向模式
![110](https://img-blog.csdnimg.cn/20190621233353796.png)
应用场景，开关按下5v，不按则悬空，需要完成的功能是，读1动作，读0不动作，这里为了能读到0，需要先对IO写0，否则是读不到0的，
![111](https://img-blog.csdnimg.cn/20190621233639233.png)
