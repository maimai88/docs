---
title: "What is libp2p?"
weight: 2
---

Good question! The one-liner pitch is that libp2p is a modular system of *protocols*, *specifications* and *libraries* that enable the development of peer-to-peer network applications.

好问题！ 一线间距是libp2p是一个协议，规范和库的模块化系统，可以开发对等网络应用程序。

<!--more-->

## Peer-to-peer basics

There's a lot to unpack in that one-liner! Let's start with the last bit, "peer-to-peer network applications." You may be here because you're knee-deep in development of a peer-to-peer system and are looking for help. Likewise, you may be here because you're just exploring the world of peer-to-peer networking for the first time. Either way, we ought to spend a minute defining our terms upfront, so we can have some [shared vocabulary][glossary] to build on.

在那个单行中打开很多东西！让我们从最后一点开始，“点对点网络应用程序。”您可能在这里，因为您在开发对等系统方面处于领先地位并且正在寻求帮助。同样，你可能在这里，因为你是第一次探索点对点网络的世界。无论哪种方式，我们都应该花一分钟时间来定义我们的术语，这样我们就可以建立一些共享的词汇表。

A [peer-to-peer network][definition_p2p] is one in which the participants (referred to as [peers][definition_peer] or nodes) communicate with one another directly, on more or less "equal footing". This does not necessarily mean that all peers are identical; some may have different roles in the overall network. However, one of the defining characteristics of a peer-to-peer network is that they do not require a priviliged set of "servers" which behave completely differently from their "clients", as is the case in the the predominant [client / server model][definition_client_server].

对等网络是参与者（称为对等体或节点）在或多或少“平等地”直接相互通信的网络。这并不一定意味着所有同伴都是相同的;有些人可能在整个网络中扮演不同的角色。然而，对等网络的一个定义特征是它们不需要一组特权的“服务器”，其行为与其“客户端”完全不同，如主要客户端/服务器模型中的情况。

Because the definition of peer-to-peer networking is quite broad, many different kinds of systems have been built that all fall under the umbrella of "peer-to-peer". The most culturally prominent examples are likely the file sharing networks like bittorrent, and, more recently, the proliferation of blockchain networks that communicate in a peer-to-peer fashion.


由于对等网络的定义相当广泛，因此构建了许多不同类型的系统，这些系统都属于“点对点”的范畴。最具文化意义的例子可能是像bittorrent这样的文件共享网络，以及最近以对等方式进行通信的区块链网络的激增。

## What problems can libp2p solve?

