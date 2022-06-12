> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_44159028/article/details/118157559)

**目录**

[创建监听器 Listener](#t0)

[生成后门](#t1)

[Windows Executable & Windows Executable(S)](#t2)

**创建监听器 Listener**
==================

什么时候需要创建一个[监听](https://so.csdn.net/so/search?q=%E7%9B%91%E5%90%AC&spm=1001.2101.3001.7020)器了，比如我们拿到一个 shell。此时我们就需要用 cs 生成一个 payload 传入到目标服务器上来，这个 payload 要与服务器进行连接就需要服务器先开启一个监听，payload 才能找到服务器。

如下进行点击，然后在最下面点击 add，进行监听器的添加

![](https://img-blog.csdnimg.cn/2021062317192975.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0MTU5MDI4,size_16,color_FFFFFF,t_70)

页面配置信息如下

```
name：为监听器名字，可任意
payload：payload类型
HTTP Hosts: shell反弹的主机，也就是我们kali的ip（如果是阿里云，则填阿里云主机的公网ip）
HTTP Hosts(Stager): Stager的马请求下载payload的地址（一般也是和上面的ip填一样）
HTTP Port(C2): C2监听的端口
```

![](https://img-blog.csdnimg.cn/20210529134011694.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0MTU5MDI4,size_16,color_FFFFFF,t_70)

这里面有一个很重要的点就是选择 payload 类型

**payload 类型**

Beacon 是 Cobalt Strike 的 payload，用于建模高级攻击者，我们可以理解为 Beacon 就是生成的 payload。使用 Beacon 来通过 HTTP，HTTPS 或 DNS 出口网络。Beacon 很灵活，支持异步通信模式和交互式通信模式。异步通信效率缓慢：Beacon 会回连团队服务 器、下载其任务，然后休眠。交互式通信是实时发生的。 Beacon 的网络流量指标具有拓展性。可以使用 Cobalt Strike 的可拓展的 C2 语言来重新定义 Beacon 的通信。这允许你掩盖 Beacon 行动，比如使其流量看起来像其他的恶意软件，又或者将其流量掺入作为合法流量。

 目前有 8 种 payload，如下

![](https://img-blog.csdnimg.cn/20210529134113300.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0MTU5MDI4,size_16,color_FFFFFF,t_70)

内部的 Listener

*   windows/beacon_dns/reverse_dns_txt
*   windows/beacon_http/reverse_http
*   windows/beacon_https/reverse_https
*   windows/beacon_bind_tcp
*   windows/beacon_bind_pipe

外部的 Listener

*   windows/foreign/reverse_http
*   windows/foreign/reverse_https

External

*   windows/beacon_ext

Beacon 为内置的 Listener，即在目标主机执行相应的 payload，获取 shell 到 CS 上; 其中包含 DNS、HTTP、HTTPS、SMB。Beacon 可以选择通过 DNS 还是 HTTP 协议出口网络，你甚至可以在使用 Beacon 通讯过程中切换 HTTP 和 DNS。其支持多主机连接，部署好 Beacon 后提交一个要连回的域名或主机的列表，Beacon 将通过这些主机轮询。目标网络的防护团队必须拦截所有的列表中的主机才可中断和其网络的通讯。通过种种方式获取 shell 以后 (比如直接运行生成的 exe)，就可以使用 Beacon 了。

Foreign 为外部结合的 Listener，常用于 MSF 的结合，例如获取 meterpreter 到 MSF 上。

**Payload Staging**

作为背景信息，一个值得一提的主题是 payload staging （分阶段传送 payload ）。在很多攻击框架的设计中，解耦了攻击和攻击执行的内容。payload 就是攻击执行的内容。 payload 通常被分为两部分： payload stage 和 payload stager 。 stager 是一个小程序 ，通常是手工优化的汇编指令，用于 下载一个 payload stage 、把它注入内存，然后对其传达执行命令。这个过程被称为 staging （分阶段）。

staging （分阶段）过程在一些攻击行动中是必要的。很多攻击中对于能加载进内存并在成功漏洞利用 后执行的数据大小存在严格限制。这会极大地限制你的后渗透选择，除非你分阶段传送你的后渗透 payload。

Cobalt Strike 在它的用户驱动攻击中使用 staging （分阶段）。大多数这类项目在 Attacks → Packages 和 Attacks → Web Drive - by 选项下。使用什么样的 stager 取决于与攻击配对 的 payload 。比如， HTTP Beacon 有一个 HTTP stager 。 DNS Beacon 有一个 DNS TXT 记录 stager 。 不是所有的 payload 都有 stager 选项。没有 stager 的 Payload 不能使用这些攻击选项投递。 如果你不需要 payload staging （分阶段），通过在你的 C2 拓展文件里把 host_stage 选项设为 false，你可以关闭这个选项。这会阻止 Cobalt Strike 在其 web 和 DNS 服务器上托管 payload stage。这种设置有助于提升行为安全（避免反溯源），因为如果开启了 staging （分阶段），任何人都能连。

**HTTP Beacon** **和** **HTTPS Beacon** 默认设置情况下， HTTP 和 HTTPS Beacon 通过 HTTP GET 请求来下载任务。这些 Beacon 通过 HTTP POST 请求传回数据 要起一个 HTTP 或 HTTPS Beacon 监听器，通过 Cobalt Stike → Listeners 。点击 Add 按钮，选择 Beacon HTTP 作为你的 payload 选项。一般只需要填写如下的选择即可 ![](https://img-blog.csdnimg.cn/20210529134011694.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0MTU5MDI4,size_16,color_FFFFFF,t_70)

*   name： 为监听器名字，可任意
*   HTTP Hosts:  进行回连的主机 (shell 反弹的主机)，也就是我们 kali 的 ip（如果是阿里云，则填阿里云主机的公网 ip），可以设置多个
*   HTTP Hosts(Stager):  Stager 的马请求下载 payload 的地址（一般也是和上面的 ip 填一样）
*   HTTP Port(C2): payload 回连的端口
*   HTTP Port(Bind)：  字段指定你的 HTTP Beacon payload web 服务器绑定的端口。如果你要设置端口弯曲重定向器（例如，接受来自 80 或 443 端口的连接但将连接路由到团队服务器开在另一个端口上的连接，这样的重定向器），那么这些选 项会很有用。
*   HTTP Host Header ：值被指定了，会影响你的 HTTP stagers ，并通过你的 HTTP 通信。这个选项 使得通过 Cobalt Strike 利用域名前置变得更加容易
*   HTTP Proxy ： 字段旁边的 ... 按钮来为此 payload 指定一个显式的代理配置

**生成后门**
========

设置了监听后，就需要生成后门 payload 了。

 如下点击 Attacks -> Packages

![](https://img-blog.csdnimg.cn/20210529135318453.png)

这里的 Attacks 有几种，如下

*   HTML Application            生成一个恶意 HTML Application 木马，后缀格式为. hta。通过 HTML 调用其他语言的应用组件进行攻击，提供了可执行文件 、PowerShell、VBA 三种方法。
*   MS Office Macro              生成 office 宏病毒文件;
*   Payload Generator          生成各种语言版本的 payload，可以生成基于 C、C#、COM Scriptlet、Java、Perl、PowerShell、Python、Ruby、VBA 等的 payload
*   Windows Executable        生成 32 位或 64 位的 exe 和基于服务的 exe、DLL 等后门程序
*   Windows Executable(S)   用于生成一个 exe 可执行文件，其中包含 Beacon 的完整 payload，不需要阶段性的请求。与 Windows Executable 模块相比，该模块额外提供了代理设置，以便在较为苛刻的环境中进行渗透测试。该模块还支持 powershell 脚本，可用于将 Stageless Payload 注入内存

**Windows Executable & Windows Executable(S)**
----------------------------------------------

这两个模块直接用于生成可执行的 exe 文件或 dll 文件。Windows Executable 是生成 Stager 类型的马，而 Windows Executable(S) 是生成 Stageless 类型的马。那 Stager 和 Stageless 有啥区别呢?

*   Stager 是分阶段传送 Payload。分阶段啥意思呢? 就是我们生成的 Stager 马其实是一个小程序，用于从服务器端下载我们真正的 shellcode。分阶段在很多时候是很有必要的，因为很多场景对于能加载进内存并成功漏洞利用后执行的数据大小存在严格限制。所以这种时候，我们就不得不利用分阶段传送了。如果不需要分阶段的话，可以在 C2 的扩展文件里面把 host_stage 选项设置为 false。
*   而 Stageless 是完整的木马，后续不需要再向服务器端请求 shellcode。所以使用这种方法生成的木马会比 Stager 生成的木马体积要大。但是这种木马有助于避免反溯源，因为如果开启了分阶段传送，任何人都能连接到你的 C2 服务器请求 payload，并分析 payload 中的配置信息。在 CobaltStrike4.0 及以后的版本中，后渗透和横向移动绝大部分是使用的 Stageless 类型的木马。

这里生成 Windows Executable 程序，如下，我们创建了监听器后生成的时候就需要先选择。然后选择保存的路径

![](https://img-blog.csdnimg.cn/20210529140547526.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0MTU5MDI4,size_16,color_FFFFFF,t_70)

这样当我们双击生成的 exe 程序后 cs 就会直接上线。可以同时连接多个相同的 exe payload。如下会话表

**会话表介绍**

会话表展示了哪些 Beacon 回连到了这台 Cobalt Strike。Beacon 是 Cobalt Strike 用于模拟高级

威胁者的 payload。在这里，你将看到每个 Beacon 的外网 IP 地址、内网 IP 地址、该 Beacon 的出口监听器、此 Beacon 最后一次回连的时间，以及其他信息。每一行最左边是一个图标，用于说明被害目 标的操作系统。如果此图标是红色的、并且带有闪电，那么说明此 Beacon 运行在管理员权限的进程中。一个褪色的图标意味着此 Beacon 会话被要求离开并且它接受了此命令。

![](https://img-blog.csdnimg.cn/2021062317020816.png)

接着可以执行 beacon 命令，相当于 cmd 命令 **beacon 命令**

 在已经上线的服务器上右击 ==> 打开 Beacon，在里面可以执行各种命令

![](https://img-blog.csdnimg.cn/20210531122435455.png)

在 cs 中默认的心跳时间是 60s，我们可以用 sleep 命令来更改心跳时间（心跳时间是指上线的服务器和 cs 服务器的通信时间）。

![](https://img-blog.csdnimg.cn/20210531123120570.png)

所以如果我们不修改的话，那么我们每次执行命令服务器要好很久才会返回。心跳时间很长就会响应的时间很慢，但也不能设置的很短，不让很容易会被监测到与 cs 通信的流量，具体多少可以根据测试环境自己来设置。这里设置为 5，这样时间到了 5s，就会返回结果

![](https://img-blog.csdnimg.cn/20210531123313882.png)

Beacon 就是一个命令解释器，供我们来执行命令。在 Beacon 中执行 cmd 命令时需要在前面加上 shell，例如：

shell whoami/systeminfo/netstat -an 

![](https://img-blog.csdnimg.cn/20210531123525312.png)

我们可以用 help 来查看更过的命令

```
beacon> help
 
Beacon Commands
===============
 
    Command                   Description
    -------                   -----------
    browserpivot              注入受害者浏览器进程
    bypassuac                 绕过UAC
    cancel                    取消正在进行的下载
    cd                        切换目录
    checkin                   强制让被控端回连一次
    clear                     清除beacon内部的任务队列
    connect                   Connect to a Beacon peer over TCP
    covertvpn                 部署Covert VPN客户端
    cp                        复制文件
    dcsync                    从DC中提取密码哈希
    desktop                   远程VNC
    dllinject                 反射DLL注入进程
    dllload                   使用LoadLibrary将DLL加载到进程中
    download                  下载文件
    downloads                 列出正在进行的文件下载
    drives                    列出目标盘符
    elevate                   尝试提权
    execute                   在目标上执行程序(无输出)
    execute-assembly          在目标上内存中执行本地.NET程序
    exit                      退出beacon
    getprivs                  Enable system privileges on current token
    getsystem                 尝试获取SYSTEM权限
    getuid                    获取用户ID
    hashdump                  转储密码哈希值
    help                      帮助
    inject                    在特定进程中生成会话
    jobkill                   杀死一个后台任务
    jobs                      列出后台任务
    kerberos_ccache_use       从ccache文件中导入票据应用于此会话
    kerberos_ticket_purge     清除当前会话的票据
    kerberos_ticket_use       从ticket文件中导入票据应用于此会话
    keylogger                 键盘记录
    kill                      结束进程
    link                      Connect to a Beacon peer over a named pipe
    logonpasswords            使用mimikatz转储凭据和哈希值
    ls                        列出文件
    make_token                创建令牌以传递凭据
    mimikatz                  运行mimikatz
    mkdir                     创建一个目录
    mode dns                  使用DNS A作为通信通道(仅限DNS beacon)
    mode dns-txt              使用DNS TXT作为通信通道(仅限D beacon)
    mode dns6                 使用DNS AAAA作为通信通道(仅限DNS beacon)
    mode http                 使用HTTP作为通信通道
    mv                        移动文件
    net                       net命令
    note                      备注       
    portscan                  进行端口扫描
    powerpick                 通过Unmanaged PowerShell执行命令
    powershell                通过powershell.exe执行命令
    powershell-import         导入powershell脚本
    ppid                      Set parent PID for spawned post-ex jobs
    ps                        显示进程列表
    psexec                    Use a service to spawn a session on a host
    psexec_psh                Use PowerShell to spawn a session on a host
    psinject                  在特定进程中执行PowerShell命令
    pth                       使用Mimikatz进行传递哈希
    pwd                       当前目录位置
    reg                       Query the registry
    rev2self                  恢复原始令牌
    rm                        删除文件或文件夹
    rportfwd                  端口转发
    run                       在目标上执行程序(返回输出)
    runas                     以另一个用户权限执行程序
    runasadmin                在高权限下执行程序
    runu                      Execute a program under another PID
    screenshot                屏幕截图
    setenv                    设置环境变量
    shell                     cmd执行命令
    shinject                  将shellcode注入进程
    shspawn                   生成进程并将shellcode注入其中
    sleep                     设置睡眠延迟时间
    socks                     启动SOCKS4代理
    socks stop                停止SOCKS4
    spawn                     Spawn a session 
    spawnas                   Spawn a session as another user
    spawnto                   Set executable to spawn processes into
    spawnu                    Spawn a session under another PID
    ssh                       使用ssh连接远程主机
    ssh-key                   使用密钥连接远程主机
    steal_token               从进程中窃取令牌
    timestomp                 将一个文件时间戳应用到另一个文件
    unlink                    Disconnect from parent Beacon
    upload                    上传文件
    wdigest                   使用mimikatz转储明文凭据
    winrm                     使用WinRM在主机上生成会话
    wmi                       使用WMI在主机上生成会话
    ﻿argue                     进程参数欺骗
```