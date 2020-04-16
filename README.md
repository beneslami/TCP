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
