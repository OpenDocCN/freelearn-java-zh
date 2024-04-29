# 第七章：*第七章*

# Java 集合框架和泛型

## 学习目标

通过本课程结束时，您将能够：

+   使用集合处理数据

+   以不同的方式比较对象

+   对对象集合进行排序

+   使用集合构建高效的算法

+   为每种用例使用最合适的集合

## 介绍

在之前的课程中，您学习了如何将对象组合在一起形成数组，以帮助您批量处理数据。数组非常有用，但它们具有静态长度的事实使得在加载未知数量的数据时很难处理。此外，访问数组中的对象需要您知道数组的索引，否则需要遍历整个数组才能找到对象。您还简要了解了 ArrayList，它的行为类似于可以动态改变大小以支持更高级用例的数组。

在本课程中，您将学习 ArrayList 的实际工作原理。您还将了解 Java 集合框架，其中包括一些更高级的数据结构，用于一些更高级的用例。作为这个旅程的一部分，您还将学习如何在许多数据结构上进行迭代，以许多不同的方式比较对象，并以高效的方式对集合进行排序。

您还将了解泛型，这是一种强大的方式，可以让编译器帮助您使用集合和其他特殊类。

## 从文件中读取数据

在我们开始之前，让我们先了解一些我们将在本课程后面部分使用的基础知识。

### 二进制与文本文件

您的计算机中有许多类型的文件：可执行文件、配置文件、数据文件等。文件可以分为两个基本组：二进制和文本。

当人类与文件的交互只会间接发生时，例如执行应用程序（可执行文件）或在 Excel 中加载的电子表格文件时，使用二进制文件。如果您尝试查看这些文件的内部，您将看到一堆无法阅读的字符。这种类型的文件非常有用，因为它们可以被压缩以占用更少的空间，并且可以被结构化，以便计算机可以快速读取它们。

另一方面，文本文件包含可读字符。如果用文本编辑器打开它们，你可以看到里面的内容。并非所有文本文件都是供人类阅读的，有些格式几乎不可能理解。但大多数文本文件都可以被人类读取和轻松编辑。

### CSV 文件

逗号分隔值（CSV）文件是一种非常常见的文本文件类型，用于在系统之间传输数据。CSV 非常有用，因为它们易于生成和阅读。这种文件的结构非常简单：

+   每行一个记录。

+   第一行是标题。

+   每个记录都是一个长字符串，其中的值使用逗号分隔（值也可以用其他分隔符分隔）。

以下是从我们将要使用的示例数据中提取出的文件的一部分。

```java
id,name,email
10,Bill Gates,william.gates@microsoft.com
30,Jeff Bezos,jeff.bezos@amazon.com
20,Marc Benioff,marc.benioff@salesforce.com
```

### 在 Java 中读取文件

Java 有两个基本的类集，用于读取文件：`Stream`，用于读取二进制文件，和`Reader`，用于读取文本文件。`io`包设计中最有趣的部分是`Stream`和`Reader`可以组合在一起逐步添加功能。这种能力被称为管道，因为它类似于将多个管道连接在一起的过程。

我们将使用一个简单的例子来解释这些，还有`FileReader`和`BufferedReader`的帮助。

`FileReader`逐个读取字符。`BufferedReader`可以缓冲这些字符以一次读取一行。这对我们在读取 CSV 时很简单，因为我们可以创建一个`FileReader`实例，然后用`BufferedReader`包装它，然后从 CSV 文件中逐行读取：

![图 7.1：从 CSV 文件中读取的过程的示意图](img/C09581_Figure_07_01.jpg)

###### 图 7.1：从 CSV 文件中读取的过程的示意图

### 练习 22：读取 CSV 文件

在这个练习中，您将使用`FileReader`和`BufferedReader`从 CSV 文件中读取行，拆分它们，并像记录一样处理它们：

1.  创建一个名为`ReadCSVFile.java`的文件，并添加一个同名的类，并向其中添加一个`main`方法：

```java
public class ReadCSVFile {
  public static void main(String [] args) throws IOException {
```

1.  首先，您需要添加一个字符串变量，该变量将从命令行参数中获取要加载的文件的名称：

```java
String fileName = args[0];  
```

1.  然后，您创建一个新的`FileReader`并将其放入`BufferedReader`中，使用 try-with-resource，如下面的代码所示：

```java
FileReader fileReader = new FileReader(fileName);
try (BufferedReader reader = new BufferedReader(fileReader)) {
```

1.  现在您已经打开了一个文件进行读取，您可以逐行读取它。`BufferedReader`将一直给您新的行，直到文件结束。当文件结束时，它将返回`null`。因此，我们可以声明一个变量行，并在`while`条件中设置它。然后，我们需要立即检查它是否为 null。我们还需要一个变量来计算我们从文件中读取的行数：

```java
String line;
int lineCounter = -1;
while ( (line = reader.readLine()) != null ) {
```

1.  在循环内，您增加了行计数并忽略了第零行，即标题。这就是为什么我们将`lineCounter`初始化为`-1`而不是零的原因：

```java
lineCounter++;
// Ignore the header
if (lineCounter == 0) {
  continue;
}
```

1.  最后，您使用`String`类的`split`方法拆分行。该方法接收一个分隔符，在我们的情况下是逗号：

```java
String [] split = line.split(",");
System.out.printf("%d - %s\n", lineCounter, split[1]);
```

#### 注意

您可以看到`FileReader`是如何传递到`BufferedReader`中，然后再也没有访问的。这是因为我们只想要行，而不关心将字符转换为行的中间过程。

恭喜！您编写了一个可以读取和解析 CSV 的应用程序。随意深入研究这段代码，并了解当您更改初始行计数值时会发生什么。

输出如下：

```java
1 - Bill Gates
2 - Jeff Bezos
3 - Marc Benioff
4 - Bill Gates
5 - Jeff Bezos
6 - Sundar Pichai
7 - Jeff Bezos
8 - Larry Ellison
9 - Marc Benioff
10 - Larry Ellison
11 - Jeff Bezos
12 - Bill Gates
13 - Sundar Pichai
14 - Jeff Bezos
15 - Sundar Pichai
16 - Marc Benioff
17 - Larry Ellison
18 - Marc Benioff
19 - Jeff Bezos
20 - Marc Benioff
21 - Bill Gates
22 - Sundar Pichai
23 - Larry Ellison
24 - Bill Gates
25 - Larry Ellison
26 - Jeff Bezos
27 - Sundar Pichai
```

### 构建 CSV 读取器

现在您知道如何从 CSV 中读取数据，我们可以开始考虑将该逻辑抽象成自己的管道。就像`BufferedReader`允许您逐行读取文本文件一样，CSV 读取器允许您逐条记录读取 CSV 文件。它建立在`BufferedReader`功能之上，并添加了使用逗号作为分隔符拆分行的逻辑。以下图表显示了我们的新管道将如何使用 CSV 读取器：

![图 7.2：CSVReader 可以添加到链中以逐条读取记录](img/C09581_Figure_07_02.jpg)

###### 图 7.2：CSVReader 可以添加到链中以逐条读取记录

### 练习 23：构建 CSV 读取器

在这个练习中，我们将遵循管道模式，并构建一个简单的`CSVReader`，我们将在本课程的其余部分中使用它：

1.  创建一个名为`CSVReader.java`的新文件，并在编辑器中打开它。

1.  在此文件中，创建一个名为`CSVReader`的公共类，并实现`Closeable`接口：

```java
public class CSVReader implements Closeable {
```

1.  添加两个字段，一个字段用于将`BufferedReader`存储为`final`，我们将从中读取，另一个字段用于存储行计数：

```java
private final BufferedReader reader;
private int lineCount = 0;
```

1.  创建一个构造函数，接收`BufferedReader`并将其设置为字段。此构造函数还将读取并丢弃传入读取器的第一行，因为那是标题，我们在本课程中不关心它们：

```java
public CSVReader(BufferedReader reader) throws IOException {
  this.reader = reader;
  // Ignores the header
  reader.readLine();
}
```

