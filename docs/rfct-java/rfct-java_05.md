

# 第五章：重构技巧

我们现在正在深入主题的核心：如何进行重构。认为我们可以把所有已知重构技术都塞进一个章节里，那会有些愚蠢。因此，我们决定专注于最常见且更有趣的几种。我们智慧的主要来源是马丁·福勒在其著作《重构：改善既有代码的设计》中的出色工作。他与肯特·贝克合作，逐步解释了如何进行重构，从最简单到最复杂的情况。对于每一种情况，他们都提供了一种“如何做”的指南，以确保你不会搞砸。在这里，我们提供了一系列我们认为最重要的重构技术，其重要性很大程度上基于我们的经验。

小贴士

对于那些不知道的人来说，马丁·福勒和肯特·贝克是软件开发界有影响力的人物。马丁·福勒以其对软件设计、重构和敏捷方法的贡献而闻名，而肯特·贝克是**极限编程**（**XP**）和 JUnit 测试框架的创造者。他们都对塑造敏捷实践和改进软件开发流程产生了重大影响。

我们试图使福勒所写的内容更加简化，并加入了一些我们自己的工作经验。如果你还没有读过，我们建议你在深入研究本章之前先阅读关于代码恶臭的章节（*第三章*）。

在本章中，我们将涵盖以下内容：

+   构建良好结构化方法的技巧

+   将功能从一个对象移动到另一个对象

+   使你的数据更有组织

+   简化那些棘手的`if`-`else`块

+   清理你的方法调用

+   一些关于使用泛化的技巧

+   当使用枚举类型时给予一点提示

但让我们直接进入正题，从基础开始：方法组合。

# 编写更好的方法

方法组合是一项基本技能，应该是每个开发者重构工具箱的一部分。简单地说，几乎可以说是直截了当，它关乎将你的代码分解成更小的部分，每个部分都以非常可读的方式执行一项任务，以一种非常程序化的方式；你几乎可以把它想象成一份食谱（请原谅我；我是意大利人，所以我总是把一切归结为食物）。细节隐藏在组合方法之下的方法中；这使我们能够以高层次阅读我们的逻辑，我们的代码——我敢说，几乎是“散文”般的。

当你对代码进行重构时，通常涉及从原始方法中提取代码。如果你发现很难为提取的方法想出有意义的名称，这显然是一个明确的迹象，表明你即将提取的代码块可能过于庞大。我还遇到过另一种情况，有人（有时包括我自己）建议为一个包含“和”或“或”等连词的方法命名...在这种情况下，很明显，我们正在处理的方法承担了过多的责任，做了太多的事情。在这种情况下，尝试识别一个更小、更专注的部分进行提取。通常，在较长的方法中，你可能已经注意到某些部分已经被注释标记；这些标记的部分通常可以重构为新的方法。

在应用这种重构技术之后，你可能会在同一类中拥有许多短方法；这本身就是一个改进。然而，在这个时候，可能希望进一步缩短类的大小，通过提取一个专门的方法来实现。

当你必须编写方法时，以下是一些你可以遵循的指南：

+   **越短越好**：在代码中追求简洁和精炼。较小的函数和代码块通常更受欢迎，因为它们提高了可读性和可维护性。

+   **最小化冗余**：在应用组合方法模式时，请注意消除重复的代码片段。这种做法不仅减少了代码库的大小，而且防止了不一致性，并简化了未来的更新。

+   **展示你的目的**：为了促进代码清晰度并采用一种命名约定，避免任何歧义。每个变量、方法和参数都应该以简洁的方式命名，传达其角色和功能。

+   **追求简化**：简化是代码改进的指导原则。它包括去除不必要的复杂性，简化逻辑，并遵循最佳实践。简化的代码不仅更加优雅，而且更易于维护。

+   **保持一致的细节级别**：在应用组合方法模式时，协调你调用的方法的复杂度是至关重要的。避免将简单直接的获取器与执行资源密集型计算的函数混合。在细节级别上的统一有助于代码的凝聚力和可理解性。

实现这些目标有许多方法；接下来，我们将看到其中的一些。

## 提取方法

我们在前几章中已经看到了一些关于它的内容，但我还想深入探讨一下。在我看来，这是我们（或应该）最常进行重构的操作。简单来说，就是将一个非常长的方法——太长了——分解成更小的方法。一个方法最长可以有多长？显然，没有普遍的规则；这取决于你的敏感性和团队。让我们说，我必须用鼠标滚动才能读完整段代码；这已经是一个长度过长的微小信号。我喜欢遵循 Martin Fowler 曾经教给我们的，尽可能地限制方法行的数量；有时我的方法只有两行或三行长。除了我进行方法提取的代码长度外，有时我也会考虑我提取的代码的可重用性，因为避免复制粘贴很重要。然而，我同意 Fowler（当然我也同意！）的说法，即方法应该根据其*意图*来编写；也就是说，这些方法应该做什么，而不是它们将如何去做：将完成某个特定事情所需的所有代码隔离在单个方法中，无论这个方法有多小。然后，方法的名字应该基于方法“做什么”，而不是“怎么做”。

让我们用一个非常简单的例子来说明这些概念。假设我们有以下方法：

```java
public void calculate(int num1, int num2) {
    int sum = num1 + num2;
    System.out.println("Sum: " + sum);
    int difference = num1 - num2;
    System.out.println("Difference: " + difference);
}
```

如您所见，这个方法非常简单：它是一个计算两个数字的和与差并打印它们的方法。当然，现实情况要复杂得多，但原则并没有改变。我们像这样重构代码：

```java
public void calculate(int num1, int num2) {
    int sum = addNumbers(num1, num2);
    printResult("Sum", sum);
    int difference = subtractNumbers(num1, num2);
    printResult("Difference", difference);
}
private static void printResult(String operationName, int result) {
    System.out.printf("Result of %s: %d%n", operationName, result);
}
public static int addNumbers(int a, int b) {
    return a + b;
}
public static int subtractNumbers(int a, int b) {
    return a - b;
}
```

我们已经提取了三种方法：一种用于求和数字，第二种用于计算差值，第三种用于在屏幕上打印结果以避免重复。不要被这段特定代码的长度所迷惑；再次强调，这只是一个示例，用于说明概念——在现实中，你必须将求和和差值操作视为更长的、实现可能更复杂逻辑的方法。我们还可以在这里提出另一个观点，即提取方法可以促进代码重用，因为这样提取的逻辑可以在许多不同的地方被调用。

## 内联方法

这与提取方法正好相反。在如此强调方法提取、短方法之后，提出有时应该内联方法，这可能会显得有些奇怪。这种微小的重构在以下情况下非常有用：当你有一个非常短的方法（通常是几行代码）且该方法从未被重用，并且已经很好地表达了它的功能；在这种情况下，将“解释”功能通过方法名本身隔离开来可能有些过度。

我将提供一个关于方法内联的非常简单的例子。假设我们有一个类中的以下三个方法：

```java
public int add(int a, int b) {
    return a + b;
}
public int multiply(int a, int b) {
    return a * b;
}
public int calculate(int x, int y) {
    int sum = add(x, y);
    int result = multiply(sum, 2);
    return result;
}
```

