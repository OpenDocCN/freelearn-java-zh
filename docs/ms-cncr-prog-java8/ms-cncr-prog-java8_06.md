# 第五章：分阶段运行任务-Phaser 类

并发 API 中最重要的元素是为程序员提供的同步机制。**同步**是协调两个或多个任务以获得期望的结果。当必须按预定义顺序执行两个或多个任务时，或者当只有一个线程可以同时执行代码片段或修改一块内存块时，可以同步两个或多个任务的执行，或者同步对共享资源的访问。Java 8 并发 API 提供了许多同步机制，从基本的`synchronized`关键字或`Lock`接口及其实现来保护关键部分，到更高级的`CyclicBarrier`或`CountDownLatch`类，允许您同步不同任务的执行顺序。在 Java 7 中，并发 API 引入了`Phaser`类。该类提供了一个强大的机制（**phaser**）来执行分阶段的任务。任务可以要求 Phaser 类等待直到所有其他参与者完成该阶段。在本章中，我们将涵盖以下主题：

+   `Phaser`类的介绍

+   第一个示例-关键词提取算法

+   第二个示例-遗传算法

# `Phaser`类的介绍

`Phaser`类是一种同步机制，旨在以并发方式控制可以分阶段执行的算法。如果您有一个具有明确定义步骤的过程，因此您必须在开始第一个步骤之前完成它，然后依此类推，您可以使用此类来制作过程的并发版本。`Phaser`类的主要特点包括：

+   Phaser 必须知道它需要控制的任务数量。Java 将此称为参与者的注册。参与者可以随时在 phaser 中注册。

+   任务必须在完成阶段时通知 phaser。Phaser 将使该任务休眠，直到所有参与者完成该阶段为止。

+   在内部，phaser 保存一个整数，用于存储该阶段已经进行的阶段变化次数。

+   参与者可以随时离开 phaser 的控制。Java 将此称为参与者的注销。

+   当 phaser 进行阶段变化时，您可以执行自定义代码。

+   您可以控制 phaser 的终止。如果 phaser 被终止，将不会接受新的参与者，并且任务之间也不会进行同步。

+   您可以使用一些方法来了解 phaser 的状态和参与者数量。

## 参与者的注册和注销

正如我们之前提到的，phaser 必须知道它需要控制的任务数量。它必须知道有多少不同的线程正在执行分阶段算法，以正确地控制同时的阶段变化。

Java 将此过程称为参与者的注册。通常情况下，参与者在执行开始时注册，但是参与者可以随时注册。

您可以使用不同的方法注册参与者：

+   当您创建`Phaser`对象时：`Phaser`类提供了四种不同的构造函数。其中两种是常用的：

+   `Phaser()`:此构造函数创建一个没有参与者的 phaser

+   `Phaser(int parties)`:此构造函数创建一个具有给定参与者数量的 phaser

+   明确地，使用其中一种方法：

+   `bulkRegister(int parties)`:同时注册给定数量的新参与者

+   `register()`:注册一个新的参与者

当由 phaser 控制的任务之一完成其执行时，它必须从 phaser 中注销。如果不这样做，phaser 将在下一个阶段变化中无休止地等待它。要注销参与者，可以使用`arriveAndDeregister()`方法。您可以使用此方法指示 phaser，该任务已完成当前阶段，并且不会参与下一个阶段。

## 同步阶段变化

phaser 的主要目的是以并发方式清晰地划分为阶段的算法的实现。在所有任务完成前一个阶段之前，没有一个任务可以进入下一个阶段。`Phaser`类提供了三种方法来表示任务已完成阶段：`arrive()`、`arriveAndDeregister()`和`arriveAndAwaitAdvance()`。如果其中一个任务没有调用这些方法之一，其他参与者任务将被 phaser 无限期地阻塞。要进入下一个阶段，使用以下方法：

+   `arriveAndAwaitAdvance()`: 任务使用此方法向 phaser 指示，它已完成当前阶段，并希望继续下一个阶段。phaser 将阻塞任务，直到所有参与者任务调用了同步方法之一。

+   `awaitAdvance(int phase)`: 任务使用此方法向 phaser 指示，如果传递的数字和 phaser 的当前阶段相等，则希望等待当前阶段的完成。如果它们不相等，此方法将立即返回。

## 其他功能

当所有参与者任务完成一个阶段的执行并在继续下一个阶段之前，`Phaser`类执行`onAdvance()`方法。此方法接收以下两个参数：

+   `phase`：这是已完成的阶段编号。第一个阶段是编号零

+   `registeredParties`：这表示参与者任务的数量

如果您想在两个阶段之间执行一些代码，例如对数据进行排序或转换，可以实现自己的 phaser，扩展`Phaser`类并覆盖此方法。

phaser 可以处于两种状态：

+   **活动**：当创建 phaser 并注册新参与者并继续进行直到终止时，phaser 进入此状态。在此状态下，它接受新的参与者并按照之前的说明工作。

+   **终止**：当`onAdvance()`方法返回`true`值时，phaser 进入此状态。默认情况下，当所有参与者已注销时，它返回`true`值。

### 注意

当 phaser 处于终止状态时，新参与者的注册不起作用，并且同步方法会立即返回。

最后，`Phaser`类提供了一些方法来获取有关 phaser 状态和参与者的信息：

+   `getRegisteredParties()`: 此方法返回 phaser 中的参与者数量

+   `getPhase()`: 此方法返回当前阶段的编号

+   `getArrivedParties()`: 此方法返回已完成当前阶段的参与者数量

+   `getUnarrivedParties()`: 此方法返回尚未完成当前阶段的参与者数量

+   `isTerminated()`: 如果 phaser 处于终止状态，则此方法返回`true`值，否则返回`false`

# 第一个示例 - 关键字提取算法

在本节中，您将使用 phaser 来实现**关键字提取算法**。这类算法的主要目的是从文本文档或文档集合中提取单词，以更好地定义文档在集合中的文档。这些术语可用于总结文档、对其进行聚类或改进信息搜索过程。

从集合中提取文档关键字的最基本算法（但现在仍然常用）是基于**TF-IDF**度量，其中：

+   **TF**（代表**词项频率**）是单词在文档中出现的次数。

+   **DF**（代表**文档频率**）是包含单词的文档数量。**IDF**（代表**逆文档频率**）度量了单词提供的信息，以区分文档与其他文档。如果一个词很常见，它的 IDF 将很低，但如果这个词只出现在少数文档中，它的 IDF 将很高。

单词*t*在文档*d*中的 TF-IDF 可以使用以下公式计算：

![第一个示例 - 关键词提取算法](img/00014.jpeg)

上述公式中使用的属性可以解释如下：

+   *F**[t,d]*是单词*t*在文档*d*中出现的次数

+   *N*是集合中文档的数量

+   *n**[t]*是包含单词*t*的文档数量

要获取文档的关键词，可以选择 TF-IDF 值较高的单词。

您将要实现的算法将执行以下阶段，计算文档集合中的最佳关键词：

+   **第一阶段**：解析所有文档并提取所有单词的 DF。请注意，只有在解析所有文档后，您才会获得确切的值。

+   **第二阶段**：计算所有文档中所有单词的 TF-IDF。选择每个文档的 10 个关键词（TF-IDF 值最高的 10 个单词）。

