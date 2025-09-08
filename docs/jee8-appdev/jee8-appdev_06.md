# 第六章：使用 JSON-P 和 JSON-B 进行 JSON 处理

**JSON**，或称**JavaScript 对象表示法**，是一种人类可读的数据交换格式。正如其名所示，JSON 源自 JavaScript。Java EE 7 引入了**JSON-P**，即 Java JSON 处理 API。Java EE 8 引入了一个额外的 JSON API，即 Java API for **JSON 绑定**（**JSON-B**）。在本章中，我们将涵盖 JSON-P 和 JSON-B。

JSON-P 包括两个用于处理 JSON 的 API，即**模型 API**和**流式 API**，这两个 API 都将在本章中介绍。JSON-B 可以透明地从 JSON 字符串填充 Java 对象，以及轻松地从 Java 对象生成 JSON 字符串。

在本章中，我们将涵盖以下主题：

+   JSON-P 模型 API：

    +   使用模型 API 生成 JSON 数据

    +   使用模型 API 解析 JSON 数据

+   JSON-P 流式 API：

    +   使用流式 API 生成 JSON 数据

    +   使用流式 API 解析 JSON 数据

+   使用 JSON-B 从 JSON 填充 Java 对象

+   使用 JSON-B 从 Java 对象生成 JSON 字符串

# JSON-P 模型 API

JSON-P 模型 API 允许我们生成一个 JSON 对象的内存表示。这个 API 比本章后面讨论的流式 API 更灵活，然而，它速度较慢且需要更多的内存，这在处理大量数据时可能是一个问题。

# 使用模型 API 生成 JSON 数据

JSON-P 模型 API 的核心是`JsonObjectBuilder`类。这个类有几个重载的`add()`方法，可以用来向生成的 JSON 数据添加属性及其对应的值。

以下代码示例说明了如何使用模型 API 生成 JSON 数据：

```java
    package net.ensode.javaee8book.jsonpobject; 

    //other imports omitted for brevity. 
    import javax.inject.Named; 
    import javax.json.Json; 
    import javax.json.JsonObject; 
    import javax.json.JsonReader; 
    import javax.json.JsonWriter; 

    @Named 
    @SessionScoped 
    public class JsonpBean implements Serializable{ 

      private String jsonStr; 

      @Inject 
      private Customer customer; 

      public String buildJson() { 
        JsonObjectBuilder jsonObjectBuilder =  
            Json.createObjectBuilder(); 
        JsonObject jsonObject = jsonObjectBuilder. 
            add("firstName", "Scott"). 
            add("lastName", "Gosling"). 
            add("email", "sgosling@example.com"). 
            build(); 

        StringWriter stringWriter = new StringWriter(); 

        try (JsonWriter jsonWriter = Json.createWriter(stringWriter)) 
         { 
           jsonWriter.writeObject(jsonObject); 
         }    

        setJsonStr(stringWriter.toString()); 

        return "display_json"; 

     } 
    //getters and setters omitted for brevity 
    }  
```

如前例所示，我们通过在`JsonObjectBuilder`的实例上调用`add()`方法来生成`JsonObject`的实例。在前例中，我们看到如何通过在`JsonObjectBuilder`上调用`add()`方法将`String`值添加到我们的`JsonObject`中。`add()`方法的第一参数是生成的 JSON 对象的属性名，第二个参数对应于该属性的值。`add()`方法的返回值是另一个`JsonObjectBuilder`的实例，因此，`add()`方法的调用可以像前例那样链式调用。

上述示例是一个对应于更大 JSF 应用的 CDI 命名 bean。应用的其他部分没有显示，因为它们与讨论无关。完整的示例应用可以作为本书示例代码下载的一部分获得。

一旦我们添加了所有需要的属性，我们需要调用`JsonObjectBuilder`的`build()`方法，它返回一个实现`JsonObject`接口的类的实例。

