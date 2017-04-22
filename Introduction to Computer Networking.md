# Introduction to Computer Networking

## 1.1 A day in the life of an application  

Common communication model of networked applications: a bidirectional, reliable byte stream

- client-server model: World Wide Web，简称Web(using HTTP)
- peer-to-peer model: BitTorrent(Tracker)
- skype: rendezvous or relay server

## 1.2 The 4 layer Internet model 

Link: packet is called **frame**

Network: packet is called **datagram**, 如果使用**Internet**，network层必须使用IP  

Transport: TCP packet is called **segment**, UDP packet is called datagram, ICMP packet is called message

Application: called **message** 

每一层只和对应的层交流，通过调用对下一层的API来利用下一层的服务，**socket**就是应用层对运输层的API

![4layer](4layer.PNG)

a **packet** has two types of fields: header fields and a payload field. The payload is typically a packet from the layer above.

a **hop** is a link between two routers

理解protocol：A **protocol **defines the format and the order of messages exchanged between two or more communicating entities, as well as the actions taken on the transmission and/or receipt of a message or other event.

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

ACK表明该值之前的bytes都收到了，下次从该值处的byte开始接收

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

## Finite State Machines(FSM)

state 

event：event that can cause a state transition

action：the action the **protocol** takes on making that state transition

## Reliable Communications

### 实现可靠连接需要解决以下问题：

- 如何发现丢包，如何保证包的顺序
- 发现丢包后如何重传
- 实现可靠连接可以不在双方维持状态，不过，Reliable communication typically benefits from have some state on each，由此产生的问题是：
  - 如何设置这些状态
  - 如何清除这次状态

### Flow control

解决的问题：Don’t send more packets than receiver can process

方法：

#### 1. stop and wait protocol

- At most one packet in flight at any time
- at most one packet的理解：被发送的、未被确认的packet数目为1

#### 2. sliding windows protocol

- At most N packets in flight at any time:Allow a “window” of unacknowledged packets in flight

- retransmission的表现：

  - RWS=1，protocol表现为GBN
  - SWS=RWS=N>=1，protocol表现为SR

- 根据RWS(receive window size)、SWS(send window size)确定sequence number:

  Supposed packets not delayed multiple timeouts, at least need RWS+SWS sequence numbers，

  关键需要足够的sequence number解决flow control中的duplicates问题，即how to detect duplicates, 如何检测新的packet，还是超时重传的以前已经接受到的packet

  原因：

  假如SWS=3，RWS=2，sequence number为3，即packet编号为0,1,2,0,1,2...(至少如此才能保证一个send window中的packet编号不同)，主机A为发送方，B为接收方。

  当B接收到第一、二、三的包(0,1,2)，交付给上层应用，发送三个ACK，接下需要第四、五个包(0,1)；

  三个ACK全部lost，则超时后，A会重传第一、二、三的包(0,1,2)，此时接收方B无法分辨出0,1是新的包(第四、五个包)，还是重传的已经接收的包；

  当sequence number为5，即packet编号为0,1,2,3,4；依然是上面的情况，B接下需要编号为3,4的包；此时新的包(3,4)与重复的包(0,1)就区分开了。而且，当B再次需要新的编号为0的包的时候，即B已接收并交付3,4，主机A处旧的编号为0的包一定已经ACK(否则B不能接收到3,4，因为SWS=3)，因此A发送过来的编号为0的包一定为新的包，不会有歧义，如图

  ![IMG_4285](IMG_4285.JPG)

### Retransmission strategies(*Top-down* P147-154)

#### 回退N步GBN(Go-Back-N): RWS=1

发送方使用一个retransmit timer，收到一个ACK，timer被重启

某个packet超时，则认为该packet丢失，重传整个send window，即重传所有已发送带还未确认过的packet

采用**cumulative ACK**，表明ACK number n以前及n在内的packets被正确接收

接收方不缓存乱序的packet，直接丢弃

#### 选择重传SR(Selective Repeat)

为每个packet维持一个timer

只有每个packet的timer超时，才认为该（一个）packet丢失，只重传该packet

采用selective ACK，ACK number为刚刚正确收到packet的sequence number

接收方缓存乱序的packet，If this packet has a sequence number equal to the base of the receive window, then this packet, and any previously buffered and consecutively numbered  packets are delivered to the upper layer

#### GBN和SR的优劣：

if there are bursts of losses, GBN is faster

假设一组连续的包丢失，SR需要等待其中的每一个packet的timer超时，才会重传该packet，由此计入了每个packet的timer和round trip time

而GBN则只需要等待一次timer超时，就会重传所有的包，timer和round trip time只计一次

### TCP实现可靠传输(P163-P169)

TCP的ACK、timer部分像GBN

TCP的重传、乱序的处理像SR

- timer：推荐使用一个retransmit timer

- TCP的ACK：基于byte，而非packet；cumulative ACK, 而且ACK number是指接下来要接收byte的sequence number

- TCP对乱序packet的处理：没有明确规定，可以丢弃，可以缓存(实际中采用)

- TCP的重传：for N packets, *n*th packet lost, but the remaining N – 1 acknowledgments arrive at the sender before their respective timeouts. In this example, GBN would retransmit not only packet n, but also all of the subsequent packets n + 1, n + 2, . . . , *N*. TCP, on the other hand, would retransmit at most one segment, namely, segment *n*. Moreover, TCP would not even retransmit segment n if the acknowledgment for segment n + 1 arrived before the timeout for segment *n*.

- TCP中的flow control：采用sliding window

  The TCP packet header includes a window size field for each side to communicate how large its receive window is.

  - receiver在TCP header中的window field设置RWS：

    `RWS = RcvBuffer – [LastByteRcvd - LastByteRead]` 

    其中LastByteRead: the number of the last byte in the data stream read from the buffer by the application process

    LastByteRcvd: the number of the last byte in the data stream that has arrived from the network and has been placed in the receive buffer

    ![rwnd](rwnd.PNG)

  - sender需要保证：`LastByteSent - LastByteAcked <= RWS` 

  - 当`RWS == 0`时，sender时不时发送仅含一个byte的“窗口探测”segment，以获取新的窗口大小的信息

### TCP Connection setup and teardown

#### setup

3-way handshake

simultaneous open

![three-handshake](three-handshake.JPG)

#### teardown

a SYN or FIN in a TCP segment consume a sequence number, it's so that the SYN and FIN themselves can be acknowledged (and therefore re-sent if they're lost).

![teardown](teardown.JPG)

- simultaneous close

![simultaneous_close](simultaneous_close.JPG)

- TIME_WAIT状态的作用

  -  to make sure that the final ACK is not lost

  - to avoid data corruption in the case that ports are immediately reused and there is a sequence number overlap

    https://vincent.bernat.im/en/blog/2014-tcp-time-wait-state-linux

- active close side如何确认它回复的ACK被passive close side成功接收了？

  如果passive close side没有收到ACK，会继续发FIN，只要active close side在2MSL中没有再收到FIN，则说明passive close side成功收到ACK。

  The final ACK is resent not because the TCP retransmits ACKs, but because the other side will retransmit its FIN. Indeed, TCP will always retransmit FINs until it receives a final ACK

![TCP_states](TCP_states.png)

## 网络工具使用

wireshark

nslookup: domain name to IP address

telnet: 

1. `telnet IP address port`
2. `ctrl + ]`
3. 进入telent后，回车进入编辑界面，编写要发送的HTTP message

ping

tracert