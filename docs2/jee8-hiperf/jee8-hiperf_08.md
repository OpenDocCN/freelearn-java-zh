# 日志记录器和性能——一种权衡

日志可能是应用程序最重要的部分之一，无论它使用什么技术。为什么是这样？因为没有日志，你不知道应用程序在做什么，也不知道为什么应用程序以某种特定方式表现。

当然，我们在第三章，“监控您的应用程序”，中看到了如何对应用程序进行配置以获取有关 JVM 和应用程序的一些监控信息，但这非常技术性，主要是性能或跟踪导向的。这很重要，但通常不足以帮助操作和支持团队，他们通常更喜欢从更高层次查看应用程序跟踪。这就是日志记录介入的地方。然而，正确使用和配置它是很重要的，这样你就不影响应用程序的性能。

你可能认为 Java EE 和日志记录没有直接关系，但实际上，确保你理解在 EE 环境中日志记录正在做什么可能更为重要。从 EE 容器中获得的主要东西是服务。服务是代码，因此具有与你的代码相同的约束。你可以使用任何 Java EE 库，大多数服务，如果不是所有服务，都会使用一些日志记录器来让你了解发生了什么，而无需查看代码。甚至有些供应商的编码实践要求你在每次方法开始和结束时都进行日志记录。简而言之，日志记录器和日志语句无处不在，无论你是否在自己的代码库中编写了它们，因为它们存在于库代码库中。

因此，在本章中，我们将涵盖：

+   何时使用日志记录

+   日志语句所暗示的工作

+   一些常见且有用的与性能相关的日志记录模式

+   一些知名的日志记录库以及如何在 EE 环境中配置它们

# 我记录，你记录，他们记录

我们何时使用日志记录？这个非常技术性的问题应该这样重新表述：*我们何时提供有关应用程序正在做什么的信息？*这正是日志记录所用的；它让用户知道应用程序在某个时刻做了什么。

这是我们作为开发者经常忘记的事情，因为我们专注于我们正在实现的功能。但如果应用程序打算投入生产，确保操作团队能够非常高效地与之合作是至关重要的。不要忘记，如果开发需要六个月，那么生产可能持续几年，而事故的成本远远高于生产启动前的微小延迟。因此，投资于一个能够传达足够信息的系统通常是值得的。

然而，记录日志并不是一个可以轻视的任务。所有的困难都关于：

+   设计对代码知识贫乏的人——或者没有——有意义的消息

+   确保无论发生什么情况，消息都被记录下来

+   记录默认情况下在代码中可能*不可见*的错误情况

让我们花一点时间来处理这个最后的情况，因为它在 EE 应用中越来越常见，尤其是在微服务环境中——一个 REST 端点中的 HTTP 客户端：

```java
@Path("quote")
@ApplicationScoped
public class QuoteEndpoint {
    @Inject
    private Client client;

    @GET
    @Path("{id}")
    public Quote getQuote(@PathParam("id") final String id) {
        return client.target("http://remote.provider.com")
                .path("quote/{id}")
                .resolveTemplate("id", id)
                .request(APPLICATION_JSON_TYPE)
                .get(Quote.class);
    }
}
```

这是在微服务基础设施中非常常见的模式，一个服务调用另一个服务。这个特定的实现是一个扁平代理（没有额外的逻辑），但它可以用来隐藏一些机器到机器的凭证或安全机制，例如。在这里需要确定的是，一个 JAX-RS 客户端在 JAX-RS 端点（`getQuote`）中被调用。现在，当你考虑错误处理时，这样的代码可能会发生什么？

如果客户端调用失败是因为服务器返回错误（让我们假设 HTTP 404 是因为 ID 无效），那么客户端`get()`调用将抛出`javax.ws.rs.NotFoundException`。由于客户端调用周围没有异常处理，你的端点将抛出相同的异常，这意味着服务器端的 JAX-RS 将是一个 HTTP 404，因此，你的端点将抛出相同的异常。

这可能正是你想要的——例如，在代理的情况下——但在实现方面并不很好，因为当你从客户端（最终客户端，而不是端点客户端）收到 HTTP 404 响应时，你如何知道是端点还是远程服务出了问题？

通过稍微改变端点的实现来减轻这种副作用的方法，如下面的代码片段所示：

```java
@GET
@Path("{id}")
public Quote getQuote(@PathParam("id") final String id) {
    try {
        return client.target("http://remote.provider.com")
                .path("quote/{id}")
                .resolveTemplate("id", id)
                .request(APPLICATION_JSON_TYPE)
                .get(Quote.class);
    } catch (final ClientErrorException ce) {
        logger.severe(ce.getMessage());
        throw ce;
    }
}
```

这不是一个完美的实现，但至少现在，在服务器日志中，你将能够识别出调用失败是由于远程服务上的错误。这已经比沉默好多了。在现实生活中，你可以通知最终客户端错误不是你的责任，或者使用另一个状态码或添加一个头信息，允许客户端根据你实现的服务类型来识别它。然而，不变的是，至少记录错误允许你的应用程序提供足够的信息，以便你调查问题的来源。然后，你可以在其基础上进行的所有增强（日志格式、MDC 等）主要是为了让信息更容易找到和快速分析。

记录日志是一种简单的方式——可能也是最简单的方式——从 JVM 向其外部进行通信。这也可能是为什么它在任何应用的各个层都得到了广泛使用。你可以确信，大多数（如果不是所有）库和容器都依赖于某个日志记录器。通常，这也是你与容器的第一次接触。当你以*空状态*启动它时，你首先看到的是以下输出：

