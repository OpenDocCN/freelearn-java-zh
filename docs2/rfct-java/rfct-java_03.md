

# 第三章：代码异味

本章的标题应该是自解释的，或者可能听起来令人厌恶，但我认为解释一下“代码异味”这个词的含义很重要。这种感觉类似于当你打开冰箱时，一股奇怪的气味扑鼻而来，一些不应该存在的东西。坏气味并不一定表明有问题，但看看它总是值得的。可能有问题，也可能没有，但忽略它并不是一个好主意。

在我们的代码库中，情况也是一样的。代码异味是一个潜在的问题，是代码中让我们皱眉的情况。它们是非常具体和可观察的情况，是我们项目中的重复模式。它们表明，在问题变得更大之前，需要尽快修复某些东西。就像一般的坏代码一样，有异味的代码可能导致广泛的不效率、有限的代码可扩展性和可理解性，以及性能问题和潜在的 bug。

我们需要学习如何识别和避免代码异味；考虑到我们如何专注于任务和截止日期，它们的生产比人们想象的要容易得多。重要的是，再次强调，花时间审查我们的工作并识别潜在问题，即从我们的代码行中产生的坏异味。随着时间的推移，通过实践和经验，你会建立一个心理指南针，就像经验丰富的寻宝者发现隐藏的宝石一样，指出编码陷阱。有时，它甚至在你完成编码之前就会引导你。

已经有许多代码异味被我们的**开发者前辈**们整理出来了；我认为在这里列出所有并不合适。我会告诉你我认为最常见且容易产生的那些。这个选择完全是基于主观标准，所以请随意——事实上，我推荐你——深入阅读**进一步阅读**部分。许多这些代码异味都与**清洁代码**和**SOLID 原则**相关。如果你跳过了前面的章节，可能回去看看是个好主意。

我们将从重复代码开始，这是几乎所有代码库的烦恼。它通常是由于开发者为了赶工期而采取终极捷径：复制粘贴。然后我们将转向长方法和大型类。当你在一个类中发现第*4215*行的 bug 时，真正令人沮丧的是，不是因为 bug 本身，而是因为该类包含的行数太多！

然后，我会向你展示，当`switch`语句重复时，它们也可能成为问题；这是一个面向对象编程可以轻松解决的问题！在讨论面向对象编程时，为什么不通过专用对象来表示我们领域的概念，而不是依赖于语言原语呢？有一个名为**原始主义**的反模式，我们将解决它。

除了大类之外，还有一些类或方法过度依赖其他人的方法；换句话说，它们就是无法管好自己的事！我们将探讨什么是**特征嫉妒**方法以及如何解决它们可能造成的潜在危害。我们还将讨论两个与代码更改密切相关且相互交织的反模式：**分歧变更**和**雷射枪手术**。根据我们在引入新功能或进行一般修改时需要在代码中做出多少更改，我们可能会遇到其中之一或另一个。

最后但同样重要的是，我们将讨论所谓的**神对象**；这个反模式有点像那个总是坚持做所有事情、承担所有责任的朋友；但然后每个人都依赖他们做所有事情，这对他们来说太沉重了。不应该将过多的责任分配给单个实体，对吧？我会告诉你这个问题在哪里，以及如何尝试解决这个问题。

在本章中，我们将学习以下主题：

+   重复代码

+   长方法

+   大类

+   开关

+   原始迷恋

+   中间人

+   消息链

+   特征嫉妒方法

+   分歧变更

+   雷射枪手术

+   God object

如前所述，代码异味最终只是异味。就像冰箱里的恶臭并不一定意味着你的食物有问题（想想蓝纹奶酪吧！），代码中的恶臭也不一定意味着有问题。然而，有一个潜在的问题值得我们考虑。在我看来，最重要的是要意识到我们代码库中的内容。我们不应该把每一个代码异味都看作是世界上最紧迫的事情，因为可能并不是。然而，不容低估它们，因为如果我们忽视了一个又一个的代码异味，最终我们得到的代码库要么是无法维护的，要么是维护成本极高。这不仅从经济角度适用，也适用于个人挫败感。

# 重复代码

让我们从马丁·福勒所说的*臭味游行*（我忍不住要提到它，因为这个词让我忍俊不禁）开始，讨论重复代码。重复代码直观上是一个异味，我们的软件工程师本能很快就会学会拒绝它，因为它会给我们带来危害。让我们尝试列出一些为什么重复代码会危害我们代码库健康的原因。

显然，复制粘贴代码是首先要避免的，至少在 99%的情况下。从一个类中取出一段代码，并完全如它在另一个类中使用，是容易避免的。通过稍作停顿进行反思，我们很可能可以提取一个方法来代替重复的代码。以这种方式集中代码确保了更好的代码可维护性。只需想想维护（例如，修复代码库中的错误）散布在代码库中的每一块复制粘贴的代码是多么的噩梦（最可怕的部分是开发者可能没有意识到所有重复的实例）。一个噩梦。一个在许多地方成为现实的噩梦，相信我。

有时，重复的代码存在是因为程序员对他们正在做的事情没有清晰的理解，他们没有拥有源代码，或者他们想要修改它。换句话说，他们难以掌握他们正在工作的代码。正如之前提到的，进行一些重构以更好地理解我们正在处理的代码是有用的。很可能即使以这种简单（比喻地说）的方法，你也能更有效地处理你正在处理的事情。

通常，代码质量会因为重复代码而受损；代码变得更长，有时可读性更低。有时可能无法重构重复的代码块，但目标应该是尽可能减少技术债务。这有助于提高代码的质量。

