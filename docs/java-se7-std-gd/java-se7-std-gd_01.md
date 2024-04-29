# 第一章：Java 入门

本章介绍了 Java 的基本元素以及如何编写简单的 Java 程序。通过简单的解释应用程序开发过程，可以全面了解 Java 开发环境。提供了一个作为起点和讨论参考的 Java 控制台程序。

在本章中，我们将研究：

+   Java 是什么

+   面向对象的开发过程

+   Java 应用程序的类型

+   创建一个简单的程序

+   类和接口的定义

+   Java 应用程序开发

+   Java 环境

+   Java 文档技术

+   Java 中的注释使用

+   核心 Java 包

# 理解 Java 作为一种技术

Sun Microsystems 在 1990 年代中期开发了该语言的原始规范。Patrick Naughton、Mike Sheridan 和 James Gosling 是 Java 的原始发明者，该语言最初被称为 Oak。

Java 是一种完整的面向对象的编程语言。它是平台无关的，通常是解释性的，而不是像 C/C++那样编译性的。它在语法和结构上模仿了 C/C++，并执行各种编译时和运行时的检查操作。Java 执行自动内存管理，有助于大大减少其他语言和动态分配内存的库中发现的内存泄漏问题。

Java 支持许多功能，在其概念产生时，其他语言中并没有直接找到。这些功能包括线程、网络、安全和图形用户界面（GUI）开发。其他语言可以用来支持这些功能，但它们没有像 Java 那样集成在语言中。

Java 使用独立的字节码，这是与体系结构无关的。也就是说，它被设计为与机器无关。字节码由 Java 虚拟机（JVM）解释和执行。正如我们将在第三章 *决策结构*中看到的那样，它的所有原始数据类型都是完全指定的。Java 开发工具包（JDK）的各个版本和其他重要时刻如下时间线图所示：

![理解 Java 作为一种技术](img/7324_01_01.jpg)

## 面向对象的软件开发

让我们暂时离题一下，考虑为什么我们要使用 Java。Java 最重要的一个方面是它是一种面向对象（OO）语言。OO 技术是一种流行的开发应用程序的范式。这种方法围绕一系列真实世界的对象建模应用程序，比如员工或船只。为了解决问题，有必要考虑构成问题域的真实世界对象。

OO 方法基于三个不同的活动：

+   面向对象分析（OOA）：这涉及确定系统的功能，即应用程序应该做什么

+   面向对象设计（OOD）：这涉及架构如何支持应用程序的功能

+   面向对象编程（OOP）：这涉及应用程序的实际实现

分析和设计步骤的产物通常被称为分析和设计工件。虽然可能会产生几种不同类型的工件，但对 OOP 步骤最感兴趣的是称为类图的工件。下图显示了一个部分类 UML 图，描述了两个类：`Customer`和`CustomerDriver`。在*一个简单的 Java 应用程序*部分，我们将研究这些类的代码。统一建模语言（UML）是一种广泛使用的 OO 技术，用于设计和记录应用程序。类图是该技术的最终产品之一，程序员用它来创建应用程序：

![面向对象软件开发](img/7324_01_02.jpg)

每个方框代表一个类，分为三个部分：

+   方框顶部的第一部分是类的名称

+   第二部分列出了构成类的变量

+   最后一部分列出了类的方法

在变量和方法名称之前的符号指定了这些类成员的可见性。以下是类图中使用的符号：

+   `-`: 私有

+   `+`: 公共

+   `#`: 受保护的（与继承一起使用）

通常，类图由许多类组成，并通过带注释的线条相互连接，显示类之间的关系。

类图旨在清楚地显示系统中包含哪些对象以及它们如何相互作用。一旦类图完成，就可以使用 Java 等面向对象编程语言来实现它。

### 注意

面向对象的方法通常用于中等规模到大规模的项目，其中许多开发人员必须进行沟通和合作，以创建一个应用程序。对于只涉及少数程序员的较小项目，例如大多数编程课程中处理的项目，通常不使用面向对象的方法。

## 面向对象编程原则

虽然关于什么才真正使编程语言成为面向对象的编程语言存在一些分歧，但通常面向对象编程语言必须支持三个基本原则：

+   数据封装

+   继承

+   多态性

**数据封装**关注于从类的用户那里隐藏不相关的信息，并暴露相关的信息。数据封装的主要目的是降低软件开发的复杂性。通过隐藏执行操作所需的细节，使用该操作变得更简单。如何在 Java 中实现数据封装将在本章后面的*访问修饰符*部分中解释。

数据封装也用于保护对象的内部状态。通过隐藏表示对象状态的变量，可以通过方法来控制对对象的修改。方法中的代码验证状态的任何更改。此外，通过隐藏变量，消除了类之间信息的共享。这减少了应用程序中可能出现的耦合量。

继承描述了两个类之间的关系，使一个类重用另一个类的功能。这样可以实现软件的重用，从而提高开发人员的生产力。继承在第七章中有详细介绍，*继承和多态性*。

第三个原则是多态性，其主要关注点是使应用程序更易于维护和扩展。多态行为是指一个或多个相同方法的行为取决于执行该方法的对象。例如，`person`对象和`square`对象都可以有一个`draw`方法。它所绘制的内容取决于执行该方法的对象。多态性在第七章中讨论，*继承和多态性*。

