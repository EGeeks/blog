# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [QT 日志 log](https://blog.csdn.net/xiaoyink/article/details/79721481)

# 使用QDebug实现
注册log handle，直接上代码吧，
```c
#ifndef LOGHELPER_H
#define LOGHELPER_H

#include <QString>
#include <QFile>
#include <QTextStream>
#include <QDebug>

class logHelper
{
public:
    static logHelperInit()
    {
#ifndef QT_DEBUG
        logFile.setFileName("log.txt");
        logFile.open(QIODevice::WriteOnly | QIODevice::Append);
        if (logFile.size() > 1024*1024)
        {
            logFile.close();
            logFile.open(QIODevice::WriteOnly | QIODevice::Truncate);
        }
        qInstallMessageHandler(logMsgOutput);
#endif
    }

#ifndef QT_DEBUG
    static void logMsgOutput(QtMsgType type, const QMessageLogContext &context, const QString &msg)
    {

        QFile logFile;
        //global function or class static function
        Q_UNUSED(context)
        //static QMutex mutex;
        //mutex.lock();

        QString text;
        switch((int)type)
        {
        case QtDebugMsg:
            text = QString("Debug:");
            break;

        case QtWarningMsg:
            text = QString("Warning:");
            break;

        case QtCriticalMsg:
            text = QString("Critical:");
            break;

        case QtFatalMsg:
            text = QString("Fatal:");
        }

        //QString context_info = QString("File:(%1) Line:(%2)").arg(QString(context.file)).arg(context.line);
        QString current_date_time = QDateTime::currentDateTime().toString("yyyy-MM-dd hh:mm:ss ddd");
        QString current_date = QString("(%1)").arg(current_date_time);
        //QString message = QString("%1 %2 %3 %4").arg(text).arg(context_info).arg(msg).arg(current_date);
        QString message = QString("%1 %2 %3").arg(text).arg(msg).arg(current_date);

        //QFile file("log.txt");
        //file.open(QIODevice::WriteOnly | QIODevice::Append);
        QTextStream text_stream(&logFile);
        text_stream << message << "\r\n";
        //file.flush();
        //file.close();

        //mutex.unlock();
    }
#endif
};

#endif // LOGHELPER_H
```
程序初始化时调用init函数。