在许多情况下，我们可能希望生成我们创建的 JSON 对象的 `String` 表示形式，以便它可以被另一个进程或服务处理。我们可以通过创建一个实现 `JsonWriter` 接口的类的实例，通过调用 `Json` 类的静态 `createWriter()` 方法并传递一个 `StringWriter` 实例作为其唯一参数来实现这一点。一旦我们有了 `JsonWriter` 实现的实例，我们需要调用它的 `writeObject()` 方法，并将我们的 `JsonObject` 实例作为其唯一参数传递。

到这一点，我们的 `StringWriter` 实例将包含我们 JSON 对象的 `String` 表示形式作为其值，因此调用其 `toString()` 方法将返回一个包含我们 JSON 对象的 `String`。

我们的特定示例将生成一个看起来像这样的 JSON 字符串：

```java
    {"firstName":"Scott","lastName":"Gosling","email":"
    sgosling@example.com "}
```

尽管在我们的示例中，我们只向我们的 JSON 对象添加了 `String` 对象，但我们并不局限于这种类型的值；`JsonObjectBuilder` 有几个重载的 `add()` 方法版本，允许我们向我们的 JSON 对象添加几种不同类型的值。

下表总结了所有可用的 `add()` 方法版本：

| `add`(`String` name, `BigDecimal` value) | 将一个 `BigDecimal` 值添加到我们的 JSON 对象中。 |
| --- | --- |
| `add`(`String` name, `BigInteger` value) | 将一个 `BigInteger` 值添加到我们的 JSON 对象中。 |
| `add`(`String` name, `JsonArrayBuilder` value) | 向我们的 JSON 对象添加一个数组。`JsonArrayBuilder` 实现允许我们创建 JSON 数组。 |
| `add`(`String` name, `JsonObjectBuilder` value) | 将另一个 JSON 对象添加到我们的原始 JSON 对象中（JSON 对象的属性值可以是其他 JSON 对象）。添加的 `JsonObject` 实现是由提供的 `JsonObjectBuilder` 参数构建的。 |
| `add`(`String` name, `JsonValue` value) | 将另一个 JSON 对象添加到我们的原始 JSON 对象中（JSON 对象的属性值可以是其他 JSON 对象）。 |
| `add`(`String` name, `String` value) | 将一个 `String` 值添加到我们的 JSON 对象中。 |
| `add`(`String` name, `boolean` value) | 将一个 `boolean` 值添加到我们的 JSON 对象中。 |
| `add`(`String` name, `double` value) | 将一个 `double` 值添加到我们的 JSON 对象中。 |
| `add`(`String` name, `int` value) | 将一个 `int` 值添加到我们的 JSON 对象中。 |
| `add`(`String` name, `long` value) | 将一个 `long` 值添加到我们的 JSON 对象中。 |

在所有情况下，`add()` 方法的第一个参数对应于我们 JSON 对象中的属性名，第二个参数对应于属性的值。

# 使用模型 API 解析 JSON 数据

在上一节中，我们看到了如何使用对象模型 API 从我们的 Java 代码中生成 JSON 数据。在本节中，我们将了解如何读取和解析现有的 JSON 数据。以下代码示例说明了如何进行此操作：

```java
    package net.ensode.javaee8book.jsonpobject; 

    //other imports omitted 
    import javax.json.Json; 
    import javax.json.JsonObject; 
    import javax.json.JsonReader; 
    import javax.json.JsonWriter; 

    @Named 
    @SessionScoped 
    public class JsonpBean implements Serializable{ 

      private String jsonStr; 

      @Inject 
      private Customer customer; 

      public String parseJson() { 
        JsonObject jsonObject; 
        try (JsonReader jsonReader = Json.createReader( 
             new StringReader(jsonStr))) { 
               jsonObject = jsonReader.readObject(); 
             } 

        customer.setFirstName( 
          jsonObject.getString("firstName")); 
        customer.setLastName( 
          jsonObject.getString("lastName")); 
        customer.setEmail(jsonObject.getString("email")); 

        return "display_parsed_json"; 
      } 

      //getters and setters omitted 

    } 
```