重复代码的最基本形式发生在你在一个类的两个方法中遇到相同的表达式时。为了解决这个问题，你所需要做的就是提取一个方法，并从两个位置调用这段代码：

```java
public void printCharacters() {
    SwCharacter darthVader = new DarthVader();
    SwCharacter obiWan = new ObiWanKenobi();
    System.out.println("Name: " + darthVader.getName());
    System.out.println("LightSaber color: " +
        darthVader.getLightSaberColor());
    System.out.println("Birth place: " +
        darthVader.getBirthPlace());
    System.out.println("Name: " + obiWan.getName());
    System.out.println("LightSaber color: " +
        obiWan.getLightSaberColor());
    System.out.println("Birth place: " +
        obiWan.getBirthPlace());
}
```

之前的代码片段中存在一些重复，你可以轻松地发现。我们将提取一个`printDetails`方法并使用它两次：

```java
public void printCharacters() {
    SwCharacter darthVader = new DarthVader();
    SwCharacter obiWan = new ObiWanKenobi();
    printDetails(darthVader);
    printDetails(obiWan);
}
private static void printDetails(SwCharacter darthVader) {
    System.out.println("Name: " + darthVader.getName());
    System.out.println("LightSaber color: " +
        darthVader.getLightSaberColor());
    System.out.println("Birth place: " +
        darthVader.getBirthPlace());
}
```

现在，IDE（集成开发环境）完全能够标记重复的代码（例如，在 IntelliJ IDEA 中，这是一个内置功能），并建议进行小的重构以提取方法。我们将在本书的后面部分讨论如何充分利用 IDE 和其他工具。

我们现在将转向长方法。

# 长方法

这是一个非常典型的代码异味，非常隐蔽，并且经常被低估。尽管这可能看起来是一个微不足道的话题，但我愿意分享一些想法。

代码的阅读量远大于编写量，因此花点额外的时间来缩短一个方法可以带来巨大的回报。当你需要修改一个长方法时，你必须阅读并理解代码的每一行，以确保安全地做出更改。在这种情况下，我们不得不将这个方法所做的一切工作都加载到大脑中，只是为了理解发生了什么，更不用说有能力修改它的任何部分了。

长代码很可能做不止一件事：根据**单一职责原则**，一个方法理想上应该只专注于做一件事。当你有这么多行代码时，几乎不可能实现这一点。

了解何时“长”变成“太长”是有用的。10 行？20 行？100 行？很难制定一个固定的规则。这有点像说蛋糕太甜或意大利面太咸的时候；我们本能地认识到过度，但我们可能无法就确切的数字达成一致。

我的建议是跟随你的直觉，最重要的是在团队内部达成共识。例如，你可以决定 20 行是最大限制，或者它不应该超过一屏的高度（当然，这取决于屏幕分辨率、字体大小等）。另一件酷的事情是使用格式化标准，即使是通过使用自动工具来处理它；关于这一点，本书后面会详细介绍。简而言之，这取决于。也可能存在这样的情况，一个方法很长，但只包含配置，这些内容非常易于阅读，不需要太多努力就能理解。在这种情况下，你可能甚至不在乎它。

此外，我想从马丁·福勒的《重构》中提出一个启发式技巧：每当你在代码中感到需要插入注释来解释时，写一个方法代替。这个方法将包含你打算注释的代码，但函数本身的名称将解释代码背后的意图，而不是其功能。

以下是一些在长方法情况下有帮助的小技巧。

## 用查询方法替换临时变量

大多数时候，从你的长方法中提取一个方法或函数应该能解决问题；寻找函数中配合得好的部分，创建一个新的方法。有时你有一个具有许多参数和临时变量的函数，它们可能成为提取过程的障碍；提取函数会导致将许多参数传递给提取的方法，这会像之前一样难以阅读。因此，用查询函数替换那些临时变量可能是有用的：

```java
Double basePrice = this.getCostPerKm() *
    this.getDistanceInKm();
if (passengerIsChild()) {
    return basePrice * 0.5;
} else {
    return basePrice;
}
```

在这个代码片段中，`basePrice`变量用于存储定价操作的结果，然后再次用于计算最终价格，对于儿童乘客，最终价格将折扣 50%。删除临时变量并将计算内联，编写一个名为`calculateBasePrice`的方法（查询函数）可能是个好主意：

```java
if(passengerIsChild()){
    return calculateBasePrice() * 0.5;
}else{
    return calculateBasePrice();
}
```

在下一节中，我们将探讨参数对象。

## 参数对象

也可能发生这样的情况，你有一些具有长或“重复”签名的函数或方法，例如以下代码片段中的`calculateFinalPrice`和`discountPrice`方法：

```java
public Double calculateFinalPrice(Double amount, String
    currency){ ... }
public Double discountPrice(Double amount, String
    currency){ ... }
```

你可以看到，我们有两个具有相同签名的方法被重复使用。同样重要的是要注意，在这种情况下，`amount`和`currency`参数必须始终一起使用，因为它们代表一个价格。一个很酷的解决方案是将“参数对象”或“请求对象”传递给这些方法（我听说它被许多不同的方式称呼）：即一个复杂对象，代表所有有意义的参数。鉴于金额和货币代表一个价格，我们为什么不创建一个`Price`类呢？我们在以下片段中这样做：

```java
public Double calculateFinalPrice(Price price){ ... }
public Double discountPrice(Price price){ ... }
class Price {
    private Double amount;
    private String currency;
}
```

