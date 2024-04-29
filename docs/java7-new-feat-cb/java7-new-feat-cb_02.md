# 第二章：使用路径定位文件和目录

在本章中，我们将涵盖以下内容：

+   创建 Path 对象

+   java.io.File 和 java.nio.file.Files 之间的互操作性

+   将相对路径转换为绝对路径

+   通过规范化路径来消除冗余

+   使用路径解析合并路径

+   在两个位置之间创建路径

+   在不同路径类型之间转换

+   确定两个路径是否等价

+   管理符号链接

# 介绍

文件系统是计算机上组织数据的一种方式。通常，它由一个或多个顶级目录组成，每个目录包含一系列文件。顶级目录通常被称为根。此外，文件系统存储在介质上，称为文件存储。

Java 7 引入了许多新的类和接口，使得与文件系统的工作更加简单和高效。这些类在很大程度上取代了`java.io`包中的旧类。

在本章和后续章节中，我们将演示如何使用目录结构管理文件系统，如下图所示：

![Introduction](img/5627_2_01.jpg)

椭圆代表目录，而矩形代表文件。基于 Unix 的系统和 Windows 系统在对根节点的支持上有所不同。Unix 系统支持单个根节点，而 Windows 系统允许多个根节点。目录或文件的位置使用路径来描述。路径的元素、目录和文件之间用正斜杠或反斜杠分隔。在 Unix 中使用正斜杠，在 Windows 中使用反斜杠。

