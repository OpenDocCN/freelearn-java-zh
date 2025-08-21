# 第四章：高级映射

到目前为止，我们已经学习了将对象映射到 Lucene 索引的基本知识。我们看到了如何处理与相关实体和嵌入对象的关系。然而，可搜索的字段大多是简单的字符串数据。

在本章中，我们将探讨如何有效地映射其他数据类型。我们将探讨 Lucene 为索引分析实体以及可以自定义该过程的 Solr 组件的过程。我们将了解如何调整每个字段的重要性，使按相关性排序更有意义。最后，我们将根据运行时实体的状态条件性地确定是否索引实体。

# 桥梁

Java 类中的成员变量可能是无数的自定义类型。通常，您也可以在自己的数据库中创建自定义类型。使用 Hibernate ORM，有数十种基本类型，可以构建更复杂的类型。

然而，在 Lucene 索引中，一切最终都归结为字符串。当你为搜索映射其他数据类型的字段时，该字段被转换为字符串表示。在 Hibernate Search 术语中，这种转换背后的代码称为桥梁。默认桥梁为您处理大多数常见情况，尽管您有能力为自定义场景编写自己的桥梁。

## 一对一自定义转换

最常见的映射场景是一个 Java 属性与一个 Lucene 索引字段绑定。`String`变量显然不需要任何转换。对于大多数其他常见数据类型，它们作为字符串的表达方式相当直观。

### 映射日期字段

`Date`值被调整为 GMT 时间，然后以`yyyyMMddHHmmssSSS`的格式存储为字符串。

尽管这一切都是自动发生的，但你确实可以选择显式地将字段注解为`@DateBridge`。当你不想索引到确切的毫秒时，你会这样做。这个注解有一个必需的元素`resolution`，让你从`YEAR`、`MONTH`、`DAY`、`HOUR`、`MINUTE`、`SECOND`或`MILLISECOND`（正常默认）中选择一个粒度级别。

可下载的`chapter4`版本的 VAPORware Marketplace 应用现在在`App`实体中添加了一个`releaseDate`字段。它被配置为仅存储日期，而不存储具体的一天中的任何时间。

```java
...
@Column
@Field
@DateBridge(resolution=Resolution.DAY)
private Date releaseDate;
...
```

### 处理 null 值

默认情况下，无论其类型如何，带有 null 值的字段都不会被索引。然而，您也可以自定义这种行为。`@Field`注解有一个可选元素`indexNullAs`，它控制了映射字段的 null 值的处理。

```java
...
@Column
@Field(indexNullAs=Field.DEFAULT_NULL_TOKEN)
private String description;
...
```

此元素的默认设置是`Field.DO_NOT_INDEX_NULL`，这导致 null 值在 Lucene 索引中被省略。然而，当使用`Field.DEFAULT_NULL_TOKEN`时，Hibernate Search 将使用一个全局配置的值索引该字段。

这个值的名称是`hibernate.search.default_null_token`，它是在`hibernate.cfg.xml`（对于传统的 Hibernate ORM）或`persistence.xml`（对于作为 JPA 提供者的 Hibernate）中设置的。如果这个值没有配置，那么空字段将被索引为字符串`"_null_"`。

### 注意

您可以使用这个机制对某些字段进行空值替换，而保持其他字段的行为。然而，`indexNullAs`元素只能与在全局级别配置的那个替代值一起使用。如果您想要为不同的字段或不同的场景使用不同的空值替代，您必须通过自定义桥接实现那个逻辑（在下一小节中讨论）。

### 自定义字符串转换

有时您需要在将字段转换为字符串值方面具有更多的灵活性。而不是依赖内置的桥接自动处理，您可以创建自己的自定义桥接。

#### StringBridge

要将对单个 Java 属性的映射映射到一个索引字段上，您的桥接可以实现 Hibernate Search 提供的两个接口中的一个。第一个，`StringBridge`，是为了在属性和字符串值之间进行单向翻译。