要解析现有的 JSON 字符串，我们需要创建一个`StringReader`对象，将包含要解析的 JSON 的`String`对象作为参数传递。然后，我们将生成的`StringReader`实例传递给`Json`类的静态`createReader()`方法。这个方法调用将返回一个`JsonReader`实例。然后，我们可以通过调用它的`readObject()`方法来获取`JsonObject`的实例。

在前面的示例中，我们使用了`getString()`方法来获取 JSON 对象中所有属性的值。此方法的第一和唯一参数是我们希望检索的属性的名称。不出所料，返回值是属性的值。

除了`getString()`方法之外，还有其他几个类似的方法可以获取其他类型的数据值。以下表格总结了这些方法：

| `get`(`Object` key) | 获取实现`JsonValue`接口的类的实例。 |
| --- | --- |
| `getBoolean`(`String` name) | 获取与给定键对应的`boolean`值。 |
| `getInt`(`String` name) | 获取与给定键对应的`int`值。 |
| `getJsonArray`(`String` name) | 获取与给定键对应的实现`JsonArray`接口的类的实例。 |
| `getJsonNumber`(`String` name) | 获取与给定键对应的实现`JsonNumber`接口的类的实例。 |
| `getJsonObject`(`String` name) | 获取与给定键对应的实现`JsonObject`接口的类的实例。 |
| `getJsonString`(`String` name) | 获取与给定键对应的实现`JsonString`接口的类的实例。 |
| `getString`(`String` name) | 获取与给定键对应的`String`。 |

在所有情况下，方法中的`String`参数对应于键名，返回值是我们希望检索的 JSON 属性值。

# JSON-P Streaming API

JSON-P Streaming API 允许从流（`java.io.OutputStream`的子类或`java.io.Writer`的子类）中顺序读取 JSON 对象。它比 Model API 更快、更节省内存，然而，代价是它的功能更加有限，因为 JSON 数据需要顺序读取，并且我们不能像 Model API 那样直接访问特定的 JSON 属性。

# 使用 Streaming API 生成 JSON 数据

JSON Streaming API 有一个`JsonGenerator`类，我们可以使用它来生成 JSON 数据并将其写入流。这个类有几个重载的`write()`方法，可以用来向生成的 JSON 数据中添加属性及其对应的值。

以下代码示例说明了如何使用 Streaming API 生成 JSON 数据：

```java
    package net.ensode.javaee8book.jsonpstreaming; 

    //other imports omitted 
    import javax.json.Json; 
    import javax.json.stream.JsonGenerator; 
    import javax.json.stream.JsonParser; 
    import javax.json.stream.JsonParser.Event; 

    @Named 
    @SessionScoped 
    public class JsonpBean implements Serializable { 

      private String jsonStr; 

      @Inject 
      private Customer customer; 

      public String buildJson() { 
        StringWriter stringWriter = new StringWriter(); 
        try (JsonGenerator jsonGenerator = 
          Json.createGenerator(stringWriter)) { 
           jsonGenerator.writeStartObject(). 
             write("firstName", "Larry"). 
             write("lastName", "Gates"). 
             write("email", "lgates@example.com"). 
             writeEnd(); 
         } 

         setJsonStr(stringWriter.toString()); 
         return "display_json"; 
     } 

     //getters and setters omitted 
    } 
```

我们通过调用 `Json` 类的 `createGenerator()` 静态方法来创建 `JsonGenerator` 实例。JSON-P API 提供了此方法的两个重载版本：一个接受一个扩展 `java.io.Writer` 类的实例（例如我们例子中使用的 `StringWriter`），另一个接受一个扩展 `java.io.OutputStream` 类的实例。

在我们开始向生成的 JSON 流添加属性之前，我们需要在 `JsonGenerator` 上调用 `writeStartObject()` 方法。此方法写入 JSON 开始对象字符（在 JSON 字符串中表示为开括号（`{`）），并返回另一个 `JsonGenerator` 实例，允许我们将 `write()` 调用链式添加到我们的 JSON 流中。

`JsonGenerator` 上的 `write()` 方法允许我们向正在生成的 JSON 流中添加属性。它的第一个参数是一个 `String`，对应于我们添加的属性名称，第二个参数是属性的值。

