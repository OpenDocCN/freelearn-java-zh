# 第四章。从任务中获取数据 - Callable 和 Future 接口

在第二章，*管理大量线程 - 执行程序*，和第三章，*从执行程序中获得最大效益*，我们介绍了执行程序框架，以提高并发应用程序的性能，并向您展示了如何实现高级特性以使该框架适应您的需求。在这些章节中，执行程序执行的所有任务都基于`Runnable`接口及其不返回值的`run()`方法。然而，执行程序框架允许我们执行基于`Callable`和`Future`接口的返回结果的其他类型的任务。在本章中，我们将涵盖以下主题：

+   Callable 和 Future 接口介绍

+   第一个例子 - 用于单词的最佳匹配算法

+   第二个例子 - 构建文档集合的倒排索引

# 介绍 Callable 和 Future 接口

执行程序框架允许程序员在不创建和管理线程的情况下执行并发任务。您创建任务并将它们发送到执行程序。它会创建和管理必要的线程。

在执行程序中，您可以执行两种类型的任务：

+   **基于 Runnable 接口的任务**：这些任务实现了不返回任何结果的`run()`方法。

+   **基于 Callable 接口的任务**：这些任务实现了`call()`接口，返回一个对象作为结果。`call()`方法返回的具体类型由`Callable`接口的泛型类型参数指定。为了获取任务返回的结果，执行程序将为每个任务返回一个`Future`接口的实现。

在之前的章节中，您学习了如何创建执行程序，将基于`Runnable`接口的任务发送到其中，并个性化执行程序以适应您的需求。在本章中，您将学习如何处理基于`Callable`和`Future`接口的任务。

## Callable 接口

`Callable`接口与`Runnable`接口非常相似。该接口的主要特点是：

+   它是一个泛型接口。它有一个单一类型参数，对应于`call()`方法的返回类型。

+   它声明了`call()`方法。当执行程序运行任务时，该方法将被执行。它必须返回声明中指定类型的对象。

+   `call()`方法可以抛出任何已检查异常。您可以通过实现自己的执行程序并覆盖`afterExecute()`方法来处理异常。

## Future 接口

当您将一个`Callable`任务发送到执行程序时，它将返回一个`Future`接口的实现，允许您控制任务的执行和状态，并获取结果。该接口的主要特点是：

+   您可以使用`cancel()`方法取消任务的执行。该方法有一个`boolean`参数，用于指定是否要在任务运行时中断任务。

+   您可以通过`isCancelled()`方法检查任务是否已被取消，或者通过`isDone()`方法检查任务是否已完成。

+   您可以使用`get()`方法获取任务返回的值。此方法有两个变体。第一个没有参数，并返回任务执行完成后的返回值。如果任务尚未执行完成，它会挂起执行线程，直到任务完成。第二个变体接受两个参数：一段时间和该时间段的`TimeUnit`。与第一个的主要区别在于线程等待作为参数传递的时间段。如果时间段结束，任务尚未执行完成，该方法会抛出`TimeoutException`异常。

# 第一个示例-用于单词的最佳匹配算法

单词的**最佳匹配算法**的主要目标是找到与作为参数传递的字符串最相似的单词。要实现这些算法之一，您需要以下内容：

