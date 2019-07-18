---
title: "Glossary"
weight: 1
---

### Circuit Relay  线路中继

A means of establishing communication between peers who are unable to communicate directly, with the assistance of a third peer willing and able to act as an intermediary.

In many real-world peer-to-peer networks, direct communication between all peers may be impossible for a variety of reasons. For example, one or more peers may be behind a firewall or have [NAT traversal](#nat-traversal) issues. Or maybe the peers don't share any common [transports](#transport).

In such cases, it's possible to "bridge the gap" between peers, so long as each of them are capable of establishing a connection to a willing relay peer. If I only speak TCP and you only speak websockets, we can still hang out with the help of a bilingual pal.

Circuit relay is implemented in libp2p according to the [relay spec](https://github.com/libp2p/specs/tree/master/relay), which defines a wire protocol and addressing scheme for relayed connections.

线路中继

在愿意并且能够充当中间人的第三同伴的帮助下，在不能直接通信的同伴之间建立通信的手段。

在许多现实世界的对等网络中，由于各种原因，所有对等体之间的直接通信可能是不可能的。 例如，一个或多个对等体可能位于防火墙后面或具有NAT遍历问题。 或者也许同伴不共享任何共同的传输。

在这种情况下，只要每个对等体能够与愿意的中继对等体建立连接，就有可能“弥合对等体之间的差距”。 如果我只说TCP并且你只会说websockets，我们仍然可以在双语朋友的帮助下闲逛。

电路继电器根据继电器规范在libp2p中实现，该规范定义了用于中继连接的有线协议和寻址方案。

### Client / Server

A network architecture defined by the presence of central "server" programs which provide services and resources to a (usually much larger) set of "client" programs. Typically clients do not communicate directly with one another, instead routing all communications through the server, which is inherently the most privileged member of the network.

由中央“服务器”程序的存在定义的网络体系结构，其为（通常更大的）“客户端”程序集提供服务和资源。 通常，客户端不直接相互通信，而是通过服务器路由所有通信，服务器本质上是网络中最有特权的成员。

### DHT

A [distributed hash table](https://en.wikipedia.org/wiki/Distributed_hash_table) whose contents are spread throughout a network of participating peers. Much like an in-process hash table, values are associated with a key and can be retrieved by key. Most DHTs assign a portion of the addressable key space to nodes in a deterministic manner, which allows for efficient routing to the node responsible for a given key.

分布式哈希表，其内容分布在参与对等体的网络中。 与进程内哈希表非常相似，值与键相关联，可以通过键检索。 大多数DHT以确定的方式将一部分可寻址密钥空间分配给节点，这允许有效地路由到负责给定密钥的节点。

Since DHTs are a foundational primitive of many peer-to-peer systems, libp2p provides a [Kademlia](https://en.wikipedia.org/wiki/Kademlia)-based DHT implementation in [Go](https://github.com/libp2p/go-libp2p-kad-dht) and [javascript](https://github.com/libp2p/js-libp2p-kad-dht).

由于DHT是许多对等系统的基础原语，因此libp2p在Go和javascript中提供了基于Kademlia的DHT实现。

libp2p uses the DHT as the foundation for one of its [peer routing](#peer-routing) implementations, and systems built with libp2p often use the DHT to provide metadata about content, advertise service availability, and more.

libp2p使用DHT作为其对等路由实现之一的基础，使用libp2p构建的系统通常使用DHT提供有关内容的元数据，宣传服务可用性等。



### Connection

A libp2p connection is a communication channel that allows peers to read and write data.

Connections between peers are established via [transports](#transport), which can be thought of as "connection factories". For example, the TCP transport allows you to create connections that use TCP/IP as their underlying substrate.

libp2p连接是允许对等体读取和写入数据的通信通道。

通过传输建立对等体之间的连接，可以将其视为“连接工厂”。 例如，TCP传输允许您创建使用TCP / IP作为其底层基板的连接。

### Dial

The process of opening a libp2p connection to another peer is known as "dialing", and accepting connections is known as ["listening"](#listen). Together, an implementation of dialing and listening forms a [transport](#transport).

打开与另一个对等体的libp2p连接的过程称为“拨号”，接受连接称为“监听”。 拨号和收听的实现共同构成了一种传输方式。


### Listen

The process of accepting incoming libp2p connections is known as "listening", and it allows other peers to ["dial"](#dial) up and open network connections to your peer.

接受传入的libp2p连接的过程称为“侦听”，它允许其他对等方“拨号”并打开与对等方的网络连接。

### mDNS

[Multicast DNS](https://en.wikipedia.org/wiki/Multicast_DNS) is a protocol for service discovery on a local network. One of libp2p's [peer routing](#peer-routing) implementations leverages mDNS to discover local peers quickly and efficiently.

多播DNS是用于本地网络上的服务发现的协议。 libp2p的一个对等路由实现利用mDNS快速有效地发现本地对等体。



### multiaddr

A `multiaddress` (often abbreviated `multiaddr`), is a convention for encoding multiple layers of addressing information into a single "future-proof" path structure.

For example: `/ip4/127.0.0.1/udp/1234` encodes two protocols along with their essential addressing information. The `/ip4/127.0.0.1` informs us that we want the `127.0.0.1` loopback address of the IPv4 protocol, and `/udp/1234` tells us we want to send UDP packets to port `1234`.

Multiaddresses can be composed to describe multiple "layers" of addresses.

For more detail, see [Concepts > Addressing](/concepts/addressing/), or the [multiaddr spec](https://github.com/multiformats/multiaddr), which has links to many implementations.

### Multiaddress

See [multiaddr](#multiaddr)


### Multihash

[Multihash](https://github.com/multiformats/multihash) is a convention for representing the output of many different [cryptographic hash functions](https://en.wikipedia.org/wiki/Cryptographic_hash_function) in a compact, deterministic encoding that is accommodating of future changes.

Hashes are central to many systems (git, for example), yet many systems store only the hash output itself, since the choice of hash function is an implicit design parameter of the system. This has the unfortunate effect of making it quite difficult to ever change your mind about what kind of hash function your system uses!

A multihash encodes the type of hash function used to produce the output, as well as the length of the output in bytes. This is added as a two-byte header to the original hash output, and in return for those two bytes, the header allows current and future systems to easily identify and validate many hash functions by leveraging common libraries. As new functions are added, you can much more easily extend your application or protocol to support them, since the old and new hash outputs will be easily distinguishable from one another.

The most prominent use of multihashes in libp2p is in the [PeerId](#peerid), which contains a hash of a peer's public key. However, systems built with libp2p, most notably [IPFS](https://ipfs.io), use multihashes for other purposes. In the IPFS case, multihashes are used both to identify content and other peers, since IPFS uses libp2p and shares the same `PeerId` conventions.

In IPFS, multihashes are a key component of the [CID, or content identifier](https://docs.ipfs.io/guides/concepts/cid/), and the "v0" version of CID is a "raw" multihash of a piece of content. A "modern" CID combines a multihash of some content with some compact contextualizing metadata, allowing content-addressed systems like IPFS to create more meaningful links between hash-addressed data. For more on the subject of hash-linked data structures in p2p systems, see [IPLD](https://ipld.io).

Multihashes are often represented as [base58-encoded](https://en.wikipedia.org/wiki/Base58) strings, for example, `QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N`. The first two characters `Qm` are the multihash header for the SHA-256 hash algorithm with a length of 256 bits, and are common to all base58-encoded multihashes using SHA-256.

Multihash是一种用于以紧凑的确定性编码表示许多不同加密散列函数的输出的约定，该编码适应未来的变化。

散列是许多系统（例如git）的核心，但许多系统只存储散列输出本身，因为散列函数的选择是系统的隐式设计参数。这有一个令人遗憾的结果，就是很难改变你对系统使用什么样的哈希函数的想法！

multihash编码用于产生输出的散列函数的类型，以及以字节为单位的输出长度。这被添加为原始哈希输出的双字节头，并且作为对这两个字节的回报，头允许当前和未来系统通过利用公共库来容易地识别和验证许多哈希函数。随着新功能的添加，您可以更轻松地扩展您的应用程序或协议以支持它们，因为旧的和新的哈希输出将很容易彼此区分。

libp2p中多重支持的最突出用途是在PeerId中，它包含对等方公钥的哈希值。但是，使用libp2p构建的系统（最值得注意的是IPFS）将多重用途用于其他目的。在IPFS情况下，多重用途用于标识内容和其他对等，因为IPFS使用libp2p并共享相同的PeerId约定。

在IPFS中，多重散列是CID或内容标识符的关键组件，CID的“v0”版本是一段内容的“原始”多重分片。 “现代”CID将一些内容的多个部分与一些紧凑的上下文元数据相结合，允许像IPFS这样的内容寻址系统在散列寻址数据之间创建更有意义的链接。有关p2p系统中与哈希链接数据结构主题的更多信息，请参阅IPLD。

多重散列通常表示为base58编码的字符串，例如，QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N。前两个字符Qm是SHA-256散列算法的多字头，长度为256位，并且对于使用SHA-256的所有base58编码的多重散列是通用的。


### Multiplexing 多路复用

Multiplexing (or "muxing"), refers to the process of combining multiple streams of communication over a single logical "medium". For example, we can maintain multiple independent data streams over a single TCP network connection, which is itself of course being multiplexed over a single physical connection (ethernet, wifi, etc).

Multiplexing allows peers to offer many [protocols](#protocol) over a single connection, which reduces network overhead and makes [NAT traversal](#nat-traversal) more efficient and effective.

libp2p supports several implementations of stream multiplexing. The [mplex specification](https://github.com/libp2p/specs/tree/master/mplex) defines a simple protocol with implementations in several languages. Other supported multiplexing protocols include [yamux](https://github.com/hashicorp/yamux) and [spdy](https://www.chromium.org/spdy/spdy-whitepaper).

See [Stream Muxer Implementations](https://libp2p.io/implementations/#stream-muxers) for status of multiplexing across libp2p language implementations.


多路复用（或“多路复用”）是指在单个逻辑“介质”上组合多个通信流的过程。例如，我们可以通过单个TCP网络连接维护多个独立的数据流，这本身当然是通过单个物理连接（以太网，wifi等）进行多路复用。

多路复用允许对等体通过单个连接提供许多协议，从而减少网络开销并使NAT遍历更加高效和有效。

libp2p支持流多路复用的几种实现。 mplex规范定义了一种简单的协议，其中包含多种语言的实现。其他支持的多路复用协议包括yamux和spdy。

有关跨libp2p语言实现的多路复用状态，请参阅流Muxer实现。

### multistream 混合数据流

[multistream](https://github.com/multiformats/multistream) is a lightweight convention for "tagging" streams of binary data with a short header that identifies the content of the stream.

libp2p uses multistream to identify the [protocols](#protocol) used for communication between peers, and a related project [multistream-select](https://github.com/multiformats/multistream-select) is used for [protocol negotiation](#protocol-negotiation).

multistream是一种轻量级约定，用于“标记”二进制数据流，并使用标识流内容的短标头。

libp2p使用多流来识别用于对等体之间通信的协议，并且相关项目多流 - 选择用于协议协商。



### NAT

[Network address translation](https://en.wikipedia.org/wiki/Network_address_translation) in general is the mapping of addresses from one address space to another, as often happens at the boundary of private networks with the global internet. It is especially essential in IPv4 networks (which are still the vast majority), as the address space of IPv4 is quite limited. Using NAT, a local, private network can have a vast range of addresses within the internal network, while only consuming one public IP address from the global pool.

An unfortunate effect of NAT in practice is that it's much easier to make outgoing connections from the private network to the public one than it is to call from outside in. This is because machines listening for connections on the internal network need to explicitly tell the router in charge of NAT that it should forward traffic for a given port (the [multiplexing](#multiplexing) abstraction for the OS networking layer) to the listening machine.

This is less of an issue in a client / server model, because outgoing connections to the server give the router enough information to route the response back to the client where it needs to go.

In the peer-to-peer model, accepting connections from other peers is often just as important as initiating them, which means that we often need our peers to be publicly reachable from the global internet. There are many viable approaches to [NAT Traversal](#nat-traversal), several of which are implemented in libp2p.


网络地址转换通常是地址从一个地址空间到另一个地址空间的映射，通常发生在私有网络与全球互联网的边界。在IPv4网络（仍然是绝大多数）中尤为重要，因为IPv4的地址空间非常有限。使用NAT，本地专用网络可以在内部网络中拥有大量地址，而只消耗全局池中的一个公共IP地址。

NAT在实践中的一个不幸影响是，从私有网络到公共网络的传输连接比从外部调用更容易。这是因为侦听内部网络连接的机器需要明确地告诉路由器负责NAT，它应该将给定端口的流量（OS网络层的多路复用抽象）转发到监听机器。

这在客户端/服务器模型中不是问题，因为到服务器的传出连接为路由器提供了足够的信息来将响应路由回到需要它的客户端。

在点对点模型中，接受来自其他对等体的连接通常与启动它们一样重要，这意味着我们经常需要我们的对等体可以从全球互联网公开访问。 NAT Traversal有许多可行的方法，其中一些方法是在libp2p中实现的。


### NAT Traversal NAT 穿透
<!-- TODO(yusef): much of this can be moved to the NAT concept doc and this definition can be trimmed -->


NAT traversal refers to the process of establishing connections with other machines across a [NAT](#nat) boundary. When crossing the boundary between IP networks (e.g. from a local network to the global internet), a [Network Address Translation](#nat) process occurs which maps addresses from one space to another.

For example, my home network has an internal range of IP addresses (10.0.1.x), which is part of a range of addresses that are reserved for private networks. If I start a program on my computer that listens for connections on its internal address, a user from the public internet has no way of reaching me, even if they know my public IP address. This is because I haven't made my router aware of my program yet. When a connection comes in from the internet to my public IP address, the router needs to figure out which internal IP to route the request to, and to which port.

There are many ways to inform one's router about services you want to expose. For consumer routers, there's likely an admin interface that can setup mappings for any range of TCP or UDP ports. In many cases, routers will allow automatic registration of ports using a protocol called [upnp](https://en.wikipedia.org/wiki/Universal_Plug_and_Play), which libp2p supports. If enabled, libp2p will try to register your service with the router for automatic NAT traversal.

In some cases, automatic NAT traversal is impossible, often because multiple layers of NAT are involved. In such cases, we still want to be able to communicate, and we especially want to be reachable and allow other peers to [dial in](#dial) and use our services. This is the one of the motivations for [Circuit Relay](#circuit-relay), which is a protocol involving a "relay" peer that is publicly reachable and can route traffic on behalf of others. Once a relay circuit is established, a peer behind an especially intractable NAT can advertise the relay circuit's [multiaddr](#multiaddr), and the relay will accept incoming connections on our behalf and send us traffic via the relay.

NAT遍历是指跨越NAT边界与其他计算机建立连接的过程。当跨越IP网络之间的边界（例如，从本地网络到全球互联网）时，发生网络地址转换过程，其将地址从一个空间映射到另一个空间。

例如，我的家庭网络具有内部IP地址范围（10.0.1.x），这是为专用网络保留的一系列地址的一部分。如果我在计算机上启动一个程序来监听其内部地址上的连接，那么来自公共互联网的用户无法联系到我，即使他们知道我的公共IP地址。这是因为我还没有让我的路由器知道我的程序。当连接从Internet到我的公共IP地址时，路由器需要确定将请求路由到哪个内部IP以及哪个端口。

有很多方法可以告知路由器有关您要公开的服务的信息。对于消费者路由器，可能有一个管理界面可以为任何范围的TCP或UDP端口设置映射。在许多情况下，路由器将允许使用libp2p支持的名为upnp的协议自动注册端口。如果启用，libp2p将尝试向路由器注册您的服务以进行自动NAT遍历。

在某些情况下，自动NAT遍历是不可能的，通常是因为涉及多层NAT。在这种情况下，我们仍然希望能够进行通信，我们特别希望能够访问并允许其他对等方拨入并使用我们的服务。这是电路中继的动机之一，电路中继是一种涉及“中继”对等体的协议，可以公开访问，并可以代表其他人路由流量。一旦建立了中继电路，特别难以处理的NAT后面的对等体就可以通告中继电路的多路径，并且中继将代表我们接收传入连接并通过中继向我们发送流量。


### Node

The word "node" is quite overloaded in general programming contexts, and this is especially the case in peer-to-peer networking circles.

One common usage is when "node" refers to a single instance of a peer-to-peer software system, running at some time and place in the universe. For example, `I'm running an orbit-db node in AWS. I think it's on version 3.2.0`. In this usage, "node" refers to the whole software program (the `daemon` in unix-speak) which participates in the network. In this documentation, we'll often use ["peer"](#peer) for this purpose instead, and the two terms are often used interchangeably in various p2p software discussions.

Another quite different meaning is the [node.js](https://nodejs.org) javascript runtime environment, which is one of the supported runtimes for the [javscript libp2p implementation][js-docs-home]. In general it should be pretty clear from context when "node" is referring to node.js.

Many members of our community are excited about graphs in many contexts, so the graph terminology of "nodes and edges" is often used when discussing various subjects. Some common contexts for graph-related discussions:

- When discussing the [topology](#topology) or structure of a peer-to-peer network, "node" is often used in the context of a graph of connected peers. Efficient construction and traversal of this graph is key to effective [peer routing](#peer-routing).

- When discussing data structures, "node" is often useful for referring to key elements of the structure. For example, a linked list consists of many "nodes" containing both a value and a link (or, in graph terms, an "edge") connecting it to the next node. Since many useful and interesting data structures can be described as graphs, much of the terminology of graph theory applies when discussing their properties. In particular, IPFS is naturally well-suited to storing and manipulating data structures which form a [Directed Acyclic Graph, or DAG](https://en.wikipedia.org/wiki/Directed_acyclic_graph).

- An especially interesting data structure for many in our community is [IPLD](https://ipld.io), or Interplanetary Linked Data. Similar to libp2p, IPLD grew out of the real-world needs of IPFS, but is broadly useful and interesting in many contexts outside of IPFS. IPLD discussions often involve "nodes" of all the types discussed here.

在一般编程上下文中，“节点”一词非常重载，在对等网络圈中尤其如此。

一种常见的用法是“节点”是指在宇宙中的某个时间和地点运行的对等软件系统的单个实例。例如，我正在AWS中运行orbit-db节点。我认为它是在3.2.0版本上。在这种用法中，“节点”指的是参与网络的整个软件程序（unix-speak中的守护程序）。在本文档中，我们通常会为此目的使用“peer”，这两个术语在各种p2p软件讨论中经常互换使用。

另一个完全不同的含义是node.js javascript运行时环境，它是javscript libp2p实现支持的运行时之一。一般来说，当“node”指的是node.js时，应该从上下文中非常清楚。

我们社区的许多成员对许多情境中的图表感到兴奋，因此在讨论各种主题时经常使用“节点和边缘”的图形术语。与图形相关的讨论的一些常见背景：

在讨论对等网络的拓扑或结构时，“节点”通常用于连接对等体的图的上下文中。有效构建和遍历此图是有效对等路由的关键。

在讨论数据结构时，“节点”通常用于引用结构的关键元素。例如，链表由许多“节点”组成，这些“节点”包含值和链接（或者，在图形方面，“边缘”），将其连接到下一个节点。由于许多有用且有趣的数据结构可以描述为图形，因此图论的大部分术语在讨论它们的属性时都适用。特别是，IPFS自然非常适合存储和操纵形成有向无环图或DAG的数据结构。

我们社区中许多人特别感兴趣的数据结构是IPLD或行星际关联数据。与libp2p类似，IPLD源于IPFS的实际需求，但在IPFS之外的许多环境中广泛使用和有趣。 IPLD讨论通常涉及此处讨论的所有类型的“节点”。

### Overlay  叠加层

An "overlay network" or just "overlay" refers to the logical structure of a peer-to-peer network, which is "overlaid" on top of the underlying [transport mechanisms](#transport) used for lower-level network communication.

Peer-to-peer systems are generally composed of one or more overlay networks, which determine how peers are identified and located, how messages are propagated throughout the system, and other key properties.

Examples of overlay networks used in libp2p are the [DHT](#dht) implementation, which is based on [Kademlia](https://en.wikipedia.org/wiki/Kademlia), and the networks formed by participants in the various [pubsub](#pubsub) implementations.

“覆盖网络”或仅“覆盖”是指对等网络的逻辑结构，其在用于较低级别网络通信的底层传输机制之上“覆盖”。

对等系统通常由一个或多个覆盖网络组成，这些覆盖网络确定如何识别和定位对等体，如何在整个系统中传播消息，以及其他关键属性。

libp2p中使用的覆盖网络的示例是基于Kademlia的DHT实现，以及由各种pubsub实现中的参与者形成的网络。


### Peer

A single participant in a peer-to-peer network. While a given peer may support many [protocols](#protocol), it has a single [PeerId](#peerid) which it uses to identify itself to other peers. Often used synonymously with [node](#node).

对等网络中的单个参与者。虽然给定的对等体可以支持许多协议，但它具有单个PeerId，它用于向其他对等体标识自己。通常与节点同义使用。

### PeerId

A unique, verifiable identifier for a [peer](#peer) that is impossible for another peer to forge or impersonate without trivial detection. In libp2p, peers are identified by their `PeerId`, which is both globally unique and allows other peers to obtain the peer's [cryptographic public key](https://en.wikipedia.org/wiki/Public-key_cryptography).

The most common form of `PeerId` is a [multihash](#multihash) of a peer's public key, which can be used to fetch the entire public key from the [DHT](#dht) for encryption or signature verification. There is also experimental support for embedding or "inlining" small public keys directly into the `PeerId`, however, this is an area of [ongoing discussion](https://github.com/libp2p/specs/issues/138) and should be treated with caution in production systems until finalized.

An important property of cryptographic peer identities is that they are decoupled from [transport](#transport), allowing peers to verify the identity of other peers regardless of what underlying network they might use to communicate. This also gives them a much longer "shelf life" than location-based identifiers (for example, IP addresses), since identities remain stable across address changes.

对等体的唯一可验证标识符，对于其他对等体而言，如果不进行简单检测，则无法进行伪造或冒充。在libp2p中，对等体由其PeerId标识，PeerId既是全局唯一的又允许其他对等体获得对等体的加密公钥。

最常见的PeerId形式是对等公钥的多哈，可用于从DHT获取整个公钥以进行加密或签名验证。还有实验性支持将小公钥直接嵌入或“内联”到PeerId中，然而，这是一个正在进行讨论的领域，在生产系统中应该谨慎对待，直到最终确定。

加密对等身份的一个重要特性是它们与传输分离，允许对等体验证其他对等体的身份，而不管它们可能用于通信的底层网络。这也使它们比基于位置的标识符（例如，IP地址）具有更长的“保质期”，因为身份在地址变化中保持稳定。

### Peer store

A data structure that stores [PeerIds](#peerid) for known peers, along with known [multiaddresses](#multiaddr) that can be used to communicate with them.

为已知对等方存储PeerIds的数据结构，以及可用于与它们通信的已知多地址。


### Peer routing

Peer routing is the process of discovering the network "route" or address for a
peer in the network, given the peer's [id](#peerid).

对等路由是在给定对等方ID的情况下发现网络中对等方的网络“路由”或地址的过程。

It may also include "ambient" discovery of local peers, for example via
[multicast DNS](#mdns).

The primary peer routing mechanism in libp2p uses a
[distributed hash table](#dht) to locate peers, taking advantage of the
Kademlia routing algorithm to efficiently locate peers.

它还可以包括本地对等体的“环境”发现，例如通过多播DNS。

libp2p中的主对等路由机制使用分布式哈希表来定位对等体，利用Kademlia路由算法有效地定位对等体。



### Peer-to-peer (p2p)

A peer-to-peer (p2p) network is one in which the participants (referred to as [peers](#peer) or [nodes](#node)) communicate with one another directly, on more or less "equal footing". This does not necessarily mean that all peers are identical; some may have different roles in the overall network. However, one of the defining characteristics of a peer-to-peer network is that they do not require a privileged set of "servers" which behave completely differently from their "clients", as is the case in the the predominant [client / server model](#client-server).

对等（p2p）网络是参与者（称为对等体或节点）在或多或少“平等”的情况下直接相互通信的网络。这并不一定意味着所有同伴都是相同的;有些人可能在整个网络中扮演不同的角色。然而，对等网络的一个定义特征是它们不需要特权的“服务器”集，其行为与其“客户端”完全不同，如主要客户端/服务器模型中的情况。


### Pubsub

In general, refers to "publish / subscribe", a communication pattern in which participants "subscribe" for updates "published" by other participants, often on a named "topic".

libp2p defines a [pubsub spec](https://github.com/libp2p/specs/blob/master/pubsub/README.md), with links to several implementations in supported languages. Pubsub is an area of ongoing research and development, with multiple implementations optimized for different use cases and environments.

通常，指的是“发布/订阅”，一种通信模式，其中参与者“订阅”由其他参与者“发布”的更新，通常在命名的“主题”上。

libp2p定义了一个pubsub规范，其中包含支持语言中多个实现的链接。 Pubsub是一个正在进行研究和开发的领域，其中多个实现针对不同的用例和环境进行了优化。

### Protocol

In general, a set of rules and data structures used for network communication.

libp2p is comprised of many protocols and makes use of many others provided by
the operating system or runtime environment.

Most core libp2p functionality is defined in terms of protocols, and libp2p
protocols are identified using [multistream](#multistream) headers.

通常，用于网络通信的一组规则和数据结构。

libp2p由许多协议组成，并使用操作系统或运行时环境提供的许多其他协议。

大多数核心libp2p功能是根据协议定义的，而libp2p协议是使用多流头标识的。

### Protocol Negotiation

The process of reaching agreement on what protocol to use for a given stream
of communication.

In libp2p, protocols are identified using a convention called
[multistream](#multistream), which adds a small header to the beginning of
a stream containing a unique name, including a version identifier.

When two peers first connect, they exchange a [handshake](#handshake) to
agree upon what protocols to use.

The implementation of the libp2p handshake is called
[multistream-select](https://github.com/multiformats/multistream-select).

For details, see the [protocol negotiation article](/concepts/protocols/#protocol-negotiation).

就给定通信流使用什么协议达成一致的过程。

在libp2p中，使用称为多流的约定来标识协议，该约定将小头添加到包含唯一名称（包括版本标识符）的流的开头。

当两个对等体首次连接时，他们交换握手以商定要使用的协议。

libp2p握手的实现称为multistream-select。

有关详细信息，请参阅协议协商文章。

### Stream

TODO: Distinguish between the various types of "stream". Could refer to

TODO：区分各种类型的“流”。 可以参考

- raw tcp connection

原始tcp连接

- one component of a multistream connection

多流连接的一个组成部分


- node.js streams / pull-streams

### Swarm

Can refer to a collection of interconnected peers.

In the libp2p codebase, "swarm" may refer to a module that allows a peer to
interact with its peers, although this component was later renamed ["switch"](#switch).

See [the discussion about the name change](https://github.com/libp2p/js-libp2p-switch/issues/40)
for context.

可以引用互连对等体的集合。

在libp2p代码库中，“swarm”可以指允许对等体与其对等体交互的模块，尽管该组件稍后被重命名为“switch”。

请参阅有关上下文的名称更改的讨论。


### Switch

A libp2p component responsible for composing multiple [translports](#transport)
into a single interface, allowing application code to [dial](#dial) peers
without having to specify what transport to use.

In addition to managing transports, the switch also coordinates the
"connection upgrade" process, which promotes a "raw" connection from
the transport layer into one that supports
[protocol negotiation](/concepts/protocols/#protocol-negotiation),
[stream multiplexing](#multiplexing), and
[secure communications](/concepts/secure-comms).

Sometimes called ["swarm"](#swarm) for historical reasons.

libp2p组件，负责将多个转换组合成单个接口，允许应用程序代码拨打对等体，而无需指定要使用的传输。

除了管理传输之外，交换机还协调“连接升级”过程，该过程促进从传输层到支持协议协商，流复用和安全通信的“原始”连接。

由于历史原因，有时被称为“群体”。

有人用UDP，有人用TCP，有人用WS，把他们封装成统一的接口

### Topology 拓扑

In a peer-to-peer context, usually refers to the shape or structure of the [overlay network](#overlay) formed by peers as they communicate with each other.

拓扑
在对等上下文中，通常是指由对等体彼此通信时形成的覆盖网络的形状或结构。

### Transport

In libp2p, `transport` refers to the technology that lets us move bits from one machine to another. This may be a TCP network provided by the operating system, a websocket connection in a browser, or anything else capable of implementing the [transport interface](https://github.com/libp2p/interface-transport).

Note that in some environments such as javascript running in the browser, not all transports will be available. In such cases, it may be possible to establish a [Circuit Relay](#circuit-relay) with the help of a peer that can support many common transports. Such a relay can act as a "transport adapter" of sorts, allowing peers that can't communicate with each other directly to interact. For example, a peer in the browser that can only make websocket connections could relay through a peer able to make TCP connections, which would enable communication with a wider variety of peers.

在libp2p中，transport是指允许我们将位从一台机器移动到另一台机器的技术。 这可以是由操作系统提供的TCP网络，浏览器中的websocket连接，或者能够实现传输接口的任何其他东西。

请注意，在某些环境中，例如在浏览器中运行的javascript，并非所有传输都可用。 在这种情况下，可以在可以支持许多常见传输的对等体的帮助下建立电路中继。 这样的中继可以充当各种“传输适配器”，允许不能彼此通信的对等体直接进行交互。 例如，浏览器中只能进行websocket连接的对等体可以通过能够进行TCP连接的对等体进行中继，这将实现与更多种类的对等体的通信。


[js-docs-home]: /reference/
