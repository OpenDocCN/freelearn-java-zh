# 第六章。继承，抽象，扩展和特殊化

在本章中，我们将学习 Java 9 中面向对象编程最重要的支柱之一：继承。我们将使用示例来学习如何创建类层次结构，覆盖和重载方法，并处理超类中定义的构造函数。我们将：

+   创建类层次结构以抽象和特殊化行为

+   理解继承

+   创建一个抽象基类

+   声明从另一个类继承的类

+   重载构造函数

+   覆盖实例方法

+   重载实例方法

# 创建类层次结构以抽象和特殊化行为

在之前的章节中，我们一直在使用 Java 9 创建类来生成现实生活中对象的蓝图。我们声明了类，然后在 JShell 中创建了这些类的实例。现在是时候利用 Java 9 中包含的许多最先进的面向对象编程特性，开始设计一个类层次结构，而不是使用孤立的类。首先，我们将根据需求设计所有需要的类，然后使用 Java 9 中可用的功能来编写设计的类。

我们使用类来表示虚拟生物。现在，让我们想象一下，我们必须开发一个复杂的 Web 服务，需要我们处理数十种虚拟动物。在项目的第一阶段，许多这些虚拟动物将类似于宠物和家畜。需求规定，我们的 Web 服务将开始处理以下四种与家畜动物物种相似的虚拟动物：

+   **马**（**Equus ferus caballus**）。不要将其与野马（Equus ferus）混淆。我们将拥有雄性和雌性马，雌性马可能怀孕。此外，我们将需要处理以下三种特定的马种：美国四分之一马，夏尔马和纯种马。

+   **鹦鹉**（**Nymphicus hollandicus**）。这种鸟也被称为鹦鹉或维罗。

+   **缅因库恩**。这是最大的家养猫品种之一（Felis silvestris catus）。

+   **家兔**（**Oryctolagus cuniculus**）。这种兔子也被称为欧洲兔。

前面的列表包括每种家畜动物物种的学名。我们肯定会使用每种物种的最常见名称，并将学名作为`String`类型的类常量。因此，我们不会有复杂的类名，比如`VirtualEquusFerusCaballus`，而是使用`VirtualHorse`。

我们的第一个需求规定，我们必须处理先前列举的四种家畜动物物种的有限数量品种。此外，将来将需要处理其他列出的家畜动物物种的其他成员，其他家畜哺乳动物，额外的家禽，特定的马种，甚至不属于家畜动物物种的爬行动物和鸟类。我们的面向对象设计必须准备好为未来的需求进行扩展，就像在现实项目中经常发生的那样。事实上，我们将使用这个例子来理解面向对象编程如何轻松地扩展现有设计以考虑未来的需求。

我们不想模拟动物王国及其分类的完整表示。我们只会创建必要的类，以便拥有一个灵活的模型，可以根据未来的需求轻松扩展。动物王国非常复杂。我们将把重点放在这个庞大家族的一些成员上。

以下示例的主要目标之一是了解面向对象编程并不会牺牲灵活性。我们将从一个简单的类层次结构开始，随着所需功能的复杂性增加以及对这些新需求的更多了解，我们将扩展它。让我们记住，需求并不是固定的，我们总是必须根据这些新需求添加新功能并对现有类进行更改。

我们将创建一个类层次结构来表示虚拟动物及其品种的复杂分类。当我们扩展一个类时，我们创建这个类的子类。以下列表列举了我们将创建的类及其描述：

+   `VirtualAnimal`：这个类概括了动物王国的所有成员。马、猫、鸟、兔子和爬行动物有一个共同点：它们都是动物。因此，创建一个类作为我们面向对象设计中可能需要表示的不同类别的虚拟动物的基线是有意义的。

+   `VirtualMammal`：这个类概括了所有哺乳动物的虚拟动物。哺乳动物与昆虫、鸟类、两栖动物和爬行动物不同。我们已经知道我们可以有母马，并且它们可以怀孕。我们还知道我们将需要对爬行动物和鸟类进行建模，因此我们创建了一个扩展`VirtualAnimal`并成为其子类的`VirtualMammal`类。

+   `VirtualBird`：这个类概括了所有鸟类。鸟类与哺乳动物、昆虫、两栖动物和爬行动物不同。我们已经知道我们还将需要对爬行动物进行建模。鹦鹉是一种鸟，因此我们将在与`VirtualMammal`同级别创建一个`VirtualBird`类。

