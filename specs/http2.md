# 教程要求

核心目标： 创建一个关于 HTTP/2 的综合性教程，不仅深入讲解其核心机制，更要与 HTTP/1.1 进行持续对比，阐明 HTTP/2 解决了哪些根本性问题。所有涉及的 RFC 文件请参考 sources/ 目录下的文件，主要包括 RFC 9113 (HTTP/2), RFC 7541 (HPACK), 以及作为语义基础的 RFC 9110 (HTTP Semantics)。

1. 风格与语言
语言: 全程使用简体中文。

风格: 采用教程风格，语言力求通俗易懂、清晰易读。对于复杂概念，请使用恰当的比喻或类比来帮助理解。

例如：将 HTTP/2 的多路复用比作“在一条超宽的高速公路上，让多辆货车（数据流）同时并行运输，而不是排队等待通过一个收费站”。

将二进制分帧比作“将原本散乱的信件（文本协议）统一装入标准化的、贴好标签的快递箱（二进制帧）中，便于机器高效处理”。

将 HPACK 头部压缩比作“和老朋友打电话，后面就不需要每次都说‘我是张三’，因为对方已经知道并记住了”。

结构: 教程应有清晰的逻辑结构，从 HTTP/1.1 的痛点出发，引出 HTTP/2 的解决方案，再深入其内部机制。建议章节划分如下：

HTTP/2 诞生背景： 回顾 HTTP/1.1 的核心问题（队头阻塞、连接数限制、头部冗余），引出 HTTP/2 (源自 SPDY) 的设计目标。

核心概念：二进制分帧层 (Binary Framing Layer): 这是 HTTP/2 的基础。详细解释连接 (Connection)、流 (Stream)、消息 (Message) 和 帧 (Frame) 之间的关系，并介绍几种核心帧类型（DATA, HEADERS, SETTINGS, PING, GOAWAY 等）。

杀手级特性：多路复用 (Multiplexing): 深入讲解 HTTP/2 如何通过在单个 TCP 连接上并行传输多个流来解决队头阻塞问题。与 HTTP/1.1 的持久连接和管道化进行详细对比。

性能的极致优化：头部压缩 (HPACK): 专门讲解 RFC 7541 的 HPACK 机制。解释其工作原理，包括静态表 (Static Table)、动态表 (Dynamic Table) 和霍夫曼编码 (Huffman Coding)，并阐明它相比 Gzip 压缩头部的优势。

服务端主动推送 (Server Push): 讲解 Server Push 的工作流程和适用场景，以及它如何减少请求的往返时间 (RTT)。

流量与资源控制：流控与优先级 (Flow Control & Priority): 解释 HTTP/2 如何通过 WINDOW_UPDATE 帧进行流量控制，以及如何通过流依赖关系和权重设置请求优先级。

协议的协商与升级： 讲解客户端和服务器如何通过 ALPN (Application-Layer Protocol Negotiation) 协商使用 HTTP/2。

2. 连接真实世界应用
将 RFC 中的抽象理论与现代 Web 工具和服务器配置紧密结合，让读者看到“活的” HTTP/2。

命令行工具: 大量使用支持 HTTP/2 的 curl 命令作为实例，并详细解释其参数。

使用 curl -v --http2 <https://example.com> 或 nghttp2 -nv <https://example.com> 来观察 HTTP/2 的连接过程和帧交换。

展示如何观察 ALPN 协商过程。

演示在单个连接上同时请求多个资源的命令行操作，以体现多路复用。

浏览器开发者工具: 结合浏览器（如 Chrome 或 Firefox）的“网络 (Network)”面板截图或描述来进行说明。

展示如何在网络面板中确认请求正在使用 h2 协议。

通过瀑布图（Waterfall）清晰地展示多个资源在同个 TCP 连接上并行加载的景象，并与 HTTP/1.1 的瀑布图进行对比。

演示如何识别由 Server Push 推送的资源。

观察 HTTP/2 的伪头部字段（:method, :scheme, :authority, :path）。

Web 服务器配置: 在讲解相关概念时，引用主流 Web 服务器（如 Nginx）的配置指令作为示例。

讲解启用 HTTP/2 时，展示 Nginx 的 listen 443 ssl http2; 指令。

讲解 Server Push 时，展示如何使用 http2_push 和 http2_push_preload 指令。

提及与流控和超时相关的配置，如 http2_body_preread_size。

3. 使用 Mermaid 图表进行可视化
请为所有关键流程和复杂概念配上 Mermaid 图表，图文并茂地进行解释。必须包含但不限于以下图表：

HTTP/1.1 队头阻塞示意图 (sequenceDiagram): 展示一个请求阻塞后续请求的场景。

HTTP/2 多路复用时序图 (sequenceDiagram): 在同一个 participant (TCP Connection) 上，清晰地展示来自不同流 (Stream) 的帧 (Frames) 是如何交错传输的。

HTTP/2 核心概念关系图 (graph TD): Connection --> Stream --> Message --> Frame

二进制帧结构图 (graph TD): 展示一个通用帧的 Length, Type, Flags, Stream Identifier, Frame Payload 结构。

HPACK 动态表示例图 (sequenceDiagram or graph TD): 简要示意客户端和服务器如何通过交换 HEADERS 帧来同步更新其动态表。

Server Push 完整流程时序图 (sequenceDiagram): 从客户端请求 HTML 开始，到服务器响应 PUSH_PROMISE 和相应的数据流。

ALPN 协议协商过程时序图 (sequenceDiagram): 展示在 TLS握手期间，客户端 ClientHello 中的 ALPN 扩展如何被服务器识别和确认。

4. 深度与细节
请确保教程内容详尽，覆盖上述 RFC 的所有核心知识点。

对比是关键： 在讲解每一个 HTTP/2 特性时，务必回顾并对比 HTTP/1.1 的对应机制或缺失。例如：

二进制分帧 vs 文本协议：解释为什么二进制对机器更友好、更不易出错。

多路复用 vs 管道化：解释为什么多路复用从根本上解决了队头阻塞，而管道化只是一个有缺陷的尝试。

HPACK vs Header Gzip：解释 HPACK 如何通过维护上下文来获得更高的压缩率和安全性（避免 CRIME 攻击）。

Server Push vs HTTP/1.1 的内联 (Inlining)：对比两种优化方式的优缺点。

核心概念辨析： 明确解释 HTTP Semantics (RFC 9110) 中定义的请求方法、状态码、头部字段等在 HTTP/2 中依然有效，HTTP/2 改变的是“传输层”或“语法层”，而非“应用语义层”。

伪头部字段： 详细解释 :method, :scheme, :authority, :path 这四个伪头部字段的作用，以及它们为何取代了 HTTP/1.1 的请求行。

5. 输出格式
请将所有内容生成为 Markdown (.md) 格式。

请将教程组织到 rfcs/http2/ 文件夹下。考虑到内容会非常多，建议将教程拆分为多个文件，例如：

rfcs/http2/01-why-http2-and-intro.md

rfcs/http2/02-binary-framing-layer.md

rfcs/http2/03-multiplexing-vs-hol.md

rfcs/http2/04-header-compression-hpack.md

rfcs/http2/05-server-push-and-advanced-features.md

rfcs/http2/06-real-world-and-migration.md
