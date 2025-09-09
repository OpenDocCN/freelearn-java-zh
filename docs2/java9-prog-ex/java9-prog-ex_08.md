# 扩展我们的电子商务应用

在上一章中，我们开始开发一个电子商务应用，并创建了根据产品 ID 和一些参数查找产品的功能。在本章中，我们将扩展功能，以便我们还可以订购我们选择的产品。在这个过程中，我们将学习新技术，重点关注 Java 中的函数式编程以及一些其他语言特性，如运行时反射和注解处理，以及脚本接口。

正如我们在前面的章节中所做的那样，我们将逐步开发应用。当我们发现新学的技术时，我们将重构代码，以引入新的工具和方法，以产生更易读和有效的代码。我们还将模仿真实项目的开发，即在开始时，我们将有简单的需求，而后来，随着我们想象中的业务发展和销售更多产品，新的需求将被设定。我们将成为想象中的百万富翁。

我们将使用上一章的代码库，并将在此基础上进一步开发，尽管如此，我们将在一个新的项目中这样做。我们将使用 Spring、Gradle、Tomcat 和 soapUI，这些在上一章中我们已经熟悉了。在本章中，你将学习以下主题：

+   注解处理

+   使用反射

+   使用以下方式在 Java 中进行函数式编程：

    +   Lambda 表达式

    +   流

    +   从 Java 调用脚本

# MyBusiness 订购

订单过程比仅仅查找产品要复杂一些。订单表单本身列出了产品和数量，并确定了该订单的客户是谁。标识符提供了产品信息。我们只需要检查这些产品是否在我们的商店中有货，并且我们可以将它们交付给指定的客户。这是最简单的方法；然而，对于某些产品，存在更多的限制。例如，当有人订购桌面台灯时，我们会单独交付电源线。这是因为电源线是特定于国家的。我们向英国和德国交付不同的电源线。一种可能的方法是识别客户的国籍。但这种方法没有考虑到我们的客户是转售商。所有客户可能都位于英国，同时他们可能希望将台灯和电源线一起运往德国。为了避免这种情况和歧义，我们的客户最好将桌面台灯和电源线作为同一订单中的单独项目订购。在某些情况下，我们不附带电源线交付桌面台灯，但这是一种特殊情况。我们需要一些逻辑来识别这些特殊情况。因此，我们必须实现逻辑来查看是否存在桌面台灯的电源线，如果没有自动处理订单，则拒绝订单。这并不意味着我们不会交付产品。我们只会将订单放入队列，然后某个操作员需要查看它。

这种方法的缺点在于，桌面台灯只是需要配置支持的一个产品。我们拥有的产品越多，它们可能具有的专长就越多，检查订单一致性的代码变得越来越复杂，直到达到无法管理的复杂程度。当一个类或方法变得过于复杂时，程序员会对其进行重构，将方法或类拆分成更小的部分。我们必须对产品检查做同样的事情。我们不应该试图创建一个巨大的类来检查产品和所有可能的订单组合，而应该有许多较小的检查，以便每个检查只针对一个小集合。

在某些情况下，检查一致性更简单。检查灯具是否有电源线，其复杂度任何新手程序员都可以编写。我们在代码中使用这个例子，因为我们想关注代码的实际结构，而不是检查本身的复杂性质。然而，在现实生活中，检查可能相当复杂。想象一下一家销售电脑的商店。它组装一个配置：电源供应、显卡和主板，适当的 CPU 和内存。有许多选择，其中一些可能无法一起工作。在现实情况下，我们需要检查主板与所选内存兼容，主板有与订单中相同数量的存储器组，它们被适当配对（一些内存只能成对安装），有一个兼容的插槽用于显卡，以及电源有足够的瓦特数来可靠地运行整个配置。这是非常复杂的，最好不要与检查灯具是否有电源线的代码混淆。

# 设置项目

由于我们仍在使用 Spring Boot，构建文件不需要任何修改；我们将像使用上一章的文件一样使用它。然而，包结构略有不同。这次，我们要做比仅仅获取请求并响应后端服务提供的内容更复杂的事情。现在，我们必须实现复杂的企业逻辑，正如我们将看到的，这需要许多类。当我们在一个包中有 10 多个类时，是时候考虑将它们放入单独的包中。相互关联并具有相似功能的类应该放在一个包中。这样，我们将有一个包用于以下内容：

+   控制器（尽管在这个例子中我们只有一个，但通常会有更多）

+   只具有存储数据功能的数据存储 bean，因此，字段、setter 和 getter

+   当订购台式灯时帮助我们检查电源线的检查器

+   为控制器执行不同服务的服务

+   包含`Application`类、`SpringConfiguration`和一些接口的我们程序的主要包

# 订单控制器和 DTOs

当服务器收到一个请求来订购一系列产品时，它以 HTTPS `POST`请求的形式到来。请求体以 JSON 编码。到目前为止，我们已经有处理`GET`参数的控制器，但当我们能够依赖 Spring 的数据绑定时，处理`POST`请求并不比这更困难。控制器代码本身很简单：

```java
package packt.java9.by.example.mybusiness.bulkorder.controllers; 

import ... 

@RestController 
public class OrderController { 
    private Logger log = 
                LoggerFactory.getLogger(OrderController.class); 
    private final Checker checker; 

    public OrderController(@Autowired Checker checker) { 
        this.checker = checker; 
    } 

    @RequestMapping("/order") 
    public Confirmation getProductInformation(@RequestBody Order order) { 
        if (checker.isConsistent(order)) { 
            return Confirmation.accepted(order); 
        } else { 
            return Confirmation.refused(order); 
        } 
    } 
}

```

在这个控制器中，我们只处理一个请求：`order`。这映射到 URL `/order`。订单会自动从请求体中转换为 JSON 格式的订单对象。这就是 `@RequestBody` 注解要求 Spring 为我们做的事情。控制器的功能仅仅是检查订单的一致性。如果订单是一致的，那么我们接受订单；否则，我们拒绝它。现实生活中的例子也会检查订单不仅是一致的，而且来自有资格购买这些产品的客户，并且这些产品在仓库中有货，或者至少根据生产者的承诺和交货期可以交付。

要检查订单的一致性，我们需要一个为我们做这项工作的东西。正如我们所知，我们必须模块化代码，不要在单个类中实现太多东西，因此我们需要一个检查对象。这是根据类的注解和控制器构造函数的 `@Autowired` 自动提供的。

`Order` 类是一个简单的 JavaBean，仅列出项目：

```java
package packt.java9.by.example.mybusiness.bulkorder.dtos; 
import ...; 
public class Order { 
    private String orderId; 
    private List<OrderItem> items; 
    private String customerId; 

... setters and getters ... 
}

```

包的名称是 `dtos`，代表 **数据传输对象**（DTO）的复数形式。DTOs 是用于在不同组件之间传输数据（通常通过网络）的对象。由于另一端可以用任何语言实现，序列化可以是 JSON、XML 或其他仅能传输数据的格式。这些类没有实际的方法。DTOs 通常只有字段、设置器和获取器。

以下是一个包含订单中一个项目的类：

```java
package packt.java9.by.example.mybusiness.bulkorder.dtos; 

public class OrderItem { 
    private double amount; 
    private String unit; 
    private String productId; 

... setters and getters ... 
}

```

订单确认也在此包中，尽管这也是一个真正的 DTO，但它有一些简单的辅助方法：

```java
package packt.java9.by.example.mybusiness.bulkorder.dtos; 

public class Confirmation { 
    private final Order order; 
    private final boolean accepted; 

    private Confirmation(Order order, boolean accepted) { 
        this.order = order; 
        this.accepted = accepted; 
    } 

    public static Confirmation accepted(Order order) { 
        return new Confirmation(order, true); 
    } 

    public static Confirmation refused(Order order) { 
        return new Confirmation(order, false); 
    } 

    public Order getOrder() { 
        return order; 
    } 

    public boolean isAccepted() { 
        return accepted; 
    } 
}

```

我们为该类提供了两个工厂方法。这有点违反了单一职责原则，这是纯粹主义者所厌恶的。大多数时候，当代码变得更加复杂，这样的捷径会反过来咬人，代码不得不重构以变得更加简洁。纯粹主义的解决方案是创建一个单独的工厂类。使用这个类或分离的类的工厂方法可以使控制器的代码更易于阅读。

我们的主要任务是进行一致性检查。到目前为止，代码几乎是微不足道的。

# 一致性检查器

我们有一个一致性检查器类，并且它的一个实例被注入到控制器中。这个类用于检查一致性，但它本身并不执行检查。它只控制我们提供的不同检查器，并依次调用它们来完成实际工作。

我们要求一致性检查器，例如当订购台灯时检查订单是否包含电源线的一致性检查器，实现 `ConsistencyChecker` 接口：

```java
package packt.java9.by.example.mybusiness.bulkorder; 

import ... 
public interface ConsistencyChecker { 
    boolean isInconsistent(Order order); 
}

```

`isInconsistent`方法应该在订单不一致时返回`true`。如果它不知道订单是否不一致，则返回`false`，但从实际检查器检查订单的角度来看，没有不一致性。由于有多个`ConsistencyChecker`类，我们必须一个接一个地调用，直到其中一个返回`true`或者我们用完它们。如果没有一个返回`true`，那么至少从自动化检查器的角度来看，我们可以安全地假设订单是一致的。

在开发初期，我们就知道我们将会真正拥有很多一致性检查器，并不是所有检查器都对所有订单都相关。我们希望避免为每个订单调用每个检查器。为了做到这一点，我们实现了一些过滤。我们让产品指定它们需要哪种类型的检查。这是一些产品信息，比如大小或描述。为了适应这一点，我们需要扩展`ProductInformation`类。

我们将创建每个`ConsistencyChecker`接口，实现类作为 Spring bean（使用`@Component`注解标注），同时，我们将使用一个注解来指定它们实现哪种类型的检查。同时，`ProductInformation`被扩展，包含一组`Annotation`类对象，这些对象指定了要调用哪些检查器。我们本可以简单地列出检查器类而不是注解，但这给了我们在配置产品与注解之间的映射时一些额外的自由度。注解指定了产品的类型，检查器类被注解。台灯有`PoweredDevice`类型，检查器类`NeedPowercord`被`@PoweredDevice`注解标注。如果有其他类型的需要电源线的产品，那么该类型的注解应该添加到`NeedPowercord`类中，我们的代码就能工作。由于我们开始深入研究注解和注解处理，我们首先必须了解注解到底是什么。自从第三章，*优化排序，使代码专业*以来，我们已经使用了注解，但我们只知道如何使用它们，而且如果不理解我们所做的，这通常是危险的。

# 注解

注解使用前面的`@`字符，可以附加到包、类、接口、字段、方法、方法参数、泛型类型声明和使用，以及最后，附加到注解上。注解几乎可以用于任何地方，它们用于描述一些程序元信息。例如，`@RestController`注解不会直接改变`OrderController`类的行为。类的行为由内部的 Java 代码描述。注解帮助 Spring 理解类是什么以及它应该如何以及如何使用。当 Spring 扫描所有包和类以发现不同的 Spring beans 时，它看到类上的注解并将其考虑在内。类上可能有 Spring 不理解的其它注解。它们可能被某些其他框架或程序代码使用。Spring 像任何良好行为的框架一样忽略它们。例如，正如我们稍后将要看到的，在我们的代码库中，有一个名为`NeedPowercord`的类，它是一个 Spring bean，因此被`@Component`注解标注。同时，它还被`@PoweredDevice`注解标注。Spring 对什么是带电设备一无所知。这是我们定义和使用的。Spring 忽略它。

