# 第六章 JSON-P 处理

**JSON**，或称**JavaScript Object Notation**，是一种人类可读的数据交换格式。正如其名所示，JSON 源自 JavaScript。Java EE 7 引入了 JSON-P，即 Java API for JSON Processing 作为**Java Specification Request**（JSR）353。

传统上，XML 一直是不同系统之间数据交换的首选格式。虽然 XML 无疑非常流行，但近年来，JSON 作为一种可能更简单的数据交换格式，其地位正在上升。存在几个 Java 库可以从 Java 代码中解析和生成 JSON 数据。Java EE 通过 Java API for JSON Processing（JSON-P）标准化了这一功能。

JSON-P 包括两个用于处理 JSON 的 API——Model API 和 Streaming API；这两个 API 将在本章中介绍。

在本章中，我们将介绍以下主题：

+   JSON-P 模型 API

    +   使用 Model API 生成 JSON 数据

    +   使用 Model API 解析 JSON 数据

+   JSON-P 流式 API

    +   使用流式 API 生成 JSON 数据

    +   使用流式 API 解析 JSON 数据

# JSON-P 模型 API

JSON-P 模型 API 允许我们生成一个预加载的、完全可遍历的、内存中的 JSON 对象表示。与本章中讨论的流式 API 相比，此 API 更灵活。然而，JSON-P 模型 API 较慢且需要更多内存，这在处理大量数据时可能成为问题。

## 使用 Model API 生成 JSON 数据

JSON-P 模型 API 的核心是`JsonObjectBuilder`类。此类有几个重载的`add()`方法，可用于向生成的 JSON 数据添加属性及其对应值。

以下代码示例说明了如何使用 Model API 生成 JSON 数据：

```java
packagepackagenet.ensode.glassfishbook.jsonpobject;

//other imports omitted for brevity.
importimportjavax.inject.Named;
importimportjavax.json.Json;
importimportjavax.json.JsonObject;
importimportjavax.json.JsonReader;
importimportjavax.json.JsonWriter;

@Named
@SessionScoped
public class JsonpBean implements Serializable{

    private String jsonStr;

    @Inject
    private Customer customer;

    public String buildJson() {
JsonObjectBuilderjsonObjectBuilder =

Json.createObjectBuilder();
JsonObjectjsonObject = jsonObjectBuilder.
 add("firstName", "Scott").
 add("lastName", "Gosling").
 add("email", "sgosling@example.com").
 build();

StringWriterstringWriter = new StringWriter();

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

### 注意

我们的例子是一个 CDI 命名 bean，对应于一个更大的 JSF 应用程序；应用程序的其他部分没有显示，因为它们与讨论无关。完整的示例应用程序包含在本书的示例代码下载中。

如前例所示，我们通过在`JsonObjectBuilder`上调用`add()`方法来生成一个`JsonObject`实例。在我们的例子中，我们看到如何通过在`JsonObjectBuilder`上调用`add()`方法将`String`值添加到我们的`JsonObject`中。`add()`方法的第一参数是生成的 JSON 对象属性名，第二个参数对应于该属性的值。`add()`方法的返回值是另一个`JsonObjectBuilder`实例；因此，可以对`add()`方法进行链式调用，如示例所示。

一旦添加了所有所需的属性，我们需要调用`JsonObjectBuilder`的`build()`方法，该方法返回实现`JsonObject`接口的类的实例。

在许多情况下，我们可能希望生成我们创建的 JSON 对象的字符串表示形式，以便它可以被其他进程或服务处理。我们可以通过创建一个实现 `JsonWriter` 接口的类的实例；调用 `Json` 类的静态 `createWriter()` 方法，并将 `StringWriter` 的实例作为其唯一参数传递。一旦我们有了 `JsonWriter` 实现的实例，我们需要调用其 `writeObject()` 方法，并将我们的 `JsonObject` 实例作为其唯一参数传递。

到目前为止，我们的 `StringWriter` 实例将包含我们 JSON 对象的字符串表示形式作为其值。因此，调用其 `toString()` 方法将返回一个包含我们的 JSON 对象的字符串值。

我们的特定示例将生成如下外观的 JSON 字符串：

```java
{"firstName":"Scott","lastName":"Gosling","email":
    "sgosling@example.com"}