假设我们的`App`实体有一个`currentDiscountPercentage`成员变量，表示该应用程序正在提供的任何促销折扣（例如，*25% 折扣!*）。为了更容易进行数学运算，这个字段被存储为浮点数(*0.25f*)。然而，如果我们想要使折扣可搜索，我们希望它们以更易读的百分比格式(*25*)进行索引。

为了提供这种映射，我们首先需要创建一个桥接类，实现`StringBridge`接口。桥接类必须实现一个`objectToString`方法，该方法期望将我们的`currentDiscountPercentage`属性作为输入参数：

```java
import org.hibernate.search.bridge.StringBridge;

/** Converts values from 0-1 into percentages (e.g. 0.25 -> 25) */
public class PercentageBridge implements StringBridge {
   public String objectToString(Object object) {
      try {
         floatfieldValue = ((Float) object).floatValue();
         if(fieldValue< 0f || fieldValue> 1f) return "0";
         int percentageValue = (int) (fieldValue * 100);
 return Integer.toString(percentageValue);
      } catch(Exception e) {
         // default to zero for null values or other problems
 return "0";
      }
   }

}
```

`objectToString`方法按照预期转换输入，并返回其`String`表示。这将是由 Lucene 索引的值。

### 注意

请注意，当给定一个空值时，或者当遇到任何其他问题时，这个方法返回一个硬编码的`"0"`。自定义空值处理是创建字段桥接的另一个可能原因。

要在索引时间调用这个桥接类，请将`@FieldBridge`注解添加到`currentDiscountPercentage`属性上：

```java
...
@Column
@Field
@FieldBridge(impl=PercentageBridge.class)
private float currentDiscountPercentage;
...

```

### 注意

这个实体字段是一个原始`float`，然而桥接类却在与一个`Float`包装对象一起工作。为了灵活性，`objectToString`接受一个泛型`Object`参数，该参数必须转换为适当的类型。然而，多亏了自动装箱，原始值会自动转换为它们的对象包装器。

#### TwoWayStringBridge

第二个接口用于将单个变量映射到单个字段，`TwoWayStringBridge`，提供双向翻译，在值及其字符串表示之间进行翻译。

实现`TwoWayStringBridge`的方式与刚刚看到的常规`StringBridge`接口类似。唯一的区别是，这个双向版本还要求有一个`stringToObject`方法，用于反向转换：

```java
...
public Object stringToObject(String stringValue) {
   return Float.parseFloat(stringValue) / 100;
}
...
```

### 提示

只有在字段将成为 Lucene 索引中的`ID`字段（即，用`@Id`或`@DocumentId`注解）时，才需要双向桥。

#### 参数化桥

为了更大的灵活性，可以向桥接类传递配置参数。为此，您的桥接类应该实现`ParameterizedBridge`接口，以及`StringBridge`或`TwoWayStringBridge`。然后，该类必须实现一个`setParameterValues`方法来接收这些额外的参数。

为了说明问题，假设我们想让我们的示例桥接能够以更大的精度写出百分比，而不是四舍五入到整数。我们可以传递一个参数，指定要使用的小数位数：

```java
public class PercentageBridge implements StringBridge,
 ParameterizedBridge {

 public static final String DECIMAL_PLACES_PROPERTY =
 "decimal_places";
 private int decimalPlaces = 2;  // default

   public String objectToString(Object object) {
      String format = "%." + decimalPlaces + "g%n";
      try {
         float fieldValue = ((Float) object).floatValue();
         if(fieldValue< 0f || fieldValue> 1f) return "0";
         return String.format(format, (fieldValue * 100f));
      } catch(Exception e) {
         return String.format(format, "0");
      }
   }
 public void setParameterValues(Map<String, String> parameters) {
      try {
         this.decimalPlaces = Integer.parseInt(
            parameters.get(DECIMAL_PLACES_PROPERTY) );
      } catch(Exception e) {}
   }

}
```

我们桥接类的这个版本期望收到一个名为`decimal_places`的参数。它的值存储在`decimalPlaces`成员变量中，然后在`objectToString`方法中使用。如果没有传递这样的参数，那么将使用两个小数位来构建百分比字符串。

