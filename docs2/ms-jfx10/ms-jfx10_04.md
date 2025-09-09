# 探索 WebDriver 的功能

到目前为止，我们已经探讨了用户可以使用 WebDriver 在网页上执行的各种基本和高级交互。在本章中，我们将讨论 WebDriver 的不同功能和特性，这些功能和特性使测试脚本开发者能够更好地控制 WebDriver，从而更好地控制正在测试的 Web 应用程序。本章将要涵盖的功能如下：

+   截图

+   定位目标窗口和 iFrames

+   探索导航

+   等待 Web 元素加载

+   处理 Cookie

让我们立即开始，不要有任何延迟。

# 截图

在 WebDriver 库中，对网页进行截图是一个非常有用的功能。当测试用例失败，你想要查看测试用例失败时应用程序的状态时，这非常有用。WebDriver 库中的 `TakesScreenShot` 接口由所有不同的 WebDriver 变体实现，例如 Firefox Driver、Internet Explorer Driver、Chrome Driver 等。

`TakesScreenShot` 功能在所有浏览器中默认启用。由于这是一个只读功能，用户无法切换它。在我们查看使用此功能的代码示例之前，我们应该看看 `TakesScreenShot` 接口的一个重要方法——`getScreenshotAs().`

`getScreenshotAs()` 的 API 语法如下：

```java
public X getScreenshotAs(OutputType target)           
```

在这里，`OutputType` 是 WebDriver 库的另一个接口。我们可以要求 WebDriver 以三种不同的格式输出截图：`BASE64`、`BYTES`（原始数据）和 `FILE`。如果你选择 `FILE` 格式，它将数据写入一个 `.png` 文件，一旦 JVM 被终止，该文件将被删除。因此，你应该始终将该文件复制到安全位置，以便以后参考。

返回类型是特定于所选 `OutputType` 的特定输出。例如，选择 `OutputType.BYTES` 将返回一个 `byte` 数组，而选择 `OutputType.FILE` 将返回一个文件对象。

根据使用的浏览器，输出截图将是以下之一，按优先级排序：

+   整个页面

+   当前窗口

+   当前框架的可视部分

+   包含浏览器的整个显示屏幕的截图

例如，如果你正在使用 Firefox Driver，`getScreenshotAs()` 会截取整个页面的截图，但 Chrome Driver 只返回当前框架的可视部分。

是时候看看以下代码示例了：

```java
@BeforeMethod
public void setup() throws IOException {
    System.setProperty("webdriver.chrome.driver",
            "./src/test/resources/drivers/chromedriver");
    driver = new ChromeDriver();
    driver.get("http://demo-store.seleniumacademy.com/");

    File scrFile  = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
    FileUtils.copyFile(scrFile, new File("./target/screenshot.png")); 
}
```

在前面的代码中，我们使用了 `getScreenshotAs()` 方法来截取网页截图并将其保存到文件格式。我们可以从目标文件夹中打开保存的图像并检查它。

# 定位目标窗口和 Frames

WebDriver 允许开发者切换到应用程序中使用的多个子窗口、浏览器标签页和框架。例如，当你在一个银行网络应用上点击互联网银行链接时，它将在一个单独的窗口或标签页中打开互联网银行应用。此时，你可能想切换回原始窗口来处理一些事件。同样，你可能需要处理一个网页上分为两个框架的网络应用。左侧的框架可能包含导航项，而右侧的框架根据左侧框架中选择的项显示相应的网页。使用 WebDriver，你可以开发出能够轻松处理这种复杂情况的测试用例。

`WebDriver.TargetLocator` 接口用于定位给定的框架或窗口。在本节中，我们将了解 WebDriver 如何处理在同一个窗口中切换浏览器窗口和两个框架。

# 在窗口之间切换

首先，我们将看到一个处理多个窗口的代码示例。对于本章，本书提供了一个名为 `Window.html` 的 HTML 文件。这是一个非常基本的网页，它链接到谷歌的搜索页面。当你点击链接时，谷歌的搜索页面将在不同的窗口中打开。每次你使用 WebDriver 在浏览器窗口中打开一个网页时，WebDriver 都会为它分配一个窗口句柄。WebDriver 使用窗口句柄来识别窗口。此时，在 WebDriver 中已注册了两个窗口句柄。现在，在屏幕上，你可以看到谷歌的搜索页面在前台并且具有焦点。此时，如果你想切换到第一个浏览器窗口，你可以使用 WebDriver 的 `switchTo()` 方法来实现。

