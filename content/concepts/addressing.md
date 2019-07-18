---
title: Addressing
weight: 6
---

Flexible networks need flexible addressing systems. Since libp2p is designed to work across a wide variety of networks, we need a way to work with a lot of different addressing schemes in a consistent way.

A `multiaddress` (often abbreviated `multiaddr`), is a convention for encoding multiple layers of addressing information into a single "future-proof" path structure. It [defines][spec_multiaddr] human-readable and machine-optimized encodings of common transport and overlay protocols and allows many layers of addressing to be combined and used together.

For example: `/ip4/127.0.0.1/udp/1234` encodes two protocols along with their essential addressing information. The `/ip4/127.0.0.1` informs us that we want the `127.0.0.1` loopback address of the IPv4 protocol, and `/udp/1234` tells us we want to send UDP packets to port `1234`.

Things get more interesting as we compose further. For example, the multiaddr `/p2p/QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N` uniquely identifies my local IPFS node, using libp2p's [registered protocol id](https://github.com/multiformats/multiaddr/blob/master/protocols.csv) `/p2p/` and the [multihash](/reference/glossary/#multihash) of my IPFS node's public key.

{{% notice "tip" %}}
For more on peer identity and its relation to public key cryptography, see [Peer Identity](./peer-identity/).
{{% /notice %}}

Let's say that I have the peer id `QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N` as above, and my public ip is `7.7.7.7`. I start my libp2p application and listen for connections on TCP port `4242`.

Now I can start [handing out multiaddrs to all my friends](/concepts/peer-routing/), of the form `/ip4/7.7.7.7/tcp/4242/p2p/QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N`. Combining my "location multiaddr" (my IP and port) with my "identity multiaddr" (my libp2p `PeerId`), produces a new multiaddr containing both key pieces of information.

Now not only do my friends know where to find me, anyone they give that address to can verify that the machine on the other side is really me, or at least, that they control the private key for my `PeerId`. They also know (by virtue of the `/p2p/` protocol id) that I'm likely to support common libp2p interactions like opening connections and negotiating what application protocols we can use to communicate. That's not bad!

This can be extended to account for multiple layers of addressing and abstraction. For example, the [addresses used for circuit relay](/concepts/circuit-relay/#relay-addresses) combine transport addresses with multiple peer identities to form an address that describes a "relay circuit":

```
/ip4/7.7.7.7/tcp/4242/p2p/QmRelay/p2p-circuit/p2p/QmRelayedPeer
```


灵活的网络需要灵活的寻址系统由于libp2p旨在适用于各种各样的网络，因此我们需要一种以一致的方式处理许多不同寻址方案的方法。

多地址（通常缩写为multiaddr）是将多层寻址信息编码成单个“面向未来”的路径结构的惯例。它定义了人工可读和机器优化的公共传输和覆盖协议编码，并允许将多层寻址组合在一起使用。

例如：/ip4/127.0.0.1/udp/1234对两个协议及其基本寻址信息进行编码。 /ip4/127.0.0.1通知我们我们想要IPv4协议的127.0.0.1环回地址，而/ udp / 1234告诉我们要将UDP数据包发送到端口1234。

随着我们的进一步构成，事情变得更加有趣。例如，multiaddr / p2p / QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N使用libp2p的注册协议id / p2p /以及我的IPFS节点公钥的multihash唯一标识我的本地IPFS节点。

{{％notice“tip”％}}有关对等身份及其与公钥加密的关系的更多信息，请参阅对等身份。 {{％ /注意 ％}}

假设我有如上所述的对等ID QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N，我的公共IP是7.7.7.7。我启动我的libp2p应用程序并侦听TCP端口4242上的连接。

现在我可以开始向所有朋友分发多个用户，格式为/ip4/7.7.7.7/tcp/4242/p2p/QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N。将我的“location multiaddr”（我的IP和端口）与我的“identity multiaddr”（我的libp2p PeerId）组合在一起，产生一个包含两个关键信息的新的multiaddr。

现在不仅我的朋友知道在哪里找到我，他们给那个地址的任何人都可以验证另一方的机器是我，或者至少是他们控制我的PeerId的私钥。他们也知道（凭借/ p2p / protocol id）我可能支持常见的libp2p交互，例如打开连接和协商我们可以用来通信的应用程序协议。那还不错！

这可以扩展到考虑多层寻址和抽象。例如，用于电路中继的地址将传输地址与多个对等体标识组合以形成描述“中继电路”的地址：


### More information

For more detail, see the [multiaddr spec][spec_multiaddr], which has links to many implementations.


[spec_multiaddr]: https://github.com/multiformats/multiaddr