包、类、接口、字段等都可以附加许多注解。这些注解应该简单地写在它们所附加的语法单元声明之前。

在包的情况下，注解必须写在`package-info.java`文件中的包名之前。此文件可以放置在包的目录中，可以用来编辑包的*JavaDoc*，也可以用来向包添加注解。此文件不能包含任何 Java 类，因为`package-info`名称不是一个有效的标识符。

我们不能随意在任意位置写任何内容作为注解。注解应该被声明。它们位于 Java 运行时的特殊接口中。例如，声明`@PoweredDevice`注解的 Java 文件看起来像这样：

```java
package packt.java9.by.example.mybusiness.bulkorder.checkers; 

import java.lang.annotation.Retention; 
import java.lang.annotation.RetentionPolicy; 

@Retention(RetentionPolicy.RUNTIME) 
public @interface PoweredDevice { 
}

```

`interface`关键字前面的`@`字符表明这是一个特殊类型：注解类型。有一些特殊规则；例如，注解接口不应扩展任何其他接口，甚至不是注解接口。另一方面，编译器自动使注解接口扩展 JDK 接口`java.lang.annotation.Annotation`。

注解位于源代码中，因此，它们在编译过程中是可用的。编译器还可以保留这些注解并将它们放入生成的类文件中，当类加载器加载类文件时，它们也可能在运行时可用。默认行为是编译器将注解与其注解的元素一起存储在类文件中，但类加载器不会在运行时保持它们可用。

要在编译过程中处理注解，必须使用注解处理器扩展 Java 编译器。这是一个相当高级的话题，在处理 Java 时你只能遇到少数几个例子。注解处理器是一个实现了特殊接口的 Java 类，当编译器处理注解处理器声明的源文件中的注解时，会调用它。

# 注解保留

Spring 和其他框架通常在运行时处理注解。编译器和类加载器必须被指示注解在运行时应该保持可用。为此，必须使用`@Retention`注解来注解注解接口本身。这个注解有一个`RetentionPolicy`类型的参数，它是一个枚举。我们很快就会讨论注解参数应该如何定义。

有趣的是要注意，注解接口上的`@Retention`注解必须在类文件中可用；否则，类加载器将不知道如何处理注解。我们如何通知编译器在编译过程结束后保留注解？我们注解注解接口声明。因此，`@Retention`的声明被自己注解，并声明为在运行时可用。

注解声明可以使用`@Retention(RetentionPolicy.SOURCE)`、`@Retention(RetentionPolicy.CLASS)`或`@Retention(RetentionPolicy.RUNTIME)`进行注解。

# 注解目标

最后的保留类型将是使用最频繁的。还有其他可以在注解声明上使用的注解。`@Target`注解可以用来限制注解的使用范围。这个注解的参数是一个`java.lang.annotation.ElementType`的值或这些值的数组。限制注解的使用有一个很好的理由。当我们将注解放置在错误的位置时，得到编译时错误要比在运行时寻找框架为什么忽略我们的注解要好得多。

# 注解参数

正如我们所见，注解可以有参数。为了在注解的`@interface`声明中声明这些参数，我们使用方法。这些方法有一个名称和返回值，但它们不应该有参数。你可以尝试声明一些参数，但 Java 编译器将会非常严格，不会编译你的代码。

这些值可以在使用注解的地方定义，使用方法名称和`=`字符，将一些与方法的类型兼容的值赋给它们。例如，假设我们修改注解`PoweredDevice`的声明如下：

```java
public @interface ParameteredPoweredDevice { 
    String myParameter(); 
}

```

在这种情况下，在使用注解时，我们应该为参数指定一些值，例如以下内容：

```java
@Component 
@ParameteredPoweredDevice(myParameter = "1966") 
public class NeedPowercord implements ConsistencyChecker { 
...

```

如果参数的名称是一个值，并且在注解的使用位置没有定义其他参数，那么名称“value”可以被省略。例如，当我们只有一个参数时，按照以下方式修改代码是一个方便的缩写：

```java
public @interface ParameteredPoweredDevice{ 
    String value(); 
} 
... 
@Component 
@ParameteredPoweredDevice("1966") 
public class NeedPowercord implements ConsistencyChecker { 
...

```

我们也可以使用`default`关键字在方法声明后定义可选参数。在这种情况下，我们必须为参数定义一个默认值。进一步修改我们已有的示例注解，我们仍然可以，但不必指定值。在后一种情况下，它将是一个空字符串：

```java
public @interface ParameteredPoweredDevice { 
    String value() default ""; 
}

```

由于我们指定的值应该在编译时是常量和可计算的，因此复杂类型并没有太大的用途。注解参数通常是字符串、整数，有时是双精度浮点数或其他原始类型。语言规范提供的类型列表如下：

+   原始类型（`double`、`int`等）

+   字符串

+   类

+   枚举

+   另一个注解

+   上述任何类型的数组

我们已经看到了`String`的例子，也看到了`enum`:`Retention`和`Target`都有`enum`参数。我们想要关注的有意思的部分是上述列表的最后两项。

当参数的值是一个数组时，值可以在`{`和`}`字符之间指定为逗号分隔的值。例如：

```java
String[] value();

```

然后，我们可以将其添加到我们可以编写的`@interface`注解中：

```java
@ParameteredPoweredDevice({"1966","1967","1991"})

```

然而，如果我们只想传递一个值作为参数值，我们仍然可以使用以下格式：

```java
@ParameteredPoweredDevice("1966")

```

在这种情况下，属性的值将是一个长度为`1`的数组。当注解的值是一个注解类型的数组时，事情会变得稍微复杂一些。我们创建一个`@interface`注解（注意名字中的复数形式）：

```java
@Retention(RetentionPolicy.RUNTIME) 
public @interface PoweredDevices { 
ParameteredPoweredDevice[] value() default {}; 
}

```

这个注解的使用可以如下所示：

```java
@PoweredDevices( 
        {@ParameteredPoweredDevice("1956"), @ParameteredPoweredDevice({"1968", "2018"})} 
)

```

注意，这并不等同于拥有三个参数的`ParameteredPoweredDevice`注解。这是一个有两个参数的注解。每个参数都是一个注解。第一个有一个字符串参数，第二个有两个。

如您所见，注解可以相当复杂，一些框架（或者更确切地说，创建它们的程序员）在使用它们时有些失控。在您开始编写框架之前，研究一下是否已经存在您可以使用的一个框架。同时，检查是否有其他方法可以解决您的问题。99%的注解处理代码可以避免，并变得简单。我们编写的代码越少，对于相同的功能就越满意。我们程序员是懒惰的类型，这就是必须这样做的原因。

最后一个例子，其中注解的参数是一个注解数组，理解我们如何创建可重复的注解是很重要的。

# 可重复注解

使用`@Repeatable`注解注解注解的声明，以表示该注解可以在一个地方多次应用。这个注解的参数是一个注解类型，它应该有一个类型为该注解数组的参数。不要试图理解！我将给出一个例子。实际上，我已经有了：我们有`@PoweredDevices`。它有一个参数是一个`@ParameteredPoweredDevice`的数组。考虑我们现在将这个`@interface`注解如下：

```java
... 
@Repeatable(PoweredDevices.class) 
public @interface ParameteredPoweredDevice { 
...

```

然后，我们可以简化`@ParameteredPoweredDevice`的使用。我们可以多次重复注解，Java 运行时会自动将其封装在包装类中，在这种情况下是`@PoweredDevices`。在这种情况下，以下两个将是等效的：

```java
... 
@ParameteredPoweredDevice("1956") 
@ParameteredPoweredDevice({"1968", "2018"}) 
public class NeedPowercord implements ConsistencyChecker { 
... 

@PoweredDevices( 
        {@ParameteredPoweredDevice("1956"), @ParameteredPoweredDevice({"1968", "2018"})} 
) 
public class NeedPowercord implements ConsistencyChecker { 
...

```

这种复杂方法的理由再次是 Java 严格遵循的后向兼容性的一个例子。注解是在 Java 1.5 中引入的，而可重复注解则只从版本 1.8 开始可用。我们很快就会谈到我们用来在运行时处理注解的反射 API。这个 API 在`java.lang.reflect.AnnotatedElement`接口中有一个`getAnnotation(annotationClass)`方法，它返回一个注解。如果一个注解可以在类、方法等上出现多次，那么就没有办法调用这个方法来获取所有不同参数的不同实例。通过引入包含类型来包装多个注解，确保了后向兼容性。

# 注解继承

注解，就像方法或字段一样，可以在类层次结构之间继承。如果一个注解声明被标记为`@Inherited`，那么扩展了具有此注解的另一个类的类可以继承它。如果子类有注解，则可以覆盖注解。因为 Java 中没有多重继承，所以接口上的注解不能继承。即使注解被继承，检索特定元素注解的应用程序代码也可以区分继承的注解和实体本身声明的注解。有方法可以获取注解，还有单独的方法可以获取实际元素上声明的注解，而不是继承的。

# `@Documented`注解

`@Documented`注解表达了这样的意图：注解是实体契约的一部分，因此它必须包含在文档中。这是一个*JavaDoc*生成器在创建引用`@Documented`注解的元素的文档时查看的注解。

# JDK 注解

除了用于定义注解的注解之外，JDK 中还定义了其他一些注解。我们已经看到了其中的一些。最常用的是`@Override`注解。当编译器看到这个注解时，它会检查该方法是否真正覆盖了某个继承的方法。如果没有这样做，将会导致错误，从而避免我们在运行时调试中的痛苦。

`@Deprecated`注解在方法、类或其他元素的文档中表示该元素不应使用。它仍然存在于代码中，因为一些用户可能仍然会使用它，但在依赖于包含该元素库的新开发中，新开发的代码不应使用它。该注解有两个参数。一个参数是`since`，它可以有一个字符串值，可以提供关于方法或类何时或从哪个版本开始被弃用的版本信息。另一个参数是`forRemoval`，如果该元素将不会出现在库的未来版本中，则应设置为`true`。某些方法可能因为存在更好的替代方案而被弃用，但开发者并不打算从库中删除该方法。在这种情况下，`forRemoval`可以设置为`false`。

`@SuppressWarning`注解也是一个常用的注解，尽管其使用是有疑问的。它可以用来抑制编译器的一些警告。如果可能，建议编写没有警告的代码。

`@FunctionalInterface`注解声明一个接口只打算有一个方法。这样的接口可以作为 lambda 表达式实现。你将在本章后面学习关于 lambda 表达式的内容。当这个注解应用于一个接口，并且接口中声明了多个方法时，编译器将发出编译错误。这将防止任何开发者早期向一个打算与函数式编程和 lambda 表达式一起使用的接口中添加另一个方法。

# 使用反射

现在你已经学会了如何声明注解以及如何将它们附加到类和方法上，我们可以回到我们的`ProductInformation`类。回想一下，我们想要指定这个类中的产品类型，并且每种产品类型都由一个`@interface`注解表示。我们已经在之前的几页中列出了它，即我们将在`@PoweredDevice`示例中实现的一个。我们将假设将来会有许多这样的注解、产品类型和带有`@Component`以及一个或多个我们注解的一致性检查器。

# 获取注解

我们将使用以下字段扩展`ProductInformation`类：

```java
private List<Class<? extends Annotation>> check;

```

由于这是一个 DTO，Spring 需要设置器和获取器，因此我们也将添加一个新的获取器和设置器。这个字段将包含每个类实现的我们的一些注解以及内置的 JDK 接口 `Annotation`，因为这就是 Java 编译器生成它们的方式。在这个阶段，这可能会有些模糊，但我保证随着我们的继续前进，曙光将破晓，光明将到来。

