# Networking notes

## TCP
TCP protocol is a connection-oriented protocol, which means that a connection is established and maintained until the application programs at each end have finished exchanging messages.

To establish a connection, TCP uses 3-way handshake.

```text
Client          Server
  | ------SYN-----> |
  | <----SYN/ACK--- |
  | ------ACK-----> |
```

* Client will send the fist package (SYN).
* The server will response with SYN/ACK, which means "I received your SYN".
* The client will send ACK to validate the connection.

To terminate a connection, TCP uses 4-way handshake.

```text
Client          Server
  | ------FIN-----> |
  | <-----ACK------ |
  | <-----FIN------ |
  | ------ACK-----> |
```
This also can be done with 3-way handshake, that the server will send to the client with a FIN/ACK and the client will replies ACK.
* Notes:
  * If the ACK flag is set then the value if this field is the next sequence number that the sender of the ACK is expecting.
### Reliable transmission
TCP uses a sequence number to identify each byte of data. The sequence number identifies the order of the bytes sent from each computer so that the data can be reconstructed in order, regardless of any out-of-order delivery that occur.

The sequence number of the first byte is chosen by the transmitter for the first packet, which is flagged SYN.

### Dupack-based retransmission
When a packet is lost in a stream, the receiver will resend ACK with the sequence number of the last packet acknowledged.
This duplicate acknowledgement is used as a signal for packet loss.

```text
Client          Server
  | <-----Seg 1---- |
  | ------ACK 2---> |
  |   X <-Seg 2 ----|
  | <-----Seg 3---- |
  | <-----Seg 4---- |
  | ------ACK 2---> |
  | <-----Seg 2---- |
  | <-----Seg 3---- |
  | <-----Seg 4---- |
  | ------ACK 3---> |
  | ------ACK 4---> |
```

This design is inefficient because, although only packet #2 was lost, the server was required to retransmit packet #3 and #4 as well, because the client had no way to confirm that it had received those packets.

And that's how the Selective Acknowledgments come in.

### Selective Acknowledgments (SACKs)
SACKs work by appending to a duplicate acknowledgment packet a TCO option containing a range of noncontiguous data received. In other words, it allows the client to say "I only have up to packet #1 in order, but I also have received packets #3 and #4". This allow the server to retransmit only the packet(s) that were not received by the client.

```text
Client          Server
  | <-----Seg 1---- |
  | ------ACK 2---> |
  |   X <-Seg 2 ----|
  | <-----Seg 3---- |
  | <-----Seg 4---- |
  | ------ACK 2---> |
  | -ACK2, SACK 3-> |
  | ACK2, SACK 3,4> |
  | <-----Seg 2---- |
  | ------ACK 4---> |
```

In the last step, after the client receives segment #2, it will send to the server ACK #4 to indicate that it has received all data up to an including segment #4.

## HOL Blocking at TCP Layer (L4)
The HTTP/2 solved HOL blocking but just only at application level (L7), and HOL blocking at L4 still there.

The packets in a TCP stream must be passed in order. If one of the packets is lost, the all subsequent packets must be held in the receiver's TCP buffer util the lost packet is retransmitted and  arrives at the receiver.

Because this work is done within the TCP layer, HTTP layer has no visibility into the TCP retransmissions or the queued packet buffers and must wait for the full sequence before it can access the data.

It is quite possible that the already received packets can form complete request/response message for a different stream and can be delivered to the HTTP layer without waiting for the delayed packet. This effect is known as TCP head-of-line (HOL) blocking.

## References
* https://packetlife.net/blog/2010/jun/17/tcp-selective-acknowledgments-sack/
* https://en.wikipedia.org/wiki/Transmission_Control_Protocol
* https://www.youtube.com/watch?v=iddGrz_LRog&ab_channel=RickGraziani
* https://www.varlog.co.in/blog/hol-blocking-http/
