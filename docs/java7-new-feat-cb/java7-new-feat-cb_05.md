# 第五章：管理文件系统

在本章中，我们将涵盖以下内容：

+   获取 FileStore 信息

+   获取 FileSystem 信息

+   使用 SimpleFileVisitor 类遍历文件系统

+   使用 SimpleFileVisitor 类删除目录

+   使用 SimpleFileVisitor 类复制目录

+   使用 DirectoryStream 接口处理目录的内容，如*使用 globbing 过滤目录*教程中所述

+   编写自己的目录过滤器

+   使用 WatchEvents 监视文件事件

+   理解 ZIP 文件系统提供程序

# 介绍

**文件系统**是一个或多个顶级根目录，包含文件层次结构。文件系统由文件存储支持，该文件存储是文件存储的提供者。本章涉及获取有关这些实体和典型文件系统任务的信息，例如确定目录的内容或监视文件系统事件。

文件存储表示存储单元。例如，它可能表示设备，比如`C`驱动器，驱动器的分区或卷。`java.nio.file.FileStore`类支持文件存储，并提供了几种方法。*获取 FileStore 信息*教程介绍了如何获取有关特定文件存储的基本信息。

文件系统支持访问目录和文件的层次结构。它在 Java 7 中用`java.nio.file.FileSystem`类表示。获取有关文件系统的一般信息在*获取 FileSystem 信息*教程中介绍。这包括如何获取文件系统的根目录列表和底层文件存储。

遍历目录层次结构对许多应用程序很有用。*使用 SimpleFileVisitor 类遍历文件系统*教程详细介绍了基本方法。这种方法在*使用 SimpleFileVisitor 类删除目录*和*使用 SimpleFileVisitor 类复制目录*教程中使用。

当操作限制在单个目录时，`java.nio.file.DirectoryStream`接口提供了一种方便的技术，用于将目录中的每个元素作为`java.nio.file.Path`对象进行检查。使用 for each 循环处理这些路径非常容易。这种方法在*使用 DirectoryStream 接口处理目录的内容*教程中探讨。

有时我们不需要整个目录的内容，而是需要其元素的子集。Java 7 提供了几种过滤目录内容的方法，如*使用 globbing 过滤目录*和*编写自己的目录过滤器*教程中所述。**Globbing**是一种类似于正则表达式但更容易使用的模式匹配技术。

在*使用 WatchEvents 监视文件事件*教程中，我们了解到 Java 7 支持通过外部进程检测目录中文件的创建、修改和删除。当需要知道对目录进行更改时，这可能非常有用。

使用 Java 7，现在可以将 ZIP 文件的内容视为文件系统。这使得更容易管理 ZIP 文件的内容并操作 ZIP 文件中包含的文件。这种技术在*理解 zip 文件系统提供程序*教程中进行了演示。

# 获取 FileStore 信息

每个文件系统都支持文件存储机制。这可能是一个设备，比如`C`驱动器，一个驱动器的分区，一个卷，或者其他组织文件系统空间的方式。`java.nio.file.FileStore`类代表其中一个存储分区。本教程详细介绍了获取有关文件存储的信息的方法。

## 准备工作

要获取并使用`FileStore`对象：

1.  获取正在使用的`java.nio.file.FileSystem`的实例。

1.  使用`FileSystem`类的`getFileStores`方法返回可用的文件存储。

## 如何做…

1.  创建一个新的控制台应用程序。在`main`方法中，我们将使用`FileStore`类的几种方法来演示该类提供的支持。让我们从添加`main`方法的第一部分开始，其中我们显示初始标题并获取一个`FileSystem`对象。还定义一个名为`kiloByte`的`long`变量：

```java
static final long kiloByte = 1024;
public static void main(String[] args) throws IOException {
String format = "%-16s %-20s %-8s %-8s %12s %12s %12s\n";
System.out.printf(format,"Name", "Filesystem", "Type",
"Readonly", "Size(KB)", "Used(KB)",
"Available(KB)");
FileSystem fileSystem = FileSystems.getDefault();
}

```

1.  接下来，我们需要使用`getFileStores`方法检索可用的文件存储，并显示它们。在代码块的第一部分中，我们使用了几个`FileStore`方法来获取相关信息。在最后一部分，我们按以下方式显示信息：

```java
for (FileStore fileStore : fileSystem.getFileStores()) {
try {
long totalSpace = fileStore.getTotalSpace() / kiloByte;
long usedSpace = (fileStore.getTotalSpace() -
fileStore.getUnallocatedSpace()) / kiloByte;
long usableSpace = fileStore.getUsableSpace() / kiloByte;
String name = fileStore.name();
String type = fileStore.type();
boolean readOnly = fileStore.isReadOnly();
NumberFormat numberFormat = NumberFormat.getInstance();
System.out.printf(format,
name, fileStore, type, readOnly,
numberFormat.format(totalSpace),
numberFormat.format(usedSpace),
numberFormat.format(usableSpace));
}
catch (IOException ex) {
ex.printStackTrace();
}
}

```

1.  执行应用程序。您的输出将与以下内容不同，但应反映系统上的驱动器：

**名称 文件系统类型 只读 大小（KB）已用（KB）可用（KB）**

**HP HP (C:) NTFS false 301,531,984 163,041,420 138,490,564**

**FACTORY_IMAGE FACTORY_IMAGE (D:) NTFS false 11,036,652 9,488,108 1,548,544**

**HP_PAVILION HP_PAVILION (E:) NTFS false 312,568,640 66,489,184 246,079,456**

**TOSHIBA TOSHIBA (H:) FAT32 false 15,618,080 3,160,768 12,457,312**

## 工作原理...

创建了一个格式化字符串，以简化文件存储信息的显示。此字符串在两个`printf`方法中都使用。两次使用相同的字符串可以确保输出的一致间距。使用此字符串显示了一个简单的标题。

