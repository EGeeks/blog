# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [SDRAM的地址映射方式BRC(Bank Row Column)和RBC(Row Bank Column)](https://blog.csdn.net/zhu8920253/article/details/41595189)

# CAS latency（CL）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190301155214759.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 地址
MT40A2G4，MT40A1G8，MT40A512M16的地址分配，
![417](https://img-blog.csdnimg.cn/8ecb737bc4b3447ea2e4085cc6a0edec.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiJ6YGN54yq,size_20,color_FFFFFF,t_70,g_se,x_16)
MT40A512M8，MT40A256M16的地址分配，
![418](https://img-blog.csdnimg.cn/2ec3e55aaaa54a3fb023af0221acbbb0.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiJ6YGN54yq,size_20,color_FFFFFF,t_70,g_se,x_16)
对比MT40A512M16和MT40A256M16可以发现，仅仅MT40A512M16的Row addressing多1位。