+   `VirtualDomesticMammal`：这个类扩展了`VirtualMammal`类。让我们进行一些研究，我们会意识到老虎（Panthera tigris）是目前最大和最重的猫科动物。老虎是一种猫，但它与缅因猫完全不同，缅因猫是一种小型家养猫。最初的需求规定我们要处理虚拟家养和虚拟野生动物，因此我们将创建一个概括所有虚拟家养哺乳动物的类。将来，我们将有一个`VirtualWildMammal`子类，它将概括所有虚拟野生哺乳动物。

+   `VirtualDomesticBird`：这个类扩展了`VirtualBird`类。让我们进行一些研究，我们会意识到鸵鸟（Struthio camelus）是目前最大的活鸟。鸵鸟是一种鸟，但它与鹦鹉完全不同，鹦鹉是一种小型家养鸟。我们将处理虚拟家养和虚拟野生鸟，因此我们将创建一个概括所有虚拟家养鸟的类。将来，我们将有一个`VirtualWildBird`类，它将概括所有虚拟野生鸟。

+   `VirtualHorse`：这个类扩展了`VirtualDomesticMammal`类。我们可以继续用额外的子类专门化`VirtualDomesticMammal`类，直到达到`VirtualHorse`类。例如，我们可以创建一个`VirtualHerbivoreDomesticMammal`子类，然后让`VirtualHorse`类继承它。然而，我们需要开发的 Web 服务不需要在`VirtualDomesticMammal`和`VirtualHorse`之间有任何中间类。`VirtualHorse`类概括了我们应用程序中虚拟马所需的所有字段和方法。`VirtualHorse`类的不同子类将代表虚拟马品种的不同家族。

+   `VirtualDomesticRabbit`：这个类扩展了`VirtualDomesticMammal`类。`VirtualDomesticRabbit`类概括了我们应用程序中虚拟家养兔所需的所有字段和方法。

+   `VirtualDomesticCat`：这个类扩展了`VirtualDomesticMammal`类。`VirtualDomesticCat`类概括了我们应用程序中虚拟家养猫所需的所有字段和方法。

+   `美国四分之一马`：这个类扩展了`虚拟马`类。`美国四分之一马`类概括了属于美国四分之一马品种的虚拟马所需的所有字段和方法。

+   `ShireHorse`：这个类扩展了`虚拟马`类。`ShireHorse`类概括了属于莱茵马品种的虚拟马所需的所有字段和方法。

+   `Thoroughbred`：这个类扩展了`虚拟马`类。`Thoroughbred`类概括了属于纯种马品种的虚拟马所需的所有字段和方法。

+   `Cockatiel`：这个类扩展了`虚拟家禽`类。`Cockatiel`类概括了属于鹦鹉家族的虚拟家禽所需的所有字段和方法。

+   `MaineCoon`：这个类扩展了`虚拟家猫`类。`MaineCoon`类概括了属于缅因库恩品种的虚拟家猫所需的所有字段和方法。

以下表格显示了前述列表中的每个类及其超类、父类或超类型。

| 子类、子类或子类型 | 超类、父类或超类型 |
| --- | --- |
| `虚拟哺乳动物` | `虚拟动物` |
| `虚拟鸟` | `虚拟动物` |
| `虚拟家畜哺乳动物` | `虚拟哺乳动物` |
| `虚拟家禽` | `虚拟鸟` |
| 虚拟马 | 虚拟家畜哺乳动物 |
| `虚拟家兔` | `虚拟家畜哺乳动物` |
| `虚拟家猫` | `虚拟家畜哺乳动物` |
| `美国四分之一马` | `虚拟马` |
| `ShireHorse` | `虚拟马` |
| `Thoroughbred` | `虚拟马` |
| `Cockatiel` | `虚拟家禽` |
| `MaineCoon` | `虚拟家猫` |

以下的 UML 图显示了以类层次结构组织的前述类。使用斜体文本格式的类名表示它们是抽象类。注意图表中不包括任何成员，只有类名。我们稍后会添加成员。

![创建类层次结构以抽象和特殊化行为](img/00065.jpeg)

# 理解继承

当一个类继承自另一个类时，它继承了组成父类的所有成员，这也被称为**超类**。继承元素的类被称为超类的**子类**。例如，`VirtualBird`子类继承了`VirtualAnimal`超类中定义的所有实例字段、类字段、实例方法和类方法。

### 提示

在 Java 9 中，子类不会从其超类那里继承任何构造函数。但是，可以调用超类中定义的构造函数，在下面的示例中我们将这样做。只有在超类中定义的任何构造函数中使用`private`访问修饰符才会使子类无法调用该构造函数。

`VirtualAnimal`抽象类是我们类层次结构的基线。我们说它是一个**抽象类**，因为我们不能创建`VirtualAnimal`类的实例。相反，我们必须创建`VirtualAnimal`的具体子类的实例，任何不是抽象类的子类。我们可以用来创建它们的类通常被称为**具体类**或在大多数情况下只是类。Java 9 允许我们声明类为抽象类，当它们不打算生成实例时。