如你容易看到的，我们有一个`calculate`方法，它接受两个整数作为输入并对它们执行一系列操作。这些操作由`add`和`multiply`方法表示，当然，它们是它们所接受的两个整数的和与积。这些方法只使用一次，并且非常简单：实际上，只有一行。所以，让我们抓住机会进行方法内联，使代码更短、更易读：

```java
public int calculate(int x, int y) {
    int sum = x + y;
    int result = sum * 2;
    return result;
}
```

`add`和`multiply`方法消失了，它们的实现，由一行代码组成，在它们被调用的地方取代了它们。我们可以进一步重构代码，通过直接返回最后一个操作的结果来避免`result`变量：

```java
public int calculate(int x, int y) {
    int sum = x + y;
    return sum * 2;
}
```

这个例子可能与我们之前章节中写的内容相矛盾（实际上，我们现在正在内联之前我们隔离的内容），但这只是为了提供一个非常简单的例子。

总结来说，要执行“内联方法”重构，你需要做的是：

1.  确认该方法在子类中没有被更改。如果已被修改，请避免使用此技术。

1.  跟踪所有调用该方法的地方，然后将这些调用替换为方法中的实际代码。

1.  继续删除该方法。

当然，既然我们在谈论重构，行为必须永远不变；为了确保这一点，没有其他方法比将测试包括在之前写的小食谱中更有效。确保在删除内联的方法之前始终执行测试。我们将在本书的后面部分讨论这个问题，但大多数现代**集成开发环境**（**IDEs**）都提供简单的点击鼠标即可进行此重构的工具。

在以下情况下使用内联方法重构：

+   **当方法的实现简单直接**：如果方法的逻辑简单直接，如前例所示，考虑使用这种重构技术来消除它。

+   **当你想要消除不必要的委托**：有时，你可能将方法的实现转移到另一个方法或类中，这会在代码中引入不必要的间接引用。为了简化代码并移除这额外的委托层，使用内联方法技术。

+   **作为后续重构的基础**：重构的过程并不总是线性的，对同一代码有各种重构方式。例如，你可以使用提取方法重构来隔离不同的代码段。然而，如果你最初选择的重构路径没有产生改进的代码，你可以使用内联方法将之前提取的代码重新整合到其原始方法中，然后探索其他重构方法。

## 提取并内联变量

我们将这些两种重构技术总结在单独的部分，因为——根据我们谦逊的观点——它们与提取方法和内联方法技术非常相似，但应用到了单个变量上。

引入（提取）一个变量而不是表达式的动机——或许可以说是唯一的动机——是当后者可能难以理解时（不仅是在技术层面上；我们在这里谈论的是表达式本身的动机）。让我们用一个例子来更好地解释自己：

```java
if (transport.getEquipment().toUpperCase().equals("PLN") || transport.getEquipment().toUpperCase().equals("TRN")) {
    //do something
} else {
    //do something else
}
```

在这个代码片段中，我们分析了一个代表交通工具的通用 `transport` 对象的字段；在其各种字段中，该对象还有一个 `equipment` 对象，告诉我们它是哪种交通工具。不幸的是，这个字段是一个字符串（参见*第三章*，*原始执着*），我们被迫编写一个 `if` 语句来理解它的类型，是飞机还是火车。如果我们插入 `equals` 的结果到一个具有自解释名称的几个变量中，代码片段就会更清晰：

```java
boolean isPlane = transport.getEquipment().toUpperCase().equals("PLN");
boolean isTrain = transport.getEquipment().toUpperCase().equals("TRN");
if (isPlane || isTrain) {
    //do something
} else {
    //do something else
}
```

有些人可能会简单地插入一条注释来解释那些神秘的字符串 `"PLN"` 和 `"TRN"` 指的是什么，但个人来说，我们发现这种方法更有效。你也可以使用枚举而不是纯字符串，这样编译器就能捕捉到可能的打字错误。通过引入由变量组成的这些“中间部分”，代码的可理解性得到了提高。缺点是……你的代码中包含了更多的变量。就像生活中的一切一样，这是一个平衡问题！选择权在你。

相反，“内联变量”的技术与“内联方法”非常相似，因此值得通过一个与相对部分中看到的例子几乎相同的例子来直接解释它：

```java
int sum = num1 + num2;
printResult("Sum", sum);
```

在这种情况下，将加法的结果赋值给 `sum` 变量——然后打印后不再使用它——与表达式本身相比，对代码的理解并没有增加多少。因此，最好将变量直接内联在它被使用的唯一点上，消除代码片段的第一行：

```java
printResult("Sum", num1 + num2);
```

## 将函数组合到类中

作为重构的最后著名例子，我们提出了一种通过“重新组合”方法来重新组织代码的技术。我们提出的一种技术是在一个类中结合几个方法。当有几个方法具有一个（或多个）共同的输入参数时，即实际上作用于相同的类时，我们使用这种技术；也就是说，在实践中，它们会作用于相同的类。再次，让我们直接展示一个应该能更好地阐明概念的例子。假设我们有以下一组方法：

```java
String getLocalizedName(Location location) { ... }
Collection<Location> getAdjacentLocations(Location location) { ... }
Coordinates getCoordinates(Location location) { ... }
```

所有这些方法都接受一个 `Location` 类型的单一参数。最好是将与 `Location` 参数相关的所有逻辑封装在一个单独的点中。为此，我们创建了一个包含所有逻辑的类：

```java
public class LocationHandler {
    private final Location location;
    public LocationHandler(Location location) {
        this.location = location;
    }
   String getLocalizedName() { ...}
   Collection<Location> getAdjacentLocations() { ...}
    Coordinates getCoordinates() { ...}
}
```

`Location`实例仅在构造函数中传递一次。然后我们从移除的方法签名中删除了参数；现在，你不再需要传递它。当然，仔细考虑是否以这种方式重构；如果你的`Location`实例经常变化，例如对于每次方法调用，使用这种模式可能不是最好的主意，因为你应该为每个调用实例化一个`LocationHandler`。

# 在对象之间移动功能

正如我们多次提到的那样，那些在这个行业工作了几年的专业人士知道，组织我们的代码可能是最具挑战性的部分之一。我个人认为，我从未在第一次尝试时就成功地设计出一个软件项目。但，再次强调，软件幸运地（或不幸地！）是可塑的，所以通过适当的限制和大量的重构，我们可以轻松地将方法和字段从一个类移动到另一个类。我们可以决定提取一个类或删除一个并将其内联。最重要的是，我们可以消除所谓的死代码。但让我们开始展示一些内容。

## 移动方法和字段

这种重构相当基础（马丁·福勒称之为重构的“黄油面包”），但它是最常执行的重构之一。这是因为，正如我们之前提到的，保持软件项目中的某种模块性是一种良好的实践，以便类、包等可以根据它们操作的上下文有组织地分组。例如，在一个包中，我们可能放置所有负责管理用户信息的类，而在另一个包中，我们可能收集所有外部服务的客户端。没有单一的标准，而是往往有一定的主观性。这就是为什么方法或单个字段经常从一个类移动到另一个类的原因。这种移动，遵循将相关事物放在一起和分离应该分离的事物的原则，简化了代码。

