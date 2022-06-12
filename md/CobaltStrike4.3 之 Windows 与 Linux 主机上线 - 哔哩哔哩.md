> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.bilibili.com](https://www.bilibili.com/read/cv12772104/)

> 作者：Zhbk 原文地址：http://33h.co/w467c * 此工具仅供技术分享、交流讨论，严禁用于非法用途。

> 作者：Zhbk
> 
> 原文地址：http://33h.co/w467c

*** 此工具仅供技术分享、交流讨论，严禁用于非法用途。后果自负**

**渗透人员必备的一款工具，CS 主机上线是基操，没有上线啥都免谈。**

几天前和朋友在泡茶的时候，谈起了渗透工程师的面试，当时 HR 问他，CS 怎么上线 Linux 呢，这时他反问我，我说我不知道，他在纸上写下 CrossC2 后，嘴里说着” 小菜鸡 “，转身离开了。

CrossC2 简而言之，就是上线 Linux 系统的拓展插件

初步搭建服务器及启动 CS 就跳过了，主要是为了记录自己学习 CS 的笔记。

毕竟拿下蓝队才是我们的终极目标。

![](http://i0.hdslb.com/bfs/article/46a523e7256133e55727af310954907982e01dac.png@942w_707h_progressive.webp)

New Connection：打开一个新的”Connect” 窗口。在当前窗口中新建一个连接，即可同时连接不同的团队服务器 (便于团队之间的协作)

Preferences：偏好设置，首选项，用于设置 Cobalt Strike 主界面、控制台、TeamServer 连接记录、报告的样式

Visualization：将主机以不同的权限展示出来（主要以输出结果的形式展示）

VPN Interfaces：设置 VPN 接口

Listeners：创建监听器

Script Manager：查看和加载 CNA 脚本

Close：关闭当前与 TeamServer 的连接

![](http://i0.hdslb.com/bfs/article/ce09be6b69f5aad624832ca0f4d43f664cd31fae.png@942w_714h_progressive.webp)

Applications：显示被控机器的应用信息  

Credentials：通过 HashDump 或 mimikatz 获取的密码或者散列值都储存在这里

Downloads：从被控机器中下载的文件

Event Log：主机上线记录，以及与团队协作相关的聊天记录和操作记录

Keystrokes：键盘记录

Proxy Pivots：代理模块

Screenshots：屏幕截图模块

Script Console：控制台，在这里可以加载各种脚本（链接）

Targets：显示目标

Web Log：Web 访问日志

![](http://i0.hdslb.com/bfs/article/20964aa7bb2885a8862222670b2f669bd6fd4fec.png@942w_710h_progressive.webp)

HTML Application：基于 HTML 应用的 Payload 模块，通过 HTML 调用其他语言的应用组件进行攻击测试，提供了可执行文件、PowerShell、 VBA 三种方法  

MS Office Macro：生成基于 Office 病毒的 Payload 模块

Payload Generator：Payload 生成器，可以生成基于 C、C#、COM Scriptlet、 Java、 Perl、 PowerShell、Python、 Ruby、 VBA 等的 Payload

Windows Executable: 可以生成 32 位或 64 位的 EXE 和基于服务的 EXE、DLL 等后门程序。

在 32 位的 Windows 操作系统中无法执行 64 位的 Payload, 而且对于后渗透测试的相关模块，使用 32 位和 64 位的 Payload 会产生不同的影响，因此在使用时应谨慎选择

Windows Executable (S): 用于生成一个 W‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍indows 可执行文件，其中包含 Beacon 的完整 Payload, 不需要阶段性的请求。

与 Windows Executable 模块相比，该模块额外提供了代理设置，以便在较为苛刻的环境中进行渗透测试。

该模块还支持 PowerShell 脚本，可用于将●Stageless Payload 注入内存

![](http://i0.hdslb.com/bfs/article/96a3849d32eceeacf010f575ac01df4a8815f360.png@942w_708h_progressive.webp)

Manage：管理器，用于对 TeamServer 上已经开启的 Web 服务进行管理，包括 Listener 及 Web Delivery 模块  

Clone Site：用于克隆指定网站的样式

Host File：用于将指定文件加载到 Web 目录中，支持修改 Mime Type

Script Web Delivery：基于 Web 的攻击测试脚本，自动生成可执行的 Payload

Signed Applet Attack：使用 Java 自签名的程序进行钓鱼攻击测试。如果用户有 Applet 运行权限，就会执行其中的恶意代码

Smart Applet Attack：自动检测 Java 的版本并进行跨平台和跨浏览器的攻击测试。

该模块使用嵌入式漏洞来禁用 Java 的安全沙盒。可利用此漏洞的 Java 版本为 1.6.0_45 以下及 1.7.0 _21 以下

System Profiler：客户端检测工具，可以用来获取一些系统信息，例如系统版本、浏览器版本、Flash 版本等

![](http://i0.hdslb.com/bfs/article/4275b294b6eecb8ee78a073e8c5b3063a64f5cba.png@942w_707h_progressive.webp)

activity report 活动报告生成  

Hosts report 主机报告

Indicators opromisef com 目标报告

Sessions report 会话报告

Social engineering report 社会工程学报告

Reset Data 重置数据

Export data 数据出口

![](http://i0.hdslb.com/bfs/article/3adf3a28b208e47fee8891115ec20f7e8677b98b.png@942w_672h_progressive.webp) ![](http://i0.hdslb.com/bfs/article/962a4c801647675ac411af085c1c4748c975da3f.png@942w_714h_progressive.webp) ![](http://i0.hdslb.com/bfs/article/87a20e1e355e751362603866a0fc47bf3c21e7fe.png@704w_887h_progressive.webp)

**Payload**

Beacon DNS（以 DNS 协议流量建立 Beacon 连接）

![](http://i0.hdslb.com/bfs/article/7775adec6c60642788457510f9bb0c38fc2e5cb0.png@702w_879h_progressive.webp)

DNS Hosts：Beacon 回连的主机，可以添加多个  

Host Rotation Strategy：Beacon 回连主机策略

HTTP Host (Stager)：配置 Stager 主机，仅当 Payload 明确需要 Stager 配合时有效

Profile：Malleable C2 配置文件，用于自定义通信流量特征

DNS Port (Bind)：绑定监听端口，实现端口重定向

DNS Resolver：指定 NS 服务器

Beacon HTTP（以 HTTP 协议流量建立 Beacon 连接）

Beacon HTTPS（以 HTTPS 协议流量建立 Beacon 连接）

![](http://i0.hdslb.com/bfs/article/5d83f11133b7f8a31f71d8e92e91f20ca058b717.png@704w_887h_progressive.webp)

HTTPS Hosts：Beacon 回连的主机，可以添加多个

Host Rotation Strategy：Beacon 回连主机策略

HTTPS Host (Stager)：配置 Stager 主机，仅当 Payload 明确需要 Stager 配合时有效

Profile：Malleable C2 配置文件，用于自定义通信流量特征

HTTPS Port (C2)：Beacon 回连的监听端口

HTTPS Port (Bind)：绑定监听端口，实现端口重定向

HTTPS Host Header：设置内层真实域名，在使用域前置技术时使用

HTTPS Proxy：为 Payload 指定代理

Beacon SMB（以 SMB 协议流量建立 Beacon 连接）

![](http://i0.hdslb.com/bfs/article/a5e155f2cf7e6005fc33a6ef4fd11551fe484f19.png@714w_600h_progressive.webp)

使用命名管道通过父级 Beacon 进行通讯，当两个 Beacon 连接后，子 Beacon 从父 Beacon 获取任务执行，两者使用 Windows 命名管道通信，流量封装在 SMB 协议中，较为隐蔽  

**Beacon TCP（以 TCP 协议流量建立 Beacon 连接）**

![](http://i0.hdslb.com/bfs/article/266b59627dfaef0f971c2ecfbc5bb58a9146b11d.png@690w_596h_progressive.webp)

**External C2**  

External C2 是一种通信规范

Foreign HTTP（以 HTTP 协议流量建立会话，适用于与外部程序联动）

Foreign HTTPS（以 HTTPS 协议流量建立会话，适用于与外部程序联动）

![](http://i0.hdslb.com/bfs/article/804bb8d42989b94a2a7dc3b10475b4260ae16310.png@942w_729h_progressive.webp)

成功开启监听，接下来就是让主机上线  

**No.1**

![](http://i0.hdslb.com/bfs/article/9d55cfac1b64ee60dd120c11da704b74807adbb1.png@942w_716h_progressive.webp) ![](http://i0.hdslb.com/bfs/article/492f9334fa352095db7d0cc7284bb0a0d42c883a.png@560w_324h_progressive.webp) ![](http://i0.hdslb.com/bfs/article/9587fdac4cabb53d79a966575800fa8d6fe54f69.png@935w_338h_progressive.webp) ![](http://i0.hdslb.com/bfs/article/7f226ffcfd8a48bdeba362cc5108bec99a8b8751.png@212w_194h_progressive.webp) ![](http://i0.hdslb.com/bfs/article/4d9da905d195e73e7ea494d775a09531758721cf.png@942w_719h_progressive.webp)

**No.2**

![](http://i0.hdslb.com/bfs/article/35b987cc175d6ba63779ebcfeb649e451e24d720.png@942w_711h_progressive.webp) ![](http://i0.hdslb.com/bfs/article/cade718dc4a191044821a6c4387f0f4b1e20c6ca.png@516w_251h_progressive.webp) ![](http://i0.hdslb.com/bfs/article/e69af4c843c48c323ea2adcedb80d6ed5a00879d.png@930w_350h_progressive.webp) ![](http://i0.hdslb.com/bfs/article/3dba7e5dab6961a2ae39fb19eda19a1df16bd61c.png@738w_486h_progressive.webp) ![](http://i0.hdslb.com/bfs/article/e99ec7aaeccd59ff88322647769ba67f139ad7d4.png@942w_126h_progressive.webp)

Emmm，作者这个无法实现上线，具体思路是这样的  

保存成一个带宏模板的办公文件，当受害人点击使用宏模板时，主机上线

大家可以试一试

**No.3**

![](http://i0.hdslb.com/bfs/article/30c332ebbd32a6b83b26bd3dfb6816caeb6be9b8.png@942w_714h_progressive.webp) ![](http://i0.hdslb.com/bfs/article/c7e0d69b30d4d3c362c73398c1e1a145fe5c6dc7.png@549w_506h_progressive.webp)

这边举个栗子就好了，Powershell Command ，会生成一个文件，保存下来，用 powershell 执行，这边的原理大致就是生成可执行木马文件

执行后，主机上线，但是只能上线 Windows 主机

![](http://i0.hdslb.com/bfs/article/072e9d62d1721e2fca77dc608a9d7da311ef8d03.png@942w_384h_progressive.webp) ![](http://i0.hdslb.com/bfs/article/f6fbf184a2273bead7a1ca50ae699f27f1e13d96.png@942w_708h_progressive.webp)

**No.4** Windows Executable 和 Windows Executable（S）用法相同  

![](http://i0.hdslb.com/bfs/article/29b4672c05feedd0d16dd0b5ee43df32960f862a.png@942w_716h_progressive.webp) ![](http://i0.hdslb.com/bfs/article/a67b327400a3699762c8eebdd9373140451f034b.png@540w_410h_progressive.webp) ![](http://i0.hdslb.com/bfs/article/ebc067f5ed0d1cbb483c0724e8655e20d1e32b9a.png@180w_167h_progressive.webp) ![](http://i0.hdslb.com/bfs/article/a33974b9fe4ea6b9c01afc96564946a526aec604.png@942w_711h_progressive.webp)

**No.5**

![](http://i0.hdslb.com/bfs/article/35f0cf4cfd3a5f33b5bba352b5b59fc9edbd1635.png@578w_489h_progressive.webp)

生成一个 Url，访问这个网站去下载，使被攻击机成功上线，目前也未能复现

**No.6**

![](http://i0.hdslb.com/bfs/article/db4505dd849ea9298654369193bcb4aff5597e57.png@942w_719h_progressive.webp) ![](http://i0.hdslb.com/bfs/article/87d5468248b385f1796b50693e5f8cf7cde9e0d7.png@566w_540h_progressive.webp)

这儿的 Type 类型 选择哪个都是没问题的，主要看是系统能不能执行，大致的意思就是生成一个 Url，让受害者去带上 7788 的参数去访问这个 Url，使被攻击机上线

![](http://i0.hdslb.com/bfs/article/1bc3014114bf98a7908bcecf08a1d7f65f543c36.png@942w_492h_progressive.webp) ![](http://i0.hdslb.com/bfs/article/34d5e12a0689b178fc4c6de46f8f82ea9cffb8c0.png@942w_714h_progressive.webp)

要先安装 CrossC2，具体得跳过了

设置一个监听端口，CrossC2 目前只支持 Beacon HTTPS

![](http://i0.hdslb.com/bfs/article/6b747011e377637b849e73a2c32200582848ce75.png@752w_837h_progressive.webp) ![](http://i0.hdslb.com/bfs/article/0e81ec09f4802e1616a115828941f2773a03b445.png@942w_698h_progressive.webp)

设置监听和 Linux/Mac 的型号 x86/x64 就 Ok 了

![](http://i0.hdslb.com/bfs/article/a984c1a9be0a8fd83b9a11f460cfeba71915a93b.png@942w_648h_progressive.webp)

点击 Build 就会生成这么个命令，把这个命令复制到要上线的 Linux 主机上  

![](http://i0.hdslb.com/bfs/article/cf30b275e42c2cb25c6426d47da39e6253e04123.png@504w_197h_progressive.webp) ![](http://i0.hdslb.com/bfs/article/9da7a9cc4de76643afaa97b06d67c77cc4d2b5c4.png@942w_195h_progressive.webp) ![](http://i0.hdslb.com/bfs/article/ad31f4c67a8b54f35281254bb8a191b92b9fc148.png@942w_707h_progressive.webp)

成功上线

网上还有个是利用命令

> ./genCrossC2.Linux 10.6.6.25（监听 IP） 443（监听端口） null null Linux（Linux/Mac） x86（x86/x32） test

大家可以试试

CS 主机上线是后面进一步的基操，没有上线啥都免谈。

老生常谈的，各位师傅切勿进行无授权渗透！