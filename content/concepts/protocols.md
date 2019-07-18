---
title: Protocols
weight: 3
---

There are protocols everywhere you look when you're writing network applications, and libp2p is
especially thick with them. 

The kind of protocols this article is concerned with are the ones built with libp2p itself,
using the core libp2p abstractions like [transport](/concepts/transport), [peer identity](/concepts/peer-id/), [addressing](/concepts/addressing/), and so on.

Throughout this article, we'll call this kind of protocol that is built with libp2p
a **libp2p protocol**, but you may also see them referred to as "wire protocols" or "application protocols". 

These are the protocols that define your application and provide its core functionality.

This article will walk through some of the key [defining features of a libp2p protocol](#what-is-a-libp2p-protocol), give an overview of the [protocol negotiation process](#protocol-negotiation), and outline some of the [core libp2p protocols](#core-libp2p-protocols) that are included with libp2p and provide key functionality.


在编写网络应用程序时，每个地方都有协议，而libp2p特别厚。

本文所涉及的协议类型是使用libp2p本身构建的协议，使用核心libp2p抽象，如传输，对等身份，寻址等。

在本文中，我们将这种使用libp2p构建的协议称为libp2p协议，但您也可以将它们称为“有线协议”或“应用程序协议”。

这些是定义应用程序并提供其核心功能的协议。

本文将介绍libp2p协议的一些关键定义功能，概述协议协商过程，并概述libp2p中包含的一些核心libp2p协议并提供关键功能。



## What is a libp2p protocol?

A libp2p protocol has these key features:

#### Protocol Ids

libp2p protocols have unique string identifiers, which are used in the [protocol negotiation](#protocol-negotiation) process when connections are first opened.

By convention, protocol ids have a path-like structure, with a version number as the final component:

```
/my-app/amazing-protocol/1.0.1
```

Breaking changes to your protocol's wire format or semantics should result in a new version
number. See the [protocol negotiation section](#protocol-neotiation) for more information about
how version selection works during the dialing and listening process.

libp2p协议具有唯一的字符串标识符，这些标识符在首次打开连接时用于协议协商过程。

按照惯例，协议ID具有类似路径的结构，版本号作为最终组件：

/my-app/amazing-protocol/1.0.1
断开对协议的有线格式或语义的更改应该会产生新的版本号。有关在拨号和侦听过程中版本选择如何工作的详细信息，请参阅协议协商部分。



{{% notice "info" %}}

While libp2p will technically accept any string as a valid protocol id,
using the recommended path structure with a version component is both
developer-friendly and enables [easier matching by version](#match-using-semver).

虽然libp2p在技术上会接受任何字符串作为有效的协议ID，但使用推荐的路径结构和版本组件是开发人员友好的，并且使版本更容易匹配。

{{% /notice %}}

#### Handler functions

To accept connections, a libp2p application will register handler functions for protocols using their protocol id with the
[switch][definition_switch] (aka "swarm"), or a higher level interface such as [go's Host interface](https://github.com/libp2p/go-libp2p-host/blob/master/host.go).

The handler function will be invoked when an incoming stream is tagged with the registered protocol id.
If you register your handler with a [match function](#using-a-match-function), you can choose whether
to accept non-exact string matches for protocol ids, for example, to match on [semantic major versions](#match-using-semver).

为了接受连接，libp2p应用程序将使用协议ID与交换机（也称为“swarm”）或更高级别的接口（如go的Host接口）来注册协议的处理函数。

当使用注册的协议ID标记传入流时，将调用处理函数。如果使用匹配函数注册处理程序，则可以选择是否接受协议ID的非精确字符串匹配，例如，匹配语义主要版本。



#### Binary streams

The "medium" over which a libp2p protocol transpires is a bi-directional binary stream with the following
properties:

libp2p协议发现的“媒介”是具有以下属性的双向二进制流：

- Bidirectional, reliable delivery of binary data

双向，可靠地传递二进制数据

  - Each side can read and write from the stream at any time
  
  每一方都可以随时从流中读写
  
  - Data is read in the same order as it was written
  
  数据的读取顺序与写入的顺序相同
  
  - Can be "half-closed", that is, closed for writing and open for reading, or closed for reading and open for writing 
  
  可以是“半封闭”，即关闭写作和开放阅读，或关闭阅读和开放写作
  
- Supports backpressure
  - Readers can't be flooded by eager writers <!-- TODO(yusef) elaborate: how is backpressure implemented? is it transport-depdendent? -->
  
  读者不能被热心的作家所淹没

  
Behind the scenes, libp2p will also ensure that the stream is [secure](/concepts/secure-comms/) and efficiently
[multiplexed](/concepts/stream-multiplexing/). This is transparent to the protocol handler, which reads and writes
unencrypted binary data over the stream.

在幕后，libp2p还将确保流安全且高效地进行多路复用。这对协议处理程序是透明的，协议处理程序通过流读取和写入未加密的二进制数据。

The format of the binary data and the mechanics of what to send when and by whom are all up to the protocol to determine. For insipiration, some [common patterns](#common-patterns) that are used in libp2p's internal protocols are outlined below.

二进制数据的格式以及何时和由谁发送的内容的机制都取决于协议来确定。对于insipiration，libp2p的内部协议中使用的一些常见模式概述如下。



## Protocol Negotiation 协议协商

When dialing out to initiate a new stream, libp2p will send the protocol id of the protocol you want to use.
The listening peer on the other end will check the incoming protocol id against the registered protcol handlers.

If the listening peer does not support the requested protocol, it will end the stream, and the dialing peer can
try again with a different protocol, or possibly a fallback version of the initially requested protocol.

If the protocol is supported, the listening peer will echo back the protocol id as a signal that future data 
sent over the stream will
use the agreed protocol semantics.

This process of reaching agreement about what protocol to use for a given stream or connection is called
**protocol negotiation**. 

当拨出以启动新流时，libp2p将发送您要使用的协议的协议ID。 另一端的侦听对等体将根据注册的protcol处理程序检查传入的协议ID。

如果侦听对等体不支持所请求的协议，则它将结束流，并且拨号对等体可以使用不同的协议再次尝试，或者可能是最初请求的协议的回退版本。

如果支持该协议，则侦听对等体将回显协议ID，作为通过流发送的未来数据将使用约定的协议语义的信号。

达成关于给定流或连接使用什么协议的协议的过程称为协议协商。


### Matching protocol ids and versions

When you register a protocol handler, there are two methods you can use.

The first takes two arguments: a protocol id, and a handler function. If an incoming stream request sends an exact
match for the protocol id, the handler function will be invoked with the new stream as an argument.

注册协议处理程序时，可以使用两种方法。

第一个接受两个参数：协议ID和处理函数。如果传入流请求发送协议ID的完全匹配，则将使用新流作为参数调用处理函数。

#### Using a match function

The second kind of protocol registration takes three arguments: the protocol id, a protocol match function, and the handler function.

When a stream request comes in whose protocol id doesn't have any exact matches, the protocol id will be passed through
all of the registered match functions. If any returns `true`, the associated handler function will be invoked.

This gives you a lot of flexibility to do your own "fuzzy matching" and define whatever rules for protocol matching
make sense for your application.

第二种协议注册有三个参数：协议ID，协议匹配函数和处理函数。

当流请求进入其协议ID没有任何完全匹配时，协议ID将通过所有已注册的匹配函数。如果any返回true，则将调用关联的处理函数。

这使您可以灵活地进行自己的“模糊匹配”，并为您的应用程序定义协议匹配的任何规则。

#### Match using semver

If you'd like to concurrently support a range of numbered versions, you may want to use semantic versioning (aka [semver](https://semver.org)).

In go-libp2p, a helper function called [`MultistreamSemverMatcher`](https://github.com/libp2p/go-libp2p-host/blob/master/match.go) can be used
as a protocol match function to see if an incoming request can be satisfied by the registered protocol version.

js-libp2p provides a [similar match function](https://github.com/multiformats/js-multistream-select/blob/master/src/listener/match-semver.js)
as part of [js-multistream-select](https://github.com/multiformats/js-multistream-select/)

如果您想同时支持一系列编号版本，您可能希望使用语义版本控制（也称为semver）。

在go-libp2p中，一个名为MultistreamSemverMatcher的辅助函数可以用作协议匹配函数，以查看注册协议版本是否可以满足传入请求。

js-libp2p提供了类似的匹配函数作为js-multistream-select的一部分

### Dialing a specific protocol

When dialing a remote peer to open a new stream, the initiating peer sends the protocol id that they'd like to use. The remote peer will use
the matching logic described above to accept or reject the protocol. If the protocol is rejected, the dialing peer can try again.

When dialing, you can optionally provide a list of protocol ids instead of a single id. When you provide multiple protocol ids, they will
each be tried in succession, and the first successful match will be used if at least one of the protocols is supported by the remote peer.
This can be useful if you support a range of protocol versions, since you can propose the most recent version and fallback to older versions
if the remote hasn't adopted the latest version yet.

当拨打远程对等体以打开新流时，发起对等体发送他们想要使用的协议ID。 远程对等体将使用上述匹配逻辑来接受或拒绝该协议。 如果协议被拒绝，拨号对等体可以再次尝试。

拨号时，您可以选择提供协议ID列表而不是单个ID。 当您提供多个协议ID时，它们将连续尝试，并且如果远程对等方支持至少一个协议，则将使用第一个成功匹配。 如果您支持一系列协议版本，这将非常有用，因为如果远程尚未采用最新版本，您可以提出最新版本并回退到旧版本。

## Core libp2p protocols

In addition to the protocols that you write when developing a libp2p application, libp2p itself defines several foundational protocols that are used for core features.

除了在开发libp2p应用程序时编写的协议，libp2p本身还定义了几个用于核心功能的基础协议。

### Common patterns

The protocols described below all use [protocol buffers](https://developers.google.com/protocol-buffers/) (aka protobuf) to define message schemas.

Messages are exchanged over the wire using a very simple convention which prefixes binary
message payloads with an integer that represents the length of the payload in bytes. The 
length is encoded as a [protobuf varint](https://developers.google.com/protocol-buffers/docs/encoding#varints)  (variable-length integer).


常见的模式
下面描述的协议都使用协议缓冲区（aka protobuf）来定义消息模式。

使用非常简单的约定通过线路交换消息，该约定为二进制消息有效负载添加一个整数，该整数表示以字节为单位的有效负载的长度。 长度编码为protobuf varint（可变长度整数）。


### Ping

| **Protocol id**    | spec |               |               | implementations   |
|--------------------|------|---------------|---------------|-------------------|
| `/ipfs/ping/1.0.0` | N/A  | [go][ping_go] | [js][ping_js] | [rust][ping_rust] |


[ping_go]: https://github.com/libp2p/go-libp2p/tree/master/p2p/protocol/ping
[ping_js]: https://github.com/libp2p/js-libp2p-ping
[ping_rust]: https://github.com/libp2p/rust-libp2p/blob/master/protocols/ping/src/lib.rs


The ping protocol is a simple liveness check that peers can use to quickly see if another peer is online.

After the initial protocol negotiation, the dialing peer sends 32 bytes of random binary data. The listening
peer echoes the data back and closes the stream, and the dialing peer will verify the response and measure 
the latency between request and response.

ping协议是一个简单的活跃性检查，对等方可以使用它来快速查看另一个对等方是否在线。

在初始协议协商之后，拨号对等体发送32字节的随机二进制数据。 侦听对等体回送数据并关闭流，拨号对等体将验证响应并测量请求和响应之间的延迟。

### Identify

| **Protocol id**  | spec                           |                   |                   | implementations       |
|------------------|--------------------------------|-------------------|-------------------|-----------------------|
| `/ipfs/id/1.0.0` | [identify spec][spec_identify] | [go][identify_go] | [js][identify_js] | [rust][identify_rust] |

<!-- TODO(yusef): update spec link on PR merge -->
[spec_identify]: https://github.com/libp2p/specs/pull/97/files
[identify_go]: https://github.com/libp2p/go-libp2p/tree/master/p2p/protocol/identify
[identify_js]: https://github.com/libp2p/js-libp2p-identify
[identify_rust]: https://github.com/libp2p/rust-libp2p/tree/master/protocols/identify/src

The `identify` protocol allows peers to exchange information about each other, most notably their public keys
and known network addresses.


The basic identify protocol works by establishing a new stream to a peer using the identify protocol id
shown in the table above.

When the remote peer opens the new stream, they will fill out an [`Identify` protobuf message][identify_proto] containing
information about themselves, such as their public key, which is used to derive their [`PeerId`](/concepts/peer-id/).

Importantly, the `Identify` message includes an `observedAddr` field that contains the [multiaddr][definition_multiaddr] that
the peer observed the request coming in on. This helps peers determine their NAT status, since it allows them to
see what other peers observe as their public address and compare it to their own view of the network.

[identify_proto]: https://github.com/libp2p/go-libp2p/blob/master/p2p/protocol/identify/pb/identify.proto

识别协议允许对等体彼此交换信息，最明显的是它们的公钥和已知的网络地址。

基本标识协议通过使用上表中显示的标识协议ID向对等方建立新流来工作。

当远程对等体打开新流时，它们将填写Identify protobuf消息，其中包含有关其自身的信息，例如用于派生其PeerId的公钥。

重要的是，Identify消息包含一个observedAddr字段，该字段包含对等方观察到请求的multiaddr。 这有助于对等体确定其NAT状态，因为它允许他们查看其他对等端观察到的公共地址，并将其与自己的网络视图进行比较。


#### identify/push

| **Protocol id**       | spec & implementations              |
|-----------------------|-------------------------------------|
| `/ipfs/id/push/1.0.0` | same as [identify above](#identify) |

A slight variation on `identify`, the `identify/push` protocol sends the same `Identify` message, but it does so proactively
instead of in response to a request.

This is useful if a peer starts listening on a new address, establishes a new [relay circuit](/concepts/circuit-relay/), or
learns of its public address from other peers using the standard `identify` protocol. Upon creating or learning of a new address,
the peer can push the new address to all peers it's currently aware of. This keeps everyones routing tables up to date and
makes it more likely that other peers will discover the new address.

身份识别/推送协议略有不同，识别/推送协议会发送相同的识别消息，但它会主动执行，而不是响应请求。

如果对等体开始侦听新地址，建立新的中继电路，或使用标准识别协议从其他对等体获知其公共地址，这将非常有用。 在创建或学习新地址时，对等体可以将新地址推送到它当前知道的所有对等体。 这使每个人的路由表保持最新，并使其他对等体更有可能发现新地址。

### secio 

| **Protocol id** | spec                     |                |                | implementations    |
|-----------------|--------------------------|----------------|----------------|--------------------|
| `/secio/1.0.0`  | [secio spec][spec_secio] | [go][secio_go] | [js][secio_js] | [rust][secio_rust] |

<!-- TODO(yusef): update spec link when PR lands -->
[spec_secio]: https://github.com/libp2p/specs/pull/106
[secio_go]: https://github.com/libp2p/go-libp2p-secio
[secio_js]: https://github.com/libp2p/js-libp2p-secio
[secio_rust]: https://github.com/libp2p/rust-libp2p/tree/master/protocols/secio

`secio` (short for secure input/output) is a protocol for encrypted communication that is similar to TLS 1.2, but without the
Certificate Authority requirements. Because each libp2p peer has a [PeerId](/concepts/peer-id) that's derived from their
public key, the identity of a peer can be validated without needing a Certificate Authority by using their public
key to validate signed messages.

See the [Secure Communication article](/concepts/secure-comms/) for more information.

secio（安全输入/输出的简称）是用于加密通信的协议，类似于TLS 1.2，但没有证书颁发机构要求。 因为每个libp2p对等体具有从其公钥派生的PeerId，所以可以通过使用其公钥来验证签名消息来验证对等体的身份而无需证书颁发机构。

有关更多信息，请参阅安全通信文章。


{{% notice "note" %}}

While secio is the default encryption protocol used by libp2p today, work is progressing on integrating TLS 1.3 into libp2p,
which is expected to become the default once completed. See [the libp2p TLS 1.3 spec](https://github.com/libp2p/specs/tree/master/tls)
for an overview of the design.

虽然secio是libp2p今天使用的默认加密协议，但是在将TLS 1.3集成到libp2p中的工作正在进行中，一旦完成，它将成为默认值。 有关设计的概述，请参阅libp2p TLS 1.3规范。

{{% /notice %}}

### kad-dht

| **Protocol id**   | spec                     |              |              | implementations  |
|-------------------|--------------------------|--------------|--------------|------------------|
| `/ipfs/kad/1.0.0` | [kad-dht spec][spec_kad] | [go][kad_go] | [js][kad_js] | [rust][kad_rust] |

`kad-dht` is a [Distributed Hash Table][wiki_dht] based on the [Kademlia][wiki_kad] routing algorithm, with some modifications.

libp2p uses the DHT as the foundation of its [peer routing](/concepts/peer-routing/) and [content routing](/concepts/content-routing/) functionality.

kad-dht是一个基于Kademlia路由算法的分布式哈希表，有一些修改。

libp2p使用DHT作为其对等路由和内容路由功能的基础。

<!-- TODO(yusef): update spec link when PR lands -->
[spec_kad]: https://github.com/libp2p/specs/pull/108
[kad_go]: https://github.com/libp2p/go-libp2p-kad-dht
[kad_js]: https://github.com/libp2p/js-libp2p-kad-dht
[kad_rust]: https://github.com/libp2p/rust-libp2p/tree/master/protocols/kad

[wiki_dht]: https://en.wikipedia.org/wiki/Distributed_hash_table
[wiki_kad]: https://en.wikipedia.org/wiki/Kademlia



### Circuit Relay

| **Protocol id**               | spec                             |                | implementations |
|-------------------------------|----------------------------------|----------------|-----------------|
| `/libp2p/circuit/relay/0.1.0` | [circuit relay spec][spec_relay] | [go][relay_go] | [js][relay_js]  |

[spec_relay]: https://github.com/libp2p/specs/tree/master/relay
[relay_js]: https://github.com/libp2p/js-libp2p-circuit
[relay_go]: https://github.com/libp2p/go-libp2p-circuit

As described in the [Circuit Relay article](/concepts/circuit-relay/), libp2p provides a protocol
for tunneling traffic through relay peers when two peers are unable to connect to each other
directly. See the article for more information on working with relays, including notes on relay
addresses and how to enable automatic relay connection when behind an intractible NAT.

如“电路中继”一文中所述，libp2p提供了一种协议，用于在两个对等体无法直接相互连接时通过中继对等体进行隧道传输。 有关使用继电器的更多信息，请参阅文章，包括有关中继地址的说明以及如何在无法使用的NAT后启用自动中继连接。

<!-- links -->

[definition_switch]: /reference/glossary/#switch
[definition_multiaddr]: /reference/glossary/#multiaddr

[repo_multistream-select]: https://github.com/multiformats/multistream-select