```java
[2017-10-26T17:48:34.737+0200] [glassfish 5.0] [INFO] [NCLS-LOGGING-00009] [javax.enterprise.logging] [tid: _ThreadID=16 _ThreadName=RunLevelControllerThread-1509032914633] [timeMillis: 1509032914737] [levelValue: 800] [[
  Running GlassFish Version: GlassFish Server Open Source Edition 5.0 (build 23)]]

[2017-10-26T17:48:34.739+0200] [glassfish 5.0] [INFO] [NCLS-LOGGING-00010] [javax.enterprise.logging] [tid: _ThreadID=16 _ThreadName=RunLevelControllerThread-1509032914633] [timeMillis: 1509032914739] [levelValue: 800] [[
  Server log file is using Formatter class: com.sun.enterprise.server.logging.ODLLogFormatter]]

[2017-10-26T17:48:35.318+0200] [glassfish 5.0] [INFO] [NCLS-SECURITY-01115] [javax.enterprise.system.core.security] [tid: _ThreadID=15 _ThreadName=RunLevelControllerThread-1509032914631] [timeMillis: 1509032915318] [levelValue: 800] [[
  Realm [admin-realm] of classtype [com.sun.enterprise.security.auth.realm.file.FileRealm] successfully created.]]
```

这里真正重要的是不是输出的内容，而是你能够从日志配置中控制它。Java EE 容器在实现上并不统一，但大多数都依赖于**Java Util Logging**（**JUL**），这是 Java 标准日志解决方案。我们稍后会回到日志实现。但为了继续给你一个为什么不是直接通过控制台输出或文件管理来做的概念，我们将打开 GlassFish 配置。

GlassFish，依赖于 JUL，使用`logging.properties`配置文件。如果你使用默认的 GlassFish 域，你将在`glassfish/domains/domain1/config/logging.properties`文件中找到它。如果你打开这个文件，你会看到以下这些行：

```java
handlers=java.util.logging.ConsoleHandler
handlerServices=com.sun.enterprise.server.logging.GFFileHandler
java.util.logging.ConsoleHandler.formatter=com.sun.enterprise.server.logging.UniformLogFormatter
com.sun.enterprise.server.logging.GFFileHandler.formatter=com.sun.enterprise.server.logging.ODLLogFormatter
com.sun.enterprise.server.logging.GFFileHandler.file=${com.sun.aas.instanceRoot}/logs/server.log
com.sun.enterprise.server.logging.GFFileHandler.rotationTimelimitInMinutes=0
com.sun.enterprise.server.logging.GFFileHandler.flushFrequency=1
java.util.logging.FileHandler.limit=50000
com.sun.enterprise.server.logging.GFFileHandler.logtoConsole=false
com.sun.enterprise.server.logging.GFFileHandler.rotationLimitInBytes=2000000
com.sun.enterprise.server.logging.GFFileHandler.excludeFields=
com.sun.enterprise.server.logging.GFFileHandler.multiLineMode=true
com.sun.enterprise.server.logging.SyslogHandler.useSystemLogging=false
java.util.logging.FileHandler.count=1
com.sun.enterprise.server.logging.GFFileHandler.retainErrorsStasticsForHours=0
log4j.logger.org.hibernate.validator.util.Version=warn
com.sun.enterprise.server.logging.GFFileHandler.maxHistoryFiles=0
com.sun.enterprise.server.logging.GFFileHandler.rotationOnDateChange=false
java.util.logging.FileHandler.pattern=%h/java%u.log
java.util.logging.FileHandler.formatter=java.util.logging.XMLFormatter

#All log level details
javax.org.glassfish.persistence.level=INFO
javax.mail.level=INFO
org.eclipse.persistence.session.level=INFO
```

我们不会在这里进入 JUL 的配置方式，但我们可以从这个片段中识别出，日志抽象允许我们：

+   配置日志（消息）的去向。我们可以看到`GFFileHandler`指向`server.log`文件，例如，但`ConsoleHandler`也被设置了，这与我们在控制台中看到日志的事实是一致的。

+   配置日志级别，我们稍后会详细说明这一点；在非常高的层面上，它允许你选择你想要保留或不要的日志。

如果实现没有使用日志抽象，你就没有选择输出的（处理程序）和级别选择将是针对每个案例的（不是标准化的），这将使操作团队的工作变得更加困难。

# 日志框架和概念

有很多日志框架，这可能是一个集成者面临的挑战，因为当你集成的库越多，你就越需要确保日志记录器是一致的，并且可能需要将它们输出到同一个地方。然而，它们都共享相同的基本概念，这些概念对于了解如何正确使用日志记录器以及如果不注意它们的用法，它们如何以不好的方式影响应用程序性能是非常重要的。

这些概念在不同的框架中可能有不同的名称，但为了识别它们，我们将在这本书中使用 JUL 的名称：

+   日志记录器

+   日志记录器工厂

+   LogRecord

+   处理程序

+   过滤器

+   格式化器

+   级别

# 日志记录器

日志记录器是日志框架的入口点。这是你用来*写入*消息的实例。API 通常有一组辅助方法，但必需的 API 元素包括：

+   允许将级别与消息一起传递。

