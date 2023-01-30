> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [medium.com](https://medium.com/@R00tendo/get-real-ip-behind-a-reverse-proxy-3da75329f3b4)

> 概述

概述
--

在渗透测试或只是对网站进行一般修补期间，您可能已经注意到很多网站都指向 “CloudFlare” 或“Akamai”服务器。这些称为**反向代理**，它们通过服务器将流量路由到实际的 Web 服务器，以保护站点免受 DDOS 攻击等。本文旨在帮助您揭开它们背后的真实 IP。

![](https://miro.medium.com/max/875/0*x4wTCAtjB5qpvD89)

技术 1. 子域
--------

设置反向代理时，它是针对特定域名的，如果服务器有子域，如果子域没有通过反向代理路由，它们可能会泄露真实 IP。以下是一些用于查找域的子域的工具和站点：

*   DNSdumpster ( [https://dnsdumpster.com/](https://dnsdumpster.com/) )
*   Censys 搜索 ( [https://search.censys.io/](https://search.censys.io/) )
*   Sublist3r ( [https://github.com/aboul3la/Sublist3r](https://github.com/aboul3la/Sublist3r) )

技术 2. SSL 证书
------------

设置反向代理后，其背后的真实服务器可能仍然具有相同的 SSL 证书，因此如果我们使用相同证书查找所有 IP 和域名，我们可能会找到真实 IP。要发现与证书相关的所有内容，我建议使用 **Censys Search (** [**https://search.censys.io/**](https://search.censys.io/) **)**

技术 3. DNS 历史
------------

如果域名在没有反向代理的情况下被使用，则服务器的旧 IP 很可能已被公司记录，例如 SecurityTrails。以下是一些查看域名 IP 历史记录的站点：

*   SecurityTrails ( [https://securitytrails.com](https://securitytrails.com/) )
*   完整的 DNS ( [https://completedns.com/dns-history/](https://completedns.com/dns-history/) )

更多网站在这里：[https ://woorkup.com/view-dns-history-free/](https://woorkup.com/view-dns-history-free/)

[https://github.com/256o/CDNRECON](https://github.com/256o/CDNRECON)