这是一些情况之一，做某事比解释它简单直观，但我会尽量给出马丁·福勒的“配方”，也许会稍微简化一下。

将方法从源类转移到目标类涉及几个步骤，必须遵循以确保重构过程成功：

1.  检查该方法所使用到的依赖和其他类成员。确定是否需要将它们也重新定位到新类中。

1.  检查该方法是否被超类或子类引用。如果它被这些类使用，可能无法移动该方法。

1.  在目标类中建立该方法，并将现有代码复制到其中。如果重新定位的方法依赖于原始类的实例，你可以将其作为方法参数传递。

1.  删除旧方法中的代码，并将调用直接指向新方法。（这是可选的；你也可以使用**功能开关**。）

1.  可选地，决定是否完全消除旧方法并直接调用新方法。

功能开关

**功能开关**，也称为**功能标志**，是一种允许我们在不直接修改源代码的情况下激活或停用特定功能的技术。这种方法带来了诸如持续部署、简化实际条件下的测试、高效的问题管理、逐步功能发布以进行风险管理以及能够比较功能变体以进行明智决策等好处。

在最抽象的意义上，功能开关在 Java 方法中可以这样工作：

`public void` `applyLogic() {`

`    if (``newFeatureEnabled) {`

`applyNewLogic();`

`}` `else {`

`applyCurrentLogic();`

`    }`

`}`

最后，建议执行你的测试套件。即使讨论重构时应该隐含这一点，但仍然需要牢记。

对于单个字段，过程非常相似。我们有一个字段，它被另一个类比定义它的类使用得更多。上下文没有得到尊重；因此，我们将字段移动到那个其他类，随后更改所有使用该字段的地方。

在应用**提取类**技术时，字段经常被移动，决定哪个类应该保留特定的字段可能有点棘手。这里有一个实用的经验法则：将字段放在使用它最多的方法所在的同一个类中，或者放在你找到大多数这些方法的类中。这个规则在字段显然位于不适当位置的场景中也可能是有益的。

再次，让我们以 Fowler 所描述的方法来展示：

1.  当处理公共字段时，通过将其转换为私有字段并提供公共访问方法（你可以利用*封装字段*技术来完成这个目的，如*字段* *封装*部分所述），重构变得显著更容易管理。

1.  在接收类中建立相同的字段及其相应的访问方法。

1.  确定如何访问接收类。你可能已经有一个字段或方法可以提供所需的对象。如果没有，你需要创建一个新的方法或字段来存储与接收类关联的对象。

1.  将接收类中所有引用旧字段的实例替换为适当的方法调用。如果字段不是私有的，根据需要在上层类和任何子类中处理这个问题。

1.  从原始类中删除该字段。

## 将语句移动到/从方法中

重复的代码是我们代码库中最常见的问题之一；有时复制的诱惑太强烈。这可以在调用某个方法之前，看到某一行代码；也可能在每次调用该方法之前重复相同的代码行。这时，你就会明白最好的选择是将该语句直接移动到该方法内部。

相反，代码中可能存在一个与该方法上下文不太吻合的语句；也许在一个执行流程中是有意义的，但在另一个执行流程中则不然。在这种情况下，采取与之前相反的措施，将语句移动到方法外部，直到它真正需要的位置。

我认为这项技术非常常见，但相当直接。对于如何工作的“机制”的更深入探索，我建议您参考*进一步阅读*部分。

## 隐藏代理

当一个类（称为客户端）通过另一个类型为*A*（称为服务器）的类获取类型*B*（称为代理）的对象时，应用这种重构技术。当客户端代码调用服务器对象字段中定义的方法时，它将直接依赖于该代理对象。因此，如果代理对象的接口发生变化，所有依赖该代理的服务器客户端都将受到影响。为了消除这种依赖关系，可以在服务器中引入一个直接的代理方法，从而有效地隐藏代理。然后，对代理接口所做的任何后续修改都只会影响服务器本身，保护客户端免受这些更改的影响。

一个例子可以稍微澄清我们刚才所说的。让我们假设有两个类：

```java
public class Itinerary {
    private final String departureAirport;
    private final String arrivalAirport;
    //constructor, getters...
}
public class Flight {
    private final Itinerary itinerary;
//constructor, getter...
}
```

如果我们想获取`departureAirport`，但我们有一个`Flight`实例，我们的代码可能如下所示：

```java
var departureAirport = flight.getItinerary().getDepartureAirport();
```

当然，这是可以的，但事实上我们现在还必须意识到`Itinerary`类；实际上这是无用的，因为我们只需要`departureAirport`。我们不希望因为`Itinerary`类的更改而更改我们的代码。因此，我们将编写如下所示的内容：

```java
public class Flight {
    private final Itinerary itinerary;
    public String getDepartureAirport(){
        return itinerary.getDepartureAirport();
    }
    //constructor, getter...
}
```

因此，我们只需调用`Flight`类的一个方法来获取我们需要的字段。当然，我们也将不得不更改客户端代码：

```java
var departureAirport = flight.getDepartureAirport();
```

代理现在被隐藏了。我们的代码更少，对象之间的关系也更少。

但作为一个配方，正如马丁·福勒所做的那样，以下是执行此重构的步骤：

1.  对于客户端类调用的每个代理类方法，在服务器类中生成一个相应的方法，将调用转发到代理类。

1.  修改客户端代码，调用服务器类中的方法，而不是直接调用代理类。

1.  如果你的修改成功消除了客户端对代理类的依赖，你可以在服务器类中安全地删除对代理类的访问方法（最初用于获取代理类的方法）。

一个缺点是，如果你必须生成大量的委托方法，服务器类最终可能成为一个不必要的中间件，导致中间人的大量增加。

## 删除死代码

啊，这里是我最满意的事情之一：删除不必要的代码！死代码指的是在程序执行过程中不再执行或无法访问的源代码部分。

死代码可能由各种原因引起，如下所示：

1.  **代码删除或重构**：当开发者修改程序时，他们可能会删除或注释掉不再需要的某些代码部分。这些遗留的代码片段成为死代码。

1.  **条件分支**：在某些情况下，代码可能被编写在执行过程中永远不会为真的条件语句的分支中，这使得这些分支中的代码实际上成为死代码。

1.  **未使用的变量或函数**：如果变量或函数被定义了，但在程序中任何地方都没有使用，它们被认为是死代码。

你不希望在项目中出现死代码：它会使项目更难理解，增加维护成本，并可能引入错误。

所以，请，如果你在你的项目中发现了死代码——现代 IDE 现在完全有能力做到这一点——请直接删除它。不——不要将其注释掉！摆脱它；如果你只是想恢复它，我敢打赌你正在使用版本控制系统（如 Git 或 Subversion），它将能解决这个问题。

