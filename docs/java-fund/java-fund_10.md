# *附录*

## 关于

本节旨在帮助学生完成书中的活动。

其中包括学生执行活动目标所需执行的详细步骤。

## 第 1 课：Java 简介

### 活动 1：打印简单算术运算的结果

解决方案：

1.  创建一个名为`Operations`的类，如下所示：

```java
public class Operations
{
```

1.  在`main()`中，打印一句话描述您将执行的值操作以及结果：

```java
    public static void main(String[] args) {
        System.out.println("The sum of 3 + 4 is " + (3 + 4));
        System.out.println("The product of 3 + 4 is " + (3 * 4));
    }
}
```

输出将如下所示：

```java
The sum of 3 + 4 is 7
The product of 3 + 4 is 12
```

### 活动 2：从用户那里读取值并使用 Scanner 类执行操作。

解决方案：

1.  右键单击`src`文件夹，然后选择**新建**|**类**。

1.  输入`ReadScanner`作为类名，然后点击**确定**。

1.  导入`java.util.Scanner`包：

```java
import java.util.Scanner;
```

1.  在`main()`中输入以下内容：

```java
public class ReadScanner
{
    static Scanner sc = new Scanner(System.in);
  public static void main(String[] args) {
    System.out.print("Enter a number: ");
    int a = sc.nextInt();
    System.out.print("Enter 2nd number: ");
    int b = sc.nextInt();
    System.out.println("The sum is " + (a + b) + ".");
    }
}
```

1.  运行主程序。

输出将如下所示：

```java
Enter a number: 12                                                                                                             
Enter 2nd number: 23
The sum is 35\.  
```

### 活动 3：计算金融工具的百分比增加或减少

解决方案：

1.  右键单击`src`文件夹，然后选择**新建**|**类**。

1.  输入`StockChangeCalculator`作为类名，然后点击**确定**。

1.  导入`java.util.Scanner`包：

```java
import java.util.Scanner;
```

1.  在`main()`中输入以下内容：

```java
public class StockChangeCalculator{
static Scanner sc = new Scanner(System.in);
public static void main(String[] args) {
    System.out.print("Enter the stock symbol: ");
    String symbol = sc.nextLine();
    System.out.printf("Enter %s's day 1 value: ", symbol);
    double day1 = sc.nextDouble();
    System.out.printf("Enter %s's day 2 value: ", symbol);
    double day2 = sc.nextDouble();
    double percentChange = 100 * (day2 - day1) / day1;
    System.out.printf("%s has changed %.2f%% in one day.", symbol, percentChange);
}
}
```

1.  运行主程序。

输出应该类似于：

```java
Enter the stock symbol: AAPL                                                                                                             
Enter AAPL's day 1 value: 100                                                                                                           
Enter AAPL's day 2 value: 91.5                                                                                                           
AAPL has changed -8.50% in one day.
```

## 第 2 课：变量、数据类型和运算符

### 活动 4：输入学生信息并输出 ID

解决方案：

1.  导入`Scanner`包并创建一个新类

```java
import java.util.Scanner;
{
public class Input{
static Scanner sc = new Scanner(System.in);
    public static void main(String[] args) 
{
```

1.  将学生姓名作为字符串。

```java
System.out.print("Enter student name: ");
String name = sc.nextLine();
```

1.  将大学名称作为字符串。

```java
System.out.print("Enter Name of the University: ");
String uni = sc.nextLine();
```

1.  将学生的年龄作为整数。

```java
System.out.print("Enter Age: ");
int age = sc.nextInt();
```

1.  打印学生详细信息。

```java
System.out.println("Here is your ID");
System.out.println("*********************************");
System.out.println("Name: " + name);
System.out.println("University: " + uni);
System.out.println("Age: " + age);
System.out.println("*********************************");
    }
} 
}
```

### 活动 5：计算满箱水果的数量

解决方案：

1.  右键单击`src`文件夹，然后选择**新建**|**类**。

1.  输入`PeachCalculator`作为类名，然后点击**确定**。

1.  导入`java.util.Scanner`包：

```java
import java.util.Scanner;
```

1.  在`main()`中输入以下内容：

```java
public class PeachCalculator{
static Scanner sc = new Scanner(System.in);
public static void main(String[] args) {
    System.out.print("Enter the number of peaches picked: ");
    int numberOfPeaches = sc.nextInt();
    int numberOfFullBoxes = numberOfPeaches / 20;
    int numberOfPeachesLeft = numberOfPeaches - numberOfFullBoxes * 20;
    System.out.printf("We have %d full boxes and %d peaches left.", numberOfFullBoxes, numberOfPeachesLeft);
}
}
```

1.  运行主程序。

输出应该类似于：

```java
Enter the number of peaches picked: 55
We have 2 full boxes and 15 peaches left.
```

## 第 3 课：控制流

### 活动 6：使用条件控制执行流程

解决方案：

1.  创建一个名为`Salary`的类并添加`main()`方法：

```java
public class Salary {
   public static void main(String args[]) { 
```

1.  初始化两个变量`workerhours`和`salary`。

```java
int workerhours = 10; 
double salary = 0;
```

1.  在`if`条件中，检查工人的工作时间是否低于所需的工作时间。如果条件成立，则工资应为（工作时间* 10）。

```java
if (workerhours <= 8 ) 
salary = workerhours*10;
```

1.  使用`else if`语句检查工作时间是否在 8 小时和 12 小时之间。如果是真的，则工资应为前 8 小时每小时$10，剩下的小时应按每小时$12 计算。

```java
else if((workerhours > 8) && (workerhours < 12)) 
salary = 8*10 + (workerhours - 8) * 12;
```

1.  使用`else`块来处理每天额外的$160（额外一天的工资）的默认情况。

```java
else
    salary = 160;
System.out.println("The worker's salary is " + salary);
}
}
```

### 活动 7：开发温度系统

解决方案：

1.  声明两个字符串，`temp`和`weatherWarning`，然后用`High`、`Low`或`Humid`初始化`temp`。

```java
public class TempSystem
{
    public static void main(String[] args) {
        String temp = "Low";
        String weatherWarning;
```

1.  创建一个 switch 语句，检查`temp`的不同情况，然后根据每种情况的`temp`初始化变量`weatherWarning`为适当的消息（`High`、`Low`、`Humid`）。

```java
switch (temp) { 
        case "High": 
            weatherWarning = "It's hot outside, do not forget sunblock."; 
            break; 
        case "Low": 
            weatherWarning = "It's cold outside, do not forget your coat."; 
            break; 
        case "Humid": 
            weatherWarning = "The weather is humid, open your windows."; 
            break;
```

1.  在默认情况下，将`weatherWarning`初始化为“天气看起来不错。出去散步吧”。

```java
default: 
  weatherWarning = "The weather looks good. Take a walk outside"; 
  break;
```

1.  完成 switch 结构后，打印`weatherWarning`的值。

```java
} 
        System.out.println(weatherWarning); 
    }
}
```

1.  运行程序以查看输出，应该类似于：

```java
It's cold outside, do not forget your coat.
```

完整代码如下：

```java
public class TempSystem
{
    public static void main(String[] args) {
        String temp = "Low";
        String weatherWarning;
            switch (temp) { 
        case "High": 
            weatherWarning = "It's hot outside, do not forget sunblock."; 
            break; 
        case "Low": 
            weatherWarning = "It's cold outside, do not forget your coat."; 
            break; 
        case "Humid": 
            weatherWarning = "The weather is humid, open your windows."; 
            break; 

        default: 
            weatherWarning = "The weather looks good. Take a walk outside"; 
            break; 
        } 
        System.out.println(weatherWarning); 
    }
}
```

### 活动 8：实现 for 循环

解决方案：

1.  右键单击`src`文件夹，然后选择**新建**|**类**。

1.  输入`PeachBoxCounter`作为类名，然后点击**确定**。

1.  导入`java.util.Scanner`包：

```java
import java.util.Scanner;
```

1.  在`main()`中输入以下内容：

```java
public class PeachBoxCounter
{
static Scanner sc = new Scanner(System.in);
public static void main(String[] args) {
System.out.print("Enter the number of peaches picked: ");
int numberOfPeaches = sc.nextInt();
for (int numShipped = 0; numShipped < numberOfPeaches; numShipped += 20)      {
System.out.printf("shipped %d peaches so far\n", numShipped);
}
}
}
```

### 活动 9：实现 while 循环

解决方案：

1.  右键单击`src`文件夹，然后选择**新建**|**类**。

1.  输入`PeachBoxCounters`作为类名，然后点击**确定**。

1.  导入`java.util.Scanner`包：

```java
import java.util.Scanner;
```

1.  在`main()`中输入以下内容：

```java
public class PeachBoxCounters{
static Scanner sc = new Scanner(System.in);
public static void main(String[] args) {
    System.out.print("Enter the number of peaches picked: ");
    int numberOfPeaches = sc.nextInt();
    int numberOfBoxesShipped = 0;
    while (numberOfPeaches >= 20) {
        numberOfPeaches -= 20;
        numberOfBoxesShipped += 1;
        System.out.printf("%d boxes shipped, %d peaches remaining\n", 
                numberOfBoxesShipped, numberOfPeaches);
    }
}
}
```

### 活动 10：实现循环结构

解决方案：

1.  导入从用户读取数据所需的包。

```java
import java.util.Scanner;
public class Theater {
public static void main(String[] args)
```

1.  声明变量以存储可用座位总数、剩余座位和请求的票数。

```java
{
int total = 10, request = 0, remaining = 10;
```

1.  在`while`循环内，实现`if else`循环，检查请求是否有效，这意味着请求的票数少于剩余座位数。

