# 设置 Selenium Grid

现在我们已经了解了 RemoteWebDriver 是什么以及它是如何工作的，我们准备学习 Selenium Grid。在本章中，我们将涵盖以下主题：

+   为什么我们需要 Selenium Grid

+   什么是 Selenium Grid

+   我们如何使用 Selenium Grid

+   使用 Selenium Grid 的测试用例

+   配置 Selenium Grid

# 探索 Selenium Grid

通过分析一个场景，让我们尝试理解为什么我们需要 Selenium Grid。您有一个 Web 应用程序需要在以下浏览器-机器组合上进行测试：

+   Windows 10 上的 Google Chrome

+   macOS 上的 Google Chrome

+   Windows 10 上的 Internet Explorer 11

+   Linux 上的 Firefox

我们可以简单地修改上一章中创建的测试脚本，将其指向在每个这些组合（即 Windows 10、macOS 或 Linux）上运行的 Selenium Standalone Server，如下面的代码所示。

Windows 10:

```java
DesiredCapabilities caps = new DesiredCapabilities();
caps.setBrowserName("chrome");
caps.setPlatform(Platform.WIN10);
WebDriver driver = new RemoteWebDriver(new URL("http://<win_10_ip>:4444/wd/hub"), capabilities);
```

macOS:

```java
DesiredCapabilities caps = new DesiredCapabilities();
caps.setBrowserName("chrome");
caps.setPlatform(Platform.MAC);
WebDriver driver = new RemoteWebDriver(new URL("http://<mac_os_ip>:4444/wd/hub"), capabilities);

```

Linux:

```java
DesiredCapabilities caps = new DesiredCapabilities();
caps.setBrowserName("chrome");
caps.setPlatform(Platform.LINUX);
WebDriver driver = new RemoteWebDriver(new URL("http://<linux_ip>:4444/wd/hub"), capabilities);
```

在前面的代码中，您的测试脚本与托管目标平台和目标浏览器的机器紧密耦合。如果 Windows 10 主机发生变化，您应该重构您的测试脚本以处理这种情况。这不是设计测试的理想方式。您的测试脚本应该关注您的 Web 应用程序的功能，而不是执行这些测试脚本所使用的底层基础设施。应该有一个中心点来管理所有不同的环境。为了解决这个问题，我们利用**Selenium Grid**。

**Selenium Grid**提供了一个跨浏览器测试环境，支持多个不同的平台（如 Windows、Mac 和 Linux）来执行测试。Selenium Grid 从中心点管理，称为**中心节点**。中心节点拥有所有不同测试平台的信息，称为**节点**（具有所需操作系统和浏览器版本并连接到中心节点的机器）。中心节点根据测试请求的能力将节点分配给执行测试，以下图显示了 Selenium Grid 的外观：

![图片](img/fbc7b253-0888-47d2-a7e7-4279585bd71d.png)

在前面的图中，有一个中心节点，四个不同平台的*节点*，以及存放测试脚本的机器。测试脚本将与中心节点通信，请求执行目标平台。中心节点将具有目标平台的节点分配给测试脚本。节点执行测试脚本并将结果发送回中心节点，中心节点再将结果转发给测试脚本。这就是 Selenium Grid 的外观及其在高级别上的工作原理。

现在我们已经从理论上了解了 Selenium Grid 的工作原理，接下来让我们看看它中的中心节点和节点是如何工作的。幸运的是，因为我们正在处理 Selenium Grid，所以我们可以使用与上一章相同的 Remote WebDriver 服务器来作为 Selenium Grid 使用。如果你还记得，我们使用 `seleniumserver-standalone-3.12.0.jar` 来启动 Selenium Standalone Server。我们可以使用相同的 JAR 文件在中心节点机器上以中心节点模式启动，并在节点机器上启动 JAR 文件的副本以节点模式运行。尝试在你的 JAR 文件上执行以下命令：

```java
java –jar selenium-server-standalone-3.12.0.jar –help
```

以下输出显示了如何在网格环境中使用服务器：

![](img/8247e16b-eaa5-4054-979b-90ffc3ad7c72.png)

您将看到两个选项：将其用作独立服务器，它作为远程 WebDriver 运行，以及将其用于网格环境，这描述了 Selenium Grid。在本章中，我们将使用此 JAR 文件作为 Selenium Grid。

# 理解中心节点

