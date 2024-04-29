# 第七章。成员继承和多态

在本章中，我们将学习 Java 9 中面向对象编程最激动人心的特性之一：多态。我们将编写许多类，然后在 JShell 中使用它们的实例，以了解对象如何呈现许多不同的形式。我们将：

+   创建从抽象超类继承的具体类

+   使用子类的实例进行操作

+   理解多态。

+   控制子类是否可以覆盖成员

+   控制类是否可以被子类化

+   使用执行与不同子类实例的操作的方法

# 创建从抽象超类继承的具体类

在上一章中，我们创建了一个名为`VirtualAnimal`的抽象基类，然后编写了以下三个抽象子类：`VirtualMammal`、`VirtualDomesticMammal`和`VirtualHorse`。现在，我们将编写以下三个具体类。每个类代表不同的马种，是`VirtualHorse`抽象类的子类。

+   `AmericanQuarterHorse`: 这个类表示属于美国四分之一马品种的虚拟马。

+   `ShireHorse`: 这个类表示属于夏尔马品种的虚拟马。

+   `Thoroughbred`: 这个类表示属于纯种赛马品种的虚拟马。

这三个具体类将实现它们从抽象超类继承的以下三个抽象方法：

+   `String getAsciiArt()`: 这个抽象方法是从`VirtualAnimal`抽象类继承的。

+   `String getBaby()`: 这个抽象方法是从`VirtualAnimal`抽象类继承的。

+   `String getBreed()`: 这个抽象方法是从`VirtualHorse`抽象类继承的。

以下 UML 图表显示了我们将编写的三个具体类`AmericanQuarterHorse`、`ShireHorse`和`Thoroughbred`的成员：我们不使用粗体文本格式来表示这三个具体类将声明的三个方法，因为它们不是覆盖方法；它们是实现类继承的抽象方法。

![创建从抽象超类继承的具体类](img/00068.jpeg)

首先，我们将创建`AmericanQuarterHorse`具体类。以下行显示了 Java 9 中此类的代码。请注意，在`class`之前没有`abstract`关键字，因此，我们的类必须确保实现所有继承的抽象方法。示例的代码文件包含在`java_9_oop_chapter_07_01`文件夹中的`example07_01.java`文件中。

```java
public class AmericanQuarterHorse extends VirtualHorse {
    public AmericanQuarterHorse(
        int age, 
        boolean isPregnant, 
        String name, 
        String favoriteToy) {
        super(age, isPregnant, name, favoriteToy);
        System.out.println("AmericanQuarterHorse created.");
    }

    public AmericanQuarterHorse(
        int age, String name, String favoriteToy) {
        this(age, false, name, favoriteToy);
    }

    public String getBaby() {
        return "AQH baby ";
    }

    public String getBreed() {
        return "American Quarter Horse";
    }

    public String getAsciiArt() {
        return
            "     >>\\.\n" +
            "    /*  )`.\n" + 
            "   // _)`^)`.   _.---. _\n" +
            "  (_,' \\  `^-)''      `.\\\n" +
            "        |              | \\\n" +
            "        \\              / |\n" +
            "       / \\  /.___.'\\  (\\ (_\n" +
            "      < ,'||     \\ |`. \\`-'\n" +
            "       \\\\ ()      )|  )/\n" +
            "       |_>|>     /_] //\n" +
            "         /_]        /_]\n";
    }
}
```

现在我们将创建`ShireHorse`具体类。以下行显示了 Java 9 中此类的代码。示例的代码文件包含在`java_9_oop_chapter_07_01`文件夹中的`example07_01.java`文件中。

```java
public class ShireHorse extends VirtualHorse {
    public ShireHorse(
        int age, 
        boolean isPregnant, 
        String name, 
        String favoriteToy) {
        super(age, isPregnant, name, favoriteToy);
        System.out.println("ShireHorse created.");
    }

    public ShireHorse(
        int age, String name, String favoriteToy) {
        this(age, false, name, favoriteToy);
    }

    public String getBaby() {
        return "ShireHorse baby ";
    }

    public String getBreed() {
        return "Shire Horse";
    }

    public String getAsciiArt() {
        return
            "                        ;;\n" + 
            "                      .;;'*\\\n" + 
            "           __       .;;' ' \\\n" +
            "         /'  '\\.~~.~' \\ /'\\.)\n" +
            "      ,;(      )    /  |\n" + 
            "     ,;' \\    /-.,,(   )\n" +
            "          ) /|      ) /|\n" +    
            "          ||(_\\     ||(_\\\n" +    
            "          (_\\       (_\\\n";
    }
}
```

最后，我们将创建`Thoroughbred`具体类。以下行显示了 Java 9 中此类的代码。示例的代码文件包含在`java_9_oop_chapter_07_01`文件夹中的`example07_01.java`文件中。

```java
public class Thoroughbred extends VirtualHorse {
    public Thoroughbred(
        int age, 
        boolean isPregnant, 
        String name, 
        String favoriteToy) {
        super(age, isPregnant, name, favoriteToy);
        System.out.println("Thoroughbred created.");
    }

