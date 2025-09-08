# 第四章：Java 9 语言增强

在上一章中，我们了解了 Java 9 中包含的一些令人兴奋的新特性。我们的重点是 javac、JDK 库和测试套件。我们学习了内存管理改进，包括内存分配、堆优化和增强的垃圾回收。我们还涵盖了编译过程、类型测试、注解和运行时编译器测试的更改。

本章涵盖了 Java 9 中的一些变化，这些变化影响了变量处理器、弃用警告、在 Java 7 中实施的 Project Coin 更改的改进以及导入语句处理。这些代表了 Java 语言本身的变更。

我们将在这里讨论的主题包括：

+   变量处理器

+   导入语句弃用警告

+   Project Coin

+   导入语句处理

# 与变量处理器协同工作 [JEP 193]

变量处理器是对变量的类型引用，并由`java.lang.invoke.VarHandle`抽象类管理。`VarHandle`方法的签名是多态的。这为方法签名和返回类型提供了极大的可变性。以下是一个代码示例，展示了如何使用`VarHandle`：

```java
    . . . 

    class Example 
    {
      int myInt;
      . . . 
    }
    . . . 
    class Sample 
    {
      static final VarHandle VH_MYINT;

      static 
      {
        try 
        {
          VH_MYINT =  
            MethodHandles.lookup().in(Example.class)
            .findVarHandle(Example.class, "myInt", int.class);
        } 
        catch (Exception e) 
        {
          throw new Error(e);
        }
      }
    }

    . . . 
VarHandle.lookup() performs the same operation as those that are performed by a MethodHandle.lookup() method.
```

这个 JEP 的目的是标准化以下类的方法调用方式：

+   `java.util.concurrent.atomic`

+   `sun.misc.Unsafe`

特别是以下方法：

+   访问/修改对象字段

+   访问/修改数组元素

此外，这个 JEP 导致了两个用于内存排序和对象可达性的栅栏操作。本着尽职尽责的精神，特别关注确保 JVM 的安全性。重要的是要确保这些更改不会导致内存错误。数据完整性、可用性和当然，性能是上述尽职尽责的关键组成部分，以下将进行解释：

+   **安全性**：必须不可能出现损坏的内存状态。

+   **数据完整性**：确保访问对象字段使用与以下相同的规则：

    +   `getfield`字节码

    +   `putfield`字节码

+   **可用性**：可用性的基准是`sun.misc.Unsafe` API。目标是使新的 API 比基准更容易使用。

+   **性能**：与使用`sun.misc.Unsafe` API 相比，性能可能不会下降。目标是超越该 API。

在 Java 中，栅栏操作是 javac 强制对内存施加约束的形式，这种约束以屏障指令的形式出现。这些操作发生在屏障指令之前和之后，实际上是将它们围起来。

# 使用 AtoMiC 工具包协同工作

`java.util.concurrent.atomic`包是一组 12 个子类，支持对线程安全和无锁的单个变量的操作。在这种情况下，线程安全指的是访问或修改共享单个变量的代码，而不会妨碍同时在该变量上执行的其它线程。这个超类是在 Java 7 中引入的。

这里是 AtoMiC 工具包中的 12 个子类的列表。正如你所期望的，类名是自我描述的：

| **原子子类** |
| --- |
| `java.util.concurrent.atomic.AtomicBoolean` |
| `java.util.concurrent.atomic.AtomicInteger` |
| `java.util.concurrent.atomic.AtomicIntegerArray` |
| `java.util.concurrent.atomic.AtomicIntegerFieldUpdater<T>` |
| `java.util.concurrent.atomic.AtomicLong` |
| `java.util.concurrent.atomic.AtomicLongArray` |
| `java.util.concurrent.atomic.AtomicLongFieldUpdater<T>` |
| `java.util.concurrent.atomic.AtomicMarkableReference<V>` |
| `java.util.concurrent.atomic.AtomicReference<V>` |
| `java.util.concurrent.atomic.AtomicReferenceArray<E>` |
| `java.util.concurrent.atomic.AtomicReferenceFieldUpdater<T,V>` |
| `java.util.concurrent.atomic.AtomicStampedReference<V>` |

可变变量、字段和数组元素可以被并发线程异步修改。

在 Java 中，`volatile` 关键字用于通知 javac 工具从主内存中读取值、字段或数组元素，而不是将其缓存。

这里有一个代码片段，演示了如何使用 `volatile` 关键字来对一个实例变量进行操作：

```java
    public class Sample 
    {
      private static volatile Sample myVolatileVariable; // a
       volatile instance variable

      public static Sample getVariable() // getter method
      {
        if (myVolatileVariable != null) 
        {
          return myVolatileVariable;
        }
        // this section executes if myVolatileVariable == null
        synchronized(Sample.class)
        {
          if (myVolatileVariable == null)
          {
            myVolatileVariable =  new Sample();
          }
        }
    }
```