音乐文件来自[`freepd.com/70s%20Sci%20Fi/`](http://freepd.com/70s%20Sci%20Fi/)。`status.txt`用于保存简单的状态信息，而`users.txt`则用于保存用户列表。音乐目录中的`users.txt`文件是指向`docs`目录中实际文件的符号链接，如红线所示。这些文件将在本章的各个示例中使用。当然，您可以使用任何您希望的文件或文件结构。

符号链接在基于 Unix 的平台上更常见。要为音乐目录中的`users.txt`文件创建符号链接，请在命令控制台中使用以下命令：`mklink users.txt c:\home\docs\users.txt`。这需要管理员权限才能执行。

本章涉及由`java.nio.file.Path`类表示的路径的管理。`Path`对象被`java.nio`包中的类广泛使用，由以下几个部分组成：

+   作为路径基础的根目录，比如 C 盘

+   用于分隔路径中组成目录和文件的名称的分隔符

+   中间目录的名称

+   终端元素，可以是文件或目录

这些内容在*理解路径*一节中进行了讨论和说明。以下是处理文件和目录的类：

+   `java.nio.file.Paths`包含用于创建`Path`对象的静态方法

+   `java.nio.file.Path`接口包含许多用于处理路径的方法

+   `java.nio.file.FileSystems`是用于访问文件系统的主要类

+   `java.nio.file.FileSystem`表示文件系统，比如 UNIX 系统上的/或 Windows 平台上的 C 盘

+   `java.nio.file.FileStore`表示实际存储设备并提供设备特定信息

+   `java.nio.file.attribute.FileStoreAttributeView`提供对文件信息的访问

最后两个类在后续章节中会更深入地讨论。为了访问文件或目录，我们通常会使用`FileSystems`类的`getDefault`方法来检索 JVM 可访问的文件系统的引用。要访问特定驱动器，我们可以使用`getFileSystem`方法，传入表示感兴趣的驱动器或目录的**统一资源标识符**（**URI**）对象。

`FileSystems`类提供了创建或访问文件系统的技术。在本章中，我们对类如何支持创建`Path`对象感兴趣。一旦我们有了文件系统对象的引用，我们就可以使用几种方法之一获取`Path`对象：

+   `getPath：`这使用系统相关路径来获取`Path`对象。`Path`对象用于定位和访问文件。

+   `getPathMatcher：`这将创建一个`PathMatcher`。它执行文件的各种匹配类型操作，并在第五章的“获取文件系统信息”配方中进行了讨论。

+   `getRootDirectories：`用于获取根目录列表。这个方法在第五章的“获取文件系统信息”配方中进行了说明。

*理解路径*配方介绍了`Path`对象的创建和一般用法。这些知识在后续配方和其他章节中使用，因此请确保理解本配方中涵盖的基本过程。

您仍然可以使用较旧的`java.io`包元素。可以使用`File`类的`toPath`方法创建表示`java.io.File`对象的路径。这在*java.io.File 和 java.nio.file.Files 之间的互操作性*配方中进行了讨论，并且在维护较旧的代码时可能会有用。

路径可以是相对的，也可以是绝对的。这些类型的路径以及处理它们的技术在“使用相对和绝对路径”配方中进行了讨论。

路径可能包含冗余和多余的元素。去除这些元素称为**规范化**。通过“通过规范化路径来删除路径中的冗余”配方，我们可以检查简化这些类型路径的可用技术。

路径可以组合成一个新的复合路径。这称为解析路径，并在*使用路径解析合并路径*配方中进行了讨论。这种技术可以用于创建新的路径，其中路径的部分来自不同的来源。

当需要文件的引用时，该路径有时相对于当前位置或其他位置。*在两个位置之间创建路径*配方说明了创建这样一个路径的过程。这个过程称为**相对化**。

不仅有相对和绝对路径，还有其他表示路径的方式，例如使用`java.net.URI`对象。创建`Path`对象时，并不一定需要实际路径存在。例如，可以创建`Path`以创建新的文件系统元素。*在不同路径类型之间转换*配方介绍了用于在这些不同类型路径之间转换的方法。

路径是依赖于系统的。也就是说，UNIX 系统上的路径与 Windows 系统上找到的路径不同。比较在同一平台上找到的两个路径可能相同，也可能不同。这在*确定两个路径是否等效*配方中进行了研究。

# 创建 Path 对象

需要路径来标识目录或文件。本配方的重点是如何为典型的文件和目录操作获取`Path`对象。路径在本章和许多后续章节中用于大多数配方，这些配方涉及文件和目录。

有几种方法可以创建或返回`Path`对象。在这里，我们将研究用于创建`Path`对象的方法以及如何使用其方法来进一步了解 Java 中使用的路径概念。

## 准备工作

为了创建`Path`对象，我们需要使用以下方法之一：

+   `FileSystem`类的`getPath`方法

+   `Paths`类的`get`方法

我们将首先使用`getPath`方法。`get`方法在本配方的*更多*部分中进行了解释。

## 如何做...

1.  创建一个带有`main`方法的控制台应用程序。在`main`方法中，添加以下代码序列，为文件`status.txt`创建一个`Path`对象。我们将使用几种`Path`类的方法来检查创建的路径，如下所示：

```java
Path path = FileSystems.getDefault().getPath("/home/docs/status.txt");
System.out.println();
System.out.printf("toString: %s\n", path.toString());
System.out.printf("getFileName: %s\n", path.getFileName());
System.out.printf("getRoot: %s\n", path.getRoot());
System.out.printf("getNameCount: %d\n", path.getNameCount());
for(int index=0; index<path.getNameCount(); index++) {
System.out.printf("getName(%d): %s\n", index, path.getName(index));
}
System.out.printf("subpath(0,2): %s\n", path.subpath(0, 2));
System.out.printf("getParent: %s\n", path.getParent());
System.out.println(path.isAbsolute());
}

```

1.  注意在`path`字符串中使用正斜杠。这种方法在任何平台上都可以工作。但是，在 Windows 上，您还可以使用如下所示的反斜杠：

```java
Path path = FileSystems.getDefault().getPath("\\home\\docs\\status.txt");

```

1.  在 Windows 平台上，任何一种方法都可以工作，但使用正斜杠更具可移植性。

1.  执行程序。您的输出应该如下所示：

toString: \home\docs\status.txt

getFileName: status.txt

getRoot: \

getNameCount: 3

getName(0): home

getName(1): docs

getName(2): status.txt

subpath(0,2): home\docs

getParent: \home\docs

false

## 它是如何工作的...

使用调用链接创建了`Path`对象，从`FileSystems`类的`getDefault`方法开始。这返回一个表示 JVM 可用文件系统的`FileSystem`对象。`FileSystem`对象通常指的是当前用户的工作目录。接下来，使用表示感兴趣文件的字符串执行了`getPath`方法。

代码的其余部分使用了各种方法来显示有关路径的信息。正如本章介绍中所详细介绍的那样，我们可以使用`Path`类的方法来显示有关路径部分的信息。`toString`方法针对路径执行，以说明默认情况下会得到什么。

`getFileName`返回了`Path`对象的文件名，`getRoot`返回了根目录。`getNameCount`方法返回了中间目录的数量加上一个文件名。for 循环列出了路径的元素。在这种情况下，有两个目录和一个文件，总共三个。这三个元素组成了路径。

虽然使用简单的 for 循环来显示这些名称，但我们也可以使用`iterator`方法来列出这些名称，如下面的代码所示：

```java
Iterator iterator = path.iterator();
while(iterator.hasNext()) {
System.out.println(iterator.next());
}

```

`Path`对象可能包括其他路径。可以使用`subpath`方法检索子路径。该方法具有两个参数。第一个表示初始索引，第二个参数指定排他性的最后索引。在此示例中，第一个参数设置为 0，表示要检索根级目录。最后一个索引设置为 2，这意味着只列出了顶部两个目录。

在这种情况下，`getParent`方法也返回相同的路径。但是，请注意它以反斜杠开头。这表示从每个元素的顶级元素开始，但最后一个元素除外的路径。

## 还有更多...

有几个问题需要进一步考虑：

+   使用`Paths`类的`get`方法

+   父路径的含义

### 使用 Paths 类的 get 方法

`Paths`类的`get`方法也可以用于创建`Path`对象。此方法使用可变数量的`String`参数来构造路径。在以下代码序列中，创建了一个从当前文件系统的根目录开始的`path`：

```java
try {
path = Paths.get("/home", "docs", "users.txt");
System.out.printf("Absolute path: %s", path.toAbsolutePath());
}
catch (InvalidPathException ex) {
System.out.printf("Bad path: [%s] at position %s",
ex.getInput(), ex.getIndex());
}

```

使用`toAbsolutePath`方法的输出显示了构建的路径。注意“E”元素。代码在 Windows 系统上执行，当前驱动器为“E”驱动器。`toAbsolutePath`方法在“使用相对路径和绝对路径”配方中进行了讨论。

绝对路径: E:\home\docs\users.txt

如果我们在路径的`String`中不使用斜杠，那么路径是基于当前工作目录创建的。删除斜杠并执行程序。您的输出应该类似于以下内容，其中“currentDirectory”被执行代码时使用的内容替换：

绝对路径: currentDirectory\home\docs\users.txt

使用“resolve”方法是一种更灵活的方法，如“使用路径解析合并路径”配方中所讨论的。

将输入参数转换为路径是依赖于系统的。如果用于创建路径的字符对于文件系统无效，则会抛出`java.nio.file.InvalidPathException`。例如，在大多数文件系统中，空值是一个非法字符。为了说明这一点，将反斜杠 0 添加到`path`字符串中，如下所示：

```java
path = Paths.get("/home\0", "docs", "users.txt");

```

执行时，部分输出将如下所示：

**错误路径：[/home \docs\users.txt] 位置在第 5 位**

`InvalidPathException`类的`getInput`方法返回用于创建路径的连接字符串。`getIndex`方法返回有问题的字符的位置，在本例中是空字符。

### 父路径的含义

`getParent`方法返回父路径。但是，该方法不访问文件系统。这意味着对于给定的`Path`对象，可能有也可能没有父级。

考虑以下路径声明：

```java
path = Paths.get("users.txt");

```

这是在当前工作目录中找到的`users.txt`文件。`getNameCount`将返回 1，`getParent`方法将返回 null。实际上，文件存在于目录结构中，并且有一个根和一个父级。因此，该方法的结果在某些情境下可能无用。

使用此方法大致相当于使用`subpath`方法：

```java
path = path.subpath(0,path.getNameCount()-1));

```

## 另请参阅

`toRealPath`方法在*使用相对路径和绝对路径*和*通过规范化路径来消除冗余*中有讨论。

# java.io.File 和 java.nio.file.Files 之间的互操作性

在引入`java.nio`包之前，`java.io`包的类和接口是 Java 开发人员用于处理文件和目录的唯一可用选项。虽然较新的包已经补充了`java.io`包的大部分功能，但仍然可以使用旧类，特别是`java.io.File`类。本文介绍了如何实现这一点。

## 准备工作

要使用`File`类获取`Path`对象，需要按照以下步骤进行：

1.  创建一个表示感兴趣文件的`java.io.File`对象

1.  应用`toPath`方法以获得`Path`对象

## 如何做...

1.  创建一个控制台应用程序。添加以下主要方法，我们在其中创建一个`File`对象和一个表示相同文件的`Path`对象。接下来，我们比较这两个对象，以确定它们是否表示相同的文件：

```java
public static void main(String[] args) {
try {
Path path =
Paths.get(new URI("file:///C:/home/docs/users.txt"));
File file = new File("C:\\home\\docs\\users.txt");
Path toPath = file.toPath();
System.out.println(toPath.equals(path));
}
catch (URISyntaxException e) {
System.out.println("Bad URI");
}
}

```

1.  当执行应用程序时，输出将为 true。

## 工作原理...

创建了两个`Path`对象。第一个`Path`对象是使用`Paths`类的`get`方法声明的。它使用`java.net.URI`对象为`users.txt`文件创建了一个`Path`对象。第二个`Path`对象`toPath`是从`File`对象使用`toPath`方法创建的。使用`Path`的`equals`方法来证明这些路径是等价的。

### 提示

注意使用正斜杠和反斜杠表示文件的字符串。`URI`字符串使用正斜杠，这是与操作系统无关的。而反斜杠用于 Windows 路径。

## 另请参阅

*理解路径*中演示了创建`Path`对象。此外，*使用相对路径和绝对路径*中讨论了创建`URI`对象。

# 将相对路径转换为绝对路径

路径可以表示为绝对路径或相对路径。两者都很常见，在不同情况下都很有用。`Path`类和相关类支持创建绝对路径和相对路径。

相对路径用于指定文件或目录的位置与当前目录位置的关系。通常，使用一个点或两个点来表示当前目录或下一个更高级目录。但是，在创建相对路径时，不需要使用点。

绝对路径从根级别开始，列出每个目录，用正斜杠或反斜杠分隔，取决于操作系统，直到达到所需的目录或文件。

在本示例中，我们将确定当前系统使用的路径分隔符，并学习如何将相对路径转换为绝对路径。在处理文件名的用户输入时，这是有用的。与绝对和相对路径相关的是路径的 URI 表示。我们将学习如何使用`Path`类的`toUri`方法来返回给定路径的这种表示。

## 准备工作

在处理绝对和相对路径时，经常使用以下方法：

+   `getSeparator`方法确定文件分隔符

+   `subpath`方法获取路径的一个部分或所有部分/元素

+   `toAbsolutePath`方法获取相对路径的绝对路径

+   `toUri`方法获取路径的 URI 表示

## 如何做...

1.  我们将逐个解决前面的每个方法。首先，使用以下`main`方法创建一个控制台应用程序：

```java
public static void main(String[] args) {
String separator = FileSystems.getDefault().getSeparator();
System.out.println("The separator is " + separator);
try {
Path path = Paths.get(new URI("file:///C:/home/docs/users.txt"));
System.out.println("subpath: " + path.subpath(0, 3));
path = Paths.get("/home", "docs", "users.txt");
System.out.println("Absolute path: " + path.toAbsolutePath());
System.out.println("URI: " + path.toUri());
}
catch (URISyntaxException ex) {
System.out.println("Bad URI");
}
catch (InvalidPathException ex) {
System.out.println("Bad path: [" + ex.getInput() + "] at position " + ex.getIndex());
}
}

```

1.  执行程序。在 Windows 平台上，输出应如下所示：

**分隔符是\**

**子路径：home\docs\users.txt**

**绝对路径：E:\home\docs\users.txt**

**URI：file:///E:/home/docs/users.txt**

## 工作原理...

`getDefault`方法返回一个表示 JVM 当前可访问的文件系统的`FileSystem`对象。对此对象执行`getSeparator`方法，返回一个反斜杠字符，表示代码在 Windows 机器上执行。

为`users.txt`文件创建了一个`Path`对象，并对其执行了`subpath`方法。这个方法在*理解路径*中有更详细的讨论。`subpath`方法总是返回一个相对路径。

接下来，使用`get`方法创建了一个路径。由于第一个参数使用了正斜杠，路径从当前文件系统的根开始。在这个例子中，提供的路径是相对的。

路径的 URI 表示与绝对和相对路径相关。`Path`类的`toUri`方法返回给定路径的这种表示。`URI`对象用于表示互联网上的资源。在这种情况下，它返回了一个文件的 URI 方案形式的字符串。

绝对路径可以使用`Path`类的`toAbsolutePath`方法获得。绝对路径包含路径的根元素和所有中间元素。当用户被要求输入文件名时，这可能很有用。例如，如果用户被要求提供一个文件名来保存结果，文件名可以添加到表示工作目录的现有路径中。然后可以获取绝对路径并根据需要使用。

## 还有更多...

请记住，`toAbsolutePath`方法无论路径引用有效文件还是目录都可以工作。前面示例中使用的文件不需要存在。考虑使用如下代码中所示的虚假文件。假设文件`bogusfile.txt`不存在于指定目录中：

```java
Path path = Paths.get(new URI("file:///C:/home/docs/bogusfile.txt"));
System.out.println("File exists: " + Files.exists(path));
path = Paths.get("/home", "docs", "bogusfile.txt");
System.out.println("File exists: " + Files.exists(path));

```

程序执行时，输出如下：

**分隔符是\**

**文件存在：false**

**子路径：home\docs\bogusfile.txt**

**文件存在：false**

**绝对路径：E:\home\docs\bogusfile.txt**

**URI：file:///E:/home/docs/bogusfile.txt**

如果我们想知道这是否是一个真实的路径，我们可以使用`toRealPath`方法，如*通过规范化路径来删除路径中的冗余*中所讨论的那样。

## 另请参阅

可以使用`normalize`方法删除路径中的冗余，如*通过规范化路径来删除路径中的冗余*中所讨论的那样。

当符号链接用于文件时，路径可能不是文件的真实路径。`Path`类的`toRealPath`方法将返回文件的真实绝对路径。这在*通过规范化路径来消除冗余*示例中进行了演示。

# 通过规范化路径消除冗余

当在定义路径时使用“.”或“..”符号时，它们的使用可能会引入冗余。也就是说，所描述的路径可能通过删除或以其他方式更改路径来简化。本示例讨论了使用`normalize`方法来影响这种转换。通过简化路径，可以避免错误并提高应用程序的性能。`toRealPath`方法还执行规范化，并在本示例的*还有更多...*部分进行了解释。

## 准备就绪

消除路径中冗余的基本步骤包括以下内容：

+   识别可能包含冗余的路径

+   使用`normalize`方法消除冗余

## 如何做...

介绍中的目录结构在此处复制以方便起见：

![如何做...](img/5627_2_01.jpg)

首先考虑以下路径：

```java
/home/docs/../music/ Space Machine A.mp3
/home/./music/ Robot Brain A.mp3

```

这些包含冗余或多余的部分。在第一个示例中，路径从`home`开始，然后进入`docs`目录的一个目录级别。然后，`.`符号将路径返回到`home`目录。然后继续进入`music`目录并到`mp3`文件。`docs/.`元素是多余的。

在第二个示例中，路径从`home`开始，然后遇到一个句点。这代表当前目录，即`home`目录。接下来，路径进入`music`目录，然后遇到`mp3`文件。`/`是多余的，不需要。

1.  创建一个新的控制台应用程序，并添加以下`main`方法：

```java
public static void main(String[] args) {
Path path = Paths.get("/home/docs/../music/Space Machine A.mp3");
System.out.println("Absolute path: " + path.toAbsolutePath());
System.out.println("URI: " + path.toUri());
System.out.println("Normalized Path: " + path.normalize());
System.out.println("Normalized URI: " + path.normalize().toUri());
System.out.println();
path = Paths.get("/home/./music/ Robot Brain A.mp3");
System.out.println("Absolute path: " + path.toAbsolutePath());
System.out.println("URI: " + path.toUri());
System.out.println("Normalized Path: " + path.normalize());
System.out.println("Normalized URI: " + path.normalize().toUri());
}

```

1.  执行应用程序。您应该获得以下输出，尽管根目录可能会根据系统配置而有所不同：

**绝对路径：E:\home\docs\..\music\Space Machine A.mp3**

**URI：file:///E:/home/docs/../music/Space%20Machine%20A.mp3**

**规范化路径：\home\music\Space Machine A.mp3**

**规范化的 URI：file:///E:/home/music/Space%20Machine%20A.mp3**

**绝对路径：E:\home\.\music\ Robot Brain A.mp3**

URI：file:///E:/home/./music/%20Robot%20Brain%20A.mp3

**规范化路径：\home\music\ Robot Brain A.mp3**

**规范化的 URI：file:///E:/home/music/%20Robot%20Brain%20A.mp3**

## 它是如何工作的...

使用`Paths`类的`get`方法使用先前讨论过的冗余多余路径创建了两个路径。`get`方法后面的代码显示了绝对路径和 URI 等效项，以说明创建的实际路径。接下来，使用了`normalize`方法，然后与`toUri`方法链接，以进一步说明规范化过程。请注意，冗余和多余的路径元素已经消失。`toAbsolutePath`和`toUri`方法在*使用相对和绝对路径*示例中进行了讨论。

`normalize`方法不会检查文件或路径是否有效。该方法只是针对路径执行语法操作。如果符号链接是原始路径的一部分，则规范化路径可能不再有效。符号链接在*管理符号链接*示例中讨论。

## 还有更多...

`Path`类的`toRealPath`将返回表示文件实际路径的路径。它会检查路径是否有效，如果文件不存在，则会返回`java.nio.file.NoSuchFileException`。

修改先前的示例，使用`toRealPath`方法并显示不存在的文件，如下面的代码所示：

```java
try
Path path = Paths.get("/home/docs/../music/NonExistentFile.mp3");
System.out.println("Absolute path: " + path.toAbsolutePath());
System.out.println("Real path: " + path.toRealPath());
}
catch (IOException ex) {
System.out.println("The file does not exist!");
}

```

执行应用程序。结果应包含以下输出：

**绝对路径：\\Richard-pc\e\home\docs\..\music\NonExistentFile.mp3**

**文件不存在！**

`toRealPath`方法规范化路径。它还解析任何符号链接，尽管在此示例中没有符号链接。

## 另请参阅

`Path`对象的创建在*理解路径*配方中有所讨论。符号链接在*管理符号链接*配方中有所讨论。

# 使用路径解析来组合路径

`resolve`方法用于组合两个路径，其中一个包含根元素，另一个是部分路径。这在创建可能变化的路径时非常有用，例如在应用程序的安装中使用的路径。例如，可能有一个默认目录用于安装应用程序。但是，用户可能能够选择不同的目录或驱动器。使用`resolve`方法创建路径允许应用程序独立于实际安装目录进行配置。

## 准备工作

使用`resolve`方法涉及两个基本步骤：

+   创建一个使用根元素的`Path`对象

+   对此路径执行`resolve`方法，使用第二个部分路径

部分路径是指仅提供完整路径的一部分，并且不包含根元素。

## 如何做...

1.  创建一个新的应用程序。将以下`main`方法添加到其中：

```java
public static void main(String[] args) {
Path rootPath = Paths.get("/home/docs");
Path partialPath = Paths.get("users.txt");
Path resolvedPath = rootPath.resolve(partialPath);
System.out.println("rootPath: " + rootPath);
System.out.println("partialPath: " + partialPath);
System.out.println("resolvedPath: " + resolvedPath);
System.out.println("Resolved absolute path: " + resolvedPath.toAbsolutePath());
}

```

1.  执行代码。您应该得到以下输出：

**rootPath: \home\docs**

**partialPath: users.txt**

**resolvedPath: \home\docs\users.txt**

**解析的绝对路径：E:\home\docs\users.txt**

## 工作原理...

以下三条路径已创建：

+   `\home\docs：`这是根路径

+   `users.txt：`这是部分路径

+   `\home\docs\users.txt：`这是生成的解析路径

通过使用`partialPath`变量作为`resolve`方法的参数执行对`rootPath`变量的操作来创建解析路径。然后显示这些路径以及`resolvedPath`的绝对路径。绝对路径包括根目录，尽管这在您的系统上可能有所不同。

## 还有更多...

`resolve`方法是重载的，一个使用`String`参数，另一个使用`Path`参数。`resolve`方法也可能被误用。此外，还有一个`overloadedresolveSibling`方法，其工作方式类似于`resolve`方法，只是它会移除根路径的最后一个元素。这些问题在这里得到解决。

### 使用`String`参数与`resolve`方法

`resolve`方法是重载的，其中一个接受`String`参数。以下语句将实现与前面示例相同的结果：

```java
Path resolvedPath = rootPath.resolve("users.txt");

```

路径分隔符也可以使用如下：

```java
Path resolvedPath = rootPath.resolve("backup/users.txt");

```

使用这些语句与先前的代码会产生以下输出：

根路径：\home\docs

**partialPath: users.txt**

**resolvedPath: \home\docs\backup\users.txt**

**解析的绝对路径：E:\home\docs\backup\users.txt**

请注意，解析的路径不一定是有效路径，因为备份目录可能存在，也可能不存在。在*通过规范化路径来消除路径中的冗余*配方中，可以使用`toRealPath`方法来确定它是否有效。

### 错误使用`resolve`方法

`resolve`方法有三种用法，可能会导致意外行为：

+   根路径和部分路径的顺序不正确

+   使用部分路径两次

+   使用根路径两次

如果我们颠倒`resolve`方法的使用顺序，也就是将根路径应用于部分路径，那么只会返回根路径。下面的代码演示了这一点：

```java
Path resolvedPath = partialPath.resolve(rootPath);

```

当执行代码时，我们得到以下结果：

根路径：\home\docs

**partialPath: users.txt**

**resolvedPath: \home\docs**

**解析的绝对路径：E:\home\docs**

这里只返回根路径。部分路径不会附加到根路径上。如下面的代码所示，使用部分路径两次：

```java
Path resolvedPath = partialPath.resolve(partialPath);

```

将产生以下输出：

**rootPath: \home\docs**

**partialPath: users.txt**

**resolvedPath: users.txt\users.txt**

**解析的绝对路径：currentWorkingDIrectory\users.txt\users.txt**

请注意，解析的路径是不正确的，绝对路径使用了当前工作目录。如下所示，使用根路径两次：

```java
Path resolvedPath = rootPath.resolve(rootPath);

```

结果与以相反顺序使用路径时相同：

**rootPath: \home\docs**

**partialPath: users.txt**

**resolvedPath: \home\docs**

**解析的绝对路径：E:\home\docs**

每当绝对路径被用作`resolve`方法的参数时，该绝对路径将被返回。如果空路径被用作方法的参数，则根路径将被返回。

### 使用`resolveSibling`

`resolveSibling`方法是重载的，可以接受`String`或`Path`对象。使用`resolve`方法时，部分路径被附加到根路径的末尾。`resolveSibling`方法与`resolve`方法不同之处在于，在附加部分路径之前，根路径的最后一个元素被移除。考虑以下代码序列：

```java
Path rootPath = Paths.get("/home/music/");
resolvedPath = rootPath.resolve("tmp/Robot Brain A.mp3");
System.out.println("rootPath: " + rootPath);
System.out.println("resolvedPath: " + resolvedPath);
System.out.println();
resolvedPath = rootPath.resolveSibling("tmp/Robot Brain A.mp3");
System.out.println("rootPath: " + rootPath);
System.out.println("resolvedPath: " + resolvedPath);

```

当执行时，我们得到以下输出：

**rootPath: \home\music**

**resolvedPath: \home\music\tmp\Robot Brain A.mp3**

**rootPath: \home\music**

**resolvedPath: \home\tmp\Robot Brain A.mp3**

请注意，解析路径在存在`music`目录时与使用`resolveSibling`方法时不同。当使用`resolve`方法时，目录存在。当使用`resolveSibling`方法时，目录不存在。如果没有父路径，或者方法的参数是绝对路径，则返回传递给方法的参数。如果参数为空，则返回父目录。

## 另请参阅

`Path`对象的创建在*理解路径*配方中有所讨论。此外，`toRealPath`方法在*通过规范化路径来消除路径中的冗余*配方中有所解释。

# 在两个位置之间创建路径

相对化路径意味着基于另外两个路径创建一个路径，使得新路径表示从原始路径中的一个导航到另一个的方式。这种技术找到了从一个位置到另一个位置的相对路径。例如，第一个路径可以表示一个应用程序默认目录。第二个路径可以表示一个目标目录。从这些目录创建的相对路径可以促进对目标的操作。

## 准备工作

要使用`relativize`方法从一个路径到另一个路径创建新路径，我们需要执行以下操作：

1.  创建一个代表第一个路径的`Path`对象。

1.  创建一个代表第二个路径的`Path`对象。

1.  对第一个路径使用第二个路径作为参数应用`relativize`方法。

## 如何做...

1.  创建一个新的控制台应用程序，并使用以下`main`方法。该方法创建两个`Path`对象，并显示它们之间的相对路径如下：

```java
public static void main(String[] args) {
Path firstPath;
Path secondPath;
firstPath = Paths.get("music/Future Setting A.mp3");
secondPath = Paths.get("docs");
System.out.println("From firstPath to secondPath: " + firstPath.relativize(secondPath));
System.out.println("From secondPath to firstPath: " + secondPath.relativize(firstPath));
System.out.println();
firstPath = Paths.get("music/Future Setting A.mp3");
secondPath = Paths.get("music");
System.out.println("From firstPath to secondPath: " + firstPath.relativize(secondPath));
System.out.println("From secondPath to firstPath: " + secondPath.relativize(firstPath));
System.out.println();
firstPath = Paths.get("music/Future Setting A.mp3");
secondPath = Paths.get("docs/users.txt");
System.out.println("From firstPath to secondPath: " + firstPath.relativize(secondPath));
System.out.println("From secondPath to firstPath: " + secondPath.relativize(firstPath));
System.out.println();
}

```

1.  执行应用程序。您的结果应该类似于以下内容：

**从 firstPath 到 secondPath: ..\..\docs**

**从 secondPath 到 firstPath: ..\music\Future Setting A.mp3**

**从 firstPath 到 secondPath: ..**

**从 secondPath 到 firstPath: Future Setting A.mp3**

**从 firstPath 到 secondPath: ..\..\docs\users.txt**

**从 secondPath 到 firstPath: ..\..\music\Future Setting A.mp3**

## 工作原理...

在第一个例子中，从`Future Setting A.mp3`文件到`docs`目录创建了一个相对路径。假定`music`和`docs`目录是兄弟目录。`.`符号表示向上移动一个目录。本章的介绍说明了这个例子的假定目录结构。

第二个例子演示了从同一目录中创建路径。从`firstpath`到`secondPath`的路径实际上是一个潜在的错误。取决于如何使用它，我们可能会最终进入`music`目录上面的目录，因为返回的路径是`.`，表示向上移动一个目录级别。第三个例子与第一个例子类似，只是两个路径都包含文件名。

该方法创建的相对路径可能不是有效的路径。通过使用可能不存在的`tmp`目录来说明，如下所示：

```java
firstPath = Paths.get("music/Future Setting A.mp3");
secondPath = Paths.get("docs/tmp/users.txt");
System.out.println("From firstPath to secondPath: " + firstPath.relativize(secondPath));
System.out.println("From secondPath to firstPath: " + secondPath.relativize(firstPath));

```

输出应该如下所示：

**从 firstPath 到 secondPath: ..\..\docs\tmp\users.txt**

**从 secondPath 到 firstPath：..\..\..\music\Future Setting A.mp3**

## 还有更多...

还有三种情况需要考虑：

+   两条路径相等

+   一条路径包含根

+   两条路径都包含根

### 两条路径相等

当两条路径相等时，`relativize`方法将返回一个空路径，如下面的代码序列所示：

```java
firstPath = Paths.get("music/Future Setting A.mp3");
secondPath = Paths.get("music/Future Setting A.mp3");
System.out.println("From firstPath to secondPath: " + firstPath.relativize(secondPath));
System.out.println("From secondPath to firstPath: " + secondPath.relativize(firstPath));
System.out.println();

```

输出如下：

**从 firstPath 到 secondPath：**

**从 secondPath 到 firstPath：**

虽然这不一定是错误，但请注意它不返回一个经常用来表示当前目录的单个点。

### 一条路径包含根

如果两条路径中只有一条包含根元素，则可能无法构造相对路径。是否可能取决于系统。在下面的例子中，第一条路径包含根元素`c:`。

```java
firstPath = Paths.get("c:/music/Future Setting A.mp3");
secondPath = Paths.get("docs/users.txt");
System.out.println("From firstPath to secondPath: " + firstPath.relativize(secondPath));
System.out.println("From secondPath to firstPath: " + secondPath.relativize(firstPath));
System.out.println();

```

当在 Windows 7 上执行此代码序列时，我们得到以下输出：

**线程"main"中的异常"java.lang.IllegalArgumentException: 'other'是不同类型的路径**

**从 firstPath 到 secondPath：.**。

**从 secondPath 到 firstPath：Future Setting A.mp3**

**atsun.nio.fs.WindowsPath.relativize(WindowsPath.java:388)**

**atsun.nio.fs.WindowsPath.relativize(WindowsPath.java:44)**

**atpackt.RelativizePathExample.main(RelativizePathExample.java:25)**

**Java 结果：1**

注意输出中对**other**的引用。这指的是`relativize`方法的参数。

### 两条路径都包含根

`relativize`方法在两条路径都包含根元素时创建相对路径的能力也取决于系统。这种情况在下面的例子中有所说明：

```java
firstPath = Paths.get("c:/music/Future Setting A.mp3");
secondPath = Paths.get("c:/docs/users.txt");
System.out.println("From firstPath to secondPath: " + firstPath.relativize(secondPath));
System.out.println("From secondPath to firstPath: " + secondPath.relativize(firstPath));
System.out.println();

```

在 Windows 7 上执行时，我们得到以下输出：

**从 firstPath 到 secondPath：..\..\docs\users.txt**

**从 secondPath 到 firstPath：..\..\music\Future Setting A.mp3**

## 另请参阅

`Path`对象的创建在*理解路径*配方中讨论。符号链接的结果取决于系统，并在*管理符号链接*配方中进行了更深入的讨论。

# 在路径类型之间进行转换

`Path`接口表示文件系统中的路径。这个路径可能是有效的，也可能不是。有时我们可能想要使用路径的另一种表示。例如，可以使用文件的`URI`在大多数浏览器中加载文件。`toUri`方法提供了路径的这种表示。在这个示例中，我们还将看到如何获取`Path`对象的绝对路径和真实路径。

## 准备好了

有三种方法提供替代路径表示：

+   `toUri`方法返回`URI`表示

+   `toAbsolutePath`方法返回绝对路径

+   `toRealPath`方法返回真实路径

## 如何做...

1.  创建一个新的控制台应用程序。在`main`方法中，我们将使用之前的每种方法。将以下`main`方法添加到应用程序中：

```java
public static void main(String[] args) {
try {
Path path;
path = Paths.get("users.txt");
System.out.println("URI path: " + path.toUri());
System.out.println("Absolute path: " + path.toAbsolutePath());
System.out.println("Real path: " + path.toRealPath(LinkOption.NOFOLLOW_LINKS));
}
catch (IOException ex) {
Logger.getLogger(ConvertingPathsExample.class.getName()).log(Level.SEVERE, null, ex);
}
}

```

1.  如果尚未存在，请在应用程序的工作目录中添加一个`users.txt`文件。执行程序。您的输出应该类似于以下内容，除了此输出中的**..**应反映`users.txt`文件的位置：

**URI 路径：file:///.../ConvertingPathsExample/users.txt**

**绝对路径...\ConvertingPathsExample\users.txt**

**真实路径：...\ConvertingPathsExample\users.txt**

## 它是如何工作的...

一个`users.txt`文件被添加到 Java 应用程序的工作目录中。该文件应包含用户名列表。`get`方法返回表示此文件的`Path`对象。然后对该对象执行了三种方法。

`toUri`和`toAbsolutePath`方法按预期返回路径。返回的路径取决于应用程序的工作目录。`toRealPath`方法应该返回与`toAbsolutePath`方法相同的输出。这是预期的，因为`users.txt`文件不是作为符号链接创建的。如果这是一个符号链接，那么将显示代表文件实际路径的不同路径。

## 还有更多...

由于`Path`对象可能实际上并不代表文件，如果文件不存在，使用`toRealPath`方法可能会抛出`java.nio.file.NoSuchFileException`。使用一个无效的文件名，如下所示：

```java
path = Paths.get("invalidFileName.txt");

```

输出应该如下所示：

**URI 路径：file:///.../ConvertingPathsExample/invalidFileName.txt**

**绝对路径：...\ConvertingPathsExample\invalidFileName.txt**

**Sep 11, 2011 6:40:40 PM packt.ConvertingPathsExample main**

**严重：null**

**java.nio.file.NoSuchFileException: ...\ConvertingPathsExample\invalidFileName.txt**

请注意，`toUri`和`toAbsolutePath`方法无论指定的文件是否存在都可以工作。在我们想要使用这些方法的情况下，我们可以使用`Files`类的`exists`方法来测试文件是否存在。前面的代码序列已经修改为使用`exists`方法，如下所示：

```java
if(Files.exists(path)) {
System.out.println("Real path: " + path.toRealPath(LinkOption.NOFOLLOW_LINKS));
}
else {
System.out.println("The file does not exist");
}

```

`java.nio.fil.LinkOption`枚举是在 Java 7 中添加的。它用于指定是否应该跟随符号链接。

执行时，输出应如下所示：

**URI 路径：file:///.../ConvertingPathsExample/invalidFileName.txt**

**绝对路径：...\ConvertingPathsExample\invalidFileName.txt**

**文件不存在**

# 确定两个路径是否等效

有时可能需要比较路径。`Path`类允许您使用`equals`方法测试路径的相等性。您还可以使用`compareTo`方法使用`Comparable`接口的实现按字典顺序比较两个路径。最后，`isSameFile`方法可用于确定两个`Path`对象是否将定位到相同的文件。

## 准备工作

为了比较两个路径，您必须：

1.  创建一个代表第一个路径的`Path`对象。

1.  创建一个代表第二个路径的`Path`对象。

1.  根据需要对路径应用`equals, compareTo`或`isSameFile`方法。

## 如何做...

1.  创建一个新的控制台应用程序并添加一个`main`方法。声明三个`Path`对象变量，如`path1，path2`和`path3`。将前两个设置为相同的文件，第三个设置为不同的路径。所有三个文件必须存在。接下来调用三个比较方法：

```java
public class ComparingPathsExample {
public static void main(String[] args) {
Path path1 = null;
Path path2 = null;
Path path3 = null;
path1 = Paths.get("/home/docs/users.txt");
path2 = Paths.get("/home/docs/users.txt");
path3 = Paths.get("/home/music/Future Setting A.mp3");
testEquals(path1, path2);
testEquals(path1, path3);
testCompareTo(path1, path2);
testCompareTo(path1, path3);
testSameFile(path1, path2);
testSameFile(path1, path3);
}

```

1.  添加三个静态方法如下：

```java
private static void testEquals(Path path1, Path path2) {
if (path1.equals(path2)) {
System.out.printf("%s and %s are equal\n",
path1, path2);
}
else {
System.out.printf("%s and %s are NOT equal\n",
path1, path2);
}
}
private static void testCompareTo(Path path1, Path path2) {
if (path1.compareTo(path2) == 0) {
System.out.printf("%s and %s are identical\n",
path1, path2);
}
else {
System.out.printf("%s and %s are NOT identical\n",
path1, path2);
}
}
private static void testSameFile(Path path1, Path path2) {
try {
if (Files.isSameFile(path1, path2)) {
System.out.printf("%s and %s are the same file\n",
path1, path2);
}
else {
System.out.printf("%s and %s are NOT the same file\n",
path1, path2);
}
}
catch (IOException e) {
e.printStackTrace();
}
}

```

1.  执行应用程序。您的输出应该类似于以下内容：

**\home\docs\users.txt 和 \home\docs\users.txt 是相等的**

**\home\docs\users.txt 和 \home\music\Future Setting A.mp3 不相等**

\home\docs\users.txt 和 \home\docs\users.txt 是相同的

**\home\docs\users.txt 和 \home\music\Future Setting A.mp3 不相同**

**\home\docs\users.txt 和 \home\docs\users.txt 是相同的文件**

**\home\docs\users.txt 和 \home\music\Future Setting A.mp3 不是同一个文件**

## 它是如何工作的...

在`testEquals`方法中，我们确定了路径对象是否被视为相等。如果它们相等，`equals`方法将返回 true。但是，相等的定义是依赖于系统的。一些文件系统将使用大小写等因素来确定路径是否相等。

`testCompareTo`方法使用`compareTo`方法按字母顺序比较路径。如果路径相同，该方法返回零。如果路径小于参数，则该方法返回小于零的整数，如果路径按字典顺序跟随参数，则返回大于零的值。

`testSameFile`方法确定路径是否指向相同的文件。首先测试`Path`对象是否相同。如果是，则该方法将返回 true。如果`Path`对象不相等，则该方法确定路径是否指向相同的文件。如果`Path`对象是由不同的文件系统提供程序生成的，则该方法将返回 false。由于该方法可能引发`IOException`，因此使用了 try 块。

## 还有更多...

`equals`和`compareTo`方法将无法成功比较来自不同文件系统的路径。但是，只要文件位于同一文件系统上，所涉及的文件无需存在，文件系统也不会被访问。如果要测试的路径对象不相等，则`isSameFile`方法可能需要访问文件。在这种情况下，文件必须存在，否则该方法将返回 false。

## 另请参阅

`Files`类的`exists`和`notExists`方法可用于确定文件或目录是否存在。这在第三章的*获取文件和目录信息*中有所涵盖。

# 管理符号链接

符号链接用于创建对实际存在于不同目录中的文件的引用。在介绍中，详细列出了文件层次结构，其中`users.txt`文件在`docs`目录和`music`目录中分别列出。实际文件位于`docs`目录中。`music`目录中的`users.txt`文件是对真实文件的符号链接。对用户来说，它们看起来是不同的文件。实际上，它们是相同的。修改任一文件都会导致真实文件被更改。

从程序员的角度来看，我们经常想知道哪些文件是符号链接，哪些不是。在本教程中，我们将讨论 Java 7 中可用于处理符号链接的方法。重要的是要了解在与符号链接一起使用方法时方法的行为。

## 准备就绪

虽然几种方法可能根据`Path`对象是否表示符号链接而有所不同，但在本章中，只有`toRealPath，exists`和`notExists`方法接受可选的`LinkOption`枚举参数。此枚举只有一个元素：`NOFOLLOW_LINKS`。如果未使用该参数，则方法默认会跟随符号链接。

## 如何做...

1.  创建一个新的控制台应用程序。使用以下`main`方法，在其中创建代表真实和符号`users.txt`文件的几个`Path`对象。演示了本章中几个`Path-related`方法的行为。

```java
public static void main(String[] args) {
Path path1 = null;
Path path2 = null;
path1 = Paths.get("/home/docs/users.txt");
path2 = Paths.get("/home/music/users.txt");
System.out.println(Files.isSymbolicLink(path1));
System.out.println(Files.isSymbolicLink(path2));
try {
Path path = Paths.get("C:/home/./music/users.txt");
System.out.println("Normalized: " + path.normalize());
System.out.println("Absolute path: " + path.toAbsolutePath());
System.out.println("URI: " + path.toUri());
System.out.println("toRealPath (Do not follow links): " + path.toRealPath(LinkOption.NOFOLLOW_LINKS));
System.out.println("toRealPath: " + path.toRealPath());
Path firstPath = Paths.get("/home/music/users.txt");
Path secondPath = Paths.get("/docs/status.txt");
System.out.println("From firstPath to secondPath: " + firstPath.relativize(secondPath));
System.out.println("From secondPath to firstPath: " + secondPath.relativize(firstPath));
System.out.println("exists (Do not follow links): " + Files.exists(firstPath, LinkOption.NOFOLLOW_LINKS));
System.out.println("exists: " + Files.exists(firstPath));
System.out.println("notExists (Do not follow links): " + Files.notExists(firstPath, LinkOption.NOFOLLOW_LINKS));
System.out.println("notExists: " + Files.notExists(firstPath));
}
catch (IOException ex) {
Logger.getLogger(SymbolicLinkExample.class.getName()).log(Level.SEVERE, null, ex);
}
catch (InvalidPathException ex) {
System.out.println("Bad path: [" + ex.getInput() + "] at position " + ex.getIndex());
}
}

```

1.  这些方法的行为可能因基础操作系统而异。当代码在 Windows 平台上执行时，我们会得到以下输出：

**false**

**true**

**标准化：C：\ home \ music \ users.txt**

**绝对路径：C：\ home \。music \ users.txt**

**URI：file:///C:/home/./music/users.txt**

toRealPath（不要跟随链接）：C：\ home \ music \ users.txt

**toRealPath：C：\ home \ docs \ users.txt**

**从 firstPath 到 secondPath：..\..\..\docs\status.txt**

**从 secondPath 到 firstPath：..\..\home\music\users.txt**

**exists（不要跟随链接）：true**

**exists：true**

**notExists（不要跟随链接）：false**

**notExists：false**

## 它是如何工作的...

创建了`path1`和`path2`对象，分别引用了真实文件和符号链接。针对这些对象执行了`Files`类的`isSymbolicLink`方法，指示哪个路径引用了真实文件。

使用多余的点符号创建了`Path`对象。针对符号链接执行的`normalize`方法的结果返回了对符号链接的标准化路径。使用`toAbsolutePath`和`toUri`方法会返回对符号链接而不是真实文件的路径。

`toRealPath`方法具有可选的`LinkOption`参数。我们使用它来获取真实文件的路径。当您需要真实路径时，这个方法非常有用，通常其他方法执行符号链接时不会返回真实路径。

`firstPath`和`secondPath`对象被用来探索`relativize`方法如何与符号链接一起工作。在这些例子中，使用了符号链接。最后一组例子使用了`exists`和`notExists`方法。使用符号链接并不影响这些方法的结果。

## 另请参阅

符号文件的使用对其他文件系统方法的影响将在后续章节中讨论。