还要注意所谓的**死注释**。正如我们之前提到的，我们对注释的看法是应该尽可能少，并且专注于解释*为什么*要做某事。在一个理想和神奇的世界里，代码本身应该能够解释，而不需要额外的注释。但往往——非常经常——注释没有随着代码一起更新，不仅变得无关紧要，而且有害，有时甚至误导。即使你在学校被教导给代码添加很多注释（至少，我是这样），请尽量将其保持在最低限度。

我们关于死代码所说的当然也可以应用到模块、服务或甚至单个功能上。在生产环境中，看到模块和服务的激增，而这些模块和服务实际上不再被使用，它们唯一的作用就是消耗宝贵的（读：昂贵的）资源，这是相当常见的。通常，每个人都会忘记它们，直到需要更新它们（例如，由于发现其依赖项中的漏洞）或有人意识到它们在硬件资源上花费了大量的钱！在功能的情况下，情况类似：为什么保留那些使代码更复杂但没有实际益处的未使用功能？一旦有机会，就摆脱它们，你不会后悔的！

我们已经看到了代码重构的一个重要部分，并且我们学习了一些在不造成太大干扰的情况下移动代码片段的技术。除了位于正确的位置外，特性还必须具有良好的结构和组织。让我们看看一些关于这个的建议。

# 组织数据

数据是如何组织的，是我们职业中最重要的一部分；有逻辑地聚合信息是构建一个坚实、可维护和可扩展的软件项目的基础。各种类型的重构帮助我们在这方面，而且，Fowler 再次帮助我们详细地导航它们。在接下来的章节中，我们将展示我们认为最常见或可能具有误导性的那些。

## 字段封装

我们将把我们认为非常相似且最终不向用户揭示类内部结构的技术组合在一起。这个原则被称为 **封装**，并且是 **面向对象编程**（**OOP**）的基本概念之一；隐藏类的内部结构并提供对其字段的访问方法是有益的，因为它简化了类的使用，保护了数据，提高了可维护性，封装了行为，并允许访问控制。数据不会与相关的行为分离，程序部分的模块化不会受到影响，维护也变得简单。要从公共字段开始实现封装，只需将字段设置为私有，并公开所谓的 *getter* 和 *setter* 方法。例如，看看以下代码：

```java
class Flight {
    public Airport departureAirport;
}
```

这将变成以下内容：

```java
class Flight {
    private Airport departureAirport;
    public Airport getDepartureAirport() {
        return departureAirport;
    }
    public void setDepartureAirport(Airport departureAirport) {
        this.departureAirport = departureAirport;
    }
}
```

`departureAirport` 字段已被设置为私有，我们提供了两个分别读取和写入该字段本身的方法。如果您想使类不可变——我们通常强烈推荐这样做——您只需将 `departureAirport` 设置为 `final`，将其添加到构造函数中，并消除 *setter* 方法即可。

有些人可能会争论说，这样我们仍然暴露了类的结构，并且...他们会是对的！然而，请记住，这是一个非常简单的案例：例如，对于所有字段来说，getters 和 setters 不一定必须存在，或者它们不一定必须仅执行对字段的读取或写入操作——它们也可以涉及一些逻辑（但在此情况下要小心性能！）。

另一个重要的注意事项：从 Java 14 开始，引入了 `Record` 关键字——一个允许您在一行中定义类，如 `Flight`，并减少样板代码的构造。您可以参考 *进一步阅读* 部分以获取更多详细信息。

我们还可以在类本身内封装字段。让我们扩展一下我们的 `Flight` 类，假设我们还需要内部访问 `departureAirport` 字段：

```java
class Flight {
    private Airport departureAirport;
    // getter and setter
    public String getDepartureAirportCode(){
        return this.departureAirport.getAirportCode();
    }
}
```

有时，直接访问类中的私有字段可能缺乏所需的灵活性。当字段中的数据被设置或接收时，你可以选择执行复杂的操作——例如，*懒加载*和字段值的验证可以轻松地集成到字段的自定义获取器和设置器中。除此之外，你还有能力在子类中重写获取器和设置器。

因此，`Flight`类看起来会是这样（请观察`getDepartureAirportCode`方法）：

```java
class Flight {
    private Airport departureAirport;
    // setter
    public Airport getDepartureAirport() {
        return departureAirport;
    }
    public String getDepartureAirportCode(){
        return this.getDepartureAirport().getAirportCode();
    }
}
```

有些人可能会争论说，我们在灵活性上有所增加，但由于需要阅读`getDepartureAirport`方法，我们在可读性上有所损失；另一些人可能会说，只要方法命名得当，仅代表获取器，这就不真实；我们也在促进松耦合。选择哪条路径取决于你。

我们想要展示的封装的最后一个案例涉及集合，我确实看到过这个问题很多次。在这种情况下，类包含一个`Collection`类型的字段，其获取器和设置器操作整个集合，这使得与之交互更具挑战性。稍微修改之前的示例，假设我们有一个包含机场`Collection`字段的`Flight`类，每个停靠点都有一个：

```java
class Flight {
    private List<Airport> itineraryAirports;
    public List<Airport> getItineraryAirports() {
        return itineraryAirports;
    }
    public void setItineraryAirports(List<Airport> itineraryAirports) {
        this.itineraryAirports = itineraryAirports;
    }
}
```

利用这些集合的协议与其他数据类型略有不同。重要的是要注意，获取器方法不应返回实际的集合对象本身。这样做将允许客户端在不通知拥有类的情况下操作集合的内容。此外，这将向客户端揭示对象内部数据结构的过多信息。相反，用于检索集合元素的方法应提供一个值，以防止对集合进行任何修改，并避免过多地揭示其结构信息。不应有直接分配值给集合的方法。相反，协议应提供添加和删除元素的操作。这种方法使拥有对象能够控制集合中元素的添加和删除：

```java
class Flight {
    private List<Airport> itineraryAirports;
    public List<Airport> getItineraryAirports() {
        return Collections.unmodifiableList(itineraryAirports);
    }
    public void addAirport(Airport itineraryAirport) {
        this.itineraryAirports.add(itineraryAirport);
    }
    public void removeAirport(Airport itineraryAirport) {
        this.itineraryAirports.remove(itineraryAirport);
    }
}
```

我们已经移除了设置器方法，并添加了两个方法用于向列表中添加和删除元素。当我们通过获取器返回列表时，我们使用 Java 17 的`Collections::unmodifiableList`方法（但我们也可以使用 Guava、Apache Commons 或其他）实例化一个新的不可变列表。

几个免责声明

我们正在修改对象，这与我们在这些页面上倡导的不变性概念相悖，所以请谨慎行事。另外，请记住，你正在执行添加和删除操作（在这种情况下，`Airport`）的集合类型必须重写`equals`和`hashCode`方法。

## 用对象替换原始类型

当处理被称为`Flight`类的代码异味时，这种重构方法会帮我们解决问题，这次，让我们假设一个包含服务级别，也称为“机舱类”的字段被定义为如下：

```java
class Flight {
    private String cabinClass;
    public Flight(String cabinClass) {
        this.cabinClass = cabinClass;
    }
    //getter and toString
}
```

如果我们只想从航班列表中过滤出最昂贵的航班，我们可能需要做如下操作：