    public Thoroughbred(
        int age, String name, String favoriteToy) {
        this(age, false, name, favoriteToy);
    }

    public String getBaby() {
        return "Thoroughbred baby ";
    }

    public String getBreed() {
        return "Thoroughbred";
    }

    public String getAsciiArt() {
        return
            "             })\\-=--.\n" +  
            "            // *._.-'\n" +
            "   _.-=-...-'  /\n" +
            " {{|   ,       |\n" +
            " {{\\    |  \\  /_\n" +
            " }} \\ ,'---'\\___\\\n" +
            " /  )/\\\\     \\\\ >\\\n" +
            "   //  >\\     >\\`-\n" +
            "  `-   `-     `-\n";
    }
}
```

在我们编码的其他子类中发生的情况，我们为这三个具体类定义了多个构造函数。第一个构造函数需要四个参数，使用`super`关键字调用基类或超类中的构造函数，也就是在`VirtualHorse`类中定义的构造函数。在超类中定义的构造函数执行完毕后，代码会打印一条消息，指示已创建了每个具体类的实例。每个类中定义的构造函数会打印不同的消息。

第二个构造函数使用`this`关键字调用先前解释的构造函数，并使用`false`作为`isPregnant`参数的值。

每个类在`getBaby`和`getBreed`方法的实现中返回不同的`String`。此外，每个类在`getAsciiArt`方法的实现中返回虚拟马的不同 ASCII 艺术表示。

# 理解多态性

我们可以使用相同的方法，即使用相同名称和参数的方法，根据调用方法的类来引起不同的事情发生。在面向对象编程中，这个特性被称为**多态性**。多态性是对象能够呈现多种形式的能力，我们将通过使用先前编写的具体类的实例来看到它的作用。

以下几行创建了一个名为`american`的`AmericanQuarterHorse`类的新实例，并使用了一个不需要`isPregnant`参数的构造函数。示例的代码文件包含在`java_9_oop_chapter_07_01`文件夹中的`example07_01.java`文件中。

```java
AmericanQuarterHorse american = 
    new AmericanQuarterHorse(
        8, "American", "Equi-Spirit Ball");
american.printBreed();
```

以下几行显示了我们在 JShell 中输入前面的代码后，不同构造函数显示的消息：

```java
VirtualAnimal created.
VirtualMammal created.
VirtualDomesticMammal created.
VirtualHorse created.
AmericanQuarterHorse created.

```

`AmericanQuarterHorse`中定义的构造函数调用了其超类的构造函数，即`VirtualHorse`类。请记住，每个构造函数都调用其超类构造函数，并打印一条消息，指示创建了类的实例。我们没有五个不同的实例；我们只有一个实例，它调用了五个不同类的链接构造函数，以执行创建`AmericanQuarterHorse`实例所需的所有必要初始化。

如果我们在 JShell 中执行以下几行，它们都会显示`true`，因为`american`属于`VirtualAnimal`、`VirtualMammal`、`VirtualDomesticMammal`、`VirtualHorse`和`AmericanQuarterHorse`类。示例的代码文件包含在`java_9_oop_chapter_07_01`文件夹中的`example07_01.java`文件中。

```java
System.out.println(american instanceof VirtualAnimal);
System.out.println(american instanceof VirtualMammal);
System.out.println(american instanceof VirtualDomesticMammal);
System.out.println(american instanceof VirtualHorse);
System.out.println(american instanceof AmericanQuarterHorse);
```

前面几行的结果意味着`AmericanQuarterHorse`类的实例，其引用保存在类型为`AmericanQuarterHorse`的`american`变量中，可以采用以下任何一个类的实例形式：

+   虚拟动物

+   虚拟哺乳动物

+   虚拟家养哺乳动物

+   虚拟马

+   美国四分之一马

以下屏幕截图显示了在 JShell 中执行前面几行的结果：

![理解多态性](img/00069.jpeg)

我们在`VirtualHorse`类中编写了`printBreed`方法，并且我们没有在任何子类中重写此方法。以下是`printBreed`方法的代码：

```java
public void printBreed() {
    System.out.println(getBreed());
}
```

代码打印了`getBreed`方法返回的`String`，在同一类中声明为抽象方法。继承自`VirtualHorse`的三个具体类实现了`getBreed`方法，它们每个都返回不同的`String`。当我们调用`american.printBreed`方法时，JShell 显示`American Quarter Horse`。

以下几行创建了一个名为`zelda`的`ShireHorse`类的实例。请注意，在这种情况下，我们使用需要`isPregnant`参数的构造函数。与创建`AmericanQuarterHorse`类的实例时一样，JShell 将显示每个执行的构造函数的消息，这是由我们编写的链接构造函数的结果。示例的代码文件包含在`java_9_oop_chapter_07_01`文件夹中的`example07_01.java`文件中。

```java
ShireHorse zelda =
    new ShireHorse(9, true, 
        "Zelda", "Tennis Ball");
