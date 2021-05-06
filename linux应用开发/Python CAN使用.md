# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 报错CAN bit stuff error
下面函数触发`CAN bit stuff error`，这个错误会刷屏，且结点出错后，再连接也无法继续工作，
```python
self.CanOpenNetworkInst[hw_id].GetRemoteNode(node_id).sdo.download(index, subindex, data)
```
错误信息来自，
```python
# python-can-3.3.4\can\interfaces\ixxat\canlib.py line227
CAN_ERROR_MESSAGES = {
    constants.CAN_ERROR_STUFF:  "CAN bit stuff error",
    constants.CAN_ERROR_FORM:   "CAN form error",
    constants.CAN_ERROR_ACK:    "CAN acknowledgment error",
    constants.CAN_ERROR_BIT:    "CAN bit error",
    constants.CAN_ERROR_CRC:    "CAN CRC error",
    constants.CAN_ERROR_OTHER:  "Other (unknown) CAN error",
}
```
关掉network，错误信息还是一直打印，
```python
# python-can-3.3.4\can\interfaces\ixxat\canlib.py line415
    def _recv_internal(self, timeout):
        """ Read a message from IXXAT device. """

        # TODO: handling CAN error messages?
        data_received = False

        if timeout == 0:
            # Peek without waiting
            try:
                _canlib.canChannelPeekMessage(self._channel_handle, ctypes.byref(self._message))
            except (VCITimeout, VCIRxQueueEmptyError):
                return None, True
            else:
                if self._message.uMsgInfo.Bits.type == constants.CAN_MSGTYPE_DATA:
                    data_received = True
        else:
            # Wait if no message available
            if timeout is None or timeout < 0:
                remaining_ms = constants.INFINITE
                t0 = None
            else:
                timeout_ms = int(timeout * 1000)
                remaining_ms = timeout_ms
                t0 = _timer_function()

            while True:
                try:
                    _canlib.canChannelReadMessage(self._channel_handle, remaining_ms, ctypes.byref(self._message))
                except (VCITimeout, VCIRxQueueEmptyError):
                    # Ignore the 2 errors, the timeout is handled manually with the _timer_function()
                    pass
                else:
                    # See if we got a data or info/error messages
                    if self._message.uMsgInfo.Bits.type == constants.CAN_MSGTYPE_DATA:
                        data_received = True
                        break

                    elif self._message.uMsgInfo.Bits.type == constants.CAN_MSGTYPE_INFO:
                        log.info(CAN_INFO_MESSAGES.get(self._message.abData[0], "Unknown CAN info message code {}".format(self._message.abData[0])))

                    elif self._message.uMsgInfo.Bits.type == constants.CAN_MSGTYPE_ERROR:
                        log.warning(CAN_ERROR_MESSAGES.get(self._message.abData[0], "Unknown CAN error message code {}".format(self._message.abData[0])))

                    elif self._message.uMsgInfo.Bits.type == constants.CAN_MSGTYPE_TIMEOVR:
                        pass
                    else:
                        log.warn("Unexpected message info type")

                if t0 is not None:
                    remaining_ms = timeout_ms - int((_timer_function() - t0) * 1000)
                    if remaining_ms < 0:
                        break

        if not data_received:
            # Timed out / can message type is not DATA
            return None, True

        # The _message.dwTime is a 32bit tick value and will overrun,
        # so expect to see the value restarting from 0
        rx_msg = Message(
            timestamp=self._message.dwTime / self._tick_resolution,  # Relative time in s
            is_remote_frame=True if self._message.uMsgInfo.Bits.rtr else False,
            is_extended_id=True if self._message.uMsgInfo.Bits.ext else False,
            arbitration_id=self._message.dwMsgId,
            dlc=self._message.uMsgInfo.Bits.dlc,
            data=self._message.abData[:self._message.uMsgInfo.Bits.dlc],
            channel=self.channel
        )

        return rx_msg, True
```
接收函数，
```python
# python-can-3.3.4\can\bus.py line64
    def recv(self, timeout=None):
        """Block waiting for a message from the Bus.

        :type timeout: float or None
        :param timeout:
            seconds to wait for a message or None to wait indefinitely

        :rtype: can.Message or None
        :return:
            None on timeout or a :class:`can.Message` object.
        :raises can.CanError:
            if an error occurred while reading
        """
        start = time()
        time_left = timeout

        while True:

            # try to get a message
            msg, already_filtered = self._recv_internal(timeout=time_left)

            # return it, if it matches
            if msg and (already_filtered or self._matches_filters(msg)):
                LOG.log(self.RECV_LOGGING_LEVEL, 'Received: %s', msg)
                return msg

            # if not, and timeout is None, try indefinitely
            elif timeout is None:
                continue

            # try next one only if there still is time, and with
            # reduced timeout
            else:

                time_left = timeout - (time() - start)

                if time_left > 0:
                    continue
                else:
                    return None

```
`class Notifier`比较可疑，这里有一个线程，所以错误会刷屏，
```python
# python-can-3.3.4\can\notifier.py line92
    def _rx_thread(self, bus):
        msg = None
        try:
            while self._running:
                if msg is not None:
                    with self._lock:
                        if self._loop is not None:
                            self._loop.call_soon_threadsafe(
                                self._on_message_received, msg)
                        else:
                            self._on_message_received(msg)
                msg = bus.recv(self.timeout)
        except Exception as exc:
            self.exception = exc
            if self._loop is not None:
                self._loop.call_soon_threadsafe(self._on_error, exc)
            else:
                self._on_error(exc)
            raise
```
后定位是应用代码错误，`class Network`的`disconnect`没有被正确调用，然后释放类变量，导致python报语法错误，没有正确执行错误恢复函数，应该是`a.b.disconnect()`，我搞成了`a.disconnect()`。

