# 第十一章：控制流语句

本章描述了一种特定类型的 Java 语句，称为控制语句，它允许根据实现的算法的逻辑构建程序流程，其中包括选择语句、迭代语句、分支语句和异常处理语句。

在本章中，我们将涵盖以下主题：

+   什么是控制流？

+   选择语句：`if`、`if....else`、`switch...case`

+   迭代语句：`for`、`while`、`do...while`

+   分支语句：`break`、`continue`、`return`

+   异常处理语句：`try...catch...finally`、`throw`、`assert`

+   练习-无限循环

# 什么是控制流？

Java 程序是一系列可以执行并产生一些数据或/和启动一些操作的语句。为了使程序更通用，一些语句是有条件执行的，根据表达式评估的结果。这些语句称为控制流语句，因为在计算机科学中，控制流（或控制流）是执行或评估单个语句的顺序。

按照惯例，它们被分为四组：选择语句、迭代语句、分支语句和异常处理语句。

在接下来的章节中，我们将使用术语块，它表示一系列用大括号括起来的语句。这是一个例子：

```java

{

x = 42;

y = method(7, x);

System.out.println("Example");

}

```

一个块也可以包括控制语句-一个娃娃里面的娃娃，里面的娃娃，依此类推。

# 选择语句

选择语句组的控制流语句基于表达式的评估。例如，这是一种可能的格式：`if(expression) do something`。或者，另一种可能的格式：`if(expression) {do something} else {do something else}`。

表达式可能返回一个`boolean`值（如前面的例子）或一个可以与常量进行比较的特定值。在后一种情况下，选择语句的格式为`switch`语句，它执行与特定常量值相关联的语句或块。

# 迭代语句

迭代语句执行某个语句或块，直到达到某个条件。例如，它可以是一个`for`语句，它执行一个语句或一组值的集合的每个值，或者直到某个计数器达到预定义的阈值，或者达到其他某些条件。执行的每个循环称为迭代。

# 分支语句

分支语句允许中断当前执行流程并从当前块后的第一行继续执行，或者从控制流中的某个（标记的）点继续执行。

方法中的`return`语句也是分支语句的一个例子。

# 异常处理语句

异常是表示程序执行过程中发生的事件并中断正常执行流程的类。我们已经看到了在相应条件下生成的`NullPointerException`、`ClassCastException`和`ArrayIndexOutOfBoundsException`的示例。

Java 中的所有异常类都有一个共同的父类，即`java.lang.Exception`类，它又扩展了`java.lang.Throwable`类。这就是为什么所有异常对象都有共同的行为。它们包含有关异常条件的原因和其起源位置（类源代码的行号）的信息。

每个异常都可以被自动（由 JVM）抛出，或者由应用程序代码使用`throw`关键字。方法调用者可以使用异常语句捕获异常，并根据异常类型和它（可选地）携带的消息执行某些操作，或者让异常进一步传播到方法调用堆栈的更高层。

如果堆栈中的应用程序方法都没有捕获异常，最终将由 JVM 捕获异常，并用错误中止应用程序执行。

因此，异常处理语句的目的是生成（`throw`）和捕获异常。

# 选择语句

选择语句有四种变体：

+   `if`语句

+   `if...else`语句

+   `if...else if-...-else`语句

+   `switch...case`语句

# if

简单的`if`语句允许有条件地执行某个语句或块，仅当表达式求值结果为`true`时：

```java

if(booelan expression){

//做一些事情

}

```

以下是一些例子：

```java

if(true) System.out.println("true");    //1: true

if(false) System.out.println("false");  //2:

int x = 1, y = 5;

if(x > y) System.out.println("x > y");  //3:

if(x < y) System.out.println("x < y");  //4: x < y

if((x + 5) > y) {                       //5: x + 5 > y

System.out.println("x + 5 > y");

x = y;

}

if(x == y){                             //6: x == y

System.out.println("x == y");

}

```

语句 1 打印`true`。语句 2 和 3 什么也不打印。语句 4 打印`x < y`。语句 5 打印`x + 5 > y`。我们使用大括号创建了一个块，因为我们希望`x = y`语句仅在此`if`语句的表达式求值为`true`时执行。语句 6 打印`x == y`。我们可以避免在这里使用大括号，因为只有一个语句需要执行。我们这样做有两个原因：

+   为了证明大括号也可以与单个语句一起使用，从而形成一个语句块。

+   良好的实践是，在`if`后面总是使用大括号`{}`；这样读起来更好，并有助于避免这种令人沮丧的错误：在`if`后添加另一个语句，假设它只在表达式返回`true`时执行：

```java

if(x > y) System.out.println("x > y");

x = y;

```

但是，此代码中的语句`x = y`是无条件执行的。如果您认为这种错误并不经常发生，您会感到惊讶。

始终在`if`语句后使用大括号`{}`是一个好习惯。

正如我们已经提到的，可以在选择语句内包含选择语句，以创建更精细的控制流逻辑：

```java

if(x > y){

System.out.println("x > y");

if(x == 3){

System.out.println("x == 3");

}

if(y == 3){

System.out.println("y == 3");

System.out.println("x == " + x);

}

}

```

它可以根据逻辑要求深入（嵌套）。

# if...else

`if...else`结构允许在表达式求值为`true`时执行某个语句或块；否则，将执行另一个语句或块：

```java

if(Boolean expression){

//做一些事情

} else {

//做一些其他事情

}

```

以下是两个例子：

```java

int x = 1, y = 1;

if(x == y){

System.out.println("x == y");  //打印：x == y

x = y - 1;

} else {

System.out.println("x != y");

}

if(x == y){

System.out.println("x == y");

} else {

System.out.println("x != y");  //打印：x != y

}

```

当大括号`{}`被一致使用时，您可以看到阅读此代码有多容易。并且，就像简单的`if`语句的情况一样，每个块都可以有另一个嵌套块，其中包含另一个`if`语句，依此类推 - 可以有多少块和多么深的嵌套。

# if...else if-...-else

您可以使用此形式来避免创建嵌套块，并使代码更易于阅读和理解。例如，看下面的代码片段：

```java

if(n > 5){

System.out.println("n > 5");

} else {

if (n == 5) {

System.out.println("n == 5");

} else {

if (n == 4) {

System.out.println("n == 4");

} else {

System.out.println("n < 4");

}

}

}

}

```

这些嵌套的`if...else`语句可以被以下`if...else...if`语句替换：

```java

if(n > 5){

System.out.println("n > 5");

} else if (n == 5) {

System.out.println("n == 5");

} else if (n == 4) {

System.out.println("n == 4");

} else {

System.out.println("n < 4");

}

```

这样的代码更容易阅读和理解。

如果`n < 4`时不需要执行任何操作，则可以省略最后的`else`子句：

```java

if(n > 5){

System.out.println("n > 5");

} else if (n == 5) {

System.out.println("n == 5");

} else if (n == 4) {

System.out.println("n == 4");

}

```

如果您需要针对每个特定值执行某些操作，可以编写如下：

```java

if(x == 5){

//做一些

} else if (x == 7) {

//做一些其他事情

} else if (x == 12) {

//做一些不同的事情

} else if (x = 50) {

//做一些不同的事情

} else {

//做一些完全不同的事情

}

```

