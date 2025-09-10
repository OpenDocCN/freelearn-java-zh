# 附录 B. 部署 Dropwizard 应用程序

在整本书中，我们展示了并使用了 Dropwizard 项目最重要的部分。我们的应用程序现在已准备好，可以投入生产。它已准备好部署到服务器，从那里可以通过互联网被每个人访问。

# 准备应用程序以进行部署

如您所猜，我们的应用程序没有很多依赖项。只需检查您的`pom.xml`文件，并查找声明`maven-compiler-plugin`的部分。

```java
<project  
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.dwbook.phonebook</groupId>
  <artifactId>dwbook-phonebook</artifactId>
  <packaging>jar</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>dwbook-phonebook</name>
  <url>http://maven.apache.org</url>
  <!-- Maven Repositories -->
  <repositories>
    <repository>
      <id>sonatype-nexus-snapshots</id>
      <name>Sonatype Nexus Snapshots</name>
<url>http://oss.sonatype.org/content/repositories/snapshots</url>
    </repository>
  </repositories>
  <!-- Dependencies -->
  <dependencies>
    <dependency>
      <groupId>io.dropwizard</groupId>
      <artifactId>dropwizard-core</artifactId>
      <version>0.7.0-SNAPSHOT</version>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.6</version>
    </dependency>
    <dependency>
      <groupId>io.dropwizard</groupId>
      <artifactId>dropwizard-jdbi</artifactId>
      <version>0.7.0-SNAPSHOT</version>
    </dependency>
    <dependency>
      <groupId>io.dropwizard</groupId>
      <artifactId>dropwizard-client</artifactId>
      <version>0.7.0-SNAPSHOT</version>
    </dependency>
    <dependency>
      <groupId>io.dropwizard</groupId>
      <artifactId>dropwizard-auth</artifactId>
      <version>0.7.0-SNAPSHOT</version>
    </dependency>
    <dependency>
      <groupId>io.dropwizard</groupId>
      <artifactId>dropwizard-views-mustache</artifactId>
      <version>0.7.0-SNAPSHOT</version>
    </dependency>
    <dependency>
      <groupId>io.dropwizard</groupId>
      <artifactId>dropwizard-assets</artifactId>
      <version>0.7.0-SNAPSHOT</version>
    </dependency>
    <dependency>
      <groupId>io.dropwizard</groupId>
      <artifactId>dropwizard-testing</artifactId>
      <version>0.7.0-SNAPSHOT</version>
    </dependency>
    <dependency>
      <groupId>org.hamcrest</groupId>
      <artifactId>hamcrest-all</artifactId>
      <version>1.3</version>
    </dependency>
  </dependencies>
  <!-- Build Configuration -->
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.1</version>
        <configuration>
          <source>1.7</source>
          <target>1.7</target>
          <encoding>UTF-8</encoding>
        </configuration>
      </plugin>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <version>1.6</version>
        <configuration>
          <filters>
            <filter>
              <artifact>*:*</artifact>
                <excludes>
                  <exclude>META-INF/*.SF</exclude>
                  <exclude>META-INF/*.DSA</exclude>
                  <exclude>META-INF/*.RSA</exclude>
                </excludes>
            </filter>
          </filters>
        </configuration>
        <executions>
          <execution>
            <phase>package</phase>
              <goals>
                <goal>shade</goal>
              </goals>
            <configuration>
              <transformers>
                <transformer
implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                  <mainClass>com.dwbook.phonebook.App</mainClass>
                </transformer>
              </transformers>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</project>
```

服务器上应该只存在 Java 运行时环境，其版本等于或高于构建插件配置部分中`<target>`元素指定的版本。

## 如何做到这一点…

一旦我们确认了我们的依赖项（Java 版本）满足要求，我们就可以通过 FTP 上传 JAR 文件，并以与我们之前相同的方式运行应用程序：

```java
$ java -jar <applicationFilename.jar> server <configFileName.yaml>

```

## 它是如何工作的…

在我们的`pom.xml`文件中，我们声明了所有必需的 Maven 参数，包括`maven-shade-plugin`，这使得我们可以构建一个包含我们应用程序使用的所有第三方模块和库的单个 JAR 文件。只需记住，也要将您的配置文件上传到服务器，或者创建一个新的配置文件，可能包含不同的设置，例如数据库连接详情。

## 还有更多…

有许多很好的理由，您可能希望将应用程序的默认端口从 8080 更改为其他端口。

这可以通过对您的配置文件`config.yaml`进行少量添加来实现。然而，为了使这些设置生效，我们需要在构建配置中添加 ServiceResourceTransformer，通过在 pom.xml 文件中的`<transformers>`部分添加以下条目来实现：`<transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>`。

添加`server`部分，并按照以下代码配置其属性：

```java
server:
    applicationConnectors:
        - type: http
          # The port the application will listen on
          port: 8181
    adminConnectors:
        - type: http
          # The admin port
          port: 8282
```

### 多个配置文件

一种良好的实践是为您的应用程序的每个环境维护不同的配置文件集（YAML）。例如，您可能会为测试和生产环境使用不同的数据库，并且最好将连接信息保存在不同的文件中。此外，您可能希望在开发或测试环境中的日志级别比生产环境更详细。根据您应用程序的性质和复杂性，肯定会有许多其他原因让您和您的应用程序从中受益。幸运的是，Dropwizard 提供了许多可以调整以匹配您应用程序需求的设置。
