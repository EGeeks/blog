# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [QCustomplot使用分享(三) 图](https://www.cnblogs.com/swarmbees/p/6057798.html)
> [QCustomplot使用分享(五) 布局](https://www.cnblogs.com/swarmbees/p/6058942.html)
> [QCustomplot官网](https://www.qcustomplot.com/)
> [QCustomPlot 使用整理](https://www.cnblogs.com/pied/p/5164000.html)
> [Qt之qcustomplot背景色改变](https://blog.csdn.net/u014252478/article/details/79928433)

# 改变颜色
改变背景色，坐标轴的颜色，坐标轴名称的颜色，坐标轴上Tick的颜色，
```c
pCustomPlotPower->xAxis->setBasePen(pen);
pCustomPlotPower->xAxis->setTickPen(pen);
pCustomPlotPower->xAxis->setSubTickPen(pen);
pCustomPlotPower->xAxis->setLabelColor(color);
pCustomPlotPower->xAxis->setTickLabelColor(color);
QBrush brush(QColor(0x44, 0x44, 0x44));
pCustomPlotPower->setBackground(brush);
```
示例，
![145](https://img-blog.csdnimg.cn/20190703105754714.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

