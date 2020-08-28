# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# C/C++混编undefined reference
调用c文件中的函数出现undefined reference to `xxxxx()'。这是C/C++兼容问题，在C的头文件的相关函数声明添加extern "C"即可解决，
```c
#ifdef __cplusplus 
extern "C" {
#endif 

#ifdef __cplusplus 
}
#endif
```

# error: 'class QGuiApplication' has no member named 'setStyleSheet'

代码如下，设置全局CSS，之前一直用的好好的，后来突然编译不过，报上面的错误，Qt5.9
```c
        QFile file(QString(":/image/%1.css").arg(styleName));
        file.open(QFile::ReadOnly);
        QString qss = QLatin1String(file.readAll());
        qApp->setStyleSheet(qss);
```
查看帮助，QCoreApplication是没有这个方法的，但是QApplication有，
> A global pointer referring to the unique application object. It is equivalent to QCoreApplication::instance(), but cast as a QApplication pointer, so only valid when the unique application object is a QApplication.
See also QCoreApplication::instance() and qGuiApp. 

这里也没有用强制转换，添加QApplication头文件，编译通过了


