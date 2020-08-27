# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# tapip收发包接口
为了在tapip中嵌入netmap，必须完成下面三个接口，
```c
static struct netdev_ops netmap_if_ops = {
	.init = netmap_if_dev_init,
	.xmit = netmap_if_xmit,
	.exit = netmap_if_dev_exit,
};
```
init函数是初始化函数，xmit是发包函数，exit退出时被调用，参考我的博客，用netmap的接口完成这三个函数即可。
> [netmap用户层接口使用](https://blog.csdn.net/Zhu_Zhu_2009/article/details/80981522)

对于接收函数，由于用户态没有中断，所以通过线程轮询实现包的实时接收，在tapip的main函数会建立下面几个线程，其中netdev_interrupt就是用于接收的线程函数，
```c
void net_stack_run(void)
{
	/* create timer thread */
	threads[0] = newthread((pfunc_t)net_timer);
	dbg("thread 0: net_timer");
	/* tcp timer */
	threads[1] = newthread((pfunc_t)tcp_timer);
	dbg("thread 1: tcp_timer");
	/* create netdev thread */
	threads[2] = newthread((pfunc_t)netdev_interrupt);
	dbg("thread 2: netdev_interrupt");
	/* shell worker thread */
	threads[3] = newthread((pfunc_t)shell_worker);
	dbg("thread 3: shell worker");
	/* net shell runs! */
	shell_master(NULL);
}
```
netdev_interrupt的实现，
```c
void netdev_interrupt(void)
{
	veth_poll();
}
```
所以在这里实现netmap的poll即可，参考那篇博客。

# 编译错误修改

tapip定义和标准协议栈冲突，netmap_user.h中删除头文件，
```c
//#include <sys/socket.h>		/* apple needs sockaddr */
//#include <net/if.h>		/* IFNAMSIZ */
```
增加IFNAMSIZ=16定义，IFNAMSIZ值可通过下面的语句获得（参考博客[编译时打印宏内容](https://blog.csdn.net/wlr_tang/article/details/21778587)）
```c
#define __PRINT_MACRO(x) #x
#define PRINT_MACRO(x) #x"="__PRINT_MACRO(x)
#pragma message(PRINT_MACRO(IFNAMSIZ))
```
增加头文件定义，解决类型找不到错误
```c
#include <sys/types.h>

../include/netmap.h:298:17: error: field ‘ts’ has incomplete type
  struct timeval ts;  /* (k) time of last *sync() */
                 ^~
../include/netmap.h:367:8: error: unknown type name ‘ssize_t’
  const ssize_t ring_ofs[0];

```
修改net子文件夹下Makefile，
```bash
OBJS	= net.o netdev.o veth.o loop.o pkb.o tap.o netmap_if.o
```
