# HTTP/1.1 完整教程

> 基于 RFC 9110 (HTTP Semantics)、RFC 9111 (HTTP Caching)、RFC 9112 (HTTP/1.1) 规范编写的中文教程

## 教程概述

这是一套面向初学者的 HTTP/1.1 完整教程,从零开始深入讲解 HTTP 协议的核心概念、报文格式、方法与状态码、连接管理、内容协商和缓存机制。

**特色**:
- ✅ **中文详解** - 适合国内开发者学习
- ✅ **RFC 规范** - 基于最新的 RFC 9110/9111/9112 标准
- ✅ **实战导向** - 包含大量 curl 命令行示例和 Nginx 配置
- ✅ **可视化图表** - 使用 Mermaid 绘制流程图和序列图
- ✅ **循序渐进** - 从基础概念到高级特性,逐步深入

## 教程目录

### [第一章: HTTP 核心概念](./01-introduction.md)

**学习目标**: 理解 HTTP 的基本原理和核心概念

**主要内容**:
- HTTP 的前世今生 - 从 HTTP/0.9 到 HTTP/3 的演进
- 客户端-服务器模型 - 请求-响应机制
- 资源与 URI - URI 的结构和组成部分
- 请求与响应的基本结构 - HTTP 消息格式概览
- HTTP 的无状态特性 - Cookie、Session、Token 状态管理

**关键知识点**:
- HTTP/1.1 相比 HTTP/1.0 的重大改进 (持久连接、分块传输、缓存机制)
- URI 的七个组成部分 (协议、主机名、端口、路径、查询参数、片段)
- 虚拟主机与 `Host` 头部的关系
- 无状态设计的优缺点和状态管理方案

**实战练习**:
- 使用 curl 发送 GET/POST 请求
- 观察浏览器的 HTTP 请求 (Chrome DevTools)
- 理解 Cookie 的工作原理

---

### [第二章: HTTP 报文格式](./02-message-format.md)

**学习目标**: 掌握 HTTP 消息的详细结构和语法规则

**主要内容**:
- HTTP 消息的通用格式 - 起始行、头部、空行、正文
- 请求行详解 - 方法、请求目标的四种形式、HTTP 版本
- 状态行详解 - HTTP 版本、状态码、原因短语
- HTTP 头部字段 - 通用头部、请求头部、响应头部、实体头部
- 消息正文与传输机制 - Content-Length vs Transfer-Encoding

**关键知识点**:
- 请求目标的四种形式: origin-form、absolute-form、authority-form、asterisk-form
- 常用头部字段: Host、Content-Type、Content-Length、User-Agent、Cache-Control
- 分块传输编码的语法和使用场景
- 头部字段的解析规则和安全考虑

**实战练习**:
- 使用 curl 查看完整的 HTTP 报文
- 发送 JSON 数据并观察 Content-Type 和 Content-Length
- 观察浏览器发送的头部字段

---

### [第三章: HTTP 方法与状态码](./03-methods-and-status.md)

**学习目标**: 理解 HTTP 方法的语义和状态码的含义

**主要内容**:
- HTTP 方法概述 - 9 个标准方法
- 方法的属性 - 安全性、幂等性、可缓存性
- 标准 HTTP 方法详解 - GET、POST、PUT、DELETE、PATCH、HEAD、OPTIONS、CONNECT、TRACE
- HTTP 状态码详解 - 1xx、2xx、3xx、4xx、5xx 五大类
- 常见状态码使用场景和调试技巧

**关键知识点**:
- GET vs POST - 数据位置、长度限制、缓存、幂等性
- PUT vs PATCH - 完整替换 vs 部分更新
- DELETE 的幂等性 - 多次删除结果相同
- 状态码分类: 1xx 信息、2xx 成功、3xx 重定向、4xx 客户端错误、5xx 服务器错误
- 常见状态码: 200、201、204、301、302、304、400、401、403、404、500、502、503

**实战练习**:
- 测试不同的 HTTP 方法 (使用 httpbin.org)
- 观察重定向 (301 vs 302)
- 触发 404 和 500 错误并调试

---

