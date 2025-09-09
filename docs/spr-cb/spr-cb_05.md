# 第五章。使用表单

在本章中，我们将介绍以下菜谱：

+   显示和处理表单

+   使用控制器方法参数获取提交的表单值

+   使用模型对象设置表单的默认值

+   自动将表单数据保存到对象中

+   使用文本、文本区域、密码和隐藏字段

+   使用选择字段

+   使用复选框

+   使用复选框列表

+   使用单选按钮列表

+   使用注解验证表单

+   上传文件

# 简介

显示和处理表单是繁琐的。Spring 通过初始化表单、生成表单小部件（文本字段、复选框等）以及在表单提交时检索数据来帮助处理。通过模型类中的注解，表单验证变得简单。

# 显示和处理表单

要显示表单并检索用户提交的数据，请使用第一个控制器方法显示表单。使用第二个控制器方法处理表单提交时的表单数据。

## 如何做...

显示和处理表单的步骤如下：

1.  创建一个控制器方法来显示表单：

    ```java
    @RequestMapping("/addUser")
    public String addUser() {
      return "addUser";
    }
    ```

1.  创建一个包含 HTML 表单的 JSP：

    ```java
    <form method="POST">
      <input type="text" name="firstName" />
      <input type="submit" />
    </form>
    ```

1.  创建另一个控制器方法来处理表单提交：

    ```java
    @RequestMapping(value="/addUser", method=RequestMethod.POST)
    public String addUserSubmit(HttpServletRequest request) {
      String firstName = request.getParameter("firstName");
      ...
      return "redirect:/home";
    }
    ```

## 它是如何工作的...

第一个控制器方法显示包含 HTML 表单的 JSP。有关更多详细信息，请参阅第三章 *使用 JSP 视图* 的菜谱，*使用控制器和视图*。

HTML 表单包含一个文本字段。它通过 `POST` 提交。表单的 `action` 属性不存在，因此表单将提交到当前页面 URL (`/addUser`)。

当表单提交时，调用第二个控制器方法。使用 `HttpServletRequest` 获取 `firstName` 表单字段的值。最后，我们将重定向到 `/home`。

两种控制器方法映射到相同的 `/addUser` URL。第一种方法用于 HTTP `GET` 请求。第二种方法用于 HTTP `POST` 请求（因为 `method=RequestMethod.POST`）。

## 还有更多...

两种控制器方法可能有不同的 URL。例如，`/addUser` 和 `/addUserSubmit`。在这种情况下，在 JSP 中，我们会使用 `action` 属性：

```java
<form method="POST" action="/addUserSubmit">
  …
</form>
```

## 参见

在第二个控制器方法中，为了避免为每个表单字段使用繁琐的 `request.getParameter()` 方法，请参阅 *使用控制器方法参数获取提交的表单值* 和 *将表单值保存到对象中自动* 的菜谱。

# 使用控制器方法参数获取提交的表单值

在这个菜谱中，你将学习如何使用控制器方法参数获取提交的表单数据。这对于与领域对象无关的简单表单来说很方便。

## 如何做...

向控制器方法添加一个带有 `@RequestParam` 注解的参数：

```java
@RequestMapping("processForm")
public void processForm(@RequestParam("name") String userName) {
...
```

## 它是如何工作的...

`userName` 参数由 Spring 初始化为表单字段 `name` 的提交值。

`@RequestParam`也可以检索 URL 参数，例如，`http://localhost:8080/springwebapp/processForm?name=Merlin`。

## 更多内容…

还可以将标准的`HttpServletRequest`对象作为控制器方法的参数，并直接从其中获取`name`的提交值：

```java
@RequestMapping("processForm")
public void processForm(HttpServletRequest request) {
  String name = request.getParameter("name");

```

## 参见

参考有关将表单值自动保存到对象的食谱以获取更多详细信息。

# 使用模型对象设置表单的默认值

在本食谱中，你将学习如何显示一个用户可以更改的初始值的表单。

## 如何操作…

在控制器中创建一个包含默认值的对象。在视图中，使用 Spring 表单标签使用该对象生成表单：

