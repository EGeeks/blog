# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [STM32CubeIDE属于一站式工具，本文带你体验它的强大](https://blog.csdn.net/ybhuangfugui/article/details/89702356)
> [第一个STM32CubeIDE项目](https://blog.csdn.net/weixin_44481398/article/details/93293881)
> [STM32CubeIDE使用记录](https://blog.csdn.net/weixin_42415539/article/details/90795405)
> [STM32CubeIDE使用笔记（03）：使用ST-LINK调试程序](https://blog.csdn.net/Naisu_kun/article/details/97393547)

# 安装
STM32终于出了这样一款工具，之前也出过开源的eclipse开发工具，但这次帮你打包了一站式的，更方便了，[点击此处下载](https://my.st.com/content/my_st_com/en/products/development-tools/software-development-tools/stm32-software-development-tools/stm32-ides/stm32cubeide.license=1568939618757.product=STM32CubeIDE-Win.version=1.0.2.html)，下载需要有自己的账号，官网下载，安装一路默认就可以了。
# 使用
新建stm32工程：`文件 > 新建 > STM32 Project`，弹出下面的向导，通过1，2，3等过滤选项，选出正点原子战舰V3对应的stm32f103ze系列芯片，点击`Next`，
![122](https://img-blog.csdnimg.cn/20191004221427361.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
下面一路默认选项，如果你是第一次使用，会自动开始下载相应的固件包，我这自动下载的过程被打断，弹出一个对话框说代码生成失败，现在尝试手动来操作，点击`Help > Manage embedded software update`，这个菜单只有在`*.ioc`的CubeMX文件打开的情况才可以点击，弹出一个对话框，下载f1系列的包，可以看到下面有一个`From Local`，可以离线导入的，如果网实在太差，就别处下好导入，我最后还是离线导入的，官网下载stm32cubef1，是的，英文官网的速度飞起，比国内的网站还快，
![123](https://img-blog.csdnimg.cn/20191004222411389.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
现在点击`Project > Generate Code`，
# 问题
在导入stm32f4的离线包时，官网上只有`en.STM32Cube_FW_F4_V1.24.0`外加一个补丁包`en.patch_cubefw_f4.zip`，
![124](https://img-blog.csdnimg.cn/20191007214001502.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)而这个补丁包是没法导入的，报错，尝试手动打补丁，stm32的包文件都安装在下面的路径`C:\Users\***\STM32Cube\Repository\STM32Cube_FW_F4_V1.24.0`，手动把补丁文件解压覆盖试一下，重启一下软件看一下，成功了。
