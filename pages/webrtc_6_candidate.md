ICE中使用的Candidate具有优先级次序
由高到低分别
  host
  srf x
  prf x
  relay
  
WebRTC的ICE机制会选择最好的链路传输音视频数据
  如果通信双方在同一网段内，则优先使用内网链路
  如果通信双方不在同一网段，则优先使用P2P
  当以上方式都无法连通时，则使用relay服务进行中转
  
ICE的连通率几乎可以达到100%。在内网和P2P无法连通的情况下，它还可以通过中继的方式让彼此连通，从而大大提高了WebRTC的连通率

WebRTC中的ICE既考虑了数据传输的效率，又考虑了网络的连通率，同时实现起来还很简单

ICE策略
在WebRTC中，ICE的连接策略是可以定制的
构造RTCPeerConnection对象时，其输入参数的类型为RTCConfiguration。
```js
dictionary RTCConfiguration {
  sequence <RTCIceServer> iceServers;
  RTCIceTransportPolicy iceTransportPolicy;
  RTCBundlePolicy bundlePolicy;
  RTCRtcMuxPolicy rtcpMuxPolicy;
  sequence <RTCCertificate> certificates;
  [EnforceRange] octet iceCandidatePoolSize = 0;
};
```

iceServers
  STUN/TURN服务器数组法法

  