1.  在控制器中，添加一个带有`@ModelAttribute`注解的方法，该方法返回一个具有默认值的对象：

    ```java
    @ModelAttribute("defaultUser")
    public User defaultUser() {
      User user = new User();
      user.setFirstName("Joe");
      user.setAge(18);
      return user;
    }
    ```

1.  在控制器中，添加一个显示表单的方法：

    ```java
    @RequestMapping("addUser")
    public String addUser() {
      return "addUser";
    }
    ```

1.  在 JSP 中，使用 Spring 表单标签生成表单：

    ```java
    <%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>

    <form:form method="POST" modelAttribute="defaultUser">
      <form:input path="firstName" />
      <form:input path="age" />
      <input type="submit" value="Submit" />
    </form:form>
    ```

## 它是如何工作的…

在控制器中，由于`@ModelAttribute`，`defaultUser()`方法会自动为每个控制器请求调用。返回的`User`对象存储在内存中作为`defaultUser`。在 JSP 中，使用`defaultUser`初始化表单：

+   它被设置为`form:form`元素的`modelAttribute`：

    ```java
    <form:form method="POST" modelAttribute="defaultUser">
    ```

+   表单字段从`defaultUser`的相应属性中获取其值。例如，`firstName`字段将使用`defaultUser.getFirstName()`返回的值进行初始化。

# 自动保存表单数据到对象中

对于直接与模型对象相关的表单，例如，用于添加`User`的表单，提交的表单数据可以自动保存到该对象的实例中。

## 如何操作…

在处理表单提交的控制器方法中，将对象作为参数添加，并确保 JSP 中的字段名称与它的属性匹配：

1.  将带有`@ModelAttribute`注解的`User`参数添加到处理表单提交的控制器方法中：

    ```java
    @RequestMapping(value="addUser", method=RequestMethod.POST)
    public void addUser(@ModelAttribute User user) {
    ...
    ```

1.  在 JSP 中，确保表单字段与对象的现有属性相对应：

    ```java
    <form:input path="firstName" />
    <form:input path="age" />
    ```

## 它是如何工作的…

当表单提交时，幕后发生的情况如下：

+   创建一个新的`User`对象

+   通过将表单字段名称与对象属性名称匹配，将表单值注入到对象中，例如：

    ```java
    user.setFirstName(request.getParameter("firstName"));
    ```

+   结果对象通过其`@ModelAttribute`参数传递给控制器方法

## 更多内容…

而不是让 Spring 创建一个新的对象，你可以提供一个默认对象的名称：

```java
public void addUser(@ModelAttribute("defaultUser") User user) {
...
```

在这种情况下，Spring 将使用控制器类中相应的`@ModelAttribute`方法返回的对象来存储提交的表单数据：

```java
@ModelAttribute("defaultUser")
public User user() {
  User user = new User();
  user.setFirstName("Joe");
  user.setAge(18);
  return user;
}
```

# 使用文本、`textarea`、密码和隐藏字段

在本食谱中，你将学习如何使用 Spring 表单标签显示文本字段、`textarea`字段、`password`字段和`hidden`字段。当表单提交时，我们将在控制器方法中检索字段值。

## 如何操作…

这里是显示和处理文本字段的步骤：

1.  如果需要默认值，请使用默认对象的 `String` 属性（参考 *使用模型对象设置表单的默认值* 菜谱）：

    ```java
    user.setFirstName("Joe");
    ```

1.  在 JSP 中，使用这些 Spring 表单标签之一：

    ```java
    <form:input path="firstName" />
    <form:textarea path="firstName" />
    <form:password path="firstName" />
    <form:hidden path="firstName" />
    ```

1.  在处理表单提交的控制器方法中，确保 `@ModelAttribute` 对象具有相应的 `String` 属性：

    ```java
    public class User {
      private String firstName;
    ...
    ```

## 它是如何工作的…

Spring 表单标签生成 HTML 表单字段，并用默认对象中定义的默认值填充它。`path` 属性对应于默认对象的属性。当表单提交时，表单字段值将保存在 `@ModelAttribute` 对象的相应属性中。

作为参考，这是生成的 HTML 代码：

```java
<input id="firstName" name="firstName" type="text" value="Joe"/>
<textarea id="firstName" name="firstName">Joe</textarea>
<input id="firstName" name="firstName" type="password" value=""/>
<input id="firstName" name="firstName" type="hidden" value="Joe"/>
```

