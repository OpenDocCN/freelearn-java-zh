# 第七章.图形用户界面改进

在本章中，我们将涵盖以下内容：

+   混合重量级和轻量级组件

+   管理窗口类型

+   管理窗口的不透明度

+   创建变化的渐变半透明窗口

+   管理窗口的形状

+   在 Java 7 中使用新的边框类型

+   在 FileDialog 类中处理多个文件选择

+   控制打印对话框类型

+   使用新的 JLayer 装饰器为密码字段

# 介绍

在 Java 7 中增强了开发具有**图形用户界面**（**GUI**）界面的应用程序的能力。其中一些是较小的改进，并在本介绍中进行了讨论。其他的，如使用`javax.swing.JLayer`装饰器类，更为复杂，分别在单独的配方中进行了讨论。

现在可以在应用程序中混合重量级和轻量级组件，而无需添加特殊代码来使其按预期工作。这一改进对 Java 7 的用户来说基本上是透明的。然而，这种方法的本质以及可能由于它们的使用而出现的特殊情况在*混合重量级和轻量级组件*配方中有详细介绍。

为了简化应用程序的开发，引入了三种基本的窗口类型。这些应该简化某些类型应用程序的创建，并在“管理窗口类型”配方中进行了讨论。

应用程序的整体外观可能包括其不透明度和形状等特征。*管理窗口的不透明度*配方说明了如何控制窗口的不透明度，*创建变化的渐变半透明窗口*配方则探讨了为这样的窗口创建渐变。详细介绍了控制窗口的形状，例如使其圆形或某种不规则形状，*管理窗口的形状*配方中有详细说明。

与**Java 6 Update 10**发布一起，透明度相关的功能最初是作为私有的`com.sun.awt.AWTUtilities`类的一部分添加的。然而，这些功能已经移动到了`java.awt`包中。

`Javax.swing.JComponents`具有可以控制外观的边框。在 Java 7 中，添加了几种新的边框。这些在*在 Java 7 中使用新的边框类型*配方中有详细说明。

文件对话框和打印对话框的使用也进行了改进。这些增强功能分别在*处理文件对话框类中的多个文件选择*和*控制打印对话框类型*配方中进行了讨论。

现在可以在`JComponent`上绘制。这允许使用特殊效果，这在早期版本的 Java 中并不容易实现。*使用新的 JLayer 装饰器为密码字段*配方说明了这个过程，并演示了如何为窗口创建水印。

本章的所有配方都使用了基于`JFrame`的应用程序。以下是用于开发基于最小窗口的应用程序的代码，这些代码是配方示例的基础。使用`ApplicationDriver`类来启动和显示`JFrame`派生的`ApplicationWindow`类。`ApplicationDriver`类如下所示：

```java
public class ApplicationDriver {
public static void main(String[] args) {
SwingUtilities.invokeLater(new Runnable() {
@Override
public void run() {
ApplicationWindow window = new ApplicationWindow();
window.setVisible(true);
}
});
}
}

```

`invokeLater`方法使用内部类来创建并显示`ApplicationWindow`。这个窗口在其构造函数中设置。这是一个简单的窗口，有一个**退出**按钮，我们将在后续的配方中用来关闭应用程序并进行增强：

```java
public class ApplicationWindow extends JFrame {
public ApplicationWindow() {
this.setTitle("Example");
this.setSize(200, 100);
this.setLocationRelativeTo(null);
this.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
JButton exitButton = new JButton("Exit");
exitButton.addActionListener(new ActionListener() {
public void actionPerformed(ActionEvent event) {
System.exit(0);
}
});
this.add(exitButton);
}
}

```

当执行此代码时，输出应该如下截图所示：

![Introduction](img/5627_07_01.jpg)

在 Java 7 中引入了一些较小的改进。例如，受保护的静态`java.awt.Cursor`数组已被弃用。而是使用`getPredefinedCursor`方法。此方法接受一个整数参数并返回一个`Cursor`对象。

`java.swing.JColorChooser`对话框引入了一个新的**HSV**选项卡。如下截图所示：

![Introduction](img/5627_07_02.jpg)

在 Java 7 中，可以自定义拖动的 JApplet 的标题，并指定是否应该装饰。这是通过`script`标签来实现的：

```java
<script src="img/javascript source file"></script>
<script>
var attributes = { code:'AppletName', width:100, height:100 };
var parameters = {jnlp_href: 'appletname.jnlp',
java_decorated_frame: 'true',
java_applet_title: 'A Custom Title'
};
deployJava.runApplet(attributes, parameters, '7'7);
</script>

```

`java_decorated_frame`参数设置为`true`，以指定窗口应该装饰。使用`java_applet_title`参数指定窗口的标题。

