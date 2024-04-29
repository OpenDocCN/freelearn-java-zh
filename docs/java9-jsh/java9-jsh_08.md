# 第八章。接口的契约编程

在本章中，我们将处理复杂的场景，在这些场景中，我们将不得不使用属于多个蓝图的实例。我们将利用接口来进行契约编程。我们将：

+   了解 Java 9 中的接口

+   了解接口与类结合的工作原理

+   在 Java 9 中声明接口

+   声明实现接口的类

+   利用接口的多重继承

+   将类继承与接口结合

# 了解接口与类结合的工作原理

假设我们必须开发一个 Web 服务，在其中我们必须处理两种不同类型的角色：漫画角色和游戏角色。

漫画角色必须在漫画中可绘制。漫画角色必须能够提供昵称并执行以下任务：

+   绘制一个带有消息的语音气泡，也称为语音气泡

+   绘制一个带有消息的思想气泡，也称为思想气泡

+   绘制带有消息的语音气泡和另一个漫画角色，在漫画中可绘制，作为目标。

游戏角色必须在游戏场景中可绘制。游戏角色必须能够提供全名和当前得分。此外，游戏角色必须能够执行以下任务：

+   将其所需的位置设置为由*x*和*y*坐标指示的特定 2D 位置

+   为其*x*坐标提供值

+   为其*y*坐标提供值

+   在当前位置绘制自身

+   检查它是否与另一个游戏角色相交，在游戏场景中可绘制

我们必须能够处理既是漫画角色又是游戏角色的对象；也就是说，它们既可以在漫画中绘制，也可以在游戏场景中绘制。然而，我们还将处理只是漫画或游戏角色的对象；也就是说，它们可以在漫画中绘制或在游戏场景中绘制。

我们不想编写执行先前描述的任务的通用方式。我们希望确保许多类能够通过一个公共接口执行这些任务。在漫画中声明自己为可绘制的每个对象必须定义与语音和思想气泡相关的任务。在游戏场景中声明自己为可绘制的每个对象必须定义如何设置其所需的 2D 位置，绘制自身，并检查它是否与另一个游戏角色相交，在游戏场景中可绘制。

**SpiderDog**是一种漫画角色，在漫画中可绘制，具有特定的绘制语音和思想气泡的方式。**WonderCat**既是漫画角色又是游戏角色，在漫画中可绘制，也在游戏场景中可绘制。因此，WonderCat 必须定义两种角色类型所需的所有任务。

WonderCat 是一个非常多才多艺的角色，它可以使用不同的服装参与游戏或漫画，并具有不同的名称。WonderCat 还可以是可隐藏的、可供能力的或可战斗的：

+   可隐藏的角色能够被隐藏。它可以提供特定数量的眼睛，并且必须能够显示和隐藏自己。

+   可供能力的角色能够被赋予能力。它可以提供一个法术能力分数值，并使用这个法术能力使一个可隐藏的角色消失。

+   可战斗的角色能够战斗。它有一把剑，并且可以提供剑的力量和重量值。此外，可战斗的角色可以在有或没有可隐藏的角色作为目标时拔出剑。

假设 Java 9 支持多重继承。我们需要基本蓝图来表示漫画角色和游戏角色。然后，代表这些类型角色的每个类都可以提供其方法的实现。在这种情况下，漫画和游戏角色非常不同，它们不执行可能导致混乱和问题的相似任务，因此多重继承不方便。因此，我们可以使用多重继承来创建一个`WonderCat`类，该类实现了漫画和游戏角色的蓝图。在某些情况下，多重继承不方便，因为相似的蓝图可能具有相同名称的方法，并且使用多重继承可能会非常令人困惑。

此外，我们可以使用多重继承将`WonderCat`类与`Hideable`、`Powerable`和`Fightable`结合在一起。这样，我们将有一个`Hideable` + `WonderCat`，一个`Powerable` + `WonderCat`，和一个`Fightable` + `WonderCat`。我们可以使用任何一个，`Hideable` + `WonderCat`，`Powerable` + `WonderCat`，或`Fightable` + `WonderCat`，作为漫画或游戏角色。

