# 您的第一个 Java 项目

在前几章中，您学到了关于 Java 的许多东西，包括其基本方面和主要工具。现在，我们将应用所学知识来完成并迈出迈向真实程序的第一步——创建一个 Java 项目。我们将向您展示如何编写应用程序代码，如何测试它以及如何执行主代码及其测试。

在本章中，我们将涵盖以下主题：

+   什么是项目？

+   创建项目

+   编写和构建应用程序代码

+   执行和单元测试应用程序

+   练习：JUnit `@Before`和`@After`注解

# 什么是项目？

让我们从项目的定义和起源开始。

# 项目的定义和起源

根据牛津词典的英语，术语*项目*是*一个个人或协作的企业，经过精心计划以实现特定目标*。这个术语被 IDE 的设计者采用，意思是组成应用程序的文件集合。这就是为什么项目这个术语经常被用作应用程序的同义词。

# 与项目相关的术语

构成项目的文件存储在文件系统的目录中。最顶层的目录称为*项目根目录*，项目的其余目录形成其下的树。这就是为什么项目也可以被看作是包含应用程序和其测试的所有`.java`文件和其他文件的目录树。非 Java 文件通常称为`资源`，并存储在同名目录中。

程序员还使用*源代码树*、*源代码*或*源*这些术语作为项目的同义词。

当一个项目使用另一个项目的类时，它们被打包成一个`.jar`文件，通常构成一个*库*（一个或多个独立类的集合）或*框架*（一组旨在共同支持某些功能的类）。库和框架之间的区别不影响您的项目如何访问其类，因此从现在开始，我们将称项目使用的所有第三方`.jar`文件为库。在*Maven 项目配置*部分，我们将向您展示如何访问这些库，如果您的代码需要它们。

# 项目的生命周期

Java 项目的生命周期包括以下阶段（步骤、阶段）：

+   可行性：是否继续进行项目的决定

+   需求收集和高级设计

+   类级设计：*开发阶段的第一阶段*

+   项目创建

+   编写应用程序代码及其单元测试

+   项目构建：代码编译

+   将源代码存储在远程存储库中并与其他程序员共享

+   项目打包：将`.class`文件和所有支持的非 Java 文件收集到一个`.jar`文件中，通常称为*项目构件*或*构件*

+   项目安装：将构件保存在二进制存储库（也称为*构件库*）中，从中可以检索并与其他程序员共享。这个阶段是开发阶段的最后一个阶段

+   在测试环境中部署和执行项目；将构件放入一个可以在类似于生产环境的条件下执行和测试的环境中，*这是测试阶段*

+   项目在生产环境中部署和执行：*这是生产（也称为维护）阶段的第一阶段*

+   项目增强和维护：修复缺陷并向应用程序添加新功能

+   在不再需要项目后关闭项目

在本书中，我们只涵盖了四个项目阶段：

+   项目设计（参见第八章，*面向对象设计（OOD）原则*）

+   项目创建

+   编写应用程序代码及其单元测试

+   项目构建，使用`javac`工具进行代码编译

我们将向您展示如何使用 IntelliJ IDEA 社区版执行所有这些阶段，但其他 IDE 也有类似的操作。

为了构建项目，IDE 使用 Java 编译器（`javac`工具）和依赖管理工具。后者设置了`javac`和`java`命令中`-classpath`选项的值。最流行的三种依赖管理工具是 Maven、Gradle 和 Ant。IntelliJ IDEA 具有内置的 Maven 功能，不需要安装外部的依赖管理工具。

# 创建项目

有几种在 IntelliJ IDEA（或其他任何 IDE）中创建项目的方法：

+   使用项目向导（请参阅“使用项目向导创建项目”部分）

+   从文件系统中读取现有源代码

+   从源代码控制系统中读取现有源代码

在本书中，我们只会介绍第一种选项——使用项目向导。其他两个选项只需一步即可完成，无需太多解释。在学会如何手动创建项目之后，您将了解在从现有源代码自动创建项目时发生了什么。

# 使用项目向导创建项目

当您启动 IntelliJ IDEA 时，除了第一次，它会显示您已创建的项目列表。否则，您只会看到以下屏幕：

![](img/d9c1ca00-c44a-4df6-a758-41466300cb6b.png)

导入项目、打开项目和从版本控制中检出这三个选项允许您处理现有项目。我们在本书中不会使用它们。

单击“创建新项目”链接，这将带您到项目创建向导的第一个屏幕。在左上角选择 Java，然后单击右上角的“新建”按钮，并选择计算机上安装的 JDK 的位置。之后，单击右下角的“确定”按钮。

在下一个窗口中，不要选择任何内容，只需单击“下一步”按钮：

![](img/61ba60bf-f3e7-4947-b186-8ce954d1be64.png)

您在上面的屏幕截图中看不到“下一步”按钮，因为它在实际屏幕的底部，其余部分是空白空间，我们决定不在这里显示。

在下一个屏幕中，在上方的字段中输入项目名称（通常是您的应用程序名称），如下所示：

![](img/1421815e-24de-4e39-8cec-9bba0f4fd9f9.png)

对于我们的演示代码，我们选择了项目（应用程序）名称为`javapath`，意思是 Java 编程的路径。单击上一个屏幕底部的“完成”按钮，您应该会看到类似于这样的内容：