使用`FileSystems`类的`getDefault`方法获取了一个`FileSystem`对象。对该对象执行`getFileStores`方法以获取`FileStore`对象的列表。

在循环中，使用 try 块捕获可能抛出的异常。按照下表详细说明的方式调用了几个方法。创建了`NumberFormat`类的实例以格式化文件存储大小信息。最后的`printf`方法显示了每个文件存储的文件存储信息：

| 方法 | 含义 |
| --- | --- |
| `getTotalSpace` | 文件存储中可用的总空间（以字节为单位） |
| `getUnallocatedSpace` | 未分配的字节数 |
| `getUsableSpace` | JVM 可用的可用字节数 |
| `name` | 表示文件存储名称的特定于实现的字符串 |
| `type` | 表示文件存储类型的特定于实现的字符串 |
| `isReadOnly` | 如果方法返回`true`，则尝试创建文件或打开文件进行写入将导致抛出`IOException` |

`getUnallocatedSpace`或`getUsableSpace`方法返回的值可能会在外部操作使用或释放文件存储空间时发生变化。

## 另请参阅

使用两种`supportsFileAttributeView`方法之一来确定`FileStore`支持的属性视图。这些方法在第三章的*确定操作系统对属性视图的支持*食谱的*还有更多..*部分中进行了说明，*获取文件和目录信息*。

# 获取文件系统信息

文件系统由一系列目录和文件组成。通常有一些关于文件系统的有限信息是有用的。例如，我们可能想知道文件系统是否为只读，或者提供者是谁。在本示例中，我们将研究用于检索文件系统属性的可用方法。

## 准备就绪

要访问文件系统的方法，我们需要：

1.  获取对`java.nio.file.FileSystem`对象的引用。

1.  使用此对象的方法来访问文件系统信息。

## 如何做...

1.  创建一个新的控制台应用程序。将以下代码添加到应用程序的`main`方法中。此序列显示了几个`fileSystem`属性，包括文件系统提供程序、文件打开状态、文件是否可读写、根目录和文件存储的名称：

```java
FileSystem fileSystem = FileSystems.getDefault();
FileSystemProvider provider = fileSystem.provider();
System.out.println("Provider: " + provider.toString());
System.out.println("Open: " + fileSystem.isOpen());
System.out.println("Read Only: " + fileSystem.isReadOnly());
Iterable<Path> rootDirectories = fileSystem.getRootDirectories();
System.out.println();
System.out.println("Root Directories");
for (Path path : rootDirectories) {
System.out.println(path);
}
Iterable<FileStore> fileStores = fileSystem.getFileStores();
System.out.println();
System.out.println("File Stores");
for (FileStore fileStore : fileStores) {
System.out.println(fileStore.name());
}

```

1.  执行应用程序。您的输出将取决于系统的配置。但是，它应该与以下输出类似：

**提供程序：sun.nio.fs.WindowsFileSystemProvider@7b60e796**

**打开：true**

**只读：false**

**根目录**

**C:\**

**D:\**

**E:\**

**F:\**

**G:\**

**H:\**

**I:\**

**J:\**

**K:\**

**L:\**

**文件存储**

**HP**

**FACTORY_IMAGE**

**HP_PAVILION**

**TOSHIBA**

## 工作原理...

`getDefault`方法返回 JVM 使用的默认文件系统。接下来，针对此对象执行了几种方法：

+   `provider`方法返回提供程序，即文件系统的实现者。在这种情况下，它是与 JVM 捆绑在一起的 Windows 文件系统提供程序。

+   `isOpen`方法指示文件系统已打开并准备就绪。

+   `isReadOnly`方法返回`false`，这意味着我们可以读写系统。

+   我们使用`getRootDirectories`方法创建了一个`Iterable`对象，允许我们列出每个根目录。

+   `getFileStores`方法返回另一个`Iterable`对象，用于显示文件存储的名称。

## 还有...

虽然我们通常不需要关闭文件系统，但是`close`方法可以用于关闭文件系统。对文件系统执行的任何后续方法都将导致抛出`ClosedFileSystemException`。与文件系统关联的任何打开通道、目录流和监视服务也将被关闭。请注意，默认文件系统无法关闭。

`FileSystems`类的`getFileSystem`方法可用于访问特定的文件系统。此外，重载的`newFileSystem`方法将创建新的文件系统。`close`方法可以用于这些实例。

文件系统是线程安全的。但是，如果一个线程尝试关闭文件系统，而另一个线程正在访问`filesystem`对象，关闭操作可能会被阻塞。

直到访问完成。

# 使用`SimpleFileVisitor`类来遍历文件系统

在处理目录系统时，常见的需求是遍历文件系统，检查文件层次结构中的每个子目录。使用`java.nio.file.SimpleFileVisitor`类可以轻松完成这项任务。该类实现了在访问目录之前和之后执行的方法。此外，对于在目录中访问每个文件实例以及发生异常的情况，还会调用回调方法。

`SimpleFileVisitor`类或派生类与`java.nio.file.Files`类的`walkFileTree`方法一起使用。它执行深度优先遍历，从特定的根目录开始。

## 准备就绪

要遍历目录，我们需要：

1.  创建一个代表根目录的`Path`对象。

1.  创建一个派生自`SimpleFileVisitor`的类的实例。

1.  将这些对象用作`Files`类的`walkFileTree`方法的参数。

## 如何做...

1.  创建一个新的控制台应用程序，并使用以下`main`方法。在这里，我们将遍历`home`目录，并列出其每个元素如下：

```java
public static void main(String[] args) {
try {
Path path = Paths.get("/home");
ListFiles listFiles = new ListFiles();
Files.walkFileTree(path, listFiles);
}
catch (IOException ex) {
ex.printStackTrace();
}
}

```

1.  将以下`ListFiles`类添加到您的项目中。它说明了每个`SimpleFileVisitor`方法的用法：

