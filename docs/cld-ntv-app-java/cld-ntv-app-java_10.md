# 平台部署 - AWS

在本章中，我们将介绍亚马逊 AWS 平台提供的一些部署选项。 AWS 平台是云服务提供商中最古老和最成熟的之一。它于 2002 年推出，并自那时以来一直是该领域的领导者。 AWS 还不断创新，并推出了几项新服务，这些服务在广泛的客户群中得到了广泛的采用，从单人创业公司到企业。

在本章中，我们将涵盖以下主题：

+   AWS 平台

+   AWS 平台部署选项

# AWS 平台

亚马逊 AWS 是云计算的先驱，并自那时起一直在扩展其云服务，以保持其领先地位。以下图表提供了 AWS 平台为应用程序开发人员提供的服务的指示性列表：

![](img/efc34941-6e16-428a-a07d-c08cdc5dc74e.jpg)

这只是一个指示性列表，绝不是详尽的列表；请参阅亚马逊 AWS 门户网站获取完整列表。

类别如下：

+   基础设施：这可能是 AWS 平台的核心，使其能够提供大量其他服务。这些可以进一步分类为：

+   计算：诸如 EC2，Lambda，ECS 和 ELB 之类的服务。我们将演示使用主要计算服务部署我们的示例应用程序，但是将它们与 AWS 提供的其他服务相结合相对容易。

+   存储：诸如 S3，EBS 和 CloudFront 之类的服务。

+   网络：诸如 VPC，Route53 和 DirectConnect 之类的服务。

+   应用程序：这些服务可用作构建和支持应用程序的组件。

+   数据库：这些服务针对数据库，提供对不同的关系数据库管理系统（RDBMS）和 NoSQL 数据存储的访问。

+   DevOps：这些服务提供了构建流水线和启用持续交付的能力。这些包括源代码托管，持续集成工具以及云和软件供应工具。

+   安全性：这些服务为 AWS 提供了基于角色的访问控制（RBAC），并提供了一种机制来指定配额并强制执行它们，密钥管理和秘密存储。

+   移动：这些服务旨在为移动应用程序和通知等服务提供后端。

+   分析：这些服务包括 MapReduce 等批处理系统，以及 Spark 等流处理系统，可用于构建分析平台。

# AWS 平台部署选项

在 AWS 平台提供的各种服务中，我们将重点关注本章涵盖的一些部署选项，这些选项专门针对我们一直作为示例使用的 Web API 类型。因此，我们将介绍部署到以下内容：

+   AWS Elastic Beanstalk

+   AWS 弹性容器服务

+   AWS Lambda

由于我们将在云环境中运行应用程序，因此我们将不需要直接管理基础设施，也就是说，我们将不会启动虚拟机并在其中安装应用程序，因此我们将不需要服务发现，因为弹性负载均衡器将自动路由到所有正在运行的应用程序实例。因此，我们将使用不使用 Eureka 发现客户端的“产品”API 的版本：

```java
package com.mycompany.product;

 import org.springframework.boot.SpringApplication;
 import org.springframework.boot.autoconfigure.SpringBootApplication;

 @SpringBootApplication
 public class ProductSpringApp {

    public static void main(String[] args) throws Exception {
       SpringApplication.run(ProductSpringApp.class, args);
    }

 }
```

# 将 Spring Boot API 部署到 Beanstalk

AWS Elastic Beanstalk（AEB）是 AWS 提供的一项服务，可在 AWS 上托管 Web 应用程序，而无需直接提供或管理 IaaS 层。 AEB 支持流行的语言，如 Java，.NET，Python，Ruby，Go 和 PHP。最近，它还提供了运行 Docker 容器的支持。我们将采用我们迄今为止在旅程中构建的“产品”服务的简化版本，并将其部署在 AEB 中作为可运行的 JAR 文件，也作为 Docker 容器。

# 部署可运行的 JAR

登录到 AWS 控制台，选择计算类别下的弹性 Beanstalk 服务，并点击“开始”按钮：

