# 13. 使用 Lambda 表达式进行函数式编程

概述

本章讨论了 Java 如何成为函数式编程语言。它还详细说明了在 Java 中如何使用 lambda 表达式进行模式匹配。它首先通过一般性地解释面向对象编程（OOP）和函数式编程（FP）之间的区别来实现这一点。然后，你将学习纯函数的基本定义，以及函数式接口和普通接口之间的区别。最后，你将练习使用 lambda 表达式作为回调事件，并使用它们来过滤数据。

# 简介

虽然 Java 已经存在了 20 多年，而函数式编程（FP）甚至比 Java 还要长，但直到最近，FP 这个话题才在 Java 社区中引起关注。这可能是由于 Java 本质上是一种命令式编程语言；当学习 Java 时，你学习的是面向对象编程（OOP）。

然而，在过去的几年里，主流编程社区的动向已经更多地转向了 FP。如今，你可以在每个平台上看到这一点——从网络到移动再到服务器。FP 概念无处不在。

## 背景

尽管在 Java 中是一个相对较新的话题，但函数式编程（FP）已经存在很长时间了。事实上，它甚至比第一台个人电脑还要早；它的起源可以追溯到 20 世纪 30 年代 Alonzo Church 创建的 lambda 演算研究。

“lambda”这个名字来源于希腊符号，这是丘奇在描述他的 lambda 演算的规则和数学函数时决定使用的符号。

lambda 身份函数非常简单，就是一个返回输入参数的函数——即，身份。在一个更正常的数学脚本中。

如你所见，lambda 演算是一种用于表达数学方程的简单方法。然而，它不一定是数学性的。在其最纯粹的形式中，它是一个具有一个参数和发生算术运算的体的函数。在 lambda 演算中，函数是一等公民——这意味着它可以像任何其他变量一样被对待。如果你需要在函数中具有多个属性，你甚至可以组合多个 lambda。

# 函数式编程

函数式编程（FP）归结为两件事：副作用和确定性。这些概念构成了我们所说的 FP 的基础，它们也是新来者在这个范式中最容易掌握的两个概念，因为它们没有引入新的、复杂的模式。

## 副作用

当编写程序时，我们常常努力获得某种形式的副作用——一个没有副作用的程序是一个非常无聊的程序，因为什么都不会发生。然而，当试图可靠地测试程序时，副作用也是一个常见的头痛问题，因为其状态可能会不可预测地改变。

Java 中一个非常实用的类是 `Math` 类；它包含各种数学辅助工具，并且很可能会在所有 Java 应用程序中直接或间接地被使用。以下是一个将伪随机数打印到控制台的示例：

```java
public static void main(String[] args) {
    System.out.println(Math.random());
}
```

如果我们深入研究 `Math.java` 的代码并回顾 `random()` 函数的细节，我们会注意到它使用了一个不属于 `random()` 函数的 `randomNumberGenerator` 对象：

```java
public final class Math {
    public static double random() {
        return RandomNumberGeneratorHolder.randomNumberGenerator.nextDouble();
    }
}
```

它还调用了 `randomNumberGenerator` 对象的 `nextDouble()` 方法。这就是我们所说的副作用；`random` 函数超出其自身的范围，或主体，并对其他变量或类进行更改。这些变量反过来又可能被其他函数或对象调用，这些函数或对象可能会也可能不会产生自己的副作用。当你试图以 FP 风格实现程序时，这种行为是一个红旗，因为它不可预测。它也可能更难在多线程环境中安全使用。

注意

`Math.random()` 函数按照设计提供不可预测的结果。然而，作为一个例子，它很好地为我们突出了副作用的概念。`random` 函数在多线程环境中（大部分情况下）也是安全的——Sun 和 Oracle 已经完成了他们的作业！

由于 `Math.random()` 函数对于相同的参数会产生不同的结果，因此它被定义为非确定性函数。

## 确定性函数

确定性函数被定义为对于相同的参数，无论执行多少次或何时执行，都会始终产生相同结果的函数：

```java
public static void main(String[] args) {
    System.out.println(Math.random());
    System.out.println(Math.random());
}
```