`TargetLocator` 的 API 语法如下：

```java
WebDriver.TargetLocator switchTo()
```

此方法返回 `WebDriver.TargetLocator` 实例，其中你可以告诉 WebDriver 是否要在浏览器窗口或框架之间切换。让我们看看 WebDriver 如何处理这个问题：

```java
public class WindowHandlingTest {

    WebDriver driver;

    @BeforeMethod
    public void setup() throws IOException {
        System.setProperty("webdriver.chrome.driver",
                "./src/test/resources/drivers/chromedriver");
        driver = new ChromeDriver();
        driver.get("http://guidebook.seleniumacademy.com/Window.html");
    }

    @Test
    public void handleWindow() {

        String firstWindow = driver.getWindowHandle();
        System.out.println("First Window Handle is: " + firstWindow);

        WebElement link = driver.findElement(By.linkText("Google Search"));
        link.click();

        String secondWindow = driver.getWindowHandle();
        System.out.println("Second Window Handle is: " + secondWindow);
        System.out.println("Number of Window Handles so for: "
                + driver.getWindowHandles().size());

 driver.switchTo().window(firstWindow);
    }

    @AfterMethod
    public void tearDown() {
        driver.quit();
    }
}
```

观察以下代码中的这一行：

```java
String firstWindow = driver.getWindowHandle();
```

在这里，驱动程序返回窗口的分配标识符。现在，在我们切换到不同的窗口之前，最好存储这个值，以便如果我们想切换回这个窗口，我们可以使用这个句柄或标识符。要检索到目前为止与你的驱动程序注册的所有窗口句柄，你可以使用以下方法：

```java
driver.getWindowHandles()
```

这将返回到目前为止在驱动程序会话中打开的所有浏览器窗口句柄的标识符集合。现在，在我们的例子中，在我们打开谷歌的搜索页面后，对应的窗口将显示在前面并具有焦点。如果你想回到第一个窗口，你必须使用以下代码：

```java
driver.switchTo().window(firstWindow);
```

这将使第一个窗口获得焦点。

# 切换框架

现在我们来看看我们如何处理网页框架之间的切换。在这本书提供的 HTML 文件中，您将看到一个名为 `Frames.html` 的文件。如果您打开它，您将看到两个不同的框架中加载了两个 HTML 文件。让我们看看我们如何在这两个框架之间切换，并输入每个框架中可用的文本框：

```java
public class FrameHandlingTest {
    WebDriver driver;

    @BeforeMethod
    public void setup() throws IOException {
        System.setProperty("webdriver.chrome.driver",
                "./src/test/resources/drivers/chromedriver");
        driver = new ChromeDriver();
        driver.get("http://guidebook.seleniumacademy.com/Frames.html");
    }

    @Test
    public void switchBetweenFrames() {

        // First Frame
        driver.switchTo().frame(0);
        WebElement firstField = driver.findElement(By.name("1"));
        firstField.sendKeys("I'm Frame One");
 driver.switchTo().defaultContent();

        // Second Frame
        driver.switchTo().frame(1);
        WebElement secondField = driver.findElement(By.name("2"));
        secondField.sendKeys("I'm Frame Two");
    }

    @AfterMethod
    public void tearDown() {
        driver.quit();
    }
}
```

在前面的代码中，我们使用了 `switchTo().frame` 而不是 `switchTo().window`，因为我们正在跨框架移动。

`frame` 的 API 语法如下：

```java
WebDriver frame(int index)
```

此方法获取您想要切换到的框架的索引。如果您的网页有三个框架，WebDriver 将它们索引为 0、1 和 2，其中零索引分配给在 DOM 中遇到的第一个框架。同样，您可以通过使用之前重载的方法使用它们的名称在框架之间切换。API 语法如下：

```java
WebDriver frame(String frameNameOrframeID)
```

您可以传递框架的名称或其 ID。使用此方法，如果您不确定目标框架的索引，您也可以切换到框架。其他重载方法如下：

```java
WebDriver frame(WebElement frameElement)
```

输入参数是框架的 `WebElement`。让我们考虑我们的代码示例：首先，我们已经切换到了我们的第一个框架并输入了文本字段。然后，我们并没有直接切换到第二个框架，而是来到了主或默认内容，然后切换到第二个框架。该代码如下：

