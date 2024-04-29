# 第十章。通过泛型最大化代码重用

在本章中，我们将学习参数多态以及 Java 9 如何通过允许我们编写通用代码来实现这一面向对象的概念。我们将开始创建使用受限泛型类型的类。我们将：

+   理解参数多态

+   了解参数多态和鸭子类型之间的区别

+   理解 Java 9 泛型和通用代码

+   声明一个用作类型约束的接口

+   声明符合多个接口的类

+   声明继承接口实现的子类

+   创建异常类

+   声明一个使用受限泛型类型的类

+   使用一个通用类来处理多个兼容类型

# 理解参数多态、Java 9 泛型和通用代码

想象一下，我们开发了一个 Web 服务，必须使用特定野生动物聚会的组织表示。我们绝对不希望把狮子和鬣狗混在一起，因为聚会最终会以鬣狗吓唬一只孤狮而结束。我们希望一个组织有序的聚会，不希望有入侵者，比如龙或猫，出现在只有狮子应该参加的聚会中。

我们想描述启动程序、欢迎成员、组织聚会以及向聚会的不同成员道别的程序。然后，我们想在天鹅聚会中复制这些程序。因此，我们希望重用我们的程序来举办狮子聚会和天鹅聚会。将来，我们将需要使用相同的程序来举办其他野生动物和家养动物的聚会，比如狐狸、鳄鱼、猫、老虎和狗。显然，我们不希望成为鳄鱼聚会的入侵者。我们也不想参加老虎聚会。

在前几章中，第八章，“使用接口进行合同编程”，和第九章，“使用接口进行高级合同编程”，我们学习了如何在 Java 9 中使用接口。我们可以声明一个接口来指定可以参加聚会的动物的要求，然后利用 Java 9 的特性编写通用代码，可以与实现接口的任何类一起使用。

### 提示

**参数多态**允许我们编写通用和可重用的代码，可以处理值而不依赖于类型，同时保持完全的静态类型安全。

我们可以通过泛型在 Java 9 中利用参数多态，也称为通用编程。在我们声明一个指示可以参加聚会的动物要求的接口之后，我们可以创建一个可以与实现此接口的任何实例一起使用的类。这样，我们可以重用生成狮子聚会的代码，并创建天鹅、鬣狗或任何其他动物的聚会。具体来说，我们可以重用生成任何实现指定可以参加聚会的动物要求的接口的类的聚会的代码。

我们要求动物在聚会中要有社交能力，因此，我们可以创建一个名为`Sociable`的接口，来指定可以参加聚会的动物的要求。但要注意，我们将用作示例的许多野生动物并不太善于社交。

### 提示

许多现代强类型编程语言允许我们通过泛型进行参数多态。如果你有使用过 C#或 Swift，你会发现 Java 9 的语法与这些编程语言中使用的语法非常相似。C#也使用接口，但 Swift 使用协议而不是接口。

其他编程语言，如 Python、JavaScript 和 Ruby，采用一种称为**鸭子类型**的不同哲学，其中某些字段和方法的存在使对象适合于其用途作为特定的社交动物。使用鸭子类型，如果我们要求社交动物具有`getName`和`danceAlone`方法，只要对象提供了所需的方法，我们就可以将任何对象视为社交动物。因此，使用鸭子类型，任何提供所需方法的任何类型的实例都可以用作社交动物。

让我们来看一个真实的情况，以理解鸭子类型的哲学。想象一下，我们看到一只鸟，这只鸟嘎嘎叫、游泳和走路都像一只鸭子。我们肯定可以称这只鸟为鸭子，因为它满足了这只鸟被称为鸭子所需的所有条件。与鸟和鸭子相关的类似例子产生了鸭子类型的名称。我们不需要额外的信息来将这只鸟视为鸭子。Python、JavaScript 和 Ruby 是鸭子类型极为流行的语言的例子。

在 Java 9 中可以使用鸭子类型，但这不是这种编程语言的自然方式。在 Java 9 中实现鸭子类型需要许多复杂的解决方法。因此，我们将专注于学习通过泛型实现参数多态性的通用代码编写。

# 声明一个接口用作类型约束

首先，我们将创建一个`Sociable`接口，以指定类型必须满足的要求，才能被视为潜在的聚会成员，也就是我们应用领域中的社交动物。然后，我们将创建一个实现了这个接口的`SociableAnimal`抽象基类，然后，我们将在三个具体的子类中专门化这个类：`SocialLion`、`SocialParrot`和`SocialSwan`。然后，我们将创建一个`Party`类，它将能够通过泛型与实现`Sociable`接口的任何类的实例一起工作。我们将创建两个新的类，它们将代表特定的异常。我们将处理一群社交狮子、一群社交鹦鹉和一群社交天鹅。

以下的 UML 图显示了接口，实现它的抽象类，以及我们将创建的具体子类，包括所有的字段和方法：

![声明一个接口用作类型约束](img/00089.jpeg)

以下几行显示了`Sociable`接口的代码。示例的代码文件包含在`java_9_oop_chapter_10_01`文件夹中的`example10_01.java`文件中。

