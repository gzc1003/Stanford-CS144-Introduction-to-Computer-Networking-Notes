# Introduction to Computer Networking

## 1.1 A day in the life of an application  

Common communication model of networked applications: a bidirectional, reliable byte stream

- client-server model: World Wide Web(using HTTP)
- peer-to-peer model: BitTorrent(Tracker)
- skype: rendezvous or relay server

## 1.2 The 4 layer Internet model 

Link: packet is called **frame**

Network: packet is called **datagram**, 如果使用**Internet**，network层必须使用IP  

Transport: packet is called **segment**

Application 

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

## 1-5: The Principle of Packet Switching

**Packet switching** is the idea that we break our data up into discrete, self-contained chunks of data. Each chunk, called a **packet**, carries sufficient information that a network can deliver the packet to its destination. 

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