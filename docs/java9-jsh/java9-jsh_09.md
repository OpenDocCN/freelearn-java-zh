# 第九章：接口的高级契约编程

在本章中，我们将深入探讨接口的契约编程。我们将更好地理解接口作为类型的工作方式。我们将：

+   使用接口作为参数的方法

+   使用接口和类进行向下转型

+   理解装箱和拆箱

+   将接口类型的实例视为不同的子类

+   利用 Java 9 中接口的默认方法

# 使用接口作为参数的方法

在上一章中，我们创建了以下五个接口：`DrawableInComic`、`DrawableInGame`、`Hideable`、`Powerable`和`Fightable`。然后，我们创建了实现不同接口的以下类，并且其中许多类还继承自超类：`SpiderDog`、`WonderCat`、`HideableWonderCat`、`PowerableWonderCat`和`FightableWonderCat`。

在 JShell 中运行以下命令以检查我们创建的所有类型：

```java
/types

```

以下截图显示了在 JShell 中执行上一个命令的结果。JShell 列举了我们在会话中创建的五个接口和五个类。

![使用接口作为参数的方法](img/00085.jpeg)

当我们使用接口时，我们使用它们来指定参数类型，而不是使用类名。多个类可能实现单个接口，因此，不同类的实例可能符合特定接口的参数。

现在我们将创建先前提到的类的额外实例，并调用指定其所需参数的方法，使用接口名称而不是类名。我们将了解在方法中使用接口作为参数类型时发生了什么。

在以下代码中，前两行创建了`SpiderDog`类的两个实例，分别命名为`teddy`和`winston`。然后，代码调用了`teddy`的`drawSpeechBalloon`方法的两个版本。对该方法的第二次调用将`winston`作为`DrawableInComic`参数传递，因为`winston`是`SpiderDog`的一个实例，而`SpiderDog`是实现`DrawableInComic`实例的类。示例的代码文件包含在`java_9_oop_chapter_09_01`文件夹中的`example09_01.java`文件中。

```java
SpiderDog teddy = new SpiderDog("Teddy");
SpiderDog winston = new SpiderDog("Winston");
teddy.drawSpeechBalloon(
    String.format("Hello, my name is %s", teddy.getNickName()));
teddy.drawSpeechBalloon(winston, "How do you do?");
winston.drawThoughtBalloon("Who are you? I think.");
```

以下代码创建了一个名为`oliver`的`WonderCat`类的实例。在构造函数中为`nickName`参数指定的值为`"Oliver"`。下一行调用了新实例的`drawSpeechBalloon`方法，介绍了`Oliver`在漫画中，然后`teddy`调用了`drawSpeechBalloon`方法，并将`oliver`作为`DrawableInComic`参数传递，因为`oliver`是`WonderCat`的一个实例，而`WonderCat`是实现`DrawableInComic`实例的类。因此，我们也可以在需要`DrawableInComic`参数时使用`WonderCat`的实例。示例的代码文件包含在`java_9_oop_chapter_09_01`文件夹中的`example09_01.java`文件中。

```java
WonderCat oliver = 
    new WonderCat("Oliver", 10, "Mr. Oliver", 0, 15, 25);
oliver.drawSpeechBalloon(
    String.format("Hello, my name is %s", oliver.getNickName()));
teddy.drawSpeechBalloon(oliver, 
    String.format("Hello %s", oliver.getNickName()));
```

以下代码创建了一个名为`misterHideable`的`HideableWonderCat`类的实例。在构造函数中为`nickName`参数指定的值为`"Mr. Hideable"`。下一行检查了使用`oliver`作为参数调用`isIntersectingWith`方法是否返回`true`。该方法需要一个`DrawableInComic`参数，因此我们可以使用`oliver`。该方法将返回`true`，因为两个实例的`x`和`y`字段具有相同的值。`if`块中的行调用了`misterHideable`的`setLocation`方法。然后，代码调用了`show`方法。示例的代码文件包含在`java_9_oop_chapter_09_01`文件夹中的`example09_01.java`文件中。

```java
HideableWonderCat misterHideable = 
    new HideableWonderCat("Mr. Hideable", 310, 
        "Mr. John Hideable", 67000, 15, 25, 3);
if (misterHideable.isIntersectingWith(oliver)) {
    misterHideable.setLocation(
        oliver.getX() + 30, oliver.getY() + 30);
}
misterHideable.show();
```

