# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [Qt嵌入浏览器（二）——QWebChannel实现与页面的通信](https://www.jianshu.com/p/e25646ee2977)
> [Qt的QWebChannel和JS、HTML通信/交互驱动百度地图](https://blog.csdn.net/u014281970/article/details/82110446)

# 方法
本来打算利用Qt实现Websocket的JsonRPC通信，后来看到了官方提供了QWebChannel类，浏览器的JS可直接调用QWebChannel中注册的类，很方便，记录一下官方Demo的使用，首先初始化流程，新建一个Websocket服务端，
```cpp
pShelfWebSocketServer = new QWebSocketServer("QWebChannel Jsonrpc Server",
                                    QWebSocketServer::NonSecureMode, this);
if (!pShelfWebSocketServer->listen(QHostAddress::Any, webSocketServerPort))
{
    myHelper::ShowMessageBoxInfo(tr("Web socket server is error"));
    qDebug("Failed to open web socket server");
    delete pShelfWebSocketServer;
    pShelfWebSocketServer = NULL;
    return;
}
// wrap WebSocket clients in QWebChannelAbstractTransport objects
pShelfWebSocketClientWrapper = new WebSocketClientWrapper(pShelfWebSocketServer, this);
// setup the channel
pShelfWebChannel = new QWebChannel(this);
connect(pShelfWebSocketClientWrapper, &WebSocketClientWrapper::clientConnected,
                 pShelfWebChannel, &QWebChannel::connectTo);
pShelfWebChannel->registerObject("shelfjs", &webChannelProc);
```
WebSocketClientWrapper在QWebSocketServer监听到一个连接之后，将WebSocketTransport对象发射到QWebChannel执行connectTo函数，告诉QWebChannel已经建立连接，
```cpp
/*!
    Construct the client wrapper with the given parent.

    All clients connecting to the QWebSocketServer will be automatically wrapped
    in WebSocketTransport objects.
*/
WebSocketClientWrapper::WebSocketClientWrapper(QWebSocketServer *server, QObject *parent)
    : QObject(parent)
    , m_server(server)
{
    connect(server, &QWebSocketServer::newConnection,
            this, &WebSocketClientWrapper::handleNewConnection);
}

/*!
    Wrap an incoming WebSocket connection in a WebSocketTransport object.
*/
void WebSocketClientWrapper::handleNewConnection()
{
    emit clientConnected(new WebSocketTransport(m_server->nextPendingConnection()));
}
```
WebSocketTransport负责将函数的输入输出序列化成Json格式，继承了`QWebChannelAbstractTransport`对象，
```cpp
class WebSocketTransport : public QWebChannelAbstractTransport
{
    Q_OBJECT
public:
    explicit WebSocketTransport(QWebSocket *socket);
    virtual ~WebSocketTransport();

    void sendMessage(const QJsonObject &message) Q_DECL_OVERRIDE;

private Q_SLOTS:
    void textMessageReceived(const QString &message);

private:
    QWebSocket *m_socket;
};

/*!
    Serialize the JSON message and send it as a text message via the WebSocket to the client.
*/
void WebSocketTransport::sendMessage(const QJsonObject &message)
{
    QJsonDocument doc(message);
    m_socket->sendTextMessage(QString::fromUtf8(doc.toJson(QJsonDocument::Compact)));
}

/*!
    Deserialize the stringified JSON messageData and emit messageReceived.
*/
void WebSocketTransport::textMessageReceived(const QString &messageData)
{
    QJsonParseError error;
    QJsonDocument message = QJsonDocument::fromJson(messageData.toUtf8(), &error);
    if (error.error) {
        qWarning() << "Failed to parse text message as JSON object:" << messageData
                   << "Error is:" << error.errorString();
        return;
    } else if (!message.isObject()) {
        qWarning() << "Received JSON message that is not an object: " << messageData;
        return;
    }
    emit messageReceived(message.object(), this);
}
```
实现自己的后端任务处理类WebChannelProcess，这个类注册到QWebChannel，`registerObject("shelfjs", &webChannelProc)`，注意这里的`shelfjs`，在JS中会用到，WebChannelProcess实现和JS的交互，signals将数据发送到JS，slots对应JS的远程调用，`OnWcGetShelf`函数中会发射`wcPutShelf`方法，把信息发给JS，
```cpp
class WebChannelProcess : public QObject
{
    Q_OBJECT
public:
    explicit WebChannelProcess(QObject *parent = nullptr);

signals:
    void wcTestSend(QString msg);
    void wcPutShelf(QString msg);
    void wcPutShelfList(QString msg);
    void wcPutGoodsList(QString msg);
    void wcPutShelfGoodsList(QString msg);
    void wcPutEvent(QString msg);
    void wcPutEventList(QString msg);

public slots:
    /*web channel msg*/
    void OnWcTestRecv(QString msg);
    void OnWcGetShelf(int id);
    void OnWcGetShelfList();
    void OnWcGetGoodsList(int start, int len, int ori);
    void OnWcGetShelfGoodsList(int shelfId);
    void OnWcGetEventList(int start, int len, int ori);
};

void WebChannelProcess::OnWcGetShelf(int id)
{
    QJsonDocument doc;
    shelf s;
    s.id = id;
    int index = dataHelperInst->listShelf.indexOf(s, 0);
    if (index < 0)
    {
        s.id = -1;
        s.hNum = 0;
        s.vNum = 0;
        doc.setObject(s.toJsonObject());
    }
    else
    {
        doc.setObject(dataHelperInst->listShelf.at(index).toJsonObject());
    }
    emit wcPutShelfList(QString(doc.toJson()));
}
```

# JS
JS部分的实现，首先引用`qwebchannel.js`，这个文件在Demo中可以找到，以`OnWcGetShelf`为例，JS中调用`shelfjs.OnWcGetShelf`，对应调用WebChannelProcess的`OnWcGetShelf`函数，
```js
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        <script type="text/javascript" src="./qwebchannel.js"></script>
        <script type="text/javascript">
            //BEGIN SETUP
            function output(message)
            {
                var output = document.getElementById("output");
                output.innerHTML = output.innerHTML + message + "\n";
            }
            window.onload = function() {
                if (location.search != "")
                    var baseUrl = (/[?&]webChannelBaseUrl=([A-Za-z0-9\-:/\.]+)/.exec(location.search)[1]);
                else
                    var baseUrl = "ws://localhost:12345";

                output("Connecting to WebSocket server at " + baseUrl + ".");
                var socket = new WebSocket(baseUrl);

                socket.onclose = function()
                {
                    console.error("web channel closed");
                };
                socket.onerror = function(error)
                {
                    console.error("web channel error: " + error);
                };
                socket.onopen = function()
                {
                    output("WebSocket connected, setting up QWebChannel.");
                    new QWebChannel(socket, function(channel) {
                        // make dialog object accessible globally
                        window.shelfjs = channel.objects.shelfjs;

                        document.getElementById("getShelf").onclick = function() {
                            var input = document.getElementById("input");
                            var text = input.value;
                            if (!text) {
                                return;
                            }

                            output("Get shelf: " + text);
                            input.value = "";
                            shelfjs.OnWcGetShelf(parseInt(text));
                        }
...
```

# 使用
进入安装目录，打开网页，点击Get Shelf按钮，会调用Qt中`OnWcGetShelf`函数，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190529211957842.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
