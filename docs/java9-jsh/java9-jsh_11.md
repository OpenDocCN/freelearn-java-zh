# 第十一章：高级泛型

在本章中，我们将深入探讨参数多态性以及 Java 9 如何允许我们编写使用两个受限泛型类型的类的通用代码。我们将：

+   在更高级的场景中使用参数多态性

+   创建一个新接口，用作第二个类型参数的约束

+   声明两个实现接口以处理两个类型参数的类

+   声明一个使用两个受限泛型类型的类

+   使用具有两个泛型类型参数的泛型类

# 创建一个新接口，用作第二个类型参数的约束

到目前为止，我们一直在处理派对，其中派对成员是善于社交的动物。然而，没有一些音乐很难享受派对。善于社交的动物需要听到一些东西，以便让它们跳舞并享受他们的派对。我们想要创建一个由善于社交的动物和一些可听到的东西组成的派对。

现在，我们将创建一个新的接口，稍后在定义另一个利用两个受限泛型类型的类时将使用该接口作为约束。以下是`Hearable`接口的代码。

示例的代码文件包含在`java_9_oop_chapter_11_01`文件夹中的`example11_01.java`文件中。

```java
public interface Hearable {
    void playMusic();
    void playMusicWithLyrics(String lyrics);
}
```

接口声明了两个方法要求：`playMusic`和`playMusicWithLyrics`。正如我们在之前的章节中学到的，接口只包括方法声明，因为实现`Hearable`接口的类将负责提供这两个方法的实现。

# 声明两个实现接口以处理两个类型参数

现在，我们将声明一个名为`Smartphone`的类，该类实现先前定义的`Hearable`接口。我们可以将类声明解读为“`Smartphone`类实现`Hearable`接口”。以下是新类的代码。

```java
public class Smartphone implements Hearable {
    public final String modelName;

    public Smartphone(String modelName) {
        this.modelName = modelName;
    }

    @Override
    public void playMusic() {
        System.out.println(
            String.format("%s starts playing music.",
                modelName));
        System.out.println(
            String.format("cha-cha-cha untz untz untz",
                modelName));
    }

    @Override
    public void playMusicWithLyrics(String lyrics) {
        System.out.println(
            String.format("%s starts playing music with lyrics.",
                modelName));
        System.out.println(
            String.format("untz untz untz %s untz untz",
                lyrics));
    }
}
```

`Smartphone`类声明了一个构造函数，将必需的`modelName`参数的值分配给`modelName`不可变字段。此外，该类实现了`Hearable`接口所需的两个方法：`playMusic`和`playMusicWithLyrics`。

`playMusic`方法打印一条消息，显示智能手机型号名称，并指示设备开始播放音乐。然后，该方法以文字形式打印多个声音。`playMusicWithLyrics`方法打印一条消息，显示智能手机型号名称，然后是另一条包含文字声音和作为参数接收的歌词的消息。

现在，我们将声明一个名为`AnimalMusicBand`的类，该类也实现了先前定义的`Hearable`接口。我们可以将类声明解读为“`AnimalMusicBand`类实现`Hearable`接口”。以下是新类的代码。示例的代码文件包含在`java_9_oop_chapter_11_01`文件夹中的`example11_01.java`文件中。

```java
public class AnimalMusicBand implements Hearable {
    public final String bandName;
    public final int numberOfMembers;

    public AnimalMusicBand(String bandName, int numberOfMembers) {
        this.bandName = bandName;
        this.numberOfMembers = numberOfMembers;
    }

    @Override
    public void playMusic() {
        System.out.println(
            String.format("Our name is %s. We are %d.",
                bandName,
                numberOfMembers));
        System.out.println(
            String.format("Meow Meow Woof Woof Meow Meow",
                bandName));
    }

    @Override
    public void playMusicWithLyrics(String lyrics) {
        System.out.println(
            String.format("%s asks you to sing together.",
                bandName));
        System.out.println(
            String.format("Meow Woof %s Woof Meow",
                lyrics));
    }
}
```

`AnimalMusicBand`类声明了一个构造函数，将必需的`bandName`和`numberOfMembers`参数的值分配给与这些参数同名的不可变字段。此外，该类实现了`Hearable`接口所需的两个方法：`playMusic`和`playMusicWithLyrics`。