```

尽管在我们的示例中，我们只向 JSON 对象添加了 `String` 对象，但我们并不局限于这种类型的值。`JsonObjectBuilder` 有几个 `add()` 方法的重载版本，允许我们向 JSON 对象添加多种不同类型的值。

以下表格总结了所有可用的 `add()` 方法版本：

| JsonObjectBuilder 方法 | 描述 |
| --- | --- |
| `add(String name, BigDecimal value)` | 此方法将一个 `BigDecimal` 值添加到我们的 JSON 对象中。 |
| `add(String name, BigInteger value)` | 此方法将一个 `BigInteger` 值添加到我们的 JSON 对象中。 |
| `add(String name, JsonArrayBuilder value)` | 此方法将一个数组添加到我们的 JSON 对象中。`JsonArrayBuilder` 实现允许我们创建 JSON 数组。 |
| `add(String name, JsonObjectBuilder value)` | 此方法将另一个 JSON 对象添加到我们的原始 JSON 对象中（JSON 对象的属性值可以是其他 JSON 对象）。添加的 `JsonObject` 实现是从提供的 `JsonObjectBuilder` 参数构建的。 |
| `add(String name, JsonValue value)` | 此方法将另一个 JSON 对象添加到我们的原始 JSON 对象中（JSON 对象的属性值可以是其他 JSON 对象）。 |
| `add(String name, String value)` | 此方法将一个 `String` 值添加到我们的 JSON 对象中。 |
| `add(String name, boolean value)` | 此方法将一个 `boolean` 值添加到我们的 JSON 对象中。 |
| `add(String name, double value)` | 此方法将一个 `double` 值添加到我们的 JSON 对象中。 |
| `add(String name, int value)` | 此方法将一个 `int` 值添加到我们的 JSON 对象中。 |
| `add(String name, long value)` | 此方法将一个 `long` 值添加到我们的 JSON 对象中。 |

在所有情况下，`add()` 方法的第一个参数对应于我们 JSON 对象中的属性名称，第二个参数对应于属性的值。

## 使用 Model API 解析 JSON 数据

在上一节中，我们看到了如何使用 Model API 从我们的 Java 代码生成 JSON 数据。在本节中，我们将了解如何读取和解析现有的 JSON 数据。以下代码示例说明了如何进行此操作：

```java
packagepackagenet.ensode.glassfishbook.jsonpobject;

//other imports omitted
importimportjavax.json.Json;
importimportjavax.json.JsonObject;
importimportjavax.json.JsonReader;
importimportjavax.json.JsonWriter;

@Named
@SessionScoped
public class JsonpBean implements Serializable{

    private String jsonStr;

    @Inject
    private Customer customer;