`@FieldBridge`注解中的`params`元素是实际传递一个或多个参数的机制：

```java
...
@Column
@Field
@FieldBridge(
   impl=PercentageBridge.class,
 params=@Parameter(
 name=PercentageBridge.DECIMAL_PLACES_PROPERTY, value="4")
)
private float currentDiscountPercentage;
...
```

### 注意

请注意，所有`StringBridge`或`TwoWayStringBridge`的实现都必须是线程安全的。通常，您应该避免任何共享资源，并且只通过`ParameterizedBridge`参数获取额外信息。

## 使用 FieldBridge 进行更复杂的映射

迄今为止所涵盖的桥接类型是将 Java 属性映射到字符串索引值的最简单、最直接的方法。然而，有时您需要更大的灵活性，因此有一些支持自由形式的字段桥接变体。

### 将单个变量拆分为多个字段

有时，类属性与 Lucene 索引字段之间的期望关系可能不是一对一的。例如，假设一个属性表示文件名。然而，我们希望能够不仅通过文件名搜索，还可以通过文件类型（即文件扩展名）搜索。一种方法是从文件名属性中解析文件扩展名，从而使用这个变量创建两个字段。

`FieldBridge`接口允许我们这样做。实现必须提供一个`set`方法，在这个例子中，它从文件名字段中解析文件类型，并将其分别存储：

```java
import org.apache.lucene.document.Document;
import org.hibernate.search.bridge.FieldBridge;
import org.hibernate.search.bridge.LuceneOptions;

public class FileBridge implements FieldBridge {

 public void set(String name, Object value, 
 Document document, LuceneOptionsluceneOptions) {
      String file = ((String) value).toLowerCase();
      String type = file.substring(
      file.indexOf(".") + 1 ).toLowerCase();
 luceneOptions.addFieldToDocument(name+".file", file, document);
 luceneOptions.addFieldToDocument(name+".file_type", type, 
 document);
   }

}
```

`luceneOptions`参数是与 Lucene 交互的帮助对象，`document`表示我们正在添加字段的 Lucene 数据结构。我们使用`luceneOptions.addFieldToDocument()`将字段添加到索引，而不必完全理解 Lucene API 的细节。

传递给`set`的`name`参数代表了被索引的实体名称。注意我们用这个作为基础来声明要添加的两个实体的名称（也就是说，对于文件名，使用`name+".file"`；对于文件类型，使用`name+".file_type"`）。

最后，`value` 参数是指当前正在映射的字段。就像在`Bridges`部分看到的`StringBridge`接口一样，这里的函数签名使用了一个通用的`Object`以提高灵活性。必须将值转换为其适当的类型。

要应用`FieldBridge`实现，就像我们已经看到的其他自定义桥接类型一样，使用`@FieldBridge`注解：

```java
...
@Column
@Field
@FieldBridge(impl=FileBridge.class)
private String file;
...
```

### 将多个属性合并为一个字段

实现`FieldBridge`接口的自定义桥接也可以用于相反的目的，将多个属性合并为一个索引字段。为了获得这种灵活性，桥接必须应用于*类*级别而不是*字段*级别。当以这种方式使用`FieldBridge`接口时，它被称为**类桥接**，并替换了整个实体类的常规映射机制。

例如，考虑我们在 VAPORware Marketplace 应用程序中处理`Device`实体时可以采取的另一种方法。而不是将`manufacturer`和`name`作为单独的字段进行索引，我们可以将它们合并为一个`fullName`字段。这个类桥接仍然实现`FieldBridge`接口，但它会将两个属性合并为一个索引字段，如下所示：

```java
public class DeviceClassBridge implements FieldBridge {

   public void set(String name, Object value, 
         Document document, LuceneOptionsluceneOptions) {
      Device device = (Device) value;
      String fullName = device.getManufacturer()
         + " " + device.getName();
 luceneOptions.addFieldToDocument(name + ".name", 
 fullName, document);
   }

}
```

而不是在`Device`类的任何特定字段上应用注解，我们可以在类级别应用一个`@ClassBridge`注解。注意字段级别的 Hibernate Search 注解已经被完全移除，因为类桥接将负责映射这个类中的所有索引字段。

