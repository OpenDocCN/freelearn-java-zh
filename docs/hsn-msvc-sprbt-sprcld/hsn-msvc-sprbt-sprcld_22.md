# 第十九章：集中日志记录的 EFK 堆栈

在本章中，我们将学习如何从微服务实例收集和存储日志记录，以及如何搜索和分析日志记录。正如我们在第一章《微服务简介》（参考*集中日志分析*部分）提到的，当每个微服务实例将日志记录写入其本地文件系统时，很难获得微服务系统景观的概览。我们需要一个组件，它可以从微服务的本地文件系统收集日志记录并将它们存储在中央数据库中进行分析、搜索和可视化。针对这一问题，一个流行的开源解决方案基于以下工具构建：

+   **Elasticsearch**，一个用于搜索和分析大数据集的分布式数据库

+   **Fluentd**，一个数据收集器，可以用来从各种来源收集日志记录，过滤和转换收集的信息，最后发送给各种消费者，例如 Elasticsearch

+   **Kibana**，一个 Elasticsearch 的图形用户界面，可以用来可视化搜索结果和对收集的日志记录进行分析

这些工具合称为**EFK 堆栈**，以每个工具的首字母命名。

本章将涵盖以下主题：

+   配置 Fluentd

+   在 Kubernetes 上部署 EFK 堆栈以供开发和测试使用

+   通过以下方式尝试 EFK 堆栈：

    +   分析收集的日志记录

    +   从微服务中发现日志记录并查找相关日志记录

    +   执行根本原因分析

# 技术要求

本书中描述的所有命令都已经在使用 macOS Mojave 的 MacBook Pro 上运行过，但应该很容易修改，以便它们可以在其他平台上运行，例如 Linux 或 Windows。

在本章中不需要安装任何新工具。

