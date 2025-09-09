# 第十二章。使用面向方面的编程

在本章中，我们将介绍以下食谱：

+   创建一个 Spring AOP 方面类

+   使用环绕通知测量方法的执行时间

+   使用前置通知记录方法参数

+   使用后置返回通知记录方法的返回值

+   使用后置抛出通知记录异常

+   使用后置通知来清理资源

+   使用引入在运行时使一个类实现一个接口

+   设置方面的执行顺序

# 简介

**面向方面的编程**（**AOP**）是关于在程序正常执行流程的各个点上，在运行时插入和执行额外的代码片段。在 AOP 术语中，这些代码片段是被称为 **建议** 的方法，而包含它们的类被称为 **方面**。AOP 是面向对象编程的补充。

本章介绍 Spring AOP 框架，它使我们能够在 Spring Beans（控制器方法、服务方法等）的方法之前和之后执行建议。对于更广泛的 AOP 功能，**AspectJ** 是参考 Java AOP 框架，并且可以无缝集成到 Spring 中。然而，它更复杂，需要定制的编译过程。

在第一个食谱中，我们将创建一个方面类并配置 Spring 使用它。我们将在后续的食谱中使用这个方面类，我们将通过实际用例来介绍 Spring AOP 提供的不同类型的建议。

# 创建一个 Spring AOP 方面类

在本食谱中，我们将创建一个方面类并配置 Spring 使用它。我们将在后续的食谱中使用这个方面类及其配置代码。

## 如何操作...

创建方面类的步骤如下：

1.  在 `pom.xml` 中添加 AspectJ Weaver Maven 依赖：

    ```java
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.8.5</version>
    </dependency>
    ```

1.  为你的应用程序的方面创建一个 Java 包。例如，`com.springcookbook.aspect`。

1.  在你的方面包中，创建一个带有 `@Component` 和 `@Aspect` 注解的类：

    ```java
    @Component
    @Aspect
    public class Aspect1 {

    }
    ```

1.  在 Spring 配置中，添加 `@EnableAspectJAutoProxy` 和你的方面包到 `@ComponentScan`：

    ```java
    @Configuration
    @EnableAspectJAutoProxy
    @ComponentScan(basePackages = {"com.spring_cookbook.controllers", "com.spring_cookbook.aspect"})
    public class AppConfig {
    ...  
    }
    ```

## 它是如何工作的...

AspectJ Weaver Maven 依赖提供了方面注解，因此我们可以使用常规的 Java 类来定义方面。

在方面类中，`@Aspect` 声明该类为方面。`@Component` 允许它被 Spring 检测并实例化为一个 Bean。

在 Spring 配置中，我们将我们的方面包包含在 `@ComponentScan` 中，因此该包中的 `@Component` 类将被 Spring 检测并实例化为 Beans。Spring 配置中的 `@EnableAspectJAutoProxy` 将使 Spring 实际上使用方面并执行它们的建议。

# 使用环绕通知测量方法的执行时间

**环绕 advice**是 advice 中最强大的一种类型；它可以完全用不同的代码替换目标方法。在这个菜谱中，我们将只使用它来在目标方法前后执行一些额外的代码。使用前面的代码，我们将获取当前时间。使用后面的代码，我们将再次获取当前时间，并将其与之前的时间进行比较，以计算目标方法执行的总时间。我们的目标方法将是控制器包中控制器类的控制器方法。

## 准备工作

我们将使用在之前菜谱中定义的方面类，*创建 Spring AOP 方面类*。

## 如何做到这一点...

这里是测量控制器方法执行时间的步骤：

1.  在方面类中，创建一个带有`@Around`注解的 advice 方法，并将`ProceedingJoinPoint`作为参数：

    ```java
    @Around("execution(* com.spring_cookbook.controllers.*.*(..))")
    public Object doBasicProfiling(ProceedingJoinPoint joinPoint) throws Throwable {
    ...
    }
    ```

1.  在那个 advice 方法中，测量目标方法的执行时间：

    ```java
    Long t1 = System.currentTimeMillis();
    Object returnValue = joinPoint.proceed();
    Long t2 = System.currentTimeMillis();        
    Long executionTime = t2 - t1;
    ```

1.  在目标方法名称之前记录执行时间：

    ```java
    String className = joinPoint.getSignature().getDeclaringTypeName();
    String methodName = joinPoint.getSignature().getName();
    System.out.println(className + "." + methodName + "() took " + executionTime + " ms");
    ```

1.  返回目标方法的返回值：

    ```java
    return returnValue;
    ```

1.  要测试 advice，你可以使用一个故意耗时的控制器方法：

    ```java
    @RequestMapping("user_list")
    @ResponseBody
    public void userList() throws Exception {
      try {
          Thread.sleep(2500);  // wait 2.5 seconds
      } catch(InterruptedException ex) {
          Thread.currentThread().interrupt();
      }
    }
    ```

