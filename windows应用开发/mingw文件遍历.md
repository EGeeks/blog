# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [linux C语言遍历文件夹所有文件](https://blog.csdn.net/rubikchen/article/details/78218388)
> [windows下编程获取磁盘(分区)使用情况](https://www.cnblogs.com/findumars/p/6253630.html)
> [windows下使用C/C++怎么遍历目录并读取目录下的文件列表？](https://segmentfault.com/q/1010000002494724)

# 方法
Qt中可用，
```cpp
[static] QFileInfoList QDir::drives()
Returns a list of the root directories on this system.
On Windows this returns a list of QFileInfo objects containing "C:/", "D:/", etc. On other operating systems, it returns a list containing just one root directory (i.e. "/").
See also root() and rootPath().
```
使用原生API，获取本地驱动器，然后遍历文件，
```c
int local_file_ls(char *path, cJSON *json)
{
    struct dirent* ent;
    DIR* dir;

    if (!path)
        return -1;

#if defined (WIN32)
    if (strcmp(path, "/") == 0 || strcmp(path, "\\") == 0) {
        DWORD dwSize = MAX_PATH;
        char szLogicalDrives[MAX_PATH] = {0};
        //获取逻辑驱动器号字符串
        DWORD dwResult = GetLogicalDriveStringsA(dwSize, szLogicalDrives);
        //处理获取到的结果
        if (dwResult > 0 && dwResult <= MAX_PATH) {
            char* szSingleDrive = szLogicalDrives;  //从缓冲区起始地址开始
            while(*szSingleDrive) {
                printf("Drive: %s\n", szSingleDrive);   //输出单个驱动器的驱动器号
                // 获取下一个驱动器号起始地址
                szSingleDrive += strlen(szSingleDrive) + 1;
            }
        }
        goto ret;
    }
#endif

    dir = opendir(path);
    while ((ent = readdir(dir)) != NULL)
    {
        printf("%s\n", ent->d_name);
    }

ret:
    return 0;
}
```

