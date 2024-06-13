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

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTE3MDM3NDU2MSwtMTQ4OTQxMTg4NywyMD
IzNTM4ODE4LDMzNDkxNTQ0Nl19
-->