在这个例子中，`Math.random()` 被调用了两次，并且总是会打印出两个不同的值到终端。无论你调用多少次 `Math.random()`，它总是会给出不同的结果——因为按照设计，它不是确定性的：

```java
public static void main(String[] args) {
    System.out.println(Math.toRadians(180));
    System.out.println(Math.toRadians(180));
}
```

运行这段简单的代码，我们可以看到 `Math.toRadians()` 函数对两个函数都会给出相同的结果，并且似乎不会改变程序中的其他任何东西。这是一个提示，表明它是确定性的——让我们深入研究这个函数并回顾它：

```java
public final class Math {
    private static final double DEGREES_TO_RADIANS = 0.017453292519943295;

    public static double toRadians(double angdeg) {
        return angdeg * DEGREES_TO_RADIANS;
    }
}
```

如预期，该函数不会从外部世界改变任何东西，并且总是产生相同的结果。这意味着我们可以将其视为确定性函数。然而，它确实读取一个存在于函数作用域之外的常量；这是我们所说的**纯函数**的一个边缘情况。

# 纯函数

最纯的函数可以被认为是黑盒，这意味着函数内部发生的事情对程序员来说并不真正感兴趣。他们只对放入盒子的内容以及作为结果从盒子里出来的内容感兴趣——因为纯函数总是会有一个结果。

纯函数接受参数并根据这些参数产生结果。纯函数永远不会改变外部世界的状态或依赖它。函数所需的所有内容都应该在函数内部可用，或者作为输入传递给它。

## 练习 1：编写纯函数

一家杂货店有一个管理系统来管理他们的库存；然而，构建他们软件的公司已经破产，并且丢失了他们系统的所有源代码。这是一个只允许客户一次购买一件东西的系统。因为他们的客户希望一次购买两件东西，不多也不少，所以他们要求你实现一个函数，该函数接受两种产品的价格并返回这两个价格的总额。他们希望你在不造成任何副作用或与当前系统不兼容的情况下实现这一点。你将作为一个纯函数来实现它：

1.  如果 IntelliJ 已经启动但没有打开项目，那么请选择 `创建新项目`。如果 IntelliJ 已经打开了一个项目，那么请从菜单中选择 `文件` -> `新建` -> `项目`。

1.  在 `新建项目` 对话框中，选择 `Java 项目`。然后，点击 `下一步`。

1.  打开复选框以从模板创建项目。选择 `命令行应用程序`。然后，点击 `下一步`。

1.  给新项目命名为 `Exercise1`。

1.  IntelliJ 会为你提供一个默认的项目位置；如果你希望选择一个，可以在这里输入。

1.  将包名设置为 `com.packt.java.chapter13`。

1.  点击`完成`。IntelliJ 将创建你的项目，命名为 `Exercise1`，并使用标准的文件夹结构。IntelliJ 还将创建你的应用程序的主入口点，命名为 `Main.java`；它将类似于以下代码片段：

    ```java
    package com.packt.java.chapter13;
    public class Main {
        public static void main(String[] args) {
        // write your code here
        }
    }
    ```

1.  将 `Main.java` 重命名为 `Exercise1.java`。

1.  在 `Main` 类中创建一个新的函数，将其放置在 `main(String[] args)` 函数之下。将新函数命名为 `sum` 并让它返回一个整数值。这个函数应该接受两个整数作为输入。为了代码的简洁性，我们将这个函数做成一个 `static` 工具函数：

    ```java
    package com.packt.java.chapter13;
    public class Exercise1 {
        public static void main(String[] args) {
        // write your code here
        }
        static int sum(int price1, int price2) {
        }
    }
    ```

1.  这个函数应该做的只是返回两个参数——`price1` 和 `price2` 的总和：

    ```java
    package com.packt.java.chapter13;
    public class Exercise1 {
        public static void main(String[] args) {
        // write your code here
        }
        static int sum(int price1, int price2) {
            return price1 + price2;
        }
    }
    ```