### 注意

注意，默认值实际上并不用于 `password` 字段。

# 使用选择字段

在本菜谱中，您将学习如何显示 `select` 字段。当表单提交时，在控制器方法中检索选中的值。

## 如何操作…

1.  在控制器中，添加一个返回包含 `select` 字段选项的 `Map` 对象的 `@ModelAttribute` 方法：

    ```java
    @ModelAttribute("countries")
    public Map<String, String>countries() {
      Map<String, String> m = new HashMap<String, String>();
      m.put("us", "United States");
      m.put("ca", "Canada");
      m.put("fr", "France");
      m.put("de", "Germany");
      return m;
    }
    ```

1.  如果需要默认值，请使用默认对象的 `String` 属性（参考 *使用模型对象设置表单的默认值* 菜谱）并使用 `Map` 中的一个键进行初始化：

    ```java
    user.setCountry("ca");
    ```

1.  在 JSP 中，使用初始化为 `@ModelAttribute` Map 的 `form:select` 元素：

    ```java
    <form:select path="country" items="${countries}" />
    ```

1.  在处理表单提交的控制器中，确保 `@ModelAttribute` 对象（用于保存表单值的对象）具有相应的 `String` 属性：

    ```java
    public class User {
      private String country;
    ...
    ```

## 它是如何工作的…

`form:select` 标签生成一个 HTML `select` 字段，并使用 `@ModelAttribute` Map 和默认对象中定义的默认值进行初始化。`path` 属性对应于默认对象的属性。当表单提交时，选中的值将保存在 `@ModelAttribute` `User` 对象的相应属性中。

作为参考，这是生成的 HTML 代码：

```java
<select id="country" name="country">
  <option value="de">Germany</option>
  <option value="fr">France</option>
  <option value="us">United States</option>
  <option value="ca" selected="selected">Canada</option>
</select>
```

## 更多内容…

对于 `@ModelAttribute` 对象，不需要使用 `Map` 类。可以使用 `List<String>` 对象或直接使用现有类的字段，这可能更方便。

### 使用 List<String> 对象

除了 `Map`，您还可以使用 `List<String>` 对象：

```java
@ModelAttribute("countries")
public List<String>countries() {
  List<String> l = new LinkedList<String>();
  l.add("com");
  l.add("ca");
  l.add("fr");
  l.add("de");    
  return l;
}
```

JSP 代码保持不变：

```java
<form:select path="country" items="${countries}" />
```

在生成的 HTML 代码中，显示的文本将与 `value` 属性相同：

```java
<select id="country" name="country">
  <option value="com">com</option>
  <option value="ca" selected="selected">ca</option>
  <option value="fr">fr</option>
  <option value="de">de</option>
</select>
```

### 使用 List<Object> 对象

除了 `Map` 对象，您还可以使用 `List<Object>` 对象。例如，如果 `Country` 是一个具有 `code` 和 `name String` 属性的类：

```java
@ModelAttribute("countries")
public List<Country>countries() {
...
```

JSP 代码将指定这些属性：

```java
<form:select path="country" items="${countries}" itemValue="code" itemLabel="name" />
```

生成的 HTML 代码将相同：

```java
<select id="country" name="country">
  <option value="de">Germany</option>
  <option value="fr">France</option>
  <option value="us">United States</option>
  <option value="ca" selected="selected">Canada</option>
</select>
```

# 使用复选框

在本菜谱中，您将学习如何显示复选框，并在控制器方法中提交表单时检索其状态（选中或不选中）。

## 如何操作…

在 JSP 中使用 `form:checkbox` 元素和一个 `boolean` 属性来存储其在表单提交时的值：

1.  如果需要默认值，使用默认对象的`boolean`属性（参考*使用模型对象设置表单的默认值*配方）：

    ```java
    user.setMarried(false);
    ```

1.  在 JSP 中，使用`form:checkbox`元素：

    ```java
    <form:checkbox path="married" />
    ```

1.  在处理表单提交的控制器中，确保`@ModelAttribute`对象有一个相应的`boolean`属性：

    ```java
    public class User {
      private boolean married;
    ...
    ```

## 它是如何工作的...

