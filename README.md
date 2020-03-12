# TFTP-Notes

## Motivation 
College course project [ **Computer Networks** ] 

Project documentation: [link](https://docs.google.com/document/d/1vQJH0F5o-q8BFCIdrF1i1xWYHEXrPvaQRS7dgELmnpQ/edit?usp=sharing)


## TFTP RFC Notes

### 1. Purpose
- TFTP is a simple protocol to transfer files between two machines implementing UDP.
- It has been implemented on top of the Internet User Datagram protocol
- The only thing it can do is read and write files (or mail) from/to a remote server.
- It has no provisions for user authentication
- It passes 8 bit bytes of data.
- Thee modes of transfere
    - Netascii 
    - Octet
    - Mail

### 2. Overview of the Protocol
- Any transfer begins with a request to read or write a file, which
also serves to request a connection.
- If the server grants the request, the connection is opened and the file is sent in fixed length blocks of 512 bytes
- Each data packet contains one block of data, and must be acknowledged by an acknowledgment packet before the
next packet can be sent.
- A data packet of less than 512 bytes signals termination of a transfer. 
- If a packet gets lost in the network, the intended recipient will timeout and may retransmit his
last packet (which may be data or an acknowledgment).
- This causing the sender of the lost packet to retransmit that lost packet.
- The sender has to keep just one packet on hand for retransmission
- The lock step acknowledgment guarantees that all older packets have
been received
- Notice that both machines involved in a transfer are
considered senders and receivers. One sends data and receives
acknowledgments, the other sends acknowledgments and receives data.

Errors:
- Most errors cause termination of the connection.
- An error is signalled by sending an error packet, This packet is not acknowledged, and not retransmitted.
- The other end of the connection may not get it. Therefore timeouts are used to detect such a termination when the error packet has been lost
- Errors are caused by three types of events:
    - not being able to satisfy the request (e.g., file not found)
    - receiving a packet which cannot be explained by a delay or duplication in the network (e.g., an incorrectly formed packet)
    - losing access to a necessary resource (e.g., access denied during a transfer)
- TFTP recognizes only one error condition that does not cause
termination, the source port of a received packet being incorrect.
In this case, an error packet is sent to the originating host.


This protocol is very restrictive, in order to simplify
implementation. For example, the fixed length blocks make allocation
straight forward, and the lock step acknowledgement provides flow
control and eliminates the need to reorder incoming data packets.
