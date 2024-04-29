# 第三章。获取文件和目录信息

在本章中，我们将涵盖以下内容：

+   确定文件内容类型

+   使用 getAttribute 方法逐个获取单个属性

+   获取文件属性的映射

+   获取文件和目录信息

+   确定操作系统对属性视图的支持

+   使用 BasicFileAttributeView 维护基本文件属性

+   使用 PosixFileAttributeView 维护 POSIX 文件属性

+   使用 DosFileAttributeView 维护 FAT 表属性

+   使用 FileOwnerAttributeView 维护文件所有权属性

+   使用 AclFileAttributeView 维护文件的 ACL

+   使用 UserDefinedFileAttributeView 维护用户定义的文件属性

# 介绍

许多应用程序需要访问文件和目录信息。这些信息包括文件是否可以执行，文件的大小，文件的所有者，甚至其内容类型等属性。在本章中，我们将研究获取有关文件或目录信息的各种技术。我们根据所需的访问类型组织了配方。

使用`java.nio.file.Files`类获取文件和目录信息的五种一般方法如下：

+   使用`Files`类的特定方法，如`isDirectory`方法，逐个获取单个属性。这在*获取文件和目录信息*配方中有详细说明。

+   使用`Files`类的`getAttribute`方法逐个获取单个属性。这在*使用 getAttribute 方法逐个获取单个属性*配方中有详细说明。

+   使用`readAttributes`方法返回使用`String`指定要返回的属性的映射。这在*获取文件属性的映射*配方中有解释。

+   使用`readAttributes`方法与`BasicFileAttributes`派生类返回该属性集的属性类。这在*使用 BasicFileAttributeView 维护基本文件属性*配方中有详细说明。

+   使用`getFileAttributes`方法返回提供对特定属性集的访问的视图。这也在*使用 BasicFileAttributeView 方法维护基本文件属性*配方中有详细说明。它在配方的*还有更多..*部分中找到。

通过几种方法支持对属性的动态访问，并允许开发人员使用`String`指定属性。`Files`类的`getAttribute`方法代表了这种方法。

Java 7 引入了一些基于文件视图的接口。视图只是关于文件或目录的信息的一种组织方式。例如，`AclFileAttributeView`提供了与文件的**访问控制列表**（**ACL**）相关的方法。`FileAttributeView`接口是提供特定类型文件信息的其他接口的基接口。`java.nio.file.attribute`包中的子接口包括以下内容：

+   `AclFileAttributeView：`用于维护文件的 ACL 和所有权属性

+   `BasicFileAttributeView：`用于访问有关文件的基本信息并设置与时间相关的属性

+   `DosFileAttributeView：`设计用于与传统**磁盘操作系统**（**DOS**）文件属性一起使用

+   `FileOwnerAttributeView：`用于维护文件的所有权

+   `PosixFileAttributeView：`支持**便携式操作系统接口**（**POSIX**）属性

+   `UserDefinedFileAttributeView：`支持文件的用户定义属性

视图之间的关系如下所示：

### 注意

低级接口继承自它们上面的接口。

![介绍](img/5627_03_01.jpg)

`readAttributes` 方法的第二个参数指定要返回的属性类型。支持三个属性接口，它们的关系如下图所示。这些接口提供了访问它们对应的视图接口的方法：

![Introduction](img/5627_03_02.jpg)

每个视图都有一个专门的配方。这里不讨论 `FileStoreAttributeView`，但在 第四章 的 *管理文件和目录* 中有相关内容。

本章中示例使用的文件和目录结构在 第二章 的介绍中有描述，*使用路径定位文件和目录*。

# 确定文件内容类型

文件的类型通常可以从其扩展名推断出来。但这可能会误导，具有相同扩展名的文件可能包含不同类型的数据。`Files` 类的 `probeContentType` 方法用于确定文件的内容类型（如果可能）。当应用程序需要一些指示文件内容以便处理时，这是很有用的。

## 准备工作

为了确定内容类型，需要完成以下步骤：

1.  获取代表文件的 `Path` 对象。

1.  使用 `Path` 对象作为 `probeContentType` 方法的参数。

1.  使用结果处理文件。

## 操作步骤...

1.  创建一个新的控制台应用程序。将三种不同类型的文件添加到 `/home/docs` 目录中。使用以下内容作为 `main` 方法。虽然你可以使用任何你选择的文件，但本示例使用了一个文本文件，一个 Word 文档和一个可执行文件：

```java
public static void main(String[] args) throws Exception {
displayContentType("/home/docs/users.txt");
displayContentType("/home/docs/Chapter 2.doc");
displayContentType("/home/docs/java.exe");
}
static void displayContentType(String pathText) throws Exception {
Path path = Paths.get(pathText);
String type = Files.probeContentType(path);
System.out.println(type);
}

```

1.  执行应用程序。你的输出应该如下所示。返回的类型取决于你使用的实际文件：

**text/plain**

**application/msword**

**application/x-msdownload**

## 工作原理...

创建了一个 `java.nio.file.Path` 变量，并分配给了三个不同的文件。对每个文件执行了 `Files` 类的 `probeContentPath` 方法。返回的结果是一个 `String`，用于说明目的。`probeContentType` 方法会抛出一个 `java.io.IOException`，我们通过让 `displayConentType` 方法和 `main` 方法抛出一个基类异常来处理这个异常。`probeContentPath` 方法也可能会抛出一个 `java.lang.SecurityException`，但你不需要处理它。

在本示例中使用的文件中，第一个文件是一个文本文件。返回的类型是 **text/plain**。另外两个是一个 Word 文档和可执行文件 `java.exe`。返回的类型分别是 **application/msword** 和 **application/x-msdownload**。

## 还有更多...

该方法的结果是一个 `String`，由 **多用途互联网邮件扩展** (**MIME**)：**RFC 2045：多用途互联网邮件扩展（MIME）第一部分：互联网消息正文的格式** 定义。这允许使用 RFC 2045 语法规范解析 `String`。如果无法识别内容类型，则返回 null。

MIME 类型由类型和子类型以及一个或多个可选参数组成。类型和子类型之间使用斜杠分隔。在前面的输出中，文本文档的类型是 text，子类型是 plain。另外两种类型都是 application 类型，但子类型不同。以 x- 开头的子类型是非标准的。

