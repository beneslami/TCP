# TCP (Transmission Control Protocol)

TCP is a transport layer protocol designed for **Reliable Communication** between processes. The theoretical OSI model like below:

![picture](data/OSI.png)

is the acronym of Open System Interconnection which is a **conceptual model** that **characterizes and standardizes** the communication functions of a telecommunication or computing system **without regards to its underlying inernal structure and technology**. So, what is TCP/IP stack? TCP/IP stack is the **Practical Implementation** of OSI model. What is actually implemented inside OS is something like this:

![picture](data/TCP:IP.png)

Researchers and implementors have decided to put presentation and session layer implementations partially into application layer and transport layer. So, according to TCP/IP protocol stack, each layer is responsible for:
* physical layer: Transmit data as electrical signal (electrical engineers' concern!)
* data link layer: responsible for transmission of data from a node to its adjacent node (node to node delivery).
* network layer: responsible for transmission of data from a source machine to destination machine (host to host delivery).
* transport layer: responsible for transmission of data from a process in a machine to another process which resides in another machine(process to process delivery).
* application layer: apps such as Ping(ICMP), whatsApp, etc...

![picture](data/layers.png)

In this tutorial, I am going to dive into **Transport Layer** in much more detail. The goals of Transport Layer which is called **Socket Layer** are:
* Facilitate communication (data exchange) between **applications** running on different machines deployed in the network.
* Transport Layer provides two world-wide standardized famous protocols to achieve its goal:
 * User Datagram Protocol (UDP)
 * Transmission Control Protocol (TCP)  
 * SCTP which is a hybrid of TCP and UDP
* TCP and UDP both have the same goal: Facilitate data exchange between processes, but they do in a different ways (Further, I'm gonna explain about the "different ways").

Before, I proceed further about TCP, I'm going to explain about UDP because it is fairly simple and straight forward.
### UDP
The essence of UDP is that it works on **Send and Forget** model. What does that mean?
UDP protocol does not maintain any state of the peer it is communicating with. So what is that mean also?

Suppose there is an application on Src, let's say application A. It wants to send a data to application B which is on Dst. How to determine the exact address of the process B? By two parameters:
* **Port number**: Each application which as an interaction with network, is identified with specific port number.
* **IP address**: Each machine on which the target process is running, is identified with IP address.

So, by having (DATA, PORT, IP), we can send data from Src to Dst using both TCP and UDP. But, UDP protocol does not remember who it was communicating with after sending data (**connection less**), and it also forgets that it has actually sent any data (**Stateless Protocol**). So, each request comes from application layer to UDP protocol, considered a fresh request. And also, UDP protocol sends data to the destination as soon as it gets data from application layer without bothering itself to say hello to the destination to see if it is alive or not, or there exists such process or not.

UDP protocol sends data in **chunks** or **discrite individual units** called **Datagram**. Also it is an unreliable protocol, which means it does not care that the packet has actually reached the destination or not. In addition, it does not guarantee **ordered delivery**, which means it does not care if datagrams are received out of order.

### TCP
TCP is way more complex compared to UDP. Here I'm diving into TCP features without having any further explanation:
* TCP is **Connection Oriented**: Before sending and receiving any data between Src and Dst over TCP, they must mutually agree and know each other first.
* TCP is **Stateful**: Both sender and receiver keep track of their connection and sent/received data.
* TCP is **byte Oriented**: It sends and receives data as continuous flow of bytes. It ensures that every byte of data is received successfully. So, TCP keeps track of data at byte level.
* TCP is also able to handle out of order delivery of packets at receiving end.
* **Reliable Delivery**: TCP ensures that all application data bytes are delivered to recipient, and none should be missed. TCP sender and receiver jointly implements Reliable Delivery procedures. Therefore, TCP implements **ARQ (Automatic Repeat Request**) for data recovery.
![picture](data/retransmission.png)
Suppose Src(left laptop) sends two packets 1 and 2. The receiver(left side laptop) only received 2nd packets. There should be a mechanism to make the Src understand that 1st packet is not delivered via Rst. Then the sender should Re-Transmit lost packet. The act of Re-transmission is called **ARQ**.

1) how receiver detects that packets are malformed? Checksum field.

2) How sender can determine whether the receiver has received the packet?

3) How long the sender should wait for ACK from the receiver?

4) What if ACK itself is lost?

5) How receiver will manage when it receives packets out of sequence?

6) What if receiver is slower than sender Or receiver receives duplicate copies of the packet?

7) What if network itself is slower or recover over a period of time?

8) With how much rate should the sender sends the packets to receiver?

**TCP ARQ** mechanism takes above stated points into consideration to implement its reliable data delivery functionality over lossy network.

Why TCP is called **Byte Oriented** protocol?

