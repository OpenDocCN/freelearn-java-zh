# 第十章：使用 Appium 在 iOS 和 Android 上进行移动测试

在所有前面的章节中，我们一直在处理在桌面浏览器中加载的 Web 应用程序。但随着移动用户数量的增加，今天的商业企业必须在其移动设备上为用户提供服务。在本章中，您将学习以下内容：

+   不同类型的移动应用程序和测试工具

+   如何使用 Selenium WebDriver 测试移动应用程序，特别是使用 Appium

+   在 Android 和 iOS 上测试移动应用程序

+   使用基于云的设备实验室进行真实设备测试

`Appium`是一个开源的移动自动化框架，用于使用 Selenium WebDriver 的 JSON 线协议在 iOS 和 Android 平台上测试移动应用程序。Appium 取代了 Selenium 2 中用于测试移动 Web 应用程序的 iPhoneDriver 和 AndroidDriver API。

# 移动应用程序的不同形式

一个应用程序可以在移动平台上以三种不同的形式触达用户：

+   **原生应用**：原生应用纯粹针对目标移动平台。它们使用平台支持的语言开发，并且与底层的 SDK 紧密相关。对于 iOS，应用程序使用 Objective-C 或 Swift 编程语言开发，并依赖于 iOS SDK；同样，对于 Android 平台，它们使用 Java 或 Kotlin 开发，并依赖于 Android SDK。

