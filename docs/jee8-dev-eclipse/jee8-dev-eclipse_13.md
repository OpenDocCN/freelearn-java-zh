# 第十四章：保护 JEE 应用程序

在上一章中，我们学习了如何在云中部署 JEE 应用程序。在本章中，我们将学习如何保护 JEE 应用程序——特别是如何执行身份验证和授权。

我们将涵盖以下主题：

+   使用部署描述符保护 JEE 网络应用程序

+   使用注解保护 JEE 网络应用程序

+   保护网络服务

+   JEE 8 中的安全增强

# JEE 中的身份验证和授权

身份验证是验证用户是否是他或她所声称的过程。这通常是通过要求用户提供用户名和密码来完成的。验证客户端身份的另一种方式是要求客户端证书。在本章中，我们将仅查看密码身份验证。

授权是确定用户是否被允许在应用程序中执行某些操作的过程。JEE 规范允许基于角色的授权。在应用程序中，您指定可以执行操作或访问资源的角色，然后将用户添加到这些角色中。

不幸的是，根据 JEE 规范，保护 JEE 应用程序并非完全独立于服务器。配置的部分是服务器通用的，而部分则是特定于服务器供应商的。常见的配置大多在 `web.xml` 中完成或通过使用注解。但是，服务器特定的配置因供应商而异。在本章中，我们将学习如何在 GlassFish 和 Tomcat 服务器中保护 JEE 应用程序。

但是，在我们学习有关保护应用程序的详细信息之前，我们需要了解在安全配置的上下文中常用的某些术语：

+   **用户**：请求访问应用程序中受保护资源的客户端

+   **组**：具有相似特征的用户的集合

+   **角色**：确定用户或组可以访问应用程序中的哪些资源，具有该特定角色

+   **域**：可以被视为一个安全域，拥有自己的用户、组和存储方法

# 修改数据库以保存身份验证信息

在本章中，我们将使用数据库来验证用户。其他用于存储安全信息的方法包括文件和 LDAP。我们需要更新我们的 `course_management` 数据库，添加存储用户和组信息的表。让我们创建三个表——`User`、`Groups` 和 `User_Group`：

![](img/00268.jpeg)

图 14.1：新的身份验证表

`User` 表存储用户名和密码。`Groups` 表存储组名。我们稍后将直接将组名分组到角色中。`User_Group` 表是一个联合表，将 `User` 和 `Groups` 表连接起来。一个用户可以属于多个组，一个组也可以有多个用户。

为了简化在 JEE 服务器中配置领域时从前面表映射信息，我们将创建一个名为`user_group_view`的视图，使得所有前面表的信息在一个视图中可用。该视图的 DDL 脚本如下：

```java
CREATE
VIEW `user_group_view` AS
    SELECT
        `user`.`user_name` AS `user_name`,
        `groups`.`group_name` AS `group_name`,
        `user`.`password` AS `password`
    FROM
        ((`user`
        JOIN `groups`)
        JOIN `user_group`)
    WHERE
        ((`user`.`id` = `user_group`.`user_id`)
            AND (`groups`.`id` = `user_group`.`group_id`))
```

如果你已经有之前章节中的`course_management`模式，那么在`add_auth_tables.sql`文件（该文件在本章源代码文件夹中）中运行脚本。如果你使用`MySQLWorkbench`，可以按照以下方式运行脚本：

1.  确保默认模式为`course_management`；右键点击模式并选择`设置为默认模式`选项。

1.  选择文件 | 打开 SQL 脚本菜单，然后选择`add_auth_tables.sql`文件。文件将在新标签页中打开。

1.  点击工具栏中的执行图标以执行此脚本。

1.  右键点击`course_management`模式并选择`刷新所有`选项。确保新表和视图已在该模式中创建。

为了测试目的，让我们在`user`表中插入以下数据**：

| **ID** | **user_name** | **password** |
| --- | --- | --- |
| `1` | `user1` | `user1_pass` |
| `2` | `user2` | `user2_pass` |

`组`**：

| **ID** | **group_name** |
| --- | --- |
| `1` | `admin` |

`用户组`**：

| **user_ID** | **group_ID** |
| --- | --- |
| `1` | `1` |

根据前面的数据，`user1`在 admin 组中，而`user2`不在任何组中。

# GlassFish 中的应用程序安全

我们将使用在第七章创建 JEE 应用程序与 EJB 中开发的*Course Management*应用程序来添加安全功能。按照以下步骤导入项目：

1.  为本章创建一个新的 Eclipse 工作空间。

1.  将源代码文件夹中第七章，“使用 EJB 创建 JEE 应用程序”，`with-maven`文件夹内的所有项目复制到当前工作空间中。

1.  将所有项目导入到新工作空间中（打开文件 | 导入菜单，然后选择 Maven | 已存在的 Maven 项目）。

你现在应该在你的 Eclipse 工作空间中有以下项目：`CourseManagementMavenEAR`，`CourseManagementMavenEJBClient`，`CourseManagementMavenEJBs`和`CourseManagementMavenWebApp`。现在让我们学习如何保护文件夹中 JSP 的访问。

# 保护 Web 应用程序中文件夹的访问

