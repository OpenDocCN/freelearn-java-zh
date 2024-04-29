# 第一章：响应式编程简介

如今，“响应式编程”这个术语正处于流行之中。各种编程语言中都出现了库和框架。有关响应式编程的博客文章、文章和演示正在被创建。Facebook、SoundCloud、Microsoft 和 Netflix 等大公司正在支持和使用这个概念。因此，我们作为程序员开始思考。为什么人们对响应式编程如此兴奋？成为响应式意味着什么？它对我们的项目有帮助吗？我们应该学习如何使用它吗？

与此同时，Java 以其多线程、速度、可靠性和良好的可移植性而备受欢迎。它用于构建各种应用程序，从搜索引擎、数据库到在服务器集群上运行的复杂 Web 应用程序。但 Java 也有不好的声誉——仅使用内置工具编写并发和简单应用程序非常困难，而且在 Java 中编程需要编写大量样板代码。此外，如果需要是异步的（例如使用 futures），你很容易陷入“回调地狱”，这实际上对所有编程语言都成立。

换句话说，Java 很强大，你可以用它创建出色的应用程序，但这并不容易。好消息是，有一种方法可以改变这种情况，那就是使用响应式编程风格。

本书将介绍**RxJava**（[`github.com/ReactiveX/RxJava`](https://github.com/ReactiveX/RxJava)），这是响应式编程范式的开源 Java 实现。使用 RxJava 编写代码需要一种不同的思维方式，但它将使您能够使用简单的结构化代码片段创建复杂的逻辑。

在本章中，我们将涵盖：

+   响应式编程是什么

+   学习和使用这种编程风格的原因

+   设置 RxJava 并将其与熟悉的模式和结构进行比较

+   使用 RxJava 的一个简单例子

# 什么是响应式编程？

响应式编程是围绕变化的传播而展开的一种范式。换句话说，如果一个程序将修改其数据的所有变化传播给所有感兴趣的方，那么这个程序就可以被称为**响应式**。

微软 Excel 就是一个简单的例子。如果在单元格 A1 中设置一个数字，在单元格'B1'中设置另一个数字，并将单元格'C1'设置为`SUM(A1, B1)`；每当'A1'或'B1'发生变化时，'C1'将被更新为它们的和。

让我们称之为**响应式求和**。

将简单变量*c*分配为*a*和*b*变量的和与响应式求和方法之间有什么区别？

在普通的 Java 程序中，当我们改变'a'或'b'时，我们必须自己更新'c'。换句话说，由'a'和'b'表示的数据流的变化不会传播到'c'。下面通过源代码进行了说明：

```java
int a = 4;
int b = 5;
int c = a + b;
System.out.println(c); // 9

a = 6;
System.out.println(c);
// 9 again, but if 'c' was tracking the changes of 'a' and 'b',
// it would've been 6 + 5 = 11
```

### 提示

**下载示例代码**

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的帐户中下载您购买的所有 Packt 图书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便直接将文件发送到您的电子邮件。

这是对“响应式”意味着什么的非常简单的解释。当然，这个想法有各种实现，也有各种问题需要这些实现来解决。

# 为什么我们应该是响应式的？

我们回答这个问题最简单的方法是考虑我们在构建应用程序时的需求。

10-15 年前，网站经过维护或响应时间缓慢是正常的，但今天一切都应该 24/7 在线，并且应该以闪电般的速度响应；如果慢或宕机，用户会选择另一个服务。今天慢意味着无法使用或损坏。我们正在处理更大量的数据，需要快速提供和处理。

HTTP 故障在最近过去并不罕见，但现在，我们必须具有容错能力，并为用户提供可读和合理的消息更新。

过去，我们编写简单的桌面应用程序，但今天我们编写应该快速响应的 Web 应用程序。在大多数情况下，这些应用程序与大量远程服务进行通信。

这些是我们必须满足的新要求，如果我们希望我们的软件具有竞争力。换句话说，我们必须是：

+   模块化/动态：这样，我们将能够拥有 24/7 系统，因为模块可以下线并上线，而不会破坏或停止整个系统。此外，这有助于我们更好地构建随着规模扩大而管理其代码库的应用程序。

+   可扩展性：这样，我们将能够处理大量数据或大量用户请求。

+   容错：这样，系统将对其用户显示稳定。

+   响应性：这意味着快速和可用。

让我们考虑如何实现这一点：

+   如果我们的系统是*事件驱动*，我们可以变得模块化。我们可以将系统分解为多个微服务/组件/模块，它们将使用通知相互通信。这样，我们将对系统的数据流做出反应，这些数据流由通知表示。

+   可扩展意味着对不断增长的数据做出反应，对负载做出反应而不会崩溃。

+   对故障/错误的反应将使系统更具容错能力。

+   响应性意味着及时对用户活动做出反应。

如果应用程序是事件驱动的，它可以分解为多个自包含组件。这有助于我们变得更具可扩展性，因为我们可以随时添加新组件或删除旧组件，而不会停止或破坏系统。如果错误和故障传递到正确的组件，它可以将它们处理为通知，应用程序可以变得更具容错能力或弹性。因此，如果我们构建我们的系统为事件驱动，我们可以更容易地实现可扩展性和故障容忍性，而且可扩展、解耦和防错的应用程序对用户快速响应。

![为什么我们应该是反应式的？](img/4305_01_01.jpg)

**Reactive Manifesto**（[`www.reactivemanifesto.org/`](http://www.reactivemanifesto.org/)）是一份文件，定义了我们之前提到的四个反应原则。每个反应系统都应该是消息驱动的（事件驱动）。这样，它可以变得松散耦合，因此可扩展和具有弹性（容错），这意味着它是可靠和响应的（请参见上图）。

请注意，Reactive Manifesto 描述了一个反应式系统，并不同于我们对反应式编程的定义。您可以构建一个消息驱动、具有弹性、可扩展和响应的应用程序，而无需使用反应式库或语言。

应用程序数据的更改可以使用通知进行建模，并且可以传播到正确的处理程序。因此，使用反应式编程编写应用程序是遵守宣言的最简单方法。

# 介绍 RxJava

要编写响应式程序，我们需要一个库或特定的编程语言，因为自己构建这样的东西是相当困难的任务。Java 并不是一个真正的响应式编程语言（它提供了一些工具，比如`java.util.Observable`类，但它们相当有限）。它是一种静态类型的面向对象的语言，我们需要编写大量样板代码来完成简单的事情（例如 POJOs）。但是在 Java 中有一些我们可以使用的响应式库。在这本书中，我们将使用 RxJava（由 Java 开源社区的人员开发，由 Netflix 指导）。

## 下载和设置 RxJava

你可以从 Github（[`github.com/ReactiveX/RxJava`](https://github.com/ReactiveX/RxJava)）下载并构建 RxJava。它不需要任何依赖，并支持 Java 8 的 lambda。它的 Javadoc 和 GitHub 维基页面提供的文档结构良好，是最好的之一。以下是如何查看项目并运行构建：

```java
$ git clone git@github.com:ReactiveX/RxJava.git
$ cd RxJava/
$ ./gradlew build
```

当然，你也可以下载预构建的 JAR。在这本书中，我们将使用 1.0.8 版本。

如果你使用 Maven，你可以将 RxJava 作为依赖项添加到你的`pom.xml`文件中：

```java
<dependency>
  <groupId>io.reactivex</groupId>
  <artifactId>rxjava</artifactId>
  <version>1.0.8</version>
</dependency>
```

或者，对于 Apache Ivy，将这个片段放入你的 Ivy 文件的依赖项中：

```java
<dependency org="io.reactivex" name="rxjava" rev="1.0.8" />
```

如果你使用 Gradle，你可以更新你的`build.gradle`文件的依赖项如下：

```java
dependencies {
  ...
  compile 'io.reactivex:rxjava:1.0.8'
  ...
}
```

### 注意

本书附带的代码示例和程序可以使用 Gradle 构建和测试。它可以从这个 Github 仓库下载：[`github.com/meddle0x53/learning-rxjava`](https://github.com/meddle0x53/learning-rxjava)。

现在，让我们来看看 RxJava 到底是什么。我们将从一些众所周知的东西开始，逐渐深入到这个库的秘密中。

## 比较迭代器模式和 RxJava Observable

作为 Java 程序员，你很可能听说过或使用过“迭代器”模式。这个想法很简单：一个“迭代器”实例用于遍历容器（集合/数据源/生成器），在需要时逐个拉取容器的元素，直到达到容器的末尾。以下是在 Java 中如何使用它的一个小例子：

```java
List<String> list = Arrays.asList("One", "Two", "Three", "Four", "Five"); // (1)

Iterator<String> iterator = list.iterator(); // (2)

while(iterator.hasNext()) { // 3
  // Prints elements (4)
  System.out.println(iterator.next());
}
```

每个`java.util.Collection`对象都是一个`Iterable`实例，这意味着它有`iterator()`方法。这个方法创建一个`Iterator`实例，它的源是集合。让我们看看前面的代码做了什么：

1.  我们创建一个包含五个字符串的新`List`实例。

1.  我们使用`iterator()`方法从这个`List`实例创建一个`Iterator`实例。

1.  `Iterator`接口有两个重要的方法：`hasNext()`和`next()`。`hasNext()`方法用于检查`Iterator`实例是否有更多元素可遍历。在这里，我们还没有开始遍历元素，所以它将返回`True`。当我们遍历这五个字符串时，它将返回`False`，程序将在`while`循环之后继续进行。

1.  前五次调用`Iterator`实例的`next()`方法时，它将按照它们在集合中插入的顺序返回元素。所以字符串将被打印出来。

在这个例子中，我们的程序使用`Iterator`实例从`List`实例中消耗项目。它拉取数据（这里用字符串表示），当前线程会阻塞，直到请求的数据准备好并接收到。所以，例如，如果`Iterator`实例在每次`next()`方法调用时向 web 服务器发送请求，我们程序的主线程将在等待每个响应到达时被阻塞。

RxJava 的构建块是可观察对象。`Observable`类（请注意，这不是 JDK 中附带的`java.util.Observable`类）是`Iterator`类的数学对偶，这基本上意味着它们就像同一枚硬币的两面。它具有产生值的基础集合或计算，可以被消费者消耗。但不同之处在于，消费者不像`Iterator`模式中那样从生产者“拉”这些值。恰恰相反；生产者通过通知将值“推送”给消费者。

这是相同程序的示例，但使用`Observable`实例编写：

```java
List<String> list = Arrays.asList("One", "Two", "Three", "Four", "Five"); // (1)

Observable<String> observable = Observable.from(list); // (2)

observable.subscribe(new Action1<String>() { // (3)
  @Override
  public void call(String element) {
    System.out.println(element); // Prints the element (4)
  }
});
```

以下是代码中发生的情况：

1.  我们以与上一个示例相同的方式创建字符串列表。

1.  然后，我们从列表中创建一个`Observable`实例，使用`from(Iterable<? extends T> iterable)`方法。此方法用于创建`Observable`的实例，它们将所有值同步地从`Iterable`实例（在我们的例子中是列表）逐个发送给它们的订阅者（消费者）。我们将在第三章中看看如何逐个将值发送给订阅者，*创建和连接 Observable、Observer 和 Subject*。

1.  在这里，我们可以订阅`Observable`实例。通过订阅，我们告诉 RxJava 我们对这个`Observable`实例感兴趣，并希望从中接收通知。我们使用实现`Action1`接口的匿名类进行订阅，通过定义一个单一方法`call(T)`。这个方法将由`Observable`实例每次有值准备推送时调用。始终创建新的`Action1`实例可能会显得太啰嗦，但 Java 8 解决了这种冗长。我们将在第二章中了解更多信息，*使用 Java 8 的函数构造*。

1.  因此，源列表中的每个字符串都将通过`call()`方法推送，并将被打印出来。

RxJava 的`Observable`类的实例行为有点像异步迭代器，它们自己通知其订阅者/消费者有下一个值。事实上，`Observable`类在经典的`Observer`模式（在 Java 中实现——参见`java.util.Observable`，参见《设计模式：可复用面向对象软件的元素》）中添加了`Iterable`类型中的两个可用的东西。

+   向消费者发出没有更多数据可用的信号的能力。我们可以附加一个订阅者来监听“`OnCompleted`”通知，而不是调用`hasNext()`方法。

+   信号订阅者发生错误的能力。我们可以将错误侦听器附加到`Observable`实例，而不是尝试捕获错误。

这些侦听器可以使用`subscribe(Action1<? super T>, Action1 <Throwable>, Action0)`方法附加。让我们通过添加错误和完成侦听器来扩展`Observable`实例示例：

```java
List<String> list = Arrays.asList("One", "Two", "Three", "Four", "Five");

Observable<String> observable = Observable.from(list);
observable.subscribe(new Action1<String>() {
  @Override
  public void call(String element) {
    System.out.println(element);
  }
},
new Action1<Throwable>() {
 @Override
 public void call(Throwable t) {
 System.err.println(t); // (1)
 }
},
new Action0() {
 @Override
 public void call() {
 System.out.println("We've finnished!"); // (2)
 }
});
```

新的东西在这里是：

1.  如果在处理元素时出现错误，`Observable`实例将通过此侦听器的`call(Throwable)`方法发送此错误。这类似于`Iterator`实例示例中的 try-catch 块。

1.  当一切都完成时，`Observable`实例将调用此`call()`方法。这类似于使用`hasNext()`方法来查看`Iterable`实例的遍历是否已经完成并打印“We've finished!”。

### 注意

此示例可在 GitHub 上查看，并可在[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter01/ObservableVSIterator.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter01/ObservableVSIterator.java)上查看/下载。

我们看到了如何使用`Observable`实例，它们与我们熟悉的`Iterator`实例并没有太大的不同。这些`Observable`实例可以用于构建异步流，并将数据更新推送给它们的订阅者（它们可以有多个订阅者）。这是反应式编程范式的一种实现。数据被传播给所有感兴趣的方，即订阅者。

使用这样的流进行编码是反应式编程的更类似函数式的实现。当然，对此有正式的定义和复杂的术语，但这是最简单的解释。

订阅事件应该是熟悉的；例如，在 GUI 应用程序中点击按钮会触发一个事件，该事件会传播给订阅者—处理程序。但是，使用 RxJava，我们可以从任何地方创建数据流—文件输入、套接字、响应、变量、缓存、用户输入等等。此外，消费者可以被通知流已关闭，或者发生了错误。因此，通过使用这些流，我们的应用程序可以对失败做出反应。

总之，流是一系列持续的消息/事件，按照它们在实时处理中的顺序排序。它可以被看作是随着时间变化的值，这些变化可以被依赖它的订阅者（消费者）观察到。因此，回到 Excel 的例子，我们实际上用"反应式变量"或 RxJava 的`Observable`实例有效地替换了传统变量。

## 实现反应式求和

现在我们熟悉了`Observable`类和如何以反应式方式使用它编码的想法，我们准备实现在本章开头提到的反应式求和。

让我们看看我们的程序必须满足的要求：

+   它将是一个在终端中运行的应用程序。

+   一旦启动，它将一直运行，直到用户输入`exit`。

+   如果用户输入`a:<number>`，*a*收集器将更新为*<number>*。

+   如果用户输入`b:<number>`，*b*收集器将更新为*<number>*。

+   如果用户输入其他内容，将被跳过。

+   当*a*和*b*收集器都有初始值时，它们的和将自动计算并以*a + b = <sum>*的格式打印在标准输出上。在*a*或*b*的每次更改时，和将被更新并打印。

源代码包含了我们将在接下来的四章中详细讨论的功能。

第一段代码代表程序的主体：

```java
ConnectableObservable<String> input = from(System.in); // (1)

Observable<Double> a = varStream("a", input); (2)
Observable<Double> b = varStream("b", input);

ReactiveSum sum = new ReactiveSum(a, b); (3)

input.connect(); (4)
```

这里发生了很多新的事情：

1.  我们必须做的第一件事是创建一个代表标准输入流（`System.in`）的`Observable`实例。因此，我们使用`from(InputStream)`方法（实现将在下一个代码片段中呈现）从`System.in`创建一个`ConnectableObservable`变量。`ConnectableObservable`变量是一个`Observable`实例，只有在调用其`connect()`方法后才开始发出来自其源的事件。在第三章中详细了解它，*创建和连接 Observables、Observers 和 Subjects*。

1.  我们使用`varStream(String, Observable)`方法创建代表`a`和`b`值的两个`Observable`实例，我们将在后面进行详细讨论。这些值的源流是输入流。

1.  我们创建了一个`ReactiveSum`实例，依赖于`a`和`b`的值。

1.  现在，我们可以开始监听输入流了。

这段代码负责在程序中建立依赖关系并启动它。*a*和*b*的值依赖于用户输入，它们的和也依赖于它们。

现在让我们看看`from(InputStream)`方法的实现，它创建了一个带有`java.io.InputStream`源的`Observable`实例：

```java
static ConnectableObservable<String> from(final InputStream stream) {
  return from(new BufferedReader(new InputStreamReader(stream)));// (1)
}

static ConnectableObservable<String> from(final BufferedReader reader) {
  return Observable.create(new OnSubscribe<String>() { // (2)
    @Override
    public void call(Subscriber<? super String> subscriber) {
      if (subscriber.isUnsubscribed()) {  // (3)
        return;
      }
      try {
        String line;
        while(!subscriber.isUnsubscribed() &&
          (line = reader.readLine()) != null) { // (4)
            if (line == null || line.equals("exit")) { // (5)
              break;
            }
            subscriber.onNext(line); // (6)
          }
        }
        catch (IOException e) { // (7)
          subscriber.onError(e);
        }
        if (!subscriber.isUnsubscribed()) // (8)
        subscriber.onCompleted();
      }
    }
  }).publish(); // (9)
}
```

这是一段复杂的代码，让我们一步一步来看：

1.  这个方法的实现将它的`InputStream`参数转换为`BufferedReader`对象，并调用`from(BufferedReader)`方法。我们这样做是因为我们将使用字符串作为数据，并且使用`Reader`实例更容易。

1.  因此，实际的实现在第二个方法中。它返回一个`Observable`实例，使用`Observable.create(OnSubscribe)`方法创建。这个方法是我们在本书中将要经常使用的方法。它用于创建具有自定义行为的`Observable`实例。传递给它的`rx.Observable.OnSubscribe`接口有一个方法，`call(Subscriber)`。这个方法用于实现`Observable`实例的行为，因为传递给它的`Subscriber`实例可以用于向`Observable`实例的订阅者发出消息。订阅者是`Observable`实例的客户端，它消耗它的通知。在第三章中了解更多信息，*创建和连接 Observables、Observers 和 Subjects*。

1.  如果订阅者已经取消订阅了这个`Observable`实例，就不应该做任何事情。

1.  主要逻辑是监听用户输入，同时订阅者已经订阅。用户在终端输入的每一行都被视为一条消息。这是程序的主循环。

1.  如果用户输入单词`exit`并按下*Enter*，主循环将停止。

1.  否则，用户输入的消息将通过`onNext(T)`方法作为通知传递给`Observable`实例的订阅者。这样，我们将一切都传递给感兴趣的各方。他们的工作是过滤和转换原始消息。

1.  如果发生 IO 错误，订阅者将通过`onError(Throwable)`方法收到一个`OnError`通知。

1.  如果程序到达这里（通过跳出主循环），并且订阅者仍然订阅了`Observable`实例，将使用`onCompleted()`方法向订阅者发送一个`OnCompleted`通知。

1.  使用`publish()`方法，我们将新的`Observable`实例转换为`ConnectableObservable`实例。我们必须这样做，否则，对于对这个`Observable`实例的每次订阅，我们的逻辑将从头开始执行。在我们的情况下，我们希望只执行一次，并且所有订阅者都收到相同的通知；这可以通过使用`ConnectableObservable`实例来实现。在第三章中了解更多信息，*创建和连接 Observables、Observers 和 Subjects*。

这说明了将 Java 的 IO 流简化为`Observable`实例的方法。当然，使用这个主循环，程序的主线程将阻塞等待用户输入。可以使用正确的`Scheduler`实例将逻辑移动到另一个线程来防止这种情况发生。我们将在第六章中重新讨论这个话题，*使用调度程序进行并发和并行处理*。

现在，用户在终端输入的每一行都会被这个方法创建的`ConnectableObservable`实例传播为一个通知。现在是时候看看我们如何将代表总和收集器的值`Observable`实例连接到这个输入`Observable`实例了。这是`varStream(String, Observable)`方法的实现，它接受一个值的名称和源`Observable`实例，并返回代表这个值的`Observable`实例：

```java
public static Observable<Double> varStream(final String varName, Observable<String> input) {
  final Pattern pattern = Pattern.compile("\\^s*" + varName + "\\s*[:|=]\\s*(-?\\d+\\.?\\d*)$"); // (1)
  return input
  .map(new Func1<String, Matcher>() {
    public Matcher call(String str) {
      return pattern.matcher(str); // (2)
    }
  })
  .filter(new Func1<Matcher, Boolean>() {
    public Boolean call(Matcher matcher) {
      return matcher.matches() && matcher.group(1) != null; // (3)
    }
  })
  .map(new Func1<Matcher, Double>() {
    public Double call(Matcher matcher) {
      return Double.parseDouble(matcher.group(1)); // (4)
    }
  });
}
```

在这里调用的`map()`和`filter()`方法是 RxJava 提供的流畅 API 的一部分。它们可以在`Observable`实例上调用，创建一个依赖于这些方法的新的`Observable`实例，用于转换或过滤传入的数据。通过正确使用这些方法，您可以通过一系列步骤表达复杂的逻辑，以达到您的目标。在第四章中了解更多信息，*转换、过滤和累积您的数据*。让我们分析一下代码：

1.  我们的变量只对格式为`<var_name>: <value>`或`<var_name> = <value>`的消息感兴趣，因此我们将使用这个正则表达式来过滤和处理这些类型的消息。请记住，我们的输入`Observable`实例会发送用户写的每一行；我们的工作是以正确的方式处理它。

1.  使用我们从输入接收的消息，我们使用前面的正则表达式作为模式创建了一个`Matcher`实例。

1.  我们只通过与正则表达式匹配的数据。其他一切都被丢弃。

1.  这里要设置的值被提取为`Double`数值。

这就是值`a`和`b`如何通过双值流表示，随时间变化。现在我们可以实现它们的总和。我们将其实现为一个实现了`Observer`接口的类，因为我想向您展示订阅`Observable`实例的另一种方式——使用`Observer`接口。以下是代码：

```java
public static final class ReactiveSum implements Observer<Double> { // (1)
  private double sum;
  public ReactiveSum(Observable<Double> a, Observable<Double> b) {
    this.sum = 0;
    Observable.combineLatest(a, b, new Func2<Double, Double, Double>() { // (5)
      public Double call(Double a, Double b) {
        return a + b;
      }
    }).subscribe(this); // (6)
  }
  public void onCompleted() {
    System.out.println("Exiting last sum was : " + this.sum); // (4)
  }
  public void onError(Throwable e) {
    System.err.println("Got an error!"); // (3)
    e.printStackTrace();
  }
  public void onNext(Double sum) {
    this.sum = sum;
    System.out.println("update : a + b = " + sum); // (2)
  }
}
```

这是实际总和的实现，依赖于表示其收集器的两个`Observable`实例：

1.  这是一个`Observer`接口。`Observer`实例可以传递给`Observable`实例的`subscribe(Observer)`方法，并定义了三个方法，这些方法以三种类型的通知命名：`onNext(T)`、`onError(Throwable)`和`onCompleted`。在第三章中了解更多关于这个接口的信息，*创建和连接 Observables、Observers 和 Subjects*。

1.  在我们的`onNext(Double)`方法实现中，我们将总和设置为传入的值，并在标准输出中打印更新。

1.  如果我们遇到错误，我们只是打印它。

1.  当一切都完成时，我们用最终的总和向用户致以问候。

1.  我们使用`combineLatest(Observable, Observable, Func2)`方法实现总和。这个方法创建一个新的`Observable`实例。当传递给 combineLatest 的两个`Observable`实例中的任何一个接收到更新时，新的`Observable`实例将被更新。通过新的`Observable`实例发出的值是由第三个参数计算的——这个函数可以访问两个源序列的最新值。在我们的情况下，我们将这些值相加。只有当传递给该方法的两个`Observable`实例都至少发出一个值时，才会收到通知。因此，只有当`a`和`b`都有通知时，我们才会得到总和。在第五章中了解更多关于这个方法和其他组合器的信息，*组合器、条件和错误处理*。

1.  我们将我们的`Observer`实例订阅到组合的`Observable`实例上。

这是这个示例的输出可能看起来像的样本：

```java
Reacitve Sum. Type 'a: <number>' and 'b: <number>' to try it.
a:4
b:5
update : a + b = 9.0
a:6
update : a + b = 11.0

```

就是这样！我们使用数据流实现了我们的响应式总和。

### 注意

这个示例的源代码可以从这里下载并尝试：[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter01/ReactiveSumV1.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter01/ReactiveSumV1.java)。

# 总结

在本章中，我们已经了解了响应式原则以及学习和使用它们的原因。构建一个响应式应用并不难；它只需要将程序结构化为一系列小的声明式步骤。通过 RxJava，可以通过构建多个正确连接的异步流来实现这一点，从而在整个数据传输过程中转换数据。

本章介绍的两个例子乍一看可能有点复杂和令人困惑，但实际上它们非常简单。它们中有很多新东西，但在接下来的章节中将会详细解释一切。

如果您想阅读更多关于响应式编程的内容，请查看《在 Netflix API 中使用 RxJava 进行响应式编程》这篇精彩的文章，可在[`techblog.netflix.com/2013/02/rxjava-netflix-api.html`](http://techblog.netflix.com/2013/02/rxjava-netflix-api.html)上找到。另一篇介绍这一概念的精彩文章可以在这里找到：[`gist.github.com/staltz/868e7e9bc2a7b8c1f754`](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)。

这些是由 RxJava 的创造者之一 Ben Christensen 制作的有关响应式编程和 RX 的幻灯片：[`speakerdeck.com/benjchristensen/reactive-programming-with-rx-at-qconsf-2014`](https://speakerdeck.com/benjchristensen/reactive-programming-with-rx-at-qconsf-2014)。

在下一章中，我们将讨论一些关于*函数式编程*的概念及其在 Java 8 中的实现。这将为我们提供在接下来的章节中所需的基本思想，并帮助我们摆脱在编写响应式程序时的 Java 冗长性。