+   **第三阶段**：获取最佳关键词列表。我们认为那些是出现在更多文档中的单词。

为了测试算法，我们将使用维基百科关于电影信息的页面作为文档集合。我们在第四章中使用了相同的集合，*从任务中获取数据 - Callable 和 Future 接口*。该集合由 100,673 个文档组成。我们已经将每个维基百科页面转换为文本文件。您可以下载包含有关该书的所有信息的文档集合。

您将要实现算法的两个不同版本：基本的串行版本和使用`Phaser`类的并发版本。之后，我们将比较两个版本的执行时间，以验证并发性能更好。

## 常见类

算法的两个版本共享一些通用功能，用于解析文档并存储有关文档、关键词和单词的信息。这些通用类包括：

+   存储包含文档名称和构成文档的单词的`Document`类

+   存储单词字符串和该单词的度量（TF，DF 和 TF-IDF）的`Word`类

+   存储单词字符串和该单词作为关键词出现在的文档数量的`Keyword`类

+   提取文档中的单词的`DocumentParser`类

让我们更详细地看看这些类。

### 单词类

`Word`类存储有关单词的信息。这些信息包括整个单词以及影响它的度量，即它在文档中的 TF，它的全局 DF 和结果 TF-IDF。

这个类实现了`Comparable`接口，因为我们将对单词数组进行排序，以获取 TF-IDF 值较高的单词。参考以下代码：

```java
public class Word implements Comparable<Word> {
```

然后，我们声明了该类的属性并实现了 getter 和 setter（这些未包含在内）：

```java
    private String word;
    private int tf;
    private int df;
    private double tfIdf;
```

我们已经实现了其他感兴趣的方法如下：

+   该类的构造函数，初始化单词（使用参数接收的单词）和`df`属性（值为`1`）。

+   `addTf()`方法，增加`tf`属性。

+   `merge()`方法接收一个`Word`对象并合并来自两个不同文档的相同单词。它将两个对象的`tf`和`df`属性相加。

然后，我们实现了`setDf()`方法的特殊版本。它接收`df`属性的值和集合中文档的总数，并计算`tfIdf`属性：

```java
    public void setDf(int df, int N) {
        this.df = df;
        tfIdf = tf * Math.log(Double.valueOf(N) / df);
    }
```

最后，我们实现`compareTo()`方法。我们希望单词按`tfIdf`属性从高到低排序：

```java
    @Override
    public int compareTo(Word o) {
        return Double.compare(o.getTfIdf(), this.getTfIdf());
    }
}
```

### 关键词类

`Keyword`类存储有关关键词的信息。这些信息包括整个单词以及该单词作为关键词出现在的文档数量。

与`Word`类一样，它实现了`Comparable`接口，因为我们将对关键字数组进行排序以获得最佳关键字：

```java
public class Keyword implements Comparable<Keyword> {
```

然后，我们声明了类的属性并实现了方法来建立和返回其值（这些方法在此处未包括）：

```java
    private String word;
    private int df;
```

最后，我们实现了`compareTo()`方法。我们希望关键词按文档数量从高到低排序：

```java
    @Override
    public int compareTo(Keyword o) {

        return Integer.compare(o.getDf(), this.getDf());
    }
}
```

### Document 类

`Document`类存储有关集合中文档的信息（请记住我们的集合有 100,673 个文档），包括文件名和构成文档的单词集。该单词集通常称为文档的词汇，它以整个单词作为字符串作为键，并以`Word`对象作为值实现为`HashMap`：

```java
public class Document {
    private String fileName;
    private HashMap <String, Word> voc;
```

我们实现了一个构造函数，创建了`HashMap`和方法来获取和设置文件名以及返回文档的词汇（这些方法未包括）。我们还实现了一个方法来在词汇中添加单词。如果单词不存在，则将其添加到其中。如果单词存在于词汇中，则增加单词的`tf`属性。我们使用了`voc`对象的`computeIfAbsent()`方法。该方法如果单词不存在，则将单词插入`HashMap`中，然后使用`addTf()`方法增加`tf`：

```java
    public void addWord(String string) {
        voc.computeIfAbsent(string, k -> new Word(k)).addTf();
    }
}
```

`HashMap`类不是同步的，但我们可以在并发应用程序中使用它，因为它不会在不同任务之间共享。一个`Document`对象只会被一个任务生成，因此我们不会在并发版本中出现由`HashMap`类的使用导致的竞争条件。

### DocumentParser 类

`DocumentParser`类读取文本文件的内容并将其转换为`Document`对象。它将文本拆分为单词并将它们存储在`Document`对象中以生成类的词汇。该类有两个静态方法。第一个是`parse()`方法，它接收一个带有文件路径的字符串并返回一个`Document`对象。它打开文件并逐行读取，使用`parseLine()`方法将每行转换为一系列单词，并将它们存储到`Document`类中：

```java
public class DocumentParser {

    public static Document parse(String path) {
        Document ret = new Document();
        Path file = Paths.get(path);
        ret.setFileName(file.toString());

        try (BufferedReader reader = Files.newBufferedReader(file)) {
            for(String line : Files.readAllLines(file)) {
                parseLine(line, ret);
            }
        } catch (IOException x) {
            x.printStackTrace();
        }
        return ret;

    }
```

`parseLine()`方法接收要解析的行和`Document`对象以存储单词作为参数。

首先，使用`Normalizer`类删除行的重音符号，并将其转换为小写：

```java
    private static void parseLine(String line, Document ret) {

        // Clean string
        line = Normalizer.normalize(line, Normalizer.Form.NFKD);
        line = line.replaceAll("[^\\p{ASCII}]", "");
        line = line.toLowerCase();
```

然后，我们使用`StringTokenizer`类将行拆分为单词，并将这些单词添加到`Document`对象中：

```java
    private static void parseLine(String line, Document ret) {

        // Clean string
        line = Normalizer.normalize(line, Normalizer.Form.NFKD);
        line = line.replaceAll("[^\\p{ASCII}]", "");
        line = line.toLowerCase();

        // Tokenizer

        for(String w: line.split("\\W+")) {
              ret.addWord(w);
        }
    }

}
```

## 串行版本

我们在`SerialKeywordExtraction`类中实现了关键字算法的串行版本。它定义了您将执行以测试算法的`main()`方法。

第一步是声明以下必要的内部变量来执行算法：

+   两个`Date`对象，用于测量执行时间

+   一个字符串，用于存储包含文档集合的目录的名称

+   一个`File`对象数组，用于存储文档集合中的文件

+   一个`HashMap`，用于存储文档集合的全局词汇

+   一个`HashMap`，用于存储关键字

+   两个`int`值，用于测量执行的统计数据

以下包括这些变量的声明：

```java
public class SerialKeywordExtraction {

    public static void main(String[] args) {

        Date start, end;

        File source = new File("data");
        File[] files = source.listFiles();
        HashMap<String, Word> globalVoc = new HashMap<>();
        HashMap<String, Integer> globalKeywords = new HashMap<>();
        int totalCalls = 0;
        int numDocuments = 0;

        start = new Date();
```