以下代码创建了一个名为`merlin`的`PowerableWonderCat`类的实例。在构造函数中为`nickName`参数指定的值是`"Merlin"`。接下来的几行调用了`setLocation`和`draw`方法。然后，代码使用`misterHideable`作为`Hideable`参数调用了`useSpellToHide`方法。该方法需要一个`Hideable`参数，因此我们可以使用`HideableWonderCat`的先前创建的实例`misterHideable`，该实例实现了`Hideable`接口。然后，对`misterHideable`的`show`方法的调用使具有三只眼睛的`Hideable`再次出现。示例的代码文件包含在`java_9_oop_chapter_09_01`文件夹中的`example09_01.java`文件中。

```java
PowerableWonderCat merlin = 
    new PowerableWonderCat("Merlin", 35, 
        "Mr. Merlin", 78000, 30, 40, 200);
merlin.setLocation(
    merlin.getX() + 5, merlin.getY() + 5);
merlin.draw();
merlin.useSpellToHide(misterHideable);
misterHideable.show();
```

以下代码创建了一个名为`spartan`的`FightableWonderCat`类的实例。在构造函数中为`nickName`参数指定的值是`"Spartan"`。接下来的几行调用了`setLocation`和`draw`方法。然后，代码使用`misterHideable`作为参数调用了`unsheathSword`方法。该方法需要一个`Hideable`参数，因此我们可以使用`HideableWonderCat`的先前创建的实现`Hideable`接口的实例`misterHideable`。示例的代码文件包含在`java_9_oop_chapter_09_01`文件夹中的`example09_01.java`文件中。

```java
FightableWonderCat spartan = 
    new FightableWonderCat("Spartan", 28, 
        "Sir Spartan", 1000000, 60, 60, 100, 50);
spartan.setLocation(
    spartan.getX() + 30, spartan.getY() + 10);
spartan.draw();
spartan.unsheathSword(misterHideable);

```

最后，代码调用了`misterHideable`的`drawThoughtBalloon`和`drawSpeechBalloon`方法。我们可以调用这些方法，因为`misterHideable`是`HideableWonderCat`的一个实例，而这个类从其超类`WonderCat`继承了`DrawableInComic`接口的实现。

对`drawSpeechBalloon`方法的调用将`spartan`作为`DrawableInComic`参数，因为`spartan`是`FightableWonderCat`的一个实例，它是一个类，也从其超类`WonderCat`继承了`DrawableInComic`接口的实现。因此，我们还可以在需要`DrawableInComic`参数时使用`FightableWonderCat`的实例，就像下面的代码中所做的那样。示例的代码文件包含在`java_9_oop_chapter_09_01`文件夹中的`example09_01.java`文件中。

```java
misterHideable.drawThoughtBalloon(
    "I guess I must be friendly...");
misterHideable.drawSpeechBalloon(
    spartan, "Pleased to meet you, Sir!");
```

在 JShell 中执行了前面解释的所有代码片段后，我们将看到以下文本输出：

```java
Teddy -> Hello, my name is Teddy
Teddy -> message: Winston, How do you do?
Winston -> ***Who are you? I think.***
Oliver -> Meow
Teddy -> message: Oliver, Hello Oliver
Moving WonderCat Mr. John Hideable to x:45, y:55
My name is Mr. John Hideable and you can see my 3 eyes.
Moving WonderCat Mr. Merlin to x:35, y:45
Drawing WonderCat Mr. Merlin at x:35, y:45
Mr. Merlin uses his 200 spell power to hide the Hideable with 3 eyes.
My name is Mr. John Hideable and you can see my 3 eyes.
Moving WonderCat Sir Spartan to x:90, y:70
Drawing WonderCat Sir Spartan at x:90, y:70
Sir Spartan unsheaths his sword.
Sword power: 100\. Sword weight: 50.
The sword targets a Hideable with 3 eyes.
Mr. Hideable thinks: 'I guess I must be friendly...'
Spartan ==> Mr. Hideable --> Pleased to meet you, Sir!

```

# 使用接口和类进行向下转型

`DrawableInComic`接口定义了`drawSpeechBalloon`方法的一个方法要求，其参数为`DrawableInComic`类型的`destination`，这与接口定义的类型相同。以下是我们示例代码中调用此方法的第一行：

```java
teddy.drawSpeechBalloon(winston, "How do you do?");
```

