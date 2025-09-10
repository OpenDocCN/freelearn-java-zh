# 第七章。最佳实践

在本书中，到目前为止，我们讨论了与 Maven 相关的多数关键概念。在本章中，我们将重点关注与所有这些核心概念相关的最佳实践。以下最佳实践是创建一个成功且高效的构建环境的基本要素。以下标准将帮助您评估您的 Maven 项目的效率，尤其是如果您正在处理一个大规模的多模块项目：

+   开发者开始新项目并添加到构建系统所需的时间

+   升级项目所有模块中依赖项版本所需的努力

+   使用全新的本地 Maven 仓库构建完整项目所需的时间

+   完全离线构建所需的时间

+   更新项目生成的 Maven 工件版本所需的时间；例如，从 1.0.0-SNAPSHOT 到 1.0.0

+   完全新的开发者理解您的 Maven 构建做什么所需的时间

+   引入新的 Maven 仓库所需的努力

+   执行单元测试和集成测试所需的时间

本章的其余部分将讨论 25 个行业公认的最佳实践，这些实践将帮助您提高开发者的生产力并减少任何维护噩梦。

# 依赖项管理

在以下示例中，您会注意到依赖项版本被添加到应用 POM 文件中定义的每个依赖项：

```java
<dependencies>
  <dependency>
    <groupId>com.nimbusds</groupId>
    <artifactId>nimbus-jose-jwt</artifactId>
    <version>2.26</version>
  </dependency>
  <dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.2</version>
  </dependency>
</dependencies>
```

假设您在一个多模块项目中有一组具有相同依赖项集的应用 POM 文件。如果您在每个依赖项中都重复了工件版本，那么要升级到最新依赖项，您需要更新所有 POM 文件，这很容易导致混乱。

不仅如此，如果您在同一个项目的不同模块中使用不同版本的相同依赖项，那么在出现问题时，调试将变成一场噩梦。

使用`dependencyManagement`，我们可以克服这两个问题。如果是一个多模块 Maven 项目，您需要在父 POM 中引入`dependencyManagement`，这样它就会被所有其他子模块继承：

```java
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>com.nimbusds</groupId>
      <artifactId>nimbus-jose-jwt</artifactId>
      <version>2.26</version>
    </dependency>
    <dependency>
      <groupId>commons-codec</groupId>
      <artifactId>commons-codec</artifactId>
      <version>1.2</version>
    </dependency>
  </dependencies>
</dependencyManagement>
```

一旦在`dependencyManagement`部分下定义了`dependencies`，如前述代码所示，您只需引用`groupId`和`artifactId`元素中的`dependency`。`version`元素从相应的`dependencyManagement`部分中选取：

```java
<dependencies>
  <dependency>
    <groupId>com.nimbusds</groupId>
    <artifactId>nimbus-jose-jwt</artifactId>
  <dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
  </dependency>
</dependencies>
```

使用这种方式，如果您想升级或降级一个依赖项，您只需更改`dependencyManagement`部分下的依赖项版本。

同样的原则也适用于插件。如果您有一组在多个模块中使用的插件，您应该在父模块的`pluginManagement`部分中定义它们。这样，您只需更改父 POM 中的`pluginManagement`部分，就可以无缝地降级或升级插件版本，如下述代码所示：

```java
<pluginManagement>
  <plugins>
    <plugin>
      <artifactId>maven-resources-plugin</artifactId>
      <version>2.4.2</version>
    </plugin>
    <plugin>
      <artifactId>maven-site-plugin</artifactId>
      <version>2.0-beta-6</version>
    </plugin>
    <plugin>
      <artifactId>maven-source-plugin</artifactId>
      <version>2.0.4</version>
    </plugin>
    <plugin>
       <artifactId>maven-surefire-plugin</artifactId>
	   <version>2.13</version>
    </plugin>
  </plugins>
</pluginManagement>
```

一旦你在插件管理部分定义了插件，如前述代码所示，你只需引用其`groupId`（可选）和`artifactId`元素即可。版本从适当的`pluginManagement`部分选择：

```java
<plugins>      
  <plugin>
    <artifactId>maven-resources-plugin</artifactId>
    <executions>……</executions>
  </plugin>
  <plugin>
    <artifactId>maven-site-plugin</artifactId>
    <executions>……</executions>
  </plugin>
  <plugin>
    <artifactId>maven-source-plugin</artifactId>
    <executions>……</executions>
  </plugin>
  <plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <executions>……</executions>
  </plugin>
</plugins>
```

在第四章*Maven 插件*中详细讨论了 Maven 插件。

# 定义父模块

在大多数多模块 Maven 项目中，有许多东西是在多个模块之间共享的。依赖版本、插件版本、属性和仓库只是其中的一部分。创建一个名为`parent`的单独模块并在其 POM 文件中定义所有公共内容是一种常见的（也是最佳）实践。此 POM 文件的打包类型为`pom`。由`pom`打包类型生成的工件本身就是一个 POM 文件。

下面是一些 Maven 父模块的示例：