但是，对于这种情况有一个专门的选择语句，称为`switch...case`，更容易阅读和理解。

# switch...case

上一节的代码示例可以表示为`switch`语句，如下所示：

```java

switch(x){

case 5:

//做一些

break;

case 7:

//做一些其他事情

break;

case 12:

//做一些不同的事情

break;

case 50:

//做一些不同的事情

break;

default:

//做一些完全不同的事情

}

```

返回`x`变量值的表达式的类型可以是`char`、`byte`、`short`、`int`、`Character`、`Byte`、`Short`、`Integer`、`String`或`enum`类型。注意`break`关键字。它强制退出`switch...case`语句。如果没有它，接下来的语句`do something`将被执行。我们将在*分支语句*部分后面讨论`break`语句。

可以在`switch`语句中使用的类型有`char`、`byte`、`short`、`int`、`Character`、`Byte`、`Short`、`Integer`、`String`和`enum`类型。在 case 子句中设置的值必须是常量。

让我们看一个利用`switch`语句的方法：

```java

void switchDemo(int n){

switch(n + 1){

case 1:

System.out.println("case 1: " + n);

break;

case 2:

System.out.println("case 2: " + n);

break;

default:

System.out.println("default: " + n);

break;

}

}

```

以下代码演示了`switch`语句的工作原理：

```java

switchDemo(0);     //打印：case1: 0

switchDemo(1);     //打印：case2: 1

switchDemo(2);     //打印：默认：2

```

与`if`语句中的`else`子句类似，如果在程序逻辑中不需要`switch`语句中的默认子句，则默认子句是不需要的：

```java

switch(n + 1){

case 1:

System.out.println("case 1: " + n);

break;

case 2:

System.out.println("case 2: " + n);

}

```

# 迭代语句

迭代语句对于 Java 编程和选择语句一样重要。您很有可能经常看到并使用它们。每个迭代语句可以是`while`、`do...while`或`for`中的一种形式。

# while

`while`语句执行布尔表达式和语句或块，直到表达式的值评估为`false`：

```java

while (布尔表达式){

//做一些

}

```

有两件事需要注意：

+   当只有一个语句需要重复执行时，大括号`{}`是不必要的，但为了一致性和更好的代码理解，建议使用。

+   该语句可能根本不会执行（当第一个表达式评估为`false`时）

让我们看一些示例。以下循环执行打印语句五次：

```java

int i = 0;

while(i++ < 5){

System.out.print(i + " ");   //打印：1 2 3 4 5

}

```

注意使用的不同的打印方法：`print()`而不是`println()`。后者在打印行之后添加了一个转义序列`\n`（我们已经解释了转义序列是什么，位于第五章，*Java 语言元素和类型*），它将光标移动到下一行。

以下是调用返回某个值并累积直到达到所需阈值的方法的示例：

```java

double result = 0d;

while (result < 1d){

result += tryAndGetValue();

System.out.println(result);

}

```

`tryAndGetValue()` 方法非常简单和不切实际，只是为了演示目的而编写的：

```java

double tryAndGetValue(){

return Math.random();

}

```

如果我们运行最后一个 `while` 语句，我们将看到类似于以下内容：

![](img/fab4ec67-c60b-4037-a459-37af35dbc04d.png)

确切的值会因运行而异，因为 `Math.random()` 方法生成大于或等于 0.0 且小于 1.0 的伪随机 `double` 值。一旦累积值等于 1.0 或超过 1.0，循环就会退出。

让这个循环变得更简单是很诱人的：

```java

double result = 0d;

while ((result += tryAndGetValue()) < 1d){

System.out.println(result);

}

```

甚至更简单：

```java

double result = 0d;

while ((result += Math.random()) < 1d){

System.out.println(result);

}

```

但如果我们运行最后两个 `while` 语句的变体中的任何一个，我们将得到以下内容：

![](img/08d31c48-80e3-42d0-89ce-c6634ee01f5d.png)

打印的值永远不会等于或超过 1.0，因为新累积值的表达式在进入执行块之前被评估。当计算包含在表达式中而不是在执行块中时，这是需要注意的事情。

# do...while

类似于 `while` 语句，`do...while` 语句重复执行布尔表达式和语句或块，直到布尔表达式的值评估为 `false`：

```java

do {

//语句或块

} while (Boolean expression)

```

但它在评估表达式之前首先执行语句或块，这意味着语句或块至少会被执行一次。

让我们看一些例子。以下代码执行打印语句六次（比类似的 `while` 语句多一次）：

```java

int i = 0;

do {

System.out.print(i + " ");   //打印：0 1 2 3 4 5

} while(i++ < 5);

```

以下代码的行为与 `while` 语句相同：

```java

double result = 0d;

do {

result += tryAndGetValue();

System.out.println(result);

} while (result < 1d);

```

如果我们运行此代码，我们将看到类似于以下内容：

![](img/7957ae0e-88ff-4d36-a4b9-9df736c28d38.png)

这是因为值在累积后被打印，然后在再次进入执行块之前评估表达式。

简化的 `do...while` 语句的行为不同。以下是一个例子：

```java

double result = 0d;

do {

System.out.println(result);

} while ((result += tryAndGetValue()) < 1d);

```

这里是相同的代码，但没有使用 `tryAndGetValue()` 方法：

```java

double result = 0d;

do {

System.out.println(result);

} while ((result += Math.random()) < 1d);

```

如果我们运行前两个示例中的任何一个，我们将得到以下截图中的内容：

![](img/da256ca8-5d1a-4fe6-8b10-20d1f22b757a.png)

`result` 变量的初始值总是首先打印，因为在第一次评估表达式之前至少执行一次该语句。

# for

基本 `for` 语句的格式如下：

```java

for(ListInit; Boolean Expression; ListUpdate) block or statement

```

但是，我们将从最流行的、更简单的版本开始，并在稍后的*带有多个初始化器和表达式的 For*部分回到完整版本。更简单的基本 `for` 语句格式如下：

```java

for(DeclInitExpr; Boolean Expression; IncrDecrExpr) block or statement

```

这个定义由以下组件组成：

+   `DeclInitExpr` 是一个声明和初始化表达式，比如 `x = 1`，它只在 `for` 语句执行的最开始被评估一次

+   Boolean Expression 是一个布尔表达式，比如 `x < 10`，它在每次迭代开始时被评估 - 在执行块或语句之前每次都会被评估；如果结果是 `false`，`for` 语句就会终止

+   `IncrDecrExpr`是增量或递减一元表达式，如`++x`、`--x`、`x++`、`x-`，它在每次迭代结束后评估-在执行块或语句后

请注意，我们谈论的是表达式，而不是语句，尽管添加了分号，它们看起来像语句。这是因为分号在`for`语句中作为表达式之间的分隔符。让我们看一个例子：

```java

for (int i=0; i < 3; i++){

System.out.print(i + " ");  //输出：0 1 2

}

```

在这段代码中：

+   `int i=0`是声明和初始化表达式，仅在一开始时评估一次

