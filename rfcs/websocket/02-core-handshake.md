# 02. 核心协议：握手与生命周期 —— 列车的升级与调度 (RFC 6455)

要让一条普通的 HTTP 铁轨（TCP 连接）变成 WebSocket 高速双向货运专列，需要经过一次严谨的“升级调度”仪式。在 RFC 6455 中，这个过程被称为**握手（Handshake）**。

---

## 1. 列车升级申请：HTTP Upgrade 机制

WebSocket 复用了 HTTP 的默认端口（80 为 ws，443 为 wss）。这意味着，初始的连接建立看起来和一个普通的网页请求（GET）一模一样。这就是我们的**客运列车**。

但在这个 GET 请求的头部，客户端悄悄塞进了几张“升级申请表”：

**客户端请求（Client Request）：**
```http
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

**关键请求头解析：**
- `Connection: Upgrade` 和 `Upgrade: websocket`：客户端在大喊：“我不想坐客运列车了！我要把这条铁轨升级成 WebSocket 货运专列！”
- `Sec-WebSocket-Version: 13`：表明客户端支持的 WebSocket RFC 规范版本（13 是目前唯一广泛使用的版本）。
- `Sec-WebSocket-Key`：这是一串随机生成的 Base64 编码字符串，相当于客户端给服务器出的一道**防伪口令**。

---

## 2. 服务器的批准与防伪校验

服务器收到申请后，如果支持 WebSocket，它不能简单地回一句“好的”。为了防止某些恶意的缓存服务器（Proxy）搞混，服务器必须通过一套严格的数学公式，计算出针对 `Sec-WebSocket-Key` 的答案。

**数学公式计算过程：**
1. 将客户端发来的 `Sec-WebSocket-Key` 与一个全局唯一的魔法字符串（GUID）拼接：`258EAFA5-E914-47DA-95CA-C5AB0DC85B11`。
   拼接后：`dGhlIHNhbXBsZSBub25jZQ==258EAFA5-E914-47DA-95CA-C5AB0DC85B11`
2. 对拼接后的字符串进行 **SHA-1** 哈希计算。
3. 将 SHA-1 的结果进行 **Base64** 编码。

**服务器响应（Server Response）：**
```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

**关键响应头解析：**
- `HTTP/1.1 101 Switching Protocols`：状态码 101 代表“协议切换成功”。
- `Sec-WebSocket-Accept`：这就是服务器算出来的正确答案！客户端核对答案无误后，**HTTP 请求正式结束，WebSocket 货运专列启动！**

从此以后，这条连接上将只跑 WebSocket 格式的二进制帧（Frames），再也没有 HTTP 头部了。

---

## 3. 列车的调度指令：Ping 与 Pong

在漫长的铁道运输中，有时候几个小时都没有货物往来。此时，沿途的隧道管理员（防火墙、NAT、代理服务器）如果看到一条铁轨久久没有动静，可能会强行把铁轨拆掉（切断连接）。

为了保持连接活跃，RFC 6455 定义了**心跳机制（Heartbeat）**：
- **Ping 帧**：一端可以随时发送一个 Ping 帧（查岗信号：“兄弟，你还在吗？”）。
- **Pong 帧**：另一端收到 Ping 后，**必须**尽快回复一个内容完全相同的 Pong 帧（响应信号：“我还在，一切正常！”）。

这种心跳机制是极其轻量级的，就像列车员在铁轨上敲击了一下，只发出很小的声响。

---

## 4. 终点站：优雅地关闭连接 (Closing Handshake)

当业务结束，货运专列需要停运时，直接拔网线（直接断开 TCP）是非常粗暴且容易丢失数据的行为。WebSocket 提供了一个**优雅关闭握手（Closing Handshake）**流程：

1. **发起方**发送一个 **Close 帧**，里面可以包含一个状态码（例如 1000 表示正常关闭）和一段简短的原因说明。
2. **接收方**收到 Close 帧后，立刻明白对方不想发数据了。它处理完手上最后的货物后，也必须回发一个 **Close 帧**。
3. 发起方收到回发的 Close 帧后，就可以安全地拆除底层的 TCP 铁轨（关闭 TCP 连接）。

---

## 5. 实战与调试 (The "Live" WebSocket)

**如何通过 Chrome DevTools 观察握手？**

1. 打开 Chrome 开发者工具（F12），切换到 **Network (网络)** 面板。
2. 过滤条件选择 **WS**。
3. 刷新页面或触发 WebSocket 连接。
4. 点击该请求，在 **Headers** 标签页，你可以清晰地看到 `Sec-WebSocket-Key` 和 `Sec-WebSocket-Accept` 的一对一映射。这证明握手成功了！
5. 切换到 **Messages** 标签页，你将看到这条专列上装载的所有货物（数据帧）的详细往来记录（绿箭头是发送，红箭头是接收）。

下一章，我们将撬开货运列车上的“集装箱”，看看 WebSocket 是如何高效打包和保护数据的！