# 技术总结文档

## 翻墙

### 为什么要翻墙

由于日益严峻的网络审查和封锁，为了使用一些常用的网站比如Google, Github等，使用一些技术手段进行翻墙

### 如何进行翻墙

在网络封锁早期，有很多免费的vpn代理软件，其中，比较有名的有：无界自由门Freegate，日本大学研究的openVPN，手机上限时免费使用的GreenVPN。

但是他们都有一些共同的缺点：

1. 首先是安全性，由于网络流量经过第三方的服务器，无法自己控制，所以安全性比较差。
2. 稳定性，这些vpn使用的都是公开的加密协议PPTP和P2TP，陆陆续续的被GFW封锁

下面是我一些曾经用过的几种翻墙工具

#### GoAgent

GoAgent，是比较早期的使用的一个翻墙工具，支持mac os，windows平台，通过在GoogleEngine布置免费的服务器来代理，有稳定性高，速度高的优点，缺点是所有流量都会通过Google的服务器, 所以在匿名性和隐藏性比较差。

原链接：[https://code.google.com/p/goagent/](https://code.google.com/p/goagent/)（已失效）

我的Fork ：[https://github.com/z563721/goagent](https://github.com/z563721/goagent)