```java
driver.switchTo().defaultContent();               
```

这非常重要。如果您不这样做，而试图在仍然处于第一个框架的情况下切换到第二个框架，您的 WebDriver 将会抱怨说它找不到索引为 1 的框架。这是因为 WebDriver 在第一个框架的上下文中搜索第二个框架，这显然是不可用的。因此，您必须首先来到顶级容器，然后切换到您感兴趣的框架。

在切换到默认内容后，您现在可以使用以下代码切换到第二个框架：

```java
driver.switchTo().frame(1);
```

因此，您可以在框架之间切换并执行相应的 WebDriver 操作。

# 处理警报

除了在窗口和框架之间切换之外，您可能还需要处理 Web 应用程序中的各种模式对话框。为此，WebDriver 提供了一个处理警报对话框的 API。该 API 如下所示：

```java
Alert alert()
```

上述方法将切换到网页上当前活动的模式对话框。这将返回一个 `Alert` 实例，您可以在该对话框上采取适当的操作。如果没有当前对话框，并且您调用此 API，它将抛出 `NoAlertPresentException`。

`Alert` 接口包含一系列 API 来执行不同的操作。以下列表逐一讨论它们：

+   `void accept()`: 这相当于对话框上的 **OK** 按钮操作。当在对话框上执行 `accept()` 操作时，将调用相应的 **OK** 按钮操作。

+   `void dismiss()`: 这相当于点击 **CANCEL** 操作按钮。

+   `java.lang.String getText()`: 这将返回对话框上显示的文本。如果你想要评估模态对话框上的文本，可以使用此方法。

+   `void sendKeys(java.lang.String keysToSend)`: 如果警报有此功能，这将允许开发者输入一些文本到警报中。

# 探索 Navigate

如我们所知，WebDriver 以原生方式与单个浏览器进行通信。这种方式不仅使它能够更好地控制网页，还能控制浏览器本身。**Navigate** 是 WebDriver 的一个功能，允许测试脚本开发者与浏览器的后退、前进和刷新控件进行交互。作为网络应用的用户，我们经常使用浏览器的前进和后退控件在单个应用或有时是多个应用之间导航。作为测试脚本开发者，你可能希望开发出在点击浏览器导航按钮时观察应用行为的测试，特别是后退按钮。例如，如果你在一个银行应用中使用导航按钮，会话应该过期，用户应该被登出。因此，使用 WebDriver 的导航功能，你可以模拟这些操作。

用于此目的的方法是 `navigate()`。以下是它的 API 语法：

```java
WebDriver.Navigation navigate()
```

显然，此方法没有输入参数，但返回类型是 `WebDriver.Navigation` 接口，它包含所有帮助你在浏览器历史记录中导航的浏览器导航选项。

现在让我们看一个代码示例，然后分析代码：

```java
@Test
public void searchProduct() {
    driver.navigate().to("http://demo-store.seleniumacademy.com/");

    // find search box and enter search string
    WebElement searchBox = driver.findElement(By.name("q"));

    searchBox.sendKeys("Phones");

    WebElement searchButton =
            driver.findElement(By.className("search-button"));

    searchButton.click();

    assertThat(driver.getTitle())
            .isEqualTo("Search results for: 'Phones'");

    driver.navigate().back();
    driver.navigate().forward();
    driver.navigate().refresh();
}
```

上述代码打开演示应用的首页，首先搜索 `Phone`；然后，在搜索结果加载后。现在，我们在浏览器中创建了一个导航历史记录，它使用 WebDriver 导航返回浏览器历史记录，然后前进并刷新页面。

让我们分析前面代码中使用的导航方法。最初加载演示应用首页的代码行使用了 `Navigation` 类的 `to()` 方法，如下所示：

```java
driver.navigate().to("http://demo-store.seleniumacademy.com/");
```

在这里，`driver.navigate()` 方法返回 `WebDriver.Navigation` 接口，在该接口上使用 `to()` 方法导航到网页 URL。

API 语法如下：

```java
void to(java.lang.String url)
```

此方法的输入参数是必须加载到浏览器中的 `url` 字符串。此方法将通过使用 `HTTP GET` 操作在浏览器中加载页面，并且直到页面完全加载，它会阻塞其他所有操作。此方法与 `driver.get(String url)` 方法相同。

