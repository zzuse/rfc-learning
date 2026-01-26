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

# Instructions for webrtc protocol

based on sources/rfc8445.txt, sources/rfc5389.txt, sources/rfc8656.txt, sources/rfc6347.txt, sources/rfc8261.txt, sources/rfc3711.txt, sources/rfc4960.txt, sources/rfc8831.txt, sources/rfc9429.txt, sources/rfc8827.txt and sources/rfc9725.txt  please help to write a complete tutorials to introduce a suite of protocols related to webrtc in Chinese. Make the writing style as a tutorial.

## Requirements

1. The doc should be in Chinese, complete tutorial to introduce given RFC, easy to understand and follow.
2. Please use your knowledge to connect the RFC to the read world linux implementation. For example, for fragmentation, you can use the linux implementation to explain the RFC.
3. Please include enough mermaid diagrams to explain the RFC.
4. Please be detailed, cover all the details of the RFC.
5. Please put the content in md format, under rfcs/docs/<protocol_name> folder. You could choose to generate multiple files if needed.
