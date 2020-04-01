# Docker最佳实践的反模式
## 提醒和经验教训，要避免的陷阱
![Photo by Belinda Fewings on Unsplash](1!8ISg7iGJi1iydzAfeSirMg.jpeg)
> Photo by Belinda Fewings on Unsplash


这是docker最佳做法指南，只有一个类似的指南，我会尽我所能

老实说，当我们大多数时间在docker上工作时，我很多人绝对不知道我们在做什么。 仅仅因为您启动了一个Docker容器并且您的应用程序正在运行并不意味着您已经实现了一个好的解决方案—有时由于时间限制，我们陷入了复制粘贴docker映像的陷阱，而没有真正理解底层的实现细节和细微差别 图像的构建方式。

在本篇文章中，我们将研究Docker反模式，也称为对重复出现的问题的常见反应，通常会导致我们实施无效和适得其反的解决方案，从而破坏我们的Docker堆栈。 我们来看看一些我们可能做错的事情。
# 我们需要那些标签

使用Docker映像时，必须添加标签，因为它可以传达有关映像的有用信息。 将标签视为您的Docker映像ID的别名。 可以将标签与引用您历史记录中特定提交的Git标签进行比较。 因此，您可以在不同的时间点对Docker映像进行版本控制。 忘记标记是一件微不足道的事情，它会带来一些反冲。 也就是说，如果未指定标签，则默认图片将使用最新示例your_image_name：latest进行标注。

如果您经常进行此操作，则您的映像可能实际上不是最新的，而是可能指向了较旧的版本。 始终确保使用适当的标签并遵守版本控制架构，例如语义版本控制。 这确保Docker映像的用户可以保证兼容性并保持更新，并以可预测的方式利用正确的版本。

您可能要避免使用最新默认标签的另一件事是，例如在Dockfile中使用FROM python3：latest从Docker注册表中提取最新映像。 乍一看，这似乎是个好主意，但一些意外的副作用是，每个最新的请求可能会衍生出与先前构建不同的整个docker映像。 尝试破译破坏您的Docker映像的内容可能很困难，因为从您的角度出发，您期望不可变。 因此，强烈建议为图像使用特定标签（例如：python3：1.0.1）—这将确保您的Dockerfile保持不变。
# 在同一个容器中运行多个服务

是的，无论如何，您可以做到这一点-可能成功了，但这是您可能不想走这条路的两个原因。 当我们使用docker服务时，我们应该努力实现单一责任。 公认的最佳做法是，组成您的应用程序的每个不同服务都应在其自己的容器中运行-尝试将每项独立功能打包到单独的独立容器映像中。

试图将多个服务添加到一个docker映像中是很诱人的，将容器映像视为虚拟机的想法也应该从人们的想法中删除。

一个容器中有多个服务，因此很难水平扩展您的应用程序。 Docker容器的核心概念是它们是瞬态的并且是为分发而设计的，这对于现代Web应用程序是理想的，瞬态特性简化了扩展和并发性，添加多个服务增加了管理此分发的难度。

此外，在单个容器上的多种服务使管理安全性变得更加困难且映像大小过大，如果您担心的话，这可能会降低CI / CD的速度。 Docker文档在进一步详细说明方面做得很好。
# 使用标签对图像进行分类

这绝不是反模式，但我认为这值得一提。 我在使用各种docker映像时注意到的一件事是，有时这些映像的创建者没有LABEL维护者标签。 如果发生错误或需要澄清时，此标记可在事件中设置图像的“作者”字段，如果图像是公开共享的，则更容易知道内部或外部与谁联系。 这绝不是您可以使用的唯一标签。 您可以根据需要定义各种标签，以对图像进行分类，记录许可信息或对自动化有用的标签。

除维护者以外的多行标签
```
# Set one or more individual labelsLABEL com.example.version="0.0.1-beta"LABEL vendor1="ACME Incorporated"LABEL vendor2=ZENITH\ IncorporatedLABEL com.example.release-date="2015-02-12"LABEL com.example.version.is-production=""
```

在docker 1.10之前的单行标签会创建新的docker层，因此，如果您使用最新的docker版本，则不必担心开销层。
```
LABEL vendor=ACME\ Incorporated \      com.example.is-beta= \      com.example.is-production="" \      com.example.version="0.0.1-beta" \      com.example.release-date="2015-02-12"
```

将尽可能多的元数据添加到不可变的Docker映像中始终是一个好习惯，以实现可追溯性，可见性和可维护性。
# 避免构建环境特定的图像

当我们构建docker镜像时，我们应该始终牢记不变性。 最好不要使用标记为例如dev，test，staging和production的不同图像，因为这会使单一真理来源的原则无效。 可能出现的另一个问题是，在不同环境上执行验证或调试时，无法保证图像本质上是相似的。
# 为什么要使用非根容器？

