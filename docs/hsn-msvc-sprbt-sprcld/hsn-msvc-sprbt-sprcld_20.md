# 第十七章：作为 Spring Cloud Gateway 的替代品实现 Kubernetes 特性

当前微服务架构包含许多实现大规模微服务架构中所需的重要设计模式的支撑服务；例如边缘、配置和授权服务器，以及分布式跟踪服务。详情请见第一章，*微服务简介*，并参考*微服务设计模式*小节。在前一章中，我们用 Kubernetes 内置的发现服务替换了基于 Netflix Eureka 的服务发现设计模式。在本章中，我们将通过减少需要部署的支撑服务数量来进一步简化微服务架构。相反，相应的设计模式将由 Kubernetes 内置功能处理。Spring Cloud Config Server 将被 Kubernetes 配置映射和机密替换。Spring Cloud Gateway 将被 Kubernetes 入口资源替换，后者可以像 Spring Cloud Gateway 一样充当边缘服务器。

在第十一章中，关于*保护 API 访问安全*，参考*使用 HTTPS 保护外部通信*小节，我们使用证书来保护外部 API。手动处理证书既耗时又容易出错。作为替代方案，我们将介绍 Cert Manager，它可以自动为入口暴露的外部 HTTPS 端点提供新证书并替换已过期的证书。我们将配置`cert-manager`使用**Let's Encrypt**发行证书。Let's Encrypt 是一个可以用来自动颁发证书的免费证书授权机构。Let's Encrypt 必须能够验证我们拥有将发行证书的 DNS 名称。由于我们的 Kubernetes 集群在 Minikube 中本地运行，我们必须让 Let's Encrypt 在配置过程中能够访问我们的集群。我们将使用`ngrok`创建一个从互联网到我们本地 Kubernetes 集群的临时 HTTP 隧道，供 Let's Encrypt 使用。

当使用像 Kubernetes 这样的平台越来越多的特性时，确保微服务的源代码不依赖于该平台是很重要的；也就是说，应该确保微服务在没有 Kubernetes 的情况下仍然可以使用。为了确保我们可以在没有 Kubernetes 的情况下使用微服务，本章将通过使用 Docker Compose 部署微服务架构并以执行`test-em-all.bash`测试脚本结束，验证微服务在没有 Kubernetes 的情况下仍然可以工作。

本章将涵盖以下主题：

+   用 Kubernetes 配置映射和机密替换 Spring Cloud Config Server

+   用 Kubernetes 入口资源替换 Spring Cloud Gateway

+   添加 Cert Manager 以自动提供由 Let's Encrypt 签发的证书

+   使用`ngrok`从互联网建立到我们本地 Kubernetes 集群的 HTTP 隧道。

+   使用 Docker Compose 部署和测试微服务架构，以确保微服务中的源代码不会锁定在 Kubernetes 上。

# 技术要求

本书中描述的所有命令都是在 MacBook Pro 上使用 macOS Mojave 运行的，但修改后可以在其他平台，如 Linux 或 Windows 上运行，应该是很简单的。

本章所需的新工具是命令行`ngrok`工具，用于建立从互联网到我们本地环境的 HTTP 隧道。它可以通过 Homebrew 使用以下命令安装：

```java
brew cask install ngrok
```

要使用`ngrok`，必须创建一个免费账户并注册一个授权令牌，具体步骤如下：

