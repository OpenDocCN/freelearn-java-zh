# 第五章：利用新的 API 来改进您的代码

在之前的课程中，我们谈到了改进 Java 应用程序性能的可能方法--从使用新的命令和监控工具到添加多线程和引入响应式编程，甚至到将当前解决方案彻底重新架构为一组不规则且灵活的小独立部署单元和微服务。在不了解您特定情况的情况下，我们无法猜测提供的建议中哪些对您有帮助。这就是为什么在本课程中，我们还将描述 JDK 的一些最新添加，这对您也可能有帮助。正如我们在上一课中提到的，性能和整体代码改进并不总是需要我们彻底重新设计它。小的增量变化有时会带来比我们预期的更显著的改进。

回到我们建造金字塔的类比，与其试图改变石块交付到最终目的地的物流以缩短建造时间，通常更明智的是首先仔细查看建筑工人正在使用的工具。如果每个操作都可以在一半的时间内完成，那么项目交付的整体时间可以相应缩短，即使每个石块的旅行距离相同，甚至更大。

这些是我们将在本课程中讨论的编程工具的改进：

+   使用流上的过滤器作为查找所需内容和减少工作量的方法

+   一种新的堆栈遍历 API，作为分析堆栈跟踪的方式，以便自动应用更正

+   创建紧凑的、不可修改的集合实例的新便利的静态工厂方法

+   新的`CompletableFuture`类作为访问异步处理结果的方法

+   JDK 9 流 API 的改进，可以加快处理速度，同时使您的代码更易读

# 过滤流

`java.util.streams.Stream`接口是在 Java 8 中引入的。它发出元素并支持执行基于这些元素的各种操作的计算。流可以是有限的或无限的，发射速度快或慢。自然地，总是担心新发出的元素的速率可能高于处理的速率。此外，跟上输入的能力反映了应用程序的性能。`Stream`实现通过使用缓冲区和其他各种技术来调整发射和处理速率来解决反压（当元素处理速率低于它们的发射速率时）。此外，如果应用程序开发人员确保尽早做出有关处理或跳过每个特定元素的决定，以便不浪费处理资源，这总是有帮助的。根据情况，可以使用不同的操作来过滤数据。

## 基本过滤

进行过滤的第一种最直接的方法是使用`filter()`操作。为了演示所有以下的功能，我们将使用`Senator`类：

```java
public class Senator {
    private int[] voteYes, voteNo;
    private String name, party;
    public Senator(String name, String party, 
                     int[] voteYes, int[] voteNo) {
        this.voteYes = voteYes;
        this.voteNo = voteNo;
        this.name = name;
        this.party = party;
    }
    public int[] getVoteYes() { return voteYes; }
    public int[] getVoteNo() { return voteNo; }
    public String getName() { return name; }
    public String getParty() { return party; }
    public String toString() {
        return getName() + ", P" + 
          getParty().substring(getParty().length() - 1);
    }
}
```

如您所见，这个类捕获了参议员的姓名、政党以及他们对每个问题的投票情况（`0`表示`否`，`1`表示`是`）。对于特定问题`i`，如果`voteYes[i]=0`，而`voteNo[i]=0`，这意味着参议员不在场。对于同一个问题，不可能同时有`voteYes[i]=1`和`voteNo[i]=1`。

假设有 100 名参议员，每个人属于两个政党中的一个：`Party1`或`Party2`。我们可以使用这些对象来收集参议员对最近 10 个问题的投票统计，使用`Senate`类：

```java
public class Senate {
  public static List<Senator> getSenateVotingStats(){
     List<Senator> results = new ArrayList<>();
     results.add(new Senator("Senator1", "Party1", 
                       new int[]{1,0,0,0,0,0,1,0,0,1}, 
                       new int[]{0,1,0,1,0,0,0,0,1,0}));
     results.add(new Senator("Senator2", "Party2", 
                       new int[]{0,1,0,1,0,1,0,1,0,0}, 
                       new int[]{1,0,1,0,1,0,0,0,0,1}));
     results.add(new Senator("Senator3", "Party1", 
                       new int[]{1,0,0,0,0,0,1,0,0,1}, 
                       new int[]{0,1,0,1,0,0,0,0,1,0}));
     results.add(new Senator("Senator4", "Party2", 
                       new int[]{1,0,1,0,1,0,1,0,0,1}, 
                       new int[]{0,1,0,1,0,0,0,0,1,0}));
     results.add(new Senator("Senator5", "Party1", 
                       new int[]{1,0,0,1,0,0,0,0,0,1}, 
                       new int[]{0,1,0,0,0,0,1,0,1,0}));
     IntStream.rangeClosed(6, 98).forEach(i -> {
       double r1 = Math.random();
       String name = "Senator" + i;
       String party = r1 > 0.5 ? "Party1" : "Party2";
       int[] voteNo = new int[10];
       int[] voteYes = new int[10];
       IntStream.rangeClosed(0, 9).forEach(j -> {
         double r2 = Math.random();
         voteNo[j] = r2 > 0.4 ? 0 : 1;
         voteYes[j] = r2 < 0.6 ? 0 : 1;
       });
       results.add(new Senator(name,party,voteYes,voteNo));
     });
     results.add(new Senator("Senator99", "Party1", 
                       new int[]{0,0,0,0,0,0,0,0,0,0}, 
                       new int[]{1,1,1,1,1,1,1,1,1,1}));
        results.add(new Senator("Senator100", "Party2",
                       new int[]{1,1,1,1,1,1,1,1,1,1}, 
                       new int[]{0,0,0,0,0,0,0,0,0,0}));
        return results;
    }
    public static int timesVotedYes(Senator senator){
        return Arrays.stream(senator.getVoteYes()).sum();
    }
}
```

我们为前五位参议员硬编码了统计数据，这样我们在测试我们的过滤器时可以获得可预测的结果，并验证过滤器的工作。我们还为最后两位参议员硬编码了投票统计数据，这样我们在寻找只对每个问题投了“是”或只投了“否”的参议员时可以获得可预测的计数。我们还添加了`timesVotedYes()`方法，它提供了给定`senator`投了多少次“是”的计数。

现在我们可以从`Senate`类中收集一些数据。例如，让我们看看每个党派在`Senate`类中有多少成员：

```java
List<Senator> senators = Senate.getSenateVotingStats();
long c1 = senators.stream()
   .filter(s -> s.getParty() == "Party1").count();
System.out.println("Members of Party1: " + c1);

long c2 = senators.stream()
   .filter(s -> s.getParty() == "Party2").count();
System.out.println("Members of Party2: " + c2);
System.out.println("Members of the senate: " + (c1 + c2));
```

由于我们在`Senate`类中使用了随机值生成器，因此前面代码的结果会因运行不同而不同，因此如果您尝试运行示例，不要期望看到完全相同的数字。重要的是两个党派成员的总数应该等于 100——`Senate`类中参议员的总数：

![基本过滤](img/05_01.jpg)

表达式`s -> s.getParty()=="Party1"`是过滤器，只过滤出那些属于`Party1`的参议员。因此，`Party2`的元素（`Senator`对象）不会通过，也不会包括在计数中。这很直接了当。

现在让我们看一个更复杂的过滤示例。让我们计算每个党派有多少名参议员在`issue 3`上投票：

```java
int issue = 3;
c1 = senators.stream()
  .filter(s -> s.getParty() == "Party1")
  .filter(s -> s.getVoteNo()[issue] != s.getVoteYes()[issue])
  .count();
System.out.println("Members of Party1 who voted on Issue" + 
                                          issue + ": " + c1);

c2 = senators.stream()
  .filter(s -> s.getParty() == "Party2" &&
               s.getVoteNo()[issue] != s.getVoteYes()[issue])
  .count();
System.out.println("Members of Party2 who voted on Issue" + 
                                          issue + ": " + c2);
System.out.println("Members of the senate who voted on Issue" 
                                 + issue + ": " + (c1 + c2));
```

对于`Party1`，我们使用了两个过滤器。对于`Party2`，我们将它们合并只是为了展示另一个可能的解决方案。这里的重点是首先使用按党派过滤（`s -> s.getParty() == "Party1"`）的过滤器，然后再使用选择只投票的过滤器。这样，第二个过滤器只用于大约一半的元素。否则，如果首先放置选择只投票的过滤器，它将应用于`Senate`的全部 100 名成员。

结果如下：

![基本过滤](img/05_02.jpg)

同样，我们可以计算每个党派有多少成员在`issue 3`上投了“是”：

```java
c1 = senators.stream()
        .filter(s -> s.getParty() == "Party1" &&
                     s.getVoteYes()[issue] == 1)
        .count();
System.out.println("Members of Party1 who voted Yes on Issue"
                                        + issue + ": " + c1);

c2 = senators.stream()
        .filter(s -> s.getParty() == "Party2" &&
                     s.getVoteYes()[issue] == 1)
        .count();
System.out.println("Members of Party2 who voted Yes on Issue"
                                        + issue + ": " + c2);
System.out.println("Members of the senate voted Yes on Issue"
                                 + issue + ": " + (c1 + c2));
```

前面代码的结果如下：

![基本过滤](img/05_03.jpg)

我们可以通过利用 Java 的函数式编程能力（使用 lambda 表达式）并创建`countAndPrint()`方法来重构前面的示例：

