# 使用 node 和 socket 实现在线聊天室

## 简介

在 h5 端通过 `socket` 实现在线聊天室的功能，服务端基于 `node` 实现。

该教程意在最简化的讲解 `socket`、`node` 的实现过程。所以会跳过 `h5` 静态页面开发的步骤。

![img](../gitimg/../gitImg/socket/socket8.gif)

## 一、静态页面

![img](../gitimg/../gitImg/socket/socket1.jpg)

这里尽可能的用了最简的代码实现了静态页面的开发，整个页面分成两个部分组成：

1、聊天窗口。重点讲解下聊天内容对象列表里对象的数据结构

| 参数    | 说明                               | 类型                 |
| ------- | ---------------------------------- | -------------------- |
| id      | -                                  | `string` \| `number` |
| name    | 用户名                             | `string`             |
| type    | 消息类型，可以有文字、图片、表情等 | `text` \| `img` 等   |
| message | 消息内容                           | -                    |

2、底部输入框。

[分支: feat-h5-static-page 静态页面分支](https://github.com/hang1017/socketDemo/tree/feat-h5-static-page)

## 二、建立连接

### node

在该 demo 中，服务端的功能主要做中转的消息的功能。把小明发送出去的消息，通过 node socket 发送给所有人。消息结构和内容可以保持不变。

目前开发小伙伴用的比较多的是两个库：`socket.io` 和 `ws`。

- `socket.io` 的域名是 `http://`
- `ws` 的域名是 `ws://`

所以 `socket.io` 会比较符合实际项目的需要。这里那 `socket.io` 来实现这个 demo。

项目初始化

```bash
mkdir nodePkg 

cd nodePkg

yarn init

yarn add express
yarn add node
yarn add socket.io
```

新建一个文件夹来放我们的 node 项目，项目初始化完成后，添加几个需要使用到的库。

新建 `server.js` 文件来实现功能。

```js
const express = require('express');
const http = require('http');
const socketIo = require('socket.io');

const app = express();
const server = http.Server(app);
const io = socketIo(server);

io.on('connection', function (socket) {
  console.log(socket);
});

server.listen(3000);

app.listen(8080, 'localhost', _ => {
  console.log('demo服务器已启动，访问地址http://localhost:8080')
})
```

- `connection` 当有客户端发起连接时，服务端就能通过 `connection` 建立起连接。
- `server.listen` socket 的连接端口：3000
- `app.listen(8080, '', () => {})` node 服务

**启动服务**

```bash
node server.js 
```

写完服务端，我们现在来开发下 web 端测试下是否能够成功建立连接。

### web

拉取上文 h5 分支的代码，安装需要的库。

```bash
yarn add socket.io-client
```

编写 `src/pages/index/index.tsx`, 下文代码为缩略内容

```js
import React, { FC, useEffect, useState } from 'react';
import io from 'socket.io-client';

const HomePage: FC<HomePageProps> = ({ location }) => {
  const [socket] = useState<any>(
    io('http://localhost:3000', { transports: ['websocket'] }),
  );
  console.log(socket);
}
```

> 当我们启动项目 web 项目后可能会发现连接失败的情况。并不用担心。node socket 首次启动需要时间。

![img](../gitimg/../gitImg/socket/socket2.jpg)

![img](../gitimg/../gitImg/socket/socket3.jpg)

当我们看到 web 端的控制台打印出 `connected: true`, 并且 `node` 服务端的 `on('connection')` 在终端有打印消息。那么说明：已经成功建立连接。

[分支: feat-connect 建立连接分支](https://github.com/hang1017/socketDemo/tree/feat-connect)

## 三、发送、接收消息

### node

有省略部分重复代码

```js
...

io.on('connection', function (socket) {
  socket.on('message', function (data) {
    console.log('服务端收到 : ', data);
    socket.send(data);
  });
  socket.on('error', function (err) {
    console.log(err);
  });
});

...
```

- `socket.on('message'` 这里的 `message` 要和 web 端保持一致，这样两端才能互通消息。
- `socket.send(data);` 收到的消息重新转发出去。

### web

```js
...

useEffect(() => {
  socket.on('message', (msg: any) => {
      console.log(msg);
      setChatList([...chatList, msg]);
    });
}, [socket]);

const sendMessage = () => {
  socket.emit('message', {
    id: random(),
    name,
    type: 'text',
    message: inputValue,
  });
};

...
```

- `socket.on` web 端接收 socket 消息。
- `socket.emit` 发送 socket 消息，这里注意：`'message'` 要保持一致，这样两端才能互通消息。

重启启动下 node 服务，来下效果。

![img](../gitimg/../gitImg/socket/socket4.jpg)

![img](../gitimg/../gitImg/socket/socket5.jpg)

好了，那么至此我们已经成功建立连接。但是细心的小伙伴会发现：

- 在 `socket.on('message'` 的方法中，没办法实时拿到 `chatList` 的数据，导致列表无法追加。
- 这时如果web 端再打开一个页面发送消息的话，两个网页的消息并不互通。

## 四、多人聊天室

### node

上个遗留问题中存在两个网页所连接同个 `socket` 不互通的问题，一起来解决下～

看下现在 node 端的代码会发现，每个 `socket` 建立连接后，只会在对当前的 `socket` 进行数据接收和发送，所以不同的 `socket` 不互通。

解决思路就是：我们把所有的 `socket` 连接保存起来，每次某个 `socket` 的连接只要接收到信息就遍历每个 `socket` 把消息发送出去。

```js
let socketSet = new Set(); // 存储 socket 

io.on('connection', function (socket) {
  socketSet.add(socket);
  socket.on('message', function (data) {
    socketSet.forEach(ws => {
      if (ws.connected) { // 判断当前的 socket 是否连接
        ws.send(data);
      } else {
        socketSet.delete(ws);
      }
    })
  });
  socket.on('error', function (err) {
    console.log(err);
  });
});
```

我们来验证下效果，通过下图可以看到，在右侧的聊天框中发送消息，左侧的聊天框能够接收到消息。

![img](../gitimg/../gitImg/socket/socket6.jpg)

### web

web 端我们来优化下几个小问题：

- 发送消息后聊天输入框置空
- 使用 useRef 保证聊天记录能够实时更新

以下只展示重点代码，需要看完整代码[feat-multi-chat 分支](https://github.com/hang1017/socketDemo/tree/feat-multi-chat)

```js
import React, { FC, useEffect, useState, useRef } from 'react';

const chatListRef = useRef<any[]>([]);

useEffect(() => {
  chatListRef.current = chatList;
  setChatList(chatList);
}, [chatList]);

useEffect(() => {
  socket.on('message', (msg: any) => {
    setChatList([...chatListRef.current, msg]);
    chatListRef.current = [...chatListRef.current, msg];
  });
}, [socket]);

const sendMessage = () => {
  ...

  setInputValue('');
};
```

![img](../gitimg/../gitImg/socket/socket7.jpg)

## 五、番外功能

提几个小功能点和优化点，通过文字描述来实现。

### 1、loading 状态

`chatList` 的对象数组增加一个 `loading` 的 `flag` 标签。

发送时设置为 `false`，若通过 `socket.on('message',{})` 监听到的为当前 id 发送的信息时，再把数据 `loading` 置为 `true`。

这段思路就能模拟出，发送消息以及是否发送成功的状态。

### 2、增加 `enter` 监听事件

```js

// 键盘绑定事件
const handleEnterKey = (e: any) => {
  if (e.keyCode === 13) {
    // 调用发送信息的方法
  }
};

useEffect(() => {
  document.addEventListener('keypress', (e) => handleEnterKey(e));
  return () => document.removeEventListener('keypress', (e) => handleEnterKey(e));
}, []);
```