### 注意

我们不能使用`new`关键字后跟类名来创建抽象类的实例。

我们要求每个`VirtualAnimal`指定它的年龄，但我们不需要为它们指定任何名字。我们只给家养动物取名字。因此，当我们创建任何`VirtualAnimal`，也就是任何`VirtualAnimal`子类的实例时，我们将不得不指定一个年龄值。该类将定义一个`age`字段，并在创建虚拟动物时打印一条消息。

但是等等；我们刚刚解释过，我们正在谈论一个抽象类，并且 Java 不允许我们创建抽象类的实例。我们不能创建`VirtualAnimal`抽象类的实例，但我们将能够创建具有`VirtualAnimal`作为超类的任何具体类的实例，这个子类最终可以调用`VirtualAnimal`抽象类中定义的构造函数。听起来有点复杂，但在我们编写类并在 JShell 中运行示例后，我们将很容易理解情况。我们将在我们定义的每个构造函数中打印消息，以便更容易理解当我们创建具有一个或多个超类的具体类的实例时会发生什么，包括一个或多个抽象超类。`VirtualAnimal`的所有子类的实例也将是`VirtualAnimal`的实例。

`VirtualAnimal`抽象类将定义抽象类方法和抽象实例方法。**抽象类方法**是声明而没有实现的类方法。**抽象实例方法**，也称为抽象方法，是声明而没有实现的实例方法。

### 提示

当我们声明任何两种类型的抽象方法时，我们只声明参数（如果有），然后放一个分号（`;`）。我们根本不使用花括号。我们只能在抽象类中声明抽象方法。任何抽象类的具体子类必须为所有继承的抽象方法提供实现，以成为我们可以使用`new`关键字创建实例的类。

`VirtualAnimal`类将声明以下七个抽象方法，满足特定家族或类型的所有成员的要求。该类只声明它们所需的参数，而不实现方法。子类将负责满足解释的要求。

+   `isAbleToFly`：返回一个布尔值，指示虚拟动物是否能飞。

+   `isRideable`：返回一个布尔值，指示虚拟动物是否可骑。可骑的动物能够被骑乘。

+   `isHerbivore`：返回一个布尔值，指示虚拟动物是否是食草动物。

+   `isCarnivore`：返回一个布尔值，指示虚拟动物是否是肉食动物。

+   `getAverageNumberOfBabies`：返回通常为虚拟动物类型一次出生的平均婴儿数量。

+   `getBaby`：返回虚拟动物类型的婴儿的`String`表示。

+   `getAsciiArt`：返回表示虚拟动物的 ASCII 艺术（基于文本的视觉艺术）的`String`。

`VirtualAnimal`类将定义以下五个方法，满足每个实例的要求。这些将是具体方法，将在`VirtualAnimal`类中编码，并由其所有子类继承。其中一些方法调用先前解释的抽象方法。我们将在稍后详细了解这是如何工作的。

+   printAsciiArt：这将打印`getAsciiArt`方法返回的`String`。

+   `isYoungerThan`：返回一个布尔值，指示`VirtualAnimal`的`age`值是否低于作为参数接收的`VirtualAnimal`实例的年龄。

+   `isOlderThan`：返回一个布尔值，指示`VirtualAnimal`类的`age`值是否大于作为参数接收的`VirtualAnimal`实例的年龄。

+   `printAge`：打印虚拟动物的`age`值。

+   `printAverageNumberOfBabies`：打印通常为虚拟动物一次出生的平均婴儿数量的表示。该方法将考虑由不同具体子类中实现的`getAverageNumberOfBabies`方法返回的值。

`VirtualMammal`类继承自`VirtualAnimal`。当创建新的`VirtualMammal`实例时，我们将不得不指定其年龄和是否怀孕。该类从`VirtualAnimal`超类继承了`age`属性，因此只需要添加一个字段来指定虚拟哺乳动物是否怀孕。请注意，我们将不会在任何时候指定性别，以保持简单。如果我们添加了性别，我们将需要验证以避免雄性怀孕。现在，我们的重点是继承。该类将在创建虚拟哺乳动物时显示一条消息；也就是说，每当执行其构造函数时。

### 提示

每个类都继承自一个类，因此，我们将定义的每个新类都只有一个超类。在这种情况下，我们将始终使用**单一继承**。在 Java 中，一个类不能从多个类继承。

