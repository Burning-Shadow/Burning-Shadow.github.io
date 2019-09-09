---
title: 四、DHCP与PXE：IP是怎么来的，又是怎么没的？
tags: [计算机网络]
categories: 网络协议

---

### 如何配置IP地址？

如果你对这一方面又了解（反正我没有），那就可以用命令自己配置一个ip地址。可以使用 <font size = 3 color = red>ifconfig </font>,也可以使用<font size = 3 color = red>ip addr </font>。设置好了以后，用下面这两个命令，将网卡 <font size = 3 color = red>up</font>一下（UP 表示网卡处于启动的状态），就可以开始工作了。

<!--more-->

#### 使用 net-tools：

```
$ sudo ifconfig eth1 10.0.0.1/24
$ sudo ifconfig eth1 up
```

#### 使用 iproute2：

```
$ sudo ip addr add 10.0.0.1/24 dev eth1
$ sudo ip link set up eth1
```

这里我觉得自己设置这个自由度太大了吧，是可以随便配置吗？那如果我配置一个和谁都不搭边的地址呢？

例如：旁边的机器都是<font size = 3 color = red>192.168.1.x</font>,我非得配置一个<font size = 3 color = red>16.158.23.6</font>，那这样会出现什么现象呢？

不会出现任何现象，就是包发不出去呗。那为什么发不出去呢？举例说明一下

<font size = 3 color = red>192.168.1.6</font> 就在你这台机器的旁边，甚至在同一个交换机上，而你却将机器的地址设置为了<font size = 3 color = red>16.158.23.6</font>。在这台起机器上，你企图去<font size = 3 color = red>ping 192.168.1.6</font>,你觉得只要包发出去了，同一个交换机的另一台机器就能马上收到，对不对？

但是Linux系统它没有你想的那么智能，它需要根据自己的逻辑去进行判断处理。

我们在第二节说过，<font size = 3 color= red>只要是在网络跑的包，都是完整的，可以有下层没上层，绝对不可能有上层没下层。</font>

现在我们有自己的<font size = 3 color= red>源IP地址 16.158.23.6</font>，也有<font size = 3 color= red>目标IP地址 192.168.1.6</font>，但是包发不出去，这是因为<font size = 3 color= red>MAC</font>层没有填。

现在<font size = 3 color= red>自己的MAC</font>地址自己知道，这个容易。但是<font size = 3 color= red>目标 MAC </font>填什么呢？是不是填<font size = 3 color= red> 192.168.1.6的 MAC 地址</font>呢？

当然不是这样。<font size = 3 color= red>Linux</font>首先会自己判断，物品要去的地址和我是一个网段的吗，或者和我的网卡是同一个网段呢？只有是一个网段的，它才会发送<font size = 3 color= red>ARP请求(后面会讲)</font>，获取<font size = 3 color= red>MAC地址</font>。如果发现不是呢？

<strong>Linux 默认的逻辑是，如果这是一个跨网段的调用，它便不会直接将包发送到网络上，而是企图将包发送到<font size = 3 color= red>网关</font>。</strong>

如果你配置了网关的话，<font size = 3 color= red>Linux</font>会获取<font size = 3 color= red>网关的MAC地址</font>，然后将包发出去，对于<font size = 3 color= red>192.168.1.6</font>这台机器来说，虽然路过它家门的这个包，目标<font size = 3 color= red>IP</font>是它，但是无奈<font size = 3 color= red>MAC</font>地址不是它的，所以它的网卡是不会将包收进去的。

如果没有配置网关呢？那包根本就发不出去。

如果将网关配置为<font size = 3 color= red>192.168.1.6</font>呢？<font size = 3 color= red>不可能</font>，<font size = 3 color= red>Linux</font>不会让你配置成功的，因为网关要和当前的网络至少一个网卡是同一个网段的，怎么可能<font size = 3 color= red>16.158.23.6</font>的网关是<font size = 3 color= red>192.168.1.6</font>呢