1.  在你的 `main` 函数中使用相同的参数调用这个新方法几次：

    ```java
    package com.packt.java.chapter13;
    public class Exercise1 {
        public static void main(String[] args) {
            System.out.println(sum(2, 3));
            System.out.println(sum(2, 3));
            System.out.println(sum(2, 3));
        }
        static int sum(int price1, int price2) {
            return price1 + price2;
        }
    }
    ```

1.  现在运行你的程序并观察输出。

    备注

    `System.out.println()` 方法被许多人视为不纯函数，因为它操作终端——当然，这是“外部世界”，因为在调用堆栈的某个点上，函数将超出其范围来操作一个 `OutputStream` 实例。

你刚才编写的函数接受两个参数并产生一个全新的输出，而不修改函数范围之外的内容。通过这一点，你已经成功迈出了编写更函数式应用程序的第一步。

在编写函数式程序时，另一个重要的考虑因素是如何处理应用程序中的状态。在面向对象编程（OOP）中，我们通过使用分而治之的策略来解决大型应用程序中状态处理的问题。在这里，应用程序中的每个对象都包含整个应用程序状态的一小部分。

这种类型状态处理的隐含属性是状态的拥有权和可变性。每个对象通常都有一个私有状态，可以通过公共接口——对象的方法来访问——即对象的方法。例如，如果我们回顾 OpenJDK 源代码中的`ParseException.java`类，我们也会找到这个模式：

```java
package java.text;
public class ParseException extends Exception {
    private static final long serialVersionUID = 2703218443322787634L;
    public ParseException(String s, int errorOffset) {
        super(s);
        this.errorOffset = errorOffset;
    }
    public int getErrorOffset() {
        return errorOffset;
    }
    private int errorOffset;
}
```

在这里，我们可以看到一个名为`errorOffset`的私有成员变量。这个成员变量可以从构造函数中写入，并且可以通过`getErrorOffest()`方法被其他对象访问。我们还可以想象一个具有另一个更改`errorOffset`值的方法的类——即一个 setter。

使用这种状态处理方法可能存在的一个问题是多线程应用程序。如果有两个或更多线程要读取或写入这个成员变量，我们通常会看到不可预测的变化。当然，在 Java 中，我们可以通过使用同步来修复这些变化。然而，同步是有代价的；准确规划访问很复杂，而且我们经常遇到竞态条件。在任何支持它的语言中，这也是一个相当昂贵的程序。

注意

使用同步非常流行，并且是构建多线程应用程序的一种安全方式。然而，同步的一个缺点——除了它非常昂贵之外——是它实际上使我们的应用程序表现得像一个单线程应用程序，因为所有访问同步数据的线程都必须等待它们的轮次来处理数据。

在 FP 中，我们试图避免使用同步，而是说我们的状态应该是始终不可变的——有效地消除了同步的需要。

## 状态不可变性

当状态是不可变的时候，本质上意味着它永远不能改变。在 FP 中，有一种常见的写法来描述这个规则，大致是这样的：替换你的数据而不是就地编辑它。

如我们在第三章*面向对象编程*中讨论的，OOP 的一个核心概念是继承；即创建子类的能力，这些子类基于或继承父类中已经存在的功能，但也可以向子类添加新功能。在函数式编程（FP）中，这变得相对复杂，因为我们针对的是永远不会改变的数据。

在 Java 中使数据不可变的最简单方法是通过使用`final`关键字。在 Java 中有三种使用`final`关键字的方法：锁定变量以更改，使方法无法被覆盖，以及使类无法扩展。当在 Java 中构建不可变数据结构时，仅仅使用这些方法中的任何一个通常是不够的；我们需要使用两个或有时甚至所有三个。

## 练习 2：创建一个不可变类

一个当地木匠在你的街道上开设了商店，并要求你为他们构建一个简单的购物车应用程序的存储机制，他们将内部使用这个应用程序来处理订购家具的人。该应用程序应该能够安全地处理来自不同线程的多人同时编辑。销售人员将通过电话接收订单，木匠将编辑花费的小时数和使用的材料。购物车必须是不可变的。为此，执行以下步骤：

1.  在 IntelliJ 的`Project`面板中，右键单击名为`src`的文件夹。