`playMusic`方法打印一条消息，向观众介绍动物音乐乐队，并指出成员数量。然后，该方法以文字形式打印多个声音。`playMusicWithLyrics`方法打印一条消息，要求观众与动物音乐乐队一起唱歌，然后是另一条带有文字和作为参数接收的歌词的消息。

# 声明一个与两个受限泛型类型一起工作的类

以下行声明了一个`PartyWithHearable`子类，该子类是先前创建的`Party<T>`类的子类，利用泛型来处理两个受限类型。类型约束声明包含在尖括号(`<>`)中。在这种情况下，我们有两个泛型类型参数：`T`和`U`。名为`T`的泛型类型参数必须实现`Sociable`和`Comparable<Sociable>`接口，就像在`Party<T>`超类中一样。名为`U`的泛型类型参数必须实现`Hearable`接口。请注意，跟随类型参数的`extends`关键字允许我们向泛型类型参数添加约束，尖括号之后的相同关键字指定该类继承自`Party<T>`超类。这样，该类为`T`和`U`泛型类型参数指定了约束，并且继承自`Party<T>`。示例的代码文件包含在`java_9_oop_chapter_11_01`文件夹中的`example11_01.java`文件中。

```java
public class PartyWithHearable<T extends Sociable & Comparable<Sociable>, U extends Hearable> extends Party<T> {
 protected final U soundGenerator;

 public PartyWithHearable(T partyLeader, U soundGenerator) {
        super(partyLeader);
 this.soundGenerator = soundGenerator;
    }

    @Override
    public void makeMembersDance() {
 soundGenerator.playMusic();
        super.makeMembersDance();
    }

    @Override
    public void makeMembersSingALyric(String lyric) {
 soundGenerator.playMusicWithLyrics(lyric);
        super.makeMembersSingALyric(lyric);
    }
}
```

### 提示

当 Java 中的类型参数有约束时，它们也被称为**有界类型参数**。此外，类型约束也被称为有界类型参数的**上界**，因为任何实现用作上界的接口或任何指定为上界的类的子类都可以用于类型参数。

现在我们将分析许多代码片段，以了解包含在`PartyWithHearable<T, U>`类中的代码是如何工作的。以下行开始类体并声明了一个受保护的不可变的`soundGenerator`字段，其类型由`U`指定：

```java
protected final U soundGenerator;
```

以下行声明了一个初始化器，该初始化器接收两个参数，`partyLeader`和`soundGenerator`，它们的类型分别为`T`和`U`。这些参数指定了将成为派对第一领导者并成为派对第一成员的第一领导者，以及将使派对成员跳舞和唱歌的声音发生器。构造函数使用`super`关键字调用其超类中定义的构造函数，并将`partyLeader`作为参数。

```java
public PartyWithHearable(T partyLeader, U soundGenerator) {
    super(partyLeader);
    this.soundGenerator = soundGenerator;
}
```

以下行声明了一个`makeMembersDance`方法，该方法覆盖了在超类中包含的相同声明的方法。代码调用`soundGenetor.playMusic`方法，然后使用`super`关键字调用`super.makeMembersDance`方法，即在`Party<T>`超类中定义的`makeMembersDance`方法：

```java
@Override
public void makeMembersDance() {
    soundGenerator.playMusic();
    super.makeMembersDance();
}
```

### 注意

当我们在子类中覆盖一个方法时，我们可以使用`super`关键字后跟一个点(`.`)和方法名来调用在超类中定义的方法，并将所需的参数传递给该方法。使用`super`关键字允许我们调用在超类中被覆盖的实例方法。这样，我们可以向方法添加新特性，同时仍然调用基本方法。

最后，以下行声明了一个`makeMembersSingALyric`方法，该方法覆盖了在超类中包含的相同声明的方法。代码调用`soundGenerator.playMusicWithLyrics`方法，并将接收到的`lyrics`作为参数。然后，代码调用`super.makeMembersSingALyric`方法，并将接收到的`lyrics`作为参数，即在`Party<T>`超类中定义的`makeMembersSingALyric`方法：