这是生成的 HTML 代码：

```java
<input id="married1" name="married" type="checkbox" value="true"/>
<input type="hidden" name="_married" value="on"/>
```

如果复选框被选中，当表单提交时发送`married=true`。如果没有选中，则不发送任何内容（这是标准的 HTML 行为）。这就是为什么还会生成一个`hidden`表单字段。字段`_married`将发送带有值`on`，无论复选框的状态如何。所以，如果已婚字段不存在，Spring 将知道将其值设置为 false。

# 使用复选框列表

在这个配方中，你将学习如何显示一组复选框，并在表单提交时，如何在控制器方法中检索选中的值。

## 如何做到这一点...

1.  在控制器中，添加一个返回`Map`对象的`@ModelAttribute`方法：

    ```java
    @ModelAttribute("languages")
    public Map<String, String>languages() {
      Map<String, String> m = new HashMap<String, String>();
      m.put("en", "English");
      m.put("fr", "French");
      m.put("de", "German");
      m.put("it", "Italian");
      return m;
    }
    ```

1.  如果需要默认值，使用默认对象的`String[]`属性（参考*使用模型对象设置表单的默认值*配方），并用一些`Map`键初始化：

    ```java
    String[] defaultLanguages = {"en", "fr"};
    user.setLanguages(defaultLanguages);
    ```

1.  在 JSP 中，使用初始化为`@ModelAttribute` Map 的`form:checkboxes`元素：

    ```java
    <form:checkboxes items="${languages}" path="languages" />
    ```

在处理表单提交的控制器中，确保`@ModelAttribute`对象（用于保存表单值的对象）有一个相应的`String[]`属性：

```java
public class User {
  private String[] languages;
...
```

## 它是如何工作的...

Spring 的`form:checkboxes`标签使用`Map`键作为值，使用`Map`值作为显示给用户的文本来生成复选框。

仅供参考，这是生成的 HTML 代码：

```java
<span>
  <input id="languages1" name="languages" type="checkbox" value="de"/>
  <label for="languages1">German</label>
</span>
<span>
  <input id="languages2" name="languages" type="checkbox" value="en"/>
  <label for="languages2">English</label>
</span>
<span>
  <input id="languages3" name="languages" type="checkbox" value="fr"/>
  <label for="languages3">French</label>
</span>

<input type="hidden" name="_languages" value="on"/>
```

如前一个配方所示，也会生成一个`hidden`表单字段。无论复选框的状态如何，`_languages=on`都会被发送。所以，如果没有选择任何选项，Spring 将知道将语言数组的值设置为空的`String[]`对象。

## 还有更多...

为了对生成的 HTML 代码有更多的控制（以避免`label`和`span`标签），可以使用具有相同`path`属性值的多个`form:checkbox`元素：

```java
<form:checkbox path="languages" value="de" />German
<form:checkbox path="languages" value="en" />English
<form:checkbox path="languages" value="fr" />French
```

生成的 HTML 代码类似，但`hidden`属性会生成多次：

```java
<input id="languages1" name="languages" type="checkbox" value="de"/>
<input type="hidden" name="_languages" value="on"/>
German

<input id="languages2" name="languages" type="checkbox" value="en"/>
<input type="hidden" name="_languages" value="on"/>
English

<input id="languages3" name="languages" type="checkbox" value="fr"/>
<input type="hidden" name="_languages" value="on"/>
French
```

# 使用单选按钮列表

在这个配方中，你将学习如何显示一组单选按钮。当表单提交时，在控制器方法中检索选中的值。

## 如何做到这一点...

1.  在控制器中，添加一个返回`Map`对象的`@ModelAttribute`方法：

    ```java
    @ModelAttribute("countries")
    public Map<String, String>countries() {
      Map<String, String> m = new HashMap<String, String>();
      m.put("us", "United States");
      m.put("ca", "Canada");
      m.put("fr", "France");
      m.put("de", "Germany");
      return m;
    }
    ```

1.  如果需要默认值，使用默认对象的`String`属性（参考*使用模型对象设置表单的默认值*配方），并用`Map`中的一个键初始化：

    ```java
    user.setCountry("ca");
    ```

