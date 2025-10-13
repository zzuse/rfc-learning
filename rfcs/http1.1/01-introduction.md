# 第一章:HTTP 核心概念

> 本章基于 RFC 9110 (HTTP Semantics) 和 RFC 9112 (HTTP/1.1) 规范编写

## 目录

- [1.1 HTTP 的前世今生](#11-http-的前世今生)
- [1.2 客户端-服务器模型](#12-客户端-服务器模型)
- [1.3 资源与 URI](#13-资源与-uri)
- [1.4 请求与响应的基本结构](#14-请求与响应的基本结构)
- [1.5 HTTP 的无状态特性](#15-http-的无状态特性)
- [1.6 实战演练](#16-实战演练)

---

## 1.1 HTTP 的前世今生

### 什么是 HTTP?

HTTP (Hypertext Transfer Protocol,超文本传输协议)是互联网上应用最广泛的应用层协议之一。你可以把它想象成 **网络世界的"快递规则"**——定义了浏览器和服务器之间如何打包、传送和签收信息。

### HTTP 的演进历程

```mermaid
graph LR
    A[HTTP/0.9<br/>1991年] --> B[HTTP/1.0<br/>1996年]
    B --> C[HTTP/1.1<br/>1997年]
    C --> D[HTTP/2<br/>2015年]
    D --> E[HTTP/3<br/>2022年]

    style C fill:#e1f5ff,stroke:#01579b,stroke-width:3px
```

| 版本         | 年份 | 主要特性                                           |
|--------------|------|------------------------------------------------|
| HTTP/0.9     | 1991 | 只支持 GET 方法,只能传输 HTML                      |
| HTTP/1.0     | 1996 | 引入 Headers、状态码、多种请求方法                   |
| **HTTP/1.1** | 1997 | **持久连接、管道化、分块传输、缓存机制** (本教程重点) |
| HTTP/2       | 2015 | 二进制分帧、多路复用、服务器推送                     |
| HTTP/3       | 2022 | 基于 QUIC 协议,改用 UDP 传输                       |

**本教程聚焦于 HTTP/1.1**,因为它:

- ✅ **应用最广泛** - 至今仍被绝大多数网站支持
- ✅ **向下兼容** - 理解 HTTP/1.1 是学习后续版本的基础
- ✅ **清晰易懂** - 文本格式,便于调试和学习

### HTTP/1.1 相比 1.0 的重大改进

HTTP/1.1 (RFC 9112 在 2022 年更新了标准) 针对 HTTP/1.0 的性能问题做了重要优化:

#### ❌ HTTP/1.0 的问题

```mermaid
sequenceDiagram
    participant C as 客户端
    participant S as 服务器

    Note over C,S: 请求第 1 个资源
    C->>S: 建立 TCP 连接 (三次握手)
    C->>S: GET /index.html
    S->>C: 返回 index.html
    C->>S: 关闭连接 (四次挥手)

    Note over C,S: 请求第 2 个资源
    C->>S: 重新建立 TCP 连接 (三次握手)
    C->>S: GET /style.css
    S->>C: 返回 style.css
    C->>S: 关闭连接 (四次挥手)

    Note over C,S: 每个资源都需要新建连接!<br/>延迟巨大,效率低下
```

**痛点**: 每次请求都要建立新的 TCP 连接,一个网页通常包含几十个资源(HTML、CSS、JS、图片),导致大量时间浪费在 TCP 握手和挥手上。

#### ✅ HTTP/1.1 的优化

```mermaid
sequenceDiagram
    participant C as 客户端
    participant S as 服务器

    C->>S: 建立 TCP 连接 (仅一次三次握手)
    C->>S: GET /index.html
    S->>C: 返回 index.html
    C->>S: GET /style.css
    S->>C: 返回 style.css
    C->>S: GET /script.js
    S->>C: 返回 script.js

    Note over C,S: 使用同一个连接!<br/>大幅降低延迟
```

**核心特性**:

1. **持久连接 (Persistent Connections)** - 默认开启,一个 TCP 连接可以发送多个请求 (详见第四章)
2. **管道化 (Pipelining)** - 允许同时发送多个请求而无需等待响应 (虽然后来因队头阻塞问题被弃用)
3. **分块传输编码 (Chunked Transfer Encoding)** - 支持流式传输,无需预先知道内容长度
4. **强大的缓存机制** - 引入 `Cache-Control`、`ETag` 等头部 (详见第六章)
5. **Host 头部强制性** - 支持虚拟主机,一台服务器托管多个域名

---

## 1.2 客户端-服务器模型

### 核心概念

HTTP 采用 **请求-响应 (Request-Response)** 模式,就像 **打电话点外卖**:

```mermaid
sequenceDiagram
    participant 客户端 (你)
    participant 服务器 (餐厅)

    客户端 (你)->>服务器 (餐厅): 📞 请求:"我要一份宫保鸡丁"
    Note over 服务器 (餐厅): 处理请求<br/>(做菜)
    服务器 (餐厅)->>客户端 (你): 📦 响应:"您的宫保鸡丁已送达"
```

### 角色分工

#### 1. **客户端 (Client)**

**定义**: 发起 HTTP 请求的一方

**常见客户端**:

- 🌐 **浏览器** (Chrome、Firefox、Safari)
- 📱 **移动应用**
- 🤖 **爬虫程序**
- 🔧 **命令行工具** (curl、wget)

**示例 - 使用 curl 作为客户端**:

```bash
# -v 参数显示详细的请求和响应过程
curl -v https://www.example.com
```

输出 (简化版):

```
> GET / HTTP/1.1                         ← 这是客户端发送的请求
> Host: www.example.com
> User-Agent: curl/7.84.0
> Accept: */*
```

#### 2. **服务器 (Server)**

**定义**: 接收请求并返回响应的一方

**常见服务器**:

- 🚀 **Nginx** - 高性能 Web 服务器
- ⚡ **Apache** - 老牌 Web 服务器
- 🟢 **Node.js** (Express、Koa)
- 🐍 **Python** (Django、Flask)
- ☕ **Java** (Tomcat、Spring Boot)

**示例 - 服务器的响应**:

```bash
curl -v https://www.example.com
```

输出:

```
< HTTP/1.1 200 OK                        ← 这是服务器返回的响应
< Content-Type: text/html; charset=UTF-8
< Content-Length: 1256
< Date: Mon, 15 Jan 2024 10:30:00 GMT
<
<!DOCTYPE html>
<html>...                                ← 响应正文 (HTML 内容)
```

#### 3. **中间人 (Intermediaries)**

在客户端和源服务器之间,还可能存在 **代理 (Proxy)** 和 **网关 (Gateway)**:

```mermaid
graph LR
    A[客户端] --> B[正向代理<br/>VPN/企业代理]
    B --> C[互联网]
    C --> D[反向代理<br/>Nginx/CDN]
    D --> E[源服务器]

    style A fill:#e8f5e9
    style E fill:#fff3e0
    style B fill:#e1f5fe
    style D fill:#fce4ec
```

**正向代理** (Forward Proxy):

- 代表客户端去请求服务器
- 例子:企业网络代理、VPN

**反向代理** (Reverse Proxy):

- 代表服务器接收客户端请求
- 例子:Nginx 负载均衡、CDN (内容分发网络)

---

## 1.3 资源与 URI

### 什么是资源?

**资源 (Resource)** 是 HTTP 的操作对象,可以是:

- 📄 **静态文件** - HTML、CSS、JS、图片、视频
- 🔢 **动态数据** - API 返回的 JSON、XML
- 📧 **服务** - 发送邮件、生成报表

**类比**: 资源就像图书馆里的 **书籍**,每本书都有一个唯一的编号 (URI)

### URI - 资源的唯一标识符

**URI (Uniform Resource Identifier,统一资源标识符)** 用于定位互联网上的资源。

#### URI 的结构

```
  ┌─────────────────────────── URI ─────────────────────────────┐
  │                                                               │
https://www.example.com:443/path/to/resource?key=value#section
 │      │              │    │                 │         │
 └─┬──┘ └──────┬──────┘└─┬─┘└────────┬───────┘└────┬───┘└───┬──┘
   │           │         │           │             │        │
  协议       主机名      端口        路径          查询参数  片段
(Scheme)   (Host)    (Port)     (Path)        (Query)  (Fragment)
```

**组成部分详解**:

| 部分                 | 说明                          | 示例                                    | 是否必需 |
|--------------------|-----------------------------|-----------------------------------------|--------|
| **协议 (Scheme)**    | 指定使用的协议                | `http`、`https`                          | ✅ 必需   |
| **主机名 (Host)**    | 服务器的域名或 IP 地址        | `www.example.com`                       | ✅ 必需   |
| **端口 (Port)**      | 服务器监听的端口号            | `80` (HTTP 默认)<br/>`443` (HTTPS 默认) | ❌ 可选   |
| **路径 (Path)**      | 资源在服务器上的位置          | `/path/to/resource`                     | ❌ 可选   |
| **查询参数 (Query)** | 传递给服务器的参数            | `?key1=value1&key2=value2`              | ❌ 可选   |
| **片段 (Fragment)**  | 资源内的锚点 (仅在客户端使用) | `#section`                              | ❌ 可选   |

#### URI 示例分析

```
https://www.github.com/search?q=http&type=repositories#top
```

解析:

- **协议**: `https` - 使用加密的 HTTPS 协议
- **主机名**: `www.github.com` - GitHub 的域名
- **路径**: `/search` - 搜索页面
- **查询参数**: `q=http&type=repositories` - 搜索关键词 "http",类型为仓库
- **片段**: `#top` - 定位到页面的 `id="top"` 元素

**curl 示例**:

```bash
curl "https://api.github.com/search/repositories?q=http&sort=stars"
```

### URL vs URI

- **URI** (Uniform Resource **Identifier**) - 统一资源**标识符** (更宽泛的概念)
- **URL** (Uniform Resource **Locator**) - 统一资源**定位符** (URI 的一种,强调位置)

**关系**: URL 是 URI 的子集,所有 URL 都是 URI,但不是所有 URI 都是 URL。

```
       ┌────────── URI ──────────┐
       │                         │
       │   ┌────── URL ──────┐   │
       │   │ http://a.com/b │   │  ← URL (包含位置信息)
       │   └────────────────┘   │
       │                         │
       │   urn:isbn:0451450523   │  ← URN (仅标识,不包含位置)
       └─────────────────────────┘
```

---

## 1.4 请求与响应的基本结构

### HTTP 消息的组成

HTTP 的请求和响应都遵循统一的消息格式:

```
┌─────────────────────────────────┐
│  起始行 (Start Line)              │  ← 请求行 或 状态行
├─────────────────────────────────┤
│  头部 (Headers)                  │  ← 零个或多个头部字段
│  Header1: value1                │
│  Header2: value2                │
├─────────────────────────────────┤
│  空行 (CRLF)                     │  ← 标志头部结束
├─────────────────────────────────┤
│  消息正文 (Message Body)         │  ← 可选的数据内容
│  [实际数据]                      │
└─────────────────────────────────┘
```

### HTTP 请求的结构

#### 完整示例

```mermaid
graph TD
    A[HTTP 请求报文] --> B[请求行<br/>Request Line]
    A --> C[请求头部<br/>Request Headers]
    A --> D[空行<br/>CRLF]
    A --> E[请求正文<br/>Request Body<br/>可选]

    B --> B1["GET /api/users HTTP/1.1"]
    C --> C1["Host: api.example.com"]
    C --> C2["User-Agent: curl/7.84.0"]
    C --> C3["Accept: application/json"]
    E --> E1["username=john&age=30"]

    style B fill:#bbdefb
    style C fill:#c8e6c9
    style D fill:#fff9c4
    style E fill:#ffccbc
```

#### 请求行 (Request Line)

格式: `<方法> <请求目标> <HTTP版本>`

```
GET /api/users?page=1 HTTP/1.1
│   │                  │
└┬┘ └────────┬────────┘└────┬────┘
 │           │               │
方法      请求目标        HTTP版本
```

**组成部分**:

1. **方法 (Method)** - 告诉服务器要执行什么操作 (详见第三章)
   - `GET` - 获取资源
   - `POST` - 提交数据
   - `PUT` - 更新资源
   - `DELETE` - 删除资源
   - 等等...

2. **请求目标 (Request Target)** - 要操作的资源
   - 通常是 URI 的路径部分和查询参数
   - 例如: `/api/users?page=1`

3. **HTTP 版本** - 客户端支持的 HTTP 协议版本
   - `HTTP/1.1` - 本教程重点
   - `HTTP/1.0` - 旧版本
   - `HTTP/2` - 新版本 (二进制格式)

#### 请求头部 (Request Headers)

头部字段格式: `<字段名>: <字段值>`

```
Host: api.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: application/json, text/html
Content-Type: application/json
Content-Length: 28
```

**常见请求头部**:

| 头部字段         | 作用                       | 示例                             |
|------------------|--------------------------|----------------------------------|
| `Host`           | **必需!** 指定服务器域名   | `Host: www.example.com`          |
| `User-Agent`     | 标识客户端类型             | `User-Agent: curl/7.84.0`        |
| `Accept`         | 告诉服务器可接受的内容类型 | `Accept: application/json`       |
| `Content-Type`   | 请求正文的数据类型         | `Content-Type: application/json` |
| `Content-Length` | 请求正文的字节数           | `Content-Length: 28`             |
| `Authorization`  | 身份认证信息               | `Authorization: Bearer token123` |

**为什么 Host 头部是必需的?**

在 HTTP/1.1 中,一台物理服务器可以托管多个网站 (虚拟主机),服务器需要 `Host` 头部来区分请求的是哪个网站:

```
                    ┌─ www.site1.com
同一IP地址 80.80.80.80 ┼─ www.site2.com
                    └─ www.site3.com
```

**curl 示例 - 查看请求头部**:

```bash
curl -v -H "Accept: application/json" https://api.github.com/users/octocat
```

输出中的请求头部:

```
> GET /users/octocat HTTP/1.1
> Host: api.github.com
> User-Agent: curl/7.84.0
> Accept: application/json
```

#### 请求正文 (Request Body)

**仅部分方法需要** (如 `POST`、`PUT`、`PATCH`),用于传输数据。

**示例 - 提交 JSON 数据**:

```bash
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Alice","age":25}'
```

完整请求:

```http
POST /users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Content-Length: 28

{"name":"Alice","age":25}
```

### HTTP 响应的结构

#### 完整示例

```mermaid
graph TD
    A[HTTP 响应报文] --> B[状态行<br/>Status Line]
    A --> C[响应头部<br/>Response Headers]
    A --> D[空行<br/>CRLF]
    A --> E[响应正文<br/>Response Body<br/>可选]

    B --> B1["HTTP/1.1 200 OK"]
    C --> C1["Content-Type: application/json"]
    C --> C2["Content-Length: 45"]
    C --> C3["Date: Mon, 15 Jan 2024 10:30:00 GMT"]
    E --> E1["json data"]

    style B fill:#bbdefb
    style C fill:#c8e6c9
    style D fill:#fff9c4
    style E fill:#ffccbc
```

#### 状态行 (Status Line)

格式: `<HTTP版本> <状态码> <原因短语>`

```
HTTP/1.1 200 OK
│        │   │
└───┬───┘└─┬─┘└┬┘
    │      │   │
HTTP版本 状态码 原因短语
```

**组成部分**:

1. **HTTP 版本** - 服务器使用的 HTTP 协议版本
2. **状态码 (Status Code)** - 三位数字,表示请求的处理结果 (详见第三章)
   - `1xx` - 信息性响应
   - `2xx` - 成功
   - `3xx` - 重定向
   - `4xx` - 客户端错误
   - `5xx` - 服务器错误
3. **原因短语 (Reason Phrase)** - 状态码的文本描述 (仅供人类阅读,客户端应忽略)

**常见状态码**:

| 状态码 | 原因短语              | 含义                  |
|--------|-----------------------|---------------------|
| `200`  | OK                    | 请求成功              |
| `201`  | Created               | 资源已创建            |
| `301`  | Moved Permanently     | 永久重定向            |
| `302`  | Found                 | 临时重定向            |
| `304`  | Not Modified          | 资源未修改 (缓存有效) |
| `400`  | Bad Request           | 请求格式错误          |
| `401`  | Unauthorized          | 需要身份验证          |
| `403`  | Forbidden             | 服务器拒绝请求        |
| `404`  | Not Found             | 资源不存在            |
| `500`  | Internal Server Error | 服务器内部错误        |

#### 响应头部 (Response Headers)

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 1256
Date: Mon, 15 Jan 2024 10:30:00 GMT
Server: nginx/1.22.0
Cache-Control: max-age=3600
```

**常见响应头部**:

| 头部字段         | 作用               | 示例                                   |
|------------------|------------------|----------------------------------------|
| `Content-Type`   | 响应正文的数据类型 | `Content-Type: application/json`       |
| `Content-Length` | 响应正文的字节数   | `Content-Length: 1256`                 |
| `Date`           | 响应生成的时间     | `Date: Mon, 15 Jan 2024 10:30:00 GMT`  |
| `Server`         | 服务器软件信息     | `Server: nginx/1.22.0`                 |
| `Cache-Control`  | 缓存策略           | `Cache-Control: max-age=3600`          |
| `Set-Cookie`     | 设置 Cookie        | `Set-Cookie: session=abc123; HttpOnly` |

**curl 示例 - 查看响应头部**:

```bash
curl -I https://www.example.com
# 或者
curl --head https://www.example.com
```

输出 (仅响应头部,不包含正文):

```http
HTTP/1.1 200 OK
Content-Encoding: gzip
Accept-Ranges: bytes
Age: 345678
Cache-Control: max-age=604800
Content-Type: text/html; charset=UTF-8
Date: Mon, 15 Jan 2024 10:30:00 GMT
Server: ECS (nyb/1D1B)
Content-Length: 648
```

#### 响应正文 (Response Body)

包含实际的资源内容,格式由 `Content-Type` 头部指定。

**示例 - HTML 响应**:

```bash
curl https://www.example.com
```

响应:

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 1256

<!DOCTYPE html>
<html>
<head>
    <title>Example Domain</title>
</head>
<body>
    <h1>Example Domain</h1>
    <p>This domain is for use in illustrative examples...</p>
</body>
</html>
```

**示例 - JSON 响应**:

```bash
curl https://api.github.com/users/octocat
```

响应:

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 1520

{
  "login": "octocat",
  "id": 583231,
  "name": "The Octocat",
  "company": "@github",
  "bio": null,
  "public_repos": 8
}
```

---

## 1.5 HTTP 的无状态特性

### 什么是无状态?

**无状态 (Stateless)** 是 HTTP 的核心特性之一,意思是:

> **每个 HTTP 请求都是独立的,服务器不会自动记住之前的请求。**

**类比**: HTTP 就像 **健忘的服务员**,每次你点单,他都不记得你上次点了什么。

```mermaid
sequenceDiagram
    participant C as 客户端
    participant S as 服务器

    Note over C,S: 第 1 次请求
    C->>S: 请求:"我要一杯咖啡"
    S->>C: 响应:"好的,这是你的咖啡"

    Note over C,S: 第 2 次请求
    C->>S: 请求:"我要加糖"
    S->>C: 响应:"??? 你要给什么加糖?"
    Note over S: 服务器不记得<br/>第 1 次请求
```

### 为什么要设计成无状态?

**优点**:

1. **简单性** - 服务器不需要管理复杂的会话状态
2. **可扩展性** - 请求可以分发到任意服务器处理 (负载均衡)
3. **可靠性** - 服务器重启不会影响后续请求

**缺点**:

- 无法实现"购物车"、"用户登录"等需要记忆状态的功能

### 如何在无状态的 HTTP 中保持状态?

虽然 HTTP 本身是无状态的,但可以通过以下机制实现状态管理:

#### 1. Cookie

服务器在响应中通过 `Set-Cookie` 头部发送一个标识符,客户端在后续请求中通过 `Cookie` 头部带回这个标识符。

```mermaid
sequenceDiagram
    participant C as 客户端
    participant S as 服务器

    Note over C,S: 首次请求 (登录)
    C->>S: POST /login<br/>username=alice&password=123
    S->>C: HTTP/1.1 200 OK<br/>Set-Cookie: session=abc123

    Note over C,S: 后续请求
    C->>S: GET /profile<br/>Cookie: session=abc123
    Note over S: 根据 session=abc123<br/>识别用户
    S->>C: HTTP/1.1 200 OK<br/>{"name":"Alice","age":25}
```

**实战示例**:

```bash
# 1. 登录,服务器返回 Cookie
curl -i -X POST https://example.com/login \
  -d "username=alice&password=123"
```

响应:

```http
HTTP/1.1 200 OK
Set-Cookie: session=abc123; HttpOnly; Secure
Content-Type: application/json

{"message":"登录成功"}
```

```bash
# 2. 后续请求携带 Cookie
curl -H "Cookie: session=abc123" https://example.com/profile
```

请求:

```http
GET /profile HTTP/1.1
Host: example.com
Cookie: session=abc123
```

#### 2. Session

Session 是在服务器端存储用户状态,Cookie 中只保存 Session ID。

```
┌─────────┐                   ┌─────────┐
│  客户端  │   Cookie:         │  服务器  │
│         │   session=abc123  │         │
│         │ ─────────────────>│         │
│         │                   │  查找   │
│         │                   │  Session│
│         │                   │  存储   │
│         │                   │  ↓      │
│         │                   │ {user:  │
│         │<───────────────── │ "alice"}│
└─────────┘   返回用户数据     └─────────┘
```

#### 3. Token (JWT)

现代 Web 应用常用 **JWT (JSON Web Token)**,将状态信息编码在 Token 中,客户端在请求头中携带:

```bash
curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  https://api.example.com/profile
```

**JWT 结构** (简化):

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.   ← Header (头部)
eyJ1c2VyIjoiYWxpY2UiLCJleHAiOjE3MDU...   ← Payload (载荷,包含用户信息)
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV...   ← Signature (签名)
```

---

## 1.6 实战演练

### 练习 1: 使用 curl 发送 GET 请求

**目标**: 获取 GitHub 用户信息

```bash
curl -v https://api.github.com/users/octocat
```

**观察点**:

1. 请求行是什么?
2. 有哪些请求头部?
3. 状态码是多少?
4. 响应的 `Content-Type` 是什么?

<details>
<summary>点击查看答案</summary>

```
> GET /users/octocat HTTP/1.1
> Host: api.github.com
> User-Agent: curl/7.84.0
> Accept: */*

< HTTP/1.1 200 OK
< Content-Type: application/json; charset=utf-8
< ...

{
  "login": "octocat",
  "id": 583231,
  ...
}
```

**分析**:

- **请求行**: `GET /users/octocat HTTP/1.1`
- **关键请求头**: `Host`, `User-Agent`, `Accept`
- **状态码**: `200 OK` (请求成功)
- **响应类型**: `application/json` (JSON 格式)

</details>

### 练习 2: 使用 curl 发送 POST 请求

**目标**: 模拟提交用户注册数据

```bash
curl -X POST https://httpbin.org/post \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","email":"alice@example.com"}'
```

**观察点**:

1. 请求方法是什么?
2. `Content-Type` 头部的作用是什么?
3. 服务器收到的数据是什么?

<details>
<summary>点击查看答案</summary>

```
> POST /post HTTP/1.1
> Host: httpbin.org
> Content-Type: application/json
> Content-Length: 50
>
> {"username":"alice","email":"alice@example.com"}

< HTTP/1.1 200 OK
< Content-Type: application/json
<
{
  "json": {
    "username": "alice",
    "email": "alice@example.com"
  },
  "headers": {
    "Content-Type": "application/json"
  }
}
```

**分析**:

- **请求方法**: `POST` (提交数据)
- **Content-Type**: 告诉服务器请求正文是 JSON 格式
- **服务器接收**: 成功接收并返回提交的数据

</details>

### 练习 3: 观察浏览器的 HTTP 请求

1. 打开 Chrome 浏览器
2. 按 `F12` 打开开发者工具
3. 切换到 "Network (网络)" 标签
4. 访问 `https://www.example.com`
5. 点击第一个请求,查看:
   - **Headers** (头部)
   - **Response** (响应)
   - **Timing** (时间线)

**观察点**:

- 浏览器发送了哪些请求头部? (特别注意 `User-Agent`、`Accept`)
- 服务器返回的状态码是什么?
- 页面加载了多少个资源? (HTML、CSS、JS、图片)
- 哪些请求使用了缓存? (看 `from disk cache` 或 `from memory cache`)

---

## 本章小结

### 核心要点

1. **HTTP 是应用层协议**,采用请求-响应模式,HTTP/1.1 通过持久连接、分块传输、缓存机制等特性大幅提升了性能。

2. **客户端-服务器模型**:
   - 客户端发起请求 (浏览器、curl、移动应用等)
   - 服务器返回响应 (Nginx、Apache、Node.js 等)
   - 中间可能存在代理和网关

3. **URI 定位资源**:
   - 格式: `协议://主机名:端口/路径?查询参数#片段`
   - `Host` 头部在 HTTP/1.1 中是必需的,支持虚拟主机

4. **HTTP 消息结构**:
   - 请求: 请求行 + 请求头部 + 空行 + 请求正文(可选)
   - 响应: 状态行 + 响应头部 + 空行 + 响应正文(可选)

5. **HTTP 是无状态的**:
   - 每个请求独立,服务器不自动记忆
   - 通过 Cookie、Session、Token 实现状态管理

### 下一章预告

在 [第二章: HTTP 报文格式](./02-message-format.md) 中,我们将深入剖析:

- 请求行和状态行的详细语法
- HTTP 头部的完整规范和常用头部字段
- 消息正文的传输机制 (Content-Length vs Transfer-Encoding)
- 使用 curl 和浏览器开发者工具进行实战调试

---

## 参考资料

- [RFC 9110 - HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)
- [RFC 9112 - HTTP/1.1](https://www.rfc-editor.org/rfc/rfc9112.html)
- [MDN - HTTP Overview](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)