```java
public interface Sociable {
    String getName();
    int getAge();
    void actAlone();
    void danceAlone();
    void danceWith(Sociable partner);
    void singALyric(String lyric);
    void speak(String message);
    void welcome(Sociable other);
    void sayGoodbyeTo(Sociable other);
}
```

接口声明了以下九个方法要求：

+   `getName`：这个方法必须返回一个`String`，表示`Sociable`的名字。

+   `getAge`：这个方法必须返回一个`int`，表示`Sociable`的年龄。

+   `actAlone`：这个方法必须让`Sociable`独自行动。

+   `danceAlone`：这个方法必须让`Sociable`独自跳舞。

+   `danceWith`：这个方法必须让`Sociable`与另一个在 partner 参数中接收到的`Sociable`一起跳舞。

+   `singALyric`：这个方法必须让`Sociable`唱接收到的歌词。

+   `speak`：这个方法让`Sociable`说一条消息。

+   `welcome`：这个方法让`Sociable`向另一个在其他参数中接收到的`Sociable`说欢迎的消息。

+   `sayGoodbyeTo`：这个方法让`Sociable`向另一个在其他参数中接收到的`Sociable`说再见。

我们没有在接口声明中包含任何默认方法，因此实现`Sociable`接口的类负责实现之前列举的九个方法。

# 声明符合多个接口的类

```java
SocialAnimal abstract class. The code file for the sample is included in the java_9_oop_chapter_10_01 folder, in the example10_01.java file.
```

```java
public abstract class SocialAnimal implements Sociable, Comparable<Sociable> {
    public final String name;
    public final int age;

    public SocialAnimal(String name, int age) {
        this.name = name;
        this.age = age;
    }

    protected void printMessageWithNameAsPrefix(String message) {
        System.out.println(
            String.format("%s %s", 
                getName(), 
                message));
    }

    public abstract String getDanceRepresentation();

    public abstract String getFirstSoundInWords();

    public abstract String getSecondSoundInWords();

    public abstract String getThirdSoundInWords();

    @Override
    public String getName() {
        return name;
    }

    @Override
    public int getAge() {
        return age;
    }
```

```java
SocialAnimal class declares a constructor that assigns the value of the required name and age arguments to the immutable name and age protected fields. Then the class declares a protected printMessageWithNameAsPrefix method that receives a message and prints the name for the SocialAnimal followed by a space and this message. Many methods will call this method to easily add the name as a prefix for many messages.
SocialAnimal abstract class. The code file for the sample is included in the java_9_oop_chapter_10_01 folder, in the example10_01.java file.
```

```java
    @Override
    public void actAlone() {
        printMessageWithNameAsPrefix("to be or not to be");
    }

    @Override
    public void danceAlone() {
        printMessageWithNameAsPrefix(
            String.format("dances alone %s", 
                getDanceRepresentation()));
    }

    @Override
    public void danceWith(Sociable partner) {
        printMessageWithNameAsPrefix(
            String.format("dances with %s %s", 
                partner.getName(),
                getDanceRepresentation()));
    }

    @Override
    public void singALyric(String lyric) {
        printMessageWithNameAsPrefix(
            String.format("sings %s %s %s %s", 
                lyric,
                getFirstSoundInWords(),
                getSecondSoundInWords(),
                getThirdSoundInWords()));
    }

    @Override
    public void speak(String message) {
        printMessageWithNameAsPrefix(
            String.format("says: %s %s", 
                message,
                getDanceRepresentation()));
    }

    @Override
    public void welcome(Sociable other) {
        printMessageWithNameAsPrefix(
            String.format("welcomes %s", 
                other.getName()));
    }

    @Override
    public void sayGoodbyeTo(Sociable other) {
        printMessageWithNameAsPrefix(
            String.format("says goodbye to %s%s%s%s", 
                other.getName(),
                getFirstSoundInWords(),
                getSecondSoundInWords(),
                getThirdSoundInWords()));
    }
```

```java
 for the SocialAnimal class implements the other methods required by the Sociable interface:
```

+   `actAlone`：这个方法打印名字，后面跟着"to be or not to be"。

+   `danceAlone`：这个方法使用调用`getDanceRepresentation`方法检索到的`String`来打印名字，后面跟着指示社交动物正在跳舞的消息。

+   `danceWith`：此方法使用调用`getDanceRepresentation`方法获取的`String`来打印名称，然后是一条消息，指示社交动物正在与`Sociable`类型的 partner 参数指定的伙伴一起跳舞。消息中包括伙伴的名称。

+   `singALyric`：此方法使用调用`getFirstSoundInWords`、`getSecondSoundInWords`和`getThirdSoundInWords`获取的字符串以及作为参数接收到的歌词来打印名称，然后是一条消息，指示社交动物唱出歌词。

+   `speak`：此方法使用调用`getDanceRepresentation`获取的`String`和作为参数接收到的消息来打印名称，然后是动物说的话，再接着是它的舞蹈表示字符。

+   `welcome`：此方法打印一条消息，欢迎另一个在其他参数中接收到的`Sociable`。消息包括目的地的名称。