我们的目标很简单，但我们面临一个小问题：Java 9 不支持类的多重继承。相反，我们可以使用接口进行多重继承，或者将接口与类结合使用。因此，我们将使用接口和类来满足我们之前的要求。

在前几章中，我们一直在使用抽象类和具体类。当我们编写抽象类时，我们声明了构造函数、实例字段、实例方法和抽象方法。抽象类中有具体的实例方法和抽象方法。

在这种情况下，我们不需要为任何方法提供实现；我们只需要确保我们提供了具有特定名称和参数的适当方法。您可以将**接口**视为一组相关的抽象方法，类必须实现这些方法才能被视为接口名称标识的类型的成员。Java 9 不允许我们在接口中指定构造函数或实例字段的要求。还要注意接口不是类。

### 注意

在其他编程语言中，接口被称为协议。

例如，我们可以创建一个`Hideable`接口，该接口指定以下无参数方法并具有空体：

+   `getNumberOfEyes()`

+   `appear()`

+   `disappear()`

一旦我们定义了一个接口，我们就创建了一个新类型。因此，我们可以使用接口名称来指定参数的所需类型。这样，我们将使用接口作为类型，而不是使用类作为类型，并且我们可以使用实现特定接口的任何类的实例作为参数。例如，如果我们使用`Hideable`作为参数的所需类型，我们可以将实现`Hideable`接口的任何类的实例作为参数传递。

### 提示

我们可以声明继承自多个接口的接口；也就是说，接口支持多重继承。

但是，您必须考虑接口与抽象类相比的一些限制。接口不能指定构造函数或实例字段的要求，因为接口与方法和签名有关。接口可以声明对以下成员的要求：

+   类常量

+   静态方法

+   实例方法

+   默认方法

+   嵌套类型

### 注意

Java 8 增加了向接口添加默认方法的可能性。它们允许我们声明实际提供实现的方法。Java 9 保留了这一特性。

# 声明接口

现在是时候在 Java 9 中编写必要的接口了。我们将编写以下五个接口：

+   `DrawableInComic`

+   `DrawableInGame`

+   `Hideable`

+   `Powerable`

+   `Fightable`

### 提示

一些编程语言，比如 C#，使用`I`作为接口的前缀。Java 9 不使用这种接口命名约定。因此，如果你看到一个名为`IDrawableInComic`的接口，那可能是由有 C#经验的人编写的，并将命名约定转移到了 Java 领域。

以下的 UML 图表显示了我们将要编码的五个接口，其中包括在图表中的必需方法。请注意，在声明接口的每个图表中，我们在类名前包含了**<<interface>>**文本。

![声明接口](img/00080.jpeg)

以下行显示了`DrawableInComic`接口的代码。`public`修饰符，后跟`interface`关键字和接口名`DrawableInComic`，构成了接口声明。与类声明一样，接口体被括在大括号（`{}`）中。示例的代码文件包含在`java_9_oop_chapter_08_01`文件夹中，名为`example08_01.java`。

```java
public interface DrawableInComic {
    String getNickName();
    void drawSpeechBalloon(String message);
    void drawSpeechBalloon(DrawableInComic destination, String message);
    void drawThoughtBalloon(String message);
}
```

### 提示

接口中声明的成员具有隐式的`public`修饰符，因此不需要为每个方法声明指定`public`。

`DrawableInComic`接口声明了一个`getNickName`方法要求，两次重载的`drawSpeechBalloon`方法要求，以及一个`drawThoughtBalloon`方法要求。该接口只包括方法声明，因为实现`DrawableInComic`接口的类将负责提供`getNickName`方法、`drawThoughtBalloon`方法和`drawSpeechBalloon`方法的两个重载的实现。请注意，没有方法体，就像我们为抽象类声明抽象方法时一样。不需要使用`abstract`关键字来声明这些方法，因为它们是隐式抽象的。

以下行显示了`DrawableInGame`接口的代码。示例的代码文件包含在`java_9_oop_chapter_08_01`文件夹中，名为`example08_01.java`。

```java
public interface DrawableInGame {
    String getFullName();
    int getScore();
    int getX();
    int getY();
    void setLocation(int x, int y);
    void draw();
    boolean isIntersectingWith(DrawableInGame otherDrawableInGame);
}
```

