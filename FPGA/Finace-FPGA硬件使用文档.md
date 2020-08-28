# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

Finace-FPGA硬件使用文档
# 硬件介绍
板卡实物图如下所示，左方标识`eth0~eth3`为4路10g以太网SFP连接座，下方标识`PCIe金手指`为PCIeX8主机接口， 支持PCIe Gen2，传输带宽高达 3.2GByte/s，板卡为标准全高半长 PCI Express 尺寸，适合于目前主流的服务器或超微工作站，可广泛应用于数据中心和大型服务器加速运算等场景。
![250](https://img-blog.csdnimg.cn/20191214145551483.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 硬件安装
在硬件板卡安装前，应关闭所有电源系统，板卡不支持带电插拔。将该板卡插入机箱的 PCI Express 槽，安装好螺丝孔，整个板卡固定后，再上电开机。开机后，用户需根据软件使用文档安装驱动程序和应用。
