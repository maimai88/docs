---
title: Peer Identity
weight: 4
---

A Peer Identity (often written `PeerId`) is a unique reference to a specific
peer within the overall peer-to-peer network.

As well as serving as a unique identifier for each peer, a PeerId is a
verifiable link between a peer and its public cryptographic key.

对等身份（通常写为PeerId）是对整个对等网络内特定对等体的唯一引用。

除了作为每个对等体的唯一标识符之外，PeerId是对等体与其公共加密密钥之间的可验证链接。

### What is a PeerId

Each libp2p peer controls a private key, which it keeps secret from all other
peers. Every private key has a corresponding public key, which is shared with
other peers.

Together, the public and private key (or "key pair") allow peers to establish
[secure communication](/concepts/secure-comms/) channels with each other.

Conceptually, a PeerId is a [cryptographic hash][wiki_hash_function] of a peer's
public key. When peers establish a secure channel, the hash can be used to
verify that the public key used to secure the channel is the same one used
to identify the peer.

The [PeerId spec][spec_peerid] goes into detail about the byte formats used
for libp2p public keys and how to hash the key to produce a valid PeerId.

PeerIds are encoded using the [multihash][definition_multihash] format, which
adds a small header to the hash itself that identifies the hash algorithm used
to produce it.

每个libp2p对等体控制一个私钥，它对所有其他对等体保密。 每个私钥都有一个相应的公钥，与其他对等方共享。

公钥和私钥（或“密钥对”）一起允许对等体彼此建立安全的通信信道。

从概念上讲，PeerId是对等方公钥的加密哈希。 当对等体建立安全信道时，可以使用散列来验证用于保护信道的公钥是否与用于识别对等体的公钥相同。

PeerId规范详细介绍了用于libp2p公钥的字节格式以及如何散列密钥以生成有效的PeerId。

PeerIds使用multihash格式进行编码，该格式为散列本身添加了一个小标头，用于标识用于生成散列算法的散列算法。

### How are Peer Ids represented as strings?

PeerIds are [multihashes][definition_multihash], which are defined as a
compact binary format.

It's very common to see multihashes encoded into
[base 58][wiki_base58], using
[the same alphabet used by bitcoin](https://en.bitcoinwiki.org/wiki/Base58#Alphabet_Base58).

Here's an example of a PeerId represented as a base58-encoded multihash:
`QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N`

While it's possible to represent multihashes in many textual formats
(for example as hexadecimal, base64, etc), PeerIds *always* use the base58
encoding, with no [multibase prefix](https://github.com/multiformats/multibase)
when encoded into strings.

PeerIds是多重的，它被定义为紧凑的二进制格式。

使用比特币使用的相同字母表看多基数编码到基数58中是很常见的。

下面是一个PeerId的示例，表示为base58编码的multihash：QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N

虽然可以用多种文本格式表示多重（例如十六进制，base64等），但PeerIds始终使用base58编码，编码成字符串时没有多基地前缀。

### PeerIds in multiaddrs

A PeerId can be encoded into a [multiaddr][definition_multiaddr] as a `/p2p`
address with the PeerId as a parameter.

If my peer id is `QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N`, a
libp2p multiaddress for me would be:

```
/p2p/QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N
```

As with other multiaddrs, a `/p2p` address can be encapsulated into
another multiaddr to compose into a new multiaddr. For example, I can combine
the above with a [transport](/concepts/transport/) address
`/ip4/7.7.7.7/tcp/4242` to produce this very useful address:

```
/ip4/7.7.7.7/tcp/4242/p2p/QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N
```

This provides enough information to dial a specific peer over a TCP/IP
transport. If some other peer has taken over that IP address or port, it will be
immediately obvious, since they will not have control over the key pair used to
produce the PeerId embedded in the address.

**For more on addresses in libp2p, see [Addressing](/concepts/addressing/)**

{{% notice "note" %}}
The multiaddr protocol for libp2p addresses was originally written `/ipfs`
and was later renamed to `/p2p`.
The two are equivalent and have the same binary
representation in multiaddrs. Which one is rendered in the string format
depends on the version of the multiaddr library in use.
{{% /notice %}}


可以将PeerId编码为multiaddr作为/ p2p地址，并将PeerId作为参数。

如果我的对等ID是QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N，那么我的libp2p多地址将是：

/ P2P / QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N
与其他multiaddr一样，可以将/ p2p地址封装到另一个multiaddr中以组成新的multiaddr。例如，我可以将上面的内容与传输地址/ip4/7.7.7.7/tcp/4242结合起来，以生成这个非常有用的地址：

/ip4/7.7.7.7/tcp/4242/p2p/QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N
这提供了足够的信息来通过TCP / IP传输拨打特定对等体。如果某个其他对等体已经接管了该IP地址或端口，那么它将立即显而易见，因为它们无法控制用于生成嵌入在该地址中的PeerId的密钥对。

有关libp2p中地址的更多信息，请参阅寻址

{{％notice“note”％}} libp2p地址的multiaddr协议最初是用/ ipfs编写的，后来被重命名为/ p2p。这两者是等价的，并且在multiaddrs中具有相同的二进制表示。以字符串格式呈现哪一个取决于正在使用的multiaddr库的版本。 {{％ /注意 ％}}


### PeerInfo

Another common libp2p data structure related to peer identity is the `PeerInfo`
structure.

A `PeerInfo` combines a `PeerId` with a set of [multiaddrs][definition_multiaddr]
that the peer is listening on.

libp2p applications will generally keep a "peer store" or "peer book" that
maintains a collection of `PeerInfo` objects for all the peers that they're
aware of.

The peer store acts as a sort of "phone book" when dialing out to
other peers; if a peer is in the peer store, we probably don't need to discover
their addresses using [peer routing](/conecpts/peer-routing/).

与对等身份相关的另一种常见libp2p数据结构是PeerInfo结构。

PeerInfo将PeerId与对等方正在侦听的一组多分析器组合在一起。

libp2p应用程序通常会保留一个“对等存储”或“对等存储”，它为所有他们知道的对等端维护一组PeerInfo对象。

拨打给其他同行时，同行商店就像一种“电话簿”; 如果对等体在对等存储中，我们可能不需要使用对等路由发现它们的地址。


{{% notice "tip" %}}
You can help improve this article! Please [refer to this issue](https://github.com/libp2p/docs/issues/12) to make suggestions and let us know how to help.
{{% /notice %}}

[wiki_hash_function]: https://en.wikipedia.org/wiki/Cryptographic_hash_function
[wiki_base58]: https://en.wikipedia.org/wiki/Base58

[definition_multiaddr]: /reference/glossary/#multiaddr
[definition_multihash]: /reference/glossary/#multihash

<!-- TODO(yusef): update link when peer-id PR lands -->
[spec_peerid]: https://github.com/libp2p/specs/pull/100
