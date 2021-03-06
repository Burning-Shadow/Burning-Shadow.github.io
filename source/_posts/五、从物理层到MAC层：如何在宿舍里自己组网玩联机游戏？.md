---
title: 五、从物理层到MAC层：如何在宿舍里自己组网玩联机游戏？
date: 2019-03-30 21:35:04
tags: [计算机网络]
categories: 网络协议
---

之前我们见证了IP的诞生，一旦机器有了IP，就可以在网络的环境里和其他机器展开沟通了

宿舍六个人，两个人买了电脑，那两台电脑能不能连接起来呢？不会说，当然啊，买个路由器不就行了吗

现在一台家用路由器非常便宜，一百多块的事情。但是在早期的时候绝对是奢侈品

<!--more-->

### 第一层（物理层）

使用路由器是第三层的事，我们先从第一层开始说。

物理层能折腾啥呢？？？

现在可能感受不到，早些年的时候去学校配电脑的地方买网线，卖网线的师傅都会问，你的网线是要电脑连电脑啊，还是电脑连网口啊？

我们要的是电脑连电脑。这种方式就是一根网线，有两个头。一头插在一台电脑的网卡上，另一头插在另一台电脑的网卡上。但是当时，普通的网线这样是通不了的，所以水晶头要做交叉线，用的就是<font size =3 >**1 - 3 ，2 - 6 交叉解法**</font>

水晶头的 1、2 和第 3、6 脚，分别起着收、发信号的作用。将一段的 1 号 和 3 号线、2 号和 6 号线互换一下位置，就能够在物理层实现一端发送的信号，另一端能收到。

<p>当然电脑连电脑，除了网线要交叉，还需要配置这两台电脑的 <font size = 3 color= red>IP</font> 地址、<font size = 3 color= red>子网掩码</font>和<font size = 3 color= red>默认网关</font>。这三个概念上一节详细描述过了。要想两台电脑能够通信，这三项必须配置成为一个网络，可以一个是<font size = 3 color= red> 192.168.0.1/24</font>，另一个是<font size = 3 color= red> 192.168.0.2/24</font>，否则是不通的。</p>
<p>这里我想问你一个问题，两台电脑之间的网络包，包含 MAC 层吗？当然包含，要完整。<font size = 3 color= red>IP 层要封装了 MAC 层才能将包放入物理层。</font></p>
到了现在，这两台电脑已经构成了一个最小的局域网，也即LAN。可以玩局域网游戏了。

那么等第三个哥们也买了电脑，怎么把三个电脑连接起来呢？？？

<p>先别说交换机，当时交换机也贵。有一个叫作<font size = 3 color= red>Hub</font>的东西，也就是<font size = 3 color= red>集线器</font>。这种设备有多个口，可以将宿舍里的多台电脑连接起来。但是，和交换机不同，<font size = 3 color= red>集线器没有大脑，它完全在物理层工作。</font>它会将自己收到的每一个字节，<font size = 3 color= red>都复制到其他端口上去</font>。这是第一层物理层联通的方案。</p>
### 第二层（数据链路层）

<p>你可能已经发现问题了。<font size = 3 color= red>Hub 采取的是广播的模式</font>，如果每一台电脑发出的包，宿舍的每个电脑都能收到，那就麻烦了。这就需要解决几个问题：</p>
1. 这个包是发给谁的？谁应该接收？
2. 大家都在发，会不会产生混乱？有没有谁先发、谁后发的规则？
3. 如果发送的时候出现了错误，怎么办？

<p>这几个问题，都是第二层，数据链路层，也即 MAC 层要解决的问题。

#### 第二个问题解决

<font size = 3 color= red>MAC</font>的全称是<font size = 3 color= red>Medium Access Control</font>，即<font size = 3 color= red>媒体访问控制。</font>控制什么呢？其实就是控制在往媒体上发数据的时候，谁先发、谁后发的问题。防止发生混乱。<font size = 3 color= red>这解决的是第二个问题</font>。这个问题中的规则，学名叫<font size = 3 color= red>多路访问</font>。有很多算法可以解决这个问题。就像车管所管束马路上跑的车，能想的办法都想过了。</p>

<p>比如接下来这三种方式：</p>
<ul>
<li><p>方式一：分多个车道。每个车一个车道，你走你的，我走我的。这在计算机网络里叫作<font size = 3 color= red>信道划分；</font></p>
</li>
<li><p>方式二：今天单号出行，明天双号出行，轮着来。这在计算机网络里叫作<font size = 3 color= red>轮流协议；</font></p>
</li>
<li><p>方式三：不管三七二十一，有事儿先出门，发现特堵，就回去。错过高峰再出。我们叫作<font size = 3 color= red>随机接入协议。</font>著名的<font size = 3 color= red>以太网</font>，用的就是这个方式。</p>
</li>
</ul>

#### 第一个问题解决