本章的源代码可以在本书的 GitHub 仓库中找到：[`github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud/tree/master/Chapter19`](https://github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud/tree/master/Chapter19)

为了能够运行本书中描述的命令，你需要将源代码下载到一个文件夹中，并设置一个环境变量`$BOOK_HOME`，它指向该文件夹。一些示例命令如下：

```java
export BOOK_HOME=~/Documents/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud
git clone https://github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud $BOOK_HOME
cd $BOOK_HOME/Chapter19
```

本章中的所有源代码示例都来自`$BOOK_HOME/Chapter19`的源代码，并使用 Kubernetes 1.15 进行了测试。

如果你想查看我们在这个章节中应用到源代码的变化，即查看我们做出的改变以便我们可以使用 EFK 栈进行集中式日志分析，你可以使用你最喜欢的 `diff` 工具，比较两个文件夹 `$BOOK_HOME/Chapter18` 和 `$BOOK_HOME/Chapter19`。

# 配置 Fluentd

在本节中，我们将学习如何配置 Fluentd 的基础知识。在那之前，让我们先了解一下 Fluentd 的背景以及它在高层次上是如何工作的。

# 介绍 Fluentd

历史上，处理日志记录最受欢迎的开源栈之一就是来自 Elastic 的 ELK 栈（[`www.elastic.co`](https://www.elastic.co)），基于 Elasticsearch、Logstash（用于日志收集和转换）和 Kibana。由于 Logstash 运行在 Java VM 上，它需要相对较多的内存。多年来，已经开发出许多比 Logstash 占用更少内存的开源替代品，其中之一就是 Fluentd([`www.fluentd.org`](https://www.fluentd.org))。

Fluentd 是由 **云原生计算基金会** （**CNCF**）管理的（[`www.cncf.io`](https://www.cncf.io)），也就是管理 Kubernetes 项目的同一个组织。因此，Fluentd 已经成为在 Kubernetes 中运行的开源日志收集器的自然选择。与 Elastic 和 Kibana 一起，它构成了 EFK 栈。

Fluentd 是用 C 和 Ruby 编写的混合语言，使用 C 处理性能关键部分，而在需要灵活性时使用 Ruby，例如，允许使用 Ruby 的 `gem install` 命令简单地安装第三方插件。

Fluentd 中的日志记录作为事件处理，并包括以下信息：

+   一个 `time` 字段，描述了日志记录创建的时间。

+   一个 `tag` 字段，用来标识它是什么类型的日志记录——这个标签由 Fluentd 的路由引擎用来确定日志记录应该如何被处理。

+   一个 **记录**，包含实际的日志信息，以 JSON 对象的形式存储。

Fluentd 配置文件用于告诉 Fluentd 如何收集、处理并最终将日志记录发送到各种目标，例如 Elasticsearch。一个配置文件由以下类型的核心元素组成：

+   `<source>`：源元素描述了 Fluentd 将会从哪里收集日志记录。例如，追踪由 Docker 容器写入的日志文件。源元素通常给日志记录打上标签，描述日志记录的类型。它可以用来自动标记来自运行在 Kubernetes 中的容器的日志记录。

+   `<filter>`：过滤器元素用于处理日志记录，例如，一个过滤器元素可以解析来自基于 Spring Boot 的微服务的日志记录，并将日志消息中的有趣部分提取到日志记录中的单独字段中。将信息提取到日志记录中的单独字段，使得可以通过 Elasticsearch 搜索这些信息。过滤器元素根据它们的标签选择要处理的日志记录。

+   `<match>`：输出元素用于执行两个主要任务：

    +   将处理后的日志记录发送到目标，如 Elasticsearch。

    +   路由是决定如何处理日志记录的方法。路由规则可以重写标签，并将日志记录重新发射到 Fluentd 路由引擎以进行进一步处理。路由规则被表达为 `<match>` 元素内的嵌入 `<rule>` 元素。输出元素决定根据日志记录的标签处理哪些日志记录，与过滤器的方式相同。

Fluentd 自带了许多内置和外部的第三方插件，这些插件被源、过滤器和输出元素所使用。在我们下一节查看配置文件时，将看到其中的一些插件。关于可用插件的更多信息，请参阅 Fluentd 的文档，该文档可访问于 [`docs.fluentd.org`](https://docs.fluentd.org)。

在介绍了 Fluentd 之后，我们准备了解 Fluentd 应该如何配置以处理我们微服务中的日志记录。

# 配置 Fluentd

Fluentd 的配置基于 GitHub 上的 Fluentd 项目的配置文件，`fluentd-kubernetes-daemonset`。该项目包含 Fluentd 配置文件，说明如何从在 Kubernetes 中运行的容器收集日志记录，以及一旦它们被处理，如何将它们发送到 Elasticsearch。我们可以不进行任何更改地重用此配置，这将极大地简化我们自己的配置。Fluentd 配置文件可以在 [`github.com/fluent/fluentd-kubernetes-daemonset/tree/master/docker-image/v1.4/debian-elasticsearch/conf`](https://github.com/fluent/fluentd-kubernetes-daemonset/tree/master/docker-image/v1.4/debian-elasticsearch/conf) 找到。

提供这种功能的配置文件有 `kubernetes.conf` 和 `fluent.conf`。`kubernetes.conf` 配置文件包含以下信息：

+   源元素用于监控容器日志文件以及运行在 Kubernetes 之外的过程的日志文件，例如`kubelet`和 Docker 守护进程。源元素还将为 Kubernetes 中的日志记录打上完整的日志文件名，并用`/`替换`.'，并加上前缀`kubernetes`。由于标签基于完整的文件名，其中包含命名空间、Pod 和容器的名称等信息，因此标签对于通过匹配标签找到感兴趣的日志记录非常有用。例如，`product-composite`微服务的标签可能类似于`kubernetes.var.log.containers.product-composite-7...s_hands-on_comp-e...b.log`，而同一 Pod 中相应的`istio-proxy`的标签可能类似于`kubernetes.var.log.containers.product-composite-7...s_hands-on_istio-proxy-1...3.log`。

+   一个过滤器元素，用于丰富来自 Kubernetes 内部运行的容器以及包含容器名称和它们运行的命名空间等信息的 Kubernetes 特定字段的日志记录。

主配置文件`fluent.conf`包含以下信息：

+   `@include`语句用于其他配置文件，例如我们之前描述的`kubernetes.conf`文件。它还包括放置在特定文件夹中的自定义配置文件，使我们能够不进行任何更改就重用这些配置文件，并只处理与自身日志记录相关的处理。我们只需将自定义配置文件放置在`fluent.conf`文件指定的文件夹中。

+   一个输出元素，用于将日志记录发送到 Elasticsearch。

正如我们在*部署 Fluentd*部分所描述的，这两个配置文件将被打包到我们将为 Fluentd 构建的 Docker 镜像中。

在我们自己的配置文件中需要覆盖的是以下内容：

+   检测并解析我们微服务中的 Spring Boot 格式的日志记录。

+   处理多行堆栈跟踪。例如，堆栈跟踪是使用多行写入日志文件的。这使得 Fluentd 难以将堆栈跟踪作为单个日志记录处理。

+   将`istio-proxy`侧边的日志记录与在同一 Pod 中运行的微服务生成的日志记录分开。由`istio-proxy`生成的日志记录不遵循我们基于 Spring Boot 的微服务生成的日志模式。因此，它们必须分开处理，以便 Fluentd 不要尝试将它们解析为 Spring Boot 格式的日志记录。

为了实现这一目标，配置很大程度上是基于使用`rewrite_tag_filter`插件。这个插件可以用于根据改变标签名称的概念来路由日志记录，然后将日志记录重新发射到 Fluentd 路由引擎。

此处理总结如下 UML 活动图：

![](img/4aa9c779-4a07-422d-af6c-00539062d30a.png)

从高层次来看，配置文件的设计如下所示：

+   来自 Istio 的所有日志记录的标签，包括`istio-proxy`，都加上`istio`前缀，以便将它们与基于 Spring Boot 的日志记录区分开来。

+   所有来自`hands-on`命名空间的日志记录（除了来自`istio-proxy`的日志记录）的标签都加上`spring-boot`前缀。

+   来自 Spring Boot 的日志记录检查是否有多行堆栈跟踪。如果日志记录是多行堆栈跟踪的一部分，它将使用第三方`detect-exceptions`插件来重新创建堆栈跟踪。否则，它将使用正则表达式提取感兴趣的信息。关于这个第三方插件的详细信息，请参见*部署 Fluentd*部分。

`fluentd-hands-on.conf`配置文件紧密遵循这个活动图。该配置文件放在一个 Kubernetes 配置映射中（参见`kubernetes/efk/fluentd-hands-on-configmap.yml`）。让我们一步一步地讲解，如下所述：

1.  首先是配置映射和配置文件 filename 的定义，`fluentd-hands-on.conf`。它看起来像这样：

```java
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-hands-on-config
  namespace: kube-system
data:
  fluentd-hands-on.conf: |
```

从前面的源代码中，我们了解到`data`元素将包含 Fluentd 的配置。它从文件名开始，并使用竖线`|`标记 Fluentd 嵌入式配置文件的开始。

1.  第一个`<match>`元素匹配来自 Istio 的日志记录，即以`kubernetes`为前缀并包含作为其命名空间或容器名称一部分的`istio`的标签。它看起来像这样：

```java
    <match kubernetes.**istio**>
      @type rewrite_tag_filter
      <rule>
        key log
        pattern ^(.*)$
        tag istio.${tag}
      </rule>
    </match>
```

让我们更详细地解释前面的源代码：

+   +   `<match>`元素匹配任何符合`kubernetes.**istio**`模式的标签，即，它以`kubernetes`开始，然后在标签名中包含单词`istio`。`istio`可以来自命名空间名称或容器名称；两者都是标签的一部分。

    +   `<match>`元素只包含一个`<rule>`元素，它为标签加上`istio`前缀。`${tag}`变量持有当前标签的值。

    +   由于这是`<match>`元素中唯一的`<rule>`元素，它被配置为如此匹配所有日志记录：

        +   由于所有来自 Kubernetes 的日志记录都有一个`log`字段，所以`key`字段被设置为`log`，即规则在日志记录中查找一个`log`字段。

        +   为了匹配`log`字段中的任何字符串，`pattern`字段被设置为`^(.*)$`正则表达式。`^`标志着一个字符串的开始，而`$`标志着一个字符串的结束。`(.*)`匹配任何数量的字符，除了换行符。

        +   日志记录被重新发送到 Fluentd 路由引擎。由于配置文件中没有其他元素与以`istio`开头的标签匹配，它们将被直接发送到我们之前描述的`fluent.conf`文件中定义的 Elasticsearch 输出元素。

1.  第二个`<match>`元素匹配来自`hands-on`命名空间的全部日志记录，也就是说，是我们微服务发出的日志记录。它看起来像这样：

```java
    <match kubernetes.**hands-on**>
      @type rewrite_tag_filter
      <rule>
        key log
        pattern ^(.*)$
        tag spring-boot.${tag}
      </rule>
    </match>
```

从前的源代码我们可以看出：

+   +   我们微服务发出的日志记录使用 Spring Boot 定义的日志消息格式化规则，因此它们的标签前缀为`spring-boot`。然后，它们被重新发出以进行进一步处理。

    +   `<match>`元素的配置方式与我们在前面查看的`<match kubernetes.**istio**>`元素相同。

1.  第三个`<match>`元素匹配`spring-boot`日志记录，并确定它们是普通的 Spring Boot 日志记录还是多行堆栈跟踪的一部分。它看起来像这样：

```java
    <match spring-boot.**>
      @type rewrite_tag_filter
      <rule>
        key log
        pattern /^\d{4}-\d{2}-\d{2}\s\d{2}:\d{2}:\d{2}\.\d{3}.*/
        tag parse.${tag}
      </rule>
      <rule>
        key log
        pattern /^.*/
        tag check.exception.${tag}
      </rule>
    </match>
```

如前所见的源代码，这是通过使用两个`<rule>`元素来确定的：

+   +   第一个使用正则表达式检查日志元素中的`log`字段是否以时间戳开头。

    +   如果`log`字段以时间戳开头，则将日志记录视为普通的 Spring Boot 日志记录，并在其标签前加上`parse`前缀。

    +   否则，第二个`<rule>`元素将匹配，并将日志记录作为多行日志记录处理。其标签前缀为`check.exception`。

    +   无论如何，在这种情况下，日志记录都会被重新发出，其标签将在此过程后以`check.exception.spring-boot.kubernetes`或`parse.spring-boot.kubernetes`开头。

1.  在第四个`<match>`元素中，所选的日志记录具有以`check.exception.spring-boot`开头的标签，即，是多行堆栈跟踪的一部分的日志记录。它看起来像这样：

```java
    <match check.exception.spring-boot.**>
      @type detect_exceptions
      languages java
      remove_tag_prefix check
      message log
      multiline_flush_interval 5
    </match>
```

`detect_exceptions`插件的源代码像这样工作：

+   +   `detect_exceptions`插件用于将多个单行日志记录组合成一个包含完整堆栈跟踪的日志记录。

    +   在将多行日志记录重新输入路由引擎之前，将标签中的`check`前缀移除，以防止日志记录的处理循环无限进行。

1.  最后，配置文件由一个过滤器元素组成，该元素使用正则表达式解析 Spring Boot 日志消息，提取感兴趣的信息。它看起来像这样：

```java
    <filter parse.spring-boot.**>
      @type parser
      key_name log
      time_key time
      time_format %Y-%m-%d %H:%M:%S.%N
      reserve_data true
      format /^(?<time>\d{4}-\d{2}-
      \d{2}\s\d{2}:\d{2}:\d{2}\.\d{3})\s+
      (?<spring.level>[^\s]+)\s+
      (\[(?<spring.service>[^,]*),(?<spring.trace>[^,]*),(?
      <spring.span>[^,]*),[^\]]*\])\s+
      (?<spring.pid>\d+)\s+---\s+\[\s*(?<spring.thread>[^\]]+)\]\s+
      (?<spring.class>[^\s]+)\s*:\s+
      (?<log>.*)$/
    </filter>
```

让我们更详细地解释前面的源代码：

+   +   请注意，过滤器元素不会重新发出日志记录；相反，它们只是将它们传递给配置文件中与日志记录的标签匹配的下一个元素。

    +   从存储在日志记录中的`log`字段的 Spring Boot 日志消息中提取以下字段：

        +   `<time>`：创建日志记录的时间戳

        +   `<spring.level>`：日志记录的日志级别，例如，`FATAL`、`ERROR`、`WARN`、`INFO`、`DEBUG`或`TRACE`

        +   `<spring.service>`：微服务名称

        +   `<spring.trace>`：用于执行分布式跟踪的跟踪 ID

        +   `<spring.span>`：跨度 ID，即这个微服务执行的分布式处理的部分的 ID

        +   `<spring.pid>`：进程 ID

        +   `<spring.thread>`：线程 ID

        +   `<spring.class>`：Java 类的名称

        +   `<log>`：实际的日志消息

使用`spring.application.name`属性指定基于 Spring Boot 的微服务名称。此属性已添加到配置存储库中的每个微服务特定属性文件中，在`config-repo`文件夹中。

准确地编写正则表达式可以说是一项挑战。幸运的是，有多个网站可以提供帮助。当涉及到与 Fluentd 一起使用正则表达式时，我推荐使用以下网站：[`fluentular.herokuapp.com/`](https://fluentular.herokuapp.com/)。

既然你已经了解了 Fluentd 如何工作以及配置文件是如何构建的，我们就可以部署 EKF 堆栈了。

# 在 Kubernetes 上部署 EFK 堆栈

在 Kubernetes 上部署 EFK 堆栈的方式将与我们部署自己的微服务的方式相同：使用 Kubernetes 定义文件为部署、服务和配置映射等对象。

EFK 堆栈的部署分为两部分：

+   我们部署 Elasticsearch 和 Kibana 的一部分

+   我们部署 Fluentd 的一部分

但首先，我们需要构建和部署我们自己的微服务。

# 构建和部署我们的微服务

使用`test-em-all.bash`测试脚本构建、部署并验证部署的方式与第十八章中的*使用服务网格提高可观测性和管理*，*运行创建服务网格的命令*部分相同。开始运行以下命令：

1.  首先，使用以下命令从源代码构建 Docker 镜像：

```java
cd $BOOK_HOME/Chapter19
eval $(minikube docker-env)
./gradlew build && docker-compose build
```

1.  重新创建命名空间`hands-on`，并将其设置为默认命名空间：

```java
kubectl delete namespace hands-on
kubectl create namespace hands-on
kubectl config set-context $(kubectl config current-context) --namespace=hands-on 
```

1.  通过运行以下命令执行部署：

```java
./kubernetes/scripts/deploy-dev-env.bash
```

1.  如果 Minikube 隧道尚未运行，请启动（如果需要，请参阅第十八章*使用服务网格提高可观测性和管理*，*设置对 Istio 服务的访问*部分）：

```java
minikube tunnel
```

请记住，此命令要求您的用户具有`sudo`权限，并且在启动和关闭时输入您的密码。在命令要求输入密码之前需要几秒钟，所以很容易错过！

1.  使用以下命令运行正常测试以验证部署：

```java
./test-em-all.bash
```

期望输出与前几章看到的内容类似：

![](img/042de643-01de-4776-8b29-ab05be9fe519.png)

1.  您还可以通过运行以下命令手动测试 API：

```java
ACCESS_TOKEN=$(curl -k https://writer:secret@minikube.me/oauth/token -d grant_type=password -d username=magnus -d password=password -s | jq .access_token -r)

curl -ks https://minikube.me/product-composite/2 -H "Authorization: Bearer $ACCESS_TOKEN" | jq .productId
```

期望在响应中收到请求的产品 ID，`2`。

微服务部署完成后，我们可以继续部署 Elasticsearch 和 Kibana！

# 部署 Elasticsearch 和 Kibana

我们将 Elasticsearch 和 Kibana 部署到其自己的命名空间 `logging`。Elasticsearch 和 Kibana 将使用 Kubernetes 部署对象部署用于开发和测试。这将通过单个 pod 和 Kubernetes 节点端口服务完成。服务将内部在 Kubernetes 集群中暴露 Elasticsearch 和 Kibana 的标准端口，即 Elasticsearch 的端口 `9200` 和 Kibana 的端口 `5601`。多亏了 `minikube tunnel` 命令，我们将能够使用以下 URL 本地访问这些服务：

+   `elasticsearch.logging.svc.cluster.local:9200` 对于 Elasticsearch。

+   `kibana.logging.svc.cluster.local:5601` 对于 Kibana。

要查看在 Kubernetes 生产环境中推荐部署，请参阅 [`www.elastic.co/elasticsearch-kubernetes`](https://www.elastic.co/elasticsearch-kubernetes)。

我们将使用在本章写作时可用的版本：

+   Elasticsearch 版本 7.3.0

+   Kibana 版本 7.3.0

在执行部署之前，让我们看看定义文件中最有趣的部分。

# 定义文件的逐步讲解

**Elasticsearch** 的定义文件 `kubernetes/efk/elasticsearch.yml` 包含了一个标准的 Kubernetes 部署和服务对象，我们在之前见过多次，例如在 第十五章 *Kubernetes 简介* 的 *尝试样本部署* 部分。正如我们之前解释的，定义文件最有趣的部分如下：

```java
apiVersion: extensions/v1beta1
kind: Deployment
...
      containers:
      - name: elasticsearch
        image: docker.elastic.co/elasticsearch/elasticsearch-oss:7.3.0
        resources:
          limits:
            cpu: 500m
            memory: 2Gi
          requests:
            cpu: 500m
            memory: 2Gi
```

让我们详细解释前面的源代码：

+   我们使用来自 Elastic 的官方 Docker 镜像，可在 `docker.elastic.co` 获得，以及只包含开源组件的包。这通过在 Docker 镜像名称中使用 `-oss` 后缀来保证，即 `elasticsearch-oss`。版本设置为 `7.3.0`。

+   Elasticsearch 容器被允许分配相对较多的内存 - 2 GB，以能够良好地执行查询。内存越多，性能越好。

**Kibana** 的定义文件 `kubernetes/efk/kibana.yml` 也包含了一个标准的 Kubernetes 部署和服务对象。定义文件中最有趣的部分如下：

```java
apiVersion: extensions/v1beta1
kind: Deployment
...
      containers:
      - name: kibana
        image: docker.elastic.co/kibana/kibana-oss:7.3.0
        env:
        - name: ELASTICSEARCH_URL
          value: http://elasticsearch:9200
```

让我们详细解释前面的源代码：

+   对于 Kibana，我们还使用来自 Elastic 的官方 Docker 镜像，可在 `docker.elastic.co` 获得，以及只包含开源组件的包 `kibana-oss`。版本设置为 `7.3.0`。

+   为了将 Kibana 与 Elasticsearch pod 连接，定义了一个环境变量 `ELASTICSEARCH_URL`，以指定 Elasticsearch 服务的地址，`http://elasticsearch:9200`。

有了这些洞察，我们准备执行 Elasticsearch 和 Kibana 的部署。

# 运行部署命令

通过执行以下步骤部署 Elasticsearch 和 Kibana：

1.  使用以下命令为 Elasticsearch 和 Kibana 创建一个命名空间：

```java
kubectl create namespace logging
```

1.  为了使部署步骤运行得更快，使用以下命令预取 Elasticsearch 和 Kibana 的 Docker 镜像：

```java
eval $(minikube docker-env)
docker pull docker.elastic.co/elasticsearch/elasticsearch-oss:7.3.0
docker pull docker.elastic.co/kibana/kibana-oss:7.3.0
```

1.  使用以下命令部署 Elasticsearch 并等待其 pod 准备好：

```java
kubectl apply -f kubernetes/efk/elasticsearch.yml -n logging
kubectl wait --timeout=120s --for=condition=Ready pod -n logging --all 
```

1.  使用以下命令验证 Elasticsearch 是否正在运行：

```java
curl http://elasticsearch.logging.svc.cluster.local:9200 -s | jq -r .tagline
```

期待响应为`You Know, for Search`。

根据您的硬件，您可能需要等待一两分钟，直到 Elasticsearch 回应此消息。

1.  使用以下命令部署 Kibana 并等待其 pod 准备好：

```java
kubectl apply -f kubernetes/efk/kibana.yml -n logging
kubectl wait --timeout=120s --for=condition=Ready pod -n logging --all
```

1.  使用以下命令验证 Kibana 是否正在运行：

```java
curl -o /dev/null -s -L -w "%{http_code}\n" http://kibana.logging.svc.cluster.local:5601
```

期待响应为 `200`。

在部署 Elasticsearch 和 Kibana 之后，我们可以开始部署 Fluentd。

# 部署 Fluentd

与部署 Elasticsearch 和 Kibana 相比，部署 Fluentd 稍微复杂一些。为了部署 Fluentd，我们将使用 Fluentd 项目在 Docker Hub 上发布的 Docker 镜像，`fluent/fluentd-kubernetes-daemonset`，并从 GitHub 上的 Fluentd 项目中采样 Kubernetes 定义文件，`fluentd-kubernetes-daemonset`。它位于 [`github.com/fluent/fluentd-kubernetes-daemonset`](https://github.com/fluent/fluentd-kubernetes-daemonset)。正如项目名称所暗示的，Fluentd 将作为 daemon set 部署，在 Kubernetes 集群中的每个节点运行一个 pod。每个 Fluentd pod 负责收集运行在同一节点上的进程和容器的日志输出。由于我们使用的是 Minikube，即单节点集群，所以我们只有一个 Fluentd pod。

为了处理包含异常堆栈跟踪的多行日志记录，我们将使用 Google 提供的一个第三方 Fluentd 插件，`fluent-plugin-detect-exceptions`，该插件可在 [`github.com/GoogleCloudPlatform/fluent-plugin-detect-exceptions`](https://github.com/GoogleCloudPlatform/fluent-plugin-detect-exceptions) 获得。为了能够使用这个插件，我们将构建我们自己的 Docker 镜像，其中将安装 `fluent-plugin-detect-exceptions` 插件。将使用 Fluentd 的 Docker 镜像 `fluentd-kubernetes-daemonset` 作为基础镜像。

我们将使用在本章编写时可用的版本：

+   Fluentd 版本 1.4.2

+   fluent-plugin-detect-exceptions 版本 0.0.12

在执行部署之前，让我们看看定义文件的最有趣的部分。

# 从微服务中发现日志记录

在本节中，我们将学习如何利用集中日志的一个主要功能，那就是从我们的微服务中找到日志记录。我们还将学习如何在日志记录中使用 trace ID 来查找属于同一进程的其他微服务的日志记录，例如 API 的请求。

让我们先通过 API 创建一些我们可以用 Kibana 查找的日志记录。我们将使用 API 创建一个具有唯一产品 ID 的产品，然后检索有关产品的信息。之后，我们可以尝试找到在检索产品信息时创建的日志记录。

与上一章相比，微服务中的日志记录创建稍有更新，以便产品组合和三个核心微服务（`product`、`recommendation`和`review`）在处理 get 请求时都设置日志级别为`INFO`。让我们来看看为每个微服务添加的源代码：

+   产品组合微服务日志创建：

```java
LOG.info("Will get composite product info for product.id={}", productId);
```

+   产品微服务日志创建：

```java
LOG.info("Will get product info for id={}", productId);
```

+   推荐微服务日志创建：

```java
LOG.info("Will get recommendations for product with id={}", productId)
```

+   评论微服务日志创建：

```java
LOG.info("Will get reviews for product with id={}", productId);
```

有关详细信息，请参阅`microservices`文件夹中的源代码。

执行以下步骤使用 API 创建日志记录，然后使用 Kibana 查找日志记录：

1.  使用以下命令获取访问令牌：

```java
ACCESS_TOKEN=$(curl -k https://writer:secret@minikube.me/oauth/token -d grant_type=password -d username=magnus -d password=password -s | jq .access_token -r)
```

1.  如本节介绍中所述，我们将首先通过 API 创建具有唯一产品 ID 的产品。为`"productId" :1234`创建一个简约的产品（不包含推荐和评论）通过执行以下命令：

```java
curl -X POST -k https://minikube.me/product-composite \
 -H "Content-Type: application/json" \
 -H "Authorization: Bearer $ACCESS_TOKEN" \
 --data '{"productId":1234,"name":"product name 1234","weight":1234}'
```

1.  使用以下命令读取产品：

```java
curl -H "Authorization: Bearer $ACCESS_TOKEN" -k 'https://minikube.me/product-composite/1234'
```

期待如下响应：

![](img/258fd470-f7ae-432b-933e-0098656e7199.png)

希望这些 API 调用创建了一些日志记录。让我们跳到 Kibana 去看看！

1.  在 Kibana 网页上，点击左侧的`Discover`菜单。你会看到类似以下内容：

![](img/75f6dd32-2743-454f-a301-007444bb2cfd.png)

在左上角，我们可以看到 Kibaba 找到了 326,642 条日志记录。时间选择器显示它们来自过去 7 天。在直方图中，我们可以看到日志记录随时间如何分布。之后是一个表，显示查询找到的最新日志事件。

1.  如果您想更改时间间隔，可以使用时间选择器。点击其日历图标调整时间间隔。

1.  为了更好地查看日志记录中的内容，请将日志记录中的某些字段添加到直方图下的表中。从左侧的可用字段列表中选择字段。滚动到底部直到找到字段。将光标悬停在字段上，会出现一个添加按钮；点击它将字段作为列添加到表中。选择以下字段，按顺序：

    1.  spring.level，日志级别

    1.  kubernetes.container_name，容器名称

    1.  spring.trace，用于分布式跟踪的跟踪 ID

+   1.  log，实际日志消息。网页应看起来与以下类似：

![](img/e42ce2ba-2d4c-4436-95d7-cde3621c572f.png)

表格现在包含了有关日志记录的信息！

1.  要查找来自`GET` API 调用的日志记录，我们可以请 Kibana 查找日志字段包含 product.id=1234 的日志记录。这匹配了前面显示的产品组合微服务的日志输出。这可以通过在搜索字段中输入`log:"product.id=1234"`并点击更新按钮（这个按钮也可以标记为刷新）来完成。预期会找到一条日志记录：

![](img/c5370730-05a8-4dc0-a414-3020057d3bb6.png)

1.  验证时间戳是否来自您调用`GET` API 的时间，并验证创建日志记录的容器名称是`comp`，即验证日志记录是由产品组合微服务发送的。

1.  现在，我们想要查看其他参与返回产品 ID 1234 信息过程的微服务的相关日志记录，即查找具有与找到的日志记录相同的跟踪 ID 的日志记录。为此，将光标悬停在日志记录的`spring.trace`字段上。字段右侧将显示两个小放大镜图标，一个带加号，一个带减号。点击带有加号的放大镜图标以根据跟踪 ID 进行过滤。

1.  清空搜索字段，以便唯一的搜索条件是跟踪字段的过滤。然后，点击更新按钮以查看结果。预期会有类似于下面的响应：

![](img/909125b4-7868-4ed9-b33c-3a5197408304.png)

我们可以看到很多详细的 debug 和 trace 消息，让视图变得杂乱，让我们去掉它们！

1.  将光标悬停在 TRACE 值上，然后点击带有减号的放大镜图标以过滤出日志级别设置为 TRACE 的日志记录。

1.  重复上述步骤以处理 DEBUG 日志记录。

1.  现在我们应该能够看到四个预期的日志记录，每个记录都涉及查找产品信息的产品 ID 1234 的每个微服务：

![](img/437104c3-c1c1-463a-a9d9-9fe2e57f6461.png)

另外，请注意应用的过滤器包括跟踪 ID，但不包括日志级别设置为 DEBUG 或 TRACE 的日志记录。

既然我们知道如何查找预期的日志记录，我们就可以进行下一步了。这将是要学习如何查找意外的日志记录，即错误消息，以及如何进行根本原因分析，即找到这些错误消息的原因。

# 定义文件的概览

用于构建 Docker 镜像的 Dockerfile，`kubernetes/efk/Dockerfile`，如下所示：

```java
FROM fluent/fluentd-kubernetes-daemonset:v1.4.2-debian-elasticsearch-1.1

RUN gem install fluent-plugin-detect-exceptions -v 0.0.12 \
 && gem sources --clear-all \
 && rm -rf /var/lib/apt/lists/* \
           /home/fluent/.gem/ruby/2.3.0/cache/*.gem
```

让我们详细解释前面的源代码：

+   基础镜像使用的是 Fluentd 的 Docker 镜像，`fluentd-kubernetes-daemonset`。标签`v1.4.2-debian-elasticsearch-1.1`指定了应使用版本 v1.4.2，并且包含发送日志记录到 Elasticsearch 的内置支持的包。基础 Docker 镜像包含了在*配置 Fluentd*部分提到的 Fluentd 配置文件。

+   使用 Ruby 的包管理器`gem`安装 Google 插件`fluent-plugin-detect-exceptions`。

守护进程集的定义文件`kubernetes/efk/fluentd-ds.yml`基于`fluentd-kubernetes-daemonset`项目中的一个示例定义文件，该文件可在此处找到[`github.com/fluent/fluentd-kubernetes-daemonset/blob/master/fluentd-daemonset-elasticsearch.yaml`](https://github.com/fluent/fluentd-kubernetes-daemonset/blob/master/fluentd-daemonset-elasticsearch.yaml)。这个文件有点复杂，所以让我们分别查看最有趣的部分：

1.  首先，以下是守护进程集的声明：

```java
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
```

让我们详细解释一下前面的源代码：

+   +   `kind`键指定了这是一个守护进程集。

    +   `namespace`键指定守护进程集应创建于`kube-system`命名空间中，而不是部署 Elasticsearch 和 Kibana 的`logging`命名空间。

1.  下一部分指定了由守护进程集创建的 Pod 模板。最有趣的部分如下：

```java
spec:
  template:
    spec:
      containers:
      - name: fluentd
        image: hands-on/fluentd:v1
        env:
          - name: FLUENT_ELASTICSEARCH_HOST
            value: "elasticsearch.logging"
          - name: FLUENT_ELASTICSEARCH_PORT
            value: "9200"
```

让我们详细解释一下前面的源代码：

+   +   用于 Pod 的 Docker 镜像为`hands-on/fluentd:v1`。我们在之前描述的 Dockerfile 中走过定义文件后，将构建这个 Docker 镜像。

    +   Docker 镜像支持许多环境变量，用于自定义它。其中最重要的两个如下：

        +   `FLUENT_ELASTICSEARCH_HOST`，指定 Elasticsearch 服务的主机名，即`elasticsearch.logging`

        +   `FLUENT_ELASTICSEARCH_PORT`，指定与 Elasticsearch 通信的端口，即`9200`

由于 Fluentd Pod 在 Elasticsearch 之外的命名空间中运行，不能使用其短名称（即`elasticsearch`）来指定主机名。相反，DNS 名称的命名空间部分也必须指定，即`elasticsearch.logging`。作为替代方案，可以使用**完全合格域名**（**FQDN**），`elasticsearch.logging.svc.cluster.local`。但由于 DNS 名称的最后一部分，`svc.cluster.local`，在 Kubernetes 集群内的所有 DNS 名称中是共享的，因此不需要指定。

1.  最后，有一系列卷，即文件系统，被映射到 Pod 中，如下所示：

```java
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
        - name: journal
          mountPath: /var/log/journal
          readOnly: true
        - name: fluentd-extra-config
          mountPath: /fluentd/etc/conf.d
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
      - name: journal
        hostPath:
          path: /run/log/journal
      - name: fluentd-extra-config
        configMap:
          name: "fluentd-hands-on-config"
```

让我们详细解释一下前面的源代码：

+   +   宿主机（即节点）上的三个文件夹被映射到 Fluentd Pod 中。这些文件夹包含 Fluentd 将跟踪和收集日志记录的日志文件。这些文件夹是：`/var/log`、`/var/lib/docker/containers`和`/run/log/journal`。

    +   我们自己的配置文件，指定 Fluentd 如何处理来自我们微服务的日志记录，通过一个名为`fluentd-hands-on-config`的 config map 映射到`/fluentd/etc/conf.d`文件夹。之前使用的 Fluentd 的 Docker 镜像`fluentd-kubernetes-daemonset`，配置 Fluentd 包括在`/fluentd/etc/conf.d`文件夹中找到的任何配置文件。有关详细信息，请参见*配置 Fluentd*部分。

关于 daemon set 定义文件的完整源代码，请参阅`kubernetes/efk/fluentd-ds.yml`文件。

既然我们已经涵盖了所有内容，我们就可以准备部署 Fluentd 了。

# 运行部署命令

要部署 Fluentd，我们必须构建 Docker 镜像，创建 config map，最后部署 daemon set。运行以下命令以执行这些步骤：

1.  使用以下命令构建 Docker 镜像并标记为`hands-on/fluentd:v1`：

```java
eval $(minikube docker-env)
docker build -f kubernetes/efk/Dockerfile -t hands-on/fluentd:v1 kubernetes/efk/
```

1.  使用以下命令创建 config map，部署 Fluentd 的 daemon set，并等待 pod 就绪：

```java
kubectl apply -f kubernetes/efk/fluentd-hands-on-configmap.yml 
kubectl apply -f kubernetes/efk/fluentd-ds.yml
kubectl wait --timeout=120s --for=condition=Ready pod -l app=fluentd -n kube-system
```

1.  使用以下命令验证 Fluentd pod 是否正常：

```java
kubectl logs -n kube-system $(kubectl get pod -l app=fluentd -n kube-system -o jsonpath={.items..metadata.name}) | grep "fluentd worker is now running worker"
```

期望的响应是`2019-08-16 15:11:33 +0000 [info]: #0 fluentd worker is now running worker=0`。

1.  Fluentd 将开始从 Minkube 实例中的各种进程和容器收集大量的日志记录。大约一分钟后，您可以使用以下命令询问 Elasticsearch 已经收集了多少日志记录：

```java
curl http://elasticsearch.logging.svc.cluster.local:9200/_all/_count
```

该命令首次执行时可能会有点慢，但应该返回类似于以下的响应：

![](img/14a27b89-c4e5-4d29-86c2-4652b7fbe575.png)

在这个例子中，Elasticsearch 包含`144750`条日志记录。

这就完成了 EFK 堆栈的部署。现在，是时候尝试它并找出收集的所有日志记录都关于什么了！

# 尝试 EFK 堆栈

在我们可以尝试 EFK 堆栈之前，我们需要做的第一件事是初始化 Kibana，这样它就知道在 Elasticsearch 中使用哪些搜索索引。一旦完成，我们将尝试以下，根据我的经验，常见的任务：

1.  我们将首先分析 Fluentd 已经收集并在 Elasticsearch 中存储了哪些类型的日志记录。Kibana 具有非常实用的可视化功能，可用于此目的。

1.  接下来，我们将学习如何发现属于同一外部 API 请求处理的不同微服务的日志记录。我们将使用日志记录中的**追踪 ID**作为相关日志记录的关联 ID 来找到相关的日志记录。

1.  第三，我们将学习如何使用 Kibana 进行**根本原因分析**，也就是说，找到错误的实际原因。

# 初始化 Kibana

在我们开始使用 Kibana 之前，我们必须指定在 Elasticsearch 中使用哪些搜索索引以及索引中哪个字段持有日志记录的时间戳。

执行以下步骤初始化 Kibana：

1.  在 web 浏览器中使用`http://kibana.logging.svc.cluster.local:5601` URL 打开 Kibana 的 web UI。

1.  在欢迎页面上，点击“自行探索”按钮。

1.  点击左下角的展开按钮以查看菜单选择的名称。这些将在左侧显示。

1.  点击左侧菜单中的发现。你将被要求定义一个模式，Kibana 用它来确定应从哪个 Elasticsearch 索引中检索日志记录。

1.  输入`logstash-*`索引模式，然后点击下一步。

1.  在下一页上，你将被要求指定包含日志记录时间戳的字段名称。点击时间过滤字段名称的下拉列表，选择唯一可用的字段，即`@timestamp`。

1.  点击创建索引模式按钮。

1.  Kibana 将显示一个页面，总结在选定的索引中可用的字段。

默认情况下，索引以`logstash`命名，这是出于历史原因，尽管用于日志收集的是 Flutentd。

初始化 Kibana 后，我们准备检查我们收集的日志记录。

# 分析日志记录

从 Fluentd 的部署来看，我们知道它立即开始收集大量的日志记录。因此，我们首先需要了解 Fluentd 已经收集并存储在 Elasticsearch 中的日志记录的类型。

我们将使用 Kibana 的可视化功能，按 Kubernetes 命名空间分割日志记录，然后要求 Kibana 显示每个命名空间内按容器类型分割的日志记录。饼图是这种分析的合适图表类型。按照以下步骤创建饼图：

1.  在 Kibana 的 Web UI 中，点击左侧菜单中的可视化。

1.  点击创建新可视化按钮。

1.  选择饼图作为可视化类型。

1.  选择`logstash-*`作为源。

1.  在饼图上方的日期选择器（日期间隔选择器）中，设置一个你喜欢的日期间隔（以下屏幕截图设置为最后 7 天）。点击其日历图标调整时间间隔。

1.  点击添加以创建第一个桶，如下所示：

    1.  选择桶类型，即分割切片。

    1.  对于聚合类型，从下拉列表中选择项。

    1.  作为字段，选择`kubernetes.namespace_name.keyword`。

    1.  对于大小，选择 10。

    1.  启用将其他值分组到单独的桶中。

    1.  启用显示缺失值。

    1.  按下应用更改按钮（Bucket 定义上方蓝色播放图标）。期待一个类似于以下的饼图：

![](img/1d79e158-3af1-495b-8415-e2b769b59454.png)

我们可以看到，日志记录已经分布在我们在前几章中工作的命名空间上：`kube-system`、`istio-system`、`logging`、`cert-manager`以及我们自己的`hands-on`命名空间。为了查看按命名空间分割的日志记录是由哪些容器创建的，我们需要创建第二个桶。

1.  点击再次添加以创建第二个桶：

    1.  选择桶类型，即分割切片。

    1.  对于子聚合类型，从下拉列表中选择项。

    1.  作为字段，选择`kubernetes.container_name.keyword`。

    1.  对于大小，选择“10”。

    1.  启用“将其他值分组到单独的桶中”。

    1.  启用“显示缺失值”。

    1.  再次点击“应用更改”按钮。预期会出现类似下面的饼图：

![](img/660ceb82-205e-401d-8595-5a23cc328e26.png)

在这里，我们可以找到来自我们微服务的日志记录。大多数日志记录来自`product-composite`微服务。

1.  在饼图的顶部，有一组被标记为`missing`的日志记录，即它们既没有 Kubernetes 命名空间也没有容器名称指定。这些缺失的日志记录背后有什么？这些日志记录来自在 Minikube 实例外的 Kubernetes 集群中运行的进程，并且它们是使用 Syslog 存储的。它们可以使用 Syslog 特定的字段进行分析，特别是*标识符字段*。让我们创建一个第三个桶，根据它们的 Syslog 标识符字段（如果有）来划分日志记录。

1.  点击“添加”再次创建一个第三个桶：

    1.  选择桶类型，即“分割切片”。

    1.  作为子聚合类型，从下拉列表中选择“Terms”。

    1.  作为字段，选择`SYSLOG_IDENTIFIER.keyword`。

    1.  启用“将其他值分组到单独的桶中”。

    1.  启用“显示缺失值”。

    1.  点击“应用更改”按钮，预期会出现类似下面的饼图：

![](img/56a1d5be-61bf-49e6-87c2-a18290f65e38.png)

`missing`日志记录最终来自`kubelet`进程，该进程从 Kubernetes 的角度管理节点，以及`dockerd`，管理所有容器的 Docker 守护进程。

既然我们已经找到了日志记录的来源，我们就可以开始从我们的微服务中定位实际的日志记录了。

# 执行根本原因分析

集中日志的最重要特性之一是，它使得使用来自许多源的日志记录来分析错误成为可能，并且基于此执行根本原因分析，即找到错误消息的实际原因。

在本节中，我们将模拟一个错误，并看看我们如何能够找到有关它的信息，一直找到系统中微服务中的某一个微服务中的源代码行引起了错误。为了模拟错误，我们将重新使用我们在第十三章中引入的故障参数*使用 Resilience4j 提高弹性*节的*添加可编程延迟和随机错误*。我们可以使用这个来强制产品微服务抛出异常。按照以下步骤进行操作：

1.  运行以下命令，在搜索具有产品 ID `666`的产品信息时，在产品微服务中生成故障：

```java
curl -H "Authorization: Bearer $ACCESS_TOKEN" -k https://minikube.me/product-composite/666?faultPercent=100
```

预期响应中会出现以下错误：

![](img/8d0a558f-7fc3-4f83-bb92-320d419859ab.png)

现在，我们必须假装我们对这个错误的原因一无所知！否则，根本原因分析将不会非常令人兴奋，对吧？假设我们工作在支持组织中，并被要求调查一些刚刚在最终用户尝试查找有关产品 ID `666`的信息时发生的问题。

1.  在开始分析问题之前，让我们在 Kibana Web UI 中删除之前的搜索过滤器，以便我们可以从头开始。对于我们在上一节中定义的每个过滤器，点击它们的关闭图标（一个 X）以删除它们。删除所有过滤器后，网页应看起来与以下类似：

![](img/82e6c766-b5d4-4bf8-900d-081dfff40319.png)

1.  首先，使用时间选择器选择一个包括问题发生时间的时间间隔。例如，如果您知道问题发生在过去七天之内，就搜索最后七天。

1.  接下来，在此时间范围内搜索日志记录，日志级别设置为 ERROR。这可以通过点击列表中的`spring.level`字段来实现。当你点击这个字段时，它最常用的值将显示在其下方。通过点击带有加号标志的放大镜过滤`ERROR`值。现在 Kibana 将显示在此选定时间范围内，其日志级别设置为 ERROR 的日志记录，如下所示：

![](img/70b40e75-4168-4207-90fb-e528fad127f7.png)

1.  我们可以看到许多与产品 ID `666`相关的错误信息。其中前四个具有相同的跟踪 ID，因此这似乎是一个值得进一步调查的跟踪 ID。

1.  我们还可以看到在顶部四个下面还有更多与同一错误相关的错误信息，但具有不同的跟踪 ID。这些是由产品组合微服务中的重试机制引起的，即在放弃并向调用者返回错误消息之前，它将请求重试几次。

1.  以与上一节相同的方式过滤第一个日志记录的跟踪 ID。

1.  移除`ERROR`日志级别的过滤器，以便能够查看属于此跟踪 ID 的所有记录。预计 Kibana 将响应大量的日志记录。查看最古老的日志记录，即最先发生的记录，该记录看起来可疑。例如，它可能具有`WARN`或`ERROR`日志级别或奇怪的日志消息。默认的排序顺序是显示最新的日志记录在顶部，因此向下滚动到底部并向后搜索（您还可以通过点击`Time`列标题旁边的向上/向下箭头小图标来更改排序顺序，以首先显示最古老的日志记录）。显示`Bad luck, and error occurred`的`WARN`日志消息看起来可能是问题的根本原因。让我们进一步调查：

![](img/47ac8d1a-707a-410f-869f-d0923b4392b2.png)

1.  一旦找到了可能成为问题根源的日志记录，能够找到描述在源代码中异常抛出的附近堆栈跟踪就显得非常重要了**.**不幸的是，我们用于收集多行异常的 Fluentd 插件`fluent-plugin-detect-exceptions`，无法将堆栈跟踪与使用的跟踪 ID 关联起来。因此，当我们在跟踪 ID 上过滤时，堆栈跟踪不会在 Kibana 中显示。相反，我们可以使用 Kibana 中的一个功能来查找显示在特定日志记录附近发生的日志记录的周围日志记录。

1.  使用日志记录左侧的箭头展开显示“坏运气”的日志记录。详细信息关于这个具体的日志记录将显示出来。还有一个名为“查看周围文档”的链接；点击它，就能看到附近的日志记录。期待一个类似于以下的网页：

![](img/d533fe87-485e-4fe5-95c6-8de6c2ae04c6.png)

1.  在显示“坏运气”日志记录上面的带有错误消息“Something went wrong...”的堆栈跟踪的日志记录看起来很有趣，并且是由产品微服务在它记录了*坏运气*日志记录的两毫秒后记录的。它们似乎有关联！那个日志记录中的堆栈跟踪指向了`ProductServiceImpl.java`的第 96 行。查看源代码（见`microservices/product-service/src/main/java/se/magnus/microservices/core/product/services/ProductServiceImpl.java`），第 96 行看起来如下：

```java
throw new RuntimeException("Something went wrong...");
```

这是错误的根本原因。我们之前知道这一点，但现在我们也看到了如何导航到它。

在这个案例中，问题的解决相当简单：只需在 API 请求中省略`faultPercent`参数即可。在其他情况下，找出根本原因的解决可能要困难得多！

本章关于使用 EFK 堆栈进行集中日志记录的内容就此结束。

# 总结

在本章中，我们了解了收集系统景观中微服务的日志记录到一个共同的集中数据库的重要性，在该数据库中可以对存储的日志记录进行分析和搜索。我们使用了 EFK 堆栈，即 Elasticsearch、Fluentd 和 Kibana，来收集、处理、存储、分析和搜索日志记录。

我们不仅用 Fluentd 收集微服务的日志记录，还收集了 Kubernetes 集群中各种支持容器和进程的日志记录。Elasticsearch 被用作文本搜索引擎。与 Kibana 一起，我们了解了识别我们收集了哪些类型的日志记录是多么容易。

我们还学会了如何使用 Kibana 执行重要任务，例如查找来自合作微服务的相关日志记录以及如何进行根本原因分析，即找到错误消息的真实问题。最后，我们学会了如何更新 Fluentd 的配置以及如何让执行的 Fluentd pod 反映出这个更改。

能够以这种方式收集和分析日志记录是在生产环境中的一项重要能力，但是这类活动总是要在日志记录被收集之后进行。另一项重要能力是能够监控微服务的当前健康状况，也就是说，收集并可视化关于硬件资源使用、响应时间等方面的运行时指标。我们在上一章，第十八章，*使用服务网格提高可观测性和管理*，和下一章，第二十章，*监控微服务*，将会了解更多关于监控微服务的内容。

# 问题

1.  一个用户在过去 30 天里在`hands-on`命名空间中搜索了 ERROR 日志信息，使用的是下面截图中的搜索条件，但是没有找到任何结果。为什么？

![](img/6d818708-6286-4a4c-b9f5-1ad9d67581cd.png)

1.  一个用户找到了一个感兴趣的日志记录。用户如何从这个以及其他微服务中找到相关日志记录，例如，那些来自处理外部 API 请求的记录？

![](img/292c8785-1180-40d6-8ad2-2b7e564cc8ef.png)

1.  一个用户找到了一个似乎指示了一个由终端用户报告的问题的根源的日志记录。用户如何找到显示错误发生在源代码中的堆栈跟踪？

![](img/6e935782-d0e3-46aa-8d97-141af6289a9c.png)

1.  下面的 Fluentd 配置元素为什么不起作用？

```java
<match kubernetes.**hands-on**>
  @type rewrite_tag_filter
  <rule>
    key log
    pattern ^(.*)$
    tag spring-boot.${tag}
  </rule>
</match>
```

1.  你怎么确定 Elasticsearch 是否正在运行？

1.  突然之间，你从网页浏览器上失去了对 Kibana 的连接。这个问题是由什么引起的？
