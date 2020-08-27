# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [RGB颜色值与十六进制颜色码转换工具](https://www.sioe.cn/yingyong/yanse-rgb-16/)

# 固件更新
工程目录下`dciot_build\private`文件夹复制到SD卡根目录，重新上电自动更新，大改2min时间，
![129](https://img-blog.csdnimg.cn/20200210202814205.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 硬件调试环境
屏幕赠送一块调试板，PC串口TXD接调试板TXD，RXD接RXD
# 软件开发环境安装
开发软件都可以到大彩的官网下载，
- VisualTFT_3.0.0.1075
- VSPD

# 用虚拟串口屏来调试调试
VSPD新建一对虚拟串口，VisualTFT点击`调试 > 运行虚拟串口屏`，分别用虚拟串口屏和串口助手打开两个串口，
![126](https://img-blog.csdnimg.cn/20191204221411651.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 设计
界面上拖拉摆放即可。
![127](https://img-blog.csdnimg.cn/20191227220902540.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
```c
小键盘设置加热时间：EE B1 11 00 00 00 2C 11 31 30 30 2E 31 00 FF FC FF FF 
小键盘设置加热时间：EE B1 11 00 00 00 2C 11 31 30 30 2E 30 00 FF FC FF FF 
小键盘设置保温时间：EE B1 11 00 00 00 2D 11 31 30 30 2E 30 00 FF FC FF FF 
小键盘设置冷却时间：EE B1 11 00 00 00 2E 11 31 30 30 2E 30 00 FF FC FF FF 
触摸屏启动按键：EE B1 11 00 00 00 18 10 01 01 FF FC FF FF EE B1 11 00 00 00 18 10 01 00 FF FC FF FF 
触摸屏复位按键：EE B1 11 00 00 00 17 10 01 01 FF FC FF FF EE B1 11 00 00 00 17 10 01 00 FF FC FF FF 
触摸屏模式按键：EE B1 11 00 00 00 19 10 01 01 FF FC FF FF EE B1 11 00 00 00 19 10 01 00 FF FC FF FF 
触摸屏设置按键：EE B1 11 00 00 00 16 10 01 01 FF FC FF FF EE B1 11 00 00 00 16 10 01 00 FF FC FF FF 
触摸屏确定按键：EE B1 11 00 00 00 3F 10 01 01 FF FC FF FF EE B1 11 00 00 00 3F 10 01 00 FF FC FF FF 
```
从设计软件怎样知道当前画面ID，
![2](https://img-blog.csdnimg.cn/20200809183758507.PNG)

# 输入
![4](https://img-blog.csdnimg.cn/2020082314113063.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)
输入大写字母的时候，需要按Shift键，切换到大写字母输入模式。

# 单片机软件Demo分析
## 屏幕刷新
```c
if(timer_tick_count - timer_tick_last_update >= 200/*TIME_100MS*/)
{
    timer_tick_last_update = timer_tick_count;   
    UpdateUI();
} 
```
## 串口命令接收部分
首先查找帧头`0xee`，找到帧头后查找帧尾`0xfffcffff`，找到之后把中间的数据提取出来就是一条命令，下一步可选择对命令进行CRC校验，帧尾的前两个字节为CRC校验值。
## 触摸屏命令处理
命令的结构，
```c
typedef struct
{
    uint8    cmd_head;                    //帧头

    uint8    cmd_type;                    //命令类型(UPDATE_CONTROL)    
    uint8    ctrl_msg;                    //CtrlMsgType-指示消息的类型
    uint16   screen_id;                   //产生消息的画面ID
    uint16   control_id;                  //产生消息的控件ID
    uint8    control_type;                //控件类型

    uint8    param[256];                  //可变长度参数，最多256个字节

    uint8  cmd_tail[4];                   //帧尾
}CTRL_MSG,*PCTRL_MSG;
```
`cmd_type`有下面几种，
```c
#define NOTIFY_TOUCH_PRESS         0X01  //触摸屏按下通知
#define NOTIFY_TOUCH_RELEASE       0X03  //触摸屏松开通知
#define NOTIFY_WRITE_FLASH_OK      0X0C  //写FLASH成功
#define NOTIFY_WRITE_FLASH_FAILD   0X0D  //写FLASH失败
#define NOTIFY_READ_FLASH_OK       0X0B  //读FLASH成功
#define NOTIFY_READ_FLASH_FAILD    0X0F  //读FLASH失败
#define NOTIFY_MENU                0X14  //菜单事件通知
#define NOTIFY_TIMER               0X43  //定时器超时通知
#define NOTIFY_CONTROL             0XB1  //控件更新通知
#define NOTIFY_READ_RTC            0XF7  //读取RTC时间
#define NOTIFY_HandShake           0X55  //握手通知
```
`ctrl_msg`有下面几种，只有当`cmd_type`是`NOTIFY_CONTROL`才有，其它情况没有这个字段，
```c
#define MSG_GET_CURRENT_SCREEN     0X01  //画面ID变化通知
#define MSG_GET_DATA               0X11  //控件数据通知
```
`control_type`有下面几种，只有当`ctrl_msg`不是`MSG_GET_CURRENT_SCREEN`才有，其它情况没有这个字段，不同`control_type`对应的`param`长度是不一样的，
```c
enum CtrlType
{
    kCtrlUnknown=0x0,
    kCtrlButton=0x10,                     //按钮
    kCtrlText,                            //文本
    kCtrlProgress,                        //进度条
    kCtrlSlider,                          //滑动条
    kCtrlMeter,                           //仪表
    kCtrlDropList,                        //下拉列表
    kCtrlAnimation,                       //动画
    kCtrlRTC,                             //时间显示
    kCtrlGraph,                           //曲线图控件
    kCtrlTable,                           //表格控件
    kCtrlMenu,                            //菜单控件
    kCtrlSelector,                        //选择控件
    kCtrlQRCode,                          //二维码
};
```
由`screen_id`和`control_id`可唯一定位一个控件。