1.  在菜单中选择`New` -> `Java Class`，并输入`Exercise2`。

1.  在你的新类中定义`main`方法：

    ```java
    package com.packt.java.chapter13;
    public class Exercise2 {
        public static void main(String[] args) {
        }
    }
    ```

1.  创建一个新的内部类`ShoppingCart`，并将其设置为`final`以确保它不能被扩展或改变其行为。你的代码现在可能看起来像这样：

    ```java
    package com.packt.java.chapter13;
    public class Exercise2 {
        public static void main(String[] args) {
        }
        public static final class ShoppingCart {
        }
    }
    ```

1.  我们还需要将项目放入这个购物车中，因此为`ShoppingItem`创建一个简单的数据对象，给它一个名称和价格属性，然后使这个类不可变。我们稍后会使用这个类来实例化几个不同的对象，以测试我们`ShoppingCart`类的可变性：

    ```java
        public static final class ShoppingCart{
        }
        private static final class ShoppingItem {
            private final String name;
            private final int price;
            public ShoppingItem(String name, int price) {
                this.name = name;
                this.price = price;
            }
        }
    ```

1.  添加一个列表，我们将在这个不可变购物车中保存所有项目。确保使用`final`关键字声明这个列表，使其不可更改：

    ```java
    package com.packt.java.chapter13;
    import java.util.ArrayList;
    import java.util.List;
    public class Exercise2 {

        public static final class ShoppingCart{
            private final List<ShoppingItem> mShoppingList = new ArrayList<>();
        }

    }
    ```

    现在我们有了一种为我们的客户创建购买项目的方法，我们也为我们的客户提供了一个放置所选项目的袋子。然而，我们缺少一种让我们的客户能够将项目添加到购物车中的方法。

1.  在面向对象的方法中解决这个问题时，我们可以添加一个名为`addItem(ShoppingItem shoppingItem)`的方法：

    ```java
    package com.packt.java.chapter13;
    import java.util.ArrayList;
    import java.util.List;
    public class Exercise2 {
        private final class ShoppingCart{
            private final List<ShoppingItem> mShoppingList = new ArrayList<>();

            public void addItem(ShoppingItem item) {
                mShoppingList.add(item);
            }
        }

    }
    ```

    从函数式编程的角度来看这个解决方案，我们可以看到它将修改集合。这是我们极力避免的事情，因为多人将同时在这个购物车上工作。在这种情况下，使用`final`关键字没有影响，因为 final 列表的内容仍然可以改变。解决这个问题的基本方法之一是在添加项目时返回一个新的`ShoppingCart`对象。

1.  向`ShoppingCart`类添加一个新的构造函数，并让它接受一个列表作为参数。然后，将这个列表传递给`ShoppingCart`类的`mShoppingList`，并使用`Collections.unmodifiableList()`方法使其不可修改：

    ```java
    package com.packt.java.chapter13;
    import java.util.ArrayList;
    import java.util.Collections;
    import java.util.List;
    public class Exercise2 {
        public static final class ShoppingCart{
            public final List<ShoppingItem> mShoppingList;
            public ShoppingCart(List<ShoppingItem> list) {
                mShoppingList = Collections.unmodifiableList(list);
            }
            public void addItem(ShoppingItem item) {
                mShoppingList.add(item);
            }
        }
    }
    ```

