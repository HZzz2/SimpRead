> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.freebuf.com](https://www.freebuf.com/articles/neopoints/368838.html)

> 本文主要分享国外一名网络安全工作者的成长经验（内含大量学习资源和学习工具）。

本文作者：[Hackergod00001](https://hackergod00001.medium.com/?source=post_page-----dfe05a09a7a0--------------------------------)

> 本文主要分享自己的网络安全成长经验，从最开始的安全小白（就跟游戏中的 NPC 一样）到后面小有成就，成为安全研究员，一路走来，有很多感悟，有很多坚持，分享给有需要的朋友。**（内含大量学习资源和学习工具）**

**最初的安全之旅：起于兴趣，终于热爱**
---------------------

在我的成长过程中，我一直对计算机的工作原理很着迷。我会花很多时间来研究它们，试图了解它们的复杂性并挑战它们的极限。但我不知道，这种迷恋最终会把我引向网络安全和黑客的道路。

这一切都始于我十几岁的时候，当时我对科幻和动作电影产生了浓厚的兴趣，如《米奇病毒》、《天才》、《阿比曼尤的回归》、《黑客》、《无法追踪》、《Shree》、《我是谁：没有绝对安全的系统》、《创：战纪》、《黑客追逐令》、《黑客》、《黑客帝国》以及许多其他电影。这些电影激起了我对黑客世界的兴趣。

最初，是好奇心把我吸引到黑客领域。我想了解黑客是如何绕过安全措施，未经授权进入计算机系统的。我开始在网上阅读文章和观看视频，试图了解黑客的基本知识。

上学时看了《机器人先生》（网络剧）后，我对白帽的兴趣更浓了。2019 年 12 月，在我的计算机科学学士 / 技术学士的第一年，我在 IIT Bombay 遇到了一个黑客讲习班，我决定去参加它。在那里，我第一次看到了《机器人先生》中的 Kali Linux，它与 Windows 有很大的不同。我惊讶地发现，除了 Windows 之外，真的还有其他操作系统，比如 Linux（Kali Linux、Ubuntu、Debian）、Mac 和其他。

在研讨会上，我听到了 SQL、SQL 注入、Blink SQLi、XXS、SSRF、CSRF、Cookies、操作系统、路由器、IP 地址、IPV4、IPV6、MAC 地址、数据库、加密、解密、TOR 浏览器、Burp Suit、VPN、NMAP、终端、钓鱼等等术语。

然而，在当时，由于我缺乏经验和技术知识，我无法与这些术语联系起来，因为这是我第一次参加技术研讨会。后来，我也是家里第一个攻读计算机工程学士学位的人，在那一刻，我意识到这将是我长久以来一直热衷的事情。

随着兴趣的增长，我开始学习编程，并在自己的计算机系统上试验黑客技术。我下载了 Kali Linux 等工具，并在虚拟机上练习使用它们。我花了几个小时试图侵入自己的系统，寻找可以利用的漏洞。这是一个激动人心的经历，我被迷住了。

然而，我很快就意识到，黑客行为不仅仅是为了好玩而闯入系统。稍有不慎，它就会产生严重的影响。我想用我的技能来做好事，我决定追求网络安全的职业，在那里我可以利用我的知识和技能帮助保护系统和网络免受恶意攻击。

为了实现这一目标，我开始学习尽可能多的网络安全知识。我阅读书籍，观看在线课程，并加入在线社区，在那里我可以向该领域的专家学习。我还开始参加 CTF 比赛，这使我能够在一个安全和合法的环境中练习我的技能。

我在网络安全领域走过的道路并非没有挑战。有的时候，我对需要学习的大量信息感到不知所措。有的时候，我感到灰心丧气，因为我还没能找到任何漏洞或赚取任何赏金。但我坚持了下来，知道每一次挫折都是学习和成长的机会。

通过我的旅程，我认识到，黑客和网络安全不仅仅是技术技能。它们也是关于心态、耐心和坚持的。要想解决复杂的问题并在面对挫折时保持动力，需要某种心态。但只要有正确的心态，一切皆有可能。

**真正的旅程：始终如一，不断尝试**
-------------------

前面我分享了我最初的网络安全之旅——从我小时候对技术的迷恋到我参加的第一个白帽研讨会。随着我在大学里的进步，我继续探索网络安全领域，并开始了解这个领域所涉及的各种细微差别和错综复杂的问题。

在大学二年级初期，我开始学习 C 语言编程，并开始在 HackerRank 等编码平台上进行练习，以提高我的编程技能。同时，我也开始探索计算机网络和 Nmap、Burpsuite 和 Kali Linux 等工具。然而，随着新冠疫情的来临，我的旅程遇到了一个重大障碍，这迫使我的大学转向在线形式。

尽管遇到了挫折，我还是坚持下来，继续我的学习之旅。感谢我父母的支持，我获得了我第一台规格超好的笔记本电脑，并开始在 LinkedIn、Twitter 和 Instagram 等社交媒体平台上搜索网络安全的实习和社区。我在 YouTube 和 Twitter 上发现了 CybersecurityMeg、nahamsec、RanaKhalil101、InsiderPhD、_JohnHammond、PhillipWylie、huskyhacks、jhaddix、STOKfredrik、alh4zr3d3、Tib3rius、FarahHawa、snyff、corgi、hAPI_hacker、thecybermentor 等内容创作者，他们通过 CTF 比赛、奖品和任务来活跃网络安全社区。

> 在这一过程中，我与他们中的一些人在 LinkedIn 和 Instagram 上进行了联系，如 Deepak Kumar 博士（D3）、OoPpSs 先生、Rakshit Tandon 先生和 Anand Prakash，同时还进行了一些远程实习，如平面设计师实习生 @Revolux Pvt. Ltd.，Python 编程与 Python 设计实习生 @Cyberace Infovision Pvt. Ltd.，前端开发实习生 @Team Abadha, Fr. CRCE，PHP 后台开发实习生 & 全栈开发实习生 @Web Shine Tech Pvt. Ltd.，GPCSSI 2021 实习生 @Gurugram Police Cyber Security Summer Internship，区块链实习生 & 网络安全实习生 @Pie Infocomm Pvt. Ltd.，网络安全实习生 @smartknower，渗透测试员实习生和网络安全工程师实习生 @Virtually Testing Foundation (VTF)。

在我 2020 年 4 月至 2022 年 6 月期间进行的这 11 次实习中，只有 8 次我觉得给我提供了宝贵的实践经验和实际知识，而其他的则没有什么用。尽管如此，我仍然致力于我的学习之旅，并继续寻找新的机会来提升我在网络安全领域的技能和知识。

在我学习的过程中，我意外地在 YouTube 上遇到了 Cryptoknight01 和她的妹妹，她们在 20 多岁的时候就实现了首次尝试破解 OSCP 的惊人壮举。我感到很受鼓舞，于是在 Udemy 上报名参加了 Zsecurity 的 Zaid Bhat 和 Internshala 的前两个白帽课程。此外，我还购买了几本与黑客有关的书籍。不幸的是，我很快就意识到，尽管这些课程对新手来说是有益的，但它们未能使我具备处理真实世界情况的必要技能。直到我偶然发现 NahamSec 的 Udemy 课程 Intro-to-bug-bounty-by-nahamsec，我终于找到了一个资源，才能在实践中派上用场。

> 随后，我开始了研究阶段，在此期间，我发现了大量可以免费使用的资源。我申请了 Coursera、Udacity 和 Edx 以及 TryHackMe、PentesterLab、HackTheBox、PortSwigger、VulnHub、APIsec 大学、TCMSecurityAcademy 和 RootMe 等平台的各种认证课程。此外，我还阅读了各种博客，并参加了各种 CTF 挑战赛，如 PicoCTF、NahamCon CTF、CTF Times、HTB 大学 CTF 和 2021 年底的 Advent of Cyber，参加了问答活动，并对 HackerOne、Bugcrowd、Intigriti 和 Synack 等漏洞赏金平台进行了研究。

随着我对网络安全领域深入研究，我发现自己被大量的资源所淹没，让我不知道该从哪里开始，该优先考虑什么。这导致我失去了动力，有一种迷失的感觉，我浪费了近两年时间，漫无目的地申请各种项目，却没有取得任何进展。为了聚焦和专注起来，我休息了几个月，放慢了我的学习进程。后来，我发起了一个为期 100 天的漏洞赏金挑战，但由于一些原因，我未能完成它。

在我追求网络安全事业的某个阶段，疑虑开始爬上心头，让我怀疑自己的能力，怀疑自己是否适合这个领域。但这是我真正的旅程开始的地方，我觉得我被困在一个地方，没有任何进展。更糟糕的是，我还在处理个人和财务问题。为了克服这些障碍，我暂停了对黑客技术的学习，专注于整理我的个人生活，同时工作以支持我的家庭。

然而，我对学习更多黑客知识的兴趣并没有完全消退。有一天，我偶然刷到了 NahamSec Twitch 视频，他不仅解释了黑客概念，还谈到了应对压力、培养耐心和保持一致性。这重新激发了我的意志力，给了我希望，我重新开始了我的学习之旅，一路上取得了进展。最终在 2022 年 7 月 29 日，我在 HackerOne 上发现了我的前三个 bug，这大大增强了我的信心。

### 我所学到的就是始终如一，不断尝试

![](https://image.3001.net/images/20230607/1686149348_648098e4c5aa937a609a8.png!small?1686149352352)

邓宁 - 克鲁格效应，这是一个很好的模型，用来解释一种让人们高估自己能力的认知偏差。特别是在他们没有什么经验的领域。当刚接触某一学科的人了解到与该学科相关的新技能、工具或术语时，会给他们带来信心的提升。

![](https://image.3001.net/images/20230607/1686149395_64809913028feb62da9d9.png!small?1686149398587)

邓宁 - 克鲁格效应的图形视图

> 注意：每个人都会经历这个阶段，甚至我也经历过这个阶段，但应对这一挑战的唯一关键咒语是 "坚持"。因此，在你所学的和你所做的方面要保持一致。

随着我的不断学习，我意识到沟通、协作和团队合作等软技能在网络安全领域至关重要，特别是在与客户合作或作为团队的一部分工作时。我开始参加网络安全活动和会议，与该领域的其他专业人士建立联系，了解最新的趋势和技术。在我的进步和新发现的激情的鼓励下，我建立了自己的社区，名为 HAWK-I CRCE，帮助其他人找到他们在 bug 赏金、渗透测试和网络安全的道路。在 HAWK-I 和 GDSC 的帮助下，我举办了几次关于网络安全、漏洞赏金、CTF、网络和 API 安全、AWS 云计算和 AWS 云安全的研讨会。

最后，在 2023 年 4 月，也就是我大学的最后一年，我组织了孟买有史以来第一次综合的网络安全会议——SummerCon CRCE 2023。行业专家在会上分享了他们对网络安全、威胁范围、如何在网络安全领域找到工作的见解，以及为产生重大影响而采取的黑客行为。

![](https://image.3001.net/images/20230607/1686149415_64809927c3aedc61ff05b.png!small?1686149420738)

SummerCon CRCE 2023 回顾

回顾过去，我意识到有的时候我感到不知所措和迷茫。但是，通过保持专注，在挑战中坚持不懈，并从错误中学习，我能够成长，并获取一个网络安全专业人士的技能。今天，我很自豪能成为这个充满活力和令人兴奋的行业的一份子，我期待着继续我的旅程，对世界产生积极的影响。

总之，虽然我最初从别人的成功中寻找灵感，但我了解到，拥有一个结构化和可管理的学习方法对于避免在浩瀚的信息和资源海洋中迷失方向至关重要。因此，在我经验分享的最后一部分，我将分享我学习 VAPT/Pentesting/Bug Bounty 的粗略结构和方法，其他人可能会觉得有帮助。

**终极旅程：“3 阶段过程” 学习方法分享**
------------------------

接下来，我将分享我学习 VAPT、Bug Bounty hunting 和玩 CTF 的方法 "3 阶段过程"，我希望它对你们开始黑客之旅也有帮助。那么，废话不多说，让我们迅速进入这 3 个阶段的过程！

### **第一阶段：巩固基础阶段**

首先，我们最重要的任务是巩固我们的根基。我们将一起深入研究迷人的网络世界，揭开网络的神秘面纱，并掌握安全的基本原则。接下来，为了让大家更好地理解，我把如何巩固的 9 个步骤分享给大家。

**第 1 步：**

1. 全面了解计算机基础知识及其工作原理

2. 了解操作系统（如 Linux、Windows 和 Mac）如何运行，并学习如何运行 Kali Linux / Ubuntu / Parrot。

> **资源：**
> 
> 1. [https://www.youtube.com/watch?v=q7tlgZg4Q1o&list=PLWKjhJtqVAbmfoj2Th9fvxhHIeqFO7wOy](https://www.youtube.com/watch?v=q7tlgZg4Q1o&list=PLWKjhJtqVAbmfoj2Th9fvxhHIeqFO7wOy)（关于计算机科学的所有内容）
> 
> 2. [https://youtu.be/8mAITcNt710](https://youtu.be/8mAITcNt710)（哈佛 CS50 — 完整的计算机科学大学课程）
> 
> 3. [https://www.youtube.com/@freecodecamp/playlists](https://www.youtube.com/@freecodecamp/playlists)(freecodecamp)
> 
> 4. [https://www.youtube.com/@noobsanetworkchuckpodcast3009/videos](https://www.youtube.com/@noobsanetworkchuckpodcast3009/videos)
> 
> 5. [https://www.youtube.com/@davidbombal/playlists](https://www.youtube.com/@davidbombal/playlists)
> 
> 6 . [https://www.youtube.com/watch?v=AnwgxRtWXLI&list=PLhfrWIlLOoKMe1Ue0IdeULQvEgCgQ3a1B](https://www.youtube.com/watch?v=AnwgxRtWXLI&list=PLhfrWIlLOoKMe1Ue0IdeULQvEgCgQ3a1B)
> 
> **书籍：**
> 
> 1. 操作系统，9e 平装本 — 作者 [William Stallings](https://www.amazon.in/William-Stallings/e/B000APXR9Q/ref=dp_byline_cont_book_1)
> 
> 2. [https://nostarch.com/linuxbasicsforhackers](https://nostarch.com/linuxbasicsforhackers)
> 
> **工具：**
> 
> 1. Kali Linux — [https://www.kali.org/downloads/](https://www.kali.org/downloads/)
> 
> 2. Ubuntu — [https://ubuntu.com/download/desktop](https://ubuntu.com/download/desktop)
> 
> 3. Parrot — [https://www.parrotsec.org/download/](https://www.parrotsec.org/download/)

**第 2 步：****了解有关网络基础知识和管理程序的所有信息**

> **资源：**
> 
> 1. [https://www.youtube.com/watch?v=4Kho3Eeyx1U&list=PLLKT__MCUeiyUKmYaakznsZeU4lZYwt_j](https://www.youtube.com/watch?v=4Kho3Eeyx1U&list=PLLKT__MCUeiyUKmYaakznsZeU4lZYwt_j)
> 
> 2. [https://youtu.be/qiQR5rTSshw](https://youtu.be/qiQR5rTSshw)
> 
> 3. [https://www.youtube.com/@PracticalNetworking](https://www.youtube.com/@PracticalNetworking)
> 
> **博客：**
> 
> 1. [https://www.vmware.com/topics/glossary/content/hypervisor.html](https://www.vmware.com/topics/glossary/content/hypervisor.html)
> 
> **书籍：**
> 
> 1. AICTE 推荐 | 计算机网络 | 作者：Pearson 平装本 — 作者：[Tanenbaum](https://www.amazon.in/s/ref=dp_byline_sr_book_1?ie=UTF8&field-author=Tanenbaum&search-alias=stripbooks)
> 
> **工具：**
> 
> 1. [https://customerconnect.vmware.com/en/downloads/info/slug/desktop_end_user_computing/vmware_workstation_player/17_0](https://customerconnect.vmware.com/en/downloads/info/slug/desktop_end_user_computing/vmware_workstation_player/17_0)(VMware)
> 
> 2. [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)(VirtualBox)

**第 3 步：学习密码学基础知识，如加密、解密、编码、解码、散列等**

> **书籍：**
> 
> 1. 密码学与网络安全 | 第 4 版平装本 — [Atul Kahate](https://www.amazon.in/s/ref=dp_byline_sr_book_1?ie=UTF8&field-author=Atul+Kahate&search-alias=stripbooks)
> 
> 2. [https://nostarch.com/seriouscrypto](https://nostarch.com/seriouscrypto)
> 
> **工具：**
> 
> 1. [https://gchq.github.io/Cyber](https://gchq.github.io/CyberChef/)​​Chef/（Cyber​​chef 在现实生活中实践和使用的最佳工具）

**第 4 步：完成以下两个课程**

> 1. 首先完成由 Angela Yu 博士在 Udemy 上完成[的完整 2023 Web 开发训练营](https://www.udemy.com/course/the-complete-web-development-bootcamp/)
> 
> **资源：**[https://www.udemy.com/course/the-complete-web-development-bootcamp/](https://www.udemy.com/course/the-complete-web-development-bootcamp/)
> 
> 2、参加 NahamSec 的 [Bug Bounty Hunting 和 Web Application Hacking](https://www.udemy.com/course/intro-to-bug-bounty-by-nahamsec/) 课程
> 
> **资源：**[https://www.udemy.com/course/intro-to-bug-bounty-by-nahamsec/](https://www.udemy.com/course/intro-to-bug-bounty-by-nahamsec/)

**第 5 步：了解 HTTP 基础知识、REST 和 DNS**

> **资源：**
> 
> **1.** [https://www.freecodecamp.org/news/http-and-everything-you-need-to-know-about-it/](https://www.freecodecamp.org/news/http-and-everything-you-need-to-know-about-it/)(Freecodecamp)
> 
> 2. [https://portswigger.net/burp/documentation /desktop/http2/http2-basics-for-burp-users](https://portswigger.net/burp/documentation/desktop/http2/http2-basics-for-burp-users)(portswigger)
> 
> 3. [http://www.steves-internet-guide.com/dns-guide-beginners/](http://www.steves-internet-guide.com/dns-guide-beginners/)
> 
> 4. [https://rapidapi.com/learn /rest](https://rapidapi.com/learn/rest)

**第 6 步：了解所有关于 Recon 的信息**

> **资源：**
> 
> 1. Pesterstrlab（免费练习）：[https://www.pentesterlab.com/badges/recon](https://www.pentesterlab.com/badges/recon)
> 
> 2. NahamSec 的 Twitch（所有关于 Recon）：[https://www.twitch.tv/nahamsec](https://www.twitch.tv/nahamsec)
> 
> 3. 你应该做什么侦察之后？！ [https://youtu.be/A6zQV9e2S1M](https://youtu.be/A6zQV9e2S1M)

**第 7 步：**

1. 了解 JavaScript 和 Bash 脚本（**资源：**[**代码学院**](https://www.codecademy.com/catalog)）

2. 学习任何一门编程语言（选择你熟悉的任何一种…… 对我来说，我选择了 python）

> **语言：**C、C#、C++、Java、Python、Rust、Go 等
> 
> **资源：**
> 
> 1. [https://www.youtube.com/@freecodecamp/playlists](https://www.youtube.com/@freecodecamp/playlists)(Freecodecamp)
> 
> 2. [https://vickieli.dev/bash%20scripting/bash-intro/ 3.](https://vickieli.dev/bash%20scripting/bash-intro/) [Hackerrank](https://www.hackerrank.com/) 和 [leetcode](https://leetcode.com/)

**第 8 步：全面了解数据库（MySQL 和 NoSQL）**

MySQL 和 NoSQL 是有助于与数据库交互的查询语言（即通过数据存储进行交互）

> **资源：**
> 
> 大量用于理论的 **YouTube 视频**和用于实践的 **Hackerrank 问题**

**第 9 步：加入一些信息安全社区学习，并在 Instagram、Twitter 和 YouTube 上关注网络安全内容创作者**

> **1. Twitter：**Cyber​​securityMeg、nahamsec、rana__khalil、InsiderPhD、_JohnHammond、PhillipWylie、huskyhacks、jhaddix、STOKfredrik、alh4zr3d3、Tib3rius、FarahHawa、snyff、corgi、hAPI_hacker、thecybermentor、vickieli7 2. Youtube：@TC 安全学院， @NahamSec, @RanaKhalil101, @InsiderPhD, @_JohnHammond, @PhillipWylie, @huskyhacks, @jhaddix, @STOKfredrik, @alh4zr3d3, @Tib3rius, @FarahHawa, @Cyber​​securityMeg, @VickieLiDev, @david bombal 3. Di scord: HTB, Hackerone, Try 、TCMsecurity、redteamvillage、CSI Linux、Nahamsecs discord 频道、John Hammond 的 discord 频道

### 第二阶段：进阶阶段，拥抱 Hacktastic！

这个阶段就是拥抱试验、错误和胜利，因为在它们中埋藏着实践智慧的种子。我还是分为 9 个步骤，以供大家学习和参考。

**第 1 步：在 Hackerrank 或 leetcode 或任何编码平台上练习您之前学习的语言**

> **资源：**[Hackerrank](https://www.hackerrank.com/) 和 [leetcode](https://leetcode.com/) 实践

**第 2 步：开始学习 DSA 并每天在 Leetcode 上练习**

> **资源：**从 youtube 学习并在 [Hackerrank](https://www.hackerrank.com/) 和 [leetcode](https://leetcode.com/) 上练习

**第 3 步：了解如何使用 Burp-Suite 和 Nmap**

> **资源：**
> 
> 1. Burp 设置 — [https://youtu.be/wNqaLalaNE0（12](https://youtu.be/wNqaLalaNE0) 分 [23](https://www.youtube.com/watch?v=wNqaLalaNE0&t=743s) 秒）
> 
> 2. burp 基础 — [https://youtu.be/G3hpAeoZ4ek](https://youtu.be/G3hpAeoZ4ek)
> 
> 3. [https://youtu.be/Ezs19sj04DU](https://youtu.be/Ezs19sj04DU)
> 
> 4. Nmap 基础知识 1— [https://youtu.be/x4AE5yOF9pE](https://youtu.be/x4AE5yOF9pE)
> 
> 5. Nmap 基础知识 2 — [https://youtu.be/80vIin4xGp8](https://youtu.be/80vIin4xGp8)
> 
> 6. Nmap 基础知识 3— [https://youtu.be/4t4kBkMsDbQ](https://youtu.be/4t4kBkMsDbQ)
> 
> 7. [https://youtu. be/qsA8zREbt6g](https://youtu.be/qsA8zREbt6g)（奖金视频源）

**第 4 步：了解如何使用 Postman API**

> **资源：**
> 
> 1. [https://learning.postman.com/docs/introduction/overview/](https://learning.postman.com/docs/introduction/overview/)

**第 5 步：注册 Portswigger Academy 以学习和测试您的 Web 应用程序安全技能**

> **资源：**
> 
> **1.** [https://portswigger.net/web-security](https://portswigger.net/web-security)
> 
> 2. [https://www.youtube.com/@RanaKhalil101](https://www.youtube.com/@RanaKhalil101)
> 
> **书籍：**
> 
> 1. The Web Application Hacker's Handbook: Finding and Exploiting Security Flaws 2nd Edition by [Dafydd Stuttard](https://www.amazon.com/Dafydd-Stuttard/e/B001JS8ORI/ref=dp_byline_cont_book_1)（作者），[Marcus Pinto](https://www.amazon.com/s/ref=dp_byline_sr_book_2?ie=UTF8&field-author=Marcus+Pinto&text=Marcus+Pinto&sort=relevancerank&search-alias=books)（作者）

**第 6 步：注册 APIsec 大学以了解有关 API 安全的所有信息**

> **资源：**
> 
> 1. [https://www.apisecuniversity.com/](https://www.apisecuniversity.com/)
> 
> **书籍：**
> 
> 1. 黑客 API | Breaking Web Application Programming Interfaces by Corey Ball ( [https://nostarch.com/hacking-apis](https://nostarch.com/hacking-apis))

**第 7 步：深入了解 OWASP 十大漏洞**

> **资源：**
> 
> 1. [https://owasp.org/www-project-web-security-testing-guide/assets/archive/OWASP_Testing_Guide_v4.pdf](https://owasp.org/www-project-web-security-testing-guide/assets/archive/OWASP_Testing_Guide_v4.pdf)
> 
> 2. [https://owasp.org/www-project-top-ten/](https://owasp.org/www-project-top-ten/)

**第 8 步：阅读这 2 本书以了解现实生活场景**

> **资源：**
> 
> 1. [https://nostarch.com/bug-bounty-bootcamp](https://nostarch.com/bug-bounty-bootcamp)
> 
> 2. [https://nostarch.com/bughunting](https://nostarch.com/bughunting)

**第 9 步：选择一种方式：**

VAPT | Bug-Bounty | CTF | Pentesting

![](https://image.3001.net/images/20230607/1686150014_64809b7e468e63d3783b9.png!small)

> **资源:**
> 
> **1.** [https://medium.com/swlh/how-to-get-into-bug-bounties-383266799832](https://medium.com/swlh/how-to-get-into-bug-bounties-383266799832)
> 
> 2. [https://codingo.com/posts/2021-04-04-bug-classes-starting-out/](https://codingo.com/posts/2021-04-04-bug-classes-starting-out/)
> 
> 3. [https://codingo.com/posts/2021-07-18-bounties-for-a-living/](https://codingo.com/posts/2021-07-18-bounties-for-a-living/)
> 
> 4. [https://vickieli.dev/hacking/intro-ctf/](https://vickieli.dev/hacking/intro-ctf/)
> 
> 5. [https://www.youtube.com/watch?v=anfA2WSihHA](https://www.youtube.com/watch?v=anfA2WSihHA)
> 
> 6. [https://youtu.be/Zfz3ZN2dTDM](https://youtu.be/Zfz3ZN2dTDM)
> 
> 7. [https://owasp.org/www-pdf-archive/Getting_Started_with_Bug_Bounty..pdf](https://owasp.org/www-pdf-archive/Getting_Started_with_Bug_Bounty..pdf)

第 10 步：获得 ISC2 CC 认证（免费）或 CEH 理论 / 实践认证（付费）（可选）

### 第三阶段：专业水平，冲刺总决赛！

在这里，我们将提升到专业化的新高度。凭借我们坚实的基础和丰富的实践经验，是时候踏上专业之路，磨练我们在 Web 应用程序安全、网络安全或其他领域的技能。这种专业化需要进一步的培训和认证，例如 OSCP 或 OSCE，我们将获得更多专业知识。这个阶段我分成 9 步以供学习和参考。

**第 1 步：****报名参加 Pentesterlab 和 THM 以提高技能**

> **资源：**
> 
> 1. [https://www.pentesterlab.com/](https://www.pentesterlab.com/)
> 
> 2. [https://tryhackme.com/](https://tryhackme.com/)
> 
> 3. [https://academy.tcm-sec.com/courses](https://academy.tcm-sec.com/courses)
> 
> 4. [https://taggartinstitute.org/courses](https://taggartinstitute.org/courses)
> 
> 5. [https://www.youtube.com/watch?v=etP1hgJXijw](https://www.youtube.com/watch?v=etP1hgJXijw)

**第 2 步：****注册 Rootme 练习 CTF**

> **资源：**
> 
> 1. [https://www.root-me.org/fr/Challenges/](https://www.root-me.org/fr/Challenges/)

**第 3 步：报名 HTB 练习 CTF，准备 OSCP**

> **资源：**
> 
> 1. [https://www.hackthebox.com/](https://www.hackthebox.com/)

**第 4 步：获得 PNPT 认证**

> **资源：**
> 
> 1. [https://certifications.tcm-sec.com/](https://certifications.tcm-sec.com/)

**第 5 步：从 INE 了解有关网络和网络安全的所有信息**

> **资源：**
> 
> 1. [https://ine.com/](https://ine.com/)

**第 6 步（可选）：获得 eJPT 认证**

> **资源：**
> 
> 1. [https://ine.com/learning/certifications/internal/elearnsecurity-certified-professional-penetration-tester](https://ine.com/learning/certifications/internal/elearnsecurity-certified-professional-penetration-tester)

**第 7 步：获得 OSCP 认证**

> **资源：**
> 
> 1. [https://www.offsec.com/courses/pen-200/](https://www.offsec.com/courses/pen-200/)

**第 8 步：通过社交媒体平台和培训平台回馈网络安全社区。**

> **资源：**
> 
> 1. [https://twitter.com/hacker_content](https://twitter.com/hacker_content)
> 
> 2. Twitter
> 
> 3. YouTube
> 
> 4. 为其他人制作实验室以在 THM 等方面进行练习。

**第 9 步：根据需要完成其他认证，并保持对知识永不满足的渴望，因为网络安全领域在不断发展**

> **博客和文章：**
> 
> 黑客文章： https: [//www.hackingarticles.in/](https://www.hackingarticles.in/)
> 
> Vickie Li 博客： https: [//vickieli.dev/](https://vickieli.dev/)
> 
> Bugcrowd 博客：[https://www.bugcrowd.com/blog/](https://www.bugcrowd.com/blog/)
> 
> Intigriti 博客：[https://blog.intigriti.com/](https://blog.intigriti.com/)
> 
> Portswigger 博客：[https://portswigger.net/blog](https://portswigger.net/blog)
> 
> **Writeups：**
> 
> Infosec Writeups：[https://infosecwriteups.com/](https://infosecwriteups.com/)
> 
> Hackerone Hacktivity：[https://hackerone.com/hacktivity](https://hackerone.com/hacktivity)

总之，进入 Pentesting、VAPT 和 Bug Bounty 狩猎领域需要坚定的承诺、无限的耐心、毅力和对不断学习的永不满足的胃口。通过踏上这个令人愉快的 3 阶段冒险之旅，您将巩固自己的基础，陶醉在激动人心的实践体验中，并最终在广泛的安全专业知识中找到适合自己的位置。

其他资源分享：
-------

1、Kali Linux : [https://www.kali.org/](https://www.kali.org/)

2、Ubuntu: [https://ubuntu.com/](https://ubuntu.com/)

3、VMware: [https://www.vmware.com/in/products/workstation-player.html](https://www.vmware.com/in/products/workstation-player.html)

如何在 VMware 中设置 kali/ubuntu，请看 Youtube 视频。

4、我阅读的最佳博客的：

[初学者 Bug Bounty - 您应该从哪些 Bug 类别开始？](https://codingo.com/posts/2021-04-04-bug-classes-starting-out/)

[您是否应该以漏洞赏金为生？](https://codingo.com/posts/2021-07-18-bounties-for-a-living/)

[访谈](https://blog.pentesterlab.com/the-interview-9706357fd532)

其他 PentesterLab 博客：[https://medium.com/@PentesterLab](https://medium.com/@PentesterLab)

5、我尝试的 Udemy 课程：

完整的 2023 年网络开发训练营：[https://www.udemy.com/course/the-complete-web-development-bootcamp/](https://www.udemy.com/course/the-complete-web-development-bootcamp/)

nahamsec 的错误赏金介绍：[https://www.udemy.com/course/intro-to-bug-bounty-by-nahamsec/](https://www.udemy.com/course/intro-to-bug-bounty-by-nahamsec/)

6、（可选）我尝试的 Coursera 课程：

[https://www.coursera.org/google-certificates/cybersecurity-certificate](https://www.coursera.org/google-certificates/cybersecurity-certificate)

7、学习应用安全的最佳免费资源：

Portswigger academy [https://portswigger.net/web-security](https://portswigger.net/web-security)

APIsec 大学 [https://www.apisecuniversity.com/](https://www.apisecuniversity.com/)

Tryhackme [https://tryhackme.com/](https://tryhackme.com/)

PentesterLab [https://www.pentesterlab.com/](https://www.pentesterlab.com/)

Taggart Institute [https://taggartinstitute.org/](https://taggartinstitute.org/)

Rootme [https://www.root-me.org/](https://www.root-me.org/)

8、关注对我启发最大的最佳网络安全内容创作者名单：

[https://twitter.com/i/lists/1654858454060924930?s=20](https://twitter.com/i/lists/1654858454060924930?s=20)