![](img/25b5df1f-4238-4be3-8534-e105e42805bc.png)

如果您在左窗格中看不到项目结构，请单击“查看”（在最顶部菜单中），然后选择“工具窗口”，然后选择“项目”，如下面的屏幕截图所示：

![](img/e275f861-da88-4088-a94a-f662403384f7.png)

现在您应该能够看到项目结构：

![](img/5ee3b110-40db-43bf-9b63-d774581d76ff.png)

前面的项目包括：

+   `.idea`目录保存了项目的 IntelliJ IDEA 设置

+   `src`目录，包括子目录：

+   `main`，将在其`java`子目录（对于`.java`文件）和`resources`子目录（对于其他类型的文件）中保存应用程序文件，

+   `test`，将在其`java`（对于`.java`文件）和`resources`子目录（对于其他类型的文件）中保存应用程序的测试。

+   `javapath.iml`文件，这是另一个带有项目配置的 IntelliJ IDEA 文件

+   `External Libraries`目录，其中包含项目使用的所有库

在前面的截图中，你还可以看到`pom.xml`文件。这个文件用于描述代码所需的其他库。我们将在“Maven 项目配置”部分解释如何使用它。IDE 会自动生成它，因为在上一章中，在配置 IDE 时，我们指示了我们希望在 IDE 默认设置中与 Maven 集成。如果你还没有这样做，现在你可以右键单击项目名称（在我们的例子中是`JavaPath`），然后选择“添加框架支持”：

![](img/fd6fe0b9-d54c-462c-b45c-d40a1e40db73.png)

然后，你将看到一个屏幕，你可以选择 Maven：

![](img/4ab050c6-5a58-44be-8f58-756afa545957.png)

点击“确定”按钮，`pom.xml`文件将被创建。如果`pom.xml`文件没有 Maven 符号，应该按照前面的截图进行相同的步骤。添加 Maven 支持后的效果如下：

![](img/9cf1a0bf-2b97-459f-9abe-2fb46d64cc2a.png)

触发`pom.xml`创建的另一种方法是响应右下角弹出的小窗口，其中包含各种建议，包括“添加为 Maven 项目”（这意味着代码依赖将由 Maven 管理）：

![](img/ad8b0ad0-e94e-465c-b8d6-ed44567a3de9.png)

如果你错过了点击前面的链接，你仍然可以通过点击底部的链接来恢复建议：

![](img/20cf88b3-d22d-42b9-bdf5-9096150d0c7c.png)

它将把建议带回到屏幕左下角：

![](img/ddd94b75-8dde-4477-9f1f-d7d8b59d5888.png)

点击“添加为 Maven 项目”链接，`pom.xml`文件将被创建。

另一个有用的建议如下：

![](img/cfc6d2f8-2f92-4f4c-8829-2911c6e9ca5a.png)

我们建议你点击“启用自动导入”链接。这将使 IDE 更好地支持你的项目，从而免除你手动执行某些操作。

如果以上方法都不适用于你，总是可以手动创建`pom.xml`文件。只需右键单击左窗格中的项目名称（`JavaPath`），选择“新建”，选择“文件”，然后输入文件名`pom.xml`，并点击“确定”按钮。

# Maven 项目配置

正如我们已经提到的，Maven 在编译和运行应用程序时帮助组成`javac`和`java`命令。它设置了`-classpath`选项的值。为了实现这一点，Maven 从`pom.xml`中读取项目所需的库列表。你有责任正确指定这些库。否则，Maven 将无法找到它们。

默认情况下，`pom.xml`文件位于项目根目录。这也是 IDE 运行`javac`命令并将`src/main/java`目录设置为类路径的目录，以便`javac`可以找到项目的源文件。它还将编译后的`.class`文件放在`target/classes`目录中，也放在根目录中，并在执行`java`命令时将此目录设置为类路径。

`pom.xml`的另一个功能是描述你的项目，以便它可以在你的计算机上唯一地被识别，甚至在互联网上的所有其他项目中也是如此。这就是我们现在要做的。让我们来看看`pom.xml`文件的内部：

![](img/ede91dcf-6317-495e-a25a-978535dca95c.png)

你可以看到标识项目的三个 XML 标签：

+   `groupId`标识组织或开源社区内项目的组

+   `artifactId`标识组内的特定项目

+   `version`标识项目的版本

`groupId`标签中设置的值必须遵循包命名约定，所以现在，我们需要解释一下包是什么。包是 Java 应用程序的最大结构单元。每个包都将相关的 Java 类分组在一起。不同包中的两个不同类可以具有相同的名称。这就是为什么包也被称为命名空间。

包名必须是唯一的。它使我们能够正确地识别一个类，即使在类路径上列出了具有相同名称的其他包中存在一个类。包可以有几个子包。它们以类似于文件系统的目录结构的层次结构组织。包含所有其他包的包称为顶级包。它的名称被用作`pom.xml`文件的`groupId`标签值。

包命名约定要求顶级包名基于创建包的组织的互联网域名（倒序）。例如，如果域名是`oracle.com`，那么顶级包名必须是`com.oracle`，后面跟着（在一个点，`.`后）项目名称。或者，可以在倒置的域名和项目名称之间插入子域、部门名称或任何其他项目组。然后，其他子包跟随。