`probeContentType`方法的实现取决于系统。该方法将使用`java.nio.file.spi.FileTypeDetector`实现来确定内容类型。它可能检查文件名或可能访问文件属性以确定文件内容类型。大多数操作系统将维护文件探测器列表。从此列表中加载并用于确定文件类型。`FileTypeDetector`类没有扩展，并且目前无法确定哪些文件探测器可用。

# 使用`getAttribute`方法一次获取一个属性

如果您有兴趣获取单个文件属性，并且知道属性的名称，则`Files`类的`getAttribute`方法简单且易于使用。它将返回基于表示属性的`String`的文件信息。本食谱的第一部分说明了`getAttribute`方法的简单用法。其他可用的属性列在本食谱的*更多内容*部分中。

## 准备就绪

获取单个文件属性值：

1.  创建一个表示感兴趣的文件的`Path`对象。

1.  将此对象用作`getAttribute`方法的第一个参数。

1.  使用包含属性名称的`String`作为方法的第二个参数。

## 如何做...

1.  创建一个新的控制台应用程序并使用以下`main`方法。在此方法中，我们确定文件的大小如下：

```java
public static void main(String[] args) {
try {
Path path = FileSystems.getDefault().getPath("/home/docs/users.txt");
System.out.println(Files.getAttribute(path, "size"));
}
catch (IOException ex) {
System.out.println("IOException");
}
}

```

1.  输出将如下所示，并将取决于所使用文件的实际大小：

**30**

## 它是如何工作的...

创建了一个表示`users.txt`文件的`Path`。然后将此路径用作`Files`类的`getAttribute`方法的第一个参数。执行代码时，将显示文件的大小。

## 更多内容...

`Files`类的`getAttribute`方法具有以下三个参数：

+   一个表示文件的`Path`对象

+   包含属性名称的`String`

+   在处理符号文件时使用的可选`LinkOption`

以下表格列出了可以与此方法一起使用的有效属性名称：

| 属性名称 | 数据类型 |
| --- | --- |
| `lastModifiedTime` | FileTime |
| `lastAccessTime` | FileTime |
| `creationTime` | FileTime |
| `size` | 长整型 |
| `isRegularFile` | 布尔值 |
| `isDirectory` | 布尔值 |
| `isSymbolicLink` | 布尔值 |
| `isOther` | 布尔值 |
| `fileKey` | 对象 |

如果使用无效的名称，则会发生运行时错误。这是这种方法的主要弱点。例如，如果名称拼写错误，我们将收到运行时错误。此方法如下所示，指定的属性在属性`String`末尾有一个额外的*s*：

```java
System.out.println(Files.getAttribute(path, "sizes"));

```

当应用程序执行时，您应该获得类似以下的结果：

**线程"main"中的异常 java.lang.IllegalArgumentException：未识别'sizes'**

**在 sun.nio.fs.AbstractBasicFileAttributeView$AttributesBuilder.<init>(AbstractBasicFile AttributeView.java:102)**

**在 sun.nio.fs.AbstractBasicFileAttributeView$AttributesBuilder.create(AbstractBasicFileAttributeView.java:112)**

**在 sun.nio.fs.AbstractBasicFileAttributeView.readAttributes(AbstractBasicFileAttributeView.java:166)**

**在 sun.nio.fs.AbstractFileSystemProvider.readAttributes(AbstractFileSystemProvider.java:92)**

**在 java.nio.file.Files.readAttributes(Files.java:1896)**

**在 java.nio.file.Files.getAttribute(Files.java:1801)**

**在 packt.SingleAttributeExample.main(SingleAttributeExample.java:15)**

**Java 结果：1**

可以按照*获取文件属性映射*食谱中的描述获取文件属性列表。这可以用来避免使用无效名称。

# 获取文件属性映射

访问文件属性的另一种方法是使用`Files`类的`readAttributes`方法。该方法有两个重载版本，在第二个参数和返回的数据类型上有所不同。在本示例中，我们将探讨返回`java.util.Map`对象的版本，因为它允许在返回的属性上更灵活。该方法的第二个版本在一系列食谱中讨论，每个食谱都专门讨论一类属性。

## 准备就绪

要获取`Map`对象形式的属性列表，需要执行以下步骤：

1.  创建一个表示文件的`Path`对象。

1.  对`Files`类应用静态的`readAttributes`方法。

1.  指定其参数的值：

+   表示感兴趣文件的`Path`对象

+   表示要返回的属性的`String`参数

+   可选的第三个参数，指定是否应该跟踪符号链接

## 如何做...

1.  创建一个新的控制台应用程序。使用以下`main`方法：

```java
public static void main(String[] args) throws Exception {
Path path = Paths.getPath("/home/docs/users.txt");
try {
Map<String, Object> attrsMap = Files.readAttributes(path, "*");
Set<String> keys = attrsMap.keySet();
for(String attribute : keys) {
out.println(attribute + ": "
+ Files.getAttribute(path, attribute));
}
}
}

```

1.  执行应用程序。您的输出应该类似于以下内容：

**lastModifiedTime: 2011-09-06T01:26:56.501665Z**

**fileKey: null**

**isDirectory: false**

**lastAccessTime: 2011-09-06T21:14:11.214057Z**

**isOther: false**

**isSymbolicLink: false**

**isRegularFile: true**

**creationTime: 2011-09-06T21:14:11.214057Z**

**大小：30**

## 它是如何工作的...

示例中使用了`docs`目录中的`users.txt`文件。声明了一个键类型为`String`，值类型为`Object`的`Map`对象，然后为其赋予了`readAttributes`方法的值。使用`Map`接口的`keySet`方法创建了一个`java.util.Set`对象。这使我们可以访问`Map`的键和值。在 for each 循环中，将集合的每个成员用作`getAttribute`方法的参数。文件的相应属性和其值将被显示。`getAttribute`方法在*使用 getAttribute 方法逐个获取属性*食谱中有解释。

在这个例子中，我们使用字符串字面值`"*"`作为第二个参数。这个值指示方法返回文件的所有可用属性。正如我们很快将看到的，其他字符串值可以用来获得不同的结果。