1.  通过调用基础读取器的`close`方法来实现`close`方法：

```java
public void close() throws IOException {
  this.reader.close();
}
```

1.  就像`BufferedReader`有一个`readLine`方法一样，我们的`CSVReader`类将有一个`readRecord`方法，该方法将从`BufferedReader`读取行，然后返回由逗号分隔的字符串。在这种方法中，我们将跟踪到目前为止已读取多少行。我们还需要检查读取器是否返回了一行，因为它可能返回 null，这意味着它已经完成了对文件的读取，并且没有更多的行可以给我们。如果是这种情况，我们将遵循相同的模式并返回 null：

```java
public String[] readRow() throws IOException {
  String line = reader.readLine();
  if (line == null) {
    return null;
  }
  lineCount++;
  return line.split(",");
}
```

#### 注意

在更复杂的实现中，我们可以存储标题以公开类的用户提供额外的功能，例如按标题名称获取值。我们还可以对行进行整理和验证，以确保没有额外的空格包裹值，并且它们包含预期数量的值（与标题计数相同）。

1.  使用 getter 公开`linecount`：

```java
public int getLineCount() {
  return lineCount;
}
```

1.  现在你的新`CSVReader`已经准备好使用了！创建一个名为`UseCSVReaderSample.java`的新文件，其中包含同名的类和一个`main`方法：

```java
public class UseCSVReaderSample {
  public static void main (String [] args) throws IOException {
```

1.  按照之前使用的模式来读取 CSV 中的行，现在你可以使用你的`CSVReader`类来从 CSV 文件中读取，将以下内容添加到你的`main`方法中：

```java
String fileName = args[0];
FileReader fileReader = new FileReader(fileName);
BufferedReader reader = new BufferedReader(fileReader);
try (CSVReader csvReader = new CSVReader(reader)) {
  String[] row;
  while ( (row = csvReader.readRow()) != null ) {
    System.out.printf("%d - %s\n", csvReader.getLineCount(), row[1]);
  }
}
```

#### 注意

从前面的片段中，你可以看到你的代码现在简单得多。它专注于提供业务逻辑（打印带有行数的第二个值），并不关心读取 CSV。这是一个很好的实际例子，说明了如何创建你的读取器来抽象出关于处理来自文件的数据的逻辑。

