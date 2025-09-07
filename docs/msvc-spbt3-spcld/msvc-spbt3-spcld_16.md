

# 第十六章：将我们的微服务部署到 Kubernetes

在本章中，我们将部署本书中的微服务到 Kubernetes。为了在不同运行环境中打包和配置微服务以进行部署，我们将使用 Kubernetes 的包管理器 **Helm**。在这样做之前，我们需要回顾一下服务发现是如何使用的。由于 Kubernetes 内置了对服务发现的支持，因此似乎没有必要部署 Netflix Eureka 来实现这一目的。最后，我们还将尝试一些有助于在 Kubernetes 中部署微服务的 Spring Boot 功能。

本章将涵盖以下主题：

+   用 Kubernetes 服务对象和 `kube-proxy` 替换 Netflix Eureka 进行服务发现

+   介绍 Kubernetes 的使用方法

+   使用 Spring Boot 对优雅关闭和存活性/就绪性探测的支持

+   使用 Helm 打包、配置和在不同环境中部署微服务

+   使用测试脚本 `test-em-all.bash` 验证部署

# 技术要求

关于如何安装本书中使用的工具以及如何访问本书的源代码的说明，请参阅：

+   *第二十一章*，*macOS 安装说明*

+   *第二十二章*，*使用 WSL 2 和 Ubuntu 的 Microsoft Windows 安装说明*

本章中的代码示例均来自 `$BOOK_HOME/Chapter16` 的源代码。

如果你想要查看本章源代码中应用的变化，即查看在 Kubernetes 上部署微服务所需的内容，你可以将此源代码与 *第十五章*，*Kubernetes 简介* 中的源代码进行比较。你可以使用你喜欢的 `diff` 工具比较两个文件夹，`$BOOK_HOME/Chapter15` 和 `$BOOK_HOME/Chapter16`。

# 用 Kubernetes 服务替换 Netflix Eureka

如前一章，*第十五章*，*Kubernetes 简介* 所示，Kubernetes 内置了一个基于 Kubernetes 服务对象和 `kube-proxy` 运行时组件的发现 **服务**。这使得部署像 Netflix Eureka 这样的单独发现服务变得不必要，我们在前几章中使用了它。

使用 Kubernetes 发现服务的一个优点是它不需要像我们与 Netflix Eureka 一起使用的 Spring Cloud LoadBalancer 这样的客户端库。这使得 Kubernetes 发现服务易于使用，与微服务基于哪种语言或框架无关。

使用 Kubernetes 发现服务的缺点是它只能在 Kubernetes 环境中工作。然而，由于发现服务基于接受服务对象 DNS 名称或 IP 地址请求的 `kube-proxy`，因此用类似的发现服务替换它应该相当简单，例如，另一个容器编排器捆绑的服务。

总结这一点，我们将从我们的微服务架构中移除基于 Netflix Eureka 的发现服务器，如下面的图所示：

![图描述自动生成](img/B19825_16_01.png)

图 16.1：用 Kubernetes 内置的发现服务替换 Netflix Eureka

要将基于 Netflix Eureka 的发现服务器替换为 Kubernetes 内置的发现服务，我们需要在我们的构建和配置文件中做一些修改。我们不需要对 Java 源代码进行任何修改，除了某些测试类，其中不再需要属性，因此将被删除。以下更改已应用于源代码：

+   Netflix Eureka 和 Spring Cloud LoadBalancer 特定配置（客户端和服务器）已从配置存储库`config-repo`中移除。

+   已从`config-repo/gateway.yml`文件中移除了网关服务到 Eureka 服务器的路由规则。

+   `spring-cloud/eureka-server`文件夹中的 Eureka 服务器项目已被移除。

+   已从 Docker Compose 文件和`settings.gradle` Gradle 文件中移除了 Eureka 服务器。

+   已从所有 Eureka 客户端构建文件`build.gradle`中移除了对`spring-cloud-starter-netflix-eureka-client`的依赖。

+   已从以前 Eureka 客户端的所有集成测试中移除了属性设置`eureka.client.enabled=false`。

+   网关服务不再使用 Spring Cloud LoadBalancer 中的客户端负载均衡器的路由。例如，`lb://product-composite`路由目标在`config-repo/gateway.yml`文件中已被替换为`http://product-composite`。

+   微服务和授权服务器使用的 HTTP 端口已从端口`8080`（在授权服务器的情况下为`9999`）更改为默认 HTTP 端口`80`。这已在`config-repo`中为每个受影响的 Service 进行配置，如下所示：

    ```java
    spring.config.activate.on-profile: docker
    server.port: 80 
    ```

我们将使用的所有 HTTP 地址都不会受到用 Kubernetes 服务替换 Netflix Eureka 的影响。例如，复合服务使用的地址不受影响：

```java
private final String productServiceUrl = "http://product";
private final String recommendationServiceUrl = "http://recommendation";
private final String reviewServiceUrl = "http://review"; 
```

这是因为我们已将微服务和授权服务器使用的 HTTP 端口更改为默认 HTTP 端口`80`，如前所述。

即使移除了 Netflix Eureka，使用 Docker Compose 仍然有效。这之所以可行，是因为 Docker Compose 文件中的容器名称与 Kubernetes 中使用的相应服务名称相同，这意味着微服务的 DNS 名称在两个环境中都是相同的。这可以用来在不部署到 Kubernetes 的情况下运行微服务的功能测试，例如，与 Docker Desktop 一起运行`test-em-all.bash`，就像我们在前面的章节中所做的那样。然而，移除 Netflix Eureka 意味着当我们使用纯 Docker 和 Docker Compose 时，不再有发现服务。因此，只有在部署到 Kubernetes 时，微服务的扩展才能工作。

在 *第十七章*，*通过实现 Kubernetes 功能简化系统景观*，在 *验证微服务无需 Kubernetes 也能工作* 的部分，我们将讨论避免微服务源代码依赖于 Kubernetes 平台的重要性，从而避免供应商锁定。我们还将使用测试脚本 `test-em-all.bash` 以及 Docker Compose 来验证微服务在功能上不需要 Kubernetes。

既然我们已经熟悉了 Netflix Eureka 将如何被 Kubernetes 服务所取代，那么让我们介绍其他我们将使用的 Kubernetes 对象。

# 介绍 Kubernetes 的使用方法

在本章的后面部分，我们将详细看到如何使用各种 Kubernetes 对象来部署微服务以及它们所依赖的资源管理器，如数据库和队列管理器。在深入所有细节之前，让我们先了解一下将要使用的 Kubernetes 对象：

+   对于将在 Kubernetes 中部署的每个微服务、数据库和队列管理器，将创建一个 Deployment 对象和一个 Service 对象。对于所有组件，除了名为 `gateway` 的边缘服务器外，Service 对象的类型将是 `ClusterIP`。对于网关，Service 对象的类型将是 `NodePort`，在端口 `30433` 接受外部 HTTPS 请求。

+   配置服务器将使用 ConfigMap，其中包含 `config-repo` 中的配置文件。

+   为了存储配置服务器及其客户端的凭证，将创建两个 Secrets：一个用于配置服务器，一个用于其客户端。

现在我们已经看到了将要创建的 Kubernetes 对象，让我们学习一下 Spring Boot 的功能，这些功能有助于简化 Kubernetes 的部署。

# 使用 Spring Boot 对优雅关闭和存活性/就绪性探测的支持

在 Spring Boot v2.3 中，添加了一些有用的功能来支持将部署到 Kubernetes：

+   **优雅关闭：**

当微服务实例需要停止时，例如在滚动升级场景中，当实例停止时可能会影响活动请求。为了最小化这种风险，Spring Boot 添加了对优雅关闭的支持。在应用优雅关闭时，微服务停止接受新的请求，并在关闭应用程序之前等待一个可配置的时间，以便完成活动请求。完成时间超过关闭等待期的请求将被终止。这些请求将被视为异常情况，关闭程序在停止应用程序之前不能等待。

通过在 `config-repo` 文件夹中的公共文件 `application.yml` 中添加以下内容，为所有微服务启用了优雅关闭，等待期为 10 秒：

```java
server.shutdown: graceful
spring.lifecycle.timeout-per-shutdown-phase: 10s 
```

