# Problems of TCP 
In [the previous section](./transport-protocols.md) we compared TCP with UDP, now the golden question: Why should we prefer one over the other? 
One might ask: "Why choose so much uncertainty with UDP when TCP is so reliable and safe?". 
That's a good question to ask yourself. 
To answer that question we will have to delve a little deeper into how TCP works. 

## Head-of-line blocking

One of the biggest problem/feature in the TCP protocol is the Head-of-line blocking. 
It is a convenient feature because it ensures that all packages are sent in [order][order]. 
However, in areas of high throughput this can cause problems.   
One of those areas is a multiplayer fast-phases FPS game.

Lets check this animation out [animation][animation] to demonstrate the issue that we are facing.  
This animation shows that if a certain packet drops in transmission, all packets have to wait before it is resent and acknowledged by the other end.

Multiplayer action games are based on a constant stream of packets sent at a speed of 10 to 30 packets per second, and for the most part, 
the data in these packages are so time-sensitive that in most cases only the most recent data is useful.
You can think of the input of the player, the position of the player, the orientation and speed, and the state of the physical objects in the world.

Take, for example, a scenario in which 30 packets per second are sent with player positions.
If a single packet drops, we should ignore it and look for the next packet with player input that arrives 2 milliseconds later, we only care about the latest input, right?
But with TCP if there is a latency of 100 milliseconds and a single packet drops all packets sent during the retransmission (retransmission request (100 ms) + retransmission (100 ms)) will be queued until the lost packet is retransmitted. 
After 200ms this would mean 100 packets (200/2) are queue. 
A fast phased game can not afford those numbers. 

Gamenetworking is not the only area were this head-of-line blocking plays creates some problems. 
HTTP-2 uses multiplexing to attack this issue. 

**Solutions**

There are various solutions that fix this head-of-line blocking problem. 
- Multiplexing
- Rebuild a variation of TCP ontop of UDP (laminar, networksockets, netcode)

QUIC uses both multiplexing but also rebuild a reliability layer ontop of UDP. 

## Connection Setup
In standard HTTP+TLS+TCP, TCP needs a handshake to establish a session between server and client, and TLS needs its own handshake to ensure that the session is secured.

![TCP-handshake](../../images/tcp-handshake.svg.png)

First, the source sends an SYN “initial request” packet to the target server in order to start the dialogue. 
Then the target server then sends a SYN-ACK packet to agree to the process.
Lastly, the source sends an ACK packet to the target to confirm the process, after which the message contents can be sent. 
 
Now if we want to secure the TCP connection, we have to use TLS on top of that. If we use an older TLS version then 1.3 then there are three more handshakes that are required.

You can see how expensive it is to create a TCP connection. In a scenario of TCP and TLS 1.2 with a 100ms latancy we need to wait 6 x 100ms = 600ms for our website to be able to load. 
If the website is big in size, then an additional load time can bring the load time over one second. Which is of course disturbing for our short attention spans. 

[order]: ./transport-guarantees.md#ordering-vs-sequencing
[reliable-ordered]: ./transport-guarantees.md#reliable-ordered
[internet-protocol-suite]: https://en.wikipedia.org/wiki/Internet_protocol_suite
[animation]: https://dirkjan.ochtman.nl/files/head-of-line-blocking.html