`VirtualDomesticMammal`类继承自`VirtualMammal`。当创建新的`VirtualDomesticMammal`实例时，我们将不得不指定其名称和最喜欢的玩具。我们给任何家养哺乳动物都起名字，它们总是会挑选一个最喜欢的玩具。有时它们只是选择满足它们破坏欲望的物品。在许多情况下，最喜欢的玩具并不一定是我们希望它们选择的玩具（我们的鞋子、运动鞋、拖鞋或电子设备），但让我们专注于我们的类。我们无法改变名称，但可以改变最喜欢的玩具。我们永远不会改变任何家养哺乳动物的名称，但我们绝对可以强迫它改变最喜欢的玩具。该类在创建虚拟家养哺乳动物时显示一条消息。

`VirtualDomesticMammal`类将声明一个`talk`实例方法，该方法将显示一条消息，指示虚拟家养哺乳动物的名称与消息“说了些什么”的连接。每个子类必须以不同的方式让特定的家养哺乳动物说话。鹦鹉确实会说话，但我们将把马的嘶鸣和兔子的牙齿咕噜声视为它们在说话。请注意，在这种情况下，`talk`实例方法在`VirtualDomesticMammal`类中具有具体的实现，而不是抽象的实例方法。子类将能够为此方法提供不同的实现。

`VirtualHorse`类继承自`VirtualDomesticMammal`，并实现了从`VirtualAnimal`超类继承的所有抽象方法，除了`getBaby`和`getAsciiArt`。这两个方法将在`VirtualHorse`的每个子类中实现，以确定马的品种。

我们希望马能够嘶鸣和嘶鸣。因此，我们需要`neigh`和`nicker`方法。马通常在生气时嘶鸣，在快乐时嘶鸣。情况比这更复杂一些，但我们将为我们的示例保持简单。

`neigh`方法必须允许虚拟马执行以下操作：

+   只嘶鸣一次

+   特定次数的嘶鸣

+   与另一个只有一次名字的虚拟家养哺乳动物相邻

+   对另一个只有特定次数名字的虚拟家养哺乳动物嘶鸣

`nicker`方法必须允许虚拟马执行以下操作：

+   只嘶鸣一次

+   特定次数的嘶鸣

+   只对另一个只有一次名字的虚拟家养哺乳动物嘶鸣

+   对另一个只有特定次数名字的虚拟家养哺乳动物嘶鸣

此外，马可以愉快地或愤怒地嘶鸣或嘶鸣。我们可以有一个`neigh`方法，其中许多参数具有默认值，或者有许多`neigh`方法。Java 9 提供了许多机制来解决虚拟马必须能够嘶鸣的不同方式的挑战。我们将对`neigh`和`nicker`方法应用相同的解决方案。

当我们为任何虚拟马调用`talk`方法时，我们希望它开心地嘶鸣一次。我们不希望显示在`VirtualDomesticMammal`类中引入的`talk`方法中定义的消息。因此，`VirtualHorse`类必须用自己的定义覆盖继承的`talk`方法。

我们想知道虚拟马属于哪个品种。因此，我们将定义一个`getBreed`抽象方法。`VirtualHorse`的每个子类在调用此方法时必须返回适当的`String`名称。`VirtualHorse`类将定义一个名为`printBreed`的方法，该方法使用`getBreed`方法来检索名称并打印品种。

到目前为止，我们提到的所有类都是抽象类。我们不能创建它们的实例。`AmericanQuarterHorse`、`ShireHorse`和`Thoroughbred`类继承自`VirtualHorse`类，并实现了继承的`getBaby`、`getAsciiArt`和`getBreed`方法。此外，它们的构造函数将打印一条消息，指示我们正在创建相应类的实例。这三个类都是具体类，我们可以创建它们的实例。

我们将稍后使用`VirtualBird`、`VirtualDomesticBird`、`Cockatiel`、`VirtualDomesticCat`和`MaineCoon`类。首先，我们将在 Java 9 中创建基类`VirtualAnimal`抽象类，然后使用简单的继承创建子类，直到`VirtualHorse`类。我们将重写方法和重载方法以满足所有要求。我们将利用多态性，这是面向对象编程中非常重要的特性，我们将在 JShell 中使用创建的类时了解到。当然，我们将深入研究分析不同类时引入的许多主题。

以下 UML 图显示了我们将在本章中编写的所有抽象类的成员：`VirtualAnimal`、`VirtualMammal`、`VirtualDomesticMammal`和`VirtualHorse`。我们将在下一章中编写其他类，并稍后将它们的成员添加到图中。我们使用斜体文本格式表示抽象方法。请记住，公共成员以加号（**+**）作为前缀。一个类有一个受保护的成员，使用井号作为前缀（**#**）。我们将使用粗体文本格式表示覆盖超类中现有方法的方法。在这种情况下，`VirtualHorse`类覆盖了`talk()`方法。