+   `sayGoodbyeTo`：此方法使用调用`getFirstSoundInWords`、`getSecondSoundInWords`和`getThirdSoundInWords`获取的字符串来构建并打印一条消息，向其他参数中接收到的另一个`Sociable`说再见。消息包括目的地的名称。

```java
for the SocialAnimal class overrides the compareTo method to implement the Comparable<Sociable> interface. In addition, this last code snippet for the SocialAnimal class overrides the equals method. The code file for the sample is included in the java_9_oop_chapter_10_01 folder, in the example10_01.java file.
```

```java
    @Override
    public boolean equals(Object other) {
        // Is other this object?
        if (this == other) {
            return true;
        }
        // Is other null?
        if (other == null) {
            return false;
        }
        // Does other have the same type?
        if (!getClass().equals(other.getClass())) {
            return false;
        }
        SocialAnimal otherSocialAnimal = (SocialAnimal) other;
        // Make sure both the name and age are equal
        return Objects.equals(getName(), otherSocialAnimal.getName())
        && Objects.equals(getAge(), otherSocialAnimal.getAge());
    }

    @Override
    public int compareTo(final Sociable otherSociable) {
        return Integer.compare(getAge(),otherSociable.getAge());
    }
}
```

```java
SocialAnimal class overrides the equals method inherited from java.lang.Object that receives the instance that we must compare with the actual instance in the other argument. Unluckily, we must use the Object type for the other argument in order to override the inherited method, and therefore, the code for the method has to use typecasting to cast the received instance to the SocialAnimal type.
```

首先，代码检查接收到的`Object`是否是对实际实例的引用。在这种情况下，代码返回`true`，不需要再进行其他检查。

然后，代码检查`other`的值是否等于`null`。如果方法接收到`null`，则代码返回`false`，因为实际实例不是`null`。

然后，代码检查实际实例的`getClass`方法返回的`String`是否与接收到的实例的相同方法返回的`String`匹配。如果这些值不匹配，则表示接收到的`Object`是不同类型的实例，因此不同，代码返回`false`。

此时，我们知道实际实例与接收到的实例具有相同的类型。因此，可以安全地将其他参数强制转换为`SocialAnimal`，并将转换后的引用保存在`SocialAnimal`类型的`otherSocialAnimal`局部变量中。

最后，代码返回评估当前实例和`otherSocialAnimal`的`getName`和`getAge`的`Object.equals`调用的结果是否都为`true`。

### 提示

当我们重写从`java.lang.Object`继承的`equals`方法时，遵循先前解释的步骤是一个好习惯。如果您有 C#的经验，重要的是要了解 Java 9 没有提供与`IEquatable<T>`接口等效的内容。此外，请注意，Java 不支持用户定义的运算符重载，这是其他面向对象编程语言（如 C++、C#和 Swift）中包含的功能。

`SocialAnimal`抽象类还实现了`Comparable<Sociable>`接口所需的`compareTo`方法。在这种情况下，代码非常简单，因为该方法在`otherSociable`参数中接收到一个`Sociable`实例，并返回调用`Integer.compare`方法的结果，即`java.lang.Integer`类的`compare`类方法。代码使用当前实例的`getAge`返回的`int`值和`otherSociable`作为两个参数调用此方法。`Integer.compare`方法返回以下结果：

+   如果第一个参数等于第二个参数，则为`0`。

+   如果第一个参数小于第二个参数，则小于`0`。

+   如果第一个参数大于第二个参数，则大于`0`。

所有继承自`SocialAnimal`的具体子类都将能够使用`SocialAnimal`抽象类中实现的`equals`和`compareTo`方法。

# 声明继承接口实现的子类

我们有一个抽象类`SocialAnimal`，它实现了`Sociable`和`Comparable<Sociable>`接口。我们不能创建这个抽象类的实例。现在，我们将创建`SocialAnimal`的一个具体子类，名为`SocialLion`。这个类声明了一个构造函数，最终调用了超类中定义的构造函数。该类实现了其超类中声明的四个抽象方法，以返回适合参加派对的狮子的适当值。示例的代码文件包含在`java_9_oop_chapter_10_01`文件夹中的`example10_01.java`文件中。

```java
public class SocialLion extends SocialAnimal {
 public SocialLion(String name, int age) {
        super(name, age);
    }

    @Override
 public String getDanceRepresentation() {
        return "*-* ^\\/^ (-)";
    }

    @Override
 public String getFirstSoundInWords() {
        return "Roar";
    }

    @Override
 public String getSecondSoundInWords() {
        return "Rrooaarr";
    }

    @Override
 public String getThirdSoundInWords() {
        return "Rrrrrrrroooooaaarrrr";
    }
}
```

我们将创建另一个名为`SocialParrot`的`SocialAnimal`的具体子类。这个新的子类也实现了`SocialAnimal`超类中定义的抽象方法，但在这种情况下，返回了鹦鹉的适当值。示例的代码文件包含在`java_9_oop_chapter_10_01`文件夹中的`example10_01.java`文件中。

