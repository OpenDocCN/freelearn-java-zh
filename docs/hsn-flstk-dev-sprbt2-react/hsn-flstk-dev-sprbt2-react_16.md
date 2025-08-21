# 第十六章：评估

# 第一章

答案 1：Spring Boot 是基于 Java 的 Web 应用程序框架，基于 Spring。使用 Spring Boot，您可以开发独立的 Web 应用程序，带有嵌入式应用服务器。

答案 2：Eclipse 是开源的**集成开发环境**（**IDE**），主要用于 Java 编程，但也支持多种其他编程语言。

答案 3：Maven 是开源软件项目管理工具。Maven 可以管理软件开发项目中的构建、文档、测试等。

答案 4：开始一个新的 Spring Boot 项目最简单的方法是在 Spring Initializr 网页上创建它。这将为您的项目创建一个包含所需模块的框架。

答案 5：如果您使用 Eclipse IDE，只需激活主类并点击运行按钮。您也可以使用 Maven 命令**`mvn spring-boot:run`**来运行应用程序。

答案 6：Spring Boot starter 包为您提供日志记录功能。您可以在`application.properties`设置文件中定义日志记录级别。

答案 7：在运行应用程序后，您可以在 Eclipse IDE 控制台中看到错误和日志消息。

# 第二章

答案 1：ORM 是一种技术，允许您使用面向对象的编程范式从数据库中提取和操作数据。JPA 为 Java 开发人员提供了对象关系映射。Hibernate 是基于 Java 的 JPA 实现。

答案 2：实体类只是一个标准的带有`@Entity`注解的 Java 类。在类内部，您必须实现构造函数、字段、getter 和 setter。将作为唯一 id 的字段用`@Id`注解标注。

答案 3：您必须创建一个新的接口，该接口扩展 Spring Data 的`CrudRepository`接口。在类型参数中，您定义实体和`id`字段的类型，例如，`<Car, Long>`。

答案 4：`CrudRepository`为您的实体提供了所有 CRUD 操作。您可以使用`CrudRepository`创建、读取、更新和删除实体。

答案 5：您必须创建实体类并使用`@OneToMany`和`@ManyToOne`注解链接实体。

答案 6：您可以在主应用程序类中使用`CommandLineRunner`添加演示数据。

答案 7：在您的`application.properties`文件中定义 H2 控制台的端点并启用它。然后您可以通过浏览器访问定义的端点来访问 H2 控制台。

答案 8：您必须在`pom.xml`文件中添加 MariaDB 依赖，并在`application.properties`文件中定义数据库连接设置。如果您使用了 H2 数据库依赖，还需要从`pom.xml`文件中删除它。

# 第三章

答案 1：REST 是一种用于创建 Web 服务的架构风格，并定义了一组约束。

答案 2：使用 Spring Data REST starter 包是使用 Spring Boot 创建 RESTful web 服务的最简单方法。默认情况下，Spring Data REST 会找到所有公共存储库，并为您的实体自动创建 RESTful Web 服务。

答案 3：通过向实体的端点发送`GET`请求。例如，如果您有一个名为`Car`的实体类，Spring Data REST 将创建一个名为`/cars`的端点，用于获取所有汽车。

答案 4：通过向单个实体项目的端点发送`DELETE`请求。例如，`/cars/1`删除 id 为`1`的汽车。

答案 5：通过向实体的端点发送`POST`请求。标头必须包含`Content-Type`字段，值为`application/json`，新项目将嵌入在请求体中。

答案 6：通过向实体的端点发送`PATCH`请求。标头必须包含`Content-Type`字段，值为`application/json`，更新的项目将嵌入在请求体中。

**答案 7**：您必须使用`@RepositoryRestResource`注解注释您的存储库。使用`@Param`注解注释查询参数。

# 第四章

**答案 1**：Spring Security 为基于 Java 的 Web 应用程序提供安全服务。

**答案 2**：您必须将 Spring Security 启动包依赖添加到您的`pom.xml`文件中。您可以通过创建安全配置类来配置 Spring Security。

**答案 3**：**JWT**（**JSON Web Token**）是在现代 Web 应用程序中实现身份验证的一种紧凑方式。令牌的大小很小，因此可以在 URL 中、`POST`参数中或标头中发送。

**答案 4**：您可以使用 Java JWT 库，这是 Java 的 JSON Web Token 库。认证服务类添加和读取令牌。过滤器类处理登录和认证过程。

