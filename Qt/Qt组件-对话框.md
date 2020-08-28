# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [Qt之标准对话框（文件对话框）](https://blog.csdn.net/zl_95520/article/details/82687556)
> [QT之文件对话框](https://www.cnblogs.com/ylan2009/archive/2012/05/06/2486606.html)
> [QT打开和保存文件对话框](https://www.cnblogs.com/liuyunfeifei/archive/2013/02/26/2933411.html)
> [如何修改Qt标准对话框的文字(例如,英文改成中文)](https://blog.csdn.net/libaineu2004/article/details/19030129)

QT自带的内建标准对话框有：`QFileDialog`、`QFontDialog`、`QColorDialog`、`QMessageBox`。

# 文件对话框
使用`QFileDialog::getOpenFileName`，`QFileDialog::getOpenFileNames`，`QFileDialog::getSaveFileName`会调用原生的操作系统的对话框，而使用`QFileDialog`使用的是Qt自绘的对话框。
```cpp
QFileDialog *fileDialog = new QFileDialog(this);
fileDialog->setWindowTitle(tr("Open/Save Ucode Configuration File"));
fileDialog->setDirectory(".");
fileDialog->setNameFilter("Json Files (*.json)");
if(fileDialog->exec() == QDialog::Accepted) {
    QString path = fileDialog->selectedFiles()[0];
    qDebug() << path;
}
```