`readAttributes`方法是一个原子文件系统操作。默认情况下，会跟踪符号链接。要指示该方法不要跟踪符号链接，使用`java.nio.file`包的`LinkOption.NOFOLLOW_LINKS`枚举常量，如下所示：

```java
Map<String, Object> attrsMap = Files.readAttributes(path, "*", LinkOption.NOFOLLOW_LINKS);

```

## 还有更多...

该方法的有趣之处在于它的第二个参数。`String`参数的语法包括一个可选的`viewName`，后面跟着一个冒号，然后是属性列表。`viewName`通常是以下之一：

+   acl

+   基本

+   所有者

+   用户

+   dos

+   posix

每个`viewNames`对应于一个视图接口的名称。

属性列表是一个逗号分隔的属性列表。属性列表可以包含零个或多个元素。如果使用无效的元素名称，则会被忽略。使用星号将返回与该`viewName`关联的所有属性。如果不包括`viewName`，则会返回所有基本文件属性，就像前面所示的那样。

以基本视图为例，以下表格说明了我们如何选择要返回的属性：

| String | 返回的属性 |
| --- | --- |
| `"*"` | 所有基本文件属性 |
| `"basic:*"` | 所有基本文件属性 |
| `"basic:isDirectory,lastAccessTime`" | 仅`isDirectory`和`lastAccessTime`属性 |
| `"isDirectory,lastAccessTime`" | 仅`isDirectory`和`lastAccessTime`属性 |
| `""` | 无 - 会生成`java.lang.IllegalArgumentException` |

`String`属性在除基本视图以外的视图中使用方式相同。

### 提示

属性`String`中不能有嵌入的空格。例如，`String, "basic:isDirectory, lastAccessTime"`，逗号后面有一个空格会导致`IllegalArgumentException`。

# 获取文件和目录信息

经常需要检索有关文件或目录的基本信息。本教程将介绍`java.nio.file.Files`类如何提供直接支持。这些方法仅提供对文件和目录信息的部分访问，并以`isRegularFile`等方法为代表。此类方法的列表可在本教程的*更多信息*部分找到。

## 准备就绪

要使用`Files`类的方法显示信息很容易，因为这些方法大多数（如果不是全部）都是静态的。这意味着这些方法可以轻松地针对`Files`类名称执行。要使用这种技术：

1.  创建一个表示文件或目录的`Path`对象。

1.  将`Path`对象用作适当的`Files`类方法的参数。

## 如何做...

1.  为了演示如何获取文件属性，我们将开发一个方法来显示文件的属性。创建一个包含以下`main`方法的新控制台应用程序。在该方法中，我们创建一个文件的引用，然后调用`displayFileAttribute`方法。它使用几种方法来显示有关路径的信息，如下所示：

```java
public static void main(String[] args) throws Exception {
Path path = FileSystems.getDefault().getPath("/home/docs/users.txt");
displayFileAttributes(path);
}
private static void displayFileAttributes(Path path) throws Exception {
String format =
"Exists: %s %n"
+ "notExists: %s %n"
+ "Directory: %s %n"
+ "Regular: %s %n"
+ "Executable: %s %n"
+ "Readable: %s %n"
+ "Writable: %s %n"
+ "Hidden: %s %n"
+ "Symbolic: %s %n"
+ "Last Modified Date: %s %n"
+ "Size: %s %n";
System.out.printf(format,
Files.exists(path, LinkOption.NOFOLLOW_LINKS),
Files.notExists(path, LinkOption.NOFOLLOW_LINKS),
Files.isDirectory(path, LinkOption.NOFOLLOW_LINKS),
Files.isRegularFile(path, LinkOption.NOFOLLOW_LINKS),
Files.isExecutable(path),
Files.isReadable(path),
Files.isWritable(path),
Files.isHidden(path),
Files.isSymbolicLink(path),
Files.getLastModifiedTime(path, LinkOption.NOFOLLOW_LINKS),
Files.size(path));
}

```

1.  执行程序。您的输出应如下所示：

**存在：true**

**不存在：false**

**目录：false**

**常规：true**

**可执行：true**

**可读：true**

**可写：true**

**隐藏：false**

**符号链接：false**

**上次修改日期：2011-10-20T03:18:20.338139Z**

**大小：29**

## 它是如何工作的...

创建了指向`users.txt`文件的`Path`。然后将此`Path`对象传递给`displayFileAttribute`方法，该方法显示了文件的许多属性。返回这些属性的方法在以下表格中进行了总结：

| 方法 | 描述 |
| --- | --- |
| `exists` | 如果文件存在则返回`true` |
| `notExists` | 如果文件不存在则返回`true` |
| `isDirectory` | 如果路径表示目录则返回`true` |
| `isRegularFile` | 如果路径表示常规文件则返回`true` |
| `isExecutable` | 如果文件可执行则返回`true` |
| `isReadable` | 如果文件可读则返回`true` |
| `isWritable` | 如果文件可写则返回`true` |
| `isHidden` | 如果文件是隐藏的且对非特权用户不可见则返回`true` |
| `isSymbolicLink` | 如果文件是符号链接则返回`true` |
| `getLastModifiedTime` | 返回文件上次修改的时间 |
| `size` | 返回文件的大小 |

其中几种方法具有第二个参数，指定如何处理符号链接。当存在`LinkOption.NOFOLLOW_LINKS`时，符号链接不会被跟踪。第二个参数是可选的。如果省略，则不会跟踪符号链接。符号链接在第二章的*使用路径定位文件和目录*中的*管理符号链接*教程中进行了讨论。

## 更多信息...

以下表格总结了抛出的异常以及方法是否为非原子操作。如果调用线程无权读取文件，则可能会抛出`SecurityException`。 

### 注意

当一个方法被称为**非原子**时，这意味着其他文件系统操作可能会与该方法同时执行。非原子操作可能导致不一致的结果。也就是说，在这些方法执行时，可能会导致对方法目标的并发操作可能修改文件的状态。在使用这些方法时应考虑到这一点。

这些方法标记为过时的结果在返回时不一定有效。也就是说，不能保证任何后续访问都会成功，因为文件可能已被删除或以其他方式修改。

被指定为**无法确定**的方法表示，如果无法确定结果，则可能返回 `false`。例如，如果 `exists` 方法无法确定文件是否存在，则会返回 `false`。它可能存在，但该方法无法确定它是否存在：