**答案 5**：您必须将 Spring Boot 测试启动器包添加到您的`pom.xml`文件中。Spring Boot 测试启动器包提供了许多很好的测试工具，例如 JUnit，AssertJ 和 Mockito。使用 JUnit 时，基本测试类使用`@SpringBootTest`注解，并且测试方法应该以`@Test`注解开头。

**答案 6**：可以通过在 Eclipse IDE 中运行测试类（Run | JUnit test）轻松执行测试用例。测试结果可以在 JUnit 选项卡中看到。

# 第五章

**答案 1**：Node.js 是一个基于 JavaScript 的开源服务器端环境。Npm 是 JavaScript 的包管理器。

**答案 2**：您可以从[`nodejs.org/en/download`](https://nodejs.org/en/download)找到多个操作系统的安装包和说明。

**答案 3**：**Visual Studio Code**（**VSCode**）是一个面向多种编程语言的开源代码编辑器。

**答案 4**：您可以从[`code.visualstudio.com`](https://code.visualstudio.com)找到多个操作系统的安装包和说明。

**答案 5**：您必须使用 npm 全局安装`create-react-app`。然后使用以下命令创建一个应用程序**`create-react-app projectname`**。

**答案 6**：您可以使用以下命令运行应用程序`npm start`或**`yarn start`**。

**答案 7**：您可以通过修改`App.js`文件开始，当您保存修改时，可以立即在 Web 浏览器中看到更改。

# 第六章

**答案 1**：组件是 React 应用程序的基本构建块。React 组件可以使用 JavaScript 函数或 ES6 类创建。

**答案 2**：props 和 state 是呈现组件的输入数据。它们是 JavaScript 对象，当 props 或 state 发生变化时，组件会重新呈现。

**答案 3**：数据流从父组件到子组件。

**答案 4**：只有 props 的组件称为无状态组件。既有 props 又有状态的组件称为有状态组件。

**答案 5**：JSX 是 JavaScript 的语法扩展，建议与 React 一起使用。

**答案 6**：组件生命周期方法在组件生命周期的特定阶段执行。

**答案 7**：处理 DOM 元素事件类似。React 中的区别在于事件命名使用驼峰命名约定，例如`onClick`或`onSubmit`。

**答案 8**：通常情况下，我们希望在表单提交后调用具有对表单数据的访问权限的 JavaScript 函数。因此，我们必须使用`preventDefault()`函数禁用默认行为。您可以使用输入字段的`onChange`事件处理程序将输入字段的值保存到状态中。

# 第七章

**答案 1**：Promise 是表示异步操作结果的对象。使用 Promise 在进行异步调用时简化了代码。

答案 2：Fetch API 提供了`fetch()`方法，您可以使用它来使用 JavaScript 进行异步调用。

答案 3：REST API 的`fetch()`调用建议在`componentDidMount()`生命周期方法中执行，该方法在组件挂载后被调用。

答案 4：您可以使用`fetch()`方法的 promises 访问响应数据。响应中的数据保存到状态中，当状态改变时组件重新渲染。

# 第八章

答案 1：您可以从多个来源找到 React 组件，例如，[`js.coach/`](https://js.coach/)或[`github.com/brillout/awesome-react-components`](https://github.com/brillout/awesome-react-components)。

答案 2：您可以使用 npm 或 yarn 软件包管理器安装 React 组件。使用 npm 时，我们使用以下命令**`npm install <componentname>`**。

答案 3：您必须安装 React Table 组件。安装后，您可以在`render()`方法中使用`ReactTable`组件。您必须使用`ReactTable`的 props 定义数据和列。数据可以是对象或数组。

答案 4：在 React 应用中创建模态表单的一种方法是使用 React Skylight 组件（[`marcio.github.io/react-skylight/`](https://marcio.github.io/react-skylight/)）。

答案 5：您必须使用以下命令`npm install @material-ui/core`安装 Material-UI 组件库。安装库后，您可以开始使用组件。不同组件的文档可以在[`material-ui.com`](https://material-ui.com)找到。

答案 6：可以使用 React Router 组件（[`github.com/ReactTraining/react-router`](https://github.com/ReactTraining/react-router)）来实现路由。

# 第九章

答案 1：通过模拟，与客户讨论需求要比在开始编写任何实际代码之前更容易。与真实前端源代码相比，对模拟的修改非常容易和快速。

答案 2：有很多适合的应用程序可以轻松进行模拟。您也可以使用纸和铅笔来创建模拟。

答案 3：您可以修改安全配置类以允许在没有身份验证的情况下访问所有端点。

# 第十章

答案 1：首先，您必须使用`fetch()`方法调用 REST API。然后，您可以使用`fetch()`方法的 promises 访问响应数据。响应中的数据保存到状态中，当状态改变时组件重新渲染。

答案 2：您必须使用`fetch()`方法发送`DELETE`方法请求。调用的端点是您想要删除的项目的链接。

答案 3：您必须使用`fetch()`方法发送`POST`方法请求到实体端点。添加的项目应嵌入在请求体中，并且您必须使用`Content-Type`头和`application/json`值。

答案 4：您必须使用`fetch()`方法发送`PATCH`方法请求。调用的端点是您想要更新的项目的链接。更新的项目应嵌入在请求体中，并且您必须使用`Content-Type`头和`application/json`值。

答案 5：您可以使用一些第三方 React 组件来显示类似 React Toastify 的提示消息。

答案 6：您可以使用一些第三方 React 组件将数据导出到 CSV 文件，例如 React CSV。

# 第十一章

答案 1：Material-UI 是用于 React 的组件库，它实现了 Google 的 Material Design。

答案 2：首先，您必须使用以下命令安装 Material-UI 库`npm install @material-ui/core`。然后，您可以开始使用库中的组件。不同组件的文档可以在[`material-ui.com/`](https://material-ui.com/)找到。

答案 3：您可以使用以下 npm 命令删除组件**`npm remove <componentname>`**。

# 第十二章

答案 1：Jest 是 Facebook 开发的 JavaScript 测试库。

答案 2：使用`.test.js`扩展名创建一个测试文件。在文件中实现您的测试用例，您可以使用以下命令运行测试**`npm test`**。

答案 3：对于快照测试，您必须安装`react-test-render`包，并将`renderer`导入到您的测试文件中。在文件中实现您的快照测试用例，您可以使用以下命令运行测试**`npm test`**。

答案 4：Enzyme 是一个用于测试 React 组件输出的 JavaScript 库。

答案 5：使用以下 npm 命令 **`npm install enzyme enzyme-adapter-react-16 --save-dev`**。

答案 6：您必须将`Enzyme`和`Adapter`组件导入到您的测试文件中。然后，您可以创建测试用例来呈现一个组件。使用 Enzyme，您可以使用 Jest 进行断言。

答案 7：Enzyme 提供了`simulate`方法，可用于测试事件。

# 第十三章

答案 1：您必须创建一个新的组件，用于呈现用户名和密码的输入字段。该组件还包含一个按钮，当按钮被按下时调用`/login`端点。

答案 2：登录组件的调用是使用`POST`方法进行的，并且用户对象嵌入在主体中。如果身份验证成功，后端将在授权标头中发送令牌回来。

答案 3：可以使用`sessionStorage.setItem()`方法将令牌保存到会话存储中。

答案 4：令牌必须包含在请求的`Authorization`标头中。

# 第十四章

答案 1：您可以使用以下 Maven 命令创建可执行的 JAR 文件 **`mvn clean install`**。

答案 2：部署 Spring Boot 应用程序的最简单方法是将应用程序源代码推送到 GitHub，并从 Heroku 使用 GitHub 链接。

答案 3：将 React 应用程序部署到 Heroku 的最简单方法是使用 Heroku Buildpack for create-react-app ([`github.com/mars/create-react-app-buildpack`](https://github.com/mars/create-react-app-buildpack))。

答案 4：Docker 是一个容器平台，使软件开发、部署和交付更容易。容器是轻量级和可执行的软件包，包括运行软件所需的一切。

答案 5：Spring Boot 应用程序只是一个可执行的 JAR 文件，可以使用 Java 执行。因此，您可以为 Spring Boot 应用程序创建 Docker 容器，方式与为任何 Java JAR 应用程序创建方式相似。

答案 6：您可以使用以下 Docker 命令从 Docker Hub 拉取最新的 MariaDB 容器**`docker pull mariadb:latest`**。

# 第十五章

答案 1：这使得代码更易读和更易于维护。它还使团队合作更容易，因为每个人在编码中都使用相同的结构。

答案 2：这使得代码更易读和更易于维护。代码的测试更容易。

答案 3：这使得代码更易读和更易于维护。它还使团队合作更容易，因为每个人在编码中都使用相同的命名约定。