`DrawableInGame`接口声明包括七个方法要求：`getFullName`、`getScore`、`getX`、`getY`、`setLocation`、`draw`和`isIntersectingWith`。

以下行显示了`Hideable`接口的代码。示例的代码文件包含在`java_9_oop_chapter_08_01`文件夹中，名为`example08_01.java`。

```java
public interface Hideable {
    int getNumberOfEyes();
    void show();
    void hide();
}
```

`Hideable`接口声明包括三个方法要求：`getNumberOfEyes`、`show`和`hide`。

以下行显示了`Powerable`接口的代码。示例的代码文件包含在`java_9_oop_chapter_08_01`文件夹中，名为`example08_01.java`。

```java
public interface Powerable {
    int getSpellPower();
    void useSpellToHide(Hideable hideable);
}
```

`Powerable`接口声明包括两个方法要求：`getSpellPower`和`useSpellToHide`。与先前声明的接口中包含的其他方法要求一样，在方法声明中，我们使用接口名作为方法声明中参数的类型。在这种情况下，`useSpellToHide`方法声明的`hideable`参数为`Hideable`。因此，我们将能够使用任何实现`Hideable`接口的类来调用该方法。

以下行显示了`Fightable`接口的代码。示例的代码文件包含在`java_9_oop_chapter_08_01`文件夹中，名为`example08_01.java`。

```java
public interface Fightable {
    int getSwordPower();
    int getSwordWeight();
    void unsheathSword();
    void unsheathSword(Hideable hideable);
}
```

`Fightable`接口声明包括四个方法要求：`getSwordPower`、`getSwordWeight`和`unsheathSword`方法的两个重载。

# 声明实现接口的类

现在，我们将在 JShell 中声明一个具体类，该类在其声明中指定实现`DrawableInComic`接口。类声明不包括超类，而是在类名（`SiperDog`）和`implements`关键字之后包括先前声明的`DrawableInComic`接口的名称。我们可以将类声明解读为“`SpiderDog`类实现`DrawableInComic`接口”。示例的代码文件包含在`java_9_oop_chapter_08_01`文件夹中的`example08_02.java`文件中。

```java
public class SpiderDog implements DrawableInComic {
}
```

Java 编译器将生成错误，因为`SpiderDog`类被声明为具体类，并且没有覆盖`DrawableInComic`接口中声明的所有抽象方法。JShell 显示以下错误，指示接口中的第一个方法声明没有被覆盖：

```java
jshell> public class SpiderDog implements DrawableInComic {
 ...> }
|  Error:
|  SpiderDog is not abstract and does not override abstract method drawThoughtBalloon(java.lang.String) in DrawableInComic

```

现在，我们将用尝试实现`DrawableInComic`接口的类替换之前声明的空`SuperDog`类，但它仍未实现其目标。以下行显示了`SuperDog`类的新代码。示例的代码文件包含在`java_9_oop_chapter_08_01`文件夹中的`example08_03.java`文件中。

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

    @Override
    String getNickName() {
        return nickName;
    }

    @Override
    void drawSpeechBalloon(String message) {
        speak(message);
    }

    @Override
    void drawSpeechBalloon(DrawableInComic destination, 
        String message) {
        speak(String.format("message: %s, %s",
            destination.getNickName(),
            message));
    }

    @Override
    void drawThoughtBalloon(String message) {
        think(message);
    }
}
```

Java 编译器将生成许多错误，因为`SpiderDog`具体类没有实现`DrawableInComic`接口。JShell 显示以下错误消息，指示接口需要许多方法声明为`public`方法。

```java
|  Error:
|  drawThoughtBalloon(java.lang.String) in SpiderDog cannot implement drawThoughtBalloon(java.lang.String) in DrawableInComic
|    attempting to assign weaker access privileges; was public
|      @Override
|      ^--------...
|  Error:
|  drawSpeechBalloon(DrawableInComic,java.lang.String) in SpiderDog cannot implement drawSpeechBalloon(DrawableInComic,java.lang.String) in DrawableInComic
|    attempting to assign weaker access privileges; was public
|      @Override
|      ^--------...
|  Error:
|  drawSpeechBalloon(java.lang.String) in SpiderDog cannot implement drawSpeechBalloon(java.lang.String) in DrawableInComic
|    attempting to assign weaker access privileges; was public
|      @Override
|      ^--------...
|  Error:
|  getNickName() in SpiderDog cannot implement getNickName() in DrawableInComic
|    attempting to assign weaker access privileges; was public
|      @Override
|      ^--------...