然后，我们已经包含了算法的第一阶段。我们使用`DocumentParser`类的`parse()`方法解析所有文档。该方法返回一个包含该文档词汇的`Document`对象。我们使用`HashMap`类的`merge()`方法将文档词汇添加到全局词汇中。如果单词不存在，则将其插入`HashMap`中。如果单词存在，则合并两个单词对象，求和`Tf`和`Df`属性：

```java
        if(files == null) {
            System.err.println("Unable to read the 'data' folder");
            return;
        }
        for (File file : files) {

            if (file.getName().endsWith(".txt")) {
                Document doc = DocumentParser.parse (file.getAbsolutePath());
                for (Word word : doc.getVoc().values()) {
                    globalVoc.merge(word.getWord(), word, Word::merge);
                }
                numDocuments++;
            }
        }
        System.out.println("Corpus: " + numDocuments + " documents.");
```

在这个阶段之后，`globalVocHashMap`类包含了文档集合中所有单词及其全局 TF（单词在集合中出现的总次数）和 DF。

然后，我们包括了算法的第二阶段。我们将使用 TF-IDF 度量来计算每个文档的关键词，正如我们之前解释的那样。我们必须再次解析每个文档以生成其词汇表。我们必须这样做，因为我们无法将由 100,673 个文档组成的文档集合的词汇表存储在内存中。如果您使用的是较小的文档集合，可以尝试仅解析一次文档并将所有文档的词汇表存储在内存中，但在我们的情况下，这是不可能的。因此，我们再次解析所有文档，并且对于每个单词，我们使用存储在`globalVoc`中的值来更新`Df`属性。我们还构建了一个包含文档中所有单词的数组：

```java
        for (File file : files) {
            if (file.getName().endsWith(".txt")) {
                Document doc = DocumentParser.parse(file.getAbsolutePath());
                List<Word> keywords = new ArrayList<>( doc.getVoc().values());

                int index = 0;
                for (Word word : keywords) {
                      Word globalWord = globalVoc.get(word.getWord());
                      word.setDf(globalWord.getDf(), numDocuments);
                }
```

现在，我们有了关键词列表，其中包含文档中所有单词的 TF-IDF 计算结果。我们使用`Collections`类的`sort()`方法对列表进行排序，将 TF-IDF 值较高的单词排在第一位。然后我们获取该列表的前 10 个单词，并使用`addKeyword()`方法将它们存储在`globalKeywordsHashMap`中。

选择前 10 个单词没有特殊原因。您可以尝试其他选项，比如单词的百分比或 TF-IDF 值的最小值，并观察它们的行为：

```java
                Collections.sort(keywords);

                int counter = 0;

                for (Word word : keywords) {
                      addKeyword(globalKeywords, word.getWord());
                      totalCalls++;
                }
            }
        }
```

最后，我们包括了算法的第三阶段。我们将`globalKeywordsHashMap`转换为`Keyword`对象的列表，使用`Collections`类的`sort()`方法对该数组进行排序，获取 DF 值较高的关键词并将其写入控制台的前 100 个单词。

参考以下代码：

```java
        List<Keyword> orderedGlobalKeywords = new ArrayList<>();
        for (Entry<String, Integer> entry : globalKeywords.entrySet()) {
          Keyword keyword = new Keyword();
          keyword.setWord(entry.getKey());
          keyword.setDf(entry.getValue());
          orderedGlobalKeywords.add(keyword);
        }

        Collections.sort(orderedGlobalKeywords);

        if (orderedGlobalKeywords.size() > 100) {
            orderedGlobalKeywords = orderedGlobalKeywords.subList(0, 100);
        }
        for (Keyword keyword : orderedGlobalKeywords) {
            System.out.println(keyword.getWord() + ": " + keyword.getDf());
        }
```

与第二阶段一样，选择前 100 个单词没有特殊原因。如果您愿意，可以尝试其他选项。

在主方法结束时，我们在控制台中写入执行时间和其他统计数据：

```java
        end = new Date();
        System.out.println("Execution Time: " + (end.getTime() - start.getTime()));
        System.out.println("Vocabulary Size: " + globalVoc.size());
        System.out.println("Keyword Size: " + globalKeywords.size());
        System.out.println("Number of Documents: " + numDocuments);
        System.out.println("Total calls: " + totalCalls);

    }
```

`SerialKeywordExtraction`类还包括`addKeyword()`方法，用于更新`globalKeywordsHashMap`类中关键词的信息。如果单词存在，该类会更新其 DF；如果单词不存在，则插入它。

```java
    private static void addKeyword(Map<String, Integer> globalKeywords, String word) {
        globalKeywords.merge(word, 1, Integer::sum);
    }

}
```

## 并发版本

为了实现这个示例的并发版本，我们使用了两个不同的类，如下所示：

+   `KeywordExtractionTasks`类实现了以并发方式计算关键词的任务。我们将以`Thread`对象的形式执行这些任务，因此这个类实现了`Runnable`接口。

+   `ConcurrentKeywordExtraction`类提供了`main()`方法来执行算法，并创建、启动和等待任务完成。

让我们详细看看这些类。

### 关键词提取任务类

正如我们之前提到的，这个类实现了计算最终关键词列表的任务。它实现了`Runnable`接口，因此我们可以将其作为`Thread`执行，并且在内部使用一些属性，其中大部分属性在所有任务之间是共享的：

+   **两个 ConcurrentHashMap 对象用于存储全局词汇表和全局关键词**：我们使用`ConcurrentHashMap`，因为这些对象将由所有任务更新，所以我们必须使用并发数据结构来避免竞争条件。

+   **两个 ConcurrentLinkedDeque 的 File 对象，用于存储构成文档集合的文件列表**：我们使用 `ConcurrentLinkedDeque` 类，因为所有任务都将同时提取（获取和删除）列表的元素，所以我们必须使用并发数据结构来避免竞争条件。如果我们使用普通的 `List`，同一个 `File` 可能会被不同的任务解析两次。我们有两个 `ConcurrentLinkedDeque`，因为我们必须两次解析文档集合。正如我们之前提到的，我们从数据结构中提取 `File` 对象来解析文档集合，因此，当我们解析完集合时，数据结构将为空。

+   **一个 Phaser 对象来控制任务的执行**：正如我们之前解释的那样，我们的关键词提取算法是在三个阶段中执行的。在所有任务完成前，没有一个任务会进入下一个阶段。我们使用 `Phaser` 对象来控制这一点。如果我们不控制这一点，我们将得到不一致的结果。

+   **最终步骤必须由一个线程执行**：我们将使用布尔值区分一个主任务和其他任务。这些主任务将执行最终阶段。

+   **集合中的文档总数**：我们需要这个值来计算 TF-IDF 度量。

我们已经包括了一个构造函数来初始化所有这些属性：

```java
public class KeywordExtractionTask implements Runnable {

    private ConcurrentHashMap<String, Word> globalVoc;
    private ConcurrentHashMap<String, Integer> globalKeywords;

    private ConcurrentLinkedDeque<File> concurrentFileListPhase1;
    private ConcurrentLinkedDeque<File> concurrentFileListPhase2;

    private Phaser phaser;

    private String name;
    private boolean main;

    private int parsedDocuments;
    private int numDocuments;

    public KeywordExtractionTask(
            ConcurrentLinkedDeque<File> concurrentFileListPhase1,
            ConcurrentLinkedDeque<File> concurrentFileListPhase2,
            Phaser phaser, ConcurrentHashMap<String, Word> globalVoc,
            ConcurrentHashMap<String, Integer> globalKeywords,
            int numDocuments, String name, boolean main) {
        this.concurrentFileListPhase1 = concurrentFileListPhase1;
        this.concurrentFileListPhase2 = concurrentFileListPhase2;
        this.globalVoc = globalVoc;
        this.globalKeywords = globalKeywords;
        this.phaser = phaser;
        this.main = main;
        this.name = name;
        this.numDocuments = numDocuments;
    }
```