+   **m.site**：也称为移动网站，它是您 Web 应用程序的迷你版，在您的移动设备浏览器上加载。在 iOS 设备上，它可以是 Safari 或 Chrome，在 Android 设备上，它可以是 Android 默认浏览器或 Chrome。例如，在您的 iOS 或 Android 设备上，打开浏览器并输入[www.facebook.com](http://www.facebook.com)。在页面加载之前，您会观察到从[www.facebook.com](http://www.facebook.com)到[m.facebook.com](http://m.facebook.com)的 URL 重定向发生。Facebook 应用程序服务器意识到请求是从移动设备发起的，并开始提供移动网站而不是桌面网站。这些 m 网站使用 JavaScript 和 HTML5 来开发，就像您的正常 Web 应用程序一样：

![图片](img/897b1347-98fc-4c96-9ce4-2e6698ec15f6.png)

+   **混合应用**：混合应用是原生应用和 Web 应用的结合。当您开发原生应用时，其中一些部分会加载 HTML 网页到应用中，试图让用户感觉他们正在使用原生应用。它们通常在原生应用中使用 WebView 来加载网页。

现在，作为一名测试脚本开发者，您必须在各种移动设备上测试所有这些不同的应用程序。

# 可用的软件工具

为了自动化在移动设备上测试您的应用程序，有许多软件工具可供选择。以下是一些基于 Selenium WebDriver 构建的工具：

+   **Appium**：一个基于 Selenium 的跨平台和跨技术移动测试框架，适用于原生、混合和移动 Web 应用程序。Appium 允许使用和扩展现有的 Selenium WebDriver 框架来构建移动测试。由于它使用 Selenium WebDriver 驱动测试，我们可以使用任何存在 Selenium 客户端库的语言来创建测试。您可以在不更改底层驱动程序的情况下，针对 Android 和 iOS 平台创建和执行测试脚本。Appium 还可以与 Firefox OS 平台协同工作。在本章的其余部分，我们将看到如何与 Appium 协同工作。

+   **Selendroid**：此驱动程序类似于 iOSDriver，可以在 Android 平台上执行您的原生、混合和 m.site 应用程序测试脚本。它使用 Google 提供的本地 UI Automator 库。测试脚本通过 JSON 线协议与 Selendroid 驱动程序通信，同时使用其最喜欢的客户端语言绑定。

# 使用 Appium 自动化 iOS 和 Android 测试

Appium 是一种流行的广泛使用的工具，可用于自动化 Android 和 iOS 平台的移动应用测试。它可以用于自动化原生、m.sites 和混合应用程序。它内部使用 WebDriver 的 JSON 线协议。

# 自动化 iOS 应用程序测试

对于自动化 iOS 应用测试，Appium 使用 XCTest 或 UI Automation（适用于较旧的 iOS 版本）：

+   **XCTest**：您可以使用 XCTest 创建和运行针对为 iOS 9.3 及更高版本构建的 iOS 应用程序的单元测试、性能测试和 UI 测试。它与 Xcode 的测试工作流程集成，用于测试 iOS 应用程序。Appium 内部使用 XCTest 自动化 iOS 应用程序。

+   **UI Automation**：对于测试为 iOS 9.3 及以下版本开发的 iOS 应用，您需要使用 UI Automation。Appium 通过 JSON 线协议从测试脚本接收命令。Appium 将这些命令发送到 Apple Instruments，以便在模拟器或真实设备上启动的应用中执行。在此过程中，Appium 将 JSON 命令转换为仪器可以理解的 UI Automation JavaScript 命令。仪器负责在模拟器或设备上启动和关闭应用。

Appium 作为远程 WebDriver，通过 JSON 线协议接收来自您的测试脚本的命令。这些命令被传递给 XCTest 或 Apple Instruments，以便在模拟器或真实设备上启动的应用中执行。此过程在以下图中展示：

![](img/0c2d9f49-39a1-4edd-b570-87d7c37113e5.png)

在模拟器或设备上对您的应用执行命令后，目标应用会将响应发送给 XCTest 或 UI Automation Instrument，这些响应以 JavaScript 响应格式传输到 Appium。Appium 将响应转换为 Selenium WebDriver JSON 线协议响应，并将其发送回您的测试脚本。

使用 Appium 进行 iOS 自动化测试的主要优势如下：

+   它使用 iOS 平台支持的 XCTest 或苹果本身提供的 UI Automation 库和工具。

+   尽管你使用的是 JavaScript 库，但你、测试脚本开发者和你的测试脚本实际上并没有真正绑定到它。你可以使用自己的 Selenium WebDriver 客户端语言绑定，如 Java、Ruby 或 Python，来开发你的测试脚本。Appium 将为你处理将它们转换为 JavaScript。

+   你不需要修改你的原生或混合应用以进行测试目的。

# 自动化 Android 应用程序测试

自动化你的 Android 应用的测试与自动化 iOS 应用的测试类似。除了你的目标平台在变化之外，你的测试脚本不会经历任何变化。以下图显示了工作流程：

![图片](img/b060e218-e8e7-489f-8092-8666734b8978.png)

再次，Appium 作为一个远程 WebDriver，通过 JSON 协议接收来自你的测试脚本的命令。这些命令被传递给 Android SDK 附带的 Google UI Automator 来在模拟器或真实设备上执行。在命令传递给 UI Automator 之前，Appium 将 JSON 命令转换为 UI Automator 可以理解的命令。这个 UI Automator 将在模拟器或真实设备上启动你的应用，并开始执行你的测试脚本命令。在模拟器或设备上执行命令后，目标应用将响应发送给 UI Automator，该响应以 UI Automator 响应格式传输到 Appium。Appium 将 UI Automator 响应转换为 Selenium WebDriver JSON 协议响应，并将它们发送回你的测试脚本。

这是帮助你理解 Appium 如何与 Android 和 iOS 设备协同工作以执行你的测试命令的高级架构。

# Appium 的先决条件

在我们开始讨论 Appium 的一些工作示例之前，我们需要为 iOS 和 Android 平台安装一些先决工具。我们需要设置 Xcode 和 Android Studio 来完成这项任务，我将在这个例子中使用 macOS。

# 设置 Xcode

要设置 Xcode，我们将执行以下步骤：

1.  你可以从[`developer.apple.com/xcode/`](https://developer.apple.com/xcode/)下载最新的 Xcode。

1.  下载后，安装并打开它。

1.  导航到“首选项”|“组件”以下载和安装命令行工具和 iOS 模拟器，如下面的截图所示：

![图片](img/438d70e8-cfdc-4060-be80-6bda85a9d803.png)

如果你使用的是真实设备，你需要在设备上安装一个 provisioning profile，并启用 USB 调试。

尝试启动 iPhone 模拟器并验证它是否工作。你可以通过导航到 Xcode | 打开开发者工具 | iOS 模拟器来启动模拟器。模拟器应该看起来与以下截图所示相似：

![图片](img/e1de65ea-0d2c-4391-8e3d-bc078040fda4.png)

# 设置 Android SDK

您需要从 [`developer.android.com/studio/`](https://developer.android.com/studio/) 安装 Android SDK。下载并安装 Android Studio。

启动已安装的 Android Studio。现在下载任何 API 级别为 27 的 Android，并安装它。您可以通过导航到“工具”|“SDK 管理器”来完成此操作。您应该会看到类似于以下截图的内容：

![](img/6e2f86c0-47a9-4085-bf04-31a4a462123a.png)

在这里，我们正在安装 API 级别为 27 的 Android 8.1。

# 创建 Android 模拟器

如果您想在 Android 模拟器上执行测试脚本，您必须创建一个。要创建一个，我们将执行以下步骤：

1.  在 Android Studio 中，通过导航到“工具”|“AVD 管理器”来打开 AVD 管理器。它将启动 AVD 管理器，如下面的截图所示：

![](img/d8b68ab6-50f7-47a3-8730-7f57528d7a1a.png)

1.  通过点击“创建虚拟设备...”按钮创建一个新的虚拟设备或模拟器。您应该会看到一个窗口，该窗口将获取您所需的所有必要信息，如下面的截图所示：

![](img/d3cca45e-c895-4045-9731-cbb2c28f06e7.png)

1.  启动模拟器以查看它是否已成功创建。Android 虚拟设备启动可能需要几分钟。下面的截图显示了一个已启动的 Android 模拟器：

![](img/6055aea8-1d4d-4ee3-8e7f-d7b6b53115e0.png)

# 安装 Appium

您可以从 [`appium.io/`](http://appium.io/) 下载 Appium。点击下载 Appium 按钮以下载适用于您工作站平台的 Appium。在这里，我使用的是 Mac，因此它将下载 Appium DMG 文件。

将 Appium 复制到 `Applications` 文件夹，并尝试启动它。第一次启动时，它会要求您的授权以运行 iOS 模拟器，如下面的截图所示：

![](img/b5f94734-f809-46db-9359-e75f1348078c.png)

点击“启动以启动服务器”按钮。默认情况下，它将在 `http://localhost:4723` 上启动。这是您的测试脚本应将测试命令定向到的远程 URL。

# 自动化 iOS

现在我们已经运行了 Appium，因此让我们创建一个测试，该测试将检查 iPhone Safari 浏览器上的搜索测试。让我们使用 `DesiredCapabilities` 类为 Appium 提供能力，以便在 iPhone X 和 iOS 11.4 上运行测试，如下面的代码所示：

```java
public class SearchTest {

    private WebDriver driver;

    @BeforeTest
    public void setUp() throws Exception {

        // Set the desired capabilities for iOS- iPhone X
        DesiredCapabilities caps = new DesiredCapabilities();
        caps.setCapability("platformName", "iOS");
        caps.setCapability("platformVersion", "11.4");
        caps.setCapability("deviceName", "iPhone X");
        caps.setCapability("browserName", "safari");

        // Create an instance of AndroidDriver for testing on Android platform
        // connect to the local Appium server running on a different machine
        // We will use WebElement type for testing the Web application
        driver = new IOSDriver<>(new URL(
                "http://192.168.0.101:4723/wd/hub"), caps);
        driver.get("http://demo-store.seleniumacademy.com/");
    }

    @Test
    public void searchProduct() {

        WebElement lookingGlassIcon =
                driver.findElement(By
                        .cssSelector("a.skip-search span.icon"));

        lookingGlassIcon.click();

        // find search box and enter search string
        WebElement searchBox = driver.findElement(By.name("q"));

        searchBox.sendKeys("Phones");

        WebElement searchButton =
                driver.findElement(By.className("search-button"));

        searchButton.click();

        List<WebElement> searchItems = new WebDriverWait(driver, 30)
                .until(ExpectedConditions
                        .presenceOfAllElementsLocatedBy(By
                                .cssSelector("h2.product-name a")));

        assertThat(searchItems.size())
                .isEqualTo(3);

    }

    @AfterTest
    public void tearDown() throws Exception {
        // Close the browser
        driver.quit();
    }
}
```

如您所见，前面的代码与 `RemoteWebDriver` 的测试脚本类似，但有一些不同。以下代码描述了这些差异：

```java
DesiredCapabilities caps = new DesiredCapabilities();
caps.setCapability("platformName", "iOS");
caps.setCapability("platformVersion", "11.4");
caps.setCapability("deviceName", "iPhone X");
caps.setCapability("browserName", "safari");
```

Appium Java 客户端库提供了 `IOSDriver` 类，它支持在 iOS 平台上执行测试，以便使用 Appium 运行测试。然而，为了使 Appium 能够使用所需的平台，我们需要传递一组所需的配置。`platformName` 配置能力被 Appium 用于决定测试脚本应该在哪个平台上执行。在这个例子中，我们使用了 iPhone X 模拟器。要在 iPad 上运行测试，我们可以指定 iPad 模拟器。

在真实设备上运行测试时，我们需要指定设备能力中 iPhone 或 iPad 的值。Appium 将选择通过 USB 连接到 Mac 的设备。我们最后使用的期望能力是 browserName，它由 Appium 用于启动 Safari 浏览器。

# 为 Android 自动化

在 Android 上测试应用程序与在 iOS 上测试类似。对于 Android，我们将使用真实设备而不是仿真器（在 Android 社区中，仿真器被称为仿真器）。我们将使用相同的应用程序在 Android 的 Chrome 浏览器上测试。

对于这个例子，我正在使用三星 Galaxy S4 Android 手机。我们需要在设备上安装 Google Chrome 浏览器。如果您设备上没有预装 Google Chrome，可以在 Google 的 Play 商店获取。接下来，我们需要将设备连接到运行 Appium 服务器的机器。让我们运行以下命令以获取连接到机器的仿真器或设备的列表：

```java
./adb devices
```

`Android Debug Bridge`（ADB）是 Android SDK 中可用的命令行工具，允许您与仿真器实例或连接到您的计算机的实际 Android 设备通信。`./adb devices`命令将显示连接到主机的所有 Android 设备的列表，如下面的输出所示：

```java
List of devices attached
4df1e76f39e54f43 device
```

让我们修改为 iOS 创建的脚本，以使用 Android 的功能和 AndroidDriver 类在真实 Android 设备上执行测试，如下面的代码所示：

```java
public class MobileBmiCalculatorTest {
    private WebDriver driver;

@BeforeTest
public void setUp() throws Exception {

    // Set the desired capabilities for Android Device
    DesiredCapabilities caps = DesiredCapabilities.android();
    caps.setCapability("deviceOrientation", "portrait");
    caps.setCapability("platformVersion", "8.1");
    caps.setCapability("platformName", "Android");
    caps.setCapability("browserName", "Chrome");

    // Create an instance of AndroidDriver for testing on Android platform
    // connect to the local Appium server running on a different machine
    // We will use WebElement type for testing the Web application
    driver = new AndroidDriver<WebElement>(new URL(
            "http://192.168.0.101:4723/wd/hub"), caps);
    driver.get("http://demo-store.seleniumacademy.com/");
}
```

在前面的例子中，我们将`platformName`能力值分配给了`Android`，这将由 Appium 用于在 Android 上运行测试。由于我们想在 Android 的 Chrome 浏览器上运行测试，我们在代码的浏览器能力部分提到了 Chrome。我们做出的另一个重要改变是使用 Appium Java 客户端库中的`AndroidDriver`类。

Appium 将使用`adb`返回的设备列表中的第一个设备，如下面的截图所示。它将使用我们提到的期望能力，并在设备上启动 Chrome 浏览器并开始执行测试脚本命令。

# 使用 Device Cloud 在真实设备上运行测试

Appium 支持在移动模拟器、仿真器和真实设备上进行测试。要设置一个包含真实设备的移动测试实验室，需要资本投资以及设备和基础设施的维护。手机制造商几乎每天都会发布新的手机型号和操作系统更新，而您的应用程序必须与新发布兼容。

为了更快地应对这些变化并保持投资最小，我们可以使用基于云的移动测试实验室。有许多供应商，如亚马逊网络服务、BrowserStack 和 Sauce Labs，提供基于云的实时移动设备实验室来执行测试，无需对真实设备进行任何前期投资。你只需为测试使用的时间付费。这些供应商还允许你在他们的设备云中运行使用 Appium 的自动化测试。

在本节中，我们将探索 BrowserStack 以在其真实设备云上运行测试：

1.  你需要一个带有**Automate**功能订阅的 BrowserStack 账户。你可以在[`www.browserstack.com/`](https://www.browserstack.com/)注册一个免费试用账户。

1.  我们需要根据设备组合从 BrowserStack 获取所需的配置能力。BrowserStack 根据所选的设备和平台组合提供能力建议。访问[`www.browserstack.com/automate/java`](https://www.browserstack.com/automate/java)并选择一个操作系统和设备：

![](img/1b973f88-79a8-4af1-804a-9f548829ccf4.png)

1.  根据你的选择，BrowserStack 将使用你的用户名和访问密钥自动生成代码：

![](img/8b734eee-5357-4da2-9c8a-cfe2a7fe24a3.png)

我们将不会使用第 3 步中建议的代码，而是将我们的测试更改为以下代码所示。记住，你需要使用自动生成的代码中显示的用户名和访问密钥：

```java
public class SearchTest {

    private WebDriver driver;

    @BeforeTest
    public void setUp() throws Exception {

 String USERNAME = "username";
        String AUTOMATE_KEY = "access_key";
        String URL = "https://" + USERNAME + ":" 
                + AUTOMATE_KEY + "@hub-cloud.browserstack.com/wd/hub";

        // Set the desired capabilities for iPhone X
        DesiredCapabilities caps = new DesiredCapabilities();
        caps.setCapability("browserName", "iPhone");
        caps.setCapability("device", "iPhone X");
        caps.setCapability("realMobile", "true");
        caps.setCapability("os_version", "11.0");

        driver = new RemoteWebDriver(new URL(URL), caps);
        driver.get("http://demo-store.seleniumacademy.com/");
    }

    @Test
    public void searchProduct() {

        WebElement lookingGlassIcon =
                driver.findElement(By
                        .cssSelector("a.skip-search span.icon"));
        lookingGlassIcon.click();

        // find search box and enter search string
        WebElement searchBox = driver.findElement(By.name("q"));
        searchBox.sendKeys("Phones");

        WebElement searchButton =
                driver.findElement(By.className("search-button"));

        searchButton.click();

        List<WebElement> searchItems = new WebDriverWait(driver, 30)
                .until(ExpectedConditions
                        .presenceOfAllElementsLocatedBy(By
                                .cssSelector("h2.product-name a")));

        assertThat(searchItems.size())
                .isEqualTo(3);
    }

    @AfterTest
    public void tearDown() throws Exception {
        // Close the browser
        driver.quit();
    }
}
```

从你的 IDE 执行测试，它将在 BrowserStack 云中运行。你可以在 BrowserStack 仪表板中监控测试，其中将显示使用的配置能力、每个步骤的状态、控制台日志、网络日志、Appium 日志和执行的视频：

![](img/ddd572c8-832e-40f6-b531-f8025357dfe0.png)

# 摘要

在本章中，我们讨论了企业如何在移动平台上接触其用户的不同方式。我们还了解了使用 Selenium WebDriver 创建的各种软件工具。最后，我们介绍了一个即将推出的软件工具，并修改了我们的测试脚本以与 iOS 和 Android 平台兼容。

在下一章节中，我们将了解如何使用`TestNG`创建参数化和数据驱动的测试。这将帮助我们重用测试并增加测试覆盖率。

# 问题

1.  移动应用有哪些不同类型？

1.  Appium Java 客户端库为测试 iOS 和 Android 应用程序提供了哪些类？

1.  列出通过 USB 端口连接到计算机的 Android 设备的命令是什么？

1.  Appium 服务器默认使用哪个端口？

# 更多信息

你可以查看以下链接以获取有关本章涵盖主题的更多信息：

+   想了解更多关于使用 Appium 的示例，请访问其网站和 GitHub 论坛[`appium.io/`](http://appium.io/)和[`github.com/appium/appium/tree/master/sample-code/java`](https://github.com/appium/appium/tree/master/sample-code/java)
