---
title: 三、ifconfig：最熟悉又陌生的命令行
tags: [计算机网络]
categories: 网络协议
---

### 怎么查看 IP 地址吗？

windows上：<font size = 3 color= red>ipconfig</font>

Linux上：<font size = 3 color= red>ifconfig，ip addr</font>

那么<font size = 3 color= red>ipconfig</font>和<font size = 3 color= red>ip addr</font>的区别吗？？？这是一个有关 <font size = 3 color= red>net-tools</font> 和 <font size = 3 color= red>iproute2</font>的"历史"故事，先不说这个，不过这个也是常考的点

<!--more-->

运行一下<font size = 3 color= red>ip addr
</font>，会显示如下结果

```
root@test:~# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether fa:16:3e:c7:79:75 brd ff:ff:ff:ff:ff:ff
    inet 10.100.122.2/24 brd 10.100.122.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fec7:7975/64 scope link 
       valid_lft forever preferred_lft forever

```

这个命令显示了这台机器上的所有网卡。大部分网卡都有一个ip地址，当然这不是必须的，后面我们会遇到<font size = 3 color= red>没有ip地址的情况</font>

<p><font size = 3 color="orange">IP 地址是一个网卡在网络世界的通讯地址，相当于我们现实世界的门牌号码。</font>既然是门牌号码，不能大家都一样，不然就会起冲突。比方说，假如大家都叫六单元 1001 号，那快递就找不到地方了。所以，有时候咱们的电脑弹出网络地址冲突，出现上不去网的情况，多半是 IP 地址冲突了。</p>
<p>如上输出的结果，<font size = 3 color= red>10.100.122.2</font> 就是一个 <font size = 3 color= red>IP</font> 地址。这个地址被点分隔为四个部分，每个部分 <font size = 3 color= red>8 个 bit</font>，所以 <font size = 3 color= red>IP</font> 地址总共是 <font size = 3 color= red>32</font> 位。这样产生的 <font size = 3 color= red>IP</font> 地址的数量很快就不够用了。因为当时设计 <font size = 3 color= red>IP</font> 地址的时候，哪知道今天会有这么多的计算机啊！因为不够用，于是就有了 <font size = 3 color= red>IPv6</font>，也就是上面输出结果里面 <font size = 3 color= red>inet6 fe80::f816:3eff:fec7:7975/64</font>。这个有 <font size = 3 color= red>128</font> 位，现在看来是够了，但是未来的事情谁知道呢？</p>
<p>本来 32 位的 IP 地址就不够，还被分成了 5 类。现在想想，当时分配地址的时候，真是太奢侈了。</p>