```java
while (remaining>=0)
{
System.out.println("Enter the number of tickets");
Scanner in = new Scanner(System.in);
request = in.nextInt();
```

1.  如果前一步中的逻辑为真，则打印一条消息以表示票已处理，将剩余座位设置为适当的值，并要求获取下一组票。

```java
if(request <= remaining)
{
System.out.println("Your " + request +" tickets have been procced. Please pay and enjoy the show.");
remaining = remaining - request;
request = 0;
}
```

1.  如果步骤 3 中的逻辑为假，则打印适当的消息并跳出循环：

```java
else
{
System.out.println("Sorry your request could not be processed");
break;
}
}
}
}
```

### 活动 11：使用嵌套循环进行连续桃子装运

解决方案：

1.  右键单击`src`文件夹，然后选择**新建** | **类**。

1.  输入`PeachBoxCounter`作为类名，然后单击**确定**。

1.  导入`java.util.Scanner`包：

```java
import java.util.Scanner;
```

1.  在`main()`中输入以下内容：

```java
public class PeachBoxCount{    
static Scanner sc = new Scanner(System.in);
public static void main(String[] args) {
    int numberOfBoxesShipped = 0;
    int numberOfPeaches = 0;
    while (true) {
        System.out.print("Enter the number of peaches picked: ");
        int incomingNumberOfPeaches = sc.nextInt();
        if (incomingNumberOfPeaches == 0) {
            break;
        }
        numberOfPeaches += incomingNumberOfPeaches;
        while (numberOfPeaches >= 20) {
            numberOfPeaches -= 20;
            numberOfBoxesShipped += 1;
            System.out.printf("%d boxes shipped, %d peaches remaining\n",
                    numberOfBoxesShipped, numberOfPeaches);
        }
    }
}
}
```

## 第 4 课：面向对象编程

### 活动 12：在 Java 中创建一个简单的类

解决方案：

1.  在 IDE 中创建一个名为`Animals`的新项目。

1.  在项目中，在`src/`文件夹下创建一个名为`Animal.java`的新文件。

1.  打开`Animal.java`并粘贴以下代码：

```java
public class Animal {

}
```

1.  在大括号内，创建以下实例变量来保存我们的数据，如下所示：

```java
public class Animal {
        int legs;
        int ears;
        int eyes;
        String family;
        String name;

    }
```

1.  在实例变量下面，定义两个构造函数。一个将不带参数并将腿初始化为 4，耳朵初始化为 2，眼睛初始化为 2。第二个构造函数将以腿、耳朵和眼睛的值作为参数，并设置这些值：

```java
public class Animal {
        int legs;
        int ears;
        int eyes;
        String family;
        String name;
        public Animal(){
            this(4, 2,2);
        }
        public Animal(int legs, int ears, int eyes){
            this.legs = legs;
            this.ears = ears;
            this.eyes = ears;
        }
}
```

1.  定义四个方法，两个用于设置和获取家庭，两个用于设置和获取名称：

#### 注意

```java
public class Animal {
    int legs;
    int ears;
    int eyes;
    String family;
    String name;
    public Animal(){
        this(4, 2,2);
    }
    public Animal(int legs, int ears, int eyes){
        this.legs = legs;
        this.ears = ears;
        this.eyes = ears;
    }
    public String getFamily() {
        return family;
    }
    public void setFamily(String family) {
        this.family = family;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

我们已经完成了构建我们的 Animal 类。让我们继续创建这个类的几个实例。

1.  创建一个名为`Animals.java`的新文件，并将以下代码复制到其中，如下所示：

```java
public class Animals {

