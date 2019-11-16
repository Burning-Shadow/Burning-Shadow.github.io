---
title: 七、ICMP与ping：投石问路的侦察兵
date: 2019-03-30 21:35:06
tags: [计算机网络]
categories: 网络协议
---

无论是在宿舍，还是在办公室，或者运维一个数据中心，我们常常会遇到网络不通的问题。那台机器明明就在那里，你甚至都可以通过机器的终端连上去看。它看着好好的，可是就是连不上去，究竟是哪里出了问题呢？

<!--more-->

### ICMP 协议的格式

这个时候呢，大家可能会想到ping一下，那你知道ping是怎么工作的吗？

<p>ping 是基于 ICMP 协议工作的。<font size = 3 color= red>ICMP</font>全称<font size = 3 color= red>Internet Control Message Protocol</font>，就是<font size = 3 color= red>互联网控制报文协议</font>。这里面的关键词是<font size = 3 color= red>“控制”</font>，那具体是怎么控制的呢？</p>
<p>网络包在异常复杂的网络环境中传输时，常常会遇到各种各样的问题。当遇到问题的时候，总不能“死个不明不白”，要传出消息来，报告情况，这样才可以调整传输策略。这就相当于我们经常看到的电视剧里，古代行军的时候，为将为帅者需要通过侦察兵、哨探或传令兵等人肉的方式来掌握情况，控制整个战局。 </p>
<p><font size = 3 color= red>ICMP 报文</font>是封装在 <font size = 3 color= red>IP</font> 包里面的。因为传输指令的时候，肯定需要<font size = 3 color= red>源地址</font>和<font size = 3 color= red>目标地址</font>。它本身非常简单。因为作为侦查兵，要轻装上阵，不能携带大量的包袱。</p>
![image](https://static001.geekbang.org/resource/image/23/ff/23aecf653d60dd94b7c5c6dc21ca21ff.jpg)

<p>ICMP 报文有很多的类型，不同的类型有不同的代码。<font size = 3 color= red>最常用的类型是主动请求为 8，主动请求的应答为 0</font>。</p>
### 查询报文类型

<p>我们经常在电视剧里听到这样的话：主帅说，来人哪！前方战事如何，快去派人打探，一有情况，立即通报！</p>
<p>这种是主帅发起的，主动查看敌情，对应 ICMP 的<font size = 3 color= red>查询报文类型</font>。例如，常用的<font size = 3 color= red>ping 就是查询报文，是一种主动请求，并且获得主动应答的 ICMP 协议。</font>所以，ping 发的包也是符合 ICMP 协议格式的，只不过它在后面增加了自己的格式。</p>
<p>对 ping 的主动请求，进行网络抓包，称为<font size = 3 color= red>ICMP ECHO REQUEST。</font>同理主动请求的回复，称为<font size = 3 color= red>ICMP ECHO REPLY</font>。比起原生的 ICMP，这里面多了两个字段，一个是<font size = 3 color= red>标识符</font>。这个很好理解，你派出去两队侦查兵，一队是侦查战况的，一队是去查找水源的，要有个标识才能区分。另一个是<font size = 3 color= red>序号</font>，你派出去的侦查兵，都要编个号。如果派出去 <font size = 3 color= red>10</font> 个，回来 <font size = 3 color= red>10</font> 个，就说明前方战况不错；如果派出去 <font size = 3 color= red>10</font> 个，回来 <font size = 3 color= red>2</font> 个，说明情况可能不妙。</p>
<p>在选项数据中，<font size = 3 color= red>ping</font> 还会存放发送请求的时间值，来计算往返时间，说明路程的长短。</p>
### 差错报文类型

<p>当然也有另外一种方式，就是<font size = 3 color= red>差错报文</font>。</p>
<p>主帅骑马走着走着，突然来了一匹快马，上面的小兵气喘吁吁的：报告主公，不好啦！张将军遭遇埋伏，全军覆没啦！这种是异常情况发起的，来报告发生了不好的事情，对应 <font size = 3 color= red>ICMP</font> 的<font size = 3 color= red>差错报文类型</font>。</p>
<p>我举几个 <font size = 3 color= red>ICMP 差错报文</font>的例子：<font size = 3 color= red>终点不可达为 3</font>，<font size = 3 color= red>源抑制为 4</font>，<font size = 3 color= red>超时为 11</font>，<font size = 3 color= red>重定向为 5</font>。这些都是什么意思呢？我给你具体解释一下。</p>
#### 第一种是终点不可达

小兵：报告主公，您让把粮草送到张将军那里，结果没有送到。

<p>如果你是主公，你肯定会问，为啥送不到？具体的原因在代码中表示就是，<font size = 3 color= red>网络不可达代码为 0</font>，<font size = 3 color= red>主机不可达代码为 1</font>，<font size = 3 color= red>协议不可达代码为 2</font>，<font size = 3 color= red>端口不可达代码为 3</font>，<font size = 3 color= red>需要进行分片但设置了不分片位代码为 4</font>。</p>
<p>具体的场景就像这样：</p>
- 网络不可达：主公，找不到地方呀？
- 主机不可达：主公，找到地方没这个人呀？
- 协议不可达：主公，找到地方，找到人，口号没对上，人家天王盖地虎，我说 12345！
- 端口不可达：主公，找到地方，找到人，对了口号，事儿没对上，我去送粮草，人家说他们在等救兵。
- 需要进行分片但设置了不分片位：主公，走到一半，山路狭窄，想换小车，但是您的将令，严禁换小车，就没办法送到了。

#### 第二种是源站抑制

也就是让源站放慢发送速度。小兵：报告主公，您粮草送的太多了吃不完。

#### 第三种是时间超时

也就是超过网络包的生存时间还是没到。小兵：报告主公，送粮草的人，自己把粮草吃完了，还没找到地方，已经饿死啦。

#### 第四种是路由重定向

也就是让下次发给另一个路由器。小兵：报告主公，上次送粮草的人本来只要走一站地铁，非得从五环绕，下次别这样了啊。

<p>差错报文的结构相对复杂一些。除了前面还是 IP，ICMP 的前 8 字节不变，后面则跟上出错的那个 <font size = 3 color= red>IP </font>包的 <font size = 3 color= red>IP </font>头和 <font size = 3 color= red>IP 正文</font>的前 8 个字节。</p>
<p>而且这类侦查兵特别恪尽职守，不但自己返回来报信，还把一部分遗物也带回来。</p>
- 侦察兵：报告主公，张将军已经战死沙场，这是张将军的印信和佩剑。
- 主公：神马？张将军是怎么死的（可以查看 <font size = 3 color= red>ICMP 的前 8 字节</font>）？没错，这是张将军的剑，是他的剑（<font size = 3 color= red>IP 数据包的头</font>及<font size = 3 color= red>正文前 8 字节</font>）。

### ping：查询报文类型的使用

<p>接下来，我们重点来看 ping 的发送和接收过程。</p>
![image](https://static001.geekbang.org/resource/image/e5/fc/e5270427819fc51c88e81a5c1cc4b8fc.jpg)

<p>假定<font size = 3 color= red>主机 A</font> 的 <font size = 3 color= red>IP</font> 地址是 <font size = 3 color= red>192.168.1.1</font>，<font size = 3 color= red>主机 B</font> 的 <font size = 3 color= red>IP</font> 地址是 <font size = 3 color= red>192.168.1.2</font>，它们都在同一个子网。那当你在<font size = 3 color= red>主机 A</font> 上运行<font size = 3 color= red>“ping 192.168.1.2”</font>后，会发生什么呢?</p>
<p><font size = 3 color= red>ping</font> 命令执行的时候，源主机首先会构建一个 <font size = 3 color= red>ICMP</font> 请求数据包，<font size = 3 color= red>ICMP</font> 数据包内包含多个字段。最重要的是两个，第一个是<font size = 3 color= red>类型字段</font>，对于请求数据包而言该字段为 <font size = 3 color= red>8</font>；另外一个是<font size = 3 color= red>顺序号</font>，主要用于区分连续 <font size = 3 color= red>ping</font> 的时候发出的多个数据包。每发出一个请求数据包，顺序号会自动加<font size = 3 color= red> 1</font>。为了能够计算往返时间<font size = 3 color= red> RTT</font>，它会在报文的数据部分插入发送时间。</p>
<p>然后，由 <font size = 3 color= red>ICMP</font> 协议将这个数据包连同地址 <font size = 3 color= red>192.168.1.2</font> 一起交给 <font size = 3 color= red>IP </font>层。<font size = 3 color= red>IP</font> 层将以 <font size = 3 color= red>192.168.1.2 </font>作为目的地址，<font size = 3 color= red>本机 IP</font> 地址作为源地址，加上一些其他控制信息，<font size = 3 color= red>构建</font>一个 <font size = 3 color= red>IP 数据包</font>。</p>
<p>接下来，需要加入 <font size = 3 color= red>MAC</font> 头。如果在本节 <font size = 3 color= red>ARP</font> 映射表中查找出 <font size = 3 color= red>IP</font> 地址 <font size = 3 color= red>192.168.1.2</font> 所对应的<font size = 3 color= red> MAC</font> 地址，则可以直接使用；如果没有，则需要发送 <font size = 3 color= red>ARP</font> 协议查询 <font size = 3 color= red>MAC 地址</font>，获得<font size = 3 color= red> MAC 地址后</font>，由<font size = 3 color= red>数据链路层</font>构建一个数据帧，目的地址是 <font size = 3 color= red>IP</font> 层传过来的 <font size = 3 color= red>MAC</font> 地址，源地址则是本机的<font size = 3 color= red> MAC</font> 地址；还要附加上一些控制信息，依据以太网的介质访问规则，将它们传送出去。</p>
<p><font size = 3 color= red>主机 B</font> 收到这个数据帧后，先检查它的目的 <font size = 3 color= red>MAC</font> 地址，并和<font size = 3 color= red>本机的 MAC</font> 地址对比，如符合，则接收，否则就丢弃。接收后检查该数据帧，将 <font size = 3 color= red>IP</font> 数据包从帧中提取出来，交给本机的 <font size = 3 color= red>IP</font> 层。同样，<font size = 3 color= red>IP</font> 层检查后，将有用的信息提取后交给 <font size = 3 color= red>ICMP 协议</font>。</p>
<p><font size = 3 color= red>主机 B</font> 会构建一个 <font size = 3 color= red>ICMP</font> 应答包，<font size = 3 color= red>应答数据包</font>的<font size = 3 color= red>类型</font>字段为<font size = 3 color= red> 0</font>，顺序号为接收到的请求数据包中的顺序号，然后再发送出去给<font size = 3 color= red>主机 A</font>。</p>
<p>在规定的时候间内，源主机如果没有接到 <font size = 3 color= red>ICMP</font> 的应答包，则说明目标主机不可达；如果接收到了 <font size = 3 color= red>ICMP</font> 应答包，则说明目标主机可达。此时，源主机会检查，用当前时刻减去该数据包最初从源主机上发出的时刻，就是 <font size = 3 color= red>ICMP</font> 数据包的时间延迟。</p>
<p>当然这只是最简单的，同一个局域网里面的情况。如果跨网段的话，还会涉及网关的转发、路由器的转发等等。但是对于 <font size = 3 color= red>ICMP</font> 的头来讲，是没什么影响的。会影响的是根据<font size = 3 color= red>目标 IP 地址</font>，选择路由的下一跳，还有每经过一个路由器到达一个新的局域网，需要<font size = 3 color= red>换 MAC 头</font>里面的 <font size = 3 color= red>MAC 地址</font>。这个过程后面几节会详细描述，这里暂时不多说。</p>
<font size = 3 >

<p>如果在自己的可控范围之内，当遇到网络不通的问题的时候，除了直接 <font size = 3 color= red>ping</font> 目标的 <font size = 3 color= red>IP</font> 地址之外，还应该有一个清晰的网络拓扑图。并且从理论上来讲，应该要清楚地知道一个网络包从源地址到目标地址都需要经过哪些设备，然后逐个 <font size = 3 color= red>ping</font> 中间的这些设备或者机器。如果可能的话，在这些关键点，通过 <font size = 3 color= red>tcpdump -i eth0 icmp</font>，查看包有没有到达某个点，回复的包到达了哪个点，可以更加容易推断出错的位置。</p>
</font>

<p>经常会遇到一个问题，如果不在我们的控制范围内，很多中间设备都是禁止 <font size = 3 color= red>ping</font> 的，但是 <font size = 3 color= red>ping</font> 不通不代表网络不通。这个时候就要使用 <font size = 3 color= red>telnet</font>，通过其他协议来测试网络是否通，这个就不在本篇的讲述范围了。</p>
<p>说了这么多，你应该可以看出 <font size = 3 color= red>ping</font> 这个程序是使用了 <font size = 3 color= red>ICMP</font> 里面的 <font size = 3 color= red>ECHO REQUEST</font> 和 <font size = 3 color= red>ECHO REPLY</font> 类型的。</p>
### Traceroute：差错报文类型的使用

<p>那其他的类型呢？是不是只有真正遇到错误的时候，才能收到呢？那也不是，有一个程序 <font size = 3 color= red>Traceroute</font>，是个<font size = 3 color= red>“大骗子”</font>。它会使用 <font size = 3 color= red>ICMP</font> 的规则，故意制造一些能够产生错误的场景。</p>
<p>所以，<strong>Traceroute 的第一个作用就是故意设置特殊的 TTL，来追踪去往目的地时沿途经过的路由器</strong>。<font size = 3 color= red>Traceroute</font> 的参数指向某个目的 <font size = 3 color= red>IP</font> 地址，它会发送一个<font size = 3 color= red> UDP</font> 的数据包。将 <font size = 3 color= red>TTL</font> 设置成<font size = 3 color= red> 1</font>，也就是说一旦遇到一个路由器或者一个关卡，就表示它<font size = 3 color= red>“牺牲”</font>了。</p>
> TTL是 Time To Live的缩写，该字段指定IP包被路由器丢弃之前允许通过的最大网段数量。TTL是IPv4包头的一个8 bit字段

<p>如果中间的路由器不止一个，当然碰到第一个就<font size = 3 color= red>“牺牲”</font>。于是，返回一个 <font size = 3 color= red>ICMP</font> 包，也就是网络差错包，类型是时间超时。那大军前行就带一顿饭，试一试走多远会被饿死，然后找个哨探回来报告，那我就知道大军只带一顿饭能走多远了。</p>
<p>接下来，将 <font size = 3 color= red>TTL</font> 设置为 2。第一关过了，第二关就<font size = 3 color= red>“牺牲”</font>了，那我就知道第二关有多远。如此反复，直到到达目的主机。这样，<font size = 3 color= red>Traceroute</font> 就拿到了所有的路由器 <font size = 3 color= red>IP</font>。当然，有的路由器压根不会回这个 <font size = 3 color= red>ICMP</font>。这也是 <font size = 3 color= red>Traceroute</font> 一个公网的地址，看不到中间路由的原因。</p>
<p>怎么知道 <font size = 3 color= red>UDP</font> 有没有到达目的主机呢？<font size = 3 color= red>Traceroute</font> 程序会发送一份 <font size = 3 color= red>UDP</font> 数据报给目的主机，但它会<font size = 3 color= red>选择一个不可能的值</font>作为 <font size = 3 color= red>UDP 端口号（大于 30000）</font>。当该数据报到达时，将使目的主机的 <font size = 3 color= red>UDP</font> 模块<font size = 3 color= red>产生</font>一份<font size = 3 color= red>“端口不可达”错误</font> ICMP 报文。如果数据报没有到达，则可能是超时。</p>
<p>这就相当于故意派人去西天如来那里去请一本《道德经》，结果人家信佛不信道，消息就会被打出来。被打的消息传回来，你就知道西天是能够到达的。为什么不去取《心经》呢？因为 UDP 是无连接的。也就是说这人一派出去，你就得不到任何音信。你无法区别到底是半路走丢了，还是真的信佛遁入空门了，只有让人家打出来，你才会得到消息。</p>
<p><strong>Traceroute 还有一个作用是故意设置不分片，从而确定路径的 MTU。</strong>要做的工作首先是发送分组，并设置“不分片”标志。发送的第一个分组的长度正好与出口 MTU 相等。如果中间遇到窄的关口会被卡住，会发送 ICMP 网络差错包，类型为“需要进行分片但设置了不分片位”。其实，这是人家故意的好吧，每次收到 ICMP“不能分片”差错时就减小分组的长度，直到到达目标主机。</p>
### 小结

- ICMP 相当于网络世界的侦察兵。我讲了两种类型的 ICMP 报文，一种是主动探查的查询报文，一种异常报告的差错报文；
- ping 使用查询报文，Traceroute 使用差错报文。