![image](https://static001.geekbang.org/resource/image/0b/9e/0b32d6e35ff0bbc5d46cfb87f6669d9e.jpg)

![image](http://img.027cgb.com/606599/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/1_131944345515311268.png)

![image](http://img.027cgb.com/606599/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/3_131944345520467518.png)

> 这里的话，我自己是这样理解的，画一条线，从0开始到255，中间的数就是127,127之前的就是A类地址，如下

![image](http://img.027cgb.com/606599/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/4_131944347895780018.png)

在当时设计IP地址的时候，对于<font size = 3 color= red>A、B、C</font>类主要分为两部分，前面是网络号，后面是主机号。这很好理解，大家都是六单元 1001 号，我是小区 A （<font size = 3 color= red>网络号</font>）的六单元 1001 号（<font size = 3 color= red>主机号</font>），而你是小区 B （<font size = 3 color= red>网络号</font>）的六单元 1001 号（<font size = 3 color= red>主机号</font>）。

下面这个表格，详细地展示了 A、B、C 三类地址所能包含的主机的数量。

![image](https://static001.geekbang.org/resource/image/e9/be/e9c59a4b2f0b804356759b10440ea7be.jpg)

![image](http://img.027cgb.com/606599/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/2_131944345518748768.png)

<p>这里面有个尴尬的事情，就是 C 类地址能包含的最大主机数量实在太少了，只有 254 个。当时设计的时候恐怕没想到，现在估计一个网吧都不够用吧。而 B 类地址能包含的最大主机数量又太多了。6 万多台机器放在一个网络下面，一般的企业基本达不到这个规模，闲着的地址就是浪费。</p>

### 无类型域间选路（CIDR）

<p>于是有了一个折中的方式叫作<font size = 3 color = red>无类型域间选路</font>，简称<font size = 3 color = red>CIDR</font>。这种方式打破了原来设计的几类地址的做法，将 32 位的 IP 地址一分为二，前面是<font size = 3 color = red>网络号</font>，后面是<font size = 3 color = red>主机号</font>。从哪里分呢？你如果注意观察的话可以看到，10.100.122.2/24，这个 IP 地址中有一个斜杠，斜杠后面有个数字 24。这种地址表示形式，就是 CIDR。后面 24 的意思是，32 位中，前 24 位是网络号，后 8 位是主机号。</p>

**就是子网划分了，比如C类地址，有255个私有地址，现在公司让你将这一个C类地址划分为两个网段的地址，本来是从一个路由器上接的，现在就需要三个路由器。之前都是8位8位的划分网络号和主机号，现在我不管了，比如C类地址是前24位为网络号，我要将这一个C类地址划分为两个网段的地址，那么我就将主机号后移一位，划分为4个就需要将主机号后移2位。**

<p>伴随着 CIDR 存在的，一个是<strong>广播地址</strong>，10.100.122.255。如果发送这个地址，所有 10.100.122 网络里面的机器都可以收到。另一个是<strong>子网掩码</strong>，255.255.255.0。</p>

> 广播地址就是主机号转换为二进制都是1，8位2进制转换为1的10进制数就是255

<p>将子网掩码和 IP 地址进行 AND 计算。前面三个 255，转成二进制都是 1。1 和任何数值取 AND，都是原来数值，因而前三个数不变，为 10.100.122。后面一个 0，转换成二进制是 0，0 和任何数值取 AND，都是 0，因而最后一个数变为 0，合起来就是 10.100.122.0。这就是<font size = 3 color = red>网络号</font>。<font size = 3 color = red>将子网掩码和 IP 地址按位计算 AND，就可得到网络号。</font></p>

### 公有 IP 地址和私有 IP 地址

<p>在日常的工作中，几乎不用划分 A 类、B 类或者 C 类，所以时间长了，很多人就忘记了这个分类，而只记得 CIDR。但是有一点还是要注意的，就是公有 IP 地址和私有 IP 地址。</p>

![image](https://static001.geekbang.org/resource/image/90/93/901778433f2d6e27b916e9e53c232d93.jpg)

<p>我们继续看上面的表格。表格最右列是私有 IP 地址段。平时我们看到的数据中心里，办公室、家里或学校的 IP 地址，一般都是私有 IP 地址段。因为这些地址允许组织内部的 IT 人员自己管理、自己分配，而且可以重复。因此，你学校的某个私有 IP 地址段和我学校的可以是一样的。</p>

> 这就像每个小区有自己的楼编号和门牌号，你们小区可以叫 6 栋，我们小区也叫 6 栋，没有任何问题。但是一旦出了小区，就需要使用公有 IP 地址。就像人民路 888 号，是国家统一分配的，不能两个小区都叫人民路 888 号。

<p>公有 <font size = 3 color = red>IP</font> 地址有个组织统一分配，你需要去买。如果你搭建一个网站，给你学校的人使用，让你们学校的 <font size = 3 color = red>IT</font> 人员给你一个 <font size = 3 color = red>IP</font> 地址就行。但是假如你要做一个类似网易 163 这样的网站，就需要有公有 <font size = 3 color = red>IP</font> 地址，这样全世界的人才能访问。</p>

<p>表格中的 <font size = 3 color = red>192.168.0.x</font> 是最常用的私有 <font size = 3 color = red>IP</font> 地址。你家里有 <font size = 3 color = red>Wi-Fi</font>，对应就会有一个 <font size = 3 color = red>IP</font> 地址。一般你家里地上网设备不会超过 <font size = 3 color = red>256</font> 个，所以 <font size = 3 color = red>/24</font> 基本就够了。有时候我们也能见到 <font size = 3 color = red>/16</font> 的<font size = 3 color = red> CIDR</font>，这两种是最常见的</p>

<p>不需要将十进制转换为二进制 32 位，就能明显看出 <font size = 3 color = red>192.168.0</font> 是<font size = 3 color = red>网络号</font>，后面是<font size = 3 color = red>主机号</font>。而整个网络里面的第一个地址 <font size = 3 color = red>192.168.0.1</font>，往往就是你这个私有网络的出口地址(<font size = 3 color = red>网关</font>)。例如，你家里的电脑连接 Wi-Fi，Wi-Fi 路由器的地址就是 <font size = 3 color = red>192.168.0.1</font>，而 <font size = 3 color = red>192.168.0.255 </font>就是<font size = 3 color = red>广播地址</font>。一旦发送这个地址，整个 <font size = 3 color = red>192.168.0</font> 网络里面的所有机器都能收到。</p>

<p>但是也不总都是这样的情况。因此，其他情况往往就会很难理解，还容易出错。</p>

### 举例：一个容易“犯错”的 CIDR

<font size = 3 color = red>
我们来看 16.158.165.91/22 这个 CIDR。求一下这个网络的第一个地址、子网掩码和广播地址。</font>

<p>你要是上来就写 <font size = 3 color = red>16.158.165.1</font>，那就大错特错了。</p>

<p><font size = 3 color = red>/22</font> 不是 <font size = 3 color = red>8</font> 的<font size = 3 color = red>整数倍</font>，不好办，只能先<font size = 3 color = red>变成二进制</font>来看。<font size = 3 color = red>16.158</font> 的部分不会动，它占了<font size = 3 color = red>前 16 位</font>。中间的<font size = 3 color = red> 165</font>，变为二进制为<font size = 3 color = red>‭10100101</font>‬。除了前面<font size = 3 color = red>的 16</font> 位，还剩<font size = 3 color = red> 6 </font>位。所以，这 8 位中前<font size = 3 color = red> 6 </font>位是网络号，<font size = 3 color = red>16.158.&lt;101001&gt;</font>，而<font size = 3 color = red> &lt;01&gt;.91 </font>是机器号。</p>

<p>第一个地址是<font size = 3 color = red> 16.158.&lt;101001&gt;&lt;00&gt;.1</font>，即<font size = 3 color = red> 16.158.164.1</font>。子网掩码是<font size = 3 color = red> 255.255.&lt;111111&gt;&lt;00&gt;.0</font>，即<font size = 3 color = red> 255.255.252.0</font>。广播地址为<font size = 3 color = red> 16.158.&lt;101001&gt;&lt;11&gt;.255</font>，即<font size = 3 color = red> 16.158.167.255</font>。</p>

> <font size = 3 color = red>看这里的广播地址，是不是将主机号的二进制变为1</font>

<p>这五类地址中，还有一类 D 类是<strong>组播地址</strong>,这个先就不考虑了，后面有时间在说。

我们接着来分析上面 ip addr的结果

### scope

<p>在 <font size = 3 color = red>IP</font> 地址的后面有个 <font size = 3 color = red>scope</font>，对于 <font size = 3 color = red>eth0</font> 这张网卡来讲，是 <font size = 3 color = red>global</font>，说明这张网卡是可以对外的，可以接收来自各个地方的包。对于 <font size = 3 color = red>lo</font> 来讲，是 <font size = 3 color = red>host</font>，说明这张网卡仅仅可以供本机相互通信。</p>

<p>lo 全称是<font size = 3 color = red>loopback</font>，又称<font size = 3 color= red>环回接口</font>，往往会被分配到<font size = 3 color = red> 127.0.0.1 </font>这个地址。这个地址<font size = 3 color = red>用于本机通信</font>，经过内核处理后直接返回，不会在任何网络中出现。</p>

### MAC 地址

<p>在 IP 地址的上一行是 link/ether fa:16:3e:c7:79:75 brd ff:ff:ff:ff:ff:ff，这个被称为<font size = 3 color = red>MAC 地址</font>，是一个网卡的物理地址，用<font size = 3 color = red>十六进制</font>，<font size = 3 color = red>6 个 byte 表示</font>。</p>
<p><font size = 3 color = red>MAC</font> 地址是一个很容易让人“误解”的地址。因为 MAC 地址<font size = 3 color = red>号称全局唯一</font>，不会有两个网卡有相同的 <font size = 3 color = red>MAC</font> 地址，而且网卡自生产出来，就带着这个地址。很多人看到这里就会想，既然这样，整个互联网的通信，全部用 <font size = 3 color = red>MAC</font> 地址好了，只要知道了对方的<font size = 3 color = red> MAC</font> 地址，就可以把信息传过去。</p>

<p>这样当然是不行的。 <font size = 3 color= red>一个网络包要从一个地方传到另一个地方，除了要有确定的地址，还需要有定位功能。</font> 而有门牌号码属性的 IP 地址，才是有远程定位功能的。</p>

<p>例如，你去杭州市网商路 599 号 B 楼 6 层找刘超，你在路上问路，可能被问的人不知道 B 楼是哪个，但是可以给你指网商路怎么去。但是如果你问一个人，你知道这个身份证号的人在哪里吗？可想而知，没有人知道。</p>

<p><font color="orange">MAC 地址更像是身份证，是一个唯一的标识。</font>它的唯一性设计是为了组网的时候，不同的网卡放在一个网络里面的时候，可以不用担心冲突。从硬件角度，保证不同的网卡有不同的标识。</p>

<p>MAC 地址是有一定定位功能的，只不过范围非常有限。你可以根据 IP 地址，找到杭州市网商路 599 号 B 楼 6 层，但是依然找不到我，你就可以靠吼了，大声喊身份证 XXXX 的是哪位？我听到了，我就会站起来说，是我啊。但是如果你在上海，到处喊身份证 XXXX 的是哪位，我不在现场，当然不会回答，因为我在杭州不在上海。</p>

> 这个比喻可以啊

<p>所以，<font size = 3 color= red>MAC 地址的通信范围比较小，局限在一个子网里面</font>。例如，从 192.168.0.2/24 访问 192.168.0.3/24 是可以用 MAC 地址的。<font size = 3 color= red>一旦跨子网</font>，即从 192.168.0.2/24 到 192.168.1.2/24，MAC 地址就不行了，需要 IP 地址起作用了。</p>

### 网络设备的状态标识

<p>解析完了 MAC 地址，我们再来看<font size = 3 color= red> &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt;</font> 是干什么的？这个叫作<font size = 3 color= red><strong>net_device flags</strong></font>，<font size = 3 color= red><strong>网络设备的状态标识</strong></font>。</p>
<p><font size = 3 color= red>UP </font>表示网卡处于启动的状态；

<font size = 3 color= red>BROADCAST</font> 表示这个网卡有广播地址，可以发送广播包；

<font size = 3 color= red>MULTICAST</font> 表示网卡可以发送多播包；

<font size = 3 color= red>LOWER_UP</font> 表示 <font size = 3 color= red>L1</font> 是启动的，也即<font size = 3 color= red>网线插着呢</font>。

<font size = 3 color= red>MTU1500</font> 是指什么意思呢？是哪一层的概念呢？最大传输单元 MTU 为 1500，这是以太网的默认值。</p>
<p>我们讲过网络包是层层封装的。<font size = 3 color= red>MTU</font> 是<font size = 3 color= red>二层</font> <font size = 3 color= red>MAC</font> 层的概念。<font size = 3 color= red>MAC</font> 层有 <font size = 3 color= red>MAC</font> 的头，以太网规定连 MAC 头带正文合起来，不允许超过 1500 个字节。正文里面有 IP 的头、TCP 的头、HTTP 的头。如果放不下，就需要分片来传输。</p>
<p><font size = 3 color= red>qdisc pfifo_fast</font> 是什么意思呢？qdisc 全称是<strong><font size = 3 color= red>queueing discipline</font></strong>，中文叫<strong><font size = 3 color= red>排队规则</font></strong>。内核如果需要通过某个网络接口发送数据包，它都需要按照为这个接口配置的 <font size = 3 color= red>qdisc（排队规则）</font>把数据包加入队列。</p>
<p>最简单的 <font size = 3 color= red>qdisc</font> 是<font size = 3 color= red> pfifo</font>，它不对进入的数据包做任何的处理，数据包采用先入先出的方式通过队列。

<font size = 3 color= red>pfifo_fast</font> 稍微复杂一些，它的队列包括三个波段（<font size = 3 color= red>band</font>）。在每个波段里面，使用先进先出规则。</p>
<p>三个波段（band）的优先级也不相同。band 0 的优先级最高，band 2 的最低。如果 band 0 里面有数据包，系统就不会处理 band 1 里面的数据包，band 1 和 band 2 之间也是一样。</p>
<p>数据包是按照服务类型（<strong>Type of Service，TOS</strong>）被分配到三个波段（band）里面的。TOS 是 IP 头里面的一个字段，代表了当前的包是高优先级的，还是低优先级的。</p>

### 小结

- IP 是地址，有定位功能；MAC 是身份证，无定位功能；
- CIDR 可以用来判断是不是本地人；
- IP 分公有的 IP 和私有的 IP。

### net-tools 和 iproute2 的“历史”故事

net-tools起源于BSD，自2001年起，Linux社区已经对其停止维护，而iproute2旨在取代net-tools，并提供了一些新功能。一些Linux发行版已经停止支持net-tools，只支持iproute2。<br>net-tools通过procfs(/proc)和ioctl系统调用去访问和改变内核网络配置，而iproute2则通过netlink套接字接口与内核通讯。<br>net-tools中工具的名字比较杂乱，而iproute2则相对整齐和直观，基本是ip命令加后面的子命令。<br>虽然取代意图很明显，但是这么多年过去了，net-tool依然还在被广泛使用，