+   `i < 3`是布尔表达式，在每次迭代开始时评估-在执行块或语句之前；如果结果为`false`（在这种情况下为`i >= 3`），则`for`语句的执行终止

+   `i++`是增量表达式，在执行块或语句后评估

并且，与`while`语句的情况一样，当只有一个语句需要执行时，大括号`{}`是不需要的，但最好有它们，这样代码就更一致，更容易阅读。

`for`语句中的任何表达式都不是必需的：

```java

int k = 0;

for (;;){

System.out.print(k++ + " ");     //输出：0 1 2

if(k > 2) break;

}

```

但在语句声明中使用表达式更方便和常规，因此更容易理解。以下是其他示例：

```java

for (int i=0; i < 3;){

System.out.print(i++ + " "); //输出：0 1 2

}

for (int i=2; i > 0; i--){

System.out.print(i + " "); //输出：2 1

}

```

请注意，在最后一个示例中，递减运算符用于减小初始`i`值。

在使用`for`语句或任何迭代语句时，请确保达到退出条件（除非您故意创建无限循环）。这是迭代语句构建的主要关注点。

# 用于增强

正如我们已经提到的，`for`语句是访问数组组件（元素）的一种非常方便的方式：

```java

int[] arr = {21, 34, 5};

for (int i=0; i < arr.length; i++){

System.out.print(arr[i] + " ");  //输出：21 34 5

}

```

注意我们如何使用数组对象的公共属性`length`来确保我们已经到达了所有的数组元素。但在这种情况下，当需要遍历整个数组时，最好（更容易编写和阅读）使用以下格式的增强`for`语句：

```java

<Type> arr = ...;              //数组或任何 Iterable

for (<Type> a: arr){

System.out.print(a + " ");

}

```

从注释中可以看出，它适用于数组或实现接口`Iterable`的类。该接口具有一个`iterator()`方法，返回一个`Iterator`类的对象，该类又有一个名为`next()`的方法，允许按顺序访问类成员。我们将在第十三章中讨论这样的类，称为集合，*Java 集合*。因此，我们可以重写最后的`for`语句示例并使用增强的`for`语句：

```java

int[] arr = {21, 34, 5};

for (int a: arr){

System.out.print(a + " ");  //输出：21 34 5

}

```

对于实现`List`接口（`List`扩展`Iterable`）的集合类，对其成员的顺序访问看起来非常相似：

```java

List<String> list = List.of("Bob", "Joe", "Jill");

for (String s: list){

System.out.print(s + " ");  //输出：Bob Joe Jill

}

```

但是，当不需要访问数组或集合的所有元素时，可能有其他形式的迭代语句更适合。

另外，请注意，自 Java 8 以来，许多数据结构可以生成流，允许编写更紧凑的代码，并且完全避免使用`for`语句。我们将在第十八章中向您展示如何做到这一点，*流和管道*。

# 用于多个初始化程序和表达式

现在，让我们再次回到基本的`for`语句格式。它允许使用的变化比许多程序员知道的要多得多。这不是因为缺乏兴趣或专业好奇心，而可能是因为通常不需要这种额外的功能。然而，偶尔当你阅读别人的代码或在面试中，你可能会遇到需要了解全貌的情况。因此，我们决定至少提一下。

`for`语句的完整格式建立在表达式列表周围：

```java

for(ListInit; Boolean Expression; ListUpdate) block or statement

```

这个定义由以下组件组成：

+   `ListInit`: 可包括声明列表和/或表达式列表

+   `Expression`: 布尔表达式

+   `ListUpdate`: 表达式列表

表达式列表成员，用逗号分隔，可以是：

+   **赋值**：`x = 3`

+   **前/后缀递增/递减表达式**：`++x`  `--x`  `x++`  `x--`

+   **方法调用**：`method(42)`

+   **对象创建表达式**：`new SomeClass(2, "Bob")`

以下两个`for`语句产生相同的结果：

```java

for (int i=0, j=0; i < 3 && j < 3; ++i, ++j){

System.out.println(i + " " + j);

}

for (int x=new A().getInitialValue(), i=x == -2 ? x + 2 : 0, j=0;

i < 3 || j < 3 ; ++i, j = i) {

System.out.println(i + " " + j);

}

```

`getInitialValue()`方法的代码如下：

```java

class A{

int getInitialValue(){ return -2; }

}

```

正如你所看到的，即使是这样一个简单的功能，当过多地使用多个初始化程序、赋值和表达式时，它看起来可能非常复杂甚至令人困惑。如果有疑问，保持你的代码简单易懂。有时候这并不容易，但根据我们的经验，总是可以做到的，而易于理解是良好代码质量的最重要标准之一。

# 分支语句

你已经在我们的例子中看到了分支语句`break`和`return`。我们将在本节中定义和讨论它们以及该组的第三个成员——分支语句`continue`。

# 中断和标记中断

你可能已经注意到，`break`语句对于`switch...case`选择语句能够正常工作是至关重要的（有关更多信息，请参阅`switch...case`部分）。如果包含在迭代语句的执行块中，它会立即终止`for`或`while`语句。

它在迭代语句中被广泛使用，用于在数组或集合中搜索特定元素。为了演示它的工作原理，例如，假设我们需要在社区学院的学生和教师中通过年龄和姓名找到某个人。首先创建`Person`，`Student`和`Teacher`类：

```java

class Person{

private int age;

private  String name;

public Person(int age, String name) {

this.age = age;

this.name = name;

}

@Override

public Boolean equals(Object o) {

if (this == o) return true;

Person person = (Person) o;

return age == person.age &&

Objects.equals(name, person.name);

}

@Override

public String toString() {

return "Person{age=" + age +

", name='" + name + "'}";

}

}

class Student extends Person {

private int year;

public Student(int age, String name, int year) {

super(age, name);

this.year = year;

}

@Override

public String toString() {

return "Student{year=" + year +

", " + super.toString() + "}";

}

}

class Teacher extends Person {

private String subject;

public Teacher(int age, String name, String subject) {

super(age, name);

this.subject = subject;

}

@Override

public String toString() {

return "Student{subject=" + subject +

", " + super.toString() + "}";

}

}

```

注意，`equals()`方法只在基类`Person`中实现。我们只通过姓名和年龄来识别一个人。还要注意使用关键字`super`，它允许我们访问父类的构造函数和`toString()`方法。

假设我们被指派在社区学院数据库中查找一个人（按姓名和年龄）。因此，我们已经创建了一个`List`类型的集合，并将在其中进行迭代，直到找到匹配项：

```java

List<Person> list =

List.of(new Teacher(32, "Joe", "History"),

new Student(29,"Joe", 4),

new Student(28,"Jill", 3),

new Teacher(33, "ALice", "Maths"));

Person personOfInterest = new Person(29,"Joe");

Person person = null;

对于（Person p: list）{

System.out.println(p);

如果（p.equals(personOfInterest)）{

person = p;

break;

}

}

如果（人== null）{

System.out.println("Not found: " + personOfInterest);

} else {

System.out.println("Found: " + person);

}

```

如果我们运行这个程序，结果将是：

![](img/5d1a86d0-d089-4e18-80b6-fb25ea409a53.png)

