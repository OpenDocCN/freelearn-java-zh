# 评估

# 第一章

1.  真或假：Selenium 是一个浏览器自动化库。

True.

1.  Selenium 提供了哪些不同类型的定位机制？

不同的定位机制类型是 ID、Name、ClassName、TagName、Link、LinkText、CSS Selector 和 XPATH。

1.  真或假：使用 `getAttribute()` 方法，我们也可以读取 CSS 属性吗？

False. `getCssValue()` 方法用于读取 CSS 属性。

1.  我们可以在 WebElement 上执行哪些操作？

执行的操作是点击、输入（sendKeys）和提交。

1.  我们如何确定复选框是被勾选还是未勾选？

通过使用 `isSelected()` 方法。

# 第二章

1.  WebDriver 成为 W3C 规范的意义是什么？

WebDriver 现在是一个 W3C 规范。这意味着浏览器必须完全遵守世界万维网联盟（简称 W3C）设定的 WebDriver 规范，并且将由浏览器厂商原生支持。HTML5 和 CSS 是其他突出的 W3C 规范。

1.  真或假：WebDriver 是一个接口。

True.

1.  哪些浏览器支持无头测试？

Google Chrome 和 Mozilla Firefox。

1.  我们如何使用 Chrome 测试移动网站？

通过使用移动仿真功能。

# 第三章

1.  哪个版本的 Java Streams API 被引入？

Java 8。

1.  解释 Streams API 的过滤函数。

Java Stream API 提供了一个 `filter()` 方法，可以根据给定的谓词过滤流元素。假设我们想获取页面上所有可见的链接元素，我们可以使用 `filter()` 方法以以下方式返回列表：

```java
List<WebElement> visibleLinks = links.stream()
   .filter(item -> item.isDisplayed())
    .collect(Collectors.toList());
```

1.  Streams API 中的哪种方法会从 `filter()` 函数返回匹配元素的数量？

`count()`。

1.  我们可以使用 `map()` 函数通过属性值：True 或 false 过滤 WebElements 列表：？

False.

# 第四章

1.  我们可以使用哪些不同的格式来输出截图？

`OutputType` 接口支持在 `BASE64`、`BYTES` 和 `FILE` 格式中的截图类型。

1.  我们如何使用 Selenium 切换到另一个浏览器标签？

我们可以使用 `driver.switchTo().window()` 方法切换到另一个浏览器标签。

1.  真或假：`defaultContent()` 方法将切换到之前选定的框架。

False. `defaultContent()` 方法将切换到页面。

1.  Selenium 提供了哪些导航方法？

`Navigate` 接口提供了 `to()`、`back()`、`forward()` 和 `refresh()` 方法。

1.  我们如何使用 Selenium 添加 cookie？

我们可以使用 `driver.manage().addCookie(Cookie cookie)` 方法添加 cookie。

1.  解释隐式等待和显式等待之间的区别。

一旦设置隐式等待，它将在 WebDriver 实例的整个生命周期中可用。当调用 `findElement` 时，它将在设定的持续时间内等待元素。如果元素在设定时间内没有出现在 DOM 中，它将抛出 `NoSuchElementFound` 异常。

另一方面，显式等待用于等待特定条件发生（例如，元素的可见性或不可见性、标题的变化、元素属性的变化、元素变为可编辑或自定义条件）。与隐式等待不同，显式等待将轮询 DOM 以满足条件，而不是等待固定的时间。如果条件在定义的超时之前得到满足，它将退出，否则将抛出异常。我们可以使用`ExpectedConditions`类中的各种预定义条件与显式等待一起使用。

# 第五章

1.  对还是错——拖放操作需要源元素和目标元素。

对。

1.  列出我们可以使用 actions API 执行的键盘方法。

`sendKeys()`、`keyUp()`和`keyDown()`。

1.  哪个 actions API 方法可以帮助执行双击操作？

`doubleClick(WebElement target)`.

1.  使用 actions API，我们如何执行保存选项（即，*Ctrl* + *S*）？

`new Actions(driver) .sendKeys(Keys.chord(Keys.CONTROL, "s")) .perform();`.