| 方法 | SecurityException | IOException | 非原子 | 过时 | 无法确定 |
| --- | --- | --- | --- | --- | --- |
| `exists` | 是 |   |   | 是 | 是 |
| `notExists` | 是 |   |   | 是 | 是 |
| `isDirectory` | 是 |   |   |   | 是 |
| `isRegularFile` | 是 |   |   |   | 是 |
| `isExecutable` | 是 |   | 是 | 是 | 是 |
| `isReadable` | 是 |   | 是 | 是 | 是 |
| `isWritable` | 是 |   | 是 | 是 | 是 |
| `isHidden` | 是 | 是 |   |   |   |
| `isSymbolicLink` | 是 |   |   |   | 是 |
| `getLastModifiedTime` | 是 | 是 |   |   |   |
| `size` | 是 | 是 |   |   |   |

请注意，`notExists` 方法不是 `exists` 方法的反义词。使用任一方法，可能无法确定文件是否存在。在这种情况下，两种方法都将返回 `false`。

`isRegularFile` 确定文件是否为常规文件。如果 `isDirectory, isSymbolicLink` 和 `isRegularFile` 方法返回 `false`，则可能是因为：

+   它不是这些类型之一

+   如果文件不存在或

+   如果无法确定它是文件还是目录

对于这些方法，它们在 `BasicFileAttributes` 接口中对应的方法可能会提供更好的结果。这些方法在 *使用 BasicFileAttributeView 维护基本文件属性* 部分中有介绍。

`isExecutable` 方法检查文件是否存在，以及 JVM 是否有执行文件的访问权限。如果文件是一个目录，则该方法确定 JVM 是否有足够的权限来搜索该目录。如果：

+   文件不存在

+   文件不可执行

+   如果无法确定是否可执行

隐藏的含义取决于系统。在 UNIX 系统上，如果文件名以句点开头，则文件是隐藏的。在 Windows 上，如果设置了 DOS 隐藏属性，则文件是隐藏的。

# 确定操作系统对属性视图的支持

操作系统可能不支持 Java 中的所有属性视图。有三种基本技术可以确定支持哪些视图。知道支持哪些视图可以让开发人员避免在尝试使用不受支持的视图时可能发生的异常。

## 准备工作

这三种技术包括使用：

+   使用 `java.nio.file.FileSystem` 类的 `supportedFileAttributeViews` 方法返回一个包含所有支持的视图的集合。

+   使用 `java.nio.file.FileStore` 类的 `supportsFileAttributeView` 方法和一个类参数。如果该类受支持，则该方法将返回 `true`。

+   使用 `FileStore` 类的 `supportsFileAttributeView` 方法和一个 `String` 参数。如果该 `String` 表示的类受支持，则该方法将返回 `true`。

第一种方法是最简单的，将首先进行说明。

## 如何做...

1.  创建一个新的控制台应用程序，其中包含以下 `main` 方法。在这个方法中，我们将显示当前系统支持的所有视图，如下所示：

```java
public static void main(String[] args)
Path path = Paths.get("C:/home/docs/users.txt");
FileSystem fileSystem = path.getFileSystem();
Set<String> supportedViews = fileSystem.supportedFileAttributeViews();
for(String view : supportedViews) {
System.out.println(view);
}
}

```

1.  当应用在 Windows 7 系统上执行时，应该会得到以下输出：

**acl**

**basic**

**owner**

**user**

**dos**

1.  当应用在 Ubuntu 10.10 版本下执行时，应该会得到以下输出：

**basic**

**owner**

**user**

**unix**

**dos**

**posix**

请注意，**acl**视图不受支持，而**unix**和**posix**视图受支持。在 Java 7 发布版中没有`UnixFileAttributeView`。但是，该接口可以作为 JSR203-backport 项目的一部分找到。

## 它是如何工作的...

为`users.txt`文件创建了一个`Path`对象。接下来使用`getFileSystem`方法获取了该`Path`的文件系统。`FileSystem`类具有`supportedFileAttributeViews`方法，该方法返回一个表示支持的视图的字符串集合。然后使用 for each 循环显示每个字符串值。

## 还有更多...

还有另外两种方法可以用来确定支持哪些视图：

+   使用带有类参数的`supportsFileAttributeView`方法

+   使用带有`String`参数的`supportsFileAttributeView`方法

这两种技术非常相似。它们都允许您测试特定的视图。

### 使用带有类参数的 supportsFileAttributeView 方法

重载的`supportsFileAttributeView`方法接受表示所讨论的视图的类对象。将以下代码添加到上一个示例的`main`方法中。在这段代码中，我们确定支持哪些视图：

```java
try {
FileStore fileStore = Files.getFileStore(path);
System.out.println("FileAttributeView supported: " + fileStore.supportsFileAttributeView(
FileAttributeView.class));
System.out.println("BasicFileAttributeView supported: " + fileStore.supportsFileAttributeView(
BasicFileAttributeView.class));
System.out.println("FileOwnerAttributeView supported: " + fileStore.supportsFileAttributeView(
FileOwnerAttributeView.class));
System.out.println("AclFileAttributeView supported: " + fileStore.supportsFileAttributeView(
AclFileAttributeView.class));
System.out.println("PosixFileAttributeView supported: " + fileStore.supportsFileAttributeView(
PosixFileAttributeView.class));
System.out.println("UserDefinedFileAttributeView supported: " + fileStore.supportsFileAttributeView(
UserDefinedFileAttributeView.class));
System.out.println("DosFileAttributeView supported: " + fileStore.supportsFileAttributeView(
DosFileAttributeView.class));
}
catch (IOException ex) {
System.out.println("Attribute view not supported");
}

```

在 Windows 7 机器上执行时，您应该获得以下输出：

**FileAttributeView supported: false**

**BasicFileAttributeView supported: true**

**FileOwnerAttributeView supported: true**

**AclFileAttributeView supported: true**

**PosixFileAttributeView supported: false**

**UserDefinedFileAttributeView supported: true**

**DosFileAttributeView supported: true**

### 使用带有`String`参数的`supportsFileAttributeView`方法

重载的`supportsFileAttributeView`方法接受一个`String`对象的工作方式类似。将以下代码添加到`main`方法的 try 块中：