![](img/b572ac6b-6c97-4e58-a2b2-88e582e21ae1.png)

在下一个屏幕中填写应用程序详细信息：

![](img/560dd240-24b2-400f-a8a6-c83ad680c952.png)

上传`target`文件夹中的`product.jar`，然后点击“配置更多选项”按钮。您将看到不同的类别，可以通过选择软件，在环境属性下，添加一个名为`SERVER_PORT`的新环境变量，并将值设置为`5000`。这是必要的，因为默认情况下，AEB 环境创建的 NGINX 服务器将代理所有请求到这个端口，通过设置变量，我们确保我们的 Spring Boot 应用将在端口`5000`上运行：

![](img/39e95de8-1379-47e4-a8d0-2455339e0eb3.png)

现在，AWS 将提供一个新的环境，我们的应用程序将在其中运行：

![](img/c31356a5-b7c7-4e21-ab0d-4a3a5e0158a7.png)

环境创建完成后，AEB 将为应用程序生成一个 URL：

![](img/1bebd5f7-5986-40b7-b578-f6fe06d55583.png)

我们可以使用此 URL 访问 API 端点：

![](img/2451c2fd-4a87-4755-94d3-544a8d0996f4.png)

# 部署 Docker 容器

现在我们已经学会了如何将可运行的 JAR 部署到弹性 Beanstalk 服务，让我们也看一下相同的变体，我们将部署运行相同应用程序的 Docker 容器。使用 Docker 容器的优势在于，我们可以使用 AWS 弹性 Beanstalk 服务尚未支持的语言和平台，并且仍然可以在云中部署它们，从而获得该服务提供的好处。

对于此部署，我们将使用**弹性容器服务**（**ECS**）提供的 Docker 注册表来存储我们从应用程序构建的 Docker 容器。当我们部署到 ECS 时，我们将介绍如何将本地 Docker 容器推送到 ECS 存储库。现在，让我们假设我们要部署的 Docker 容器在名为`<aws-account-id>.dkr.ecr.us-west-2.amazonaws.com/product-api`的存储库中可用。由于我们需要访问此存储库，我们需要将 AmazonEC2ContainerRegistryReadOnly 策略添加到默认的弹性 Beanstalk 角色 aws-elasticbeanstalk-ec2-role。

这可以在 IAM 控制台的角色部分完成：

![](img/df0c9e42-5a70-4c5d-815e-458437dfef96.png)

创建一个名为`Dockerfile.aws.json`的文件，内容如下：

```java
{ 
  "AWSEBDockerrunVersion": "1", 
  "Image": { 
    "Name": "<aws-account-id>.dkr.ecr.us-west-2.amazonaws.com/product-api", 
    "Update": "true" 
  }, 
  "Ports": [ 
    { 
      "ContainerPort": "8080" 
    } 
  ] 
} 
```

现在我们准备部署我们的 Docker 容器。在弹性 Beanstalk 控制台中，我们将选择单个 Docker 容器而不是 Java，并创建一个新的应用程序：

![](img/8a07d769-e612-486c-8d5e-9d1619d4d81a.png)

选择并上传`Dockerfile.aws.json`以创建环境：

![](img/db25cea9-3dd0-4102-8643-40ef14503fe7.png)

我们可以测试我们的 API 端点，以验证我们的 Docker 容器是否正常运行。我们还可以配置容器使用 Amazon CloudWatch 日志记录和监控，以更好地监视我们的应用程序：

![](img/72b589b2-40fd-439c-aaee-774137effd99.png)

# 将 Spring Boot 应用程序部署到弹性容器服务

AWS **弹性容器服务**（**ECS**）是一项允许用户使用托管的 Docker 实例部署应用程序的服务。在这里，AWS ECS 服务负责提供虚拟机和 Docker 安装。我们可以通过以下步骤部署我们的应用程序：

1.  启动 ECS，点击“继续”：

![](img/9c5dd2c2-6787-4539-91ca-bdb9b28e8b9f.png)