1.  重写`addItem(ShoppingItem item)`方法，让它返回一个新的`ShoppingCart`项目而不是`void`。将上一个`ShoppingCart`项目的列表复制到一个临时列表中，并添加另一个项目。然后，将这个临时列表传递给构造函数，并返回新创建的`ShoppingCart`对象：

    ```java
        public static final class ShoppingCart{
            public final List<ShoppingItem> mShoppingList;
            public ShoppingCart(List<ShoppingItem> list) {
                mShoppingList = Collections.unmodifiableList(list);
            }
            public ShoppingCart addItem(ShoppingItem item) {
                List<ShoppingItem> newList = new ArrayList<>(mShoppingList);
                newList.add(item);
                return new ShoppingCart(newList);
            }
        }
    ```

    在此代码中，我们可以看到构造函数现在接受`ShoppingItem`类的一个列表；我们还可以看到列表被直接保存为一个不可修改的列表。这是 Java 中的一种特殊列表——在您尝试以任何方式直接或通过其迭代器修改它时，它会抛出异常。

    我们还可以看到，`addItem(ShoppingItem item)`函数现在返回一个新的`ShoppingCart`，包含全新的列表，但两个`ShoppingCart`实例之间共享了之前的购物列表中的项目。这对于多线程环境来说是一个可接受的解决方案，因为`ShoppingItem`类是最终的，因此它们的状态可能永远不会改变。

    注意

    Java 8 引入了 Stream API，这是一种全新的处理集合的方式，即更基于函数式编程的方法。您可以在第十五章“使用 Stream 处理数据”中了解更多关于 Stream API 的信息。在本章中，我们将关注不使用 Stream API 的解决方案。

1.  现在您需要在程序中使用这个新的`ShoppingCart`。编辑您的`main`方法，然后先创建一个空的`ShoppingCart`。然后，向该购物车添加一个新的购物项，并将新创建的`ShoppingCart`存储在另一个变量中。最后，向第二个`ShoppingCart`添加另一个`ShoppingItem`，再次将新的`ShoppingCart`存储在新的变量中：

    ```java
    Exercise2.java
    7  public class Exercise2 {
    8  
    9      public static void main(String[] args) {
    10         ShoppingCart myFirstCart =             new ShoppingCart(new ArrayList<ShoppingItem>());
    11         ShoppingCart mySecondCart =             myFirstCart.addItem(new ShoppingItem("Chair", 150));
    12         ShoppingCart myThirdCart =             mySecondCart.addItem(new ShoppingItem("Table",350));
    13     }
    https://packt.live/2Jdr10l
    ```

1.  在最后一行设置断点并调试您的代码。您会注意到在调用`addItem`时创建的购物车维护着自己的不可修改的`ShoppingItem`列表，但不可变的`ShoppingItem`在列表之间是共享的。

`Collections.unmodifiableList()`方法和其他类似方法（如`Set`、`Map`和`SortedList`）并没有为列表本身提供任何不可变性。它们产生了一个禁止任何更改的列表视图。然而，任何拥有实际列表引用的人仍然能够更改数据。

在这个练习中，列表是安全的，因为`main`方法没有保留任何对列表的引用，所以没有人可以从外部更改它。然而，当尝试使用函数式方法实现程序时，这不是推荐的方法；除非他们必须严格遵循规则，否则不要相信任何人。自 Java 9 以来，现在有真正的不可变集合可用。

## 活动一：修改不可变列表

向您的`ShoppingCart`添加一个新的行为：

1.  创建一个`removeItem(ShoppingItem)`函数。

1.  创建一个函数，该函数接受多个`ShoppingItem`作为参数，可以是列表或可变参数。

1.  修改您的`ShoppingCart`以接受每个`ShoppingItem`的多个项目——例如，四把椅子和一张桌子。此外，修改`addItem(ShoppingItem)`和`removeItem(ShoppingItem)`函数。

    注意

    此活动的解决方案可在第 561 页找到。

## 不可变集合

使用 `Collections.unmodifiableList` 是提供现有列表不可修改版本的一种快速方法。自 Java 9 以来，另一个选项是使用具有工厂方法的不可变集合。这些工厂方法允许您创建三种不同的不可变集合类型：`List`、`Set` 和 `Map`。

注意

有几个库提供了更优化的不可变集合；一个流行的例子是 Guava，它有 `ImmutableArrayList` 和其他类型。

如果我们使用 `List` 工厂方法而不是 `Collections` 类来处理购物车，它可能看起来像这样：

```java
public class Main {
    public static final class ShoppingCart {
        public final List<ShoppingItem> mShoppingList;
        public ShoppingCart(List<ShoppingItem> list) {
            mShoppingList = List.copyOf(list);
        }
        public ShoppingCart addItem(ShoppingItem item) {
            List<ShoppingItem> newList = new ArrayList<>(mShoppingList);
            newList.add(item);
            return new ShoppingCart(newList);
        }
    }
}
```

