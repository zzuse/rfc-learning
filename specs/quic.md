```
QUIC is a modern, UDP-based, secure transport protocol developed by Google and standardized by the IETF in 2021 (RFC 9000). It improves upon TCP by reducing connection latency, solving head-of-line blocking, and natively integrating TLS 1.3, enabling faster web performance (especially for HTTP/3). Key related RFCs include 9001 (TLS), 9002 (Congestion Control), 9114 (HTTP/3), and 9221 (Datagrams). 
Core QUIC RFCs (The Foundation)
RFC 9000 (QUIC: A UDP-Based Multiplexed and Secure Transport): The main specification defining the core protocol, transport mechanisms, and frame formats.
RFC 9001 (Using TLS to Secure QUIC): Details how TLS 1.3 is integrated into QUIC for packet encryption and connection security.
RFC 9002 (QUIC Loss Detection and Congestion Control): Describes how QUIC detects packet loss and manages network congestion.
RFC 8999 (Version-Independent Properties of QUIC): Defines invariant properties that remain consistent across different versions of the protocol. 
Key Related and Extension RFCs
RFC 9114 (HTTP/3): Maps HTTP semantics over the QUIC transport protocol.
RFC 9204 (QPACK): Defines header compression for HTTP/3, similar to HPACK for HTTP/2.
RFC 9221 (An Unreliable Datagram Extension to QUIC): Allows for sending unreliable data over a reliable, secured connection.
RFC 9369 (QUIC Version 2): Defines an updated, more efficient version of the protocol.
RFC 9250 (DNS over Dedicated QUIC Connections): Used for secure, low-latency DNS queries.
RFC 9308 (Applicability of the QUIC Transport Protocol): Guidance on when to use QUIC, such as in web browsers and servers.
RFC 9312 (Manageability of the QUIC Transport Protocol): Provides guidance for network operators on managing QUIC traffic. 
Key Features of QUIC
Low Latency: 0-RTT or 1-RTT connection setup.
Multiplexing: Multiple streams over a single connection without head-of-line blocking.
Connection Migration: Connections remain active even if the client IP address changes (e.g., switching from Wi-Fi to cellular).
Built-in Encryption: TLS 1.3 is mandatory. 
```

# Instructions for QUIC protocol

based on sources/rfc9000.txt and sources/rfc9001.txt sources/9002.txt sources/8999.txt sources/9114.txt sources/9204.txt sources/9221.txt sources/9369.txt sources/9250.txt sources/9308.txt and sources/9312.txt please help to write a complete tutorials to introduce QUIC protocol in Chinese. Make the writing style as a tutorial.

## Requirements

1. The doc should be in Chinese, complete tutorial to introduce given RFC, easy to understand and follow.
2. Please use your knowledge to connect the RFC to the read world linux implementation. For example, for fragmentation, you can use the linux implementation to explain the RFC.
3. Please include enough mermaid diagrams to explain the RFC.
4. Please be detailed, cover all the details of the RFC.
5. Please put the content in md format, under rfcs/docs/<protocol_name> folder. You could choose to generate multiple files if needed.
