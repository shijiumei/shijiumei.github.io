媒体协商
当RTCPeerConnection对象与音视频绑定后，紧接着需要进行媒体协商

媒体协商
  通信的双方在真正通信之前，以了解彼此都有哪些能力
  
进行媒体协商时，交换的内容是sdp格式

在WebRTC中，媒体协商是有严格的协商顺序的
共8步：
  A：
  调用RTCPeerConnection对象的createOffer接口生成SDP格式的本地协商信息Offer
  本地协商信息Offer生成后，再调用setLocalDescription接口，将Offer保存起来
  之后通过客户端的信令系统将Offer信息发送给远端用户B
  B:
  用户B收到Offer信息后，调用本地RTCPeerConnection对象的setRemoteDescription接口，将Offer信息保存起来
  再调用createAnswer接口创建Answer消息，Answer消息也是SDP格式，里面记录的是B端的协商信息
  Answer消息创建好后，用户B调用setLocalDescription接口将Answer信息保存起来
  用户B将Answer消息发送给A端，以便让用户A继续完成自己的媒体协商
  A:
  用户A收到用户B的Answer消息后，，调用setRemoteDescription接口将收到Answer消息保存起来
  
执行完这些后，整个媒体协商过程才算最终完成