`WebDriver.Navigation` 接口还提供了一个重载的 `to()` 方法，以便更容易传递 URL。它的 API 语法如下：

```java
void to(java.net.URL url)
```

接下来，在代码示例中，我们搜索了 `Phone`。然后，我们尝试使用 Navigation 的 `back()` 方法来模拟浏览器的前进按钮，使用以下代码行：

```java
driver.navigate().back();
```

这将使浏览器跳转到主页。此方法的 API 语法相当简单；如下所示：

```java
void back()
```

此方法也不接收任何输入，也不返回任何内容，但它将浏览器向后移动一个历史级别。

然后，导航中的下一个方法是`forward()`方法，它与`back()`方法非常相似，但它将浏览器带到相反方向的一个级别。在先前的代码示例中，以下方法被调用：

```java
 driver.navigate().forward();
```

该方法的 API 语法如下：

```java
void forward()
```

此方法不接收任何输入，也不返回任何内容，但它将浏览器向前移动一个历史级别。

代码示例中的最后一行使用了 WebDriver 的导航`refresh()`方法：

```java
driver.navigate().refresh();
```

此方法将重新加载当前 URL 以模拟浏览器的*刷新*（*F5*键）操作。API 语法如下：

```java
void refresh()
```

如你所见，语法与`back()`和`forward()`方法非常相似，并且此方法将重新加载当前 URL。因此，这些都是 WebDriver 为开发者提供的用于模拟一些浏览器操作的各种方法。

# 等待 Web 元素加载

如果你之前有 UI 自动化经验，我敢肯定你遇到过这样的情况：你的测试脚本因为网页仍在加载而无法在网页上找到元素。这可能是由于各种原因造成的。一个经典的例子是当应用服务器或 Web 服务器由于资源限制而响应页面太慢时；另一种情况可能是当你在一个非常慢的网络中访问页面时。原因可能是当你的测试脚本尝试找到它时，网页上的元素尚未加载。这就是你必须计算和配置测试脚本等待 Web 元素在网页上加载的平均等待时间的地方。

WebDriver 为测试脚本开发者提供了一个非常实用的功能来管理等待时间。"等待时间"是指你的驱动程序在放弃并抛出`NoSuchElementException`之前，将等待 Web 元素加载的时间。记住，在第一章*介绍 WebDriver 和 Web 元素*中，我们讨论了`findElement(By by)`方法，当它无法找到目标 Web 元素时会抛出`NoSuchElementException`。

你可以通过两种方式让 WebDriver 等待 WebElement。它们是**隐式等待时间**和**显式等待时间**。隐式超时对所有 Web 元素都是通用的，并且与它们关联着一个全局超时周期，但显式超时可以配置为单个 Web 元素。让我们在这里讨论每一个。

# 隐式等待时间

当你想为正在测试的应用程序配置 WebDriver 的整体等待时间时，会使用隐式等待时间。想象一下，你已经在本地服务器和远程服务器上托管了一个 Web 应用程序。显然，托管在本地服务器的网页加载时间会小于托管在远程服务器的相同页面的时间，这是由于网络延迟造成的。现在，如果你想要针对每个服务器执行你的测试用例，你可能需要相应地配置等待时间，以确保你的测试用例不会花费太多时间等待页面，或者几乎不花时间，从而导致超时。为了处理这类等待时间问题，WebDriver 提供了一个选项，可以通过`manage()`方法为驱动器执行的所有操作设置隐式等待时间。

让我们看看一个隐式等待时间的代码示例：

```java
driver = new ChromeDriver();
driver.navigate().to("http://demo-store.seleniumacademy.com/");
driver.manage().timeouts().implicitlyWait(10, TimeUnit.SECONDS);
```

让我们分析以下高亮的代码行：

```java
driver.manage().timeouts().implicitlyWait(10, TimeUnit.SECONDS);
```

在这里，`driver.manage().timeouts()`返回`WebDriver.Timeouts`接口，该接口声明了一个名为`implicitlyWait`的方法，在这里你指定当在网页上搜索一个 WebElements 时，如果它不是立即出现，驱动器应该等待多长时间。WebDriver 会定期在网页上轮询 WebElements，直到之前方法指定的最大等待时间结束。在前面的代码中，10 秒是驱动器将等待任何 WebElements 在浏览器上加载的最大时间。如果在这个时间段内加载，WebDriver 将继续执行其余代码；否则，它将抛出`NoSuchElementException`。