许多 JDK 标准库的包以`jdk`、`java`或`javax`开头，例如。但最佳实践是遵循 Java 规范第 6.1 节中定义的命名约定（[`docs.oracle.com/javase/specs`](https://docs.oracle.com/javase/specs)）。

选择一个独特的包名可能会有问题，当一个开源项目开始时，没有任何组织在脑海中。在这种情况下，程序员通常使用`org.github.<作者的名字>`或类似的东西。

在我们的项目中，我们有一个顶级的`com.packt.javapath`包。这样做有一点风险，因为另一个 Packt 的作者可能决定以相同的名称开始包。最好以`com.packt.nicksamoylov.javapath`开始我们的包。这样，作者的名字将解决可能的冲突，除非当然，另一个同名的作者开始为 Packt 写 Java 书。但是，我们决定冒险简洁。此外，我们认为我们在这本书中创建的代码不会被另一个项目使用。

因此，我们项目的`groupId`标签值将是`com.packt.javapath`。

`artifactId`标签值通常设置为项目名称。

`version`标签值包含项目版本。

`artifactId`和`version`用于在项目打包期间形成`.jar`文件名。例如，如果项目名称是`javapath`，版本是`1.0.0`，`.jar`文件名将是`javapath-1.0.0.jar`。

因此，我们的`pom.xml`现在看起来像这样：

![](img/7101640e-2074-4130-82b3-97ef4adedbc0.png)

注意版本中的`-SNAPSHOT`后缀。它的用处只有当您要与其他程序员共享同一个项目时才会显现出来。但我们现在会解释它，这样您就能理解这个值的目的。当一个项目的构件（一个`.jar`文件）被创建时，它的名称将是`javapath-1.0-SNAPSHOT.jar`。文件名中的`-SNAPSHOT`表示它是一个正在进行的工作，代码正在从构建到构建中改变。这样，使用您的构件的其他 Maven 管理的项目将在`.jar`文件上的时间戳更改时每次下载它。

当代码稳定下来，更改变得罕见时，您可以将版本值设置为`1.0.0`，并且只有在代码更改并发布新项目版本时才更改它——例如`javapath-1.0.0.jar`、`javapath-1.0.1.jar`或`javapath-1.2.0.jar`。然后，使用`javapath`的其他项目不会自动下载新的文件版本。相反，另一个项目的程序员可以阅读每个新版本的发布说明，并决定是否使用它；新版本可能会引入不希望的更改，或者与他们的应用程序代码不兼容。如果他们决定需要一个新版本，他们会在项目的`pom.xml`文件中的`dependencies`标签中设置它，然后 Maven 会为他们下载它。

在我们的`pom.xml`文件中，还没有`dependencies`标签。但它可以放置在`<project>...</project>`标签的任何位置。让我们看一下`pom.xml`文件中依赖项的一些示例。我们现在可以将它们添加到项目中，因为无论如何我们以后都会使用它们：

```java

<dependencies>

<dependency>

<groupId>org.junit.jupiter</groupId>

<artifactId>junit-jupiter-api</artifactId>

<version>5.1.0-M1</version>

</dependency>

<dependency>

<groupId>org.postgresql</groupId>

<artifactId>postgresql</artifactId>

<version>42.2.2</version>

</dependency>

<dependency>

<groupId>org.apache.commons</groupId>

<artifactId>commons-lang3</artifactId>

<version>3.4</version>

</dependency>

</dependencies>

```

第一个`org.junit.jupiter`依赖项是指`junit-jupiter-api-5.1.0-M1.jar`文件，其中包含编写测试所需的`.class`文件。我们将在下一节*编写应用程序代码和测试*中使用它。

第二个`org.postgresql`依赖项是指`postgresql-42.2.2.jar`文件，允许我们连接并使用 PostgreSQL 数据库。我们将在第十六章中使用此依赖项，*数据库编程*。

第三个依赖项是指`org.apache.commons`文件`commons-lang3-3.4.jar`，其中包含许多称为实用程序的小型、非常有用的方法，其中一些我们将大量使用，用于各种目的。

每个`.jar`文件都存储在互联网上的一个仓库中。默认情况下，Maven 将搜索其自己的中央仓库，位于[`repo1.maven.org/maven2`](http://repo1.maven.org/maven2)。您需要的绝大多数库都存储在那里。但在您需要指定其他仓库的罕见情况下，除了 Maven 中央仓库之外，您可以这样做：

```java

<repositories>

<repository>

<id>my-repo1</id>

<name>your custom repo</name>

<url>http://jarsm2.dyndns.dk</url>

</repository>

<repository>

<id>my-repo2</id>

<name>your custom repo</name>

<url>http://jarsm2.dyndns.dk</url>

</repository>

</repositories>

```

阅读 Maven 指南，了解有关 Maven 的更多详细信息[`maven.apache.org/guides`](http://maven.apache.org/guides)。

配置了`pom.xml`文件后，我们可以开始为我们的第一个应用程序编写代码。但在此之前，我们想提一下如何自定义 IntelliJ IDEA 的配置，以匹配您对 IDE 外观和其他功能的偏好。

# 随时更改 IDE 设置

您可以随时更改 IntelliJ IDEA 的设置和项目配置，以调整 IDE 的外观和行为，使其最适合您的风格。花点时间看看您可以在以下每个配置页面上设置什么。

要更改 IntelliJ IDEA 本身的配置：

+   在 Windows 上：点击顶部菜单上的文件，然后选择设置

+   在 Linux 和 macOS 上：点击顶部菜单上的 IntelliJ IDEA，然后选择首选项

您访问的配置屏幕将类似于以下内容：

![](img/d957132c-f95f-4808-8c69-af37ff21b84c.png)

四处点击并查看您在这里可以做什么，以便了解 IDE 的可能性。

要更改特定于项目的设置，请单击文件，然后选择项目结构，并查看可用的设置和选项。请注意，可以通过右键单击项目名称（在左窗格中）然后选择打开模块设置来访问相同的屏幕。

在您建立了自己的风格并了解了自己的偏好之后，您可以将它们设置为 IDE 配置的默认设置，方法是通过文件|其他设置|默认设置。

默认项目结构也可以通过文件|其他设置|默认项目结构进行设置。这些默认设置将在每次创建新项目时自动应用。

有了这些，我们可以开始编写我们的应用程序代码了。

# 编写应用程序代码

这是程序员职业中最有趣的活动。这也是本书的目的——帮助你写出优秀的 Java 代码。

让我们从你的第一个应用程序的需求开始。它应该接受一个整数作为输入，将其乘以`2`，并以以下格式打印结果：`<输入数字> * 2 = <结果>`。

现在，让我们来设计一下。我们将创建`SimpleMath`类，其中包含`multiplyByTwo(int i)`方法，该方法将接受一个整数并返回结果。这个方法将被`MyApplication`类的`main()`方法调用。`main()`方法应该：

+   从用户那里接收一个输入数字

+   将输入值传递给`multiplyByTwo(int i)`方法

+   得到结果

+   以所需的格式在屏幕上打印出来

我们还将为`multiplyByTwo(int i)`方法创建测试，以确保我们编写的代码能够正确运行。

我们将首先创建包含我们的`.java`文件的目录。目录路径必须与每个类的包名匹配。我们已经讨论过包，并将顶级包名设置为`groupId`值。现在，我们将描述如何在`.java`文件中声明它。

# Java 包声明

包声明是任何 Java 类的第一行。它以`package`关键字开头，后面跟着包名。`javac`和`java`工具使用完全限定的类名在类路径上搜索类，这是一个在类名前附加包名的类名。例如，如果我们将`MyApplication`类放在`com.packt.javapath.ch04demo`包中，那么这个类的完全限定名将是`com.packt.javapath.ch04demo.MyApplication`。你可以猜到，`ch04demo`代表第四章的*演示代码*。这样，我们可以在不同的章节中使用相同的类名，它们不会冲突。这就是包名用于唯一标识类在类路径上的目的。

包的另一个功能是定义`.java`文件的位置，相对于`src\main\java`目录（适用于 Windows）或`src/main/java`目录（适用于 Linux）。包名必须与属于该包的文件的路径匹配：

```java

src\main\java\com\packt\javapath\ch04demo\MyApplication.java (适用于 Windows)

src/main/java/com/packt/javapath/ch04demo/MyApplication.java (适用于 Linux)

```

包名与文件位置之间的任何不匹配都会触发编译错误。当使用 IDE 向包名右键单击后使用 IDE 向导创建新类时，IDE 会自动将正确的包声明添加为`.java`文件的第一行。但是，如果不使用 IDE 创建新的源文件，那么就需要自己负责匹配包名和`.java`文件的位置。

如果`.java`文件位于`src\main\java`目录（适用于 Windows）或`src/main/java`目录（适用于 Linux）中，则可以不声明包名。Java 规范将这样的包称为默认包。使用默认包只适用于小型或临时应用程序，因为随着类的数量增加，一百甚至一千个文件的平面列表将变得难以管理。此外，如果你编写的代码要被其他项目使用，那么这些其他项目将无法在没有包名的情况下引用你的类。在第七章《包和可访问性（可见性）》中，我们将更多地讨论这个问题。

在编译过程中，`.class`文件的目录树是由`javac`工具创建的，并且它反映了`.java`文件的目录结构。Maven 在项目根目录中创建了一个`target`目录，并在其中创建了一个`classes`子目录。然后，Maven 在`javac`命令中使用`-d`选项指定这个子目录作为生成文件的输出位置：

```java

//对于 Windows：

javac -classpath src\main\java -d target\classes

com.packt.javapath.ch04demo.MyApplication.java

//对于 Linux：

javac -classpath src/main/java -d target/classes

com.packt.javapath.ch04demo.MyApplication.java

```

在执行过程中，`.class`文件的位置设置在类路径上：

```java

//对于 Windows：

java -classpath target\classes com.packt.javapath.ch04demo.MyApplication

//对于 Linux：

java -classpath target/classes com.packt.javapath.ch04demo.MyApplication

```

有了包声明、其功能以及与目录结构的关系的知识，让我们创建我们的第一个包。

# 创建一个包

我们假设您已经按照“使用项目向导创建项目”的步骤创建了项目。如果您已经关闭了 IDE，请重新启动它，并通过在“最近项目”列表中选择`JavaPath`来打开创建的项目。

项目打开后，在左窗格中点击`src`文件夹，然后点击`main`文件夹。现在应该看到`java`文件夹：

![](img/7f2a3362-e71f-4b18-b321-994848eff558.png)

右键单击`java`文件夹，选择“新建”菜单项，然后选择“包”菜单项：

![](img/12d9f974-ec53-408e-a288-edc7b25dca93.png)

在弹出窗口中输入`com`：

![](img/6bf9e2d4-2ca4-4d79-8073-f976cff359bb.png)

点击“确定”按钮。将创建`com`文件夹。

在左窗格中右键单击它，选择“新建”菜单项，然后选择“包”菜单项，在弹出窗口中输入`packt`：

![](img/8bc316e8-3f10-4edf-9fff-2b5784b465cf.png)

重复这个过程，在`packt`文件夹下创建`javapath`文件夹，然后在`javapath`文件夹下创建`ch04demo`文件夹。在`com.packt.javapath.ch04demo`包就位后，我们可以创建它的成员——`MyApplication`类。

# 创建`MyApplication`类

要创建一个类，在左窗格中右键单击`com.packt.javapath.che04demo`包，选择“新建”菜单项，然后选择“Java 类”菜单项，在弹出窗口中输入`MyApplication`：

![](img/12cce37d-6bdc-4d61-a9a2-3a990766a9a5.png)

点击“确定”按钮，类将被创建：

![](img/c6371bf2-f8d5-44dc-a830-12f4c166a782.png)

右窗格中`MyApplication`类的名称变得模糊。这就是 IntelliJ IDEA 指示它尚未被使用的方式。

# 构建应用程序

在幕后，IDE 会在每次更改代码时编译您正在编写的代码。例如，尝试删除右窗格中类名称中的第一个字母`M`。IDE 会立即警告您有语法错误：

![](img/df804b66-9c28-487f-84ec-a9d3ca2b67d1.png)

如果将鼠标移到前面截图中类声明的红色气泡或任何下划线类声明的红线上，您将看到“类'yApplication'是公共的，应该在名为'yApplication.java'的文件中声明”的消息。您可能还记得我们在第二章中谈到过这一点，*Java 语言基础知识*。

每个`.java`文件只包含一个`public`类。文件名必须与公共类名匹配。

因为 IDE 在每次更改后都会编译代码，所以在少量`.java`文件的情况下，显式构建项目是不必要的。但是当应用程序的大小增加时，您可能不会注意到出现问题。

这就是为什么请求 IDE 定期重新编译（或者换句话说，构建）应用程序的所有`.java`文件是一个好的做法，方法是点击顶部菜单中的“构建”，然后选择“重建项目”菜单项：

![](img/a1f2f3c7-e8e0-463c-9ef4-366adf39fdc4.png)

您可能已经注意到其他相关的菜单项：Build Project 和 Build Module 'javapath'。模块是一种跨包捆绑类的方式。但是使用模块超出了本书的范围。Build Project 仅重新编译已更改的类以及使用更改的类的类。只有在构建时间显着时才有意义。另一方面，Rebuild Projects 重新编译所有`.java`文件，无论它们是否已更改，我们建议您始终使用它。这样，您可以确保每个类都已重新构建，并且没有遗漏依赖项。

单击 Rebuild Projects 后，您将在左窗格中看到一个新的`target`文件夹：

![](img/eace96bb-ca9f-452d-b0fb-c24037973683.png)

这是 Maven（和 IntelliJ IDEA 使用的内置 Maven）存储`.class`文件的地方。您可能已经注意到`javac`工具为包名的每个部分创建一个文件夹。这样，编译类的树完全反映了源类的树。

现在，在继续编写代码之前，我们将执行一个技巧，使您的源树看起来更简单。

# 隐藏一些文件和目录

如果您不希望看到特定于 IDE 的文件（例如`.iml`文件）或临时文件和目录（例如`target`文件夹），可以配置 IntelliJ IDEA 不显示它们。只需单击 File | Settings（在 Windows 上）或 IntelliJ IDEA | Preferences（在 Linux 和 macOS 上），然后单击左列中的 Editor 菜单项，然后单击 File Types。生成的屏幕将具有以下部分：

![](img/182f27f3-539a-4f24-b9aa-4fc65604cd81.png)

在屏幕底部，您可以看到忽略文件和文件夹标签以及带有文件名模式的输入字段。在列表的末尾添加以下内容：`*.iml;.idea;target;`。然后，单击 OK 按钮。现在，您的项目结构应该如下所示：

![](img/f17e89e1-be92-4df8-972a-f1c5e0d93a20.png)

它仅显示应用程序源文件和第三方库（在外部库下）。

# 创建 SimpleMath 类

现在让我们创建另一个包`com.packt.javapath.math`，并在其中创建`SimpleMath`类。这样做的原因是，将来我们计划在此包中有几个类似的与数学相关的类，以及其他与数学无关的类。

在左窗格中，右键单击`com.packt.javapath.ch04demo`包，选择 New，然后单击 Package。在提供的输入字段中键入`math`，然后单击 OK 按钮。

右键单击`math`包名称，选择 New，然后单击 Java Class，在提供的输入字段中键入`SimpleMath`，然后单击 OK 按钮。

你应该创建一个新的`SimpleMath`类，看起来像这样：

![](img/45202168-ee23-40e7-9910-2122a88846bd.png)

# 创建方法

首先，我们将以下方法添加到`SimpleMath`类中：

```java

public int multiplyByTwo(int i){

return i * 2;

}

```

现在，我们可以将使用上述方法的代码添加到`MyApplication`类中：

```java

public static void main(String[] args) {

int i = Integer.parseInt(args[0]);

SimpleMath simpleMath = new SimpleMath();

int result = simpleMath.multiplyByTwo(i);

System.out.println(i + " * 2 = " + result);

}

```

上述代码非常简单。应用程序从`String[] args`输入数组的第一个元素接收一个整数作为输入参数。请注意，Java 数组中的第一个元素的索引是 0，而不是 1。参数作为字符串传递，并且必须通过使用标准 Java 库中`java.lang.Integer`类的`parseInt()`静态方法转换（解析）为`int`类型。我们将在第五章中讨论 Java 类型，*Java 语言元素和类型*。

然后，创建了一个`SimpleMath`类的对象，并调用了`multiplyByTwo()`方法。返回的结果存储在`int`类型的`result`变量中，然后使用标准 Java 库的`java.lang.System`类以所需的格式打印出来。这个类有一个`out`静态属性，它持有一个对`java.io.PrintStream`类对象的引用。而`PrintStream`类又有`println()`方法，它将结果打印到屏幕上。

# 执行和单元测试应用程序

有几种方法可以执行我们的新应用程序。在*构建应用程序*部分，我们看到所有编译后的类都存储在`target`文件夹中。这意味着我们可以使用`java`工具并列出带有`-classpath`选项的`target`文件夹来执行应用程序。

要做到这一点，打开命令提示符或终端窗口，然后转到我们新项目的根目录。如果不确定在哪里，可以查看 IntelliJ IDEA 窗口顶部显示的完整路径。一旦进入项目根目录（即存放`pom.xml`文件的文件夹），运行以下命令：

![](img/dad25f60-410a-4c64-9847-d83b2932c7f5.png)

在上述截图中，可以看到`-classpath`选项（我们使用了缩写版本`-cp`）列出了所有编译后的类所在的目录。之后，我们输入了`com.packt.javapath.ch04demo.MyApplication`主类的名称，因为我们必须告诉`java`工具哪个类是应用程序的入口点，并包含`main()`方法。然后，我们输入`2`作为主类的输入参数。你可能还记得，`main()`方法期望它是一个整数。

当我们运行该命令时，结果以预期格式显示输出：`2 * 2 = 4`。

或者，我们可以将所有编译后的类收集到一个`myapp.jar`文件中，并使用类似的`java`命令在类路径上列出`myapp.jar`文件来运行：

![](img/68445744-4225-4d7a-a13a-b29e1f44445e.png)

在上述截图中，可以看到我们首先进入了`target`文件夹及其`classes`子文件夹，然后使用`jar`命令将其内容（所有编译后的类）收集到`myapp.jar`文件中。然后，我们使用`java`命令并列出了`myapp.jar`文件和`-classpath`选项。由于`myapp.jar`文件在当前目录中，我们不包括任何目录路径。`java`命令的结果与之前相同：`2 * 2 = 4`。

另一种进入项目根目录的方法是直接从 IDE 打开终端窗口。在 IntelliJ IDEA 中，可以通过单击左下角的 Terminal 链接来实现：

![](img/b634572c-f39d-48af-9664-10a12682bd46.png)

然后，我们可以在 IDE 内部的终端窗口中输入所有上述命令。

但是，有一种更简单的方法可以在项目开发阶段从 IDE 中执行应用程序，而不必输入所有上述命令，这是推荐的方法。这是你的 IDE，记住吗？我们将在下一节中演示如何做到这一点。

# 使用 IDE 执行应用程序

为了能够从 IDE 执行应用程序，首次需要进行一些配置。在 IntelliJ IDEA 中，如果单击最顶部的菜单项，点击 Run，然后选择 Edit Configurations...，将会看到以下屏幕：

![](img/57ee7aff-829c-47f4-a2c3-b3c1153ff62d.png)

单击左上角的加号（+）符号，并在新窗口中输入值：

![](img/4e6536d9-cea0-4cea-a7d5-c70c4db164f3.png)

在名称字段中输入`MyApplication`（或其他任何名称）。

在主类字段中输入`com.packt.javapath.ch02demo.MyApplication`。

在程序参数字段中输入`2`（或其他任何数字）。

在右上角的单一实例复选框中选中。这将确保您的应用程序始终只运行一个实例。

在填写了所有描述的值之后，单击右下角的 OK 按钮。

现在，如果您打开`MyApplication`类，您将看到两个绿色箭头 - 一个在类级别，另一个在`main()`方法中：

![](img/44313d36-8ea3-40bf-bd74-ce920ac2a46d.png)

单击其中任何一个绿色箭头，您的应用程序将被执行。

结果将显示在 IntelliJ IDEA 左下角。将打开一个名为 Run 的窗口，并且您将看到应用程序执行的结果。如果您在程序参数字段中输入了`2`，则结果应该是相同的：`2 * 2 = 4`。

# 创建单元测试

现在，让我们为`SimpleMath`类的`multiplyByTwo()`方法编写一个测试，因为我们希望确保`multiplyByTwo()`方法按预期工作。只要项目存在，这样的测试就很有用，因为您可以在每次更改代码时运行它们，并验证现有功能没有意外更改。

方法是应用程序中最小的可测试部分。这就是为什么这样的测试被称为单元测试。为您创建的每个方法编写单元测试是一个好主意（例如，除了诸如 getter 和 setter 之类的微不足道的方法）。

我们将使用一个名为 JUnit 的流行测试框架。有几个版本。在撰写本文时，版本 5 是最新版本，但版本 3 和 4 仍在积极使用。我们将使用版本 5。它需要 Java 8 或更高版本，并且我们假设您的计算机上至少安装了 Java 9。

如我们已经提到的，在使用第三方库或框架时，您需要在`pom.xml`文件中将其指定为依赖项。一旦您这样做，Maven 工具（或 IDE 的内置 Maven 功能）将在 Maven 在线存储库中查找相应的`.jar`文件。它将下载该`.jar`文件到您计算机主目录中自动创建的`.m2`文件夹中的本地 Maven 存储库。之后，您的项目可以随时访问并使用它。

我们已经在*Maven 项目配置*部分的`pom.xml`中设置了对 JUnit 5 的依赖。但是，假设我们还没有这样做，以便向您展示程序员通常如何做。

首先，您需要进行一些研究并决定您需要哪个框架或库。例如，通过搜索互联网，您可能已经阅读了 JUnit 5 文档（[`junit.org/junit5`](http://junit.org/junit5)）并发现您需要在`junit-jupiter-api`上设置 Maven 依赖项。有了这个，您可以再次搜索互联网，这次搜索`maven dependency junit-jupiter-api`，或者只搜索`maven dependency junit 5`。您搜索结果中的第一个链接很可能会将您带到以下页面：

![](img/25e23750-6422-4a3e-adfc-297ea4cd2aff.png)

选择您喜欢的任何版本（我们选择了最新版本 5.1.0-M1）并单击它。

将打开一个新页面，告诉您如何在`pom.xml`中设置依赖项：

![](img/3c580196-a134-452e-a96d-b7e352f511d9.png)

或者，您可以转到 Maven 存储库网站（[`mvnrepository.com`](https://mvnrepository.com)）并在其搜索窗口中键入`junit-jupiter-api`。然后，单击提供的链接之一，您将看到相同的页面。

如果您在阅读第三章 *您的开发环境设置*时没有添加`junit-jupiter-api`依赖项，现在可以通过将提供的依赖项复制到`pom.xml`文件中的`<dependencies></dependencies>`标签内来添加它：

![](img/1a19f2bc-aebf-42ee-aefd-43d8f4c3e9c7.png)

现在，您可以使用 JUnit 框架创建单元测试。

在 IntelliJ IDEA 中，`junit-jupiter-api-5.1.0-M1.jar`文件也列在左侧窗格的`External Libraries`文件夹中。如果您打开列表，您将看到还有两个其他库，这些库没有在`pom.xml`文件中指定：`junit-latform-commons-1.0.0-M1.jar`和`opentest4j-1.0.0.jar`。它们存在是因为`junit-jupiter-api-5.1.0-M1.jar`依赖于它们。这就是 Maven 的工作原理-它发现所有依赖项并下载所有必要的库。

现在，我们可以为`SimpleMath`类创建一个测试。我们将使用 IntelliJ IDEA 来完成。打开`SimpleMath`类，右键单击类名，然后选择 Go To，点击 Test：

![](img/2432ae77-367c-4583-8688-33a3955ee126.png)

您将会看到一个小弹出窗口：

![](img/a8498649-3985-495c-aee9-94585cd58e4d.png)

单击 Create New Test...，然后以下窗口将允许您配置测试：

![](img/b349dccc-04c1-4b39-9c78-af90cfef7b8d.png)

在 IntelliJ IDEA 中有对 JUnit 5 的内置支持。在前面的屏幕中，选择 JUnit5 作为测试库，并选中`multiplyByTwo()`方法的复选框。然后，单击右下角的 OK 按钮。测试将被创建：

![](img/1e3526b4-c252-4073-ad78-73a72c9c099c.png)

请注意，在左侧窗格的`test/java`文件夹下，创建了一个与`SimpleMath`类的包结构完全匹配的包结构。在右侧窗格中，您可以看到`SimpleMathTest`测试类，其中包含一个针对`multiplyByTwo()`方法的测试（目前为空）。测试方法可以有任何名称，但必须在其前面加上`@Test`，这被称为注解。它告诉测试框架这是其中一个测试。

让我们实现测试。例如，我们可以这样做：

![](img/3640513c-02b3-481e-9869-09c7297fcbc5.png)

正如你所看到的，我们已经创建了`SimpleMath`类的对象，并调用了带有参数`2`的`multiplyByTwo()`方法。我们知道正确的结果应该是`4`，我们使用来自 JUnit 框架的`assertEquals()`方法来检查结果。我们还在类和测试方法中添加了`@DisplayName`注解。您很快就会看到这个注解的作用。

现在让我们修改`SimpleMath`类中的`mutliplyByTwo()`方法：

![](img/a6427a90-f3ca-4f1e-abb6-e8ea76953674.png)

我们不仅仅是乘以`2`，我们还将`1`添加到结果中，所以我们的测试将失败。首先在错误的代码上运行测试是一个好习惯，这样我们可以确保我们的测试能够捕捉到这样的错误。

# 执行单元测试

现在，让我们回到`SimpleMathTest`类，并通过单击绿色箭头之一来运行它。类级别上的绿色箭头运行所有测试方法，而方法级别上的绿色箭头只运行该测试方法。因为我们目前只有一个测试方法，所以单击哪个箭头都无所谓。结果应该如下所示：

![](img/168972fa-0fe1-42c0-9063-8a9127bbdef0.png)

这正是我们希望看到的：测试期望得到一个等于`4`的结果，但实际得到了`5`。这让我们对我们的测试是否正确工作有了一定的信心。

请注意，在左侧窗格中，我们可以看到来自`@DisplayName`注解的显示名称-这就是这些注解的目的。

还要单击右侧窗格中的每个蓝色链接，以查看它们的作用。第一个链接提供有关预期和实际结果的更详细信息。第二个链接将带您到测试的行，其中包含失败测试的断言，这样您就可以看到确切的上下文并纠正错误。

现在，您可以再次转到`SimpleMath`类，并删除我们添加的`1`。然后，单击左上角的绿色三角形（参见前面的屏幕截图）。这意味着*重新运行测试*。结果应该如下所示：

![](img/8851a540-9056-4835-a8e3-45437fcb18d8.png)

顺便说一下，您可能已经注意到我们的屏幕截图和项目路径已经略有改变。这是因为我们现在是从在 macOS 上运行的 IntelliJ IDEA 中获取屏幕截图，所以我们可以覆盖 Windows 和 macOS。正如您所看到的，IntelliJ IDEA 屏幕在 Windows 和 macOS 系统上的外观基本相同。

# 多少单元测试足够？

这总是任何程序员在编写新方法或修改旧方法时都会考虑的问题-有多少单元测试足以确保应用程序得到彻底测试，以及应该是什么样的测试？通常，仅为应用程序的每个方法编写一个测试是不够的。通常需要测试许多功能方面。但是，每个测试方法应该只测试一个方面，这样更容易编写和理解。

例如，对于我们简单的`multiplyByTwo()`方法，我们可以添加另一个测试（我们将称之为`multiplyByTwoRandom()`），它会将随机整数作为输入传递给方法，并重复一百次。或者，我们可以考虑一些极端的数字，比如`0`和负数，并查看我们的方法如何处理它们（例如，我们可以称它们为`multiplyByZero()`和`multiplyByNegative()`）。另一个测试是使用一个非常大的数字-比 Java 允许的最大整数的一半还要大（我们将在第五章中讨论这样的限制，*Java 语言元素和类型*）。我们还可以考虑在`multiplyByTwo()`方法中添加对传入参数值的检查，并在传入参数大于最大整数的一半时抛出异常。我们将在第十章中讨论异常，*控制流语句*。

您可以看到最简单的方法的单元测试数量增长得多快。想象一下，对于一个比我们简单代码做得多得多的方法，可以编写多少单元测试。

我们也不希望写太多的单元测试，因为我们需要在项目的整个生命周期中维护所有这些代码。过去，不止一次，一个大项目因为编写了太多复杂的单元测试而变得维护成本过高，而这些测试几乎没有增加任何价值。这就是为什么通常在项目代码稳定并在生产中运行一段时间后，如果有理由认为它有太多的单元测试，团队会重新审视它们，并确保没有无用的测试、重复的测试或其他明显的问题。

编写良好的单元测试，可以快速工作并彻底测试代码，这是一种随着经验而来的技能。在本书中，我们将利用一切机会与您分享单元测试的最佳实践，以便在本书结束时，您将在这个非常重要的专业 Java 编程领域中有一些经验。

# 练习-JUnit @Before 和@After 注释

阅读 JUnit 用户指南（[`junit.org/junit5/docs/current/user-guide`](https://junit.org/junit5/docs/current/user-guide)）和类`SampleMathTest`两个新方法：

+   只有在任何测试方法运行之前执行一次的方法

+   只有在所有测试方法运行后执行一次的方法

我们没有讨论它，所以您需要进行一些研究。

# 答案

对于 JUnit 5，可以用于此目的的注释是`@BeforeAll`和`@AfterAll`。这是演示代码：

```java

public class DemoTest {

@BeforeAll

在所有之前的静态方法中：

System.out.println("beforeAll is executed");

}

@AfterAll

在所有之后的静态方法中：

System.out.println("afterAll is executed");

}

@Test

void test1(){

System.out.println("test1 is executed");

}

@Test

void test2(){

System.out.println("test2 is executed");

}

}

```

如果您运行它，输出将是：

```java

beforeAll is executed

test1 被执行

test2 is executed

afterAll is executed

```

# 总结

在本章中，您了解了 Java 项目以及如何设置和使用它们来编写应用程序代码和单元测试。您还学会了如何构建和执行应用程序代码和单元测试。基本上，这就是 Java 程序员大部分时间所做的事情。在本书的其余部分，您将更详细地了解 Java 语言、标准库以及第三方库和框架。

在下一章中，我们将深入探讨 Java 语言的元素和类型，包括`int`、`String`和`arrays`。您还将了解标识符是什么，以及如何将其用作变量的名称，以及有关 Java 保留关键字和注释的信息。
