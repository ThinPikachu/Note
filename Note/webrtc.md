### 1、webrtc建立连接的过程
<img width="768" alt="image" src="https://github.com/ThinPikachu/Note/assets/55798328/098c9610-af33-42ec-8c9c-a5b078e67832">

上述序列中，标注的场景是ClientA向ClientB发起对聊请求，调用描述如下：

1.  ClientA首先创建PeerConnection对象，然后打开本地音视频设备，将音视频数据封装成MediaStream添加到PeerConnection中。
2.  ClientA调用PeerConnection的CreateOffer方法创建一个用于offer的SDP对象，SDP对象中保存当前音视频的相关参数。ClientA通过PeerConnection的SetLocalDescription方法将该SDP对象保存起来，并通过Signal服务器发送给ClientB。
3.  ClientB接收到ClientA发送过的offer SDP对象，通过PeerConnection的SetRemoteDescription方法将其保存起来，并调用PeerConnection的CreateAnswer方法创建一个应答的SDP对象，通过PeerConnection的SetLocalDescription的方法保存该应答SDP对象并将它通过Signal服务器发送给ClientA。
4.  ClientA接收到ClientB发送过来的应答SDP对象，将其通过PeerConnection的SetRemoteDescription方法保存起来。
5.  在SDP信息的offer/answer流程中，ClientA和ClientB已经根据SDP信息创建好相应的音频Channel和视频Channel并开启Candidate数据的收集，Candidate数据可以简单地理解成Client端的IP地址信息（本地IP地址、公网IP地址、Relay服务端分配的地址）。
6.  当ClientA收集到Candidate信息后，PeerConnection会通过OnIceCandidate接口给ClientA发送通知，ClientA将收到的Candidate信息通过Signal服务器发送给ClientB，ClientB通过PeerConnection的AddIceCandidate方法保存起来。同样的操作ClientB对ClientA再来一次。
7.  这样ClientA和ClientB就已经建立了音视频传输的P2P通道，ClientB接收到ClientA传送过来的音视频流，会通过PeerConnection的OnAddStream回调接口返回一个标识ClientA端音视频流的MediaStream对象，在ClientB端渲染出来即可。同样操作也适应ClientB到ClientA的音视频流的传输。

总的来说就是：**offer、answer、ice、dtls**

> server端的ice信息可能在answer中就携带了
### 2、webrtc常见的payload type
音频用到的RTP payload类型有：`111 103 104 9 0 8 106 105 13 110 112 113 126`。

视频用到的RTP payload类型有：`96 97 98 99 100 101 102 121 127 120 125 107 108 109 124 119 123 118 114 115 116`。

参考：[https://blog.jianchihu.net/webrtc-research-m89-key-update.html](https://blog.jianchihu.net/webrtc-research-m89-key-update.html)
### 3、rtcp type
参考：
[https://blog.csdn.net/chenzheng_blog/article/details/111594990](https://blog.csdn.net/chenzheng_blog/article/details/111594990)

[https://blog.csdn.net/The_Old_man_and_sea/article/details/114780559](https://blog.csdn.net/The_Old_man_and_sea/article/details/114780559)

[https://blog.csdn.net/qq_22658119/article/details/121785298](https://blog.csdn.net/qq_22658119/article/details/121785298)

[https://datatracker.ietf.org/doc/html/rfc3550#section-6.7](https://datatracker.ietf.org/doc/html/rfc3550#section-6.7)

[https://blog.csdn.net/aggresss/article/details/108019463](https://blog.csdn.net/aggresss/article/details/108019463)（推荐）
### 4、NACK&RTX
NACK、RTX是WebRTC里丢包重传策略，两个策略之间有一定的联系。 

**NACK**：接收端通过RTCP将丢包的序列号通知给发送端，让发送端重传该包。 

**RTX**：发送端在新的SSRC上发送重传包或者冗余包。

在发送端收到NACK后，要重发接收端丢掉的包，发送的模式有两种：
-   RTX模式
在接收端通过SDP使能发送端的RTX以后，重发的包封装到RTX包里发送，RTX包与原RTP有不同的SSRC，这样有助于避免SRTP的重放攻击，也能让接收端更好的估算带宽；
-   普通模式
在没有使能RTX时，发送端只是简单的重发原来的RTP包，这种模式会影响接收端的RTCP统计，比如会出现负的丢包率。

发送端发送的冗余Padding包 发送端的初始码率在达不到目标码率的情况下，会通过发送RTX包来补充，以能够逼近目标码率，当然这个机制必须启用RTX才能激活。因此，接收端可能会收到两种RTX包，一种是被NACK触发的，一种是发送端用来补充发送码率的冗余包。
### 5、webrtc发送rtcp包流程
首先会调用ModuleRtpRtcpImpl::Process，其中会判断是否是时间去发送rtcp也就是TimeToSendRTCPReport，如果是则调用RTCPSender::SendRTCP，RTCPSender::SendCompoundRTCP，而RTCPSender::SendCompoundRTCP中会通过回调的方式去构造RTCP包，具体逻辑如下图：
![输入图片说明](/imgs/2024-06-14/AdF8tSCnlS8MIBK1.png)
builders_会在RTCPSender的构造函数中进行初始化：
![输入图片说明](/imgs/2024-06-14/FN2wdgjadePgyju7.png)
### 6、webrtc接收rtcp包流程
在PeerConnection::Initialize中，会在network_thread线程上异步调用Call::DeliverPacket函数
![输入图片说明](/imgs/2024-06-14/aN1by8oMf6rcrHxX.png)
Call::DeliverPacket中会区分rtp和rtcp来分别处理。如果是RTCP则调用Call::DeliverRtcp。
接着分别调用VideoReceiveStream、AudioReceiveStream、VideoSendStream和AudioSendStream的DeliverRtcp函数。但最终都会走到RTCPReceiver::IncomingPacket中按RTCP的type来处理。包括把fb传给gcc来处理也在这里实现。
### 7、mediasoup接收处理rtcp包流程
WebRtcTransport::OnPacketReceived - WebRtcTransport::OnRtcpDataReceived - Packet::Parse - 然后根据各个类型的rtcp包进行parse - Transport::ReceiveRtcpPacket - Transport::HandleRtcpPacket - 然后根据rtcp包的类型调用对应的操作函数
### 8、mediasoup发送处理rtcp包流程
WebRtcTransport::OnPacketReceived - WebRtcTransport::OnRtpDataReceived - Transport::ReceiveRtpPacket - TransportCongestionControlServer::IncomingPacket - 

Transport::OnTimer - Transport::SendRtcp

请求关键帧、NACK等发送RTCP包：Producer::OnRtpStreamSendRtcpPacket - Transport::OnProducerSendRtcpPacket - WebRtcTransport::SendRtcpPacket
 
<!--stackedit_data:
eyJoaXN0b3J5IjpbNjI5NTc4NTAyLDczMTYxNzMzOCwtNjg5MD
c0NjM1LC03OTE5OTc5OTcsMTk3NzM4MjIyOSw1NTU2MDE1Mywy
MDEzNzU0MjAxLDExNzAzNzQ1NjEsLTE0ODk0MTE4ODcsMjAyMz
UzODgxOCwzMzQ5MTU0NDZdfQ==
-->