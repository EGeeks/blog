# yocto
> [yocto官网](https://www.yoctoproject.org/)
> [Yocto详解](https://blog.csdn.net/qq_28992301/article/details/52872209)
> [Yocto的使用实例](https://blog.csdn.net/qq_28992301/article/details/52922314)
> [Yocto项目实践](https://blog.csdn.net/sy373466062/column/info/yocto-project)
> [Yocto实用技巧](https://blog.csdn.net/sy373466062/column/info/yocto)
> [（一）Yocto的介绍](https://blog.csdn.net/yangteng0210/article/details/81566950)
> [（六）yocto SDK的生成及eclipse配置](https://blog.csdn.net/yangteng0210/article/details/82587256)
> [Yocto 实用技巧](https://www.taterli.com/4478/)
> [IMX6Q-Yocto环境搭建](https://www.jianshu.com/p/f6e0debb5e1f)
> [I.MX6 Linux yocto开发环境搭建](https://blog.csdn.net/u013007904/article/details/80936933)
> [imx6开发环境搭建之yocto全记录(L4.1.15_2.0.0)](https://blog.csdn.net/cking0906/article/details/76099025)
> [Yocto设置/修改root用户密码](https://blog.csdn.net/weixin_43045713/article/details/102628523)
> [How to set a root password in yocto which has dollar($) symbol](https://stackoverflow.com/questions/62649271/how-to-set-a-root-password-in-yocto-which-has-dollar-symbol)
> [OpenBmc开发5：bitbake介绍与使用](https://blog.csdn.net/qq_34160841/article/details/105163958)
> [OpenBmc开发6：创建新layer（适配自己的board）](https://blog.csdn.net/qq_34160841/article/details/106185242)
> [Useful bitbake commands](https://community.nxp.com/t5/i-MX-Processors-Knowledge-Base/Useful-bitbake-commands/ta-p/1128559)

# recipes
默认的recipes文件路径
```
     FILES_${PN} = "${bindir}/* ${sbindir}/* ${libexecdir}/* ${libdir}/lib*${SOLIBS} \
                 ${sysconfdir} ${sharedstatedir} ${localstatedir} \
                 ${base_bindir}/* ${base_sbindir}/* \
                 ${base_libdir}/*${SOLIBS} \
                 ${base_prefix}/lib/udev/rules.d ${prefix}/lib/udev/rules.d \
                 ${datadir}/${BPN} ${libdir}/${BPN}/* \
                 ${datadir}/pixmaps ${datadir}/applications \
                 ${datadir}/idl ${datadir}/omf ${datadir}/sounds \
                 ${libdir}/bonobo/servers"
```

# bitbake

# devtool