```java
long countAndPrint(List<Senator> senators, 
       Predicate<Senator> pred1, Predicate<Senator> pred2, 
                                           String prefix) {
    long c = senators.stream().filter(pred1::test)
                              .filter(pred2::test).count();
    System.out.println(prefix + c);
    return c;
}
```

现在所有之前的代码可以以更紧凑的方式表达：

```java
int issue = 3;

Predicate<Senator> party1 = s -> s.getParty() == "Party1";
Predicate<Senator> party2 = s -> s.getParty() == "Party2";
Predicate<Senator> voted3 = 
       s -> s.getVoteNo()[issue] != s.getVoteYes()[issue];
Predicate<Senator> yes3 = s -> s.getVoteYes()[issue] == 1;

long c1 = countAndPrint(senators, party1, s -> true, 
                                   "Members of Party1: ");
long c2 = countAndPrint(senators, party2, s -> true, 
                                   "Members of Party2: ");
System.out.println("Members of the senate: " + (c1 + c2));

c1 = countAndPrint(senators, party1, voted3, 
   "Members of Party1 who voted on Issue" + issue + ": ");
c2 = countAndPrint(senators, party2, voted3, 
   "Members of Party2 who voted on Issue" + issue + ": ");
System.out.println("Members of the senate who voted on Issue"
                                 + issue + ": " + (c1 + c2));

c1 = countAndPrint(senators, party1, yes3, 
  "Members of Party1 who voted Yes on Issue" + issue + ": ");
c2 = countAndPrint(senators, party2, yes3, 
  "Members of Party2 who voted Yes on Issue" + issue + ": ");
System.out.println("Members of the senate voted Yes on Issue" 
                                 + issue + ": " + (c1 + c2));
```

我们创建了四个谓词，`party1`，`party2`，`voted3`和`yes3`，并且我们多次将它们用作`countAndPrint()`方法的参数。这段代码的输出与之前的示例相同：

![基本过滤](img/05_04.jpg)

使用`Stream`接口的`filter()`方法是过滤的最流行方式。但是也可以使用其他`Stream`方法来实现相同的效果。

### 使用其他 Stream 操作进行过滤

或者，或者除了前一节中描述的基本过滤之外，其他操作（`Stream`接口的方法）也可以用于选择和过滤发出的流元素。

例如，让我们使用`flatMap()`方法按其党派成员身份过滤出参议院成员：

```java
long c1 = senators.stream()
        .flatMap(s -> s.getParty() == "Party1" ? 
                      Stream.of(s) : Stream.empty())
        .count();
System.out.println("Members of Party1: " + c1);
```

这种方法利用了`Stream.of()`（生成一个元素的流）和`Stream.empty()`工厂方法（它生成一个没有元素的流，因此不会向下游发出任何内容）。或者，可以使用一个新的工厂方法（在 Java 9 中引入）`Stream.ofNullable()`来实现相同的效果：

```java
c1 = senators.stream().flatMap(s -> 
  Stream.ofNullable(s.getParty() == "Party1" ? s : null))
                                                 .count();
System.out.println("Members of Party1: " + c1);
```

`Stream.ofNullable()`方法如果不是`null`则创建一个元素的流；否则，创建一个空流，就像前面的示例一样。如果我们对相同的参议院组成运行它们，那么前面的两个代码片段会产生相同的输出：

![使用其他 Stream 操作进行过滤](img/05_05.jpg)

然而，使用`java.uti.Optional`类也可以实现相同的结果，该类可能包含或不包含值。如果值存在（且不为`null`），则其`isPresent()`方法返回`true`，`get()`方法返回该值。以下是我们如何使用它来过滤一个党派的成员：

```java
long c2 = senators.stream()
  .map(s -> s.getParty() == "Party2" ? 
                         Optional.of(s) : Optional.empty())
  .flatMap(o -> o.map(Stream::of).orElseGet(Stream::empty))
  .count();
System.out.println("Members of Party2: " + c2);
```

首先，我们将一个元素（`Senator`对象）映射（转换）为一个带有或不带有值的`Optional`对象。接下来，我们使用`flatMap()`方法来生成一个单个元素的流，或者是一个空流，然后计算通过的元素数量。在 Java 9 中，`Optional`类获得了一个新的工厂`stream()`方法，如果`Optional`对象携带非空值，则生成一个元素的流；否则，生成一个空流。使用这个新方法，我们可以将前面的代码重写如下：

```java
long c2 = senators.stream()
  .map(s -> s.getParty() == "Party2" ? 
                         Optional.of(s) : Optional.empty())
  .flatMap(Optional::stream)
  .count();
System.out.println("Members of Party2: " + c2);
```

如果我们对相同的参议院组成运行这两个示例，前面两个示例的输出结果是相同的：

![使用其他流操作进行过滤](img/05_06.jpg)

当我们需要捕获流发出的第一个元素时，我们可以应用另一种过滤。这意味着在发出第一个元素后终止流。例如，让我们找到`Party1`中投了`issue 3`上的第一位参议员：

```java
senators.stream()
  .filter(s -> s.getParty() == "Party1" &&
                            s.getVoteYes()[3] == 1)    
  .findFirst()
  .ifPresent(s -> System.out.println("First senator "
         "of Party1 found who voted Yes on issue 3: " 
                                     + s.getName()));
```

```java
findFirst() method, which does the described job. It returns the Optional object, so we have added another ifPresent() operator that is invoked only if the Optionalobject contains a non-null value. The resulting output is as follows:
```

![使用其他流操作进行过滤](img/05_07.jpg)

这正是我们在`Senate`类中设置数据时所期望的。

同样，我们可以使用`findAny()`方法来找到在`issue 3`上投了`Yes`的任何`senator`：

```java
senators.stream().filter(s -> s.getVoteYes()[3] == 1)
        .findAny()
        .ifPresent(s -> System.out.println("A senator " +
                 "found who voted Yes on issue 3: " + s));
```

结果也和我们预期的一样：

![使用其他流操作进行过滤](img/05_08.jpg)

这通常（但不一定）是流的第一个元素。但是，人们不应该依赖这一假设，特别是在并行处理的情况下。

`Stream`接口还有三种`match`方法，虽然它们返回一个布尔值，但如果不需要特定对象，只需要确定这样的对象是否存在，也可以用于过滤。这些方法的名称分别是`anyMatch()`、`allMatch()`和`noneMatch()`。它们每个都接受一个谓词并返回一个布尔值。让我们从演示`anyMatch()`方法开始。我们将使用它来查找`Party1`中至少有一个投了`issue 3`上的`Yes`的`senator`：

```java
boolean found = senators.stream()
       .anyMatch(s -> (s.getParty() == "Party1" && 
                             s.getVoteYes()[3] == 1));
String res = found ? 
  "At least one senator of Party1 voted Yes on issue 3"
  : "Nobody of Party1 voted Yes on issue 3";
System.out.println(res);
```

运行前面代码的结果应该如下所示：

![使用其他流操作进行过滤](img/05_09.jpg)

为了演示`allMatch()`方法，我们将使用它来查找`Senate`类中`Party1`的所有成员是否在`issue 3`上投了`Yes`：

```java
boolean yes = senators.stream()
    .allMatch(s -> (s.getParty() == "Party1" &&
                           s.getVoteYes()[3] == 1));
String res = yes ? 
  "All senators of Party1 voted Yes on issue 3"
  : "Not all senators of Party1 voted Yes on issue 3";
System.out.println(res);
```

前面代码的结果可能如下所示：

![使用其他流操作进行过滤](img/05_10.jpg)

三种`match`方法中的最后一种--`noneMatch()`方法--将用于确定`Party1`的一些参议员是否在`issue 3`上投了`Yes`：

```java
boolean yes = senators.stream()
   .noneMatch(s -> (s.getParty() == "Party1" && 
                            s.getVoteYes()[3] == 1));
String res = yes ? 
  "None of the senators of Party1 voted Yes on issue 3"
  : "Some of senators of Party1 voted Yes on issue 3";
System.out.println(res);
```

前面示例的结果如下：

![使用其他流操作进行过滤](img/05_11.jpg)

然而，在现实生活中，情况可能会截然不同，因为`Senate`类中有相当多的问题是按党派线投票的。

当我们需要跳过流中的所有重复元素并仅选择唯一元素时，我们需要另一种类型的过滤。`distinct()`方法就是为此设计的。我们将使用它来找到在`Senate`类中有成员的党派的名称：

```java
senators.stream().map(s -> s.getParty())
        .distinct().forEach(System.out::println);
```

结果如预期的那样：

![使用其他流操作进行过滤](img/05_12.jpg)

嗯，这一点并不奇怪吧？

我们还可以使用`limit()`方法来过滤掉`stream`中除了前几个元素之外的所有元素：

```java
System.out.println("These are the first 3 senators " 
                          + "of Party1 in the list:");
senators.stream()
        .filter(s -> s.getParty() == "Party1")
.limit(3)
        .forEach(System.out::println);

System.out.println("These are the first 2 senators "
                           + "of Party2 in the list:");
senators.stream().filter(s -> s.getParty() == "Party2")
.limit(2)
        .forEach(System.out::println);
```

如果你记得我们如何设置列表中的前五位参议员，你可以预测结果会是这样的：

