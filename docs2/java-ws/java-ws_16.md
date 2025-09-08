# 第十六章：16. 谓词和其他函数式接口

概述

本章探讨了所有有效的函数式接口的使用案例。它将首先定义这些接口是什么（从谓词接口开始），以及如何在代码中最佳地使用它们。然后，你将学习如何构建和应用谓词，研究它们的组合以及如何使用这种组合来模拟复杂行为。你将练习创建消费者接口以改变程序的状态，并最终使用函数来提取有用的结构。

# 简介

除了 Java 8 的许多其他改进（如流式 API、方法引用、可选和收集器）之外，还有接口改进，允许默认和静态方法，这些方法被称为函数式接口。这些接口只有一个抽象方法，这使得它们可以转换为 lambda 表达式。你可以在第十三章*使用 Lambda 表达式的函数式编程*中了解更多相关信息。

在 `java.util.function` 包中总共有 43 个独特的函数式接口；它们大多数是同一类接口的变体，尽管数据类型不同。在本章中，我们将向您介绍**谓词函数式接口**，以及一些其他精选的接口。

在这里，你会发现许多函数式接口以非常相似的方式操作，通常只是替换了接口可以操作的数据类型。

# 谓词接口

谓词接口是一个相当简单，但出奇地优雅和复杂的函数式接口，它允许你作为程序员，以布尔形式定义描述程序状态的函数。在 Java 中，言语谓词是一元函数，返回一个布尔值。

谓词 API 看起来是这样的：

```java
boolean test(T t);
```

然而，谓词 API 还利用了 Java 8 的新接口功能。它使用默认和静态函数来丰富 API，允许更复杂地描述程序的状态。在这里，有三个函数很重要：

```java
Predicate<T> and(Predicate<T>);
Predicate<T> or(Predicate<T>);
Predicate<T> not(Predicate<T>);
```

使用这三个函数，你可以将谓词链起来，以描述对程序状态的更复杂查询。`and` 函数将组合两个或多个谓词，确保提供的每个谓词都返回 true。

`or` 函数等同于逻辑或，允许在需要时短路谓词链。

最后，`not` 函数返回所提供的谓词的否定版本，并且它具有与在所提供的谓词上调用 `negate()` 相同的效果。

此外，还有一个 `helper` 函数用于构建一个谓词，该谓词根据对象上的 `equals` 方法检查两个对象是否相同。我们可以使用静态的 `isEqual(Object target)` 方法为两个对象构建该谓词。以下练习将作为定义谓词的示例。

## 练习 1：定义谓词

定义一个谓词相当简单。考虑构建一个家庭警报系统的后端服务器。这个系统需要能够同时轻松理解多个不同传感器的状态——例如：门是开还是关？电池是否健康？传感器是否连接？

构建这样一个系统是一个复杂任务。我们将尝试在这个练习中简化这个过程：

1.  如果 IntelliJ 已经启动，但没有打开项目，请选择 `创建新项目`。如果 IntelliJ 已经打开了项目，请从菜单中选择 `文件` -> `新建` -> `项目`。

1.  在 `新建项目` 对话框中，选择一个 Java 项目。点击 `下一步`。

1.  打勾以从模板创建项目。选择 `命令行应用程序`。点击 `下一步`。

1.  给新的项目命名为 `Chapter16`。

1.  IntelliJ 会为你提供一个默认的项目位置。如果你希望选择一个，你可以在这里输入。

1.  将包名设置为 `com.packt.java.chapter16`。

1.  点击 `完成`。

    你的项目将以标准的文件夹结构创建，并包含一个程序的入口点类。它看起来可能像这样：

    ```java
    package com.packt.java.chapter16;
    public class Main {
        public static void main(String[] args) {
        // write your code here
        }
    }
    ```

1.  将此文件重命名为 `Exercise1.java`，确保使用 `重构` | `重命名` 菜单。完成时，它应该看起来像这样：

    ```java
    package com.packt.java.chapter16;
    public class Exercise1 {
        public static void main(String[] args) {
        // write your code here
        }
    }
    ```

1.  警报系统将有三类不同的传感器——一个 `网关传感器`、一个 `运动传感器` 和一个 `火灾传感器`。它们都将具有相同的基本特性，但在某些方面可能有所不同。创建 `Base` 传感器接口，并让它有两个 getter/setter 对，第一对应该命名为 `batteryHealth`，它将返回一个介于 0 和 100 之间的整数，第二对将是一个布尔值，命名为 `triggered`：

    ```java
    package com.packt.java.chapter16;
    public interface Sensor {
        int batteryHealth();
        void batteryHealth(int health);
        boolean triggered();
        void triggered(boolean state);
    }
    ```