# 使用 sun.misc.Unsafe 类

`sun.misc.Unsafe` 类，像其他 `sun` 类一样，没有官方文档或支持。它被用来绕过 Java 内置的某些内存管理安全特性。虽然这可以被视为对我们代码中更大控制和灵活性的窗口，但它是一种糟糕的编程实践。

该类有一个单独的私有构造函数，因此无法轻松实例化该类的实例。因此，如果我们尝试使用 `myUnsafe = new Unsafe()` 实例化一个实例，在大多数情况下会抛出 `SecurityException`。这个相对难以触及的类有超过 100 个方法，允许对数组、类和对象进行操作。以下是这些方法的简要示例：

| **数组** | **类** | **对象** |
| --- | --- | --- |
| `arrayBaseOffset` | `defineAnonymousClass` | `allocateInstance` |
| `arrayIndexScale` | `defineClass` | `objectFieldOffset` |
|  | `ensureClassInitialized` |  |
|  | `staticFieldOffset` |  |

这里是 `sun.misc.Unsafe` 类方法的次要分组，按信息、内存和同步进行分组：

| **信息** | **内存** | **同步** |
| --- | --- | --- |
| `addressSize` | `allocateMemory` | `compareAndSwapInt` |
| `pageSize` | `copyMemory` | `monitorEnter` |
|  | `freeMemory` | `monitorExit` |
|  | `getAddress` | `putOrderedEdit` |
|  | `getInt` | `tryMonitorEnter` |
|  | `putInt` |  |

`sun.misc.Unsafe` 类在 Java 9 中被标记为移除。实际上，在编程行业中对此决定有一些反对意见。为了平息他们的担忧，该类已被弃用，但不会完全移除。可以向 JVM 发送一个特殊标志来利用原始 API。

# 在导入语句中省略弃用警告 [JEP 211]

这是一些更简单的 Java 9 JEP 之一。在编译我们的程序时，我们经常会收到许多警告和错误。编译器错误必须修复，因为它们通常是语法性的。另一方面，警告应该被审查并适当处理。一些警告消息被开发者忽略。

此 JEP 在我们收到的警告数量上提供了一些缓解。具体来说，由导入语句引起的过时警告不再生成。在 Java 9 之前，我们可以使用以下注解来抑制过时警告消息：

```java
    @SupressWarnings
```

现在，随着 Java 9 的推出，编译器将在以下情况之一为真时抑制已过时的警告：

+   如果使用 `@Deprecated` 注解

+   如果使用 `@SuppressWarnings` 注解

+   如果生成警告的代码和声明在祖先类中

+   如果生成警告的代码在导入语句中使用

列出的第四个条件是在 Java 9 中添加的。

# Milling Project Coin [JEP 213]

Project Coin 是在 Java 7 中引入的微小更改的功能集。以下列出了这些更改：

+   `switch` 语句中的字符串

+   二进制整数字面量

+   在数字字面量中使用下划线

+   实现多捕获

+   允许更精确地重新抛出异常

+   泛型实例创建改进

+   添加 `try-with-resources` 语句

+   调用 `varargs` 方法的改进