中心节点是 Selenium Grid 的中心点。它有一个所有已连接节点和特定网格的注册表。中心节点是一个以中心节点模式运行的 Selenium Standalone 服务器，默认情况下监听机器的 `4444` 端口。测试脚本将尝试连接到该端口的中心节点，就像任何 Remote WebDriver 一样。中心节点将负责重新路由测试脚本流量到适当的测试平台节点。让我们看看如何启动一个中心节点。导航到您 Selenium 服务器 JAR 文件的位置，并执行以下命令：

```java
java -jar selenium-server-standalone-3.12.0.jar -role hub
```

这样做将在中心节点模式下启动您的服务器。默认情况下，服务器将在 `4444` 端口上监听；然而，您可以选择在您选择的端口上启动服务器。假设您想将服务器启动在 `1111` 端口上；可以按照以下方式完成：

```java
java -jar selenium-server-standalone-3.12.0.jar -role hub –port 1111
```

以下截图显示了在 `1111` 端口上启动的 Grid 中心节点的控制台输出：

![](img/6459fe27-fa58-4c64-b753-8cccc6bc503b.png)

所有测试脚本都应该连接到该端口的中心节点。现在启动您的浏览器并连接到在 `1111` 端口上托管您的中心节点的机器。在这里，托管我的中心节点的机器的 IP 地址是 192.168.0.101。

您在浏览器上应该看到的是以下截图所示：

![](img/f77746ed-d5f5-4e91-9b25-a59b4462e838.png)

它显示了正在用作 Grid 中心节点的服务器的版本。现在点击控制台链接以导航到 Grid 控制台：

![](img/1af75304-9500-4607-a906-510654f2310b.png)

如您所见，页面讨论了许多配置参数。我们将在 *配置 Selenium Grid* 部分讨论这些配置参数。所以，你现在已经学会了如何在端口上启动网格并监听连接。

# 理解节点

由于我们的中心节点正在运行，现在是时候启动一个节点并将其连接到中心节点了。在这个例子中，我们将配置一台安装了 Chrome 的 macOS 机器。因此，如果任何测试脚本请求中心节点 macOS 平台和 Chrome 浏览器，中心节点将选择这个节点。让我们看看我们如何启动节点。启动节点并将其注册到中心节点的命令如下：

```java
java –jar selenium-server-standalone-3.12.0.jar –role node –hub http://192.168.0.101:1111/grid/register
```

这将启动 Selenium 服务器在节点模式下，并将此节点注册到已启动的中心节点：

![](img/91992fe4-f543-45ad-abe3-006e42cbc925.png)

如果你回到浏览器上的 Grid 控制台，你会看到以下内容：

![](img/77dc8da7-4873-4de6-9730-50f8447e3f8e.png)

前面的截图显示了 http://192.168.0.101:16784 节点 URL，在这种情况下，它运行在 Mac 平台上。默认情况下，每个节点列出的浏览器数量为 11：Firefox 有 5 个，Chrome 有 5 个，IE 有 1 个。这可以通过指定`browser`选项来覆盖，我们将在*配置 Selenium Grid*部分中看到。

同样，在 Windows 上启动另一个节点并使用与在 macOS 上启动节点相同的命令将其注册到中心节点。

# 将现有测试脚本修改为使用 Selenium Grid

到目前为止，我们已经看到了在本地机器或 Selenium Standalone 服务器上运行的测试脚本。在 Selenium Grid 上执行测试脚本与在 Remote WebDriver 上执行测试非常相似，只是你还需要提及 Grid 的平台详情。

让我们看看一个使用 Remote WebDriver 服务器的测试脚本：

```java
public class SearchTest {

    WebDriver driver;

    @BeforeMethod
    public void setup() throws MalformedURLException {

        DesiredCapabilities caps = new DesiredCapabilities();

        caps.setBrowserName("chrome");
        caps.setPlatform(Platform.MAC);

        driver = new RemoteWebDriver(new URL("http://192.168.0.101:1111/wd/hub"), caps);
        driver.get("http://demo-store.seleniumacademy.com/");

    }

    @Test
    public void searchProduct() {

        // find search box and enter search string
        WebElement searchBox = driver.findElement(By.name("q"));

        searchBox.sendKeys("Phones");

        WebElement searchButton =
                driver.findElement(By.className("search-button"));

        searchButton.click();

        assertThat(driver.getTitle())
                .isEqualTo("Search results for: 'Phones'");
    }

    @AfterMethod
    public void tearDown() {
        driver.quit();
    }
} 
```