1.  创建 `Gateway Sensor` 类，并允许它实现 `Sensor` 接口并返回实例变量：

    ```java
    package com.packt.java.chapter16;
    public class Gateway implements Sensor {
        private int batteryHealth;
        private boolean triggered;
        @Override
        public int batteryHealth() {
            return batteryHealth;
        }
        @Override
        public void batteryHealth(int health) {
            this.batteryHealth = health;
        }
        @Override
        public boolean triggered() {
            return triggered;
        }
        @Override
        public void triggered(boolean state) {
            triggered = state;
        }
    }
    ```

1.  对 `Movement` 和 `Fire` 传感器类做同样的事情，除了 `Fire` 传感器还将有当前的 `温度`，而 `运动传感器` 将返回房间内环境光的强度：

    ```java
    package com.packt.java.chapter16;
    public class Fire implements Sensor {
        private int batteryHealth;
        private boolean triggered;
        private int temperature;
        @Override
        public int batteryHealth() {
            return batteryHealth;
        }
        @Override
        public void batteryHealth(int health) {
        }
        @Override
        public boolean triggered() {
            return triggered;
        }
        @Override
        public void triggered(boolean state) {
        }
        public int temperature() {
            return temperature;
        }
    }
    ```

    `Movement` 类的代码如下：

    ```java
    package com.packt.java.chapter16;
    public class Movement implements Sensor {
        private int batteryHealth;
        private boolean isTriggered;
        private int ambientLight;
        @Override
        public int batteryHealth() {
            return batteryHealth;
        }
        @Override
        public void batteryHealth(int health) {
        }
        @Override
        public boolean triggered() {
            return isTriggered;
        }
        @Override
        public void triggered(boolean state) {
        }
        public int ambientLight() {
            return ambientLight;
        }
    }
    ```

1.  为所有三个传感器类添加构造函数，利用 IntelliJ 的助手来完成这个任务。打开 `Fire` 类，使用 `代码` | `生成` 菜单，并选择 `构造函数`。

1.  选择所有三个变量并点击 `确定`。你的 `Fire` 类现在应该看起来像这样：

    ```java
    package com.packt.java.chapter16;
    public class Fire implements Sensor {
        private int batteryHealth;
        private boolean triggered;
        private int temperature;
        public Fire(int batteryHealth, boolean isTriggered, int temperature) {
            this.batteryHealth = batteryHealth;
            this.triggered = isTriggered;
            this.temperature = temperature;
        }
        @Override
        public int batteryHealth() {
            return batteryHealth;
        }
        @Override
        public void batteryHealth(int health) {
        }
        @Override
        public boolean triggered() {
            return triggered;
        }
        @Override
        public void triggered(boolean state) {
        }
        public int temperature() {
            return temperature;
        }
    }
    ```

1.  为 `Gateway` 和 `Movement` 传感器生成构造函数。

1.  现在，你应该在你的程序中有了三个代表传感器状态的类。

1.  现在是时候描述你的第一个谓词类了，这个谓词用来描述传感器是否触发了警报。创建一个新的类，并将其命名为 `HasAlarm`：

    ```java
    package com.packt.java.chapter16;
    public class HasAlarm {
    }
    ```

1.  实现 `Predicate` 接口，使用 `Sensor` 作为类型定义。在 `test` 函数中，返回传感器的触发状态：

    ```java
    package com.packt.java.chapter16;
    import java.util.function.Predicate;
    public class HasAlarm implements Predicate<Sensor> {
        @Override
        public boolean test(Sensor sensor) {
            return sensor.triggered();
        }
    }
    ```

1.  在你的程序入口点，即 `main` 方法中，创建一个传感器列表，并添加几个 `Gateway` 传感器到其中：

    ```java
    package com.packt.java.chapter16;
    import java.util.ArrayList;
    import java.util.List;
    public class Exercise1 {
        public static void main(String[] args) {
            List<Sensor> sensors = new ArrayList<>();
            sensors.add(new Gateway(34, false));
            sensors.add(new Gateway(14, true));
            sensors.add(new Gateway(74, false));
            sensors.add(new Gateway(8, false));
            sensors.add(new Gateway(18, false));
            sensors.add(new Gateway(9, false));
        }
    }
    ```