这些原则总结在以下表中：

| 原则 | 是什么 | 为什么使用它 | 如何做到 |
| --- | --- | --- | --- |
| 数据封装 | 从类的用户隐藏信息的技术 | 降低软件开发复杂性的级别 | 使用`public`、`private`和`protected`等访问修饰符 |
| 继承 | 允许派生或子类使用基类或父类的部分的技术 | 促进软件的重用 | 使用`extends`关键字 |
| 多态性 | 支持方法的不同行为，取决于执行该方法的对象 | 使应用程序更易于维护 | Java 语言的固有特性 |

`implements`关键字用于支持多态行为，如第七章*继承和多态*中所解释的。

## 检查 Java 应用程序的类型

有几种类型的 Java 应用程序。这些类型使 Java 得以在许多不同领域蓬勃发展，并促使 Java 成为一种非常流行的编程语言。Java 用于开发以下内容：

+   控制台和窗口应用程序

+   由 Servlet、JSP、JSF 和其他 JEE 标准支持的基于服务器的 Web 应用程序

+   在浏览器中执行的小程序

+   嵌入式应用程序

+   使用 JavaBeans 的组件化构建块

虽然对 Java 应用程序类型的基本理解有助于将 Java 置于上下文中，但也有助于能够识别这些应用程序的基本代码。您可能不完全理解这些应用程序类型的所有细节，但看到简单的代码示例是有用的。

阅读代码对于理解一种语言和特定程序有很大帮助。在整本书中，我们将使用许多示例来说明和解释 Java 的各个方面。以下通过呈现对应用程序类型至关重要的简短代码片段来展示 Java 应用程序的基本类型。

一个简单的控制台应用程序由一个带有`main`方法的单个类组成，如下面的代码片段所示：

```java
public class Application {
   public static void main(String[] args) {
      // Body of method
   }
}
```

我们将更深入地研究这种类型的应用程序。

小程序通常嵌入在 HTML 页面中，并提供了一种实现客户端执行代码的方法。它没有`main`方法，而是使用浏览器用来管理应用程序的一系列回调方法。以下代码提供了小程序的一般外观：

```java
import java.applet.*;
import java.awt.Graphics;

public class SimpleApplet extends Applet {

   @Override
   public void init() {
      // Initialization code
   }

   @Override
   public void paint( Graphics g ) {
      // Display graphics
   }
}
```

`@Override`注解用于确保接下来的方法实际上是被覆盖的。这在本章的*注解*部分中有更详细的讨论。

Servlet 是一个在服务器端运行的应用程序，它呈现给客户端一个 HTML 页面。`doGet`或`doPut`方法响应客户端请求。以下示例中的`out`变量代表 HTML 页面。`println`方法用于编写 HTML 代码，如下面的代码片段所示：

```java
class Application extends HttpServlet {
   public void doGet(HttpServletRequest req,
            HttpServletResponse res)
            throws ServletException, IOException {
      res.setContentType("text/html");

      // then get the writer and write the response data
      PrintWriter out = res.getWriter();
      out.println(
         "<HEAD><TITLE> Simple Servlet</TITLE></HEAD><BODY>");
      out.println("<h1> Hello World! </h1>");
      out.println(
         "<P>This is output is from a Simple Servlet.");
      out.println("</BODY>");
      out.close();
   }
}
```

JavaServer Page（JSP）实际上是一个伪装的 Servlet。它提供了一种更方便的开发网页的方式。以下示例使用一个 JavaBean 在网页上显示“Hello World”。JavaBean 在以下示例中有详细说明：

```java
<html>
<head>
   <title>A Simple JSP Page</title>
</head>
<body>
Hello World!<br/>

<%
   // This is a scriptlet that can contain Java code
%>
<hr>
<jsp:useBean id="namebean" class="packt.NameBean" scope="session" >
<jsp:setProperty name="namebean" property="name" value=" Hello world"" />
</jsp:useBean>
<h1> <jsp:getProperty name="namebean" property="name" /></h1>
</body>
</html>
```

JavaBean 是共享应用程序功能的构建块。它们经常被设计用于多个应用程序，并遵循标准的命名约定。以下是一个简单的 JavaBean，用于保存一个名称（它在前面的 JSP 页面中使用）：

```java
package packt;
public class NameBean {

  private String name= "Default Name"";

  public String getName() {
     return this.name;
  }
  public void setName(String name) {
     this.name = name;
  }
}
```

企业 JavaBean（EJB）是设计用于在 Web 服务器上的客户端/服务器配置中使用的组件。这是一个相当专业化的主题，与认证的副级别无关。

还有其他几种 Java 技术，如 JSF 和 Facelets，它们是 JEE 的一部分。这些是对用于开发网页的旧 Servlet 和 JSP 技术的改进。

在本书中，我们只会使用简单的 Java 控制台应用程序。这种类型的应用程序已经足够解释 Java 的本质。

# 探索 Java 控制台程序的结构