我们已经找到了我们要找的人。但是如果我们改变我们的搜索并寻找另一个人（只相差一岁）：

```java

Person personOfInterest = new Person(30,"Joe");

```

结果将是：

![](img/61fb11ee-a3a3-4c3b-a0cc-c2228878a8f7.png)

正如你所看到的，`break`语句允许在找到感兴趣的对象时立即退出循环，从而不浪费时间在迭代整个可能相当大的集合上。

在第十八章中，*流和管道*，我们将向您展示另一种（通常更有效）搜索集合或数组的方法。但在许多情况下，迭代元素仍然是一种可行的方法。

`break`语句也可以用于在多维数据结构中搜索特定元素。假设我们需要搜索一个三维数组，并找到其元素之和等于或大于 4 的最低维度数组。这是这样一个数组的示例：

```java

int[][][] data = {

{{1,0,2},{1,2,0},{2,1,0},{0,3,0}},

{{1,1,1},{1,3,0},{2,0,1},{1,0,1}}};

```

我们要找的最低维度数组是`{1,3,0}`。如果第一维是`x`，第二维是`y`，那么这个数组的位置是`x=1`，`y=1`，或`[1][1]`。让我们编写一个程序来找到这个数组：

```java

int[][][] data = {

{{1,0,2},{1,2,0},{2,1,0},{0,3,0}},

{{1,1,1},{1,3,0},{2,0,1},{1,0,1}}};

int threshold = 4;

int x = 0, y = 0;

Boolean isFound = false;

对于（int[][] dd: data）{

y = 0;

对于（int[] d: dd）{

int sum = 0;

对于（int i: d）{

sum += i;

如果（sum >= threshold）{

isFound = true;

break;

}

}

如果（isFound）{

break;

}

y++;

}

如果（isFound）{

break;

}

x++;

}

System.out.println("isFound=" + isFound + ", x=" + x + ", y=" + y);

//打印：isFound=true, x=1, y=1

```

正如你所看到的，我们使用一个名为`isFound`的布尔变量来方便地从最内层循环中退出，一旦在内部循环中找到了期望的结果。检查`isFound`变量的值的无聊需要使 Java 作者引入了一个标签 - 一个标识符后跟着一个冒号（`:`），可以放在语句的前面。`break`语句可以利用它。以下是如何使用标签更改先前的代码：

```java

int[][][] data = {

{{1,0,2},{1,2,0},{2,1,0},{0,3,0}},

{{1,1,1},{1,3,0},{2,0,1},{1,0,1}}};

int threshold = 4;

int x = 0, y = 0;

Boolean isFound = false;

退出：

对于（int[][] dd: data）{

y = 0;

对于（int[] d: dd）{

int sum = 0;

对于（int i: d）{

sum += i;

如果（sum >= threshold）{

isFound = true;

break exit;

}

}

y++;

}

x++;

}

System.out.println("isFound=" + isFound + ", x=" + x + ", y=" + y);

//打印：isFound=true, x=1, y=1

```

我们仍然使用变量`isFound`，但仅用于报告目的。`exit:`标签允许`break`语句指定哪个语句必须停止执行。这样，我们就不需要编写检查`isFound`变量值的样板代码。

# 继续和标记继续

`continue`语句支持与`break`语句支持的功能类似。但是，它不是退出循环，而是强制退出当前迭代，所以循环继续执行。为了演示它的工作原理，让我们假设，就像前一节中`break`语句的情况一样，我们需要搜索一个三维数组，并找到其元素总和等于或大于 4 的最低维度的数组。但是这次，总和不应包括等于 1 的元素。这是数组：

```java

int[][][] data = {

{{1,1,2},{0,3,0},{2,4,1},{2,3,2}},

{{0,2,0},{1,3,4},{2,0,1},{2,2,2}}};

```

我们的程序应该找到以下数组：

+   `data[0][2] = {2,4,1}`, `sum = 6` (因为 1 必须被跳过)

+   `data[0][3] = {2,3,2}`, `sum = 7`

+   `data[1][1] = {1,3,4}`, `sum = 7` (因为 1 必须被跳过)

+   `data[1][3]={2,2,2}`, `sum = 6`

如果跳过 1，则其他数组元素的总和不会达到 4。

这是程序：

```java

int[][][] data = {

{{1,1,2},{0,3,0},{2,4,1},{2,3,2}},

{{0,2,0},{1,3,4},{2,0,1},{2,2,2}}};

int threshold = 4;

int x = 0, y;

for(int[][] dd: data){

y = 0;

for(int[] d: dd){

int sum = 0;

for(int i: d){

if(i == 1){

continue;

}

sum += i;

}

if(sum >= threshold){

System.out.println("sum=" + sum + ", x=" + x + ", y=" + y);

}

y++;

}

x++;

}

```

如果我们运行它，结果将是：

![](img/74f53e31-3c98-4c50-9dc5-ea22b18790dd.png)

如您所见，结果正如我们所期望的那样：所有元素 1 都被跳过了。

为了演示如何使用带标签的`continue`语句，让我们改变要求：不仅要跳过元素 1，还要忽略包含这样一个元素的所有数组。换句话说，我们需要找到不包含 1 并且元素的总和等于或大于 4 的数组。

我们的程序应该只找到两个数组：

+   `data[0][3] = {2,3,2}`, `sum = 7`

+   `data[1][3] = {2,2,2}`, `sum = 6`

这是实现它的代码：

```java

int[][][] data = {

{{1,1,2},{0,3,0},{2,4,1},{2,3,2}},

{{0,2,0},{1,3,4},{2,0,1},{2,2,2}}};

int threshold = 4;

int x = 0, y;

for(int[][] dd: data){

y = 0;

cont: for(int[] d: dd){

int sum = 0;

for(int i: d){

if(i == 1){

y++;

continue cont;

}

sum += i;

}

if(sum >= threshold){

System.out.println("sum=" + sum + ", x=" + x + ", y=" + y);

}

y++;

}

x++;

}

```

如您所见，我们添加了一个名为`cont:`的标签，并在`continue`语句中引用它，因此内部循环的当前迭代和下一个外部循环的迭代停止执行。外部循环然后继续执行下一个迭代。如果我们运行代码，结果将是：

![](img/9e701141-fa2f-411a-8827-6e7aa41bc93e.png)

所有其他数组都被跳过，因为它们包含 1 或其元素的总和小于 4。

# 返回

`return`语句只能放在方法或构造函数中。它的功能是返回控制权给调用者，有或没有值。

在构造函数的情况下，不需要`return`语句。如果放在构造函数中，它必须是最后一条不返回值的语句：

```java

类 ConstructorDemo{

private int field;

public ConstructorDemo(int i) {

this.field = i;

return;

}

}

```

试图将`return`语句放在构造函数的最后一条语句之外，或者使其返回任何值，都会导致编译错误。

在方法的情况下，如果方法被声明为返回某种类型：

+   `return`语句是必需的

+   `return`语句必须有效地（见下面的示例）是方法的最后一条语句

+   可能有几个返回语句，但其中一个必须有效地（见下面的示例）是方法的最后一条语句，而其他的必须在选择语句内部；否则，将生成编译错误