```java
public class SocialParrot extends SocialAnimal {
    public SocialParrot(String name, int age) {
        super(name, age);
    }

    @Override
 public String getDanceRepresentation() {
        return "/|\\ -=- % % +=+";
    }

    @Override
 public String getFirstSoundInWords() {
        return "Yeah";
    }

    @Override
 public String getSecondSoundInWords() {
        return "Yeeaah";
    }

    @Override
 public String getThirdSoundInWords() {
        return "Yeeeaaaah";
    }
}
```

最后，我们将创建另一个名为`SocialSwan`的`SocialAnimal`的具体子类。这个新的子类也实现了`SocialAnimal`超类中定义的抽象方法，但在这种情况下，返回了天鹅的适当值。示例的代码文件包含在`java_9_oop_chapter_10_01`文件夹中的`example10_01.java`文件中。

```java
public class SocialSwan extends SocialAnimal {
    public SocialSwan(String name, int age) {
        super(name, age);
    }

    @Override
 public String getDanceRepresentation() {
        return "^- ^- ^- -^ -^ -^";
    }

    @Override
 public String getFirstSoundInWords() {
        return "OO-OO-OO";
    }

    @Override
 public String getSecondSoundInWords() {
        return "WHO-HO WHO-HO";
    }

    @Override
 public String getThirdSoundInWords() {
        return "WHO-WHO WHO-WHO";
    }
}
```

我们有三个具体类，它们继承了两个接口的实现，这两个接口来自它们的抽象超类`SociableAnimal`。以下三个具体类都实现了`Sociable`和`Comparable<Sociable>`接口，并且它们可以使用继承的重写的`equals`方法来比较它们的实例：

+   `SocialLion`

+   `SocialParrot`

+   `SocialSwan`

# 创建异常类

我们将创建两个异常类，因为我们需要抛出 Java 9 平台中没有表示的异常类型。具体来说，我们将创建`java.lang.Exception`类的两个子类。

以下行声明了`InsufficientMembersException`类，它继承自`Exception`。当一个派对的成员数量不足以执行需要更多成员的操作时，我们将抛出这个异常。该类定义了一个不可变的`numberOfMembers`私有字段，类型为`int`，它在构造函数中初始化为接收到的值。此外，该类声明了一个`getNumberOfMembers`方法，返回这个字段的值。示例的代码文件包含在`java_9_oop_chapter_10_01`文件夹中的`example10_01.java`文件中。

```java
public class InsufficientMembersException extends Exception {
    private final int numberOfMembers;

    public InsufficientMembersException(int numberOfMembers) {
        this.numberOfMembers = numberOfMembers;
    }

    public int getNumberOfMembers() {
        return numberOfMembers;
    }
}
```

以下行声明了`CannotRemovePartyLeaderException`类，它继承自`Exception`。当一个方法试图从派对成员列表中移除当前的派对领袖时，我们将抛出这个异常。在这种情况下，我们只声明了一个继承自`Exception`的空类，因为我们不需要额外的功能，我们只需要新的类型。示例的代码文件包含在`java_9_oop_chapter_10_01`文件夹中的`example10_01.java`文件中。

```java
public class CannotRemovePartyLeaderException extends Exception {
}
```

# 声明一个与受限泛型类型一起工作的类

以下行声明了一个`Party`类，利用泛型来处理多种类型。 我们导入`java.util.concurrent.ThreadLocalRandom`，因为它是一个非常有用的类，可以轻松地在范围内生成伪随机数。 类名`Party`后面跟着一个小于号(`<`)，一个标识泛型类型参数的`T`，`extends`关键字，以及`T`泛型类型参数必须实现的接口名称`Sociable`，一个和号(`&`)，以及`T`泛型类型必须实现的另一个接口名称`Comparable<Sociable>`。 大于号(`>`)结束了包含在尖括号(`<>`)中的类型约束声明。 因此，`T`泛型类型参数必须是一个既实现`Sociable`接口又实现`Comparable<Sociable>`接口的类型。 以下代码突出显示了使用`T`泛型类型参数的行。 示例的代码文件包含在`java_9_oop_chapter_10_01`文件夹中的`example10_01.java`文件中。

