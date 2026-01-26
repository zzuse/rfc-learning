```
电子邮件（Email）相关的核心 RFC 标准主要涉及传输、消息格式、MIME 扩展和安全性。最关键的包括：负责传输的 RFC 5321（SMTP）和负责内容格式的 RFC 5322（邮件格式），它们取代了旧的 RFC 821/822。 

# 核心邮件格式和结构
RFC 5322 （Internet Message Format）：目前仍然是核心标准，但被 RFC 6854 进行了少量更新，主要涉及更新邮件头中“From"字段的作者指定方式。
MIME （RFC 2045, 2046,2047）：这些RFCS仍然是MIME标准的基础，但多年来已经有许多其他的RFCs对其进行了扩展和补充，以支持新的媒体类型和功能。
# 邮件传输协议
RFC 5321（SMTP）：仍然是SMTP协议的核心标准，但被 RFC 7504进行了更新，该更新主要关于SMTP的实现和操作建议。
RFC 1939（POP3）：仍然是POP3协议的核心标准，但有多个RFC对其进行了更新和扩展，包括RFC 1957, RFC 2449, RFC 6186,and RFC 8314，这些更新主要涉及扩展和安全性的改进。
RFC 3501（IMAP4）：已被 RFC 9051 （Internet Message Access Protocal
（IMAP）- Version 4rev2）完全取代。RFC 9051整合并澄清了自RFC3501发布以来的所有更新和勘误。
# 电子邮件安全
RFC 6376 （DKIM）：仍然是DKIM的核心标准，但有多个RFC对其进行了更新，包括 RFC 8301，RFC 8463, RFC 8553, and RFC 8616，这些更新主要涉及加密算法和操作的改进。
RFC 7208（SPF）：仍然是SPF的核心标准，但有多个RFC对其进行了更新，包括 RFC 7372，
RFC 8553, and RFC 8616.
RFC 7489（DMARC）：仍然是DMARC的核心标准，但有多个RFC对其进行了更新，包括 RFC8553 and RFC8616。另外，一个名为 DMARCbis的新规范正在制定中，预计将作为新的标准发布，以解决原始协议的一些局限性。
# 其他相关 RFC:
RFC 2231: 用于 MIME 参数值和编码字的扩展。
RFC 2142: 定义了通用的服务和角色邮箱名称（如 postmaster, abuse）。
RFC 6530: 国际化电子邮件概述（支持非英文邮件地址）。
RFC 1035: 域名系统 (DNS) 规范，用于 MX 记录解析
```
