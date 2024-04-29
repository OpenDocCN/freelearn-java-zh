# 第二章：lambda 介绍

在本章中，我们将介绍 lambda 的概念，我们将：

+   讨论一些关于 lambda 和函数式编程的背景知识

+   讨论 Java 中函数与类的区别

+   看一下 Java 中 lambda 的基本语法

# 在函数式编程中的λs

在我们更深入地看问题之前，让我们先看一些关于 lambda 的一般背景。

如果你以前没有见过，希腊字母λ（**lambda**）在谈论 lambda 时经常被用作简写。

## 20 世纪 30 年代和 lambda 演算

在计算机科学中，lambda 演算可以追溯到 lambda 演算。这是 20 世纪 30 年代由**Alonzo Church**引入的函数的数学表示法。这是一种使用函数探索数学的方法，后来被重新发现为计算机科学中的有用工具。

它正式化了*lambda 项*的概念和转换这些项的规则。这些规则或*函数*直接映射到现代计算机科学的思想。lambda 演算中的所有函数都是匿名的，这在计算机科学中再次被字面理解。

这是一个 lambda 演算表达式的例子：

**一个 lambda 演算表达式**

```java
 λx.x+1
```

这定义了一个带有单个参数`x`的匿名函数或*lambda*。主体跟在点后，将一个加到该参数。

## 20 世纪 50 年代和 LISP

在 20 世纪 50 年代，*John McCarthy*在麻省理工学院发明了 LISP。这是一种旨在模拟数学问题的编程语言，受到了 lambda 演算的重大影响。

它使用 lambda 这个词作为运算符来定义一个匿名函数。

这是一个例子：

**一个 LISP 表达式**

```java
(lambda (arg) (+ arg 1))
```

这个 LISP 表达式求值为一个函数，当应用时将接受一个参数，将其绑定到`arg`，然后加`1`。

这两个表达式产生相同的结果，一个增加数字的函数。你可以看到这两个非常相似。

lambda 演算和 LISP 对函数式编程产生了巨大影响。应用函数和使用函数推理问题的思想直接转移到了编程语言中。因此，在我们的领域中使用了这个术语。演算中的 lambda 与现代编程语言中的 lambda 是相同的，并且以相同的方式使用。

## 什么是 lambda

简而言之，lambda 只是一个匿名函数。就是这样。没有什么特别的。这只是一种定义函数的简洁方式。当您想要传递可重用功能片段时，匿名函数是有用的。例如，将函数传递到其他函数中。

许多主流语言已经支持 lambda，包括 Scala、C#、Objective-C、Ruby、C++(11)、Python 等。

# 函数与类

请记住，Java 中的匿名*函数*与 Java 中的匿名*类*不同。Java 中的匿名类仍然需要实例化为对象。它可能没有一个适当的名称，但只有当它是一个对象时才有用。

另一方面，*函数*没有与之关联的实例。函数与它们所作用的数据是分离的，而对象与它所作用的数据是密切相关的。

您可以在现代 Java 中使用 lambda 的任何地方，以前您可能使用单方法接口，因此它可能看起来只是语法糖，但实际上并非如此。让我们看看它们的区别，并将匿名类与 lambda 进行比较；类与函数。

## 现代 Java 中的 lambda

在 Java 8 之前，匿名类（单方法接口）的典型实现可能看起来像这样。`anonymousClass`方法调用`waitFor`方法，传入`Condition`的一些实现；在这种情况下，它在说，等待某个服务器关闭：

**匿名类的典型用法**

```java
void anonymousClass() {
    final Server server = new HttpServer();
    waitFor(new Condition() {
        @Override
        public Boolean isSatisfied() {
            return !server.isRunning();
        }
    });
}

```

等效的 lambda 看起来是这样的：

**等效的 lambda 功能**

```java
 void closure() { 
     Server server = new HttpServer();
     waitFor(() -> !server.isRunning()); 
 }
```

在完整性的利益上，一个天真的轮询`waitFor`方法可能看起来像这样：

```java
class WaitFor {
    static void waitFor(Condition condition) throws   
    InterruptedException {
        while (!condition.isSatisfied())
            Thread.sleep(250);
    }
}

```

## 一些理论上的差异

首先，这两种实现实际上都是闭包，后者也是 lambda。我们将在*Lambda 与闭包*部分更详细地讨论这一区别。这意味着它们都必须在运行时捕获它们的“环境”。在 Java 8 之前，这意味着将闭包需要的东西复制到一个类的实例中（Condition 的匿名实例）。在我们的例子中，server 变量需要被复制到实例中。