在下一节中，让我们用一个命令来替换一个函数/方法。

## 用命令替换函数或方法

如果你尝试提取函数并创建参数对象，但你仍然剩下过多的临时变量和参数，你可以做的是用命令替换函数。假设你有一个以下方法，它接受很多参数并且非常长：

```java
public Double calculatePrice(Double basePrice, Integer
    adults, Integer children, Integer infants) {
    Double calculatedPrice = 0.0;
    // long body...
    return calculatedPrice;
}
```

如果我们实现一个`PriceCalculator`对象，一切都会变得清晰；这样，你就用一个命令替换了一个函数。所有参数都在构造函数的开始处传递。你可以通过每次传递所有参数来重用这个组件（实际上，这也使得你的组件更具可重用性）。方法的逻辑在`calculatePrice()`方法中执行，它不接受任何参数；可读性也得到了提高：

```java
class PriceCalculator {
    private final Double basePrice;
    private final Integer adults;
    private final Integer children;
    private final Integer infants;
    public PriceCalculator(Double basePrice, Integer
        adults, Integer children, Integer infants) {
        this.basePrice = basePrice;
        this.adults = adults;
        this.children = children;
        this.infants = infants;
    }
    public Double calculatePrice() {
        Double calculatedPrice = 0.0;
        // long body...
        return calculatedPrice;
    }
}
```

当处理代码提取，需要将其放入不同的方法中时，有时仅仅识别出想要分离的代码就变得很困难。你如何找到要提取的代码块？一个实用的方法是检查注释。它们通常指示这些有意义的部分。如果你遇到一个带有解释其目的的注释的代码块，你可以用注释命名的函数（然后你可以决定是否保留注释）。即使是一行代码，如果它需要解释，也值得提取。

## 分解条件

同样，还有其他需要提取的代码的指示器；例如，条件和循环也提供了有用的提取提示。如果你有一个条件表达式，比如以下片段中的`if`，应用“分解条件”技术可能很有用：

```java
if (1 <= rowNumber && rowNumber <= 30) {
    price = basePrice * plane.getFirstRowsCharge();
} else {
    price = basePrice * plane.getStandardCharge();
}
```

这段代码片段根据行号计算飞机座位的票价（这或许没有太多意义，但在这个例子中很有用！）。如果座位位于前 30 行，我们需要支付不同的票价。这里有一个`if`语句，在这里很简单，但你要想象在现实世界场景中，条件可能更加复杂，涉及更多的参数和对象。在重构的代码片段中，我们隔离了一些逻辑，包括条件，以提高可读性和潜在的代码重用（并且在未来，你可能会通过向第三方系统或甚至规则引擎发起请求来外部化条件计算）：

```java
if (firstRows()) {
    price = firstRowsPrice();
} else {
    price = standardPrice();
}
```

但条件不仅仅是`if`，还有`switch`（在某种程度上，`switch`是伪装的`if`）。如果你遇到一个冗长的`switch`语句，可以使用我们在上一节中介绍过的“提取函数”方法，将每个情况转换为单独的函数调用。

## 拆分循环

有时，循环也可能成为问题；例如，常见的循环体执行多项操作。在这种情况下，你应该为循环本身和其中的代码创建一个独立的方法。如果你难以找到一个合适的提取循环名称，可能是因为它执行了两个不同的任务。在这种情况下，不要犹豫，使用以下称为**拆分循环**的技术，将任务分离到不同的部分：

```java
Integer totalBaggageNumber = 0;
Double totalPrice = 0.0;
for (Passenger passenger : passengers) {
    totalBaggageNumber += passenger.getBaggageNumber();
    totalPrice += passenger.getPrice();
}
```

之前的代码片段由一个非常基本的`for`循环组成。即使在这两行中，也隐藏了一个问题！实际上，你在同一个`for`循环中做了两件不同的事情。我都能听到你在说，“*哎呀，这只是两行而已!*”但——再次——我在这里尽可能地简化事情。现实往往更加复杂，但问题是一样的。通过拆分循环重构代码可以使代码更清晰、更可重用；如果你需要将部分逻辑移动到其他地方，现在应该会简单一些（我们也可以使用 Java 流，但我们想尽可能保持简单）：

```java
for (Passenger passenger : passengers) {
    totalBaggageNumber += passenger.getBaggageNumber();
}
for (Passenger passenger : passengers) {
    totalPrice += passenger.getPrice();
}
```

许多程序员不喜欢这种重构，因为它会让你运行循环两次。但请记住，我们正在将重构与优化分开。首先，让你的代码变得整洁，然后你可以优化它。如果你发现遍历列表会导致速度减慢，你可以轻松地将循环合并。但说实话，遍历一个大列表通常并不是真正的问题，拆分循环实际上可以打开更好的优化机会。

# 大型类

这又是一个非常常见的代码异味。你有多少次发现自己正在处理拥有数千行代码的类？这不是任何人的错，只是类开始时很小，非常合理地逐渐包含了更多的信息和功能。这并没有什么问题；重要的是要意识到这一点并进行重构。

当一个类中有太多字段（实例变量）时，这是一个实际的方法来注意到这个类太大。如果一个类中有太多字段，它很可能会做太多事情；除了有太多的职责外，通常在这种情况下，代码重复成为了一个问题。

你可以创建（提取）一个新的类来将几个变量组合在一起。选择那些自然结合在一起的变量并将它们放入一个组件中。例如，如果你看到一些名为 `priceAmount` 和 `priceCurrency` 的参数，它们可能意味着是同一个组件的一部分。一般来说，如果你注意到一个类中变量子集的常见前缀或后缀，这表明了创建组件的可能性：

