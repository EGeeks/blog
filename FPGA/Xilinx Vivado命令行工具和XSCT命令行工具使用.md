# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> ug894-vivado-tcl-scripting.pdf
> ug835-vivado-tcl-commands.pdf
> [Programming PL in ZCU102 via FPGA Manager with BIN loaded over FTP](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/69107837/Programming+PL+in+ZCU102+via+FPGA+Manager+with+BIN+loaded+over+FTP)
> [TCL script to auto-generate a jtag boot script based on HDF file for Zynq Ultrascale](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/84444479/TCL+script+to+auto-generate+a+jtag+boot+script+based+on+HDF+file+for+Zynq+Ultrascale)

# Vivado
TCL约束报错，
```bash
foreach inst [get_cells -hier -filter {(ORIG_REF_NAME == tdma_ber_ch || REF_NAME == tdma_ber_ch)}] {
    puts "Inserting timing constraints for tdma_ber_ch instance $inst"

    # get clock periods
    set clk [get_clocks -of_objects [get_pins $inst/tx_prbs31_enable_reg_reg/C]]
    set tx_clk [get_clocks -of_objects [get_pins $inst/phy_tx_prbs31_enable_reg_reg/C]]
    set rx_clk [get_clocks -of_objects [get_pins $inst/phy_rx_prbs31_enable_reg_reg/C]]

    set clk_period [get_property -min PERIOD $clk]
    set tx_clk_period [get_property -min PERIOD $tx_clk]
    set rx_clk_period [get_property -min PERIOD $rx_clk]

    set min_clk_period [expr $tx_clk_period < $write_clk_period ? $tx_clk_period : $write_clk_period]

    # control synchronization
    set_property ASYNC_REG TRUE [get_cells -hier -regexp ".*/phy_(rx|tx)_prbs31_enable_reg_reg" -filter "PARENT == $inst"]

    set_max_delay -from [get_cells "$inst/tx_prbs31_enable_reg_reg"] -to [get_cells "$inst/phy_tx_prbs31_enable_reg_reg"] -datapath_only $clk_period
    set_max_delay -from [get_cells "$inst/rx_prbs31_enable_reg_reg"] -to [get_cells "$inst/phy_rx_prbs31_enable_reg_reg"] -datapath_only $clk_period

    # data synchronization
    set_property ASYNC_REG TRUE [get_cells -hier -regexp ".*/rx_flag_sync_reg_\[123\]_reg" -filter "PARENT == $inst"]

    set_max_delay -from [get_cells "$inst/phy_rx_flag_reg_reg"] -to [get_cells $inst/rx_flag_sync_reg_1_reg] -datapath_only $rx_clk_period

    set_max_delay -from [get_cells "$inst/phy_rx_error_count_reg_reg[*]"] -to [get_cells $inst/phy_rx_error_count_sync_reg_reg[*]] -datapath_only $rx_clk_period
    set_bus_skew  -from [get_cells "$inst/phy_rx_error_count_reg_reg[*]"] -to [get_cells $inst/phy_rx_error_count_sync_reg_reg[*]] $clk_period
}
```
`get_property`报错，
```bash
ERROR: [Common 17-55] 'get_property' expects at least one object. [/home/qe/project/vivado/2020.1/corundum/fpga/common/syn/tdma_ber_ch.tcl:33]
Resolution: If [get_<value>] was used to populate the object, check to make sure this command returns at least one valid object.
```
下面来定位一下，直接在Vivado Tcl Console中执行`get_cells -hier`，提示先打开xxx design，
```bash
get_cells -hier
ERROR: [Common 17-53] User Exception: No open design. Please open an elaborated, synthesized or implemented design before executing this command.
open_run synth_1 -name synth_1
```
下面开始调试这个语句问题，首先`help get_cells`看看怎么用，
```bash
get_cells -hier *tdma_ber_ch*
genblk1[0].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[16]_i_1 genblk1[0].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[24]_i_1 genblk1[0].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[8]_i_1 genblk1[1].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[16]_i_1 genblk1[1].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[24]_i_1 genblk1[1].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[8]_i_1 genblk1[2].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[16]_i_1 genblk1[2].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[24]_i_1 genblk1[2].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[8]_i_1 genblk1[3].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[16]_i_1 genblk1[3].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[24]_i_1 genblk1[3].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[8]_i_1 genblk1[4].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[16]_i_1 genblk1[4].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[24]_i_1 genblk1[4].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[8]_i_1 genblk1[5].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[16]_i_1 genblk1[5].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[24]_i_1 genblk1[5].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[8]_i_1 genblk1[6].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[16]_i_1 genblk1[6].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[24]_i_1 genblk1[6].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[8]_i_1 genblk1[7].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[16]_i_1 genblk1[7].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[24]_i_1 genblk1[7].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[8]_i_1 core_inst/tdma_ber_inst/genblk1[0].tdma_ber_ch_inst core_inst/tdma_ber_inst/genblk1[1].tdma_ber_ch_inst core_inst/tdma_ber_inst/genblk1[2].tdma_ber_ch_inst core_inst/tdma_ber_inst/genblk1[3].tdma_ber_ch_inst core_inst/tdma_ber_inst/genblk1[4].tdma_ber_ch_inst core_inst/tdma_ber_inst/genblk1[5].tdma_ber_ch_inst core_inst/tdma_ber_inst/genblk1[6].tdma_ber_ch_inst core_inst/tdma_ber_inst/genblk1[7].tdma_ber_ch_inst
```
全打印到一行了，换一种方式，看着方便些，
```bash
join [get_cells -hier *tdma_ber_ch*] \n
genblk1[0].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[16]_i_1
genblk1[0].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[24]_i_1
genblk1[0].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[8]_i_1
genblk1[1].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[16]_i_1
genblk1[1].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[24]_i_1
genblk1[1].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[8]_i_1
genblk1[2].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[16]_i_1
genblk1[2].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[24]_i_1
genblk1[2].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[8]_i_1
genblk1[3].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[16]_i_1
genblk1[3].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[24]_i_1
genblk1[3].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[8]_i_1
genblk1[4].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[16]_i_1
genblk1[4].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[24]_i_1
genblk1[4].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[8]_i_1
genblk1[5].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[16]_i_1
genblk1[5].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[24]_i_1
genblk1[5].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[8]_i_1
genblk1[6].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[16]_i_1
genblk1[6].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[24]_i_1
genblk1[6].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[8]_i_1
genblk1[7].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[16]_i_1
genblk1[7].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[24]_i_1
genblk1[7].tdma_ber_ch_inst/rx_ts_update_count_reg_reg[8]_i_1
core_inst/tdma_ber_inst/genblk1[0].tdma_ber_ch_inst
core_inst/tdma_ber_inst/genblk1[1].tdma_ber_ch_inst
core_inst/tdma_ber_inst/genblk1[2].tdma_ber_ch_inst
core_inst/tdma_ber_inst/genblk1[3].tdma_ber_ch_inst
core_inst/tdma_ber_inst/genblk1[4].tdma_ber_ch_inst
core_inst/tdma_ber_inst/genblk1[5].tdma_ber_ch_inst
core_inst/tdma_ber_inst/genblk1[6].tdma_ber_ch_inst
core_inst/tdma_ber_inst/genblk1[7].tdma_ber_ch_inst
```
使用`report_property`查看属性，可以看到`ORIG_REF_NAME`是`NULL`，`REF_NAME`是`tdma_ber_ch`，
```bash
report_property -all [get_cells -hier -filter { NAME == core_inst/tdma_ber_inst/genblk1[0].tdma_ber_ch_inst }] -regexp .*NAME.*
Property        Type    Read-only  Value
AIE.TILE.NAME   string  false      
FILE_NAME       string  true       /home/qe/project/vivado/2020.1/corundum/fpga/mqnic/AU200/fpga_10g/rtl/common/tdma_ber.v
MACRO_NAME      string  true       
NAME            string  true       core_inst/tdma_ber_inst/genblk1[0].tdma_ber_ch_inst
ORIG_CELL_NAME  string  false      
ORIG_REF_NAME   string  true       
REF_NAME        string  true       tdma_ber_ch
RTL_RAM_NAME    string  false      
report_property -all [get_cells -hier -regexp -filter { NAME == core_inst/tdma_ber_inst/genblk1[0].tdma_ber_ch_inst }] -regexp .*NAME.*
Property        Type    Read-only  Value
AIE.TILE.NAME   string  false      
FILE_NAME       string  true       /home/qe/project/vivado/2020.1/corundum/fpga/mqnic/AU200/fpga_10g/rtl/common/tdma_ber.v
MACRO_NAME      string  true       
NAME            string  true       core_inst/tdma_ber_inst/genblk1[0].tdma_ber_ch_inst
ORIG_CELL_NAME  string  false      
ORIG_REF_NAME   string  true       
REF_NAME        string  true       tdma_ber_ch
RTL_RAM_NAME    string  false  
```
执行，
```bash
join [get_cells -hier -filter {(ORIG_REF_NAME == tdma_ber_ch || REF_NAME == tdma_ber_ch_inst)}] \n
core_inst/tdma_ber_inst/genblk1[1].tdma_ber_ch_inst
core_inst/tdma_ber_inst/genblk1[2].tdma_ber_ch_inst
core_inst/tdma_ber_inst/genblk1[3].tdma_ber_ch_inst
core_inst/tdma_ber_inst/genblk1[4].tdma_ber_ch_inst
core_inst/tdma_ber_inst/genblk1[5].tdma_ber_ch_inst
core_inst/tdma_ber_inst/genblk1[6].tdma_ber_ch_inst
core_inst/tdma_ber_inst/genblk1[7].tdma_ber_ch_inst
```
执行，
```bash
join [get_pins -filter {NAME =~ *prbs31*} -of [get_cells -hier -regexp -filter { NAME == core_inst/tdma_ber_inst/genblk1[0].tdma_ber_ch_inst }]] \n
core_inst/tdma_ber_inst/genblk1[0].tdma_ber_ch_inst/phy_rx_prbs31_enable_reg_reg_0[0]
core_inst/tdma_ber_inst/genblk1[0].tdma_ber_ch_inst/phy_tx_prbs31_enable_reg_reg_0[0]
core_inst/tdma_ber_inst/genblk1[0].tdma_ber_ch_inst/rx_prbs31_enable_reg_reg_0
core_inst/tdma_ber_inst/genblk1[0].tdma_ber_ch_inst/tx_prbs31_enable_next
core_inst/tdma_ber_inst/genblk1[0].tdma_ber_ch_inst/tx_prbs31_enable_reg_reg_0
```
定位到错误是`phy_rx_prbs31_enable_reg_reg`变为`phy_rx_prbs31_enable_reg_reg_0`了。
```bash
join [get_pins -regexp -filter {NAME =~ .*/tx_prbs31_enable_reg_reg.*} -of [get_cells -hier -regexp -filter { NAME == core_inst/tdma_ber_inst/genblk1[0].tdma_ber_ch_inst }]] \n
core_inst/tdma_ber_inst/genblk1[0].tdma_ber_ch_inst/tx_prbs31_enable_reg_reg_0
join [get_pins -filter {NAME =~ */tx_prbs31_enable_reg_reg*} -of [get_cells -hier -regexp -filter { NAME == core_inst/tdma_ber_inst/genblk1[0].tdma_ber_ch_inst }]] \n
core_inst/tdma_ber_inst/genblk1[0].tdma_ber_ch_inst/tx_prbs31_enable_reg_reg_0
```
执行，找不到时钟，`pcie_user_clk`来自于顶层PCIe硬核模块，
```bash
join [get_clocks -of_objects [get_pins -filter {NAME =~ */tx_prbs31_enable_reg_reg*} -of [get_cells -hier -regexp -filter { NAME == core_inst/tdma_ber_inst/genblk1[0].tdma_ber_ch_inst }]]] \n
INFO: [Timing 38-35] Done setting XDC timing constraints.
WARNING: [Timing 38-3] User defined clock exists on pin pcie4_uscale_plus_inst/inst/gt_top_i/diablo_gt.diablo_gt_phy_wrapper/gt_wizard.gtwizard_top_i/pcie4_uscale_plus_0_gt_i/inst/gen_gtwizard_gtye4_top.pcie4_uscale_plus_0_gt_gtwizard_gtye4_inst/gen_gtwizard_gtye4.gen_channel_container[32].gen_enabled_channel.gtye4_channel_wrapper_inst/channel_inst/gtye4_channel_gen.gen_gtye4_channel_inst[3].GTYE4_CHANNEL_PRIM_INST/TXOUTCLK [See /home/qe/project/vivado/2020.1/corundum/fpga/mqnic/AU200/fpga_10g/fpga/fpga.srcs/sources_1/ip/pcie4_uscale_plus_0/source/ip_pcie4_uscale_plus_x1y2.xdc:127] and will prevent any subsequent automatic derivation of generated clocks on that pin. If the user defined clock specifies '-add', any existing auto-derived clocks on that pin are retained.
WARNING: [Vivado 12-1008] No clocks found for command 'get_clocks -of_objects [get_pins -filter {NAME =~ */tx_prbs31_enable_reg_reg*} -of [get_cells -hier -regexp -filter { NAME == core_inst/tdma_ber_inst/genblk1[0].tdma_ber_ch_inst }]]'.
Resolution: Verify the create_clock command was called to create the clock object before it is referenced.
```
查看PCIe IP的example，
```bash
#
#
#
# CLOCK_ROOT LOCKing to Reduce CLOCK SKEW
# Add/Edit  Clock Routing Option to improve clock path skew
#
# BITFILE/BITSTREAM compress options
# ##############################################################################
# Flash Programming Example Settings: These should be modified to match the target board.
# ##############################################################################
#
#
# sys_clk vs TXOUTCLK
set_clock_groups -name async18 -asynchronous -group [get_clocks {sys_clk}] -group [get_clocks -of_objects [get_pins -hierarchical -filter {NAME =~ *gen_channel_container[32].*gen_gtye4_channel_inst[3].GTYE4_CHANNEL_PRIM_INST/TXOUTCLK}]]
set_clock_groups -name async19 -asynchronous -group [get_clocks -of_objects [get_pins -hierarchical -filter {NAME =~ *gen_channel_container[32].*gen_gtye4_channel_inst[3].GTYE4_CHANNEL_PRIM_INST/TXOUTCLK}]] -group [get_clocks {sys_clk}]
#
#
#
#
#
#
# ASYNC CLOCK GROUPINGS
# sys_clk vs user_clk
set_clock_groups -name async5 -asynchronous -group [get_clocks {sys_clk}] -group [get_clocks -of_objects [get_pins pcie4_uscale_plus_0_i/inst/gt_top_i/diablo_gt.diablo_gt_phy_wrapper/phy_clk_i/bufg_gt_userclk/O]]
set_clock_groups -name async6 -asynchronous -group [get_clocks -of_objects [get_pins pcie4_uscale_plus_0_i/inst/gt_top_i/diablo_gt.diablo_gt_phy_wrapper/phy_clk_i/bufg_gt_userclk/O]] -group [get_clocks {sys_clk}]
# sys_clk vs pclk
set_clock_groups -name async1 -asynchronous -group [get_clocks {sys_clk}] -group [get_clocks -of_objects [get_pins pcie4_uscale_plus_0_i/inst/gt_top_i/diablo_gt.diablo_gt_phy_wrapper/phy_clk_i/bufg_gt_pclk/O]]
set_clock_groups -name async2 -asynchronous -group [get_clocks -of_objects [get_pins pcie4_uscale_plus_0_i/inst/gt_top_i/diablo_gt.diablo_gt_phy_wrapper/phy_clk_i/bufg_gt_pclk/O]] -group [get_clocks {sys_clk}]
#
#
#
# Add/Edit Pblock slice constraints for 512b soft logic to improve timing
#create_pblock soft_512b; add_cells_to_pblock [get_pblocks soft_512b] [get_cells {pcie4_uscale_plus_0_i/inst/pcie_4_0_pipe_inst/pcie_4_0_init_ctrl_inst pcie4_uscale_plus_0_i/inst/pcie_4_0_pipe_inst/pcie4_0_512b_intfc_mod}]
# Keep This Logic Left/Right Side Of The PCIe Block (Whichever is near to the FPGA Boundary)
#resize_pblock [get_pblocks soft_512b] -add {SLICE_X157Y300:SLICE_X168Y370}
#set_property EXCLUDE_PLACEMENT 1 [get_pblocks soft_512b]
#
set_clock_groups -name async24 -asynchronous -group [get_clocks -of_objects [get_pins pcie4_uscale_plus_0_i/inst/gt_top_i/diablo_gt.diablo_gt_phy_wrapper/phy_clk_i/bufg_gt_intclk/O]] -group [get_clocks {sys_clk}]
#
#create_waiver -type METHODOLOGY -id {LUTAR-1} -user "pcie4_uscale_plus" -desc "user link up is synchroized in the user clk so it is safe to ignore"  -internal -scoped -tags 1024539  -objects [get_cells { pcie_app_uscale_i/PIO_i/len_i[5]_i_4 }] -objects [get_pins { pcie4_uscale_plus_0_i/inst/user_lnk_up_cdc/arststages_ff_reg[0]/CLR pcie4_uscale_plus_0_i/inst/user_lnk_up_cdc/arststages_ff_reg[1]/CLR }] 
```
研究一下`set_clock_groups`的意思，


