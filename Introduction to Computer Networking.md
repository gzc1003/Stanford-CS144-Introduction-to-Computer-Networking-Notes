# Introduction to Computer Networking

## 1.1 A day in the life of an application  

Common communication model of networked applications: a bidirectional, reliable byte stream

- client-server model: World Wide Web，简称Web(using HTTP)
- peer-to-peer model: BitTorrent(Tracker)
- skype: rendezvous or relay server

## 1.2 The 4 layer Internet model 

Link: packet is called **frame**

Network: packet is called **datagram**, 如果使用**Internet**，network层必须使用IP  

Transport: packet is called **segment**

Application: called **message** 

每一层只和对应的层交流，通过调用对下一层的API来利用下一层的服务，**socket**就是应用层对运输层的API

![4layer](4layer.PNG)

a **packet** has two types of fields: header fields and a payload field. The payload is typically a packet from the layer above.

a **hop** is a link between two routers

## 1.3 The IP Service Model

### IP提供哪些服务？

基本服务：

- Datagram: 接受来自Transport层的segment作为datagram的data，将datagram作为Link层的data传给Link层
- Unreliable: 不保证按照顺序、一定到达、duplicate packet，而且发生这些错误也不会报告
- Best-effort
- Connectionless: 每个datagram之间是独立的，互相不知道的；IP service model doesn't contain a flow state that records the whole communication (it only cares about destination) in its network

更多服务：

- 防止loop，通过datagram header中的**time to  live(TTL)**阻止
- 如果Link层允许的packet大小小于datagram的，router可以**fragment** datagram
- 减小送错datagram的几率，通过datagram header中的**checksum**
- 允许新的IP版本：ipv4，ipv6
- 允许向datagram header中添加新的**field**从而添加新的feature

### 为什么如此设计IP？

- **end-to-end principle**：能在**end host**上加的feature，就不在network中加
- reliable和unreliable的服务都可以建在Network层上
- IP可以建在任何Link层上：IP makes very few assumptions about the link layer below
- 简单的IP快速、便宜，可以用dedicated hardware实现

## 1.4 A Day in the Life of a Packet

传输packet的过程：

![IMG_4257](IMG_4257.JPG)

Network layer is responsible for delivering packets to computers 

Transport layer is responsible for delivering packets to applications

- Tools

  - Wireshark

  - `traceroute -w 1 网址` 

    `-w 1`指明wait for reply的timeout为1s

    >**Traceroute** is a software program that displays the connection (path) through the Internet between this traceroute server and the location you enter above.  The Internet path between the two locations has many routers, computers and other devices along it which help move your information.  Each device along this path looks at your request and then sends it off to the next device until it reaches its destination. The Traceroute program shows the amount of time each device on the route takes to do three things: (1) receive the traceroute request (ICMP), (2) process that request and (3) send it back.  Traceroute performs this test three times on each device (each device is called a "hop").

  - windows中是`tracert` 

## 1-5: The Principle of Packet Switching

**Packet switching** is the idea that we break our data up into discrete, self-contained chunks of data. Each chunk, called a **packet**, carries sufficient information that a network can deliver the packet to its destination. 

属于Link层

### 设计理念

Packet switch(分组交换机): 不需要知道per-flow state，即packet属于哪个**flow**

而对应的在电话网络中应用的**circuit switching**，其switch是需要维护发送方-接收方connection state，并且为每个发送方-接收方维护一条连接链路

packet switching有两种方式：（1）**self routing**(不再使用)：packet的header中包含有它从source到destination的路线；（2）forwarding table: a switch can have a table of destination addresses and the next hop

**Flow**: A collection of datagrams belonging to the same end-to-end communication, e.g. a TCP connection.

### 好处

packet switch简单

多个flow可以充分利用link的能力(*Statistical Multiplexing*)

##  1-6: The Principle of Layering

layer只和上下层的layer交流，下层layer为上层提供了well-defined service，各layer可以随着时间改进而不影响其他层

举例：邮局系统；航空公司；编程

有时为了特殊目的，需要打破layering，比如在c语言中加入assembly code，使得上层和底层的processor不再相互独立。

## 1-7: The Principle of Encapsulation

![Packet](Packet.png)

Encapsulation allows you to layer recursively, 比如TCP segment可以在另一个TCP segment，应用VPN(virtual private network)

Encapsulation is how layering manifests in data representation

- Help separation of concerns


- Help enforce boundaries/layering


