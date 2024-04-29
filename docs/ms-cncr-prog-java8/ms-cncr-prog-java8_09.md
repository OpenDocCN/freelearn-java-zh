# 第八章：使用并行流处理大型数据集-映射和收集模型

在第七章中，*使用并行流处理大型数据集-映射和减少模型*，我们介绍了流的概念，这是 Java 8 的新功能。流是可以以并行或顺序方式处理的元素序列。在本章中，您将学习如何处理流，内容包括以下主题：

+   collect()方法

+   第一个例子-没有索引的搜索数据

+   第二个例子-推荐系统

+   第三个例子-社交网络中的常见联系人

# 使用流来收集数据

在第七章中，*使用并行流处理大型数据集-映射和减少模型*，我们对流进行了介绍。让我们记住它们最重要的特点：

+   流的元素不存储在内存中

+   流不能重复使用

+   流对数据进行延迟处理

+   流操作不能修改流源

+   流允许您链接操作，因此一个操作的输出是下一个操作的输入

流由以下三个主要元素组成：

+   生成流元素的源

+   零个或多个生成另一个流作为输出的中间操作

+   生成结果的一个终端操作，可以是简单对象、数组、集合、映射或其他任何东西

Stream API 提供了不同的终端操作，但有两个更重要的操作，因为它们具有灵活性和强大性。在第七章中，*使用并行流处理大型数据集-映射和减少模型*，您学习了如何使用 reduce()方法，在本章中，您将学习如何使用 collect()方法。让我们介绍一下这个方法。

## collect()方法

collect()方法允许您转换和分组流的元素，生成一个新的数据结构，其中包含流的最终结果。您可以使用最多三种不同的数据类型：输入数据类型，来自流的输入元素的数据类型，用于在 collect()方法运行时存储元素的中间数据类型，以及 collect()方法返回的输出数据类型。

collect()方法有两个不同的版本。第一个版本接受以下三个函数参数：

+   供应商：这是一个创建中间数据类型对象的函数。如果您使用顺序流，此方法将被调用一次。如果您使用并行流，此方法可能会被多次调用，并且必须每次产生一个新的对象。

+   **累加器**：此函数用于处理输入元素并将其存储在中间数据结构中。

+   **组合器**：此函数用于将两个中间数据结构合并为一个。此函数仅在并行流中调用。

这个版本的 collect()方法使用两种不同的数据类型：来自流的元素的输入数据类型和将用于存储中间元素并返回最终结果的中间数据类型。

collect()方法的第二个版本接受实现 Collector 接口的对象。您可以自己实现这个接口，但使用 Collector.of()静态方法会更容易。此方法的参数如下：

+   **供应商**：此函数创建中间数据类型的对象，并且它的工作方式如前所述

+   **累加器**：调用此函数来处理输入元素，必要时对其进行转换，并将其存储在中间数据结构中

+   **组合器**：调用此函数将两个中间数据结构合并为一个，它的工作方式如前所述

+   **完成器**：如果需要进行最终转换或计算，则调用此函数将中间数据结构转换为最终数据结构

+   **特征**：您可以使用这个最终变量参数来指示您正在创建的收集器的一些特征

实际上，这两个版本之间有轻微的区别。三参数 collect 接受一个组合器，即`BiConsumer`，它必须将第二个中间结果合并到第一个中间结果中。与此不同的是，这个组合器是`BinaryOperator`，应该返回组合器。因此，它有自由地将第二个合并到第一个中间结果中，或者将第一个合并到第二个中间结果中，或者创建一个新的中间结果。`of()`方法还有另一个版本，它接受相同的参数，除了完成器；在这种情况下，不执行完成转换。

Java 为您提供了`Collectors`工厂类中的一些预定义收集器。您可以使用其中的一个静态方法获取这些收集器。其中一些方法是：

+   `averagingDouble()`，`averagingInt()`和`averagingLong()`：这将返回一个收集器，允许您计算`double`，`int`或`long`函数的算术平均值。

+   `groupingBy()`: 这将返回一个收集器，允许您根据对象的属性对流的元素进行分组，生成一个映射，其中键是所选属性的值，值是具有确定值的对象的列表。

+   `groupingByConcurrent()`: 这与前一个方法类似，除了两个重要的区别。第一个区别是它在并行模式下可能比`groupingBy()`方法更快，但在顺序模式下可能更慢。第二个最重要的区别是`groupingByConcurrent()`函数是一个无序的收集器。列表中的项目不能保证与流中的顺序相同。另一方面，`groupingBy()`收集器保证了顺序。

+   `joining()`: 这将返回一个`Collector`工厂类，将输入元素连接成一个字符串。

+   `partitioningBy()`: 这将返回一个`Collector`工厂类，根据谓词的结果对输入元素进行分区。

+   `summarizingDouble()`，`summarizingInt()`和`summarizingLong()`：这些返回一个`Collector`工厂类，用于计算输入元素的摘要统计信息。

+   `toMap()`: 这将返回一个`Collector`工厂类，允许您根据两个映射函数将输入元素转换为一个映射。

+   `toConcurrentMap()`: 这与前一个方法类似，但是以并发方式进行。没有自定义合并器，`toConcurrentMap()`对于并行流只是更快。与`groupingByConcurrent()`一样，这也是一个无序的收集器，而`toMap()`使用遇到的顺序进行转换。

+   `toList()`: 这将返回一个`Collector`工厂类，将输入元素存储到一个列表中。

+   `toCollection()`: 这个方法允许你将输入元素累积到一个新的`Collection`工厂类（`TreeSet`，`LinkedHashSet`等）中，按照遇到的顺序。该方法接收一个`Supplier`接口的实现作为参数，用于创建集合。

+   `maxBy()`和`minBy()`：这将返回一个`Collector`工厂类，根据传递的比较器产生最大和最小的元素。