+   如果`return`语句不返回任何内容，将导致编译错误

+   如果`return`语句返回的类型不是方法定义中声明的类型，也不是其子类型，它会导致编译错误

+   装箱、拆箱和类型扩宽是自动执行的，而类型缩窄需要类型转换

以下示例演示了`return`语句有效地成为方法的最后一条语句：

```java

public String method(int n){

if(n == 1){

返回“一个”；

} else {

return "不是一个";

}

}

```

方法的最后一条语句是选择语句，但`return`语句是选择语句内最后执行的语句。

这是一个具有许多返回语句的方法的示例：

```java

public static String methodWithManyReturns(){

if(true){

return "唯一返回的";

}

if(true){

return "永远不会到达";

}

return "永远不会到达";

}

```

尽管在方法中，只有第一个`return`语句总是返回，但编译器不会抱怨，方法会在没有运行时错误的情况下执行。它只是总是返回一个`唯一返回的`文字。

以下是具有多个返回语句的更现实的方法示例：

```java

public Boolean method01(int n){

if(n < 0) {

return true;

} else {

return false;

}

}

public Boolean sameAsMethod01(int n){

if(n < 0) {

return true;

}

return false;

}

public Boolean sameAsAbove(int n){

return n < 0 ? true : false;

}

public int method02(int n){

if(n < 0) {

return 1;

} else if(n == 0) {

return 2;

} else if (n == 1){

return 3;

} else {

return 4;

}

}

public int methodSameAsMethod02(int n){

if(n < 0) {

return 1;

}

switch(n) {

case 0:

return 2;

case 1:

return 3;

default:

return 4;

}

}

```

这里有关于装箱、拆箱、类型扩宽和缩窄的示例：

```java

public Integer methodBoxing(){

return 42;

}

public int methodUnboxing(){

return Integer.valueOf(42);

}

public int methodWidening(){

byte b = 42;

return b;

}

public byte methodNarrowing(){

int n = 42;

return (byte)n;

}

```

我们还可以重新审视程序，该程序在教师和学生名单中寻找特定的人：

```java

List<Person> list =

List.of(new Teacher(32, "Joe", "History"),

new Student(29,"Joe", 4),

new Student(28,"Jill", 3),

new Teacher(33, "ALice", "Maths"));

Person personOfInterest = new Person(29,"Joe");

Person person = null;

for (Person p: list){

System.out.println(p);

if(p.equals(personOfInterest)){

person = p;

break;

}

}

if(person == null){

System.out.println("未找到：" + personOfInterest);

} else {

System.out.println("找到了：" + person);

}

```

使用返回语句，我们现在可以创建`findPerson()`方法：

```java

Person findPerson(List<Person> list, Person personOfInterest){

Person person = null;

for (Person p: list){

System.out.println(p);

if(p.equals(personOfInterest)){

person = p;

break;

}

}

return person;

}

```

这个方法可以这样使用：

```java

List<Person> list = List.of(new Teacher(32, "Joe", "History"),

new Student(29,"Joe", 4),

new Student(28,"Jill", 3),

new Teacher(33, "ALice", "Maths"));

Person personOfInterest = new Person(29,"Joe");

Person person = findPerson(list, personOfInterest);

if(person == null){

System.out.println("未找到：" + personOfInterest);

} else {

System.out.println("找到了：" + person);

}

```

利用新的代码结构，我们可以进一步改变`findPerson()`方法，并展示`return`语句使用的更多变化：

```java

Person findPerson(List<Person> list, Person personOfInterest){

for (Person p: list){

System.out.println(p);

if(p.equals(personOfInterest)){

return p;

}

}

return null;

}

```

正如您所看到的，我们已经用返回语句替换了`break`语句。现在代码更易读了吗？一些程序员可能会说不，因为他们更喜欢只有一个`return`语句是返回结果的唯一来源。否则，他们认为，人们必须研究代码，看看是否有另一个——第三个——`return`语句，可能会返回另一个值。如果代码不那么简单，人们永远不确定是否已经识别了所有可能的返回。相反派的程序员可能会反驳说，方法应该很小，因此很容易找到所有的返回语句。但是，将方法变得很小通常会迫使创建深度嵌套的方法，这样就不那么容易理解了。这个争论可能会持续很长时间。这就是为什么我们让您自己尝试并决定您更喜欢哪种风格。

如果方法的返回类型定义为`void`：

+   不需要`return`语句

+   如果存在`return`语句，则不返回任何值

+   如果`return`语句返回一些值，会导致编译错误

+   可能有几个返回语句，但其中一个必须有效地成为方法的最后一个语句，而其他语句必须在选择语句内部；否则，将生成编译错误

为了演示没有值的`return`语句，我们将再次使用`findPerson()`方法。如果我们只需要打印结果，那么方法可以更改如下：

```java

void findPerson2(List<Person> list, Person personOfInterest){

for (Person p: list){

System.out.println(p);

if(p.equals(personOfInterest)){

System.out.println("Found: " + p);

return;

}

}

System.out.println("Not found: " + personOfInterest);

return;  //此语句是可选的

}

```

并且客户端代码看起来更简单：

```java

List<Person> list = List.of(new Teacher(32, "Joe", "History"),

new Student(29,"Joe", 4),

new Student(28,"Jill", 3),

new Teacher(33, "ALice", "Maths"));

Person personOfInterest = new Person(29,"Joe");

findPerson(list, personOfInterest);

```

或者它甚至可以更紧凑：

```java

List<Person> list = List.of(new Teacher(32, "Joe", "History"),

new Student(29,"Joe", 4),

new Student(28,"Jill", 3),

new Teacher(33, "ALice", "Maths"));

findPerson(list, new Person(29, "Joe");

```

与先前的讨论一样，有不同的风格将参数传递到方法中。有些人更喜欢更紧凑的代码风格。其他人则认为每个参数都必须有一个变量，因为变量的名称携带了额外的信息，有助于传达意图（比如`personOfInterest`的名称）。

这样的讨论是不可避免的，因为同样的代码必须由不同的人理解和维护，每个开发团队都必须找到适合所有团队成员需求和偏好的风格。

# 异常处理语句

正如我们在介绍中解释的那样，意外条件可能会导致 JVM 创建并抛出异常对象，或者应用程序代码可以这样做。一旦发生这种情况，控制流就会转移到异常处理`try`语句（也称为`try-catch`或`try-catch-finally`语句），如果异常是在`try`块内抛出的。这是一个捕获异常的例子：

```java

void exceptionCaught(){

try {

method2();

} catch (Exception ex){

ex.printStackTrace();

}

}

void method2(){

method1(null);

}

void method1(String s){

s.equals("whatever");

}

```

方法`exceptionCaught()`调用`method2()`，`method2()`调用`method1()`并将`null`传递给它。行`s.equals("whatever")`抛出`NullPointerException`，它通过方法调用堆栈传播，直到被`exceptionCaught()`方法的`try-catch`块捕获，并打印其堆栈跟踪（哪个方法调用了哪个方法以及类的哪一行）：

![](img/67e2282b-6d05-4627-ab9f-55ae0b280afa.png)

