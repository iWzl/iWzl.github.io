---
title: HTTP3协议的深入剖析
date: 2021-03-24 11:55:05
tags:
	- 网络协议
	- HTTP协议
	- 应用层
categories:
    - 网络协议
abstract: 原文来自陶辉老师的《深入剖析HTTP3协议》,这里做一些转载和自己的记录,免得忘了
---

# HTTP3协议的深入剖析

自2017年起HTTP3协议已发布了34个Draft，推出在即，Chrome、Nginx等软件都在跟进实现最新的草案。本文将介绍HTTP3协议规范、应用场景及实现原理。

2015年HTTP2协议正式推出后，已经有接近一半的互联网站点在使用它：

![http2站点使用率_20200824](https://img.hellobyebye.com/doc/2021032412271216165600321616560032829XkbSZ6.png)

（图片来自https://w3techs.com/technologies/details/ce-http2）

HTTP2协议虽然大幅提升了HTTP/1.1的性能，然而，基于TCP实现的HTTP2遗留下3个问题：

- 有序字节流引出的 **队头阻塞**[（Head-of-line blocking）](https://en.wikipedia.org/wiki/Head-of-line_blocking)，使得HTTP2的多路复用能力大打折扣；
- **TCP与TLS叠加了握手时延**，建链时长还有1倍的下降空间；
- 基于TCP四元组确定一个连接，这种诞生于有线网络的设计，并不适合移动状态下的无线网络，这意味着**IP地址的频繁变动会导致TCP连接、TLS会话反复握手**，成本高昂。

HTTP3协议解决了这些问题：

- HTTP3基于UDP协议重新定义了连接，在QUIC层实现了无序、并发字节流的传输，解决了队头阻塞问题（包括基于QPACK解决了动态表的队头阻塞）；
- HTTP3重新定义了TLS协议加密QUIC头部的方式，既提高了网络攻击成本，又降低了建立连接的速度（仅需1个RTT就可以同时完成建链与密钥协商）；
- HTTP3 将Packet、QUIC Frame、HTTP3 Frame分离，实现了连接迁移功能，降低了5G环境下高速移动设备的连接维护成本。

# HTTP3协议到底是什么？

就像HTTP2协议一样，HTTP3并没有改变HTTP1的语义。那什么是HTTP语义呢？在我看来，它包括以下3个点：

- 请求只能由客户端发起，而服务器针对每个请求返回一个响应；
- 请求与响应都由Header、Body（可选）组成，其中请求必须含有URL和方法，而响应必须含有响应码；
- Header中各Name对应的含义保持不变。

HTTP3在保持HTTP1语义不变的情况下，更改了编码格式，这由2个原因所致：

首先，是为了减少编码长度。下图中HTTP1协议的编码使用了ASCII码，用空格、冒号以及\r\n作为分隔符，编码效率很低：

![HTTP协议语义](https://img.hellobyebye.com/doc/2021032412285116165601311616560131825jRwhn2.png)

HTTP2与HTTP3采用二进制、静态表、动态表与Huffman算法对HTTP Header编码，不只提供了高压缩率，还加快了发送端编码、接收端解码的速度。

其次，由于HTTP1协议不支持多路复用，这样高并发只能通过多开一些TCP连接实现。然而，通过TCP实现高并发有3个弊端：

- 实现成本高。TCP是由操作系统内核实现的，如果通过多线程实现并发，并发线程数不能太多，否则线程间切换成本会以指数级上升；如果通过异步、非阻塞socket实现并发，开发效率又太低；
- 每个TCP连接与TLS会话都叠加了2-3个RTT的建链成本；
- TCP连接有一个防止出现拥塞的慢启动流程，它会对每个TCP连接都产生减速效果。

因此，HTTP2与HTTP3都在应用层实现了多路复用功能

![多路复用](https://img.hellobyebye.com/doc/2021032412294816165601881616560188614XVaK0p.png)

（图片来自：https://blog.cloudflare.com/http3-the-past-present-and-future/）

HTTP2协议基于TCP有序字节流实现，因此**应用层的多路复用并不能做到无序地并发，在丢包场景下会出现队头阻塞问题。**如下面的动态图片所示，服务器返回的绿色响应由5个TCP报文组成，而黄色响应由4个TCP报文组成，当第2个黄色报文丢失后，即使客户端接收到完整的5个绿色报文，但TCP层不会允许应用进程的read函数读取到最后5个报文，并发成了一纸空谈：

![阻塞队列](https://img.hellobyebye.com/doc/2021032412310516165602651616560265144TQYO1C.gif)

当网络繁忙时，丢包概率会很高，多路复用受到了很大限制。因此， **HTTP3采用UDP作为传输层协议，重新实现了无序连接，并在此基础上通过有序的QUIC Stream提供了多路复用** ，如下图所示：

![协议的调整](https://img.hellobyebye.com/doc/2021032412320016165603201616560320171ONaX6s.png)

（图片来自：https://blog.cloudflare.com/http3-the-past-present-and-future/）

最早这一实验性协议由Google推出，并命名为gQUIC，因此，IETF草案中仍然保留了QUIC概念，用来描述HTTP3协议的传输层和表示层。HTTP3协议规范由以下5个部分组成：

1. QUIC层由https://tools.ietf.org/html/draft-ietf-quic-transport-29 描述，它定义了连接、报文的可靠传输、有序字节流的实现；
2. TLS协议会将QUIC层的部分报文头部暴露在明文中，方便代理服务器进行路由。https://tools.ietf.org/html/draft-ietf-quic-tls-29 规范定义了QUIC与TLS的结合方式；
3. 丢包检测、RTO重传定时器预估等功能由https://tools.ietf.org/html/draft-ietf-quic-recovery-29 定义，目前拥塞控制使用了类似TCP New RENO的算法，未来有可能更换为基于带宽检测的算法（例如BBR）；
4. 基于以上3个规范，https://tools.ietf.org/html/draft-ietf-quic-http-29 定义了HTTP语义的实现，包括服务器推送、请求响应的传输等；
5. 在HTTP2中，由HPACK规范定义HTTP头部的压缩算法。由于HPACK动态表的更新具有时序性，无法满足HTTP3的要求。在HTTP3中，QPACK定义HTTP头部的编码：https://tools.ietf.org/html/draft-ietf-quic-qpack-16 。注意，以上规范的最新草案都到了29，而QPACK相对简单，它目前更新到16。

自1991年诞生的HTTP/0.9协议已不再使用， **但1996推出的HTTP/1.0、1999年推出的HTTP/1.1、2015年推出的HTTP2协议仍然共存于互联网中（HTTP/1.0在企业内网中还在广为使用，例如Nginx与上游的默认协议还是1.0版本），即将面世的HTTP3协议的加入，将会进一步增加协议适配的复杂度** 。



# 文章来源和引用说明

> **本文作者：** 陶辉 
> **本文链接：** https://www.taohui.tech/2021/02/04/网络协议/深入剖析HTTP3协议/