我们调用了`SpiderDog`类中实现的方法，因为`teddy`是`SpiderDog`的一个实例。我们将`SpiderDog`实例`winston`传递给`destination`参数。该方法使用`destination`参数作为实现`DrawableInComic`接口的实例。因此，每当我们引用`destination`变量时，我们只能看到`DrawableInComic`类型定义的内容。

当 Java 将类型从其原始类型向下转换为目标类型时，例如转换为类符合的接口，我们可以很容易地理解发生了什么。在这种情况下，`SpiderDog`被向下转换为`DrawableInComic`。如果我们在 JShell 中输入以下代码并按*Tab*键，JShell 将枚举名为`winston`的`SpiderDog`实例的成员：

```java
winston.
```

JShell 将显示以下成员：

```java
drawSpeechBalloon(    drawThoughtBalloon(   equals(
getClass()            getNickName()         hashCode()
nickName              notify()              notifyAll()
speak(                think(                toString()
wait(

```

每当我们要求 JShell 列出成员时，它将包括从`java.lang.Object`继承的以下成员：

```java
equals(       getClass()    hashCode()    notify()      notifyAll()
toString()    wait(

```

删除先前输入的代码（`winston.`）。如果我们在 JShell 中输入以下代码并按*Tab*键，括号中的`DrawableInComic`接口类型作为`winston`变量的前缀将强制将其降级为`DrawableInComic`接口类型。因此，JShell 将只列举`SpiderDog`实例`winston`中作为`DrawableInComic`接口所需成员：

```java
((DrawableInComic) winston).
```

JShell 将显示以下成员：

```java
drawSpeechBalloon(    drawThoughtBalloon(   equals(
getClass()            getNickName()         hashCode()
notify()              notifyAll()           toString()
wait(

```

让我们看一下当我们输入`winston.`并按*Tab*键时的结果与最新结果之间的区别。上一个列表中显示的成员不包括在`SpiderDog`类中定义但在`DrawableInComic`接口中不是必需的两个方法：`speak`和`think`。因此，当 Java 将`winston`降级为`DrawableInComic`时，我们只能使用`DrawableInComic`接口所需的成员。

### 提示

如果我们使用支持自动补全功能的任何 IDE，我们会注意到在使用自动补全功能而不是在 JShell 中按*Tab*键时，成员的枚举中存在相同的差异。

现在我们将分析另一种情况，即将一个实例降级为其实现的接口之一。`DrawableInGame`接口为`isIntersectingWith`方法定义了一个对`DrawableInGame`类型的`otherDrawableInGame`参数的要求，这与接口定义的类型相同。以下是我们调用此方法的示例代码中的第一行：

```java
if (misterHideable.isIntersectingWith(oliver)) {
```

我们调用了`WonderCat`类中定义的方法，因为`misterHideable`是`HideableWonderCat`的一个实例，它继承了`WonderCat`类中`isIntersectingWith`方法的实现。我们将`WonderCat`实例`oliver`传递给了`otherDrawableInGame`参数。该方法使用`otherDrawableInGame`参数作为一个实现了`DrawableInGame`实例的实例。因此，每当我们引用`otherDrawableInGame`变量时，我们只能看到`DrawableInGame`类型定义的内容。在这种情况下，`WonderCat`被降级为`DrawableInGame`。

如果我们在 JShell 中输入以下代码并按*Tab*键，JShell 将列举`WonderCat`实例`oliver`的成员：

```java
oliver.
```

JShell 将显示`oliver`的以下成员：

```java
age                   draw()                drawSpeechBalloon(
drawThoughtBalloon(   equals(               fullName
getAge()              getClass()            getFullName()
getNickName()         getScore()            getX()
getY()                hashCode()            isIntersectingWith(
nickName              notify()              notifyAll()
score                 setLocation(          toString()
wait(                 x                     y

```

删除先前输入的代码（`oliver.`）。如果我们在 JShell 中输入以下代码并按*Tab*键，括号中的`DrawableInGame`接口类型作为`oliver`变量的前缀将强制将其降级为`DrawableInGame`接口类型。因此，JShell 将只列举`WonderCat`实例`oliver`中作为`DrawableInGame`实例所需成员：

```java
((DrawableInComic) oliver).
```

JShell 将显示以下成员：

```java
draw()                equals(               getClass()
getFullName()         getScore()            getX()
getY()                hashCode()            isIntersectingWith(
notify()              notifyAll()           setLocation(
toString()            wait(

```