```

公共`DrawableInComic`接口指定了隐式公共方法。因此，当我们声明一个类时，该类没有将所需成员声明为`public`时，Java 编译器会生成错误，并指出我们不能尝试分配比接口要求的更弱的访问权限。

### 注意

每当我们声明一个指定实现接口的类时，它必须满足接口中指定的所有要求。如果不满足，Java 编译器将生成错误，指示未满足哪些要求，就像在前面的示例中发生的那样。在使用接口时，Java 编译器确保实现接口的任何类都遵守其中指定的要求。

最后，我们将用真正实现`DrawableInComic`接口的类替换`SpiderDog`类的先前声明。以下行显示了`SpiderDog`类的新代码。示例的代码文件包含在`java_9_oop_chapter_08_01`文件夹中的`example08_04.java`文件中。

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
}
```

`SpiderDog`类声明了一个构造函数，将所需的`nickName`参数的值分配给`nickName`不可变的受保护字段。该类实现了`getNickName`方法，该方法只返回`nickName`不可变的受保护字段。该类声明了两个版本的`drawSpeechBalloon`方法的代码。两种方法都调用受保护的`speak`方法，该方法打印一个包括`nickName`值作为前缀的特定格式的消息。此外，该类声明了`drawThoughtBalloon`方法的代码，该方法调用受保护的`think`方法，该方法也打印一个包括`nickName`值作为前缀的消息。

`SpiderDog`类实现了`DrawableInComic`接口中声明的方法。该类还声明了一个构造函数，一个`protected`的不可变字段和两个`protected`方法。

### 提示

只要我们实现了类声明中`implements`关键字后列出的接口中声明的所有成员，就可以向类添加任何所需的额外成员。

现在，我们将声明另一个类，该类实现了`SpiderDog`类实现的相同接口，即`DrawableInComic`接口。以下行显示了`WonderCat`类的代码。示例的代码文件包含在`java_9_oop_chapter_08_01`文件夹中的`example08_04.java`文件中。

```java
public class WonderCat implements DrawableInComic {
    protected final String nickName;
    protected final int age;

    public WonderCat(String nickName, int age) {
        this.nickName = nickName;
        this.age = age;
    }

    public int getAge() {
        return age;
    }

    @Override
 public String getNickName() {
        return nickName;
    }

    @Override
 public void drawSpeechBalloon(String message) {
        String meow = 
            (age > 2) ? "Meow" : "Meeoow Meeoow";
        System.out.println(
            String.format("%s -> %s",
                nickName,
                meow));
    }

    @Override
 public void drawSpeechBalloon(DrawableInComic destination, 
 String message) {
        System.out.println(
            String.format("%s ==> %s --> %s",
                destination.getNickName(),
                nickName,
                message));
    }

    @Override
 public void drawThoughtBalloon(String message) {
        System.out.println(
            String.format("%s thinks: '%s'",
                nickName,
                message));
    }
}
```

`WonderCat`类声明了一个构造函数，将所需的`nickName`和`age`参数的值分配给`nickName`和`age`不可变字段。该类声明了两个版本的`drawSpeechBalloon`方法的代码。只需要`message`参数的版本使用`age`属性的值，在`age`值大于`2`时生成不同的消息。此外，该类声明了`drawThoughtBalloon`和`getNickName`方法的代码。

`WonderCat`类实现了`DrawableInComic`接口中声明的方法。但是，该类还声明了一个额外的不可变字段`age`和一个`getAge`方法，这些并不是接口所要求的。

### 提示

Java 9 中的接口允许我们确保实现它们的类定义接口中指定的所有成员。如果没有，代码将无法编译。

# 利用接口的多重继承