1.  在主方法中使用`for`循环遍历列表。在`for`循环中，添加一个使用谓词检查是否触发了警报的`if`语句：

    ```java
    package com.packt.java.chapter16;
    import java.util.ArrayList;
    import java.util.List;
    import java.util.function.Predicate;
    public class Exercise1 {
        public static void main(String[] args) {
            List<Sensor> sensors = new ArrayList<>();
            sensors.add(new Gateway(34, false));
            sensors.add(new Gateway(14, true));
            sensors.add(new Gateway(74, false));
            sensors.add(new Gateway(8, false));
            sensors.add(new Gateway(18, false));
            sensors.add(new Gateway(9, false));
            for (Sensor sensor : sensors) {
                if (new HasAlarm().test(sensor)) {
                    System.out.println("Alarm was triggered");
                }
            }
        }
    }
    ```

    注意

    你可能会问自己这有什么用。这与使用传感器的`public triggered()`函数没有区别。这也是应用谓词的一种不常见方式，但它说明了谓词的工作原理。一个更常见的方法是使用流和 lambda 表达式。

1.  现在，创建另一个谓词，并将其命名为`HasWarning`。在这个类中，我们将简单地检查电池状态是否低于`10`的阈值，在我们的例子中这表示 10%：

    ```java
    package com.packt.java.chapter16;
    import java.util.function.Predicate;
    public class HasWarning implements Predicate<Sensor> {
        public static final int BATTERY_WARNING = 10;
        @Override
        public boolean test(Sensor sensor) {
            return sensor.batteryHealth() < BATTERY_WARNING;
        }
    }
    ```

1.  使用`HasAlarm`和`HasWarning`谓词生成一个新的组合谓词。实例化`HasAlarm`谓词，并应用默认的`or()`函数来链式添加`HasWarning`谓词：

    ```java
    package com.packt.java.chapter16;
    import java.util.ArrayList;
    import java.util.List;
    import java.util.function.Predicate;
    public class Exercise1 {
        public static void main(String[] args) {
            List<Sensor> sensors = new ArrayList<>();
            sensors.add(new Gateway(34, false));
            sensors.add(new Gateway(14, true));
            sensors.add(new Gateway(74, false));
            sensors.add(new Gateway(8, false));
            sensors.add(new Gateway(18, false));
            sensors.add(new Gateway(9, false));
            Predicate<Sensor> hasAlarmOrWarning = new HasAlarm().or(new           HasWarning());
            for (Sensor sensor : sensors) {
                if (new HasAlarm().test(sensor)) {
                    System.out.println("Alarm was triggered");
                }
            }
        }
    }
    ```

1.  在`for`循环中使用新组成的谓词添加一个新的`if`语句：

```java
Exercise1.java
1  package com.packt.java.chapter16;
2  
3  import java.util.ArrayList;
4  import java.util.List;
5  import java.util.function.Predicate;
6  
7  public class Exercise1 {
8  
9      public static void main(String[] args) {
10         List<Sensor> sensors = new ArrayList<>();
11         sensors.add(new Gateway(34, false));
12         sensors.add(new Gateway(14, true));
13         sensors.add(new Gateway(74, false));
14         sensors.add(new Gateway(8, false));
15         sensors.add(new Gateway(18, false));
16         sensors.add(new Gateway(9, false));
https://packt.live/2P9njsy
```

如前所述，在像这样的循环中对对象直接应用谓词（或任何其他功能性接口）是不常见的。相反，你将主要使用 Java 流 API。

# 活动一：切换传感器状态

重新编写程序一次，向你的程序中添加一个扫描器以从命令行切换传感器状态。每个传感器应至少能够切换电池健康状态和触发状态。当传感器更新时，你应该检查系统是否有变化，并在命令行上生成适当的响应，如果已触发警告或警报。

注意

本活动的解决方案可在第 565 页找到。

## 消费者接口

在函数式编程中，我们经常被告知要避免在代码中产生副作用。然而，消费者功能性接口是这一规则的例外。它的唯一目的是根据参数的状态产生副作用。消费者有一个相当简单的 API，其核心功能称为`accept()`，不返回任何内容：

```java
void accept(T);
```

这也可以通过使用`andThen()`函数来链式使用多个消费者，该函数返回新链式连接的消费者：

```java
Consumer<T> andThen(Consumer<T>);
```

## 练习 2：产生副作用

