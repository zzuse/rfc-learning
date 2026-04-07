# 04. 压缩扩展 —— 货物真空打包技术 (RFC 7692)

在高速货运专列（WebSocket）上，我们已经拥有了轻量级的标准化集装箱（Data Framing）。但是，如果我们频繁地发送巨大的 JSON 数据包、XML 或者大段的文本消息，带宽消耗依然非常可观。

这时候，我们需要引入一台**“真空打包机”**，在把货物装进集装箱前，先将空气抽干，极大减小体积。这就是 **RFC 7692: Compression Extensions for WebSocket** 所定义的内容。

---

## 1. 扩展机制：`permessage-deflate`

WebSocket 协议本身是一个可扩展的框架。RFC 7692 定义了目前最著名、应用最广泛的扩展：`permessage-deflate`。

`deflate` 是一种广为人知的无损数据压缩算法（结合了 LZ77 算法和哈夫曼编码，这也是 gzip 的底层算法）。`permessage` 意味着这种压缩是**基于完整消息（Message）**级别的，即使这个消息被分片成了多个数据帧（Frames）。

### 为什么不直接在应用层（如 JS）压缩？
如果在代码里手动用 pako.js 等库进行压缩再发送，确实可行，但这会带来几个问题：
1. 双方都需要额外编写大量的压缩和解压代码。
2. 无法利用流式处理，影响性能。
3. **无法共享上下文字典**（Context Takeover，稍后重点讲解）。

交给底层 WebSocket 协议来处理，对上层应用是完全透明的。开发者依然只是 `socket.send(JSON.stringify(data))`，但底层已经帮你做好了极致的压缩。

---

## 2. 安装“真空打包机”：扩展协商握手

就像买票时确认是否需要特殊服务一样，压缩扩展必须在初始的 HTTP 握手阶段进行协商。

**客户端申请（Client Request）：**
客户端在请求头中加入：
```http
Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits
```
*含义：我想使用 `permessage-deflate` 扩展，并且请告诉我你能接受的窗口大小。*

**服务器同意（Server Response）：**
如果服务器也支持并且同意使用该扩展，它会在响应头中回执：
```http
Sec-WebSocket-Extensions: permessage-deflate; client_no_context_takeover; server_no_context_takeover
```
*含义：同意使用！但为了节省服务器内存，我们双方都不要保留上下文（no context takeover）。*

---

## 3. 上下文接管（Context Takeover）：压缩的黑科技

这是 RFC 7692 中最硬核的特性。

通常我们用 zip 压缩一个文件，它是利用文件内部的重复字符串来进行替换。但如果每次 WebSocket 发送的消息都很短（比如聊天消息），单次消息内的重复率很低，压缩效果极差。

`Context Takeover` 机制允许压缩器和解压器**跨越多个消息保留滑动窗口字典（Sliding Window）**。
- 假设第一条消息发了："Hello, this is a very long and repetitive JSON structure."
- 这句话中的字符串会被加入到底层的字典树中。
- 当发送第二条消息："Bye, this is a very long and repetitive JSON structure." 时，压缩器发现后面一大串之前出现过！它只需要发送几个字节的指针，指向第一条消息在字典中的位置即可。

**惊人的效果**：在传输结构高度相似的实时数据流（如股票行情、系统状态 JSON）时，**压缩率经常可以达到 80% 甚至 90% 以上！**

**代价与妥协**：
维持滑动窗口意味着每个 WebSocket 连接，服务器都要在内存中保留一份压缩上下文（通常是几十 KB）。如果你有一百万个并发连接，这就是几十 GB 的内存开销！
因此，RFC 7692 提供了几个参数来妥协：
- `server_no_context_takeover` / `client_no_context_takeover`：禁用上下文接管，每次发消息都重置字典。省内存，但压缩率下降。
- `server_max_window_bits` / `client_max_window_bits`：缩小滑动窗口的尺寸（默认15，代表 32KB。可以调小到 8，代表 256 bytes），平衡内存与压缩率。

---

## 4. 集装箱上的特殊标记：`RSV1` 位

在上一章的数据帧图谱中，我们提到了 `RSV1` 预留位。一旦启用了 `permessage-deflate`，这个原本闲置的位就派上了大用场。

- 对于一列火车（一个完整的 Message），只有它的**第一个数据帧（First Frame）**的 `RSV1` 位会被置为 `1`。
- `RSV1 = 1` 等同于在集装箱上贴了一张醒目的标签：**【警告：内含真空压缩货物，请解压后再开箱】**。
- 如果是不需要压缩的消息（比如短促的心跳内容），`RSV1` 依然是 `0`，原样传输。

---

## 5. 实战与调试 (The "Live" WebSocket)

**如何验证你的网站是否开启了 WebSocket 压缩？**

1. 打开 Chrome DevTools -> **Network**。
2. 找到你的 `101 Switching Protocols` 响应。
3. 检查 **Response Headers** 中是否包含了 `Sec-WebSocket-Extensions: permessage-deflate`。
4. 如果有，恭喜你，你的数据跑在了省带宽的绿色通道上！
5. 在 **Messages** 标签页中，你可以看到 `Length` 这一列。如果是压缩过的数据，你能看到实际在网络上只传输了非常少的字节数，但解压后的文本又很长。

了解了基础协议与压缩技术，我们的货运专列似乎已经完美了。但在现实的互联网中，沿途有着无数的隧道、防火墙和负载均衡器，它们常常对我们的专列造成干扰。下一章，我们将研读 **RFC 7936 最佳实践**，学习如何在复杂地形下平稳驾驶。