# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 方法
新建工程，一路默认到选择器件，这里根据项目选择自己的芯片，
![186](https://img-blog.csdnimg.cn/20190821100527970.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
新建Block Design，
![187](https://img-blog.csdnimg.cn/2019082110074297.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
添加IP，
![188](https://img-blog.csdnimg.cn/20190821100902296.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
这里先添加zynq，
![189](https://img-blog.csdnimg.cn/20190821100943938.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
点击Run Block Automation，
![190](https://img-blog.csdnimg.cn/20190821101136644.png)
双击zynq的图标，配置CPU和外设，首先是时钟，我的是33.33MHz，FCLK是PS提供给PL用的时钟，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190821102005650.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
配置DDR，这里只配了Memory Part参数，我们假定PCB Layout是完美的，
![192](https://img-blog.csdnimg.cn/20190821102437626.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
配置外设，
![193](https://img-blog.csdnimg.cn/20190821141636985.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
点击OK，确定，创建顶层文件，
![194](https://img-blog.csdnimg.cn/20190821141748901.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
点击Run Synthesis，进入Synthesis Design，添加约束，编译即可。
外接了axi模块，
![195](https://img-blog.csdnimg.cn/20190821161115867.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