现在尝试执行前面的测试脚本并观察中心节点和节点的日志输出。中心节点的输出日志如下：

![](img/f1aa8480-8f1b-499d-bf4f-f24a999d0261.png)

在中心节点发生的步骤顺序如下：

1.  中心节点收到创建新会话的请求，`platform=MAC, browserName=chrome`。

1.  它验证与`capabilities`请求匹配的可用节点。

1.  如果可用，它将使用节点主机创建一个新的会话；如果没有，它将拒绝测试脚本的请求，表示期望的能力不匹配任何已注册的节点。

1.  如果在前一步中与节点主机创建了会话，则创建一个新的测试槽会话并将测试脚本交给节点。同样，你应在中心节点的控制台日志中看到以下输出：

![](img/4c9ca183-59a5-45a3-a5f0-b81d29f053cd.png)

在节点上执行的步骤顺序如下：

1.  节点主机使用请求的期望能力创建一个新的会话。这将启动浏览器。

1.  它在启动的浏览器上执行测试脚本的步骤。

1.  它结束会话并将结果转发到中心节点，中心节点随后将其发送到测试脚本。

# 请求非注册的能力

当测试脚本请求中心节点未注册的能力时，中心节点将拒绝请求。让我们修改前面的测试脚本，请求 Opera 浏览器而不是 Chrome。测试脚本应如下所示：

```java
@BeforeMethod
public void setup() throws MalformedURLException {

    DesiredCapabilities caps = new DesiredCapabilities();

    caps.setBrowserName("opera");
    caps.setPlatform(Platform.MAC);

    driver = new RemoteWebDriver(new URL("http://192.168.0.101:1111/wd/hub"), caps);
    driver.get("http://demo-store.seleniumacademy.com/");

}
```

中心节点检查是否有任何节点与所需的能力相匹配。如果没有找到（如本例所示），它将通过抛出 `CapabilityNotPresentOnTheGridException` 异常来拒绝测试脚本的请求，如下面的截图所示：

![图片](img/c0e6b5a2-26eb-47c3-b637-65aa72d3d77e.png)

# 如果节点忙碌，则排队请求

默认情况下，您可以向任何节点发送五个测试脚本请求。尽管可以更改此配置，但让我们看看当节点已经处理了五个请求，并且您通过中心节点向该节点发送另一个请求时会发生什么。中心节点将一直轮询节点，直到从节点获得一个空闲的测试槽位。测试脚本将在这段时间内等待。中心节点表示没有空闲槽位来建立与同一节点的第六个会话。同时，在节点主机上，节点试图为五个请求创建会话并开始执行测试脚本。

在创建会话后，将启动五个 Chrome 窗口，并在其上执行测试脚本。在处理前五个测试脚本请求后，中心节点将与节点建立等待的第六个会话，并处理第六个请求。

# 处理具有匹配能力的两个节点

Selenium Grid 提供了许多配置选项，用于在执行测试脚本时控制节点和中心节点的行为。我们将在下面讨论它们。

# 配置 Selenium Grid

Selenium Grid 提供了许多配置选项，用于在执行测试脚本时控制节点和中心节点的行为。我们将在下面讨论它们。

# 指定节点配置参数

在本节中，我们将介绍节点的配置参数。

# 设置节点支持的浏览器

如我们之前所看到的，当我们使用中心节点注册节点时，默认情况下，节点显示为支持五个 Firefox 浏览器实例、五个 Chrome 浏览器实例和一个 Internet Explorer 实例，无论节点实际上是否支持它们。但为了使用您选择的浏览器注册您的节点，Selenium Grid 提供了一个浏览器选项，我们可以使用该选项实现这一点。假设我们希望我们的节点注册为支持 Firefox、Chrome 和 Safari；我们可以使用以下命令来完成：

```java
java -jar selenium-server-standalone-3.12.0.jar -role node -hub http://192.168.0.1:1111/grid/register -browser browserName=firefox -browser browserName=chrome -browser browserName=safari
```

网格控制台看起来如下所示：

![图片](img/bf5f2d66-9ec1-4d46-9a7b-83d57842e2b6.png)

# 设置节点超时

此参数在将节点注册到中心节点时设置。提供给这些参数的值是中心节点在终止节点上的测试脚本执行之前可以等待的秒数，如果测试脚本在节点上不执行任何活动。

配置节点的节点超时的命令如下：

