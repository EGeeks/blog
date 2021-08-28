# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [Utilizing PS memory to execute Microblaze application on Zynq Ultrascale](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841793/Utilizing+PS+memory+to+execute+Microblaze+application+on+Zynq+Ultrascale)
> [聊一聊如何实现Xilinx Microblaze Bootloader](https://hellocode.blog.csdn.net/article/details/110678734)
> [xilinx vivado 烧录microblaze ](http://blog.chinaaet.com/lichenllin/p/5100056695)
> [关于MicroBlaze软核固化的方法](https://blog.csdn.net/qq_42712308/article/details/109319005)
> [Microblaze程序固化流程](https://blog.csdn.net/zhengshuo5444/article/details/107357806/)
> [【JokerのKintex7325】SDK程序从QSPI启动。](https://blog.csdn.net/natty715/article/details/104084681)
> [【JokerのKintex7325】SDK程序从QSPI启动过慢分析。](https://blog.csdn.net/natty715/article/details/104241415)
> [Understanding MEMDATA flow and how to manually create MMI file](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842458/Understanding+MEMDATA+flow+and+how+to+manually+create+MMI+file)

# 固化elf
Ubuntu18.04，Vivado2020.1，` Associate ELF Files`UI操作失败，
```bash
set_property SCOPED_TO_CELLS microblaze_0 [get_files *.elf]
set_property SCOPED_TO_REF design_1 [get_files *.elf]
```

# section `.stack' will not fit in region
SDK2018.2，报这个错是因为内存小了，改大之后还要手动改ld脚本，Xilinx不会自动更新，
```bash
mb-gcc -Wl,-T -Wl,../src/lscript.ld -L../../pcierc_wrapper_mb_bsp/microblaze_0/lib -mlittle-endian -mxl-barrel-shift -mxl-pattern-compare -mcpu=v10.0 -mno-xl-soft-mul -Wl,--no-relax -Wl,--gc-sections -o "nvme_mb_zc706.elf"  ./src/user/nr_micro_shell_commands.o ./src/user/user_app.o  ./src/shell/ansi.o ./src/shell/ansi_port.o ./src/shell/nr_micro_shell.o  ./src/hw/cache.o ./src/hw/debug.o ./src/hw/gpio.o ./src/hw/interrupt.o ./src/hw/ipc_hw_shm.o ./src/hw/nvme.o ./src/hw/pci.o ./src/hw/pci_auto.o ./src/hw/pcie_xilinx.o ./src/hw/shm_simple_mgt.o ./src/hw/timer.o  ./src/main.o ./src/platform.o   -Wl,--start-group,-lxil,-lgcc,-lc,--end-group
d:/xilinx/sdk/2018.2/gnu/microblaze/nt/bin/../lib/gcc/microblaze-xilinx-elf/7.2.0/../../../../microblaze-xilinx-elf/bin/ld.exe: nvme_mb_zc706.elf section `.stack' will not fit in region `microblaze_0_local_memory_ilmb_bram_if_cntlr_Mem_microblaze_0_local_memory_dlmb_bram_if_cntlr_Mem'
d:/xilinx/sdk/2018.2/gnu/microblaze/nt/bin/../lib/gcc/microblaze-xilinx-elf/7.2.0/../../../../microblaze-xilinx-elf/bin/ld.exe: region `microblaze_0_local_memory_ilmb_bram_if_cntlr_Mem_microblaze_0_local_memory_dlmb_bram_if_cntlr_Mem' overflowed by 72 bytes
collect2.exe: error: ld returned 1 exit status
make: *** [nvme_mb_zc706.elf] 错误 1
```

