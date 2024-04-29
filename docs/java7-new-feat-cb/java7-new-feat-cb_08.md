# 第八章：处理事件

在本章中，我们将涵盖以下内容：

+   管理额外的鼠标按钮和高分辨率鼠标滚轮

+   在显示窗口时控制焦点

+   使用辅助循环模拟模态对话框

+   处理虚假线程唤醒

+   使用事件处理程序处理小程序初始化状态

# 介绍

Java 7 还增加了几个与事件相关的事件或与事件相关的事件。这包括对鼠标事件的处理，其中提供了增强的支持来检测鼠标按钮和使用高分辨率鼠标滚轮，正如我们将在*管理额外的鼠标按钮和高分辨率鼠标滚轮*示例中看到的。

当使用`setVisible`或`toFront`方法使窗口可见时，现在我们可以控制它们是否应该获得焦点。有些窗口可能是为了信息或状态而显示的，并不一定需要或有权获得焦点。如何控制这种行为在*控制 AutoRequestFocus*示例中有解释。

读者应该熟悉模态对话框的行为。基本上，模态对话框在关闭之前不会将焦点返回到主窗口。有时候，希望模仿这种行为而不使用对话框。例如，执行相对较长的计算的按钮的选择可能会受益于这种行为。*使用辅助循环模拟模态对话框*示例探讨了如何实现这一点。

虽然不常见，但在使用`wait`方法时可能会发生虚假中断。`java.awt.event.InvocationEvent`类的`isDispatched`方法可用于处理虚假中断，详细信息请参阅*处理虚假线程唤醒*示例。

小程序在与 JavaScript 代码通信方面也得到了增强。*使用事件处理程序处理小程序初始化状态*示例描述了 JavaScript 代码如何能够意识到并利用小程序加载的时间。

Java 7 中还有一些与事件相关的小改进，不值得列入示例的包括访问扩展键代码和为`JSlider`类实现`java.awt.iamg.ImageObserver`接口的可用性。

`KeyEvent`类已增加了两个新方法：`getExtendedKeyCode`和`getExtendedKeyCodeForChar`。第一个方法返回一个键的唯一整数，但与`getKeyCode`方法不同，它的值取决于键盘当前的配置。第二个方法返回给定 Unicode 字符的扩展键代码。

`imageUpdate`方法已添加到`JSlider`类中。这允许该类监视正在加载的图像的状态，尽管这种能力可能最好与从`JSlider`派生的类一起使用。

# 管理额外的鼠标按钮和高分辨率鼠标滚轮

Java 7 提供了更多处理鼠标事件的选项。`java.awt.Toolkit`类的`areExtraMouseButtonsEnabled`方法允许您确定系统是否支持标准按钮集之外的更多按钮。`java.awt.event.MouseWheelEvent`类的`getPreciseWheelRotation`方法可用于控制高分辨率鼠标滚轮的操作。在这个示例中，我们将编写一个简单的应用程序来确定启用的鼠标按钮数量并测试鼠标滚轮旋转。

## 准备工作

首先，使用第七章*图形用户界面改进*中的入门处找到的`ApplicationWindow`和`ApplicationDriver`起始类创建一个新的应用程序。

1.  实现`MouseListener`和`MouseWheelListener`接口以捕获鼠标事件。

1.  使用`areExtraMouseButtonsEnabled`和`getPreciseWheelRotation`方法来确定鼠标的具体信息。

## 如何做...

1.  首先，我们将使用以下代码示例设置关于我们正在创建的`JFrame`的基本信息：

```java
public class ApplicationWindow extends JFrame {
public ApplicationWindow() {
this.setTitle("Example");
this.setSize(200, 100);
this.setLocationRelativeTo(null);
this.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
this.setLayout(new FlowLayout());
JButton exitButton = new JButton("Exit");
this.add(exitButton);
}
}

```

1.  接下来，我们想要收集有关鼠标的一些信息。我们执行`getNumberOfButtons`方法来确定鼠标上有多少个按钮。然后我们使用`areExtraMouseButtonsEnabled`方法来确定我们鼠标上有多少个按钮可供我们使用。我们将这些信息打印到控制台上，如下所示：

```java
int totalButtons = MouseInfo.getNumberOfButtons();
System.out.println(Toolkit.getDefaultToolkit().areExtraMouseButtonsEnabled());
System.out.println("You have " + totalButtons + " total buttons");

```