继续上一个练习，考虑以下示例，我们将添加对系统警告和警报的反应功能。你可以使用消费者来产生副作用并将系统的当前状态存储在变量中：

1.  复制`Exercise1.java`类，并将其命名为`Exercise2`。删除整个`for`循环，但保留实例化的谓词。

1.  在`Exercise2`中创建一个新的静态布尔变量，并将其命名为`alarmServiceNotified`：

    ```java
    package com.packt.java.chapter16;
    import java.util.ArrayList;
    import java.util.List;
    import java.util.function.Predicate;
    public class Exercise2 {
        static boolean alarmServiceNotified;
        public static void main(String[] args) {
            List<Sensor> sensors = new ArrayList<>();
            sensors.add(new Gateway(34, false));
            sensors.add(new Gateway(14, true));
            sensors.add(new Gateway(74, false));
            sensors.add(new Gateway(8, false));
            sensors.add(new Gateway(18, false));
            sensors.add(new Gateway(9, false));
            Predicate<Sensor> hasAlarmOrWarning = new HasAlarm().or(new           HasWarning());
        }
    }
    ```

    注意

    当然，这并不是你通常应用静态变量的方式（如果你真的应该使用静态变量）。然而，在这个例子中，它使说明副作用变得容易得多。

1.  创建一个新的类，命名为`SendAlarm`，并允许它实现消费者接口。它看起来可能像这样：

    ```java
    package com.packt.java.chapter16;
    import java.util.function.Consumer;
    public class SendAlarm implements Consumer<Sensor> {
        @Override
        public void accept(Sensor sensor) {
        }
    }
    ```

1.  在 `accept(Sensor sensor)` 函数内部，检查传感器是否已被触发。如果已被触发，将静态变量设置为 `true`：

    ```java
    package com.packt.java.chapter16;
    import java.util.function.Consumer;
    public class SendAlarm implements Consumer<Sensor> {
        @Override
        public void accept(Sensor sensor) {
            if (sensor.triggered()) {
                Exercise2.alarmServiceNotified = true;
            }
        }
    }
    ```

1.  回到 `main` 方法中，实例化一个新的 `SendAlarm` 消费者：

    ```java
    package com.packt.java.chapter16;
    import java.util.ArrayList;
    import java.util.List;
    import java.util.function.Predicate;
    public class Exercise2 {
        static boolean alarmServiceNotified;
        public static void main(String[] args) {
            List<Sensor> sensors = new ArrayList<>();
            sensors.add(new Gateway(34, false));
            sensors.add(new Gateway(14, true));
            sensors.add(new Gateway(74, false));
            sensors.add(new Gateway(8, false));
            sensors.add(new Gateway(18, false));
            sensors.add(new Gateway(9, false));
            Predicate<Sensor> hasAlarmOrWarning = new HasAlarm().or(new       HasWarning());
            SendAlarm sendAlarm = new SendAlarm();
        }
    }
    ```

1.  使用流，首先，根据之前定义的复合谓词过滤传感器列表。然后，使用 `forEach` 将 `SendAlarm` 消费者应用于每个触发警报或警告的传感器：

    ```java
            sensors.stream().filter(hasAlarmOrWarning).forEach(sendAlarm);
    ```

1.  现在，添加一个 `if` 语句，检查是否通知了警报服务，如果是的话，打印一条消息：

    ```java
            if (alarmServiceNotified) {
                System.out.println("Alarm service notified");
            }
    ```

1.  构建另一个消费者，这次命名为 `ResetAlarm`：

    ```java
    package com.packt.java.chapter16;
    import java.util.function.Consumer;
    public class ResetAlarm implements Consumer<Sensor> {
        @Override
        public void accept(Sensor sensor) {
        }
    }
    ```

1.  在 `ResetAlarm` 的 `accept()` 函数中添加逻辑，将 `batteryHealth` 设置为 `50`，将 `Triggered` 设置为 `false`。同时，将静态通知变量设置为 `false`：

    ```java
    package com.packt.java.chapter16;
            import java.util.function.Consumer;
    public class ResetAlarm implements Consumer<Sensor> {
        @Override
        public void accept(Sensor sensor) {
            sensor.triggered(false);
            sensor.batteryHealth(50);
            Exercise2.alarmServiceNotified = false;
        }
    }
    ```