```java
java -jar selenium-server-standalone-3.12.0.jar -role node -hub http://192.168.0.1:1111/grid/register -nodeTimeout 300
```

这里，我们注册了一个节点，其节点超时值为 300 秒。因此，如果节点在 300 秒内没有任何活动，中心节点将终止测试脚本。

# 设置浏览器实例的限制

我们已经看到，默认情况下，有 11 个浏览器实例被注册到节点上。我们已经看到了如何注册我们自己的浏览器。在本节中，我们将看到我们可以在节点上允许多少个这样的浏览器实例。为了控制这一点，Selenium Grid 提供了一个名为`maxInstances`的配置参数，使用它可以指定我们希望节点提供的特定浏览器实例的数量。执行该命令如下：

```java
java -jar selenium-server-standalone-3.12.0.jar -role node -hub http://192.168.0.1:1111/grid/register -browser "browserName=firefox,max Instances=3" -browser "browserName=chrome,maxInstances=3" -browser "browserName=safari,maxInstances=1"
```

这里，我们正在注册一个提供三个 Firefox 实例、三个 Chrome 实例和一个 Safari 实例的节点。

# 自动重新注册节点

如果一个节点在注册到中心节点后，中心节点崩溃或重启，那么已经注册的节点的所有信息都会丢失。手动回到每个节点并重新注册它们将是一件繁琐的事情。如果我们还没有意识到中心节点已经重启，那么影响将会更糟，因为所有测试脚本都会因此失败。因此，为了处理这种情况，Selenium Grid 为节点提供了一个配置参数，通过这个参数我们可以指定节点在指定时间后自动重新注册到中心节点。如果没有指定，重新注册的默认时间为五秒。这样，我们真的不必担心；即使中心节点崩溃或重启，我们的节点也会每五秒尝试重新注册。

如果你想要修改这个时间间隔，需要处理的配置参数是`registerCycle`。指定命令如下：

```java
java -jar selenium-server-standalone-3.12.0.jar -role node -hub http://192.168.0.1:1111/grid/register -registerCycle 10000
```

在启动时，你将在节点日志控制台看到以下输出：

```java
17:47:01.231 INFO - starting auto register thread. Will try to register every 10000 ms.
 17:47:01.232 INFO - Registering the node to hub :http://192.168.0.1:1111/grid/register
```

节点将尝试每 1,000 毫秒向中心节点注册一次。

# 设置节点健康检查时间

使用这个配置参数，我们可以指定中心节点可以多频繁地轮询节点以检查其可用性。用于实现这一点的参数是`nodePolling`。通过在节点级别指定此参数到中心节点，每个节点可以指定它自己的健康检查频率。配置节点的命令如下：

```java
 java -jar selenium-server-standalone-3.12.0.jar -role node -hub http://192.168.0.1:1111/grid/register -nodePolling 10
```

现在，中心节点将每 10 秒轮询这个节点，以检查其可用性。

# 注销不可用的节点

虽然配置参数`nodePolling`会使中心节点频繁轮询节点，但`unregisterIfStillDownAfter`配置参数将允许中心节点在轮询没有产生预期结果时注销节点。假设一个节点已经关闭，中心节点尝试轮询该节点并无法连接到它。在这种情况下，中心节点将轮询节点以检查其可用性的时间由`unregisterIfStillDownAfter`参数确定。超过这个时间，中心节点将注销节点。

执行该命令的方式如下：

```java
java -jar selenium-server-standalone-3.12.0.jar -role node -hub http://192.168.0.1:1111/grid/register -nodePolling 5 -unregistIfStillDownAfter 20000
```

在这里，中心节点将每五秒轮询一次节点；如果节点关闭，轮询将持续 20 秒，即中心节点将轮询四次，然后从网格中注销该节点。

# 设置浏览器超时

此配置是为了让节点知道在浏览器似乎挂起之前应该等待多长时间才结束测试脚本会话。在此时间之后，节点将终止浏览器会话，并开始下一个等待的测试脚本。此配置参数为`browserTimeout`。指定此参数的命令如下：

```java
java -jar selenium-server-standalone-3.12.0.jar -role node -hub http://192.168.0.1:1111/grid/register –browserTimeout 60
```

因此，这些是一些您可以在节点端指定以更好地控制 Selenium Grid 环境的配置参数。

# 中心节点配置参数

本节讨论了中心节点上的一些配置参数。

# 等待所需能力的匹配

