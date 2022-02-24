SDP与Candidate消息交换

客户端发送消息
```js
function sendMessage(roomid, data){
  socket.emit('message', roomid, data);
}
```

服务端收到消息后转发
```js
socket.on('message', (room, data)=> {
  socket.to(room).emit('message'), room, data);
});
```

客户端接收消息
```js
socket.on('message', (roomid, data)=> {
  if(data.hasOwnProperty('type') && data.type === 'offer'){
    ...
  } else if(data.hasOwnProperty('type') && data.type === 'answer'){
    ...
  } else if(data.hasOwnProperty('type') && data.type === 'candidate'){
    ...
  }
});
```
