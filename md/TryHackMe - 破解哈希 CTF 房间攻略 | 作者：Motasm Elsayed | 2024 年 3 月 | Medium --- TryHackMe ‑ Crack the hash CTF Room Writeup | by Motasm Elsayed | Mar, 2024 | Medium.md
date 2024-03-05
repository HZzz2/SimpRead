> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [medium.com](https://medium.com/@x0Mo3tAsmx0/tryhackme-crack-the-hash-ctf-room-writeup-0d944afd7d76)

> Challenge description: This challenge tests your knowledge of identifying and analyzing password hash......

[

![](https://miro.medium.com/v2/resize:fill:88:88/1*EwHiKLyxrOevMUUBZmWcfQ.png)

](https://medium.com/@x0Mo3tAsmx0?source=post_page-----0d944afd7d76--------------------------------)![](https://miro.medium.com/v2/resize:fit:963/0*35ognl7JI0Rgdtuc.jpg)

**Challenge description:** This challenge tests your knowledge of identifying and analyzing password hashes, cracking password hashes using online tools, and conducting password-dictionary attacks using tools such as Hashcat and John The Ripper.  
挑战描述：这个挑战测试你对于识别和分析密码哈希、使用在线工具破解密码哈希以及使用 Hashcat 和 John The Ripper 等工具进行密码字典攻击的知识。

**Challenge category:** Cryptography — Password Cracking — Password Dictionary Attacks  
挑战类别：密码学 - 密码破解 - 密码字典攻击

**Challenge link:** [Crack the hash](https://tryhackme.com/room/crackthehash)  
挑战链接：破解哈希

Mastering the craft of conducting password-cracking attacks with diverse tools and techniques is a paramount skill for every penetration tester. Beyond mere unauthorized access, this proficiency is a linchpin in understanding and fortifying digital security. It serves as a crucial dimension of a penetration tester’s toolkit, allowing them to uncover vulnerabilities within systems, assess the robustness of cryptographic defenses, and ultimately enhance the overall resilience of an organization’s cybersecurity posture. In the dynamic landscape of cyber threats, the ability to proficiently navigate and decrypt passwords is not just a tactical advantage but a strategic imperative, reinforcing the pentester’s capability to identify and mitigate potential points of exploitation. So let’s delve into these challenge questions together!  
掌握使用各种工具和技术进行密码破解攻击的技能对于每个渗透测试人员来说都是至关重要的。除了简单的未经授权访问之外，这种熟练程度是理解和加固数字安全的关键。它作为渗透测试人员工具包的一个重要维度，使他们能够发现系统中的漏洞，评估加密防御的强度，并最终增强组织网络安全姿态的整体弹性。在动态的网络威胁环境中，熟练地导航和解密密码不仅是一种战术优势，更是一种战略必要，加强了渗透测试人员识别和减轻潜在利用点的能力。让我们一起深入探讨这些挑战性问题吧！

When it comes to cracking password hashes in the hope of retrieving the password value of the hash, it’s a good practice to start by making use of **online hash cracking tools** as these tools have massive pre-computed lookup tables to crack password hashes and it’s more likely to find the hash value, if it’s there, in less than a second, on the other hand, using tools like **Hashcat** or **John The Ripper** to crack the password hash using brute-force attacks or even password-dictionary attacks may take longer.  
当涉及到破解密码哈希以期望获取哈希的密码值时，一个好的做法是首先利用在线哈希破解工具，因为这些工具拥有庞大的预计算查找表来破解密码哈希，如果哈希值存在的话，很可能在不到一秒钟的时间内找到哈希值；另一方面，使用 Hashcat 或 John The Ripper 等工具进行暴力破解或密码字典攻击来破解密码哈希可能需要更长的时间。

As we said above, it’s a good practice to first look for the hash values on online hash-cracking tools or databases, let’s try to crack the hash values in Task 1.  
正如我们上面所说的，首先在在线的哈希破解工具或数据库中查找哈希值是一个好的做法，让我们尝试破解任务 1 中的哈希值。

There are a lot of online hash-cracking tools out there we can use, but to name a few, the following websites are some of the most popular ones:  
有很多在线哈希破解工具可供使用，但以下是一些最受欢迎的网站：

*   [https://crackstation.net/](https://crackstation.net/)
*   [https://hashes.com/en/decrypt/hash](https://hashes.com/en/decrypt/hash)
*   [https://md5hashing.net/](https://md5hashing.net/)
*   [https://www.onlinehashcrack.com/](https://www.onlinehashcrack.com/)

Throughout this writeup, we are gonna use [https://crackstation.net/](https://crackstation.net/) and [https://hashes.com/en/decrypt/hash](https://hashes.com/en/decrypt/hash).  
在整个写作过程中，我们将使用 https://crackstation.net / 和 https://hashes.com/en/decrypt/hash。

Well, let’s open them and then copy and paste all the hash values in task 1, but make sure to write one hash per line.  
好的，让我们打开它们，然后将所有哈希值复制并粘贴到任务 1 中，但请确保每行写一个哈希值。

![](https://miro.medium.com/v2/resize:fit:963/0*XL0Ed5G33KBed5wA.png)![](https://miro.medium.com/v2/resize:fit:963/0*TnJiFpRkEGTaNYe_.png)

Well done! As you see, we retrieved all the passwords by using online hash-cracking tools! You may also have noticed that CrackStation could not retrieve the password of the hash number 4, but we were able to retrieve it using hashes.com. So keep in mind to try different tools.  
做得好！正如你所看到的，我们使用在线哈希破解工具成功恢复了所有密码！你可能也注意到，CrackStation 无法恢复哈希编号 4 的密码，但我们能够使用 hashes.com 恢复它。所以记住要尝试不同的工具。

In task 2 we will not use online hash cracking tools just to get familiar with the manual process of cracking hashes and start using tools such as **Hash-Identifier** to identify hash type and **Hashcat** to conduct password-dictionary attacks. So let’s get started!  
在任务 2 中，我们不会使用在线哈希破解工具，只是为了熟悉手动破解哈希的过程，并开始使用诸如 Hash-Identifier 来识别哈希类型和 Hashcat 来进行密码字典攻击。所以让我们开始吧！

**The following are the steps we will follow to find the password from its hash value using Hashcat:  
以下是我们将使用 Hashcat 找到密码的步骤：**

1.  Identify the hash type using tools such as Hash-Identifier, Hashid, or any online hash identifier tool  
    使用 Hash-Identifier、Hashid 或任何在线哈希标识工具来识别哈希类型
2.  Find the corresponding Hashcat “Hash-Mode” value from the following website “[https://hashcat.net/wiki/doku.php?id=example_hashes](https://hashcat.net/wiki/doku.php?id=example_hashes)" to use it when running **Hashcat**  
    从以下网站 “https://hashcat.net/wiki/doku.php?id=example_hashes” 中找到对应的 Hashcat“Hash-Mode”值，以便在运行 Hashcat 时使用。
3.  Save the hash value in a text file
4.  Run **Hashcat** to conduct a password-dictionary attack  
    运行 Hashcat 进行密码字典攻击

So keep these steps in mind as we will keep using them till the end of this writeup!  
I'm sorry, but I couldn't find the specific details on that. You can email support@

Level 2 — Hash 1  
等级 2 — 哈希 1
------------------------------

1.  Identifying the hash using the **Hash-Identifier** tool  
    使用 Hash-Identifier 工具识别哈希值

![](https://miro.medium.com/v2/resize:fit:851/0*J79ztD3xjCOekB68.png)

2. Finding the “Hash-Mode” value  
找到 “哈希模式” 值

From the following website “[https://hashcat.net/wiki/doku.php?id=example_hashes](https://hashcat.net/wiki/doku.php?id=example_hashes)", the value is `1400`  
从以下网站 “https://hashcat.net/wiki/doku.php?id=example_hashes”，该数值为 `1400`

3. Saving the hash value in a text file using the following command:  
使用以下命令将哈希值保存在文本文件中：

```
$ echo -n 'F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85' > hash.txt
```

4. Running **Hashcat** using the following command:  
使用以下命令运行 Hashcat：

```
$ hashcat -m 1400 -d 1 hash.txt /usr/share/wordlists/rockyou.txt
```

![](https://miro.medium.com/v2/resize:fit:963/0*hAjst9ymORCY0tKX.png)

Level 2 — Hash 2  
等级 2 — 哈希 2
------------------------------

1.  Identifying the hash using [https://hashes.com/en/tools/hash_identifier](https://hashes.com/en/tools/hash_identifier)  
    使用 https://hashes.com/en/tools/hash_identifier 识别哈希值。

![](https://miro.medium.com/v2/resize:fit:963/0*zc3-0Z_xu8p97uev.png)

2. Finding the “Hash-Mode” value  
找到 “哈希模式” 值

From the following website “[https://hashcat.net/wiki/doku.php?id=example_hashes](https://hashcat.net/wiki/doku.php?id=example_hashes)", the value is `1000`  
从以下网站 “https://hashcat.net/wiki/doku.php?id=example_hashes”，该值为 `1000`

3. Saving the hash value in a text file using the following command:  
将哈希值保存在文本文件中，使用以下命令：

```
$ echo -n '1DFECA0C002AE40B8619ECF94819CC1B' > hash.txt
```

4. Running **Hashcat** using the following command:  
使用以下命令运行 Hashcat：

```
$ hashcat -m 1000 -d 1 hash.txt /usr/share/wordlists/rockyou.txt
```

![](https://miro.medium.com/v2/resize:fit:931/0*1JVthcARS-iuxrIM.png)

Level 2 — Hash 3  
等级 2 — 哈希 3
------------------------------

1.  Identifying the hash using [https://hashes.com/en/tools/hash_identifier](https://hashes.com/en/tools/hash_identifier)  
    使用 https://hashes.com/en/tools/hash_identifier 识别哈希值。

![](https://miro.medium.com/v2/resize:fit:963/0*xIWgDP1iMwDoFseT.png)

2. Finding the “Hash-Mode” value  
找到 “哈希模式” 值

From the following website “ [https://hashcat.net/wiki/doku.php?id=example_hashes](https://hashcat.net/wiki/doku.php?id=example_hashes)", the value is `1800`  
从以下网站 “https://hashcat.net/wiki/doku.php?id=example_hashes”，该数值为 `1800`

3. Saving the hash value in a text file using the following command:  
使用以下命令将哈希值保存在文本文件中：

```
$ echo -n '$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.' > hash.txt
```

4. Running **Hashcat** using the following command:  
使用以下命令运行 Hashcat：

```
$ hashcat -m 1800 -d 1 hash.txt /usr/share/wordlists/rockyou.txt
```

![](https://miro.medium.com/v2/resize:fit:963/0*N6LESd9pWYd47yw6.png)

Level 2 — Hash 4  
等级 2 — 哈希 4
------------------------------

1.  Identifying the hash using the **Hash-Identifier** tool  
    使用 Hash-Identifier 工具识别哈希值

![](https://miro.medium.com/v2/resize:fit:839/0*cnwGb_Lu1iYD40PW.png)

2. Finding the “Hash-Mode” value  
找到 “哈希模式” 值

After trying different “Hash-Mode” values but in fail, we used the provided hint and after finding that the right hash type is _“HMAC-SHA1”_ then from the following website “ [https://hashcat.net/wiki/doku.php?id=example_hashes](https://hashcat.net/wiki/doku.php?id=example_hashes)", the value is `160`  
尝试了不同的 “哈希模式” 值，但失败了，我们使用了提供的提示，并在找到正确的哈希类型为 “HMAC-SHA1” 后，从以下网站 “https://hashcat.net/wiki/doku.php?id=example_hashes” 中找到了值 `160` 。

3. Saving the hash value with its salt _“tryhackme”_ in a text file using the following command:  
使用以下命令将哈希值与其盐值 “tryhackme” 保存在文本文件中：

```
$ echo -n 'e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme' > hash.txt
```

4. Running **Hashcat** using the following command:  
使用以下命令运行 Hashcat：

```
$ hashcat -m 160 -d 1 hash.txt /usr/share/wordlists/rockyou.txt
```

![](https://miro.medium.com/v2/resize:fit:931/0*sdBC0xlpB9CKD0ri.png)

**Note:** the `-d 1` option in the hashcat command is to run **Hashcat** using our GPU as it's always faster than using the CPU, so if you have a powerful dedicated GPU use it instead of your CPU.  
注意：hashcat 命令中的 `-d 1` 选项是使用我们的 GPU 运行 Hashcat，因为使用 GPU 比使用 CPU 更快，所以如果你有一块强大的独立 GPU，请使用它而不是 CPU。

**Additional note:** If you face any problem using your GPU with **Hashcat**, check out the following video from Nvidia as it will help you a lot in solving this problem. [https://www.youtube.com/watch?v=JaHVsZa2jTc&ab_channel=NVIDIADeveloper](https://www.youtube.com/watch?v=JaHVsZa2jTc&ab_channel=NVIDIADeveloper)  
附注：如果您在使用 Hashcat 时遇到任何与 GPU 相关的问题，请查看 Nvidia 的以下视频，它将对您解决这个问题非常有帮助。https://www.youtube.com/watch?v=JaHVsZa2jTc&ab_channel=NVIDIADeveloper

In conclusion, I hope this walkthrough has been informative and shed light on our thought processes, strategies, and the techniques used to tackle each task. CTFs are not just about competition; they’re about learning, challenging yourself and your knowledge, and getting hands-on experience through applying your theoretical knowledge.  
总之，希望这篇指南能够提供信息，揭示我们的思维过程、策略和解决每个任务所使用的技巧。CTF 不仅仅是竞争，它们更多是关于学习，挑战自己和自己的知识，通过应用理论知识获得实践经验。