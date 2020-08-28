# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

这里用Json文件来保存程序的配置信息。
# 参考
> [Qt 之 JSON 生成与解析](https://blog.csdn.net/liang19890820/article/details/52767153)
> [Qt中的JSON操作](https://blog.csdn.net/amnes1a/article/details/68957112)
> [Qt 判断文件或文件夹是否存在及创建文件夹](https://blog.csdn.net/lusirking/article/details/51644782)

# QJson
QJsonValue类封装了一个json格式中的值。
QJsonObject类封装了一个json对象。
QJsonArray类封装了一个json数组。
QJsonDocument类提供了读写json文档的方法。
# 写Json文件
```c
void MainWindow::SaveUiTcpClientCfg()
{
    tcpClientIp = pIpLeTcpClientServerIp->text();
    tcpClientPort = pLeTcpClientPort->text().toInt();

    QJsonObject json;
    json.insert("ip", tcpClientIp);
    json.insert("port", tcpClientPort);

    QJsonDocument jsonDoc;
    jsonDoc.setObject(json);
    QByteArray ba = jsonDoc.toJson();

    QString dirPath = QString("%1/config").arg(QDir::currentPath());
    QDir dir(dirPath);
    if(!dir.exists())
    {
       bool ok = dir.mkdir(dirPath);//只创建一级子目录，即必须保证上级目录存在
       if (!ok)
       {
           qDebug() << "create dir config failed";
           return;
       }
    }

    QFile file("./config/tcpclient.json");
    if(!file.open(QIODevice::WriteOnly))
    {
        qDebug() << "open json file writeonly failed";
        return;
    }
    file.write(ba);
    file.close();
}
```
# 读Json文件
```c
void MainWindow::LoadUiTcpClientCfg()
{
    tcpClientIp = "127.0.0.1";
    tcpClientPort = 8000;

    QFile file("./config/tcpclient.json");
    if(!file.open(QIODevice::ReadOnly))
    {
        qDebug() << "open json file readonly failed save default";
        SaveUiTcpClientCfg();
        return;
    }
    QByteArray ba = file.readAll();
    QJsonParseError jsonError;
    QJsonDocument jsonDoc = QJsonDocument::fromJson(ba, &jsonError);  // 转化为 JSON 文档
    if (!jsonDoc.isNull() && (jsonError.error == QJsonParseError::NoError))
    {  // 解析未发生错误
        if (jsonDoc.isObject())
        { // JSON 文档为对象
            QJsonObject object = jsonDoc.object();  // 转化为对象
            if (object.contains("ip"))
            {  // 包含指定的 key
                QJsonValue value = object.value("ip");  // 获取指定 key 对应的 value
                if (value.isString())
                {  // 判断 value 是否为字符串
                    tcpClientIp = value.toString();  // 将 value 转化为字符串
                    qDebug() << "tcp client ip: " << tcpClientIp;
                }
            }
            if (object.contains("port"))
            {
                QJsonValue value = object.value("port");
                if (value.isDouble())
                {
                    tcpClientPort = value.toVariant().toInt();
                    qDebug() << "tcp client port: " << tcpClientPort;
                }
            }
        }
    }
}
```
