# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 方法
以zc706上sfp光口外回环来测试，选择10.3125，10GBASE-R，时钟156.25MHz，
![175](https://img-blog.csdnimg.cn/20190814101404651.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
这里时钟选择，FMC HPC的时钟，参考ug954或者原理图，
![176](https://img-blog.csdnimg.cn/20190814101515394.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
系统时钟选择外部，vivado2015.2.1上选择GTX时钟会导致编译不过去，官网说是少了IBUFDS，
![177](https://img-blog.csdnimg.cn/20190814101630765.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
配置好之后，选择
![178](https://img-blog.csdnimg.cn/20190814101846680.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
检查一下example代码，之前用了一个第三方开发板，这些代码没有自动生成，但是zc706的自动生成了，所以还是检查一下，
```c
  //
  // Refclk IBUFDS instantiations
  //

    IBUFDS_GTE2 u_buf_q1_clk0
      (
        .O            (refclk0_i[0]),
        .ODIV2        (),
        .CEB          (1'b0),
        .I            (GTREFCLK0P_I[0]),
        .IB           (GTREFCLK0N_I[0])
      );
  //
  // Sysclock IBUFDS instantiation
  //
  IBUFGDS 
   #(.DIFF_TERM("FALSE"))
   u_ibufgds
    (
      .I(SYSCLKP_I),
      .IB(SYSCLKN_I),
      .O(sysclk_i)
    );
```
约束xdc，
```c
##
## System clock pin locs and timing constraints
##
set_property PACKAGE_PIN H9 [get_ports SYSCLKP_I]
set_property IOSTANDARD DIFF_SSTL15 [get_ports SYSCLKP_I]
set_property PACKAGE_PIN G9 [get_ports SYSCLKN_I]
set_property IOSTANDARD DIFF_SSTL15 [get_ports SYSCLKN_I]

##
## MGT reference clock BUFFERS location constraints
##
set_property LOC IBUFDS_GTE2_X0Y2 [get_cells u_buf_q1_clk0]
```

# 验证
打开Hardware Manager，下载bit，选择自动扫描link，这里的errors在实验室环境下必须一直为0，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190814101956862.png)
用zc706板载的si5324在3.125和6.25G上都没问题，但是10.3125G上错误过多，扫描参数，虽然有一组参数可以将error降到个位数，但还是没法用，error在实验室环境下必须为0，否则链路质量不过关，后用FMC HPC的156.25MHz晶振，10.3125G仍然错误过多，zc706有问题。
![174](https://img-blog.csdnimg.cn/20190814102212660.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

