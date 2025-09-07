

# 第二十章：微服务监控

在本章中，我们将学习如何使用 Prometheus 和 Grafana 收集、监控和警报性能指标。正如我们在*第一章*，*微服务简介*中提到的，在生产环境中，能够收集应用程序性能和硬件资源使用的指标至关重要。监控这些指标是避免 API 请求和其他过程出现长时间响应或中断的必要条件。

为了以成本效益和主动的方式监控微服务系统景观，我们还必须能够定义当指标超过配置的限制时自动触发的警报。

在本章中，我们将涵盖以下主题：

+   使用 Prometheus 和 Grafana 的性能监控简介

+   收集应用程序指标的源代码更改

+   构建和部署微服务

+   使用 Grafana 仪表板监控微服务

+   在 Grafana 中设置警报

# 技术要求

关于如何安装本书中使用的工具以及如何访问本书源代码的说明，请参阅：

+   *第二十一章*，*macOS 的安装说明*

+   *第二十二章*，*使用 WSL 2 和 Ubuntu 的 Microsoft Windows 安装说明*

本章中的代码示例均来自`$BOOK_HOME/Chapter19`的源代码。

如果您想查看本章源代码中应用的变化，以便使用 Prometheus 和 Grafana 监控和警报性能指标，您可以将其与*第十九章*，*使用 EFK 堆栈进行集中日志记录*的源代码进行比较。您可以使用您喜欢的 diff 工具比较两个文件夹，`$BOOK_HOME/Chapter19`和`$BOOK_HOME/Chapter20`。

# 使用 Prometheus 和 Grafana 的性能监控简介

在本章中，我们将重用我们在*第十八章*，*在 Kubernetes 集群中部署 Istio*部分中创建的 Prometheus 和 Grafana 的部署。在那章中，我们还简要介绍了 Prometheus，这是一个流行的开源数据库，用于收集和存储时间序列数据，如性能指标。我们还了解了 Grafana，这是一个开源工具，用于可视化性能指标。随着 Grafana 的部署，还附带了一套 Istio 特定的仪表板。Kiali 也可以在不使用 Grafana 的情况下渲染一些与性能相关的图表。在本章中，我们将获得一些使用这些工具的实践经验。

我们在*第十八章*中部署的 Istio 配置包括 Prometheus 的配置，该配置会自动从 Kubernetes 中的 Pod 收集指标。我们所需做的只是在我们微服务中设置一个端点，该端点以 Prometheus 可以消费的格式产生指标。我们还需要向 Kubernetes Pods 添加注释，以便 Prometheus 可以找到这些端点的地址。有关如何设置此配置的详细信息，请参阅本章的*源代码更改以收集应用程序指标*部分。为了展示 Grafana 提升警报的能力，我们还将部署一个本地邮件服务器。

以下图表说明了我们刚才讨论的运行时组件之间的关系：

![计算机屏幕截图  描述由低置信度自动生成](img/B19825_20_01.png)

图 20.1：将 Prometheus 和 Grafana 添加到系统架构中

在这里，我们可以看到 Prometheus 如何使用 Kubernetes Pods 定义中的注释来收集我们的微服务指标。然后，它将这些指标存储在其数据库中。用户可以通过**网络浏览器**访问 Kiali 和 Grafana 的 Web UI 来监控这些指标。网络浏览器使用在第十八章的*设置访问 Istio 服务*部分中介绍的**minikube 隧道**来访问 Kiali、Grafana，以及来自邮件服务器的网页，以查看 Grafana 发送的警报。

请记住，*第十八章*中用于部署 Istio 的配置仅适用于开发和测试，而不是生产。例如，存储在 Prometheus 数据库中的性能指标将不会在 Prometheus Pod 重启后幸存！

本书使用的 Istio 版本为 v1.17.0，Grafana 版本为 v9.0.1，Prometheus 版本为 v2.34.0。在 Grafana v8 中引入了新的警报系统。为了支持使用 Grafana v8 以下旧版本的本书读者，我们将使用旧的警报配置方式。在下面的*配置 Grafana*部分中，Grafana 将被配置为使用旧的警报系统。

在下一节中，我们将查看对源代码进行了哪些更改，以便微服务产生 Prometheus 可以收集的性能指标。

# 源代码更改以收集应用程序指标