![理解继承](img/00066.jpeg)

在上一个 UML 图中，我们将注意到以下约定。我们将在包括类成员的所有 UML 图中使用这些约定。

+   构造函数与类名相同，不指定任何返回类型。它们始终是方法部分中列出的第一个方法。

+   字段的类型在字段名称之后用冒号（**：**）分隔。

+   每个方法的参数列表中的参数都用分号（**;**）分隔。

+   方法的返回类型在参数列表之后用冒号（**：**）分隔。

+   我们始终使用 Java 类型名称。

# 创建抽象基类

首先，我们将创建抽象类，该类将成为其他类的基类。以下是 Java 9 中`VirtualAnimal`抽象基类的代码。`class`之前的`abstract`关键字表示我们正在创建一个抽象类。示例的代码文件包含在`java_9_oop_chapter_06_01`文件夹中的`example06_01.java`文件中。

```java
public abstract class VirtualAnimal {
    public final int age;

    public VirtualAnimal(int age) {
        this.age = age;
        System.out.println("VirtualAnimal created.");
    }

    public abstract boolean isAbleToFly();

    public abstract boolean isRideable();

    public abstract boolean isHerbivore();

    public abstract boolean isCarnivore();

    public abstract int getAverageNumberOfBabies();

    public abstract String getBaby();

    public abstract String getAsciiArt();

    public void printAsciiArt() {
        System.out.println(getAsciiArt());
    }

    public void printAverageNumberOfBabies() {
        System.out.println(new String(
            new char[getAverageNumberOfBabies()]).replace(
                "\0", getBaby()));
    }

    public void printAge() {
        System.out.println(
            String.format("I am %d years old", age));
    }

    public boolean isYoungerThan(VirtualAnimal otherAnimal) {
        return age < otherAnimal.age; 
    }

    public boolean isOlderThan(VirtualAnimal otherAnimal) {
        return age > otherAnimal.age;
    }
}
```

前面的类声明了一个名为`age`的`int`类型的不可变字段。构造函数需要一个`age`值来创建类的实例，并打印一条消息指示创建了一个虚拟动物。该类声明了以下抽象方法，这些方法在返回类型之前包含`abstract`关键字，以便让 Java 知道我们只想声明所需的参数，并且不会为这些方法提供实现。我们已经解释了这些方法的目标，它们将在`VirtualAnimal`的子类中实现。 

+   `isAbleToFly`

+   `isRideable`

+   `isHerbivore`

+   `isCarnivore`

+   获取平均婴儿数量

+   `getBaby`

+   `getAsciiArt`

此外，该类声明了以下五个方法：

+   打印 AsciiArt：此方法调用`System.out.println`来打印`getAsciiArt`方法返回的`String`。

+   `printAverageNumberOfBabies`：此方法创建一个新的`char`数组，其元素数量等于`getAverageNumberOfBabies`方法返回的值。然后，代码创建一个初始化为`char`数组的新`String`，并调用`replace`方法来用`getBaby`方法返回的`String`替换每个`"\0"`。这样，我们生成一个`String`，其中包含`getBaby`返回的`String`的`getAverageNumberOfBabies`倍。代码调用`System.out.println`来打印生成的`String`。

+   打印年龄：此方法调用`System.out.println`来打印使用`String.format`生成的`String`，其中包括`age`不可变字段的值。

+   `isYoungerThan`：此方法在`otherAnimal`参数中接收一个`VirtualAnimal`实例，并返回在此实例的`age`字段值和`otherAnimal.age`之间应用小于运算符的结果。这样，只有当此实例的年龄小于`otherAnimal`的年龄时，该方法才会返回`true`。

+   `isOlderThan`：此方法在`otherAnimal`参数中接收一个`VirtualAnimal`实例，并返回在此实例的`age`字段值和`otherAnimal.age`之间应用大于运算符的结果。这样，只有当此实例的年龄大于`otherAnimal`的年龄时，该方法才会返回`true`。

如果我们在声明`VirtualAnimal`类之后在 JShell 中执行以下行，Java 将生成致命错误，并指出`VirtualAnimal`类是抽象的，不能被实例化。示例的代码文件包含在`java_9_oop_chapter_06_01`文件夹中的`example06_02.java`文件中。

```java
VirtualAnimal virtualAnimal1 = new VirtualAnimal(5);
```

以下屏幕截图显示了在 JShell 中执行上一个代码的结果：

![创建抽象基类](img/00067.jpeg)

# 声明从另一个类继承的类

现在我们将创建另一个抽象类。具体来说，我们将创建一个最近创建的`VirtualAnimal`抽象类的子类。以下行显示了扩展`VirtualAnimal`类的`VirtualMammal`抽象类的代码。请注意`abstract class`关键字后面跟着类名`VirtualMammal`，`extends`关键字和`VirtualAnimal`，即超类。