```java
@Override
public void makeMembersSingALyric(String lyric) {
    soundGenerator.playMusicWithLyrics(lyric);
    super.makeMembersSingALyric(lyric);
}
```

以下 UML 图显示了我们将创建的接口和具体子类，包括所有字段和方法。

![声明一个与两个受限泛型类型一起工作的类](img/00091.jpeg)

# 使用两个泛型类型参数创建泛型类的实例

我们可以通过用符合`PartyWithHearable<T, U>`类声明中指定的约束或上界的任何类型名称替换`T`和`U`泛型类型参数来创建`PartyWithHearable<T, U>`类的实例。我们有三个具体类实现了`T`泛型类型参数所需的`Sociable`和`Comparable<Sociable>`接口：`SocialLion`、`SocialParrot`和`SocialSwan`。我们有两个实现了`U`泛型类型参数所需的`Hearable`接口的类：`Smartphone`和`AnimalMusicBand`。

我们可以使用`SocialLion`和`Smartphone`来创建`PartyWithHearable<SocialLion, Smartphone>`的实例，即社交狮和智能手机的聚会。然后，我们可以使用`SocialParrot`和`AnimalMusicBand`来创建`PartyWithHearable<SocialParrot, AnimalMusicBand>`的实例，即社交鹦鹉和动物音乐乐队的聚会。

以下行创建了一个名为`android`的`Smartphone`实例。然后，代码创建了一个名为`nalaParty`的`PartyWithHearable<SocialLion, Smartphone>`实例，并将`nala`和`android`作为参数传递。我们利用了类型推断，并使用了我们在上一章学到的菱形符号表示法，第十章, *泛型的代码重用最大化*。这样，我们创建了一个使用智能手机的社交狮聚会，其中`Nala`是聚会领袖，`Super Android Smartphone`是可听或音乐生成器。示例的代码文件包含在`java_9_oop_chapter_11_01`文件夹中的`example11_01.java`文件中。

```java
Smartphone android = new Smartphone("Super Android Smartphone");
PartyWithHearable<SocialLion, Smartphone> nalaParty = 
    new PartyWithHearable<>(nala, android);
```

`nalaParty`实例将只接受`SocialLion`实例，用于类定义中使用泛型类型参数`T`的所有参数。`nalaParty`实例将只接受`Smartphone`实例，用于类定义中使用泛型类型参数`U`的所有参数。以下行通过调用`addMember`方法将之前创建的三个`SocialLion`实例添加到聚会中。示例的代码文件包含在`java_9_oop_chapter_11_01`文件夹中的`example11_01.java`文件中。

```java
nalaParty.addMember(simba);
nalaParty.addMember(mufasa);
nalaParty.addMember(scar);
```

以下屏幕截图显示了在 JShell 中执行上述代码的结果：

![创建具有两个泛型类型参数的泛型类的实例](img/00092.jpeg)

以下行调用`makeMembersDance`方法，使智能手机的播放列表邀请所有狮子跳舞并使它们跳舞。然后，代码调用`removeMember`方法来移除不是聚会领袖的成员，使用`declareNewPartyLeader`方法来声明一个新的领袖，最后调用`makeMembersSingALyric`方法来使智能手机的播放列表邀请所有狮子唱特定的歌词并使他们唱这个歌词。请记住，在调用`removeMember`和`declareNewPartyLeader`之前，我们在这些方法前加上`try`关键字，因为这些方法可能会抛出异常。示例的代码文件包含在`java_9_oop_chapter_11_01`文件夹中的`example11_01.java`文件中。

```java
nalaParty.makeMembersDance();
try {
    nalaParty.removeMember(mufasa);
} catch (CannotRemovePartyLeaderException e) {
    System.out.println(
        "We cannot remove the party leader.");
}
try {
    nalaParty.declareNewPartyLeader();
} catch (InsufficientMembersException e) {
    System.out.println(
        String.format("We just have %s member",
            e.getNumberOfMembers()));
}
nalaParty.makeMembersSingALyric("It's the eye of the tiger");
```

以下屏幕截图显示了在 JShell 中执行上述代码的结果：

![创建具有两个泛型类型参数的泛型类的实例](img/00093.jpeg)