```

接下来的几行调用了`american`（`AmericanQuarterHorse`的实例）和`zelda`（`ShireHorse`的实例）的`printAverageNumberOfBabies`和`printAsciiArt`实例方法。示例的代码文件包含在`java_9_oop_chapter_07_01`文件夹中的`example07_01.java`文件中。

```java
american.printAverageNumberOfBabies();
american.printAsciiArt();
zelda.printAverageNumberOfBabies();
zelda.printAsciiArt();
```

我们在`VirtualAnimal`类中编写了`printAverageNumberOfBabies`和`printAsciiArt`方法，并且没有在任何子类中对它们进行重写。因此，当我们为`american`或`Zelda`调用这些方法时，Java 将执行`VirtualAnimal`类中定义的代码。

`printAverageNumberOfBabies`方法使用`getAverageNumberOfBabies`返回的`int`值和`getBaby`方法返回的`String`来生成代表虚拟动物平均幼崽数量的`String`。`VirtualHorse`类实现了继承的`getAverageNumberOfBabies`抽象方法，其中的代码返回`1`。`AmericanQuarterHorse`和`ShireHorse`类实现了继承的`getBaby`抽象方法，其中的代码返回代表虚拟马种类的幼崽的`String`："AQH baby"和"ShireHorse baby"。因此，我们对`printAverageNumberOfBabies`方法的调用将在每个实例中产生不同的结果，因为它们属于不同的类。

`printAsciiArt`方法使用`getAsciiArt`方法返回的`String`来打印代表虚拟马的 ASCII 艺术。`AmericanQuarterHorse`和`ShireHorse`类实现了继承的`getAsciiArt`抽象方法，其中的代码返回适用于每个类所代表的虚拟马的 ASCII 艺术的`String`。因此，我们对`printAsciiArt`方法的调用将在每个实例中产生不同的结果，因为它们属于不同的类。

以下屏幕截图显示了在 JShell 中执行前几行的结果。两个实例对在`VirtualAnimal`抽象类中编写的两个方法运行相同的代码。然而，每个类为最终被调用以生成结果并导致输出差异的方法提供了不同的实现。

![理解多态性](img/00070.jpeg)

以下行创建了一个名为`willow`的`Thoroughbred`类的实例，然后调用了它的`printAsciiArt`方法。与之前一样，JShell 将显示每个构造函数执行的消息，这是我们编写的链式构造函数的结果。示例的代码文件包含在`java_9_oop_chapter_07_01`文件夹中的`example07_01.java`文件中。

```java
Thoroughbred willow = 
    new Thoroughbred(5,
        "Willow", "Jolly Ball");