如我们之前所看到的，当测试脚本请求具有所需能力的测试平台时，如果中心节点找不到具有所需能力的合适节点，它将拒绝请求。

修改`throwOnCapabilityNotPresent`参数的值可以改变这种行为。默认情况下，它设置为`true`，这意味着如果中心节点找不到具有该能力的合适节点，它将拒绝请求。但将此参数设置为`false`将排队请求，中心节点将等待直到具有该能力的节点添加到网格中。必须调用的命令如下：

```java
java -jar selenium-server-standalone-3.12.0.jar -role hub -port 1111 -throwOnCapabilityNotPresent false
```

现在，中心节点不会拒绝请求，而是将请求放入队列中，等待请求的平台可用。

# 定制的能力匹配器

默认情况下，中心节点将使用`org.openqa.grid.internal.utils.DefaultCapabilityMatcher`类来匹配请求的节点。如果您不喜欢`DefaultCapabilityMatcher`类的实现逻辑，您可以扩展该类，实现自己的`CapabilityMatcher`类，并在其中提供自己的逻辑。

一旦开发完成，您可以让中心节点使用该类来匹配节点的能力，使用一个名为`capabilityMatcher`的配置参数。实现此目的的命令如下：

```java
java -jar selenium-server-standalone-3.12.0.jar -role hub -port 1111 -capabilityMatcher com.yourcomp.CustomCapabilityMatcher
```

中心节点将使用您在`CustomCapabilityMatcher`类中定义的逻辑来识别分配给测试脚本请求的节点。

# 新会话的等待超时

当一个能力匹配的节点正在执行其他测试脚本时，最新的测试脚本将等待节点可用。默认情况下，没有等待超时；也就是说，测试脚本将无限期地等待节点可用。为了改变这种行为，并让测试脚本在有限时间内无法获取节点时抛出异常，Selenium Grid 打开一个配置，使测试脚本能够这样做。控制该行为的配置参数是`newSessionWaitTimeout`。该命令如下：

```java
java -jar selenium-server-standalone-3.12.0.jar -role hub -port 1111 -newSessionWaitTimeout 120000
```

在这里，测试脚本将在抛出异常说它无法获取执行自身的节点之前等待两分钟。

# 指定配置的不同方式

有两种方式可以指定配置参数给 Selenium Grid 的 hub 和节点。第一种就是我们一直看到的方法；即通过命令行指定配置参数。第二种方法是提供一个包含所有这些配置参数的 JSON 文件。

一个节点配置文件（例如，`nodeConfig.json`）——一个典型的包含所有配置参数的 JSON 文件——看起来类似于以下内容：

```java
{
  "class": "org.openqa.grid.common.RegistrationRequest",
  "capabilities": [
   {
   "seleniumProtocol": "WebDriver",
   "browserName": "internet explorer",
   "version": "10",
   "maxInstances": 1,
   "platform" : "WINDOWS"
   }
  ],
  "configuration": {
   "port": 5555,
   "register": true,
   "host": "192.168.1.102",
   "proxy": "org.openqa.grid.selenium.proxy.
   DefaultRemoteProxy",
   "maxSession": 2,
   "hubHost": "192.168.1.100",
   "role": "webdriver",
   "registerCycle": 5000,
   "hub": "http://192.168.1.101:111/grid/register",
   "hubPort": 1111,
   "remoteHost": "http://192.168.1.102:5555"
  }
}
```

一旦这些文件配置完成，可以使用以下命令将它们提供给节点和 hub：

```java
java -jar selenium-server-standalone-3.12.0.jar -role node -nodeConfig nodeconfig.json 
```

这样，你可以使用 JSON 文件指定你的 hub 和节点的配置。

# 使用基于云的网格进行跨浏览器测试

要设置一个用于跨浏览器测试的 Selenium Grid，你需要配置具有不同浏览器和操作系统的物理或虚拟机。这需要投资必要的硬件、软件和支持，以运行测试实验室。你还需要付出努力，确保这个基础设施保持最新版本和补丁。并不是每个人都能承担这些成本和努力。

与投资和设置跨浏览器测试实验室相比，你可以轻松地将虚拟测试实验室外包给第三方云服务提供商进行跨浏览器测试。Sauce Labs 和 BrowserStack 是领先的基于云的跨浏览器测试云服务提供商。这两个都支持超过 400 种不同的浏览器和操作系统配置，包括移动和平板设备，并支持在他们的云中运行 Selenium WebDriver 测试。

