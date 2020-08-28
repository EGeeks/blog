# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> ug476

# 方法
验证平台为Xilinx ZC706开发板，xc7z045型号，收发器为GTX类似于xc7k325t，SATA3.0~SATA1.0的GTX配置均采用vivado的默认设置，数据宽度为16-20位，采用CPLL，CPLL VCO频率为3GHZ，N1=5，N2=4，M=1，D=1（SATA3.0），2（SATA2.0），4（SATA1.0），
![224](https://img-blog.csdnimg.cn/20191117171726373.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
TXOUTCLKSEL在SATA1.0时为100b，剩下为011b，通过GTX原语GTXE2_CHANNEL的管脚设置，
![225](https://img-blog.csdnimg.cn/20191117172144333.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
RXCDR_CFG取一下表格中的值，RXOUT_DIV，即D=1（SATA3.0），2（SATA2.0），4（SATA1.0），
![226](https://img-blog.csdnimg.cn/20191117172713853.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
上述几组参数对应的DRP地址为，原语为GTXE2_CHANNEL，
| name | addr      |
|:--------:|:-------------:|
| TXOUT_DIV | 0x88 [6:4] |
| RXOUT_DIV | 0x88 [2:0] |
| RXCDR_CFG [15:00] | 0xa8 |
| RXCDR_CFG [31:16] | 0xa9 |
| RXCDR_CFG [47:32] | 0xaa |
| RXCDR_CFG [63:48] | 0xab |
| RXCDR_CFG [71:64] | 0xac [7:0] |
使能OOB，PCS_RSVD_ATTR_IN[8]=1，PCS_RSVD_ATTR_IN[3]=0，这个参数如果包含了共享逻辑，是不要关心的，
![227](https://img-blog.csdnimg.cn/20191118142332744.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