       public static void main(String[] args){

       }
}
```

1.  创建两个`Animal`类的对象：

```java
public class Animals {
        public static void main(String[] args){
            Animal cow = new Animal();
            Animal goat = new Animal();
        }
}
```

1.  让我们再创建一个有 2 条腿、2 只耳朵和 2 只眼睛的动物：

```java
Animal duck = new Animal(2, 2, 2);
```

1.  为了设置动物的名称和家庭，我们将使用在类中创建的 getter 和 setter。将以下行复制/写入`Animals`类中：

```java
public class Animals {
    public static void main(String[] args){
        Animal cow = new Animal();
        Animal goat = new Animal();
        Animal duck = new Animal(2, 2, 2);
        cow.setName("Cow");
        cow.setFamily("Bovidae");
        goat.setName("Goat");
        goat.setFamily("Bovidae");
        duck.setName("Duck");
        duck.setFamily("Anatidae");

        System.out.println(cow.getName());
        System.out.println(goat.getName());
        System.out.println(duck.getFamily());
    }
}
```

前面代码的输出如下：

```java
Cow
Goat
Anatide
```

![](img/C09581_04_09.jpg)

###### 图 4.9：Animal 类的输出

### 活动 13：编写一个计算器类

解决方案：

1.  创建一个名为 Calculator 的类：

```java
public class Calculator {
```

1.  创建三个字段`double operand1`，`double operand2`和`String operator`。添加一个设置所有三个字段的构造函数。

```java
private final double operand1;
private final double operand2;
private final String operator;
public Calculator(double operand1, double operand2, String operator){
this.operand1 = operand1;
this.operand2 = operand2;
this.operator = operator;
}
```

1.  在这个类中，添加一个`operate`方法，它将检查运算符是什么（"+"，"-"，"x"或"/"）并执行正确的操作，返回结果：

```java
public double operate() {
if (this.operator.equals("-")) {
return operand1 - operand2;
} else if (this.operator.equals("x")) {
return operand1 * operand2;
} else if (this.operator.equals("/")) {
return operand1 / operand2;
} else {
// If operator is sum or unknown, return sum
return operand1 + operand2;
}
}
```

1.  编写一个`main()`方法如下：

```java
public static void main (String [] args) {
        System.out.println("1 + 1 = " + new Calculator(1, 1, "+").operate());
        System.out.println("4 - 2 = " + new Calculator(4, 2, "-").operate());
        System.out.println("1 x 2 = " + new Calculator(1, 2, "x").operate());
        System.out.println("10 / 2 = " + new Calculator(10, 2, "/").operate());
    }
}
```

### 活动 14：使用 Java 创建计算器

解决方案：

1.  创建一个名为`Operator`的类，它具有一个在构造函数中初始化的 String 字段，表示运算符。这个类应该有一个代表默认运算符的默认构造函数，即 sum。`Operator`类还应该有一个名为`operate`的方法，它接收两个 double 并将运算符的结果作为 double 返回。默认操作是`sum`：

```java
public class Operator {
    public final String operator;
    public Operator() {
        this("+");
    }
    public Operator(String operator) {
        this.operator = operator;
    }
    public boolean matches(String toCheckFor) {
        return this.operator.equals(toCheckFor);
    }
    public double operate(double operand1, double operand2) {
        return operand1 + operand2;
    }
}
```

1.  创建另一个名为`Subtraction`的类。它继承自`Operator`并覆盖

`operate`方法与它所代表的每个操作一起使用。它还需要一个不带参数的构造函数，调用 super 传递它所代表的运算符

代表：

```java
public class Subtraction extends Operator {
    public Subtraction() {
        super("-");
    }
    @Override
    public double operate(double operand1, double operand2) {
        return operand1 - operand2;
    }
}
```

1.  创建另一个名为`Multiplication`的类。它继承自 Operator 并覆盖`operate`方法，其中包含它所代表的每个操作。它还需要一个不带参数的构造函数，调用 super 传递它所代表的运算符：

```java
public class Multiplication extends Operator {
    public Multiplication() {
        super("x");
    }
    @Override
    public double operate(double operand1, double operand2) {
        return operand1 * operand2;
    }
}
```

1.  创建另一个名为`Division`的类。它继承自 Operator 并覆盖`operate`方法，其中包含它所代表的每个操作。它还需要一个不带参数的构造函数，调用 super 传递它所代表的运算符：

```java
public class Division extends Operator {
    public Division() {
        super("/");
    }
    @Override
    public double operate(double operand1, double operand2) {
        return operand1 / operand2;
    }
}
```

1.  与前一个`Calculator`类一样，这个类也将有一个`operate`方法，但它只会委托给运算符实例。最后，编写一个`main`方法，调用新的计算器几次，打印每次操作的结果：

```java
public class CalculatorWithFixedOperators {
    public static void main (String [] args) {
        System.out.println("1 + 1 = " + new CalculatorWithFixedOperators(1, 1, "+").operate());
        System.out.println("4 - 2 = " + new CalculatorWithFixedOperators(4, 2, "-").operate());
        System.out.println("1 x 2 = " + new CalculatorWithFixedOperators(1, 2, "x").operate());
        System.out.println("10 / 2 = " + new CalculatorWithFixedOperators(10, 2, "/").operate());
    }
    private final double operand1;
    private final double operand2;
    // The current operator
    private final Operator operator;
    // All possible operations
    private final Division division = new Division();
    private final Multiplication multiplication = new Multiplication();
    private final Operator sum = new Operator();
    private final Subtraction subtraction = new Subtraction();
    public CalculatorWithFixedOperators(double operand1, double operand2, String operator) {
        this.operand1 = operand1;
        this.operand2 = operand2;
        if (subtraction.matches(operator)) {
            this.operator = subtraction;
        } else if (multiplication.matches(operator)) {
            this.operator = multiplication;
        } else if (division.matches(operator)) {
            this.operator = division;
        } else {
            this.operator = sum;
        }
    }
    public double operate() {
        return operator.operate(operand1, operand2);
    }
}
```

### 活动 15：理解 Java 中的继承和多态

解决方案：

1.  创建一个继承自`Animal`的`Cat`类：

```java
 public class Cat extends Animal {
```

1.  创建实例变量`owner`，`numberOfTeeth`和`age`如下：

```java
//Fields specific to the Cat family
String owner;
int numberOfTeeth;
int age;
```

1.  创建`main()`方法如下：

```java
public static void main(String[] args){
Cat myCat = new Cat();
//Since Cat inherits from Animal, we have access to it's methods and fields
//We don't need to redefine these methods and fields
myCat.setFamily("Cat");
myCat.setName("Puppy");
myCat.ears = 2;
myCat.legs = 4;
myCat.eyes = 2;
System.out.println(myCat.getFamily());
System.out.println(myCat.getName());
System.out.println(myCat.ears);
System.out.println(myCat.legs);
System.out.println(myCat.eyes);
}
}
```

输出如下

```java
Cat
Puppy
2
4
2
```

### 第 5 课：深入面向对象编程

### 活动 16：在 Java 中创建和实现接口

解决方案：

1.  从我们之前的课程中打开`Animals`项目。

1.  创建一个名为`AnimalBehavior`的新接口。

1.  在此创建两个方法`void move()`和`void makeSound()。`

1.  创建一个名为`Cow`的新的`public`类，并实现`AnimalBehavior`接口。重写这两个方法，但现在先留空。

1.  在`Cow`类中，创建两个字段，如下所示：

```java
public class Cow implements AnimalBehavior, AnimalListener {
String sound;
String movementType;
```

编辑重写的方法，使其如下所示：

```java
@Override
public void move() {
    this.movementType = "Walking";
    this.onAnimalMoved();
}
@Override
public void makeSound() {
    this.sound = "Moo";
    this.onAnimalMadeSound();
}
```

1.  创建另一个名为`AnimalListener`的接口，其中包含以下方法：

```java
public interface AnimalListener {
   void onAnimalMoved();
   void onAnimalMadeSound();
}
```

1.  让`Cow`类也实现这个接口。确保你重写接口中的两个方法。

1.  编辑两个方法，使其如下所示：

```java
@Override
   public void onAnimalMoved() {
       System.out.println("Animal moved: " + this.movementType);
   }
@Override
public void onAnimalMadeSound() {
    System.out.println("Sound made: " + this.sound);
}
```

1.  最后，创建一个`main`方法来测试你的代码：

```java
public static void main(String[] args){
   Cow myCow = new Cow();
   myCow.move();
   myCow.makeSound();
}
}
```

1.  运行`Cow`类并查看输出。它应该看起来像这样：

```java
Animal moved: Walking
Sound made: Moo
```

### 活动 17：使用 instanceof 和类型转换

解决方案：

1.  导入`Random`包以生成随机员工：

```java
import java.util.Random;
```

1.  创建一个`EmployeeLoader`类，作为数据源，如下所示：

```java
public class EmployeeLoader {
```

1.  声明一个静态伪随机生成器如下：

```java
private static Random random = new Random(15);
```

1.  生成一个新的随机选择的员工如下：

```java
public static Employee getEmployee() {
        int nextNumber = random.nextInt(4);
        switch(nextNumber) {
            case 0:
                // A sales person with total sales between 5000 and 1550000
                double grossSales = random.nextDouble() * 150000 + 5000;
                return new SalesWithCommission(grossSales);
            case 1:
                return new Manager();
            case 2:
                return new Engineer();
            case 3:
                return new Sales();
            default:
                return new Manager();
        }
    }
```

1.  创建另一个名为`SalesWithCommission`的文件，该文件扩展`Sales`。添加一个接收毛销售额作为 double 的构造函数，并将其存储为字段。还添加一个名为`getCommission`的方法，该方法返回毛销售额乘以 15%（0.15）的 double：

```java
public class SalesWithCommission extends Sales implements Employee {
    private final double grossSales;
    public SalesWithCommission(double grossSales) {
        this.grossSales = grossSales;
    }
    public double getCommission() {
        return grossSales * 0.15;
    }
}
```

1.  编写一个名为`ShowSalaryAndCommission`的类，其中包含`main()`方法，该方法在`for`循环中重复调用`getEmployee()`并打印有关员工工资和税收的信息。如果员工是`SalesWithCommission`的实例，还要打印他的佣金：

```java
public class ShowSalaryAndCommission {
    public static void main (String [] args) {
        for (int i = 0; i < 10; i++) {
            Employee employee = EmployeeLoader.getEmployee();
            System.out.println("--- " + employee.getClass().getName());
            System.out.println("Net Salary: " + employee.getNetSalary());
            System.out.println("Tax: " + employee.getTax());
            if (employee instanceof SalesWithCommission) {
                // Cast to sales with commission
                SalesWithCommission sales = (SalesWithCommission) employee;
                System.out.println("Commission: " + sales.getCommission());
            }
        }
    }
}
```

### 活动 18：理解 Java 中的类型转换

解决方案：

1.  打开我们的`Animals`项目。

1.  创建一个名为`AnimalTest`的新类，并在其中创建`main`方法：

```java
public class AnimalTest {
   public static void  main(String[] args){
   }
}
```

1.  在`main`方法中，创建两个变量：

```java
Cat cat = new Cat();
Cow cow = new Cow();
```

1.  打印`cat`的所有者：

```java
System.out.println(cat.owner);
```

1.  将`cat`向上转型为`Animal`，再次尝试打印所有者。你得到了什么错误？为什么？

```java
Animal animal = (Animal)cat;
System.out.println(animal.owner);
```

错误消息如下：

![](img/C09581_05_07.jpg)

###### 图 5.7：在向上转型时访问子类变量时出现异常

原因：由于我们进行了向上转型，所以我们不能再访问子类的变量。

1.  打印`cow`的声音：

```java
System.out.println(cow.sound);
```

1.  尝试将`cow`向上转型为`Animal`。为什么会出错？为什么？

```java
Animal animal1 = (Animal)cow;
```

错误消息如下：

![](img/C09581_05_08.jpg)

###### 图 5.8：将 cow 向上转型为 Animal 时出现异常

原因：牛没有继承自 Animal 类，所以它们不共享相同的层次树。

1.  将`animal`向下转型为`cat1`并再次打印所有者：

```java
Cat cat1 = (Cat)animal;
System.out.println(cat1.owner);
```

1.  完整的`AnimalTest`类应该如下所示：

```java
public class AnimalTest {
   public static void  main(String[] args){
       Cat cat = new Cat();
       Cow cow = new Cow();
       System.out.println(cat.owner);

       Animal animal = (Animal)cat;
       //System.out.println(animal.owner);
       System.out.println(cow.sound);
       //Animal animal1 = (Animal)cow;
       Cat cat1 = (Cat)animal;
       System.out.println(cat1.owner);
   }
}
```

输出如下：

![](img/C09581_05_09.jpg)

###### 图 5.9：AnimalTest 类的输出

### 活动 19：在 Java 中实现抽象类和方法

解决方案：

1.  创建一个名为`Hospital`的新项目并打开它。

1.  在`src`文件夹中，创建一个名为`Person`的抽象类：

```java
public abstract class Patient {
}
```

1.  创建一个返回医院中人员类型的抽象方法。命名此方法为`getPersonType()`，返回一个字符串：

```java
public abstract String getPersonType();
```

我们已经完成了抽象类和方法。现在，我们将继承它并实现这个抽象方法。

1.  创建一个名为 Doctor 的继承自 Person 类的新类：

```java
public class Doctor extends Patient {
}
```

1.  在我们的`Doctor`类中重写`getPersonType`抽象方法。返回字符串"`Arzt`"。这是德语中的医生：

```java
@Override
public String getPersonType() {
   return "Arzt";
}
```

1.  创建另一个名为`Patient`的类来代表医院中的患者。同样，确保该类继承自`Person`并重写`getPersonType`方法。返回"`Kranke`"。这是德语中的患者：

```java
public class People extends Patient{
   @Override
   public String getPersonType() {
       return "Kranke";
   }
}
```

现在，我们有了两个类。我们将使用第三个测试类来测试我们的代码。

1.  创建第三个名为`HospitalTest`的类。我们将使用这个类来测试我们之前创建的两个类。

1.  在`HospitalTest`类中，创建`main`方法：

```java
public class HospitalTest {
   public static void main(String[] args){

   }
}
```

1.  在`main`方法中，创建一个`Doctor`的实例和另一个`Patient`的实例：

```java
Doctor doctor = new Doctor();
People people = new People();
```

1.  尝试为每个对象调用`getPersonType`方法并将其打印到控制台。输出是什么？

```java
String str = doctor.getPersonType();
String str1 = patient.getPersonType();
System.out.println(str);
System.out.println(str1);
```

输出如下：

![](img/C09581_05_10.jpg)

###### 图 5.10：调用 getPersonType()的输出

### 活动 20：使用抽象类封装通用逻辑

解决方案：

1.  创建一个抽象类`GenericEmployee`，它具有一个接收总工资并将其存储在字段中的构造函数。它应该实现 Employee 接口，并具有两个方法：`getGrossSalary()`和`getNetSalary()`。第一个将只返回传递给构造函数的值。后者将返回总工资减去调用`getTax()`方法的结果：

```java
public abstract class GenericEmployee implements Employee {
    private final double grossSalary;
    public GenericEmployee(double grossSalary) {
        this.grossSalary = grossSalary;
    }
    public double getGrossSalary() {
        return grossSalary;
    }
    @Override
    public double getNetSalary() {
        return grossSalary - getTax();
    }
}
```

1.  创建每种类型员工的新通用版本：`GenericEngineer`。它将需要一个接收总工资并将其传递给超级构造函数的构造函数。它还需要实现`getTax()`方法，返回每个类的正确税值：

```java
public class GenericEngineer extends GenericEmployee {
    public GenericEngineer(double grossSalary) {
        super(grossSalary);
    }
    @Override
    public double getTax() {
        return (22.0/100) * getGrossSalary();
    }
}
```

1.  创建每种类型员工的新通用版本：`GenericManager`。它将需要一个接收总工资并将其传递给超级构造函数的构造函数。它还需要实现`getTax()`方法，返回每个类的正确税值：

```java
public class GenericManager extends GenericEmployee {
    public GenericManager(double grossSalary) {
        super(grossSalary);
    }
    @Override
    public double getTax() {
        return (28.0/100) * getGrossSalary();
    }
}
```

1.  创建每种类型员工的新通用版本：`GenericSales`。它将需要一个接收总工资并将其传递给超级构造函数的构造函数。它还需要实现`getTax()`方法，返回每个类的正确税值：

```java
public class GenericSales extends GenericEmployee {
    public GenericSales(double grossSalary) {
        super(grossSalary);
    }
    @Override
    public double getTax() {
        return (19.0/100) * getGrossSalary();
    }
}
```

1.  创建每种类型员工的新通用版本：`GenericSalesWithCommission`。它将需要一个接收总工资并将其传递给超级构造函数的构造函数。它还需要实现`getTax()`方法，返回每个类的正确税值。记得在`GenericSalesWithCommission`类中也接收总销售额，并添加计算佣金的方法：

```java
public class GenericSalesWithCommission extends GenericEmployee {
    private final double grossSales;
    public GenericSalesWithCommission(double grossSalary, double grossSales) {
        super(grossSalary);
        this.grossSales = grossSales;
    }
    public double getCommission() {
        return grossSales * 0.15;
    }
    @Override
    public double getTax() {
        return (19.0/100) * getGrossSalary();
    }
}
```

1.  向`EmployeeLoader`类添加一个新方法`getEmployeeWithSalary`。此方法将在返回之前为新创建的员工生成一个介于 70,000 和 120,000 之间的随机工资。记得在创建`GenericSalesWithCommission`员工时也提供总销售额：

```java
public static Employee getEmployeeWithSalary() {
        int nextNumber = random.nextInt(4);
        // Random salary between 70,000 and 70,000 + 50,000
        double grossSalary = random.nextDouble() * 50000 + 70000;
        switch(nextNumber) {
            case 0:
                // A sales person with total sales between 5000 and 1550000
                double grossSales = random.nextDouble() * 150000 + 5000;
                return new GenericSalesWithCommission(grossSalary, grossSales);
            case 1:
                return new GenericManager(grossSalary);
            case 2:
                return new GenericEngineer(grossSalary);
            case 3:
                return new GenericSales(grossSalary);
            default:
                return new GenericManager(grossSalary);
        }
    }
}
```

1.  编写一个应用程序，从`for`循环内多次调用`getEmployeeWithSalary`方法。此方法将像上一个活动中的方法一样工作：打印所有员工的净工资和税收。如果员工是`GenericSalesWithCommission`的实例，还要打印他的佣金。

```java
public class UseAbstractClass {
    public static void main (String [] args) {
        for (int i = 0; i < 10; i++) {
            Employee employee = EmployeeLoader.getEmployeeWithSalary();
            System.out.println("--- " + employee.getClass().getName());
            System.out.println("Net Salary: " + employee.getNetSalary());
            System.out.println("Tax: " + employee.getTax());
            if (employee instanceof GenericSalesWithCommission) {
                // Cast to sales with commission
                GenericSalesWithCommission sales = (GenericSalesWithCommission) employee;
                System.out.println("Commission: " + sales.getCommission());
            }
        }
    }
}
```

## 第 6 课：数据结构、数组和字符串

### 活动 21：在数组中找到最小的数字

解决方案：

1.  在名为`ExampleArray`的新类文件中设置`main`方法：

```java
public class ExampleArray {
  public static void main(String[] args) {
  }
}
```

1.  创建一个包含 20 个数字的数组：

```java
double[] array = {14.5, 28.3, 15.4, 89.0, 46.7, 25.1, 9.4, 33.12, 82, 11.3, 3.7, 59.99, 68.65, 27.78, 16.3, 45.45, 24.76, 33.23, 72.88, 51.23};
```

1.  将最小的浮点数设为第一个数字

```java
double min = array[0];
```

1.  创建一个 for 循环来检查数组中的所有数字

```java
for (doublefloat f : array) {
}
```

1.  使用 if 来测试每个数字是否小于最小值。如果小于最小值，则将该数字设为新的最小值：

```java
if (f < min)
min = f;
}
```

1.  循环完成后，打印出最小的数字：

```java
System.out.println("The lowest number in the array is " + min);
}
}
```

完整的代码应该如下所示。

```java
public class ExampleArray {
        public static void main(String[] args) {
            double[] array = {14.5, 28.3, 15.4, 89.0, 46.7, 25.1, 9.4, 33.12, 82, 11.3, 3.7, 59.99, 68.65, 27.78, 16.3, 45.45, 24.76, 33.23, 72.88, 51.23};
            double min = array[0];
            for (double f : array) {
                if (f < min)
                    min = f;
            }
            System.out.println("The lowest number in the array is " + min);
        }
}
```

### 活动 22：带有操作符数组的计算器

解决方案：

1.  创建一个名为`Operators`的类，它将包含基于字符串确定要使用的操作符的逻辑。在这个类中创建一个`public`常量字段`default_operator`，它将是`Operator`类的一个实例。然后创建另一个名为`operators`的常量字段，类型为`Operator`数组，并使用每个操作符的实例进行初始化：

```java
public class Operators {
    public static final Operator DEFAULT_OPERATOR = new Operator();
    public static final Operator [] OPERATORS = {
        new Division(),
        new Multiplication(),
        DEFAULT_OPERATOR,
        new Subtraction(),
    };
```

1.  在`Operators`类中，添加一个名为`findOperator`的`public static`方法，该方法接收操作符作为字符串并返回`Operator`的实例。在其中迭代可能的操作符数组，并对每个操作符使用`matches`方法，返回所选操作符，如果没有匹配任何操作符，则返回默认操作符：

```java
public static Operator findOperator(String operator) {
        for (Operator possible : OPERATORS) {
            if (possible.matches(operator)) {
                return possible;
            }
        }
        return DEFAULT_OPERATOR;
    }
}
```

1.  创建一个新的`CalculatorWithDynamicOperator`类，其中包含三个字段：`operand1`和`operator2`为`double`类型，`operator`为`Operator`类型：

```java
public class CalculatorWithDynamicOperator {
    private final double operand1;
    private final double operand2;
    // The current operator
    private final Operator operator;
```

1.  添加一个接收三个参数的构造函数：`operand1`和`operand2`的类型为`double`，`operator`为 String 类型。在构造函数中，不要使用 if-else 来选择操作符，而是使用`Operators.findOperator`方法来设置操作符字段：

```java
public CalculatorWithDynamicOperator(double operand1, double operand2, String operator) {
        this.operand1 = operand1;
        this.operand2 = operand2;
        this.operator = Operators.findOperator(operator);
    }
    public double operate() {
        return operator.operate(operand1, operand2);
    }
```

1.  添加一个`main`方法，在其中多次调用`Calculator`类并打印结果：

```java
public static void main (String [] args) {
        System.out.println("1 + 1 = " + new CalculatorWithDynamicOperator(1, 1, "+").operate());
        System.out.println("4 - 2 = " + new CalculatorWithDynamicOperator(4, 2, "-").operate());
        System.out.println("1 x 2 = " + new CalculatorWithDynamicOperator(1, 2, "x").operate());
        System.out.println("10 / 2 = " + new CalculatorWithDynamicOperator(10, 2, "/").operate());
    }
}
```

### 活动 23：使用 ArrayList

解决方案：

1.  从`java.util`导入`ArrayList`和`Iterator`：

```java
import java.util.ArrayList;
import java.util.Iterator;
```

1.  创建一个名为`StudentsArray`的新类：

```java
public class StudentsArray extends Student{
```

1.  在`main`方法中定义一个`Student`对象的`ArrayList`。插入 4 个学生实例，用我们之前创建的不同类型的构造函数实例化：

```java
public static void main(String[] args){
       ArrayList<Student> students = new ArrayList<>();
       Student james = new Student();
       james.setName("James");
       Student mary = new Student();
       mary.setName("Mary");
       Student jane = new Student();
       jane.setName("Jane");
       Student pete = new Student();
       pete.setName("Pete");
       students.add(james);
       students.add(mary);
       students.add(jane);
       students.add(pete);
```

1.  为您的列表创建一个迭代器并打印每个学生的姓名：

```java
       Iterator studentsIterator = students.iterator();
       while (studentsIterator.hasNext()){
           Student student = (Student) studentsIterator.next();
           String name = student.getName();
           System.out.println(name);
       }    
```

1.  清除所有的“学生”：

```java
       students.clear();
   }
}
```

最终的代码应该如下所示：

```java
import java.util.ArrayList;
import java.util.Iterator;
public class StudentsArray extends Student{
   public static void main(String[] args){
       ArrayList<Student> students = new ArrayList<>();
       Student james = new Student();
       james.setName("James");
       Student mary = new Student();
       mary.setName("Mary");
       Student jane = new Student();
       jane.setName("Jane");
       students.add(james);
       students.add(mary);
       students.add(jane);
       Iterator studentsIterator = students.iterator();
       while (studentsIterator.hasNext()){
           Student student = (Student) studentsIterator.next();
           String name = student.getName();
           System.out.println(name);
       }

       students.clear();
   }
}
```

输出如下：

![](img/C09581_06_30.jpg)

###### 图 6.30：StudentsArray 类的输出

### 活动 24：输入一个字符串并将其长度输出为数组

解决方案：

1.  导入`java.util.Scanner`包：

```java
import java.util.Scanner;
```

1.  创建一个名为`NameTell`的公共类和一个`main`方法：

```java
public class NameTell
{
  public static void main(String[] args)
  {
```

1.  使用`Scanner`和`nextLine`在提示“输入您的姓名：”处输入一个字符串

```java
System.out.print("Enter your name:");
Scanner sc = new Scanner(System.in);
String name = sc.nextLine();
```

1.  计算字符串的长度并找到第一个字符：

```java
int num = name.length();
char c = name.charAt(0);
```

1.  打印一个输出：

```java
System.out.println("\n Your name has " + num + " letters including spaces.");
System.out.println("\n The first letter is: " + c);
  }
}
```

输出如下：

![](img/C09581_06_31.jpg)

###### 图 6.31：NameTell 类的输出

### 活动 25：计算器从输入中读取

解决方案：

1.  创建一个名为`CommandLineCalculator`的新类，其中包含一个`main()`方法：

```java
import java.util.Scanner;
public class CommandLineCalculator {
    public static void main (String [] args) throws Exception {
        Scanner scanner = new Scanner(System.in);
```

1.  使用无限循环使应用程序保持运行，直到用户要求退出。

```java
while (true) {
            printOptions();
            String option = scanner.next();
            if (option.equalsIgnoreCase("Q")) {
                break;
            }
```

1.  收集用户输入以决定要执行的操作。如果操作是**Q**或**q**，则退出循环：

```java
System.out.print("Type first operand: ");
            double operand1 = scanner.nextDouble();
            System.out.print("Type second operand: ");
            double operand2 = scanner.nextDouble();
            Operator operator = Operators.findOperator(option);
            double result = operator.operate(operand1, operand2);
            System.out.printf("%f %s %f = %f\n", operand1, operator.operator, operand2, result);
            System.out.println();
        }
    }
```

1.  如果操作是其他任何操作，请查找操作符并请求另外两个输入，这些输入将是覆盖它们为 double 的操作数：

```java
  private static void printOptions() {
        System.out.println("Q (or q) - To quit");
        System.out.println("An operator. If not supported, will use sum.");
        System.out.print("Type your option: ");
    }
}
```

在找到的操作符上调用`operate`方法并将结果打印到控制台。

### 活动 26：从字符串中删除重复字符

解决方案：

1.  创建一个名为 Unique 的类，如下所示：

```java
public class Unique {
```

1.  创建一个名为`removeDups`的新方法，该方法接受并返回一个字符串。这就是我们的算法所在的地方。此方法应该是`public`和`static`的：

```java
public static String removeDups(String string){
```

1.  在方法内部，检查字符串是否为 null、空或长度为 1。如果这些情况中的任何一个为真，则只返回原始字符串，因为不需要检查：

```java
if (string == null)
           return string;
       if (string == "")
           return string;
       if (string.length() == 1)
           return string;
```

1.  创建一个名为`result`的空字符串。这将是要返回的唯一字符串：

```java
String result = "";
```

1.  创建一个从`0`到传递到方法中的字符串长度的 for 循环。在`for`循环内，获取字符串当前索引处的字符。将变量命名为`c`。还创建一个名为`isDuplicate`的`boolean`并将其初始化为`false`。当我们遇到重复时，我们将其更改为`true`。

```java
for (int i = 0; i < string.length() ; i++){
           char c = string.charAt(i);
           boolean isDuplicate = false;
```

1.  创建另一个嵌套的`for`循环，从`0`到`result`的`length()`。在内部的`for`循环中，还要获取结果当前索引处的字符。将其命名为`d`。比较`c`和`d`。如果它们相等，则将`isDuplicate`设置为`true`并`break`。关闭内部的`for`循环并进入第一个`for`循环。检查`isDuplicate`是否为 false。如果是，则将`c`附加到结果。退出第一个`for`循环并返回结果。这就结束了我们的算法：

```java
for (int j = 0; j < result.length(); j++){
               char d = result.charAt(j);
               if (c  == d){ //duplicate found
                   isDuplicate = true;
                   break;
               }
           }
           if (!isDuplicate)
               result += ""+c;
       }
       return result;
   }
```

1.  创建一个如下所示的`main()`方法：

```java
public static void main(String[] args){
       String a = "aaaaaaa";
       String b = "aaabbbbb";
       String c = "abcdefgh";
       String d = "Ju780iu6G768";
       System.out.println(removeDups(a));
       System.out.println(removeDups(b));
       System.out.println(removeDups(c));
       System.out.println(removeDups(d));
   }
}
```

输出如下：

![](img/C09581_06_32.jpg)

###### 图 6.32：Unique 类的输出

完整的代码如下：

```java
public class Unique {
   public static String removeDups(String string){
       if (string == null)
           return string;
       if (string == "")
           return string;
       if (string.length() == 1)
           return string;
      String result = "";
       for (int i = 0; i < string.length() ; i++){
           char c = string.charAt(i);
           boolean isDuplicate = false;
           for (int j = 0; j < result.length(); j++){
               char d = result.charAt(j);
               if (c  == d){ //duplicate found
                   isDuplicate = true;
                   break;
               }
           }
           if (!isDuplicate)
               result += ""+c;
       }
       return result;
   }
public static void main(String[] args){
       String a = "aaaaaaa";
       String b = "aaabbbbb";
       String c = "abcdefgh";
       String d = "Ju780iu6G768";
       System.out.println(removeDups(a));
       System.out.println(removeDups(b));
       System.out.println(removeDups(c));
       System.out.println(removeDups(d));
   }
}
```

输出如下：

![图 6.30：Unique 类的输出](img/C09581_06_33.jpg)

###### 图 6.33：Unique 类的输出

## 第 7 课：Java 集合框架和泛型

### 活动 27：使用具有初始容量的数组从 CSV 中读取用户

解决方案：

1.  创建一个名为`UseInitialCapacity`的类，其中包含一个`main()`方法

```java
public class UseInitialCapacity {
  public static final void main (String [] args) throws Exception {
  }
}
```

1.  添加一个常量字段，它将是数组的初始容量。当数组需要增长时，也将使用它：

```java
private static final int INITIAL_CAPACITY = 5;
```

1.  添加一个`static`方法，用于调整数组大小。它接收两个参数：一个用户数组和一个表示数组新大小的`int`。它还应返回一个用户数组。使用`System.arraycopy`实现调整大小算法，就像在上一个练习中所做的那样。请注意，新大小可能小于传入数组的当前大小：

```java
private static User[] resizeArray(User[] users, int newCapacity) {
  User[] newUsers = new User[newCapacity];
  int lengthToCopy = newCapacity > users.length ? users.length : newCapacity;
  System.arraycopy(users, 0, newUsers, 0, lengthToCopy);
  return newUsers;
}
```

1.  编写另一个`static`方法，将用户从 CSV 文件加载到数组中。它需要确保数组有能力接收从文件加载的用户。您还需要确保在加载用户后，数组不包含额外的插槽：

```java
public static User[] loadUsers(String pathToFile) throws Exception {
  User[] users = new User[INITIAL_CAPACITY];
  BufferedReader lineReader = new BufferedReader(new FileReader(pathToFile));
  try (CSVReader reader = new CSVReader(lineReader)) {
    String [] row = null;
    while ( (row = reader.readRow()) != null) {
      // Reached end of the array
      if (users.length == reader.getLineCount()) {
        // Increase the array by INITIAL_CAPACITY
        users = resizeArray(users, users.length + INITIAL_CAPACITY);
      }
      users[users.length - 1] = User.fromValues(row);
    } // end of while

    // If read less rows than array capacity, trim it
    if (reader.getLineCount() < users.length - 1) {
      users = resizeArray(users, reader.getLineCount());
    }
  } // end of try

  return users;
}
```

1.  在`main`方法中，调用加载用户的方法并打印加载的用户总数：

```java
User[] users = loadUsers(args[0]);
System.out.println(users.length);
```

1.  添加导入：

```java
import java.io.BufferedReader;
import java.io.FileReader;
```

输出如下：

```java
27
```

### 活动 28：使用 Vector 读取真实数据集

解决方案：

1.  在开始之前，将您的`CSVLoader`更改为支持没有标题的文件。为此，添加一个新的构造函数，接收一个`boolean`，告诉它是否应该忽略第一行：

```java
public CSVReader(BufferedReader reader, boolean ignoreFirstLine) throws IOException {
  this.reader = reader;
  if (ignoreFirstLine) {
    reader.readLine();
  }
}
```

1.  将旧构造函数更改为调用此新构造函数，传递 true 以忽略第一行。这将避免您返回并更改任何现有代码：

```java
public CSVReader(BufferedReader reader) throws IOException {
  this(reader, true);
}
```

1.  创建一个名为`CalculateAverageSalary`的类，其中包含`main`方法：

```java
public class CalculateAverageSalary {
  public static void main (String [] args) throws Exception {
  }
}
```

1.  创建另一个方法，从 CSV 中读取数据并将工资加载到 Vector 中。该方法应在最后返回 Vector：

```java
private static Vector loadWages(String pathToFile) throws Exception {
  Vector result = new Vector();
  FileReader fileReader = new FileReader(pathToFile);
  BufferedReader bufferedReader = new BufferedReader(fileReader);
  try (CSVReader csvReader = new CSVReader(bufferedReader, false)) {
    String [] row = null;
    while ( (row = csvReader.readRow()) != null) {
      if (row.length == 15) { // ignores empty lines
        result.add(Integer.parseInt(row[2].trim()));
      }
    }
  }
  return result;
}
```

1.  在`main`方法中，调用`loadWages`方法并将加载的工资存储在 Vector 中。还要存储应用程序启动时的初始时间：

```java
Vector wages = loadWages(args[0]);
long start = System.currentTimeMillis();
```

1.  初始化三个变量来存储所有工资的最小值、最大值和总和：

```java
int totalWage = 0;
int maxWage = 0;
int minWage = Integer.MAX_VALUE;
```

1.  在`for-each`循环中，处理所有工资，存储最小值、最大值并将其添加到总和中：

```java
for (Object wageAsObject : wages) {
  int wage = (int) wageAsObject;
  totalWage += wage;
  if (wage > maxWage) {
    maxWage = wage;
  }
  if (wage < minWage) {
    minWage = wage;
  }
}
```

1.  最后打印加载的工资数量和加载和处理它们所花费的总时间。还打印平均工资、最低工资和最高工资：

```java
System.out.printf("Read %d rows in %dms\n", wages.size(), System.currentTimeMillis() - start);
System.out.printf("Average, Min, Max: %d, %d, %d\n", totalWage / wages.size(), minWage, maxWage);
```

1.  添加导入：

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.util.Vector;
```