默认情况下，Docker容器以root身份运行。 以root用户身份运行的Docker容器可以完全控制主机系统。 由于安全考虑，这是不希望的。 使用非根目录运行的Docker容器映像会增加一层安全性，通常建议在生产环境中使用。 但是，由于特权任务通常以非root用户身份运行，因此它们通常是禁止访问的，因此，如果需要利用USER指令指定非root用户，您将需要进行一些上下文切换，如下例所示。
```
FROM python:3.6-slim-busterLABEL maintainer="Timothy Mugayi <timothy.mugayi@gmail.com>"RUN apt-get update && apt-get install -y --no-install-recommends \    wget && rm -rf /var/lib/apt/lists/*# Dumb initRUN wget -O /usr/local/bin/dumb-init https://github.com/Yelp/dumb-init/releases/download/v1.2.0/dumb-init_1.2.0_amd64RUN chmod +x /usr/local/bin/dumb-initRUN pip install --upgrade pipWORKDIR /usr/src/appCOPY requirements.txt .RUN pip install -r requirements.txtCOPY helloworld.py .USER 1001ENTRYPOINT ["/usr/local/bin/dumb-init", "python3", "-u", "./helloworld.py"]
```

如果您使用的不是使用root生成的基本映像，而必须切换回去，则可以执行以下操作
```
FROM <namespace>/<image>:<tag_version>USER root
```
# 不要在单个容器中运行太多进程

容器的优点以及容器相对于虚拟机的优势在于，很容易使多个容器相互交互以组成一个完整的应用程序。 无需在单个容器中运行完整的应用程序。 相反，应将您的应用程序尽可能分解为离散的服务，然后在多个容器之间分配服务。 这样可以最大程度地提高灵活性和可靠性。
# 不要在Docker容器中安装操作系统

如果您使用docker的时间足够长，则可能在某个时候遇到了这个问题-是的，可以在容器中安装和运行完整的Linux操作系统。 你应该这样做吗？ 可能不是一个好主意。 Docker映像是使用层的概念构建的，因此，添加到映像的数量越多，映像就越膨胀。 因此，拥有完善的操作系统会使docker成为理想的用例。 理想情况下，您只需要在容器内部安装必要的必要部件即可。
# 不要在容器内运行不必要的服务

为了充分利用容器，您希望每个容器都尽可能地瘦。 这样可以最大程度地提高性能，并最大程度地降低安全风险。 因此，请避免运行并非绝对必要的服务。 例如，除非在容器内运行SSH服务是绝对必要的（可能不是这种情况，因为还有其他登录到容器的方法（例如，通过docker exec调用）），否则不包括SSH 服务。
# 不要使用构建时依赖项和工具来堆叠运行时映像

这是一个必须知道的反模式。 与docker合作时，倾向于使用声纳之类的工具覆盖我们的图像，以进行代码覆盖。

使用“生成器模式”。 Docker Multistage构建。 由于Docker CE 17.05+在单个Dockerfile中，因此您现在可以具有多个FROMstage。 临时构建器阶段容器将被丢弃，以便最终的运行时容器映像将保持精简。

衍生权益：
+ 构建速度更快，即您可以通过更精简的映像来改进CI / CD流程，从而减少通过网络传输的时间
+ 需要更少的存储空间
+ 冷启动（图像拉动）更快
+ 潜在的攻击面更少

缺点：
+ 容器内的工具较少，但要付出一定的代价以保持容器的倾斜
# 在您的容器中包括Oberservablity工具

您可以在不监视解决方案的情况下运行容器，但是需要牢记的一点是，从本质上来说，您很难知道容器内部的情况，尤其是随着容器数量的增加。 利用Docker随时为您提供的静态功能，Docker开箱即用地通过Docker远程API的/ stats端点公开了有关每个运行容器的CPU，内存，网络和I / O使用情况的非常详细的指标。 App dynamics和Newrelic提到了一些现成的代理程序，这些代理程序可以与docker映像预先打包在一起，以提供更多可见性，包括有关应用程序和容器性能的应用程序级可见性。
# 爱恨任意基地图片

认真地说，您知道谁制作了图片以及添加了哪些图片吗？

一切都与可追溯性有关，牢记安全性是所有软件开发人员应始终追求的目标。 了解如何跟踪Docker映像的来源并了解Docker映像的内容。 它们是您需要牢记的几件事。
+ 图像是如何创建的
+ 验证其创建后未更改
+ 验证图像的内容
+ 扫描安全漏洞

您可能想开始使用一些工具来探索容器的静态分析。 这些工具非常全面，超出了本文的范围，因此，我建议您花一些时间，看看它们如何工作。

Clair是一个有趣的工具，可为您的Docker应用程序提供自动容器漏洞和安全扫描。 扫描基于“常见漏洞和披露（CVE）”数据库。 如果您在本地运行docker，可以通过下载Postgres并将Clair链接到它来尝试一下。 以下是启动和运行所需的最低配置。
```
$ mkdir $PWD/clair_config$ curl -L https://raw.githubusercontent.com/coreos/clair/master/config.yaml.sample -o $PWD/clair_config/config.yaml$ docker run -d -e POSTGRES_PASSWORD="" -p 5432:5432 postgres:9.6$ docker run --net=host -d -p 6060-6061:6060-6061 -v $PWD/clair_config:/config quay.io/coreos/clair:latest -config=/config/config.yaml
```