要获取产品信息，我们必须通过 ID 查找它。这是我们在上一章中开发的接口和服务，除了这次我们有一个新的字段。实际上，这是一个重大的区别，尽管 `ProductLookup` 接口完全没有变化。在上一章中，我们开发了两个版本。其中一个版本是从属性文件中读取数据，另一个版本是连接到 REST 服务。

属性文件很丑陋，是过时的技术，但如果你打算通过 Java 面试或在 21 世纪初开发的企业应用程序中工作，则是必须的。我不得不将其包含在最后一章中。这是我自己强烈希望在书中包含它的愿望。同时，在编写这一章的代码时，我没有勇气继续使用它。我还想向你展示相同的内容可以用 JSON 格式管理。

现在，我们将扩展 `ResourceBasedProductLookup` 的实现，以从 JSON 格式的资源文件中读取产品信息。在类中，大部分代码保持不变；因此，我们只列出这里的不同之处：

```java
package packt.java9.by.example.mybusiness.bulkorder.services; 

import ... 

@Service 
public class ResourceBasedProductLookup implements ProductLookup { 
    private static final Logger log = LoggerFactory.getLogger(ResourceBasedProductLookup.class); 

    private ProductInformation fromJSON(InputStream jsonStream) 
                                              throws IOException { 
        ObjectMapper mapper = new ObjectMapper(); 
        return mapper.readValue(jsonStream, 
                                   ProductInformation.class); 
    } 

... 
    private void loadProducts() { 
        if (productsAreNotLoaded) { 
            try { 
                Resource[] resources =  
                     new PathMatchingResourcePatternResolver(). 
                        getResources("classpath:products/*.json"); 
                for (Resource resource : resources) { 
                    loadResource(resource); 
                } 
                productsAreNotLoaded = false; 
            } catch (IOException ex) { 
                log.error("Test resources can not be read", ex); 
            } 
        } 
    } 

    private void loadResource(Resource resource) 
                                       throws IOException { 
        final int dotPos = 
                      resource.getFilename().lastIndexOf('.'); 
        final String id = 
                      resource.getFilename().substring(0, dotPos); 
        final ProductInformation pi = 
                      fromJSON(resource.getInputStream()); 
        pi.setId(id); 
        products.put(id, pi); 
    } 
...

```

在 `project resources/products` 目录中，我们有一些 JSON 文件。其中之一包含台灯产品的信息：

```java
{ 
  "id" : "124", 
  "title": "Desk Lamp", 
  "check": [ 
    "packt.java9.by.example.mybusiness.bulkorder.checkers.PoweredDevice" 
  ], 
  "description": "this is a lamp that stands on my desk", 
  "weight": "600", 
  "size": [ "300", "20", "2" ] 
}

```

产品类型在 JSON 数组中指定。在这个例子中，这个数组只有一个元素，而这个元素是表示产品类型的注解接口的完全限定名称。当 JSON 序列化器将 JSON 转换为 Java 对象时，它会识别出需要此信息的字段是一个 `List`，因此它将数组转换为列表，并将元素从 `String` 转换为表示注解接口的 `Class` 对象。

现在我们已经从 JSON 格式的资源中加载了资源，并看到了使用 Spring 读取 JSON 数据是多么容易，我们可以回到订单一致性检查。`Checker` 类实现了收集可插拔检查器并调用它们的逻辑。它还实现了基于注解的筛选，以便不调用我们实际上不需要用于实际产品实际订单的检查器：

```java
package packt.java9.by.example.mybusiness.bulkorder.services; 

import ... 

@Component() 
@RequestScope 
public class Checker { 
    private static final Logger log = 
                        LoggerFactory.getLogger(Checker.class); 

    private final Collection<ConsistencyChecker> checkers; 
    private final ProductInformationCollector piCollector; 
    private final ProductsCheckerCollector pcCollector; 

    public Checker( 
              @Autowired Collection<ConsistencyChecker> checkers, 
              @Autowired ProductInformationCollector piCollector, 
              @Autowired ProductsCheckerCollector pcCollector) { 
        this.checkers = checkers; 
        this.piCollector = piCollector; 
        this.pcCollector = pcCollector; 
    } 

    public boolean isConsistent(Order order) { 
        Map<OrderItem, ProductInformation> map = 
                piCollector.collectProductInformation(order); 
        if (map == null) { 
            return false; 
        } 
        Set<Class<? extends Annotation>> annotations =  
                pcCollector.getProductAnnotations(order); 
        for (ConsistencyChecker checker :  
                checkers) { 
            for (Annotation annotation :  
                    checker.getClass().getAnnotations()) { 
                if (annotations.contains( 
                                 annotation.annotationType())) { 
                    if (checker.isInconsistent(order)) { 
                        return false; 
                    } 
                    break; 
                } 
            } 
        } 
        return true; 
    } 
}

```

有一个有趣的事情要提一下，那就是 Spring 的自动装配非常聪明。我们有一个`Collection<ConsistencyChecker>`类型的字段。通常，如果有一个恰好与要连接的资源类型相同的类，自动装配就会工作。在我们的情况下，我们没有这样的候选者，因为这是一个集合，但我们有多个`ConsistencyChecker`类。我们所有的检查器都实现了这个接口，Spring 能够识别它们，实例化它们，神奇地创建一个它们的集合，并将这个集合注入到这个字段中。

通常一个好的框架在逻辑上是合理的。我之前并不了解 Spring 的这个特性，但我想这应该是合理的，神奇的是，它真的工作了。如果事情是合理的并且顺利运行，你就不需要阅读和记住文档。然而，一点谨慎并不会有害。在我体验过这个功能是这样工作的之后，我在文档中查看了这个功能，发现这确实是 Spring 的一个保证特性，而不是仅仅偶然工作，可能在未来的版本中未经通知而改变。只使用保证的特性非常重要，但我们的行业中经常被忽视。

当`isConsistent`方法被调用时，它首先将产品信息收集到`HashMap`中，为每个`OrderItem`分配一个`ProductInformation`实例。这是在一个单独的类中完成的。之后，`ProductsCheckerCollector`收集一个或多个产品项需要的`ConsistencyChecker`实例。当我们有了这个集合后，我们只需要调用那些带有这个集合中注解之一的检查器。我们在循环中这样做。

在此代码中，我们使用反射。我们遍历每个检查器拥有的注解。为了获取注解集合，我们调用`checker.getClass().getAnnotations()`。这个调用返回一个对象集合。每个对象都是实现我们自己在源文件中声明的注解接口的 JDK 运行时生成的类的实例。然而，没有保证动态创建的类只实现了我们的`@interface`而没有实现其他接口。因此，为了获取实际的注解类，我们必须调用`annotationType`方法。

`ProductCheckerCollector`和`ProductInformationCollector`类非常简单，我们将在学习流时再讨论它们。它们将在那个地方作为一个很好的例子，当我们使用循环实现它们，然后立即使用流来实现。

在拥有它们之后，我们最终可以创建我们的实际检查器类。帮助我们看到我们的台灯订购了电源线的那个类如下：

```java
package packt.java9.by.example.mybusiness.bulkorder.checkers; 

import ... 
@Component 
@PoweredDevice 
public class NeedPowercord implements ConsistencyChecker { 
    private static final Logger log = 
               LoggerFactory.getLogger(NeedPowercord.class); 

    @Override 
    public boolean isInconsistent(Order order) { 
        log.info("checking order {}", order); 
        CheckHelper helper = new CheckHelper(order); 
        return !helper.containsOneOf("126", "127", "128"); 
    } 
}

```

辅助类包含许多检查器需要的简单方法，例如：

```java
public boolean containsOneOf(String... ids) { 
    for (final OrderItem item : order.getItems()) { 
        for (final String id : ids) { 
            if (item.getProductId().equals(id)) { 
                return true; 
            } 
        } 
    } 
    return false; 
}

```

# 调用方法

在这个例子中，我们只使用了一个单独的反射调用来获取附加到类上的注解。反射可以做更多的事情。处理注解是这些调用最重要的用途，因为注解没有自己的功能，并且在运行时无法以任何其他方式处理。然而，反射并没有停止告诉我们一个类或任何其他可注解元素有哪些注解。反射可以用来获取类的列表、方法名称作为字符串、类的实现接口、它扩展的父类、字段、字段类型等等。反射通常提供方法和类来遍历实际的代码结构，直到方法级别，以编程方式。

这种遍历不仅允许读取类型和代码结构，而且还使得在编译时不知道方法名称的情况下设置字段值和调用方法成为可能。我们甚至可以设置那些是`private`且通常无法由外界访问的字段。还值得注意的是，通过反射访问方法和字段通常比通过编译代码慢，因为它总是涉及到在代码中通过元素名称进行查找。

经验法则表明，如果你发现必须使用反射来创建代码，那么意识到你可能正在创建一个框架（或者正在编写一本关于 Java 的书籍，详细介绍了反射）。这听起来熟悉吗？

Spring 也使用反射来发现类、方法和字段，以及注入对象。它使用 URL 类加载器列出类路径上所有的 JAR 文件和目录，加载它们，并检查类。

为了举例说明，为了演示目的，让我们假设`ConsistencyChecker`的实现是由许多外部软件供应商编写的，而最初设计程序结构的架构师恰好忘记在接口中包含`isConsistent`方法。（同时，为了保护我们的心理健康，我们也可以想象这个人因为这样做而不再在公司工作。）结果，不同的供应商提供了“实现”这个接口的 Java 类，但我们无法调用该方法，不仅因为我们没有具有此方法的公共父接口，而且因为供应商恰好使用了不同的方法名称。

在这种情况下我们能做什么呢？从业务角度来看，要求所有供应商重写他们的检查器是不被允许的，因为他们知道我们遇到麻烦，这给任务贴上了高昂的标签。我们的经理们希望避免这种成本，我们开发者也希望表明我们可以修复这种情况并创造奇迹。（稍后，我会对此发表评论。）

我们可以创建一个类，它知道每个校验器以及如何以多种不同的方式调用它们。这要求我们每次向系统中引入新的校验器时都要维护这个类，而我们希望避免这样做。我们最初发明整个插件架构正是为了这个目的。

我们如何调用一个对象上的方法，我们知道这个对象只有一个声明的方法，它接受一个订单作为参数？这就是反射发挥作用的地方。我们不是调用`checker.isInconsistent(order)`，而是实现一个小的`private`方法`isInconsistent`，它通过反射调用该方法，无论其名称是什么：

```java
private boolean isInconsistent(ConsistencyChecker checker, Order order) { 
    Method[] methods = checker.getClass().getDeclaredMethods(); 
    if (methods.length != 1) { 
        log.error( 
                "The checker {} has zero or more than one methods", 
                checker.getClass()); 
        return false; 
    } 
    final Method method = methods[0]; 
    final boolean inconsistent; 
    try { 
        inconsistent = (boolean) method.invoke(checker, order); 
    } catch (InvocationTargetException | 
            IllegalAccessException | 
            ClassCastException e) { 
        log.error("Calling the method {} on class {} threw exception", 
                method, checker.getClass()); 
        log.error("The exception is ", e); 
        return false; 
    } 
    return inconsistent; 
}

```

我们可以通过调用`getClass`方法来获取对象的类，对于代表类的对象本身，我们可以调用`getDeclaredMethods`。幸运的是，校验器类并没有被许多方法所充斥，因此我们检查校验器类中只声明了一个方法。请注意，反射库中还有一个`getMethods`方法，但它总是会返回多个方法。它返回声明和继承的方法。因为每个类都继承自`java.lang.Object`，至少`Object`类的那些方法都会存在。