输出如下：

```java
Read 32561 rows in 198ms
Average, Min, Max: 57873, 12285, 1484705
```

### 活动 29：对用户的 Vector 进行迭代

解决方案：

1.  创建一个名为`IterateOnUsersVector`的新类，其中包含`main`方法：

```java
public class IterateOnUsersVector {
  public static void main(String [] args) throws IOException {
  }
}
```

1.  在主方法中，调用`UsersLoader.loadUsersInVector`，传递从命令行传递的第一个参数作为要加载的文件，并将数据存储在 Vector 中：

```java
Vector users = UsersLoader.loadUsersInVector(args[0]);
```

1.  使用`for-each`循环迭代用户 Vector，并将有关用户的信息打印到控制台：

```java
for (Object userAsObject : users) {
  User user = (User) userAsObject;
  System.out.printf("%s - %s\n", user.name, user.email);
}
```

1.  添加导入：

```java
import java.io.IOException;
import java.util.Vector;
```

输出如下：

```java
Bill Gates - william.gates@microsoft.com
Jeff Bezos - jeff.bezos@amazon.com
Marc Benioff - marc.benioff@salesforce.com
Bill Gates - william.gates@microsoft.com
Jeff Bezos - jeff.bezos@amazon.com
Sundar Pichai - sundar.pichai@google.com
Jeff Bezos - jeff.bezos@amazon.com
Larry Ellison - lawrence.ellison@oracle.com
Marc Benioff - marc.benioff@salesforce.com
Larry Ellison - lawrence.ellison@oracle.com
Jeff Bezos - jeff.bezos@amazon.com
Bill Gates - william.gates@microsoft.com
Sundar Pichai - sundar.pichai@google.com
Jeff Bezos - jeff.bezos@amazon.com
Sundar Pichai - sundar.pichai@google.com
Marc Benioff - marc.benioff@salesforce.com
Larry Ellison - lawrence.ellison@oracle.com
Marc Benioff - marc.benioff@salesforce.com
Jeff Bezos - jeff.bezos@amazon.com
Marc Benioff - marc.benioff@salesforce.com
Bill Gates - william.gates@microsoft.com
Sundar Pichai - sundar.pichai@google.com
Larry Ellison - lawrence.ellison@oracle.com
Bill Gates - william.gates@microsoft.com
Larry Ellison - lawrence.ellison@oracle.com
Jeff Bezos - jeff.bezos@amazon.com
Sundar Pichai - sundar.pichai@google.com
```

