# 第三章。深入了解 Lambda

在本节中，我们将更详细地看一些相关主题，比如：

+   函数接口

+   方法和构造函数引用

+   范围和有效的最终变量

+   异常透明度

+   Lambda 和闭包之间的区别

+   正如我们谈到的，lambda 不仅仅是语法糖，我们将看一下 lambda 生成的字节码

# 函数接口

Java 将 lambda 视为接口类型的实例。它将这个形式化为它所谓的*函数接口*。函数接口只是一个具有单一方法的接口。Java 将这个方法称为"函数方法"，但通常使用"单一抽象方法"或 SAM。

JDK 中所有现有的单一方法接口，如`Runnable`和`Callable`，现在都是函数接口，lambda 可以在任何使用单一抽象方法接口的地方使用。事实上，正是函数接口允许了所谓的*目标类型*；它们提供了足够的信息，以便编译器推断参数和返回类型。

## @FunctionalInterface

Oracle 引入了一个新的注解`@FunctionalInterface`来标记一个接口。这基本上是为了传达意图，但也允许编译器进行一些额外的检查。

例如，这个接口可以编译通过：

```java
public interface FunctionalInterfaceExample {
    // compiles ok
}
```

但当你通过添加新的注解来指示它应该是一个*函数接口*时，

```java
@FunctionalInterface // <- error here
    public interface FunctionalInterfaceExample {
      // doesn't compile
}
```

