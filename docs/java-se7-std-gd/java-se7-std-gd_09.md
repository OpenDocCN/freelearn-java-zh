# 第九章：Java 应用程序

在本章中，我们将从包的角度来检查 Java 应用程序的结构。将介绍包和导入语句的使用，以及用于包的基础目录结构。

我们还将看到 Java 如何通过使用区域设置和资源包支持国际化。将介绍 JDBC 的使用，以及如何回收未使用的对象。这通常被称为**垃圾回收**。

# 代码组织

代码的组织是应用程序的重要部分。可以说，正是这种组织（以及数据组织）决定了应用程序的质量。

Java 应用程序是围绕包组织的。包含类的包。类包含数据和代码。代码可以在初始化列表或方法中找到。这种基本组织如下图所示：

![代码组织](img/7324_09_01.jpg)

代码可以被认为在性质上既是静态的又是动态的。Java 程序的组织在静态上围绕包、类、接口、初始化列表和方法进行结构化。这种组织的唯一变化来自执行程序的不同版本。然而，当程序执行时，不同的可能执行路径导致执行序列通常变得复杂。

Java API 被组织成许多包，其中包含数百个类。新的包和类正在不断添加，使得跟上 Java 的所有功能变得具有挑战性。

然而，正如在第七章*继承和多态*中的*对象类*部分中提到的，Java 中的所有类都有一个基类—`java.lang`.`Object`—直接或间接地。在您定义的类中，如果不明确扩展另一个类，Java 将自动从`Object`类扩展此类。

## 包

包的目的是将相关的类和其他元素组合在一起。理想情况下，它们形成一组连贯的类和接口。一个包可以包括：

+   类

+   接口

+   枚举

+   异常

类似功能的类应该自然地被组合在一起。大多数 Java 的 IO 类都被分组在`java.io`或`java.nio`相关的包中。所有 Java 的网络类都在`java.net`包中找到。这种分组机制为我们提供了一个更容易讨论和处理的单一逻辑分组。

所有类都属于一个包。如果未指定包，则类属于一个未命名的默认包。该包包括未声明为属于某个包的目录中的所有类。

## 包的目录/文件组织

要将类放入包中，有必要：

+   在类源文件中使用包语句

+   将相应的`.class`文件移动到包目录中

包语句需要是类源文件中的第一个语句。该语句由关键字`package`和包的名称组成。以下示例声明了类`Phone`属于`acme.telephony`包：

```java
package acme.telephony;

class Phone {
   …
}
```

Java 源代码文件以`.java`扩展名保存在与类同名的文件中。如果一个文件中保存了多个类，则只能声明一个类为 public，并且文件必须以此公共类命名。`java.lang`包包含许多常用的类，并且在每个应用程序中自动包含。

第二个要求是将类文件移动到适当的包目录中。系统中的某个地方必须存在一个反映包名的目录结构。例如，对于包名`employee.benefits`，需要有一个名为`employee`的目录，其中有一个名为`benefits`的子目录。`employee`包的所有类文件都放在`employee`目录中。`employee.benefits`包的所有类文件都放在`benefits`子目录中。下图显示了这一点，其中目录和文件位于`C`驱动器的某个地方：

![包的目录/文件组织](img/7324_09_02.jpg)

您可能还会发现一个包的目录和类被压缩成一个**Java 存档**（**JAR**）或`.jar`文件。如果您在目录系统中寻找特定的包结构，可能会找到一个 JAR 文件。通过将包压缩成 JAR 文件，可以最小化内存。如果找到这样的文件，请不要解压缩，因为 Java 编译器和 JVM 希望它们在 JAR 文件中。

大多数集成开发环境将源文件和类文件分开放置在不同的目录中。这种分离使它们更容易处理和部署。

## 导入语句

`import`语句向编译器提供了关于在程序中使用的类的定义在哪里找到的信息。关于导入语句有几点需要考虑，我们将在下文中进行讨论：

+   它的使用是可选的。

+   使用通配符字符

+   访问具有相同名称的多个类

+   静态导入语句

### 避免使用 import 语句

`import`语句是可选的。在下面的例子中，我们没有使用`import`语句来导入`BigDecimal`类，而是直接在代码中明确使用包名：

```java
private java.math.BigDecimal balance;
     …
this.balance = new java.math.BigDecimal("0");
```

这更冗长，但更具表现力。它毫无疑问地表明`BigDecimal`类是在`java.math`包中找到的。但是，如果我们在程序中多次使用该类，这将变得很烦人。通常会使用`import`语句。

