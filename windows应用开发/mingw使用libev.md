# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [libev Document](https://metacpan.org/pod/distribution/EV/libev/ev.pod)
> [MinGW 编译libevent](https://blog.csdn.net/wo_Niu123/article/details/85719737)
> [用Libev作TCP server的问题](https://bbs.csdn.net/topics/390315012)
> [Libev on Windows](https://stackoverflow.com/questions/8042796/libev-on-windows)
> [libev源码解析——总览](https://cloud.tencent.com/developer/article/1383770)
> [重写 libev 的 EV_WIN32_HANDLE_TO_FD](https://www.cnblogs.com/JesseFang/p/4682305.html)
> [libev学习之ev_run](https://www.cnblogs.com/xiangshancuizhu/p/3250558.html)
> [Libev轻网络库 源码浅析](http://chenzhenianqing.com/articles/1051.html)
> [libev源码解读](https://blog.csdn.net/drdairen/article/details/53785447)
> [网络库libevent、libev、libuv对比](https://blog.csdn.net/lijinqi1987/article/details/71214974)

# 说明
libev对windows支持不好，当然也可能是对mingw支持不好，官方是支持windows的，libuv貌似对windows的支持更好，libuv封装了libev，linux下用libev实现，Windows下用IOCP实现。

# 方法
mingw下编译libev，打开msys，
```shell
make clean
./configure --prefix=$cur_path/mingw-static-530_32 CFLAGS=-static --enable-static LDFLAGS=-static --disable-shared
make 
make install
```
qt引用，测试初始化会卡死，
```makefile
INCLUDEPATH += libev/include
LIBS += ..\recorder-server\libev\lib\libev.a
```
现采用直接在工程中使用源文件的方式，添加文件，
```c
ev.c
ev.h
ev_select.c
ev_win32.c
ev_vars.h
ev_wrap.h
```
在前4个文件文件头部添加下面三个宏，
```c
#define EV_STANDALONE              /* keeps ev from requiring config.h */
#define EV_USE_SELECT 1
#define EV_SELECT_IS_WINSOCKET 1   /* configure libev for windows select */
```
修改代码，这两个assert注释，
```c
ev.c:        line2134 assert (("libev: only socket fds supported in this configuration", ret == 0));
ev_select.c: line85   assert (("libev: fd >= FD_SETSIZE passed to fd_set-based select backend", fd < FD_SETSIZE));
```
建一个线程跑EV_RUN，
```c
void *jsonrpc_cmd_thread_func(void * param)
{
    struct jrpc_server *server = (struct jrpc_server *)param;
    printf("%s line%d thread enter\n", __FUNCTION__, __LINE__);

    EV_RUN(server->loop, 0);
	printf("%s line%d thread exit\n", __FUNCTION__, __LINE__);
    EV_BREAK(server->loop, EVBREAK_ALL);
    
    return 0;
}

ret = pthread_create(&tid, NULL, jsonrpc_cmd_thread_func, server);
if(ret != 0)
{
    printf("%s failed\n", __FUNCTION__);
    return ret;
}
```
现在可以编译过，我测试，客户端连接上之后，发送数据，服务端返回数据，然后`EV_RUN`就退出了。。。无法再处理数据，有在mingw下成功使用libev的，不吝赐教。。。