让我们看一下当我们输入`oliver.`并按*Tab*键时的结果与最新结果之间的区别。当 Java 将`oliver`降级为`DrawableInGame`时，我们只能使用`DrawableInGame`接口所需的成员。

我们可以使用类似的语法来强制将先前的表达式转换为原始类型，即`WonderCat`类型。如果我们在 JShell 中输入以下代码并按*Tab*键，JShell 将再次列举`WonderCat`实例`oliver`的所有成员：

```java
((WonderCat) ((DrawableInGame) oliver)).
```

JShell 将显示以下成员，即当我们输入`oliver.`并按*Tab*键时，JShell 列举的所有成员，而没有任何类型的强制转换：

```java
age                      draw()             drawSpeechBalloon(
drawThoughtBalloon(      equals(            fullName
getAge()                 getClass()         getFullName()
getNickName()            getScore()         getX()
getY()                   hashCode()         isIntersectingWith(
nickName                 notify()           notifyAll()
score                    setLocation(       toString()
wait(                    x                  y

```

# 将接口类型的实例视为不同的子类

在第七章中，*成员继承和多态性*，我们使用了多态性。下一个示例并不代表最佳实践，因为多态性是使其工作的方式。但是，我们将编写一些代码，这些代码并不代表最佳实践，只是为了更多地了解类型转换。

以下行创建了一个名为`doSomethingWithWonderCat`的方法在 JShell 中。我们将使用这个方法来理解如何将以接口类型接收的实例视为不同的子类。示例的代码文件包含在`java_9_oop_chapter_09_01`文件夹中的`example09_02.java`文件中。

```java
// The following code is just for educational purposes
// and it doesn't represent a best practice
// We should always take advantage of polymorphism instead
public void doSomethingWithWonderCat(WonderCat wonderCat) {
    if (wonderCat instanceof HideableWonderCat) {
        HideableWonderCat hideableCat = (HideableWonderCat) wonderCat;
        hideableCat.show();
    } else if (wonderCat instanceof FightableWonderCat) {
        FightableWonderCat fightableCat = (FightableWonderCat) wonderCat;
        fightableCat.unsheathSword();
    } else if (wonderCat instanceof PowerableWonderCat) {
        PowerableWonderCat powerableCat = (PowerableWonderCat) wonderCat;
        System.out.println(
            String.format("Spell power: %d", 
                powerableCat.getSpellPower()));
    } else {
        System.out.println("This WonderCat isn't cool.");
    }
}
```

`doSomethingWithWonderCat`方法在`wonderCat`参数中接收一个`WonderCat`实例。该方法评估了许多使用`instanceof`关键字的表达式，以确定`wonderCat`参数中接收的实例是否是`HideableWonderCat`、`FightableWonderCat`或`PowerableWonder`的实例。

如果`wonderCat`是`HideableWonderCat`的实例或任何潜在的`HideableWonderCat`子类的实例，则代码声明一个名为`hideableCat`的`HideableWonderCat`局部变量，以保存`wonderCat`转换为`HideableWonderCat`的引用。然后，代码调用`hideableCat.show`方法。

如果`wonderCat`不是`HideableWonderCat`的实例，则代码评估下一个表达式。如果`wonderCat`是`FightableWonderCat`的实例或任何潜在的`FightableWonderCat`子类的实例，则代码声明一个名为`fightableCat`的`FightableWonderCat`局部变量，以保存`wonderCat`转换为`FightableWonderCat`的引用。然后，代码调用`fightableCat.unsheathSword`方法。

如果`wonderCat`不是`FightableWonderCat`的实例，则代码评估下一个表达式。如果`wonderCat`是`PowerableWonderCat`的实例或任何潜在的`PowerableWonderCat`子类的实例，则代码声明一个名为`powerableCat`的`PowerableWonderCat`局部变量，以保存`wonderCat`转换为`PowerableWonderCat`的引用。然后，代码使用`powerableCat.getSpellPower()`方法返回的结果来打印咒语能量值。

最后，如果最后一个表达式评估为`false`，则表示`wonderCat`实例只属于`WonderCat`，代码将打印一条消息，指示`WonderCat`不够酷。

### 提示

如果我们必须执行类似于此方法中显示的代码的操作，我们必须利用多态性，而不是使用`instanceof`关键字基于实例所属的类来运行代码。请记住，我们使用这个示例来更多地了解类型转换。