在我们的例子中，我们只向创建的 JSON 流添加 `String` 值，但我们并不局限于字符串；JSON-P Streaming API 提供了几个重载的 `write()` 方法，允许我们向 JSON 流添加多种不同类型的数据。以下表格总结了所有可用的 `write()` 方法版本：

| `write(String name, BigDecimal value)` | 将一个 `BigDecimal` 值写入我们的 JSON 流。 |
| --- | --- |
| `write(String name, BigInteger value)` | 将一个 `BigInteger` 值写入我们的 JSON 流 |
| `write(String name, JsonValue value)` | 将一个 JSON 对象写入我们的 JSON 流（JSON 流的属性值可以是其他 JSON 对象） |
| `write(String name, String value)` | 将一个 `String` 值写入我们的 JSON 流 |
| `write(String name, boolean value)` | 将一个 `boolean` 值写入我们的 JSON 流 |
| `write(String name, double value)` | 将一个 `double` 值写入我们的 JSON 流 |
| `write(String name, int value)` | 将一个 `int` 值写入我们的 JSON 流 |
| `write(String name, long value)` | 将一个 `long` 值写入我们的 JSON 流 |

在所有情况下，`write()` 方法的第一个参数对应于我们添加到 JSON 流中的属性名称，第二个参数对应于属性的值。

当我们完成向我们的 JSON 流添加属性后，我们需要在 `JsonGenerator` 上调用 `writeEnd()` 方法。此方法在 JSON 字符串中添加 JSON 结束对象字符（由一个闭合花括号（`}`）表示）。

在这一点上，我们的流或读取器中填充了我们生成的 JSON 数据。我们如何处理它取决于我们的应用程序逻辑。在我们的例子中，我们简单地调用了 `StringReader` 的 `toString()` 方法，以获取我们创建的 JSON 数据的字符串表示形式。

# 使用 Streaming API 解析 JSON 数据

在本节中，我们将介绍如何解析我们从流中接收到的 JSON 数据。请参考以下代码：

```java
    package net.ensode.javaee8book.jsonpstreaming; 

    //other imports omitted 
    import javax.json.Json; 
    import javax.json.stream.JsonGenerator; 
    import javax.json.stream.JsonParser; 
    import javax.json.stream.JsonParser.Event; 

    @Named 
    @SessionScoped 
    public class JsonpBean implements Serializable { 

      private String jsonStr; 

      @Inject 
      private Customer customer; 

      public String parseJson() { 

        StringReader stringReader = new StringReader(jsonStr); 

        JsonParser jsonParser = Json.createParser(stringReader); 

        Map<String, String> keyValueMap = new HashMap<>(); 
        String key = null; 
        String value = null; 

        while (jsonParser.hasNext()) { 
            JsonParser.Event event = jsonParser.next(); 

            if (event.equals(Event.KEY_NAME)) { 
                key = jsonParser.getString(); 
            } else if (event.equals(Event.VALUE_STRING)) { 
                value = jsonParser.getString(); 
            } 

            keyValueMap.put(key, value); 
        } 

        customer.setFirstName(keyValueMap.get("firstName")); 
        customer.setLastName(keyValueMap.get("lastName")); 
        customer.setEmail(keyValueMap.get("email")); 

        return "display_parsed_json"; 
      } 

      //getters and setters omitted 

    } 
```

使用 Streaming API 读取 JSON 数据的第一步是创建一个 `JsonParser` 实例，通过在 `Json` 类上调用静态方法 `createJsonParser()` 实现。`createJsonParser()` 方法有两种重载版本：一个接受一个扩展 `java.io.InputStream` 类的类的实例，另一个接受一个扩展 `java.io.Reader` 类的类的实例。在我们的示例中，我们使用后者，传递一个 `java.io.StringReader` 的实例，它是 `java.io.Reader` 的子类。

下一步是循环遍历 JSON 数据以获取要解析的数据。我们可以通过在 `JsonParser` 上调用 `hasNext()` 方法来实现这一点，如果还有更多数据要读取，则返回 `true`，否则返回 `false`。