```java
class ListFiles extends SimpleFileVisitor<Path> {
private final int indentionAmount = 3;
private int indentionLevel;
public ListFiles() {
indentionLevel = 0;
}
private void indent() {
for(int i=0 ; i<indentionLevel; i++) { {
System.out.print(' ');
}
}
@Override
public FileVisitResult visitFile(Path file, BasicFileAttributes attributes) {
indent();
System.out.println("Visiting file:" + file.getFileName());
return FileVisitResult.CONTINUE;
}
@Override
public FileVisitResult postVisitDirectory(Path directory, IOException e) throws IOException {
indentionLevel -= indentionAmount;
indent();
System.out.println("Finished with the directory: " + directory.getFileName());
return FileVisitResult.CONTINUE;
}
@Override
public FileVisitResult preVisitDirectory(Path directory, BasicFileAttributes attributes) throws IOException {
indent();
System.out.println("About to traverse the directory: " + directory.getFileName());
indentionLevel += indentionAmount;
return FileVisitResult.CONTINUE;
}
SimpleFileVisitor classusing, for filesystem traverse@Override
public FileVisitResult visitFileFailed(Path file, IOException exc) throws IOException {
System.out.println("A file traversal error ocurred");
return super.visitFileFailed(file, exc);
}
}

```

1.  执行应用程序。根据您的`home`目录的结构，您可能会得到与以下不同的结果：

即将遍历目录：home

**即将遍历目录：docs**

**访问文件：users.bak**

**访问文件：users.txt**

**完成目录：docs**

**即将遍历目录：music**

**访问文件：Future Setting A.mp3**

**访问文件：Robot Brain A.mp3**

**访问文件：Space Machine A.mp3**

**完成目录：music**

**完成目录：home**

检查`backup`目录，以验证它是否成功创建。

## 工作原理...

在`main`方法中，我们为`home`目录创建了一个`Path`对象。接下来，创建了`ListFiles`类的一个实例。这些对象被用作`walkFileTree`方法的参数。该方法影响了`home`目录的遍历，并根据需要调用了`ListFiles`类的方法。

`walkFileTree`方法从根目录开始，并对目录层次结构进行深度优先遍历。在遍历目录之前，将调用`preVisitDirectory`方法。接下来，处理目录的每个元素。如果是文件，则调用`visitFile`方法。一旦处理了目录的所有元素，将调用`postVisitDirectory`方法。如果发生异常，则将调用`visitFileFailed`方法。

添加了私有辅助方法，使输出更易读。`indentionAmount`变量控制了每个缩进的深度。`indentionLevel`变量在访问每个子目录时递增和递减。`indent`方法执行实际的缩进。

## 还有更多...

有两个重载的`walkFileTree`方法。一个接受`Path`和`FileVisitor`对象，之前已经说明过。它不会跟踪链接，并将访问目录的所有级别。第二个方法接受两个额外的参数：一个指定要访问的目录级别的数量，另一个用于配置遍历。目前，唯一可用的配置选项是`FileVisitOption.FOLLOW_LINKS`，它指示方法跟随符号链接。

默认情况下不会跟随符号链接。如果在`walkFileTree`方法的参数中指定了跟随它们，则会注意检测循环链接。如果检测到循环链接，则将其视为错误条件。

要访问的目录级别的数量由整数参数控制。值为 0 将导致只访问顶级目录。值为`Integer.MAX_VALUE`表示将访问所有级别。值为 2 表示只遍历前两个目录级别。

遍历将在以下条件之一发生时终止：

+   所有文件都已被遍历

+   `visit`方法返回`FileVisitResult.TERMINATE`

+   `visit`方法以`IOException`或其他异常终止时，将被传播回来

任何不成功的操作通常会导致调用`visitFileFailed`方法并抛出`IOException`。

当遇到文件时，如果它不是目录，则尝试读取其`BasicFileAttributes`。如果成功，将属性传递给`visitFile`方法。如果不成功，则调用`visitFileFailed`方法，并且除非处理，否则会抛出`IOException`。

如果文件是目录并且目录可以打开，则调用`preVisitDirectory`，并访问目录及其后代的元素。

如果文件是目录且无法打开该目录，则将调用`visitFileFailed`方法，并将抛出`IOException`。但是，深度优先搜索将继续进行下一个兄弟节点。

以下表总结了遍历过程。

| 遇到的元素 | 可以打开 | 无法打开 |
| --- | --- | --- |
| 文件 | 调用`visitFile` | 调用`visitFileFailed` |
| 目录 | 调用`preVisitDirectory`目录元素被处理调用`postVisitDirectory` | 调用`visitFileFailed` |

为方便起见，列出了枚举`FileVisitResult`的枚举常量如下：

| 值 | 含义 |
| --- | --- |
| `CONTINUE` | 继续遍历 |
| `SKIP_SIBLINGS` | 继续而不访问此文件或目录的兄弟节点 |
| `SKIP_SUBTREE` | 继续而不访问此目录中的条目 |
| `TERMINATE` | 终止 |

## 另请参阅

*使用 SimpleFileVisitor 类删除目录*和*使用 SimpleFileVisitor 类复制目录*的方法利用了本方法中描述的方法来分别删除和复制目录。

# 使用 SimpleFileVisitor 类删除目录

删除目录是一些应用程序的要求。这可以通过使用`walkFileTree`方法和一个`java.nio.file.SimpleFileVisitor`派生类来实现。这个示例建立在*使用 SimpleFileVisitor 类遍历文件系统*示例提供的基础上。

## 准备工作

要删除一个目录，我们需要：

1.  创建一个代表根目录的`Path`对象。

1.  创建一个从`SimpleFileVisitor`派生的类的实例如下：

+   重写`visitFile`方法来删除文件

+   重写`postVisitDirectory`方法来删除目录

1.  将这些对象作为参数传递给`Files`类的`walkFileTree`方法。

## 如何做...

1.  创建一个新的控制台应用程序。在这里，我们将删除`home`目录及其所有元素。将以下代码添加到`main`方法中：

