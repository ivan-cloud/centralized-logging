# 日志聚合和可视化
标签（空格分隔）： 架构 日志 聚合

---

# 一、背景需求&常见方案
## 1、日志分类和作用

![image.png](https://upload-images.jianshu.io/upload_images/1636821-a0712e0d249ec0e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2、传统日志管理方案&存在的问题

![image.png](https://upload-images.jianshu.io/upload_images/1636821-b103478ec06c4db9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 3、常用日志聚合方案
1. Elastic Stack（ELK）
>&nbsp;&nbsp;&nbsp;&nbsp;[ELK](https://www.elastic.co/webinars/introduction-elk-stack)，即 Elasticsearch、Logstash 和 Kibana 简称，是最流行的开源日志聚合工具。它被 Netflix、Facebook、微软、LinkedIn 和思科使用。这三个组件都是由 [Elastic](https://www.elastic.co/) 开发和维护的。[Elasticsearch](https://www.elastic.co/products/elasticsearch) 本质上是一个 NoSQL 数据库，以 Lucene 搜索引擎实现的。[Logstash](https://www.elastic.co/products/logstash) 是一个日志管道系统，可以接收数据，转换数据，并将其加载到像 Elasticsearch 这样的应用中。[Kibana](https://www.elastic.co/products/kibana) 是 Elasticsearch 之上的可视化层。
<br>&nbsp;&nbsp;&nbsp;&nbsp;几年前，引入了 Beats 。Beats 是数据采集器。它们简化了将数据运送到 Logstash 的过程。用户不需要了解每种日志的正确语法，而是可以安装一个 Beats 来正确导出 NGINX 日志或 Envoy 代理日志，以便在 Elasticsearch 中有效地使用它们。

2. Graylog 
>&nbsp;&nbsp;&nbsp;&nbsp;[Graylog](https://www.graylog.org/) 最近越来越受欢迎，但它是在 2010 年由 Lennart Koopmann 创建并开发的。两年后，一家公司以同样的名字诞生了。尽管它的使用者越来越多，但仍然远远落后于 ELK 套件。这也意味着它具有较少的社区开发特征，但是它可以使用与 ELK 套件相同的 Beats 。由于 Graylog Collector Sidecar 使用 [Go](https://opensource.com/tags/go) 编写，所以 Graylog 在 Go 社区赢得了赞誉。
<br>&nbsp;&nbsp;&nbsp;&nbsp;Graylog 使用 Elasticsearch、[MongoDB](https://www.mongodb.com/) 和底层的 Graylog Server 。这使得它像 ELK 套件一样复杂，也许还要复杂一些。然而，Graylog 附带了内置于开源版本中的报警功能，以及其他一些值得注意的功能，如流、消息重写和地理定位。

3. Fluentd
>&nbsp;&nbsp;&nbsp;&nbsp;[Fluentd](https://www.fluentd.org/) 是 [Treasure Data](https://www.treasuredata.com/) 开发的，[CNCF](https://www.cncf.io/) 已经将它作为一个孵化项目。它是用 C 和 Ruby 编写的，并被 [AWS](https://aws.amazon.com/blogs/aws/all-your-data-fluentd/) 和 [Google Cloud](https://cloud.google.com/logging/docs/agent/) 所推荐。Fluentd 已经成为许多系统中 logstach 的常用替代品。它可以作为一个本地聚合器，收集所有节点日志并将其发送到中央存储系统。它不是日志聚合系统。

TODO: Fluentd vs Beats
![image.png](https://upload-images.jianshu.io/upload_images/1636821-ed5ecb18b75c1ad4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. 阿里云日志服务（云计算）
![image.png](https://upload-images.jianshu.io/upload_images/1636821-2460cb748a156f61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 4、日志采集工具对比

### Logstash
Logstash 不是这个列表里最老的传输工具（最老的应该是 syslog-ng ，讽刺的是它也是唯一一个名字里带有 new 的），但 Logstash 绝对可以称得上最有名的。因为它有很多插件：输入、编解码器、过滤器以及输出。基本上，可以获取并丰富任何数据，然后将它们推送到多种目标存储。

- **优势**

Logstash 主要的有点就是它的**灵活性**，这还主要因为它有很多插件。然后它清楚的文档已经直白的配置格式让它可以再多种场景下应用。这样的良性循环让我们可以在网上找到很多资源，几乎可以处理任何问题。

- **劣势**

Logstash 致命的问题是它的**性能以及资源消耗**（默认的堆大小是 1GB）。尽管它的性能在近几年已经有很大提升，与它的替代者们相比还是要慢很多的。这里有 Logstash 与 rsyslog 性能对比以及Logstash 与 filebeat 的性能对比。它在大数据量的情况下会是个问题。

另一个问题是它目前**不支持缓存**，目前的典型替代方案是将 Redis 或 Kafka 作为中心缓冲池：

- **典型应用场景**

因为 Logstash 自身的灵活性以及网络上丰富的资料，Logstash 适用于原型验证阶段使用，或者解析非常的复杂的时候。在不考虑服务器资源的情况下，如果服务器的性能足够好，我们也可以为每台服务器安装 Logstash 。我们也不需要使用缓冲，因为文件自身就有缓冲的行为，而 Logstash 也会记住上次处理的位置。

如果服务器性能较差，并不推荐为每个服务器安装 Logstash ，这样就需要一个轻量的日志传输工具，将数据从服务器端经由一个或多个 Logstash 中心服务器传输到 Elasticsearch。

随着日志项目的推进，可能会因为性能或代价的问题，需要调整日志传输的方式（log shipper）。当判断 Logstash 的性能是否足够好时，重要的是对吞吐量的需求有着准确的估计，这也决定了需要为 Logstash 投入多少硬件资源。


### Filebeat

作为 Beats 家族的一员，Filebeat 是一个轻量级的日志传输工具，它的存在正弥补了 Logstash 的缺点：Filebeat 作为一个轻量级的日志传输工具可以将日志推送到中心 Logstash。

在版本 5.x 中，Elasticsearch 具有解析的能力（像 Logstash 过滤器）— Ingest。这也就意味着可以将数据直接用 Filebeat 推送到 Elasticsearch，并让 Elasticsearch 既做解析的事情，又做存储的事情。也不需要使用缓冲，因为 Filebeat 也会和 Logstash 一样记住上次读取的偏移：

如果需要缓冲（例如，不希望将日志服务器的文件系统填满），可以使用 Redis/Kafka，因为 Filebeat 可以与它们进行通信：

- **优势**

Filebeat 只是一个二进制文件没有任何依赖。它**占用资源极少**，尽管它还十分年轻，正式因为它简单，所以几乎没有什么可以出错的地方，所以它的可靠性还是很高的。它也为我们提供了很多可以调节的点，例如：它以何种方式搜索新的文件，以及当文件有一段时间没有发生变化时，何时选择关闭文件句柄。

- **劣势**

Filebeat 的**应用范围十分有限**，所以在某些场景下我们会碰到问题。例如，如果使用 Logstash 作为下游管道，我们同样会遇到性能问题。正因为如此，Filebeat 的范围在扩大。开始时，它只能将日志发送到 Logstash 和 Elasticsearch，而现在它可以将日志发送给 Kafka 和 Redis，在 5.x 版本中，它还具备过滤的能力。

- **典型应用场景**

Filebeat 在解决某些特定的问题时：日志存于文件，我们希望将日志直接传输存储到 Elasticsearch。这仅在我们只是抓去（grep）它们或者日志是存于 JSON 格式（Filebeat 可以解析 JSON）。或者如果打算使用 Elasticsearch 的 Ingest 功能对日志进行解析和丰富。

将日志发送到 Kafka/Redis。所以另外一个传输工具（例如，Logstash 或自定义的 Kafka 消费者）可以进一步丰富和转发。这里假设选择的下游传输工具能够满足我们对功能和性能的要求。

### Fluentd

Fluentd 创建的初衷主要是尽可能的使用 JSON 作为日志输出，所以传输工具及其下游的传输线不需要猜测子字符串里面各个字段的类型。这样，它为几乎所有的语言都提供库，这也意味着，我们可以将它插入到我们自定义的程序中。

- **优势**

和多数 Logstash 插件一样，Fluentd 插件是用 Ruby 语言开发的非常易于编写维护。所以它数量很多，几乎所有的源和目标存储都有插件（各个插件的成熟度也不太一样）。这也意味这我们可以用 Fluentd 来串联所有的东西。

- **劣势**

因为在多数应用场景下，我们会通过 Fluentd 得到结构化的数据，它的**灵活性并不好**。但是我们仍然可以通过正则表达式，来解析非结构化的数据。尽管，性能在大多数场景下都很好，但它并不是最好的，和 syslog-ng 一样，它的缓冲只存在与输出端，单线程核心以及 Ruby GIL 实现的插件意味着它大的节点下**性能是受限的**，不过，它的资源消耗在大多数场景下是可以接受的。对于小的或者嵌入式的设备，可能需要看看 Fluent Bit，它和 Fluentd 的关系与 Filebeat 和 Logstash 之间的关系类似

- **典型应用场景**

Fluentd 在日志的数据源和目标存储各种各样时非常合适，因为它有很多插件。而且，如果大多数数据源都是自定义的应用，所以可以发现用 fluentd 的库要比将日志库与其他传输工具结合起来要容易很多。特别是在我们的应用是多种语言编写的时候，即我们使用了多种日志库，日志的行为也不太一样。

### Logagent

Logagent 是 Sematext 提供的传输工具，它用来将日志传输到 Logsene（一个基于 SaaS 平台的 Elasticsearch API），因为 Logsene 会暴露 Elasticsearch API，所以 Logagent 可以很容易将数据推送到 Elasticsearch 。

- **优势**

可以获取 /var/log 下的所有信息，解析各种格式（Elasticsearch，Solr，MongoDB，Apache HTTPD等等），它可以掩盖敏感的数据信息，例如，个人验证信息（PII），出生年月日，信用卡号码，等等。它还可以基于 IP 做 GeoIP 丰富地理位置信息（例如，access logs）。同样，**它轻量又快速**，可以将其置入任何日志块中。在新的 2.0 版本中，它以第三方 node.js 模块化方式增加了支持对输入输出的处理插件。重要的是 Logagent **有本地缓冲**，所以不像 Logstash ，在数据传输目的地不可用时会丢失日志。

- **劣势**

尽管 Logagent 有些比较有意思的功能（例如，接收 Heroku 或 CloudFoundry 日志），但是它并**没有 Logstash 灵活**。

- **典型应用场景**

Logagent 作为一个可以做所有事情的传输工具是值得选择的（提取、解析、缓冲和传输）。


### logtail

阿里云日志服务的生产者，目前在阿里集团内部机器上运行，经过3年多时间的考验，目前为阿里公有云用户提供日志收集服务。

采用C++语言实现，对稳定性、资源控制、管理等下过很大的功夫，性能良好。相比于logstash、fluentd的社区支持，logtail功能较为单一，专注日志收集功能。

- **优势**

logtai**l占用机器cpu、内存资源最少**，结合阿里云日志服务的E2E体验良好。

- **劣势**

logtail目前**对特定日志类型解析的支持较弱**，后续需要把这一块补起来。


### rsyslog

绝大多数 Linux 发布版本默认的 syslog 守护进程，rsyslog 可以做的不仅仅是将日志从 syslog socket 读取并写入 /var/log/messages 。它可以提取文件、解析、缓冲（磁盘和内存）以及将它们传输到多个目的地，包括 Elasticsearch 。可以从此处找到如何处理 Apache 以及系统日志。

- **优势**

rsyslog 是经测试过的**最快的传输工具**。如果只是将它作为一个简单的 router/shipper 使用，几乎所有的机器都会受带宽的限制，但是它非常擅长处理解析多个规则。它基于语法的模块（mmnormalize）无论规则数目如何增加，它的处理速度始终是线性增长的。这也就意味着，如果当规则在 20-30 条时，如解析 Cisco 日志时，它的性能可以大大超过基于正则式解析的 grok ，达到 100 倍（当然，这也取决于 grok 的实现以及 liblognorm 的版本）。

它同时也是我们能找到的最轻的解析器，当然这也取决于我们配置的缓冲。

- **劣势**

rsyslog 的**配置工作需要更大的代价**（这里有一些例子），这让两件事情非常困难：

文档难以搜索和阅读，特别是那些对术语比较陌生的开发者。

5.x 以上的版本格式不太一样（它扩展了 syslogd 的配置格式，同时也仍然支持旧的格式），尽管新的格式可以兼容旧格式，但是新的特性（例如，Elasticsearch 的输出）只在新的配置下才有效，然后旧的插件（例如，Postgres 输出）只在旧格式下支持。

尽管在配置稳定的情况下，rsyslog 是可靠的（它自身也提供多种配置方式，最终都可以获得相同的结果），它还是存在一些 bug 。

- **典型应用场景**

rsyslog 适合那些非常轻的应用（应用，小VM，Docker容器）。如果需要在另一个传输工具（例如，Logstash）中进行处理，可以直接通过 TCP 转发 JSON ，或者连接 Kafka/Redis 缓冲。

rsyslog 还适合我们对性能有着非常严格的要求时，特别是在有多个解析规则时。那么这就值得为之投入更多的时间研究它的配置。


### syslog-ng

可以将 syslog-ng 当作 rsyslog 的替代品（尽管历史上它们是两种不同的方式）。它也是一个模块化的 syslog 守护进程，但是它可以做的事情要比 syslog 多。它可以接收磁盘缓冲并将 Elasticsearch HTTP 作为输出。它使用 PatternDB 作为语法解析的基础，作为 Elasticsearch 的传输工具，它是一个不错的选择。

- **优势**

和 rsyslog 一样，作为一个**轻量级的传输工具**，它的性能也非常好。它曾经比 rsyslog 慢很多，但是 2 年前能达到 570K Logs/s 的性能并不差。并不像 rsyslog ，它有着明确一致的配置格式以及完好的文档。

- **劣势**

Linux 发布版本转向使用 rsyslog 的原因是 syslog-ng 高级版曾经有很多功能在开源版中都存在，但是后来又有所限制。我们这里只关注与开源版本，所有的日志传输工具都是开源的。现在又有所变化，例如磁盘缓冲，曾经是高级版存在的特性，现在开源版也有。但有些特性，例如带有应用层的通知的可靠传输协议（reliable delivery protocol）还没有在开源版本中。

- **典型应用场景**

和 rsyslog 类似，可能将 syslog-ng 部署在资源受限的环境中，但仍希望它能在处理复杂计算时有着良好的性能。如果使用 rsyslog ，它可以输出至 Kafka ，以 Kafka 作为一个中心队列，并以 Logstash 作为一个自定义消费者。

不同的是，**syslog-ng 使用起来比 rsyslog 更容易，性能没有 rsyslog 那么极致**：例如，它只对输出进行缓冲，所以它所有的计算处理在缓冲之前就完成了，这也意味着它会给日志流带来压力。

# 二、Elastic Stack

## 1、产品架构&介绍
### 1.1 传统ELK
直到一两年前，ELK Stack是三个开源产品的集合 -  [Elasticsearch](https://logz.io/tag/elasticsearch/)，  [Logstash](https://logz.io/tag/logstash/)和[Kibana--](https://logz.io/tag/kibana/)全部由[Elastic](https://www.elastic.co/)开发，管理和维护。

![下载 (1).png](https://upload-images.jianshu.io/upload_images/1636821-b6e046539b9bca06.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Logstash在JVM上运行并消耗大量资源来执行此操作。关于Logstash显着的内存消耗，许多讨论都在浮动。显然，当您想要从小型机器（例如AWS微实例）发送日志而不损害应用程序性能时，这可能是一个巨大的挑战。

### 1.2 从ELK到BELK（Elastic Stack）

最新版本的Logstash和ELk Stack已经改善了这种固有的弱点。使用Filebeat和/或Elasticsearch Ingest Node，可以将一些处理外包给堆栈中的其他组件。您还可以使用监视API来识别瓶颈和有问题的处理。

Logstash的Java执行引擎（在版本6.3中公布为试验版）默认在版本7.x中启用。它取代了旧的Ruby执行引擎，拥有更好的性能，更少的内存使用和整体 - 更快的体验。

Beats的引入和后续添加将堆栈变成了一个四脚的项目，并导致堆栈重命名为Elastic Stack。

Filebeat是一个非常轻量级的托运者，占用空间小，虽然很少发现有关Filebeat的抱怨，但在某些情况下，您可能会遇到高CPU使用率。

影响所用计算能力的一个因素是扫描频率 - Filebeat配置为扫描文件的频率。可以使用Filebeat配置文件中的scan_frequency设置为每个探测器定义此频率，因此如果您有大量探测器以严格的扫描频率运行，则可能导致CPU使用率过高。
![下载.png](https://upload-images.jianshu.io/upload_images/1636821-67ccf3f8c8b2c2ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 1.3 Elastic Stack生态

![image.png](https://upload-images.jianshu.io/upload_images/1636821-8868900dda08b37e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![Jietu20190602-112754.jpg](https://upload-images.jianshu.io/upload_images/1636821-ceb22ad68b94bae6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其它参考：[Elastic发展历程](https://www.elastic.co/cn/about/history-of-elasticsearch#group-0)

## 2、部署Elastic Stack最简方案
本节介绍部署Elastic Stack最简方案：Filebeat+Elasticsearch+Kibana方案，示例使用Filebeat收集应用日志文件，通过Elasticsearch的pipeline进行字段摄取再进行存储，最后使用Kibana进行可视化查询。

![image.png](https://upload-images.jianshu.io/upload_images/1636821-675631f18f5415e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


应用日志内容示例，它存在单行的日志，也存在包含堆栈的多行日志：
```
[2019-05-30 15:25:23.609] [logging_demo] [pool-2-thread-2] INFO  ylz.test.TimerTaskFactory - 这是一个info log, tick: 1559201123609
[2019-05-30 15:25:23.609] [logging_demo] [pool-2-thread-4] ERROR ylz.test.TimerTaskFactory - 运行时异常:
java.lang.Exception: 这是一个error log, tick: 1559201123609
	at ylz.test.TimerTaskFactory$2.run(TimerTaskFactory.java:28)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.runAndReset(FutureTask.java:308)
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$301(ScheduledThreadPoolExecutor.java:180)
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:294)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)
```

### 2.1 安装&启动Elasticsearch
下载安装包：[https://www.elastic.co/cn/downloads/elasticsearch](https://www.elastic.co/cn/downloads/elasticsearch)

命令行启动：
```
linux & Mac:
./bin/elasticsearch
```
守护进程启动&关闭：
```
./bin/elasticsearch -d -p pid
```
```
pkill -F pid
```
访问端口（默认）：[http://localhost:9200/](http://localhost:9200/)

### 2.2 安装&启动Kibana
下载安装包：[https://www.elastic.co/cn/downloads/kibana](https://www.elastic.co/cn/downloads/kibana)

命令行启动：
```
> linux & Mac:
./bin/kibana

> windows:
.\bin\kibana.bat
```
访问端口（默认）：[http://localhost:5601](http://localhost:5601/)

### 2.3 安装&启动FileBeat
下载安装包：[https://www.elastic.co/downloads/beats/filebeat](https://www.elastic.co/downloads/beats/filebeat)

配置./filebeat.yml：
```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    ### 此处配置需要收集的日志路径
    - /Users/ivan/Projects/logging_demo/logs/*.log
output.elasticsearch:
  hosts: ["127.0.0.1:9200"]
```
启动filebeat:
```
> linux:
./filebeat -e
```
启动完毕后，可在Kibana中看到的结果：

![image.png](https://upload-images.jianshu.io/upload_images/1636821-99347ee49c17b061.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2.4 Kibana操作介绍
Kibana几个常用菜单：

![image.png](https://upload-images.jianshu.io/upload_images/1636821-8b7e30c6aca929f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- #### Managemnt
>[Management](https://github.com/elastic/kibana/edit/7.1/docs/management.asciidoc "Edit this page on GitHub")
The Management application is where you perform your runtime configuration of Kibana, including both the initial setup and ongoing configuration of index patterns, advanced settings that tweak the behaviors of Kibana itself, and the various "objects" that you can save throughout Kibana such as searches, visualizations, and dashboards.
Management是您执行Kibana的运行时配置的地方，包括索引模式的初始设置和持续配置，调整Kibana本身行为的高级设置，以及您可以在整个Kibana中保存的各种“对象”，例如搜索，可视化和仪表板。


&nbsp;&nbsp;&nbsp;&nbsp;使用Beats或LogStash采集数据到Elasticsearch后，可在Index Management中看到指定名称的Index：

![image.png](https://upload-images.jianshu.io/upload_images/1636821-b2a7ace8ec96bb63.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Kibana使用Index Pattern从ElasticSearch索引中检索诸如可视化之类的数据，因此在使用Discover前，需要先创建Create index pattern：

![image.png](https://upload-images.jianshu.io/upload_images/1636821-401425b0d1250f8b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- #### Discover
>[Discover](https://github.com/elastic/kibana/edit/7.1/docs/discover.asciidoc "Edit this page on GitHub")
**Discover** enables you to explore your data with Kibana’s data discovery functions. You have access to every document in every index that matches the selected index pattern. You can submit search queries, filter the search results, and view document data. You can also see the number of documents that match the search query and get field value statistics. If a time field is configured for the selected index pattern, the distribution of documents over time is displayed in a histogram at the top of the page.
**Discover使您可以使用Kibana的数据发现功能探索数据。您可以访问与所选索引模式匹配的每个索引中的每个文档。您可以提交搜索查询，过滤搜索结果以及查看文档数据。您还可以查看与搜索查询匹配的文档数量并获取字段值统计信息。如果为所选索引模式配置了时间字段，则文档随时间的分布将显示在页面顶部的直方图中。**

##### KQL查询语法

查询数据时，可以使用KQL（[Kibana Query Language]([https://www.elastic.co/guide/en/kibana/7.1/kuery-query.html](https://www.elastic.co/guide/en/kibana/7.1/kuery-query.html)
)）查询语法，它支持全文搜索、关系运算等功能，搜索结果中关键字也会高亮显示：

![image.png](https://upload-images.jianshu.io/upload_images/1636821-7b1a5a8ba09bfa2c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- #### Dev Tools
[Console](https://www.elastic.co/guide/en/kibana/current/console-kibana.html)使您可以与Elasticsearch的REST API进行交互。

![image.png](https://upload-images.jianshu.io/upload_images/1636821-f08ac7e437e21426.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在数据处理管道中使用之前，您可以在Kibana [Grok Debugger](https://www.elastic.co/guide/en/kibana/current/xpack-grokdebugger.html)中构建和调试grok模式。

![image.png](https://upload-images.jianshu.io/upload_images/1636821-bd70a4c7904406da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- #### Visualize
[*Visualize*](https://www.elastic.co/guide/en/kibana/current/visualize.html)使您可以在Elasticsearch索引中创建数据的可视化。然后，您可以构建显示相关可视化的Dashboard。

- #### Dashboard
Kibana [Dashboard](https://www.elastic.co/guide/en/kibana/current/dashboard.html "仪表板")显示Visualize和搜索的集合。

## 3、进阶
目前方案下在Kibana中查看到的日志存在如下问题：
>1. 包含堆栈的多行日志被按行记录为多笔日志
>2. 日志内容全部记录在单个message字段上，没有按日志内容的含义提取到不同的字段进行存储，不利于查询

### 3.1 管理多行消息
>[Manage multiline messages]([https://www.elastic.co/guide/en/beats/filebeat/7.1/multiline-examples.html](https://www.elastic.co/guide/en/beats/filebeat/7.1/multiline-examples.html)
)
The files harvested by Filebeat may contain messages that span multiple lines of text. For example, multiline messages are common in files that contain Java stack traces. In order to correctly handle these multiline events, you need to configure `multiline` settings in the `filebeat.yml` file to specify which lines are part of a single event.
由filebeat收集的文件可能包含跨多行文本的消息。例如，多行消息在包含Java堆栈跟踪的文件中是常见的。为了正确处理这些多行事件，需要在filebeat.yml文件中配置多行设置，以指定哪些行是单个事件的一部分。

在`filebeat.yml`的`filebeat.inputs`配置节下添加如下配置：
```
filebeat.inputs:
  ...
  ### Multiline options
  multiline.pattern: ^\[
  multiline.negate: true
  multiline.match: after
```

### 3.2 使用Ingest Node解析数据
>[Parse data by using ingest node](https://www.elastic.co/guide/en/beats/filebeat/current/configuring-ingest-node.html)
When you use Elasticsearch for output, you can configure Filebeat to use [ingest node](https://www.elastic.co/guide/en/elasticsearch/reference/7.1/ingest.html) to pre-process documents before the actual indexing takes place in Elasticsearch. Ingest node is a convenient processing option when you want to do some extra processing on your data, but you do not require the full power of Logstash. For example, you can create an ingest node **pipeline** in Elasticsearch that consists of one processor that removes a field in a document followed by another processor that renames a field.
使用Elasticsearch进行输出时，可以将Filebeat配置为使用 [摄取节点](https://www.elastic.co/guide/en/elasticsearch/reference/7.1/ingest.html)在Elasticsearch中进行实际索引之前预处理文档。当您想对数据进行一些额外处理时，摄取节点是一个方便的处理选项，但您不需要Logstash的全部功能。例如，您可以在Elasticsearch中创建一个摄取节点管道，该管道由一个处理器组成，该处理器删除文档中的字段，然后是另一个重命名字段的处理器。

>[ingest node](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/ingest.html)
Use an ingest node to pre-process documents before the actual document indexing happens. The ingest node intercepts bulk and index requests, it applies transformations, and it then passes the documents back to the index or bulk APIs.
使用摄取节点在实际文档索引发生之前预处理文档。摄取节点拦截批量和索引请求，应用转换，然后将文档传递回索引或批量API。

![image.png](https://upload-images.jianshu.io/upload_images/1636821-c0a5a37c6f06963c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/1636821-2b7976a2ec96ccca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

要在索引前预处理文档，请定义指定一系列处理器的管道。每个处理器都以某种特定的方式转换文档

新增一个json文件，如`app-log-pipeline.json`：
```
{
  "description" : "app-log-pipeline",
  "processors" : [
    {
      ### 从文档中的单个文本字段message中提取结构化字段
      "grok" : {
        "field" : "message",
        "patterns" :["\\[%{TIMESTAMP_ISO8601:timestamp}\\] \\[%{WORD:topic}\\] \\[%{DATA:thread}\\] %{WORD:level} (?<msg>[\\s\\S]*)"]
      }
    }
  ]
}
```
*TODO: 有关grok processor语法的说法，详见...*

在Elasticsearch中添加管道：
```
### 如果elasticsearch启用了身份验证，则需要再加上参数`--user username:password`
curl -H 'Content-Type: application/json' -XPUT 'http://localhost:9200/_ingest/pipeline/app-log-pipeline' -d@/work/app-log-pipeline.json
```

修改配置文件：
```
output.elasticsearch:
  ...
  ### 指定管道
  pipeline: "app-log-pipeline"
```
调整后，在Kibana中看到的结果：

![image.png](https://upload-images.jianshu.io/upload_images/1636821-b6fe3e15a3d48033.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3.3 Grok

Grok是迄今为止使蹩脚的、非结构化的日志结构化和可查询的最好方式。Grok在解析 syslog logs、apache and other webserver logs、mysql logs等任意格式的文件上表现完美。

在LogStash的[Grok Filter](https://www.elastic.co/guide/en/logstash/7.2/plugins-filters-grok.html)、Elasticsearch的[Grok Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/grok-processor.html)中都使用到了Grok。

在上节中的`app-log-pipeline.json`中，我们使用了Grok  Processor将非结构化事件数据message解析为多个字段timestamp、topic、thread、level、msg：

```
      ### 从文档中的单个文本字段message中提取结构化字段
      "grok" : {
        "field" : "message",
        "patterns" :["\\[%{TIMESTAMP_ISO8601:timestamp}\\] \\[%{WORD:topic}\\] \\[%{DATA:thread}\\] %{WORD:level} (?<msg>[\\s\\S]*)"]
      }
```

Grok内置了120多种的正则表达式库，源代码地址：[https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns](https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns)。

>提示：在数据处理管道中使用之前，您可以在Kibana [Grok Debugger](https://www.elastic.co/guide/en/kibana/current/xpack-grokdebugger.html)中构建和调试grok模式。

### 3.4 Processors/Filter
除了Grok Processor/Filter，经常还需要用到其它的一些Processor/Filter，如geoid、date、user_agent、remove等，示例如下：
```
{
  "description" : "apache-log-pipeline",
  "processors" : [
    {
      ### 从文档中的单个文本字段message中提取结构化字段
      "grok" : {
        "field" : "message",
        "patterns" :["%{IPORHOST:clientip} %{USER:ident} %{USER:auth} \\[%{HTTPDATE:timestamp}\\] \"%{WORD:verb} %{DATA:request} HTTP/%{NUMBER:httpversion}\" %{NUMBER:response:int} (?:-|%{NUMBER:bytes:int}) %{QS:referrer} %{QS:useragent}"]
      },
      ### 添加关于IP地址的地理位置信息（基于MaxMind数据库中的数据）
      "geoip" : {
        "field" : "clientip",
        "target_field": "geo"
      },
      ### 解析字段中的日期，然后使用日期或时间戳作为文档的时间戳
      "date" : {
        "field" : "timestamp",
        "target_field" : "@timestamp",
        "formats" : ["dd/MMM/yyyy:HH:mm:ss Z"],
        "timezone" : "Asia/Shanghai"
      },
      ### 从浏览器发送Web请求中所述的user_agent字符串中提取细节信息
      "user_agent" : {
        "field" : "useragent"
      },
      ### 删除现有字段
      "remove": {
      	"field": "message"
      }
    }
  ]
}
```

#### Processors

*   [Append Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/append-processor.html)
*   [Bytes Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/bytes-processor.html)
*   [Convert Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/convert-processor.html)
*   [Date Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/date-processor.html)
*   [Date Index Name Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/date-index-name-processor.html)
*   [Dissect Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/dissect-processor.html)
*   [Dot Expander Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/dot-expand-processor.html)
*   [Drop Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/drop-processor.html)
*   [Fail Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/fail-processor.html)
*   [Foreach Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/foreach-processor.html)
*   [GeoIP Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/geoip-processor.html)
*   [Grok Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/grok-processor.html)
*   [Gsub Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/gsub-processor.html)
*   [HTML Strip Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/htmlstrip-processor.html)
*   [Join Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/join-processor.html)
*   [JSON Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/json-processor.html)
*   [KV Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/kv-processor.html)
*   [Pipeline Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/pipeline-processor.html)
*   [Remove Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/remove-processor.html)
*   [Rename Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/rename-processor.html)
*   [Script Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/script-processor.html)
*   [Set Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/set-processor.html)
*   [Set Security User Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/ingest-node-set-security-user-processor.html)
*   [Split Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/split-processor.html)
*   [Sort Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/sort-processor.html)
*   [Trim Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/trim-processor.html)
*   [Uppercase Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/uppercase-processor.html)
*   [URL Decode Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/urldecode-processor.html)
*   [User Agent processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/user-agent-processor.html)

#### Filter plugins
详见[Filter plugins](https://www.elastic.co/guide/en/logstash/7.2/filter-plugins.html)

## 4、引入消息队列作为缓冲
为了处理为生产中处理大量数据而构建的更复杂的管道，可能会在日志记录体系结构中添加其他组件，以实现弹性（Kafka，RabbitMQ，Redis）和安全性（nginx）：

![image.png](https://upload-images.jianshu.io/upload_images/1636821-cd5fdd978f52f208.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


引入消息队列中间件作为缓冲，beats采集到的数据先输出到消息队列中，再由logstash作为队列的消费者进行数据处理后，输出到elasticsearch。

出于说明的目的，这当然是简化图。完整的生产级体系结构将包含多个Elasticsearch节点，可能包含多个Logstash实例、归档机制、警报插件以及跨数据中心区域或部分的完全复制，以实现高可用性。

![image.png](https://upload-images.jianshu.io/upload_images/1636821-9930a5dc09096dcd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 4.1 redis方案
- #### 安装启动redis
linux: [https://redis.io/download](https://redis.io/download)
windows: [https://github.com/microsoftarchive/redis/releases](https://github.com/microsoftarchive/redis/releases)
- #### 配置`filebeat.yml`：
```
output.redis:
  hosts: ["127.0.0.1:6379"]
  password: "123456"
  key: "filebeat-redis"
  db: 0
  timeout: 5
```
- #### 配置`logstash.conf`的`input`和`output`节:

```
input {
  redis {
    host => "127.0.0.1"
    port => "6379"
    password => "123456"
    data_type => "list"
    db => 0
    batch_count => 1
    #type => "log"
    key => "filebeat-redis"
  }
}

# filter {
#  ...
# }

output {
  elasticsearch {
    hosts => "localhost:9200"
    manage_template => false
    index => "%{[@metadata][beat]}-redis-%{[@metadata][version]}-%{+YYYY.MM.dd}"
    pipeline => "app-log-pipeline"
  }
}
```


### 4.2 kafka方案

- #### 安装启动zookeeper
[https://zookeeper.apache.org/doc/current/zookeeperStarted.html](https://zookeeper.apache.org/doc/current/zookeeperStarted.html)

- #### 安装启动Kafka

[https://kafka.apache.org/quickstart](https://kafka.apache.org/quickstart)

- #### 配置`filebeat.yml`：
```
output.kafka:
  # initial brokers for reading cluster metadata
  hosts: ["localhost:9092"] #["kafka1:9092", "kafka2:9092", "kafka3:9092"]

  # message topic selection + partitioning
  topic: "filebeat-kafka" #'%{[fields.log_topic]}'
  partition.round_robin:
    reachable_only: false

  required_acks: 1
  compression: gzip
  max_message_bytes: 1000000
```

- #### 配置`logstash.conf`的`input`和`output`节:

```
input {
  kafka {
    bootstrap_servers => "localhost:9092"
    topics => "filebeat-kafka"
    group_id => "logstash"
  }
}

filter {  
  grok {    
      match => {
         "message" => "\[%{TIMESTAMP_ISO8601:timestamp}\] \[%{WORD:topic}\] \[%{DATA:thread}\] %{WORD:level} (?<msg>[\s\S]*)"
      }
  }
}

output {
  elasticsearch {
    hosts => "localhost:9200"
    manage_template => false
    index => "filebeat-kafka-%{+YYYY.MM.dd}"
    # pipeline => "app-log-pipeline"
  }
}
```

## 5、其它最佳实践

### 5.1 使用Security增加访问安全性

- #### 身份验证-安全登录

  &nbsp;&nbsp;&nbsp;&nbsp;要想保护流经 Elasticsearch、Kibana、Beats 和 Logstash 的数据，以免数据受到意外修改或被未经授权用户的访问，这是第一步。

[启用Elasticsearch安全功能](https://www.elastic.co/guide/en/elastic-stack-overview/7.2/get-started-enable-security.html)

```
xpack.security.enabled: true
### 需要额外设置此选项，否则启动时会报错
xpack.security.transport.ssl.enabled: true
```

[为内置用户创建密码](https://www.elastic.co/guide/en/elastic-stack-overview/7.2/get-started-built-in-users.html)

```
./bin/elasticsearch-setup-passwords interactive
```

[将内置用户添加到Kibana](https://www.elastic.co/guide/en/elastic-stack-overview/7.2/get-started-kibana-user.html)

如果您不介意在配置文件中显示密码，请取消注释并更新kibana.yml
```
elasticsearch.username: "kibana"
elasticsearch.password: "123456"
```

如果您不想将用户ID和密码放在kibana.yml文件中，请将它们存储在密钥库中
```
./bin/kibana-keystore create
./bin/kibana-keystore add elasticsearch.username
./bin/kibana-keystore add elasticsearch.password
```
此时，登录Kibana将出现登录页面，需要输入内置的超级用户权限帐号`elastic`进行登录。
![image.png](https://upload-images.jianshu.io/upload_images/1636821-3a47b7c8f8f6522a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- #### 授权-管理用户和角色

交付给用户使用时，一般会创建最小权限的用户，示例：
在`Management / Security`中创建用户，并分配分配`kibana_user `内置角色

- [在Logstash中添加用户信息](https://www.elastic.co/guide/en/elastic-stack-overview/7.2/get-started-logstash-user.html)

- 如果您不介意在配置文件中显示密码，请 在Logstash目录中的文件中添加以下内容user和password设置demo-metrics-pipeline.conf：

```
...
output {
  elasticsearch {
    ...
    user => "logstash_system" 
    password => "123456" 
  }
}
```

- 如果您不想将您的用户ID和密码放在配置文件中，请将它们存储在密钥库中。

```
set +o history
export LOGSTASH_KEYSTORE_PASS=mypassword 
set -o history
./bin/logstash-keystore create
./bin/logstash-keystore add ES_USER
./bin/logstash-keystore add ES_PWD
```

```
output {
  elasticsearch {
    ...
    user => "${ES_USER}"
    password => "${ES_PWD}"
  }
}
```

- 在filebeat中添加用户信息
- 如果您不介意在配置文件中显示密码，请 在Logstash目录中的文件中添加以下内容user和password设置demo-metrics-pipeline.conf：

```
...
output {
  elasticsearch {
    ...
    user => "logstash_system" 
    password => "123456" 
  }
}
```

- 如果您不想将您的密码放在配置文件中，请将它们存储在密钥库中。

```
filebeat keystore create
filebeat keystore add ES_PWD
```

```
output {
  elasticsearch {
    ...
    user => "logstash_system"
    password => "${ES_PWD}"
  }
}
```

>注意：
如果您直接从Metricbeat连接到Elasticsearch，则需要在Metricbeat配置文件中为Elasticsearch输出配置身份验证凭据。在 [入门弹性堆栈](https://www.elastic.co/guide/en/elastic-stack-get-started/7.2/get-started-elastic-stack.html)，但是，你配置Metricbeat发送数据到Logstash额外的解析，所以在Metricbeat不需要额外的设置。


参考：[Tutorial: Getting started with security](https://www.elastic.co/guide/en/elastic-stack-overview/7.2/security-getting-started.html)

以下功能为非基础许可证所有功能，详见：[订阅 · Elastic Stack 产品和支持 | Elastic](https://www.elastic.co/cn/subscriptions)
- 加密-防止嗅探、篡改和监听
- 分层安全-全方位保护，直到字段级
- 审核日志-记录何人何时做过何事
- 合规性-符合安全标准

### 5.2 使用Alerting告警

[Alerting告警](https://www.elastic.co/cn/products/stack/alerting)为非基础许可证所有功能，详见：[订阅 · Elastic Stack 产品和支持 | Elastic](https://www.elastic.co/cn/subscriptions)
TODO:待研究...[ELK借助*ElastAlert*实现故障提前感知预警功能](https://www.baidu.com/link?url=dlWj6gc5Pgl-3TGgbMlZdsch6Kp1bofta_VD6a7ml966nZdt03bMcp0MSzAMXIaz8kSAi3D6bmxxv34klmVZoa&wd=&eqid=b238721400080836000000065cf62e16)


### 5.3 使用MetricBeat监控服务器

### 5.4 自定义输出elasticsearch index
由于系统默认启用ilm，`output.elasticsearch.index`配置将被覆盖，因此可以使用如下配置自定义输出elasticsearch index：
```
### customize index name
setup.ilm.rollover_alias: "applog"
setup.ilm.pattern: "{now/d}-000001"
output.elasticsearch:
  ...
  # index: "applog-%{[agent.version]}-%{+yyyy.MM.dd}"
```
### 5.4 可视化分析Web访问日志

通过采集Nginx等Web服务器上的访问日志，使用图表和仪表盘进行可视化分析：

![FireShot Capture 006 - yh-Web访问日志分析 - Kibana - localhost.png](https://upload-images.jianshu.io/upload_images/1636821-e09e5f94a24c03de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 5.5 APM插件

### 5.7 Index Lifecycle Policies

### 5.6 使用Logstash限流

### 5.8 调整JVM内存大小

### 5.9 ELASTIC 运行状态监控

Beats平台设置最简单的架构包括一个或多个Beats，Elasticsearch和Kibana。这种架构易于入门，足以满足低流量网络的需求。它还使用最少量的服务器：运行Elasticsearch和Kibana的单台机器。Beats将事务直接插入Elasticsearch实例。

但是，如果要对数据执行其他处理或缓冲，则需要安装Logstash。
![image.png](https://upload-images.jianshu.io/upload_images/1636821-11cf462d0a63e42d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![image.png](https://upload-images.jianshu.io/upload_images/1636821-af1e91152a94891d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 6、Cluster方案

## 7、快速安装包

## 8、大数据扩展

# 三、...

# 四、阿里云日志服务

## 使用Producer Library采集日志

Aliyun LOG Java Producer 是一个易于使用且高度可配置的 Java 类库，专门为运行在大数据、高并发场景下的 Java 应用量身打造。

Github 项目地址以及更多详细说明请参见[Aliyun LOG Java Producer](https://github.com/aliyun/aliyun-log-producer)

1. 编写继承自`AppenderBase`的自定义Appender`AliyunLogAppender`
2. 配置`logback.xml`，增加新的Appender

```
<appender name="aliyun" class="com.example.log.AliyunLogAppender">
        <projectName>${project.name}</projectName>
        <logstore>${env}</logstore>
        <endpoint>exe-test.cn-beijing.log.aliyuncs.com</endpoint>
        <accessKey>${access.key}</accessKeyId>
        <accessSecret>${access.secret}</accessKey>
        <topic>${app.name}</topic>
        <timeFormat>yyyy-MM-dd HH:mm:ss</timeFormat>
        <timeZone>GMT+8</timeZone>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>WARN</level>
        </filter>
        <layout class="com.exe.core.log.FilterMessagePatternLayout">
            <pattern>%d [%thread] %-5level %logger{36} - %msg%n</pattern>
        </layout>
    </appender>
    ...
    <root level="debug">
        ...
        <appender-ref ref="aliyun"/>
    </root>
```

只需要这两步，就可以开始使用日志聚合和可视化查询：
![image.png](https://upload-images.jianshu.io/upload_images/1636821-71eb82ace75a2e04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 云产品采集-负载均衡7层访问日志
[https://help.aliyun.com/document_detail/66828.html?spm=a2c4g.11186623.6.646.42551519Egyw0M](https://help.aliyun.com/document_detail/66828.html?spm=a2c4g.11186623.6.646.42551519Egyw0M)

## 可视化分析、图表分析、仪表盘
[https://help.aliyun.com/document_detail/102530.html?spm=a2c4g.11186623.6.844.5485566aKsTXr6](https://help.aliyun.com/document_detail/102530.html?spm=a2c4g.11186623.6.844.5485566aKsTXr6)

## 聚类分析
