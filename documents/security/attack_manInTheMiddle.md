
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [Man-In-The-Middle Attack 中间人攻击](#man-in-the-middle-attack-中间人攻击)
	* [1. 什么是中间人攻击](#1-什么是中间人攻击)
	* [2. 中间人攻击方式](#2-中间人攻击方式)
		* [2.1 DNS欺骗(DNSSpoofing)](#21-dns欺骗dnsspoofing)
		* [2.2 会话劫持(SessionHijack)](#22-会话劫持sessionhijack)
		* [2.3 代理服务器(ProxyServer)](#23-代理服务器proxyserver)
	* [3. 防御手段](#3-防御手段)

<!-- /code_chunk_output -->

# Man-In-The-Middle Attack 中间人攻击

## 1. 什么是中间人攻击
中间人攻击是一种“间接”的入侵攻击，这种攻击模式是通过各种技术手段将受入侵者控制的一台计算机虚拟放置在网络连接中的两台通信计算机之间，这台计算机就称为“中间人”。

如图所示：

![中间人攻击_1](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/assets/images/security/Security_MITM_Attack_1.png?raw=true)

* 第1步：浏览器发起一次HTTP请求，但实际上会被攻击者拦截下来。
* 第2步：攻击者作为代理，把当前请求转发给钓鱼网站。
* 第3步：钓鱼网站返回假冒的网页内容。
* 第4步：攻击者把假冒的网页内容返回给浏览器。

这样就发生了一次中间人攻击。

## 2. 中间人攻击方式
### 2.1 DNS欺骗(DNSSpoofing)
攻击者通过入侵DNS服务器、控制路由器等方法把受害者要访问的目标机器域名对应的IP解析为攻击者所控制的机器，这样受害者原本要发送给目标机器的数据就发到了攻击者的机器上，这时攻击者就可以监听甚至修改数据，从而收集到大量的信息。
* 如果攻击者只是想监听双方会话的数据，他会转发所有的数据到真正的目标机器上，让目标机器进行处理，再把处理结果发回到原来的受害者机器。
* 如果攻击者要进行彻底的破坏，他会伪装目标机器返回数据，这样受害者接收处理的就不再是原来期望的数据，而是攻击者所期望的了。

例如让DNS服务器解析银行网站的IP为自己机器IP，同时在自己机器上伪造银行登录页面，那么受害者的真实账号和密码就暴露给入侵者了。

然而实际上它却很少派上大用场，因为DNS欺骗的攻击模型太理想了。在实际生活中，大部分用户的DNS解析请求均是通过自己的ISP服务器进行的，换句话说，就是系统在连接网络时会获取到ISP服务器提供的DNS服务器地址，所有解析请求都是直接发往这个DNS服务器的，攻击者根本无处入手，除非他能入侵更改ISP服务器上DNS服务的解析指向。所以这种手法在广域网上成功的几率不大。

### 2.2 会话劫持(SessionHijack)
简单地说，会话劫持就是攻击者把自己插入到受害者和目标机器之间，并设法让受害者和目标机器之间的数据通道变为受害者和目标机器之间存在一个看起来像“中转站”的代理机器（攻击者的机器）的数据通道，从而干涉两台机器之间的数据传输，例如监听敏感数据、替换数据等。由于攻击者已经介入其中，他能轻易知道双方传输的数据内容，还能根据自己的意愿去左右它。这个“中转站”可以是逻辑上的，也可以是物理上的，关键在于它能否获取到通信双方的数据。

### 2.3 代理服务器(ProxyServer)
代理服务器的工作模式，正是典型的“中间人攻击”模型。代理服务器在其中充当了一个“中间人”的角色，通讯双方计算机的数据都要通过它。因此，“代理服务器进行的‘中间人攻击’”逐步成为现实，相对于其他“中间人攻击”方法，这种利用代理服务器暗渡陈仓的做法简直天衣无缝，攻击者可以自己写一个带有数据记录功能的代理服务程序，放到任意一台稳定的肉鸡甚至直接在自己机器上，然后通过一些社会工程学手段让受害者使用这个做了手脚的“代理服务器”，便可守株待兔了。这种方法最让人不设防，因为它利用的是人们对代理的无条件信任和贪便宜的想法，使得一个又一个“兔子”自动撞了上来，在享受这顿似乎美味的“胡萝卜”的同时却不知道安全正在逐渐远离自己。

## 3. 防御手段
对于DNS欺骗，要记得检查本机的HOSTS文件，以免被攻击者加了恶意站点进去；其次要确认自己使用的DNS服务器是ISP提供的，因为当前ISP服务器的安全工作还是做得比较好的，一般水平的攻击者无法成功进入；如果是依靠网关设备自带的DNS解析来连接Internet的，就要拜托管理员定期检查网关设备是否遭受入侵。

至于局域网内各种各样的会话劫持，因为它们都要结合嗅探以及欺骗技术在内的攻击手段，必须依靠ARP和MAC做基础，所以网管应该使用交换式网络（通过交换机传输）代替共享式网络（通过集线器传输），这可以降低被窃听的机率，当然这样并不能根除会话劫持，还必须使用静态ARP、捆绑MAC+IP等方法来限制欺骗，以及采用认证方式的连接等。

但是对于代理服务器而言，以上方法就难以见效了，因为代理服务器本来就是一个“中间人”角色，攻击者不需要进行任何欺骗就能让受害者自己连接上来，而且代理也不涉及MAC等因素，所以一般的防范措施都不起作用。遇到这种情况最好的办法就是换一个代理。

另外，可以通过设置[HSTS](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/documents/security/http_response_header_Strick-Transport-Security.md)来一定程度上避免中间人攻击。