### 活动 30：使用 Hashtable 对数据进行分组

解决方案：

1.  创建一个名为`GroupWageByEducation`的类，其中包含一个`main`方法：

```java
public class GroupWageByEducation {
  public static void main (String [] args) throws Exception {
  }
}
```

1.  创建一个`static`方法，创建并返回一个键类型为 String，值类型为整数向量的`Hashtable`：

```java
private static Hashtable<String, Vector<Integer>> loadWages(String pathToFile) throws Exception {
  Hashtable<String, Vector<Integer>> result = new Hashtable<>();
  return result;
}
```

1.  在创建`Hashtable`和返回它之间，加载来自 CSV 的行，确保它们具有正确的格式：

```java
FileReader fileReader = new FileReader(pathToFile);
BufferedReader bufferedReader = new BufferedReader(fileReader);
try (CSVReader csvReader = new CSVReader(bufferedReader, false)) {
  String [] row = null;
  while ( (row = csvReader.readRow()) != null) {
    if (row.length == 15) {
    }
  }
}
```

1.  在`while`循环内的`if`中，获取记录的教育水平和工资：

```java
String education = row[3].trim();
int wage = Integer.parseInt(row[2].trim());
```

1.  在`Hashtable`中找到与当前教育水平相对应的 Vector，并将新工资添加到其中：