1.  接下来，我们启用我们的监听器：

```java
this.addMouseListener(this);
this.addMouseWheelListener(this);
exitButton.addActionListener(new ActionListener() {
public void actionPerformed(ActionEvent event) {
System.exit(0);
}
});

```

1.  在`mousePressed`事件方法中，只需使用`getButton`方法打印出按下的按钮编号，如下所示：

```java
public void mousePressed(MouseEvent e) {
System.out.println("" + e.getButton());
}

```

1.  实现`MouseListener`接口方法的其余部分。在`mouseWheelMoved`事件方法中，使用`getPreciseWheelRotation`和`getWheelRotation`方法打印有关鼠标滚轮移动的具体信息：

```java
public void mouseWheelMoved(MouseWheelEvent e) {
System.out.println("" + e.getPreciseWheelRotation() +
" - " + e.getWheelRotation());
}

```

1.  执行应用程序。您应该看到一个类似以下的`JFrame`窗口：![操作步骤...](img/5627_08_01.jpg)

1.  当您在窗口中单击时，您将在控制台中看到不同的输出，具体取决于您的鼠标、您单击的按钮以及您移动鼠标滚轮的方向。以下是可能的输出之一：

true

您总共有 5 个按钮

1

2

3

4

5

0.75 - 0

1.0 - 1

1.0 - 1

1.1166666666666667 - 1

-1.0 - 0

-1.0 - -1

-1.2916666666666667 - -1

-1.225 - -1

## 它是如何工作的...

`getNumberOfButtons`方法返回了鼠标上的按钮总数。在先前的示例中，有五个按钮，但如果在没有鼠标的系统上执行该应用程序，它将返回`-1`。在我们的`mousePressed`方法中，我们打印了由`getButton`方法返回的点击的按钮的名称。

我们执行了`areExtraMouseButtonsEnabled`方法来确定实际上支持额外的按钮，并且允许将它们添加到`EventQueue`中。如果要更改此值，必须在`Toolkit`类初始化之前进行，如*还有更多..*部分所述。

因为启用了多个鼠标按钮，我们的输出显示了所有五个鼠标按钮的编号。在大多数情况下，鼠标滚轮也被视为按钮，并包括在计数中。

先前控制台输出的最后几行是鼠标滚轮的移动指示。第一行，**0.75 - 0**，表示鼠标滚轮向后移动，或者向用户方向移动。这是通过`getPreciseWheelRotation`方法返回值 0.75 和`getWheelRotation`方法返回值 0 来表明的。输出的最后一行，**-1.225 - -1**，相反表示鼠标滚轮向前移动，或者远离用户。这是通过两种方法的负返回值来表示的。

使用高分辨率鼠标滚轮执行了此应用程序。低分辨率鼠标滚轮将只返回整数值。

## 还有更多...

有两种控制是否启用额外鼠标按钮的方法。第一种技术是使用以下命令行启动应用程序，并将`sun.awt.enableExtraMouseButtons`属性设置为`true`或`false`：

```java
java -Dsun.awt.enableExtraMouseButtons=false ApplicationDriver

```

选项`D`使用了一个`false`值，指定不启用额外的鼠标按钮。第二种方法是在`Toolkit`类初始化之前设置相同的属性。可以使用以下代码实现：

```java
System.setProperty("sun.awt.enableExtraMouseButtons", "true");

```

# 在显示窗口时控制焦点

`setAutoRequestFocus`方法已添加到`java.awt.Window`类中，用于指定窗口在使用`setVisible`或`toFront`方法显示时是否应该接收焦点。有时候，窗口被显示出来，但我们不希望窗口获得焦点。例如，如果显示的窗口包含状态信息，使其可见就足够了。让它获得焦点可能没有意义，并且可能会让用户感到沮丧，因为他们被迫将焦点切换回原始窗口。

## 做好准备

在窗口可见时控制焦点，如果应该接收焦点则调用`setAutoRequestFocus`方法并传入`true`，否则传入`false`。

## 如何做...

1.  为了演示这种技术，我们将创建两个窗口。一个用于隐藏然后显示第二个窗口。通过在第二个窗口中使用`setAutoRequestFocus`方法，我们可以控制它是否接收焦点。

1.  首先，使用以下驱动程序创建一个新项目。在驱动程序中，我们将创建第一个窗口如下：

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