<p>解决了第二个问题，就是解决了媒体接入控制的问题，MAC 的问题也就解决好了。这和 MAC 地址没什么关系。</p>
<p><font size = 3 color= red>接下来要解决第一个问题</font>：发给谁，谁接收？这里用到一个物理地址，叫作<font size = 3 color= red>链路层地址。</font>但是因为第二层主要解决媒体接入控制的问题，所以它常被称为<font size = 3 color= red>MAC 地址</font>。</p>
<p>解决第一个问题就牵扯到第二层的网络包<font size = 3 color= red>格式</font>。对于以太网，第二层的最开始，就是目标的 MAC 地址和源的 MAC 地址。</p>
![image](https://static001.geekbang.org/resource/image/ce/ed/cef93d665ca863fef40f7f854d5d33ed.jpg)

<p>接下来是<strong>类型</strong>，大部分的类型是 IP 数据包，然后 IP 里面包含 TCP、UDP，以及 HTTP 等，这都是里层封装的事情。</p>
<p>有了这个目标 MAC 地址，数据包在链路上广播，MAC 的网卡才能发现，这个包是给它的。MAC 的网卡把包收进来，然后打开 IP 包，发现 IP 地址也是自己的，再打开 TCP 包，发现端口是自己，也就是 80，而 nginx 就是监听 80。</p>
<p>于是将请求提交给 nginx，nginx 返回一个网页。然后将网页需要发回请求的机器。然后层层封装，最后到 MAC 层。因为来的时候有源 MAC 地址，返回的时候，源 MAC 就变成了目标 MAC，再返给请求的机器。</p>
<p>对于以太网，第二层的最后面是<font size = 3 color= red>CRC</font>，也就是<font size = 3 color= red>循环冗余检测</font>。通过 <font size = 3 color= red>XOR 异或</font>的算法，来计算整个包是否在发送的过程中出现了错误，<font size = 3 color= red>主要解决第三个问题</font>。</p>
<p>这里还有一个没有解决的问题，当源机器知道目标机器的时候，可以将目标地址放入包里面，如果不知道呢？一个广播的网络里面接入了 N 台机器，<font size = 3 color= red>我怎么知道每个 MAC 地址是谁呢</font>？这就是<font size = 3 color= red>ARP 协议</font>，也就是<font size = 3 color= red>已知 IP 地址，求 MAC 地址的协议。</font></p>
![image](https://static001.geekbang.org/resource/image/17/3d/17ac2f46ef531e2b4380300f10267e3d.jpg)

<p>在一个局域网里面，当知道了 IP 地址，不知道 MAC 怎么办呢？靠“吼”。</p>
![image](https://static001.geekbang.org/resource/image/5f/68/5fe88a40a8b5d507601968efb50ac668.jpg)

<p>广而告之，发送一个广播包，谁是这个 IP 谁来回答。具体询问和回答的报文就像下面这样：</p>
![image](https://static001.geekbang.org/resource/image/2b/2f/2bc53afb25515e96d0e646e297b1ce2f.jpg)

<p>为了避免每次都用 ARP 请求，机器本地也会进行 ARP 缓存。当然机器会不断地上线下线，IP 也可能会变，所以 ARP 的 MAC 地址缓存过一段时间就会过期。</p>
### 局域网

<p>好了，至此我们宿舍四个电脑就组成了一个局域网。用 Hub 连接起来，就可以玩局域网版的《魔兽争霸》了。</p>
![image](https://static001.geekbang.org/resource/image/33/ac/33d180e376439ca10e3f126eb2e36bac.jpg)

<p>这种组网的方法，对一个宿舍来说没有问题，但是一旦机器数目增多，问题就出现了。因为<font size = 3 color= red> Hub 是广播的</font>，不管某个接口是否需要，<font size = 3 color= red>所有的 Bit 都会被发送出去</font>，然后<font size = 3 color= red>让主机来判断</font>是不是需要。这种方式路上的车少就没问题，车一多，产生冲突的概率就提高了。而且把不需要的包转发过去，纯属浪费。看来 <font size = 3 color= red>Hub</font> 这种不管三七二十一都转发的设备是不行了，需要点儿智能的。因为每个口都只连接一台电脑，这台电脑又不怎么换<font size = 3 color= red> IP</font> 和 <font size = 3 color= red>MAC </font>地址，只要记住这台电脑的 <font size = 3 color= red>MAC</font> 地址，如果<font size = 3 color= red>目标 MAC</font> 地址不是这台电脑的，这个口就不用转发了。</p>
<p>谁能知道目标 MAC 地址是否就是连接某个口的电脑的 MAC 地址呢？这就需要一个能把 MAC 头拿下来，检查一下目标 MAC 地址，然后根据策略转发的设备，按第二节课中讲过的，这个设备显然是个二层设备，我们称为<font size = 3 color= red>交换机</font>。</p>
<p>交换机怎么知道每个口的电脑的<font size = 3 color= red> MAC</font> 地址呢？这需要交换机会学习。</p>
<p>一台 <font size = 3 color= red>MAC1</font> 电脑将一个包发送给另一台 <font size = 3 color= red>MAC2</font> 电脑，当这个包到达交换机的时候，一开始交换机也不知道 <font size = 3 color= red>MAC2</font> 的电脑在哪个口，所以没办法，它只能将包转发给除了来的那个口之外的其他所有的口。<font size = 3 color= red>但是，这个时候，交换机会干一件非常聪明的事情，就是交换机会记住，MAC1 是来自一个明确的口。</font>以后有包的目的地址是 <font size = 3 color= red>MAC1</font> 的，直接发送到这个口就可以了。</p>
<p>当交换机作为一个关卡一样，过了一段时间之后，就有了整个网络的一个结构了，这个时候，基本上不用广播了，全部可以准确转发。当然，每个机器的<font size = 3 color= red> IP</font> 地址会变，所在的口也会变，因而交换机上的学习的结果，我们称为<font size = 3 color= red>转发表</font>，是<font size = 3 color= red>有</font>一个<font size = 3 color= red>过期时间</font>的。</p>
### 小结

- MAC 层是用来解决多路访问的堵车问题的；
- ARP 是通过吼的方式来寻找目标 MAC 地址的，吼完之后记住一段时间，这个叫作缓存；
- 交换机是有 MAC 地址学习能力的，学完了它就知道谁在哪儿了，不用广播了。