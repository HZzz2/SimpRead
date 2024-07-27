> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [medium.com](https://medium.com/mii-cybersec/efficiently-parsing-nessus-data-e7929bb44639)

> Hello everyone, I want to share how I process a lot of data from various segments in my vulnerability......

[

![](https://miro.medium.com/v2/resize:fill:88:88/2*1CusqdlOuU16NrTOMnqNHw.jpeg)

](https://medium.com/@riodrwn?source=post_page-----e7929bb44639--------------------------------)[

![](https://miro.medium.com/v2/resize:fill:48:48/1*fJpPVHpsy-fmvtjWzLAP_g.png)

](https://medium.com/mii-cybersec?source=post_page-----e7929bb44639--------------------------------)![](https://miro.medium.com/v2/resize:fit:1094/1*p8F6uJ5ymy-HafjbPrq7QQ.jpeg)

Hello everyone, I want to share how I process a lot of data from various segments in my vulnerability assessment project using Nessus. In the last few months I have been involved in a vulnerability assessment project, in this client I was involved in a large network network and had many segments. Everything went smoothly during the scanning process, but at the end of this project my client asked me to make a summary report in a short time.My problem is how going to go through one by one Nessus file, extract it out? load a large number of vulnerability results and post them in a report? too much data was used, and I ran out of time to make this report.  
大家好，我想分享一下我如何使用 Nessus 处理我漏洞评估项目中的各种细分数据。在过去的几个月中，我参与了一项漏洞评估项目，在这个客户那里，我参与了一个大型网络，并且有许多细分。在扫描过程中一切顺利，但在该项目结束时，我的客户要求我在短时间内制作一个摘要报告。我的问题是如何逐一浏览 Nessus 文件，提取出它们？载入大量漏洞结果并将它们发布在报告中？使用的数据太多了，而我没有时间制作这份报告。

Parsing Nessus Data 解析 Nessus 数据
--------------------------------

Day after day, I try surf the internet trying to find something that would make my job easier. i found this [blog](https://www.melcara.com/archives/253) from that blog referred me to a script on github [here](https://github.com/interference-security/nessus_parser). This tool is so sick!! I really didn’t expect that this tool could make my work easier. all I have to do is put my nessus output (.nessus / .XML) into one folder and the parser will run and combine all the report of those seperate Nessus output. BAM!! within seconds you would get your output and the result is very beautiful  
日复一日，我尝试在互联网上搜索能够让我的工作更轻松的东西。我在一个博客上发现了另一个博客，其中提到一个在 GitHub 上的脚本。这个工具真是太棒了！我真的没想到这个工具能让我的工作变得更轻松。我只需要把我的 nessus 输出（.nessus / .XML）放入一个文件夹，解析器就会运行并汇总所有这些单独的 Nessus 输出的报告。瞧！在几秒钟内，您将获得您的输出，结果非常美观。

**How To Use 如何使用**
-------------------

1.  The first step is export your nessus scanning results.  
    第一步是导出您的 nessus 扫描结果。

![](https://miro.medium.com/v2/resize:fit:1094/1*5S6cuZvvp9uI7gQup3BAhg.jpeg)![](https://miro.medium.com/v2/resize:fit:1094/1*h063HA-5AcEPEFqD5Yl8Jg.png)

2. put the script and nessus output in same folder  
2. 将脚本和 nessus 输出放在同一个文件夹中

![](https://miro.medium.com/v2/resize:fit:1056/1*4MZwnFSYqe5aQvUJI6n7hg.png)

3. wait until the process successfully you will get file .xlsx  
3. 等待过程成功后，您将获得文件. xlsx。

![](https://miro.medium.com/v2/resize:fit:864/1*KjnOBgAVwIhhIliPkJfF9Q.png)

4. open with excel, and you will see the results parser  
4. 用 Excel 打开，您将看到结果解析器

![](https://miro.medium.com/v2/resize:fit:1094/1*DVR2vid_M-Ba1YdGDMYKPw.jpeg)host scan data 主机扫描数据![](https://miro.medium.com/v2/resize:fit:1094/1*I5sqM1TToBjAgHPS-p4SiA.jpeg) scan information 扫描信息![](https://miro.medium.com/v2/resize:fit:1094/1*Ao37RKoZJL45X_LEIs8o9A.jpeg) summary vulnerability 摘要漏洞![](https://miro.medium.com/v2/resize:fit:1094/1*9SEy3FpZxXHkGvXzNiR-RA.jpeg) Vulnerability Results 漏洞结果![](https://miro.medium.com/v2/resize:fit:1094/1*bhHJcJY3e1E6th8BYdVakA.jpeg) Summary Report Data 总结报告数据

I hope this article can help others who are experiencing a similar situation as mine.  
我希望这篇文章能帮助那些正经历类似情况的人。