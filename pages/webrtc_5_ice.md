当媒体协商完成后
WebRTC就开始建立网络连接了，其过程称为ice Interactive Connectivity Establishment 交互式连接建立

更确切地说，ice是在各端调用setLocalDescription()接口后就开始了，
其操作过程如下：
  收集Candidate。可连接的候选者，每个候选者是包含IP地址和端口等内容的信息集合
  交换Candiate
  按优先级尝试连接
  
Candidate
  是WebRTC用来描述它可以连接的远端的基本信息
  因此它至少包括 address、port、protocol 三元组的一个信息集，也包括CandidateType、ufrag等
  
```js
IceCandidate{
  "candidate": "udp 192.168.1.9 45845 type host ... ufrag aOj8 ...",
  "sdpMid":"0",
  "sdpMLineIndex":0
}
```
通过candidate内容可以看到 传输协议udp、ip地址、端口号、candidate类型type host、以及用户名ufrag a0j8

有了这条信息，WebRTC就可以尝试与远端进行连接了
需要注意的是，实际中使用的IceCandidate结构与WebRTC 1.0规范中定义的IceCandidate结构有很大出入
之所以会出现这种情况，主要是因为WebRTC 1.0规范出来得比较晚，各浏览器厂商还是按之前的草案来实现的

WebRTC将Candidate分成四种类型，即host、srf x、prf x及relay，
且他们还有优先级次序，其中host优先级最高，relay优先级最低

比如WebRTC收集到两个Candidate，一个是host类型，一个是srf x类型
那么WebRTC一定会先尝试与host类型的Candidate建立连接，
如果不成功，才会使用srf x类型的Candidate

收集Candidate
WebRTC收集Candidate时有几种途径：
  host类型的Candidate，是根据主机的网卡个数决定的
  一般来说，一个网卡对应一个IP地址
  给每个ipd地址随机分配一个端口从而生成一个host类型的Candidate
  
  srf x类型的Candidate，是从STUN服务器获得的IP地址和端口生成的
  
  relay类型的Candidate，是通过TRUN服务器获得的IP地址和端口生成的
  
收集到Candidate后，为了通知上层，WebRTC还在RTCPeerConnection对象中提供了一个事件，即onicecandidate
为了将收集到的Candidate交换给对端，需要为onicecandidate事件设置一个回调函数

```js
pc.onicecandidate = (e) => {
  if(e.candidate) {
    ...
  }
}
```
通过该回调函数就可以获得WebRTC底层收集到的所有Candidate了。同时，还可以在该函数中将收集到的Candidate发送给对端

交换Candidate
WebRTC收集好Candidate后，会通过信令系统将它们发送给对端
对端接收到这些Candidate后，会与本地的Candidate形成CandidatePair（即连接候选者对）。
有了CandidatePair，WebRTC就可以开始测试建立连接了

这里需要注意的是，Candidate的交换不是等所有Candidate收集好后才进行的，而是边收集边交换

尝试连接
当WebRTC形成CandidatePair后，便开始尝试进行连接
一旦WebRTC发现其中有一个可以连通的CandidatePair时，它就不再进行后面的连接尝试了，
但发现新的Candidate时仍然可以继续进行交换

