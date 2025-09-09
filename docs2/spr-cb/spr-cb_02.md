# 第二章。定义 Bean 和使用依赖注入

在本章中，我们将介绍以下菜谱：

+   定义一个 Bean 显式使用 @Bean

+   使用 @Component 隐式定义一个 Bean

+   通过 @Autowired 注入依赖使用 Bean

+   直接使用 Bean

+   列出所有 Bean

+   使用多个配置类

# 简介

**Bean** 是 Spring 的核心。它们是由 Spring 实例化和管理的标准 Java 对象。

Bean 主要用于：

+   以某种方式配置 Spring（数据库连接参数、安全等）

+   避免使用 **依赖注入** 硬编码依赖，这样我们的类才能保持自包含和可单元测试

在本章中，你将学习如何定义 Bean 并使用它们。

# 使用 @Bean 显式定义一个 Bean

定义 Bean 的最简单方法是在 Spring 配置类中创建一个带有 `@Bean` 注解的方法，返回一个对象（实际的 Bean）。这类 Bean 通常用于以某种方式配置 Spring（数据库、安全、视图解析等）。在这个菜谱中，我们将定义一个包含数据库连接详情的 Bean。

## 如何做…

在 Spring 配置类中添加一个带有 `@Bean` 注解的 `dataSource()` 方法，并返回一个 `Datasource` 对象。在这个方法中，创建一个初始化了数据库连接详情的 `DriverManagerDataSource` 对象：

```java
@Bean
public DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();

        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/db1");
        dataSource.setUsername("root");
        dataSource.setPassword("123");

        return dataSource;
}
```

## 它是如何工作的…

在启动时，由于 `@Bean`，`dataSource()` 方法会自动执行并返回一个 `Datasource` 对象，该对象被 Spring 存储在一个称为 `ApplicationContext` 的 Spring 对象中。Bean 名称是 `dataSource`，与它的方法名称相同。从这一点开始，任何对 `dataSource()` 的调用都将返回相同的缓存 `DataSource` 对象；`dataSource()` 不会再次实际执行。这是通过面向切面编程实现的；任何对 `dataSource()` 的调用都会被 Spring 拦截，直接返回对象而不是执行方法。

## 还有更多…

要自定义 Bean 名称，请使用名称参数：

```java
@Bean(name="theSource")
public DataSource dataSource() {
...
```

要强制 `dataSource()` 在每次调用时执行（并返回不同的对象），请使用带有 `prototype` 范围的 `@Scope` 注解：

```java
@Bean
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public DataSource dataSource() {
...
```

使用我们自己的类定义 Bean 是可能的。例如，如果我们有一个 `UserService` 类，我们可以在 Spring 配置类中定义一个 `UserService` Bean：

```java
@Bean
public UserService userService() {
        return new UserService();
}
```

然而，通常更简单的是让 Spring 通过在 `UserService` 类上使用 `@Component` 注解自动生成这类 Bean，正如在 *使用 @Component 隐式定义一个 Bean* 菜谱中解释的那样。

# 使用 @Component 隐式定义一个 Bean

Bean 不必定义在 Spring 配置类中。Spring 会自动从任何带有 `@Component` 注解的类生成 Bean。

## 准备工作

我们将在 第一章 中创建的基本 Web 应用程序 *创建 Spring Web 应用程序* 菜单中，使用 *创建 Spring 应用程序*。

创建 `com.springcookbook.service` 包以及其中的以下服务类：

```java
public class UserService {
  public int findNumberOfUsers() {
    return 10;
  }
}
```

## 如何做到这一点...

定义 Bean 的步骤如下，通过向现有类添加 `@Component`：

1.  在 Spring 配置文件中，在 `@ComponentScan` 类注解中添加 `com.springcookbook.service` 基础包：

    ```java
    @Configuration
    @EnableWebMvc
    @ComponentScan(basePackages = {"com.springcookbook.controller", "com.springcookbook.service"})
    public class AppConfig {  
    }
    ```

1.  在 `UserService` 类中添加 `@Component`：

    ```java
    @Component
    public class UserService {
      public int findNumberOfUsers() {
        return 10;
      }
    }
    ```

## 它是如何工作的...

在启动时，Spring 会扫描 `com.springcookbook.service` 包。`UserService` 类被注解为 `@Component`，因此会自动从它实例化一个 Bean。默认情况下，Bean 的名称将是 `userService`，基于类名。

要指定一个自定义名称，请使用以下代码：

```java
@Component('anAmazingUserService')
public class UserService {
```

## 更多内容...

如果 `UserService` Bean 需要进行一些自定义初始化，例如基于当前环境，可以像在之前的配方中解释的那样显式定义和初始化 Bean，即 *使用 @Bean 显式定义 Bean*。