```java
@Entity
@Indexed
@ClassBridge(impl=DeviceClassBridge.class)
public class Device {

   @Id
   @GeneratedValue
   private Long id;

   @Column
   private String manufacturer;

   @Column
   private String name;

   // constructors, getters and setters...
}

```

### TwoWayFieldBridge

之前我们看到了简单的`StringBridge`接口有一个`TwoWayStringBridge`对应接口，为文档 ID 字段提供双向映射能力。同样，`FieldBridge`接口也有一个`TwoWayFieldBridge`对应接口出于相同原因。当你将字段桥接接口应用于 Lucene 用作 ID 的属性（即，用`@Id`或`@DocumentId`注解）时，你必须使用双向变体。

`TwoWayStringBridge`接口需要与`StringBridge`相同的`objectToString`方法，以及与`FieldBridge`相同的`set`方法。然而，这个双向版本还需要一个`get`对应方法，用于从 Lucene 检索字符串表示，并在真实类型不同时进行转换：

```java
...
public Object get(String name, Object value, Document document) {
   // return the full file name field... the file type field
   // is not needed when going back in the reverse direction
   return = document.get(name + ".file");
}
public String objectToString(Object object) {
   // "file" is already a String, otherwise it would need conversion
      return object;
}
...
```

# 分析

当一个字段被 Lucene 索引时，它会经历一个称为**分析**的解析和转换过程。在第三章《执行查询》中，我们提到了默认的**分析器**会分词字符串字段，如果你打算对该字段进行排序，则应该禁用这种行为。

然而，在分析过程中可以实现更多功能。Apache Solr 组件可以组装成数百种组合。 它们可以在索引过程中以各种方式操纵文本，并打开一些非常强大的搜索功能的大门。

为了讨论可用的 Solr 组件，或者如何将它们组装成自定义分析器定义，我们首先必须了解 Lucene 分析的三个阶段：

+   字符过滤

+   标记化

+   标记过滤

分析首先通过应用零个或多个**字符过滤器**进行，这些过滤器在处理之前去除或替换字符。 过滤后的字符串然后进行**标记化**，将其拆分为更小的标记，以提高关键字搜索的效率。 最后，零个或多个**标记过滤器**在将它们保存到索引之前去除或替换标记。

### 注意

这些组件由 Apache Solr 项目提供，总共有三十多个。 本书无法深入探讨每一个，但我们可以查看三种类型的一些关键示例，并了解如何一般地应用它们。