1.  我们如何使用 actions API 打开上下文菜单？

通过调用`contextClick()`方法。

# 第六章

1.  你可以使用 WebDriverEventListener 接口来监听 WebDriver 事件：对还是错？

对。

1.  你可以使用 WebDriverEventListener 接口在调用`sendKeys`方法之前自动清除输入字段吗？

我们可以在`beforeChangeValueOf()`事件处理程序中调用`WebElement.clear()`方法。

1.  Selenium 支持可访问性测试：对还是错？

错。Selenium 不支持可访问性测试

# 第七章

1.  对还是错：使用 Selenium，我们可以在远程机器上执行测试吗？

对。

1.  用于在远程机器上运行测试的哪个驱动类？

`RemoteWebDriver`类。

1.  解释`DesiredCapabilities`类。

`DesiredCapabilities`类用于指定测试脚本从 RemoteWebDriver 需要的浏览器能力。例如，我们可以在`DesiredCapabilities`中指定浏览器名称、操作系统和版本，并将其传递给`RemoteWebDriver`。Selenium Standalone Server 将匹配配置的能力与可用的节点，并在匹配的节点上运行测试。

1.  Selenium 测试和 Selenium Standalone Server 之间使用什么协议？

JSON-Wire。

1.  Selenium Standalone Server 使用的默认端口是什么？

端口`4444`。

# 第八章

1.  哪个参数可以用来指定节点可以支持的浏览器实例数量？

`maxInstances`。

1.  解释如何使用 Selenium Grid 来支持跨浏览器测试。

使用 Selenium Grid，我们可以为各种浏览器和操作系统组合设置节点，并在分布式架构中运行测试。根据测试提供的功能，Selenium Grid 选择适当的节点，并在所选节点上执行测试。我们可以根据所需的跨浏览器测试矩阵组合添加所需数量的节点。

1.  使用`RemoteWebDriver`运行 Selenium Grid 测试时，需要指定哪个 URL？

`http://gridHostnameOrIp:4444/wd/hub`。

1.  Selenium Grid Hub 充当负载均衡器：对或错？

对。Selenium Grid Hub 根据节点的可用性在多个节点上分发测试。

# 第九章

1.  如何初始化使用 PageFactory 实现的 PageObject？

`PageFactory.initElements(driver, pageObjectClass)`。

1.  使用哪个类可以实现验证页面是否加载的方法？

`LoadableComponent`。

1.  @FindBy 支持哪些`By class`方法？

ID、Name、ClassName、TagName、Link、PartialLinkText、CSS Selector 和 XPATH。

1.  在使用 PageFactory 时，如果你通过相同的 ID 或 name 属性命名 WebElement 变量，那么你不需要使用`@FindBy`注解：对或错？

对。你可以声明与 id 或 name 属性值相同的 WebElement 变量，PageFactory 将解析它而无需使用`@FindBy`注解。

# 第十章

1.  有哪些不同类型的移动应用？

原生、混合和移动 Web 应用。

1.  Appium Java 客户端库为测试 iOS 和 Android 应用提供了哪些类？

`AndroidDriver`和`IOSDriver`。

1.  列出通过 USB 端口连接到计算机的 Android 设备的命令是什么？

`adb devices`。

1.  Appium 服务器默认使用哪个端口？

端口`4723`。

# 第十一章

1.  解释数据驱动测试。

数据驱动是一种测试自动化框架方法，其中输入测试数据以表格格式或电子表格格式存储，单个测试脚本读取数据中的每一行，这可以是一个独特的测试用例，并执行步骤。这使测试脚本可重用，并通过不同的测试数据组合增加了测试覆盖率。

1.  对或错：Selenium 支持数据驱动测试。

错误。

1.  TestNG 中有哪两种方法可以创建数据驱动测试？

TestNG 提供了两种数据驱动测试方法：套件参数和数据提供者。

1.  解释 TestNG 中的 DataProvider 方法。

TestNG 中的 DataProvider 方法是一个被`@DataProvider`注解特殊标记的方法。它返回一个对象数组。我们可以通过从任何格式（如 CSV 或 Excel）读取表格数据来返回，以使用数据提供者测试测试用例。