从堆栈跟踪中，您可以看到所有涉及的方法都属于同一个类`ExceptionHandlingDemo`。从下往上阅读，您可以看到：

+   方法`main()`在`ExceptionHandlingDemo`的第 5 行调用了方法`exceptionCaught()`

+   方法`exceptionCaught()`在同一类的第 10 行调用了`method2()`

+   `method2()`在第 17 行调用了`method1()`

+   `method1()`在第 21 行抛出了`java.lang.NullpointerException`

如果我们不看代码，我们就不知道这个异常是故意抛出的。例如，`method1()`可能如下所示：

```java

void method1(String s){

如果 s == null){

throw new NullPointerException();

}

}

```

但通常，程序员会添加一条消息来指示问题是什么：

```java

void method1(String s){

if(s == null){

throw new NullPointerException("参数 String 为空");

}

}

```

如果是这种情况，堆栈跟踪将显示一条消息：

![](img/42776307-171b-476f-a280-ba2e3d606546.png)

但是消息并不是自定义异常的可靠指标。一些标准异常也携带自己的消息。异常包是自定义异常的更好证据，或者异常是基类之一（`java.lang.Exception`或`java.langRuntimeException`）并且其中有一条消息。例如，以下代码自定义了`RuntimeException`：

```java

void method1(String s){

如果 s == null){

throw new RuntimeException("参数 String 为空");

}

}

```

以下是使用此类自定义异常的堆栈跟踪：

![](img/13e41dba-0766-4f70-9448-6c0cc91cce5a.png)

稍后我们将在*自定义异常*部分更多地讨论异常定制。

如果异常在`try...catch`块之外抛出，则程序执行将由 JVM 终止。以下是一个未被应用程序捕获的异常的示例：

```java

void exceptionNotCaught(){

method2();

}

void method2(){

method1(null);

}

void method1(String s){

s.equals("whatever");

}

```

如果我们运行此代码，结果是：

![](img/7f0cbbec-63b6-49a5-81a6-0db76a269dbc.png)

现在，让我们谈谈异常处理语句，然后再回到关于处理异常的最佳方法的讨论。

# throw

`throw`语句由关键字`throw`和`java.lang.Throwable`的变量或引用类型的值，或`null`引用组成。由于所有异常都是`java.lang.Throwable`的子类，因此以下任何一个`throw`语句都是正确的：

```java

throw new Exception("发生了一些事情");

Exception ex = new Exception("发生了一些事情");

throw ex;

Throwable thr = new Exception("发生了一些事情");

throw thr;

throw null;

```

如果抛出`null`，就像在最后一条语句中一样，那么 JVM 会将其转换为`NullPointerException`，因此这两条语句是等价的：

```java

throw null;

throw new NullPointerException;

```

另外，提醒一下，包`java.lang`不需要被导入。您可以通过名称引用`java.lang`包的任何成员（接口或类），而无需使用完全限定名称（包括包名）。这就是为什么我们能够写`NullPointerException`而不导入该类，而不是使用其完全限定名称`java.lang.NullPointerException`。我们将在第十二章 *Java 标准和外部库*中查看`java.lang`包的内容。

您还可以通过扩展`Throwable`或其任何子类来创建自己的异常，并抛出它们，而不是抛出`java.lang`包中的标准异常：

```java

class MyNpe extends NullPointerException{

public MyNpe(String message){

super(message);

}

//您需要在此处添加任何代码

}

class MyRuntimeException extends RuntimeException{

public MyRuntimeException(String message){

super(message);

}

//您需要在此处添加任何代码

}

class MyThrowable extends Throwable{

public MyThrowable(String message){

super(message);

}

//您需要在此处编写的任何代码

}

class MyException extends Exception{

public MyException(String message){

调用 super(message);

}

//您需要在此处编写的任何代码

}

```

为什么要这样做将在阅读*自定义异常*部分后变得清晰。

# 尝试...捕获

当在`try`块内抛出异常时，它将控制流重定向到其第一个`catch`子句（在下面的示例中捕获`NullPointerException`）：

```java

void exceptionCaught(){

尝试{

method2();

} catch (NullPointerException ex){

System.out.println("NPE 捕获");

ex.printStackTrace();

} catch (RuntimeException ex){

System.out.println("捕获 RuntimeException");

ex.printStackTrace();

} catch (Exception ex){

System.out.println("捕获异常");

ex.printStackTrace();

}

}

```

如果有多个`catch`子句，编译器会强制您安排它们，以便子异常在父异常之前列出。在我们之前的示例中，`NullPointerException`扩展了`RuntimeException`扩展了`Exception`。如果抛出的异常类型与最顶层的`catch`子句匹配，此`catch`块处理异常（我们将很快讨论它的含义）。如果最顶层子句不匹配异常类型，则下一个`catch`子句获取控制流并处理异常（如果匹配子句类型）。如果不匹配，则控制流传递到下一个子句，直到异常被处理或尝试所有子句。如果没有一个子句匹配，异常将被抛出直到它被某个 try-catch 块处理，或者它传播到程序代码之外。在这种情况下，JVM 终止程序执行（准确地说，它终止线程执行，但我们将在第十一章，*JVM 进程和垃圾回收*中讨论线程）。

让我们通过运行示例来演示这一点。如果我们像之前展示的那样在`exceptionCaught()`方法中使用三个`catch`子句，并在`method1()`中抛出`NullPointerException`：

```java

void method1(String s){

throw new NullPointerException("参数 String 为空");

}

```

结果将如下截图所示：

![](img/b8721c37-ab6d-4418-9406-f8c0e2517e3c.png)

您可以看到最顶层的`catch`子句按预期捕获了异常。

如果我们将`method1()`更改为抛出`RuntimeException`：

```java

void method1(String s){

throw new RuntimeException("参数 String 为空");

}

```

您可能不会感到惊讶，看到第二个`catch`子句捕获它。因此，我们不打算演示它。我们最好再次更改`method1()`，让它抛出`ArrayIndexOutOfBoundsException`，它是`RuntimeException`的扩展，但未列在任何捕获子句中：

```java

void method1(String s){

throw new ArrayIndexOutOfBoundsException("索引...更大" +

"比数组长度...更大");

}

```

如果我们再次运行代码，结果将如下所示：

![](img/4b8464ff-2568-4c63-a7bf-c8a8fb30d954.png)

正如您所看到的，异常被第一个匹配其类型的`catch`子句捕获。这就是编译器强制您列出它们的原因，以便子类通常在其父类之前列出，因此最具体的类型首先列出。这样，第一个匹配的子句总是最佳匹配。

现在，您可能完全希望看到任何非`RuntimeException`都被最后一个`catch`子句捕获。这是一个正确的期望。但在我们抛出它之前，我们必须解决*已检查*和*未检查*（也称为*运行时*）异常之间的区别。

# 已检查和未检查（运行时）异常

为了理解为什么这个主题很重要，让我们尝试在`method1()`中抛出`Exception`类型的异常。为了进行这个测试，我们将使用`InstantiationException`，它扩展了`Exception`。假设有一些输入数据的验证（来自某些外部来源），结果证明它们不足以实例化某些对象：