```java
try {
Files.walkFileTree(Paths.get("/home"), new DeleteDirectory());
}
catch (IOException ex) {
ex.printStackTrace();
}

```

1.  `DeleteDirectory`类如下所示。在删除每个文件和目录时，都会显示相应的消息：

```java
public class DeleteDirectory extends SimpleFileVisitor<Path> {
@Override
public FileVisitResult visitFile(Path file, BasicFileAttributes attributes)
throws IOException {
System.out.println("Deleting " + file.getFileName());
Files.delete(file);
return FileVisitResult.CONTINUE;
}
@Override
public FileVisitResult postVisitDirectory(Path directory, IOException exception)
throws IOException {
if (exception == null) {
System.out.println("Deleting " + directory.getFileName());
Files.delete(directory);
return FileVisitResult.CONTINUE;
}
else {
throw exception;
}
}
}

```

1.  备份`home`目录，然后执行应用程序。根据实际的目录结构，你应该会得到以下输出：

**删除 users.bak**

**删除 users.txt**

**删除 docs**

**删除 Future Setting A.mp3**

**删除 Robot Brain A.mp3**

**删除 Space Machine A.mp3**

**删除音乐**

**删除 home**

验证目录是否已被删除。

## 它是如何工作的...

在`main`方法中，我们创建了一个代表`home`目录的`Path`对象。接下来，我们创建了`DeleteDirectory`类的一个实例。这两个对象被用作`walkFileTree`方法的参数，该方法启动了遍历过程。

当遇到一个文件时，`visitFile`方法被执行。在这个方法中，我们显示了一个指示文件正在被删除的消息，然后使用`Files`类的`delete`方法来删除文件。当遇到一个目录时，`postVisitDirectory`方法被调用。进行了一个测试以确保没有发生错误，然后显示了一个指示目录正在被删除的消息，随后调用了该目录的`delete`方法。这两个方法都返回了`FileVisitResult.CONTINUE`，这将继续删除过程。

## 另请参阅

*使用 SimpleFileVisitor 类遍历文件系统*示例提供了关于使用`walkFileTree`方法和`SimpleFileVisitor`类的更多细节。*使用 SimpleFileVisitor 类复制目录*示例也提供了这种方法的变体。

# 使用 SimpleFileVisitor 类复制目录

复制目录是一些应用程序的要求。这可以通过使用`walkFileTree`方法和一个`java.nio.file.SimpleFileVisitor`派生类来实现。这个示例建立在*使用 SimpleFileVisitor 类遍历文件系统*示例提供的基础上。

## 准备工作

要删除一个目录，我们需要：

1.  创建一个代表根目录的`Path`对象。

1.  创建一个从`SimpleFileVisitor`派生的类的实例如下：

+   重写`visitFile`方法来复制文件

+   重写`preVisitDirectory`方法来复制目录

1.  将这些对象作为参数传递给`Files`类的`walkFileTree`方法。

## 如何做...

1.  创建一个新的控制台应用程序。在这里，我们将把`home`目录及其所有元素复制到一个`backup`目录中。将以下代码添加到`main`方法中：

```java
try {
Path source = Paths.get("/home");
Path target = Paths.get("/backup");
Files.walkFileTree(source,
EnumSet.of(FileVisitOption.FOLLOW_LINKS),
Integer.MAX_VALUE,
new CopyDirectory(source, target));
}
catch (IOException ex) {
ex.printStackTrace();
}

```

1.  `CopyDirectory`类如下所示。在删除每个文件和目录时，都会显示相应的消息：

```java
public class CopyDirectory extends SimpleFileVisitor<Path> {
private Path source;
private Path target;
public CopyDirectory(Path source, Path target) {
this.source = source;
this.target = target;
}
@Override
public FileVisitResult visitFile(Path file, BasicFileAttributes attributes) throws IOException {
SimpleFileVisitor classusing, for directory copySystem.out.println("Copying " + source.relativize(file));
Files.copy(file, target.resolve(source.relativize(file)));
return FileVisitResult.CONTINUE;
}
@Override
public FileVisitResult preVisitDirectory(Path directory, BasicFileAttributes attributes) throws IOException {
Path targetDirectory = target.resolve(source.relativize(directory));
try {
System.out.println("Copying " + source.relativize(directory));
Files.copy(directory, targetDirectory);
}
catch (FileAlreadyExistsException e) {
if (!Files.isDirectory(targetDirectory)) {
throw e;
}
}
return FileVisitResult.CONTINUE;
}
}

```

1.  执行应用程序。确切的输出取决于你使用的源文件结构，但应该类似于以下内容：

**复制**

**复制 docs**

**复制 docs\users.bak**

**复制 docs\users.txt**

**复制音乐**

**复制 music\Future Setting A.mp3**

**复制 music\Robot Brain A.mp3**

**复制 music\Space Machine A.mp3**

## 它是如何工作的...

在`main`方法中，我们为`home`和`backup`目录创建了`Path`对象。我们使用这些对象创建了一个`CopyDirectory`对象。我们使用了一个两参数的`CopyDirectory`构造函数，这样它的方法就可以直接访问这两个路径。

使用源`Path`调用了`walkFileTree`方法。它还作为第二个参数传递，一个`EnumSet`，指定不要跟随符号链接。这个参数需要一组选项。`EnumSet`类的静态方法创建了这个集合。

`walkFileTree`方法的第三个参数是一个值，表示要跟随多少级。我们传递了一个`Integer.MAX_VALUE`的值，这将导致复制`home`目录的所有级别。最后一个参数是`CopyDirectory`对象的一个实例。

在遍历过程中遇到文件时，将调用`CopyDirectory`类的`visitFile`方法。显示一个消息指示正在复制文件，然后使用`copy`方法将源文件复制到目标目录。使用`relativize`方法获取到源文件的相对路径，然后用作`resolve`方法的参数。结果是一个代表目标目录和源文件名的`Path`对象。这些方法在第二章的*使用路径解析组合路径*和*在两个位置之间创建路径*示例中进行了讨论，*使用路径定位文件和目录*。

