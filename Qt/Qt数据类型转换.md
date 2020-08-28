# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# QString转char*
```c
QByteArray ba;
ba = pDevWgtLeManualScanIp->text().toUtf8(); 
sprintf(cfg->manualScanIp, "%s", ba.data());
```

