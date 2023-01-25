> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.sentinelone.com](https://www.sentinelone.com/labs/dragonspark-attacks-evade-detection-with-sparkrat-and-golang-source-code-interpretation/)

> 一组攻击使用了一种新技术，即 Golang 源代码解释，以避免检测到......

**作者：Aleksandar Milenkoski、Joey Chen 和 Amitai Ben Shushan Ehrlich**

执行摘要
----

*   SentinelLabs 跟踪了最近针对东亚组织的一系列机会主义攻击，如 DragonSpark。
*   SentinelLabs 评估说 DragonSpark 攻击的幕后黑手极有可能是说中文的演员。
*   这些攻击提供的证据表明，讲中文的威胁参与者正在采用鲜为人知的开源工具 SparkRAT。
*   威胁行为者使用 Golang 恶意软件，该恶意软件实施了一种不常见的技术来阻碍静态分析和逃避检测：Golang 源代码解释。
*   DragonSpark 攻击利用位于中国和台湾的受损基础设施来部署 SparkRAT 以及其他工具和恶意软件。

概述
--

SentinelLabs 一直在监控最近针对我们追踪为“DragonSpark”的东亚组织的攻击。这些攻击的特点是使用鲜为人知的开源 SparkRAT 和试图通过 Golang 源代码解释来逃避检测的恶意软件。

DragonSpark 攻击代表了第一个具体的恶意活动，我们观察到开源[SparkRAT](https://github.com/XZB-1248/Spark)的一致使用，这是威胁领域的一个相对较新的事件。SparkRAT 是多平台的，功能丰富，并且经常更新新功能，这使得 RAT 对威胁行为者具有吸引力。

Microsoft 安全威胁情报团队于 2022 年 12 月下旬[报告](https://www.microsoft.com/en-us/security/blog/2022/12/21/microsoft-research-uncovers-new-zerobot-capabilities/)了威胁行为者使用 SparkRAT 的迹象。但是，我们没有观察到具体证据将 DragonSpark 与 Microsoft 报告中记录的活动联系起来。

我们观察到 DragonSpark 攻击背后的威胁行为者使用 Golang 恶意软件，该恶意软件在运行时将嵌入式 Golang 源代码解释为一种阻碍静态分析和逃避静态分析机制检测的技术。这种不常见的技术为威胁行为者提供了另一种通过混淆恶意软件实现来逃避检测机制的方法。

入侵向量
----

我们观察到暴露在 Internet 上的 Web 服务器和 MySQL 数据库服务器遭到破坏，这是 DragonSpark 攻击的初始指标。将 MySQL 服务器暴露[于 Internet](https://www.shadowserver.org/news/over-3-6m-exposed-mysql-servers-on-ipv4-and-ipv6/)是一种基础设施状态缺陷，通常会导致涉及数据泄露、凭据盗窃或跨网络横向移动的严重事件。在受感染的 Web 服务器上，我们观察到 China Chopper webshel​​l 的使用，可通过虚拟终端请求中的序列[识别。](https://blog.talosintelligence.com/china-chopper-still-active-9-years-later/)[China Chopper](https://www.cyber.nj.gov/threat-center/threat-profiles/trojan-variants/china-chopper)通常被中国威胁行为者使用，众所周知，他们通过不同的媒介部署 webshel​​l，例如利用 Web 服务器漏洞、跨站点脚本或 SQL 注入。`&echo [S]&cd&echo [E]`[](https://www.cyber.nj.gov/threat-center/threat-profiles/trojan-variants/china-chopper)

在获得对环境的访问权限后，威胁行为者进行了各种恶意活动，例如横向移动、特权升级以及部署托管在攻击者控制的基础设施中的恶意软件和工具。我们观察到威胁行为者严重依赖由中文开发人员或中国供应商开发的开源工具。这包括 SparkRAT 以及其他工具，例如：

*   [SharpToken](https://github.com/BeichenDream/SharpToken)：一种权限升级工具，可以使用 SYSTEM 权限执行 Windows 命令。该工具还具有枚举用户和进程信息，以及添加、删除或更改系统用户密码的功能。
*   [BadPotato](https://github.com/BeichenDream/BadPotato)：类似于 SharpToken 的工具，将用户权限提升到 SYSTEM 以执行命令。该工具已在中国威胁行为者发起的旨在获取情报的[攻击活动中被发现。](https://www.trellix.com/en-us/about/newsroom/stories/research/operation-harvest-a-deep-dive-into-a-long-term-campaign.html)
*   [GotoHTTP](https://gotohttp.com/)：一个跨平台的远程访问工具，实现了广泛的功能，例如建立持久性、文件传输和屏幕视图。

除了上述工具外，攻击者还使用了两个定制的恶意软件来执行恶意代码：ShellCode_Loader，用 Python 实现并作为 PyInstaller 包提供，以及 m6699.exe，用 Golang 实现。

踢鼠
--

SparkRAT 是一款使用 Golang 开发的 RAT，由中文开发者[XZB-1248作为](https://github.com/XZB-1248)[开源](https://github.com/XZB-1248/Spark)软件发布。SparkRAT 是一个功能丰富的多平台工具，支持 Windows、Linux 和 macOS 操作系统。[](https://github.com/XZB-1248)

SparkRAT 使用 WebSocket 协议与 C2 服务器通信，并具有升级系统。这使 RAT 能够在启动时通过发出升级请求自动将自身升级到 C2 服务器上可用的最新版本。这是一个 HTTP POST 请求，提交查询参数存储了该工具的当前版本。

![](https://www.sentinelone.com/wp-content/uploads/2023/01/SaprkRAT_6.jpg)SparkRAT 升级请求

在我们观察到的攻击中，SparkRAT 的版本是`6920f726d74efb7836a03d3acfc0f23af196765e`，构建于 2022 年 11 月 1 日 UTC。此版本支持 26 个命令，可实现广泛的功能：

*   命令执行：包括执行任意Windows系统和PowerShell命令。
*   系统操作：包括系统关机、重启、休眠、挂起等。
*   文件和进程操作：包括进程终止以及文件上传、下载和删除。
*   信息窃取：包括平台信息（CPU、网络、内存、磁盘、系统运行时间信息）外泄、截图窃取、进程和文件枚举。

![](https://www.sentinelone.com/wp-content/uploads/2023/01/SaprkRAT_5.jpg)SparkRAT版本

规避检测的Golang源码解读
---------------

Golang 恶意软件 m6699.exe 使用[Yaegi](https://github.com/traefik/yaegi)框架在运行时解释存储在已编译二进制文件中的编码 Golang 源代码，并像编译后一样执行代码。这是一种阻碍静态分析和逃避静态分析机制检测的技术。

m6699.exe 的主要目的是执行第一阶段 shellcode，为第二阶段 shellcode 实现加载程序。

m6699.exe 首先解码 Base-64 编码的字符串。该字符串是执行以下活动的 Golang 源代码：

*   `Main`将函数声明为`Run`包的一部分。该`run.Main`函数将一个字节数组作为参数——第一阶段的 shellcode。
*   该`run.Main`函数调用[HeapCreate](https://learn.microsoft.com/en-us/windows/win32/api/heapapi/nf-heapapi-heapcreate)函数来分配可执行和可增长的堆内存 ( `HEAP_CREATE_ENABLE_EXECUTE`)。
*   该`run.Main`函数将第一阶段 shellcode（在调用时作为参数提供给它）放入分配的内存中并执行它。

![](https://www.sentinelone.com/wp-content/uploads/2023/01/SaprkRAT_9.jpg)m6699.exe中的Golang源码

然后 m6699.exe 在 Yaegi 解释器的上下文中评估源代码，并使用 Golang[反射](https://go.dev/blog/laws-of-reflection)来执行`run.Main`函数。m6699.exe 作为参数传递给`run.Main`第一阶段 shellcode，该函数如前所述执行。m6699.exe 将 shellcode 存储为双 Base64 编码字符串，恶意软件在传递给 run.Main 执行之前对其进行解码。

![](https://www.sentinelone.com/wp-content/uploads/2023/01/SaprkRAT_4.jpg)run.Main以双重Base64编码和解码形式执行的第一阶段shellcode

第一阶段的 shellcode 实现了一个 shellcode 加载器。[shellcode 使用Windows Sockets 2](https://learn.microsoft.com/en-us/windows/win32/api/_winsock/)库连接到 C2 服务器并接收一个 4 字节的大值。该值是第二阶段 shellcode 的大小，第一阶段 shellcode 为其分配接收大小的内存。然后，第一阶段 shellcode 从 C2 服务器接收第二阶段 shellcode 并执行它。

当 m6699.exe 执行时，攻击者可以建立 Meterpreter 会话以执行远程命令。

![](https://www.sentinelone.com/wp-content/uploads/2023/01/SaprkRAT_3.jpg)带有 m6699.exe 实例的 Meterpreter 会话（在实验室环境中）

ShellCode_Loader
----------------

ShellCode_Loader 是在 Python 中实现的 PyInstaller 打包恶意软件的内部名称。ShellCode_Loader 用作实现反向 shell 的 shellcode 的加载程序。

ShellCode_Loader 使用编码和加密来阻碍静态分析。恶意软件首先进行 Base-64 解码，然后解密 shellcode。ShellCode_Loader 使用 AES CBC 加密算法，并使用 Base-64 编码的 AES 密钥和初始化向量进行解密。

![](https://www.sentinelone.com/wp-content/uploads/2023/01/SaprkRAT_2.jpg)ShellCode_Loader 解码解密shellcode

ShellCode_Loader 使用用于访问 Windows API 的 Python [ctypes](https://docs.python.org/3/library/ctypes.html)库在内存中加载 shellcode 并启动一个执行 shellcode 的新线程。执行这些活动的 Python 代码是 Base-64 编码的，以试图逃避静态分析机制，该机制警告将 Windows API 用于恶意目的。

![](https://www.sentinelone.com/wp-content/uploads/2023/01/SaprkRAT_7.jpg)ShellCode_Loader 执行shellcode

[shellcode 创建一个线程并使用Windows Sockets 2](https://learn.microsoft.com/en-us/windows/win32/api/_winsock/)库连接到 C2 服务器。当 shellcode 执行时，威胁参与者可以建立 Meterpreter 会话以执行远程命令。

![](https://www.sentinelone.com/wp-content/uploads/2023/01/SaprkRAT_8.jpg)带有 ShellCode_Loader 实例的 Meterpreter 会话（在实验室环境中）

基础设施
----

DragonSpark 攻击利用位于台湾、香港、中国和新加坡的基础设施来部署 SparkRAT 和其他工具和恶意软件。C2 服务器位于香港和美国。

恶意软件暂存基础设施包括受感染的合法台湾组织和企业的基础设施，例如婴儿用品零售商、艺术画廊以及游戏和赌博网站。我们还观察到 Amazon Cloud EC2 实例作为该基础设施的一部分。

下表概述了 DragonSpark 攻击中使用的基础设施。

### 恶意软件暂存基础设施

<table><tbody><tr><td><strong>IP地址/域名</strong></td><td><strong>国家</strong></td><td><strong>笔记</strong></td></tr><tr><td>211.149.237[.]108</td><td>中国</td><td>托管与赌博相关的 Web 内容的受感染服务器。</td></tr><tr><td>43.129.227[.]159</td><td>香港</td><td>计算机名称为 的 Windows Server 2012 R2 实例<code>172_19_0_3</code>。威胁行为者可能已经使用共享或购买的帐户获得了对该服务器的访问权限。我们观察到带有服务器名称的登录凭据在 Telegram 频道的不同时间段内共享，<code>King of VP$</code>并<code>SellerVPS</code>用于共享和/或出售对虚拟专​​用服务器的访问权限。</td></tr><tr><td>www[.]bingoplanet[.]com[.]tw</td><td>台湾</td><td>托管与赌博相关的 Web 内容的受感染服务器。在撰写本文时，网站资源已被删除。该域已与其他几个合法业务网站共同托管，包括旅行社和英语幼儿园。</td></tr><tr><td>www[.]moongallery.com[.]tw</td><td>台湾</td><td>托管台湾艺术画廊 Moon Gallery 网站的受感染服务器。</td></tr><tr><td>www[.]holybaby.com[.]tw</td><td>台湾</td><td>托管台湾婴儿用品店零售商 Holy Baby 网站的受感染服务器。</td></tr><tr><td>13.213.41[.]125</td><td>新加坡</td><td>一个名为 的 Amazon Cloud EC2 实例<code>EC2AMAZ-4559AU9</code>。</td></tr></tbody></table>

### C2 服务器基础设施

<table><tbody><tr><td><strong>IP地址/域名</strong></td><td><strong>国家</strong></td><td><strong>笔记</strong></td></tr><tr><td>103.96.74[.]148</td><td>香港</td><td>计算机名称为 的 Windows Server 2012 R2 实例<code>CLOUD2012R2</code>。<br>威胁行为者可能已经使用共享或购买的帐户获得了对该服务器的访问权限。我们观察到带有服务器名称的登录凭据在 Telegram 频道的不同时间段内共享<code>Premium Acc</code>，<code>IRANHACKERS</code>以及<code>!Only For Voters</code>用于共享和/或出售对虚拟专​​用服务器的访问权。<br>在撰写本文时，已观察到这组基础设施解析为<code>jiance.ittoken[.]xyz</code>。在过去几年中，这个特定域可以链接到更广泛的中国网络钓鱼基础设施。目前还不清楚他们是否与同一演员有关。</td></tr><tr><td>104.233.163[.]190</td><td>美国</td><td>计算机名称为 的 Windows Server 2012 R2 实例<code>WIN-CLC0OFDKTMK</code>。<br>与此 IP 地址相关的最新被动 DNS 记录指向具有中文 TLD 的域名 - <code>kanmn[.]cn</code>. 然而，这是通过 Aquanx 共享的托管基础​​设施，可能被各种客户使用。<br>已知此 IP 地址托管了 Cobalt Strike C2 服务器并参与了其他恶意活动，例如托管已知的恶意软件样本。</td></tr></tbody></table>

归因分析
----

我们评估说 DragonSpark 攻击的幕后黑手很可能是说中文的威胁演员。由于缺乏可靠的攻击者特定指标，我们目前无法将 DragonSpark 与特定威胁攻击者联系起来。

演员可能有间谍或网络犯罪动机。2022 年 9 月，也就是我们首次发现 DragonSpark 指标的几周前，据报道，Zegost 恶意软件样本 ( [bdf792c8250191bd2f5c167c8dbea5f7a63fa3b4](https://www.virustotal.com/gui/file/1233a3d7bb4cfc8b9783a6bde15edfd8f5274acb7666e14f75ed5348cf7699e9/relations) )——一种历史上归因于中国网络犯罪分子的信息窃取程序，但也被观察为[间谍](https://www.fortinet.com/blog/threat-research/zegost-campaign-targets-internal-interests)活动的一部分——[据报道](https://www.virustotal.com/gui/ip-address/104.233.163.190/relations)与`104.233.163[.]190`. 我们观察到这个相同的 C2 IP 地址是 DragonSpark 攻击的一部分。微步情报局[此前的研究报告称，中国网络犯罪分子 FinGhost 正在使用 Zegost，包括上述样本的变体。](https://www.ctfiot.com/41387.html)

此外，DragonSpark 背后的威胁行为者使用 China Chopper webshel​​l 部署恶意软件。China Chopper 历史上一直被中国网络犯罪分子和间谍组织使用，例如[TG-3390](https://www.secureworks.com/research/threat-group-3390-targets-organizations-for-cyberespionage)和[Leviathan](https://www.mandiant.com/resources/blog/suspected-chinese-espionage-group-targeting-maritime-and-engineering-industries)。此外，威胁行为者进行 DragonSpark 攻击所使用的所有开源工具都是由说中文的开发人员或中国供应商开发的。这包括[XZB-1248](https://github.com/XZB-1248)的[SparkRAT](https://github.com/XZB-1248/Spark)，[BeichenDream](https://github.com/BeichenDream/BadPotato)的[SharpToken和](https://github.com/BeichenDream/SharpToken)[BadPotato](https://github.com/BeichenDream/)，以及[Pingbo](https://gotohttp.com/) Inc的GotoHTTP。[](https://github.com/XZB-1248)[](https://github.com/BeichenDream/SharpToken)[](https://github.com/BeichenDream/BadPotato)[](https://github.com/BeichenDream/)[](https://gotohttp.com/)

最后，恶意软件暂存基础设施仅位于东亚（台湾、香港、中国和新加坡），这种行为在以该地区的受害者为目标的讲中文的威胁参与者中很常见。这一证据与我们的评估一致，即 DragonSpark 攻击极有可能是由说中文的威胁演员精心策划的。

结论
--

[众所周知](https://www.cisa.gov/uscert/ncas/alerts/aa22-158a)，讲中文的威胁行为者经常在恶意活动中使用开源软件。我们在 DragonSpark 攻击中观察到的鲜为人知的 SparkRAT 是这些攻击者工具集中的最新成员之一。

由于 SparkRAT 是一个多平台且功能丰富的工具，并且会定期更新新功能，我们估计 RAT 在未来仍将对网络犯罪分子和其他威胁行为者具有吸引力。

此外，威胁参与者几乎肯定会继续探索执行环境的技术和特性，以逃避检测和混淆恶意软件，例如我们在本文中记录的 Golang 源代码解释。

SentinelLabs 继续监控 DragonSpark 活动集群，并希望防御者利用本文中提供的发现来加强他们的防御。

妥协指标
----

<table><tbody><tr><td><strong>描述</strong></td><td><strong>指标</strong></td></tr><tr><td>ShellCode_Loader（一个 PyInstaller 包）</td><td>83130d95220bc2ede8645ea1ca4ce9afc4593196</td></tr><tr><td>m6699.exe</td><td>14ebbed449ccedac3610618b5265ff803243313d</td></tr><tr><td>踢鼠</td><td>2578efc12941ff481172dd4603b536a3bd322691</td></tr><tr><td>ShellCode_Loader 的 C2 服务器网络端点</td><td>103.96.74[.]148:8899</td></tr><tr><td>SparkRAT 的 C2 服务器网络端点</td><td>103.96.74[.]148[:]6688</td></tr><tr><td>m6699.exe 的 C2 服务器网络端点</td><td>103.96.74[.]148:6699</td></tr><tr><td>China Chopper的C2服务器IP地址</td><td>104.233.163[.]190</td></tr><tr><td>ShellCode_Loader 的暂存 URL</td><td>hxxp://211.149.237[.]108:801/py.exe</td></tr><tr><td>m6699.exe 的暂存 URL</td><td>hxxp://211.149.237[.]108:801/m6699.exe</td></tr><tr><td>SparkRAT 的暂存 URL</td><td>hxxp://43.129.227[.]159:81/c.exe</td></tr><tr><td>GotoHTTP 的暂存 URL</td><td>hxxp://13.213.41.125:9001/go.exe</td></tr><tr><td>ShellCode_Loader 的暂存 URL</td><td>hxxp://www.bingoplanet[.]com[.]tw/images/py.exe</td></tr><tr><td>ShellCode_Loader 的暂存 URL</td><td>hxxps://www.moongallery.com[.]tw/upload/py.exe</td></tr><tr><td>ShellCode_Loader 的暂存 URL</td><td>hxxp://www.holybaby.com[.]tw/api/ms.exe</td></tr></tbody></table>