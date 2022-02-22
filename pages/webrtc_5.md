对设备进行检测

```js
navigater.mediaDevices.enumerateDevices();
```
``enumerateDevices()``接口执行成功后，会返回deviceInfo数组。
数组中的每一项为一个deviceInfo对象

```js
interface MediaDeviceInfo {
  readonly attribute DOMString deviceId;
  readonly attribute MediaDeviceKind kind;
  readonly attribute DOMString label;
  readonly attribute DOMString groupId;
};

enum MediaDeviceKind {
  "audioinput",
  "audiooutput",
  "videoinput"
};
```

- ``deviceId``表示每个设备的唯一编号，通过该编号可以从WebRTC的音视频设备管理找到该设备。
- ``kind``表示设备的种类。音视频设备包括三种类型： 音频输入设备、音频输出设备以及视频输入设备
  音频的输入设备和输出设备是两种不同类型的设备。
  而对视频设备来说，它只有输入设备，
  视频的输出则是由显示器完成的。
  由于显示器是默认设备，所以不需要通过音视频设备管理器进行管理
- ``label``是设备的名字，该名字是便于人民记忆的名字，不像deviceId那样是一串毫无规律的字符串。
- ``groundId``表示组Id。如果两个设备是在同一个硬件上，则他们属于同一组，因此他们的``groundId``是一致的，例如音频的输入与输出设备就是集成到一起的。


```js
// 如果遍历设备失败，则回调该函数
function handleError(error){
  console.log('err:', error);
}

// 如果得到音视频设备，则回调该函数
function gotDevices(deviceInfos) {
  ...
  // 遍历所有设备信息
  for(let i = 0; i !== deviceInfos.length; ++i){
    // 取每个设备信息
    const deviceInfo = deviceInfos[i];
    ...
  }
  ...
}

// 遍历所有音视频设备
navigator.mediaDevices.enumerateDevices()
  .then(gotDevices)
  .catch(handleError);
```
这里需要注意的是，基于安全方面的原因，浏览器有可能不允许调用enumerateDevices()函数

采集音视频数据
在浏览器下采集音视频数据也很方便，调用``getUserMedia()``这个API就可以采集到
```
navigator.mediaDevices.getUserMedia(MediaStreamConstrains);
```
Constrains 约束
该接口有一个MediaStreamConstrains类型的输入参数，可以用来控制从哪个设备上采集音视频数据，以及限制采集到的数据的格式

```js
dictionary MediaStreamConstrains {
  (boolean) or (MediaTrackConstrains) video = false;
  (boolean) or (MediaTrackConstrains) audio = false;
}
```
因此，我们既可以直接给video和audio赋值true/false,简单地指明是否采集视频或音频数据，
也可以给它赋值一个MediaTrackConstrains类型的值，对音视频设备做更精准的设置

如果直接给video/audio属性赋值true,则浏览器会使用默认设备和默认参数采集音视频数据

```
dictionary MediaTrackConstraintSet {
  // 视频相关
  ConstrainULong width; // 分辨率
  ConstrainULong height;
  ConstrainDouble aspectRatio; // 宽高比
  ConstrainDouble frameRate; // 帧率
  ConstrainDOMString facingMode; // 前置、后置摄像头
  ConstrainDOMString resizeMdoe; // 缩放、裁剪
  
  // 音频相关
  ConstrainULong sampleRate; // 采样率
  ConstrainULong sampleSize; // 采样大小
  ConstrainBoolean echoCancellation; // 是否开启回音消除
  ConstrainBoolean autoGainControl; // 是否开启自动增益
  ConstrainBoolean noiseSuppression; // 是否开启降噪
  ConstrainDouble latency; // 目标延迟
  ConstrainULong channelCount; // 声道数
  
  // 设备相关
  ConstrainDOMString deviceId;
  ConstrainDOMString groupId;
};


// 采集到某路流
function gotMediaStream(stream){
  ...
}

// 从设备选项栏里选择某个设备
var deviceId = xxx;

// 设置采集限制
var constraints = {
  video:{
    width: 640,
    height: 480,
    frameRate: 15,
    facingMode: 'enviroment',
    deviceId: deviceId?{exact:deviceId}: undefined
  },
  audio: false
}

// 开始采集数据
navigator.mediaDevices.getUserMedia(constraints)
  .then(gotMediaStream)
  .catch(handleError);
```


