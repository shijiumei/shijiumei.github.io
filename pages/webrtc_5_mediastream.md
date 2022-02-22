在WebRTC中有两个重要的概念
MediaStream
MediaStreamTrack

MediaStreamTrack
  称为轨，表示单一类型的媒体源，
MediaStream
  称为流，它可以包括0个或多个MediaStreamTrack
  两个作用：
  一是可以作为录制或者渲染的源，这样我们就可以将Stream中的内容录制成文件或将Stream中的数据通过浏览器中的``<video>``标签播放出来
  二是在同一个MediaStream中的MediaStreamTrack数据会进行同步，而不同MediaStream中的MediaStreamTrack之间不进行时间同步
  
当使用getUserMedia()接口获得本地音视频流MediaStream后，可以使用H5的``<video>``标签将其展示出来
要实现这个功能很简单，只要将MediaStream赋值给``<video>``标签的srcObject属性即可

```html
// h5
<html>
<body>
  <video autoplay playsinline></video>
  <script src='./demo.js'></script>
</body>
<html>
```

```js
// demo.js
const lv = document.querySelector('video');

const constrains = {
  video: true,
  audio: true
};

function gotLocalStream(mediaStream){
  lv.srcObject = mediaStream;
}

navigator.mediaDevices.getUserMedia(constrains)
  .then(gotLocalStream)
  .catch(handleLocalMediaStreamError);
```

autoplay 表示``<video>``标签收到音视频数据后立即开始播放
playsinline 的作用是让播放器在页面内播放，而不是调用外部的系统播放器播放视频




  
