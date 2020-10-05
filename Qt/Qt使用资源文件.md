# qrc资源文件
项目右键菜单添加资源文件，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190620200034953.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
打开资源文件，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190620201151785.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
添加素材，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190620210340511.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 代码
采用`:/xxx`的格式，
```c
QListWidgetItem* pLwiShelf = new QListWidgetItem(QIcon(":/image/bullet_green.png"), tr("Shelf"));
pLwLeftPannel->addItem(pLwiShelf);
```