`run()` 方法实现了算法的三个阶段。首先，我们调用 phaser 的 `arriveAndAwaitAdvance()` 方法等待其他任务的创建。所有任务将在同一时刻开始执行。然后，就像我们在算法的串行版本中解释的那样，我们解析所有文档，并使用所有单词和它们的全局 TF 和 DF 值构建 `globalVocConcurrentHashMap` 类。为了完成第一阶段，我们再次调用 `arriveAndAwaitAdvance()` 方法，等待其他任务在执行第二阶段之前的最终化：

```java
    @Override
    public void run() {
        File file;

        // Phase 1
        phaser.arriveAndAwaitAdvance();
        System.out.println(name + ": Phase 1");
        while ((file = concurrentFileListPhase1.poll()) != null) {
            Document doc = DocumentParser.parse(file.getAbsolutePath());
            for (Word word : doc.getVoc().values()) {
                globalVoc.merge(word.getWord(), word, Word::merge);
            }
            parsedDocuments++;
        }

        System.out.println(name + ": " + parsedDocuments + " parsed.");
        phaser.arriveAndAwaitAdvance();
```

如您所见，为了获取要处理的 `File` 对象，我们使用 `ConcurrentLinkedDeque` 类的 `poll()` 方法。这个方法*检索并删除*`Deque`的第一个元素，所以下一个任务将获得一个不同的文件进行解析，不会有文件被解析两次。

第二阶段计算 `globalKeywords` 结构，就像我们在算法的串行版本中解释的那样。首先，计算每个文档的最佳 10 个关键词，然后将它们插入 `ConcurrentHashMap` 类。代码与串行版本相同，只是将串行数据结构更改为并发数据结构：

```java
        // Phase 2
        System.out.println(name + ": Phase 2");
        while ((file = concurrentFileListPhase2.poll()) != null) {

            Document doc = DocumentParser.parse(file.getAbsolutePath());
            List<Word> keywords = new ArrayList<>(doc.getVoc().values());

            for (Word word : keywords) {
              Word globalWord = globalVoc.get(word.getWord());
              word.setDf(globalWord.getDf(), numDocuments);
            }
            Collections.sort(keywords);

            if(keywords.size() > 10) keywords = keywords.subList(0, 10);
            for (Word word : keywords) {
              addKeyword(globalKeywords, word.getWord());
            }
        }
        System.out.println(name + ": " + parsedDocuments + " parsed.");
```

最终阶段对于主任务和其他任务是不同的。主任务使用 `Phaser` 类的 `arriveAndAwaitAdvance()` 方法等待所有任务的第二阶段最终化，然后在控制台中写入整个集合中最佳的 100 个关键词。最后，它使用 `arriveAndDeregister()` 方法从 phaser 中注销。

其他任务使用 `arriveAndDeregister()` 方法标记第二阶段的最终化，从 phaser 中注销，并完成它们的执行。

当所有任务都完成了他们的工作，它们都从 phaser 中注销了自己。phaser 将没有任何 parties，并且进入终止状态。

```java
        if (main) {
            phaser.arriveAndAwaitAdvance();

            Iterator<Entry<String, Integer>> iterator = globalKeywords.entrySet().iterator();
            Keyword orderedGlobalKeywords[] = new Keyword[globalKeywords.size()];
            int index = 0;
            while (iterator.hasNext()) {
                Entry<String, AtomicInteger> entry = iterator.next();
                Keyword keyword = new Keyword();
                keyword.setWord(entry.getKey());
                keyword.setDf(entry.getValue().get());
                orderedGlobalKeywords[index] = keyword;
                index++;
            }

            System.out.println("Keyword Size: " + orderedGlobalKeywords.length);

            Arrays.parallelSort(orderedGlobalKeywords);
            int counter = 0;
            for (int i = 0; i < orderedGlobalKeywords.length; i++){

                Keyword keyword = orderedGlobalKeywords[i];
                System.out.println(keyword.getWord() + ": " + keyword.getDf());
                counter++;
                if (counter == 100) {
                    break;
                }
            }
        }
        phaser.arriveAndDeregister();

        System.out.println("Thread " + name + " has finished.");
    }
```

### ConcurrentKeywordExtraction 类

`ConcurrentKeywordExtraction` 类初始化了共享对象，创建了任务，执行它们，并等待它们的最终化。它实现了一个 `main()` 方法，可以接收一个可选参数。默认情况下，我们根据 `Runtime` 类的 `availableProcessors()` 方法确定任务的数量，该方法返回 **Java 虚拟机** (**JVM**) 可用的硬件线程数。如果我们收到一个参数，我们将其转换为整数，并将其用作可用处理器数量的乘数，以确定我们将创建的任务数量。

首先，我们初始化所有必要的数据结构和参数。为了填充两个`ConcurrentLinkedDeque`结构，我们使用`File`类的`listFiles()`方法来获取以`txt`后缀结尾的文件的`File`对象数组。

我们还使用不带参数的构造函数创建`Phaser`对象，因此所有任务必须显式地在屏障中注册自己。参考以下代码：

```java
public class ConcurrentKeywordExtraction {

    public static void main(String[] args) {

        Date start, end;

        ConcurrentHashMap<String, Word> globalVoc = new ConcurrentHashMap<>();
        ConcurrentHashMap<String, Integer> globalKeywords = new ConcurrentHashMap<>();

        start = new Date();
        File source = new File("data");

        File[] files = source.listFiles(f -> f.getName().endsWith(".txt"));
        if (files == null) {
            System.err.println("The 'data' folder not found!");
            return;
        }
        ConcurrentLinkedDeque<File> concurrentFileListPhase1 = new ConcurrentLinkedDeque<>(Arrays.asList(files));
        ConcurrentLinkedDeque<File> concurrentFileListPhase2 = new ConcurrentLinkedDeque<>(Arrays.asList(files));

        int numDocuments = files.length();
        int factor = 1;
        if (args.length > 0) {
            factor = Integer.valueOf(args[0]);
        }

        int numTasks = factor * Runtime.getRuntime().availableProcessors();
        Phaser phaser = new Phaser();

        Thread[] threads = new Thread[numTasks];
        KeywordExtractionTask[] tasks = new KeywordExtractionTask[numTasks];
```

然后，我们使用`true`作为主参数创建第一个任务，其余使用`false`作为主参数。在创建每个任务之后，我们使用`Phaser`类的`register()`方法来注册新的参与者到屏障中，如下所示：

```java
        for (int i = 0; i < numTasks; i++) {
            tasks[i] = new KeywordExtractionTask(concurrentFileListPhase1, concurrentFileListPhase2, phaser, globalVoc, globalKeywords, concurrentFileListPhase1.size(), "Task" + i, i==0);
            phaser.register();
            System.out.println(phaser.getRegisteredParties() + " tasks arrived to the Phaser.");
        }
```

