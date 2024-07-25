> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [medium.com](https://medium.com/@redfanatic7/satellite-hacking-part-1-starting-with-the-basics-264494a23d2d)

> In this series of guides, we will explore satellite hacking from various approaches. Satellites have ......

[

![](https://miro.medium.com/v2/resize:fill:88:88/1*IrNJAPXobeCEabHG9eFHlQ.jpeg)

](https://medium.com/@redfanatic7?source=post_page-----264494a23d2d--------------------------------)

In this series of guides, we will explore satellite hacking from various approaches. Satellites have now become an essential technology in our daily lives. They provide us with internet access, television and radio signals, location services, satellite telephony services and, of course, satellite imagery necessary for domestic and military uses.  
在这一系列指南中，我们将从各种角度探讨卫星黑客攻击。卫星现在已成为我们日常生活中的重要技术。它们为我们提供互联网接入、电视和广播信号、定位服务、卫星电话服务，当然还有国内和军事用途所必需的卫星图像。

If these essential technological devices are vulnerable to hacking, a large part of our daily communication could be intercepted or affected. In addition, these satellites play a key role in military capabilities and strategy, such as the recent GPS jamming and tampering by Russia in the Ukraine/Russia war.  
如果这些重要的技术设备容易受到黑客攻击，我们日常通信的很大一部分可能会被拦截或受到影响。此外，这些卫星在军事能力和战略中扮演关键角色，比如最近俄罗斯在乌克兰 / 俄罗斯战争中进行的 GPS 干扰和篡改。

![](https://miro.medium.com/v2/resize:fit:1094/0*rsW99We9UH-vWZWB.png)

Like many components of the Internet of Things (IoT), much of their security is based on security through opacity. We intend to change that!  
就像物联网 (IoT) 的许多组成部分一样，它们的大部分安全性都是基于不透明的安全性。我们打算改变这一点！

Before we begin this series, we need to discuss some essentials and hardware requirements if you are going to pursue this cutting-edge field of cybersecurity.  
在我们开始这个系列之前，如果你打算从事这个前沿领域的网络安全，我们需要讨论一些基本要点和硬件要求。

![](https://miro.medium.com/v2/resize:fit:1094/0*UnbzdR82YALE19gz.png)

Table of Contents 目录

*   [Satellite communication frequencies  
    卫星通信频率](https://en.iguru.gr/satellite-hacking-part-1-ksekinontas-vasika/#%CE%A3%CF%85%CF%87%CE%BD%CF%8C%CF%84%CE%B7%CF%84%CE%B5%CF%82_%CE%B4%CE%BF%CF%81%CF%85%CF%86%CE%BF%CF%81%CE%B9%CE%BA%CE%AE%CF%82_%CE%B5%CF%80%CE%B9%CE%BA%CE%BF%CE%B9%CE%BD%CF%89%CE%BD%CE%AF%CE%B1%CF%82)
*   [1. L-Band Antenna 1. L 波段天线](https://en.iguru.gr/satellite-hacking-part-1-ksekinontas-vasika/#1_L-Band_Antenna)
*   [2. SDR receiver 2. SDR 接收器](https://en.iguru.gr/satellite-hacking-part-1-ksekinontas-vasika/#2_SDR_receiver)
*   [3. Dragon OS 3. 龙 OS](https://en.iguru.gr/satellite-hacking-part-1-ksekinontas-vasika/#3_Dragon_OS)

Satellites communicate at frequencies from 1 to 30 GHz. These frequency ranges or bands are identified by letters. These bands are characterized from lowest frequency to highest, L, S, C, X Ku, K, Ka and V-bands.  
卫星在从 1 到 30 GHz 的频率范围内进行通信。这些频率范围或波段由字母进行标识。这些波段按从低到高频率依次为 L、S、C、X、Ku、K、Ka 和 V 波段。

![](https://miro.medium.com/v2/resize:fit:781/0*C2p7zmsm8MEIf_vz.png)

L-band satellite communications are between 1,525 GHz and 1,66 GHz. L-Band includes satellite communication systems such as GPS, Immarsat and Iridium.  
L 波段卫星通信频率为 1,525 千兆赫至 1,66 千兆赫。L 波段包括 GPS、Immarsat 和 Iridium 等卫星通信系统。

For optimal performance, you will need an antenna optimized for L-band communication between 1,525Ghz and 1,660Ghz. I bought a cheap L-Band planar antenna for $60 and it performs more than adequately. These are available through Amazon, rtl-sdr.com and many electronics stores.  
为了获得最佳性能，您将需要一款优化的天线，用于 1,525 GHz 至 1,660 GHz 之间的 L 波段通信。我花了 60 美元买了一款便宜的 L 波段平面天线，性能超出了预期。这些可以通过亚马逊、rtl-sdr.com 和许多电子商店购买。

![](https://miro.medium.com/v2/resize:fit:547/0*VVMkO39BGfiDnNxm.png)

There are many SDR receivers on the market. Probably the most popular is the RTL-SDR because of its low cost (around $35). Since the RTL-SDR can receive frequencies from 500khz to 1,75Ghz, it will work fine for intercepting L-Band satellite communications, but will be inadequate for higher frequency communications.  
市面上有许多 SDR 接收器。也许最受欢迎的是 RTL-SDR，因为它价格低廉（约 35 美元）。由于 RTL-SDR 可以接收从 500kHz 到 1.75GHz 的频率，它可以很好地拦截 L 波段卫星通信，但对于更高频率的通信则不足够。

Both the HackRF One and the LimeSDR Mini are capable of transmitting and receiving signals in the 1Mhz to 6GHz range. If you can afford these higher priced devices ($300-$600), I recommend getting one of these.  
HackRF One 和 LimeSDR Mini 都能在 1MHz 至 6GHz 范围内发送和接收信号。如果你能负担得起这些价格较高的设备（300 美元至 600 美元），我建议购买其中之一。

![](https://miro.medium.com/v2/resize:fit:547/0*Om5uv_oLkzzK_5CO.png)

I have used various operating systems for SDR and satellite hacking and I have to say DragonOS is the best! It has a large number of tools already installed and configured, saving you hours of work. These tools include some of the satellite hacking tools we’ll use here (we’ll also install others).  
我在 SDR 和卫星黑客方面使用过各种操作系统，不过我必须说 DragonOS 是最好的！它预装并配置了大量工具，能够节省你大量工作时间。这些工具包括我们将在此处使用的一些卫星黑客工具（我们还将安装其他工具）。

In this series and subsequent lessons, we will exclusively use the [DragonOS](https://en.iguru.gr/dragonos-linux-dianomi-gia-radio-hacking/).  
在这个系列和后续课程中，我们将专门使用 DragonOS。

![](https://miro.medium.com/v2/resize:fit:781/0*IhzYSC9sczYgtzDF.png)

Once you have each of these items, you are ready to begin our Satellite Hacking adventure!  
一旦你拥有了这些物品，你就准备好开始我们的卫星黑客之旅了！