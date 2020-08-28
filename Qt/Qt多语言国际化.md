# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [Qt 之国际化](https://blog.csdn.net/liang19890820/article/details/50276673)
> [实现多国语言的动态切换](https://blog.csdn.net/wang_hufeng/article/details/39676085)
> [Qt 国际化之二：多国语界面动态切换的实现](https://www.cnblogs.com/lvdongjie/p/4053008.html)

# 制作翻译文件
打开pro文件添加，
```c
TRANSLATIONS += \
    image/shelf_zh.ts \
    image/shelf_en.ts
```
我这里添加了中文和英文，我的代码里都是英文的，点击菜单栏 "工具"-> "外部" -> "Qt语言家" -> "更新翻译（lupdate）",将生成语言文件
![这里写图片描述](https://img-blog.csdn.net/2018090419132510?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
打开Qt Linguist，打开之前生成的文件，或者用其他文本编辑器编辑，
![这里写图片描述](https://img-blog.csdn.net/20180904161754759?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
编辑完之后，发布语言文件，使用Qt Linguist或者Qt Creator发布。
![这里写图片描述](https://img-blog.csdn.net/20180904161957245?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

# 加载
上代码，一定要在widget之前加载translator，否则没效果。
```c
void trHelper::trHelperInit()
{
    trHelper::InitTrCfg();
    trHelper::LoadTrCfg();
    trHelper::trHelperSetLang(lang);
    //qApp->installTranslator(&tl);
}

void trHelper::trHelperSetLang(QLocale::Language l)
{
    qApp->removeTranslator(&tl);
    if (l == TR_LANG_EN)
    {
        //qDebug() << "trHelperSetLang en";
        tl.load(QString(":/image/shelf_en"));
    }
    else if (l == TR_LANG_CH)
    {
        //qDebug() << "trHelperSetLang zh";
        tl.load(QString(":/image/shelf_zh"));
    }
    else
    {
        return;
    }
    qApp->installTranslator(&tl);
    lang = l;
    trHelper::SaveTrCfg();
}
```
关于动态加载，这里的方法就不是很方便了，没有ui文件的方式方便。