当在遍历过程中遇到一个目录时，将调用`preVisitDirectory`方法。它的工作方式与`visitFile`方法相同，只是我们复制的是一个目录而不是一个文件。这两种方法都返回`FileVisitResult.CONTINUE`，这将继续复制过程。仍然需要复制目录的各个文件，因为`copy`方法只能复制单个文件。

请注意，`CopyDirectory`类扩展了`SimpleFileVisitor`类，使用`Path`作为通用值。`walkFileTree`方法需要一个实现`Path`接口的对象。因此，我们必须使用`Path`或扩展`Path`的接口。

## 另请参阅

*使用 SimpleFileVisitor 类遍历文件系统*示例提供了更多关于`walkFileTree`方法和`SimpleFileVisitor`类的使用细节。*使用 SimpleFileVisitor 类删除目录*示例也提供了这种方法的变体。

# 使用`DirectoryStream`接口处理目录的内容

确定目录的内容是一个相当常见的需求。有几种方法可以做到这一点。在这个示例中，我们将研究使用`java.nio.file.DirectoryStream`接口来支持这个任务。

目录将包含文件或子目录。这些文件可能是常规文件，也可能是链接或隐藏文件。`DirectoryStream`接口将返回所有这些元素类型。我们将使用`java.nio.file.Files`类的`newDirectoryStream`方法来获取`DirectoryStream`对象。这个方法有三个重载版本。首先演示了这个方法的最简单用法。用于过滤目录内容的版本在*使用 globbing 过滤目录*示例和*编写自己的目录过滤器*示例中展示。

## 准备工作

为了使用`DirectoryStream`，我们需要：

1.  获取`DirectoryStream`对象的实例。

1.  通过`DirectoryStream`迭代处理其元素。

## 如何做...

1.  创建一个新的控制台应用程序，并添加以下`main`方法。我们创建了一个新的`DirectoryStream`对象，然后使用 for each 循环来迭代目录元素，如下所示：

```java
public static void main(String[] args) {
Path directory = Paths.get("/home");
try (DirectoryStream<Path> directoryStream = Files.newDirectoryStream(directory)) {
for (Path file : directoryStream) {
System.out.println(file.getFileName());
}
}
catch (IOException | DirectoryIteratorException ex) {
ex.printStackTrace();
}
}

```

1.  执行应用程序。您的输出应该反映出您的`home`目录的内容，并且应该类似于以下内容：

**文档**

**音乐**

## 它是如何工作的...

为`home`目录创建了一个`Path`对象。这个对象与`newDirectoryStream`方法一起使用，该方法返回了一个目录的`DirectoryStream`对象。`DirectoryStream`接口扩展了`Iterable`接口。这允许`DirectoryStream`对象与 for each 语句一起使用，它简单地打印了`home`目录的每个元素的名称。在这种情况下，只有两个子目录：`docs`和`music`。

注意使用 try-with-resource 块。这是 Java 7 中的新功能，并在第一章中的*使用 try-with-resource 块改进异常处理代码*中进行了讨论，*Java 语言改进*。这保证了目录流将被关闭。如果没有使用这种 try 块，则在不再需要流之后关闭流是很重要的。

使用的`Iterable`对象不是通用的`iterator`。它在几个重要方面有所不同，如下所示：

+   它只支持单个`Iterator`

+   `hasNext`方法执行至少一个元素的预读

+   它不支持`remove`方法

`DirectoryStream`接口有一个方法`iterator`，它返回一个`Iterator`类型的对象。第一次调用该方法时，将返回一个`Iterator`对象。对该方法的后续调用将抛出`IllegalStateException`。

`hasNext`方法将至少提前读取一个元素。如果该方法返回`true`，则对其 next 方法的下一次调用将保证返回一个元素。返回的元素的顺序没有指定。此外，许多操作系统在许多 shell 中以`"."`或`"..`"表示对自身和/或其父级的链接。这些条目不会被返回。

`iterator`有时被称为**弱一致**。这意味着虽然`iterator`是线程安全的，但在`iterator`返回后对目录的任何更新都不会导致`iterator`的更改。

## 还有更多...

有两个重载的`newDirectoryStream`方法，允许该方法的结果通过通配符模式或`DirectoryStream.Filter`对象进行过滤。**通配符模式**是一个包含一系列字符的字符串，用于定义模式。该模式用于确定要返回哪些目录元素。`DirectoryStream.Filter`接口有一个方法`accept`，它返回一个布尔值，指示是否应返回目录元素。

## 另请参阅

*使用通配符过滤目录*示例说明了通配符模式的使用。*编写自己的目录过滤器*示例展示了如何创建和使用`DirectoryStream.Filter`对象来过滤目录的内容。

# 使用通配符过滤目录

通配符模式类似于正则表达式，但更简单。与正则表达式一样，它可以用于匹配特定的字符序列。我们可以将通配符与`newDirectoryStream`方法结合使用，以过滤目录的内容。该方法的使用在*使用 DirectoryStream 接口处理目录的内容*示例中进行了演示。

## 准备工作

要使用这种技术，我们需要：

1.  创建一个符合我们过滤要求的 globbing 字符串。

1.  为感兴趣的目录创建一个`java.nio.file.Path`对象。

1.  将这两个对象用作`newDirectoryStream`方法的参数。

## 如何做...

1.  创建一个新的控制台应用程序，并使用以下`main`方法。在这个例子中，我们将只列出那些以`java`开头并以`.exe`结尾的目录元素。我们将使用 Java 7 的`bin`目录。`globbing`字符串使用特殊字符`*`来表示零个或多个字符，如下所示：

