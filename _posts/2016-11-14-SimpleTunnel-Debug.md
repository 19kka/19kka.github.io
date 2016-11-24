---
layout:     post
title:      SimpleTunnel Debug
date:       2016-11-17 12:11:38
summary:    Apple Network Extension Demo
categories: iOS Network
---


###### 前言: 由于国内网络情况日益严峻，希望多学习些计算机网络的知识。Network Extension提供了在iOS平台上较为完善的网络代理API，特此学习记录。如有错误，欢迎讨论大家指正。

#### 编译之前的准备工作如下：

> Apple 开发者账号  
> Network Extension API 权限  
> *.entitlements 配置

打开工程后包含以下目录  

```C
.
├── AppProxy
├── FilterControlProvider
├── FilterDataProvider
├── LICENSE.txt
├── PacketTunnel （VPN 隧道协议实现）*
├── README.md
├── SimpleTunnel 
├── SimpleTunnel.xcodeproj
├── SimpleTunnelServices
└── tunnel_server (Demo 的服务器端) *
```

我们主要讨论 带 *  号的内容

客户端 配置 Server Address 地址为 *192.168.1.1:7888*(地址根据你运行tunnel_server的机器为准)  
先看看服务端这里的情况。

## tunnel_server

#### 0:运行服务

Edit Schemes ->Arguments Passed On Launch-> + 

添加2个参数 1.端口 2.配置文件 
```sudo tunnel_server <port> <path-to-config-plist>```

将工程中的config.plist 放到Products 的Debug目录即可


#### 1.监听端口:

```objective-c
/// Start the network service.
	class func startListeningOnPort(_ port: Int32) -> NetService {
		let service = NetService(domain:Tunnel.serviceDomain, type:Tunnel.serviceType, name: "vv", port: port)

        

		simpleTunnelLog("Starting network service on port \(port)")

		service.delegate = ServerTunnel.serviceDelegate
		service.publish(options: NetService.Options.listenForConnections)
		service.schedule(in: .main, forMode: RunLoopMode.defaultRunLoopMode)

		return service
	}
```

#### 2.处理新连接

```objective-c
/// Handle the "new connection" event.
	func netService(_ sender: NetService, didAcceptConnectionWith inputStream: InputStream, outputStream: OutputStream) {
        simpleTunnelLog("Accepted a new connection")
		_ = ServerTunnel(newReadStream: inputStream, newWriteStream: outputStream)

	}
```

#### 3.处理输入输出流


##### 3.1 流配置委托并加入到RunLoop


```objective-c
init(newReadStream: InputStream, newWriteStream: OutputStream) {
		super.init()
		delegate = self

		for stream in [newReadStream, newWriteStream] {
			stream.delegate = self
			stream.open()
			stream.schedule(in: RunLoop.main, forMode: RunLoopMode.defaultRunLoopMode)
		}
		readStream = newReadStream
		writeStream = newWriteStream
	}
```

##### 3.2 处理委托

```objective-c
/// Handle a stream event.
    func stream(_ aStream: Stream, handle eventCode: Stream.Event) {
		switch aStream {

			case writeStream!:
				//****

			case readStream!:
				//****

			default:
				break
        }
    }
```

在处理Stream这里，由于Swift 2.3转Swift 3.0的过程中，自己对Swift 3.0的语法并不熟悉。在这里碰了许多坑，还没有绕出来。最后找到了用了 dake 已经转换好的Demo（再次感谢）。附上地址：   
[dake - Demo](https://github.com/dake/SimpleTunnelCustomizedNetworkingUsingtheNetworkExtensionFramework) 



如果时间不多，可以使用该Demo，可以节约不少时间。



#### References:  

Null