所以当你手动配置一台机器的网络<font size = 3 color= red>IP</font>时，一定要好好问问网络管理员。如果在机房里面，要去网络管理员那里申请，让他给你分配一段正确的 <font size = 3 color= red>IP</font> 地址。当然，真正配置的时候，一定不是直接用命令配置的，而是放在一个配置文件里面。<font size = 3>**不同系统的配置文件格式不同，但是无非就是 CIDR、子网掩码、广播地址和网关地址**</font>。

### 动态主机配置协议（DHCP）

<p>原来配置 IP 有这么多门道儿啊。你可能会问了，配置了 IP 之后一般不能变的，配置一个服务端的机器还可以，但是如果是客户端的机器呢？我抱着一台笔记本电脑在公司里走来走去，或者白天来晚上走，每次使用都要配置 IP 地址，那可怎么办？还有人事、行政等非技术人员，如果公司所有的电脑都需要 IT 人员配置，肯定忙不过来啊。</p>

<p>因此，我们需要有一个自动配置的协议，也就是称<strong>动态主机配置协议（Dynamic Host Configuration Protocol）</strong>，简称<strong>DHCP</strong>。</p>

有了这个协议，我们的网络管理员只需要配置一段共享的<font size = 3 color= red>IP</font>地址（自己理解为<font size = 3 color= red>IP池</font>）。每一台新接入的机器都通过<font size = 3 color= red>DHCP协议</font>，来这一段共享IP地址里<font size = 3 color= red>申请</font>，然后自己配置，等人走了，再<font size = 3 color= red>回收IP</font>地址

所以说，<font size = 3>**如果是数据中心里面的服务器，IP 一旦配置好，基本不会变，这就相当于买房自己装修。<font size = 3 color= red>DHCP 的方式就相当于租房</font>。你不用装修，都是帮你配置好的。你暂时用一下，用完退租就可以了。**</font>

### 解析 DHCP 的工作方式

<p>当一台机器新加入一个网络的时候，肯定一脸懵，啥情况都不知道，只知道自己的 MAC 地址。怎么办？先吼一句，我来啦，有人吗？这时候的沟通基本靠<font size = 3 color= red>“吼”</font>。这一步，我们称为<font size = 3 color= red>DHCP Discover。</font></p>

<p>新来的机器使用<font size = 3 color= red> IP</font> 地址<font size = 3 color= red> 0.0.0.0 </font>发送了一个<font size = 3 color= red>广播包</font>，<font size = 3 color= red>目的 IP </font>地址为<font size = 3 color= red> 255.255.255.255</font>。广播包封装了<font size = 3 color= red> UDP</font>，<font size = 3 color= red>UDP</font> 封装了 <font size = 3 color= red>BOOTP</font>。其实<font size = 3 color= red> DHCP</font> 是<font size = 3 color= red> BOOTP</font> 的增强版，但是如果你去抓包的话，很可能看到的名称还是<font size = 3 color= red> BOOTP 协议</font>。</p>

<p>在这个广播包里面，新人大声喊：我是新来的（Boot request），我的 MAC 地址是这个，我还没有 IP，谁能给租给我个 IP 地址！</p>

<p>格式就像这样：</p>