# 远程调试记录

```bash
$ source /tools/Xilinx/Vivado/2019.2/settings64.sh
Vivado% open_hw_manager
Vivado% connect_hw_server -help
Vivado% connect_hw_server
INFO: [Labtools 27-2285] Connecting to hw_server url TCP:localhost:3121
INFO: [Labtools 27-2222] Launching hw_server...
INFO: [Labtools 27-2221] Launch Output:

****** Xilinx hw_server v2019.2
  **** Build date : Nov  6 2019 at 22:13:42
    ** Copyright 1986-2019 Xilinx, Inc. All Rights Reserved.


INFO: [Labtools 27-3415] Connecting to cs_server url TCP:localhost:3042
INFO: [Labtools 27-3417] Launching cs_server...
INFO: [Labtools 27-2221] Launch Output:


****** Xilinx cs_server v2019.2.0
  **** Build date : Nov 07 2019-13:41:48
    ** Copyright 2017-2019 Xilinx, Inc. All Rights Reserved.



localhost:3121
Vivado% current_hw_target
localhost:3121/xilinx_tcf/Xilinx/2130069AF03KA
Vivado% open_hw_target -jtag_mode on
INFO: [Labtoolstcl 44-466] Opening hw_target localhost:3121/xilinx_tcf/Xilinx/2130069AF03KA
INFO: [Labtoolstcl 44-467] Setting hw_target localhost:3121/xilinx_tcf/Xilinx/2130069AF03KA into jtag_mode
Vivado% get_hw_devices
xcu200_0
Vivado% repo
report_bus_skew report_carry_chains report_cdc report_clock_interaction report_clock_networks report_clock_utilization report_clocks report_compile_order report_config_implementation report_config_timing report_control_sets report_datasheet report_debug_core report_design_analysis report_disable_timing report_drc report_environment report_exceptions report_high_fanout_nets report_hw_axi_txn report_hw_ddrmc report_hw_mig report_hw_targets report_incremental_reuse report_io report_ip_status report_methodology report_operating_conditions report_param report_phys_opt report_pipeline_analysis report_power report_power_opt report_pr_configuration_analysis report_property report_pulse_width report_qor_assessment report_qor_suggestions report_ram_utilization report_route_status report_sim_device report_simlib_info report_ssn report_switching_activity report_synchronizer_mtbf report_timing report_timing_summary report_transformed_primitives report_utilization report_waivers 
Vivado% report_property [get_hw_devices]
Property                                                            Type          Read-only  Value
BSCAN_SWITCH_USER_MASK                                              string        false      0001
CLASS                                                               string        true       hw_device
DID                                                                 string        true       jsn-A-U200-P64G FT4232H-2130069AF03KA-14b37093-0
FULL_PROBES.FILE                                                    string        false      
IDCODE                                                              string        true       00010100101100110111000010010011
IDCODE_HEX                                                          string        true       14B37093
INDEX                                                               int           true       0
IR_LENGTH                                                           int           true       18
IS_SYSMON_SUPPORTED                                                 bool          true       1
MASK                                                                string        true       00001111111111111111111111111111
MASK_HEX                                                            string        true       0FFFFFFF
NAME                                                                string        true       xcu200_0
PART                                                                string        true       xcu200
PARTIAL_PROBES.FILES                                                string*       false      
PROBES.FILE                                                         string        false      
PROGRAM.DPA_COUNT                                                   int           false      0
PROGRAM.DPA_MODE                                                    string        false      
PROGRAM.DPA_PROTECT                                                 bool          false      0
PROGRAM.FILE                                                        string        false      
PROGRAM.HW_BITSTREAM                                                hw_bitstream  true       
PROGRAM.HW_CFGMEM                                                   hw_cfgmem     true       
PROGRAM.HW_CFGMEM_BITFILE                                           string        true       
PROGRAM.HW_CFGMEM_TYPE                                              string        true       
PROGRAM.IS_AES_PROGRAMMED                                           bool          true       0
PROGRAM.IS_RSA_PROGRAMMED                                           bool          true       0
PROGRAM.IS_SUPPORTED                                                bool          true       1
PROGRAM.OPTIONS                                                     string        false      
PROGRAM.READBACK_FILE                                               string        false      
REGISTER.BOOT_STATUS                                                string        true       00000000000000000000000000000101
REGISTER.BOOT_STATUS.BIT00_0_STATUS_VALID                           string        true       1
REGISTER.BOOT_STATUS.BIT01_0_FALLBACK                               string        true       0
REGISTER.BOOT_STATUS.BIT02_0_INTERNAL_PROG                          string        true       1
REGISTER.BOOT_STATUS.BIT03_0_WATCHDOG_TIMEOUT_ERROR                 string        true       0
REGISTER.BOOT_STATUS.BIT04_0_ID_ERROR                               string        true       0
REGISTER.BOOT_STATUS.BIT05_0_CRC_ERROR                              string        true       0
REGISTER.BOOT_STATUS.BIT06_0_WRAP_ERROR                             string        true       0
REGISTER.BOOT_STATUS.BIT07_0_SECURITY_ERROR                         string        true       0
REGISTER.BOOT_STATUS.BIT08_1_STATUS_VALID                           string        true       0
REGISTER.BOOT_STATUS.BIT09_1_FALLBACK                               string        true       0
REGISTER.BOOT_STATUS.BIT10_1_INTERNAL_PROG                          string        true       0
REGISTER.BOOT_STATUS.BIT11_1_WATCHDOG_TIMEOUT_ERROR                 string        true       0
REGISTER.BOOT_STATUS.BIT12_1_ID_ERROR                               string        true       0
REGISTER.BOOT_STATUS.BIT13_1_CRC_ERROR                              string        true       0
REGISTER.BOOT_STATUS.BIT14_1_WRAP_ERROR                             string        true       0
REGISTER.BOOT_STATUS.BIT15_1_SECURITY_ERROR                         string        true       0
REGISTER.BOOT_STATUS.BIT16_RESERVED                                 string        true       0000000000000000
REGISTER.CONFIG_STATUS                                              string        true       00010000100100000111100111111100
REGISTER.CONFIG_STATUS.BIT00_CRC_ERROR                              string        true       0
REGISTER.CONFIG_STATUS.BIT01_DECRYPTOR_ENABLE                       string        true       0
REGISTER.CONFIG_STATUS.BIT02_PLL_LOCK_STATUS                        string        true       1
REGISTER.CONFIG_STATUS.BIT03_DCI_MATCH_STATUS                       string        true       1
REGISTER.CONFIG_STATUS.BIT04_END_OF_STARTUP_(EOS)_STATUS            string        true       1
REGISTER.CONFIG_STATUS.BIT05_GTS_CFG_B_STATUS                       string        true       1
REGISTER.CONFIG_STATUS.BIT06_GWE_STATUS                             string        true       1
REGISTER.CONFIG_STATUS.BIT07_GHIGH_STATUS                           string        true       1
REGISTER.CONFIG_STATUS.BIT08_MODE_PIN_M[0]                          string        true       1
REGISTER.CONFIG_STATUS.BIT09_MODE_PIN_M[1]                          string        true       0
REGISTER.CONFIG_STATUS.BIT10_MODE_PIN_M[2]                          string        true       0
REGISTER.CONFIG_STATUS.BIT11_INIT_B_INTERNAL_SIGNAL_STATUS          string        true       1
REGISTER.CONFIG_STATUS.BIT12_INIT_B_PIN                             string        true       1
REGISTER.CONFIG_STATUS.BIT13_DONE_INTERNAL_SIGNAL_STATUS            string        true       1
REGISTER.CONFIG_STATUS.BIT14_DONE_PIN                               string        true       1
REGISTER.CONFIG_STATUS.BIT15_IDCODE_ERROR                           string        true       0
REGISTER.CONFIG_STATUS.BIT16_SECURITY_ERROR                         string        true       0
REGISTER.CONFIG_STATUS.BIT17_SYSTEM_MONITOR_OVER-TEMP_ALARM_STATUS  string        true       0
REGISTER.CONFIG_STATUS.BIT18_CFG_STARTUP_STATE_MACHINE_PHASE        string        true       100
REGISTER.CONFIG_STATUS.BIT21_SECURITY_STATUS                        string        true       100
REGISTER.CONFIG_STATUS.BIT24_RESERVED                               string        true       0
REGISTER.CONFIG_STATUS.BIT25_CFG_BUS_WIDTH_DETECTION                string        true       00
REGISTER.CONFIG_STATUS.BIT27_SECURITY_AUTH_ERROR                    string        true       0
REGISTER.CONFIG_STATUS.BIT28_PUDC_B_PIN                             string        true       1
REGISTER.CONFIG_STATUS.BIT29_BAD_PACKET_ERROR                       string        true       0
REGISTER.CONFIG_STATUS.BIT30_CFGBVS_PIN                             string        true       0
REGISTER.CONFIG_STATUS.BIT31_RESERVED                               string        true       0
REGISTER.COR0.BIT00_GWE_CYCLE                                       string        true       101
REGISTER.COR0.BIT03_GTS_CYCLE                                       string        true       100
REGISTER.COR0.BIT06_LOCK_CYCLE                                      string        true       111
REGISTER.COR0                                                       string        true       384235e5
REGISTER.COR0.BIT09_MATCH_CYCLE                                     string        true       010
REGISTER.COR0.BIT12_DONE_CYCLE                                      string        true       011
REGISTER.COR0.BIT15_RESERVED                                        string        true       00
REGISTER.COR0.BIT17_OSCFSEL                                         string        true       100001
REGISTER.COR0.BIT23_RESERVED                                        string        true       0
REGISTER.COR0.BIT24_DRIVE_DONE                                      string        true       0
REGISTER.COR0.BIT25_RESERVED                                        string        true       0
REGISTER.COR0.BIT26_ECLK_EN                                         string        true       0
REGISTER.COR0.BIT27_RESERVED                                        string        true       00111
REGISTER.COR1.BIT00_BPI_PAGE_SIZE                                   string        true       00
REGISTER.COR1.BIT02_BPI_1ST_READ_CYCLE                              string        true       00
REGISTER.COR1.BIT04_RESERVED                                        string        true       0000
REGISTER.COR1.BIT08_RBCRC_EN                                        string        true       0
REGISTER.COR1.BIT09_RBCRC_NO_PIN                                    string        true       0
REGISTER.COR1.BIT10_RESERVED                                        string        true       00000
REGISTER.COR1.BIT15_RBCRC_ACTION                                    string        true       00
REGISTER.COR1.BIT17_PERSIST_DEASSERT_AT_DESYNC                      string        true       0
REGISTER.COR1.BIT18_RESERVED                                        string        true       00000000010000
REGISTER.COR1                                                       string        true       00400000
REGISTER.EFUSE.DNA_PORT                                             string        true       40020000012942A04CA02345
REGISTER.EFUSE.FUSE_CNTL                                            string        true       144400
REGISTER.EFUSE.FUSE_DNA                                             string        true       40020000012942A04CA02345
REGISTER.EFUSE.FUSE_KEY                                             string        true       Unreadable
REGISTER.EFUSE.FUSE_RSA                                             string        true       000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
REGISTER.EFUSE.FUSE_SECURITY                                        string        true       00
REGISTER.EFUSE.FUSE_USER                                            string        true       00000000
REGISTER.EFUSE.FUSE_USER_128                                        string        true       00000000000000000000000000000000
REGISTER.IR                                                         string        true       110101110101110101
REGISTER.IR.BIT00_ALWAYS_ONE                                        string        true       1
REGISTER.IR.BIT01_ALWAYS_ZERO                                       string        true       0
REGISTER.IR.BIT02_ISC_DONE                                          string        true       1
REGISTER.IR.BIT03_ISC_ENABLED                                       string        true       0
REGISTER.IR.BIT04_INIT_COMPLETE                                     string        true       1
REGISTER.IR.BIT05_DONE                                              string        true       1
REGISTER.IR.BIT06                                                   string        true       1
REGISTER.IR.BIT07                                                   string        true       0
REGISTER.IR.BIT08                                                   string        true       1
REGISTER.IR.BIT09                                                   string        true       0
REGISTER.IR.BIT10                                                   string        true       1
REGISTER.IR.BIT11                                                   string        true       1
REGISTER.IR.BIT12                                                   string        true       1
REGISTER.IR.BIT13                                                   string        true       0
REGISTER.IR.BIT14                                                   string        true       1
REGISTER.IR.BIT15                                                   string        true       0
REGISTER.IR.BIT16                                                   string        true       1
REGISTER.IR.BIT17                                                   string        true       1
REGISTER.TIMER                                                      string        true       00000000
REGISTER.TIMER.BIT00_TIMER_VALUE                                    string        true       000000000000000000000000000000
REGISTER.TIMER.BIT30_TIMER_CFG_MON                                  string        true       0
REGISTER.TIMER.BIT31_TIMER_USR_MON                                  string        true       0
REGISTER.USERCODE                                                   string        true       ffffffff
REGISTER.USR_ACCESS                                                 string        true       00000000
REGISTER.WBSTAR                                                     string        true       00000000
REGISTER.WBSTAR.BIT00_START_ADDR                                    string        true       00000000000000000000000000000
REGISTER.WBSTAR.BIT29_RS_TS_B                                       string        true       0
REGISTER.WBSTAR.BIT30_RS                                            string        true       00
UNKNOWN_DEVICE                                                      bool          true       0
USER_CHAIN_COUNT                                                    string        true       4
VARIANT_NAME                                                        string        true       
XSDB_USER_BSCAN                                                     string        false      1,3
Vivado% set_property -help
Vivado% set_property PROGRAM.FILE fpga.bit # table键补全
Desktop/ Documents/ Downloads/ Music/ Pictures/ Public/ Sunlogin Files/ SunloginClient-10.1.1.38139_amd64.rpm Templates/ Videos/ anaconda-ks.cfg fpga.bit install_drivers/ install_drivers.tar.gz root rpm/ sensors/ shell/ vivado.jou vivado.log vivado_20001.backup.jou vivado_20001.backup.log vivado_6643.backup.jou vivado_6643.backup.log vivado_882.backup.jou vivado_882.backup.log vivado_pid20001.str 
Vivado% set_property PROGRAM.FILE fpga.bit [get_hw_devices]
Vivado% report_property [get_hw_devices] # 再次查看已经设置正确
Vivado% program_hw_device [current_hw_device]
ERROR: [Labtools 27-3082] Target in jtag mode, cannot program device
```
不知道为什么不对，
```bash
Vivado% set_property PROGRAM.FILE {/root/fpga.bit} [lindex [get_hw_devices] 0]
Vivado% 
Vivado% report_property [lindex [get_hw_devices] 0]
Vivado% program_hw_device [current_hw_device]
ERROR: [Labtools 27-3082] Target in jtag mode, cannot program device
```
曲线救国，调试一下，但是我的2020.1和2019.2不兼容，
```bash
[root@localhost ~]# hw_server

****** Xilinx hw_server v2019.2
  **** Build date : Nov  6 2019 at 22:13:42
    ** Copyright 1986-2019 Xilinx, Inc. All Rights Reserved.

INFO: hw_server application started
INFO: Use Ctrl-C to exit hw_server application

INFO: To connect to this hw_server instance use url: TCP:localhost.localdomain:3121
```
把2020.1的hw_server搞到服务器上，从GUI操作对应的tcl命令，
```bash
open_hw_manager
INFO: [IP_Flow 19-234] Refreshing IP repositories
INFO: [IP_Flow 19-1704] No user IP repositories specified
INFO: [IP_Flow 19-2313] Loaded Vivado IP repository '/opt/Xilinx/Vivado/2020.1/data/ip'.
connect_hw_server -allow_non_jtag 
# connect_hw_server -url localhost:3121 -allow_non_jtag
# connect_hw_server -url 172.20.77.109:3121 -allow_non_jtag
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



open_hw_target
INFO: [Labtoolstcl 44-466] Opening hw_target localhost:3121/xilinx_tcf/Xilinx/2130069AF04JA
current_hw_device [get_hw_devices xcu200_0]
refresh_hw_device -update_hw_probes false [lindex [get_hw_devices xcu200_0] 0]
INFO: [Labtools 27-2302] Device xcu200 (JTAG device index = 0) is programmed with a design that has 2 ILA core(s).
set_property PROBES.FILE {/root/system_wrapper.ltx} [get_hw_devices xcu200_0]
set_property FULL_PROBES.FILE {/root/system_wrapper.ltx} [get_hw_devices xcu200_0]
set_property PROGRAM.FILE {/root/download.bit} [get_hw_devices xcu200_0]
program_hw_devices [get_hw_devices xcu200_0]
INFO: [Labtools 27-3164] End of startup status: HIGH
program_hw_devices: Time (s): cpu = 00:00:19 ; elapsed = 00:00:19 . Memory (MB): peak = 8414.184 ; gain = 0.000 ; free physical = 57721 ; free virtual = 92510
refresh_hw_device [lindex [get_hw_devices xcu200_0] 0]
close_hw_manager
```
 提取出脚本备用，
