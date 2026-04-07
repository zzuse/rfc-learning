```
The following RFCs were downloaded:
   * RFC 6455 (The WebSocket Protocol) saved as sources/rfc6455.txt
   * RFC 7692 (Compression Extensions for WebSocket) saved as sources/rfc7692.txt
   * RFC 7936 (Clarifying HTTP/1.1 WebSocket usage) saved as sources/rfc7936.txt
   * RFC 8441 (Bootstrapping WebSockets with HTTP/2) saved as sources/rfc8441.txt
```

# Websocket 协议栈深度解析教程要求
请根据我提供的 RFC: sources/rfc6455.txt, sources/rfc7692.txt, sources/rfc7936.txt, sources/rfc8441.txt 为我生成一份系统性、深度与实战并重的 websocket 学习教程。

## 1. 风格与语言
语言: 全程使用简体中文。

风格: 采用“专家导读”风格，既要有协议规范的严谨性，又要具备教学的生动性。

比喻: websocket 涉及多个协议的配合，请使用统一的比喻体系

## 2. 章节规划与 RFC 映射
请将教程分为以下几个章节，必须涵盖指定的 RFC. 也可根据提供的内容的关联情况, 重新规划章节和内容和题目.

### 第一部分：穿越迷雾
WebSocket 的规范主要由 IETF（互联网工程任务组）定义，以下是几个至关重要的 RFC：
RFC 6455 (核心协议)
地位：这是 WebSocket 的主规范。
内容：定义了数据帧格式（Framing）、握手协议、关闭连接的过程、掩码（Masking）机制以及基本的错误处理。它是目前全球实现 WebSocket 的基石。

### 第二部分：压缩扩展
RFC 7692 (压缩扩展)
内容：定义了 Compression Extensions for WebSocket（通常称为 permessage-deflate）。
作用：允许在传输消息之前进行压缩，大大节省了大文本数据的带宽开销。

### 第三部分：最佳实践
RFC 7936 (最佳实践)
内容：主要讨论了在受限环境或特定拓扑结构中使用 WebSocket 的指导建议。

### 第四部分：双向快车道 —— 数据通道
RFC 8441 (HTTP/2 上的 WebSocket)
内容：规定了如何在 HTTP/2 帧中隧道传输 WebSocket。
意义：在传统的 HTTP/1.1 中，WebSocket 需要独占一个 TCP 连接；通过 RFC 8441，WebSocket 可以作为 HTTP/2 的一个流运行，实现连接复用。


## 3. 结合实战与调试 (The "Live" websocket)
调试工具与抓包:

## 4. Mermaid 图表可视化要求
必须包含以下图表来辅助理解：
websocket 数据帧格式图 (graph TD)
websocket 协议栈总览图 (graph TD)

## 5. 输出格式
请将所有内容生成为 Markdown (.md) 格式。请将教程组织到 rfcs/websocket/ 文件夹下。考虑到内容会非常多，建议将教程拆分为多个文件，例如：

rfcs/websocket/01-architecture-overview.md (协议栈总览与基础)

rfcs/websocket/02-*.md (核心协议)

rfcs/websocket/03-*.md (核心协议)

rfcs/websocket/04-*.md (压缩扩展 - RFC 7692)

rfcs/websocket/05-*.md (最佳实践 - RFC 7936)

rfcs/websocket/06-*.md (双向快车道 - RFC 8441)










