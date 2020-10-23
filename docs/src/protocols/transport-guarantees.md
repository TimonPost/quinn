Before diving into any protocol specifics, lets define what we mean by terminology that is often used while talking about protocols.
The protocols that we will be discussing share some combinations of grantees notes on this page. 
In your protocol selection for the right protocol you must clearly have in mind what it is for combination that you need.   

## Ordering VS Sequencing
Let's define two concepts here:
_"Sequencing: this is the process of only caring about the newest items."_ [1](https://dictionary.cambridge.org/dictionary/english/sequencing)
_"Ordering: this is the process of putting something in a particular order."_ [2](https://dictionary.cambridge.org/dictionary/english/ordering)

- Sequencing: Only the newest items will be passed trough e.g. `1,3,2,5,4` which results in `1,3,5`.
- Ordering: All items are returned in order `1,3,2,5,4` which results in `1,2,3,4,5`.

Now, lets discuss those different transport guarantees a protocol can have. 

## The 5 Reliability Guarantees
There are 5 different ways you can transfer data:

| Reliability Type             | Packet Drop     | Packet Duplication | Packet Order     | Packet Delivery|
| :-------------:              | :-------------: | :-------------:    | :-------------:  |  :-------------:
|   **Unreliable Unordered**   |       Any       |      Yes           |     No           |    No
|   **Unreliable Sequenced**   |    Any + old    |      No            |     Sequenced    |    No
|   **Reliable Unordered**     |       No        |      No            |     No           |    Yes
|   **Reliable Ordered**       |       No        |      No            |     Ordered      |    Yes
|   **Reliable Sequenced**     |    Only old     |      No            |     Sequenced    |    Only newest


### Unreliable
Unreliable: Packets can be dropped, duplicated or arrive in any order.

**Details**

| Packet Drop     | Packet Duplication | Packet Order     | Packet Delivery |
| :-------------: | :-------------:    | :-------------:  | :-------------: |
|       Any       |      Yes           |     No           |   No

Basically just bare UDP. The packet may or may not be delivered.

### Unreliable Sequenced
Unreliable Sequenced: Packets can be dropped, but could not be duplicated and arrive in sequence.

*Details*

| Packet Drop     | Packet Duplication | Packet Order     | Packet Delivery |
| :-------------: | :-------------:    | :-------------:  | :-------------: |
|    Any + old    |      No            |     Sequenced    |   No

Basically just bare UDP, free to be dropped, but has some sequencing to it so that only the newest packets are kept.

### Reliable Unordered
Reliable UnOrder: All packets will be sent and received, but without order.

*Details*

|   Packet Drop   | Packet Duplication | Packet Order     | Packet Delivery |
| :-------------: | :-------------:    | :-------------:  | :-------------: |
|       No        |      No            |     No           |   Yes

Basically, this is almost TCP without ordering of packets.

### Reliable Ordered
Reliable Unordered: All packets will be sent and received, but in the order in which they arrived.

*Details*

|   Packet Drop   | Packet Duplication | Packet Order   | Packet Delivery |
| :-------------: | :-------------:    | :-------------:| :-------------: |
|       No        |      No            |     Ordered    |   Yes

Basically this is almost like TCP.

### Reliable Sequenced
Reliable; All packets will be sent and received but arranged in sequence.
Which means that only the newest packets will be let through, older packets will be received but they won't get to the user.

*Details*

|   Packet Drop   | Packet Duplication | Packet Order     | Packet Delivery |
| :-------------: | :-------------:    | :-------------:  | :-------------: |
|    Only old     |      No            |     Sequenced    |   Only newest

Basically this is almost TCP-like but then sequencing instead of ordering.
