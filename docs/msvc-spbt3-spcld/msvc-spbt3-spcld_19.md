

# 第十九章：使用 EFK 堆栈进行集中日志记录

在本章中，我们将学习如何从微服务实例收集和存储日志记录，以及如何搜索和分析日志记录。正如我们在*第一章*，*微服务介绍*中提到的，当每个微服务实例将其日志记录写入其本地文件系统时，在微服务系统景观中很难获得整体概览。我们需要一个组件可以从微服务的本地文件系统中收集日志记录，并将它们存储在中央数据库中以供分析、搜索和可视化。针对此问题的流行开源解决方案基于以下工具：

+   **Elasticsearch**，一个具有强大搜索和分析大数据集能力的分布式数据库

+   **Fluentd**，一种可以用于从各种来源收集日志记录、过滤和转换收集到的信息，并将其最终发送到各种消费者（例如，Elasticsearch）的数据收集器

+   **Kibana**，Elasticsearch 的图形前端，可用于可视化搜索结果和运行收集到的日志记录的分析

这些工具一起被称为**EFK 堆栈**，以每个工具的首字母命名。

本章将涵盖以下主题：

+   配置 Fluentd

+   在 Kubernetes 上部署 EFK 堆栈以供开发和测试使用

+   分析收集到的日志记录

+   从微服务中发现日志记录并找到相关的日志记录

+   执行根本原因分析

# 技术要求

关于如何安装本书中使用的工具以及如何访问本书源代码的说明，请参阅：

+   *第二十一章*，*macOS 安装说明*

+   *第二十二章*，*使用 WSL 2 和 Ubuntu 的 Microsoft Windows 安装说明*

本章中的代码示例均来自`$BOOK_HOME/Chapter19`的源代码。

如果你想查看本章源代码中应用的变化，即查看我们做出的更改，以便我们可以使用 EFK 堆栈进行集中日志分析，你可以将其与*第十八章*，*使用服务网格提高可观察性和管理*的源代码进行比较。你可以使用你喜欢的`diff`工具比较两个文件夹，`$BOOK_HOME/Chapter18`和`$BOOK_HOME/Chapter19`。

# 介绍 Fluentd

在本节中，我们将学习如何配置 Fluentd 的基础知识。在我们这样做之前，让我们了解一下 Fluentd 的背景以及它在高层次上是如何工作的。

## Fluentd 概述

