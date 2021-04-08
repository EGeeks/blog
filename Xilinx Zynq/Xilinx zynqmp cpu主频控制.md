# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [CPU frequency scaling](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841831/CPU+frequency+scaling)
> [Common Clock Framework](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841636/Common+Clock+Framework)
> [CPU Power Management](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/1417117726/CPU+Power+Management)
> [Zynq UltraScale＋ MPSoC Power Management](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841813/Zynq+UltraScale+MPSoC+Power+Management)

# 内核启动打印
```shell
[    2.926159] cpufreq: cpufreq_online: CPU0: Running at unlisted freq: 1333333 KHz
[    2.933381] cpu cpu0: dev_pm_opp_set_rate: failed to find current OPP for freq 1333333320 (-34)
[    2.942153] cpufreq: cpufreq_online: CPU0: Unlisted initial frequency changed to: 1199999 KHz
[    2.950512] cpu cpu0: dev_pm_opp_set_rate: failed to find current OPP for freq 1333333320 (-34)
```

