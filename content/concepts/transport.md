---
title: Transport
weight: 1
---

When you make a connection from your computer to a machine on the internet,
chances are pretty good you're sending your bits and bytes using TCP/IP, the
wildly successful combination of the Internet Protocol, which handles addressing
and delivery of data packets, and the Transmission Control Protocol, which
ensures that the data that gets sent over the wire is received completely and in
the right order.

当您从计算机连接到互联网上的计算机时，很可能使用TCP / IP发送您的位和字节，这是处理数据包寻址和传送的Internet协议的非常成功的组合，和传输控制协议，确保通过线路发送的数据完全按正确的顺序接收。

Because TCP/IP is so ubiquitous and well-supported, it's often the default
choice for networked applications. In some cases, TCP adds too much overhead,
so applications might use [UDP](https://en.wikipedia.org/wiki/User_Datagram_Protocol),
a much simpler protocol with no guarantees about reliability or ordering.

由于TCP / IP无处不在且得到很好的支持，因此它通常是网络应用程序的默认选择。在某些情况下，TCP会增加太多开销，因此应用程序可能会使用UDP，这是一种更简单的协议，无法保证可靠性或排序。

While TCP and UDP (together with IP) are the most common protocols in use today,
they are by no means the only options. Alternatives exist at lower levels
(e.g. sending raw ethernet packets or bluetooth frames), and higher levels
(e.g. QUIC, which is layered over UDP).


虽然TCP和UDP（以及IP）是当今最常用的协议，但它们绝不是唯一的选择。替代品存在于较低级别（例如，发送原始以太网分组或蓝牙帧）和较高级别（例如，QUIC，其在UDP上分层）。

In libp2p, we call these foundational protocols that move bits around
**transports**, and one of libp2p's core requirements is to be
*transport agnostic*. This means that the decision of what transport protocol
to use is up to the developer, and in fact one application can support many
different transports at the same time.

在libp2p中，我们称这些基本协议在传输中移动位，而libp2p的核心要求之一是与传输无关。这意味着决定使用什么传输协议取决于开发人员，实际上一个应用程序可以同时支持许多不同的传输。




## Listening and Dialing
Transports are defined in terms of two core operations, **listening** and
**dialing**.

传输是根据两个核心操作定义的，即侦听和拨号。

Listening means that you can accept incoming connections from other peers,
using whatever facility is provided by the
transport implementation. For example, a TCP transport on a unix platform could
use the `bind` and `listen` system calls to have the operating system route
traffic on a given TCP port to the application.

侦听意味着您可以使用传输实现提供的任何工具接受来自其他对等方的传入连接。 例如，unix平台上的TCP传输可以使用bind和listen系统调用，让操作系统将给定TCP端口上的流量路由到应用程序。

Dialing is the process of opening an outgoing connection to a listening peer.
Like listening, the specifics are determined by the implementation, but every
transport in a libp2p implementation will share the same programmatic interface.

拨号是打开与侦听对等方的传出连接的过程。 与监听一样，具体细节由实现决定，但libp2p实现中的每个传输都将共享相同的编程接口。


## Addresses

Before you can dial up a peer and open a connection, you need to know how to
reach them. Because each transport will likely require its own address scheme,
libp2p uses a convention called a "multiaddress" or `multiaddr` to encode
many different addressing schemes.

在您拨打对等方并打开连接之前，您需要知道如何联系它们。由于每个传输可能都需要自己的地址方案，因此libp2p使用称为“multiaddress”或multiaddr的约定来编码许多不同的寻址方案。

The [addressing doc](/concepts/addressing/) goes into more detail, but an overview of
how multiaddresses work is helpful for understanding the dial and listen
interfaces.

寻址文档更详细，但概述了多地址的工作原理有助于理解拨号和监听界面。

Here's an example of a multiaddr for a TCP/IP transport:

```
/ip4/7.7.7.7/tcp/6543
```

This is equivalent to the more familiar `7.7.7.7:6542` construction, but it
has the advantage of being explicit about the protocols that are being
described. With the multiaddr, you can see at a glance that the `7.7.7.7`
address belongs to the IPv4 protocol, and the `6543` belongs to TCP.

这相当于更熟悉的7.7.7.7:6542结构，但它具有明确描述正在描述的协议的优点。使用multiaddr，您可以一目了然地看到7.7.7.7地址属于IPv4协议，而6543属于TCP。

For more complex examples, see [Addressing](/concepts/addressing/).


有关更复杂的示例，请参阅寻址。

Both dial and listen deal with multiaddresses. When listening, you give the
transport the address you'd like to listen on, and when dialing you provide the
address to dial to.

When dialing a remote peer, the multiaddress should include the
[PeerId](/concepts/peer-identity/) of the peer you're trying to reach.
This lets libp2p establish a [secure communication channel](/concepts/secure-comms/)
and prevents impersonation.


拨号和收听都处理多地址。在收听时，您可以为传输机提供您想要收听的地址，并在拨号时提供要拨打的地址。

拨打远程对等方时，多地址应包含您尝试访问的对等方的PeerId。这使libp2p可以建立安全的通信通道并防止模拟。

An example multiaddress that includes a `PeerId`:

```
/ip4/1.2.3.4/tcp/4321/p2p/QmcEPrat8ShnCph8WjkREzt5CPXF2RwhYxYBALDcLC1iV6
```

The `/p2p/QmcEPrat8ShnCph8WjkREzt5CPXF2RwhYxYBALDcLC1iV6` component uniquely
identifies the remote peer using the hash of its public key.
For more, see [Peer Identity](/concepts/peer-id/).

{{% notice "tip" %}}

When [peer routing](/concepts/peer-routing/) is enabled, you can dial peers
using just their PeerId, without needing to know their transport addresses
before hand.

启用对等路由后，您可以仅使用PeerId拨打对等体，而无需事先知道其传输地址。

{{% /notice %}}

## Supporting multiple transports

libp2p applications often need to support multiple transports at once. For
example, you might want your services to be usable from long-running daemon
processes via TCP, while also accepting websocket connections from peers running
in a web browser.

libp2p应用程序通常需要同时支持多个传输。 例如，您可能希望通过TCP从长时间运行的守护程序进程中使用您的服务，同时还接受来自Web浏览器中运行的对等方的websocket连接。

The libp2p component responsible for managing the transports is called the
[switch][definition_switch], which also coordinates
[protocol negotiation](/concepts/protocol-negotiation),
[stream multiplexing](/concepts/stream-multiplexing),
[establishing secure communication](/concepts/secure-comms/) and other forms of
"connection upgrading".


负责管理传输的libp2p组件称为交换机，它还协调协议协商，流复用，建立安全通信和其他形式的“连接升级”。

The switch provides a single "entry point" for dialing and listening, and frees
up your application code from having to worry about the specific transports
and other pieces of the "connection stack" that are used under the hood.


该交换机为拨号和收听提供了一个“入口点”，使您的应用程序代码免于担心特定的传输和引擎盖下使用的“连接堆栈”的其他部分。

{{% notice "note" %}}
The term "swarm" was previously used to refer to what is now called the "switch",
and some places in the codebase still use the "swarm" terminology.

术语“swarm”以前用于指代现在称为“switch”的内容，代码库中的某些地方仍然使用“swarm”术语。

{{% /notice %}}

[definition_switch]: /reference/glossary/#switch


