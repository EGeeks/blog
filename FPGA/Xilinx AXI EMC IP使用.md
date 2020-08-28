# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [2015.2 - UltraScale - How can I interface a STARTUPE3 primitive to axi_emc_ip or axi_quad_spi_ip so that I can access parallel NOR/BPI flash or QSPI flash after configuration?](https://china.xilinx.com/support/answers/62376.html)
> [Vivado 2013.3 - AXI EMC 2.0 results in "ERROR: [IP_​Flow 19-3460] Validation failed on parameter 'Base Address(C_​S_​AXI_​MEM0_​BASEADDR)' for Address overlapping among various memory banks..."](https://china.xilinx.com/support/answers/58320.html)
> [LogiCORE IP AXI External Memory Controller (EMC) - Release Notes and Known Issues for Vivado 2013.4 and older tool versions](https://china.xilinx.com/support/answers/54429.html)
> [如何使用KC705开发板上的BPI Flash](https://forums.xilinx.com/t5/7-Series-FPGA-%E5%85%B6%E4%BB%96-FPGA-%E5%99%A8%E4%BB%B6/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8KC705%E5%BC%80%E5%8F%91%E6%9D%BF%E4%B8%8A%E7%9A%84BPI-Flash/m-p/983467#M2330)

# BPI Flash
MT28GU01GAAA1EGC-0SIT是新款，VCU108上的，以前的名字叫PC28F00AG18FE，开发板上使用的是PC28F00AP30TF，一个是G18系列，一个是P30系列，MT28GU01GAAA1EGC-0SIT的参数，
![220](https://img-blog.csdnimg.cn/20191113103011177.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
配置axi-emc参数，
![219](https://img-blog.csdnimg.cn/20191113102823603.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
官网例子AXI_EMC_IP_STARTUPE3直接使用STARTUPE3，k7325t对应的是STARTUPE2，参考ug470，另外vivado2015.2.1的版本可以生成axi_quad_spi的代码看下官方怎么使用这个原语，
```c
   STARTUPE2 #(
      .PROG_USR("FALSE"),  // Activate program event security feature. Requires encrypted bitstreams.
      .SIM_CCLK_FREQ(0.0)  // Set the Configuration Clock Frequency(ns) for simulation.
   )
   STARTUPE2_inst (
      .CFGCLK(cfg_clk),       // 1-bit output: Configuration main clock output
      .CFGMCLK(),     // 1-bit output: Configuration internal oscillator clock output
      .EOS(),             // 1-bit output: Active high output signal indicating the End Of Startup.
      .PREQ(),           // 1-bit output: PROGRAM request to fabric output
      .CLK(1'b0),//cfg_clk             // 1-bit input: User start-up clock input
      .GSR(1'b0),             // 1-bit input: Global Set/Reset input (GSR cannot be used for the port name)
      .GTS(1'b0),             // 1-bit input: Global 3-state input (GTS cannot be used for the port name)
      .KEYCLEARB(), // 1-bit input: Clear AES Decrypter Key input from Battery-Backed RAM (BBRAM)
      .PACK(),           // 1-bit input: PROGRAM acknowledge input
      .USRCCLKO(emc_rdclk),   // 1-bit input: User CCLK input
      .USRCCLKTS(1'b0), // 1-bit input: User CCLK 3-state enable input
      .USRDONEO(1'b1),   // 1-bit input: User DONE pin output control
      .USRDONETS(1'b1)  // 1-bit input: User DONE 3-state enable output
   );
```