willow.printAsciiArt();
```

以下屏幕截图显示了在 JShell 中执行前几行的结果。新实例来自一个提供了`getAsciiArt`方法不同实现的类，因此，我们将看到与之前对其他实例调用相同方法时所看到的不同 ASCII 艺术。

![理解多态性](img/00071.jpeg)

以下行调用了名为`willow`的实例的`neigh`方法，使用不同数量的参数。这样，我们利用了使用不同参数重载了四次的`neigh`方法。请记住，我们在`VirtualHorse`类中编写了这四个`neigh`方法，而`Thoroughbred`类通过其继承树从这个超类继承了重载的方法。示例的代码文件包含在`java_9_oop_chapter_07_01`文件夹中的`example07_01.java`文件中。

```java
willow.neigh();
willow.neigh(2);
willow.neigh(2, american);
willow.neigh(3, zelda, true);
american.nicker();
american.nicker(2);
american.nicker(2, willow);
american.nicker(3, willow, true);
```

以下屏幕截图显示了在 JShell 中使用不同参数调用`neigh`和`nicker`方法的结果：

![理解多态性](img/00072.jpeg)

我们为名为`willow`的`Thoroughbred`实例调用了`VirtualHorse`类中定义的`neigh`方法的四个版本。调用`neigh`方法的第三行和第四行指定了类型为`VirtualDomesticMammal`的`otherDomesticMammal`参数的值。第三行指定`american`作为`otherDomesticMammal`的值，第四行指定相同参数的值为`zelda`。`AmericanQuarterHorse`和`ShireHorse`具体类都是`VirtualHorse`的子类，`VirtualHorse`是`VirtualDomesticMammal`的子类。因此，我们可以在需要`VirtualDomesticMammal`实例的地方使用`american`和`zelda`作为参数。

然后，我们为名为`american`的`AmericanQuarterHorse`实例调用了`VirtualHorse`类中定义的`nicker`方法的四个版本。调用`nicker`方法的第三行和第四行指定了类型为`VirtualDomesticMammal`的`otherDomesticMammal`参数的值为`willow`。`Thoroughbred`具体类也是`VirtualHorse`的子类，`VirtualHorse`是`VirtualDomesticMammal`的子类。因此，我们可以在需要`VirtualDomesticMammal`实例的地方使用`willow`作为参数。

# 控制子类中成员的可覆盖性

我们将编写`VirtualDomesticCat`抽象类及其具体子类：`MaineCoon`。然后，我们将编写`VirtualBird`抽象类、其`VirtualDomesticBird`抽象子类和`Cockatiel`具体子类。最后，我们将编写`VirtualDomesticRabbit`具体类。在编写这些类时，我们将使用 Java 9 的功能，允许我们决定子类是否可以覆盖特定成员。

所有虚拟家猫都必须能够说话，因此，我们将覆盖从`VirtualDomesticMammal`继承的`talk`方法，以打印代表猫叫声的单词：“`"Meow"`”。我们还希望提供一个方法来指定打印`"Meow"`的次数。因此，此时我们意识到我们可以利用在`VirtualHorse`类中声明的`printSoundInWords`方法。

我们无法在`VirtualDomesticCat`抽象类中访问此实例方法，因为它不是从`VirtualHorse`继承的。因此，我们将把这个方法从`VirtualHorse`类移动到它的超类：`VirtualDomesticMammal`。

### 提示

我们将在不希望在子类中被覆盖的方法的返回类型前使用`final`关键字。当一个方法被标记为最终方法时，子类无法覆盖该方法，如果它们尝试这样做，Java 9 编译器将显示错误。

并非所有的鸟类在现实生活中都能飞。然而，我们所有的虚拟鸟类都能飞，因此，我们将实现继承的`isAbleToFly`抽象方法作为一个返回`true`的最终方法。这样，我们确保所有继承自`VirtualBird`抽象类的类都将始终运行此代码以进行`isAbleToFly`方法，并且它们将无法对其进行覆盖。

以下 UML 图显示了我们将编写的新抽象和具体类的成员。此外，该图显示了从`VirtualHorse`抽象类移动到`VirtualDomesticMammal`抽象类的`printSoundInWords`方法。

![控制子类中成员的可覆盖性](img/00073.jpeg)

首先，我们将创建`VirtualDomesticMammal`抽象类的新版本。我们将添加在`VirtualHorse`抽象类中的`printSoundInWords`方法，并使用`final`关键字指示我们不希望允许子类覆盖此方法。以下行显示了`VirtualDomesticMammal`类的新代码。示例的代码文件包含在`java_9_oop_chapter_07_01`文件夹中的`example07_02.java`文件中。

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
        int age, String name, String favoriteToy) {
        this(age, false, name, favoriteToy);
    }

 protected final void printSoundInWords(
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

    public void talk() {
        System.out.println(
            String.format("%s: says something", name));
    }
}
```

在输入上述行后，JShell 将显示以下消息：

```java
|    update replaced class VirtualHorse which cannot be referenced until this error is corrected:
|      printSoundInWords(java.lang.String,int,VirtualDomesticMammal,boolean) in VirtualHorse cannot override printSoundInWords(java.lang.String,int,VirtualDomesticMammal,boolean) in VirtualDomesticMammal
|        overridden method is final
|          protected void printSoundInWords(String soundInWords, int times,
|          ^---------------------------------------------------------------...
|    update replaced class AmericanQuarterHorse which cannot be referenced until class VirtualHorse is declared
|    update replaced class ShireHorse which cannot be referenced until class VirtualHorse is declared
|    update replaced class Thoroughbred which cannot be referenced until class VirtualHorse is declared
|    update replaced variable american which cannot be referenced until class AmericanQuarterHorse is declared
|    update replaced variable zelda which cannot be referenced until class ShireHorse is declared
|    update replaced variable willow which cannot be referenced until class Thoroughbred is declared
|    update overwrote class VirtualDomesticMammal

```

JShell 告诉我们，`VirtualHorse`类及其子类在我们纠正该类的错误之前不能被引用。该类声明了`printSoundInWords`方法，并在`VirtualDomesticMammal`类中重写了最近添加的具有相同名称和参数的方法。我们在新声明中使用了`final`关键字，以确保任何子类都不能覆盖它，因此，Java 编译器生成了 JShell 显示的错误消息。

现在，我们将创建`VirtualHorse`抽象类的新版本。以下行显示了删除了`printSoundInWords`方法并使用`final`关键字确保许多方法不能被任何子类覆盖的新版本。在下面的行中，使用`final`关键字避免方法被覆盖的声明已经被突出显示。示例的代码文件包含在`java_9_oop_chapter_07_01`文件夹中的`example07_02.java`文件中。