### 使用导入语句

为了避免每个类都要用其包名作为前缀，`import`语句可以用来指示编译器类的位置。在这个例子中，`java.io`包的`BufferedReader`类可以在使用时不必每次都用其包名作为前缀：

```java
import java.io.BufferReader;
   …
   BufferedReader br = new BufferedReader();
```

### 使用通配符字符

如果需要使用多个类，并且它们在同一个包中，可以使用星号代替包括多个导入语句，每个类一个。例如，如果我们需要在应用程序中同时使用`BufferedReader`和`BufferedWriter`类，可以使用两个导入语句，如下所示：

```java
import java.io.BufferedReader;
import java.io.BufferedWriter;
```

通过显式列出每个类，代码的读者将立即知道在哪里找到该类。否则，当通配符字符与多个导入语句一起使用时，读者可能会猜测一个类来自哪个包。

虽然每个类的显式导入更好地记录了文档，但导入列表可能会变得很长。大多数集成开发环境支持折叠或隐藏列表的功能。

另一种方法是使用带有星号的单个导入语句，如下所示：

```java
import java.io.*;
```

现在可以在不使用包名的情况下使用该包的所有元素。但是，这并不意味着子包的类可以以同样的方式使用。例如，有许多以`java.awt`开头的包。以下图表显示了其中一些及其元素：

![使用通配符字符](img/7324_09_03.jpg)

当针对“基本”包使用通配符字符时，可能会觉得通配符字符应该包括这些附加包中找到的类，如下面的代码所示：

```java
import java.awt.*;
```

但是，它只导入`java.awt`包中的那些类，而不导入`java.awt.font`或类似包中的任何类。为了还引用`java.awt.font`的所有类，需要第二个导入语句：

```java
import java.awt.*;
import java.awt.font.*;
```

### 具有相同名称的多个类

由于可能在不同的包中有多个同名类，因此导入语句用于指定要使用的类。但是，第二个类将需要显式使用包名称。

例如，假设我们在`com.company.account`包中创建了一个`BigDecimal`类，并且我们需要使用它和`java.math.BigDecimal`类。我们不能为两个类使用导入，如下面的代码片段所示，因为这将生成一个语法错误，指出名称冲突。

```java
import java.math.BigDecimal;
import com.company.customer.BigDecimal;
```

相反，我们需要：

+   使用导入语句声明一个，并在使用时显式前缀第二个类名，或者

d.根本不使用导入语句，并在使用它们时显式前缀两个类

假设我们使用`import`语句与`java.math`类，我们在代码中使用两个类，如下所示：

```java
this.balance = new BigDecimal("0");
com.company.customer.BigDecimal secondary = 
   new com.company.customer.BigDecimal();
```

请注意，我们必须在第二个语句中为`BigDecimal`的两个用法添加前缀，否则它会假定未加前缀的那个在`java.math`包中，从而生成类型不匹配的语法错误。

### 静态导入语句

静态导入语句可用于简化方法的使用。这通常与`println`方法一起使用。在下面的示例中，我们多次使用`println`方法：

```java
System.out.println("Employee Information");
System.out.println("Name: ");
System.out.println("Department: ");
System.out.println("Pay grade: ");
```

在每种情况下，都需要`System`类名。但是，如果我们使用以下`import`语句，其中添加了`static`关键字，我们将不需要使用`System`类名：

```java
import static java.lang.System.out;
```

以下代码语句序列实现了相同的结果：

```java
out.println("Employee Information");
out.println("Name: ");
out.println("Department: ");
out.println("Pay grade: ");   
```

虽然这种方法节省了输入时间，但对于不了解静态导入语句的人来说可能会感到困惑。

## 垃圾回收

Java 执行自动垃圾回收。使用`new`关键字分配内存时，内存是从程序堆中获取的。这是程序堆栈上方的内存区域。分配的对象将由程序保留，直到程序释放它。这是通过删除对对象的所有引用来完成的。一旦释放，垃圾回收例程最终将运行并回收对象分配的内存。

以下代码序列说明了如何创建`String`对象。然后将其分配给第二个引用变量：

```java
String s1 = new String("A string object");
String s2 = s1;
```

此时，`s1`和`s2`都引用字符串对象。以下图表说明了`s1`和`s2`的内存分配：

![垃圾回收](img/7324_09_04.jpg)

