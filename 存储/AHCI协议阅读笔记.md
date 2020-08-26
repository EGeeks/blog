# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# SCRx Port Register
`SCR0~SCR4`寄存器，可判断端口Link状态，是否Link Up，端口速度等
![299](https://img-blog.csdnimg.cn/2020050716104028.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 系统内存分配
HBA寄存器指向系统内存空间，最多32个端口，每个端口包含`Command List`和`Received FIS Structure`，
![300](https://img-blog.csdnimg.cn/20200507210903379.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
`Command List`最多32个，
![301](https://img-blog.csdnimg.cn/20200507212816102.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
`Received FIS Structure`大小是256字节，

1. `DMA setup FIS`被拷贝到DSFIS
2. `PIO setup FIS`被拷贝到PSFIS
3. `D2H Register FIS`被拷贝到RFIS
4. `Set  Device  Bits  FIS`被拷贝到SDBFIS
5. When an unknown FIS arrives from the device, the HBA copies it to the UFIS area in this structure, and 
sets PxSERR.DIAG.F, which is reflected in PxIS.UFS when the FIS is posted to memory.  A maximum of 
64-bytes of an unknown FIS type may be sent to an HBA.  If an unknown FIS arrives that is longer than 
64-bytes, the FIS is considered illegal and is handled as described in section  6.1.2.  While the length of 
the FIS is unknown to the HBA, it is expected to be known by system software, and therefore only the 
valid bytes shall be processed by software.  The HBA is not required to tolerate receiving an unknown FIS 
when the HBA is expecting a Data FIS from the device or when the HBA is about to transfer a Data FIS to 
the device based on the command protocol being used.

![302](https://img-blog.csdnimg.cn/20200507213201237.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
`Command List`的每一项`Command Header`的结构，每一个`Command Header`都指向一个`Command Table`，`PRDTL`指示`Command Table`里有多少项（第二级了），16bit，最多65535个，`W`指示数据方向，`A`指示这是一个ATAPI命令，`CFL`指示`Command FIS`的长度，`PRDBC`指示读写长度，32bit，所以单个`Command Header`最大传输2GB数据，每页4KB，则最大0xFFFF000，256MB-4KB。
![303](https://img-blog.csdnimg.cn/20200507214105102.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
![304](https://img-blog.csdnimg.cn/20200507214338702.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
![305](https://img-blog.csdnimg.cn/20200507214402732.png)
`Command Table`结构如下，`CFIS`长度由`CFL`决定，最大64字节，`ACMD`只有在ATAPI命令时才用到，`PRDT`的每一项最大指向一片4MB的内存。
![306](https://img-blog.csdnimg.cn/2020050721504425.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
![307](https://img-blog.csdnimg.cn/20200507221053676.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
`CFIS`内容参考SATA协议，下面是类型`0x27`的FIS，
![308](https://img-blog.csdnimg.cn/20200509171438874.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
![309](https://img-blog.csdnimg.cn/20200509171627591.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)