```java

void method1(String s) {

//一些输入数据验证

throw new InstantiationException("字段没有值" +

" SomeClass 的 someField。"

}

```

我们编写了这段代码，突然编译器生成了一个错误，`Unhandled exception java.lang.InstantiationException`，尽管我们在客户端代码中有一个`catch`子句，它将匹配这种类型的异常（在方法`exceptionCaught()`中的最后一个`catch`子句）。

错误的原因是所有扩展`Exception`类但不是其子类`RuntimeException`的异常类型在编译时都会被检查，因此得名。编译器会检查这些异常是否在其发生的方法中得到处理：

+   如果在异常发生的方法中有一个`try-catch`块捕获了这个异常并且不让它传播到方法外部，编译器就不会抱怨

+   否则，它会检查方法声明中是否有列出此异常的`throws`子句；这里是一个例子：

```java

void method1(String s) throws Exception{

//一些输入数据验证

throw new InstantiationException("字段没有值" +

" SomeClass 的 someField。");

}

```

`throws`子句必须列出所有可能传播到方法外部的已检查异常。通过添加`throws Exception`，即使我们决定抛出任何其他已检查异常，编译器也会满意，因为它们都是`Exception`类型，因此都包含在新的`throws`子句中。

在下一节`Throws`中，您将阅读一些使用`throws`子句中基本异常类的优缺点，在稍后的*异常处理的一些最佳实践*部分中，我们将讨论一些其他可能的解决方案。

与此同时，让我们继续讨论已检查异常的使用。在我们的演示代码中，我们决定在`method1()`的声明中添加`throws Exception`子句。这个改变立即在`method2()`中触发了相同的错误`Unhandled exception java.lang.InstantiationException`，因为`method2()`调用了`method1()`但没有处理`Exception`。因此，我们不得不在`method2()`中也添加一个`throws`子句：

```java

void method2() throws Exception{

method1(null);

}

```

只有`method2()`的调用者——`exceptionCaught()`方法——不需要更改，因为它处理`Exception`类型。代码的最终版本是：

```java

void exceptionCaught(){

try {

method2();

} catch (NullPointerException ex){

System.out.println("捕获 NPE");

ex.printStackTrace();

} catch (RuntimeException ex){

System.out.println("捕获 RuntimeException");

ex.printStackTrace();

} catch (Exception ex){

System.out.println("捕获异常");

ex.printStackTrace();

}

}

void method2() throws Exception{

method1(null);

}

void method1(String s) throws Exception{

throw new InstantiationException("字段没有值" +

" SomeClass 的 someField。");

}

```

如果我们现在调用`exceptionCaught()`方法，结果将是：

![](img/8faf4c71-38d5-42b8-96bb-6b261faeae1d.png)

这正是我们所期望的。`Exception`类型的最后一个`catch`子句匹配了`InstantiationException`类型。

未检查的异常——`RuntimeExceptions`类的后代——在编译时不会被检查，因此得名，并且不需要在`throws`子句中列出。

一般来说，已检查异常（应该）用于可恢复的条件，而未检查异常用于不可恢复的条件。我们将在稍后的*什么是异常处理？*和*一些最佳实践* *异常处理*部分中更多地讨论这个问题。

# 抛出

`throws`子句必须列出方法或构造函数可以抛出的所有已检查异常类（`Exception`类的后代，但不是`RuntimeException`类的后代）。在`throws`子句中列出未检查的异常类（`RuntimeException`类的后代）是允许的，但不是必需的。以下是一个例子：

```java

void method1(String s)

throws InstantiationException, InterruptedException {

//一些输入数据验证

if(一些数据丢失){

throw new InstantiationException("字段没有值" +

" SomeClass 的 someField 字段。");

}

//一些其他代码

if(其他原因){

throw new InterruptedException("原因..."); //检查异常

}

}

```

或者，可以只列出`throws`子句中的基类异常，而不是声明抛出两种不同的异常：

```java

void method1(String s) throws Exception {

//一些输入数据验证

if(一些数据丢失){

throw new InstantiationException("字段没有值" +

" SomeClass 的 someField 字段。");

}

//一些其他代码

if(其他原因){

throw new InterruptedException("原因..."); //检查异常

}

}

```

然而，这意味着潜在失败的多样性和可能的原因将隐藏在客户端，因此一个人必须要么：

+   在方法内处理异常

+   假设客户端代码将根据消息的内容来确定其行为（这通常是不可靠的并且可能会发生变化）

+   假设客户端无论实际的异常类型是什么都会表现相同

+   假设该方法永远不会抛出任何其他已检查异常，如果确实抛出，客户端的行为不应该改变

有太多的假设让人感到不舒服，只声明`throws`子句中的基类异常。但有一些最佳实践可以避免这种困境。我们将在*异常处理的一些最佳实践*部分中讨论它们。

# 自定义异常

在这一部分，我们承诺讨论自定义异常创建的动机。以下是两个例子：

```java

//未检查的自定义异常

class MyRuntimeException extends RuntimeException{

public MyRuntimeException(String message){

super(message);

}

//这里需要任何代码

}

//检查自定义异常

class MyException extends Exception{

public MyException(String message){

super(message);

}

//这里需要任何代码

}

```

直到你意识到注释`这里需要任何代码`允许你在自定义类中放入任何数据或功能，并利用异常处理机制将这样的对象从任何代码深度传播到任何你需要的级别，这些示例看起来并不特别有用。

由于这只是 Java 编程的介绍，这些情况超出了本书的范围。我们只是想确保你知道这样的功能存在，所以当你需要它或构建你自己的创新解决方案时，你可以在互联网上搜索。

然而，在 Java 社区中有关利用异常处理机制进行业务目的的讨论仍在进行中，我们将在*异常处理的一些最佳实践*部分中稍后讨论。

# 什么是异常处理？

正如我们已经提到的，检查异常最初被认为是用于可恢复的条件，当调用者代码可能会自动执行某些操作并根据捕获的异常类型和可能携带的数据采取另一个执行分支时。这就是异常处理的主要目的和功能。

不幸的是，这种利用异常的方式被证明并不是非常有效，因为一旦发现异常条件，代码就会得到增强，并使这样的条件成为可能的处理选项之一，尽管并不经常执行。

次要功能是记录错误条件和所有相关信息，以供以后分析和代码增强。

异常处理的第三个同样重要的功能是保护应用程序免受完全失败。意外情况发生了，但希望这种情况很少，主流处理仍然可用于应用程序继续按设计工作。

异常处理的第四个功能是在其他手段不够有效的特殊情况下提供信息传递的机制。异常处理的这最后一个功能仍然存在争议，且并不经常使用。我们将在下一节讨论它。

# 异常处理的一些最佳实践

Java 异常处理机制旨在解决可能的边缘情况和意外的程序终止。预期的错误类别是：

+   **可恢复的**：可以根据应用逻辑自动修复的异常

+   **不可恢复的**：无法自动纠正并导致程序终止的异常

通过引入已检查的异常（`Exception`类的后代）来解决第一类错误，而第二类错误则成为未经检查的异常领域（`RuntimeException`类的后代）。