TCP keeps track of application data sent and received at **Byte Level**. TCP sender and receiver keeps track of how many bytes is sent and received by keeping explicit track of each byte of data separately. Each byte of data is tracked by a unique id called **sequence No.** at either ends. However, sending and receiving speed may not be the same in both machines. Therefore, TCP sender and receiver both need **Sending and Receiving buffers** implemented as **circular queues**. Remember, TCP is bi-directional.
![picture](data/TCP_stream.png)
Note that, data in sender buffer can be categorized into three parts: data which is sent to the receiver, data which is about to send to the receiver and data which has not yet come from sending process. TCP protocol in transport layer receives application data from application layer and glues **TCP header** to the application data, forms **TCP Segment**.
![picture](data/segment.png)
Note that, TCP is byte Oriented, while IP protocol is packet Oriented. Simply, IP datagram is called **packet**. In receiving side, the transport layer identifies which process the packet belongs to by specifying port number in TCP header. Size of segments is determined dynamically and keeps on changing depending on network or recipient state. Also, TCP chooses segment size to avoid unnecessary fragmentation of IP layer. Segments contain "N" bytes of data, where N is segment size. For example if a segment size is 512bytes, it means there are 512 bytes of application data is in the segment. TCP stamps every byte it is sending in segments with a unique number called **sequence number**. The SEQ No of first byte is also treated a **Segment number**.
![picture](data/TCP-headers.png)
Following Segment number, there is **Acknowledgment Number** which is a sequence number of a segment which TCP receiver expects from the sender to send in the next segment. ACK NO. 2000 means "Hey! I have successfully received 1999 bytes you sent, Now I'm waiting for 2000 bytes and more"

**TCP Piggyback**: In the same segment, TCP sender can ship next payload bytes, specifying new sequence number and at the same time ACKnowledge the previous TCP data it has received from peer using ACK NO and ACK bit. To understand more, there are three types of segments: Data Segment, Pure Ack Segment, Data + Ack Segment.

Each time a sender sends a message, it starts a timer. Receiver sends an acknowledgment back to the sender when it receives a message, so that, sender knows that it successfully transmitted the message. If a message is lost, the timer goes off, and sender retransmits the data.

The concept of connection oriented is illustrated in below picture:
![picture](data/connection_oriented.png)

Each stage is executed one after another. For connection less programs, phase 1,2 and 4 are not present.
TCP connection is uniquely defined by 4 tuples:

[TCP client IP address, TCP client Port Address, TCP server IP address, TCP Server Port address]

Above tuple is a unique identity of TCP connection.

### Three way handshake

In three way handshake stage, client which is Active Opener, sends **TCP SYN** packet with a sequence number. Note that, TCP SYN packet is a packet which the SYN flag is set. (ISN is abbreviated of Initial sequence Number).

1) SYN = want to initiate TCP connection. All my future segments have seq no 100+. The packet does not contain any application data, consume 1 sequence number.

2) ACK- client's request for connection initiation specified in segment with seq no 101. All future segments from the sever will have seq no 1000+.

3) ACK- request specified in server's segment with sequence no 1000 -1 is accepted.

In the 1st and 2nd, each party is telling the other party the ISN it wishes to use. So, steps 1 and 2 combined is called sequence number synchronization. At the end of stage 2, client can send TCP data segments to the server. TCP server can only ACK the TCP data from client. TCP server cannot send its own TCP data segments to the client. So, this situation is called **uni-direction (Half Open) communication**. At the end of stage 3, the server has got the permission from the client, and now TCP server can also send data to the TCP client.(**bi-directional communication**)

In closing the connection, the approach is quite similar with that of establishing the connection. The client is active closer and the server is passive closer. For finishing the connection, first the client sends a **FIN** message. The client wishes to terminate the connection using close(). Server receives the connection termination request, and acknowledges the request by sending ACK. When the client receives the Ack from the server, it closes the connection. So, after the second stage, the client has closed the connection successfully. After this point, the client cannot send segments with progressive seq# anymore. However, it can only ACKnowledge the segments coming from the server(Half close). Since the server knows that the client is looking to terminate the connection, it will also initiate connection termination by sending FIN segment to the client. At the 4th stage, the client sends ACK confirming that 1600th segments has been received and approves the connection termination request by sending (ACK 1601). Note that, the sequence no in state 2 and 3 is the same.
![picture](data/threewayhandshake.png)

some useful notes:
* SYN segments do not contain any application data, yet they consume 1 sequence number because they need to be acknowledged.
* FIN segments **MAY** not contain any application data, yet they consume at least 1 sequence number because they need to be acknowledged.
* Pure ACKs do not contain any application data, they do not consume any sequence number either because ACKs are not acknowledged.
* Data segments consume as many sequence numbers as the number of application bytes they are carrying as payload.

Rule: **Any segments that needs an acknowledgment consumes a sequence number**.

Now, suppose there is a dead server and the client wants to reach it out. The client sends SYN message but it does not receive any SYN/ACK message from the server. The client waits for some times and resend SYN again, and there is no answer still. For how long the client tends to send the server its SYN message?
Answer: By default, number of maximum retries is **5**. But, there is another concept called **Exponential Back-off**. Each time the client does not receive any response from the server **for a specific time**, it re-sends it. The specific time starts from one and grows exponentially with the power of 2.
![picture](data/backoff.png) 
