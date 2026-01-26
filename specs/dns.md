
```txt
RFC 1034 ("Domain Names - Concepts and Facilities") and RFC 1035 ("Domain Names - Implementation and Specification"), both published in 1987. These foundational documents define the concepts, structure, and technical specifications for DNS, with many subsequent updates. 
Key details regarding DNS RFCs include:
RFC 1034: Defines the concepts, domain name space, and basic operations.
RFC 1035: Specifies the implementation, including the message format, query/response structure, and resource records.
Important Updates: Other essential RFCs include RFC 2136 (DNS Update), RFC 1996 (DNS NOTIFY), and RFC 4033 (DNSSEC).
Terminology: RFC 9499 provides current terminology for DNS
DNS over HTTPS (DoH) is defined in RFC 8484, "DNS Queries over HTTPS (DoH)", published in October 2018 by the IETF. It specifies a protocol for sending encrypted DNS queries and receiving responses over HTTPS (port 443), enhancing privacy by hiding DNS traffic within standard web traffic. It supports both HTTP/2 and HTTP/3.
RFC 9230 (Oblivious DNS over HTTPS - ODoH): An experimental protocol that adds an extra layer of privacy
RFC 7858 (DNS over TLS - DoT): Specifies how to send DNS queries over a dedicated TLS-encrypted connection (port 853)
RFC 8446 (TLS 1.3): The encryption standard used to secure both DoH and DoT.
RFC 7871 (EDNS0 Client Subnet): Defines a mechanism for sending part of a client's IP address to a recursive resolver. 
```

# Instructions for DNS protocol

based on sources/rfc1034.txt and sources/rfc1035.txt please help to write a complete tutorials to introduce DNS protocol in Chinese. Make the writing style as a tutorial.

## Requirements

1. The doc should be in Chinese, complete tutorial to introduce given RFC, easy to understand and follow.
2. Please use your knowledge to connect the RFC to the read world linux implementation. For example, for fragmentation, you can use the linux implementation to explain the RFC.
3. Please include enough mermaid diagrams to explain the RFC.
4. Please be detailed, cover all the details of the RFC.
5. Please put the content in md format, under rfcs/docs/<protocol_name> folder. You could choose to generate multiple files if needed.