1.  接下来，添加`ApplicationWindow`类。在这个类中，我们添加了两个按钮来隐藏和显示第二个窗口，以及一个用于退出应用程序的第三个按钮，如下所示：

```java
public class ApplicationWindow extends JFrame {
private SecondWindow second;
public ApplicationWindow() {
this.setTitle("Example");
this.setBounds(100, 100, 200, 200);
this.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
this.setLayout(new FlowLayout());
second = new SecondWindow();
second.setVisible(true);
JButton secondButton = new JButton("Hide");
this.add(secondButton);
secondButton.addActionListener(new ActionListener() {
public void actionPerformed(ActionEvent event) {
second.setVisible(false);
});
JButton thirdButton = new JButton("Reveal");
this.add(thirdButton);
thirdButton.addActionListener(new ActionListener() {
public void actionPerformed(ActionEvent event) {
second.setVisible(true);
}
});
JButton exitButton = new JButton("Exit");
this.add(exitButton);
exitButton.addActionListener(new ActionListener() {
public void actionPerformed(ActionEvent event) {
System.exit(0);
}
});
}
}

```

1.  接下来添加`SecondWindow`类。这个简单的窗口除了使用`setAutoRequestFocus`方法来控制其行为外，什么也不做：

```java
public class SecondWindow extends JFrame {
public SecondWindow() {
this.setTitle("Second Window");
this.setBounds(400, 100, 200, 200);
this.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
this.setAutoRequestFocus(false);
}
}

```

1.  执行应用程序。两个窗口应该都出现，并且焦点在第一个窗口上，如下截图所示：![如何做...](img/5627_08_02.jpg)

1.  第二个窗口显示如下：![如何做...](img/5627_08_03.jpg)

1.  选择**隐藏**按钮。第二个窗口应该消失。接下来，选择**显示**按钮。第二个窗口应该重新出现，并且不应该有焦点。这是`setAutoRequestFocus`方法与`false`值一起使用时的效果。

1.  停止应用程序并将`setAutoRequestFocus`方法的参数更改为`true`。重新执行应用程序，隐藏然后显示第二个窗口。当它显示时，第二个窗口应该接收焦点。

## 工作原理...

应用程序驱动程序显示了应用程序窗口。在`ApplicationWindow`类中，创建并显示了第二个窗口。此外，创建了三个按钮和内部类来影响它们的操作。`setAutoRequestFocus`方法传递了一个`false`值，指定在窗口显示时不保留焦点。

## 还有更多...

这种方法可能对从系统托盘运行的应用程序有用。

### 注意

请注意，`isAutoRequestFocus`方法可用于确定`autoRequestFocus`值的值。

# 使用次要循环模拟模态对话框

`java.awt.EventQueue`类的`SecondaryLoop`接口提供了一种方便的技术来模拟模态对话框的行为。模态对话框有两种行为。第一种是从用户的角度来看。在对话框完成之前，用户不被允许与主窗口交互。第二个角度是从程序执行的角度来看。调用对话框的线程在对话框关闭之前被阻塞。

`SecondaryLoop`允许在阻塞当前线程的同时执行某些任务，直到次要循环完成。它可能没有与之关联的用户界面。当用户选择一个按钮时，虽然它不显示对话框，但涉及到长时间运行的计算时，这可能会很有用。在本教程中，我们将演示如何使用次要循环并检查其行为。

## 准备工作

要创建和使用次要循环，需要按照以下步骤进行：

1.  获取应用程序的默认`java.awt.Toolkit`实例。

1.  使用此方法获取系统事件队列的引用。

1.  使用事件队列创建一个`SecondaryLoop`对象。

1.  使用`SecondaryLoop`接口的`enter`方法来启动循环。

1.  在次要循环中实现所需的行为。

1.  使用`SecondaryLoop`接口的`exit`方法来终止循环。

## 如何做...

1.  使用以下`ApplicationDriver`类创建一个新的应用程序。它简单地显示应用程序的窗口如下：

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

1.  添加以下`ApplicationWindow`类。它创建了两个按钮，用于演示次要循环的行为：

```java
public class ApplicationWindow extends JFrame implements ActionListener {
private JButton firstButton;
private JButton secondButton;
public ApplicationWindow() {
this.setTitle("Example");
this.setBounds(100, 100, 200, 200);
this.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
this.setLayout(new FlowLayout());
firstButton = new JButton("First");
this.add(firstButton);
firstButton.addActionListener(this);
secondButton = new JButton("Second");
this.add(secondButton);
secondButton.addActionListener(this);
}
}

```