`@Controller`、`@Service` 和 `@Repository` 也是一种组件注解；Spring 在启动时会自动从带有这些注解的类中实例化一个 Bean。使用这些组件注解并不是严格必要的，但它们可以使组件类的角色更加清晰；`@Controller` 用于控制器类，`@Service` 用于服务类（因此我们会为我们的 `UserService` 类使用它），而 `@Repository` 用于持久化类。它们还为组件类添加了一些额外的功能。请参阅 [`docs.spring.io/spring-framework/docs/current/spring-framework-reference/html/beans.html#beans-stereotype-annotations`](http://docs.spring.io/spring-framework/docs/current/spring-framework-reference/html/beans.html#beans-stereotype-annotations)。

# 通过 @Autowired 进行依赖注入使用 Bean

Spring 配置 Bean，如 *使用 @Bean 显式定义 Bean* 配方中的 Bean，会被 Spring 自动发现和使用。要在你的类中使用一个 Bean（任何类型的 Bean），请将 Bean 添加为字段并注解它为 `@Autowired`。Spring 会自动初始化这个字段为 Bean。在这个配方中，我们将在控制器类中使用一个现有 Bean。

## 准备工作

我们将使用 *使用 @Component 隐式定义 Bean* 配方中的代码，其中我们定义了一个 `UserService` Bean。

## 如何做到这一点...

这里是使用现有 Bean 的步骤：

1.  在控制器类中，添加一个被 `@Autowired` 注解的 `UserService` 字段：

    ```java
    @Autowired
    UserService userService;
    ```

1.  在控制器方法中，使用 `UserService` 字段：

    ```java
    @RequestMapping("hi")
    @ResponseBody
    public String hi() {
      return "nb of users: " + userService.findNumberOfUsers();
    }
    ```

1.  在浏览器中，访问 `http://localhost:8080/hi` 以检查它是否正常工作。

## 它是如何工作的...

当控制器类被实例化时，Spring 会自动将现有的 `UserService` Bean 初始化到 `@Autowired` 字段中。这被称为依赖注入；控制器类只需声明其依赖项，即一个 `UserService` 字段。是 Spring 通过向其中注入一个 `UserService` 对象来初始化这个字段的。

如果 Spring 无法找到该依赖项的现有 bean，则会抛出异常。

## 更多...

可以设置要使用的 bean 的名称：

```java
@Autowired("myUserService")
UserService userService;
```

当使用接口时，依赖注入非常有用。例如，我们可以用`UserService`接口及其实现`UserServiceImpl`来替换我们的`UserService`类。一切都会按原样工作，除了现在可以简单地用另一个类（例如，为了单元测试目的）替换`UserServiceImpl`。

# 直接使用 bean

可以通过将包含所有 bean 的 Spring 的`ApplicationContext`作为你的类的依赖项来直接从 Spring 获取 bean，而不是使用依赖注入。在这个配方中，我们将向控制器类注入一个现有的 bean。

## 准备工作

我们将使用*使用@Component 注解隐式定义 bean*配方中的代码，其中我们定义了一个`UserService`bean。

## 如何操作...

这里是获取和使用一个豆类直接的方法：

1.  在控制器类中，添加一个被`@Autowired`注解的`ApplicationContext`字段：

    ```java
    @Autowired
    private ApplicationContext applicationContext;
    ```

1.  在控制器方法中，使用`ApplicationContext`对象及其`getBean()`方法来检索`UserService`bean：

    ```java
    UserService userService = (UserService)applicationContext.getBean("userService");        
    ```

## 它是如何工作的...

当控制器类被实例化时，Spring 会自动用它的`ApplicationContext`对象初始化`@Autowired`字段。`ApplicationContext`对象引用了所有的 Spring beans，因此我们可以直接通过名称获取一个 bean。

## 更多...

可以通过类来获取一个 bean，而不需要知道它的名字。

```java
applicationContext.getBean(UserService.class);  
```

# 列出所有 bean

这在调试目的上可能很有用。

## 准备工作

我们将使用*使用@Component 注解隐式定义 bean*配方中的代码，其中我们定义了一个`UserService`bean。

## 如何操作...

这里是检索当前 Spring 的`ApplicationContext`对象中 bean 名称的步骤：

1.  在你的类中，添加一个被`@Autowired`注解的`ApplicationContext`字段：

    ```java
    @Autowired
    private ApplicationContext applicationContext;
    ```

1.  在该类的某个方法中，使用`ApplicationContext`及其`getBeanDefinitionNames()`方法来获取 bean 名称列表：

    ```java
    String[] beans = applicationContext.getBeanDefinitionNames();
    for (String bean : beans) {
      System.out.println(bean);
    }  
    ```

## 它是如何工作的...

当控制器类被实例化时，Spring 会自动用它的`ApplicationContext`对象初始化`@Autowired`字段。`ApplicationContext`对象引用了所有的 Spring beans，因此我们可以获取使用它的所有 bean 的列表。

## 更多...

要从其名称获取 bean 本身，请使用`getBean()`方法：

```java
applicationContext.getBean("aBeanName");
```

# 使用多个配置类

当配置类中有许多 bean 定义时，Spring 配置类可能会变得相当长。在这种情况下，将其拆分为多个类可能很方便。

## 准备工作

我们将使用*使用@Bean 注解显式定义 bean*配方中的代码。

## 如何操作...

这是添加第二个配置类的方法：

1.  创建一个新的配置类，例如，在`com.springcookbook.config`包中的`DatabaseConfig`：

    ```java
    @Configuration
    public class DatabaseConfig {
    …
    ```

1.  在`ServletInitializer`类中，在`getServletConfigClasses()`方法中添加`DatabaseConfig`类：

    ```java
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[]{AppConfig.class, DatabaseConfig.class};
    }
    ```

1.  将`Datasource`实体从`AppConfig`类移动到`DatabaseConfig`类。

## 还有更多...

如果你正在使用没有`ServletInitializer`类的 Spring 应用程序，你可以从你的主要配置类中包含其他配置类：

```java
@Configuration
@Import({ DatabaseConfig.class, SecurityConfig.class })
public class AppConfig {
…
}
```