Java 9 不允许我们声明具有多个超类或基类的类，因此不支持类的多重继承。子类只能继承一个类。但是，一个类可以实现一个或多个接口。此外，我们可以声明从超类继承并实现一个或多个接口的类。因此，我们可以将基于类的继承与接口的实现结合起来。

我们希望`WonderCat`类实现`DrawableInComic`和`DrawableInGame`接口。我们希望能够将任何`WonderCat`实例用作漫画角色和游戏角色。为了实现这一点，我们必须更改类声明，并将`DrawableInGame`接口添加到类实现的接口列表中，并在类中声明此添加接口中包含的所有方法。

以下行显示了新的类声明，指定`WonderCat`类实现`DrawableInComic`和`DrawableInGame`接口。类主体保持不变，因此我们不重复代码。示例的代码文件包含在`java_9_oop_chapter_08_01`文件夹中的`example08_05.java`文件中。

```java
public class WonderCat implements 
    DrawableInComic, DrawableInGame {
```

更改类声明后，Java 编译器将生成许多错误，因为`WonderCat`具体类的新版本没有实现`DrawableInGame`接口。JShell 显示以下错误消息。

```java
|  Error:
|  WonderCat is not abstract and does not override abstract method isIntersectingWith(DrawableInGame) in DrawableInGame
|  public class WonderCat implements
|  ^--------------------------------...

```

```java
java_9_oop_chapter_08_01 folder, in the example08_06.java file.
```

```java
public class WonderCat implements 
 DrawableInComic, DrawableInGame {
    protected final String nickName;
    protected final int age;
 protected int score;
 protected final String fullName;
 protected int x;
 protected int y;

 public WonderCat(String nickName, 
 int age, 
 String fullName, 
 int score, 
 int x, 
 int y) {
        this.nickName = nickName;
        this.age = age;
 this.fullName = fullName;
 this.score = score;
 this.x = x;
 this.y = y;
    }

    public int getAge() {
        return age;
    }

    @Override
    public String getNickName() {
        return nickName;
    }

    @Override
    public void drawSpeechBalloon(String message) {
        String meow = 
            (age > 2) ? "Meow" : "Meeoow Meeoow";
        System.out.println(
            String.format("%s -> %s",
                nickName,
                meow));
    }

    @Override
    public void drawSpeechBalloon(DrawableInComic destination, 
        String message) {
        System.out.println(
            String.format("%s ==> %s --> %s",
                destination.getNickName(),
                nickName,
                message));
    }

    @Override
    public void drawThoughtBalloon(String message) {
        System.out.println(
            String.format("%s thinks: '%s'",
                nickName,
                message));
    }

 @Override
 public String getFullName() {
 return fullName;
 }

 @Override
 public int getScore() {
 return score;
 }

 @Override
 public int getX() {
 return x;
 }

 @Override
 public int getY() {
 return y;
 }

 @Override
 public void setLocation(int x, int y) {
 this.x = x;
 this.y = y;
 System.out.println(
 String.format("Moving WonderCat %s to x:%d, y:%d",
 fullName,
 this.x,
 this.y));
 }

 @Override
 public void draw() {
 System.out.println(
 String.format("Drawing WonderCat %s at x:%d, y:%d",
 fullName,
 x,
 y));
 }

 @Override
 public boolean isIntersectingWith(
 DrawableInGame otherDrawableInGame) {
 return ((x == otherDrawableInGame.getX()) &&
 (y == otherDrawableInGame.getY()));
 }
}
```

新的构造函数将额外需要的`fullName`、`score`、`x`和`y`参数的值分配给同名的字段。因此，每当我们想要创建`AngryCat`类的实例时，我们将需要指定这些额外的参数。此外，该类添加了`DrawableInGame`接口中指定的所有方法的实现。

# 结合类继承和接口

我们可以将类继承与接口的实现结合起来。以下行显示了一个新的`HideableWonderCat`类的代码，它继承自`WonderCat`类并实现了`Hideable`接口。请注意，类声明在`extends`关键字后包括超类（`WonderCat`），在`implements`关键字后包括实现的接口（`Hideable`）。示例的代码文件包含在`java_9_oop_chapter_08_01`文件夹中的`example08_07.java`文件中。