```java
class Order {
    private String name;
    private String surname;
    private String streetName;
    private String streetType;
    private String streetDirection;
    private Double priceAmount;
    private String priceCurrency;
   //constructor and getter methods
}
```

`Order` 类包含许多字段，每个字段都代表订单的不同属性。然而，如果你仔细观察，很明显，其中一些字段可以从逻辑上“分组”在一起，因为它们属于订单属性的特定方面。例如，`name` 和 `surname` 字段可以看作是代表买家的信息（为了简化概念）。同样，`streetName`、`streetType` 和 `streetDirection` 字段与订单的送货地址相关，而 `priceAmount` 和 `priceCurrency` 提供了关于销售价格的信息。

基于这些见解，我们可以通过将某些字段分组来重新结构化 `Order` 类，创建新的 **实体**。这些实体可以在整个项目中有效地重用，增强代码组织和可维护性。

重新结构化的 `Order` 类应该看起来像这样：

```java
class Order {
    private User orderBuyer;
    private Address shippingAddress;
    private Price orderPrice;
   //constructor and getter methods
}
```

当然，它应该将所有这些字段“分布”在不同的类中：

```java
class User {
    private String name;
    private String surname;
    //constructor and getter methods
}
class Address {
    private String streetName;
    private String streetType;
    private String streetDirection;
    //constructor and getter methods
}
class Price {
    private Double priceAmount;
    private String priceCurrency;
    //constructor and getter methods
}
```

如果使用继承对组件来说是有意义的，你可能会发现提取超类或用子类替换 `type`（也称为提取子类）等技术通常更容易应用。以下是一个非常常见的情况；我们有一个类，它接受一个 `type` 参数，因为根据其值，该类必须以稍微不同的方式行为：

```java
public Transport createTransportation(String name, String
    type){
    return new Transport(name, type);
}
```

我敢打赌，`Transport` 类中隐藏着一些 `if` 语句！但是面向对象编程伴随着继承，那么我们为什么不使用它呢？以下是一个例子：

```java
public Transport createTransport(String name, TransportType
    type) {
    switch (type) {
        case PLANE -> {
            return new Plane(name);
        }
        case TRAIN -> {
            return new Train(name);
        }
        case BUS -> {
            return new Bus(name);
        }
    }
    return null;
}
```

在这个重构的代码片段中，我们有两个主要改进。我们将 `type` 类从 `String` 改为名为 `TransportType` 的枚举；这允许我们以更安全的方式执行后续的 switch 语句，因为我们将在不考虑类型的一些可能值时被警告（可能是一个 IDE）。然后，我们为每种类型的运输创建了一个特定的类，避免了混淆；当然，所有这些类都将实现相同的接口。

除了拥有大量实例参数外，一个太大的类通常不是很好；它是重复代码、代码不清晰、混乱和绝望的温床。一个简单而有效的方法是从消除类内部的重复开始。简单来说，你可以寻找有意义的代码组合，并为它们编写独立的方法。缩短一个非常大的类中的方法。也许方法数量会增加，但你可能会意识到，如果你可以将它们移动到另一个类中，如果这些方法有意义并且关注单一方面，那么你很可能会这样做。在非常大的类中，非常常见的是有重复的代码或类似的重复。在那个点上，我们可以从 100 行的方法中提取出两到三个 10 行的方法，再加上一些额外的代码来调用它们。最困难的部分可能是理解如何拆分代码。这里有一个非常简单的技巧，通常很有效：这样的类的**客户端**通常可以提供有价值的拆分提示。仔细观察并执行一些代码的静态分析（即打开代码并遍历它。还有自动工具可以做到这一点；我们稍后会遇到它们）以查看客户端如何仅使用类的一部分功能。每个不同的功能子集都可能成为一个单独的类。

# `switch` 语句

简单来说，`switch` 语句本身并没有问题。事实上，我们发现它非常优雅且易于理解。许多编程语言甚至提供了更高级的 `switch` 语句形式，它们可以使用更复杂的代码作为基础。它们可以简化代码并替换掉丑陋的嵌套 `if` 语句。我必须承认，我们对 `switch` 语句情有独钟。

然而，当我们遇到面向对象编程中的**重复** `switch` 语句时，问题就出现了。我们认为这有几个原因使其成为问题。`switch` 语句违反了每个部分的 `case` 语句，这导致修改现有代码，这与原则相悖。此外，`switch` 语句的维护可能具有挑战性。随着新需求的产生，`switch` 语句可能会变得更加复杂，使得代码更难管理。`switch` 语句的另一个问题是，在某些情况下可能存在冗余代码。

在这种情况下，解决方案相当简单——使用多态：

```java
public String printLightSaberColor(SwCharacter character){
   switch (character.getName()) {
       case "LUKE" -> {
           return "GREEN";
       }
       case "OBI-WAN" -> {
           return "BLUE";
       }
       case "DARTH VADER" -> {
           return "RED";
       }
   }
}
```

我假设你已经熟悉这些角色，并且知道什么是**光剑**。如果不是，我强烈建议你立即在 Google 上搜索它们并观看那些电影。前一段代码可以被重构为以下类/方法，其中我们设计了一个抽象类 `SwCharacter`，它将被扩展为每个特定角色。`printLightSaberColor` 方法将接受一个“通用”的 `SwCharacter` 并调用相应的 `getLightSaberColor` 方法的实现。在以下代码中，为了简化，我们将省略一些样板代码，例如构造函数或其他获取器：

