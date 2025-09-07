

# 第十八章：使用服务网格提高可观察性和管理

在本章中，您将了解服务网格的概念，并了解其功能如何用于处理微服务系统景观中的挑战，包括安全性、策略执行、弹性和流量管理。服务网格还可以用于提供可观察性，即可视化微服务之间流量流向的能力。

服务网格部分重叠了我们在这本书中之前学到的 Spring Cloud 和 Kubernetes 的功能。但正如本章所示，服务网格中的大多数功能都是与 Spring Cloud 和 Kubernetes 相补充的。

本章将涵盖以下主题：

+   服务网格概念和 Istio 介绍，Istio 是一个流行的开源实现

+   在 Kubernetes 中部署 Istio

+   创建、观察和保障服务网格

+   确保服务网格具有弹性

+   执行零停机更新

+   使用 Docker Compose 测试微服务景观，以确保微服务中的源代码不会被锁定在 Kubernetes 或 Istio 中

# 技术要求

关于如何安装本书中使用的工具以及如何访问本书源代码的说明，请参阅：

+   *第二十一章*，*macOS 安装说明*

+   *第二十二章*，*使用 WSL 2 和 Ubuntu 的 Microsoft Windows 安装说明*

本章中的代码示例全部来自 `$BOOK_HOME/Chapter18` 的源代码。

如果您想查看对本章源代码所做的更改，即查看使用 Istio 创建服务网格所需的步骤，您可以将其与 *第十七章*，*实现 Kubernetes 功能以简化系统景观* 的源代码进行比较。您可以使用您喜欢的 `diff` 工具比较两个文件夹，`$BOOK_HOME/Chapter17` 和 `$BOOK_HOME/Chapter18`。

# 使用 Istio 介绍服务网格

服务网格是一个基础设施层，它控制和观察服务之间的通信，例如微服务。服务网格中的功能，例如可观察性、安全性、策略执行、弹性和流量管理，是通过控制和监控服务网格内部的所有内部通信来实现的，即服务网格中的微服务之间的通信。

服务网格中的一个核心组件是一个轻量级的 **代理** 组件，它被注入到将成为服务网格一部分的每个微服务中。所有进出微服务的流量都被配置为通过其代理组件。代理组件由服务网格中的 **控制平面** 在运行时通过代理公开的 API 进行配置。控制平面还通过这些 API 从代理收集遥测数据，以可视化服务网格中的流量流向。

服务网格还包含一个**数据平面**，由代理组件以及处理服务网格外部流量（分别称为**入口网关**和**出口网关**）的独立组件组成。网关组件也通过代理组件与控制平面通信。这将在以下图中说明：

![包含文本、截图、矩形、图表的图片，自动生成描述](img/B19825_18_01.png)

图 18.1：具有控制平面和数据平面的服务网格

