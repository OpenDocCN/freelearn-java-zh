

# 第四章：JSON 处理和 JSON 绑定

**JavaScript 对象表示法**（**JSON**）是一种人类可读的数据交换格式。正如其名称所暗示的，JSON 源自 JavaScript。Jakarta EE 提供了对两种不同的 JSON 操作 API 的支持，即**Jakarta JSON Processing**，这是一个低级 API，允许细粒度控制，以及**Jakarta JSON Binding**，这是一个高级 API，允许我们轻松地从 JSON 数据填充 Java 对象，以及快速从 Java 对象生成 JSON 格式的数据。在本章中，我们将介绍 JSON 处理和 JSON 绑定。

JSON 处理包括两个用于处理 JSON 的 API：**模型 API**和**流式 API**。这两个 API 都将在本章中介绍。JSON 绑定透明地将 Java 对象从 JSON 字符串中填充，以及轻松地从 Java 对象生成 JSON 字符串。

在本章中，我们将涵盖以下主题：

+   Jakarta JSON 处理

+   Jakarta JSON 绑定

注意

本章的示例源代码可以在 GitHub 上找到，链接为[`github.com/PacktPublishing/Jakarta-EE-Application-Development/tree/main/ch04_src`](https://github.com/PacktPublishing/Jakarta-EE-Application-Development/tree/main/ch04_src)。

# Jakarta JSON 处理

在以下章节中，我们将讨论如何使用 Jakarta JSON Processing 提供的两个 API（即模型 API 和流式 API）来处理 JSON 数据。我们还将讨论如何使用 JSON 指针从 JSON 数据中检索值，以及如何通过 JSON 补丁部分修改 JSON 数据。

## JSON 处理模型 API

JSON 处理模型 API 允许我们生成 JSON 对象的内存表示。与本章后面讨论的流式 API 相比，此 API 更灵活。然而，它较慢且需要更多内存，这在处理大量数据时可能是一个问题。

### 使用模型 API 生成 JSON 数据

JSON 处理模型 API 的核心是`JsonObjectBuilder`类。此类有几个重载的`add()`方法，可用于将属性及其对应值添加到生成的 JSON 数据中。

以下代码示例说明了如何使用模型 API 生成 JSON 数据：

```java
package com.ensode.jakartaeebook.jsonpobject;
//imports omitted for brevity
@Path("jsonpmodel")
public class JsonPModelResource {
  private static final Logger LOG =
    Logger.getLogger(JsonPModelResource.class.getName());
  @Path("build")
  @GET
  @Produces(MediaType.APPLICATION_JSON)
  public String jsonpModelBuildJson() {
    JsonObject jsonObject = Json.createObjectBuilder().
      add("firstName", "Scott").
      add("lastName", "Gosling").
      add("email", "sgosling@example.com").
      build();
    StringWriter stringWriter = new StringWriter();
    try (JsonWriter jsonWriter = Json.createWriter(
      stringWriter)) {
      jsonWriter.writeObject(jsonObject);
    }
    return stringWriter.toString();
  }
}
```

如示例所示，我们通过在`JsonObjectBuilder`实例上调用`add()`方法来生成`JsonObject`实例。在我们的示例中，我们看到如何通过在`JsonObjectBuilder`上调用`add()`方法将`String`值添加到我们的`JsonObject`中。`add()`方法的第一参数是生成的 JSON 对象的属性名，第二个参数对应于该属性的值。`add()`方法的返回值是另一个`JsonObjectBuilder`实例；因此，可以对`add()`方法进行链式调用，如示例所示。

注意

上述示例是对应于更大的 Jakarta RESTful Web Services 应用程序的 RESTful 网络服务。由于它们与讨论无关，因此未显示应用程序的其他部分。完整的示例应用程序可以从本书的 GitHub 仓库中获取，网址为 [`github.com/PacktPublishing/Jakarta-EE-Application-Development`](https://github.com/PacktPublishing/Jakarta-EE-Application-Development)。

一旦我们添加了所有所需的属性，我们需要调用 `JsonObjectBuilder` 的 `build()` 方法，它返回一个实现 `JsonObject` 接口的类的实例。

在许多情况下，我们可能希望生成我们创建的 JSON 对象的 `String` 表示形式，以便它可以被另一个进程或服务处理。我们可以通过创建一个实现 `JsonWriter` 接口的类的实例，通过调用 `Json` 类的静态 `createWriter()` 方法，并将 `StringWriter` 的实例作为其唯一参数来实现这一点。一旦我们有了 `JsonWriter` 实现的实例，我们需要调用其 `writeObject()` 方法，并将我们的 `JsonObject` 实例作为其唯一参数传递。

在这一点上，我们的 `StringWriter` 实例将包含我们 JSON 对象的字符串表示形式作为其值，因此调用其 `toString()` 方法将返回一个包含我们 JSON 对象的 `String`。|

我们的具体示例将生成一个看起来像这样的 JSON 字符串：

```java
{"firstName":"Scott","lastName":"Gosling","email":"sgosling@example.com"}
```

尽管在我们的示例中我们只向 JSON 对象添加了 `String` 对象，但我们并不局限于这种类型的值；`JsonObjectBuilder` 有几个重载的 `add()` 方法版本，允许我们向 JSON 对象添加几种不同类型的值。|

下表总结了所有可用的 `add()` 方法版本：

| **JsonObjectBuilder.add() 方法** | **描述** |
| --- | --- |
| `add(String name,` `BigDecimal value)` | 将 `BigDecimal` 值添加到我们的 JSON 对象中。 |
| `add(String name,` `BigInteger value)` | 将 `BigInteger` 值添加到我们的 JSON 对象中。 |
| `add(String name,` `JsonArrayBuilder value)` | 将数组添加到我们的 JSON 对象中。`JsonArrayBuilder` 实现允许我们创建 JSON 数组。 |
| `add(String name,` `JsonObjectBuilder value)` | 将另一个 JSON 对象添加到我们的原始 JSON 对象中（JSON 对象的属性值可以是其他 JSON 对象）。添加的 `JsonObject` 实现是从提供的 `JsonObjectBuilder` 参数构建的。 |
| `add(String name,` `JsonValue value)` | 将另一个 JSON 对象添加到我们的原始 JSON 对象中（JSON 对象的属性值可以是其他 JSON 对象）。 |
| `add(String name,` `String value)` | 将 `String` 值添加到我们的 JSON 对象中。 |
| `add(String name,` `boolean value)` | 将 `boolean` 值添加到我们的 JSON 对象中。 |
| `add(String name,` `double value)` | 将 `double` 值添加到我们的 JSON 对象中。 |
| `add(String name,` `int value)` | 将 `int` 值添加到我们的 JSON 对象中。 |
| `add(String name, long value)` | 向我们的 JSON 对象添加一个`long`值。 |

表 4.1 – JsonObjectBuilder add()方法

在所有情况下，`add()`方法的第一参数对应于我们 JSON 对象中的属性名称，第二个参数对应于属性的值。

### 使用 Model API 解析 JSON 数据

在上一节中，我们看到了如何使用对象模型 API 从我们的 Java 代码中生成 JSON 数据。在本节中，我们将看到如何读取和解析现有的 JSON 数据。以下代码示例说明了如何进行此操作：

```java
package com.ensode.jakartaeebook.jsonpobject;
//imports omitted for brevity
@Path("jsonpmodel")
public class JsonPModelResource {
  @Path("parse")
  @POST
  @Produces(MediaType.TEXT_PLAIN)
  @Consumes(MediaType.APPLICATION_JSON)
  public String jsonpModelParseJson(String jsonStr) {
    LOG.log(Level.INFO, String.format(
      "received the following JSON string: %s", jsonStr));
    Customer customer = new Customer();
    JsonObject jsonObject;
    try (JsonReader jsonReader = Json.createReader(new
    StringReader(jsonStr))) {
      jsonObject = jsonReader.readObject();
    }
    customer.setFirstName(
      jsonObject.getString("firstName"));
    customer.setLastName(jsonObject.getString("lastName"));
    customer.setEmail(jsonObject.getString("email"));
    return customer.toString();
  }
}
```

要解析现有的 JSON 字符串，我们需要创建一个`StringReader`对象，将包含要解析的 JSON 的`String`对象作为参数传递。然后，我们将生成的`StringReader`实例传递给`Json`类的静态`createReader()`方法。此方法调用将返回一个`JsonReader`实例。然后，我们可以通过调用其上的`readObject()`方法来获取`JsonObject`实例。

在这个例子中，我们使用`getString()`方法来获取我们 JSON 对象中所有属性的值。此方法唯一的第一个参数是我们希望检索的属性的名称；不出所料，返回值是该属性的值。

除了`getString()`方法之外，还有其他几个类似的方法可以获取其他类型的数据值。以下表格总结了这些方法：

| JsonObject 方法 | **描述** |
| --- | --- |
| `get(Object key)` | 获取实现`JsonValue`接口的类的实例 |
| `getBoolean(String name)` | 获取与给定键对应的`boolean`值 |
| `getInt(String name)` | 获取与给定键对应的`int`值 |
| `getJsonArray(String name)` | 获取与给定键对应的实现`JsonArray`接口的类的实例 |
| `getJsonNumber(String name)` | 获取与给定键对应的实现`JsonNumber`接口的类的实例 |
| `getJsonObject(String name)` | 获取与给定键对应的实现`JsonObject`接口的类的实例 |
| `getJsonString(String name)` | 获取与给定键对应的实现`JsonString`接口的类的实例 |
| `getString(String Name)` | 获取与给定键对应的`String` |

表 4.2 – 从 JSON 数据中检索值的 JsonObject 方法

在所有情况下，方法的`String`参数对应于键名，返回值是我们希望检索的 JSON 属性值。

## JSON 处理流式 API

JSON 处理流式 API 允许从流（`java.io.OutputStream` 的子类或 `java.io.Writer` 的子类）中顺序读取 JSON 对象。它比模型 API 更快、更节省内存；然而，与模型 API 相比，直接访问特定的 JSON 属性不那么直接。当使用流式 API 时，我们需要使用 JSON Pointer 和 JSON Patch 来检索或修改 JSON 数据中的特定值。

### 使用流式 API 生成 JSON 数据

JSON 流式 API 有一个 `JsonGenerator` 类，我们可以使用它来生成 JSON 数据并将其写入流。此类有几个重载的 `write()` 方法，可以用来向生成的 JSON 数据中添加属性及其对应的值。

以下代码示例说明了如何使用流式 API 生成 JSON 数据：

```java
package com.ensode.jakartaeebook.jsonpstreaming;
//imports omitted for brevity
@Path("jsonpstreaming")
public class JsonPStreamingResource {
  @Path("build")
  @GET
  @Produces(MediaType.APPLICATION_JSON)
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
    return stringWriter.toString();
  }
}
```

我们通过调用 `Json` 类的 `createGenerator()` 静态方法来创建一个 `JsonGenerator` 实例。JSON 处理 API 提供了此方法的两个重载版本；一个接受一个扩展 `java.io.Writer` 类（例如 `StringWriter`，我们在示例中使用）的类的实例，另一个接受一个扩展 `java.io.OutputStream` 类的类的实例。

在我们开始向生成的 JSON 流添加属性之前，我们需要在 `JsonGenerator` 上调用 `writeStartObject()` 方法。此方法写入 JSON 起始对象字符（在 JSON 字符串中表示为开括号 `{`）并返回另一个 `JsonGenerator` 实例，允许我们将 `write()` 调用链式添加到我们的 JSON 流中。

`JsonGenerator` 上的 `write()` 方法允许我们向正在生成的 JSON 流中添加属性；它的第一个参数是我们添加的属性的名称对应的 `String`，第二个参数是属性的值。

在我们的示例中，我们只向创建的 JSON 流添加 `String` 值。然而，我们并不局限于字符串；流式 API 提供了几个重载的 `write()` 方法，允许我们向 JSON 流添加多种不同类型的数据。以下表格总结了所有可用的 `write()` 方法版本：

| `JsonGenerator.write() 方法` | **描述** |
| --- | --- |
| `write(String name,` `BigDecimal value)` | 将一个 `BigDecimal` 值写入我们的 JSON 流 |
| `write(String name,` `BigInteger value)` | 将一个 `BigInteger` 值写入我们的 JSON 流 |
| `write(String name,` `JsonValue value)` | 将一个 JSON 对象写入我们的 JSON 流（JSON 流的属性值可以是其他 JSON 对象） |
| `write(String name,` `String value)` | 将一个 `String` 值写入我们的 JSON 流 |
| `write(String name,` `boolean value)` | 将一个 `boolean` 值写入我们的 JSON 流 |
| `write(String name,` `double value)` | 将一个 `double` 值写入我们的 JSON 流 |
| `write(String name,` `int value)` | 将一个 `int` 值写入我们的 JSON 流 |
| `write(String name,` `long value)` | 将 `long` 值写入我们的 JSON 流 |

表 4.3 – JsonGenerator write() 方法

在所有情况下，`write()` 方法的第一个参数对应于我们添加到 JSON 流中的属性名称，第二个参数对应于属性的值。

一旦我们完成向 JSON 流添加属性，我们需要在 `JsonGenerator` 上调用 `writeEnd()` 方法；此方法添加 JSON 结束对象字符（在 JSON 字符串中由一个关闭花括号（}）表示）。

在这一点上，我们的流或读取器被我们生成的 JSON 数据填充。我们如何处理它取决于我们的应用程序逻辑。在我们的例子中，我们简单地调用了 `StringReader` 的 `toString()` 方法来获取我们创建的 JSON 数据的字符串表示。

### 使用流式 API 解析 JSON 数据

在本节中，我们将介绍如何解析从流中接收到的 JSON 数据。

以下示例说明了我们如何使用流式 API 从 JSON 数据中填充 Java 对象。

```java
package com.ensode.jakartaeebook.jsonpstreaming;
//imports omitted for brevity
@Path("jsonpstreaming")
public class JsonPStreamingResource {
  @Path("parse")
  @POST
  @Produces(MediaType.TEXT_PLAIN)
  @Consumes(MediaType.APPLICATION_JSON)
  public String parseJson(String jsonStr) {
    Customer customer = new Customer();
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
    return customer.toString();
  }
}
```

要使用流式 API 读取 JSON 数据，我们首先需要通过在 `Json` 类上调用静态 `createJsonParser()` 方法来创建一个 `JsonParser` 实例。`createJsonParser()` 方法有两种重载版本：一个接受一个扩展 `java.io.InputStream` 的类的实例，另一个接受一个扩展 `java.io.Reader` 的类的实例。在我们的例子中，我们使用后者，传递一个 `java.io.StringReader` 的实例，它是 `java.io.Reader` 的一个子类。

下一步是遍历 JSON 数据以获取要解析的数据。我们可以通过在 `JsonParser` 上调用 `hasNext()` 方法来实现这一点，该方法在还有更多数据要读取时返回 true，否则返回 false。

然后，我们需要读取流中的下一份数据，`JsonParser.next()` 方法返回一个 `JsonParser.Event` 实例，该实例指示我们刚刚读取的数据类型。在我们的例子中，我们只检查键名称（即 `firstName`、`lastName` 和 `email`），以及相应的字符串值。我们通过将 `JsonParser.next()` 返回的事件与 `JsonParser.Event` 中定义的几个值进行比较来检查我们刚刚读取的数据类型。

下表总结了 `JsonParser.next()` 可以返回的所有可能的事件：

| `JsonParser Event` | **描述** |
| --- | --- |
| `Event.START_OBJECT` | 表示 JSON 对象的开始 |
| `Event.END_OBJECT` | 表示 JSON 对象的结束 |
| `Event.START_ARRAY` | 表示数组的开始 |
| `Event.END_ARRAY` | 表示数组的结束 |
| `Event.KEY_NAME` | 表示读取了 JSON 属性的名称；我们可以通过在 `JsonParser` 上调用 `getString()` 来获取键名称 |
| `Event.VALUE_TRUE` | 表示读取了布尔值 `true` |
| `Event.VALUE_FALSE` | 表示读取了布尔值 `false` |
| `Event.VALUE_NULL` | 表示读取到了空值 |
| `Event.VALUE_NUMBER` | 表示读取到了数值 |
| `Event.VALUE_STRING` | 表示读取到了字符串值 |

表 4.4 – JsonParser 事件

如示例所示，可以通过在 `JsonParser` 上调用 `getString()` 来检索 `String` 类型的值。数值可以以几种不同的格式检索。以下表格总结了 `JsonParser` 中可以用来检索数值的方法：

| `JsonParser method` | **描述** |
| --- | --- |
| `getInt()` | 以 `int` 类型检索数值 |
| `getLong()` | 以 `long` 类型检索数值 |
| `getBigDecimal()` | 以 `java.math.BigDecimal` 实例的形式检索数值 |

表 4.5 – 用于检索数值的 JsonParser 方法

`JsonParser` 还提供了一个方便的 `isIntegralNumber()` 方法，如果数值可以安全地转换为 `int` 或 `long` 类型，则返回 `true`。

我们对从流中获取的值所采取的操作取决于我们的应用程序逻辑；在我们的示例中，我们将它们放入一个 `Map` 中，然后使用该 `Map` 来填充一个 Java 类。

### 从数据中检索 JSON Pointer 值

Jakarta JSON Processing 支持 JSON Pointer，这是一个 **互联网工程任务组** (**IETF**) 标准，它定义了一种字符串语法，用于在 JSON 文档中标识特定的值，类似于 XPath 为 XML 文档提供的功能。

JSON Pointer 的语法很简单。例如，假设我们有一个以下 JSON 文档：

```java
{
  "dateOfBirth": "1997-03-03",
  "firstName": "David",
  "lastName": "Heffelfinger",
  "middleName": "Raymond",
  "salutation": "Mr"
}
```

如果我们想要获取文档中 `lastName` 属性的值，所使用的 JSON Pointer 表达式应该是 `"/lastName"`。

如果我们的 JSON 文档由一个数组组成，那么我们必须在属性前加上数组中的索引；例如，假设我们想要获取以下 JSON 数组中第二个元素的 `lastName` 属性：

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

要这样做，所使用的 JSON Pointer 表达式应该是 `"/1/lastName"`。表达式开头的 `"/1"` 指的是数组中的元素索引。就像在 Java 中一样，JSON 数组是从 0 开始索引的；因此，在这个例子中，我们正在获取数组第二个元素中 `lastName` 属性的值。现在让我们看看如何使用 JSON Pointer API 来执行此任务的示例：

```java
package com.ensode.jakartaeebook.jsonpointer;
//imports omitted for brevity
@Path("jsonpointer")
public class JsonPointerDemoService {
  private String jsonString; //initialization omitted
  @GET
  @Produces(MediaType.TEXT_PLAIN)
  public String jsonPointerDemo() {
    JsonReader jsonReader = Json.createReader(
    new StringReader(jsonString));
    JsonArray jsonArray = jsonReader.readArray();
    JsonPointer jsonPointer = Json.createPointer("/1/lastName");
    return jsonPointer.getValue(jsonArray).toString();
  }
}
```

以下代码示例是一个使用 Jakarta RESTful Web Services 编写的 RESTful 网络服务。为了从 JSON 文档中读取属性值，我们首先需要通过在 `jakarta.json.Json` 上调用静态的 `createReader()` 方法来创建一个 `jakarta.json.JsonReader` 实例。`createReader()` 方法接受任何实现 `java.io.Reader` 接口的对象实例作为参数。在我们的示例中，我们动态创建了一个新的 `java.io.StringReader` 实例，并将我们的 JSON 字符串作为参数传递给其构造函数。

注意

`JSON.createReader()` 有一个重载版本，它接受任何实现 `java.io.InputStream` 类的实例。

在我们的例子中，我们的 JSON 文档由一个对象数组组成；因此，我们通过在创建的 `JsonReader` 对象上调用 `readArray()` 方法来填充 `jakarta.json.JsonArray` 实例。（如果我们的 JSON 文档由一个单独的 JSON 对象组成，我们将调用 `JsonReader.readObject()` 而不是 `readArray()`。）

现在我们已经填充了 `JsonArray` 变量，我们创建了一个 `jakarta.json.JsonPointer` 实例，并用我们想要使用的 JSON 指针表达式初始化它。回想一下，我们正在寻找数组第二个元素中 `lastName` 属性的值；因此，适当的 JSON 指针表达式是 `/1/lastName`。

现在我们已经使用适当的 JSON 指针表达式创建了一个 `JsonPointer` 实例，我们只需调用它的 `getValue()` 方法，并将我们的 `JsonArray` 对象作为参数传递；然后，我们在结果上调用 `toString()`，这个调用的返回值将是 JSON 文档中 `"Heffelfinger"` 的 `lastName` 属性的值（在我们的例子中）。

### 使用 JSON 补丁更新 JSON 数据值

Jakarta JSON Processing 包括对 JSON 补丁的支持，这是另一个 IETF 标准。它提供了一系列可以应用于 JSON 文档的操作。JSON 补丁允许我们对 JSON 对象执行部分更新。

JSON 补丁支持以下操作：

| **JSON** **补丁操作** | **描述** |
| --- | --- |
| 添加 | 向 JSON 文档中添加一个元素 |
| 删除 | 从 JSON 文档中删除一个元素 |
| 替换 | 将 JSON 文档中的一个值替换为新值 |
| 移动 | 将 JSON 文档中的一个值从其在文档中的当前位置移动到新位置 |
| 复制 | 将 JSON 文档中的一个值复制到文档中的新位置 |
| 测试 | 验证 JSON 文档中特定位置的值是否等于指定的值 |

表 4.6 – JSON 补丁操作

Jakarta JSON Processing 支持所有上述 JSON 补丁操作，这些操作依赖于 JSON 指针表达式来定位 JSON 文档中的源和目标位置。

以下示例说明了我们如何使用 Jakarta JSON Processing 的 JSON 补丁：

```java
package com.ensode.jakartaeebook.jsonpatch
//imports omitted for brevity
@Path("jsonpatch")
public class JsonPatchDemoService {
  private String jsonString; //initialization omitted
  @GET
  public Response jsonPatchDemo() {
    JsonReader jsonReader = Json.createReader(
      new StringReader(jsonString));
    JsonArray jsonArray = jsonReader.readArray();
    JsonPatch jsonPatch = Json.createPatchBuilder()
      .replace("/1/dateOfBirth", "1977-01-01")
      .build();
    JsonArray modifiedJsonArray = jsonPatch.apply(
jsonArray);
    return Response.ok(modifiedJsonArray.toString(),
      MediaType.APPLICATION_JSON).build();
  }
}
```

在这个例子中，让我们假设我们正在处理与上一个例子相同的 JSON 文档，一个包含两个单独 JSON 对象的数组，每个对象都有一个 `dateOfBirth` 属性（以及其他属性）。

在我们的示例中，我们像之前一样创建了一个 `JsonArray` 实例，然后修改了数组中第二个元素的 `dateOfBirth`。为了做到这一点，我们通过 `jakarta.json.Json` 类中的静态 `createPatchBuilder()` 方法创建了一个 `jakarta.json.JsonPatchBuilder` 实例。在我们的示例中，我们用新值替换了一个属性的值；我们使用 `JsonPatch` 实例的 `replace()` 方法来完成这个操作。方法中的第一个参数是一个 JSON Pointer 表达式，指示我们要修改的属性的定位，第二个参数是属性的新值。正如其名称所暗示的，`JsonPatchBuilder` 遵循 `Builder` 设计模式，这意味着其大多数方法返回另一个 `JsonPatchBuilder` 实例；这允许我们在 `JsonPatchBuilder` 的结果实例上链式调用方法（在我们的示例中，我们只执行了一个操作，但这不必是这种情况）。一旦我们指定了要在我们的 JSON 对象上执行的操作，我们通过在 `JsonPatchBuilder` 上调用 `build()` 方法创建一个 `jakarta.json.JsonPatch` 实例。

一旦我们创建了补丁，我们通过调用其 `patch()` 方法并将 JSON 对象（在我们的示例中为 `JsonArray` 实例）作为参数传递，将其应用到我们的 JSON 对象上。

我们的示例展示了如何在 Jakarta JSON-Processing 中通过 JSON Patch 支持，用另一个值替换 JSON 属性的值。Jakarta JSONProcessing 支持所有标准的 JSON Patch 操作。有关如何使用 JSON Processing 与其他 JSON Patch 操作的详细信息，请参阅 [`jakarta.ee/specifications/platform/10/apidocs/`](https://jakarta.ee/specifications/platform/10/apidocs/) 的 Jakarta EE API 文档。

现在我们已经看到了如何直接使用 JSON Processing 操作 JSON 数据，我们将关注如何使用 Jakarta JSON Binding 将 JSON 数据绑定，这是一个更高级的 API，它允许我们快速轻松地完成常见任务。

# Jakarta JSON Binding

Jakarta JSON Binding 是一个高级 API，它允许我们几乎无缝地从 JSON 数据填充 Java 对象，以及轻松地从 Java 对象生成 JSON 格式的数据。

## 使用 JSON Binding 从 JSON 填充 Java 对象

一个常见的编程任务是填充 Java 对象来自 JSON 字符串。这是一个如此常见的任务，以至于已经创建了几个库来透明地填充 Java 对象来自 JSON，从而让应用程序开发者免于手动编写此功能。存在几个完成此任务的非标准 Java 库，例如 Jackson ([`github.com/FasterXML/jackson`](https://github.com/FasterXML/jackson))、json-simple ([`code.google.com/archive/p/json-simple/`](https://code.google.com/archive/p/json-simple/)) 或 Gson ([`github.com/google/gson`](https://github.com/google/gson))。Jakarta EE 包含一个提供此功能的标准化 API，即 JSON Binding。在本节中，我们将介绍如何从 JSON 字符串透明地填充 Java 对象。

以下示例展示了使用 Jakarta RESTful Web Services 编写的 RESTful Web 服务。该服务在其 `addCustomer()` 方法中响应 HTTP POST 请求。`addCustomer()` 方法接受一个 `String` 参数；预期此字符串包含有效的 JSON：

```java
package com.ensode.jakartaeebook.jsonbjsontojava.service;
//imports omitted for brevity
@Path("/customercontroller")
public class CustomerControllerService {
  @POST
  @Consumes(MediaType.APPLICATION_JSON)
  public String addCustomer(String customerJson) {
    Jsonb jsonb = JsonbBuilder.create();
    Customer customer = jsonb.fromJson(customerJson,
      Customer.class);
    return customer.toString();
  }
}
```

我们的应用服务器提供的 JSON Binding 实现提供了一个实现 `JsonbBuilder` 接口的类的实例；这个类提供了一个静态的 `create()` 方法，我们可以使用它来获取 `Jsonb` 的实例。

一旦我们有了 `Jsonb` 实例，我们可以用它来解析 JSON 字符串并自动填充 Java 对象。这是通过它的 `fromJson()` 方法完成的。`fromJson()` 方法接受一个包含需要解析的 JSON 数据的 `String` 作为其第一个参数，以及我们希望填充的对象类型作为其第二个参数。在我们的例子中，我们正在填充一个包含 `firstName`、`middleName`、`lastName` 和 `dateOfBirth` 等字段的简单 `Customer` 类。Jakarta JSON Binding 将寻找与 Java 对象中的属性名称匹配的 JSON 属性名称，并自动用相应的 JSON 属性填充 Java 对象。这再简单不过了！

一旦我们填充了我们的 Java 对象，我们就可以用它做任何我们需要做的事情。在我们的例子中，我们只是将 `Customer` 对象的 `String` 表示形式返回给客户端。

## 使用 JSON Binding 从 Java 对象生成 JSON 数据

除了从 JSON 数据填充 Java 对象之外，JSON Binding 还可以从 Java 对象生成 JSON 字符串。以下示例说明了如何做到这一点：

```java
package com.ensode.jakartaeebook.jsonbjavatojson.service;
//imports omitted for brevity
@Path("/customercontroller")
public class CustomerControllerService {
  @GET
  public String getCustomerAsJson() {
    String jsonString;
    DateTimeFormatter dateTimeFormatter =
      DateTimeFormatter.ofPattern("d/MM/yyyy");
    Customer customer = new Customer("Mr", "David",
      "Raymond", "Heffelfinger",
      LocalDate.parse("03/03/1997", dateTimeFormatter));
    Jsonb jsonb = JsonbBuilder.create();
    jsonString = jsonb.toJson(customer);
    return jsonString;
  }
}
```

在这个例子中，我们正在从 `Customer` 对象生成 JSON 数据。

就像之前一样，我们通过调用静态方法 `jakarta.json.bind.JsonbBuilder.create()` 来创建一个 `jakarta.json.bind.Jsonb` 实例。一旦我们有了 `Jsonb` 实例，我们只需调用它的 `toJson()` 方法，将对象列表转换为等效的 JSON 表示形式。

# 摘要

在本章中，我们介绍了如何使用两个 Jakarta EE API（JSON Processing 和 JSON Binding）来处理 JSON 数据。

我们涵盖了以下主题：

+   我们看到了如何使用 JSON Processing 的模型 API 生成和解析 JSON 数据

+   我们还探讨了如何使用 JSON Processing 的流式 API 生成和解析 JSON 数据

+   此外，我们还介绍了如何使用 JSON Pointer 从 JSON 数据中提取值

+   同样，我们也看到了如何使用 JSON Patch 在 JSON 数据中更新特定值

+   最后，我们介绍了如何使用 Jakarta JSON Binding 轻松地从 JSON 数据填充 Java 对象，以及如何轻松地从 Java 对象生成 JSON 数据

在处理 RESTful Web 服务和微服务时，JSON 格式的数据已经成为一种事实上的标准。Jakarta JSON Processing 和 JSON Binding API 为此提供了出色的支持，正如本章所示。