从历史上看，处理日志记录最受欢迎的开源堆栈之一是 Elastic 的 ELK 堆栈（[`www.elastic.co`](https://www.elastic.co)），它基于 Elasticsearch、Logstash（用于日志收集和转换）和 Kibana。由于 Logstash 运行在 Java 虚拟机上，它需要相对较大的内存量。多年来，已经开发出许多开源替代方案，这些方案比 Logstash 需要的内存量少得多，其中之一就是 Fluentd（[`www.fluentd.org`](https://www.fluentd.org)）。

Fluentd 由 **云原生计算基金会**（**CNCF**）（[`www.cncf.io`](https://www.cncf.io)）管理，该基金会还管理着 Kubernetes 项目。因此，Fluentd 成为了一个基于开源的日志收集器，它自然地成为在 Kubernetes 中运行的日志收集器的首选。与 Elastic 和 Kibana 一起，它形成了 EFK 堆栈。

CNCF 维护了多个类别的替代产品列表，例如，对于日志记录。有关 CNCF 列出的 Fluentd 替代方案，请参阅 [`landscape.cncf.io/card-mode?category=logging&grouping=category`](https://landscape.cncf.io/card-mode?category=logging&grouping=category)。

Fluentd 是用 C 和 Ruby 混合编写的，使用 C 语言处理性能关键的部分，而在需要更多灵活性的地方使用 Ruby，例如，允许使用 Ruby 的 `gem install` 命令简单地安装第三方插件。

日志记录在 Fluentd 中作为事件进行处理，并包含以下信息：

+   一个描述日志记录创建时间的 `time` 字段

+   一个标识日志记录类型的 `tag` 字段——该标签由 Fluentd 的路由引擎用于确定日志记录的处理方式

+   一个包含实际日志信息的 `record`，该信息存储为 JSON 对象

Fluentd 配置文件用于告诉 Fluentd 如何收集、处理，并将日志记录最终发送到各种目标，例如 Elasticsearch。配置文件由以下类型的核心元素组成：

+   `<source>`：源元素描述 Fluentd 将收集日志记录的位置，例如，收集由 Docker 容器写入的日志文件。

    *跟踪日志文件*意味着监控写入日志文件的内容。一个常用的 Unix/Linux 工具，用于监控文件附加的内容，名为 `tile`。

    源元素通常会对日志记录进行标记，描述日志记录的类型。例如，它们可以用来标记日志记录，表明它们来自在 Kubernetes 中运行的容器。

+   `<filter>`：过滤器元素用于处理日志记录。例如，一个过滤器元素可以解析来自基于 Spring Boot 的微服务的日志记录，并将日志消息的有趣部分提取到日志记录中的单独字段。将信息提取到日志记录中的单独字段使得信息可以通过 Elasticsearch 进行搜索。过滤器元素根据日志记录的标签选择要处理的日志记录。

+   `<match>`：匹配元素决定将日志记录发送到何处，充当输出元素。它们用于执行两个主要任务：

    +   将处理后的日志记录发送到目标，例如 Elasticsearch。

    +   路由决定如何处理日志记录。路由规则可以重写标记并将日志记录重新发射到 Fluentd 路由引擎以进行进一步处理。路由规则以 `<match>` 元素内部的嵌入式 `<rule>` 元素表示。输出元素决定要处理哪些日志记录，与过滤器类似：基于日志记录的标记。

Fluentd 随带一些内置和外部第三方插件，这些插件被源、过滤器和输出元素使用。在下一节中，我们将通过配置文件了解其中的一些插件。有关可用插件的信息，请参阅 Fluentd 的文档，可在 [`docs.fluentd.org`](https://docs.fluentd.org) 找到。

在完成对 Fluentd 的概述之后，我们准备了解 Fluentd 如何配置以处理来自我们的微服务的日志记录。

## 配置 Fluentd

Fluentd 的配置基于 GitHub 上 Fluentd 项目的配置文件，即 `fluentd-kubernetes-daemonset`。该项目包含 Fluentd 配置文件，说明如何从在 Kubernetes 中运行的容器收集日志记录，以及一旦处理完毕如何将它们发送到 Elasticsearch。

我们将不修改地重用此配置，这将极大地简化我们的配置。Fluentd 配置文件可以在 [`github.com/fluent/fluentd-kubernetes-daemonset/tree/master/archived-image/v1.4/debian-elasticsearch/conf`](https://github.com/fluent/fluentd-kubernetes-daemonset/tree/master/archived-image/v1.4/debian-elasticsearch/conf) 找到。

提供此功能的配置文件是 `kubernetes.conf` 和 `fluent.conf`。`kubernetes.conf` 配置文件包含以下信息：

+   源元素跟踪容器日志文件以及运行在 Kubernetes 外部的进程（例如，`kubelet` 和 Docker 守护进程）的日志文件。源元素还会将 Kubernetes 的日志记录标记为具有完整文件名的全名，其中 `/` 被替换为 `.` 并以 `kubernetes` 为前缀。由于标记基于完整文件名，因此名称包含命名空间、Pod 和容器等名称。因此，标记对于通过匹配标记查找感兴趣的日志记录非常有用。

    例如，来自 `product-composite` 微服务的标记可能类似于 `kubernetes.var.log.containers.product-composite-7...s_hands-on_comp-e...b.log`，而同一 Pod 中相应的 `istio-proxy` 的标记可能类似于 `kubernetes.var.log.containers.product-composite-7...s_hands-on_istio-proxy-1...3.log`。

+   一个过滤器元素，丰富了来自运行在 Kubernetes 内部容器中的日志记录，以及包含容器名称和运行命名空间等信息的特定于 Kubernetes 的字段。

主配置文件`fluent.conf`包含以下信息：

+   `@include`语句用于其他配置文件，例如我们之前描述的`kubernetes.conf`文件。它还包括放置在特定文件夹中的自定义配置文件，这使得我们能够非常容易地重用这些配置文件而无需任何更改，并提供我们自己的配置文件，该文件仅处理与我们的日志记录相关的处理。我们只需将我们的配置文件放置在`fluent.conf`文件指定的文件夹中即可。

+   一个输出元素，将日志记录发送到 Elasticsearch。

如后续的*部署 Fluentd*部分所述，这两个配置文件将被打包到我们将为 Fluentd 构建的 Docker 镜像中。

在我们自己的配置文件中需要覆盖的内容如下：

+   识别和解析来自我们的微服务的 Spring Boot 格式日志记录。

+   处理多行堆栈跟踪。堆栈跟踪使用多行写入日志文件，这使得 Fluentd 难以将堆栈跟踪作为一个单独的日志记录处理。

+   将来自`istio-proxy`边车和同一 Pod 中运行的微服务创建的日志记录分开。由`istio-proxy`创建的日志记录不遵循我们基于 Spring Boot 的微服务创建的日志模式。因此，它们必须单独处理，以免 Fluentd 尝试将它们解析为 Spring Boot 格式的日志记录。

为了实现这一点，配置在很大程度上是基于使用`rewrite_tag_filter`插件。此插件可用于根据更改标签名称的概念路由日志记录，然后将日志记录重新发射到 Fluentd 路由引擎。

该处理由以下 UML 活动图总结：

![包含文本、截图、图表的图片，自动生成描述](img/B19825_19_01.png)

图 19.1：Fluentd 处理日志记录

从高层次来看，配置文件的设计如下：

+   Istio 的所有日志记录的标签，包括`istio-proxy`，都以前缀`istio`开头，以便它们可以与基于 Spring Boot 的日志记录分开。

+   来自`hands-on`命名空间的所有日志记录的标签（`istio-proxy`的日志记录除外）都以前缀`spring-boot`开头。

+   Spring Boot 的日志记录会检查是否存在多行堆栈跟踪。如果日志记录是多行堆栈跟踪的一部分，它将由第三方 `detect-exceptions` 插件处理以重新创建堆栈跟踪。否则，它将使用正则表达式解析以提取感兴趣的信息。有关此第三方插件的详细信息，请参阅 *Deploying Fluentd* 部分。

`fluentd-hands-on.conf` 配置文件实现了此活动图。配置文件放置在 Kubernetes ConfigMap 内（参见 `kubernetes/efk/fluentd-hands-on-configmap.yml`）。让我们一步一步地来了解这个过程，如下所示：

1.  首先是 ConfigMap 的定义和配置文件名，`fluentd-hands-on.conf`。它看起来如下：

    ```java
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: fluentd-hands-on-config
      namespace: kube-system
    data:
      fluentd-hands-on.conf: | 
    ```

    我们可以看到，`data` 元素将包含 Fluentd 的配置。它以文件名开始，并使用垂直线 `|` 标记 Fluentd 嵌入式配置文件的开始。

1.  第一个 `<match>` 元素匹配来自 Istio 的日志记录，即以 `Kubernetes` 开头并包含 `istio` 作为其命名空间或容器名称一部分的标签。它看起来像这样：

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

    让我们解释前面的源代码：

+   `<match>` 元素匹配任何符合 `kubernetes.**istio**` 模式的标签，即以 `Kubernetes` 开头并在标签名称中包含 `istio` 的标签。`istio` 可以来自命名空间或容器的名称；两者都是标签的一部分。

+   `<match>` 元素只包含一个 `<rule>` 元素，该元素将标签前缀设置为 `istio`。`${tag}` 变量持有当前标签的值。

+   由于这是 `<match>` 元素中的唯一 `<rule>` 元素，因此它被配置为匹配所有日志记录。

+   由于所有来自 Kubernetes 的日志记录都有一个 `log` 字段，因此 `key` 字段被设置为 `log`，即规则在日志记录中查找 `log` 字段。

+   为了匹配 `log` 字段中的任何字符串，`pattern` 字段被设置为 `^(.*)$` 正则表达式。`^` 标记字符串的开始，而 `$` 标记字符串的结束。`(.*)` 匹配任意数量的字符，除了换行符。

+   日志记录被重新发送到 Fluentd 路由引擎。由于配置文件中没有其他元素匹配以 `istio` 开头的标签，因此日志记录将直接发送到之前描述的 `fluent.conf` 文件中定义的 Elasticsearch 输出元素。

1.  第二个 `<match>` 元素匹配来自 `hands-on` 命名空间的所有日志记录，即由我们的微服务发出的日志记录。它看起来像这样：

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

    从源代码中，我们可以看到：

+   我们微服务发出的日志记录使用由 Spring Boot 定义的日志消息格式规则，因此它们的标签以 `spring-boot` 开头。然后，它们被重新发送以进行进一步处理。

+   `<match>` 元素配置方式与之前查看的 `<match kubernetes.**istio**>` 元素相同，以匹配所有记录。

1.  第三个 `<match>` 元素匹配 `spring-boot` 日志记录，并确定它们是普通 Spring Boot 日志记录还是多行堆栈跟踪的一部分。自 Spring Boot 3 以来，Project Reactor 已向堆栈跟踪添加了额外信息，以澄清导致异常的原因。（有关详细信息，请参阅 [`projectreactor.io/docs/core/release/reference/#_reading_a_stack_trace_in_debug_mode`](https://projectreactor.io/docs/core/release/reference/#_reading_a_stack_trace_in_debug_mode)。）

    为了能够解析实际的堆栈跟踪，我们将过滤掉此信息。`<match>` 元素看起来像这样：

    ```java
     <match spring-boot.**>
          @type rewrite_tag_filter
          <rule>
            key log
            pattern /^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}\.\d{3}([-+]\d{2}:\d{2}|Z).*/
            tag parse.${tag}
          </rule>
          # Get rid of Reactor debug info:
          #
          #   Suppressed: reactor.core.publisher.FluxOnAssembly$OnAssemblyException:
          # Error has been observed at the following site(s):
          #   *__checkpoint ,á¢ Handler se.magnus.microservices.core.product.services.ProductServiceImpl#getProduct(HttpHeaders, int, int, int) [DispatcherHandler]
          #   *__checkpoint ,á¢ org.springframework.web.filter.reactive.ServerHttpObservationFilter [DefaultWebFilterChain]
          #   *__checkpoint ,á¢ HTTP GET "/product/1?faultPercent=100" [ExceptionHandlingWebHandler]
          # Original Stack Trace:
          <rule>
            key log
            pattern /^\s+Suppressed:.*$/
            tag skip.${tag}
          </rule>
          <rule>
            key log
            pattern /^Error has been observed at the following site.*/
            tag skip.${tag}
          </rule>
          <rule>
            key log
            pattern /^\s+\*__checkpoint.*/
            tag skip.${tag}
          </rule>
          <rule>
            key log
            pattern /^Original Stack Trace:.*/
            tag skip.${tag}
          </rule>
          <rule>
            key log
            pattern /^.*/
            tag check.exception.${tag}
          </rule>
        </match> 
    ```

    如源代码所示，这是通过使用六个 `<rule>` 元素来确定的：

+   第一个使用正则表达式检查日志元素中的 `log` 字段是否以时间戳开头。

+   如果 `log` 字段以时间戳开头，则该日志记录被视为普通的 Spring Boot 日志记录，并且其标签以 `parse` 为前缀。

+   接下来是四个规则元素，用于过滤掉 Project Reactor 添加的额外信息；它们都将标签前缀设置为 `skip`。

+   否则，最后一个 `<rule>` 元素将匹配，并将日志记录作为多行日志记录处理。其标签以 `check.exception` 为前缀。

+   在此处理之后，日志记录将重新发射，并且其标签将开始于 `check.exception.spring-boot`、`skip.spring-boot` 或 `parse.spring-boot`。

1.  第四个 `<match>` 元素用于消除 Project Reactor 的日志输出，即以 `skip.spring-boot` 开头的匹配标签。该 `<match>` 元素应用了 `null` 输出插件，丢弃事件。它看起来像这样：

    ```java
     <match skip.spring-boot.**>
          @type null
        </match> 
    ```

1.  在第五个 `<match>` 元素中，选定的日志记录具有以 `check.exception.spring-boot` 开头的标签，即作为多行堆栈跟踪一部分的日志记录。它看起来像这样：

    ```java
     <match check.exception.spring-boot.**>
          @type detect_exceptions
          languages java
          remove_tag_prefix check
          message log
          multiline_flush_interval 5
        </match> 
    ```

    `detect_exceptions` 插件的工作方式如下：

+   `detect_exceptions` 插件用于将多个单行日志记录合并成一个包含完整堆栈跟踪的单个日志记录。

+   在多行日志记录重新发射到路由引擎之前，将 `check` 前缀从标签中移除，以防止日志记录的无限循环处理。

1.  最后，配置文件由一个 `<filter>` 元素组成，该元素使用正则表达式解析 Spring Boot 日志消息，提取感兴趣的信息。它看起来像这样：

    ```java
     <filter parse.spring-boot.**>
          @type parser
          key_name log
          time_key time
          time_format %Y-%m-%dT%H:%M:%S.%N
          reserve_data true
          format /^(?<time>\d{4}-\d{2}-
          \d{2}T\d{2}:\d{2}:\d{2}\.\d{3}([-+]\d{2}:\d{2}|Z))\s+
          (?<spring.level>[^\s]+)\s+
          (\[(?<spring.service>[^,]*),(?<spring.trace>[^,]*),(?
          <spring.span>[^\]]*)]*\])\s+
          (?<spring.pid>\d+)\s+---\s+\[\s*(?<spring.thread>[^\]]+)\]\s+
          (?<spring.class>[^\s]+)\s*:\s+
          (?<log>.*)$/
        </filter> 
    ```

注意，过滤器元素不会重新发射日志记录；相反，它们只是将它们传递到配置文件中匹配日志记录标签的下一个元素。

从存储在日志记录中 `log` 字段的 Spring Boot 日志消息中提取以下字段：

+   `<time>`：日志记录创建时的日期和时间戳

+   `<spring.level>`：日志记录的日志级别：`FATAL`、`ERROR`、`WARN`、`INFO`、`DEBUG` 或 `TRACE`

+   `<spring.service>`：微服务的名称

+   `<spring.trace>`：用于执行分布式跟踪的跟踪 ID

+   `<spring.span>`：span ID，表示分布式处理中该微服务执行的部分的 ID

+   `<spring.pid>`：进程 ID

+   `<spring.thread>`：线程 ID

+   `<spring.class>`：Java 类的名称

+   `<log>`：实际的日志消息

使用 `spring.application.name` 属性指定基于 Spring Boot 的微服务的名称。此属性已添加到配置仓库中每个微服务特定的属性文件，位于 `config-repo` 文件夹中。

正确使用正则表达式可能具有挑战性，至少可以说。幸运的是，有几个网站可以帮助。当涉及到与 Fluentd 一起使用正则表达式时，我建议使用以下网站：[`fluentular.herokuapp.com/`](https://fluentular.herokuapp.com/)。

现在我们已经了解了 Fluentd 的工作原理以及配置文件的构建方式，我们准备部署 EFK 栈。

# 在 Kubernetes 上部署 EFK 栈

在 Kubernetes 上部署 EFK 栈将与我们的微服务部署方式相同：使用 Kubernetes 清单文件来部署对象，如 Deployments、Services 和 ConfigMaps。

EFK 栈的部署分为三个部分：

+   部署 Elasticsearch 和 Kibana

+   部署 Fluentd

+   设置访问 Elasticsearch 和 Kibana

但首先，我们需要构建和部署我们自己的微服务。

## 构建和部署我们的微服务

使用 `test-em-all.bash` 测试脚本构建、部署和验证部署的方式与第十八章 *使用服务网格提高可观察性和管理* 中 *运行命令创建服务网格* 部分的方式相同。这些说明假定 cert-manager 和 Istio 已按第十七章和第十八章中的说明安装。

运行以下命令开始：

1.  首先，使用以下命令从源代码构建 Docker 镜像：

    ```java
    cd $BOOK_HOME/Chapter19
    eval $(minikube docker-env -u)
    ./gradlew build
    eval $(minikube docker-env)
    docker-compose build 
    ```

`eval $(minikube docker-env -u)` 命令确保 `./gradlew build` 命令使用主机的 Docker 引擎，而不是 Minikube 实例中的 Docker 引擎。`build` 命令使用 Docker 运行测试容器。

1.  重新创建 `hands-on` 命名空间并将其设置为默认命名空间：

    ```java
    kubectl delete namespace hands-on
    kubectl apply -f kubernetes/hands-on-namespace.yml
    kubectl config set-context $(kubectl config current-context) --namespace=hands-on 
    ```

1.  使用以下命令解决 Helm 图表的依赖关系。

    首先，我们更新 `components` 文件夹中的依赖项：

    ```java
    for f in kubernetes/helm/components/*; do helm dep up $f; done 
    ```

    接下来，我们更新 `environments` 文件夹中的依赖项：

    ```java
    for f in kubernetes/helm/environments/*; do helm dep up $f; done 
    ```

1.  使用 Helm 部署系统景观并等待所有部署完成：

    ```java
    helm install hands-on-dev-env \
      kubernetes/helm/environments/dev-env \
      -n hands-on --wait 
    ```

1.  如果 Minikube 隧道尚未运行，请在单独的终端窗口中启动它（如需回顾，请参阅第十八章，*设置访问 Istio 服务*部分）： 

    ```java
    minikube tunnel 
    ```

    记住，这个命令要求你的用户具有`sudo`权限，并且在启动时输入你的密码。在命令请求密码之前需要几秒钟的时间，所以很容易错过！

    1.  使用以下命令运行正常测试以验证部署：

        ```java
        ./test-em-all.bash 
        ```

    预期输出将与我们在前几章中看到的结果相似：

    ![计算机屏幕截图  自动生成的描述](img/B19825_19_02.png)

    图 19.2：测试运行良好

1.  你也可以通过运行以下命令手动尝试 API：

    ```java
    ACCESS_TOKEN=$(curl -k https://writer:secret-writer@minikube.me/oauth2/token -d grant_type=client_credentials -d scope="product:read product:write" -s | jq .access_token -r)
    echo ACCESS_TOKEN=$ACCESS_TOKEN
    curl -ks https://minikube.me/product-composite/1 -H "Authorization: Bearer $ACCESS_TOKEN" | jq .productId 
    ```

预期在响应中收到请求的产品 ID，`1`。

部署了微服务后，我们可以继续部署 Elasticsearch 和 Kibana！

## 部署 Elasticsearch 和 Kibana

我们将部署 Elasticsearch 和 Kibana 到它们自己的命名空间`logging`。Elasticsearch 和 Kibana 将使用 Kubernetes Deployment 和 Service 对象进行开发和测试部署。服务将在 Kubernetes 集群内部公开 Elasticsearch 和 Kibana 的标准端口，即 Elasticsearch 的端口`9200`和 Kibana 的端口`5601`。

要为 Elasticsearch 和 Kibana 提供外部 HTTP 访问，我们将创建与第十八章中相同的 Istio 对象，即使用服务网格来提高可观察性和管理性，用于 Kiali 和 Jaeger – 如果需要，请参阅“设置对 Istio 服务的访问”部分进行回顾。这将使 Elasticsearch 和 Kibana 在[`elasticsearch.minikube.me`](https://elasticsearch.minikube.me)和[`kibana.minikube.me`](https://kibana.minikube.me)可用。

清单文件已打包在`kubernetes/helm/environments/logging`文件夹中的 Helm 图表中。

有关在 Kubernetes 生产环境中部署 Elasticsearch 和 Kibana 的推荐选项，请参阅[`www.elastic.co/elastic-cloud-kubernetes`](https://www.elastic.co/elastic-cloud-kubernetes)。

我们将使用在撰写本章时 7.0 版本可用的最新版本：

+   Elasticsearch 版本 7.17.10

+   Kibana 版本 7.17.10

由于 Fluentd 插件对 Elasticsearch 的支持有限，因此没有使用 Elasticsearch 版本 8；请参阅[`github.com/uken/fluent-plugin-elasticsearch/issues/1005`](https://github.com/uken/fluent-plugin-elasticsearch/issues/1005)。我们将在下一节中使用此插件来安装 Fluentd 的基 Docker 镜像，即`fluentd-kubernetes-daemonset`。

在我们部署之前，让我们看看 Helm 图表的`template`文件夹中清单文件中最有趣的部分。

### 清单文件的概述

Elasticsearch 的清单文件`elasticsearch.yml`包含了一个标准的 Kubernetes Deployment 和 Service 对象，我们之前已经多次见过，例如在*第十五章*，*Kubernetes 简介*中的*尝试示例部署*部分。清单文件中最有趣的部分如下：

```java
apiVersion: apps/v1
kind: Deployment
...
      containers:
      - name: elasticsearch
        image: docker.elastic.co/elasticsearch/elasticsearch:7.17.10
        resources:
          limits:
            cpu: 500m
            memory: 2Gi
          requests:
            cpu: 500m
            memory: 2Gi 
```

让我们解释一下这些清单文件中的某些内容：

+   我们使用来自 Elastic 的官方 Docker 镜像，可在 `docker.elastic.co` 获取。版本设置为 `7.17.10`。

+   Elasticsearch 容器允许分配相对较大的内存量——2 GB——以便能够以良好的性能运行查询。内存越多，性能越好。

Kibana 的配置文件 `kibana.yml` 也包含标准的 Kubernetes 部署和服务对象。配置文件中最有趣的部分如下：

```java
apiVersion: apps/v1
kind: Deployment
...
      containers:
      - name: kibana
        image: docker.elastic.co/kibana/kibana:7.17.10
        env:
        - name: ELASTICSEARCH_URL
          value: http://elasticsearch:9200 
```

让我们解释一下一些配置文件：

+   对于 Kibana，我们也使用来自 Elastic 的官方 Docker 镜像，可在 `docker.elastic.co` 获取。版本设置为 `7.17.10`。

+   为了将 Kibana 连接到 Elasticsearch Pod，定义了一个环境变量 `ELASTICSEARCH_URL`，用于指定 Elasticsearch 服务的地址，`http://elasticsearch:9200`。

最后，设置外部访问的 Istio 配置文件位于 `expose-elasticsearch.yml` 和 `expose-kibana.yml` 文件中。关于如何使用 `Gateway`、`VirtualService` 和 `DestinationRule` 对象的复习，请参阅第十八章的 *创建服务网格* 部分。它们将提供以下外部请求转发：

+   [`elasticsearch.minikube.me`](https://elasticsearch.minikube.me) → `http://elasticsearch:9200`

+   [`kibana.minikube.me`](https://kibana.minikube.me) → `http://kibana:5601`

基于这些见解，我们准备部署 Elasticsearch 和 Kibana。

### 执行部署命令

通过以下步骤部署 Elasticsearch 和 Kibana：

1.  为了使部署步骤运行得更快，使用以下命令预取 Elasticsearch 和 Kibana 的 Docker 镜像：

    ```java
    eval $(minikube docker-env)
    docker pull docker.elastic.co/elasticsearch/elasticsearch:7.17.10
    docker pull docker.elastic.co/kibana/kibana:7.17.10 
    ```

1.  使用 Helm 图表创建 `logging` 命名空间，在其中部署 Elasticsearch 和 Kibana，并等待 Pod 准备就绪：

    ```java
    helm install logging-hands-on-add-on kubernetes/helm/environments/logging \
        -n logging --create-namespace --wait 
    ```

1.  使用以下命令验证 Elasticsearch 是否正在运行：

    ```java
    curl https://elasticsearch.minikube.me -sk | jq -r .tagline 
    ```

    预期响应为 `You Know, for Search`。

根据您的硬件配置，您可能需要等待一分钟或两分钟，Elasticsearch 才会响应此消息。

1.  使用以下命令验证 Kibana 是否正在运行：

    ```java
    curl https://kibana.minikube.me \
      -kLs -o /dev/null -w "%{http_code}\n" 
    ```

预期响应为 `200`。

同样，您可能需要等待一分钟或两分钟，Kibana 才会初始化并响应 `200`。

部署了 Elasticsearch 和 Kibana 后，我们可以开始部署 Fluentd。

## 部署 Fluentd

相比于部署 Elasticsearch 和 Kibana，部署 Fluentd 要复杂一些。为了部署 Fluentd，我们将使用 Fluentd 项目在 Docker Hub 上发布的 Docker 镜像 `fluent/fluentd-kubernetes-daemonset` 以及来自 GitHub 上 Fluentd 项目的示例 Kubernetes 清单文件 `fluentd-kubernetes-daemonset`。它位于 [`github.com/fluent/fluentd-kubernetes-daemonset`](https://github.com/fluent/fluentd-kubernetes-daemonset)。正如项目名称所暗示的，Fluentd 将作为 DaemonSet 部署，在每个 Kubernetes 集群的节点上运行一个 Pod。每个 Fluentd Pod 负责收集与 Pod 在同一节点上运行的进程和容器的日志输出。由于我们使用的是具有单个节点集群的 Minikube，因此我们只有一个 Fluentd Pod。

为了处理包含异常堆栈跟踪的多行日志记录，我们将使用由 Google 提供的第三方 Fluentd 插件 `fluent-plugin-detect-exceptions`，该插件可在 [`github.com/GoogleCloudPlatform/fluent-plugin-detect-exceptions`](https://github.com/GoogleCloudPlatform/fluent-plugin-detect-exceptions) 找到。为了能够使用此插件，我们将构建自己的 Docker 镜像，其中将安装 `fluent-plugin-detect-exceptions` 插件。

Fluentd 的 Docker 镜像 `fluentd-kubernetes-daemonset` 将用作基础镜像。

我们将使用以下版本：

+   Fluentd 版本 1.4.2

+   `fluent-plugin-detect-exceptions` 版本 0.0.12

在我们部署之前，让我们看看清单文件中最有趣的部分。

### 对清单文件的概述

用于构建 Docker 镜像的 Dockerfile，`kubernetes/efk/Dockerfile`，如下所示：

```java
FROM fluent/fluentd-kubernetes-daemonset:v1.4.2-debian-elasticsearch-1.1
RUN gem install fluent-plugin-detect-exceptions -v 0.0.12 \
 && gem sources --clear-all \
 && rm -rf /var/lib/apt/lists/* \
           /home/fluent/.gem/ruby/2.3.0/cache/*.gem 
```

让我们详细解释一下：

+   基础镜像是 Fluentd 的 Docker 镜像 `fluentd-kubernetes-daemonset`。`v1.4.2-debian-elasticsearch-1.1` 标签指定将使用包含内置支持将日志记录发送到 Elasticsearch 的软件包的 1.4.2 版本。基础 Docker 镜像包含在 *配置 Fluentd* 部分中提到的 Fluentd 配置文件。

+   Google 插件 `fluent-plugin-detect-exceptions` 使用 Ruby 的包管理器 `gem` 进行安装。

DaemonSet 的清单文件 `kubernetes/efk/fluentd-ds.yml` 基于在 `fluentd-kubernetes-daemonset` 项目中的一个示例清单文件，该文件可以在 [`github.com/fluent/fluentd-kubernetes-daemonset/blob/master/fluentd-daemonset-elasticsearch.yaml`](https://github.com/fluent/fluentd-kubernetes-daemonset/blob/master/fluentd-daemonset-elasticsearch.yaml) 找到。

这个文件有点复杂，所以让我们分别查看最有趣的部分：

1.  首先，这是 DaemonSet 的声明：

    ```java
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: fluentd
      namespace: kube-system 
    ```

    `kind` 键指定这是一个 DaemonSet。`namespace` 键指定 DaemonSet 将在 `kube-system` 命名空间中创建，而不是在 Elasticsearch 和 Kibana 部署的 `logging` 命名空间中。

1.  下一个部分指定由 DaemonSet 创建的 Pod 的模板。最有趣的部分如下：

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

    用于 Pod 的 Docker 镜像为`hands-on/fluentd:v1`。我们将使用之前描述的 Dockerfile 构建此 Docker 镜像。

    Docker 镜像支持多个环境变量，并用于自定义它。其中两个最重要的如下：

+   `FLUENT_ELASTICSEARCH_HOST`，指定 Elasticsearch 服务的域名，`elasticsearch.logging`

+   `FLUENT_ELASTICSEARCH_PORT`，指定用于与 Elasticsearch 通信的端口，`9200`

由于 Fluentd Pod 运行在不同的命名空间中，因此不能使用其短名称`elasticsearch`来指定主机名。相反，DNS 名称的命名空间部分也必须指定，即`elasticsearch.logging`。作为替代，也可以使用**完全限定域名**（**FQDN**）`elasticsearch.logging.svc.cluster.local`。但由于 DNS 名称的最后部分`svc.cluster.local`在 Kubernetes 集群内的所有 DNS 名称中都是共享的，因此不需要指定。

1.  最后，将多个卷（即文件系统）映射到 Pod，如下所示：

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

让我们详细查看源代码：

+   主机（即节点）上的三个文件夹映射到 Fluentd Pod。这些文件夹包含 Fluentd 将跟踪和收集日志记录的日志文件。这些文件夹是`/var/log`、`/var/lib/docker/containers`和`/run/log/journal`。

+   我们自己的配置文件，该文件指定了 Fluentd 如何处理来自我们的微服务的日志记录，通过名为`fluentd-hands-on-config`的 ConfigMap 映射到`/fluentd/etc/conf.d`文件夹。基础 Docker 镜像配置 Fluentd 以包含在`/fluentd/etc/conf.d`文件夹中找到的任何配置文件。有关详细信息，请参阅*配置 Fluentd*部分。

对于 DaemonSet 的完整源代码清单文件，请参阅`kubernetes/efk/fluentd-ds.yml`文件。

现在我们已经了解了所有内容，我们准备执行 Fluentd 的部署。

### 运行部署命令

要部署 Fluentd，我们必须构建 Docker 镜像，创建 ConfigMap，最后部署 DaemonSet。运行以下命令执行这些步骤：

1.  使用以下命令构建 Docker 镜像并使用`hands-on/fluentd:v1`标记它：

    ```java
    eval $(minikube docker-env)
    docker build -f kubernetes/efk/Dockerfile -t hands-on/fluentd:v1 kubernetes/efk/ 
    ```

1.  使用以下命令创建 ConfigMap，部署 Fluentd 的 DaemonSet，并等待 Pod 就绪：

    ```java
    kubectl apply -f kubernetes/efk/fluentd-hands-on-configmap.yml 
    kubectl apply -f kubernetes/efk/fluentd-ds.yml
    kubectl wait --timeout=120s --for=condition=Ready pod -l app=fluentd -n kube-system 
    ```

1.  使用以下命令验证 Fluentd Pod 是否健康：

    ```java
    kubectl logs -n kube-system -l app=fluentd --tail=-1 | grep "fluentd worker is now running worker" 
    ```

    预期响应为`2023-05-22 14:59:46 +0000 [info]: #0 fluentd worker is now running worker=0`。

对于 Elasticsearch 和 Kibana，在 Fluentd 响应此消息之前，可能需要等待一分钟或两分钟。

1.  Fluentd 将从 Minikube 实例中的各种容器开始收集相当数量的日志记录。大约一分钟后，你可以使用以下命令询问 Elasticsearch 收集了多少日志记录：

    ```java
    curl https://elasticsearch.minikube.me/_all/_count -sk | jq .count 
    ```

1.  第一次执行此命令时可能会有些慢，但应该返回数千条日志记录的总数。在我的情况下，它返回了`55607`。

这完成了 EFK 堆栈的部署。现在，是时候尝试它并找出所有收集到的日志记录的内容了！

# 尝试 EFK 堆栈

在我们尝试 EFK 堆栈之前，我们需要先初始化 Kibana，以便它知道在 Elasticsearch 中使用哪些索引。

在 Elasticsearch 中，**索引**对应于 SQL 概念中的**数据库**。SQL 概念中的**表**、**行**和**列**对应于 Elasticsearch 中的**类型**、**文档**和**属性**。

完成上述操作后，我们将尝试以下常见任务：

1.  我们将首先分析 Fluentd 收集并存储在 Elasticsearch 中的日志记录类型。Kibana 具有非常实用的可视化功能，可用于此目的。

1.  接下来，我们将学习如何在处理外部请求时查找由微服务创建的所有相关日志记录。我们将使用日志记录中的**跟踪 ID**作为关联 ID 来查找相关日志记录。

1.  最后，我们将学习如何使用 Kibana 进行**根本原因分析**，找到错误的真正原因。

## 初始化 Kibana

在我们开始使用 Kibana 之前，我们必须指定在 Elasticsearch 中使用哪些搜索索引以及索引中哪个字段持有日志记录的时间戳。

提醒一下，我们使用的是由我们自己的 CA 创建的证书，这意味着它不被网络浏览器信任！有关如何使网络浏览器接受我们的证书的复习，请参阅第十八章的*观察服务网格*部分。

执行以下步骤以初始化 Kibana：

1.  使用网络浏览器中的[`kibana.minikube.me`](https://kibana.minikube.me) URL 打开 Kibana 的 Web UI。

1.  在**欢迎主页**上，点击左上角的汉堡菜单**≡**（三条横线）并在菜单底部点击左侧的**堆栈管理**。

1.  在**管理**菜单中，滚动到最底部并选择**索引模式**。

1.  点击名为**创建索引模式**的按钮。

1.  将索引模式名称输入为`logstash-*`并点击**下一步**按钮。

由于历史原因，索引默认命名为`logstash`，尽管使用了 Fluentd 进行日志收集。

1.  点击**时间戳字段**的下拉列表并选择唯一可用的字段，`@timestamp`。

1.  点击**创建索引模式**按钮。

Kibana 将显示一个页面，总结所选索引中可用的字段。

在 Kibana 初始化后，我们准备好检查收集到的日志记录。

## 分析日志记录

从 Fluentd 的部署开始，我们就知道它立即开始收集大量的日志记录。因此，我们首先需要做的是了解 Fluentd 收集并存储在 Elasticsearch 中的日志记录类型。

我们将使用 Kibana 的可视化功能按 Kubernetes 命名空间划分日志记录，然后要求 Kibana 显示每个命名空间内日志记录按容器类型划分的情况。饼图是此类分析合适的图表类型。按照以下步骤创建饼图：

1.  在 Kibana 的 Web UI 中，再次点击汉堡菜单，并在菜单中选择**分析**下的**可视化库**。

1.  点击**创建新可视化**按钮，并在下一页选择**透镜**类型。将显示如下网页：

![计算机截图  描述自动生成，置信度中等](img/B19825_19_03.png)

图 19.3：在 Kibana 中开始分析日志记录

1.  确认**logstash-***是左上角下拉菜单中选定的索引模式。

1.  在索引模式旁边的**垂直堆叠条形图**下拉菜单中，选择**饼图**作为可视化类型。

1.  在饼图上方的日期选择器（一个日期区间选择器）中，设置一个足够大的日期区间以覆盖感兴趣的日志记录（在下述截图中设置为**最后 15 分钟**）。点击其日历图标以调整时间区间。

1.  在索引模式下方名为**搜索字段名称**的域中，输入`kubernetes.namespace_name.keyword`。

1.  在**可用字段**列表下，现在出现了字段**kubernetes.namespace_name.keyword**。将此字段拖放到页面中间名为**在此处放下一些字段以开始**的大框中。Kibana 将立即开始分析日志记录并按 Kubernetes 命名空间绘制饼图。

    在我的情况下，看起来是这样的：

    ![计算机截图  描述自动生成，置信度中等](img/B19825_19_04.png)

    图 19.4：Kibana 按 Kubernetes 命名空间分析日志记录

    我们可以看到日志记录被分为我们在前几章中一直在使用的命名空间：`kube-system`、`istio-system`、`logging`以及我们自己的`hands-on`命名空间。要查看每个命名空间中创建了哪些容器日志记录，我们需要添加第二个字段。

1.  在**搜索字段名称**字段中，输入`kubernetes.container_name.keyword`。

1.  在**可用字段**列表中，现在出现了字段`kubernetes.container_name.keyword`。将此字段拖放到页面中间显示饼图的较大框中。Kibana 将立即开始分析日志记录并按 Kubernetes 命名空间和容器名称绘制饼图。

1.  在*步骤 9*的结果中，我们可以看到来自`coredns`的大量日志记录，在我的情况下占 67%。由于我们对此类日志记录不特别感兴趣，我们可以通过以下步骤添加过滤器来删除它们：

    1.  点击**+ 添加过滤器**（在左上角）。

    1.  选择**字段**`kubernetes.container_name.keyword`和**不是**`-****Operator`。最后，输入**值**`coredns`并点击**保存**按钮。

1.  在我的情况下，渲染的饼图现在看起来是这样的：![计算机屏幕截图  描述由中等置信度自动生成](img/B19825_19_05.png)

    图 19.5：Kibana 按命名空间和容器分析日志记录

    在这里，我们可以找到我们的微服务的日志记录。大多数日志记录来自`review`和`recommendation`微服务。`product`和`product-composite`微服务可以在饼图的**其他**部分找到。

1.  通过在仪表板中保存此饼图来总结如何分析我们收集的日志记录类型。点击右上角的**保存**按钮。

1.  在名为**保存透镜可视化**的页面上，执行以下操作：

    1.  给它一个**标题**，例如，`hands-on-visualization`。

    1.  输入一个**描述**，例如，`这是我在 Kibana 中的第一个可视化`。

    1.  在**添加到仪表板**框中，选择**新建**。页面应该看起来像这样：

![计算机屏幕截图  描述由中等置信度自动生成](img/B19825_19_06.png)

图 19.6：在 Kibana 中创建仪表板

1.  点击名为**保存并转到仪表板**的按钮。应该会呈现以下仪表板：

![计算机屏幕截图  描述由中等置信度自动生成](img/B19825_19_07.png)

图 19.7：Kibana 中的新仪表板

1.  点击右上角的**保存**按钮，给仪表板起一个名字，例如，`hands-on-dashboard`，然后点击**保存**按钮。

您现在可以通过从汉堡菜单中选择**仪表板**始终返回此仪表板。

Kibana 包含大量用于分析日志记录的功能——请随意尝试。为了获得灵感，请参阅[`www.elastic.co/guide/en/kibana/7.17/dashboard.html`](https://www.elastic.co/guide/en/kibana/7.17/dashboard.html)。现在，我们将继续前进，开始从我们的微服务中定位实际的日志记录。

## 发现微服务的日志记录

在本节中，我们将学习如何利用集中式日志记录的主要功能之一，即从我们的微服务中查找日志记录。我们还将学习如何使用日志记录中的跟踪 ID 来查找属于同一进程的其他微服务的日志记录，例如，处理发送到公共 API 的外部请求。

让我们先创建一些日志记录，我们可以借助 Kibana 来查找它们。我们将使用 API 创建一个具有唯一产品 ID 的产品，然后检索有关该产品的信息。之后，我们可以尝试查找在检索产品信息时创建的日志记录。

微服务中的日志记录创建与上一章略有不同，以便 `product-composite` 和三个核心微服务 `product`、`recommendation` 和 `review` 在开始处理 `get` 请求时都记录一个日志级别设置为 `INFO` 的日志记录。让我们回顾每个微服务中添加的源代码：

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
    LOG.info("Will get recommendations for product with id={}", productId); 
    ```

+   审查微服务日志创建：

    ```java
    LOG.info("Will get reviews for product with id={}", productId); 
    ```

更多详细信息，请参阅 `microservices` 文件夹中的源代码。

执行以下步骤以使用 API 创建日志记录，然后使用 Kibana 查找日志记录：

1.  使用以下命令获取访问令牌：

    ```java
    ACCESS_TOKEN=$(curl -k https://writer:secret-writer@minikube.me/oauth2/token -d grant_type=client_credentials -d scope="product:read product:write" -s | jq .access_token -r)
    echo ACCESS_TOKEN=$ACCESS_TOKEN 
    ```

1.  如本节介绍中所述，我们将首先创建一个具有唯一产品 ID 的产品。通过执行以下命令创建一个最小化产品（没有推荐和评论）为 `"productId" :1234`：

    ```java
    curl -X POST -k https://minikube.me/product-composite \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer $ACCESS_TOKEN" \
      --data '{"productId":1234,"name":"product name 1234","weight":1234}' 
    ```

    使用以下命令读取产品：

    ```java
    curl -H "Authorization: Bearer $ACCESS_TOKEN" -k 'https://minikube.me/product-composite/1234' -s | jq . 
    ```

    预期响应类似于以下内容：

    ![计算机程序屏幕截图  描述自动生成，置信度中等](img/B19825_19_08.png)

    图 19.8：查找 productId = 1234 的产品

    希望我们能通过这些 API 调用创建一些日志记录。让我们跳转到 Kibana 并查看一下！

1.  在 Kibana 网页上，从汉堡菜单点击 **Discover**。你会看到如下内容：![计算机屏幕截图  描述自动生成，置信度中等](img/B19825_19_09.png)

    图 19.9：Kibana 网页 UI 及其主要部分

    在左上角，我们可以看到 Kibana 找到了 **5,350** 条日志记录。时间选择器显示它们来自 **过去 15 分钟**。在直方图中，我们可以看到日志记录随时间分布的情况。直方图下方是一个表格，显示了查询找到的最新的日志事件。

1.  如果你想更改时间间隔，可以使用时间选择器。点击其日历图标以调整时间间隔。

1.  为了更好地查看日志记录中的内容，将一些日志记录字段添加到直方图下方的表格中的列。

1.  要查看所有可用字段，点击 **按类型筛选** 标签右侧的向下箭头，并取消选择 **隐藏空字段**。

1.  从左侧的 **可用字段** 列表中选择字段。向下滚动直到找到该字段。为了更容易地找到字段，使用名为 **搜索字段名称** 的字段来过滤可用字段列表。

    将光标悬停在字段上，将出现一个 **+** 按钮（蓝色圆圈中的白色十字）；点击它将字段添加为表格中的列。按以下顺序选择以下字段：

    1.  `spring.level`，日志级别

    1.  `kubernetes.namespace_name`，Kubernetes 命名空间

    1.  `kubernetes.container_name`，容器的名称

    1.  `spring.trace`，用于分布式跟踪的跟踪 ID

    1.  `log`，实际的日志消息

    为了节省空间，你可以通过点击索引模式字段（包含文本`logstash-*`）旁边的折叠图标来隐藏字段列表。

    网页应该看起来像以下这样：

    ![计算机屏幕截图  描述自动生成，置信度中等](img/B19825_19_10.png)

    图 19.10：Kibana 网页 UI 显示日志记录

    现在表格中包含了一些关于日志记录的有用信息！

1.  要找到来自`GET` API 调用的日志记录，我们可以要求 Kibana 找到日志字段包含文本`product.id=1234`的日志记录。这与之前显示的`product-composite`微服务的日志输出相匹配。

    这可以通过在左上角的**搜索**字段中输入`log:"product.id=1234"`并点击**更新**按钮（此按钮也可以命名为**刷新**）来完成。预期会找到一条日志记录：

    ![计算机屏幕截图  描述自动生成，置信度中等](img/B19825_19_11.png)

    图 19.11：Kibana 网页 UI 显示 productId = 1234 的日志记录

1.  确认时间戳是你调用`GET` API 时的时间，并确认创建日志记录的容器名称是`product-composite`，也就是说，确认日志记录是由产品组合微服务发送的。

1.  现在，我们想看到参与返回产品 ID 为`1234`的产品信息过程的其它微服务的相关日志记录。换句话说，我们想找到与我们所找到的日志记录具有相同**跟踪 ID**的日志记录。

    要做到这一点，将光标移至日志记录的`spring.trace`字段。字段右侧将显示两个小放大镜，一个带有**+**符号，一个带有**-**符号。点击带有**+**符号的放大镜以过滤跟踪 ID。

1.  清除**搜索**字段，以便唯一的搜索条件是跟踪字段的过滤器。然后，点击**更新**按钮查看结果。预期会得到以下类似的响应：![计算机屏幕截图  描述自动生成，置信度中等](img/B19825_19_12.png)

    图 19.12：Kibana 网页 UI 显示跟踪 ID 的日志记录

    我们可以看到一些详细的调试消息，它们使视图变得杂乱；让我们去掉它们！

1.  将光标移至**DEBUG**值上，并点击带有**–**符号的放大镜以过滤出日志级别设置为**DEBUG**的日志记录。

1.  现在我们应该能够看到四个预期的日志记录，每个对应于查找产品 ID 为`1234`的产品信息时涉及的每个微服务：

![计算机屏幕截图  描述自动生成，置信度中等](img/B19825_19_13.png)

图 19.13：Kibana 网页 UI 显示日志级别为 INFO 的跟踪 ID 的日志记录

此外，请注意，应用的过滤器包括跟踪 ID，但排除了日志级别设置为**DEBUG**的日志记录。

现在我们知道了如何找到预期的日志记录，我们就可以进行下一步了。这将是学习如何找到意外的日志记录，即错误信息，以及如何进行根本原因分析以找到这些错误信息的原因。

## 进行根本原因分析

集中日志最重要的特性之一是它使得使用来自许多来源的日志记录来分析错误成为可能，并且基于这些分析进行根本原因分析，找出错误信息的真正原因。

在本节中，我们将模拟一个错误，并查看我们如何能够找到关于它的所有信息，直到找到在系统景观中某个微服务中引起错误的源代码行。为了模拟错误，我们将重用我们在第十三章的*使用 Resilience4j 提高弹性*部分中引入的故障参数，在*添加可编程延迟和随机错误*部分中使用它来强制`product`微服务抛出异常。执行以下步骤：

1.  运行以下命令在搜索产品 ID 为`1234`的产品信息时在`product`微服务中生成故障：

    ```java
    curl -H "Authorization: Bearer $ACCESS_TOKEN" -k https://minikube.me/product-composite/1234?faultPercent=100 -s | jq . 
    ```

    预期以下错误响应：

    ![包含文本、截图、软件、多媒体的图片，自动生成描述](img/B19825_19_14.png)

    图 19.14：一个在处理中引起错误的请求

    现在，我们必须假装我们对这个错误的原因一无所知！否则，根本原因分析就不会那么令人兴奋，对吧？

    假设我们在一个支持组织中工作，并被要求调查一个问题，即当最终用户尝试查找产品 ID 为`1234`的产品信息时，却收到了显示“`500 Internal Server Error`”错误信息的响应。

1.  在我们开始分析问题之前，让我们在 Kibana web UI 中删除之前的搜索过滤器，以便我们可以从头开始。对于我们在上一节中定义的每个过滤器，点击其关闭图标（一个**x**）来移除它。

1.  首先，使用时间选择器选择一个包含问题发生时间点的时段。在我的情况下，15 分钟就足够了。

1.  选择属于我们的命名空间`hands-on`的日志记录。可以通过以下步骤完成：

    1.  通过点击左上角的汉堡图标（**≡**）来展开左侧的字段列表。

    1.  点击列表中的**kubernetes.namespace_name**字段。显示的是前五个命名空间列表。

    1.  点击**hands-on**命名空间后面的**+**号。

1.  接下来，搜索在此时间范围内设置日志级别为**WARN**的日志记录，其中日志消息提到了产品 ID `1234`。这可以通过点击所选字段列表中的`spring.level`字段来完成。当你点击这个字段时，其最常用的值将显示在其下方。通过点击其**+**号来过滤**WARN**值。Kibana 现在将显示在所选时间范围内设置日志级别为**WARN**的日志记录，来自**hands-on**命名空间，如下所示：![计算机的截图  描述自动生成，置信度中等](img/B19825_19_15.png)

    图 19.15：Kiali 网页 UI，显示报告 ERRORs 的日志记录

    我们可以看到与产品 ID `1234`相关的多个错误消息。顶部日志条目具有相同的跟踪 ID，因此这似乎是一个值得进一步调查的跟踪 ID。第一条日志条目还包含最终用户报告的文本**500**和**Internal Server Error**，以及错误消息**Something went wrong…**，这可能与错误的根本原因有关。

1.  在上一节中，我们按照同样的方式过滤了第一条日志记录的跟踪 ID。

1.  移除`WARN`日志级别的过滤器，以便能够看到属于此跟踪 ID 的所有记录。预期 Kibana 将响应大量类似以下的日志记录：![计算机的截图  描述自动生成，置信度中等](img/B19825_19_16.png)

    图 19.16：Kiali 网页 UI，寻找根本原因

    很遗憾，我们无法通过使用跟踪 ID 来找到识别根本原因的堆栈跟踪。这是由于我们用于收集多行异常的 Fluentd 插件`fluent-plugin-detect-exceptions`的限制。它无法将堆栈跟踪与所使用的跟踪 ID 相关联。相反，我们可以使用 Kibana 中的功能来查找在特定日志记录附近发生的时间相近的日志记录。

1.  使用日志记录左侧的箭头展开显示**Error body: {… status”:500,”error”:”Internal Server Error”,”message”:”Something went wrong...”…}**的日志记录。关于此特定日志记录的详细信息将被揭示：

![计算机的截图  描述自动生成，置信度中等](img/B19825_19_17.png)

图 19.17：Kiali 网页 UI，展开带有根本原因日志消息的日志记录

1.  还有一个名为**查看周围文档**的链接；点击它以查看附近的日志记录。滚动到页面底部以找到可以指定记录数量的**加载**字段。将默认值从 5 增加到 10。预期网页如下所示：

![计算机的截图  描述自动生成，置信度中等](img/B19825_19_18.png)

图 19.18：Kiali 网页 UI，找到了根本原因

1.  在展开的日志记录下方第三条日志记录包含了错误消息**出了点问题...**的堆栈跟踪。这个错误消息看起来很有趣。它是在展开的日志记录前五毫秒由`product`微服务记录的。它们似乎有关联！该日志记录中的堆栈跟踪指向`ProductServiceImpl.java`中的第 104 行。查看源代码（见`microservices/product-service/src/main/java/se/magnus/microservices/core/product/services/ProductServiceImpl.java`），第 104 行如下：

    ```java
    throw new RuntimeException("Something went wrong..."); 
    ```

    这是错误的根本原因。我们事先确实知道这一点，但现在我们也看到了如何导航到它。

在这种情况下，问题非常简单就能解决；只需在 API 请求中省略`faultPercent`参数即可。在其他情况下，确定根本原因可能要困难得多！

1.  这就结束了根本原因分析。点击网页浏览器中的后退按钮返回主页。

1.  为了能够重用搜索条件和表格布局的配置，可以通过 Kibana 保存其定义。例如，选择对`hands-on`命名空间中的日志记录进行过滤，然后在右上角的菜单中点击**保存**链接。为搜索定义命名并点击**保存**按钮。当需要时，可以使用菜单中的**打开**链接恢复搜索定义。

这就结束了关于使用 EFK 栈进行集中日志记录的章节。

# 摘要

在本章中，我们学习了在系统视图中从微服务收集日志记录并将其放入一个公共集中数据库的重要性，在那里可以对存储的日志记录进行分析和搜索。我们使用了 EFK 栈，包括 Elasticsearch、Fluentd 和 Kibana，来收集、处理、存储、分析和搜索日志记录。

Fluentd 不仅用于从我们的微服务中收集日志记录，还用于 Kubernetes 集群中的各种支持容器。Elasticsearch 被用作文本搜索引擎。与 Kibana 一起，我们看到了理解我们收集了哪些类型日志记录是多么容易。

我们还学习了如何使用 Kibana 执行重要任务，例如从协作微服务中查找相关日志记录以及如何进行根本原因分析，找到错误消息的真实问题。

能够以这种方式收集和分析日志记录在生产环境中是一个重要的能力，但这些类型的活动总是在收集日志记录之后才进行。另一个重要的能力是能够监控微服务的当前健康状况，从硬件资源的使用、响应时间等方面收集和可视化运行时指标。我们在上一章中提到了这个主题，在下一章中，我们将学习更多关于监控微服务的内容。

# 问题

1.  用户使用以下截图所示的搜索条件在`hands-on`命名空间中搜索了最后 30 天的`ERROR`日志消息，但没有找到任何结果。为什么？

![计算机屏幕截图  描述自动生成，置信度中等](img/B19825_19_19.png)

图 19.19：Kiali 网页界面，未显示预期的日志记录

1.  用户发现了一条有趣的日志记录（如下所示）。用户如何从这条和其他微服务中找到相关的日志记录，例如，来自处理外部 API 请求的微服务？

![计算机屏幕截图  描述自动生成，置信度中等](img/B19825_19_20.png)

图 19.20：Kiali 网页界面；我们如何找到相关的日志记录？

1.  用户发现了一条日志记录，似乎表明了由最终用户报告的问题的根本原因。用户如何找到显示错误发生位置的源代码中的堆栈跟踪？

![计算机屏幕截图  描述自动生成，置信度低](img/B19825_19_21.png)

图 19.21：Kiali 网页界面；我们如何找到根本原因？

1.  为什么以下 Fluentd 配置元素不起作用？

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

1.  你如何确定 Elasticsearch 是否正在运行？

1.  你突然从网络浏览器中失去了与 Kibana 的连接。可能是什么原因导致了这个问题？