1.  创建名为`product-api`的 ECS 存储库，然后点击“下一步”：

![](img/2be15da6-f07b-4c03-9688-e1148fc1d326.png)

1.  构建并推送 Docker 容器到存储库，按照屏幕上给出的说明进行：

![](img/a35fb748-ff23-4d25-b394-590ea2f14536.png)

1.  GUI 生成的 Docker 登录命令多了一个`http://`，应该去掉：

![](img/e843f488-de3c-4c27-9d49-c59bc9ad4c70.png)

1.  我们现在可以构建并推送 Docker 容器到创建的存储库：

![](img/e8072215-a0ca-40b5-9e1c-8a015e9d383b.png)

1.  在配置任务定义时，我们将使用此容器存储库：

![](img/39e6452a-6c6f-485b-b71c-f4fb03a62bcd.png)

1.  在高级选项中，我们可以配置 AWS CloudWatch 日志记录，以捕获来自 Docker 容器的日志，在存储和日志记录部分下：

![](img/7a1d2053-56e2-4371-8b5a-5100beb71a59.png)

1.  我们需要在 CloudWatch 控制台中创建相应的日志组，以捕获从我们的应用程序创建的日志：

![](img/302e0cad-c1ff-4f58-ad4b-e635e80d89a9.png)

1.  我们可以创建一个服务映射到容器中公开的端口，即`8080：`

![](img/16a2d1fc-b921-41d6-8b8f-e987a1201dff.png)

1.  可选地，我们可以描绘 EC2 实例类型并配置密钥对，以便我们能够登录到 ECS 将为我们的应用程序创建的 EC2 实例中：

![](img/b96f9480-7312-4b84-9a4d-79e12335d242.png)

1.  一旦我们审查配置并提交，ECS 将开始创建 EC2 实例并将我们的应用程序部署到其中：

![](img/bd112413-b82d-47df-add8-b3c08c6bd29c.png)

1.  我们可以点击自动扩展组并找到已启动的实例：

![](img/8be45561-dffd-4b6a-873e-e73cf39ff2cc.png)

1.  找到实例：

![](img/66100781-e44f-4520-b946-4bc202fc41fa.png)

1.  找到实例主机名：

![](img/eecd2566-2434-4c5a-ba2d-4e70088f4358.png)

1.  通过实例主机名访问应用程序：

![](img/d76205d8-b879-47e8-a636-a304f16d3050.png)

但是逐个通过它们的主机名访问应用程序是不可行的，因此，我们将创建一个弹性负载均衡器，它将路由请求到实例，从而允许我们在扩展或缩减时拥有稳定的端点：

1.  我们将转到 EC2 控制台，并在应用程序负载均衡器下选择创建：

![](img/93c7bf88-f0f4-4a01-a16e-54f6b6baa465.png)

1.  配置负载均衡器端口：

![](img/4a349305-fd46-46b9-b707-cd4ef596de84.png)

1.  配置目标组和健康检查端点：

![](img/570e525d-cd1c-419d-b3e0-19e5713b62ba.png)

1.  将目标实例注册到我们的集群定义创建的实例：

![](img/0dc9ccdb-c7b7-4bb6-8b7b-7cd70c4b5b05.png)

1.  找到负载均衡器的 DNS 记录：

![](img/21b3b419-d99c-4c84-8884-989aa0532e6f.png)

1.  连接到负载均衡器端点并验证应用程序是否正常工作：

![](img/25bb98e1-320c-4504-953d-0bbc2c88468f.png)

# 部署到 AWS Lambda

AWS Lambda 服务允许部署简单函数以在事件触发器上调用。这些事件触发器可以分为四种类型，即：

+   数据存储（例如，AWS DyanmoDB）

+   队列和流（例如，AWS Kinesis）

+   Blob 存储（例如，AWS S3）

+   API 数据门户：

![](img/bc0e31ba-40c8-477a-a75a-ced5421df4fd.jpg)