```java
System.out.println("FileAttributeView supported: " + fileStore.supportsFileAttributeView(
"file"));
System.out.println("BasicFileAttributeView supported: " + fileStore.supportsFileAttributeView(
"basic"));
System.out.println("FileOwnerAttributeView supported: " + fileStore.supportsFileAttributeView(
"owner"));
System.out.println("AclFileAttributeView supported: " + fileStore.supportsFileAttributeView(
"acl"));
System.out.println("PosixFileAttributeView supported: " + fileStore.supportsFileAttributeView(
"posix"));
System.out.println("UserDefinedFileAttributeView supported: " + fileStore.supportsFileAttributeView(
"user"));
System.out.println("DosFileAttributeView supported: " + fileStore.supportsFileAttributeView(
"dos"));

```

在 Windows 7 平台上执行时，您应该获得以下输出：

**FileAttributeView supported: false**

**BasicFileAttributeView supported: true**

**FileOwnerAttributeView supported: true**

**AclFileAttributeView supported: true**

**PosixFileAttributeView supported: false**

**UserDefinedFileAttributeView supported: true**

**DosFileAttributeView supported: true**

# 使用 BasicFileAttributeView 维护基本文件属性

`java.nio.file.attribute.BasicFileAttributeView`提供了一系列方法，用于获取有关文件的基本信息，例如其创建时间和大小。该视图具有一个`readAttributes`方法，该方法返回一个`BasicFileAttributes`对象。`BasicFileAttributes`接口具有几种用于访问文件属性的方法。该视图提供了一种获取文件信息的替代方法，而不是由`Files`类支持的方法。该方法的结果有时可能比`Files`类的结果更可靠。

## 准备工作

有两种方法可以获取`BasicFileAttributes`对象。第一种方法是使用`readAttributes`方法，该方法使用`BasicFileAttributes.class`作为第二个参数。第二种方法使用`getFileAttributeView`方法，并在本章的*更多内容..*部分中进行了探讨。

`Files`类的`readAttributes`方法最容易使用：

1.  将表示感兴趣的文件的`Path`对象用作第一个参数。

1.  将`BasicFileAttributes.class`用作第二个参数。

1.  使用返回的`BasicFileAttributes`对象方法来访问文件属性。

这种基本方法用于本章中所示的其他视图。只有属性视图类不同。

## 操作步骤...

1.  创建一个新的控制台应用程序。使用以下`main`方法。在该方法中，我们创建了一个`BasicFileAttributes`对象，并使用其方法来显示有关文件的信息：

```java
public static void main(String[] args) {
Path path
= FileSystems.getDefault().getPath("/home/docs/users.txt");
try {
BasicFileAttributes attributes = Files.readAttributes(path, BasicFileAttributes.class);
System.out.println("Creation Time: " + attributes.creationTime());
System.out.println("Last Accessed Time: " + attributes.lastAccessTime());
System.out.println("Last Modified Time: " + attributes.lastModifiedTime());
System.out.println("File Key: " + attributes.fileKey());
System.out.println("Directory: " + attributes.isDirectory());
System.out.println("Other Type of File: " + attributes.isOther());
System.out.println("Regular File: " + attributes.isRegularFile());
System.out.println("Symbolic File: " + attributes.isSymbolicLink());
System.out.println("Size: " + attributes.size());
}
catch (IOException ex) {
System.out.println("Attribute error");
}
}

```

1.  执行应用程序。您的输出应该类似于以下内容：

**Creation Time: 2011-09-06T21:14:11.214057Z**

**Last Accessed Time: 2011-09-06T21:14:11.214057Z**

**Last Modified Time: 2011-09-06T01:26:56.501665Z**

**文件键：null**

**目录：false**

**其他类型的文件：false**

**常规文件：true**

**符号文件：false**

**大小：30**

## 它是如何工作的...

首先，我们创建了一个代表`users.txt`文件的`Path`对象。接下来，我们使用`Files`类的`readAttributes`方法获取了一个`BasicFileAttributes`对象。该方法的第一个参数是一个`Path`对象。第二个参数指定了我们想要返回的对象类型。在这种情况下，它是一个`BasicFileAttributes.class`对象。

然后是一系列打印语句，显示有关文件的特定属性信息。`readAttributes`方法检索文件的所有基本属性。由于它可能会抛出`IOException`，代码序列被包含在 try 块中。

大多数`BasicFileAttributes`接口方法很容易理解，但有一些需要进一步解释。首先，如果`isOther`方法返回`true`，这意味着文件不是常规文件、目录或符号链接。此外，尽管文件大小以字节为单位，但由于文件压缩和稀疏文件的实现等问题，实际大小可能会有所不同。如果文件不是常规文件，则返回值的含义取决于系统。

`fileKey`方法返回一个唯一标识该文件的对象。在 UNIX 中，设备 ID 或 inode 用于此目的。如果文件系统及其文件发生更改，文件键不一定是唯一的。它们可以使用`equals`方法进行比较，并且可以用于集合。再次强调的是，假设文件系统没有以影响文件键的方式发生更改。两个文件的比较在第二章的*确定两个路径是否等效*中有所涉及，*使用路径定位文件和目录*。

## 还有更多...

获取对象的另一种方法是使用`Files`类的`getFileAttributeView`方法。它根据第二个参数返回一个基于`AttributeView`的派生对象。要获取`BasicFileAttributeView`对象的实例：

1.  使用代表感兴趣的文件的`Path`对象作为第一个参数。

1.  使用`BasicFileAttributeView`作为第二个参数。

不要使用以下语句：

```java
BasicFileAttributes attributes = Files.readAttributes(path, BasicFileAttributes.class);

```

我们可以用以下代码序列替换它：

```java
BasicFileAttributeView view = Files.getFileAttributeView(path, BasicFileAttributeView.class);
BasicFileAttributes attributes = view.readAttributes();

```

使用`getFileAttributeView`方法返回`BasicFileAttributeView`对象。然后`readAttributes`方法返回`BasicFileAttributes`对象。这种方法更长，但现在我们可以访问另外三种方法，如下所示：

+   `name：`这返回属性视图的名称

+   `readAttributes：`这返回一个`BasicFileAttributes`对象