```java
Path directory = Paths.get("C:/Program Files/Java/jdk1.7.0/bin");
try (DirectoryStream<Path> directoryStream = Files.newDirectoryStream(directory,"java*.exe")) {
for (Path file : directoryStream) {
System.out.println(file.getFileName());
}
}
catch (IOException | DirectoryIteratorException ex) {
ex.printStackTrace();
}

```

1.  执行应用程序。输出应该类似于以下内容：

**java-rmi.exe**

**java.exe**

**javac.exe**

**javadoc.exe**

**javah.exe**

**javap.exe**

**javaw.exe**

**javaws.exe**

## 它是如何工作的...

首先，创建了一个代表`bin`目录的`Path`对象。然后将其用作`newDirectoryStream`方法的第一个参数。第二个参数是`globbing`字符串。在这种情况下，它匹配以`java`开头并以`.exe`结尾的目录元素。允许任意数量的中间字符。然后使用 for each 循环显示过滤后的文件。

## 还有更多...

Globbing 字符串基于模式，使用特殊字符来匹配字符串序列。这些特殊字符在`Files`类的`getPathMatcher`方法的文档中定义。在这里，我们将更深入地研究这些字符串。以下表格总结了几个特殊字符：

| 特殊符号 | 意义 |
| --- | --- |
| * | 匹配不跨越目录边界的名称组件的零个或多个字符 |
| ** | 匹配跨越目录边界的零个或多个字符 |
| ? | 匹配名称组件的一个字符 |
| \ | 用于匹配特殊符号的转义字符 |
| [ ] | 匹配括号内找到的单个字符。A - 匹配一个范围。!表示否定。*、?和\字符匹配它们自己，-如果是括号内的第一个字符或!后的第一个字符，则匹配它自己。 |
| { } | 可以同时指定多个子模式。这些模式使用花括号分组在一起，但在花括号内部用逗号分隔。 |

匹配通常以实现相关的方式执行。这包括匹配是否区分大小写。`**`符号在这里不适用，因为`newDirectoryStream`方法返回单独的元素。在这里没有机会匹配跨越目录边界的序列。其他方法将使用这种能力。

以下表格列出了几个可能有用的 glob 模式示例：

| Globbing 字符串 | 将匹配 |
| --- | --- |
| `*.java` | 以`.java`结尾的任何文件名 |
| `*.{java,class,jar}` | 以`.java, .class`或`.jar`结尾的任何文件 |
| `java*[ph].exe` | 仅匹配以 java 开头并以`p.exe`或`h.exe`结尾的文件 |
| `j*r.exe` | 以`j`开头并以`r.exe`结尾的文件 |

现在，让我们讨论`PathMatcher`接口的使用。

### 使用 PathMatcher 接口来过滤目录

`java.nio.file.PathMatcher`接口提供了使用**glob**匹配文件名的方法。它有一个名为`matches`的方法，接受一个`Path`参数。如果文件与 glob 模式匹配，则返回`true`。否则返回`false`。

在以下代码序列中，我们通过使用 glob 模式`glob:java?.exe`创建了一个`PathMatcher`对象。在 for 循环中，我们使用`matches`方法进一步过滤以`java`开头，后跟一个字符，然后以`.exe`结尾的文件的子集：

```java
Path directory = Paths.get("C:/Program Files/Java/jdk1.7.0/bin");
PathMatcher pathMatcher = FileSystems.getDefault().getPathMatcher("glob:java?.exe");
try (DirectoryStream<Path> directoryStream =
Files.newDirectoryStream(directory,"java*.exe")) {
for (Path file : directoryStream) {
if(pathMatcher.matches(file.getFileName())) {
System.out.println(file.getFileName());
}
}
}
catch (IOException | DirectoryIteratorException ex) {
ex.printStackTrace();
}

```

当您执行此序列时，您应该得到以下输出：

**javac.exe**

**javah.exe**

**javap.exe**

**javaw.exe**

注意`matches`方法中使用的**glob:**前缀。这种方法需要使用这个前缀，但`newDirectoryStream`方法不需要。此外，`matches`方法接受一个`Path`参数。但是，请注意我们使用了从`Path`类的`getFileName`方法返回的`String`。仅使用`Path`对象或使用`String`文字均不起作用。

与使用 glob:前缀不同，我们可以改用正则表达式。为此，请使用**reg:**前缀，后跟正则表达式。

通常，对于简单的目录过滤，我们会在`newDirectoryStream`方法中使用更严格的 glob 模式。我们在这里使用它是为了举例说明。然而，如果我们想要在循环的一部分执行多个过滤操作，那么使用模式作为`newDirectoryStream`方法的一部分，然后使用一个或多个`matches`方法调用是一种可行的策略。

## 另请参阅

*编写自己的目录过滤器*配方探讨了如何创建更强大的过滤器，以匹配基于文件名以外的属性的文件名。

# 编写自己的目录过滤器

在使用`java.nio.file.Files`类的`newDirectoryStream`方法时，目录过滤器用于控制返回哪些目录元素。当我们需要限制流的输出时，这是很有用的。例如，我们可能只对超过一定大小或在某个日期后修改的文件感兴趣。正如本配方中描述的`java.nio.file.DirectoryStream.Filter`接口，它将限制流的输出。它比使用 globbing 更强大，因为决策可以基于文件名以外的因素。

## 准备工作

要使用这种技术，我们需要：

1.  创建一个满足我们过滤要求的`DirectoryStream.Filter`对象。

1.  为感兴趣的目录创建一个`Path`对象。

1.  使用这两个对象作为`newDirectoryStream`方法的参数。

## 如何做...

1.  创建一个新的控制台应用程序，并将以下序列添加到`main`方法中。在这个例子中，我们将只过滤出那些隐藏的目录元素。我们将使用 Windows 系统目录。然而，任何其他适当的目录都可以工作：