在此之后，我们尝试使用代表反射类中方法的`Method`对象来调用类。请注意，这个`Method`对象并不是直接附加到一个实例上的。我们是从类中检索到这个方法的，因此当我们调用它时，我们应该将作为第一个参数传递给它应该工作的对象。这样，`x.y(z)`就变成了`method.invoke(x,z)`。`invoke`方法的最后一个参数是一个可变数量的参数，它们作为`Object`数组传递。在大多数情况下，当我们调用一个方法时，即使我们不知道方法的名称，也必须使用反射，我们也会知道我们的代码中的参数。当即使参数也不为人所知但可以通过计算获得时，我们必须将它们作为`Object`数组传递。

通过反射调用方法是一个风险很大的调用。如果我们尝试以正常的方式调用一个方法，比如`private`，编译器将会报错。如果参数的数量或类型不合适，编译器会再次给我们报错。如果返回值不是`boolean`，或者根本没有返回值，那么我们再次得到编译器错误。在反射的情况下，编译器是无知的。它不知道代码执行时将调用哪个方法。另一方面，`invoke`方法在调用时可以并会注意到所有这些失败。如果上述任何问题发生，我们将会得到异常。如果`invoke`方法本身看到它无法执行我们所要求的事情，那么它将抛出`InvocationTargetException`或`IllegalAccessException`。如果从实际返回值到`boolean`的转换不可能，那么我们将得到`ClassCastException`。

关于做魔术，这是一种自然的冲动，我们想要创造出非凡的、杰出的事物。当我们进行实验或从事爱好工作时，这是可以接受的。另一方面，当我们从事专业工作时，这却绝对是不妥的。那些不理解你聪明解决方案的普通程序员，会在企业环境中维护代码。他们会将你精心整理的代码变成一堆乱麻，在修复一些错误或实现一些小的新功能时。即使你是编程界的莫扎特，他们充其量也只是不知名的歌手。在企业环境中，一段精彩的代码可能就像一首挽歌，带有所有这个比喻的含义。

最后但同样重要的是，一个令人悲伤的现实是，我们通常不是编程界的莫扎特。

注意，如果原始值的返回值是原始类型，那么它将通过反射转换为对象，然后我们将其转换回原始值。如果没有返回值，换句话说，如果它是`void`，那么反射将返回一个`java.lang.Void`对象。`Void`对象只是一个占位符。我们不能将其转换为任何原始值或任何其他类型的对象。这是必需的，因为 Java 是严格的，`invoke`必须返回一个`Object`，所以运行时需要一些可以返回的东西。我们所能做的就是检查返回值的类是否确实是`Void`。

让我们继续故事情节和我们的解决方案。我们提交了代码，它在生产环境中运行了一段时间，直到软件供应商的新更新将其破坏。我们在测试环境中调试代码，看到现在这个类包含多个方法。我们的文档明确指出，它们应该只有一个`public`方法，他们提供了一个有...嗯...我们意识到其他方法都是`private`的。他们是正确的；根据合同，他们可以有`private`方法，所以我们必须修改代码。我们替换了查找唯一方法的行：

```java
Method[] methods = checker.getClass().getDeclaredMethods(); 
if (methods.length != 1) { 
... 
} 
final Method method = methods[0];

```

新的代码如下：

```java
final Method method = getSingleDeclaredPublicMethod(checker); 
if (method == null) { 
    log.error( 
            "The checker {} has zero or more than one methods", 
            checker.getClass()); 
    return false; 

}

```

我们编写的新方法来查找唯一的`public`方法如下：

```java
private Method getSingleDeclaredPublicMethod( 
                           ConsistencyChecker checker) { 
    final Method[] methods = 
        checker.getClass().getDeclaredMethods(); 
    Method singleMethod = null; 
    for (Method method : methods) { 
        if (Modifier.isPublic(method.getModifiers())) { 
            if (singleMethod != null) { 
                return null; 
            } 
            singleMethod = method; 
        } 
    } 
    return singleMethod; 
}

```

要检查方法是否是`public`的，我们使用`Modifier`类的一个`static`方法。有方法可以检查所有可能的修饰符。`getModifiers`方法返回的值是一个`int`位字段。不同的位有不同的修饰符，有一些常量定义了这些。这种简化导致了不一致性，你可以检查一个方法是否是接口或易失性的，实际上这是荒谬的。事实是，只能用于其他类型反射对象的位永远不会被设置。

有一个例外，就是`volatile`。这个位被重新用于表示桥接方法。桥接方法是由编译器自动创建的，可能存在深奥和复杂的问题，我们在这本书中不讨论。由于重用了相同的位，所以不会引起混淆，因为字段可以是`volatile`的，但作为一个字段，它不能是桥接方法。显然，字段就是字段，而不是方法。同样，方法不能是`volatile`字段。一般规则是：不要在反射对象上使用没有意义的方法；否则，了解你在做什么。

为了使故事线更加复杂，一个检查器的版本意外地将检查方法实现为包`private`。程序员只是忘记使用`public`关键字。为了简化，让我们再次假设类只声明了一个方法，但这个方法不是公开的。我们如何使用反射来解决这个问题？

显然，最简单的解决方案是要求供应商修复问题：这是他们的责任。然而，在某些情况下，我们必须在某个问题上创建一个解决方案。另一个解决方案是创建一个具有相同包中`public`方法的类，从其他类中调用包`private`方法，从而传递其他类。实际上，这个解决方案，作为此类错误的解决方案，似乎更加合理和干净，但这次，我们想使用反射。

为了避免`java.lang.IllegalAccessException`，我们必须将`method`对象设置为可访问。为此，我们必须在调用之前插入以下行：

```java
method.setAccessible(true);

```

注意，这不会将方法改为`public`。它只会使方法对于通过我们设置为可访问的`method`对象实例的可访问。

我见过一些代码通过调用`isAccessible`方法来检查一个方法是否可访问，并将此信息保存下来；如果方法原本不可访问，则将其设置为可访问，并在调用后恢复原始的可访问性。这完全是多余的。一旦`method`变量超出作用域，并且没有引用我们设置可访问性标志的对象，设置的效果就会消失。此外，设置`public`或可调用的方法的可访问性没有惩罚。

# 设置字段

我们也可以在`Field`对象上调用`setAccessible`，然后我们可以甚至使用反射设置私有字段的值。不进一步编造故事，只是为了举例，让我们创建一个名为`ConsistencyChecker`的`SettableChecker`：

```java
@Component 
@PoweredDevice 
public class SettableChecker implements ConsistencyChecker { 
    private static final Logger log = LoggerFactory.getLogger(SettableChecker.class); 

    private boolean setValue = false; 

    public boolean isInconsistent(Order order) { 
        return setValue; 
    } 
}

```

这个检查器将返回`false`，除非我们使用反射将字段设置为`true`。我们确实这样做了。我们在`Checker`类中创建了一个方法，并在检查过程中的每个检查器中调用它：

```java
private void setValueInChecker(ConsistencyChecker checker) { 
    Field[] fields = checker.getClass().getDeclaredFields(); 
    for( final Field field : fields ){ 
        if( field.getName().equals("setValue") && 
            field.getType().equals(boolean.class)){ 
            field.setAccessible(true); 
            try { 
                log.info("Setting field to true"); 
                field.set(checker,true); 
            } catch (IllegalAccessException e) { 
                log.error("SNAFU",e); 
            } 
        } 
    } 
}

```

该方法遍历所有声明的字段，如果名称是`setValue`且类型是`boolean`，则将其设置为`true`。这将本质上使所有包含有源设备的订单被拒绝。

注意，尽管`boolean`是一个内置的语言原语，它绝对不是一个类，但它仍然有一个类，以便反射可以比较字段的类型与`boolean`人为具有的类。现在`boolean.class`是语言中的一个类字面量，对于每个原语，都可以使用类似的常量。编译器将这些识别为类字面量，并在字节码中创建适当的伪类引用，以便原语也可以以这种方式进行检查，正如在`setValueInChecker`方法的示例代码中所展示的。

我们检查了字段是否具有适当的类型，并且我们还对该字段调用了`setAccessible`方法。尽管编译器不知道我们确实已经做了所有事情来避免`IllegalAccessException`，但它仍然认为在`field`上调用`set`可能会抛出这样的异常，因为它被声明了。然而，我们知道这种情况不应该发生。（这是程序员的著名最后遗言吗？）为了处理这种情况，我们在方法调用周围包围了一个`try`块，并在`catch`分支中记录了异常。

# Java 中的函数式编程

由于我们在本章的示例中创建了大量的代码，我们将查看 Java 的函数式编程功能，这将帮助我们从我们的代码中删除许多行。我们拥有的代码越少，维护应用程序就越容易；因此，程序员喜欢函数式编程。但这并不是函数式编程如此受欢迎的唯一原因。它也是以更可读和更不易出错的方式描述某些算法的绝佳方式，比传统的循环更好。

函数式编程不是新事物。它在 20 世纪 30 年代为其开发了数学背景。第一个（如果不是第一个）函数式编程语言是 LISP。它在 20 世纪 50 年代开发，至今仍在使用，以至于有该语言在 JVM 上的一个版本（Clojure）。

简而言之，函数式编程意味着我们用函数来表示程序结构。在这个意义上，我们应该将函数视为数学中的函数，而不是像 C 等编程语言中使用的术语。在 Java 中，我们有方法，当我们遵循函数式编程范式时，我们创建并使用像数学函数一样行为的方法。一个方法是函数式的，如果无论我们调用多少次它都给出相同的结果，就像*sin(0)*总是零一样。函数式编程避免了改变对象的状态，因为状态没有改变，结果总是相同的。这也简化了调试。

如果一个函数对于给定的参数已经返回了一个特定的值，它将始终返回相同的值。我们也可以将代码读作是对计算的声明，而不是依次执行的命令。如果执行顺序不重要，那么代码的可读性也可能提高。

Java 通过 lambda 表达式和流操作帮助实现函数式编程风格。请注意，这些流不是 I/O 流，实际上与那些流没有真正的关联。

我们将首先简要了解 lambda 表达式和流操作是什么，然后，我们将将程序的一些部分转换为使用这些编程结构。我们还将看到这些代码的可读性提高了多少。

可读性是一个有争议的话题。一段代码可能对一个开发者来说是可读的，而对另一个开发者来说可能不太可读。这很大程度上取决于他们习惯了什么。我多次经历过开发者被流操作分散注意力。当开发者第一次遇到流操作时，他们思考流的方式以及流的外观都是陌生的。但这就像刚开始学习骑自行车一样。当你还在学习如何使用它并且摔倒的次数比滚动次数多时，它肯定比走路慢。另一方面，一旦你学会了如何骑自行车...

# Lambda

我们已经在第三章，*优化排序 - 使代码专业化*中使用了 lambda 表达式，当时我们编写了异常抛出测试。在那段代码中，我们将比较器设置为在每次调用时抛出`RuntimeException`的特殊值：

```java
sort.setComparator((String a, String b) -> { 
        throw new RuntimeException(); 
    });

```

参数类型是`Comparator`；因此，我们必须要设置的是一个实现了`java.util.Comparator`接口的类的实例。该接口只定义了一个实现必须定义的方法：`compare`。因此，我们可以将其定义为 lambda 表达式。如果没有 lambda，如果我们需要一个实例，我们必须输入很多。我们必须创建一个类，给它命名，在其中声明`compare`方法，并编写方法的主体，如下面的代码段所示：