```java
import java.util.concurrent.ThreadLocalRandom;

public class Party<T extends Sociable & Comparable<Sociable>> {
 protected final List<T> members;
 protected T partyLeader;

 public Party(T partyLeader) {
        this.partyLeader = partyLeader;
 members = new ArrayList<>();
        members.add(partyLeader);
    }

 public T getPartyLeader() {
        return partyLeader;
    }
 public void addMember(T newMember) {
        members.add(newMember);
        partyLeader.welcome(newMember);
    }

 public T removeMember(T memberToRemove) throws CannotRemovePartyLeaderException {
        if (memberToRemove.equals(partyLeader)) {
            throw new CannotRemovePartyLeaderException();
        }
        int memberIndex = members.indexOf(memberToRemove);
        if (memberIndex >= 0) {
            members.remove(memberToRemove);
            memberToRemove.sayGoodbyeTo(partyLeader);
            return memberToRemove;
        } else {
            return null;
        }
    }

    public void makeMembersAct() {
 for (T member : members) {
            member.actAlone();
        }
    }

    public void makeMembersDance() {
 for (T member : members) {
            member.danceAlone();
        }
    }

    public void makeMembersSingALyric(String lyric) {
 for (T member : members) {
            member.singALyric(lyric);
        }
    }

    public void declareNewPartyLeader() throws InsufficientMembersException {
        if (members.size() == 1) {
            throw new InsufficientMembersException(members.size());
        }
 T newPartyLeader = partyLeader;
        while (newPartyLeader.equals(partyLeader)) {
            int pseudoRandomIndex = 
                ThreadLocalRandom.current().nextInt(
                    0, 
                    members.size());
            newPartyLeader = members.get(pseudoRandomIndex);
        }
        partyLeader.speak(
            String.format("%s is our new party leader.", 
                newPartyLeader.getName()));
        newPartyLeader.danceWith(partyLeader);
        if (newPartyLeader.compareTo(partyLeader) < 0) {
            // The new party leader is younger
            newPartyLeader.danceAlone();
        }
        partyLeader = newPartyLeader;
    }
}
```

现在我们将分析许多代码片段，以了解包含在`Party<T>`类中的代码是如何工作的。 以下行开始了类体，声明了一个受保护的`List<T>`，即元素类型为`T`或实现`T`接口的元素列表。 `List`使用泛型来指定将被接受和添加到列表中的元素的类型。

```java
protected final List<T> members;
```

以下行声明了一个受保护的`partyLeader`字段，其类型为`T`：

```java
protected T partyLeader;
```

以下行声明了一个接收`partyLeader`参数的构造函数，其类型为`T`。 该参数指定了第一位党领导者，也是党的第一位成员，即添加到`membersList<T>`的第一个元素。 创建新的`ArrayList<T>`的代码利用了 Java 7 中引入的类型推断，Java 8 中改进，并在 Java 9 中保留。 我们指定`new ArrayList<>()`而不是`new` `ArrayList<T>()`，因为 Java 9 可以使用空类型参数集(`<>`)从上下文中推断出类型参数。 `members`受保护字段具有`List<T>`类型，因此，Java 的类型推断可以确定`T`是类型，并且`ArrayList<>()`意味着`ArrayList<T>()`。 最后一行将`partyLeader`添加到`members`列表中。

```java
public Party(T partyLeader) {
    this.partyLeader = partyLeader;
    members = new ArrayList<>();
    members.add(partyLeader);
}
```

### 提示

当我们使用空类型参数集调用泛型类的构造函数时，尖括号(`<>`)被称为**diamond**，并且该表示法称为**diamond notation**。

以下行声明了`getPartyLeader`方法，指定`T`作为返回类型。 该方法返回`partyLeader`。

```java
public T getPartyLeader() {
    return partyLeader;
}
```

以下行声明了`addMember`方法，该方法接收一个类型为`T`的`newMember`参数。 该代码将接收到的新成员添加到`members`列表中，并调用`partyLeader.sayWelcomeTo`方法，将`newMember`作为参数，使得党领导者欢迎新成员：

```java
public void addMember(T newMember) {
    members.add(newMember);
    partyLeader.welcome(newMember);
}
```

以下行声明了`removeMember`方法，该方法接收一个类型为`T`的`memberToRemove`参数，返回`T`，并且可能抛出`CannotRemovePartyLeaderException`异常。 方法参数后面的`throws`关键字，后跟异常名称，表示该方法可以抛出指定的异常。 代码检查要移除的成员是否与党领导者匹配，使用`equals`方法进行检查。 如果成员是党领导者，则该方法抛出`CannotRemovePartyLeaderException`异常。 代码检索列表中`memberToRemove`的索引，并在该成员是列表成员的情况下调用`members.remove`方法，参数为`memberToRemove`。 然后，代码调用成功移除成员的`sayGoodbyeTo`方法，参数为`partyLeader`。 这样，离开党的成员向党领导者道别。 如果成员被移除，则该方法返回被移除的成员。 否则，该方法返回`null`。

```java
public T removeMember(T memberToRemove) throws CannotRemovePartyLeaderException {
    if (memberToRemove.equals(partyLeader)) {
        throw new CannotRemovePartyLeaderException();
    }
    int memberIndex = members.indexOf(memberToRemove);
    if (memberIndex >= 0) {
        members.remove(memberToRemove);
        memberToRemove.sayGoodbyeTo(partyLeader);
        return memberToRemove;
    } else {
        return null;
    }
}
```

以下行声明了`makeMembersAct`方法，该方法调用`members`列表中每个成员的`actAlone`方法：

```java
public void makeMembersAct() {
    for (T member : members) {
        member.actAlone();
    }
}
```

### 注意

在接下来的章节中，我们将学习在 Java 9 中将面向对象编程与函数式编程相结合的其他编码方法，以执行列表中每个成员的操作。

以下行声明了`makeMembersDance`方法，该方法调用`members`列表中每个成员的`danceAlone`方法：