```java
DirectoryStream.Filter<Path> filter = new DirectoryStream.Filter<Path>() {
public boolean accept(Path file) throws IOException {
return (Files.isHidden(file));
}
};
Path directory = Paths.get("C:/Windows");
try (DirectoryStream<Path> directoryStream = Files.newDirectoryStream(directory,filter)){
own directory filterwritingfor (Path file : directoryStream) {
System.out.println(file.getFileName());
}
}
catch (IOException | DirectoryIteratorException ex) {
ex.printStackTrace();
}

```

1.  执行时，您的输出应该只列出那些隐藏的文件。以下是一个可能的输出：

**SwSys1.bmp**

**SwSys2.bmp**

**WindowsShell.Manifest**

## 工作原理...

首先，我们创建了一个匿名内部类来定义一个实现`DirectoryStream.Filter`接口的对象。在`accept`方法中，使用`isHidden`方法来确定元素文件是否隐藏。`DirectoryStream.Filter`接口使用其`accept`方法来确定是否应该返回目录元素。该方法返回`true`或`false`，指示`newDirectoryStream`方法是否应该返回该元素。因此，它过滤掉了不需要的元素，这种情况下是非隐藏元素。使用 for each 循环来显示隐藏元素。当声明`filter`变量时，它是使用`Path`作为其泛型值声明的。扩展`Path`接口的接口也可以使用。

## 另请参阅

这种技术过滤单个目录。如果需要过滤多个目录，则可以根据*使用 SimpleFileVisitor 类遍历文件系统*配方中使用的示例来适应多个目录。

# 使用 WatchEvents 监视文件事件

当应用程序需要了解目录中的更改时，监视服务可以监听这些更改，然后通知应用程序这些更改。服务将根据感兴趣的事件类型注册要监视的目录。事件发生时，将排队一个监视事件，随后可以根据应用程序的需求进行处理。

## 准备工作

要监视目录的事件，我们需要执行以下操作：

1.  创建一个代表目录的`java.nio.file.Path`对象。

1.  使用`java.nio.file.FileSystem`类的`newWatchService`方法创建一个新的监视服务。

1.  确定我们感兴趣监视的事件。

1.  使用监视服务注册目录和事件。

1.  处理事件发生时的事件。

## 如何做...

1.  创建一个新的控制台应用程序。我们将在`main`方法中添加代码来创建一个观察服务，确定我们想要观察的事件，将`docs`目录注册到服务中，然后处理事件。让我们从创建观察服务和目录的`Path`对象开始。将以下代码添加到`main`方法中：

```java
try {
FileSystem fileSystem = FileSystems.getDefault();
WatchService watchService = fileSystem.newWatchService();
Path directory = Paths.get("/home/docs");

```

1.  接下来，创建一个监视文件创建、删除和修改的事件数组，如下所示：

```java
WatchEvent.Kind<?>[] events = {
StandardWatchEventKinds.ENTRY_CREATE,
StandardWatchEventKinds.ENTRY_DELETE,
StandardWatchEventKinds.ENTRY_MODIFY};
directory.register(watchService, events);

```

1.  添加以下 while 循环以监视和处理任何目录事件：

```java
while (true) {
System.out.println("Waiting for a watch event");
WatchKey watchKey = watchService.take();
System.out.println("Path being watched: " + watchKey.watchable());
System.out.println();
if (watchKey.isValid()) {
for (WatchEvent<?> event : watchKey.pollEvents()) {
System.out.println("Kind: " + event.kind());
System.out.println("Context: " + event.context());
System.out.println("Count: " + event.count());
System.out.println();
}
boolean valid = watchKey.reset();
if (!valid) {
// The watchKey is not longer registered
}
}
}
}
catch (IOException ex) {
ex.printStackTrace();
}
catch (InterruptedException ex) {
ex.printStackTrace();
}

```

1.  执行应用程序。您应该得到以下输出：

等待观察事件

1.  使用文本编辑器，在`docs`目录中创建一个名为`temp.txt`的新文件并保存。然后应用程序应该显示类似以下的输出。如果这是您第一次在目录中创建文件，则您的输出可能会有所不同。这些条目表示文件已被创建，其内容随后被保存：

正在观察的路径：\home\docs

类型：ENTRY_CREATE

上下文：temp.txt

计数：1

等待观察事件

正在观察的路径：\home\docs

类型：ENTRY_MODIFY

上下文：temp.txt

计数：2

等待观察事件

1.  接下来，再次保存文件。现在您应该得到以下输出：

正在观察的路径：\home\docs

类型：ENTRY_MODIFY

上下文：temp.txt

计数：1

等待观察事件

1.  从文件管理器中删除文件。您的输出应该反映出它的删除：

类型：ENTRY_DELETE

上下文：temp1.txt

计数：1

等待观察事件

## 它是如何工作的...

我们需要的第一件事是一个`WatchService`对象。这是通过获取默认文件系统，然后对其应用`newWatchService`方法来获得的。接下来，我们创建了一个代表`docs`目录的`Path`对象和一个涵盖创建、删除和修改类型事件的事件数组。

然后进入了一个无限循环，以监视和处理`docs`目录中发生的文件事件。循环开始时显示一个消息，指示它正在等待事件。执行了`WatchService`类的`take`方法。此方法将阻塞，直到发生事件。

当事件发生时，它返回一个`WatchKey`对象，其中包含有关事件的信息。它的`watchable`方法返回被观察的对象，然后为了信息目的而显示。

使用`isValid`方法验证了观察键的有效性，并且它的`pollEvents`方法被用作 for each 循环的一部分。`pollEvents`方法返回所有待处理事件的列表。显示了与事件相关的类型、上下文和计数值。

我们监视的事件的上下文是目标目录和引起事件的条目之间的相对路径。计数值取决于事件，并在下一节中讨论。

最后的活动重置了观察键。这是为了将键重新置于就绪状态，直到再次需要它。如果方法返回`false`，则键不再有效。

## 还有更多...

