> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.bleepingcomputer.com](https://www.bleepingcomputer.com/news/security/new-stealthy-python-rat-malware-targets-windows-in-attacks/)

> 在野外发现了一种新的基于 Python 的恶意软件，具有远程访问木马 (RAT) 功能......

![](https://www.bleepstatic.com/content/hl-images/2022/05/05/malware-header.jpg)

在野外发现了一种新的基于 Python 的恶意软件，它具有远程访问木马 (RAT) 功能，可以让其操作员控制被破坏的系统。

威胁分析公司 Securonix 的研究人员将这种新型 RAT 命名为 PY#RATION，它使用 WebSocket 协议与命令和控制 (C2) 服务器通信，并从受害主机中窃取数据。

该公司的一份技术报告分析了恶意软件的工作原理。研究人员指出，RAT 正在积极开发中，因为自 8 月 PY#RATION 活动开始以来，他们已经看到了它的多个版本。

通过快捷方式文件分发
----------

PY#RATION 恶意软件通过网络钓鱼活动进行分发，该活动使用受密码保护的 ZIP 文件附件，其中包含两个伪装成图像的快捷方式 .LNK 文件，即_front.jpg.lnk_ 和_back.jpg.lnk_。

![](https://www.bleepstatic.com/images/news/u/1220909/2023/Malware/7/lnk-files.png)**获取两个批处理文件的两个 LNK 文件** _(Securonix)_

启动后，捷径受害者会看到驾照的正面和背面。然而，恶意代码也会被执行以联系 C2（在后来的攻击中是 Pastebin）并下载两个 .TXT 文件（“front.txt”和“back.txt”），这些文件最终被重命名为 BAT 文件以适应恶意软件的执行。

启动后，恶意软件会在用户的临时目录中创建“Cortana”和“Cortana/Setup”目录，然后从该位置下载、解压缩和运行其他可执行文件。

通过将批处理文件（“CortanaAssist.bat”）添加到用户的启动目录中来建立持久性。

使用 Microsoft 在 Windows 上的个人助理解决方案 Cortana，旨在将恶意软件条目伪装成系统文件。

![](https://www.bleepstatic.com/images/news/u/1220909/2023/Malware/7/infection-chain.png)**该活动的完整感染链** _（Securonix）_

隐秘的 PY#RATION RAT
-----------------

传送到目标的恶意软件是使用“pyinstaller”和“py2exe”等自动打包程序打包成可执行文件的 Python RAT，它们可以将 Python 代码转换为包含执行所需的所有库的 Windows 可执行文件。

这种方法会导致负载大小膨胀，1.0 版（初始）为 14MB，1.6.0 版（最新）为 32MB。最近的版本更大，因为它具有额外的代码（+1000 行）和一层 fernet 加密。

这有助于恶意软件逃避检测，并且根据 Securonix 的测试，除了 VirusTotal 上的一个防病毒引擎外，部署的 1.6.0 版有效载荷未被所有防病毒引擎检测到。

虽然 Securonix 没有共享恶意软件样本的哈希值，但 BleepingComputer 能够找到以下似乎来自该活动的文件：

![](https://www.bleepstatic.com/images/news/u/1100723/2023/PyRation_detection.png) _Py#Ration RAT （BleepingComputer_ ）**的检测率**

Securonix 的分析师提取了有效载荷的内容，并使用“pyinstxtractor”工具检查代码函数以确定恶意软件的功能。

![](https://www.bleepstatic.com/images/news/u/1220909/2023/Malware/7/extracted.png)**从可执行文件中提取的组件** _(Securonix)_

PY#RATION RAT 1.6.0 版中的功能如下：

*   执行网络枚举
*   执行从被破坏系统到 C2 的文件传输，反之亦然
*   执行键盘记录以记录受害者的击键
*   执行外壳命令
*   执行主机枚举
*   从网络浏览器中提取密码和 cookie
*   从剪贴板窃取数据
*   检测主机上运行的杀毒工具

![](https://www.bleepstatic.com/images/news/u/1220909/2023/Malware/7/browser-stealer.png)**从 Chrome、Brave、Opera 和 Edge 浏览器窃取数据** _(Securonix)_

[Securonix 研究人员表示](http://www.securonix.com/blog/security-advisory-python-based-pyration-attack-campaign/)，该恶意软件“利用了 Python 的内置 Socket.IO 框架，该框架为客户端和服务器 WebSocket 通信提供了功能。” 该通道用于通信和数据泄露。

WebSockets 的优势在于，恶意软件可以使用 80 和 443 等网络中通常开放的端口，通过单个 TCP 连接同时从 C2 接收和发送数据。

分析师注意到，威胁参与者在整个活动中使用相同的 C2 地址（“169[.]239.129.108”），从恶意软件版本 1.0 到 1.6.0。

据研究人员称，该 IP 并未在 IPVoid 检查系统上被阻止，这表明 PY#RATION 已经几个月未被发现。

目前，有关使用该恶意软件的具体活动及其目标、分布量和背后的运营商的详细信息仍不清楚。