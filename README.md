---
description: 关于这些年对于翻墙技术探索的点点滴滴
---

# 翻墙技术与实践

## 

## 墙是什么

墙的全称是指的是长城防火墙（Great Fire-wall ）简称GFW。中国政府为了进行网路审查和筛选不良信息，在国际出口网关附近对网络数据进行检查。

明面上的理由是为了筛选网络上的黄赌毒政治信息和保护国内互联网发展，实际上同时也过滤了不配合进行言论控制的外国网站。比如Google，Facebook，Twitter，维基百科\(wikipedia\)各大资讯平台，不接受言论审查的全球最大代码托管网站Github。

### 墙的工作原理

GFW会对在黑名单上的网站的http请求发送reset信息对双方的tcp链接进行重置来达到禁止访问的效果。墙还会通过对数据流的特征分析，对疑似vpn的数据流量进行封锁，比如常用的PPTP和P2TP协议。

## 为什么要翻墙

由于日益严峻的网络审查和封锁，为了使用一些常用的网站比如Google, Github等，使用一些技术手段进行翻墙

## 如何进行翻墙

在网络封锁早期，有很多免费的vpn代理软件，其中，比较有名的有：无界自由门Freegate，日本大学研究的openVPN，手机上限时免费使用的GreenVPN。

但是他们都有一些共同的缺点：

1. 首先是安全性，由于网络流量经过第三方的服务器，无法自己控制，所以安全性比较差。
2. 稳定性，这些vpn使用的都是公开的加密协议PPTP和P2TP，陆陆续续的被GFW封锁

下面是我一些曾经用过的几种翻墙工具

### GoAgent

GoAgent，是比较早期的使用的一个翻墙工具，支持mac os，windows平台，通过在GoogleEngine布置免费的服务器来代理，有稳定性高，速度高的优点，缺点是所有流量都会通过Google的服务器, 所以在匿名性和隐藏性比较差。

原链接：[https://code.google.com/p/goagent/](https://code.google.com/p/goagent/)（已失效）

我的Fork ：[https://github.com/z563721/goagent](https://github.com/z563721/goagent)