1.  接下来，添加以下的`actionPerformed`方法。创建一个`SecondaryLoop`对象，并根据所选的按钮创建`WorkerThread`对象如下：

```java
@Override
public void actionPerformed(ActionEvent e) {
Thread worker;
JButton button = (JButton) e.getSource();
Toolkit toolkit = Toolkit.getDefaultToolkit();
EventQueue eventQueue = toolkit.getSystemEventQueue();
SecondaryLoop secondaryLoop = eventQueue.createSecondaryLoop();
Calendar calendar = Calendar.getInstance();
String name;
if (button == firstButton) {
name = "First-"+calendar.get(Calendar.MILLISECOND);
}
else {
name = "Second-"+calendar.get(Calendar.MILLISECOND);
}
worker = new WorkerThread(secondaryLoop, name);
worker.start();
if (!secondaryLoop.enter()) {
System.out.println("Error with the secondary loop");
}
else {
System.out.println(name + " Secondary loop returned");
}
}

```

1.  添加以下的`WorkerThread`类作为内部类。它的构造函数保存了`SecondaryLoop`对象，并传递了一条消息。这条消息将被用来帮助我们解释结果。`run`方法在睡眠两秒之前显示消息：

```java
class WorkerThread extends Thread {
private String message;
private SecondaryLoop secondaryLoop;
public WorkerThread(SecondaryLoop secondaryLoop, String message) {
this.secondaryLoop = secondaryLoop;
this.message = message;
}
@Override
public void run() {
System.out.println(message + " Loop Sleeping ... ");
try {
Thread.sleep(2000);
}
catch (InterruptedException ex) {
ex.printStackTrace();
}
System.out.println(message + " Secondary loop completed with a result of " +
secondaryLoop.exit());
}
}

```

1.  执行应用程序。应该会出现以下窗口。这里已经调整了大小：![操作步骤...](img/5627_08_04.jpg)

1.  接下来，选择**First**按钮。以下控制台输出应该说明了次级循环的执行。跟在**First-**后面的数字可能与您的输出不同：

**First-433 Loop Sleeping ...**

**First-433 Secondary loop completed with a result of true**

**First-433 Secondary loop returned**

1.  虽然次级循环阻塞了当前线程，但并不妨碍窗口继续执行。窗口的 UI 线程仍然活动。为了证明这一点，重新启动应用程序并选择**First**按钮。在两秒内未过去之前，选择**Second**按钮。控制台输出应该类似于以下内容：

**First-360 Loop Sleeping ...**

**Second-416 Loop Sleeping ...**

**First-360 Secondary loop completed with a result of true**

**Second-416 Secondary loop completed with a result of true**

**Second-416 Secondary loop returned**

**First-360 Secondary loop returned**

这说明了次级循环的两个方面。第一是应用程序仍然可以与用户交互，第二是同时执行两个次级循环的行为。具体来说，如果在第一个次级循环完成之前启动第二个次级循环，第一个次级循环将不会恢复，直到嵌套的（第二个）循环终止。

请注意，应用程序仍然响应用户输入。另外，请注意**Second-416**循环在**First-360**之后开始执行。然而，虽然**First-360**在**Second-416**之前完成，正如你所期望的那样，**First-360**循环直到**Second-416**循环返回后才返回并恢复被阻塞的线程的执行。如果在两秒内两次选择**First**按钮，将会看到相同的行为。

## 工作原理...

在`ApplicationWindow`中，我们创建了两个按钮。这些按钮被添加到应用程序中，并与应用程序对`ActionListener`接口的实现相关联。我们使用**First**按钮来说明执行次级循环。

在`actionPerformed`方法中，我们使用`Toolkit`类的`getSystemEventQueue`方法来获取`EventQueue`的实例。然后使用`createSecondaryLoop`方法创建了一个次级循环。

为了跟踪潜在的多个次级循环，我们创建了`Calendar`类的一个实例，并根据当前毫秒数派生了一个以**First-**或**Second-**为后缀的唯一名称。虽然这种技术不能保证唯一的名称，但是两个循环具有相同的名称是不太可能的，这对我们的示例来说已经足够了。

根据按下的按钮，使用`secondaryLoop`对象和一个唯一的名称创建了`WorkerThread`的实例。然后启动了工作线程，并对`secondaryLoop`执行了`enter`方法。