```java
var expensiveFlights = flightList.stream().filter(f -> f.getCabinClass().equals("Business") || f.getCabinClass().equals("First Class")).collect(Collectors.toList());
```

这并不理想。我需要知道机舱类所有可能的值，并且我必须确切知道哪一个比另一个更贵。在这些情况下，最好使用一个类来替换原始类型，这不仅允许封装，还可以实现自定义逻辑：

```java
var expensiveFlights = flightList.stream().filter(f -> f.getCabinClass().higherThan(new CabinClass("Economy"))).collect(Collectors.toList());
```

如您所见，`higherThan`方法允许我们实现一种逻辑，比较我们的机舱类与另一个，而无需担心管理维护这种逻辑本身。

我认为看一下`CabinClass`类是值得的：

```java
public class CabinClass {
    private final String name;
    private final int value;
    public CabinClass(String name) {
        this.name = name;
        switch (name) {
            case "Economy":
                value = 1;
                break;
            case "Premium":
                value = 2;
                break;
            case "Business":
                value = 3;
                break;
            case "First":
                value = 4;
                break;
            default:
                throw new IllegalArgumentException();
        }
    }
    //getters
    public boolean higherThan(CabinClass other){
        return this.getValue() > other.getValue();
    }
}
```

它有一个名称和一个值；后者用于确定机舱类在“价值尺度”中的位置，并且它被分配给类构造函数。我们周围还有很多字符串，但我认为我们可以做得更好。让我们继续前进。

## 用子类替换类型代码

让我们从之前的例子开始，即机舱类。您可能已经注意到，通过在构造函数中传递一个字符串来定义机舱类的“类型”并不是理想的。在简单场景中，我们可能在编写类型时犯拼写错误；实际上，真正的问题是我们在将知识委托给调用者，而这些知识应该是`CabinClass`类内部的知识。当我们遇到这种情况时，即我们的类有一个“类型”，并且类的行为本身可能依赖于这个类型，一个有用的重构是创建子类。我们的`CabinClass`类将变成以下这样：

```java
public class CabinClass {
    protected final String name;
    protected final int value;
    protected CabinClass(String name, int value) {
        this.name = name;
        this.value = value;
    }
    //getters, toString, and higherThan
}
```

但我们也会定义一系列如下所示的子类：

```java
public class Economy extends CabinClass{
    public Economy() {
        super("Economy", 1);
    }
}
public class Premium extends CabinClass{
    public Premium() {
        super("Premium", 2);
    }
}
public class Business extends CabinClass{
    public Business() {
        super("Business", 3);
    }
}
public class First extends CabinClass{
    public First() {
        super("First", 4);
    }
}
```

注意参数是如何在构造函数中定义的，以及调用者无法修改它们的情况。这样，除了其他优点外，我们还利用了面向对象编程的所有优势，并且代码的可读性也因此得到了提高。为了完整性，这里是一个相应修改后用于过滤昂贵航班的代码片段：

```java
var economyClass = new Economy();
var expensiveFlights = flightList.stream().filter(f -> f.getCabinClass().higherThan(economyClass)).toList();
```

注意，我们只是简单地实例化一个`Economy`类，以便在`higherThan`方法中使用。

良好地组织数据意味着，最终目的是简化事物——或者至少使它们更易于阅读。可以使代码非常难以阅读的一件事是条件逻辑。让我们看看如何稍微简化一下。

# 简化条件逻辑

对于本节，我们也将依赖 Martin Fowler，并尝试解释一些我们认为最常见问题的重构。选择是任意的，仅基于我们的经验。有关更多详细信息，我们建议您参考*进一步阅读*部分。

## 返回特殊值而不是 null

不要返回`null`。这是一个每个人——甚至是有多年经验的工程师——有时会忘记的咒语。有些情况下，方法应该返回一个结果，但无法返回：执行流程中的某些错误；某些异常情况。Java 和许多其他语言允许返回`null`，但出于明显的原因，最好不要这样做——其中之一是避免调用者中的`NullPointerException`，或者迫使调用者每次都检查方法的结果是否不是`null`。

小贴士

托尼·霍尔（Tony Hoare）在 1965 年将空引用（Null references）引入到**ALGOL W**中，据他所说，“仅仅因为它很容易实现”。回顾这个决定，他将其称为自己的“十亿美元的错误”。更多内容请参阅*进一步阅读*部分。

避免空引用有各种方法，但最常见的一种是返回一个所谓的**特殊情况**对象，它是一个具有预定义值的默认对象。

以以下代码片段为例：

```java
CustomerAddress customerAddress = addressRepository.findByCustomerId(customer.getId());
if (customerAddress == null) {
   customerStreet = "Unknown";
   customerCity = "Unknown";
} else {
   customerStreet = customerAddress.getStreet();
   customerCity = customerAddress.getCity();
}
```

我们可以注意到，我们需要通过检查存储库返回的对象是否为`null`来区分逻辑。在这种情况下，如果存储库方法返回了一个`Optional`，那么这甚至可能不是很有用，因为我们仍然需要编写一个`if`语句；然而，在我们看来，这会更好，因为它至少会让调用者意识到调用可能不会返回期望的结果。尽管如此，一个解决方案是引入一个特殊情况对象。以下是一个可能的 Java 实现：

```java
interface CustomerAddress {
    String getStreet();
    String getCity();
}
```

我们引入了一个接口，允许我们实现特殊情况，然后我们在两个类中实现了它：

```java
class UnknownCustomerAddress implements CustomerAddress {
    @Override
    public String getStreet() {
        return "unknown";
    }
    @Override
    public String getCity() {
        return "unknown";
    }
}
class ActualCustomerAddress implements CustomerAddress {
    String street;
    String city;
   //constructor and getters
}
```

注意到`ActualCustomerAddress`类是实际的“真实”类，而另一个则是一种“占位符对象”，用于允许调用者在调用方法时不会改变执行流程。当然，我们有一个（小？）缺点，就是创建了比之前更多的类；代码的整洁性因此得到了极大的提升。

理论上，我们也可以为我们的情况创建一个特定的异常，并用它来代替返回一个假对象。但老实说，我们并没有看到这种情况发生很多（我们也不是特别支持这样做！）。然而，为了完整性，承认这一点是值得的。

## 使用多态而不是条件

让我们回顾一下在*第三章*中讨论过的代码异味：重复的`switch`语句。当存在大量的条件逻辑时，引入一些结构会更好。当代码中多次重复`switch`语句，尤其是在名为`type`的变量周围时，重构可能是一个更好的方法：

```java
public Long calculateDistance(Itinerary itinerary) {
    Long distance;
    switch (itinerary.getType()) {
        case "TRAIN": {
            var departureLocation = getDepartureStation(itinerary);
            var arrivalLocation = getArrivalStation(itinerary);
            distance = calculateItineraryDistance(departureLocation, arrivalLocation);
            break;
        }
        case "FLIGHT": {
            var departureLocation = getDepartureAirport(itinerary);
            var arrivalLocation = getArrivalAirport(itinerary);
            distance = calculateItineraryDistance(departureLocation, arrivalLocation);
            break;
        }
        default:
            throw new IllegalArgumentException("Unknown type");
    }
    return distance;
}
```

