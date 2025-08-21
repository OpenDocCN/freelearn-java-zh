# 第二十章：监控微服务

在本章中，我们将学习如何使用 Prometheus 和 Grafana 来收集、监控和关于性能指标的警报。正如我们在第一章中提到的*微服务简介*，在*集中监控和警报*部分，在生产环境中，能够收集应用程序性能和硬件资源使用情况的数据至关重要。监控这些指标是为了避免 API 请求和其他进程的响应时间过长或出现故障。

为了能够以成本效益高和主动的方式监控微服务系统架构，我们需要定义一些自动触发的警报，如果指标超过配置的限制就会触发这些警报。

在本章中，我们将涵盖以下主题：

+   使用 Prometheus 和 Grafana 进行性能监控简介

+   源代码中收集应用程序指标的变化

+   构建和部署微服务

+   使用 Grafana 仪表板监控微服务

+   在 Grafana 中设置警报

# 技术要求

本书中描述的所有命令都已经在使用 macOS Mojave 的 MacBook Pro 上运行过，但应该很容易进行修改，以便它们可以在其他平台上运行，例如 Linux 或 Windows。

本章的源代码可以在本书的 GitHub 仓库中找到：[`github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud/tree/master/Chapter20`](https://github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud/tree/master/Chapter20)。

为了能够运行本书中描述的命令，你需要将源代码下载到一个文件夹中，并设置一个环境变量`$BOOK_HOME`，使其指向该文件夹。一些示例命令如下：

```java
export BOOK_HOME=~/Documents/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud
git clone https://github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud $BOOK_HOME
cd $BOOK_HOME/Chapter20
```

本章中的所有源代码示例均来自`$BOOK_HOME/Chapter20`的源代码，并且已经使用 Kubernetes 1.15 进行了测试。

如果你想要查看我们为本章源代码所做的更改，以便你可以使用 Prometheus 和 Grafana 监控一个性能指标的警报，你可以将它们与第十九章的源代码*使用 EFK 堆栈进行集中日志记录*进行比较。你可以使用你最喜欢的区分工具，比较两个文件夹`$BOOK_HOME/Chapter19`和`$BOOK_HOME/Chapter20`。

# 使用 Prometheus 和 Grafana 进行性能监控简介

在本章中，我们将重用我们在第十八章 *使用服务网格提高可观测性和管理*中创建的 Prometheus 和 Grafana 部署，在*在 Kubernetes 集群中部署 Istio*部分。在那一章中，在*介绍 Istio 的运行时组件*部分，我们简要介绍了 Prometheus，这是一个流行的开源时间序列数据库，如性能指标。我们还介绍了 Grafana 作为一个开源工具，用于可视化性能指标。Istio 的可观测性控制台 Kiali 与 Grafana 集成。用户可以从 Kiali 中的感兴趣服务导航到其在 Grafana 中的相应性能指标。Kiali 还可以在不使用 Grafana 的情况下渲染一些与性能相关的图表。在本章中，我们将通过使用这些工具一起获得与这种集成相关的实践经验。

我们在第十八章 *使用服务网格提高可观测性和管理*中部署的 Istio 配置包括一个 Prometheus 配置，它自动从 Kubernetes 中的 pods 收集指标。我们只需要在我们的微服务中设置一个端点，以产生 Prometheus 可以消费的指标格式。我们还需要向 Kubernetes pod 添加注解，以便 Prometheus 可以找到端点的地址。有关如何设置的详细信息，请参见本章的*收集应用程序指标的源代码变化*部分。

以下图表说明了刚刚讨论的运行时组件之间的关系：

![](img/53cbe67e-e1ec-419c-8113-92bb9bd15231.png)

在这里，我们可以看到 Prometheus 如何使用 Kubernetes pod 定义中的注解来能够从我们的微服务中收集指标。然后，它将这些指标存储在其数据库中。用户可以通过访问 Kiali 和 Grafana 的 web UI 来监控这些指标，**网页浏览器**。**网页浏览器**使用在第十八章 *使用服务网格提高可观测性和管理*中介绍的**minikube 隧道**，访问 Kiali 和 Grafana。

请记住，第十八章 *使用服务网格提高可观测性和管理*中用于部署 Istio 的配置仅适用于开发和测试，而不是生产。例如，存储在 Prometheus 数据库中的性能指标在 Prometheus pod 重新启动后将不会保留！

在下一节中，我们将查看对源代码进行了哪些更改，以使微服务产生 Prometheus 可以收集的性能指标。

# 源代码中收集应用程序指标的变化

Spring Boot 2 支持使用 Micrometer 库产生性能指标，并以普罗米修斯格式呈现（[`micrometer.io`](https://micrometer.io)）。我们只需要对源代码做一次更改：需要在每个微服务的 Gradle 构建文件`build.gradle`中添加对 Micrometer 库的依赖`micrometer-registry-prometheus`。在此，已添加以下依赖项：

```java
implementation("io.micrometer:micrometer-registry-prometheus")
```

这将使得微服务在端口`4004`上使用`"/actuator/prometheus"` URI 产生普罗米修斯指标。为了让普罗米修斯了解这些端点，每个微服务的容器都用以下代码进行了注释：

```java
annotations:
  prometheus.io/scrape: "true"
  prometheus.io/port: "4004"
  prometheus.io/scheme: http
  prometheus.io/path: "/actuator/prometheus"
```

查看`kubernetes/services/base/deployments`文件夹中的部署定义以获取更多详细信息。

为了在普罗米修斯收集完指标后更容易识别指标的来源，指标被标记为产生该指标的微服务名称。这通过在共同配置文件`config-repo/application.yml`中添加以下配置来实现：

```java
management.metrics.tags.application: ${spring.application.name}
```

这将导致每个产生的指标都有一个额外的标签，名为`application`。它将包含微服务名称的标准 Spring 属性值，即`spring.application.name`。

这就是准备微服务产生性能指标并使普罗米修斯意识到要使用哪些端点来开始收集这些指标所需的所有更改。在下一节中，我们将构建和部署微服务。

# 构建和部署微服务

使用`test-em-all.bash`测试脚本构建、部署并验证部署的方式与在*构建和部署微服务*章节中的第十九章、*使用 EFK 堆栈进行集中日志记录*相同。运行以下命令：

1.  从源代码构建 Docker 镜像，使用以下命令：

```java
cd $BOOK_HOME/Chapter20
eval $(minikube docker-env)
./gradlew build && docker-compose build
```

1.  重新创建命名空间`hands-on`，并将其设置为默认命名空间：

```java
kubectl delete namespace hands-on
kubectl create namespace hands-on
kubectl config set-context $(kubectl config current-context) --namespace=hands-on 
```

1.  通过运行以下命令执行`deploy-dev-env.bash`脚本：

```java
./kubernetes/scripts/deploy-dev-env.bash 
```

1.  如果 Minikube 隧道尚未运行，按照以下方式启动（如果需要，请回顾第十八章、*使用服务网格提高可观测性和管理*、*设置对 Istio 服务的访问*部分）：

```java
minikube tunnel
```

1.  使用以下命令验证部署的正常测试：

```java
./test-em-all.bash
```

预期输出将与前几章中看到的内容类似：

![](img/c0e56736-a6d9-4bad-9624-d1daf3195629.png)

微服务部署完成后，我们可以开始使用 Grafana 监控我们的微服务！

# 使用 Grafana 仪表板监控微服务

正如我们在介绍中提到的，Kiali 与 Grafana 集成，并提供一些非常实用的仪表板。一般来说，它们关注的是每秒请求数、响应时间和处理请求的故障百分比等应用级性能指标。正如我们即将看到的，它们在应用层面非常有用。但如果我们想要了解底层硬件资源的使用情况，我们需要更详细的指标，例如 Java VM 相关指标。

Grafana 有一个活跃的社区，社区成员会分享可重用的仪表板。我们将尝试使用社区提供的仪表板，该仪表板专为从像我们的微服务这样的 Spring Boot 2 应用程序获取许多有价值的 Java VM 相关指标而设计。最后，我们将了解如何在 Grafana 中创建我们自己的仪表板。但首先，让我们探索一下 Kiali 和 Grafana 之间的集成。

在这样做之前，我们需要做两项准备工作：

1.  为测试安装一个本地邮件服务器并配置 Grafana，使其能够向其发送电子邮件。

    我们将在“在 Grafana 中设置警报”一节中使用邮件服务器。

1.  为了能够监视一些指标，我们将启动在前几章中使用的负载测试工具。

# 为测试安装一个本地邮件服务器

在本节中，我们将搭建一个本地测试邮件服务器，并配置 Grafana 以将警报电子邮件发送到该邮件服务器。

Grafana 可以向任何 SMPT 邮件服务器发送电子邮件，但为了保持测试本地化，我们将部署一个名为`maildev`的测试邮件服务器。考虑以下步骤：

1.  使用以下命令安装测试邮件服务器：

```java
kubectl create deployment mail-server --image djfarrelly/maildev:1.1.0
kubectl expose deployment mail-server --port=80,25 --type=ClusterIP
kubectl wait --timeout=60s --for=condition=ready pod -l app=mail-server
```

1.  通过访问[`mail-server.hands-on.svc.cluster.local`](http://mail-server.hands-on.svc.cluster.local)上的网页来验证测试邮件服务器是否正在运行。期待显示出如下类似的网页：

![](img/c3dc6129-9eed-4130-af97-33e94b21a429.png)

1.  通过设置一些环境变量配置 Grafana 以向测试邮件服务器发送电子邮件。运行以下命令：

```java
kubectl -n istio-system set env deployment/grafana \
 GF_SMTP_ENABLED=true \
 GF_SMTP_SKIP_VERIFY=true \
 GF_SMTP_HOST=mail-server.hands-on.svc.cluster.local:25 \
 GF_SMTP_FROM_ADDRESS=grafana@minikube.me
kubectl -n istio-system wait --timeout=60s --for=condition=ready pod -l app=grafana
```

有关详细信息，请参阅[`hub.docker.com/r/djfarrelly/maildev`](https://hub.docker.com/r/djfarrelly/maildev)。

现在，我们已经运行了一个测试邮件服务器，并且 Grafana 已经配置为向其发送电子邮件。在下一节中，我们将启动负载测试工具。

# 启动负载测试

为了监视一些指标，让我们使用在前几章中使用的 Siege 启动负载测试。运行以下命令：

```java
ACCESS_TOKEN=$(curl -k https://writer:secret@minikube.me/oauth/token -d grant_type=password -d username=magnus -d password=password -s | jq .access_token -r)
siege https://minikube.me/product-composite/2 -H "Authorization: Bearer $ACCESS_TOKEN" -c1 -d1
```

现在，我们准备学习 Kiali 和 Grafana 之间的集成，并探索随 Istio 提供的 Grafana 仪表板。

# 使用 Kiali 内置的 Grafana 仪表板

在第十八章“使用服务网格提高可观测性和管理”的“观察服务网格”一节中，我们了解了 Kiali，但我们跳过了 Kiali 显示性能指标的部分。现在，是时候回到这个主题了！

执行以下步骤了解 Kiali 与 Grafana 的集成：

1.  使用 `http://kiali.istio-system.svc.cluster.local:20001` URL 在 web 浏览器中打开 Kiali web UI。如果需要，使用 `admin`/`admin` 登录。

1.  通过点击左侧菜单中的服务标签，进入服务页面。

1.  通过点击它，选择产品服务页面。

1.  在服务：产品页面上，选择入站指标标签。您将看到以下页面：

![](img/457f3485-6af5-4f6f-a6ef-c7b0adf28a24.png)

1.  Kiali 将可视化一些整体性能图表。然而，在 Grafana 中还有更多详细的性能指标。点击“在 Grafana 中查看”链接，Grafana 将在新标签页中打开。期待一个类似于以下的网页：

![](img/e4795b3f-5a78-41f3-8401-5f3de515cd62.png)

1.  这里展示了大量应用级别的性能指标，例如 HTTP 请求率、响应时间和错误率。这些指标是针对选中的 Kiali 中的 Product 服务的。在页面上方左侧点击服务下拉菜单，选择另一个服务。随意查看！

Istio 带有一组预安装的 Grafana 仪表板；点击 Istio/Istio 服务仪表板查看可用的仪表板列表。现在，选择 Istio 网格仪表板。您将看到一个类似于以下内容的网页：

![](img/8d9b08eb-3211-45ee-a1b1-04e82a48aeaa.png)

这个仪表板能很好地概述服务网格中的微服务以及它们每秒请求数、响应时间和成功率等当前状态。

正如我们之前提到的，Istio 仪表板能提供很好的应用级别概述。但是，也需要监控每个微服务的硬件使用指标。在下一节，我们将学习如何导入现有的仪表板——具体来说，一个显示基于 Spring Boot 2 的应用程序的 Java VM 指标的仪表板。

# 导入现有的 Grafana 仪表板

正如我们之前提到的，Grafana 有一个活跃的社区，他们分享可重用的仪表板。它们可以在 [`grafana.com/grafana/dashboards`](https://grafana.com/grafana/dashboards) 上找到。我们将尝试一个名为 **JVM** (**Micrometer**) 的仪表板，它是为从 Spring Boot 2 应用程序获取许多有价值的 JVM 相关指标而量身定制的。仪表板的 URL 是 [`grafana.com/grafana/dashboards/4701`](https://grafana.com/grafana/dashboards/4701)。在 Grafana 中导入仪表板非常容易。按照以下步骤导入这个仪表板：

1.  按照这些步骤导入名为 JVM (Micrometer) 的仪表板：

    1.  在 Grafana 网页上，点击左侧菜单中的 + 符号，然后选择导入。

    1.  在导入页面，将仪表板 ID `4701` 粘贴到 Grafana.com 仪表板字段中，并按 Tab 键离开该字段。

    1.  在将显示的导入页面中，点击 Prometheus 下拉菜单，选择 Prometheus。

    1.  现在，点击“导入”按钮，JVM（Micrometer）仪表板将被导入并显示。

1.  通过以下步骤检查 JVM（Micrometer）仪表板：

    1.  为了获得良好的指标视图，点击页面右上角的时钟选择器。这将允许您选择一个适当的时间间隔：

        1.  选择最后 5 分钟作为范围。再次点击时钟选择器，将刷新率设置为 5 秒。

        1.  在指定刷新率后点击“应用”按钮。

    1.  在页面左上角的“应用”下拉菜单中，选择 product-composite 微服务。

+   1.  由于我们在后台使用 Siege 进行负载测试，我们将看到很多指标。以下是的一个示例屏幕截图：

![](img/21a2ab89-07a0-4a0a-9836-4ddb9fbac91a.png)

在这个仪表板中，我们可以找到所有类型的与 Java VM 相关的指标，例如 CPU、内存和 I/O 使用情况，以及与 HTTP 相关的指标，如每秒请求数、平均持续时间和错误率。请随意自行探索这些指标！

能够导入现有的仪表板在我们想要快速开始时非常有价值。然而，更重要的是要知道如何创建自己的仪表板。我们将在下一节学习这一点。

# 开发自己的 Grafana 仪表板

开始开发 Grafana 仪表板是很直接的。我们需要理解的是 Prometheus 为我们提供了哪些指标。

在本节中，我们将学习如何检查可用的指标。基于这些指标，我们将创建一个仪表板，用于监控一些更有趣的指标。

# 检查 Prometheus 指标

在*源代码中收集应用指标的变化*一节中，我们配置了 Prometheus 从我们的微服务中收集指标。我们实际上可以调用相同的端点并查看 Prometheus 收集的指标。运行以下命令：

```java
curl http://product-composite.hands-on.svc.cluster.local:4004/actuator/prometheus -s
```

预期命令会有很多输出，如下例所示：

![](img/6ca8a2ed-3cb0-4233-84eb-18c1ba2f8f0b.png)

在所有报告的指标中，有两个非常有趣的：

+   `resilience4j_retry_calls`：Resilience4j 报告其重试机制的运行情况。它成功和失败的请求，以及是否有重试，各报告了四个不同的值。

+   `resilience4j_circuitbreaker_state`：Resilience4j 报告电路断路器的状态。

请注意，这些指标有一个名为`application`的标签，包含微服务名称。这个字段来自我们在*源代码中收集应用指标的变化*一节中配置的`management.metrics.tags.application`属性。

这些指标看起来很有趣，值得我们监控。我们迄今为止使用的所有仪表板都没有使用来自 Resilience4j 的指标。在下一节中，我们将为这些指标创建一个仪表板。

# 创建仪表板

在本节中，我们将学习如何创建一个可视化我们在上一节中描述的 Resilience4j 指标的仪表板。

我们将在以下子节中设置仪表板：

+   创建一个空白的仪表板

+   为电路断路器指标创建一个新的面板

+   为重试指标创建一个新的面板

+   排列面板

# 创建一个空白的仪表板

执行以下步骤以创建一个空白的仪表板：

1.  在 Grafana 网页上，点击左侧菜单中的+号，然后选择仪表板。

1.  将显示一个名为“新仪表板”的网页：

![](img/177f95ee-296f-4b8e-9877-a27a1d127388.png)

1.  点击仪表板设置按钮（它的图标是一个齿轮），如前一个屏幕截图所示。然后，按照以下步骤操作：

    1.  在“名称”字段中指定仪表板名称，并将其设置为“动手实践仪表板”。

    1.  点击“保存”按钮。

1.  点击时间选择器以选择仪表板的默认值，如下所示：

    1.  选择“最后 5 分钟”作为时间范围。

    1.  再次点击时间选择器，并在面板底部的“每 5 秒刷新”字段中指定 5 秒作为刷新率。

    1.  在指定刷新率后点击“应用”按钮。

    1.  从页面顶部的菜单中点击“保存”按钮。

    1.  启用“保存当前时间范围”功能，并在“保存更改”对话框中点击“保存”按钮。

# 为电路断路器指标创建一个新的面板

执行以下步骤以创建用于电路断路器指标的新面板：

1.  点击页面顶部左侧的“添加面板”按钮（它的图标是一个带有+号的图表）。

1.  点击“添加查询”按钮。将显示一个页面，可以在其中配置新面板。

1.  在查询字段中，在“A 字母”下指定电路断路器指标的名称，即“resilience4j_circuitbreaker_state”。

1.  在“图例”字段中指定格式，即`{{application}}.{{namespace}}`。这将创建一个图例，在面板中为涉及的微服务标记其名称和命名空间。

1.  填写的值应如下所示：

![](img/e086766e-4246-4776-a512-e23c79ef4d16.png)

1.  从左侧菜单中点击第三个标签，名为“通用”，并将“标题”字段设置为“电路断路器”。

1.  点击页面左上角的返回按钮以返回到仪表板。

# 为重试指标创建一个新的面板

在这里，我们将重复我们为先前电路断路器指标添加面板的相同步骤，但这次我们将指定重试指标的值：

1.  在查询字段中指定`rate(resilience4j_retry_calls[30s])`。由于重试指标是一个计数器，其值只会增加。不断增加的指标对于监控来说相当无趣。**rate**函数用于将重试指标转换为每秒的速率指标。指定的时间窗口，即 30 秒，是 rate 函数用来计算速率平均值的时间窗口。

1.  对于图例，指定`{{application}}.{{namespace}} ({{kind}})`。就像前面 Prometheus 端点的输出一样，我们将为重试机制获得四个指标。为了在图例中区分它们，需要添加`kind`属性。

1.  请注意，Grafana 立即开始在面板编辑器中根据指定值渲染图表。

1.  将“重试”指定为标题。

1.  点击返回按钮返回到仪表板。

# 安排面板

执行以下步骤以在仪表板上安排面板：

1.  您可以通过拖动其右下角来调整面板的大小到所需大小。

1.  您还可以通过拖动其标题将面板移动到所需位置。

1.  以下是对两个面板的一个示例布局：

![](img/b77f5f50-990c-482e-afcc-126be94bac9b.png)

1.  最后，点击页面顶部的“保存”按钮。将显示“保存更改”对话框；输入可选描述并点击保存按钮。

创建仪表板后，我们准备尝试它：在下一节中，我们将尝试两者指标。

# 尝试新的仪表板

在我们开始测试新仪表板之前，我们必须停止负载测试工具 Siege。为此，转到 Siege 正在运行的命令窗口并按*Ctrl + C*停止它。

首先，我们通过测试如何监视断路器开始。之后，我们将尝试重试指标。

# 测试断路器指标

如果我们强制打开断路器，它的状态将从一个**关闭**改变为**打开**，然后最终变为**半开**状态。这应该在断路器面板中报告。

打开电路，就像我们在第十三章《使用 Resilience4j 提高弹性》中的*尝试断路器和重试机制*部分所做的那样；也就是说，连续对 API 发起三次请求，全部失败。运行以下命令：

```java
ACCESS_TOKEN=$(curl -k https://writer:secret@minikube.me/oauth/token -d grant_type=password -d username=magnus -d password=password -s | jq .access_token -r)
for ((n=0; n<3; n++)); do curl -o /dev/null -skL -w "%{http_code}\n" https://minikube.me/product-composite/2?delay=3 -H "Authorization: Bearer $ACCESS_TOKEN" -s; done
```

我们可以预期会收到三个 500 响应，表示连续三次错误，也就是说，这是打开断路器所需要的一切！

在某些罕见的情况下，我注意到断路器指标没有在 Grafana 中报告。如果它们在一分钟后没有出现，简单地重新运行前面的命令重新打开断路器。

期望断路器指标的值上升到`1`，表示电路已打开。过了一会儿，它应该上升到`2`，表示电路现在是半开状态。这证明了我们可以在出现问题时监控断路器的打开。通过以下命令关闭电路断路器：

```java
for ((n=0; n<3; n++)); do curl -o /dev/null -skL -w "%{http_code}\n" https://minikube.me/product-composite/2?delay=0 -H "Authorization: Bearer $ACCESS_TOKEN" -s; done
```

我们将得到三个`200`作为响应。注意，断路器指标在仪表板中又回到了 0；也就是说，它是关闭的。

进行此测试后，Grafana 仪表板应如下所示：

![](img/1050575f-3831-4835-a706-bf25efa841f4.png)

从上面的屏幕截图中，我们可以看到重试机制也报告了成功和失败的指标。

既然我们已经看到了电路断路器指标的实际应用，让我们看看重试指标的实际应用！

# 测试重试指标

为了触发重试机制，我们将使用在前几章中使用的`faultPercentage`参数。为了避免触发电路断路器，我们需要为该参数使用相对较低的值。运行以下命令：

```java
while true; do curl -o /dev/null -s -L -w "%{http_code}\n" -H "Authorization: Bearer $ACCESS_TOKEN" -k https://minikube.me/product-composite/2?faultPercent=10; sleep 3; done
```

前一条命令将每三秒调用一次 API。它指定有 10%的请求应失败，以便重试机制启动并重试失败的请求。几分钟之后，仪表板应报告如下指标：

![](img/21697539-9aba-49a7-9102-1efeca3cd906.png)

在前一个屏幕截图中，我们可以看到大多数请求已经成功执行，没有进行重试。大约有 10%的请求被重试机制重试并成功执行。在继续下一节之前，记得停止我们为前一个重试测试启动的请求循环！

在下一节中，我们将学习如何在 Grafana 中根据这些指标设置警报。

# 在 Grafana 中设置警报

能够监控电路断路器和重试指标非常有价值，但更重要的是，能够根据这些指标定义自动化警报。自动化警报使我们不必手动监控指标。

Grafana 内置了对定义警报和向多个目的地发送通知的支持。在本节中，我们将为电路断路器定义警报，并配置 Grafana 在警报触发时向测试邮件服务器发送邮件。本地的测试邮件服务器在“为测试安装本地邮件服务器”一节中安装。

在下一节中，我们将定义一个基于邮件的通知渠道，该渠道将在这一节之后的警报中使用。

# 设置基于邮件的通知渠道

要在 Grafana 中配置基于邮件的通知渠道，请执行以下步骤：

1.  在 Grafana 网页上，在左侧菜单中，点击“警报”菜单选项（带有警铃图标的选项）并选择通知渠道。

1.  单击“添加频道”按钮。

1.  将名称设置为`mail`。

1.  选择类型为`Email`。

1.  启用所有警报发送。

1.  启用包含图片。

1.  输入你选择的任意电子邮件地址。邮件只会发送到本地的测试邮件服务器，无论指定的是哪个电子邮件地址。通知渠道的配置应如下所示：

![](img/868fa4cb-c3fb-4419-8640-e3e10655d322.png)

1.  点击“发送测试”按钮发送测试邮件。

1.  点击“保存”按钮。

1.  点击左侧菜单中的“仪表板”按钮，然后点击主页按钮。

1.  从列表中选择“动手仪表板”以返回仪表板。

1.  检查测试邮件服务器的网页，确保我们已经收到测试邮件。你应该收到以下输出：

![](img/81a70ae5-3443-432f-935e-0987ae320c54.png)

有了通知渠道后，我们就可以定义断路器上的警报了。

# 在断路器上设置警报。

要创建断路器的警报，我们需要先创建警报，然后将警报列表添加到仪表板中，我们可以在其中查看随着时间的推移发生了哪些警报事件。

执行以下步骤创建断路器的警报：

1.  在互动仪表板中，点击断路器面板的标题。一个下拉菜单将出现。

1.  选择编辑菜单选项。

1.  在左侧标签列表中选择警报标签（显示为警铃图标）。

1.  点击创建警报按钮。

1.  在“评估每”字段中，将值设置为`10s`。

1.  在“对于”字段中，将值设置为`0m`。

1.  在条件部分，指定以下值：

    +   将“WHEN”字段设置为`max()`。

    +   将“OF”字段设置为`query(A, 1m, now)`。

    +   将“IS ABOVE”字段设置为`0.5`。

1.  滚动到底部以确认通知已发送到默认通知渠道，即我们之前定义的邮件渠道。警报定义应如下所示：

![](img/42d25ca0-c993-40b4-8b9c-85487788edda.png)

1.  点击返回按钮（左箭头）回到仪表板。

然后，我们需要执行以下步骤来创建警报列表：

1.  点击页面顶部的添加面板按钮。

1.  在“新面板”页面中选择“选择可视化”。

1.  在提供的可视化中，选择“警报列表”。点击它两次以显示选项列表。

1.  选择显示名为“最近状态更改”的选项。

1.  启用从这个仪表板发出的警报。设置应该如下所示：

![](img/104910bd-eef6-4656-be83-50c72c884211.png)

1.  点击返回按钮以回到仪表板。

1.  重新排列面板以满足您的需求。

1.  保存仪表板更改。

以下是带有警报列表添加的示例布局：

![](img/901da6a7-bf7d-40d5-8ef3-ab3a19a31e89.png)

我们可以看到断路器报告指标为正常（带有绿色心形），并且警报列表包含电路断路器的正常事件。

现在，是尝试警报的时候了！

# 尝试断路器警报。

在这里，我们将重复“测试断路器指标”部分中的测试，但这次我们希望警报被触发并且还发送电子邮件！让我们开始吧：

1.  首先打开断路器：

```java
for ((n=0; n<3; n++)); do curl -o /dev/null -skL -w "%{http_code}\n" https://minikube.me/product-composite/2?delay=3 -H "Authorization: Bearer $ACCESS_TOKEN" -s; done
```

仪表板应该报告电路像以前一样打开。一分钟后，应该触发警报并且还会发送电子邮件。预期仪表板应该与下面的屏幕截图类似：

![](img/c59df830-87fe-4b2c-986a-42f792e063d6.png)

注意断路器面板标题中的警报图标（一个红色的心形）。红线标记警报事件的时间，并且已经将警报添加到警报列表中。

1.  在测试邮件服务器上，您应该看到一个电子邮件，如下面的屏幕截图所示：

![](img/80de707b-9f84-480d-8419-01872070ef0b.png)

1.  太好了；我们得到了警报，就像我们预期的那样！现在，关闭电路，用以下命令使问题消失：

```java
for ((n=0; n<3; n++)); do curl -o /dev/null -skL -w "%{http_code}\n" https://minikube.me/product-composite/2?delay=0 -H "Authorization: Bearer $ACCESS_TOKEN" -s; done
```

指标应该恢复到正常值，即`0`，一分钟之后，警报应该再次变为绿色。

期望仪表板看起来像下面的屏幕截图：

![](img/5c851716-2af5-4b61-a141-a7aa0c6f25ba.png)

请注意，电路 breaker 面板顶部图标是绿色的；绿色线条标志着 OK 事件的时间，并且已经在警报列表中添加了一个 OK 事件。

在测试邮件服务器上，你应该看到一封电子邮件，如下面的屏幕截图所示：

![](img/9adcc46b-e266-4566-8fe9-4c45566b7133.png)

这样我们就完成了使用 Prometheus 和 Grafana 监控微服务的方法。

# 总结

在本章中，我们学习了如何使用 Prometheus 和 Grafana 收集和监控性能指标的警报。

我们了解到，在 Kubernetes 环境中收集性能指标时，可以使用 Prometheus。接着，我们学习了当在 pods 定义中添加几个 Prometheus 注解时，Prometheus 如何能够自动从 pods 收集指标。为了在我们的微服务中产生指标，我们使用了 Micrometer。

然后，我们了解了如何使用 Grafana 仪表板来监控收集的指标。既有 Kiali 自带的仪表板，也有 Grafana 社区分享的仪表板。我们还学习了如何开发我们自己的仪表板，其中我们使用了来自 Resilience4j 的指标来监控其断路器和重试机制的使用。

最后，我们学习了如何在 Grafana 中定义指标警报以及如何使用 Grafana 发送警报通知。我们使用了一个本地的测试邮件服务器来接收 Grafana 的电子邮件警报。

我希望这本书能帮助你学习如何使用 Spring Boot、Spring Cloud、Kubernetes 和 Istio 的所有惊人特性来开发微服务，并鼓励你去尝试它们！

# 问题

1.  我们需要对微服务中的源代码进行哪些更改，以便它们产生被 Prometheus 消费的指标？

1.  `management.metrics.tags.application` 配置参数是用来做什么的？

1.  如果你想要分析一个关于高 CPU 消耗的支持案例，你会从本章中的哪个仪表板开始？

1.  如果你想要分析一个关于慢 API 响应的支持案例，你会从本章中的哪个仪表板开始？

1.  基于计数器类型的指标（如 Resilience4J 的重试指标）存在哪些问题，我们又能做些什么以使它们以有用的方式被监控？

1.  为什么电路 breaker 的指标在报告 2 之前暂时报告 1？请看下面的屏幕截图：

![](img/4ea782fb-939e-4629-8a5d-b0d2f181f8d1.png)

。