然后，我们创建并启动运行任务的线程对象，并等待其完成：

```java
        for (int i = 0; i < numTasks; i++) {
            threads[i] = new Thread(tasks[i]);
            threads[i].start();
        }

        for (int i = 0; i < numTasks; i++) {
            try {
                threads[i].join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
```

最后，我们在控制台中写入有关执行的一些统计信息，包括执行时间：

```java
        System.out.println("Is Terminated: " + phaser.isTerminated());

        end = new Date();
        System.out.println("Execution Time: " + (end.getTime() - start.getTime()));
        System.out.println("Vocabulary Size: " + globalVoc.size());
        System.out.println("Number of Documents: " + numDocuments);

    }

}
```

## 比较两种解决方案

让我们比较我们的关键词提取 100,673 个文档的串行和并发版本。我们使用 JMH 框架（[`openjdk.java.net/projects/code-tools/jmh/`](http://openjdk.java.net/projects/code-tools/jmh/)）执行示例，该框架允许您在 Java 中实现微基准测试。使用基准测试框架比简单地使用`currentTimeMillis()`或`nanoTime()`等方法测量时间更好。我们在一个四核处理器的计算机上执行了 10 次，并计算了这 10 次的平均执行时间。

| 算法 | 因子 | 执行时间（秒） |
| --- | --- | --- |
| **串行** | N/A | 194.45 |
| **并发** | 1 | 64.52 |
| 2 | 65.55 |
| 3 | 68,23 |

我们可以得出以下结论：

+   算法的并发版本提高了串行版本的性能。

+   如果我们使用的任务数量超过了可用的硬件线程数量，我们不会得到更好的结果。只会稍微差一点，因为额外的同步工作必须由屏障执行。

我们比较并发和串行版本的算法，使用以下公式计算加速比：

![比较两种解决方案](img/00015.jpeg)

# 第二个例子 - 遗传算法

**遗传算法**是基于自然选择原理的自适应启发式搜索算法，用于生成**优化**和**搜索问题**的良好解决方案。它们处理问题的可能解决方案，称为**个体**或**表型**。每个个体都有一个由一组属性组成的表示，称为**染色体**。通常，个体由一系列位表示，但你可以选择最适合你问题的表示形式。

你还需要一个确定解决方案好坏的函数，称为**适应度函数**。遗传算法的主要目标是找到最大化或最小化该函数的解决方案。

遗传算法从一组可能的问题解决方案开始。这组可能的解决方案被称为种群。你可以随机生成这个初始集，或者使用某种启发式函数来获得更好的初始解决方案。

一旦你有了初始种群，你就开始一个迭代过程，包括三个阶段。该迭代过程的每一步被称为一代。每一代的阶段包括：

+   **选择**: 你选择种群中更好的个体。这些个体在适应度函数中具有更好的值。

+   **交叉**: 你交叉选择在上一步中选定的个体，以生成新的个体，形成新的一代。这个操作需要两个个体，并生成两个新的个体。这个操作的实现取决于你想要解决的问题以及你选择的个体的表示。

+   **突变**：您可以应用突变运算符来改变个体的值。通常，您会将该操作应用于非常少量的个体。虽然突变是找到良好解决方案的一个非常重要的操作，但我们不会在简化示例中应用它。

您重复这三个操作，直到满足您的完成标准。这些完成标准可以是：

+   固定数量的世代

+   预定义的适应度函数值

+   找到符合预定义标准的解决方案

+   时间限制

+   手动停止

通常，您会在种群之外存储您在整个过程中找到的最佳个体。这个个体将是算法提出的解决方案，通常情况下，它会是一个更好的解决方案，因为我们会生成新的世代。

在本节中，我们将实现一个遗传算法来解决著名的**旅行推销员问题**（**TSP**）。在这个问题中，您有一组城市和它们之间的距离，您希望找到一条最佳路线，穿过所有城市并最小化旅行的总距离。与其他示例一样，我们实现了一个串行版本和一个并发版本，使用了`Phaser`类。应用于 TSP 问题的遗传算法的主要特征是：

+   **个体**：个体表示城市的遍历顺序。

+   **交叉**：在交叉操作之后，您必须创建有效的解决方案。您必须只访问每个城市一次。

+   **适应度函数**：算法的主要目标是最小化穿过城市的总距离。

+   **完成标准**：我们将执行预定义数量的世代算法。

例如，您可以有一个包含四个城市的距离矩阵，如下表所示：

|   | 城市 1 | 城市 2 | 城市 3 | 城市 4 |
| --- | --- | --- | --- | --- |
| **城市 1** | 0 | 11 | 6 | 9 |
| **城市 2** | 7 | 0 | 8 | 2 |
| **城市 3** | 7 | 3 | 0 | 3 |
| **城市 4** | 10 | 9 | 4 | 0 |

这意味着城市 2 和城市 1 之间的距离是 7，但城市 1 和城市 2 之间的距离是 11。一个个体可以是（2,4,3,1），其适应度函数是 2 和 4 之间的距离、4 和 3 之间的距离、3 和 1 之间的距离以及 1 和 2 之间的距离的总和，即 2+4+7+11=24。

如果您想在个体（1,2,3,4）和（1,3,2,4）之间进行交叉，您不能生成个体（1,2,2,4），因为您访问了城市 2 两次。您可以生成个体（1,2,4,3）和（1,3,4,2）。

为了测试算法，我们使用了两个**城市距离数据集**的例子（[`people.sc.fsu.edu/~jburkardt/datasets/cities/cities.html`](http://people.sc.fsu.edu/~jburkardt/datasets/cities/cities.html)），分别为 15 个城市（`lau15_dist`）和 57 个城市（`kn57_dist`）。

## 常见类

两个版本都使用以下三个常见类：

+   `DataLoader`类从文件加载距离矩阵。我们不在这里包括该类的代码。它有一个静态方法，接收文件名并返回一个`int[][]`矩阵，其中存储了城市之间的距离。距离存储在 csv 文件中（我们对原始格式进行了一些小的转换），因此很容易进行转换。

+   `Individual`类存储种群中个体的信息（问题的可能解决方案）。为了表示每个个体，我们选择了一个整数值数组，它存储您访问不同城市的顺序。

+   `GeneticOperators`类实现了种群或个体的交叉、选择和评估。

让我们看看`Individual`和`GeneticOperators`类的详细信息。

### 个体类

这个类存储了我们 TSP 问题的每个可能解。我们称每个可能解为一个个体，它的表示是染色体。在我们的情况下，我们将每个可能解表示为一个整数数组。该数组包含推销员访问城市的顺序。这个类还有一个整数值来存储适应函数的结果。我们有以下代码：

```java
public class Individual implements Comparable<Individual> {
    private Integer[] chromosomes;
    private int value;
```

我们包括了两个构造函数。第一个接收你必须访问的城市数量，然后创建一个空数组。另一个接收一个`Individual`对象，并将其染色体复制如下：

```java
    public Individual(int size) {
        chromosomes=new Integer[size];
    }

    public Individual(Individual other) {
        chromosomes = other.getChromosomes().clone();

    }
```

我们还实现了`compareTo()`方法，使用适应函数的结果来比较两个个体：

```java
    @Override
    public int compareTo(Individual o) {
          return Integer.compare(this.getValue(), o.getValue());
    }
```

最后，我们已经包括了获取和设置属性值的方法。

### 遗传算法操作类

这是一个复杂的类，因为它实现了遗传算法的内部逻辑。它提供了初始化、选择、交叉和评估操作的方法，就像在本节开头介绍的那样。我们只描述了这个类提供的方法，而没有描述它们是如何实现的，以避免不必要的复杂性。你可以获取示例的源代码来分析这些方法的实现。

这个类提供的方法有：

+   `initialize(int numberOfIndividuals, int size)`: 这将创建一个新的种群。种群的个体数量将由`numberOfIndividuals`参数确定。染色体的数量（在我们的情况下是城市）将由大小参数确定。它返回一个`Individual`对象的数组。它使用初始化方法(`Integer[]`)来初始化每个个体。

+   `initialize(Integer[] chromosomes)`: 它以随机方式初始化个体的染色体。它生成有效的个体（你必须只访问每个城市一次）。

+   `selection(Individual[] population)`: 这个方法实现了选择操作，以获取种群中最好的个体。它以一个数组的形式返回这些个体。数组的大小将是种群大小的一半。你可以测试其他标准来确定选择的个体数量。我们选择适应函数最好的个体。

+   `crossover(Individual[] selected, int numberOfIndividuals, int size)`: 这个方法接收上一代选择的个体作为参数，并使用交叉操作生成下一代的种群。下一代的个体数量将由同名的参数确定。每个个体的染色体数量将由大小参数确定。它使用交叉方法(`Individual`, `Individual`, `Individual`, `Individual`)从两个选择的个体生成两个新的个体。

+   `crossover(Individual parent1, Individual parent2, Individual individual1, Individual individual2)`: 这个方法执行交叉操作，以获取`parent1`和`parent2`个体生成下一代的`individual1`和`individual2`个体。

+   `evaluate(Individual[] population, int [][] distanceMatrix)`: 这将使用接收的距离矩阵对种群中的所有个体应用适应函数。最后，它将种群从最佳到最差的解进行排序。它使用评估方法(`Individual`, `int[][]`)来评估每个个体。

+   `evaluate(Individual individual, int[][] distanceMatrix)`: 这将适用于一个个体的适应函数。

有了这个类和它的方法，你就有了实现解决 TSP 问题的遗传算法所需的一切。

## 串行版本

我们使用以下两个类实现了算法的串行版本：

+   实现算法的`SerialGeneticAlgorithm`类

+   `SerialMain`类执行算法，并测量执行时间

让我们详细分析这两个类。

### SerialGeneticAlgorithm 类

这个类实现了我们遗传算法的串行版本。在内部，它使用以下四个属性：

+   包含所有城市之间距离的距离矩阵

+   代的数量

+   种群中的个体数

+   每个个体中的染色体数

该类还有一个构造函数来初始化所有属性：

```java
    private int[][] distanceMatrix;

    private int numberOfGenerations;
    private int numberOfIndividuals;

    private int size;

    public SerialGeneticAlgorithm(int[][] distanceMatrix,
            int numberOfGenerations, int numberOfIndividuals) {
        this.distanceMatrix = distanceMatrix;
        this.numberOfGenerations = numberOfGenerations;
        this.numberOfIndividuals = numberOfIndividuals;
        size = distanceMatrix.length;
    }
```

该类的主要方法是`calculate()`方法。首先，使用`initialize()`方法创建初始种群。然后，评估初始种群，并将其最佳个体作为算法的第一个解决方案：

```java
    public Individual calculate() {
        Individual best;

        Individual[] population = GeneticOperators.initialize(
                numberOfIndividuals, size);
        GeneticOperators.evaluate(population, distanceMatrix);

        best = population[0];
```

然后，它执行一个由`numberOfGenerations`属性确定的循环。在每个周期中，它使用`selection()`方法获取选定的个体，使用`crossover()`方法计算下一代，评估这个新一代，并且如果新一代的最佳解决方案比到目前为止的最佳个体更好，我们就替换它。当循环结束时，我们将最佳个体作为算法提出的解决方案返回：

```java
        for (int i = 1; i <= numberOfGenerations; i++) {
            Individual[] selected = GeneticOperators.selection(population);
            population = GeneticOperators.crossover(selected, numberOfIndividuals, size);
            GeneticOperators.evaluate(population, distanceMatrix);
            if (population[0].getValue() < best.getValue()) {
                best = population[0];
            }

        }

        return best;
    }
```

### SerialMain 类

该类为本节中使用的两个数据集执行遗传算法——包含 15 个城市的`lau15`和包含 57 个城市的`kn57`。

`main()`方法必须接收两个参数。第一个是我们想要创建的代数，第二个参数是我们想要每一代中拥有的个体数：

```java
public class SerialMain {

    public static void main(String[] args) {

        Date start, end;

        int generations = Integer.valueOf(args[0]);
        int individuals = Integer.valueOf(args[1]);
```

对于每个示例，我们使用`DataLoader`类的`load()`方法加载距离矩阵，创建`SerialGeneticAlgorith`对象，执行`calculate()`方法并测量执行时间，并将执行时间和结果写入控制台：

```java
    for (String name : new String[] { "lau15_dist", "kn57_dist" }) {
        int[][] distanceMatrix = DataLoader.load(Paths.get("data", name + ".txt"));
        SerialGeneticAlgorithm serialGeneticAlgorithm = new SerialGeneticAlgorithm(distanceMatrix, generations, individuals);
            start = new Date();
            Individual result = serialGeneticAlgorithm.calculate();
            end = new Date();
        System.out.println ("=======================================");
        System.out.println("Example:"+name);
        System.out.println("Generations: " + generations);
        System.out.println("Population: " + individuals);
        System.out.println("Execution Time: " + (end.getTime() - start.getTime()));
        System.out.println("Best Individual: " + result);
        System.out.println("Total Distance: " + result.getValue());
        System.out.println ("=======================================");
    }
```

## 并发版本

我们已经实现了遗传算法的并发版本不同的类：

+   `SharedData`类存储所有任务之间共享的对象

+   `GeneticPhaser`类扩展了`Phaser`类，并覆盖了它的`onAdvance()`方法，以在所有任务完成一个阶段时执行代码

+   `ConcurrentGeneticTask`类实现了遗传算法阶段的任务

+   `ConcurrentGeneticAlgorithm`类将使用前面的类实现遗传算法的并发版本

+   `ConcurrentMain`类将在我们的两个数据集中测试遗传算法的并发版本

在内部，`ConcurrentGeneticTask`类将执行三个阶段。第一个阶段是选择阶段，只有一个任务执行。第二个阶段是交叉阶段，所有任务将使用选定的个体构建新一代，最后一个阶段是评估阶段，所有任务将评估新一代的个体。

让我们详细看看这些类中的每一个。

### SharedData 类

正如我们之前提到的，这个类包含了所有任务共享的对象。这包括以下内容：

+   种群数组，包含一代中的所有个体。

+   选定的数组与选定的个体。

+   一个名为`index`的原子整数。这是唯一的线程安全对象，用于知道任务必须生成或处理的个体的索引。

+   所有代中的最佳个体将作为算法的解决方案返回。

+   包含城市之间距离的距离矩阵。

所有这些对象将被所有线程共享，但我们只需要使用一个并发数据结构。这是唯一一个有效被所有任务共享的属性。其余的对象将只被读取（距离矩阵），或者每个任务将访问对象的不同部分（种群和选定的数组），因此我们不需要使用并发数据结构或同步机制来避免竞争条件：

```java
public class SharedData {

    private Individual[] population;
    private Individual selected[];
    private AtomicInteger index;
    private Individual best;
    private int[][] distanceMatrix;
}
```

该类还包括获取器和设置器，用于获取和建立这些属性的值。

### GeneticPhaser 类

我们需要在任务的阶段变化时执行代码，因此我们必须实现自己的阶段器并重写`onAdvance()`方法，该方法在所有参与方完成一个阶段之后执行，然后开始执行下一个阶段。`GeneticPhaser`类实现了这个阶段器。它存储`SharedData`对象以便与其一起工作，并将其作为构造函数的参数接收：

```java
public class GeneticPhaser extends Phaser {

    private SharedData data;

    public GeneticPhaser(int parties, SharedData data) {
        super(parties);
        this.data=data;
    }
```

`onAdvance()`方法将接收阶段器的阶段号和注册方的数量作为参数。阶段器内部将阶段号作为整数存储，随着每次阶段变化而递增。相反，我们的算法只有三个阶段，将被执行很多次。我们必须将阶段器的阶段号转换为遗传算法的阶段号，以了解任务是否将执行选择、交叉或评估阶段。为此，我们计算阶段器阶段号除以三的余数，如下所示：

```java
    protected boolean onAdvance(int phase, int registeredParties) {
        int realPhase=phase%3;
        if (registeredParties>0) {
            switch (realPhase) {
            case 0:
            case 1:
                data.getIndex().set(0);
                break;
            case 2:
                Arrays.sort(data.getPopulation());
                if (data.getPopulation()[0].getValue() < data.getBest().getValue()) {
                    data.setBest(data.getPopulation()[0]);
                }
                break;
            }
            return false;
        }
        return true;
    }
```

如果余数为零，则任务已经完成了选择阶段，并将执行交叉阶段。我们用值零初始化索引对象。

如果余数为一，则任务已经完成了交叉阶段，并将执行评估阶段。我们用值零初始化索引对象。

最后，如果余数为二，则任务已经完成了评估阶段，并将重新开始选择阶段。我们根据适应度函数对种群进行排序，并在必要时更新最佳个体。

请注意，这个方法只会由一个线程执行，与任务无关。它将在任务的线程中执行，这个任务是最后完成上一个阶段的（在`arriveAndAwaitAdvance()`调用中）。其余的任务将处于睡眠状态，等待阶段器。

### ConcurrentGeneticTask 类

这个类实现了协作执行遗传算法的任务。它们执行算法的三个阶段（选择、交叉和评估）。选择阶段将只由一个任务执行（我们称之为主任务），而其余的阶段将由所有任务执行。

在内部，它使用了四个属性：

+   一个`GeneticPhaser`对象，用于在每个阶段结束时同步任务

+   一个`SharedData`对象来访问共享数据

+   它必须计算的代数

+   指示是否为主任务的布尔标志

所有这些属性都在类的构造函数中初始化：

```java
public class ConcurrentGeneticTask implements Runnable {
    private GeneticPhaser phaser;
    private SharedData data;
    private int numberOfGenerations;
    private boolean main;

    public ConcurrentGeneticTask(GeneticPhaser phaser, int numberOfGenerations, boolean main) {
        this.phaser = phaser;
        this.numberOfGenerations = numberOfGenerations;
        this.main = main;
        this.data = phaser.getData();
    }
```

`run()`方法实现了遗传算法的逻辑。它有一个循环来生成指定的代数。正如我们之前提到的，只有主任务才会执行选择阶段。其余的任务将使用`arriveAndAwaitAdvance()`方法等待此阶段的完成。参考以下代码：

```java
    @Override
    public void run() {

        Random rm = new Random(System.nanoTime()); 
		for (int i = 0; i < numberOfGenerations; i++) {
            if (main) {
                data.setSelected(GeneticOperators.selection(data
                        .getPopulation()));
            }
            phaser.arriveAndAwaitAdvance();
```

第二阶段是交叉阶段。我们使用`SharedData`类中存储的`AtomicInteger`变量索引来获取每个任务将计算的种群数组中的下一个位置。正如我们之前提到的，交叉操作会生成两个新个体，因此每个任务首先在种群数组中保留两个位置。为此，我们使用`getAndAdd(2)`方法，它返回变量的实际值并将其值增加两个单位。它是一个原子变量，因此我们不需要使用任何同步机制。这是原子变量固有的。参考以下代码：

```java
            // Crossover
            int individualIndex;
            do {
                individualIndex = data.getIndex().getAndAdd(2);
                if (individualIndex < data.getPopulation().length) {
                    int secondIndividual = individualIndex++;

                    int p1Index = rm.nextInt (data.getSelected().length);
                    int p2Index;
                    do {
                        p2Index = rm.nextInt (data.getSelected().length);
                    } while (p1Index == p2Index);

                    Individual parent1 = data.getSelected() [p1Index];
                    Individual parent2 = data.getSelected() [p2Index];
                    Individual individual1 = data.getPopulation() [individualIndex];
                    Individual individual2 = data.getPopulation() [secondIndividual];

                    GeneticOperators.crossover(parent1, parent2, individual1, individual2);
                }
            } while (individualIndex < data.getPopulation().length);
            phaser.arriveAndAwaitAdvance();
```

当新种群的所有个体都生成时，任务使用`arriveAndAwaitAdvance()`方法来同步阶段的结束。

最后一个阶段是评估阶段。我们再次使用`AtomicInteger`索引。每个任务都会得到变量的实际值，该值代表种群中个体的位置，并使用`getAndIncrement()`方法递增其值。一旦所有个体都被评估，我们使用`arriveAndAwaitAdvance()`方法来同步这个阶段的结束。请记住，当所有任务都完成了这个阶段时，`GeneticPhaser`类将执行对种群数组的排序，并根据需要更新最佳个体变量，如下所示：

```java
            // Evaluation
            do {
                individualIndex = data.getIndex().getAndIncrement();
                if (individualIndex < data.getPopulation().length) {
                    GeneticOperators.evaluate(data.getPopulation() [individualIndex], data.getDistanceMatrix());
                }
            } while (individualIndex < data.getPopulation().length);
            phaser.arriveAndAwaitAdvance();

        }

        phaser.arriveAndDeregister();
    }
```

最后，当所有代数都被计算时，任务使用`arriveAndDeregister()`方法来指示其执行的结束，因此 phaser 将进入最终状态。

### ConcurrentGeneticAlgorithm 类

这个类是遗传算法的外部接口。在内部，它创建、启动并等待计算不同代数的任务的完成。它使用四个属性：代数的数量、每一代中个体的数量、每个个体的染色体数量和距离矩阵，如下所示：

```java
public class ConcurrentGeneticAlgorithm {

    private int numberOfGenerations;
    private int numberOfIndividuals;
    private int[][] distanceMatrix;
    private int size;

    public ConcurrentGeneticAlgorithm(int[][] distanceMatrix, int numberOfGenerations, int numberOfIndividuals) {
        this.distanceMatrix=distanceMatrix;
        this.numberOfGenerations=numberOfGenerations;
        this.numberOfIndividuals=numberOfIndividuals;
        size=distanceMatrix.length;
    }
```

`calculate()`方法执行遗传算法并返回最佳个体。首先，它使用`initialize()`方法创建初始种群，评估该种群，并创建和初始化一个带有所有必要数据的`SharedData`对象，如下所示：

```java
    public Individual calculate() {

        Individual[] population= GeneticOperators.initialize(numberOfIndividuals,size);
        GeneticOperators.evaluate(population,distanceMatrix);

        SharedData data=new SharedData();
        data.setPopulation(population);
        data.setDistanceMatrix(distanceMatrix);
        data.setBest(population[0]);
```

然后，它创建任务。我们使用计算机的可用硬件线程数，该数由`Runtime`类的`availableProcessors()`方法返回，作为我们将要创建的任务数。我们还创建了一个`GeneticPhaser`对象来同步这些任务的执行，如下所示：

```java
        int numTasks=Runtime.getRuntime().availableProcessors();
        GeneticPhaser phaser=new GeneticPhaser(numTasks,data);

        ConcurrentGeneticTask[] tasks=new ConcurrentGeneticTask[numTasks];
        Thread[] threads=new Thread[numTasks];

        tasks[0]=new ConcurrentGeneticTask(phaser, numberOfGenerations, true);
        for (int i=1; i< numTasks; i++) {
            tasks[i]=new ConcurrentGeneticTask(phaser, numberOfGenerations, false);
        }
```

然后，我们创建`Thread`对象来执行任务，启动它们，并等待它们的完成。最后，我们返回存储在`ShareData`对象中的最佳个体，如下所示：

```java
        for (int i=0; i<numTasks; i++) {
            threads[i]=new Thread(tasks[i]);
            threads[i].start();
        }

        for (int i=0; i<numTasks; i++) {
            try {
                threads[i].join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        return data.getBest();
    }
}
```

### ConcurrentMain 类

这个类执行了遗传算法，用于本节中使用的两个数据集——有 15 个城市的`lau15`和有 57 个城市的`kn57`。它的代码类似于`SerialMain`类，但是使用`ConcurrentGeneticAlgorithm`代替`SerialGeneticAlgorithm`。

## 比较两种解决方案

现在是时候测试两种解决方案，看看它们哪个性能更好。正如我们之前提到的，我们使用了城市距离数据集（[`people.sc.fsu.edu/~jburkardt/datasets/cities/cities.html`](http://people.sc.fsu.edu/~jburkardt/datasets/cities/cities.html)）中的两个数据集——有 15 个城市的`lau15`和有 57 个城市的`kn57`。我们还测试了不同规模的种群（100、1,000 和 10,000 个个体）和不同代数的数量（10、100 和 1,000）。为了测试算法，我们使用了 JMH 框架（[`openjdk.java.net/projects/code-tools/jmh/`](http://openjdk.java.net/projects/code-tools/jmh/)），它允许您在 Java 中实现微基准测试。使用基准测试框架比简单地使用`currentTimeMillis()`或`nanoTime()`等方法来测量时间更好。我们在一个四核处理器的计算机上执行了 10 次，并计算了这 10 次的中间执行时间。

### Lau15 数据集

第一个数据集的执行时间（毫秒）为：

|   | 人口 |
| --- | --- |
|   | **100** | **1,000** | **10,000** |
| 代数 | 串行 | 并发 | 串行 | 并发 | 串行 | 并发 |
| **10** | 8.42 | 13.309 | 30.783 | 36.395 | 182.213 | 99.728 |
| **100** | 25.848 | 29.292 | 135.562 | 69.257 | 1488.457 | 688.840 |
| **1,000** | 117.929 | 71.771 | 1134.983 | 420.145 | 11810.518 | 4102.72 |

### Kn57 数据集

第二个数据集的执行时间（毫秒）为：

|   | 人口 |
| --- | --- |
|   | **100** | **1,000** | **10,000** |
| **Generations** | **Serial** | **Concurrent** | **Serial** | **Concurrent** | **Serial** | **Concurrent** |
| **10** | 19.205 | 22.246 | 80.509 | 63.370 | 758.235 | 300.669 |
| **100** | 75.129 | 63.815 | 680.548 | 225.393 | 7406.392 | 2561.219 |
| **1,000** | 676.390 | 243.572 | 6796.780 | 2159.124 | 75315.885 | 26825.115 |

### 结论

算法的行为与两个数据集相似。您可以看到，当个体数量和世代数量较少时，算法的串行版本具有更好的执行时间，但是当个体数量或世代数量增加时，并发版本具有更好的吞吐量。例如，对于包含 1,000 代和 10,000 个体的`kn57`数据集，加速比为：

![结论](img/00016.jpeg)

# 摘要

在本章中，我们解释了 Java 并发 API 提供的最强大的同步机制之一：phaser。其主要目标是在执行分阶段算法的任务之间提供同步。在其余任务完成前，没有一个任务可以开始执行下一个阶段。

Phaser 必须知道有多少任务需要同步。您必须使用构造函数、`bulkRegister()`方法或`register()`方法在 phaser 中注册您的任务。

任务可以以不同的方式与 phaser 同步。最常见的是使用`arriveAndAwaitAdvance()`方法通知 phaser 已经完成一个阶段的执行，并希望继续下一个阶段。此方法将使线程休眠，直到其余任务完成当前阶段。但是还有其他方法可以用来同步您的任务。`arrive()`方法用于通知 phaser 您已经完成当前阶段，但不等待其余任务（使用此方法时要非常小心）。`arriveAndDeregister()`方法用于通知 phaser 您已经完成当前阶段，并且不希望继续在 phaser 中（通常是因为您已经完成了工作）。最后，`awaitAdvance()`方法可用于等待当前阶段的完成。

您可以使用`onAdvance()`方法控制相位变化，并在所有任务完成当前阶段并开始新阶段之前执行代码。此方法在两个阶段的执行之间调用，并接收相位编号和相位中参与者的数量作为参数。您可以扩展`Phaser`类并覆盖此方法以在两个阶段之间执行代码。

Phaser 可以处于两种状态：活动状态，当它正在同步任务时；终止状态，当它完成了其工作时。当所有参与者调用`arriveAndDeregister()`方法或`onAdvance()`方法返回`true`值时（默认情况下，它总是返回`false`），Phaser 将进入终止状态。当`Phaser`类处于终止状态时，它将不再接受新的参与者，并且同步方法将立即返回。

我们使用`Phaser`类来实现两种算法：关键词提取算法和遗传算法。在这两种情况下，我们都得到了与这些算法的串行版本相比的重要吞吐量增加。

在下一章中，您将学习如何使用另一个 Java 并发框架来解决特殊类型的问题。这就是 Fork/Join 框架，它已经被开发出来以并发方式执行那些可以使用分而治之算法解决的问题。它基于具有特殊工作窃取算法的执行程序，以最大化执行程序的性能。
