# 第十九章：文件上传和错误处理

在本章中，我们将使用户能够上传个人资料图片。我们还将看到如何在 Spring MVC 中处理错误。

# 上传文件

我们现在将使用户能够上传个人资料图片。这将在以后的个人资料页面上可用，但现在，我们将简化事情，并在`profile/uploadPage.html`目录下的模板目录中创建一个新页面：

```java
<!DOCTYPE html>
<html 

      layout:decorator="layout/default">
<head lang="en">
    <title>Profile Picture Upload</title>
</head>
<body>
<div class="row" layout:fragment="content">

    <h2 class="indigo-text center">Upload</h2>

    <form th:action="@{/upload}" method="post" enctype="multipart/form-data" class="col m8 s12 offset-m2">

        <div class="input-field col s6">
            <input type="file" id="file" name="file"/>
        </div>

        <div class="col s6 center">
            <button class="btn indigo waves-effect waves-light" type="submit" name="save" th:text="#{submit}">Submit
                <i class="mdi-content-send right"></i>
            </button>
        </div>
    </form>
</div>
</body>
</html>
```

除了表单上的`enctype`属性外，没有什么可看的。文件将通过`POST`方法发送到`upload` URL。我们现在将在`profile`包中的`ProfileController`旁边创建相应的控制器：

```java
package masterSpringMvc.profile;

import org.apache.tomcat.util.http.fileupload.IOUtils;
import org.springframework.core.io.FileSystemResource;
import org.springframework.core.io.Resource;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.multipart.MultipartFile;

import java.io.*;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

@Controller
public class PictureUploadController {
    public static final Resource PICTURES_DIR = new FileSystemResource("./pictures");

    @RequestMapping("upload")
    public String uploadPage() {
        return "profile/uploadPage";
    }

    @RequestMapping(value = "/upload", method = RequestMethod.POST)
    public String onUpload(MultipartFile file) throws IOException {
        String filename = file.getOriginalFilename();
        File tempFile = File.createTempFile("pic", getFileExtension(filename), PICTURES_DIR.getFile());

        try (InputStream in = file.getInputStream();
             OutputStream out = new FileOutputStream(tempFile)) {
            IOUtils.copy(in, out);
        }

        return "profile/uploadPage";
    }

    private static String getFileExtension(String name) {
        return name.substring(name.lastIndexOf("."));
    }
}
```

这段代码将做的第一件事是在`pictures`目录中创建一个临时文件，该目录位于项目的根目录内；因此，请确保它存在。在 Java 中，临时文件只是一个方便的方法，用于在文件系统上获取唯一的文件标识符。用户可以选择删除它。

在项目的根目录下创建一个 pictures 目录，并添加一个名为`.gitkeep`的空文件，以确保您可以在 Git 中提交它。

### 提示

**Git 中的空目录**

Git 是基于文件的，不可能提交一个空目录。一个常见的解决方法是在目录中提交一个空文件，比如`.gitkeep`，以强制 Git 将其纳入版本控制。

用户上传的文件将作为`MultipartFile`接口注入到我们的控制器中。该接口提供了几种方法来获取文件的名称、大小和内容。

这里特别感兴趣的方法是`getInputStream()`。我们确实将复制这个流到一个`fileOutputStream`方法，感谢`IOUtils.copy`方法。将输入流写入输出流的代码非常无聊，所以在类路径中有 Apache Utils 很方便（它是`tomcat-embedded-core.jar`文件的一部分）。

我们大量使用了相当酷的 Spring 和 Java 7 NIO 功能：

+   字符串的资源类是一个实用类，表示可以以不同方式找到的资源的抽象

+   `try…with`块将自动关闭我们的流，即使出现异常，删除了编写`finally`块的样板

通过上述代码，用户上传的任何文件都将被复制到`pictures`目录中。

Spring Boot 中有一些可用的属性来自定义文件上传。看一下`MultipartProperties`类。

最有趣的是：

+   `multipart.maxFileSize`：这定义了上传文件的最大文件大小。尝试上传更大的文件将导致`MultipartException`类。默认值为`1Mb`。

+   `multipart.maxRequestSize`：这定义了多部分请求的最大大小。默认值为`10Mb`。

默认值对我们的应用程序已经足够好了。经过几次上传后，我们的图片目录将如下所示：

![上传文件](img/image00952.jpeg)

等等！有人上传了一个 ZIP 文件！我简直不敢相信。我们最好在我们的控制器中添加一些检查，以确保上传的文件是真实的图片：

```java
package masterSpringMvc.profile;

import org.apache.tomcat.util.http.fileupload.IOUtils;
import org.springframework.core.io.FileSystemResource;
import org.springframework.core.io.Resource;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import java.io.*;

@Controller
public class PictureUploadController {
    public static final Resource PICTURES_DIR = new FileSystemResource("./pictures");

    @RequestMapping("upload")
    public String uploadPage() {
        return "profile/uploadPage";
    }

    @RequestMapping(value = "/upload", method = RequestMethod.POST)
    public String onUpload(MultipartFile file, RedirectAttributes redirectAttrs) throws IOException {

 if (file.isEmpty() || !isImage(file)) {
 redirectAttrs.addFlashAttribute("error", "Incorrect file. Please upload a picture.");
 return "redirect:/upload";
 }

        copyFileToPictures(file);

        return "profile/uploadPage";
    }

    private Resource copyFileToPictures(MultipartFile file) throws IOException {
        String fileExtension = getFileExtension(file.getOriginalFilename());
        File tempFile = File.createTempFile("pic", fileExtension, PICTURES_DIR.getFile());
        try (InputStream in = file.getInputStream();
             OutputStream out = new FileOutputStream(tempFile)) {

            IOUtils.copy(in, out);
        }
        return new FileSystemResource(tempFile);
    }

    private boolean isImage(MultipartFile file) {
 return file.getContentType().startsWith("image");
 }

    private static String getFileExtension(String name) {
        return name.substring(name.lastIndexOf("."));
    }
}
```