详细信息可以在以下 Oracle 演示文稿中找到：[`www.oracle.com/us/technologies/java/project-coin-428201.pdf`](http://www.oracle.com/us/technologies/java/project-coin-428201.pdf)。

JEP 213 专注于 Project Coin 的改进。共有五个这样的改进，如下详细说明。

# 使用 `@SafeVarargs` 注解

在 Java 9 中，我们可以使用 `@SafeVarargs` 注解与私有实例方法。当我们使用此注解时，我们断言该方法不包含对作为方法参数传递的 `varargs` 执行任何有害操作。

使用语法为：

```java
    @SafeVarargs // this is the annotation
    static void methodName(...) 
    {

      /*
      The contents of the method or constructor must not 
      perform any unsafe or potentially unsafe operations 
      on the varargs parameter or parameters.
      */

    }
```

`@SafeVarargs` 注解的使用限制为：

+   静态方法

+   最终实例方法

+   私有实例方法

# `try-with-resource` 语句

在 Java 9 之前，`try-with-resource` 语句在每次使用最终变量时，需要为语句中的每个资源声明一个新变量。以下是 Java 9（Java 7 或 8）之前 `try-with-resource` 语句的语法：

```java
    try ( // open resources ) 
    {
      // use resources
    } catch (// error) 
    {  // handle exceptions
    }
    // automatically close resources
```

这里是一个使用上述语法的代码片段：

```java
    try ( Scanner xmlScanner = new Scanner(new File(xmlFile));
    {
       while (xmlScanner.hasNext())
       {
          // read the xml document and perform needed operations
       }
      xmlScanner.close();
    } catch (FileNotFoundException fnfe)
      {
         System.out.println("Your XML file was not found.");
      }
```

现在，随着 Java 9 的推出，`try-with-resource` 语句可以管理最终变量，而无需声明新变量。因此，我们现在可以像下面这样重写之前的代码，这是在 Java 9 中的示例：

```java
    Scanner xmlScanner = new Scanner(newFile(xmlFile));
    try ( while (xmlScanner.hasNext())
    {
       {
         // read the xml document and perform needed operations
       }
       xmlScanner.close();
    } catch (FileNotFoundException fnfe)
      {
         System.out.println("Your XML file was not found.");
      }
```

如您所见，`xmlScanner` 对象引用包含在 `try-with-resource` 语句块中，这提供了自动资源管理。资源将在退出 `try-with-resource` 语句块后自动关闭。

您还可以将`finally`块作为`try-with-resource`语句的一部分使用。

# 使用菱形运算符

在 Java 9 中引入的菱形运算符，如果推断的数据类型是可表示的，可以与匿名类一起使用。当推断数据类型时，它表明 Java 编译器可以确定方法调用中的数据类型。这包括声明和任何包含的参数。

菱形运算符是小于和大于符号对（`<>`）。它对 Java 9 来说并不新鲜；相反，它与匿名类的特定使用才是。

菱形运算符是在 Java 7 中引入的，它使泛型类的实例化变得更加简单。以下是一个 Java 7 之前的示例：

```java
    ArrayList<Student> roster = new ArrayList<Student>();
```

然后，在 Java 7 中，我们可以这样重写它：

```java
    ArrayList<Student> roster = new ArrayList<>();
```

问题在于这个方法不能用于匿名类。以下是一个 Java 8 中的示例，它运行良好：

```java
    public interface Example<T> 
    {
      void aMethod()
      {
        // interface code goes here
      }
    }

    Example example = new Example<Integer>() 
    {
      @Override
      public void aMethod() 
      {
        // code
      }
    };
```

虽然前面的代码运行良好，但当我们将其更改为使用菱形运算符，如这里所示时，将发生编译器错误：

```java
    public interface Example<T> 
    {
      void aMethod()
      {
        // interface code goes here
      }
    }

    Example example = new Example<>() 
    {
      @Override
      public void aMethod() 
      {
        // code
      }
    };
```

错误是由于使用菱形运算符与匿名内部类引起的。Java 9 拯救了这个问题。虽然前面的代码在 Java 8 中会导致编译时错误，但在 Java 9 中它运行良好。

# 停止使用下划线

下划线字符（`_`）不能再用作合法的标识符名称。之前尝试从标识符名称中删除下划线的尝试是不完整的。这样的使用会产生错误和警告的组合。在 Java 9 中，这些警告现在是错误。考虑以下示例代码：

```java
    public class Java9Tests 
    { 
      public static void main(String[] args) 
      {
        int _ = 319;
        if ( _ > 300 )
        {
          System.out.println("Your value us greater than 300."); 
        } 
        else 
        {
          System.out.println("Your value is not greater than 300.");
        }
      }
    }
```

在 Java 8 中，前面的代码会导致`int _ = 319;`和`if (_ > 300)`语句的编译器警告。警告是：从版本 9 开始，`_`是一个关键字，不能用作标识符。因此，在 Java 9 中，您将无法单独使用下划线作为合法的标识符。

被认为是不良的编程实践使用非自我描述的标识符名称。因此，将下划线字符单独用作标识符名称不应是一个有问题的变化。

# 使用私有接口方法

Lambda 表达式是 Java 8 发布的一个重要部分。作为对该改进的后续，接口中的私有方法现在变得可行。之前，我们无法在接口的非抽象方法之间共享数据。在 Java 9 中，这种数据共享成为可能。接口方法现在可以是私有的。让我们看看一些示例代码。

这个第一个代码片段是我们在 Java 8 中编写接口的方式：

```java
    . . . 
    public interface characterTravel
    {
      pubic default void walk()
      {
        Scanner scanner = new Scanner(System.in);
        System.out.println("Enter desired pacing: ");
        int p = scanner.nextInt();
        p = p +1;
      }
      public default void run()
      {
        Scanner scanner = new Scanner(System.in);
        System.out.println("Enter desired pacing: ");
        int p = scanner.nextInt();
        p = p +4;
      }
      public default void fastWalk()
      {
        Scanner scanner = new Scanner(System.in);
        System.out.println("Enter desired pacing: ");
        int p = scanner.nextInt();
        p = p +2;
      }
      public default void retreat()
      {
        Scanner scanner = new Scanner(System.in);
        System.out.println("Enter desired pacing: ");
        int p = scanner.nextInt();
        p = p - 1;
      }
      public default void fastRetreat()
      {
        Scanner scanner = new Scanner(System.in);
        System.out.println("Enter desired pacing: ");
        int p = scanner.nextInt();
        p = p - 4;
      }
    }
```

现在，在 Java 9 中，我们可以重写这段代码。正如您接下来可以看到的，冗余的代码已经被移动到一个名为`characterTravel`的单独私有方法中：

```java
    . . . 
    public interface characterTravel
    {
      pubic default void walk()
      {
        characterTravel("walk");
      }
      public default void run()
      {
        characterTravel("run");
      }
      public default void fastWalk()
      {
        characterTravel("fastWalk");
      }
      public default void retreat()
      {
        characterTravel("retreat");
      }
      public default void fastRetreat()
      {
        characterTravel("fastRetreat");
      }
      private default void characterTravel(String pace)
      {
        Scanner scanner = new Scanner(System.in);
        System.out.println("Enter desired pacing: ");
        int p = scanner.nextInt();
        if (pace.equals("walk"))
        {
          p = p +1;
        }
        else if (pace.equals("run"))
        {
          p = p + 4;
        }
        else if (pace.equals("fastWalk"))
        {
          p = p + 2;
        }
        else if (pace.equals("retreat"))
        {
          p = p - 1;
        }
        else if (pace.equals("fastRetreat"))
        {
          p = p - 4;
        }
        else
        {
          //
        }
```

# 正确处理导入语句 [JEP 216]

JEP 216 是为了修复 javac 在处理导入语句方面的缺陷而发布的。在 Java 9 之前，存在一些情况下导入语句的顺序会影响源代码是否被接受。

当我们在 Java 中开发应用程序时，我们通常会根据需要添加导入语句，从而产生一个无序的导入语句列表。IDEs 在对未使用的导入语句进行着色编码方面做得很好，同时也通知我们那些需要但尚未包含的导入语句。导入语句的顺序无关紧要；没有适用的层次结构。

javac 以两个主要步骤编译类。针对处理导入语句，这些步骤是类型解析和成员解析。类型解析包括对抽象语法树的审查，以识别类和接口的声明。成员解析包括确定类层次结构和单个类变量和成员。

在 Java 9 中，我们在类和文件中列出导入语句的顺序将不再影响编译过程。让我们来看一个例子：

```java
    package samplePackage;

    import static SamplePackage.OuterPackage.Nested.*;
    import SamplePackage.Thing.*;

    public class OuterPackage 
    {
      public static class Nested implements Inner 
      { 
        // code
      }
    }

    package SamplePackage.Thing;

    public interface Inner 
    {
      // code
    }
```

在前面的例子中，类型解析发生并导致以下认识：

+   `SamplePackage.OuterPackage` 存在

+   `SamplePackage.OuterPackage.Nested` 存在

+   `SamplePackage.Thing.Innner` 存在

下一步是成员解析，这正是 Java 9 之前存在的问题所在。以下是 javac 为我们的示例代码执行成员解析的顺序步骤概述：

1.  `SamplePackage.OuterPackage` 的解析开始。

1.  处理 `SamplePackage.OuterPackage.Nested` 的导入。

1.  `SamplePackage.Outer.Nested` 类的解析开始。

1.  内部接口进行了类型检查，尽管由于它目前不在作用域内，内部无法解析。

1.  `SamplePackage.Thing` 的解析开始。这一步骤包括将 `SamplePackage.Thing` 的所有成员类型导入作用域。

因此，在我们的例子中，错误发生是因为在尝试解析时 `Inner` 超出了作用域。如果步骤 4 和 5 互换，就不会有问题。

Java 9 中实现的问题解决方案是将成员解析步骤分解为额外的子步骤。以下是这些步骤：

1.  分析导入语句。

1.  创建层次结构（类和接口）。

1.  分析类头和类型参数。

# 摘要

在本章中，我们介绍了 Java 9 中关于变量处理器及其与 Atomic Toolkit 相关性的变化。我们还介绍了弃用警告及其为何在特定情况下被抑制的原因。还回顾了 Java 7 作为 Project Coin 部分引入的五个变化增强。最后，我们探讨了导入语句处理的改进。

在下一章中，我们将检查由 Project Jigsaw 指定的 Java 模块结构。我们将深入探讨 Project Jigsaw 作为 Java 平台一部分的实现方式。本章中使用了来自一个示例电子商务应用程序的代码片段，以展示 Java 9 的模块化系统。还讨论了 Java 平台在模块化系统方面的内部变化。