当你想指定一个最大等待时间时，请使用此方法，这在你的 Web 应用程序中的大多数 WebElements 中通常是常见的。影响你页面性能的各种因素包括网络带宽、服务器配置等等。基于这些条件，作为你的 WebDriver 测试用例的开发者，你必须确定一个最大隐式等待时间的值，以确保你的测试用例不会执行得太久，同时也不会频繁超时。

# 显式等待时间

隐式超时适用于网页上的所有 WebElements。但是，如果你有一个特定的 WebElements 在你的应用程序中，你希望等待非常长的时间，这种方法可能不起作用。将隐式等待时间设置为这个非常长的时间段将会延迟整个测试套件的执行。因此，你必须为特定的情况，比如这个 WebElements，做出例外。为了处理这类场景，WebDriver 为 WebElements 提供了一个显式等待时间。

那么，让我们看看如何使用以下代码使用 WebDriver 等待特定的 WebElements：

```java
WebElement searchBox = (new WebDriverWait(driver, 20))
 .until((ExpectedCondition<WebElement>) d -> d.findElement(By.name("q")));
```

突出的代码是我们创建了对特定 WebElement 的条件等待的地方。可以使用`ExpectedCondition`接口将条件等待应用于 WebElement。在这里，WebDriver 将等待最多 20 秒以等待这个特定的 WebElement。对于这个 WebElement，不会应用隐式超时。如果 WebElement 在 20 秒的最大等待时间内没有加载，正如我们所知，驱动器会抛出`NoSuchElementException`异常。因此，你可以通过使用这个方便的显式等待时间来专门覆盖你认为将花费更多时间的 WebElements 的隐式等待时间。

# 处理 cookie

假设你正在自动化演示应用程序。你可能想要自动化的场景有很多，例如搜索产品、将产品添加到购物车、结账、退货等。对于所有这些操作，一个共同点是需要在每个测试用例中登录到演示应用程序。因此，在你的每个测试用例中登录会增加整体测试执行时间。为了减少测试用例的执行时间，你实际上可以跳过每个测试用例的登录。这可以通过一次性登录并将该域的所有 cookie 写入文件来实现。从下一次登录开始，你实际上可以从文件中加载 cookie 并将其添加到驱动器中。

要获取为网页加载的所有 cookie，WebDriver 提供了以下方法：

```java
driver.manage().getCookies()
```

这将返回当前会话中网页存储的所有 cookie。每个 cookie 都与一个名称、值、域名、路径、过期时间和是否安全的状态相关联。服务器解析客户端 cookie 时会解析所有这些值。现在，我们将为每个 cookie 存储所有这些信息到一个文件中，以便我们的单个测试用例从这个文件中读取并加载这些信息到驱动器中。因此，你可以跳过登录，因为一旦你的驱动器会话中有了这些信息，应用程序服务器就会将你的浏览器会话视为已认证，并直接带你到请求的 URL。以下是一个快速存储 cookie 信息的代码示例：

```java
public class StoreCookieInfo {
    WebDriver driver;

    @BeforeMethod
    public void setup() throws IOException {
        System.setProperty("webdriver.chrome.driver",
                "./src/test/resources/drivers/chromedriver");
        driver = new ChromeDriver();
        driver.get("http://demo-store.seleniumacademy.com/customer/account/login/");
    }

    @Test
    public void storeCookies() {
        driver.findElement(By.id("email")).sendKeys("user@seleniumacademy.com");
        driver.findElement(By.id("pass")).sendKeys("tester");
        driver.findElement(By.id("send2")).submit();

        File dataFile = new File("./target/browser.data");
        try {
            dataFile.delete();
            dataFile.createNewFile();
            FileWriter fos = new FileWriter(dataFile);
            BufferedWriter bos = new BufferedWriter(fos);
 for (Cookie ck : driver.manage().getCookies()) {
 bos.write((ck.getName() + ";" + ck.getValue() + ";" + ck.
 getDomain()
 + ";" + ck.getPath() + ";" + ck.getExpiry() + ";" + ck.
 isSecure()));
                bos.newLine();
            }
            bos.flush();
            bos.close();
            fos.close();
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }

    @AfterMethod
    public void tearDown() {
        driver.quit();
    }
}
```