1.  测试它是否工作。当你在浏览器中访问`/user_list`时，你应该在你的服务器日志中看到以下内容：

    ```java
    com.spring_cookbook.controllers.UserController.userList() took 2563 ms
    ```

## 它是如何工作的...

在 advice 方法之前的前置`@Around`注解是一个 pointcut 表达式：

```java
@Around("execution(* com.spring_cookbook.controllers.*.*(..))")
```

一个 pointcut 表达式确定目标方法（advice 将要应用的方法）。它就像一个正则表达式。在这里，它匹配所有控制器方法。具体来说：

+   `execution()`表示我们正在针对方法执行

+   第一个星号表示*任何返回类型*

+   第二个星号表示*任何类*（来自`com.spring_cookbook.controllers`包）

+   第三个星号表示*任何方法*

+   `(..)`表示*任何类型和数量的方法参数*

`joinPoint.proceed()`指令执行目标方法。跳过这个指令将跳过目标方法的执行。**连接点**是另一个 AOP 术语。它是在程序执行流程中可以执行 advice 的时刻。使用 Spring AOP，连接点始终指定目标方法。为了总结，advice 方法应用于不同的连接点，这些连接点由 pointcut 表达式识别。

我们还使用`joinPoint`对象来获取当前目标方法的名称：

```java
String className = joinPoint.getSignature().getDeclaringTypeName();
String methodName = joinPoint.getSignature().getName();
```

# 使用前置 advice 记录方法参数

**前置 advice**在目标方法执行之前执行一些额外的代码。在这个菜谱中，我们将记录目标方法的参数。

## 准备工作

我们将使用在*创建 Spring AOP 方面类*菜谱中定义的方面类。

## 如何做到这一点...

这里是使用前置 advice 记录方法参数的步骤：

1.  在你的方面类中，创建一个带有`@Before`注解的 advice 方法，并将`JoinPoint`作为参数：

    ```java
    @Before("execution(* com.spring_cookbook.controllers.*.*(..))")
    public void logArguments(JoinPoint joinPoint) {
    ...
    }
    ```

1.  在那个方法中，获取目标方法的参数列表：

    ```java
    Object[] arguments = joinPoint.getArgs();
    ```

1.  在目标方法名称之前记录参数列表：

    ```java
    String className = joinPoint.getSignature().getDeclaringTypeName();
    String methodName = joinPoint.getSignature().getName();
    System.out.println("-----" + className + "." + methodName + "() -----");

    for (int i = 0; i < arguments.length; i++) {
      System.out.println(arguments[i]);
    }
    ```

1.  使用带有参数的控制器方法测试 advice：

    ```java
    @RequestMapping("user_list")
    @ResponseBody
    public String userList(Locale locale, WebRequest request) {
    ...
    }
    ```

1.  检查是否正常工作。当你在浏览器中访问`/user_list`时，你应该在你的服务器日志中看到以下内容：

    ```java
    -----
    com.spring_cookbook.controllers.UserController.userList()
    -----
    en_US
    ServletWebRequest:
    uri=/spring_webapp/user_list;client=10.0.2.2
    ```

## 它是如何工作的…

建议方法之前的`@Before`注解是一个切入点表达式：

```java
@Before("execution(* com.spring_cookbook.controllers.*.*(..))")
```

参考更多细节，请参阅*使用环绕通知测量方法执行时间*配方。

`joinPoint.getArgs()`指令检索目标方法的参数值。

# 使用 after-returning advice 记录方法的返回值

**after-returning advice**在目标方法成功执行之后执行一些额外的代码。在这个配方中，我们将记录目标方法的返回值。

## 准备工作

我们将使用在*创建 Spring AOP 方面类*配方中定义的方面类。

## 如何操作…

这里是使用 after-returning advice 记录方法返回值的步骤：

1.  在你的方面类中，创建一个带有`@AfterReturning`注解的建议方法。让它接受一个`JoinPoint`对象和目标方法的返回值作为参数：

    ```java
    @AfterReturning(pointcut="execution(* com.spring_cookbook.controllers.*.*(..))", returning="returnValue")
    public void logReturnValue(JoinPoint joinPoint, Object returnValue) {
    ...
    }
    ```

1.  在那个建议方法中，记录目标方法名称之前返回的值：

    ```java
    String className = joinPoint.getSignature().getDeclaringTypeName();
    String methodName = joinPoint.getSignature().getName();
    System.out.println("-----" + className + "." + methodName + "() -----");
    System.out.println("returnValue=" + returnValue);
    ```

