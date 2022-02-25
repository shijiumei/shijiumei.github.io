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
  STUN/TURN服务器数组。
  其中每个元素为一个RTCIceServer类型的对象。
  该字段决定了RTCPeerConnection对象都有哪些中继服务器可用
  
```js
dictionary RTCIceServer {
  required (DOMString or sequence <DOMString>) urls;
  DOMString username;
  DOMString credential;
  RTCIceCredentialType credentialType = "password";
};
```
url
  指明了服务器地址
username和credential
  指明了访问url地址时使用的用户名和密码
credentialType
  指明授权方式为密码方式，也是目前唯一的授权方式
  
iceTransportPolicy
  ICE连接策略
  该字段有两个值all或relay。
  如果指定为all，则ICE连接的过程就会尝试内网，p2p和中继
  如果为relay，则收集的Candidate只能是relay类型的Candidate
  
bundlePolicy
  该字段指定了收集Candidate的策略
  它有三个选项，分别是
  balanced
    表示按媒体类型（音频、视频）收集Candidate
    所有的音频媒体流走同一个音频传输通道，
    所有的视频媒体流走同一个视频传输通道
  max-compat
    收集Candidate时，为每个track对应一个Candidate，即每个媒体流都走自己的传输通道
  max-bundble
    所有的媒体流走同一个通道
 
rtcpMuxPolicy
  RTCP的Candidate
  之前有多个选项，但目前只保留了一项require，其含义是与RTP共用同一个Candidate
  
certificates
  端对端连接的证书
  一般不需要设置它，WebRTC内部会自己创建私有证书
  
iceCandidatePoolSize
  如果该值大于0，则WebRTC会预先生成Candidate
  
```js
var pcConfig = {
  'iceServers': [
    {
      'urls': 'turn:stun.learningrtc.cn:3478',
      'username': 'username1',
      'credential': 'password1',
    },
    {
      'urls': 'turn:stun.avdancedu.com:3478',
      'username': 'username2',
      'credential': 'passwords2',
    }
  ],
  'iceTransportPolicy': 'all',
  'bundlePolicy': 'max-bundle',
  'rtcpMuxPolicy': 'require'
};

let pc = new RTCPeerConnection(pcConfig);
```

上面的RTCPeerConnection对象中，包含了两个TURN服务器，分别是stun.learningrtc.cn和stun.avdancedu.com，
相当于为WebRTC增加了两个relay类型的Candidate
当内网和P2P无法连通时，RTCPeerConnection会尝试与这两台中继服务器连接，
如果其中一台连接成功，就不再尝试连接另一台

iceTransportPolicy字段设置为all，表示按Candidate优先级次序尝试连接
bundlePolicy设置为max-bundle，表示让所有媒体数据使用同一个Candidate，这样更有利于端口资源的利用
rtcpMuxPolicy设置为require，表示RTCP与RTP数据共用同一个Candidate

  








