---
title: NAT Traversal
weight: 2
---

The internet is composed of countless networks, bound together into shared address spaces by foundational [transport protocols](/concepts/transport/).

互联网由无数网络组成，通过基础传输协议绑定到共享地址空间。

As traffic moves between network boundaries, it's very common for a process called Network Address Translation to occur. Network Address Translation (NAT) maps an address from one address space to another.


当流量在网络边界之间移动时，发生称为网络地址转换的过程非常常见。网络地址转换（NAT）将地址从一个地址空间映射到另一个地址空间。

NAT allows many machines to share a single public address, and it is essential for the continued functioning of the IPv4 protocol, which would otherwise be unable to serve the needs of the modern networked population with its 32-bit address space.

NAT允许许多机器共享单个公共地址，这对于IPv4协议的持续运行至关重要，否则IPv4协议无法满足具有32位地址空间的现代网络群体的需求。

For example, when I connect to my home wifi, my computer gets an IPv4 address of `10.0.1.15`. This is part of a range of IP addresses reserved for internal use by private networks. When I make an outgoing connection to a public IP address, the router replaces my internal IP with its own public IP address. When data comes back from the other side, the router will translate back to the internal address.

例如，当我连接到家庭wifi时，我的计算机获得的是IPv4地址10.0.1.15。这是专用网络内部使用的一系列IP地址的一部分。当我与公共IP地址建立传出连接时，路由器用自己的公共IP地址替换我的内部IP。当数据从另一侧返回时，路由器将转换回内部地址。

While NAT is usually transparent for outgoing connections, listening for incoming connections requires some configuration. The router listens on a single public IP address, but any number of machines on the internal network could handle the request. To serve requests, your router must be configured to send certain traffic to a specific machine, usually by mapping one or more TCP or UDP ports from the public IP to an internal one.

虽然NAT对于传出连接通常是透明的，但是侦听传入连接需要一些配置。路由器侦听单个公共IP地址，但内部网络上的任意数量的计算机都可以处理该请求。要处理请求，必须将路由器配置为将特定流量发送到特定计算机，通常是将一个或多个TCP或UDP端口从公共IP映射到内部IP。

While it's usually possible to manually configure routers, not everyone that wants to run a peer-to-peer application or other network service will have the ability to do so.

虽然通常可以手动配置路由器，但并非所有想要运行对等应用程序或其他网络服务的人都能够这样做。

We want libp2p applications to run everywhere, not just in data centers or on machines with stable public IP addresses. To enable this, here are the main approaches to NAT traversal available in libp2p today.

我们希望libp2p应用程序可以在任何地方运行，而不仅仅是在数据中心或具有稳定公共IP地址的计算机上运行。为了实现这一点，以下是今天libp2p中可用的NAT遍历的主要方法。



### Automatic router configuration

Many routers support automatic configuration protocols for port forwarding, most commonly [UPnP][wiki_upnp] or [nat-pmp.][wiki_nat-pmp]

许多路由器支持端口转发的自动配置协议，最常见的是UPnP或nat-pmp。

If you router supports one of those protocols, libp2p will attempt to automatically configure a port mapping that will allow it to listen for incoming traffic. This is usually the simplest option if supported by the network and libp2p implementation.



如果路由器支持其中一种协议，libp2p将尝试自动配置端口映射，以便侦听传入流量。 如果网络和libp2p实现支持，这通常是最简单的选项。

