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
```












