在上面的示例中，我们在`Season`类中添加了三个属性：`feel`、`toString`和`averageTemperature`。我们还创建了一个构造函数（用于为对象状态分配初始值的特殊方法），该构造函数接受这三个属性并添加获取器和`toString()`返回值的方法。然后，在每个常量的括号中，我们设置了在创建此常量时要传递给构造函数的值。

这是我们将要使用的演示方法：

```java
void enumDemo(Season season){
  System.out.println(season + " is " + season.getFeel());
  System.out.println(season + " has average temperature around " 
                               + season.getAverageTemperature());
}
```

`enumDemo()`方法接受`enum Season`常量并构造并显示两个句子。让我们为每个季节运行上述代码，就像这样：

```java
enumDemo2(Season3.SPRING);
enumDemo2(Season3.SUMMER);
enumDemo2(Season3.AUTUMN);
enumDemo2(Season3.WINTER);

```

结果如下：

![图片](img/90d857fd-ef52-4317-97b9-d2435ab70fb9.png)

`enum`类是一种非常强大的工具，它允许我们简化代码，并使其在运行时更加受保护，因为所有可能的值都是可预测的，并且可以提前测试。例如，我们可以使用以下单元测试来测试`SPRING`常量的获取器：

```java
@DisplayName("Enum Season tests")
public class EnumSeasonTest {
  @Test
  @DisplayName("Test Spring getters")
  void multiplyByTwo(){
    assertEquals("Spring", Season.SPRING.toString());
    assertEquals("warmer than winter", Season.SPRING.getFeel());
    assertEquals(60, Season.SPRING.getAverageTemperature());
  }
}
```

当然，获取器的代码不会出现太多错误。但如果`enum`类有更复杂的方法，或者固定值列表来自于一些应用需求文档，这样的测试将确保我们已按照要求编写了代码。

在标准的 Java 库中，有几个`enum`类。以下是这些类中常量的几个例子，可以让你了解其中的内容：

```java
Month.FEBRUARY;
TimeUnit.DAYS;
TimeUnit.MINUTES;
DayOfWeek.FRIDAY;
Color.GREEN;
Color.green;

```

所以，在创建自己的`enum`之前，尝试检查并查看标准库是否已提供具有所需值的类。

# 将引用类型值作为方法参数传递

一种需要特别讨论的引用类型和原始类型之间的重要区别是它们的值在方法中的使用方式。让我们通过示例来看看区别。首先，我们创建`SomeClass`类：

```java
class SomeClass{
  private int count;
  public int getCount() {
    return count;
  }
  public void setCount(int count) {
      this.count = count;
    }
}
```

然后我们创建一个使用它的类：