从现在开始，对于每一个测试用例或一组测试用例，需要从`browser.data`文件中加载 cookie 信息，并使用以下方法将其添加到驱动器中：

```java
driver.manage().addCookie(ck);
```

在你将此信息添加到浏览器会话并转到仪表板页面后，它将自动重定向到你主页，而无需请求登录，从而避免了每次测试用例都需要登录。将所有之前的 cookie 添加到驱动器的代码如下：

```java
public class LoadCookieInfo {
    WebDriver driver;

    @BeforeMethod
    public void setup() throws IOException {
        System.setProperty("webdriver.chrome.driver",
                "./src/test/resources/drivers/chromedriver");
        driver = new ChromeDriver();
       driver.get("http://demo-store.seleniumacademy.com");
    }

    @Test
    public void loadCookies() {
        try {
            File dataFile = new File("./target/browser.data");
            FileReader fr = new FileReader(dataFile);
            BufferedReader br = new BufferedReader(fr);
            String line;
            while ((line = br.readLine()) != null) {
                StringTokenizer str = new StringTokenizer(line, ";");
                while (str.hasMoreTokens()) {
                    String name = str.nextToken();
                    String value = str.nextToken();
                    String domain = str.nextToken();
                    String path = str.nextToken();
                    Date expiry = null;
                    String dt;
                    if (!(dt = str.nextToken()).equals("null")) {
                        SimpleDateFormat formatter =
                                new SimpleDateFormat("E MMM d HH:mm:ss z yyyy");
                        expiry = formatter.parse(dt);
                    }

                    boolean isSecure = new Boolean(str.nextToken()).
                            booleanValue();
 Cookie ck = new Cookie(name, value, domain, path, expiry, isSecure);
                    driver.manage().addCookie(ck);
                }
            }

            driver.get("http://demo-store.seleniumacademy.com/customer/account/index/");
            assertThat(driver.findElement(By.cssSelector("div.page-title")).getText())
                    .isEqualTo("MY DASHBOARD");

        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }

    @AfterMethod
    public void tearDown() {
        driver.quit();
    }
}
```

因此，我们可以直接进入主页而无需再次登录。正如你所见，在创建驱动器实例后，我们有以下行：

```java
driver.get("http://demo-store.seleniumacademy.com");
```

理想情况下，此行应该在我们将 cookies 设置到驱动程序之后可见。但它在顶部的原因是 WebDriver 不允许你直接将 cookies 设置到这个会话中，因为它将这些 cookies 视为来自不同域的。尝试删除上一行代码并执行它，你就会看到错误。所以，最初，你将尝试访问主页，将驱动程序的域值设置为应用服务器域，并加载所有 cookies。当你执行此代码时，最初你会看到应用的主页。

因此，通过使用 WebDriver 的 cookies 功能，你可以避免在服务器上输入用户名和密码，并且为每次测试再次验证它们，从而节省大量时间。

# 摘要

在本章中，我们讨论了 WebDriver 的各种功能，例如捕获屏幕截图和处理`Windows`和`Frames`。我们还讨论了同步的隐式和显式等待条件，并使用了导航和 cookies API。使用这些功能将帮助你更有效地测试目标 Web 应用，通过设计更创新的测试框架和测试用例。在下一章中，我们将探讨**Actions** API，以使用键盘和鼠标事件执行用户交互。

# 问题

1.  我们可以使用哪些不同的格式来输出屏幕截图？

1.  我们如何使用 Selenium 切换到另一个浏览器标签页？

1.  对或错：`defaultContent()`方法将切换到之前选定的框架。

1.  Selenium 提供了哪些导航方法？

1.  我们如何使用 Selenium 添加一个 cookie？

1.  解释隐式等待和显式等待之间的区别。

# 更多信息

你可以查看以下链接，获取本章涵盖主题的更多信息：

+   你可以在[`seleniumhq.github.io/selenium/docs/api/java/org/openqa/selenium/support/ui/ExpectedConditions.html`](https://seleniumhq.github.io/selenium/docs/api/java/org/openqa/selenium/support/ui/ExpectedConditions.html)了解更多关于如何在显式等待时使用一组预定义的预期条件。

+   你可以在 Unmesh Gundecha 所著的《Selenium 测试工具烹饪书》（第 2 版），Packt Publications 出版的第四章*“与 Selenium API 一起工作”*和第五章*“同步测试”*中了解更多关于 WebDriver 功能的信息。