更多信息，请参阅 [`docs.spring.io/spring-boot/docs/3.0.4/reference/htmlsingle/#features.graceful-shutdown`](https://docs.spring.io/spring-boot/docs/3.0.4/reference/htmlsingle/#features.graceful-shutdown)。

+   **存活性和就绪性探针：**

如在第十五章《Kubernetes 简介》中所述，正确实现存活性和就绪性探针对于 Kubernetes 能够管理我们的 Pods 至关重要。

简要回顾一下，存活性探针告诉 Kubernetes 是否需要替换 Pod，而就绪性探针告诉 Kubernetes 其 Pod 是否准备好接受请求。为了简化这项工作，Spring Boot 添加了对实现存活性和就绪性探针的支持。这些探针分别暴露在 `/actuator/health/liveness` 和 `/actuator/health/readiness` URL 上。如果需要比配置提供的更多控制，它们可以通过配置或源代码中的实现来声明。当通过配置声明探针时，可以为每个探针声明一个 **健康组**，指定它应包含哪些现有的健康指标。例如，如果微服务无法访问其 MongoDB 数据库，就绪性探针应报告 `DOWN`。在这种情况下，就绪性探针的健康组应包括 `mongo` 健康指标。有关可用的健康指标，请参阅 [`docs.spring.io/spring-boot/docs/3.0.4/reference/htmlsingle/#actuator.endpoints.health.auto-configured-health-indicators`](https://docs.spring.io/spring-boot/docs/3.0.4/reference/htmlsingle/#actuator.endpoints.health.auto-configured-health-indicators)。

在本章中，我们将使用以下配置在 `config-repo` 文件夹中的公共文件 `application.yml` 中声明探针：

```java
management.endpoint.health.probes.enabled: true
management.endpoint.health.group.readiness.include: readinessState, 
rabbit, db, mongo 
```

配置文件的第一行启用了存活性和就绪性探针。第二行声明，如果可用，就绪性探针将包括 RabbitMQ、MongoDB 和 SQL 数据库的健康指标。对于存活性探针，我们不需要添加任何额外的健康指标。在本章的范围内，只要 Spring Boot 应用程序正在运行，存活性探针报告 `UP` 就足够了。

更多信息，请参阅 [`docs.spring.io/spring-boot/docs/3.0.4/reference/htmlsingle/#actuator.endpoints.kubernetes-probes`](https://docs.spring.io/spring-boot/docs/3.0.4/reference/htmlsingle/#actuator.endpoints.kubernetes-probes)。

我们将在将微服务部署到 Kubernetes 之后尝试这些功能。在我们这样做之前，我们需要了解 Helm 并看看它是如何帮助我们打包、配置和部署微服务到 Kubernetes 的。

# 介绍 Helm

如上所述，将微服务部署到 Kubernetes 需要编写声明部署对象和服务对象所需状态的清单文件。如果我们还需要为微服务添加一些配置，就必须添加 ConfigMaps 和 Secrets 的清单。声明所需状态并将责任交给 Kubernetes 以确保实际状态始终尽可能接近所需状态的方法非常有用。

然而，编写和维护这些清单文件可能会变成一项重大的维护负担。这些文件将包含大量的样板代码，意味着所有微服务的清单看起来都相同。即使只需要更新内容的一小部分，处理特定环境的设置而不重复整个清单文件集也是一件麻烦事。

在只有少数微服务将被部署到少数环境（如测试、QA 和生产环境）的情况下，这可能不是处理的主要问题。

当微服务的数量增长到数十甚至数百，并且必须能够将不同的微服务组部署到不同的测试、QA 和生产环境中时，这很快就会变成一个难以管理的维护问题。

为了解决这些不足，我们将使用基于开源的包管理器 Helm ([`helm.sh`](https://helm.sh))。Helm 附带了一种模板语言，可以用来从各种 Kubernetes 对象的通用定义中提取特定于微服务或环境的设置。

对于只有少数部署对象的较小系统景观，简单的模板工具可能就足够了。例如，如果你已经熟悉**Ansible**及其**Jinja2**模板，它们可以替代使用。此外，`kubectl`本身也内置了对**Kustomize**的支持，提供了一种无需模板即可自定义 Kubernetes 清单文件的替代方案。

在 Helm 中，一个包被称为**chart**。一个 chart 包含模板、模板的默认值以及可选的依赖项，这些依赖项来自其他 chart 中的定义。每个需要部署的组件，即微服务和它们所依赖的资源管理器（如数据库和队列管理器），都将有自己的 chart，描述如何部署它。

为了从组件的 chart 中提取样板定义，将使用一种特殊的 chart，即**库 chart**。库 chart 不包含任何可部署的定义，而只包含其他 chart 用于 Kubernetes 清单的模板——在我们的案例中，用于 Deployment、Service、ConfigMap 和 Secret 对象。

最后，为了能够描述如何将所有组件部署到不同类型的环境中，例如，用于开发和测试或预生产和生产，我们将使用**父图表**和**子图表**的概念。我们将定义两种环境类型，`dev-env`和`prod-env`。每个环境都将实现为一个依赖于不同子图表集的父图表，例如，微服务图表。环境图表还将提供特定于环境的默认值，例如请求的 Pod 数量、Docker 镜像版本、凭证以及资源请求和限制。

总结来说，我们将有一个可重用的库图表，命名为`common`；一组针对微服务和资源管理器的特定图表，放置在`components`文件夹中；以及两个环境特定的父图表，放置在`environments`文件夹中。文件结构如下：

```java
|-- common
|   |-- Chart.yaml
|   |-- templates
|   |-- templates_org
|   `-- values.yaml
|-- components
|   |-- auth-server
|   |-- config-server
|   |-- gateway
|   |-- mongodb
|   |-- mysql
|   |-- product
|   |-- product-composite
|   |-- rabbitmq
|   |-- recommendation
|   |-- review
|   `-- zipkin-server
`-- environments
    |-- dev-env
    `-- prod-env 
```

这些文件可以在文件夹`$BOOK_HOME/Chapter16/kubernetes/helm`中找到。

要与他人共享 Helm 图表，可以将它们发布到 Helm **图表仓库**。在这本书中，我们不会发布任何图表，但在第十七章*实现 Kubernetes 功能以简化系统景观*中，我们将使用来自图表仓库的 Helm 图表安装名为**cert-manager**的组件。

在我们了解图表是如何构建之前，让我们了解最常用的 Helm 命令以及如何运行它们。

## 运行 Helm 命令

要让 Helm 为我们执行某些操作，我们将使用其 CLI 工具，`helm`。

最常用的 Helm 命令包括：

+   `create`：用于创建新的图表。

+   `dependency update`（简称`dep up`）：解决对其他图表的依赖。图表放置在`charts`文件夹中，并更新文件`Chart.lock`。

+   `dependency build`：根据文件`Chart.lock`中的内容重建依赖。

+   `template`：渲染由模板创建的定义文件。

+   `install`：安装一个图表。此命令可以覆盖图表提供的值，可以使用`--set`标志覆盖单个值，或使用`--values`标志提供自己的`yaml`文件，其中包含值。

+   `install --Dry-run`：模拟一个部署而不执行它；在执行之前验证部署很有用。

+   `list`：列出当前命名空间中的安装。

+   `upgrade`：更新现有安装。

+   `uninstall`：删除安装。

要查看 Helm 提供的命令的完整文档，请参阅[`helm.sh/docs/helm/`](https://helm.sh/docs/helm/)。

让我们把这些 Helm 命令放在上下文中，看看一个图表由哪些文件组成。

## 查看 Helm 图表

Helm 图表有一个预定义的文件结构。我们将使用以下文件：

+   `Chart.yaml`，其中包含有关图表的一般信息和可能依赖的其他图表列表。

+   `templates`，一个包含用于部署图表的模板的文件夹。

+   `values.yaml`，它包含模板使用的变量的默认值。

+   `Chart.lock`，Helm 在解决`Chart.yaml`文件中描述的依赖关系时创建的文件。该信息更详细地描述了实际使用的依赖关系。Helm 使用它来跟踪整个依赖关系树，使得能够精确地重新创建上一次图表工作时的依赖关系树。

+   `charts`，一个文件夹，在 Helm 解决依赖关系后，将包含此图表所依赖的图表。

+   `.helmignore`，一个类似于`.gitignore`的忽略文件。它可以用来列出在构建图表时应排除的文件。

现在我们已经了解了 Helm 图表内部的架构，让我们来学习 Helm 的核心功能之一：其模板机制以及如何向其传递值。

## Helm 模板和值

Helm 模板用于参数化 Kubernetes 清单文件。使用模板，我们不再需要为每个微服务维护冗长的 Deployment 清单。相反，我们可以定义一个包含在模板中占位符的通用模板，当为特定微服务渲染清单时，这些占位符将放置微服务特定的值。让我们看一个例子，它来自`kubernetes/helm/common/templates/_deployment.yaml`：

```java
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "common.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
        - name: {{ .Chart.Name }} 
```

它看起来与我们在第十五章“Kubernetes 简介”中看到的 Deployment 清单非常相似，唯一的区别是使用了`{{ ... }}`构造，用于将微服务特定的值插入到模板中。构造`{{ include "common.fullname" . }}`用于调用其他模板，如下所述。其他两个构造用于使用 Helm 中的**内置对象**之一插入值。

常用内置对象的以下部分：

+   `Values`: 用于引用图表的`values.yaml`文件中的值或运行 Helm 命令（如`install`）时提供的值。

+   `Release`: 用于提供有关当前已安装发布版本的元数据。它包含如下字段：

    +   `Name`: 发布版本的名称

    +   `Namespace`: 执行安装的命名空间名称

    +   `Service`: 安装服务的名称，总是返回`Helm`

+   `Chart`: 用于从`Chart.yaml`文件中访问信息。以下是一些可用于为部署提供元数据的字段示例：

    +   `Name`: 图表的名称

    +   `Version`: 图表的版本号

+   `Files`: 包含访问特定图表文件的函数。在本章中，我们将使用`Files`对象中的以下两个函数：

    +   `Glob`: 根据 glob 模式返回图表中的文件。例如，模式`"config-repo/*"`将返回在`config-repo`文件夹中找到的所有文件

    +   `AsConfig`: 返回适合在`ConfigMap`中声明值的 YAML 映射文件内容

+   `Capabilities`：可用于查找有关在安装上执行的操作的 Kubernetes 集群的功能信息。例如，模板可以使用此对象中的信息根据实际 Kubernetes 集群支持的 API 版本采用清单。我们本章不会使用此对象，但我认为对于更高级的使用案例，了解它对我们是有益的。

有关内置对象的更多详细信息，请参阅[`helm.sh/docs/chart_template_guide/builtin_objects`](https://helm.sh/docs/chart_template_guide/builtin_objects)。

所有对象都可以在一个树结构中访问，其中`root`上下文，在大多数情况下，可以使用当前作用域来表示，用点`.`表示，也称为**点**。从上面的例子中我们可以看到点的使用，例如在`.Values.replicaCount`和`.Chart.Name`中，我们可以看到内置对象`Values`和`Chart`可以直接在当前作用域下访问。在上面的`include`指令中，我们也可以看到点被用作参数发送给名为`common.fullname`的模板，这意味着整个树被发送到模板。而不是将整个树发送到模板，可以传递一个子树。

当使用一些 Helm 函数时，当前作用域将发生变化，不再指向`root`上下文。例如，我们将在稍后遇到`range`函数，它可以用来遍历值集合。如果我们需要在`range`函数的作用域内访问`root`上下文，我们可以使用预定义变量`$`。

Helm 模板还支持声明变量以引用其他对象。例如：

```java
$name := .Release.Name 
```

在本例中，已声明一个变量`name`，用于存储当前正在处理的 Helm 发布版本值。我们将在稍后了解变量如何在更高级的结构中使用。

如果你认识从使用`kubectl`中使用的`{{ ... }}`构造的格式，你是正确的。在两种情况下，它们都是基于 Go 模板。有关更多信息，请参阅[`golang.org/pkg/text/template/`](https://golang.org/pkg/text/template/)。

在引入模板机制后，让我们了解三种图表类型是如何构建的。我们将从最重要的图表`common`图表开始，之后解释`components`和`environments`图表。

## 常用库图表

此图表包含可重用的模板，也称为**命名模板**，用于本章中我们将使用的四种 Kubernetes 清单类型：`Deployment`、Service、`ConfigMap`和`Secret`。常用图表的结构和内容基于`helm create`命令的输出。具体来说，模板文件`_helpers.tpl`已被保留，以便重用命名约定的最佳实践。它声明以下模板，这些模板封装了命名约定：

+   `common.name`：基于图表名称。

+   `common.fullname`：基于发布名称和图表名称的组合。在本书中，我们将覆盖这个命名约定，并简单地使用图表名称。

+   `common.chart`：基于图表名称和版本。

有关详细信息，请参阅 `_helpers.tpl` 文件中的实现。

命名模板，这些模板将仅被其他模板使用，而不会用于创建自己的清单，必须以下划线 `_` 开头。这是为了防止 Helm 尝试仅使用它们来创建清单。

由于之前提到的 Kubernetes 清单的命名模板包含 Helm 图表中的主要逻辑部分，因此包含大部分复杂性，我们将逐一介绍它们。

### ConfigMap 模板

这个模板旨在从 `config-repo` 文件夹中的文件创建 ConfigMap。每个 ConfigMap 将包含特定 Deployment 所需的所有非敏感配置。Deployment 清单将把 ConfigMap 的内容映射到其 Pod 模板中的卷。这将导致由 Deployment 创建的 Pod 能够将其配置作为本地文件系统中的文件访问。有关详细信息，请参阅下面的 *Deployment 模板* 部分。`config-repo` 文件夹需要放置在使用通用图表的图表中。

在本章中，这个模板将仅在 `components` 文件夹中的配置服务器图表中使用。在下一章中，所有其他微服务也将使用这个模板来定义它们自己的 ConfigMaps，因为配置服务器将被移除。

模板文件命名为 `_configmap_from_file.yaml`，其内容如下：

```java
{{- define "common.configmap_from_file" -}}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "common.fullname" . }}
  labels:
    app.kubernetes.io/name: {{ include "common.name" . }}
    helm.sh/chart: {{ include "common.chart" . }}
    app.kubernetes.io/managed-by: {{ .Release.Service }}
data:
{{ (.Files.Glob "config-repo/*").AsConfig | indent 2 }}
{{- end -}} 
```

模板的解释如下：

+   第一行，`{{- define "common.configmap_from_file " -}}`，用于声明可重用模板的名称。模板的作用域以匹配的 `{{- end -}}` 结束，本例中的最后一行。

+   为了设置 ConfigMap 的名称，使用了来自 `_helpers.tpl` 文件的模板 `common.fullname`。

+   接下来，定义了一系列标签以便于稍后更容易地识别 ConfigMap。同样，这里使用了 `_helpers.tpl` 文件中的模板来设置 `name` 并指定所使用的 `chart`。为了标记这个服务是使用 Helm 创建的，将标签 `app.kubernetes.io/managed-by` 设置为字段 `.Release.Service` 的值。根据对 `Release` 对象的早期描述，我们知道它总是返回值 `Helm`。

+   接下来是 ConfigMap 的核心部分，其 `data` 部分。为了在 ConfigMap 中指定实际的配置，使用了 `Files` 对象中的 `Glob` 函数来获取 `config-repo` 文件夹中的所有文件。然后，将 `AsConfig` 函数应用于文件中的内容，以形成一个正确的 YAML 映射。结果通过管道传递到 `indent` 函数，该函数确保渲染出适当的缩进，在这种情况下，使用两个字符。

`{{-` 和 `-}}` 中的连字符用于删除大括号内指令处理后剩余的前导和尾随空白。

#### 使用 ConfigMap 模板的示例

在本章中，只有配置服务器将使用 ConfigMap。请参阅关于 *组件图表* 的部分，了解此模板的使用方法。

要查看 Helm 使用此模板创建的 ConfigMap，请运行以下命令：

```java
cd $BOOK_HOME/Chapter16/kubernetes/helm/components/config-server
helm dependency update .
helm template . -s templates/configmap_from_file.yaml 
```

期望从 `helm template` 命令得到如下输出：

```java
---
# Source: config-server/templates/configmap_from_file.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-server
  labels:
    app.kubernetes.io/name: config-server
    helm.sh/chart: config-server-1.0.0
    app.kubernetes.io/managed-by: Helm
data:
  application.yml: |-
    app:
      auth-server: localhost
  ...
  auth-server.yml: |-
    server.port: 9999
  ... 
```

`data` 字段包含 `config-repo` 文件夹中所有文件的内容。

### Secrets 模板

此模板旨在创建由环境 `dev-env` 和 `prod-env` 提供的凭证等值定义的 Secrets。Secrets 将作为 Pod 中的环境变量映射。请参阅下面的 *部署模板* 部分以获取详细信息。由于环境必须能够定义多个 Secrets，因此此模板设计为使用 Helm 中的 `range` 函数创建多个 Secret 清单。模板文件命名为 `_secrets.yaml`，其外观如下：

```java
{{- define "common.secrets" -}}
{{- range $secretName, $secretMap := .Values.secrets }}
apiVersion: v1
kind: Secret
metadata:
  name: {{ $secretName }}
  labels:
    app.kubernetes.io/name: {{ $secretName }}
    helm.sh/chart: {{ include "common.chart" $ }}
    app.kubernetes.io/managed-by: {{ $.Release.Service }}
type: Opaque
data:
{{- range $key, $val := $secretMap }}
  {{ $key }}: {{ $val | b64enc }}
{{- end }}
---
{{- end -}}
{{- end -}} 
```

如下解释模板：

+   在第 1 行声明模板之后，第 2 行使用了 `range` 函数。该函数假定 `.Values.secrets` 字段包含一个 Secret 名称映射和一个 Secret 的键/值对映射。在某个环境的 `values.yaml` 文件中声明 `Secrets` 字段的示例如下：

    ```java
    secrets:
      a-secret:
        key-1: secret-value-1
        key-2: secret-value-2
      another-secret:
        key-3: secret-value-3 
    ```

此定义将渲染两个 Secrets，分别命名为 `a-secret` 和 `another-secret`。`range` 函数将当前 Secret 名称及其映射分配给变量 `$secretName` 和 `$secretMap`。

+   由于 `range` 函数改变了当前作用域，我们不能再使用点符号将 `root` 上下文传递给 `common.chart` 模板。相反，必须使用变量 `$`。

+   在清单的 `data` 部分中，再次应用第二个 `range` 函数来遍历当前 Secret 的键/值对。每个键/值对由 `range` 函数分配给变量 `$key` 和 `$val`。

+   最后，Secret 的键/值对在 `data` 部分定义为映射条目。`$val` 变量的值通过管道传递到 `b64enc` 函数，以获得符合 Secret 清单要求的正确 **Base64** 编码。

使用 `---` 来分隔渲染的 Secret 清单，以便它们作为单独的 YAML 文档进行处理。

#### 使用 Secrets 模板的示例

秘密仅由环境图表 `dev-env` 和 `prod-env` 定义。它们用于创建特定环境的凭证。请参阅关于 *环境图表* 的部分，了解此模板的使用方法。

要查看 Helm 使用此模板为 `dev-env` 创建的 Secrets，请运行以下命令：

```java
cd $BOOK_HOME/Chapter16/kubernetes/helm
for f in components/*; do helm dependency update $f; done
helm dependency update environments/dev-env
helm template environments/dev-env -s templates/secrets.yaml 
```

期望从 `helm template` 命令得到如下输出：

```java
---
# Source: dev-env/templates/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: config-client-credentials
  labels:
    app.kubernetes.io/name: config-client-credentials
    helm.sh/chart: dev-env-1.0.0
    app.kubernetes.io/managed-by: Helm
type: Opaque
data:
  CONFIG_SERVER_PWD: ZGV2LXB3ZA==
  CONFIG_SERVER_USR: ZGV2LXVzcg==
---
# Source: dev-env/templates/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: config-server-secrets
  labels:
    app.kubernetes.io/name: config-server-secrets
    helm.sh/chart: dev-env-1.0.0
    app.kubernetes.io/managed-by: Helm
type: Opaque
data:
  ENCRYPT_KEY: bXktdmVyeS1zZWN1cmUtZW5jcnlwdC1rZXk=
  SPRING_SECURITY_USER_NAME: ZGV2LXVzcg==
  SPRING_SECURITY_USER_PASSWORD: ZGV2LXB3ZA== 
```

### 服务模板

服务模板引入了对从通用图表覆盖默认值的支持，使用通用图表的特定图表的值。例如，通用图表将为服务的 `type` 和服务将暴露的 `ports` 提供默认值。这对于大多数微服务来说将很有用，但其中一些需要在它们自己的 `values.yaml` 文件中覆盖这些默认值。

模板文件命名为 `_service.yaml`，其结构与其他命名模板类似，首先声明其名称，然后是实现覆盖机制。它看起来是这样的：

```java
{{- define "common.service" -}}
{{- $common := dict "Values" .Values.common -}} 
{{- $noCommon := omit .Values "common" -}} 
{{- $overrides := dict "Values" $noCommon -}} 
{{- $noValues := omit . "Values" -}} 
{{- with merge $noValues $overrides $common -}} 
```

这个结构可以这样解释：

+   当微服务使用 `_service.yaml` 模板来渲染其服务清单时，微服务的 `values.yaml` 文件中的值将在 `.Values` 对象中可用，而通用图表的值将在 `.Values.common` 字段下可用。

+   因此，变量 `$common` 将引用一个由 `dict` 函数创建的字典，其中有一个键 `Values`，其值将是通用图表的默认值。这些值来自 `.Values` 对象中的 `common` 键。

+   `$noCommon` 变量将保留微服务中除 `common` 键下的值之外的所有值，该键使用 `omit` 函数指定。

+   `$overrides` 变量将引用一个字典，同样只有一个键 `Values`，但其值将是微服务的值，除了 `common` 值。它从上一行声明的 `$noCommon` 变量中获取值。

+   `$noValues` 变量将保留所有其他内置对象，除了 `Values` 对象。

+   现在，覆盖将在这里发生；`merge` 函数将基于变量 `$noValues`、`$overrides` 和 `$common` 所引用的字典创建一个字典。在这种情况下，`$overrides` 字典中找到的值将优先于 `$common` 字典中的值，从而覆盖其值。

+   最后，`with` 函数将改变后续模板代码的作用域，直到其 `{{- end -}}` 定义为止。因此，当前的作用域 `.` 现在将指向合并后的字典。

让我们通过一个例子来看看这将如何工作。`common` 图表的 `values.yaml` 文件包含以下服务类型和暴露端口的默认设置：

```java
 Service:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: http
    protocol: TCP
    name: http 
```

此设置将渲染类型为 `ClusterIP` 的服务对象。服务对象将暴露端口 `80` 并将请求转发到其端口上名为 `http` 的 Pod。

网关服务需要暴露 `NodePort` 并使用其他端口设置。为了覆盖上述默认值，它在图表的 `values.yaml` 文件中声明以下内容：

```java
service:
  type: NodePort
  ports:
  - port: 443
    targetPort: 8443
    nodePort: 30443 
```

网关的 `values.yaml` 文件位于文件夹 `$BOOK_HOME/Chapter16/kubernetes/helm/components/gateway/values.yaml`。

服务模板文件的其余部分如下所示：

```java
apiVersion: v1
kind: Service
metadata:
  name: {{ include "common.fullname" . }}
  labels: 
    app.kubernetes.io/name: {{ include "common.name" . }}
    helm.sh/chart: {{ include "common.chart" . }}
    app.kubernetes.io/managed-by: {{ .Release.Service }}
spec:
  type: {{ .Values.service.type }}
  ports:
{{ toYaml .Values.service.ports | indent 4 }}
  selector:
    app.kubernetes.io/name: {{ include "common.name" . }}
{{- end -}}
{{- end -}} 
```

模板解释如下：

+   `name`和`labels`的元数据字段与之前看到的模板定义方式相同。

+   Service 的`type`由字段`.Values.service.type`设置。

+   使用字段`.Values.service.ports`指定 Service 暴露的端口。内置函数`toYaml`用于将值格式化为`yaml`，然后将结果传递给`indent`函数，以确保正确缩进，在这种情况下为`4`个字符。

+   最后，定义了 Pod 的`选择器`。它基于标签`app.kubernetes.io/name`，并使用模板`common.name`来指定名称。

#### 使用 Service 模板的示例

每个组件使用 Service 模板来创建其 Service 清单。如上所述，核心微服务重用通用图表的`values.yaml`文件中的配置，而其他组件在其自己的`values.yaml`文件中覆盖这些值。

要查看为核心组件生成的 Service 清单，对于`product`微服务，运行以下命令：

```java
cd $BOOK_HOME/Chapter16/kubernetes/helm
helm dependency update components/product
helm template components/product -s templates/service.yaml 
```

预期`helm template`命令的输出如下：

```java
# Source: product/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: product
  labels:
    app.kubernetes.io/name: product
    helm.sh/chart: product-1.0.0
    app.kubernetes.io/managed-by: Helm
spec:
  type: ClusterIP
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: http
  selector:
    app.kubernetes.io/name: product 
```

要查看为覆盖通用图表设置的组件生成的 Service 清单，对于`gateway`组件，运行以下命令：

```java
cd $BOOK_HOME/Chapter16/kubernetes/helm
helm dependency update components/gateway
helm template components/gateway -s templates/service.yaml 
```

预期`helm template`命令的输出如下：

```java
---
# Source: gateway/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: gateway
  labels:
    app.kubernetes.io/name: gateway
    helm.sh/chart: gateway-1.0.0
    app.kubernetes.io/managed-by: Helm
spec:
  type: NodePort
  ports:
    - nodePort: 30443
      port: 443
      targetPort: 8443
  selector:
    app.kubernetes.io/name: gateway 
```

### 部署模板

最后，我们有用于渲染 Deployment 清单的模板。这是最复杂的模板，因为它必须处理清单中可选的许多部分。不同的组件将使用 Deployment 清单的不同部分。通用图表的`values.yaml`文件包含适用于大多数组件的默认设置值，最小化在每个组件自己的图表的`values.yaml`文件中覆盖这些设置的需求。以下 Deployment 清单的部分对于组件的使用是可选的：

+   容器启动时传递给容器的参数

+   环境变量

+   来自 Secrets 的环境变量

+   活跃探针

+   就绪探针

+   一个 ConfigMap 和相应的卷

模板文件命名为`_deployment.yaml`，其开头几行看起来与 Service 模板非常相似，使用相同的覆盖机制：

```java
{{- define "common.deployment" -}}
{{- $common := dict "Values" .Values.common -}} 
{{- $noCommon := omit .Values "common" -}} 
{{- $overrides := dict "Values" $noCommon -}} 
{{- $noValues := omit . "Values" -}} 
{{- with merge $noValues $overrides $common -}}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "common.fullname" . }}
  labels:
    app.kubernetes.io/name: {{ include "common.name" . }}
    helm.sh/chart: {{ include "common.chart" . }}
    app.kubernetes.io/managed-by: {{ .Release.Service }} 
```

对于模板的这一部分的解释，请参阅上面 Service 模板的描述。

当涉及到清单的`spec`部分时，它从以下内容开始：

```java
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app.kubernetes.io/name: {{ include "common.name" . }}
  template:
    metadata:
      labels:
        app.kubernetes.io/name: {{ include "common.name" . }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}/{{ .Values.image.name }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }} 
```

在这里我们可以看到如何定义 spec 的核心部分：请求的`replicas`数量、Pod 的`selector`以及用于创建新 Pod 的`template`。模板定义了与选择器匹配的`labels`、`name`、Docker`image`和启动容器时使用的`imagePullPolicy`。

接下来是清单中描述的各个可选部分：

```java
 args:
            {{- toYaml . | nindent 12 }}
          {{- end }}
          {{- if .Values.env }}
          env:
          {{- range $key, $val := .Values.env }}
          - name: {{ $key }}
            value: {{ $val }}
          {{- end }}
          {{- end }}
          {{- if .Values.envFromSecretRefs }}
          envFrom:
          {{- range .Values.envFromSecretRefs }}
          - secretRef:
              name: {{ . }}
          {{- end }}
          {{- end }}
          {{- if .Values.livenessProbe_enabled }}
          livenessProbe:
{{ toYaml .Values.livenessProbe | indent 12 }}
          {{- end }}
          {{- if .Values.readinessProbe_enabled }}
          readinessProbe:
{{ toYaml .Values.readinessProbe | indent 12 }}
          {{- end }} 
```

对于映射到环境变量的环境变量和机密信息，使用 `range` 函数的方式与 `secrets` 模板使用的方式相同。环境变量可以在组件或环境级别指定，具体取决于其用例。机密信息始终通过环境图表指定。有关组件和环境图表的更多信息，请参阅以下章节。

清单以声明容器暴露的 `ports`、`resource` 请求和限制以及最后可选地声明一个 ConfigMap 和相应的卷来映射 ConfigMap 中的文件结束：

```java
 ports:
{{ toYaml .Values.ports | indent 12 }}
          resources:
{{ toYaml .Values.resources | indent 12 }}
      {{- if .Values.configmap.enabled }}
          volumeMounts:
          - name: {{ include "common.fullname" . }}
            mountPath: {{ .Values.configmap.volumeMounts.mountPath }}
      volumes:
        - name: {{ include "common.fullname" . }}
          configMap:
            name: {{ include "common.fullname" . }}
      {{- end }}
{{- end -}}
{{- end -}} 
```

从公共图表的 `values.yaml` 文件中，我们可以找到一些有趣的默认值，例如，如何定义存活和就绪探针的默认值：

```java
livenessProbe_enabled: false
livenessProbe:
  httpGet:
    scheme: HTTP
    path: /actuator/health/liveness
    port: 80
  initialDelaySeconds: 10
  periodSeconds: 10
  timeoutSeconds: 2
  failureThreshold: 20
  successThreshold: 1
readinessProbe_enabled: false
readinessProbe:
  httpGet:
    scheme: HTTP
    path: /actuator/health/readiness
    port: 80
  initialDelaySeconds: 10
  periodSeconds: 10
  timeoutSeconds: 2
  failureThreshold: 3
  successThreshold: 1 
```

从这些声明中，我们可以看到：

+   探针默认是禁用的，因为并非所有部署都使用探针。

+   探针基于发送到 Spring Boot 暴露的端点的 HTTP `GET` 请求，如上节所述的 *使用 Spring Boot 对优雅关闭和存活就绪探针的支持*。

    +   只要端点以 `2xx` 或 `3xx` 响应代码响应，探针就被认为是成功的。

+   探针可以使用以下参数进行配置：

    +   `initialDelaySeconds` 指定 Kubernetes 在容器启动后等待多长时间才对其进行探针。

    +   `periodSeconds` 指定 Kubernetes 发送探针请求之间的时间间隔。

    +   `timeoutSeconds` 指定 Kubernetes 在将探针视为失败之前等待响应的时间。

    +   `failureThreshold` 指定 Kubernetes 在放弃之前尝试失败的次数。在存活探针的情况下，这意味着重新启动 Pod。在就绪探针的情况下，这意味着 Kubernetes 将不会向容器发送更多请求，直到就绪探针再次成功。

    +   `successThreshold` 指定探针在失败后再次被视为成功所需的成功尝试次数。这仅适用于就绪探针，因为如果为存活探针指定，则必须设置为 `1`。

寻找探针的最佳设置可能具有挑战性，也就是说，在当 Pod 的可用性发生变化时，从 Kubernetes 获得快速反应和不过度加载 Pod 以探针请求之间找到一个适当的平衡。

具体来说，配置一个存活探针，其值过低可能导致 Kubernetes 重新启动不需要重新启动的 Pod；它们只需要一些额外的时间来启动。同时启动大量 Pod，这也可能导致启动时间过长，同样可能导致大量不必要的重启。

在探针（除 `successThreshold` 值外）上设置配置值过高会使 Kubernetes 响应更慢，这在开发环境中可能会很烦人。适当的值还取决于可用的硬件，这会影响 Pod 的启动时间。在本书的范围内，为了防止在硬件资源有限的计算机上不必要的重启，将 liveness 探针的 `failureThreshold` 设置为高值，`20`。

#### 使用 Deployment 模板的示例

每个组件使用 Deployment 模板来创建其 Deployment 清单。核心微服务在通用图表的 `values.yaml` 文件中重用了大部分配置，最小化了组件特定配置的需求，而其他组件在其自己的 `values.yaml` 文件中覆盖了这些值中的更多。

要查看为核心组件生成的 Deployment 清单，请为 `product` 微服务运行以下命令：

```java
cd $BOOK_HOME/Chapter16/kubernetes/helm
helm dependency update components/product
helm template components/product -s templates/deployment.yaml 
```

要查看为覆盖通用图表设置的组件生成的 Deployment 清单，请为 MongoDB 组件运行以下命令：

```java
cd $BOOK_HOME/Chapter16/kubernetes/helm
helm dependency update components/mongodb
helm template components/mongodb -s templates/deployment.yaml 
```

期望 `helm template` 命令的输出如下：

```java
---
# Source: mongodb/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb
  labels:
    app.kubernetes.io/name: mongodb
    helm.sh/chart: mongodb-1.0.0
    app.kubernetes.io/managed-by: Helm
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: mongodb
  template:
    metadata:
      labels:
        app.kubernetes.io/name: mongodb
    spec:
      containers:
        - name: mongodb
          image: "registry.hub.docker.com/library/mongo:6.0.4"
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 27017
          resources:
            limits:
              memory: 350Mi 
```

这完成了对通用图表中可重用命名模板的浏览。文件可以在文件夹 `$BOOK_HOME/Chapter16/kubernetes/helm/common` 中找到。

接下来，让我们看看如何定义特定组件的图表。

## 组件图表

微服务图表和资源管理器图表存储在 `components` 文件夹中，它们都共享相同的文件结构：

+   `Chart.yaml` 表达了对 `common` 库图表的依赖。

+   `template` 文件夹包含两个模板，`deployment.yaml` 和 `Service.yaml`。这两个模板都应用来自通用图表的相应命名模板。例如，`Service.yaml` 模板看起来像这样：

    ```java
    {{- template "common.service" . -}} 
    ```

+   `values.yaml` 文件包含特定于微服务的设置。例如，`auth-server` 图表的 `values` 文件看起来像这样：

    ```java
    fullnameOverride: auth-server
    image:
      name: auth-server
    env:
      SPRING_PROFILES_ACTIVE: "docker"
    livenessProbe_enabled: true
    readinessProbe_enabled: true 
    ```

`auth-server` 只需要声明其名称、Docker 镜像、Spring 配置文件，以及它希望使用默认的 liveness 和 readiness 探针配置。

配置服务器与其他图表不同，因为它使用 ConfigMap 来存储包含所有其他微服务配置文件的 `config-repo`。在其 `template` 文件夹中，它定义了一个基于我们之前已介绍过的通用图表中 ConfigMaps 命名模板的模板：

```java
{{- template "common.configmap_from_file" . -}} 
```

模板期望在图表文件夹 `config-repo` 中找到属性文件。为了避免从 `$BOOK_HOME/Chapter16/config-repo` 复制 `config-repo`，已使用命令创建了一个**软链接**，也称为**符号链接**：

```java
cd $BOOK_HOME/Chapter16/kubernetes/helm/components/config-server
ln -s ../../../../config-repo config-repo 
```

由于 Git 保留软链接，您不需要重新创建软链接 - `git clone` 命令会为您创建它！

如在通用图表的说明中已提到的，网关服务与其他微服务不同，因为它需要暴露类型为 `NodePort` 的服务。

除了微服务的图表之外，`components` 文件夹还包含我们使用的数据库、消息代理和 Zipkin 服务器的图表。它们的结构方式与微服务相同。由于通用模板已被设计用于简化微服务的图表，因此与其他图表相比，其他图表需要在 `values.yaml` 文件中覆盖更多的默认值。有关更多详细信息，请查看以下文件夹中的 `values.yaml` 文件：`mongodb`、`mysql`、`rabbitmq` 和 `zipkin-server`。

## 环境图表

最后，`environments` 文件夹中的 `dev-env` 和 `prod-env` 图表将所有内容结合起来，以完成典型开发/测试或预发布/生产环境的安装包。它们的 `Charts.yaml` 文件包含对 `common` 图表和 `components` 文件夹中图表的依赖，而 `template` 文件夹包含一个 `secrets.yaml` 模板，用于创建特定环境的凭据作为密钥。它基于通用图表中密钥的命名模板，看起来如下：

```java
{{- template "common.secrets" . -}} 
```

查看 `dev-env` 图表的 `values.yaml` 文件，我们可以找到为 `config-server-secrets` 密钥定义的以下密钥值：

```java
secrets:
  config-server-secrets:
    ENCRYPT_KEY: my-very-secure-encrypt-key
    SPRING_SECURITY_USER_NAME: dev-usr
    SPRING_SECURITY_USER_PASSWORD: dev-pwd 
```

这将导致 `config-server-secrets` 密钥包含三个密钥值，全部为 Base64 编码。其清单看起来如下：

```java
apiVersion: v1
kind: Secret
metadata:
  name: config-server-secrets
  labels:
    ...
type: Opaque
data:
  ENCRYPT_KEY: bXktdmVyeS1zZWN1cmUtZW5jcnlwdC1rZXk=
  SPRING_SECURITY_USER_NAME: ZGV2LXVzcg==
  SPRING_SECURITY_USER_PASSWORD: ZGV2LXB3ZA== 
```

注意，此 `values.yaml` 文件包含敏感信息，例如配置服务器使用的加密密钥和访问配置服务器的密码。此文件必须安全存储。如果不适于安全存储此文件，则另一种选择是在执行 `helm install` 命令时从该文件中删除敏感信息并提供。

要在配置服务器的部署清单中使用密钥，`dev-env` 图表的 `values.yaml` 文件中定义了以下内容：

```java
config-server:
  envFromSecretRefs:
    - config-server-secrets 
```

这将由上面描述的部署模板使用，以在配置服务器的部署清单中将密钥作为环境变量添加。

`prod-env` 图表覆盖的值比 `dev-env` 图表更多。例如，`prod-env` 图表中的 `values.yaml` 文件指定应使用额外的 Spring 配置文件 `prod`，以及 Docker 镜像的版本。对于 `product` 微服务，这看起来如下：

```java
product:
  image:
    tag: v1
  env:
    SPRING_PROFILES_ACTIVE: "docker,prod" 
```

通过介绍各种类型图表包含的内容，让我们继续前进，并使用我们学到的 Helm 命令将微服务部署到 Kubernetes 中！

# 将应用程序部署到 Kubernetes 进行开发和测试

在本节中，我们将部署用于开发和技术活动的微服务环境，例如，系统集成测试。此类环境主要用于功能测试，因此配置为使用最少的系统资源和微服务 Docker 镜像的最新版本。

为了能够运行功能测试，我们将部署微服务及其所需的资源管理器在同一命名空间中，我们将称之为 `hands-on`。这使得设置测试环境变得容易，一旦我们完成测试，也可以轻松删除。我们可以简单地删除命名空间来移除测试环境使用的所有资源。

以下图表说明了此部署场景：

![图描述自动生成](img/B19825_16_02.png)

图 16.2：在开发环境中的微服务相同 Kubernetes 命名空间中部署的资源管理器

在我们可以部署系统景观之前，我们需要构建我们的 Docker 镜像并解决 Helm 图表的依赖项。

## 构建 Docker 镜像

通常，我们必须将镜像推送到 Docker 仓库并配置 Kubernetes 从仓库拉取镜像。在我们的案例中，由于我们有一个本地单节点集群，我们可以通过将 Docker 客户端指向 Minikube 中的 Docker 引擎，然后运行 `docker-compose build` 命令来简化此过程。这将导致 Docker 镜像立即对 Kubernetes 可用。对于开发，我们将使用 `latest` 作为微服务的 Docker 镜像版本。

您可以从源构建 Docker 镜像，如下所示：

```java
cd $BOOK_HOME/Chapter16
./gradlew build
eval $(minikube docker-env)
docker-compose build 
```

`eval $(minikube docker-env)` 命令将本地 Docker 客户端指向 Minikube 中的 Docker 引擎。

`docker-compose.yml` 文件已更新，以指定它构建的 Docker 镜像的名称。例如，对于 `product` 服务，我们有以下内容：

```java
 product:
    build: microservices/product-service
    image: hands-on/product-service 
```

`latest` 是 Docker 镜像名称的默认标签，因此无需指定。

Docker 镜像构建完成后，是时候构建 Helm 图表了。

## 解决 Helm 图表依赖项

首先，我们更新 `components` 文件夹中的依赖项：

```java
for f in kubernetes/helm/components/*; do helm dep up $f; done 
```

接下来，我们更新 `environments` 文件夹中的依赖项：

```java
for f in kubernetes/helm/environments/*; do helm dep up $f; done 
```

最后，我们验证 `dev-env` 文件夹的依赖项看起来良好：

```java
helm dep ls kubernetes/helm/environments/dev-env/ 
```

预期命令会响应如下：

![](img/B19825_16_03.png)

图 16.3：已解决的 Helm 图表依赖项

在构建了 Docker 镜像并解决了 Helm 依赖项后，我们可以开始部署到 Kubernetes！

## 部署到 Kubernetes

将系统部署到 Kubernetes 意味着创建或更新 Kubernetes 对象。我们将按照以下步骤使用 Helm 进行部署：

1.  为了避免由于 Kubernetes 下载 Docker 镜像（可能引起我们之前描述的存活探针重启我们的 Pod）而导致的缓慢部署过程，请运行以下 `docker pull` 命令预先下载镜像：

    ```java
    eval $(minikube docker-env)
    docker pull mysql:8.0.32 
    docker pull mongo:6.0.4
    docker pull rabbitmq:3.11.8-management
    docker pull openzipkin/zipkin:2.24.0 
    ```

1.  在使用 Helm 图表之前，使用`helm template`命令渲染模板，以查看清单将是什么样子：

    ```java
    helm template kubernetes/helm/environments/dev-env 
    ```

    注意，这里没有与 Kubernetes 集群进行交互，因此集群信息将被伪造，并且不会运行测试来验证渲染的清单是否会被集群接受。

1.  要验证 Kubernetes 集群实际上会接受渲染的清单，可以通过将`–-dry-run`传递给`helm install`命令来执行安装的**dry run**。传递`--debug`标志也会显示 Helm 在渲染清单时将使用哪些用户提供的和计算出的值。运行以下命令以执行 dry run：

    ```java
    helm install --dry-run --debug hands-on-dev-env \
     kubernetes/helm/environments/dev-env 
    ```

1.  要启动整个系统景观的部署，包括创建命名空间、`hands-on`，请运行以下命令：

    ```java
    helm install hands-on-dev-env \
      kubernetes/helm/environments/dev-env \
      -n hands-on \
      --create-namespace 
    ```

    注意，这里是 Helm 机制开始发挥作用的地方。它将使用我们在上面*介绍 Helm*部分中介绍的图表来渲染和应用 Kubernetes 清单，从而创建部署所需的 Kubernetes 对象。

1.  将新创建的命名空间设置为`kubectl`的默认命名空间：

    ```java
    kubectl config set-context $(kubectl config current-context) --namespace=hands-on 
    ```

1.  要查看 Pod 启动，请运行以下命令：

    ```java
    kubectl get pods --watch 
    ```

    此命令将连续报告新 Pod 的状态为**运行中**，如果出现问题，它将报告状态，例如**错误**和**CrashLoopBackOff**。过了一会儿，你可能会看到**网关**、**产品组合**和**zipkin-server** Pod 报告了错误。这是因为它们都需要在启动时访问外部资源。如果没有，它们将会崩溃。网关和产品组合服务依赖于认证服务器，而 Zipkin 服务器依赖于对 RabbitMQ 的访问。通常，它们启动速度比它们依赖的资源快，导致这种情况。然而，Kubernetes 会检测到崩溃的 Pod，并将它们重启。一旦资源启动并运行，所有 Pod 都将启动并报告为就绪，**READY**列显示为**1/1**。命令的示例输出如下：

    ![包含文本的图片  自动生成的描述](img/B19825_16_04.png)

    图 16.4：Pod 在外部依赖就绪之前重启

    在看到一些如上所示的输出后，使用*Ctrl*+*C*中断命令。

    1.  使用以下命令等待命名空间中的所有 Pod 就绪：

        ```java
        kubectl wait --timeout=600s --for=condition=ready pod --all 
        ```

    预期命令将响应 11 行日志，如`pod/... condition met`，其中三个点（`...`）将被报告为就绪的实际 Pod 名称所替换。

1.  要查看使用的 Docker 镜像，请运行以下命令：

    ```java
    kubectl get pods -o json | jq .items[].spec.containers[].image 
    ```

响应应如下所示：

![文本、聊天或文本消息  自动生成的描述](img/B19825_16_05.png)

图 16.5：测试环境中使用的 Docker 镜像

注意，微服务的 Docker 镜像的版本标签设置为**latest**。

我们现在准备测试我们的部署！然而，在我们能够这样做之前，我们需要通过测试脚本中所需的更改，以便与 Kubernetes 一起使用。

## 用于 Kubernetes 的测试脚本更改

为了验证部署，我们将像往常一样运行测试脚本，`test-em-all.bash`。为了与 Kubernetes 一起工作，断路器测试已经略有修改。断路器测试在 `product-composite` 服务上调用 `actuator` 端点来检查其健康状态并获取访问断路器事件的权限。由于此端点未对外暴露，前几章使用了 `docker-compose exec` 命令在 `product-composite` 服务内部运行 `curl` 命令以执行测试。

从本章开始，测试脚本可以使用 `docker-compose exec` 命令或相应的 `kubectl` 命令，`kubectl exec`，这取决于我们是否使用 Docker Compose 或 Kubernetes 运行微服务。

要知道使用哪个命令，脚本中已添加了一个新参数，`USE_K8S`。它默认为 `false`。有关详细信息，请参阅测试脚本中的 `testCircuitBreaker()` 函数。

## 测试部署

在启动测试脚本时，我们必须提供运行 Kubernetes 的主机地址，即我们的 Minikube 实例，以及我们的网关服务监听外部请求的 `NodePort`。网关可以通过端口 `30443` 访问。如 *第十五章* 中所述，由于我们使用 Minikube 的 `docker` 驱动程序，主机名始终是 `localhost`。由于主机名与使用 Docker Compose 运行测试时相同，我们不需要指定它；只需指定端口，以及 `USE_K8S` 参数即可。

使用以下命令开始测试：

```java
PORT=30443 USE_K8S=true ./test-em-all.bash 
```

在脚本的输出中，我们可以看到 `NodePort` 的使用情况，但除此之外，一切看起来都和我们在前几章中使用 Docker Compose 时的样子一样：

![文本描述自动生成](img/B19825_16_06.png)

图 16.6：系统景观自动化测试的输出

在系统景观验证完成后，让我们看看我们如何测试 Spring Boot 的新功能，优雅关闭以及存活性和就绪性探测。

### 测试 Spring Boot 对优雅关闭和存活性及就绪性探测的支持

在本节中，我们将测试新的 Spring Boot 功能，并查看它们如何与其他 Kubernetes 组件交互。

让我们先来测试 Spring Boot 对优雅关闭的支持，在这个阶段，应用程序将等待一个可配置的时间长度，以完成活跃请求。记住，在关闭阶段不允许新的请求。

要测试优雅关闭机制，我们将运行一个持续向复合服务发送请求的客户端。首先，我们将使用它发送需要 5 秒的请求，这比关闭等待期短。等待期配置为 10 秒。然后，我们将使用它发送需要更长时间的请求，15 秒，以查看它们是如何处理的。作为测试客户端，我们将使用 **Siege**，这是一个基于命令行的负载测试工具。

为了能够测试运行这种长时间完成的请求，我们需要暂时增加 `product-composite` 服务的超时。否则，其断路器将启动并阻止我们运行长时间请求。

要在复合服务中增加超时，请执行以下步骤：

1.  在 `values` 文件中 `dev-env` 的 `product-composite` 部分添加以下内容，对于 `dev-env`，`kubernetes/helm/environments/dev-env/values.yaml`：

    ```java
     env:
        RESILIENCE4J_TIMELIMITER_INSTANCES_PRODUCT_TIMEOUTDURATION: 20s 
    ```

    更改后，配置文件应如下所示：

    ```java
    product-composite:
      env:
        RESILIENCE4J_TIMELIMITER_INSTANCES_PRODUCT_TIMEOUTDURATION: 20s
      envFromSecretRefs:
        - config-client-credentials 
    ```

只要此设置处于活动状态，`test-em-all.bash` 中的断路器测试将不再工作，因为它们假设超时为 2 秒。

1.  使用 Helm 的 `upgrade` 命令更新 Helm 安装，使用 `--wait` 标志以确保在命令终止时更新完成：

    ```java
    helm upgrade hands-on-dev-env -n hands-on \
      kubernetes/helm/environments/dev-env --wait 
    ```

现在，我们可以运行测试，按照以下步骤使用短于关闭等待期的请求进行测试：

1.  获取访问令牌：

    ```java
    ACCESS_TOKEN=$(curl -d grant_type=client_credentials \
     -ks https://writer:secret-writer@localhost:30443/oauth2/token \
     -d scope="product:read product:write" \
     | jq .access_token -r) 
    ```

    通过运行命令 `echo $ACCESS_TOKEN` 确保你获得了访问令牌。如果它是空的，你必须检查上面的 `curl` 命令和网关以及身份验证服务器的日志。

1.  发送测试请求并使用 `delay` 查询参数请求 5 秒的延迟：

    ```java
    time curl -kH "Authorization: Bearer $ACCESS_TOKEN" \
      https://localhost:30443/product-composite/1?delay=5 
    ```

    如果你得到正常响应，并且 `time` 命令报告了 5 秒的响应时间，增加超时的配置更改已生效！

1.  使用 Siege 启动需要 5 秒才能完成的请求，有五个并发用户发送请求，请求之间有 0 到 2 秒的随机延迟，以稍微分散请求：

    ```java
    siege -c5 -d2 -v -H "Authorization: Bearer $ACCESS_TOKEN" \
      https://localhost:30443/product-composite/1?delay=5 
    ```

    对于每个完成的请求，工具的输出应如下所示：

    ```java
    HTTP/1.1 200 5.04 secs: 771 bytes ==> GET /product-composite/1?delay=5 
    ```

1.  使用以下命令在单独的终端窗口中监视 `product` 服务的日志输出：

    ```java
    kubectl logs -f --tail=0 -l app.kubernetes.io/name=product 
    ```

1.  现在，我们将要求 Kubernetes 重新启动 `product` 部署。重启将首先启动一个新的 Pod，然后关闭旧的 Pod，这意味着 Siege 发送的任何请求都不应受到重启的影响。特别关注的是，当旧 Pod 开始关闭时，它处理的少量请求。如果优雅关闭按预期工作，则不应有任何活跃请求失败。通过在单独的窗口中运行以下命令来执行重启：

    ```java
    kubectl rollout restart deploy/product 
    ```

1.  确保从负载测试工具 Siege 的输出中只报告成功的请求，显示 **200** **(OK)**。

1.  在已停止的`product` Pod 的日志输出中，你应该看到在应用程序停止之前，所有请求都被允许优雅地终止。期望看到如下日志输出，在日志输出的末尾：

![文本描述自动生成](img/B19825_16_07.png)

图 16.7：所有请求都允许完成的优雅关闭

具体来说，注意两个日志消息之间的时间（在这种情况下为 4 秒），这表明关闭程序实际上等待最后一个请求完成。

现在让我们运行第二个测试，请求完成所需时间比关闭等待期更长：

1.  重新启动 Siege，请求更长的响应时间，超过 10 秒的等待限制。启动五个并发用户，请求 15 秒的响应时间和请求之间的 0-5 秒随机延迟。使用*Ctrl*+*C*停止 Siege 并运行以下命令：

    ```java
    siege -c5 -d5 -v -H "Authorization: Bearer $ACCESS_TOKEN" \
      https://localhost:30443/product-composite/1?delay=15 
    ```

1.  使用以下命令监视`product` Pod 的日志输出：

    ```java
    kubectl logs -f --tail=0 -l app.kubernetes.io/name=product 
    ```

1.  重新启动`product`部署：

    ```java
    kubectl rollout restart deploy/product 
    ```

1.  跟踪`product` Pod 的日志输出。一旦它关闭，你应该能够看到在应用程序停止之前，并非所有请求都被允许优雅地终止。期望看到如下日志输出，在日志输出的末尾：![文本描述自动生成](img/B19825_16_08.png)

    图 16.8：某些长时间运行的请求被终止的优雅关闭

    日志消息**优雅关闭因一个或多个请求仍然活跃而被终止**表明在应用程序停止之前至少有一个请求没有被允许完成。

1.  在负载测试工具 Siege 的输出中，现在应该出现一个或几个失败的请求报告**500**（内部服务器错误），如下所示：

![计算机屏幕截图  描述自动生成，中等置信度](img/B19825_16_09.png)

图 16.9：在关闭期间长时间运行的请求失败

这证明了关闭程序在配置的等待时间之后如何进行，以及剩余的长时间运行请求如预期的那样被终止。

这完成了 Spring Boot 优雅关闭机制的测试，这显然有助于避免正常客户端请求受到 Pod 停止的影响，例如，由于缩放或滚动升级执行的结果。

清理测试后的环境：

1.  使用*Ctrl*+*C*停止 Siege 负载测试工具。

1.  回滚最新的 Helm 发布以去除增加的超时：

    ```java
    helm rollback hands-on-dev-env -n hands-on --wait 
    ```

`helm rollback`命令也很有用，可以回滚失败的升级。

1.  还需要在文件`kubernetes/helm/environments/dev-env/values.yaml`中移除增加的超时设置。

1.  运行`test-em-all.bash`以验证配置已回滚：

    ```java
    PORT=30443 USE_K8S=true ./test-em-all.bash 
    ```

最后，让我们看看 Spring Boot 的存活和就绪探针报告了什么信息。我们将使用`product`服务，但也可以尝试其他服务的探针：

1.  运行以下命令以获取`product`服务的存活探针输出：

    ```java
    kubectl exec -it deploy/product -- \
      curl localhost/actuator/health/liveness -s | jq . 
    ```

    预期它会这样响应：

![图形用户界面，文本，应用程序，自动生成描述](img/B19825_16_10.png)

图 16.10：存活性探测的响应

1.  运行以下命令以获取`product`服务就绪性探测的输出：

    ```java
    kubectl exec -it deploy/product -- \
      curl localhost/actuator/health/readiness -s | jq . 
    ```

预期其响应会更加详细：

![计算机屏幕截图，自动生成描述，中等置信度](img/B19825_16_11.png)

图 16.11：就绪性探测的响应

从上面的输出中，我们可以确认现在`product`的可用性取决于其访问 MongoDB 和 RabbitMQ。这是预期的，因为我们配置了就绪健康组以包括 RabbitMQ、MongoDB 和 SQL 数据库的健康指标，如果可用的话。如果需要，请参阅*使用 Spring Boot 对优雅关闭和存活性及就绪性探测的支持*部分进行回顾。

在我们继续之前，让我们清理一下在开发环境中安装的内容。我们可以通过简单地删除命名空间来完成这项工作。删除命名空间将递归地删除命名空间中存在的资源，包括关于 Helm 安装的信息。

使用以下命令删除命名空间：

```java
kubectl delete namespace hands-on 
```

如果你只想卸载`helm install`命令安装的内容，你可以运行命令`helm uninstall hands-on-dev-env`。

在移除开发环境后，我们可以继续前进，并设置一个针对预发布和生产的开发环境。

# 将应用程序部署到 Kubernetes 进行预发布和发布

在本节中，我们将部署用于预发布和生产使用的微服务环境。预发布环境用于在将新版本投入生产之前执行**质量保证**（**QA**）和**用户验收测试**（**UATs**）。为了能够验证新版本不仅满足功能需求，还满足非功能需求，例如性能、健壮性、可扩展性和弹性，预发布环境被配置得尽可能接近生产环境。

将应用程序部署到预发布或生产环境时，与开发或测试部署相比，需要做出一些更改：

+   **资源管理器应在 Kubernetes 集群外部运行**：从技术上讲，可以在 Kubernetes 上作为有状态容器运行数据库和队列管理器以供生产使用，使用`StatefulSets`和`PersistentVolumes`。在撰写本文时，我建议不要这样做，主要是因为 Kubernetes 中对有状态容器的支持相对较新且未经证实。相反，我建议使用现有的数据库和队列管理器本地或作为云中的托管服务，让 Kubernetes 做它最擅长的事情：运行无状态容器。就本书的范围而言，为了模拟生产环境，我们将在 Kubernetes 外部作为普通 Docker 容器运行 MySQL、MongoDB 和 RabbitMQ，使用已存在的 Docker Compose 文件。

+   **锁定**：

    +   由于安全原因，像`actuator`端点和日志级别这样的东西在生产环境中需要受到限制。

    +   从安全角度也应该审查外部暴露的端点。例如，在生产环境中，可能需要锁定对配置服务器的访问，但为了方便，我们将在本书中保持其暴露。

    +   必须指定 Docker 镜像标签，以便能够跟踪已部署的微服务的版本。

+   **扩大可用资源**：为了满足高可用性和更高负载的需求，我们需要每个 Deployment 运行至少两个 Pod。我们可能还需要增加每个 Pod 允许使用的内存和 CPU 的数量。为了避免在 Minikube 实例中内存不足，我们将每个 Deployment 运行一个 Pod，但在生产环境中增加允许的最大内存。

+   **设置一个生产就绪的 Kubernetes 集群**：这超出了本书的范围，但如果可行，我建议使用主要云提供商提供的托管 Kubernetes 服务之一。就本书的范围而言，我们将部署到我们的本地 Minikube 实例。

这并不是在设置生产环境时必须考虑的所有事情的详尽列表，但这是一个良好的开始。

我们的模拟生产环境将如下所示：

![图描述自动生成](img/B19825_16_12.png)

图 16.12：部署在 Kubernetes 外部的资源管理器

## 源代码中的更改

对源代码进行了以下更改，以准备在用于预演和生产的环境中进行部署：

+   在`config-repo`配置存储库中的配置文件中添加了一个名为`prod`的 Spring 配置文件：

    ```java
    spring.config.activate.on-profile: prod 
    ```

+   在`prod`配置文件中，已添加以下内容：

    +   运行作为普通 Docker 容器的资源管理器 URL：

        ```java
        spring.rabbitmq.host: 172.17.0.1
        spring.data.mongodb.host: 172.17.0.1
        spring.datasource.url: jdbc:mysql://172.17.0.1:3306/review-db 
        ```

我们使用`172.17.0.1` IP 地址来在 Minikube 实例中定位 Docker 引擎。这是使用 Minikube 创建 Docker 引擎时的默认 IP 地址，至少对于版本 1.18 以下的 Minikube 来说是这样。

正在进行工作，以建立一个标准 DNS 名称，供容器在需要访问其运行的 Docker 主机时使用，但在撰写本文时，这项工作尚未完成。

+   日志级别已设置为警告或更高，即错误或致命。例如：

    ```java
    logging.level.root: WARN 
    ```

+   在 HTTP 上公开的唯一 `actuator` 端点是 `info` 和 `health` 端点，这些端点用于 Kubernetes 中的存活和就绪探测，以及由测试脚本 `test-em-all.bash` 使用的 `circuitbreakerevents` 端点：

    ```java
    management.endpoints.web.exposure.include: health,info,circuitbreakerevents 
    ```

在现实世界的生产环境中，我们还应该将 `imagePullPolicy: Never` 设置更改为 `IfNotPresent`，以便从 Docker 仓库下载 Docker 镜像。然而，由于我们将部署生产设置到我们手动构建和标记 Docker 镜像的 Minikube 实例中，我们将不会更新此设置。

## 部署到 Kubernetes

为了模拟使用生产级资源管理器，MySQL、MongoDB 和 RabbitMQ 将使用 Docker Compose 在 Kubernetes 外部运行。我们将像前几章那样启动它们：

```java
eval $(minikube docker-env)
docker-compose up -d mongodb mysql rabbitmq 
```

我们还需要使用以下命令对现有的 Docker 镜像进行 `v1` 标记：

```java
docker tag hands-on/auth-server hands-on/auth-server:v1
docker tag hands-on/config-server hands-on/config-server:v1
docker tag hands-on/gateway hands-on/gateway:v1 
docker tag hands-on/product-composite-service hands-on/product-composite-service:v1 
docker tag hands-on/product-service hands-on/product-service:v1
docker tag hands-on/recommendation-service hands-on/recommendation-service:v1
docker tag hands-on/review-service hands-on/review-service:v1 
```

从这里开始，命令与我们在开发环境中部署的方式非常相似：

1.  使用 Helm 部署：

    ```java
    helm install hands-on-prod-env \ kubernetes/helm/environments/prod-env \
    -n hands-on --create-namespace 
    ```

1.  等待部署正常运行：

    ```java
    kubectl wait --timeout=600s --for=condition=ready pod --all 
    ```

1.  要查看当前生产环境中使用的 Docker 镜像，请运行以下命令：

    ```java
    kubectl get pods -o json | jq .items[].spec.containers[].image 
    ```

    响应应该看起来像以下这样：

    ![文本描述自动生成](img/B19825_16_13.png)

    图 16.13：生产环境中使用的 Docker 镜像

    注意 Docker 镜像的 `v1` 版本！

    还要注意，MySQL、MongoDB 和 RabbitMQ 的资源管理器 Pod 已消失；这些可以使用 `docker-compose ps` 命令找到。

1.  运行测试脚本 `test-em-all.bash` 以验证模拟的生产环境：

    ```java
    CONFIG_SERVER_USR=prod-usr \
    CONFIG_SERVER_PWD=prod-pwd \
    PORT=30443 USE_K8S=true ./test-em-all.bash 
    ```

预期与在开发环境中运行测试脚本时相同的输出类型。

这完成了测试；让我们清理一下，以便 Kubernetes 环境为下一章做好准备。

## 清理

要删除我们使用的资源，请运行以下命令：

1.  删除命名空间：

    ```java
    kubectl delete namespace hands-on 
    ```

1.  关闭运行在 Kubernetes 外部的资源管理器：

    ```java
    eval $(minikube docker-env)
    docker-compose down
    eval $(minikube docker-env -u) 
    ```

`eval $(minikube docker-env -u)` 命令指示本地 Docker 客户端与本地 Docker 引擎通信，而不再与 Minikube 中的 Docker 引擎通信。

如本章前面所述，`kubectl delete namespace` 命令将递归删除命名空间中存在的所有 Kubernetes 资源，而 `docker-compose down` 命令将停止 MySQL、MongoDB 和 RabbitMQ。随着生产环境的移除，我们已到达本章的结尾。

# 摘要

在本章中，我们学习了如何使用 Helm 在 Kubernetes 上部署本书中的微服务。我们看到了如何使用 Helm 创建可重用的模板，从而最小化创建 Kubernetes 清单所需的样板代码。可重用模板存储在公共图表中，而特定于微服务的图表提供每个微服务特有的值。在顶层，我们有父图表，描述了如何使用微服务图表部署开发/测试和阶段/生产环境，可选地还可以与数据库和队列管理器等资源管理器的图表一起部署。

我们还看到了如何利用 Spring Boot 功能来简化部署到 Kubernetes 的过程。Spring Boot 对优雅关闭的支持可以在停止基于 Spring Boot 的微服务之前允许活动请求完成，例如，在滚动升级期间。对存活性和就绪性探测的支持使得声明对特定微服务所依赖的外部资源可用性的探测变得容易。

最后，为了能够在 Kubernetes 中部署我们的微服务，我们不得不将 Netflix Eureka 替换为 Kubernetes 内置的发现服务。更改发现服务没有对 Java 源代码进行任何更改——我们只需要对构建依赖项和一些配置进行更改。

在下一章中，我们将看到如何进一步利用 Kubernetes 来减少在 Kubernetes 中需要部署的支持服务的数量。前往下一章，了解我们如何消除对配置服务器的需求，以及我们的边缘服务器如何可以被 Kubernetes Ingress 控制器所替代。

# 问题

1.  为什么我们在 Kubernetes 上部署时从微服务领域中移除了 Eureka 服务器？

1.  我们用什么替换了 Eureka 服务器，以及这个变化如何影响了微服务的源代码？

1.  存活性和就绪性探测的目的是什么？

1.  Spring Boot 的优雅关闭机制有什么用？

1.  以下 Helm 模板指令的目的是什么？

    ```java
    {{- $common := dict "Values" .Values.common -}} 
    {{- $noCommon := omit .Values "common" -}} 
    {{- $overrides := dict "Values" $noCommon -}} 
    {{- $noValues := omit . "Values" -}} 
    {{- with merge $noValues $overrides $common -}} 
    ```

1.  为什么以下命名的 Helm 模板会失败？

    ```java
    {{- define "common.secrets" -}}
    {{- range $secretName, $secretMap := .Values.secrets }}
    apiVersion: v1
    kind: Secret
    metadata:
      name: {{ $secretName }}
      labels:
        app.kubernetes.io/name: {{ $secretName }}
    type: Opaque
    data:
    {{- range $key, $val := $secretMap }}
      {{ $key }}: {{ $val | b64enc }}
    {{- end }}
    {{- end -}}
    {{- end -}} 
    ```

1.  为什么以下清单不能一起工作？

    ```java
    apiVersion: v1
    kind: Service
    metadata:
      name: review
      labels:
        app.kubernetes.io/name: review
    spec:
      type: ClusterIP
      ports:
        - name: http
          port: 80
          protocol: TCP
          targetPort: http
      selector:
        app.kubernetes.io/pod-name: review
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: review
      labels:
        app.kubernetes.io/name: review
    spec:
      replicas: 1
      selector:
        matchLabels:
          app.kubernetes.io/name: review
      template:
        metadata:
          labels:
            app.kubernetes.io/name: review
        spec:
          containers:
            - name: review
              image: "hands-on/review-service:latest"
              ports:
                - containerPort: 80
                  name: http-port
                  protocol: TCP 
    ```