在类定义中，跟在`extends`关键字后面的类名表示新类从中继承的超类。示例的代码文件包含在`java_9_oop_chapter_06_01`文件夹中的`example06_03.java`文件中。

```java
public abstract class VirtualMammal extends VirtualAnimal {
    public boolean isPregnant;

    public VirtualMammal(int age, boolean isPregnant) {
 super(age);
        this.isPregnant = isPregnant;
        System.out.println("VirtualMammal created.");
    }

    public VirtualMammal(int age) {
        this(age, false);
    }
}
```

`VirtualMammal`抽象类继承了先前声明的`VirtualAnimal`抽象类的成员，并添加了一个名为`isPregnant`的新的`boolean`可变字段。新的抽象类声明了两个构造函数。其中一个构造函数需要一个`age`值来创建类的实例，就像`VirtualAnimal`构造函数一样。另一个构造函数需要`age`和`isPregnant`值。

如果我们只用一个 `age` 参数创建这个类的实例，Java 将使用第一个构造函数。如果我们用两个参数创建这个类的实例，一个是 `age` 的 `int` 值，一个是 `isPregnant` 的 `boolean` 值，Java 将使用第二个构造函数。

### 提示

我们已经重载了构造函数并提供了两个不同的构造函数。我们不会使用 `new` 关键字来使用这些构造函数，因为我们正在声明一个抽象类。但是，我们将能够通过使用 `super` 关键字从子类中调用这些构造函数。

需要 `isPregnant` 参数的第一个构造函数使用 `super` 关键字来调用基类或超类中的构造函数，也就是在 `VirtualAnimal` 类中定义的需要 `age` 参数的构造函数。在超类中定义的构造函数执行完毕后，代码会设置 `isPregnant` 可变字段的值，并打印一条消息，指示已创建了一个虚拟哺乳动物。

### 提示

我们使用 `super` 关键字来引用超类，并且可以使用这个关键字来调用超类中定义的任何构造函数。在 Java 9 中，子类不会继承其超类的构造函数。在其他编程语言中，子类会继承构造函数或初始化程序，因此，非常重要的是要理解在 Java 9 中这种情况并不会发生。

第二个构造函数使用 `this` 关键字来调用先前解释的构造函数，接收 `age` 和 `false` 作为 `isPregnant` 参数的值。

我们将创建另一个抽象类。具体来说，我们将创建一个最近创建的 `VirtualMammal` 抽象类的子类。以下几行显示了扩展 `VirtualMammal` 类的 `VirtualDomesticMammal` 抽象类的代码。注意 `abstract class` 关键字后面跟着类名 `VirtualDomesticMammal`，`extends` 关键字和 `VirtualMammal`，也就是超类。跟在 `extends` 关键字后面的类名指示了新类在类定义中继承的超类。示例的代码文件包含在 `java_9_oop_chapter_06_01` 文件夹中的 `example06_04.java` 文件中。

```java
public abstract class VirtualDomesticMammal extends VirtualMammal {
    public final String name;
    public String favoriteToy;

    public VirtualDomesticMammal(
        int age, 
        boolean isPregnant, 
        String name, 
        String favoriteToy) {
 super(age, isPregnant);
        this.name = name;
        this.favoriteToy = favoriteToy;
        System.out.println("VirtualDomesticMammal created.");
    }

    public VirtualDomesticMammal(
        int age, 
        String name, 
        String favoriteToy) {
        this(age, false, name, favoriteToy);
    }

    public void talk() {
        System.out.println(
            String.format("%s: says something", name));
    }
}
```

`VirtualDomesticMammal` 抽象类继承了先前声明的 `VirtualMammal` 抽象类的成员。重要的是要理解，新类也继承了超类从其超类继承的成员，也就是从 `VirtualAnimal` 抽象类继承的成员。例如，我们的新类继承了在 `VirtualAnimal` 抽象类中声明的 `age` 不可变字段以及在这个类中声明的所有其他成员。

`VirtualDomesticMammal` 类添加了一个名为 `name` 的新的不可变字段和一个名为 `favoriteToy` 的新的可变字段。这个新的抽象类声明了两个构造函数。其中一个构造函数需要四个参数来创建类的实例：`age`、`isPregnant`、`name` 和 `favoriteToy`。另一个构造函数需要除了 `isPregnant` 之外的所有参数。

