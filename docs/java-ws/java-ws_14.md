# 第十四章：14. 递归

概述

在本章中，我们将看到如何使用递归帮助你编写有效的代码。章节从一项初始练习开始，该练习说明了你可以用递归犯的最简单的错误之一：忘记编写终止条件。因此，第一步是学习如何在 Java 栈溢出时挽救你的程序。从那里，你将学习编写递归方法来处理数学公式和其他重复处理需求。最后，通过这些技术（以及本章进一步定义的技术），你将练习使用 **文档对象模型**（**DOM**）API 创建和处理 XML 文件。

# 简介

递归是一个方法一次又一次地调用自己。当谨慎使用时，递归可以是一种有用的编程技术；但关键是正确使用它。

一个重要点是，递归只是一种编程技术。如果你愿意，你通常可以通过编写某种形式的迭代循环来避免它。然而，如果你需要解决的问题确实是递归的，那么迭代方法可能比相对简单且更优雅的递归代码复杂得多。

本章深入探讨了这种实用的编程技巧。

# 深入递归

递归对于许多数学问题很有用，例如在处理细胞自动机、谢宾斯基三角形和分形时。在计算机图形学中，递归可以用来帮助生成看起来逼真的山脉、植物和其他自然现象。经典的汉诺塔问题非常适合使用递归。

在 Java 应用程序中，你将经常在遍历树形数据结构时使用递归，包括 XML 和 HTML 文档。

注意