![image](https://static001.geekbang.org/resource/image/39/1f/395b304f49559034af34c882bd86f11f.jpg)

<p>如果一个网络管理员在网络里面配置了<font size = 3 color= red>DHCP Server</font>的话，他就相当于这些 IP 的管理员。他立刻能知道来了一个“新人”。这个时候，我们可以体会 MAC 地址唯一的重要性了。当一台机器带着自己的 MAC 地址加入一个网络的时候，MAC 是它唯一的身份，如果连这个都重复了，就没办法配置了。</p>

<p>只有 <font size = 3 color= red>MAC</font> 唯一，<font size = 3 color= red>IP</font> 管理员才能知道这是一个新人，需要租给它一个<font size = 3 color= red> IP </font>地址，这个过程我们称为<font size = 3 color= red>DHCP Offer</font>。同时，<font size = 3 color= red>DHCP Server</font> 为此客户保留为它提供的 <font size = 3 color= red>IP</font> 地址，从而不会为其他<font size = 3 color= red> DHCP 客户</font>分配<font size = 3 color= red>此 IP </font>地址。</p>

<p>DHCP Offer 的格式就像这样，里面有给新人分配的地址。</p>

![image](https://static001.geekbang.org/resource/image/54/86/54ffefbe4f493f0f4a39f45504bd5086.jpg)

<p><font size = 3 color= red>DHCP Server</font> 仍然使用广播地址作为目的地址，因为，此时请求分配 <font size = 3 color= red>IP</font> 的新人还没有自己的 <font size = 3 color= red>IP</font>。<font size = 3 color= red>DHCP Server</font> 回复说，我分配了一个可用的 <font size = 3 color= red>IP</font> 给你，你看如何？除此之外，服务器还发送了<font size = 3 color= red>子网掩码</font>、<font size = 3 color= red>网关</font>和 <font size = 3 color= red>IP 地址租用期</font>等信息。</p>

<p>新来的机器很开心，它的“吼”得到了回复，并且有人愿意租给它一个 IP 地址了，这意味着它可以在网络上立足了。当然更令人开心的是，如果有多个 <font size = 3 color= red>DHCP Server</font>，这台新机器会收到多个<font size = 3 color= red> IP</font> 地址，简直受宠若惊。</p>
<p>它会选择其中一个 <font size = 3 color= red>DHCP Offer</font>，一般是最先到达的那个，并且会向网络发送一个<font size = 3 color= red> DHCP Request </font>广播数据包，包中包含<font size = 3 color= red>客户端的 MAC 地址</font>、接受的租约中的<font size = 3 color= red> IP 地址</font>、提供此租约的<font size = 3 color= red> DHCP</font> 服务器地址等，并告诉所有<font size = 3 color= red> DHCP Server</font> 它将接受哪一台服务器提供的<font size = 3 color= red> IP 地址</font>，告诉其他<font size = 3 color= red> DHCP</font> 服务器，谢谢你们的接纳，并请求撤销它们提供的 <font size = 3 color= red>IP</font> 地址，以便提供给下一个 <font size = 3 color= red>IP</font> 租用请求者。</p>

![image](https://static001.geekbang.org/resource/image/e1/24/e1e45ba0d86d2774ec80a1d86f87b724.jpg)

<p>此时，由于还没有得到 DHCP Server 的最后确认，客户端仍然使用 0.0.0.0 为源 IP 地址、255.255.255.255 为目标地址进行广播。在 BOOTP 里面，接受某个 DHCP Server 的分配的 IP。</p>

<p>当 DHCP Server 接收到客户机的 DHCP request 之后，会广播返回给客户机一个 DHCP ACK 消息包，表明已经接受客户机的选择，并将这一 IP 地址的合法租用信息和其他的配置信息都放入该广播包，发给客户机，欢迎它加入网络大家庭。</p>

![image](https://static001.geekbang.org/resource/image/7d/0e/7da571c18b974582a9cfe4718c5dea0e.jpg)

<p>最终租约达成的时候，还是需要广播一下，让大家都知道。</p>

### IP 地址的收回和续租

<p>既然是租房子，就是有租期的。租期到了，管理员就要将 IP 收回。</p>
<p>如果不用的话，收回就收回了。就像你租房子一样，如果还要续租的话，不能到了时间再续租，而是要提前一段时间给房东说。DHCP 也是这样。</p>
<p>客户机会在租期过去 50% 的时候，直接向为其提供 IP 地址的 DHCP Server 发送 DHCP request 消息包。客户机接收到该服务器回应的 DHCP ACK 消息包，会根据包中所提供的新的租期以及其他已经更新的 TCP/IP 参数，更新自己的配置。这样，IP 租用更新就完成了。</p>



