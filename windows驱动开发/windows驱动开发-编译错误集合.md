# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# WDK7600 wdmguid.h redefinition
编译输出，
```shell
1>c:\winddk\7600.16385.1\inc\ddk\wdmguid.h(27): error C2374: 'GUID_HWPROFILE_QUERY_CHANGE' : redefinition; multiple initialization
```
同一个头文件中不能包含wdmguid.h两次，一个头文件包含另一个头文件，另一个头文件又会包含头文件，仔细检查。

#  warning C4311: “类型强制转换”: 从“PVOID”到“ULONG”的指针截断
使用下面的方式，用ULONG_PTR做一个过渡，
```c
PVOID SystemArgument1;
MessageId = (ULONG)((ULONG_PTR)SystemArgument1);
```

