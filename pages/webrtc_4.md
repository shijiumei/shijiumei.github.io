要实现WebRTC一对一通信，信令服务器是重要的一环

没有信令服务器，通信的双方就好像站在两个孤立的岛上，彼此无法见到对方

信令服务器的作用主要有两个：
一是实现业务层的管理
  用户创建房间
  加入房间
  退出房间
  等
二是让通信的双方彼此交换信息
  最常见的是交换通信双方的IP地址和端口
  
WebRTC一对一架构

WebRTC由四部分组成，分别为
  两个WebRTC终端、
  一个信令服务器、
  一台中继服务器（STUN/TURN）、
  和两个NAT
  
这是最经典的一对一通信架构

STUN Session Traversal Utilities for NAT
NAT 会话穿越实用工具协议

TURN Traversal Using Relay NAT
通过中继方式穿越NAT

NAT Network Address Translation
网络地址转换

其中，信令服务器与中继服务器都在NAT外，也就是属于外网。
而两个WebRTC终端在NAT内，属于内网。


通信流程：
  通信前
    A-登录->信令服务器
    B-登录->信令服务器
  
    A<-获取外网IP和端口-STUN/TURN
    B<-获取外网IP和端口-STUN/TURN
    
    A-将A的外网IP和端口->通过信令服务器->B
    B-将B的外网IP和端口->通过信令服务器->A
    
  彼此获得对方地址后
    就可以尝试NAT穿越了，进行P2P连接


Call ---> Called 主叫 被叫

在Call端内部
  首先，调用音视频设备检测模块检测终端是否有可用的音视频设备
  然后，调用音视频采集模块从设备采集音视频数据
  采集到数据后，开启客户端录制（是否开启录制是可选的，用户可以根据自己的需求选择录制或不录制）
  当数据采集相关的工作就绪后，通过信令模块与信令服务器建立连接
  紧接着创建RTCPeerConnection对象（RTCPeerConnection对象时WebRTC最核心的对象，后面音视频数据的传输都靠它来完成）
  RTCPeerConnection创建好后，系统要先将它与之前采集的音视频数据绑定到一起，这样RTCPeerConnection才知道从哪里获取要发送的数据
  
  RTCPeerConnection创建socket连接的过程：
    首先，向STUN/TRUN服务器发送请求
    STUN/TURN服务器收到Call的请求，会将Call的外网IP地址和端口号作为应答消息返回去
    之后终端，通过信令服务器将Call的连接地址发送给对端
    同理，Called也会将它的IP地址和端口发给Call
    当通信双方都获得对端的地址后，此时socket连接就被建立起来了
    
信令定义
  当收到用户成功加入房间的信令后
    系统需要立即将RTCPeerConnection对象创建好，
    以便向STUN/TURN服务器请求其外网的IP地址和端口
  当收到另一个用户加入房间的消息时
    系统需要将自己的外网IP地址和端口交换给对方
    从而建立起socket连接
    
第一类客户端发给服务端的信令
  join          -用户加入房间
  leave         -用户离开房间
  message       -端到端命令（offer、answer、candidate）
    
第二类服务端发送给客户端的信令
  joined        -用户已加入
  left          -用户已离开
  other_joined  -其他用户已加入
  bye           -其他用户已离开
  full          -房间已满
    
信令传输协议的选择
  一般选择TCP或基于TCP的Http/https、WS/WSS等协议作为信令服务器的传输协议
  好处：
    一是不用担心信令丢失，
    因为TCP是可靠的传输协议，能保证传输的数据可靠、有序到达
    二是在TCP上传输的数据是流式的
    因此不必担心传输的数据过大导致拆包传输的问题
    
构建信令服务器
  一是信令服务器的实现方案
  二是信令服务器的业务逻辑
  三是信令服务器的实现
  四是信令服务器的安装与部署
  
信令服务器的实现方案
  方案一
    使用C/C++、Java等语言从零开始开发一个信令服务器
    成本高
  方案二
    利用现成的Web服务器做应用开发，
    如Apache、Nginx、NodeJs为服务，
    在其上做应用开发是非常不错的选择
    
  建议采用第二种方案
    - 一般信令系统都需要使用Http/https、WS/WSS等传输协议，
    而Apache、Nginx、NodeJS等服务器显然在这方面有天然的优势
    - 实时通信的信令服务器一般负载都不是特别高
    举个例子，假设有1w个房间同时在线，我们可以评估出大部分房间只需要处理几个信令
    那么总的消息量也不过是几万个
    这个量级对于Nginx和NodeJS来说，单台服务器就可以应付了
    - 通过Nginx或NodeJS实现信令服务器特别简单，只需几行代码就可以实现
    - 稳定性高
    像Apache、Nginx、NodeJS这类服务器都经过长时间的验证，所以它们的稳定性是可以得到保障的
    
    
信令服务器的实现
```
  const http = require('http'); // 引入http库
  const express = require('express'); // 引入express库
  
  const app = express(); // 创建http服务，并侦听8980端口
  const http_server= http.createServer(app);
  http_server.listen(8080, '0.0.0.0');
  
  发送消息 socket.emit('cmd');
  发送带参消息 socket.emit('cmd', arg1..);
  发给房间其他人消息 socket.to(room).emit('cmd');
  接收消息 socket.on('cmd', function(){});
  接收带参消息 socket.on('cmd',function(arg1){});
  
  io.sockets.on('connection', (socket) => {
    // 收到message时，进行转发
    socket.on('message',（message)=> {
      socket.to(room).emit('message', message);
    }
    
    // 收到join消息
    socket.on('join', (room)=> {
      var o = io.sockets.adapter.rooms[room];
      
      // 得到房间的人数
      var nc = o ? Object.keys(o.sockets).length : 0;
      
      if(nc < 2){
        socket.join(room);
        socket.emit('joined', room);
      } else {
        socket.emit('full', room);
      }
    }
  }
```

信令服务器的安装与部署

安装NodeJs
  ubunt ： apt install nodejs
  centos: yum install nodejs
  macos: brew install nodejs
  
安装npm Node package manager
  ubunt: apt install npm
  
  安装好npm后，用它来安装NodeJs的依赖库express和socket.io库（http库是NodeJs自带的，不需要安装）
  npm install socket.io@2.0.3
  npm install express
  
现在代码编写好了，运行环境也搭建好了，接下来就可以启动服务了
  node sigserver.js
  此时，你可以在控制终端上执行以下命令，来观察信令服务器是否正常启动
  netstat -ntpl | grep 8080
  
  
  
  
  
  
  
  
  
    
    