While peer-to-peer networks have many advantages over the client / server model, there are also challenges that are unique and require careful thought and practice to overcome. In our process of overcoming these challenges while building [IPFS](https://ipfs.io), we took care to build our solutions in a modular, composable way, into what is now libp2p. Although libp2p grew out of IPFS, it does not require or depend on IPFS, and today [many projects][built_with_libp2p] use libp2p as their network transport layer. Together we can leverage our collective experience and solve these foundational problems in a way that benefits an entire ecosystem of developers and a world of users.

虽然点对点网络比客户端/服务器模型具有许多优势，但也存在独特的挑战，需要仔细思考和实践才能克服。在构建IPFS的过程中克服这些挑战，我们注重以模块化，可组合的方式构建我们的解决方案，现在是libp2p。尽管libp2p源于IPFS，但它不需要或依赖于IPFS，而今天许多项目都使用libp2p作为其网络传输层。我们可以共同利用我们的集体经验，以一种有利于整个开发人员生态系统和用户世界的方式解决这些基础问题。

Here I'll try to briefly outline the main problem areas that are addressed by libp2p today (early 2019). This is an ever-growing space, so don't be surprised if things change over time. We'll do our best to keep this section up-to-date as things progress, but if you notice something missing or have other ideas for improving this documentation, please [reach out to let us know][help_improve_docs].


在这里，我将简要概述今天（2019年初）libp2p解决的主要问题领域。这是一个不断增长的空间，所以如果事情随着时间的推移而变化，不要感到惊讶。随着事情的进展，我们将尽最大努力使这一部分保持最新状态，但如果您发现缺少某些内容或有其他改进此文档的想法，请与我们联系以告知我们。

<!-- TODO: as concept articles are written expanding on the below, add links -->

### Transport

At the foundation of libp2p is the transport layer, which is responsible for the actual transmission and receipt of data from one peer to another. There are many ways to send data across networks in use today, with more in development and still more yet to be designed. libp2p provides a simple [interface](https://github.com/libp2p/interface-transport) that can be adapted to support existing and future protocols, allowing libp2p applications to operate in many different runtime and networking environments.

在libp2p的基础上是传输层，它负责从一个对等体到另一个对等体的实际传输和接收。 有许多方法可以在当前使用的网络中发送数据，还有更多的开发方式，还有更多的设计方法。 libp2p提供了一个简单的接口，可以适应现有和未来的协议，允许libp2p应用程序在许多不同的运行时和网络环境中运行。

### Identity

In a world with billions of networked devices, knowing who you're talking to is key to secure and reliable communication. libp2p uses [public key cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography) as the basis of peer identity, which serves two complementary purposes.  First, it gives each peer a globally unique "name", in the form of a [PeerId][definition_peerid]. Second, the `PeerId` allows anyone to retrieve the public key for the identified peer, which enables secure communication between peers.

在拥有数十亿网络设备的世界中，了解与您交谈的对象是安全可靠通信的关键。 libp2p使用公钥加密作为对等身份的基础，这有两个互补的目的。 首先，它以[PeerId] [definition_peerid]的形式为每个对等体提供全局唯一的“名称”。 其次，PeerId允许任何人为所识别的对等体检索公钥，从而实现对等体之间的安全通信。

### Security

It's essential that we be able to send and receive data between peers *securely*, meaning that we can trust the [identity](#identity) of the peer we're communicating with and that no third-party can read our conversation or alter it in-flight.

至关重要的是，我们能够安全地在对等体之间发送和接收数据，这意味着我们可以信任我们正在与之通信的同伴的身份，并且没有第三方可以阅读我们的对话或在飞行中改变它。

libp2p supports "upgrading" a connection provided by a [transport](#transport) into a securely encrypted channel. The process is flexible, and can support multiple methods of encrypting communication. The current default is [secio][definition_secio], with support for [TLS 1.3](https://www.ietf.org/blog/tls13/) under development.

libp2p支持将传输提供的连接“升级”到安全加密的通道中。 该过程非常灵活，可以支持多种加密通信的方法。 当前默认值为[secio] [definition_secio]，支持开发中的TLS 1.3。

### Peer Routing

When you want to send a message to another peer, you need two key pieces of information: their [PeerId][definition_peerid], and a way to locate them on the network to open a connection.

当您想要向另一个对等体发送消息时，您需要两个关键信息：它们的[PeerId] [definition_peerid]，以及在网络上找到它们以打开连接的方法。

There are many cases where we only have the `PeerId` for the peer we want to contact, and we need a way to discover their network address. Peer routing is the process of discovering peer addresses by leveraging the knowledge of other peers.

在许多情况下，我们只为我们想要联系的同行提供PeerId，我们需要一种方法来发现他们的网络地址。对等路由是通过利用其他对等方的知识来发现对等地址的过程。

In a peer routing system, a peer can either give us the address we need if they have it, or else send our inquiry to another peer who's more likely to have the answer. As we contact more and more peers, we not only increase our chances of finding the peer we're looking for, we build a more complete view of the network in our own routing tables, which enables us to answer routing queries from others.

在对等路由系统中，对等方可以向我们提供我们需要的地址，或者将我们的查询发送给更有可能得到答案的另一个对等方。随着我们与越来越多的同行联系，我们不仅增加了找到我们正在寻找的同行的机会，我们在自己的路由表中构建了一个更完整的网络视图，这使我们能够回答其他人的路由查询。

The current stable implementation of peer routing in libp2p uses a [distributed hash table][definition_dht] to iteratively route requests closer to the desired `PeerId` using the [Kademlia][wiki_kademlia] routing algorithm.

libp2p中当前稳定的对等路由实现使用[分布式哈希表] [definition_dht]使用Kademlia路由算法迭代地路由更接近期望的PeerId的请求。


### Content Discovery

In some systems, we care less about who we're speaking with than we do about what they can offer us. For example, we may want some specific piece of data, but we don't care who we get it from since we're able to verify its integrity.

在某些系统中，我们不关心我们与谁交谈，而不关心他们能为我们提供什么。 例如，我们可能需要一些特定的数据，但我们不关心我们从谁那里得到它，因为我们能够验证其完整性。

libp2p provides a [content routing interface][interface_content_routing] for this purpose, with the primary stable implementation using the same [Kademlia][wiki_kademlia]-based DHT as used in peer routing.

libp2p为此提供了内容路由接口，主要稳定实现使用与对等路由中使用的相同的基于Kademlia的DHT。

### Messaging / PubSub

Sending messages to other peers is at the heart of most peer-to-peer systems, and pubsub (short for publish / subscribe) is a very useful pattern for sending a message to groups of interested receivers.

向其他对等体发送消息是大多数对等系统的核心，而pubsub（发布/订阅的简称）是用于向有兴趣的接收者组发送消息的非常有用的模式。

libp2p defines a [pubsub interface][interface_pubsub] for sending messages to all peers subscribed to a given "topic". The interface currently has two stable implementations; `floodsub` uses a very simple but inefficient  "network flooding" strategy, and [gossipsub](https://github.com/libp2p/specs/tree/master/pubsub/gossipsub) defines an extensible gossip protocol.  There is also active development in progress on [episub](https://github.com/libp2p/specs/blob/master/pubsub/gossipsub/episub.md), an extended `gossipsub` that is optimized for single source multicast and scenarios with a few fixed sources broadcasting to a large number of clients in a topic.

libp2p定义了一个pubsub接口，用于向订阅给定“主题”的所有对等方发送消息。该接口目前有两个稳定的实现; floodsub使用一种非常简单但效率低下的“网络洪泛”策略，而gossipsub定义了一种可扩展的八卦协议。在episub上还有一个正在进行的积极开发，这是一个针对单源多播进行优化的扩展gossipsub，以及一些固定源广播到一个主题中的大量客户端的场景。


[glossary]: {{< ref "/reference/glossary.md" >}}
[definition_dht]: {{< ref "/reference/glossary.md#dht" >}}
[definition_p2p]: {{< ref "/reference/glossary.md#p2p" >}}
[definition_peer]: {{< ref "/reference/glossary.md#peer" >}}
[definition_peerid]: {{< ref "/reference/glossary.md#peerid" >}}
[definition_secio]: {{< ref "/reference/glossary.md#secio" >}}
[definition_muiltiaddress]: {{< ref "/reference/glossary.md#multiaddr" >}}
[definition_client_server]: {{< ref "/reference/glossary.md#client-server" >}}

[interface_content_routing]: https://github.com/libp2p/interface-content-routing
[interface_pubsub]: https://github.com/libp2p/specs/tree/master/pubsub


[built_with_libp2p]: /TODO
[help_improve_docs]: /TODO

[wiki_kademlia]: https://en.wikipedia.org/wiki/Kademlia