1.  使用返回值的控制器方法测试这个建议：

    ```java
    @RequestMapping("user_list")
    @ResponseBody
    public String userList() {
      return "just a test";
    }
    ```

1.  检查是否正常工作。当你在浏览器中访问`/user_list`时，你应该在你的服务器日志中看到以下内容：

    ```java
    -----
    com.spring_cookbook.controllers.UserController.userList()
    -----
    returnValue=just a test
    ```

## 它是如何工作的…

建议方法之前的`@AfterReturning`注解是一个切入点表达式：

```java
@AfterReturning(pointcut="execution(* com.spring_cookbook.controllers.*.*(..))", returning="returnValue")
```

参考更多细节，请参阅*使用环绕通知测量方法执行时间*配方。`returning`属性是用于建议方法的返回值的参数名称。

### 注意

注意，如果在目标方法执行过程中抛出异常，则不会执行 after-returning advice。

# 使用 after-throwing advice 记录异常

**after-throwing advice**在目标方法执行过程中抛出异常时执行一些额外的代码。在这个配方中，我们只记录异常。

## 准备工作

我们将使用在*创建 Spring AOP 方面类*配方中定义的方面类。

## 如何操作…

这里是使用 after-throwing advice 记录异常的步骤：

1.  在你的方面类中，创建一个带有`@AfterThrowing`注解的建议方法。让它接受一个`JoinPoint`对象和一个`Exception`对象作为参数：

    ```java
    @AfterThrowing(pointcut="execution(* com.spring_cookbook.controllers.*.*(..))", throwing="exception")
    public void logException(JoinPoint joinPoint, Exception exception) {
    ...
    }
    ```

1.  在那个建议方法中，记录目标方法名称之前记录的异常：

    ```java
    String className = joinPoint.getSignature().getDeclaringTypeName();
    String methodName = joinPoint.getSignature().getName();
    System.out.println("-----" + className + "." + methodName + "() -----");
    System.out.println("exception message:" + exception.getMessage());
    ```

1.  使用抛出异常的控制器方法测试这个建议：

    ```java
    @RequestMapping("user_list")
    @ResponseBody
    public String userList() throws Exception  {
      throw new Exception("a bad exception");
    }
    ```

1.  检查是否正常工作。当你在浏览器中访问`/user_list`时，你应该在你的服务器日志中看到以下内容：

    ```java
    -----
    com.spring_cookbook.controllers.UserController.userList()
    -----
    exception message:a bad exception
    ```

## 它是如何工作的…

建议方法之前的`@AfterThrowing`注解是一个切入点表达式：

```java
@AfterThrowing(pointcut="execution(* com.spring_cookbook.controllers.*.*(..))", throwing="exception")
```

参考更多细节，请参阅*使用环绕通知测量方法执行时间*配方。`throwing`属性是用于目标方法抛出的异常对象的建议方法的参数名称。

### 注意

注意，如果在目标方法的执行过程中没有抛出异常，则不会执行 after-throwing 通知。

# 使用后置通知来清理资源

**后置通知**在目标方法执行之后执行一些额外的代码，即使在其执行过程中抛出了异常。使用此通知通过删除临时文件或关闭数据库连接来清理资源。在这个配方中，我们只是记录目标方法名称。

## 准备工作

我们将使用在 *创建一个 Spring AOP 方面类* 烹饪配方中定义的方面类。

## 如何做到这一点…

使用后置通知的步骤如下：

1.  在你的方面类中，创建一个带有 `@After` 注解的通知方法。让它接受 `JoinPoint` 作为参数：

    ```java
    @After("execution(* com.spring_cookbook.controllers.*.*(..))")
    public void cleanUp(JoinPoint joinPoint) {
    ...
    }
    ```

1.  在那个通知方法中，记录目标方法名称：

    ```java
    String className = joinPoint.getSignature().getDeclaringTypeName();
    String methodName = joinPoint.getSignature().getName();
    System.out.println("-----" + className + "." + methodName + "() -----");
    ```

1.  使用两个控制器方法测试通知：一个正常执行，另一个抛出异常：

    ```java
    @RequestMapping("user_list")
    @ResponseBody
    public String userList() {
      return "method returning normally";
    }

    @RequestMapping("user_list2")
    @ResponseBody
    public String userList2() throws Exception  {
      throw new Exception("just a test");
    }
    ```

1.  检查它是否正在工作。当你在浏览器中访问 `/user_list` 或 `/user_list2` 时，你应该在你的服务器日志中看到以下内容：

    ```java
    -----
    com.spring_cookbook.controllers.UserController.userList()
    -----
    ```

## 它是如何工作的…

在通知方法之前的前置 `@After` 注解是一个切入点表达式：

