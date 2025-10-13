# 教程要求

所有涉及的 RFC 文件请参考 sources/ 目录下的文件，主要包括 RFC 9112 (HTTP/1.1), RFC 9110 (HTTP Semantics), 和 RFC 9111 (HTTP Caching)。

1. 风格与语言
语言: 全程使用简体中文。

风格: 采用教程风格，语言力求通俗易懂、清晰易读。对于复杂概念，请使用恰当的比喻或类比来帮助理解（例如，将 HTTP Headers 比作“包裹上的标签”，将持久连接比作“保持通话的电话线”）。

结构: 教程应有清晰的逻辑结构，从基础概念到高级主题，循序渐进。建议章节划分如下：

HTTP 核心概念 (RFC 9110 & 9112): HTTP 的前世今生、客户端-服务器模型、资源与 URI、请求与响应的基本结构。

HTTP 报文格式 (RFC 9112 & 9110): 深入剖析请求行/状态行、HTTP 头部（Headers）、消息正文（Body）的详细构成。

HTTP 语义：方法与状态码 (RFC 9110): 详细讲解核心请求方法（GET, POST, PUT, DELETE, HEAD, OPTIONS 等）的用途与幂等性，并系统梳理五大类 HTTP 状态码（1xx, 2xx, 3xx, 4xx, 5xx）的含义。

连接管理与性能优化 (RFC 9112): 重点讲解 HTTP/1.1 的核心特性，包括持久连接（Persistent Connections）、管道化（Pipelining）及其问题、分块传输编码（Chunked Transfer Encoding）。

内容协商与表示 (RFC 9110): 解释服务器如何根据客户端请求（Accept, Accept-Language, Accept-Encoding 等头部）返回最合适的内容版本。

HTTP 缓存机制 (RFC 9111): 专门讲解 HTTP 缓存的工作原理，包括强缓存（Expires, Cache-Control）和协商缓存（Last-Modified/If-Modified-Since, ETag/If-None-Match）。

2. 连接真实世界应用
这是本教程的核心要求。请将 RFC 中的抽象理论与常见的 Web 工具和服务器配置紧密结合，让读者看到“活的” HTTP。

命令行工具: 大量使用 curl 命令作为实例，并详细解释其参数。例如：

使用 curl -v 或 curl --include 展示完整的请求和响应报文，包括所有头部信息。

使用 -X 参数演示不同的 HTTP 方法（如 POST, DELETE）。

使用 -H 参数自定义请求头，以演示内容协商或缓存验证。

使用 -d 参数发送 POST 请求的数据。

浏览器开发者工具: 结合浏览器（如 Chrome 或 Firefox）的“网络 (Network)”面板截图或描述来进行说明。例如：

展示如何查看一次网络请求的 Headers, Payload, Response 和 Timing。

演示浏览器如何处理 301/302 重定向。

观察 Cache-Control 头部如何影响浏览器行为（例如，从“memory cache”或“disk cache”加载）。

Web 服务器配置: 在讲解相关概念时，引用主流 Web 服务器（如 Nginx）的配置指令作为示例。例如：

讲解持久连接时，提及 Nginx 中的 keepalive_timeout 和 keepalive_requests 指令。

讲解缓存时，展示如何用 add_header Cache-Control "public, max-age=3600"; 来设置缓存策略。

讲解内容编码时，提及 gzip on; 指令。

3. 使用 Mermaid 图表进行可视化
请为所有关键流程和复杂概念配上 Mermaid 图表，图文并茂地进行解释。必须包含但不限于以下图表：

HTTP 请求报文结构图 (graph TD)

HTTP 响应报文结构图 (graph TD)

一次完整的 HTTP/1.1 请求-响应流程时序图 (sequenceDiagram)

持久连接（Keep-Alive）与非持久连接的对比时序图 (sequenceDiagram)

分块传输编码（Chunked Transfer Encoding）的工作原理示意图 (sequenceDiagram)

浏览器基于协商缓存（ETag 和 Last-Modified）的验证流程时序图 (sequenceDiagram)

浏览器 302 重定向流程时序图 (sequenceDiagram)

基于 Accept-* 头部的内容协商流程示意图 (sequenceDiagram)

4. 深度与细节
请确保教程内容详尽，覆盖上述 RFC 的所有核心知识点。

对于重要的 HTTP 头部（如 Host, Connection, Content-Type, Content-Length, Transfer-Encoding, Cache-Control, ETag），请务必解释其确切含义和作用。

解释各种机制（如持久连接、分块传输、缓存）是为了解决 HTTP/1.0 中的什么具体问题（如高延迟、队头阻塞）而设计的。

在讲解管道化时，明确指出其理论上的优势以及在实践中几乎被弃用的原因（队头阻塞问题）。

5. 输出格式
请将所有内容生成为 Markdown (.md) 格式。

请将教程组织到 rfcs/http1.1/ 文件夹下。考虑到内容会非常多，建议将教程拆分为多个文件，例如：

rfcs/http1.1/01-introduction.md

rfcs/http1.1/02-message-format.md

rfcs/http1.1/03-semantics-methods-status.md

rfcs/http1.1/04-connection-and-performance.md

rfcs/http1.1/05-content-negotiation.md

rfcs/http1.1/06-caching.md