```java
public class ExceptionThrowingComparator implements Comparator { 
  public int compare(T o1, T o2){ 
    throw new RuntimeException(); 
  } 
}

```

在使用位置，我们应该实例化类并将其作为参数传递：

```java
sort.setComparator(new ExceptionThrowingComparator());

```

如果我们将类定义为匿名类，我们可以节省一些字符，但开销仍然存在。我们真正需要的是必须定义的那个单一方法的主体。这就是 lambda 出现的地方。

我们可以在任何需要定义只有一个方法的类的实例的地方使用 lambda 表达式。从`Object`继承的方法不算，我们也不关心接口中定义的作为`default`方法的方法。它们就在那里。Lambda 定义了尚未定义的那个。换句话说，lambda 以比匿名类更少的开销清晰地描述了值是一个作为参数传递的功能。

lambda 表达式的简单形式如下：

```java
parameters -> body

```

参数可以放在括号内，也可以不放在括号内。同样，主体可以放在 `{` 和 `}` 字符之间，也可以是一个简单的表达式。这样，lambda 表达式可以将开销降到最低，只在真正需要的地方使用括号。

lambda 表达式还有一个极其有用的特性，即如果我们使用的表达式上下文已经很明显，我们不需要指定参数的类型。因此，前面的代码段甚至可以更短，如下所示：

```java
sort.setComparator((a, b) -> { 
    throw new RuntimeException(); 
});

```

参数 `a` 和 `b` 将具有所需的类型。为了使其更加简单，如果只有一个参数，我们甚至可以省略参数周围的 `(` 和 `)` 字符。

如果有多个参数，括号不是可选的。这是为了避免在某些情况下产生歧义。例如，方法调用 `f(x,y->x+y)` 可能是一个有两个参数的方法：`x` 和一个只有一个参数的 lambda 表达式 `y`。同时，它也可能是一个有两个参数的 lambda 表达式的方法调用，参数分别是 `x` 和 `y`。

当我们想要将功能作为参数传递时，lambda 表达式非常方便。在方法声明的地方声明参数类型应该是函数式接口类型。这些接口可以选择性地使用 `@FunctionalInterface` 进行注解。Java 运行时在 `java.util.function` 包中定义了许多这样的接口。我们将在下一节讨论其中的一些，以及它们在流中的使用。其余的，可以通过 Oracle 的标准 Java 文档获取。

# 流（Streams）

流（Streams）也是 Java 8 的新特性，就像 lambda 表达式一样。它们之间有着非常紧密的协作，因此它们同时出现并不令人惊讶。lambda 表达式和流都支持函数式编程风格。

首先要明确的是，流与输入输出流没有任何关系，除了名称。它们是完全不同的东西。流更像是集合，但有一些显著的不同。（如果没有差异，它们就只是集合。）流本质上是一系列可以顺序或并行运行的运算管道。它们从集合或其他来源获取数据，包括即时生成的数据。

流支持对多个数据执行相同的计算。这种结构被称为**单指令多数据**（**SIMD**）。不要害怕这个表达式。这实际上是一个非常简单的事情。我们在这本书中已经多次这样做过了。循环也是一种 SIMD 结构。当我们遍历检查类以查看是否有任何反对顺序的检查时，我们对每个检查执行相同的指令。多个检查器就是多个数据。

循环的一个问题是，我们在不需要时定义了执行顺序。在棋盘游戏的情况下，我们并不关心棋子执行的顺序。我们只关心所有棋子都按照顺序执行。我们仍然在编写循环时指定了某些顺序。这是循环的本质，我们无法改变这一点。这就是它们的工作方式。然而，如果我们能够以某种方式简单地表示“对每个棋子执行这个和那个”，那将是非常好的。这是流出现的一个地方。

另一点是，使用循环的代码更偏向于命令式而不是描述性。当我们阅读循环结构的程序时，我们关注的是各个步骤。我们首先看到循环中的命令做了什么。这些命令作用于数据中的单个元素，而不是整个集合或数组。

在我们的大脑中将各个步骤组合起来后，我们意识到整体图景是什么，循环的目的何在。在流的情况下，操作的描述是一个更高的层次。一旦我们学会了流方法，阅读它们就更容易了。流方法作用于整个流而不是单个元素，因此更具描述性。

`java.lang.Stream` 是一个接口。实现此接口的类型对象代表许多对象，并提供可以用于对这些对象执行指令的方法。当我们对其中一个对象开始操作时，这些对象可能可用，也可能不可用，或者可能仅在需要时创建。这取决于 `Stream` 接口的实际实现。例如，假设我们使用以下代码生成包含 `int` 值的流：

```java
IntStream.iterate( 0, (s) -> s+1 )

```

在前面的代码片段中，由于流包含无限数量的元素，因此无法生成所有元素。此示例将返回数字 0、1、2 等等，直到进一步的非列表流操作终止计算。

当我们编程 `Stream` 时，我们通常从一个 `Collection` 创建一个流——不是总是这样，但很多时候。Java 8 中扩展了 `Collection` 接口以提供 `stream` 和 `parallelStream` 方法。这两个方法都返回代表集合元素的流对象。当存在自然顺序时，`stream` 返回与集合中相同的顺序的元素，而 `parallelStream` 创建一个可以并行处理的流。在这种情况下，如果我们使用的某些流方法是以这种方式实现的，代码就可以利用计算机中可用的多个处理器。

一旦我们有了流，我们就可以使用 `Stream` 接口定义的方法。首先开始的是 `forEach` 方法。此方法有一个参数，通常作为 lambda 表达式提供，并将对流的每个元素执行 lambda 表达式。

在`Checker`类中，我们有`isConsistent`方法。在这个方法中，有一个遍历 checker 类注解的循环。如果我们想记录循环中注解实现的接口，我们可以添加以下内容：

```java
for (ConsistencyChecker checker :checkers) { 
  for (Annotation annotation : 
checker.getClass().getAnnotations()) { 
Arrays.stream(annotation.getClass().getInterfaces()) 
.forEach( 
t ->log.info("annotation implemented interfaces {}",t) 
); 
...

```

在这个例子中，我们使用`Arrays`类的工厂方法从一个数组创建一个流。该数组包含由反射方法`getInterfaces`返回的接口。lambda 表达式只有一个参数；因此，我们不需要在它周围使用括号。表达式的主体是一个返回无值的调用；因此，我们也省略了`{`和`}`字符。

为什么会有这么多麻烦？有什么好处？为什么我们不能只写一个简单的循环来记录数组的元素？

获益在于可读性和可维护性。当我们创建一个程序时，我们必须关注程序应该做什么，而不是它应该如何做。在一个理想的世界里，规范应该只是可执行的。我们可能在将来实现这一点，那时编程工作将由人工智能取代。（当然不是程序员。）我们还没有达到那里。我们必须告诉计算机如何完成我们想要实现的事情。我们过去不得不在 PDP-11 的控制台上输入二进制代码，以便将机器代码部署到内存中执行。后来，我们有了汇编器；再后来，我们有了 FORTRAN 和其他高级编程语言，它们取代了 40 年前的大部分编程工作。所有这些编程发展都使方向从“怎么做”转向了“做什么”。今天，我们使用 Java 9 编程，这条路还有很长的路要走。

我们能更多地表达“做什么”而不是“怎么做”，我们的程序就会越短、越容易理解。它将包含本质，而不是一些机器为了仅仅完成我们想要的事情而需要的人工垃圾。

当我看到我需要维护的代码中的一个循环时，我假设循环执行的顺序很重要。可能根本不重要。几秒钟后可能就显而易见了。可能需要几分钟甚至更长时间才能意识到顺序并不重要。这种时间是浪费的，可以通过更好的表达“做什么”的编程结构来节省，而不是“怎么做”。

# 函数式接口

该方法的参数应该是`java.util.function.Consumer`。这是一个需要定义`accept`方法的接口，该方法返回`void`。lambda 表达式或实现此接口的类将“消费”`accept`方法的参数，不产生任何内容。

在那个包中定义了几个其他接口，每个接口都作为功能接口使用，用于描述可以作为实际参数给出的 lambda 表达式的方法参数。

例如，`Consumer`的对立面是`Supplier`。这个接口有一个名为`get`的方法，它不需要任何参数，但会返回一些`Object`作为返回值。

如果有一个参数和一个返回值，那么这个接口就被称为`Function`。如果返回值必须与参数具有相同的类型，那么`UnaryOperator`接口就是我们的朋友。同样，还有一个`BinaryOperator`接口，它返回与参数相同类型的对象。正如我们从`Function`到`UnaryOperator`所看到的那样，在另一个方向上，如果参数和返回值不共享类型，也存在`BiFunction`。

这些接口并不是相互独立定义的。如果一个方法需要`Function`，而我们手头有`UnaryOperator`可以传递，那么这不应该是个问题。`UnaryOperator`实际上就是具有相同类型参数的`Function`。一个可以与`Function`一起工作，接受一个对象并返回一个对象的方法，如果它们具有相同的类型，那么应该不会有问题。这些可以是，但不必是，不同的。

为了让这种情况发生，`UnaryOperator`接口扩展了`Function`，因此可以在`Function`的位置使用。

我们在这个类中遇到的接口是使用泛型定义的。因为泛型类型不能是原始类型，所以操作原始值的接口应该单独定义。例如，`Predicate`是一个定义`booleantest(T t)`的接口。它是一个返回`boolean`值的函数，在流方法中被多次使用。

还有一些接口，例如`BooleanSupplier`、`DoubleConsumer`、`DoubleToIntFunction`等，它们与原始的`boolean`、`double`和`int`类型一起工作。不同参数类型和返回值的可能组合数量是无限的……几乎如此。

**有趣的事实**：非常精确地说，它并不是无限的。一个方法最多可以有 254 个参数。这个限制是在 JVM 中指定的，而不是在 Java 语言规范中。当然，没有一个是无用的。有 8 种原始类型（加上`Object`，加上可能少于 254 个参数的可能性），这意味着可能的函数式接口总数是 10²⁵⁴，上下几个数量级。实际上，是无限的！

我们不应该期望在这个包中定义所有可能的接口。这里只包含那些最有用的接口。例如，没有使用`short`或`char`的接口。如果我们需要类似的东西，我们可以在我们的代码中定义这个`interface`。或者，仔细思考并找出如何使用已经定义好的一个。 （在我的职业生涯中，我从未使用过`short`类型。它从未被需要过。）

这些功能接口在流中是如何使用的？`Stream`接口定义了具有某些功能接口类型作为参数的方法。例如，`allMatch`方法有一个`Predicate`参数，并返回一个`Boolean`值，如果流中的所有元素都匹配`Predicate`，则返回`true`。换句话说，此方法仅在`Predicate`作为参数返回`true`对于流中的每个元素时才返回`true`。

在下面的代码中，我们将使用流重写我们在示例代码中使用循环实现的一些方法，并通过这些示例，我们将讨论流提供的重要方法。我们保存了两个类，`ProductsCheckerCollector`和`ProductInformationCollector`，以展示流的使用。我们可以从这里开始。`ProductsCheckerCollector`遍历`Order`中包含的所有产品，并收集产品中列出的注释。每个产品可能包含零个、一个或多个注释。这些都在列表中。相同的注释可能被引用多次。为了避免重复，我们使用`HashSet`，即使产品中有多个实例，它也只包含元素的一个实例：

