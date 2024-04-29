# 第十六章：使用 JavaFX 进行 GUI 编程

在本章中，我们将涵盖以下配方：

+   使用 JavaFX 控件创建 GUI

+   使用 FXML 标记创建 GUI

+   使用 CSS 为 JavaFX 中的元素设置样式

+   创建柱状图

+   创建饼图

+   在应用程序中嵌入 HTML

+   在应用程序中嵌入媒体

+   向控件添加效果

+   使用机器人 API

# 介绍

自 JDK 1.0 以来，Java 一直有 GUI 编程，通过名为**抽象窗口工具包**（**AWT**）的 API。在那个时代，这是一件了不起的事情，但它也有自己的局限性，其中一些如下：

+   它有一组有限的组件。

+   由于 AWT 使用本机组件，因此无法创建自定义可重用组件。

+   组件的外观和感觉无法控制，它们采用了主机操作系统的外观和感觉。

然后，在 Java 1.2 中，引入了一种名为**Swing**的新 GUI 开发 API，它通过提供以下功能来解决 AWT 的不足：

+   更丰富的组件库。

+   支持创建自定义组件。

+   本地外观和感觉，以及支持插入不同的外观和感觉。一些著名的 Java 外观和感觉主题包括 Nimbus、Metal、Motif 和系统默认。

已经构建了许多使用 Swing 的桌面应用程序，其中许多仍在使用。然而，随着时间的推移，技术必须不断发展；否则，它最终将过时并且很少被使用。2008 年，Adobe 的**Flex**开始引起关注。这是一个用于构建**富互联网应用程序**（**RIA**）的框架。桌面应用程序一直是基于丰富组件的 UI，但是 Web 应用程序并不那么令人惊叹。Adobe 推出了一个名为 Flex 的框架，它使 Web 开发人员能够在 Web 上创建丰富、沉浸式的 UI。因此，Web 应用程序不再无聊。

Adobe 还为桌面引入了一个富互联网应用程序运行时环境，称为**Adobe AIR**，它允许在桌面上运行 Flex 应用程序。这对古老的 Swing API 是一个重大打击。但让我们回到市场：2009 年，Sun Microsystems 推出了一个名为**JavaFX**的东西。这个框架受 Flex 的启发（使用 XML 定义 UI），并引入了自己的脚本语言称为**JavaFX 脚本**，它与 JSON 和 JavaScript 有些相似。您可以从 JavaFX 脚本中调用 Java API。引入了一个新的架构，其中有一个新的窗口工具包和一个新的图形引擎。这是一个比 Swing 更好的选择，但它有一个缺点——开发人员必须学习 JavaFX 脚本来开发基于 JavaFX 的应用程序。除了 Sun Microsystems 无法在 JavaFX 和 Java 平台上投入更多投资之外，JavaFX 从未像预期的那样起飞。

Oracle（收购 Sun Microsystems 后）宣布推出了一个新的 JavaFX 版本 2.0，这是对 JavaFX 的整体重写，从而消除了脚本语言，并使 JavaFX 成为 Java 平台内的 API。这使得使用 JavaFX API 类似于使用 Swing API。此外，您可以在 Swing 中嵌入 JavaFX 组件，从而使基于 Swing 的应用程序更加功能强大。从那时起，JavaFX 就再也没有回头看了。