![使用其他流操作进行过滤](img/05_13.jpg)

现在让我们只在流中找到一个元素--最大的一个。为此，我们可以使用`Stream`接口的`max()`方法和`Senate.timeVotedYes()`方法（我们将对每个参议员应用它）：

```java
senators.stream()
   .max(Comparator.comparing(Senate::timesVotedYes))
   .ifPresent(s -> System.out.println("A senator voted "
        + "Yes most of times (" + Senate.timesVotedYes(s) 
                                            + "): " + s));
```

```java
timesVotedYes() method to select the senator who voted Yes most often. You might remember, we have assigned all instances of Yes to Senator100. Let's see if that would be the result:
```

![使用其他流操作进行过滤](img/05_14.jpg)

是的，我们过滤出了`Senator100`，他是在所有 10 个问题上都投了赞成票的人。

同样，我们可以找到在所有 10 个问题上都投了反对票的参议员：

```java
senators.stream()
  .min(Comparator.comparing(Senate::timesVotedYes))
  .ifPresent(s -> System.out.println("A senator voted "
       + "Yes least of times (" + Senate.timesVotedYes(s) 
                                            + "): " + s));
```

我们期望它是`Senator99`，结果如下：

![使用其他流操作进行过滤](img/05_15.jpg)

这就是为什么我们在`Senate`类中硬编码了几个统计数据，这样我们就可以验证我们的查询是否正确。

由于最后两种方法可以帮助我们进行过滤，我们将演示 JDK 9 中引入的`takeWhile()`和`dropWhile()`方法。我们将首先打印出前五位参议员的数据，然后使用`takeWhile()`方法打印出第一位参议员，直到我们遇到投票超过四次的参议员，然后停止打印：

```java
System.out.println("Here is count of times the first "
                            + "5 senators voted Yes:");
senators.stream().limit(5)
  .forEach(s -> System.out.println(s + ": " 
                           + Senate.timesVotedYes(s)));
System.out.println("Stop printing at a senator who "
                     + "voted Yes more than 4 times:");
senators.stream().limit(5)
        .takeWhile(s -> Senate.timesVotedYes(s) < 5)
        .forEach(s -> System.out.println(s + ": " 
                           + Senate.timesVotedYes(s)));
```

前面代码的结果如下：

![使用其他流操作进行过滤](img/05_16.jpg)

`dropWhile()`方法可以用于相反的效果，即过滤掉，跳过前几位参议员，直到我们遇到投票超过四次的参议员，然后继续打印剩下的所有参议员：

```java
System.out.println("Here is count of times the first " 
                             + "5 senators voted Yes:");
senators.stream().limit(5)
        .forEach(s -> System.out.println(s + ": " 
                            + Senate.timesVotedYes(s)));
System.out.println("Start printing at a senator who "
                      + "voted Yes more than 4 times:");
senators.stream().limit(5)
        .dropWhile(s -> Senate.timesVotedYes(s) < 5)
        .forEach(s -> System.out.println(s + ": " 
                            + Senate.timesVotedYes(s)));
System.out.println("...");
```

结果将如下：

![使用其他流操作进行过滤](img/05_17.jpg)

这结束了我们对元素流可以被过滤的方式的演示。我们希望您已经学会了足够的知识，能够为您的任何过滤需求找到解决方案。然而，我们鼓励您自己学习和尝试 Stream API，这样您就可以保留到目前为止学到的知识，并对 Java 9 丰富的 API 有自己的看法。

# 堆栈遍历 API

异常确实会发生，特别是在开发过程中或软件稳定期间。但在一个大型复杂系统中，即使在生产环境中，也有可能出现异常，特别是当多个第三方系统被整合在一起，并且需要以编程方式分析堆栈跟踪以应用自动修正时。在本节中，我们将讨论如何做到这一点。

## Java 9 之前的堆栈分析

使用`java.lang.Thread`和`java.lang.Throwable`类的对象来传统地读取堆栈跟踪是通过从标准输出中捕获它来完成的。例如，我们可以在代码的任何部分包含这行：

```java
Thread.currentThread().dumpStack();
```

前一行将产生以下输出：

![Java 9 之前的堆栈分析](img/05_18.jpg)

同样，我们可以在代码中包含这行：

```java
new Throwable().printStackTrace();
```

然后输出看起来像这样：

![Java 9 之前的堆栈分析](img/05_19.jpg)

这个输出可以被程序捕获、读取和分析，但需要相当多的自定义代码编写。

JDK 8 通过使用流使这变得更容易。以下是允许从流中读取堆栈跟踪的代码：

```java
Arrays.stream(Thread.currentThread().getStackTrace())
        .forEach(System.out::println);
```

前一行产生以下输出：

![Java 9 之前的堆栈分析](img/05_20.jpg)

或者，我们可以使用这段代码：

```java
Arrays.stream(new Throwable().getStackTrace())
        .forEach(System.out::println);
```

前面代码的输出以类似的方式显示堆栈跟踪：

![Java 9 之前的堆栈分析](img/05_21.jpg)

例如，如果您想要找到调用者类的完全限定名，可以使用以下方法之一：

```java
new Throwable().getStackTrace()[1].getClassName();

Thread.currentThread().getStackTrace()[2].getClassName();
```

这种编码是可能的，因为`getStackTrace()`方法返回`java.lang.StackTraceElement`类的对象数组，每个对象代表堆栈跟踪中的一个堆栈帧。每个对象都携带着可以通过`getFileName()`、`getClassName()`、`getMethodName()`和`getLineNumber()`方法访问的堆栈跟踪信息。

为了演示它是如何工作的，我们创建了三个类，`Clazz01`、`Clazz02`和`Clazz03`，它们相互调用：

```java
public class Clazz01 {
  public void method(){ new Clazz02().method(); }
}
public class Clazz02 {
  public void method(){ new Clazz03().method(); }
}
public class Clazz03 {
  public void method(){
    Arrays.stream(Thread.currentThread()
                        .getStackTrace()).forEach(ste -> {
      System.out.println();
      System.out.println("ste=" + ste);
      System.out.println("ste.getFileName()=" + 
                                     ste.getFileName());
      System.out.println("ste.getClassName()=" +
                                    ste.getClassName());
      System.out.println("ste.getMethodName()=" + 
                                   ste.getMethodName());
      System.out.println("ste.getLineNumber()=" + 
                                   ste.getLineNumber());
    });
  }
}
```

现在，让我们调用`Clazz01`的`method()`方法：

```java
public class Demo02StackWalking {
    public static void main(String... args) {
        demo_walking();
    }
    private static void demo_walking(){
        new Clazz01().method();
    }
}
```

以下是前面代码打印出的六个堆栈跟踪帧中的两个（第二个和第三个）：

![Java 9 之前的堆栈分析](img/05_22.jpg)

原则上，每个被调用的类都可以访问这些信息。但是要找出哪个类调用了当前类可能并不容易，因为你需要找出哪个帧代表了调用者。此外，为了提供这些信息，JVM 会捕获整个堆栈（除了隐藏的堆栈帧），这可能会影响性能。

这就是引入 JDK 9 中的`java.lang.StackWalker`类、其嵌套的`Option`类和`StackWalker.StackFrame`接口的动机。

## 更好的堆栈遍历方式

`StackWalker`类有四个`getInstance()`静态工厂方法：

+   `getInstance()`: 这返回一个配置为跳过所有隐藏帧和调用者类引用的`StackWalker`类实例

+   `getInstance(StackWalker.Option option)`: 这创建一个具有给定选项的`StackWalker`类实例，指定它可以访问的堆栈帧信息

+   `getInstance(Set<StackWalker.Option> options)`: 这创建一个具有给定选项集的`StackWalker`类实例

+   `getInstance(Set<StackWalker.Option> options, int estimatedDepth)`: 这允许您传入指定估计堆栈帧数量的`estimatedDepth`参数，以便 Java 虚拟机可以分配可能需要的适当缓冲区大小。

作为选项传递的值可以是以下之一：

+   `StackWalker.Option.RETAIN_CLASS_REFERENCE`

+   `StackWalker.Option.SHOW_HIDDEN_FRAMES`

+   `StackWalker.Option.SHOW_REFLECT_FRAMES`

`StackWalker`类的另外三种方法如下：

+   `T walk(Function<Stream<StackWalker.StackFrame>, T> function)`: 这将传入的函数应用于堆栈帧流，第一个帧代表调用`walk()`方法的方法

+   `void forEach(Consumer<StackWalker.StackFrame> action)`: 这对当前线程的流中的每个元素（`StalkWalker.StackFrame`接口类型）执行传入的操作

+   `Class<?> getCallerClass()`: 这获取调用者类的`Class`类对象

正如你所看到的，它允许更加直接的堆栈跟踪分析。让我们使用以下代码修改我们的演示类，并在一行中访问调用者名称：

```java
public class Clazz01 {
  public void method(){ 
    System.out.println("Clazz01 was called by " +
      StackWalker.getInstance(StackWalker
        .Option.RETAIN_CLASS_REFERENCE)
        .getCallerClass().getSimpleName());
    new Clazz02().method(); 
  }
}
public class Clazz02 {
  public void method(){ 
    System.out.println("Clazz02 was called by " +
      StackWalker.getInstance(StackWalker
        .Option.RETAIN_CLASS_REFERENCE)
        .getCallerClass().getSimpleName());
    new Clazz03().method(); 
  }
}
public class Clazz03 {
  public void method(){
    System.out.println("Clazz01 was called by " +
      StackWalker.getInstance(StackWalker
        .Option.RETAIN_CLASS_REFERENCE)
        .getCallerClass().getSimpleName());
  }
}
```