+   允许传递一个预先计算的消息，该消息是计算所需消息的（它可以是一个带有一些变量的模式或 Java 8 的`Supplier<>`）。在后一种情况下，目标是如果不需要就不评估/插值消息，如果消息是*隐藏的*，就避免支付这种计算的代价。

+   允许将`Exception`（主要针对错误情况）与消息关联。

日志记录器使用的最常见例子可能是：

```java
logger.info("Something happent {0}", whatHappent);
```

这个日志调用与以下行等价，但避免了如果不需要就进行连接：

```java
logger.info("Something happent" + whatHappent);
```

然后，它还会触发将消息（`String`）发送到最终输出（控制台、文件等）。这可能看起来并不重要，但想想在第二章中看到的多个和复杂的层中，你可以在单个请求中调用多少个日志记录器。避免所有这些小操作可能很重要，尤其是在一般情况下，你不仅仅是一个简单的连接，而是在复杂对象上有多个连接。

# 日志工厂

日志工厂通常是一个实用方法（静态），为你提供一个日志记录器实例。

大多数情况下，它看起来像这样：

```java
final Logger logger = LoggerFactory.getLogger(...);
// or
final Logger logger = Logger.getLogger(...);
```

根据日志框架，日志工厂要么是一个特定的类，要么是`Logger`类本身。但在所有情况下，它都为你提供了一个`Logger`实例。这个工厂方法的参数可能会变化，但通常是一个`String`（许多库允许`Class`快捷方式），可以用来配置日志级别，就像我们在前面的 JUL 配置文件中看到的那样。

为什么日志框架需要一个工厂，为什么不允许你使用普通的`new`来实例化日志记录器？因为日志记录器实例的解析方式可能取决于环境。不要忘记，大多数日志消费者（使用日志的代码）可以部署在许多环境中，例如：

+   一个具有平坦类路径的独立应用程序

+   一个具有分层类加载器（树）的 JavaEE 容器

+   一个具有图类加载的 OSGI 容器

在所有日志框架中，总是可以配置配置解析的方式，从而确定日志记录器的实例化方式。特别地，一旦你有一个容器，你将想要处理全局容器配置和*每个应用程序*配置，以便使一个应用程序配置比默认配置更具体。为此，容器（或当实现足够通用时为日志框架）将实现自定义配置解析，并让日志框架使用此配置实例化一个日志记录器。

通常，在一个 EE 容器中，你将得到每个应用程序不同的日志配置。如果应用程序没有提供任何配置，则将使用容器配置。以 Apache Tomcat 实现为例，它默认会读取`conf/logging.properties`，而对于每个应用程序，如果该文件存在，它将尝试读取`WEB-INF/logging.properties`。

# LogRecord

`LogRecord`是日志消息结构。它封装了比你传递的`String`消息更多的数据，允许你获取我们经常在日志消息中看到的信息，例如：

+   日志级别

+   可选的，一个日志序列号

+   源类名

+   源方法名

+   消息确实

+   线程（通常通过其标识符而不是其名称来识别，但最后一个并不总是唯一的）

+   日志调用日期（通常自 1970 年以来以毫秒为单位）

+   可选地，与日志调用关联的异常

+   日志记录器名称

+   可选地，如果日志记录器支持国际化，则可以是一个资源包

+   可选地，一个调用上下文，例如基于自定义值（MDC）或当前 HTTP 请求的一组上下文数据

在这个列表中，我们找到我们看到的信息（如消息），但还包括与日志调用关联的所有元数据，例如调用者（类和方法）、调用上下文（例如日期和线程），等等。

因此，正如我们将看到的，是这条`*记录*`被传递到日志链中。

# 处理程序

有时称为`Appender`，处理程序是输出实现。它们是接收上一部分的`LogRecord`并对其进行某种操作的那些。

最常见的实现包括：

+   `FileHandler`：将消息输出到文件中。

+   一个`MailHandler`：通过邮件发送消息。这是一个特定的处理程序，不应用于大量消息，但它可以与一个`*特定*`的日志记录器一起使用，该日志记录器专门用于在特定情况下发送一些消息。

+   `ConsoleHandler`用于将消息输出到控制台。

+   还有更多，例如`JMSHandler`、`ElasticsearchHandler`、`HTTPHandler`等。

在任何情况下，处理程序都将数据发送到`*backend*`，它可以是一切，并且日志框架始终确保您可以在需要扩展默认处理程序时插入自己的实现。

# 过滤器

一个`Filter`是一个简单的类，允许您让`LogRecord`通过或不通过。在 Java 8 生态系统中，它可以被视为一个`Predicate<LogRecord>`；这个类自从 Java 1.4 版本以来就在 Java 中，远在`Predicate`被创建之前。

它通常与特定的`Handler`绑定。

# 格式化器

一个`Formatter`是一个接受`LogRecord`并将其转换为`String`的类。它负责准备要发送到后端的内容。想法是将`*写作*`和`*格式化*`关注点分开，允许您重用一部分而无需创建一个新的`Handler`。

同样，它通常与特定的`Handler`绑定。

# 级别

`级别`是一个简单的概念。它是日志记录的元数据，也是我们刚刚查看的大多数日志组件。主要思想是能够将日志记录级别与记录通过的组件级别进行比较，如果不兼容则跳过消息。常见的（排序）日志级别包括：

+   `OFF`：不直接用于日志记录，但通常仅由其他组件使用，它禁用任何日志消息。