- Simplify layer implementations

##  1-8: Byte order and packet formats

![endian](endian.PNG)

![endian quiz](endianQuiz.JPG)

**Internet**使用的协议都是**big endian**，即packet to be in a big endian format

network (byte) order is big endian，网络编程时需要将**network order**转为**host order**

文本数据在内存中的layout与字节顺序无关，在big endian和little endian的机器上显示结果相同

## 1-9: IPv4 addresses and CIDR

### IP address: 

- 32 bits, written as 4 octets, `a.b.c.d`
- IP address是与主机或路由器的**接口(Interface)**相关联的，主机和物理链路的边界叫做**接口(Interface)**，路由器有多个接口，继而多个IP address

### Netmask:

- A netmask tells the device which IP addresses are local -- on the same link/network(不需要经过router转发) -- and which require going through an IP router.


- 连续的1，从最高位开始
- `if IP address1 && netmask == IP address2 && netmask`, then they belong to the same subnet

### IP address如何分配的

- IP address: network+host

- 方法一(不再使用)：Class A, Class B, Class C: **prefix(or network prefix)**固定为8,16,24bits

- 方法二：**CIDR**：define **a block of** address

  address block/CIDR block：`a.b.c.d/x` describes 2^(32-x) addresses

  `x` is so called **CIDR address/prefix**, when we talk about a CIDR address, we refer to its netmask length

- 现实中的实现：

  IANA gives out /8s to **Regional Internet Registries (RIRs)**. 

  RIRs each have their own policy for how they break up the /8s into smaller blocks of addresses and assign them to parties who need them

  RISs再提供给**Internet Service Providers (ISPs)**

 ## 1-10: Longest prefix match (LPM) 

**Router**中有**forwarding table**，其包含CIDR entry(describing **a block of** address)和next hop

IP address与CIDR entry `a.b.c.d/x`匹配意味着，IP address的前`x`bits与`a.b.c.d`的前`x`bits相同

LPM意味着如果IP address和多个CIDR entry匹配，选prefix`x`大的

0.0.0.0/0匹配任意IP address

![routingTable](routingTable.png)

## 1-11: Address resolution protocol (ARP)

### IP address and link address

| IP address       | link address(such as Ethernet address)   |
| ---------------- | ---------------------------------------- |
| 32 bits          | 48 bits,  a colon delimited set of 6 octets written in hexidecimal |
| describes a host | describes a particular network interface card/network adaptor |

举例：（1）网关：the gateway or router has multiple interfaces, each with their own link layer address to identify the card, and also each with their own network layer address to identify the host within the network that card is part of.

![encapsulation](encapsulation.png)

（2）PC has Ethernet card and wireless card, each has their own IP address and link layer address

### ARP过程

ARP用于由IP address找到link layer address(MAC address, MAC地址是随网卡预置的)，仅为同一子网中的主机和路由器

过程：

0. Every node keeps a cache of mappings from IP addresses on its network to link layer addresses. 


1. If a node needs to send a packet to an IP address it doesn’t have a mapping for, it sends a request. 以广播地址`ff:ff:ff:ff:ff:ff`作为帧的目的地址，将ARP packet封装到link层的frame中

   ![ARP](ARP.png)

2. Every node in the network receives it and refreshes its mapping between its link address，and its network address, or inserts a mapping if it doesn’t have one.

3. The node that has that network address responds, the replier whether send response to requester’s link layer address or broadcast it.

4. Nodes can also send what are called gratuitous ARP packets, requesting non-existent mappings, in order to advertise themselves on a network.

### 总结：发送IP datagram的过程（结合ARP）

![IMG_4275](IMG_4275.JPG)

## 2-0: Transport

Network layer的服务是在主机间传输数据

Transport layer的基本服务是在进程间传输数据，包括**多路复用(multiplexing)**和**多路分解(demultiplexing)**

一个进程有一个或多个套接字(socket)，The job of delivering the data in a transport-layer segment to the correct socket is called **demultiplexing**. The job of gathering data chunks at the source host from different sockets, encapsulating each data chunk with header information (that will later be used in demultiplexing) to create segments, and passing the segments to the network
layer is called **multiplexing**. 

更通用的讲，whenever a single protocol at one layer (at the transport layer or elsewhere) is used by multiple protocols at the next higher layer，就会有multiplexing and demultiplexing 

## 2-1: TCP service model