```bash
$ vim download.tcl
$ cat download.tcl 
open_hw_manager
connect_hw_server -allow_non_jtag
current_hw_target [lindex [get_hw_targets] 0]
set_property PARAM.FREQUENCY 15000000 [lindex [get_hw_targets] 0]
open_hw_target
current_hw_device [get_hw_devices xcu200_0]
refresh_hw_device -update_hw_probes false [lindex [get_hw_devices xcu200_0] 0]
set_property PROBES.FILE {/root/system_wrapper.ltx} [get_hw_devices xcu200_0]
set_property FULL_PROBES.FILE {/root/system_wrapper.ltx} [get_hw_devices xcu200_0]
set_property PROGRAM.FILE {/root/download.bit} [get_hw_devices xcu200_0]
program_hw_devices [get_hw_devices xcu200_0]
close_hw_manager

$ chmod +x download.tcl 
$ vim prog_au200.sh
$ cat prog_au200.sh 
#!/bin/sh

source /opt/Xilinx/Vivado/2020.1/settings64.sh
echo "exit
" | vivado -mode tcl -source download.tcl

$ chmod +x prog_au200.sh 
$ ./prog_au200.sh 

****** Vivado v2020.1 (64-bit)
  **** SW Build 2902540 on Wed May 27 19:54:35 MDT 2020
  **** IP Build 2902112 on Wed May 27 22:43:36 MDT 2020
    ** Copyright 1986-2020 Xilinx, Inc. All Rights Reserved.

source download.tcl
# open_hw_manager
# connect_hw_server -allow_non_jtag
INFO: [Labtools 27-2285] Connecting to hw_server url TCP:localhost:3121
INFO: [Labtools 27-3415] Connecting to cs_server url TCP:localhost:3042
INFO: [Labtools 27-3414] Connected to existing cs_server.
# current_hw_target [lindex [get_hw_targets] 0]
# set_property PARAM.FREQUENCY 15000000 [lindex [get_hw_targets] 0]
# open_hw_target
INFO: [Labtoolstcl 44-466] Opening hw_target localhost:3121/xilinx_tcf/Xilinx/2130069AF04JA
# current_hw_device [get_hw_devices xcu200_0]
# refresh_hw_device -update_hw_probes false [lindex [get_hw_devices xcu200_0] 0]
INFO: [Labtools 27-2302] Device xcu200 (JTAG device index = 0) is programmed with a design that has 2 ILA core(s).
# set_property PROBES.FILE {/root/system_wrapper.ltx} [get_hw_devices xcu200_0]
# set_property FULL_PROBES.FILE {/root/system_wrapper.ltx} [get_hw_devices xcu200_0]
# set_property PROGRAM.FILE {/root/download.bit} [get_hw_devices xcu200_0]
# program_hw_devices [get_hw_devices xcu200_0]
INFO: [Labtools 27-3164] End of startup status: HIGH
program_hw_devices: Time (s): cpu = 00:00:18 ; elapsed = 00:00:18 . Memory (MB): peak = 2943.668 ; gain = 9.004 ; free physical = 56181 ; free virtual = 90974
# close_hw_manager
****** Webtalk v2020.1 (64-bit)
  **** SW Build 2902540 on Wed May 27 19:54:35 MDT 2020
  **** IP Build 2902112 on Wed May 27 22:43:36 MDT 2020
    ** Copyright 1986-2020 Xilinx, Inc. All Rights Reserved.

source /root/.Xil/Vivado-31363-localhost.localdomain/webtalk/labtool_webtalk.tcl -notrace
INFO: [Common 17-206] Exiting Webtalk at Fri Aug 13 21:23:51 2021...
exit
INFO: [Common 17-206] Exiting Vivado at Fri Aug 13 21:23:51 2021...
```