```java
public void makeMembersDance() {
    for (T member : members) {
        member.danceAlone();
    }
}
```

以下行声明了`makeMembersSingALyric`方法，该方法接收一个`lyricString`并调用`members`列表中每个成员的`singALyric`方法，参数为接收到的`lyric`：

```java
public void makeMembersSingALyric(String lyric) {
    for (T member : members) {
        member.singALyric(lyric);
    }
}
```

### 提示

请注意，方法没有标记为 final，因此，我们将能够在将来的子类中重写这些方法。

最后，以下行声明了`declareNewPartyLeader`方法，该方法可能会抛出`InsufficientMembersException`。与`removeMember`方法一样，方法参数后的`throws`关键字后跟着`InsufficientMembersException`表示该方法可能会抛出`InsufficientMembersException`异常。如果`members`列表中只有一个成员，代码将抛出`InsufficientMembersException`异常，并使用从`members.size()`返回的值创建继承自`Exception`的类的实例。请记住，此异常类使用此值初始化一个字段，调用此方法的代码将能够检索到不足的成员数量。如果至少有两个成员，代码将生成一个新的伪随机党领袖，与现有的不同。代码使用`ThreadLocalRandom.current().nextInt`生成一个伪随机的`int`范围内的数字。代码调用`speak`方法让现任领袖向其他党员解释他们有了新的党领袖。代码调用`danceWith`方法，让新领袖与前任党领袖一起跳舞。如果调用`newPartyLeader.compareTo`方法与前任党领袖作为参数返回小于`0`，则意味着新的党领袖比前任年轻，代码将调用`newPartyLeader.danceAlone`方法。最后，代码将新值设置为`partyLeader`字段。

```java
public void declareNewPartyLeader() throws InsufficientMembersException {
    if (members.size() == 1) {
        throw new InsufficientMembersException(members.size());
    }
    T newPartyLeader = partyLeader;
    while (newPartyLeader.equals(partyLeader)) {
        int pseudoRandomIndex = 
            ThreadLocalRandom.current().nextInt(
                0, 
                members.size());
        newPartyLeader = members.get(pseudoRandomIndex);
    }
    partyLeader.speak(
        String.format("%s is our new party leader.", 
            newPartyLeader.getName()));
    newPartyLeader.danceWith(partyLeader);
    if (newPartyLeader.compareTo(partyLeader) < 0) {
        // The new party leader is younger
        newPartyLeader.danceAlone();
    }
    partyLeader = newPartyLeader;
}
```

# 使用通用类处理多个兼容类型

我们可以通过将`T`通用类型参数替换为符合`Party<T>`类声明中指定的类型约束的任何类型名称来创建`Party<T>`类的实例。到目前为止，我们有三个实现了`Sociable`和`Comparable<Sociable>`接口的具体类：`SocialLion`、`SocialParrot`和`SocialSwan`。因此，我们可以使用`SocialLion`来创建`Party<SocialLion>`的实例，即`SocialLion`的派对。我们利用类型推断，并使用先前解释的菱形符号。这样，我们将创建一个狮子派对，而`Simba`是党领袖。示例的代码文件包含在`java_9_oop_chapter_10_01`文件夹中的`example10_01.java`文件中。

```java
SocialLion simba = new SocialLion("Simba", 10);
SocialLion mufasa = new SocialLion("Mufasa", 5);
SocialLion scar = new SocialLion("Scar", 9);
SocialLion nala = new SocialLion("Nala", 7);
Party<SocialLion> lionsParty = new Party<>(simba);
```

`lionsParty`实例将仅接受`SocialLion`实例，其中类定义使用名为`T`的通用类型参数。以下行通过调用`addMember`方法为狮子派对添加了先前创建的三个`SocialLion`实例。示例的代码文件包含在`java_9_oop_chapter_10_01`文件夹中的`example10_01.java`文件中。

```java
lionsParty.addMember(mufasa);
lionsParty.addMember(scar);
lionsParty.addMember(nala);
```

以下行调用`makeMembersAct`方法使所有狮子行动，调用`makeMembersDance`方法使所有狮子跳舞，使用`removeMember`方法删除不是派对领袖的成员，使用`declareNewPartyLeader`方法声明一个新领袖，最后调用`makeMembersSingALyric`方法使所有狮子唱歌。我们将在调用`removeMember`和`declareNewPartyLeader`之前添加`try`关键字，因为这些方法可能会抛出异常。在这种情况下，我们不检查`removeMember`返回的结果。示例的代码文件包含在`java_9_oop_chapter_10_01`文件夹中的`example10_01.java`文件中。

```java
lionsParty.makeMembersAct();
lionsParty.makeMembersDance();
try {
    lionsParty.removeMember(nala);
} catch (CannotRemovePartyLeaderException e) {
    System.out.println(
        "We cannot remove the party leader.");
}
try {
    lionsParty.declareNewPartyLeader();
} catch (InsufficientMembersException e) {
    System.out.println(
        String.format("We just have %s member",
            e.getNumberOfMembers()));
}
lionsParty.makeMembersSingALyric("Welcome to the jungle");
```

