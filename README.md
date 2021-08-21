# Goonvif

轻松管理 IP 设备，包括摄像机。Goonvif 是用于管理 IP 设备的 ONVIF 协议的实现。该库的目的是方便、轻松地管理支持 ONVIF 标准的 IP 摄像机和其他设备。

## 安装

要安装库，您需要使用 go get 实用程序：

```
go get -u github.com/hepeichun/goonvif
```

## 支持的服务

以下服务已全面实施：

* 设备
* 媒体
* 云台
* 成像

## 用法

### 一般概念

1. 连接到设备
2. 身份验证（如果需要）
3. 定义数据类型
4. 执行所需的方法

#### 连接到设备

如果 192.168.13.42 网络上有设备，并且其 ONVIF 服务使用端口 1234 ，那么您可以通过以下方式连接到该设备：

```
dev, err := goonvif.NewDevice("192.168.13.42:1234")
```

_ONVIF 端口可能因设备而异，要了解要使用的端口，您可以转到设备的 Web 界面。 **这通常是端口 80** 。

#### 验证

如果 ONVIF 服务之一的任何功能需要身份验证，则必须使用该方法`Authenticate`。

```
device := onvif.NewDevice("192.168.13.42:1234")
device.Authenticate("username", "password")

```

#### 定义数据类型

该库中的每个ONVIF服务都有自己的包，其中定义了该服务的所有数据类型，包名与服务名称相同，以大写字母开头。
Goonvif 为该库支持的每个 ONVIF 服务的每个功能定义了结构。让我们定义`GetCapabilities`服务函数的数据类型`Device`。
这是按如下方式完成的：

```
capabilities := Device.GetCapabilities{Category:"All"}

```

为什么 GetCapabilities 结构有一个 Category 字段，为什么这个字段有 All？

下图显示了 [GetCapabilities](https://www.onvif.org/ver10/device/wsdl/devicemgmt.wsdl) 函数的文档。可以看出，该函数采用了一个参数 Category，其值必须是以下之一： `'All', 'Analytics', 'Device', 'Events', 'Imaging', 'Media' или 'PTZ'`.

![设备获取能力](img/exmp_GetCapabilities.png)

定义 [云台](https://www.onvif.org/ver20/ptz/wsdl/ptz.wsdl) 服务的 GetServiceCapabilities 函数的数据类型的示例：

```
ptzCapabilities := PTZ.GetServiceCapabilities{}

```

在下图中，您可以看到 GetServiceCapabilities 不带任何参数。

![PTZ 获取服务能力](img/GetServiceCapabilities.png)

_常见的数据类型在 xsd / onvif 包中。所有服务通用的数据类型（结构）在 onvif 包中定义。_

定义 [Device](https://www.onvif.org/ver10/device/wsdl/devicemgmt.wsdl) 服务的 CreateUsers 函数的数据类型的示例：

```
createUsers := Device.CreateUsers{User: onvif.User{Username:"admin", Password:"qwerty", UserLevel:"User"}}

```

下图显示，在本例中，CreateUsers结构体的字段必须是User，其数据类型为User结构体，包含Username、Password、UserLevel字段和一个可选的Extension。User 结构在 onvif 包中。

![设备创建用户](img/exmp_CreateUsers.png)

#### 执行所需的方法

要执行已构建的 ONVIF 服务之一的任何功能，您必须使用`CallMethod`设备对象。

```
createUsers := Device.CreateUsers{User: onvif.User{Username:"admin", Password:"qwerty", UserLevel:"User"}}
device := onvif.NewDevice("192.168.13.42:1234")
device.Authenticate("username", "password")
resp, err := dev.CallMethod(createUsers)
```