你可以参考 [`packt.live/2JaIre8`](https://packt.live/2JaIre8) 获取关于汉诺塔问题的更多信息。

递归的一个简单例子如下：

```java
public int add(int num) {
    return add(num + 1);
}
```

在这个例子中，每次对 `add()` 方法的调用都会用比当前调用使用的数字大一的数字调用自己。

注意

你总是需要一个终止条件来停止递归。这个例子没有。

## 练习 1：使用递归溢出栈

这个例子演示了当你没有为递归方法提供停止方式时会发生什么。你的程序会遇到一些问题。按照以下步骤进行练习：

1.  在 IntelliJ 的“文件”菜单中选择“新建”，然后选择“项目...”。

1.  选择项目的类型为`Gradle`。点击“下一步”。

1.  对于“组 ID”，输入`com.packtpub.recursion`。

1.  对于“工件 ID”，输入`chapter14`。

1.  对于“版本”，输入`1.0`。

1.  在下一页接受默认设置。点击“下一步”。

1.  将项目名称保留为`chapter14`。

1.  点击“完成”。

1.  在 IntelliJ 文本编辑器中打开 `build.gradle`。

1.  将 `sourceCompatibility` 设置为 `12`，如图所示：

    ```java
    sourceCompatibility = 12
    ```

1.  在 `src/main/java` 文件夹中创建一个新的 Java 包。

1.  将包名输入为`com.packtpub.recursion`。

1.  在`Project`窗格中右键单击此包，创建一个名为`RunForever`的新 Java 类。

1.  按如下方式进入递归方法：

    ```java
    public int add(int num) {
        return add(num + 1);
    }
    ```

1.  按如下方式进入`main()`方法：

    ```java
    public class RunForever {
        public static void main(String[] args) {
            RunForever runForever = new RunForever();
            System.out.println(runForever.add(1));
        }
    }
    ```

1.  运行这个程序；你会看到它因异常而失败：

    ```java
    Exception in thread "main" java.lang.StackOverflowError
    at com.packtpub.recursion.RunForever.add(RunForever.java:11)
    ```

    完整的代码如下所示：

    ```java
    package com.packtpub.recursion;
    public class RunForever {
        public int add(int num) {
            return add(num + 1);
        }
        public static void main(String[] args) {
            RunForever runForever = new RunForever();
            System.out.println(runForever.add(1));
        }
    }
    ```

    我们可以通过提供终止条件来停止递归，如下面的`RunAndStop.java`文件所示：

    ```java
    package com.packtpub.recursion;
    public class RunAndStop {
        public int add(int num) {
            if (num < 100) {
                return add(num + 1);
            }
            return num;
        }
        public static void main(String[] args) {
            RunAndStop runAndStop = new RunAndStop();
            System.out.println( runAndStop.add(1) );
        }
    }
    ```

    当你运行这个程序时，你会看到以下输出：

    ```java
    100
    ```

## 尝试尾递归

尾递归是指递归方法的最后一个可执行语句是对自身的调用。尾递归很重要，因为 Java 编译器可以——但在此刻还没有——跳回方法的开头。这有助于编译器不需要为方法调用存储栈帧，使其更高效，并在调用栈上使用更少的内存。

## 练习 2：使用递归计算阶乘

阶乘是演示递归工作原理的绝佳例子。

你可以通过将数字与所有小于它的正数相乘来计算一个整数的阶乘。例如，4 的阶乘，也写作 4!，计算为 4 * 3 * 2 * 1。执行以下步骤来完成练习：

1.  右键单击`com.packtpub.recursion`包名。

1.  创建一个名为`Factorial`的新 Java 类。

1.  进入递归方法：

    ```java
    public static int factorial(int number) {
        if (number == 1) {
            return 1;
        } else {
            return number * factorial(number - 1);
        }
    }
    ```

    由于阶乘是一个数乘以所有小于它的正数，因此在每次调用`factorial()`方法时，它返回该数乘以比该数小一的阶乘。如果传入的数是 1，它就简单地返回数字 1。

1.  进入`main()`方法，它启动阶乘计算：

    ```java
    public static void main(String[] args) {
        System.out.println( factorial(6) );
    }
    ```

    此代码将计算 6 的阶乘，也称为 6 阶乘或 6!。

1.  当你运行这个程序时，你会看到以下输出：

    ```java
    720
    ```

    完整的代码如下所示：

    ```java
    package com.packtpub.recursion;
    public class Factorial {
        public static int factorial(int number) {
            if (number == 1) {
                return 1;
            } else {
                return number * factorial(number - 1);
            }
        }
        public static void main(String[] args) {
            System.out.println( factorial(6) );
        }
    }
    ```

阶乘以及许多其他数学概念与递归很好地结合。另一个适合这种编程技术的常见任务是处理层次文档，例如 XML 或 HTML。

## 处理 XML 文档

XML 文档有节点。每个节点可能有子节点；例如，考虑以下：

```java
cities.xml
1  <?xml version="1.0" encoding="UTF-8" standalone="no"?>
2  <cities>
3    <city>
4      <name>London</name>
5      <country>United Kingdom</country>
6      <summertime-high-temp>20.4 C</summertime-high-temp>
7      <in-year-2100>
8          <with-moderate-emission-cuts>
9            <name>Paris</name>
10          <country>France</country>
11          <summertime-high-temp>22.7 C</summertime-high-temp>
12        </with-moderate-emission-cuts>
https://packt.live/2N4X4Rl
```

在这个 XML 片段中，`<cities>`元素有一个子元素`<city>`。`<city>`子元素反过来又有四个子元素。

注意

这些数据来自[`packt.live/33IrCyR`](https://packt.live/31yBoSL)，并在*第六章*，*库、包和模块*中的练习中使用。

现在，考虑你将如何编写代码来处理上述 XML 数据。Java 提供了解析 XML 文件的类。唯一的问题是解析成 Java 对象后如何处理 XML 文档。这就是递归可以发挥作用的地方。

你可以编写代码来处理每个 `<city>` 元素，例如 `London` 的数据。在该元素中，代码将提取子元素中的数据，例如城市名称、国家名称和夏令时高温。

注意如何显示两个额外的城市，`Paris` 和 `Milan`。这些数据可以以与 `London` 数据类似的方式处理。一旦你看到相似性，你可能会发现递归非常有用。

## 练习 3：创建一个 XML 文件

为了演示如何解析并递归遍历 XML 文档，我们需要一些 XML 数据：

1.  右键单击 `src/main/resources` 并选择 `New`，然后选择 `File`。

1.  将文件名输入为 `cities.xml`。

1.  将以下 XML 数据输入到文件中：

```java
cities.xml
2  <cities>
3    <city>
4      <name>London</name>
5      <country>United Kingdom</country>
6      <summertime-high-temp>20.4 C</summertime-high-temp>
7      <in-year-2100>
8          <with-moderate-emission-cuts>
9            <name>Paris</name>
10          <country>France</country>
11          <summertime-high-temp>22.7 C</summertime-high-temp>
12        </with-moderate-emission-cuts>
https://packt.live/2N4X4Rl
```

Java 包含多个用于处理 XML 数据的 API。使用 **简单 XML API**（**SAX**），你可以一次处理一个 XML 文档的事件。事件包括开始一个元素、从元素内部获取一些文本以及结束一个元素。

使用 **文档对象模型**（**DOM**），API 读取 XML 文档。从这一点开始，你的代码可以遍历 DOM 元素的树。最适合递归处理的 API 是 DOM API。

注意

你可以在 [`packt.live/31yBoSL`](https://packt.live/31yBoSL) 和 [`packt.live/2BvD2tJ`](https://packt.live/2BvD2tJ) 找到有关 Java XML API 的更多信息。

## 介绍 DOM XML API

使用 DOM API，你可以使用 `DocumentBuilder` 类将 XML 文件解析为内存中的对象树。这些对象都实现了 `org.w3c.Node` 接口。节点接口允许你从每个 XML 元素中提取数据，然后检索节点下的所有子节点。

在我们的例子中，如 `<city>` 这样的常规 XML 元素实现了 `Element` 接口，该接口扩展了 `Node` 接口。此外，文本项实现了 `Text` 接口。整个文档由 `Document` 接口表示。

整个 DOM 是分层的。例如，考虑以下内容：

```java
<city>
    <name>London</name>
</city>
```

在这个简短的片段中，`<city>` 是一个元素，并且有一个子元素 `<name>`。`London` 文本是 `<name>` 元素的子元素。`London` 文本将保存在一个实现 `Text` 接口的对象中。

注意

DOM API 需要将整个 XML 文档加载到节点层次结构中。对于大型 XML 文档，DOM API 可能不合适，因为你可能会耗尽内存。

使用 DOM API 时，第一步是加载一个 XML 文件并将其解析为对象层次结构。

要做到这一点，你需要一个 `DocumentBuilder` 类：

```java
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = factory.newDocumentBuilder();
```

一旦你有了 `DocumentBuilder` 类，你可以解析 XML 文件以获取 `Document` 接口：

```java
File xmlFile = new File("src/main/resources/cities.xml");
Document document = builder.parse(xmlFile);
```

由于 `Document` 是 `Node`，你可以开始处理所有子节点。通常，你从 `Document` 接口的第一子节点开始（在我们的早期示例中是 `<cities>`）：

```java
Node node = document.getFirstChild();
NodeList children = node.getChildNodes();
for (int i = 0; i < children.getLength(); i++) {
    Node child = children.item(i);
}
```

`getFirstChild()`的调用返回`document`的第一个子节点，即顶级 XML 元素。然后你可以调用`getChildNodes()`来检索所有直接子元素。不幸的是，返回的`NodeList`对象既不是`List`也不是`Collection`接口，这使得遍历子节点变得更加困难。

然后，你可以使用递归获取任何给定节点的子节点，以及这些子节点的子节点，依此类推。例如，看看以下内容：

```java
if (node.hasChildNodes()) {
    indentation += 2;
    NodeList children = node.getChildNodes();
    for (int i = 0; i < children.getLength(); i++) {
        Node child = children.item(i);
        if (child.getNodeType() == Node.TEXT_NODE) {
            printText(child.getTextContent() );
        } else {            
            traverseNode(child, indentation);
        }
    }
}
```

在这个例子中，我们首先检查给定的节点是否有子节点。如果没有，我们就没有什么可做的。如果有子节点，我们将使用之前展示的相同技术来获取每个子节点。

一旦我们有一个节点，代码就会使用`getNodeType()`方法检查节点是否是`Text`节点。如果是`Text`节点，我们将打印出文本。如果不是，我们将对子节点进行递归调用。这将检索子节点的所有子节点。

## 练习 4：遍历 XML 文档

在这个练习中，我们将编写代码来遍历从我们在*练习 3*中创建的`cities.xml`文件解析出的节点对象树。该代码将打印出 XML 元素作为文本。按照以下步骤完成练习：

1.  编辑`build.gradle`文件。为`Apache Commons Lang`库添加新的依赖项：

    ```java
    dependencies {
        testCompile group: 'junit', name: 'junit', version: '4.12'
        // https://mvnrepository.com/artifact/org.apache.commons/commons-lang3
        implementation group: 'org.apache.commons', name: 'commons-lang3', version: '3.8.1'
    }
    ```

    此库有几个有用的实用方法，我们将在生成输出时使用。

1.  右键单击`com.packtpub.recursion`包名。

1.  创建一个名为`XmlTraverser`的新 Java 类。

1.  输入以下方法以将 XML 文件加载到 DOM 树中：

    ```java
    XmlTraverser.java
    17 public Document loadXml() {
    18   Document document = null;
    19 
    20   DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
    21   try {
    22     DocumentBuilder builder = factory.newDocumentBuilder();
    23 
    24     File xmlFile = new File("src/main/resources/cities.xml");
    25     document = builder.parse(xmlFile);
    26 
    27   } 
    https://packt.live/33MDhN2
    ```

    注意这段代码如何捕获从读取文件和解析 XML 内容中可能出现的所有异常。

1.  接下来，输入一个方法来打印`Text`节点的内容：

    ```java
    public void printText(String text) {
        if (StringUtils.isNotBlank(text)) {
            System.out.print(text);
        }
    }
    ```

    此方法使用 Apache `StringUtils`类来检查文本是否为空。你会发现 DOM API 填充了很多空的`Text`节点。

1.  为了帮助表示 XML 文档的层次结构，输入一个缩进实用方法：

    ```java
    public void indent(int indentation) {
        System.out.print( StringUtils.leftPad("", indentation));
    }
    ```

    再次，我们使用`StringUtils`类来完成用给定数量的空格填充空字符串的繁琐工作。

1.  接下来，我们创建主递归方法：

    ```java
        public void traverseNode(Node node, int indentation) {
            indent(indentation);
            System.out.print(node.getNodeName() + " ");
            if (node.hasChildNodes()) {
                indentation += 2;
                NodeList children = node.getChildNodes();
                for (int i = 0; i < children.getLength(); i++) {
                    Node child = children.item(i);
                    if (child.getNodeType() == Node.TEXT_NODE) {
                        printText( child.getTextContent() );
                    } else {
                        System.out.println();       // previous line
                        traverseNode(child, indentation);
                    }
                }
            }
        }
    ```

1.  此方法打印出输入节点的名称（这将是一个城市、国家或类似的东西）。然后检查子节点。如果子节点是`Text`节点，则打印出文本。否则，此方法将递归调用自身以处理子节点的所有子节点。

1.  要开始，创建一个简短的方法从 XML 文档的第一个子节点开始递归调用：

    ```java
    public void traverseDocument(Document document) {
        traverseNode(document.getFirstChild(), 0);
    }
    ```

1.  接下来，我们需要一个`main()`方法来加载 XML 文件并遍历文档：

    ```java
    public static void main(String[] args) {
        XmlTraverser traverser = new XmlTraverser();
        Document document = traverser.loadXml();
        // Traverse XML document.
        traverser.traverseDocument(document);
    }
    ```

1.  当你运行此程序时，你将看到以下输出：

    ```java
    cities 
      city 
        name London
        country United Kingdom
        summertime-high-temp 20.4 C
        in-year-2100 
          with-moderate-emission-cuts 
            name Paris
            country France
            summertime-high-temp 22.7 C
          with-no-emission-cuts 
            name Milan
            country Italy
            summertime-high-temp 25.2 C
    ```

    注意

    上述输出被截断。此练习的完整源代码可以在以下位置找到：[`packt.live/33VDygZ`](https://packt.live/33VDygZ)。

## 活动 1：计算斐波那契数列

斐波那契序列是一系列数字，其中每个数字都是前两个数字的和。编写一个递归方法来生成斐波那契序列的前 15 个数字。请注意，斐波那契值对于 0 是 0，对于 1 是 1。

斐波那契序列为 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55，等等。

因此，你可以使用以下内容作为指南：

```java
fibonacci(4) = 
fibonacci(3) + fibonacci(2) =
{fibonacci(2) + fibonacci(1)} + {fibonacci(1) + fibonacci(0)} =
{fibonacci(1) + fibonacci(0) + fibonacci(1) + fibonacci(0)} + {fibonacci(1) + fibonacci(0)} =
1 + 0 + 1 + 0 + 1 + 0 = 3
```

我们将使用递归方法来计算给定输入的斐波那契值，然后创建一个循环来显示序列。为此，请执行以下步骤：

1.  创建`fibonacci`方法。

1.  检查传递给`fibonacci`方法的值是否为 0，如果是，则返回 0。

1.  还要检查传递给`fibonacci`方法的值是否为 1，如果是，则返回 1。

1.  否则，加上前两个数的斐波那契值。

1.  在主方法中，创建一个从 0 到 15 的 for 循环并调用`fibonaci`方法。

当你运行你的程序时，你应该看到以下类似的输出：

```java
0
1
1
2
3
5
8
13
21
34
55
89
144
233
377
```

注意

本活动的解决方案可以在第 562 页找到。

# 摘要

递归是一种方便的编程技术，用于解决一些复杂问题。你通常会在数学公式中找到递归，以及在遍历二叉树或 XML 文档等分层数据结构时。使用递归，Java 方法或类会调用自身。但不要忘记编写终止条件，否则你会发现你的应用程序在 Java 调用栈上很快就会耗尽内存。

在下一章中，你将学习使用 Java 进行谓词和函数式编程。