前面的代码将产生以下输出：

![更好的堆栈遍历方式](img/05_23.jpg)

你可以欣赏到这种解决方案的简单性。如果我们需要查看整个堆栈跟踪，我们可以在`Clazz03`的代码中添加以下行：

```java
StackWalker.getInstance().forEach(System.out::println);
```

产生的输出将如下所示：

![更好的堆栈遍历方式](img/05_24.jpg)

再次，只需一行代码，我们就实现了更加可读的输出。我们也可以使用`walk()`方法来实现相同的结果：

```java
StackWalker.getInstance().walk(sf -> { 
  sf.forEach(System.out::println); return null; 
});
```

我们不仅可以打印`StackWalker.StackFrame`，如果需要的话，还可以对其进行更深入的分析，因为它的 API 比`java.lang.StackTraceElement`的 API 更加广泛。让我们运行打印每个堆栈帧及其信息的代码示例：

```java
StackWalker stackWalker = 
   StackWalker.getInstance(Set.of(StackWalker
                   .Option.RETAIN_CLASS_REFERENCE), 10);
stackWalker.forEach(sf -> {
    System.out.println();
    System.out.println("sf="+sf);
    System.out.println("sf.getFileName()=" + 
                                       sf.getFileName());
    System.out.println("sf.getClass()=" + sf.getClass());
    System.out.println("sf.getMethodName()=" + 
                                     sf.getMethodName());
    System.out.println("sf.getLineNumber()=" + 
                                     sf.getLineNumber());
    System.out.println("sf.getByteCodeIndex()=" +
                                  sf.getByteCodeIndex());
    System.out.println("sf.getClassName()=" + 
                                      sf.getClassName());
    System.out.println("sf.getDeclaringClass()=" + 
                                 sf.getDeclaringClass());
    System.out.println("sf.toStackTraceElement()=" +
                               sf.toStackTraceElement());
});
```

前面代码的输出如下：

![更好的堆栈遍历方式](img/05_25.jpg)

注意`StackFrameInfo`类实现了`StackWalker.StackFrame`接口并实际执行了任务。该 API 还允许将其转换回熟悉的`StackTraceElement`对象，以实现向后兼容性，并让那些习惯于它并且不想改变他们的代码和习惯的人享受它。

