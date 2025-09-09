# 第九章。处理手机和平板

在本章中，我们将介绍以下配方：

+   安装 Spring Mobile

+   检测手机和平板

+   在手机上切换到正常视图

+   自动为手机使用不同的 JSP 视图

+   在手机上使用`.mobi`域名

+   在手机上使用`m.`子域名

+   在手机上使用不同的域名

+   在手机上使用子文件夹路径

# 简介

要构建一个适合移动设备的网站，当前趋势是使用响应式设计，其中页面会适应屏幕宽度。这样，同一页面可以在所有设备上很好地显示：电脑、平板和手机。

本章介绍的一种另一种方法是为移动设备构建一个独立的网站。这需要为网站的每一页构建两个页面（不同的 HTML 和不同的 URL）：一个用于电脑，一个用于手机。当：

+   性能很重要。例如，像[`www.flickr.com/`](https://www.flickr.com/)这样的响应式网站的加载时间在移动设备上会太长，因为桌面版本的图片分辨率很高。一个独立的移动网站可以更容易地优化移动设备上的用户体验。

+   网站的电脑版本已经存在；在这种情况下，通常更简单的是构建一个独立的移动网站。

在本章中，我们将介绍如何使用 Spring Mobile（一个 Spring 项目）为移动设备提供不同的页面。

# 安装 Spring Mobile

在这个配方中，你将学习如何安装 Spring Mobile 并为其他配方准备 Spring 配置类。

## 如何操作…

安装 Spring Mobile 的步骤如下：

1.  在`pom.xml`中添加 Spring Mobile 的 Maven 依赖项：

    ```java
    <dependency>
      <groupId>org.springframework.mobile</groupId>
      <artifactId>spring-mobile-device</artifactId>
      <version>1.1.3.RELEASE</version>
    </dependency>
    ```

1.  使 Spring 配置类继承`WebMvcConfigurerAdapter`：

    ```java
    @Configuration
    @EnableWebMvc
    @ComponentScan(basePackages = {"com.spring_cookbook.controllers"})
    public class AppConfig extends WebMvcConfigurerAdapter {
    ```

1.  覆盖`WebMvcConfigurerAdapter`中的`addInterceptors()`方法：

    ```java
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
    }
    ```

## 它是如何工作的…

在以下配方中，`addInterceptors()`方法将用于注册各种拦截器。

## 更多内容…

关于拦截器的更多信息，请参考第三章中的*在控制器和视图之前和之后执行代码使用拦截器*配方，*使用控制器和视图*。

# 检测手机和平板

在这个配方中，你将学习如何从控制器方法中检测当前 HTTP 请求是否来自桌面电脑、手机或平板。

## 如何操作…

在控制器方法中注册`DeviceResolverHandlerInterceptor`拦截器并使用`DeviceUtils`：

1.  在 Spring 配置类中，声明一个`DeviceResolverHandlerInterceptor`豆：

    ```java
    @Bean
    public DeviceResolverHandlerInterceptor deviceResolverHandlerInterceptor() {
        return new DeviceResolverHandlerInterceptor();
    }
    ```

1.  在`addInterceptors()`方法中将`DeviceResolverHandlerInterceptor`豆注册为拦截器：

    ```java
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(deviceResolverHandlerInterceptor()) ;
    }
    ```

1.  在你的控制器方法中添加一个`HttpServletRequest`参数：

    ```java
    @Controller
    public class UserController {  
      @RequestMapping("/user_list")
      public void userList(HttpServletRequest request) {
    ```

1.  使用`DeviceUtils.getCurrentDevice()`从`HttpServletRequest`生成一个`Device`对象：

    ```java
    Device currentDevice = DeviceUtils.getCurrentDevice(request);
    ```

1.  使用`Device`对象来检测发送请求的设备类型：

    ```java
    if(currentDevice == null) {
      // detection failed
    }
    if(currentDevice.isMobile()) {
      // mobile
    }
    else if(currentDevice.isTablet()) {
      // tablet  
    }
    else if(currentDevice.isNormal()) {
      // desktop computer
    }
    ```

## 它是如何工作的…

`DeviceResolverHandlerInterceptor`拦截器从 HTTP 请求生成`Device`对象，并将其存储在传递给控制器方法的`HttpServletRequest`中。`DeviceUtils.getCurrentDevice()`是一个方便的方法来检索`Device`对象。

然后，您可以选择根据`Device`类型显示不同的 JSP。

为了生成`Device`对象，`DeviceResolverHandlerInterceptor`默认使用`LiteDeviceResolver`，它使用 HTTP 请求的**User-Agent**头。该算法基于 WordPress Mobile Pack 的检测算法。

# 在移动设备上切换到普通视图

移动用户默认获取网站的移动版本，但他/她可能想访问仅在普通版本上显示的内容。Spring Mobile 为此提供了`SitePreference`对象，该对象应替代之前菜谱中使用的`Device`对象。

## 如何操作…

按照以下步骤创建链接，在网站的普通版本和移动版本之间切换：

1.  在 Spring 配置类中，声明一个`DeviceResolverHandlerInterceptor` bean 和一个`SitePreferenceHandlerInterceptor` bean，并将它们注册为拦截器：

    ```java
    @Bean
    public DeviceResolverHandlerInterceptor deviceResolverHandlerInterceptor() {
        return new DeviceResolverHandlerInterceptor();
    }

    @Bean
    public SitePreferenceHandlerInterceptor sitePreferenceHandlerInterceptor() {
        return new SitePreferenceHandlerInterceptor();
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(deviceResolverHandlerInterceptor()) ;
        registry.addInterceptor(sitePreferenceHandlerInterceptor()) ;
    }
    ```

1.  在您的控制器方法中，添加一个`HttpServletRequest`参数：

    ```java
    @Controller
    public class UserController {  
      @RequestMapping("/user_list")
      public void userList(HttpServletRequest request) {
    ```

1.  使用`SitePreferenceUtils.getCurrentSitePreference()`生成一个`SitePreference`对象：

    ```java
    SitePreference sitePreference = SitePreferenceUtils.getCurrentSitePreference(request);
    ```

1.  使用`SitePreference`对象来检测将要显示的页面版本：

    ```java
    if(sitePreference == null || sitePreference.isNormal()) {
      // normal
    }
    if(sitePreference.isMobile()) {
      // mobile
    }
    ```

1.  在视图中，添加到页面两个版本的链接：

    ```java
    Site:
    <a href="?site_preference=normal">Normal</a>
    | 
    <a href="?site_preference=mobile">Mobile</a>
    ```

## 它是如何工作的…

Spring Mobile 自动检测`site_preference`参数（通过`SitePreferenceHandlerInterceptor`拦截器）并相应地调整`SitePreference`值。

默认情况下，`SitePreference`值与`Device`相同。例如，对于*移动*设备，有一个*移动*偏好。当用户点击包含`site_preference`参数的链接时，站点偏好会改变，但设备类型保持不变。例如，对于*移动*设备，有一个*普通*偏好。

## 更多内容…

Spring Mobile 支持*平板*站点偏好。一些网站提供针对平板优化的页面；例如，Google Search 提供了针对平板优化的页面。

# 自动使用不同的 JSP 视图为移动设备

而不是在每个控制器方法中手动选择正确的 JSP，根据请求设备或站点偏好，使用 Spring Mobile 提供的`LiteDeviceDelegatingViewResolver`。

## 如何操作…

在 Spring 配置类中，将任何现有的`ViewResolver` bean 替换为`LiteDeviceDelegatingViewResolver` bean：

```java
@Bean
public LiteDeviceDelegatingViewResolver liteDeviceAwareViewResolver() {
    InternalResourceViewResolver delegate = new InternalResourceViewResolver();
    delegate.setPrefix("/WEB-INF/jsp/");
    delegate.setSuffix(".jsp");
    LiteDeviceDelegatingViewResolver resolver = new LiteDeviceDelegatingViewResolver(delegate);
    resolver.setMobilePrefix("mobile/");
    resolver.setEnableFallback(true);
    return resolver;
}
```

## 它是如何工作的…

对于返回`userList`字符串的控制器，如果站点偏好是*普通*，则将使用`/WEB-INF/userList.jsp` JSP 视图。如果站点偏好是*移动*，则将使用`/WEB-INF/mobile/userList.jsp` JSP 视图。

如果站点偏好是*移动*，并且`/WEB-INF/mobile/userList.jsp`不存在，则将使用`/WEB-INF/userList.jsp`作为后备。这是通过以下行启用的：

```java
resolver.setEnableFallback(true);
```

## 更多内容…

`LiteDeviceDelegatingViewResolver` 支持平板电脑的自定义 JSP 视图：

```java
resolver.setTabletPrefix("tablet/");
```

# 在手机上使用 .mobi 域名

在这个菜谱中，您将学习如何为网站的移动页面使用顶级 `.mobi` 域名。例如：

+   `mysite.com` 为正常网站

+   `mysite.mobi` 为移动版本

最高级域名 `.mobi` 的创建是为了让网站的访客明确地请求其移动版本。例如，[`google.mobi`](http://google.mobi)。Google、Microsoft、Nokia 和 Samsung 最初赞助了它。

## 准备工作

确保在 Spring 配置中声明了 `SitePreferenceHandlerInterceptor` 拦截器。参考本章中的 *在手机上切换到正常视图* 菜单。

## 如何操作…

按照以下步骤为网站的移动版本使用 `.mobi` 域名：

1.  在 Spring 配置中，声明一个使用 `dotMobi()` 方法初始化的 `SiteSwitcherHandlerInterceptor` 实例，并将您的域名作为参数：

    ```java
    @Bean
    public SiteSwitcherHandlerInterceptor siteSwitcherHandlerInterceptor() {
        return SiteSwitcherHandlerInterceptor.dotMobi("mywebsite.com");
    }
    ```

1.  将该实例声明为拦截器：

    ```java
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
      ...
    registry.addInterceptor(siteSwitcherHandlerInterceptor());
    }
    ```

## 它是如何工作的…

在幕后，`SiteSwitcherHandlerInterceptor` 读取当前的 `SitePreference` 值（*正常*、*平板*或*移动*），并在必要时执行重定向到正确的域名。例如，来自移动设备的对 `mywebsite.com` 的 HTTP 请求将被自动重定向到 `mywebsite.mobi`。平板电脑将访问正常网站。

# 在手机上使用 m. 子域名

在这个菜谱中，您将学习如何为网站的移动页面使用 `m.` 子域名。例如：

+   `mysite.com` 为正常网站

+   `m.mysite.com` 为移动版本

`m.` 子域名的某些优点是：

+   无需购买另一个域名（如果您使用 HTTPS，则无需购买另一个 SSL 证书）

+   对用户来说很容易记住

## 准备工作

确保在 Spring 配置中声明了 `SitePreferenceHandlerInterceptor` 拦截器。参考本章中的 *在手机上切换到正常视图* 菜单。

## 如何操作…

按照以下步骤为网站的移动版本使用 `m.` 子域名：

1.  在 Spring 配置中，声明一个使用 `mDot()` 方法初始化的 `SiteSwitcherHandlerInterceptor` 实例，并将您的域名作为参数：

    ```java
    @Bean
    public SiteSwitcherHandlerInterceptor siteSwitcherHandlerInterceptor() {
        return SiteSwitcherHandlerInterceptor.mDot("mywebsite.com");
    }
    ```

1.  将该实例声明为拦截器：

    ```java
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
      ...
    registry.addInterceptor(siteSwitcherHandlerInterceptor());
    }
    ```

## 它是如何工作的…

在幕后，`SiteSwitcherHandlerInterceptor` 读取当前的 `SitePreference` 值（*正常*、*平板*或*移动*），并在必要时执行重定向到正确的域名。例如，来自移动设备的对 `mywebsite.com` 的 HTTP 请求将被自动重定向到 `m.mywebsite.com`。平板电脑将访问正常网站。

# 在手机上使用不同的域名

在这个菜谱中，您将学习如何为网站的移动页面使用不同的域名。例如：

+   `mysite.com` 为正常网站

+   `mymobilesite.com` 为移动版本

## 准备工作

确保在 Spring 配置中声明了`SitePreferenceHandlerInterceptor`拦截器。参考本章中的*在移动设备上切换到正常视图*配方。

## 如何操作…

按照以下步骤为网站的移动版本使用不同的域名：

1.  在 Spring 配置中，声明一个使用`standard()`方法初始化的`SiteSwitcherHandlerInterceptor` bean，并指定你的主域名、移动域名以及`Set-Cookie` HTTP 头字段的值：

    ```java
    @Bean
    public SiteSwitcherHandlerInterceptor siteSwitcherHandlerInterceptor() {
        return SiteSwitcherHandlerInterceptor.standard("mywebsite.com", "mymobilewebsite.com", ".mywebsite.com");
    }
    ```

1.  将该 bean 声明为拦截器：

    ```java
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
      ...
    registry.addInterceptor(siteSwitcherHandlerInterceptor());
    }
    ```

## 它是如何工作的…

在幕后，`SiteSwitcherHandlerInterceptor`读取当前的`SitePreference`值（*normal*、*tablet*或*mobile*）并在必要时重定向到正确的域名。例如，来自移动设备的对`mywebsite.com`的 HTTP 请求将被自动重定向到`mymobilewebsite.com`。平板电脑将访问正常网站。

`Set-Cookie` HTTP 头字段包含`SitePreference`值。该 cookie 允许我们与子域共享该值。在这个配方中，`.mywebsite.com`使得`SitePreference`值对`www.mywebsite.com`等可用。

# 在移动设备上使用子文件夹路径

在这个配方中，你将学习如何使用 URL 中的子文件夹来为你的网站创建移动页面。例如：

+   为正常网站指定`mysite.com`

+   为移动版本指定`mysite.com/mobile`

## 准备工作

确保在 Spring 配置中声明了`SitePreferenceHandlerInterceptor`拦截器。参考本章中的*在移动设备上切换到正常视图*配方。

## 如何操作…

按照以下步骤为网站的移动版本使用子文件夹路径：

1.  在 Spring 配置中，声明一个使用`urlPath()`方法初始化的`SiteSwitcherHandlerInterceptor` bean，并指定子文件夹名称以及必要时使用的 Web 应用程序根路径：

    ```java
    @Bean
    public SiteSwitcherHandlerInterceptor siteSwitcherHandlerInterceptor() {
        return SiteSwitcherHandlerInterceptor.urlPath("/mobile", "spring_webapp");
    }
    ```

1.  将该 bean 声明为拦截器：

    ```java
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
      ...
    registry.addInterceptor(siteSwitcherHandlerInterceptor());
    }
    ```

1.  为带有`mobile`子文件夹的 URL 添加控制器方法：

    ```java
    @RequestMapping("/mobile/user_list")
    public String userListMobile(HttpServletRequest request) {
      ...
      return "user_list";
    }
    ```

## 它是如何工作的…

在幕后，`SiteSwitcherHandlerInterceptor`读取当前的`SitePreference`值（*normal*、*tablet*或*mobile*）并在必要时对 URL 进行重定向以添加或删除`mobile`子文件夹。例如，来自移动设备的对`mywebsite.com`的 HTTP 请求将被自动重定向到`mywebsite.com/mobile`。平板电脑将访问正常网站。