{{% notice "info" %}}
Support for automatic NAT configuration varies by libp2p implementation.
Check the [current implementation status](https://libp2p.io/implementations/#nat-traversal) for details.
支持自动NAT配置因libp2p实现而异。 检查当前的实施状态以获取详细信息 
{{% /notice %}}


### Hole-punching (STUN)

When an internal machine "dials out" and makes a connection to a public address, the router will map a public port to the internal IP address to use for the connection. In some cases, the router will also accept *incoming* connections on that port and route them to the same internal IP.

当内部计算机“拨出”并连接到公共地址时，路由器会将公共端口映射到内部IP地址以用于连接。在某些情况下，路由器还将接受该端口上的传入连接，并将它们路由到相同的内部IP。

libp2p will try to take advantage of this behavior when using IP-backed transports by using the same port for both dialing and listening, using a socket option called [`SO_REUSEPORT`](https://lwn.net/Articles/542629/).

libp2p将使用名为SO_REUSEPORT的套接字选项，通过使用相同的端口进行拨号和侦听，在使用IP支持的传输时尝试利用此行为。

If our peer is in a favorable network environment, they will be able to make an outgoing connection and get a publicly-reachable listening port "for free," but they might never know it. Unfortunately, there's no way for the dialing program to discover what port was assigned to the connection on its own.

如果我们的对等体处于有利的网络环境中，他们将能够进行外向连接并“免费”获得可公开访问的侦听端口，但他们可能永远不会知道它。不幸的是，拨号程序无法自己发现分配给连接的端口。

However, an external peer can can tell us what address they observed us on. We can then take that address and advertise it to other peers in our [peer routing network](/concepts/peer-routing/) to let them know where to find us.

但是，外部同行可以告诉我们他们观察我们的地址。然后我们可以获取该地址并将其通告给我们的对等路由网络中的其他对等方，以便让他们知道在哪里找到我们。

This basic premise of peers informing each other of their observed addresses is the foundation of [STUN][wiki_stun] (Session Traversal Utilities for NAT), which [describes][rfc_stun] a client / server protocol for discovering publicly reachable IP address and port combinations.

对等体通知对方观察到的地址的基本前提是STUN（NAT的会话遍历实用程序）的基础，其描述了用于发现可公开到达的IP地址和端口组合的客户端/服务器协议。

One of libp2p's core protocols is the [identify protocol][spec_identify], which allows one peer to ask another for some identifying information. When sending over their [public key](/concepts/peer-id/) and some other useful information, the peer being identified includes the set of addresses that it has observed for the peer asking the question.

libp2p的核心协议之一是识别协议，它允许一个对等方向另一个对等方询问某些识别信息。当通过他们的公钥和一些其他有用信息发送时，被识别的对等体包括它已经为提出问题的对等体观察到的一组地址。

This external discovery mechanism serves the same role as STUN, but without the need for a set of "STUN servers".

此外部发现机制与STUN具有相同的作用，但不需要一组“STUN服务器”。

The identify protocol allows some peers to communicate across NATs that would otherwise be impenetrable.

识别协议允许一些对等体通过NAT进行通信，否则这将是不可穿透的。

### AutoNAT

While the [identify protocol][spec_identify] described above lets peers inform each other about their observed network addresses, not all networks will allow incoming connections on the same port used for dialing out.

虽然上述识别协议允许对等方相互通知其观察到的网络地址，但并非所有网络都允许在用于拨出的同一端口上进行传入连接。

Once again, other peers can help us observe our situation, this time by attempting to dial us at our observed addresses. If this succeeds, we can rely on other peers being able to dial us as well and we can start advertising our listen address.

再一次，其他同行可以帮助我们观察我们的情况，这次尝试通过我们观察到的地址拨打我们。 如果成功，我们可以依赖其他同行也可以拨打我们，我们可以开始宣传我们的收听地址。

A libp2p protocol called AutoNAT lets peers request dial-backs from peers providing the AutoNAT service.

名为AutoNAT的libp2p协议允许对等方从请求提供AutoNAT服务的对等方请求回拨。

{{% notice "info" %}}
AutoNAT is currently implemented in go-libp2p via [go-libp2p-autonat](https://github.com/libp2p/go-libp2p-autonat).
 AutoNAT目前通过go-libp2p-autonat在go-libp2p中实现。
{{% /notice %}}

### Circuit Relay (TURN) 电路继电器（TURN）

In some cases, peers will be unable to traverse their NAT in a way that makes them publicly accessible.

在某些情况下，同行将无法以可公开访问的方式遍历其NAT。

libp2p provides a [Circuit Relay protocol](/concepts/circuit-relay/) that allows peers to communicate indirectly via a helpful intermediary peer.

libp2p提供了一种电路中继协议，允许对等体通过有用的中间对等体间接通信。

This serves a similar function to the [TURN protocol](https://tools.ietf.org/html/rfc5766) in other systems.

这在其他系统中提供与TURN协议类似的功能。

[wiki_upnp]: https://en.wikipedia.org/wiki/Universal_Plug_and_Play
[wiki_nat-pmp]: https://en.wikipedia.org/wiki/NAT_Port_Mapping_Protocol
[wiki_stun]: https://en.wikipedia.org/wiki/STUN
[rfc_stun]: https://tools.ietf.org/html/rfc3489
[lwn_reuseport]: https://lwn.net/Articles/542629/

<!-- TODO: update identify spec link after PR merge -->
[spec_identify]: https://github.com/libp2p/specs/pull/97



