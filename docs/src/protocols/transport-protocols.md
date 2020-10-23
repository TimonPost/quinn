# Networking protocols
The internet is unreliable, internet is changing every second, cables could be cut, congestion on the network can defer etc. 
Once we send any package it could take any path to eventually arrive at its destination. 

It is an excellent question to ask what protocol suits your project the most. 
Different protocols serve different use cases and the wrong protocol can be catastrophic. 
Before jumping directly into the meat of QUIC, it can be usefull to understand its underlying motivations. 
For those motivations we have to inspect the flaws of TCP and nature of UDP.
 
If your already familiar with terminologies as IP/TCP/UDP and their guarantees and differences feel free to skipp to this section. 
For this section we will be using the [Internet protocol suite][internet-protocol-suite] as a guidance. 

![OSI model](../../images/osi-model.png)

## IP  - Internet layer

All communication over the internet is happening ontop of IP (Internet Protocol). 
The internet protocol works by splitting data into little chunks called datagrams or packets. 
The chunks are then sent across the internet from one IP address to another.
However, this protocol transfers packets across the network without any guarantee and it is by nature [unreliable][unreliable].
For certain applications, we need certain specific guarantees. 
This is exactly were transport protocols, like TCP, UPD, and application protocols, like QUIC, HTTP, come in. 

## TCP/IP and UDP comparison - Transport layer

**TCP:** stands for 'transmission control protocol' and adds certain guarantees ontop of [IP](#ip). 
It forms the backbone for almost everything you do online, from web browsing to IRC to email to file transfer, it’s all built on top of TCP/IP.

**UDP** stands for 'user datagram protocol' and it’s another protocol built on top of IP, but unlike TCP, 
instead of adding lots of features and complexity, UDP is a very thin layer over IP and is [unreliable][unreliable] in nature.

| Feature |  TCP  | UDP |
| :-------------: | :-------------: | :-------------:    |
|  [Connection-Oriented][connection-oriented]  |       Yes                              |      No                       |
|  Reliability Guarantees                      | [Reliable Ordered][reliable-ordered]   |      [Unreliable][unreliable] |
|  Packet Transfer                             | [Stream-based][stream-based]           |      Message based            |
|  Automatic [fragmentation][ip-fragmentation] | Yes                                    |      Yes, but better is to stay below datagram size limit |
|  Header Size                                 |  20 bytes                              |      8 bytes                  |
|  [Control Flow, Congestion Avoidance/Control][congestion-control] | Yes               |      No                       |                                            

[stream-based]: https://en.wikipedia.org/wiki/Stream_(computing)
[congestion-control]: https://en.wikipedia.org/wiki/TCP_congestion_control
[connection-oriented]: https://en.wikipedia.org/wiki/Connection-oriented_communication
[ip-fragmentation]: https://en.wikipedia.org/wiki/IP_fragmentation
[unreliable]: ./transport-guarantees.md#unreliable
[reliable-ordered]: ./transport-guarantees.md#reliable-ordered
[reliable-sequenced]: ./transport-guarantees.md#reliable-sequenced
