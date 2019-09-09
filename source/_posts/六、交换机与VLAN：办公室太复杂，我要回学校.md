---
title: 六、交换机与VLAN：办公室太复杂，我要回学校
tags: [计算机网络]
categories: 网络协议

---

上一次，我们在宿舍组件了一个本地的局域网，可以愉快的玩游戏了。当然这是一个简单的场景，因为只有一台交换机，电脑数目很少。今天，我们来看看办公室的网络

<!--more-->

### 拓扑结构是怎么形成的？

<p>我们常见到的办公室大多是一排排的桌子，每个桌子都有网口，一排十几个座位就有十几个网口，一个楼层就会有几十个甚至上百个网口。如果算上所有楼层，这个场景自然比你宿舍里的复杂多了。具体哪里复杂呢？我来给你具体讲解。</p>

<p>首先，这个时候，一个交换机肯定不够用，需要多台交换机，交换机之间连接起来，就形成一个稍微复杂的<font size = 3 color= red>拓扑结构</font>。</p>

<p>我们先来看<font size = 3 color= red>两台交换机</font>的情形。两台交换机连接着三个局域网，每个局域网上都有多台机器。如果<font size = 3 color= red>机器 1 </font>只知道<font size = 3 color= red>机器 4 </font>的 <font size = 3 color= red>IP</font> 地址，当它想要访问<font size = 3 color= red>机器 4</font>，把包发出去的时候，它必须要知道<font size = 3 color= red>机器 4</font> 的 <font size = 3 color= red>MAC</font> 地址。</p>