为了保护 Web 文件夹中的任何资源，你需要在`web.xml`中声明安全约束。在安全约束中，你可以声明要保护 URL，以及哪些角色可以访问受保护的 URL。在`CourseManagementMavenWebApp`项目中打开`web.xml`并添加以下声明在`<web-app>`标签内：

```java
<security-constraint>
    <display-name>Admin resources</display-name>
    <web-resource-collection>
        <web-resource-name>admins</web-resource-name>
        <url-pattern>/admin/*</url-pattern>
    </web-resource-collection>
    <auth-constraint>
        <role-name>admin </role-name>
    </auth-constraint>
    <!--         
    <user-data-constraint>
        <transport-guarantee>CONFIDENTIAL</transport-guarantee>
    </user-data-constraint>
    -->
</security-constraint>
<security-role>
    <description>Admins</description>
    <role-name>admin</role-name>
</security-role>
```

在这里，我们声明所有通过`/admin/*` URL 访问的资源都是受保护的，并且只有`admin`角色的用户可以访问这些资源。我们还使用`<security-role>`标签声明了`admin`角色。如果您希望 URL 资源仅通过 SSL（使用 HTTPS）访问，则将`<transport-guarantee>`设置为`CONFIDENTIAL`。然而，您需要从证书颁发机构（如 Verisign）获取（购买）SSL 证书，并将其安装在服务器上。