TCP establish a two-way communication channel between the TCP peers at both ends. We call the two way communication a **connection**. At both ends of the connection, TCP keeps a **state machine** to keep track of how the connection is doing. 

### TCP提供哪些服务

![TCP_service_model](TCP_service_model.png)

### TCP过程

![IMG_4280](IMG_4280.JPG)

### TCP segment构成

TCP将应用要传输的数据看成byte stream，sequence number和acknowledgment sequence number就是指在应用数据byte stream中的位置，而不是在TCP segments中的位置。

TCP’s use of sequence numbers reflects this view in that sequence numbers are over **the stream of transmitted bytes** and not over the series of transmitted segments.

The Sequence number indicates **the position in the byte stream** of the **first byte in the TCP Data field**

ACK表明该值之前的bytes都收到了，下次从该值除的byte开始接收

## 2-2: UDP service model

UDP(User Datagram Protocol/User Demultiplexing Protocol)

### UDP服务

![UDP_service_model](UDP_service_model.png)

### UDP datagram构成

the UDP checksum calculation also includes a portion of the IPv4 header as well, including the IP source and destination addresses and the protocol ID which has the value of 17 and tells us that the IP datagram carries UDP data. It allows the UDP layer to detect datagrams that were delivered to the wrong destination.

### 使用UDP的application

一类应用仅仅是request-response，request都在一个UDP datagram中，If the request is unsuccessful, it simply times out and is resent. 

- DNS(domain name system): turn a hostname into an IP address, the request is fully contained in one UDP datagram
- DHCP(Dynamic Host Configuration Protocol): DHCP helps a new host find out its IP address when it joins a network. 

另一类应用是自己实现retransmission

- A few real-time streaming audio and video services
- Early versions of NFS(the network file system)

## 2-3: ICMP service model

ICMP(Internet Control Message Protocol)是Transport Layer的protocol，但描述的是Network Layer的信息, 为Network Layer服务

### ICMP message构成

![ICMP_format](ICMP_format.jpg)

### 使用ICMP的application

- ping

  source host sends a Echo Request type ICMP message to destination host

  destination host sends a  Echo Response type ICMP message back

- traceroute：得到source host到达每个router和destination host的round trip delay

  1. source host发送UDP segment，其port number故意设置为不可用，IP datagram中的TTL field由1开始增加；
  2. 当TTL减为0时，router会发送给source host一个TTL Expired ICMP message；
  3. 当destination host最终收到UDP segment，找不到port，会发送给source host一个port un-reachable ICMP message；
  4. source host收到后停止发送UDP segment

## 2-4: End-to-end principle

### end-to-end principle

要想正确实现endpoint需要的功能，需要endpoint的相关信息，因此network不可能实现；

不过，出于提高performance的目的，network可以实现部分功能；

但是，最终保证endpoint功能正确性的责任，还是落在endpoint上

举例

- file transfer

  虽然link layer有error detection，不过这仅保证传输过程没有错误，不保证在endpoint上的储存没有错误。因此，保证文件正确到达的唯一方法只有end-to-end check

- TCP reliable delivery

  wireless link layers improve their reliability by retransmitting at the link layer. TCP will work correctly -- it will reliably transfer data -- **without this link layer help**. But the link layer help greatly improves TCP’s performance.

### strong end-to-end principle

功能不能在network中实现（一点儿也不行），只能在fringes处实现

因为如果出于某些performance的考虑，在network实现了endpoint部分功能，那么这些功能是基于endpoint的某些假设实现的，那么今后其他的endpoint都要受限于network这些假设。

即虽然network获得短期的性能提升，不过导致network未来难以改变。

## 2-5: Error detection

### 各种error detection算法能够检测出那些错误

|            | Checksum           | CRC(Cyclic Redundancy Check)             | MAC(Message Authentication Code) |
| ---------- | ------------------ | ---------------------------------------- | -------------------------------- |
| 100%检测出的错误 | a single bit error | a single burst of errors ≤ c bits long; an odd number of bit errors;  2 bits in error | None                             |
| 很大可能检测出的   | None               | `1 - 1 / 2^c`                            | `1 - 1 / 2^c`                    |
| 采用该算法的协议   | IP TCP             | Ethernet                                 | TLS                              |

### 各种算法如何实现(简要)

[burst error](https://en.wikipedia.org/wiki/Burst_error#cite_note-1) 

个人理解：

- a burst error of n bits error：连续n bits的错误
- burst error length：首个错误bit和最后一个错误bit之间的距离