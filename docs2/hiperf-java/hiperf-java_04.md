# 4

# Java 对象池

我们使 Java 应用程序高度性能化的使命包括对 Java **对象池**的考察。本章深入探讨了 Java 中对象池的概念以及如何在您的 Java 应用程序中使用它们来实现高性能。

本章首先解释了对象池及其在 Java 中的实现方法。提供了示例代码，以帮助您理解 Java 编程语言中特定的对象池操作。您还将有机会了解 Java 中对象池的优点和缺点。最后，本章展示了如何使用 Java 对象池进行性能测试。

本章涵盖了以下主要主题：

+   进入对象池

+   优点和缺点

+   性能测试

到本章结束时，您应该对 Java 对象池有坚实的理论理解，以及实际操作经验。这种经验可以帮助确保您从 Java 应用程序中获得高性能。

# 技术要求

要遵循本章中的示例和说明，您需要能够加载、编辑和运行 Java 代码。如果您尚未设置您的开发环境，请参阅*第一章*。

本章的完成代码可以在这里找到：[`github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter04`](https://github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter04)。

# 进入对象池

在我们进入对象池之前，让我们看看什么是对象池。

对象池

对象池是一组可以重复使用的对象。

使用对象池是一种优化方法，可以积极影响应用程序的性能。我们不需要每次需要时都重新创建对象，而是将一组对象池化，并简单地回收它们。为了帮助理解对象池，可以考虑一个现实世界的例子，即物理图书馆。图书馆可以借出书籍（我们的对象），当读者用完书后，将它们归还到集合（我们的池）中。这允许图书馆将书籍重新借给下一个需要的人。考虑一下另一种情况。如果图书馆在每次使用后销毁（垃圾收集）书籍，那么每次需要时都必须创建一个新的。这将不会高效。

## 数据库示例

在 Java 编程中，一个常见的对象池实现是使用数据库连接。数据库连接的典型方法是与数据库建立连接，然后执行所需的操作来更新或查询数据库。使用**打开-查询-关闭**过程。这种方法的问题在于频繁地打开和关闭数据库可能会影响 Java 应用程序的整体性能。这种处理开销是我们应该尽量避免的。

使用对象池的方法，以我们的数据库示例为例，涉及维护一个空闲的预创建数据库请求池。当应用程序请求数据库连接时，会从池中取出一个。下一节将演示如何在 Java 应用程序中创建和使用这些池。

## 在 Java 中实现对象池

根据前一小节中的数据库连接示例实现对象池，涉及一个数据库连接类、一个对象池类以及一个包含`main()`方法的类。让我们逐一查看这些类。

注意

此示例模拟对象池，并不连接到实际的数据库。

首先，我们有我们的`DBConnect`类。这是我们将会池化的类：

```java
public class DBConnect {
  static int dbConnectionCount = 0;
  private int dbConnectID;
  public DBConnect() {
    this.dbConnectID = ++dbConnectionCount;
    System.out.println("Database connection created: DBConnect" + 
    dbConnectID + ".");
  }
  public void dbMethod() {
    // placeholder
  }
  public void dbConnectionClose() {
    System.out.println("Database connection closed: DBConnect" + 
    dbConnectID + ".");
  }
}
```

如前述代码所示，这里有一些功能占位符。

接下来，我们创建一个`DBConnectObjectPool`类来维护一个`DBConnect`对象集合（池）：

```java
import java.util.LinkedList;
import java.util.Queue;
public class DBConnectObjectPool {
  private final Queue<DBConnect> pool;
  private final int maxSize;
  public DBConnectObjectPool(int size) {
    this.maxSize = size;
    this.pool = new LinkedList<>();
  }
  public synchronized DBConnect getConnection() {
    if (pool.isEmpty()) {
      if (DBConnect.dbConnectionCount < maxSize) {
        return new DBConnect();
      }
      throw new RuntimeException("Error: Maximum object pool size 
      reached. There are no DB connections available.");
    }
    return pool.poll();
  }
  public synchronized void releaseConnection(DBConnect dbConnection) {
    if (pool.size() < maxSize) {
      pool.offer(dbConnection);
      System.out.println("Splash: Connection object returned to the 
      pool.");
    } else {
      dbConnection.dbConnectionClose();
    }
  }
}
```

如前述代码所示，我们假设了一个最大连接数。这被认为是一种最佳实践。

最后，我们有一个部分应用示例来展示如何使用我们的对象池：

```java
public class ObjectPoolDemoApp {
  public static void main(String[] args) {
    DBConnectObjectPool objectPool = new DBConnectObjectPool(8);
    for (int i = 0; i < 10; i++) {
      DBConnect conn = objectPool.getConnection();
      conn.dbMethod();
      objectPool.releaseConnection(conn);
    }
  }
}
```

当我们的应用程序请求数据库连接时，会从池中提供一个。在连接不可从池中获取的情况下，会创建一个新的连接。我们确实检查以确保不超过允许的最大连接数。最后，在`DBConnect`对象使用后，将其返回到对象池。

# 对象池的优势和劣势

现在你已经了解了对象池是什么以及如何在 Java 中实现它，我们应该考虑这是否是我们应用程序的正确策略。与大多数应用程序代码优化方法一样，都有其优缺点。本节将探讨这两者。

## 优势

使用对象池有几个潜在的优势。这些优势可以分为性能、资源管理和可扩展性类别。

### 性能优势

实现对象池可以让我们避免对象创建的开销。这种方法在需要高交易量应用和系统响应时间重要的场景中尤其有用。通过对象池，我们可以帮助确保我们的 Java 应用程序能够通过避免过多的对象创建来提高性能。

我们还可以在应用程序使用过程中体验到一致的性能。例如，使用对象池应该导致在最小负载和重负载下应用程序性能的一致性。这种可预测的行为是由于我们应用程序的稳定性。这种稳定性是通过避免频繁的对象创建和对垃圾回收的过度依赖来实现的。

### 资源管理优势

在对象池优势的背景下，资源指的是实时、处理负载和内存。减少对象创建和销毁操作的数量是对象池方法的一个好处。本章前面提到的例子是数据库连接。之所以使用这个例子，是因为数据库连接操作是众所周知的资源消耗者。对象池方法减少了执行这些操作所需的时间，并且资源消耗较少。

另一个资源管理优势是它增加了我们的内存管理架构。当对象创建不受控制时，消耗的内存量是可变的，可能会导致系统错误。

### 可扩展性优势

第三个优势类别是使我们的应用程序更具可扩展性。这在我们有大量同时用户的应用程序中尤其如此。在处理数据库连接时，数据库是共享资源，这也很有益。对象池本质上充当了这些请求的缓冲。

另一个原因是我们使用对象池的应用程序更具可扩展性，那就是我们对资源的控制能力得到了增强。在本章前面提到的数据库连接示例中，我们设置了池中对象的最大数量。

## 劣势

不幸的是，使用对象池的潜在劣势比优势多。这些劣势可以分为代码复杂性和资源管理类别。

### 代码复杂性劣势

就像任何非标准编程方法一样，对象池会增加我们的代码复杂性。我们创建了与对象池相关的类，这些类必须包含管理对象池的算法以及与主程序的接口。尽管这不太可能导致代码膨胀，但它可能会使维护变得困难。

在 Java 应用程序中实现对象池时，每次系统、连接系统或数据发生变化时，都会增加一个测试组件。这可能会消耗时间和资源。

### 资源管理劣势

总是有风险，尤其是在高峰负载时段，资源可能不足。当我们设置对象池的最大大小时，它们可能不足以处理那些高峰负载时段。这也可以被称为**资源饥饿**，因为我们的池中所有对象都已分配，阻止新的请求入队。这些延迟可能导致整体性能下降和用户不满。

与内存分配和释放一起工作可能会出现问题。如果我们没有，例如，管理对象返回池的方式，可能会造成数据丢失。这可能会加剧池中无可用对象的情况。实现错误检查和异常处理变得至关重要。

最后，我们需要保持对象池过大或过小的平衡。如果它太小，可能会导致池化对象的大量排队时间。如果池子太大，应用程序可能会过度消耗内存，从而影响其他可能利用它的应用程序区域。

在考虑了优点和缺点之后，你应该能够确定对象池是否最适合你的应用程序。

# 性能测试

当我们在 Java 应用程序中实现对象池时，我们想要做三件事：

+   确保我们的程序正常工作

+   证明我们的实现提高了性能

+   量化优化

在前面的章节中，我们探讨了如何在 Java 中实现对象池。在本节中，我们将探讨如何设计性能测试，如何实施对象池性能测试，以及如何分析测试结果。

## 设计性能测试

在我们决定实施性能测试后，我们的第一个行动是设计测试。我们需要回答的问题包括以下内容：

+   我们的目标是什么？

+   我们将测量什么？

+   我们将如何测量？

+   我们的测试将存在什么条件？

在考虑这些问题的基础上，我们可以开始设计我们的性能测试。我们应该为我们的性能测试设定一个明确的目标或一系列目标。例如，我们可能希望关注系统内存、CPU 负载等。

一旦我们有一个具体的目标，我们必须决定要测量什么。在测试中，我们测量的是被认为是**关键性能指标**（**KPIs**）。对象池的性能测试可能包括内存使用、CPU 使用、数据吞吐量和响应时间。这些只是其中的一些例子。

接下来，我们需要设置我们的测试环境并创建测试场景。测试环境紧密地复制了生产系统。你可能会在开发环境中复制你的系统，这样就不会影响实时系统。同样，测试场景应该紧密地反映你系统在现实世界中的使用。在尽可能的范围内，我们应该创建尽可能多的不同场景来代表我们的实时系统所处理的内容。

到目前为止，你已经准备好记录你的测试计划并实施它。下一节将介绍如何实施性能测试。

## 实施性能测试

实施你的测试计划不应该非常困难。在这里，你只是在将你的计划付诸实施。测试环境已经建立，你运行你的测试场景。在测试运行时，你应该收集数据以供后续分析。至关重要的是能够重现你的测试条件以支持未来的比较测试。

让我们看看如何使用本章中的数据库连接示例在 Java 中编写性能测试。我们的目标是减少应用程序从对象池获取数据库连接并在此数据库上执行简单操作所需的时间。我们的测试计划将比较我们的测试结果与未实现对象池的应用程序版本上的相同测试结果。

我们的代码从类声明和类变量开始：

```java
public class DBConnectionPerformanceTest {
  private static final int NUMBER_OF_TESTS = 3000;
  private static DBConnectObjectPool dbPool = new 
  DBConnectObjectPool(24);
```

接下来，我们将编写`main()`方法的第一部分。这段代码将是我们使用对象池进行测试的方式：

```java
public static void main(String[] args) {
  long startTime_withPooling = System.nanoTime();
  for (int i = 0; i < NUMBER_OF_TESTS; i++) {
    DBConnect conn = dbPool.getConnection();
    conn.dbMethod();
    dbPool.releaseConnection(conn);
  }
  long endTime_withPooling = System.nanoTime();
```

现在，我们将编写代码来测试不使用对象池的情况：

```java
long startTime_withoutPooling = System.nanoTime();
  for (int i = 0; i < NUMBER_OF_TESTS; i++) {
    DBConnect conn = new DBConnect();
    conn.dbMethod();
    conn.dbConnectionClose();
  }
long endTime_withoutPooling = System.nanoTime();
```

在编写完两组性能测试后，我们需要添加计算和输出结果的能力。我们通过简单地从`endTime`值中减去`startTime`值并将其转换为毫秒来生成结果。然后我们将结果输出到控制台：

```java
long totalTime_withPooling = (endTime_withPooling - startTime_withPooling) / 1_000_000;
long totalTime_withoutPooling = (endTime_withoutPooling - startTime_withoutPooling) / 1_000_000;
System.out.println("Total time with object pooling: " + totalTime_withPooling + " ms");
System.out.println("Total time without object pooling: " + totalTime_withoutPooling + " ms");
```

这个简单的对象池性能测试示例旨在给你一个如何编写这些测试的一般概念。每个应用程序都是不同的，你编写性能测试的方式也会有所不同。

## 分析结果

一旦我们的测试完成，我们就可以分析结果。你如何分析结果将取决于你的目标和关键绩效指标（KPIs）。分析任务不应仓促完成。记住，你收集这些数据是为了帮助你在对象池上做出决策。复杂性将根据性能测试计划而变化。

以数据库连接为例，我们可以简单地将它添加到`DBConnectionPerformanceTest`类的底部，以比较两组结果。以下是该代码的第一部分：

```java
if (totalTime_withPooling < totalTime_withoutPooling) {
  System.out.println("Results with object pooling: " + totalTime_
  withPooling);
  System.out.println("Results without object pooling: " + totalTime_
  withoutPooling);
  System.out.println("Analysis: Object pooling is faster by " + 
  (totalTime_withoutPooling - totalTime_withPooling) + " ms");
}
```

如您所见，我们只是检查`totalTime_withPooling`是否小于`totalTime_withoutPooling`。如果是这种情况，相关结果将在控制台上显示。

接下来，我们将检查`totalTime_withPooling`是否大于`totalTime_withoutPooling`。相关结果将在控制台上显示：

```java
} else if (totalTime_withPooling > totalTime_withoutPooling) {
  System.out.println("Results with object pooling: " + totalTime_
  withPooling);
  System.out.println("Results without object pooling: " + totalTime_
  withoutPooling);
  System.out.println("Analysis: Object pooling is slower by " + 
  (totalTime_withPooling - totalTime_withoutPooling) + " ms");
}
```

当前两个条件不满足时，我们的最终代码片段将执行。这意味着两次测试花费了相同的时间：

```java
} else {
  System.out.println("Results with object pooling: " + totalTime_
  withPooling);
  System.out.println("Results without object pooling: " + totalTime_
  withoutPooling);
  System.out.println("Analysis: No significant time difference between 
  object pooling and non-pooling.");
}
```

与所有测试一样，你应该记录你的计划、测试结果、分析、结论以及测试后的行动。这种稳健的文档方法将帮助你详细保留测试的历史。

# 摘要

本章深入探讨了 Java 对象池。建议对象池是确保我们的 Java 应用程序在高水平上运行的重要技术。在掌握理论知识的基础上，本章探讨了对象池的优点和缺点。我们关注了内存、CPU 使用和代码复杂度等方面。最后，我们展示了如何创建性能测试计划，如何实施它，以及如何分析结果。

在下一章中，我们将关注算法效率。我们的目标将是确保我们的算法具有低时间复杂度。本章将展示低效算法以及如何将它们转换为支持高性能。