接下来，我们需要读取流中的下一份数据。`JsonParser.next()` 方法返回一个 `JsonParser.Event` 实例，它表示我们刚刚读取的数据类型。在我们的示例中，我们只检查键名（即 `firstName`、`lastName` 和 `email`），以及相应的字符串值。我们通过将 `JsonParser.next()` 返回的事件与在 `JsonParser` 中定义的 `Event` 枚举中定义的几个值进行比较来检查我们刚刚读取的数据类型。

以下表格总结了 `JsonParser.next()` 可以返回的所有可能的事件：

| `Event.START_OBJECT` | 表示 JSON 对象的开始。 |
| --- | --- |
| `Event.END_OBJECT` | 表示 JSON 对象的结束。 |
| `Event.START_ARRAY` | 表示数组的开始。 |
| `Event.END_ARRAY` | 表示数组的结束。 |
| `Event.KEY_NAME` | 表示读取了一个 JSON 属性的名称；我们可以通过在 `JsonParser` 上调用 `getString()` 来获取键名。 |
| `Event.VALUE_TRUE` | 表示读取了一个 `true` 的布尔值。 |
| `Event.VALUE_FALSE` | 表示读取了一个 `false` 的布尔值。 |
| `Event.VALUE_NULL` | 表示读取了一个 `null` 值。 |
| `Event.VALUE_NUMBER` | 表示读取了一个 `numeric` 值。 |
| `Event.VALUE_STRING` | 表示读取了一个 `string` 值。 |

如示例所示，可以通过在 `JsonParser` 上调用 `getString()` 来检索 `String` 值。数值可以以几种不同的格式检索；以下表格总结了 `JsonParser` 中可以用来检索数值的方法：

| `getInt()` | 以 `int` 的形式检索数值。 |
| --- | --- |
| `getLong()` | 以 `long` 的形式检索数值。 |
| `getBigDecimal()` | 以 `java.math.BigDecimal` 实例的形式检索数值。 |

`JsonParser` 还提供了一个方便的 `isIntegralNumber()` 方法，如果数值可以安全地转换为 `int` 或 `long`，则返回 `true`。

我们对从流中获取的值所做的事情取决于我们的应用程序逻辑。在我们的示例中，我们将它们放置在一个 `Map` 中，然后使用这个 `Map` 来填充一个 Java 类。

# JSON 指针

Java EE 8 中引入的 JSON-P 1.1 引入了对 JSON Pointer 的支持。JSON Pointer 是一个 **互联网工程任务组** (**IETF**) 标准，它定义了一种字符串语法，用于在 JSON 文档中标识特定的值，类似于 XPath 为 XML 文档提供的功能。

JSON Pointer 的语法很简单，例如，假设我们有以下 JSON 文档：

```java
 { 
  "dateOfBirth": "1997-03-03",
  "firstName": "David",
  "lastName": "Heffelfinger",
  "middleName": "Raymond",
  "salutation": "Mr" 
 }
```

如果我们想要获取文档中 `lastName` 属性的值，要使用的 JSON Pointer 表达式将是 `"/lastName"`。

如果我们的 JSON 文档由一个数组组成，那么我们必须在属性前加上数组中的索引作为前缀，例如，要获取以下 JSON 数组中第二个元素的 `lastName` 属性，我们需要这样做：

```java
    [  
     {  
       "dateOfBirth": "1997-01-01", 
       "firstName": "David", 
       "lastName": "Delabassee", 
       "salutation": "Mr"  
      }, 
     {  
       "dateOfBirth": "1997-03-03", 
       "firstName": "David", 
       "lastName": "Heffelfinger", 
       "middleName": "Raymond", 
       "salutation": "Mr" 
     } 
    ] 
```

要这样做，JSON Pointer 表达式将是 `"/1/lastName"`。表达式开头的 `"/1"` 指的是数组中的元素索引。就像在 Java 中一样，JSON 数组是 `0` 索引的，因此，在这个例子中，我们正在获取数组中第二个元素的 `lastName` 属性的值。现在让我们看看如何使用新的 JSON-P JSON Pointer API 来执行此任务的示例：