相比之下，与在内存中生成并存储完整堆栈跟踪（就像传统堆栈跟踪实现的情况）不同，`StackWalker`类只提供了请求的元素。这是它引入的另一个动机，除了演示的使用简单性。有关`StackWalker`类 API 及其用法的更多详细信息，请参阅[`docs.oracle.com/javase/9/docs/api/java/lang/StackWalker.html`](https://docs.oracle.com/javase/9/docs/api/java/lang/StackWalker.html)。

# 集合的便利工厂方法

随着 Java 中函数式编程的引入，对不可变对象的兴趣和需求增加了。传递到方法中的函数可能在与创建它们的上下文大不相同的情况下执行，因此减少意外副作用的可能性使得不可变性的案例更加有力。此外，Java 创建不可修改集合的方式本来就相当冗长，所以这个问题在 Java 9 中得到了解决。以下是在 Java 8 中创建`Set`接口的不可变集合的代码示例：

```java
Set<String> set = new HashSet<>();
set.add("Life");
set.add("is");
set.add("good!");
set = Collections.unmodifiableSet(set); 
```

做了几次之后，作为任何软件专业人员思考的基本重构考虑的一部分，自然而然地会出现对方便方法的需求。在 Java 8 中，前面的代码可以改为以下形式：

```java
Set<String> immutableSet = 
  Collections.unmodifiableSet(new HashSet<>(Arrays
                          .asList("Life", "is", "good!")));
```

或者，如果流是你的朋友，你可以写如下代码：

```java
Set<String> immutableSet = Stream.of("Life","is","good!")
  .collect(Collectors.collectingAndThen(Collectors.toSet(),
                             Collections::unmodifiableSet));
```

前面代码的另一个版本如下：

```java
Set<String> immutableSet =
  Collections.unmodifiableSet(Stream.of("Life","is","good!")
                               .collect(Collectors.toSet()));
```

然而，它比你试图封装的值有更多的样板代码。因此，在 Java 9 中，前面的代码的更短版本成为可能：

```java
Set<String> immutableSet = Set.of("Life","is","good!");
```

类似的工厂被引入来生成`List`接口和`Map`接口的不可变集合：

```java
List<String> immutableList = List.of("Life","is","good!");

Map<Integer,String> immutableMap1 = 
                   Map.of(1, "Life", 2, "is", 3, "good!");

Map<Integer,String> immutableMap2 = 
       Map.ofEntries(entry(1, "Life "), entry(2, "is"), 
                                        entry(3, "good!");

Map.Entry<Integer,String> entry1 = Map.entry(1,"Life");
Map.Entry<Integer,String> entry2 = Map.entry(2,"is");
Map.Entry<Integer,String> entry3 = Map.entry(3,"good!");
Map<Integer,String> immutableMap3 = 
                    Map.ofEntries(entry1, entry2, entry3);
```

## 为什么要使用新的工厂方法？

能够以更紧凑的方式表达相同的功能是非常有帮助的，但这可能不足以成为引入这些新工厂的动机。更重要的是要解决现有的`Collections.unmodifiableList()`、`Collections.unmodifiableSet()`和`Collections.unmodifiableMap()`实现的弱点。虽然使用这些方法创建的集合在尝试修改或添加/删除元素时会抛出`UnsupportedOperationException`类，但它们只是传统可修改集合的包装器，因此可能会受到修改的影响，取决于构造它们的方式。让我们通过示例来说明这一点。另外，现有的不可修改实现的另一个弱点是它不会改变源集合的构造方式，因此`List`、`Set`和`Map`之间的差异--它们可以被构造的方式--仍然存在，这可能是程序员在使用它们时的错误或甚至是挫折的来源。新的工厂方法也解决了这个问题，只使用`of()`工厂方法（以及`Map`的附加`ofEntries()`方法）。话虽如此，让我们回到示例。看一下以下代码片段：

```java
List<String> list = new ArrayList<>();
list.add("unmodifiableList1: Life");
list.add(" is");
list.add(" good! ");
list.add(null);
list.add("\n\n");
List<String> unmodifiableList1 = 
                      Collections.unmodifiableList(list);
//unmodifiableList1.add(" Well..."); //throws exception
//unmodifiableList1.set(2, " sad."); //throws exception
unmodifiableList1.stream().forEach(System.out::print);

list.set(2, " sad. ");
list.set(4, " ");
list.add("Well...\n\n");
unmodifiableList1.stream().forEach(System.out::print);
```

直接修改`unmodifiableList1`的元素会导致`UnsupportedOperationException`。然而，我们可以通过底层的`list`对象来修改它们。如果我们运行前面的示例，输出将如下所示：

![为什么要使用新的工厂方法？](img/05_26.jpg)

即使我们使用`Arrays.asList()`来创建源列表，它也只能保护创建的集合免受添加新元素的影响，而不能防止修改现有元素。以下是一个代码示例：

```java
List<String> list2 = 
           Arrays.asList("unmodifiableList2: Life", 
                        " is", " good! ", null, "\n\n");
List<String> unmodifiableList2 = 
                    Collections.unmodifiableList(list2);
//unmodifiableList2.add(" Well..."); //throws exception
//unmodifiableList2.set(2, " sad."); //throws exception
unmodifiableList2.stream().forEach(System.out::print);

list2.set(2, " sad. ");
//list2.add("Well...\n\n");  //throws exception
unmodifiableList2.stream().forEach(System.out::print);
```

如果我们运行前面的代码，输出将如下所示：

![为什么要使用新的工厂方法？](img/05_27.jpg)

我们还包括了一个`null`元素来演示现有实现如何处理它们，因为相比之下，不可变集合的新工厂不允许包含`null`。另外，它们也不允许在`Set`中包含重复元素（而现有的实现只是忽略它们），但我们将在后面的代码示例中使用新的工厂方法来演示这一方面。

公平地说，使用现有的实现也可以创建`List`接口的真正不可变集合。看一下以下代码：

```java
List<String> immutableList1 =
        Collections.unmodifiableList(new ArrayList<>() {{
            add("immutableList1: Life");
            add(" is");
            add(" good! ");
            add(null);
            add("\n\n");
        }});
//immutableList1.set(2, " sad.");     //throws exception
//immutableList1.add("Well...\n\n");  //throws exception
immutableList1.stream().forEach(System.out::print);
```

创建不可变列表的另一种方法如下：

```java
List<String> immutableList2 =
  Collections.unmodifiableList(Stream
   .of("immutableList2: Life"," is"," good! ",null,"\n\n")
   .collect(Collectors.toList()));
//immutableList2.set(2, " sad.");     //throws exception
//immutableList2.add("Well...\n\n");  //throws exception
immutableList2.stream().forEach(System.out::print);
```

以下是前面代码的变体：

```java
List<String> immutableList3 = 
  Stream.of("immutableList3: Life",
                             " is"," good! ",null,"\n\n")
  .collect(Collectors.collectingAndThen(Collectors.toList(),
                             Collections::unmodifiableList));
//immutableList3.set(2, " sad.");     //throws exception
//immutableList3.add("Well...\n\n");  //throws exception
immutableList3.stream().forEach(System.out::print);
```

如果我们运行前面的三个示例，将看到以下输出：

![为什么要使用新的工厂方法？](img/05_28.jpg)

请注意，尽管我们不能修改这些列表的内容，但我们可以在其中放入`null`。

与先前看到的列表情况相似，`Set`的情况也是如此。以下是显示如何修改不可修改的`Set`接口集合的代码：

```java
Set<String> set = new HashSet<>();
set.add("unmodifiableSet1: Life");
set.add(" is");
set.add(" good! ");
set.add(null);
Set<String> unmodifiableSet1 = 
                       Collections.unmodifiableSet(set);
//unmodifiableSet1.remove(" good! "); //throws exception
//unmodifiableSet1.add("...Well..."); //throws exception
unmodifiableSet1.stream().forEach(System.out::print);
System.out.println("\n");

set.remove(" good! ");
set.add("...Well...");
unmodifiableSet1.stream().forEach(System.out::print);
System.out.println("\n");
```

即使我们将原始集合从数组转换为列表，然后再转换为集合，也可以修改`Set`接口的结果集合，如下所示：

```java
Set<String> set2 = 
   new HashSet<>(Arrays.asList("unmodifiableSet2: Life", 
                                " is", " good! ", null));
Set<String> unmodifiableSet2 = 
                       Collections.unmodifiableSet(set2);
//unmodifiableSet2.remove(" good! "); //throws exception
//unmodifiableSet2.add("...Well..."); //throws exception
unmodifiableSet2.stream().forEach(System.out::print);
System.out.println("\n");

set2.remove(" good! ");
set2.add("...Well...");
unmodifiableSet2.stream().forEach(System.out::print);
System.out.println("\n");
```

运行前两个示例的输出如下：

![为什么要使用新的工厂方法？](img/05_29.jpg)

如果您在 Java 9 中没有使用集合，可能会对输出中集合元素的异常顺序感到惊讶。实际上，这是 JDK 9 中引入的集合和映射的另一个新特性。过去，`Set`和`Map`的实现不能保证保留元素的顺序。但很多时候，顺序是被保留的，一些程序员编写了依赖于此的代码，从而在应用程序中引入了一个令人讨厌的不一致且不易重现的缺陷。新的`Set`和`Map`实现更经常地改变顺序，如果不是在每次运行代码时都改变。这样，它可以在开发的早期暴露潜在的缺陷，并减少其传播到生产环境的机会。

与列表类似，即使不使用 Java 9 的新不可变集合工厂，我们也可以创建不可变集合。其中一种方法如下：

```java
Set<String> immutableSet1 =
     Collections.unmodifiableSet(new HashSet<>() {{
            add("immutableSet1: Life");
            add(" is");
            add(" good! ");
            add(null);
        }});
//immutableSet1.remove(" good! "); //throws exception
//immutableSet1.add("...Well..."); //throws exception
immutableSet1.stream().forEach(System.out::print);
System.out.println("\n");
```

与列表的情况一样，这里还有另一种方法：

```java
Set<String> immutableSet2 =
     Collections.unmodifiableSet(Stream
        .of("immutableSet2: Life"," is"," good! ", null)
                           .collect(Collectors.toSet()));
//immutableSet2.remove(" good!");  //throws exception
//immutableSet2.add("...Well..."); //throws exception
immutableSet2.stream().forEach(System.out::print);
System.out.println("\n");
```

前面代码的另一种变体如下：

```java
Set<String> immutableSet3 = 
  Stream.of("immutableSet3: Life"," is"," good! ", null)
  .collect(Collectors.collectingAndThen(Collectors.toSet(),
                            Collections::unmodifiableSet));
//immutableList5.set(2, "sad.");  //throws exception
//immutableList5.add("Well...");  //throws exception
immutableSet3.stream().forEach(System.out::print);
System.out.println("\n");
```

如果我们运行刚刚介绍的创建`iSet`接口的不可变集合的三个示例，结果将如下：

![为什么要使用新的工厂方法？](img/05_30.jpg)

对于`Map`接口，我们只能想出一种修改`unmodifiableMap`对象的方法：

```java
Map<Integer, String> map = new HashMap<>();
map.put(1, "unmodifiableleMap: Life");
map.put(2, " is");
map.put(3, " good! ");
map.put(4, null);
map.put(5, "\n\n");
Map<Integer, String> unmodifiableleMap = 
                       Collections.unmodifiableMap(map);
//unmodifiableleMap.put(3, " sad.");   //throws exception
//unmodifiableleMap.put(6, "Well..."); //throws exception
unmodifiableleMap.values().stream()
                             .forEach(System.out::print);
map.put(3, " sad. ");
map.put(4, "");
map.put(5, "");
map.put(6, "Well...\n\n");
unmodifiableleMap.values().stream()
                             .forEach(System.out::print);
```

前面代码的输出如下：

![为什么要使用新的工厂方法？](img/05_31.jpg)

我们找到了四种在不使用 Java 9 增强功能的情况下创建`Map`接口的不可变集合的方法。以下是第一个示例：

```java
Map<Integer, String> immutableMap1 =
        Collections.unmodifiableMap(new HashMap<>() {{
            put(1, "immutableMap1: Life");
            put(2, " is");
            put(3, " good! ");
            put(4, null);
            put(5, "\n\n");
        }});
//immutableMap1.put(3, " sad. ");   //throws exception
//immutableMap1.put(6, "Well...");  //throws exception
immutableMap1.values().stream().forEach(System.out::print);
```

第二个示例有点复杂：

```java
String[][] mapping = 
       new String[][] {{"1", "immutableMap2: Life"}, 
                       {"2", " is"}, {"3", " good! "}, 
                          {"4", null}, {"5", "\n\n"}};

Map<Integer, String> immutableMap2 =
  Collections.unmodifiableMap(Arrays.stream(mapping)
    .collect(Collectors.toMap(a -> Integer.valueOf(a[0]), 
                          a -> a[1] == null? "" : a[1])));
immutableMap2.values().stream().forEach(System.out::print);
```

```java
null value in the source array:
```

```java
String[][] mapping = 
    new String[][]{{"1", "immutableMap3: Life"}, 
       {"2", " is"}, {"3", " good! "}, {"4", "\n\n"}};
Map<Integer, String> immutableMap3 =
   Collections.unmodifiableMap(Arrays.stream(mapping)
     .collect(Collectors.toMap(a -> Integer.valueOf(a[0]), 
a -> a[1])));
//immutableMap3.put(3, " sad.");   //throws Exception
//immutableMap3.put(6, "Well..."); //throws exception
immutableMap3.values().stream().forEach(System.out::print);
```

前面代码的另一种变体如下：

```java
mapping[0][1] = "immutableMap4: Life";
Map<Integer, String> immutableMap4 = Arrays.stream(mapping)
           .collect(Collectors.collectingAndThen(Collectors
             .toMap(a -> Integer.valueOf(a[0]), a -> a[1]),
                             Collections::unmodifiableMap));
//immutableMap4.put(3, " sad.");    //throws exception
//immutableMap4.put(6, "Well...");  //throws exception
immutableMap4.values().stream().forEach(System.out::print);
```

在运行了所有四个最后的示例之后，输出如下：

![为什么要使用新的工厂方法？](img/05_32.jpg)

通过对现有集合实现的修订，我们现在可以讨论并欣赏 Java 9 中集合的新工厂方法。

## 在实践中使用新的工厂方法

在重新审视集合创建的现有方法之后，我们现在可以回顾并享受 Java 9 中引入的相关 API。就像在前一节中一样，我们从`List`接口开始。使用新的`List.of()`工厂方法创建不可变列表是多么简单和一致：

```java
List<String> immutableList = 
  List.of("immutableList: Life", 
      " is", " is", " good!\n\n"); //, null);
//immutableList.set(2, "sad.");    //throws exception
//immutableList.add("Well...");    //throws exception
immutableList.stream().forEach(System.out::print);
```

从前面的代码注释中可以看出，新的工厂方法不允许将`null`包括在列表值中。

`immutableSet`的创建看起来类似于这样：

```java
Set<String> immutableSet = 
    Set.of("immutableSet: Life", " is", " good!");
                                      //, " is" , null);
//immutableSet.remove(" good!\n\n");  //throws exception
//immutableSet.add("...Well...\n\n"); //throws exception
immutableSet.stream().forEach(System.out::print);
System.out.println("\n");
```

从前面的代码注释中可以看出，`Set.of()`工厂方法在创建`Set`接口的不可变集合时不允许添加`null`或重复元素。

不可变的`Map`接口集合格式也类似：

```java
Map<Integer, String> immutableMap = 
   Map.of(1</span>, "immutableMap: Life", 2, " is", 3, " good!");
                                    //, 4, null);
//immutableMap.put(3, " sad.");    //throws exception
//immutableMap.put(4, "Well...");  //throws exception
immutableMap.values().stream().forEach(System.out::print);
System.out.println("\n");
```

`Map.of()`方法也不允许值为`null`。`Map.of()`方法的另一个特性是它允许在编译时检查元素类型，这减少了运行时问题的可能性。

对于那些更喜欢更紧凑代码的人，这是另一种表达相同功能的方法：

```java
Map<Integer, String> immutableMap3 = 
            Map.ofEntries(entry(1, "immutableMap3: Life"), 
                      entry(2, " is"), entry(3, " good!"));
immutableMap3.values().stream().forEach(System.out::print);
System.out.println("\n");
```

如果我们运行所有先前使用新工厂方法的示例，输出如下：

![实践中使用新的工厂方法](img/05_33.jpg)

正如我们已经提到的，具有不可变集合的能力，包括空集合，对于函数式编程非常有帮助，因为这个特性确保这样的集合不能作为副作用被修改，也不能引入意外和难以追踪的缺陷。新工厂方法的完整种类包括多达 10 个显式条目，再加上一个具有任意数量元素的条目。对于`List`接口，它看起来是这样的：

```java
static <E> List<E> of()
static <E> List<E> of(E e1)
static <E> List<E> of(E e1, E e2)
static <E> List<E> of(E e1, E e2, E e3)
static <E> List<E> of(E e1, E e2, E e3, E e4)
static <E> List<E> of(E e1, E e2, E e3, E e4, E e5)
static <E> List<E> of(E e1, E e2, E e3, E e4, E e5, E e6)
static <E> List<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7)
static <E> List<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8)
static <E> List<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8, E e9)
static <E> List<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8, E e9, E e10)
static <E> List<E> of(E... elements)
```

`Set`工厂方法看起来类似：

```java
static <E> Set<E> of()
static <E> Set<E> of(E e1)
static <E> Set<E> of(E e1, E e2)
static <E> Set<E> of(E e1, E e2, E e3)
static <E> Set<E> of(E e1, E e2, E e3, E e4)
static <E> Set<E> of(E e1, E e2, E e3, E e4, E e5)
static <E> Set<E> of(E e1, E e2, E e3, E e4, E e5, E e6)
static <E> Set<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7)
static <E> Set<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8)
static <E> Set<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8, E e9)
static <E> Set<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8, E e9, E e10)
static <E> Set<E> of(E... elements)
```

此外，`Map`工厂方法也遵循相同的规则：

```java
static <K,V> Map<K,V> of()
static <K,V> Map<K,V> of(K k1, V v1)
static <K,V> Map<K,V> of(K k1, V v1, K k2, V v2)
static <K,V> Map<K,V> of(K k1, V v1, K k2, V v2, K k3, V v3)
static <K,V> Map<K,V> of(K k1, V v1, K k2, V v2, K k3, V v3, K k4, V v4)
static <K,V> Map<K,V> of(K k1, V v1, K k2, V v2, K k3, V v3, K k4, V v4, K k5, V   v5
static <K,V> Map<K,V> of(K k1, V v1, K k2, V v2, K k3, V v3, K k4, V v4, K k5, V v5, K k6, V v6)
static <K,V> Map<K,V> of(K k1, V v1, K k2, V v2, K k3, V v3, K k4, V v4, K k5, V v5, K k6, V v6, K k7, V v7
static <K,V> Map<K,V> of(K k1, V v1, K k2, V v2, K k3, V v3, K k4, V v4, K k5, V v5, K k6, V v6, K k7, V v7,
K k8, V v8)
static <K,V> Map<K,V> of(K k1, V v1, K k2, V v2, K k3, V v3, K k4, V v4, K k5, V v5, K k6, V v6, K k7, V v7,
K k8, V v8, K k9, V v9)
static <K,V> Map<K,V> of(K k1, V v1, K k2, V v2, K k3, V v3, K k4, V v4, K k5, V v5, K k6, V v6, K k7, V v7,
K k8, V v8, K k9, V v9, K k10, V v10)
static <K,V> Map<K,V> ofEntries(Map.Entry<? extends K,? extends V>... entries
```

决定不为不可变集合添加新接口使它们容易引起偶尔的混淆，当程序员假设可以在它们上调用`add()`或`put()`时。如果没有经过测试，这样的假设会导致抛出`UnsupportedOperationException`的运行时错误。尽管存在这种潜在的陷阱，不可变集合创建的新工厂方法是 Java 中非常有用的补充。

# 支持异步处理的 CompletableFuture

`java.util.concurrent.CompletableFuture<T>`类首次在 Java 8 中引入。它是对`java.util.concurrent.Future<T>`接口的异步调用控制的下一级。它实际上实现了`Future`，以及`java.util.concurrent.CompletionStage<T>`。在 Java 9 中，通过添加新的工厂方法、支持延迟和超时以及改进的子类化，增强了`CompletableFuture`——我们将在接下来的章节中更详细地讨论这些特性。但首先，让我们概述一下`CompletableFuture`的 API。

## `CompletableFuture` API 概述

`CompletableFuture`的 API 包括 70 多个方法，其中 38 个是`CompletionStage`接口的实现，5 个是`Future`的实现。因为`CompletableFuture`类实现了`Future`接口，它可以被视为`Future`，并且不会破坏基于`Future`API 的现有功能。

因此，API 的大部分来自`CompletionStage`。大多数方法返回`CompletableFuture`（在`CompletionStage`接口中，它们返回`CompletionStage`，但在`CompletableFuture`类中实现时会转换为`CompletableFuture`），这意味着它们允许链接操作，类似于`Stream`方法在管道中只有一个元素通过时的操作。每个方法都有一个接受函数的签名。一些方法接受`Function<T,U>`，它将被应用于传入的值`T`并返回结果`U`。其他方法接受`Consumer<T>`，它接受传入的值并返回`void`。还有其他方法接受`Runnable`，它不接受任何输入并返回`void`。以下是其中一组这些方法：

```java
thenRun(Runnable action)
thenApply(Function<T,U> fn)
thenAccept(Consumer<T> action)
```

它们都返回`CompletableFuture`，它携带函数或 void 的结果（在`Runnable`和`Consumer`的情况下）。它们每个都有两个执行相同功能的异步伴侣方法。例如，让我们看一下`thenRun(Runnable action)`方法。以下是它的伴侣们：

+   `thenRunAsync(Runnable action)`方法会在另一个线程中运行操作，使用默认的`ForkJoinPool.commonPool()`线程池。

+   `thenRun(Runnable action, Executor executor)`方法会在传入的参数 executor 作为线程池的另一个线程中运行操作。

因此，我们已经介绍了`CompletionStage`接口的九种方法。

另一组方法包括以下内容：

```java
thenCompose(Function<T,CompletionStage<U>> fn)
applyToEither(CompletionStage other, Function fn)
acceptEither(CompletionStage other, Consumer action)
runAfterBoth(CompletionStage other, Runnable action)
runAfterEither(CompletionStage other, Runnable action)
thenCombine(CompletionStage<U> other, BiFunction<T,U,V> fn)
thenAcceptBoth(CompletionStage other, BiConsumer<T,U> action)
```

这些方法在一个或两个`CompletableFuture`（或`CompletionStage`）对象产生作为输入传递给操作的结果后执行传入的操作。这里的“两个”指的是提供方法的`CompletableFuture`和作为方法参数传入的`CompletableFuture`。从这些方法的名称中，你可以相当可靠地猜测它们的意图。我们将在接下来的示例中演示其中一些。每个这七个方法都有两个用于异步处理的伴侣。这意味着我们已经描述了`CompletionStage`接口的 30 种（共 38 种）方法。

还有一组通常用作终端操作的两种方法，因为它们可以处理前一个方法的结果（作为`T`传入）或异常（作为`Throwable`传入）：

```java
handle(BiFunction<T,Throwable,U> fn)
whenComplete(BiConsumer<T,Throwable> action)
```

我们稍后将看到这些方法的使用示例。当链中的方法抛出异常时，所有其余的链接方法都会被跳过，直到遇到第一个`handle()`方法或`whenComplete()`方法。如果链中没有这两个方法中的任何一个，那么异常将像其他 Java 异常一样冒泡。这两个方法也有异步伴侣，这意味着我们已经讨论了`CompletionStage`接口的 36 种（共 38 种）方法。

还有一个仅处理异常的方法（类似于传统编程中的 catch 块）：

```java
exceptionally(Function<Throwable,T> fn)
```

这个方法没有异步伴侣，就像最后剩下的方法一样。

```java
toCompletableFuture()
```

它只返回一个具有与此阶段相同属性的`CompletableFuture`对象。有了这个，我们已经描述了`CompletionStage`接口的所有 38 种方法。

`CompletableFuture`类中还有大约 30 种不属于任何实现接口的方法。其中一些在异步执行提供的函数后返回`CompletableFuture`对象：

```java
runAsync(Runnable runnable)
runAsync(Runnable runnable, Executor executor)
supplyAsync(Supplier<U> supplier)
supplyAsync(Supplier<U> supplier, Executor executor)
```

其他人并行执行几个`CompletableFuture`对象：

```java
allOf(CompletableFuture<?>... cfs)
anyOf(CompletableFuture<?>... cfs)
```

还有一组生成已完成 future 的方法，因此返回的`CompletableFuture`对象上的`get()`方法将不再阻塞：

```java
complete(T value)
completedStage(U value)
completedFuture(U value)
failedStage(Throwable ex)
failedFuture(Throwable ex)
completeAsync(Supplier<T> supplier)
completeExceptionally(Throwable ex)
completeAsync(Supplier<T> supplier, Executor executor)
completeOnTimeout(T value, long timeout, TimeUnit unit)
```

其余的方法执行各种其他有用的功能：

```java
join()
defaultExecutor()
newIncompleteFuture()
getNow(T valueIfAbsent)
getNumberOfDependents()
minimalCompletionStage()
isCompletedExceptionally()
obtrudeValue(T value)
obtrudeException(Throwable ex)
orTimeout(long timeout, TimeUnit unit)
delayedExecutor(long delay, TimeUnit unit)
```

请参考官方 Oracle 文档，其中描述了`CompletableFuture` API 的这些和其他方法，网址为[`download.java.net/java/jdk9/docs/api/index.html?java/util/concurrent/CompletableFuture.html`](http://download.java.net/java/jdk9/docs/api/index.html?java/util/concurrent/CompletableFuture.html)。

## Java 9 中的`CompletableFuture` API 增强

Java 9 为`CompletableFuture`引入了几项增强功能：

+   `CompletionStage<U> failedStage(Throwable ex)`工厂方法返回使用给定异常完成的`CompletionStage`对象

+   `CompletableFuture<U> failedFuture(Throwable ex)`工厂方法返回使用给定异常完成的`CompletableFuture`对象

+   新的`CompletionStage<U> completedStage(U value)`工厂方法返回使用给定`U`值完成的`CompletionStage`对象

+   `CompletableFuture<T> completeOnTimeout(T value, long timeout, TimeUnit unit)`如果在给定的超时之前未完成，则使用给定的`T`值完成`CompletableFuture`任务

+   `CompletableFuture<T> orTimeout(long timeout, TimeUnit unit)`如果在给定的超时之前未完成，则使用`java.util.concurrent.TimeoutException`完成`CompletableFuture`

+   现在可以重写`defaultExecutor()`方法以支持另一个默认执行程序

+   一个新方法`newIncompleteFuture()`使得子类化`CompletableFuture`类更容易

## 问题和解决方案使用 Future

为了演示和欣赏`CompletableFuture`的强大功能，让我们从使用`Future`解决的问题开始，然后看看使用`CompletableFuture`可以更有效地解决多少。假设我们的任务是对由四个阶段组成的建筑进行建模：

+   收集地基、墙壁和屋顶的材料

+   安装地基

+   竖起墙壁

+   搭建和完成屋顶

在传统的单线程顺序编程中，模型如下：

```java
StopWatch stopWatch = new StopWatch();
Stage failedStage;
String SUCCESS = "Success";

stopWatch.start();
String result11 = doStage(Stage.FoundationMaterials);
String result12 = doStage(Stage.Foundation, result11);
String result21 = doStage(Stage.WallsMaterials);
String result22 = doStage(Stage.Walls, 
                       getResult(result21, result12));
String result31 = doStage(Stage.RoofMaterials);
String result32 = doStage(Stage.Roof, 
                       getResult(result31, result22));
System.out.println("House was" + 
       (isSuccess(result32)?"":" not") + " built in " 
                + stopWatch.getTime()/1000\. + " sec");
```

这里，`Stage`是一个枚举：

```java
enum Stage {
    FoundationMaterials,
    WallsMaterials,
    RoofMaterials,
    Foundation,
    Walls,
    Roof
}
```

`doStage()`方法有两个重载版本。这是第一个版本：

```java
String doStage(Stage stage) {
    String result = SUCCESS;
    boolean failed = stage.equals(failedStage);
    if (failed) {
        sleepSec(2);
        result = stage + " were not collected";
        System.out.println(result);
    } else {
        sleepSec(1);
        System.out.println(stage + " are ready");
    }
    return result;
}
```

第二个版本如下：

```java
String doStage(Stage stage, String previousStageResult) {
  String result = SUCCESS;
  boolean failed = stage.equals(failedStage);
  if (isSuccess(previousStageResult)) {
    if (failed) {
      sleepSec(2);
      result = stage + " stage was not completed";
      System.out.println(result);
    } else {
      sleepSec(1);
      System.out.println(stage + " stage is completed");
    }
  } else {
      result = stage + " stage was not started because: " 
                                    + previousStageResult;
      System.out.println(result);
  }
  return result;
}
```

`sleepSec()`、`isSuccess()`和`getResult()`方法如下：

```java
private static void sleepSec(int sec) {
    try {
        TimeUnit.SECONDS.sleep(sec);
    } catch (InterruptedException e) {
    }
}
boolean isSuccess(String result) {
    return SUCCESS.equals(result);
}
String getResult(String result1, String result2) {
    if (isSuccess(result1)) {
        if (isSuccess(result2)) {
            return SUCCESS;
        } else {
            return result2;
        }
    } else {
        return result1;
    }
}
```

成功的房屋建造（如果我们运行之前的代码而没有为`failedStage`变量分配任何值）如下所示：

![使用 Future 的问题和解决方案](img/05_34.jpg)

如果我们设置`failedStage=Stage.Walls`，结果将如下：

![使用 Future 的问题和解决方案](img/05_35.jpg)

使用`Future`，我们可以缩短建造房屋所需的时间：

```java
ExecutorService execService = Executors.newCachedThreadPool();
Callable<String> t11 = 
                     () -> doStage(Stage.FoundationMaterials);
Future<String> f11 = execService.submit(t11);
List<Future<String>> futures = new ArrayList<>();
futures.add(f11);

Callable<String> t21 = () -> doStage(Stage.WallsMaterials);
Future<String> f21 = execService.submit(t21);
futures.add(f21);

Callable<String> t31 = () -> doStage(Stage.RoofMaterials);
Future<String> f31 = execService.submit(t31);
futures.add(f31);

String result1 = getSuccessOrFirstFailure(futures);

String result2 = doStage(Stage.Foundation, result1);
String result3 = 
       doStage(Stage.Walls, getResult(result1, result2));
String result4 = 
        doStage(Stage.Roof, getResult(result1, result3));
```

这里，`getSuccessOrFirstFailure()`方法如下：

```java
String getSuccessOrFirstFailure(
                      List<Future<String>> futures) {
    String result = "";
    int count = 0;
    try {
        while (count < futures.size()) {
            for (Future<String> future : futures) {
                if (future.isDone()) {
                    result = getResult(future);
                    if (!isSuccess(result)) {
                        break;
                    }
                    count++;
                } else {
                    sleepSec(1);
                }
            }
            if (!isSuccess(result)) {
                break;
            }
        }
    } catch (Exception ex) {
        ex.printStackTrace();
    }
    return result;
}
```

现在成功建造房屋的速度更快，因为材料收集是并行进行的：

![使用 Future 的问题和解决方案](img/05_36.jpg)

通过利用 Java 函数式编程，我们可以将实现的后半部分改为以下内容：

```java
Supplier<String> supplier1 = 
                 () -> doStage(Stage.Foundation, result1);
Supplier<String> supplier2 = 
                () -> getResult(result1, supplier1.get());
Supplier<String> supplier3 = 
              () -> doStage(Stage.Walls, supplier2.get());
Supplier<String> supplier4 = 
                () -> getResult(result1, supplier3.get());
Supplier<String> supplier5 = 
               () -> doStage(Stage.Roof, supplier4.get());
System.out.println("House was" + 
              (isSuccess(supplier5.get()) ? "" : " not") + 
      " built in " + stopWatch.getTime() / 1000\. + " sec");
```

前面嵌套函数的链由最后一行的`supplier5.get()`触发。它会阻塞，直到所有函数按顺序完成，因此没有性能改进：

![使用 Future 的问题和解决方案](img/05_38.jpg)

这就是我们可以使用`Future`的地方。现在让我们看看是否可以使用`CompletableFuture`改进之前的代码。

## 使用 CompletableFuture 的解决方案

以下是我们如何使用`CompletableFuture` API 链接相同操作的方式：

```java
stopWatch.start();
ExecutorService pool = Executors.newCachedThreadPool();
CompletableFuture<String> cf1 =
   CompletableFuture.supplyAsync(() -> 
           doStageEx(Stage.FoundationMaterials), pool);
CompletableFuture<String> cf2 =
   CompletableFuture.supplyAsync(() -> 
                doStageEx(Stage.WallsMaterials), pool);
CompletableFuture<String> cf3 =
   CompletableFuture.supplyAsync(() -> 
                 doStageEx(Stage.RoofMaterials), pool);
CompletableFuture.allOf(cf1, cf2, cf3)
  .thenComposeAsync(result -> 
      CompletableFuture.supplyAsync(() -> SUCCESS), pool)
  .thenApplyAsync(result -> 
                 doStage(Stage.Foundation, result), pool)
  .thenApplyAsync(result -> 
                      doStage(Stage.Walls, result), pool)
  .thenApplyAsync(result -> 
                       doStage(Stage.Roof, result), pool)
  .handleAsync((result, ex) -> {
       System.out.println("House was" +
         (isSuccess(result) ? "" : " not") + " built in " 
                 + stopWatch.getTime() / 1000\. + " sec");
       if (result == null) {
         System.out.println("Because: " + ex.getMessage());
         return ex.getMessage();
       } else {
         return result;
       }
  }, pool);
System.out.println("Out!!!!!");
```

为了使其工作，我们不得不将`doStage()`中的一个实现更改为`doStageEx()`方法：

```java
String doStageEx(Stage stage) {
  boolean failed = stage.equals(failedStage);
  if (failed) {
    sleepSec(2);
    throw new RuntimeException(stage + 
                          " stage was not completed");
  } else {
    sleepSec(1);
    System.out.println(stage + " stage is completed");
  }
  return SUCCESS;
}
```

```java
Out!!!!!) came out first, which means that all the chains of the operations related to building the house were executed asynchronously
```

现在，让我们看看系统在收集材料的第一个阶段失败时的行为（`failedStage = Stage.WallsMaterials`）：

![使用 CompletableFuture 的解决方案](img/05_39.jpg)

异常由`WallsMaterials`阶段抛出，并被`handleAsync()`方法捕获，正如预期的那样。而且，在打印`Out!!!!!`消息后，处理是异步进行的。

## CompletableFuture 的其他有用功能

`CompletableFuture`的一个巨大优势是它可以作为对象传递并多次使用，以启动不同的操作链。为了演示这种能力，让我们创建几个新操作：

```java
String getData() {
  System.out.println("Getting data from some source...");
  sleepSec(1);
  return "Some input";
}
SomeClass doSomething(String input) {
  System.out.println(
    "Doing something and returning SomeClass object...");
  sleepSec(1);
  return new SomeClass();
}
AnotherClass doMore(SomeClass input) {
  System.out.println("Doing more of something and " +
                    "returning AnotherClass object...");
  sleepSec(1);
  return new AnotherClass();
}
YetAnotherClass doSomethingElse(AnotherClass input) {
  System.out.println("Doing something else and " +
                "returning YetAnotherClass object...");
  sleepSec(1);
  return new YetAnotherClass();
}
int doFinalProcessing(YetAnotherClass input) {
  System.out.println("Processing and finally " +
                                "returning result...");
  sleepSec(1);
  return 42;
}
AnotherType doSomethingAlternative(SomeClass input) {
  System.out.println("Doing something alternative " +
               "and returning AnotherType object...");
  sleepSec(1);
  return new AnotherType();
}
YetAnotherType doMoreAltProcessing(AnotherType input) {
  System.out.println("Doing more alternative and " +
                  "returning YetAnotherType object...");
  sleepSec(1);
  return new YetAnotherType();
}
int doFinalAltProcessing(YetAnotherType input) {
  System.out.println("Alternative processing and " +
                         "finally returning result...");
  sleepSec(1);
  return 43;
}
```

这些操作的结果将由`myHandler()`方法处理：

```java
int myHandler(Integer result, Throwable ex) {
    System.out.println("And the answer is " + result);
    if (result == null) {
        System.out.println("Because: " + ex.getMessage());
        return -1;
    } else {
        return result;
    }
}
```

注意所有操作返回的不同类型。现在我们可以构建一个在某个点分叉的链：

```java
ExecutorService pool = Executors.newCachedThreadPool();
CompletableFuture<SomeClass> completableFuture =
   CompletableFuture.supplyAsync(() -> getData(), pool)
     .thenApplyAsync(result -> doSomething(result), pool);

completableFuture
   .thenApplyAsync(result -> doMore(result), pool)
   .thenApplyAsync(result -> doSomethingElse(result), pool)
   .thenApplyAsync(result -> doFinalProcessing(result), pool)
   .handleAsync((result, ex) -> myHandler(result, ex), pool);

completableFuture
   .thenApplyAsync(result -> doSomethingAlternative(result), pool)
   .thenApplyAsync(result -> doMoreAltProcessing(result), pool)
   .thenApplyAsync(result -> doFinalAltProcessing(result), pool)
   .handleAsync((result, ex) -> myHandler(result, ex), pool);

System.out.println("Out!!!!!");
```

这个例子的结果如下：

![CompletableFuture 的其他有用功能](img/05_40.jpg)

`CompletableFuture` API 提供了一个非常丰富和经过深思熟虑的 API，支持最新的反应式微服务趋势，因为它允许完全异步地处理数据，根据需要拆分流，并扩展以适应输入的增加。我们鼓励您学习示例（本书附带的代码提供了更多示例），并查看 API：[`download.java.net/java/jdk9/docs/api/index.html?java/util/concurrent/CompletableFuture.html`](http://download.java.net/java/jdk9/docs/api/index.html?java/util/concurrent/CompletableFuture.html)。

# Stream API 改进

Java 9 中大多数新的`Stream` API 功能已经在描述`Stream`过滤的部分中进行了演示。为了提醒您，以下是我们基于 JDK 9 中`Stream` API 改进所演示的示例：

```java
long c1 = senators.stream()
        .flatMap(s -> Stream.ofNullable(s.getParty() 
                              == "Party1" ? s : null))
        .count();
System.out.println("OfNullable: Members of Party1: " + c1);

long c2 = senators.stream()
        .map(s -> s.getParty() == "Party2" ? Optional.of(s) 
                                        : Optional.empty())
        .flatMap(Optional::stream)
        .count();
System.out.println("Optional.stream(): Members of Party2: "
                                                      + c2);

senators.stream().limit(5)
        .takeWhile(s -> Senate.timesVotedYes(s) < 5)
        .forEach(s -> System.out.println("takeWhile(<5): " 
                     + s + ": " + Senate.timesVotedYes(s)));

senators.stream().limit(5)
         .dropWhile(s -> Senate.timesVotedYes(s) < 5)
        .forEach(s -> System.out.println("dropWhile(<5): " 
                     + s + ": " + Senate.timesVotedYes(s)));
```

我们还没有提到的是新的重载`iterate()`方法：

```java
static <T> Stream<T> iterate(T seed, UnaryOperator<T> f)
```

其用法示例如下：

```java
String result = 
    IntStream.iterate(1, i -> i + 2)
             .limit(5)
             .mapToObj(i -> String.valueOf(i))
             .collect(Collectors.joining(", "));
System.out.println("Iterate: " + result);
```

我们不得不添加`limit(5)`，因为`iterate()`方法的这个版本创建了一个无限的整数流。前面代码的结果如下：

![Stream API 改进](img/05_41.jpg)

在 Java 9 中，新增了一个重载的`iterate()`方法：

```java
static <T> Stream<T> iterate(T seed, 
     Predicate<? super T> hasNext, UnaryOperator<T> next)
```

正如您所看到的，现在它作为参数具有`Predicate`函数接口，允许根据需要限制流。例如，以下代码产生的结果与先前的`limit(5)`示例完全相同：

```java
String result = 
   IntStream.iterate(1, i -> i < 11, i -> i + 2)
            .mapToObj(i -> String.valueOf(i))
            .collect(Collectors.joining(", "));
System.out.println("Iterate: " + result);
```

请注意，流元素的类型不需要是整数。它可以是源产生的任何类型。因此，新的`iterate()`方法可以用于提供任何类型数据流终止的条件。

# 总结

在这节课中，我们涵盖了 Java 9 引入的新功能领域。首先，我们从基本的`filter()`方法开始，介绍了许多流过滤的方法，并最终使用了 JDK 9 的`Stream` API 新增功能。然后，您学会了使用新的`StackWalker`类来分析堆栈跟踪的更好方法。讨论通过具体示例进行了说明，帮助您看到真正的工作代码。

我们在为创建不可变集合的新便利工厂方法和与`CompletableFuture`类及其 JDK 9 中的增强一起提供的新异步处理功能时使用了相同的方法。

我们通过列举`Stream` API 的改进来结束了这节课--这些改进我们在过滤代码示例和新的`iterate()`方法中已经演示过。

通过这些内容，我们结束了这本书。现在你可以尝试将你学到的技巧和技术应用到你的项目中，或者如果不适合，可以为高性能构建自己的 Java 项目。在这个过程中，尝试解决真实的问题。这将迫使你学习新的技能和框架，而不仅仅是应用你已经掌握的知识，尽管后者也很有帮助--它可以保持你的知识更新和实用。

学习的最佳方式是自己动手。随着 Java 的不断改进和扩展，敬请期待 Packt 出版的本书及类似书籍的新版本。

# 评估

1.  在 Java 8 中引入了 _______ 接口，用于发出元素并支持执行基于流元素的各种操作的计算。

1.  `StackWalker`类的以下哪个工厂方法创建具有指定堆栈帧信息访问权限的`StackWalker`类实例？

1.  `getInstance()`

1.  `getInstance(StackWalker.Option option)`

1.  `getInstance(Set<StackWalker.Option> options)`

1.  `getInstance(Set<StackWalker.Option> options, int estimatedDepth)`

1.  判断是 True 还是 False：`CompletableFuture` API 由许多方法组成，这些方法是`CompletionStage`接口的实现，并且是`Future`的实现。 

1.  以下哪种方法用于在流中需要进行过滤类型以跳过所有重复元素并仅选择唯一元素。

1.  `distinct()`

1.  `unique()`

1.  `selectall()`

1.  `filtertype()`

1.  判断是 True 还是 False：`CompletableFuture`的一个巨大优势是它可以作为对象传递并多次使用，以启动不同的操作链。