在这里，我们可以看到与我们之前所拥有的几乎没有区别。我们不是使用 `Collections.unmodifiableList()` 来创建列表的不修改视图，而是使用 `List.copyOf()` 创建此列表的不可变副本。在我们的例子中，对用户来说这种差异是看不见的。然而，在底层，它们基于不同的实现——分别是 `UnmodifiableCollection` 和 `ImmutableCollections` 类。

## 练习 3：重写 String 方法

在这个练习中，我们将做一个小的技术证明，说明 `UnmodifiableCollection` 和 `ImmutableCollection` 类之间的差异。为此，我们需要重写 `ShoppingItem` 和 `ShoppingCart` 类的 `toString()` 方法：

1.  将 `toString()` 方法添加到 `ShoppingItem` 类中，然后让它返回名称：

    ```java
        private static final class ShoppingItem {
            @Override
            public String toString() {
                return name + ", " + price;
            }
        }
    ```

1.  将 `toString()` 方法添加到 `ShoppingCart` 类中。然后，让它返回列表中所有 `ShoppingItem` 的连接字符串：

    ```java
        public static final class ShoppingCart {
            public String toString() {
                StringBuilder sb = new StringBuilder("Cart: ");
                for (int i = 0; i < mShoppingList.size(); i++) {
                    sb.append(mShoppingList.get(i)).append(", ");
                }
                return sb.toString();
            }
        }
    ```

1.  现在我们有一个简单的方法来使用 `toString()` 方法打印 `ShoppingCart` 的内容。为了展示差异，替换 `main` 方法中的代码。向标准列表中添加几本书，然后将此列表复制到一个不可修改的版本和一个不可变版本。打印这两个副本：

    ```java
        public static void main(String[] args) {
            List<ShoppingItem> books = new ArrayList<>();
            books.add(new ShoppingItem("Java Fundamentals", 100));
            books.add(new ShoppingItem("Java 11 Quick Start", 200));
            List<ShoppingItem> immutableCopy = List.copyOf(books);
            List<ShoppingItem> unmodifiableCopy =           Collections.unmodifiableList(books);
            System.out.println(immutableCopy);
            System.out.println(unmodifiableCopy);
        }
    ```

1.  现在从原始的 `books` 列表中移除第一个项目，即《Java 基础知识》这本书，然后再次打印这两个副本：

    ```java
        public static void main(String[] args) {
            List<ShoppingItem> books = new ArrayList<>();
            books.add(new ShoppingItem("Java Fundamentals", 100));
            books.add(new ShoppingItem("Java 11 Quick Start", 200));
            List<ShoppingItem> immutableCopy = List.copyOf(books);
            List<ShoppingItem> unmodifiableCopy =           Collections.unmodifiableList(books);
            System.out.println(immutableCopy);
            System.out.println(unmodifiableCopy);
            books.remove(0);
            System.out.println(immutableCopy);
            System.out.println(unmodifiableCopy);
        }
    ```

这个简单的例子提供了不可修改视图和不可变副本之间差异的证明。在不可修改版本中，列表仍然可以被更改，并且不可修改视图将捕捉到这种更改，而不可变版本将忽略这种更改，因为它包含了一个新的项目列表。

## 功能接口

功能接口被声明为标准的 Java 接口，除了它们只能包含一个抽象函数，但可以包含任意数量的默认或静态函数。

`Comparator` 接口是 Java 中较老的接口之一。它自 1.2 版本以来一直伴随着我们，并且多年来经历了许多变化。然而，迄今为止最大的变化可能是 Java 8 中将其转变为功能接口。

回顾 Java 8 中 `Comparator` 接口的变化，你会注意到一些有趣的变化。首先，接口从 4 行代码增长到 80 行，不包括包声明和注释。然后，你会注意到顶部有一个新的注解：

```java
@FunctionalInterface
```

这个注解标记表明这是一个功能接口。它的主要目的是告诉读者，这个接口的目的是遵循 Java 8 中定义的功能接口规范。如果它未能遵循这些指南，Java 编译器应该打印出一个错误。

