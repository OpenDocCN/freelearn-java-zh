# 模块化编程

在本章中，我们将涵盖以下技巧：

+   使用 jdeps 在 Java 应用程序中查找依赖关系

+   创建一个简单的模块化应用程序

+   创建一个模块化 JAR

+   在使用 Pre-Project Jigsaw JDK 应用程序中使用模块 JAR

+   自下而上的迁移

+   自上而下的迁移

+   使用服务来创建消费者和提供者模块之间的松耦合

+   使用 jlink 创建自定义模块化运行时映像

+   为旧平台版本编译

+   创建多版本 JAR

+   使用 Maven 开发模块化应用程序

+   使您的库对模块路径友好

+   如何为反射打开一个模块

# 介绍

模块化编程使我们能够将代码组织成独立的、内聚的模块，这些模块可以组合在一起以实现所需的功能。这使我们能够创建代码：

+   更加内聚，因为模块是为特定目的构建的，所以驻留在那里的代码往往倾向于迎合特定目的。

+   封装，因为模块只能与其他模块提供的 API 进行交互。

+   可靠，因为可发现性是基于模块而不是基于个别类型的。这意味着如果一个模块不存在，依赖的模块在被依赖的模块发现之前无法执行。这有助于防止运行时错误。

+   松耦合。如果使用服务接口，模块接口和服务接口实现可以松散耦合。

因此，在设计和组织代码的思考过程中，现在将涉及识别模块、代码和配置文件进入模块以及代码在模块内部组织的包。之后，我们必须决定模块的公共 API，从而使它们可供依赖模块使用。

