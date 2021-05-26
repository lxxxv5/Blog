# websocket

## 为什么要有 websocket？

Websocket 之前，如果需要在客户端和服务端之间双向通信，需要通过 HTTP 轮询来实现。

HTTP 轮询分为轮询和长轮询：

    1. 轮询是指浏览器通过 JavaScript 启动一个定时器，然后以固定的时间给服务器发请求，询问服务器有没有新消息，缺点：
        - 实时性不够（在这个固定轮询时间内如果出了新消息则不能立马发送给客户端）；
        - 频繁的请求会给服务器带来极大的压力。
    2. 长轮询是指浏览器发送一个请求时，服务器先拖一段时间，等到有消息了再回复，这个机制暂时地解决了实时性问题，但也带来了新的问题：
        - 服务端将过多请求挂起等待时可能占用过多的资源；
        - 一个 HTTP 长时间没有数据传输的情况下，链路上的任何一个网关都可能关闭这个连接，而网关是我们不可控的。

## 通信方式简单介绍

1. 单向通信

只能由 A 向 B 发送数据，而 B 不能向 A 发送数据，只需要有一条信道（通信系统中的传输通道）。例如广播、打印机等。

2. 双向通信

两端可以互相发送和接收数据。一般细分为双向交替通信、双向同时通信，他们都需要两条信道（每个方向各一条）。

    - 双向交替通信（又称半双工通信）：通信的双方都可以发送信息，但不能双方同时发送，当然也不能同时接收。例如对讲机，在同一时间内只允许一方讲话。
    - 双向同时通信（又称全双工通信）：通信的双方可以同时发送和接受数据。例如电话，可以两端同时讲话和接听。

## HTTP 中的通信方式

HTTP 由于各版本中的实现不同，工作模式也不同：

1. http1.0：单工。因为是短连接，客户端发起请求后，服务端处理完请求并受到客户端的响应后即断开连接。

2. http1.1：半双工。默认开启长连接`keep-alive`，开启一个连接可发送多个请求。

3. http2.0：全双工，允许服务端主动向客户端发送数据。

## Websocket 简单介绍

Websocket 规范定义了在 web 浏览器和服务器之间建立“套接字”连接的 API。简而言之，客户端和服务器之间有一个长久的连接，双方可以随时开始发送数据。

其优点有：

1. 较少的控制开销：在连接创建后，服务器和客户端之间交换数据时，用于协议控制的数据包头部相对较小；
2. 更强的实时性：由于协议时全双工的，所以服务器可以随时主动给客户端下发数据；
3. 更好的二进制支持：Websocket 定义了二进制帧，相对 HTTP，可以更轻松地处理二进制内容；
4. 没有同源限制：客户端可以与任意服务器通信。

## Websocket 连接建立过程

1. 客户端申请协议升级（标准的 http 报文格式，只支持 get 方法）

```js
var socket = new WebSocket('ws://localhost:8088/')
socket.onopen = function (evt) {
  console.log('连接开启')
}
```

具体请求如下：

```
GET ws://localhost:8888/ HTTP/1.1
Host: localhost:8888
Connection: Upgrade  表示要升级协议
Upgrade: websocket  要升级的协议
Sec-WebSocket-Version: 13  协议版本
Sec-WebSocket-Key: xSuDlUO60IEyju+kaGrBKw== 与服务端响应首部的Sec-WebSocket-Accept是配套的，提供基本的防护，确保服务端理解websocket连接, 避免恶意\无意等非法连接。
```

2. 服务端响应协议升级

```
HTTP/1.1 101 Switching Protocols  // 101标识协议转换
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: afNQkOuXwobYwZiWaNT5viPA2jU=
到此完成协议升级，后续的数据交互都按照新的协议来。
```

## 参考链接

1. [http header 怎么判断协议是不是 websocket](https://mp.weixin.qq.com/s/Z7tYp9MLMbHKnPFyFFlJEg)
2. [双向通信之 websocket](https://juejin.cn/post/6844903839385010189)
3. [Node.js 实现 WebSocket 聊天室的例子](https://waylau.com/node.js-websocket-chat/)
