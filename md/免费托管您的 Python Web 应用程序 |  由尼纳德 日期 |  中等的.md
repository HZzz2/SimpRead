> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [medium.com](https://medium.com/@nindate2020/host-your-python-web-app-for-free-d0613074196a)

> Consider you have written a great Python web application, and now you want to make it available to us......

假设你已经编写了一个很棒的 Python Web 应用程序，现在你想让它对用户可用。您有多种选择，免费的和付费的。吸引我的一个是 [Heroku](https://www.heroku.com/)，并想分享如何将它用于您的 Web 应用程序。

Heroku 是一个云平台，可让公司构建、交付、监控和扩展应用程序，而无需管理底层基础设施。Heroku 提供平台即服务 (PaaS)。

我喜欢 Heroku 的一些原因是：

*   对于个人项目和概念证明，Heroku 提供 0 美元选项
*   您可以每月免费运行 550–1,000 达诺[小时](https://www.heroku.com/dynos)（500 小时无需信用卡验证，1000 小时需要验证。）
*   如果您的应用程序长时间处于空闲状态，则测功机进入睡眠模式并且测功机小时数不会被消耗
*   每当您的应用程序被访问时，如果测功机处于睡眠状态，它将自动唤醒并开始为应用程序服务（唤醒过程中会有一个小延迟。测功机小时计量将恢复）
*   您可以使用任何语言部署 Web 应用程序——Python、Ruby、PHP、Node.js、Java、Go、Closure、Scala

现在让我们看看如何将 Python Web 应用程序部署到 Heroku。

在 Heroku 中创建一个帐户
----------------

如果您还没有 Heroku 帐户，请创建一个。

转到 [https://www.heroku.com/](https://www.heroku.com/) 并注册

![](https://miro.medium.com/max/1370/1*6OJS5VOZrsBJZUBYw-XHNw.png)

您还可以在手机上设置 Google 身份验证器应用程序以获取令牌 (OTP)，并在登录 Heroku 时将其用于多因素身份验证。

登录后，您可以使用任何语言部署 Web 应用程序——Python、Ruby、PHP、Node.js、Java、Go、Closure、Scala。

在您的机器上安装 Heroku 命令行界面 (CLI)
---------------------------

要将应用程序部署到 Heroku，您可以使用 Heroku 仪表板，也可以使用 Heroku 提供的命令行界面 (CLI)。这真的取决于什么适合您，但是我将向您展示如何使用 CLI 方法，因为它花费的时间更少。

要安装 Heroku CLI，您可以下载安装程序文件并安装，也可以下载 tarball 文件（这是一个存档文件）并解压缩文件。如果选择使用 tarball 方式，则需要手动将目录 heroku/bin 添加到环境变量 PATH 中。

对于 Linux，您可以使用以下命令将目录添加到 PATH 变量

> 导出 PATH=$PATH:/download-dir/heroku/bin

要将 heroku 永久添加到 PATH，您可以将上述行添加到主目录中的 .bashrc（对于 bash shell）或 .profile（对于 sh shell）文件。

对于 Windows，请参阅 [https://helpdeskgeek.com/windows-10/add-windows-path-environment-variable/](https://helpdeskgeek.com/windows-10/add-windows-path-environment-variable/) 以了解如何添加 / 更新环境变量。

您可以将 Web 应用程序作为 docker 容器部署到 Heroku，也可以不使用 docker 进行部署。下面我解释了这两个选项。

A. 在没有 docker 容器的情况下部署应用程序
--------------------------

要将您的 Web 应用程序部署到 Heroku，您需要将所有代码放在一个地方并创建一些配置文件。

**一个。准备代码库  
**为您的部署创建一个目录并复制 Web 应用程序的代码。

接下来的步骤假设您位于此部署目录中。

**湾。创建配置文件**

1.  _指定 python 版本_：文件名 **runtime.txt**

这是一个可选步骤，仅当您希望在 Heroku 中使用特定的 Python 版本来运行您的 Web 应用程序时才需要。在名为 runtime.txt 的文件中指定 python 版本（确保文件中的所有内容都是小写并且您提到完整版本）。

![](https://miro.medium.com/max/1400/0*vk9vijSNMC7z7TaY.png)

> 您可以使用 command 在您测试过代码的机器上检查 Python 版本`python -V`。

![](https://miro.medium.com/max/1400/0*IbXBAm1fRDAZVrBM.png)

_有关详细信息，__请参阅_ [_https://devcenter.heroku.com/articles/python-runtimes_](https://devcenter.heroku.com/articles/python-runtimes)

2. _指定 Python 依赖 / 包_：文件名 **requirements.txt**

创建一个 requirements.txt 文件并指定执行 Web 应用程序所需的 python 包。

![](https://miro.medium.com/max/1400/0*ct0YcuSPKvgPYa8A.png)

> 您可以使用该命令使用`pip freeze > requirement.txt`当前活动 python 环境中安装的所有包填充 requirements.txt。或者您可以手动将包名称添加到文件中。

3. _指定应用程序启动时要执行的命令_：文件名 **Procfile**

当您将 Web 应用程序部署到 Heroku 时，您需要指定应该运行的命令来启动 / 运行您的应用程序。您可以在名为 Procfile 的文件中指定该命令（不带任何扩展名）。

该命令的语法是  
`<process type>`：`<command>`

> `<process type>`是您的命令的字母数字名称，例如`web`, `worker`, `urgentworker`, `clock`, 等等。
> 
> `<command>`表示每个进程类型的 dyno 在启动时应该执行的命令，例如`rake jobs:work`.

[在此处](https://devcenter.heroku.com/articles/procfile)查找有关 Procfile 的更多信息[](https://devcenter.heroku.com/articles/procfile)

要部署 python 烧瓶应用程序，您将使用以下语法。

```
web: gunicorn <不带 .py 的 python 脚本的名称>:<Flask 的实例名称>

```

例如`web: gunicorn w5-hw-svc-predict:app`

*   web：表示 web 服务器进程可以接收来自 Heroku 路由器的外部 HTTP 流量。
*   gunicorn 是将要使用的 Web 服务器（软件）
*   Flask 应用程序的 Python 脚本是 w5-hw-svc-predict.py
*   当您定义类似的东西时，app 是 Fl​​ask 的实例名称`app = Flask('somename_for_yourapp')`

![](https://miro.medium.com/max/1400/0*KfpC_-zbNOSnfKTu.png)

[_在此处_](https://devcenter.heroku.com/articles/procfile)_查找有关 Procfile 的更多信息_[](https://devcenter.heroku.com/articles/procfile)

**C。将 Web 应用程序部署到 Heroku**

1.  _初始化本地 git 仓库并提交代码和配置文件_

在这种部署方法中，您需要在您的机器上安装 git。

> 在 Linux 上，您可以使用包管理器安装 git（例如，对于 Ubuntu/Debian 使用命令，对于 RedHat/CentOS 使用命令） `sudo apt install git``sudo yum install git`
> 
> 对于 Windows，您可以下载并安装 git [https://git-scm.com/download/win](https://git-scm.com/download/win)

初始化本地 git 存储库：收集代码并定义必要的配置文件后，初始化本地 git 存储库

`git init`

将目录的所有内容添加到 git repo

`git add .`

提交对本地 git repo 的更改

`git commit -m "some commit message"`

2. _登录 Heroku_

使用 Heroku CLI，登录 Heroku。

`heroku login`

![](https://miro.medium.com/max/1400/0*SWChz90pNBa6QHzJ.png)

要求时按任意键。这将在您的网络浏览器中打开一个选项卡，要求您登录 Heroku。登录 Heroku。

![](https://miro.medium.com/max/1400/0*79NqLDw648uA5oLC.png)

![](https://miro.medium.com/max/1000/0*1UNMaqoihnYJwJYx.png)

现在您可以关闭此选项卡并返回终端上的命令提示符

![](https://miro.medium.com/max/1400/0*aq4V7qIZFNf9FrYt.png)

3. _创建应用程序_

在将代码部署到 Heroku 之前，您需要在 Heroku 中创建一个新应用程序。使用命令创建应用程序

```
heroku create <应用程序名称>例如
heroku 创建 hw5-ml-zoom

```

![](https://miro.medium.com/max/1400/0*d2OLz-s56vOPQ72g.png)

这将创建一个应用程序以及该应用程序的 Heroku git 存储库。您可以转到 Heroku 仪表板查看正在使用默认网页创建的应用程序。

![](https://miro.medium.com/max/1400/0*_ZK1Daa2RlmDvNBQ.png)

![](https://miro.medium.com/max/1400/0*b1I-ZkI-7_4MpS6_.png)

4. _将代码和配置推送到 Heroku git 进行部署_

您现在可以将代码和配置文件推送到远程 git (Heroku git)。将内容推送到 Heroku git 后，它会自动触发部署。

![](https://miro.medium.com/max/1400/0*KaTlOu-PGmv1LIgG.png)

![](https://miro.medium.com/max/1400/0*Yp5kWggcAnjg__D-.png)

恭喜！！！您的应用程序现在已部署到 Heroku。您可以通过访问应用程序的 URL 进行验证。URL 采用以下形式：

[https://<您的应用程序名称>.herokuapp.com/](https://ml-zoom-docker.herokuapp.com/)

例如，对于在 Heroku 中创建的名为 [hw5-ml-zoom](https://ml-zoom-docker.herokuapp.com/) 的应用程序，URL 将是 [https://hw5-ml-zoom.herokuapp.com/](https://ml-zoom-docker.herokuapp.com/)

B. 将应用程序部署为 docker 容器
---------------------

为了能够作为 docker 容器部署到 Heroku，您首先需要安装并运行 docker。为此，您有 2 个选项：

*   当您在本地计算机上安装 docker 时。
*   当您没有本地安装 docker 时，您可以使用 Google Cloud shell（如果您有 Gmail 帐户，Google Cloud shell 会提供免费的机器，并且预装了 docker）。

**一个。准备代码库  
**为您的部署创建一个目录并复制您的代码用于 Web 应用程序、python 包依赖项（此示例显示了 pipenv 的使用以及 Pipfile 和 Pipfile.lock，但是您可以使用 requirements.txt 或适用于您的文件） .

如果您想使用 Google Cloud Shell，因为您的本地计算机上没有 Docker，请参阅此[链接](https://github.com/nindate/ml-zoomcamp-exercises/blob/main/how-to-use-google-cloud-shell-for-docker.md)中的指南。该指南介绍了如何使用 Google Cloud Shell、如何将文件上传到 Google Cloud Shell 以及如何在 Google Cloud Shell 上使用 docker。

接下来的步骤假设您位于此部署目录中。

**湾。创建 Dockerfile**

在同一目录中创建名为 Dockerfile 的文件（文件名应与 D 大写完全一致），并为您的 docker 映像添加适当的行。

下面的例子显示，python:3.8.12-slim 被用作基础镜像，安装 pipenv，然后复制指定 python 依赖包的 Pipfile 和 Pipfile.lock，使用 pipenv 安装依赖包，然后复制 web 应用程序的 python 代码，然后定义要公开的端口和应该在 docker 容器启动时运行的 ENTRYPOINT 命令。

![](https://miro.medium.com/max/1152/0*vo3CX9CvYxOY3aS9.png)

**注意**：当您在本地部署烧瓶应用程序时，您通常可以指定应用程序应侦听的端口。但是，当您部署到 Heroku 时，您不能使用您选择的端口，而是使用默认端口（即 HTTPS 的 443）。因此，如果您要在本地部署一个 docker 容器，您可以使用如下 ENTRYPOINT

`ENTRYPOINT ["gunicorn", "--bind=0.0.0.0:9696", "sample_app:app"]`

但是，当部署到 Heroku 时，ENTRYPOINT 应该如下所示

`ENTRYPOINT ["gunicorn", "sample_app:app"]`

**C。作为 docker 容器部署到 Heroku**

按顺序运行以下步骤，将您的 Web 应用程序作为 docker 容器部署到 Heroku。

1.  **_登录 Heroku_**

如果您从本地计算机执行这些步骤，请按照下面_选项 I._ 中的步骤操作。但是，如果您是从 Google Cloud shell 执行这些步骤，请按照_选项 II 中的步骤操作。_以下。不同之处在于从本地计算机登录的步骤将打开一个 Web 浏览器，您可以在其中提供凭据登录 Heroku。而在 Google Cloud shell 中，无法打开 Web 浏览器选项卡供您登录。因此，对于 Google Cloud shell 场景，您需要使用 Heroku 的 API 密钥。

_选项 I. 从本地机器登录 Heroku_

使用 Heroku CLI，登录 Heroku。要求时按任意键。

`heroku login`

![](https://miro.medium.com/max/1400/0*ck_HhCnRJzDMcqFK.png)

这将在您的网络浏览器中打开一个选项卡，要求您登录 Heroku。登录 Heroku。

![](https://miro.medium.com/max/1400/0*qeqLbb_QIvgbkI51.png)

![](https://miro.medium.com/max/1000/0*dkBmD-977TLrauxJ.png)

现在您可以关闭此选项卡并返回终端上的命令提示符。

您可以跳过_选项二。_下面继续执行 **_2. Login to Heroku Container repository_** 中解释的步骤。

_选项二。从 Google Cloud Shell 登录 Heroku_

从 Heroku 创建 API 身份验证密钥

在您的网络浏览器中，打开另一个选项卡并登录到 Heroko。转到帐户设置。在帐户设置屏幕上，转到应用程序选项卡，然后在授权下单击创建授权

![](https://miro.medium.com/max/1400/0*tUBo54Zm_4L24O2s.png)

为授权提供一些名称，并可选择设置过期时间并创建授权（这将创建一个 API 密钥，可用于从 Google Cloud shell 登录到 Heroku）

![](https://miro.medium.com/max/790/0*ix3fHbEvo765BYw4.png)

![](https://miro.medium.com/max/824/0*8jBXWHD2G02T31T5.png)

复制生成的令牌（即 API 密钥）并转到您的 Google Cloud Shell 提示符。在这里，定义并导出一个名为 HEROKU_API_KEY 的变量，其值作为您复制的键。示例命令如下所示（请注意等号前后不能有空格，键要用双引号括起来）

`export HEROKU_API_KEY="8fb57c22-c44e-4dd5-b5d7-6315705b7bbe"`

![](https://miro.medium.com/max/1400/0*xbmyhoQWsDBD0et1.png)

运行以下步骤将 Google DNS 服务器条目添加到文件 /etc/resolv.conf（我必须这样做才能从 Google Cloud Shell 连接到 Heroku 容器注册表）。

```
$ echo "nameserver 8.8.8.8" > /tmp/1 
$ cat /etc/resolv.conf >> /tmp/1 
$ sudo cp /tmp/1 /etc/resolv.conf

```

2. **_登录 Heroku 容器注册中心_**

`heroku container:login`

![](https://miro.medium.com/max/1400/0*WmaggSlRC1X_ZywZ.png)

3. **_在 Heroku 中创建应用程序_**

在 Heroku 中创建一个应用程序。下面的示例显示了创建一个名为 ml-zoom-docker 的应用程序。

`heroku create ml-zoom-docker`

![](https://miro.medium.com/max/1400/0*6WxT6ecHX2IGy5LB.png)

4. **_将 docker 镜像推送到 Heroku_**

将 docker 镜像推送到 Heroku 容器注册表。当您运行以下命令时，Dockerfile 将用于在您的机器上本地构建 docker 镜像，然后将镜像推送到 Heroku 容器注册表。使用 -a 标志指定应用程序名称（例如，您在上面创建的 ml-zoom-docker 应用程序）。

`heroku container:push web -a ml-zoom-docker`

![](https://miro.medium.com/max/1400/0*KR8FbZ8Y6CzuVHp7.png)

![](https://miro.medium.com/max/1400/0*Cev_7M-rysQZr6Ky.png)

发布容器：在 Heroku 上部署容器。当您运行以下命令时，将在 Heroku 中从您推送到 Heroku 容器注册表的 docker 映像启动一个 docker 容器。

`heroku container:release web -a ml-zoom-docker`

![](https://miro.medium.com/max/1400/0*p1tj_Z8fvEFU5m0P.png)

5. **_启动你的应用_**

打开您的网络应用程序。为此，您可以转到您的网络浏览器并使用适当的路径打开您的应用程序 URL（例如 [https://ml-zoom-docker.herokuapp.com/welcome](https://ml-zoom-docker.herokuapp.com/welcome)，因为在 Heroku 中创建的应用程序名为 ml-zoom-docker 并且路径通过应用程序提供服务是 /welcome）

![](https://miro.medium.com/max/1400/0*XIvOmBLYtnbqw9xP.png)

伟大的 ！！！您的 Web 应用程序现在作为 docker 容器在 Heroku 中运行！！！

本文的目的是让您了解 Heroku，并指导您在 Heroku 上托管 Python Web 应用程序。将应用程序托管到 Heroku 时，您可以做更多的事情，例如：

*   监控应用程序的运行状况，监控应用程序的指标，获取请求失败警报，在有更多需求时自动扩展应用程序
*   使用 Heroku 数据库即服务将您的应用程序连接到数据库。Heroku 目前提供 Postgres、Redis 或 Apache Kafka 作为服务
*   和更多 …

希望您喜欢这篇文章并从中受益。

祝你有美好的一天 ！！！