```java
public String printLightSaberColor(SwCharacter character){
    return character.getLightSaberColor();
}
public abstract class SwCharacter {
    private final String name;
    protected SwCharacter(String name) {
        this.name = name;
    }
    public abstract String getLightSaberColor();
}
public class LukeSkywalker extends SwCharacter {
    protected LukeSkywalker() {
        super("Luke Skywalker");
    }
    @Override
    public String getLightSaberColor() {
        return "GREEN";
    }
}
public class ObiWanKenobi extends SwCharacter{
    public ObiWanKenobi() {
        super("Obi-Wan Kenobi");
    }
    @Override
    public String getLightSaberColor() {
        return "BLUE";
    }
}
public class DarthVader extends SwCharacter{
    public DarthVader(String name) {
        super(name);
    }
    @Override
    public String getLightSaberColor() {
        return "RED";
    }
}
```

值得注意的是，在这个特定案例中，我们增加了代码行的数量。然而，在我们看来，这是完全可以接受的，因为我们提高了可读性。在我给出的例子中，我尽量保持简单，大部分时间使用基本的类型，如`int`或`String`。但这种方法并不总是最佳选择。这取决于具体情况。有时，过度依赖语言的基本类型可能会导致一种被称为**原始类型迷恋**的代码问题。

# 原始类型迷恋

要了解什么是原始类型迷恋以及如何处理它，让我们回顾一下编程语言中的原始类型是什么。原始类型就像语言自带的自定义类型。你可以把它们想象成直接从制造商那里来的。每种语言都有自己的原始类型集合，既然我们在这里讨论 Java，让我们关注一下它在 Java 中的工作方式。

在 Java 中，`java.lang.String`类。尽管`String`类在技术上不是一个原始数据类型，但在语言中得到了特殊处理，所以你可以将其视为一种。

原始类型迷恋是指代码过度依赖基本数据类型。这意味着一个简单的值负责类逻辑，缺乏类型安全。用更简单的话说，这是一种在特定关注区域使用基本类型来表示对象的坏习惯。

以表示网站 URL 为例。通常，我们将其存储为字符串。然而，问题在于 URL 比简单的字符串具有更多的细节和独特的特征。它包括诸如方案、查询参数和协议等元素。当我们将其存储为字符串时，我们失去了直接访问这些 URL 特定组件（URL 的基本部分）的能力，而不需要编写额外的代码。我们可以使用`java.net.URL`代替。

字符串往往是这种问题的滋生地。以电话号码为例。它不仅仅是随机字符的集合。在许多情况下，适当的数据类型可以在需要显示在用户界面时提供一致的显示逻辑。将这种类型表示为字符串会引发一个如此常见的问题，以至于它们有时被称为*字符串类型化*变量。

如果你仔细想想，解决这个问题其实很简单；必要的是用对象替换原始类型。以下是一个例子：

```java
String url = "https://www.packtpub.com/";
String protocol = extractProtocol(url);
String host = extractHost(url);
```

这可以变成以下形式：

```java
URL url = new URL("https://www.packtpub.com/");
String protocol = url.getProtocol();
String host = url.getHost();
```

如果你好奇`URL`类是如何工作的，可以查看其文档（`java.net.URL`类包含在标准 Java 库中）。

如果你有一组可以一起使用的原始类型，你可以通过提取一个类来将它们分组，就像我们在*大型*类部分中讨论的地址示例一样。

现在我们已经处理了原始类型迷恋，让我们来谈谈那些不太愿意工作，而让其他事物做所有工作的类。这只是一个玩笑的说法，我们将讨论一种被称为**中介**的代码问题。

# 中介

中间人是当我们在一个类或更一般地说，一个对象中只做一件事，那就是...将工作委托给其他人时出现的代码异味。我们并不是建议一个类应该只有一个方法才能被认为是“中间人”。相反，它的每个方法应该只调用其他方法。因此，自然而然地出现了一个问题：为什么在我们的代码库中保留一个什么也不做的类？事实上，几乎没有理由。让我们删除这个类或方法，并确保客户端直接调用“目的地”。这样，我们将消除不必要的复杂性，并确保当我们修改或添加目标对象的特性时，我们不必也修改我们的中间人。

例如，假设我们有一个如下结构的类：

```java
public class Person {
    private Address address;
    public City getCity() {
        return address.getCity();
    }
    public Address getAddress() {
        return address;
    }
}
```

一个“客户端”方法将按以下方式使用它：

```java
Person person = getPerson();
var person = person.getCity();
```

你可以很容易地注意到`getCity()`方法仅仅是一个代理，用于其他类。那么为什么不直接使用其他类呢？我们的例子就会变成：

```java
var city = person.getAddress().getCity();
```

因此，我们可以消除由`Person`类中的`getCity()`方法组成的中间人。

然而，这种重构可能出人意料地导致另一种代码异味，即消息链！我们是不是最终陷入了一个时间循环？事实是，这取决于。这些重构确实必须被适当地权衡，试图选择较小的恶（同时考虑可能这个建模自己的类有缺陷，可能需要某种方式的变化）。

# 消息链

当一个客户端请求一个对象，该对象随后请求另一个对象，依此类推时，就会发生**消息链**。这使得所有对象都依赖于被调用对象的架构，而这些对象的任何变化都会必然影响整个调用链，在每个它们被调用的点上。基于上一节中的例子，我们可能会有如下情况：

