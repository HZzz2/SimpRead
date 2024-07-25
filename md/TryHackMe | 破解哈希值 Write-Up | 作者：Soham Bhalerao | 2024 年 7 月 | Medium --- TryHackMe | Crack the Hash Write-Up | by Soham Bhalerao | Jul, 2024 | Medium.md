> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [medium.com](https://medium.com/@sohambhalerao33/tryhackme-crack-the-hash-write-up-bbb9fb5b76e6)

> The hashing function is a one way function that converts a given input into a hash value. normally pa......

[

![](https://miro.medium.com/v2/da:true/resize:fill:88:88/0*pTRxPUFYt9JT6sCt)

](https://medium.com/@sohambhalerao33?source=post_page-----bbb9fb5b76e6--------------------------------)

The hashing function is a one way function that converts a given input into a hash value. normally passwords are passed through a hash function and a hash value is obtained, which is then stored in the database as opposed to a plaintext password.  
哈希函数是一种单向函数，它将给定的输入转换为哈希值。通常，密码会通过哈希函数，生成哈希值，然后将该值存储在数据库中，而不是明文密码。

Since hashing is a one-way function, it is practically impossible to convert a hash value back to plaintext. Instead, hash values can be cracked by finding the original input used to generate the hash value. Tools such as HashCat, John the Ripper, and Crackstation are useful for cracking hashes.  
由于哈希是单向函数，几乎不可能将哈希值转换回明文。相反，可以通过找到用于生成哈希值的原始输入来破解哈希值。诸如 HashCat、John the Ripper 和 Crackstation 之类的工具对破解哈希值很有用。

**Required Tools: 所需工具：**
-------------------------

**_1.HashCat 1. HashCat_**

**_2.Hash-id 2. 哈希标识_**

**_3.Crackstation 3. 破解站_**

The Following write-up contains Solutions and commands used to obtain the solution for this room.  
以下的文章包含了用于解决这个房间问题的解决方案和命令。

Hash value- 48bb6e862e54f2a795ffc4e541caed4d  
哈希值 - 48bb6e862e54f2a795ffc4e541caed4d

Solution — easy 解决方案 - 简单

ID — md5 ID — md5 ID - md5

```
hashcat -m 0 "48bb6e862e54f2a795ffc4e541caed4d" /usr/share/wordlists/rockyou.txt
```

Hash value- CBFDAC6008F9CAB4083784CBD1874F76618D2A97  
哈希值 - CBFDAC6008F9CAB4083784CBD1874F76618D2A97

Solution- password123 解决方案 - password123

ID- sha1 ID- sha1 ID- sha1

```
hashcat -m 100 "CBFDAC6008F9CAB4083784CBD1874F76618D2A97" /usr/share/wordlists/rockyou.txt
```

Hash value- 1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032  
哈希值 - 1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032

Solution- letmein 解决方案 - letmein

ID- sha256 ID- sha256 ID- sha256（标识符 - sha256）

```
hashcat -m 1400 "1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032" /usr/share/wordlists/rockyou.txt
```

Hash identification- 哈希识别 -

```
hashid -m "\\$2y\\$12\\$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom"
```

Hash value- $2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom  
哈希值 - $2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom

Solution- bleh 解决方案 - 嘘

ID- Blowfish ID- Blowfish 身份 - 吹鱼

```
cat rockyou.txt | grep -E '^.{4}' > shortenedrockyou.txt
```

```
hashcat -m 3200 "\\$2y\\$12\\$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom" /root/shortenedrockyou.txt
```

```
hashcat -m 3200 "\\$2y\\$12\\$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom" /usr/share/wordlists/rockyou.txt
```

Hash value- 279412f945939ba78ce0758d3fd83daa  
哈希值 - 279412f945939ba78ce0758d3fd83daa

Solution- Eternity22 解决方案 - Eternity22

ID- md4

This task can be completed on Crackstation.  
这项任务可以在 Crackstation 上完成。

with the exception of 1.4, all the tasks from 1.1–1.5 can be completed using Crackstation.  
除了 1.4 之外，1.1-1.5 的所有任务都可以使用 Crackstation 完成。

![](https://miro.medium.com/v2/resize:fit:1094/1*x0OtFl3KBcs7j3qQut4IyA.png)

Hash value- F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85  
散列值 - F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85

Solution- paule 解决方案 - paule

```
hashid -m "F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85"
```

ID- sha256 ID- sha256 身份证 - sha256

```
hashcat -m 1400 "F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85" /usr/share/wordlists/rockyou.txt
```

**Task 2.2 任务 2.2**

Hash value- 1DFECA0C002AE40B8619ECF94819CC1B  
哈希值 - 1DFECA0C002AE40B8619ECF94819CC1B

Solution- n63umy8lkf4i 解决方案 - n63umy8lkf4i

ID- NTLM ID- NTLM 身份验证 - NTLM

```
hashcat -m 1000 "1DFECA0C002AE40B8619ECF94819CC1B" /usr/share/wordlists/rockyou.txt
```

**Task 2.3 任务 2.3**

Hash value- $6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.  
哈希值 - $6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.

Solution- waka99 解决方案 - waka99

ID- sha512

```
hashid -m "\\$6\\$aReallyHardSalt\\$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02."
```

```
hashcat -m 1800 "\\$6\\$aReallyHardSalt\\$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02." /usr/share/wordlists/rockyou.txt
```

**Task 2.4 任务 2.4**

Hash value- e5d8870e5bdd26602cab8dbe07a942c8669e56d6  
哈希值 - e5d8870e5bdd26602cab8dbe07a942c8669e56d6

Solution- 481616481616 解决方案 - 481616481616

ID- HMAC-sha1 ID- HMAC-sha1 身份验证 - HMAC-sha1

```
hashcat -m 160 e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme /usr/share/wordlists/rockyou.txt
```