+   `toSet()`: 这将返回一个`Collector`，将输入元素存储到一个集合中。

# 第一个例子 - 在没有索引的情况下搜索数据

在第七章中，*使用并行流处理大规模数据集 - 映射和归约模型*，您学习了如何实现搜索工具，以查找与输入查询类似的文档，使用倒排索引。这种数据结构使搜索操作更容易和更快，但会有情况，您将不得不对大量数据进行搜索操作，并且没有倒排索引来帮助您。在这些情况下，您必须处理数据集的所有元素才能获得正确的结果。在本例中，您将看到其中一种情况以及`Stream` API 的`reduce()`方法如何帮助您。

为了实现这个例子，您将使用**亚马逊产品共购买网络元数据**的子集，其中包括亚马逊销售的 548,552 个产品的信息，包括标题、销售排名以及相似产品、分类和评论列表。您可以从[`snap.stanford.edu/data/amazon-meta.html`](https://snap.stanford.edu/data/amazon-meta.html)下载这个数据集。我们已经取出了前 20,000 个产品，并将每个产品记录存储在单独的文件中。我们已更改了一些字段的格式，以便简化数据处理。所有字段都具有`property:value`格式。

## 基本类

我们有一些在并发和串行版本之间共享的类。让我们看看每个类的细节。

### Product 类

`Product`类存储有关产品的信息。以下是`Product`类：

+   `id`：这是产品的唯一标识符。

+   `asin`：这是亚马逊的标准识别号。

+   `title`：这是产品的标题。

+   `group`：这是产品的组。该属性可以取值`Baby Product`、`Book`、`CD`、`DVD`、`Music`、`Software`、`Sports`、`Toy`、`Video`或`Video Games`。

+   `salesrank`：这表示亚马逊的销售排名。

+   `similar`：这是文件中包含的相似商品的数量。

+   `categories`：这是一个包含产品分类的`String`对象列表。

+   `reviews`：这是一个包含产品评论（用户和值）的`Review`对象列表。

该类仅包括属性的定义和相应的`getXXX()`和`setXXX()`方法，因此其源代码未包含在内。

### 评论类

正如我们之前提到的，`Product`类包括一个`Review`对象列表，其中包含用户对产品的评论信息。该类将每个评论的信息存储在以下两个属性中：

+   `user`：进行评论的用户的内部代码

+   `value`：用户对产品给出的评分

该类仅包括属性的定义和相应的`getXXX()`和`setXXX()`方法，因此其源代码未包含在内。

### ProductLoader 类

`ProductLoader`类允许您从文件加载产品的信息到`Product`对象中。它实现了`load()`方法，该方法接收一个包含产品信息文件路径的`Path`对象，并返回一个`Product`对象。以下是其源代码：

```java
public class ProductLoader {
    public static Product load(Path path) {
        try (BufferedReader reader = Files.newBufferedReader(path)) {
            Product product=new Product();
            String line=reader.readLine();
            product.setId(line.split(":")[1]);
            line=reader.readLine();
            product.setAsin(line.split(":")[1]);
            line=reader.readLine();
            product.setTitle(line.substring (line.indexOf(':')+1));
            line=reader.readLine();
            product.setGroup(line.split(":")[1]);
            line=reader.readLine();
            product.setSalesrank(Long.parseLong (line.split(":")[1]));
            line=reader.readLine();
            product.setSimilar(line.split(":")[1]);
            line=reader.readLine();

            int numItems=Integer.parseInt(line.split(":")[1]);

            for (int i=0; i<numItems; i++) {
                line=reader.readLine();
                product.addCategory(line.split(":")[1]);
            }

            line=reader.readLine();
            numItems=Integer.parseInt(line.split(":")[1]);
            for (int i=0; i<numItems; i++) {
                line=reader.readLine();
                String tokens[]=line.split(":");
                Review review=new Review();
                review.setUser(tokens[1]);
                review.setValue(Short.parseShort(tokens[2]));
                product.addReview(review);
            }
            return product;
        } catch (IOException x) {
            throw newe UncheckedIOException(x);
        } 

    }
}
```

## 第一种方法 - 基本搜索

第一种方法接收一个单词作为输入查询，并搜索存储产品信息的所有文件，无论该单词是否包含在定义产品的字段中的一个中。它只会显示包含该单词的文件的名称。

为了实现这种基本方法，我们实现了`ConcurrentMainBasicSearch`类，该类实现了`main()`方法。首先，我们初始化查询和存储所有文件的基本路径：

```java
public class ConcurrentMainBasicSearch {

    public static void main(String args[]) {
        String query = args[0];
        Path file = Paths.get("data");
```

我们只需要一个流来生成以下结果的字符串列表：

```java
        try {
            Date start, end;
            start = new Date();
            ConcurrentLinkedDeque<String> results = Files
                    .walk(file, FileVisitOption.FOLLOW_LINKS)
                    .parallel()
                    .filter(f -> f.toString().endsWith(".txt"))
                    .collect(ArrayList<String>::new,
                            new ConcurrentStringAccumulator (query),
                            List::addAll);
            end = new Date();
```

我们的流包含以下元素：

+   我们使用`Files`类的`walk()`方法启动流，将我们文件集合的基本`Path`对象作为参数传递。该方法将返回所有文件和存储在该路径下的目录作为流。

+   然后，我们使用`parallel()`方法将流转换为并发流。

+   我们只对以`.txt`扩展名结尾的文件感兴趣，因此我们使用`filter()`方法对它们进行过滤。

+   最后，我们使用`collect()`方法将`Path`对象的流转换为`ConcurrentLinkedDeque`对象，其中包含文件名的`String`对象。

我们使用`collect()`方法的三个参数版本，使用以下功能参数：

+   **供应商**：我们使用`ArrayList`类的`new`方法引用来为每个线程创建一个新的数据结构，以存储相应的结果。

+   **累加器**：我们在`ConcurrentStringAccumulator`类中实现了自己的累加器。稍后我们将描述这个类的细节。

+   **组合器**：我们使用`ConcurrentLinkedDeque`类的`addAll()`方法来连接两个数据结构。在这种情况下，第二个集合中的所有元素将被添加到第一个集合中。第一个集合将用于进一步组合或作为最终结果。

最后，我们在控制台中写入流获得的结果：

```java
            System.out.println("Results for Query: "+query);
            System.out.println("*************");
            results.forEach(System.out::println);
            System.out.println("Execution Time: "+(end.getTime()- start.getTime()));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

每当我们要处理流的路径以评估是否将其名称包含在结果列表中时，累加器功能参数将被执行。为了实现这个功能，我们实现了`ConcurrentStringAccumulator`类。让我们看看这个类的细节。

### ConcurrentStringAccumulator 类

`ConcurrentStringAccumulator`类加载包含产品信息的文件，以确定是否包含查询的术语。它实现了`BiConsumer`接口，因为我们希望将其用作`collect()`方法的参数。我们已经使用`List<String>`和`Path`类对该接口进行了参数化：

```java
public class ConcurrentStringAccumulator implements BiConsumer<List<String>, Path> {
```

它将查询定义为内部属性，在构造函数中初始化如下：

```java
    private String word;

    public ConcurrentStringAccumulator (String word) {
        this.word=word.toLowerCase();
    }
```

然后，我们实现了`BiConsumer`接口中定义的`accept()`方法。该方法接收两个参数：`ConcurrentLinkedDeque<String>`类和`Path`类中的一个。

为了加载文件并确定它是否包含查询，我们使用以下流：

```java
    @Override
    public void accept(List<String> list, Path path) {
        boolean result;

try (Stream<String> lines = Files.lines(path)) {
            result = lines
                    .parallel()
                    .map(l -> l.split(":")[1].toLowerCase())
                    .anyMatch(l -> l.contains(word))
```

我们的流包含以下元素：

+   我们使用`Files`类的`lines()`方法创建`String`对象的流，在 try-with-resources 语句中。该方法接收一个指向文件的`Path`对象作为参数，并返回文件的所有行的流。

+   然后，我们使用`parallel()`方法将流转换为并发流。

+   然后，我们使用`map()`方法获取每个属性的值。正如我们在本节的介绍中提到的，每行都具有`property:value`格式。

+   最后，我们使用`anyMatch()`方法来知道是否有任何属性的值包含查询词。

如果计数变量的值大于`0`，则文件包含查询词，我们将文件名包含在结果的`ConcurrentLinkedDeque`类中：

```java
            if (counter>0) {
                list.add(path.toString());
            }
        } catch (Exception e) {
            System.out.println(path);
            e.printStackTrace();
        }
    }

}
```

## 第二种方法-高级搜索

我们的基本搜索有一些缺点：

+   我们在所有属性中寻找查询词，但也许我们只想在其中一些属性中寻找，例如标题

+   我们只显示文件的名称，但如果我们显示额外信息，如产品的标题，将更具信息性

为了解决这些问题，我们将实现实现`main()`方法的`ConcurrentMainSearch`类。首先，我们初始化查询和存储所有文件的基本`Path`对象：

```java
public class ConcurrentMainSearch {
    public static void main(String args[]) {
        String query = args[0];
        Path file = Paths.get("data");
```

然后，我们使用以下流生成`Product`对象的`ConcurrentLinkedDeque`类：

```java
        try {
            Date start, end;
            start=new Date();
            ConcurrentLinkedDeque<Product> results = Files
                    .walk(file, FileVisitOption.FOLLOW_LINKS)
                    .parallel()
                    .filter(f -> f.toString().endsWith(".txt"))
                    .collect(ArrayList<Product>::new,
                            new ConcurrentObjectAccumulator (query),
                            List::addAll);
            end=new Date();
```

这个流与我们在基本方法中实现的流具有相同的元素，有以下两个变化：

+   在`collect()`方法中，我们在累加器参数中使用`ConcurrentObjectAccumulator`类

+   我们使用`Product`类参数化`ConcurrentLinkedDeque`类

最后，我们将结果写入控制台，但在这种情况下，我们写入每个产品的标题：

```java
            System.out.println("Results");
            System.out.println("*************");
            results.forEach(p -> System.out.println(p.getTitle()));
            System.out.println("Execution Time: "+(end.getTime()- start.getTime()));

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

您可以更改此代码以写入有关产品的任何信息，如销售排名或类别。

这个实现与之前的实现之间最重要的变化是`ConcurrentObjectAccumulator`类。让我们看看这个类的细节。

### ConcurrentObjectAccumulator 类

`ConcurrentObjectAccumulator`类实现了参数化为`ConcurrentLinkedDeque<Product>`和`Path`类的`BiConsumer`接口，因为我们希望在`collect()`方法中使用它。它定义了一个名为`word`的内部属性来存储查询词。这个属性在类的构造函数中初始化：

```java
public class ConcurrentObjectAccumulator implements
        BiConsumer<List<Product>, Path> {

    private String word;

    public ConcurrentObjectAccumulator(String word) {
        this.word = word;
    }
```

`accept()`方法的实现（在`BiConsumer`接口中定义）非常简单：

```java
    @Override
    public void accept(List<Product> list, Path path) {

        Product product=ProductLoader.load(path);

        if (product.getTitle().toLowerCase().contains (word.toLowerCase())) {
            list.add(product);
        }

    }

}
```

该方法接收指向我们要处理的文件的`Path`对象作为参数，并使用`ConcurrentLinkedDeque`类来存储结果。我们使用`ProductLoader`类将文件加载到`Product`对象中，然后检查产品的标题是否包含查询词。如果包含查询词，我们将`Product`对象添加到`ConcurrentLinkedDeque`类中。

## 示例的串行实现

与本书中的其他示例一样，我们已经实现了搜索操作的两个版本的串行版本，以验证并行流是否能够提高性能。

您可以通过删除`Stream`对象中的`parallel()`调用来实现前面描述的四个类的串行等效版本，以使流并行化。

我们已经包含了书籍的源代码，其中包括`SerialMainBasicSearch`、`SerialMainSearch`、`SerialStringAccumulator`和`SerialObjectAccumulator`类，它们是串行版本的等效类，其中包括前面注释的更改。

## 比较实现

我们已经测试了我们的实现（两种方法：串行和并行版本）以比较它们的执行时间。为了测试它们，我们使用了三个不同的查询：

+   模式

+   Java

+   树

对于每个查询，我们已经执行了串行和并行流的两个搜索操作（基本和对象）。我们使用了 JMH 框架（[`openjdk.java.net/projects/code-tools/jmh/`](http://openjdk.java.net/projects/code-tools/jmh/)）来执行它们，该框架允许您在 Java 中实现微基准测试。使用基准测试框架比简单地使用`currentTimeMillis()`或`nanoTime()`等方法来测量时间更好。我们在一个四核处理器的计算机上执行了 10 次，并计算了这 10 次的平均执行时间。以下是以毫秒为单位的结果：

|   | 字符串搜索 | 对象搜索 |
| --- | --- | --- |
|   | **Java** | **模式** | **树** | **Java** | **模式** | **树** |
| **串行** | 4318.551 | 4372.565 | 4364.674 | 4573.985 | 4588.957 | 4591.100 |
| **并行** | 32402.969 | 2428.729 | 2412.747 | 2190.053 | 2173.511 | 2173.936 |

我们可以得出以下结论：

+   不同查询的结果非常相似

+   使用串行流，字符串搜索的执行时间比对象搜索的执行时间更好

+   使用并行流，对象搜索的执行时间比字符串搜索的执行时间更好

+   并行流在所有情况下都比串行流获得更好的性能

例如，如果我们比较并行和串行版本，对于使用速度提升的查询模式进行对象搜索，我们会得到以下结果：

![比较实现](img/00023.jpeg)

# 第二个示例 - 推荐系统

**推荐系统**根据客户购买/使用的产品/服务以及购买/使用与他相同服务的用户购买/使用的产品/服务向客户推荐产品或服务。

我们已经使用了前一节中解释的示例来实现推荐系统。每个产品的描述都包括一些客户对产品的评论。这个评论包括客户对产品的评分。

在这个例子中，您将使用这些评论来获取对客户可能感兴趣的产品的列表。我们将获取客户购买的产品列表。为了获取该列表，我们对购买这些产品的用户列表以及这些用户购买的产品列表进行排序，使用评论中给出的平均分数。这将是用户的建议产品。

## 通用类

我们已经添加了两个新的类到前一节中使用的类中。这些类是：

+   `ProductReview`：这个类通过添加两个新属性扩展了产品类

+   `ProductRecommendation`：这个类存储了对产品的推荐的信息

让我们看看这两个类的细节。

### ProductReview 类

`ProductReview`类通过添加两个新属性扩展了`Product`类：

+   `buyer`：这个属性存储产品的客户的姓名

+   `value`：这个属性存储客户在评论中给产品的评分

该类包括属性的定义：相应的`getXXX()`和`setXXX()`方法，一个从`Product`对象创建`ProductReview`对象的构造函数，以及新属性的值。它非常简单，所以它的源代码没有包含在内。

### ProductRecommendation 类

`ProductRecommendation`类存储了产品推荐所需的信息，包括以下内容：

+   `title`：我们正在推荐的产品的标题

+   `value`：该推荐的分数，计算为该产品所有评论的平均分数

这个类包括属性的定义，相应的`getXXX()`和`setXXX()`方法，以及`compareTo()`方法的实现（该类实现了`Comparable`接口），这将允许我们按照其值的降序对推荐进行排序。它非常简单，所以它的源代码没有包含在内。

## 推荐系统 - 主类

我们已经在`ConcurrentMainRecommendation`类中实现了我们的算法，以获取推荐给客户的产品列表。这个类实现了`main()`方法，该方法接收客户的 ID 作为参数，我们想要获取推荐的产品。我们有以下代码：

```java
    public static void main(String[] args) {
        String user = args[0];
        Path file = Paths.get("data");
        try {
            Date start, end;
            start=new Date();
```

我们已经使用不同的流来转换最终解决方案中的数据。第一个加载整个`Product`对象列表的流来自其文件：

```java
            List<Product> productList = Files
                .walk(file, FileVisitOption.FOLLOW_LINKS)
                .parallel()
                .filter(f -> f.toString().endsWith(".txt"))
                .collect(ConcurrentLinkedDeque<Product>::new
                 ,new ConcurrentLoaderAccumulator(), ConcurrentLinkedDeque::addAll);
```

这个流有以下元素：

+   我们使用`Files`类的`walk()`方法开始流。这个方法将创建一个流来处理数据目录下的所有文件和目录。

+   然后，我们使用`parallel()`方法将流转换为并发流。

+   然后，我们只获取扩展名为`.txt`的文件。

+   最后，我们使用`collect()`方法来获取`ConcurrentLinkedDeque`类的`Product`对象。它与前一节中使用的方法非常相似，不同之处在于我们使用了另一个累加器对象。在这种情况下，我们使用`ConcurrentLoaderAccumulator`类，稍后我们将对其进行描述。

一旦我们有了产品列表，我们将使用客户的标识符作为地图的键来组织这些产品。我们使用`ProductReview`类来存储产品的客户信息。我们将创建与`Product`有关的评论数量相同的`ProductReview`对象。我们使用以下流进行转换：

```java
        Map<String, List<ProductReview>> productsByBuyer=productList
                .parallelStream()
                .<ProductReview>flatMap(p -> p.getReviews().stream().map(r -> new ProductReview(p, r.getUser(), r.getValue())))
                .collect(Collectors.groupingByConcurrent( p -> p.getBuyer()));
```

这个流有以下元素：

+   我们使用`productList`对象的`parallelStream()`方法开始流，因此我们创建了一个并发流。

+   然后，我们使用`flatMap()`方法将我们拥有的`Product`对象流转换为唯一的`ProductReview`对象流。

+   最后，我们使用`collect()`方法生成最终的映射。在这种情况下，我们使用`Collectors`类的`groupingByConcurrent()`方法生成的预定义收集器。返回的收集器将生成一个映射，其中键将是买家属性的不同值，值将是购买该用户的产品信息的`ProductReview`对象列表。如方法名称所示，此转换将以并发方式完成。

下一个流是此示例中最重要的流。我们获取客户购买的产品，并为该客户生成推荐。这是一个由一个流完成的两阶段过程。在第一阶段，我们获取购买原始客户购买的产品的用户。在第二阶段，我们生成一个包含这些客户购买的产品以及这些客户所做的所有产品评论的映射。以下是该流的代码：

```java
            Map<String,List<ProductReview>> recommendedProducts=productsByBuyer.get(user)
                    .parallelStream()
                    .map(p -> p.getReviews())
                    .flatMap(Collection::stream)
                    .map(r -> r.getUser())
                    .distinct()
                    .map(productsByBuyer::get)
                    .flatMap(Collection::stream)
                    .collect(Collectors.groupingByConcurrent(p -> p.getTitle()));
```

在该流中，我们有以下元素：

+   首先，我们获取用户购买的产品列表，并使用`parallelStream()`方法生成并发流。

+   然后，我们使用`map()`方法获取该产品的所有评论。

+   此时，我们有一个`List<Review>`流。我们将该流转换为`Review`对象的流。现在我们有了一个包含用户购买产品的所有评论的流。

+   然后，我们将该流转换为包含进行评论的用户名称的`String`对象流。

+   然后，我们使用`distinct()`方法获取用户的唯一名称。现在我们有一个包含购买与原始用户相同产品的用户名称的`String`对象流。

+   然后，我们使用`map()`方法将每个客户转换为其购买产品的列表。

+   此时，我们有一个`List<ProductReview>`对象的流。我们使用`flatMap()`方法将该流转换为`ProductReview`对象的流。

+   最后，我们使用`collect()`方法和`groupingByConcurrent()`收集器生成产品的映射。映射的键将是产品的标题，值将是先前获得的客户所做的评论的`ProductReview`对象列表。

要完成我们的推荐算法，我们需要最后一步。对于每个产品，我们想计算其评论的平均分，并按降序对列表进行排序，以便将评分最高的产品显示在第一位。为了进行该转换，我们使用了额外的流：

```java
        List<ProductRecommendation> recommendations = recommendedProducts
                    .entrySet()
                    .parallelStream()
                    .map(entry -> new
                     ProductRecommendation(
                         entry.getKey(),
                         entry.getValue().stream().mapToInt(p -> p.getValue()).average().getAsDouble()))
                    .sorted()
                    .collect(Collectors.toList());
            end=new Date();
         recommendations. forEach(pr -> System.out.println (pr.getTitle()+": "+pr.getValue()));

            System.out.println("Execution Time: "+(end.getTime()- start.getTime()));

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

我们处理前一步得到的映射。对于每个产品，我们处理其评论列表，生成一个`ProductRecommendation`对象。该对象的值是使用`mapToInt()`方法将`ProductReview`对象的流转换为整数流，并使用`average()`方法获取字符串中所有数字的平均值来计算每个评论的平均值。

最后，在推荐`ConcurrentLinkedDeque`类中，我们有一个`ProductRecommendation`对象列表。我们使用另一个带有`sorted()`方法的流对该列表进行排序。我们使用该流将最终列表写入控制台。

## ConcurrentLoaderAccumulator 类

为了实现此示例，我们使用了`ConcurrentLoaderAccumulator`类作为`collect()`方法中的累加器函数，将包含所有要处理文件路径的`Path`对象流转换为`Product`对象的`ConcurrentLinkedDeque`类。以下是该类的源代码：

```java
public class ConcurrentLoaderAccumulator implements
        BiConsumer<ConcurrentLinkedDeque<Product>, Path> {

    @Override
    public void accept(ConcurrentLinkedDeque<Product> list, Path path) {

        Product product=ProductLoader.load(path);
        list.add(product);

    }
}
```

它实现了`BiConsumer`接口。`accept()`方法使用`ProducLoader`类（在本章前面已经解释过）从文件中加载产品信息，并将生成的`Product`对象添加到作为参数接收的`ConcurrentLinkedDeque`类中。

## 串行版本

与本书中的其他示例一样，我们实现了此示例的串行版本，以检查并行流是否提高了应用程序的性能。要实现此串行版本，我们必须按照以下步骤进行：

+   将`ConcurrentLinkedDeque`数据结构替换为`List`或`ArrayList`数据结构

+   将`parallelStrem()`方法替换为`stream()`方法

+   将`gropingByConcurrent()`方法替换为`groupingBy()`方法

您可以在本书的源代码中看到此示例的串行版本。

## 比较两个版本

为了比较我们的推荐系统的串行和并行版本，我们已经为三个用户获取了推荐的产品：

+   `A2JOYUS36FLG4Z`

+   `A2JW67OY8U6HHK`

+   `A2VE83MZF98ITY`

对于这三个用户，我们使用 JMH 框架（[`openjdk.java.net/projects/code-tools/jmh/`](http://openjdk.java.net/projects/code-tools/jmh/)）执行了两个版本，该框架允许您在 Java 中实现微基准测试。使用基准测试框架比简单地使用`currentTimeMillis()`或`nanoTime()`等方法来测量时间更好。我们在一个四核处理器的计算机上执行了 10 次，并计算了这 10 次的中位执行时间。以下是以毫秒为单位的结果：

|   | A2JOYUS36FLG4Z | A2JW67OY8U6HHK | A2VE83MZF98ITY |
| --- | --- | --- | --- |
| **串行** | 4848.672 | 4830.051 | 4817.216 |
| **并行** | 2454.003 | 2458.003 | 2527.194 |

我们可以得出以下结论：

+   所得结果对于这三个用户来说非常相似

+   并行流的执行时间始终优于顺序流的执行时间

如果我们比较并行和串行版本，例如使用加速比的第二个用户，我们得到以下结果：

![比较两个版本](img/00024.jpeg)

# 第三个例子 - 社交网络中的共同联系人

社交网络正在改变我们的社会以及人们之间的关系方式。Facebook、Linkedin、Twitter 或 Instagram 拥有数百万用户，他们使用这些网络与朋友分享生活时刻，建立新的职业联系，推广自己的专业品牌，结识新朋友，或者了解世界上的最新趋势。

我们可以将社交网络视为一个图，其中用户是图的节点，用户之间的关系是图的弧。与图一样，有些社交网络（如 Facebook）中用户之间的关系是无向的或双向的。如果用户*A*与用户*B*连接，用户*B*也与*A*连接。相反，有些社交网络（如 Twitter）中用户之间的关系是有向的。在这种情况下，我们说用户*A*关注用户*B*，但反之则不一定成立。

在这一部分，我们将实现一个算法来计算社交网络中每对用户的共同联系人，这些用户之间存在双向关系。我们将实现[`stevekrenzel.com/finding-friends-with-mapreduce`](http://stevekrenzel.com/finding-friends-with-mapreduce)中描述的算法。该算法的主要步骤如下。

我们的数据源将是一个文件，其中我们存储了每个用户及其联系人：

```java
A-B,C,D,
B-A,C,D,E,
C-A,B,D,E,
D-A,B,C,E,
E-B,C,D,
```

这意味着用户*A*的联系人是*B*、*C*和*D*。请注意，关系是双向的，因此如果*A*是*B*的联系人，*A*也将是*B*的联系人，并且这两种关系都必须在文件中表示。因此，我们有以下两个部分的元素：

+   用户标识符

+   该用户的联系人列表

接下来，我们为每个元素生成一个由三部分组成的元素集。这三部分是：

+   用户标识符

+   朋友的用户标识符

+   该用户的联系人列表

因此，对于用户*A*，我们将生成以下元素：

```java
A-B-B,C,D
A-C-B,C,D
A-D-B,C,D
```

我们对所有元素采取相同的处理过程。我们将按字母顺序存储两个用户标识符。因此，对于用户*B*，我们生成以下元素：

```java
A-B-A,C,D,E
B-C-A,C,D,E
B-D-A,C,D,E
B-E-A,C,D,E
```

一旦我们生成了所有新元素，我们就将它们分组为两个用户标识符。例如，对于元组*A*-*B*，我们将生成以下组：

```java
A-B-(B,C,D),(A,C,D,E)
```

最后，我们计算两个列表之间的交集。结果列表是两个用户之间的共同联系人。例如，用户*A*和*B*共同拥有联系人*C*和*D*。

为了测试我们的算法，我们使用了两个数据集：

+   之前呈现的测试样本。

+   社交圈：您可以从[`snap.stanford.edu/data/egonets-Facebook.html`](https://snap.stanford.edu/data/egonets-Facebook.html)下载的 Facebook 数据集包含来自 Facebook 的 4,039 个用户的联系信息。我们已将原始数据转换为我们示例使用的数据格式。

## 基类

与书中其他示例一样，我们实现了此示例的串行和并发版本，以验证并行流改进了我们应用程序的性能。两个版本共享一些类。

### 人员类

`Person`类存储了社交网络中每个人的信息，包括以下内容：

+   用户 ID，存储在 ID 属性中

+   该用户的联系人列表，存储为`String`对象列表，存储在 contacts 属性中

该类声明了属性和相应的`getXXX()`和`setXXX()`方法。我们还需要一个构造函数来创建列表，以及一个名为`addContact()`的方法，用于将单个联系人添加到联系人列表中。该类的源代码非常简单，因此不会在此处包含。

### PersonPair 类

`PersonPair`类扩展了`Person`类，添加了存储第二个用户标识符的属性。我们将此属性称为`otherId`。该类声明了属性并实现了相应的`getXXX()`和`setXXX()`方法。我们需要一个额外的方法，名为`getFullId()`，它返回一个由逗号分隔的两个用户标识符的字符串。该类的源代码非常简单，因此不会在此处包含。

### 数据加载器类

`DataLoader`类加载包含用户及其联系人信息的文件，并将其转换为`Person`对象列表。它只实现了一个名为`load()`的静态方法，该方法接收文件路径作为`String`对象参数，并返回`Person`对象列表。

如前所述，文件的格式如下：

```java
User-C1,C2,C3...CN
```

在这里，`User`是用户的标识符，`C1、C2、C3….CN`是该用户的联系人的标识符。

该类的源代码非常简单，因此不会在此处包含。

## 并发版本

首先，让我们分析此算法的并发版本。

### 通用人员映射器类

`CommonPersonMapper`类是一个辅助类，稍后将使用它。它将从`Person`对象生成所有可能的`PersonPair`对象。该类实现了使用`Person`和`List<PersonPair>`类参数化的`Function`接口。

它实现了`Function`接口中定义的`apply()`方法。首先，我们初始化要返回的`List<PersonPair>`对象，并获取并对该人的联系人列表进行排序：

```java
public class CommonPersonMapper implements Function<Person, List<PersonPair>> {

    @Override
    public List<PersonPair> apply(Person person) {

        List<PersonPair> ret=new ArrayList<>();

        List<String> contacts=person.getContacts();
        Collections.sort(contacts);
```

然后，我们处理整个联系人列表，为每个联系人创建一个`PersonPair`对象。如前所述，我们将两个联系人按字母顺序排序。较小的联系人存储在 ID 字段中，另一个存储在`otherId`字段中：

```java
        for (String contact : contacts) {
            PersonPair personExt=new PersonPair();
            if (person.getId().compareTo(contact) < 0) {
                personExt.setId(person.getId());
                personExt.setOtherId(contact);
            } else {
                personExt.setId(contact);
                personExt.setOtherId(person.getId());
            }
```

最后，我们将联系人列表添加到新对象中，然后将对象添加到结果列表中。处理完所有联系人后，我们返回结果列表：

```java
            personExt.setContacts(contacts);
            ret.add(personExt);
        }
        return ret;
    }
}
```

### ConcurrentSocialNetwork 类

`ConcurrentSocialNetwork`是这个示例的主要类。它只实现了一个名为`bidirectionalCommonContacts()`的静态方法。该方法接收社交网络中的人员列表及其联系人，并返回一个`PersonPair`对象列表，其中包含每对联系人之间的共同联系人。

在内部，我们使用两个不同的流来实现我们的算法。我们使用第一个流将`Person`对象的输入列表转换为映射。该映射的键将是每对用户的两个标识符，值将是包含两个用户联系人的`PersonPair`对象列表。因此，这些列表始终有两个元素。我们有以下代码：

```java
public class ConcurrentSocialNetwork {

    public static List<PersonPair> bidirectionalCommonContacts(
            List<Person> people) {

        Map<String, List<PersonPair>> group = people.parallelStream()
                .map(new CommonPersonMapper())
                .flatMap(Collection::stream)
                .collect(Collectors.groupingByConcurrent (PersonPair::getFullId));
```

该流具有以下组件：

+   我们使用输入列表的`parallelStream()`方法创建流。

+   然后，我们使用`map()`方法和前面解释的`CommonPersonMapper`类来将每个`Person`对象转换为包含该对象所有可能性的`PersonPair`对象列表。

+   此时，我们有一个`List<PersonPair>`对象的流。我们使用`flatMap()`方法将该流转换为`PersonPair`对象的流。

+   最后，我们使用`collect()`方法使用`groupingByConcurrent()`方法返回的收集器生成映射，使用`getFullId()`方法返回的值作为映射的键。

然后，我们使用`Collectors`类的`of()`方法创建一个新的收集器。该收集器将接收一个字符串集合作为输入，使用`AtomicReference<Collection<String>>`作为中间数据结构，并返回一个字符串集合作为返回类型。

```java
        Collector<Collection<String>, AtomicReference<Collection<String>>, Collection<String>> intersecting = Collector.of(
                () -> new AtomicReference<>(null), (acc, list) -> {
                  acc.updateAndGet(set -> set == null ? new ConcurrentLinkedQueue<>(list) : set).retainAll(list);
                }, (acc1, acc2) -> {
                  if (acc1.get() == null)
                    return acc2;
                  if (acc2.get() == null)
                    return acc1;
                  acc1.get().retainAll(acc2.get());
                  return acc1;
                }, (acc) -> acc.get() == null ? Collections.emptySet() : acc.get(), Collector.Characteristics.CONCURRENT, Collector.Characteristics.UNORDERED);
```

`of()`方法的第一个参数是 supplier 函数。当我们需要创建数据的中间结构时，总是调用此 supplier。在串行流中，此方法只调用一次，但在并发流中，此方法将根据线程数调用一次。

```java
() -> new AtomicReference<>(null),
```

在我们的例子中，我们只需创建一个新的`AtomicReference`来存储`Collection<String>`对象。

`of()`方法的第二个参数是累加器函数。此函数接收中间数据结构和输入值作为参数：

```java
(acc, list) -> {
      acc.updateAndGet(set -> set == null ? new ConcurrentLinkedQueue<>(list) : set).retainAll(list);
                },
```

在我们的例子中，`acc`参数是`AtomicReference`，`list`参数是`ConcurrentLinkedDeque`。我们使用`AtomicReference`的`updateAndGet()`方法。此方法更新当前值并返回新值。如果`AtomicReference`为 null，则使用列表的元素创建一个新的`ConcurrentLinkedDeque`。如果`AtomicReference`不为 null，则它将存储一个`ConcurrentLinkedDeque`。我们使用`retainAll()`方法添加列表的所有元素。

`of()`方法的第三个参数是 combiner 函数。此函数仅在并行流中调用，并接收两个中间数据结构作为参数，以生成一个中间数据结构。

```java
   (acc1, acc2) -> {
      if (acc1.get() == null)
        return acc2;
       if (acc2.get() == null)
        return acc1;
      acc1.get().retainAll(acc2.get());
      return acc1;
    },
```

在我们的例子中，如果其中一个参数为 null，则返回另一个。否则，我们使用`acc1`参数中的`retainAll()`方法并返回结果。

`of()`方法的第四个参数是 finisher 函数。该函数将最终的中间数据结构转换为我们想要返回的数据结构。在我们的例子中，中间和最终的数据结构是相同的，因此不需要转换。

```java
(acc) -> acc.get() == null ? Collections.emptySet() : acc.get(),
```

最后，我们使用最后一个参数来指示收集器是并发的，也就是说，累加器函数可以从多个线程同时调用相同的结果容器，并且是无序的，也就是说，此操作不会保留元素的原始顺序。

现在我们已经定义了收集器，我们必须将第一个流生成的映射转换为具有每对用户的共同联系人的`PersonPair`对象列表。我们使用以下代码：

```java
        List<PersonPair> peopleCommonContacts = group.entrySet()
                  .parallelStream()
                  .map((entry) -> {
                    Collection<String> commonContacts =  
                      entry.getValue()
                        .parallelStream()
                        .map(p -> p.getContacts())
                        .collect(intersecting);
                    PersonPair person = new PersonPair();
                    person.setId(entry.getKey().split(",")[0]);
                    person.setOtherId(entry.getKey().split (",")[1]);
                    person.setContacts(new ArrayList<String> (commonContacts));
                    return person;
                  }).collect(Collectors.toList());

        return peopleCommonContacts;
    }
}
```

我们使用`entySet()`方法处理映射的所有元素。我们创建一个`parallelStream()`方法来处理所有`Entry`对象，然后使用`map()`方法将每个`PersonPair`对象列表转换为具有共同联系人的唯一`PersonPair`对象。

对于每个条目，键是由用户对的标识符连接而成的，作为分隔符，值是两个`PersonPair`对象的列表。第一个包含一个用户的联系人，另一个包含另一个用户的联系人。

我们为该列表创建一个流，以生成具有以下元素的两个用户的共同联系人：

+   我们使用列表的`parallelStream()`方法创建流

+   我们使用`map()`方法来替换其中存储的联系人列表的每个`PersonPair()`对象

+   最后，我们使用我们的收集器生成带有共同联系人的`ConcurrentLinkedDeque`

最后，我们创建一个新的`PersonPair`对象，其中包含两个用户的标识符和共同联系人列表。我们将该对象添加到结果列表中。当映射的所有元素都被处理时，我们可以返回结果列表。

### ConcurrentMain 类

`ConcurrentMain`类实现了`main()`方法来测试我们的算法。正如我们之前提到的，我们已经使用以下两个数据集进行了测试：

+   一个非常简单的数据集，用于测试算法的正确性

+   基于 Facebook 真实数据的数据集

这是这个类的源代码：

```java
public class ConcurrentMain {

    public static void main(String[] args) {

        Date start, end;
        System.out.println("Concurrent Main Bidirectional - Test");
        List<Person> people=DataLoader.load("data","test.txt");
        start=new Date();
        List<PersonPair> peopleCommonContacts= ConcurrentSocialNetwork.bidirectionalCommonContacts (people);
        end=new Date();
        peopleCommonContacts.forEach(p -> System.out.println (p.getFullId()+": "+getContacts(p.getContacts())));
        System.out.println("Execution Time: "+(end.getTime()- start.getTime()));

        System.out.println("Concurrent Main Bidirectional - Facebook");
        people=DataLoader.load("data","facebook_contacts.txt");
        start=new Date();
        peopleCommonContacts= ConcurrentSocialNetwork.bidirectionalCommonContacts (people);
        end=new Date();
        peopleCommonContacts.forEach(p -> System.out.println (p.getFullId()+": "+getContacts(p.getContacts())));
        System.out.println("Execution Time: "+(end.getTime()- start.getTime()));

    }

    private static String formatContacts(List<String> contacts) {
        StringBuffer buffer=new StringBuffer();
        for (String contact: contacts) {
            buffer.append(contact+",");
        }
        return buffer.toString();
    }
}
```

## 串行版本

与本书中的其他示例一样，我们实现了这个示例的串行版本。这个版本与并发版本相同，做出以下更改：

+   用`stream()`方法替换`parallelStream()`方法

+   用`ArrayList`数据结构替换`ConcurrentLinkedDeque`数据结构

+   用`groupingBy()`方法替换`groupingByConcurrent()`方法

+   不要在`of()`方法中使用最终参数

### 比较两个版本

我们使用 JMH 框架（[`openjdk.java.net/projects/code-tools/jmh/`](http://openjdk.java.net/projects/code-tools/jmh/)）执行了两个版本和两个数据集。该框架允许您在 Java 中实现微基准测试。使用基准测试框架比简单使用`currentTimeMillis()`或`nanoTime()`等方法测量时间更好。我们在具有四核处理器的计算机上执行了 10 次，并计算了这 10 次的中等执行时间。以下是以毫秒为单位的结果：

|   | **示例** | **Facebook** |
| --- | --- | --- |
| **串行** | 0.861 | 7002.485 |
| **并发** | 1.352 | 5303.990 |

我们可以得出以下结论：

+   对于示例数据集，串行版本获得了更好的执行时间。这个结果的原因是示例数据集的元素很少。

+   对于 Facebook 数据集，并发版本获得了更好的执行时间。

如果我们比较 Facebook 数据集的并发和串行版本，我们会得到以下结果：

![比较两个版本](img/00025.jpeg)

# 摘要

在本章中，我们使用`Stream`框架提供的不同版本的`collect()`方法来转换和分组`Stream`的元素。这和第七章，“使用并行流处理大型数据集 - 映射和归约模型”，教你如何使用整个流 API。

基本上，`collect()` 方法需要一个收集器来处理流的数据，并生成由流形成的一组聚合操作返回的数据结构。收集器与三种不同的数据结构一起工作——输入元素的类，用于处理输入元素的中间数据结构，以及返回的最终数据结构。

我们使用了不同版本的`collect()`方法来实现一个搜索工具，该工具必须在没有倒排索引的文件集中查找查询，一个推荐系统，以及一个工具来计算社交网络中两个用户之间的共同联系人。

在下一章中，我们将深入研究 Java 并发 API 提供的并发数据结构和同步机制。