+   `SEVERE`（或`ERROR`）：最高的日志级别。它旨在在发生严重问题时使用。如果组件级别不是`OFF`，则记录条目将被记录。

+   `WARNING`：通常在发生错误时使用（但不会阻止应用程序工作）；如果组件级别不是`OFF`或`SEVERE`，则记录条目将被记录。

+   `INFO`：许多应用程序的默认日志级别，用于通知我们发生了正常但有趣的事情。

+   `CONFIG`：不是最常用的级别，它旨在用于与配置相关的消息。在实践中，应用程序和库通常使用`INFO`或`FINE`。

+   `FINE`,`FINER`,`FINEST`,`DEBUG`：这些级别旨在提供关于应用程序的低粒度信息。消息计算可能很昂贵，通常不打算在生产环境中启用。然而，当调查问题时，它可能是一个非常有用的信息。

+   `ALL`：不用于日志记录本身，但仅用于组件级别，它允许记录任何消息。

日志级别按顺序排序（与整数相关联），如果所有组件级别都低于日志记录级别，则日志记录级别是*活动*的。例如，如果组件级别是`INFO`或`FINE`，则将记录`WARNING`消息。

# 日志调用

在查看如何使用一些常见模式将记录器正确集成到您的应用程序之前，让我们看看记录器调用将触发什么。

一个简单的记录器调用，如`logger.info(message)`，可以检查以表示为以下步骤的等价：

+   检查记录器级别是否处于活动状态；如果不是，则退出

+   创建日志记录（设置消息和级别，初始化日志源、类、方法等）

+   检查消息是否被`Filter`过滤；如果过滤器过滤了它，则退出

+   对于所有处理器，将日志记录发布到处理器：

    +   检查处理器级别与日志记录级别的匹配情况，如果不兼容则退出（注意：此检查通常执行多次；因为它只是一个整数比较，所以很快）

    +   格式化日志记录（将其转换为`String`）

    +   将格式化后的消息写入实际的后端（文件、控制台等）

这种对单个记录器调用的逐层深入分析显示了关于记录器的许多有趣之处。首先，使用记录器并在所有日志组件中设置级别，允许日志框架在级别不兼容的情况下绕过大量逻辑。这在记录器级别是正确的，可能在过滤器级别，最终在处理器级别。然后，我们可以确定有两个层次，逻辑依赖于配置，处理时间将是这些元素复杂性的函数，包括过滤和格式化。最后，实际工作——通常链中最慢的部分——是与后端交互。具体来说，与硬件（你的硬盘）交互时，在文件中写入一行比链中的其他部分要慢。

# 过滤器

在 JUL 中，没有默认的过滤器实现。但您可以在其他框架或 JUL 扩展中找到一些常见的过滤器：

+   基于时间的过滤器：

    +   如果日志消息不在时间范围内，则跳过它

    +   如果日志消息的年龄超过一些配置的持续时间，则跳过它（这取决于链中过滤器之前的工作以及机器是否有热点峰值）

+   基于**映射诊断上下文（MDC**）的过滤器：通常，如果 MDC 值匹配（例如，如果 MDC['tenant']是*hidden_customer*），则跳过该消息

+   节流：如果你知道你使用的手柄无法支持每秒超过 1,000 条消息，或者如果实际的后端，例如数据库，无法支持更多，那么你可以使用过滤器来强制执行这个限制

+   基于正则表达式的：如果消息不匹配（或匹配）正则表达式，则跳过

这些例子只是你可能会遇到的潜在过滤器的一个简短列表，但它说明了复杂性可能重要或不太重要，因此过滤器层的执行时间可能快或慢。

# 格式化器

对于过滤器，有几个格式化器，并且由于它实际上关于如何将日志记录——日志 API——转换为后端表示（字符串），所以它可能更昂贵或更便宜。为了获得一个高级的概念，这里有一些例子：

+   XML：将日志记录转换为 XML 表示形式；它通常使用字符串连接并记录所有记录信息（记录器、线程、消息、类等）。

+   简单：只是日志级别和消息。

+   模式：基于配置的模式，输出被计算。通常，日志框架允许你在该模式中包含线程标识符或名称、消息、日志级别、类、方法、如果有异常则包括异常，等等。

默认的 JUL 模式是`%1$tb %1$td, %1$tY %1$tl:%1$tM:%1$tS %1$Tp %2$s%n%4$s: %5$s%6$s%n`。这个模式导致这种类型的输出：

```java
oct. 27, 2017 6:58:16 PM com.github.rmannibucau.quote.manager.Tmp main
INFOS: book message
```

我们在这里不会详细说明语法，但它重用了`SimpleFormatter`背后的 Java `java.util.Formatter`语法，该语法用于输出。此实现将以下参数传递给格式化器：

+   日志事件的日期

+   日志事件的来源（`类方法`）

+   记录器名称

+   级别

+   消息

+   异常

关于这种最后一种格式化器的有趣之处在于，它允许你自定义输出并根据自己的需求更改其格式。例如，你可以在两行上使用默认格式，但你也可以设置格式为`%1$tb %1$td, %1$tY %1$tl:%1$tM:%1$tS %1$Tp %2$s %4$s: %5$s%6$s%n`，然后输出将会是：

```java
oct. 27, 2017 7:02:42 PM com.github.rmannibucau.quote.manager.Tmp main INFOS: book message
```

最大的优势是真正根据你的需求自定义输出，并可能匹配像 Splunk 或 Logstash 这样的日志转发器。