# XSCT
使用xsct执行u-boot，如果出现`Memory write error at 0x0. MMU page translation fault`，对PS复位引脚按一下，即可继续dow。

## zynq
```bash
# zynq
xsct% connect
xsct% source ps7_init.tcl
xsct% targets -set -filter {name =~ "APU"}
xsct% ps7_init
xsct% ps7_post_config
xsct% targets -set -filter {name =~ "ARM Cortex-A9 MPCore #0"}
xsct% dow -data BOOT.BIN 0x08000000
xsct% dow u-boot.elf
xsct% con
xsct% dow zynq_fsbl.elf 
xsct% con
```

## zynqmp
```bash
xsct% connect
xsct% rst -system
xsct% fpga C:/dog/tftp/mwt.bit # 注意文件路径分隔符是/，和Windows默认不一样
xsct% targets -set -filter {name =~ "PSU"}
xsct% mwr 0xffca0038 0x1FF
xsct% targets 13 # targets -set -filter {name =~ "MicroBlaze PMU"}
xsct% dow zynqmp_pmufw.elf
xsct% con
xsct% targets 9
xsct% rst -processor
xsct% dow zynqmp_fsbl_pcie.elf
xsct% con
xsct% dow u-boot.elf
xsct% dow bl31.elf
xsct% con

The correct load order is:
1. Load and run pmufw.elf into PMU ( You should "open" PMU, it's not visible with "targets" command after board reset).
2. Switch to A53-0, reset it,  download and run fsbl. At this step you should see something in console.
3. Download, but not run, u-boot.elf
4. Download and run bl31.elf. At this step, you should see atf notices on console then execution should automatically transfer to u-boot and you should see u-boot output on the console.
Check that U-Boot messages are correct according your hardware (especially memory size)
If these are correct, try to download and start FIT image
```

