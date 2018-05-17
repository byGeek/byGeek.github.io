---
title: c#中使用RegisterNotification来接收特定GUID的设备消息
date: 2018-04-20 16:00:56
tags:
- csharp
categories:
- coding
keywords:
- interop
description:

---



在WIN32 API中提供了一个[RegisterNotification](https://msdn.microsoft.com/en-us/library/windows/desktop/aa363431%28v=vs.85%29.aspx)函数来接收系统发过来的message。本文介绍一下在csharp中调用该接口来监听特定GUID的设备消息。

<!--more-->

在MSDN中，RegisterDeviceNotification的原型为

```cpp
HDEVNOTIFY WINAPI RegisterDeviceNotification(
  _In_ HANDLE hRecipient,
  _In_ LPVOID NotificationFilter,
  _In_ DWORD  Flags
);
```

其中NotificationFilter起到消息过滤的作用，也就是说如果我们只想收到特定的消息，可以传递特定的NotificationFilter参数。

> *NotificationFilter* [in]
>
> A pointer to a block of data that specifies the type of device for which notifications should be sent. This block always begins with the [**DEV_BROADCAST_HDR**](https://msdn.microsoft.com/en-us/library/windows/desktop/aa363246%28v=vs.85%29.aspx) structure. The data following this header is dependent on the value of the **dbch_devicetype** member, which can be **DBT_DEVTYP_DEVICEINTERFACE** or **DBT_DEVTYP_HANDLE**. For more information, see Remarks.

需要注意的是，对于PORT设备，arrival和removal消息会自动广播给顶层窗口。所以如果你的设备不是PORT设备，又想收到arrival和removal消息，需要手动注册消息监听。

> The [DBT_DEVICEARRIVAL](https://msdn.microsoft.com/en-us/library/windows/desktop/aa363205%28v=vs.85%29.aspx) and [DBT_DEVICEREMOVECOMPLETE](https://msdn.microsoft.com/en-us/library/windows/desktop/aa363208%28v=vs.85%29.aspx) events are automatically broadcast to all top-level windows for port devices. Therefore, it is not necessary to call **RegisterDeviceNotification** for ports, and the function fails if the **dbch_devicetype**member is **DBT_DEVTYP_PORT**. Volume notifications are also broadcast to top-level windows, so the function fails if **dbch_devicetype**is **DBT_DEVTYP_VOLUME**. OEM-defined devices are not used directly by the system, so the function fails if **dbch_devicetype** is **DBT_DEVTYP_OEM**.

关于设备类型，猜测这个应该跟设备的驱动有关，在设备驱动中会指明设备的类型，然后在设备插入到系统中，会有一个枚举过程，这样系统就知道了该设备类型。上述的DBT_DEVICEARRIVAL和DBT_DEVICEREMOVECOMPLETE是[WM_DEVICECHANGE](https://msdn.microsoft.com/en-us/library/windows/desktop/aa363480%28v=vs.85%29.aspx)消息中的参数。



如果设备类型为[**DBT_DEVTYP_DEVICEINTERFACE** ](https://msdn.microsoft.com/en-us/library/windows/desktop/aa363244%28v=vs.85%29.aspx)，查看MSDN可知，该结构体中有一个guid的参数，正是我们需要的通过GUID来判断是否是特定设备的参数。我们在接受windows消息的函数中，判断是否是需要监测的设备即可。

```csharp
public void ProcessWinMessage(int msg, IntPtr wParam, IntPtr lParam)
        {
            if (msg == Native.WM_DEVICECHANGE)
            {
                switch (wParam.ToInt32())
                {
                    case (int)Native.WM_DEVICECHANGE_WPPARAMS.DBT_DEVICEARRIVAL:

                        if (IsDesiredDevice(lParam))
                        {
                            if (StateChanged != null)
                            {
                                StateChanged(true);
                            }
                        }

                        break;

                    case (int)Native.WM_DEVICECHANGE_WPPARAMS.DBT_DEVICEREMOVECOMPLETE:

                        if (IsDesiredDevice(lParam))
                        {
                            if (StateChanged != null)
                            {
                                StateChanged(false);
                            }
                        }
                        break;
                    /*
                     * this event will be fired when device been added and removed
                case Native.DBT_DEVNODES_CHANGED:
                        
                    if (StateChanged != null)
                    {
                        StateChanged(false);
                    }
                    break;
                     * */
                    default:
                        break;
                }
            }
        }
```

`IsDesignedDevice`定义如下

```csharp
private bool IsDesiredDevice(IntPtr lParam)
        {
            var hdr = (Native.DEV_BROADCAST_HDR)Marshal.PtrToStructure(lParam, typeof(Native.DEV_BROADCAST_HDR));
            if (hdr.dbcc_devicetype == (uint)_deviceType)
            {
                if (_deviceType == Native.DeviceType.DBT_DEVTYP_DEVICEINTERFACE)
                {
                    Native.DEV_BROADCAST_DEVICEINTERFACE deviceInterface =
                        (Native.DEV_BROADCAST_DEVICEINTERFACE)Marshal.PtrToStructure(lParam, typeof(Native.DEV_BROADCAST_DEVICEINTERFACE));

                    var str = Encoding.Default.GetString(deviceInterface.dbcc_classguid);
                    var guid = new Guid(deviceInterface.dbcc_classguid);
                    if (guid.ToString() == DEVICE_INTERFACE_GUID)
                    {
                        return true;
                    }
                }
                else
                {
                    //TODO: other device type
                }
            }
            return false;
        }
```

[DEV_BROADCAST_HDR](https://msdn.microsoft.com/en-us/library/windows/desktop/aa363246%28v=vs.85%29.aspx)结构体中的三个字段包含在所有设备类型结构体中，所以可以先将WM_DEVICECHANGE中的lParam转化为DEV_BROADCAST_HDR类型，然后根据其`dbch_devicetype`来转化为特定设备类型结构体类型，如`DBT_DEVTYP_DEVICEINTERFACE`。



完整的例子请参考：[UsbDetector](https://github.com/byGeek/UsbDetector)