不幸的是，这种分类方法在编程实践中并不符合实际情况，特别是对于与开发旨在在各种环境和执行上下文中使用的库和框架无关的编程领域。典型的应用程序开发总是能够直接在代码中解决问题，而无需编写复杂的恢复机制。这种区别很重要，因为作为库的作者，你永远不知道你的方法将在何处以及如何被使用，而作为应用程序开发人员，你确切地了解环境和执行上下文。

即使在写作时，Java 的作者们间接地确认了这一经验，向`java.lang`包中添加了 15 个未经检查的异常和仅九个已检查的异常。如果原始期望得到了实践的确认，人们会期望只有少数不可恢复的（未经检查的）异常和更多类型的可恢复的（已检查的）异常。与此同时，甚至`java.lang`包中的一些已检查的异常看起来也不太可恢复：

+   `ClassNotFoundException`：当 JVM 无法找到所引用的类时抛出

+   `CloneNotSupportedException`：指示对象类中的克隆方法未实现`Cloneable`接口

+   `IllegalAccessException`：当当前执行的方法无法访问指定类、字段、方法或构造函数的定义时抛出

实际上，很难找到一种情况，其中编写自动恢复代码比只是在主流处理中添加另一个逻辑分支更值得。

考虑到这一点，让我们列举一些被证明是有用和有效的最佳实践：

+   始终捕获所有异常

+   尽可能接近源头处理每个异常

+   除非必须，否则不要使用已检查的异常

+   通过重新抛出它们作为带有相应消息的`RuntimeException`，将第三方已检查的异常转换为未经检查的异常

+   除非必须，否则不要创建自定义异常

+   除非必须，否则不要使用异常处理机制来驱动业务逻辑

+   通过使用消息系统和可选的枚举类型自定义通用的`RuntimeException`，而不是使用异常类型来传达错误的原因

# 最后

`finally`块可以添加到带有或不带有`catch`子句的`try`块中。格式如下：

```java

尝试 {

//尝试块的代码

} catch (...){

//可选的捕获块代码

} 最后 {

//最后块的代码

}

```

如果存在，则`finally`块中的代码总是在方法退出之前执行。无论`try`块中的代码是否抛出异常，以及这个异常是否在`catch`块中的一个中被处理，或者`try`块中的代码是否没有抛出异常，`finally`块都会在方法返回控制流到调用者之前执行。

最初，`finally`块用于关闭`try`块中需要关闭的一些资源。例如，如果代码已经打开了到数据库的连接，或者已经在磁盘上与文件建立了读取或写入连接，那么在操作完成或抛出异常时必须关闭这样的连接。否则，未及时关闭的连接会使资源（维护连接所需的资源）被锁定而不被使用。我们将在[第十一章]（e8c37d86-291d-4500-84ea-719683172477.xhtml）*JVM 进程和垃圾回收*中讨论 JVM 进程。

因此，典型的代码看起来像这样：

```java

Connection conn = null;

尝试{

conn = createConnection（）;

//尝试块的代码

} catch（...）{

//可选的 catch 块代码

}最后{

if（conn！= null）{

conn.close（）;

}

}

```

它运行得很好。但是，一个名为`try...with...resources`的新的 Java 功能允许在连接类实现`AutoCloseable`时自动关闭连接（大多数流行的连接类都是这样）。我们将在[第十六章]（d77f1f16-0aa6-4d13-b9a8-f2b6e195f0f1.xhtml）*数据库编程*中讨论`try...with...resources`结构。这一发展降低了`finally`块的实用性，现在它主要用于处理一些不能使用`AutoCloseable`接口执行的代码，但必须在方法无条件返回之前执行。例如，我们可以通过利用`finally`块来重构我们的`exceptionCaught（）`方法，如下所示：

```java

void exceptionCaught（）{

Exception exf = null;

尝试{

method2（）;

} catch（NullPointerException ex）{

exf = ex;

System.out.println（"NPE 捕获"）;

} catch（RuntimeException ex）{

exf = ex;

System.out.println（"RuntimeException 捕获"）;

} catch（Exception ex）{

exf = ex;

System.out.println（"捕获异常"）;

}最后{

if（exf！= null）{

exf.printStackTrace（）;

}

}

```

还有其他情况下的`finally`块使用，基于它在控制流返回给方法调用者之前的保证执行。

# Assert 需要 JVM 选项-ea

分支`assert`语句可用于验证应用程序测试中的数据，特别是用于访问很少使用的执行路径或数据组合。这种能力的独特之处在于，除非 JVM 使用选项`-ea`运行，否则不会执行代码。

本书不讨论`assert`语句的功能和可能的应用。我们只会演示它的基本用法以及如何在 IntelliJ IDEA 中打开它。

看看下面的代码：

```java

public class AssertDemo {

public static void main（String ... args）{

int x = 2;

assert x>1：“x<=1”;

断言 x == 1：“x！= 1”;

}

}

```

第一个`assert`语句评估表达式`x>1`，如果表达式`x>1`评估为`false`，则停止程序执行（并报告`x<=1`）。

第二个`assert`语句评估表达式`x == 1`，如果表达式`x == 1`评估为`false`，则停止程序执行（并报告`x！= 1`）。

如果我们现在运行这个程序，将不会执行任何`assert`语句。要打开它们，请单击 IntelliJ IDEA 菜单中的 Run 并选择 Edit Configurations，如下面的屏幕截图所示：

！[]（img / 4cfd5dda-e07c-45ec-b9bd-13c4e4b6ac33.png）

运行/调试配置屏幕将打开。在 VM 选项字段中键入`-ea`，如下面的屏幕截图所示：

！[]（img / 8019cb61-0d4f-4d29-8d28-d10aef60490e.png）

然后，点击屏幕底部的确定按钮。

如果现在运行`AssertDemo`程序，结果将是：

![](img/9666cbdf-3943-494e-a7b8-d8a9eebdd592.png)

`-ea`选项不应该在生产中使用，除非可能是为了测试目的而临时使用，因为它会增加开销并影响应用程序的性能。

# 练习-无限循环

写一个或两个无限循环的例子。

# 答案

以下是一个可能的无限循环实现：

```java

while(true){

System.out.println("尝试阻止我"); //无限打印

}

```

以下是另一个：

```java

for (;;){

System.out.println("尝试阻止我"); //无限打印

}

```

这也是一个无限循环：

```java

for (int x=2; x > 0; x--){

System.out.println(x++ + " "); //无限打印 2

}

```

在这段代码中，布尔表达式`x > 0`总是被评估为`true`，因为`x`被初始化为`2`，然后在每次迭代中递增和递减`1`。

# 总结

本章描述了 Java 语句，让您根据实现的算法逻辑构建程序流，使用条件语句、迭代语句、分支语句和异常处理。对 Java 异常的广泛讨论帮助您在这个复杂且经常正确使用的领域中进行导航。为最有效和最少混淆的异常处理提供了最佳实践。

在下一章中，我们将深入了解 JVM 的内部工作机制，讨论其进程和其他重要方面，包括线程和垃圾回收机制，这些对于有效的 Java 编程非常重要，它们帮助应用程序重新获得不再使用的内存。