```java
// Get or create the vector with the wages for the specified education
Vector<Integer> wages = result.getOrDefault(education, new Vector<>());
wages.add(wage);
// Ensure the vector will be in the hashtable next time
result.put(education, wages);
```

1.  在主方法中，调用您的`loadWages`方法，传递命令行的第一个参数作为要加载数据的文件：

```java
Hashtable<String,Vector<Integer>> wagesByEducation = loadWages(args[0]);
```

1.  使用`for-each`循环迭代`Hashtable`条目，并为每个条目获取相应工资的 Vector，并初始化最小值、最大值和总和变量：

```java
for (Entry<String, Vector<Integer>> entry : wagesByEducation.entrySet()) {
  Vector<Integer> wages = entry.getValue();
  int totalWage = 0;
  int maxWage = 0;
  int minWage = Integer.MAX_VALUE;
}
```

1.  初始化变量后，遍历所有工资并存储最小值、最大值和总和：

```java
for (Integer wage : wages) {
  totalWage += wage;
  if (wage > maxWage) {
    maxWage = wage;
  }
  if (wage < minWage) {
    minWage = wage;
  }
}
```

1.  然后，打印找到的指定条目的信息，该条目表示教育水平：

```java
System.out.printf("%d records found for education %s\n", wages.size(), entry.getKey());
System.out.printf("\tAverage, Min, Max: %d, %d, %d\n", totalWage / wages.size(), minWage, maxWage);
```

1.  添加导入：

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.util.Hashtable;
import java.util.Map.Entry;
import java.util.Vector;
```

输出如下：

```java
1067 records found for education Assoc-acdm
        Average, Min, Max: 193424, 19302, 1455435
433 records found for education 12th
        Average, Min, Max: 199097, 23037, 917220
1382 records found for education Assoc-voc
        Average, Min, Max: 181936, 20098, 1366120
5355 records found for education Bachelors
        Average, Min, Max: 188055, 19302, 1226583
51 records found for education Preschool
        Average, Min, Max: 235889, 69911, 572751
10501 records found for education HS-grad
        Average, Min, Max: 189538, 19214, 1268339
168 records found for education 1st-4th
        Average, Min, Max: 239303, 34378, 795830
333 records found for education 5th-6th
        Average, Min, Max: 232448, 32896, 684015
576 records found for education Prof-school
        Average, Min, Max: 185663, 14878, 747719
514 records found for education 9th
        Average, Min, Max: 202485, 22418, 758700
1723 records found for education Masters
        Average, Min, Max: 179852, 20179, 704108
933 records found for education 10th
        Average, Min, Max: 196832, 21698, 766115
413 records found for education Doctorate
        Average, Min, Max: 186698, 19520, 606111
7291 records found for education Some-college
        Average, Min, Max: 188742, 12285, 1484705
646 records found for education 7th-8th
        Average, Min, Max: 188079, 20057, 750972
1175 records found for education 11th
        Average, Min, Max: 194928, 19752, 806316
