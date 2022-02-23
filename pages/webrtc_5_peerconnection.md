RTCPeerConnection对象时WebRTC的核心
它是WebRTC暴露给用户的统一接口
内部由多个模块组成
  网络处理模块
  服务质量模块
  音视频引擎模块
  
可以把它想象成一个超级socket
通过它可以轻松地完成端到端数据的传输

它还可以根据网络情况动态调整最佳的服务质量

创建RTCPeerConnection对象
```js
const configuration = {
  iceServers: [
    { urls: 'stun:stun.example.org' }
  ]
};

let pc = new RTCPeerConnection(configuration);
```

通信时是以中继方式传输数据，还是使用P2P方式传输数据？

如果使用中继方式，那它可以使用哪些中继服务器？

RTCPeerConnection与本地音视频数据绑定
对于绑定数据的问题，RTCPeerConnection对象为我们提供了两种方法：
  一个是addTrack()
  另一个是addStream() 过时
  
这两种方法都可以实现将采集到的数据与RTCPeerConnection绑定的作用
不过由于WebRTC规范中已经将addStream()标记为过时，因此建议尽量使用addTrack()方法，以免以后出现兼容性问题

当客户端收到joined消息后，它会创建RTCPeerConnection对象，
然后调用bindTracks()函数将其与之前通过getUserMedia()接口采集到的音视频数据绑定到一起

```js
function bindTracks() {
  ls.getTracks().forEach((track)=>{
    pc.addTrack(track, ls);
  });
}
```

ls是一个全局变量
  当通过getUserMedia()接口采集到MediaStream后，需要交其由ls管理
  
pc是RTCPeerConnection的缩写，也是一个全局变量
  当RTCPeerConnection创建好后，交由pc管理
这样当调用bindTracks()函数时，它就可以从ls中获取每个准备好的track，然后将其加入RTCPeerConnection对象中，
从而实现了音视频数据与RTCPeerConnection对象绑定的工作