```java
public abstract class VirtualHorse extends VirtualDomesticMammal {
    public VirtualHorse(
        int age, 
        boolean isPregnant, 
        String name, 
        String favoriteToy) {
        super(age, isPregnant, name, favoriteToy);
        System.out.println("VirtualHorse created.");        
    }

    public VirtualHorse(
        int age, String name, String favoriteToy) {
        this(age, false, name, favoriteToy);
    }

 public final boolean isAbleToFly() {
        return false;
    }

 public final boolean isRideable() {
        return true;
    }

 public final boolean isHerbivore() {
        return true;
    }

 public final boolean isCarnivore() {
        return false;
    }

    public int getAverageNumberOfBabies() {
        return 1;
    }

    public abstract String getBreed();

 public final void printBreed() {
        System.out.println(getBreed());
    }

 public final void printNeigh(
 int times, 
 VirtualDomesticMammal otherDomesticMammal,
 boolean isAngry) {
        printSoundInWords("Neigh ", times, otherDomesticMammal, isAngry);
    }

 public final void neigh() {
        printNeigh(1, null, false);
    }

 public final void neigh(int times) {
        printNeigh(times, null, false);
    }

 public final void neigh(int times, 
 VirtualDomesticMammal otherDomesticMammal) {
        printNeigh(times, otherDomesticMammal, false);
    }

 public final void neigh(int times, 
 VirtualDomesticMammal otherDomesticMammal, 
 boolean isAngry) {
        printNeigh(times, otherDomesticMammal, isAngry);
    }

 public final void printNicker(int times, 
 VirtualDomesticMammal otherDomesticMammal,
 boolean isAngry) {
        printSoundInWords("Nicker ", times, otherDomesticMammal, isAngry);
    }

 public final void nicker() {
        printNicker(1, null, false);
    }

 public final void nicker(int times) {
        printNicker(times, null, false);
    }

 public final void nicker(int times, 
 VirtualDomesticMammal otherDomesticMammal) {
        printNicker(times, otherDomesticMammal, false);
    }

 public final void nicker(int times, 
 VirtualDomesticMammal otherDomesticMammal, 
 boolean isAngry) {
        printNicker(times, otherDomesticMammal, isAngry);
    }

 @Override
 public final void talk() {
        nicker();
    }
}
```

输入上述行后，JShell 将显示以下消息：

```java
|    update replaced class AmericanQuarterHorse
|    update replaced class ShireHorse
|    update replaced class Thoroughbred
|    update replaced variable american, reset to null
|    update replaced variable zelda, reset to null
|    update replaced variable willow, reset to null
|    update overwrote class VirtualHorse

```

我们替换了`VirtualHorse`类的定义，并且子类也已更新。重要的是要知道，在 JShell 中声明的变量，它们持有`VirtualHorse`的子类实例的引用被设置为 null。

# 控制类的子类化

`final`关键字有一个额外的用法。我们可以在类声明中的`class`关键字之前使用`final`作为修饰符，告诉 Java 我们要生成一个**final 类**，即一个不能被扩展或子类化的类。Java 9 不允许我们为 final 类创建子类。

现在，我们将创建`VirtualDomesticCat`抽象类，然后我们将声明一个名为`MaineCoon`的具体子类作为 final 类。这样，我们将确保没有人能够创建`MaineCoon`的子类。以下行显示了`VirtualDomesticCat`抽象类的代码。示例的代码文件包含在`java_9_oop_chapter_07_01`文件夹中的`example07_02.java`文件中。

```java
public abstract class VirtualDomesticCat extends VirtualDomesticMammal {
    public VirtualDomesticCat(
        int age, 
        boolean isPregnant, 
        String name, 
        String favoriteToy) {
        super(age, isPregnant, name, favoriteToy);
        System.out.println("VirtualDomesticCat created.");        
    }

    public VirtualDomesticCat(
        int age, String name, String favoriteToy) {
        this(age, false, name, favoriteToy);
    }

    public final boolean isAbleToFly() {
        return false;
    }

    public final boolean isRideable() {
        return false;
    }

    public final boolean isHerbivore() {
        return false;
    }

    public final boolean isCarnivore() {
        return true;
    }

    public int getAverageNumberOfBabies() {
        return 5;
    }

    public final void printMeow(int times) {
        printSoundInWords("Meow ", times, null, false);
    }

    @Override
    public final void talk() {
        printMeow(1);
    }
}
```

`VirtualDomesticCat`抽象类将从`VirtualDomesticMammal`超类继承的许多抽象方法实现为 final 方法，并用 final 方法重写了`talk`方法。因此，我们将无法创建一个覆盖`isAbleToFly`方法返回`true`的`VirtualDomesticCat`子类。我们将无法拥有能够飞行的虚拟猫。

以下行显示了从`VirtualDomesticCat`继承的`MaineCoon`具体类的代码。我们将`MaineCoon`声明为 final 类，并且它重写了继承的`getAverageNumberOfBabies`方法以返回`6`。此外，该 final 类实现了以下继承的抽象方法：`getBaby`和`getAsciiArt`。示例的代码文件包含在`java_9_oop_chapter_07_01`文件夹中的`example07_02.java`文件中。