```java
var cityName = person.getLocation().getAddress().getCity().getName();
```

解决方案是隐藏“委托”，即调用其他方法的方法。我们可以将前面的例子转换为如下形式：

```java
var cityName = person.getCityName();
```

其中调用链隐藏在`getCityName()`方法中。

然而，这种重构如果走向极端，可能会导致中间人异味！我们是不是陷入了一个像克里斯托弗·诺兰那样的时间循环？事实是，这取决于。这些重构确实必须被适当地权衡，试图选择较小的恶（同时考虑可能这个建模自己的类有缺陷，可能需要某种方式的变化）。

选择中间人而不是消息链的优势在于，在单元测试期间需要更少的模拟。当你必须为它们的直接和间接依赖提供模拟时，测试类变得具有挑战性。此外，它有助于分离关注点。一个具有*A*组件并需要*C*组件的代码应该理想上对*B*组件的参与一无所知。这有助于更好的模块化。虽然消息链的主要论点是避免在中间编写样板代码，并且可能存在一些情况下这样做是有意义的，但一般指南应该倾向于更喜欢中间人。

选择中间人而非消息链符合迪米特法则的原则，通常总结为“只与你的直接依赖项交流。”这一指南封装了一种设计哲学，倡导面向对象系统中封装和降低耦合。我们曾在*第二章*中讨论过迪米特法则。

# 特性贪婪方法

当我们尝试根据我们的用例和领域将代码库分解成有意义的组件时，我们实际上是在将代码划分为*区域*。我们需要小心的一件事是最大化区域内的交互，同时相反地，最小化不同区域之间的交互。“特性贪婪”就像代码中的一个警告标志，描述了当对象访问另一个对象的字段来执行操作，而不是简单地指示对象做什么时的情况。

作为一个简单的例子，让我们考虑一个基于支付请求计算价格的方法：

```java
public Double calculatePrice(PaymentRequest paymentRequest);
```

一切似乎都很顺利，直到我们意识到我们需要订单和一些用户信息来计算价格。那时，我们开始编写其实现，例如，我们必须检索与支付请求相关的订单；在方法的第一行，我们将编写如下内容：

```java
Order order = orderRepository.findById
    (paymentRequest.getOrderId());
```

很可能还需要用户信息。因此，在获取订单后，我们可能会添加一行代码来检索用户，因为我们需要关于他们的某些信息：

```java
User user = userRepositoty.findById
    (paymentRequest.getUserId());
```

从这些对象中，我们将提取一些特定的属性来完成支付。例如，我们将编写如下内容：

```java
order.getItems();
order.getOrderDate();
user.getLevel();
```

再次，我们面临大量查找/获取方法的调用。在方法中调用大量其他对象的 getter 方法以检索或计算某些值是非常常见的。在这种情况下，你的 bean，即你的 Java 类，很可能有很多*协作者*，也就是说，有很多其他类通过构造函数注入到你的类中。

在最基本的情况下，你可以简单地将你正在使用的方法从一个类转移到另一个类（如果需要，修改其名称）。如果只有方法的一部分需要访问另一个对象的数据，你可以将其提取为单独的方法。然而，并非所有情况都那么简单；有时，一个函数依赖于多个模块的功能，因此它变成了一个问题：它应该属于哪个模块。一个经验法则（如 Martin Fowler 所建议）是确定拥有最多数据的模块，并将函数放在那里。这个过程通常通过提取方法来实现，将函数分解成更小的部分，可以放置在不同的位置。

有时，即使这个指南不适用，你也可以选择使用书中描述的*四人帮*的策略或访问者模式。然而，提供这些模式的详细解释超出了我们的范围。对于更深入的解释，你可以参考*进一步阅读*部分。基本原理保持不变：将经历变化的元素分组在一起。通常，数据和依赖于该数据的行为会同时改变，尽管存在例外。在这些情况下，重新定位行为可以确保变化集中在单一位置。策略和访问者提供了一种通过隔离需要覆盖的具体部分来修改行为的方法，尽管这种方法引入了一些额外的间接性。

# 分歧性变化

作为软件工程师，我们知道我们的工作产品是柔软和灵活的。曾经，一位大学教授告诉我们，“*我们是我们唯一被要求多次更改项目的工程师，即使项目已经完成。想象一下用桥梁来做这件事。这是我们力量的源泉，也是我们的诅咒。*”

如我们从敏捷哲学中得知，*变化是好的*，但需要被控制和管理的。回到代码异味，当我们需要经常修改某个类时，即使我们接触的是看似无关的方面，我们也会出现分歧性的变化。想象一下这样一个类，它处理旅行预订网站（那些你可以输入你想什么时候出发、什么时候返回、你想去哪里，然后他们会给你提供无数选择——飞机、火车、公共汽车等等）的搜索结果。在某个时候，我们需要更改应用程序依赖的数据库。因此，我们修改了类 *X*。然后，我们需要集成一种新的交通方式，我们必须再次修改类 *X*。新的支付方式？再次，类 *X* 需要修改。

所有这些方面都不相关，但类 *X* 总是在中间！当模块（或类、或项目）因各种原因频繁修改时，就会发生分歧性变化，导致不同上下文在单一位置混合。为了提高我们的编程体验，我们可以通过将这些上下文分离到不同的模块中来改进它。

最可能的原因是违反了**单一职责原则**。一般来说，我们要么有一个有缺陷的编程结构，要么过度使用了复制粘贴。因此，有必要创建适当的模块并相应地移动方法。有时可能需要提取临时方法（也许还需要重新定位它们）或者甚至提取整个类。