此时，次级循环将执行，当前线程将被阻塞。在`WorkerThread`类中，显示了执行了哪个次级循环的消息。然后暂停两秒，随后显示了次级循环完成以及`exit`方法的返回值。

然后`actionPerformed`方法的线程被解除阻塞，并显示了一条最后的消息，指示次级循环已完成。请注意，此线程在次级循环完成之前被阻塞。

这模仿了从应用程序角度看模态对话框的行为。创建次级循环的线程将被阻塞，直到循环完成。虽然其他线程方法也可以用来实现类似的结果，但这种方法方便且易于使用。

## 还有更多...

如果一个`SecondaryLoop`对象已经处于活动状态，则不可能使用相同的`SecondaryLoop`对象启动一个新的循环。任何尝试这样做都将导致`enter`方法返回`false`。然而，一旦循环完成，循环可以被重用于其他循环。这意味着`enter`方法随后可以针对相同的`SecondaryLoop`对象执行。

## 另请参阅

在第七章中查看*使用新的 JLayer 装饰器为密码字段*的示例，*图形用户界面改进*。如果需要创建一个可以显示在指示长时间运行进程的按钮上的计时器-沙漏类型动画，这个示例可能会有用。

# 处理虚假线程唤醒

当使用多个线程时，一个线程可能需要等待一个或多个其他线程完成。在这种情况下，一种方法是使用`Object`类的`wait`方法等待其他线程完成。这些其他线程需要使用`Object`类的`notify`或`notifyAll`方法来允许等待的线程继续。

然而，在某些情况下可能会发生虚假唤醒。在 Java 7 中，引入了`java.awt.event.InvocationEvent`类的`isDispatched`方法来解决这个问题。

## 准备工作

避免虚假唤醒：

1.  添加一个同步块。

1.  根据特定于应用程序的条件和`isDispatched`方法创建一个`while`循环。

1.  在循环体中使用`wait`方法。

## 如何做...

1.  由于虚假中断的性质，不可能创建一个能够始终展示这种行为的演示应用程序。处理`wait`的推荐方法如下所示：

```java
synchronized (someObject) {
Toolkit toolkit = Toolkit.getDefaultToolkit();
EventQueue eventQueue = toolkit.getSystemEventQueue();
while(someCondition && !eventQueue.isDispatchThread()) {
try {
wait();
}
catch (InterruptedException e) {
}
}
// Continue processing
}

```

1.  这种方法将消除虚假中断。

## 它是如何工作的...

首先，我们为我们正在处理的对象使用了一个同步块。接下来，我们获取了`EventQueue`的一个实例。`while`循环将测试一个特定于应用程序的条件，以确定是否应处于`wait`状态。这可能只是一个布尔变量，指示队列已准备好被处理。循环将在条件为`true`且`isDispatched`方法返回`false`时继续执行。这意味着如果方法返回`true`，则事件实际上是从事件队列中分派出来的。这也将发生在`EventQueue.invokeAndWait`方法中。

线程可能会无缘无故地从`wait`方法中醒来。可能没有调用`notify`或`notifyAll`方法。这可能是由于通常是低级和微妙的 JVM 外部条件引起的。