AWS Lamda 支持的事件源的完整列表可以在[`docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html#api-gateway-with-lambda.`](https://docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html#api-gateway-with-lambda)找到

与之前讨论的其他部署选项不同，AWS Lambda 提供了最透明的扩展选项，AWS 平台根据需求自动扩展所需的实例。我们无需配置实例、负载均衡器等，而是可以专注于应用程序逻辑。

我们现在将构建一个简单的 AWS Lambda 函数，并将其绑定到 API 端点以调用它。

我们将首先创建一个新的 Spring Boot 应用程序，具有以下依赖项。我们还将使用`maven-shade-plugin`创建可运行的 JAR：

```java
<project  
          xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
   <groupId>com.mycompany</groupId>
   <artifactId>hello-lambda</artifactId>
   <version>0.0.1-SNAPSHOT</version>

   <dependencies>
     <dependency>
       <groupId>junit</groupId>
       <artifactId>junit</artifactId>
       <version>4.12</version>
       <scope>test</scope>
     </dependency>
     <dependency>
       <groupId>com.amazonaws</groupId>
       <artifactId>aws-lambda-java-core</artifactId>
       <version>1.1.0</version>
     </dependency>
     <dependency>
       <groupId>com.amazonaws</groupId>
       <artifactId>aws-lambda-java-events</artifactId>
       <version>2.0.1</version>
     </dependency>
     <dependency>
       <groupId>com.amazonaws</groupId>
       <artifactId>aws-lambda-java-log4j2</artifactId>
       <version>1.0.0</version>
     </dependency>
   </dependencies>

   <build>
     <finalName>hello-lambda</finalName>
     <plugins>
       <plugin>
         <groupId>org.apache.maven.plugins</groupId>
         <artifactId>maven-compiler-plugin</artifactId>
         <configuration>
           <source>1.8</source>
           <target>1.8</target>
         </configuration>
       </plugin>
       <plugin>
         <groupId>org.apache.maven.plugins</groupId>
         <artifactId>maven-shade-plugin</artifactId>
         <version>3.0.0</version>
         <configuration>
           <createDependencyReducedPom>false</createDependencyReducedPom>
         </configuration>
         <executions>
           <execution>
             <phase>package</phase>
             <goals>
               <goal>shade</goal>
             </goals>
           </execution>
         </executions>
       </plugin>
     </plugins>
   </build>

 </project>
```

现在创建`HelloHandler.java`，内容如下：

```java
package com.mycompany;

 import com.amazonaws.services.lambda.runtime.Context;
 import com.amazonaws.services.lambda.runtime.RequestHandler;
 import com.amazonaws.services.lambda.runtime.events.APIGatewayProxyRequestEvent;
 import com.amazonaws.services.lambda.runtime.events.APIGatewayProxyResponseEvent;

 import java.net.HttpURLConnection;

 public class HelloHandler implements RequestHandler<APIGatewayProxyRequestEvent, APIGatewayProxyResponseEvent> {

   @Override
   public APIGatewayProxyResponseEvent handleRequest(APIGatewayProxyRequestEvent request, Context context) {
     String who = "World";
     if ( request.getPathParameters() != null ) {
       String name  = request.getPathParameters().get("name");
       if ( name != null && !"".equals(name.trim()) ) {
         who = name;
       }
     }
     return new APIGatewayProxyResponseEvent().withStatusCode(HttpURLConnection.HTTP_OK).withBody(String.format("Hello %s!", who));
   }

 }
```

由于 lambda 函数是简单的函数，我们可以通过使用函数的输入和输出很容易地测试它们。例如，一个示例测试用例可能是：

```java
package com.mycompany;

 import com.amazonaws.services.lambda.runtime.events.APIGatewayProxyRequestEvent;
 import com.amazonaws.services.lambda.runtime.events.APIGatewayProxyResponseEvent;
 import org.junit.Before;
 import org.junit.Test;
 import org.junit.runner.RunWith;
 import org.junit.runners.BlockJUnit4ClassRunner;

 import java.util.Collections;
 import java.util.HashMap;
 import java.util.Map;

 import static org.junit.Assert.*;

 @RunWith(BlockJUnit4ClassRunner.class)
 public class HelloHandlerTest {

   HelloHandler handler;
   APIGatewayProxyRequestEvent input;
   @Before
   public void setUp() throws Exception {
     handler = new HelloHandler();
     Map<String, String> pathParams = new HashMap<>();
     pathParams.put("name", "Universe");
     input = new APIGatewayProxyRequestEvent().withPath("/hello").withPathParamters(pathParams);
   }

   @Test
   public void handleRequest() {
     APIGatewayProxyResponseEvent res = handler.handleRequest(input, null);
     assertNotNull(res);
     assertEquals("Hello Universe!", res.getBody());
   }
   @Test
   public void handleEmptyRequest() {
     input.withPathParamters(Collections.emptyMap());
     APIGatewayProxyResponseEvent res = handler.handleRequest(input, null);
     assertNotNull(res);
     assertEquals("Hello World!", res.getBody());
   }
 }
```

现在我们可以使用 Maven 构建 lambda 函数：

```java
$ mvn clean package 
[INFO] Scanning for projects... 
[WARNING] 
[WARNING] Some problems were encountered while building the effective model for com.mycompany:hello-lambda:jar:0.0.1-SNAPSHOT 
[WARNING] 'build.plugins.plugin.version' for org.apache.maven.plugins:maven-compiler-plugin is missing. @ line 35, column 15 
[WARNING] 
[WARNING] It is highly recommended to fix these problems because they threaten the stability of your build. 
[WARNING] 
[WARNING] For this reason, future Maven versions might no longer support building such malformed projects. 
[WARNING] 
[INFO] 
[INFO] ------------------------------------------------------------------------ 
[INFO] Building hello-lambda 0.0.1-SNAPSHOT 
[INFO] ------------------------------------------------------------------------ 
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ hello-lambda --- 
[INFO] Deleting /Users/shyam/workspaces/msa-wsp/CloudNativeJava/chapter-09/hello-lambda/target 
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ hello-lambda --- 
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent! 
[INFO] skip non existing resourceDirectory /Users/shyam/workspaces/msa-wsp/CloudNativeJava/chapter-09/hello-lambda/src/main/resources 
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ hello-lambda --- 
[INFO] Changes detected - recompiling the module! 
[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent! 
[INFO] Compiling 1 source file to /Users/shyam/workspaces/msa-wsp/CloudNativeJava/chapter-09/hello-lambda/target/classes 
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ hello-lambda --- 
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent! 
[INFO] skip non existing resourceDirectory /Users/shyam/workspaces/msa-wsp/CloudNativeJava/chapter-09/hello-lambda/src/test/resources 
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ hello-lambda --- 
[INFO] Changes detected - recompiling the module! 
[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent! 
[INFO] Compiling 1 source file to /Users/shyam/workspaces/msa-wsp/CloudNativeJava/chapter-09/hello-lambda/target/test-classes 
[INFO] 
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ hello-lambda --- 
[INFO] Surefire report directory: /Users/shyam/workspaces/msa-wsp/CloudNativeJava/chapter-09/hello-lambda/target/surefire-reports 

------------------------------------------------------- 
 T E S T S 
------------------------------------------------------- 
Running com.mycompany.HelloHandlerTest 
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.055 sec 

Results : 

Tests run: 2, Failures: 0, Errors: 0, Skipped: 0 

[INFO] 
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ hello-lambda --- 
[INFO] Building jar: /Users/shyam/workspaces/msa-wsp/CloudNativeJava/chapter-09/hello-lambda/target/hello-lambda.jar 
[INFO] 
[INFO] --- maven-shade-plugin:3.0.0:shade (default) @ hello-lambda --- 
[INFO] Including com.amazonaws:aws-lambda-java-core:jar:1.1.0 in the shaded jar. 
[INFO] Including com.amazonaws:aws-lambda-java-events:jar:2.0.1 in the shaded jar. 
[INFO] Including joda-time:joda-time:jar:2.6 in the shaded jar. 
[INFO] Including com.amazonaws:aws-lambda-java-log4j2:jar:1.0.0 in the shaded jar. 
[INFO] Including org.apache.logging.log4j:log4j-core:jar:2.8.2 in the shaded jar. 
[INFO] Including org.apache.logging.log4j:log4j-api:jar:2.8.2 in the shaded jar. 
[INFO] Replacing original artifact with shaded artifact. 
[INFO] Replacing /Users/shyam/workspaces/msa-wsp/CloudNativeJava/chapter-09/hello-lambda/target/hello-lambda.jar with /Users/shyam/workspaces/msa-wsp/CloudNativeJava/chapter-09/hello-lambda/target/hello-lambda-0.0.1-SNAPSHOT-shaded.jar 
[INFO] ------------------------------------------------------------------------ 
[INFO] BUILD SUCCESS 
[INFO] ------------------------------------------------------------------------ 
[INFO] Total time: 2.549 s 
[INFO] Finished at: 2018-02-12T13:52:14+05:30 
[INFO] Final Memory: 25M/300M 
[INFO] ------------------------------------------------------------------------ 
```

我们现在已经构建了`hello-lambda.jar`，我们将上传到 AWS 控制台中创建的 AWS Lambda 函数。

1.  我们将首先转到 API Gateway 控制台，该控制台出现在 AWS 控制台的网络和内容交付类别中，并创建一个新的 API：

![](img/334bc46e-e18e-404b-88d5-dc128fed3166.png)

1.  我们将为路径`/hello`添加一个名为`hello`的新资源：

![](img/fa8a523f-d9d8-4c20-b099-165ea8f92079.png)

1.  我们还将创建一个带有路径参数的子资源：

![](img/f55c4fd3-9201-4afb-844b-5b0b314462ae.png)

1.  现在，我们将附加 HTTP `GET`方法：

![](img/8ceae71e-6427-4b32-8373-2ea5946d2b3a.png)

1.  创建一个具有以下详细信息的 Lambda 函数：

![](img/fde71ba7-3782-4c20-b3e3-fc912d621c42.png)

1.  上传可运行的 JAR 并设置处理程序方法：

![](img/36753b3c-d25b-4226-9c6d-54ecd4efa730.png)

1.  现在将此 Lambda 函数添加到 API 方法中：

![](img/8d953143-4597-431a-ad81-ade0d5a06fde.png)

1.  确保选择使用 Lambda 代理集成，以便我们可以使用特定的`RequestHandler`接口，而不是使用通用的`RequestStreamHandler`。这也将使 API Gateway 获得对 Lambda 函数的权限：

![](img/aba7b22e-dfa9-4368-b3bc-978479ca4ea9.png)

1.  使用 Lambda 函数调用完成 API 定义：

![](img/a9a15393-71ec-4d46-a7e5-77d41ab348a6.png)

1.  我们可以从控制台测试 API 端点：

![](img/400e475d-1e2c-451a-8ff9-01a6f1ff1f2e.png)

1.  现在我们可以部署 API：

![](img/d90a2915-d03f-4fa9-ad68-97edba4fa417.png)

1.  成功部署 API 将导致 API 端点：

![](img/439bb781-b264-4f30-9f52-5a81dff1e1aa.png)

1.  现在我们可以使用为此部署环境生成的 API 端点来访问应用程序：

![](img/c2afc5c0-6922-48c9-8f82-1cde13ab4ed5.png)

# 总结

在本章中，我们介绍了 AWS 平台提供的一些选项，以及我们如何可以从弹性 Beanstalk 部署我们的应用程序，这是针对 Web 应用程序的。我们部署到 ECS，用于部署容器化工作负载，不限于 Web 应用程序工作负载。然后，我们部署了一个 AWS Lambda 函数，无需配置底层硬件。在接下来的章节中，我们将看一下使用 Azure 进行部署，以了解它为部署云原生应用程序提供的一些服务。