1.  为了使代码编译通过，你需要从`java.io`包中添加导入：

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
```

输出如下：

```java
1 - Bill Gates
2 - Jeff Bezos
3 - Marc Benioff
4 - Bill Gates
5 - Jeff Bezos
6 - Sundar Pichai
7 - Jeff Bezos
8 - Larry Ellison
9 - Marc Benioff
10 - Larry Ellison
11 - Jeff Bezos
12 - Bill Gates
13 - Sundar Pichai
14 - Jeff Bezos
15 - Sundar Pichai
16 - Marc Benioff
17 - Larry Ellison
18 - Marc Benioff
19 - Jeff Bezos
20 - Marc Benioff
21 - Bill Gates
22 - Sundar Pichai
23 - Larry Ellison
24 - Bill Gates
25 - Larry Ellison
26 - Jeff Bezos
27 - Sundar Pichai
```

### 数组

正如你已经从之前的课程中学到的，数组非常强大，但它们的静态特性使事情变得困难。假设你有一段代码，从某个数据库或 CSV 文件中加载用户。直到完成加载所有数据之前，从数据库或文件中获取的数据量是未知的。如果你使用的是数组，你将不得不在每次读取记录时调整数组的大小。这将是非常昂贵的，因为数组无法调整大小；它们需要一遍又一遍地复制。

以下是一些代码，用于说明如何调整数组的大小：

```java
// Increase array size by one
// Create new array
User[] newUsers = new User[users.length + 1];
// Copy data over
System.arraycopy(users, 0, newUsers, 0, users.length);
// Switch
users = newUsers;
```

为了更有效，你可以初始化数组的容量，并在完成读取所有记录后修剪数组，以确保它不包含任何额外的空行。你还需要确保数组在添加新记录时有足够的容量。如果没有，你将不得不创建一个具有足够空间的新数组，并复制数据。

### 练习 24：从 CSV 文件中读取用户到数组中

在这个练习中，你将学习如何使用数组来存储来自数据源的无限数量的数据。在我们的例子中，我们将使用在前几节中一直使用的相同的用户 CSV：

1.  创建一个名为`User.java`的文件，并添加一个同名的类。这个类将有三个字段：`id`、`name`和`email`。它还将有一个可以用所有三个值初始化的构造函数。我们将使用这个类来表示一个`User`：

```java
public class User {
  public int id;
  public String name;
  public String email;
  public User(int id, String name, String email) {
    this.id = id;
    this.name = name;
    this.email = email;
  }
}
```

1.  在`User`类的开头，添加一个`static`方法，该方法将从作为字符串数组传递的值创建一个用户。当从 CSV 中读取的值创建一个`User`时，这将非常有用：

```java
public static User fromValues(String [] values) {
  int id = Integer.parseInt(values[0]);
  String name = values[1];
  String email = values[2];
  return new User(id, name, email);
}
```

1.  创建另一个名为`IncreaseOnEachRead.java`的文件，并添加一个同名的类和一个`main`方法，该方法将把命令行的第一个参数传递给另一个名为`loadUsers`的方法。然后，打印加载的用户数量，如下所示：

```java
public class IncreaseOnEachRead {
  public static final void main (String [] args) throws Exception {
    User[] users = loadUsers(args[0]);
    System.out.println(users.length);
  }
}
```

1.  在同一个文件中，添加另一个名为`loadUsers`的方法，它将返回一个用户数组，并接收一个名为`fileToRead`的字符串，它将是要读取的 CSV 文件的路径：

```java
public static User[] loadUsers(String fileToReadFrom) throws Exception {
```

1.  在这个方法中，首先创建一个空的用户数组，并在最后返回它：

```java
User[] users = new User[0];
return users;
```

1.  在这两行之间，添加逻辑来使用你的`CSVReader`逐条读取 CSV 记录。对于每条记录，增加数组的大小，并将新创建的`User`添加到数组的最后位置：

```java
BufferedReader lineReader = new BufferedReader(new FileReader(fileToReadFrom));
try (CSVReader reader = new CSVReader(lineReader)) {
  String [] row = null;
  while ( (row = reader.readRow()) != null) {
    // Increase array size by one
    // Create new array
    User[] newUsers = new User[users.length + 1];
    // Copy data over
    System.arraycopy(users, 0, newUsers, 0, users.length);
    // Swap
    users = newUsers;
    users[users.length - 1] = User.userFromRow(row);
  }
}
```

输出如下：

```java
27
```

现在你可以从 CSV 文件中读取，并拥有了从中加载的所有用户的引用。这实现了在每次读取记录时增加数组的方法。你将如何实现更有效的方法，即初始化数组的容量，并在需要时增加它，并在最后修剪它？

### 活动 27：使用具有初始容量的数组从 CSV 中读取用户

在这个活动中，你将从 CSV 中读取用户，类似于你在上一个练习中所做的，但不是在每次读取时增加数组，而是使用初始容量创建数组，并在需要时增加它。最后，你需要检查数组是否还有空余空间，并将其缩小，以返回一个确切大小与加载的用户数量相同的数组。

要完成此活动，您需要：

1.  用初始容量初始化数组。

1.  在循环中从命令行传入的路径读取 CSV，创建用户并将它们添加到数组中。

1.  跟踪加载的用户数量。

1.  在向数组添加用户之前，您需要检查数组的大小，并在必要时进行扩展。

1.  最后，根据需要缩小数组，以返回加载的确切用户数量。

#### 注意

此活动的解决方案可在第 345 页找到。

## Java 集合框架

在构建复杂的应用程序时，您需要以不同的方式操作对象的集合。最初，核心 Java 库仅限于三种选项：数组、向量和哈希表。它们都以自己的方式强大，但随着时间的推移，变得清楚这是不够的。人们开始构建自己的框架来处理更复杂的用例，如分组、排序和比较。

Java 集合框架被添加到 Java 标准版中，以减少编程工作量，并通过提供高效且易于使用的数据结构和算法来改进 Java 应用程序的性能和互操作性。这组接口和实现类旨在为 Java 开发人员提供一种简单的方式来构建可以共享和重用的 API。

### 向量

向量解决了数组是静态的问题。它们提供了一种动态和可扩展的存储许多对象的方式。它们随着添加新元素而增长，可以准备接收大量元素，并且很容易迭代元素。

为了处理内部数组而不必要地调整大小，向量使用一些容量进行初始化，并使用指针值跟踪最后一个元素添加的位置，这个指针值只是一个标记该位置的整数。默认情况下，初始容量为 10。当您添加的元素超过数组的容量时，内部数组将被复制到一个更大的数组中，留下更多的空间，以便您可以添加额外的元素。复制过程就像您在*练习 24*中手动处理数组时所做的那样：*从 CSV 文件中读取用户到数组*。以下是它的工作原理的插图：

![图 7.3：向量的插图](img/C09581_Figure_07_03.jpg)

###### 图 7.3：向量的插图

在 Java 集合框架之前，使用向量是在 Java 中获得动态数组的方法。然而，存在两个主要问题：

+   缺乏易于理解和扩展的定义接口

+   完全同步，这意味着它受到多线程代码的保护

在 Java 集合框架之后，向量被改装以符合新的接口，解决了第一个问题。

### 练习 25：从 CSV 文件中读取用户到向量

由于向量解决了根据需要增长和缩小的问题，在这个练习中，我们将重写以前的练习，但是不再处理数组的大小，而是委托给一个向量。我们还将开始构建一个`UsersLoader`类，在所有未来的练习中都会使用：

1.  创建一个名为`UsersLoader.java`的文件，并在其中添加一个同名的类：

```java
public class UsersLoader {
}
```

1.  您将使用这个类来添加共享方法，以便在未来的课程中从 CSV 文件中加载用户。您将首先编写的方法将从 CSV 中加载用户到向量中。添加一个公共静态方法，返回一个向量。在这个方法中，实例化`Vector`并在最后返回它：

```java
private static Vector loadUsersInVector(String pathToFile)
    throws IOException {
  Vector users = new Vector();
  return users;
}
```

1.  在创建`Vector`并返回它之间，从 CSV 中加载数据并将其添加到`Vector`中：

```java
BufferedReader lineReader = new BufferedReader(new FileReader(pathToFile));
try (CSVReader reader = new CSVReader(lineReader)) {
  String [] row = null;
  while ( (row = reader.readRow()) != null) {
    users.add(User.fromValues(row));
  }
}
```

1.  添加编译此文件所需的导入项：

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.util.Vector;
```

1.  创建一个名为`ReadUsersIntoVector.java`的文件，并在其中添加一个同名的类和一个`main`方法：

```java
public class ReadUsersIntoVector {
  public static void main (String [] args) throws IOException {
  }
}
```

1.  在`main`方法中，类似于我们在数组情况下所做的，调用从 CSV 加载用户到`Vector`的方法，然后打印`Vector`的大小。在这种情况下，使用我们在上一步中创建的`loadUsersInVector()`方法：

```java
Vector users = UserLoader.loadUsersInVector(args[0]);
System.out.println(users.size());
```

1.  将此文件的导入添加到编译：

```java
import java.io.IOException;
import java.util.Vector;
```

输出如下：

```java
27
```

恭喜您完成了又一个练习！这一次，您可以看到您的代码要简单得多，因为大部分加载 CSV、将其拆分为值、创建用户和调整数组大小的逻辑现在都被抽象化了。

### 活动 28：使用 Vector 读取真实数据集

在此活动中，您将下载一个包含来自美国人口普查的收入信息的 CSV，并对文件中的值进行一些计算。

要开始，请转到此页面：[`github.com/TrainingByPackt/Java-Fundamentals/tree/master/Lesson07/data`](https://github.com/TrainingByPackt/Java-Fundamentals/tree/master/Lesson07/data)。要下载 CSV，您可以单击**Adult_Data**。它将在浏览器中打开数据文件。下载文件并将其保存到计算机中的某个位置。扩展名无关紧要，但您需要记住文件名和路径。

您可以在网站上阅读有关数据格式的更多信息，或者只需将其作为文本文件打开。在处理此文件时要记住两件事：

+   文件末尾有一个额外的空行

+   此文件没有标题行

创建一个应用程序，将计算此文件中的最低工资、最高工资和平均工资。在读取所有行之后，您的应用程序应打印这些结果。为了实现这一点，您需要：

1.  使用您的`CSVReader`将文件中的所有工资加载到整数向量中。您可以修改您的`CSVReader`以支持没有标题的文件。

1.  迭代向量中的值，并跟踪三个值：最小值、最大值和总和。

1.  在最后打印结果。请记住，平均值只是向量的总和除以大小。

#### 注意

此活动的解决方案可以在第 347 页找到。

### 遍历集合

在处理数组时，您有两种迭代的方式：您可以使用带有索引的`for`循环：

```java
for (int i = 0; i < values.length; i++) {
  System.out.printf("%d - %s\n", i, values[i]);
}
```

您还可以使用`for-each`循环进行迭代，其中您无法访问元素的索引：

```java
for (String value : values) {
  System.out.println(value);
}
```

当您需要迭代向量时，您可以使用带有索引的循环，就像数组一样：

```java
for (int i = 0; i < values.size(); i++) {
  String value = (String) values.get(i);
  System.out.printf("%d - %s\n", i, value);
}
```

您还可以在`for-each`循环中使用`Vector`，就像数组一样：

```java
for (Object value : values) {
  System.out.println(value);
}
```

这是因为`Vector`实现了`Iterable`。 Iterable 是一个简单的接口，告诉编译器该实例可以在`for-each`循环中使用。实际上，您可以将您的`CSVReader`更改为实现 Iterable，然后在`for-each`循环中使用它，就像以下代码中一样：

```java
try (IterableCSVReader csvReader = new IterableCSVReader(reader)) {
  for (Object rowAsObject : csvReader) {
    User user = User.fromValues((String[]) rowAsObject);
    System.out.println(user.name);
  }
}
```

Iterable 是一个非常简单的接口；它只有一个方法需要实现：`iterator()`。该方法返回一个迭代器。迭代器是另一个简单的接口，只有两个方法需要实现：

+   `hasNext()`: 如果迭代器仍有要返回的元素，则返回`true`。

+   `next()`: 获取下一个记录并返回它。如果在调用此方法之前`hasNext()`返回`false`，它将抛出异常。

迭代器表示从集合中获取事物的一种简单方法。但它还有另一个在一些更高级的上下文中很重要的方法，`remove()`，它会删除刚刚从`next()`调用中获取的当前元素。

这个`remove`方法很重要，因为当您在集合上进行迭代时，您不能修改它。这意味着如果您编写一个`for-each`循环来从向量中读取元素，然后在此循环中调用`remove(Object)`来从中删除一个元素，将会抛出`ConcurrentModificationException`。因此，如果您想使用循环迭代集合，并且在此循环中需要从向量中删除一个元素，您将需要使用迭代器。

你一定在想，“为什么它要设计成这样？”因为 Java 是一种多线程语言。你不会在这本书中学习如何创建线程或使用它们，因为这是一个高级主题。但多线程的背后思想是，内存中的一块数据可以被两段代码同时访问。这是可能的，因为现代计算机具有多核能力。在处理多线程应用程序时，使用集合和数组时必须非常小心。以下是说明它发生的过程：

![图 7.4：ConcurrentModificationException 发生的说明](img/C09581_Figure_07_04.jpg)

###### 图 7.4：ConcurrentModificationException 发生的说明

`ConcurrentModificationException`比我们预期的更常见。以下是使用迭代器的示例 for 循环，避免了这个问题：

```java
for (Iterator it = values.iterator(); it.hasNext();) {
  String value = (String) it.next();
  if (value.equals("Value B")) {
    it.remove();
  }
}
```

### 活动 29：在用户向量上进行迭代

现在你有了一个从 CSV 文件中加载所有用户的方法，并且知道如何在向量上进行迭代，编写一个应用程序，打印文件中所有用户的姓名和电子邮件。要完成这个活动，你需要按照以下步骤进行：

1.  创建一个新的 Java 应用程序，从一个向量中加载来自 CSV 文件的数据。文件将从命令行指定。

1.  遍历向量中的用户，并打印一个字符串，其中包含他们的姓名和电子邮件的连接。

#### 注意

这个活动的解决方案可以在第 349 页找到。

### 哈希表

当处理需要按顺序处理的许多对象时，数组和向量非常有用。但是当你有一组需要通过键（例如某种标识）进行索引的对象时，它们就变得笨重了。

引入了哈希表。它们是一个非常古老的数据结构，是为了解决这个问题而创建的：快速识别给定值并在数组中找到它。为了解决这个问题，哈希表使用哈希函数来唯一标识对象。从哈希中，它们可以使用另一个函数（通常是除法的余数）将值存储在数组中。这使得将元素添加到表中的过程是确定性的，并且获取它非常快。以下是说明值如何存储在哈希表中的过程：

![图 7.5：哈希表存储和提取值的过程](img/C09581_Figure_07_05.jpg)

###### 图 7.5：哈希表存储和提取值的过程

哈希表使用数组来内部存储一个条目，代表一个键值对。当你将一对放入哈希表时，你提供键和值。键用于找到条目将被存储在数组中的位置。然后，创建并存储一个持有键和值的条目在指定的位置。

要获取值，你传入从中计算哈希的键，然后可以快速在数组中找到条目。

从这个过程中，你免费获得的一个有趣的特性是去重。因为使用相同的键添加值将生成相同的哈希，当你这样做时，它将覆盖之前存储在那里的任何内容。

就像向量一样，`Hashtable`类是在 Java 的集合框架之前添加的。它遭受了向量遭受的两个问题：缺乏定义的接口和完全同步。它还违反了 Java 的命名约定，没有遵循驼峰命名法来分隔单词。

与向量一样，在引入集合框架后，哈希表也经过了改造，以符合新的接口，使它们成为框架的无缝部分。

### 练习 26：编写一个通过电子邮件查找用户的应用程序

在这个练习中，你将编写一个应用程序，从指定的 CSV 文件中读取用户到哈希表中，使用他们的电子邮件作为键。然后从命令行接收一个电子邮件地址，并在哈希表中搜索它，打印它的信息或者友好的消息，如果找不到的话：

1.  在您的`UsersLoader.java`文件中，添加一个新方法，该方法将使用电子邮件将用户加载到 Hashtable 中。在开始时创建一个`Hashtable`，并在结束时返回它：

```java
public static Hashtable loadUsersInHashtableByEmail(String pathToFile) 
    throws IOException {
  Hashtable users = new Hashtable();
  return users;
}
```

1.  在创建`Hashtable`并返回它之间，使用`email`作为键从 CSV 中加载用户并将它们放入`Hashtable`中：

```java
BufferedReader lineReader = new BufferedReader(new FileReader(pathToFile));
try (CSVReader reader = new CSVReader(lineReader)) {
  String [] row = null;
  while ( (row = reader.readRow()) != null) {
    User user = User.fromValues(row);
    users.put(user.email, user);
  }
}
```

1.  导入`Hashtable`以便文件正确编译：

```java
import java.util.Hashtable;
```

1.  创建一个名为`FindUserHashtable.java`的文件，并添加一个同名的类，并添加一个`main`方法：

```java
public class FindUserHashtable {
  public static void main(String [] args) throws IOException {
  }
}
```

1.  在您的`main`方法中，使用我们在之前步骤中创建的方法将用户加载到`Hashtable`中，并打印找到的用户数量：

```java
Hashtable users = UsersLoader.loadUsersInHashtableByEmail(args[0]);
System.out.printf("Loaded %d unique users.\n", users.size());
```

1.  打印一些文本，通知用户您正在等待他们输入电子邮件地址：

```java
System.out.print("Type a user email: ");
```

1.  通过使用`Scanner`从用户那里读取输入：

```java
try (Scanner userInput = new Scanner(System.in)) {
  String email = userInput.nextLine();
```

1.  检查`Hashtable`中是否存在电子邮件地址。如果没有，打印友好的消息并退出应用程序：

```java
if (!users.containsKey(email)) {
  // User email not in file
  System.out.printf("Sorry, user with email %s not found.\n", email);
  return;
}
```

1.  如果找到，打印有关找到的用户的一些信息：

```java
User user = (User) users.get(email);
System.out.printf("User with email '%s' found!", email);
System.out.printf(" ID: %d, Name: %s", user.id, user.name);
```

1.  添加必要的导入：

```java
import java.io.IOException;
import java.util.Hashtable;
import java.util.Scanner;
```

这是第一种情况的输出：

```java
Loaded 5 unique users.
Type a user email: william.gates@microsoft.com
User with email 'william.gates@microsoft.com' found! ID: 10, Name: Bill Gates
```

这是第二种情况的输出：

```java
Loaded 5 unique users.
Type a user email: randomstring
Sorry, user with email randomstring not found.
```

恭喜！在这个练习中，您使用了`Hashtable`来快速通过电子邮件地址找到用户。

### 活动 30：使用 Hashtable 对数据进行分组

Hashtable 的一个非常常见的用法是根据某个键对记录进行分组。在这个活动中，您将使用它来计算上一个活动中下载的文件的最低、最高和平均工资。

如果还没有，请转到此页面：[`github.com/TrainingByPackt/Java-Fundamentals/tree/master/Lesson07/data`](https://github.com/TrainingByPackt/Java-Fundamentals/tree/master/Lesson07/data)。要下载 CSV，可以单击**Adult_Data**。如前所述，此文件包含来自美国人口普查的收入数据。

有许多属性与每个工资相关联。在这个练习中，您将根据教育属性对记录进行分组。然后，像之前一样，打印最低、最高和平均工资，但现在是对每组工资进行的。

要完成此活动，您需要：

1.  使用`CSVReader`加载`adult.data` CSV 文件。这次，您将数据加载到一个 Hashtable 中，其中键是字符串，值是整数的向量。键将是教育属性，并且在向量中，您将存储与该教育相关的所有工资。

1.  现在，将所有工资分组在 Hashtable 中，现在可以遍历条目、键值对，并执行与上一个活动中相同的计算。

1.  对于每个条目，打印文件中找到的每个教育水平的最低、最高和平均工资。

#### 注意

此活动的解决方案可以在第 351 页找到。

## 泛型

与 Vector 等以通用方式与其他类一起工作的类一样，没有明确告诉编译器只接受一种类型的方法。因此，它在任何地方都使用 Object，并且需要在任何地方进行`instanceof`和转换等运行时检查。

为了解决这个问题，Java 5 中引入了泛型。在本节中，您将更好地了解问题、解决方案以及如何使用它。

### 问题是什么？

在声明数组时，您告诉编译器数组中包含的数据类型。如果尝试在其中添加其他内容，它将无法编译。看看以下代码：

```java
// This compiles and work
User[] usersArray = new User[1];
usersArray[0] = user;
// This wouldn't compile
// usersArray[0] = "Not a user";
/* If you uncomment the last line and try to compile, you would get the following error: */
File.java:15: error: incompatible types: String cannot be converted to User
        usersArray[0] = "Not a user";
                        ^
```

假设您尝试使用`Vector`做类似的事情，如下所示：

```java
Vector usersVector = new Vector();
usersVector.add(user); // This compiles
usersVector.add("Not a user"); // This also compiles
```

编译器将一点帮助也没有。`Hashtable`也是如此：

```java
Hashtable usersTable = new Hashtable();
usersTable.put(user.id, user); // This compiles
usersTable.put("Not a number", "Not a user"); // This also compiles
```

这也发生在获取数据时。当从数组中获取数据时，编译器知道其中包含的数据类型，因此您不需要对其进行转换：

```java
User userFromArray = usersArray[0];
```

要从集合中获取数据，您需要对数据进行转换。一个简单的例子是在向先前的`usersVector`添加两个元素后添加以下代码：

```java
User userFromVector = (User) usersVector.get(1);
```

它将编译，但会在运行时抛出`ClassCastException`：

```java
Exception in thread "main" java.lang.ClassCastException: java.lang.String cannot be cast to User
```

这在 Java 世界中很长一段时间是一个很大的错误源。然后泛型出现了，改变了一切。

泛型是一种告诉编译器泛型类只能与指定类型一起使用的方法。让我们看看这意味着什么：

+   **泛型类**：泛型类是一个具有泛型功能的类，可以与不同类型一起使用，比如 Vector，可以存储任何类型的对象。

+   **指定类型**：使用泛型时，当你实例化一个泛型类时，你要指定该泛型类将与何种类型一起使用。例如，你可以指定你只想在你的 Vector 中存储用户。

+   **编译器**：需要强调的是，泛型是一个仅在编译时存在的特性。在运行时，关于泛型类型定义的信息是不存在的。在运行时，一切都像在泛型之前一样运行。

泛型类有一个特殊的声明，公开了它需要多少种类型。一些泛型类需要多种类型，但大多数只需要一种。在泛型类的 Javadoc 中，有一个特殊的尖括号参数列表，指定了它需要多少个类型参数，比如`<T, R>`。以下是`java.util.Map`的 Javadoc 截图，它是集合框架中的一个接口之一：

![图 7.6：java.util.Map 的 Javadoc 截图，显示了泛型类型声明](img/C09581_Figure_07_06.jpg)

###### 图 7.6：java.util.Map 的 Javadoc 截图，显示了泛型类型声明

### 如何使用泛型

使用泛型时，在声明泛型类的实例时，你要使用尖括号指定该实例将使用的类型。以下是如何声明一个只处理用户的向量：

```java
Vector<User> usersVector = new Vector<>();
```

对于哈希表，你需要指定键和值的类型。对于一个将用户及其 ID 存储为键的哈希表，声明将如下所示：

```java
Hashtable<Integer, User> usersTable = new Hashtable<>();
```

只需使用正确的参数声明泛型类型，就可以解决我们之前描述的问题。例如，假设你正在声明一个只处理用户的向量。你会尝试将一个字符串添加到其中，如下面的代码所示：

```java
usersVector.add("Not a user");
```

然而，这将导致编译错误：

```java
File.java:23: error: no suitable method found for add(String)
        usersVector.add("Not a user");
                   ^
```

现在编译器确保只有用户会被添加到向量中，你可以从中获取数据而无需进行类型转换。编译器会自动为你转换类型：

```java
// No casting needed anymore
User userFromVector = usersVector.get(0);
```

### 练习 27：通过姓名或电子邮件中的文本查找用户

在这个练习中，你将编写一个应用程序，从 CSV 文件中读取用户到一个向量中，就像之前一样。然后，你将被要求输入一个字符串，该字符串将用于过滤用户。应用程序将打印出所有包含传入字符串的姓名或电子邮件的用户的一些信息：

1.  打开你的`UsersLoader.java`文件，并将所有的方法设置为使用集合的泛型版本。你的`loadUsersInHashtableByEmail`应该如下所示（只显示了已更改的行）：

```java
public static Hashtable<String, User> loadUsersInHashtableByEmail(String pathToFile)   
    throws IOException {
  Hashtable<String, User> users = new Hashtable<>();
  // Unchanged lines
}
```

你的`loadUsersInVector`应该如下所示（只显示了已更改的行）：

```java
public static Vector<User> loadUsersInVector(String pathToFile) throws IOException{
  Vector<User> users = new Vector<>();
  // Unchanged lines
}
```

#### 注意：

你不必更改其他调用这些方法的地方，因为使用它们作为非泛型版本仍然有效。

1.  创建一个名为`FindByStringWithGenerics.java`的文件，并添加一个同名的类和一个`main`方法，如下所示：

```java
public class FindByStringWithGenerics {
  public static void main (String [] args) throws IOException {
  }
}
```

1.  在你的`main`方法中添加一个对`loadUsersInVector`方法的调用，使用指定泛型类型的向量存储值。打印加载的用户数量：

```java
Vector<User> users = UsersLoader.loadUsersInVector(args[0]);
System.out.printf("Loaded %d users.\n", users.size());
```

1.  之后，要求用户输入一个字符串，并将其存储在一个变量中，转换为小写后存储：

```java
System.out.print("Type a string to search for: ");
// Read user input from command line
try (Scanner userInput = new Scanner(System.in)) {
  String toFind = userInput.nextLine().toLowerCase();
}
```

1.  在 try-with-resource 块内，创建一个变量来计算找到的用户数量。然后，遍历我们之前加载的向量中的用户，并为每个用户在电子邮件和姓名中搜索字符串，确保将所有字符串转换为小写：

```java
int totalFound = 0;
for (User user : users) {
  if (user.email.toLowerCase().contains(toFind)
        ||user.name.toLowerCase().contains(toFind)) {
    System.out.printf("Found user: %s",user.name);
    System.out.printf(" Email: %s\n", user.email);
    totalFound++;
  }
}
```

1.  最后，如果`totalFound`为零，表示没有找到用户，则打印友好的消息。否则，打印你找到的用户数量：

```java
if (totalFound == 0) {
  System.out.printf("No user found with string '%s'\n", toFind);
} else {
  System.out.printf("Found %d users with '%s'\n", totalFound, toFind);
}
```

以下是第一个案例的输出：

```java
Loaded 27 users.
Type a string to search for: will
Found user: Bill Gates Email: william.gates@microsoft.com
Found user: Bill Gates Email: william.gates@microsoft.com
Found user: Bill Gates Email: william.gates@microsoft.com
Found user: Bill Gates Email: william.gates@microsoft.com
Found user: Bill Gates Email: william.gates@microsoft.com
Found 5 users with 'will'
```

以下是第二个案例的输出：

```java
Loaded 27 users.
Type a string to search for: randomstring
No user found with string 'randomstring'
```

恭喜！现在你明白了泛型如何帮助你编写安全且易于使用的代码来处理你的集合。

### 排序和比较

在日常生活中，我们经常比较事物：冷/热，短/高，薄/厚，大/小。对象可以使用不同的标准进行比较。你可以按颜色、大小、重量、体积、高度、宽度等进行比较。在比较两个对象时，通常你想找出哪一个在某个标准上更多（或更少）或者它们在你使用的任何度量上是否相等。

有两种基本情况下比较对象很重要：找到最大值（或最小值）和排序。

在找到最大值或最小值时，你将所有对象相互比较，然后根据你所关注的标准选择获胜者。其他一切都可以忽略。你不需要跟踪其他对象，只要确保你不会无限次地重复比较同样的两个对象。

另一方面，排序更加复杂。你需要跟踪到目前为止已经比较过的所有元素，并确保在比较过程中保持它们排序。

集合框架包括一些接口、类和算法，可以帮助你处理所有这些。

### 可比较和比较器

在 Java 中，有一个描述对象如何相互比较的接口。`java.lang.Comparable`接口是一个泛型接口，只有一个需要实现的方法：`compareTo(T)`。根据 Javadocs，`compareTo`应该返回"负整数、零或正整数，表示此对象小于、等于或大于指定对象"。

为了理解它是如何工作的，让我们以一个字符串为例。字符串实现了`java.lang.Comparable<String>`，这意味着你可以比较两个字符串，如下所示：

```java
"A".compareTo("B") < 0 // -> true
"B".compareTo("A") > 0 // -> true
```

如果比较中第一个对象"小于"第二个，则它将返回一个负数（可以是任何数字，大小无关紧要）。如果两者相同，则返回零。如果第一个大于第二个，则返回一个正数（同样，大小无关紧要）。

这一切都很好，直到你遇到以下情况：

```java
"a".compareTo("B") < 0 // -> false
```

当你查看 String 的 Javadoc 时，它的`compareTo`方法说它"按字典顺序比较两个字符串"。这意味着它使用字符代码来检查哪个字符串排在前面。不同之处在于字符代码首先包括所有大写字母，然后是所有小写字母。因此，"A"在"B"之后，因为 B 的字符代码在 A 之前。

但是，如果我们想按字母顺序而不是按词典顺序比较字符串怎么办？如前所述，对象可以在许多不同的标准下进行比较。因此，Java 提供了另一个接口，可以用于比较两个对象：`java.util.Comparator`。类可以实现一个比较器，使用最常见的用例，比如数字可以使用它们的自然顺序进行比较。然后，我们可以创建另一个实现`Comparator`的类，使用一些其他自定义算法来比较对象。

### 练习 28：创建一个按字母顺序比较字符串的比较器

在这个练习中，你将创建一个实现`java.util.Comparator<String>`的类，用于按字母顺序比较字符串，而不是按词典顺序：

1.  创建一个名为`AlphabeticComparator.java`的文件，并添加一个同名的类，该类实现`java.util.Comparator<String>`（不要忘记导入）：

```java
import java.util.Comparator;
public class AlphabeticComparator implements Comparator<String> {
  public int compare(String first, String second) {
  }
}
```

1.  在`compareTo`方法中，你只需将两个字符串转换为小写，然后进行比较：

```java
return first.toLowerCase().compareTo(second.toLowerCase());
```

1.  创建一个名为`UseAlphabeticComparator.java`的新文件，并添加一个同名的类，其中包含一个`main`方法，以便你可以测试你的新比较器：

```java
public class UseAlphabeticComparator {
  public static void main (String [] args) {
  }
}
```

1.  现在实例化你的类，并编写一些测试用例，以确保你的类按预期工作：

```java
AlphabeticComparator comparator = new AlphabeticComparator();
System.out.println(comparator.compare("A", "B") < 0); // -> true
System.out.println(comparator.compare("B", "A") > 0); // -> true
System.out.println(comparator.compare("a", "B") < 0); // -> true
System.out.println(comparator.compare("b", "A") > 0); // -> true
System.out.println(comparator.compare("a", "b") < 0); // -> true
System.out.println(comparator.compare("b", "a") > 0); // -> true
```

输出如下：

```java
true
true
true
true
true
true
```

恭喜！你写了你的第一个比较器。现在，让我们继续看看你可以用 Comparables 和 Comparators 做些什么。

### 排序

当你有对象的集合时，很常见希望以某种方式对它们进行排序。能够比较两个对象是所有排序算法的基础。现在你知道如何比较对象了，是时候利用它来为你的应用程序添加排序逻辑了。

有许多排序算法，每种算法都有其自身的优势和劣势。为简单起见，我们只讨论两种：冒泡排序，因为它简单；归并排序，因为它的稳定性表现良好，这也是 Java 核心实现者选择它的原因。

### 冒泡排序

最天真的排序算法是冒泡排序，但它也是最简单的，易于理解和实现。它通过迭代每个元素并将其与下一个元素进行比较来工作。如果找到两个未排序的元素，它会交换它们并继续下一个。当它到达数组的末尾时，它会检查有多少元素被交换。它会继续这个循环，直到一个循环中交换的元素数为零，这意味着整个数组或集合已经排序完成。

以下是使用冒泡排序对包含七个元素的数组进行排序的示例：

![图 7.7：展示冒泡排序工作原理的示例](img/C09581_Figure_07_07.jpg)

###### 图 7.7：展示冒泡排序工作原理的示例

冒泡排序非常节省空间，因为它不需要任何额外的数组或存储变量的地方。然而，它使用了大量的迭代和比较。在示例中，总共有 30 次比较和 12 次交换。

### 归并排序

冒泡排序虽然有效，但你可能已经注意到，它真的很天真，感觉浪费了很多循环。另一方面，归并排序更有效，基于分而治之的策略。它通过递归地将数组/集合一分为二，直到最终得到多个一元素对。然后，在排序的同时将它们合并在一起。你可以在下面的示例中看到它是如何工作的：

![图 7.8：归并排序算法的示例](img/C09581_Figure_07_08.jpg)

###### 图 7.8：归并排序算法的示例

与冒泡排序相比，归并排序的比较次数要小得多-仅为示例中的 13 次。它使用更多的内存空间，因为每个合并步骤都需要额外的数组来存储正在合并的数据。

在前面的示例中没有明确表达的一点是，归并排序具有稳定的性能，因为它总是执行相同数量的步骤；无论数据是多么混乱或排序。与冒泡排序相比，如果遇到数组/集合是反向排序的情况，交换的次数可能会非常高。 

稳定性对于诸如 Collections Framework 之类的核心库非常重要，这就是为什么归并排序被选为`java.util.Collections`实用类中排序的实现算法的原因。

### 活动 31：对用户进行排序

编写三个用户比较器：一个按 ID 比较，一个按名称比较，一个按电子邮件比较。然后，编写一个应用程序，加载唯一用户并按从命令行输入中选择的字段对用户进行排序。要完成此活动，你需要按照以下步骤进行：

1.  编写三个实现`java.util.Comparator<User>`的类。一个按 ID 比较，一个按名称比较，一个按电子邮件比较。

1.  使用返回`Hashtable`实例的方法从 CSV 中加载用户，这样你就有了一个包含唯一用户的集合。

1.  将`Hashtable`中的值加载到向量中，以便按指定顺序保留它们。

1.  从命令行读取输入以决定使用哪个字段进行排序。

1.  使用正确的比较器来使用`java.util.Collections`的 sort 方法对向量进行排序。

1.  打印用户。

#### 注意

这个活动的解决方案可以在第 354 页找到。

### 数据结构

构建应用程序最基本的部分是处理数据。存储数据的方式受到读取和处理数据的影响。数据结构定义了存储数据的方式。不同的数据结构针对不同的用例进行了优化。到目前为止，我们已经提到了两种访问数据的方式：

+   顺序地，就像数组或向量一样

+   键值对，就像哈希表一样

#### 注意

在接下来的几节中，我们将讨论已添加到集合框架中的基本数据结构接口，以及它们与其他接口的区别。我们还将深入研究每个实现以及它们解决的用例。

## 集合

这是最通用的接口，是除 Map 之外所有集合的基础。文档描述它表示一个称为元素的对象的集合。它声明了所有集合的基本接口，具有以下最重要的方法：

+   `add(Element)`: 将元素添加到集合中

+   `clear()`: 从集合中删除所有元素

+   `contains(Object)`: 检查对象是否在集合中

+   `remove(Object)`: 从集合中删除指定的元素（如果存在）

+   `size()`: 返回集合中存储的元素数量

### 列表

列表接口表示一个可以无限增长的元素的顺序集合。列表中的元素可以通过它们的索引访问，这是它们被放置的位置，但如果在其他元素之间添加元素，索引可能会改变。

当遍历列表时，元素将以确定性的顺序获取，并且始终基于它们的索引顺序，就像数组一样。

正如我们之前提到的，Vector 被改装以支持集合框架，并实现了列表接口。让我们看看其他可用的实现。

`List`扩展了`Collection`，因此它继承了我们之前提到的所有方法，并添加了一些其他重要的方法，主要与基于位置的访问相关：

+   `add(int, Element)`: 在指定位置添加一个元素

+   `get(int)`: 返回指定位置的元素

+   `indexOf(Object)`: 返回对象的索引，如果不在集合中则返回`-1`

+   `set(int, Element)`: 替换指定位置的元素

+   `subList(int, int)`: 从原始列表创建一个子列表

### ArrayList

就像 Vector 一样，ArrayList 包装了一个数组，并在需要时对其进行扩展，表现得就像一个动态数组。两者之间的主要区别在于向量是完全同步的。这意味着它们保护您免受并发访问（多线程应用程序）的影响。这也意味着在非并发应用程序中，这在大多数情况下发生，向量由于添加到其中的锁定机制而变慢。因此，建议您使用 ArrayList，除非您真的需要一个同步列表。

正如我们之前提到的，就所有目的而言，ArrayList 和 Vector 可以互换使用。它们的功能是相同的，都实现了相同的接口。

### LinkedList

LinkedList 是 List 的一种实现，它不像 ArrayList 或 Vector 那样在底层数组中存储元素。它将每个值包装在另一个称为节点的对象中。节点是一个包含对其他节点的两个引用（下一个节点和上一个节点）以及存储该元素的值的内部类。这种类型的列表被称为双向链表，因为每个节点都链接两次，一次在每个方向上：从前一个到下一个，从下一个到前一个。

在内部，LinkedList 存储对第一个和最后一个节点的引用，因此它只能从开始或结束处遍历列表。与数组、ArrayList 和向量一样，它不适用于随机或基于位置的访问，但在非常快速地添加不确定数量的元素时非常适用。

LinkedList 还存储一个变量，用于跟踪列表的大小。这样，它就不必每次都遍历列表来检查大小。

以下插图显示了 LinkedList 的实现方式：

![图 7.9：LinkedList 在内部是如何工作的。](img/C09581_Figure_07_09.jpg)

###### 图 7.9：LinkedList 在内部是如何工作的

### 地图

当您需要存储与键关联的元素时，可以使用地图。正如我们之前所看到的，Hashtable 是一种通过某个键对对象进行索引的强大机制，并且在添加了集合框架之后，Hashtable 被改装为实现 Map。

地图的最基本属性是它们不能包含重复的键。

地图之所以强大，是因为它们允许您从三个不同的角度查看数据集：键、值和键值对。将元素添加到地图后，您可以从这三个角度中的任何一个迭代它们，从而在从中提取数据时提供额外的灵活性。

`Map`接口中最重要的方法如下：

+   `clear()`: 从地图中删除所有键和值

+   `containsKey(Object)`: 检查地图中是否存在该键

+   `containsValue(Object)`: 检查地图中是否存在该值

+   `entrySet()`: 返回地图中所有键值对的集合

+   `get(Object)`: 如果存在，返回与指定键关联的值

+   `getOrDefault(Object, Value)`: 如果存在，返回与指定键关联的值，否则返回指定的值

+   `keySet()`: 包含地图中所有键的集合

+   `put(Key, Value)`: 添加或替换键值对

+   `putIfAbsent(Key, Value)`: 与上一个方法相同，但如果键已经存在，则不会替换

+   `size()`: 此地图中键值对的数量

+   `values()`: 返回此地图中所有值的集合

### HashMap

就像`Hashtable`一样，`HashMap`实现了哈希表来存储键值对的条目，并且工作方式完全相同。正如 Vector 是 ArraySet 一样，Hashtable 是`HashMap`一样。`Hashtable`存在于 Map 接口之前，因此 HashMap 被创建为哈希表的非同步实现。

正如我们之前提到的，哈希表，因此 HashMap，非常快速地通过键找到元素。它们非常适合用作内存缓存，您可以在其中加载已由某个字段键入的数据，就像在*练习 26*中所做的那样：*编写一个按电子邮件查找用户的应用程序*。

### TreeMap

`TreeMap`是可以按键或指定比较器对键值对进行排序的 Map 的实现。

正如其名称所示，TreeMap 使用树作为底层存储机制。树是非常特殊的数据结构，用于在插入发生时保持数据排序，并且同时使用非常少的迭代获取数据。以下插图显示了树的外观以及如何快速找到元素的获取操作，即使在非常大的树中也是如此：

![图 7.10：正在遍历树数据结构以获取元素](img/C09581_Figure_07_10.jpg)

###### 图 7.10：正在遍历树数据结构以获取元素

树具有代表分支的节点。一切都始于根节点，并扩展为多个分支。在叶节点的末端，有没有子节点的节点。TreeMap 实现了一种称为红黑树的特定类型的树，这是一种二叉树，因此每个节点只能有两个子节点。

### LinkedHashMap

`LinkedHashMap`类的名称有点神秘，因为它在内部使用了两种数据结构来支持一些 HashMap 不支持的用例：哈希表和链表。哈希表用于快速向地图中添加和获取元素。链表用于通过任何方式迭代条目：键、值或键值对。这使得它能够以确定的顺序迭代条目，这取决于它们被插入的顺序。

### Set

集合的主要特征是它们不包含重复元素。当您想要收集元素并同时消除重复值时，集合非常有用。

关于集合的另一个重要特征是，根据实现的不同，从集合中获取元素的顺序也会有所不同。这意味着如果您想要消除重复项，您必须考虑之后如何读取它们。

集合框架中的所有集合实现都基于它们对应的 Map 实现。唯一的区别是它们将集合中的值处理为映射中的键。

### HashSet

迄今为止，所有集合中最常见的 HashSet 使用 HashMap 作为底层存储机制。它根据 HashMap 中使用的哈希函数存储其元素的随机顺序。

### TreeSet

由 TreeMap 支持，`TreeSet`在想要按其自然顺序（可比较的）或使用比较器对其进行排序的唯一元素时非常有用。

### LinkedHashSet

由`LinkedHashMap`支持，`LinkedHashSet`将保持插入顺序并在添加到集合时删除重复项。它具有与 LinkedHashSet 相同的优点：像 HashSet 一样快速插入和获取，像 LinkedList 一样快速迭代。

### 练习 29：使用 TreeSet 打印排序后的用户

在*Activity 31*：*Sorting Users*中，您编写了三个可用于对用户进行排序的比较器。让我们使用它们和 TreeSet 来制作一个以更高效的方式打印排序后用户的应用程序：

1.  向您的`UsersLoader`类添加一个可以将用户加载到`Set`中的方法：

```java
public static void loadUsersIntoSet(String pathToFile, Set<User> usersSet)
    throws IOException {
  FileReader fileReader = new FileReader(pathToFile);
  BufferedReader lineReader = new BufferedReader(fileReader);
  try(CSVReader reader = new CSVReader(lineReader)) {
    String [] row = null;
    while ( (row = reader.readRow()) != null) {
      usersSet.add(User.fromValues(row));
    }
  }
}
```

1.  导入`Set`如下：

```java
java.util.Set;
```

1.  创建一个名为`SortUsersTreeSet.java`的新文件，并添加一个同名的类并添加一个`main`方法：

```java
public class SortUsersTreeSet {
  public static void main (String [] args) throws IOException {
  }
}
```

1.  从命令行读取我们将按哪个字段进行排序：

```java
Scanner reader = new Scanner(System.in);
System.out.print("Type a field to sort by: ");
String input = reader.nextLine();
Comparator<User> comparator;
switch(input) {
  case "id":
    comparator = new ByIdComparator();
    break;
  case "name":
    comparator = new ByNameComparator();
    break;
  case "email":
    comparator = new ByEmailComparator();
    break;
  default:
    System.out.printf("Sorry, invalid option: %s\n", input);
    return;
}
System.out.printf("Sorting by %s\n", input);
```

1.  使用指定的比较器创建一个用户的`TreeSet`，使用您的新方法将用户加载到其中，然后将加载的用户打印到命令行：

```java
TreeSet<User> users = new TreeSet<>(comparator);
UsersLoader.loadUsersIntoSet(args[0], users);
for (User user : users) {
  System.out.printf("%d - %s, %s\n", user.id, user.name, user.email);
}
```

以下是第一种情况的输出：

```java
Type a field to sort by: address
Sorry, invalid option: address
```

以下是第二种情况的输出

```java
Type a field to sort by: email
Sorting by email
30 - Jeff Bezos, jeff.bezos@amazon.com
50 - Larry Ellison, lawrence.ellison@oracle.com
20 - Marc Benioff, marc.benioff@salesforce.com
40 - Sundar Pichai, sundar.pichai@google.com
10 - Bill Gates, william.gates@microsoft.com
```

以下是第三种情况的输出

```java
Type a field to sort by: id
Sorting by id
10 - Bill Gates, william.gates@microsoft.com
20 - Marc Benioff, marc.benioff@salesforce.com
30 - Jeff Bezos, jeff.bezos@amazon.com
40 - Sundar Pichai, sundar.pichai@google.com
50 - Larry Ellison, lawrence.ellison@oracle.com
```

以下是第四种情况的输出

```java
Type a field to sort by: name
Sorting by name
10 - Bill Gates, william.gates@microsoft.com
30 - Jeff Bezos, jeff.bezos@amazon.com
50 - Larry Ellison, lawrence.ellison@oracle.com
20 - Marc Benioff, marc.benioff@salesforce.com
40 - Sundar Pichai, sundar.pichai@google.com
```

恭喜！在这个练习中，您使用 TreeSet 对从 CSV 文件加载的元素进行排序和去重，同时完成了这些操作。

### Queue

队列是一种特殊的数据结构，遵循先进先出（FIFO）模式。这意味着它按插入顺序保留元素，并且可以从第一个插入的元素开始返回元素，同时将元素添加到末尾。这样，新的工作可以排队在队列的末尾，而要处理的工作可以从前面出列。以下是此过程的示例：

![图 7.11：存储要处理的工作的队列](img/C09581_Figure_07_11.jpg)

###### 图 7.11：存储要处理的工作的队列

在集合框架中，队列由`java.util.Queue`接口表示。要将元素入队，可以使用`add(E)`或`offer(E)`。第一个如果队列已满将抛出异常，而第二个则只会返回`true`或`false`，告诉您操作是否成功。它还有出队元素或只检查队列前面的元素的方法。`remove()`将返回并移除队列前面的元素，如果队列为空则抛出异常。`poll()`将返回并移除元素，如果队列为空则返回 null。`element()`和`peek()`的工作方式相同，但只返回元素而不从队列中移除，第一个抛出异常，后者如果队列为空则返回 null。

`java.util.Deque`是一个接口，它扩展了`java.util.Queue`，具有额外的方法，允许在队列的两侧添加、移除或查看元素。

`java.util.LinkedList`是`java.util.Queue`和`java.util.Deque`的实现，也实现了`java.util.List`。

### java.util.ArrayDeque

队列和双端队列的实现使用数组作为底层数据存储。数组会自动增长以支持添加到其中的数据。

### java.util.PriorityQueue

队列的实现使用堆来保持元素的排序顺序。如果元素实现了`java.lang.Comparable`，则可以由元素来确定顺序，或者可以通过传入的比较器来确定顺序。堆是一种特殊类型的树，它可以保持元素排序，类似于`TreeMap`。这种队列的实现非常适合需要按一定优先级处理的元素。

### 练习 30：虚假电子邮件发送器

在这个练习中，您将模拟使用一个处理器向用户发送电子邮件的过程。为此，您将编写两个应用程序：一个模拟发送电子邮件，另一个从 CSV 中读取并为每个用户调用第一个。强制您使用队列的约束是一次只能运行一个进程。这意味着当用户从 CSV 中加载时，您将对其进行排队，并在可能的情况下发送电子邮件：

1.  创建一个名为`EmailSender.java`的文件，其中包含一个类和一个`main`方法。为了模拟发送电子邮件，该类将休眠随机的一段时间，最多一秒：

```java
System.out.printf("Sending email to %s...\n", args[0]);
Thread.sleep(new Random().nextInt(1000));
System.out.printf("Email sent to %s!\n", args[0]);
```

1.  创建另一个名为`SendAllEmails.java`的文件，其中包含一个类和一个`main`方法。

```java
public class SendAllEmails {
```

1.  添加一个名为`runningProcess`的`static`字段。这将代表正在运行的发送电子邮件过程：

```java
private static Process runningProcess = null;
```

1.  创建一个`static`方法，该方法将尝试通过从队列中出队一个元素来启动发送电子邮件的过程，如果该过程可用：

```java
private static void sendEmailWhenReady(ArrayDeque<String> queue)
    throws Exception {
  // If running, return
  if (runningProcess != null && runningProcess.isAlive()) {
    System.out.print(".");
    return;
  }
  System.out.print("\nSending email");
  String email = queue.poll();
  String classpath = System.getProperty("java.class.path");
  String[] command = new String[]{
    "java", "-cp", classpath, "EmailSender", email
  };
  runningProcess = Runtime.getRuntime().exec(command);
}
```

1.  在`main`方法中，创建一个字符串的`ArrayDeque`，表示要发送的电子邮件队列：

```java
ArrayDeque<String> queue = new ArrayDeque<>();
```

1.  打开 CSV 文件以从中读取每一行。您可以使用`CSVReader`来实现这一点：

```java
FileReader fileReader = new FileReader(args[0]);
BufferedReader bufferedReader = new BufferedReader(fileReader);
try (CSVReader reader = new CSVReader(bufferedReader)) {
  String[] row;
  while ( (row = reader.readRow()) != null) {
    User user = User.fromValues(row);
  }
}
```

1.  用户加载后，我们可以将其电子邮件添加到队列中，并立即尝试发送电子邮件：

```java
queue.offer(user.email);
sendEmailWhenReady(queue);
```

1.  由于从文件中读取通常非常快，我们将通过添加一些睡眠时间来模拟缓慢读取：

```java
Thread.sleep(100);
```

1.  在 try-with-resources 块之外，也就是在我们完成从文件中读取所有用户之后，我们需要确保排空队列。为此，我们可以使用一个`while`循环，只要队列不为空就运行：

```java
while (!queue.isEmpty()) {
  sendEmailWhenReady(queue);

  // Wait before checking again
  Thread.sleep(100);
}
```

#### 注意

在这种情况下，很重要的一点是在你睡觉的时候不要使用 100%的 CPU。这在处理队列中的元素时非常常见，就像在这种情况下一样。

1.  现在您可以等待最后一个发送电子邮件过程完成，遵循类似的模式：检查并在睡眠时等待：

```java
while (runningProcess.isAlive()) {
  System.out.print(".");
  Thread.sleep(100);
}
System.out.println("\nDone sending emails!");
```

恭喜！您编写了一个应用程序，使用受限资源（仅一个进程）模拟发送电子邮件。该应用程序忽略了文件中用户的重复情况。它还忽略了发送电子邮件过程的输出。您将如何实现重复发送检测器并避免该问题？您认为发送过程的输出如何影响重复避免的决定？

### 集合的属性

在选择数据结构解决问题时，您将不得不考虑以下事项：

+   排序 - 如果在访问数据时顺序很重要，数据将以什么顺序被访问？

+   独特性 - 如果在集合内部多次具有相同的元素，这是否重要？你如何定义独特性？

+   可空性 - 值是否可以为空？如果将键映射到值，空键是否有效？在任何情况下使用空是否有意义？

使用以下表格确定哪种集合更适合您的用例：

![表 7.1：表示集合属性的表格](img/C09581_Table_07_01.jpg)

###### 表 7.1：表示集合属性的表格

#### 注意

“自然排序”意味着它将根据元素（或键）进行排序，如果元素实现了`Comparable`，或者使用传入的比较器进行排序。

## 摘要

在开发应用程序时，处理数据是最基本的任务之一。在本课程中，您学会了如何从文件中读取和解析数据，以便能够将其作为应用程序的一部分进行处理。您还学会了如何比较对象，以便以不同的方式对其进行排序。

作为处理数据的一部分，您学会了如何使用基本和高级数据结构存储数据。了解如何高效地处理数据非常重要，以便避免资源争用场景，例如内存耗尽，或者需要太多的处理或时间来执行手头的任务。高效处理数据的一个重要部分是选择适合特定问题的正确数据结构和算法。您添加到工具库中的所有新工具将帮助您在构建 Java 应用程序时做出正确的决策。

在下一课中，我们将看一些高级数据结构。