1.  实例化新的 `ResetAlarm` 消费者，然后使用 `andThen()` 函数在 `SendAlarm` 消费者之后应用它：

    ```java
    package com.packt.java.chapter16;
    import java.util.ArrayList;
    import java.util.List;
    import java.util.function.Consumer;
    import java.util.function.Predicate;
    public class Exercise2 {
        static boolean alarmServiceNotified;
        public static void main(String[] args) {
            List<Sensor> sensors = new ArrayList<>();
            sensors.add(new Gateway(34, false));
            sensors.add(new Gateway(14, true));
            sensors.add(new Gateway(74, false));
            sensors.add(new Gateway(8, false));
            sensors.add(new Gateway(18, false));
            sensors.add(new Gateway(9, false));
            Predicate<Sensor> hasAlarmOrWarning = new HasAlarm().or(new       HasWarning());
            if (sensors.stream().anyMatch(hasAlarmOrWarning)) {
                System.out.println("Alarm or warning was triggered");
            }
            SendAlarm sendAlarm = new SendAlarm();
            ResetAlarm resetAlarm = new ResetAlarm();
            sensors.stream().filter(hasAlarmOrWarning)      .forEach(sendAlarm.andThen(resetAlarm));
            if (alarmServiceNotified) {
                System.out.println("Alarm service notified");
            }
        }
    }
    ```

1.  最后，一个额外的奖励。在 *练习 2* 的最后，*产生副作用* 的 `main` 方法中，应用 `hasAlarmOrWarning` 谓词的否定版本，并打印出 `一切正常` 消息：

```java
Exercise2.java
21         Predicate<Sensor> hasAlarmOrWarning = new HasAlarm().or(new       HasWarning());
22 
23         if (sensors.stream().anyMatch(hasAlarmOrWarning)) {
24             System.out.println("Alarm or warning was triggered");
25         }
26 
27         SendAlarm sendAlarm = new SendAlarm();
28 
29         ResetAlarm resetAlarm = new ResetAlarm();
30 
31         sensors.stream().filter(hasAlarmOrWarning)      .forEach(sendAlarm.andThen(resetAlarm));
32 
33         if (alarmServiceNotified) {
34             System.out.println("Alarm service notified");
35         }
36 
37         if (sensors.stream().anyMatch(hasAlarmOrWarning.negate())) {
38             System.out.println("Nothing was triggered");
39         }
https://packt.live/2JqD7n9
```

# 函数

函数（是的，它被称为函数）主要被引入来将一个值转换成另一个值。它通常用于映射场景。它还包含默认方法，可以将多个函数组合成一个，并在函数之间进行链式调用。

接口中的主函数被称为 `apply`，它看起来像这样：

```java
R apply(T);
```

它定义了一个返回值 `R` 和函数的输入。想法是返回值和输入不必是同一类型。

组合由 `compose` 函数处理，它也返回接口的一个实例，这意味着你可以链式调用组合。顺序是从右到左；换句话说，参数函数在调用函数之前被应用：

```java
Function<V, R> compose(Function<V, T>);
```

最后，`andThen` 函数允许你将函数一个接一个地链式调用：

```java
Function<T, V> andThen(Function<R, V>);
```

在接下来的练习中，你将练习使用这些函数。

## 练习 3：提取数据

提取所有警报系统的数据作为整数——电池百分比、温度、触发状态等，具体取决于你将警报系统推进到什么程度。首先，提取电池健康数据：

1.  复制 `Exercise2` 类并命名为 `Exercise3`。

1.  删除除了传感器列表之外的所有内容。你的类应该看起来像这样：

    ```java
    package com.packt.java.chapter16;
    import java.util.ArrayList;
    import java.util.List;
    public class Exercise3 {
        public static void main(String[] args) {
            List<Sensor> sensors = new ArrayList<>();
            sensors.add(new Gateway(34, false));
            sensors.add(new Gateway(14, true));
            sensors.add(new Gateway(74, false));
            sensors.add(new Gateway(8, false));
            sensors.add(new Gateway(18, false));
            sensors.add(new Gateway(9, false));
        }
    }
    ```

1.  创建一个新的类，命名为 `ExtractBatteryHealth`，并让它实现 `Function<T, R>` 函数式接口。重写 `apply` 函数。你的类应该看起来像这样：

    ```java
    package com.packt.java.chapter16;
    import java.util.function.Function;
    public class ExtractBatteryHealth implements Function<Sensor, Integer> {
        @Override
        public Integer apply(Sensor sensor) {
            return null;
        }
    }
    ```