1.  在 JSP 中，使用初始化为`@ModelAttribute` Map 的`form:radiobuttons`元素：

    ```java
    <form:radiobuttons items="${countries}" path="country" />
    ```

1.  在处理表单提交的控制器中，确保`@ModelAttribute`对象（用于保存表单值的对象）有一个相应的`String`属性：

    ```java
    public class User {
      private String country;
    ...
    ```

## 它是如何工作的…

Spring 的`form:radiobuttons`标签使用`Map`键作为值，使用`Map`值作为显示给用户的文本来生成单选按钮。

为了参考，这是生成的 HTML 代码：

```java
<span>
  <input id="country1" name="country" type="radio" value="de"/>
  <label for="country1">Germany</label>
</span>
<span>
  <input id="country2" name="country" type="radio" value="fr"/>
  <label for="country2">France</label>
</span>
<span>
  <input id="country3" name="country" type="radio" value="us"/>
  <label for="country3">United States</label>
</span>
<span>
  <input id="country4" name="country" type="radio" value="ca" checked="checked"/>
  <label for="country4">Canada</label>
</span>    
```

## 还有更多…

为了更好地控制生成的 HTML 代码（以避免`label`和`span`标签），可以使用具有相同`path`属性的多个`form:radiobutton`元素：

```java
<form:radiobutton path="country" value="de" />Germany
<form:radiobutton path="country" value="fr" />France
<form:radiobutton path="country" value="us" />United States
<form:radiobutton path="country" value="ca" />Canada
```

输入字段的生成 HTML 代码与`form:radiobutton`标签生成的代码相同：

```java
<input id="country1" name="country" type="radio" value="de"/>Germany
<input id="country2" name="country" type="radio" value="fr"/>France
<input id="country3" name="country" type="radio" value="us"/>United States
<input id="country4" name="country" type="radio" value="ca" checked="checked"/>Canada
```

# 使用注解验证表单

在这个菜谱中，你将学习如何通过在模型类中使用注解直接添加约束来添加表单验证规则。例如：

```java
public class User {
  @NotEmpty
  private String firstName;
```

我们将使用 Java Bean 注解 API 和 Hibernate Validator（Hibernate ORM 独立的项目）中的约束注解。

如果验证失败，表单将再次显示给用户，并显示需要修复的错误。

## 如何做到这一点…

将约束注解添加到模型类中。在控制器方法中检查验证是否成功。在 JSP 中添加错误标签：

1.  在`pom.xml`中添加 Maven 依赖项：

    ```java
    <dependency>
      <groupId>javax.validation</groupId>
      <artifactId>validation-api</artifactId>
      <version>1.1.0.Final</version>
    </dependency>

    <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-validator</artifactId>
      <version>5.1.2.Final</version>
    </dependency>
    ```

1.  使用注解将约束添加到模型类：

    ```java
    @NotEmpty
    private String firstName;

    @Min(18) @Max(130)
    private int age;
    ```

1.  在处理表单提交的控制器方法中，将`@Valid`添加到结果对象参数，并在其后添加`BindingResult`参数：

    ```java
    @RequestMapping(value="addUser", method=RequestMethod.POST)
    public void addUser(@ModelAttribute("defaultUser") @Valid User user, BindingResult result) {
    ...
    ```

1.  在相同的控制器方法中，检查验证是否成功：

    ```java
    if(result.hasErrors()) {
      // show the form page again, with the errors
      return "addUser";  
     }
     else {
       // validation was successful, redirect to another page
       return "redirect:/home";            
     }
    ```

1.  在表单 JSP 中添加表单错误标签：

    ```java
    <form:input path="firstName" />
    <form:errors path="firstName" cssclass="error"></form:errors>

    <form:input path="age" />
    <form:errors path="age" cssclass="error"></form:errors>
    ```

## 它是如何工作的…

当表单提交时，模型类中的约束注解用于验证`User`对象。错误存储在`BindingResult`对象中。当表单再次显示时，`form:errors`元素将显示它们。以下是一个带有错误的生成 HTML 代码示例：

```java
<input id="firstName" name="firstName" type="text" value=""/>
<span id="firstName.errors" cssclass="error">may not be empty</span>

<input id="age" name="age" type="text" value="1233"/>
<span id="age.errors" cssclass="error">must be less than or equal to 120</span> 
```