在这个方法中，它计算旅行行程中的行驶距离——无论是乘坐飞机还是火车——我们遇到了几个问题。调用者必须知道`type`可能采取的可能值，并在代码中需要的地方实现不同的逻辑。此外，我们预计代码中还会有其他需要根据行程类型不同而表现不同的点。

相反，我们可以简单地利用多态性：

```java
abstract class Itinerary {
    public abstract Long calculateItineraryDistance();
}
final class FlightItinerary extends Itinerary {
    @Override
    public Long calculateItineraryDistance() {
        //calculations for a flight...
    }
}
final class TrainItinerary extends Itinerary {
    @Override
    public Long calculateItineraryDistance() {
        //calculations for a train...
    }
}
```

在前面的代码中，多态性允许我们将不同的行程类型（`FlightItinerary`和`TrainItinerary`）视为通用基类（`Itinerary`）的实例。这使得你可以在不知道它们的具体类型的情况下调用`calculateItineraryDistance`方法，从而提高代码的灵活性和复用性，并创建两个扩展相同抽象类的类。在这个时候，调用者将不再需要担心任何事情：

```java
public Long calculateDistance(Itinerary itinerary) {
    Long distance = itinerary.calculateItineraryDistance();
    return distance;
}
```

## 移除重复的条件

有时可能会发生这样的情况，在条件语句中的`if`-`else`块的两个分支中重复相同的代码。为什么会出现这种情况，我老实说不知道；然而，我可以证明，当方法非常长或者由于重构的结果时，这种情况出现的频率更高。在我看来，这通常是一个简单的疏忽，但它发生的频率比你想象的要高。例如，在前面的例子中，我们可以精确地观察到`calculateItineraryDistance(departureLocation, arrivalLocation);`在`switch`语句的两个不同情况下被调用。一般来说，当我们处于重复相同调用的条件下时，我们可以这样做：

```java
var result = doCalculations(x, y);
if (doCalculations(x, y) > 5) {
    //do something
    printResult(result);
} else {
    //do something else
    printResult(result);
}
```

简单的建议是将重复的调用从`if`语句中移除，如下所示：

```java
var result = doCalculations(x, y);
if (doCalculations(x, y) > 5) {
    //do something
} else {
    //do something else
}
printResult(result);
```

请记住，可能相关的方法尚未存在；然而，你可以使用**提取** **方法**技术将重复的代码组合在一起。

## 保护子句

我给你的最后一条建议是关于简化条件逻辑中的嵌套`if`语句；我必须承认，当我发现它们时，代码会越来越向屏幕的右侧移动，几乎形成需要非常大的显示器才能欣赏的曼荼罗！这是向深渊的倾斜。

这里是一个方法的例子，除了非常简单之外，它还有许多嵌套的`if`语句。它接受三个参数作为输入，并找出最大的一个，但它们都不能是负数：

```java
public void printLargestPositive(int x, int y, int z) {
    if (x > 0) {
        if (y > 0) {
            if (z > 0) {
                if (x > y && x > z) {
                    print("x is the largest.");
                } else if (y > x && y > z) {
                    print("y is the largest.");
                } else {
                    print("z is the largest.");
                }
            } else {
                print("z is not positive.");
            }
        } else {
            print("y is not positive.");
        }
    } else {
        print("x is not positive.");
    }
}
```

由于典型的代码执行流程中的不清晰，理解每个条件的目的和功能具有挑战性。这些条件表明开发过程缺乏组织，每个条件都是作为一个临时解决方案添加的，而没有考虑整体结构的优化。

为了简化情况，将异常情况分离成不同的条件，如果守卫子句评估为`true`，则立即终止执行并返回一个`null`值。本质上，你的目标在这里是简化代码结构，使其更加线性，如下所示：

```java
public void printLargestPositiveRefactored(int x, int y, int z){
    if (x <= 0) {
        print("x is not positive.");
        return;
    }
    if (y <= 0) {
        print("y is not positive.");
        return;
    }
    if (z <= 0) {
        print("z is not positive.");
        return;
    }
    if (x > y && x > z) {
        print("x is the largest.");
    } else if (y > x && y > z) {
        print("y is the largest.");
    } else {
        print("z is the largest.");
    }
}
```

我们知道这段代码可以进一步重写，但这只是为了解释概念。有些人可能会争论这个方法内部有太多的返回，认为最好只有一个。我坦白地说，我不确定是否太多，但这确实是一个值得观察的好点。然而，我同意史蒂夫·麦克康奈尔在他的书《Code Complete》中的说法：当使用`return`语句可以使你的代码更容易理解时，就使用它。在某些函数中，一旦你有了答案，就立即将其返回给调用函数。如果函数在找到错误后不需要进行任何额外的清理，不立即返回就意味着需要编写更多的代码。

此外，如果代码主要关注性能，在处理最常见的情况后停止可以跳过一些额外的检查。这实际上非常有帮助，特别是如果一种或两种情况占运行时间的 80%或 90%。

# 简化方法调用

方法调用在面向对象编程中至关重要，因为它们使对象能够执行特定的任务或操作。有许多旨在简化对象之间交互的技术；我们将看到其中一些最有趣的技术。

## 避免副作用

我们已经在*第三章*中讨论了对象的外部和可变性，解释了为什么它们在软件项目中不是理想的。引起副作用的一种典型方式是在查询（代码的一部分，仅用于检索信息）和修改器（对某些数据或系统执行操作并改变其状态的代码）之间混合。以下是一个例子：

```java
public Price getTotalItineraryPrice(User user, Itinerary itinerary){
    Price totalPrice = calculateTotalPrice(itinerary);
    emailService.sendPriceRecap(user);
    return totalPrice;
}
```

在这个例子中，该方法计算特定旅行行程的总价格，并同时向客户发送总结电子邮件。这里有几个问题。首先（但这不是最严重的问题），名称具有误导性，因为它与方法的实际行为不一致。此外，该方法做了不止一件事；特别是，它在一方返回信息（一个`Price`实例），而在另一方通过通知用户改变事物的状态。最好将这些关注点分开，如下所示：

```java
public Price getTotalItineraryPrice(Itinerary itinerary) {
    return calculateTotalPrice(itinerary);
}
public void sendEmailRecap(User user) {
    emailService.sendPriceRecap(user);
}
```

现在我们有两个不同的方法，一个用于请求信息，另一个以某种方式修改系统的状态。值得注意的是，查询方法可以在不影响系统状态的情况下被多次调用。代码更清晰，我们还从`getTotalItineraryPrice`方法中移除了一个参数。

## 移除设置方法

实际上，这一节几乎应该是上一节的延续。移除所谓的*setter*方法，这些方法允许你设置对象字段的值，可以使它们不可变，从而消除不希望产生的副作用。

我们已经在本章中讨论了这一点，但我们想更详细地探讨一下。让我们考虑一个代表人的非常简单的类；为了简单起见，我们只包括两个字段：