+   `setTimes：`这用于设置文件的时间属性

1.  然后我们使用如下所示的`name`方法：

```java
System.out.println("Name: " + view.name());

```

这导致了以下输出：

**名称：basic**

然而，这并没有为我们提供太多有用的信息。`setTimes`方法在第四章的*设置文件或目录的时间相关属性*中有所说明，*管理文件和目录*。

# 使用 PosixFileAttributeView 维护 POSIX 文件属性

许多操作系统支持**可移植操作系统接口**（**POSIX**）标准。这提供了一种更便携的方式来编写可以在不同操作系统之间移植的应用程序。Java 7 支持使用`java.nio.file.attribute.PosixFileAttributeView`接口访问文件属性。 

并非所有操作系统都支持 POSIX 标准。*确定操作系统是否支持属性视图*的示例说明了如何确定特定操作系统是否支持 POSIX。

## 准备工作

为了获取文件或目录的 POSIX 属性，我们需要执行以下操作：

1.  创建一个代表感兴趣的文件或目录的`Path`对象。

1.  使用`getFileAttributeView`方法获取`PosixFileAttributeView`接口的实例。

1.  使用`readAttributes`方法获取一组属性。

## 如何做...

1.  创建一个新的控制台应用程序。使用以下`main`方法。在此方法中，我们获取`users.txt`文件的属性如下：

```java
public static void main(String[] args) throws Exception {
Path path = Paths.get("home/docs/users.txt");
FileSystem fileSystem = path.getFileSystem();
PosixFileAttributeView view = Files.getFileAttributeView(path, PosixFileAttributeView.class);
PosixFileAttributes attributes = view.readAttributes();
System.out.println("Group: " + attributes.group());
System.out.println("Owner: " + attributes.owner().getName());
Set<PosixFilePermission> permissions = attributes.permissions();
for(PosixFilePermission permission : permissions) {
System.out.print(permission.name() + " ");
}
}

```

1.  执行应用程序。您的输出应如下所示。所有者名称可能会有所不同。在这种情况下，它是**richard：**

**组：richard**

**所有者：richard**

**OWNER_READ OWNER_WRITE OTHERS_READ GROUP_READ**

## 它是如何工作的...

为`users.txt`文件创建了一个`Path`对象。这被用作`Files`类的`getFileAttributeView`方法的第一个参数。第二个参数是`PosixFileAttributeView.class`。返回了一个`PosixFileAttributeView`对象。

接下来，使用`readAttributes`方法获取了`PosixFileAttributes`接口的实例。使用`group`和`getName`方法显示了文件的组和所有者。权限方法返回了一组`PosixFilePermission`枚举。这些枚举表示分配给文件的权限。

## 还有更多...

`PosixFileAttributes`接口扩展了`java.nio.file.attribute.BasicFileAttributes`接口，因此可以访问其所有方法。`PosixFileAttributeView`接口扩展了`java.nio.file.attribute.FileOwnerAttributeView`和`BasicFileAttributeView`接口，并继承了它们的方法。

`PosixFileAttributeView`接口具有`setGroup`方法，可用于配置文件的组所有者。可以使用`setPermissions`方法维护文件的权限。在第四章*管理文件和目录*中讨论了维护文件权限的*管理 POSIX 属性*配方。

## 另请参阅

*Maintaining basic file attributes using the BasicFileAttributeView*配方详细介绍了通过此视图可用的属性。*使用 FileOwnerAttributeView 维护文件所有权属性*配方讨论了所有权问题。要确定操作系统是否支持 POSIX，请查看*确定属性视图的操作系统支持*配方。

# 使用`DosFileAttributeView`维护 FAT 表属性

`java.nio.file.attribute.DosFileAttributeView`涉及较旧的**磁盘操作系统**（**DOS**）文件。在今天的大多数计算机上，它的价值有限。但是，这是唯一可以用来确定文件是否标记为归档文件或系统文件的接口。

## 准备就绪

要使用`DosFileAttributeView`接口：

1.  使用`Files`类的`getFileAttributeView`方法获取`DosFileAttributeView`的实例。

1.  使用视图的`readAttributes`方法返回`DosFileAttributes`的实例。

1.  使用`DosFileAttributes`类的方法获取文件信息。

此视图支持以下四种方法：

+   `isArchive：`关注文件是否需要备份

+   `isHidden：`如果文件对用户不可见，则返回`true`

+   `isReadOnly：`如果文件只能读取，则返回`true`

+   `isSystem：`如果文件是操作系统的一部分，则返回`true`

## 如何做...

1.  创建一个新的控制台应用程序，并添加以下`main`方法。在此方法中，我们创建`DosFileAttributes`的一个实例，然后使用其方法显示有关文件的信息：

```java
public static void main(String[] args) {
Path path = FileSystems.getDefault().getPath("/home/docs/users.txt");
try {
DosFileAttributeView view = Files.getFileAttributeView(path, DosFileAttributeView.class);
DosFileAttributes attributes = view.readAttributes();
System.out.println("isArchive: " + attributes.isArchive());
System.out.println("isHidden: " + attributes.isHidden());
System.out.println("isReadOnly: " + attributes.isReadOnly());
System.out.println("isSystem: " + attributes.isSystem());
}
catch (IOException ex) {
ex.printStackTrace();
}
}

```

1.  执行程序。您的输出应如下所示：

**isArchive：true**

**isHidden：false**

**isReadOnly：false**

**isSystem：false**

## 它是如何工作的...

创建了一个代表`users.txt`文件的`Path`对象。将此对象用作`Files`类的`getFileAttributeView`方法的参数，以及`DosFileAttributeView.class`。返回了`DosFileAttributeView`接口的一个实例。这被用于创建`DosFileAttributes`接口的一个实例，该实例与接口的四个方法一起使用。

`DosFileAttributeView`扩展了`BasicFileAttributes`接口，并因此继承了其所有属性，如*使用 BasicFileAttributeView 维护基本文件属性*配方中所述。

## 另请参阅

有关其方法的更多信息，请参阅*使用 BasicFileAttributeView 维护基本文件属性*配方。

# 使用 FileOwnerAttributeView 来维护文件所有权属性