+   **单词列表**：在我们的案例中，我们使用了为填字游戏社区编制的**英国高级谜语词典**（**UKACD**）。它有 250,353 个单词和习语。可以从[`www.crosswordman.com/wordlist.html`](http://www.crosswordman.com/wordlist.html)免费下载。

+   **衡量两个单词相似性的度量标准**：我们使用了 Levenshtein 距离，用于衡量两个**字符**序列之间的差异。**Levenshtein 距离**是将第一个字符串转换为第二个字符串所需的最小插入、删除或替换次数。您可以在[`en.wikipedia.org/wiki/Levenshtein_distance`](https://en.wikipedia.org/wiki/Levenshtein_distance)中找到对此度量标准的简要描述。

在我们的示例中，您将实现两个操作：

+   第一个操作使用 Levenshtein 距离返回与**字符序列**最相似的单词列表。

+   第二个操作使用 Levenshtein 距离确定字符序列是否存在于我们的字典中。如果使用`equals()`方法会更快，但我们的版本对于本书的目标来说是一个更有趣的选择。

您将实现这些操作的串行和并发版本，以验证并发在这种情况下是否有帮助。

## 常见类

在此示例中实现的所有任务中，您将使用以下三个基本类：

+   `WordsLoader`类将单词列表加载到`String`对象列表中。

+   `LevenshteinDistance`类计算两个字符串之间的 Levenshtein 距离。

+   `BestMatchingData`类存储最佳匹配算法的结果。它存储单词列表以及这些单词与输入字符串的距离。

UKACD 在一个文件中，每行一个单词，因此`WordsLoader`类实现了`load()`静态方法，该方法接收包含单词列表的文件的路径，并返回一个包含 250,353 个单词的字符串对象列表。

`LevenshteinDistance`类实现了`calculate()`方法，该方法接收两个字符串对象作为参数，并返回这两个单词之间的距离的`int`值。这是这个分类的代码：

```java
public class LevenshteinDistance {

    public static int calculate (String string1, String string2) {
        int[][] distances=new int[string1.length()+1][string2.length()+1];

        for (int i=1; i<=string1.length();i++) {
            distances[i][0]=i;
        }

        for (int j=1; j<=string2.length(); j++) {
            distances[0][j]=j;
        }

        for(int i=1; i<=string1.length(); i++) {
            for (int j=1; j<=string2.length(); j++) {
                if (string1.charAt(i-1)==string2.charAt(j-1)) {
                    distances[i][j]=distances[i-1][j-1];
                } else {
                    distances[i][j]=minimum(distances[i-1][j], distances[i][j-1],distances[i-1][j-1])+1;
                }
            }
        }

        return distances[string1.length()][string2.length()];
    }

    private static int minimum(int i, int j, int k) {
        return Math.min(i,Math.min(j, k));
    }
}
```

`BestMatchingData`类只有两个属性：一个字符串对象列表，用于存储单词列表，以及一个名为距离的整数属性，用于存储这些单词与输入字符串的距离。

## 最佳匹配算法-串行版本

首先，我们将实现最佳匹配算法的串行版本。我们将使用此版本作为并发版本的起点，然后我们将比较两个版本的执行时间，以验证并发是否有助于提高性能。

我们已经在以下两个类中实现了最佳匹配算法的串行版本：

+   `BestMatchingSerialCalculation`类计算与输入字符串最相似的单词列表

+   `BestMatchingSerialMain`包括`main()`方法，执行算法，测量执行时间，并在控制台中显示结果

让我们分析一下这两个类的源代码。

### `BestMatchingSerialCalculation`类

这个类只有一个名为`getBestMatchingWords`()的方法，它接收两个参数：一个带有我们作为参考的序列的字符串，以及包含字典中所有单词的字符串对象列表。它返回一个`BestMatchingData`对象，其中包含算法的结果：

```java
public class BestMatchingSerialCalculation {

    public static BestMatchingData getBestMatchingWords(String word, List<String> dictionary) {
        List<String> results=new ArrayList<String>();
        int minDistance=Integer.MAX_VALUE;
        int distance;
```

在内部变量初始化之后，算法处理字典中的所有单词，计算这些单词与参考字符串之间的 Levenshtein 距离。如果一个单词的计算距离小于实际最小距离，我们清除结果列表并将实际单词存储到列表中。如果一个单词的计算距离等于实际最小距离，我们将该单词添加到结果列表中：

```java
        for (String str: dictionary) {
            distance=LevenshteinDistance.calculate(word,str);
            if (distance<minDistance) {
                results.clear();
                minDistance=distance;
                results.add(str);
            } else if (distance==minDistance) {
                results.add(str);
            }
        }
```

最后，我们创建了`BestMatchingData`对象来返回算法的结果：

```java
        BestMatchingData result=new BestMatchingData();
        result.setWords(results);
        result.setDistance(minDistance);
        return result;
    }

}
```

### `BestMachingSerialMain`类

这是示例的主要类。它加载 UKACD 文件，使用作为参数接收的字符串调用`getBestMatchingWords()`，并在控制台中显示结果，包括算法的执行时间。

```java
public class BestMatchingSerialMain {

    public static void main(String[] args) {

        Date startTime, endTime;
        List<String> dictionary=WordsLoader.load("data/UK Advanced Cryptics Dictionary.txt");

        System.out.println("Dictionary Size: "+dictionary.size());

        startTime=new Date();
        BestMatchingData result= BestMatchingSerialCalculation.getBestMatchingWords (args[0], dictionary);
        List<String> results=result.getWords();
        endTime=new Date();
        System.out.println("Word: "+args[0]);
        System.out.println("Minimum distance: " +result.getDistance());
        System.out.println("List of best matching words: " +results.size());
        results.forEach(System.out::println);
        System.out.println("Execution Time: "+(endTime.getTime()- startTime.getTime()));
    }

}
```

在这里，我们使用了一个名为**方法引用**的新的 Java 8 语言构造和一个新的`List.forEach()`方法来输出结果。

## 最佳匹配算法 - 第一个并发版本

我们实现了两个不同的并发版本的最佳匹配算法。第一个是基于`Callable`接口和`AbstractExecutorService`接口中定义的`submit()`方法。

我们使用了以下三个类来实现算法的这个版本：

+   `BestMatchingBasicTask`类实现了实现`Callable`接口的任务，并将在执行器中执行

+   `BestMatchingBasicConcurrentCalculation`类创建执行器和必要的任务，并将它们发送到执行器

+   `BestMatchingConcurrentMain`类实现了`main()`方法，用于执行算法并在控制台中显示结果

让我们来看看这些类的源代码。

### `BestMatchingBasicTask`类

如前所述，这个类将实现将获得最佳匹配单词列表的任务。这个任务将实现参数化为`BestMatchingData`类的`Callable`接口。这意味着这个类将实现`call()`方法，而这个方法将返回一个`BestMatchingData`对象。

每个任务将处理字典的一部分，并返回该部分获得的结果。我们使用了四个内部属性，如下所示：

+   字典的第一个位置（包括）

+   它将分析的字典的最后位置（不包括）

+   作为字符串对象列表的字典

+   参考输入字符串

这段代码如下：

```java
public class BestMatchingBasicTask implements Callable <BestMatchingData > {

    private int startIndex;

    private int endIndex;

    private List < String > dictionary;

    private String word;

    public BestMatchingBasicTask(int startIndex, int endIndex, List < String > dictionary, String word) {
        this.startIndex = startIndex;
        this.endIndex = endIndex;
        this.dictionary = dictionary;
        this.word = word;
    }
```

`call()`方法处理`startIndex`和`endIndex`属性之间的所有单词，并计算这些单词与输入字符串之间的 Levenshtein 距离。它只会返回距离输入字符串最近的单词。如果在过程中找到比之前更接近的单词，它会清除结果列表并将新单词添加到该列表中。如果找到一个与目前找到的结果距离相同的单词，它会将该单词添加到结果列表中，如下所示：

```java
    @Override
    public BestMatchingData call() throws Exception {
        List<String> results=new ArrayList<String>();
        int minDistance=Integer.MAX_VALUE;
        int distance;
        for (int i=startIndex; i<endIndex; i++) {
            distance = LevenshteinDistance.calculate (word,dictionary.get(i));
            if (distance<minDistance) {
                results.clear();
                minDistance=distance;
                results.add(dictionary.get(i));
            } else if (distance==minDistance) {
                results.add(dictionary.get(i));
            }
        }
```

最后，我们创建了一个`BestMatchingData`对象，其中包含我们找到的单词列表及其与输入字符串的距离，并返回该对象。

```java
        BestMatchingData result=new BestMatchingData();
        result.setWords(results);
        result.setDistance(minDistance);
        return result;
    }
}
```

基于`Runnable`接口的任务与`run()`方法中包含的返回语句的主要区别。`run()`方法不返回值，因此这些任务无法返回结果。另一方面，`call()`方法返回一个对象（该对象的类在实现语句中定义），因此这种类型的任务可以返回结果。

### BestMatchingBasicConcurrentCalculation 类

这个类负责创建处理完整字典所需的任务，执行器来执行这些任务，并控制执行器中任务的执行。

它只有一个方法`getBestMatchingWords()`，接收两个输入参数：完整单词列表的字典和参考字符串。它返回一个包含算法结果的`BestMatchingData`对象。首先，我们创建并初始化了执行器。我们使用机器的核心数作为我们想要在其上使用的最大线程数。

```java
public class BestMatchingBasicConcurrentCalculation {

    public static BestMatchingData getBestMatchingWords(String word, List<String> dictionary) throws InterruptedException, ExecutionException {

        int numCores = Runtime.getRuntime().availableProcessors();
        ThreadPoolExecutor executor = (ThreadPoolExecutor) Executors.newFixedThreadPool(numCores);
```

然后，我们计算每个任务将处理的字典部分的大小，并创建一个`Future`对象的列表来存储任务的结果。当您将基于`Callable`接口的任务发送到执行器时，您将获得`Future`接口的实现。您可以使用该对象来：

+   知道任务是否已执行

+   获取任务执行的结果（`call()`方法返回的对象）

+   取消任务的执行

代码如下：

```java
        int size = dictionary.size();
        int step = size / numCores;
        int startIndex, endIndex;
        List<Future<BestMatchingData>> results = new ArrayList<>();
```

然后，我们创建任务，使用`submit()`方法将它们发送到执行器，并将该方法返回的`Future`对象添加到`Future`对象的列表中。`submit()`方法立即返回。它不会等待任务执行。我们有以下代码：

```java
        for (int i = 0; i < numCores; i++) {
            startIndex = i * step;
            if (i == numCores - 1) {
                endIndex = dictionary.size();
            } else {
                endIndex = (i + 1) * step;
            }
            BestMatchingBasicTask task = new BestMatchingBasicTask(startIndex, endIndex, dictionary, word);
            Future<BestMatchingData> future = executor.submit(task);
            results.add(future);
        }
```

一旦我们将任务发送到执行器，我们调用执行器的`shutdown()`方法来结束其执行，并迭代`Future`对象的列表以获取每个任务的结果。我们使用不带任何参数的`get()`方法。如果任务已经完成执行，该方法将返回`call()`方法返回的对象。如果任务尚未完成，该方法将使当前线程休眠，直到任务完成并且结果可用。

我们用任务的结果组成一个结果列表，因此我们将只返回与参考字符串最接近的单词列表如下：

```java
        executor.shutdown();
        List<String> words=new ArrayList<String>();
        int minDistance=Integer.MAX_VALUE;
        for (Future<BestMatchingData> future: results) {
            BestMatchingData data=future.get();
            if (data.getDistance()<minDistance) {
                words.clear();
                minDistance=data.getDistance();
                words.addAll(data.getWords());
            } else if (data.getDistance()==minDistance) {
                words.addAll(data.getWords());
            }

        }
```

最后，我们创建并返回一个`BestMatchingData`对象，其中包含算法的结果：

```java
        BestMatchingData result=new BestMatchingData();
        result.setDistance(minDistance);
        result.setWords(words);
        return result;
    }
}
```

### 注意

`BestMatchingConcurrentMain`类与之前介绍的`BestMatchingSerialMain`非常相似。唯一的区别是使用的类（`BestMatchingBasicConcurrentCalculation`而不是`BestMatchingSerialCalculation`），因此我们不在这里包含源代码。请注意，我们既没有使用线程安全的数据结构，也没有同步，因为我们的并发任务在独立的数据片段上工作，并且在并发任务终止后，最终结果是以顺序方式合并的。

## 最佳匹配算法 - 第二个并发版本

我们使用`AbstractExecutorService`的`invokeAll()`方法（在`ThreadPoolExecutorClass`中实现）实现了最佳匹配算法的第二个版本。在之前的版本中，我们使用了接收`Callable`对象并返回`Future`对象的`submit()`方法。`invokeAll()`方法接收`Callable`对象的`List`作为参数，并返回`Future`对象的`List`。第一个`Future`与第一个`Callable`相关联，依此类推。这两种方法之间还有另一个重要的区别。虽然`submit()`方法立即返回，但`invokeAll()`方法在所有`Callable`任务结束执行时返回。这意味着所有返回的`Future`对象在调用它们的`isDone()`方法时都将返回`true`。

为了实现这个版本，我们使用了前面示例中实现的`BestMatchingBasicTask`类，并实现了`BestMatchingAdvancedConcurrentCalculation`类。与`BestMatchingBasicConcurrentCalculation`类的区别在于任务的创建和结果的处理。在任务的创建中，现在我们创建一个列表并将其存储在我们要执行的任务上：

```java
        for (int i = 0; i < numCores; i++) {
            startIndex = i * step;
            if (i == numCores - 1) {
                endIndex = dictionary.size();
            } else {
                endIndex = (i + 1) * step;
            }
            BestMatchingBasicTask task = new BestMatchingBasicTask(startIndex, endIndex, dictionary, word);
            tasks.add(task);
        }
```

为了处理结果，我们调用`invokeAll()`方法，然后遍历返回的`Future`对象列表：

```java
        results = executor.invokeAll(tasks);
        executor.shutdown();
        List<String> words = new ArrayList<String>();
        int minDistance = Integer.MAX_VALUE;
        for (Future<BestMatchingData> future : results) {
            BestMatchingData data = future.get();
            if (data.getDistance() < minDistance) {
                words.clear();
                minDistance = data.getDistance();
                words.addAll(data.getWords());
            } else if (data.getDistance()== minDistance) {
                words.addAll(data.getWords());
            }
        }
        BestMatchingData result = new BestMatchingData();
        result.setDistance(minDistance);
        result.setWords(words);
        return result;
    }
```

为了执行这个版本，我们实现了`BestMatchingConcurrentAdvancedMain`。它的源代码与之前的类非常相似，因此不包括在内。

## 单词存在算法-串行版本

作为这个示例的一部分，我们实现了另一个操作，用于检查一个字符串是否存在于我们的单词列表中。为了检查单词是否存在，我们再次使用 Levenshtein 距离。如果一个单词与列表中的一个单词的距离为`0`，我们认为这个单词存在。如果我们使用`equals()`或`equalsIgnoreCase()`方法进行比较，或者将输入单词读入`HashSet`并使用`contains()`方法进行比较（比我们的版本更有效），会更快，但我们认为我们的版本对于本书的目的更有用。

与之前的示例一样，首先我们实现了操作的串行版本，以便将其作为实现并发版本的基础，并比较两个版本的执行时间。

为了实现串行版本，我们使用了两个类：

+   `ExistSerialCalculation`类实现了`existWord()`方法，将输入字符串与字典中的所有单词进行比较，直到找到它

+   `ExistSerialMain`类，启动示例并测量执行时间

让我们分析这两个类的源代码。

### `ExistSerialCalculation`类

这个类只有一个方法，即`existWord()`方法。它接收两个参数：我们要查找的单词和完整的单词列表。它遍历整个列表，计算输入单词与列表中的单词之间的 Levenshtein 距离，直到找到单词（距离为`0`）为止，此时返回`true`值，或者在没有找到单词的情况下完成单词列表，此时返回`false`值。

```java
public class ExistSerialCalculation {

    public static boolean existWord(String word, List<String> dictionary) {
        for (String str: dictionary) {
            if (LevenshteinDistance.calculate(word, str) == 0) {
                return true;
            }
        }
        return false;
    }
}
```

### `ExistSerialMain`类

这个类实现了`main()`方法来调用`exist()`方法。它将主方法的第一个参数作为我们要查找的单词，并调用该方法。它测量其执行时间并在控制台中显示结果。我们有以下代码：

```java
public class ExistSerialMain {

    public static void main(String[] args) {

        Date startTime, endTime;
        List<String> dictionary=WordsLoader.load("data/UK Advanced Cryptics Dictionary.txt");

        System.out.println("Dictionary Size: "+dictionary.size());

        startTime=new Date();
        boolean result=ExistSerialCalculation.existWord(args[0], dictionary);
        endTime=new Date(); 
        System.out.println("Word: "+args[0]);
        System.out.println("Exists: "+result);
        System.out.println("Execution Time: "+(endTime.getTime()- startTime.getTime()));
    }
}
```

## 单词存在算法-并发版本

要实现这个操作的并发版本，我们必须考虑它最重要的特点。我们不需要处理整个单词列表。当我们找到单词时，我们可以结束列表的处理并返回结果。这种不处理整个输入数据并在满足某些条件时停止的操作称为**短路操作**。

`AbstractExecutorService`接口定义了一个操作（在`ThreadPoolExecutor`类中实现），与这个想法完美契合。它是`invokeAny()`方法。这个方法将`Callable`任务列表发送到执行器，并返回第一个完成执行而不抛出异常的任务的结果。如果所有任务都抛出异常，这个方法会抛出`ExecutionException`异常。

与之前的示例一样，我们实现了不同的类来实现这个算法的版本：

+   `ExistBasicTask`类实现了我们将在执行器中执行的任务

+   `ExistBasicConcurrentCalculation`类创建执行器和任务，并将任务发送到执行器。

+   `ExistBasicConcurrentMain`类执行示例并测量其运行时间

### ExistBasicTasks 类

这个类实现了将要搜索这个单词的任务。它实现了参数化为`Boolean`类的`Callable`接口。如果任务找到单词，`call()`方法将返回`true`值。它使用四个内部属性：

+   完整的单词列表

+   列表中任务将处理的第一个单词（包括）

+   任务将处理的列表中的最后一个单词（不包括）

+   任务将要查找的单词

我们有以下代码：

```java
public class ExistBasicTask implements Callable<Boolean> {

    private int startIndex;

    private int endIndex;

    private List<String> dictionary;

    private String word;

    public ExistBasicTask(int startIndex, int endIndex, List<String> dictionary, String word) {
        this.startIndex=startIndex;
        this.endIndex=endIndex;
        this.dictionary=dictionary;
        this.word=word;
    }
```

`call`方法将遍历分配给该任务的列表部分。它计算输入单词与列表中单词之间的 Levenshtein 距离。如果找到单词，它将返回`true`值。

如果任务处理了所有的单词但没有找到这个单词，它将抛出一个异常以适应`invokeAny()`方法的行为。如果任务在这种情况下返回`false`值，`invokeAny()`方法将立即返回`false`值，而不会等待其他任务。也许另一个任务会找到这个单词。

我们有以下代码：

```java
    @Override
    public Boolean call() throws Exception {
        for (int i=startIndex; i<endIndex; i++) {
            if (LevenshteinDistance.calculate(word, dictionary.get(i))==0) {
                return true;
            }
        }
            if (Thread.interrupted()) {
                return false;
            }
        throw new NoSuchElementException("The word "+word+" doesn't exists.");
    }
```

### ExistBasicConcurrentCalculation 类

这个类将在完整的单词列表中执行输入单词的搜索，创建并执行必要的任务。它只实现了一个名为`existWord()`的方法。它接收两个参数，输入字符串和完整的单词列表，并返回一个布尔值，指示单词是否存在。

首先，我们创建执行任务的执行器。我们使用`Executor`类，并创建一个`ThreadPoolExecutor`类，最大线程数由机器的可用硬件线程数确定，如下所示：

```java
public class ExistBasicConcurrentCalculation {

    public static boolean existWord(String word, List<String> dictionary) throws InterruptedException, ExecutionException{
        int numCores = Runtime.getRuntime().availableProcessors();
        ThreadPoolExecutor executor = (ThreadPoolExecutor) Executors.newFixedThreadPool(numCores);
```

然后，我们创建与执行器中运行的线程数相同数量的任务。每个任务将处理单词列表的一个相等部分。我们创建任务并将它们存储在一个列表中：

```java
        int size = dictionary.size();
        int step = size / numCores;
        int startIndex, endIndex;
        List<ExistBasicTask> tasks = new ArrayList<>();

        for (int i = 0; i < numCores; i++) {
            startIndex = i * step;
            if (i == numCores - 1) {
                endIndex = dictionary.size();
            } else {
                endIndex = (i + 1) * step;
            }
            ExistBasicTask task = new ExistBasicTask(startIndex, endIndex, dictionary,
                    word);
            tasks.add(task);
        }
```

然后，我们使用`invokeAny()`方法在执行器中执行任务。如果方法返回布尔值，则单词存在。我们返回该值。如果方法抛出异常，则单词不存在。我们在控制台打印异常并返回`false`值。在这两种情况下，我们调用执行器的`shutdown()`方法来终止其执行，如下所示：

```java
        try {
            Boolean result=executor.invokeAny(tasks);
            return result;
        } catch (ExecutionException e) {
            if (e.getCause() instanceof NoSuchElementException)
                return false;
            throw e;
        } finally {
            executor.shutdown();
        }
    }
}
```

### ExistBasicConcurrentMain 类

这个类实现了这个示例的`main()`方法。它与`ExistSerialMain`类相同，唯一的区别是它使用`ExistBasicConcurrentCalculation`类而不是`ExistSerialCalculation`，因此它的源代码没有包含。

## 比较解决方案

让我们比较我们在本节中实现的两个操作的不同解决方案（串行和并发）。为了测试算法，我们使用了 JMH 框架（[`openjdk.java.net/projects/code-tools/jmh/`](http://openjdk.java.net/projects/code-tools/jmh/)），它允许您在 Java 中实现微基准测试。使用基准测试框架比简单地使用`currentTimeMillis()`或`nanoTime()`方法来测量时间更好。我们在一个四核处理器的计算机上执行了 10 次，并计算了这 10 次的中等执行时间。让我们分析执行结果。

### 最佳匹配算法

在这种情况下，我们实现了算法的三个版本：

+   串行版本

+   并发版本，一次发送一个任务

+   并发版本，使用`invokeAll()`方法

为了测试算法，我们使用了三个不在单词列表中的不同字符串：

+   `Stitter`

+   `Abicus`

+   `Lonx`

这些是最佳匹配算法对每个单词返回的单词：

+   `Stitter`：`sitter`、`skitter`、`slitter`、`spitter`、`stilter`、`stinter`、`stotter`、`stutter`和`titter`

+   `Abicus`：`abacus`和`amicus`

+   `Lonx`：`lanx`、`lone`、`long`、`lox`和`lynx`

下表讨论了中等执行时间及其毫秒标准偏差：

| 算法 | Stitter | Abicus | lonx |
| --- | --- | --- | --- |
| 串行 | 467.01 ± 23.40 | 408.03 ± 14.66 | 317.60 ± 28.78 |
| 并发：`submit()`方法 | 209.72 ± 74.79 | 184.10 ± 90.47 | 155.61 ± 65.43 |
| 并发：`invokeAll()`方法 | 217.66 ± 65.46 | 188.28 ± 81.28 | 160.43 ± 65.14 |

我们可以得出以下结论：

+   算法的并发版本比串行版本获得更好的性能。

+   算法的并发版本之间获得了类似的结果。所有并发版本的标准偏差值都非常高。我们可以使用单词`lonx`的加速度比来比较并发版本方法和串行版本，以了解并发如何提高算法的性能：![最佳匹配算法](img/00011.jpeg)

### 存在的算法

在这种情况下，我们实现了两个版本的算法：

+   串行版本

+   使用`invokeAny()`方法的并发版本

为了测试算法，我们使用了一些字符串：

+   在单词列表中不存在的单词`xyzt`

+   在单词列表的末尾附近存在的单词`stutter`

+   在单词列表的开始附近存在的单词`abacus`

+   在单词列表的后半部分之后存在的单词`lynx`

毫秒中的中等执行时间和它们的标准偏差显示在下表中：

| 算法 | 单词 | 执行时间（毫秒） |
| --- | --- | --- |
| 串行 | `abacus` | 50.70 ± 13.95 |
|   | `lynx` | 194.41 ± 26.02 |
| `stutter` | 398.11 ± 23.4 |
| `xyzt` | 315.62 ± 28.7 |
| 并发 | `abacus` | 50.72 ± 7.17 |
|   | `lynx` | 69.15 ± 62.5 |
| `stutter` | 126.74 ± 104.52 |
| `xyzt` | 203.37 ± 76.67 |

我们可以得出以下结论：

+   一般来说，并发版本的算法比串行版本提供更好的性能。

+   单词在列表中的位置是一个关键因素。对于单词`abacus`，它出现在列表的开头，两种算法给出了类似的执行时间，但对于单词`stutter`，差异非常大。

+   并发情况下的标准偏差非常大。

如果我们使用加速度比较并发版本和串行版本的单词`lynx`，结果是：

![存在的算法](img/00012.jpeg)

# 第二个例子 - 为文档集合创建倒排索引

在**信息检索**领域，**倒排索引**是一种常用的数据结构，用于加速对文档集合中文本的搜索。它存储文档集合的所有单词以及包含该单词的文档列表。

要构建索引，我们必须解析集合中的所有文档，并以增量方式构建索引。对于每个文档，我们提取该文档的重要单词（删除最常见的单词，也称为停用词，可能应用词干算法），然后将这些单词添加到索引中。如果单词存在于索引中，我们将文档添加到与该单词关联的文档列表中。如果单词不存在，则将单词添加到索引的单词列表中，并将文档与该单词关联。您可以添加参数到关联中，如单词在文档中的**词频**，这将为您提供更多信息。

当您在文档集合中搜索一个单词或一组单词时，您使用倒排索引来获取与每个单词关联的文档列表，并创建一个包含搜索结果的唯一列表。

在本节中，您将学习如何使用 Java 并发工具来为文档集合构建倒排索引文件。作为文档集合，我们已经获取了包含有关电影信息的维基百科页面，以构建一组 100,673 个文档。我们已经将每个维基百科页面转换为文本文件。您可以下载包含有关该书的所有信息的文档集合。

为了构建倒排索引，我们不删除任何单词，也不使用任何词干算法。我们希望尽可能简单地保持算法，以便将注意力集中在并发工具上。

这里解释的相同原则可以用于获取关于文档集合的其他信息，例如，每个文档的向量表示，可以作为**聚类算法**的输入，正如您将在第六章中学到的，*优化分治解决方案 - 分叉/加入框架*。

与其他示例一样，您将实现这些操作的串行和并发版本，以验证并发在这种情况下是否有帮助。

## 通用类

串行和并发版本都共同使用类将文档集合加载到 Java 对象中。我们使用了以下两个类：

+   存储在文档中的单词列表的`Document`类

+   `DocumentParse`类将存储在文件中的文档转换为文档对象

让我们分析这两个类的源代码。

### Document 类

`Document`类非常简单。它只有两个属性和用于获取和设置这些属性值的方法。这些属性是：

+   文件名，作为字符串。

+   词汇表（即文档中使用的单词列表）作为`HashMap`。**键**是**单词**，值是单词在文档中出现的次数。

### DocumentParser 类

正如我们之前提到的，这个类将存储在文件中的文档转换为`Document`对象。它将这个单词分成三个方法。第一个是`parse()`方法，它接收文件路径作为参数，并返回该文档的词汇`HashMap`。这个方法逐行读取文件，并使用`parseLine()`方法将每一行转换为一个单词列表，并将它们添加到词汇中，如下所示：

```java
public class DocumentParser {

    public Map<String, Integer>  parse(String route) {
        Map<String, Integer> ret=new HashMap<String,Integer>();
        Path file=Paths.get(route);
        try ( BufferedReader reader = Files.newBufferedReader(file)) {
                String line = null;
                while ((line = reader.readLine()) != null) {
                    parseLine(line,ret);
                }
            } catch (IOException x) {
              x.printStackTrace();
            } catch (Exception e) {
              e.printStackTrace();
            }
        return ret;

    }
```

`parseLine()`方法处理提取其单词的行。我们认为一个单词是一个字母序列，以便继续这个例子的简单性。我们已经使用了`Pattern`类来提取单词，使用`Normalizer`类将单词转换为小写并删除元音的重音，如下所示：

```java
private static final Pattern PATTERN = Pattern.compile("\\P{IsAlphabetic}+");

private void parseLine(String line, Map<String, Integer> ret) {
  for(String word: PATTERN.split(line)) {
    if(!word.isEmpty())
      ret.merge(Normalizer.normalize(word, Normalizer.Form.NFKD).toLowerCase(), 1, (a, b) -> a+b);
  }
}
```

## 串行版本

这个示例的串行版本是在`SerialIndexing`类中实现的。这个类有一个`main()`方法，它读取所有文档，获取其词汇，并以增量方式构建倒排索引。

首先，我们初始化必要的变量。文档集合存储在数据目录中，因此我们将所有文档存储在`File`对象的数组中。我们还初始化了`invertedIndex`对象。我们使用`HashMap`，其中键是单词，值是包含该单词的文件名的字符串对象列表，如下所示：

```java
public class SerialIndexing {

    public static void main(String[] args) {

        Date start, end;

        File source = new File("data");
        File[] files = source.listFiles();
        Map<String, List<String>> invertedIndex=new HashMap<String,List<String>> ();
```

然后，我们使用`DocumentParse`类解析所有文档，并使用`updateInvertedIndex()`方法将从每个文档获得的词汇添加到倒排索引中。我们测量整个过程的执行时间。我们有以下代码：

```java
        start=new Date();
        for (File file : files) {

            DocumentParser parser = new DocumentParser();

            if (file.getName().endsWith(".txt")) {
                Map<String, Integer> voc = parser.parse (file.getAbsolutePath());
                updateInvertedIndex(voc,invertedIndex, file.getName());
            }
        }
        end=new Date();
```

最后，我们在控制台上显示执行结果：

```java
        System.out.println("Execution Time: "+(end.getTime()- start.getTime()));
        System.out.println("invertedIndex: "+invertedIndex.size());
    }
```

`updateInvertedIndex()`方法将文档的词汇添加到倒排索引结构中。它处理构成词汇的所有单词。如果单词存在于倒排索引中，我们将文档的名称添加到与该单词关联的文档列表中。如果单词不存在，我们将单词添加并将文档与该单词关联，如下所示：

```java
private static void updateInvertedIndex(Map<String, Integer> voc, Map<String, List<String>> invertedIndex, String fileName) {
  for (String word : voc.keySet()) {
    if (word.length() >= 3) {
      invertedIndex.computeIfAbsent(word, k -> new ArrayList<>()).add(fileName);
    }
  }
}
```

## 第一个并发版本 - 每个文档一个任务

现在是时候实现文本索引算法的并发版本了。显然，我们可以并行处理每个文档的过程。这包括从文件中读取文档并处理每一行以获取文档的词汇表。任务可以将该词汇表作为它们的结果返回，因此我们可以基于`Callable`接口实现任务。

在前面的例子中，我们使用了三种方法将`Callable`任务发送到执行程序：

+   提交()

+   调用所有()

+   调用任意()

我们必须处理所有文档，因此我们必须放弃`invokeAny()`方法。另外两种方法都不方便。如果我们使用`submit()`方法，我们必须决定何时处理任务的结果。如果我们为每个文档发送一个任务，我们可以处理结果：

+   在发送每个任务之后，这是不可行的

+   在所有任务完成后，我们必须存储大量的`Future`对象

+   在发送一组任务后，我们必须包含代码来同步这两个操作。

所有这些方法都有一个问题：我们以顺序方式处理任务的结果。如果我们使用`invokeAll()`方法，我们就处于类似于第 2 点的情况。我们必须等待所有任务完成。

一个可能的选择是创建其他任务来处理与每个任务相关的`Future`对象，而 Java 并发 API 为我们提供了一种优雅的解决方案，即使用`CompletionService`接口及其实现，即`ExecutorCompletionService`类。

`CompletionService`对象是一个具有执行程序的机制，它允许您解耦任务的生产和对这些任务结果的消费。您可以使用`submit()`方法将任务发送到执行程序，并在任务完成时使用`poll()`或`take()`方法获取任务的结果。因此，对于我们的解决方案，我们将实现以下元素：

+   一个`CompletionService`对象来执行任务。

+   每个文档一个任务，解析文档并生成其词汇表。这个任务将由`CompletionService`对象执行。这些任务在`IndexingTask`类中实现。

+   两个线程来处理任务的结果并构建倒排索引。这些线程在`InvertedIndexTask`类中实现。

+   一个`main()`方法来创建和执行所有元素。这个`main()`方法是在`ConcurrentIndexingMain`类中实现的。

让我们分析这些类的源代码。

### IndexingTask 类

这个类实现了解析文档以获取其词汇表的任务。它实现了参数化为`Document`类的`Callable`接口。它有一个内部属性来存储代表它必须解析的文档的`File`对象。看一下下面的代码：

```java
public class IndexingTask implements Callable<Document> {
    private File file;
    public IndexingTask(File file) {
        this.file=file;
    }
```

在`call()`方法中，它简单地使用`DocumentParser`类的`parse()`方法来解析文档并获取词汇表，并创建并返回包含获取的数据的`Document`对象：

```java
    @Override
    public Document call() throws Exception {
        DocumentParser parser = new DocumentParser();

        Map<String, Integer> voc = parser.parse(file.getAbsolutePath());

        Document document=new Document();
        document.setFileName(file.getName());
        document.setVoc(voc);
        return document;
    }
}
```

### InvertedIndexTask 类

这个类实现了获取`IndexingTask`对象生成的`Document`对象并构建倒排索引的任务。这些任务将作为`Thread`对象执行（在这种情况下我们不使用执行程序），因此它们基于`Runnable`接口。

`InvertedIndexTask`类使用三个内部属性：

+   一个参数化为`Document`类的`CompletionService`对象，以访问`IndexingTask`对象返回的对象。

+   一个`ConcurrentHashMap`来存储倒排索引。键是单词，值是`ConcurrentLinkedDeque`，其中包含文件的名称。在这种情况下，我们必须使用并发数据结构，而串行版本中使用的数据结构没有同步。

+   一个布尔值来指示任务可以完成其工作。

其代码如下：

```java
public class InvertedIndexTask implements Runnable {

    private CompletionService<Document> completionService;
    private ConcurrentHashMap<String, ConcurrentLinkedDeque<String>> invertedIndex;

    public InvertedIndexTask(CompletionService<Document> completionService,
            ConcurrentHashMap<String, ConcurrentLinkedDeque<String>> invertedIndex) {
        this.completionService = completionService;
        this.invertedIndex = invertedIndex;

    }
```

`run()`方法使用`CompletionService`的`take()`方法获取与任务关联的`Future`对象。我们实现一个循环，直到线程被中断为止。一旦线程被中断，它将使用`take()`方法再次处理所有未决的`Future`对象。我们使用`take()`方法返回的对象更新倒排索引，使用`updateInvertedIndex()`方法。我们有以下方法：

```java
public void run() {
        try {
            while (!Thread.interrupted()) {
                try {
                    Document document = completionService.take().get();
                    updateInvertedIndex(document.getVoc(), invertedIndex, document.getFileName());
                } catch (InterruptedException e) {
                    break;
                }
            }
            while (true) {
                Future<Document> future = completionService.poll();
                if (future == null)
                    break;
                Document document = future.get();
                updateInvertedIndex(document.getVoc(), invertedIndex, document.getFileName());
            }
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
    }
```

最后，`updateInvertedIndex`方法接收从文档中获取的词汇表、倒排索引和已处理文件的名称作为参数。它处理词汇表中的所有单词。如果单词不存在，我们使用`computeIfAbsent()`方法将单词添加到`invertedIndex`中：

```java
     private void updateInvertedIndex(Map<String, Integer> voc, ConcurrentHashMap<String, ConcurrentLinkedDeque<String>> invertedIndex, String fileName) {
        for (String word : voc.keySet()) {
            if (word.length() >= 3) {
                invertedIndex.computeIfAbsent(word, k -> new ConcurrentLinkedDeque<>()).add(fileName);
            }
        }
    }
```

### 并发索引类

这是示例中的主要类。它创建和启动所有组件，等待其完成，并在控制台中打印最终执行时间。

首先，它创建并初始化了所有需要执行的变量：

+   一个执行器来运行`InvertedTask`任务。与之前的示例一样，我们使用机器的核心数作为执行器中工作线程的最大数量，但在这种情况下，我们留出一个核心来执行独立线程。

+   一个`CompletionService`对象来运行任务。我们使用之前创建的执行程序来初始化这个对象。

+   一个`ConcurrentHashMap`来存储倒排索引。

+   一个`File`对象数组，其中包含我们需要处理的所有文档。

我们有以下方法：

```java
public class ConcurrentIndexing {

    public static void main(String[] args) {

        int numCores=Runtime.getRuntime().availableProcessors();
        ThreadPoolExecutor executor=(ThreadPoolExecutor) Executors.newFixedThreadPool(Math.max(numCores-1, 1));
        ExecutorCompletionService<Document> completionService=new ExecutorCompletionService<>(executor);
        ConcurrentHashMap<String, ConcurrentLinkedDeque<String>> invertedIndex=new ConcurrentHashMap <String,ConcurrentLinkedDeque<String>> ();

        Date start, end;

        File source = new File("data");
        File[] files = source.listFiles();
```

然后，我们处理数组中的所有文件。对于每个文件，我们创建一个`InvertedTask`对象，并使用`submit()`方法将其发送到`CompletionService`类：

```java
        start=new Date();
        for (File file : files) {
            IndexingTask task=new IndexingTask(file);
            completionService.submit(task);
        }
```

然后，我们创建两个`InvertedIndexTask`对象来处理`InvertedTask`任务返回的结果，并将它们作为普通的`Thread`对象执行：

```java
        InvertedIndexTask invertedIndexTask=new InvertedIndexTask(completionService,invertedIndex);
        Thread thread1=new Thread(invertedIndexTask);
        thread1.start();
        InvertedIndexTask invertedIndexTask2=new InvertedIndexTask(completionService,invertedIndex);
        Thread thread2=new Thread(invertedIndexTask2);
        thread2.start();
```

一旦我们启动了所有元素，我们等待执行器的完成，使用`shutdown()`和`awaitTermination()`方法。`awaitTermination()`方法将在所有`InvertedTask`任务完成执行时返回，因此我们可以完成执行`InvertedIndexTask`任务的线程。为此，我们中断这些线程（参见我关于`InvertedIndexTask`的评论）。

```java
        executor.shutdown();
        try {
            executor.awaitTermination(1, TimeUnit.DAYS);
            thread1.interrupt();
            thread2.interrupt();
            thread1.join();
            thread2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
```

最后，我们在控制台中写入倒排索引的大小和整个过程的执行时间：

```java
        end=new Date();
        System.out.println("Execution Time: "+(end.getTime()- start.getTime()));
        System.out.println("invertedIndex: "+invertedIndex.size());
    }

}
```

## 第二个并发版本 - 每个任务处理多个文档

我们实现了这个示例的第二个并发版本。基本原则与第一个版本相同，但在这种情况下，每个任务将处理多个文档而不是只有一个。每个任务处理的文档数量将是主方法的输入参数。我们已经测试了每个任务处理 100、1,000 和 5,000 个文档的结果。

为了实现这种新方法，我们将实现三个新类：

+   `MultipleIndexingTask`类，相当于`IndexingTask`类，但它将处理一个文档列表，而不是只有一个

+   `MultipleInvertedIndexTask`类，相当于`InvertedIndexTask`类，但现在任务将检索一个`Document`对象的列表，而不是只有一个

+   `MultipleConcurrentIndexing`类，相当于`ConcurrentIndexing`类，但使用新的类

由于大部分源代码与之前的版本相似，我们只展示不同之处。

### 多重索引任务类

正如我们之前提到的，这个类与之前介绍的`IndexingTask`类相似。主要区别在于它使用一个`File`对象的列表，而不是只有一个文件：

```java
public class MultipleIndexingTask implements Callable<List<Document>> {

    private List<File> files;

    public MultipleIndexingTask(List<File> files) {
        this.files = files;
    }
```

`call()`方法返回一个`Document`对象的列表，而不是只有一个：

```java
    @Override
    public List<Document> call() throws Exception {
        List<Document> documents = new ArrayList<Document>();
        for (File file : files) {
            DocumentParser parser = new DocumentParser();

            Hashtable<String, Integer> voc = parser.parse (file.getAbsolutePath());

            Document document = new Document();
            document.setFileName(file.getName());
            document.setVoc(voc);

            documents.add(document);
        }

        return documents;
    }
}
```

### 多重倒排索引任务类

正如我们之前提到的，这个类与之前介绍的`InvertedIndexClass`类相似。主要区别在于`run（）`方法。`poll（）`方法返回的`Future`对象返回一个`Document`对象列表，因此我们必须处理整个列表。

```java
    @Override
    public void run() {
        try {
            while (!Thread.interrupted()) {
                try {
                    List<Document> documents = completionService.take().get();
                    for (Document document : documents) {
                        updateInvertedIndex(document.getVoc(), invertedIndex, document.getFileName());
                    }
                } catch (InterruptedException e) {
                    break;
                }
            }
            while (true) {
                Future<List<Document>> future = completionService.poll();
                if (future == null)
                    break;
                List<Document> documents = future.get();
                for (Document document : documents) {
                    updateInvertedIndex(document.getVoc(), invertedIndex, document.getFileName());
                }
            }
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
    }
```

### MultipleConcurrentIndexing 类

正如我们之前提到的，这个类与`ConcurrentIndexing`类相似。唯一的区别在于利用新类和使用第一个参数来确定每个任务处理的文档数量。我们有以下方法：

```java
        start=new Date();
        List<File> taskFiles=new ArrayList<>();
        for (File file : files) {
            taskFiles.add(file);
            if (taskFiles.size()==NUMBER_OF_TASKS) {
                MultipleIndexingTask task=new MultipleIndexingTask(taskFiles);
                completionService.submit(task);
                taskFiles=new ArrayList<>();
            }
        }
        if (taskFiles.size()>0) {
            MultipleIndexingTask task=new MultipleIndexingTask(taskFiles);
            completionService.submit(task);
        }

        MultipleInvertedIndexTask invertedIndexTask=new MultipleInvertedIndexTask (completionService,invertedIndex);
        Thread thread1=new Thread(invertedIndexTask);
        thread1.start();
        MultipleInvertedIndexTask invertedIndexTask2=new MultipleInvertedIndexTask (completionService,invertedIndex);
        Thread thread2=new Thread(invertedIndexTask2);
        thread2.start();
```

## 比较解决方案

让我们比较一下我们实现的三个版本的解决方案。正如我们之前提到的，就像文档集合一样，我们已经获取了包含有关电影信息的维基百科页面，构建了一组 100,673 个文档。我们已经将每个维基百科页面转换成了一个文本文件。您可以下载包含有关该书的所有信息的文档集合。

我们执行了五个不同版本的解决方案：

+   串行版本

+   每个文档一个任务的并发版本

+   具有多个任务的并发版本，每个文档 100、1,000 和 5,000 个文档

以下表格显示了五个版本的执行时间：

| 算法 | 执行时间（毫秒） |
| --- | --- |
| 串行 | 69,480.50 |
| 并发：每个任务一个文档 | 49,655.49 |
| 并发：每个任务 100 个文档 | 48,438.14 |
| 并发：每个任务 1,000 个文档 | 49,362.37 |
| 并发：每个任务 5,000 个文档 | 58,362.22 |

我们可以得出以下结论：

+   并发版本总是比串行版本获得更好的性能

+   对于并发版本，如果我们增加每个任务的文档数量，结果会变得更糟。

如果我们使用加速比将并发版本与串行版本进行比较，结果如下：

![比较解决方案](img/00013.jpeg)

## 其他感兴趣的方法

在本章中，我们使用了`AbstractExecutorService`接口（在`ThreadPoolExecutor`类中实现）和`CompletionService`接口（在`ExecutorCompletionService`中实现）的一些方法来管理`Callable`任务的结果。但是，我们还有其他版本的方法和其他要在这里提到的方法。

关于`AbstractExecutorService`接口，让我们讨论以下方法：

+   `invokeAll（Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit）`：此方法在所有任务完成其执行或第二个和第三个参数指定的超时到期时，返回与作为参数传递的`Callable`任务列表相关联的`Future`对象列表。

+   `invokeAny（Collection<? Extends Callable<T>> tasks, long timeout, TimeUnit unit）`：此方法返回作为参数传递的`Callable`任务列表中第一个任务的结果，如果它在第二个和第三个参数指定的超时之前完成执行而不抛出异常，则超时后抛出`TimeoutException`异常。

关于`CompletionService`接口，让我们讨论以下方法：

+   `poll（）`方法：我们使用了带有两个参数的此方法的版本，但也有一个不带参数的版本。从内部数据结构来看，此版本检索并删除自上次调用`poll（）`或`take（）`方法以来已完成的下一个任务的`Future`对象。如果没有任务完成，其执行返回`null`值。

+   “take（）”方法：此方法类似于上一个方法，但如果没有任务完成，它会使线程休眠，直到一个任务完成其执行。

# 总结

在本章中，您学习了可以用来处理返回结果的任务的不同机制。这些任务基于`Callable`接口，该接口声明了`call（）`方法。这是一个由`call`方法返回的类的参数化接口。

当您在执行器中执行`Callable`任务时，您将始终获得`Future`接口的实现。您可以使用此对象来取消任务的执行，了解任务是否已完成其执行或获取“call（）”方法返回的结果。

您可以使用三种不同的方法将`Callable`任务发送到执行器。使用“submit（）”方法，您发送一个任务，并且将立即获得与此任务关联的`Future`对象。使用“invokeAll（）”方法，您发送一个任务列表，并在所有任务完成执行时获得`Future`对象列表。使用“invokeAny（）”方法，您发送一个任务列表，并且将接收第一个完成而不抛出异常的任务的结果（不是`Future`对象）。其余任务将被取消。

Java 并发 API 提供了另一种机制来处理这些类型的任务。这种机制在`CompletionService`接口中定义，并在`ExecutorCompletionService`类中实现。该机制允许您解耦任务的执行和其结果的处理。`CompletionService`接口在内部使用执行器，并提供“submit（）”方法将任务发送到`CompletionService`接口，并提供“poll（）”和“take（）”方法来获取任务的结果。这些结果以任务完成执行的顺序提供。

您还学会了如何在两个真实世界的例子中实现这些概念：

+   使用 UKACD 数据集的最佳匹配算法

+   使用从维基百科提取的有关电影的信息的数据集的倒排索引构造器

在下一章中，您将学习如何以并发方式执行可以分为阶段的算法，例如关键词提取算法。您可以按照以下三个步骤实现该算法：

1.  第一步 - 解析所有文档并提取所有单词。

1.  第二步 - 计算每个文档中每个单词的重要性。

1.  第三步 - 获取最佳关键词。

这些步骤的主要特点是，您必须在开始下一个步骤之前完全完成一个步骤。Java 并发 API 提供了`Phaser`类来促进这些算法的并发实现。它允许您在阶段结束时同步涉及其中的所有任务，因此在所有任务完成当前任务之前，没有一个任务会开始下一个任务。