```java
class Person {
    private String taxCode;
    private String name;
    public String getTaxCode() {
        return taxCode;
    }
    public void setTaxCode(String taxCode) {
        this.taxCode = taxCode;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

`Person`类通过`taxCode`和`name`来表示。`taxCode`是唯一的，代表一个人的标识符。正如我们所见，你可以实例化这个类并设置其字段，如下所示：

```java
Person p = new Person();
p.setTaxCode("4598308JKFLD3424243");
p.setName("John Doe");
```

如果我们将这个实例返回给调用方法，没有人能保证其字段不会被修改。所讨论的重构简单来说就是移除 setter 方法，将其参数添加到构造函数中，如下所示：

```java
Person p = new Person("4598308JKFLD3424243");
p.setName("John Doe");
```

注意`taxCode`是如何在构造函数中传递的。在`Person`类中，`taxCode`字段可以成为`final`。

### 真正的不可变性——构建器模式

要使对象在所有字段上都具有真正的不可变性，我们建议使用**构建器模式**。

构建器模式是一种设计模式，用于逐步构建复杂对象。它将对象的构建与其表示分离，允许你创建同一对象的不同的表示形式。

这就是构建器模式通常是如何工作的：

+   **创建构建器类**：首先，你创建一个单独的构建器类来构建对象。这个构建器类有设置你想要创建的对象的各种属性的方法。

+   **设置属性**：你使用构建器类中的方法来设置对象的属性。每个方法通常返回构建器对象本身，允许你链式调用方法（这被称为**方法链式调用**）。

+   构建器对象上的`build`方法，它构建并返回具有指定配置的最终对象。

为`Person`类创建的构建器类看起来是这样的：

```java
public final class PersonBuilder {
    private String name;
    private final String taxCode;
    private PersonBuilder(String taxCode) {
        this.taxCode = taxCode;
    }
    public static PersonBuilder builder(String taxCode) {
        return new PersonBuilder(taxCode);
    }
    public PersonBuilder name(String name) {
        this.name = name;
        return this;
    }
    public Person build() {
        Person person = new Person(taxCode);
        person.setName(name);
        return person;
    }
}
```

你可以清楚地看到静态的`builder`方法，它实例化构建器本身，只接受`taxCode`作为其唯一参数，这是强制性的。`name`方法只是`name`属性的 setter；你可以为你的类的每个字段有一个 setter。最后，`build`方法创建并填充一个`Person`实例。在类型上使用构建器模式是有意义的，只要你不提供任何其他方法来实例化该类型；例如，你可以将`Person`类型的构造函数设为私有，并将`builder`作为一个内部类创建。

`PersonBuilder`的使用相当简单：

```java
Person p = PersonBuilder.builder("4598308JKFLD3424243")
        .name("John Doe")
        .build();
