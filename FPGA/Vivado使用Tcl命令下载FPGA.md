# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 使用
```tcl
# vivado -mode tcl

****** Vivado v2020.1 (64-bit)
  **** SW Build 2902540 on Wed May 27 19:54:35 MDT 2020
  **** IP Build 2902112 on Wed May 27 22:43:36 MDT 2020
    ** Copyright 1986-2020 Xilinx, Inc. All Rights Reserved.


Vivado% connect_hw_server -url localhost:3121
ERROR: [Labtoolstcl 44-235] The labtools system is not initialized. You must execute 'open_hw_manager' to initialize the labtools system prior using this command.
Vivado% open_hw
open_hw_manager open_hw_platform open_hw_target 
Vivado% open_hw_manager 
Vivado% connect_hw_server -url localhost:3121
INFO: [Labtools 27-2285] Connecting to hw_server url TCP:localhost:3121
INFO: [Labtools 27-2222] Launching hw_server...
INFO: [Labtools 27-2221] Launch Output:

****** Xilinx hw_server v2020.1
  **** Build date : May 27 2020 at 20:33:44
    ** Copyright 1986-2020 Xilinx, Inc. All Rights Reserved.


INFO: [Labtools 27-3415] Connecting to cs_server url TCP:localhost:3042
INFO: [Labtools 27-3417] Launching cs_server...
INFO: [Labtools 27-2221] Launch Output:


****** Xilinx cs_server v2020.1.0
  **** Build date : May 14 2020-09:10:29
    ** Copyright 2017-2020 Xilinx, Inc. All Rights Reserved.



localhost:3121
Vivado% get_hw_targets
localhost:3121/xilinx_tcf/Xilinx/2130069AF04JA
Vivado% current_hw_target [get_hw_targets]
localhost:3121/xilinx_tcf/Xilinx/2130069AF04JA
Vivado% open_hw_target
INFO: [Labtoolstcl 44-466] Opening hw_target localhost:3121/xilinx_tcf/Xilinx/2130069AF04JA
Vivado% get_hw_devices
xcu200_0
Vivado% set_property PROGRAM.FILE {/root/system_wrapper.bit} [get_hw_devices]
Vivado% program_hw_devices [get_hw_devices]
```

