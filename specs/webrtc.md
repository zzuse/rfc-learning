```
WebRTC (Web Real-Time Communication) is a free, open-source project and a set of industry standards that enable web browsers and mobile applications to perform real-time, peer-to-peer (P2P) audio, video, and data streaming. It eliminates the need for users to install plugins or download native apps for communication, as it is built directly into modern web browsers (Safari, Chrome, Firefox, Edge). 
WebRTC was developed by Google and is maintained by the World Wide Web Consortium (W3C) for its JavaScript APIs and the Internet Engineering Task Force (IETF) for its transport protocols. 
Core Components of WebRTC
getUserMedia(): Accesses the user’s camera and microphone to capture media.
RTCPeerConnection(): Handles audio/video communication, codec handling, security, and bandwidth management between peers.
RTCDataChannel(): Allows direct, bidirectional, low-latency communication of arbitrary data (files, game state) between peers. 
RFC-Related Components of WebRTC
WebRTC is fundamentally a suite of protocols rather than just one technology. The IETF's rtcweb working group defined these protocols, with many appearing as "Requests for Comments" (RFCs). 
ICE (Interactive Connectivity Establishment) - RFC 8445: A framework used to traverse NATs and firewalls, allowing peers to find the best way to connect.
STUN (Session Traversal Utilities for NAT) - RFC 5389: Helps a peer discover its own public IP address and port.
TURN (Traversal Using Relays around NAT) - RFC 8656: A relay server used as a fallback if a direct peer-to-peer connection cannot be established.
DTLS (Datagram Transport Layer Security) - RFC 6347 & 8261: Secures the data channel, ensuring that data is encrypted and authenticated.
SRTP (Secure Real-time Transport Protocol) - RFC 3711: Encrypts the audio and video media streams.
SCTP (Stream Control Transmission Protocol) - RFC 4960 & 8831: Transports non-media data (like messages or files) over the Data Channel.
JSEP (JavaScript Session Establishment Protocol) - RFC 9429: JavaScript Session Establishment Protocol.
WebRTC Security Architecture - RFC 8827: Defines the security model, which requires encryption for all media and data.
WHIP (WebRTC-HTTP Ingestion Protocol) - RFC 9725: A recent standard for using WebRTC as a media ingestion protocol for broadcasting. 
Why WebRTC Matters
Peer-to-Peer: Data flows directly between browsers, which reduces latency and saves server costs.
No Plugins: It is natively integrated into web browsers.
Secure: Encryption is mandatory for all data, voice, and video.
Versatile: Used in video conferencing (Google Meet, Discord), file sharing (WebTorrent), and IoT.
```

# WebRTC 协议栈深度解析教程要求
请根据我提供的 RFC 列表（rfc8445, rfc5389, rfc8656, rfc8261, rfc3711, rfc4960, rfc8831, rfc9429, rfc8827, rfc9725），为我生成一份系统性、深度与实战并重的 WebRTC 学习教程。

## 1. 风格与语言
语言: 全程使用简体中文。

风格:

采用“专家导读”风格，既要有协议规范的严谨性，又要具备教学的生动性。

比喻: WebRTC 涉及多个协议的配合，请使用统一的比喻体系（例如：将建立 WebRTC 连接比作“在两个戒备森严的城堡间建立一条秘密地下铁路”）。

STUN 是“询问门牌号”。

TURN 是“中转仓库”。

ICE 是“探路者”。

DTLS 是“查验身份和交换钥匙”。

SCTP 是“多车道运输车”。

结构: 逻辑上需遵循 WebRTC 连接建立的生命周期：连接发现 -> 安全握手 -> 数据/媒体传输。

## 2. 章节规划与 RFC 映射
请将教程分为以下几个核心模块，并涵盖指定的 RFC：

### 第一部分：穿越迷雾 —— NAT 穿透与连接建立

核心 RFC: RFC 5389 (STUN), RFC 8656 (TURN), RFC 8445 (ICE)。

STUN (RFC 5389): 解释 Binding Request/Response，以及 Mapped Address 属性的作用。

TURN (RFC 8656): 解释为何需要 Relay，Allocation 的创建过程，以及 Send/Data indication。

ICE (RFC 8445):

这是本部分的重点。详细讲解 Candidate（候选者）的收集（Host, Srflx, Relay）。

解释 Connectivity Checks（连通性检查）和 STUN 的关系。

解释 Nominating（提名）和选路过程。

### 第二部分：谈判专家 —— 会话建立与 JSEP

核心 RFC: RFC 9429 (JSEP)。

JSEP (RFC 9429):

这是 WebRTC 的“大脑”。必须详细讲解 Offer/Answer 模型 如何驱动 ICE 和 DTLS 的启动。

状态机: 使用状态机图解释 stable, have-local-offer, have-remote-offer 等状态的流转。

