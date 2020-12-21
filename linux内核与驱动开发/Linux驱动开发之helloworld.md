# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 开发环境搭建
Ubuntu安装软件包，

# 驱动代码
```c

```

# Makefile
单独编译驱动，
```shell
ifneq ($(KERNELRELEASE),)
obj-m:=helloworld.o
else
KERNELDIR:=/lib/modules/$(shell uname -r)/build
PWD:=$(shell pwd)
#modules:
#	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules
#modules_install:
#	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules_install
#default:
all:
	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules
.PHONY: clean
clean:
	rm -rf *.o *.mod.c *.mod.o *.ko *.order *symvers
endif
```

# 测试