从 JDK 11 开始（无论是 Oracle JDK 还是 OpenJDK 构建），JavaFX 都不再捆绑在一起。OpenJDK 10 构建也不再捆绑 JavaFX。它们必须从 OpenJFX 项目页面（[`wiki.openjdk.java.net/display/OpenJFX/Main`](https://wiki.openjdk.java.net/display/OpenJFX/Main)）单独下载。OpenJFX 还推出了一个新的社区网站（[`openjfx.io/`](https://openjfx.io/)）。

在这一章中，我们将完全专注于围绕 JavaFX 的配方。我们将尽量涵盖尽可能多的配方，以便让您充分体验使用 JavaFX。

# 使用 JavaFX 控件创建 GUI

在这个示例中，我们将看到如何使用 JavaFX 控件创建一个简单的 GUI 应用程序。我们将构建一个应用程序，可以在您提供出生日期后帮助您计算您的年龄。您还可以输入您的姓名，应用程序将向您问候并显示您的年龄。这是一个相当简单的示例，试图展示如何通过使用布局、组件和事件处理来创建 GUI。

# 准备工作

以下是 JavaFX 的模块的一部分：

+   `javafx.base`

+   `javafx.controls`

+   `javafx.fxml`

+   `javafx.graphics`

+   `javafx.media`

+   `javafx.swing`

+   `javafx.web`

如果您使用的是 Oracle JDK 10 和 9，它会随着之前提到的 JavaFX 模块一起安装；也就是说，您可以在`JAVA_HOME/jmods`目录中找到它们。如果您使用的是 OpenJDK 10 及更高版本和 JDK 11 及更高版本，您需要从[`gluonhq.com/products/javafx/`](https://gluonhq.com/products/javafx/)下载 JavaFX SDK，并将`JAVAFX_SDK_PATH/libs`位置的 JAR 文件可用于您的`modulepath`。

```java
javac -p "PATH_TO_JAVAFX_SDK_LIB" <other parts of the command line> 

#Windows
java -p "PATH_TO_JAVAFX_SDK_LIB;COMPILED_CODE" <other parts of the command line>
#Linux 
java -p "PATH_TO_JAVAFX_SDK_LIB:COMPILED_CODE" <other parts of the command line>
```

在我们的示例中，我们将根据需要从上面的列表中使用一些模块。

# 如何做...

1.  创建一个扩展`javafx.application.Application`的类。`Application`类管理 JavaFX 应用程序的生命周期。`Application`类有一个抽象方法`start(Stage stage)`，您必须实现这个方法。这将是 JavaFX UI 的起始点：

```java
        public class CreateGuiDemo extends Application{
          public void start(Stage stage){
            //to implement in new steps
          }
        }
```

该类还可以通过提供一个`public static void main(String [] args) {}`方法成为应用程序的起始点：

```java
        public class CreateGuiDemo extends Application{
          public void start(Stage stage){
            //to implement in new steps
          }
          public static void main(String[] args){
            //launch the JavaFX application
          }
        }
```

后续步骤的代码必须在`start(Stage stage)`方法中编写。

1.  让我们创建一个容器布局，以正确对齐我们将要添加的组件。在这种情况下，我们将使用`javafx.scene.layout.GridPane`来以行和列的形式布置组件：

```java
        GridPane gridPane = new GridPane();
        gridPane.setAlignment(Pos.CENTER);
        gridPane.setHgap(10);
        gridPane.setVgap(10);
        gridPane.setPadding(new Insets(25, 25, 25, 25));
```

除了创建`GridPane`实例之外，我们还设置了它的布局属性，比如`GridPane`的对齐方式，行和列之间的水平和垂直间距，以及网格中每个单元格的填充。

1.  创建一个新的标签，它将显示我们应用程序的名称，具体来说是`年龄计算器`，并将其添加到我们在前一步中创建的`gridPane`中：

```java
        Text appTitle = new Text("Age calculator");
        appTitle.setFont(Font.font("Arial", FontWeight.NORMAL, 15));
        gridPane.add(appTitle, 0, 0, 2, 1);
```

1.  创建一个标签和文本输入组合，用于接受用户的姓名。然后将这两个组件添加到`gridPane`中：

```java
        Label nameLbl = new Label("Name");
        TextField nameField = new TextField();
        gridPane.add(nameLbl, 0, 1);
        gridPane.add(nameField, 1, 1);
```

1.  创建一个标签和日期选择器组合，用于接受用户的出生日期：

```java
        Label dobLbl = new Label("Date of birth");
        gridPane.add(dobLbl, 0, 2);
        DatePicker dateOfBirthPicker = new DatePicker();
        gridPane.add(dateOfBirthPicker, 1, 2);
```

1.  创建一个按钮，用户将用它来触发年龄计算，并将其添加到`gridPane`中：

```java
        Button ageCalculator = new Button("Calculate");
        gridPane.add(ageCalculator, 1, 3);
```

1.  创建一个组件来保存计算出的年龄的结果：

```java
        Text resultTxt = new Text();
        resultTxt.setFont(Font.font("Arial", FontWeight.NORMAL, 15));
        gridPane.add(resultTxt, 0, 5, 2, 1);
```

1.  现在我们需要为第 6 步中创建的按钮绑定一个动作。动作将是获取在名称字段中输入的名称和在日期选择器字段中输入的出生日期。如果提供了出生日期，则使用 Java 时间 API 来计算现在和出生日期之间的时间段。如果提供了名称，则在结果前加上一个问候语，`你好，<name>`：

```java
        ageCalculator.setOnAction((event) -> {
          String name = nameField.getText();
          LocalDate dob = dateOfBirthPicker.getValue();
          if ( dob != null ){
            LocalDate now = LocalDate.now();
            Period period = Period.between(dob, now);
            StringBuilder resultBuilder = new StringBuilder();
            if ( name != null && name.length() > 0 ){
              resultBuilder.append("Hello, ")
                         .append(name)
                         .append("n");
            }
            resultBuilder.append(String.format(
              "Your age is %d years %d months %d days",
              period.getYears(), 
              period.getMonths(), 
              period.getDays())
            );
            resultTxt.setText(resultBuilder.toString());
          }
        });
```

1.  通过提供我们在第 2 步中创建的`gridPane`对象和场景的宽度和高度，创建`Scene`类的一个实例：

```java
        Scene scene = new Scene(gridPane, 300, 250);
```

`Scene`的一个实例保存了 UI 组件的图形，这被称为**场景图**。

1.  我们已经看到`start()`方法为我们提供了一个`Stage`对象的引用。`Stage`对象是 JavaFX 中的顶级容器，类似于 JFrame。我们将`Scene`对象设置为`Stage`对象，并使用它的`show()`方法来渲染 UI：

```java
        stage.setTitle("Age calculator");
        stage.setScene(scene);
        stage.show();
```

1.  现在我们需要从主方法启动这个 JavaFX UI。我们使用`Application`类的`launch(String[] args)`方法来启动 JavaFX UI：

```java
        public static void main(String[] args) {
          Application.launch(args);
        }
```

完整的代码可以在`Chapter16/1_create_javafx_gui`中找到。

我们在`Chapter16/1_create_javafx_gui`中提供了两个脚本，`run.bat`和`run.sh`。`run.bat`脚本用于在 Windows 上运行应用程序，`run.sh`用于在 Linux 上运行应用程序。

使用`run.bat`或`run.sh`运行应用程序，您将看到 GUI，如下面的屏幕截图所示：

![](img/f78eab15-9491-41ba-b503-27def3e0c8e9.png)

输入姓名和出生日期，然后单击`Calculate`查看年龄：

![](img/f52396d1-5c74-462e-b2b4-4e265c7d99b6.png)

# 工作原理...

在进入其他细节之前，让我们简要概述一下 JavaFX 架构。我们从 JavaFX 文档中获取了以下描述架构堆栈的图表（[`docs.oracle.com/javase/8/javafx/get-started-tutorial/jfx-architecture.htm#JFXST788`](http://docs.oracle.com/javase/8/javafx/get-started-tutorial/jfx-architecture.htm#JFXST788)）：

![](img/fe62de47-1d02-4f31-afd7-e09bea089b5b.png)

让我们从堆栈的顶部开始：

+   **JavaFX API 和场景图**: 这是应用程序的起点，我们大部分关注点将围绕这部分。它提供了不同组件、布局和其他实用程序的 API，以便开发基于 JavaFX 的 UI。场景图保存了应用程序的可视元素。

+   **Prism、Quantum Toolkit 和蓝色的其他内容**: 这些组件管理 UI 的渲染，并在底层操作系统和 JavaFX 之间提供桥梁。在图形硬件无法提供丰富 UI 和 3D 元素的硬件加速渲染的情况下，此层提供软件渲染。

+   **玻璃窗口工具包**: 这是窗口工具包，就像 Swing 使用的 AWT 一样。

+   **媒体引擎**: 这支持 JavaFX 中的媒体。

+   **Web 引擎**: 这支持 Web 组件，允许完整的 HTML 渲染。

+   **JDK API 和 JVM**: 这些与 Java API 集成，并将代码编译为字节码以在 JVM 上运行。

让我们回到解释这个示例。`javafx.application.Application`类是启动 JavaFX 应用程序的入口点。它具有以下方法，这些方法映射到应用程序的生命周期（按其调用顺序）：

+   **`init()`**: 此方法在`javafx.application.Application`实例化后立即调用。您可以重写此方法，在应用程序启动之前进行一些初始化。默认情况下，此方法不执行任何操作。

+   `start(javafx.stage.Stage)`: 此方法在`init()`之后立即调用，并在系统完成运行应用程序所需的初始化后调用。此方法传递了一个`javafx.stage.Stage`实例，这是组件呈现的主要 stage。您可以创建其他`javafx.stage.Stage`对象，但应用程序提供的是主要 stage。

+   `stop()`: 当应用程序应该停止时调用此方法。您可以执行必要的退出相关操作。

*stage*是一个顶级的 JavaFX 容器。作为`start()`方法的参数传递的主要 stage 是由平台创建的，应用程序可以根据需要创建其他`Stage`容器。

与`javafx.application.Application`相关的另一个重要方法是`launch()`方法。有两种变体：

+   `launch(Class<? extends Application> appClass, String... args)`

+   `launch(String... args)`

此方法从主方法中调用，应该只调用一次。第一个变体带有扩展`javafx.application.Application`类的类名以及传递给主方法的参数，第二个变体不带类名，而是应该从扩展`javafx.application.Application`类的类内部调用。在我们的示例中，我们使用了第二个变体。

我们创建了一个类`CreateGuiDemo`，扩展了`javafx.application.Application`。这将是 JavaFX UI 的入口点，我们还向类中添加了一个 main 方法，使其成为我们应用程序的入口点。

布局构造确定了组件的布局方式。JavaFX 支持多种布局，如下所示：

+   `javafx.scene.layout.HBox`和`javafx.scene.layout.VBox`：这些用于水平和垂直对齐组件。

+   `javafx.scene.layout.BorderPane`：这允许在顶部、右侧、底部、左侧和中心位置放置组件。

+   `javafx.scene.layout.FlowPane`：此布局允许在流中放置组件，即相邻放置，并在流面板的边界处换行。

+   `javafx.scene.layout.GridPane`：此布局允许在行和列的网格中放置组件。

+   `javafx.scene.layout.StackPane`：此布局将组件放置在一个从后到前的堆栈中。

+   `javafx.scene.layout.TilePane`：此布局将组件放置在统一大小的网格中。

在我们的示例中，我们使用了`GridPane`并配置了布局，以便实现以下目标：

+   在中心放置网格（`gridPane.setAlignment(Pos.CENTER);）

+   将列之间的间隙设置为 10（`gridPane.setHgap(10);）

+   将行之间的间隙设置为 10（`gridPane.setVgap(10);`)

+   在网格的单元格内设置填充（`gridPane.setPadding(new Insets(25, 25, 25, 25));）

`javafx.scene.text.Text`组件的字体可以使用`javafx.scene.text.Font`对象来设置，如下所示：`appTitle.setFont(Font.font("Arial", FontWeight.NORMAL, 15));`

在将组件添加到`javafx.scene.layout.GridPane`时，我们必须提到列号、行号和列跨度，即组件占据多少列，以及行跨度，即组件在该顺序中占据多少行。列跨度和行跨度是可选的。在我们的示例中，我们将`appTitle`放在第一行和列中，并占据两列空间和一行空间，如下所示：`appTitle.setFont(Font.font("Arial", FontWeight.NORMAL, 15));`

在这个示例中的另一个重要部分是为`ageCalculator`按钮设置事件。我们使用`javafx.scene.control.Button`类的`setOnAction()`方法来设置按钮点击时执行的操作。这接受`javafx.event.EventHandler<ActionEvent>`接口的实现。由于`javafx.event.EventHandler`是一个功能接口，因此可以以 lambda 表达式的形式编写其实现，如下所示：

```java
ageCalculator.setOnAction((event) -> {
  //event handling code here
});
```

上述语法看起来类似于 Swing 时期广泛使用的匿名内部类。您可以在第四章“进入函数”中的示例中了解有关功能接口和 lambda 表达式的更多信息。

在我们的事件处理代码中，我们通过使用`getText()`和`getValue()`方法从`nameField`和`dateOfBirthPicker`中获取值。`DatePicker`将所选日期作为`java.time.LocalDate`的实例返回。这是 Java 8 中添加的新日期时间 API 之一。它表示一个日期，即年、月和日，没有任何与时区相关的信息。然后我们使用`java.time.Period`类来找到当前日期和所选日期之间的持续时间，如下所示：

```java
LocalDate now = LocalDate.now();
Period period = Period.between(dob, now);
```

`Period`表示基于日期的持续时间，例如三年、两个月和三天。这正是我们试图用这行代码提取的内容：`String.format("你的年龄是%d 年%d 月%d 天", period.getYears(), period.getMonths(), period.getDays())`。

我们已经提到，JavaFX 中的 UI 组件以场景图的形式表示，然后将此场景图呈现到一个称为`Stage`的容器上。创建场景图的方法是使用`javafx.scene.Scene`类。我们通过传递场景图的根以及提供场景图将呈现在其中的容器的尺寸来创建`javafx.scene.Scene`实例。

我们利用提供给`start()`方法的容器，这只是`javafx.stage.Stage`的一个实例。为`Stage`对象设置场景，然后调用其`show()`方法，使完整的场景图在显示器上呈现：

```java
stage.setScene(scene);
stage.show();
```

# 使用 FXML 标记创建 GUI

在我们的第一个示例中，我们使用 Java API 构建了一个 UI。经常发生的情况是，精通 Java 的人可能不是一个好的 UI 设计师；也就是说，他们可能不擅长为他们的应用程序确定最佳的用户体验。在 Web 开发领域，我们有开发人员根据 UX 设计师提供的设计进行前端开发，另一组开发人员则负责后端开发，构建由前端使用的服务。

两个开发人员团队同意一组 API 和一个共同的数据交换模型。前端开发人员使用基于数据交换模型的一些模拟数据，并将 UI 与所需的 API 集成。另一方面，后端开发人员致力于实现 API，以便它们返回协商的数据交换模型中的数据。因此，双方同时使用各自领域的专业知识进行工作。

如果桌面应用程序能够以某种程度上（至少在某种程度上）复制相同的功能将是很棒的。这方面的一大进展是引入了一种基于 XML 的语言，称为**FXML**。这使得 UI 开发具有声明性方法，开发人员可以独立使用相同的 JavaFX 组件开发 UI，但可用作 XML 标记。JavaFX 组件的不同属性可用作 XML 标记的属性。事件处理程序可以在 Java 代码中声明和定义，然后从 FXML 中引用。

在本示例中，我们将指导您通过使用 FXML 构建 UI，然后将 FXML 与 Java 代码集成，以绑定操作并启动在 FXML 中定义的 UI。

# 准备工作

由于我们知道自从 Oracle JDK 11 开始和 Open JDK 10 开始，JavaFX 库不再随 JDK 安装一起提供，我们将不得不从[`gluonhq.com/products/javafx/`](https://gluonhq.com/products/javafx/)下载 JavaFX SDK，并使用`-p`选项将 SDK 的`lib`文件夹中的 JAR 文件添加到模块路径中，如下所示：

```java
javac -p "PATH_TO_JAVAFX_SDK_LIB" <other parts of the command line> 

#Windows
java -p "PATH_TO_JAVAFX_SDK_LIB;COMPILED_CODE" <other parts of the command line> 
#Linux 
java -p "PATH_TO_JAVAFX_SDK_LIB:COMPILED_CODE" <other parts of the command line> 
```

我们将开发一个简单的年龄计算器应用程序。该应用程序将要求用户输入姓名（可选）和出生日期，并根据给定的出生日期计算年龄并显示给用户。

# 如何做...

1.  所有 FXML 文件应以`.fxml`扩展名结尾。让我们在`src/gui/com/packt`位置创建一个空的`fxml_age_calc_gui.xml`文件。在随后的步骤中，我们将使用 JavaFX 组件的 XML 标签更新此文件。

1.  创建一个`GridPane`布局，它将在行和列的网格中容纳所有组件。我们还将使用`vgap`和`hgap`属性为行和列之间提供所需的间距。此外，我们将为`GridPane`（我们的根组件）提供对 Java 类的引用，我们将在其中添加所需的事件处理。这个 Java 类将类似于 UI 的控制器：

```java
        <GridPane alignment="CENTER" hgap="10.0" vgap="10.0"

          fx:controller="com.packt.FxmlController">
        </GridPane>
```

1.  我们将通过在`GridPane`中定义`Insets`标签来为网格中的每个单元格提供填充：

```java
        <padding>
          <Insets bottom="25.0" left="25.0" right="25.0" top="25.0" />
        </padding>
```

1.  接下来是添加一个`Text`标签，用于显示应用程序的标题——`年龄计算器`。我们在`style`属性中提供所需的样式信息，并使用`GridPane.columnIndex`和`GridPane.rowIndex`属性将`Text`组件放置在`GridPane`中。可以使用`GridPane.columnSpan`和`GridPane.rowSpan`属性提供单元格占用信息：

```java
        <Text style="-fx-font: NORMAL 15 Arial;" text="Age calculator"
          GridPane.columnIndex="0" GridPane.rowIndex="0" 
          GridPane.columnSpan="2" GridPane.rowSpan="1">
        </Text>
```

1.  然后我们添加`Label`和`TextField`组件来接受名称。注意在`TextField`中使用了`fx:id`属性。这有助于通过创建与`fx:id`值相同的字段来绑定 Java 控制器中的这个组件：

```java
        <Label text="Name" GridPane.columnIndex="0" 
          GridPane.rowIndex="1">
        </Label>
        <TextField fx:id="nameField" GridPane.columnIndex="1" 
          GridPane.rowIndex="1">
        </TextField>
```

1.  我们添加`Label`和`DatePicker`组件来接受出生日期：

```java
        <Label text="Date of Birth" GridPane.columnIndex="0" 
          GridPane.rowIndex="2">
        </Label>
        <DatePicker fx:id="dateOfBirthPicker" GridPane.columnIndex="1" 
          GridPane.rowIndex="2">
        </DatePicker>
```

1.  然后，我们添加一个`Button`对象，并将其`onAction`属性设置为 Java 控制器中处理此按钮点击事件的方法的名称：

```java
        <Button onAction="#calculateAge" text="Calculate"
          GridPane.columnIndex="1" GridPane.rowIndex="3">
        </Button>
```

1.  最后，我们添加一个`Text`组件来显示计算出的年龄：

```java
        <Text fx:id="resultTxt" style="-fx-font: NORMAL 15 Arial;"
          GridPane.columnIndex="0" GridPane.rowIndex="5"
          GridPane.columnSpan="2" GridPane.rowSpan="1"
        </Text>
```

1.  下一步是实现与前面步骤中创建的基于 XML 的 UI 组件相关的 Java 类。创建一个名为`FxmlController`的类。这将包含与 FXML UI 相关的代码；也就是说，它将包含对在 FXML 中创建的组件的引用，以及对在 FXML 中创建的组件的动作处理程序：

```java
        public class FxmlController {
          //to implement in next few steps
        }
```

1.  我们需要引用`nameField`，`dateOfBirthPicker`和`resultText`组件。我们使用前两个分别获取输入的名称和出生日期，第三个用于显示年龄计算的结果：

```java
        @FXML private Text resultTxt;
        @FXML private DatePicker dateOfBirthPicker;
        @FXML private TextField nameField;
```

1.  下一步是实现`calculateAge`方法，该方法注册为`Calculate`按钮的动作事件处理程序。实现与上一个示例中类似。唯一的区别是它是一个方法，而不是上一个示例中的 lambda 表达式：

```java
        @FXML
        public void calculateAge(ActionEvent event){
          String name = nameField.getText();
          LocalDate dob = dateOfBirthPicker.getValue();
          if ( dob != null ){
            LocalDate now = LocalDate.now();
            Period period = Period.between(dob, now);
            StringBuilder resultBuilder = new StringBuilder();
            if ( name != null && name.length() > 0 ){
              resultBuilder.append("Hello, ")
                           .append(name)
                           .append("n");
            }
            resultBuilder.append(String.format(
              "Your age is %d years %d months %d days", 
              period.getYears(), 
              period.getMonths(), 
              period.getDays())
            );
            resultTxt.setText(resultBuilder.toString());
          }
        }
```

1.  在步骤 10 和 11 中，我们使用了一个注解`@FXML`。这个注解表示该类或成员对基于 FXML 的 UI 是可访问的。

1.  接下来，我们将创建另一个 Java 类`FxmlGuiDemo`，负责渲染基于 FXML 的 UI，也将是启动应用程序的入口点：

```java
        public class FxmlGuiDemo extends Application{ 
          //code to launch the UI + provide main() method
        }
```

1.  现在我们需要通过覆盖`javafx.application.Application`类的`start(Stage stage)`方法从 FXML UI 定义创建一个场景图，并在传递的`javafx.stage.Stage`对象中渲染场景图：

```java
        @Override
        public void start(Stage stage) throws IOException{
          FXMLLoader loader = new FXMLLoader();
          Pane pane = (Pane)loader.load(getClass()
              .getModule()
              .getResourceAsStream("com/packt/fxml_age_calc_gui.fxml")
          );

          Scene scene = new Scene(pane,300, 250);
          stage.setTitle("Age calculator");
          stage.setScene(scene);
          stage.show();
        }
```

1.  最后，我们提供`main()`方法的实现：

```java
        public static void main(String[] args) {
          Application.launch(args);
        }
```

完整的代码可以在`Chapter16/2_fxml_gui`位置找到。

我们在`Chapter16/2_fxml_gui`中提供了两个运行脚本，`run.bat`和`run.sh`。`run.bat`脚本用于在 Windows 上运行应用程序，`run.sh`用于在 Linux 上运行应用程序。

使用`run.bat`或`run.sh`运行应用程序，您将看到如下屏幕截图所示的 GUI：

![](img/44729bd6-e935-48d0-bccc-d26e225c8469.png)

输入名称和出生日期，然后点击`Calculate`来查看年龄：

![](img/d7ef594d-c776-4d68-993e-5e94e3c657c2.png)

# 工作原理...

没有定义 FXML 文档模式的 XSD。因此，要知道要使用的标签，它们遵循一个简单的命名约定。组件的 Java 类名也是 XML 标签的名称。例如，`javafx.scene.layout.GridPane`布局的 XML 标签是`<GridPane>`，`javafx.scene.control.TextField`的 XML 标签是`<TextField>`，`javafx.scene.control.DatePicke`的 XML 标签是`<DatePicker>`：

```java
Pane pane = (Pane)loader.load(getClass()
    .getModule()
    .getResourceAsStream("com/packt/fxml_age_calc_gui.fxml")
 );
```

上面的代码行利用`javafx.fxml.FXMLLoader`的一个实例来读取 FXML 文件并获取 UI 组件的 Java 表示。`FXMLLoader`使用基于事件的 SAX 解析器来解析 FXML 文件。通过反射创建 XML 标签的相应 Java 类的实例，并将 XML 标签的属性值填充到 Java 类的相应属性中。

由于我们的 FXML 的根是`javafx.scene.layout.GridPane`，它扩展了`javafx.scene.layout.Pane`，我们可以将`FXMLoader.load()`的返回值转换为`javafx.scene.layout.Pane`。

这个示例中的另一个有趣的地方是`FxmlController`类。这个类充当了与 FXML 的接口。我们在 FXML 中使用`fx:controller`属性指示相同的内容到`<GridPane>`标签。我们可以通过在`FxmlController`类的成员字段上使用`@FXML`注解来获取在 FXML 中定义的 UI 组件，就像我们在这个示例中所做的那样：

```java
@FXML private Text resultTxt;
@FXML private DatePicker dateOfBirthPicker;
@FXML private TextField nameField;
```

成员的名称与 FXML 中的`fx:id`属性值相同，成员的类型与 FXML 中的标签类型相同。例如，第一个成员绑定到以下内容：

```java
<Text fx:id="resultTxt" style="-fx-font: NORMAL 15 Arial;"
  GridPane.columnIndex="0" GridPane.rowIndex="5" 
  GridPane.columnSpan="2" GridPane.rowSpan="1">
</Text>
```

在类似的情况下，我们在`FxmlController`中创建了一个事件处理程序，并用`@FXML`对其进行了注释，在 FXML 中使用了`<Button>`的`onAction`属性引用了它。请注意，我们在`onAction`属性值的方法名称前面添加了`#`。

# 使用 CSS 来为 JavaFX 中的元素设置样式

来自 Web 开发背景的人将能够欣赏到**层叠样式表**（**CSS**）的实用性，而对于那些不了解的人，我们将在深入介绍 JavaFX 中的 CSS 应用之前，提供它们的概述和用途。

在网页上看到的元素或组件通常根据网站的主题进行样式设置。这种样式是通过使用一种叫做**CSS**的语言实现的。CSS 由一组以分号分隔的`name:value`对组成。当这些`name:value`对与 HTML 元素关联时，比如`<button>`，它就会获得所需的样式。

有多种方法将这些`name:value`对与元素关联起来，最简单的方法是将这些`name:value`对放在 HTML 元素的 style 属性中。例如，要给按钮设置蓝色背景，我们可以这样做：

```java
<button style="background-color: blue;"></button>
```

不同样式属性有预定义的名称，并且它们接受特定的一组值；也就是说，属性`background-color`只能接受有效的颜色值。

另一种方法是在一个扩展名为`.css`的不同文件中定义这些`name:value`对的组。让我们称这组`name:value`对为**CSS 属性**。我们可以将这些 CSS 属性与不同的选择器关联起来，即选择器用于选择要应用 CSS 属性的 HTML 页面上的元素。有三种不同的提供选择器的方法：

1.  直接给出 HTML 元素的名称，即是锚标签（`<a>`）、按钮或输入。在这种情况下，CSS 属性将应用于页面中所有类型的 HTML 元素。

1.  通过使用 HTML 元素的`id`属性。假设我们有一个`id="btn1"`的按钮，那么我们可以定义一个选择器`#btn1`，针对它提供 CSS 属性。看下面的例子：

```java
        #btn1 { background-color: blue; }
```

1.  通过使用 HTML 元素的 class 属性。假设我们有一个`class="blue-btn"`的按钮，那么我们可以定义一个选择器`.blue-btn`，针对它提供 CSS 属性。看下面的例子：

```java
        .blue-btn { background-color: blue; }
```

使用不同的 CSS 文件的优势在于，我们可以独立地改变网页的外观，而不会与元素的位置紧密耦合。此外，这鼓励在不同页面之间重用 CSS 属性，从而使它们在所有页面上具有统一的外观。

当我们将类似的方法应用于 JavaFX 时，我们可以利用我们的 Web 设计人员已经掌握的 CSS 知识来为 JavaFX 组件构建 CSS，这有助于比使用 Java API 更轻松地为组件设置样式。当这种 CSS 与 FXML 混合在一起时，它就成为了 Web 开发人员熟悉的领域。

在这个示例中，我们将使用外部 CSS 文件来为一些 JavaFX 组件设置样式。

# 准备就绪

由于我们知道从 Oracle JDK 11 开始和 Open JDK 10 开始，JavaFX 库不再随 JDK 安装一起提供，我们需要从[`gluonhq.com/products/javafx/`](https://gluonhq.com/products/javafx/)下载 JavaFX SDK，并使用`-p`选项将 SDK 的`lib`文件夹中的 JAR 包包含在模块路径上，如下所示：

```java
javac -p "PATH_TO_JAVAFX_SDK_LIB" <other parts of the command line> 

#Windows
java -p "PATH_TO_JAVAFX_SDK_LIB;COMPILED_CODE" <other parts of the command line> 
#Linux 
java -p "PATH_TO_JAVAFX_SDK_LIB:COMPILED_CODE" <other parts of the command line> 
```

在定义 JavaFX 组件的 CSS 属性时有一个小区别。所有的属性都必须以`-fx-`为前缀，也就是说，`background-color`变成了`-fx-background-color`。选择器，即`#id`和`.class-name`在 JavaFX 世界中仍然保持不变。我们甚至可以为 JavaFX 组件提供多个类，从而将所有这些 CSS 属性应用到组件上。

我在这个示例中使用的 CSS 基于一个名为**Bootstrap**的流行 CSS 框架（[`getbootstrap.com/css/`](http://getbootstrap.com/css/)）。

# 如何做...

1.  让我们创建一个`GridPane`，它将以行和列的网格形式容纳组件：

```java
        GridPane gridPane = new GridPane();
        gridPane.setAlignment(Pos.CENTER);
        gridPane.setHgap(10);
        gridPane.setVgap(10);
        gridPane.setPadding(new Insets(25, 25, 25, 25));
```

1.  首先，我们将创建一个按钮，并为其添加两个类，`btn`和`btn-primary`。在下一步中，我们将为这些选择器定义所需的 CSS 属性：

```java
        Button primaryBtn = new Button("Primary");
        primaryBtn.getStyleClass().add("btn");
        primaryBtn.getStyleClass().add("btn-primary");
        gridPane.add(primaryBtn, 0, 1);
```

1.  现在让我们为类`btn`和`btn-primary`提供所需的 CSS 属性。这些类的选择器形式为`.<class-name>`：

```java
        .btn{
          -fx-border-radius: 4px;
          -fx-border: 2px;
          -fx-font-size: 18px;
          -fx-font-weight: normal;
          -fx-text-align: center;
        }
        .btn-primary {
          -fx-text-fill: #fff;
          -fx-background-color: #337ab7;
          -fx-border-color: #2e6da4;
        }
```

1.  让我们创建另一个具有不同 CSS 类的按钮：

```java
        Button successBtn = new Button("Sucess");
        successBtn.getStyleClass().add("btn");
        successBtn.getStyleClass().add("btn-success");
        gridPane.add(successBtn, 1, 1);
```

1.  现在我们为`.btn-success`选择器定义 CSS 属性如下：

```java
        .btn-success {
          -fx-text-fill: #fff;
          -fx-background-color: #5cb85c;
          -fx-border-color: #4cae4c;
        }
```

1.  让我们创建另一个具有不同 CSS 类的按钮：

```java
        Button dangerBtn = new Button("Danger");
        dangerBtn.getStyleClass().add("btn");
        dangerBtn.getStyleClass().add("btn-danger");
        gridPane.add(dangerBtn, 2, 1);
```

1.  我们将为选择器`.btn-danger`定义 CSS 属性。

```java
        .btn-danger {
          -fx-text-fill: #fff;
          -fx-background-color: #d9534f;
          -fx-border-color: #d43f3a;
        }
```

1.  现在，让我们添加一些具有不同选择器的标签，即`badge`和`badge-info`：

```java
        Label label = new Label("Default Label");
        label.getStyleClass().add("badge");
        gridPane.add(label, 0, 2);

        Label infoLabel = new Label("Info Label");
        infoLabel.getStyleClass().add("badge");
        infoLabel.getStyleClass().add("badge-info");
        gridPane.add(infoLabel, 1, 2);
```

1.  前面选择器的 CSS 属性如下：

```java
        .badge{
          -fx-label-padding: 6,7,6,7;
          -fx-font-size: 12px;
          -fx-font-weight: 700;
          -fx-text-fill: #fff;
          -fx-text-alignment: center;
          -fx-background-color: #777;
          -fx-border-radius: 4;
        }

        .badge-info{
          -fx-background-color: #3a87ad;
        }
        .badge-warning {
          -fx-background-color: #f89406;
        }
```

1.  让我们添加一个带有`big-input`类的`TextField`：

```java
        TextField bigTextField = new TextField();
        bigTextField.getStyleClass().add("big-input");
        gridPane.add(bigTextField, 0, 3, 3, 1);
```

1.  我们定义 CSS 属性，使得文本框的内容尺寸大且颜色为红色：

```java
        .big-input{
          -fx-text-fill: red;
          -fx-font-size: 18px;
          -fx-font-style: italic;
          -fx-font-weight: bold;
        }
```

1.  现在，让我们添加一些单选按钮：

```java
        ToggleGroup group = new ToggleGroup();
        RadioButton bigRadioOne = new RadioButton("First");
        bigRadioOne.getStyleClass().add("big-radio");
        bigRadioOne.setToggleGroup(group);
        bigRadioOne.setSelected(true);
        gridPane.add(bigRadioOne, 0, 4);
        RadioButton bigRadioTwo = new RadioButton("Second");
        bigRadioTwo.setToggleGroup(group);
        bigRadioTwo.getStyleClass().add("big-radio");
        gridPane.add(bigRadioTwo, 1, 4);
```

1.  我们定义 CSS 属性，使得单选按钮的标签尺寸大且颜色为绿色：

```java
        .big-radio{
          -fx-text-fill: green;
          -fx-font-size: 18px;
          -fx-font-weight: bold;
          -fx-background-color: yellow;
          -fx-padding: 5;
        }
```

1.  最后，我们将`javafx.scene.layout.GridPane`添加到场景图中，并在`javafx.stage.Stage`上渲染场景图。我们还需要将`stylesheet.css`与`Scene`关联起来：

```java
        Scene scene = new Scene(gridPane, 600, 500);
        scene.getStylesheets().add("com/packt/stylesheet.css");
        stage.setTitle("Age calculator");
        stage.setScene(scene);
        stage.show();
```

1.  添加一个`main()`方法来启动 GUI：

```java
        public static void main(String[] args) {
          Application.launch(args);
        }
```

完整的代码可以在这里找到：`Chapter16/3_css_javafx`。

我们提供了两个运行脚本，`run.bat`和`run.sh`，在`Chapter16/3_css_javafx`下。`run.bat`用于在 Windows 上运行应用程序，`run.sh`用于在 Linux 上运行应用程序。

使用`run.bat`或`run.sh`运行应用程序，你会看到以下的 GUI：

![](img/fa402889-ce42-4e51-9568-7c90a1ab4cf9.png)

# 它是如何工作的...

在这个示例中，我们利用类名及其对应的 CSS 选择器来关联具有不同样式属性的组件。JavaFX 支持 CSS 属性的子集，不同类型的 JavaFX 组件适用不同的属性。JavaFX CSS 参考指南（[`docs.oracle.com/javase/8/javafx/api/javafx/scene/doc-files/cssref.html`](http://docs.oracle.com/javase/8/javafx/api/javafx/scene/doc-files/cssref.html)）将帮助您识别支持的 CSS 属性。

所有的场景图节点都是从一个抽象类`javax.scene.Node`继承的。这个抽象类提供了一个 API，`getStyleClass()`，返回一个添加到节点或 JavaFX 组件的类名列表（这些都是普通的`String`）。由于这是一个简单的类名列表，我们甚至可以通过使用`getStyleClass().add("new-class-name")`来添加更多的类名。

使用类名的优势在于它允许我们通过共同的类名对类似的组件进行分组。这种技术在 Web 开发世界中被广泛使用。假设我在 HTML 页面上有一个按钮列表，我希望在单击每个按钮时执行类似的操作。为了实现这一点，我将为每个按钮分配相同的类，比如`my-button`，然后使用`document.getElementsByClassName('my-button')`来获取这些按钮的数组。现在我们可以循环遍历获取的按钮数组并添加所需的操作处理程序。

在为组件分配类之后，我们需要为给定的类名编写 CSS 属性。然后这些属性将应用于所有具有相同类名的组件。

让我们从我们的配方中挑选一个组件，看看我们是如何进行样式化的。考虑以下具有两个类`btn`和`btn-primary`的组件： 

```java
primaryBtn.getStyleClass().add("btn");
primaryBtn.getStyleClass().add("btn-primary");
```

我们使用了选择器`.btn`和`.btn-primary`，并将所有的 CSS 属性分组在这些选择器下，如下所示：

```java
.btn{
  -fx-border-radius: 4px;
  -fx-border: 2px;
  -fx-font-size: 18px;
  -fx-font-weight: normal;
  -fx-text-align: center;
}
```

```java
.btn-primary {
  -fx-text-fill: #fff;
  -fx-background-color: #337ab7;
  -fx-border-color: #2e6da4;
}
```

请注意，在 CSS 中，我们有一个`color`属性，它在 JavaFX 中的等价物是`-fx-text-fill`。其余的 CSS 属性，即`border-radius`、`border`、`font-size`、`font-weight`、`text-align`、`background-color`和`border-color`，都以`-fx-`为前缀。

重要的是如何将样式表与`Scene`组件关联起来。

`scene.getStylesheets().add("com/packt/stylesheet.css");`这行代码将样式表与场景组件关联起来。由于`getStylesheets()`返回一个字符串列表，我们可以向其中添加多个字符串，这意味着我们可以将多个样式表关联到一个场景中。

`getStylesheets()`的文档说明如下：

"URL 是形式为[scheme:][//authority][path]的分层 URI。如果 URL 没有[scheme:]组件，则 URL 被视为仅有[path]组件。[path]的任何前导'/'字符都将被忽略，[path]将被视为相对于应用程序类路径的路径。"

在我们的配方中，我们只使用了`path`组件，因此它在类路径中查找文件。这就是我们将样式表添加到与场景相同的包中的原因。这是使其在类路径上可用的更简单的方法。

# 创建条形图

当数据以表格形式表示时，很难理解，但当数据以图表形式图形化表示时，它对眼睛来说是舒适的，易于理解的。我们已经看到了很多用于 Web 应用程序的图表库。然而，在桌面应用程序方面缺乏相同的支持。Swing 没有本地支持创建图表，我们不得不依赖于第三方应用程序，如**JFreeChart**（[`www.jfree.org/jfreechart/`](http://www.jfree.org/jfreechart/)）。不过，JavaFX 具有创建图表的本地支持，我们将向您展示如何使用 JavaFX 图表组件以图表形式表示数据。

JavaFX 支持以下图表类型：

+   条形图

+   折线图

+   饼图

+   散点图

+   区域图

+   气泡图

在接下来的几个配方中，我们将介绍每种图表类型的构建。将每种图表类型分开成自己的配方将有助于我们以更简单的方式解释配方，并有助于更好地理解。

这个配方将全部关于条形图。一个样本条形图看起来像这样：

![](img/07097a6b-2eb0-4075-a3d1-b4e10a38f206.png)

条形图可以在*x*轴上的每个值上具有单个条或多个条（如前面的图表中）。多个条可以帮助我们比较*x*轴上每个值的多个值点。

# 准备工作

由于我们知道从 Oracle JDK 11 开始和 Open JDK 10 开始，JavaFX 库不会随 JDK 安装一起提供，因此我们必须从[`gluonhq.com/products/javafx/`](https://gluonhq.com/products/javafx/)下载 JavaFX SDK，并使用`-p`选项将 SDK 的`lib`文件夹中的 JAR 包包含在模块路径上，如下所示：

```java
javac -p "PATH_TO_JAVAFX_SDK_LIB" <other parts of the command line> 

#Windows
java -p "PATH_TO_JAVAFX_SDK_LIB;COMPILED_CODE" <other parts of the command line> 
#Linux 
java -p "PATH_TO_JAVAFX_SDK_LIB:COMPILED_CODE" <other parts of the command line>
```

我们将使用来自学生表现机器学习存储库（[`archive.ics.uci.edu/ml/datasets/Student+Performance`](https://archive.ics.uci.edu/ml/datasets/Student+Performance)）的数据子集。数据集包括学生在数学和葡萄牙语两门科目中的表现，以及他们的社会背景信息，如父母的职业和教育等其他信息。数据集中有很多属性，但我们将选择以下属性：

+   学生的性别

+   学生的年龄

+   父亲的教育

+   父亲的职业

+   母亲的教育

+   母亲的职业

+   学生是否参加了额外课程

+   第一学期成绩

+   第二学期成绩

+   最终成绩

正如我们之前提到的，数据中捕获了很多属性，但我们只需要一些重要的属性就可以绘制一些有用的图表。因此，我们已经从机器学习存储库中的数据集中提取了信息，并将其放入了一个单独的文件中，该文件可以在书籍的代码下载中的`Chapter16/4_bar_charts/src/gui/com/packt/students`中找到。以下是从学生文件中摘录的一部分内容：

```java
"F";18;4;4;"at_home";"teacher";"no";"5";"6";6
"F";17;1;1;"at_home";"other";"no";"5";"5";6
"F";15;1;1;"at_home";"other";"yes";"7";"8";10
"F";15;4;2;"health";"services";"yes";"15";"14";15
"F";16;3;3;"other";"other";"yes";"6";"10";10
"M";16;4;3;"services";"other";"yes";"15";"15";15
```

条目由分号(`;`)分隔。已解释每个条目代表的内容。教育信息（字段 3 和 4）是一个数字值，其中每个数字代表教育水平，如下所示：

+   `0`：无

+   `1`：小学教育（四年级）

+   `2`：五到九年级

+   `3`：中等教育

+   `4`：高等教育

我们已经创建了一个用于处理学生文件的模块。模块名称是`student.processor`，其代码可以在`Chapter16/101_student_data_processor`中找到。因此，如果您想更改那里的任何代码，可以通过运行`build-jar.bat`或`build-jar.sh`文件来重新构建 JAR。这将在`mlib`目录中创建一个模块化的 JAR，`student.processor.jar`。然后，您必须将此模块化 JAR 替换为本配方的`mlib`目录中的模块化 JAR，即`Chapter16/4_bar_charts/mlib`。

我们建议您从`Chapter16/101_student_data_processor`中提供的源代码构建`student.processor`模块化 jar。我们提供了`build-jar.bat`和`build-jar.sh`脚本来帮助您构建 JAR。您只需运行与您的平台相关的脚本，然后将构建的 jar 复制到`101_student_data_processor/mlib`中，然后再复制到`4_bar_charts/mlib`中。

这样，我们可以在涉及图表的所有配方中重复使用此模块。

# 操作步骤...

1.  首先，创建`GridPane`并配置它以放置我们将要创建的图表：

```java
        GridPane gridPane = new GridPane();
        gridPane.setAlignment(Pos.CENTER);
        gridPane.setHgap(10);
        gridPane.setVgap(10);
        gridPane.setPadding(new Insets(25, 25, 25, 25));
```

1.  使用`student.processor`模块中的`StudentDataProcessor`类来解析学生文件并将数据加载到`Student`的`List`中：

```java
        StudentDataProcessor sdp = new StudentDataProcessor();
        List<Student> students = sdp.loadStudent();
```

1.  原始数据，即`Student`对象的列表，对于绘制图表来说并不有用，因此我们需要通过根据他们母亲和父亲的教育水平对学生进行分组，并计算这些学生的成绩（所有三个学期）的平均值来处理学生的成绩。为此，我们将编写一个简单的方法，接受`List<Student>`，一个分组函数（即学生需要根据其值进行分组的值），以及一个映射函数（即用于计算平均值的值）：

```java
        private Map<ParentEducation, IntSummaryStatistics> summarize(
          List<Student> students,
          Function<Student, ParentEducation> classifier,
          ToIntFunction<Student> mapper
        ){
          Map<ParentEducation, IntSummaryStatistics> statistics =
            students.stream().collect(
              Collectors.groupingBy(
                classifier,
                Collectors.summarizingInt(mapper)
              )
          );
          return statistics;
        }
```

前面的方法使用了新的基于流的 API。这些 API 非常强大，它们使用`Collectors.groupingBy()`对学生进行分组，然后使用`Collectors.summarizingInt()`计算他们的成绩统计信息。

1.  条形图的数据以`XYChart.Series`的实例提供。每个系列对于给定的*x*值会产生一个*y*值，这是给定*x*值的一个条形图。我们将有多个系列，一个用于每个学期，即第一学期成绩、第二学期成绩和期末成绩。让我们创建一个方法，该方法接受每个学期成绩的统计数据和`seriesName`，并返回一个`series`对象：

```java
        private XYChart.Series<String,Number> getSeries(
            String seriesName,
            Map<ParentEducation, IntSummaryStatistics> statistics
        ){
         XYChart.Series<String,Number> series = new XYChart.Series<>();
          series.setName(seriesName);
          statistics.forEach((k, v) -> {
            series.getData().add(
              new XYChart.Data<String, Number>(
                k.toString(),v.getAverage()
              )
            );
          });
          return series;
        }
```

1.  我们将创建两个条形图——一个用于母亲教育的平均成绩，另一个用于父亲教育的平均成绩。为此，我们将创建一个方法，该方法将获取`List<Student>`和一个分类器，即一个将返回用于对学生进行分组的值的函数。该方法将进行必要的计算并返回一个`BarChart`对象：

```java
       private BarChart<String, Number> getAvgGradeByEducationBarChart(
          List<Student> students,
          Function<Student, ParentEducation> classifier
        ){
          final CategoryAxis xAxis = new CategoryAxis();
          final NumberAxis yAxis = new NumberAxis();
          final BarChart<String,Number> bc = 
                new BarChart<>(xAxis,yAxis);
          xAxis.setLabel("Education");
          yAxis.setLabel("Grade");
          bc.getData().add(getSeries(
            "G1",
            summarize(students, classifier, Student::getFirstTermGrade)
          ));
          bc.getData().add(getSeries(
            "G2",
           summarize(students, classifier, Student::getSecondTermGrade)
          ));
          bc.getData().add(getSeries(
            "Final",
            summarize(students, classifier, Student::getFinalGrade)
          ));
          return bc;
        }
```

1.  为母亲的教育平均成绩创建`BarChart`，并将其添加到`gridPane`：

```java
        BarChart<String, Number> avgGradeByMotherEdu = 
            getAvgGradeByEducationBarChart(
              students, 
              Student::getMotherEducation
            );
        avgGradeByMotherEdu.setTitle(
            "Average grade by Mother's Education"
        );
        gridPane.add(avgGradeByMotherEdu, 1,1);
```

1.  为父亲的教育平均成绩创建`BarChart`并将其添加到`gridPane`：

```java
        BarChart<String, Number> avgGradeByFatherEdu = 
            getAvgGradeByEducationBarChart(
              students, 
              Student::getFatherEducation
            );
        avgGradeByFatherEdu.setTitle(
            "Average grade by Father's Education");
        gridPane.add(avgGradeByFatherEdu, 2,1);
```

1.  使用`gridPane`创建一个场景图，并将其设置为`Stage`：

```java
        Scene scene = new Scene(gridPane, 800, 600);
        stage.setTitle("Bar Charts");
        stage.setScene(scene);
        stage.show();
```

完整的代码可以在`Chapter16/4_bar_charts`中找到。

我们提供了两个运行脚本：`run.bat`和`run.sh`，位于`Chapter16/4_bar_charts`下。`run.bat`脚本将用于在 Windows 上运行应用程序，`run.sh`将用于在 Linux 上运行应用程序。

使用`run.bat`或`run.sh`运行应用程序，您将看到以下 GUI：

![](img/68ae46d4-b460-449d-9774-a2e18a499c33.png)

# 它是如何工作的...

让我们首先看看创建`BarChart`需要什么。`BarChart`是一个基于两个轴的图表，其中数据绘制在两个轴上，即*x*轴（水平轴）和*y*轴（垂直轴）。另外两个基于轴的图表是面积图、气泡图和折线图。

在 JavaFX 中，支持两种类型的轴：

+   `javafx.scene.chart.CategoryAxis`：这支持轴上的字符串值

+   `javafx.scene.chart.NumberAxis`：这支持轴上的数值

在我们的示例中，我们使用`CategoryAxis`作为*x*轴创建了`BarChart`，在这里我们绘制教育，使用`NumberAxis`作为*y*轴，在这里我们绘制等级，如下所示：

```java
final CategoryAxis xAxis = new CategoryAxis();
final NumberAxis yAxis = new NumberAxis();
final BarChart<String,Number> bc = new BarChart<>(xAxis,yAxis);
xAxis.setLabel("Education");
yAxis.setLabel("Grade");
```

在接下来的几段中，我们将向您展示`BarChart`的绘制方式。

要在`BarChart`上绘制的数据应该是一对值，其中每对表示*(x, y)*值，即*x*轴上的一个点和*y*轴上的一个点。这一对值由`javafx.scene.chart.XYChart.Data`表示。`Data`是`XYChart`内的一个嵌套类，它表示基于两个轴的图表的单个数据项。`XYChart.Data`对象可以很简单地创建，如下所示：

```java
XYChart.Data item = new XYChart.Data("Cat1", "12");
```

这只是一个数据项。一个图表可以有多个数据项，也就是一系列数据项。为了表示一系列数据项，JavaFX 提供了一个名为`javafx.scene.chart.XYChart.Series`的类。这个`XYChart.Series`对象是`XYChart.Data`项的一个命名系列。让我们创建一个简单的系列，如下所示：

```java
XYChart.Series<String,Number> series = new XYChart.Series<>();
series.setName("My series");
series.getData().add(
  new XYChart.Data<String, Number>("Cat1", 12)
);
series.getData().add(
  new XYChart.Data<String, Number>("Cat2", 3)
);
series.getData().add(
  new XYChart.Data<String, Number>("Cat3", 16)
);
```

`BarChart`可以有多个数据系列。如果我们为其提供多个系列，那么在*x*轴上的每个数据点上将有多个条形图。为了演示这是如何工作的，我们将坚持使用一个系列。但是我们示例中的`BarChart`类使用了多个系列。让我们将系列添加到`BarChart`，然后将其渲染到屏幕上：

```java
bc.getData().add(series);
Scene scene = new Scene(bc, 800, 600);
stage.setTitle("Bar Charts");
stage.setScene(scene);
stage.show();
```

这导致了以下图表：

![](img/a2a1490f-110f-4139-aac4-6685cced09c1.png)

此示例的另一个有趣部分是根据母亲和父亲的教育对学生进行分组，然后计算他们的第一学期、第二学期和期末成绩的平均值。进行分组和平均计算的代码如下：

```java
Map<ParentEducation, IntSummaryStatistics> statistics =
        students.stream().collect(
  Collectors.groupingBy(
    classifier,
    Collectors.summarizingInt(mapper)
  )
);
```

上述代码执行以下操作：

+   它从`List<Student>`创建了一个流。

+   它使用`collect()`方法将此流减少到所需的分组。

+   `collect()`的重载版本之一需要两个参数。第一个是返回学生需要分组的值的函数。第二个参数是一个额外的映射函数，将分组的学生对象映射为所需的格式。在我们的情况下，所需的格式是对任何一个成绩值的学生组获取`IntSummaryStatistics`。

设置条形图的数据和创建所需的对象以填充`BarChart`实例是本示例的重要部分；了解它们将使您对示例有更清晰的认识。

# 创建饼图

饼图，顾名思义，是带有切片（连接或分离）的圆形图表，其中每个切片及其大小表示切片代表的项目的大小。饼图用于比较不同类别、产品等的大小。这是一个示例饼图的样子：

![](img/e42a09ae-bc16-4a33-909c-6c55a4522f70.png)

# 准备工作

由于我们知道从 Oracle JDK 11 开始和 Open JDK 10 开始，JavaFX 库不会随 JDK 安装一起提供，因此我们将不得不从[`gluonhq.com/products/javafx/`](https://gluonhq.com/products/javafx/)下载 JavaFX SDK，并使用`-p`选项将 SDK 的`lib`文件夹中存在的 JAR 包含在模块路径中，如下所示：

```java
javac -p "PATH_TO_JAVAFX_SDK_LIB" <other parts of the command line> 

#Windows
java -p "PATH_TO_JAVAFX_SDK_LIB;COMPILED_CODE" <other parts of the command line> 
#Linux 
java -p "PATH_TO_JAVAFX_SDK_LIB:COMPILED_CODE" <other parts of the command line>
```

我们将使用相同的学生数据（从机器学习存储库中获取并在我们这里处理）来创建饼图，这是我们在*创建条形图*示例中讨论过的。为此，我们创建了一个名为`student.processor`的模块，它将读取学生数据并为我们提供`Student`对象的列表。模块的源代码可以在`Chapter16/101_student_data_processor`找到。我们已经在本示例代码的`Chapter16/5_pie_charts/mlib`中提供了`student.processor`模块的模块化 jar。

我们建议您从`Chapter16/101_student_data_processor`中提供的源代码构建`student.processor`模块化 jar。我们提供了`build-jar.bat`和`build-jar.sh`脚本来帮助您构建 jar。您只需运行与您的平台相关的脚本，然后将构建的 jar 复制到`101_student_data_processor/mlib`中的`4_bar_charts/mlib`。

# 如何操作...

1.  让我们首先创建和配置`GridPane`来容纳我们的饼图：

```java
        GridPane gridPane = new GridPane();
        gridPane.setAlignment(Pos.CENTER);
        gridPane.setHgap(10);
        gridPane.setVgap(10);
        gridPane.setPadding(new Insets(25, 25, 25, 25));
```

1.  创建`StudentDataProcessor`的实例（来自`student.processor`模块）并使用它加载`Student`的`List`：

```java
        StudentDataProcessor sdp = new StudentDataProcessor();
        List<Student> students = sdp.loadStudent();
```

1.  现在我们需要按照他们母亲和父亲的职业来统计学生的数量。我们将编写一个方法，该方法将接受学生列表和分类器，即返回学生需要分组的值的函数。该方法返回`PieChart`的实例：

```java
        private PieChart getStudentCountByOccupation(
            List<Student> students,
            Function<Student, String> classifier
        ){
          Map<String, Long> occupationBreakUp = 
                  students.stream().collect(
            Collectors.groupingBy(
              classifier,
              Collectors.counting()
            )
          );
          List<PieChart.Data> pieChartData = new ArrayList<>();
          occupationBreakUp.forEach((k, v) -> {
            pieChartData.add(new PieChart.Data(k.toString(), v));
          });
          PieChart chart = new PieChart(
            FXCollections.observableList(pieChartData)
          );
          return chart;
        }
```

1.  我们将调用前面的方法两次，一次以母亲的职业作为分类器，另一次以父亲的职业作为分类器。然后将返回的`PieChart`实例添加到`gridPane`中。这应该在`start()`方法中完成：

```java
        PieChart motherOccupationBreakUp = 
        getStudentCountByOccupation(
          students, Student::getMotherJob
        );
        motherOccupationBreakUp.setTitle("Mother's Occupation");
        gridPane.add(motherOccupationBreakUp, 1,1);

        PieChart fatherOccupationBreakUp = 
        getStudentCountByOccupation(
          students, Student::getFatherJob
        );
        fatherOccupationBreakUp.setTitle("Father's Occupation");
        gridPane.add(fatherOccupationBreakUp, 2,1);
```

1.  下一步是使用`gridPane`创建场景图并将其添加到`Stage`：

```java
        Scene scene = new Scene(gridPane, 800, 600);
        stage.setTitle("Pie Charts");
        stage.setScene(scene);
        stage.show();
```

1.  可以通过调用`Application.launch`方法从主方法启动 UI：

```java
        public static void main(String[] args) {
          Application.launch(args);
        }
```

完整的代码可以在`Chapter16/5_pie_charts`找到。

我们提供了两个运行脚本，`run.bat`和`run.sh`，位于`Chapter16/5_pie_charts`下。`run.bat`脚本用于在 Windows 上运行应用程序，`run.sh`用于在 Linux 上运行应用程序。

使用`run.bat`或`run.sh`运行应用程序，您将看到以下 GUI：

![](img/4fcb686c-5890-47b5-8e73-05d663f16210.png)

# 工作原理...

在这个示例中，完成所有工作的最重要的方法是`getStudentCountByOccupation()`。它执行以下操作：

1.  它按职业对学生人数进行分组。这可以通过使用新的流式 API 的强大功能（作为 Java 8 的一部分添加）来完成一行代码：

```java
        Map<String, Long> occupationBreakUp = 
                    students.stream().collect(
          Collectors.groupingBy(
            classifier,
            Collectors.counting()
          )
        );
```

1.  构建`PieChart`所需的数据。`PieChart`实例的数据是`ObservableList`的`PieChart.Data`。我们首先利用前面步骤中获得的`Map`来创建`ArrayList`的`PieChart.Data`。然后，我们使用`FXCollections.observableList()`API 从`List<PieChart.Data>`中获取`ObservableList<PieChart.Data>`：

```java
        List<PieChart.Data> pieChartData = new ArrayList<>();
        occupationBreakUp.forEach((k, v) -> {
          pieChartData.add(new PieChart.Data(k.toString(), v));
        });
        PieChart chart = new PieChart(
          FXCollections.observableList(pieChartData)
        );
```

该示例中的另一个重要事项是我们使用的分类器：`Student::getMotherJob`和`Student::getFatherJob`。这是两个方法引用，它们在`Student`列表中的不同实例上调用`getMotherJob`和`getFatherJob`方法。

一旦我们获得了`PieChart`实例，我们将把它们添加到`GridPane`中，然后使用`GridPane`构建场景图。场景图必须与`Stage`相关联，才能在屏幕上呈现。

主方法通过调用`Application.launch(args);`方法启动 UI。

JavaFX 提供了用于创建不同类型图表的 API，例如以下类型：

+   区域图

+   气泡图

+   折线图

+   散点图

所有这些图表都是基于*x*和*y*轴的图表，可以像条形图一样构建。我们提供了一些示例实现来创建这些类型的图表，它们可以在这些位置找到：`Chapter16/5_2_area_charts`，`Chapter16/5_3_line_charts`，`Chapter16/5_4_bubble_charts`和`Chapter16/5_5_scatter_charts`。

# 在应用程序中嵌入 HTML

JavaFX 提供了通过`javafx.scene.web`包中定义的类来管理网页的支持。它支持加载网页，可以通过接受网页 URL 或接受网页内容来实现。它还管理网页的文档模型，应用相关的 CSS，并运行相关的 JavaScript 代码。它还扩展了 JavaScript 和 Java 代码之间的双向通信支持。

在这个示例中，我们将构建一个非常原始和简单的支持以下功能的网络浏览器：

+   浏览访问过的页面的历史记录

+   重新加载当前页面

+   用于接受 URL 的地址栏

+   用于加载输入的 URL 的按钮

+   显示网页

+   显示网页加载状态

# 准备工作

由于我们知道 JavaFX 库从 Oracle JDK 11 开始和 Open JDK 10 开始不再随 JDK 安装而来，我们将不得不从这里[`gluonhq.com/products/javafx/`](https://gluonhq.com/products/javafx/)下载 JavaFX SDK，并使用`-p`选项将 SDK 的`lib`文件夹中的 JAR 包包含在模块路径中，如下所示：

```java
javac -p "PATH_TO_JAVAFX_SDK_LIB" <other parts of the command line> 

#Windows
java -p "PATH_TO_JAVAFX_SDK_LIB;COMPILED_CODE" <other parts of the command line> 
#Linux 
java -p "PATH_TO_JAVAFX_SDK_LIB:COMPILED_CODE" <other parts of the command line>
```

我们需要一个互联网连接来测试页面的加载。因此，请确保您已连接到互联网。除此之外，没有特定的要求来使用这个示例。

# 如何做...

1.  让我们首先创建一个空方法的类，它将代表启动应用程序以及 JavaFX UI 的主要应用程序：

```java
        public class BrowserDemo extends Application{
          public static void main(String[] args) {
            Application.launch(args);
          }
          @Override
          public void start(Stage stage) {
            //this will have all the JavaFX related code
          }
        }
```

在接下来的步骤中，我们将在`start(Stage stage)`方法中编写所有的代码。

1.  让我们创建一个`javafx.scene.web.WebView`组件，它将呈现我们的网页。这个组件有必需的`javafx.scene.web.WebEngine`实例，用于管理网页的加载：

```java
        WebView webView = new WebView();
```

1.  获取`webView`使用的`javafx.scene.web.WebEngine`实例。我们将使用这个`javafx.scene.web.WebEngine`实例来浏览历史记录并加载其他网页。然后，我们将默认加载 URL，[`www.google.com`](http://www.google.com)：

```java
        WebEngine webEngine = webView.getEngine();
        webEngine.load("http://www.google.com/");
```

1.  现在让我们创建一个`javafx.scene.control.TextField`组件，它将充当我们浏览器的地址栏：

```java
        TextField webAddress = new
        TextField("http://www.google.com/");
```

1.  我们希望根据完全加载的网页的标题和 URL 来更改浏览器和地址栏中的网页标题。这可以通过监听从`javafx.scene.web.WebEngine`实例获取的`javafx.concurrent.Worker`的`stateProperty`的更改来实现：

```java
        webEngine.getLoadWorker().stateProperty().addListener(
          new ChangeListener<State>() {
            public void changed(ObservableValue ov, 
                                State oldState, State newState) {
              if (newState == State.SUCCEEDED) {
                stage.setTitle(webEngine.getTitle());
                webAddress.setText(webEngine.getLocation());
              }
            }
          }
        );
```

1.  让我们创建一个`javafx.scene.control.Button`实例，点击后将加载地址栏中输入的 URL 标识的网页：

```java
        Button goButton = new Button("Go");
        goButton.setOnAction((event) -> {
          String url = webAddress.getText();
          if ( url != null && url.length() > 0){
            webEngine.load(url);
          }
        });
```

1.  让我们创建一个`javafx.scene.control.Button`实例，点击后将转到历史记录中的上一个网页。为此，我们将在操作处理程序内执行 JavaScript 代码`history.back()`：

```java
        Button prevButton = new Button("Prev");
        prevButton.setOnAction(e -> {
          webEngine.executeScript("history.back()");
        });
```

1.  让我们创建一个`javafx.scene.control.Button`实例，点击后将转到`javafx.scene.web.WebEngine`实例维护的历史记录中的下一个条目。为此，我们将使用`javafx.scene.web.WebHistory` API：

```java
        Button nextButton = new Button("Next");
        nextButton.setOnAction(e -> {
          WebHistory wh = webEngine.getHistory();
          Integer historySize = wh.getEntries().size();
          Integer currentIndex = wh.getCurrentIndex();
          if ( currentIndex < (historySize - 1)){
            wh.go(1);
          }
        });
```

1.  接下来是重新加载当前页面的按钮。再次，我们将使用`javafx.scene.web.WebEngine`来重新加载当前页面：

```java
        Button reloadButton = new Button("Refresh");
        reloadButton.setOnAction(e -> {
          webEngine.reload();
        });
```

1.  现在我们需要将迄今为止创建的所有组件，即`prevButton`、`nextButton`、`reloadButton`、`webAddress`和`goButton`分组，以使它们在水平方向上对齐。为此，我们将使用`javafx.scene.layout.HBox`，并设置相关间距和填充，使组件看起来间距均匀：

```java
        HBox addressBar = new HBox(10);
        addressBar.setPadding(new Insets(10, 5, 10, 5));
        addressBar.setHgrow(webAddress, Priority.ALWAYS);
        addressBar.getChildren().addAll(
          prevButton, nextButton, reloadButton, webAddress, goButton
        );
```

1.  我们想知道网页是否正在加载以及是否已经完成。让我们创建一个`javafx.scene.layout.Label`字段来更新网页加载状态。然后，我们监听`javafx.scene.web.WebEngine`实例的`workDoneProperty`的更新，这可以从`javafx.concurrent.Worker`实例中获取：

```java
Label websiteLoadingStatus = new Label();
webEngine
  .getLoadWorker()
  .workDoneProperty()
  .addListener(
    new ChangeListener<Number>(){

      public void changed(
        ObservableValue ov, 
        Number oldState, 
        Number newState
      ) {
        if (newState.doubleValue() != 100.0){
          websiteLoadingStatus.setText(
            "Loading " + webAddress.getText());
        }else{
          websiteLoadingStatus.setText("Done");
        }
      }

    }
  );
```

1.  让我们垂直对齐整个地址栏（及其导航按钮）、`webView`和`websiteLoadingStatus`：

```java
        VBox root = new VBox();
        root.getChildren().addAll(
          addressBar, webView, websiteLoadingStatus
        );
```

1.  使用在前一步中创建的`VBox`实例作为根创建一个新的`Scene`对象：

```java
        Scene scene = new Scene(root);
```

1.  我们希望`javafx.stage.Stage`实例占据整个屏幕大小；为此，我们将使用`Screen.getPrimary().getVisualBounds()`。然后，像往常一样，我们将在舞台上呈现场景图：

```java
        Rectangle2D primaryScreenBounds = 
                    Screen.getPrimary().getVisualBounds();
        stage.setTitle("Web Browser");
        stage.setScene(scene);
        stage.setX(primaryScreenBounds.getMinX());
        stage.setY(primaryScreenBounds.getMinY());
        stage.setWidth(primaryScreenBounds.getWidth());
        stage.setHeight(primaryScreenBounds.getHeight());
        stage.show();
```

完整的代码可以在`Chapter16/6_embed_html`位置找到。

在`Chapter16/6_embed_html`下提供了两个运行脚本，`run.bat`和`run.sh`。`run.bat`脚本用于在 Windows 上运行应用程序，`run.sh`用于在 Linux 上运行应用程序。

使用`run.bat`或`run.sh`运行应用程序，您将看到以下 GUI：

![](img/be7f6c16-0275-4ca8-99b7-b11ec644d75d.png)

# 它是如何工作的...

与 Web 相关的 API 可在`javafx.web`模块中使用，因此我们需要在`module-info`中声明它：

```java
module gui{
  requires javafx.controls;
  requires javafx.web;
  opens com.packt;
}
```

下面是在处理 JavaFX 中的网页时`javafx.scene.` web 包中的重要类：

+   `WebView`：此 UI 组件使用`WebEngine`来管理加载、渲染和与网页的交互。

+   `WebEngine`：这是处理加载和管理网页的主要组件。

+   `WebHistory`：记录当前`WebEngine`实例中访问的网页。

+   `WebEvent`：这些是传递给由 JavaScript 事件调用的`WebEngine`事件处理程序的实例。

在我们的示例中，我们使用了前三个类。

我们不直接创建`WebEngine`的实例；相反，我们利用`WebView`来获取由其管理的`WebEngine`实例的引用。`WebEngine`实例通过将加载页面的任务提交给`javafx.concurrent.Worker`实例来异步加载网页。然后，我们在这些 worker 实例属性上注册更改侦听器，以跟踪加载网页的进度。在这个示例中，我们使用了两个这样的属性，即`stateProperty`和`workDoneProperty`。前者跟踪 worker 状态的更改，后者跟踪工作完成的百分比。

Worker 可以经历以下状态（如列在`javafx.concurrent.Worker.State`枚举中）：

+   `取消`

+   `失败`

+   `准备就绪`

+   `运行中`

+   `已安排`

+   `成功`

在我们的教程中，我们只检查`SUCCEEDED`，但您也可以增强它以检查`FAILED`。这将帮助我们报告无效的 URL，甚至从事件对象中获取消息并显示给用户。

我们添加监听器以跟踪属性变化的方式是使用`*Property()`上的`addListener()`方法，其中`*`可以是`state`、`workDone`或工作器的任何其他属性，这些属性已公开为属性：

```java
webEngine
  .getLoadWorker()
  .stateProperty()
  .addListener( 
    new ChangeListener<State>() {
      public void changed(ObservableValue ov, 
       State oldState, State newState) {
         //event handler code here
       }
    }
);

webEngine
  .getLoadWorker()
  .workDoneProperty()
  .addListener(
    new ChangeListener<Number>(){
      public void changed(ObservableValue ov, 
        Number oldState, Number newState) {
          //event handler code here
      }
   }
);
```

然后，`javafx.scene.web.WebEngine`组件还支持以下功能：

+   重新加载当前页面

+   获取加载的页面的历史记录

+   执行 JavaScript 代码

+   监听 JavaScript 属性，例如显示警报框或确认框

+   使用`getDocument()`方法与网页的文档模型进行交互

在这个教程中，我们还介绍了从`WebEngine`获取的`WebHistory`的使用。`WebHistory`存储了给定`WebEngine`实例加载的网页，这意味着一个`WebEngine`实例将有一个`WebHistory`实例。`WebHistory`支持以下功能：

+   使用`getEntries()`方法获取条目列表。这也将为我们获取历史记录中的条目数。这在导航历史记录时是必需的；否则，我们将遇到索引越界异常。

+   获取`currentIndex`，即其在`getEntries()`列表中的索引。

+   导航到`WebHistory`的条目列表中的特定条目。这可以通过使用`go()`方法实现，该方法接受一个偏移量。该偏移量指示要加载的网页相对于当前位置。例如，*+1*表示下一个条目，*-1*表示上一个条目。检查边界条件很重要；否则，您将会在*0*之前，即*-1*之前，或者超出条目列表大小。

# 还有更多...

在这个教程中，我们向您展示了使用 JavaFX 提供的支持来创建一个基本的 Web 浏览器的方法。您可以增强它以支持以下功能：

+   更好的错误处理和用户消息，即通过跟踪工作器的状态变化来显示网址是否有效

+   多个标签页

+   书签

+   将浏览器的状态存储在本地，以便下次运行时加载所有书签和历史记录。

# 在应用程序中嵌入媒体

JavaFX 提供了一个组件`javafx.scene.media.MediaView`，用于查看视频和听音频。该组件由一个媒体引擎`javafx.scene.media.MediaPlayer`支持，该引擎加载和管理媒体的播放。

在这个教程中，我们将学习如何播放一个示例视频，并通过使用媒体引擎上的方法来控制其播放。

# 准备就绪

由于我们知道 JavaFX 库从 Oracle JDK 11 开始和 Open JDK 10 开始不再随 JDK 安装，我们将不得不从这里[`gluonhq.com/products/javafx/`](https://gluonhq.com/products/javafx/)下载 JavaFX SDK，并使用`-p`选项将 SDK 的`lib`文件夹中的 JAR 包包含在模块路径上，如下所示：

```java
javac -p "PATH_TO_JAVAFX_SDK_LIB" <other parts of the command line> 

#Windows
java -p "PATH_TO_JAVAFX_SDK_LIB;COMPILED_CODE" <other parts of the command line> 
#Linux 
java -p "PATH_TO_JAVAFX_SDK_LIB:COMPILED_CODE" <other parts of the command line>
```

我们将使用位于`Chapter16/7_embed_audio_video/sample_video1.mp4`的示例视频。

# 如何做...

1.  首先，让我们创建一个带有空方法的类，该类将代表启动应用程序以及 JavaFX UI 的主要应用程序：

```java
        public class EmbedAudioVideoDemo extends Application{
          public static void main(String[] args) {
            Application.launch(args);
          }
          @Override
          public void start(Stage stage) {
            //this will have all the JavaFX related code
          }
        }
```

1.  为位于`Chapter16/7_embed_audio_video/sample_video1.mp4`的视频创建一个`javafx.scene.media.Media`对象：

```java
        File file = new File("sample_video1.mp4");
        Media media = new Media(file.toURI().toString());
```

1.  使用上一步创建的`javafx.scene.media.Media`对象创建一个新的媒体引擎`javafx.scene.media.MediaPlayer`：

```java
        MediaPlayer mediaPlayer = new MediaPlayer(media);
```

1.  通过在`javafx.scene.media.MediaPlayer`对象的`statusProperty`上注册更改监听器来跟踪媒体播放器的状态：

```java
        mediaPlayer.statusProperty().addListener(
                    new ChangeListener<Status>() {
          public void changed(ObservableValue ov, 
                              Status oldStatus, Status newStatus) {
            System.out.println(oldStatus +"->" + newStatus);
          }
        });
```

1.  现在让我们使用上一步创建的媒体引擎来创建一个媒体查看器：

```java
        MediaView mediaView = new MediaView(mediaPlayer);
```

1.  我们将限制媒体查看器的宽度和高度：

```java
        mediaView.setFitWidth(350);
        mediaView.setFitHeight(350); 
```

1.  接下来，我们创建三个按钮来暂停视频播放、恢复播放和停止播放。我们将使用`javafx.scene.media.MediaPlayer`类中的相关方法：

```java
        Button pauseB = new Button("Pause");
        pauseB.setOnAction(e -> {
          mediaPlayer.pause();
        });

        Button playB = new Button("Play");
        playB.setOnAction(e -> {
          mediaPlayer.play();
        });

        Button stopB = new Button("Stop");
        stopB.setOnAction(e -> {
          mediaPlayer.stop();
        });
```

1.  使用`javafx.scene.layout.HBox`水平对齐所有这些按钮：

```java
        HBox controlsBox = new HBox(10);
        controlsBox.getChildren().addAll(pauseB, playB, stopB);
```

1.  使用`javafx.scene.layout.VBox`垂直对齐媒体查看器和按钮栏：

```java
        VBox vbox = new VBox();
        vbox.getChildren().addAll(mediaView, controlsBox);
```

1.  使用`VBox`对象创建一个新的场景图，并将其设置为舞台对象：

```java
        Scene scene = new Scene(vbox);
        stage.setScene(scene);
        // Name and display the Stage.
        stage.setTitle("Media Demo");
```

1.  在显示器上呈现舞台：

```java
        stage.setWidth(400);
        stage.setHeight(400);
        stage.show();
```

完整的代码可以在`Chapter16/7_embed_audio_video`中找到。

我们提供了两个运行脚本，`run.bat`和`run.sh`，位于`Chapter16/7_embed_audio_video`下。`run.bat`脚本将用于在 Windows 上运行应用程序，`run.sh`将用于在 Linux 上运行应用程序。

使用`run.bat`或`run.sh`运行应用程序，您将看到以下 GUI：

![](img/ba1e14aa-5eca-4fd3-b6f1-eea3e18edad4.png)

# 它是如何工作的...

用于媒体播放的`javafx.scene.media`包中的重要类如下：

+   `Media`：表示媒体的来源，即视频或音频。这以 HTTP/HTTPS/FILE 和 JAR URL 的形式接受来源。

+   `MediaPlayer`：管理媒体的播放。

+   `MediaView`：这是允许查看媒体的 UI 组件。

还有一些其他类，但我们在本示例中没有涵盖它们。与媒体相关的类位于`javafx.media`模块中。因此，请不要忘记在此处声明对其的依赖关系：

```java
module gui{
  requires javafx.controls;
  requires javafx.media;
  opens com.packt;
}
```

在本示例中，我们有一个位于`Chapter16/7_embed_audio_video/sample_video1.mp4`的示例视频，并且我们使用`java.io.File` API 来构建`File` URL 以定位视频：

```java
File file = new File("sample_video1.mp4");
Media media = new Media(file.toURI().toString());
```

使用`javafx.scene.media.MediaPlayer`类公开的 API 管理媒体播放。在本示例中，我们使用了其中的一些方法，即`play()`、`pause()`和`stop()`。`javafx.scene.media.MediaPlayer`类通过使用`javafx.scene.media.Media`对象进行初始化：

```java
MediaPlayer mediaPlayer = new MediaPlayer(media);
```

在 UI 上呈现媒体由`javafx.scene.media.MediaView`类管理，它由`javafx.scene.media.MediaPlayer`对象支持：

```java
MediaView mediaView = new MediaView(mediaPlayer);
```

我们可以使用`setFitWidth()`和`setFitHeight()`方法设置查看器的高度和宽度。

# 还有更多...

我们在 JavaFX 中提供了媒体支持的基本演示。还有很多可以探索的地方。您可以添加音量控制选项，向前或向后搜索选项，播放音频和音频均衡器。

# 向控件添加效果

以受控方式添加效果可以使用户界面看起来更好。有多种效果，如模糊、阴影、反射、绽放等。JavaFX 提供了一组类，位于`javafx.scene.effects`包下，可用于添加效果以增强应用程序的外观。此包在`javafx.graphics`模块中可用。

在本示例中，我们将查看一些效果——模糊、阴影和反射。

# 准备就绪

由于我们知道 JavaFX 库从 Oracle JDK 11 开始和 Open JDK 10 开始不再随 JDK 安装一起提供，因此我们必须从此处[`gluonhq.com/products/javafx/`](https://gluonhq.com/products/javafx/)下载 JavaFX SDK，并使用`-p`选项将 SDK 的`lib`文件夹中的 JAR 包包含在模块路径上，如下所示：

```java
javac -p "PATH_TO_JAVAFX_SDK_LIB" <other parts of the command line> 

#Windows
java -p "PATH_TO_JAVAFX_SDK_LIB;COMPILED_CODE" <other parts of the command line> 
#Linux 
java -p "PATH_TO_JAVAFX_SDK_LIB:COMPILED_CODE" <other parts of the command line>
```

# 如何做...

1.  让我们首先创建一个带有空方法的类，该类将代表启动应用程序以及 JavaFX UI 的主要应用程序：

```java
        public class EffectsDemo extends Application{
          public static void main(String[] args) {
            Application.launch(args);
          }
          @Override
          public void start(Stage stage) {
            //code added here in next steps
          }
        }
```

1.  随后的代码将在`start(Stage stage)`方法中编写。创建并配置`javafx.scene.layout.GridPane`：

```java
        GridPane gridPane = new GridPane();
        gridPane.setAlignment(Pos.CENTER);
        gridPane.setHgap(10);
        gridPane.setVgap(10);
        gridPane.setPadding(new Insets(25, 25, 25, 25));
```

1.  创建矩形，用于应用模糊效果：

```java
        Rectangle r1 = new Rectangle(100,25, Color.BLUE);
        Rectangle r2 = new Rectangle(100,25, Color.RED);
        Rectangle r3 = new Rectangle(100,25, Color.ORANGE);
```

1.  将`javafx.scene.effect.BoxBlur`添加到`Rectangle r1`，将`javafx.scene.effect.MotionBlur`添加到`Rectangle r2`，将`javafx.scene.effect.GaussianBlur`添加到`Rectangle r3`：

```java
        r1.setEffect(new BoxBlur(10,10,3));
        r2.setEffect(new MotionBlur(90, 15.0));
        r3.setEffect(new GaussianBlur(15.0));
```

1.  将矩形添加到`gridPane`：

```java
        gridPane.add(r1,1,1);
        gridPane.add(r2,2,1);
        gridPane.add(r3,3,1);
```

1.  创建三个圆，用于应用阴影：

```java
        Circle c1 = new Circle(20, Color.BLUE);
        Circle c2 = new Circle(20, Color.RED);
        Circle c3 = new Circle(20, Color.GREEN);
```

1.  将`javafx.scene.effect.DropShadow`添加到`c1`，将`javafx.scene.effect.InnerShadow`添加到`c2`：

```java
        c1.setEffect(new DropShadow(0, 4.0, 0, Color.YELLOW));
        c2.setEffect(new InnerShadow(0, 4.0, 4.0, Color.ORANGE));
```

1.  将这些圆添加到`gridPane`：

```java
        gridPane.add(c1,1,2);
        gridPane.add(c2,2,2);
        gridPane.add(c3,3,2);
```

1.  在我们将应用反射效果的简单文本`Reflection Sample`上：

```java
        Text t = new Text("Reflection Sample");
        t.setFont(Font.font("Arial", FontWeight.BOLD, 20));
        t.setFill(Color.BLUE);
```

1.  创建一个`javafx.scene.effect.Reflection`效果并将其添加到文本中：

```java
        Reflection reflection = new Reflection();
        reflection.setFraction(0.8);
        t.setEffect(reflection);
```

1.  将文本组件添加到`gridPane`：

```java
        gridPane.add(t, 1, 3, 3, 1);
```

1.  使用`gridPane`作为根节点创建一个场景图形：

```java
        Scene scene = new Scene(gridPane, 500, 300);
```

1.  将场景图形设置为舞台并在显示器上呈现它：

```java
        stage.setScene(scene);
        stage.setTitle("Effects Demo");
        stage.show();
```

完整的代码可以在`Chapter16/8_effects_demo`中找到。

我们提供了两个运行脚本，`Chapter16/8_effects_demo`下的`run.bat`和`run.sh`。`run.bat`脚本将用于在 Windows 上运行应用程序，`run.sh`将用于在 Linux 上运行应用程序。

使用`run.bat`或`run.sh`运行应用程序，您将看到以下 GUI：

![](img/b86be218-5d25-4a77-ae37-be9a25ee8b56.png)

# 它是如何工作的...

在这个示例中，我们使用了以下效果：

+   `javafx.scene.effect.BoxBlur`

+   `javafx.scene.effect.MotionBlur`

+   `javafx.scene.effect.GaussianBlur`

+   `javafx.scene.effect.DropShadow`

+   `javafx.scene.effect.InnerShadow`

+   `javafx.scene.effect.Reflection`

通过指定模糊效果的宽度和高度以及需要应用效果的次数来创建`BoxBlur`效果：

```java
BoxBlur boxBlur = new BoxBlur(10,10,3);
```

通过提供模糊的角度和半径来创建`MotionBlur`效果。这会产生一种在运动中捕获到的效果：

```java
MotionBlur motionBlur = new MotionBlur(90, 15.0);
```

通过提供效果的半径来创建`GaussianBlur`效果，并且该效果使用高斯公式来应用效果：

```java
GaussianBlur gb = new GaussianBlur(15.0);
```

`DropShadow`在物体后面添加阴影，而`InnerShadow`在物体内部添加阴影。每个阴影都有阴影的半径，阴影开始的*x*和*y*位置，以及阴影的颜色：

```java
DropShadow dropShadow = new DropShadow(0, 4.0, 0, Color.YELLOW);
InnerShadow innerShadow = new InnerShadow(0, 4.0, 4.0, Color.ORANGE);
```

`Reflection`是一个非常简单的效果，它添加了物体的反射。我们可以设置原始物体的反射比例：

```java
Reflection reflection = new Reflection();
reflection.setFraction(0.8);
```

# 还有更多...

还有很多其他效果：

+   混合效果，将两个不同的输入与预定义的混合方法混合

+   波纹效果，使更亮的部分看起来更亮。

+   发光效果，使物体发光

+   光照效果模拟了物体上的光源，从而使其呈现出 3D 外观。

我们建议您尝试以与我们尝试的方式相同的方式尝试这些效果。

# 使用 Robot API

**Robot API**用于模拟屏幕上的键盘和鼠标操作，这意味着您会指示代码在文本字段中输入一些文本，选择一个选项，然后单击一个按钮。来自 Web UI 测试背景的人可以将其与 Selenium 测试库联系起来。**抽象窗口工具包**（**AWT**）是 JDK 中的一个较旧的窗口工具包，提供了 Robot API，但在 JavaFX 上使用相同的 API 并不直接，需要一些技巧。JavaFX 窗口工具包称为**Glass**有自己的 Robot API（[`openjfx.io/javadoc/11/javafx.graphics/javafx/scene/robot/Robot.html`](https://openjfx.io/javadoc/11/javafx.graphics/javafx/scene/robot/Robot.html)），但这些 API 不是公开的。因此，在 OpenJFX 11 发布的一部分中，为其引入了新的公共 API。

在这个示例中，我们将使用 Robot API 来模拟一些 JavaFX UI 上的操作。

# 准备就绪

由于我们知道 JavaFX 库从 Oracle JDK 11 开始和 Open JDK 10 开始不再随 JDK 安装一起提供，因此我们必须从这里（[`gluonhq.com/products/javafx/`](https://gluonhq.com/products/javafx/)）下载 JavaFX SDK，并使用`-p`选项将 SDK 的`lib`文件夹中的 JAR 包含在模块路径上，如下所示：

```java
javac -p "PATH_TO_JAVAFX_SDK_LIB" <other parts of the command line> 

#Windows
java -p "PATH_TO_JAVAFX_SDK_LIB;COMPILED_CODE" <other parts of the command line> 
#Linux 
java -p "PATH_TO_JAVAFX_SDK_LIB:COMPILED_CODE" <other parts of the command line> 

```

在这个示例中，我们将创建一个简单的应用程序，接受用户的姓名，并在点击按钮时向用户打印一条消息。整个操作将使用 Robot API 模拟，并且在退出应用程序之前，我们将使用 Robot API 捕获屏幕。

# 如何做...

1.  创建一个简单的类`RobotApplication`，它扩展了`javafx.application.Application`并设置了测试 Robot API 所需的 UI，还创建了一个`javafx.scene.robot.Robot`的实例。这个类将被定义为`RobotAPIDemo`主类的静态内部类：

```java
public static class RobotApplication extends Application{

  @Override
  public void start(Stage stage) throws Exception{
    robot = new Robot();
    GridPane gridPane = new GridPane();
    gridPane.setAlignment(Pos.CENTER);
    gridPane.setHgap(10);
    gridPane.setVgap(10);
    gridPane.setPadding(new Insets(25, 25, 25, 25));

    Text appTitle = new Text("Robot Demo");
    appTitle.setFont(Font.font("Arial", 
        FontWeight.NORMAL, 15));
    gridPane.add(appTitle, 0, 0, 2, 1);

    Label nameLbl = new Label("Name");
    nameField = new TextField();
    gridPane.add(nameLbl, 0, 1);
    gridPane.add(nameField, 1, 1);

    greeting = new Button("Greet");
    gridPane.add(greeting, 1, 2);

    Text resultTxt = new Text();
    resultTxt.setFont(Font.font("Arial", 
        FontWeight.NORMAL, 15));
    gridPane.add(resultTxt, 0, 5, 2, 1);

    greeting.setOnAction((event) -> {

      String name = nameField.getText();
      StringBuilder resultBuilder = new StringBuilder();
      if ( name != null && name.length() > 0 ){
        resultBuilder.append("Hello, ")
            .append(name).append("\n");
      }else{
        resultBuilder.append("Please enter the name");
      }
      resultTxt.setText(resultBuilder.toString());
      btnActionLatch.countDown();
    });

    Scene scene = new Scene(gridPane, 300, 250);

    stage.setTitle("Age calculator");
    stage.setScene(scene);
    stage.setAlwaysOnTop(true);
    stage.addEventHandler(WindowEvent.WINDOW_SHOWN, e -> 
      Platform.runLater(appStartLatch::countDown));
    stage.show();
    appStage = stage;
  }
}
```

1.  由于 JavaFX UI 将在不同的 JavaFX 应用程序线程中启动，并且在执行与 UI 交互的命令之前，UI 渲染完全需要一些延迟，我们将使用`java.util.concurrent.CountDownLatch`来指示不同的事件。为了使用`CountDownLatch`，我们在`RobotAPIDemo`类中创建了一个简单的静态辅助方法，定义如下：

```java
public static void waitForOperation(
    CountDownLatch latchToWaitFor, 
    int seconds, String errorMsg) {
  try {
    if (!latchToWaitFor.await(seconds, 
         TimeUnit.SECONDS)) {
      System.out.println(errorMsg);
    }
  } catch (Exception ex) {
    ex.printStackTrace();
  }
}
```

1.  `typeName()`方法是一个辅助方法，用于在文本字段中输入人的姓名：

```java
public static void typeName(){
  Platform.runLater(() -> {
    Bounds textBoxBounds = nameField.localToScreen(
      nameField.getBoundsInLocal());
    robot.mouseMove(textBoxBounds.getMinX(), 
      textBoxBounds.getMinY());
    robot.mouseClick(MouseButton.PRIMARY);
    robot.keyType(KeyCode.CAPS);
    robot.keyType(KeyCode.S);
    robot.keyType(KeyCode.CAPS);
    robot.keyType(KeyCode.A);
    robot.keyType(KeyCode.N);
    robot.keyType(KeyCode.A);
    robot.keyType(KeyCode.U);
    robot.keyType(KeyCode.L);
    robot.keyType(KeyCode.L);
    robot.keyType(KeyCode.A);
  });
}
```

1.  `clickButton()`方法是一个辅助方法；它点击正确的按钮来触发问候消息的显示：

```java
public static void clickButton(){
  Platform.runLater(() -> {
    //click the button
    Bounds greetBtnBounds = greeting
      .localToScreen(greeting.getBoundsInLocal());

    robot.mouseMove(greetBtnBounds.getCenterX(), 
      greetBtnBounds.getCenterY());
    robot.mouseClick(MouseButton.PRIMARY);
  });
}
```

1.  `captureScreen()`方法是一个辅助方法，用于对应用程序进行截屏并将其保存到文件系统：

```java
public static void captureScreen(){
  Platform.runLater(() -> {
    try{

      WritableImage screenCapture = 
        new WritableImage(
          Double.valueOf(appStage.getWidth()).intValue(), 
          Double.valueOf(appStage.getHeight()).intValue()
        );

      robot.getScreenCapture(screenCapture, 
        appStage.getX(), appStage.getY(), 
        appStage.getWidth(), appStage.getHeight());

      BufferedImage screenCaptureBI = 
        SwingFXUtils.fromFXImage(screenCapture, null);
      String timePart = LocalDateTime.now()
        .format(DateTimeFormatter.ofPattern("yyyy-dd-M-m-H-ss"));
      ImageIO.write(screenCaptureBI, "png", 
        new File("screenCapture-" + timePart +".png"));
      Platform.exit();
    }catch(Exception ex){
      ex.printStackTrace();
    }
  });
}
```

1.  我们将在`main()`方法中绑定 UI 的启动和创建的辅助方法，如下所示：

```java
public static void main(String[] args) 
  throws Exception{
  new Thread(() -> Application.launch(
    RobotApplication.class, args)).start();

  waitForOperation(appStartLatch, 10,
    "Timed out waiting for JavaFX Application to Start");
  typeName();
  clickButton();
  waitForOperation(btnActionLatch, 10, 
    "Timed out waiting for Button to complete operation");
  Thread.sleep(1000);
  captureScreen();
}
```

完整的代码可以在`Chapter16/9_robot_api`中找到。您可以通过使用`run.bat`或`run.sh`来运行示例。运行应用程序将启动 UI，执行操作，截取屏幕并退出应用程序。截图将放在启动应用程序的文件夹中，并且将遵循命名约定——`screenCapture-yyyy-dd-M-m-H-ss.png`。这是一个示例截图：

![](img/2e02b304-5a14-4fe8-805f-d3d9265cada7.png)

# 工作原理...

由于 JavaFX 应用程序在不同的线程中运行，我们需要确保 Robot API 的操作被正确排序，并且只有在完整的 UI 显示后才执行 Robot API 的操作。为了确保这一点，我们使用了`java.util.concurrent.CountDownLatch`来通信以下事件：

+   完成 UI 的加载

+   完成为按钮定义的操作的执行

通过使用`CountDownLatch`来通信 UI 加载的完成，如下所示：

```java
# Declaration of the latch
static public CountDownLatch appStartLatch = new CountDownLatch(1);

# Using the latch 
stage.addEventHandler(WindowEvent.WINDOW_SHOWN, e ->
                Platform.runLater(appStartLatch::countDown));
```

当窗口显示时，`countDown()`方法在`Stage`事件处理程序中被调用，从而释放锁并触发主方法中以下代码块的执行：

```java
typeName();
clickButton();
```

然后主线程再次被阻塞，等待`btnActionLatch`被释放。在按钮问候中的操作完成后，`btnActionLatch`被释放。一旦`btnActionLatch`被释放，主线程继续执行以调用`captureScreen()`方法。

让我们讨论一些我们从`javafx.scene.robot.Robot`类中使用的方法：

`mouseMove()`：此方法用于将鼠标光标移动到从其*x*和*y*坐标标识的给定位置。我们使用以下代码行来获取组件的边界：

```java
Bounds textBoxBounds = nameField.localToScreen(nameField.getBoundsInLocal());
```

组件的边界包括以下内容：

+   左上角的*x*和*y*坐标

+   右下角的*x*和*y*坐标

+   组件的宽度和高度

因此，对于我们的 Robot API 用例，我们使用左上角的*x*和*y*坐标，如下所示：

```java
robot.mouseMove(textBoxBounds.getMinX(), textBoxBounds.getMinY());
```

`mouseClick()`：此方法用于单击鼠标上的按钮。鼠标按钮由`javafx.scene.input.MouseButton`枚举中的以下`enums`标识：

+   `PRIMARY`：代表鼠标的左键单击

+   `SECONDARY`：代表鼠标的右键单击

+   `MIDDLE`：代表鼠标的滚动或中间按钮。

因此，为了能够使用`mouseClick()`，我们需要移动需要执行单击操作的组件的位置。在我们的情况下，如在`typeName()`方法的实现中所示，我们使用`mouseMove()`移动到文本字段的位置，然后调用`mouseClick()`，如下所示：

```java
robot.mouseMove(textBoxBounds.getMinX(), 
    textBoxBounds.getMinY());
robot.mouseClick(MouseButton.PRIMARY);
```

`keyType()`: 该方法用于向接受文本输入的组件中输入字符。要输入的字符由`javafx.scene.input.KeyCode`枚举中的枚举表示。在我们的`typeName()`方法实现中，我们输入字符串`Sanaulla`，如下所示：

```java
robot.keyType(KeyCode.CAPS);
robot.keyType(KeyCode.S);
robot.keyType(KeyCode.CAPS);
robot.keyType(KeyCode.A);
robot.keyType(KeyCode.N);
robot.keyType(KeyCode.A);
robot.keyType(KeyCode.U);
robot.keyType(KeyCode.L);
robot.keyType(KeyCode.L);
robot.keyType(KeyCode.A);
```

`getScreenCapture()`: 该方法用于对应用程序进行截屏。捕获截屏的区域由传递给该方法的*x*和*y*坐标以及宽度和高度信息确定。然后将捕获的图像转换为`java.awt.image.BufferedImage`并保存到文件系统中，如下面的代码所示：

```java
WritableImage screenCapture = new WritableImage(
    Double.valueOf(appStage.getWidth()).intValue(), 
    Double.valueOf(appStage.getHeight()).intValue()
  );
robot.getScreenCapture(screenCapture, 
  appStage.getX(), appStage.getY(), 
  appStage.getWidth(), appStage.getHeight());

BufferedImage screenCaptureBI = 
  SwingFXUtils.fromFXImage(screenCapture, null);
String timePart = LocalDateTime.now().format(
  DateTimeFormatter.ofPattern("yyyy-dd-M-m-H-ss"));
ImageIO.write(screenCaptureBI, "png", 
  new File("screenCapture-" + timePart +".png"));
```
