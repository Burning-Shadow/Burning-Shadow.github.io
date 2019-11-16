---
title: SSL && TSL
date: 2019-04-09 22:24:00
categories: 网络协议
tags:
 - TSL
 - SSL
---

众所周知 HTTPS 是基于 HTTP 协议的，安全性有一定保障的协议，而其保障安全性的工具就是通过公私钥不对称加密，及 CA（证书机制）来完成对数据的加密，以此达到提升安全性的作用。

但是！！！**HTTPS 加密中真正起作用的其实是 SSL/TSL 协议。**

Ps：本文转自掘金 [hi_xgb](<https://juejin.im/post/584b76d3a22b9d0058d5036f#heading-1>)大大。

<!--more-->

![](https://pic.superbed.cn/item/5cac94383a213b0417f0b826)

如图，SSL/TSL 协议在 HTTP 协议之下的表示层，对于上层的应用层来说，原来的发送接收数据流程不变，如此就可以完美兼容老版 HTTP 协议

### SSL/TSL握手过程

![](https://pic.superbed.cn/item/5cac9af73a213b0417f0fc66)

#### Client Hello

握手第一步是客户端向服务端发送`Client Hello`消息，其中包含了一个客户端生成的随机数**`Random1`**、客户端支持的加密套件（`Support Ciphers`）和`SSL Version`等信息，通过`Wireshark`抓包可以看到如下信息

![](https://pic.superbed.cn/item/5cac9b8c3a213b0417f101b3)

#### Server Hello

第二步是服务端向客户端发送`Server Hello`消息。

此消息会从`Client Hello`传过来的`Support Ciphers`里确定一份加密套件，这个套件决定了后续加密和生成摘要时具体使用那些算法。

另外还会生成一份随机数**`Random2`**。`Random1`和`Random2`会在后续生成对称密钥时用到。

![](https://pic.superbed.cn/item/5cac9c4e3a213b0417f10987)

#### Certificate

这一步是服务端将自己的证书下发给客户端，让客户端验证自己的身份。客户端验证通过后去除证书中的公钥

![](https://pic.superbed.cn/item/5cac9c873a213b0417f10c11)

#### Server Key Exchange

若是 DH 算法，这里发送服务器使用的 DH 参数。RSA 算法则不需要这一步

![](https://pic.superbed.cn/item/5cac9cd13a213b0417f10f8f)

#### Certificate Request

`Certificate Request`是服务端要求客户端上保证书。这一步是可选的，对于安全性要求高的场景会用到

#### Server Hello Done

`Server Hello Done`通知客户端`Server Hello`过程结束

![](https://pic.superbed.cn/item/5cac9d3c3a213b0417f1167e)

#### Certificate Verify

客户端收到服务端传拉力的证书后，先从 CA 验证该证书的合法性，通过后取出证书中的服务端公钥，再生成一个随机数**`Random3`**，再用服务端公钥非对称加密`Random3`生成**`PreMaster Key`**

#### Client Key Exchange

上面客户端根据服务器传来的公钥生成了**`PreMaster Key`**，`Client Key Exchange`就是将这个 key 传给服务端，服务端再用自己的私钥解出这个**`PreMaster Key`** 得到客户端生成的**`Random3`**。至此，客户端和服务端都拥有**`Random1`** + **`Random2`** + **`Random3`**，两边再根据同样的算法就可以生成一份秘钥，握手结束后的应用层数据都是使用这个秘钥进行对称加密。为什么要使用三个随机数呢？这是因为 SSL/TLS 握手过程的数据都是明文传输的，并且多个随机数种子来生成秘钥不容易被暴力破解出来。客户端将**`PreMaster Key`**传给服务端的过程如下图所示：

![](https://pic.superbed.cn/item/5cac9eac3a213b0417f12546)

#### Change Cipher Spec(Client)

这里客户端通知服务端后面再发送的消息都会使用前面协商出来的密钥加密，是一条事件消息

![](https://pic.superbed.cn/item/5cac9f1c3a213b0417f12ac8)

#### Encrypted Handshake Message(Client)

这步对应`Client Finish`消息。客户端将前面的握手消息生成摘要再用协商好的密钥加密。

这是客户端发出的第一条加密消息，服务端接收后会用密钥解密，能接出来说明前面协商出来的密钥是一致的

![](https://pic.superbed.cn/item/5cac9f9e3a213b0417f13270)

#### Change Cipher Spec(Server)

服务端通知客户端后面的消息都会使用加密，也是一条事件信息

#### Encrypted Handshake Message(Server)

此处对应的是`Server Finish`消息。服务端也会将握手过程的消息生成摘要再用密钥加密。这是服务端发出的第一条加密消息，客户端接收后会用密钥解密，能解出来说明协商的密钥是一致的

![](https://pic.superbed.cn/item/5caca0293a213b0417f13a76)

#### Application Data

此时，双方已安全地协商除了同一份密钥，将所有的应用层数据都用这个密钥加密过后再通过`TCP`进行可靠传输

### 双向验证

![](https://pic.superbed.cn/item/5caca6803a213b0417f1812f)

### 握手过程优化

如果每次重连都要重新握手，那么显然会浪费大量的资源和时间。所以我们不妨在`Client Hello`消息中附带上上一次的`Session ID`。服务端接收到这个`Session ID`后若能复用就不再进行后续的握手过程

![](https://pic.superbed.cn/item/5cac9b8c3a213b0417f101b3)

除了上述的`Session`复用，SSL/TSL 握手还有一些优化技术，例如`False Start`、`Session Ticket`等。[这方面的介绍具体可以参考这个](<https://imququ.com/post/optimize-tls-handshake.html>)

### Final

最后再次感谢一次`hi_xgb`大大的博客，码住我每天都要看一遍！。