在早期版本的**Java 语言规范**中，没有提到这个问题。然而，在 Java 5 中，`wait`方法的文档中包括了对这个问题的讨论。对这个问题的澄清可以在 Java 语言规范的第三版中找到，**第 17.8.1 节等待**，位于[`java.sun.com/docs/books/jls/third_edition/html/memory.html#17.8.1`](http://java.sun.com/docs/books/jls/third_edition/html/memory.html#17.8.1)。

# 使用事件处理程序处理小程序初始化状态

JavaScript 代码能够调用小程序方法。但是，在小程序初始化之前是不可能的。任何尝试与小程序通信都将被阻塞，直到小程序加载完成。为了确定小程序何时已加载，Java 7 引入了一个加载状态变量，可以从 JavaScript 代码中访问。我们将探讨如何设置 HTML 文件以检测和响应这些事件。

## 准备工作

使用小程序的加载状态：

1.  创建 JavaScript 函数来处理 applet 加载事件。

1.  部署 applet，将参数`java_status_events`设置为`true`。

## 如何做...

1.  为 Java applet 创建一个新的应用程序。在`java.applet.Applet`类的`init`方法中，我们将创建一个`Graphics`对象来显示一个简单的蓝色矩形，然后延迟两秒。这个延迟将模拟 applet 的加载：

```java
public class SampleApplet extends Applet {
BufferedImage image;
Graphics2D g2d;
public void init() {
int width = getWidth();
int height = getHeight();
image = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
g2d = image.createGraphics();
g2d.setPaint(Color.BLUE);
g2d.fillRect(0, 0, width, height);
try {
Thread.sleep(2000);
}
catch (InterruptedException ie) {
ie.printStackTrace();
}
}
public void paint(Graphics g) {
g.drawImage(image, 0, 0, this);
}
}

```

1.  将 applet 打包在`SampleApplet.jar`文件中。接下来，创建一个 HTML 文件如下。第一部分包括声明一个标题和创建`determineAppletState`函数来检查 applet 的加载状态如下：

```java
<HTML>
<HEAD>
<TITLE>Checking Applet Status</TITLE>
<SCRIPT>
function determineAppletState() {
if (sampleApplet.status == 1) {
document.getElementById("statediv").innerHTML = "Applet loading ...";
sampleApplet.onLoad = onLoadHandler;
}
else if (sampleApplet.status == 2) {
document.getElementById("statediv").innerHTML = "Applet already loaded";
}
else {
document.getElementById("statediv").innerHTML = "Applet entered error while loading";
}
}
function onLoadHandler() {
document.getElementById("loadeddiv").innerHTML = "Applet has loaded";
}
</SCRIPT>
</HEAD>

```

1.  在 HTML 文件的 body 部分之后。它使用`onload`事件调用`determineAppletState`函数。然后是一个标题字段和两个分区标签。这些分区将用于显示目的如下：

```java
<BODY onload="determineAppletState()">
<H3>Sample Applet</H3>
<DIV ID="statediv">state</DIV>
<DIV ID="loadeddiv"></DIV>

```

1.  使用 JavaScript 序列完成 HTML 文件，配置和执行 applet 如下：

```java
<DIV>
<SCRIPT src="img/deployJava.js"></SCRIPT>
<SCRIPT>
var attributes = {id:'sampleApplet', code:'SampleApplet.class', archive:'SampleApplet.jar', width:200,
height:100};
var parameters = {java_status_events: 'true'};
deployJava.runApplet(attributes, parameters, '7'7);
</SCRIPT>
</DIV>
</BODY>
</HTML>

```

1.  将 applet 加载到浏览器中。这里，它加载到 Chrome 中如下：

![如何做...](img/5627_08_05.jpg)

## 它是如何工作的...

`SampleApplet`拥有两个方法：`init`和`paint`。`init`方法创建了一个`BufferedImage`对象，用于显示一个蓝色的正方形，其大小由分配给 applet 的区域确定。最初，使用`sleep`方法延迟加载两秒，以模拟加载缓慢的 applet。`paint`方法只是显示图像。当状态为加载时，指定`onLoadHandler`作为 applet 加载完成时要调用的函数。执行此函数时，在`loadeddiv`分区元素中显示相应的消息。

在 HTML 文件的 body 标签中，指定了`determineAppletState`函数作为在 HTML 加载到浏览器时执行的函数。这确保了在加载 HTML 文件时检查加载状态。

将变量和属性与`SampleApplet`类相关联的`sampleApplet` ID。还指定了包含类的存档文件和 applet 的大小。为了利用这一功能，applet 需要使用`java_status_events`参数设置为`true`进行部署。

`determineAppletState`函数使用加载状态变量 status 来显示加载过程的状态。在 HTML 分区元素中显示的消息显示了操作的顺序。

`deployJava.js`是**Java 部署工具包**的一部分，用于检测 JRE 的存在，如果需要则安装一个，然后运行 applet。它也可以用于其他**Web Start**程序。在这种情况下，它被用来使用属性和参数以及要使用的 JRE 版本（即 Java 7）来执行 applet。

### 注意

有关使用`deployJava.js`执行 Java 应用程序部署的更多信息，请访问[`download.oracle.com/javase/7/docs/technotes/guides/jweb/index.html.`](http://download.oracle.com/javase/7/docs/technotes/guides/jweb/index.html.)

以下表格详细介绍了三种 applet 状态值：

| 状态 | 值 | 含义 |
| --- | --- | --- |
| `LOADING` | *1* | applet 正在加载 |
| `READY` | *2* | applet 已加载 |