有时候，我们会遇到一种特殊情况，其中两个方面，错误地纠缠在一个类中，自然形成一个序列。一个经典的例子是我们从数据库中获取数据，处理它，然后将其提供给一个使用它的函数。在这种情况下，我们可以使用一种称为**分割阶段**的重构技术。在这个重构过程中，我们将复杂的计算分解为两个阶段（或更多）。在第一阶段，我们计算一个结果，然后我们将这个结果和一些中间数据结构传递到第二阶段进行进一步处理。

例如，看看下面的代码片段：它从一个数据库中检索一些飞行数据数组，将数组中的一个元素解析为双精度浮点数，然后使用一些输出结果进行进一步的计算...重点是它一个接一个地做了很多不同的事情：

```java
String[] flightData =
    getFlightDataFromDatabase(flightId).split("-");
double basePrice = parseDouble(flightData[3]);
double totalPrice = basePrice * parseDouble(flightData[0])
    + basePrice * parseDouble(flightData[1]) + basePrice *
        parseDouble(flightData[2]);
```

很好地重构这段代码，让它变得更易于阅读，并且尽可能模块化，即使这个术语在讨论一小段代码时可能不太合适。无论如何，我们将把这段代码分成几个阶段，几乎就像是一个工作流程：我们将调用`getFlightDataFromDatabase`（不立即分割结果；这是在下一行代码中需要的）。当然，`flightData`字符串并不太方便（我们假设我们无法更改返回类型，因为我们正在使用第三方库），所以我们将使用`parseFlightData`将其解析为`FlightInfo`对象，这个对象以非常方便的方式处理飞行信息。最后，我们将最终实现`calculateTotalPrice`，使用`flightInfo`对象。看看这段代码——即使是四行代码——如何变得更容易阅读：

```java
String flightData = getFlightDataFromDatabase(flightId);
FlightInfo flightInfo = parseFlightData(flightData);
double totalPrice = calculateTotalPrice(flightInfo);
...
double calculateTotalPrice(FlightInfo flightInfo) {
    var basePrice = flightInfo.getBasePrice();
    return basePrice * flightInfo.getAdultsCount() +
         basePrice * flightInfo.getChildrenCount() +
            basePrice * flightInfo.getInfantsCount();
}
```

当你需要更改的不是经常需要更改的单个类，而是分散在整个项目中的代码时，你就被迫在许多不同的地方修改代码。这就是你有了被称为*集束式手术*的代码异味。

# 集束式手术

虽然一开始可能觉得这个词有点暴力，但说实话，我觉得这个词也很幽默；这就是我为什么真的很喜欢它的原因。有时候，我听到它被称作*用火箭筒打苍蝇*。基本上，它是发散变化的反义词。

当你处理集束式手术时，这意味着你必须对你的代码库进行大量更改，只是为了解决看似简单的问题。通常，你会发现自己在修改看起来相当相似、直接复制粘贴或具有类似目的的代码。这个术语指的是外科医生的工作，这种工作非常精确，使用手术刀，以尽可能最少侵入的方式进行。现在，将手术刀的图像替换为霰弹枪，...嗯，你就能理解这个意思了。

可能存在几个例子或原因会导致出现集束式手术；让我们快速浏览一些，以便我们可以预防这个问题：

+   **复制粘贴**：这是首先想到的例子；你做了一些老式的代码复制粘贴！这意味着，当然，对一些复制的代码进行更改需要你对每个复制的副本都进行相同的更改。实际上，这很简单。

+   **过度分层**：有时，我们的应用程序似乎过于分层。例如，想象一个基本的 CRUD 应用程序，它使用了多个层，包括数据传输对象、数据访问对象和领域对象。每次你想向数据库添加一个表时，你不仅需要在所有四层中添加内容，还要处理那些层中的所有属性包对象。这是另一种**散弹枪手术**的情况。（但世界并非全黑或全白；有许多灰色地带。有时，具有良好定义的层的应用程序有助于隔离概念和责任。所以，要小心；我并不是说在架构中设计多个层本质上就是坏的。你总是需要找到正确的平衡，就像在生活中一样！）

+   **分层有限**：就像有时应用程序的设计中存在太多层一样，也有时候责任过度分离：一个类用于常量，另一个用于静态变量，等等。然后，当你需要做出更改时，你会发现自己不得不触及它们所有！这有没有发生在你身上？这对我来说经常发生。然后，你又回到了手术室，使用散弹枪。

你应该关注这些问题，因为随着你需要编辑代码库的更多地方，变化变得更加耗时。*合并冲突*的可能性更大，因为更多的人在不同的位置接触代码，使得协作项目变得更加困难。你也更有可能因为需要记住在多个地方更改代码的认知负荷而引入错误。此外，由于知识重复和需要额外的结构来连接不同的部分，你最终会有更多的代码。最后，对于新团队成员来说，随着开发变得像在代码库中寻宝一样，学习曲线更高。

当你的逻辑遍布整个代码库时，你可以简单地将你的方法或字段移动到同一个类或模块中。通过这样做，你确保相关的软件元素聚集在一起，使其更容易定位和理解它们之间的联系。所有相关的上下文都会集中在一个地方，增强封装，并使软件的其他部分减少对这个模块的特定依赖。