如果您希望更改Postgres的端口，请确保同时更改config.yaml文件。 如果您的系统上已经运行了另一个Postgres，请不要将docker端口向前更改为5432以外的其他端口。
```
clair:  database:    # Database driver    type: pgsql    options:      # PostgreSQL Connection string      # https://www.postgresql.org/docs/current/static/libpq-connect.html#LIBPQ-CONNSTRING      source: host=localhost port=5432 user=postgres password=123456 sslmode=disable statement_timeout=60000
```

Bayan Collector —是一款轻巧的应用程序，可让您通过从注册表启动容器，运行任意脚本并收集有用的信息（例如，已安装的软件包），执行策略，验证图像中的不变量来执行静态分析。

Docker Bench for Security-由Docker团队创建的工具可帮助您遍历安全最佳实践清单，以遵守Docker主机并标记发现的任何问题。
# 不要在容器图片中存储敏感数据

这是您要避免的一个错误，如果您的Docker镜像是公开共享的，或者如果开发人员无意中将图像推送到了公共Docker注册表，则可能导致泄漏私人和敏感信息到外界。 因此，切勿在Dockerfile中将COPY或ADD与敏感信息一起使用。

为避免这种情况，请将敏感数据存储在容器可以连接到的安全文件系统上。 在大多数情况下，此文件系统将存在于容器主机上，或者可通过诸如AWS Elastic Block Storage（EBS）之类的块存储或诸如S3之类的对象存储服务来使用。

此外，您应避免在Docker映像中存储安全凭证。 作为开发人员，我们有时会尝试捷径，硬编码密码和秘密Keyes。 养成在运行时使用-e参数为docker容器指定环境变量的习惯。 利用— env文件，也可以从文件中读取环境变量。 通过CMD或ENTRYPOINT从第三方来源提取凭证的自定义脚本也可以用于提取相关凭证。 docker容器需要。
# 不要在容器中存储数据或日志

容器化改变了日志记录的性质，容器非常适合无状态应用，这些应用本质上是瞬态的，并且是短暂的（短暂的）。 本质上，可能存储在正在运行的容器中的所有数据都是短暂的，因此，如果您注意到容器关闭时的数据将丢失。 有必要将数据存储在Docker容器之外以供我们使用，我们可以利用各种工具来帮助您将Docker日志提取到更多永久数据存储中。

在处理如何使用Docker解决日志记录时要记住的一点。 Docker安装至少具有3种不同的日志记录级别：Docker容器，Docker服务和主机操作系统，您选择的日志记录应该能够提取所有层上的日志。
# 不要写入容器的文件系统

每次将内容写入容器的文件系统时，它都会激活“写时复制”策略。 使用存储驱动程序（devicemapper，overlayfs或其他）创建新的存储层。 在活动使用的情况下，它可能会给存储驱动程序带来很多负担，尤其是在Devicemapper或BTRFS的情况下。

确保您的容器仅将数据写入卷。 您可以将tmpfs用于小型临时文件，因为tmpfs是驻留在内存和/或交换分区中的临时文件系统，因此tmpfs将所有内容存储在内存中
# 不要运行PID 1

这是大多数人可能不知道的常见问题。 Docker不会在能够正确获得子进程的特殊初始化进程下运行进程，这样-容器最终可能会死于僵尸进程，这可能会引起意想不到的问题，至少您不会意识到。
## 使用tini或dumb-init

PID 1在UNIX中是特殊的，因此省略初始化系统通常会导致错误地处理进程和信号，并可能导致出现问题，例如无法正常停止的容器或泄漏本应被破坏的容器。

僵尸进程是已经停止运行的进程，但是它们的进程表条目仍然存在，因为父进程尚未通过wait syscall检索到它。 从技术上讲，每个终止过程在很短的时间内都是僵尸，但它们可以生存更长的时间。

如果您的进程产生了新进程，但没有实现良好的信号处理程序来捕获子信号并在应停止进程（例如bash脚本）的情况下停止了您的孩子，则可以使用tini或dumb-init，例如，不要处理 并正确发出信号。

这是一个如何运行dumb init的示例，其中prepare.sh是Shell脚本或执行应用程序的命令
```
RUN wget -O /usr/local/bin/dumb-init https://github.com/Yelp/dumb-init/releases/download/v1.2.0/dumb-init_1.2.0_amd64RUN chmod +x /usr/local/bin/dumb-initENTRYPOINT ["/usr/local/bin/dumb-init", "/usr/bin/prepare.sh"]
```

或者，如果您选择Tini
```
RUN conda install --yes \    -c conda-forge tiniENTRYPOINT ["tini", "-g", "--", "/usr/bin/prepare.sh"]
```
# 最后的想法

感谢您的阅读，如果您还有其他反模式，请避免发表评论。 如果您认为该内容对下一位开发人员有用，请继续进行。

快乐的码头工人形象建设。
```
(本文翻译自Timothy Mugayi的文章《Docker Best Practises Anti-Patterns For the Layman》，参考：https://medium.com/swlh/docker-best-practises-anti-patterns-for-the-layman-63b22e2b981f)
```