```java
    package net.ensode.javaee8book.jsonpointer; 
    //imports omitted 

    @Path("jsonpointer") 
    public class JsonPointerDemoService { 

      private String jsonString; //initialization omitted 

      @GET 
      public String jsonPointerDemo() { 
        initializeJsonString(); //method body omitted for brevity 
        JsonReader jsonReader = Json.createReader
 (new StringReader(jsonString)); 
        JsonArray jsonArray = jsonReader.readArray(); 
        JsonPointer jsonPointer = Json.createPointer("/1/lastName"); 

        return jsonPointer.getValue(jsonArray).toString(); 
      } 
    } 
```

上述代码示例是一个使用 Java EE 的 JAX-RS API 编写的 RESTful 网络服务（有关详细信息，请参阅第十章 [4987ac18-1f2a-410c-9613-530174a64bad.xhtml]，*使用 JAX-RS 的 RESTful 网络服务*）。为了从 JSON 文档中读取属性值，我们首先需要通过在 `javax.json.Json` 上调用静态的 `createReader()` 方法来创建一个 `javax.json.JsonReader` 的实例。`createReader()` 方法接受任何实现 `java.io.Reader` 接口类的实例作为参数。在我们的例子中，我们即时创建了一个新的 `java.io.StringReader` 实例，并将我们的 JSON 字符串作为参数传递给其构造函数。

`JSON.createReader()` 有一个重载版本，它接受任何实现 `java.io.InputStream` 类的实例。

在我们的例子中，我们的 JSON 文档由一个对象数组组成，因此，我们通过在创建的 `JsonReader` 对象上调用 `readArray()` 方法来填充 `javax.json.JsonArray` 的一个实例（如果我们的 JSON 文档由一个单独的 JSON 对象组成，我们则会调用 `JsonReader.readObject()`）。

现在我们已经填充了我们的 `JsonArray` 变量，我们创建了一个 `javax.json.JsonPointer` 的实例，并用我们想要使用的 JSON Pointer 表达式初始化它。记住，我们正在寻找数组中第二个元素的 `lastName` 属性的值，因此，适当的 JSON Pointer 表达式是 `/1/lastName`。

现在我们已经创建了一个带有适当 JSON Pointer 表达式的 `JsonPointer` 实例，我们只需调用它的 `getValue()` 方法，并将我们的 `JsonArray` 对象作为参数传递，然后对结果调用 `toString()`。这个调用的返回值将是 JSON 文档中 `lastName` 属性的值（在我们的示例中是 "Heffelfinger"）。

# JSON Patch

JSON-P 1.1 还引入了对 JSON Patch 的支持，这是另一个 **互联网工程任务组** 标准，它提供了一系列可以应用于 JSON 文档的操作。JSON Patch 允许我们对 JSON 对象进行部分更新。

JSON Patch 支持以下操作：

| **JSON Patch 操作** | **描述** |
| --- | --- |
| `add` | 向 JSON 文档中添加一个元素。 |
| `remove` | 从 JSON 文档中删除一个元素。 |
| `replace` | 用新值替换 JSON 文档中的一个值。 |
| `move` | 将 JSON 文档中的一个值从其在文档中的当前位置移动到新位置。 |
| `copy` | 将 JSON 文档中的一个值复制到文档中的新位置。 |
| `test` | 验证 JSON 文档中特定位置的值是否等于指定的值。 |

JSON-P 支持所有上述 JSON Patch 操作，这些操作依赖于 JSON Pointer 表达式来定位 JSON 文档中的源和目标位置。

以下示例说明了我们如何使用 JSON-P 1.1 与 JSON Patch：

```java
    package net.ensode.javaee8book.jsonpatch; 

    //imports omitted for brevity 

    @Path("jsonpatch") 
    public class JsonPatchDemoService { 

      private String jsonString; 

      @GET 
      public Response jsonPatchDemo() { 
        initializeJsonString(); //method declaration omitted 
        JsonReader jsonReader = Json.createReader( 
            new StringReader(jsonString)); 
        JsonArray jsonArray = jsonReader.readArray(); 
        JsonPatch jsonPatch = Json.createPatchBuilder() 
                .replace("/1/dateOfBirth", "1977-01-01") 
                .build(); 
        JsonArray modifiedJsonArray =jsonPatch.apply(jsonArray); 

        return Response.ok(modifiedJsonArray.toString(), 
        MediaType.APPLICATION_JSON).build(); 
      } 
    } 
```