现在我们将在 JShell 中多次调用最近编写的`doSomethingWithWonderCat`方法。我们将使用`WonderCat`及其子类的实例调用此方法，这些实例是在我们声明此方法之前创建的。我们将使用以下值调用`doSomethingWithWonderCat`方法作为`wonderCat`参数：

+   `misterHideable`：`HideableWonderCat`类的实例

+   `spartan`：`FightableWonderCat`类的实例

+   `merlin`：`PowerableWonderCat`类的实例

+   `oliver`：`WonderCat`类的实例

以下四行在 JShell 中使用先前枚举的参数调用`doSomethingWithWonderCat`方法。示例的代码文件包含在`java_9_oop_chapter_09_01`文件夹中的`example09_02.java`文件中。

```java
doSomethingWithWonderCat(misterHideable);
doSomethingWithWonderCat(spartan);
doSomethingWithWonderCat(merlin);
doSomethingWithWonderCat(oliver);
```

以下屏幕截图显示了 JShell 为前面的行生成的输出。每次调用都会触发不同的类型转换，并调用类型转换后的实例的方法：

![将接口类型的实例视为不同的子类](img/00086.jpeg)

# 利用 Java 9 中接口的默认方法

`SpiderDog`和`WonderCat`类都实现了`DrawableInComic`接口。所有继承自`WonderCat`类的类都继承了`DrawableInComic`接口的实现。假设我们需要向`DrawableInComic`接口添加一个新的方法要求，并且我们将创建实现这个新版本接口的新类。我们将添加一个新的`drawScreamBalloon`方法，用于绘制一个带有消息的尖叫气泡。

我们将在`SpiderDog`类中添加新方法的实现。但是，假设我们无法更改实现`DrawableInComic`接口的某个类的代码：`WonderCat`。这会带来一个大问题，因为一旦我们更改了`DrawableInComic`接口的代码，Java 编译器将为`WonderCat`类生成编译错误，我们将无法编译这个类及其子类。

在这种情况下，Java 8 引入的接口默认方法以及 Java 9 中也可用的接口默认方法非常有用。我们可以为`drawScreamBalloon`方法声明一个默认实现，并将其包含在`DrawableInComic`接口的新版本中。这样，`WonderCat`类及其子类将能够使用接口中提供的方法的默认实现，并且它们将符合接口中指定的要求。

以下的 UML 图显示了`DrawableInComic`接口的新版本，其中包含了名为`drawScreamBalloon`的默认方法，以及覆盖默认方法的`SpiderDog`类的新版本。请注意，`drawScreamBalloon`方法是唯一一个不使用斜体文本的方法，因为它不是一个抽象方法。

![利用 Java 9 中接口的默认方法](img/00087.jpeg)

以下几行显示了声明`DrawableInComic`接口的新版本的代码，其中包括对`drawScreamBalloon`方法的方法要求和默认实现。请注意，在方法的返回类型之前使用`default`关键字表示我们正在声明一个默认方法。默认实现调用了每个实现接口的类将声明的`drawSpeechBalloon`方法。这样，实现这个接口的类默认情况下将在接收到绘制尖叫气泡的请求时绘制一个对话气泡。

示例的代码文件包含在`java_9_oop_chapter_09_01`文件夹中的`example09_03.java`文件中。

```java
public interface DrawableInComic {
    String getNickName();
    void drawSpeechBalloon(String message);
    void drawSpeechBalloon(DrawableInComic destination, String message);
    void drawThoughtBalloon(String message);
 default void drawScreamBalloon(String message) {
 drawSpeechBalloon(message);
 }
}
```

### 提示

在我们创建接口的新版本后，JShell 将重置所有持有实现`DrawableInComic`接口的类实例引用的变量为`null`。因此，我们将无法使用我们一直在创建的实例来测试接口的更改。

以下几行显示了`SpiderDog`类的新版本的代码，其中包括新的`drawScreamBalloon`方法。新的行已经高亮显示。示例的代码文件包含在`java_9_oop_chapter_09_01`文件夹中的`example09_03.java`文件中。

