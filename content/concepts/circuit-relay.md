---
title: Circuit Relay
weight: 3
---

Circuit relay is a [transport protocol](/concepts/transport/) that routes traffic between two peers over a third-party "relay" peer.


电路中继是一种传输协议，它通过第三方“中继”对等体在两个对等体之间路由流量。

In many cases, peers will be unable to [traverse their NAT](/concepts/nat/) in a way that makes them publicly accessible. Or they may not share common [transport protocols](/concepts/transport/) that would allow them to communicate directly.

在许多情况下，同行将无法以可公开访问的方式遍历其NAT。或者他们可能不共享允许他们直接通信的公共传输协议。

To enable peer-to-peer architectures in the face of connectivity barriers like NAT, libp2p [defines a protocol called p2p-circuit][spec_relay]. When a peer isn't able to listen on a public address, it can dial out to a relay peer, which will keep a long-lived connection open. Other peers will be able to dial through the relay peer using a `p2p-circuit` address, which will forward traffic to its destination.

为了在面对NAT等连接障碍时启用对等体系结构，libp2p定义了一种称为p2p-circuit的协议。当对等体无法监听公共地址时，它可以拨出到中继对等体，这将保持长期连接打开。其他对等体将能够使用p2p电路地址通过中继对等体拨号，该地址将流量转发到其目的地。

