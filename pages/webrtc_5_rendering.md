远端音视频渲染
当各端将收集到的Candidate通过信令系统交换给对方后，WebRTC内部就开始尝试建立连接了

连接一旦建成，音视频数据就开始源源不断地由发送端发送给接收端

不过，此刻音视频数据即使到达了接收端，我们页看不见，听不到它，
这是因为WebRtc还不知道如何处理收到的音视频数据

事实上，在播放音视频数据之前，我们需要将远端传来的音视频数据流与本地``<video>``标签绑定才行

具体的做法是将接收到音视频数据流（MediaStream）赋值给``<video>``标签的srcObject属性
接下来的问题就是如何才能获得远端的音视频流（MediaStream）

在这方面，WebRTC给我们提供了一个非常好的接口，即RTCPeerConnection对象的ontrack()事件

每当有远端的音视频数据传过来时，ontrack()事件就会被触发，因此你只需要给ontrac()事件设置一个回调函数
就可以拿到远端的MediaStream了

```js
function getRemoteStream(e) {
}

let pc = new RTCPeerConnection(...);

pc.ontrack = getRemoteStream();
```
