# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [TCL script to auto-generate a jtag boot script based on HDF file for Zynq Ultrascale](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/84444479/TCL+script+to+auto-generate+a+jtag+boot+script+based+on+HDF+file+for+Zynq+Ultrascale)
> [Programming QSPI from U-boot ZC702](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842240/Programming+QSPI+from+U-boot+ZC702)
# tcl语法
（1）if 判断，{}中的语句需要用[]括起来
```shell
if {} { 必须留在这一行
}
elseif而不是else if
```
（2）注释单起一行，不要在命令末尾
（3）procedure的参数用空格隔开
（4）file exists判断文件是否存在

# 生成工程
打开xsct，
```shell
zc@ubuntu:~/xilinx/fsbl$ $PETALINUX/tools/hsm/bin/xsct
$PETALINUX/tools/hsm/bin/xsct 
rlwrap: warning: your $TERM is 'xterm-256color' but rlwrap couldn't find it in the terminfo database. Expect some problems.
                                                                                                                                                                                                            
****** Xilinx Software Commandline Tool (XSCT) v2017.4
  **** Build date : Dec 13 2017-18:17:23
    ** Copyright 1986-2017 Xilinx, Inc. All Rights Reserved.


xsct% 
```
调出帮助，
```shell
xsct% help                                                                                                                                                                                                  
Available Help Categories

connections   - Target Connection Management
registers     - Target Registers
running       - Program Execution
memory        - Target Memory
download      - Target Download FPGA/BINARY
reset         - Target Reset
breakpoints   - Target Breakpoints/Watchpoints
streams       - Jtag UART
miscellaneous - Miscellaneous
jtag          - JTAG Access
tfile         - Target File System
svf           - SVF Operations
sdk           - SDK Projects
petalinux     - Petalinux commands
hsi           - HSI commands

Type "help" followed by above "category" for more details or
help" followed by the keyword "commands" to list all the commands
```
调出命令集帮助，
```shell
xsct% help hsi                                                                                                                                                                                              
Category commands

hsi create_dt_node         - Create a DT Node
hsi create_dt_tree         - Create a DT tree
hsi current_dt_tree        - Set or get current tree
hsi get_dt_nodes           - Get a list of DT node objects
hsi get_dt_trees           - Get a list of dts trees created
hsi close_hw_design        - Close a HW design
hsi current_hw_design      - Set or get current hardware design
hsi get_cells              - Get a list of cells
hsi get_hw_designs         - Get a list of hardware designs opened
hsi get_hw_files           - Get a list of hardware design supporting files
hsi get_intf_nets          - Get a list of interface nets
hsi get_intf_pins          - Get a list of interface pins
hsi get_intf_ports         - Get a list of interface ports
hsi get_mem_ranges         - Get a list of memory ranges
hsi get_nets               - Get a list of nets
hsi get_pins               - Get a list of pins
...
Type "help" followed by above "command", or the above "command" followed by
"-help" for more details
```
调出命令帮助，
```shell
xsct% hsi::open_hw_design -help                                                                                                                                                                             
hsi::open_hw_design

Description: 
Open a hardware design from disk file.

Syntax: 
hsi::open_hw_design  [-name <arg>] [-quiet] [-verbose] [<file>]

Returns: 
Hardware design object. Returns nothing if the command fails.

Usage: 
  Name        Description
  -----------------------
  [-name]     Hardware design name
  [-quiet]    Ignore command errors
  [-verbose]  Suspend message limits during command execution
  [<file>]    Hardware design file to open

Categories: 
Hardware
```
执行shell命令，利用exec，
```shell
xsct% exec ls 
```
打开hw design，
```shell
xsct% hsi::open_hw_design -name mwm178_hw hw/178.hdf                                                                                                                                                        
ERROR: [Hsi 61-74] Option name is only supported for dsa files
ERROR: [Common 17-39] 'hsi::open_hw_design' failed due to earlier errors.
xsct% hsi::open_hw_design hw/178.hdf
INFO: [Hsi 55-1698] elapsed time for repository loading 0 seconds                                                                                                                                           
hsi::open_hw_design: Time (s): cpu = 00:00:14 ; elapsed = 00:00:16 . Memory (MB): peak = 517.078 ; gain = 170.008 ; free physical = 125 ; free virtual = 2841
MWM178_V1_U6_V1
xsct% exec ls hw/                                                                                                                                                                                           
178.hdf
psu_init.c
psu_init_gpl.c
psu_init_gpl.h
psu_init.h
psu_init.html
psu_init.tcl
xsct% sdk::setws .                                                                                                                                                                      
xsct% hsi::utils::openhw hw/178.hdf 
INFO: [Hsi 55-1698] elapsed time for repository loading 0 seconds 
```
获取cell，
```shell
xsct% hsi::get_cells * -filter {NAME=~*sd*}                                                                                                                                                                               
ps7_sd_0
```
获取property，
```shell
xsct% hsi::report_property [hsi::get_cells * -filter {NAME=~*sd*}]                                                                                                                                                        
Property                             Type     Read-only  Value
ADDRESS_TAG                          string   true       
CLASS                                string   true       cell
CONFIG.C_HAS_CD                      string   true       1
CONFIG.C_HAS_POWER                   string   true       0
CONFIG.C_HAS_WP                      string   true       1
CONFIG.C_INTERCONNECT_S_AXI_MASTERS  string   true       ps7_cortexa9_0.M_AXI_DP & ps7_cortexa9_1.M_AXI_DP
CONFIG.C_SDIO_CLK_FREQ_HZ            string   true       100000000
CONFIG.C_S_AXI_BASEADDR              string   true       0xE0100000
CONFIG.C_S_AXI_HIGHADDR              string   true       0xE0100FFF
CONFIGURABLE                         bool     true       0
DRIVER_MODE                          string   true       
HIER_NAME                            string   true       
IP_NAME                              string   true       ps7_sdio
IP_TYPE                              enum     true       PERIPHERAL
IS_HIERARCHICAL                      bool     true       0
IS_PL                                bool     true       0
NAME                                 string   true       ps7_sd_0
PRODUCT_GUIDE                        string   true       
SLAVES                               string*  true       
VLNV                                 string   true       xilinx.com:ip:ps7_sdio:1.00.a
xsct% hsi::get_property CONFIG.C_SDIO_CLK_FREQ_HZ [hsi::get_cells * -filter {NAME=~*sd*}]                                                                                                                                 
100000000
```
创建fsbl，
```shell
xsct% hsi::get_cells * -filter {IP_TYPE==PROCESSOR} 
xsct% hsi::generate_app -app zynqmp_fsbl -proc psu_cortexa53_0 -dir zynqmp_fsbl -os standalone -verbose
```
tcl加法，自动支持10/16进制，
```shell
xsct% set ddr_len [expr $ddr_len + 1]
```
# 板卡下载调试
zynq上执行u-boot，
```bash
connect
source ps7_init.tcl
targets -set -filter {name =~ "APU"}
ps7_init
ps7_post_config
targets -set -filter {name =~ "ARM Cortex-A9 MPCore #0"}
dow -data BOOT.BIN 0x08000000
dow u-boot.elf
con
```