```java
public class ProductsCheckerCollector { 

    private final ProductInformationCollector pic; 
    public ProductsCheckerCollector(@Autowired 
      ProductInformationCollector pic) { this.pic = pic; } 

    public Set<Class<? extends Annotation>> 
                       getProductAnnotations(Order order) { 
        Map<OrderItem, ProductInformation> piMap = 
                          pic.collectProductInformation(order); 
        final Set<Class<? extends Annotation>> 
                            annotations = new HashSet<>(); 
        for (OrderItem item : order.getItems()) { 
            final ProductInformation pi = piMap.get(item); 
            if (pi != null && pi.getCheck() != null) { 
                for (Class<? extends Annotation> check : 
                                              pi.getCheck()) { 
                    annotations.addAll(pi.getCheck()); 
                } 
        } 
        return annotations; 
    } 
}

```

现在，让我们看看当我们使用流重新编写此方法时，它看起来如何：

```java
public Set<Class<? extends Annotation>> 
                getProductAnnotations(Order order) { 
    Map<OrderItem, ProductInformation> piMap = 
                      pic.collectProductInformation(order); 

    return order.getItems().stream() 
            .map(piMap::get) 
            .filter(Objects::nonNull) 
            .peek(pi -> { 
                if (pi.getCheck() == null) { 
                    log.info("Product {} has no annotation", 
                                                  pi.getId()); 
                } 
            }) 
            .filter(pi -> pi.getCheck() != null) 
            .peek(pi -> log.info("Product {} is annotated with class {}", pi.getId(), pi.getCheck())) 
            .flatMap(pi -> pi.getCheck().stream()) 
            .collect(Collectors.toSet()); 
}

```

该方法的主要工作被包含在一个单一、尽管庞大的流表达式中。我们将在接下来的几页中介绍表达式的元素。`order.getItems`返回的`List`通过调用`stream`方法进行转换：

```java
returnorder.getItems().stream()

```

正如我们之前简要提到的，`stream`方法是`Collection`接口的一部分。任何实现`Collection`接口的类都将拥有此方法，即使是在 Java 8 引入流之前实现的类也是如此。这是因为`stream`方法在接口中作为`default`方法实现。这样，如果我们偶然实现了一个实现此接口的类，即使我们不需要流，我们也会免费获得它作为额外的功能。

Java 8 中的`default`方法是为了支持接口的后向兼容性。JDK 的一些接口需要修改以支持 lambda 和函数式编程。一个例子是`stream`方法。在 Java 8 之前的特性集中，实现一些修改后接口的类应该被修改。它们将需要实现新方法。这种更改不是向后兼容的，Java 作为一种语言和 JDK 非常关注向后兼容性。因此，引入了`default`方法。这些方法允许开发者扩展接口并保持向后兼容性，为新的方法提供默认实现。

与此相反，Java 8 JDK 中的全新函数式接口也有`default`方法，尽管它们在 JDK 中没有先前的版本，因此没有兼容性问题。在 Java 9 中，接口也得到了扩展，现在它们不仅可以包含`default`和`static`方法，还可以包含`private`方法。这样，接口就变成了类似于抽象类的东西，尽管接口中没有字段，除了常量`static`字段。这种接口功能开放是一个备受批评的特性，它仅仅提出了其他允许多重类继承的语言所面临的编程风格和结构问题。Java 一直避免这种情况，直到 Java 8 和 Java 9。

从这个例子中我们能学到什么？要小心使用`default`方法和接口中的`private`方法。如果确实需要使用，请明智地使用它们。

这个流中的元素是`OrderItem`对象。我们需要为每个`OrderItem`提供`ProductInformation`。

# 方法引用

幸运的是，我们拥有`Map`，它将订单项与产品信息配对，因此我们可以对`Map`调用`get`方法：

```java
.map(piMap::get)

```

`map`方法在 Java 中与另一个具有相同名称的元素相关联，不应混淆。虽然`Map`类是一个数据结构，但`Stream`接口中的`map`方法执行流元素的映射。该方法的一个参数是`Function`（回想一下，这是一个我们最近讨论过的函数式接口）。这个函数将一个值`T`转换为另一个值`R`，这个值作为原始流（`Stream<T>`）的元素可用，`map`方法的返回值是`Stream<R>`。`map`方法使用给定的`Function<T,R>`将`Stream<T>`转换为`Stream<R>`，为原始流的每个元素调用它，并从转换后的元素创建一个新的流。

我们可以说，`Map`接口以静态方式在数据结构中将键映射到值，而`Stream`方法`map`则动态地将一种类型的值映射到另一种（或相同的）类型的值。

我们已经看到，我们可以以 lambda 表达式的形式提供一个函数式接口的实例。这个参数不是 lambda 表达式。这是一个方法引用。它表示`map`方法应该使用实际的流元素作为参数，在`Map piMap`上调用`get`方法。我们很幸运，`get`也需要一个参数，不是吗？我们也可以这样写：

```java
.map( orderItem ->piMap.get(orderItem))

```

然而，这将与`piMap::get`完全相同。

这样，我们可以引用一个在特定实例上工作的实例方法。在我们的例子中，这个实例是由`piMap`变量引用的。也可以引用`static`方法。在这种情况下，类名应写在`::`字符之前。当我们使用`Objects`类的`static`方法`nonNull`时，我们将很快看到这个例子（注意，类名是复数形式，它位于`java.util`包中，而不是`java.lang`包）。

也有可能引用一个实例方法，而不给出它应该调用的引用。这可以在功能接口方法有一个额外的第一个参数的地方使用，该参数将用作实例。我们已经在第三章，“优化排序 - 使代码专业化”中使用了这种方法，当时我们传递了`String::compareTo`，当期望的参数是一个`Comparator`时。`compareTo`方法期望一个参数，但`Comparator`接口中的`compare`方法需要两个参数。在这种情况下，第一个参数将用作`compare`必须调用的实例，第二个参数传递给`compare`。在这种情况下，`String::compareTo`与编写 lambda 表达式`(String a, String b) -> a.compareTo(b)`相同。

最后但同样重要的是，我们可以使用构造函数引用。当我们需要一个（让我们简单一点）`Object`的`Supplier`时，我们可以写`Object::new`。

下一步是从流中过滤掉`null`元素。注意，在这个阶段，流中包含`ProductInformation`元素：

```java
.filter(Objects::nonNull)

```

`filter`方法使用`Predicate`并创建一个只包含匹配谓词的元素的流。在这种情况下，我们使用了`static`方法的引用。`filter`方法不会改变流的类型。它只过滤掉元素。

我们接下来应用的方法有点反函数式。纯函数式流方法不会改变任何对象的任何状态。它们创建并返回新的对象，但除此之外，没有副作用。`peek`本身也没有不同，因为它只返回一个与它应用在上的相同元素的流。然而，这个“无操作”特性诱使新手程序员做一些非函数式的事情，并编写带有副作用的代码。毕竟，如果没有（副作用）在调用它时，为什么还要使用它呢？

```java
.peek(pi -> { 
    if (pi.getCheck() == null) { 
        log.info("Product {} has no annotation", pi.getId()); 
    } 
})

```

虽然`peek`方法本身没有副作用，但 lambda 表达式的执行可能会有。然而，这同样适用于任何其他方法。只是在这种情况下，做一些不恰当的事情更有诱惑力。不要这样做。我们是守纪律的成年人。正如方法名所暗示的，我们可能可以窥视流，但我们不应该做其他任何事情。在这种情况下，编程是一种特定的活动，窥视是恰当的。这正是我们在代码中实际做的事情：我们记录了一些信息。

然后，我们去除没有`ProductInformation`的元素；我们还想去除那些有，但没有定义检查器的元素：

```java
.filter(pi ->pi.getCheck() != null)

```

在这种情况下，我们不能使用方法引用。相反，我们使用 lambda 表达式。作为另一种解决方案，我们可以在`ProductInformation`中创建一个`boolean hasCheck`方法，当`private`字段检查不是`null`时返回`true`。这将如下所示：

```java
.filter(ProductInformation::hasCheck)

```

这完全有效且可行，尽管该类没有实现任何函数式接口并且有很多方法，不仅仅是这个方法。然而，方法引用是明确的，并指定了要调用的方法。

在这个第二个过滤器之后，我们再次记录元素：

```java
.peek(pi -> log.info( 
     "Product {} is annotated with class {}", pi.getId(), 
                                            pi.getCheck()))

```

下一个方法是`flatMap`，这是一个特别且不易理解的东西。至少对我来说，它比学习函数式编程时的`map`和`filter`要困难一些：

```java
.flatMap(pi ->pi.getCheck().stream())

```

此方法期望传入的 lambda 表达式、方法引用或其他作为参数传递的内容，为原始流中每个元素创建一个新的对象流。然而，结果并不是流中的流，尽管这也是可能的，而是返回的流被连接成一个巨大的流。

如果我们应用此流的流是整数流，例如 1，2，3，...，并且每个数字*n*的函数返回一个包含三个元素*n*，*n+1*和*n+2*的流，那么`flatMap`产生的流将包含 1，2，3，2，3，4，3，4，5，4，5，6，等等。

最后，我们应该将流收集到一个`Set`中。这是通过调用`collector`方法来完成的：

```java
.collect(Collectors.toSet());

```

`collector`方法的参数是（再次是名称滥用）`Collector`。它可以用来收集流中的元素到某个集合中。请注意，`Collector`不是一个函数式接口。您不能仅仅使用 lambda 表达式或简单的方法来收集某些内容。为了收集元素，我们确实需要一个地方来收集元素，因为随着从流中来的新元素，元素会被收集。`Collector`接口并不简单。幸运的是，`java.util.streams.Collectors`类（再次注意复数形式）有很多`static`方法，这些方法创建并返回创建并返回`Collector`对象的`Object`。

其中之一是`toSet`，它返回一个`Collector`，帮助将流中的元素收集到一个`Set`中。当所有元素都到达时，`collect`方法将返回`Set`。还有其他方法可以帮助通过求和元素、计算平均值或将元素收集到`List`、`Collection`或`Map`中。将元素收集到`Map`中是一件事，因为`Map`的每个元素实际上是一个键值对。当我们查看`ProductInformationCollector`时，我们将看到这个示例。

`ProductInformationCollector`类代码中包含`collectProductInformation`方法，我们将从`Checker`类以及`ProductsCheckerCollector`类中使用该方法：

```java
private Map<OrderItem, ProductInformation> map = null; 

public Map<OrderItem, ProductInformation>  
                  collectProductInformation(Order order) { 
    if (map == null) { 
        map = new HashMap<>(); 
        for (OrderItem item : order.getItems()) { 
            final ProductInformation pi = 
                     lookup.byId(item.getProductId()); 
            if (!pi.isValid()) { 
                map = null; 
                return null; 
            } 
            map.put(item, pi); 
        } 
    } 
    return map; 
}

```

简单的技巧是将收集的值存储在`Map`中，如果该值不是`null`，则只需返回已计算出的值，这可能在处理相同的 HTTP 请求时多次调用此方法时节省大量的服务调用。

有两种方式来编码这样的结构。一种是通过检查`Map`的非空性，如果`Map`已经存在则返回。这种模式被广泛使用，并且有一个名字。这被称为保护`if`。在这种情况下，方法中有多于一个的返回语句，这可能会被视为一种弱点或反模式。另一方面，方法的缩进应该比之前浅一个制表符。

这纯粹是个人喜好问题。如果你发现自己正处于关于一个或另一个解决方案的辩论中，就请自己行个方便，让你的同伴在这件事上获胜，并为你节省精力，用于更重要的问题，例如，你是否应该使用流还是仅仅使用普通的循环。

现在，让我们看看如何将这个解决方案转换为函数式风格：