在这种情况下，使用`new`关键字确保从堆中分配了字符串对象。如果我们使用字符串文字，如下所示，对象将分配给内部池，如第二章中的*字符串比较*部分所讨论的那样，*Java 数据类型及其用法*：

```java
String s1 = "A string object";
```

下面的两个语句说明了如何删除对对象的引用：

```java
s1 = null;
s2 = null;
```

执行这些语句后，应用程序的状态如下图所示：

![垃圾回收](img/7324_09_05.jpg)

存在一个 JVM 后台线程，定期执行以回收未使用的对象。在将来的某个时候，线程将执行。当对象准备回收时，线程将执行以下操作：

+   执行方法的`finalize`方法

+   回收内存以供堆管理器重用

`finalize`方法通常不由开发人员实现。它最初的目的是对应于诸如 C++之类的语言中找到的析构函数。它们用于执行清理活动。

在 Java 中，不应依赖于方法执行。对于小型程序，垃圾回收例程可能永远不会运行，因为程序可能在有机会执行之前终止。多年来，已经尝试了几次提供程序员强制执行方法的能力。这些尝试都没有成功。

# 资源包和 Locale 类

`Locale`类用于表示世界的一部分。与区域关联的是一组与货币或日期显示方式控制相关的约定。使用区域有助于应用的国际化。开发人员指定区域，然后在应用的各个部分中使用该区域。

除了`Locale`类，我们还可以使用资源包。它们提供了一种根据区域设置自定义外观的方式，适用于除数字和日期之外的数据类型。在处理根据区域设置更改的字符串时特别有用。

例如，GUI 应用程序将具有不同的可视组件，其文本在世界不同地区使用时应有所不同。在西班牙，文本和货币应以西班牙语显示。在中国，应使用中文字符和约定。使用区域设置可以简化将应用程序适应世界不同地区的过程。

在本节中，我们将讨论用于支持应用国际化的三种方法：

+   使用`Locale`类

+   使用属性资源文件

+   使用`ListResourceBundle`类

## 使用 Locale 类

为了说明区域设置的使用，我们首先创建`Locale`类的实例。该类具有许多预定义的区域设置常量。在以下示例中，我们将创建一个美国的区域设置，然后显示该区域设置：

```java
Locale locale;

locale = Locale.US;
System.out.println(locale);
```

输出如下：

```java
en_US

```

第一部分`en_`代表英语。第二部分指定为美国。如果我们将区域设置更改为德国如下：

```java
locale = Locale.GERMANY;
System.out.println(locale);
```

您将得到以下输出：

```java
de_DE

```

您可以使用区域设置格式化货币值。在以下示例中，我们使用静态的`getCurrencyInstance`方法使用美国的区域返回`NumberFormat`类的实例。然后使用`format`方法对双精度数进行格式化：

```java
NumberFormat currencyFormatter = 
   NumberFormat.getCurrencyInstance(Locale.US);
System.out.println(currencyFormatter.format(23.45));
```

输出如下：

```java
$23.45

```

如果我们使用德国区域设置，将得到以下输出：

```java
23,45 €

```

日期也可以根据区域设置进行格式化。在以下代码片段中，使用`DateFormat`类的`getDateInstance`方法，使用美国的区域设置。`format`方法使用`Date`对象获取日期的字符串表示，如以下代码片段所示：

```java
DateFormat dateFormatter = 
   DateFormat.getDateInstance(DateFormat.LONG, Locale.US);
System.out.println(dateFormatter.format(new Date()));
```

输出将类似于以下内容：

```java
May 2, 2012

```

在以下代码片段中，我们将使用法国的区域设置：

```java
dateFormatter = DateFormat.getDateInstance(
   DateFormat.LONG, Locale.FRANCE);
System.out.println(dateFormatter.format(new Date()));
```

此示例的输出如下：

```java
2 mai 2012

```

## 使用资源包

资源包是按区域组织的对象集合。例如，我们可能有一个资源包，其中包含英语用户的字符串和 GUI 组件，另一个资源包适用于西班牙语用户。这些语言组可以进一步分为语言子组，例如美国英语用户与加拿大英语用户。

资源包可以存储为文件，也可以定义为类。属性资源包存储在`.properties`文件中，仅限于字符串。`ListResourceBundle`是一个类，可以保存字符串和其他对象。

### 使用属性资源包