```java
public class HideableWonderCat extends WonderCat implements Hideable {
    protected final int numberOfEyes;

    public HideableWonderCat(String nickName, int age, 
        String fullName, int score, 
        int x, int y, int numberOfEyes) {
        super(nickName, age, fullName, score, x, y);
        this.numberOfEyes = numberOfEyes;
    }

    @Override
    public int getNumberOfEyes() {
        return numberOfEyes;
    }

    @Override
    public void show() {
        System.out.println(
            String.format(
                "My name is %s and you can see my %d eyes.",
                getFullName(), 
                numberOfEyes));
    }

    @Override
    public void hide() {
        System.out.println(
            String.format(
                "%s is hidden.", 
                getFullName()));
    }
}
```

由于前面的代码，我们有了一个名为`HideableWonderCat`的新类，它实现了以下三个接口：

+   `DrawableInComic`：这个接口由`WonderCat`超类实现，并被`HideableWonderCat`继承

+   `DrawableInGame`：这个接口由`WonderCat`超类实现，并被`HideableWonderCat`继承

+   `Hideable`：这个接口由`HideableWonderCat`实现

`HideableWonderCat`类中定义的构造函数在构造函数中添加了一个`numberOfEyes`参数，该参数在`WonderCat`超类中声明的参数列表中。在这种情况下，构造函数使用`super`关键字调用超类中定义的构造函数，然后使用接收到的`numberOfEyes`参数初始化`numberOfEyes`不可变字段。该类实现了`Hideable`接口所需的`getNumberOfEyes`、`show`和`hide`方法。

以下几行显示了一个新的`PowerableWonderCat`类的代码，该类继承自`WonderCat`类并实现了`Powerable`接口。请注意，类声明在`extends`关键字后包括超类（`WonderCat`），在`implements`关键字后包括实现的接口（`Powerable`）。示例的代码文件包含在`java_9_oop_chapter_08_01`文件夹中的`example08_07.java`文件中。

```java
public class PowerableWonderCat extends WonderCat implements Powerable {
    protected final int spellPower;

    public PowerableWonderCat(String nickName, 
        int age, 
        String fullName, 
        int score, 
        int x, 
        int y, 
        int spellPower) {
        super(nickName, age, fullName, score, x, y);
        this.spellPower = spellPower;
    }

    @Override
    public int getSpellPower() {
        return spellPower;
    }

    @Override
    public void useSpellToHide(Hideable hideable) {
        System.out.println(
            String.format(
                "%s uses his %d spell power to hide the Hideable with %d eyes.",
                getFullName(),
                spellPower,
                hideable.getNumberOfEyes()));
    }
}
```

就像`HideableWonderCat`类一样，新的`PowerableWonderCat`类实现了三个接口。其中两个接口由`WonderCat`超类实现，并被`HideableWonderCat`继承：`DrawableInComic`和`DrawableInGame`。`HideableWonderCat`类添加了`Powerable`接口的实现。

`PowerableWonderCat`类中定义的构造函数在构造函数中添加了一个`spellPower`参数，该参数在`WonderCat`超类中声明的参数列表中。在这种情况下，构造函数使用`super`关键字调用超类中定义的构造函数，然后使用接收到的`spellPower`参数初始化`spellPower`不可变字段。该类实现了`Powerable`接口所需的`getSpellPower`和`useSpellToHide`方法。

`hide`方法接收一个`Hideable`作为参数。因此，任何`HideableWonderCat`的实例都可以作为该方法的参数，也就是符合`Hideable`实例的任何类的实例。

以下几行显示了一个新的`FightableWonderCat`类的代码，该类继承自`WonderCat`类并实现了`Fightable`接口。请注意，类声明在`extends`关键字后包括超类（`WonderCat`），在`implements`关键字后包括实现的接口（`Fightable`）。示例的代码文件包含在`java_9_oop_chapter_08_01`文件夹中的`example08_07.java`文件中。