```java
public final class MaineCoon extends VirtualDomesticCat {
    public MaineCoon(
        int age, 
        boolean isPregnant, 
        String name, 
        String favoriteToy) {
        super(age, isPregnant, name, favoriteToy);
        System.out.println("MaineCoon created.");        
    }

    public MaineCoon(
        int age, String name, String favoriteToy) {
        this(age, false, name, favoriteToy);
    }

    public String getBaby() {
        return "Maine Coon baby ";
    }

    @Override
    public int getAverageNumberOfBabies() {
        return 6;
    }

    public String getAsciiArt() {
        return
            "  ^_^\n" + 
            " (*.*)\n" +
            "  |-|\n" +
            " /   \\\n";
    }
}
```

### 提示

我们没有将任何方法标记为`final`，因为在 final 类中的所有方法都是隐式 final 的。

然而，当我们在 JShell 之外运行 Java 代码时，final 类将被创建，我们将无法对其进行子类化。

现在，我们将创建从`VirtualAnimal`继承的`VirtualBird`抽象类。以下行显示了`VirtualBird`抽象类的代码。示例的代码文件包含在`java_9_oop_chapter_07_01`文件夹中的`example07_02.java`文件中。

```java
public abstract class VirtualBird extends VirtualAnimal {
    public String feathersColor;

    public VirtualBird(int age, String feathersColor) {
        super(age);
        this.feathersColor = feathersColor;
        System.out.println("VirtualBird created.");
    }

    public final boolean isAbleToFly() {
        // Not all birds are able to fly in real-life
        // However, all our virtual birds are able to fly
        return true;
    }

}
```

`VirtualBird`抽象类继承了先前声明的`VirtualAnimal`抽象类的成员，并添加了一个名为`feathersColor`的新的可变的`String`字段。新的抽象类声明了一个构造函数，该构造函数需要`age`和`feathersColor`的初始值来创建类的实例。构造函数使用`super`关键字调用来自基类或超类的构造函数，即在`VirtualAnimal`类中定义的构造函数，该构造函数需要`age`参数。在超类中定义的构造函数执行完毕后，代码设置了`feathersColor`可变字段的值，并打印了一条消息，指示已创建了一个虚拟鸟类。

`VirtualBird`抽象类实现了继承的`isAbleToFly`方法作为一个最终方法，返回`true`。我们希望确保我们应用程序领域中的所有虚拟鸟都能飞。

现在，我们将创建从`VirtualBird`继承的`VirtualDomesticBird`抽象类。以下行显示了`VirtualDomesticBird`抽象类的代码。示例的代码文件包含在`java_9_oop_chapter_07_01`文件夹中的`example07_02.java`文件中。

```java
public abstract class VirtualDomesticBird extends VirtualBird {
    public final String name;

    public VirtualDomesticBird(int age, 
        String feathersColor, 
        String name) {
        super(age, feathersColor);
        this.name = name;
        System.out.println("VirtualDomesticBird created.");
    }
}
```

`VirtualDomesticBird`抽象类继承了先前声明的`VirtualBird`抽象类的成员，并添加了一个名为`name`的新的不可变的`String`字段。新的抽象类声明了一个构造函数，该构造函数需要`age`、`feathersColor`和`name`的初始值来创建类的实例。构造函数使用`super`关键字调用来自超类的构造函数，即在`VirtualBird`类中定义的构造函数，该构造函数需要`age`和`feathersColor`参数。在超类中定义的构造函数执行完毕后，代码设置了`name`不可变字段的值，并打印了一条消息，指示已创建了一个虚拟家禽。

以下行显示了从`VirtualDomesticBird`继承的`Cockatiel`具体类的代码。我们将`Cockatiel`声明为最终类，并实现以下继承的抽象方法：`isRideable`、`isHerbivore`、`isCarnivore`、`getAverageNumberOfBabies`、`getBaby`和`getAsciiArt`。如前所述，最终类中的所有方法都是隐式最终的。示例的代码文件包含在`java_9_oop_chapter_07_01`文件夹中的`example07_02.java`文件中。

```java
public final class Cockatiel extends VirtualDomesticBird {
    public Cockatiel(int age, 
        String feathersColor, String name) {
        super(age, feathersColor, name);
        System.out.println("Cockatiel created.");
    }

    public boolean isRideable() {
        return true;
    }

    public boolean isHerbivore() {
        return true;
    }

    public boolean isCarnivore() {
        return true;
    }

    public int getAverageNumberOfBabies() {
        return 4;
    }

    public String getBaby() {
        return "Cockatiel baby ";
    }

    public String getAsciiArt() {
        return
            "     ///\n" +
            "      .////.\n" +
            "      //   //\n" +
            "      \\ (*)\\\n" +
            "      (/    \\\n" +
            "       /\\    \\\n" +
            "      ///     \\\\\n" +
            "     ///|     |\n" +
            "    ////|     |\n" +
            "   //////    /\n" +
            "  ////  \\   \\\n" +
            "  \\\\    ^    ^\n" +
            "   \\\n" +
            "    \\\n";
    }
}
```