以下行显示了在 JShell 中运行前面的代码片段后的输出。但是，我们必须考虑到新派对领袖的伪随机选择，因此结果在每次执行时会有所不同：

```java
Simba welcomes Mufasa
Simba welcomes Scar
Simba welcomes Nala
Simba to be or not to be
Mufasa to be or not to be
Scar to be or not to be
Nala to be or not to be
Simba dances alone *-* ^\/^ (-)
Mufasa dances alone *-* ^\/^ (-)
Scar dances alone *-* ^\/^ (-)
Nala dances alone *-* ^\/^ (-)
Nala says goodbye to Simba RoarRrooaarrRrrrrrrroooooaaarrrr
Simba says: Scar is our new party leader. *-* ^\/^ (-)
Scar dances with Simba *-* ^\/^ (-)
Scar dances alone *-* ^\/^ (-)
Simba sings Welcome to the jungle Roar Rrooaarr Rrrrrrrroooooaaarrrr
Mufasa sings Welcome to the jungle Roar Rrooaarr Rrrrrrrroooooaaarrrr
Scar sings Welcome to the jungle Roar Rrooaarr Rrrrrrrroooooaaarrrr

```

我们可以使用`SocialParrot`创建`Party<SocialParrot>`的实例，即`SocialParrot`的`Party`。我们使用先前解释的菱形符号。这样，我们将创建一个鹦鹉派对，`Rio`是派对领袖。示例的代码文件包含在`java_9_oop_chapter_10_01`文件夹中的`example10_01.java`文件中。

```java
SocialParrot rio = new SocialParrot("Rio", 3);
SocialParrot thor = new SocialParrot("Thor", 6);
SocialParrot rambo = new SocialParrot("Rambo", 4);
SocialParrot woody = new SocialParrot("Woody", 5);
Party<SocialParrot> parrotsParty = new Party<>(rio);
```

`parrotsParty`实例将仅接受`SocialParrot`实例，用于类定义使用名为`T`的泛型类型参数的所有参数。以下行通过为每个实例调用`addMember`方法，将先前创建的三个`SocialParrot`实例添加到鹦鹉派对中。示例的代码文件包含在`java_9_oop_chapter_10_01`文件夹中的`example10_01.java`文件中。

```java
parrotsParty.addMember(thor);
parrotsParty.addMember(rambo);
parrotsParty.addMember(woody);
```

以下行调用`makeMembersDance`方法使所有鹦鹉跳舞，使用`removeMember`方法删除不是派对领袖的成员，使用`declareNewPartyLeader`方法声明一个新领袖，最后调用`makeMembersSingALyric`方法使所有鹦鹉唱歌。示例的代码文件包含在`java_9_oop_chapter_10_01`文件夹中的`example10_01.java`文件中。

```java
parrotsParty.makeMembersDance();
try {
    parrotsParty.removeMember(rambo);
} catch (CannotRemovePartyLeaderException e) {
    System.out.println(
        "We cannot remove the party leader.");
}
try {
    parrotsParty.declareNewPartyLeader();
} catch (InsufficientMembersException e) {
    System.out.println(
        String.format("We just have %s member",
            e.getNumberOfMembers()));
}
parrotsParty.makeMembersSingALyric("Fly like a bird");
```

以下行显示了在 JShell 中运行前面的代码片段后的输出。再次，我们必须考虑到新派对领袖的伪随机选择，因此结果在每次执行时会有所不同：

```java
Rio welcomes Thor
Rio welcomes Rambo
Rio welcomes Woody
Rio dances alone /|\ -=- % % +=+
Thor dances alone /|\ -=- % % +=+
Rambo dances alone /|\ -=- % % +=+
Woody dances alone /|\ -=- % % +=+
Rambo says goodbye to Rio YeahYeeaahYeeeaaaah
Rio says: Woody is our new party leader. /|\ -=- % % +=+
Woody dances with Rio /|\ -=- % % +=+
Rio sings Fly like a bird Yeah Yeeaah Yeeeaaaah
Thor sings Fly like a bird Yeah Yeeaah Yeeeaaaah
Woody sings Fly like a bird Yeah Yeeaah Yeeeaaaah

```

以下行将无法编译，因为我们使用了不兼容的类型。首先，我们尝试将`SocialParrot`实例`rio`添加到`Party<SocialLion>`的`lionsParty`。然后，我们尝试将`SocialLion`实例`simba`添加到`Party<SocialParrot>`的`parrotsParty`。这两行都将无法编译，并且 JShell 将显示一条消息，指示类型不兼容，它们无法转换为每个派对所需的必要类型。示例的代码文件包含在`java_9_oop_chapter_10_01`文件夹中的`example10_02.java`文件中。

```java
// The following lines won't compile
// and will generate errors in JShell
lionsParty.addMember(rio);
parrotsParty.addMember(simba);
```

以下屏幕截图显示了在我们尝试执行前面的行时 JShell 中显示的错误：

![使用泛型类处理多个兼容类型](img/00090.jpeg)