在本例中，让我们假设我们正在处理与之前示例相同的 JSON 文档：一个包含两个单独 JSON 对象的数组，每个对象都有一个 `dateOfBirth` 属性（以及其他属性）。

在我们的示例中，我们创建了一个 `JsonArray` 实例，就像之前一样，然后修改数组中第二个元素的 `dateOfBirth` 属性。为了做到这一点，我们通过 `javax.json.Json` 类中的静态 `createPatchBuilder()` 方法创建了一个 `javax.json.JsonPatchBuilder` 实例。在我们的示例中，我们用一个新值替换了一个属性的值。我们使用 `JsonPatch` 实例的 `replace()` 方法来完成这个操作；方法中的第一个参数是一个 JSON Pointer 表达式，指示我们要修改的属性的定位，第二个参数是属性的新值。正如其名称所暗示的，`JsonPatchBuilder` 遵循 `Builder` 设计模式，这意味着它的大多数方法都返回另一个 `JsonPatchBuilder` 实例；这允许我们在 `JsonPatchBuilder` 的结果实例上链式调用方法（在我们的示例中，我们只执行一个操作，但这不必是这种情况）。

一旦我们指定了要在我们的 JSON 对象上执行的操作，我们就通过在 `JsonPatchBuilder` 上调用 `build()` 方法来创建一个 `javax.json.JsonPatch` 实例。

一旦我们创建了补丁，我们就通过调用其 `patch()` 方法并将其作为参数传递 JSON 对象（在我们的示例中是一个 `JsonArray` 实例）来将其应用到我们的 JSON 对象上。

