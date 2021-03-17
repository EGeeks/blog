# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 方法
主要是使用IXXAT设备的`UniqueHardwareId`参数，此参数要和`channel`区分，`UniqueHardwareId`定位某个设备，而`channel`是指该设备上的某个通道，比如一个USB2CAN设备可能有两个通道，
```python
class CanOpenService:
    def __init__(self, EDS_PATH, HW_ID):
        self.EDS_PATH = EDS_PATH
        self.IsOccupy = False
        self.NetworkStatus = False
        self.Response = ""
        
        self.network = canopen.Network()
        self.network.connect(bustype='ixxat', channel=0, bitrate=1000000, UniqueHardwareId=HW_ID)
        self.object_dictionary = self.EDS_PATH
        self.RemoteNodeList = {}
        self.HeartbeatTimestamp = {}
        self.LocalNodeList = {}
        self.CreateLocalNode()
        self.CreateRemoteNode()  
        self.mCanOpenNetwork = self.network
        self.NetworkStatus = True
        for i in range(33, 65):
            self.LocalNodeList[i].add_write_callback(self.ScpiResponse)

```
`UniqueHardwareId`参数在如下位置获取，
![32](https://img-blog.csdnimg.cn/20201216000317518.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
[CANopen for Python](https://github.com/christiansandberg/canopen)内部使用[python-can](https://github.com/hardbyte/python-can)，python-can封装了各个CAN接口厂家的模块驱动，python-can内部使用IXXAT的C接口，
```python
try:
    # Map all required symbols and initialize library ---------------------------
    #HRESULT VCIAPI vciInitialize ( void );
    _canlib.map_symbol("vciInitialize", ctypes.c_long, (), __check_status)

    #void VCIAPI vciFormatError (HRESULT hrError, PCHAR pszText, UINT32 dwsize);
    _canlib.map_symbol("vciFormatError", None, (ctypes_HRESULT, ctypes.c_char_p, ctypes.c_uint32))
    # Hack to have vciFormatError as a free function
    vciFormatError = functools.partial(__vciFormatError, _canlib)

    # HRESULT VCIAPI vciEnumDeviceOpen( OUT PHANDLE hEnum );
    _canlib.map_symbol("vciEnumDeviceOpen", ctypes.c_long, (PHANDLE,), __check_status)
    # HRESULT VCIAPI vciEnumDeviceClose ( IN HANDLE hEnum );
    _canlib.map_symbol("vciEnumDeviceClose", ctypes.c_long, (HANDLE,), __check_status)
    # HRESULT VCIAPI vciEnumDeviceNext( IN  HANDLE hEnum, OUT PVCIDEVICEINFO pInfo );
    _canlib.map_symbol("vciEnumDeviceNext", ctypes.c_long, (HANDLE, structures.PVCIDEVICEINFO), __check_status)
```
使用`vciEnumDeviceNext`枚举，获取`PVCIDEVICEINFO`，这个结构体中包含`UniqueHardwareId`，从而定位到某个CAN设备，
![33](https://img-blog.csdnimg.cn/20201216000627343.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
![31](https://img-blog.csdnimg.cn/20201216000645335.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)