我们可以使用`SocialSwan`创建`Party<SocialSwan>`的实例，即`SocialSwan`的`Party`。这样，我们将创建一个天鹅派对，`Kevin`是派对领袖。示例的代码文件包含在`java_9_oop_chapter_10_01`文件夹中的`example10_03.java`文件中。

```java
SocialSwan kevin = new SocialSwan("Kevin", 3);
SocialSwan brandon = new SocialSwan("Brandon", 5);
SocialSwan nicholas = new SocialSwan("Nicholas", 6);
Party<SocialSwan> swansParty = new Party<>(kevin);
```

`swansParty`实例将仅接受`SocialSwan`实例，用于类定义使用名为`T`的泛型类型参数的所有参数。以下行通过为每个实例调用`addMember`方法，将先前创建的两个`SocialSwan`实例添加到天鹅派对中。示例的代码文件包含在`java_9_oop_chapter_10_01`文件夹中的`example10_03.java`文件中。

```java
swansParty.addMember(brandon);
swansParty.addMember(nicholas);
```

以下行调用`makeMembersDance`方法使所有鹦鹉跳舞，使用`removeMember`方法尝试移除党领袖，使用`declareNewPartyLeader`方法宣布新领袖，最后调用`makeMembersSingALyric`方法使所有天鹅唱歌。示例的代码文件包含在`java_9_oop_chapter_10_01`文件夹中的`example10_03.java`文件中。

```java
swansParty.makeMembersDance();
try {
    swansParty.removeMember(kevin);
} catch (CannotRemovePartyLeaderException e) {
    System.out.println(
        "We cannot remove the party leader.");
}
try {
    swansParty.declareNewPartyLeader();
} catch (InsufficientMembersException e) {
    System.out.println(
        String.format("We just have %s member",
            e.getNumberOfMembers()));
}
swansParty.makeMembersSingALyric("It will be our swan song");
```

以下行显示了在 JShell 中运行前面的代码片段后的输出。再次，我们必须考虑到新的党领袖是伪随机选择的，因此，结果在每次执行时都会有所不同：

```java
Kevin welcomes Brandon
Kevin welcomes Nicholas
Kevin dances alone ^- ^- ^- -^ -^ -^
Brandon dances alone ^- ^- ^- -^ -^ -^
Nicholas dances alone ^- ^- ^- -^ -^ -^
We cannot remove the party leader.
Kevin says: Brandon is our new party leader. ^- ^- ^- -^ -^ -^
Brandon dances with Kevin ^- ^- ^- -^ -^ -^
Kevin sings It will be our swan song OO-OO-OO WHO-HO WHO-HO WHO-WHO WHO-WHO
Brandon sings It will be our swan song OO-OO-OO WHO-HO WHO-HO WHO-WHO WHO-WHO
Nicholas sings It will be our swan song OO-OO-OO WHO-HO WHO-HO WHO-WHO WHO-WHO

```

# 测试你的知识

1.  `public class Party<T extends Sociable & Comparable<Sociable>>`行的意思是：

1.  泛型类型约束指定`T`必须实现`Sociable`或`Comparable<Sociable>`接口之一。

1.  泛型类型约束指定`T`必须实现`Sociable`和`Comparable<Sociable>`接口。

1.  该类是`Sociable`和`Comparable<Sociable>`类的子类。

1.  以下哪行与 Java 9 中的`List<SocialLion> lionsList = new ArrayList<SocialLion>();`等效：

1.  `List<SocialLion> lionsList = new ArrayList();`

1.  `List<SocialLion> lionsList = new ArrayList<>();`

1.  `var lionsList = new ArrayList<SocialLion>();`

1.  以下哪行使用了钻石符号来利用 Java 9 的类型推断：

1.  `List<SocialLion> lionsList = new ArrayList<>();`

1.  `List<SocialLion> lionsList = new ArrayList();`

1.  `var lionsList = new ArrayList<SocialLion>();`

1.  Java 9 允许我们通过以下方式使用参数多态性：

1.  鸭子打字。

1.  兔子打字。

1.  泛型。

1.  以下哪个代码片段声明了一个类，其泛型类型约束指定`T`必须实现`Sociable`和`Convertible`接口：

1.  `public class Game<T extends Sociable & Convertible>`

1.  `public class Game<T: where T is Sociable & Convertible>`

1.  `public class Game<T extends Sociable> where T: Convertible`

# 总结

在本章中，您学会了通过编写能够与不同类型的对象一起工作的代码来最大化代码重用，也就是说，能够实现特定接口的类的实例或其类层次结构包括特定超类的类的实例。我们使用了接口、泛型和受限泛型类型。

我们创建了能够使用受限泛型类型的类。我们结合了类继承和接口，以最大化代码的可重用性。我们可以使类与许多不同类型一起工作，我们能够编写一个能够被重用来创建狮子、鹦鹉和天鹅的派对行为的类。

现在您已经学会了关于参数多态性和泛型的基础知识，我们准备在 Java 9 中与泛型最大化代码重用的更高级场景一起工作，这是我们将在下一章讨论的主题。