关于**Java 平台模块系统**的开发，它由**Java 规范请求**（JSR）376（[`www.jcp.org/en/jsr/detail?id=376`](https://www.jcp.org/en/jsr/detail?id=376)）进行管理。JSR 提到，模块系统应该解决以下基本问题：

+   **可靠的配置**：提供了一个替代类路径的方式来声明组件之间的依赖关系，使开发人员可以防止他们的应用程序由于类路径中缺少依赖关系而在运行时出现意外。

+   **强封装**：提供更严格的访问控制，使组件中的私有内容真正私有，即使通过反射也无法访问，并允许开发人员有选择地公开组件中的部分内容供其他组件使用。

JSR 列出了解决上述问题所带来的优势：

+   **可扩展的平台**：JSR 376 中的规范将允许通过使用新平台创建的不同组件/模块来正确地利用 JSR 337 中引入的不同配置文件，从而创建配置文件。这个模块化平台还将允许其他开发人员打包 Java 平台的不同组件，以创建自定义运行时，从而为他们提供创建仅适合他们使用的运行时的选项。

+   **更大的平台完整性**：强封装将阻止有意或意外地使用 Java 内部 API，从而提供更大的平台完整性。

+   **性能提升**：由于组件之间的明确依赖关系，现在更容易根据它们在 Java SE 平台内部和外部交互的组件来优化各个组件。

在本章中，我们将介绍一些重要的技巧，帮助您开始模块化编程。

# 使用 jdeps 在 Java 应用程序中查找依赖关系

模块化应用程序的第一步是识别其依赖关系。JDK 8 引入了一个名为`jdeps`的静态分析工具，使开发人员能够找到其应用程序的依赖关系。命令中支持多个选项，使开发人员能够检查 JDK 内部 API 的依赖关系，显示包级别的依赖关系，显示类级别的依赖关系，并过滤依赖关系等。

在这个示例中，我们将探讨如何使用`jdeps`工具来探索其功能，并使用它支持的多个命令行选项。

# 准备就绪

我们需要一个示例应用程序，可以针对`jdeps`命令运行以查找其依赖关系。因此，我们考虑创建一个非常简单的应用程序，使用 Jackson API 来消耗来自 REST API 的 JSON：[`jsonplaceholder.typicode.com/users`](http://jsonplaceholder.typicode.com/users)。

在示例代码中，我们还添加了对已弃用的 JDK 内部 API`sun.reflect.Reflection.getCallerClass()`的调用。这样，我们可以看到`jdeps`如何帮助找到 JDK 内部 API 的依赖关系。

以下步骤将帮助您设置此示例的先决条件：

1.  您可以从`Chapter03/1_json-jackson-sample`获取示例的完整代码。我们已经针对 Java 9 构建了这段代码，也使用了 Java 8，并且编译成功。因此，您只需要安装 Java 9 来进行编译。如果您尝试使用 JDK 11 进行编译，由于已弃用的内部 API 不再可用，您将遇到错误。

1.  一旦您有了代码，就使用以下命令进行编译：

```java
 # On Linux
 javac -cp 'lib/*' -d classes 
            -sourcepath src $(find src -name *.java)

 # On Windows
 javac -cp lib*;classes 
            -d classes src/com/packt/model/*.java
                       src/com/packt/*.java
```

注意：如果您的`javac`指向 JDK 11，您可以声明环境变量，如`JAVA8_HOME`或`JAVA9_HOME`，分别指向您的 JDK 8 和 JDK9 安装。这样，您可以使用以下命令进行编译：

```java
# On Linux
"$JAVA8_HOME"/bin/javac -cp 'lib/*' 
   -d classes -sourcepath src $(find src -name *.java)

# On Windows
"%JAVA8_HOME%"\bin\javac -cp lib\*;classes 
       -d classes src\com\packt\*.java 
                 src\com\packt\model\*.java
```

您将看到有关使用内部 API 的警告，您可以放心地忽略。我们添加了这个目的是为了演示`jdeps`的功能。现在，您应该在 classes 目录中有已编译的类文件。

1.  您可以创建一个可执行的 JAR 文件，并通过以下命令运行示例程序：

```java
 # On Linux:
        jar cvfm sample.jar manifest.mf -C classes .
        "$JAVA8_HOME"/bin/java -jar sample.jar
 # On Windows:
        jar cvfm sample.jar manifest.mf -C classes .
 "%JAVA8_HOME%"\bin\java -jar sample.jar
```

1.  我们在`Chapter03/1_json-jackson-sample`中提供了`run.bat`和`run.sh`脚本。您也可以使用这些脚本进行编译和运行。

如果您使用`run.bat`或`run.sh`或上述命令创建 JAR，则会在当前目录中创建一个`sample.jar`文件。如果 JAR 尚未创建，您可以使用`build-jar.bat`或`build.-jar.sh`脚本来编译和构建 JAR。

因此，我们有一个非模块化的示例应用程序，我们将使用`jdeps`来分析其依赖关系，以及可能依赖的模块名称。

# 如何做...

1.  使用`jdeps`的最简单方法如下：

```java
# On Linux
jdeps -cp classes/:lib/* classes/com/packt/Sample.class

# On Windows
jdeps -cp "classes/;lib/*" classes/com/packt/Sample.class 
```

上述命令等同于以下命令：

```java
# On Linux jdeps -verbose:package -cp classes/:lib/*
      classes/com/packt/Sample.class

# On Windows
jdeps -verbose:package -cp "classes/;lib/*" classes/com/packt/Sample.class 
```

上述代码的输出如下：

![](img/c3c8f969-df9f-43a6-826c-ace8f1ea5606.png)

在上述命令中，我们使用`jdeps`来列出包级别的类文件`Sample.class`的依赖关系。我们必须提供`jdeps`要搜索的代码依赖的路径。这可以通过设置`jdeps`命令的`-classpath`、`-cp`或`--class-path`选项来实现。

`-verbose:package`选项列出了包级别的依赖关系。

1.  让我们列出类级别的依赖关系：

```java
# On Linux      
jdeps -verbose:class -cp classes/:lib/* 
classes/com/packt/Sample.class

# On Windows
jdeps -verbose:class -cp "classes/;lib/*" classes/com/packt/Sample.class 
```

上述命令的输出如下：

![](img/7a210b16-f42b-4e6f-b5af-7ed6d664521f.png)

在这种情况下，我们利用`-verbose:class`选项来列出类级别的依赖关系，这就是为什么你可以看到`com.packt.Sample`类依赖于`com.packt.model.Company`、`java.lang.Exception`、`com.fasterxml.jackson.core.type.TypeReference`等等。

1.  让我们得到依赖关系的摘要：

```java
# On Linux
jdeps -summary -cp classes/:lib/* 
                   classes/com/packt/Sample.class

# On Windows
jdeps -summary -cp "classes/;lib/*" 
                    classes/com/packt/Sample.class 
```

输出如下：

![](img/ebc43ab3-e07e-4853-bc4f-a86864ee9961.png)

1.  让我们检查对 JDK 内部 API 的依赖：

```java
# On Linux
jdeps -jdkinternals -cp classes/:lib/*
 classes/com/packt/Sample.class

# On Windows
jdeps -jdkinternals -cp "classes/;lib/*"   
                         classes/com/packt/Sample.class 
```

以下是上述命令的输出：

![](img/5f7a9c3e-554a-46d7-b7fd-6920ddf9b0ae.png)

StackWalker API 是 Java 9 中引入的用于遍历调用堆栈的新 API。这是`sun.reflect.Reflection.getCallerClass()`方法的替代品。我们将在第十一章中讨论此 API，即*内存管理和调试*。

1.  让我们对 JAR 文件`sample.jar`运行`jdeps`：

```java
# On Linux and Windows
      jdeps -s -cp lib/* sample.jar
```

我们得到的输出如下：

![](img/f30a59fd-1ed6-4b4c-8ea2-e4c3bf8fc189.png)

通过使用`jdeps`对`sample.jar`进行调查获得的信息非常有用。它清楚地说明了我们的 JAR 文件的依赖关系，在我们尝试将该应用程序迁移到模块化应用程序时非常有用。

1.  让我们找出是否有任何依赖于给定包名称的依赖项：

```java
# On Linux and Windows
      jdeps -p java.util sample.jar
```

输出如下：

![](img/593012f7-f4cf-4d5c-b331-485c571a9205.png)

`-p`选项用于查找对给定包名称的依赖关系。因此，我们知道我们的代码依赖于`java.util`包。让我们尝试另一个包名称：

```java
 jdeps -p java.util.concurrent sample.jar
```

没有输出，这意味着我们的代码不依赖于`java.util.concurrent`包。

1.  我们只想对我们的代码运行依赖性检查。是的，这是可能的。假设我们运行`jdeps -cp lib/* sample.jar`；你会看到甚至库 JAR 也被分析了。我们不想要那样，对吧？让我们只包括`com.packt`包的类：

```java
# On Linux 
jdeps -include 'com.packt.*' -cp lib/* sample.jar

# On Windows
jdeps -include "com.packt.*" -cp lib/* sample.jar 
```

输出如下：

![](img/bfb4eb40-2b7b-4516-939f-6ba305d424e2.png)

1.  让我们检查我们的代码是否依赖于特定包：

```java
# On Linux
jdeps -p 'com.packt.model' sample.jar

# On Windows
jdeps -p "com.packt.model" sample.jar 
```

输出如下：

![](img/c77a08a3-044a-4c1d-ae1d-636d2331977d.png)

1.  我们可以使用`jdeps`来分析 JDK 模块。让我们选择`java.httpclient`模块进行分析：

```java
 jdeps -m java.xml
```

以下是输出：

![](img/09e20526-032c-4210-952a-8c9283d429ce.png)

我们还可以通过使用`--require`选项找出一个给定模块是否依赖于另一个模块，如下所示：

```java
# On Linux and Windows
      jdeps --require java.logging -m java.sql
```

以下是输出：

![](img/82ffa439-4881-4ad7-b4a8-9e169b4ed907.png)

在上述命令中，我们试图找出`java.sql`模块是否依赖于`java.logging`模块。我们得到的输出是`java.sql`模块的依赖摘要和`java.sql`模块中使用`java.logging`模块代码的包。

# 它是如何工作的...

`jdeps`命令是一个静态类依赖性分析器，用于分析应用程序及其库的静态依赖关系。`jdeps`命令默认显示输入文件（可以是`.class`文件、目录或 JAR 文件）的包级别依赖关系。这是可配置的，并且可以更改为显示类级别的依赖关系。有多个选项可用于过滤依赖关系并指定要分析的类文件。我们已经看到了`-cp`命令行选项的常规用法。此选项用于提供要搜索分析代码依赖关系的位置。

我们已经分析了类文件、JAR 文件和 JDK 模块，并尝试了`jdeps`命令的不同选项。有一些选项，如`-e`、`-regex`、`--regex`、`-f`、`--filter`和`-include`，可以接受正则表达式（regex）。了解`jdeps`命令的输出是很重要的。对于每个被分析的类/JAR 文件，都有两部分信息：

1.  分析文件（JAR 或类文件）的依赖摘要。左侧是类或 JAR 文件的名称，右侧是依赖实体的名称。依赖实体可以是目录、JAR 文件或 JDK 模块，如下所示：

```java
 Sample.class -> classes
      Sample.class -> lib/jackson-core-2.9.6.jar
      Sample.class -> lib/jackson-databind-2.9.6.jar
      Sample.class -> java.base
      Sample.class -> jdk.unsupported
```

1.  在包或类级别（取决于命令行选项）分析文件内容的更详细的依赖信息。这由三列组成——第 1 列包含包/类的名称，第 2 列包含依赖包的名称，第 3 列包含找到依赖项的模块/JAR 的名称。示例输出如下：

```java
 com.packt  -> com.fasterxml.jackson.core.type  
                    jackson-core-2.9.6.jar
      com.packt  -> com.fasterxml.jackson.databind   
                    jackson-databind-2.9.6.jar
      com.packt  -> com.packt.model  sample.jar
```

# 还有更多...

我们已经看到了`jdeps`命令的许多选项。还有一些与过滤依赖项和过滤要分析的类相关的选项。除此之外，还有一些处理模块路径的选项。

以下是可以尝试的选项：

+   `-e`，`-regex`，`--regex`：这些找到与给定模式匹配的依赖项。

+   `-f`，`-filter`：这些排除与给定模式匹配的依赖项。

+   `-filter:none`：这允许不应用`filter:package`或`filter:archive`的过滤。

+   `-filter:package`：这排除了同一包内的依赖项。这是默认选项。例如，如果我们向`jdeps sample.jar`添加`-filter:none`，它将打印包对自身的依赖项。

+   `-filter:archive`：这排除了同一存档内的依赖项。

+   `-filter:module`：这排除了同一模块中的依赖项。

+   `-P`，`-profile`：用于显示包的配置文件，无论它是在 compact1、compact2、compact3 还是完整的 JRE 中。

+   `-R`，`-recursive`：这些递归遍历所有运行时依赖项；它们等同于`-filter:none`选项。

# 创建一个简单的模块化应用程序

您一定想知道模块化是什么，以及如何在 Java 中创建模块化应用程序。在这个示例中，我们将尝试通过一个简单的示例来澄清在 Java 中创建模块化应用程序的困惑。我们的目标是向您展示如何创建一个模块化应用程序；因此，我们选择了一个简单的示例，以便专注于我们的目标。

我们的示例是一个简单的高级计算器，用于检查数字是否为质数，计算质数的和，检查数字是否为偶数，并计算偶数和奇数的和。

# 准备就绪

我们将应用程序分为两个模块：

+   包含用于执行数学计算的 API 的`math.util`模块

+   启动高级计算器的`calculator`模块

# 如何做...

1.  让我们在`com.packt.math.MathUtil`类中实现 API，从`isPrime(Integer number)`API 开始：

```java
        public static Boolean isPrime(Integer number){
          if ( number == 1 ) { return false; }
          return IntStream.range(2,num).noneMatch(i -> num % i == 0 );
        }
```

1.  实现`sumOfFirstNPrimes(Integer count)`API：

```java
        public static Integer sumOfFirstNPrimes(Integer count){
          return IntStream.iterate(1,i -> i+1)
                          .filter(j -> isPrime(j))
                          .limit(count).sum();
        }
```

1.  让我们编写一个函数来检查数字是否为偶数：

```java
        public static Boolean isEven(Integer number){
          return number % 2 == 0;
        }
```

1.  `isEven`的否定告诉我们数字是否为奇数。我们可以有函数来找到前*N*个偶数和前*N*个奇数的和，如下所示：

```java
        public static Integer sumOfFirstNEvens(Integer count){
          return IntStream.iterate(1,i -> i+1)
                          .filter(j -> isEven(j))
                          .limit(count).sum();
        }

        public static Integer sumOfFirstNOdds(Integer count){
          return IntStream.iterate(1,i -> i+1)
                          .filter(j -> !isEven(j))
                          .limit(count).sum();
        }
```

我们可以看到在前面的 API 中重复了以下操作：

+   从`1`开始的无限序列

+   根据某些条件过滤数字

+   将数字流限制为给定数量

+   找到因此获得的数字的和

根据我们的观察，我们可以重构前面的 API，并将这些操作提取到一个方法中，如下所示：

```java
Integer computeFirstNSum(Integer count,
                                 IntPredicate filter){
  return IntStream.iterate(1,i -> i+1)
                  .filter(filter)
                  .limit(count).sum();
 }
```

在这里，`count`是我们需要找到和的数字的限制，`filter`是选择数字进行求和的条件。

让我们根据刚刚进行的重构重写 API：

```java
public static Integer sumOfFirstNPrimes(Integer count){
  return computeFirstNSum(count, (i -> isPrime(i)));
}

public static Integer sumOfFirstNEvens(Integer count){
  return computeFirstNSum(count, (i -> isEven(i)));
}

public static Integer sumOfFirstNOdds(Integer count){
  return computeFirstNSum(count, (i -> !isEven(i)));
}
```

您一定想知道以下内容：

+   `IntStream`类和相关方法的链接

+   代码库中使用`->`

+   `IntPredicate`类的使用

如果您确实想知道，那么您无需担心，因为我们将在第四章 *进入功能*和第五章 *流和管道*中涵盖这些内容。

到目前为止，我们已经看到了一些围绕数学计算的 API。这些 API 是我们的`com.packt.math.MathUtil`类的一部分。该类的完整代码可以在为本书下载的代码库中的`Chapter03/2_simple-modular-math-util/math.util/com/packt/math`中找到。

让我们将这个小型实用程序类作为名为`math.util`的模块的一部分。以下是我们用来创建模块的一些约定：

1.  将与模块相关的所有代码放在名为`math.util`的目录下，并将其视为我们的模块根目录。

1.  在根文件夹中，插入一个名为`module-info.java`的文件。

1.  将包和代码文件放在根目录下。

`module-info.java`包含什么？以下内容：

+   模块的名称

+   它导出的包，即它提供给其他模块使用的包。

+   它依赖的模块

+   它使用的服务

+   它提供实现的服务

如第一章中所述，*安装和 Java 11 的预览*，JDK 捆绑了许多模块，即现有的 Java SDK 已经模块化！其中一个模块是名为`java.base`的模块。所有用户定义的模块都隐式依赖于（或需要）`java.base`模块（可以将每个类都隐式扩展`Object`类）。

我们的`math.util`模块不依赖于任何其他模块（当然，除了`java.base`模块）。但是，它使其 API 可用于其他模块（如果不是这样，那么该模块的存在就是有问题的）。让我们继续并将此语句放入代码中：

```java
module math.util{
  exports com.packt.math;
}
```

我们告诉 Java 编译器和运行时，我们的`math.util`模块正在*导出*`com.packt.math`包中的代码给任何依赖于`math.util`的模块。

该模块的代码可以在`Chapter03/2_simple-modular-math-util/math.util`中找到。

现在，让我们创建另一个使用`math.util`模块的计算器模块。该模块有一个`Calculator`类，其工作是接受用户选择要执行的数学运算，然后输入执行操作所需的输入。用户可以从五种可用的数学运算中进行选择：

+   素数检查

+   偶数检查

+   *N*个素数的和

+   *N*个偶数的和

+   *N*个奇数的和

让我们在代码中看看这个：

```java
private static Integer acceptChoice(Scanner reader){
  System.out.println("************Advanced Calculator************");
  System.out.println("1\. Prime Number check");
  System.out.println("2\. Even Number check");
  System.out.println("3\. Sum of N Primes");
  System.out.println("4\. Sum of N Evens");
  System.out.println("5\. Sum of N Odds");
  System.out.println("6\. Exit");
  System.out.println("Enter the number to choose operation");
  return reader.nextInt();
}
```

然后，对于每个选择，我们接受所需的输入并调用相应的`MathUtil` API，如下所示：

```java
switch(choice){
  case 1:
    System.out.println("Enter the number");
    Integer number = reader.nextInt();
    if (MathUtil.isPrime(number)){
      System.out.println("The number " + number +" is prime");
    }else{
      System.out.println("The number " + number +" is not prime");
    }
  break;
  case 2:
    System.out.println("Enter the number");
    Integer number = reader.nextInt();
    if (MathUtil.isEven(number)){
      System.out.println("The number " + number +" is even");
    }
  break;
  case 3:
    System.out.println("How many primes?");
    Integer count = reader.nextInt();
    System.out.println(String.format("Sum of %d primes is %d", 
          count, MathUtil.sumOfFirstNPrimes(count)));
  break;
  case 4:
    System.out.println("How many evens?");
    Integer count = reader.nextInt();
    System.out.println(String.format("Sum of %d evens is %d", 
          count, MathUtil.sumOfFirstNEvens(count)));
  break;
  case 5: 
    System.out.println("How many odds?");
    Integer count = reader.nextInt();
    System.out.println(String.format("Sum of %d odds is %d", 
          count, MathUtil.sumOfFirstNOdds(count)));
  break;
}
```

`Calculator`类的完整代码可以在`Chapter03/2_simple-modular-math-util/calculator/com/packt/calculator/Calculator.java`中找到。

让我们以与`math.util`模块相同的方式创建我们的`calculator`模块的模块定义：

```java
module calculator{
  requires math.util;
}
```

在前面的模块定义中，我们提到`calculator`模块使用`required`关键字依赖于`math.util`模块。

该模块的代码可以在`Chapter03/2_simple-modular-math-util/calculator`中找到。

让我们编译代码：

```java
javac -d mods --module-source-path . $(find . -name "*.java")
```

上述命令必须从`Chapter03/2_simple-modular-math-util`执行。

此外，您应该在`mods`目录中跨两个模块`math.util`和`calculator`的编译代码。编译器负责处理所有内容，包括模块之间的依赖关系。我们不需要构建工具（如`ant`）来管理模块的编译。

`--module-source-path`命令是`javac`的新命令行选项，指定我们模块源代码的位置。

让我们执行上述代码：

```java
java --module-path mods -m calculator/com.packt.calculator.Calculator
```

`--module-path`命令，类似于`--classpath`，是`java`的新命令行选项，指定编译模块的位置。

在运行上述命令之后，您将看到计算器在运行中：

![](img/11c0340c-401d-4a57-b8af-614a95011b6f.png)

恭喜！有了这个，我们有一个简单的模块化应用程序正在运行。

我们已经提供了脚本来测试在 Windows 和 Linux 平台上的代码。 请在 Windows 上使用`run.bat`，在 Linux 上使用`run.sh`。

# 它是如何工作的...

现在您已经通过了示例，我们将看看如何将其概括，以便我们可以在所有模块中应用相同的模式。 我们遵循了特定的约定来创建模块：

```java
|application_root_directory
|--module1_root
|----module-info.java
|----com
|------packt
|--------sample
|----------MyClass.java
|--module2_root
|----module-info.java
|----com
|------packt
|--------test
|----------MyAnotherClass.java
```

我们将模块特定的代码放在其文件夹中，并在文件夹的根目录下放置一个相应的`module-info.java`文件。 这样，代码就组织得很好。

让我们看看`module-info.java`可以包含什么。 从 Java 语言规范（[`cr.openjdk.java.net/~mr/jigsaw/spec/lang-vm.html`](http://cr.openjdk.java.net/~mr/jigsaw/spec/lang-vm.html)）中，模块声明的形式如下：

```java
{Annotation} [open] module ModuleName { {ModuleStatement} }
```

这是语法，解释如下：

+   `{Annotation}`：这是任何形式为`@Annotation(2)`的注释。

+   `open`：这个关键字是可选的。 开放模块通过反射在运行时使其所有组件可访问。 但是，在编译时和运行时，只有明确导出的组件才可访问。

+   `module`：这是用于声明模块的关键字。

+   `ModuleName`：这是模块的名称，是一个有效的 Java 标识符，标识符名称之间可以使用允许的点（`.`）-类似于`math.util`。

+   `{ModuleStatement}`：这是模块定义中允许的语句的集合。 让我们接下来扩展这个。

模块语句的形式如下：

```java
ModuleStatement:
  requires {RequiresModifier} ModuleName ;
  exports PackageName [to ModuleName {, ModuleName}] ;
  opens PackageName [to ModuleName {, ModuleName}] ;
  uses TypeName ;
  provides TypeName with TypeName {, TypeName} ;
```

模块语句在这里被解码：

+   `requires`：这用于声明对模块的依赖关系。 `{RequiresModifier}`可以是**transitive**，**static**，或两者兼有。 Transitive 表示任何依赖于给定模块的模块也隐式地依赖于给定模块传递地。 Static 表示模块依赖在编译时是强制性的，但在运行时是可选的。 一些示例是`requires math.util`，`requires transitive math.util`，和`requires static math.util`。

+   `exports`：这用于使给定的包对依赖模块可访问。 可选地，我们可以通过指定模块名称来强制包的可访问性到特定模块，例如`exports com.package.math to claculator`。

+   `opens`：这用于打开特定包。 我们之前看到可以通过在模块声明中使用`open`关键字来打开模块。 但这可能不够严格。 因此，我们可以使用`opens`关键字在运行时打开特定包以进行反射访问-`opens com.packt.math`。

+   `uses`：这用于声明对通过`java.util.ServiceLoader`可访问的服务接口的依赖关系。 服务接口可以在当前模块中或在当前模块依赖的任何模块中。

+   `provides`：这用于声明服务接口并提供至少一个实现。 服务接口可以在当前模块中声明，也可以在任何其他依赖模块中声明。 但是，服务实现必须在同一模块中提供； 否则，将发生编译时错误。

我们将更详细地查看`uses`和`provides`子句，*使用服务来创建消费者和提供者模块之间的松耦合*的示例中。

所有模块的模块源可以使用`--module-source-path`命令行选项一次性编译。 这样，所有模块将被编译并放置在由`-d`选项提供的目录下的相应目录中。 例如，`javac -d mods --module-source-path . $(find . -name "*.java")`将当前目录中的代码编译成一个`mods`目录。

运行代码同样简单。我们使用命令行选项`--module-path`指定我们所有模块编译后的路径。然后，我们使用命令行选项`-m`指定模块名称以及完全限定的主类名称，例如，`java --module-path mods -m calculator/com.packt.calculator.Calculator`。

# 另请参阅

在第一章的*编译和运行 Java 应用程序*中，*安装和预览 Java 11*

# 创建一个模块化 JAR

将模块编译成类是不错的，但不适合共享二进制文件和部署。JAR 是更好的共享和部署格式。我们可以将编译后的模块打包成 JAR，并且包含`module-info.class`的 JAR 文件被称为**模块化 JAR**。在这个示例中，我们将看看如何创建模块化 JAR，并且还将看看如何执行由多个模块化 JAR 组成的应用程序。

# 准备工作

我们已经在*创建一个更简单的模块化应用程序*中看到并创建了一个简单的模块化应用程序。为了构建一个模块化 JAR，我们将使用`Chapter03/3_modular_jar`中提供的示例代码。这个示例代码包含两个模块：`math.util`和`calculator`。我们将为这两个模块创建模块化 JAR 文件。

# 如何做...

1.  编译代码并将编译后的类放入一个目录，比如`mods`：

```java
 javac -d mods --module-source-path . $(find . -name *.java)
```

1.  为`math.util`模块构建一个模块化 JAR：

```java
      jar --create --file=mlib/math.util@1.0.jar --module-version 1.0
      -C mods/math.util .
```

不要忘记在上述代码的末尾加上点（`.`）。

1.  为`calculator`模块构建一个模块化 JAR，指定主类以使 JAR 可执行：

```java
 jar --create --file=mlib/calculator@1.0.jar --module-version 1.0 
      --main-class com.packt.calculator.Calculator -C mods/calculator .
```

上述命令中的关键部分是`--main-class`选项。这使我们能够在执行时不提供主类信息来执行 JAR 文件。

1.  现在，我们在`mlib`目录中有两个 JAR 文件：`math.util@1.0.jar`和`calculator@1.0.jar`。这些 JAR 文件被称为模块化 JAR 文件。如果您想要运行示例，可以使用以下命令：

```java
 java -p mlib -m calculator
```

1.  在 Java 9 中引入了 JAR 命令的一个新的命令行选项，称为`-d`或`--describe-module`。这会打印模块化 JAR 包含的模块信息：

```java
jar -d --file=mlib/calculator@1.0.jar
```

`calculator@1.0.jar`的`jar -d`输出如下：

```java
calculator@1.0
  requires mandated java.base
  requires math.util
  conceals com.packt.calculator
  main-class com.packt.calculator.Calculator

jar -d --file=mlib/math.util@1.0.jar
```

`math.util@1.0.jar`的`jar -d`输出如下：

```java
math.util@1.0
  requires mandated java.base
  exports com.packt.math
```

我们已经提供了以下脚本来在 Windows 上尝试示例代码：

+   `compile-math.bat`

+   `compile-calculator.bat`

+   `jar-math.bat`

+   `jar-calculator.bat`

+   `run.bat`

我们已经提供了以下脚本来在 Linux 上尝试示例代码：

+   `compile.sh`

+   `jar-math.sh`

+   `jar-calculator.sh`

+   `run.sh`

您必须按照它们列出的顺序运行脚本。

# 在 Project Jigsaw JDK 应用程序之前使用模块 JAR

如果我们的模块化 JAR 可以在 Project Jigsaw JDK 应用程序之前运行，那将是很棒的。这样，我们就不需要为 JDK 9 之前的应用程序编写另一个版本的 API。好消息是，我们可以像使用普通 JAR 一样使用我们的模块化 JAR，也就是说，没有`module-info.class`的 JAR。我们将在这个示例中看到如何做到这一点。

# 准备工作

对于这个示例，我们将需要一个模块化 JAR 和一个非模块化应用程序。我们的模块化代码可以在`Chapter03/4_modular_jar_with_pre_java9/math.util`中找到（这是我们在*创建一个简单的模块化应用程序*中创建的相同的`math.util`模块）。让我们使用以下命令编译这个模块化代码并创建一个模块化 JAR：

```java
javac -d classes --module-source-path . $(find math.util -name *.java)
mkdir mlib
jar --create --file mlib/math.util.jar -C classes/math.util .
```

我们还在`Chapter03/4_modular_jar_with_pre_java9`中提供了一个`jar-math.bat`脚本，可以用于在 Windows 上创建模块化 JAR。我们有我们的模块化 JAR。让我们使用`jar`命令的`-d`选项来验证它：

```java
jar -d --file mlib/math.util@1.0.jar
math.util@1.0
  requires mandated java.base
  exports com.packt.math
```

# 如何做...

现在，让我们创建一个非模块化的简单应用程序。我们的应用程序将包含一个名为`NonModularCalculator`的类，它从*创建一个简单的模块化应用程序*中的`Calculator`类中借用其代码。

您可以在`Chapter03/4_modular_jar_with_pre_java9/calculator`目录下的`com.packt.calculator`包中找到`NonModularCalculator`类的定义。由于它是非模块化的，所以不需要`module-info.java`文件。该应用程序利用我们的模块化 JAR`math.util.jar`来执行一些数学计算。

此时，您应该拥有以下内容：

+   一个名为`math.util@1.0.jar`的模块化 JAR

+   一个由`NonModularCalculator`包组成的非模块化应用程序

现在，我们需要编译我们的`NonModularCalculator`类：

```java
javac -d classes/ --source-path calculator $(find calculator -name *.java)
```

运行上一个命令后，您将看到一系列错误，指出`com.packt.math`包不存在，找不到`MathUtil`符号等等。您已经猜到了；我们没有为编译器提供我们的模块化 JAR 的位置。让我们使用`--class-path`选项添加模块化`jar`的位置：

```java
javac --class-path mlib/* -d classes/ --source-path calculator $(find calculator -name *.java)
```

现在，我们已成功编译了依赖于模块化 JAR 的非模块化代码。让我们运行编译后的代码：

```java
java -cp classes:mlib/* com.packt.calculator.NonModularCalculator
```

恭喜！您已成功地将您的模块化 JAR 用于非模块化应用程序。很棒，对吧？

我们在`Chapter03/4_modular_jar_with_pre_java9`提供了以下脚本来在 Windows 平台上运行代码：

+   `compile-calculator.bat`

+   `run.bat`

# 自下而上的迁移

现在 Java 9 已经发布，备受期待的模块化功能现在可以被开发人员采用。在某个时候，您将参与将您的应用程序迁移到 Java 9，并因此尝试将其模块化。这种涉及第三方库和重新思考代码结构的重大变化需要适当的规划和实施。Java 团队提出了两种迁移方法：

+   自下而上的迁移

+   自上而下的迁移

在学习自下而上的迁移之前，了解无名模块和自动模块是很重要的。假设您正在访问一个在任何模块中都不可用的类型；在这种情况下，模块系统将在类路径上搜索该类型，如果找到，该类型将成为无名模块的一部分。这类似于我们编写的不属于任何包的类，但 Java 会将它们添加到无名包中，以简化新类的创建。

因此，无名模块是一个没有名称的通用模块，其中包含所有那些不属于任何模块但在类路径中找到的类型。无名模块可以访问所有命名模块（用户定义的模块）和内置模块（Java 平台模块）的所有导出类型。另一方面，命名模块（用户定义的模块）将无法访问无名模块中的类型。换句话说，命名模块无法声明对无名模块的依赖关系。如果您想声明依赖关系，该怎么办？无名模块没有名称！

有了无名模块的概念，您可以将您的 Java 8 应用程序保持原样，并在 Java 9 上运行它（除了任何已弃用的内部 API，这些 API 在 Java 9 中可能不可用于用户代码）。

如果您尝试过*使用 jdeps 在 Java 应用程序中查找依赖项*的示例，您可能已经看到了这一点，在那个示例中，我们有一个非模块化的应用程序，并且能够在 Java 9 上运行它。然而，在 Java 9 上按原样运行将违背引入模块化系统的初衷。

如果一个包在命名模块和无名模块中都有定义，那么命名模块中的包将优先于无名模块中的包。这有助于防止当它们来自命名模块和无名模块时的包冲突。

自动模块是 JVM 自动创建的模块。当我们将打包在 JAR 中的类引入模块路径而不是类路径时，将创建这些模块。该模块的名称将从 JAR 的名称中派生，因此与未命名模块不同。或者，可以通过在 JAR 清单文件中对`Automatic-Module-Name`提供模块名称来为这些自动模块提供名称。这些自动模块导出其中的所有包，并且还依赖于所有自动和命名（用户/JDK）模块。

根据这个解释，模块可以分为以下几类：

+   **未命名模块**：在类路径上可用但在模块路径上不可用的代码放置在未命名模块中。

+   **命名模块**：所有具有与之关联的名称的模块 - 这些可以是用户定义的模块和 JDK 模块。

+   **自动模块**：所有由 JVM 根据模块路径中存在的 JAR 文件隐式创建的模块。

+   **隐式模块**：隐式创建的模块。它们与自动模块相同。

+   **显式模块**：所有由用户或 JDK 显式创建的模块。

但未命名模块和自动模块是开始迁移的良好第一步。所以，让我们开始吧！

# 准备工作

我们需要一个非模块化的应用程序，最终我们将对其进行模块化。我们已经创建了一个简单的应用程序，其源代码位于`Chapter03/6_bottom_up_migration_before`。这个简单的应用程序有三个部分：

+   一个包含我们最喜爱的数学 API 的数学实用程序库：素数检查器，偶数检查器，素数之和，偶数之和和奇数之和。其代码位于`Chapter03/6_bottom_up_migration_before/math_util`。

+   一个银行实用程序库，其中包含用于计算简单利息和复利的 API。其代码位于`Chapter03/6_bottom_up_migration_before/banking_util`。

+   我们的计算器应用程序帮助我们进行数学和银行业务计算。为了使其更有趣，我们将以 JSON 格式输出结果，为此我们将使用 Jackson JSON API。其代码位于`Chapter03/6_bottom_up_migration_before/calculator`。

在您复制或下载了代码之后，我们将编译和构建相应的 JAR。因此，请使用以下命令来编译和构建 JAR：

```java
#Compiling math util

javac -d math_util/out/classes/ -sourcepath math_util/src $(find math_util/src -name *.java)
jar --create --file=math_util/out/math.util.jar 
-C math_util/out/classes/ .

#Compiling banking util

javac -d banking_util/out/classes/ -sourcepath banking_util/src $(find banking_util/src -name *.java)
jar --create --file=banking_util/out/banking.util.jar 
-C banking_util/out/classes/ .

#Compiling calculator

javac -cp calculator/lib/*:math_util/out/math.util.jar:banking_util/out/banking.util.jar -d calculator/out/classes/ -sourcepath calculator/src $(find calculator/src -name *.java)
```

让我们也为此创建一个 JAR（我们将使用该 JAR 来构建依赖关系图，但不用于运行应用程序）：

```java
jar --create --file=calculator/out/calculator.jar -C calculator/out/classes/ .
```

请注意，我们的 Jackson JARs 位于 calculator/lib 中，所以您不需要担心下载它们。让我们使用以下命令运行我们的计算器：

```java
java -cp calculator/out/classes:calculator/lib/*:math_util/out/math.util.jar:banking_util/out/banking.util.jar com.packt.calculator.Calculator
```

您将看到一个菜单询问操作的选择，然后您可以尝试不同的操作。现在，让我们对这个应用程序进行模块化！

我们提供了`package-*.bat`和 run.bat 来在 Windows 上打包和运行应用程序。您可以使用`package-*.sh`和`run.sh`在 Linux 上打包和运行应用程序。

# 如何做...

模块化应用程序的第一步是了解其依赖关系图。让我们为我们的应用程序创建一个依赖关系图。为此，我们使用`jdeps`工具。如果您想知道`jdeps`工具是什么，请立即阅读*在 Java 应用程序中使用 jdeps 查找依赖关系*。好的，让我们运行`jdeps`工具：

```java
jdeps -summary -R -cp calculator/lib/*:math_util/out/*:banking_util/out/* calculator/out/calculator.jar
```

我们要求`jdeps`给我们`calculator.jar`的依赖关系摘要，然后对`calculator.jar`的每个依赖项进行递归处理。我们得到的输出如下：

```java
banking.util.jar -> java.base
calculator.jar -> banking_util/out/banking.util.jar
calculator.jar -> calculator/lib/jackson-databind-2.8.4.jar
calculator.jar -> java.base
calculator.jar -> math_util/out/math.util.jar
jackson-annotations-2.8.4.jar -> java.base
jackson-core-2.8.4.jar -> java.base
jackson-databind-2.8.4.jar -> calculator/lib/jackson-annotations-2.8.4.jar
jackson-databind-2.8.4.jar -> calculator/lib/jackson-core-2.8.4.jar
jackson-databind-2.8.4.jar -> java.base
jackson-databind-2.8.4.jar -> java.logging
jackson-databind-2.8.4.jar -> java.sql
jackson-databind-2.8.4.jar -> java.xml
math.util.jar -> java.base
```

前面的输出很难理解，可以用图表形式表示如下：

![](img/93b83846-6af9-420c-b832-333dbaa80c30.png)

在自下而上的迁移中，我们首先将叶节点模块化。在我们的图中，`java.xml`，`java.sql`，`java.base`和`java.logging`叶节点已经被模块化。让我们将`banking.util.jar`模块化。

本节的所有代码都可以在`Chapter03/6_bottom_up_migration_after`中找到。

# 模块化 banking.util.jar

1.  将`BankUtil.java`从`Chapter03/6_bottom_up_migration_before/banking_util/src/com/packt/banking`复制到`Chapter03/6_bottom_up_migration_after/src/banking.util/com/packt/banking`。有两件事需要注意：

+   我们已经将文件夹从`banking_util`改名为`banking.util`。这是为了遵循将与模块相关的代码放在带有模块名称的文件夹下的惯例。

+   我们将包直接放在`banking.util`文件夹下，而不是放在`src`下。同样，这是为了遵循惯例。我们将把所有的模块放在`src`文件夹下。

1.  在`Chapter03/6_bottom_up_migration_after/src/banking.util`下创建模块定义文件`module-info.java`，内容如下：

```java
        module banking.util{   
          exports com.packt.banking;
        }
```

1.  从`6_bottom_up_migration_after`文件夹中，通过运行以下命令编译模块的 Java 代码：

```java
 javac -d mods --module-source-path src 
      $(find src -name *.java)
```

1.  你会看到`banking.util`模块中的 Java 代码被编译到了 mods 目录中。

1.  让我们为这个模块创建一个模块化的 JAR：

```java
 jar --create --file=mlib/banking.util.jar -C mods/banking.util .
```

如果你想知道什么是模块化的 JAR，请随意阅读本章节中的*创建模块化 JAR*部分。

现在我们已经将`banking.util.jar`模块化了，让我们在*准备工作*部分之前使用这个模块化的`jar`来替代之前使用的非模块化 JAR。你应该从`6_bottom_up_migration_before`文件夹中执行以下操作，因为我们还没有完全将应用程序模块化：

```java
java --add-modules ALL-MODULE-PATH --module-path ../6_bottom_up_migration_after/mods/banking.util -cp calculator/out/classes:calculator/lib/*:math_util/out/math.util.jar com.packt.calculator.Calculator
```

`--add-modules`选项告诉 Java 运行时要包含模块，可以通过模块名称或预定义常量，即`ALL-MODULE-PATH`，`ALL-DEFAULT`和`ALL-SYSTEM`来实现。我们使用了`ALL-MODULE-PATH`来添加模块，该模块可在我们的模块路径上使用。

`--module-path`选项告诉 Java 运行时我们模块的位置。

你会看到我们的计算器正常运行。尝试简单利息计算，复利计算，以检查`BankUtil`类是否被找到。因此，我们的依赖图现在如下所示：

![](img/d708231d-d5e9-4fea-b1ab-44d956a531c8.png)

# 模块化 math.util.jar

1.  将`MathUtil.java`从`Chapter03/6_bottom_up_migration_before/math_util/src/com/packt/math`复制到`Chapter03/6_bottom_up_migration_after/src/math.util/com/packt/math`。

1.  在`Chapter03/6_bottom_up_migration_after/src/math.util`下创建模块定义文件`module-info.java`，内容如下：

```java
        module math.util{
          exports com.packt.math;
        }
```

1.  从`6_bottom_up_migration_after`文件夹中，通过运行以下命令编译模块的 Java 代码：

```java
 javac -d mods --module-source-path src $(find src -name *.java)
```

1.  你会看到`math.util`和`banking.util`模块中的 Java 代码被编译到了`mods`目录中。

1.  让我们为这个模块创建一个模块化的 JAR：

```java
 jar --create --file=mlib/math.util.jar -C mods/math.util .
```

如果你想知道什么是模块化的`jar`，请随意阅读本章节中的*创建模块化 JAR*部分。

1.  现在我们已经将`math.util.jar`模块化了，让我们在*准备工作*部分之前使用这个模块化的`jar`来替代非模块化的`jar`。你应该从`6_bottom_up_migration_before`文件夹中执行以下操作，因为我们还没有完全将应用程序模块化：

```java
 java --add-modules ALL-MODULE-PATH --module-path
      ../6_bottom_up_migration_after/mods/banking.util:
      ../6_bottom_up_migration_after/mods/math.util 
      -cp calculator/out/classes:calculator/lib/*
      com.packt.calculator.Calculator
```

我们的应用程序运行正常，并且依赖图如下所示：

![](img/a82c6458-5c5b-4270-a8a5-308a39b9eb12.png)

我们无法将`calculator.jar`模块化，因为它依赖于另一个非模块化的代码`jackson-databind`，而我们无法将`jackson-databind`模块化，因为它不是我们维护的。这意味着我们无法为我们的应用程序实现 100%的模块化。我们在本教程开始时向您介绍了未命名模块。我们类路径中的所有非模块化代码都被分组在未命名模块中，这意味着所有与 jackson 相关的代码仍然可以保留在未命名模块中，我们可以尝试将`calculator.jar`模块化。但是我们无法这样做，因为`calculator.jar`不能声明对`jackson-databind-2.8.4.jar`的依赖（因为它是一个未命名模块，命名模块不能声明对未命名模块的依赖）。

解决这个问题的一种方法是将与 jackson 相关的代码作为自动模块。我们可以通过移动与 jackson 相关的 jar 来实现这一点：

+   `jackson-databind-2.8.4.jar`

+   `jackson-annotations-2.8.4.jar`

+   `jackson-core-2.8.4.jar`

我们将使用以下命令将它们移动到`6_bottom_up_migration_after`文件夹下：

```java
$ pwd 
/root/java9-samples/Chapter03/6_bottom_up_migration_after
$ cp ../6_bottom_up_migration_before/calculator/lib/*.jar mlib/
$ mv mlib/jackson-annotations-2.8.4.jar mods/jackson.annotations.jar
$ mv mlib/jackson-core-2.8.4.jar mods/jackson.core.jar
$ mv mlib/jackson-databind-2.8.4.jar mods/jackson.databind.jar
```

重命名 JAR 的原因是模块的名称必须是有效的标识符（不能仅为数字，不能包含`-`和其他规则），用`.`分隔。由于名称是从 JAR 文件的名称派生的，我们必须将 JAR 文件重命名以符合 Java 标识符规则。

如果不存在，创建一个新的`mlib`目录，在`6_bottom_up_migration_after`下。

现在，让我们再次运行我们的计算器程序，使用以下命令：

```java
java --add-modules ALL-MODULE-PATH --module-path ../6_bottom_up_migration_after/mods:../6_bottom_up_migration_after/mlib -cp calculator/out/classes com.packt.calculator.Calculator
```

应用程序将像往常一样运行。您会注意到我们的`-cp`选项值正在变小，因为所有依赖库都已经作为模块移动到了模块路径中。依赖关系图现在看起来像这样：

![](img/235a1fd5-2c09-4046-8c84-fb0abafd266b.png)

# 模块化 calculator.jar

迁移的最后一步是将`calculator.jar`模块化。按照以下步骤进行模块化：

1.  将`Chapter03/6_bottom_up_migration_before/calculator/src`中的`com`文件夹复制到`Chapter03/6_bottom_up_migration_after/src/calculator`。

1.  在`Chapter03/6_bottom_up_migration_after/src/calculator`下创建模块定义文件`module-info.java`，定义如下：

```java
        module calculator{ 
          requires math.util; 
          requires banking.util; 
          requires jackson.databind; 
          requires jackson.core; 
          requires jackson.annotations; 
        }
```

1.  从`6_bottom_up_migration_after`文件夹中，通过运行以下命令编译模块的 Java 代码：

```java
 javac -d mods --module-path mlib:mods --module-source-path src $(find src -name *.java)
```

1.  您将看到我们所有模块中的 Java 代码都编译到了 mods 目录中。请注意，您应该已经将自动模块（即与 jackson 相关的 JAR）放置在`mlib`目录中。

1.  让我们为这个模块创建一个模块化的 JAR，并指定哪个是`main`类：

```java
 jar --create --file=mlib/calculator.jar --main-
      class=com.packt.calculator.Calculator -C mods/calculator .
```

1.  现在，我们有了我们的计算器模块的模块化 JAR，这是我们的主要模块，因为它包含了`main`类。通过这样做，我们还模块化了我们的完整应用程序。让我们从文件夹`6_bottom_up_migration_after`运行以下命令：

```java
 java -p mlib:mods -m calculator
```

因此，我们已经看到了如何使用自下而上的迁移方法将非模块化的应用程序模块化。最终的依赖关系图看起来像这样：

![](img/68490d34-cdaf-44a0-b7bd-02e6754c10bf.png)

这个模块化应用程序的最终代码可以在`Chapter03/6_bottom_up_migration_after`中找到。

我们本可以在同一目录`6_bottom_up_migration_before`中进行修改，即在同一目录中对代码进行模块化。但我们更喜欢在不同的目录`6_bottom_up_migration_after`中单独进行，以保持代码整洁，不干扰现有的代码库。

# 它是如何工作的...

未命名模块的概念帮助我们在 Java 9 上运行我们的非模块化应用程序。在迁移过程中，模块路径和类路径的使用帮助我们运行部分模块化的应用程序。我们从模块化那些不依赖于任何非模块化代码的代码库开始，而我们无法模块化的任何代码库，我们将其转换为自动模块，从而使我们能够模块化依赖于这样一个代码库的代码。最终，我们得到了一个完全模块化的应用程序。

# 自上而下的迁移

迁移的另一种技术是自上而下的迁移。在这种方法中，我们从 JAR 的依赖图中的根 JAR 开始。

JAR 表示一个代码库。我们假设代码库以 JAR 的形式可用，因此我们得到的依赖图具有节点，这些节点是 JAR。

将依赖图的根模块化意味着该根模块依赖的所有其他 JAR 都必须是模块化的。否则，这个模块化根模块就无法声明对未命名模块的依赖。让我们考虑一下我们在底向上迁移食谱中介绍的非模块化应用程序的例子。依赖图看起来像这样：

![](img/90299e09-6b9c-4e5e-a65f-e6e31db6158d.png)

我们在自上而下的迁移中广泛使用自动模块。自动模块是由 JVM 隐式创建的模块。这些模块是基于模块路径中可用的非模块化 JAR 创建的。

# 准备工作

我们将使用我们在前一篇食谱*自下而上的迁移*中介绍的计算器示例。继续从`Chapter03/7_top_down_migration_before`复制非模块化代码。如果您希望运行它并查看它是否正常工作，请使用以下命令：

```java
$ javac -d math_util/out/classes/ -sourcepath math_util/src $(find math_util/src -name *.java)

$ jar --create --file=math_util/out/math.util.jar 
-C math_util/out/classes/ .

$ javac -d banking_util/out/classes/ -sourcepath banking_util/src $(find banking_util/src -name *.java)

$ jar --create --file=banking_util/out/banking.util.jar 
-C banking_util/out/classes/ .

$ javac -cp calculator/lib/*:math_util/out/math.util.jar:banking_util/out/banking.util.jar -d calculator/out/classes/ -sourcepath calculator/src $(find calculator/src -name *.java)

$ java -cp calculator/out/classes:calculator/lib/*:math_util/out/math.util.jar:banking_util/out/banking.util.jar com.packt.calculator.Calculator
```

我们提供了`package-*.bat`和`run.bat`来在 Windows 上打包和运行代码，并在 Linux 上使用`package-*.sh`和`run.sh`来打包和运行代码。

# 如何做...

我们将对`Chapter03/7_top_down_migration_after`目录下的应用程序进行模块化。在`Chapter03/7_top_down_migration_after`下创建两个目录，`src`和`mlib`。

# 模块化计算器

1.  在我们模块化所有依赖项之前，我们无法模块化计算器。但是，在某些情况下，模块化其依赖项可能更容易，而在其他情况下可能不那么容易，特别是在依赖项来自第三方的情况下。在这种情况下，我们使用自动模块。我们将非模块化的 JAR 复制到`mlib`文件夹中，并确保 JAR 的名称采用`<identifier>(.<identifier>)*`的形式，其中`<identifier>`是有效的 Java 标识符。

```java
 $ cp ../7_top_down_migration_before/calculator/lib/jackson-
      annotations-
 2.8.4.jar mlib/jackson.annotations.jar 

 $ cp ../7_top_down_migration_before/calculator/lib/jackson-core-
      2.8.4.jar
 mlib/jackson.core.jar 

 $ cp ../7_top_down_migration_before/calculator/lib/jackson-
      databind-
 2.8.4.jar mlib/jackson.databind.jar 

 $ cp 
      ../7_top_down_migration_before/banking_util/out/banking.util.jar 
      mlib/ 

 $ cp ../7_top_down_migration_before/math_util/out/math.util.jar 
      mlib/
```

我们已经提供了`copy-non-mod-jar.bat`和`copy-non-mod-jar.sh`脚本，以便轻松复制 jar 包。

让我们看看我们复制到`mlib`中的内容：

```java
     $ ls mlib
      banking.util.jar  jackson.annotations.jar  jackson.core.jar 
 jackson.databind.jar  math.util.jar
```

`banking.util.jar`和`math.util.jar`只有在您已经在`Chapter03/7_top_down_migration_before/banking_util`和`Chapter03/7_top_down_migration_before/math_util`目录中编译和打包了代码时才会存在。我们在*准备工作*部分中已经做过这个。我们在*准备工作*部分中已经做过这个。

1.  在`src`下创建一个新的`calculator`文件夹。这将包含`calculator`模块的代码。

1.  在`Chapter03/7_top_down_migration_after/src/calculator`目录下创建`module-info.java`，其中包含以下内容**：**

```java
        module calculator{ 
          requires math.util; 
          requires banking.util; 
          requires jackson.databind; 
          requires jackson.core; 
          requires jackson.annotations; 
        }
```

1.  将`Chapter03/7_top_down_migration_before/calculator/src/com`目录及其下的所有代码复制到`Chapter03/7_top_down_migration_after/src/calculator`。

1.  编译 calculator 模块：

```java
 #On Linux
 javac -d mods --module-path mlib --module-source-path src $(find
      src -name *.java)

 #On Windows
 javac -d mods --module-path mlib --module-source-path src 
      srccalculatormodule-info.java 
      srccalculatorcompacktcalculatorCalculator.java 
      srccalculatorcompacktcalculatorcommands*.java
```

1.  为`calculator`模块创建模块化 JAR：

```java
 jar --create --file=mlib/calculator.jar --main-
      class=com.packt.calculator.Calculator -C mods/calculator/ .
```

1.  运行`calculator`模块：

```java
 java --module-path mlib -m calculator
```

我们将看到我们的计算器是否正确执行。您可以尝试不同的操作来验证它们是否都正确执行。

# 模块化 banking.util

由于这不依赖于其他非模块化代码，我们可以通过以下步骤直接将其转换为模块：

1.  在`src`下创建一个新的`banking.util`文件夹。这将包含`banking.util`模块的代码。

1.  在`Chapter03/7_top_down_migration_after/src/banking.util`目录下创建`module-info.java`，其中包含以下内容：

```java
        module banking.util{
          exports com.packt.banking; 
        }
```

1.  将`Chapter03/7_top_down_migration_before/banking_util/src/com`目录及其下所有代码复制到`Chapter03/7_top_down_migration_after/src/banking.util`。

1.  编译模块：

```java
 #On Linux
 javac -d mods --module-path mlib --module-source-path src $(find 
      src -name *.java)

 #On Windows
 javac -d mods --module-path mlib --module-source-path src 
      srcbanking.utilmodule-info.java 
      srcbanking.utilcompacktbankingBankUtil.java
```

1.  为`banking.util`模块创建一个模块化的 JAR。这将替换`mlib`中已经存在的非模块化`banking.util.jar`：

```java
 jar --create --file=mlib/banking.util.jar -C mods/banking.util/ .
```

1.  运行`calculator`模块，测试`banking.util`模块化 JAR 是否已成功创建：

```java
 java --module-path mlib -m calculator
```

1.  您应该看到计算器被执行。尝试不同的操作，以确保没有“找不到类”的问题。

# 模块化 math.util

1.  在`src`下创建一个新的`math.util`文件夹。这将包含`math.util`模块的代码。

1.  在`Chapter03/7_top_down_migration_after/src/math.util`目录下创建`module-info.java`，其中包含以下内容：

```java
        module math.util{ 
          exports com.packt.math; 
        }
```

1.  将`Chapter03/7_top_down_migration_before/math_util/src/com`目录及其下所有代码复制到`Chapter03/7_top_down_migration_after/src/math.util`。

1.  编译模块：

```java
 #On Linux
 javac -d mods --module-path mlib --module-source-path src $(find 
      src -name *.java)

 #On Windows
 javac -d mods --module-path mlib --module-source-path src 
      srcmath.utilmodule-info.java 
      srcmath.utilcompacktmathMathUtil.java
```

1.  为`banking.util`模块创建一个模块化的 JAR。这将替换`mlib`中已经存在的非模块化`banking.util.jar`：

```java
 jar --create --file=mlib/math.util.jar -C mods/math.util/ .
```

1.  运行`calculator`模块，测试`math.util`模块化 JAR 是否已成功创建。

```java
 java --module-path mlib -m calculator
```

1.  您应该看到计算器被执行。尝试不同的操作，以确保没有*找不到类*的问题。

有了这个，我们已经完全模块化了应用程序，除了 Jackson 库，我们已经将其转换为自动模块。

我们更喜欢自上而下的迁移方法。这是因为我们不必同时处理类路径和模块路径。我们可以将所有内容都转换为自动模块，然后在将非模块化的 JAR 迁移到模块化 JAR 时使用模块路径。

# 使用服务来创建消费者和提供者模块之间的松耦合

通常，在我们的应用程序中，我们有一些接口和这些接口的多个实现。然后，在运行时，根据某些条件，我们使用特定的实现。这个原则叫做**依赖反转**。依赖注入框架（如 Spring）使用这个原则来创建具体实现的对象，并将其分配（或注入）到抽象接口类型的引用中。

很长一段时间以来，Java（自 Java 6 以来）一直支持通过`java.util.ServiceLoader`类进行服务提供者加载。使用 Service Loader，您可以有一个**服务提供者接口**（SPI）和 SPI 的多个实现，简称服务提供者。这些服务提供者位于类路径中，并在运行时加载。当这些服务提供者位于模块中时，由于我们不再依赖于类路径扫描来加载服务提供者，我们需要一种机制来告诉我们的模块有关服务提供者和服务提供者接口的机制，以及它提供实现的服务提供者。在这个配方中，我们将通过一个简单的例子来看一下这种机制。

# 准备工作

对于这个配方，我们没有特定的设置。在这个配方中，我们将举一个简单的例子。我们有一个`BookService`抽象类，支持 CRUD 操作。现在，这些 CRUD 操作可以在 SQL DB、MongoDB、文件系统等上工作。通过使用服务提供者接口和`ServiceLoader`类来加载所需的服务提供者实现，可以提供这种灵活性。

# 如何做...

我们在这个配方中有四个模块：

+   `book.service`：这是包含我们服务提供者接口的模块，也就是服务

+   `mongodb.book.service`：这是其中一个服务提供者模块

+   `sqldb.book.service`：这是另一个服务提供者模块

+   `book.manage`：这是服务消费者模块

以下步骤演示了如何利用`ServiceLoader`实现松耦合：

1.  在`Chapter03/8_services/src`目录下创建一个名为`book.service`的文件夹。我们的`book.service`模块的所有代码将放在这个文件夹中。

1.  创建一个新的包`com.packt.model`，并在新包下创建一个名为`Book`的新类。这是我们的模型类，包含以下属性：

```java
        public String id; 
        public String title; 
        public String author;
```

1.  创建一个新的包`com.packt.service`，并在新包下创建一个名为`BookService`的新类。这是我们的主要服务接口，服务提供者将为此服务提供实现。除了 CRUD 操作的抽象方法之外，值得一提的是`getInstance()`方法。该方法使用`ServiceLoader`类加载任何一个服务提供者（具体来说是最后一个），然后使用该服务提供者获取`BookService`的实现。让我们看一下以下代码：

```java
        public static BookService getInstance(){ 
          ServiceLoader<BookServiceProvider> sl = 
                 ServiceLoader.load(BookServiceProvider.class);
          Iterator<BookServiceProvider> iter = sl.iterator(); 
          if (!iter.hasNext()) 
            throw new RuntimeException("No service providers found!");

          BookServiceProvider provider = null; 
          while(iter.hasNext()){ 
            provider = iter.next(); 
            System.out.println(provider.getClass()); 
          } 
          return provider.getBookService(); 
        }
```

第一个`while`循环只是为了演示`ServiceLoader`加载所有服务提供者，然后我们选择其中一个服务提供者。您也可以有条件地返回服务提供者，但这完全取决于要求。

1.  另一个重要部分是实际的服务提供者接口。其责任是返回服务实现的适当实例。在我们的示例中，`com.packt.spi`包中的`BookServiceProvider`是一个服务提供者接口：

```java
        public interface BookServiceProvider{ 
          public BookService getBookService(); 
        }
```

1.  我们在`Chapter03/8_services/src/book.service`目录下创建`module-info.java`，其中包含以下内容：

```java
        module book.service{ 
          exports com.packt.model; 
          exports com.packt.service; 
          exports com.packt.spi; 
          uses com.packt.spi.BookServiceProvider; 
        }
```

在前面的模块定义中，`uses`语句指定了模块使用`ServiceLoader`发现的服务接口。

1.  现在让我们创建一个名为`mongodb.book.service`的服务提供者模块。这将在`book.service`模块中为`BookService`和`BookServiceProvider`接口提供实现。我们的想法是，这个服务提供者将使用 MongoDB 数据存储实现 CRUD 操作。

1.  在`Chapter03/8_services/src`目录下创建一个`mongodb.book.service`文件夹。

1.  在`com.packt.mongodb.service`包中创建一个`MongoDbBookService`类，它继承了`BookService`抽象类，并提供了我们的抽象 CRUD 操作方法的实现：

```java
        public void create(Book book){ 
          System.out.println("Mongodb Create book ... " + book.title); 
        } 

        public Book read(String id){ 
          System.out.println("Mongodb Reading book ... " + id); 
          return new Book(id, "Title", "Author"); 
        } 

        public void update(Book book){ 
          System.out.println("Mongodb Updating book ... " + 
              book.title); 
        }

        public void delete(String id){ 
          System.out.println("Mongodb Deleting ... " + id); 
        }
```

1.  在`com.packt.mongodb`包中创建一个`MongoDbBookServiceProvider`类，它实现了`BookServiceProvider`接口。这是我们的服务发现类。基本上，它返回`BookService`实现的相关实例。它重写了`BookServiceProvider`接口中的方法，如下所示：

```java
        @Override 
        public BookService getBookService(){ 
          return new MongoDbBookService(); 
        }
```

1.  模块定义非常有趣。我们必须在模块定义中声明该模块是`BookServiceProvider`接口的服务提供者，可以这样做：

```java
        module mongodb.book.service{ 
          requires book.service; 
          provides com.packt.spi.BookServiceProvider 
                   with com.packt.mongodb.MongoDbBookServiceProvider; 
        }
```

`provides .. with ..`语句用于指定服务接口和其中一个服务提供者。

1.  现在让我们创建一个名为`book.manage`的服务使用者模块。

1.  在`Chapter03/8_services/src`下创建一个新的`book.manage`文件夹，其中将包含模块的代码。

1.  在`com.packt.manage`包中创建一个名为`BookManager`的新类。这个类的主要目的是获取`BookService`的实例，然后执行其 CRUD 操作。由`ServiceLoader`加载的服务提供者决定返回的实例。`BookManager`类大致如下：

```java
        public class BookManager{ 
          public static void main(String[] args){ 
            BookService service = BookService.getInstance();
            System.out.println(service.getClass()); 
            Book book = new Book("1", "Title", "Author"); 
            service.create(book); 
            service.read("1"); 
            service.update(book); 
            service.delete("1"); 
          }
        }
```

1.  通过以下命令编译和运行我们的主模块：

```java
 $ javac -d mods --module-source-path src 
      $(find src -name *.java) 
 $ java --module-path mods -m 
      book.manage/com.packt.manage.BookManager 
 class com.packt.mongodb.MongoDbBookServiceProvider
 class com.packt.mongodb.service.MongoDbBookService
 Mongodb Create book ... Title
 Mongodb Reading book ... 1
 Mongodb Updating book ... Title
 Mongodb Deleting ... 1
```

在前面的输出中，第一行列出了可用的服务提供者，第二行列出了我们正在使用的`BookService`实现。

1.  有了一个服务提供者，看起来很简单。让我们继续添加另一个模块`sqldb.book.service`，其模块定义如下：

```java
        module sqldb.book.service{ 
          requires book.service; 
          provides com.packt.spi.BookServiceProvider 
                   with com.packt.sqldb.SqlDbBookServiceProvider; 
        }
```

1.  `com.packt.sqldb`包中的`SqlDbBookServiceProvider`类是`BookServiceProvider`接口的实现，如下所示：

```java
        @Override 
        public BookService getBookService(){     
          return new SqlDbBookService(); 
        }
```

1.  CRUD 操作的实现由`com.packt.sqldb.service`包中的`SqlDbBookService`类完成。

1.  让我们编译并运行主模块，这次使用两个服务提供程序：

```java
 $ javac -d mods --module-source-path src 
      $(find src -name *.java) 
 $ java --module-path mods -m  
      book.manage/com.packt.manage.BookManager 
 class com.packt.sqldb.SqlDbBookServiceProvider
 class com.packt.mongodb.MongoDbBookServiceProvider
 class com.packt.mongodb.service.MongoDbBookService
 Mongodb Create book ... Title
 Mongodb Reading book ... 1
 Mongodb Updating book ... Title
 Mongodb Deleting ... 1
```

前两行打印出可用服务提供程序的类名，第三行打印出我们正在使用哪个`BookService`实现。

# 使用 jlink 创建自定义模块化运行时镜像

Java 有两种版本：

+   仅 Java 运行时，也称为 JRE：支持 Java 应用程序的执行

+   带有 Java 运行时的 Java 开发工具包，也称为 JDK：支持 Java 应用程序的开发和执行

除此之外，Java 8 引入了三个紧凑配置文件，旨在提供较小的占地面积的运行时，以便在嵌入式和较小的设备上运行，如下所示：

![](img/ee1d7126-b787-41bf-8b20-96920bb76650.png)

前面的图像显示了不同的配置文件和它们支持的功能。

Java 9 引入了一个名为`jlink`的新工具，它使得可以创建模块化运行时镜像。这些运行时镜像实际上是一组模块及其依赖项的集合。有一个名为 JEP 220 的 Java 增强提案，规定了这个运行时镜像的结构。

在这个示例中，我们将使用`jlink`创建一个运行时镜像，其中包括我们的`math.util`，`banking.util`和`calculator`模块，以及 Jackson 自动模块。

# 准备工作

在*创建一个简单的模块化应用程序*的示例中，我们创建了一个简单的模块化应用程序，包括以下模块：

+   `math.util`

+   `calculator`：包括主类

我们将重用相同的模块和代码来演示`jlink`工具的使用。为了方便我们的读者，代码可以在`Chapter03/9_jlink_modular_run_time_image`中找到。

# 如何做...

1.  让我们编译这些模块：

```java
 $ javac -d mods --module-path mlib --module-source-path 
        src $(find src - name *.java)
```

1.  让我们为所有模块创建模块化 JAR：

```java
     $ jar --create --file mlib/math.util.jar -C mods/math.util . 

 $ jar --create --file=mlib/calculator.jar --main-
 class=com.packt.calculator.Calculator -C mods/calculator/ .
```

1.  让我们使用`jlink`创建一个运行时镜像，其中包括`calculator`和`math.util`模块及其依赖项：

```java
 $ jlink --module-path mlib:$JAVA_HOME/jmods --add-modules 
 calculator,math.util --output image --launcher 
 launch=calculator/com.packt.calculator.Calculator
```

运行时镜像在指定位置使用`--output`命令行选项创建。

1.  在 image 目录下创建的运行时镜像包含`bin`目录等其他目录。这个`bin`目录包含一个名为`calculator`的 shell 脚本。这可以用来启动我们的应用程序。

```java
    $ ./image/bin/launch 

 ************Advanced Calculator************
 1\. Prime Number check
 2\. Even Number check
 3\. Sum of N Primes
 4\. Sum of N Evens
 5\. Sum of N Odds
 6\. Exit
 Enter the number to choose operation
```

我们无法创建包含自动模块的模块的运行时镜像。如果 JAR 文件不是模块化的，或者没有`module-info.class`，`jlink`会报错。

# 为旧平台版本编译

在某些时候，我们使用`-source`和`-target`选项来创建 Java 构建。`-source`选项用于指示编译器接受的 Java 语言版本，`-target`选项用于指示类文件支持的版本。通常，我们忘记使用`-source`选项，默认情况下，`javac`会针对最新可用的 Java 版本进行编译。由于这个原因，有可能使用了新的 API，结果在目标版本上构建不会按预期运行。

为了克服提供两个不同命令行选项的混淆，Java 9 引入了一个新的命令行选项`--release`。这充当了`-source`，`-target`和`-bootclasspath`选项的替代。`-bootclasspath`用于提供给定版本的引导类文件的位置*N*。

# 准备工作

我们创建了一个简单的模块，名为 demo，其中包含一个非常简单的名为`CollectionsDemo`的类，该类只是将一些值放入地图并对其进行迭代，如下所示：

```java
public class CollectionsDemo{
  public static void main(String[] args){
    Map<String, String> map = new HashMap<>();
    map.put("key1", "value1");
    map.put("key2", "value3");
    map.put("key3", "value3");
    map.forEach((k,v) -> System.out.println(k + ", " + v));
  }
}
```

让我们编译并运行它以查看其输出：

```java
$ javac -d mods --module-source-path src srcdemomodule-info.java srcdemocompacktCollectionsDemo.java
$ java --module-path mods -m demo/com.packt.CollectionsDemo
```

我们得到的输出如下：

```java
key1, value1
key2, value3
key3, value3
```

现在让我们编译它以在 Java 8 上运行，然后在 Java 8 上运行它。

# 如何做...

1.  由于较旧版本的 Java，即 Java 8 及之前，不支持模块，因此如果我们在较旧版本上进行编译，就必须摆脱`module-info.java`。这就是为什么我们在编译过程中没有包括`module-info.java`。我们使用以下代码进行编译：

```java
 $ javac --release 8 -d mods srcdemocompacktCollectionsDemo.java
```

您可以看到我们使用了`--release`选项，针对 Java 8，而不是编译`module-info.java`。

1.  让我们创建一个 JAR 文件，因为这样可以更容易地传输 Java 构建，而不是复制所有类文件：

```java
 $jar --create --file mlib/demo.jar --main-class 
      com.packt.CollectionsDemo -C mods/ .
```

1.  让我们在 Java 9 上运行上述 JAR：

```java
 $ java -version
 java version "9"
 Java(TM) SE Runtime Environment (build 9+179)
 Java HotSpot(TM) 64-Bit Server VM (build 9+179, mixed mode)

 $ java -jar mlib/demo.jar
 key1, value1
 key2, value3
 key3, value3
```

1.  让我们在 Java 8 上运行这个 JAR：

```java
 $ "%JAVA8_HOME%"binjava -version 
 java version "1.8.0_121"
 Java(TM) SE Runtime Environment (build 1.8.0_121-b13)
 Java HotSpot(TM) 64-Bit Server VM (build 25.121-b13, mixed mode)

 $ "%JAVA8_HOME%"binjava -jar mlibdemo.jar
 key1, value1
 key2, value3
 key3, value3
```

如果我们在构建 Java 9 时没有使用`-release`选项会怎样？我们也试试这个：

1.  编译时不使用`--release`选项，并将生成的类文件创建为 JAR：

```java
 $ javac -d mods srcdemocompacktCollectionsDemo.java 
 $ jar --create --file mlib/demo.jar --main-class 
      com.packt.CollectionsDemo -C mods/ .
```

1.  让我们在 Java 9 上运行这个 JAR：

```java
 $ java -jar mlib/demo.jar 
 key1, value1
 key2, value3
 key3, value3
```

它按预期工作。

1.  让我们在 Java 8 上运行这个 JAR：

```java
 $ "%JAVA8_HOME%"binjava -version
 java version "1.8.0_121"
 Java(TM) SE Runtime Environment (build 1.8.0_121-b13)
 Java HotSpot(TM) 64-Bit Server VM (build 25.121-b13, mixed mode)
```

输出如下：

```java
$ java -jar mlibdemo.jar

Exception in thread "main" java.lang.UnsupportedClassVersionError:

com/packt/CollectionsDemo has been compiled by a more recent version of the Java Runtime (class file version 53.0), this version of the Java Runtime only recognizes class file versions up to 52.0
```

它清楚地说明了类文件版本不匹配。因为它是为 Java 9（版本 53.0）编译的，所以在 Java 8（版本 52.0）上无法运行。

# 它的工作原理...

编译到较旧版本所需的数据存储在`$JDK_ROOT/lib/ct.sym`文件中。此信息被`--release`选项用于定位`bootclasspath`。`ct.sym`文件是一个 ZIP 文件，其中包含与目标平台版本的类文件对应的剥离类文件（直接从[`openjdk.java.net/jeps/247`](http://openjdk.java.net/jeps/247)复制）。

# 创建多版本 JAR

在 Java 9 之前，对于库的开发人员来说，要采用语言中引入的新功能而不发布新的库版本是很困难的。但是在 Java 9 中，多版本 JAR 提供了这样一个功能，可以在使用更高版本的 Java 时捆绑某些类文件以运行。

在这个示例中，我们将向您展示如何创建这样一个多版本 JAR。

# 如何做到...

1.  为 Java 8 平台创建所需的 Java 代码。我们将在`src8compackt`目录中添加两个类，`CollectionUtil.java`和`FactoryDemo.java`：

```java
        public class CollectionUtil{
          public static List<String> list(String ... args){
            System.out.println("Using Arrays.asList");
            return Arrays.asList(args);
          }

          public static Set<String> set(String ... args){
            System.out.println("Using Arrays.asList and set.addAll");
            Set<String> set = new HashSet<>();
            set.addAll(list(args));
            return set;
          }
        }

        public class FactoryDemo{
          public static void main(String[] args){
            System.out.println(CollectionUtil.list("element1", 
                       "element2", "element3"));
            System.out.println(CollectionUtil.set("element1", 
                       "element2", "element3"));
          }
        }
```

1.  我们希望使用在 Java 9 中引入的`Collection`工厂方法。因此，我们将在`src`下创建另一个子目录，将我们的与 Java 9 相关的代码放在其中：`src9compackt`。在这里，我们将添加另一个`CollectionUtil`类：

```java
        public class CollectionUtil{
          public static List<String> list(String ... args){
            System.out.println("Using factory methods");
            return List.of(args);
          }
          public static Set<String> set(String ... args){
            System.out.println("Using factory methods");
            return Set.of(args);
          }
        }
```

1.  上述代码使用了 Java 9 集合工厂方法。使用以下命令编译源代码：

```java
 javac -d mods --release 8 src8compackt*.java
      javac -d mods9 --release 9 src9compackt*.java
```

注意使用`--release`选项为不同的 Java 版本编译代码。

1.  现在让我们创建多版本 JAR：

```java
 jar --create --file mr.jar --main-class=com.packt.FactoryDemo 
      -C mods . --release 9 -C mods9 .
```

在创建 JAR 时，我们还提到，当在 Java 9 上运行时，我们使用了 Java 9 特定的代码。

1.  我们将在 Java 9 上运行`mr.jar`：

```java
 java -jar mr.jar
 [element1, element2, element3]
 Using factory methods
 [element2, element3, element1]
```

1.  我们将在 Java 8 上运行`mr.jar`：

```java
      #Linux
 $ /usr/lib/jdk1.8.0_144/bin/java -version
 java version "1.8.0_144"
 Java(TM) SE Runtime Environment (build 1.8.0_144-b01)
 Java HotSpot(TM) 64-Bit Server VM (build 25.144-b01, mixed mode)
 $ /usr/lib/jdk1.8.0_144/bin/java -jar mr.jar
 Using Arrays.asList
 [element1, element2, element3]
 Using Arrays.asList and set.addAll
 Using Arrays.asList
 [element1, element2, element3]

 #Windows
 $ "%JAVA8_HOME%"binjava -version 
 java version "1.8.0_121"
 Java(TM) SE Runtime Environment (build 1.8.0_121-b13)
 Java HotSpot(TM) 64-Bit Server VM (build 25.121-b13, mixed mode)
 $ "%JAVA8_HOME%"binjava -jar mr.jar
 Using Arrays.asList
 [element1, element2, element3]
 Using Arrays.asList and set.addAll
 Using Arrays.asList
 [element1, element2, element3]
```

# 它的工作原理...

让我们看看`mr.jar`中内容的布局：

```java
jar -tvf mr.jar
```

JAR 的内容如下：

![](img/ea10f27a-5853-477a-a589-4beba6202ae3.png)

在上述布局中，我们有`META-INF/versions/9`，其中包含 Java 9 特定的代码。另一个重要的事情是注意`META-INF/MANIFEST.MF`文件的内容。让我们提取 JAR 并查看其内容：

```java
jar -xvf mr.jar

$ cat META-INF/MANIFEST.MF
Manifest-Version: 1.0
Created-By: 9 (Oracle Corporation)
Main-Class: com.packt.FactoryDemo
Multi-Release: true
```

新的`Multi-Release`清单属性用于指示 JAR 是否是多版本 JAR。

# 使用 Maven 开发模块化应用程序

在这个示例中，我们将使用 Maven，Java 生态系统中最流行的构建工具，开发一个简单的模块化应用程序。我们将在本章的*服务*示例中介绍的想法。

# 准备工作

我们的示例中有以下模块：

+   `book.manage`：这是与数据源交互的主模块

+   `book.service`：这是包含服务提供者接口的模块

+   `mongodb.book.service`：这是为服务提供者接口提供实现的模块

+   `sqldb.book.service`：这是为服务提供者接口提供另一个实现的模块

在本示例中，我们将创建一个 maven 项目，并将之前的 JDK 模块作为 maven 模块包含进来。让我们开始吧。

# 如何做...

1.  创建一个包含所有模块的文件夹。我们称之为`12_services_using_maven`，具有以下文件夹结构：

```java
      12_services_using_maven
 |---book-manage
 |---book-service
 |---mongodb-book-service
 |---sqldb-book-service
 |---pom.xml
```

1.  父级的`pom.xml`如下：

```java
        <?xml version="1.0" encoding="UTF-8"?>
        <project 

         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
          <modelVersion>4.0.0</modelVersion>
          <groupId>com.packt</groupId>
          <artifactId>services_using_maven</artifactId>
          <version>1.0</version>
          <packaging>pom</packaging>
          <modules>
            <module>book-service</module>
            <module>mongodb-book-service</module>
            <module>sqldb-book-service</module>
            <module>book-manage</module>
          </modules>
          <build>
            <plugins>
              <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.6.1</version>
                <configuration>
                  <source>9</source>
                  <target>9</target>
                  <showWarnings>true</showWarnings>
                  <showDeprecation>true</showDeprecation>
                </configuration>
              </plugin>
            </plugins>
          </build>
        </project>
```

1.  让我们按照以下方式创建`book-service` Maven 模块的结构：

```java
 book-service
 |---pom.xml
 |---src
 |---main
 |---book.service
 |---module-info.java
 |---com
 |---packt
 |---model
 |---Book.java
 |---service
 |---BookService.java
 |---spi
 |---BookServiceProvider.java
```

1.  `book-service` Maven 模块的`pom.xml`内容如下：

```java
        <?xml version="1.0" encoding="UTF-8"?>
        <project 

        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
        http://maven.apache.org/xsd/maven-4.0.0.xsd">
          <modelVersion>4.0.0</modelVersion>
          <parent>
            <groupId>com.packt</groupId>
            <artifactId>services_using_maven</artifactId>
            <version>1.0</version>
          </parent>
          <artifactId>book-service</artifactId>
          <version>1.0</version>
          <build>
            <sourceDirectory>src/main/book.service</sourceDirectory>
          </build>
        </project>
```

1.  这是`module-info.java`：

```java
        module book.service{
          exports com.packt.model;
          exports com.packt.service;
          exports com.packt.spi;
          uses com.packt.spi.BookServiceProvider;
       }
```

1.  这是`Book.java`：

```java
       public class Book{
         public Book(String id, String title, String author){
           this.id = id;
           this.title = title
           this.author = author;
         }
         public String id;
         public String title;
         public String author;
       }
```

1.  这是`BookService.java`：

```java
 public abstract class BookService{
 public abstract void create(Book book); 
 public abstract Book read(String id); 
 public abstract void update(Book book); 
 public abstract void delete(String id);
 public static BookService getInstance(){ 
   ServiceLoader<BookServiceProvider> sl =     
        ServiceLoader.load(BookServiceProvider.class);          
   Iterator<BookServiceProvider> iter = sl.iterator();        
   if (!iter.hasNext())
      throw new RuntimeException("No service providers found!");              
   BookServiceProvider provider = null;        
   while(iter.hasNext()){
       provider = iter.next();
       System.out.println(provider.getClass());        
   }        
   return provider.getBookService(); 
   }
 }
```

1.  这是`BookServiceProvider.java`：

```java
        public interface BookServiceProvider{
          public BookService getBookService();
        }
```

同样，我们定义了另外三个 Maven 模块，`mongodb-book-service`，`sqldb-book-service`和`book-manager`。此代码可以在`Chapter03/12_services_using_maven`找到。

我们可以使用以下命令编译类并构建所需的 JAR 文件：

```java
mvn clean install
```

我们提供了`run-with-mongo.*`来使用`mongodb-book-service`作为服务提供者实现，以及`run-with-sqldb.*`来使用`sqldb-book-service`作为服务提供者实现。

这个示例的完整代码可以在`Chapter03/12_services_using_maven`找到。

# 使您的库模块路径友好

要使应用程序完全模块化，它应该自身模块化以及其依赖项。现在，使第三方模块化不在应用程序开发人员的手中。一种方法是将第三方`jar`包含在模块路径中，并使用`jar`的名称作为模块的名称来声明依赖关系。在这种情况下，`jar`将成为自动模块。这是可以的，但通常`jar`的名称不符合模块名称的规范。在这种情况下，我们可以利用 JDK 9 中添加的另一种支持，其中可以在`jar`的`MANIFEST.mf`文件中定义`jar`的名称，库使用者可以声明对定义名称的依赖关系。这样，将来，库开发人员可以将他们的库模块化，同时仍然使用相同的模块名称。

在这个示例中，我们将向您展示如何为从非模块化`jar`创建的自动模块提供名称。首先，我们将向您展示如何使用 maven 实现这一点，然后在*更多内容*部分中，我们将看到如何在不使用任何构建工具的情况下创建一个 JAR。

# 准备工作

您至少需要 JDK 9 来运行这个示例，但我们将在 Maven 构建插件中使用 JDK 11。您还需要安装 Maven 才能使用它。您可以在互联网上搜索 Maven 的安装过程。

# 如何做...

1.  使用 Maven 生成一个空项目：

```java
mvn archetype:generate -DgroupId=com.packt.banking -DartifactId=13_automatic_module -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

1.  通过复制以下依赖项，更新位于`13_automatic_module`目录中的`pom.xml`文件中的依赖项：

```java
<dependencies>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
  </dependency>
  <dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-core</artifactId>
    <version>3.10.0</version>
    <scope>test</scope>
  </dependency>
</dependencies>
```

1.  我们需要配置`maven-compiler-plugin`以便能够编译 JDK 11。因此，我们将在`<dependencies></dependencies>`之后添加以下插件配置：

```java
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.6.1</version>
      <configuration>
        <source>11</source>
        <target>11</target>
        <showWarnings>true</showWarnings>
        <showDeprecation>true</showDeprecation>
      </configuration>
    </plugin>
  </plugins>
</build>
```

1.  配置`maven-jar-plugin`，通过在新的`<Automatic-Module-Name>`标签中提供名称来提供自动模块名称，如下所示：

```java
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-jar-plugin</artifactId>
  <configuration>
    <archive>
      <manifestEntries>
        <Automatic-Module-Name>com.packt.banking</Automatic-Module-
         Name>
      </manifestEntries>
    </archive>
  </configuration>
</plugin>
```

1.  我们将在`com.packt.banking.Banking`类中添加一个用于计算简单利息的 API，如下所示：

```java
public class Banking {
  public static Double simpleInterest(Double principal, 
                            Double rateOfInterest, Integer years){
    Objects.requireNonNull(principal, "Principal cannot be null");
    Objects.requireNonNull(rateOfInterest,  
                               "Rate of interest cannot be null");
    Objects.requireNonNull(years, "Years cannot be null");
    return ( principal * rateOfInterest * years ) / 100;
  }
}
```

1.  我们还添加了一个测试，您可以在本章下载的代码中的`Chapter03\13_automatic_module\src\test\java\com\packt\banking`找到。让我们运行`mvn package`命令来构建一个 JAR。如果一切顺利，您将看到以下内容：

![](img/20cf9b91-b447-44bd-be79-2ae8dc64ce9c.png)

1.  您可以使用任何压缩实用程序，例如 7z，来查看 JAR 的内容，特别是`Manifest.MF`文件，其内容如下：

```java
Manifest-Version: 1.0
Archiver-Version: Plexus Archiver
Created-By: Apache Maven 3.3.9
Built-By: sanaulla
Build-Jdk: 11-ea
Automatic-Module-Name: com.packt.banking
```

这些步骤的代码可以在`Chapter03\13_automatic_module`找到。

# 它是如何工作的...

到目前为止，我们已经创建了一个具有自动模块名称的 Java 库 JAR。现在，让我们看看如何在模块化应用程序中将这个非模块化 JAR 用作自动模块。这个示例的完整代码可以在`Chapter03\13_using_automatic_module`找到。

让我们将在`How to do it...`部分创建的`jar`文件复制到`13_automatic_module\target\13_automatic_module-1.0.jar`中，然后放入`13_using_automatic_module\mods`文件夹中。这样我们即可让即将创建的模块化应用程序使用随`jar`一起提供的`com.packt.banking`模块。

复制 jar 文件后，我们需要为我们的模块创建模块定义，并在`module-info.java`中声明其依赖项，放置在`13_using_automatic_module\src\banking.demo`中：

```java
module banking.demo{
    requires com.packt.banking;
}
```

接下来是创建`main` `com.packt.demo.BankingDemo`类，它将使用银行工具。创建路径为`13_using_automatic_module\src\banking.demo\com\packt\demo`，如下所示：

```java
package com.packt.demo;
import com.packt.banking.Banking;
public class BankingDemo{
  public static void main(String[] args) {
    Double principal = 1000.0;
    Double rateOfInterest = 10.0;
    Integer years = 2;
    Double simpleInterest = Banking.simpleInterest(principal, 
                                      rateOfInterest, years);
        System.out.println("The simple interest is: " + 
                                             simpleInterest);
    }
}
```

我们可以通过使用从`13_using_automatic_module`执行的以下命令来编译前面的代码：

```java
javac -d mods -p mods --module-source-path src src\banking.demo\*.java src\banking.demo\com\packt\demo\*.java
```

然后通过使用从相同位置执行的以下命令来运行前面的代码：

```java
java --module-path mods -m banking.demo/com.packt.demo.BankingDemo
```

您将看到以下输出：

```java
The simple interest is: 200.0
```

注意：您可以使用`run.bat`或`run.sh`脚本来编译和运行代码。

因此，通过这样做，我们有：

+   创建一个非模块化的 JAR，并自动命名模块。

+   通过使用其自动模块名称声明对其的依赖，将非模块化的 JAR 用作自动模块。

您还将看到，我们已完全删除了对类路径的使用，而仅使用模块路径；这是我们迈向完全模块化应用程序的第一步。

# 还有更多...

我们将向您展示如何创建您的银行实用程序的 JAR，以及自动模块名称（如果您不使用 Maven）。此代码可以在`Chapter03\13_automatic_module_no_maven`中找到。我们仍将把`Banking .java`复制到`13_automatic_module_no_maven\src\com\packt\banking`目录中。

接下来，我们需要定义一个包含以下自动模块名称的`manifest.mf`清单文件：

```java
Automatic-Module-Name: com.packt.banking
```

我们可以通过从`Chapter03\13_automatic_module_no_maven`发出以下命令来编译前面的类：

```java
javac -d classes src/com/packt/banking/*.java
```

然后通过从相同位置发出以下命令来构建一个`jar`：

```java
jar cvfm banking-1.0.jar manifest.mf -C classes .
```

我们还提供了用于创建您的`jar`的脚本。您可以使用`build-jar.bat`或`build-jar.sh`来编译和创建`jar`。现在，您可以将`banking-1.0.jar`复制到`Chapter03\13_using_automatic_module\mods`并替换`13_automati_module-1.0.jar`。然后，使用`run.bat`或`run.sh`脚本在`Chapter03\13_using_automatic_module`中运行代码，具体取决于您的平台。您仍将看到与上一节相同的输出。

# 如何为反射打开模块

模块系统引入了严格的封装，如果类没有明确允许反射，则其私有成员不能通过反射访问。大多数库，如 hibernate 和 Jackson，依赖于反射来实现其目的。模块系统提供的严格封装将立即破坏这些库在新的 JDK 9 及更高版本上的运行。

为了支持这样重要的库，Java 团队决定引入功能，模块开发人员可以声明一些包或完整包，以便通过反射进行检查。在本教程中，我们将看看如何确切地实现这一点。

# 准备工作

您需要安装 JDK 9 或更高版本。在本教程中，我们将使用 Jackson API，其`jar`文件可以在本书的代码下载的`Chapter03/14_open_module_for_rflxn/mods`中找到。这些`jar`文件很重要，因为我们将使用 Jackson API 从 Java 对象创建 JSON 字符串。这些 Jackson API 将被用作自动模块。

# 如何做...

1.  在`14_open_module_for_rflxn/src/demo/com/packt/demo`中创建一个`Person`类，定义如下：

```java
package com.packt.demo;

import java.time.LocalDate;

public class Person{
    public Person(String firstName, String lastName, 
        LocalDate dob, String placeOfBirth){
        this.firstName = firstName;
        this.lastName = lastName;
        this.dob = dob;
        this.placeOfBirth = placeOfBirth;
    }
    public final String firstName;
    public final String lastName;
    public final LocalDate dob;
    public final String placeOfBirth;
}
```

1.  创建一个`OpenModuleDemo`类，该类创建一个`Person`类的实例，并使用`com.fasterxml.jackson.databind.ObjectMapper`将其序列化为 JSON。新日期时间 API 的序列化需要对`ObjectMapper`实例进行一些配置更改，这也在静态初始化块中完成，如下所示：

```java
package com.packt.demo;

import java.time.LocalDate;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;

public class OpenModuleDemo{
    final static ObjectMapper MAPPER = new ObjectMapper();
    static{
        MAPPER.registerModule(new JavaTimeModule());
        MAPPER.configure(SerializationFeature.
                         WRITE_DATES_AS_TIMESTAMPS, false);
    }
    public static void main(String[] args) 
        throws Exception {
        Person p = new Person("Mohamed", "Sanaulla", 
            LocalDate.now().minusYears(30), "India");
        String json = MAPPER.writeValueAsString(p);
        System.out.println("The Json for Person is: ");
        System.out.println(json);
    }
}
```

1.  在`14_open_module_for_rflxn/src/demo`中创建`module-info.java`，它声明了模块的名称、其依赖关系，以及另一个有趣的东西叫做`opens`。`opens`是允许外部库进行反射的解决方案，如下所示：

```java
module demo{
    requires com.fasterxml.jackson.annotation;
    requires com.fasterxml.jackson.core;
    requires com.fasterxml.jackson.databind;
    requires com.fasterxml.jackson.datatype.jsr310;
    opens com.packt.demo;
}
```

# 它是如何工作的...

有两种方法可以打开模块以供反射检查：

+   在模块级别上声明开放：

```java
open module demo { }
```

+   在单个包级别上声明开放：

```java
module demo { 
    opens com.packt.demo;
}
```

后者比前者更加严格（即只能使一个包对反射可用）。还有另一种方法可以实现这一点，那就是将特定包导出到正确的 Jackson 包，如下所示：

```java
module demo{
   exports com.packt.demo to <relevant Jackson package here>
}
```