```

### 活动 31：对用户进行排序

解决方案：

1.  编写一个比较器类来比较用户的 ID：

```java
import java.util.Comparator;
public class ByIdComparator implements Comparator<User> {
  public int compare(User first, User second) {
    if (first.id < second.id) {
      return -1;
    }
    if (first.id > second.id) {
      return 1;
    }
    return 0;
  }
}
```

1.  编写一个比较器类，按电子邮件比较用户：

```java
import java.util.Comparator;
public class ByEmailComparator implements Comparator<User> {
  public int compare(User first, User second) {
    return first.email.toLowerCase().compareTo(second.email.toLowerCase());
  }
}
```

1.  编写一个比较器类，按用户名比较用户：

```java
import java.util.Comparator;
public class ByNameComparator implements Comparator<User> {
  public int compare(User first, User second) {
    return first.name.toLowerCase().compareTo(second.name.toLowerCase());
  }
}
```

1.  创建一个名为`SortUsers`的新类，其中包含一个`main`方法，该方法按电子邮件加载唯一的用户：

```java
public class SortUsers {
  public static void main (String [] args) throws IOException {
    Hashtable<String, User> uniqueUsers = UsersLoader.loadUsersInHashtableByEmail(args[0]);
  }
}
```

1.  加载用户后，将用户转移到用户的 Vector 中，以便保留顺序，因为`Hashtable`不会这样做：

```java
Vector<User> users = new Vector<>(uniqueUsers.values());
```

1.  要求用户选择要按其对用户进行排序的字段，并从标准输入收集输入：

```java
Scanner reader = new Scanner(System.in);
System.out.print("What field you want to sort by: ");
String input = reader.nextLine();
```

1.  使用`switch`语句中的输入来选择要使用的比较器。如果输入无效，则打印友好的消息并退出：

```java
Comparator<User> comparator;
switch(input) {
  case "id":
    comparator = newByIdComparator();
    break;
  case "name":
    comparator = new ByNameComparator();
    break;
  case "email":
    comparator = new ByEmailComparator();
    break;
  default:
    System.out.printf("Sorry, invalid option: %s\n", input);
    return;
}
```

1.  告诉用户你要按什么字段排序，并对用户的向量进行排序：

```java
System.out.printf("Sorting by %s\n", input);
Collections.sort(users, comparator);
```

1.  使用`for-each`循环打印用户：

```java
for (User user : users) {
  System.out.printf("%d - %s, %s\n", user.id, user.name, user.email);
}
```

1.  添加导入：

```java
import java.io.IOException;
import java.util.Collections;
import java.util.Comparator;
import java.util.Hashtable;
import java.util.Scanner;
import java.util.Vector;
```

输出如下：

```java
5 unique users found.
What field you want to sort by: email
Sorting by email
30 - Jeff Bezos, jeff.bezos@amazon.com
50 - Larry Ellison, lawrence.ellison@oracle.com
20 - Marc Benioff, marc.benioff@salesforce.com
40 - Sundar Pichai, sundar.pichai@google.com
10 - Bill Gates, william.gates@microsoft.com
```

## 第 8 课：Java 中的高级数据结构

### 活动 32：在 Java 中创建自定义链表

解决方案：

1.  创建一个名为`SimpleObjLinkedList`的类。

```java
public class SimpleObjLinkedList {
```

1.  创建一个名为 Node 的类，表示链表中的每个元素。每个节点将有一个它需要保存的对象，并且将引用下一个节点。`LinkedList`类将引用头节点，并且可以通过使用`Node.getNext()`遍历到下一个节点。头部是第一个元素，我们可以通过移动当前节点中的`next`来遍历到下一个元素。这样，我们可以遍历到列表的最后一个元素：

```java
static class Node {
Object data;
Node next;
Node(Object d) {
data = d;
next = null;
}
Node getNext() {
return next;
}
void setNext(Node node) {
next = node;
}
Object getData() {
return data;
}
}
```

1.  实现`toString()`方法来表示这个对象。从头节点开始，迭代所有节点，直到找到最后一个节点。在每次迭代中，构造存储在每个节点中的对象的字符串表示：

```java
public String toString() {
String delim = ",";
StringBuffer stringBuf = new StringBuffer();
if (head == null)
return "LINKED LIST is empty";    
Node currentNode = head;
while (currentNode != null) {
stringBuf.append(currentNode.getData());
currentNode = currentNode.getNext();
if (currentNode != null)
stringBuf.append(delim);
}
return stringBuf.toString();
}
```

1.  实现`add(Object item)`方法，以便将任何项目/对象添加到此列表中。通过传递`newItem = new Node(item)` Item 来构造一个新的 Node 对象。从头节点开始，爬到列表的末尾。在最后一个节点中，将下一个节点设置为我们新创建的节点（`newItem`）。增加索引：

```java
// appends the specified element to the end of this list.    
public void add(Object element) {
// create a new node
Node newNode = new Node(element);
//if head node is empty, create a new node and assign it to Head
//increment index and return
if (head == null) {
head = newNode;
return;
}
Node currentNode = head;
// starting at the head node
// move to last node
while (currentNode.getNext() != null) {
currentNode = currentNode.getNext();
}
// set the new node as next node of current
currentNode.setNext(newNode);
}
```

1.  实现`get(Integer index)`方法，根据索引从列表中检索项目。索引不能小于 0。编写一个逻辑来爬到指定的索引，识别节点，并从节点返回值。

```java
public Object get(int index) {
// Implement the logic returns the element
// at the specified position in this list.
if (head == null || index < 0)
return null;        
if (index == 0){
return head.getData();
}    
Node currentNode = head.getNext();
for (int pos = 0; pos < index; pos++) {
currentNode = currentNode.getNext();
if (currentNode == null)
return null;
}
return currentNode.getData();
}
```

1.  实现`remove(Integer index)`方法，根据索引从列表中删除项目。编写逻辑来爬到指定索引之前的节点并识别节点。在这个节点中，将`next`设置为`getNext()`。如果找到并删除了元素，则返回 true。如果未找到元素，则返回 false：

```java
public boolean remove(int index) {
if (index < 0)
return false;
if (index == 0)
{
head = null;
return true;
}
Node currentNode = head;
for (int pos = 0; pos < index-1; pos++) {
if (currentNode.getNext() == null)
return false;
currentNode = currentNode.getNext();
}    
currentNode.setNext(currentNode.getNext().getNext());
return true;
}
```

1.  创建一个指向头节点的 Node 类型的成员属性。编写一个`main`方法，创建一个`SimpleObjLinkedList`对象，并依次向其中添加五个字符串（"INPUT-1"，"INPUT-2"，"INPUT-3"，"INPUT-4"，"INPUT-5"）。打印`SimpleObjLinkedList`对象。在`main`方法中，使用`get(2)`从列表中获取项目并打印检索到的项目的值。还要从列表中删除项目`remove(2)`并打印列表的值。列表中应该已经删除了一个元素：

```java
Node head;    
    public static void main(String[] args) {
        SimpleObjLinkedList list = new SimpleObjLinkedList();
        list.add("INPUT-1");
        list.add("INPUT-2");
        list.add("INPUT-3");
        list.add("INPUT-4");
        list.add("INPUT-5");
        System.out.println(list);
        System.out.println(list.get(2));
        list.remove(3);
        System.out.println(list);
}
}
```

输出如下：

```java
[INPUT-1 ,INPUT-2 ,INPUT-3 ,INPUT-4 ,INPUT-5 ]
INPUT-3
[INPUT-1 ,INPUT-2 ,INPUT-3 ,INPUT-5 ]
```

### 活动 33：实现 BinarySearchTree 类中的方法，以找到 BST 中的最高和最低值

解决方案：

1.  使用我们在上一个练习中使用的相同类：`BinarySearchTree`。添加一个新方法`int getLow()`，以找到 BST 中的最低值并返回它。正如我们所了解的 BST，最左边的节点将是所有值中最低的。迭代所有左节点，直到达到一个空的左节点，并获取其根的值：

```java
    /**
     * As per BST, the left most node will be lowest of the all. iterate all the
     * left nodes until we reach empty left and get the value of it root.
     * @return int lowestValue
     */
    public int getLow() {
        Node current = parent;
        while (current.left != null) {
            current = current.left;
        }
        return current.data;
    }
```

1.  添加一个新方法`int getHigh()`，以找到 BST 中的最高值并返回它。正如我们所了解的 BST，最右边的节点将是所有值中最高的。迭代所有右节点，直到达到一个空的右节点，并获取其根的值：

```java
    /**
     * As per BST, the right most node will be highest of the all. iterate all
     * the right nodes until we reach empty right and get the value of it root.
     * @return int highestValue
     */
    public int getHigh() {
        Node current = parent;
        while (current.right != null) {
            current = current.right;
        }
        return current.data;
    }
```

1.  在`main`方法中，构造一个 BST，向其中添加值，然后通过调用`getLow()`和`getHigh()`来打印最高和最低的值：

```java
/**
     * Main program to demonstrate the BST functionality.
     * - Adding nodes
     * - finding High and low 
     * - Traversing left and right
     * @param args
     */
    public static void main(String args[]) {
        BinarySearchTree bst = new BinarySearchTree();
        // adding nodes into the BST
        bst.add(32);
        bst.add(50);
        bst.add(93);
        bst.add(3);
        bst.add(40);
        bst.add(17);
        bst.add(30);
        bst.add(38);
        bst.add(25);
        bst.add(78);
        bst.add(10);
        //printing lowest and highest value in BST
        System.out.println("Lowest value in BST :" + bst.getLow());
        System.out.println("Highest value in BST :" + bst.getHigh());
    }
