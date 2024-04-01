> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [sankalppatil12112001.medium.com](https://sankalppatil12112001.medium.com/google-hacking-google-dorks-for-sensitive-information-f1d5a8eedb32)

> In the vast internet landscape, Google is the gateway to an ocean of information. While most of us us......

[

![](https://miro.medium.com/v2/da:true/resize:fill:88:88/0*qig40e00DhgR6iSi)

](https://sankalppatil12112001.medium.com/?source=post_page-----f1d5a8eedb32--------------------------------)

In the vast internet landscape, Google is the gateway to an ocean of information. While most of us use Google for simple searches, a powerful tool known as “Google Dorks” unlocks a deeper level of search capability. In this blog post, we’ll look into what Google Dorks are, how they work, and how to use them ethically and responsibly.  
在广阔的互联网世界中，谷歌是通往海量信息的门户。虽然大多数人只是用谷歌进行简单的搜索，但一个强大的工具称为 “谷歌黑客” 可以解锁更深层次的搜索能力。在这篇博文中，我们将探讨谷歌黑客是什么，它们如何工作，以及如何在道德和负责任的情况下使用它们。

![](https://miro.medium.com/v2/resize:fit:875/1*XME-mdm-rYPoIx2OU49a2w.png)

Google Dorks, or Google hacking or Google-fu, refers to specialized search queries that utilize advanced operators to pinpoint specific information on the web. These operators allow users to narrow down their searches and find hidden data that may not be accessible through our regular search methods.  
Google Dorks，又称 Google 黑客术或 Google-fu，指的是利用高级运算符进行专业搜索查询，以准确定位网上特定信息的技术。这些运算符允许用户缩小搜索范围，找到通过常规搜索方法可能无法访问的隐藏数据。

Google Dorking utilizes custom queries with advanced search operators (specific symbols or words) to fetch targeted search results. All you have to do is type in the custom Google dork query in the Google search bar.  
谷歌 Dorking 利用自定义查询和高级搜索运算符（特定的符号或单词）来获取目标搜索结果。您只需在谷歌搜索栏中输入自定义的谷歌 Dork 查询。

When the Google search engine crawls the web, it indexes many parts of websites, some of which may not be readily available to regular internet users. Google Dorking lets you see some of that information using more precise search queries.  
当谷歌搜索引擎爬取网络时，它会索引网站的许多部分，其中一些可能不容易被普通互联网用户访问到。谷歌 Dorking 让你可以使用更精确的搜索查询来查看其中的一些信息。

Dork > site:microsoft.com keys
------------------------------

Using ‘site:microsoft.com keys’ in Google helps find specific key-related pages on Microsoft’s site. This simplifies searching for products, licenses, or encryption keys directly from Microsoft’s domain.  
在谷歌中使用 “site:microsoft.com keys” 可以帮助找到微软网站上与密钥相关的特定页面。这简化了直接从微软域名搜索产品、许可证或加密密钥的过程。

We can use different words in which we are interested like keys, emails, passwords, admin, etc. We can specify the website we want to do recon on finding information about.  
我们可以使用我们感兴趣的不同词汇，比如关键词、电子邮件、密码、管理员等。我们可以指定我们想要进行侦察以查找有关信息的网站。

![](https://miro.medium.com/v2/resize:fit:875/1*YyaXA4ERdjKEneH55s2Fzw.png)Example 01: site: example.com <keyword>  
示例 01：site: example.com <关键词>

Dork > “Not for Public Release” + “Confidential” ext:pdf | ext:doc | ext:xlsx  
白痴 > “非公开发布” + “机密” ext:pdf | ext:doc | ext:xlsx
--------------------------------------------------------------------------------------------------------------------------------

Using “Not for Public Release” + “Confidential” with file extensions pdf, doc, or xlsx helps find potentially sensitive documents not intended for public distribution. This advanced search technique can uncover confidential information across various file types.  
使用带有文件扩展名 pdf、doc 或 xlsx 的 “不得公开发布”+“机密” 有助于查找可能包含敏感信息且不适合公开发布的文件。这种高级搜索技术可以发现各种文件类型中的机密信息。

There are many file types indexable by Google which are shared in the last part of the blog and can be included for better results.  
Google 可以索引许多文件类型，这些文件类型在博客的最后部分共享，并且可以包含在其中以获得更好的结果。

![](https://miro.medium.com/v2/resize:fit:875/1*UlfCexkFWFGWeba-Q5qZ4Q.png)Example 02: “Not for Public Release” + “Confidential” ext:pdf | ext:doc | ext:xlsx  
示例 02：“不得公开发布”+“机密” ext:pdf | ext:doc | ext:xlsx

Dork > allintext:username filetype:log  
白痴 > 全文：用户名 文件类型：日志
------------------------------------------------------------

Using ‘allintext:username filetype: log’ in your search query can reveal log files containing usernames. This approach streamlines finding logs containing user-related information, aiding in security analysis or troubleshooting.  
在您的搜索查询中使用 “allintext:username filetype: log” 可以显示包含用户名的日志文件。这种方法简化了查找包含用户相关信息的日志，有助于安全分析或故障排除。

These usernames can be later used by the malicious threat actor for different password attacks like brute force, etc.  
这些用户名以后可以被恶意威胁行为者用于不同的密码攻击，如暴力破解等。

![](https://miro.medium.com/v2/resize:fit:875/1*wsxGE1VOnWi70AF1RzPI5A.png)Example 03: allintext:username filetype:log  
示例 03：所有文本中包含 "用户名" 且文件类型为 "log" 的内容

Dork > inurl:email.xls ext:xls  
傻瓜 > inurl:email.xls ext:xls
-------------------------------------------------------------

Using ‘inurl:email.xls ext:xls’ in your search query helps pinpoint Excel files with ‘email’ in their URL. This method efficiently locates spreadsheet files specifically tailored for email data, streamlining data retrieval tasks.  
在搜索查询中使用 “inurl:email.xls ext:xls” 有助于精确定位 URL 中带有 “email” 的 Excel 文件。这种方法有效地定位了专门用于电子邮件数据的电子表格文件，简化了数据检索任务。

To make it more efficient use it with the ‘site:’ so that a particular result can be obtained instead of searching for a pin in the straw of hay.  
为了使其更有效，请使用 “site:” 来获取特定结果，而不是在一堆草中搜索针。

![](https://miro.medium.com/v2/resize:fit:875/1*uTswQdXRTcnwg6cIGGU8qw.png)Example 04: inurl:email.xls ext:xls  
示例 04：inurl:email.xls ext:xls

Dork > filetype:txt intext:@gmail.com intext:password  
白痴 > filetype:txt intext:@gmail.com intext:password
-----------------------------------------------------------------------------------------------------------

Using ‘filetype: txt intext:@gmail.com intext: password’ in your search query helps find text files containing email addresses with associated passwords.  
在您的搜索查询中使用 “filetype: txt intext:@gmail.com intext: password” 有助于找到包含与密码相关的电子邮件地址的文本文件。

This search method can reveal potentially compromised accounts or security vulnerabilities. Using tools like DeHashed, and Have I Been Pwned can make it more effective as we can analyse whether the accounts have been breached earlier or not.  
这种搜索方法可以揭示潜在的被妥协的账户或安全漏洞。使用像 DeHashed 和 Have I Been Pwned 这样的工具可以使其更加有效，因为我们可以分析这些账户是否之前曾遭到攻击。

![](https://miro.medium.com/v2/resize:fit:875/1*emVp6zLJyiKc7oJQV2djVQ.png)Example 05: filetype:txt intext:@gmail.com intext:password  
示例 05：文件类型：txt intext：@gmail.com intext：password

Scope-restricting dorks help you specify the target range of websites and data types. You can add additional query items to these dorks for more specificity, like in the “filetype:”  
范围限制的 dorks 帮助您指定网站和数据类型的目标范围。您可以为这些 dorks 添加额外的查询项以获得更多的特定性，比如 “filetype:”。

Keep in mind that when you want to restrict search results to an exact phrase, you have to enclose the phrase within double quotation marks.  
请记住，当您想要将搜索结果限制为精确短语时，必须将短语用双引号括起来。

![](https://miro.medium.com/v2/resize:fit:875/1*jHQwI6jubJaJZq1o5uDP_A.png)Image By: Nordvpn 图片由 Nordvpn 提供

Informational dorks specify the type of information you are looking for and work best without additional query items.  
信息豆子指定您要查找的信息类型，并且最好不要添加额外的查询项。

![](https://miro.medium.com/v2/resize:fit:875/1*uqkuv8P-CzUEqdKuijRXLw.png)Image By: Nordvpn 图片由 Nordvpn 提供

Text dorks are useful when you’re looking for pages containing specific text strings.  
文本搜索在查找包含特定文本字符串的页面时非常有用。

![](https://miro.medium.com/v2/resize:fit:875/1*lqLWFcusAl-LmM3Acmqsfg.png)Image By: Nordvpn 图片由 Nordvpn 提供

This is a list of operators that help you refine your Google search:  
这是一个帮助您优化谷歌搜索的运算符列表：

![](https://miro.medium.com/v2/resize:fit:875/1*8zFj850cw42w2GGshW9qcA.png)Image By: Nordvpn 图片由 Nordvpn 提供

Google can index the content of most text-based files and certain encoded document formats. The most common file types we index include:  
谷歌可以索引大多数基于文本的文件内容和某些编码的文档格式。我们索引的最常见文件类型包括：

```
Adobe Portable Document Format (.pdf)
Adobe PostScript (.ps)
Comma-Separated Values (.csv)
Google Earth (.kml, .kmz)
GPS eXchange Format (.gpx)
Hancom Hanword (.hwp)
HTML (.htm, .html, other file extensions)
Microsoft Excel (.xls, .xlsx)
Microsoft PowerPoint (.ppt, .pptx)
Microsoft Word (.doc, .docx)
OpenOffice presentation (.odp)
OpenOffice spreadsheet (.ods)
OpenOffice text (.odt)
Rich Text Format (.rtf)
Scalable Vector Graphics (.svg)
TeX/LaTeX (.tex)
Text (.txt, .text, other file extensions), including source code in common programming languages, such as:
Basic source code (.bas)
C/C++ source code (.c, .cc, .cpp, .cxx, .h, .hpp)
C# source code (.cs)
Java source code (.java)
Perl source code (.pl)
Python source code (.py)
Wireless Markup Language (.wml, .wap)
XML (.xml)
Google can also index the following media formats:

Image formats: BMP, GIF, JPEG, PNG, WebP, and SVG
Video formats: 3GP, 3G2, ASF, AVI, DivX, M2V, M3U, M3U8, M4V, MKV, MOV, MP4, MPEG, OGV, QVT, RAM, RM, VOB, WebM, WMV, and XAP
```

No more stressing over syntax — just plug in what you need, and let the magic happen. Memorizing complex queries was not only time-consuming but also prone to human error. So just let the AI do its work!  
不再为语法而烦恼 — 只需输入你需要的内容，让魔法发生。记忆复杂的查询不仅耗时，而且容易出错。所以让人工智能去做它的工作吧！

[DorkGenius](https://dorkgenius.com/#!): 呆萌天才
---------------------------------------------

DorkGenius simplifies Google dork creation for cybersecurity pros, streamlining searches for vulnerabilities and sensitive data.  
DorkGenius 简化了网络安全专家的 Google dork 创建过程，简化了对漏洞和敏感数据的搜索。

![](https://miro.medium.com/v2/resize:fit:875/1*8Xd7hGm45buBKZKUEJ1hTQ.png)

[DorkGPT](https://dorkgpt.com/): DorkGPT：
-----------------------------------------

DorkGPT generates tailored Google dorks, aiding security experts in pinpointing potential exploits and exposed information.  
DorkGPT 生成定制的 Google dorks，帮助安全专家准确定位潜在的漏洞和暴露的信息。

![](https://miro.medium.com/v2/resize:fit:875/1*liXR1ccEGr_1asN1hYIyOg.png)

[Bug Bounty Dork](https://dork.bugbountyhunting.com/): 漏洞赏金黑客：
--------------------------------------------------------------

Targeted at bug bounty hunters, Bug Bounty Dork speeds up vulnerability discovery with optimized Google dorks for web applications.  
Bug Bounty Dork 旨在针对漏洞赏金猎人，通过优化的 Google dorks 加快发现 Web 应用程序的漏洞。

![](https://miro.medium.com/v2/resize:fit:875/1*fyoeml-fZA-TqSIYi5l1Gw.png)

[DorkSearch](https://www.dorksearch.com/): 白痴搜索
-----------------------------------------------

DorkSearch is a centralized engine for Google dorks, aiding security pros in finding relevant queries for assessments and recon.  
DorkSearch 是一个集中的引擎，用于 Google dorks，帮助安全专家找到相关的查询以进行评估和侦察。

![](https://miro.medium.com/v2/resize:fit:875/1*Lo21BYkER25xBEX0-ygltw.png)

[Google Dork Maker by StationX](https://www.stationx.net/google-dork-maker/):  
Google Dork Maker 由 StationX 制作:
----------------------------------------------------------------------------------------------------------------

StationX’s Dork Maker offers a user-friendly interface for crafting custom Google dorks, essential for penetration testing and data gathering.  
StationX 的 Dork Maker 提供了一个用户友好的界面，用于制作定制的 Google dorks，这对于渗透测试和数据收集至关重要。

![](https://miro.medium.com/v2/resize:fit:875/1*oAJr18PLpI7GZt_4VtJ92g.png)

Along with several Google Dork commands and operators, there are some advanced combinations of operators too that you can use to filter search results to maximize efficiency.  
除了几个 Google Dork 命令和操作符外，还有一些高级操作符的组合，您可以使用这些组合来过滤搜索结果，以最大化效率。

However, you can refer to the Google Hacker database to avoid typing these operators and combinations every time to search for any information. This database contains hundreds of combinations of multiple and advanced operators.  
然而，您可以参考 Google 黑客数据库，以避免每次搜索信息时都要输入这些运算符和组合。该数据库包含数百种多个和高级运算符的组合。

1. Searching for Vulnerable Webcams  
搜索易受攻击的网络摄像头
--------------------------------------------------

Find webcams with known vulnerabilities:  
查找具有已知漏洞的网络摄像头  
`intitle:"D-Link" inurl:"/view.htm"`

2. Finding Open Elasticsearch Instances with Specific Data  
查找具有特定数据的开放 Elasticsearch 实例
-----------------------------------------------------------------------------------------

Search for Elasticsearch instances containing specific data:  
搜索包含特定数据的 Elasticsearch 实例：  
`intext:"kibana" intitle:"Kibana"`

3. Exploring Open MongoDB Instances with Authentication Bypass  
通过身份验证绕过，探索开放的 MongoDB 实例。
-------------------------------------------------------------------------------------------

Search for MongoDB instances without authentication:  
搜索没有身份验证的 MongoDB 实例:  
`intext:"MongoDB Server Information" intitle:"MongoDB" -intext:"MongoDB Server Version"`

4. Identifying Exposed OpenCV Instances  
识别暴露的 OpenCV 实例
---------------------------------------------------------

Search for OpenCV instances with exposed data:  
搜索具有公开数据的 OpenCV 实例:  
`intitle:"OpenCV Server" inurl:"/cgi-bin/guestimage.html"`

5. Finding Exposed InfluxDB Instances  
发现暴露的 InfluxDB 实例
---------------------------------------------------------

Search for InfluxDB instances with default configurations:  
搜索具有默认配置的 InfluxDB 实例  
`intitle:"InfluxDB - Admin Interface"`

6. Locating Exposed RabbitMQ Management Interfaces  
定位暴露的 RabbitMQ 管理接口
------------------------------------------------------------------------

Search for RabbitMQ management interfaces:  
搜索 RabbitMQ 管理界面:  
`intitle:"RabbitMQ Management"`

7. Discovering Exposed Jenkins Builds  
发现暴露的 Jenkins 构建
--------------------------------------------------------

Search for Jenkins builds with specific information:  
搜索具有特定信息的 Jenkins 构建:  
`intitle:"Console Output" intext:"Finished: SUCCESS"`

8. Finding Exposed Grafana Dashboards  
查找暴露的 Grafana 仪表盘
---------------------------------------------------------

Search for Grafana dashboards:  
搜索 Grafana 仪表板：  
`intitle:"Grafana" inurl:"/dashboard/db"`

9. Exploring Open NVIDIA Jetson Devices  
9. 探索开放的 NVIDIA Jetson 设备
-------------------------------------------------------------------

Search for NVIDIA Jetson devices with open ports:  
搜索具有开放端口的 NVIDIA Jetson 设备:  
`intitle:"NVIDIA Jetson" intext:"NVIDIA Jetson"`

10. Locating Open Fortinet Devices  
定位开放的 Fortinet 设备
------------------------------------------------------

Search for Fortinet devices with open interfaces:  
搜索具有开放接口的 Fortinet 设备  
`intext:"FortiGate Console" intitle:"Dashboard"`

11. Discovering Exposed OpenEMR Installations  
11. 发现暴露的 OpenEMR 安装
--------------------------------------------------------------------

Search for OpenEMR installations with specific data:  
搜索具有特定数据的 OpenEMR 安装：  
`intitle:"OpenEMR Login" inurl:"/interface"`

12. Finding Exposed Jenkins Script Console  
12. 寻找暴露的 Jenkins 脚本控制台
--------------------------------------------------------------------

Search for Jenkins script consoles with default credentials:  
搜索具有默认凭据的 Jenkins 脚本控制台:  
`intitle:"Jenkins Script Console" intext:"Run groovy script"`

These advanced commands for Google dorking can be useful for specific security assessments and research purposes. Always ensure you have proper authorization and follow ethical guidelines when using advanced Google Dorking commands. Unauthorized or malicious use can have serious legal and ethical consequences.  
这些用于 Google Dorking 的高级命令可以在特定安全评估和研究目的中发挥作用。在使用高级 Google Dorking 命令时，请始终确保您拥有适当的授权并遵守道德准则。未经授权或恶意使用可能会带来严重的法律和道德后果。

Summary: 摘要:
------------

![](https://miro.medium.com/v2/resize:fit:875/0*NZt3AD6tAKJnr87k.png)Image By: StationX 图片由：StationX

Google Dorking is safe as long as you use it responsibly and ethically. Attempting to exploit security vulnerabilities in the configuration and code of websites without authorization is against the terms of service of most websites and might lead to legal consequences.  
只要你负责任和合乎道德地使用，Google Dorking 是安全的。未经授权地试图利用网站配置和代码中的安全漏洞违反了大多数网站的服务条款，并可能导致法律后果。

Even though Google dorking is legal, you should apply this method responsibly and adhere to the legal guidelines of websites. Misusing Google dorks for breaching security and accessing unauthorized information is illegal.  
尽管 Google dorking 是合法的，但您应该负责任地使用这种方法，并遵守网站的法律准则。滥用 Google dorks 来违反安全和访问未经授权的信息是非法的。

![](https://miro.medium.com/v2/resize:fit:875/1*fFRDG6KEwrEQDPXZat3Ctw.jpeg)

Google Dorking is also called “Google hacking” for a reason — cybercriminals sometimes use Google hacking as a form of passive attack to find and exploit security vulnerabilities and access sensitive content on poorly protected websites. Hackers might carry out cyberattacks to get hold of usernames, passwords, and personally identifiable information by using advanced Google dorks. So be careful what Google dorks you use and never abuse them for accessing private information without proper authorization.  
Google Dorking 也被称为 “Google 黑客攻击”，原因是网络犯罪分子有时会利用 Google 黑客攻击作为一种被动攻击的形式，以发现和利用安全漏洞，并访问安全保护不良的网站上的敏感内容。黑客可能会使用高级的 Google 黑客攻击技术进行网络攻击，以获取用户名、密码和个人身份信息。因此，在使用 Google 黑客攻击技术时要小心，并且不要滥用它们来未经适当授权访问私人信息。

Hey! If you enjoyed this blog, hop over to my other blogs too! There’s a whole world of fascinating content waiting for you to explore. Let’s dive in and soak up knowledge together!  
嘿！如果你喜欢这篇博客，也可以去看看我的其他博客！那里有一整个世界等着你去探索。让我们一起深入了解，共同吸收知识吧！

> “W**ith Google Dorks, we’re not just hackers; we’re digital commandos, wielding information as our weapon to penetrate the impenetrable.”**  
> 通过 Google Dorks，我们不仅仅是黑客；我们是数字突击队，挥舞信息作为我们渗透不可渗透的武器。

☣ Happy Hacking ☣  
— XoX  
☣ 快乐黑客 ☣ — XoX
------------------------------------------