此示例改编自[`download.oracle.com/javase/tutorial/deployment/applet/draggableApplet.html`](http://download.oracle.com/javase/tutorial/deployment/applet/draggableApplet.html)。可以在该网站上找到有关如何创建可拖动小程序的更多详细信息。

还需要注意一些杂项更改。**Nimbus 外观**已从`com.sun.java.swing`包移动到`javax.swing`包。`isValidateRoot`方法已添加到`Applet`类中，以指示容器是有效的根。最后，基于**X11 XRender**扩展的新**Java2D**图形管道已添加，以提供更好的访问**图形处理单元**（**GPU**）。

# 混合重量级和轻量级组件

Java 提供了两种基本的组件集用于开发 GUI 应用程序：**抽象窗口工具包**（**AWT**）和**Swing**。 AWT 依赖于本地系统的底层代码，因此这些组件被称为重量级组件。另一方面，Swing 组件完全独立于本地系统运行，完全由 Java 代码实现，因此被称为轻量级组件。在以前的 Java 版本中，混合重量级和轻量级组件是低效且麻烦的。在`Java 6 Update 12`中，并持续到 Java 7，JVM 处理了重量级和轻量级组件的混合。

## 准备工作

如果您正在使用同时实现重量级和轻量级组件的代码，无需对代码进行任何更改，因为 Java 7 会自动处理这些组件。我们将修改本章开头的代码来演示这一点：

1.  使用介绍部分的代码示例创建一个新的应用程序。

1.  修改代码以使用重量级和轻量级示例。

1.  使用旧版本的 Java 运行应用程序，然后再次使用 Java 7 运行。

## 如何做...

1.  按照本章的介绍指定创建一个新的窗口应用程序。将以下代码段添加到`ApplicationWindow`构造函数中：

```java
JMenuBar menuBar = new JMenuBar();
JMenu menu = new JMenu("Overlapping Menu");
JMenuItem menuItem = new JMenuItem("Overlapping Item");
menu.add(menuItem);
menuBar.add(menu);
this.setJMenuBar(menuBar);
this.validate();

```

1.  接下来，修改**Exit**按钮的声明，使其现在使用重量级的`Button`而不是轻量级的`JButton`，如下所示：

```java
Button exitButton = new Button("Exit");

```

1.  执行应用程序。您需要使用**Java 6 Build 10**之前的版本运行应用程序，否则重叠问题将不会显示。当窗口打开时，点击菜单，注意虽然菜单项重叠了**Exit**按钮，但按钮显示出来并覆盖了菜单文本。以下是重叠的示例：![如何做...](img/5627_07_03.jpg)

1.  现在，使用 Java 7 再次运行应用程序。当您这次点击菜单时，您应该注意到重叠问题已经解决，如下面的截图所示：

![如何做...](img/5627_07_04.jpg)

## 它是如何工作的...

JVM 会自动处理组件的混合。在这个例子中，我们创建了一个场景来说明重叠问题，然后展示了如何在最新的 Java 版本中解决了这个问题。然而，调用顶层框架的`validate`方法以确保所有形状正确重绘是一个好的做法。以前用于混合组件的解决方法也可能需要被移除。

## 还有更多...

在使用 Java 7 时，有一些特定的地方需要考虑，当使用混合组件时。

+   高级 Swing 事件可能无法正常工作，特别是由`javax.swing.InputMap`维护的事件。

+   不支持部分透明的轻量级组件，这些组件旨在允许重量级组件透过它们看到。重量级项目将不会显示在半透明像素下面。

+   重量级组件必须作为框架或小程序的一部分创建。

+   如果在您的应用程序中已经处理了重量级和轻量级组件的混合，并且 Java 7 的新增功能引起了问题，您可以使用私有的`sun.awt.disableMixing`系统属性来关闭混合支持。

# 管理窗口类型

`JFrame`类支持`setType`方法，该方法将窗口的一般外观配置为三种类型之一。这可以简化窗口外观的设置。在本教程中，我们将研究这些类型及其在 Windows 和 Linux 平台上的外观。

## 准备工作

要设置窗口类型，使用`setType`方法，其中包括`java.awt.Window`类中的三种窗口类型之一：

+   `Type.NORMAL:` 这代表一个正常的窗口，是窗口的默认值

+   `Type.POPUP:` 这是一个临时窗口，用于小区域，如工具提示

+   `Type.UTILITY:` 这也是一个用于对象的小窗口，例如调色板

## 如何做...

1.  按照本章介绍中的说明创建一个新的窗口应用程序。在**退出**按钮创建之前添加以下语句：

```java
this.setType(Type.POPUP);

```

1.  执行应用程序。在 Windows 系统上，窗口应如下所示：

![如何做...](img/5627_07_05.jpg)

## 它是如何工作的...

该方法的使用相当简单。`Type`枚举可以在`java.awt`包中找到。在 Windows 上，窗口显示如下截图所示。正常和弹出样式具有相同的外观。实用程序类型缺少最小化和最大化按钮：

以下截图显示了`Type.NORMAL`窗口类型的示例：

![它是如何工作的...](img/5627_07_06.jpg)

以下截图显示了`Type.POPUP`窗口类型的示例：

![它是如何工作的...](img/5627_07_07.jpg)

以下截图显示了`Type.UTILITY`窗口类型的示例：

![它是如何工作的...](img/5627_07_08.jpg)

在 Ubuntu 上，窗口显示如下截图所示。正常和实用程序具有相同的外观，而弹出类型缺少其按钮：

以下截图显示了`Type.NORMAL`窗口类型的示例：

![它是如何工作的...](img/5627_07_09.jpg)

以下截图显示了`Type.POPUP`窗口类型的示例：

![它是如何工作的...](img/5627_07_10.jpg)

以下截图显示了`Type.UTILITY`窗口类型的示例：

![它是如何工作的...](img/5627_07_11.jpg)

# 管理窗口的不透明度

窗口的不透明度指的是窗口的透明程度。当窗口完全不透明时，屏幕上窗口后面的东西是看不见的。部分不透明的窗口允许背景透过。在本教程中，我们将学习如何控制窗口的不透明度。

## 准备工作

要控制窗口的不透明度，使用`JFrame`类的`setOpacity`方法，使用表示窗口应该有多不透明的浮点值。

## 如何做...

1.  按照本章介绍中的说明创建一个新的标准 GUI 应用程序。将`invokeLater`方法调用替换为以下代码：

```java
JFrame.setDefaultLookAndFeelDecorated(true);
SwingUtilities.invokeLater(new Runnable() {
@Override
public void run() {
ApplicationWindow window = new ApplicationWindow();
window.setOpacity(0.75f);
window.setVisible(true);
}
});

```

1.  执行应用程序。窗口应如下所示：

![如何做...](img/5627_07_12.jpg)

注意这个应用程序后面的窗口是可以看到的。在这种情况下，背景是应用程序的代码。

## 它是如何工作的...

`setOpacity`使用`0.75f`来设置窗口的不透明度。这导致它变得 75％透明，可以通过代码看到。

不透明度的值范围是 0.0f 到 1.0f。值为 1.0f 表示完全不透明的窗口，值为 0.0f 表示完全透明的窗口。如果不透明度设置为 0.0f，则可能启用或禁用鼠标。这由底层系统决定。要设置小于 1.0f 的值：

+   必须支持透明度

+   窗口必须是无装饰的

+   窗口不能处于全屏模式

下一节将介绍如何确定是否支持透明度。`getOpacity`方法可用于确定当前不透明度级别。

## 还有更多...

要确定平台是否支持不透明度，我们需要使用`java.awt.GraphicsDevice`类的一个实例。`java.awt.GraphicsEnvironment`类包含当前平台的`GraphicsDevice`对象列表。`GraphicsDevice`通常指的是可用的屏幕，但也可以包括打印机或图像缓冲区。每个`GraphicsDevice`还可以包含一组`GraphicsConfiguration`对象，用于指定设备可能的配置，例如分辨率和支持的颜色模型。

在以下代码序列中，我们获取`GraphicsEnvironment`对象的一个实例，然后使用它的`getDefaultScreenDevice`方法获取一个`GraphicsDevice`对象。使用`isWindowTranslucencySupported`方法针对`GraphicsDevice`对象来确定是否支持透明度：

```java
GraphicsEnvironment graphicsEnvironment =
GraphicsEnvironment.getLocalGraphicsEnvironment();
GraphicsDevice graphicsDevice = graphicsEnvironment.getDefaultScreenDevice();
if (!graphicsDevice.isWindowTranslucencySupported(
GraphicsDevice.WindowTranslucency.TRANSLUCENT)) {
System.err.println(
"Translucency is not supported on this platform");
System.exit(0);
}

```

`GraphicsDevice.WindowTranslucency`枚举表示平台可能支持的透明度类型。其值总结在以下表中。alpha 值指的是透明度级别：

| 值 | 意义 |
| --- | --- |
| `PERPIXEL_TRANSLUCENT` | 表示系统支持一些像素设置为可能不同的 alpha 值 |
| `PERPIXEL_TRANSPARENT` | 表示系统支持所有像素设置为 0.0f 或 1.0f |
| `TRANSLUCENT` | 表示系统支持所有像素设置为 alpha 值 |

## 另请参阅

*使用新的 JLayer 装饰器为密码字段*配方解决了如何在`JComponent`上绘制。

# 创建一个变化的渐变半透明窗口

有时，通过添加特殊的图形特性，应用程序窗口可以在美学上得到增强。Java 7 支持使用渐变半透明窗口，透明度既可以在视觉上有趣，也可以在功能上有用。

这个配方将演示如何在窗口上同时使用透明度特性和颜色渐变。

## 准备工作

为了创建一个半透明的渐变颜色窗口，您需要：

1.  执行检查以确保系统环境支持每像素半透明。

1.  设置背景颜色，使窗口最初完全透明。

1.  创建一个`java.awt.GradientPaint`对象来指定渐变的颜色和位置。

## 如何做...

1.  按照本章介绍中的描述创建一个新的标准 GUI 应用程序。在线程开始之前，将以下代码添加到`ApplicationDriver`类中：

```java
GraphicsEnvironment envmt =
GraphicsEnvironment.getLocalGraphicsEnvironment();
GraphicsDevice device = envmt.getDefaultScreenDevice();
if (!device.isWindowTranslucencySupported (WindowTranslucency.PERPIXEL_TRANSLUCENT)) {
System.out.println("Translucent windows are not supported on your system.");
System.exit(0);
}
JFrame.setDefaultLookAndFeelDecorated(true);

```

1.  接下来，用以下代码序列替换`ApplicationWindow`构造函数的主体：

```java
this.setTitle("Gradient Translucent Window");
setBackground(new Color(0, 0, 0, 0));
this.setSize(500, 700);
this.setLocationRelativeTo(null);
this.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
JPanel panel = new JPanel() {
@Override
protected void paintComponent(Graphics gradient) {
if (gradient instanceof Graphics2D) {
final int Red = 120;
final int Green = 50;
final int Blue = 150;
Paint paint = new GradientPaint(0.0f, 0.0f,
new Color(Red, Green, Blue, 0),
getWidth(), getHeight(),
new Color(Red, Green, Blue, 255));
Graphics2D gradient2d = (Graphics2D) gradient;
gradient2d.setPaint(paint);
gradient2d.fillRect(0, 0, getWidth(), getHeight());
}
}
};
this.setContentPane(panel);
this.setLayout(new FlowLayout());
JButton exitButton = new JButton("Exit");
this.add(exitButton);
exitButton.addActionListener(new ActionListener() {
public void actionPerformed(ActionEvent event) {
System.exit(0);
}
});

```

1.  执行应用程序。您的窗口应该类似于以下内容：

![如何做...](img/5627_07_13.jpg)

## 它是如何工作的...

首先，我们在`ApplicationDriver`类中添加了代码，以测试系统是否支持每像素半透明。在我们的示例中，如果不支持，应用程序将退出。这在*还有更多...*部分的*管理窗口的不透明度*配方中有更详细的讨论。

不应在装饰窗口上使用渐变。我们调用`setDefaultLookAndFeelDecorated`方法来确保使用默认外观。在 Windows 7 上执行时，这会导致一个无装饰的窗口。

在`ApplicationDriver`类中，我们首先设置了窗口的背景颜色。我们使用`(0, 0, 0, 0)`来指定每种颜色的饱和度级别，红色、绿色和蓝色的 alpha 值都为零。颜色值可以是 0 到 255 之间的任何整数，但我们希望我们的窗口起始时没有颜色。alpha 值为零意味着我们的窗口将完全透明。

接下来，我们创建了一个新的`JPanel`。在`JPanel`中，我们重写了`paintComponent`方法，并创建了一个新的`GradientPaint`对象。`GradientPaint`类有四个构造函数。我们选择使用需要浮点数的 X 和 Y 坐标以及`Color`对象来指定渐变颜色的构造函数。您还可以选择传递`Point2D`对象而不是浮点数。

首先指定的点，可以是浮点数，也可以是`Point2D`对象，表示渐变的起点。在我们的示例中，第二个点由`getWidth`和`getHeight`方法确定，确定了渐变的终点。在我们的示例中，结果是一个从左上角开始浅色，随着向下和向右移动逐渐变暗的渐变。

最后，我们将渐变强制转换为`Graphics2D`对象，并调用`setPaint`和`fillRect`方法在整个窗口上绘制我们的渐变。

## 另请参阅

使用`GraphicsDevice`对象来确定透明度支持级别的讨论在*还有更多..*部分的*管理窗口的不透明度*配方中有更详细的介绍。

# 管理窗口的形状

在应用程序开发中，有时创建特殊形状的窗口可能很有趣和有用。从 Java 7 版本开始，这个功能现在已经可用。在这个配方中，我们将开发一个停止标志形状的窗口，以确保用户想要继续某些操作。

## 准备工作

要创建一个特殊形状的窗口，您必须：

1.  验证给定系统是否支持逐像素透明度。

1.  创建一个组件监听器来捕获`componentResized`事件。

1.  创建一个形状的实例并将其传递给`setShape`方法。

## 如何做...

1.  按照本章介绍中描述的方式创建一个新的标准 GUI 应用程序。在`main`方法中，在启动线程之前，通过添加以下代码来测试系统是否支持有形窗口：

```java
GraphicsEnvironment envmt =
GraphicsEnvironment.getLocalGraphicsEnvironment();
GraphicsDevice device = envmt.getDefaultScreenDevice();
if (!device.isWindowTranslucencySupported(
WindowTranslucency.PERPIXEL_TRANSLUCENT)) {
System.out.println("Shaped windows not supported");
System.exit(0);
}

```

1.  创建一个名为`StopPanel`的新类，它是从`JPanel`派生的，并向其添加以下构造函数：

```java
public StopPanel() {
this.setBackground(Color.red);
this.setForeground(Color.red);
this.setLayout(null);
JButton okButton = new JButton("YES");
JButton cancelButton = new JButton("NO");
okButton.setBounds(90, 225, 65, 50);
cancelButton.setBounds(150, 225, 65, 50);
okButton.addActionListener(new ActionListener() {
public void actionPerformed(ActionEvent event) {
System.exit(0);
}
});
cancelButton.addActionListener(new ActionListener() {
public void actionPerformed(ActionEvent event) {
System.exit(0);
}
});
this.add(okButton);
this.add(cancelButton);
}

```

1.  您还需要为`StopPanel`类实现一个`paintComponent`方法。它负责在我们的窗口中显示文本。以下是实现此方法的一种方式：

```java
@Override
public void paintComponent(Graphics g) {
super.paintComponent(g);
Graphics2D g2d = (Graphics2D) g;
int pageHeight = this.getHeight();
int pageWidth = this.getWidth();
int bigHeight = (pageHeight+80)/2;
int bigWidth = (pageWidth-305)/2;
int smallHeight = (pageHeight+125)/2;
int smallWidth = (pageWidth-225)/2;
Font bigFont = new Font("Castellar", Font.BOLD, 112);
Font smallFont = new Font("Castellar", Font.PLAIN, 14);
g2d.setFont(bigFont);
g2d.setColor(Color.white);
g2d.drawString("STOP", bigWidth, bigHeight);
g2d.setFont(smallFont);
g2d.drawString("Are you sure you want to continue?", smallWidth, smallHeight);
}

```

1.  在`ApplicationWindow`类中，在创建**Exit**按钮之前，创建一个`StopPanel`的新实例。接下来，创建一个`Shape`的新实例。在我们的示例中，我们使用`getPolygon`方法创建了一个`Polygon`对象，如下所示：

```java
this.add(new StopPanel());
final Polygon myShape = getPolygon();

```

1.  然后在创建**Exit**按钮的代码前添加一个`componentListener`来捕获`componentResized`事件。在监听器中，对`Shape`对象调用`setShape`方法。我们还将在这一点上设置前景色和背景色：

```java
this.addComponentListener(new ComponentAdapter() {
@Override
public void componentResized(ComponentEvent e) {
setShape(myShape);
((JFrame) e.getSource()).setForeground(Color.red);
((JFrame) e.getSource()).setBackground(Color.red);
}
});

```

1.  添加一个调用`setUndecorated`方法并将属性设置为`true:`

```java
setUndecorated(true);

```

1.  接下来，将`getPolygon`方法添加到类中。该方法使用两个整数数组和`Polygon`类的`addPoint`方法创建一个八边形：

```java
private Polygon getPolygon() {
int x1Points[] = {0, 0, 100, 200, 300, 300, 200, 100};
int y1Points[] = {100, 200, 300, 300, 200, 100, 0, 0};
Polygon polygon = new Polygon();
for (int i = 0; i < y1Points.length; i++) {
polygon.addPoint(x1Points[i], y1Points[i]);
}
return polygon;
}

```

1.  当应用程序执行时，您应该看到一个八边形窗口，格式如下：

![如何做...](img/5627_07_14.jpg)

## 它是如何工作的...

我们最初的测试验证了逐像素的透明度，使我们能够根据系统的需求定制应用程序。在我们的例子中，如果该属性不受支持，我们只是退出应用程序，尽管在现实世界的环境中，您可能希望打开一个不太复杂的窗口。在*更多内容*部分的*管理窗口的不透明度*配方中更详细地讨论了检测操作系统支持的内容。

`StopPanel`类实现了`JPanel`接口，并允许我们在窗口中添加自定义文本和按钮。因为我们在窗口中使用了特殊的形状，所以我们选择使用`null`参数调用`setLayout`方法，这样就可以使用`setBounds`方法来明确地放置我们想要的按钮在窗口上。重要的是要注意，虽然窗口显示为八边形，或者您选择的其他形状，但实际上窗口仍然是一个矩形，由`setSize`方法指定。因此，按钮和其他对象可以放置在窗口上，但如果它们超出了您的形状设置的边界，它们就不可见了。

`paintComponent`方法用于自定义窗口上的文本。在这个方法中，我们设置了文本的大小、样式和位置，并调用`drawString`方法将其实际绘制到屏幕上。

要实际创建一个八边形窗口，我们创建了我们的`getPolygon`方法并手动绘制了多边形。然而，如果您想要使用一个已经由实现`Shape`接口的类定义形状的窗口，您就不需要创建一个单独的方法。您只需将`Shape`对象传递给`setShape`方法。如果`setShape`方法的参数是`null`，窗口将调整为给定系统的默认大小，通常是一个矩形。

在`componentResized`事件中执行`setShape`方法非常重要。这确保了每次窗口被重绘时，`setShape`方法都会被调用并且形状会被保持。调用`setUndecorated`方法也很重要，因为目前，特殊形状的窗口会丢失装饰。此外，窗口可能不是全屏模式。

## 另请参阅

使用`GraphicsDevice`对象来确定透明度支持的级别在*更多内容*部分的*管理窗口的不透明度*配方中有更详细的讨论。

# 在 Java 7 中使用新的边框类型

边框用于 swing 组件的轮廓。在 Java 7 中，有几种新的边框选项可用。在这个配方中，我们将开发一个简单的应用程序来演示如何创建边框以及这些边框的外观。

## 准备工作

创建和使用边框：

1.  使用`javax.swing.BorderFactory`方法创建一个新的边框。

1.  将边框对象作为`setBorder`方法的参数应用于`JComponent`对象。

## 如何做...

1.  按照本章介绍中的描述创建一个新的标准 GUI 应用程序。修改`ApplicationWindow`类以替换以下行：

```java
JButton exitButton = new JButton("Exit");
this.add(exitButton);

```

1.  使用以下代码：

```java
JPanel panel = new JPanel();
panel.setBorder(BorderFactory.createRaisedSoftBevelBorder());
this.setLayout(new FlowLayout());
JButton exitButton = new JButton("Exit");
panel.add(exitButton);
this.add(panel);

```

1.  执行应用程序。窗口应该如下所示：![如何做...](img/5627_07_15.jpg)

## 它是如何工作的...

`setBorder`方法将`JPanel`的边框更改为凸起的软斜角边框。`BorderFactory`方法具有许多静态方法来创建边框。下表总结了 Java 7 中可用的新边框：

| 方法 | 视觉效果 |
| --- | --- |
| 默认边框 | ![它是如何工作的...](img/5627_07_16.jpg) |
| `createRaisedSoftBevelBorder()` | ![它是如何工作的...](img/5627_07_17.jpg) |
| `createLineBorder(Color.BLACK, 1, true)`第一个参数是边框的颜色。第二个是它的厚度，而第三个参数指定边角是否应该是圆角的。 | ![它是如何工作的...](img/5627_07_18.jpg) |
| `createLoweredSoftBevelBorder()` | ![它是如何工作的...](img/5627_07_19.jpg) |
| `createSoftBevelBorder(BevelBorder.LOWERED)`这与`createLoweredSoftBevelBorder()`具有相同的效果。 | ![工作原理...](img/5627_07_20.jpg) |
| `createSoftBevelBorder(BevelBorder.RAISED)`这与`createRaisedSoftBevelBorder()`具有相同的效果。 | ![工作原理...](img/5627_07_21.jpg) |
| `createSoftBevelBorder(BevelBorder.LOWERED, Color.lightGray, Color.yellow)`第一个参数是边框的类型：`RAISED`或`LOWERED`。第二个参数是外部突出区域的颜色。第三个参数是内边缘的颜色。 | ![工作原理...](img/5627_07_22.jpg) |
| `createSoftBevelBorder(BevelBorder.RAISED,Color.lightGray, Color.yellow)`与`createSoftBevelBorder`相同的参数。 | ![工作原理...](img/5627_07_23.jpg) |
| `createSoftBevelBorder(BevelBorder.LOWERED, Color.lightGray, Color.lightGray, Color.white, Color.orange)`这些参数用于边框的高亮和阴影区域的内部和外部边缘。 | ![工作原理...](img/5627_07_24.jpg) |
| `createStrokeBorder(new BasicStroke(1.0f))`第二个重载的方法将`Paint`对象作为第二个参数，并用于生成颜色。 | ![工作原理...](img/5627_07_25.jpg) |
| `createDashedBorder(Color.red)` | ![工作原理...](img/5627_07_26.jpg) |
| `createDashedBorder(Color.red, 4.0f, 1.0f)`第二个参数是虚线的相对长度，第三个参数是空格的相对长度。 | ![工作原理...](img/5627_07_27.jpg) |
| `createDashedBorder(Color.red, 2.0f, 10.0f, 1.0f, true)`第二个参数指定线条的厚度。第三和第四个参数分别指定长度和间距，而最后的布尔参数确定端点是否是圆形的。 | ![工作原理...](img/5627_07_28.jpg) |

边框可以更改为任何`JComponent`类。然而，外观并不总是令人满意。就像我们在这个例子中所做的那样，有时最好在一个封闭的`JPanel`对象上更改边框。

# 在 FileDialog 类中处理多个文件选择

使用*Ctrl*和/或*Shift*键与鼠标结合来在文件对话框中选择两个或多个文件或目录。在 Java 7 中，文件对话框使用`java.awt.FileDialog`类的`setMultipleMode`方法启用或禁用此功能。这个简单的增强功能在这个示例中有所体现。

## 准备工作

在打印对话框中启用或禁用多个文件的选择：

1.  创建一个新的`FileDialog`对象。

1.  使用其`setMultipleMode`方法来确定其行为。

1.  显示对话框。

1.  使用返回值来确定选择了哪些文件。

## 如何操作...

1.  按照本章介绍中的描述创建一个新的标准 GUI 应用程序。修改`ApplicationWindow`类以添加一个按钮来显示文件对话框，如下面的代码所示。在匿名内部类中，我们将显示对话框：

```java
public ApplicationWindow() {
this.setTitle("Example");
this.setSize(200, 100);
this.setLocationRelativeTo(null);
this.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
this.setLayout(new FlowLayout());
final FileDialog fileDialog = new FileDialog(this, "FileDialog");
fileDialog.setMultipleMode(true);
JButton fileDialogButton = new JButton("File Dialog");
fileDialogButton.addActionListener(new ActionListener() {
public void actionPerformed(ActionEvent event) {
fileDialog.setVisible(true);
}
});
this.add(fileDialogButton);
JButton exitButton = new JButton("Exit");
exitButton.addActionListener(new ActionListener() {
public void actionPerformed(ActionEvent event) {
System.exit(0);
}
});
this.add(exitButton);
}

```

1.  执行应用程序。应用程序窗口应该如下所示：![如何操作...](img/5627_07_29.jpg)

1.  选择**文件对话框**按钮，应该出现以下对话框。转到一个目录并选择一些文件。在接下来的窗口中，已选择了`/home/music`目录的两个文件：![如何操作...](img/5627_07_30.jpg)

## 工作原理...

`fileDialog`类的`setMultipleMode`方法使用`true`参数执行。这使得可以选择多个文件。创建了一个匿名内部类来处理文件按钮事件的选择。在`actionPerformed`方法中，对话框被显示出来。

## 还有更多...

要确定选择了哪些文件，我们可以使用`fileDialog`类的`getFiles`方法。在`fileDialog`类的`setVisible`方法之后添加以下代码：

```java
File files[] = fileDialog.getFiles();
for (File file : files) {
System.out.println("File: " + file.getName());
}

```

该方法返回一个`File`对象数组。使用 for each 循环，我们可以显示每个选定文件的名称。执行应用程序并选择几个文件。所选音乐文件的输出应如下所示：

**文件：Future Setting A.mp3**

**文件：Space Machine A.mp3**

# 控制打印对话框类型

`java.awt.PrintJob`类的标准打印对话框允许使用通用和本机对话框。这提供了更好地适应平台的能力。对话框类型的规范很简单。

## 准备工作

要指定打印对话框类型并使用打印对话框，需要按照以下步骤进行：

1.  创建一个`javax.print.attribute.PrintRequestAttributeSet`对象。

1.  将所需的对话框类型分配给此对象。

1.  创建一个`PrinterJob`对象。

1.  将`PrintRequestAttributeSet`对象用作`PrinterJob`类的`printDialog`方法的参数。

## 如何做...

1.  创建一个新的标准 GUI 应用程序，如章节介绍中所述。修改`ApplicationWindow`类以添加一个按钮，显示如下所示的打印对话框。在一个匿名内部类中，我们将显示一个打印对话框：

```java
public ApplicationWindow() {
this.setTitle("Example");
this.setSize(200, 100);
this.setLocationRelativeTo(null);
this.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
this.setLayout(new FlowLayout());
JButton printDialogButton = new JButton("Print Dialog");
printDialogButton.addActionListener(new ActionListener() {
public void actionPerformed(ActionEvent event) {
final PrintRequestAttributeSet attributes = new HashPrintRequestAttributeSet();
attributes.add(DialogTypeSelection.COMMON);
PrinterJob printJob = PrinterJob.getPrinterJob();
printJob.printDialog(attributes);
}
});
this.add(printDialogButton);
JButton exitButton = new JButton("Exit");
exitButton.addActionListener(new ActionListener() {
public void actionPerformed(ActionEvent event) {
System.exit(0);
}
});
this.add(exitButton);
}

```

1.  执行应用程序并选择**打印**按钮。出现的对话框应该使用通用外观类型，如下面的屏幕截图所示：

![如何做...](img/5627_07_31.jpg)

## 它是如何工作的...

创建了一个新的**打印**按钮，允许用户显示打印对话框。在用于处理按钮动作事件的匿名内部类中，我们创建了一个基于`javax.print.attribute.HashPrintRequestAttributeSet`类的`PrintRequestAttributeSet`对象。这使我们能够向集合添加`DialogTypeSelection.NATIVE`属性。`DialogTypeSelection`类是 Java 7 中的新类，提供了两个字段：`COMMON`和`NATIVE`。

接下来，我们创建了一个`PrinterJob`对象，并对该对象执行了`printDialog`方法。然后打印对话框将显示出来。如果我们使用了`NATIVE`类型，将如下所示：

```java
attributes.add(DialogTypeSelection.NATIVE);

```

然后在 Windows 平台上，打印对话框将如下所示：

![它是如何工作的...](img/5627_07_32.jpg)

# 使用新的 JLayer 装饰器为密码字段

Java 7 支持装饰 GUI 组件，如文本框和面板。装饰是在组件顶部绘制的过程，使其具有特殊外观。例如，我们可能希望在界面上加水印，以显示它是测试版，或者可能为文本框中的错误提供图形 X 的指示，而这在其他情况下是不可能的。

`javax.swing.JLayer`类提供了一种将显示的组件、在组件上绘制额外图形以及拦截事件的方法绑定在一起的方式。事件的处理和显示被委托给一个`javax.swing.plaf.LayerUI`派生对象。当事件发生时，将执行一个处理事件的方法。当组件被绘制时，将执行`LayerUI`派生对象的`paint`方法，根据需要显示图形。

在本教程中，我们将学习 Java 如何支持此功能。在第一部分中，我们将演示如何为密码字段显示错误消息。在*还有更多..*部分，我们将展示如何为窗口创建水印。

## 准备工作

要装饰一个组件：

1.  创建要装饰的组件。

1.  创建一个实现装饰图形操作的`LayerUI`派生类。

1.  创建一个基于组件和`LayerUI`派生类的`JLayer`对象。

1.  将`JLayer`对象添加到应用程序中。

## 如何做...

1.  按照本章介绍中的描述创建一个新的标准 GUI 应用程序。使用以下`ApplicationWindow`。在它的构造函数中，我们将使用`getPanel`方法执行必要的步骤来返回我们的密码`JPanel`对象。当用户输入密码时，窗口将被装饰，显示密码太短的消息，直到至少输入六个字符：

```java
public ApplicationWindow() {
this.setTitle("Example");
this.setSize(300, 100);
this.setLocationRelativeTo(null);
this.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
LayerUI<JPanel> layerUI = new PasswordLayerUI();
JLayer<JPanel> jlayer = new JLayer<JPanel>(getPanel(), layerUI);
this.add(jlayer);
}
private JPanel getPanel() {
JPanel panel = new JPanel(new BorderLayout());
JPanel gridPanel = new JPanel(new GridLayout(1, 2));
JLabel quantityLabel = new JLabel("Password");
gridPanel.add(quantityLabel);
JPasswordField passwordField = new JPasswordField();
gridPanel.add(passwordField);
panel.add(gridPanel, BorderLayout.CENTER);
JPanel buttonPanel = new JPanel(new FlowLayout(FlowLayout.LEFT));
JButton okButton = new JButton("OK");
buttonPanel.add(okButton);
JButton cancelButton = new JButton("Cancel");
buttonPanel.add(cancelButton);
panel.add(buttonPanel, BorderLayout.SOUTH);
return panel;
}

```

1.  接下来，按照以下代码创建`PasswordLayerUI`类。`paint`方法将执行实际的装饰。其余的方法用于启用键盘事件并在发生时处理它们：

```java
class PasswordLayerUI extends LayerUI<JPanel> {
private String errorMessage = "Password too short";
@Override
public void paint(Graphics g, JComponent c) {
FontMetrics fontMetrics;
Font font;
int height;
int width;
super.paint(g, c);
Graphics2D g2d = (Graphics2D) g.create();
int componentWidth = c.getWidth();
int componentHeight = c.getHeight();
// Display error message
g2d.setFont(c.getFont());
fontMetrics = g2d.getFontMetrics(c.getFont());
height = fontMetrics.getHeight();
g2d.drawString(errorMessage,
componentWidth / 2 + 10, componentHeight / 2 + height);
g2d.dispose();
}
@Override
public void installUI(JComponent component) {
super.installUI(component);
((JLayer) component).setLayerEventMask(AWTEvent.KEY_EVENT_MASK);
}
@Override
public void uninstallUI(JComponent component) {
new JLayer decoratorusingsuper.uninstallUI(component);
((JLayer) component).setLayerEventMask(0);
}
protected void processKeyEvent(KeyEvent event, JLayer layer) {
JTextField f = (JTextField) event.getSource();
if (f.getText().length() < 6) {
errorMessage = "Password too short";
}
else {
errorMessage = "";
}
layer.repaint();
}
}

```

1.  执行应用程序。在文本框中输入一些字符。您的窗口应该看起来类似于以下内容：![操作步骤...](img/5627_07_33.jpg)

1.  输入至少六个字符。此时装饰应该消失如下：

![操作步骤...](img/5627_07_34.jpg)

## 工作原理...

在`ApplicationWindow`中，我们创建了`PasswordLayerUI`类的一个实例。我们使用这个对象以及`getPanel`方法返回的`JPanel`来创建`JLayer`对象。然后将`JLayer`对象添加到窗口中。

注意在`LayerUI`和`JLayer`对象中使用泛型。这是为了确保元素都是兼容的。我们使用`JPanel`，因为这是我们要装饰的组合组件。

`JLayer`类提供了一种将密码框、错误消息的显示和键盘事件拦截绑定在一起的方法。键盘事件的处理和错误消息的显示被委托给了`PasswordLayerUI`对象。按下键时，将执行`processKeyEvent`方法。当组件被绘制时，将执行`paint`方法，通过密码框显示错误消息。

在`PasswordLayerUI`类中，我们声明了一个私有的`String`变量来保存我们的错误消息。它被声明在这个级别，因为它在多个方法中被使用。

`paint`方法执行实际的装饰。它接收一个代表我们可以绘制的区域的`Graphics`对象，以及一个组件`JComponent`，在这种情况下是一个`JPanel`。在`paint`方法中，我们使用了组件的字体，还为错误消息创建了一个新的`font`。计算并使用了组件和错误字符串的高度和宽度来定位显示的错误字符串。

`installUI`和`uninstallUI`方法用于执行装饰所需的任何初始化。在这种情况下，它们被用来使键盘事件能够被拦截并由该类处理。`setLayerEventMask`方法与`AWTEvent.KEY_EVENT_MASK`参数一起使用，以启用键盘事件的处理。`processKeyEvent`方法执行实际的键盘事件处理。在这个方法中，密码文本字段内容的长度被用来确定要显示哪个错误消息。

## 还有更多...

这个例子可以考虑使用标签来执行。然而，这个例子旨在提供如何使用装饰的简单演示。创建其他装饰，比如水印，如果没有使用`JLayer`和`LayerUI`类，就不容易执行。

在`dispose`方法之前添加以下代码。这个序列将在窗口上添加一个水印，指示这是界面的测试版。使用`Castellar`字体提供更多的模板化文本外观。使用`Composite`对象来改变字符串的 alpha 值。这有效地控制了显示的字符串的透明度。`getComposite`方法用于获取窗口的当前复合体，然后用于确定正在使用的规则。规则以及`0.25f`的 alpha 值用于使水印淡入背景，如下所示：

```java
// Display watermark
String displayText = "Beta Version";
font = new Font("Castellar",Font.PLAIN, 16);
fontMetrics = g2d.getFontMetrics(font);
g2d.setFont(font);
width = fontMetrics.stringWidth(displayText);
height = fontMetrics.getHeight();
Composite com = g2d.getComposite();
AlphaComposite ac = AlphaComposite.getInstance(
((AlphaComposite)com).getRule(),0.25f);
g2d.setComposite(ac);
g2d.drawString(displayText,
(componentWidth - width) / 2,
(componentHeight - height) / 2);

```

当执行时，您的应用程序应该看起来类似于以下屏幕截图。请注意，水印是全大写的。这是使用`Castellar`字体的结果，这是 一种全大写字母字体，模仿了奥古斯都纪念罗马柱上使用的字母。

![更多内容...](img/5627_07_35.jpg)