```

输出如下：

```java
Lowest value in BST :3
Highest value in BST :93
```

### 活动 34：使用枚举来保存大学部门的详细信息

解决方案：

1.  使用`enum`关键字创建一个`DeptEnum`枚举。添加两个私有属性（`String deptName`和`int deptNo`）来保存枚举中的值。重写一个构造函数，以取一个缩写和`deptNo`并将其放入成员变量中。添加符合构造函数的枚举常量：

```java
    public enum DeptEnum {
    BE("BACHELOR OF ENGINEERING", 1), BCOM("BACHELOR OF COMMERCE", 2), BSC("BACHELOR OF SCIENCE",
            3), BARCH("BACHELOR OF ARCHITECTURE", 4), DEFAULT("BACHELOR", 0);
    private String acronym;
    private int deptNo;
    DeptEnum(String accr, int deptNo) {
        this.accronym = acr;
        this.deptNo = deptNo;
    }
```

1.  为`deptName`和`deptNo`添加 getter 方法：

```java
    public String getAcronym() {
        return acronym;
    }
    public int getDeptNo() {
        return deptNo;
    }
```

1.  让我们编写一个`main`方法和一个示例程序来演示枚举的用法：

```java
public static void main(String[] args) {
// Fetching the Enum using Enum name as string
DeptEnum env = DeptEnum.valueOf("BE");
System.out.println(env.getAcronym() + " : " + env.getDeptNo());
// Printing all the values of Enum
for (DeptEnum e : DeptEnum.values()) {
System.out.println(e.getAcronym() + " : " + e.getDeptNo());    }
// Compare the two enums using the the equals() method or using //the == operator.                
System.out.println(DeptEnum.BE == DeptEnum.valueOf("BE"));
}
}
```

1.  输出：

```java
BACHELOR OF ENGINEERING : 1
BACHELOR OF ENGINEERING : 1
BACHELOR OF COMMERCE : 2
BACHELOR OF SCIENCE : 3
BACHELOR OF ARCHITECTURE : 4
BACHELOR : 0
True
```

### 活动 35：实现反向查找

解决方案：

1.  创建一个枚举`App`，声明常量 BE、BCOM、BSC 和 BARC，以及它们的全称和部门编号。

```java
public enum App {
    BE("BACHELOR OF ENGINEERING", 1), BCOM("BACHELOR OF COMMERCE", 2), BSC("BACHELOR OF SCIENCE", 3), BARCH("BACHELOR OF ARCHITECTURE", 4), DEFAULT("BACHELOR", 0);
```

1.  还声明两个私有变量`accronym`和`deptNo`。

```java
    private String accronym;
    private int deptNo;
```

1.  创建一个带参数的构造函数，并将变量`accronym`和`deptNo`分配为传递的值。

```java
    App(String accr, int deptNo) {
        this.accronym = accr;
        this.deptNo = deptNo;
    }
```

1.  声明一个公共方法`getAccronym()`，返回变量`accronym`，以及一个公共方法`getDeptNo()`，返回变量`deptNo`。

```java
    public String getAccronym() {
        return accronym;
    }
    public int getDeptNo() {
        return deptNo;
    }
```

1.  实现反向查找，接受课程名称，并在`App`枚举中搜索相应的缩写。

```java
    //reverse lookup 
    public static App get(String accr) {
        for (App e : App.values()) {
            if (e.getAccronym().equals(accr))
                return e;
        }
        return App.DEFAULT;
    }
```

1.  实现主方法，并运行程序。

```java
    public static void main(String[] args) {

        // Fetching Enum with value of Enum (reverse lookup)
        App noEnum = App.get("BACHELOR OF SCIENCE");
        System.out.println(noEnum.accronym + " : " + noEnum.deptNo);
        // Fetching Enum with value of Enum (reverse lookup)

        System.out.println(App.get("BACHELOR OF SCIENCE").name());
    }
}
```

您的输出应类似于：

```java
BACHELOR OF SCIENCE : 3
BSC
```

### 第 9 课：异常处理

### 活动 36：处理数字用户输入中的错误

解决方案：

1.  右键单击**src**文件夹，然后选择**New** | **Class**。

1.  创建一个名为`Adder`的类，然后单击**OK**。

1.  导入`java.util.Scanner`包：

```java
import java.util.Scanner;
```

1.  创建一个名为`Adder`的类：

```java
import java.util.Scanner;
public class Adder {
```

1.  在`main()`方法中，使用`for`循环从用户那里读取值：

```java
   public static void main(String[] args) {
       Scanner input = new Scanner(System.in);
       int total = 0;
       for (int i = 0; i < 3; i++) {
           System.out.print("Enter a whole number: ");
```

1.  在同一个循环中，检查是否输入了有效值。如果值有效，则添加一个 try 块来计算三个数字的总和。

```java
           boolean isValid = false;
           while (!isValid) {
               if (input.hasNext()) {
                   String line = input.nextLine();
                   try {
                       int newVal = Integer.parseInt(line);
                       isValid = true;
                       total += newVal;
```

1.  catch 块应提示用户输入有效数字。

```java
} catch (NumberFormatException e) {
                       System.out.println("Please provide a valid whole number");
                   }
               }
           }
       }
```

1.  打印总和：

```java
System.out.println("Total is " + total);
   }
}
```

将结果打印到控制台。以下是一个没有错误的案例的示例输出：

```java
Enter a whole number: 10
Enter a whole number: 11
Enter a whole number: 12
Total is 33
```

以下是带有错误的运行的示例输出：

```java
Enter a whole number: 10
Enter a whole number: hello
Please provide a valid whole number
11.1
Please provide a valid whole number
11
Enter a whole number: 12
Total is 33
```

### 活动 37：在 Java 中编写自定义异常

解决方案：

1.  右键单击**src**文件夹，然后选择**New** | **Class**。

1.  输入`RollerCoasterWithAge`作为类名，然后单击**OK**。

1.  导入`java.util.Scanner`包：

```java
import java.util.Scanner;
```

1.  创建一个异常类，`TooYoungException`：

```java
class TooYoungException extends Exception {
   int age;
   String name;
   TooYoungException(int age, String name) {
       this.age = age;
       this.name = name;
   }
}
```

1.  在`main()`中，创建一个循环，读取访客的姓名：

```java
public class RollerCoasterWithAge {
   public static void main(String[] args) {
       Scanner input = new Scanner(System.in);
       while (true) {
           System.out.print("Enter name of visitor: ");
           String name = input.nextLine().trim();
           if (name.length() == 0) {
               break;
           }
```

1.  `try`块，读取访客的年龄，如果年龄低于 15 岁，则抛出`TooYoungException`，打印乘坐过山车的访客的姓名：

```java
           try {
               System.out.printf("Enter %s's age: ", name);
               int age = input.nextInt();
               input.nextLine();
               if (age < 15) {
                   throw new TooYoungException(age, name);
               }
               System.out.printf("%s is riding the roller coaster.\n", name);
```

1.  catch 块将显示 15 岁以下访客的消息：

```java
           } catch (TooYoungException e) {
               System.out.printf("%s is %d years old, which is too young to ride.\n", e.name, e.age);
           }
       }
   }
}
```

### 活动 38：在一个块中处理多个异常

解决方案：

1.  右键单击**src**文件夹，然后选择**New** | **Class**。

1.  输入`RollerCoasterWithAgeAndHeight`作为类名，然后单击**OK**。

1.  导入`java.util.Scanner`包：

```java
import java.util.Scanner;
```

1.  创建一个异常类，`TooYoungException`：

```java
class TooYoungException extends Exception {
   int age;
   String name;
   TooYoungException(int age, String name) {
       this.age = age;
       this.name = name;
   }
}
```

1.  创建一个异常类，`TooShortException`：

```java
class TooShortException extends Exception {
   int height;
   String name;
   TooShortException(int height, String name) {
       this.height = height;
       this.name = name;
   }
}
```

1.  在`main()`中，创建一个循环，读取访客的姓名：

```java
public class RollerCoasterWithAgeAndHeight {
   public static void main(String[] args) {
       Scanner input = new Scanner(System.in);
       while (true) {
           System.out.print("Enter name of visitor: ");
           String name = input.nextLine().trim();
           if (name.length() == 0) {
               break;
           }
```

1.  `try`块，读取访客的年龄，如果年龄低于 15 岁，则抛出`TooYoungException`，如果身高低于 130，则抛出`TooShortException`，并打印乘坐过山车的访客的姓名：

```java
           try {
               System.out.printf("Enter %s's age: ", name);
               int age = input.nextInt();
               input.nextLine();
               if (age < 15) {
                   throw new TooYoungException(age, name);
               }
               System.out.printf("Enter %s's height: ", name);
               int height = input.nextInt();
               input.nextLine();
               if (height < 130) {
                   throw new TooShortException(height, name);
               }
               System.out.printf("%s is riding the roller coaster.\n", name);
           } 
```

1.  catch 块将显示 15 岁以下或身高低于 130 的访客的消息：

```java
catch (TooYoungException e) {
               System.out.printf("%s is %d years old, which is too young to ride.\n", e.name, e.age);
           } catch (TooShortException e) {
               System.out.printf("%s is %d cm tall, which is too short to ride.\n", e.name, e.height);
           }
       }
   }
}
```

### 活动 39：使用多个自定义异常处理

解决方案：

1.  右键单击**src**文件夹，然后选择**New** | **Class**。

1.  输入`RollerCoasterWithAgeAndHeight`作为类名，然后单击`OK`。

1.  导入`java.util.Scanner`包：

```java
import java.util.Scanner;
```

1.  创建一个异常类，`TooYoungException`：

```java
class TooYoungException extends Exception {
   int age;
   String name;
   TooYoungException(int age, String name) {
       this.age = age;
       this.name = name;
   }
}
```

1.  创建一个异常类，`TooShortException`

```java
class TooShortException extends Exception {
   int height;
   String name;
   TooShortException(int height, String name) {
       this.height = height;
       this.name = name;
   }
}
```

1.  在`main()`中，创建一个循环，读取访客的姓名：

```java
public class Main {
   public static void main(String[] args) {
       Scanner input = new Scanner(System.in);
       while (true) {
           System.out.print("Enter name of visitor: ");
           String name = input.nextLine().trim();
           if (name.length() == 0) {
               break;
           }
```

1.  `try`块，读取访客的年龄，如果年龄低于 15 岁，则抛出`TooYoungException`，如果身高低于 130，则抛出`TooShortException`，并打印乘坐过山车的访客的姓名：

```java
           try {
               System.out.printf("Enter %s's age: ", name);
               int age = input.nextInt();
               input.nextLine();
               if (age < 15) {
                   throw new TooYoungException(age, name);
               }
               System.out.printf("Enter %s's height: ", name);
               int height = input.nextInt();
               input.nextLine();
               if (height < 130) {
                   throw new TooShortException(height, name);
               }
               System.out.printf("%s is riding the roller coaster.\n", name);
           } 
```

1.  为`TooYoungException`创建一个 catch 块：

```java
catch (TooYoungException e) {
               System.out.printf("%s is %d years old, which is too young to ride.\n", e.name, e.age);
           } 
```

1.  为`TooShortException`创建一个 catch 块：

```java
catch (TooShortException e) {
               System.out.printf("%s is %d cm tall, which is too short to ride.\n", e.name, e.height);
           } 
```

1.  创建一个最终块，打印一条消息，将访客护送离开场地：

```java
finally {
               System.out.printf("Escorting %s outside the premises.\n", name);
           }
       }
   }
}
```
