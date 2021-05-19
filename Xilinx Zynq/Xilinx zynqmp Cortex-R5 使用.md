# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考

> [OpenAMP](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841718/OpenAMP)
> ug1186
> [zcu102 petalinux+RPU 当linux启动后，RPU不工作了，请教要如何设置](https://forums.xilinx.com/t5/%E5%B5%8C%E5%85%A5%E5%BC%8F%E7%A1%AC%E4%BB%B6%E5%BC%80%E5%8F%91-MPSoC-Zynq-7000/zcu102-petalinux-RPU-%E5%BD%93linux%E5%90%AF%E5%8A%A8%E5%90%8E-RPU%E4%B8%8D%E5%B7%A5%E4%BD%9C%E4%BA%86-%E8%AF%B7%E6%95%99%E8%A6%81%E5%A6%82%E4%BD%95%E8%AE%BE%E7%BD%AE/m-p/1002756#M2275)
> [AR# 67506 Zynq UltraScale+ MPSoC： 当 R5 是引导主机时，FSBL 从 R5 引导并加载一个从 A53 主机启动的用户应用程序。但 A53 无法引导。](https://china.xilinx.com/support/answers/67506.html)
> [A53中运行的u-boot可以加载RPU standalone application么?](https://mp.weixin.qq.com/s?__biz=Mzg3NDAxNzU1MA==&mid=2247488332&idx=1&sn=b0cab1f6147ad8a0360e741fdf1493ca&chksm=ced6752df9a1fc3b000ac94392678c4b412f43f50247b7c5be89585c91f331365aed6f64eb79&scene=178&cur_album_id=1471618535444512769#rd)

# RPU串口
RPU使用的串口在Linux中不能使能，否则RPU无打印，可以在Linux system-user.dtsi 文件中disable掉留给RPU使用的串口。
```c
&uart1 {
    status = "disabled";
}
```



