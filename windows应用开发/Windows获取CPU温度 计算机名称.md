# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [官网手册](https://openlibsys.org/manual/)
> [温度读取vc++获取cpu温度](https://blog.csdn.net/weixin_33713503/article/details/85563472)
> [获取cpu温度](https://blog.csdn.net/u013837488/article/details/80390319)
> [C#利用开源库OpenHardwareMonitor获取CPU或显卡温度、使用率、时钟频率](https://blog.csdn.net/qq_38588710/article/details/78557795)
> [Open Hardware Monitor](https://openhardwaremonitor.org/)
> [BLHWScaner](https://github.com/BurnellLiu/BLHWScaner)
> [VC++ 得到计算机名和用户名 GetComputerName GetUserName](https://blog.csdn.net/morewindows/article/details/8659417)

# 温度
通过WinRing0，在内核态直接读取CPU寄存器，获取温度，Open Hardware Monitor也使用了WinRing0，

# 计算机名称
```c
    char  szBuffer[256];
    DWORD dwNameLen;

    dwNameLen = 256;
    if (!GetComputerNameA(szBuffer, &dwNameLen))
        printf("GetComputerNameA failed %d\n", (int)GetLastError());
    else {
        snprintf(dev->name, RECORDER_DEVICE_MAX_NAME_LEN, "%s", szBuffer);
        if (json)
            cJSON_AddStringToObject(json, "name", szBuffer);
    }
```