要在 JUL 中激活此模式，你需要设置系统属性`"-Djava.util.logging.SimpleFormatter.format=<the pattern>"`。

# 使用处理器

在处理性能问题时，定义你希望在日志方面做出的权衡是很重要的。一个有趣的指标可以是将没有任何活动日志语句的应用程序与你想保留的（用于生产）进行比较。如果你发现启用日志后性能急剧下降，那么你可能有一个配置问题，要么是在日志层中，要么更常见的是在与后端的使用（如过度使用远程数据库）有关。

处理器，作为退出应用程序的部分，是需要最多关注的。这并不意味着其他层不重要，但它们通常检查起来更快，因为它们往往会导致恒定的评估时间。

处理器的实现有几种，但在公司中拥有特定实现并不罕见，因为你可能想要针对特定的后端或进行定制集成。在这些情况下，你必须确保它不会引入一些瓶颈或性能问题。为了说明这一点，我们将使用一个例子，其中你想要以 JSON 格式将日志记录发送到 HTTP 服务器。对于每条消息，如果你发送一个请求，那么你可以并行发送多个请求作为线程，并且你将为每个日志调用支付 HTTP 延迟。当你想到一个方法可以有多个日志调用，并且一个日志可以有多个处理器（你可以在控制台、文件和服务器中记录相同的消息），那么你很快就会理解这种按消息同步 *首次* 实现不会长期扩展。

正是因为所有后端集成，它们都暗示着远程操作，所以都在使用替代实现，并且通常支持将消息批量发送（一次发送多个消息）。然后，处理器的消息接收只是触发在 *栈* 中的添加，稍后，另一个条件将触发实际请求（在我们之前的例子中是 HTTP 请求）。在性能方面，我们将高延迟实现转换为低延迟操作，因为操作的速度就像将对象添加到队列中一样快。

# 日志组件和 Java EE

使用 Java EE 来实现日志组件可能会有诱惑力。虽然并非不可能，但在这样做之前，有一些要点需要考虑：

+   JUL 并不总是支持使用容器或应用程序类加载器加载类，因此你可能需要一个上下文加载容器或应用程序类的门面实现。换句话说，你并不总是能够程序化地依赖于 CDI，但你可能需要一些反射，而这会带来你想要最小化的成本。所以，如果你能的话，确保保留你的 CDI 查找结果。

+   在前几章中，我们探讨了 Java EE 层。确保你不会依赖于对日志实现来说过于沉重的某些东西，以避免受到所有这些工作的影响，并避免通过你的记录器隐藏你有一个*应用程序下的应用程序*的事实。

+   日志上下文不受控制。通常，你不知道何时使用记录器。因此，如果你使用 CDI 实现一个组件，确保你使用所有上下文都有的功能。具体来说，如果你没有使用`RequestContextController`来激活作用域，就不要使用`@RequestScoped`。此外，确保你只在一个用于 EE 上下文的记录器上配置了*EE*组件。

做一个日志-EE 桥接不是不可能的，但对于日志记录，我们通常希望非常高效且尽可能原始。更确切地说，它更像是一个潜在的回退方案，如果你不能修改应用程序的话，而不是默认的相反方案。从现实的角度来看，最好发送你观察到的 EE 事件，并从观察者那里调用记录器，而不是相反。

# 日志模式

有一些重要的日志模式你可以利用来尝试最小化隐含的日志开销，而不会从功能角度获得任何好处。让我们来看看最常见的几个。

# 测试你的级别

关于日志消息最重要的东西是其级别。这是允许你高效忽略消息——以及它们的格式化/模板化——的信息，如果它们最终会被忽略的话。

例如，考虑这个依赖于不同级别日志记录器的这个方法：

```java
public void save(long id, Quote quote) {
    logger.finest("Value: " + quote);
    String hexValue = converter.toHexString(id);
    doSave(id, quote);
    logger.info("Saved: " + hexValue);
    logger.finest("Id: " + hexValue + ", quote=" + quote);
}
```

这种方法混合了*调试*和*信息*日志消息。尽管如此，在大多数情况下，*调试*消息不太可能被激活，但如果你将*信息*级别作为可选消息使用，那么对*警告*级别进行相同的推理。因此，在大多数情况下记录*调试*消息是没有意义的。同样，计算这些消息的连接也是无用的，因为它们不会被记录。

为了避免这些计算——别忘了在某些情况下`toString()`可能很复杂，或者至少计算起来`long`——一个常见的模式是自己在应用程序中测试日志级别，而不是等待日志框架去完成：

```java
public void save(long id, Quote quote) {
    if (logger.isLoggable(Level.FINEST)) {
        logger.finest("Value: " + quote);
    }

    String hexValue = converter.toHexString(id);
    doSave(id, quote);
    logger.info("Saved: " + hexValue);

    if (logger.isLoggable(Level.FINEST)) {
        logger.finest("Id: " + hexValue + ", quote=" + quote);
    }
}
```

简单地将很少记录的消息包裹在`isLoggable()`条件块中，将对日志记录器级别进行快速测试，并绕过消息计算，以及大多数情况下绕过所有日志链，确保性能不会因调试语句而受到太大影响。

自 Java 8 以来，有一个有趣的替代模式：使用`Supplier<>`来创建消息。它提供了一种计算消息而不是消息本身的方法。这样，由于 lambda 与相关签名兼容，代码更加紧凑。但无论如何，字符串评估的成本并没有支付：