    public String parseJson() {
JsonObjectjsonObject;

try (JsonReaderjsonReader = Json.createReader(
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

要解析现有的 JSON 字符串，我们需要创建一个`StringReader`对象，将包含要解析的 JSON 数据的`String`对象作为参数传递。然后，我们将生成的`StringReader`实例传递给`Json`类的静态`createReader()`方法。此方法调用将返回一个`JsonReader`实例。然后，我们可以通过调用`readObject()`方法来获取`JsonObject`的实例。

在我们的示例中，我们使用了`getString()`方法来获取 JSON 对象中所有属性的值；此方法的第一和唯一参数是我们希望检索的属性名称。不出所料，返回值是属性的值。

除了`getString()`方法之外，还有其他几个类似的方法可以用来获取其他类型的数据值。以下表格总结了这些方法：

| JsonObject 方法 | 描述 |
| --- | --- |
| `get(Object key)` | 此方法返回实现`JsonValue`接口的类的实例。 |
| `getBoolean(String name)` | 此方法返回与给定键对应的`boolean`值。 |
| `getInt(String name)` | 此方法返回与给定键对应的`int`值。 |
| `getJsonArray(String name)` | 此方法返回与给定键对应的实现`JsonArray`接口的类的实例。 |
| `getJsonNumber(String name)` | 此方法返回与给定键对应的实现`JsonNumber`接口的类的实例。 |
| `getJsonObject(String name)` | 此方法返回与给定键对应的实现`JsonObject`接口的类的实例。 |
| `getJsonString(String name)` | 此方法返回与给定键对应的实现`JsonString`接口的类的实例。 |
| `getString(String Name)` | 此方法返回与给定键对应的`String`。 |

在所有情况下，方法的`String`参数对应于键名，返回值是我们希望检索的 JSON 属性值。

# JSON-P Streaming API

JSON-P Streaming API 允许从流（`java.io.OutputStream`的子类或`java.io.Writer`的子类）中顺序读取 JSON 对象。它比 Model API 更快、更节省内存。然而，它的缺点是功能更有限，因为 JSON 数据需要顺序读取，我们无法像 Model API 那样直接访问特定的 JSON 属性。

## 使用 Streaming API 生成 JSON 数据

JSON Streaming API 有一个`JsonGenerator`类，我们可以使用它来生成 JSON 数据并将其写入流。这个类有几个重载的`write()`方法，可以用来向生成的 JSON 数据中添加属性及其对应的值。

以下代码示例说明了如何使用 Streaming API 生成 JSON 数据：

```java
packagepackagenet.ensode.glassfishbook.jsonpstreaming;

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

StringWriterstringWriter = new StringWriter();
try (JsonGeneratorjsonGenerator =

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

我们通过调用`Json`类的`createGenerator()`静态方法来创建一个`JsonGenerator`实例。JSON-P 流式 API 提供了`createGenerator()`方法的两个重载版本；一个接受一个扩展`java.io.Writer`类（例如我们示例中使用的`StringWriter`）的类的实例，另一个接受一个扩展`java.io.OutputStream`类的类的实例。

在我们开始向生成的 JSON 流添加属性之前，我们需要在`JsonGenerator`上调用`writeStartObject()`方法。此方法写入 JSON 开始对象字符（在 JSON 字符串中由一个开括号`{`表示）并返回另一个`JsonGenerator`实例，允许我们将`write()`调用链式添加到我们的 JSON 流中。

`JsonGenerator`中的`write()`方法允许我们向生成的 JSON 流添加属性。它的第一个参数是一个`String`，对应于我们添加的属性的名称，第二个参数是属性的值。

在我们的示例中，我们只向创建的 JSON 流添加`String`值；然而，我们并不局限于`Strings`。JSON-P 流式 API 提供了几个重载的`write()`方法，允许我们向 JSON 流添加多种不同类型的数据。以下表格总结了所有可用的`write()`方法版本：

| JsonGenerator write()方法 | 描述 |
| --- | --- |
| `write(String name, BigDecimal value)` | 此方法将一个`BigDecimal`值写入我们的 JSON 流。 |
| `write(String name, BigInteger value)` | 此方法将一个`BigInteger`值写入我们的 JSON 流。 |
| `write(String name, JsonValue value)` | 此方法将一个 JSON 对象写入我们的 JSON 流（JSON 流的属性值可以是其他 JSON 对象）。 |
| `write(String name, String value)` | 此方法将一个`String`值写入我们的 JSON 流。 |
| `write(String name, boolean value)` | 此方法将一个`boolean`值写入我们的 JSON 流。 |
| `write(String name, double value)` | 此方法将一个`double`值写入我们的 JSON 流。 |
| `write(String name, int value)` | 此方法将一个`int`值写入我们的 JSON 流。 |
| `write(String name, long value)` | 此方法将一个`long`值写入我们的 JSON 流。 |

在所有情况下，`write()`方法的第一个参数对应于我们添加到 JSON 流中的属性的名称，第二个参数对应于属性的值。

一旦我们完成向我们的 JSON 流添加属性，我们需要在`JsonGenerator`上调用`writeEnd()`方法；此方法添加 JSON 结束对象字符（在 JSON 字符串中由一个闭合花括号`}`表示）。

在这个阶段，我们的流或读取器已经包含了我们生成的 JSON 数据；我们如何处理它取决于我们的应用程序逻辑。在我们的例子中，我们简单地调用了 `StringReader` 类的 `toString()` 方法来获取我们创建的 JSON 数据的 `String` 表示形式。

## 使用流式 API 解析 JSON 数据

在上一节中，我们看到了如何使用流式 API 从我们的 Java 代码中生成 JSON 数据。在本节中，我们将看到如何读取和解析我们从流中接收到的现有 JSON 数据。以下代码示例说明了如何做到这一点：

```java
package net.ensode.glassfishbook.jsonpstreaming;

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

StringReaderstringReader = new StringReader(jsonStr);

JsonParserjsonParser = Json.createParser(stringReader);

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

为了使用流式 API 读取 JSON 数据，我们首先需要通过在 `Json` 类上调用静态 `createJsonParser()` 方法来创建一个 `JsonParser` 实例。`createJsonParser()` 方法有两种重载版本；一个接受一个扩展 `java.io.InputStream` 的类的实例，另一个接受一个扩展 `java.io.Reader` 的类的实例。在我们的例子中，我们使用后者，通过传递一个 `java.io.StringReader` 的实例来实现，它是 `java.io.Reader` 的一个子类。 

下一步是遍历 JSON 数据以获取要解析的数据。我们可以通过在 `JsonParser` 上调用 `hasNext()` 方法来实现这一点，如果还有更多数据要读取，则返回 `true`，否则返回 `false`。

然后，我们需要读取流中的下一份数据。`JsonParser.next()` 方法返回一个 `JsonParser.Event` 实例，该实例指示我们刚刚读取的数据类型。在我们的例子中，我们只检查键名（即，“`firstName`”，“`lastName`”和“`email`”）以及相应的字符串值。我们可以通过将 `JsonParser.next()` 返回的事件与在 `JsonParser` 中定义的 `Event` 枚举中定义的几个值进行比较来检查我们刚刚读取的数据类型。

以下表格总结了 `JsonParser.next()` 可以返回的所有可能的常量：

| JsonParser 事件常量 | 描述 |
| --- | --- |
| `Event.START_OBJECT` | 此常量表示 JSON 对象的开始。 |
| `Event.END_OBJECT` | 此常量表示 JSON 对象的结束。 |
| `Event.START_ARRAY` | 此常量表示数组的开始 |
| `Event.END_ARRAY` | 此常量表示数组的结束。 |
| `Event.KEY_NAME` | 此常量表示读取的 JSON 属性的名称。我们可以通过在 `JsonParser` 上调用 `getString()` 来获取键名。 |
| `Event.VALUE_TRUE` | 此常量表示读取了一个 `boolean` 值为 `true`。 |
| `Event.VALUE_FALSE` | 此常量表示读取了一个 `boolean` 值为 `false`。 |
| `Event.VALUE_NULL` | 此常量表示读取了一个 `null` 值。 |
| `Event.VALUE_NUMBER` | 此常量表示读取了一个数值。 |
| `Event.VALUE_STRING` | 此常量表示读取了一个字符串值。 |

如示例所示，可以通过在 `JsonParser` 上调用 `getString()` 来检索 `String` 值。数值可以以几种不同的格式检索；以下表格总结了 `JsonParser` 中可以用来检索数值的方法：|

| JsonParser 方法 | 描述 |
| --- | --- |
| `getInt()` | 此方法检索数值作为 `int` 类型的值。 |
| `getLong()` | 此方法检索数值作为 `long` 类型的值。 |
| `getBigDecimal()` | 此方法检索数值作为 `java.math.BigDecimal` 类型的实例。 |

`JsonParser` 还提供了一个方便的 `isIntegralNumber()` 方法，如果数值可以安全地转换为 `int` 或 `long` 类型，则返回 `true`。|

我们对从流中获取的值所采取的操作取决于我们的应用程序逻辑。在我们的示例中，我们将它们放入 `Map` 中，然后使用该 `Map` 来填充一个 Java 类。|

# 摘要 |

在本章中，我们介绍了 Java API for JSON Processing (JSON-P)。我们介绍了 JSON-P 的两个主要 API：模型 API 和流式 API。|

我们展示了如何通过 JSON-P 的模型 API 生成 JSON 数据，特别是 `JsonBuilder` 类。我们还介绍了如何通过 `JsonReader` 类通过 JSON-P 的模型 API 解析 JSON 数据。|

此外，我们解释了如何通过使用 `JsonGenerator` 类来生成 JSON 数据，通过 JSON-P 的流式 API。|

最后，我们介绍了如何通过 JSON-P 的流式 API 解析 JSON 数据，特别是通过 `JsonParser` 类。|

在下一章中，我们将介绍 Java API for WebSocket。