```java
public Map<OrderItem, ProductInformation> collectProductInformation(Order order) { 
    if (map == null) { 
        map = 
        order.getItems() 
                .stream() 
                .map(item -> tuple(item, item.getProductId())) 
                .map(t -> tuple(t.r, lookup.byId((String) t.s))) 
                .filter(t -> ((ProductInformation)t.s).isValid()) 
                .collect( 
                    Collectors.toMap( t -> (OrderItem)t.r, 
                                      t -> (ProductInformation)t.s 
                                    ) 
                ); 
        if (map.keySet().size() != order.getItems().size()) { 
            log.error("Some of the products in the order do not have product information, {} != {} ",map.keySet().size(),order.getItems().size()); 
            map = null; 
        } 
    } 
    return map; 
}

```

我们使用一个辅助类`Tuple`，它只是两个名为`r`和`s`的`Object`实例。我们稍后会列出这个类的代码。它非常简单。

在流表达式，我们首先从集合中创建流，然后将`OrderItem`元素映射到`OrderItem`和`productId`元组的流。然后我们将这些元组映射到现在包含`OrderItem`和`ProductInformation`的元组。这两个映射可以在一个映射调用中完成，这将只执行这两个步骤。我决定创建两个，希望这样可以使每行的步骤更简单，从而使得生成的代码更容易理解。

过滤步骤也并非什么新鲜事物。它只是过滤掉无效的产品信息元素。实际上，不应该有任何。如果订单包含一个指向不存在产品的订单 ID，就会发生这种情况。在下一个语句中，我们会检查收集到的产品信息元素的数量，以确保所有项目都有适当的信息。

有趣的代码是如何将流元素收集到一个`Map`中。为了做到这一点，我们再次使用`collect`方法和`Collectors`类。这次，`toMap`方法创建了一个`Collector`。这需要两个`Function`结果表达式。第一个应该将流元素转换为键，第二个应该产生用于`Map`中的值。因为键和值的实际类型是从传递的 lambda 表达式的结果计算出来的，所以我们必须显式地将元组的字段转换为所需类型。

最后，简单的`Tuple`类如下：

```java
public class Tuple<R, S> { 
    final public R r; 
    final public S s; 

    private Tuple(R r, S s) { 
        this.r = r; 
        this.s = s; 
    } 

    public static <R, S> Tuple tuple(R r, S s) { 
        return new Tuple<>(r, s); 
    } 
}

```

在我们的代码中，还有一些类值得转换为函数式风格。这些是`Checker`和`CheckerHelper`类。

在`Checker`类中，我们可以重写`isConsistent`方法：

```java
public boolean isConsistent(Order order) { 
    Map<OrderItem, ProductInformation> map = 
                  piCollector.collectProductInformation(order); 
    if (map == null) { return false; } 
    final Set<Class<? extends Annotation>> annotations = 
                       pcCollector.getProductAnnotations(order); 
    return !checkers.stream().anyMatch( 
                 checker -> Arrays.stream( 
                              checker.getClass().getAnnotations() 
                            ).filter( 
                              annotation -> 
                                annotations.contains( 
                                      annotation.annotationType()) 
                            ).anyMatch( 
                              x ->  
                                checker.isInconsistent(order) 
                            )); 
}

```

由于你已经学会了大多数重要的流方法，这里几乎没有什么新问题。我们可以提到`anyMatch`方法，如果至少有一个元素使得传递给`anyMatch`的`Predicate`参数为`true`，则它将返回`true`。它可能还需要一些调整，以便我们可以在另一个流中使用流。这很可能是一个流表达式过于复杂，需要使用局部变量将其拆分成更小部分的例子。

最后，在我们离开函数式风格之前，我们重新编写了`CheckHelper`类中的`containsOneOf`方法。这个方法没有引入新元素，将帮助你检查关于`map`、`filter`、`flatMap`和`Collector`所学的知识。请注意，正如我们讨论的那样，如果`order`包含至少一个作为字符串给出的订单 ID，则此方法返回`true`。

```java
public boolean containsOneOf(String... ids) { 
    return order.getItems().stream() 
            .map(OrderItem::getProductId) 
            .flatMap(itemId -> Arrays.stream(ids) 
                    .map(id -> tuple(itemId, id))) 
            .filter(t -> Objects.equals(t.s, t.r)) 
            .collect(Collectors.counting()) > 0; 
}

```

我们创建了`OrderItem`对象的流，并将其映射到包含在该流中的产品 ID 的流。然后，我们为每个 ID 创建另一个流，包含 ID 的元素和作为参数给出的一个字符串 ID。接着，我们将这些子流展平成一个流。这个流将包含`order.getItems().size()`乘以`ids.length`个元素：所有可能的配对。我们将过滤掉包含相同 ID 两次的配对，最后，我们将计算流中元素的数量。

# Java 9 中的脚本

我们几乎完成了本章的示例程序。不过有一个问题，尽管它不是专业的。当我们有一个需要新检查器的新产品时，我们必须创建代码的新版本。

在专业环境中，程序有发布。当代码被修改、错误被修复或实现了一个新功能时，组织在应用程序可以投入生产之前需要执行许多步骤。这些步骤构成了发布过程。一些环境有轻量级的发布流程；而另一些则需要严格且昂贵的检查。这并不是因为组织内部人员的口味。当非工作生产代码的成本很低，程序出现故障或功能错误并不重要时，发布流程可以很简单。这样，发布可以更快、更便宜地完成。例如，一些用户用于娱乐的聊天程序。在这种情况下，发布新功能可能比确保无错误运行更重要。在另一端，如果你创建的代码控制着核电站，失败的成本可能相当高。即使是微小的更改，也要进行严肃的测试和仔细检查所有功能，这可能会带来回报。

在我们的例子中，简单的检查器可能是一个不太可能引起严重错误的问题领域。虽然并非不可能，但代码如此简单……是的，我知道这样的论点有点牵强，但让我们假设这些小例程可以通过比代码的其他部分更少的测试和更简单的方式进行更改。那么，如何将这些小脚本的代码分离出来，以便它们不需要技术发布、应用程序的新版本，甚至不需要重新启动应用程序？我们有一个新产品需要新的检查，我们希望有一种方法可以将这个检查注入到应用程序环境中，而不会造成任何服务中断。

我们选择的解决方案是脚本。Java 程序可以执行用*JavaScript*、*Groovy*、*Jython*（这是*Python*语言的*JVM*版本）和许多其他语言编写的脚本。除了*JavaScript*之外，这些语言的解释器不是 JDK 的一部分，但它们都提供了一个标准接口，该接口在 JDK 中定义。结果是，我们可以在我们的代码中实现脚本执行，提供脚本的开发者可以自由选择任何可用的语言；我们不需要关心执行*JavaScript*代码。我们将使用与执行*Groovy*或*Jython*相同的 API。我们唯一需要知道的是脚本使用的语言。这通常很简单：我们可以从文件扩展名中猜测，如果猜测不够，我们可以要求脚本开发者将*JavaScript*放入以`.js`扩展名命名的文件中，*Jython*放入以`.jy`或`.py`扩展名命名的文件中，*Groovy*放入以`.groovy`扩展名命名的文件中，依此类推。同样重要的是要注意，如果我们想让我们的程序执行这些语言之一，我们应该确保解释器在类路径上。在*JavaScript*的情况下，这是既定的；因此，在本章的演示中，我们将用*JavaScript*编写我们的脚本。不会有太多；毕竟，这是一本 Java 书，而不是*JavaScript*书。

脚本通常是我们想要通过编程方式配置或扩展应用程序功能时的一个不错的选择。这正是我们现在的情形。

我们必须做的第一件事是扩展生产信息。如果有脚本检查产品中订单的一致性，我们需要一个字段来指定脚本的名称：

```java
    private String checkScript; 
    public String getCheckScript() { 
        return checkScript; 
    } 
    public void setCheckScript(String checkScript) { 
        this.checkScript = checkScript; 
    }

```

我们不希望每个产品指定多个脚本；因此，我们不需要脚本名称列表。我们只指定了一个名为的脚本。

坦白说，检查器类和注释的数据结构太复杂了，允许每个产品以及每个检查器类有多个注释。我们虽然无法避免这一点，但需要足够复杂的结构来展示流表达式的强大功能和能力。现在我们已经过了这个主题，我们可以继续使用更简单的数据结构，专注于脚本执行。

我们还必须修改`Checker`类，使其不仅使用检查器类，还使用脚本。我们不能丢弃检查器类，因为当我们意识到我们最好需要脚本时，我们已经有大量的检查器类，我们没有资金将它们重写为脚本。是的，我们是在书中，而不是现实生活中，但在企业中，情况就是这样。这就是为什么在设计企业解决方案时你应该非常小心。结构和解决方案将长期存在，而且很难仅仅因为技术上不是最好的就丢弃一段代码。如果它有效并且已经存在，业务将非常不愿意在代码维护和重构上花钱。

摘要：我们修改了`Checker`类。我们需要一个新的类来执行我们的脚本；因此，构造函数被修改：

```java
private final CheckerScriptExecutor executor; 

    public Checker( 
        @Autowired Collection<ConsistencyChecker> checkers, 
        @Autowired ProductInformationCollector piCollector, 
        @Autowired ProductsCheckerCollector pcCollector, 
        @Autowired CheckerScriptExecutor executor ) { 
        this.checkers = checkers; 
        this.piCollector = piCollector; 
        this.pcCollector = pcCollector; 
        this.executor = executor; 
    }

```

我们还必须在`isConsistent`方法中使用这个`executor`：

```java
public boolean isConsistent(Order order) { 
        final Map<OrderItem, ProductInformation> map = 
                piCollector.collectProductInformation(order); 
        if (map == null) { 
            return false; 
        } 
        final Set<Class<? extends Annotation>> annotations = 
                pcCollector.getProductAnnotations(order); 
        Predicate<Annotation> annotationIsNeeded = annotation -> 
                annotations.contains(annotation.annotationType()); 
        Predicate<ConsistencyChecker> productIsConsistent = 
                checker -> 
                Arrays.stream(checker.getClass().getAnnotations()) 
                        .parallel().unordered() 
                        .filter(annotationIsNeeded) 
                        .anyMatch( 
                             x -> checker.isInconsistent(order)); 
        final boolean checkersSayConsistent = !checkers.stream(). 
                anyMatch(productIsConsistent); 
        final boolean scriptsSayConsistent = 
                !map.values(). 
                        parallelStream(). 
                        map(ProductInformation::getCheckScript). 
                        filter(Objects::nonNull). 
                        anyMatch(s -> 
                           executor.notConsistent(s,order)); 
        return checkersSayConsistent && scriptsSayConsistent; 
    }

```

注意，在这段代码中，我们使用并行流，因为为什么不呢？只要可能，我们都可以使用并行流，即使是无序的，以告知底层系统以及维护代码的程序员同行，顺序并不重要。

我们还修改了我们产品的一个 JSON 文件，通过一些注释来引用脚本而不是检查器类：

```java
{ 
  "id" : "124", 
  "title": "Desk Lamp", 
  "checkScript" : "powered_device", 
  "description": "this is a lamp that stands on my desk", 
  "weight": "600", 
  "size": [ "300", "20", "2" ] 
}

```

即使是 JSON 也更为简单。请注意，由于我们决定使用 JavaScript，在命名脚本时我们不需要指定文件名扩展名。

我们可能稍后考虑进一步的开发，届时我们将允许产品检查脚本维护者使用不同的脚本语言。在这种情况下，我们可能仍然要求他们指定扩展名，如果没有扩展名，我们的程序将自动添加为`.js`。在我们的当前解决方案中，我们没有检查这一点，但我们可能会花几秒钟时间思考，以确保解决方案可以进一步开发。重要的是，我们不要为了进一步开发而开发额外的代码。开发者不是算命先生，无法可靠地预测未来的需求。这是业务人员的任务。