属性资源包是一个文件，包含一组以键值对形式存在的字符串，文件名以`.properties`结尾。字符串键用于标识特定的字符串值。例如，`WINDOW_CAPTION`键可以与字符串值`Editor`相关联。以下是`ResourceExamples.properties`文件的内容：

```java
WINDOW_CAPTION=Editor
FILE_NOT_FOUND=The file could not be found
FILE_EXISTS=The file already exists
UNKNOWN=Unknown problem with application

```

要访问资源文件中的值，我们需要创建一个`ResourceBundle`类的实例。我们可以使用`ResourceBundle`类的静态`getBundle`方法来实现这一点，如下面的代码片段所示。请注意，资源文件名被用作方法的参数，但不包括文件扩展名。如果我们知道键，我们可以使用`getString`方法来返回其对应的值：

```java
ResourceBundle bundle = ResourceBundle.getBundle(
      "ResourceExamples");
System.out.println("UNKNOWN" + ":" +
      bundle.getString("UNKNOWN"));
```

输出将如下所示：

```java
UNKNOWN:Unknown problem with application

```

我们可以使用`getKeys`方法来获取一个`Enumeration`对象。如下面的代码片段所示，用于显示文件的所有键值对的枚举：

```java
ResourceBundle bundle = ResourceBundle.getBundle(
      "ResourceExamples");

Enumeration keys = bundle.getKeys();
while (keys.hasMoreElements()) {
   String key = (String) keys.nextElement();
   System.out.println(key + ":" + bundle.getString(key));
}
```

这个序列的输出如下：

```java
FILE_NOT_FOUND:The US file could not be found
UNKNOWN:Unknown problem with application
FILE_EXISTS:The US file already exists
WINDOW_CAPTION:Editor
```

请注意，输出与`ResourceExamples.properties`文件的顺序或内容不匹配。顺序由枚举控制。`FILE_NOT_FOUND`和`FILE_EXISTS`键的内容不同。这是因为它实际上使用了不同的文件，`ResourceExamples_en_US.properties`。属性资源包之间存在层次关系。代码序列是在默认区域设置为美国的情况下执行的。系统查找`ResourceExamples_en_US.properties`文件，因为它代表特定于该区域设置的字符串。资源文件中的任何缺失元素都会从其“基本”文件中继承。

我们将创建四个不同的资源包文件，以说明资源包的使用以及它们之间的层次关系： 

+   `ResourceExamples.properties`

+   `ResourceExamples_en.properties`

+   `ResourceExamples_en_US.properties`

+   `ResourceExamples_sp.properties`

这些在层次上是相关的，如下图所示：

![使用属性资源包](img/7324_09_06.jpg)

这些文件将包含四个键的字符串，如下表所示：

| 文件 | 键 | 值 |
| --- | --- | --- |
|   | `WINDOW_CAPTION` | 编辑器 |
| `FILE_NOT_FOUND` | 文件找不到 |
| `FILE_EXISTS` | 文件已经存在 |
| `UNKNOWN` | 应用程序出现未知问题 |
| `en` | `WINDOW_CAPTION` | 编辑器 |
| `FILE_NOT_FOUND` | 英文文件找不到 |
| `UNKNOWN` | 应用程序出现未知问题 |
| `en_US` | `WINDOW_CAPTION` | 编辑器 |
| `FILE_NOT_FOUND` | 美国文件找不到 |
| `FILE_EXISTS` | 美国文件已经存在 |
| `UNKNOWN` | 应用程序出现未知问题 |
| `sp` | `FILE_NOT_FOUND` | El archivo no se pudo encontrar |
| `FILE_EXISTS` | El archivo ya existe |
| `UNKNOWN` | Problema desconocido con la aplicación |

`en`条目缺少`FILE_EXISTS`键的值，`sp`条目缺少`WINDOW_CAPTION`键。它们将继承默认资源文件的值，如下所示，对于`en`区域设置：

```java
bundle = ResourceBundle.getBundle("ResourceExamples",
      new Locale("en"));
System.out.println("en");
keys = bundle.getKeys();
while (keys.hasMoreElements()) {
   String key = (String) keys.nextElement();
   System.out.println(key + ":" + bundle.getString(key));
}
```

输出列出了`FILE_EXISTS`的值，即使它在`ResourceExamples_en.properties`文件中找不到：

```java
en
WINDOW_CAPTION:Editor
FILE_NOT_FOUND:The English file could not be found
UNKNOWN:Unknown problem with application
FILE_EXISTS:The file already exists

```