1.  在这里注册：[`dashboard.ngrok.com/signup`](https://dashboard.ngrok.com/signup)。

1.  账户创建后，运行以下命令：

```java
ngrok authtoken <YOUR_AUTH_TOKEN>
```

在这里，`<YOUR_AUTH_TOKEN>`用以下页面找到的授权令牌替换——[`dashboard.ngrok.com/auth`](https://dashboard.ngrok.com/auth)。

本章的源代码可以在 GitHub 上找到：[`github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud/tree/master/Chapter17`](https://github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud/tree/master/Chapter17)。

为了能够按照书中描述运行命令，你需要将源代码下载到一个文件夹中，并设置一个环境变量，`$BOOK_HOME`，指向那个文件夹。示例命令如下：

```java
export BOOK_HOME=~/Documents/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud
git clone https://github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud $BOOK_HOME
cd $BOOK_HOME/Chapter17
```

Java 源代码是为 Java 8 编写的，并在 Java 12 上进行了测试。本章使用了 Spring Cloud 2.1, SR2（也称为**Greenwich**版本），Spring Boot 2.1.6 和 Spring 5.1.8——这些是编写本章时可用的 Spring 组件的最新版本。源代码已经使用 Kubernetes v1.15 进行了测试。

本章中所有源代码示例均来自`$BOOK_HOME/Chapter17`的源代码，但在某些情况下进行了编辑，去除了源代码中不相关的内容，例如注释、导入和日志语句。

如果你想查看对第十七章中源代码所做的更改，即替换 Spring Cloud Config Server 和 Spring Cloud Gateway 以便使用 Kubernetes 中对应特性的更改，你可以将其与第十六章的源代码进行比较，《将我们的微服务部署到 Kubernetes》。你可以使用你喜欢的`diff`工具，比较`$BOOK_HOME/Chapter16`和`$BOOK_HOME/Chapter17`文件夹。

# 替换 Spring Cloud Config Server。

正如我们在上一章看到的，第十六章，*将我们的微服务部署到 Kubernetes*，在*部署到 Kubernetes*部分，配置映射和秘钥可用于持有我们微服务的配置信息。Spring Cloud Config Server 增加了诸如将所有配置放在一个地方、可选的 Git 版本控制以及加密磁盘上的敏感信息等价值。但它也消耗了相当大的内存（就像任何基于 Java 和 Spring 的应用程序一样）并在启动时增加了显著的开销。例如，当运行本书中使用的自动化集成测试，如测试脚本`test-em-all.bash`时，所有微服务都会同时启动，包括配置服务器。由于其他微服务必须在启动之前从配置服务器获取其配置，它们都必须等待配置服务器启动并运行后才能启动。这导致集成测试运行时出现显著延迟。如果我们使用 Kubernetes 的配置映射和秘钥，则消除了这个延迟，使自动化集成测试运行得更快。在我看来，在底层平台不提供类似功能时使用 Spring Cloud Config Server 是有意义的，但在部署到 Kubernetes 时，最好使用配置映射和秘钥。

使用 Kubernetes 的配置映射和秘钥，而不是 Spring Cloud Config Server，可以加快微服务架构的启动速度，并减少所需内存。它还将通过消除一个支持服务，即配置服务器，简化微服务架构。如下图所示：

![](img/7ab391b2-00bd-4930-8d8f-371cd083215a.png)

让我们看看替换 Spring Cloud Config Server 所需的步骤！

特别注意，我们只更改了配置；也就是说，不需要更改 Java 源代码！

# 源代码中的更改，以替换 Spring Cloud Config Server

已应用于源代码配置的以下更改，以替换 Spring Cloud Config Server 与 Kubernetes 配置映射和秘钥：

+   删除了项目`spring-cloud/config-server`，还包括：

    +   删除了`settings.gradle`构建文件中的项目。

    +   删除了`config-server`的 YAML 文件及其在`kubernetes/services/base`和`kubernetes/services/overlays/prod`文件夹中的声明。

+   删除了所有微服务中的配置：

    +   删除了`build.gradle`构建文件中的`spring-cloud-starter-config`依赖项。

    +   删除了每个项目`src/main/resource`文件夹中的`bootstrap.yml`文件。

    +   删除了集成测试中的`spring.clod.config.enabled=false`属性设置。

+   `config-repo`文件夹中的配置文件更改：

    +   移除了包含敏感信息的属性，也就是说，MongoDB、MySQL、RabbitMQ 的凭据以及边缘服务器使用的 TLS 证书的密码。它们将被 Kubernetes 密钥替换。

    +   在边缘服务器的配置中移除了对配置服务器 API 的路径

+   `kubernetes/services/base`文件夹中部署资源定义文件的变化：

    +   配置映射作为卷挂载，也就是说，作为容器文件系统中的文件夹。每个微服务都有自己的配置映射，包含适用于特定微服务的配置文件。

    +   定义`SPRING_CONFIG_LOCATION`环境变量以指出卷中的配置文件。

    +   使用密钥定义访问资源管理器的凭据。

大部分变化都在 Kubernetes 部署资源的定义文件中。让我们以`product`微服务的部署资源定义为例来看一下：

```java
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product
spec:
  template:
    spec:
      containers:
      - name: pro
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "docker"
        - name: SPRING_CONFIG_LOCATION
          value: file:/config-repo/application.yml,file:/config-
           repo/product.yml
        envFrom:
        - secretRef:
            name: rabbitmq-credentials
        - secretRef:
            name: mongodb-credentials
        volumeMounts:
        - name: config-repo-volume
          mountPath: /config-repo
      volumes:
      - name: config-repo-volume
        configMap:
          name: config-repo-product
```

请注意，由于变化未影响到的定义部分被省略以提高可读性。完整的源代码请查看`kubernetes/services/base/product.yml`。

以下解释了前面的源代码：

+   `config-repo-product`配置映射映射在一个名为`config-repo-volume`的卷中。

+   `config-repo-volume`卷挂载在文件系统的`/config-repo`目录下。

+   `SPRING_CONFIG_LOCATION`环境变量告诉 Spring 在哪里可以找到属性文件，在这个例子中，是`/config-repo/application.yml`和`/config-repo/product.yml`文件。

+   访问 RabbitMQ 和 MongoDB 的凭据是基于`rabbitmq-credentials`和`mongodb-credentials`密钥的内容设置为环境变量的。

在*使用 ConfigMaps、密钥和 ingress 进行测试*节中创建 Kubernetes 配置映射和密钥。

替换配置服务器需要的就是这些，接下来的一节中，我们将学习如何用 Kubernetes 的 ingress 资源替换 Spring Cloud Gateway。

# 替换 Spring Cloud Gateway

在本节中，我们将通过用 Kubernetes 内置的 ingress 资源替换 Spring Cloud Gateway，进一步简化微服务架构，减少需要部署的支持服务数量。

如第十五章、*Kubernetes 简介*中介绍的，Kubernetes 中的 ingress 资源可以像 Spring Cloud Gateway 一样作为边缘服务器使用。Spring Cloud Gateway 带有比 ingress 资源更丰富的路由功能。但是 ingress 特性是 Kubernetes 平台的一部分，也可以使用 Cert Manager 自动提供证书，如我们在本章后面所看到的。

我们还使用 Spring Cloud Gateway 保护我们的微服务免受未经验证的请求；也就是说，微服务需要一个来自可信 OAuth 授权服务器或 OIDC 提供者的有效 OAuth 2.0/OIDC 访问令牌。如果需要回顾，请参见第十一章，*保护 API 访问*。通常，Kubernetes 入口资源不支持这一点。然而，入口控制器的特定实现可能会支持它。

最后，我们在第十章，*使用 Spring Cloud Gateway 将微服务隐藏在边缘服务器后面*中向网关添加的综合健康检查可以被每个微服务部署资源中定义的 Kubernetes 存活和就绪探针所替换。对我来说，在底层平台不提供类似功能时使用 Spring Cloud Gateway 是有意义的，但在部署到 Kubernetes 时，最好使用入口资源。

在本章中，我们将验证请求是否包含有效访问令牌的责任委托给`product-composite`微服务。下一章将介绍服务网格的概念，我们将看到一个完全支持验证 JWT 编码访问令牌的入口的替代实现，即我们在本书中使用的访问令牌类型。

在*验证微服务在没有 Kubernetes 的情况下是否工作*部分，我们仍然将使用 Spring Cloud Gateway 和 Docker Compose，所以我们将不会移除项目。

以下图表展示了在将 Spring Cloud Gateway 部署到 Kubernetes 时，如何将其从微服务架构中移除：

![](img/d19dde2e-6974-428d-a6e3-ba564833e00a.png)

让我们看看替换 Spring Cloud Gateway 为 Kubernetes 入口资源需要什么！

特别注意，我们只更改了配置；也就是说，不需要更改 Java 源代码！

# Spring Cloud Gateway 源代码的变化

以下更改已应用于源代码配置，以将 Spring Cloud Gateway 替换为 Kubernetes 入口资源：

+   `kubernetes/services`文件夹中部署资源的变化。

    +   删除了网关的 YAML 文件及其在`base`和`overlays/prod`文件夹中的声明。

    +   在`base/ingress-edge-server.yml`中添加了入口资源，并在`base/kustomization.yml`中对其进行了引用

入口资源的定义如下代码所示：

```java
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: edge
spec:
  tls:
    - hosts:
      - minikube.me
      secretName: tls-certificate
  rules:
  - host: minikube.me
    http:
      paths:
      - path: /oauth
        backend:
          serviceName: auth-server
          servicePort: 80
      - path: /product-composite
        backend:
          serviceName: product-composite
          servicePort: 80 
      - path: /actuator/health
        backend:
          serviceName: product-composite
          servicePort: 80
```

以下是对前面源代码的解释：

+   入口资源的名称为`edge`。

+   `tls`部分指定了入口将需要使用 HTTPS，并且它将使用为`minikube.me`主机名发行的证书。

+   证书存储在一个名为`tls-certificate`的秘密中

    在*使用 Kubernetes ConfigMaps、secrets 和 ingress 资源进行测试*部分的*步骤 4*中创建`tls-certificate`密钥。

+   为请求到`minikube.me`主机名定义路由规则。

    在下一主题中，DNS 名称`minikube.me`将被映射到 Minikube 实例的 IP 地址。

+   为以下内容定义路由：

    +   在`/oauth`路径上的`auth-server`。

    +   在`/product-composite`路径上的`product-composite`微服务。

    +   `product-composite`微服务中的`/actuator/health`路径上的`health`端点。

在下一节中，我们将创建 Kubernetes 的 ingress 资源，并一起测试微服务架构，包括 Kubernetes 的 config maps、secrets 和 ingress 资源。

# 使用 Kubernetes ConfigMaps、secrets 和 ingress 资源进行测试。

在前述更改描述之后，我们准备测试使用 Spring Cloud Config Server 和 Spring Cloud Gateway 替换 Kubernetes config maps、secrets 和 ingress 资源的系统架构。与之前一样，当我们使用 Spring Cloud Gateway 作为边缘服务器时，外部 API 将通过 HTTPS 保护。在此部署中，我们将配置 ingress 资源以重用我们与 Spring Cloud Gateway 一起用于 HTTPS 的自签名证书。以下图表展示了这一点：

![](img/5eb38462-0a35-4944-a8e6-0e6ff5e338b1.png)

在下一节中，我们将增强证书使用，并将自签名证书替换为 Let's Encrypt 发行的证书。

ingress 将在 Minikube 实例的默认 HTTPS 端口`443`上暴露。这由我们在执行`minikube addons enable ingress`命令时安装的 ingress 控制器处理；参见第十五章*Kubernetes 简介*，并参考*创建 Kubernetes 集群*部分进行回顾。ingress 控制器由在`kube-system`命名空间中的部署`nginx-ingress-controller`组成。部署使用`hostPort`将其端口`443`映射到宿主机上，即 Minikube 实例中的端口`443`，该端口在容器中运行。部署的定义中的主要部分如下所示：

```java
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-ingress-controller
spec:
  template:
    spec:
      containers:
        image: quay.io/kubernetes-ingress-controller/nginx-ingress-
         controller:0.23.0
        ports:
        - containerPort: 443
          hostPort: 443
```

此设置适用于用于开发和测试的单节点 Kubernetes 集群。在多节点 Kubernetes 集群中，外部负载均衡器用于暴露高可用性和可伸缩性的 ingress 控制器。

部署使用了与我们在第十六章*部署我们的微服务到 Kubernetes*中使用的相同类型的命令；参考*部署到 Kubernetes 以进行开发和测试*部分。

主要区别是，此部署将：

+   为每个微服务创建一个 config map，而不是为配置服务器创建一个 config map。

+   创建用于资源管理器凭据的秘密和用于 ingress 的 TLS 证书的秘密，而不是创建用于配置服务器凭据的秘密

+   创建一个 Ingress，而不是使用 Spring Cloud Gateway

为了简化部署，已将开发和生产环境的部署脚本添加到源代码中。让我们来看看我们将在此部分使用的开发环境的部署脚本。

# 浏览部署脚本

`kubernetes/scripts/deploy-dev-env.bash`脚本包含了执行部署所需的所有命令。该脚本将执行以下步骤：

1.  它将创建一个配置文件，每个微服务一个。例如，对于`product`微服务，我们有以下内容：

```java
kubectl create configmap config-repo-product --from-file=config-repo/application.yml --from-file=config-repo/product.yml --save-config
```

1.  然后，它将创建所需的秘密。例如，使用以下命令创建访问 RabbitMQ 的凭据：

```java
kubectl create secret generic rabbitmq-credentials \
 --from-literal=SPRING_RABBITMQ_USERNAME=rabbit-user-dev \
 --from-literal=SPRING_RABBITMQ_PASSWORD=rabbit-pwd-dev \
 --save-config
```

1.  也为资源管理器创建了秘密；它们的名称以`server-credentials`结尾。它们在`kubernetes/services/overlays/dev`文件夹的 Kubernetes 定义文件中使用。例如，使用以下命令创建 RabbitMQ 的凭据：

```java
kubectl create secret generic rabbitmq-server-credentials \
 --from-literal=RABBITMQ_DEFAULT_USER=rabbit-user-dev \
 --from-literal=RABBITMQ_DEFAULT_PASS=rabbit-pwd-dev \
 --save-config
```

1.  包含 ingress 的 TLS 证书的秘密`tls-certificate`基于`kubernetes/cert`文件夹中已经存在的自签名证书。它使用以下命令创建：

```java
kubectl create secret tls tls-certificate --key kubernetes/cert/tls.key --cert kubernetes/cert/tls.crt
```

以下命令创建了自签名证书：

`openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout kubernetes/cert/tls.key -out kubernetes/cert/tls.crt -subj "/CN=minikube.me/O=minikube.me"`

1.  基于`dev`覆盖层部署开发环境的微服务，使用`-k`开关激活 Kustomize：

```java
kubectl apply -k kubernetes/services/overlays/dev
```

1.  等待部署及其 pods 运行：

```java
kubectl wait --timeout=600s --for=condition=ready pod --all
```

完成本指南后，我们就可以运行部署和测试所需的命令了！

# 部署和测试命令的运行

在我们能够执行部署之前，需要做以下准备工作：

+   将 ingress 使用的 DNS 名称`minikube.me`映射到 Minikube 实例的 IP 地址

+   从源代码构建 Docker 镜像

+   在 Kubernetes 中创建一个命名空间

运行以下命令以准备、部署和测试：

1.  将`minikube.me`映射到 Minikube 实例的 IP 地址，通过在`/etc/hosts`文件中添加以下命令的行来实现：

```java
sudo bash -c "echo $(minikube ip) minikube.me | tee -a /etc/hosts" 
```

使用`cat /etc/hosts`命令验证结果。预期会出现包含 Minikube 实例 IP 地址后跟`minikube.me`的行，如下所示：

![](img/e620ccae-5f94-4d2f-a0cc-9c5b94ffb268.png)

1.  使用以下命令从源代码构建 Docker 镜像：

```java
cd $BOOK_HOME/Chapter17
eval $(minikube docker-env)
./gradlew build && docker-compose build
```

1.  重新创建命名空间`hands-on`，并将其设置为默认命名空间：

```java
kubectl delete namespace hands-on
kubectl create namespace hands-on
kubectl config set-context $(kubectl config current-context) --namespace=hands-on 
```

1.  运行以下命令执行部署脚本：

```java
./kubernetes/scripts/deploy-dev-env.bash 
```

1.  部署完成后，使用以下命令启动测试：

```java
HOST=minikube.me PORT=443 ./test-em-all.bash
```

期待我们看到前几章中的正常输出，如下面的屏幕截图所示：

![](img/2029adbf-aead-45bf-b8a5-78f258baa0f6.png)

1.  您可以尝试手动执行早期章节中的相同步骤来测试 API：只需将主机和端口替换为 `minikube.me`。获取 OAuth/OIDC 访问令牌，并使用它调用具有以下命令的 API：

```java
ACCESS_TOKEN=$(curl -k https://writer:secret@minikube.me/oauth/token -d grant_type=password -d username=magnus -d password=password -s | jq .access_token -r)

curl -ks https://minikube.me/product-composite/2 -H "Authorization: Bearer $ACCESS_TOKEN" | jq .productId
```

期待在响应中收到请求的产品 ID，`2`。

本节中设置的部署是基于一个用于开发和测试的环境。如果您想要建立一个用于 staging 和生产的环境，例如在第十六章《将我们的微服务部署到 Kubernetes》中描述的那样，请参考“为 staging 和生产部署到 Kubernetes”部分。为此，您可以使用 `./kubernetes/scripts/deploy-prod-env.bash` 脚本。在第 4 步中像以前概述的那样使用它，而不是 `deploy-dev-env.bash` 脚本。

注意，`deploy-prod-env.bash` 脚本将使用 Docker Compose 启动 MySQL、MongoDB 和 RabbitMQ 的三个资源管理器；也就是说，它们将作为 Docker 容器在 Kubernetes 外运行（正如在第十六章《将我们的微服务部署到 Kubernetes》中所描述的那样）。

此部署使用了一个由 Kubernetes ingress 暴露的自签名证书，该证书需要手动提供。手动处理证书既耗时又容易出错。例如，很容易忘记及时续签一个证书。在下一节中，我们将学习如何使用 Cert Manager 和 Let's Encrypt 来自动化这个提供过程！

# 自动化证书的提供

正如本章介绍中提到的，我们将使用 Cert Manager 来自动化由 ingress 暴露的外部 HTTPS 端点所使用的证书的提供。Cert Manager 将在 Kubernetes 中作为一个附加组件运行，并配置为从 Let's Encrypt 请求免费证书颁发机构的证书，以自动化证书的颁发。为了能够验证我们拥有证书将要颁发的 DNS 名称，Let's Encrypt 要求访问我们要为其颁发证书的端点。由于我们的 Kubernetes 集群在本地运行 Minikube，我们必须让 Let's Encrypt 在证书提供期间能够访问我们的集群。我们将使用 ngrok 工具创建一个从互联网到我们的本地 Kubernetes 集群的临时 HTTP 隧道，供 Let's Encrypt 使用。

有关每个产品的更多信息，请参阅以下内容：

+   Cert Manager: [`docs.cert-manager.io/en/latest/index.html`](http://docs.cert-manager.io/en/latest/index.html)

+   Let's Encrypt: [`letsencrypt.org/docs/`](https://letsencrypt.org/docs/)

+   ngrok: [`ngrok.com/docs`](https://ngrok.com/docs)

所有这一切可能看起来有些令人望而却步，所以让我们一步一步来：

1.  部署证书管理器并在 Kubernetes 中基于 Let's Encrypt 定义发行者。

1.  使用 ngrok 创建 HTTP 隧道。

1.  使用证书管理器（Cert Manager）和让我们加密（Let's Encrypt）提供证书。

1.  验证我们是否从 Let's Encrypt 获取了证书。

1.  清理。

只有当你的 Kubernetes 集群无法通过互联网访问时，才需要 HTTP 隧道。如果其入口资源可以直接从互联网访问，则可以跳过使用 ngrok。

# 部署证书管理器并定义 Let's Encrypt 发行者

要部署证书管理器（Cert Manager），我们可以执行一个 Kubernetes 定义文件，这将创建一个名为 `cert-manager` 的命名空间，然后将证书管理器部署到该命名空间中。我们将安装编写本章时的最新版本 0.8.1。运行以下命令：

```java
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v0.8.1/cert-manager.yaml
```

如果你收到类似“无法识别 "https://github.com/jetstack/cert-manager/releases/download/v0.8.1/cert-manager.yaml"：在版本 "certmanager.k8s.io/v1alpha1" 中没有找到 "Issuer" 种类”的错误消息，那么只需重新运行命令即可。

等待部署及其 Pod 可用：

```java
kubectl wait --timeout=600s --for=condition=ready pod --all -n cert-manager
```

预期从命令中输出与以下类似的结果：

![](img/196e1fc3-83ec-4d71-84dc-18e5f84a59ed.png)

有了证书管理器（Cert Manager）之后，我们可以在 Kubernetes 中定义基于 Let's Encrypt 的发行者。

让我们加密（Let's Encrypt）暴露了以下发行者：

+   **开发和测试阶段使用的暂存环境**，可以预期在这一阶段会请求很多短期证书。暂存环境允许创建许多证书，但证书中的根**CA**（证书授权中心）不被信任。这意味着来自暂存环境的证书不能用于保护由网页浏览器使用的网页或 API。当用户打开由暂存环境中的证书保护的网页时，网页浏览器不会信任其根 CA 并抱怨。

+   **生产环境**，它使用一个受信任的根 CA 来颁发证书。因此，它可以用来颁发被网页浏览器信任的证书。生产环境限制了可以颁发的证书数量。例如，每个注册域名每周只能发行 50 个新证书，例如在 `ngrok.io` 的情况下。

我们将在 Kubernetes 中注册两个发行者，一个用于暂存环境，一个用于生产环境。发行者可以在集群全局或命名空间局部注册。为了保持一致性，我们将使用命名空间局部发行者。

在证书提供过程中，证书管理器（Cert Manager）和让我们加密（Let's Encrypt）之间的通信基于一个标准协议，即**自动化证书管理环境 v2**（**ACME v2**）。让我们加密将充当证书授权中心（CA），而证书管理器将充当 ACME 客户端。为了验证 DNS 名称的所有权，ACME 协议规定了 CA 可以使用两种类型的挑战：

+   `http-01`: 认证机构（CA）要求 ACME 客户端在以下 URL 下创建一个随机命名的文件：`http://<domainname>/.well-known/acme-challenge/<randomfilename>`。如果 CA 能够使用这个 URL 成功访问该文件，那么域名所有权得到验证。

+   `dns-01`: 认证机构（CA）要求 ACME 客户端在 DNS 服务器下的域名`_acme-challenge.<YOUR_DOMAIN>`放置一个指定的值。这通常通过 DNS 提供商的 API 来实现。如果 CA 能够在 DNS 服务器中的 TXT 记录中访问到指定的内容，那么域名所有权得到验证。

自动化`dns-01`挑战通常比自动化`http-01`挑战要困难，但在 HTTP 端点不可通过互联网的情况下，它是更优的选择。`dns-01`挑战还支持发行通配符证书，这是`http-01`挑战无法使用的。在本章中，我们将配置 Cert Manager 以使用基于`http-01`的挑战。

对于 Let's Encrypt 测试环境的发行者定义如下：

```java
apiVersion: certmanager.k8s.io/v1alpha1
kind: Issuer
metadata:
  name: letsencrypt-issuer-staging
spec:
  acme:
    email: <your email address>
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-issuer-staging-account-key
    solvers:
    - http01:
        ingress:
          class: nginx
```

以下解释了前面的源代码：

+   发行者的`name`，`letsencrypt-issuer-staging`，将在负载均衡器中引用发行者时使用，以提供证书。

+   `email`必须填写您的电子邮件地址。Let's Encrypt 将使用该电子邮件地址联系您关于到期的证书和与您账户相关的问题。

+   `server`字段指出了 Let's Encrypt 测试环境的 URL。

+   `privateKeySecretRef`字段包含一个密钥的名字。这个密钥将由 Cert Manager 创建，并将包含一个 ACME/Let's Encrypt `account private key`。这个密钥用来识别您（或您的公司）作为 ACME 服务的用户，即 Let's Encrypt。它用于签署发送到 Let's Encrypt 的请求，以验证您的身份。

+   `solver`定义声明使用`http-01`挑战来验证域名所有权。

对于 Let's Encrypt 生产环境的发行者定义看起来是一样的，主要区别在于使用的 ACME 服务器 URL：[`acme-v02.api.letsencrypt.org/directory`](https://acme-v02.api.letsencrypt.org/directory)。

编辑以下文件，并将`<your email address>`替换为您的电子邮件地址：

+   `kubernetes/services/base/letsencrypt-issuer-staging.yaml`

+   `kubernetes/services/base/letsencrypt-issuer-prod.yaml`

使用以下命令应用定义：

```java
kubectl apply -f kubernetes/services/base/letsencrypt-issuer-staging.yaml
kubectl apply -f kubernetes/services/base/letsencrypt-issuer-prod.yaml
```

现在我们已经有了 Cert Manager，并注册了 Let's Encrypt 测试环境和生产环境的发行者。下一步是使用`ngrok`创建一个 HTTP 隧道。

# 使用 ngrok 创建 HTTP 隧道

`ngrok`的免费订阅可以用来创建一个 HTTP 隧道，其中`ngrok`使用其自己的通配符证书`***.ngrok.io`来终止 HTTPS 流量，也就是说，在 HTTP 请求到达 Kubernetes 中的 ingress 资源之前。发送 HTTPS 请求的客户端只能看到`ngrok`证书，而看不到 Kubernetes 中 ingress 资源暴露的证书。这意味着我们不能使用 HTTP 隧道来测试由 Let's Encrypt 发行并由 Kubernetes 中的 ingress 资源使用的证书。这在以下图表中有所说明：

![](img/a5583460-7ef4-4f1a-a566-507395d3be49.png)

但是在提供证书的过程中，可以让 HTTP 隧道来使用，这时 Let's Encrypt 需要验证 ACME 客户端是否拥有它请求颁发证书的 DNS 名称。DNS 名称将是 HTTP 隧道分配给的主机名，例如，`6cc09528.ngrok.io`。一旦完成提供证书的过程，我们可以关闭 HTTP 隧道并将主机名重定向到 Minikube 实例的 IP 地址（使用本地的`/etc/hosts`文件）。这在以下图表中有所说明：

![](img/d42306ac-0d80-43ca-bbc8-f783f35353ed.png)

对于付费客户，`ngrok`提供了一个 TLS 隧道，它通过 HTTPS 流量而不是终止它；也就是说，发送 HTTPS 请求的客户端能够看到并验证 Kubernetes 中 ingress 资源暴露的证书。使用 TLS 隧道而不是 HTTP 隧道应该使这一额外步骤变得 unnecessary。

执行以下步骤以创建 HTTP 隧道：

1.  使用以下命令创建 HTTP 隧道：

```java
ngrok http https://minikube.me:443
```

1.  期望输出类似于以下屏幕截图：

![](img/cc23906d-f9c7-44e8-aa0d-da848bd6e750.png)

1.  拿起 HTTP 隧道的主机名，例如前面的示例中的`6cc09528.ngrok.io`，并将其保存在一个环境变量中，如下所示：

```java
NGROK_HOST=6cc09528.ngrok.io
```

在 HTTP 隧道就位的情况下，我们可以为使用 Cert Manager 和 Let's Encrypt 自动提供其证书的 ingress 资源的定义做好准备！

# 使用 Cert Manager 和 Let's Encrypt 提供证书

在配置 ingress 资源之前，了解提供证书的高级过程可能会有所帮助。使用 Cert Manager 和 Let's Encrypt 自动提供证书的过程如下：

![](img/ed4a016d-32e6-4381-af09-0f5d93641c42.png)

以下步骤将在提供过程中进行：

1.  创建了一个注有`certmanager.k8s.io/issuer: "name of a Let's Encrypt issuer"`的 ingress。

1.  这个注解将触发 Cert Manager 开始使用 Let's Encrypt 为 ingress 提供证书。

1.  在提供证书的过程中，Let's Encrypt 将执行一个`http-01`挑战，并使用 HTTP 隧道来验证 Cert Manager 拥有 DNS 名称。

1.  一旦配置完成，Cert Manager 将在 Kubernetes 中存储证书，并创建一个由入口指定的名称的秘密。

我们将添加一个新的入口`edge-ngrok`，定义在`ingress-edge-server-ngrok.yml`文件中，它将路由请求到 HTTP 隧道的主机名。这个入口将具有与现有入口相同的路由规则。不同的部分看起来像以下这样：

```java
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: edge-ngrok
  annotations:
    certmanager.k8s.io/issuer: "letsencrypt-issuer-staging"
spec:
  tls:
  - hosts:
    - xxxxxxxx.ngrok.io
    secretName: tls-ngrok-letsencrypt-certificate
  rules:
  - host: xxxxxxxx.ngrok.io
```

以下是对前面源代码的解释：

+   使用`certmanager.k8s.io/issuer: "letsencrypt-issuer-staging"`注解，我们请求 Cert Manager 使用名为`letsencrypt-issuer-staging`的发行版为这个入口配置证书。

+   `tls`和`rules`声明中的`xxxxxxxx.ngrok.io`主机名必须用你的 HTTP 隧道实际主机名替换。

+   一旦配置完成，证书将被存储在名为`tls-ngrok-letsencrypt-certificate`的秘密中。

在深入了解配置过程的高级层面并准备好使用它的入口资源之后，我们可以开始使用 Let's Encrypt 支持的两个环境来配置证书。首先从适合开发和测试活动的 staging 环境开始。

# 使用 Let's Encrypt 的 staging 环境

执行以下步骤从 Let's Encrypt 的 staging 环境配置证书并验证它是否工作：

1.  编辑`kubernetes/services/base/ingress-edge-server-ngrok.yml`文件，并将`xxxxxxxx.ngrok.io`用你的 HTTP 隧道的主机名替换两次！

    （之前的示例中的`6cc09528.ngrok.io`。）

1.  在启动配置之前，在另一个终端窗口中运行一个 watch 命令来监控证书的配置。运行以下命令：

```java
kubectl get cert --watch
```

1.  通过以下命令应用新的入口定义来启动配置：

```java
kubectl apply -f kubernetes/services/base/ingress-edge-server-ngrok.yml
```

1.  Cert Manager 现在将检测到新的入口并开始使用由`ngrok`设置的 HTTP 隧道通过 ACME v2 协议提供 Let's Encrypt staging 环境的证书。

1.  过了一会儿，你应该注意到在运行 HTTP 隧道的终端窗口中出现了`http-01`挑战。预期输出中将有一个像以下的请求：

![](img/48532285-e42c-41b8-87c5-953c433986d2.png)

1.  将创建一个`tls-ngrok-letsencrypt-certificate`证书，并将其存储在`tls-ngrok-letsencrypt-certificate`秘密中，如入口指定。预期`kubectl get cert --watch`命令的输出将与以下类似：

![](img/75548aa6-b6b1-4b86-b7f3-24a40a679869.png)

1.  过了一会儿，证书的`READY`状态将变为`True`，这意味着证书已经配置好，我们可以尝试使用它！

1.  为了尝试由 Let's Encrypt 提供的证书，我们需要将`ngrok`主机名重定向以直接指向 Minikube IP 地址。我们将向`/etc/hosts`文件中添加 HTTP 隧道的主机名，使其解析到 Minikube 实例的 IP 地址。这将导致本地请求发送到 HTTP 隧道的主机名被引导到 Minikube 实例，如下所示图所示：

![](img/e82140ea-df2b-41c7-9abd-a64f465ec373.png)

1.  编辑`/etc/hosts`文件，在我们在本章早些时候添加的行中添加 HTTP 隧道的主机名在`minikube.me`之后。编辑后，该行应类似于以下内容：

![](img/bc8718bb-49a9-4a2d-835a-7a07a6b48936.png)

1.  使用`keytool`命令查看 HTTP 隧道主机名暴露的证书：

```java
keytool -printcert -sslserver $NGROK_HOST:443 | grep -E "Owner:|Issuer:"
```

1.  期望收到如下响应：

![](img/02d1d310-1b40-4966-ab53-2a8b2a6dd43a.png)

如果您的`keytool`是本地化的，也就是说，它将其输出打印成除英语外的其他语言，您将需要更改前面`grep`命令使用的`Owner:|Issuer:`字符串，以使用本地化版本。

1.  证书是为 HTTP 隧道的主机名（在前一个示例中为`6cc09528.ngrok.io`）签发的，并由`Fake LE Intermediate X1`使用`Fake LE Root X1`作为其根 CA 签发。这证明 ingress 使用了 Let's Encrypt 的测试证书！

1.  通过使用相同的命令运行`test-em-all.bash`测试脚本总结：

```java
HOST=$NGROK_HOST PORT=443 ./test-em-all.bash
```

期望测试脚本输出通常的结果；检查它以如下结论结束：

![](img/b922e10e-eac0-41a4-8ca6-c714f34b08e5.png)

由 Let's Encrypt 测试环境提供的证书，如前所述，适合开发和测试活动。但由于其根 CA 不被网络浏览器信任，因此它们不能用于生产场景。让我们也尝试使用 Let's Encrypt 的生产环境，该环境能够提供可信的证书，尽管数量有限。

# 使用 Let's Encrypt 的生产环境

为了从 Let's Encrypt 的生产环境获取证书，而不是测试环境，我们必须在 ingress 定义中更改发行者，然后应用更新后的定义。执行以下步骤：

1.  编辑`kubernetes/services/base/ingress-edge-server-ngrok.yml`文件，并更改以下代码：

```java
certmanager.k8s.io/issuer: "letsencrypt-issuer-staging"
```

前面的代码现在应该如下所示：

```java
certmanager.k8s.io/issuer: "letsencrypt-issuer-prod"
```

1.  通过运行以下命令应用更改：

```java
kubectl apply -f kubernetes/services/base/ingress-edge-server-ngrok.yml
```

1.  监控`kubectl get cert --watch`命令的输出，并等待新证书被提供。在应用命令后立即，其就绪状态将更改为`False`，然后过一会儿将回到`True`。这意味着 Cert Manager 已经提供了由 Let's Encrypt 生产环境签发的证书！

1.  使用以下`keytool`命令检查证书：

```java
keytool -printcert -sslserver $NGROK_HOST:443 | grep -E "Owner:|Issuer:"
```

期望输出如下所示：

![](img/5d0e1392-a7d9-469e-9b9e-6c49555ebdcb.png)

1.  新证书就像之前为 HTTP 隧道的主机名发行的证书（在前一个示例中为 `6cc09528.ngrok.io`），但这次发行者和根 CA 来自生产环境。这意味着证书应该被网页浏览器信任。

1.  在本地网络浏览器中打开 `https://6cc09528.ngrok.io/actuator/health` URL（将 `6cc09528.ngrok.io` 替换为你的 HTTP 隧道的主机名）。如果你使用 Google Chrome 并点击证书图标（URL 前的锁），你应该会看到如下输出：

![](img/152c06f8-df1e-4c52-b633-84ac22f7717e.png)

如前一个屏幕截图所示，Chrome 报告：此证书有效！

1.  通过以下方式验证 `test-em-all.bash` 测试脚本是否也与此证书一起工作：

```java
HOST=$NGROK_HOST PORT=443 ./test-em-all.bash
```

期望测试脚本输出通常的结果；检查它是否以以下输出结束：

![](img/6d63d35e-1728-4d39-a779-d1f4424d0918.png)

您可以按照相同的过程返回 staging 发行者，同时在 ingress 定义中也将回到 staging 发行者。

# 清理

当你完成后，使用 Docker Compose 运行以下命令清理在 Kubernetes 中（可选在 Docker 中）创建的资源：

1.  使用 *Ctrl + C* 停止 `kubectl get cert --watch` 命令。

1.  使用 *Ctrl* *+* *C.* 停止 HTTP 隧道。

1.  使用以下命令在 Kubernetes 中删除命名空间：

```java
kubectl delete namespace hands-on
```

1.  如果你尝试使用 `./kubernetes/scripts/deploy-prod-env.bash` 脚本部署生产环境，你还需要停止使用 Docker Compose 启动的资源管理器。运行以下命令停止它们：

```java
docker-compose down mongodb mysql rabbitmq
```

既然我们已经自动化了证书的发放，让我们看看如何在没有 Kubernetes 的情况下验证微服务是否工作。让我们看看这是如何完成的。

# 验证在没有 Kubernetes 的情况下微服务是否工作

在本章和上一章中，我们看到了 Kubernetes 平台上的特性，如 config maps、secrets、服务以及 ingress 资源，如何简化开发合作微服务的景观的努力。但重要的是确保微服务的源代码在功能上不依赖于平台。避免此类锁定使得未来如果需要，可以以最小的努力切换到另一个平台。切换平台不应该需要更改源代码，而只需要更改微服务的配置。

使用 Docker Compose 和 `test-em-all.bash` 测试脚本测试微服务，可以确保从功能角度来说它们是工作的，这意味着它们将验证微服务源代码中的功能在没有 Kubernetes 的情况下仍然有效。在没有 Kubernetes 的情况下运行微服务时，我们将失去 Kubernetes 为我们提供的非功能性特性，例如监控、扩展和重启容器。

当使用 Docker Compose 时，我们将映射以下 Kubernetes 特性：

+   而不是使用配置图，我们使用卷，直接从宿主文件系统映射配置文件。

+   而不是使用密钥，我们将敏感信息（如凭据）保存在`.env`文件中。

+   而不是入口，我们将使用 Spring Cloud Gateway。

+   代替服务，我们将将客户端使用的主机名直接映射到容器的主机名，这意味着我们将不实施任何服务发现，并且无法扩展容器。

使用这种方式的 Docker Compose 将在非功能性方面与使用 Kubernetes 相比具有显著劣势。但是，鉴于 Docker Compose 只用于运行功能测试，这是可以接受的。

在我们使用 Docker Compose 运行测试之前，让我们查看一下`docker-compose*.yml`文件中的代码更改。

# Docker Compose 源代码的变化

要在 Kubernetes 之外使用 Docker Compose 运行微服务，对`docker-compose*.yml`文件应用了以下更改：

+   移除了配置服务器定义

+   移除了以下配置服务器环境变量的使用：`CONFIG_SERVER_USR`和`CONFIG_SERVER_PWD`

+   在需要从配置仓库读取配置文件的每个容器中，将`config-repo`文件夹作为卷映射。

+   定义了`SPRING_CONFIG_LOCATION`环境变量，以指向配置仓库中的配置文件。

+   将敏感信息如凭据和密码存储在 Docker Compose 的`.env`文件中的 TLS 证书中

+   使用在`.env`文件中定义的变量，为访问资源管理器的凭据定义环境变量。

例如，在`docker-compose.yml`中，`product`微服务的配置如下所示：

```java
product:
  build: microservices/product-service
  image: hands-on/product-service
  environment:
    - SPRING_PROFILES_ACTIVE=docker
    - SPRING_CONFIG_LOCATION=file:/config-repo/application.yml,file:/config-repo/product.yml
    - SPRING_RABBITMQ_USERNAME=${RABBITMQ_USR}
    - SPRING_RABBITMQ_PASSWORD=${RABBITMQ_PWD}
    - SPRING_DATA_MONGODB_AUTHENTICATION_DATABASE=admin
    - SPRING_DATA_MONGODB_USERNAME=${MONGODB_USR}
    - SPRING_DATA_MONGODB_PASSWORD=${MONGODB_PWD}
  volumes:
    - $PWD/config-repo:/config-repo
```

以下是前述源代码的解释：

+   `config-repo`文件夹作为卷映射到容器的`/config-repo`。

+   `SPRING_CONFIG_LOCATION`环境变量告诉 Spring 在哪里找到属性文件，在本例中，是`/config-repo/application.yml`和`/config-repo/product.yml`文件。

+   根据`.env`文件的内容，设置访问 RabbitMQ 和 MongoDB 的凭据作为环境变量。

前述源代码中提到的凭据在`.env`文件中定义为：

```java
RABBITMQ_USR=rabbit-user-prod
RABBITMQ_PWD=rabbit-pwd-prod
MONGODB_USR=mongodb-user-prod
MONGODB_PWD=mongodb-pwd-prod
```

# 使用 Docker Compose 进行测试

要用 Docker Compose 进行测试，我们将使用 Docker Desktop（以前称为 Docker for macOS）而不是 Minikube。执行以下步骤：

1.  要以 Docker Desktop 而不是 Minikube 运行 Docker 客户端，请运行以下命令：

```java
eval $(minikube docker-env --unset)
```

1.  为了节省内存，您可能想要停止 Minikube 实例：

```java
minikube stop 
```

1.  启动 Docker Desktop（如果尚未运行）。

1.  使用以下命令在 Docker Desktop 中构建 Docker 镜像：

```java
docker-compose build
```

1.  使用 RabbitMQ（每个主题一个分区）运行测试：

```java
COMPOSE_FILE=docker-compose.yml ./test-em-all.bash start stop
```

测试应该从启动所有容器开始，运行测试，最后停止所有容器。预期输出如下：

![](img/949cd3b9-d551-4336-a25d-a256cae5ad6e.png)

1.  可选地，使用具有多个分区的 RabbitMQ 运行测试：

```java
COMPOSE_FILE=docker-compose-partitions.yml ./test-em-all.bash start stop
```

预期输出应该与前面的测试类似。

1.  或者，使用具有多个分区的 Kafka 运行测试：

```java
COMPOSE_FILE=docker-compose-kafka.yml ./test-em-all.bash start stop
```

预期输出应该与前面的测试类似。

1.  停止 Docker Desktop 以节省内存。

1.  如果之前停止了 Minikube 实例，请启动它，并将默认命名空间设置为`hands-on`：

```java
minikube start
kubectl config set-context $(kubectl config current-context) --namespace=hands-on
```

1.  将 Docker 客户端指向 Minikube 实例中的 Kubernetes 集群：

```java
eval $(minikube docker-env)
```

通过这些测试的成功执行，我们验证了在没有 Kubernetes 的情况下微服务也能正常工作。

# 总结

在本章中，我们看到了 Kubernetes 的功能如何简化微服务架构，这意味着我们需要开发的支撑服务数量减少了，这些服务原本需要和微服务一起部署。我们看到了如何使用 Kubernetes 的配置映射（config maps）和机密（secrets）来替代 Spring Cloud Config Server，以及如何使用 Kubernetes 入口（ingress）来替代基于 Spring Cloud Gateway 的边缘服务。

使用 Cert Manager 和 Let's Encrypt 允许我们自动为 ingress 暴露的 HTTPS 端点提供证书，从而消除了手动且繁琐的工作。由于我们运行在本地 Minikube 实例中的 Kubernetes 集群无法从互联网访问，我们使用了`ngrok`来建立从互联网到 Minikube 实例的 HTTP 隧道。HTTP 隧道被 Let's Encrypt 用来验证我们是否是请求证书的 DNS 名的所有者。

为了验证微服务的源代码可以在其他平台上运行，也就是说不是锁定在 Kubernetes 上，我们使用 Docker Compose 部署了微服务并运行了`test-em-all.bash`测试脚本。

在下一章中，我们将介绍服务网格（service mesh）的概念，并学习如何使用服务网格产品**Istio**来改善部署在 Kubernetes 上的合作微服务景观的可观测性、安全性、弹性和路由。

前往下一章！

# 问题

1.  Spring Cloud Config Server 是如何被 Kubernetes 资源替代的？

1.  Spring Cloud Gateway 是如何被 Kubernetes 资源替代的？

1.  ACME 代表什么，它有什么用途？

1.  Cert Manager 和 Let's Encrypt 在自动化提供证书中扮演什么角色？

1.  在自动化证书提供过程中涉及哪些 Kubernetes 资源？

1.  我们为什么使用`ngrok`，以及如何修改以去除对`ngrok`的使用？

1.  我们为什么使用 Docker Compose 运行测试？