1.  在 `apply` 函数中，让它返回电池健康状态，如下所示：

    ```java
    package com.packt.java.chapter16;
    import java.util.function.Function;
    public class ExtractBatteryHealth implements Function<Sensor, Integer> {
        @Override
        public Integer apply(Sensor sensor) {
            return sensor.batteryHealth();
        }
    }
    ```

1.  实例化你的新 `ExtractBatteryHealth` 函数，如果你还没有这样做的话，添加一些传感器到列表中：

    ```java
    package com.packt.java.chapter16;
    import java.util.ArrayList;
    import java.util.List;
    public class Exercise3 {
        public static void main(String[] args) {
            List<Sensor> sensors = new ArrayList<>();
            sensors.add(new Gateway(34, false));
            sensors.add(new Gateway(14, true));
            sensors.add(new Fire(78, false, 21));
            sensors.add(new Gateway(74, false));
            sensors.add(new Gateway(8, false));
            sensors.add(new Movement(87, false, 45));
            sensors.add(new Gateway(18, false));
            sensors.add(new Fire(32, false, 23));
            sensors.add(new Gateway(9, false));
            sensors.add(new Movement(76, false, 41));
            ExtractBatteryHealth extractBatteryHealth = new ExtractBatteryHealth();
        }
    }
    ```

1.  最后，使用 Java 流的 `map` 操作并应用你的新 `ExtractBatteryHealth` 实例。使用 `toArray` 操作终止流。你现在应该有一个包含所有电池健康状态的数组：

    ```java
    package com.packt.java.chapter16;
    import java.util.ArrayList;
    import java.util.List;
    public class Exercise3 {
        public static void main(String[] args) {
            List<Sensor> sensors = new ArrayList<>();
            sensors.add(new Gateway(34, false));
            sensors.add(new Gateway(14, true));
            sensors.add(new Fire(78, false, 21));
            sensors.add(new Gateway(74, false));
            sensors.add(new Gateway(8, false));
            sensors.add(new Movement(87, false, 45));
            sensors.add(new Gateway(18, false));
            sensors.add(new Fire(32, false, 23));
            sensors.add(new Gateway(9, false));
            sensors.add(new Movement(76, false, 41));
            ExtractBatteryHealth extractBatteryHealth = new           ExtractBatteryHealth();
            Integer[] batteryHealths =           sensors.stream().map(extractBatteryHealth)          .toArray(Integer[]::new);
        }
    }
    ```

1.  将你的电池健康信息打印到终端：

    ```java
    package com.packt.java.chapter16;
    import java.util.ArrayList;
    import java.util.Arrays;
    import java.util.List;
    public class Exercise3 {
        public static void main(String[] args) {
            List<Sensor> sensors = new ArrayList<>();
            sensors.add(new Gateway(34, false));
            sensors.add(new Gateway(14, true));
            sensors.add(new Fire(78, false, 21));
            sensors.add(new Gateway(74, false));
            sensors.add(new Gateway(8, false));
            sensors.add(new Movement(87, false, 45));
            sensors.add(new Gateway(18, false));
            sensors.add(new Fire(32, false, 23));
            sensors.add(new Gateway(9, false));
            sensors.add(new Movement(76, false, 41));
            ExtractBatteryHealth extractBatteryHealth = new ExtractBatteryHealth();
            Integer[] batteryHealths =           sensors.stream().map(extractBatteryHealth)          .toArray(Integer[]::new);
            System.out.println(Arrays.toString(batteryHealths));
        }
    }
    ```

## 活动二：使用递归函数

计算你的警报系统中的平均电池健康——可以通过循环、流或递归函数来实现。

注意

本活动的解决方案可以在第 566 页找到。

## 活动三：使用 Lambda 函数

不要实例化`ExtractBatteryHealth`功能接口，而是使用 lambda 表达式并存储其引用。

注意

本活动的解决方案可以在第 567 页找到。

# 摘要

在本章中，你已经探索了如何使用 Java 8 提供的功能接口。你已经在循环、单个实例和流中使用了它们，所有这些都是功能接口的有效用例。然而，你很快会发现这些功能接口的实例（简称为 lamdas）通常与流一起使用。

Java 中有许多预定义的功能接口，但其中只有少数在功能上是独特的。大多数只是不同函数的原生版本，例如`IntPredicate`、`LongPredicate`、`DoublePredicate`和`Predicate`。

在下一章中，你将了解更多关于反应式流（Reactive Streams）倡议、Flow API 以及 Java 如何构建良好的基础接口以支持反应式编程的内容。
