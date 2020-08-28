# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# Vivado HLS 2019.2导出的IP在Vivado中例化丢失管脚
Vivado HLS 2019.2有bug，导出某些ip的时候（有些IP是对的），对应的`component.xml`丢失管脚，其实`.v/.vhd`的文件是有这个管脚的，导致在Vivado中综合报错，你可以手动更改`component.xml`文件。2018.2的版本没有这个问题。