+   **Apache Axis2 项目**: [`svn.apache.org/repos/asf/axis/axis2/java/core/trunk/modules/parent/`](http://svn.apache.org/repos/asf/axis/axis2/java/core/trunk/modules/parent/)

+   **WSO2 Carbon 项目**: [`svn.wso2.org/repos/wso2/carbon/platform/trunk/parent/`](https://svn.wso2.org/repos/wso2/carbon/platform/trunk/parent/)

并非所有项目都遵循这种方法。有些项目只是将父 POM 文件放在根目录下（而不是在`parent`模块下）。以下是一些示例：

+   **Apache Synapse 项目**: [`svn.apache.org/repos/asf/synapse/trunk/java/pom.xml`](http://svn.apache.org/repos/asf/synapse/trunk/java/pom.xml)

+   **Apache HBase 项目**: [`svn.apache.org/repos/asf/hbase/trunk/pom.xml`](http://svn.apache.org/repos/asf/hbase/trunk/pom.xml)

这两种方法都能达到相同的结果，但第一种方法更受欢迎。在第一种方法中，父 POM 文件仅定义了项目内不同 Maven 模块之间的共享资源，同时在项目根目录下还有一个 POM 文件，它定义了要包含在项目构建中的所有模块。在第二种方法中，你将在项目根目录下的同一个 POM 文件中定义所有共享资源以及要包含在项目构建中的所有模块。基于*关注点分离*原则，第一种方法比第二种方法更好。

# POM 属性

你可以在 Maven 应用 POM 文件中使用六种类型的属性：

+   内置属性

+   项目属性

+   本地设置

+   环境变量

+   Java 系统属性

+   自定义属性

始终建议你在应用 POM 文件中使用属性而不是硬编码值。让我们看看几个例子。

让我们考虑 Apache Axis2 分发模块中的应用 POM 文件示例，该模块可在[`svn.apache.org/repos/asf/axis/axis2/java/core/trunk/modules/distribution/pom.xml`](http://svn.apache.org/repos/asf/axis/axis2/java/core/trunk/modules/distribution/pom.xml)找到。这定义了需要在最终分发中包含的 Axis2 项目中创建的所有工件。所有工件都共享相同的`groupId`元素以及`distribution`模块的`version`元素。这在大多数多模块 Maven 项目中是一个常见的场景。大多数模块（如果不是所有模块）都共享相同的`groupId`和`version`元素：

```java
<dependencies>
  <dependency>
    <groupId>org.apache.axis2</groupId>
    <artifactId>axis2-java2wsdl</artifactId>
    <version>${project.version}</version>
  </dependency>
  <dependency>
    <groupId>org.apache.axis2</groupId>
    <artifactId>axis2-kernel</artifactId>
    <version>${project.version}</version>
   </dependency>
   <dependency>
    <groupId>org.apache.axis2</groupId>
    <artifactId>axis2-adb</artifactId>
    <version>${project.version}</version>
   </dependency>
</dependencies>
```

在前面的配置中，Axis2 不是重复`version`元素，而是使用项目属性`${project.version}`。当 Maven 发现这个项目属性时，它会从项目 POM 的`version`元素中读取值。如果项目 POM 文件没有`version`元素，那么 Maven 将尝试从直接父 POM 文件中读取它。这里的优点是，当有一天你升级项目版本时，你只需要升级`distribution` POM 文件的`version`元素（或其父元素）。

前面的配置并不完美；它可以进一步改进如下：

```java
<dependencies>
  <dependency>
    <groupId>${project.groupId}</groupId>
    <artifactId>axis2-java2wsdl</artifactId>
    <version>${project.version}</version>
  </dependency>
  <dependency>
    <groupId>${project.groupId}</groupId>

    <artifactId>axis2-kernel</artifactId>
    <version>${project.version}</version>
   </dependency>
   <dependency>
     <groupId>${project.groupId}</groupId>
     <artifactId>axis2-adb</artifactId>
     <version>${project.version}</version>
   </dependency>
</dependencies>
```

在这里，我们还用项目属性`${project.groupid}`替换了所有依赖项中硬编码的`groupId`元素值。当 Maven 发现这个项目属性时，它会从项目 POM 的`groupId`元素中读取值。如果项目 POM 文件没有`groupId`元素，那么 Maven 将尝试从直接父 POM 文件中读取它。

这里列出了 Maven 的一些内置属性和项目属性：

+   `project.version`：这指的是项目 POM 文件中`version`元素的值

+   `project.groupId`：这指的是项目 POM 文件中`groupId`元素的值

+   `project.artifactId`：这指的是项目 POM 文件中`artifactId`元素的值

+   `project.name`：这指的是项目 POM 文件中`name`元素的值

+   `project.description`：这指的是项目 POM 文件中`description`元素的值

+   `project.basedir`：这指的是项目基本目录的路径

    以下是一个示例，展示了这个项目属性的用法。在这里，我们有一个系统依赖，需要从文件`system`路径中引用：

    ```java
    <dependency>
      <groupId>org.apache.axis2.wso2</groupId>
      <artifactId>axis2</artifactId>
      <version>1.6.0.wso2v2</version>
      <scope>system</scope>
      <systemPath>${project.basedir}/
                lib/axis2-1.6.jar</systemPath> 
    </dependency>
    ```

除了项目属性之外，你还可以从`USER_HOME/.m2/settings.xml`文件中读取属性。例如，如果你想读取本地 Maven 仓库的路径，你可以使用属性`${settings.localRepository}`。同样，使用相同的模式，你可以读取`settings.xml`文件中定义的任何配置元素。

在系统定义的环境变量可以使用应用程序 POM 文件中的 `env` 前缀读取。`${env.M2_HOME}` 属性将返回 Maven 主目录的路径，而 `${env.java_home}` 返回 Java 主目录的路径。这些属性在特定的 Maven 插件中非常有用。

Maven 还允许你定义自己的自定义属性集。自定义属性主要用于定义依赖项版本。

你不应该将自定义属性散布在各个地方。在多模块 Maven 项目中定义它们的理想位置是父 POM 文件，然后所有其他子模块将继承这些属性。

如果你查看 WSO2 Carbon 项目的父 POM 文件，你会找到一个大量自定义属性的定义（[`svn.wso2.org/repos/wso2/carbon/platform/branches/turing/parent/pom.xml`](https://svn.wso2.org/repos/wso2/carbon/platform/branches/turing/parent/pom.xml)）。以下代码块包含了一些这些自定义属性：

```java
<properties>
  <rampart.version>1.6.1-wso2v10</rampart.version>
  <rampart.mar.version>1.6.1-wso2v10</rampart.mar.version>
  <rampart.osgi.version>1.6.1.wso2v10</rampart.osgi.version>
</properties>
```

当你向 Rampart jar 添加依赖项时，你不需要在那里指定版本。只需通过 `${rampart.version}` 属性名称引用它。此外，请记住，所有自定义定义的属性都是继承的，并且可以在任何子 POM 文件中覆盖：

```java
<dependency>
  <groupId>org.apache.rampart.wso2</groupId>
  <artifactId>rampart-core</artifactId>
  <version>${rampart.version}</version>
</dependency>
```

# 避免重复的 groupIds 和版本，并从父 POM 继承

在一个多模块 Maven 项目中，大多数模块（如果不是所有模块）共享相同的 `groupId` 和 `version` 元素。在这种情况下，你可以避免在你的应用程序 POM 文件中添加 `version` 和 `groupId` 元素，因为这些将自动从相应的父 POM 继承。

如果你查看 `axis2-kernel`（它是 Apache Axis2 项目的模块），你会发现没有定义 `groupId` 或 `version` 元素：（[`svn.apache.org/repos/asf/axis/axis2/java/core/trunk/modules/kernel/pom.xml`](http://svn.apache.org/repos/asf/axis/axis2/java/core/trunk/modules/kernel/pom.xml)）。Maven 从父 POM 文件中读取它们：

```java
<project>
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.apache.axis2</groupId>
        <artifactId>axis2-parent</artifactId>
        <version>1.7.0-SNAPSHOT</version>
        <relativePath>../parent/pom.xml</relativePath>
    </parent>

    <artifactId>axis2-kernel</artifactId>
    <name>Apache Axis2 - Kernel</name>

</project>
```

# 遵循命名约定

当定义你的 Maven 项目的坐标时，你必须始终遵循命名约定。

`groupId` 元素的值应该遵循你在 Java 包名称中使用的相同命名约定。它必须是一个域名（域名的反转）——你拥有它——或者至少你的项目是在其下开发的。

以下列表涵盖了部分 `groupId` 命名约定：

+   `groupId` 元素的名称必须是小写。

+   使用可以唯一标识你的项目的域名反转。这也有助于避免不同项目产生的工件之间的冲突。

+   避免使用数字或特殊字符（例如，`org.wso2.carbon.identity-core`）。

+   不要尝试通过驼峰式命名将两个单词组合成一个单词（例如，`org.wso2.carbon.identityCore`）。

+   确保同一公司内不同团队开发的子项目最终都继承自相同的`groupId`，并扩展父`groupId`的名称，而不是定义自己的。

让我们通过一些例子来了解一下。你会注意到，所有在**Apache 软件基金会**（**ASF**）下开发的开源项目都使用相同的父`groupId`（`org.apache`）并定义自己的`groupId`，该`groupId`从父`groupId`扩展而来：

+   **Apache Axis2 项目**：`org.apache.axis2`，继承自父`groupId` `org.apache`

+   **Apache Synapse 项目**：`org.apache.synapse`，继承自父`groupId` `org.apache`

+   **Apache ServiceMix 项目**：`org.apache.servicemix`，继承自父`groupId` `org.apache`

+   **WSO2 Carbon 项目**：`org.wso2.carbon`

除了`groupId`之外，在定义`artifactIds`时也应遵循命名规范。

下面的列表列出了一些`artifactId`的命名规范：

+   `artifactId`的名称必须全部小写。

+   避免在`artifactId`内部重复`groupId`的值。如果你发现需要在`artifactId`元素的开头使用`groupId`元素并在末尾添加一些内容，那么你需要重新审视你的项目结构。你可能需要添加更多的模块组。

+   避免使用特殊字符（例如，#、$、&、%）。

+   不要尝试通过驼峰命名法将两个单词组合成一个单词（例如，identityCore）。

以下`version`的命名规范同样重要。给定 Maven 艺术品的版本可以分成四个部分：

```java
<Major version>.<Minor version>.<Incremental version>-<Build number or the qualifier>
```

主版本号反映了新主要功能的引入。给定艺术品的版本号的变化也可能意味着新更改不一定与之前发布的艺术品向后兼容。次版本号以向后兼容的方式反映了在之前发布的版本中引入的新功能。增量版本反映了修复错误的版本。构建号可以是源代码仓库的修订号。

这种版本规范不仅适用于 Maven 艺术品。苹果在 2014 年 9 月发布了一个 iOS 移动操作系统的重大版本：iOS 8.0.0。发布后不久，他们发现了一个影响蜂窝网络连接和 iPhone 的 TouchID 的严重错误。然后他们发布了 iOS 8.0.1 作为补丁版本来修复这些问题。

让我们通过一些例子来了解一下：

+   **Apache Axis2 1.6.0 版本发布**：[`svn.apache.org/repos/asf/axis/axis2/java/core/tags/v1.6.0/pom.xml`](http://svn.apache.org/repos/asf/axis/axis2/java/core/tags/v1.6.0/pom.xml)。

+   **Apache Axis2 1.6.2 版本发布**：[`svn.apache.org/repos/asf/axis/axis2/java/core/tags/v1.6.2/pom.xml`](http://svn.apache.org/repos/asf/axis/axis2/java/core/tags/v1.6.2/pom.xml)。

+   **Apache Axis2 1.7.0-SNAPSHOT**：[`svn.apache.org/repos/asf/axis/axis2/java/core/trunk/pom.xml`](http://svn.apache.org/repos/asf/axis/axis2/java/core/trunk/pom.xml)。

+   **Apache Synapse 2.1.0-wso2v5**：[`svn.wso2.org/repos/wso2/tags/carbon/3.2.3/dependencies/synapse/2.1.0-wso2v5/pom.xml`](http://svn.wso2.org/repos/wso2/tags/carbon/3.2.3/dependencies/synapse/2.1.0-wso2v5/pom.xml)。在这里，synapse 代码是在 WSO2 源代码库中维护的，而不是在 Apache 下。在这种情况下，我们使用 wso2v5 分类器使其与 Apache Synapse 产生的相同工件区分开来。

# 在编写自己的插件之前，请三思而后行。你可能并不需要它！

Maven 的一切都是关于插件的！几乎你可以找到完成任何所需任务的一个插件。如果你发现需要编写一个插件，花些时间在网上做一些研究，看看是否可以找到类似的东西——可能性非常高。你还可以在[`maven.apache.org/plugins`](http://maven.apache.org/plugins)找到可用的 Maven 插件列表。

# Maven 发布插件

发布一个项目需要执行许多重复的任务。Maven `release`插件的目的是自动化这些任务。发布插件定义了以下八个目标，这些目标分为两个阶段执行——准备发布和执行发布：

+   `release:clean`：这会在发布准备之后进行清理。

+   `release:prepare`：这为软件配置管理（SCM）中的发布做准备。

+   `release:prepare-with-pom`：这为 SCM 中的发布做准备，并通过完全解析依赖项生成发布 POM。

+   `release:rollback`：这会回滚到之前的发布。

+   `release:perform`：这从 SCM 执行发布。

+   `release:stage`：这从 SCM 执行发布到临时文件夹或仓库。

+   `release:branch`：这创建了一个包含所有版本更新的当前项目分支。

+   `release:update-versions`：这更新 POM 中的版本。

准备阶段将使用`release:prepare`目标完成以下任务：

+   确认源代码中的所有更改都已提交。

+   确保没有 SNAPSHOT 依赖项。在项目开发阶段，我们使用 SNAPSHOT 依赖项，但在发布时，所有依赖项都应该更改为已发布的版本。

+   项目 POM 文件的版本将从 SNAPSHOT 更改为具体的版本号。

+   项目 POM 中的 SCM 信息将更改，以包括标记的最终目的地。

+   对修改后的 POM 文件执行所有测试。

+   将修改后的 POM 文件提交到 SCM，并使用版本名称标记代码。

+   将主分支中的 POM 文件版本更改为 SNAPSHOT 版本，然后将修改后的 POM 文件提交到主分支。

最后，将使用`release:perform`目标执行发布。这将从 SCM 中的发布标签检出代码，并运行一系列预定义的目标：`site`和`deploy-site`。

`maven-release-plugin` 在父 POM 中未定义，应在您的项目 POM 文件中显式定义。`releaseProfiles` 配置元素定义了要发布的配置文件，而 `goals` 配置元素定义了在 `release:perform` 期间要执行的插件目标。在以下配置中，`maven-deploy-plugin` 的 `deploy` 目标和 `maven-assembly-plugin` 的 `single` 目标将被执行：

```java
<plugin>
    <artifactId>maven-release-plugin</artifactId>
    <version>2.5</version>
    <configuration>
      <releaseProfiles>release</releaseProfiles>
      <goals>deploy assembly:single</goals>
    </configuration>
</plugin>
```

### 注意

有关 Maven Release 插件的更多详细信息，请参阅 [`maven.apache.org/maven-release/maven-release-plugin/`](http://maven.apache.org/maven-release/maven-release-plugin/)。

# Maven enforcer 插件

Maven Enforce 插件允许您控制或强制构建环境中的约束。这些可能包括 Maven 版本、Java 版本、操作系统参数，甚至是用户定义的规则。

该插件定义了两个目标：`enforce` 和 `displayInfo`。`enforcer:enforce` 目标将对多模块 Maven 项目的所有模块执行所有定义的规则，而 `enforcer:displayInfo` 将显示项目与标准规则集的合规性细节。

`maven-enforcer-plugin` 在父 POM 中未定义，应在您的项目 POM 文件中显式定义：

```java
<plugins>
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-enforcer-plugin</artifactId>
    <version>1.3.1</version>
    <executions>
      <execution>
        <id>enforce-versions</id>
        <goals>
          <goal>enforce</goal>
        </goals>
        <configuration>
 <rules>
 <requireMavenVersion>
 <version>3.2.1</version>
 </requireMavenVersion>
 <requireJavaVersion>
 <version>1.6</version>
 </requireJavaVersion>
 <requireOS>
 <family>mac</family>
 </requireOS>
 </rules>
        </configuration>
      </execution>
    </executions>
  </plugin>
</plugins>
```

上述插件配置强制 Maven 版本为 3.2.1，Java 版本为 1.6，操作系统为 Mac 系列。

Apache Axis2 项目使用 `enforcer` 插件确保没有应用程序 POM 文件定义 Maven 仓库。Axis2 所需的所有工件都应位于 Maven 中央仓库中。以下配置元素是从 [`svn.apache.org/repos/asf/axis/axis2/java/core/trunk/modules/parent/pom.xml`](http://svn.apache.org/repos/asf/axis/axis2/java/core/trunk/modules/parent/pom.xml) 中提取的。在这里，它禁止所有仓库和插件仓库，除了快照仓库：

```java
<plugin>
  <artifactId>maven-enforcer-plugin</artifactId>
  <version>1.1</version>
  <executions>
    <execution>
      <phase>validate</phase>
      <goals>
        <goal>enforce</goal>
      </goals>
      <configuration>
        <rules>
          <requireNoRepositories>
            <banRepositories>true</banRepositories>
            <banPluginRepositories>true</banPluginRepositories>
            <allowSnapshotRepositories>true
                                   </allowSnapshotRepositories>
            <allowSnapshotPluginRepositories>true
                             </allowSnapshotPluginRepositories>
          </requireNoRepositories>
        </rules>
      </configuration>
    </execution>
  </executions>
</plugin>
```

### 注意

除了 `enforcer` 插件附带的标准规则集外，您还可以定义自己的规则。有关如何编写自定义规则的更多详细信息，请参阅 [`maven.apache.org/enforcer/enforcer-api/writing-a-custom-rule.html`](http://maven.apache.org/enforcer/enforcer-api/writing-a-custom-rule.html)。

# 避免使用未指定版本的插件

如果您已将插件与您的应用程序 POM 关联，但没有指定版本，那么 Maven 将下载相应的 `maven-metadata.xml` 文件并将其存储在本地。只有插件的最新发布版本将被下载并用于项目。这可能会轻易地产生不确定性。您的项目可能使用当前版本的插件运行良好，但后来，如果相同的插件有新的发布版本，您的 Maven 项目将自动开始使用最新的版本。这可能会导致不可预测的行为，并导致调试混乱。

### 小贴士

始终建议您在插件配置中指定插件版本。

你可以使用 Maven `enforcer`插件强制执行此规则，如下所示：

```java
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-enforcer-plugin</artifactId>
  <version>1.3.1</version>
  <executions>
    <execution>
      <id>enforce-plugin-versions</id>
      <goals>
        <goal>enforce</goal>
      </goals>
      <configuration>
        <rules>
          <requirePluginVersions>
               <message>………… <message>
            <banLatest>true</banLatest>
            <banRelease>true</banRelease>
            <banSnapshots>true</banSnapshots>
            <phases>clean,deploy,site</phases>
            <additionalPlugins>
 <additionalPlugin>
                  org.apache.maven.plugins:maven-eclipse-plugin
              </additionalPlugin>
 <additionalPlugin>
                  org.apache.maven.plugins:maven-reactor-plugin
              </additionalPlugin>
 </additionalPlugins>
            <unCheckedPluginList>
                  org.apache.maven.plugins:maven-enforcer-plugin,
                  org.apache.maven.plugins:maven-idea-plugin
            </unCheckedPluginList>
          </requirePluginVersions>
        </rules>
      </configuration>
    </execution>
  </executions>
</plugin>
```

以下是对前面代码中定义的每个关键配置元素的说明：

+   `message`：此选项用于定义用户可选的消息，如果规则执行失败。

+   `banLatest`：此选项用于限制任何插件使用"Latest"作为版本。

+   `banRelease`：此选项用于限制任何插件使用"RELEASE"作为版本。

+   `banSnapshots`：此选项用于限制使用 SNAPSHOT 插件。

+   `banTimestamps`：此选项用于限制使用带时间戳版本的 SNAPSHOT 插件。

+   `phases`：这是一个以逗号分隔的阶段列表，用于查找生命周期插件绑定。默认值是"`clean`,`deploy`,`site`"。

+   `additionalPlugins`：这是一个额外的插件列表，用于强制执行版本。这些插件可能没有在应用程序 POM 文件中定义，但仍然被使用，如 help、eclipse 等。插件应以`groupId:artifactId`的形式指定。

+   `unCheckedPluginList`：这是一个以逗号分隔的插件列表，用于跳过版本检查。

### 注意

你可以阅读更多关于`requirePluginVersions`规则的信息，请参阅[`maven.apache.org/enforcer/enforcer-rules/requirePluginVersions.html`](http://maven.apache.org/enforcer/enforcer-rules/requirePluginVersions.html)。

# 描述性父 POM 文件

确保你的项目父 POM 文件描述足够详细，以便列出项目所做的工作、开发者及贡献者、他们的联系方式、项目工件发布的许可证、报告问题的位置等信息。以下是一个描述性良好的 POM 文件示例，可在[`svn.apache.org/repos/asf/axis/axis2/java/core/trunk/modules/parent/pom.xml`](http://svn.apache.org/repos/asf/axis/axis2/java/core/trunk/modules/parent/pom.xml)找到：

```java
<project>
  <name>Apache Axis2 - Parent</name>       
  <inceptionYear>2004</inceptionYear>     
  <description>Axis2 is an effort to re-design and totally re-  
                 implement both Axis/Java……</description>
  <url>http://axis.apache.org/axis2/java/core/</url>     
  <licenses><license>http://www.apache.org/licenses/LICENSE-2.0.html</license></licenses>
  <issueManagement>         
    <system>jira</system>         
    <url>http://issues.apache.org/jira/browse/AXIS2</url>       
  </issueManagement>
  <mailingLists>         
    <mailingList>             
      <name>Axis2 Developer List</name>             
      <subscribe>java-dev-subscribe@axis.apache.org</subscribe>             
      <unsubscribe>java-dev-unsubscribe@
                                  axis.apache.org</unsubscribe>             
      <post>java-dev@axis.apache.org</post>             
      <archive>http://mail-archives.apache.org/mod_mbox/axis-java-dev/</archive>             
      <otherArchives>                    
        <otherArchive>http://markmail.org/search/list:
                         org.apache.ws.axis-dev</otherArchive>             
      </otherArchives>         
    </mailingList>
    <developers>         
      <developer>             
        <name>Sanjiva Weerawarana</name>             
        <id>sanjiva</id>             
        <email>sanjiva AT wso2.com</email>             
        <organization>WSO2</organization>         
      </developer>
    <developers>  
    <contributors>
      <contributor>             
        <name>Dobri Kitipov</name>             
        <email>kdobrik AT gmail.com</email>             
        <organization>Software AG</organization>           
        </contributor>     
    </contributors>
</project>
```

# 文档是你的朋友

如果你是一个优秀的开发者，你应该知道文档的价值。你所写的一切都不应该是晦涩难懂的，或者只有你自己能理解。让它成为 Java、.NET、C++项目，或者 Maven 项目——文档是你的朋友。带有良好文档的代码非常易于阅读。如果你在应用程序 POM 文件中添加的任何配置都不是自我描述的，请确保至少添加一行注释来解释其功能。

下面将跟随一些来自 Apache Axis2 项目的良好示例：

```java
<profile>             
  <id>java16</id>             
  <activation>                 
    <jdk>1.6</jdk>             
  </activation>             
  <!-- JDK 1.6 build still use JAX-WS 2.1 because integrating 
 Java endorsed mechanism with Maven is bit of complex - 
  ->               
  <properties>                 
    <jaxb.api.version>2.1</jaxb.api.version>
    <jaxbri.version>2.1.7</jaxbri.version>                 
    <jaxws.tools.version>2.1.3</jaxws.tools.version>                 
    <jaxws.rt.version>2.1.3</jaxws.rt.version>               
  </properties>         
</profile>
```

-------------------------------------------------------------------------------------------------------

```java
<plugin>                     
  <artifactId>maven-assembly-plugin</artifactId>                       
  <!-- Minimum required version here is 2.2-beta-4 because 
 org.apache:apache:7 uses the runOnlyAtExecutionRoot 
 parameter, which is not supported in earlier 
 versions. --> 
  <version>2.2-beta-5</version>                     
  <configuration>                         
 <!-- Workaround for MASSEMBLY-422 / MASSEMBLY-449 --> 
    <archiverConfig>                               
      <fileMode>420</fileMode><!-- 420(dec)=644(oct) --> 
      <directoryMode>493</directoryMode><!--493(dec)=755(oct)--> 
      <defaultDirectoryMode>493</defaultDirectoryMode>                           
    </archiverConfig>                     
  </configuration>                 
</plugin>
```

-------------------------------------------------------------------------------------------------------

```java
<!-- No chicken and egg problem here because the plugin doesn't expose any extension. We can always use the version from the current build. --> 
<plugin>                     
  <groupId>org.apache.axis2</groupId>                     
  <artifactId>axis2-repo-maven-plugin</artifactId>                        
  <version>${project.version}</version>
</plugin>
```

# 避免覆盖默认的目录结构

Maven 遵循设计哲学 *约定优于配置*。在没有任何配置更改的情况下，Maven 假设源代码的位置是 `${basedir}/src/main/java`，测试的位置是 `${basedir}/src/test/java`，资源位于 `${basedir}/src/main/resources`。在构建成功后，Maven 知道编译后的类放在哪里（`${basedir}/target/classes`）以及最终工件应该复制到哪（`${basedir}/target/`）。虽然可以更改这种目录结构，但建议不要这样做。为什么？

保持默认结构可以提高项目的可读性。即使是一个新手开发者，如果他熟悉 Maven，也知道该往哪里找。此外，如果你已经将插件和其他 Maven 扩展与你的项目关联，那么如果你没有更改默认的 Maven 目录结构，你将能够以最小的更改使用它们。大多数这些插件和其他扩展默认假设 Maven 约定。

# 在开发过程中使用 SNAPSHOT 版本控制

如果你的项目工件仍在开发中并且定期部署到 Maven 快照仓库，你应该使用 `SNAPSHOT` 限定符。如果即将发布的版本是 1.7.0，那么在开发期间你应该使用版本 `1.7.0-SNAPSHOT`。Maven 对 `SNAPSHOT` 版本有特殊处理。如果你尝试将 `1.7.0-SNAPSHOT` 部署到仓库，Maven 会首先将 `SNAPSHOT` 限定符扩展为 UTC（协调世界时）的日期和时间值。如果部署时的日期/时间是 2014 年 11 月 10 日上午 10:30，那么 `SNAPSHOT` 限定符将被替换为 20141110-103005-1，并且工件将以版本 `1.7.0-20141110-103005-1` 部署。

# 移除未使用的依赖项

总是确保你维护一个干净的应用程序 POM 文件。你不应该定义或使用任何未声明的依赖项。Maven 的 `dependency` 插件帮助你识别此类差异。

`maven-dependency-plugin` 在父 POM 中未定义，应在你的项目 POM 文件中显式定义：

```java
<plugin>
  <artifactId>maven-dependency-plugin</artifactId>
  <version>2.0</version>
</plugin>
```

一旦将前面的配置添加到你的应用程序 POM 文件中，你需要运行 `dependency` 插件的 `analyze` 目标来针对你的 Maven 项目：

```java
$ mvn dependency:analyze

```

在这里，你可以看到一份样本输出，它抱怨一个未使用的声明依赖项：

```java
[WARNING] Unused declared dependencies found:
[WARNING] org.apache.axis2:axis2-kernel:jar:1.6.2:compile

```

### 注意

关于 Maven 依赖插件的更多详细信息可在 [`maven.apache.org/plugins/maven-dependency-plugin/`](http://maven.apache.org/plugins/maven-dependency-plugin/) 找到。

# 避免在应用程序 POM 文件中保留凭证

在 Maven 构建过程中，你需要连接到防火墙外的外部仓库。在一个高度安全的环境中，任何出站连接都必须通过内部代理服务器。以下 `MAVEN_HOME/conf/settings.xml` 中的配置显示了如何通过安全的代理服务器连接到外部仓库：

```java
<proxy>
  <id>internal_proxy</id>
  <active>true</active>
  <protocol>http</protocol>
  <username>proxyuser</username>
  <password>proxypass</password>
  <host>proxy.host.net</host>
  <port>80</port>
  <nonProxyHosts>local.net|some.host.com</nonProxyHosts>
</proxy>
```

此外，Maven 仓库可以受到合法访问的保护。如果某个仓库通过 HTTP 基本身份验证进行保护，那么相应的凭据应该在`MAVEN_HOME/conf/settings.xml`的`server`元素下定义如下：

```java
<server>
  <id>central</id>
  <username>my_username</username> 
  <password>my_password</password>
</server>
```

在配置文件中明文存储机密数据是一种安全威胁，必须避免。Maven 提供了一种在`settings.xml`中加密配置数据的方法。

首先，我们需要创建一个主加密密钥，如下所示：

```java
$ mvn -emp mymasterpassword
{lJ1MrCQRnngHIpSadxoyEKyt2zIGbm3Yl0ClKdTtRR6TleNaEfGOEoJaxNcdMr+G}

```

使用上述命令的输出，我们需要在`USER_HOME/.m2/`下创建一个名为`settings-security.xml`的文件，并将加密的主密码添加到其中，如下所示：

```java
<settingsSecurity>   
<master>
{lJ1MrCQRnngHIpSadxoyEKyt2zIGbm3Yl0ClKdTtRR6TleNaEfGOEoJaxNcdMr+G}
</master>
</settingsSecurity>
```

一旦正确配置了主密码，我们就可以开始加密`settings.xml`中的其余机密数据。让我们看看如何加密服务器密码。首先，我们需要使用以下命令生成明文密码的加密密码。注意，之前我们使用了 emp（加密主密码），而现在我们使用 ep（加密密码）：

```java
$  mvn -ep my_password
{PbYw8YaLb3cHA34/5EdHzoUsmmw/u/nWOwb9e+x6Hbs=}

```

复制加密密码的值，并将其替换为`settings.xml`中相应的值：

```java
<server>
  <id>central</id>
  <username>my_username</username> 
  <password>
    {PbYw8YaLb3cHA34/5EdHzoUsmmw/u/nWOwb9e+x6Hbs=}
  </password>
</server>
```

# 避免使用已弃用的引用

从 Maven 3.0 开始，所有以`pom.*`开头的属性都已弃用。避免使用任何已弃用的 Maven 属性，如果你已经使用了它们，确保迁移到等效的属性。

# 避免重复 - 使用原型

当我们创建一个 Java 项目时，需要根据项目的类型以不同的方式对其进行结构化。如果它是一个 Java EE 网络应用程序，那么我们需要有一个 WEB-INF 目录和一个`web.xml`文件。如果它是一个 Maven 插件项目，我们需要有一个 Mojo 类，该类从`org.apache.maven.plugin.AbstractMojo`扩展。由于每种类型的项目都有自己的预定义结构，为什么每个人都必须一次又一次地构建相同结构呢？为什么我们不从一个模板开始呢？每个项目都可以有自己的模板，开发者可以根据他们的需求扩展模板。Maven 原型解决了这个问题。每个原型都是一个项目模板。

我们在第三章中详细讨论了 Maven 原型。

# 避免使用 maven.test.skip

你可能可以管理一个非常小的项目，这个项目没有太多的发展，而不需要单元测试。但是，任何大规模的项目都不能没有单元测试。单元测试提供了第一层保证，确保你不会因为引入新的代码更改而破坏任何现有功能。在理想情况下，你不应该在没有使用单元测试构建完整项目的情况下将任何代码提交到源代码库。

Maven 使用`surefire`插件来运行测试，并且作为不良做法，开发者习惯于通过将`maven.test.skip`属性设置为`true`来跳过单元测试的执行，如下所示：

```java
$ mvn clean install –Dmaven.test.skip=true

```

这可能导致项目后期出现严重后果，您必须确保所有开发人员在构建过程中不要跳过测试。

使用 Maven `enforcer`插件的`requireProperty`规则，您可以禁止开发人员使用`maven.test.skip`属性。

以下展示了您需要添加到应用程序 POM 文件中的`enforcer`插件配置：

```java
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-enforcer-plugin</artifactId>
  <version>1.3.1</version>
  <executions>
    <execution>
      <id>enforce-property</id>
      <goals>
        <goal>enforce</goal>
      </goals>
      <configuration>
        <rules>
          <requireProperty>
            <property>maven.test.skip</property>
            <message>maven.test.skip must be specified</message>
            <regex>false</regex>
            <regexMessage>You cannot skip tests</regexMessage>
          </requireProperty>
        </rules>
        <fail>true</fail>
      </configuration>
    </execution>
  </executions>
</plugin>
```

现在，如果您对项目运行`mvn clean install`，您将看到以下错误信息：

```java
maven.test.skip must be specified

```

这意味着您需要在每次运行`mvn clean install`时指定`Dmaven.test.skip=false`：

```java
$ mvn clean install –Dmaven.test.skip=false

```

但如果您设置了`–Dmaven.test.skip=true`，那么您将看到以下错误：

```java
You cannot skip tests

```

尽管如此，每次运行构建时都输入`–Dmaven.test.skip=false`可能会让您感到有些烦恼。为了避免这种情况，在您的应用程序 POM 文件中添加属性`maven.test.skip`并将其值设置为`false`：

```java
<project>

  <properties>
    <maven.test.skip>false</maven.test.skip>
  </properties>

</project>
```

### 注意

关于`requireProperty`规则的更多详细信息，请参阅[`maven.apache.org/enforcer/enforcer-rules/requireProperty.html`](http://maven.apache.org/enforcer/enforcer-rules/requireProperty.html)。

# 摘要

在本章中，我们探讨了并强调了在大型开发项目中遵循的一些最佳实践。本书前几章详细讨论了这里强调的大部分内容。始终推荐遵循最佳实践，因为它将极大地提高开发人员的工作效率，并减少任何维护噩梦。

总体而言，本书涵盖了 Apache Maven 3 及其核心概念，包括 Maven 原型、插件、构建和生命周期，并通过示例进行了说明。