`WatchService`接口具有获取观察键和关闭服务的方法。`poll`和`take`方法检索下一个观察键，就像我们之前看到的那样。如果没有观察键存在，`poll`方法将返回`null`。但是，`take`方法将阻塞，直到观察键可用。有一个重载的`poll`方法，它接受额外的参数来指定在返回之前等待事件的时间。这些参数包括超时值和`TimeUnit`值。`TimeUnit`枚举的使用在第四章的*理解 FileTime 类部分*中讨论，*管理文件和目录*。

`Path`类的`register`方法将注册由其执行的`Path`对象指定的文件。该方法接受参数：

+   指定监视服务

+   要监视的事件类型

+   确定`Path`对象注册方式的修饰符

`WatchEvent.Modifier`接口指定了如何使用监视服务注册`Path`对象。在此版本的 Java 中，没有定义修饰符。

`java.nio.file.StandardWatchEventKinds`类定义了标准事件类型。此接口的字段总结在以下表中：

| 类型 | 含义 | 计数 |
| --- | --- | --- |
| `ENTRY_CREATE` | 创建目录条目 | 总是 1 |
| `ENTRY_DELETE` | 删除目录条目 | 总是 1 |
| `ENTRY_MODIFY` | 修改目录条目 | 大于 1 |
| `OVERFLOW` | 表示事件可能已丢失或被丢弃的特殊事件 | 大于 1 |

当事件发生时，监视服务将返回一个代表事件的`WatchKey`对象。此键用于同一类型事件的多次发生。当发生该类型的事件时，与事件关联的计数将增加。如果在处理事件之前发生了该类型的多个事件，则每次增加的计数值取决于事件类型。

在前面的示例中使用`reset`方法将重新排队监视键并将计数重置为零。对于重复事件，上下文是相同的。每个目录条目将为该事件类型拥有自己的监视键。

可以使用`WatchKey`接口的`cancel`方法取消事件。这将取消事件在监视服务中的注册。队列中的任何待处理事件将保留在队列中，直到被移除。如果监视服务关闭，监视事件也将被取消。

监视服务是线程安全的。这意味着如果多个线程正在访问事件，那么在使用`reset`方法时应该小心。在所有使用该事件的线程完成处理事件之前，不应该使用该方法。

可以使用`close`方法关闭监视服务。如果多个线程正在使用此服务，那么后续尝试检索监视键将导致`ClosedWatchServiceException`。

文件系统可能能够比监视服务更快地报告事件。一些监视服务的实现可能会对排队的事件数量施加限制。当有意忽略事件时，将使用`OVERFLOW`类型的事件来报告此问题。溢出事件会自动为目标注册。溢出事件的上下文取决于实现。

监视服务的许多方面都依赖于实现，包括：

+   是否使用本机事件通知服务或模拟

+   事件被排队的及时性

+   处理事件的顺序

+   是否报告短暂事件

# 了解 ZIP 文件系统提供程序

处理 ZIP 文件比 Java 7 之前要简单得多。在这个版本中引入的 ZIP 文件系统提供程序处理 ZIP 和 JAR 文件，就好像它们是文件系统一样，因此您可以轻松访问文件的内容。您可以像处理普通文件一样操作文件，包括复制、删除、移动和重命名文件。您还可以修改文件的某些属性。本教程将向您展示如何创建 ZIP 文件系统的实例并向系统添加目录。

## 准备就绪

我们必须首先创建一个`java.net.URI`对象的实例来表示我们的 ZIP 文件，然后创建新的`java.nio.file.FileSystem`，然后才能对 ZIP 文件的内容进行任何操作。在这个例子中，我们还将使用`java.util.HashMap`来设置`FileSystem`的可选属性如下：。

1.  创建一个`URI`对象来表示 ZIP 文件。

1.  创建一个`HashMap`对象来指定`create`属性为`true`。

1.  使用`newFileSystem`方法创建`FileSystem`对象。

## 如何做...

1.  创建一个带有`main`方法的控制台应用程序。在`main`方法中，添加以下代码序列。我们将在 ZIP 文件中创建一个新的文件系统，然后将一个目录添加到其中，如下所示：

```java
Map<String, String> attributes = new HashMap<>();
attributes.put("create", "true");
try {
URI zipFile = URI.create("jar:file:/home.zip");
try (FileSystem zipFileSys = FileSystems.newFileSystem(zipFile, attributes);) {
Path path = zipFileSys.getPath("docs");
Files.createDirectory(path);
try (DirectoryStream<Path> directoryStream =
Files.newDirectoryStream(zipFileSys.getPath("/"));) {
for (Path file : directoryStream) {
System.out.println(file.getFileName());
}
}
}
}
catch (IOException e) {
e.printStackTrace();
}

```

1.  执行程序。您的输出应如下所示：

**docs/**

## 它是如何工作的...

`URI`对象通过使用`HashMap`对象指定了 ZIP 文件的位置，我们指定如果 ZIP 文件不存在，它应该被创建。`FileSystem`对象`zipFileSys`是在 try-with-resources 块中创建的，因此资源将自动关闭，但如果您不希望使用嵌套的 try-with-resources 块，您必须使用`FileSystem`类的`close`方法手动关闭资源。try-with-resources 块在第一章中有详细介绍，*Java 语言改进*，配方：*使用 try-with-resources 块改进异常处理代码*。

为了演示如何将 ZIP 文件作为`FileSystem`对象进行操作，我们调用了`createDirectory`方法在我们的 ZIP 文件中添加了一个文件夹。在这一点上，我们还有选择执行其他`FileSystem`操作的选项，比如复制文件、重命名文件和删除文件。我们使用了一个`java.nio.file.DirectoryStream`来浏览我们的 ZIP 文件结构并打印出我们的`docs`目录，但您也可以在计算机上导航到 ZIP 文件的位置来验证其创建。

## 另请参阅

有关`DirectoryStream`类的更多信息，请参阅*使用 DirectoryStream 接口处理目录的内容*配方。