```java
public class FightableWonderCat extends WonderCat implements Fightable {
    protected final int swordPower;
    protected final int swordWeight;

    public FightableWonderCat(String nickName, 
        int age, 
        String fullName, 
        int score, 
        int x, 
        int y, 
        int swordPower,
        int swordWeight) {
        super(nickName, age, fullName, score, x, y);
        this.swordPower = swordPower;
        this.swordWeight = swordWeight;
    }

    private void printSwordInformation() {
        System.out.println(
            String.format(
                "%s unsheaths his sword.", 
                getFullName()));
        System.out.println(
            String.format(
                "Sword power: %d. Sword weight: %d.", 
                swordPower,
                swordWeight));
    }

    @Override
    public int getSwordPower() {
        return swordPower;
    }

    @Override
    public int getSwordWeight() {
        return swordWeight;
    }

    @Override
    public void unsheathSword() {
        printSwordInformation();
    }

    @Override
    public void unsheathSword(Hideable hideable) {
        printSwordInformation();
        System.out.println(
            String.format("The sword targets a Hideable with %d eyes.",
                hideable.getNumberOfEyes()));
    }
}
```

就像之前编写的两个从`WonderCat`类继承并实现接口的类一样，新的`FightableWonderCat`类实现了三个接口。其中两个接口由`WonderCat`超类实现，并被`FightableWonderCat`继承：`DrawableInComic`和`DrawableInGame`。`FightableWonderCat`类添加了`Fightable`接口的实现。

`FightableWonderCat`类中定义的构造函数在构造函数中添加了`swordPower`和`swordWeight`参数，这些参数在`WonderCat`超类中声明的参数列表中。在这种情况下，构造函数使用`super`关键字调用超类中定义的构造函数，然后使用接收到的`swordPower`和`swordWeight`参数初始化`swordPower`和`swordWeight`不可变字段。

该类实现了`getSpellPower`、`getSwordWeight`和`Fightable`接口所需的两个版本的`unsheathSword`方法。两个版本的`unsheathSword`方法调用了受保护的`printSwordInformation`方法，而接收`Hideable`实例作为参数的重载版本则打印了一个额外的消息，该消息包含了`Hideable`实例的眼睛数量作为目标。

以下表格总结了我们创建的每个类实现的接口：

| 类名 | 实现以下接口 |
| --- | --- |
| `SpiderDog` | `DrawableInComic` |
| `WonderCat` | `DrawableInComic` 和 `DrawableInGame` |
| `HideableWonderCat` | `DrawableInComic`、`DrawableInGame` 和 `Hideable` |
| `PowerableWonderCat` | `DrawableInComic`、`DrawableInGame` 和 `Powerable` |
| `FightableWonderCat` | `DrawableInComic`、`DrawableInGame` 和 `Fightable` |

以下简化的 UML 图显示了类的层次结构树及其与接口的关系。该图表不包括任何接口和类的成员，以使其更容易理解关系。以虚线结束的带箭头的线表示类实现了箭头指示的接口。

![Combining class inheritance and interfaces](img/00081.jpeg)

以下 UML 图显示了接口和类及其所有成员。请注意，我们不重复类实现的接口中声明的成员，以使图表更简单，并避免重复信息。我们可以使用该图表来理解我们将在基于这些类和先前定义的接口的使用的下一个代码示例中分析的所有内容：

![Combining class inheritance and interfaces](img/00082.jpeg)

以下行创建了每个先前创建的类的一个实例。示例的代码文件包含在`java_9_oop_chapter_08_01`文件夹中的`example08_08.java`文件中。

```java
SpiderDog spiderDog1 = 
    new SpiderDog("Buddy");
WonderCat wonderCat1 = 
    new WonderCat("Daisy", 1, "Mrs. Daisy", 100, 15, 15);
HideableWonderCat hideableWonderCat1 =
    new HideableWonderCat("Molly", 5, "Mrs. Molly", 450, 20, 10, 3); 
PowerableWonderCat powerableWonderCat1 =
    new PowerableWonderCat("Princess", 5, "Mrs. Princess", 320, 20, 10, 7);
FightableWonderCat fightableWonderCat1 =
    new FightableWonderCat("Abby", 3, "Mrs. Abby", 1200, 40, 10, 7, 5);
```

以下表格总结了我们使用前面的代码片段创建的实例名称及其类名称：