让我们从一个简单的 Java 程序开始，然后使用它来探索 Java 的许多基本方面。首先，Java 应用程序由一个或多个文件组成，这些文件位于文件系统的某个位置。文件的名称和位置都很重要，我们很快就会看到。

### 提示

您可以从您在[`www.PacktPub.com`](http://www.PacktPub.com)的帐户中购买的所有 Packt 图书的示例代码文件。如果您在其他地方购买了本书，您可以访问[`www.PacktPub.com/support`](http://www.PacktPub.com/support)并注册，以便直接通过电子邮件接收文件。

## 一个简单的 Java 应用程序

我们的简单程序定义了一个`Customer`类，然后在`CustomerDriver`类中使用它，如下所示：

```java
package com.company.customer;

import java.math.BigDecimal;
import java.util.Locale;

public class Customer {
  private String name;
  private int accountNumber;
  private Locale locale;
  private BigDecimal balance;

  public Customer() {
    this.name = "Default Customer";
    this.accountNumber = 12345;
    this.locale = Locale.ITALY;
    this.balance = new BigDecimal("0");
  }

  public String getName() {
    return name;
  }
  public void setName(String name) throws Exception {
    if(name == null) {
         throw new IllegalArgumentException(
            "Names must not be null");
    } else {
      this.name = name;
    }
  }
  public int getAccountNumber() {
    return accountNumber;
  }

  public void setAccountNumber(int accountNumber) {
    this.accountNumber = accountNumber;
  }

  public BigDecimal getBalance() {
    return balance;
  }

  public void setBalance(float balance) {
    this.balance = new BigDecimal(balance);
  }

   public String toString() {
      java.text.NumberFormat format =
         java.text.NumberFormat.getCurrencyInstance(locale);
      StringBuilder value = new StringBuilder();
      value.append(String.format("Name: %s%n", this.name));
      value.append(String.format("Account Number: %d%n", 
            this.accountNumber));
      value.append(String.format("Balance: %s%n",
            format.format(this.balance)));
      return value.toString();
    }  
}

package com.company.customer;

public class CustomerDriver {

  public static void main(String[] args) {
      // Define a reference and creates a new Customer object
    Customer customer;      
    customer = new Customer();
    customer.setBalance(12506.45f);
    System.out.println(customer.toString());
  }
```

如何编译和执行此应用程序的详细信息在*在没有 IDE 的情况下开发 Java 应用程序*部分提供。执行此应用程序时，您将获得以下输出：

```java
Name: Default Customer
Account number: 12345
Balance: € 12.506,45

```

详细了解应用程序 以下部分详细介绍了示例程序的重要方面。这些将在接下来的章节中更详细地阐述。请注意，此应用程序中有两个类。`CustomerDriver`类包含`main`方法，并首先执行。在`main`方法中创建并使用了`Customer`类的一个实例。

### 包

包语句指定了类的`com.company.customer`包。包提供了一种将相似的类、接口、枚举和异常分组的方法。它们在第九章的*包*部分中更深入地讨论了*Java 应用程序*。

### 导入

`import`语句指示类使用了哪些包和类。这允许编译器确定包的成员是否被正确使用。所有类都需要导入包，但以下类除外：

+   在`java.lang`包中找到

+   位于当前包（在本例中为`com.company.customer`）

+   显式标记，如在`Customer`类的`toString`方法中使用的`java.text.NumberFormat`

### 注意

`import`语句通知编译器应用程序使用了哪些包和类以及如何使用它们。

### Customer 类

类定义的第一个单词是关键字`public`，这是 Java 为面向对象软件开发提供的支持的一部分。在这种情况下，它指定类在包外可见。虽然不是必需的，但大多数类经常使用它，并引出了第二个关键字`class`，它标识了一个 Java 类。

### 实例变量

接下来声明了四个私有实例变量。使用`private`关键字将它们隐藏在类的用户之外。`Locale`类支持可以在国际上透明工作的应用程序。`BigDecimal`是在 Java 中表示货币的最佳方式。

### 方法

通过将这些实例变量设为私有，设计者限制了对变量的访问。然后只能通过公共方法访问它们。私有变量和公共方法的组合是数据封装的一个例子。如果将实例变量改为公共的，其他用户可以直接访问变量。这将提高程序的效率，但可能会阻碍未来的维护工作。更改这些变量并对其进行任何验证检查将更加困难。

一系列的 getter 和 setter 方法用于返回和设置与私有实例变量相关的值。这以受控的方式暴露它们。使用 getter 和 setter 方法是实现封装的标准方法。例如，尝试将空值分配给名称将引发`IllegalArmumentException`异常。这些类型的方法在*方法声明*部分中讨论。

`toString`方法返回表示客户实例的字符串。在这种情况下，返回名称、帐号和余额的本地化版本。`StringBuilder`类的使用在第二章中讨论，*Java 数据类型及其使用*。

### 注意

方法在类中找到，类在包中找到。

### CustomerDriver 类的 main 方法

`CustomerDriver`类被称为驱动程序或控制器类。它的目的是拥有一个将创建和使用其他类的`main`方法。

在 Java 应用程序中，`main`方法是要执行的第一个方法。如果应用程序由多个类组成，通常只有一个类有`main`方法。Java 应用程序通常只需要一个`main`方法。

在`main`方法中，创建一个新的客户，设置余额，然后显示客户。在语句中添加了 C++风格的注释，以记录客户的声明和创建。这是以双斜杠（`//`）开头的行。注释在*注释*部分详细解释。

### 注意

在 Java 控制台应用程序中执行的第一种方法是`main`方法。

# 探索类的结构

编程可以被认为是代码操作数据。在 Java 中，代码围绕以下内容组织：

+   包

+   类

+   方法

包是具有类似功能的类的集合。类由支持类功能的方法组成。这种组织为应用程序提供了结构。类将始终在一个包中，方法将始终在一个类中。

### 注意

如果类定义中没有包语句，则该类将成为默认包的一部分，该默认包包括同一目录中没有包语句的所有类。

## 类、接口和对象

类是面向对象程序的基本构建块。它通常代表现实世界的对象。Java 中的类定义包括成员变量声明和方法声明。它以`class`关键字开始。类的主体用大括号括起来，包含所有实例变量和方法：

```java
  class classname {
    // define class level variables
    // define methods
  }
```

### 注意

一对开放和关闭大括号构成一个块语句。这在 Java 的许多其他部分中使用。

### 类和对象

类是用于创建具有相似特征的多个对象的模式或模板。它定义了类的变量和方法。它声明了类的功能。但是，在使用这些功能之前，必须创建一个对象。对象是类的实例化。也就是说，对象由为类的成员变量分配的内存组成。每个对象都有自己的一组成员变量。

### 提示

创建新对象时发生以下情况：

+   使用 new 关键字创建类的实例

+   为类的新实例物理分配内存

+   执行任何静态初始化程序（如第六章中*Java 初始化顺序*部分所述），*类、构造函数和方法*）

+   调用构造函数进行初始化

+   返回对对象的引用

对象的状态通常对对象的用户隐藏，并反映在其实例变量的值中。对象的行为由它拥有的方法决定。这是数据封装的一个例子。

### 注意

对象是类的实例化。每个类的实例都有自己独特的一组实例变量。

Java 中的对象总是分配在堆上。堆是用于动态分配内存（如对象）的内存区域。在 Java 中，对象在程序中分配，然后由 JVM 释放。这种内存释放称为垃圾回收，由 JVM 自动执行。应用程序对此过程的控制很少。这种技术的主要好处是最大限度地减少内存泄漏。

### 注意

当动态分配内存但从未释放时，就会发生内存泄漏。这在诸如 C 和 C++等语言中是一个常见问题，程序员有责任管理堆。

在 Java 中，如果分配了一个对象但在不再需要该对象时没有释放对该对象的引用，就可能发生内存泄漏。

### 构造函数

构造函数用于初始化对象。每当创建一个对象时，都会执行构造函数。默认构造函数是没有参数的构造函数，对所有类都会自动提供。这个构造函数将把所有实例变量初始化为默认值。

然而，如果开发人员提供了构造函数，编译器就不会再添加默认构造函数。开发人员需要显式添加一个默认构造函数。始终具有一个默认的无参数构造函数是一个良好的实践。

### 接口

接口类似于抽象类。它使用`interface`关键字声明，只包含抽象方法和最终变量。抽象类通常有一个或多个抽象方法。抽象方法是没有实现的方法。它旨在支持多态行为，如第七章中讨论的，*继承和多态*。以下代码定义了一个用于指定类能够被绘制的接口：

```java
  interface Drawable {
    final int unit = 1;
    public void draw();
  }
```

## 方法

所有可执行代码都在初始化程序列表或方法中执行。在这里，我们将研究方法的定义和用法。初始化程序列表在第六章中讨论，*类，构造函数和方法*。方法将始终包含在类中。方法的可见性由其访问修饰符控制，详细信息请参阅*访问修饰符*部分。方法可以是静态的或实例的。在这里，我们将考虑实例方法。正如我们将在第六章中看到的，*类，构造函数和方法*，静态方法通常访问类的对象之间共享的静态变量。

无论方法的类型如何，方法只有一个副本。也就是说，虽然一个类可能有零个、一个或多个方法，但类的每个实例（对象）都使用方法的相同定义。

### 方法声明

一个典型的方法包括：

+   一个可选的修饰符

+   返回类型

+   方法名称

+   括在括号中的参数列表

+   可选的 throws 子句

+   包含方法语句的块语句

以下`setName`方法说明了方法的这些部分：

```java
  public void setName(String name) throws Exception {
    if(name == null) {
      throw new Exception("Names must not be null");
    } else {
      this.name = name;
    }
  }
```

虽然在这个例子中 else 子句在技术上不是必需的，但始终使用 else 子句是一个良好的实践，因为它代表了可能的执行顺序。在这个例子中，如果 if 语句的逻辑表达式求值为 true，那么异常将被抛出，方法的其余部分将被跳过。异常处理在第八章中有详细介绍，*应用程序中的异常处理*。

方法经常操作实例变量以定义对象的新状态。在设计良好的类中，实例变量通常只能由类的方法更改。它们对类是私有的。因此，实现了数据封装。

方法通常是可见的，并允许对象的用户操作该对象。有两种方法对方法进行分类：

+   **Getter 方法**：这些方法返回对象的状态（也称为**访问器方法**）

+   **Setter 方法**：这些方法可以改变对象的状态（也称为**变异方法**）

在`Customer`类中，为所有实例变量提供了 setter 和 getter 方法，除了 locale 变量。我们本可以很容易地为这个变量包括一个 get 和 set 方法，但为了节省空间，我们没有这样做。

### 注意

具有获取方法但没有其他可见的设置方法的变量被称为**只读成员变量**。类的设计者决定限制对变量的直接访问。

具有设置方法但没有其他可见的获取方法的变量被称为**只写成员变量**。虽然您可能会遇到这样的变量，但它们很少见。

### 方法签名

方法的签名由以下组成：

+   方法的名称

+   参数的数量

+   参数的类型

+   参数的顺序

签名是一个重要的概念，用于方法和构造函数的重载/覆盖，如第七章中所讨论的，*继承和多态*。构造函数也将有一个签名。请注意，签名的定义不包括返回类型。

### 主方法

书中使用的示例将是控制台程序应用。这些程序通常从键盘读取并在控制台上显示输出。当操作系统执行控制台应用程序时，首先执行`main`方法。然后可能执行其他方法。

`main`方法可以用于从命令行传递信息。这些信息传递给`main`方法的参数。它由代表程序参数的字符串数组组成。我们将在第四章中看到这一点，*使用数组和集合*。

在 Java 中只有一种`main`方法的形式，如下所示：

```java
    public static void main(String[] args) {
       // Body of method
    }
```

以下表格显示了`main`方法的元素：

| 元素 | 意义 |
| --- | --- |
| `public` | 方法在类外可见。 |
| `static` | 该方法可以在不创建类类型对象的情况下调用。 |
| `void` | 该方法不返回任何内容。 |
| `args` | 代表传递的参数的字符串数组。 |

#### 从应用程序返回一个值

`main`方法返回`void`，这意味着在正常的方法调用序列中无法将值返回给操作系统。但是，有时将返回一个值以指示程序是否成功终止是有用的。当程序用于批处理类型操作时，返回这些信息是有用的。如果在执行序列中一个程序失败，那么序列可能会被改变。可以使用`System.exit`方法从应用程序返回信息。以下方法的使用将终止应用程序并返回零给操作系统：

```java
    System.exit(0);
```

### 注意

`exit`方法：

+   强制终止应用程序的所有线程

+   是极端的，应该避免

+   不提供优雅终止程序的机会

## 访问修饰符

变量和方法可以声明为以下四种类型之一，如下表所示：

| 访问类型 | 关键字 | 意义 |
| --- | --- | --- |
| 公共的 | `public` | 提供给类外用户的访问。 |
| 私有的 | `private` | 限制对类成员的访问。 |
| 受保护的 | `protected` | 提供给继承类或同一包中成员的访问。 |
| 包范围 | 无 | 提供对同一包中成员的访问。 |

大多数情况下，成员变量声明为私有，方法声明为公共。但是，其他访问类型的存在意味着控制成员可见性的其他潜在方法。这些用法将在第七章*继承和多态*中进行检查。

在`Customer`类中，所有类变量都声明为私有，所有方法都声明为公共。在`CustomerDriver`类中，我们看到了`setBalance`和`toString`方法的使用：

```java
    customer.setBalance(12506.45f);
    System.out.println(customer.toString());
```

由于这些方法被声明为 public，它们可以与`Customer`对象一起使用。不可能直接访问 balance 实例变量。以下语句尝试这样做：

```java
    customer.balance = new BigDecimal(12506.45f);
```

编译器将发出类似以下的编译时错误：

**balance 在 com.company.customer.Customer 中具有私有访问权限**

### 注意

访问修饰符用于控制应用程序元素的可见性。

## 文档

程序的文档是软件开发过程中的重要部分。它向其他开发人员解释代码，并提醒开发人员他们为什么这样做。

文档是通过几种技术实现的。在这里，我们将讨论三种常见的技术：

+   **注释**：这是嵌入在应用程序中的文档

+   **命名约定**：遵循标准的 Java 命名约定可以使应用程序更易读

+   **Javadoc**：这是一种用于生成 HTML 文件形式的应用程序文档的工具

### 注释

注释用于记录程序。它们不可执行，编译器会忽略它们。良好的注释可以大大提高程序的可读性和可维护性。注释可以分为三种类型——C 样式、C++样式和 Java 样式，如下表所总结：

| 注释类型 | 描述 |
| --- | --- |
| 例子 |
| --- |
| C 样式 | C 样式注释在注释的开头和结尾使用两个字符序列。这种类型的注释可以跨越多行。开始字符序列是`/*`，而结束序列由`*/`组成。 |

|

```java
  /* A multi-line comment
     …
  */

  /* A single line comment */
```

|

| C++样式 | C++样式注释以两个斜杠开头，注释一直持续到行尾。实质上，从`//`到行尾的所有内容都被视为注释。 |
| --- | --- |

|

```java
  // The entire line is a comment
  int total;	// Comment used to clarify variable
  area = height*width; 	// This computes the area of a rectangle
```

|

| Java 样式 | Java 样式与 C 样式注释的语法相同，只是它以`/**`开头，而不是`/*`。此外，可以在 Java 样式注释中添加特殊标签以进行文档目的。一个名为`javadoc`的程序将读取使用这些类型注释的源文件，并生成一系列 HTML 文件来记录程序。有关更多详细信息，请参阅*使用 Javadocs*部分。 |
| --- | --- |

|

```java
    /**
     * This method computes the area of a rectangle
     *
     * @param height	The height of the rectangle
     * @param width	The width of the rectangle
     * @return		The method returns the area of a rectangle
     *
     */
   public int computeArea(int height, int width)  {
      return height * width;
   }
```

|

### Java 命名约定

Java 使用一系列命名约定来使程序更易读。建议您始终遵循这些命名约定。通过这样做：

+   使您的代码更易读

+   它支持 JavaBeans 的使用

### 注意

有关命名约定的更多细节，请访问[`www.oracle.com/technetwork/java/codeconvtoc-136057.html`](http://www.oracle.com/technetwork/java/codeconvtoc-136057.html)。

Java 命名约定的规则和示例显示在以下表中：

| 元素 | 约定 | 例子 |
| --- | --- | --- |
| 包 | 所有字母都小写。 | `com.company.customer` |
| 类 | 每个单词的第一个字母大写。 | `CustomerDriver` |
| 接口 | 每个单词的第一个字母大写。 | `Drawable` |
| 变量 | 第一个单词不大写，但后续单词大写 | `grandTotal` |
| 方法 | 第一个单词不大写，但后续单词大写。方法应该是动词。 | `computePay` |
| 常量 | 每个字母都大写。 | `LIMIT` |

### 注意

遵循 Java 命名约定对于保持程序可读性和支持 JavaBeans 很重要。

### 使用 Javadocs

Javadoc 工具基于源代码和源代码中嵌入的 Javadoc 标签生成一系列 HTML 文件。该工具也随 JDK 一起分发。虽然以下示例并不试图提供 Javadocs 的完整处理，但它应该能给你一个关于 Javadocs 能为你做什么的好主意：

```java
public class SuperMath {
   /**
    * Compute PI - Returns a value for PI.
    *    Able to compute pi to an infinite number of decimal 
    *    places in a single machine cycle.
    * @return A double number representing PI
   */

   public static double computePI() {
      //
   }
}
```

`javadoc`命令与此类一起使用时，会生成多个 HTML 文件。`index.html`文件的一部分如下截图所示：

![使用 Javadocs](img/7324_01_03.jpg)

### 注意

有关 Javadoc 文件的使用和创建的更多信息可以在[`www.oracle.com/technetwork/java/javase/documentation/index-137868.html`](http://www.oracle.com/technetwork/java/javase/documentation/index-137868.html)找到。

# 调查 Java 应用程序开发过程

Java 源代码被编译为中间字节码。然后在任何安装有**Java 虚拟机**（**JVM**）的平台上运行时解释这些字节码。然而，这个说法有些误导，因为 Java 技术通常会直接将字节码编译为机器码。已经有了许多即时编译器的改进，加快了 Java 应用程序的执行速度，通常会运行得几乎和本地编译的 C 或 C++应用程序一样快，有时甚至更快。

Java 源代码位于以`.java`扩展名结尾的文件中。Java 编译器将源代码编译为字节码表示，并将这些字节码存储在以`.class`扩展名结尾的文件中。

有几种**集成开发环境**（**IDE**）用于支持 Java 应用程序的开发。也可以使用**Java 开发工具包**（**JDK**）中的基本工具从命令行开发 Java 应用程序。

生产 Java 应用程序通常在一个平台上开发，然后部署到另一个平台。目标平台需要安装**Java 运行环境**（**JRE**）才能执行 Java 应用程序。有几种工具可以协助这个部署过程。通常，Java 应用程序会被压缩成一个**Java 存档**（**JAR**）文件，然后部署。JAR 文件只是一个嵌入有清单文档的 ZIP 文件。清单文档通常详细说明了正在创建的 JAR 文件的内容和类型。

## 编译 Java 应用程序

用于开发 Java 应用程序的一般步骤包括：

+   使用编辑器创建应用程序

+   使用 Java 编译器（`javac`）编译它

+   使用 Java 解释器（`java`）执行它

+   根据需要使用 Java 调试器可选择地调试应用程序

这个过程总结在下图中：

![编译 Java 应用程序](img/7324_01_04.jpg)

Java 源代码文件被编译为字节码文件。这些字节码文件具有**.class**扩展名。当 Java 包被分发时，源代码文件通常不会存储在与`.class`文件相同的位置。

## SDK 文件结构

**Java 软件开发工具包**（**SDK**）可下载并用于创建和执行许多类型的 Java 应用程序。**Java 企业版**（**JEE**）是一个不同的 SDK，用于开发以 Web 应用程序为特征的企业应用程序。该 SDK 也被称为**Java 2 企业版**（**J2EE**），你可能会看到它被引用为 J2EE。在这里，我们只处理 Java SDK。

虽然 SDK 分发的实际结构会因版本而异，但典型的 SDK 由一系列目录组成，如下所列：

+   `bin`：这包含用于开发 Java 应用程序的工具，包括编译器和 JVM

+   `db`：这是 Apache Derby 关系数据库

+   `demo`：这包含一系列演示应用程序

+   `include`：这包含用于与 C 应用程序交互的头文件

+   `jre`：这是 JDK 使用的 JRE

+   `sample`：这个目录包含 Java 各种特性的示例代码

SDK 可能包括核心类的实际源代码。这通常可以在位于`JAVA_HOME`根目录下的`src.zip`文件中找到。

## IDE 文件结构

每个 IDE 都有一种首选的组织应用程序文件的方式。这些组织方案并不总是固定的，但这里介绍的是常见的文件排列方式。

例如，在 Eclipse IDE 中，一个简单的应用程序由两个项目文件和三个子目录组成。这些文件和目录列举如下：

+   `.classpath`：这是包含与类路径相关信息的 XML 文件

+   `.project`：这是描述项目的 XML 文档

+   `.settings`：这是一个包含`org.eclipse.jdt.core.prefs`文件的目录，该文件指定了编译器的偏好设置。

+   `bin`：这个目录用于包含包文件结构和应用程序的类文件

+   `src`：这个目录用于包含包文件结构和应用程序的源文件

这种组织方案是由开发工具使用的。这些工具通常包括编辑器、编译器、链接器、调试器等。这些语言经常使用 Make 工具来确定需要编译或以其他方式处理的文件。

## 在没有 IDE 的情况下开发 Java 应用程序

在本节中，我们将演示如何在 Windows 平台上使用 Java 7 编译和执行 Java 应用程序。这种方法与其他操作系统的方法非常相似。

在我们编译和执行示例程序之前，我们需要：

+   安装 JDK

+   为应用程序创建适当的文件结构

+   创建用于保存我们的类的文件

JDK 的最新版本可以在[`www.oracle.com/technetwork/java/javase/downloads/index.html`](http://www.oracle.com/technetwork/java/javase/downloads/index.html)找到。下载并安装符合您需求的版本。注意安装位置，因为我们很快将会用到这些信息。

如前所述，Java 类必须位于特定的文件结构中，与其包名称相对应。在文件系统的某个地方创建一个文件结构，其中有一个名为`com`的顶级目录，该目录下有一个名为`company`的目录，然后在`company`目录下有一个名为`customer`的目录。

在`customer`目录中创建两个文件，分别命名为`Customer.java`和`CustomerDriver.java`。使用在*一个简单的 Java 应用程序*部分中找到的相应类。

JDK 工具位于 JDK 目录中。当安装 JDK 时，通常会设置环境变量以允许成功执行 JDK 工具。然而，需要指定这些工具的位置。这可以通过`set`命令来实现。在下面的命令中，我们将`path`环境变量设置为引用`C:\Program Files\Java\jdk1.7.0_02\bin`目录，这是本章撰写时的最新版本：

```java
set path= C:\Program Files\Java\jdk1.7.0_02\bin;%path%

```

这个命令在之前分配的路径前面加上了`bin`目录的路径。`path`环境变量被操作系统用来查找在命令提示符下执行的命令。没有这些信息，操作系统将不知道 JDK 命令的位置。

要使用 JDK 编译程序，导航到`com`目录的上一级目录。由于作为该应用程序一部分的类属于`com.company.customer`包，我们需要： 

+   在`javac`命令中指定路径

+   从`com`目录的上一级目录执行该命令

由于这个应用程序由两个文件组成，我们需要编译它们两个。可以使用以下两个单独的命令来完成：

```java
javac com.company.customer.Customer.java
javac com.company.customer.CustomerDriver.java

```

或者，可以使用单个命令和星号通配符来完成：

```java
javac com.company.customer.*.java

```

编译器的输出是一个名为`CustomerDriver.class`的字节码文件。要执行程序，使用 Java 解释器和你的类文件，如下命令所示。类扩展名不包括在内，如果包含在文件名中会导致错误：

```java
java com.company.customer.CustomerDriver

```

你的程序的输出应该如下：

```java
Name: Default Customer
Account number: 12345
Balance: € 12.506,45

```

## Java 环境

Java 环境是用于开发和执行 Java 应用程序的操作系统和文件结构。之前，我们已经检查了 JDK 的结构，这些都是 Java 环境的一部分。与这个环境相关的是一系列的环境变量，它们被用来在不同时间进行各种操作。在这里，我们将更详细地检查其中的一些：

+   `CLASSPATH`

+   `PATH`

+   `JAVA_VERSION`

+   `JAVA_HOME`

+   `OS_NAME`

+   `OS_VERSION`

+   `OS_ARCH`

这些变量在下表中总结：

| 名称 | 目的 | 示例 |
| --- | --- | --- |
| `CLASSPATH` | 指定类的根目录。 | `.;C:\Program Files (x86)\Java\jre7\lib\ext\QTJava.zip` |
| `PATH` | 命令的位置。 |   |
| `JAVA_VERSION` | 要使用的 Java 版本。 | `<param name="java_version" value="1.5.0_11">` |
| `JAVA_HOME` | Java 目录的位置。 | `C:\Program Files (x86)\Java\jre6\bin` |
| `OS_NAME` | 操作系统的名称。 | Windows 7 |
| `OS_VERSION` | 操作系统的版本 | 6.1 |
| `OS_ARCH` | 操作系统架构 | AMD64 |

`CLASSPATH`环境变量用于标识包的根目录。设置如下：

```java
 c:>set CLASSPATH=d:\development\increment1;%CLASSPATH%

```

`CLASSPATH`变量只需要设置非标准包。Java 编译器将始终隐式地将系统的类目录附加到`CLASSPATH`。默认的`CLASSPATH`是当前目录和系统的类目录。

与应用程序相关的还有许多其他环境变量。以下代码序列可用于显示这些变量的列表：

```java
    java.util.Properties properties = System.getProperties();
    properties.list(System.out);
```

这段代码序列的部分输出如下：

```java
-- listing properties --
java.runtime.name=Java(TM) SE Runtime Environment
sun.boot.library.path=C:\Program Files\Java\jre7\bin
java.vm.version=22.0-b10
java.vm.vendor=Oracle Corporation
java.vendor.url=http://java.oracle.com/
path.separator=;
java.vm.name=Java HotSpot(TM) 64-Bit Server VM
…

```

## 注解

注解提供关于程序的信息。这些信息不驻留在程序中，也不会影响其执行。注解用于支持诸如编译器和程序执行期间的工具。例如，`@Override`注解通知编译器一个方法正在覆盖基类的方法。如果该方法实际上没有覆盖基类的方法，因为拼写错误，编译器将生成一个错误。

注解应用于应用程序的元素，如类、方法或字段。它以 at 符号`@`开头，后面跟着注解的名称，可选地跟着一组括号括起来的值的列表。

常见的编译器注解在下表中详细说明：

| 注解 | 用法 |
| --- | --- |
| `@Deprecated` | 编译器用来指示不应该使用该元素 |
| `@Override` | 该方法覆盖了基类的方法 |
| `@SuppressWarnings` | 用于抑制特定的编译器警告 |

注解可以添加到应用程序中，并由第三方工具用于特定目的。在需要时也可以编写自己的注解。

### 注意

注解对于向工具和运行时环境传达关于应用程序的信息非常有用

## Java 类库

Java 包括许多支持应用程序开发的类库。其中包括以下内容：

+   `java.lang`

+   `java.io`

+   `java.net`

+   `java.util`

+   `java.awt`

这些库是按包组织的。每个包包含一组类。包的结构反映在其底层文件系统中。`CLASSPATH`环境变量保存了包的位置。

有一组核心的包是 JDK 的一部分。这些包通过提供对一组标准功能的简单访问，为 Java 的成功提供了至关重要的元素，这些功能在其他语言中并不容易获得。

以下表格显示了一些常用包的列表：

| 包 | 用法 |
| --- | --- |
| `java.lang` | 这是基本语言类型的集合。它包括根类`Object`和`Class`，以及线程，异常，包装器和其他基本类等其他项目。 |
| `java.io` | 包括流和随机访问文件。 |
| `java.net` | 支持套接字，telnet 接口和 URL。 |
| `java.util` | 支持容器和实用类，如`Dictionary`，`HashTable`和`Stack`。编码器和解码器技术，如`Date`和`Time`，也可以在此库中找到。 |
| `java.awt` | 包含**抽象窗口工具包**（**AWT**），其中包含支持**图形用户界面**（**GUI**）的类和方法。它包括用于事件，颜色，字体和控件的类。 |

# 摘要

在本章中，我们研究了 Java 的基本方面和一个简单的 Java 控制台应用程序。从认证的角度来看，我们研究了一个使用`main`方法的类和 Java 应用程序的结构。

我们还介绍了一些将在后续章节中更详细讨论的其他主题。这包括对象的创建和操作，字符串和`StringBuilder`类的使用，类的实例和静态成员，以及在方法的重载和重写中使用签名。

有了这个基础，我们准备继续第二章，*Java 数据类型及其用法*，在那里我们将研究变量的性质以及它们的用法。

# 认证目标涵盖

在本章中，我们介绍了一些将在后续章节中更详细讨论的认证主题。在这里，我们深入讨论了以下主题：

+   定义 Java 类的结构（在*探索类的结构*部分）

+   创建一个带有主方法的可执行的 Java 应用程序（在*探索 Java 控制台程序结构*部分）

# 测试你的知识

1.  如果以下代码使用`java SomeClass hello world`命令运行，会打印出什么？

```java
public class SomeClass{
    public static void main(String argv[])
    {
  System.out.println(argv[1]);
    }
}
```

a. `world`

b. `hello`

c. `hello` `world`

d. 抛出`ArrayIndexOutOfBoundsException`

1.  考虑以下代码序列：

```java
public class SomeClass{
   public int i;
   public static void main(String argv[]){
      SomeClass sc = new SomeClass();
      // Comment line
   }
}
```

如果它们替换注释行，以下哪个语句将在不会出现语法或运行时错误的情况下编译？

a. `sc.i = 5;`

b. `int j = sc.i;`

c. `sc.i = 5.0;`

d. `System.out.println(sc.i);`