所有这些 Solr 分析器组件的完整文档可以在[`wiki.apache.org/solr/AnalyzersTokenizersTokenFilters`](http://wiki.apache.org/solr/AnalyzersTokenizersTokenFilters)找到，Javadocs 在[`lucene.apache.org/solr/api-3_6_1`](http://lucene.apache.org/solr/api-3_6_1)。

## 字符过滤

定义自定义分析器时，字符过滤是一个可选步骤。如果需要此步骤，只有三种字符过滤类型可用：

+   `MappingCharFilterFactory`：此过滤器将字符（或字符序列）替换为特定定义的替换文本，例如，您可能会将*1*替换为*one*，*2*替换为*two*，依此类推。

    字符（或字符序列）与替换值之间的映射存储在一个资源文件中，该文件使用标准的`java.util.Properties`格式，位于应用程序的类路径中的某个位置。对于每个属性，键是查找的序列，值是映射的替换。

    这个映射文件相对于类路径的位置被传递给`MappingCharFilterFactory`定义，作为一个名为`mapping`的参数。传递这个参数的确切机制将在*定义和选择分析器*部分中详细说明。

+   `PatternReplaceCharFilter`：此过滤器应用一个通过名为`pattern`的参数传递的正则表达式。 任何匹配项都将用通过`replacement`参数传递的静态文本字符串替换。

+   `HTMLStripCharFilterFactory`：这个极其有用的过滤器移除 HTML 标签，并将转义序列替换为其通常的文本形式（例如，`&gt;`变成`>`）。

## 标记化

在定义自定义分析器时，字符和标记过滤器都是可选的，您可以组合多种过滤器。然而，`tokenizer`组件是唯一的。分析器定义必须包含一个，最多一个。

总共有 10 个`tokenizer`组件可供使用。一些说明性示例包括：

+   `WhitespaceTokenizerFactory`：这个组件只是根据空白字符分割文本。例如，*hello world* 被分词为 *hello* 和 *world*。

+   `LetterTokenizerFactory`：这个组件的功能与`WhitespaceTokenizrFactory`类似，但这个分词器还会在非字母字符处分割文本。非字母字符完全被丢弃，例如，*please don't go*被分词为*please*, *don*, *t*, 和 *go*。

+   `StandardTokenizerFactory`：这是默认的`tokenizer`，在未定义自定义分析器时自动应用。它通常根据空白字符分割，丢弃多余字符。例如，*it's 25.5 degrees outside!!!* 变为 *it's*, *25.5*, *degrees*, 和 *outside*。

### 小贴士

当有疑问时，`StandardTokenizerFactory`几乎总是合理的选择。

## 分词过滤器

到目前为止，分析器功能的最大多样性是通过分词过滤器实现的，Solr 提供了二十多个选项供单独或组合使用。以下是更有用的几个示例：

+   `StopFilterFactory`：这个过滤器简单地丢弃“停用词”，或者根本没有人想要对其进行关键词查询的极其常见的词。列表包括 *a*, *the*, *if*, *for*, *and*, *or* 等（Solr 文档列出了完整列表）。

+   `PhoneticFilterFactory`：当你使用主流搜索引擎时，你可能会注意到它在处理你的拼写错误时非常智能。这样做的一种技术是寻找与搜索关键字听起来相似的单词，以防它被拼写错误。例如，如果你本想搜索*morning*，但误拼为*mourning*，搜索仍然能匹配到意图的词条！这个分词过滤器通过与实际分词一起索引音似字符串来实现这一功能。该过滤器需要一个名为`encoder`的参数，设置为支持的字符编码算法名称（`"DoubleMetaphone"`是一个合理的选择）。

+   `SnowballPorterFilterFactory`：词干提取是一个将分词转化为其根形式的过程，以便更容易匹配相关词汇。Snowball 和 Porter 指的是词干提取算法。例如，单词 *developer* 和 *development* 都可以被分解为共同的词干 *develop*。因此，Lucene 能够识别这两个较长词汇之间的关系（即使没有一个词汇是另一个的子串！）并能返回两个匹配项。这个过滤器有一个参数，名为 `language`（例如，`"English"`）。

## 定义和选择分析器

**分析器定义**将一些这些组件的组合成一个逻辑整体，在索引实体或单个字段时可以引用这个整体。分析器可以在静态方式下定义，也可以根据运行时的一些条件动态地组装。

### 静态分析器选择

定义自定义分析器的任何方法都以在相关持久类上的`@AnalyzerDef`注解开始。在我们的`chapter4`版本的 VAPORware Marketplace 应用程序中，让我们定义一个自定义分析器，用于与`App`实体的`description`字段一起使用。它应该移除任何 HTML 标签，并应用各种分词过滤器以减少杂乱并考虑拼写错误：

```java
...
@AnalyzerDef(
 name="appAnalyzer",
 charFilters={    
      @CharFilterDef(factory=HTMLStripCharFilterFactory.class) 
   },
 tokenizer=@TokenizerDef(factory=StandardTokenizerFactory.class),
 filters={ 
      @TokenFilterDef(factory=StandardFilterFactory.class),
      @TokenFilterDef(factory=StopFilterFactory.class),
      @TokenFilterDef(factory=PhoneticFilterFactory.class, 
            params = {
         @Parameter(name="encoder", value="DoubleMetaphone")
            }),
      @TokenFilterDef(factory=SnowballPorterFilterFactory.class, 
            params = {
      @Parameter(name="language", value="English") 
      })
   }
)
...
```

`@AnalyzerDef`注解必须有一个名称元素设置，正如之前讨论的，分析器必须始终包括一个且只有一个分词器。

`charFilters`和`filters`元素是可选的。如果设置，它们分别接收一个或多个工厂类列表，用于字符过滤器和分词过滤器。

### 提示

请注意，字符过滤器和分词过滤器是按照它们列出的顺序应用的。在某些情况下，更改顺序可能会影响最终结果。

`@Analyzer`注解用于选择并应用一个自定义分析器。这个注解可以放在个别字段上，或者放在整个类上，影响每个字段。在这个例子中，我们只为`desciption`字段选择我们的分析器定义：

```java
...
@Column(length = 1000)
@Field
@Analyzer(definition="appAnalyzer")
private String description;
...
```

在一个类中定义多个分析器是可能的，通过将它们的`@AnalyzerDef`注解包裹在一个复数`@AnalyzerDefs`中来实现：

```java
...
@AnalyzerDefs({
   @AnalyzerDef(name="stripHTMLAnalyzer", ...),
   @AnalyzerDef(name="applyRegexAnalyzer", ...)
})
...
```

显然，在后来应用`@Analyzer`注解的地方，其定义元素必须与相应的`@AnalyzerDef`注解的名称元素匹配。

### 注意

`chapter4`版本的 VAPORware Marketplace 应用程序现在会从客户评论中移除 HTML。如果搜索包括关键词*span*，例如，不会在包含`<span>`标签的评论中出现假阳性匹配。

Snowball 和音译过滤器被应用于应用描述中。关键词*mourning*找到包含单词*morning*的匹配项，而*development*的搜索返回了描述中包含*developers*的应用程序。

### 动态分析器选择

可以等到运行时为字段选择一个特定的分析器。最明显的场景是一个支持不同语言的应用程序，为每种语言配置了分析器定义。您希望根据每个对象的言语属性选择适当的分析器。

为了支持这种动态选择，对特定的字段或整个类添加了`@AnalyzerDiscriminator`注解。这个代码段使用了后者的方法：

```java
@AnalyzerDefs({
   @AnalyzerDef(name="englishAnalyzer", ...),
   @AnalyzerDef(name="frenchAnalyzer", ...)
})
@AnalyzerDiscriminator(impl=CustomerReviewDiscriminator.class)
public class CustomerReview {
   ...
   @Field
   private String language;
   ...
}
```

有两个分析器定义，一个是英语，另一个是法语，类`CustomerReviewDiscriminator`被宣布负责决定使用哪一个。这个类必须实现`Discriminator`接口，并它的`getAnalyzerDefinitionName`方法：

```java
public class LanguageDiscriminator implements Discriminator {

 public String getAnalyzerDefinitionName(Object value, 
 Object entity, String field) {
      if( entity == null || !(entity instanceofCustomerReview) ) {
         return null;
      }
      CustomerReview review = (CustomerReview) entity;
      if(review.getLanguage() == null) {
         return null;
       } else if(review.getLanguage().equals("en")) {
         return "englishAnalyzer";
       } else if(review.getLanguage().equals("fr")) {
         return "frenchAnalyzer";
       } else {
         return null;
      }
   }

}
```

如果`@AnalyzerDiscriminator`注解放在字段上，那么其当前对象的值会自动作为第一个参数传递给`getAnalyzerDefinitionName`。如果注解放在类本身上，则传递`null`值。无论如何，第二个参数都是当前实体对象。

在这种情况下，鉴别器应用于类级别。所以我们将第二个参数转换为`CustomerReview`类型，并根据对象的`language`字段返回适当的分析器名称。如果语言未知或存在其他问题，则该方法简单地返回`null`，告诉 Hibernate Search 回退到默认分析器。

# 提升搜索结果的相关性

我们已经知道，搜索结果的默认排序顺序是按相关性，即它们与查询匹配的程度。如果一个实体在两个字段上匹配，而另一个只有一个字段匹配，那么第一个实体是更相关的结果。

Hibernate Search 允许我们通过在索引时调整实体或字段的相对重要性来调整相关性**提升**。这些调整可以是静态和固定的，也可以是动态的，由运行时数据状态驱动。

## 索引时间的静态提升

固定的提升，无论实际数据如何，都像注解一个类或字段一样简单，只需要使用`@Boost`。这个注解接受一个浮点数参数作为其相对权重，默认权重为 1.0\。所以，例如，`@Boost(2.0f)`会将一个类或字段的权重相对于未注解的类和字段加倍。

我们的 VAPORware Marketplace 应用程序在几个字段和关联上进行搜索，比如支持设备的名称，以及客户评论中的评论。然而，文本应该比来自外部各方的文本更重要，这难道不是合情合理的吗？（每个应用的名称和完整描述）

为了进行此调整，`chapter4`版本首先注释了`App`类本身：

```java
...
@Boost(2.0f)
public class App implements Serializable {
...
```

这实际上使得`App`的权重是`Device`或`CustomerReview`的两倍。接下来，我们对名称和完整描述字段应用字段级提升：

```java
...
@Boost(1.5f)
private String name;
...
@Boost(1.2f)
private String description;
...
```

我们在这里声明`name`的权重略高于`description`，并且它们相对于普通字段都带有更多的权重。

### 注意

请注意，类级别和字段级别的提升是级联和结合的！当给定字段应用多个提升因子时，它们会被乘以形成总因子。

在这里，因为已经对`App`类本身应用了 2.0 的权重，`name`的总有效权重为 3.0，`description`为 2.4。

## 索引时间的动态提升

让我们假设我们希望在评论者给出五星评价时，给`CustomerReview`对象更多的权重。为此，我们在类上应用一个`@DynamicBoost`注解：

```java
...
@DynamicBoost(impl=FiveStarBoostStrategy.class)
public class CustomerReview {
...
```

这个注解必须传递一个实现`BoostStrategy`接口的类，以及它的`defineBoost`方法：

```java
public class FiveStarBoostStrategy implements BoostStrategy {

 public float defineBoost(Object value) {
      if(value == null || !(value instanceofCustomerReview)) {
         return 1;
      }
      CustomerReviewcustomerReview = (CustomerReview) value;
      if(customerReview.getStars() == 5) {
         return 1.5f;
      } else {
         return 1;
      }
   }

}
```

当`@DynamicBoost`注解应用于一个类时，传递给`defineBoost`的参数自动是该类的一个实例（在这个例子中是一个`CustomerReview`对象）。如果注解是应用于一个特定的字段，那么自动传递的参数将是那个字段的值。

`defineBoost`返回的`float`值变成了被注解的类或字段的权重。在这个例子中，当`CustomerReview`对象代表一个五星评论时，我们将它的权重增加到 1.5。否则，我们保持默认的 1.0。

# 条件索引

字段索引有专门的处理方式，比如使用类桥接或程序化映射 API。总的来说，当一个属性被注解为`@Field`时，它就会被索引。因此，避免索引字段的一个明显方法就是简单地不应用这个注解。

然而，如果我们希望一个实体类通常可被搜索，但我们需要根据它们数据在运行时的状态排除这个类的某些实例怎么办？

`@Indexed`注解有一个实验性的第二个元素`interceptor`，它给了我们条件索引的能力。当这个元素被设置时，正常的索引过程将被自定义代码拦截，这可以根据实体的当前状态阻止实体被索引。

让我们给我们的 VAPORware Marketplace 添加使应用失效的能力。失效的应用仍然存在于数据库中，但不应该向客户展示或进行索引。首先，我们将向`App`实体类添加一个新属性：

```java
...
@Column
private boolean active;
...
public App(String name, String image, String description) {
   this.name = name;
   this.image = image;
   this.description = description;
 this.active = true;
}
...
public booleanisActive() {
   return active;
}
public void setActive(boolean active) {
   this.active = active;
}
...
```

这个新的`active`变量有标准的 getter 和 setter 方法，并且在我们的正常构造函数中被默认为`true`。我们希望在`active`变量为`false`时，个别应用被排除在 Lucene 索引之外，所以我们给`@Indexed`注解添加了一个`interceptor`元素：

```java
...
import com.packtpub.hibernatesearch.util.IndexWhenActiveInterceptor;
...
@Entity
@Indexed(interceptor=IndexWhenActiveInterceptor.class)
public class App {
...
```

这个元素必须绑定到一个实现`EntityIndexingInterceptor`接口的类上。由于我们刚刚指定了一个名为`IndexWhenActiveInterceptor`的类，所以我们现在需要创建这个类。

```java
package com.packtpub.hibernatesearch.util;

import org.hibernate.search.indexes.interceptor.EntityIndexingInterceptor;
import org.hibernate.search.indexes.interceptor.IndexingOverride;
import com.packtpub.hibernatesearch.domain.App;

public class IndexWhenActiveInterceptor
 implementsEntityIndexingInterceptor<App> {

   /** Only index newly-created App's when they are active */
 public IndexingOverrideonAdd(App entity) {
      if(entity.isActive()) {
         return IndexingOverride.APPLY_DEFAULT;
      }
      return IndexingOverride.SKIP;
   }
 public IndexingOverrideonDelete(App entity) {
      return IndexingOverride.APPLY_DEFAULT;
   }

   /** Index active App's, and remove inactive ones */
 public IndexingOverrideonUpdate(App entity) {
      if(entity.isActive()) {
         return IndexingOverride.UPDATE;
            } else {
         return IndexingOverride.REMOVE;
      }
   }

   public IndexingOverrideonCollectionUpdate(App entity) {
      retur nonUpdate(entity);
   }

}
```

`EntityIndexingInterceptor`接口声明了**四个方法**，Hibernate Search 会在实体对象的生命周期的不同阶段调用它们：

+   `onAdd()`: 当实体实例第一次被创建时调用。

+   `onDelete()`: 当实体实例从数据库中被移除时调用。

+   `onUpdate()`: 当一个现有实例被更新时调用。

+   `onCollectionUpdate()`: 当一个实体作为其他实体的批量更新的一部分被修改时使用这个版本。通常，这个方法的实现简单地调用`onUpdate()`。

这些方法中的每一个都应该返回`IndexingOverride`枚举的四种可能值之一。可能的**返回值**告诉 Hibernate Search 应该做什么：

+   `IndexingOverride.SKIP`：这告诉 Hibernate Search 在当前时间不要修改此实体实例的 Lucene 索引。

+   `IndexingOverride.REMOVE`：如果实体已经在索引中，Hibernate Search 将删除该实体；如果实体没有被索引，则什么也不做。

+   `IndexingOverride.UPDATE`：实体将在索引中更新，或者如果它还没有被索引，将被添加。

+   `IndexingOverride.APPLY_DEFAULT`：这等同于自定义拦截器根本没有被使用。Hibernate Search 将索引实体，如果这是一个`onAdd()`操作；如果这是一个`onDelete()`，则将其从索引中移除；或者如果这是`onUpdate()`或`onCollectionUpdate()`，则更新索引。

尽管这四种方法在逻辑上暗示了某些返回值，但实际上如果你处理的是异常情况，可以任意组合它们。

在我们的示例应用程序中，我们的拦截器在`onAdd()`和`onDelete()`中检查实体。当创建一个新的`App`时，如果其`active`变量为 false，则跳过索引。当`App`被更新时，如果它变得不活跃，它将被从索引中移除。

# 总结

在本章中，我们全面了解了为搜索而映射持久对象所提供的所有功能。现在我们可以调整 Hibernate Search 内置类型桥接的设置，并且可以创建高度先进的自定义桥接。

现在我们对 Lucene 分析有了更深入的了解。我们使用了一些最实用的自定义分析器组件，并且知道如何独立获取数十个其他 Solr 组件的信息。

我们现在可以通过提升来调整类和字段的相对权重，以在按相关性排序时提高我们的搜索结果质量。最后但同样重要的是，我们学会了如何使用条件索引动态地阻止某些数据根据其状态变得可搜索。

在下一章中，我们将转向更高级的查询概念。我们将学习如何过滤和分类搜索结果，并从 Lucene 中提取数据，而不需要数据库调用。
