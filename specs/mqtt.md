## Instructions MQTT protocol
```
MQTT Version 5.0 specification, an open and lightweight messaging transport protocol designed specifically for the Internet of Things (IoT) and constrained network environments. 
```

### 教程要求
请根据我提供的列表 rfc9431.txt, https://docs.oasis-open.org/mqtt/mqtt/v3.1.1/mqtt-v3.1.1.html, https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html，为我生成一份系统性、深度与实战并重的 MQTT 学习教程。

### 风格与语言
语言: 全程使用简体中文。
风格: 采用教程风格，语言力求通俗易懂、清晰易读。对于复杂概念，请使用恰当的比喻或类比来帮助理解（例如，将发布/订阅模式比作“报纸的订阅与投递系统”，将共享订阅比作“银行柜台的排队叫号系统”）。

### 结构
教程应有清晰的逻辑结构，从基础概念到高级主题，循序渐进。建议章节划分如下：

#### MQTT 核心概念与架构
发布/订阅（Pub/Sub）模型、客户端与代理（Broker）架构、主题（Topic）与通配符机制。

#### 连接与会话管理 (MQTT 5.0)
CONNECT/CONNACK 报文交互、保活机制（Keep Alive）、遗嘱消息（Will Message）。深入讲解 MQTT 5.0 的“清理起始”（Clean Start）与“会话过期间隔”（Session Expiry Interval）如何取代 3.1.1 的 Clean Session。

#### 服务质量与可靠传递
深入解释 QoS 0、1、2 的工作原理与四次握手（PUBLISH, PUBACK, PUBREC, PUBREL, PUBCOMP）、消息过期间隔（Message Expiry Interval）和保留消息（Retained Messages）。

#### MQTT 5.0 高级与高性能特性
专门讲解 MQTT 5.0 的新特性，包括原因码（Reason Codes）、共享订阅（Shared Subscriptions）、主题别名（Topic Alias）、用户属性（User Properties）、请求/响应模式（Request/Response），以及流量控制（Receive Maximum）。

#### 安全、身份验证与授权 (RFC 9431)
结合 ACE（Authentication and Authorization for Constrained Environments）框架，详细讲解如何在 MQTT 和 TLS 上实现安全机制。涵盖授权服务器（AS）、访问令牌（Token）、所有权证明（PoP）密钥，以及如何通过 CONNECT 报文（借用 User Name 和 Password 字段）或 authz-info 主题传输 Token。

#### 连接实际实现
这是本教程的核心要求。请将规范中的抽象理论与实际的 MQTT 工具（如 Mosquitto）和网络分析结合起来，让读者看到“活的” MQTT。

##### 服务端配置与参数
在讲解相关概念时，引用开源 Broker（如 Mosquitto）的配置参数。例如：
讲解安全与授权时，提及 mosquitto.conf 中的 acl_file、动态安全插件（Dynamic Security Plugin）以及基于证书的 TLS 配置。
讲解会话与消息保留时，提及 max_keepalive、persistent_client_expiration 等配置。

##### 命令行工具与抓包
使用 mosquitto_pub、mosquitto_sub 或 wireshark/tcpdump 的输出来作为实例。例如：
使用 mosquitto_sub --topic-alias 展示主题别名如何减少报文体积。
使用 wireshark 抓包展示 QoS 2 的 PUBLISH -> PUBREC -> PUBREL -> PUBCOMP 完整交互过程，观察 Packet Identifier 的变化。
使用 CLI 展示共享订阅的负载均衡效果（如 $share/group/topic）。

##### 代码/伪代码
在讲解 RFC 9431 的 JWT/CWT Token 结构和 PoP（Proof-of-Possession）签名验证时，提供 JSON 或 CDDL 的格式示例，帮助读者理解 Token 载荷和范围（Scope）的内部结构。

##### 使用 Mermaid 图表进行可视化
请为所有关键流程和复杂概念配上 Mermaid 图表，图文并茂地进行解释。必须包含但不限于以下图表：
MQTT 发布/订阅架构图 (graph TD) MQTT 控制报文通用结构图 (graph TD 或 classDiagram，展示 Fixed Header, Variable Header, Payload 的关系) CONNECT 与 CONNACK 连接建立时序图 (sequenceDiagram) QoS 1 和 QoS 2 消息传递机制的时序图 (sequenceDiagram) MQTT 5.0 共享订阅（Shared Subscriptions）消息分发示意图 (graph LR) RFC 9431 中 Client、Broker (RS) 和 Authorization Server (AS) 之间的 Token 申请与连接授权时序图 (sequenceDiagram)

### 深度与细节
请确保教程内容详尽，覆盖上述 MQTT 5.0 和 RFC 9431 的所有核心知识点。 对于重要字段（如 Packet Identifier, Remaining Length, Reason Codes, Property Length），请务必解释其确切含义、字节编码方式（如 Variable Byte Integer）和作用。 解释各种机制（如 QoS 降级、共享订阅的会话状态、Token 刷新与重新认证）是为了解决物联网环境下的什么具体问题（如高延迟、弱网络、设备资源受限）而设计的。 在介绍 MQTT 5.0 时，请明确指出其相对于 3.1.1 版本的改进之处（如报文级别的错误反馈、更小的传输开销）。

### 输出格式
请将所有内容生成为 Markdown (.md) 格式。
请将教程组织到 rfcs/mqtt/ 文件夹下。考虑到内容会非常多，建议将教程拆分为多个文件，例如：

rfcs/mqtt/01-introduction.md (核心概念与架构) 

rfcs/mqtt/02-connection-and-session.md (连接与会话管理) 

rfcs/mqtt/03-qos-and-delivery.md (服务质量与可靠传递) 

rfcs/mqtt/04-mqtt5-new-features.md (MQTT 5.0 高级特性) 

rfcs/mqtt/05-security-and-rfc9431.md (安全机制与 ACE 框架授权)