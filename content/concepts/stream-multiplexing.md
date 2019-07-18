---
title: Stream Multiplexing
---

Stream Multiplexing (often abbreviated as "stream muxing") allows multiple independent logical streams to all share a common underlying transport medium.

libp2p applications often open many independent streams of communication between peers and may have several concurrent streams open at the same time with a given remote peer. Stream multiplexing allows us to amortize the overhead of establishing new [transport](/concepts/transport/) connections across the lifetime of our interaction with a peer. We also only need to deal with [NAT traversal](/concepts/nat/) once to be able to open as many streams as we need, since they will all share the same underlying transport connection.

Multiplexing is by no means unique to libp2p. Most communication networks involve some kind of multiplexing, as the transport medium is generally scarce and needs to be shared by many participants. For example, the TCP/IP stack multiplexes many TCP streams over an underlying network connection, using unique port numbers to distinguish streams. libp2p's stream multiplexer sits "above" the transport stack and allows many streams to flow over a single TCP port or other raw transport connection.

libp2p provides a common [interface](#interface) for stream multiplexers with several [implementations](#implementations) available. Applications can enable support for multiple multiplexers, which will allow you to fall back to a widely-supported multiplexer if a preferred choice is not supported by a remote peer. 

流多路复用（通常缩写为“流多路复用”）允许多个独立的逻辑流共享共同的底层传输介质。

libp2p应用程序通常在对等体之间打开许多独立的通信流，并且可能与给定的远程对等体同时打开多个并发流。流复用允许我们在与对等体交互的整个生命周期中分摊建立新传输连接的开销。我们还需要处理NAT遍历一次才能打开所需数量的流，因为它们将共享相同的底层传输连接。

多路复用绝不是libp2p独有的。大多数通信网络涉及某种多路复用，因为传输介质通常很少并且需要许多参与者共享。例如，TCP / IP堆栈通过底层网络连接复用许多TCP流，使用唯一的端口号来区分流。 libp2p的流多路复用器位于传输堆栈“上方”，允许许多流在单个TCP端口或其他原始传输连接上流动。

libp2p为流多路复用器提供了一个通用接口，可以使用多种实现。应用程序可以启用对多个多路复用器的支持，如果远程对等方不支持首选，则可以回退到广泛支持的多路复用器。

## Where it fits in the libp2p stack

libp2p's multiplexing happens at the "application layer", meaning it's not provided by the operating system's network stack. However, developers writing libp2p applications rarely need to interact with stream multiplexers directly, except during initial configuration to control which modules are enabled. 

libp2p的多路复用发生在“应用层”，这意味着它不是由操作系统的网络堆栈提供的。但是，编写libp2p应用程序的开发人员很少需要直接与流多路复用器交互，除非在初始配置期间控制启用哪些模块。

### Switch / swarm

libp2p maintains some state about known peers and existing connections in a component known as the switch (or "swarm", depending on the implementation). The switch provides a dialing and listening interface that abstracts the details of which stream multiplexer is used for a given connection.

When configuring libp2p, applications enable stream muxing modules, which the switch will use when dialing peers and listening for connections. If the remote peers support any of the same stream muxing implementations, the switch will select and use it when establishing the connection. If you dial a peer that the switch already has an open connection to, the new stream will automatically be multiplexed over the existing connection.

Reaching agreement on which stream multiplexer to use happens early in the connection establishment process. Peers use [protocol negotiation](/concepts/protocols/#protocol-negotiation) to agree on a commonly supported multiplexer, which upgrades a "raw" transport connection into a muxed connection capable of opening new streams.

libp2p在称为switch（或“swarm”，取决于实现）的组件中维护有关已知对等体和现有连接的某些状态。该交换机提供拨号和监听接口，该接口抽象出用于给定连接的流复用器的细节。

在配置libp2p时，应用程序启用流复用模块，交换机在拨打对等端和侦听连接时将使用这些模块。如果远程对等体支持任何相同的流复用实现，则交换机将在建立连接时选择并使用它。如果您拨打对等方已确定交换机已打开连接，则新流将自动在现有连接上进行多路复用。

在连接建立过程的早期达成关于使用哪个流多路复用器的协议。对等方使用协议协商来商定通常支持的多路复用器，它将“原始”传输连接升级为能够打开新流的多路复用连接。

## Interface and Implementations

### Interface
The [stream multiplexing interface][interface-stream-muxing] defines how a stream muxing module can be applied to a connection and what operations are supported by a multiplexed connection.

流复用接口定义了如何将流复用模块应用于连接以及多路复用连接支持哪些操作。


### Implementations

There are several stream multiplexing modules available in libp2p. Please note that not all stream muxers are supported by every libp2p language implementation.

libp2p中有几个流复用模块。请注意，并非每个libp2p语言实现都支持所有流复用器。

#### mplex

mplex is a protocol developed for libp2p. The [spec](https://github.com/libp2p/specs/tree/master/mplex) defines a simple protocol for multiplexing that is widely supported across libp2p language implementations:

mplex是为libp2p开发的协议。规范定义了一个简单的多路复用协议，它在libp2p语言实现中得到广泛支持：

- Go: [go-mplex](https://github.com/libp2p/go-mplex)
- Javascript: [js-mplex](https://github.com/libp2p/js-libp2p-mplex)
- Rust: [rust-libp2p mplex module](https://github.com/libp2p/rust-libp2p/tree/master/muxers/mplex)

#### yamux

[yamux](https://github.com/hashicorp/yamux) is a multiplexing protocol designed by [Hashicorp](https://www.hashicorp.com/).

yamux offers more sophisticated flow control than mplex, and can scale to thousands of multiplexed streams over a single connection.

yamux是由Hashicorp设计的多路复用协议。

yamux提供比mplex更复杂的流量控制，并且可以通过单个连接扩展到数千个多路复用流。


yamux is currently supported in go and rust:

- Go: [go-smux-yamux](https://github.com/whyrusleeping/go-smux-yamux)
- Rust: [rust-libp2p yamux module](https://github.com/libp2p/rust-libp2p/tree/master/muxers/yamux).

#### quic

[QUIC][wiki-quic] is a [transport](/concepts/transport/) protocol that contains a "native" stream multiplexer. libp2p will automatically use the native multiplexer for streams using a quic transport.

QUIC是一种包含“本地”流复用器的传输协议。 libp2p将使用quic传输自动使用本机多路复用器进行流。

目前通过go-libp2p-quic-transport支持quic。

quic is currently supported in go via [go-libp2p-quic-transport](https://github.com/libp2p/go-libp2p-quic-transport).

#### spdy

[SPDY][wiki-spdy] is a Google-developed protocol that was the precursor to HTTP/2. SPDY implements a stream multiplexer, which is supported by some libp2p implementations:

SPDY是Google开发的协议，是HTTP / 2的前身。 SPDY实现了一个流多路复用器，一些libp2p实现支持它：

- Go: [go-smux-spdystream](https://github.com/whyrusleeping/go-smux-spdystream)
- Javascript: [js-libp2p-spdy](https://github.com/libp2p/js-libp2p-spdy)

#### muxado

[muxado](https://github.com/inconshreveable/muxado) is a go stream muxing library, supported by go-libp2p via [go-smux-muxado](https://github.com/whyrusleeping/go-smux-muxado).

muxado是一个go stream muxing库，由go-libp2p通过go-smux-muxado支持。

<!-- links -->
[interface-stream-muxing]: https://github.com/libp2p/interface-stream-muxer

[repo-multistream-select]: https://github.com/multiformats/multistream-select 

[wiki-quic]: https://en.wikipedia.org/wiki/QUIC
[wiki-spdy]: https://en.wikipedia.org/wiki/SPDY