以下行显示了在 JShell 中运行前面代码片段后的输出。但是，我们必须考虑到新聚会领袖的伪随机选择，因此结果在每次执行时会有所不同：

```java
Nala welcomes Simba
Nala welcomes Mufasa
Nala welcomes Scar
Super Android Smartphone starts playing music.
cha-cha-cha untz untz untz
Nala dances alone *-* ^\/^ (-)
Simba dances alone *-* ^\/^ (-)
Mufasa dances alone *-* ^\/^ (-)
Scar dances alone *-* ^\/^ (-)
Mufasa says goodbye to Nala RoarRrooaarrRrrrrrrroooooaaarrrr
Nala says: Simba is our new party leader. *-* ^\/^ (-)
Simba dances with Nala *-* ^\/^ (-)
Super Android Smartphone starts playing music with lyrics.
untz untz untz It's the eye of the tiger untz untz
Nala sings It's the eye of the tiger Roar Rrooaarr Rrrrrrrroooooaaarrrr
Simba sings It's the eye of the tiger Roar Rrooaarr Rrrrrrrroooooaaarrrr
Scar sings It's the eye of the tiger Roar Rrooaarr Rrrrrrrroooooaaarrrr

```

以下行创建了一个名为`band`的`AnimalMusicBand`实例。然后，代码创建了一个名为`ramboParty`的`PartyWithHearable<SocialParrot, AnimalMusicBand>`实例，并将`rambo`和`band`作为参数传递。与之前的示例一样，我们利用了类型推断，并且使用了我们在上一章学习的菱形符号，第十章, *泛型的代码重用最大化*。这样，我们创建了一个由四只动物组成的音乐乐队的社交鹦鹉派对，其中`Rambo`是派对领袖，`Black Eyed Paws`是可听到的或音乐发生器。示例的代码文件包含在`java_9_oop_chapter_11_01`文件夹中的`example11_02.java`文件中。

```java
AnimalMusicBand band = new AnimalMusicBand(
    "Black Eyed Paws", 4);
PartyWithHearable<SocialParrot, AnimalMusicBand> ramboParty = 
    new PartyWithHearable<>(rambo, band);
```

`ramboParty`实例只接受`SocialParrot`实例作为类定义中使用泛型类型参数`T`的所有参数。`ramboParty`实例只接受`AnimalMusicBand`实例作为类定义中使用泛型类型参数`U`的所有参数。以下行通过调用`addMember`方法将之前创建的三个`SocialParrot`实例添加到派对中。示例的代码文件包含在`java_9_oop_chapter_11_01`文件夹中的`example11_02.java`文件中。

```java
ramboParty.addMember(rio);
ramboParty.addMember(woody);
ramboParty.addMember(thor);
```

以下截图显示了在 JShell 中执行上一个代码的结果。

![使用两个泛型类型参数创建泛型类的实例](img/00094.jpeg)

以下行调用`makeMembersDance`方法，使动物音乐乐队邀请所有鹦鹉跳舞，告诉它们乐队中有四名成员并让它们跳舞。然后，代码调用`removeMember`方法来移除不是派对领袖的成员，使用`declareNewPartyLeader`方法来声明一个新的领袖，最后调用`makeMembersSingALyric`方法，使动物音乐乐队邀请所有鹦鹉唱特定的歌词并让它们唱这个歌词。请记住，在调用`removeMember`和`declareNewPartyLeader`之前我们加上了`try`关键字，因为这些方法可能会抛出异常。示例的代码文件包含在`java_9_oop_chapter_11_01`文件夹中的`example11_02.java`文件中。

```java
ramboParty.makeMembersDance();
try {
    ramboParty.removeMember(rio);
} catch (CannotRemovePartyLeaderException e) {
    System.out.println(
        "We cannot remove the party leader.");
}
try {
    ramboParty.declareNewPartyLeader();
} catch (InsufficientMembersException e) {
    System.out.println(
        String.format("We just have %s member",
            e.getNumberOfMembers()));
}
ramboParty.makeMembersSingALyric("Turn up the radio");
```

以下截图显示了在 JShell 中执行上一个代码的结果：

![使用两个泛型类型参数创建泛型类的实例](img/00095.jpeg)