另一种解决 Shotgun Surgery 问题的有趣方法是采用内联重构。换句话说，合并方法或类以巩固不良分离的逻辑可能是有益的。这可能会导致方法过长或类过大；我知道我们刚刚说过这会不好，但这种情况只会是暂时的（但非常有用），这样你就可以随后使用提取来将其分解成更连贯的部分。当然，我们通常更喜欢代码中的小函数和类，但不要犹豫，创建一个大的作为重组的中间步骤。

将方法内联意味着，基本上，是做与提取方法相反的事情。假设我们有以下情况：

```java
public Double calculatePrice(Integer passenger, Double
    baseFare){
    var basePrice = passenger * baseFare;
    var basePriceWithVAT = VATCalculator.addVat(basePrice);
    return basePriceWithVAT;
}
class VATCalculator {
    public static Double addVat(double basePrice) {
        return basePrice * (1 + DefaultValues.DEFAULT_VAT);
    }
}
class DefaultValues {
    public static final Double DEFAULT_VAT = 0.22;
}
```

你创建了一个单独的类，并使用静态方法来实现增值税添加。假设这个方法只使用一次，这似乎有点过多；这里的建议只是将静态方法的内容内联，避免创建一个单独的类，这会使代码更难阅读。你可以简单地这样做：

```java
public Double calculatePrice(Integer passenger, Double
    baseFare){
    var basePrice = passenger * baseFare;
    return basePrice * (1 + 0.22);
}
```

无论如何，真正的要点是在工作中始终保持对 Shotgun Surgery 的关注。不断问自己，未来的维护程序员（这可能是你未来的自己）是否需要枪管或手术刀来做出预期的更改。如果答案是枪管……那么是时候重新思考你的方法了。

# God 对象

在本章提到的所有反模式（还有很多；我鼓励你深入研究*进一步阅读*部分），也许被称为**God 对象**（或**God 类**）的那个反模式是我最经常遇到的。正如其名所示（我必须说，在重构和清洁代码的世界里，命名的水平是相当高的！），God 类是一种负面的编码实践，其特征是一个承担了过多职责的类。它表现为一个将多个对象整合并监督，执行应用程序中所有任务的类。这种反模式违反了保持类专注和模块化的原则，导致维护性差和代码可读性差。

你可能认为这可能是一个好的模式；此外，名称也可能暗示一些积极的东西。好吧，这并不是情况，我们马上就会看到原因。

God 对象违反了 SOLID 原则，尤其是单一职责原则。God 类通过承担过多的职责和缺乏焦点，完全违反了单一职责原则。结果，工程师们不得不重新发明轮子，复制代码，并积累技术债务。

由于 God 对象，你的代码将更加脆弱。由于 God 类的职责繁重，它需要频繁更新，这大大增加了引入破坏性更改的可能性。理解对 God 类的修改对整个应用程序的影响可能具有挑战性，甚至是不可能的。

神对象几乎无法进行测试。一般来说，当类较小、专注且独立于其他类型时，它很容易进行测试。相反，神类以相反的方式运作。作为一个庞大、包罗万象的实体，它控制和管理着众多其他对象，因此与这些类型紧密耦合。因此，由于复杂的设置和为了测试目的而实例化它所需的模拟或存根大量依赖项，测试神类变得是一项艰巨的任务。你常常会在它有太多的*协作者*时意识到你正在编写神对象，也就是说，有太多其他对象是类本身工作所必需的。

最后，主要的痛点。我们总是遇到这种情况。神对象使你的代码难以阅读和理解。考虑到它们广泛的职责和与多个类型的交互，这些类中的代码变得复杂也就不足为奇了。我这里说的不仅仅是循环复杂度；例如，依赖于引起副作用和可变性的外部依赖项。这些复杂性的累积效应增加了代码的整体认知复杂性，从而在理解上设置了障碍，并降低了工程师的生产力和表现。

这里的解决方案不是唯一的，但方法可以是。创建神对象（尤其是在面向对象编程中）相当容易，但撤销它们可能非常困难。因此，使用古老的*分割与征服*技术（再次，单一职责原则），将类拆分成更小的部分；以你的领域为指导；并在保持类松散耦合的同时，将常见的方法和属性组合在一个类中。如果类或方法中有大量代码或功能，将它们拆分成简单、可管理、可测试的类和方法。最后，删除或弃用神对象。

# 摘要

在本章中，我们学习了如何发现一些——不是全部——代码异味。我们涵盖了我认为最常见的一些。其中一些你会在编码时本能地避免。我们处理了重复代码、长方法、大类和重复的开关。我们还学会了避免对原始类型的执着（使用字符串做一切事情是一个强烈的诱惑！）并研究了特征嫉妒方法。最后，我们讨论了发散性变更及其对立面，即散弹枪手术。哦，别忘了神对象。这只是现有所有代码异味的一部分，但我相信它是避免它们的一个良好起点。

进行任何类型的重构，尤其是神对象重构，的一个关键条件是良好的测试覆盖率。在下一章中，我们将处理这个极其重要的问题。

# 进一步阅读

+   **原始数据类型** [`docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html`](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html)

+   **策略**: [`refactoring.guru/design-patterns/strategy`](https://refactoring.guru/design-patterns/strategy)

+   **访问者**: [`refactoring.guru/design-patterns/visitor`](https://refactoring.guru/design-patterns/visitor)

+   埃里希·伽玛、理查德·赫尔姆、拉尔夫·约翰逊和约翰·弗利西斯（1994 年）。*设计模式：可重用面向对象软件的元素*。Addison-Wesley Professional.

+   **代码** **异味**: [`refactoring.guru/refactoring/smells`](https://refactoring.guru/refactoring/smells)