## microblaze
```bash
xsct% target
  1  APU
     2  ARM Cortex-A9 MPCore #0 (Running)
     3  ARM Cortex-A9 MPCore #1 (Running)
  4* xc7z045
     5  MicroBlaze Debug Module at USER2
        6  MicroBlaze #0 (Running)
     7  Legacy Debug Hub
        8  JTAG2AXI
xsct% target 6    
xsct% jtagterminal # 打开jtag uart
xsct% rst -processor
xsct% dow C:/dog/program/sdk2018.2/nvme_mb_zc706/Debug/nvme_mb_zc706.elf # 注意文件路径分隔符是/，和Windows默认不一样
xsct% con
```
如果Zynq reboot重新加载了bit，就需要关掉重新打开jtagterminal，
![383](https://img-blog.csdnimg.cn/20201228162336917.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
更新elf固件脚本，
```bash
$ exec updatemem -force -meminfo /media/qe/SN550/project/au200/au200.runs/impl_1/system_wrapper.mmi -bit /media/qe/SN550/project/au200/au200.runs/impl_1/system_wrapper.bit -data /media/qe/SN550/project/vitis/mppr/Debug/mppr.elf -proc system_i/microblaze_0 -out /media/qe/SN550/project/au200/au200.runs/impl_1/temp1.bit 
$ exec updatemem -force -meminfo /media/qe/SN550/project/au200/au200.runs/impl_1/system_wrapper.mmi -bit /media/qe/SN550/project/au200/au200.runs/impl_1/temp1.bit -data /media/qe/SN550/project/au200/au200.srcs/sources_1/bd/system/ip/system_cms_subsystem_0_0/fw/cms.elf -proc system_i/cms_subsystem_0/inst/shell_cmc_subsystem/inst/microblaze_cmc -out /media/qe/SN550/project/au200/au200.runs/impl_1/download.bit 
$ exec rm /media/qe/SN550/project/au200/au200.runs/impl_1/temp1.bit 
```
或，这个验证不行，本身我的2021.1出bug了，` Associate ELF Files`UI操作失败，
```bash
get_files *.elf
/media/qe/SN550/project/au200/au200.srcs/sources_1/bd/system/ip/system_microblaze_0_0/data/mb_bootloop_le.elf /media/qe/SN550/project/au200/au200.srcs/sources_1/bd/system/ip/system_cms_subsystem_0_0/bd_1/ip/ip_23/data/mb_bootloop_le.elf /media/qe/SN550/project/au200/au200.srcs/sources_1/bd/system/ip/system_cms_subsystem_0_0/fw/cms.elf /media/qe/SN550/project/vitis/mppr/Debug/mppr.elf

set_property SCOPED_TO_CELLS microblaze_0 [get_files *mppr.elf]
set_property SCOPED_TO_REF system [get_files *mppr.elf]
report_property [get_files *mppr.elf]
Property                Type     Read-only  Value
CLASS                   string   true       file
CORE_CONTAINER          string   true       
FILE_TYPE               enum     false      ELF
IS_AVAILABLE            bool     true       1
IS_ENABLED              bool     false      1
IS_GENERATED            bool     true       0
NAME                    string   true       /media/qe/SN550/project/vitis/mppr/Debug/mppr.elf
NEEDS_REFRESH           bool     true       0
PATH_MODE               enum     false      RelativeFirst
SCOPED_TO_CELLS         string*  false      microblaze_0
SCOPED_TO_REF           string   false      system
USED_IN                 string*  false      implementation
USED_IN_IMPLEMENTATION  bool     false      1
USED_IN_SIMULATION      bool     false      0
```

