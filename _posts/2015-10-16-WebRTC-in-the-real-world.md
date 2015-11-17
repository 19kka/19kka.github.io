---
layout:     post
title:      在真实的世界的WebRTC (一)（译）
date:       2015-11-16 9:20:18
summary:    what is signaling|什么是信令
categories: WebRTC,iOS
---

####原文地址[http://www.html5rocks.com/en/tutorials/webrtc/infrastructure/](http://www.html5rocks.com/en/tutorials/webrtc/infrastructure/)

译者注释：  

1. Signaling翻译的时候有时候写作 信令 有时候写作信号。  
2. 部分图片在墙内可能无法显示


---


WebRTC允许P2P的连接。但是，
WebRTC还是需要一些服务器:  

  * 为客户端配合交换元数据的连接：这个叫做信号(signaling)
  * 为了对付网络地址转换(NATs)和防火墙  

本文中我们会告诉你如何去建立一个信令服务器，以及如何去处理在真实世界中的连接(real-world connectivity)使用STUN和TURN服务器出现的一些难以预料的情况。我们也解释WebRTC应用是如何使用服务器处理多方呼叫和相互配合(例如VoIP和PSTN 也称为电话)。

如果你不熟悉WebRTC的基础知识，我们强烈建议你在看本文之前先去看[Getting Started With WebRTC](http://www.html5rocks.com/en/tutorials/webrtc/basics/)

---

###什么是信号（信令）（Signaling）
信令是用来协调连接的过程。WebRTC应用有序的建立一个呼叫，客户端需要交换一些信息： 

* 会话控制消息用来打开或者关闭连接
* 错误消息
* 多媒体元数据就像<b>多媒体数字信号编解码器(codec)和多媒体数字信号编解码器的设置</b>，带宽和多媒体类型。
* 密钥数据，建立加密的连接
* 网络数据，比如主机外网的IP地址和端口  

这个发信令的过程需要一条让客户端接收与发送消息的路径。这个结构在WebRTC的API中没有实现：你需要自己去建立。我们会下在下方描述一些建立信令服务器的方法。首先如何，然后如何，还有一些简单的来龙去脉。

###为什么发信令没有在WebRTC中定义？

为了避免过多的冗余和最大限度的兼容已有的技术，信令方法和协议没有被规定在WebRTC的标准当中。这个处理方式被JESP写在[ JavaScript Session Establishment Protocol](http://tools.ietf.org/html/draft-ietf-rtcweb-jsep-03#section-1.1):  

>The thinking behind WebRTC call setup has been to fully specify and control the media plane, but to leave the signaling plane up to the application as much as possible. The rationale is that different applications may prefer to use different protocols, such as the existing SIP or Jingle call signaling protocols, or something custom to the particular application, perhaps for a novel use case. In this approach, the key information that needs to be exchanged is the multimedia session description, which specifies the necessary transport and media configuration information necessary to establish the media plane.  

JSEP的结构也避免浏览器不得不去保存状态：那就是去搞一个信令状态机。这样可能会带来一个问题，举例来说，信令数据每次都会在一个页面刷新的时候丢失，相反信号状态可以保存在服务器上。

![JSEP architecture][1]
<center>JESP architecture</center>

JESP要求交换双发给出邀请和回答：前面提到的多媒体元数据。邀请和回答被连接用在<b>会话描述协议的格式(Session Description Protocol format SDP)</b>中。格式看起来像这样：

>v=0  
o=- 7614219274584779017 2 IN IP4 127.0.0.1  
s=-  
t=0 0  
a=group:BUNDLE audio video  
a=msid-semantic: WMS  
m=audio 1 RTP/SAVPF 111 103 104 0 8 107 106 105 13 126  
c=IN IP4 0.0.0.0  
a=rtcp:1 IN IP4 0.0.0.0  
a=ice-ufrag:W2TGCZw2NZHuwlnf  
a=ice-pwd:xdQEccP40E+P0L5qTyzDgfmW  
a=extmap:1 urn:ietf:params:rtp-hdrext:ssrc-audio-level  
a=mid:audio  
a=rtcp-mux  
a=crypto:1 AES_CM_128_HMAC_SHA1_80 inline:  9c1AHz27dZ9xPI91YNfSlI67/EMkjHHIHORiClQe  
a=rtpmap:111 opus/48000/2  
…  


想要知道SDP的全部官方的解释？看这里[IETF](http://datatracker.ietf.org/doc/draft-nandakumar-rtcweb-sdp/?include_text=1)的例子。  

记住WebRTC被设计在邀请或者回答设置本地或者远程的描述之前微调，*（感觉翻译的不太好，原文是这样的：Bear in mind that WebRTC is designed so that the offer or answer can be tweaked before being set as the local or remote description）*通过编辑SDP文本中的值。举例：perferAudioCodec()函数在apprtc.appspot.com能用于设置默认的编解码器和比特率。SDP用JavaScript操作起来有点痛苦，这里有些讨论关于未来WebRTC版本中是否需要用JSON替代，但是SDP也有点优点让人坚持使用。

###RTCPeerConnection+信号：邀请，回答和申请者
RTCPeerConnection是用于创建节点的连接和通信，音频和视频的API。
初始化这个过程RTCPeerConnection有两个任务： 

* 确定本地媒体设备条件，比如分辨率、编码能力。这些是用于请求和回答的元数据。
* 为应用的主机地址获取潜在的网络地址作为候选。一旦本地数据确立，这个会通过远程信令机制进行交换。

想想一下 Alice 呼叫 Eve。下面是一个完整的呼叫回答过程：  
1. Alice创建了一个RTCPeerConnection对象  
2. Alice创建了一个offer（一个SDP会话描述）用RTCPeerConnection createOffer()这个方法  
3. Alice调用offer的setLocalDescription()
4. Alice讲offer字符串化并使用信令机制发送给Eve
5. Eve调用setRemoteDescipition()，所以她的RTCPeerConnection知道Alice的设置
6. Eve调用createAnswer(),然后成功回调通过一个本地的会话描述：Eve的answer
7. Eve调用setLocalDescription()设定她的answer作为本地描述 
8. Eve使用信令机制发送她字符串化之后的answer给Alice，Alice根据Eve的answer座位远程的会话描述使用setRemoteDescription()这个方法

搞定。
Alice和Eve同样也需要交换网络信息。使用ICE框架找到网络接口和端口的过程就是“寻找候选人”的意思。

Alice用Onicecandidate handle创建RTCPeerConnection对象。这个Handle当网络候选人可用的时候被调用。
在这个Handle中，Alice发送序列号的获选人信息给Eve，通过他们的信令频道。
当Eve从Alice那获取到到候选人信息时，她就调用addIceCandidate()去添加候选人给远程节点描述(remote peer description)
JSEP支持ICE候选人流动？，它允许呼叫者提供候选人的一些信息但并不用在线。也不用等待所有的候选人到达。

###为WebRTC的信令编程
下面完整总结了信令流程的[W3C](http://www.w3.org/TR/webrtc/#simple-peer-to-peer-example)代码示例。这个代码假定一些信号机制，SignalingChannel。信令会在下面更详细的讨论。

{% highlight js lineno %}
	var signalingChannel = new SignalingChannel();
var configuration = {
  'iceServers': [{
    'url': 'stun:stun.example.org'
  }]
};
var pc;

// call start() to initiate

function start() {
  pc = new RTCPeerConnection(configuration);

  // send any ice candidates to the other peer
  pc.onicecandidate = function (evt) {
    if (evt.candidate)
      signalingChannel.send(JSON.stringify({
        'candidate': evt.candidate
      }));
  };

  // let the 'negotiationneeded' event trigger offer generation
  pc.onnegotiationneeded = function () {
    pc.createOffer(localDescCreated, logError);
  }

  // once remote stream arrives, show it in the remote video element
  pc.onaddstream = function (evt) {
    remoteView.src = URL.createObjectURL(evt.stream);
  };

  // get a local stream, show it in a self-view and add it to be sent
  navigator.getUserMedia({
    'audio': true,
    'video': true
  }, function (stream) {
    selfView.src = URL.createObjectURL(stream);
    pc.addStream(stream);
  }, logError);
}

function localDescCreated(desc) {
  pc.setLocalDescription(desc, function () {
    signalingChannel.send(JSON.stringify({
      'sdp': pc.localDescription
    }));
  }, logError);
}

signalingChannel.onmessage = function (evt) {
  if (!pc)
    start();

  var message = JSON.parse(evt.data);
  if (message.sdp)
    pc.setRemoteDescription(new RTCSessionDescription(message.sdp), function () {
      // if we received an offer, we need to answer
      if (pc.remoteDescription.type == 'offer')
        pc.createAnswer(localDescCreated, logError);
    }, logError);
  else
    pc.addIceCandidate(new RTCIceCandidate(message.candidate));
};

function logError(error) {
  log(error.name + ': ' + error.message);
}
{% endhighlight %}


想了解的过程可以参考[simpl.info/pc](http://simpl.info/rtcpeerconnection/)

###发现小伙伴
这句话的意思是，如何找到对方。

用于电话呼叫，我们有电话号码和目录。对于网络视频聊天和短信，我们需要的身份和状态管理系统，并为用户发起会话的手段。实现WebRTC的应用程序需要一种方式，为客户发信号给对方自己要启动或加入呼叫。

同伴发现机制没有在WebRTC中实现，也不会去实现。这个过程可以很简单，如电子邮件或短信网址：用于视频聊天等应用talky.io，tawk.com和browsermeeting.com你分享一个自定义链接邀请的人打来的电话。开发者Chris已经建立了一个耐人寻味的服务器，实现WebRTC的实验，使实现WebRTC呼叫参与任何消息服务，他们喜欢的，比如IM，电子邮件等交流方式。



[1]:http://www.html5rocks.com/en/tutorials/webrtc/infrastructure/jsep.png