有关 SSL 证书的详细信息，请参阅[`www.verisign.com/en_US/website-presence/website-optimization/ssl-certificates/index.xhtml`](https://www.verisign.com/en_US/website-presence/website-optimization/ssl-certificates/index.xhtml)。

每个服务器安装证书的过程都不同。然而，我们不会在本书中讨论如何安装 SSL 证书。因此，前述代码中描述了`<user-data-constraint>`配置。

有关安全约束的更多详细信息，请参阅[`javaee.github.io/tutorial/security-webtier002.html#specifying-security-constraints`](https://javaee.github.io/tutorial/security-webtier002.html#specifying-security-constraints)。

到目前为止，让我们看看应用程序是如何工作的。在 GlassFish 中部署应用程序之前，让我们创建一个受保护的资源。由于我们已经使用`/admin/*` URL 保护了所有访问的资源，请在`src/main/webapp`文件夹中创建一个名为`admin`的文件夹。在这个文件夹内，使用以下内容创建`admin.jsp`：

```java
<!DOCTYPE HTML>
<html>
<head>
<title>Course Management Admin</title>
</head>
<body>
       Welcome to Course Management Admin<br>
</body>
</html>
```

请参考第七章中的《在 Eclipse 中配置 GlassFish 服务器》部分，以了解如何将 GlassFish 5 服务器添加到您的 Eclipse 工作空间。

我们需要构建两个应用程序：`CourseManagementMavenWebApp`和`CourseManagementMavenEAR`。EAR 项目只是一个容器项目；实际内容是由`CourseManagementMavenWebApp`提供的。因此，我们需要构建这两个项目。在 Eclipse 项目资源管理器中右键单击`CourseManagementMavenWebApp`，然后选择运行方式 | Maven 安装。对于`CourseManagementMavenEAR`项目也执行相同的操作。然后，从 GlassFish 5 的目标文件夹中部署`CourseManagementMavenEAR-1.ear`。

要在 GlassFish 5 中部署应用程序，请浏览到`http://localhost:4848`并配置数据源，如第七章中《使用 EJB 创建 JEE 应用程序》的《在 GlassFish 中配置数据源》部分所述。然后，单击应用程序节点并部署`CourseManagementMavenEAR-1.ear`<q class="calibre258">。</q>

一旦应用程序部署完成，请浏览到`http://localhost:8080/CourseManagementMavenWebApp/course.xhtml`并确保页面可以在不进行任何认证的情况下访问，因为这是一个未受保护的资源/页面。现在，尝试浏览到`http://localhost:8080/CourseManagementMavenWebApp/admin/admin.jsp`。由于我们已经将`/admin/*` URL 模式标记为受保护资源，浏览器会弹出此认证对话框：

![](img/00269.jpeg)

图 14.2：浏览器认证对话框

我们尚未配置应用程序进行用户认证。因此，无论你输入什么用户名和密码，前面的对话框都会认证失败。让我们通过配置数据库在 GlassFish 中认证用户来解决这个问题。

# 在 GlassFish 中配置 JDBC 域

GlassFish 支持不同的域用于 JEE 认证；例如，文件、LDAP 和 JDBC 域。我们将创建一个 JDBC 域，它将使用存储在`User`、`Groups`和`User_Groups`表中的信息（通过`user_group_view`公开）。

要在 GlassFish 中创建一个新的 JDBC 域，请浏览到 GlassFish 管理页面（`http://localhost:4848`），然后在左侧的导航菜单中，转到配置 | 服务器配置 | 安全 | 域。在域页面，点击新建按钮。

![](img/00270.jpeg)

图 14.3：在 GlassFish 管理页面上创建 JDBC 域

将以下信息输入到表单中：

类名

| **字段名** | **值** | **备注** |
| --- | --- | --- |
| 名称 | `courseManagementJDBCRealm` |  |
|  | `com.sun.enterprise.security.auth.realm.jdbc.JDBCRealm` | 从下拉菜单中选择。 |
| JAAS 上下文 | `jdbcRealm` |  |
| JNDI | `jdbc/CourseManagement` | 我们创建的 JDBC 数据源。有关更多详细信息，请参阅第七章，*使用 EJB 创建 JEE 应用程序*。 |
| 用户表 | `user_group_view` | 包含用户信息的表。我们指定了我们之前创建的视图。 |
| 用户名列 | `user_name` | 我们`user_group_view`中的用户名列。 |
| 密码列 | `password` | 我们`user_group_view`中的密码列。 |
| 组表 | `user_group_view` | 组数据也通过我们的`user_group_view`公开。 |
| 组表用户名列 | `user_name` | 在`user_group_view`中。 |
| 组名称列 | `group_name` | 在`user_group_view`中。 |
| 密码加密算法 | `AES` | 加密数据库中密码的算法。我们在应用程序外部预先填充密码。因此，这对我们的示例影响不大。 |
| 摘要算法 | `none` | 我们在表中输入的密码没有被哈希，所以在这里输入`none`。 |

点击 OK 按钮创建域。

我们需要告诉我们的应用程序使用之前创建的 JDBC 域。这配置在应用程序的`web.xml`文件中的`<login-config>`标签中。`<login-config>`支持两种认证方法：基本认证和基于表单的认证。

在基本身份验证中，浏览器显示登录表单，就像在 *图 14.2* 中所示。实际上，这是默认的身份验证方法，因此在我们之前的 `web.xml` 中没有 `<login-config>` 标签的情况下，服务器默认为基本身份验证。

在基于表单的身份验证中，你可以指定登录页面。这给了你一个机会来自定义登录体验。

让我们先使用基本身份验证配置域。

# 在 GlassFish 中使用 JDBC 域的基本身份验证

我们将对在 *保护对 web 应用程序文件夹的访问* 部分中添加的标签进行一些更改。以下是更改内容：

1.  将 `role-name` 从 `admin` 重命名为 `admin-role`

1.  删除 `<security-role>` 标签

1.  添加 `<login-config>` 标签

这里是更改后的声明应该看起来像什么：

```java
<security-constraint>
    <display-name>Admin resources</display-name>
    <web-resource-collection>
        <web-resource-name>admins</web-resource-name>
        <url-pattern>/admin/*</url-pattern>
    </web-resource-collection>
    <auth-constraint>
        <role-name>admin-role</role-name>
    </auth-constraint>
</security-constraint>
<login-config>
    <auth-method>BASIC</auth-method>
    <realm-name>courseManagementJDBCRealm</realm-name>
</login-config>
```

注意，我们在 `<login-config>` 标签中指定了我们配置的域的名称（在 GlassFish 管理页面上配置的）。我们删除了 `<security-role>`，因为角色现在保存在数据库中的 Groups 表中。然而，我们需要将 `web.xml` 中声明的角色映射到数据库中的组。这种映射是在 `glassfish-web.xml` 中完成的。在 `web.xml` 相同的文件夹中创建 `glassfish-web.xml`，即 `src/main/webapp/WEB-INF`，在 `CourseManagementMavenWebApp` 项目中。向其中添加以下内容：

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE glassfish-web-app PUBLIC "-//GlassFish.org//DTD GlassFish Application Server 3.1 Servlet 3.0//EN" "http://glassfish.org/dtds/glassfish-web-app_3_0-1.dtd">
<glassfish-web-app error-url="">
    <security-role-mapping>
       <role-name>admin-role</role-name>
       <group-name>admin</group-name>
    </security-role-mapping>
</glassfish-web-app>
```

这里，我们将我们在 `web.xml` 中声明的 `admin-role` 映射到数据库中 Groups 表的 admin 组。

现在，通过右键单击项目并选择 Run As | Maven Install 来构建 `CourseManagementMavenWebApp` 和 `CourseManagementMavenEAR` 项目（按相同顺序），然后按照 *保护对 web 应用程序文件夹的访问* 部分中描述的方式在 GlassFish 中部署应用程序。

浏览到 `http://localhost:8080/CourseManagementMavenWebApp/admin/admin.jsp`。这次，一旦你输入有效的管理员凭据；即用户名为 `user1`，密码为 `user1_pass`，浏览器应该显示 `admin.jsp` 的内容。

# 在 GlassFish 中使用 JDBC 域的基于表单的身份验证

让我们将基本身份验证更改为基于表单的身份验证，这样我们就可以自定义登录页面。我们需要更新 `web.xml` 中的 `<login-config>`。将之前的 `<login-config>` 块替换为以下内容：

```java
<login-config>
    <auth-method>FORM</auth-method>
    <realm-name>courseManagementJDBCRealm</realm-name>
    <form-login-config>
        <form-login-page>/login.jsp</form-login-page>
        <form-error-page>/login-error.jsp</form-error-page>
    </form-login-config>
</login-config>
```

我们已经将 `<auth-method>` 从 BASIC 更改为 FORM。对于基于表单的身份验证，我们需要指定 form-login-page，我们已指定为 `login.jsp`。form-error-page 是可选的，但我们已将其设置为 `login-error.jsp`。

下一步是创建 `login.jsp` 和 `login-error.jsp`。在 `src/main/webapp` 文件夹中创建这两个文件，内容如下。

这里是 `login.jsp` 的源代码。我们已经将其配置为登录页面，如前面的代码块所示：

```java
<!DOCTYPE HTML>
<html>
<head>
<title>Admin Login</title>
</head>
<body>
    <form method=post action="j_security_check">
        <table>
            <tr>
                <td>User Name: </td>
                <td><input type="text" name="j_username"></td>
            </tr>
            <tr>
               <td>Password: </td>
               <td><input type="password" name="j_password"></td>
            </tr>
            <tr>
                <td colspan="2"><input type="submit" value="Login"></td>
            </tr>
        </table>
    </form>
</body>
</html>
```

为了使基于表单的身份验证生效，有一些要求：

1.  表单操作必须设置为 `j_security_check`

1.  用户名输入字段必须命名为 `j_username`

1.  密码输入字段必须命名为`j_password`

这里是`login-error.jsp`的源代码。我们已将其配置为`<form-error-page>`中的错误页面，如前一个代码块所示：

```java
<!DOCTYPE HTML>
<html>
<head>
<title>Login Failed</title>
</head>
<body>
       Invalid user name or password<br>
       <a href="<%=request.getContextPath()%>/admin/admin.jsp">Try Again</a>
</body>
</html>
```

错误页面显示错误消息并显示重试链接。即使重试链接指向`admin.jsp`，因为它是一个受保护资源，用户将被重定向到`login.jsp`。如果登录成功，则将重定向到`admin.jsp`。

在用户成功登录后提供注销选项将很棒。此选项可以添加到`admin.jsp`中。如下所示在`admin.jsp`中添加一个注销链接：

```java
<!DOCTYPE HTML>
<html>
<head>
<title>Course Management Admin</title>
</head>
<body>
       Welcome to Course Management Admin<br>
       <a href="../logout.jsp">Logout</a>
</body>
</html>
```

在与`login.jsp`相同的文件夹中创建`logout.jsp`，内容如下：

```java
<%@ page session="true"%>
Logged out <%=request.getRemoteUser()%>
<% session.invalidate(); %>
```

注销页面简单地调用`session.invalidate()`来注销用户。

要查看基于表单的认证的实际操作，通过右键单击项目并选择运行 As | Maven Install 来构建`CourseManagementMavenWebApp`和`CourseManagementMavenEAR`项目（按相同顺序），然后按照*保护 Web 应用程序中文件夹的访问*部分所述在 GlassFish 中部署应用程序。

浏览到`http://localhost:8080/CourseManagementMavenWebApp/admin/admin.jsp`。这次，浏览器应该显示带有登录表单的`login.jsp`，而不是它自己的认证弹出窗口。

# 在 Tomcat 中保护应用程序

在本节中，我们将学习如何在 Tomcat 服务器中保护资源。为了使示例与我们在上一节中为 GlassFish 学习的示例保持一致，我们将保护`admin`文件夹中的所有页面。我们将使用我们在第四章，“创建 JEE 数据库应用程序”中创建的`CourseManagementJDBC`项目开始。回想一下，在[第四章](https://cdp.packtpub.com/java_ee_development_with_eclipse___third_edition/wp-admin/post.php?post=378&action=edit#post_216)，“创建 JEE 数据库应用程序”中，我们在 Tomcat 服务器中部署了此项目。执行以下步骤将项目导入本章的新工作区并配置 Tomcat：

1.  将`CourseManagementJDBC`项目从第七章，“使用 EJB 创建 JEE 应用程序”的项目文件夹复制到当前工作区。将项目导入新工作区（打开文件 | 导入菜单，然后选择 Maven | 已存在的 Maven 项目）。

1.  如第一章“介绍 JEE 和 Eclipse”中的*配置 Eclipse 中的 Tomcat*部分所述配置 Tomcat。

1.  确保应用程序已添加到服务器并按预期运行。请参阅第二章“创建简单的 JEE Web 应用程序”中的*在 Tomcat 中运行 JSP*部分。

1.  将`CourseManagementMavenWebApp`（参见本章前面的上一节）中的`admin`文件夹复制到`CourseManagementJDBC`项目的`src/main/webapp`中。因此，保护 admin 文件夹的代码对于 GlassFish 和 Tomcat 中的项目都是相同的。

因此，现在你应该已经在 Eclipse 中配置了`CourseManagementJDBC`项目和 Tomcat。

我们现在将修改`web.xml`以添加安全约束，就像我们在上一节中为 GlassFish 所做的那样：

```java
<security-constraint>
    <display-name>Admin resources</display-name>
    <web-resource-collection>
        <web-resource-name>admins</web-resource-name>
        <url-pattern>/admin/*</url-pattern>
    </web-resource-collection>
    <auth-constraint>
        <role-name>admin</role-name>
    </auth-constraint>
</security-constraint>
<login-config>
    <auth-method>FORM</auth-method>
    <form-login-config>
        <form-login-page>/login.jsp</form-login-page>
        <form-error-page>/login-error.jsp</form-error-page>
    </form-login-config>
</login-config> 
```

与 GlassFish 的配置相比，前面的配置有两个不同之处：

+   没有必要像在 GlassFish 中那样将`role-name`映射到组名。因此，在`<auth-constaint>`中将角色名称从`admin-role`更改为仅`admin`。

+   在`<login-config>`中不需要`<realm-name>`标签。

现在我们通过在`server.xml`中添加`<realm>`标签来配置 Tomcat 中的 JDBC realm。如果你使用在 Eclipse 中配置的 Tomcat 来运行应用程序，那么你可以通过展开项目资源管理器中的“Servers”节点来访问`server.xml`。

![图片](img/00271.jpeg)

图 14.4：在 Eclipse 配置的 Tomcat 服务器中访问 server.xml

如果你是在 Eclipse 之外运行 Tomcat，那么你将在`$CATALINA_BASE/conf/server.xml`中找到`server.xml`。

在`server.xml`中的`<Engine defaultHost="localhost" name="Catalina">`节点内添加以下 realm 标签：

```java
<Realm  className="org.apache.catalina.realm.JDBCRealm"
    driverName="com.mysql.jdbc.Driver"
    connectionURL="jdbc:mysql://localhost:3306/course_management"
    connectionName="<your-db-username>"
    connectionPassword="<your-db-password>"
    userTable="user_group_view"
    userNameCol="user_name"
    userCredCol="password"
    userRoleTable="user_group_view"
    roleNameCol="group_name" />
```

Tomcat 的管理模块需要访问我们的 MySQL 数据库，因此我们需要使 MySQL JDBC 驱动程序对管理模块可用。将 MySQL JDBC 驱动程序复制到`<tomcat-install-dir>/lib`。如果你还没有这样做，可以从[`dev.mysql.com/downloads/connector/j/`](https://dev.mysql.com/downloads/connector/j/)下载驱动程序。

这就是保护 Tomcat 中文件夹所需的所有内容。重新启动服务器，并浏览到`http://localhost:8080/CourseManagementJDBC/admin/admin.jsp`。你应该看到登录页面。

# 使用注解保护 servlet

到目前为止，我们已经看到了指定安全约束的声明性语法；即通过在`web.xml`中指定`<security-constraint>`。然而，安全约束也可以使用 Java 注解来指定，特别是对于 servlet。在本节中，我们将创建`AdminServlet`并使用注解来保护它。按照上一节中的步骤从`Chapter09`导入`CourseManagementJDBC`项目，但将其重命名为`CourseManagementJDBC-SecureAnnotations`，并将其导入工作区。然后，只在`web.xml`中添加`<login-config>`，但不指定`<security-constraint>`：

```java
  <login-config>
    <auth-method>FORM</auth-method>
    <form-login-config>
      <form-login-page>/login.jsp</form-login-page>
      <form-error-page>/login-error.jsp</form-error-page>
    </form-login-config>
  </login-config>
```

确保你已经按照上一节中的描述复制了`login.jsp`和`login-error.jsp`。

现在在`packt.book.jee.eclipse.ch4.servlet`包中创建一个名为`AdminServlet`的 servlet，内容如下：

```java
package packt.book.jee.eclipse.ch4.servlet;
// skipping imports to save space
@WebServlet("/AdminServlet")
@ServletSecurity(@HttpConstraint(rolesAllowed = "admin"))
public class AdminServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;
    public AdminServlet() {
        super();
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {
        try {
            request.authenticate(response);
            response.getWriter()
                    .append("Served at: ").append(request.getContextPath());
        } finally {
            response.getWriter().close();
        }      
     }

    protected void doPost(HttpServletRequest request, 
        HttpServletResponse response) throws ServletException, IOException {
        doGet(request, response);
    }
}
```

`@ServletSecurity(@HttpConstraint(rolesAllowed = "admin"))`指定了 servlet 的安全约束。使用此注解，我们只允许具有 admin 角色的用户访问 servlet。如果你浏览到`http://localhost:8080/CourseManagementJDBC-SecurityAnnotations/AdminServlet`，你应该看到登录页面。

# 保护 Web 服务

保护 Web 服务的过程与保护 Web URL 的过程类似，我们已经在前面的章节中看到了两个例子。我们在`web.xml`中指定了`<security-constraint>`和`<login-config>`。让我们看看如何保护我们在第九章中开发的 REST Web 服务：*创建 Web 服务*：

1.  将`CourseManagementREST`和`CourseManagementRESTClient`项目从`Chapter09`复制并导入到本章的工作区。正如其名称所暗示的，第一个项目是 REST 服务，第二个项目是一个独立的客户端应用程序，它调用 Web 服务。

1.  在 Tomcat 中部署`CourseManagementREST`项目（有关如何操作的详细信息，请参阅前节）。

1.  确保从`CourseManagementClient.java`中的`main`方法调用`testGetCoursesJSON`方法，来自`*CourseManagementRESTClient*`项目。

1.  运行应用程序（在项目资源管理器中右键单击文件并选择 Run As | Java Application），并验证服务是否运行正常。

要使用基本认证保护 Web 服务，请在`web.xml`中添加以下配置：

```java
:<security-constraint>
    <display-name>Admin resources</display-name>
    <web-resource-collection>
        <web-resource-name>admins</web-resource-name>
            <url-pattern>/services/*</url-pattern>
        </web-resource-collection>
        <auth-constraint>
            <role-name>admin</role-name>
        </auth-constraint>
 </security-constraint>
<login-config>
    <auth-method>BASIC</auth-method>
</login-config>   
```

使用上述配置，我们正在保护包含`/services/`的任何 URL。我们还指定只有 admin 角色可以访问此 URL，并且认证方法是 BASIC。

现在，在 Tomcat 的`server.xml`中添加如前节所述的`<Realm>`配置。如果你此时运行`CourseManagementClient.java`，你会得到一个未授权错误。这是因为客户端应用程序没有在`GET`请求中发送认证信息——即用户名和密码。对于基本认证方法，这些信息应该通过`authorization`头传递。此头参数的值应设置为`Basic`，后跟 base64 编码的`username:password`字符串；例如，`authorization: Basic dXNlcjE6dXNlcjFfcGFzcw==`。

在前面的头中，`dXNlcjE6dXNlcjFfcGFzcw==`是`user1:user1_pass`字符串的 base64 编码格式。

现在，让我们修改`CourseManagementClient.java`中的`testGetCoursesJSON`方法，以传递前面的头信息。以下是你需要在检查响应状态之前添加的代码：

```java
String userName = "user1";
String password = "user1_pass";
String authString = userName + ":" + password;
String encodedAuthStr =  Base64.*getEncoder*().encodeToString(authString.getBytes());
//Execute HTTP get method
Response response = webTarget.request(MediaType.*APPLICATION_JSON*).header(HttpHeaders.*AUTHORIZATION*, "Basic " + encodedAuthStr).get();
```

注意，`java.util.Base64`从 JDK 1.8 开始可用。如果你使用的是低于 1.8 的版本，你可以使用 Apache `commons-codec`中的`org.apache.commons.codec.binary.Base64`。在`pom.xml`中添加以下依赖项：

```java
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.11</version>
</dependency>
```

右键单击项目，选择 Run As | Maven Install。然后，通过调用以下方式编码`String`：

```java
encodedAuthStr =  new String(org.apache.commons.codec.binary.Base64.encodeBase64(authString.getBytes()));
```

当你现在运行应用程序时，Web 服务应该能够无错误地执行。

# JEE 8 的安全性增强

JEE 8 集成了 Java EE Security API 1.0 (JSR 375, [`javaee.github.io/security-spec/`](https://javaee.github.io/security-spec/))。这些 API 的增强被广泛分为四个类别：

+   支持 Servlet 4.0 认证机制。您可以使用注解在 Servlet 中指定认证类型。例如，`@BasicAuthenticationMechanismDefinition` 用于基本认证，`@FormAuthenticationMechanismDefinition` 用于基于表单的认证，以及 `@CustomFormAuthenticationMechanismDefinition` 用于自定义认证。更多信息，请参阅 [`javaee.github.io/security-spec/spec/jsr375-spec.html#authentication-mechanism`](https://javaee.github.io/security-spec/spec/jsr375-spec.html#authentication-mechanism).

+   身份存储 API。通过实现 Identity Store 接口，您可以指定如何将用户、密码和组信息提供给 JEE 认证和授权 API。通过实现此接口，您可以使与安全性相关的代码在 JEE 8 容器之间具有可移植性。更多信息，请参阅 [`javaee.github.io/security-spec/spec/jsr375-spec.html#_introduction_2`](https://javaee.github.io/security-spec/spec/jsr375-spec.html#_introduction_2).

+   新的 `SecurityContext` API 提供了一致的 API 来获取有关用户和角色的信息。更多信息，请参阅 [`javaee.github.io/security-spec/spec/jsr375-spec.html#security-context`](https://javaee.github.io/security-spec/spec/jsr375-spec.html#security-context).

+   新的 `HttpAuthenticationMechanism` API 允许你通过编程方式完全控制你想要在应用程序中实现的安全性。更多信息，请参阅 [`javaee.github.io/security-api/apidocs/javax/security/enterprise/authentication/mechanism/http/HttpAuthenticationMechanism.html`](https://javaee.github.io/security-api/apidocs/javax/security/enterprise/authentication/mechanism/http/HttpAuthenticationMechanism.html).

我们在本章中不会涵盖所有上述增强功能，但我们将详细探讨前三个 API。

在本章的前几节中，我们已经看到安全配置在容器之间并不统一。具体来说，角色到组的映射并不统一。这个问题可以通过使用新的 JEE 8 安全 API 来解决。让我们通过开发一个应用程序来看看如何实现。有关源代码中的 `CourseManagementMavenWebApp-jee8` 项目，请参阅。

# 在 JEE 8 中实现可移植安全性

我们将在本节中修改 `CourseManagementMavenWebApp`，它来自 第七章，*使用 EJB 创建 JEE 应用程序*。这个项目是 EJB `CourseManagementMavenEAR`项目的部分，但在本节中，我们将独立地使用`CourseManagementMavenWebApp`。将`CourseManagementMavenWebApp`项目从`Chapter07`复制到本章节的 Eclipse 工作空间中，命名为`CourseManagementMavenWebApp-jee8`。

我们将修改这个项目以提供以下功能：

+   `AdminServlet`是一个受保护的 servlet，需要登录。我们将实现基本认证。

+   有三种可能的用户角色：管理员、经理和用户。

+   只有管理员角色的用户才能看到由`AdminServlet`提供的管理页面。

+   只有在管理员角色的用户才能看到由`ManagementServlet`提供的管理页面。

JEE 8 安全 API 需要启用应用程序中的**上下文和依赖注入**（**CDI**）。我们只需要在`src/main/webapp/WEB-INF`文件夹中创建一个空的`beans.xml`文件来启用 CDI。

接下来，我们需要在`pom.xml`中添加以下 Maven 依赖项，以便在应用程序中提供 JEE 8 API：

```java
  <dependency>
   <groupId>javax</groupId>
   <artifactId>javaee-api</artifactId>
   <version>8.0</version>
   <scope>provided</scope>
  </dependency>
```

让我们在`packt.book.jee.eclipse.ch7.web.servlet`包中创建一个名为`ApplicationConfig`的类（用于声明应用程序中允许的所有用户角色）。以下是`ApplicationConfig`类的源代码：

```java
package packt.book.jee.eclipse.ch7.web.servlet;
import javax.annotation.security.DeclareRoles;
import javax.enterprise.context.ApplicationScoped;
@DeclareRoles({"admin", "user", "manager"})
@ApplicationScoped
public class ApplicationConfig {}
```

现在，让我们在`packt.book.jee.eclipse.ch7.web.servlet`包中创建两个 servlet，`AdminServlet`和`ManagementServlet`。如果你使用 Eclipse 向导创建这些类，那么它会自动在`web.xml`中添加 servlet 声明和映射。如果你没有使用 Eclipse 向导，那么你可以手动添加声明和映射，或者在 servlet 类中添加`@WebServlet`注解。以下是`AdminServlet`的源代码：

```java
package packt.book.jee.eclipse.ch7.web.servlet;

// skipped imports

@BasicAuthenticationMechanismDefinition()
@ServletSecurity(@HttpConstraint(rolesAllowed = {"admin", "manager"} ))
public class AdminServlet extends HttpServlet {

 private static final long serialVersionUID = 1L;

 @Inject
 private SecurityContext securityContext;

 public AdminServlet() {
 super();
 }

 protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
 if (securityContext.isCallerInRole("manager")) {
 request.getRequestDispatcher("/ManagementServlet").forward(request, response);
 } else {
 response.getWriter().append("Welcome to Admin Page!");
 }
 }

 protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
 doGet(request, response);
 }
}
```

这个 servlet 使用`@ServletSecurity`注解允许只有管理员和经理角色访问，并通过`@BasicAuthenticationMechanismDefinition`指定基本认证类型。我们还要求 JEE 容器注入一个`SecurityContext`实例，该实例在`doGet`方法中使用以检查用户的角色。如果用户是经理角色，则请求被转发到`ManagementServlet`，否则，允许当前 servlet 的访问。注意调用`securityContext.isCallerInRole`来检查用户的角色。

下面是`ManagementServlet`的源代码：

```java
package packt.book.jee.eclipse.ch7.web.servlet;
// skipped imports
@BasicAuthenticationMechanismDefinition(realmName="basic")
@ServletSecurity(@HttpConstraint(rolesAllowed = {"manager"} ))
public class ManagementServlet extends HttpServlet {
 private static final long serialVersionUID = 1L;

 public ManagementServlet() {
   super();
 }

 protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
 response.getWriter().append("Welcome to Management Page!");
 }

 protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
 doGet(request, response);
 }
}
```

前面的 servlet 也使用了基本认证，并且只允许管理员角色的用户访问。

使用前面的注解，不需要在`web.xml`或任何自定义容器特定文件中进行声明性配置。但是，我们如何告诉安全 API 哪些是有效的用户和角色呢？我们通过实现`IdentityStore`接口来实现。在`packt.book.jee.eclipse.ch7.web.servlet`包中创建`SimpleMapIdentityStore`类。这个类应该实现`IdentityStore`接口：

```java
package packt.book.jee.eclipse.ch7.web.servlet;

// skipped imports

@ApplicationScoped
public class SimpleMapIdentityStore implements IdentityStore {
 class UserInfo {
   String userName;
   String password;
   String role;

   public UserInfo(String userName, String password, String role) {
     this.userName = userName;
     this.password = password;
     this.role = role;
   }
 }

 private HashMap<String, UserInfo> store = new HashMap<>();

 public SimpleMapIdentityStore() {
   UserInfo user1 = new UserInfo("user1", "user1_pass", "admin");
   UserInfo user2 = new UserInfo("user2", "user2_pass", "user");
   UserInfo user3 = new UserInfo("user3", "user3_pass", "manager");
   store.put(user1.userName, user1);
   store.put(user2.userName, user2);
   store.put(user3.userName, user3);
 }

 public CredentialValidationResult validate(UsernamePasswordCredential usernamePasswordCredential) {
   String userName = usernamePasswordCredential.getCaller();
   String password = usernamePasswordCredential.getPasswordAsString(); 

   UserInfo userInfo = this.store.get(userName.toLowerCase());
   if (userInfo == null || !userInfo.password.equals(password)) {
     return INVALID_RESULT;
   }

   return new CredentialValidationResult(userInfo.userName, new HashSet<>(asList(userInfo.role)));
 } 
}
```

重要的是，前面的类需要注解为`@ApplicationScoped`，这样它就可以在整个应用程序中可用，并且 CDI 可以注入它。我们在前面的类中硬编码了用户和角色到一个`HashMap`中，但你也可以编写代码从任何来源获取用户和角色，例如数据库、LDAP 或文件。在应用程序中，可以有多个`IdentityStore`。容器将调用每个`validate`方法。在`validate`方法中，我们首先验证用户名和密码是否有效，然后返回一个带有用户角色的`CredentialValidationResult`实例。

构建应用程序（在项目上右键单击并选择 Run As | Maven Install），然后按照前面章节的描述将其部署到 GlassFish 5 服务器。确保应用程序的上下文设置为`/CourseManagementMavenWebApp-jee8`。你可以在 GlassFish 管理页面上通过编辑已部署的应用程序并验证上下文根字段的值来验证这一点。然后浏览到`http://localhost:8080/CourseManagementMavenWebApp-jee8/AdminServlet`。如果你使用 user1 凭据登录，则将显示管理页面。如果你以 user3 的身份登录，则将显示管理页面。其他所有用户的访问都被阻止。你需要关闭浏览器窗口以尝试使用不同的用户登录，因为一旦登录，用户凭据将被记住，直到会话被无效化。应用程序可以很容易地扩展以添加注销选项，就像我们在前面的章节中所做的那样。

在前面的示例中，我们已经创建了一个自定义的身份存储。你可以在这个地方实现任何代码来获取用户信息，无论是从数据库还是 LDAP。但是，JEE 安全 API 提供了内置的注解来访问数据库和 LDAP 作为身份存储；即`@DatabaseIdentityStoreDefinition`和`@LdapIdentityStoreDefinition`。例如，我们可以修改`ApplicationConfig`类来声明一个数据库身份存储，如下所示：

```java
package packt.book.jee.eclipse.ch7.web.servlet;

import javax.enterprise.context.ApplicationScoped;
import javax.security.enterprise.identitystore.DatabaseIdentityStoreDefinition;
import javax.security.enterprise.identitystore.PasswordHash;

@DatabaseIdentityStoreDefinition (
 dataSourceLookup = "jdbc/CourseManagement",
 callerQuery = "select password from user_group_view where user_name = ?",
 groupsQuery = "select group_name from user_group_view where user_name = ?",
 hashAlgorithm = PasswordHash.class,
 priority = 10 
)
@ApplicationScoped
public class ApplicationConfig {
}
```

我们需要传递 JDBC 资源的 JNDI 查找名称，即`jdbc/CourseManagement`，以及用于验证用户名和密码以及获取组的 SQL 查询。这些与我们在 GlassFish 管理页面上创建 Realm 时配置的 SQL 查询类似，但使用新的安全 API，使配置更加便携。有关`IdentityStore`注解的更多详细信息，请参阅[`javaee.github.io/security-spec/spec/jsr375-spec.html#_annotations_and_built_in_identitystore_beans`](https://javaee.github.io/security-spec/spec/jsr375-spec.html#_annotations_and_built_in_identitystore_beans)。

在前面的示例中，我们使用了基本的认证类型。但是，你可以使用基于表单的认证，通过使用`@FormAuthenticationMechanismDefinition`注解。例如，我们可以将`@BasicAuthenticationMechanismDefinition`替换为`@FormAuthenticationMechanismDefinition`，如下所示：

```java
package packt.book.jee.eclipse.ch7.web.servlet;
// ...
@FormAuthenticationMechanismDefinition(
 loginToContinue = @LoginToContinue(
 loginPage = "/loginServlet",
 errorPage = "/loginErrorServlet"
 )
)
@DeclareRoles({"admin"})
@ServletSecurity(@HttpConstraint(rolesAllowed = "admin"))
public class AdminServlet extends HttpServlet {
 ...
}
```

此配置类似于我们在早期示例中在`web.xml`中配置的`<form-login-config>`。

注意，新的安全 API 主要在 Java 类上工作，例如 servlets、EJBs 和 beans，但如果你想要保护 JSP 页面，那么你需要使用我们在前几节中学到的声明性配置。

JEE 中的安全是一个非常大的主题，无法在通用的书中涵盖。本章的范围仅限于使用用户名和密码保护 JEE 资源。有关 JEE 中安全性的详细信息，请参阅[`javaee.github.io/tutorial/security-intro.html`](https://javaee.github.io/tutorial/security-intro.html)。

# 概述

在本章中，我们学习了如何在 JEE 应用程序中保护由 URL 表示的资源。在 JEE 中，声明性保护资源的过程并不完全通用；其中一部分在所有服务器中是通用的，特别是`web.xml`中的配置。声明性安全域的配置在不同服务器中有所不同。然而，JEE 8 添加了新的 Java EE 安全 API，使得基于注解的配置对 Java 类是可移植的。

我们学习了如何在 GlassFish 和 Tomcat 服务器中保护文件夹。我们还学习了如何保护 RESTful Web 服务，并在客户端应用程序中使用安全凭据调用它们。