| 实例名称 | 类名称 |
| --- | --- |
| `spiderDog1` | `SpiderDog` |
| `wonderCat1` | `WonderCat` |
| `hideableWonderCat1` | `HideableWonderCat` |
| `powerableWonderCat1` | `PowerableWonderCat` |
| `fightableWonderCat1` | `FightableWonderCat` |

现在，我们将评估许多使用`instanceof`关键字的表达式，以确定实例是指定类的实例还是实现特定接口的类的实例。请注意，所有表达式的评估结果都为`true`，因为在`instanceof`关键字后面的右侧指定的类型对于每个实例来说，都是它的主类、超类或主类实现的接口。

例如，`powerableWonderCat1` 是 `PowerableWonderCat` 的一个实例。此外，`powerableWonderCat1` 属于 `WonderCat`，因为 `WonderCat` 是 `PowerableWonderCat` 类的超类。同样，`powerableWonderCat1` 实现了三个接口：`DrawableInComic`、`DrawableInGame` 和 `Powerable`。`PowerableWonderCat` 的超类 `WonderCat` 实现了以下两个接口：`DrawableInComic` 和 `DrawableInGame`。因此，`PowerableWonderCat` 继承了接口的实现。最后，`PowerableWonderCat` 类不仅继承自 `WonderCat`，还实现了 `Powerable` 接口。

在第三章*Classes and Instances*中，我们学习了`instanceof`关键字允许我们测试对象是否是指定类型。这种类型可以是类，也可以是接口。如果我们在 JShell 中执行以下行，所有这些行的评估结果都将打印为`true`。示例的代码文件包含在`java_9_oop_chapter_08_01`文件夹中的`example08_08.java`文件中。

```java
spiderDog1 instanceof SpiderDog
spiderDog1 instanceof DrawableInComic

wonderCat1 instanceof WonderCat
wonderCat1 instanceof DrawableInComic
wonderCat1 instanceof DrawableInGame

hideableWonderCat1 instanceof WonderCat
hideableWonderCat1 instanceof HideableWonderCat
hideableWonderCat1 instanceof DrawableInComic
hideableWonderCat1 instanceof DrawableInGame
hideableWonderCat1 instanceof Hideable

powerableWonderCat1 instanceof WonderCat
powerableWonderCat1 instanceof PowerableWonderCat
powerableWonderCat1 instanceof DrawableInComic
powerableWonderCat1 instanceof DrawableInGame
powerableWonderCat1 instanceof Powerable

fightableWonderCat1 instanceof WonderCat
fightableWonderCat1 instanceof FightableWonderCat
fightableWonderCat1 instanceof DrawableInComic
fightableWonderCat1 instanceof DrawableInGame
fightableWonderCat1 instanceof Fightable
```

以下两个屏幕截图显示了在 JShell 中评估先前表达式的结果：

![Combining class inheritance and interfaces](img/00083.jpeg)![Combining class inheritance and interfaces](img/00084.jpeg)

# 测试你的知识

1.  一个类可以实现：

1.  只有一个接口。

1.  一个或多个接口。

1.  最多两个接口。

1.  当一个类实现一个接口：

1.  它也可以继承自一个超类。

1.  它不能从一个超类继承。

1.  它只能从抽象超类继承，而不能从具体超类继承。

1.  一个接口：

1.  可以从一个超类继承。

1.  不能继承自超类或另一个接口。

1.  可以继承另一个接口。

1.  哪一行声明了一个名为`WonderDog`的类，该类实现了`Hideable`接口：

1.  `public class WonderDog extends Hideable {`

1.  `public class WonderDog implements Hideable {`

1.  `public class WonderDog: Hideable {`

1.  接口是：

1.  一种方法。

1.  一种类型。

1.  抽象类。

# 总结

在本章中，您学习了声明和组合多个蓝图以生成单个实例。我们声明了指定所需方法的接口。然后，我们创建了许多实现单个和多个接口的类。

我们将类继承与接口实现结合在一起。我们意识到一个类可以实现多个接口。我们在 JShell 中执行代码，以了解单个实例属于类类型和接口类型。

现在您已经了解了接口和基本的契约编程知识，我们准备开始处理高级契约编程场景，这是我们将在下一章讨论的主题。