The circuit relay protocol is inspired by [TURN](https://tools.ietf.org/html/rfc5766), which is part of the [Interactive Connectivity Establishment](https://tools.ietf.org/html/rfc8445) collection of NAT traversal techniques.

电路中继协议的灵感来自TURN，它是NAT穿越技术的交互式连接建立集合的一部分。

{{% notice "note" %}}
Relay connections are end-to-end encrypted, which means that the peer acting as the relay is unable to read or tamper with any traffic that flows through the connection.
中继连接是端到端加密的，这意味着充当中继的对等方无法读取或篡改流经连接的任何流量。 
{{% /notice %}}

An important aspect of the relay protocol is that it is not "transparent". In other words, both the source and destination are aware that traffic is being relayed. This is useful, since the destination can see the relay address used to open the connection and can potentially use it to construct a path back to the source. It is also not anonymous - all participants are identified using their peer id, including the relay node.

中继协议的一个重要方面是它不是“透明的”。换句话说，源和目的地都知道正在中继流量。这很有用，因为目标可以看到用于打开连接的中继地址，并且可以使用它来构建返回源的路径。它也不是匿名的 - 所有参与者都使用他们的对等身份识别，包括中继节点。

#### Relay addresses

A relay circuit is identified using a [multiaddr][definition_muiltiaddress] that includes the [peer id](/concepts/peer-id/) of the peer whose traffic is being relayed (the listening peer or "relay target").

使用multiaddr识别中继电路，该multiaddr包括正在中继流量的对等体的对等ID（侦听对等体或“中继目标”）。

Let's say that I have a peer with the peer id `QmAlice`. I want to give out my address to my friend `QmBob`, but I'm behind a NAT that won't let anyone dial me directly.

假设我有一个同行ID为QmAlice的对等体。我想把我的地址发给我的朋友QmBob，但我背后的NAT不会让任何人直接拨打我。

The most basic `p2p-circuit` address I can construct looks like this:

我可以构造的最基本的p2p电路地址如下所示：

`/p2p-circuit/p2p/QmAlice`

The address above is interesting, because it doesn't include any [transport](/concepts/transport/) addresses for either the peer we want to contact (`QmAlice`) or for the relay peer that will convey the traffic. Without that information, the only chance a peer has of dialing me is to discover a relay and hope they have a connection to me.

上面的地址很有意思，因为它不包括我们想要联系的对等端（QmAlice）或传输流量的中继对等端的任何传输地址。如果没有这些信息，同伴拨打我的唯一机会就是发现转发并希望他们与我联系。

A better address would be something like `/p2p/QmRelay/p2p-circuit/p2p/QmAlice`. This includes the identity of a specific relay peer, `QmRelay`. If a peer already knows how to open a connection to `QmRelay`, they'll be able to reach us.

一个更好的地址就像/ p2p / QmRelay / p2p-circuit / p2p / QmAlice。这包括特定中继对等体QmRelay的标识。如果对等方已经知道如何打开与QmRelay的连接，他们将能够联系我们。

Better still is to include the transport addresses for the relay peer in the address. Let's say that I've established a connection to a specific relay with the peer id `QmRelay`. They told me via the identify protocol that they're listening for TCP connections on port `55555` at IPv4 address `7.7.7.7`. I can construct an address that describes a path to me through that specific relay over that transport:


更好的是在地址中包含中继对等体的传输地址。假设我已经使用对等ID QmRelay建立了与特定中继的连接。他们通过身份识别协议告诉我，他们正在侦听端口55555上IPv4地址7.7.7.7的TCP连接。我可以构造一个地址来描述通过该传输的特定中继的路径：

`/ip4/7.7.7.7/tcp/55555/p2p/QmRelay/p2p-circuit/p2p/QmAlice`

Everything prior to the `/p2p-circuit/` above is the address of the relay peer, which includes the transport address and their peer id `QmRelay`. After `/p2p-circuit/` is the peer id for my peer at the other end of the line, `QmAlice`.


/ p2p-circuit / above之前的所有内容都是中继对等体的地址，其中包括传输地址和它们的对等体ID QmRelay。在/ p2p-circuit /之后是同行的另一端的同伴ID，QmAlice。

By giving the full relay path to my friend `QmBob`, they're able to quickly establish a relayed connection without having to "ask around" for a relay that has a route to `QmAlice`.

通过向我的朋友QmBob提供完整的中继路径，他们能够快速建立中继连接，而无需“询问”具有到QmAlice路由的中继。

{{% notice "tip" %}}
When [advertising your address](/concepts/peer-routing/), it's best to provide relay addresses that include the transport address of the relay peer. If the relay has many transport addresses, you can advertise a `p2p-circuit` through each of them.

在宣传您的地址时，最好提供包含中继对等体传输地址的中继地址。如果中继具有许多传输地址，则可以通过每个传输地址通告p2p电路。

{{% /notice %}}




#### Autorelay

The circuit relay protocol is only effective if peers can discover willing relay peers that are accessible to both sides of the relayed connection.


电路中继协议仅在对等体能够发现中继连接的两端都可访问的自愿中继对等体时才有效。

While it's possible to simply "hard-code" a list of well-known relays into your application, this adds a point of centralization to your architecture that you may want to avoid. This kind of bootstrap list is also a potential point of failure if the bootstrap nodes become unavailable.

虽然可以简单地将众所周知的继电器列表“硬编码”到您的应用程序中，但这会为您的架构增加一个集中点，您可能希望避免这种情况。如果引导程序节点不可用，则此类引导列表也是潜在的故障点。

Autorelay is a feature (currently implemented in go-libp2p) that a peer can enable to attempt to discover relay peers using libp2p's [content routing](/concepts/content-routing/) interface.


自动重新映射是一种功能（目前在go-libp2p中实现），对等体可以使用libp2p的内容路由接口尝试发现中继对等体。

When Autorelay is enabled, a peer will try to discover one or more public relays and open relayed connections. If successful, the peer will advertise the relay addresses using libp2p's [peer routing](/concepts/peer-routing/) system.

启用自动重新映射后，对等方将尝试发现一个或多个公共中继并打开中继连接。如果成功，对等体将使用libp2p的对等路由系统通告中继地址。

{{% notice "warning" %}}
Autorelay is under active development and should be considered experimental. There are currently no protections against malicious or malfunctioning relays which could advertise relay services and refuse to provide them.

自动播放正在积极开发中，应该被视为实验性的。目前没有针对可能通告中继服务并拒绝提供它们的恶意或故障中继的保护。
{{% /notice %}}

##### How Autorelay works

The Autorelay service is responsible for:

Autorelay服务负责：

1. discovering relay nodes around the world,   


发现世界各地的中继节点，

2. establishing long-lived connections to them, and

建立与他们的长期联系

3. advertising relay-enabled addresses for ourselves to our peers, thus making ourselves routable through delegated routing.

为我们自己的同行提供广告中继功能的地址，从而通过委派路由使自己可路由。

When [AutoNAT service](/concepts/nat/#autonat) detects we're behind a NAT that blocks inbound connections, Autorelay jumps into action, and the following happens:

当AutoNAT服务检测到我们位于阻止入站连接的NAT后面时，Autorelay会跳转到操作中，并发生以下情况：

1. We locate candidate relays by running a DHT provider search for the `/libp2p/relay` namespace.

我们通过运行DHT提供程序搜索/ libp2p / relay命名空间来定位候选中继。

2. We select three results at random, and establish a long-lived connection to them (`/libp2p/circuit/relay/0.1.0` protocol). Support for using latency as a selection heuristic will be added soon.

我们随机选择三个结果，并建立一个长期连接（/libp2p/circuit/relay/0.1.0协议）。将很快添加对使用延迟作为选择启发式的支持。

3. We enhance our local address list with our newly acquired relay-enabled multiaddrs, with format: `/ip4/1.2.3.4/tcp/4001/p2p/QmRelay/p2p-circuit`, where:
   `1.2.3.4` is the relay's public IP address, `4001` is the libp2p port, and `QmRelay` is the peer ID of the relay.
   Elements in the multiaddr can change based on the actual transports at use.

我们使用新获得的支持中继的multiaddrs增强我们的本地地址列表，格式为：/ip4/1.2.3.4/tcp/4001/p2p/QmRelay/p2p-circuit，其中：1.2.3.4是中继的公共IP地址，4001是libp2p端口，QmRelay是中继的对等ID。 multiaddr中的元素可以根据使用时的实际传输进行更改。
   
4. We announce our new relay-enabled addresses to the peers we're already connected to via the `IdentifyPush` protocol.

我们通过IdentifyPush协议向我们已经连接的对等方发布新的启用中继的地址。

The last step is crucial, as it enables peers to learn our updated addresses, and in turn return them when another peer looks us up.

最后一步至关重要，因为它使同行能够学习我们更新的地址，并在另一个同伴看到我们时返回它们。

[spec_relay]: https://github.com/libp2p/specs/tree/master/relay
[definition_muiltiaddress]: /reference/glossary/#mulitaddress