### [第四章: 连接管理与性能优化](./04-connection-management.md)

**学习目标**: 理解 HTTP 连接管理和性能优化策略

**主要内容**:
- HTTP 连接模型演进 - HTTP/1.0 短连接 vs HTTP/1.1 持久连接
- 持久连接 (Keep-Alive) - Connection 头部、超时机制
- 管道化 (Pipelining) - 工作原理、队头阻塞问题
- 连接管理最佳实践 - 客户端连接池、服务器上游连接池
- 性能优化策略 - 减少请求、域名分片、Gzip 压缩、HTTP/2、CDN

**关键知识点**:
- 持久连接减少 66% 的 RTT
- Connection: keep-alive vs Connection: close
- 管道化的队头阻塞问题 (Head-of-Line Blocking)
- Nginx 连接配置: keepalive_timeout、keepalive_requests
- 浏览器并发连接数限制 (6 个/域名)

**实战练习**:
- 观察持久连接 (使用 telnet 或 curl)
- 测试连接超时
- 观察浏览器 Network Waterfall 中的并发连接
- 对比 HTTP/1.1 vs HTTP/2 性能

---

### [第五章: 内容协商](./05-content-negotiation.md)

**学习目标**: 掌握 HTTP 内容协商机制

**主要内容**:
- 内容协商概述 - 主动协商、被动协商、透明协商
- 主动协商 (Proactive Negotiation) - Accept、Accept-Language、Accept-Encoding
- 被动协商 (Reactive Negotiation) - 300 Multiple Choices
- 质量值 (Quality Values) - q-value 的使用和优先级
- Vary 头部与缓存 - 缓存变化维度

**关键知识点**:
- Accept 头部 - 媒体类型协商 (application/json、text/html 等)
- Accept-Language 头部 - 语言协商 (zh-CN、en-US 等)
- Accept-Encoding 头部 - 编码协商 (gzip、br、deflate)
- 质量值范围 0.0-1.0,默认 1.0,q=0 表示不接受
- Vary 头部确保缓存返回正确的变体

**实战练习**:
- 测试 Accept 头部 (请求 JSON vs HTML)
- 测试 Accept-Language (中文版 vs 英文版)
- 测试 Accept-Encoding (对比压缩率)
- 观察 Vary 头部的作用

---

### [第六章: HTTP 缓存机制](./06-caching.md)

**学习目标**: 深入理解 HTTP 缓存原理和配置

**主要内容**:
- HTTP 缓存概述 - 私有缓存 vs 共享缓存
- 强缓存 (Strong Caching) - Cache-Control、Expires
- 协商缓存 (Conditional Requests) - ETag、Last-Modified
- 缓存新鲜度计算 - freshness_lifetime、current_age
- 缓存配置最佳实践 - 不同资源类型的缓存策略

**关键知识点**:
- Cache-Control 指令: max-age、public、private、no-cache、no-store、must-revalidate、s-maxage
- 强缓存 (200 from cache) vs 协商缓存 (304 Not Modified)
- ETag vs Last-Modified - 精度、唯一性、性能、优先级
- 新鲜度算法: response_is_fresh = (freshness_lifetime > current_age)
- 文件指纹 (Cache Busting) - 使用内容哈希实现缓存失效

**实战练习**:
- 观察强缓存 (200 from disk cache)
- 测试协商缓存 (304 Not Modified)
- 浏览器缓存观察 (Chrome DevTools)
- 对比强制刷新 (Ctrl+F5) vs 普通刷新 (F5)

---

## 快速导航

### 按主题学习

**初学者入门** (建议按顺序阅读):
1. [第一章: HTTP 核心概念](./01-introduction.md) - 了解 HTTP 基础
2. [第二章: HTTP 报文格式](./02-message-format.md) - 学习消息结构
3. [第三章: HTTP 方法与状态码](./03-methods-and-status.md) - 掌握核心语义

**进阶深入**:
4. [第四章: 连接管理与性能优化](./04-connection-management.md) - 优化性能
5. [第五章: 内容协商](./05-content-negotiation.md) - 多语言和多格式支持
6. [第六章: HTTP 缓存机制](./06-caching.md) - 缓存策略和配置

