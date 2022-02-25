```html
<html>
  <head>
    <title>WebRTC PeerConnection</title>
    <link href="./css/client.css" rel="stylesheet" />
  </head>
  
  <body>
    <div>
      <!-- 用户操作区 -->
      <!-- 一个用户连接信令服务器，另一个用户与信令服务器断开连接 -->
      <div>
        <button id="connserver">ConnServer</button>
        <button id="leave" disabled>Leave</button>
      </div>
      
      <!-- 显示区 -->
      <div id="preview">
        <!-- 本地视频与Offer的展示区 -->
        <div>
          <h2>Local:</h2>
          <video id="localvideo" autoplay playsinline muted></video>
          <h2>Offer SDP:</h2>
          <textarea id="offer"></textarea>
        </div>
        
        <!-- 远端视频与Answer的展示区 -->
        <div>
          <h2>Remote:</h2>
          <video id="remotevideo" autoplay playsinline></video>
          <h2>Answer SDP:</h2>
          <textarea id="answer"></textarea>
        </div>
      </div>
    </div>
    
    <!-- 引用的JavaScript脚本库 -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/2.0.3/socket.io.js"></script>
    <!-- adapter-latest.js 用于浏览器适配Chrome，Firefox -->
    <script src="https://webrtc.github.io/adapter/adapter-latest.js"></script>
    <script src="js/main.js"></script>
  </body>
</html>
```

```css
button {
  margin: 10px 20px 25px 0;
  vertical-align: top;
  width: 134px;
}

table {
  margin: 200px (50%-100) 0 0;
}

textarea {
  color: #444;
  font-size: 0.9em;
  font-weight: 300;
  height: 20.0em;
  padding: 5px;
  width: calc (100%-10px);
}

div#getUserMedia {
  padding: 0 0 8px 0;
}

div.input {
  display: inline-block;
  margin: 0 4px 0 0;
  vertical-align: top;
  width: 310px;
}

div.input > div {
  margin: 0 0 20px 0;
  vertical-align: top;
}

div.output {
  background-color: #ee;
  display: inline-block;
  font-family: 'Inconsolata', 'Courier New', monospace;
  font-size: 0.9em;
  padding: 10px 10px 10px 25px;
  position: relative;
  top: 10px;
  white-spacke: pre;
  width: 270px;
}

div.label {
  display: inline-block;
  font-weight: 400;
  width: 120px;
}

div.graph-container {
  background-color: #ccc;
  float: left;
  margin: 0.5em;
  width: calc(50%-lem);
}

div#preview {
  border-bottom: 1px solid #eee;
  margin: 0 0 1em 0;
  padding: 0 0 0.5em 0;
}

div#preview > div {
  display: inline-block;
  vertical-align: top;
}

section#statistics div {
  display: inline-block;
  font-family: 'Inconsolata', 'Corier New', monospace;
  vertical-align: top;
  width: 308px;
}

section#statistics div#senderStats {
  margin: 0 20px 0 0;
}

section#constraints > div {
  margin: 0 0 20px 0;
}

h2 {
  margin: 0 0 1em 0;
}

section#constraints label {
  display: inline-block;
  width: 156px;
}

section {
  margin: 0 0 20px 0;
  padding: 0 0 15px 0;
}

video {
  background: #222;
  margin: 0 0 0 0;
  --width: 100%;
  width: var(--width);
  height: 225px;
}

@media screen and (max-width: 720px) {
button {
  font-wight: 500;
  height: 56px;
  line-height: 1.3em;
  width: 90px;
}

div#getUserMedia {
  padding: 0 0 40px 0;
}

section#statistics div {
  width: calc ( 50% - 14px);
}
}
```