Spring Boot 2 引入了对使用**Micrometer**库（[`micrometer.io`](https://micrometer.io)）以 Prometheus 格式产生性能指标的支持。我们只需要对微服务的源代码进行一个更改：我们需要在 Gradle 构建文件`build.gradle`中添加对 Micrometer 库`micrometer-registry-prometheus`的依赖项。依赖项看起来是这样的：

```java
implementation 'io.micrometer:micrometer-registry-prometheus' 
```

这将使微服务在端口`4004`上使用`/actuator/prometheus`路径产生 Prometheus 指标。

在第十八章，我们将管理端口（由 actuator 使用）与微服务暴露的 API 请求服务的端口分开。如需回顾，请参阅“观察服务网格”部分。

为了让 Prometheus 了解这些端点，每个微服务的 Pod 都被注解了以下代码：

```java
annotations:
  prometheus.io/scrape: "true"
  prometheus.io/port: "4004"
  prometheus.io/scheme: http
  prometheus.io/path: "/actuator/prometheus" 
```

这被添加到每个组件的 Helm 图表的 `values.yaml` 文件中。请参阅 `kubernetes/helm/components`。

为了在 Prometheus 收集指标后更容易识别其来源，它们被标记为生成该指标的微服务的名称。这是通过向通用配置文件 `config-repo/application.yml` 添加以下配置来实现的：

```java
management.metrics.tags.application: ${spring.application.name} 
```

这将导致每个生成的指标都有一个额外的标签名为 `application`。它将包含微服务的标准 Spring 属性名称的值，即 `spring.application.name`。

最后，为了确保我们从配置的 Prometheus 端点获取指标，已在 `test-em-all.bash` 中添加了一个测试。它看起来像：

```java
if [[ $USE_K8S == "true" ]]
then
  # Verify access to Prometheus formatted metrics
  echo "Prometheus metrics tests"
  assertCurl 200 "curl -ks https://health.minikube.me/actuator/prometheus"
fi 
```

注意，只有当针对 Kubernetes 运行测试脚本时，此测试才会运行。

这些都是准备微服务以生成性能指标并让 Prometheus 了解要使用哪些端点开始收集它们的必要更改。在下一节中，我们将构建和部署微服务。

# 构建和部署微服务

使用 `test-em-all.bash` 测试脚本构建、部署和验证部署的方式与第十九章“使用 EFK 栈进行集中日志记录”中“构建和部署微服务”部分的方式相同。运行以下命令：

1.  使用以下命令从源构建 Docker 镜像：

    ```java
    cd $BOOK_HOME/Chapter20
    eval $(minikube docker-env -u)
    ./gradlew build
    eval $(minikube docker-env)
    docker-compose build 
    ```

`eval $(minikube docker-env -u)` 命令确保 `./gradlew build` 命令使用主机的 Docker 引擎，而不是 Minikube 实例中的 Docker 引擎。`build` 命令使用 Docker 来运行测试容器。

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

1.  如果 Minikube 隧道尚未运行，请按照以下步骤启动（如需回顾，请参阅第十八章“设置访问 Istio 服务”部分）： 

    ```java
    minikube tunnel 
    ```

请记住，此命令需要您的用户具有 `sudo` 权限，并且在启动时输入您的密码。在命令请求密码之前需要几秒钟，所以很容易错过！

1.  使用以下命令运行正常测试以验证部署：

    ```java
    ./test-em-all.bash 
    ```

预期输出将与我们在前几章中看到的结果相似：

![计算机截图  描述由中等置信度自动生成](img/B19825_20_02.png)

图 20.2：所有测试正常

微服务部署后，我们可以继续前进，并开始使用 Grafana 监控我们的微服务！

# 使用 Grafana 仪表板监控微服务

正如我们在引言中提到的，Kiali 提供了一些非常实用的仪表板，它们通常是针对应用级性能指标，例如每秒请求数、响应时间和处理请求的故障百分比。不久我们将看到，它们在应用级非常有用。但如果我们想了解底层硬件资源的利用率，我们需要更详细的指标，例如 Java VM 相关的指标。

Grafana 有一个活跃的社区，其中之一就是分享可重用的仪表板。我们将尝试使用社区中的一个仪表板，该仪表板专门用于从 Spring Boot 应用程序（如我们的微服务）中获取大量有价值的 Java VM 相关指标。最后，我们将看到如何在 Grafana 中构建我们自己的仪表板。但让我们先探索 Kiali 和 Grafana 中随盒提供的仪表板。

在我们这样做之前，我们需要做两项准备工作：

1.  安装本地邮件服务器进行测试并配置 Grafana，使其能够向其发送警报邮件。我们将在*在 Grafana 中设置警报*部分使用该邮件服务器。

1.  为了能够监控一些指标，我们将启动我们在前几章中使用的负载测试工具。

## 安装本地邮件服务器进行测试

在本节中，我们将设置一个本地测试邮件服务器，并配置 Grafana 向邮件服务器发送警报邮件。

Grafana 可以向任何 SMTP 邮件服务器发送邮件，但为了保持测试的本地化，我们将部署一个名为`maildev`的测试邮件服务器。按照以下步骤进行：

1.  使用以下命令在 Istio 的命名空间中安装测试邮件服务器：

    ```java
    kubectl -n istio-system create deployment mail-server --image maildev/maildev:2.0.5
    kubectl -n istio-system expose deployment mail-server --port=1080,1025 --type=ClusterIP
    kubectl -n istio-system wait --timeout=60s --for=condition=ready pod -l app=mail-server 
    ```

1.  为了使邮件服务器的 Web UI 可以从 Minikube 外部访问，我们在 Istio 的 Helm 图表中为邮件服务器添加了一组`Gateway`、`VirtualService`和`DestinationRule`清单文件。请参阅模板`kubernetes/helm/environments/istio-system/templates/expose-mail.yml`。运行`helm upgrade`命令以应用新的清单文件：

    ```java
    helm upgrade istio-hands-on-addons kubernetes/helm/environments/istio-system -n istio-system 
    ```

1.  通过访问其网页[`mail.minikube.me`](https://mail.minikube.me)来验证测试邮件服务器是否正在运行。预期将渲染如下网页：

![计算机的截图  描述由中等置信度自动生成](img/B19825_20_03.png)

图 20.3：邮件服务器网页

有关邮件服务器的更多信息，请参阅[`hub.docker.com/r/maildev/maildev`](https://hub.docker.com/r/maildev/maildev)。

邮件服务器安装完成后，我们可以在下一节中配置 Grafana，使其能够向服务器发送警报邮件。

## 配置 Grafana

配置 Grafana 可以通过在其 Kubernetes Deployment 对象中设置环境变量来完成。要启用旧警报系统并配置 Grafana 向测试邮件服务器发送邮件，请运行以下命令：

```java
kubectl -n istio-system set env deployment/grafana \
    GF_ALERTING_ENABLED=true \
    GF_UNIFIED_ALERTING_ENABLED=false \
    GF_SMTP_ENABLED=true \
    GF_SMTP_SKIP_VERIFY=true \
    GF_SMTP_HOST=mail-server:1025 \
    GF_SMTP_FROM_ADDRESS=grafana@minikube.me
kubectl -n istio-system wait --timeout=60s --for=condition=ready pod -l app=Grafana 
```

变量 `GF_ALERTING_ENABLED` 和 `GF_UNIFIED_ALERTING_ENABLED` 用于启用上面提到的 *使用 Prometheus 和 Grafana 介绍性能监控* 部分中提到的旧版警报系统。变量 `GF_SMTP_ENABLED` 用于允许 Grafana 发送电子邮件。变量 `GF_SMTP_SKIP_VERIFY` 用于告诉 Grafana 跳过与测试邮件服务器的 SSL 检查。

`GF_SMTP_HOST` 变量指向我们的邮件服务器，最后，`GF_SMTP_FROM_ADDRESS` 变量指定在邮件中使用的“发件人”地址。

现在已经配置了 Grafana，在下一节中，我们将开始启动负载测试工具。

## 启动负载测试

为了有东西可以监控，让我们使用在前面章节中使用的 Siege 启动负载测试。运行以下命令以获取访问令牌，然后使用访问令牌进行授权启动负载测试：

```java
ACCESS_TOKEN=$(curl https://writer:secret-writer@minikube.me/oauth2/token -d grant_type=client_credentials -d scope="product:read product:write" -ks | jq .access_token -r)
echo ACCESS_TOKEN=$ACCESS_TOKEN
siege https://minikube.me/product-composite/1 -H "Authorization: Bearer $ACCESS_TOKEN" -c1 -d1 -v 
```

请记住，访问令牌仅有效 1 小时——之后，您需要获取一个新的。

现在，我们准备好学习 Kiali 和 Grafana 中的仪表板，并探索 Istio 一起提供的 Grafana 仪表板。

## 使用 Kiali 内置的仪表板

在 *第十八章* 中，我们学习了 Kiali，但跳过了 Kiali 显示性能指标的部分。现在，是时候回到这个主题了！

执行以下步骤来了解 Kiali 的内置仪表板：

1.  使用 [`kiali.minikube.me`](https://kiali.minikube.me) URL 在网页浏览器中打开 Kiali 网页 UI。如果需要，请使用 `admin`/`admin` 登录。

1.  要查看我们的部署，请通过点击左侧菜单上的 **工作负载** 选项卡进入工作负载页面。

1.  通过点击它来选择 **product-composite** 部署。

1.  在 **product-composite** 页面上，选择 **出站指标** 选项卡。您将看到一个类似于以下截图的页面：![计算机的截图  自动生成的描述，中等置信度](img/B19825_20_04.png)

    图 20.4：Kiali 出站指标

    Kiali 将可视化一些非常有价值的一般性能图表，并且还有更多图表可以探索。请随意尝试它们！

1.  然而，在 Grafana 中还有更多详细性能指标可供使用。使用 [`grafana.minikube.me`](https://grafana.minikube.me) URL 在网页浏览器中打开 Grafana 网页 UI。

1.  您将看到一个带有文本 **欢迎使用 Grafana** 的欢迎页面。欢迎文本上方有一个 **主页** 链接；点击它，您将看到一个可用仪表板的概览。您将看到一个名为 **Istio** 的文件夹，其中包含在 *第十八章* 中与 Istio 一起部署的仪表板。点击文件夹以展开并选择名为 **Istio 网格仪表板** 的仪表板。

    预期一个类似于以下网页：

    ![计算机的截图  自动生成的描述](img/B19825_20_05.png)

    图 20.5：Grafana 显示 Istio 网格仪表板

    此仪表板为服务网格中涉及的微服务的指标提供了非常好的概览，如请求率、响应时间和成功率。

1.  有很多详细的性能指标可供查看。返回**Istio**文件夹（在顶部菜单中点击**Istio**），并选择名为**Istio Workload Dashboard******的仪表板。选择**hands-on**命名空间和**product-composite**工作负载。最后，展开**出站服务**选项卡。网页应类似于以下屏幕截图：

![计算机屏幕截图 自动生成描述](img/B19825_20_06.png)

图 20.6：微服务的大量指标展示的 Grafana

页面显示响应代码、持续时间以及每个目的地的发送字节数等指标。请随意浏览 Istio 提供的其他仪表板！

正如我们之前提到的，Istio 仪表板在应用层面上提供了非常好的概览。但还需要监控每个微服务的硬件使用指标。在下一节中，我们将了解如何导入现有的仪表板——特别是显示基于 Spring Boot 应用程序的 Java VM 指标的仪表板。

## 导入现有的 Grafana 仪表板

正如我们之前提到的，Grafana 有一个活跃的社区，分享可重用的仪表板。它们可以在[`grafana.com/grafana/dashboards`](https://grafana.com/grafana/dashboards)中探索。我们将尝试一个名为**JVM (Micrometer) - Kubernetes - Prometheus by Istio**的仪表板，该仪表板专门用于从 Kubernetes 环境中的 Spring Boot 应用程序中获取大量有价值的 JVM 相关指标。仪表板的链接是[`grafana.com/grafana/dashboards/11955`](https://grafana.com/grafana/dashboards/11955)。按照以下步骤导入此仪表板：

1.  按照以下步骤导入名为**JVM (Micrometer)**的仪表板：

    1.  在 Grafana 网页上，将鼠标悬停在左侧菜单中的**仪表板**图标（四个方块）上。从弹出的菜单中选择**+ 导入**。

    1.  在**导入**页面，将仪表板 ID `11955` 输入到**通过 grafana.com 导入**字段中，并点击其旁边的**加载**按钮。

    1.  在将显示的**导入**页面上，点击**Prometheus**下拉菜单并选择**Prometheus**数据源。

    1.  现在，通过点击**导入**按钮，**JVM (Micrometer)**仪表板将被导入并渲染。

1.  按照以下步骤检查**JVM (Micrometer)**仪表板：

    1.  为了获得良好的指标视图，使用时间选择器（位于右上角）选择**过去 5 分钟**，并在右侧下拉菜单中选择**5 秒**的刷新率。

    1.  在页面左上角的**应用**下拉菜单中，选择**product-composite**微服务。

    1.  由于我们在后台使用 Siege 进行负载测试，我们将看到很多指标。以下是一个示例屏幕截图：

![计算机屏幕截图  描述由中等置信度自动生成](img/B19825_20_07.png)

图 20.7：显示 Java VM 指标的 Grafana

在这个仪表板中，我们可以找到所有类型的 Java VM 相关指标，包括 CPU、内存、堆和 I/O 使用情况，以及与 HTTP 相关的指标，如每秒请求数、平均持续时间以及错误率。请随意探索这些指标！

当我们想要快速开始时，能够导入现有的仪表板非常有价值。然而，更重要的是要知道如何创建我们自己的仪表板。我们将在下一节中学习这一点。

## 开发自己的 Grafana 仪表板

开始开发 Grafana 仪表板的过程非常简单。我们需要理解的重要一点是 Prometheus 为我们提供了哪些指标。

在本节中，我们将学习如何检查可用的指标。基于这些指标，我们将创建一个仪表板，用于监控一些更有趣的指标。

### 检查 Prometheus 指标

在之前的*修改源代码以收集应用指标*部分，我们配置了 Prometheus 从我们的微服务中收集指标。我们可以调用相同的端点并查看 Prometheus 收集了哪些指标。运行以下命令：

```java
curl https://health.minikube.me/actuator/prometheus -ks 
```

命令将产生大量输出，如下例所示：

![计算机屏幕截图  描述由中等置信度自动生成](img/B19825_20_08.png)

图 20.8：Prometheus 指标

在所有报告的指标中，有两个非常有趣的指标：

+   `resilience4j_retry_calls`：Resilience4j 报告了重试机制的工作方式。它报告了成功和失败请求的四个不同值，包括和不含重试的情况。

+   `resilience4j_circuitbreaker_state`：Resilience4j 报告了断路器的状态。

注意到这些指标有一个名为`application`的标签，其中包含微服务的名称。这个字段来自我们在*修改源代码以收集应用指标*部分中配置的`management.metrics.tags.application`属性。

这些指标值得监控。我们之前使用的所有仪表板都没有使用 Resilience4j 的指标。在下一节中，我们将为这些指标创建一个仪表板。

### 创建仪表板

在本节中，我们将学习如何创建一个仪表板，用于可视化我们在上一节中描述的 Resilience4j 指标。

我们将分以下阶段设置仪表板：

+   创建一个空白的仪表板

+   为断路器指标创建一个新的面板

+   为重试指标创建一个新的面板

+   安排面板

#### 创建一个空白的仪表板

执行以下步骤以创建一个空白的仪表板：

1.  在 Grafana 网页上，将鼠标悬停在左侧菜单中的**仪表板**图标（四个方块）上。从弹出的菜单中选择**+ 新仪表板**。将显示一个名为**新仪表板**的网页：

![手机截图  描述自动生成，置信度低](img/B19825_20_09.png)

图 20.9：在 Grafana 中创建新的仪表板

1.  点击仪表板设置按钮（其图标为齿轮），在前面截图显示的菜单中。然后，按照以下步骤操作：

    1.  在**名称**字段中指定仪表板的名称，并将其值设置为`Hands-on Dashboard`。

    1.  点击网页左上角的返回按钮（不要与网页浏览器的返回按钮混淆）。

1.  点击时间选择器并选择**最后 5 分钟**作为范围。

1.  点击右侧的刷新率图标并指定**5s**作为刷新率。

#### 为断路器指标创建新的面板

执行以下步骤以创建一个新的断路器指标面板：

1.  在**添加面板**部分，点击**添加新面板**按钮。

将显示一个页面，其中可以配置新的面板。

1.  在右侧标签页中，将**面板标题**设置为`Circuit Breaker`。

1.  同样在右侧标签页中，将**工具提示模式**设置为**全部**。

1.  在左下角的**查询**面板中，在字母**A**下，指定查询为**关闭**状态的断路器指标名称，如下所示：

    1.  将**指标**设置为`resilience4j_circuitbreaker_state`。

    1.  将**标签**设置为**state**并指定它应等于**closed**。

    1.  确认**原始查询**设置为`resilience4j_circuitbreaker_state{state="closed"}`。

1.  展开选项卡，并在**图例**下拉框中选择**自定义**。在**图例**字段中指定值`{{state}}`。这将在面板中创建一个图例，显示不同状态的名字。

    填充的值应如下所示：

![计算机截图  描述自动生成，置信度中等](img/B19825_20_10.png)

图 20.10：在 Grafana 中指定断路器指标

1.  点击页面底部的**+ 查询**按钮，在**B**下为**open**状态输入一个新的查询。重复查询**A**的步骤，但将状态值设置为**open**。确认**原始查询**字段设置为`resilience4j_circuitbreaker_state{state="open"}`，并将**图例**字段设置为`{{state}}`。

1.  再次点击**+ 查询**按钮，在**C**下为**half_open**状态输入一个新的查询。将状态值设置为`half_open`，并确认**原始查询**字段设置为`resilience4j_circuitbreaker_state{state="half_open"}`，并将**图例**字段设置为`{{state}}`。

1.  点击页面左上角的返回按钮以返回仪表板。

#### 为重试指标创建新的面板

在这里，我们将重复之前为前一个断路器指标添加面板的相同步骤，但我们将指定重试指标的值：

1.  通过点击顶级菜单中的**添加面板**图标（一个带有加号的图表）创建一个新的面板，并在新面板中点击**添加新面板**。

1.  将**面板标题**指定为`Retry`，并且与上一个面板一样，将**工具提示模式**设置为**全部**。

1.  将**度量**设置为`resilience4j_retry_calls_total`。

1.  由于重试指标是一个计数器，其值只会上升。一个不断上升的指标在监控上并不那么有趣。因此，使用**速率**函数将重试指标转换为每秒速率指标。指定的时窗，即`30s`，由速率函数用来计算速率的平均值。要应用速率函数：

    1.  点击**+ 操作**按钮。

    1.  点击**范围函数**并选择**速率**函数。

    1.  将**范围**设置为**30s**。

1.  在**原始查询**字段中，确认它设置为`rate(resilience4j_retry_calls_total[30s])`。

1.  展开选项卡**选项**，然后在**图例**下拉框中选择**自定义**。在**图例**字段中，指定值`{{kind}}`。这将创建一个面板图例，其中显示不同类型重试的名称。

1.  注意，Grafana 会立即根据指定的值在面板编辑器中开始渲染图表。

1.  点击后退按钮返回仪表板。

#### 安排面板

执行以下步骤以在仪表板上安排面板：

1.  您可以通过拖动其右下角到所需大小来调整面板的大小。

1.  您也可以通过拖动其标题到所需位置来移动一个面板。

下面是两个面板的示例布局：

![计算机屏幕截图  自动生成描述](img/B19825_20_11.png)

图 20.11：在 Grafana 中移动和调整面板大小

由于这个屏幕截图是在 Siege 在后台运行时拍摄的，**重试**面板报告`successful_without_retry`指标，而**断路器**报告**关闭**等于**1**，**打开**和**半开**等于**0**，这意味着它是关闭的并且正常运行（这在下一节中将要改变）。

1.  最后，点击页面顶部的**保存**按钮。将显示**保存仪表板为...**对话框；确保名称是**动手仪表板**，然后点击**保存**按钮。

如果你在配置仪表板时遇到困难，请查看*尝试使用断路器警报*部分的末尾。那里描述了一个简单的解决方案。

创建仪表板后，我们就可以尝试使用了。在下一节中，我们将尝试这两个指标。

### 尝试使用新的仪表板

在我们开始测试新的仪表板之前，我们必须停止负载测试工具 Siege。为此，转到 Siege 运行的命令窗口，并按**Ctrl + C**停止它。

让我们先测试如何监控断路器。之后，我们将尝试重试指标。

#### 测试断路器指标

如果我们强制断路器打开，其状态将从**关闭**变为**打开**，然后最终变为**半开**状态。这应该在断路器面板中报告。

打开断路器，就像我们在第十三章“使用 Resilience4j 提高弹性”部分中的“尝试断路器和重试机制”中所做的那样——也就是说，连续向 API 发送一些请求，所有这些请求都将失败。运行以下命令：

```java
ACCESS_TOKEN=$(curl https://writer:secret-writer@minikube.me/oauth2/token -d grant_type=client_credentials -d scope="product:read product:write" -ks | jq .access_token -r)
echo ACCESS_TOKEN=$ACCESS_TOKEN
for ((n=0; n<4; n++)); do curl -o /dev/null -skL -w "%{http_code}\n" https://minikube.me/product-composite/1?delay=3 -H "Authorization: Bearer $ACCESS_TOKEN" -s; done 
```

我们可以预期收到三个`500`响应和一个最终的`200`响应，表示连续三个错误，这是打开断路器所需的条件。最后的`200`表示当`product-composite`微服务检测到断路器开启时，它发出的**快速失败**响应。

在一些罕见的情况下，我注意到在创建仪表板后，断路器指标并没有直接在 Grafana 中报告。如果一分钟内没有显示，只需重新运行前面的命令再次打开断路器。

预期**关闭**状态值将降至**0**，而**开启**状态值将取值为**1**，这意味着断路器现在已开启。10 秒后，断路器将转为半开启状态，由**半开启**指标值为**1**和**开启**设置为**0**表示。这意味着断路器已准备好测试一些请求，以查看导致断路器开启的问题是否已消失。

再次关闭断路器，通过以下命令向 API 发出三个成功的请求：

```java
for ((n=0; n<4; n++)); do curl -o /dev/null -skL -w "%{http_code}\n" https://minikube.me/product-composite/1?delay=0 -H "Authorization: Bearer $ACCESS_TOKEN" -s; done 
```

我们将只收到`200`响应。请注意，断路器指标又恢复正常，这意味着**关闭**指标值恢复到**1**。

在此测试之后，Grafana 仪表板应如下所示：

![计算机屏幕截图  描述由低置信度自动生成](img/B19825_20_12.png)

图 20.12：在 Grafana 中查看的重试和断路器操作

从前面的屏幕截图可以看出，重试机制也报告了成功和失败的指标。当断路器开启时，所有请求都未进行重试而失败。当断路器关闭时，所有请求都未进行重试而成功。这是预期的结果。

现在我们已经看到了断路器指标在操作中的表现，让我们看看重试指标在操作中的表现！

如果你想检查断路器的状态，可以使用以下命令：

```java
curl -ks https://health.minikube.me/actuator/health | jq -r .components.circuitBreakers.details.product.details.state 
```

它应报告其状态为`CLOSED`、`OPEN`或`HALF_OPEN`，具体取决于其状态。

#### 测试重试指标

要触发重试机制，我们将使用我们在前几章中使用的`faultPercentage`参数。为了避免触发断路器，我们需要为该参数使用相对较低的值。运行以下命令：

```java
while true; do curl -o /dev/null -s -L -w "%{http_code}\n" -H "Authorization: Bearer $ACCESS_TOKEN" -k https://minikube.me/product-composite/1?faultPercent=10; sleep 3; done 
```

此命令将每三秒调用 API 一次。它指定 10%的请求应该失败，以便重试机制能够启动并重试失败的请求。

几分钟后，仪表板应报告以下指标：

![计算机屏幕截图  描述由中等置信度自动生成](img/B19825_20_13.png)

图 20.13：在 Grafana 中查看的重试测试结果

在前面的屏幕截图中，我们可以看到大多数请求已成功执行，没有重试。大约 10%的请求通过重试机制重试，并在重试后成功执行。

在我们离开创建仪表板的章节之前，我们将学习如何导出和导入仪表板。

## 导出和导入 Grafana 仪表板

一旦创建了一个仪表板，我们通常想要执行以下两个操作：

+   将仪表板的定义作为源代码保存在 Git 仓库中

+   将仪表板移动到其他 Grafana 实例，例如，用于 QA 和生产环境的那些实例

要执行这些操作，我们可以使用 Grafana 的导出和导入仪表板的 API。由于我们只有一个 Grafana 实例，我们将执行以下步骤：

1.  将仪表板导出为 JSON 文件。

1.  删除仪表板。

1.  从 JSON 文件导入仪表板。

在执行这些步骤之前，我们需要了解仪表板具有两种不同类型的 ID：

+   `id`，一个在 Grafana 实例内唯一的自增标识符。

+   `uid`，一个可以在多个 Grafana 实例中使用的唯一标识符。它是访问仪表板时 URL 的一部分，这意味着只要仪表板的`uid`保持不变，仪表板的链接就会保持相同。当创建仪表板时，Grafana 会创建一个随机的`uid`。

当我们导入仪表板时，如果设置了`id`字段，Grafana 将尝试更新它。为了能够在没有安装仪表板的 Grafana 实例中测试导入仪表板，我们需要将`id`字段设置为`null`。

执行以下操作以导出并导入您的仪表板：

1.  识别您的仪表板的`uid`。

`uid`值可以在显示仪表板的网页浏览器中的 URL 中找到。它看起来像这样：

```java
[`grafana.minikube.me/d/YMcDoBg7k/hands-on-dashboard`](https://grafana.minikube.me/d/YMcDoBg7k/hands-on-dashboard) 
```

1.  上面的 URL 中的`uid`是`YMcDoBg7k`。在终端窗口中，创建一个包含其值的变量。在我的情况下，它将是：

```java
`ID=YMcDoBg7k` 
```

1.  使用以下命令将仪表板导出为 JSON 文件：

    ```java
    curl -sk https://grafana.minikube.me/api/dashboards/uid/$ID | jq '.dashboard.id=null' > "Hands-on-Dashboard.json" 
    ```

`curl`命令将仪表板导出为 JSON 格式。`jq`语句将`id`字段设置为`null`，`jq`命令的输出被写入名为`Hands-on-Dashboard.json`的文件。

1.  删除仪表板。

在网页浏览器中，选择左侧菜单中的**仪表板**和**浏览**。在仪表板列表中识别**Hands-on Dashboard**，通过点击其前面的复选框来选择它。将显示一个红色的**删除**按钮；点击它，然后点击弹出的确认对话框中显示的新**删除**按钮。

1.  使用以下命令通过导入 JSON 文件重新创建仪表板：

    ```java
    curl -i -XPOST -H 'Accept: application/json' -H 'Content-Type: application/json' -k \
        'https://grafana.minikube.me/api/dashboards/db' \
        -d @Hands-on-Dashboard.json 
    ```

注意，用于访问仪表板的 URL 仍然有效，在我的情况下，是[`grafana.minikube.me/d/YMcDoBg7k/hands-on-dashboard`](https://grafana.minikube.me/d/YMcDoBg7k/hands-on-dashboard)。

1.  确认导入的仪表板报告的指标与删除和重新导入之前的方式相同。由于在*测试重试指标*部分启动的请求循环仍在运行，因此应该报告该部分的相同指标。

有关 Grafana API 的更多信息，请参阅[`grafana.com/docs/grafana/v9.0/developers/http_api/dashboard/#get-dashboard-by-uid`](https://grafana.com/docs/grafana/v9.0/developers/http_api/dashboard/#get-dashboard-by-uid)。

在继续下一节之前，请记住通过在执行请求循环的终端窗口中按*Ctrl* + *C*来停止我们为重试测试开始的请求循环！

在下一节中，我们将学习如何根据这些指标在 Grafana 中设置警报。

# 在 Grafana 中设置警报

能够监控断路器和重试指标非常有价值，但更重要的是能够在这些指标上定义自动警报。自动警报可以让我们免于手动监控指标。

Grafana 自带内置支持，可以定义警报并将通知发送到多个渠道。在本节中，我们将定义断路器的警报，并配置 Grafana 在警报触发时向测试邮件服务器发送电子邮件。本地测试邮件服务器是在*为测试安装本地邮件服务器*部分中安装的。

有关本章节中使用的 Grafana 版本支持的其它类型渠道，请参阅[`grafana.com/docs/grafana/v7.2/alerting/notifications/#list-of-supported-notifiers`](https://grafana.com/docs/grafana/v7.2/alerting/notifications/#list-of-supported-notifiers)。

在下一节中，我们将定义一个基于邮件的通知渠道，该渠道将在本节之后的警报中使用。

## 设置基于邮件的通知渠道

要在 Grafana 中配置基于邮件的通知渠道，请执行以下步骤：

1.  在 Grafana 网页上，点击左侧菜单中的警报菜单选项（图标为警钟）并选择**通知渠道**。

1.  点击**添加渠道**按钮。

1.  将名称设置为`mail`。

1.  选择类型为**电子邮件**。

1.  输入您选择的电子邮件地址。电子邮件将仅发送到本地测试邮件服务器，与指定的电子邮件地址无关。

1.  展开通知设置并选择**默认（为所有警报使用此通知）**。

通知渠道的配置应如下所示：

![计算机的截图  描述由中等置信度自动生成](img/B19825_20_14.png)

图 20.14：设置基于电子邮件的通知渠道

1.  点击**测试**按钮发送测试邮件。

1.  点击**保存**按钮。

1.  点击左侧菜单中的**仪表板**按钮，然后点击**浏览**菜单项。

1.  从列表中选择**动手仪表板**以返回仪表板。

1.  检查测试邮件服务器的网页，以确保我们已收到测试邮件。你应该会收到以下内容：

![计算机的截图  描述自动生成，置信度中等](img/B19825_20_15.png)

图 20.15：在邮件服务器的网页上验证测试邮件

在已设置通知通道的情况下，我们准备好在断路器上定义警报。

## 设置断路器警报

要在断路器上创建警报，我们需要创建警报，然后将警报列表添加到仪表板中，我们可以看到随着时间的推移发生了哪些警报事件。

执行以下步骤以创建断路器警报：

1.  在**Hands-on Dashboard**中，点击**Circuit Breaker**面板的标题。将出现一个下拉菜单。

1.  选择**Edit**菜单选项。

1.  在标签列表中选择**Alert**标签（显示为警钟图标）。

1.  点击**Create Alert**按钮。

1.  在**Evaluate every**字段中，将值设置为`10s`。

1.  在**For**字段中，将值设置为`0m`。

1.  在**Conditions**部分，指定以下值：

    +   对于**WHEN**字段，选择`max()`。

    +   将**OF**字段设置为`query(A, 10s, now)`。

    +   将**IS ABOVE**更改为**IS BELOW**，并将其值设置为`0.5`。

这些设置会导致如果**closed**状态（与`A`变量相关）在最后 10 秒内低于 0.5 时触发警报。当断路器关闭时，此变量的值为 1，否则为 0。因此，这意味着当断路器不再关闭时，会触发警报。

1.  滚动到**Notifications**部分以确认通知将发送到默认的通知通道，即我们之前定义的邮件通道。警报定义应如下所示：

![计算机的截图  描述自动生成，置信度中等](img/B19825_20_16.png)

图 20.16：在 Grafana 中设置警报，第一部分

1.  在右上角点击**Save**按钮，输入类似`Added an alarm`的备注，然后点击**Save**按钮。

1.  点击返回按钮（左箭头）返回到仪表板。

然后，我们需要执行以下步骤来创建警报列表：

1.  在顶级菜单中点击**Add panel**按钮。

1.  在新面板中点击**Add new panel**按钮。

1.  在右上角，点击**Time series**下拉按钮，并选择**Alert list**选项。

1.  在右侧的标签页中，执行以下操作：

    1.  将**Panel title**输入为`Circuit Breaker Alerts`。

    1.  在**Option**部分，将**Show**字段设置为值**Recent state changes**。

    1.  最后，启用名为**Alerts from this dashboard**的切换开关。

1.  在**Settings**行下方展开**Visualization**行，并选择**Alert list**。

1.  在**Panel** **options**行下方，将**Show**字段设置为**Recent state changes**，将**Max items**设置为`10`，并启用**Alerts from this dashboard**选项。

设置应如下所示：

![计算机截图，描述自动生成，置信度中等](img/B19825_20_17.png)

图 20.17：在 Grafana 中设置警报，第二部分

1.  点击后退按钮返回仪表盘。

1.  调整面板以满足你的需求。

1.  使用类似`添加了警报列表`这样的备注保存仪表盘的更改。

这里是一个添加了警报列表的示例布局：

![设备截图，描述自动生成，置信度中等](img/B19825_20_18.png)

图 20.18：在 Grafana 中设置带有重试、电路断路器和警报面板的布局

我们可以看到电路断路器报告的指标为健康状态（带有绿色心形图标），并且警报列表目前为空。

现在，是时候尝试警报了！

## 尝试电路断路器警报

在这里，我们将重复*测试电路断路器指标*部分中的测试，但这次，我们预期会发出警报，并发送电子邮件！让我们开始吧：

1.  如果需要，获取新的访问令牌（有效期为 1 小时）：

    ```java
    ACCESS_TOKEN=$(curl https://writer:secret-writer@minikube.me/oauth2/token -d grant_type=client_credentials -d scope="product:read product:write" -ks | jq .access_token -r)
    echo ACCESS_TOKEN=$ACCESS_TOKEN 
    ```

1.  如我们之前所做的那样打开电路断路器：

    ```java
    for ((n=0; n<4; n++)); do curl -o /dev/null -skL -w "%{http_code}\n" https://minikube.me/product-composite/1?delay=3 -H "Authorization: Bearer $ACCESS_TOKEN" -s; done 
    ```

仪表盘应报告电路状态为之前的状态。几秒钟后，应发出警报，并发送电子邮件。预期仪表盘看起来如下截图所示（你可能需要刷新网页以使警报显示出来）：

![设备截图，描述自动生成，置信度低](img/B19825_20_19.png)

图 20.19：Grafana 中发出的警报

注意到电路断路器面板标题中的警报图标（一个红色的破碎心形）。红色线条标记了警报事件的时间，并且已将警报添加到警报列表中。

1.  在测试邮件服务器中，你应该会看到如下所示的电子邮件截图：

![计算机截图，描述自动生成](img/B19825_20_20.png)

图 20.20：警报电子邮件

1.  太好了！我们得到了警报，正如我们预期的那样！现在，使用以下命令关闭电路，模拟问题已解决：

    ```java
    for ((n=0; n<4; n++)); do curl -o /dev/null -skL -w "%{http_code}\n" https://minikube.me/product-composite/1?delay=0 -H "Authorization: Bearer $ACCESS_TOKEN" -s; done 
    ```

**关闭**指标应恢复正常，即**1**，警报应再次变为绿色。

预期仪表盘看起来如下截图所示：

![设备截图，描述自动生成，置信度低](img/B19825_20_21.png)

图 20.21：如 Grafana 中报告的错误已解决

注意到电路断路器面板标题中的警报图标再次变为绿色；绿色线条标记了**OK**事件的时间，并且它已被添加到警报列表中。

1.  在测试邮件服务器中，你应该会看到如下所示的电子邮件截图：

![计算机截图，描述自动生成，置信度中等](img/B19825_20_22.png)

图 20.22：如电子邮件中报告的错误已解决

这样就完成了如何使用 Prometheus 和 Grafana 监控微服务的步骤。

如果你想要导出邮件通知的配置，可以使用以下命令：

```java
curl https://grafana.minikube.me/api/alert-notifications/1 -ks | jq '.id=null' > mail-notification.jso 
```

要导入它，可以使用以下命令：

```java
curl -ik -XPOST -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  'https://grafana.minikube.me/api/alert-notifications' \
  -d @mail-notification.json 
```

为了您的方便，文件夹 `$BOOK_HOME/Chapter20/kubernetes/grafana/api-export-import` 包含了邮件通知器和仪表板的导出文件，我们在 *导出和导入 Grafana 仪表板* 部分学习了如何导出和导入。

# 摘要

在本章中，我们学习了如何使用 Prometheus 和 Grafana 收集和监控性能指标的警报。

我们看到，为了收集性能指标，我们可以在 Kubernetes 环境中使用 Prometheus。然后我们学习了当在 Pod 的定义中添加一些 Prometheus 注释时，Prometheus 如何自动从 Pod 收集指标。为了在我们的微服务中产生指标，我们使用了 Micrometer。

然后，我们看到了如何使用 Kiali 和 Grafana 中的仪表板来监控收集的指标，这些仪表板是 Istio 安装的一部分。我们还体验了如何消费 Grafana 社区共享的仪表板，并学习了如何开发我们自己的仪表板，其中我们使用了 Resilience4j 的指标来监控其断路器和重试机制的使用。使用 Grafana API，我们可以导出创建的仪表板并将它们导入到其他 Grafana 实例。

最后，我们学习了如何在 Grafana 中定义指标警报以及如何使用 Grafana 发送警报通知。我们使用本地测试邮件服务器接收来自 Grafana 的电子邮件警报通知。

接下来的两个章节应该已经熟悉了，涵盖了在 Mac 或 Windows PC 上安装工具。相反，你可以翻到这本书的最后一章，它将介绍如何使用全新的 **Spring Native** 项目（当时仍处于测试阶段）将基于 Java 的微服务编译成二进制可执行文件。这将使微服务能够在几秒钟内启动，但在构建它们时涉及了增加的复杂性和时间。

# 问题

1.  我们需要对微服务的源代码进行哪些更改，以便它们产生 Prometheus 可以消费的指标？

1.  `management.metrics.tags.application` 配置参数用于什么？

1.  如果你想分析一个关于高 CPU 消耗的支持案例，你将从这个章节的哪个仪表板开始？

1.  如果你想分析一个关于慢速 API 响应的支持案例，你将从这个章节的哪个仪表板开始？

1.  基于计数器的指标，如 Resilience4j 的重试指标，有什么问题，我们该如何以有用的方式进行监控？

1.  这里发生了什么？

![](img/B19825_20_23.png)

图 20.23：这里发生了什么？

如果你阅读的是以灰度渲染的截图，可能很难弄清楚指标的含义。所以，这里有一些帮助：

1.  断路器报告的状态转换按顺序是：

    1.  **half_open** → **open**

    1.  **open** → **half_open**

    1.  **half_open** → **closed**

1.  重试机制报告：

    1.  一波初始请求，其中大部分被报告为**failed_without_retry**，而少数被报告为**successful_without_retry**。

    1.  第二波请求，所有请求都被报告为**successful_without_retry**。

# 加入我们的 Discord 社区

加入我们的社区 Discord 空间，与作者和其他读者进行讨论：

[`packt.link/SpringBoot3e`](https://packt.link/SpringBoot3e)

![](img/QR_Code1849216352344398875.png)
