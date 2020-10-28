# Transport Guarantees

Before diving into any protocol specifics, lets define what we mean by terminology that is often used while talking about protocols.
The protocols that we will be discussing share some combinations of guarantees noted on this page. 
In your protocol selection you must clearly have in mind what it that you need.    

## Ordering VS Sequencing

Packet arrival is possible in two ways. 
They can arrive in sequence and in order. 

Let's define two concepts here:
- "Sequencing: this is the process of only caring about the newest items."_ [1](https://dictionary.cambridge.org/dictionary/english/sequencing)
- "Ordering: this is the process of putting something in a particular order."_ [2](https://dictionary.cambridge.org/dictionary/english/ordering)

**Example**

- Sequencing: Only the newest items will be passed trough e.g. `1,3,2,5,4` which results in `1,3,5`.
- Ordering: All items are returned in order `1,3,2,5,4` which results in `1,2,3,4,5`.

Now, lets discuss those different transport guarantees a protocol can have. 

## The 5 Transport Guarantees

There are 5 main different ways you can transfer data:

| Transport Guarantees         | Packet Drop [(1)][1]  | Packet Duplication [(2)][2] | Packet Order [(3)](#ordering-vs-sequencing) | Packet Delivery |
| :-------------:              | :-------------: | :-------------:    | :-------------:  |  :-------------:
|   **Unreliable Unordered**   |       Any       |      Yes           |     No           |    No
|   **Unreliable Sequenced**   |    Any + old    |      No            |     Sequenced    |    No
|   **Reliable Unordered**     |       No        |      No            |     No           |    Yes
|   **Reliable Ordered**       |       No        |      No            |     Ordered      |    Yes
|   **Reliable Sequenced**     |    Only old     |      No            |     Sequenced    |    Only newest

UDP is unreliable, while TCP is reliable.
Reliability gives great uncertainty but a lot of freedom, while reliability gives certainty with costs in speed and freedom.
This is why protocols like QUIC build on UDP instead of TCP. 
UDP has few limitations that give the end-user more control over his transport. 

Many protocols play with the above-mentioned transport guarantees to support a certain use case. 
You can take protocols like RUDP, SCTP, QUIC, netcode, laminar as an example.    
It is therefore important to understand what you need for your use-case.

[1]: https://en.wikipedia.org/wiki/Packet_loss
[2]: https://observersupport.viavisolutions.com/html_doc/current/index.html#page/gigastor_hw/packet_deduplicating.html