```java
logger.finest(() -> "Value: " + quote);
```

在这里，我们传递了一个 lambda 表达式，仅在日志语句通过了第一个测试（即日志记录器的日志级别）时才计算实际的消息。这确实非常接近之前的模式，但仍然比通常的`isLoggable()`实现中的整数测试要昂贵一些。然而，开销并不大，代码也更简洁，但通常足够高效。

如果你多次使用相同的日志级别，在方法开始时一次性将日志级别检查因式分解可能是有价值的，而不是在相同的方法中多次调用它。你使用日志抽象越多，因此经过的层级越多，这一点就越正确。由于这是一个非常简单的优化——你的 IDE 甚至可以建议你这样做——你不应该犹豫去实施它。尽管如此，不要在类级别（例如，将可记录的测试存储在`@PostConstruct`中）这样做，因为大多数日志实现都支持动态级别；你可能会破坏该功能）。

# 在你的消息中使用模板

在上一节中，我们探讨了如何绕过消息的日志记录。在 JUL 中，这通常是通过调用具有日志级别的日志方法来完成的，但有一个更通用的日志方法称为`log`，它可以接受级别、消息（如果你对消息进行国际化，则是一个资源包中的键）以及一个对象数组参数。此类方法存在于所有框架中，其中大多数也会提供一些特定的签名，具有一个、两个或更多参数，以使其使用更加流畅。

在任何情况下，想法是使用消息作为模式，并使用参数来赋值消息中的某些变量。这正是此日志模式使用的功能：

```java
logger.log(Level.INFO, "id={0}, quote={1}", new Object[]{ id, quote });
```

此日志语句将用`Object[]`数组中的第 i 个值替换`{i}`模板。

使用此模式很有趣，因为它避免了在不需要时计算实际的字符串值。从代码影响的角度来看，这似乎比之前的`isLoggable()`检查要好，对吧？实际上，这取决于日志实现。JUL 不支持它。但是，对于支持参数而不使用数组的框架，它们可以进行一些优化，从而使这个假设成立。然而，对于 JUL 或所有需要创建数组以存储足够参数的情况，这是不正确的。你必须创建数组的事实具有影响，因此如果你不需要它，最好是跳过它，这意味着回退到之前的模式或基于`Supplier<>`的 API。

# 异步或非异步？

由于现代对扩展性的要求，需要增强日志记录器以支持更高的消息速率，但仍然需要减少对应用程序性能本身的影响。

减少日志记录延迟的第一步是使处理程序异步。在 JUL 中这还不是标准功能，但你可以找到一些提供该功能的库——一些容器，如 Apache TomEE，甚至提供开箱即用的功能。这个想法与我们关于处理程序的章节中描述的完全一样，计算日志记录的最小上下文，然后在调用者线程中的 *队列* 中推送记录，然后在另一个线程（或线程，取决于后端）中实际 *存储/发布* 消息。

这种模式已经解决了大多数关于性能的日志记录器影响，但一些日志框架（如 Log4j2）更进一步，使日志记录器本身异步。由于过滤（有时还有格式化）现在是完全异步完成的，因此调用者的持续时间大大缩短，性能影响也大大降低（仍然，考虑到你有足够的 CPU 来处理这个额外的工作，因为你并行执行更多的代码）。

