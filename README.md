# TFTP Trivial File Transfere Protocol

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


### 3. Relation to other Protocols
- Since Datagram is implemented on the
Internet protocol, packets will have an Internet header, a Datagram
header, and a TFTP header.
- TFTP does not specify any of the
values in the Internet header. On the other hand, the source and
destination port fields of the Datagram header (its format is given
in the appendix) are used by TFTP and the length field reflects the
size of the TFTP packet.
- The Transfer IDentifiers (TID’s) used by
TFTP are passed to the Datagram layer to be used as ports; therefore
they must be between 0 and 65,535. The initialization of TID’s is
discussed in the section on initial connection protocol.
- The TFTP header consists of a 2 byte opcode field which indicates
the packet’s type (e.g., DATA, ERROR, etc.) These opcodes and the
formats of the various types of packets are discussed further in the
section on TFTP packets.

### 4. Initial Connection Protocol
- A transfer is established by sending a request (WRQ to write onto a
foreign file system, or RRQ to read from it), and receiving a
positive reply, an acknowledgment packet for write, or the first data
packet for read.
- In general an acknowledgment packet will contain
the block number of the data packet being acknowledged.
- Each data packet has associated with it a block number; block numbers are
consecutive and begin with one.
- Since an acknowledgment packet is acknowledging a data packet, the acknowledgment packet will
contain the block number of the data packet being acknowledged.
- If the reply is an error packet, then the request has been denied.
 
 ---
 
- In order to create a connection, each end of the connection chooses a TID for itself, to be used for the duration of that connection.
- TID’s chosen for a connection should be randomly chosen, so that the probability that the same number is chosen twice in immediate succession is very low.
- Every packet has associated with it the two TID’s of the ends of the connection, the source TID and the destination TID.
- These TID’s are handed to the supporting UDP (or other datagram protocol) as the source and destination ports
- A requesting host chooses its source TID, and sends its initial request to the known TID 69 decimal (105 octal) on the serving host.
-  The response to this request, uses a random TID chosen by the server as its source TID and the TID chosen for the previous message by the requestor as its destination TID.
- The two chosen TID’s are then used for the remainder of the transfer.

**Example shows how to establish a connection to write a file**

1. Host A sends a "WRQ" to host B with source= A’s TID, destination= 69.
2. Host B sends a "ACK" (with block number= 0) to host A with source= B’s TID, destination= A’s TID.

**Wrong port error**
- In the next step, and in all succeeding steps, the hosts should make sure that the source TID matches the value that was agreed on in steps 1 and 2. 
- If a source TID does not match, the packet should be discarded as erroneously sent from somewhere else. 
- An error packet should be sent to the source of the incorrect packet, while not disturbing the transfer.

**Example for this error type**

The following example demonstrates a correct operation of the protocol in which the above situation can occur. Host A sends a request to host B. Somewhere in the network, the request packet is duplicated, and as a result two acknowledgments are returned to host A, with different TID’s chosen on host B in response to the two requests. When the first response arrives, host A continues the connection. When the second response to the request arrives, it should be rejected, but there is no reason to terminate the first connection. Therefore, if different TID’s are chosen for the two connections on host B and host A checks the source TID’s of the messages it receives, the first connection can be maintained while the second is rejected by returning an error packet.

### 5. TFTP Packets

TFTP supports five types of packets, all of which have been mentioned
above:
|opcode | operation|
|---|----|
|1 | Read request (RRQ)|
|2 | Write request (WRQ)|
|3 | Data (DATA)|
|4 | Acknowledgment (ACK)|
|5 | Error (ERROR)|

The TFTP header of a packet contains the opcode associated with that packet.

**5.1RRQ/WRQ packet**

|2 bytes|     string  |  1 byte   |  string |  1 byte |
|-|-|-|-|-|
| Opcode | Filename | 0 | Mode | 0| 

- The file name is a sequence of bytes in netascii terminated by a zero byte. 
- The mode field contains the string "netascii", "octet", or "mail".
- Octet mode is used to transfer a file that is in the 8-bit format of the machine from which the file is being transferred.
- It is assumed that each type of machine has a single 8-bit format that is more common, and that that format is chosen. 

**5.2 DATA packet**

|2 bytes  |   2 bytes    |  n bytes|
|---------|--------------|-----------|
| Opcode | Block # | Data | 


- Data is actually transferred in DATA packets depicted in Figure 5-2. DATA packets (opcode = 3) have a block number and data field. 
- The block numbers on data packets begin with one and increase by one for each new block of data. 
- This restriction allows the program to use a single number to discriminate between new packets and duplicates.
- The data field is from zero to 512 bytes long. If it is 512 bytes long, the block is not the last block of data; if it is from zero to 511 bytes long, it signals the end of the transfer. (See the section on Normal Termination for details.)


### 5.3 Acknowledgments 

All packets other than duplicate ACK’s and those used for termination are acknowledged unless a timeout occurs [4]. Sending a DATA packet is an acknowledgment for the first ACK packet of the previous DATA packet. The WRQ and DATA packets are acknowledged by ACK or ERROR packets, while RRQ and ACK packets are acknowledged by DATA or ERROR packets.

|2 bytes  |   2 bytes|
|---------|------------|
| Opcode | Block # |

- The opcode is 4. 
- The block number in an ACK echoes the block number of the DATA packet being acknowledged. 
- A WRQ is acknowledged with an ACK packet having a block number of zero.

### 5. 4 Error Packet

|2 bytes |    2 bytes  |    string  |  1 byte |
|----------|-----------|-------------|-------|
| Opcode | ErrorCode | ErrMsg | 0 | 

- An ERROR packet (opcode 5) takes the form depicted in Figure 5-4. 
- An ERROR packet can be the acknowledgment of any other type of packet. 
- The error code is an integer indicating the nature of the error. 
- A table of values and meanings is given in the appendix. (Note that several error codes have been added to this version of this document.) 
- The error message is intended for human consumption, and should be in netascii. Like all other strings, it is terminated with a zero byte.