需要四个参数的第一个构造函数使用 `super` 关键字来调用基类或超类中的构造函数，也就是在 `VirtualMammal` 类中定义的需要两个参数 `age` 和 `isPregnant` 的构造函数。在超类中定义的构造函数执行完毕后，代码会设置 `name` 和 `favoriteToy` 字段的值，并打印一条消息，指示已创建了一个虚拟家养哺乳动物。

第二个构造函数使用 `this` 关键字来调用先前解释的构造函数，接收参数和 `false` 作为 `isPregnant` 参数的值。

最后，这个类声明了一个`talk`方法，显示了一个以`name`值开头，后跟一个冒号(`:`)和`says something`的消息。请注意，我们可以在`VirtualDomesticMammal`的任何子类中覆盖这个方法，因为每个虚拟家养哺乳动物都有自己不同的说话方式。

# 覆盖和重载方法

Java 允许我们多次使用相同的方法名定义不同参数的方法。这个特性被称为**方法重载**。在之前创建的抽象类中，我们重载了构造函数。

例如，我们可以利用方法重载来定义`VirtualHorse`抽象类中必须定义的`neigh`和`nicker`方法的多个版本。然而，在重载方法时，避免代码重复是非常重要的。

有时，我们在一个类中定义一个方法，我们知道子类可能需要提供一个不同版本的方法。一个明显的例子就是我们在`VirtualDomesticMammal`类中定义的`talk`方法。当一个子类提供了一个与超类中同名、参数和返回类型相同的方法的不同实现时，我们称之为**覆盖**方法。当我们覆盖一个方法时，子类中的实现会覆盖超类中提供的代码。

```java
VirtualHorse abstract class that extends the VirtualDomesticMammal class. Note the abstract class keywords followed by the class name, VirtualHorse, the extends keyword, and VirtualDomesticMammal, that is, the superclass. We will split the code for this class in many snippets to make it easier to analyze. The code file for the sample is included in the java_9_oop_chapter_06_01 folder, in the example06_05.java file.
```

```java
public abstract class VirtualHorse extends VirtualDomesticMammal {
    public VirtualHorse(
        int age, 
        boolean isPregnant, 
        String name, 
        String favoriteToy) {
 super(age, isPregnant, name, favoriteToy);
        System.out.println("VirtualHouse created.");        
    }

    public VirtualHorse(
        int age, 
        String name, 
        String favoriteToy) {
        this(age, false, name, favoriteToy);
    }

    public boolean isAbleToFly() {
        return false;
    }

    public boolean isRideable() {
        return true;
    }

    public boolean isHerbivore() {
        return true;
    }

    public boolean isCarnivore() {
        return false;
    }

    public int getAverageNumberOfBabies() {
        return 1;
    }
```

```java
VirtualHorse abstract class that extends the VirtualDomesticMammal class. The code file for the sample is included in the java_9_oop_chapter_06_01 folder, in the example06_05.java file.
```

```java
    public abstract String getBreed();

    public void printBreed() {
        System.out.println(getBreed());
    }

    protected void printSoundInWords(
        String soundInWords, 
        int times, 
        VirtualDomesticMammal otherDomesticMammal,
        boolean isAngry) {
        String message = String.format("%s%s: %s%s",
            name,
            otherDomesticMammal == null ? 
                "" : String.format(" to %s ", otherDomesticMammal.name),
            isAngry ?
                "Angry " : "",
            new String(new char[times]).replace("\0", soundInWords));
        System.out.println(message);
    }
```

```java
VirtualHorse abstract class that extends the VirtualDomesticMammal class. The code file for the sample is included in the java_9_oop_chapter_06_01 folder, in the example06_05.java file.
```

```java
    public void printNeigh(int times, 
        VirtualDomesticMammal otherDomesticMammal,
        boolean isAngry) {
        printSoundInWords("Neigh ", times, otherDomesticMammal, isAngry);
    }

    public void neigh() {
        printNeigh(1, null, false);
    }

    public void neigh(int times) {
        printNeigh(times, null, false);
    }

    public void neigh(int times, 
        VirtualDomesticMammal otherDomesticMammal) {
        printNeigh(times, otherDomesticMammal, false);
    }

    public void neigh(int times, 
        VirtualDomesticMammal otherDomesticMammal, 
        boolean isAngry) {
        printNeigh(times, otherDomesticMammal, isAngry);
    }

    public void printNicker(int times, 
        VirtualDomesticMammal otherDomesticMammal,
        boolean isAngry) {
        printSoundInWords("Nicker ", times, otherDomesticMammal, isAngry);
    }

    public void nicker() {
        printNicker(1, null, false);
    }

    public void nicker(int times) {
        printNicker(times, null, false);
    }

    public void nicker(int times, 
        VirtualDomesticMammal otherDomesticMammal) {
        printNicker(times, otherDomesticMammal, false);
    }

    public void nicker(int times, 
        VirtualDomesticMammal otherDomesticMammal, 
        boolean isAngry) {
        printNicker(times, otherDomesticMammal, isAngry);
    }

 @Override
 public void talk() {
 nicker();
 }
}
```