以下行显示了在 JShell 中运行前面的代码片段后的输出。但是，我们必须考虑到新派对领袖的伪随机选择，因此结果在每次执行时会有所不同：

```java
Rambo welcomes Rio
Rambo welcomes Woody
Rambo welcomes Thor
Our name is Black Eyed Paws. We are 4.
Meow Meow Woof Woof Meow Meow
Rambo dances alone /|\ -=- % % +=+
Rio dances alone /|\ -=- % % +=+
Woody dances alone /|\ -=- % % +=+
Thor dances alone /|\ -=- % % +=+
Rio says goodbye to Rambo YeahYeeaahYeeeaaaah
Rambo says: Thor is our new party leader. /|\ -=- % % +=+
Thor dances with Rambo /|\ -=- % % +=+
Black Eyed Paws asks you to sing together.
Meow Woof Turn up the radio Woof Meow
Rambo sings Turn up the radio Yeah Yeeaah Yeeeaaaah
Woody sings Turn up the radio Yeah Yeeaah Yeeeaaaah
Thor sings Turn up the radio Yeah Yeeaah Yeeeaaaah

```

# 测试你的知识

1.  `PartyWithHearable<T extends Sociable & Comparable<Sociable>, U extends Hearable>`这一行的意思是：

1.  泛型类型约束指定`T`必须实现`Sociable`或`Comparable<Sociable>`接口，`U`必须实现`Hearable`接口。

1.  该类是`Sociable`、`Comparable<Sociable>`和`Hearable`类的子类。

1.  泛型类型约束指定`T`必须实现`Sociable`和`Comparable<Sociable>`接口，`U`必须实现`Hearable`接口。

1.  以下哪一行等同于 Java 9 中的`PartyWithHearable<SocialLion, Smartphone>lionsParty = new PartyWithHearable<SocialLion, Smartphone>(nala, android);`：

1.  `PartyWithHearable<SocialLion, Smartphone> lionsParty = new PartyWithHearable<>(nala, android);`

1.  `PartyWithHearable<SocialLion, Smartphone> lionsParty = new PartyWithHearable(nala, android);`

1.  `let lionsParty = new PartyWithHearable(nala, android);`

1.  当我们在使用`extends`关键字的有界类型参数时：

1.  实现指定为上界的接口的任何类都可以用于类型参数。如果指定的名称是一个类的名称，则其子类不能用于类型参数。

1.  实现指定为上界的接口或指定为上界的类的任何子类都可以用于类型参数。

1.  指定为上界的类的任何子类都可以用于类型参数。如果指定的名称是一个接口的名称，则实现该接口的类不能用于类型参数。

1.  当 Java 中的类型参数具有约束时，它们也被称为：

1.  灵活的类型参数。

1.  无界类型参数。

1.  有界类型参数。

1.  以下哪个代码片段声明了一个类，其泛型类型约束指定`T`必须实现`Sociable`接口，`U`必须实现`Convertible`接口：

1.  `public class Game<T: where T is Sociable, U: where U is Convertible>`

1.  `public class Game<T extends Sociable> where U: Convertible`

1.  `public class Game<T extends Sociable, U extends Convertible>`

# 摘要

在本章中，您学习了通过编写能够与两个类型参数一起工作的代码来最大化代码重用。我们处理了涉及接口、泛型和具有约束的多个类型参数的更复杂的情况，也称为有界类型参数。

我们创建了一个新接口，然后声明了两个实现了这个新接口的类。然后，我们声明了一个使用了两个受限泛型类型参数的类。我们结合了类继承和接口，以最大化代码的可重用性。我们可以使类与许多不同类型一起工作，并且能够编写具有不同音乐生成器的派对的行为，然后可以重用这些行为来创建带有智能手机的狮子派对和带有动物乐队的鹦鹉派对。

Java 9 允许我们处理更复杂的情况，在这些情况下，我们可以为泛型类型参数指定更多的限制或边界。然而，大多数情况下，我们将处理本章和上一章中学到的示例所涵盖的情况。

现在您已经学会了参数多态性和泛型的高级用法，我们准备在 Java 9 中将面向对象编程和函数式编程相结合，这是我们将在下一章中讨论的主题。
