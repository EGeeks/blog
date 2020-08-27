# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 串口收发工具无法控制modem
使能RTS才可以，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181204233424505.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

> [UART中的硬件流控RTS与CTS](https://blog.csdn.net/zeroboundary/article/details/8966586)

RS232中RTS 与CTS 是用来半双工模式下的方向切换，如果UART只有RX、TX两个信号，要流控的话只能是软流控；如果有RX，TX，CTS，RTS 四个信号，则多半是支持硬流控的UART；如果有 RX，TX，CTS ，RTS ，DTR，DSR 六个信号是RS232标准信号。
RTS （Require To Send，发送请求）为输出信号，用于指示本设备准备好可接收数据，低电平有效，低电平说明本设备可以接收数据。
CTS （Clear To Send，发送允许）为输入信号，用于判断是否可以向对方发送数据，低电平有效，低电平说明本设备可以向对方发送数据。


# 发短信操作
注意Ctrl+Z为Z-0x40，即0x1A
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181204233920561.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