在这里，我们将设置并运行 Sauce Labs 云上的测试。如果你想要使用 BrowserStack 运行测试，步骤是相似的。

让我们使用 Sauce Labs 设置并运行一个测试。你需要一个免费的 Sauce Labs 账户。在 Sauce Labs 上注册一个免费账户，网址为 [`saucelabs.com/`](https://saucelabs.com/)，并获取用户名和访问密钥。Sauce Labs 提供所有必要的硬件和软件基础设施，以便你在云中运行测试。你可以在登录后从“我的账户”页面获取访问密钥：

![截图](img/d93aacf3-b822-4864-80df-e010bfede7a3.png)

让我们在 Sauce Labs 云上创建一个新的测试来执行。我们需要将 Sauce 用户名和访问密钥添加到测试中，并将网格地址更改为 Sauce Labs 网格地址，而不是本地 Selenium Grid，如下面的代码示例所示：

```java
public class BmiCalculatorTest {

    WebDriver driver;

    @BeforeMethod
    public void setUp() throws Exception {

        String SAUCE_USER = "upgundecha";
        String SAUCE_KEY = "5768f2a9-33be-4ebd-9a5f-3826d7c38ec9";

        DesiredCapabilities caps = new DesiredCapabilities();
        caps.setCapability("platform", "OS X 10.9");
        caps.setCapability("browserName", "Safari");
        caps.setCapability("name", "BMI Calculator Test");
        driver = new RemoteWebDriver(
                new URL(MessageFormat.format("http://{0}:{1}@ondemand.saucelabs.com:80/wd/hub'",
                SAUCE_USER, SAUCE_KEY)), caps);
        driver.get("http://bit.ly/1zdNrFZ");

    }

    @Test
    public void testBmiCalc() {
        WebElement height = driver.findElement(By.name("heightCMS"));
        height.sendKeys("181");

        WebElement weight = driver.findElement(By.name("weightKg"));
        weight.sendKeys("80");

        WebElement calculateButton = driver.findElement(By.id("Calculate"));
        calculateButton.click();

        WebElement bmi = driver.findElement(By.name("bmi"));
        assertEquals(bmi.getAttribute("value"), "24.4");

        WebElement bmi_category = driver.findElement(By.name("bmi_category"));
        assertEquals(bmi_category.getAttribute("value"), "Normal");
    }

    @AfterMethod
    public void tearDown() throws Exception {
        driver.quit();
    }
}
```

当你执行测试时，它将连接到 Sauce Lab 的 hub 并请求所需的操作系统和浏览器配置。Sauce Labs 的云管理软件会自动分配一个虚拟机，以便在我们的测试配置上运行。我们可以在以下截图所示的仪表板上监控这个运行：

![截图](img/dc6ef1a3-3007-4769-8682-456574cbfa93.png)

我们可以进一步深入会话，查看运行期间确切发生了什么。它提供了 Selenium 命令、截图、日志以及多个标签页上的执行视频的详细信息，如下面的截图所示：

![图片](img/fae2e51c-21b7-4d7b-9b47-2f29c7a10bd8.png)

Selenium 详情窗口

您还可以通过使用 Sauce Connect 实用程序来测试安全托管在内部服务器上的应用程序。Sauce Connect 在您的机器和 Sauce 云之间创建一个安全隧道。

# 摘要

在本章中，我们学习了 Selenium Grid，了解了 hub 和 node 的工作方式，更重要的是，学习了如何配置您的 Selenium Grid 以更好地控制环境和基础设施。Selenium Grid 将通过覆盖操作系统和浏览器的组合来为应用程序提供跨浏览器测试。我们还看到了如何使用云服务，如 Sauce Labs，在远程云环境中执行测试。

在下一章节中，我们将学习如何使用页面对象模式创建可重用和模块化的测试。

# 问题

1.  哪个参数可以用来指定节点可以支持多少浏览器实例？

1.  解释如何使用 Selenium Grid 来支持跨浏览器测试。

1.  使用 RemoteWebDriver 运行 Selenium Grid 测试需要指定哪个 URL？

1.  Selenium Grid Hub 充当负载均衡器——对还是错？

# 更多信息

您可以查看以下链接以获取有关本章涵盖主题的更多信息：

+   在[`www.seleniumhq.org/docs/07_selenium_grid.jsp`](https://www.seleniumhq.org/docs/07_selenium_grid.jsp)上了解更多关于 Selenium Grid 的信息