```java
public class SpiderDog implements DrawableInComic {
    protected final String nickName;

    public SpiderDog(String nickName) {
        this.nickName = nickName;
    }

    protected void speak(String message) {
        System.out.println(
            String.format("%s -> %s",
                nickName,
                message));
    }

    protected void think(String message) {
        System.out.println(
            String.format("%s -> ***%s***",
                nickName,
                message));
    }

 protected void scream(String message) {
 System.out.println(
 String.format("%s screams +++ %s +++",
 nickName,
 message));
 }

    @Override
    public String getNickName() {
        return nickName;
    }

    @Override
    public void drawSpeechBalloon(String message) {
        speak(message);
    }

    @Override
    public void drawSpeechBalloon(DrawableInComic destination, 
        String message) {
        speak(String.format("message: %s, %s",
            destination.getNickName(),
            message));
    }

    @Override
    public void drawThoughtBalloon(String message) {
        think(message);
    }

 @Override
 public void drawScreamBalloon(String message) {
 scream(message);
 }
}
```

`SpiderDog`类覆盖了`drawScreamBalloon`方法的默认实现，使用了一个调用受保护的`scream`方法的新版本，该方法以特定格式打印接收到的`message`，并将`nickName`值作为前缀。这样，这个类将不使用`DrawableInComic`接口中声明的默认实现，而是使用自己的实现。

在下面的代码中，前几行创建了`SpiderDog`类的新版本实例`rocky`，以及`FightableWonderCat`类的新版本实例`maggie`。然后，代码调用`drawScreamBalloon`方法，并为两个创建的实例`rocky`和`maggie`传递消息。示例的代码文件包含在`java_9_oop_chapter_09_01`文件夹中的`example09_03.java`文件中。

```java
SpiderDog rocky = new SpiderDog("Rocky");
FightableWonderCat maggie = 
    new FightableWonderCat("Maggie", 2, 
        "Mrs. Maggie", 5000000, 10, 10, 80, 30);
rocky.drawScreamBalloon("I am Rocky!");
maggie.drawScreamBalloon("I am Mrs. Maggie!");
```

当我们调用`rocky.drawScreamBalloon`时，Java 执行了在`SpiderDog`类中声明的这个方法的重写实现。当我们调用`maggie.drawScreamBalloon`时，Java 执行了在`DrawableInComic`接口中声明的默认方法，因为`WonderCat`和`FightableWonderCat`类都没有重写这个方法的默认实现。不要忘记`FightableWonderCat`是`WonderCat`的子类。以下截图显示了在 JShell 中执行前面几行代码的结果：

![利用 Java 9 中接口的默认方法](img/00088.jpeg)

# 测试你的知识

1.  默认方法允许我们声明：

1.  一个默认的构造函数，当实现接口的类没有声明构造函数时，Java 会使用这个默认构造函数。

1.  在实现接口的类的实例执行任何方法之前将被调用的方法。

1.  在接口中的一个方法的默认实现，当实现接口的类没有提供自己的方法实现时，Java 会使用这个默认实现。

1.  考虑到我们有一个现有的接口，许多类实现了这个接口，所有的类都能够编译通过而没有错误。如果我们向这个接口添加一个默认方法：

1.  实现接口的类在提供新方法要求的实现之前不会编译。

1.  实现接口的类在提供新构造函数要求的实现之前不会编译。

1.  实现接口的类将会编译。

1.  以下关键字中哪些允许我们确定一个实例是否是实现特定接口的类的实例：

1.  `instanceof`

1.  `isinterfaceimplementedby`

1.  `implementsinterface`

1.  以下哪些代码片段强制将`winston`变量向下转型为`DrawableInComic`接口：

1.  `(winston as DrawableInComic)`

1.  `((DrawableInComic) < winston)`

1.  `((DrawableInComic) winston)`

1.  以下哪些代码片段强制将`misterHideable`变量向下转型为`HideableWonderCat`类：

1.  `(misterHideable as HideableWonderCat)`

1.  `((HideableWonderCat) < misterHideable)`

1.  `((Hid``eableWonderCat) misterHideable)`

# 摘要

在本章中，你学会了当一个方法接收一个接口类型的参数时，在幕后发生了什么。我们使用了接收接口类型参数的方法，并且通过接口和类进行了向下转型。我们理解了如何将一个对象视为不同兼容类型的实例，以及当我们这样做时会发生什么。JShell 让我们能够轻松理解当我们使用类型转换时发生了什么。

我们利用了接口中的默认方法。我们可以向接口添加一个新方法并提供默认实现，以避免破坏我们无法编辑的现有代码。

现在你已经学会了在接口中使用高级场景，我们准备在 Java 9 中通过泛型最大化代码重用，这是我们将在下一章讨论的主题。 