```js
use strict

// 本地视频预览窗口
var localVideo = document.querySelector('video#localvideo');
// 远端视频预览窗口
var remoteVideo = document.querySelector('video#remotevideo');

// 连接信令服务器Button
var btnConn = document.querySelector('button#connserver');
// 与信令服务器断开连接Button
var btnLeave = document.querySelector('button#leave');

// 查看Offer文本窗口
var offer = document.querySelector('textarea#offer');
// 查看Answer文本窗口
var answer = document.querySelector('textarea#answer');

var pcConfig = {
    'iceServers': [
        {
            // TURN服务器地址
            'urls': 'turn:xxx.avdancedu.com:3478',
            // TURN服务器用户名
            'username':'xxx',
            // TURN服务器密码
            'credential':'xxx'
        }
    ],
    // 默认使用relay方式传输数据
    "iceTransportPolicy": "relay",
    "iceCandidatePoolSize": "0"
};

// 本地视频流
var localStream = null;
// 远端视频流
var remoteStream = null;

// PeerConnection
var pc = null;

// 房间号
var roomid;
var socket = null;

// offer描述
var offerdesc = null;

// 状态机，初始为init
var state = 'init';

function IsPC() {
    var userAgentInfo = navigator.userAgent;
    var Agents = [
        "Android",
        "iPhone",
        "SymbianOs",
        "Windows Phone",
        "iPad",
        "iPod"
    ];
    var flag = true;
    for(var v = 0; v < Agents.length; v++){
        if(userAgentInfo.indexOf(Agents[v]) > 0){
            flag = false;
            break;
        }
    }

    return flag;
}

function IsAndroid() {
    var u = navigator.userAgent, app = navigator.appVersion;
    var isAndroid = u.indexOf('Android') > -1 || u.indexOf('Linux') > -1;
    var isIOS = !!u.match(/\(i[^;]+;(U;)?CPU.+Mac OS X/);
    if(isAndroid){
        // 这是Android系统
        return true;
    }
    if(isIOS){
        // 这是iOS系统
        return false;
    }
}

function getQueryVariable(variable){
    var query = window.location.search.substring(1);
    var vars = query.split("&");
    for(var i = 0; i < vars.length; i++){
        var pair = vars[i].split("=");
        if(pair[0] == variable){
            return pair[1];
        }
    }
    return false;
}

function sendMessage(roomid, data){
    console.log('send message to other end', roomid, data);
    if(!socket){
        console.log('socket is null');
    }
    socket.emit('message', roomid, data);
}

function conn() {
    socket = io.connect();

    socket.on('joined', (roomid, id)=> {
        console.log('recive joined message!', roomid, id);

        // 状态机变更为'joined'
        state = 'joined';

        // 如果是Mesh方案，第一个人不应该在这里创建peerConnection，而是等到所有端都收到一个'otherjoin'，消息时再创建

        // 创建PeerConnection并绑定音视频轨
        createPeerConnection();
        bindTracks();

        // 设置button状态
        btnConn.disabled = true;
        btnLeave.disabled = false;
        console.log('receive joined message, state=', state);
    });
    
    socket.on('otherjoin', (roomid)=> {
      console.log('receive joined message:', roomid, state);
      
      // 如果是多人，每加入一个人都要创建一个新的PeerConnection
      if(state === 'joined_unbind'){
        createPeerConnection();
        bindTracks();
      }
      
      state = 'joined_conn';
      
      call();
      
      console.log('receive other_join message, state=', state);
    });
    
    socket.on('full', (roomid, id)=> {
      console.log('receive full message ', roomid, id);
      
      socket.disconnect();
      hangup();
      closeLocalMedia();
      
      state = 'leaved';
      
      console.log('receive full message, state=', state);
      alert('the room is full!');
    });
    
    socket.on('leaved', (roomid, id)=> {
      console.log('receive leaved message', roomid, id);
      
      state='leaved';
      socket.disconnect();
      console.log('receive leaved message, state=', state);
      
      // 改变button状态
      btnConn.disabled = false;
      btnLeave.disabled = true;
    });
    
    socket.on('bye', (roomid, id)=> {
      console.log('receive bye message', roomid, id);
      
      // 当是Mesh方案时，应该带上当前房间的用户数，如果当前房间用户数不小于2，则不用修改状态
      // 并且关闭的应该是对应用户的PeerConnection
      // 在客户端应该维护一张PeerConnection表，它是key:value的格式，key=userid，value=PeerConnection
      
      // state = 'joined_unbind';
      
      hangup();
      
      offer.value = '';
      answer.value = '';
      console.log('receive bye message, state=', state);
    });
    
    socket.on('disconnect', (socket)=> {
      console.log('receive disconnect message!', roomid);
      
      if(!(state === 'leaved')){
        hangup();
        closeLocalMedia();
      }
      
      state = 'leaved';
    });
    
    socket.on('message', (roomid, data)=> {
      console.log('receive message!', roomid, data);
      
      if(data === null || data === undefined) {
        console.error('the message is invalied!');
        return ;
      }
      
      if(data.hasOwnProperty('type') && data.type === 'offer') {
        offer.value = data.sdp;
        
        // 进行媒体协商
        pc.setRemoteDescription(new RTCSessionDescription(data));
        
        // 创建answer
        pc.createAnswer()
          .then(getAnswer)
          .catch(handleAnswerError);
      } else if(data.hasOwnProperty('type') && data.type === 'answer') {
        answer.value = data.sdp;
        
        // 进行媒体协商
        pc.setRemoteDescription(new RTCSessionDescription(data));
      } else if(data.hasOwnProperty('type') && data.type === 'candidate') {
        var candidate = new RTCIceCandidate({
          sdpMLineIndex: data.label,
          candidate: data.candidate
        });
        
        // 将远端Candidate消息添加到PeerConnection中
        pc.addIceCandidate(candidate);
      } else {
        console.log('the message is invalid!', data);
      }
    });

    roomid = getQueryVariable('room');
    
    socket.emit('join', roomid);
    
    return true;
}

function connSignalServer() {
    // 开启本地视频
    start();
    return true;
}

function getMediaStream(stream) {
    // 将从设备上获得到的音视频track添加到localStream中
    if(localStream) {
        stream.getAudioTracks().forEach((track)=> {
            localStream.addTrack(track);
            stream.remoteTrack(track);
        });
    }else {
        localStream = stream;
    }

    // 本地视频标签与本地流绑定
    localVideo.srcObject = localStream;

    // 调用conn()函数的位置特别重要，
    // 一定要在getMediaStream调用之后再调用它，
    // 否则就会出现绑定失败的情况
    conn();
}

function handleError(err){
    console.error('Failed to get Media Stream!', err);
}

function start() {
    if(!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
        console.error('the getUserMedia is not supported!');
        return ;
    } else {
        var constraints;
        constraints = {
            video: true,
            audio: {
                echoCancellation: true,
                noiseSuppression: true,
                autoGainControl: true
            }
        };

        navigator.mediaDevices.getUserMedia(constraints)
            .then(getMediaStream)
            .catch(handleError);
    }
}

function getRemoteStream(e) {
    // 存放远端视频流
    remoteStream = e.streams[0];

    // 远端视频标签与远端视频流绑定
    remoteVideo.srcObject = e.streams[0];
}

function handleOfferError(err) {
    console.error('Failed to create offer:', err);
}

function handleAnswerError(err) {
    console.error('Failed to create answer:', err);
}

function getAnswer(desc) {
    // 设置Answer
    pc.setLocalDescription(desc);
    // 将Answer显示出来
    answer.value = desc.sdp;

    // 将Answer SDP发送给对端
    sendMessage(roomid, desc);
}

function getOffer(desc) {
    // 设置Offer
    pc.setLocalDescription(desc);
    // 将offer显示出来
    offer.value = desc.sdp;
    offerdesc = desc;

    // 将Offer SDP发送给对端
    sendMessage(roomid, offerdesc);
}

function createPeerConnection() {
    // 如果是多人的话，在这里要创建一个新的连接，新创建好的要放到一个映射表中
    // key=userid, value=peerConnection
    console.log('create RTCPeerConnection!');

    if(!pc){
        // 创建PeerConnection对象
        pc = new RTCPeerConnection(pcConfig);

        // 当收集到Candidate后
        pc.onicecandidate = (event)=> {
            if(e.candidate) {
                console.log("candidate" + JSON.stringify(event.candidate.toJSON()));

                sendMessage(roomid, {
                    type: 'candidate',
                    label: event.candidate.sdpMLineIndex,
                    id: event.candidate.sdpMid,
                    candidate: event.candidate.candidate
                });
            } else {
                console.log('this is the end candidate');
            }
        }

        // 当PeerConnection对象收到远端音视频流时，触发ontrack事件
        // 并回调getRemoteStream函数
        pc.ontrack = getRemoteStream;
    } else {
        console.log('the pc have be created');
    }

    return ;
}

function bindTracks() {
    console.log('bind tracks into RTCPeerConnection!');

    if(pc === null && localStream === undefined) {
        console.error('pc is null or underfined!');
        return ;
    }

    if(localStream === null && localStream === undefined){
        console.error('localStream is null or undefined!');
        return ;
    }

    // 将本地音视频流中所有的track添加到PeerConnection对象中
    localStream.getTracks().forEach((track) => {
        pc.addTrack(track, localStream);
    });
}

function call() {
    if(state === 'joined_conn'){
        var offerOptions = {
            offerToReceiveAudio: 1,
            offerToReceiveVideo: 1
        };

        pc.createOffer(offerOptions)
            .then(getOffer)
            .catch(handleOfferError);
    }
}

function hangup() {
    if(!pc) {
        return ;
    }

    offerdesc = null;

    pc.close();
    pc = null;
}

function closeLocalMedia() {
    if(!(localStream === null || localStream === undefined)) {
        localStream.getTracks().forEach((track)=> {
            track.stop();
        });
    }

    localStream = null;
}

function leave() {
    socket.emit('leave', roomid);

    hangup();
    closeLocalMedia();

    offer.value = '';
    answer.value = '';
    btnConn.disabled = false;
    btnLeave.disabled = true;
}

btnConn.onclick = connSignalServer
btnLeave.onclick = leave;

```