在我们的示例中，如何通过 JSON-P 1.1 中的 JSON Patch 支持替换 JSON 属性的值，JSON-P 支持当前 JSON Patch 所支持的所有操作。API 是直接的。有关如何使用 JSON-P 中的其他 JSON Patch 操作的详细信息，请参阅 Java EE 8 API 文档，网址为 [`javaee.github.io/javaee-spec/javadocs/`](https://javaee.github.io/javaee-spec/javadocs/)。

# 使用 JSON-B 从 JSON 填充 Java 对象

填充 Java 对象从 JSON 字符串是一个常见的编程任务。这是一个如此常见的任务，以至于已经创建了几个库来透明地填充 Java 对象从 JSON，从而让应用程序开发者免于手动编写此功能。有一些非标准的 Java 库可以完成这个任务，例如 Jackson ([`github.com/FasterXML/jackson`](https://github.com/FasterXML/jackson))、JSON-simple ([`github.com/fangyidong/json-simple`](https://github.com/fangyidong/json-simple)) 和 Gson ([`github.com/google/gson`](https://github.com/google/gson))。Java EE 8 引入了一个提供此功能的新 API，即 Java API for JSON Binding (JSON-B)。在本节中，我们将介绍如何透明地从 JSON 字符串填充 Java 对象。

以下示例展示了使用 Java API for RESTful Web Services (JAX-RS) 编写的 RESTful 网络服务。该服务在其 `addCustomer()` 方法中响应 HTTP POST 请求。此方法接受一个 `String` 参数，并且这个字符串预期包含有效的 JSON。请参考以下代码：

```java
    package net.ensode.javaee8book.jaxrs21example.service; 

    import net.ensode.javaee8book.jaxrs21example.dto.Customer; 
    import java.util.logging.Level; 
    import java.util.logging.Logger; 
    import javax.json.bind.Jsonb; 
    import javax.json.bind.JsonbBuilder; 
    import javax.ws.rs.POST; 
    import javax.ws.rs.Consumes; 
    import javax.ws.rs.Path; 
    import javax.ws.rs.core.MediaType; 
    import javax.ws.rs.core.Response; 

    @Path("/customercontroller") 
    public class CustomerControllerService { 

    private static final Logger LOG = 
    Logger.getLogger(CustomerControllerService.class.getName()); 

    @POST 
    @Consumes(MediaType.APPLICATION_JSON) 
    public Response addCustomer(String customerJson) { 
      Response response; 
      Jsonb jsonb = JsonbBuilder.create(); 
 Customer customer = jsonb.fromJson(customerJson, 
          Customer.class); 
      LOG.log(Level.INFO, "Customer object populated from JSON"); 
      LOG.log(Level.INFO, String.format("%s %s %s %s %s", 
      customer.getSalutation(), 
      customer.getFirstName(), 
      customer.getMiddleName(), 
      customer.getLastName(), 
      customer.getDateOfBirth())); 
      response = Response.ok("{}").build(); 
      return response; 
     } 

    } 
```

我们的应用服务器提供的 JSON-B 实现提供了一个实现 `JsonbBuilder` 接口的类的实例。这个类提供了一个静态的 `create()` 方法，我们可以使用它来获取 `Jsonb` 的实例。

一旦我们有了 `Jsonb` 的实例，我们可以使用它来解析 JSON 字符串并自动填充 Java 对象。这是通过其 `fromJson()` 方法完成的。`fromJson()` 方法接受一个包含我们需要解析的 JSON 数据的 `String` 作为其第一个参数，以及我们希望填充的对象的类型作为其第二个参数。在我们的例子中，我们正在填充一个包含 `firstName`、`middleName`、`lastName` 和 `dateOfBirth` 等字段的简单 `Customer` 类。JSON-B 将寻找与 Java 对象中的属性名称匹配的 JSON 属性名称，并自动用相应的 JSON 属性填充 Java 对象。这再简单不过了。

一旦我们填充了我们的 Java 对象，我们就可以用它做我们需要的任何事情。在我们的例子中，我们只是记录 Java 对象的属性，以验证它是否正确填充。

# 使用 JSON-B 从 Java 对象生成 JSON 字符串

除了从 JSON 数据填充 Java 对象之外，JSON-B 还可以从 Java 对象生成 JSON 字符串。以下示例说明了如何做到这一点：

```java
    package net.ensode.javaee8book.jsonbjavatojson.service; 

    //imports omitted for brevity 

    @Path("/customersearchcontroller") 
    public class CustomerSearchControllerService { 
      private final List<Customer> customerList = new ArrayList<>(); 

      @GET 
      @Path("{firstName}") 
      public Response getCustomerByFirstName(@PathParam("firstName")   
      String firstName) { 
        List<Customer> filteredCustomerList; 
        String jsonString; 
        initializeCustomerList(); //method declaration omitted 

        Jsonb jsonb = JsonbBuilder.create(); 

        filteredCustomerList = customerList.stream().filter( 
                customer -> customer.getFirstName().equals(firstName)). 
                collect(Collectors.toList()); 

        jsonString = jsonb.toJson(filteredCustomerList); 

        return Response.ok(jsonString).build(); 
     } 
   } 
```

在这个例子中，我们将 `Customer` 对象的 `List` 转换为 JSON。

我们选择 `List` 作为示例来说明 JSON-B 支持此功能，但当然，我们也可以将单个对象转换为它的 JSON 表示形式。

就像之前一样，我们通过调用静态方法 `javax.json.bind.JsonbBuilder.create()` 来创建一个 `javax.json.bind.Jsonb` 实例。一旦我们有了 `Jsonb` 实例，我们只需调用它的 `toJson()` 方法，将对象列表转换为等价的 JSON 表示形式。

# 摘要

在本章中，我们介绍了 Java API for JSON Processing (JSON-P)。我们展示了如何通过 JSON-P 的模型和流式 API 生成和解析 JSON 数据。此外，我们还介绍了新的 JSON-P 1.1 功能，例如对 JSON Pointer 和 JSON Patch 的支持。最后，我们介绍了如何无缝地将 Java 对象从 JSON 中填充，以及如何通过新的 JSON-B API 简单地生成 Java 对象的 JSON 字符串。