```java
@After("execution(* com.spring_cookbook.controllers.*.*(..))")
```

有关更多详细信息，请参阅 *使用环绕通知测量方法执行时间* 烹饪配方。

# 使用引入在运行时让一个类实现一个接口

**引入**允许我们在运行时让一个 Java 类（我们将称之为 *目标类*）实现一个接口。使用 Spring AOP，引入只能应用于 Spring Bean（控制器、服务等等）。在这个配方中，我们将创建一个接口、它的实现，并使用这个实现让一个 Spring 控制器在运行时实现该接口。为了检查它是否工作，我们还将向控制器方法添加一个前置通知来执行接口实现中的方法。

## 准备工作

我们将使用在 *创建一个 Spring AOP 方面类* 烹饪配方中定义的方面类。

## 如何做到这一点…

使用引入的步骤如下：

1.  创建 `Logging` 接口：

    ```java
    public interface Logging {
      public void log(String str);
    }
    ```

1.  为它创建一个实现类：

    ```java
    public class LoggingConsole implements Logging {

      public void log(String str) {
        System.out.println(str);
      }
    }
    ```

1.  在你的方面类中，添加一个带有 `@DeclareParents` 注解的 `Logging` 属性。将实现类添加到 `@DeclareParents`：

    ```java
    @DeclareParents(value = "com.spring_cookbook.controllers.*+", defaultImpl = LoggingConsole.class)
    public static Logging mixin;
    ```

1.  添加一个带有 `@Before` 注解的通知方法。让它接受一个 `Logging` 对象作为参数：

    ```java
    @Before("execution(* com.spring_cookbook.controllers.*.*(..)) && this(logging)")
    public void logControllerMethod(Logging logging) {
    ...
    }
    ```

1.  在通知方法中，使用 `Logging` 对象：

    ```java
    logging.log("this is displayed just before a controller method is executed.");
    ```

1.  使用标准控制器方法测试它是否工作：

    ```java
    @RequestMapping("user_list")
    @ResponseBody
    public String userList() {
      return "method returning normally";
    }
    ```

1.  检查它是否正在工作。当你在浏览器中访问 `/user_list` 时，你应该在你的服务器日志中看到以下内容：

    ```java
    this is displayed just before a controller method is executed.
    ```

## 它是如何工作的…

在方面类中，`@DeclareParents` 注解之前的前置 `Logging` 属性是一个切入点表达式：

```java
@DeclareParents(value = "com.spring_cookbook.controllers.*+", defaultImpl = LoggingConsole.class)
```

这个切入点表达式和 `Logging` 属性定义了以下内容：

+   引入将应用于所有控制器类：`com.spring_cookbook.controllers.*+`

+   引入将使这些控制器类实现 Logging 接口：`public static Logging mixin;`

+   介绍将使这些控制器类使用`LoggingConsole`作为 Logging 接口的实现：`defaultImpl = LoggingConsole.class`

在*使用环绕建议测量方法执行时间*菜谱中，前置建议工作方式相同。它只需要一个额外的条件：

```java
this(logging)
```

这意味着建议将仅应用于实现 Logging 接口的对象。

# 设置方面的执行顺序

当使用多个方面类时，可能需要设置方面执行的顺序。在这个菜谱中，我们将使用两个带有针对控制器方法的前置建议的方面类。

## 准备工作

我们将使用来自*创建 Spring AOP 方面类*菜谱的配置。

我们将使用包含建议并执行时记录一些文本的两个方面类：

```java
@Component
@Aspect
public class Aspect1 {

  @Before("execution(* com.spring_cookbook.controllers.*.*(..))")
  public void advice1() {  
    System.out.println("advice1");
  }

}

@Component
@Aspect
public class Aspect2 {

  @Before("execution(* com.spring_cookbook.controllers.*.*(..))")
  public void advice2() {  
    System.out.println("advice2");
  }

}
```

## 如何操作...

设置两个方面类执行顺序的步骤如下：

1.  首个方面添加带有数字参数的`@Order`：

    ```java
    @Component
    @Aspect
    @Order(1)
    public class Aspect1 {
    ```

1.  第二个方面添加带有另一个数字参数的`@Order`：

    ```java
    @Component
    @Aspect
    @Order(2)
    public class Aspect2 {
    ```

1.  测试是否正常工作。当你在浏览器中访问`/user_list`时，你应该在你的服务器日志中看到以下内容：

    ```java
    advice1
    advice2
    ```

1.  交换`@Order`数字并检查执行顺序是否改变：

    ```java
    advice2
    advice1
    ```

## 它是如何工作的...

方面是按照`@Order`设置的升序执行的。

## 还有更多…

在同一方面的类中，无法设置建议方法的顺序。如果变得有必要，为这些建议创建新的方面类。