![image](https://static001.geekbang.org/resource/image/7a/73/7a40046c5a2c7f7cd3c95b54488b9773.jpg)

```
图一和图二有点看不懂，图里的交换机和PC 是物理设备，这个LAN 是什么？不是应该交换机和PC 直接用一根线相连么？
这是个虚指的局域网，不一定直连，里面可以隐藏一些设备，例如hub，交换机
```

<p>于是<font size = 3 color= red>机器 1</font> 发起<font size = 3 color= red>广播</font>，<font size = 3 color= red>机器 2</font> 收到这个广播，但是这不是找它的，所以没它什么事。<font size = 3 color= red>交换机 A</font> 一开始是不知道任何拓扑信息的，在它收到这个广播后，<font size = 3 color= red>采取的策略</font>是，<font size = 3 color= red>除了广播包来的方向外，它还要转发给其他所有的网口。</font>于是<font size = 3 color= red>机器 3</font> 也收到广播信息了，但是这和它也没什么关系。</p>

> 注意：上节课我们使用HUB组件了局域网，这里这个局域网就是HUB组件的，工作在第一层，就是物理层，而交换机工作在第二层，也即数据链路层

<p>当然，<font size = 3 color= red>交换机 B</font> 也是能够收到广播信息的，但是这时候它也是不知道任何拓扑信息的，因而也是<font size = 3 color= red>进行广播的策略</font>，将包转发到局域网三。这个时候，<font size = 3 color= red>机器 4</font> 和<font size = 3 color= red>机器 5</font> 都收到了广播信息。<font size = 3 color= red>机器 4</font> 主动<font size = 3 color= red>响应</font>说，这是找我的，这是我的 <font size = 3 color= red>MAC 地址</font>。于是一个 <font size = 3 color= red>ARP 请求</font>就成功完成了。</p>

<p>在上面的过程中，<font size = 3 color= red>交换机 A</font>和<font size = 3 color= red>交换机 B</font> 都是能够学习到这样的信息：<font size = 3 color= red>机器 1</font> 是在<font size = 3 color= red>左边</font>这个网口的。当了解到这些拓扑信息之后，情况就好转起来。当<font size = 3 color= red>机器 2</font> 要访问<font size = 3 color= red>机器 1 </font>的时候，<font size = 3 color= red>机器 2 </font>并不知道<font size = 3 color= red>机器 1 </font>的 <font size = 3 color= red>MAC </font>地址，所以<font size = 3 color= red>机器 2</font> 会发起一个 <font size = 3 color= red>ARP</font> 请求。这个广播消息会到达<font size = 3 color= red>机器 1</font>，也同时会到达<font size = 3 color= red>交换机 A</font>。这个时候<font size = 3 color= red>交换机 A </font>已经知道<font size = 3 color= red>机器 1</font> 是不可能在<font size = 3 color= red>右边</font>的网口的，所以这个广播信息就<font size = 3 color= red>不会</font><font size = 3 color= red>广播到局域网二和局域网三</font>。</p>

<p>当</font><font size = 3 color= red>机器 3 </font>要访问</font><font size = 3 color= red>机器 1 </font>的时候，也需要发起一个广播的 </font><font size = 3 color= red>ARP</font> 请求。这个时候</font><font size = 3 color= red>交换机 A </font>和</font><font size = 3 color= red>交换机 B </font>都能够收到这个广播请求。</font><font size = 3 color= red>交换机 A </font>当然知道</font><font size = 3 color= red>主机 A</font> 是在左边这个网口的，所以会把广播消息转发到局域网一。同时，</font><font size = 3 color= red>交换机 B</font> 收到这个广播消息之后，由于它知道</font><font size = 3 color= red>机器 1</font> 是不在右边这个网口的，所以不会将消息广播到局域网三。</p>

### 如何解决常见的环路问题？

<p>这样看起来，两台交换机工作得非常好。随着办公室越来越大，交换机数目肯定越来越多。当整个拓扑结构复杂了，这么多网线，绕过来绕过去，不可避免地会出现一些意料不到的情况。其中常见的问题就是</font><font size = 3 color= red>环路问题</font>。</p>
<p>例如这个图，当两个交换机将两个局域网同时连接起来的时候。你可能会觉得，这样反而有了高可用性。但是却不幸地出现了环路。出现了环路会有什么结果呢？</p>

![image](https://static001.geekbang.org/resource/image/c8/d2/c829b28978c3d9686680e4b62fdf53d2.jpg)

<p>我们来想象一下</font><font size = 3 color= red>机器 1</font> 访问</font><font size = 3 color= red>机器 2</font> 的过程。一开始，</font><font size = 3 color= red>机器 1</font> 并不知道</font><font size = 3 color= red>机器 2</font> 的</font><font size = 3 color= red> MAC</font> 地址，所以它需要发起一个</font><font size = 3 color= red> ARP</font> 的广播。广播到达</font><font size = 3 color= red>机器 2</font>，</font><font size = 3 color= red>机器 2</font> 会把 </font><font size = 3 color= red>MAC</font> 地址返回来，看起来没有这两个交换机什么事情。</p>

<p>但是问题来了，这两个交换机还是都能够收到广播包的。<font size = 3 color= red>交换机 A</font> 一开始是不知道<font size = 3 color= red>机器 2</font> 在哪个局域网的，所以它会把广播消息放到局域网二，在局域网二广播的时候，<font size = 3 color= red>交换机 B右边</font>这个网口也是能够收到广播消息的。交换机 B 会将这个广播息信息发送到局域网一。局域网一的这个广播消息，又会到达<font size = 3 color= red>交换机 A 左边</font>的这个接口。交换机 A 这个时候还是不知道<font size = 3 color= red>机器 2</font> 在哪个局域网，于是将广播包又转发到局域网二。左转左转左转，好像是个圈哦。 </p>

<p>可能有人会说，当两台交换机都能够逐渐学习到拓扑结构之后，是不是就可以了？</p>

<p>别想了，压根儿学不会的。<font size = 3 color= red>机器 1</font> 的广播包到达<font size = 3 color= red>交换机 A</font> 和<font size = 3 color= red>交换机 B</font> 的时候，本来两个<font size = 3 color= red>交换机</font>都学会了<font size = 3 color= red>机器 1</font> 是<font size = 3 color= red>在局域网一</font>的，但是当<font size = 3 color= red>交换机 A</font> 将包广播到局域网二之后，<font size = 3 color= red>交换机 B 右边</font>的网口收到了来自<font size = 3 color= red>交换机 A</font> 的广播包。根据学习机制，这<font size = 3 color= red>彻底损坏了交换机 B</font> 的三观，刚才<font size = 3 color= red>机器 1 </font>还在<font size = 3 color= red>左边</font>的网口呢，怎么又出现在右边的网口呢？哦，那肯定是<font size = 3 color= red>机器 1</font> 换位置了，于是就误会了，<font size = 3 color= red>交换机 B</font> 就学会了，<font size = 3 color= red>机器 1</font> 是从<font size = 3 color= red>右边</font>这个网口来的，把刚才学习的那一条清理掉。同理，<font size = 3 color= red>交换机 A 右边</font>的网口，也能收到<font size = 3 color= red>交换机 B </font>转发过来的<font size = 3 color= red>广播包</font>，同样也误会了，于是也学会了，<font size = 3 color= red>机器 1 </font>从<font size = 3 color= red>右边</font>的网口来，<font size = 3 color= red>不是</font>从<font size = 3 color= red>左边</font>的网口来。</p>

<p>然而当广播包从左边的局域网一广播的时候，两个交换机再次刷新三观，原来机器 1 是在左边的，过一会儿，又发现不对，是在右边的，过一会，又发现不对，是在左边的。</p>

<p>这还是一个包转来转去，每台机器都会发广播包，交换机转发也会复制广播包，当广播包越来越多的时候，按照上一节讲过一个共享道路的算法，也就是路会越来越堵，最后谁也别想走。所以，必须有一个方法解决环路的问题，<font size = 3 color= red>怎么破除环路呢？</font></p>

### STP协议中那些难以理解的概念

![image](http://img.027cgb.com/606599/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/1_131946719650625000.png)

![image](http://img.027cgb.com/606599/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/2_131946719652656250.png)

### STP 的工作过程是怎样的？

![image](http://img.027cgb.com/606599/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/3_131946720727500000.png)

![image](http://img.027cgb.com/606599/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/4_131946720730312500.png)

![image](http://img.027cgb.com/606599/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/5_131946720731718750.png)

#### 情形一：掌门遇到掌门

![image](http://img.027cgb.com/606599/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/1_131946722228125000.png)

#### 情形二：同门相遇

![image](http://img.027cgb.com/606599/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/2_131946723646406250.png)

![image](http://img.027cgb.com/606599/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/3_131946723648437500.png)

#### 情形三：掌门与其他帮派小弟相遇

![image](http://img.027cgb.com/606599/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/4_131946724181093750.png)

#### 情形四：不同门小弟相遇

![image](http://img.027cgb.com/606599/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/5_131946724703750000.png)
![image](http://img.027cgb.com/606599/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/6_131946724705781250.png)

### 小结

- 当交换机的数目越来越多的时候，会遭遇环路问题，让网络包迷路，这就需要使用 STP 协议，通过华山论剑比武的方式，将有环路的图变成没有环路的树，从而解决环路问题。
- 交换机数目多会面临隔离问题，可以通过 VLAN 形成虚拟局域网，从而解决广播问题和安全问题。

STP 协议能够很好的解决环路问题，但是也有它的缺点，你能举几个例子吗？<br/>

STP 对于跨地域甚至跨国组织的网络支持，就很难做了，计算量摆着呢。