很简单！`getContentType()`方法返回文件的**多用途互联网邮件扩展**（**MIME**）类型。它将是`image/png`，`image/jpg`等等。因此，我们只需检查 MIME 类型是否以"image"开头。

我们在表单中添加了一个错误消息，因此我们应该在我们的网页中添加一些内容来显示它。将以下代码放在`uploadPage`标题下方：

```java
<div class="col s12 center red-text" th:text="${error}" th:if="${error}">
    Error during upload
</div>
```

下次您尝试上传 ZIP 文件时，将会收到错误！如下截图所示：

![上传文件](img/image00953.jpeg)

## 将图像写入响应

上传的图片不是从静态目录中提供的。我们需要采取特殊措施来在我们的网页中显示它们。

让我们在我们的上传页面上方添加以下行，就在表单上方：

```java
<div class="col m8 s12 offset-m2">
    <img th:src="img/uploadedPicture}" width="100" height="100"/>
</div>
```

这将尝试从我们的控制器获取图像。让我们在`PictureUploadController`类中添加相应的方法：

```java
@RequestMapping(value = "/uploadedPicture")
public void getUploadedPicture(HttpServletResponse response) throws IOException {
    ClassPathResource classPathResource = new ClassPathResource("/images/anonymous.png");
    response.setHeader("Content-Type", URLConnection.guessContentTypeFromName(classPathResource.getFilename()));
    IOUtils.copy(classPathResource.getInputStream(), response.getOutputStream());
}
```

这段代码将直接将`src/main/resources/images/anonymous.png`目录中找到的图像写入响应！多么令人兴奋！

如果我们再次转到我们的页面，我们会看到以下图片：

![将图像写入响应](img/image00954.jpeg)

### 提示

