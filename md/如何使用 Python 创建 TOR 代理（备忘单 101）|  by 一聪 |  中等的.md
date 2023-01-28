> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [ohyicong.medium.com](https://ohyicong.medium.com/how-to-create-tor-proxy-with-python-cheat-sheet-101-3d2d619a1d39)

> Hey folks! Stay truly anonymous by creating your own TOR proxy with python.

![](https://miro.medium.com/max/875/1*gF9MtfHlIi5vIjwvLTYZDw.png)

回到 1990 年代，互联网是一个不安全的地方，网络流量大部分是未加密和可追踪的，这使得黑客可以执行中间人攻击并轻松收集个人数据。为了解决这个问题，美国海军研究实验室的一组研究人员启动了洋葱路由 (TOR) 项目，旨在匿名化和保护互联网上的通信。

TOR 依靠分布式信任模型来保护您在互联网上的网络流量。这是一种方法，您的数据由多方加密以提供分层数据保护（如洋葱）。因此您的数据将是安全的，除非有人能够劫持参与加密的所有各方。

在实践中，TOR 选择 3 个由不同实体运营的独特中继来加密和路由您的网络流量，然后再到达最终目的地。

这是一个简单的图表来说明该方法。

![](https://miro.medium.com/max/784/0*MN3XfWF0i1j1Kbcx)洋葱路由原理（[https://medium.com/swlh/tor-how-does-it-work-d0be02ddb539](https://medium.com/swlh/tor-how-does-it-work-d0be02ddb539)）

要了解更多关于 TOR 的信息，[Velentin Quelquejay](https://medium.com/swlh/tor-how-does-it-work-d0be02ddb539) 有一篇很棒的文章。

对于本文，我将重点关注使用 Python 配置 TOR 代理的实际方面，因为我觉得在这方面缺乏在线资料和讨论。

为了帮助您更有效地学习，我将本教程分为 3 个难度级别，涵盖了 TOR 的有用功能，让您充分发挥其潜力。

在继续本教程之前，请确保您已安装以下内容。

1.  Python3
2.  pip 安装 stem
3.  Git 克隆 [https://github.com/ohyicong/Tor](https://github.com/ohyicong/Tor)

下面是使用默认配置创建 TOR 代理服务器的简单有效的代码：

1.  在本地端口 9050 上建立代理
2.  通过 3 个唯一中继建立 TOR 连接（随机）
3.  每 10 分钟更改一次 IP 地址

```
# create_basic_tor_proxy.py 中的完整代码
import io 
import os 
import stem.process 
import reSOCKS_PORT = 9050 
TOR_PATH = os.path.normpath(os.getcwd()+"\\tor\\tor.exe")tor_process = stem.process.launch_tor_with_config( 
  config = { 
    'SocksPort': str(SOCKS_PORT), 
  }, 
  init_msg_handler = lambda line: print(line) if re.search('Bootstrapped', line) 否则为假，
  tor_cmd = TOR_PATH 
)
```

要检查您的设置是否成功，您可以向 [http://ip-api.com/json/](http://ip-api.com/json/) 发出 GET 请求。如果 IP 地址来自不同的国家，那么您已经成功设置了您的第一个 TOR 代理！

```
导入请求从日期时间
导入 json
导入日期时间PROXIES = { 
    'http': 'socks5://127.0.0.1:9050', 
    'https': 'socks5://127.0.0.1:9050' 
} 
response = requests.get(" http://ip-api. com/json/ ", proxies=PROXIES) 
result = json.loads(response.content) 
print('TOR IP [%s]: %s %s'%(datetime.now().strftime("%d-% m-%Y %H:%M:%S"), 结果["查询"], 结果["国家"]))
```

![](https://miro.medium.com/max/875/1*9KHOQu5Jrf8MOFzuVWV3Ug.png)

请记住在开始下一个练习之前停止您的代理。这是停止它的命令。

```
tor_process.kill()
```

在某些情况下，您可能需要更多地控制 TOR 连接。以下是一些可能的原因：

1.  由于网络安全 / 法律 / 政治原因，避免来自某些国家 / 地区的中继。
2.  更改 TOR IP 地址刷新率。
3.  对 TOR 连接使用不同的身份验证方法

下面的代码使用以下配置在本地端口 9050 上创建一个 TOR 代理服务器：

1.  入口节点：TOR 应使用来自法国的中继作为其入口节点。
2.  ExitNodes：TOR 应使用来自日本的中继作为其出口节点。
3.  StrictNodes：TOR 连接应严格遵循用户配置。
4.  CookieAuthentication：使用 cookie 认证。
5.  MaxCircuitDirtiness：每 60 秒刷新一次 IP。
6.  GeoIPFile：使用指定的地理 ipv4 文件。可从 [https://raw.githubusercontent.com/torproject/tor/main/src/config/geoip 下载](https://raw.githubusercontent.com/torproject/tor/main/src/config/geoip)

```
# 在 create_intermediate_tor_proxy.py 中找到的完整代码
import io 
import os 
import stem.process 
import re 
import urllib.requestSOCKS_PORT = 9050 
TOR_PATH = os.path.normpath(os.getcwd()+"\\tor\\tor.exe") 
GEOIPFILE_PATH = os.path.normpath(os.getcwd()+"\\data\\tor\ \geoip") 
try: 
    urllib.request.urlretrieve(' https://raw.githubusercontent.com/torproject/tor/main/src/config/geoip' , GEOIPFILE_PATH) 
except: 
    print ('[INFO] 无法更新 geoip文件。使用本地副本。')tor_process = stem.process.launch_tor_with_config( 
  config = { 
    'SocksPort' : str(SOCKS_PORT), 
    'EntryNodes' : '{FR}', 
    'ExitNodes' : '{JP}', 
    'StrictNodes' : '1', 
    'CookieAuthentication '：'1'，
    'MaxCircuitDirtiness'：'60'，
    'GeoIPFile'：' https://raw.githubusercontent.com/torproject/tor/main/src/config/geoip'，  }，
  init_msg_handler = lambda 行：打印(line) if re.search('Bootstrapped', line) else False, 
  tor_cmd = TOR_PATH 
)
```

要检查您的设置是否成功，您可以向 [http://ip-api.com/json/](http://ip-api.com/json/) 发出 GET 请求。如果 IP 地址来自日本并且每 60 秒更改一次，则您已成功配置 TOR 连接。

```
导入请求从日期
时间导入时间
导入日期时间PROXIES = { 
    'http': 'socks5://127.0.0.1:9050', 
    'https': 'socks5://127.0.0.1:9050' 
}for i in range(10): 
    response = requests.get(" http://ip-api.com/json/ ", proxies=PROXIES) 
    result = json.loads(response.content) 
    print('TOR IP [% s]: %s %s'%(datetime.now().strftime("%d-%m-%Y %H:%M:%S"), result["查询"], 结果["国家" ]))
    时间.睡眠(60)
```

![](https://miro.medium.com/max/875/1*DuzWNI5ForvRE3bwXsjo7w.png)

在前面的练习中，TOR 负责选择用于创建其连接的中继。但是，在某些情况下，您可能希望选择一个特定的中继来增强您在 Internet 上的匿名性和安全性。

以下是选择优质继电器的一些注意事项：

1.  网络速度和可靠性
2.  供应商的信誉 / 声誉
3.  服务正常运行时间

要找到一个好的 TOR 中继，我们可以使用 [https://metrics.torproject.org/rs.html](https://metrics.torproject.org/rs.html)

1.  搜索公共中继。我个人喜欢使用由 “Hetzner” 托管的中继，因为他们是具有可靠背景的数据中心提供商。

![](https://miro.medium.com/max/875/1*tBjHh9iel_Ukqpsrmh2Ozg.png)

2. 复制继电器的指纹

![](https://miro.medium.com/max/875/1*MBLsBaBOgK4u0k1CQf8bLg.png)

3. 此代码将允许您使用选定的中继作为入口节点（第一个中继）。

```
# 在 create_advanced_tor_proxy.py 中找到的完整代码
import io 
import os 
import stem.process 
from stem.control import Controller 
from stem import 信号
导入重新
导入请求
从日期时间导入json 
导入日期 
时间导入时间SOCKS_PORT = 9050  
CONTROL_PORT = 9051  
TOR_PATH = os.path.normpath(os.getcwd()+ "\\tor\\tor.exe" )
tor_process = stem.process.launch_tor_with_config(
  配置= {
    'SocksPort'：str（SOCKS_PORT）， 
     'ControlPort'：str（CONTROL_PORT）， 
     'CookieAuthentication'：'1'， 
     'EntryNodes'：'9695DFC35FFEB861329B9F1AB04C46397020CE31'， 
     'StrictNodes'：'1' 
  }, 
  init_msg_handler = lambda line: print(line) if re.search( '自举' , line) else  False ,
  tor_cmd = TOR_PATH
)
```

4. 要检查您的设置是否成功，您可以列出您的 TOR 电路。确认您的第一个节点具有与您的配置相同的指纹。

```
从 stem 导入 CircStatus
从 stem.control 导入控制器以 Controller.from_port(port = 9051) 作为控制器：
    controller.authenticate() 认证
    对于circ in sorted(controller.get_circuits()): 
         if circ.status == CircStatus.BUILT: 
             print ( "Circuit %s (%s)" % (circ.id, circ.purpose)) 
             for i, entry in enumerate （循环路径）：
                div = '+' if (i == len(circ.path) - 1) 其他 '|' 
                指纹，昵称 = 条目
                desc = controller.get_network_status（指纹，无）
                address = desc.address if desc else  'unknown'  
                print ( " %s- %s (%s, %s)" % (div, fingerprint, nickname,地址））
```

如果您看到一些电路，请不要担心，TOR 会自动创建额外的电路作为备份。

![](https://miro.medium.com/max/875/1*MP430xFASAwjgDCmt45xXg.png)

我希望您和我一样喜欢这个实用教程。研究 TOR 和词干库很有趣。实际上，我们可以用 TOR 做更多有趣的事情，我将在接下来的几周内写下来！敬请关注 ：）