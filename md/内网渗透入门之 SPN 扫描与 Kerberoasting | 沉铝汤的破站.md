> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [chenlvtang.top](https://chenlvtang.top/2022/02/21/%E5%86%85%E7%BD%91%E6%B8%97%E9%80%8F%E5%85%A5%E9%97%A8%E4%B9%8BSPN%E6%89%AB%E6%8F%8F/)

> IS LIFE ALWAYS THIS HARD, OR IS IT JUST WHEN YOU'RE A KID

[](#0x00-前言 "0x00 前言")0x00 前言
-----------------------------

* * *

虽然之前的票据传递还有一些问题没有解决，但囿于目前对于内网渗透还没有形成一个整体的认知，所以对于那些问题就先放一放，先继续学习其他的相关知识。本文将学习 SPN 的基本概念，并学习利用 SPN 进行扫描，来实现在内网中以较小动静进行信息收集并为之后的 Kerberoasting（注意这个单词和 Kerberos 的区别）作准备。

[](#0x01-SPN "0x01 SPN")0x01 SPN
--------------------------------

* * *

### [](#简介 "简介")简介

SPN，全名为：Service Principal Names，即 “服务主体名称”。它是域中服务的唯一标识，每个 Kerberos 服务都必须要有一个 SPN，服务在加入域时，会自动注册一个 SPN。 如果未进行 SPN 注册或注册失败（名称不唯一），则 Windows 安全层无法确定与 SPN 关联的帐户，因而无法使用 Kerberos 身份验证。

### [](#格式 "格式")格式

SPN 的格式为：`<service class>/<host>:<port>/<service name>`，其中 service class 和 host 为必需参数。

*   service class 为服务类型名称，你可以使用除 “/” 之外的任何名称（因为 SPN 使用它作为分隔符），只需要保证它是唯一的名称，但是一般建议使用通用名称，如 “www”，“ldap” 等
*   host 为运行服务的主机名，可以使用 DNS 名（如：os.hacker.com）或 NetBIOS 名（如：os），但要注意的是因为 NetBIOS 名可能会在林中不唯一，会导致 SPN 注册失败。
*   host 为可选参数，同一服务在同一 host 上运行时，使用此来加以区别。服务仅使用默认端口时（如 80），可以省略。
*   service name 为服务实例名称，不太重要，微软有个这样的例子: MyDBService/host1/CN=hrdb,OU=mktg,DC=example,DC=com

一个 SPN 命名实例：`MySQLSvc/os.hacker.com:3306` 或 `MySQLSvc/hacker`等。

### [](#手动注册SPN "手动注册SPN")手动注册 SPN

1. 使用 Windows 内置的 Setspn 工具可以读取、修改和删除 SPN 目录属性。

注：这里有的微软文档使用的是 - A 参数，而 setspn 文档中又使用的是 - s 参数，并且无 - A 参数，但两者似乎都可以。另外参数似乎大小写不敏感。

最后面接的是对应的账户名，此处使用的是域下账户格式，其支持的用户与对应格式如下：

*   username@domain 或 domain\username（适用于域用户帐户）
*   machine$@domain 或 host\FQDN（适用于计算机域帐户，如 Local System 或 NETWORK SERVICES）。

### [](#SPN查询 "SPN查询")SPN 查询

借助 setspn 我们可以查询对应的 SPN，并会返回对应的账户：

-T 指定了目标域，-Q 指定对应服务，*/* 表示所有服务。

扫描结果实际上是使用了 ldap 中的记录，当不存在 ldap 服务时，就会报错：

![](https://cdn.jsdelivr.net/gh/chenlvtang/picbed/img/202202232135862.png)

正确查询结果如下（因为还未搭建环境，所以借用 Y4er 师傅的图片）：

![](https://cdn.jsdelivr.net/gh/chenlvtang/picbed/img/202202232138916.png)

其中 CN=Users 的为域用户账号，CN=Computers 的为计算机域帐户（不可远程连接）。

[](#0x02-SPN扫描 "0x02 SPN扫描")0x02 SPN 扫描
---------------------------------------

* * *

### [](#简介-1 "简介")简介

SPN 扫描，即利用 SPN 查询，来收集内网中所运行的服务。这种扫描相对于端口扫描来说，动静更小（对域控制器发起 LDAP 查询，这是正常 kerberos 票据行为的一部分，因此查询 SPN 的操作很难被检测）, 且避免了端口扫描的盲目性。

### [](#其他工具 "其他工具")其他工具

除了上文 Windows 内置的 setspn.exe 外，还可以使用如下的工具或脚本（未全部列出）：

1.  使用 kerberoast 项目中的 GetUserSPNs.vbs（或 ps 版本），需要自己下载：[kerberoast/GetUserSPNs.vbs](https://github.com/nidem/kerberoast/blob/master/GetUserSPNs.vbs)
    
2.  PowerView, [PowerSploit/PowerView.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)
    

[](#0x03-Kerberoasting攻击 "0x03 Kerberoasting攻击")0x03 Kerberoasting 攻击
---------------------------------------------------------------------

* * *

### [](#简介-2 "简介")简介

Kerberoasting 攻击是 Tim Medin 在 DerbyCon 2014 上发布的一种域口令攻击方法，Tim Medin 同时发布了配套的攻击工具 kerberoast。此后，不少研究人员对 Kerberoasting 进行了改进和扩展，在 GitHub 上开发发布了大量工具，使得 Kerberoasting 逐渐发展成为域攻击的常用方法之一。

通俗来说，就是针对 Kerberos 认证中的哈希密码进行爆破，进而对 DC 进行攻击的一种方法。

### [](#原理 "原理")原理

在 Kerberos 认证中，我们知道最后会返回一个使用 Service 的 NTML-Hash 加密的 ST。而域中所有主机都能通过查询 SPN 获取到服务对应的账户，且任何已经认证的用户都可以请求 ST。

所以这就给了我们一个通过 SPN 寻找账户并请求访问获得对应 ST 的机会，这有什么用呢？

ST 是由 Service HASH 加密而成，所以我们可以对其进行爆破，这样就能获得其明文密码。(首先应该是要爆破出加密的密钥，然后又要爆破密钥出明文，关于其中的过程，以后我再来分析，密码学基础有点差……)

### [](#利用 "利用")利用

1. 寻找有价值的 SPN 服务。什么是有价值呢？**可以远程连接，高权限**，因为计算机域帐户不可以远程连接，所以我们目标一般都是域用户。

*   工具有很多种，直接谷歌搜索就能找到对应工具了，这里使用上文 SPN 扫描中的工具即可。

2. 请求 TGS

*   请求某个服务的 TGS (powershell)
    

3. 导出票据，并爆破明文

方法一：

*   使用 mimikatz 导出
    
*   使用 kerberoast 工具中的脚本爆破
    

方法二：

*   使用 Empire 中的 Invoke-Kerberoast.ps1([Empire/Invoke-Kerberoast.ps1](https://github.com/EmpireProject/Empire/blob/6ee7e036607a62b0192daed46d3711afc65c3921/data/module_source/credentials/Invoke-Kerberoast.ps1))，导出为 hashcat 格式
    
    只要显示 hash 的话，使用：
    
*   将 hash 保存为 hash.txt，然后使用 hashcat(一般 kali 中有) 来破解
    

（还有其他很多类似的工具, 如 Rubeus）

[](#0x0-参考 "0x0 参考")0x0 参考
--------------------------

* * *

[为 Kerberos 连接注册服务主体名称 - SQL Server | Microsoft Docs](https://docs.microsoft.com/zh-cn/sql/database-engine/configure-windows/register-a-service-principal-name-for-kerberos-connections?view=sql-server-ver15#spn-formats)

[Name Formats for Unique SPNs - Win32 apps | Microsoft Docs](https://docs.microsoft.com/en-us/windows/win32/ad/name-formats-for-unique-spns)

[【域渗透】SPN 扫描利用 | RcoIl 的窝](https://rcoil.me/2019/06/%E3%80%90%E5%9F%9F%E6%B8%97%E9%80%8F%E3%80%91SPN%20%E6%89%AB%E6%8F%8F%E5%88%A9%E7%94%A8%2F)

[Kerberos 协议之 Kerberoasting 和 SPN - Y4er 的博客](https://y4er.com/post/kerberos-kerberoasting-spn/)

[内网安全: Kerberoasting 攻击和 SPN 服务 (zeo.cool)](https://zeo.cool/2020/05/27/%E5%86%85%E7%BD%91%E5%AE%89%E5%85%A8!Kerberoasting%E6%94%BB%E5%87%BB%E5%92%8CSPN%E6%9C%8D%E5%8A%A1/)

[域渗透——Kerberoasting (3gstudent.github.io)](https://3gstudent.github.io/%E5%9F%9F%E6%B8%97%E9%80%8F-Kerberoasting)