## 还有更多…

一些常见的约束注解包括：

+   `@Max(120)`: 此字段必须有一个小于或等于给定数字的值

+   `@Min(18)`: 此字段必须有一个等于或大于给定数字的值

+   `@NotNull`: 此字段不能为 null

+   `@Valid`: 此字段必须是一个有效的对象

+   `@NotBlank`: 此`String`字段不能为 null，并且其修剪后的长度必须大于 0

+   `@NotEmpty`: 此集合字段不能为 null 或空

所有约束注解的完整列表可以在此处找到：

+   [`docs.oracle.com/javaee/6/tutorial/doc/gircz.html`](http://docs.oracle.com/javaee/6/tutorial/doc/gircz.html)

+   [`docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#section-builtin-constraints`](http://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#section-builtin-constraints)

要创建自定义约束注解，请参阅：

[`docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#validator-customconstraints`](http://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#validator-customconstraints).

# 上传文件

要允许用户从 HTML 表单上传文件，我们需要将表单编码设置为`multipart/form-data`。在服务器端，我们将使用 Apache Commons 库中的`fileupload`包来处理上传的文件。

## 如何实现它…

这里是上传文件到表单的步骤：

1.  在`pom.xml`中添加 Apache Commons 的`fileupload`的 Maven 依赖项：

    ```java
    <dependency>
      <groupId>commons-fileupload</groupId>
      <artifactId>commons-fileupload</artifactId>
      <version>1.3.1</version>
    </dependency>
    ```

1.  在 Spring 配置中，声明一个具有数据上传大小限制（以字节为单位）的`MultipartResolver` bean：

    ```java
    @Bean
    MultipartResolver multipartResolver() {
      CommonsMultipartResolver resolver = new CommonsMultipartResolver();
      resolver.setMaxUploadSize(500000000);
      return resolver;
    }
    ```

1.  在 JSP 中，设置 HTML 表单编码为`multipart/form-data`：

    ```java
    <form:form method="POST" modelAttribute="defaultUser" enctype="multipart/form-data">

    ```

1.  添加一个文件上传小部件：

    ```java
    <input type="file" name="file" />
    ```

1.  在处理表单提交的控制器方法中，将`MultipartFile`作为`@RequestParam`参数添加：

    ```java
    @RequestMapping(value="addUser", method=RequestMethod.POST)
    public void addUser(User user, @RequestParam("file") MultipartFile formFile) {
    ...
    ```

1.  将上传的文件保存到`Tomcat`目录下的`files`文件夹中：

    ```java
    try {
        // Create the folder "files" if necessary
        String tomcatFolderPath = System.getProperty("catalina.home");
        File filesFolder = new File(tomcatFolderPath + File.separator + "files");
        if ( ! filesFolder.exists()) {
            filesFolder.mkdirs();
        }

        // Write the uploaded file
        File file = new File(filesFolder.getAbsolutePath() + File.separator + formFile.getName());
        BufferedOutputStream fileStream = new BufferedOutputStream(new FileOutputStream(file));
        fileStream.write(formFile.getBytes());
        fileStream.close();
    } catch (Exception e) {
      // deal with the exception…
    }
    ```

## 它是如何工作的…

在 JSP 中，表单需要使用`multipart/form-data`编码才能对文件进行编码并发送。

在控制器中，我们将创建一个`files`文件夹（如果不存在）。此时，上传的文件位于服务器的内存中。我们还需要将其写入文件系统。我们使用`formFile`参数来完成这个操作。请注意，如果已存在同名文件，它将被覆盖。

## 还有更多…

要上传多个文件，你可以有多个表单字段（file1、file2 等等）以及它们对应的参数（formFile1、formFile2 等等）。也可以为多个文件上传小部件使用相同的字段名（这方便用户上传不确定数量的文件）：

```java
<input type="file" name="file" />
<input type="file" name="file" />
<input type="file" name="file" />
```

在这种情况下，我们将在控制器方法中将文件作为`MultipartFile`数组检索：

```java
@RequestMapping(value="addUser", method=RequestMethod.POST)
public void addUser(User user, @RequestParam("file") MultipartFile[] form
FileArray) {
...
```
