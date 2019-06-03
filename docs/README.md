标签（空格分隔）： 架构 日志 聚合

---

在此输入正文

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

4. 阿里云日志服务（云计算）

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

在上节中的`app-log-pipeline.json`中，我们使用了Grok将非结构化事件数据message解析为多个字段timestamp、topic、thread、level、msg：
```
      "grok" : {
        "field" : "message",
        "patterns" :["\\[%{TIMESTAMP_ISO8601:timestamp}\\] \\[%{WORD:topic}\\] \\[%{DATA:thread}\\] %{WORD:level} (?<msg>[\\s\\S]*)"]
      }
```
Grok是迄今为止使蹩脚的、非结构化的日志结构化和可查询的最好方式。Grok在解析 syslog logs、apache and other webserver logs、mysql logs等任意格式的文件上表现完美。

在LogStash的[Grok Filter](https://www.elastic.co/guide/en/logstash/7.2/plugins-filters-grok.html)（更多Filter详见[Filter Plugins](https://www.elastic.co/guide/en/logstash/7.2/filter-plugins.html)）、Elasticsearch的[Grok Processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/grok-processor.html)（更多Processor详见[Processors](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/ingest-processors.html)）中都使用到了Grok。

Grok内置了120多种的正则表达式库，源代码地址：[https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns](https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns)。

>提示：在数据处理管道中使用之前，您可以在Kibana [Grok Debugger](https://www.elastic.co/guide/en/kibana/current/xpack-grokdebugger.html)中构建和调试grok模式。

### 3.4 Processors
[Processors](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/ingest-processors.html)

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

## 4、引入消息队列
(背景说明）
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

### 5.2 使用Alerting告警

### 5.3 使用MetricBeat监控服务器

### 5.4 使用图表和仪表盘可视化分析Web访问日志

### 5.5 APM插件

### 5.7 Index Lifecycle Policies

### 5.6 使用Logstash限流
Beats平台设置最简单的架构包括一个或多个Beats，Elasticsearch和Kibana。这种架构易于入门，足以满足低流量网络的需求。它还使用最少量的服务器：运行Elasticsearch和Kibana的单台机器。Beats将事务直接插入Elasticsearch实例。

但是，如果要对数据执行其他处理或缓冲，则需要安装Logstash。
![image.png](https://upload-images.jianshu.io/upload_images/1636821-11cf462d0a63e42d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![image.png](https://upload-images.jianshu.io/upload_images/1636821-af1e91152a94891d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## Filebeat+Elasticsearch+Kibana方案

## Filebeat+Logstash+Elasticsearch+Kibana方案

## Cluster方案

# 三、...

# 四、阿里云日志服务