SDP 分离: 解释 JSEP 如何将 SDP 的生成与应用（createOffer vs setLocalDescription）解耦，以及它如何处理 "Glare"（冲突）情况。

### 第三部分：构筑防线 —— 安全传输层

核心 RFC: RFC 8827 (Security Arch), RFC 8261 (DTLS-SRTP), RFC 3711 (SRTP)。

WebRTC Security Architecture (RFC 8827):

宏观视角: 讲解 WebRTC 的威胁模型。为什么浏览器需要强制加密？

身份与权限: 讲解同源策略 (SOP) 在 WebRTC 中的体现，以及设备权限（麦克风/摄像头）的管理。

IP 泄露保护: 解释 mDNS 及其在隐藏内网 IP 中的作用。

DTLS (RFC 8261):

解释 WebRTC 为什么要用 DTLS（基于 UDP 的 TLS）。

重点讲解 DTLS 握手过程中的 ClientHello, ServerHello, Certificate, Fingerprint 验证。

解释它是如何导出 SRTP 所需的密钥（Key Derivation）的。

SRTP (RFC 3711):

简述 RTP 的基础，重点讲解 SRTP 如何对媒体载荷进行加密，以及 ROC（Rollover Counter）的作用。

### 第四部分：双向快车道 —— 数据通道

核心 RFC: RFC 4960 (SCTP), RFC 8831 (WebRTC Data Channels)。

SCTP (RFC 4960):

注意: 重点讲解 SCTP 在 WebRTC 中是运行在 DTLS 之上的（SCTP over DTLS）。

讲解 SCTP 的核心特性：多流（Multi-streaming）、消息分片、有序/无序投递。

解释四次握手（INIT, INIT ACK, COOKIE ECHO, COOKIE ACK）。

Data Channel (RFC 8831):

解释 DCEP (Data Channel Establishment Protocol) 协议。

展示如何通过 SCTP 流打开数据通道。

### 第五部分：标准化推流 —— WHIP 协议

核心 RFC: RFC 9725 (WebRTC-HTTP Ingestion Protocol)。

WHIP:

痛点解决: 解释 WHIP 如何解决传统 WebRTC 信令不标准的问题，使 OBS 等软件能直接推流。

HTTP 交互: 详细解析 WHIP 的 HTTP POST 请求（携带 Offer SDP）和 201 Created 响应（携带 Answer SDP）。

Bearer Token: 讲解简单的认证机制。

REST API 设计: 解释如何通过 HTTP DELETE 终止会话。

## 3. 结合实战与调试 (The "Live" WebRTC)
不同于 TCP 的内核参数，WebRTC 的实战体现在 SDP 协商 和 抓包分析。请在讲解理论时结合以下内容：

SDP (Session Description Protocol) 映射:

在讲解 ICE 时，展示 SDP 中的 a=candidate 行，解释每个字段的含义（Priority, IP, Port, Type）。

在讲解 DTLS 时，展示 a=fingerprint 和 a=setup (active/passive) 属性。

在讲解 SCTP/DataChannel 时，展示 m=application 和 a=sctp-port。

调试工具与抓包:

Wireshark: 请给出具体的 Display Filter。例如：

过滤 STUN/TURN: stun

过滤 DTLS: dtls

过滤 SCTP: sctp

Chrome Internals: 提及 chrome://webrtc-internals，并说明如何用它查看 iceConnectionState 或 bytesReceived 等关键指标。

## 4. Mermaid 图表可视化要求
必须包含以下图表来辅助理解：

WebRTC 协议栈总览图 (graph TD): 清晰展示 IP -> UDP -> DTLS -> (SRTP / SCTP) 的层级包裹关系。

ICE 连接建立流程 (sequenceDiagram): 展示由 STUN Binding Request 触发的 Candidate 收集和连通性检查过程。

DTLS 握手与密钥导出 (sequenceDiagram): 展示在 ICE 连通后，如何进行 DTLS 握手并生成 SRTP Key。

SCTP over UDP 封装示意图 (graph LR): 展示数据包结构。

## 5. 输出格式
请将教程组织为 Markdown (.md) 格式，并建议按照以下文件结构输出（请按顺序生成）：

webrtc/01-architecture-overview.md (协议栈总览与 RFC 8827 安全架构基础)

webrtc/02-ice-stun-turn.md (连接与穿透 - RFC 8445/5389/8656)

webrtc/03-jsep-signaling.md (会话谈判与状态机 - RFC 9429)

webrtc/04-security-dtls-srtp.md (加密传输细节 - RFC 8261/3711)

webrtc/05-sctp-data-channels.md (数据通道 - RFC 4960/8831)

webrtc/06-whip-ingestion.md (现代直播推流 - RFC 9725)