如果我们只对访问文件或目录的所有者的信息感兴趣，那么`java.nio.file.attribute.FileOwnerAttributeView`接口提供了检索和设置此类信息的方法。文件所有权的设置在第四章的*设置文件和目录所有者*配方中有所涵盖，*管理文件和目录*。

## 准备就绪

检索文件的所有者：

1.  获取`FileOwnerAttributeView`接口的实例。

1.  使用其`getOwner`方法返回代表所有者的`UserPrincipal`对象。

## 如何做...

1.  创建一个新的控制台应用程序。将以下`main`方法添加到其中。在此方法中，我们将确定`users.txt`文件的所有者如下：

```java
public static void main(String[] args) {
Path path = Paths.get("C:/home/docs/users.txt");
try {
FileOwnerAttributeView view = Files.getFileAttributeView(path, FileOwnerAttributeView.class);
UserPrincipal userPrincipal = view.getOwner();
System.out.println(userPrincipal.getName());
}
catch (IOException e) {
e.printStackTrace();
}
}

```

1.  执行应用程序。您的输出应该类似于以下内容，除了 PC 和用户名应该不同。

**Richard-PC\Richard**

## 它是如何工作的...

为`users.txt`文件创建了一个`Path`对象。接下来，使用`Path`对象作为第一个参数调用了`Files`类的`getFileAttributeView`方法。第二个参数是`FileOwnerAttributeView.class`，这导致返回文件的`FileOwnerAttributeView`对象。

然后调用视图的`getOwner`方法返回一个`UserPrincipal`对象。它的`getName`方法返回用户的名称，然后显示出来。

## 另请参阅

有关其方法的更多信息，请参阅*使用 BasicFileAttributeView 维护基本文件属性*配方。

# 使用 AclFileAttributeView 维护文件的 ACL

`java.nio.file.attribute.AclFileAttributeView`接口提供了对文件或目录的 ACL 属性的访问。这些属性包括用户主体、属性类型以及文件的标志和权限。使用此接口的能力允许用户确定可用的权限并修改这些属性。

## 准备就绪

确定文件或目录的属性：

1.  创建代表该文件或目录的`Path`对象。

1.  使用此`Path`对象作为`Files`类的`getFileAttributeView`方法的第一个参数。

1.  使用`AclFileAttributeView.class`作为其第二个参数。

1.  使用返回的`AclFileAttributeView`对象访问该文件或目录的 ACL 条目列表。

## 如何做...

1.  创建一个新的控制台应用程序。在`main`方法中，我们将检查`users.txt`文件的 ACL 属性。使用`getFileAttributeView`方法获取视图并访问 ACL 条目列表。使用两个辅助方法来支持此示例：`displayPermissions`和`displayEntryFlags`。使用以下`main`方法：

```java
public static void main(String[] args) {
Path path = Paths.get("C:/home/docs/users.txt");
try {
AclFileAttributeView view = Files.getFileAttributeView(path, AclFileAttributeView.class);
List<AclEntry> aclEntryList = view.getAcl();
for (AclEntry entry : aclEntryList) {
System.out.println("User Principal Name: " + entry.principal().getName());
System.out.println("ACL Entry Type: " + entry.type());
displayEntryFlags(entry.flags());
displayPermissions(entry.permissions());
System.out.println();
}
}
catch (IOException e) {
e.printStackTrace();
}
}

```

1.  创建`displayPermissions`方法以显示文件的权限列表如下：

```java
private static void displayPermissions(Set<AclEntryPermission> permissionSet) {
if (permissionSet.isEmpty()) {
System.out.println("No Permissions present");
}
else {
System.out.println("Permissions");
for (AclEntryPermission permission : permissionSet) {
System.out.print(permission.name() + " " );
}
System.out.println();
}
}

```

1.  创建`displayEntryFlags`方法以显示文件的 ACL 标志列表如下：

```java
private static void displayEntryFlags(Set<AclEntryFlag> flagSet) {
if (flagSet.isEmpty()) {
System.out.println("No ACL Entry Flags present");
}
else {
System.out.println("ACL Entry Flags");
for (AclEntryFlag flag : flagSet) {
System.out.print(flag.name() + " ");
}
System.out.println();
}
}

```

1.  执行应用程序。您应该得到类似以下的输出：

**用户主体名称：BUILTIN\Administrators**

**ACL 条目类型：允许**

**没有 ACL 条目标志**

**权限**

WRITE_ATTRIBUTES EXECUTE DELETE READ_ATTRIBUTES WRITE_DATA READ_ACL READ_DATA WRITE_OWNER READ_NAMED_ATTRS WRITE_ACL APPEND_DATA SYNCHRONIZE DELETE_CHILD WRITE_NAMED_ATTRS

用户主体名称：NT AUTHORITY\SYSTEM

ACL 条目类型：允许

未出现 ACL 条目标志

权限

WRITE_ATTRIBUTES EXECUTE DELETE READ_ATTRIBUTES WRITE_DATA READ_ACL READ_DATA WRITE_OWNER READ_NAMED_ATTRS WRITE_ACL APPEND_DATA SYNCHRONIZE DELETE_CHILD WRITE_NAMED_ATTRS

用户主体名称：BUILTIN\Users

ACL 条目类型：允许

未出现 ACL 条目标志

权限

READ_DATA READ_NAMED_ATTRS EXECUTE SYNCHRONIZE READ_ATTRIBUTES READ_ACL

用户主体名称：NT AUTHORITY\Authenticated Users

ACL 条目类型：允许

未出现 ACL 条目标志

权限

READ_DATA READ_NAMED_ATTRS WRITE_ATTRIBUTES EXECUTE DELETE APPEND_DATA SYNCHRONIZE READ_ATTRIBUTES WRITE_NAMED_ATTRS WRITE_DATA READ_ACL

## 它是如何工作的...

创建了到`users.txt`文件的`Path`。然后将其与`AclFileAttributeView.class`参数一起用作`getFileAttributeView`方法的参数。这将返回`AclFileAttributeView`的一个实例。

`AclFileAttributeView`接口有三种方法：`name, getAcl`和`setAcl`。在本例中，只使用了`getAcl`方法，它返回了一个`AclEntry`元素列表。每个条目代表文件的特定 ACL。

