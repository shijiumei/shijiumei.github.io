最简单的办法是通过状态机实现
其原理：
  每次发送/接收一个信令后，客户端都根据状态机当前的状态做相应的逻辑处理。
  
  /
    -> Init
  
  Init
    join -> joined
    
  joined
    other_join -> joined_conn
    other_leave -> joined_unbind ??????
    leave -> init
    
  joined_conn
    other_leave -> joined_unbind
    leave -> init
    
  joined_unbind 和 joined状态基本一致，只是判断用户之前的状态
    other_join -> joined_conn
    leave -> init
    
```js
var state = init;

// 连接信令服务器并根据信令更新状态机
function conn(){
  // 建立socket.io
  socket = io.connect();
  
  socket.on('joined', (roomid, id)=> {
    state = 'joined';
    ..
    createPeerConnection();
    bindTracks();
  }
  
  socket.on('otherjoin', (roomid)=> {
    state = 'joined_conn';
    ..
    call();
  }
  
  socket.on('full', (roomid, id)=> {
    hangup();
    socket.disconnect();
    state = 'init';
  }
  
  socket.on('left', (roomid, id)=> {
    hangup();
    socket.disconnect();
    state = 'init';
  }
  
  socket.on('bye', (room, id)=> {
    state = 'joined_unbind;
    hangup();
  }
  
  roomid = getQueryVariable('room');
  socket.emit('join', roomid);
}

...
conn(); // 与信令服务器建立连接
```
  
  