```

一旦对象被构建（实例化），你不能修改其字段。在这个例子中，`builder`方法（在这个例子中，实际上是构建器，而不是`Person`类）接受`taxCode`作为参数，这是强制性的。

有工具可以自动化构建器类的创建，我们将在本书的后面部分讨论。

# 使用泛化

**泛化**是面向对象编程中最强大的特性之一，必须明智地使用（*权力越大，责任越大*）。在这里，我将仅报告这一领域的一些最有趣的重构，快速处理最基础的，对其他方面则深入探讨。

## 提升字段

这种技术包括将一个字段（或变量）从子类移动到超类。这通常发生在多个子类共享相同的字段或当你想在超类中建立公共接口或行为时。以下是一个例子：

```java
public class Triangle {
    private Integer sidesNumber;
}
public class Square {
    private Integer sidesNumber;
}
```

`Triangle` 和 `Square` 有一个共同的字段；只需提取一个接口或一个 `abstract` 类来达到目的：

```java
public abstract class Polygon {
    private Integer sidesNumber;
}
public class Square extends Polygon {
}
public class Triangle extends Polygon{
}
```

## 降级字段

这与降级字段相反，当你在超类中定义了一个字段（或属性），但它只对特定的子类或子类集合相关时使用。你只需将字段从超类移动到实际使用的子类中。

例如，让我们以一个包含单个字段 `engine` 的 `Vehicle` 类为例。我们有两个从它扩展出的子类：

```java
public class Vehicle {
    protected EngineType engine;
}
public class Car extends Vehicle{
}
public class Bicycle extends Vehicle{
}
```

当然，对于汽车来说，有一个引擎是有意义的，但对于自行车来说则不然。让我们将 `engine` 字段移动到唯一需要的类中：

```java
public class Vehicle {
}
public class Car extends Vehicle{
    protected EngineType engine;
}
public class Bicycle extends Vehicle{
}
```

## 提升方法

这包括将一个方法从子类移动到超类。这通常发生在多个子类共享一个共同行为时，你希望在该超类中建立这种行为以促进代码重用。以下是一个例子：

```java
public class Triangle extends Polygon{
    public Long calculatePerimeter(){
        //calculations...
    }
}
public class Square extends Polygon {
    public Long calculatePerimeter(){
        //calculations...
    }
}
```

将 `calculatePerimeter` 方法提升到超类中，并从 `Triangle` 和 `Square` 中移除是值得的：

```java
public abstract class Polygon {
    private Integer sidesNumber;
    public Long calculatePerimeter(){
        //calculations...
    }
}
public class Triangle extends Polygon{
}
public class Square extends Polygon {
}
```

## 降级方法

这与提升方法相反，当你在超类中定义了一个方法，但其行为只对特定的子类或子类集合相关时使用。你只需将方法从超类移动到实际使用的子类中。

例如，让我们再次以一个包含单个方法 `fillTank` 的 `Vehicle` 类为例。我们还从它扩展出两个子类：

```java
public class Vehicle {
    protected void fillTank() {
        //method implementation
    }
}
public class Car extends Vehicle{
}
public class Bycicle extends Vehicle{
}
```

如你容易猜到的，对于代表自行车的类来说，拥有 `fillTank` 方法没有任何意义，但只对代表汽车的类有意义。所以，我们要做的是将方法降级到 `Car` 子类中：

```java
public class Vehicle {
}
public class Car extends Vehicle{
    protected void fillTank() {
        //method implementation
    }
}
public class Bycicle extends Vehicle{
}
```

## 模板方法

在编程过程中，我们经常遇到需要将某些算法或过程应用于对象的情况，这些对象几乎完全相同，但又不完全一样。某种逻辑可能适用于整个对象类别，但这些对象可能有一些特定的特征使它们不同。在先前的例子中，我们使用了多边形的经典例子。我们可以这样说，对于多边形，周长的计算总是涉及其相应边长的总和。然而，例如，在三角形和正方形之间，边的数量是不同的。因此，在超类中有一个在子类中实现的方法，代表它们的特定特征，这可能是有用的。我相信用例子来说明这一点更容易：

```java
public class Triangle extends Polygon {
    private final Long aLength; //length of side a
    private final Long bLength; //length of side b
    private final Long cLength; //length of side c
    public Triangle(Long aLength, Long bLength, Long cLength) {
        this.aLength = aLength;
        this.bLength = bLength;
        this.cLength = cLength;
    }
    public Long getPerimeter() {
        return aLength + bLength + cLength;
    }
}
public class Square extends Polygon {
    private final Long sideLength;
    public Square(Long sideLength) {
        this.sideLength = sideLength;
    }
    public Long getPerimeter() {
        return sideLength * 4;
    }
}
public abstract class Polygon {
}
```

`Triangle`和`Square`都有`getPerimeter`方法，这个方法基本上是计算它们边长的总和；我们并不能直接调用这个方法，因为实现方式不同——一个有三个可能不同的边，而另一个有四个相等的边。我们将要实现的是一个模板方法，在这个方法中，我们将调用子类中定义的另一个方法，该方法仅返回边长的`Collection`字段：

```java
public class Triangle extends Polygon {
    private final Long aLength; //length of side a
    private final Long bLength; //length of side b
    private final Long cLength; //length of side c
    public Triangle(Long aLength, Long bLength, Long cLength) {
        this.aLength = aLength;
        this.bLength = bLength;
        this.cLength = cLength;
    }
    @Override
    protected Collection<Long> getSideLengths() {
        return List.of(aLength, bLength, cLength);
    }
}
public class Square extends Polygon {
    private final Long sideLength;
    public Square(Long sideLength) {
        this.sideLength = sideLength;
    }
    @Override
    protected Collection<Long> getSideLengths() {
        return List.of(sideLength, sideLength, sideLength, sideLength);
    }
}
public abstract class Polygon {
    public Long getPerimeter() {
        Collection<Long> sideLengths = getSideLengths();
        Long perimeter = 0L;
        for (Long length : sideLengths) {
            perimeter += length;
        }
        return perimeter;
    }
    protected abstract Collection<Long> getSideLengths();
}
```

我们创建了一个`getSideLengths`模板方法，通过在超类中合并常见的算法步骤来减少冗余，同时允许子类保持其区别。这是一个**开闭原则**（**OCP**）在实践中的生动例子。如果引入了新版本的算法，你只需创建一个新的子类，而无需修改现有代码。

# 使用枚举而不是常量

到目前为止，我们已经看到了文献中讨论的一些主要的重构技术。现在，我们进入最后一部分，在这里我们可以给你一些建议，关于如何更好地组织你的代码设计。这些可能看起来很微不足道，但它们往往会导致相当大的烦恼。本节的第一个建议，正如标题所暗示的，涉及到常量的过度使用（仅基于我的感知和经验进行彻底的研究无疑表明，这些常量 99%的情况下将是字符串）。

假设我们有一个如下所示的类：

```java
public class Itinerary {
    private String transportType;
    private String cabinClass;
    //getters and setters
}
```

在另一个类中，我们定义了以下常量：

```java
private static final String FLIGHT = "FLIGHT";
private static final String TRAIN = "TRAIN";
private static final String ECONOMY = "ECONOMY";
private static final String FIRSTCLASS = "FIRSTCLASS";
```

在一个方法中，让我们实例化我们的`Itinerary`类：

```java
Itinerary itinerary = new Itinerary();
itinerary.setCabinClass(FLIGHT);
itinerary.setTransportType(ECONOMY);
```

你看到什么奇怪的地方了吗？我们意外地将`cabinClass`和`transportType`交换了！因为我们只是在处理字符串时，一切都能编译，所以我们甚至没有意识到这一点。如果我们真的不需要创建自定义类型来表示这些概念，比如运输类型和客舱等级，我们的建议是使用枚举。

实际上，让我们引入枚举：

```java
enum TransportType {FLIGHT, TRAIN}
enum CabinClass {ECONOMY, FIRSTCLASS}
```

然后，我们可以这样重写代码：

```java
public class Itinerary {
    private TransportType transportType;
    private CabinClass cabinClass;
    public Enums.TransportType getTransportType() {
        return transportType;
    }
    public void setTransportType(Enums.TransportType transportType) {
        this.transportType = transportType;
    }
    public Enums.CabinClass getCabinClass() {
        return cabinClass;
    }
    public void setCabinClass(Enums.CabinClass cabinClass) {
        this.cabinClass = cabinClass;
    }
}
```

这样就可以避免错误地使用：

```java
Itinerary itinerary = new Itinerary();
itinerary.setCabinClass(CabinClass.ECONOMY);
itinerary.setTransportType(TransportType.FLIGHT);
```

在 Java 中，使用枚举通常比使用常量更好，因为它们提供了以下功能：

+   **类型安全**：枚举本身就是类型，这意味着它们允许你定义一组具有相同类型的不同值。这可以防止你在代码中意外使用不正确的值，从而减少运行时错误。

+   **可读性**：枚举为它们的每个值提供了有意义的名称，使代码自文档化。这提高了代码的可读性和理解性，因为你可以通过查看其名称来理解每个值的目的。

+   **IDE 支持**：IDE 提供针对枚举的特定功能，如代码补全和错误检查。当你使用常量时，你可能不会得到相同级别的支持，并且可能需要记住或查找有效的值。

+   **重构便捷性**：当你需要更改或添加值时，枚举使这变得更容易。IDE 可以自动更新代码中所有对枚举值的引用，从而在维护期间减少错误的机会。

+   **额外行为**：枚举可以具有与之关联的方法和字段，允许你封装与每个枚举值相关的行为。例如，你可以定义一个方法，为每个枚举计算特定的值或行为，从而提高代码的组织性和可维护性。

+   `EnumSet`和`EnumMap`在处理枚举时非常高效。这些集合专门针对枚举值进行定制，使你的代码更简洁、性能更优。

+   **编译时检查**：与不正确的常量值相关的错误可能仅在运行时出现，可能引发意外问题。相比之下，与枚举相关的问题，如缺失值或错误的引用，将由编译器捕获，确保在代码执行之前代码是正确的。

+   **序列化支持**：枚举具有内置的序列化和反序列化支持，这简化了将枚举值保存到文件或通过网络传输等任务。

基本上，当你在 Java 中使用枚举时，与普通的常量相比，你将获得一系列的好处。这些好处包括使你的代码更安全、更易于阅读和更易于维护。枚举还与你的开发工具配合良好，使你的生活更轻松。但这并不意味着常量没有它们的用途——它们确实有，尤其是在特定情况下。然而，对于大多数具有一些额外行为且值固定的情况，枚举是最佳选择。

# 摘要

到此，我们已完成了对重构技术的探索之旅。我们希望我们提供了一些关于如何更好地构建方法、使代码清晰易读的想法。我们探讨了有时在对象之间重新排列功能以及更好地组织数据可能是有帮助的。我们简化了条件逻辑（你知道的——那些帮助我们快速解决问题的`if`语句和开关，但可能会使我们的代码难以阅读！）。我们简化了方法调用，使用了一些古老而优秀的泛化，并简要讨论了对象的不可变性。在下一章中，我们将深入探讨如何使用自动化工具（希望！）使我们的代码更加出色。

# 进一步阅读

+   对于其他示例和用例：马丁·福勒，《重构》，Addison-Wesley

+   *空引用：十亿美元的错误*：[`www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/`](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/)