服务网格的第一个公开实施是开源项目**Linkerd**，由 Buoyant 管理([`linkerd.io`](https://linkerd.io))，其起源可以追溯到 Twitter 的 Finagle 项目([`twitter.github.io/finagle`](http://twitter.github.io/finagle))。它于 2016 年推出，一年后，即 2017 年，IBM、Google 和 Lyft 推出了开源项目**Istio**([`istio.io`](https://istio.io))。从那时起，已经启动了几个服务网格项目。

想要了解可用的实现概述，请参阅 CNCF 云原生景观图中的**服务网格**类别：[`landscape.cncf.io/card-mode?category=service-mesh&grouping=category`](https://landscape.cncf.io/card-mode?category=service-mesh&grouping=category)。在本书中，我们将使用 Istio。

## 介绍 Istio

可以使用各种安装工具在多个 Kubernetes 发行版和平台上部署 Istio，具体请参阅[`istio.io/docs/setup`](https://istio.io/docs/setup)。我们将使用 Istio 的 CLI 工具`istioctl`在我们的基于 minikube 的单节点 Kubernetes 集群中安装 Istio。

如前所述，Istio 分为控制平面和数据平面。作为操作员，我们将通过在 Kubernetes API 服务器中创建 Istio 对象来定义所需状态，例如，声明路由规则。控制平面将读取这些对象，并向数据平面中的代理发送命令以根据所需状态采取行动，例如，配置路由规则。代理处理微服务之间的实际通信，并将遥测数据报告给控制平面。遥测数据用于控制平面以可视化服务网格中的情况。

当在 Kubernetes 上部署 Istio 时，其大多数运行时组件都部署在单独的 Kubernetes 命名空间`istio-system`中。对于本书中将要使用的配置，我们将在该命名空间中找到以下部署：

+   `istiod`，运行整个控制平面的 Istio 守护进程。

**有趣的事实**：直到 Istio v1.4，控制平面被分成一组协作的微服务。从 v1.5 开始，它们被合并为一个由 `istiod` 运行的单一二进制文件，简化了运行时控制平面的安装和配置。此外，启动时间、资源使用和响应性等运行时特性也得到了改善。我认为，Istio 控制平面的这种演变是关于使用细粒度微服务时学到的有趣经验教训。

+   `istio-ingressgateway` 和 `istio-egressgateway`，Istio 的入口和出口网关组件，是数据平面的一部分。

+   Istio 支持与其他流行的开源项目进行多种集成，以将额外的功能引入控制平面。在这本书中，我们将集成以下组件：

    +   **Kiali**：为服务网格提供可观察性，可视化网格中的情况。更多信息，请参阅[`www.kiali.io`](https://www.kiali.io)。

    +   **跟踪**：基于 Jaeger 或 Zipkin 处理和可视化分布式跟踪信息。我们将使用 Jaeger。更多信息，请参阅[`www.jaegertracing.io`](https://www.jaegertracing.io)。

    +   **Prometheus**：执行基于时间序列数据（例如性能指标）的数据摄取和存储。更多信息，请参阅[`prometheus.io`](https://prometheus.io)。

    +   **Grafana**：可视化 Prometheus 收集的性能指标和其他时间序列相关数据。更多信息，请参阅[`grafana.com`](https://grafana.com)。

    在第二十章“监控微服务”中，我们将探讨使用 Prometheus 和 Grafana 的性能监控能力。

+   关于 Istio 中可用的集成更多信息，请参阅[`istio.io/latest/docs/ops/integrations/`](https://istio.io/latest/docs/ops/integrations/)。

除了 `istio-system` 命名空间之外，唯一部署的 Istio 组件是代理组件，这些组件被注入到服务网格中的微服务中。代理组件基于 Lyft 的 Envoy 代理([`www.envoyproxy.io`](https://www.envoyproxy.io))。

Istio 控制平面和数据平面中的运行时组件总结如下图所示：

![包含文本、截图、矩形 描述自动生成](img/B19825_18_02.png)

图 18.2：Istio 运行时组件

现在我们已经介绍了这些内容，我们将探讨如何将这些代理对象注入到微服务中。

## 将 Istio 代理注入到微服务中

在前几章中，我们在 Kubernetes 中部署的微服务作为一个单独的容器在 Kubernetes Pod 中运行（回顾一下，请参考第十五章“Kubernetes 简介”中的“介绍 Kubernetes API 对象”部分）。

要使微服务加入基于 Istio 的服务网格，需要在每个微服务中注入 Istio 代理。这是通过向运行 Istio 代理的 Pod 中添加一个额外的容器来实现的。

添加到 Pod 中，旨在支持主容器（如 Istio 代理）的容器被称为 **sidecar**。

以下图显示了 Istio 代理如何作为侧边容器注入到示例 Pod，**Pod A**：

![包含文本、截图、数字、字体描述的图片，自动生成](img/B19825_18_03.png)

图 18.3：注入到 Pod A 的 Istio 代理

Pod 中的主容器，**Container A**，被配置为将所有流量路由通过 Istio 代理。

Istio 代理可以在创建 Pod 对象时自动注入，或者使用 `istioctl` 工具手动注入。要告诉 Istio 自动将 Istio 代理注入到命名空间中的新 Pod，可以将命名空间标记为 `istio-injection: enabled`。如果命名空间中的某些 Pod 要排除在自动注入之外，它们可以注解为 `sidecar.istio.io/inject: "false"`。

要将 Istio 代理手动注入到现有 Deployment 对象的 Pod 中，可以使用以下命令：

```java
kubectl get deployment sample-deployment -o yaml | istioctl kube-inject -f - | kubectl apply -f - 
```

这个命令乍一看可能有些令人畏惧，但实际上它只是三个独立的命令。前面的命令通过管道将输出发送到下一个命令，即 `|` 字符。让我们逐一分析每个命令：

+   `kubectl get deployment` 命令从 Kubernetes API 服务器获取名为 `sample-deployment` 的 Deployment 的当前定义，并以 YAML 格式返回其定义。

+   `istioctl kube-inject` 命令从 `kubectl get deployment` 命令读取定义，并在 Deployment 对象中添加一个用于 Istio 代理的额外容器。`Deployment` 对象中现有容器的配置被更新，以便进出流量通过 Istio 代理。

+   `istioctl` 命令返回包含 Istio 代理容器的 Deployment 对象的新定义。

+   `kubectl apply` 命令从 `istioctl kube-inject` 命令读取更新的配置，并应用更新后的配置。属于 Deployment 的 Pod 的升级将以我们之前看到的方式启动（参考第十五章，*Kubernetes 简介*中的*尝试一个示例部署*部分）。

在本书中，我们将通过应用以下 `hands-on` 命名空间的定义来自动注入 Istio 代理。

```java
apiVersion: v1
kind: Namespace
metadata:
  name: hands-on
  labels:
    istio-injection: enabled 
```

从前面的定义中，我们可以看到命名空间被赋予 `istio-injection` 标签，其值为 `enabled`。

在撰写本文时，Istio 还不能完全胜任作为 MySQL、MongoDB 和 RabbitMQ 的代理，因此它们将通过在它们的 Helm 图表的 `values.yaml` 文件中添加以下注解而被排除在服务网格之外：

```java
annotations:
  sidecar.istio.io/inject: "false" 
```

在介绍如何将 Istio 代理注入到 Pod 之后，我们现在可以学习本书中使用的 Istio API 对象。

## 介绍 Istio API 对象

Istio 还附带了一套 Kubernetes **自定义资源定义**（**CRDs**）。CRDs 用于 Kubernetes 以扩展其 API，即向其 API 添加新对象。请参阅 *Chapter 15*，*Introduction to Kubernetes* 中的 *Introducing Kubernetes API objects* 部分，以回顾 Kubernetes API。

在本书中，我们将使用以下 Istio 对象：

+   **Gateway** 用于配置如何处理进入和离开服务网格的流量。网关依赖于将入站流量路由到 Kubernetes 服务的虚拟服务。我们将使用 `gateway` 对象来接受以 `minikube.me` 结尾的 DNS 名称的入站流量，并使用 HTTPS。Istio 网关对象将替换前一章中使用的 **Ingress** 对象。有关详细信息，请参阅 *Replacing Kubernetes Ingress Controller with Istio ingress gateway* 部分。

+   **VirtualService** 用于在服务网格中定义路由规则。我们将使用虚拟服务来描述如何将来自 Istio 网关的入站流量路由到 Kubernetes 服务，以及服务之间的路由。我们还将使用虚拟服务来注入故障和延迟，以测试服务网格的可靠性和弹性能力。

+   **DestinationRule** 用于定义路由到特定服务（即目的地）的流量策略和规则。我们将使用目的地规则来设置加密策略以加密内部 HTTP 流量，并定义描述服务可用版本的子集。在从现有版本的微服务到新版本的零停机（蓝绿）部署时，我们将使用服务子集。

`VirtualService` 和 `DestinationRule` 之间的责任划分在开始时可能显得有些不清楚。`VirtualService` 对象用于配置路由**到**服务，而 `DestinationRule` 用于配置如何处理选定服务的流量。因此，首先是 `VirtualService` 对象，用于确定请求发送到哪里。一旦确定，接收服务的 `DestinationRule` 就会被应用。

+   **PeerAuthentication** 用于控制服务网格内部的服务间认证。Istio 可以通过自动配置传输认证的**双向 TLS**（**mTLS**）来保护服务网格中服务之间的通信，其中客户端服务通过 Istio 提供的客户端证书进行认证。为了允许 Kubernetes 使用纯 HTTP 调用存活性和就绪性探针，我们将配置 Istio 允许 mTLS 和纯 HTTP 的混合模式，称为 `PERMISSIVE` 模式。

+   **RequestAuthentication** 用于根据请求中提供的凭据对最终用户进行身份验证。Istio 支持在一般情况下使用 **JSON Web Tokens**（**JWTs**），特别是当按照 **OpenID Connect**（**OIDC**）规范使用时。Istio 支持使用 OIDC 的标准发现端点来指定 Istio 可以从中获取用于验证 JWT 签名的公钥集 **JSON Web Key Set**（JWKS）。我们将配置 Istio 使用 auth 服务器通过指定其 JWKS 发现端点来对外部请求进行身份验证。为了回顾，请参阅第十一章 *保护 API 访问*。

+   **AuthorizationPolicy** 用于在 Istio 中提供访问控制。在本书中，我们不会使用 Istio 的访问控制。相反，我们将重用 `product-composite` 微服务中实现的现有访问控制。因此，我们将配置一个允许任何经过身份验证的用户（即包含有效 JWT 的 OIDC 访问令牌的请求）访问 `product-composite` 微服务的 `AuthorizationPolicy` 对象。

更多关于这些 API 对象的信息，请参阅 [`istio.io/v1.17/docs/reference/config/networking/`](https://istio.io/v1.17/docs/reference/config/networking/) 和 [`istio.io/v1.17/docs/reference/config/security/`](https://istio.io/v1.17/docs/reference/config/security/)。

现在我们已经介绍了我们将使用的 API 对象，我们将通过 Istio 的引入来查看微服务景观产生的变更。

# 简化微服务景观

如前所述，Istio 包含与微服务景观中当前使用的组件在功能上重叠的组件：

+   Istio 网关可以作为边缘服务器，作为 Kubernetes Ingress 控制器的替代方案

+   随 Istio 一起捆绑的 Jaeger 组件可以用作分布式跟踪，而不是我们与微服务一起部署的 Zipkin 服务器

在接下来的两个小节中，我们将概述为什么以及如何用 Istio 网关替换 Kubernetes Ingress 控制器，以及我们的 Zipkin 服务器被 Istio 集成的 Jaeger 组件所替换的原因。

## 用 Istio 网关替换 Kubernetes Ingress 控制器

在上一章中，我们介绍了 Kubernetes Ingress 控制器作为一个边缘服务器（参考第十七章 *实现 Kubernetes 功能以简化系统景观* 中的 *替换 Spring Cloud Gateway* 部分）。与 Kubernetes Ingress 控制器相比，Istio ingress 网关具有许多优势：

+   它可以向控制平面报告通过它的流量遥测数据

+   它可以用于更细粒度的路由

+   它可以在将请求路由到服务网格之前对请求进行身份验证和授权

为了利用这些优势，我们将用 Istio 入口网关替换 Kubernetes Ingress 控制器。Istio 入口网关是通过创建 `Gateway` 和 `VisualService` 对象来使用的，如之前在 *介绍 Istio API 对象* 部分中所述。

从 `kubernetes/helm/environments` 中的 `dev-env` 和 `prod-env` Helm 图表中删除了之前使用的 Ingress 对象的定义。Istio `Gateway` 和 `VirtualService` 对象的定义文件将在 *创建服务网格* 部分中解释。

使用与访问 Kubernetes Ingress 控制器所用的不同 IP 地址来访问 Istio 入口网关，因此我们还需要更新映射到主机名 `minikube.me` 的 IP 地址，这是我们运行测试时使用的。这已在 *设置访问 Istio 服务* 部分中处理。

## 用 Istio 的 Jaeger 组件替换 Zipkin 服务器

如 *介绍 Istio* 部分所述，Istio 内置了对使用 Jaeger 的分布式跟踪的支持。使用 Jaeger，我们可以通过移除我们在 *第十四章*，*理解分布式跟踪* 中引入的 Zipkin 服务器，来减轻并简化 Kubernetes 中的微服务景观。我们还将更改微服务之间传播跟踪和跨度 ID 的方式，从使用默认的 W3C 跟踪上下文头信息更改为使用 OpenZipkin 的 `B3` 头信息。有关更多信息，请参阅 *第十四章*，*理解分布式跟踪* 中的 *使用 Micrometer 跟踪和 Zipkin 介绍分布式跟踪* 部分。

以下更改已应用于源代码：

+   在所有微服务构建文件 `build.gradle` 中已替换以下依赖项：

    ```java
    implementation 'io.micrometer:micrometer-tracing-bridge-otel'
    implementation 'io.opentelemetry:opentelemetry-exporter-zipkin' 
    ```

    依赖项已替换为以下内容：

    ```java
    implementation 'io.micrometer:micrometer-tracing-bridge-brave'
    implementation 'io.zipkin.reporter2:zipkin-reporter-brave' 
    ```

+   公共配置文件 `config-repo/application.yml` 中的 `management.zipkin.tracing.endpoint` 属性指向 Istio 中的 Jaeger 组件。它具有主机名 `jaeger-collector.istio-system`。

+   在三个 Docker Compose 文件 `docker-compose.yml`，`docker-compose-partitions.yml` 和 `docker-compose-kafka.yml` 中保留了对 Zipkin 服务器的定义，以便能够在 Kubernetes 和 Istio 之外使用分布式跟踪，但 Zipkin 服务器已被赋予与 Istio 中 Jaeger 组件相同的名称，即 `jaeger-collector.istio-system`。

+   已删除 Zipkin 服务器的 Helm 图表。

Jaeger 将在接下来的 *在 Kubernetes 集群中部署 Istio* 部分中安装。

在解释了微服务景观的简化之后，我们已准备好在 Kubernetes 集群中部署 Istio。

# 在 Kubernetes 集群中部署 Istio

在本节中，我们将学习如何在 Kubernetes 集群中部署 Istio 以及如何访问其中的 Istio 服务。

我们将使用 Istio 的 CLI 工具 `istioctl`，使用适合在开发环境中测试 Istio 的 `demo` 配置安装 Istio，即，启用大多数功能但配置为最小化资源使用。

此配置不适合生产使用和性能测试。

对于其他安装选项，请参阅 [`istio.io/latest/docs/setup/install/`](https://istio.io/latest/docs/setup/install/)。

要部署 Istio，请执行以下步骤：

1.  确保上一章中的 Minikube 实例已通过以下命令启动并运行：

    ```java
    minikube status 
    ```

    预期以下类似的响应，前提是它已启动并运行：

![计算机屏幕截图  描述自动生成，置信度中等](img/B19825_18_04.png)

图 18.4：Minikube 状态正常

1.  运行预检查以验证 Kubernetes 集群是否已准备好在其中安装 Istio：

    ```java
    istioctl experimental precheck 
    ```

    预期以下类似的响应：

![计算机屏幕截图  描述自动生成，置信度中等](img/B19825_18_05.png)

图 18.5：Istio 预检查正常

1.  使用以下命令使用 `demo` 配置文件安装 Istio：

    ```java
    cd $BOOK_HOME/Chapter18
    istioctl install --skip-confirmation \
      --set profile=demo \
      --set meshConfig.accessLogFile=/dev/stdout \
      --set meshConfig.accessLogEncoding=JSON \
      --set values.pilot.env.PILOT_JWT_PUB_KEY_REFRESH_INTERVAL=15s \
      -f kubernetes/istio-tracing.yml 
    ```

    命令参数执行以下操作：

+   `accessLog` 参数用于启用 Istio 代理记录处理请求。一旦 Pods 配置了 Istio 代理并启动运行，可以使用命令 `kubectl logs <MY-POD> -c istio-proxy` 检查访问日志。

+   `PILOT_JWT_PUB_KEY_REFRESH_INTERVAL` 参数配置 Istio 的守护进程 `istiod` 每 15 秒刷新获取的 JWKS 公钥。此参数的用法将在 *部署 v1 和 v2 版本的微服务并路由到 v1 版本* 部分中解释。

+   配置文件 `kubernetes/istio-tracing.yml` 启用创建用于分布式跟踪的跟踪跨度。它还配置 Istio 为所有请求创建跟踪跨度。它看起来像这样：

    ```java
    apiVersion: install.istio.io/v1alpha1
    kind: IstioOperator
    spec:
      meshConfig:
        enableTracing: true
        defaultConfig:
          tracing:
            sampling: 100 
    ```

1.  使用以下命令等待 Deployment 对象及其 Pods 可用：

    ```java
    kubectl -n istio-system wait --timeout=600s --for=condition=available deployment --all 
    ```

1.  接下来，使用以下命令安装 *介绍 Istio* 部分中描述的额外组件——Kiali、Jaeger、Prometheus 和 Grafana：

    ```java
    istio_version=$(istioctl version --short --remote=false)
    echo "Installing integrations for Istio v$istio_version"
    kubectl apply -n istio-system -f https://raw.githubusercontent.com/istio/istio/${istio_version}/samples/addons/kiali.yaml
    kubectl apply -n istio-system -f https://raw.githubusercontent.com/istio/istio/${istio_version}/samples/addons/jaeger.yaml
    kubectl apply -n istio-system -f https://raw.githubusercontent.com/istio/istio/${istio_version}/samples/addons/prometheus.yaml
    kubectl apply -n istio-system -f https://raw.githubusercontent.com/istio/istio/${istio_version}/samples/addons/grafana.yaml 
    ```

如果这些命令中的任何一个失败，请尝试重新运行失败的命令。错误可能由于时间问题引起，可以通过再次运行命令来解决。特别是，Kiali 的安装可能会导致以 `unable to recognize` 开头的错误消息。重新运行命令会使这些错误消息消失。

1.  再次等待以下命令以使额外组件可用：

    ```java
    kubectl -n istio-system wait --timeout=600s --for=condition=available deployment --all 
    ```

1.  最后，运行以下命令以查看我们安装了什么：

    ```java
    kubectl -n istio-system get deploy 
    ```

    预期以下类似的输出：

![黑色屏幕的屏幕截图  描述自动生成，置信度低](img/B19825_18_06.png)

图 18.6：Istio 命名空间中的部署

Istio 现在已在 Kubernetes 中部署，但在我们继续创建服务网格之前，我们需要了解如何在 Minikube 环境中访问 Istio 服务。

## 设置访问 Istio 服务的权限

在前一个部分中用于安装 Istio 的`demo`配置包含一些与连接相关的问题，我们需要解决。Istio 入口网关被配置为一个负载均衡的 Kubernetes 服务；也就是说，其类型是`LoadBalancer`。为了能够访问网关，我们需要在 Kubernetes 集群前面运行一个负载均衡器。

Minikube 包含一个可以用来模拟本地负载均衡器的命令，`minikube tunnel`。此命令为每个负载均衡的 Kubernetes 服务分配一个外部 IP 地址，包括 Istio 入口网关。我们在测试中使用的`minikube.me`主机名需要被转换成 Istio 入口网关的外部 IP 地址。为了简化对 Kiali 和 Jaeger 等组件的 Web UI 的访问，我们还将添加专门用于这些服务的域名，例如，`kiali.minikube.me`。

我们还将注册一个主机名到外部`健康`端点，如*观察服务网格*部分所述。最后，还会注册后续章节中安装和使用的服务的几个主机名，这样我们就不需要在以下章节中添加新的主机名。下一章我们将安装的服务包括 Kibana、Elasticsearch 和一个邮件服务器。

要使用这些主机名通过外部访问 Istio 服务，已创建了一个 Helm 图表；请参阅`kubernetes/helm/environments/istio-system`。该图表包含每个 Istio 组件的`Gateway`、`VirtualService`和`DestinationRule`对象。为了保护对这些主机名的请求免受窃听，只允许 HTTPS 请求。在前一章中引入的`cert-manager`由图表用于自动为这些主机名提供 TLS 证书并将其存储在名为`hands-on-certificate`的 Secret 中。所有网关对象都配置为在 HTTPS 协议的配置中使用此 Secret。所有定义文件都可以在 Helm 图表的`templates`文件夹中找到。

这些 API 对象的使用将在下面的*创建服务网格*和*使用 HTTPS 和证书保护外部端点*部分中更详细地描述。

运行以下命令以应用 Helm 图表：

```java
helm upgrade --install istio-hands-on-addons kubernetes/helm/environments/istio-system -n istio-system --wait 
```

这将使网关能够将以下主机名的请求路由到相应的 Kubernetes 服务：

+   `kiali.minikube.me`请求被路由到`kiali:20001`

+   `tracing.minikube.me`请求被路由到`tracing:80`

+   `prometheus.minikube.me`请求被路由到`prometheus:9000`

+   `grafana.minikube.me`请求被路由到`grafana:3000`

为了验证`证书`和`密钥`对象是否已创建，请运行以下命令：

```java
kubectl -n istio-system get secret hands-on-certificate
kubectl -n istio-system get certificate  hands-on-certificate 
```

预期输出如下：

![计算机的截图，中等置信度自动生成描述](img/B19825_18_07.png)

图 18.7：cert-manager 已交付 TLS Secret 和证书

以下图表总结了如何访问组件：

![包含文本、截图、字体、行描述的图片，自动生成](img/B19825_18_08.png)

图 18.8：通过 Minikube 隧道访问组件时要使用的主机名

执行以下步骤以设置 Minikube 隧道并注册主机名：

1.  在另一个终端窗口中运行以下命令（当隧道启动并运行时，该命令会锁定终端窗口）：

    ```java
    minikube tunnel 
    ```

    注意，此命令要求你的用户具有 `sudo` 权限，并且在启动时输入你的密码。在命令请求密码之前可能需要几秒钟，所以很容易错过！

    一旦隧道启动并运行，它将列出 `istio-ingressgateway` 作为其公开的服务之一（在我们的案例中是唯一的）。

1.  配置主机名，使其解析到 Istio 入口网关的 IP 地址。首先，获取 `minikube tunnel` 命令为 Istio 入口网关暴露的 IP 地址，并将其保存到名为 `INGRESS_IP` 的环境变量中：

    ```java
    INGRESS_IP=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    echo $INGRESS_IP 
    ```

    `echo` 命令将打印一个 IP 地址。由于我们使用 Minikube 的 Docker 驱动程序，它始终是 `127.0.0.1`。

1.  更新 `/etc/hosts`，以便所有 `minikube.me` 主机名都将使用 Istio 入口网关的 IP 地址：

    ```java
    MINIKUBE_HOSTS="minikube.me grafana.minikube.me kiali.minikube.me prometheus.minikube.me tracing.minikube.me kibana.minikube.me elasticsearch.minikube.me mail.minikube.me health.minikube.me"
    echo `127.0.0.1` $MINIKUBE_HOSTS" | sudo tee -a /etc/hosts 
    ```

1.  在 Windows 上，我们还需要更新 Windows 的 `hosts` 文件：

    1.  在 Windows 中，打开一个 `PowerShell` 终端。

    1.  使用以下命令在 Visual Code Studio 中打开 Windows 的 `hosts` 文件：

        ```java
        code C:\Windows\System32\drivers\etc\hosts 
        ```

    1.  将类似的行添加到 Windows 的 `hosts` 文件中：

        ```java
        127.0.0.1 minikube.me grafana.minikube.me kiali.minikube.me prometheus.minikube.me tracing.minikube.me kibana.minikube.me elasticsearch.minikube.me mail.minikube.me health.minikube.me 
        ```

    1.  当你尝试保存时，你会得到一个关于 `Insufficient permissions` 的错误。点击 **Retry as Admin...** 按钮以管理员身份更新 `hosts` 文件。

    1.  验证更新：

        ```java
        cat C:\Windows\System32\drivers\etc\hosts 
        ```

默认情况下，当 WSL 重新启动时，`/etc/hosts` 文件会被 Windows 的 `hosts` 文件内容覆盖。重启 WSL 需要很长时间，因为它也会重启 Docker。重启 Docker 又会导致 Minikube 实例停止，因此需要手动重启。为了避免这个缓慢且繁琐的重启过程，我们简单地更新了这两个文件。

1.  删除 `/etc/hosts` 中指向 Minikube 实例 IP 地址（`127.0.0.1`）的 `minikube.me` 行。验证 `/etc/hosts` 只包含一行，将 `minikube.me` 转换为指向 Istio 入口网关的 IP 地址，即 `127.0.0.1`：

![计算机的截图，中等置信度自动生成描述](img/B19825_18_09.png)

图 18.9：已更新的 `/etc/hosts` 文件

1.  使用以下命令验证 Kiali、Jaeger、Grafana 和 Prometheus 是否可以通过隧道访问：

    ```java
    curl -o /dev/null -sk -L -w "%{http_code}\n" https://kiali.minikube.me/kiali/
    curl -o /dev/null -sk -L -w "%{http_code}\n" https://tracing.minikube.me
    curl -o /dev/null -sk -L -w "%{http_code}\n" https://grafana.minikube.me
    curl -o /dev/null -sk -L -w "%{http_code}\n" https://prometheus.minikube.me/graph#/ 
    ```

每个命令都应该返回 `200`（`OK`）。如果发送给 Kiali 的请求没有返回 `200`，通常意味着其内部初始化尚未完成。在这种情况下，请等待一分钟，然后重试。

如果例如您的计算机或 Minikube 实例被暂停或重启，`minikube tunnel` 命令将停止运行。在这些情况下，需要手动重启。因此，如果您在 `minikube.me` 任何主机名上无法调用 API，请始终检查 Minikube 隧道是否正在运行，并在需要时重启它。

在 Minikube 隧道就绪后，我们现在可以创建服务网格了。

# 创建服务网格

部署了 Istio 后，我们就可以创建服务网格了。创建服务网格所需的步骤基本上与我们用于 *第十七章*，*通过实现 Kubernetes 功能简化系统景观*（参考 *使用 Kubernetes ConfigMaps、Secrets、Ingress 和 cert-manager 进行测试* 部分）中使用的步骤相同。在运行创建服务网格的命令之前，让我们先看看 Helm 模板中为设置服务网格所做的添加。

## 源代码更改

为了能够在由 Istio 管理的服务网格中运行微服务，`dev-env` Helm 图表从 `common` 图表引入了两个新的命名模板 `_istio_base.yaml` 和 `_istio_dr_mutual_tls.yaml`。让我们逐一查看它们。

### `_istio_base.yaml` 模板中的内容

`_istio_base.yaml` 定义了将被 `dev-env` 和 `prod-env` 环境图表共同使用的多个 Kubernetes 清单。首先，它定义了三个与 Istio 相关的安全相关清单：

+   一个名为 `product-composite-require-jwt` 的 `AuthorizationPolicy` 清单

+   一个名为 `default` 的 `PeerAuthentication` 清单

+   一个名为 `product-composite-request-authentication` 的 `RequestAuthentication` 清单

下面的 *Securing a service mesh* 部分将解释这三个清单。

剩下的四个清单将在这里讨论。它们是两对用于配置访问和从主机名 `minikube.me` 和 `health.minikube.me` 路由的 `Gateway` 和 `VirtualService` 清单。将使用 Gateway 对象来定义如何接收外部流量，而 `VirtualService` 对象用于描述如何在服务网格内部路由传入的流量。

控制访问 `minikube.me` 的 `Gateway` 清单如下所示：

```java
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: hands-on-gw
spec:
  selector:
    istio: ingressgateway
  servers:
  - hosts:
    - minikube.me
    port:
      name: https
      number: 443
      protocol: HTTPS
    tls:
      credentialName: hands-on-certificate
      mode: SIMPLE 
```

这里有一些关于源代码的解释：

+   网关命名为 `hands-on-gw`；这个名称被下方的虚拟服务所使用。

+   `selector` 字段指定网关对象将由默认的 Istio 入口网关，名为 `ingressgateway`，处理。

+   `hosts` 和 `port` 字段指定网关将使用 HTTPS 通过端口 `443` 处理对 `minikube.me` 主机名的传入请求。

+   `tls`字段指定 Istio 入口网关可以在名为`hands-on-certificate`的 TLS Secret 中找到用于 HTTPS 通信的证书和私钥。有关如何创建这些证书文件的详细信息，请参阅下面的`Protecting external endpoints with HTTPS and certificates`部分。`SIMPLE`模式表示将应用正常的 TLS 语义。

路由发送到`minikube.me`的请求的`VirtualService`清单如下所示：

```java
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: hands-on-vs
spec:
  gateways:
  - hands-on-gw
  hosts:
  - minikube.me
  http:
  - match:
    - uri:
        prefix: /oauth2
    route:
    - destination:
        host: auth-server
  – match:
    ... 
```

前面清单的解释如下：

+   `gateways`和`hosts`字段指定虚拟服务将通过`hands-on-gw`网关路由发送到`minikube.me`主机名的请求。

+   在`http`元素之后是一个`match`和`route`块的数组，指定了 URL 路径将如何转发到相关的 Kubernetes 服务。在上面的清单中，只显示了第一对`match`和`route`元素。它们将使用路径`/oauth2`发送到`auth-server`服务的请求映射。这种映射应该与我们在前几章中指定路由规则的方式相似。其余的`match`和`route`元素配置了与我们在 Spring Cloud Gateway 和 Ingress 对象中看到的相同的路由规则：

    +   `/login → auth-server`

    +   `/error → auth-server`

    +   `/product-composite → product-composite`

    +   `/openapi → product-composite`

    +   `/webjars → product-composite`

有关详细信息，请参阅`kubernetes/helm/common/templates/_istio_base.yaml`。

在前面的源代码中，使用其短名称指定了目标主机，换句话说，是`product-composite`。这有效，因为示例基于同一命名空间`hands-on`的 Kubernetes 定义。如果不是这种情况，建议在 Istio 文档中使用主机的**完全限定域名**（**FQDN**）代替。在这种情况下，它是`product-composite.hands-on.svc.cluster.local`。

### `_istio_dr_mutual_tls.yaml`模板中的内容

`_istio_dr_mutual_tls.yaml`定义了一个模板，用于指定多个`DestinationRule`对象。它用于指定在路由请求到其对应服务时应使用 mTLS。它还可以选择性地用于指定`subsets`，我们将在下面的`Performing zero-downtime updates`部分中的`prod-env`图表中使用它。模板看起来如下所示：

```java
{{- define "common.istio_dr_mutual_tls" -}}
{{- range $idx, $dr := .Values.destinationRules }}
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: {{ $dr.name }}
spec:
  host: {{ $dr.name }}
{{- if $dr.subsets }}
{{- with $dr.subsets }}
  subsets:
{{ toYaml . | indent 2 }}
{{- end }}
{{- end }}
  trafficPolicy:
    tls:
      mode: ISTIO_MUTUAL
---
{{- end -}}
{{- end -}} 
```

下面是一些关于前面模板的注释：

+   `range`指令遍历在`destinationRules`变量中定义的元素

+   清单的`spec`部分的`host`字段用于指定此`DestinationRule`应用到的 Kubernetes Service 的名称

+   如果在`destinationRules`列表中的当前元素`$dr`中找到相应的元素，则仅定义`subsets`部分

+   总是使用`trafficPolicy`来强制要求 mTLS

模板在 `dev-end` Helm 图表中使用，通过在 `values.yaml` 文件中指定 `destinationRules` 变量如下：

```java
destinationRules:
  - name: product-composite
  - name: auth-server
  - name: product
  - name: recommendation
  - name: review 
```

文件位于 `kubernetes/helm/common/templates/_istio_dr_mutual_tls.yaml` 和 `kubernetes/helm/environments/dev-env/values.yaml`。

在源代码中进行这些更改后，我们现在可以创建服务网格。

## 运行命令以创建服务网格

通过运行以下命令创建服务网格：

1.  使用以下命令从源代码构建 Docker 镜像：

    ```java
    cd $BOOK_HOME/Chapter18
    eval $(minikube docker-env -u)
    ./gradlew build
    eval $(minikube docker-env)
    docker-compose build 
    ```

`eval $(minikube docker-env -u)` 命令确保 `./gradlew build` 命令使用主机的 Docker 引擎，而不是 Minikube 实例中的 Docker 引擎。`build` 命令使用 Docker 运行测试容器。

1.  重新创建 `hands-on` 命名空间，并将其设置为默认命名空间：

    ```java
    kubectl delete namespace hands-on
    kubectl apply -f kubernetes/hands-on-namespace.yml
    kubectl config set-context $(kubectl config current-context) --namespace=hands-on 
    ```

    注意，`hands-on-namespace.yml` 文件创建了一个带有 `istio-injection: enabled` 标签的 `hands-on` 命名空间。这意味着在此命名空间中创建的 Pod 将自动注入 `istio-proxy` 容器作为边车。

1.  使用以下命令解决 Helm 图表依赖项：

    1.  首先，我们更新 `components` 文件夹中的依赖项：

        ```java
        for f in kubernetes/helm/components/*; do helm dep up $f; done 
        ```

    1.  接下来，我们更新 `environments` 文件夹中的依赖项：

        ```java
        for f in kubernetes/helm/environments/*; do helm dep up $f; done 
        ```

1.  使用 Helm 部署系统架构，并等待所有 Deployment 完成：

    ```java
    helm install hands-on-dev-env \
      kubernetes/helm/environments/dev-env \
      -n hands-on --wait 
    ```

1.  一旦 Deployment 完成，验证每个微服务 Pod 中都有两个容器：

    ```java
    kubectl get pods 
    ```

    预期响应将与以下内容类似：

    ![计算机程序的截图  描述由中等置信度自动生成](img/B19825_18_10.png)

    图 18.10：Pod 启动并运行

    注意，运行我们的微服务的 Pod 报告每个 Pod 有两个容器；也就是说，它们注入了 Istio 代理作为边车！

1.  使用以下命令运行常规测试：

    ```java
    ./test-em-all.bash 
    ```

    `test-em-all.bash` 脚本的默认值已从前面的章节更新，以适应在 Minikube 中运行的 Kubernetes。

    预期输出将与我们在前面的章节中看到的内容相似：

    ![计算机的截图  描述由中等置信度自动生成](img/B19825_18_11.png)

    图 18.11：测试运行成功

1.  在我们开始尝试 Istio 及其各种组件之前，让我们看看如何使用上面 *用 Istio 的 Jaeger 组件替换 Zipkin 服务器* 部分中提到的 B3 标头来记录跟踪和跨度 ID 的传播。

## 跟踪和跨度 ID 的日志传播

我们可以在 `product-composite` 微服务的出站请求中看到跟踪和跨度 ID，就像我们在 *第十四章，理解分布式跟踪* 中的 *发送成功的 API 请求* 部分所做的那样。由于我们现在在 Kubernetes 中运行微服务，我们需要更改 ConfigMap 中的日志配置，然后删除正在运行的 Pod 以使其影响微服务：

1.  使用以下命令编辑 ConfigMap：

    ```java
    kubectl edit cm product-composite 
    ```

    查找以下行：

    ```java
    # To see tracing headers, uncomment the following two lines and restart the product-composite service
    # spring.codec.log-request-details: true
    # logging.level.org.springframework.web.reactive.function.client.ExchangeFunctions: TRACE 
    ```

1.  取消注释这两行最后的注释并退出编辑器。

1.  使用以下命令重启 `product-composite` 微服务，通过删除其 Pod：

    ```java
    kubectl delete pod -l app=product-composite 
    ```

1.  使用以下命令将日志输出打印到终端窗口：

    ```java
    kubectl logs -f -l app=product-composite 
    ```

1.  获取一个访问令牌并使用访问令牌进行请求：

    ```java
    unset ACCESS_TOKEN
    ACCESS_TOKEN=$(curl -k https://writer:secret-writer@minikube.me/oauth2/token -d grant_type=client_credentials -d scope="product:read product:write" -s | jq -r .access_token)
    echo $ACCESS_TOKEN
    curl -H "Authorization: Bearer $ACCESS_TOKEN" -k https://minikube.me/product-composite/1 -w "%{http_code}\n" -o /dev/null -s 
    ```

    验证命令返回的 HTTP 状态码是否为成功，即 `200`。

1.  在日志输出中，应该可以看到如下类似的行：

    ```java
    2023-05-15T15:39:25.919Z TRACE [product-composite,01bd9fb5815a7889dea69ec33afee5c5,94d9157ae179554c] 1 --- [     parallel-1] o.s.w.r.f.client.ExchangeFunctions       : [14b00bcd] HTTP GET http://product/
    product/1?delay=0&faultPercent=0, headers=[X-B3-TraceId:"01bd9fb5815a7889dea69ec33afee5c5", X-B3-SpanId:"94d9157ae179554c", X-B3-ParentSpanId:"aa3e97771ef9155e", X-B3-Sampled:"1"] 
    ```

    在上面的示例日志输出中，我们可以看到标准的 `B3` 头部，如 `X-B3-TraceId` 和 `X-B3-SpanId`。

1.  通过在 ConfigMap 中重新添加注释并删除其 Pod 来重启微服务，以停止记录跟踪和跨度 ID。

在服务网格运行起来之后，让我们看看如何使用 Kiali 来观察其中的情况！

# 观察服务网格

在本节中，我们将使用 Kiali 与 Jaeger 一起观察服务网格中的情况。

在我们这样做之前，我们需要了解如何消除 Kubernetes 的存活和就绪探针执行的健康检查产生的噪音。在之前的章节中，它们使用了与 API 请求相同的端口。这意味着 Istio 将收集健康检查和发送到 API 的请求的使用情况指标。这将导致 Kiali 显示的图表变得不必要地杂乱。Kiali 可以过滤掉我们不感兴趣的流量，但一个更简单的解决方案是为健康检查使用不同的端口。

微服务可以被配置为使用单独的端口来处理发送到 actuator 端点的请求，例如发送到 `/actuator/health` 端点的健康检查。以下行已被添加到所有微服务的通用配置文件 `config-repo/application.yml` 中：

```java
management.server.port: 4004 
```

这将使所有微服务使用端口 `4004` 来暴露 `health` 端点。`common` Helm 图表的 `values.yaml` 文件已被更新，以在默认的存活和就绪探针中使用端口 `4004`。请参阅 `kubernetes/helm/common/values.yaml`。

`product-composite` 微服务不仅将其管理端口暴露给 Kubernetes 探针，还对外部健康检查进行暴露，例如由 `test-em-all.bash` 执行的健康检查。这是通过 Istio 的入口网关完成的，因此端口 `4004` 被添加到 `product-composite` 微服务的 Deployment 和 Service 清单中。请参阅 `kubernetes/helm/components/product-composite/values.yaml` 中的 `ports` 和 `service.ports` 定义。

Spring Cloud Gateway（保留以在 Docker Compose 中运行测试）将继续使用相同的端口来处理 API 请求和 `health` 端点的请求。在 `config-repo/gateway.yml` 配置文件中，`management` 端口被恢复为 API 所使用的端口：

```java
management.server.port: 8443 
```

为了简化对由 `product-composite` 微服务暴露的健康检查的外部访问，为 `health.minikube.me` 主机名配置了一个路由到 `product-composite` 微服务的 `management` 端口。请参阅上面 `_istio_base.yaml` 模板的说明。

在发送到 `health` 端点的请求处理完毕后，我们可以开始通过服务网格发送一些请求。

我们将使用 `siege` 启动低流量负载测试，我们在第十六章 *将我们的微服务部署到 Kubernetes* 中了解到它。之后，我们将查看 Kiali 的几个重要部分，以了解它如何用于在 Web 浏览器中观察服务网格。我们还将了解 Jaeger 如何用于分布式跟踪。

由于我们使用的证书是自签名的，因此网络浏览器不会自动依赖它。大多数网络浏览器允许您访问网页，如果您向它们保证您理解安全风险。如果网络浏览器拒绝，打开私密窗口在某些情况下可能会有帮助。

特别是关于 Chrome，如果它不允许您访问网页，并说 **您的连接不安全**，您可以点击 **高级** 按钮，然后点击链接 **继续访问 …（不安全）**。

使用以下命令启动测试客户端：

```java
ACCESS_TOKEN=$(curl https://writer:secret-writer@minikube.me/oauth2/token -d grant_type=client_credentials -d scope="product:read product:write" -ks | jq .access_token -r)
echo ACCESS_TOKEN=$ACCESS_TOKEN
siege https://minikube.me/product-composite/1 -H "Authorization: Bearer $ACCESS_TOKEN" -c1 -d1 -v 
```

第一个命令将获取一个 OAuth 2.0/OIDC 访问令牌，该令牌将在下一个命令中使用，其中 `siege` 用于每秒提交一个 HTTP 请求到 `product-composite` API。

预期 `siege` 命令的输出如下：

![计算机屏幕截图  描述由中等置信度自动生成](img/B19825_18_12.png)

图 18.12：系统在攻击下

使用您选择的接受自签名证书的 Web 浏览器，并按照以下步骤操作：

1.  使用[`kiali.minikube.me`](https://kiali.minikube.me) URL 打开 Kiali 的 Web UI。默认情况下，您将以匿名用户身份登录。预期网页将与以下类似：

![计算机屏幕截图  描述自动生成](img/B19825_18_13.png)

图 18.13：Kiali Web UI

1.  如果 **概览** 选项卡尚未激活，请点击它。

1.  点击名为 **动手实践**（右上角三个垂直点）的菜单，并选择 **图形**。预期会显示一个图形，表示当前通过服务网格流动的流量。

1.  点击 **显示** 按钮，取消选择除 **响应时间**、**中位数** 和 **流量动画** 之外的所有选项。

1.  在 **隐藏…** 字段中，指定 `name = jaeger` 以避免将发送到 Jaeger 的跟踪视图搞乱。

1.  Kiali 现在显示一个图形，表示当前通过服务网格发送的请求，其中活动请求由箭头旁的小移动圆圈表示，如下所示：

![计算机屏幕截图  描述由中等置信度自动生成](img/B19825_18_14.png)

图 18.14：显示动手实践命名空间的 Kiali 图表

1.  从**未知**到**auth-server**的流量代表了对授权服务器进行调用以获取 JWKS 公钥的请求。

    这为服务网格中正在发生的事情提供了一个相当好的初步概述！

1.  现在我们来看看使用 Jaeger 的一些分布式追踪。使用[`tracing.minikube.me`](https://tracing.minikube.me) URL 打开 Web UI。在左侧菜单中点击**服务**下拉菜单并选择**istio-ingressgateway.istio-system**服务。点击**查找追踪**按钮，你应该会看到一个如下所示的结果：

![计算机屏幕截图  描述由自动生成](img/B19825_18_15.png)

图 18.15：Jaeger 可视化的分布式追踪

1.  点击报告包含**23 个跨度**的其中一个追踪来检查它。预期会看到一个如下所示的网页：

![计算机屏幕截图  描述由中等置信度自动生成](img/B19825_18_16.png)

图 18.16：Jaeger 中完整追踪调用树的视图

这基本上与 Zipkin 在*第十四章*，*理解分布式追踪*中提供的追踪信息相同。请注意，我们可以看到来自 Istio 代理和微服务本身的追踪信息。由 Istio 代理报告的跨度后面跟随着 Kubernetes 命名空间，即，`.istio-system`和`.hands-on`。

还有更多内容可以探索，但作为介绍已经足够了。请随意探索 Kiali 和 Jaeger 上的 Web UI。

注意，为测试客户端`siege`获取的访问令牌仅在一小时内有效。如果流量意外下降，检查`siege`的输出；如果它报告`4XX`而不是`200`，那么是时候更新访问令牌了！

让我们继续学习如何使用 Istio 在服务网格中提高安全性！

# 保护服务网格

在本节中，我们将学习如何使用 Istio 来提高服务网格的安全性。我们将涵盖以下主题：

+   如何使用 HTTPS 和证书保护外部端点

+   如何要求外部请求使用 OAuth 2.0/OIDC 访问令牌进行认证

+   如何使用**相互认证**（**mTLS**）保护内部通信

现在我们将在以下章节中了解这些内容的每一个。

## 使用 HTTPS 和证书保护外部端点

从*设置对 Istio 服务的访问*和*`_istio_base.yaml`模板中的内容*部分，我们了解到网关对象使用存储在名为`hands-on-certificate`的 Secret 中的 TLS 证书为其 HTTPS 端点。

该 Secret 是由 cert-manager 根据`istio-system` Helm 图中的配置创建的。图表的模板`selfsigned-issuer.yaml`用于定义一个内部自签名 CA，并具有以下内容：

```java
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: selfsigned-issuer
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: ca-cert
spec:
  isCA: true
  commonName: hands-on-ca
  secretName: ca-secret
  issuerRef:
    name: selfsigned-issuer
---
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: ca-issuer
spec:
  ca:
    secretName: ca-secret 
```

从前面的清单中，我们可以看到以下内容：

+   一个名为`selfsigned-issuer`的自签名发行者。

+   此发行者用于创建一个名为`ca-cert`的自签名证书。

+   证书被赋予通配名`hands-on-ca`。

+   最后，使用证书 `ca-cert` 作为其根证书定义了一个自签名 CA，`ca-issuer`。此 CA 将用于签发网关对象使用的证书。

图表的模板 `hands-on-certificate.yaml` 将此证书定义为：

```java
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: hands-on-certificate
spec:
  commonName: minikube.me
  subject:
    ...
  dnsNames:
  - minikube.me
  - health.minikube.m
  - dashboard.minikube.me
  - kiali.minikube.me
  - tracing.minikube.me
  - prometheus.minikube.me
  - grafana.minikube.me
  - kibana.minikube.me
  - elasticsearch.minikube.me
  - mail.minikube.me
  issuerRef:
    name: ca-issuer
  secretName: hands-on-certificate 
```

从此清单中，我们可以了解到：

+   证书命名为 `hands-on-certificate`

+   其通用名称设置为 `minikube.me`

+   它指定了一些关于其 `subject` 的可选额外详细信息（为了清晰起见省略）

+   所有其他主机名都在证书中声明为 **Subject Alternative Names**

+   它将使用上面声明的名为 `ca-issuer` 的发行者

+   `cert-manager` 将 TLS 证书存储在名为 `hands-on-certificate` 的 Secret 中

当 `istio-system` Helm 图表安装时，这些模板被用来在 Kubernetes 中创建相应的 API 对象。这触发了 `cert-manager` 创建证书和 Secrets。

模板文件可以在 `kubernetes/helm/environments/istio-system/templates` 文件夹中找到。

要验证 Istio 入口网关使用的是这些证书，我们可以运行以下命令：

```java
keytool -printcert -sslserver minikube.me | grep -E "Owner:|Issuer:" 
```

预期以下输出：

![计算机屏幕截图  描述由中等置信度自动生成](img/B19825_18_17.png)

图 18.17：检查 minikube.me 的证书

输出显示，该证书是为通用名称 `minikube.se` 签发的，并且是由我们自己的 CA 签发的，使用其通用名称为 `hands-on-ca` 的根证书。

如 *第十七章*，*实现 Kubernetes 功能以简化系统景观*（参见 *自动化证书供应* 部分）所述，此自签名 CA 需要替换为生产用例中的 Let’s Encrypt 或其他 `cert-manager` 可以用来提供信任证书的 CA。

在验证证书配置后，让我们继续看看 Istio 入口网关如何保护微服务免受未认证请求。

## 使用 OAuth 2.0/OIDC 访问令牌验证外部请求

Istio 入口网关可以要求并验证基于 JWT 的 OAuth 2.0/OIDC 访问令牌，换句话说，保护服务网格中的微服务免受外部未认证请求。关于 JWT、OAuth 2.0 和 OIDC 的概述，请参阅 *第十一章*，*保护 API 访问*（见 *使用 OAuth 2.0 和 OpenID Connect 保护 API* 部分）。Istio 还可以配置为执行授权，但如 *介绍 Istio API 对象* 部分所述，我们不会使用它。

这是在 `common` Helm 图表的模板 `_istio_base.yaml` 中配置的。两个清单看起来如下：

```java
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: product-composite-request-authentication
spec:
  jwtRules:
  - forwardOriginalToken: true
    issuer: http://auth-server
    jwksUri: http://auth-server.hands-on.svc.cluster.local/oauth2/jwks
  selector:
    matchLabels:
      app.kubernetes.io/name: product-composite
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: product-composite-require-jwt
spec:
  action: ALLOW
  rules:
  - {}
  selector:
    matchLabels:
      app.kubernetes.io/name: product-composite 
```

从清单中，我们可以看到以下内容：

+   命名为 `product-composite-request-authentication` 的 `RequestAuthentication` 要求对发送到 `product-composite` 服务的请求使用有效的 JWT 编码访问令牌。

+   它根据标签选择器选择它执行请求认证的服务，`app.kubernetes.io/name: product-composite`。

+   它允许来自提供者`http://auth-server`的令牌。

+   它将使用`http://auth-server.hands-on.svc.cluster.local/oauth2/jwks` URL 来获取 JWKS。密钥集用于验证访问令牌的数字签名。

+   它将访问令牌转发到底层服务，在我们的例子中，是`product-composite`微服务。

+   命名为`product-composite-require-jwt`的`AuthorizationPolicy`配置为允许对`product-composite`服务的所有请求；它不会应用任何授权规则。

要理解 Istio 的`RequestAuthentication`是否正在验证访问令牌，或者只是`product-composite`服务正在执行验证可能有点困难。确保 Istio 正在执行其工作的一种方法是将`RequestAuthentication`的配置更改为始终拒绝访问令牌。

为了验证`RequestAuthentication`正在生效，应用以下命令：

1.  发送一个正常请求：

    ```java
    ACCESS_TOKEN=$(curl https://writer:secret-writer@minikube.me/oauth2/token  -d grant_type=client_credentials -d scope="product:read product:write" -ks | jq .access_token -r)
    echo ACCESS_TOKEN=$ACCESS_TOKEN
    curl -k https://minikube.me/product-composite/1 -H "Authorization: Bearer $ACCESS_TOKEN" -i 
    ```

    验证它返回的 HTTP 响应状态码为`200`（`OK`）。

1.  编辑`RequestAuthentication`对象，并暂时更改`issuer`，例如，更改为`http://auth-server-x`：

    ```java
    kubectl edit RequestAuthentication product-composite-request-authentication 
    ```

1.  验证更改：

    ```java
    kubectl get RequestAuthentication product-composite-request-authentication -o yaml 
    ```

    验证`issuer`是否已更新，在我的情况下，更新为`http://auth-server-x`。

1.  再次发送请求。它应该以 HTTP 响应状态码`401`（未授权）和错误消息`Jwt issuer is not configured`失败：

    ```java
    curl -k https://minikube.me/product-composite/1 -H "Authorization: Bearer $ACCESS_TOKEN" -i 
    ```

    由于 Istio 传播更改需要几秒钟，您可能需要重复命令几次，直到它失败。

    这证明了 Istio 正在验证访问令牌！

1.  将`issuer`的更改名称恢复为`http://auth-server`：

    ```java
    kubectl edit RequestAuthentication product-composite-request-authentication 
    ```

1.  验证请求是否再次工作。首先，等待几秒钟以使更改传播。然后，运行以下命令：

    ```java
    curl -k https://minikube.me/product-composite/1 -H "Authorization: Bearer $ACCESS_TOKEN" 
    ```

**建议的附加练习**：尝试 Auth0 OIDC 提供者，如第十一章中所述的*保护 API 访问*（参考*使用外部 OpenID Connect 提供者进行测试*部分）。将您的 Auth0 提供者添加到`jwt-authentication-policy.yml`中。在我的情况下，它如下所示：

```java
 - jwtRules:
      issuer: "https://dev-magnus.eu.auth0.com/" 
      jwksUri: "https://dev-magnus.eu.auth0.com/.well-known/jwks.json" 
```

现在，让我们继续讨论在 Istio 中将要覆盖的最后一个安全机制：使用双向认证，mTLS 自动保护服务网格中的内部通信。

## 使用双向认证（mTLS）保护内部通信

在本节中，我们将学习如何配置 Istio 以自动使用**mTLS**保护服务网格内部的通信。在使用双向认证时，不仅服务通过暴露证书来证明其身份，客户端也通过暴露客户端证书来向服务证明其身份。这比仅证明服务身份的正常 TLS/HTTPS 使用提供了更高的安全性。设置和维护双向认证，即为客户分配新证书和轮换过期的证书，被认为是复杂的，因此很少使用。Istio 完全自动化了服务网格内部通信所使用的双向认证证书的分配和轮换。

与手动设置相比，这使得使用双向认证变得容易得多。

那么，为什么我们应该使用双向认证呢？使用 HTTPS 和 OAuth 2.0/OIDC 访问令牌保护外部 API 不是足够了吗？

只要攻击是通过外部 API 发起的，这可能就足够了。但如果 Kubernetes 集群内的 Pod 被攻陷呢？例如，如果攻击者控制了一个 Pod，他们可以开始监听 Kubernetes 集群中其他 Pod 之间的流量。如果内部通信以明文形式发送，攻击者将很容易获取集群中 Pod 之间发送的敏感信息。为了最小化此类入侵造成的损害，可以使用双向认证来防止攻击者窃听内部网络流量。

要启用由 Istio 管理的双向认证的使用，Istio 需要在服务器端配置，使用名为`PeerAuthentication`的策略，并在客户端使用`DestinationRule`。

该策略配置在`common` Helm 图表的模板`_istio_base.yaml`中。其配置文件看起来像这样：

```java
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
spec:
  mtls:
    mode: PERMISSIVE 
```

如在*介绍 Istio API 对象*部分中提到的，`PeerAuthentication`策略配置为使用`PERMISSIVE`模式允许同时进行 mTLS 和平文 HTTP 请求。这使 Kubernetes 能够使用平文 HTTP 调用存活和就绪探测。

我们已经在*`_istio_dr_mutual_tls.yaml`模板中的内容*部分遇到了`DestinationRule`配置文件。要求 mTLS 的`DestinationRule`配置文件的核心部分如下所示：

```java
 trafficPolicy:
    tls:
      mode: ISTIO_MUTUAL 
```

要验证内部通信是否由 mTLS 保护，请执行以下步骤：

1.  确保在前面的*观察服务网格*部分启动的负载测试仍在运行并报告`200`（`OK`）。

1.  在网页浏览器中访问 Kiali 图([`kiali.minikube.me`](https://kiali.minikube.me))。

1.  点击**显示**按钮并启用**安全**标签。图表将显示所有由 Istio 的自动双向认证保护的数据通信链接上的锁状图标，如下所示：

![计算机截图，描述自动生成，置信度中等](img/B19825_18_18.png)

图 18.18：在 Kiali 中检查 mTLS 设置

预期所有链接都将显示一个锁形图标。

对 RabbitMQ、MySQL 和 MongoDB 的调用不由 Istio 代理处理，因此如果需要，需要手动配置以使用 TLS 进行保护。

通过这种方式，我们已经看到了 Istio 中所有三种安全机制的实际应用，现在是时候看看 Istio 如何帮助我们验证服务网格的弹性了。

# 确保服务网格具有弹性

在本节中，我们将学习如何使用 Istio 确保服务网格具有弹性，也就是说，它能够处理服务网格中的临时故障。Istio 提供了类似于 Spring 框架提供的超时、重试以及一种称为**异常检测**的断路器类型来处理临时故障。

当涉及到决定是否应该使用语言原生机制来处理临时故障，或者是否应该将其委托给像 Istio 这样的服务网格时，我倾向于更喜欢使用语言原生机制，就像 *第十三章*，*使用 Resilience4j 提高弹性* 中的例子一样。在许多情况下，保持错误处理的逻辑，例如，处理断路器的回退选项，与其他微服务的业务逻辑一起是很重要的。将处理临时故障的逻辑保留在源代码中也使得使用 JUnit 和测试容器等工具进行测试变得更加容易，如果将处理临时故障的任务委托给像 Istio 这样的服务网格，这个过程会变得更加复杂。

有时候，Istio 中的相应机制可能非常有帮助。例如，如果一个微服务已经部署，并且确定它无法处理生产中偶尔发生的临时故障，那么使用 Istio 添加超时或重试机制可能非常方便，而不是等待带有相应错误处理功能的微服务新版本发布。

Istio 在弹性领域提供的另一个功能是向现有的服务网格中注入故障和延迟的能力。我们为什么想要这样做呢？

以受控的方式注入故障和延迟对于验证微服务的弹性功能是否按预期工作非常有用！我们将在本节中尝试这些功能，验证 `product-composite` 微服务中的重试、超时和断路器是否按预期工作。

在 *第十三章*，*使用 Resilience4j 提高弹性*（参考 *添加可编程延迟和随机错误* 部分）中，我们添加了对将故障和延迟注入微服务源代码的支持。最好用 Istio 的功能来替换这部分源代码，以便在运行时注入故障和延迟，如下面的子节所示。

我们将首先注入故障以查看 `product-composite` 微服务中的重试机制是否按预期工作。之后，我们将延迟 `product` 服务的响应并验证断路器是否按预期处理延迟。

## 通过注入故障测试弹性

让我们让 `product` 服务抛出随机错误，并验证微服务景观是否正确处理这种情况。我们预计 `product-composite` 微服务中的重试机制将启动并重试请求，直到成功或达到最大重试次数限制。这将确保短暂的故障不会比重试尝试引入的延迟更影响最终用户。请参阅 *第十三章*，*使用 Resilience4j 提高弹性* 中的 *添加重试机制* 部分，以回顾 `product-composite` 微服务中的重试机制。

可以使用类似 `kubernetes/resilience-tests/product-virtual-service-with-faults.yml` 的虚拟服务注入故障。如下所示：

```java
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: product
spec:
  hosts:
    - product
  http:
  - route:
    - destination:
        host: product
    fault:
      abort:
        httpStatus: 500
        percentage:
          value: 20 
```

定义说明，发送到 `product` 服务的 `20`% 请求将使用 HTTP 状态码 `500` (`Internal Server Error`) 被终止。

执行以下步骤以测试此功能：

1.  确保使用 `siege` 进行的负载测试，如 *观察服务网格* 部分中启动的，正在运行。

1.  使用以下命令应用故障注入：

    ```java
    kubectl apply -f kubernetes/resilience-tests/product-virtual-service-with-faults.yml 
    ```

1.  监控 `siege` 负载测试工具的输出。期望输出类似于以下内容：![计算机屏幕截图  自动生成的描述](img/B19825_18_19.png)

    图 18.19：观察重试机制在行动

    从样本输出中，我们可以看到所有请求仍然成功，换句话说，返回了状态 `200` (`OK`)；然而，其中一些（20%）需要额外一秒钟来完成。这表明 `product-composite` 微服务中的重试机制已经启动，并重试了对 `product` 服务的失败请求。

1.  使用以下命令移除故障注入以结束测试：

    ```java
    kubectl delete -f kubernetes/resilience-tests/product-virtual-service-with-faults.yml 
    ```

现在让我们进入下一节，我们将注入延迟以触发断路器。

## 通过注入延迟测试弹性

从 *第十三章*，*使用 Resilience4j 提高弹性* 我们知道，断路器可以用来防止在接收请求后，由于服务响应缓慢或完全无响应而产生的问题。

让我们通过使用 Istio 向 `product` 服务注入延迟来验证 `product-composite` 服务中的断路器是否按预期工作。可以使用虚拟服务注入延迟。

请参阅 `kubernetes/resilience-tests/product-virtual-service-with-delay.yml`。其代码如下：

```java
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: product
spec:
  hosts:
    - product
  http:
  - route:
    - destination:
        host: product
    fault:
      delay:
        fixedDelay: 3s
        percent: 100 
```

此定义说明，发送到 `product` 服务的所有请求都将延迟 `3` 秒。

从`product-composite`服务发送到`product`服务的请求被配置为在 2 秒后超时。断路器被配置为在连续三次请求失败时打开电路。当电路打开时，它将快速失败；换句话说，它将立即抛出异常，不会尝试调用底层服务。`product-composite`微服务中的业务逻辑将捕获此异常并应用回退逻辑。为了回顾，请参阅*第十三章*，*使用 Resilience4j 提高弹性*（参考*添加断路器和时间限制器*部分）。

按照以下步骤通过注入延迟来测试断路器：

1.  在运行`siege`的终端窗口中按*Ctrl* + *C*停止负载测试。

1.  使用以下命令在`product`服务中创建临时延迟：

    ```java
    kubectl apply -f kubernetes/resilience-tests/product-virtual-service-with-delay.yml 
    ```

1.  按照以下方式获取访问令牌：

    ```java
    ACCESS_TOKEN=$(curl https://writer:secret-writer@minikube.me/oauth2/token  -d grant_type=client_credentials -d scope="product:read product:write" -ks | jq .access_token -r)
    echo ACCESS_TOKEN=$ACCESS_TOKEN 
    ```

1.  连续发送六个请求：

    ```java
    for i in {1..6}; do time curl -k https://minikube.me/product-composite/1 -H "Authorization: Bearer $ACCESS_TOKEN"; done 
    ```

    预期以下情况：

    1.  断路器在第一次三次失败调用后打开

    1.  断路器对最后三次调用应用快速失败逻辑

    1.  最后三次调用返回回退响应

    预期前三次调用的响应将是一个与超时相关的错误消息，响应时间为 2 秒（换句话说，超时时间）。预期前三次调用的响应如下：

    ![包含文本、屏幕截图、字体的图片，描述自动生成](img/B19825_18_20.png)

    图 18.20：观察超时

    预期最后三次调用的响应将来自回退逻辑，响应时间较短。预期最后三次调用的响应如下：

    ![一张包含计算机屏幕截图的图片，描述自动生成，置信度中等](img/B19825_18_21.png)

    图 18.21：回退方法在行动

1.  使用以下命令通过移除临时延迟来模拟延迟问题已被解决：

    ```java
    kubectl delete -f kubernetes/resilience-tests/product-virtual-service-with-delay.yml 
    ```

1.  通过使用*步骤 4*中的`for`循环命令发送新的请求，验证是否能够再次返回正确答案，并且没有任何延迟。

如果你想检查断路器的状态，可以使用以下命令：

```java
curl -ks https://health.minikube.me/actuator/health | jq -r .components.circuitBreakers.details.product.details.state 
```

它应该报告`CLOSED`、`OPEN`或`HALF_OPEN`，具体取决于其状态。

这证明了当使用 Istio 注入延迟时，断路器反应如预期。这标志着测试了可用于验证微服务景观弹性的 Istio 功能。我们将探索 Istio 的下一个功能是它对流量管理的支持；我们将看到它如何被用来实现零停机部署。

# 执行零停机更新

如*第十六章*中所述，*将我们的微服务部署到 Kubernetes*，随着越来越多的独立更新的自主微服务数量的增长，能够在不停机的情况下部署更新变得至关重要。

在本节中，我们将了解 Istio 的流量管理和路由能力，以及如何使用它们来部署微服务的新版本，而无需停机。在*第十五章*，*Kubernetes 简介*中，我们了解到 Kubernetes 可以用于进行滚动升级，而无需停机。使用 Kubernetes 的滚动升级机制可以自动化整个过程，但遗憾的是，它没有提供在所有用户都被路由到新版本之前测试新版本的选择。

使用 Istio，我们可以部署新版本，但最初将所有用户路由到现有版本（在本章中称为**旧**版本）。之后，我们可以使用 Istio 的细粒度路由机制来控制用户如何被路由到新版本和旧版本。我们将看到如何使用 Istio 实现两种流行的升级策略：

+   **金丝雀部署**：在金丝雀部署中，所有用户都被路由到旧版本，除了被选中的测试用户组，他们被路由到新版本。当测试用户批准新版本后，可以使用蓝绿部署将常规用户路由到新版本。

+   **蓝绿部署**：传统上，蓝绿部署意味着所有用户都会切换到蓝色或绿色版本，一个版本是新的，另一个版本是旧的。如果在切换到新版本时出现问题，切换回旧版本非常简单。使用 Istio，可以通过逐渐将用户转移到新版本来细化这种策略，例如，从 20%的用户开始，然后逐渐增加百分比。在任何时候，如果在新版本中发现了致命错误，都可以非常容易地将所有用户路由回旧版本。

如同在*第十六章*中已经提到的，我们需要记住，这些升级策略的一个先决条件是升级必须是**向后兼容**的。这种升级在 API 和消息格式方面都是兼容的，这些 API 和消息格式用于与其他服务和数据库结构进行通信。如果新版本的微服务需要对旧版本无法处理的外部 API、消息格式或数据库结构进行更改，那么这些升级策略就不能应用。

我们将讨论以下部署场景：

1.  我们将首先部署微服务的`v1`和`v2`版本，并将路由配置为将所有请求发送到微服务的`v1`版本。

1.  接下来，我们将允许一个测试组运行金丝雀测试；也就是说，我们将验证微服务的`v2`新版本。为了简化测试，我们只部署核心微服务的新版本，即`product`、`recommendation`和`review`微服务。

1.  最后，我们将开始使用蓝绿部署将常规用户迁移到新版本；最初是少量用户，然后随着时间的推移，越来越多的用户，直到最终所有用户都被路由到新版本。如果在新 `v2` 版本中检测到致命错误，我们还将看到如何快速切换回 `v1` 版本。

让我们先看看需要应用到源代码中的哪些更改，以便能够部署并将流量路由到核心微服务的两个并发版本，`v1` 和 `v2`。

## 源代码更改

为了能够同时运行微服务的多个版本，部署对象及其对应的 Pod 必须有不同的名称，例如，`product-v1` 和 `product-v2`。然而，每个微服务必须只有一个 Kubernetes 服务对象。所有流量都通过同一个服务对象到达特定的微服务，无论请求最终将被路由到哪个版本的 Pod。为了配置金丝雀测试和蓝绿部署的实际路由规则，使用 Istio 的 `VirtualService` 和 `DestinationRule` 对象。最后，`prod-env` Helm 图表中的 `values.yaml` 文件用于指定生产环境中将使用的每个微服务的版本。

让我们以下一节中的每个定义的详细信息进行说明：

+   虚拟服务和目标规则

+   部署和服务

+   在 `prod-env` Helm 图表中整合事物

### 虚拟服务和目标规则

为了在微服务的两个版本之间分割流量，我们需要在虚拟服务中指定两个版本之间的权重分布，在发送方。虚拟服务将在两个子集之间分配流量，称为 `old` 和 `new`。`new` 和 `old` 子集的确切含义在接收方的相应 `DestinationRule` 中定义。它使用 `labels` 来确定哪些 Pod 运行微服务的旧版本和新版本。

为了支持金丝雀测试，虚拟服务中需要一个始终将金丝雀测试者路由到 `new` 子集的路由规则。为了识别金丝雀测试者，我们将假设来自金丝雀测试者的请求包含一个名为 `X-group` 的 HTTP 头部，其值为 `test`。

已将一个模板添加到 `common` Helm 图表，用于创建一组虚拟服务，这些服务可以在微服务的两个版本之间分割流量。该模板名为 `_istio_vs_green_blue_deploy.yaml`，其内容如下：

```java
{{- define "common.istio_vs_green_blue_deploy" -}}
{{- range $name := .Values.virtualServices }}
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: {{ $name }}
spec:
  hosts:
  - {{ $name }}
  http:
  - match:
    - headers:
        X-group:
          exact: test
    route:
    - destination:
        host: {{ $name }}
        subset: new
  - route:
    - destination:
        host: {{ $name }}
        subset: old
      weight: 100
    - destination:
        host: {{ $name }}
        subset: new
      weight: 0
---
{{- end -}}
{{- end -}} 
```

从模板中，我们可以看到以下内容：

+   `range` 指令遍历 `virtualServices` 变量中定义的元素

+   清单 `spec` 部分的 `hosts` 字段用于指定此 `VirtualService` 将应用到的 Kubernetes 服务的名称

+   在 `http` 部分中，声明了三个路由目标：

    +   一个与金丝雀测试者的 HTTP 头部 `X-group` 匹配的路由，设置为 `test`。此路由始终将请求发送到 `new` 子集。

    +   `old`子集和一个`new`子集的一个路由目标。

    +   `weight`被指定为百分比，权重的总和始终为 100。

+   所有流量最初都路由到`old`子集

为了能够根据基于头部的路由将金丝雀测试者路由到新版本，`product-composite`微服务已被更新为转发 HTTP 头`X-group`。有关详细信息，请参阅`se.magnus.microservices.composite.product.services.ProductCompositeServiceImpl`类中的`getCompositeProduct()`方法。

对于目标规则，我们将重用上面*在 _istio_dr_mutual_tls.yaml 模板中的内容*部分中引入的模板。此模板将由`prod-env` Helm 图表用于指定要使用的微服务版本。这将在下面的*在 prod-env Helm 图表中整合各项*部分中描述。

### 部署和服务

为了使目标规则能够根据 Pod 的标签识别其版本，`common` Helm 图表中的部署模板`_deployment.yaml`已添加了一个`version`标签。其值设置为 Pod 的 Docker 镜像的`tag`。我们将使用`v1`和`v2`的 Docker 镜像标签，因此这也将是`version`标签的值。添加的行看起来像这样：

```java
 version: {{ .Values.image.tag }} 
```

为了给 Pod 及其 Deployment 对象命名，包含它们的版本信息，`prod-env`图表中已覆盖了它们的默认名称。在它们的`values.yaml`文件中，使用`fullnameOverride`字段指定包含版本信息的名称。这是为三个核心微服务所做的，看起来像这样：

```java
product:
  fullnameOverride: product-v1
recommendation:
  fullnameOverride: recommendation-v1
review:
  fullnameOverride: review-v1 
```

这的不希望产生的副作用是，相应的 Service 对象也将获得一个包含版本信息的名称。如上所述，我们需要有一个服务可以将请求路由到 Pod 的不同版本。为了避免这种命名问题，`common` Helm 图表中的服务模板`_service.yaml`已更新为使用`common.name`模板，而不是之前在*第十七章*中使用的`common.fullname`模板。

最后，为了能够部署三个核心微服务的多个版本，它们的 Helm 图表已在`kubernetes/helm/components`文件夹中进行了复制。新图表的名称后缀为`-green`。与现有图表相比，唯一的区别是它们不包括来自`common`图表的服务模板，从而避免了每个核心微服务创建两个 Service 对象。新图表的名称为`product-green`、`recommendation-green`和`review-green`。

### 在 prod-env Helm 图表中整合各项

`prod-env` Helm 图表包括来自`common` Helm 图表的`_istio_vs_green_blue_deploy.yaml`模板，以及`dev-env`图表中包含的模板；请参阅*创建服务网格*部分。

将三个新的`*-green` Helm 图表作为依赖项添加到`Chart.yaml`文件中。

在其`values.yaml`文件中，所有内容都紧密相连。从前一节中，我们看到了如何使用包含版本信息的名称定义核心微服务的`v1`版本。

对于`v2`版本，使用三个新的`*-green` Helm 图表。值与`v1`版本相同，除了名称和 Docker 镜像标签。例如，`product`微服务的`v2`版本配置如下：

```java
product-green:
  fullnameOverride: product-v2
  image:
    tag: v2 
```

为了声明三个核心微服务的虚拟服务，使用以下声明：

```java
virtualServices:
  - product
  - recommendation
  - review 
```

最后，以与`dev-env` Helm 图表类似的方式声明目标规则。主要区别在于我们现在使用子集来声明当虚拟服务将流量路由到`old`或`new`子集时应使用的实际版本。例如，`product`微服务的目标规则声明如下：

```java
destinationRules:
  - ...
  - name: product
    subsets:
    - labels:
        version: v1
      name: old
    - labels:
        version: v2
      name: new
... 
```

从上面的声明中，我们可以看到发送到`old`子集的流量被导向`product`微服务的`v1` Pods，而对于`new`子集，则导向`v2` Pods。

有关详细信息，请参阅`kubernetes/helm/environments/prod-env`文件夹中`prod-env`图表中的文件。

注意，这是我们声明生产环境中的现有（旧版）和即将到来（新版）版本的地方，在这个场景中是`v1`和`v2`。在未来的场景中，当升级`v2`到`v3`时，`old`子集应更新为使用`v2`，而`new`子集应使用`v3`。

现在，我们已经看到了源代码的所有更改，我们准备部署微服务的`v1`和`v2`版本。

## 部署将路由到`v1`版本的微服务的`v1`和`v2`版本

要能够测试微服务的`v1`和`v2`版本，我们需要移除本章前面使用的开发环境，并创建一个可以部署微服务的`v1`和`v2`版本的生产环境。

要实现这一点，运行以下命令：

1.  卸载开发环境：

    ```java
    helm uninstall hands-on-dev-env 
    ```

1.  要监控开发环境中 Pod 的终止，运行以下命令直到报告“在 hands-on 命名空间中未找到资源”：

    ```java
    kubectl get pods 
    ```

1.  在 Kubernetes 外部启动 MySQL、MongoDB 和 RabbitMQ：

    ```java
    eval $(minikube docker-env)
    docker-compose up -d mongodb mysql rabbitmq 
    ```

1.  使用`v1`和`v2`版本标记 Docker 镜像：

    ```java
    docker tag hands-on/auth-server hands-on/auth-server:v1
    docker tag hands-on/product-composite-service hands-on/product-composite-service:v1
    docker tag hands-on/product-service hands-on/product-service:v1
    docker tag hands-on/recommendation-service hands-on/recommendation-service:v1
    docker tag hands-on/review-service hands-on/review-service:v1
    docker tag hands-on/product-service hands-on/product-service:v2
    docker tag hands-on/recommendation-service hands-on/recommendation-service:v2
    docker tag hands-on/review-service hands-on/review-service:v2 
    ```

微服务的`v1`和`v2`版本将是这个测试中微服务的相同版本。但对于 Istio 来说，这并不重要，因此我们可以使用这种简化的方法来测试 Istio 的路由功能。

1.  使用 Helm 部署系统架构并等待所有部署完成：

    ```java
    helm install hands-on-prod-env \
      kubernetes/helm/environments/prod-env \
      -n hands-on --wait 
    ```

1.  一旦部署完成，使用以下命令验证我们为三个核心微服务创建了`v1`和`v2` Pods 并正在运行：

    ```java
    kubectl get pods 
    ```

    预期收到如下响应：

![计算机程序截图  描述由中等置信度自动生成](img/B19825_18_22.png)

图 18.22：同时部署的 v1 和 v2 Pods

1.  运行常规测试以验证一切是否正常工作：

    ```java
    ./test-em-all.bash 
    ```

    不幸的是，测试最初会失败，并显示如下错误信息：

    ```java
    - Response Body: Jwks doesn't have key to match kid or alg from Jwt 
    ```

此错误是由 Istio 守护程序 `istiod` 在开发环境中缓存来自身份验证服务器的 JWKS 公钥引起的。生产环境中的身份验证服务器将具有新的 JWKS 密钥，但与 `istiod` 具有相同的身份，因此它试图重用旧的 JWKS 公钥，导致此失败。Istio 默认情况下将 JWKS 公钥缓存 20 分钟，但在安装 Istio 时，我们将刷新间隔降低到 15 秒；请参阅*在 Kubernetes 集群中部署 Istio*部分。因此，等待一段时间后，最长可达一分钟，具体取决于刷新密钥传播的速度，你应该能够成功运行测试。一旦缓存的 JWKS 问题消失，测试可能会失败，出现如下错误：

```java
Test FAILED, EXPECTED VALUE: 3, ACTUAL VALUE: 0, WILL ABORT 
```

然后，只需重新运行命令，它应该运行良好！这些错误是由 JWKS 缓存引起的原始错误导致的次级故障。

预期输出与之前章节中看到的内容类似：

![计算机截图  描述自动生成，置信度中等](img/B19825_18_23.png)

图 18.23：测试运行成功

我们现在可以运行一些**零停机部署**测试了。让我们首先验证所有流量都路由到微服务的 `v1` 版本！

## 验证所有流量最初都路由到微服务的 v1 版本

为了验证所有请求都路由到微服务的 `v1` 版本，我们将启动负载测试工具 `siege`，然后使用 Kiali 观察通过服务网格的流量。

执行以下步骤：

1.  获取新的访问令牌并启动 `siege` 负载测试工具，使用以下命令：

    ```java
    ACCESS_TOKEN=$(curl https://writer:secret-writer@minikube.me/oauth2/token -d grant_type=client_credentials -d scope="product:read product:write" -ks | jq .access_token -r)
    echo ACCESS_TOKEN=$ACCESS_TOKEN
    siege https://minikube.me/product-composite/1 -H "Authorization: Bearer $ACCESS_TOKEN" -c1 -d1 -v 
    ```

1.  前往 Kiali 的 Web UI 中的**图**视图([`kiali.minikube.me`](https://kiali.minikube.me))：

    1.  点击**显示**菜单按钮并选择**命名空间框**。

    1.  点击**应用图**菜单按钮并选择**版本化应用图**。

    1.  预期只有流量路由到微服务的**v1**版本，如下所示：

![图表截图  描述自动生成，置信度低](img/B19825_18_24.png)

图 18.24：所有请求都发送到 v1 Pods

这意味着，尽管微服务的 `v2` 版本已部署，但它们没有收到任何流量路由到它们。现在让我们尝试金丝雀测试，允许选定的测试用户尝试微服务的 `v2` 版本！

## 运行金丝雀测试

为了运行金丝雀测试，使一些用户被路由到新版本，而所有其他用户仍然被路由到已部署微服务的旧版本，我们需要在我们的发送到外部 API 的请求中添加 `X-group` HTTP 标头，并将其设置为 `test` 值。

要查看哪个版本的微服务处理了请求，可以在响应中的`serviceAddresses`字段进行检查。`serviceAddresses`字段包含参与创建响应的每个服务的主机名。主机名等于 Pod 的名称，因此我们可以在主机名中找到版本；例如，对于版本为`v1`的`product`服务，是`product-v1-...`，对于版本为`v2`的`product`服务，是`product-v2-...`。

让我们从发送一个正常请求并验证它是否是响应我们请求的微服务的`v1`版本开始。接下来，我们将发送一个带有`X-group` HTTP 头设置为值`test`的请求，并验证新的`v2`版本是否在响应。

要做到这一点，请执行以下步骤：

1.  通过使用`jq`过滤响应中的`serviceAddresses`字段来执行正常请求以验证请求是否被路由到微服务的`v1`版本：

    ```java
    ACCESS_TOKEN=$(curl https://writer:secret-writer@minikube.me/oauth2/token  -d grant_type=client_credentials -d scope="product:read product:write" -ks | jq .access_token -r)
    echo ACCESS_TOKEN=$ACCESS_TOKEN
    curl -ks https://minikube.me/product-composite/1 -H "Authorization: Bearer $ACCESS_TOKEN" | jq .serviceAddresses 
    ```

    预期响应将与以下类似：

    ![计算机屏幕截图  自动生成描述](img/B19825_18_25.png)

    图 18.25：所有请求都进入 v1 Pods

    如预期，所有三个核心服务都是微服务的`v1`版本。

1.  如果我们添加`X-group=test`头，我们期望请求由核心微服务的`v2`版本提供服务。运行以下命令：

    ```java
    curl -ks https://minikube.me/product-composite/1 -H "Authorization: Bearer $ACCESS_TOKEN" -H "X-group: test" | jq .serviceAddresses 
    ```

    预期响应将与以下类似：

![计算机屏幕截图  自动生成描述](img/B19825_18_26.png)

图 18.26：将 HTTP 头设置为 X-group=test 使请求进入 v2 Pods

如预期，所有响应的三个核心微服务现在都是`v2`版本；作为金丝雀测试员，我们被路由到新的`v2`版本！

由于金丝雀测试返回了预期的结果，我们现在准备允许正常用户通过蓝绿部署路由到新的`v2`版本。

## 运行蓝绿部署

要将部分正常用户路由到微服务的新的`v2`版本，我们需要修改虚拟服务中的权重分布。它们目前是 100/0；换句话说，所有流量都被路由到旧的`v1`版本。我们可以通过编辑虚拟服务的清单文件并执行一个`kubectl apply`命令来实现这一点，以使更改生效。作为替代方案，我们可以在 Kubernetes API 服务器中的`virtual Service`对象上直接使用`kubectl patch`命令来更改权重分布。

当需要对同一对象进行多项更改以尝试新事物时，我发现`patch`命令很有用，例如，更改路由规则中的权重分配。在本节中，我们将使用`kubectl patch`命令快速更改微服务`v1`和`v2`版本之间路由规则中的权重分配。要获取执行几个`kubectl patch`命令后虚拟服务的状态，可以发出类似`kubectl get vs NNN -o yaml`的命令。例如，要获取`product`微服务虚拟服务的状态，可以发出以下命令：`kubectl get vs product -o yaml`。

由于我们之前没有使用过`kubectl patch`命令，并且一开始可能会有些复杂，所以在进行蓝绿部署之前，让我们简要介绍一下它是如何工作的。

### kubectl patch 命令简介

`kubectl patch`命令可用于更新 Kubernetes API 服务器中现有对象的特定字段。我们将尝试在名为`review`的虚拟服务上使用补丁命令，该虚拟服务是用于`review`微服务的。虚拟服务`review`定义的相关部分如下：

```java
spec:
  http:
  - match:
    ...
  - route:
    - destination:
        host: review
        subset: old
      weight: 100
    - destination:
        host: review
        subset: new
      weight: 0 
```

一个示例`patch`命令，该命令更改了`review`微服务中`v1`和`v2` Pods 的路由权重分配，如下所示：

```java
kubectl patch virtualservice review --type=json -p='[
  {"op": "add", "path": "/spec/http/1/route/0/weight", "value": 80},
  {"op": "add", "path": "/spec/http/1/route/1/weight", "value": 20}
]' 
```

命令将配置`review`微服务的路由规则，将 80%的请求路由到旧版本，20%的请求路由到新版本。

要指定在`review`虚拟服务中更改`weight`值，为旧版本提供`/spec/http/1/route/0/weight`路径，为新版本提供`/spec/http/1/route/1/weight`路径。

路径中的`0`和`1`用于指定虚拟服务定义中数组元素的索引。例如，`http/1`表示`http`元素下的数组中的第二个元素。参见前面的`review`虚拟服务定义。

从定义中我们可以看到，索引为`0`的第一个元素是`match`元素，我们不会更改它。第二个元素是`route`元素，我们想要更改它。

既然我们对`kubectl patch`命令有了更多了解，我们就准备好测试蓝绿部署了。

### 执行蓝绿部署

是时候逐渐使用蓝绿部署将越来越多的用户迁移到新版本了。要执行部署，请按照以下步骤操作：

1.  确保负载测试工具`siege`仍在运行。注意，它是在前面的*验证所有初始流量都流向微服务的 v1 版本*部分启动的。

1.  要允许 20%的用户被路由到新的`v2`版本的`review`微服务，我们可以使用以下命令修补虚拟服务并更改权重：

    ```java
    kubectl patch virtualservice review --type=json -p='[
      {"op": "add", "path": "/spec/http/1/route/0/weight", "value":  
      80},
      {"op": "add", "path": "/spec/http/1/route/1/weight", "value":  
      20}
    ]' 
    ```

1.  要观察路由规则的变化，请访问 Kiali Web UI（[`kiali.minikube.me`](https://kiali.minikube.me)）并选择**图形**视图。

1.  点击**显示**菜单，将边标签更改为**流量分布**。

1.  在 Kiali 更新指标之前等待一分钟，以便我们可以观察变化。预期 Kiali 中的图表将显示如下：

![计算机屏幕截图  描述自动生成，置信度中等](img/B19825_18_27.png)

图 18.27：80%流向 v1 服务，20%流向 v2 服务

根据您等待的时间长短，图表可能看起来略有不同！在屏幕截图中，我们可以看到 Istio 现在将流量路由到`review`微服务的`v1`和`v2`版本。

从`product-composite`微服务发送到`review`微服务的流量中，6.4%被路由到新的`v2` Pod，22.3%被路由到旧的`v1` Pod。这意味着 6.4/(6.4 + 22.3) = 22%的请求被路由到`v2` Pod，78%被路由到`v1` Pod。这与我们请求的 20/80 分布相一致。

请随意尝试前面的`kubectl patch`命令，以影响其他核心微服务（`product`和`recommendation`）的路由规则。

为了简化对所有三个核心微服务的权重分布的更改，可以使用`./kubernetes/routing-tests/split-traffic-between-old-and-new-services.bash`脚本。例如，要将所有流量路由到所有微服务的`v2`版本，运行以下脚本，并传入权重分布`0 100`：

```java
./kubernetes/routing-tests/split-traffic-between-old-and-new-services.bash 0 100 
```

在 Kiali 能够可视化路由变化之前，您需要给它一分钟左右的时间来收集指标，但请记住，实际路由的变化是立即发生的！

预期一段时间后，图表中将只显示将请求路由到微服务`v2`版本的请求：

![计算机屏幕截图  描述自动生成，置信度中等](img/B19825_18_28.png)

图 18.28：所有流量都流向 v2 服务

根据您等待的时间长短，图表可能看起来略有不同！

如果升级到`v2`版本后出现严重错误，可以使用以下命令将所有微服务的流量回滚到`v1`版本：

```java
./kubernetes/routing-tests/split-traffic-between-old-and-new-services.bash 100 0 
```

稍后，Kiali 中的图表应该看起来像之前*验证所有流量最初都流向微服务的 v1 版本*部分中的屏幕截图，再次显示所有请求都流向所有微服务的`v1`版本。

这就完成了对服务网格概念及其实现者 Istio 的介绍。

在结束本章之前，让我们回顾一下如何使用 Docker Compose 运行测试，以确保我们的微服务源代码不依赖于 Kubernetes 的部署或 Istio 的存在。

# 使用 Docker Compose 运行测试

正如之前多次提到的，确保微服务的源代码从功能角度来看不依赖于像 Kubernetes 或 Istio 这样的平台是很重要的。

为了验证微服务在没有 Kubernetes 和 Istio 的情况下按预期工作，请运行*第十七章*中描述的测试（参考*使用 Docker Compose 进行测试*部分）。由于测试脚本`test-em-all.bash`的默认值已经改变，如*运行创建服务网格的命令*部分之前所述，使用 Docker Compose 时必须设置以下参数：`USE_K8S=false HOST=localhost PORT=8443 HEALTH_URL=https://localhost:8443`。例如，要使用默认的 Docker Compose 文件`docker-compose.yml`运行测试，请执行以下命令：

```java
USE_K8S=false HOST=localhost PORT=8443 HEALTH_URL=https://localhost:8443 ./test-em-all.bash start stop 
```

测试脚本应该像以前一样，首先启动所有容器；然后运行测试，最后停止所有容器。有关预期输出的详细信息，请参阅*第十七章*（参考*验证微服务在没有 Kubernetes 的情况下工作*部分）。

在使用 Docker Compose 成功执行测试后，我们已经验证了从功能角度来看，微服务既不依赖于 Kubernetes 也不依赖于 Istio。这些测试完成了关于使用 Istio 作为服务网格的章节。

# 摘要

在本章中，我们学习了服务网格的概念和 Istio，这是该概念的开放源代码实现。服务网格为处理微服务系统景观中的挑战提供了能力，例如安全性、策略执行、弹性和流量管理。服务网格还可以用来使微服务系统景观可观察，通过可视化通过微服务的流量。

对于可观察性，Istio 可以与 Kiali、Jaeger 和 Grafana（更多关于 Grafana 和 Prometheus 的内容请参阅*第二十章*，*监控微服务*）集成。当涉及到安全性时，Istio 可以被配置为使用证书来保护外部 API 使用 HTTPS，并要求外部请求包含有效的基于 JWT 的 OAuth 2.0/OIDC 访问令牌。最后，Istio 可以被配置为使用相互认证（mTLS）自动保护内部通信。

为了弹性和健壮性，Istio 提供了处理重试、超时和类似于熔断器的异常检测机制的机制。在许多情况下，如果可能的话，最好在微服务的源代码中实现这些弹性能力。在 Istio 中注入故障和延迟的能力对于验证服务网格中的微服务作为一个弹性和健壮的系统景观协同工作非常有用。Istio 还可以用来处理零停机时间部署。使用其细粒度的路由规则，既可以进行金丝雀部署，也可以进行蓝绿部署。

我们还没有涉及的一个重要领域是如何收集和分析所有微服务实例创建的日志文件。在下一章中，我们将看到如何使用一个流行的工具栈，称为 EFK 工具栈，基于 Elasticsearch、Fluentd 和 Kibana 来实现这一点。

# 问题

1.  服务网格中代理组件的目的是什么？

1.  服务网格中的控制平面和数据平面有什么区别？

1.  `istioctl kube-inject` 命令用于什么？

1.  `minikube tunnel` 命令用于什么？

1.  Istio 集成了哪些工具来实现可观察性？

1.  要使 Istio 使用相互认证保护服务网格内的通信，需要哪些配置？

1.  虚拟服务中的 `abort` 和 `delay` 元素可以用作什么？

1.  设置蓝绿部署场景需要哪些配置？