这些文件的继承行为允许开发人员基于基本文件名创建资源文件的层次结构，然后通过添加区域设置后缀来扩展它们。这将导致自动使用特定于当前区域设置的字符串。如果需要的区域设置不同于默认区域设置，则可以指定特定的区域设置。

### 使用`ListResourceBundle`类

`ListResourceBundle`类也用于保存资源。它不仅可以保存字符串，还可以保存其他类型的对象。但是，键仍然是字符串值。为了演示这个类的使用，我们将创建一个从`ListResourceBundle`类派生的`ListResource`类，如下所示。创建一个包含键值对的静态二维对象数组。请注意，最后一对包含一个`ArrayList`。类的`getContents`方法将资源作为二维对象数组返回：

```java
public class ListResource extends ListResourceBundle {

   @Override
   protected Object[][] getContents() {
      return resources;
   }

   static Object[][] resources = {
      {"FILE_NOT_FOUND", "The file could not be found"},
      {"FILE_EXISTS", "The file already exists"},
      {"UNKNOWN", "Unknown problem with application"},
      {"PREFIXES",new 
            ArrayList(Arrays.asList("Mr.","Ms.","Dr."))}

   };
}
```

创建的`ArrayList`旨在存储各种名称前缀。它使用`asList`方法创建，该方法传递可变数量的字符串参数，并将一个`List`返回给`ArrayList`构造函数。

以下代码演示了如何使用`ListResource`。创建了一个`ListResource`的实例，然后使用字符串键执行了`getString`方法。对于`PREFIXES`键，使用了`getObject`方法：

```java
System.out.println("ListResource");
ListResource listResource = new ListResource();

System.out.println(
   listResource.getString("FILE_NOT_FOUND"));
System.out.println(
   listResource.getString("FILE_EXISTS"));
System.out.println(listResource.getString("UNKNOWN"));
ArrayList<String> salutations = 
       (ArrayList)listResource.getObject("PREFIXES");
for(String salutation : salutations) {
   System.out.println(salutation);
}

```

此序列的输出如下：

```java
ListResource
The file could not be found
The file already exists
Unknown problem with application
Mr.
Ms.
Dr.
```

# 使用 JDBC

JDBC 用于连接到数据库并操作数据库中的表。使用 JDBC 的过程包括以下步骤：

1.  连接到数据库

1.  创建要提交到数据库的 SQL 语句

1.  处理生成的结果和任何异常

在 Java 7 中，使用 JDBC 已经通过添加 try-with-resources 块得到增强，该块简化了连接的打开和关闭。有关此块的详细说明，请参见第八章中的“使用 try-with-resource 块”部分。

## 连接到数据库

连接到数据库涉及两个步骤：

1.  加载适当的驱动程序

1.  建立连接

这假设数据库已经设置并且可访问。在以下示例中，我们将使用 MySQL Version 5.5。MySQL 带有包含`customer`表的`Sakila`模式。我们将使用此表来演示各种 JDBC 技术。

### 加载适当的驱动程序

