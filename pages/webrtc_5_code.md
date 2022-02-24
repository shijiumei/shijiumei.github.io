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

    
}



```