我在 iconmonstr（[`iconmonstr.com/user-icon`](http://iconmonstr.com/user-icon)）上找到了匿名用户图片，并将其下载为 128 x 128 的 PNG 文件。

## 管理上传属性

在这一点上，一个好的做法是允许通过`application.properties`文件配置上传目录和匿名用户图片的路径。

让我们在新创建的`config`包内创建一个`PicturesUploadProperties`类：

```java
package masterSpringMvc.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.core.io.DefaultResourceLoader;
import org.springframework.core.io.Resource;

import java.io.IOException;
@ConfigurationProperties(prefix = "upload.pictures")
public class PicturesUploadProperties {
    private Resource uploadPath;
    private Resource anonymousPicture;

    public Resource getAnonymousPicture() {
        return anonymousPicture;
    }

    public void setAnonymousPicture(String anonymousPicture) {
        this.anonymousPicture = new DefaultResourceLoader().getResource(anonymousPicture);
    }

    public Resource getUploadPath() {
        return uploadPath;
    }

    public void setUploadPath(String uploadPath) {
        this.uploadPath = new DefaultResourceLoader().getResource(uploadPath);
    }
}
```

在这个类中，我们使用了 Spring Boot 的`ConfigurationProperties`。这将告诉 Spring Boot 以类型安全的方式自动映射类路径中发现的属性（默认情况下，在`application.properties`文件中）。

请注意，我们定义了以'String'作为参数的 setter，但可以自由地让 getter 返回任何类型是最有用的。

现在我们需要将`PicturesUploadProperties`类添加到我们的配置中：

```java
@SpringBootApplication
@EnableConfigurationProperties({PictureUploadProperties.class})
public class MasterSpringMvc4Application extends WebMvcConfigurerAdapter {
  // code omitted
}
```

现在我们可以在`application.properties`文件中添加属性值：

```java
upload.pictures.uploadPath=file:./pictures
upload.pictures.anonymousPicture=classpath:/images/anonymous.png
```

因为我们使用了 Spring 的`DefaultResourceLoader`类，我们可以使用诸如`file:`或`classpath:`之类的前缀来指定我们的资源可以被找到的位置。

这相当于创建一个`FileSystemResource`类或`ClassPathResource`类。

这种方法还有一个优点，就是可以对代码进行文档化。我们可以很容易地看到图片目录将在应用程序根目录中找到，而匿名图片将在类路径中找到。

就是这样。我们现在可以在我们的控制器中使用我们的属性。以下是`PictureUploadController`类的相关部分：

```java
package masterSpringMvc.profile;

import masterSpringMvc.config.PictureUploadProperties;
import org.apache.tomcat.util.http.fileupload.IOUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.Resource;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import javax.servlet.http.HttpServletResponse;
import java.io.*;
import java.net.URLConnection;

@Controller
public class PictureUploadController {
    private final Resource picturesDir;
 private final Resource anonymousPicture;

 @Autowired
 public PictureUploadController(PictureUploadProperties uploadProperties) {
 picturesDir = uploadProperties.getUploadPath();
 anonymousPicture = uploadProperties.getAnonymousPicture();
 }

    @RequestMapping(value = "/uploadedPicture")
    public void getUploadedPicture(HttpServletResponse response) throws IOException {
        response.setHeader("Content-Type", URLConnection.guessContentTypeFromName(anonymousPicture.getFilename()));
 IOUtils.copy(anonymousPicture.getInputStream(), response.getOutputStream());
    }

  private Resource copyFileToPictures(MultipartFile file) throws IOException {
       String fileExtension = getFileExtension(file.getOriginalFilename());
       File tempFile = File.createTempFile("pic", fileExtension, picturesDir.getFile());
       try (InputStream in = file.getInputStream();
            OutputStream out = new FileOutputStream(tempFile)) {

           IOUtils.copy(in, out);
       }
       return new FileSystemResource(tempFile);
   }    
// The rest of the code remains the same
}
```

此时，如果您再次启动应用程序，您会发现结果并没有改变。匿名图片仍然显示，用户上传的图片仍然会出现在项目根目录的`pictures`目录中。

## 显示上传的图片

现在，我们需要向我们的`PictureUploadController`类添加一个模型属性来显示用户的图片：

```java
@ModelAttribute("picturePath")
public Resource picturePath() {
  return anonymousPicture;
}
```

现在我们可以注入它以在提供上传的图片时检索其值：

```java
@RequestMapping(value = "/uploadedPicture")
public void getUploadedPicture(HttpServletResponse response, @ModelAttribute("picturePath") Path picturePath) throws IOException {
    response.setHeader("Content-Type", URLConnection.guessContentTypeFromName(picturePath.toString()));
    Files.copy(picturePath, response.getOutputStream());
}
```

`@ModelAttribute`注解是一种方便的方法，可以使用带注解的方法创建模型属性。然后可以使用相同的注解将它们注入到控制器方法中。使用此代码，只要我们没有重定向到另一个页面，模型中就会有一个`picturePath`参数。它的默认值是我们在属性中定义的匿名图片。

当文件上传时，我们需要更新此值。更新`onUpload`方法：

```java
@RequestMapping(value = "/upload", method = RequestMethod.POST)
public String onUpload(MultipartFile file, RedirectAttributes redirectAttrs, Model model) throws IOException {

    if (file.isEmpty() || !isImage(file)) {
        redirectAttrs.addFlashAttribute("error", "Incorrect file. Please upload a picture.");
        return "redirect:/upload";
    }

    Resource picturePath = copyFileToPictures(file);
 model.addAttribute("picturePath", picturePath);

    return "profile/uploadPage";
}
```

通过注入模型，我们可以在上传完成后更新`picturePath`参数。

现在，问题是我们的两个方法`onUpload`和`getUploadedPicture`将出现在不同的请求中。不幸的是，模型属性将在每次之间重置。

这就是为什么我们将`picturePath`参数定义为会话属性。我们可以通过向我们的控制器类添加另一个注解来实现这一点：

```java
@Controller
@SessionAttributes("picturePath")
public class PictureUploadController {
}
```

哎呀！这么多注解只是为了处理一个简单的会话属性。您将获得以下输出：

![显示上传的图片](img/image00955.jpeg)

这种方法使代码组合变得非常容易。此外，我们没有直接使用`HttpServletRequest`或`HttpSession`。此外，我们的对象可以很容易地进行类型化。

## 处理文件上传错误

我的细心读者一定会想到我们的代码可能会抛出两种异常：

+   `IOException`: 如果在将文件写入磁盘时发生了不好的情况，就会抛出此错误。

+   `MultipartException`: 如果上传文件时发生错误，则会抛出此错误。例如，当超过最大文件大小时。

这将为我们提供一个很好的机会来看一下在 Spring 中处理异常的两种方式：

+   在控制器方法中使用`@ExceptionHandler`注解

+   使用在 Servlet 容器级别定义的全局异常处理程序

让我们通过在我们的`PictureUploadController`类中使用`@ExceptionHandler`注解来处理`IOException`，添加以下方法：

```java
@ExceptionHandler(IOException.class)
public ModelAndView handleIOException(IOException exception) {
    ModelAndView modelAndView = new ModelAndView("profile/uploadPage");
    modelAndView.addObject("error", exception.getMessage());
    return modelAndView;
}
```

这是一种简单而强大的方法。每当我们的控制器中抛出`IOException`时，将调用此方法。

为了测试异常处理程序，由于使 Java IO 代码抛出异常可能会很棘手，只需在测试期间替换`onUpload`方法体：

```java
@RequestMapping(value = "/upload", method = RequestMethod.POST)
public String onUpload(MultipartFile file, RedirectAttributes redirectAttrs, Model model) throws IOException {
    throw new IOException("Some message");
}
```

更改后，如果我们尝试上传图片，将在上传页面上看到此异常的错误消息显示：

![处理文件上传错误](img/image00956.jpeg)

现在，我们将处理`MultipartException`。这需要在 Servlet 容器级别（即在 Tomcat 级别）进行，因为此异常不是直接由我们的控制器抛出的。

我们需要向我们的配置中添加一个新的`EmbeddedServletContainerCustomizer` bean。将此方法添加到`WebConfiguration`类中：

```java
@Bean
public EmbeddedServletContainerCustomizer containerCustomizer() {
    EmbeddedServletContainerCustomizer 
embeddedServletContainerCustomizer = new EmbeddedServletContainerCustomizer() {
        @Override
        public void customize(ConfigurableEmbeddedServletContainer container) {
            container.addErrorPages(new ErrorPage(MultipartException.class, "/uploadError"));
        }
    };
    return embeddedServletContainerCustomizer;
}
```

这有点啰嗦。请注意，`EmbeddedServletContainerCustomizer`是一个包含单个方法的接口；因此，它可以被 lambda 表达式替换：

```java
@Bean
public EmbeddedServletContainerCustomizer containerCustomizer() {
    EmbeddedServletContainerCustomizer embeddedServletContainerCustomizer
            = container -> container.addErrorPages(new ErrorPage(MultipartException.class, "/uploadError"));
    return embeddedServletContainerCustomizer;
}
```

因此，我们只需编写以下内容：

```java
@Bean
public EmbeddedServletContainerCustomizer containerCustomizer() {
    return container -> container.addErrorPages(new ErrorPage(MultipartException.class, "/uploadError"));
}
```

此代码创建了一个新的错误页面，当发生`MultipartException`时将调用该页面。它还可以映射到 HTTP 状态。`EmbeddedServletContainerCustomizer`接口还具有许多其他功能，将允许自定义我们的应用程序运行的 Servlet 容器。有关更多信息，请访问[`docs.spring.io/spring-boot/docs/current/reference/html/boot-features-developing-web-applications.html#boot-features-customizing-embedded-containers`](http://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-developing-web-applications.html#boot-features-customizing-embedded-containers)。

现在，我们需要在我们的`PictureUploadController`类中处理这个`uploadError` URL：

```java
@RequestMapping("uploadError")
public ModelAndView onUploadError(HttpServletRequest request) {
    ModelAndView modelAndView = new ModelAndView("uploadPage");
    modelAndView.addObject("error", request.getAttribute(WebUtils.ERROR_MESSAGE_ATTRIBUTE));
    return modelAndView;
}
```

在 Servlet 环境中定义的错误页面包含许多有助于调试错误的有趣属性：

| 属性 | 描述 |
| --- | --- |
| `javax.servlet.error.status_code` | 这是错误的 HTTP 状态码。 |
| `javax.servlet.error.exception_type` | 这是异常类。 |
| `javax.servlet.error.message` | 这是抛出的异常的消息。 |
| `javax.servlet.error.request_uri` | 这是发生异常的 URI。 |
| `javax.servlet.error.exception` | 这是实际的异常。 |
| `javax.servlet.error.servlet_name` | 这是捕获异常的 Servlet 的名称。 |

所有这些属性都可以方便地在 Spring Web 的`WebUtils`类上访问。

如果有人试图上传太大的文件，他们将收到非常明确的错误消息。

您现在可以通过上传一个非常大的文件（> 1Mb）或将`multipart.maxFileSize`属性设置为较低的值（例如 1kb）来测试错误是否被正确处理：

![处理文件上传错误](img/image00957.jpeg)

# 翻译错误消息

对于开发人员来说，看到应用程序抛出的异常是非常好的。然而，对于我们的用户来说，它们的价值很小。因此，我们将对它们进行翻译。为了做到这一点，我们必须将我们应用程序的`MessageSource`类注入到我们控制器的构造函数中：

```java

private final MessageSource messageSource;

@Autowired
public PictureUploadController(PictureUploadProperties uploadProperties, MessageSource messageSource) {
    picturesDir = uploadProperties.getUploadPath();
    anonymousPicture = uploadProperties.getAnonymousPicture();
    this.messageSource = messageSource;
}
```

现在，我们可以从消息包中检索消息：

```java
@ExceptionHandler(IOException.class)
public ModelAndView handleIOException(Locale locale) {
    ModelAndView modelAndView = new ModelAndView("profile/uploadPage");
    modelAndView.addObject("error", messageSource.getMessage("upload.io.exception", null, locale));
    return modelAndView;
}

@RequestMapping("uploadError")
public ModelAndView onUploadError(Locale locale) {
    ModelAndView modelAndView = new ModelAndView("profile/uploadPage");
    modelAndView.addObject("error", messageSource.getMessage("upload.file.too.big", null, locale));
    return modelAndView;
}
```

以下是英文消息：

```java
upload.io.exception=An error occurred while uploading the file. Please try again.
upload.file.too.big=Your file is too big.
```

现在，法语消息：

```java
upload.io.exception=Une erreur est survenue lors de l'envoi du fichier. Veuillez réessayer.
upload.file.too.big=Votre fichier est trop gros.
```

# 将配置文件放入会话中

我们希望的下一步是将配置文件存储在会话中，以便每次进入配置文件页面时都不会被重置。这对一些用户来说可能会很烦人，我们必须解决这个问题。

### 提示

**HTTP 会话**是在请求之间存储信息的一种方式。HTTP 是一种无状态协议，这意味着没有办法将来自同一用户的两个请求联系起来。大多数 Servlet 容器所做的是将名为`JSESSIONID`的 cookie 与每个用户关联起来。该 cookie 将在请求头中传输，并允许您在一个称为`HttpSession`的抽象中存储任意对象。这样的会话通常会在用户关闭或切换 Web 浏览器或预定义的不活动期之后结束。

我们刚刚看到了一种使用`@SessionAttributes`注解将对象放入会话中的方法。这在控制器内部效果很好，但在多个控制器之间共享数据时会变得困难。我们必须依赖字符串来从其名称解析属性，这很难重构。出于同样的原因，我们不希望直接操作`HttpSession`。另一个阻止直接使用会话的论点是，依赖于它的控制器很难进行单元测试。

在使用 Spring 保存会话中的内容时，还有另一种流行的方法：使用`@Scope("session")`为 bean 添加注释。

然后，您将能够在控制器和其他 Spring 组件中注入会话 bean，以设置或检索其中的值。

让我们在`profile`包中创建一个`UserProfileSession`类：

```java
package masterSpringMvc.profile;

import org.springframework.context.annotation.Scope;
import org.springframework.context.annotation.ScopedProxyMode;
import org.springframework.stereotype.Component;
import java.io.Serializable;
import java.time.LocalDate;
import java.util.ArrayList;
import java.util.List;

@Component
@Scope(value = "session", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class UserProfileSession implements Serializable {
    private String twitterHandle;
    private String email;
    private LocalDate birthDate;
    private List<String> tastes = new ArrayList<>();

    public void saveForm(ProfileForm profileForm) {
        this.twitterHandle = profileForm.getTwitterHandle();
        this.email = profileForm.getEmail();
        this.birthDate = profileForm.getBirthDate();
        this.tastes = profileForm.getTastes();
    }

    public ProfileForm toForm() {
        ProfileForm profileForm = new ProfileForm();
        profileForm.setTwitterHandle(twitterHandle);
        profileForm.setEmail(email);
        profileForm.setBirthDate(birthDate);
        profileForm.setTastes(tastes);
        return profileForm;
    }
}
```

我们已经方便地提供了一种从`ProfileForm`对象转换的方法。这将帮助我们从`ProfileController`构造函数中存储和检索表单数据。我们需要在控制器的构造函数中注入我们的`UserProfileSession`变量并将其存储为字段。我们还需要将`ProfileForm`公开为模型属性，这将消除在`displayProfile`方法中注入它的需要。最后，一旦验证通过，我们就可以保存配置文件：

```java
@Controller
public class ProfileController {

    private UserProfileSession userProfileSession;
    @Autowired
    public ProfileController(UserProfileSession userProfileSession) {
        this.userProfileSession = userProfileSession;
    }

    @ModelAttribute
    public ProfileForm getProfileForm() {
        return userProfileSession.toForm();
    }

    @RequestMapping(value = "/profile", params = {"save"}, method = RequestMethod.POST)
    public String saveProfile(@Valid ProfileForm profileForm, BindingResult bindingResult) {
        if (bindingResult.hasErrors()) {
            return "profile/profilePage";
        }
        userProfileSession.saveForm(profileForm);
        return "redirect:/profile";
    }

    // the rest of the code is unchanged
}
```

这就是使用 Spring MVC 在会话中保存数据所需的全部内容。

现在，如果您完成了配置文件表单并刷新页面，数据将在请求之间持久保存。

在进入下一章之前，我想详细介绍一些我们刚刚使用的概念。

首先是通过构造函数进行注入。`ProfileController`构造函数使用`@Autowired`进行注释，这意味着 Spring 将在实例化 bean 之前从应用程序上下文中解析构造函数参数。另一种稍微不那么冗长的替代方法是使用字段注入：

```java
@Controller
public class ProfileController {

    @Autowired
    private UserProfileSession userProfileSession;
}
```

构造函数注入可能更好，因为如果我们要远离`spring-test`框架，它会使我们的控制器的单元测试更容易，并且它会使我们的 bean 的依赖关系更加明确。

有关字段注入和构造函数注入的详细讨论，请参阅 Oliver Gierke 在[`olivergierke.de/2013/11/why-field-injection-is-evil/`](http://olivergierke.de/2013/11/why-field-injection-is-evil/)上的出色博客文章。

可能需要澄清的另一件事是`Scope`注解上的`proxyMode`参数：

```java
@Scope(value = "session", proxyMode = ScopedProxyMode.TARGET_CLASS)
```

Spring 有三个`proxyMode`参数可用，如果不计算默认值：

+   `TARGET_CLASS`：这使用 CGI 代理

+   `INTERFACES`：这将创建一个 JDK 代理

+   `NO`：这不会创建任何代理

代理的优势通常在将某些东西注入到长期存在的组件中（如单例）时发挥作用。因为注入只发生一次，当 bean 被创建时，对注入的 bean 的后续调用可能不会反映其实际状态。

在我们的情况下，会话 bean 的实际状态存储在会话中，而不是直接存储在 bean 中。这解释了为什么 Spring 必须创建代理：它需要拦截对我们 bean 方法的调用，并监听其变化。这样，bean 的状态可以透明地存储和检索到底层的 HTTP 会话中。

对于会话 bean，我们被迫使用代理模式。CGI 代理将对您的字节码进行检测，并在任何类上工作，而 JDK 方法可能会更轻量级一些，但需要您实现一个接口。

最后，我们使`UserProfileSession` bean 实现了`Serializable`接口。这并不是严格要求的，因为 HTTP 会话可以在内存中存储任意对象，但是使最终存储在会话中的对象可序列化确实是一个好习惯。

实际上，我们可能会更改会话的持久化方式。事实上，我们将在第七章中将会话存储在 Redis 数据库中，*优化您的请求*，在那里 Redis 必须使用`Serializable`对象。最好始终将会话视为通用数据存储。我们必须提供一种从该存储系统中写入和读取对象的方法。

为了使我们的 bean 上的序列化正常工作，我们还需要使其每个字段都可序列化。在我们的情况下，字符串和日期是可序列化的，所以我们可以继续。

# 自定义错误页面

Spring Boot 允许您定义自己的错误视图，而不是我们之前看到的 Whitelabel 错误页面。它必须具有名称`error`，其目的是处理所有异常。默认的`BasicErrorController`类将公开许多有用的模型属性，您可以在此页面上显示这些属性。

让我们在`src/main/resources/templates`中创建一个自定义错误页面。让我们称之为`error.html`：

```java
<!DOCTYPE html>
<html >
<head lang="en">
    <meta charset="UTF-8"/>
    <title th:text="${status}">404</title>

    <link href="/webjars/materializecss/0.96.0/css/materialize.css" type="text/css" rel="stylesheet"
          media="screen,projection"/>
</head>
<body>
<div class="row">
    <h1 class="indigo-text center" th:text="${error}">Not found</h1>

    <p class="col s12 center" th:text="${message}">
        This page is not available
    </p>
</div>
</body>
</html>
```

现在，如果我们导航到我们的应用程序未处理的 URL，我们会看到我们的自定义错误页面：

![自定义错误页面](img/image00958.jpeg)

处理错误的更高级选项是定义自己的`ErrorController`类的实现，这是负责全局处理所有异常的控制器。查看`ErrorMvcAutoConfiguration`类和`BasicErrorController`类，它是默认实现。

# 使用矩阵变量的 URL 映射

我们现在知道我们的用户对什么感兴趣。改进我们的 Tweet 控制器，以便它允许从关键字列表中进行搜索，这是一个好主意。

在 URL 中传递键值对的一个有趣方式是使用矩阵变量。这与请求参数非常相似。考虑以下代码：

```java
someUrl/param?var1=value1&var2=value2
```

矩阵变量理解前面的参数：

```java
someUrl/param;var1=value1;var2=value2
```

它们还允许每个参数都是一个列表：

```java
someUrl/param;var1=value1,value2;var2=value3,value4
```

矩阵变量可以映射到控制器内的不同对象类型：

+   `Map<String, List<?>>`：这处理多个变量和多个值

+   `Map<String, ?>`：这处理每个变量只有一个值的情况

+   `List<?>`：如果我们对一个可以配置名称的单个变量感兴趣，就会使用这个

在我们的情况下，我们想要处理这样的情况：

```java
http://localhost:8080/search/popular;keywords=scala,java
```

第一个参数`popular`是 Twitter 搜索 API 已知的结果类型。它可以取以下值：`mixed`、`recent`或`popular`。

我们 URL 的其余部分是关键字列表。因此，我们将它们映射到一个简单的`List<String>`对象。

默认情况下，Spring MVC 会删除 URL 中分号后面的每个字符。我们需要做的第一件事是关闭这种行为，以启用我们应用程序中的矩阵变量。

让我们在`WebConfiguration`类中添加以下代码：

```java
@Override
public void configurePathMatch(PathMatchConfigurer configurer) {
    UrlPathHelper urlPathHelper = new UrlPathHelper();
    urlPathHelper.setRemoveSemicolonContent(false);
    configurer.setUrlPathHelper(urlPathHelper);
}
```

让我们在`search`包中创建一个新的控制器，我们将其称为`SearchController`。它的作用是处理以下请求：

```java
package masterSpringMvc.search;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.social.twitter.api.Tweet;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.MatrixVariable;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import java.util.List;

@Controller
public class SearchController {
    private SearchService searchService;
    @Autowired
    public SearchController(SearchService searchService) {
        this.searchService = searchService;
    }

    @RequestMapping("/search/{searchType}")
    public ModelAndView search(@PathVariable String searchType, @MatrixVariable List<String> keywords) {
        List<Tweet> tweets = searchService.search(searchType, keywords);
        ModelAndView modelAndView = new ModelAndView("resultPage");
        modelAndView.addObject("tweets", tweets);
        modelAndView.addObject("search", String.join(",", keywords));
        return modelAndView;
    }
}
```

正如你所看到的，我们能够重用现有的结果页面来显示推文。我们还希望将搜索委托给另一个名为`SearchService`的类。我们将在与`SearchController`相同的包中创建这个服务：

```java
package masterSpringMvc.search;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.social.twitter.api.Tweet;
import org.springframework.social.twitter.api.Twitter;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class SearchService {
    private Twitter twitter;

    @Autowired
    public SearchService(Twitter twitter) {
        this.twitter = twitter;
    }

    public List<Tweet> search(String searchType, List<String> keywords) {
        return null;
    }
}
```

现在，我们需要实现`search()`方法。

`twitter.searchOperations().search(params)`上可访问的搜索操作以`searchParameters`作为高级搜索的参数。这个对象允许我们根据十几个标准进行搜索。我们对`query`、`resultType`和`count`属性感兴趣。

首先，我们需要创建一个`ResultType`构造函数，其中包含`searchType`路径变量。`ResultType`是一个枚举，所以我们可以迭代它的不同值，并找到与输入匹配的值，忽略大小写：

```java
private SearchParameters.ResultType getResultType(String searchType) {
    for (SearchParameters.ResultType knownType : SearchParameters.ResultType.values()) {
        if (knownType.name().equalsIgnoreCase(searchType)) {
            return knownType;
        }
    }
    return SearchParameters.ResultType.RECENT;
}
```

现在我们可以创建一个带有以下方法的`SearchParameters`构造函数：

```java
private SearchParameters createSearchParam(String searchType, String taste) {

    SearchParameters.ResultType resultType = getResultType(searchType);
    SearchParameters searchParameters = new SearchParameters(taste);
    searchParameters.resultType(resultType);
    searchParameters.count(3);
    return searchParameters;
}
```

现在，创建`SearchParameters`构造函数的列表就像进行映射操作一样简单（获取关键字列表并为每个关键字返回一个`SearchParameters`构造函数）：

```java
List<SearchParameters> searches = keywords.stream()
        .map(taste -> createSearchParam(searchType, taste))
        .collect(Collectors.toList());
```

现在，我们想要为每个`SearchParameters`构造函数获取推文。你可能会想到这样的东西：

```java
List<Tweet> tweets = searches.stream()
        .map(params -> twitter.searchOperations().search(params))
        .map(searchResults -> searchResults.getTweets())
        .collect(Collectors.toList());
```

然而，如果你仔细想想，这将返回一个推文列表。我们想要的是将所有推文展平，以便得到一个简单的列表。原来调用`map`然后展平结果的操作称为`flatMap`。所以我们可以写：

```java
List<Tweet> tweets = searches.stream()
        .map(params -> twitter.searchOperations().search(params))
        .flatMap(searchResults -> searchResults.getTweets().stream())
        .collect(Collectors.toList());
```

`flatMap`函数的语法，它以流作为参数，一开始有点难以理解。让我向你展示`SearchService`类的整个代码，这样我们就可以退一步看看：

```java
package masterSpringMvc.search;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.social.twitter.api.SearchParameters;
import org.springframework.social.twitter.api.Tweet;
import org.springframework.social.twitter.api.Twitter;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.stream.Collectors;

@Service
public class SearchService {
    private Twitter twitter;

    @Autowired
    public SearchService(Twitter twitter) {
        this.twitter = twitter;
    }

    public List<Tweet> search(String searchType, List<String> keywords) {
        List<SearchParameters> searches = keywords.stream()
                .map(taste -> createSearchParam(searchType, taste))
                .collect(Collectors.toList());

        List<Tweet> results = searches.stream()
                .map(params -> twitter.searchOperations().search(params))
                .flatMap(searchResults -> searchResults.getTweets().stream())
                .collect(Collectors.toList());

        return results;
    }

    private SearchParameters.ResultType getResultType(String searchType) {
        for (SearchParameters.ResultType knownType : SearchParameters.ResultType.values()) {
            if (knownType.name().equalsIgnoreCase(searchType)) {
                return knownType;
            }
        }
        return SearchParameters.ResultType.RECENT;
    }

    private SearchParameters createSearchParam(String searchType, String taste) {
        SearchParameters.ResultType resultType = getResultType(searchType);
        SearchParameters searchParameters = new SearchParameters(taste);
        searchParameters.resultType(resultType);
        searchParameters.count(3);
        return searchParameters;
    }
}
```

现在，如果我们导航到`http://localhost:8080/search/mixed;keywords=scala,java`，我们会得到预期的结果。首先搜索 Scala 关键字，然后搜索 Java：

![带矩阵变量的 URL 映射](img/image00959.jpeg)

# 把它放在一起

现在，一切都可以单独工作，是时候将一切组合起来了。我们将分三步完成这个过程：

1.  将上传表单移动到个人资料页面，并删除旧的上传页面。

1.  将个人资料页面上的提交按钮更改为直接触发口味搜索。

1.  更改我们应用程序的主页。它应该立即显示与我们用户口味匹配的搜索结果。如果不可用，转到个人资料页面。

我鼓励你尝试自己做。你会在途中遇到一些很容易解决的问题，但你应该知道足够的知识来自己解决它们。我相信你。

好了，现在你已经完成了工作（你已经完成了，对吧？），让我们来看看我的解决方案。

第一步是删除旧的`uploadPage`标题。不要回头，就这么做。

接下来，将这些行放在`profilePage`标题的下方：

```java
<div class="row">

    <div class="col m8 s12 offset-m2">
        <img th:src="img/uploadedPicture}" width="100" height="100"/>
    </div>

    <div class="col s12 center red-text" th:text="${error}" th:if="${error}">
        Error during upload
    </div>

    <form th:action="@{/profile}" method="post" enctype="multipart/form-data" class="col m8 s12 offset-m2">

        <div class="input-field col s6">
            <input type="file" id="file" name="file"/>
        </div>

        <div class="col s6 center">
            <button class="btn indigo waves-effect waves-light" type="submit" name="upload" th:text="#{upload}">Upload
                <i class="mdi-content-send right"></i>
            </button>
        </div>
    </form>
</div>
```

这与以前的`uploadPage`的内容非常相似。我们只是删除了标题，并更改了提交按钮的标签。将相应的翻译添加到包中。

在英语中：

```java
upload=Upload
```

用法语：

```java
Upload=Envoyer
```

我们还将提交按钮的名称更改为`upload`。这将帮助我们在控制器端识别这个操作。

现在，如果我们尝试上传我们的图片，它会将我们重定向到旧的上传页面。我们需要在`PictureUploadController`类的`onUpload`方法中修复这个问题：

```java
@RequestMapping(value = "/profile", params = {"upload"}, method = RequestMethod.POST)
public String onUpload(@RequestParam MultipartFile file, RedirectAttributes redirectAttrs) throws IOException {

    if (file.isEmpty() || !isImage(file)) {
        redirectAttrs.addFlashAttribute("error", "Incorrect file. Please upload a picture.");
        return "redirect:/profile";
    }

    Resource picturePath = copyFileToPictures(file);
    userProfileSession.setPicturePath(picturePath);

    return "redirect:profile";
}
```

请注意，我们更改了处理 POST 的 URL。现在是`/profile`而不是`/upload`。当`GET`和`POST`请求具有相同的 URL 时，表单处理会更简单，并且在处理异常时会节省我们很多麻烦。这样，我们就不必在错误发生后重定向用户。

我们还删除了模型属性`picturePath`。因为我们现在在会话中有一个代表用户的 bean，`UserProfileSession`，我们决定将其添加在那里。我们在`UserProfileSession`类中添加了一个`picturePath`属性以及相关的 getter 和 setter。

不要忘记在我们的`PictureUploadController`类中注入`UserProfileSession`类，并将其作为字段可用。

请记住，我们会话 bean 的所有属性都必须是可序列化的，与资源不同。因此我们需要以不同的方式存储它。URL 类似乎是一个很好的选择。它是可序列化的，而且很容易使用`UrlResource`类从 URL 创建资源：

```java
@Component
@Scope(value = "session", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class UserProfileSession implements Serializable {
    private URL picturePath;

    public void setPicturePath(Resource picturePath) throws IOException {
        this.picturePath = picturePath.getURL();
    }

    public Resource getPicturePath() {
        return picturePath == null ? null : new UrlResource(picturePath);
    }
}
```

我需要做的最后一件事是在错误后将`profileForm`作为模型属性可用。这是因为在呈现`profilePage`时需要它。

总之，这是`PictureUploadController`类的最终版本：

```java
package masterSpringMvc.profile;

import masterSpringMvc.config.PictureUploadProperties;
import org.apache.tomcat.util.http.fileupload.IOUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.MessageSource;
import org.springframework.core.io.FileSystemResource;
import org.springframework.core.io.Resource;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import javax.servlet.http.HttpServletResponse;
import java.io.*;
import java.net.URLConnection;
import java.util.Locale;

@Controller
public class PictureUploadController {
    private final Resource picturesDir;
    private final Resource anonymousPicture;
    private final MessageSource messageSource;
    private final UserProfileSession userProfileSession;

    @Autowired
    public PictureUploadController(PictureUploadProperties uploadProperties,
                                   MessageSource messageSource,
                                   UserProfileSession userProfileSession) {
        picturesDir = uploadProperties.getUploadPath();
        anonymousPicture = uploadProperties.getAnonymousPicture();
        this.messageSource = messageSource;
        this.userProfileSession = userProfileSession;
    }

    @RequestMapping(value = "/uploadedPicture")
    public void getUploadedPicture(HttpServletResponse response) throws IOException {
        Resource picturePath = userProfileSession.getPicturePath();
        if (picturePath == null) {
            picturePath = anonymousPicture;
        }
        response.setHeader("Content-Type", URLConnection.guessContentTypeFromName(picturePath.getFilename()));
        IOUtils.copy(picturePath.getInputStream(), response.getOutputStream());
    }

    @RequestMapping(value = "/profile", params = {"upload"}, method = RequestMethod.POST)
    public String onUpload(@RequestParam MultipartFile file, RedirectAttributes redirectAttrs) throws IOException {

        if (file.isEmpty() || !isImage(file)) {
            redirectAttrs.addFlashAttribute("error", "Incorrect file. Please upload a picture.");
            return "redirect:/profile";
        }

        Resource picturePath = copyFileToPictures(file);
        userProfileSession.setPicturePath(picturePath);

        return "redirect:profile";
    }

    private Resource copyFileToPictures(MultipartFile file) throws IOException {
        String fileExtension = getFileExtension(file.getOriginalFilename());
        File tempFile = File.createTempFile("pic", fileExtension, picturesDir.getFile());
        try (InputStream in = file.getInputStream();
             OutputStream out = new FileOutputStream(tempFile)) {

            IOUtils.copy(in, out);
        }
        return new FileSystemResource(tempFile);
    }

    @ExceptionHandler(IOException.class)
    public ModelAndView handleIOException(Locale locale) {
        ModelAndView modelAndView = new ModelAndView("profile/profilePage");
        modelAndView.addObject("error", messageSource.getMessage("upload.io.exception", null, locale));
        modelAndView.addObject("profileForm", userProfileSession.toForm());
        return modelAndView;
    }

    @RequestMapping("uploadError")
    public ModelAndView onUploadError(Locale locale) {
        ModelAndView modelAndView = new ModelAndView("profile/profilePage");
        modelAndView.addObject("error", messageSource.getMessage("upload.file.too.big", null, locale));
        modelAndView.addObject("profileForm", userProfileSession.toForm());
        return modelAndView;
    }

    private boolean isImage(MultipartFile file) {
        return file.getContentType().startsWith("image");
    }

    private static String getFileExtension(String name) {
        return name.substring(name.lastIndexOf("."));
    }
}
```

因此，现在我们可以转到个人资料页面，上传我们的图片，并提供个人信息，如下截图所示：

![将其放在一起](img/image00960.jpeg)

现在，让我们在个人资料完成后将用户重定向到搜索页面。为此，我们需要修改`ProfileController`类中的`saveProfile`方法：

```java
@RequestMapping(value = "/profile", params = {"save"}, method = RequestMethod.POST)
public String saveProfile(@Valid ProfileForm profileForm, BindingResult bindingResult) {
    if (bindingResult.hasErrors()) {
        return "profile/profilePage";
    }
    userProfileSession.saveForm(profileForm);
    return "redirect:/search/mixed;keywords=" + String.join(",", profileForm.getTastes());
}
```

现在我们能够从我们的个人资料搜索推文，我们不再需要之前创建的`searchPage`或`TweetController`。只需删除`searchPage.html`页面和`TweetController`。

最后，我们可以修改我们的主页，这样如果我们已经完成了我们的个人资料，它就会重定向我们到一个符合我们口味的搜索页面。

让我们在控制器包中创建一个新的控制器。它负责将访问我们网站根目录的用户重定向到他们的个人资料（如果资料不完整）或`resultPage`（如果他们的口味可用）：

```java
package masterSpringMvc.controller;

import masterSpringMvc.profile.UserProfileSession;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

import java.util.List;

@Controller
public class HomeController {
    private UserProfileSession userProfileSession;

    @Autowired
    public HomeController(UserProfileSession userProfileSession) {
        this.userProfileSession = userProfileSession;
    }

    @RequestMapping("/")
    public String home() {
        List<String> tastes = userProfileSession.getTastes();
        if (tastes.isEmpty()) {
            return "redirect:/profile";
        }
        return "redirect:/search/mixed;keywords=" + String.join(",", tastes);
    }
}
```

# 检查点

在本章中，我们添加了两个控制器，`PictureUploadController`负责将上传的文件写入磁盘并处理上传错误，`SearchController`可以使用矩阵参数从关键字列表中搜索推文。

然后，该控制器将搜索委托给一个新的服务，`SearchService`。

我们删除了旧的`TweetController`。

我们创建了一个会话 bean，`UserProfileSession`，来存储关于用户的信息。

最后，我们在`WebConfiguration`中添加了两个内容。我们为 Servlet 容器添加了错误页面，并支持矩阵变量。

![检查点](img/image00961.jpeg)

在资源方面，我们添加了一个代表匿名用户的图片和一个处理错误的静态页面。我们将文件上传到`profilePage`，并且删除了旧的`searchPage`。

![检查点](img/image00962.jpeg)

# 总结

在本章中，我们讨论了文件上传和错误处理。上传文件并不是很复杂。然而，一个重要的设计决定是如何处理上传的文件。我们本可以将图片存储在数据库中，但我们选择将其写入磁盘，并将每个用户的图片位置保存在他们的会话中。

我们看到了在控制器级别和 Servlet 容器级别处理异常的典型方法。有关 Spring MVC 错误处理的其他资源，您可以参考[`spring.io/blog/2013/11/01/exception-handling-in-spring-mvc`](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc)上的博客文章。

我们的应用程序已经看起来相当不错，但我们需要编写的代码量非常合理。

敬请关注下一章，我们将看到 Spring MVC 也是构建 REST 应用程序的强大框架。