编译器会报错。它告诉我们"Example 不是一个函数接口"，因为"没有找到抽象方法"。通常 IDE 也会提示，[IntelliJ](http://www.jetbrains.com/idea/)会说类似"没有找到目标方法"。它在暗示我们忘了函数方法。"单一抽象方法"需要一个`abstract`方法。

那么如果我们尝试向接口添加第二个方法会怎样呢？

```java
@FunctionalInterface
public interface FunctionalInterfaceExample {
    void apply();
    void illegal(); // <- error here
}
```

编译器会再次报错，这次的错误信息是"找到多个非覆盖的抽象方法"。函数接口只能有**一个**方法。

## 扩展

那么接口扩展另一个接口的情况呢？

让我们创建一个名为`A`的新函数接口，另一个名为`B`的函数接口，它扩展了`A`。`B`接口仍然是一个函数接口。它继承了父类的`apply`方法，正如你所期望的那样：

```java
@FunctionalInterface
interface A {
    abstract void apply();
}

interface B extends A {
}
```

如果你想要更清晰地表达这一点，你也可以覆盖父类的函数方法：

```java
@FunctionalInterface
interface A {
    abstract void apply();
}

interface B extends A {
    @Override
    abstract void apply();
}
```

如果我们将它作为 lambda 使用，我们可以验证它作为函数接口是否有效。我们将实现一个方法来展示 lambda 可以分配给类型 A 和类型 B。实现只是打印出`"A"`或`"B"`。

```java
@FunctionalInterface
public interface A {
    void apply();
}

public interface B extends A {
    @Override
    void apply();
}

public static void main(String... args) {
   A a = () -> System.out.println("A");
   B b = () -> System.out.println("B");
}
```

你不能向扩展接口（B）添加新的`abstract`方法，因为结果类型会有两个`abstract`方法，IDE 会警告我们，编译器会报错。

```java
@FunctionalInterface
public interface A {
    void apply();
}

public interface B extends A {
    void illegal();     // <- can't do this
}

public static void main(String... args) {
    A a = () -> System.out.println("A");
    B b = () -> System.out.println("B");    // <- error
}
```

在这两种情况下，你可以覆盖`Object`中的方法而不会引起问题。你也可以添加默认方法（从 Java 8 开始的新功能）。正如你可能期望的那样，将抽象类标记为函数接口是没有意义的。

## 其他接口改进

接口通常有一些新的特性，它们包括：

+   默认方法（虚拟扩展方法）

+   静态接口方法

+   以及`java.util.function`包中一堆新的函数接口；像`Function`和`Predicate`这样的东西

## 摘要

在本节中，我们谈到了任何具有单一方法的接口现在都是"函数接口"，而且这个单一方法通常被称为"函数方法"或 SAM（单一抽象方法）。

我们看了一下新的注解，并看了一些现有 JDK 接口如`Runnable`和`Callable`是如何使用这个注解进行改造的。

我们还介绍了*目标类型*的概念，即编译器如何使用函数方法的签名来帮助确定可以在哪里使用 lambda 表达式。我们稍微提了一下这个概念，因为我们将在类型推断部分稍后讨论它。

我们讨论了一些函数接口的示例，编译器和 IDE 在我们犯错时如何帮助我们，并对我们可能遇到的错误有了一些了解。例如，向函数接口添加多个方法。我们还看到了这个规则的例外情况，即当我们重写`Object`的方法或实现默认方法时。

我们快速查看了接口继承以及它对事物的影响，并提到了我们稍后将要涵盖的其他接口改进。

这里要记住的一个重要观点是，任何地方都可以使用函数接口，现在都可以使用 lambda。Lambda 可以用来代替函数接口的匿名实现。使用 lambda 而不是匿名类可能看起来像语法糖，但它们实际上是有很大不同的。有关更多详细信息，请参阅*函数 vs. 类*部分。

# 类型推断改进

现代 Java 中有几个类型推断改进。为了支持 lambda，编译器推断的方式已经得到改进，广泛使用*目标类型*。这些改进和 Java 7 的推断改进都是在开放**JDK 增强提案**（**JEP**）101 下进行管理的。

在我们深入讨论这些之前，让我们回顾一下基础知识。

类型推断是指编程语言自动推断表达式类型的能力。

静态类型语言在*编译*时知道事物的类型。动态类型语言在*运行时*知道类型。静态类型语言可以使用类型推断，在源代码中删除类型信息，并使用编译器来找出缺失的部分。

类型推断改进

这意味着静态类型语言（如 Scala）可以使用类型推断来“看起来”像动态语言（如 JavaScript）。至少在源代码级别上是这样。

以下是 Scala 中的一行代码示例：

```java
val name = "Henry"
```

你不需要明确告诉编译器该值是一个字符串。它会自己推断出来。你可以像这样明确地写出来，

```java
val name : String = "Henry"
```

但没有必要。

顺便说一句，Scala 还尝试根据其**抽象语法树**（**AST**）来确定何时完成语句或表达式。因此，通常情况下，你甚至不需要添加终止分号。

## Java 类型推断

类型推断是一个相当广泛的主题，Java 不支持我刚刚谈到的类型推断类型。至少对于像删除变量的类型注释这样的事情。我们必须记住：

```java
String name = "Henry"; // <- we can't drop the String like Scala
```

因此，Java 在更广义上不支持类型推断。它不能像某些语言那样猜测*一切*。因此，Java 中的类型推断通常指的是编译器如何为泛型推断类型。Java 7 在引入了菱形操作符（`<>`）时改进了这一点，但在 Java *可以*推断出的内容上仍然有很多限制。

Java 编译器是基于类型擦除构建的；它在编译过程中主动删除类型信息。由于类型擦除，`List<String>`在编译后变成了`List<Object>`。

出于历史原因，在 Java 5 中引入泛型时，开发人员无法轻易地逆转使用擦除的决定。Java 需要理解要替换给定泛型类型的类型，但没有信息如何做到这一点，因为所有信息都已被擦除。类型推断是解决方案。

所有泛型值在后台实际上都是`Object`类型，但通过使用类型推断，编译器可以检查所有源代码用法是否与它认为的泛型一致。在运行时，一切都将作为`Object`的实例传递，并在后台进行适当的转换。类型推断只是允许编译器提前检查转换是否有效。

因此，类型推断是关于猜测类型的，Java 对类型推断的支持在 Java 8 中将以几种方式得到改进：

1.  lambda 的目标类型推断。

并使用泛化的目标类型推断：

1.  在方法调用中添加对参数类型推断的支持。

1.  在链式调用中添加对参数类型推断的支持。

让我们看看当前的问题以及现代 Java 如何解决它们。

## 用于 lambda 的目标类型推断

现代 Java 对类型推断的一般改进意味着 lambda 可以推断它们的类型参数；所以不再需要使用：

```java
(Integer x, Integer y) -> x + y;
```

你可以去掉`Integer`类型注释，使用以下代替：

```java
(x, y) -> x + y;
```

这是因为函数接口描述了类型，它给编译器提供了所有需要的信息。

例如，如果我们拿一个例子函数接口。

```java
@FunctionalInterface
interface Calculation {
    Integer apply(Integer x, Integer y);
}
```

当 lambda 被用来代替接口时，编译器首先要做的是确定 lambda 的*目标*类型。所以如果我们创建一个接受接口和两个整数的方法`calculate`。

```java
static Integer calculate(Calculation operation, Integer x, Integer y) { 
    return operation.apply(x, y);
}
```

然后创建两个 lambda；一个加法和减法函数

```java
Calculation addition = (x, y) -> x + y;
Calculation subtraction = (x, y) -> x - y;
```

并且像这样使用它们：

```java
calculate(addition, 2, 2);
calculate(subtraction, 5, calculate(addition, 3, 2));
```

编译器理解 lambda 的加法和减法具有 Calculation 的目标类型（它是唯一符合 calculate 方法签名的“形状”）。然后它可以使用方法签名来推断 lambda 参数的类型。接口上只有一个方法，所以没有歧义，参数类型显然是`Integer`。

我们将看很多目标类型推断的例子，所以现在，只需知道 Java 用来实现很多 lambda 功能的机制依赖于类型推断的改进和*目标*类型的概念。

### 方法调用中的类型参数

在 Java 8 之前，有一些情况下编译器无法推断类型。其中之一是在调用带有泛型类型参数的方法时。

例如，`Collections`类有一个用于生成空列表的泛型方法。它看起来像这样：

```java
public static final <T> List<T> emptyList() { ... }
```

在 Java 7 中，这个编译：

```java
List<String> names = Collections.emptyList(); // compiles in Java 7
```

因为 Java 7 编译器可以推断出`emptyList`方法所需的泛型类型是`String`类型。然而，它在泛型方法的结果作为参数传递给另一个方法调用时会有困难。

所以如果我们有一个处理列表的方法看起来像这样：

```java
static void processNames(List<String> names) {
    System.out.println("hello " + name);
}
```

然后用空列表方法调用它。

```java
processNames(Collections.emptyList()); // doesn't compile in Java 7
```

它不会编译，因为参数的泛型类型已被擦除为`Object`。它实际上看起来像这样。

```java
processNames(Collections.<Object>emptyList names);
```

这与`processList`方法不匹配。

```java
processNames(Collections.<String>emptyList names);
```

所以在我们使用显式的“类型见证”给它一个额外的提示之前，它不会编译。

```java
processNames(Collections.<String>emptyList());   // compiles in Java 7
```

现在编译器已经足够了解传递到方法中的泛型类型。

Java 8 中的改进包括更好地支持这一点，因此一般来说，你不再需要类型见证。

我们现在调用`processNames`的例子编译通过了！

```java
processNames(Collections.emptyList());           // compiles in Java 8
```

### 链式方法调用中的类型参数

类型推断的另一个常见问题是方法链。假设我们有一个`List`类：

```java
static class List<E> {

    static <T> List<T> emptyList() {
        return new List<T>();
    }

    List<E> add(E e) {
        // add element
        return this;
    }
}
```

并且我们想要链式调用添加一个元素到创建空列表的方法。类型擦除再次出现；类型被擦除，所以下一个方法无法知道它。它不会编译。

```java
List<String> list = List.emptyList().add(":(");
```

这本来应该在 Java 8 中修复，但不幸的是它被放弃了。所以，至少目前，你仍然需要明确地向编译器提供一个类型；你仍然需要一个类型见证。

```java
List<String> list = List.<String>emptyList().add(":(");
```

# 方法引用

我之前提到方法引用有点像是 lambda 的快捷方式。它们是指向方法的一种简洁方便的方式，并允许该方法在任何 lambda 可以使用的地方使用。

当你创建一个 lambda 时，你创建了一个匿名函数并提供方法体。当你将方法引用作为 lambda 使用时，它实际上指向一个*已存在*的*命名*方法；它已经有了一个方法体。

你可以把它们想象成*将*一个常规方法转换为一个函数接口。

基本语法看起来像这样：

```java
Class::method
```

或者，一个更具体的例子：

```java
String::valueOf
```

双冒号之前的部分是目标引用，之后是方法名。所以，在这种情况下，我们的目标是`String`类，并且正在寻找一个叫做`valueOf`的方法；我们正在引用`String`上的`static`方法。

```java
public static String valueOf(Object obj) { ... }
```

双冒号被称为*分隔符*。当我们使用它时，我们不是在调用方法，只是*引用*它。所以记住不要在末尾加括号。

```java
String::valueOf(); // <-- error
```

你不能直接调用方法引用，它们只能用作 lambda 的替代。所以任何地方可以使用 lambda，你都可以使用方法引用。

## 例子

这个语句本身不会编译。

```java
public static void main(String... args) {
    String::valueOf;
}
```

这是因为方法引用不能转换为 lambda，因为编译器无法推断出要创建什么类型的 lambda。

*我们*碰巧知道这个引用等同于

```java
(x) -> String.valueOf(x)
```

但是编译器还不知道。它可以知道一些事情。它知道，作为 lambda，返回值应该是`String`类型，因为`String`中所有名为`valueOf`的方法都返回一个字符串。但它不知道要提供什么样的参数。我们需要给它一点帮助，给它一些更多的上下文。

我们将创建一个名为`Conversion`的函数接口，它接受一个整数并返回一个字符串。这将是我们 lambda 的目标类型。

```java
@FunctionalInterface
interface Conversion {
    String convert(Integer number);
}
```

接下来，我们需要创建一个场景，在这个场景中我们将使用它作为 lambda。所以我们创建一个小方法，接受一个函数接口并将一个整数应用到它上面。

```java
public static String convert(Integer number, Conversion function) {
    return function.convert(number);
}
```

现在，问题来了。我们刚刚给编译器足够的信息，以将方法引用转换为等效的 lambda。

当我们调用`convert`方法时，我们可以传入一个 lambda。

```java
convert(100, (number) -> String.valueOf(number));
```

我们可以用`valueOf`方法的引用替换 lambda。编译器现在知道我们需要一个返回字符串并接受一个整数的 lambda。它现在知道`valueOf`方法“适合”，并且可以替换整数参数。

```java
convert(100, String::valueOf);
```

给编译器提供所需的信息的另一种方法就是将引用分配给一个类型。

```java
Conversion b = (number) -> String.valueOf(number);
```

以及作为方法引用

```java
Conversion a = String::valueOf;
```

“形状”匹配，所以可以分配。

有趣的是，我们可以将相同的 lambda 分配给任何需要相同“形状”的接口。例如，如果我们有另一个具有相同“形状”的函数接口，

在这里，`Example`返回一个`String`并接受一个`Object`，所以它的签名形状与`valueOf`相同。

```java
interface Example {
    String theNameIsUnimportant(Object object);
}
```

我们仍然可以将方法引用（或 lambda）分配给它。

```java
Example a = String::valueOf;
```

## 方法引用类型

有四种方法引用类型：

+   构造函数引用

+   静态方法引用

+   和两种实例方法引用

最后两个有点混淆。第一个是特定对象的方法引用，第二个是*任意*对象的方法引用，但是是特定类型的。区别在于你想如何使用这个方法，以及是否提前有实例。

首先，让我们看看构造函数引用。

## 构造函数引用

基本语法看起来像这样：

```java
String::new
```

*目标*类型后跟双冒号，然后是`new`关键字。它将创建一个 lambda，将调用`String`类的零参数构造函数。

它等同于这个 lambda

```java
() -> new String()
```

记住，方法引用永远不带括号；它们不是在调用方法，只是引用一个方法。这个例子是指`String`类的构造函数，但*不*实例化一个字符串。

让我们看看我们可能如何实际使用构造函数引用。

如果我们创建一个对象列表，我们可能想要填充该列表，比如十个项目。因此，我们可以创建一个循环，并添加一个新对象十次。

```java
public void usage() {
    List<Object> list = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        list.add(new Object());
  }
}
```

但是，如果我们想要能够重用该初始化函数，我们可以将代码提取到一个名为`initialise`的新方法中，然后使用工厂来创建对象。

```java
public void usage() {
    List<Object> list = new ArrayList<>();
    initialise(list, ...);
}

private void initialise(List<Object> list, Factory<Object> factory){
    for (int i = 0; i < 10; i++) {
        list.add(factory.create());
    }
 }
```

`Factory`类只是一个带有名为`create`的方法的功能接口，该方法返回一些对象。然后我们可以将它创建的对象添加到列表中。因为它是一个功能接口，我们可以使用 lambda 来实现初始化列表的工厂：

```java
public void usage() {
    List<Object> list = new ArrayList<>();
    initialise(list, () -> new Object());
}
```

或者我们可以换成构造函数引用。

```java
public void usage() {
    List<Object> list = new ArrayList<>();
    initialise(list, Object::new);
}
```

这里还有一些其他事情我们可以做。如果我们给`initialise`方法添加一些泛型，我们可以在初始化任何类型的列表时重用它。例如，我们可以回过头将列表的类型更改为`String`，并使用构造函数引用进行初始化。

```java
public void usage() {
    List<String> list = new ArrayList<>();
    initialise(list, String::new);
}

private <T> void initialise(List<T> list, Factory<T> factory) {
    for (int i = 0; i < 10; i++) {
        list.add(factory.create());
    }
}
```

我们已经看到它如何适用于零参数构造函数，但是当类有多个参数构造函数时呢？

当存在多个构造函数时，您可以使用相同的语法，但编译器会找出哪个构造函数最匹配。它是基于*目标*类型和推断出的功能接口来完成这一点，它可以用来创建该类型。

让我们以`Person`类为例，它看起来像这样，您可以看到构造函数接受一堆参数。

```java
class Person {
    public Person(String forename, String surname, LocalDate    
    birthday, Sex gender, String emailAddress, int age) {
      // ...
    }
}
```

回到我们之前的例子，看看通用的`initialise`方法，我们可以使用这样的 lambda：

```java
initialise(people, () -> new Person(forename, surname, birthday,
                                    gender, email, age));
```

但是，要能够使用构造函数引用，我们需要一个带有可变参数的 lambda，看起来像这样：

```java
(a, b, c, d, e, f) -> new Person(a, b, c, d, e, f);
```

但这并不直接转换为构造函数引用。如果我们尝试使用

```java
Person::new
```

它不会编译，因为它不知道任何关于参数的信息。如果您尝试编译它，错误会说您创建了一个无效的构造函数引用，不能应用于给定的类型；它找不到参数。

相反，我们必须引入一些间接性，以便为编译器提供足够的信息来找到合适的构造函数。我们可以创建一些可以用作功能接口*并且*具有正确类型以适应适当构造函数的东西。

让我们创建一个名为`PersonFactory`的新功能接口。

```java
@FunctionalInterface
interface PersonFactory {
    Person create(String forename, String surname, LocalDate 
    birthday, Sex gender, String emailAddress, int age);
}
```

在这里，来自`PersonFactory`的参数与`Person`上可用的构造函数匹配。神奇的是，这意味着我们可以回过头使用`Person`的构造函数引用。

```java
public void example() {
    List<Person> list = new ArrayList<>();
    PersonFactory factory = Person::new;
    // ...
}
```

请注意，我正在使用`Person`的构造函数引用。这里需要注意的是，构造函数引用可以分配给目标功能接口，即使我们还不知道参数。

方法引用的类型是`PersonFactory`而不是`Person`可能看起来有点奇怪。这种额外的目标类型信息帮助编译器知道它必须通过`PersonFactory`来创建`Person`。有了这个额外的提示，编译器就能够创建一个基于工厂接口的 lambda，*稍后*创建一个`Person`。

写出来的话，编译器会生成这个。

```java
public void example() {
    PersonFactory factory = (a, b, c, d, e, f) -> new Person(a, b,   
    c, d, e, f);
}
```

之后可以像这样使用：

```java
public void example() {
    PersonFactory factory = (a, b, c, d, e, f) -> new Person(a, b,  
    c, d, e, f);
    Person person = factory.create(forename, surname, birthday,
                                gender, email, age);
}
```

幸运的是，一旦我们引入了间接性，编译器就可以为我们做到这一点。

它了解要使用的目标类型是`PersonFactory`，并且了解它的单个`abstract`方法可以用来代替构造函数。这有点像一个两步过程，首先，找出`abstract`方法与构造函数具有相同的参数列表，并且返回正确的类型，然后使用双冒号新语法应用它。

为了完成示例，我们需要调整我们的`initialise`方法以添加类型信息（替换泛型），添加参数来表示人的详细信息，并实际调用工厂。

```java
private void initialise(List<Person> list, PersonFactory factory, 
                       String forename, String surname,                           
                       LocalDate birthday, Sex gender,
                       String emailAddress, int age) {
                         for (int i = 0; i < 10; i++) {
                           list.add(factory.create(forename,  
                           surname, birthday, gender,        
                           emailAddress, age));
                         }
                       }
```

然后我们可以像这样使用它：

```java
public void example() {
    List<Person> list = new ArrayList<>();
    PersonFactory factory = Person::new;
    initialise(people, factory, a, b, c, d, e, f);
}
```

或者内联，像这样：

```java
public void example() {
    List<Person> list = new ArrayList<>();
    initialise(people, Person::new, a, b, c, d, e, f);
}
```

## 静态方法引用

方法引用可以直接指向静态方法。例如，

```java
String::valueOf
```

这一次，左侧指的是可以找到静态方法（在这种情况下是`valueOf`）的类型。它相当于这个 lambda

```java
x -> String.valueOf(x))
```

更多的例子是我们使用对`Comparators`类的静态方法的引用对集合进行排序。

```java
Collections.sort(Arrays.asList(5, 12, 4), Comparators::ascending);

// equivalent to
Collections.sort(Arrays.asList(5, 12, 4), (a, b) -> Comparators.ascending(a, b));
```

在这里，静态方法`ascending`可能是这样定义的：

```java
public static class Comparators {
    public static Integer ascending(Integer first, Integer second)   
    {
        return first.compareTo(second);
     }
}
```

## 特定对象的实例方法引用（在这种情况下是闭包）

这是特定实例的实例方法引用的一个示例：

```java
x::toString
```

`x`是我们想要获取的特定实例。它的 lambda 等效如下：

```java
() -> x.toString()
```

引用特定实例的方法的能力还为我们提供了一种方便的方法，可以在不同的功能接口类型之间进行转换。例如：

```java
Callable<String> c = () -> "Hello";
```

`Callable`的功能方法是调用。当调用时，lambda 将返回`"Hello"`。

如果我们有另一个功能接口`Factory`，我们可以使用方法引用将`Callable`转换。

```java
Factory<String> f = c::call;
```

我们可以重新创建 lambda，但这个技巧是一种有用的方法，可以重复使用预定义的 lambda。将它们分配给变量并重复使用，以避免重复。

以下是使用它的示例：

```java
public void example() {
    String x = "hello";
    function(x::toString);
}
```

这是一个使用闭包的方法引用的示例。它创建一个 lambda，将在实例`x`上调用`toString`方法。

上面的函数的签名和实现如下：

```java
public static String function(Supplier<String> supplier) {
    return supplier.get();
}
```

`Supplier`接口是一个看起来像这样的功能接口：

```java
@FunctionalInterface
public interface Supplier<T> {
  T get();
}
```

当在我们的函数中使用时，它提供一个字符串值（通过调用`get`），它唯一能做到这一点的方式是如果值在构造时已经被提供。它相当于：

```java
public void example() {
  String x = "";
  function(() -> x.toString());
}
```

请注意，这里的 lambda 没有参数（它使用“汉堡包”符号）。这表明`x`的值在 lambda 的局部范围内不可用，因此只能从其范围外部获得。它是一个闭包，因为必须关闭`x`（它将*捕获*`x`）。

如果您有兴趣看到长手，匿名类等效，它将如下所示。再次注意`x`必须传入：

```java
public void example() {
    String x = "";
    function(new Supplier<String>() {
        @Override
        public String get() {
            return x.toString(); // <- closes over 'x'
        }
    });
}
```

这三种都是等效的。将其与实例方法引用的 lambda 变体进行比较，其中它的参数不是显式从外部范围传入的。

## 稍后提供实例的任意对象的实例方法引用（lambda）

最后一种情况是指向由其类型引用的任意对象的方法引用：

```java
Object::toString
```

因此，在这种情况下，尽管看起来左侧指向一个类（就像`static`方法引用一样），但实际上指向一个实例。`toString`方法是`Object`上的实例方法，而不是`static`方法。之所以可能不使用常规实例方法语法，是因为您可能尚未有实例可供参考。

因此，在以前，当我们调用`x`冒号冒号`toString`时，我们知道`x`的值。有些情况下，您没有`x`的值，在这些情况下，您仍然可以传递方法的引用，但稍后使用此语法提供值。

例如，lambda 等效没有`x`的绑定值。

```java
(x) -> x.toString()
```

两种类型的实例方法引用之间的区别基本上是学术性的。有时，您需要传递一些内容，其他时候，lambda 的使用将为您提供它。

该示例类似于常规方法引用；它调用字符串的`toString`方法，只是这一次，字符串是由使用 lambda 的函数提供的，而不是从外部范围传入的。

```java
public void lambdaExample() {
    function("value", String::toString);
}
```

`String`部分看起来像是在引用一个类，但实际上是在引用一个实例。我知道这很令人困惑，但为了更清楚地看到事情，我们需要看到使用 lambda 的函数。它看起来像这样。

```java
public static String function(String value, Function<String, String> function) {
    return function.apply(value);
}
```

因此，字符串值直接传递给函数，它看起来像这样作为完全限定的 lambda。

```java
public void lambdaExample() {
    function("value", x -> x.toString());
}
```

Java 可以快捷方式查看`String::toString`；它表示在运行时“提供对象实例”。

如果将其完全扩展为匿名接口，它看起来像这样。`x`参数可用且未关闭。因此它是一个 lambda 而不是闭包。

```java
public void lambdaExample() {
    function("value", new Function<String, String>() {
      @Override
      // takes the argument as a parameter, doesn't need to close 
      over it
      public String apply(String x) {
        return x.toString();
      }
    });
}
```

## 总结

Oracle 描述了四种方法引用（[`docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html`](http://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)）如下：

| **种类** | **示例** |
| --- | --- |
| 对静态方法的引用 | `ContainingClass::staticMethodName` |
| 对特定对象的实例方法的引用 | `ContainingObject::instanceMethodName` |
| 对特定类型的任意对象的实例方法的引用 | `ContainingType::methodName` |
| 对构造函数的引用 | `ClassName::new` |

但是实例方法的描述只是纯粹令人困惑。到底是什么意思呢？特定类型的任意对象的实例方法？难道不是所有对象都是*特定类型的*吗？为什么重要的是对象是*任意的*？

我更喜欢将第一个视为*特定*对象的实例方法，而将第二个视为稍后将*提供*的任意对象的实例方法。有趣的是，这意味着第一个是*闭包*，第二个是*lambda*。一个是*绑定*的，另一个是*未绑定*的。方法引用的区别是闭包（闭包）和不闭包（lambda）可能有点学术，但至少这是一个比 Oracle 无用的描述更正式的定义。

| **种类** | **语法** | **示例** |
| --- | --- | --- |
| 对静态方法的引用 | `Class::staticMethodName` | `String::valueOf` |
| 对特定对象的实例方法的引用 | `object::instanceMethodName` | `x::toString` |
| 参考任意对象后提供的实例方法的引用 | `Class::instanceMethodName` | `String::toString` |
| 对构造函数的引用 | `ClassName::new` | `String::new` |

或等效的 lambda：

| **种类** | **语法** | **作为 Lambda** |
| --- | --- | --- |
| 对静态方法的引用 | `Class::staticMethodName` | `(s) -> String.valueOf(s)` |
| 对特定对象的实例方法的引用 | `object::instanceMethodName` | `() -> "hello".toString()` |
| 对任意对象后提供的实例方法的引用 | `Class::instanceMethodName` | `(s) -> s.toString()` |
| 对构造函数的引用 | `ClassName::new` | `() -> new String()` |

请注意，`static`方法引用的语法看起来非常类似于对类的实例方法的引用。编译器通过遍历每个适用的静态方法和每个适用的实例方法来确定使用哪一个。如果它找到两者都匹配，结果将是编译器错误。

您可以将整个过程视为从方法引用到 lambda 的转换。编译器提供了一个*转换*函数，它接受一个方法引用和目标类型，并可以推导出一个 lambda。

# 作用域

关于 lambda 的好消息是它们不会引入任何新的作用域。在 lambda 中使用变量将引用封闭环境中的变量。

这就是所谓的**词法作用域**。这意味着 lambda 根本不会引入新的作用域级别；您可以直接访问封闭范围中的字段、方法和变量。这也适用于**this**和**super**关键字。因此，我们不必担心解析作用域的疯狂嵌套类语法。

让我们来看一个例子。我们在这里有一个示例类，其中一个成员变量`i`设置为`5`的值。

```java
public static class Example {
    int i = 5;

    public Integer example() {
        Supplier<Integer> function = () -> i * 2;
        return function.get();
    }
}
```

在`example`方法中，lambda 使用一个名为`i`的变量，并将其乘以 2。

因为 lambda 是词法作用域的，`i`简单地指的是封闭类的变量。它在运行时的值将是 5。使用`this`来强调这一点；在 lambda 中，`this`与外部相同。

```java
public static class Example {
    int i = 5;
    public Integer example() {
        Supplier<Integer> function = () -> this.i * 2;
        return function.get();
    }
}
```

在下面的`anotherExample`方法中，使用了一个方法参数，也叫做`i`。通常的遮蔽规则在这里生效，`i`将指代方法参数而不是类成员变量。方法变量*遮蔽*了类变量。它的值将是传入方法的任何值。

```java
public static class Example {
    int i = 5;

    public Integer anotherExample(int i) {
        Supplier<Integer> function = () -> i * 2;
        return function.get();
    }
}
```

如果你想引用类变量`i`而不是参数`i`，你可以使用这个方法使变量明确。例如，`Supplier<Integer> function = () -> i * 2;`。

以下示例在`yetAnotherExample`方法中定义了一个局部作用域变量。记住，lambda 使用其封闭作用域作为自己的作用域，所以在这种情况下，lambda 中的`i`指的是方法的变量；`i`将是`15`而不是`5`。

```java
public static class Example {
    int i = 5;

    public Integer yetAnotherExample() {
        int i = 15;
        Supplier<Integer> function = () -> i * 2;
        return function.get();
    }
}
```

如果你想亲自看看，你可以使用以下方法打印出数值：

```java
public static void main(String... args) {
    Example scoping = new Example();
    System.out.println("class scope        = " +    
    scoping.example());
    System.out.println("method param scope = " +  
    scoping.anotherExample(10));
    System.out.println("method scope       = " +   
    scoping.yetAnotherExample());
}
```

输出将如下所示：

```java
class scope        = 10
method param scope = 20
method scope       = 30
```

因此，第一个方法打印`10`；类变量乘以 2 得到 5。第二个方法打印`20`，因为参数值为 10，乘以 2 得到 20，最后一个方法打印`30`，因为本地方法变量设置为 15，再次乘以 2 得到 30。

词法作用域意味着推迟到封闭环境。每个例子都有一个不同的封闭环境或作用域。你看到一个变量定义为类成员、方法参数和方法内部的局部变量。在所有情况下，lambda 的行为都是一致的，并引用了这些封闭作用域中的变量。

如果你已经熟悉基本的 Java 作用域，Lambda 作用域应该是直观的，这里真的没有什么新东西。

## 实际上 final

在 Java 7 中，传递给匿名类实例的任何变量都需要被设为 final。

这是因为编译器实际上将它所需的所有上下文或*环境*复制到匿名类的实例中。如果这些值在其下发生变化，可能会发生意外的副作用。因此 Java 要求变量是 final，以确保它不会改变，内部类可以安全地对其进行操作。安全地，我是指没有线程之间的竞争条件或可见性问题。

让我们来看一个例子。首先，我们将使用 Java 7 创建一个名为`filter`的方法，该方法接受一个人员列表和一个谓词。我们将创建一个临时列表来包含我们找到的任何匹配项，然后枚举每个元素，测试谓词对每个人是否成立。如果测试是积极的，我们将把他们添加到临时列表中，然后返回所有匹配项。

```java
// java 7
private List<Person> filter(List<Person> people, Predicate<Person> predicate) {
    ArrayList<Person> matches = new ArrayList<>();
    for (Person person : people)
        if (predicate.test(person))
            matches.add(person);
    return matches;
}
```

然后我们将创建一个方法，使用这个方法来找到列表中所有符合退休条件的人。我们设置一个退休年龄变量，然后调用`filter`方法，传入一个任意的人员列表和一个`Predicate`接口的新匿名实例。

我们将实现这个方法，如果一个人的年龄大于或等于退休年龄变量，就返回 true。

```java
 public void findRetirees() {
     int retirementAge = 55;
     List<Person> retirees = filter(allPeople, new 
     Predicate<Person>   
     () {
         @Override
         public boolean test(Person person) {
             return person.getAge() >= retirementAge; // <-- 
             compilation error
         }
     });
 }
```

如果你尝试编译这个，当访问变量时会得到编译器失败。这是因为变量不是 final。我们需要添加`final`使其编译。

```java
final int retirementAge = 55;
```

### 注意

像这样将环境传递给匿名内部类是闭包的一个例子。环境是闭包"封闭"的内容；它必须捕获它需要执行任务的变量。Java 编译器使用复制技巧来实现这一点，而不是尝试管理对同一变量的多次更改。在闭包的上下文中，这被称为*变量捕获*。

Java 8 引入了"实际上 final"的概念，这意味着如果编译器可以确定特定变量*永远*不会改变，它可以在任何需要 final 变量的地方使用。它将其解释为"实际上"final。

在我们的例子中，如果我们切换到 Java 8 并删除`final`关键字。事情仍然可以编译。不需要使变量为 final。Java 知道变量不会改变，因此它实际上是 final。

```java
int retirementAge = 55;
```

当然，如果你把它设为`final`，它仍然可以编译。

但是如果我们在初始化后尝试修改变量呢？

```java
int retirementAge = 55;
// ...
retirementAge = 65;
```

编译器发现了变化，不能再将变量视为有效的最终值。我们得到了原始的编译错误，要求我们将其声明为最终值。相反，如果在变量声明中添加`final`关键字不会导致编译错误，那么该变量就是有效的最终值。

我一直在用匿名类示例来演示这一点，因为有效最终值的概念并不是特定于 lambda 的。当然，它也适用于 lambda。您可以将上面的匿名类转换为 lambda，而不会发生任何变化。仍然不需要将变量声明为最终值。

### 规避最终值

您仍然可以通过传递最终对象或数组来绕过安全网，然后在 lambda 中更改它们的内部。

例如，我们拿着我们的人员名单，假设我们想要计算他们所有的年龄总和。我们可以创建一个循环和求和的方法，就像这样：

```java
private static int sumAllAges(List<Person> people) {
    int sum = 0;
    for (Person person : people) {
        sum += person.getAge();
    }
    return sum;
}
```

其中，当列表被枚举时，保持总和计数。作为一种替代，我们可以尝试抽象出循环行为，并传入一个要应用于每个元素的函数。就像这样：

```java
public final static Integer forEach(List<Person> people, Function<Integer, Integer> function) {
  Integer result = null;
  for (Person t : people) {
    result = function.apply(t.getAge());
  }
  return result;
}
```

为了实现求和行为，我们只需要创建一个可以求和的函数。您可以使用匿名类来做到这一点：

```java
private static void badExample() {
    Function<Integer, Integer> sum = new Function<Integer, Integer>   
   () {
        private Integer sum = 0;

        @Override
        public Integer apply(Integer amount) {
            sum += amount;
            return sum;
        }
   };
}
```

函数的方法接受一个整数并返回一个整数。在实现中，总和变量是一个类成员变量，并且每次应用函数时都会发生变化。这种变化通常在函数式编程中是不好的形式。

尽管如此，我们可以像这样将其传递给我们的`forEach`方法：

```java
forEach(allPeople, sum);
```

我们将得到所有人年龄的总和。这是因为我们使用了相同的函数实例，所以`sum`变量在每次迭代期间被重用和改变。

坏消息是我们不能直接将这个转换成 lambda；在 lambda 中没有成员变量的等价物，所以除了在 lambda 之外没有地方可以放置`sum`变量。

```java
double sum = 0;
forEach(allPeople, x -> {
    return sum += x;
});
```

但这突显了变量并不是有效的最终值（它在 lambda 的函数体中被改变），所以它必须被声明为最终值。

但如果我们将它声明为最终值

```java
final double sum = 0;
forEach(allPeople, x -> {
    return sum += x;
});
```

我们不能再在函数体内修改它了！这是一个鸡生蛋蛋生鸡的情况。

绕过最终值

```java
int[] sum = {0};
forEach(allPeople, x -> sum[0] += x);
```

数组引用在这里确实是最终的，但我们可以修改数组内容而不重新分配引用。然而，这通常是不好的形式，因为它会引发我们之前讨论的所有安全问题。我提到这个只是为了举例说明，但不建议您经常这样做。通常最好不要创建具有副作用的函数，如果您使用更多的函数式方法，完全可以避免这些问题。这种求和的更符合惯例的方法是使用所谓的*fold*或在 Java 术语中称为*reduce*。

# 异常处理

在 lambda 中没有新的异常处理语法。lambda 中抛出的异常会传播到调用者，就像您对常规方法调用的预期一样。调用 lambda 或处理它们的异常没有任何特殊之处。

然而，有一些微妙之处需要您注意。首先，作为 lambda 的调用者，您可能不知道可能抛出什么异常，如果有的话，其次，作为 lambda 的作者，您可能不知道您的 lambda 将在什么上下文中运行。

当您创建一个 lambda 时，通常会将如何执行该 lambda 的责任交给您传递给它的方法。就你所知，您的 lambda 可能会并行运行，或者在将来的某个时刻运行，因此您抛出的任何异常可能不会得到您的预期处理。您不能依赖异常处理来控制程序的流程。

为了证明这一点，让我们编写一些代码来依次调用两个东西。我们将使用`Runnable`作为一个方便的 lambda 类型。

```java
public static void runInSequence(Runnable first, Runnable second) {
    first.run();
    second.run();
}
```

如果第一次调用 run 抛出异常，方法将终止，第二个方法将不会被调用。调用者需要处理异常。如果我们使用这种方法在两个银行账户之间转账，我们可能会编写两个 lambda。一个用于借方操作，一个用于贷方。

```java
public void transfer(BankAccount a, BankAccount b, Integer amount) {
    Runnable debit = () -> a.debit(amount);
    Runnable credit = () -> b.credit(amount);
 }
```

然后我们可以这样调用我们的`runInSequence`方法：

```java
public void transfer(BankAccount a, BankAccount b, Integer amount) {
    Runnable debit = () -> a.debit(amount);
    Runnable credit = () -> b.credit(amount);
    runInSequence(debit, credit);
 }
```

任何异常都可以通过使用`try`/`catch`来捕获和处理，就像这样：

```java
public void transfer(BankAccount a, BankAccount b, Integer amount) {
    Runnable debit = () -> a.debit(amount);
    Runnable credit = () -> b.credit(amount);
    try {
        runInSequence(debit, credit);
    } catch (Exception e) {
        // check account balances and rollback
    }
  }
```

这就是问题所在。作为 lambda 的作者，我可能根本不知道`runInSequence`是如何实现的。它很可能是异步运行的，就像这样：

```java
public static void runInSequence(Runnable first, Runnable second) {
    new Thread(() -> {
        first.run();
        second.run();
    }).start();
}
```

在这种情况下，第一个调用中的任何异常都将终止线程，异常将消失到默认的异常处理程序，我们原始的客户端代码将没有机会处理异常。

## 使用回调

顺便说一句，解决在不同线程上引发异常的特定问题的一种方法是使用回调函数。首先，您可以防御`runInSequence`方法中的异常：

```java
public static void runInSequence(Runnable first, Runnable second) {
    new Thread(() -> {
        try {
            first.run();
            second.run();
        } catch (Exception e) {
            // ...
        }
    }).start();
}
```

然后引入一个异常处理程序，可以在发生异常时调用：

```java
public static void runInSequence(Runnable first, Runnable second,
        Consumer<Throwable> exceptionHandler) {
    new Thread(() -> {
        try {
            first.run();
            second.run();
        } catch (Exception e) {
            exceptionHandler.accept(e);
        }
    }).start();
}
```

Consumer 是一个函数接口（Java 8 中的新功能），在这种情况下，它将异常作为参数传递给它的`accept`方法。

当我们将其连接到客户端时，我们可以传递一个回调 lambda 来处理任何异常。

```java
public void nonBlockingTransfer(BankAccount a, BankAccount b, Integer amount) {
    Runnable debit = () -> a.debit(amount);
    Runnable credit = () -> b.credit(amount);
    runInSequence(debit, credit, (exception) -> {
      /* check account balances and rollback */
    });
}
```

这是延迟执行的一个很好的例子，因此它有自己的缺点。异常处理程序方法可能（或可能不会）在以后的某个时间点执行。`nonBlockingTransfer`方法将已经完成，银行账户本身可能在它触发时处于其他状态。您不能指望在方便的时候调用异常处理程序；我们已经打开了一个并发问题的大潘头。

## 处理编写 lambda 时的异常

让我们从 lambda 作者的角度来看待处理异常。之后，我们将看看在调用 lambda 时如何处理异常。

让我们看看如果我们想使用 lambda 来实现`transfer`方法，但这次想要重用一个提供`runInSequence`方法的现有库。

在开始之前，让我们看一下`BankAccount`类。您会注意到这次，`debit`和`credit`方法都会抛出一个已检查的异常；`InsufficientFundsException`。

```java
class BankAccount {
    public void debit(int amount) throws InsufficientFundsException    
    {
        // ...
     }

     public void credit(int amount) throws 
     InsufficientFundsException   
     {
         // ...
      }
}

class InsufficientFundsException extends Exception { }
```

让我们重新创建`transfer`方法。我们将尝试创建借方和贷方 lambda，并将其传递给`runInSequence`方法。请记住，`runInSequence`方法是由某个库的作者编写的，我们无法看到或修改它的实现。

```java
public void transfer(BankAccount a, BankAccount b, Integer amount) {
    Runnable debit = () -> a.debit(amount);   <- compiler error
    Runnable credit = () -> b.credit(amount); <- compiler error
    runInSequence(debit, credit);
 }
```

借方和贷方都会抛出一个已检查的异常，所以这次你会看到一个编译错误。如果我们将其添加到方法签名中也没有任何区别；异常会发生在 lambda 内部。记住我说过 lambda 中的异常会传播给调用者吗？在我们的情况下，这将是`runInSequence`方法，而不是我们定义 lambda 的地方。它们之间没有相互通信，可能会引发异常。

```java
// still doesn't compile
public void transfer(BankAccount a, BankAccount b, Integer amount)
       throws InsufficientFundsException {
    Runnable debit = () -> a.debit(amount);
    Runnable credit = () -> b.credit(amount);
    runInSequence(debit, credit);
}
```

因此，如果我们无法强制要求已检查的异常在 lambda 和调用者之间是*透明*的，一种选择是将已检查的异常包装为运行时异常，就像这样：

```java
public void transfer(BankAccount a, BankAccount b, Integer amount) {
    Runnable debit = () -> {
        try {
            a.debit(amount);
        } catch (InsufficientFundsException e) {
            throw new RuntimeException(e);
        }
   };
   Runnable credit = () -> {
       try {
           b.credit(amount);
       } catch (InsufficientFundsException e) {
           throw new RuntimeException(e);
       }
   };
   runInSequence(debit, credit);
 }
```

这样可以解决编译错误，但这还不是全部。这非常冗长，我们仍然需要捕获和处理现在是运行时异常的调用`runInSequence`。

```java
public void transfer(BankAccount a, BankAccount b, Integer amount){
    Runnable debit = () -> { ... };
    };
    Runnable credit = () -> { ... };
    try {
        runInSequence(debit, credit);
    } catch (RuntimeException e) {
        // check balances and rollback
    }
}

```

尽管还有一两个小问题；我们抛出并捕获了一个`RuntimeException`，这可能有点松散。我们真的不知道在`runInSequence`方法中可能会抛出什么其他异常。也许更明确一些会更好。让我们创建`RuntimeException`的一个新子类型，然后使用它。

```java
class InsufficientFundsRuntimeException extends RuntimeException {
    public   
    InsufficientFundsRuntimeException(InsufficientFundsException   
    cause) {
        super(cause);
    }
}
```

在我们修改原始 lambda 以抛出新异常之后，我们可以限制 catch 处理只处理我们知道的异常；即`InsufficientFundsRuntimeException`。

现在我们可以实现某种余额检查和回滚功能，并确信我们理解了可能导致它的所有情况。

```java
public void transfer(BankAccount a, BankAccount b, Integer amount) {
    Runnable debit = () -> {
        try {
            a.debit(amount);
        } catch (InsufficientFundsException e) {
            throw new InsufficientFundsRuntimeException(e);
        }
   };
   Runnable credit = () -> {
       try {
           b.credit(amount);
       } catch (InsufficientFundsException e) {
           throw new InsufficientFundsRuntimeException(e);
       }
   };
   try {
       runInSequence(debit, credit);
   } catch (InsufficientFundsRuntimeException e) {
       // check balances and rollback
   }
 }
```

所有这些麻烦之处在于，代码中的异常处理样板比实际业务逻辑更多。Lambda 应该使事情变得不那么冗长，但这里充满了噪音。如果我们将受检异常的包装泛化为运行时等价物，我们可以做得更好。我们可以创建一个捕获异常类型的泛型签名的函数接口。

让我们称它为`Callable`，它的单个方法是`call`。不要将其与 JDK 中同名的类混淆；我们正在创建一个新类来说明如何处理异常。

```java
@FunctionalInterface
interface Callable<E extends Exception> {
    void call() throws E;
}
```

我们将更改 transfer 的旧实现，并创建 lambda 以匹配新函数接口的“形状”。我暂时省略了类型。

```java
public void transfer(BankAccount a, BankAccount b, Integer amount) {
    ??? debit = () -> a.debit(amount);
    ??? credit = () -> b.credit(amount);
 }
```

从类型推断部分可以看出，Java 可以将其视为`Callable`类型，因为它没有参数，`Callable`也没有参数，它具有相同的返回类型（无）并且抛出与接口相同类型的异常。我们只需要给编译器一个提示，这样我们就可以将其分配给`Callable`的实例。

```java
public void transfer(BankAccount a, BankAccount b, Integer amount) {
    Callable<InsufficientFundsException> debit = () ->   
    a.debit(amount);
    Callable<InsufficientFundsException> credit = () -> 
    b.credit(amount);
 }
```

像这样创建 lambda 不会导致编译错误，因为函数接口声明可能会抛出异常。它不需要在我们创建 lambda 的时候警告我们，因为函数方法的签名会在我们尝试调用它时引发编译器错误，如果需要的话。就像普通方法一样。

如果我们尝试将它们传递给`runInSequence`方法，我们将会得到一个编译错误。

```java
public void transfer(BankAccount a, BankAccount b, Integer amount) {
    Callable<InsufficientFundsException> debit = () -> 
    a.debit(amount);
    Callable<InsufficientFundsException> credit = () -> 
    b.credit(amount);
    runInSequence(debit, credit); <- doesn't compile
 }
```

lambda 的类型是错误的。我们仍然需要一个`Runnable`类型的 lambda。我们将不得不编写一个方法，可以将`Callable`转换为`Runnable`。与此同时，我们将受检异常包装为运行时异常。就像这样：

```java
public static Runnable unchecked(Callable<InsufficientFundsException> function) {
    return () -> {
        try {
            function.call();
        } catch (InsufficientFundsException e) {
            throw new InsufficientFundsRuntimeException(e);
        }
    };
}
```

现在要做的就是将其连接到我们的 lambda 中：

```java
public void transfer(BankAccount a, BankAccount b, Integer amount) {
    Runnable debit = unchecked(() -> a.debit(amount));
    Runnable credit = unchecked(() -> b.credit(amount));
    runInSequence(debit, credit);
 }
```

一旦我们把异常处理放回去，我们就回到了更简洁的方法体，并且以与以前相同的方式处理了潜在的异常。

```java
public void transfer(BankAccount a, BankAccount b, Integer amount) {
    Runnable debit = unchecked(() -> a.debit(amount));
    Runnable credit = unchecked(() -> b.credit(amount));
    try {
         runInSequence(debit, credit);
    } catch (InsufficientFundsRuntimeException e) {
        // check balances and rollback
    }
 }
```

这个缺点是这不是一个完全通用的解决方案；我们仍然需要为不同的函数创建 unchecked 方法的变体。我们只是把冗长的语法隐藏起来了。冗长仍然存在，只是被移动了。是的，我们已经从中获得了一些重复利用，但是如果异常处理是透明的，或者我们没有检查异常，我们就不需要掩盖问题了。

值得指出的是，如果我们在 Java 7 中使用匿名类而不是 lambda，我们可能最终会做类似的事情。在 Java 8 之前仍然可以做很多这样的事情，你最终会创建辅助方法来推迟冗长。

当然，lambda 为小型匿名功能提供了更简洁的表示，但由于 Java 的受检异常模型，处理 lambda 中的异常通常会导致我们之前遇到的所有冗长问题。

## 作为调用者（在调用 lambda 时处理异常）

我们已经从编写 lambda 的角度看到了一些事情，现在让我们看看在调用 lambda 时的情况。

现在假设我们正在编写一个提供`runInSequence`方法的库。这次我们有更多的控制权，不再局限于使用`Runnable`作为 lambda 类型。因为我们不想强迫客户在他们的 lambda 中处理异常（或将它们包装为运行时异常），我们将提供一个函数接口，声明可能会抛出受检异常。

我们将称它为`FinancialTransfer`，有一个`transfer`方法：

```java
@FunctionalInterface
interface FinancialTransfer {
    void transfer() throws InsufficientFundsException;
}
```

我们在说，每当发生银行交易时，都有可能出现资金不足的情况。然后当我们实现我们的`runInSequence`方法时，我们接受这种类型的 lambda。

```java
public static void runInSequence(FinancialTransfer first,
        FinancialTransfer second) throws InsufficientFundsException      
{
    first.transfer();
    second.transfer();
 }
```

这意味着当客户端使用该方法时，他们不必强制在他们的 lambda 内部处理异常。例如，编写这样的方法。

```java
// example client usage
public void transfer(BankAccount a, BankAccount b, Integer amount) {
    FinancialTransfer debit = () -> a.debit(amount);
    FinancialTransfer credit = () -> b.credit(amount);
 }
```

这次在创建 lambda 时不会有编译器错误。无需将`BankAccount`方法的异常包装为运行时异常；函数接口已经声明了异常。但是，`runInSequence`现在会抛出一个已检查的异常，因此明确表示客户端必须处理可能性，您将看到一个编译器错误。

```java
public void transfer(BankAccount a, BankAccount b, Integer amount) {
    FinancialTransfer debit = () -> a.debit(amount);
    FinancialTransfer credit = () -> b.credit(amount);
    runInSequence(debit, credit);  <- compiler error
 }
```

因此，我们需要用`try`/`catch`来包装调用，以使编译器满意：

```java
public void transfer(BankAccount a, BankAccount b, Integer amount) {
    FinancialTransfer debit = () -> a.debit(amount);
    FinancialTransfer credit = () -> b.credit(amount);
    try {
        runInSequence(debit, credit);  <- compiler error
    } catch (InsufficientFundsException e) {
        // whatever
    }
 }
```

最终的结果类似于我们之前看到的，但不需要未经检查的方法。作为库开发人员，我们使客户更容易与我们的代码集成。

但是，如果我们尝试一些更奇特的东西呢？让我们再次将`runInSequence`方法异步化。无需从方法签名中抛出异常，因为如果从不同的线程中抛出异常，它不会传播到调用者。因此，这个版本的`runInSequence`方法不包括 throws 子句，`transfer`方法也不再被强制处理异常。然而，对`.transfer`的调用仍然会抛出异常。

```java
public static void runInSequence(Runnable first, Runnable second) {
    new Thread(() -> {
        first.transfer();   <- compiler error
        second.transfer();  <- compiler error
    }).start();
}

public void transfer(BankAccount a, BankAccount b, Integer amount) {
    FinancialTransfer debit = () -> a.debit(amount);
    FinancialTransfer credit = () -> b.credit(amount);
    runInSequence(debit, credit);  <- compiler error
 }
```

由于`runInSequence`方法中仍然存在编译器错误，我们需要另一种处理异常的方法。一种技术是传入一个在发生异常时将被调用的函数。我们可以使用这个 lambda 将异步运行的代码与调用者连接起来。

首先，我们将重新添加`catch`块，并传入一个函数接口作为异常处理程序。我将在这里使用`Consumer`接口，它是 Java 8 中的新功能，并且是`java.util.function`包的一部分。然后在 catch 块中调用接口方法，传入原因。

```java
public void runInSequence(FinancialTransfer first,     
       FinancialTransfer second,
       Consumer<InsufficientFundsException> exceptionHandler) {
    new Thread(() -> {
        try {
            first.transfer();
            second.transfer();
        } catch (InsufficientFundsException e) {
            exceptionHandler.accept(e);
        }
    }).start();
}
```

要调用它，我们需要更新`transfer`方法，以传入一个回调的 lambda。下面的参数 exception 将是传递给`runInSequence`中的`accept`方法的任何内容。它将是`InsufficientFundsException`的一个实例，客户端可以自行处理。

```java
public void transfer(BankAccount a, BankAccount b, Integer amount) {
    FinancialTransfer debit = () -> a.debit(amount);
    FinancialTransfer credit = () -> b.credit(amount);
    Consumer<InsufficientFundsException> handler = (exception) -> {
        /* check account balances and rollback */
   };
   runInSequenceNonBlocking(debit, credit, handler);
}
```

我们已经为我们的库提供了一种替代的异常处理机制，而不是强制它们捕获异常。

我们已经将异常处理内部化到库代码中。这是延迟执行的一个很好的例子；如果有异常，客户端不一定知道何时会调用他的异常处理程序。例如，由于我们在另一个线程中运行，银行账户本身可能在执行时已经被更改。再次强调，使用异常来控制程序流是一种有缺陷的方法。您不能指望在方便时调用异常处理程序。

# Lambda vs 闭包

术语*闭包*和*lambda*经常可以互换使用，但它们实际上是不同的。在本节中，我们将看一下它们之间的区别，以便您清楚地知道哪个是哪个。

下面是显示每个 Java 主要版本的发布日期的表格。Java 5.0 于 2004 年推出，包括了第一个重大的语言更改，包括泛型支持等内容：

![Lambdas vs closures](img/timeline.jpg)

大约在 2008 年至 2010 年期间，有很多工作正在进行，以引入闭包到 Java 中。原本计划在 Java 7 中引入，但最终没有及时实现。相反，它演变为了 Java 8 中的 lambda 支持。不幸的是，在那个时候，人们经常互换使用术语“闭包”和“lambda”，因此对于 Java 社区来说有点混淆。事实上，OpenJDK 网站上仍然有一个[closures](http://openjdk.java.net/projects/closures/)的项目页面，还有一个[lambdas](http://openjdk.java.net/projects/lambda/)的项目页面。

从 OpenJDK 项目的角度来看，他们真的应该从一开始就一直使用“lambda”。事实上，OpenJDK 搞错了，他们忽略了 Java 自 1.1 版本以来就已经支持闭包。

我在这里有点追究，因为尽管闭包和 lambda 之间存在技术上的差异，但两个项目的目标都是实现相同的事情，即使它们在术语上使用不一致。

那么 lambda 和闭包之间的区别是什么？基本上，闭包*是*lambda 的一种类型，但 lambda 不一定是闭包。

## 基本区别

就像 lambda 一样，闭包实际上是一个匿名的功能块，但有一些重要的区别。闭包依赖于外部值（不仅仅是它的参数），而 lambda 只依赖于它的参数。闭包被认为是“关闭”它所需的环境。

例如，以下内容：

```java
(server) -> server.isRunning();
```

是一个 lambda，但这

```java
() -> server.isRunning();
```

这是一个闭包。

它们都返回一个布尔值，指示某个服务器是否正在运行，但一个使用它的参数，另一个必须从其他地方获取变量。它们都是 lambda；从一般意义上讲，它们都是匿名的功能块，在 Java 语言意义上，它们都使用新的 lambda 语法。

第一个例子是指传递给 lambda 的服务器变量作为参数，而第二个例子（闭包）从其他地方获取服务器变量；也就是环境。为了获取变量的实例，lambda 必须“关闭”环境或捕获 server 的值。我们在之前谈到“有效最终”时已经看到了这一点。

让我们扩展这个例子，以便更清楚地看到事情。首先，我们将在一个静态类中创建一个方法来执行一个简单的轮询和等待。它将在每次轮询时检查一个函数接口，以查看是否满足某个条件。

```java
class WaitFor {
    static <T> void waitFor(T input, Predicate<T> predicate)
            throws InterruptedException {
        while (!predicate.test(input))
            Thread.sleep(250);
    }
}
```

我们使用`Predicate`（另一个新的`java.util`接口）作为我们的函数接口，并测试它，如果条件不满足，就暂停一小段时间。我们可以用一个简单的 lambda 调用这个方法，检查某个 HTTP 服务器是否正在运行。

```java
void lambda() throws InterruptedException {
    waitFor(new HttpServer(), (server) -> !server.isRunning());
}
```

服务器参数由我们的`waitFor`方法提供，并且将是我们刚刚定义的`HttpServer`的实例。它是一个 lambda，因为编译器不需要捕获服务器变量，因为我们在运行时手动提供它。

### 注意

顺便说一句，我们可能可以使用方法引用... `waitFor(new HttpServer(), HttpServer::isRunning);`

现在，如果我们将其重新实现为一个闭包，它将看起来像这样。首先，我们必须添加另一个`waitFor`方法。

```java
static void waitFor(Condition condition) throws InterruptedException {
    while (!condition.isSatisfied())
        Thread.sleep(250);
}
```

这次，使用一个更简单的签名。我们传递一个不需要参数的函数接口。`Condition`接口有一个简单的`isSatisfied`方法，没有参数，这意味着我们必须提供实现可能需要的任何值。它已经暗示了它的用法可能会导致闭包。

使用它，我们会写成这样：

```java
void closure() throws InterruptedException {
    Server server = new HttpServer();
    waitFor(() -> !server.isRunning());
}
```

这里服务器实例没有作为参数传递给 lambda，而是从封闭的作用域中访问。我们已经定义了变量，lambda 直接使用它。这个变量必须被编译器捕获或复制。lambda“关闭”了服务器变量。

这个“关闭”表达式来自这样一个想法，即具有开放绑定（或自由变量）的 lambda 表达式已经被词法环境或作用域关闭（或绑定）。结果是一个*封闭的表达式*。没有未绑定的变量。更准确地说，闭包关闭的是*值*而不是*变量*。

我们已经看到闭包被用来提供一个匿名的功能块，以及等效的 lambda 之间的区别，但是我们仍然可以做出更有用的区分。

## 其他区别

匿名函数是一个没有名称的函数文字，而闭包是函数的一个实例。根据定义，lambda 没有实例变量；它不是一个实例。它的变量是作为参数提供的。然而，闭包有实例变量，这些变量在实例创建时提供。

有了这个理解，lambda 通常比闭包更有效，因为它只需要评估一次。一旦有了函数，就可以重复使用它。由于闭包封闭了其本地环境之外的东西，因此每次调用它都必须进行评估。每次使用时都必须新建一个实例。

我们在函数与类部分看到的所有问题在这里也是相关的。使用闭包而不是 lambda 可能会考虑到内存问题。

## 总结

我们在这里谈论了很多，让我们简要总结一下区别。

Lambda 只是匿名函数，类似于 Java 中的静态方法。就像静态方法一样，它们不能引用其范围之外的变量，除了它们的参数。一种特殊类型的 lambda，称为闭包，可以捕获其范围之外的变量（或者封闭它们），以便可以使用外部变量或它们的参数。因此，简单的规则是，如果 lambda 使用了其范围之外的变量，它也是一个闭包。

闭包可以被视为函数的实例。这对于 Java 开发人员来说有点奇怪的概念。

一个很好的例子是传统的匿名类，如果没有新的 lambda 语法，我们将传递它们。这些可以“封闭”变量，因此它们本身也是闭包。因此，自 Java 1.1 以来，我们一直在 Java 中支持闭包。

看看这个例子。服务器变量必须由编译器封闭，以便在`Condition`接口的匿名实例中使用。这既是匿名类实例，也是闭包。

```java
@since Java 1.1!
void anonymousClassClosure() {
    Server server = new HttpServer();
    waitFor(new Condition() {
        @Override
        public Boolean isSatisfied() {
            return !server.isRunning();
        }
    });
}
```

Lambda 并不总是闭包，但闭包总是 lambda。

在本节中，我们将探讨当您将匿名类编译为 lambda 时，编译器输出的差异。首先，我们会提醒自己关于 java 字节码以及如何阅读它。然后，我们将研究匿名类和 lambda 在捕获变量和不捕获变量时的情况。我们将比较 Java 8 之前的闭包与 lambda，并探讨 lambda 并不仅仅是语法糖，而是与传统方法产生非常不同的字节码。

# 字节码回顾

首先，让我们回顾一下我们对字节码的了解。

从源代码到可执行的机器代码。Java 编译器生成字节码。这要么由 JVM 解释，要么由即时编译器重新编译。

当它被解释时，字节码会即时转换为机器码并执行。这每次遇到字节码时都会发生，但是 JVM。

当它被即时编译时，JVM 会直接将其编译成机器码，然后继续执行。

两者都发生在运行时，但即时编译提供了许多优化。

因此，Java 字节码是源代码和机器代码之间的中间表示。

### 注意

作为一个快速的旁注：Java 的 JIT 编译器多年来一直享有很好的声誉。但回到我们的介绍，是约翰·麦卡锡在 1960 年首次写了关于 JIT 编译的。因此，有趣的是，不仅 lambda 支持受到 LISP 的影响。 ([Aycock 2003, 2. JIT Compilation Techniques, 2.1 Genesis, p. 98](http://user.it.uu.se/~kostis/Teaching/KT2-04/jit_survey.pdf))。

字节码是 JVM 的指令集。顾名思义，字节码由单字节指令（称为*操作码*）以及与参数相关的字节组成。因此，可能有 256 个操作码可用，尽管实际使用的只有大约 200 个。

JVM 使用基于栈的计算模型，如果我们想要增加一个数字，必须使用栈来实现。所有指令或操作码都针对栈进行操作。

所以例如，5 + 1 变成了 5 1 +，其中 5 被推入栈中，

![字节码概述](img/Push5.jpg)

1 被推入栈，然后...

![字节码概述](img/Push1.jpg)

应用了 `+` 运算符。加号会弹出栈顶的两个帧，将数字相加，然后将结果推回栈上。结果如下。

![字节码概述](img/iadd6.jpg)

每个操作码都像这样针对栈进行操作，所以我们可以将我们的示例转换为一系列 Java 字节码：

![字节码概述](img/Push5.jpg)

`push 5` 操作码变成了 `iconst_5`。

![字节码概述](img/iconst1.jpg)

`push 1` 操作码变成了 `iconst_1`：

![字节码概述](img/iconst_1_5.jpg)

`add` 变成了 `iadd`。

![字节码概述](img/iadd6.jpg)

`iconst_x` 操作码和 `iadd` 操作码是操作码的示例。操作码通常具有前缀和/或后缀来指示它们作用的类型，在这些示例中，`i` 指的是整数，`x` 是特定于操作码的后缀。

我们可以将操作码分为以下几类：

| **组** | **示例** |
| --- | --- |
| 栈操作 | `aload_n`, `istore`, `swap`, `dup2` |
| 控制流指令 | `goto`, `ifeq`, `iflt` |
| 对象交互 | `new`, `invokespecial`, `areturn` |
| 算术、逻辑和类型转换 | `iadd`, `fcmpl`, `i2b` |

与栈操作有关的指令，如 `aload` 和 `istore`。

使用像 `goto` 和 if equal 这样的操作码来控制程序流程。

创建对象和访问方法使用像 `new` 和 `invokespecial` 这样的代码。当我们查看用于调用 lambda 表达式的不同操作码时，我们将特别关注这个组。

最后一组是关于算术、逻辑和类型转换的，包括像 `iadd`、float compare long (`fcmpl`) 和 integer to byte (`i2b`) 这样的代码。

# 描述符

操作码通常会使用参数，它们在字节码中看起来有点晦涩，因为它们通常是通过查找表引用的。在内部，Java 使用所谓的 *描述符* 来描述这些参数。

它们使用特定的语法描述类型和签名，你将在整个字节码中看到这种语法。你经常会在编译器或调试输出中看到相同的语法，所以在这里回顾一下是有用的。

这是一个方法签名描述符的示例。

```java
Example$1."<init>":(Lcom/foo/Example;Lcom/foo/Server;)V
```

它描述了一个名为 `$1` 的类的构造函数，我们碰巧知道这是 JVM 对另一个类内第一个匿名类实例的命名。在这种情况下是 `Example`。所以我们有一个匿名类的构造函数，它接受两个参数，一个是外部类 `com.foo.Example` 的实例，另一个是 `com.foo.Server` 的实例。

作为构造函数，该方法不返回任何东西。`V` 符号表示 void。

看一下下面的描述符语法的分解。如果在描述符中看到大写的 `Z`，它指的是布尔值，大写的 `B` 指的是字节，依此类推。

![描述符](img/descriptors.jpg)

还有一些需要提到的：

+   类使用大写的 `L` 描述，后面跟着完全限定的类名，然后是一个分号。类名使用斜杠而不是点来分隔。

+   数组使用一个开放的方括号后跟列表中的类型来描述。没有闭合方括号。

# 转换方法签名

让我们将以下方法签名转换为方法描述符：

```java
long f (int n, String s, int[] array);
```

该方法返回一个 `long`，所以我们用括号描述它是一个方法，返回一个大写的 `J`。

```java
()J
```

第一个参数是 `int` 类型，所以我们使用大写的 `I`。

```java
(I)J
```

下一个参数是一个对象，所以我们使用 `L` 来描述它是一个对象，完全限定名称并用分号关闭。

```java
(ILString;)J
```

最后一个参数是整数数组，所以我们插入数组语法，然后是 `int` 类型：

```java
(ILString;[I)J
```

然后我们完成了。一个 JVM 方法描述符。

# 代码示例

让我们看一下一些示例生成的字节码。

我们将根据我们在“lambda vs 闭包”部分中看到的示例，查看四个不同功能块的字节码。

我们将探讨：

1.  一个简单的匿名类。

1.  一个匿名类关闭某些变量（旧样式闭包）。

1.  没有参数的 lambda。

1.  带参数的 lambda。

1.  闭包覆盖某些变量的 lambda（新样式闭包）。

示例字节码是使用`javap`命令行工具生成的。本节仅显示了部分字节码列表，完整的源代码和字节码列表请参见`附录 A`。此外，完全限定的类名已经被缩短以更好地适应页面。

# 示例 1

第一个示例是一个简单的匿名类实例，传递给我们的`waitFor`方法。

```java
public class Example1 {
    // anonymous class
    void example() throws InterruptedException {
        waitFor(new Condition() {
            @Override
            public Boolean isSatisfied() {
                return true;
            }
        });
    }
}
```

如果我们看一下下面的字节码，要注意的是匿名类的一个实例在第 6 行被新建。`#2`是一个查找，其结果显示在注释中。因此，它使用`new`操作码与常量池中的`#2`进行操作，这恰好是匿名类`Example$1`。

```java
void example() throws java.lang.InterruptedException;
    descriptor: ()V 
    flags:
    Code:
      stack=3, locals=1, args_size=1
         0: new #2 // class Example1$1
         3: dup
         4: aload_0
         5: invokespecial #3 // Method Example1$1."":(LExample1;)V
         8: invokestatic #4 // Method WaitFor.waitFor:  
            (LCondition;)V
        11: return
      LineNumberTable:
        line 10: 0
        line 16: 11
      LocalVariableTable:
        Start Length Slot Name Signature
            0     12    0 this LExample1;
    Exceptions: 
      throws java.lang.InterruptedException
```

创建后，使用`invokespecial`在第 9 行调用构造函数。这个操作码用于调用构造方法、私有方法和超类的可访问方法。您可能会注意到方法描述符包括对`Example1`的引用。所有匿名类实例都有对父类的这个隐式引用。

下一步使用`invokestatic`调用我们的`waitFor`方法，传递匿名类在第 10 行。`invokestatic`操作码用于调用静态方法，非常快速，因为它可以直接拨号方法，而不是像在对象层次结构中那样找出要调用的方法。

# 示例 2

`Example 2`类是另一个匿名类，但这次它关闭了`server`变量。这是一种旧式的闭包：

```java
public class Example2 {
    // anonymous class (closure)
    void example() throws InterruptedException {
        Server server = new HttpServer();
        waitFor(new Condition() {
            @Override
            public Boolean isSatisfied() {
                return !server.isRunning();
            }
        });
    }
}
```

字节码与之前类似，只是`Server`类的一个实例被新建（在第 3 行），并且在第 5 行调用了它的构造函数。匿名类`$1`的实例仍然是用`invokespecial`构造的（在第 11 行），但这次它除了调用类的实例外，还将`Server`的实例作为参数。

为了关闭`server`变量，它直接传递给匿名类：

```java
void example() throws java.lang.InterruptedException;
    Code:
       0: new #2 // class Server$HttpServer
       3: dup
       4: invokespecial #3 // Method Server$HttpServer."":()V
       7: astore_1
       8: new #4 // class Example2$1
      11: dup
      12: aload_0
      13: aload_1
      14: invokespecial #5 // Method Example2$1."":   
          (LExample2;LServer;)V
      17: invokestatic #6 // Method WaitFor.waitFor:(LCondition;)V
      20: return
```

# 示例 3

`Example 3`类使用了一个 Java lambda 与我们的`waitFor`方法。这个 lambda 除了返回 true 之外什么也不做。它相当于示例 1。

```java
public class Example3 {
    // simple lambda
    void example() throws InterruptedException {
        waitFor(() -> true);
    }
}
```

这次字节码非常简单。它使用`invokedynamic`操作码在第 3 行创建 lambda，然后将其传递给下一行的`invokestatic`操作码。

```java
 void example() throws java.lang.InterruptedException;
     Code:
        0: invokedynamic #2, 0 // InvokeDynamic #0:isSatisfied:   
           ()LCondition;
        5: invokestatic #3 // Method WaitFor.waitFor:(LCondition;)V
        8: return
```

`invokedynamic`调用的描述符针对`Condition`接口的`isSatisfied`方法（第 3 行）。

我们在这里看不到`invokedynamic`的机制。`invokedynamic`操作码是 Java 7 的新操作码，旨在为 JVM 上的动态语言提供更好的支持。它通过直到运行时才将类型链接到方法来实现这一点。其他“invoke”操作码都在编译时解析类型。

对于 lambda 来说，这意味着占位符方法调用可以像我们刚刚看到的那样放入字节码中，并且实现可以在 JVM 上在运行时完成。

如果我们查看包含常量池的更详细的字节码，我们可以解除查找。例如，如果我们查找编号 2，我们可以看到它引用了`#0`和`#26`。

```java
Constant pool:
   #1 = Methodref #6.#21 // Object."":()V
   #2 = InvokeDynamic #0:#26 // #0:isSatisfied:()LCondition;
   ...
BootstrapMethods:
    0: #23 invokestatic LambdaMetafactory.metafactory:              
            (LMethodHandles$Lookup;LString;
            LMethodType;LMethodType;
            LMethodHandle;LMethodType;)LCallSite;
      Method arguments:
        #24 ()LBoolean;
        #25 invokestatic Example3.lambda$example$25:()LBoolean;
        #24 ()LBoolean;
```

常量`0`在一个特殊的查找表中用于引导方法（第 6 行）。它引用了对 JDK `LambdaMetafactory`的静态方法调用，以创建 lambda。这就是繁重的工作所在。所有目标类型推断以适应类型和任何部分参数评估都在这里进行。

实际的 lambda 显示为一个名为`lambda$example$25`的方法句柄（第 12 行），没有参数，返回一个布尔值。它使用`invokestatic`调用，这表明它被视为一个真正的函数；它没有与之关联的对象。与之前的匿名示例不同，它也没有对包含类的隐式引用。

它被传递到`LambdaMetafactory`，我们知道它是一个方法句柄，通过在常量池中查找它。lambda 的编号由编译器分配，并且对于每个需要的 lambda，它只是从零递增。

```java
Constant pool:
    // invokestatic Example3.lambda$example$25:()LBoolean;
    #25 = MethodHandle #6:#35
```

# 示例 4

`Example 4`类是另一个 lambda，但这次它以`Server`的实例作为参数。它在功能上等同于示例 2，但它不会闭包变量；它不是一个闭包。

```java
public class Example4 {
    // lambda with arguments
    void example() throws InterruptedException {
        waitFor(new HttpServer(), (server) -> server.isRunning());
    }
}
```

就像示例 2 一样，字节码必须创建服务器的实例，但这次，`invokedynamic`操作码引用了`Predicate`类型的`test`方法。如果我们按照引用（`＃4`）到引导方法表，我们会看到实际的 lambda 需要一个`HttpServer`类型的参数，并返回一个`Z`，这是一个原始布尔值。

```java
void example() throws java.lang.InterruptedException;
    descriptor: ()V
    flags:
    Code:
      stack=2, locals=1, args_size=1
         0: new #2 // class Server$HttpServer
         3: dup
         4: invokespecial #3 // Method Server$HttpServer."":()V
         7: invokedynamic #4, 0 // InvokeDynamic #0:test:  
            ()LPredicate;
        12: invokestatic #5 // Method WaitFor.waitFor:     
            (LObject;LPredicate;)V
        15: return
      LineNumberTable:
        line 13: 0
        line 15: 15
      LocalVariableTable:
        Start Length Slot Name Signature
            0     16    0 this LExample4;
    Exceptions:
      throws java.lang.InterruptedException
```

因此，对 lambda 的调用仍然是静态方法调用，但这次在调用时将变量作为参数传递。

# 示例 4（使用方法引用）

有趣的是，如果我们使用方法引用，功能完全相同，但得到的字节码不同。

```java
public class Example4_method_reference {
    // lambda with method reference
    void example() throws InterruptedException {
        waitFor(new HttpServer(), HttpServer::isRunning);
    }
}
```

通过对`LambdaMetafactory`的调用，当最终执行发生时，`method_reference`会导致调用`invokevirtual`而不是`invokestatic`。`invokevirtual`操作码用于调用公共、受保护和包保护方法，因此需要一个实例。实例由`metafactory`方法提供，根本不需要 lambda（或静态函数）；在这个字节码中没有`lambda$`。

```java
void example() throws java.lang.InterruptedException;
    descriptor: ()V
    flags:
    Code:
      stack=2, locals=1, args_size=1
         0: new #2 // class Server$HttpServer
         3: dup
         4: invokespecial #3 // Method Server$HttpServer."":()V
         7: invokedynamic #4, 0 // InvokeDynamic #0:test:   
            ()LPredicate;
        12: invokestatic #5 // Method WaitFor.waitFor:
            (LObject;LPredicate;)V
        15: return
      LineNumberTable:
        line 11: 0
        line 12: 15
      LocalVariableTable:
        Start Length Slot Name Signature
            0     16    0 this LExample4_method_reference;
    Exceptions:
      throws java.lang.InterruptedException
```

# 示例 5

最后，示例 5 使用 lambda，但是闭包了服务器实例。它等同于示例 2，是一种新的样式闭包。

```java
public class Example5 {
    // closure
    void example() throws InterruptedException {
        Server server = new HttpServer();
        waitFor(() -> !server.isRunning());
    }
}
```

它以与其他 lambda 相同的方式介绍了基础知识，但是如果在引导方法表中查找`metafactory`方法，您会注意到这次 lambda 的方法句柄具有`Server`类型的参数。它使用`invokestatic`（第 9 行）调用，并且变量在调用时直接传递到 lambda 中。

```java
BootstrapMethods:
    0: #34 invokestatic LambdaMetafactory.metafactory:
            (LMethodHandles$Lookup;
             LString;LMethodType;
             LMethodType;
             LMethodHandle;LMethodType;)LCallSite;
      Method arguments:
        #35 ()LBoolean; // <-- SAM method to be implemented by the        
            lambda
        #36 invokestatic Example5.lambda$example$35:
          (LServer;)LBoolean;
        #35 ()LBoolean; // <-- type to be enforced at invocation   
        time
```

因此，就像示例 2 中的匿名类一样，编译器添加了一个参数来捕获该术语，但这次是方法参数而不是构造函数参数。

# 总结

我们看到使用匿名类将创建一个新实例，并使用`invokespecial`调用它的构造函数。

我们看到闭包变量的匿名类在其构造函数上有一个额外的参数，用于捕获该变量。

我们看到 Java lambda 使用`invokedynamic`指令来推迟类型的绑定，并且特殊的`lambda$`方法句柄用于实际表示 lambda。在这种情况下，该方法句柄没有参数，并且使用`invokestatic`调用它，使其成为真正的函数。

lambda 是由`LambdaMetafactory`类创建的，它本身是`invokedynamic`调用的目标。

当 lambda 有参数时，我们看到`LambdaMetafactory`描述要传递到 lambda 中的参数。`invokestatic`操作码用于执行 lambda，就像以前一样。但我们还看了一个方法引用，代替 lambda。在这种情况下，没有创建`lambda$`方法句柄，而是使用`invokevirtual`直接调用方法。

最后，我们看了一个闭包变量的 lambda。这个创建了一个`lambda$`方法句柄上的参数，并且再次使用`invokestatic`调用。