如果你添加一些基于环形缓冲区模式的现代实现来处理异步，就像 Log4j2 使用 disruptor ([`lmax-exchange.github.io/disruptor/`](https://lmax-exchange.github.io/disruptor/)) 库所做的那样，那么你将有一个非常可扩展的解决方案。线程越多，这种实现的冲击力就越显著，甚至与异步处理程序（log4j2 的追加器）相比。

# 日志实现 – 选择哪一个

日志是你在计算机科学中可以遇到的最早的话题之一，但它也是被多次解决的问题之一。理解你将找到许多关于日志的框架。让我们快速看一下它们，看看它们有时是如何相关的。

# 日志门面 – 好坏参半的想法

日志门面是像 SLF4J ([`www.slf4j.org/`](https://www.slf4j.org/))、commons-logging、jboss-logging 或更近期的 log4j2 API ([`logging.apache.org/log4j/2.x/`](https://logging.apache.org/log4j/2.x/)) 这样的框架。它们的目的是提供一个统一的 API，可以与任何类型的日志实现一起使用。你必须真正将其视为一个 API（就像 Java EE 是一个 API 一样），而日志框架则是实现（就像 GlassFish、WildFly 或 TomEE 是 Java EE 实现）。

这些门面需要一种方式来找到它们必须使用的实现。你可能遇到几种策略，如下所示：

+   例如，SLF4J 将查找所有实现在其分发中提供的特定类（在 SLF4J 中称为 *绑定*），一旦实例化，它将给 SLF4J API 提供与最终实现（JUL、Log4J、Logback 等）的链接。这里的问题是，你无法在同一个类加载器中拥有多个实现，你也不能只配置你想要的那个。

+   Commons-logging 将读取配置文件以确定选择哪个实现。

+   基于全局系统属性配置的硬编码默认值，允许选择要使用的实现。

使用日志外观通常是一个好主意，因为它允许你的代码，或者你使用的库的代码，与日志实现解耦，将实现的选择委托给应用程序打包者或部署者。它允许你在所有情况下运行它，而无需你在开发期间关心它。

即使忽略存在多个此类 API 实现的事实，这已经使事情变得复杂，但根据你所使用的实现，它们的用法并不那么优雅。一些实现将需要填写一些代价高昂的参数。

最好的例子是计算源类和方法。在几个实现中，这将通过创建一个 `Exception` 来获取其关联的堆栈跟踪，并在丢弃已知的日志框架调用者之后，推断出业务调用者。在 Java EE 环境中，由于它使用的堆栈以提供简单的编程模型，异常堆栈可能非常大且 *慢*（分配数组）。这意味着每个日志消息都会稍微减慢一点，以实现这个外观 API 和实际使用的日志实现之间的桥梁。

因此，检查你使用的此类外观实现可能是有价值的。对于最常见的一个，SLF4J，有两个非常知名的实现与 API 紧密结合：

+   Logback：API 的本地实现

+   Log4j2：具有 SLF4J 直接实现（绑定）

# 日志框架

如前所述，你可能会遇到几个日志框架。最著名的是，你可能已经听说过的，包括：

+   Log4j1：历史上事实上的标准，逐渐被 log4j2 取代。

+   Log4j2：可能是今天最先进的实现之一。支持异步日志记录和环形缓冲区集成。

+   Logback：SLF4J 的本地实现。在 log4j2 完成之前，这可能是最佳选择。

+   Java Util Logging：JVM 标准日志 API。虽然不是最先进的 API，但它可以工作，并且不需要任何依赖项，尽管如此，你可能需要一些自定义集成（处理程序）以用于生产。检查你的服务器，它可能已经提供了一些解决方案。

这些都共享我们刚刚讨论过的概念，但可能有一些小的差异。为了让你更快地使用它们，我们将快速浏览这些实现，以定义框架使用的语义，并展示如何配置每个实现。当你进行基准测试时，了解如何配置日志并确保它不会因为配置不当而减慢性能是非常重要的。

# Log4j

Logj4 (1.x) 使用 `org.apache.log4j` 基础包。以下是针对 log4j1 语义调整的我们讨论过的日志概念：

| **概念** | **名称** |
| --- | --- |
| 记录器 | 记录器 |
| 日志工厂 | 记录器 |
| 处理程序 | 追加器 |
| 过滤器 | 过滤器 |
| 格式化器 | 布局 |
| 级别 | 级别 |

大多数概念与 JUL 相同，但有一些不同的名称。

在使用方面，它与 JUL 相同，除了它使用另一个包：

```java
final Logger logger = Logger.getLogger("name.of.the.logger");
logger.info("message");
```

与之前的不同之处在于配置。它使用类路径中的`log4j.properties`或`log4j.xml`（默认情况下），其外观如下：

```java
log4j.rootLogger=DEBUG, stdout

log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%-4r [%t] %-5p %c %x - %m%n

log4j.com.packt.quote = WARN
```

使用此示例配置，根日志记录器（默认）级别为`DEBUG`，它将使用`stdout`追加器。追加器使用`ConsoleAppender`；它将在`System.out`上记录消息，并使用自定义模式（`ConversionPattern`）的格式布局。`com.packt.quote`包的日志级别设置为`WARN`。因此，使用此名称或此包子名称的日志记录器将仅记录`WARN`和`ERROR`消息。

# Log4j2

Log4j2 明显受到了 Log4j1 的启发，但被完全重写。它在行为和性能方面仍然有一些差异。以下是 log4j2 的概念映射：

| **概念** | **名称** |
| --- | --- |
| 日志记录器 | 日志记录器 |
| 日志工厂 | 日志管理器 |
| 处理器 | 追加器 |
| 过滤器 | 过滤器 |
| 格式化器 | 布局 |
| 级别 | 级别 |

配置有一些回退，但默认文件在类路径中查找，并称为`log4j2.xml`。它使用与 Log4j1 的 XML 版本不同的语法，基于 Log4j2 的新插件系统，以获得更简洁的语法（更少的技术性）：

```java
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
  <Appenders>
    <Console name="stdout" target="SYSTEM_OUT">
      <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} -
      %msg%n"/>
    </Console>
  </Appenders>
  <Loggers>
    <Root level="debug">
      <AppenderRef ref="stdout"/>
    </Root>
  </Loggers>
</Configuration>
```

这与上一节中的配置类似，但它使用这种新的语法，该语法依赖于插件名称（例如，在 log4j2 中，`Console`是控制台追加器的名称）。尽管如此，我们仍然有相同的结构，其中日志记录器在日志记录器块中定义，具有特定的根日志记录器，并且追加器有自己的块，通过引用/标识符（`ref`）与日志记录器链接。

Log4j2 具有其他一些不错的功能，如配置的热重载、JMX 扩展等，这些都值得一看。它可以帮助你在基准测试期间更改日志配置而无需重新启动应用程序。

# Logback

Logback 是 SLF4J 的原生实现，是一个高级日志实现。以下是它与我们所讨论的概念的映射：

| **概念** | **名称** |
| --- | --- |
| 日志记录器 | 日志记录器（来自 SLF4J） |
| 日志工厂 | 日志工厂（来自 SLF4J） |
| 处理器 | 追加器 |
| 过滤器 | 过滤器 |
| 格式化器 | 编码器/布局 |
| 级别 | 级别 |

注意，logback 还有将消息链接到`byte[]`的`Encoder`概念。

默认情况下，配置依赖于类路径中的`logback.xml`文件，其外观如下：

```java
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <root level="debug">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

这是一种接近之前配置（log4j1，log4j2）的配置，并识别相同类型的配置，除了`encoder`层包裹`pattern`。这主要是因为编码器将传递一个`byte[]`值给追加器，而模式将传递一个`String`给编码器，这使得实现更容易组合，即使很少使用。

# JUL

我们使用 JUL 来命名我们讨论的概念，因此我们不需要为概念创建映射表。然而，了解 JUL 的配置方式很有趣，因为它被广泛应用于许多容器中。

高级的 JUL 使用一个`LogManager`，这是记录器工厂（隐藏在`Logger.getLogger(...)`记录器工厂后面）。

`LogManager`是从传递给`java.util.logging.config.class`系统属性的类实例化的。如果没有设置，将使用默认实现，但请注意，大多数 EE 容器会覆盖它以支持额外的功能，例如每个应用程序的配置，例如，或自定义配置，通常是动态的，并通过一个很好的 UI 进行管理。

配置位于系统属性`java.util.logging.config.file`中，或者回退到`${java.jre.home}/lib/logging.properties`——请注意，Java 9 使用了`conf`文件夹而不是`lib`。此属性文件的语法与我们之前看到的相同：

```java
handlers= java.util.logging.FileHandler, java.util.logging.ConsoleHandler

java.util.logging.FileHandler.pattern = %h/java%u.log
java.util.logging.FileHandler.limit = 50000
java.util.logging.FileHandler.count = 1
java.util.logging.FileHandler.formatter = java.util.logging.XMLFormatter

java.util.logging.ConsoleHandler.level = INFO
java.util.logging.ConsoleHandler.formatter = java.util.logging.SimpleFormatter

.level = INFO
com.packt.level = SEVERE
```

在代码片段的末尾，我们确定了如何为包设置级别（键只是后缀为`.level`，值是级别名称）。同样的逻辑也适用于默认的记录器，它有一个空的名字，因此其级别是通过`.level`键直接设置的。`handlers`键获取要配置的处理程序列表（以逗号分隔）。它通常是处理程序的完全限定名。然后，中间的两个块，从处理程序名称开始，是处理程序配置。它通常使用点分表示法（`<handler class>.<configuration> = <value>`），但这是可选的，因为处理程序可以通过`LogManager`访问所有属性。

Apache Tomcat/TomEE 的`ClassLoaderLogManager`允许你使用一个以数字开头的自定义前缀值来前缀处理程序。它使你能够定义 N 次相同的处理程序，具有不同的配置，这是 JUL 默认不支持的功能，JUL 只能定义一个处理程序一次。

# 选择实现

如果你可以选择你的实现，你应该检查你想要发送日志的地方，并选择已经做了你所需要的工作的实现，或者具有非常接近功能的实现。然后，如果有多个实现符合你的要求，你需要检查哪一个是最快的。

从 log4j2 或 logback 开始通常是不错的选择。然而，在实践中，您很少有这样的选择，您的一部分堆栈是由您的环境（依赖项和容器）强加的。因此，您很可能还需要配置 JUL。在这种情况下，一个好的选择是检查您是否可以重用您的容器骨干，而无需在代码上依赖于您的容器（也就是说，您可以使用 JUL API 并委托给容器，例如 JUL 配置）。然后，您需要评估 JUL 是否满足您在性能方面的运行时需求。JUL 在互联网上收到了很多负面评价，但它对于许多异步将日志记录到文件的应用程序来说已经足够了。在没有评估您具体需求的情况下，不要忽视它。它可以在很多情况下避免配置头痛和依赖。

另一个标准可以是将所有日志重定向到同一日志堆栈的简便性。在这方面最好的之一是 log4j2，它几乎支持所有集成（SLF4J，commons-logging，log4j1，log4j2，等等）。

如果您使用基于文件的处理器/追加器，这可能是最常见的情况，您还应该查看轮换策略。这通常是可配置的，最常见的方法包括：

+   每日：每天，处理器都会为新的日志文件创建一个新的日志文件（例如，mylog.2017-11-14.log，mylog.2017-11-15.log，等等）。

+   在重启时：每次服务器重启时，都会创建一个新的日志文件。请注意，这种策略对于批处理实例或没有长时间运行的实例除外。

+   按大小：如果日志文件大小超过某些磁盘空间，则创建一个新的文件。

注意，所有这些策略通常可以混合或累积。只有 JUL 不支持这一点，但容器通常会尝试填补这一空白。并不是因为您的容器使用 JUL，您就没有这个功能。在拒绝 JUL 作为 API 之前，不要犹豫去查看您的容器日志配置并对其进行调查。

# 摘要

在本章中，我们探讨了为什么日志记录对于获得良好且易于监控的级别很重要。我们还看到，日志语句必须尽可能减少对性能的影响，以免抵消您在应用程序的其他部分可能已经进行的优化和编码。

本章为您提供了一些常见且简单的模式，可以帮助您尽可能依赖日志框架，以确保您保持良好的性能。

最后，我们注意到在您的应用程序中可能需要配置多个实现，但它们都将共享相同的概念，并且可以依赖于单个 API，甚至来自多个 API 的单个实现。

到这本书的这一部分，您已经了解了 Java EE 的功能以及如何编码和监控应用程序。现在，是时候看看您应该如何对待基准测试了。这将是下一章的主题。