因为它是一个副本，必须声明为 final，以确保它在被捕获和使用时不能被更改。这两个时间点可能非常不同，因为闭包通常用于推迟执行直到以后的某个时间点（例如，参见[惰性求值](http://en.wikipedia.org/wiki/Lazy_evaluation)）。现代 Java 使用了一个巧妙的技巧，即如果它可以推断出一个变量永远不会被更新，那么它可能就是 final，因此它被视为*实际上是 final*，你不需要显式地声明它为 final。

另一方面，lambda 不需要复制它的环境或捕获任何术语。这意味着它可以被视为真正的函数，而不是一个类的实例。有什么区别？很多。

## 函数与类

首先，函数；[真正的函数](http://en.wikipedia.org/wiki/Pure_function)，不需要多次实例化。当谈论分配内存和加载一块机器代码作为函数时，我甚至不确定实例化是否是正确的词。关键是，一旦它可用，它可以被重复使用，它的本质是幂等的，因为它不保留状态。静态类方法是 Java 中最接近函数的东西。

对于 Java 来说，这意味着 lambda 在每次评估时不需要被实例化，这是一个很大的问题。与实例化匿名类不同，内存影响应该是最小的。

在一些概念上的区别方面：

+   类必须被实例化，而函数不需要。

+   当类被实例化时，内存被分配给对象。

+   函数只需要分配一次内存。它们存储在堆的*永久*区域。

+   对象作用于它们自己的数据，函数作用于不相关的数据。

+   在 Java 中，静态类方法大致相当于函数。

## 一些具体的区别

函数和类之间的一些具体区别包括它们的捕获语义以及它们如何遮蔽变量。

### 捕获语义

另一个区别在于对 this 的捕获语义。在匿名类中，this 指的是匿名类的实例。例如，`Foo$InnerClass`而不是`Foo`。这就是为什么当你从匿名类引用封闭范围时，会出现略显奇怪的语法，比如`Foo.this.x`。

另一方面，在 lambda 中，this 指的是封闭范围（在我们的例子中直接指的是 Foo）。事实上，lambda 是**完全词法作用域**，这意味着它们不继承任何名称来自超类型，也不引入新的作用域级别；你可以直接访问封闭范围的字段、方法和局部变量。

例如，这个类表明 lambda 可以直接引用`firstName`变量。

```java
public class Example {
    private String firstName = "Jack";

    public void example() {
        Function<String, String> addSurname = surname -> {
            // equivalent to this.firstName
            return firstName + " " + surname;  // or even,   
            this.firstName
        };
    }
}
```

在这里，`firstName`是`this.firstName`的简写，因为 this 指的是封闭范围（类`Example`），它的值将是“Jack”。

匿名类等效需要明确地引用来自封闭范围的`firstName`。在这种情况下，你不能使用 this，因为这意味着匿名实例，那里没有`firstName`。因此，以下内容将编译：

```java
public class Example {
    private String firstName = "Charlie";

    public void anotherExample() {
        Function<String, String> addSurname = new Function<String,  
        String>() {
            @Override
            public String apply(String surname) {
                return Example.this.firstName + " " + surname;   
                // OK
            }
        };
    }
}
```

但这不会。

```java
public class Example {
    private String firstName = "Charlie";

  public void anotherExample() {
    Function<String, String> addSurname = new Function<String,   
    String>() {
      @Override
      public String apply(String surname) {
        return this.firstName + " " + surname;   // compiler error
      }
    };
  }
}
```

你仍然可以直接访问字段（即简单地调用`return firstName + " " + surname`），但不能使用 this。这里的重点是演示在 lambda 和匿名实例中使用 this 时捕获语义的差异。

### 遮蔽变量

引用遮蔽变量变得更加直接，可以通过简化的`this`语义来推理。例如，

```java
public class ShadowingExample {

    private String firstName = "Charlie";

    public void shadowingExample(String firstName) {
        Function<String, String> addSurname = surname -> {
            return this.firstName + " " + surname;
        };
    }
}
```

在这里，因为`this`在 lambda 内部，它指的是外部范围。所以`this.firstName`将有值`"Charlie"`，而不是同名方法参数的值。捕获语义使其更清晰。如果你使用`firstName`（并且去掉`this`），它将指的是参数。

在下一个示例中，使用匿名实例，`firstName`只是指参数。如果你想引用外部版本，你会使用`Example.this.firstName`：

```java
public class ShadowingExample {

    private String firstName = "Charlie";

    public void anotherShadowingExample(String firstName) {
        Function<String, String> addSurname = new Function<String,  
        String>() {
            @Override
            public String apply(String surname) {
                return firstName + " " + surname;
            }
        };
    }
}
```

## 总结

学术意义上的函数与匿名类（我们在 Java 8 之前经常将其视为函数）是非常不同的东西。了解这些区别是有用的，可以为了除了简洁的语法之外的其他原因来使用 lambda。当然，使用 lambda 还有很多其他优势（至少是 JDK 的大量使用）。

当我们看下新的 lambda 语法时，请记住，尽管 lambda 在 Java 中的使用方式与匿名类非常相似，但它们在技术上是不同的。Java 中的 lambda 不需要在每次评估时实例化，不像匿名类的实例。

这应该提醒你，Java 中的 lambda 不仅仅是语法糖。

# λ基本语法

让我们来看一下基本的 lambda 语法。

lambda 基本上是一个匿名的功能块。它很像使用匿名类实例。例如，如果我们想在 Java 中对数组进行排序，我们可以使用`Arrays.sort`方法，它接受`Comparator`接口的实例。

它会看起来像这样：

```java
Arrays.sort(numbers, new Comparator<Integer>() {
    @Override
    public int compare(Integer first, Integer second) {
        return first.compareTo(second);
    }
});
```

这里的`Comparator`实例是功能的一个抽象部分；它本身没有意义；只有当它被`sort`方法使用时才有目的。

使用 Java 的新语法，你可以用 lambda 替换它，看起来像这样：

```java
Arrays.sort(numbers, (first, second) -> first.compareTo(second));
```

这是一种更简洁的实现相同功能的方式。事实上，Java 将其视为`Comparator`类的实例。如果我们要提取 lambda 的变量（第二个参数），它的类型将是`Comparator<Integer>`，就像上面的匿名实例一样。

```java
Comparator<Integer> ascending = (first, second) -> first.compareTo(second);
Arrays.sort(numbers, ascending);
```

因为`Comparator`上只有一个抽象方法；`compareTo`，编译器可以推断出当我们有一个这样的匿名块时，我们实际上是指`Comparator`的一个实例。它可以做到这一点，多亏了我们稍后将讨论的一些其他新特性；函数接口和类型推断的改进。

## 语法分解

你总是可以从使用单个抽象方法转换为使用 lambda。

假设我们有一个接口`Example`，有一个方法`apply`，返回某种类型并接受某个参数：

```java
interface Example {
     R apply(A arg);
}
```

我们可以用这样的方式实例化一个实例：

```java
new Example() {
    @Override
    public R apply(A args) {
        body
    }
};
```

要转换为 lambda，我们基本上是去掉多余的部分。我们去掉实例化和注释，去掉方法细节，只留下参数列表和主体。

```java
(args) {
    body
}
```

然后我们引入新的箭头符号来表示整个内容是 lambda，并且接下来的部分是 lambda 的主体，这就是我们的基本 lambda 语法：

```java
(args) -> {
    body
}
```

让我们通过以下步骤来看之前的排序示例。我们从匿名实例开始：

```java
Arrays.sort(numbers, new Comparator<Integer>() {
    @Override
    public int compare(Integer first, Integer second) {
        return first.compareTo(second);
    }
});
```

并去掉实例化和方法签名：

```java
Arrays.sort(numbers, (Integer first, Integer second) {
    return first.compareTo(second);
});
```

引入 lambda

```java
Arrays.sort(numbers, (Integer first, Integer second) -> {
    return first.compareTo(second);
});
```

然后我们完成了。不过我们还可以做一些优化。如果编译器足够聪明以推断类型，你可以省略类型。

```java
Arrays.sort(numbers, (first, second) -> {
    return first.compareTo(second);
});
```

对于简单的表达式，你可以去掉大括号来产生一个 lambda 表达式：

```java
Arrays.sort(numbers, (first, second) -> first.compareTo(second));
```

在这种情况下，编译器可以推断出足够的信息来知道你的意思。单个语句返回一个与接口一致的值，所以它说，“不需要告诉我你要返回什么，我自己就能看到”。

对于单参数接口方法，甚至可以去掉第一个括号。例如，接受参数`x`并返回`x + 1`的 lambda；

```java
(x) -> x + 1
```

可以不带括号写

```java
x -> x + 1
```

## 总结

让我们用一个语法选项的总结来回顾一下。

语法总结：

```java
(int x, int y) -> { return x + y; }
(x, y) -> { return x + y; }
(x, y) -> x + y; x -> x * 2
() -> System.out.println("Hey there!");
System.out::println;
```

第一个例子 `((int x, int y) -> { return x + y; })` 是创建 lambda 的最冗长的方式。函数的参数及其类型在括号内，然后是新的箭头语法，然后是代码块；要执行的代码块。

您通常可以从参数列表中省略类型，比如 `(x, y) -> { return x + y; }`。编译器会在这里使用类型推断来尝试猜测类型。它会根据您尝试使用 lambda 的上下文来进行推断。

如果您的代码块返回某些内容或是单行表达式，您可以省略大括号和返回语句，例如 `(x, y) -> x + y;`。

在只有一个参数的情况下，您可以省略括号 `x -> x * 2`。

如果根本没有参数，则需要使用“汉堡包”符号。

`() -> System.out.println("Hey there!");`。

为了完整起见，还有另一种变体；一种称为*方法引用*的 lambda 的快捷方式。例如，`System.out::println;`，基本上是对 lambda `(value -> System.out.prinltn(value)` 的简写。

我们将在稍后更详细地讨论方法引用，所以现在只需知道它们存在，并且可以在任何可以使用 lambda 的地方使用。