使用 for each 循环来遍历列表。显示了用户主体的名称和条目类型。接下来调用了`displayEntryFlags`和`displayPermissions`方法来显示有关条目的更多信息。

这两种方法在构造上相似。进行了检查以确定集合中是否有任何元素，并显示了适当的消息。接下来，将集合的每个元素显示在单独的一行上，以节省输出的垂直空间。

## 还有更多...

`AclFileAttributeView`源自`java.nio.file.attribute.FileOwnerAttributeView`接口。这提供了对`getOwner`和`setOwner`方法的访问。这些方法分别为文件或目录返回或设置`UserPrincipal`对象。

有三种`AclFileAttributeView`方法：

+   `getAcl`方法，返回 ACL 条目列表，如前所示

+   `setAcl`方法，允许我们向文件添加新属性

+   `name`方法，简单地返回`acl`

`getAcl`方法将返回一个`AclEntrys`列表。条目的一个元素是一个`java.nio.file.attribute.UserPrincipal`对象。正如我们在前面的示例中看到的，这代表了可以访问文件的用户。访问用户的另一种技术是使用`java.nio.file.attribute.UserPrincipalLookupService`类。可以使用`FileSystem`类的`getUserPrincipalLookupService`方法获取此类的实例，如下所示：

```java
try {
UserPrincipalLookupService lookupService = FileSystems.getDefault().getUserPrincipalLookupService();
GroupPrincipal groupPrincipal = lookupService.lookupPrincipalByGroupName("Administrators");
UserPrincipal userPrincipal = lookupService.lookupPrincipalByName("Richard");
System.out.println(groupPrincipal.getName());
System.out.println(userPrincipal.getName());
}
catch (IOException e) {
e.printStackTrace();
}

```

服务可用的两种方法可以按用户名或组名查找用户。在前面的代码中，我们使用了`Administrators`组和用户`Richard`。

将此代码添加到上一个示例中，并更改名称以反映系统中的组和用户。当代码执行时，您应该收到类似以下的输出：

BUILTIN\Administrators

Richard-PC\Richard

然而，请注意，`UserPrincipal`和`java.nio.file.attribute.GroupPrincipal`对象的方法提供的信息比用户的名称更少。用户或组名称可能是大小写敏感的，这取决于操作系统。如果使用无效的名称，将抛出`java.nio.file.attribute.UserPrincipalNotFoundException`。

## 另请参阅

在第四章中讨论了管理文件所有权和权限的内容，*管理文件和目录*中的*设置文件和目录所有者*配方。第四章还涵盖了在*管理 ACL 文件权限*配方中说明的 ACL 属性的设置。

# 使用 UserDefinedFileAttributeView 维护用户定义的文件属性

`java.nio.file.attribute.UserDefinedFileAttributeView`接口允许将非标准属性附加到文件或目录。这些类型的属性有时被称为**扩展**属性。通常，用户定义的属性存储有关文件的元数据。这些数据不一定被文件系统理解或使用。

这些属性存储为名称/值对。名称是一个`String`，值存储为`ByteBuffer`对象。该缓冲区的大小不应超过`Integer.MAX_VALUE`。

## 准备就绪

用户定义的属性必须首先附加到文件上。这可以通过以下方式实现：

1.  获取`UserDefinedFileAttributeView`对象的实例

1.  创建一个以`String`名称和`ByteBuffer`值形式的属性

1.  使用`write`方法将属性附加到文件

读取用户定义属性的过程在本配方的*更多内容*部分进行了说明。

## 如何做...

1.  创建一个新的控制台应用程序。在`main`方法中，我们将创建一个名为`publishable`的用户定义属性，并将其附加到`users.txt`文件。使用以下`main`方法：

```java
public static void main(String[] args) {
Path path = Paths.get("C:/home/docs/users.txt");
try {
UserDefinedFileAttributeView view = Files.getFileAttributeView(path, UserDefinedFileAttributeView.class);
view.write("publishable", Charset.defaultCharset().encode("true"));
System.out.println("Publishable set");
}
catch (IOException e) {
e.printStackTrace();
}
}

```

1.  执行应用程序。您的输出应如下所示：

**设置为可发布**

## 它是如何工作的...

首先，创建一个代表`users.txt`文件的`Path`对象。然后使用`Files`类的`getFileAttributeView`方法，使用`Path`对象和`UserDefinedFileAttributeView.class`作为第二个参数。这将返回文件的`UserDefinedFileAttributeView`的实例。

使用这个对象，我们对其执行`write`方法，使用属性`publishable`，创建了一个包含属性值`true`的`java.nio.ByteBuffer`对象。`java.nio.Charset`类的`defaultCharset`方法返回一个使用底层操作系统的区域设置和字符集的`Charset`对象。`encode`方法接受`String`并返回属性值的`ByteBuffer`。然后我们显示了一个简单的消息，指示进程成功完成。

## 还有更多...

`read`方法用于读取属性。要获取与文件关联的用户定义属性，需要按照以下步骤进行：

1.  获取`UserDefinedFileAttributeView`对象的实例。

1.  为属性名称创建一个`String`。

1.  分配一个`ByteBuffer`来保存值。

1.  使用`read`方法获取属性值。

以下代码序列完成了先前附加的`publishable`属性的任务：

```java
String name = "publishable";
ByteBuffer buffer = ByteBuffer.allocate(view.size(name));
view.read(name, buffer);
buffer.flip();
String value = Charset.defaultCharset().decode(buffer).toString();
System.out.println(value);

```

首先创建属性名称的`String`。接下来，创建一个`ByteBuffer`来保存要检索的属性值。`allocate`方法根据`UserDefinedFileAttributeView`接口的`size`方法指定的空间分配空间。此方法确定附加属性的大小并返回大小。

然后对`view`对象执行`read`方法。缓冲区填充了属性值。`flip`方法重置了缓冲区。使用`decode`方法将缓冲区转换为`String`对象，该方法使用操作系统的默认字符集。

在`main`方法中，用这个`read`序列替换用户定义的属性`write`序列。当应用程序执行时，您应该得到类似以下的输出：

**true**

还有一个`delete`方法，用于从文件或目录中删除用户定义的属性。另外，需要注意使用`UserDefinedFileAttributeView`对象需要运行时权限`accessUserDefinedAttributes`。