在两个原始的抽象函数声明之后，你会发现至少有七个默认函数。这些默认函数是在 Java 8 中引入的，目的是在不破坏向后兼容性的情况下向接口添加新功能。默认函数始终是公共的，并且总是包含一个代码块。它们可以返回一个值，但这不是规范所要求的。

最后，我们将找到总共九个 `static` 函数。自从 Java 8 以来，函数式接口可以包含任意数量的 `static` 方法，它们与普通类中找到的静态方法非常相似。你将在本书的后续章节中了解更多关于构建和使用函数式接口的细节。

# Lambda 表达式

随着 Java 8 中功能性的改进，也出现了 `Lambda` 表达式。Lambda 的一大主要改进是代码可读性——接口的大多数样板代码现在都不见了。

一个非常常用的接口是 **Runnable 接口**；它在多线程应用程序中用于在后台执行任何类型的任务，例如从网络下载大文件。在 Java 7 及更早版本中，你经常看到 Runnable 接口被用作匿名实例：

```java
new Thread(new Runnable() {
    @Override
    public void run() {
    }
}).start();
```

自从 Java 8 以来，前面的五行代码现在可以通过使用 lambda 表达式来简化：

```java
new Thread(() -> {}).start();
```

如你所见，当我们移除大量样板代码时，代码的可读性变得更高。

Lambda 表达式由两个主要部分组成：参数和主体。此外，在这两个部分之间，始终有一个箭头操作符（也称为 lambda 操作符）。主体还包含可选的返回值。括号包含 lambda 表达式的可选参数。尽管它是一个函数式编程组件，但你仍然会想使用参数：

```java
(int arg1, int arg2) -> { return arg1 + arg2; }
```

你也可以省略参数的类型，因为它们将由 lambda 表达式实现的函数式接口推断出来：

```java
(arg1, arg2) -> { return arg1 + arg2; }
```

如果你只有一个参数，你可以省略括号：

```java
arg1 -> { return arg1; }
```

然而，如果你的 lambda 没有参数，那么你必须包含括号：

```java
() -> { return 5; }
```

然后是函数主体；如果你在 lambda 逻辑中有许多行代码，你必须使用花括号来包围主体：

```java
(arg1, arg2) -> { 
    int sum = arg1 + arg2;
    return sum;
}
```

然而，如果你只有一行代码，你可以省略花括号，并立即返回值：

```java
(arg1, arg2) -> return arg1 + arg2;
```

最后，如果你只有一行代码，也可以省略 `return` 关键字：

```java
(arg1, arg2) -> arg1 + arg2;
```

如果我们要在 Java 中编写 lambda 算法的恒等函数，假设我们有一个名为 `Identity` 的函数式接口，它看起来可能像这样：

```java
Identity identity = x -> x;
```

一个常用的接口是 `Comparator` 接口，它用于几乎任何需要排序的对象，特别是在某些形式的集合中。

## 练习 4：列出备用轮胎

一支赛车队联系了你，希望你能组织他们的备用轮胎库存，因为它们现在一团糟。他们要求你编写一个应用程序，以按大小顺序显示可用的轮胎列表，从最大的轮胎开始。

要做到这一点，你需要构建一个实现 `Comparator` 函数式接口的 lambda 函数。为了参考，这是 `Comparator` 接口的基本视图，不包括默认和静态函数：

```java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
}
```

1.  在 IntelliJ 的 `Project` 面板中，右键单击名为 `src` 的文件夹。

1.  在菜单中选择 `New` -> `Java Class`，然后输入 `Exercise4`。

1.  在你的新类中定义 `main` 方法：

    ```java
    package com.packt.java.chapter13;
    public class Exercise4 {
        public static void main(String[] args) {
        }
    }
    ```

1.  创建一个名为 `Tire` 的新内部类。它应该有一个名为 `size` 的变量，表示轮胎的直径（单位为英寸）。确保将类和大小声明为 `final` 以符合 FP 指南：

    ```java
    package com.packt.java.chapter13;
    public class Exercise4 {
        public static void main(String[] args) {
        }
        public static final class Tire {
            private final int size;
        }
    }
    ```