首先我们需要加载一个驱动程序。 JDBC 支持多种驱动程序，如[`developers.sun.com/product/jdbc/drivers`](http://developers.sun.com/product/jdbc/drivers)中所讨论的。在这里，我们将使用`MySQLConnector/J`驱动程序。我们使用`Class`类的`forName`方法加载驱动程序，如下面的代码片段所示：

```java
try {
   Class.forName(
            "com.mysql.jdbc.Driver").newInstance();
} catch (InstantiationException | 
         IllegalAccessException |
         ClassNotFoundException e) {
   e.printStackTrace();
}
```

该方法会抛出几个需要捕获的异常。

请注意，从 JDBC 4.0 开始，不再需要上述序列，假设使用的 JDBC 驱动程序支持 JDBC 4.0。这适用于与 MySQL Version 5.5 一起使用的 MySQL 驱动程序。这里使用此序列是因为您可能会在旧程序中遇到这种方法。

### 建立连接

接下来，需要建立与数据库的连接。 `java.sql.Connection`表示与数据库的连接。 `DriverManager`类的静态`getConnection`方法将返回与数据库的连接。它的参数包括：

+   代表数据库的 URL

+   用户 ID

+   一个密码

以下代码序列将使用 try-with-resources 块来建立与数据库的连接。第一个参数是特定于 MySQL 的连接字符串。连接字符串是供应商特定的：

```java
try (Connection connection = DriverManager.getConnection(
      "jdbc:mysql://localhost:3306/", "id", "password")) {
         ...
} catch (SQLException e) {
   e.printStackTrace();
}
```

## 创建 SQL 语句

接下来，我们需要创建一个`Statement`对象，用于执行查询。 `Connection`类的`createStatement`方法将返回一个`Statement`对象。我们将把它添加到 try-with-resources 块中以创建对象：

```java
try (Connection connection = DriverManager.getConnection(
      "jdbc:mysql://localhost:3306/", "root", "explore");
     Statement statement = connection.createStatement()) {
      ...
} catch (SQLException e) {
   e.printStackTrace();
}
```

然后形成一个查询字符串，该字符串将选择`customer`表中`address_id`小于 10 的那些客户的名字和姓氏。我们选择此查询以最小化结果集的大小。使用`executeQuery`方法执行查询并返回一个包含与所选查询匹配的表的行的`ResultSet`对象：

```java
try (Connection connection = DriverManager.getConnection(
      "jdbc:mysql://localhost:3306/", "root", "explore");
     Statement statement = connection.createStatement()) {
      String query = "select first_name, last_name"
         + " from sakila.customer "
         + "where address_id < 10";
      try (ResultSet resultset = 
                        statement.executeQuery(query)) {
         ...
      }
         ...
} catch (SQLException e) {
   e.printStackTrace();
}
```

## 处理结果

最后一步是使用 while 循环遍历结果集并显示返回的行。在下面的示例中，`next`方法将在`resultset`中从一行移到另一行。`getString`方法返回与指定要访问的列对应的值的方法参数：

```java
try (Connection connection = DriverManager.getConnection(
      "jdbc:mysql://localhost:3306/", "root", "explore");
     Statement statement = connection.createStatement()) {
      String query = "select first_name, last_name"
         + " from sakila.customer "
         + "where address_id < 10";
      try (ResultSet resultset = 
                        statement.executeQuery(query)) {
         while (resultset.next()) {
            String firstName = 
                       resultset.getString("first_name");
            String lastName = 
                       resultset.getString("last_name");
            System.out.println(firstName + " " + lastName);
         }
      }
} catch (SQLException e) {
   e.printStackTrace();
}
```

输出如下：

```java
MARY SMITH
PATRICIA JOHNSON
LINDA WILLIAMS
BARBARA JONES
ELIZABETH BROWN

```

JDBC 支持其他 SQL 语句的使用，如`update`和`delete`。此外，它支持使用参数化查询和存储过程。

# 摘要

在本章中，我们重新审视了 Java 应用程序的整体结构。我们讨论了`import`和`package`语句的使用，并讨论了包库与其支持的目录/文件之间的关系。我们学习了如何在`import`语句中使用星号通配符。此外，我们看到了静态导入语句的使用。

我们讨论了初始化程序列表的使用以及 Java 中垃圾回收的工作原理。这个过程导致对象在不再需要时自动回收。

我们探讨了国际化的支持，从`Locale`类开始，然后是资源包。我们讨论了属性资源包和`ListResourceBundle`类。我们学习了当使用一致的命名约定组织时，属性资源包的继承如何工作。

最后，我们讨论了 JDBC 的使用。我们看到需要驱动程序来建立与数据库的连接，以及如何使用`Statement`类来检索`ResultSet`对象。这个对象允许我们遍历`select`查询返回的行。

# 涵盖的认证目标

本章涵盖的认证目标包括：

+   定义 Java 类的结构

+   根据区域设置选择资源包

+   使用适当的 JDBC API 提交查询并从数据库中读取结果。

# 测试你的知识

1.  以下哪个将不会出现编译错误？

a. `package somepackage;`

`import java.nio.*;`

`class SomeClass {}`

b. `import java.nio.*;`

`package somepackage;`

`class SomeClass {}`

c. `/*这是一个注释*/`

`package somepackage;`

`import java.nio.*;`

`class SomeClass {}`

1.  对于资源属性文件的层次结构，如果一个键在一个派生文件中丢失，根据丢失的键，以下哪些关于返回的值是真的？

a. 返回值将是一个空字符串

b. 返回值将是一个空值

c. 返回值将是来自基本资源包的字符串

d. 会抛出运行时异常

1.  `forName`方法不会抛出哪个异常：

a. `InstantiationException`

b. `ClassNotFoundException`

c. `ClassDoesNotExistException`

d. `IllegalAccessException`