### 按场景学习

**API 开发**:
- [第三章: HTTP 方法](./03-methods-and-status.md#33-标准-http-方法详解) - GET、POST、PUT、DELETE、PATCH
- [第三章: 状态码](./03-methods-and-status.md#34-http-状态码详解) - 200、201、400、404、500
- [第五章: 内容协商](./05-content-negotiation.md#52-主动协商-proactive-negotiation) - JSON vs XML
- [第六章: API 缓存](./06-caching.md#65-缓存配置最佳实践) - private、max-age

**Web 性能优化**:
- [第四章: 持久连接](./04-connection-management.md#42-持久连接-keep-alive) - Keep-Alive 配置
- [第四章: 性能优化](./04-connection-management.md#45-性能优化策略) - 减少请求、Gzip、CDN
- [第六章: 缓存策略](./06-caching.md#65-缓存配置最佳实践) - 静态资源缓存
- [第六章: 文件指纹](./06-caching.md#文件指纹-cache-busting) - Cache Busting

**Nginx 运维**:
- [第四章: Nginx 连接配置](./04-connection-management.md#服务器最佳实践) - keepalive、上游连接池
- [第五章: Nginx Gzip 配置](./05-content-negotiation.md#accept-encoding---编码协商) - 启用压缩
- [第六章: Nginx 缓存配置](./06-caching.md#nginx-完整配置示例) - 完整示例

**前端开发**:
- [第一章: URI 结构](./01-introduction.md#13-资源与-uri) - 理解 URL
- [第二章: 常用头部](./02-message-format.md#重要头部详解) - Content-Type、Host
- [第六章: 浏览器缓存](./06-caching.md#浏览器缓存位置) - from memory cache、from disk cache

---

## 工具与资源

### 命令行工具

**curl** - HTTP 客户端:

```bash
# 查看完整的 HTTP 报文
curl -v https://www.example.com

# 只查看响应头部
curl -I https://www.example.com

# 发送 POST 请求
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Alice"}'

# 跟随重定向
curl -L https://github.com

# 启用压缩
curl -H "Accept-Encoding: gzip" --compressed https://www.example.com
```

**httpie** - 更友好的 HTTP 客户端:

```bash
# 安装
pip install httpie

# GET 请求
http https://api.github.com/users/octocat

# POST 请求
http POST https://httpbin.org/post name=Alice age:=25
```

**telnet** - 手动发送 HTTP 请求:

```bash
telnet www.example.com 80

GET / HTTP/1.1
Host: www.example.com

```

### 浏览器工具

**Chrome DevTools** (F12):
- **Network** 标签 - 观察 HTTP 请求和响应
- **Waterfall** 列 - 查看资源加载时序
- **Size** 列 - 查看缓存状态 (from cache、304、200)
- **Headers** 面板 - 查看完整的请求和响应头部
- **Timing** 面板 - 查看请求各阶段耗时

**在线工具**:
- [httpbin.org](https://httpbin.org/) - HTTP 请求测试
- [reqbin.com](https://reqbin.com/) - 在线 HTTP 客户端
- [webhookrelay.com](https://webhookrelay.com/) - Webhook 调试

### 服务器软件

**Nginx**:

```bash
# 安装
sudo apt install nginx

# 配置文件
/etc/nginx/nginx.conf
/etc/nginx/sites-available/default

# 测试配置
sudo nginx -t

# 重新加载
sudo nginx -s reload
```

**Node.js (Express)**:

```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello World');
});

app.listen(3000);
```

---

## RFC 规范参考

本教程基于以下 RFC 标准编写:

- **[RFC 9110 - HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)** (2022 年 6 月)
  - HTTP 协议的核心语义
  - 方法、状态码、头部字段定义
  - 内容协商、条件请求

- **[RFC 9111 - HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111.html)** (2022 年 6 月)
  - HTTP 缓存机制
  - Cache-Control、Expires、ETag、Last-Modified
  - 缓存新鲜度计算

- **[RFC 9112 - HTTP/1.1](https://www.rfc-editor.org/rfc/rfc9112.html)** (2022 年 6 月)
  - HTTP/1.1 消息语法
  - 连接管理
  - 分块传输编码

**历史 RFC** (已被上述 RFC 废弃):
- RFC 7230, 7231, 7232, 7233, 7234, 7235 (HTTP/1.1 旧规范)
- RFC 2616 (HTTP/1.1 原始规范,1999 年)

---

## 学习建议

### 推荐学习路径

**第 1 周: 基础入门**
- 阅读第一章和第二章
- 实践 curl 命令
- 观察浏览器 Network 面板

**第 2 周: 核心语义**
- 阅读第三章
- 测试不同的 HTTP 方法
- 理解常见状态码

**第 3 周: 性能优化**
- 阅读第四章和第六章
- 配置 Nginx 缓存
- 优化网站性能

**第 4 周: 高级特性**
- 阅读第五章
- 实现多语言 API
- 完整项目实战

### 实践项目建议

1. **静态博客网站**:
   - 配置 Nginx 缓存策略
   - 使用文件指纹 (Webpack)
   - 启用 Gzip 压缩
   - 优化 Lighthouse 评分

2. **RESTful API**:
   - 实现 CRUD 操作 (GET、POST、PUT、DELETE)
   - 支持内容协商 (JSON、XML)
   - 实现 ETag 缓存验证
   - 返回正确的状态码

3. **图片 CDN**:
   - 支持多种图片格式 (WebP、JPEG、PNG)
   - 实现 Accept 协商
   - 配置长期缓存
   - 使用 Nginx 反向代理

### 常见问题 (FAQ)

**Q: HTTP/1.1 还有学习的必要吗? HTTP/2 和 HTTP/3 不是更新吗?**

A: 非常有必要!原因:
1. HTTP/1.1 是基础,理解它才能更好地理解 HTTP/2/3
2. 很多网站仍在使用 HTTP/1.1
3. HTTP/2 兼容 HTTP/1.1 的语义
4. 调试和运维需要理解底层协议

**Q: 我应该先学 HTTP 还是先学 HTTPS?**

A: 建议先学 HTTP,再学 HTTPS。HTTPS 是 HTTP + TLS,理解 HTTP 后再学加密传输会更容易。

**Q: 如何调试 HTTP 问题?**

A: 推荐工具:
1. Chrome DevTools - 查看请求和响应
2. curl -v - 查看完整的 HTTP 报文
3. Wireshark - 抓包分析 (高级)
4. Nginx access.log 和 error.log

**Q: 缓存总是不生效,怎么办?**

A: 常见原因:
1. 检查响应是否包含 `Cache-Control` 或 `Expires`
2. 确认没有 `Cache-Control: no-store`
3. 检查 `Vary` 头部是否匹配
4. 清除浏览器缓存后重试 (Ctrl+Shift+Delete)

**Q: 如何学习 Nginx 配置?**

A: 推荐资源:
1. [Nginx 官方文档](https://nginx.org/en/docs/)
2. [Nginx 配置生成器](https://nginxconfig.io/)
3. 本教程中的 Nginx 配置示例
4. 实践中逐步学习

---

## 贡献与反馈

本教程基于 RFC 规范编写,力求准确和实用。如果您发现错误或有改进建议,欢迎:

- 提交 Issue 报告问题
- 提交 Pull Request 改进内容
- 分享您的学习心得

---

## 致谢

感谢以下资源和社区:

- **IETF (Internet Engineering Task Force)** - 制定 HTTP 标准
- **MDN Web Docs** - 优秀的 Web 技术文档
- **RFC Editor** - 维护 RFC 文档
- **开源社区** - Nginx、Node.js、curl 等工具

---

## 许可证

本教程内容基于 RFC 规范 (公开标准) 编写,仅供学习和参考使用。

**RFC 版权声明**:
- RFC 9110, 9111, 9112 © 2022 IETF Trust
- 详见各 RFC 文档的 Copyright Notice 部分

---

**开始学习**: [第一章: HTTP 核心概念](./01-introduction.md) →