1.  创建 `Tire` 构造函数，接受一个参数——`size`，并将其传递给成员变量。此外，重写 `toString()` 方法以打印轮胎的大小：

    ```java
        public static void main(String[] args) {
        }
        public static final class Tire {
            private final int size;
            public Tire(int size) {
                this.size = size;
            } 
            @Override
            public String toString() {
                return String.valueOf(size);
            }
        }
    }
    ```

1.  在你的 `main` 方法中创建一个需要排序的轮胎列表：

    ```java
        public static void main(String[] args) {
            List<Tire> tires = List.of(
                new Tire(17),
                new Tire(16),
                new Tire(18),
                new Tire(14),
                new Tire(15),
                new Tire(16));
        }
    ```

1.  创建实际的 lambda 表达式，使用 `Comparator` 函数式接口，这将用于对不可变轮胎列表进行排序。它应该接受两个参数，并返回大小差异。记住，lambda 表达式推断了很多结构；在这个简单的例子中，你不需要指定类型或返回关键字。lambda 表达式是一等公民，因此可以将其存储在变量中以供以后使用：

    ```java
        public static void main(String[] args) {
            List<Tire> tires = List.of(
                new Tire(17),
                new Tire(16),
                new Tire(18),
                new Tire(14),
                new Tire(15),
                new Tire(16));
            Comparator<Tire> sorter = (t1, t2) -> t2.size - t1.size;
        }
    ```

    注意

    当然，你也可以将 lambda 表达式作为匿名实例应用——这样，你可以节省几行代码，同时保持代码的可读性。

1.  在 `sort` 方法中应用 lambda 表达式。`List.sort()` 方法会修改列表的内容，因此排序之前需要复制你的不可变轮胎列表：

    ```java
        public static void main(String[] args) {
            List<Tire> tires = List.of(
                new Tire(17),
                new Tire(16),
                new Tire(18),
                new Tire(14),
                new Tire(15),
                new Tire(16));
            Comparator<Tire> sorter = (t1, t2) -> t2.size - t1.size;
            List<Tire> sorted = new ArrayList<>(tires);
            sorted.sort(sorter);
        }
    ```

1.  最后，打印结果：

    ```java
        public static void main(String[] args) {
            List<Tire> tires = List.of(
                new Tire(17),
                new Tire(16),
                new Tire(18),
                new Tire(14),
                new Tire(15),
                new Tire(16));
            Comparator<Tire> sorter = (t1, t2) -> t2.size - t1.size;
            List<Tire> sorted = new ArrayList<>(tires);
            sorted.sort(sorter);
            System.out.println(sorted);
        }
    ```

1.  要使这个程序具有函数式特性，可以将排序智能移动到一个纯函数，该函数接受一个列表作为参数，然后在列表的副本上进行排序，并返回不可变的排序列表。这样，你将避免在主程序中保留可变列表的引用：

    ```java
    https://packt.live/35OxQiJ.
    ```

你刚刚创建了一个基于现有 `Functional` 接口的第一个 lambda 表达式，然后使用它对轮胎列表进行排序。自从 Java 8 以来，有很多函数式接口可用，你可能已经使用过其中大部分；我们将在本书的后面更详细地探讨这一点。

# 摘要

不同线程对你的数据执行操作的顺序不应该很重要，你应该能够轻松地添加不会影响应用程序旧部分的功能。遵循这些 FP 概念可以使你构建的代码容易在多线程应用程序中使用，以及构建可以非常容易地测试问题的回归错误。这通常也使你的代码更加易于阅读。

使用你在本章中学到的 FP（函数式编程）的核心概念——纯函数和不可变性——在某些情况下可能会导致性能问题，特别是在修改大型数据集时。我们将在后面的章节中探讨解决这些问题的方法。

由于 Java 是为面向对象的方法设计的，因此一开始进入 FP（函数式编程）可能会有些令人畏惧，但如果你只在代码的某些部分“采用函数式”，那么从 OOP（面向对象编程）的过渡可能会变得更容易。

在下一章中，我们将关注如何在不使用循环的情况下导航更大的数据集并重复代码。