`VirtualHorse`类覆盖了从`VirtualDomesticMammal`继承的`talk`方法。代码只是调用了没有参数的`nicker`方法，因为马不会说话，它们会嘶叫。这个方法不会调用其超类中同名的方法；也就是说，我们没有使用`super`关键字来调用`VirtualDomesticMammal`中定义的`talk`方法。

### 提示

我们在方法声明之前使用`@Override`注解来通知 Java 9 编译器，该方法意在覆盖在超类中声明的同名方法。当我们覆盖方法时，添加这个注解并不是强制的，但是将其包括进去是一个好习惯，我们在覆盖方法时总是会使用它，因为它有助于防止错误。例如，如果我们在方法名和参数中写成了`tak()`而不是`talk()`，使用`@Override`注解会使 Java 9 编译器生成一个错误，因为标记为`@Override`的`talk`方法未能成功覆盖其中一个超类中具有相同名称和参数的方法。

`nicker`方法被重载了四次，使用了不同的参数声明。以下几行展示了类体中包括的四个不同声明：

```java
public void nicker()
public void nicker(int times) 
public void nicker(int times, 
    VirtualDomesticMammal otherDomesticMammal) 
public void nicker(int times, 
    VirtualDomesticMammal otherDomesticMammal, 
    boolean isAngry)
```

这样，我们可以根据提供的参数调用任何定义的`nicker`方法。这四个方法最终都会调用`printNicker`公共方法，使用不同的默认值来调用具有相同名称但未在`nicker`调用中提供的参数。该方法调用`printSoundInWords`公共方法，将`"Nicker "`作为`soundInWords`参数的值，并将其他参数设置为接收到的具有相同名称的参数。这样，`printNicker`方法根据指定的次数(`times`)、可选的目标虚拟家养哺乳动物(`otherDomesticMammal`)以及马是否生气(`isAngry`)来构建并打印嘶叫消息。

`VirtualHorse`类对`neigh`方法也使用了类似的方法。这个方法也被重载了四次，使用了不同的参数声明。以下几行展示了类体中包括的四个不同声明。它们使用了我们刚刚分析过的`nicker`方法的相同参数。

```java
public void neigh()
public void neigh(int times) 
public void neigh(int times, 
    VirtualDomesticMammal otherDomesticMammal) 
public void neigh(int times, 
    VirtualDomesticMammal otherDomesticMammal, 
    boolean isAngry)
```

这样，我们可以根据提供的参数调用任何定义的`neigh`方法。这四种方法最终会使用不同的默认值调用`printNeigh`公共方法，这些默认值是与调用`nicker`时未提供的同名参数。该方法调用`printSoundInWords`公共方法，将`"Neigh "`作为`soundInWords`参数的值，并将其他参数设置为具有相同名称的接收参数。

# 测试你的知识

1.  在 Java 9 中，一个子类：

1.  继承其超类的所有构造函数。

1.  不继承任何构造函数。

1.  从其超类继承具有最大数量参数的构造函数。

1.  我们可以声明抽象方法：

1.  在任何类中。

1.  只在抽象类中。

1.  只在抽象类的具体子类中。

1.  任何抽象类的具体子类：

1.  必须为所有继承的抽象方法提供实现。

1.  必须为所有继承的构造函数提供实现。

1.  必须为所有继承的抽象字段提供实现。

1.  以下哪行声明了一个名为`Dog`的抽象类，作为`VirtualAnimal`的子类：

1.  `public abstract class Dog subclasses VirtualAnimal`

1.  `public abstract Dog subclasses VirtualAnimal`

1.  `public abstract class Dog extends VirtualAnimal`

1.  在方法声明之前指示 Java 9 编译器该方法意味着重写超类中同名方法的注解是：

1.  `@Overridden`

1.  `@OverrideMethod`

1.  `@Override`

# 总结

在本章中，您学习了抽象类和具体类之间的区别。我们学会了如何利用简单的继承来专门化基本抽象类。我们设计了许多类，从上到下使用链接的构造函数，不可变字段，可变字段和实例方法。

然后我们在 JShell 中编写了许多这些类，利用了 Java 9 提供的不同特性。我们重载了构造函数，重写和重载了实例方法，并利用了一个特殊的注解来重写方法。

现在您已经了解了继承，抽象，扩展和专门化，我们准备完成编写其他类，并了解如何使用类型转换和多态，这是我们将在下一章讨论的主题。