我们将脚本放入我们项目的`scripts`目录下的`resource`目录中。文件名必须是`powered_device.js`，因为这是我们已在 JSON 文件中指定的名称：

```java
function isInconsistent(order){ 
    isConsistent = false 
    items = order.getItems() 
    for( i in items ){ 
    item = items[i] 
    print( item ) 
        if( item.getProductId() == "126" || 
            item.getProductId() == "127" || 
            item.getProductId() == "128"  ){ 
            isConsistent = true 
            } 
    } 
    return ! isConsistent 
}

```

这是一个极其简单的 JavaScript 程序。作为旁注，当你使用 JavaScript 遍历列表或数组时，循环变量将遍历集合或数组的索引。由于我很少在 JavaScript 中编程，我陷入了这个陷阱，调试我犯的错误花了我超过半小时的时间。

我们已经准备好了调用脚本所需的一切。我们仍然需要调用它。为此，我们使用 JDK 脚本 API。首先，我们需要一个`ScriptEngineManager`。这个管理器用于获取访问 JavaScript 引擎的权限。尽管 JavaScript 解释器自 Java 7 以来一直是 JDK 的一部分，但它仍然以抽象的方式管理。它是 Java 程序可以使用的许多可能的解释器之一，用于执行脚本。它恰好存在于 JDK 中，所以我们不需要将解释器 JAR 添加到类路径中。`ScriptEngineManager`发现类路径上的所有解释器并将它们注册。

它使用服务提供者规范来实现这一点，这个规范已经很长时间是 JDK 的一部分，到了 Java 9，它在模块处理方面也得到了额外的支持。这要求脚本解释器实现`ScriptEngineFactory`接口，并在`META-INF/services/javax.script.ScriptEngineFactory`文件中列出实现它的类。这些文件，从所有构成类路径的 JAR 文件中，都被`ScriptEngineManager`作为资源读取，通过这种方式，它知道哪些类实现了脚本解释器。`ScriptEngineFactory`接口要求解释器提供诸如`getNames`、`getExtensions`和`getMimeTypes`等方法。管理器调用这些方法来收集关于解释器的信息。当我们询问 JavaScript 解释器时，管理器将返回由工厂创建的，并声称其名称之一是`JavaScript`的解释器。

通过名称、文件名扩展名或 MIME 类型来访问解释器只是`ScriptEngineManager`的功能之一。另一个功能是管理`Bindings`。

当我们从 Java 代码内部执行脚本时，我们并不是因为我们想增加我们的多巴胺水平。在脚本的情况下，这种情况不会发生。我们想要一些结果。我们想要传递参数，并在脚本执行后，我们想要从脚本中获取我们可以用于 Java 代码的值。这可以通过两种方式发生。一种是通过将参数传递给脚本中实现的方法或函数，并从脚本中获取返回值。这通常有效，但甚至可能发生某些脚本语言甚至没有函数或方法的观念。在这种情况下，这不是一个可能性。可能的是，将一些环境传递给脚本，并在脚本执行后从环境中读取值。这个环境由`Bindings`表示。

`Bindings`是一个具有`String`键和`Object`值的映射。

在大多数脚本语言的情况下，例如在 JavaScript 中，`Bindings`与我们在执行的脚本中的全局变量连接。换句话说，如果我们在我们 Java 程序中在调用脚本之前执行以下命令，那么 JavaScript 全局变量`globalVariable`将引用`myObject`对象：

```java
myBindings.put("globalVariable",myObject)

```

我们可以创建`绑定`并将其传递给`ScriptEngineManager`，同样我们也可以使用它自动创建的那个，并且我们可以直接在引擎对象上调用`put`方法。

当我们执行脚本时，有两种`绑定`。一种是设置在`ScriptEngineManager`级别上的。这被称为全局绑定。还有一个由`ScriptEngine`本身管理的。这是局部`绑定`。从脚本的角度来看，没有区别。从嵌入的角度来看，有一些区别。如果我们使用同一个`ScriptEngineManager`来创建多个`ScriptEngine`实例，那么全局绑定将由它们共享。如果一个获取了值，所有其他实例都将看到相同的值；如果一个设置了值，所有其他实例稍后都将看到这个改变后的值。局部绑定特定于它所管理的引擎。由于我们在这本书中只介绍了 Java 脚本 API，所以我们不会深入探讨，并且我们不会使用`绑定`。我们只需要调用 JavaScript 函数并从中获取结果即可。

实现脚本调用的类是`CheckerScriptExecutor`：

```java
package packt.java9.by.example.mybusiness.bulkorder.services; 

import ... 

@Component 
public class CheckerScriptExecutor { 
    private static final Logger log = ... 

    private final ScriptEngineManager manager = 
                             new ScriptEngineManager(); 

    public boolean notConsistent(String script, Order order) { 

        try { 
            final Reader scriptReader = getScriptReader(script); 
            final Object result =  
                         evalScript(script, order, scriptReader); 
            assertResultIsBoolean(script, result); 
            log.info("Script {} was executed and returned {}", 
                                                 script, result); 
            return (boolean) result; 

        } catch (Exception wasAlreadyHandled) { 
            return true; 
        } 
    }

```

唯一的`public`方法`notConsistent`获取要执行的脚本的名称以及`order`。后者必须传递给脚本。首先它获取`Reader`，可以读取脚本文本，评估它，并在结果是`boolean`或至少可以转换为`boolean`的情况下最终返回结果。如果我们在这个类中实现并调用的任何方法出现错误，它将抛出异常，但只有在适当记录之后才会这样做。在这种情况下，安全的方式是拒绝订单。

实际上，这是业务应该决定的事情。如果有一个无法执行的检查脚本，这显然是一个错误的情况。在这种情况下，接受订单并在之后手动处理问题是有一定成本的。由于某些内部错误而拒绝订单或确认也不是订单流程中的愉快路径。我们必须检查哪种方法对公司的损害最小。这当然不是程序员的职责。在我们的情况下，我们处于一个简单的情况。

我们假设业务代表说在这种情况下应该拒绝订单。在现实生活中，类似的决策很多时候是由业务代表拒绝的，他们说这根本不应该发生，IT 部门必须确保程序和整个操作完全无错误。这种反应有一个心理原因，但这确实让我们离 Java 编程非常远。

引擎可以通过`Reader`或作为`String`执行传递给它的脚本。因为现在我们的脚本代码在资源文件中，似乎让引擎读取资源而不是将其读取到`String`中是一个更好的主意：

```java

        private Reader getScriptReader(String script) 
                                throws IOException { 
        final Reader scriptReader; 
        try { 
            final InputStream scriptIS = new ClassPathResource( 
                    "scripts/" + script + ".js").getInputStream(); 
            scriptReader = new InputStreamReader(scriptIS); 
        } catch (IOException ioe) { 
            log.error("The script {} is not readable", script); 
            log.error("Script opening exception", ioe); 
            throw ioe; 
        } 
        return scriptReader; 
    }

```

要从资源文件中读取脚本，我们使用 Spring 的`ClassPathResource`类。脚本的名称在`scripts`目录前缀，以`.js`扩展名后缀。其余部分相当标准，我们在这本书中已经见过。下一个评估脚本的方程序更有趣：

```java
        private Object evalScript(String script, 
                              Order order, 
                              Reader scriptReader)  
            throws ScriptException, NoSuchMethodException { 
        final Object result; 
        final ScriptEngine engine = 
                          manager.getEngineByName("JavaScript"); 
        try { 
            engine.eval(scriptReader); 
            Invocable inv = (Invocable) engine; 
            result = inv.invokeFunction("isInconsistent", order); 
        } catch (ScriptException | NoSuchMethodException se) { 
            log.error("The script {} thruw up", script); 
            log.error("Script executing exception", se); 
            throw se; 
        } 
        return result; 
    }

```

要执行脚本中的方法，首先我们需要一个能够处理**JavaScript**的脚本引擎。我们通过名称从管理器获取引擎。如果不是**JavaScript**，我们应该检查返回的`engine`不是`null`。在**JavaScript**的情况下，解释器是**JDK**的一部分，检查**JDK**是否符合标准可能会有些过度谨慎。

如果我们想要扩展这个类来处理不仅**JavaScript**，还要处理其他类型的脚本，这个检查必须进行，并且脚本引擎可能需要通过文件名扩展从管理器请求，而我们在这个`private`方法中无法访问这个扩展。但这将是未来的开发，而不是这本书的内容。

当我们拥有引擎时，我们必须评估脚本。这将定义脚本中的函数，以便我们之后可以调用它。为了调用它，我们需要一个`Invocable`对象。在**JavaScript**的情况下，引擎也实现了`Invocable`接口。并非所有脚本引擎都实现了这个接口。有些脚本没有函数或方法，其中没有可以调用的内容。同样，这也是我们稍后要做的事情，当我们想要允许不仅**JavaScript**脚本，还要允许其他类型的脚本时。

要调用函数，我们将它的名称传递给`invokeFunction`方法，并传递我们想要传递的参数。在这种情况下，这是`order`。在**JavaScript**的情况下，两种语言之间的集成相当成熟。正如我们的示例所示，我们可以访问作为参数传递的 Java 对象的字段和方法，并且返回的 JavaScript `true`或`false`值也会神奇地转换为`Boolean`。尽管有些情况下访问并不那么简单：

```java

private void assertResultIsBoolean(String script, 
                                       Object result) { 
        if (!(result instanceof Boolean)) { 
            log.error("The script {} returned non boolean", 
                                                    script); 
            if (result == null) { 
                log.error("returned value is null"); 
            } else { 
                log.error("returned type is {}", 
                                 result.getClass()); 
            } 
            throw new IllegalArgumentException(); 
        } 
    } 
}

```

类的最后一个方法检查返回的值，由于这是一个脚本引擎，它可以是一切，确保它可以转换为`boolean`。

需要注意的是，虽然一些功能是用脚本实现的，但这并不能保证应用程序能够无缝运行。可能会有多个问题，脚本可能会影响整个应用程序的内部工作。一些脚本引擎提供了特殊的方法来保护应用程序免受恶意脚本的影响，而其他则没有。我们不对脚本进行传递，而是进行命令，但这并不能保证脚本不能访问其他对象。使用反射、`static`方法和其他技术，我们可以在 Java 程序内部访问任何东西。当我们的代码库中只有脚本发生变化时，我们可能会在测试周期中更容易一些，但这并不意味着我们应该盲目地信任任何脚本。

在我们的例子中，让产品的制作者上传脚本到我们的系统可能是一个非常糟糕的主意。他们可能会提供他们的检查脚本，但这些脚本在部署到系统中之前必须从安全角度进行审查。如果这样做得当，那么脚本就是 Java 生态系统的一个极其强大的扩展，为我们的程序提供了极大的灵活性。

# 摘要

在本章中，我们开发了企业应用程序的排序系统。随着代码的开发，我们遇到了许多新事物。你学习了关于注解以及它们如何通过反射来处理的知识。尽管它们之间没有很强的关联，但你学习了如何使用 lambda 表达式和流来表示比传统循环更简单的编程结构。在章节的最后部分，我们通过从 Java 调用 JavaScript 函数以及从 JavaScript 调用 Java 方法，使用脚本扩展了应用程序。

事实上，凭借所有这些知识，我们成熟到了企业编程所需的 Java 水平。书中涵盖的其他主题则是为高手准备的。但你想成为其中的一员，不是吗？这就是我写下其余章节的原因。继续阅读吧！