以下行显示了从`VirtualDomesticMammal`继承的`VirtualDomesticRabbit`具体类的代码。我们将`VirtualDomesticRabbit`声明为最终类，因为我们不希望有额外的子类。我们只会在我们的应用程序领域中有一种虚拟家兔。最终类实现了以下继承的抽象方法：`isAbleToFly`、`isRideable`、`isHerbivore`、`isCarnivore`、`getAverageNumberOfBabies`、`getBaby`和`getAsciiArt`。示例的代码文件包含在`java_9_oop_chapter_07_01`文件夹中的`example07_02.java`文件中。

```java
public final class VirtualDomesticRabbit extends VirtualDomesticMammal {
    public VirtualDomesticRabbit(
        int age, 
        boolean isPregnant, 
        String name, 
        String favoriteToy) {
        super(age, isPregnant, name, favoriteToy);
        System.out.println("VirtualDomesticRabbit created.");        
    }

    public VirtualDomesticRabbit(
        int age, String name, String favoriteToy) {
        this(age, false, name, favoriteToy);
    }

    public final boolean isAbleToFly() {
        return false;
    }

    public final boolean isRideable() {
        return false;
    }

    public final boolean isHerbivore() {
        return true;
    }

    public final boolean isCarnivore() {
        return false;
    }

    public int getAverageNumberOfBabies() {
        return 6;
    }

    public String getBaby() {
        return "Rabbit baby ";
    }

    public String getAsciiArt() {
        return
            "   /\\ /\\\n" + 
            "   \\ V /\n" + 
            "   | **)\n" + 
            "   /  /\n" + 
            "  /  \\_\\_\n" + 
            "*(__\\_\\\n";
    }
}
```

### 注意

JShell 忽略`final`修饰符，因此，使用`final`修饰符声明的类将允许在 JShell 中存在子类。

# 创建与不同子类实例一起工作的方法

在声明所有新类之后，我们将创建以下两个方法，这两个方法接收一个`VirtualAnimal`实例作为参数，即`VirtualAnimal`实例或`VirtualAnimal`的任何子类的实例。每个方法调用`VirtualAnimal`类中定义的不同实例方法：`printAverageNumberOfBabies`和`printAsciiArg`。示例的代码文件包含在`java_9_oop_chapter_07_01`文件夹中的`example07_02.java`文件中。

```java
void printBabies(VirtualAnimal animal) {
    animal.printAverageNumberOfBabies();
}

void printAsciiArt(VirtualAnimal animal) {
    animal.printAsciiArt();
}
```

然后以下行创建了下列类的实例：`Cockatiel`、`VirtualDomesticRabbit`和`MaineCoon`。示例的代码文件包含在`java_9_oop_chapter_07_01`文件夹中的`example07_02.java`文件中。

```java
Cockatiel tweety = 
    new Cockatiel(3, "White", "Tweety");
VirtualDomesticRabbit bunny = 
    new VirtualDomesticRabbit(2, "Bunny", "Sneakers");
MaineCoon garfield = 
    new MaineCoon(3, "Garfield", "Lassagna");
```

以下截图显示了在 JShell 中执行先前行的结果。在我们输入代码创建每个实例后，我们将看到不同构造函数在 JShell 中显示的消息。这些消息将帮助我们轻松理解 Java 在创建每个实例时调用的所有链接构造函数。

![创建与不同子类实例一起工作的方法](img/00074.jpeg)

然后，以下行调用了`printBabies`和`printAsciiArt`方法，并将先前创建的实例作为参数传递。示例的代码文件包含在`java_9_oop_chapter_07_01`文件夹中的`example07_02.java`文件中。

```java
System.out.println(tweety.name);
printBabies(tweety);
printAsciiArt(tweety);

System.out.println(bunny.name);
printBabies(bunny);
printAsciiArt(bunny);

System.out.println(garfield.name);
printBabies(garfield);
printAsciiArt(garfield);
```

这三个实例成为不同方法的`VirtualAnimal`参数，即它们采用`VirtualAnimal`实例的形式。然而，字段和方法使用的值并非在`VirtualAnimal`类中声明的。对`printAverageNumberOfBabies`和`printAsciiArt`实例方法的调用考虑了所有在子类中声明的成员，因为每个实例都是`VirtualAnimal`的子类的实例：

### 提示

接受`VirtualAnimal`实例作为参数的`printBabies`和`printAsciiArt`方法只能访问为它们接收的实例在`VirtualAnimal`类中定义的成员，因为参数类型是`VirtualAnimal`。如果需要，我们可以解开接收到的`animal`参数中的`Cockatiel`、`VirtualDomesticRabbit`和`MaineCoon`实例。然而，随着我们涵盖更高级的主题，我们将在以后处理这些情景。

以下截图显示了在 JShell 中为名为`tweety`的`Cockatiel`实例执行先前行的结果。

![创建与不同子类实例一起工作的方法](img/00075.jpeg)

以下截图显示了在 JShell 中为名为`bunny`的`VirtualDomesticRabbit`实例执行先前行的结果。

![创建与不同子类实例一起工作的方法](img/00076.jpeg)

以下截图显示了在 JShell 中为名为`garfield`的`MaineCoon`实例执行先前行的结果。

![创建与不同子类实例一起工作的方法](img/00077.jpeg)

现在我们将创建另一个方法，该方法接收一个`VirtualDomesticMammal`实例作为参数，即`VirtualDomesticMammal`实例或`VirtualDomesticMammal`的任何子类的实例。以下函数调用了在`VirtualDomesticMammal`类中定义的`talk`实例方法。示例的代码文件包含在`java_9_oop_chapter_07_01`文件夹中的`example07_02.java`文件中。

```java
void makeItTalk(VirtualDomesticMammal domestic) {
    domestic.talk();
}
```

然后，以下两行调用了`makeItTalk`方法，并将`VirtualDomesticRabbit`和`MaineCoon`实例作为参数：`bunny`和`garfield`。示例的代码文件包含在`java_9_oop_chapter_07_01`文件夹中的`example07_02.java`文件中。

```java
makeItTalk(bunny);
makeItTalk(garfield);
```

对接收到的`VirtualDomesticMammal`实例调用相同方法会产生不同的结果。`VirtualDomesticRabbit`没有覆盖继承的`talk`方法，而`MaineCoon`类继承了在`VirtualDomesticCat`抽象类中被覆盖的`talk`方法，使家猫发出喵喵的声音。以下截图显示了在 JShell 中进行的两个方法调用的结果。

![创建与不同子类实例一起工作的方法](img/00078.jpeg)

`VirtualAnimal`抽象类声明了两个实例方法，允许我们确定虚拟动物是比另一个虚拟动物更年轻还是更年长：`isYoungerThan`和`isOlderThan`。这两个方法接收一个`VirtualAnimal`参数，并返回在实例的`age`值和接收实例的`age`值之间应用运算符的结果。

以下行调用`printAge`方法的三个实例：`tweety`，`bunny`和`garfield`。此方法在`VirtualAnimal`类中声明。然后，下一行调用`isOlderThan`和`isYoungerThan`方法，并将这些实例作为参数，以显示比较不同实例年龄的结果。示例的代码文件包含在`java_9_oop_chapter_07_01`文件夹中的`example07_02.java`文件中。

```java
tweety.printAge();
bunny.printAge();
garfield.printAge();
tweety.isOlderThan(bunny);
garfield.isYoungerThan(tweety);
bunny.isYoungerThan(garfield);
```

以下屏幕截图显示了在 JShell 中执行前面行的结果：

![创建可以与不同子类的实例一起工作的方法](img/00079.jpeg)

# 测试您的知识

1.  以下哪行声明了一个实例方法，不能在任何子类中被覆盖：

1.  `public void talk(): final {`

1.  `public final void talk() {`

1.  `public notOverrideable void talk() {`

1.  我们有一个名为`Shape`的抽象超类。`Circle`类是`Shape`的子类，是一个具体类。如果我们创建一个名为`circle`的`Circle`实例，这个实例也将是：

1.  `Shape`的一个实例。

1.  `Circle`的子类。

1.  `Circle`的一个抽象超类。

1.  在 UML 图中，使用斜体文本格式的类名表示它们是：

1.  具体类。

1.  覆盖了至少一个从其超类继承的成员的具体类。

1.  抽象类。

1.  以下哪行声明了一个不能被子类化的类：

1.  `public final class Dog extends VirtualAnimal {`

1.  `public final class Dog subclasses VirtualAnimal {`

1.  `public final Dog subclasses VirtualAnimal {`

1.  以下哪行声明了一个名为`Circle`的具体类，可以被子类化，其超类是`Shape`抽象类：

1.  `public final class Shape extends Circle {`

1.  `public class Shape extends Circle {`

1.  `public concrete class Shape extends Circle {`

# 总结

在本章中，我们创建了许多抽象和具体类。我们学会了控制子类是否可以覆盖成员，以及类是否可以被子类化。

我们使用了许多子类的实例，并且了解到对象可以采用许多形式。我们在 JShell 中使用了许多实例及其方法，以了解我们编写的类和方法是如何执行的。我们使用了执行与具有共同超类的不同类的实例的操作的方法。

现在您已经了解了成员继承和多态性，我们准备在 Java 9 中使用接口进行契约编程，这是我们将在下一章中讨论的主题。
