# *第七章*: 类型、转换器和绑定

数据类型、转换器和绑定代表了通过基于 Java 的 **领域特定语言** (**DSL**) **应用程序编程接口** (**API**) 与数据库交互的主要方面。迟早，标准的 **结构化查询语言** (**SQL**)/**Java 数据库连接** (**JDBC**) 数据类型将不足以满足需求，或者 Java 类型和 JDBC 类型之间的默认映射将在你的特定场景中引起一些不足。在那个时刻，你将感兴趣于创建新的数据类型，使用自定义数据类型，以及你的 DSL API 的类型转换和类型绑定能力。幸运的是，**jOOQ 面向对象的查询** (**jOOQ**) DSL 提供了多功能且易于使用的 API，专注于以下议程，这也是本章的主题：

+   默认数据类型转换

+   自定义数据类型和类型转换

+   自定义数据类型和类型绑定

+   操作枚举

+   数据类型重写

+   处理可嵌入类型

让我们开始吧！

# 技术要求

本章的代码可以在 GitHub 上找到：[`github.com/PacktPublishing/jOOQ-Masterclass/tree/master/Chapter07`](https://github.com/PacktPublishing/jOOQ-Masterclass/tree/master/Chapter07)。

# 默认数据类型转换

jOOQ 允许我们以平滑方式使用的一个方面是其 *默认数据类型转换*。大多数时候，jOOQ 会隐藏 JDBC 和 Java 类型之间转换的仪式。例如，你是否好奇以下显式转换是如何工作的？看看下面：

```java
Record1<Integer> fiscalYear = ctx.select(field("fiscal_year", 
```

```java
  Integer.class)).from(table("sale")).fetchAny();
```

```java
// Offtake is a POJO
```

```java
Offtake offtake = ctx.select(field("fiscal_year"), 
```

```java
  field("sale"), field("employee_number")).from(table("sale"))            
```

```java
  .fetchAnyInto(Offtake.class);
```

这两种转换都通过 *默认数据类型转换* 或 *自动转换* 来解决。在幕后，jOOQ 依赖于它自己的 API，能够为 `Object` 类型、数组以及集合执行软类型安全的转换。

你可以在 *ConvertUtil* 应用程序中查看这个示例。

# 自定义数据类型和类型转换

在 jOOQ 中，所有特定方言数据类型的通用接口命名为 `org.jooq.DataType<T>`，其中 `T` 代表与 SQL 数据类型关联的 Java 类型。每个 `T` Java 数据类型与由 `java.sql.Types` 表示的 SQL 数据类型的关联（称为通用 SQL 类型，即标准 JDBC 类型）都存在于 jOOQ 的 `org.jooq.impl.SQLDataType` API 中。jOOQ 代码生成器自动将 Java 类型映射到这个 `SQLDataType` API，对于大多数方言，它与数据库的数据类型几乎 1:1 匹配。当然，我们不包括一些供应商特定的数据类型，例如空间数据类型、PostgreSQL 的 `INET`/`HSTORE`，以及其他非标准 JDBC 类型（JDBC 未明确支持的数据类型）。

大概来说，任何与 jOOQ API 没有关联标准 JDBC 类型的非数据类型都被视为和当作*自定义数据类型*。然而，正如卢卡斯·埃德尔提到的：“*我认为有些数据类型应该是标准 JDBC 类型，但不是。它们也列在* `SQLDataType`*中，包括：`JSON`、`JSONB`、`UUID`、`BigInteger`（!）、无符号数字、区间。这些不需要自定义数据类型。”

当你的自定义数据类型需要映射到标准 JDBC 类型——即`org.jooq.impl.SQLDataType`类型时，你需要提供并明确指定一个`org.jooq.Converter`实现。这个转换器执行涉及类型之间的转换的艰苦工作。

重要提示

当我们想要将一个类型映射到非标准 JDBC 类型（不在`org.jooq.impl.SQLDataType`中的类型）时，我们需要关注`org.jooq.Binding` API，这将在后面进行介绍。因此，如果这是你的情况，不要试图将你的转换逻辑强行应用到`Converter`上。只需使用`Binding`（我们将在本章后面看到这一点）。

请注意，尝试在不通过转换器的情况下插入自定义数据类型的价值/数据可能会导致数据库中插入`null`值（正如卢卡斯·埃德尔分享的那样：“*这种 null 行为是一个旧的设计缺陷。很久以前，我没有遵循尽早失败策略抛出异常*”），而尝试在没有转换器的情况下获取自定义数据类型的数据可能会导致`org.jooq.exception.DataTypeException`，*没有找到 Foo 和 Buzz 类型的转换器*。

## 编写 org.jooq.Converter 接口

`org.jooq.Converter`是一个接口，它表示两种类型之间的转换，这两种类型被泛型表示为`<T>`和`<U>`。通过`<T>`，我们表示数据库类型，通过`<U>`，我们表示`<T>`到`<U>`的转换是通过一个名为`U from(T)`的方法完成的，而从`<U>`到`<T>`的转换是通过一个名为`T to(U)`的方法完成的。

如果你发现很难记住哪个方向是"`from()`"`，哪个方向是"`to()`"`，那么可以这样想：前者可以读作"*从数据库到客户端*"，后者可以读作"*从客户端到数据库*"。此外，请注意不要混淆`T`和`U`，因为这样你可能会花费数小时盯着生成的代码中的编译错误。

换句话说，通过`U from(T)`，我们将数据库类型转换为 UDT（例如，这在`SELECT`语句中很有用），通过`T to(U)`，我们将 UDT 转换为数据库类型（例如，这在`INSERT`、`UPDATE`和`DELETE`语句中很有用）。此外，`T to(U)`方向用于任何使用绑定变量的地方，所以也在`SELECT`时编写谓词——例如，`T.CONVERTED.eq(u)`。`org.jooq.Converter`的存根如下：

```java
public interface Converter<T, U> {
```

```java
   U from(T databaseObject);  // convert to user-defined type
```

```java
   T to(U userDefinedObject); // convert to database type
```

```java
   // Class instances for each type
```

```java
   Class<T> fromType();
```

```java
   Class<U> toType();
```

```java
}
```

jOOQ 提供了这个接口的抽象实现（`AbstractConverter`）以及一些抽象的具体系列（转换器），您可以在以下链接中探索：[`www.jooq.org/javadoc/latest/org.jooq/org/jooq/impl/AbstractConverter.html`](https://www.jooq.org/javadoc/latest/org.jooq/org/jooq/impl/AbstractConverter.html)。但正如您接下来会看到的，我们可以编写自己的转换器。

例如，如果您想在应用程序中使用 Java 8 的 `java.time.YearMonth` 类型，但在数据库中存储为 SQL `INTEGER` 类型，您可以编写如下转换器：

```java
public class YearMonthConverter 
```

```java
             implements Converter<Integer, YearMonth> {
```

```java
  @Override
```

```java
  public YearMonth from(Integer t) {
```

```java
    if (t != null) {
```

```java
      return YearMonth.of(1970, 1)
```

```java
               .with(ChronoField.PROLEPTIC_MONTH, t);
```

```java
    }
```

```java
    return null;
```

```java
  }
```

```java
  @Override
```

```java
  public Integer to(YearMonth u) {
```

```java
    if (u != null) {
```

```java
      return (int) u.getLong(ChronoField.PROLEPTIC_MONTH);
```

```java
    }
```

```java
    return null;
```

```java
  }
```

```java
  @Override
```

```java
  public Class<Integer> fromType() {
```

```java
    return Integer.class;
```

```java
  }
```

```java
  @Override
```

```java
  public Class<YearMonth> toType() {
```

```java
    return YearMonth.class;
```

```java
  }
```

```java
}
```

通过 `new YearMonthConverter()` 或定义一个方便的 `static` 类型，如下所示来使用这个转换器：

```java
public static final Converter<Integer, YearMonth> 
```

```java
  INTEGER_YEARMONTH_CONVERTER = new YearMonthConverter();
```

此外，使用此转换器处理数组可以通过以下 `static` 类型完成，如下所示：

```java
public static final Converter<Integer[], YearMonth[]>    
```

```java
  INTEGER_YEARMONTH_ARR_CONVERTER
```

```java
    = INTEGER_YEARMONTH_CONVERTER.forArrays();
```

一旦我们有了转换器，我们就可以定义一个新的数据类型。更确切地说，我们通过调用 `asConvertedDataType(Converter)` 或 `asConvertedDataType(Binding)` 程序化地定义自己的 `DataType` 类型。例如，在这里，我们定义了一个 `YEARMONTH` 数据类型，它可以像 `SQLDataType` 中定义的任何其他数据类型一样使用：

```java
public static final DataType<YearMonth> YEARMONTH
```

```java
  = INTEGER.asConvertedDataType(INTEGER_YEARMONTH_CONVERTER);
```

这里，`INTEGER` 是 `org.jooq.impl.SQLDataType.INTEGER` 数据类型。

在 `CUSTOMER` 表中，我们有一个名为 `FIRST_BUY_DATE` 的字段，其类型为 `INT`。当客户进行首次购买时，我们将日期（年月）存储为整数。例如，日期 *2020-10* 被存储为 *24249*（我们手动应用了 `Integer to(YearMonth u)` 方法）。如果没有转换器，我们必须显式地插入 *24249*；否则，代码将无法编译（例如，类型安全的 `INSERT` 语句将无法编译）或者我们可能会得到一个无效的插入（例如，非类型安全的 `INSERT` 语句可能会存储 `null`）。依赖我们的转换器，我们可以编写以下类型安全的 `INSERT` 语句：

```java
ctx.insertInto(CUSTOMER, CUSTOMER.CUSTOMER_NAME, ... ,  
```

```java
               CUSTOMER.FIRST_BUY_DATE)
```

```java
   .values("Atelier One", ..., 
```

```java
       INTEGER_YEARMONTH_CONVERTER.to(YearMonth.of(2020, 10)))
```

```java
   .execute();
```

接下来，在不使用转换器的情况下获取 *Atelier One* 的所有 `FIRST_BUY_DATE` 值将导致得到一个整数数组或列表。要获取 `YearMonth` 类型的数组/列表，我们可以使用转换器，如下所示：

```java
List<YearMonth> resultListYM 
```

```java
  = ctx.select(CUSTOMER.FIRST_BUY_DATE).from(CUSTOMER)
```

```java
       .where(CUSTOMER.CUSTOMER_NAME.eq("Atelier One"))
```

```java
       .fetch(CUSTOMER.FIRST_BUY_DATE, 
```

```java
              INTEGER_YEARMONTH_CONVERTER);
```

在捆绑的代码（*YearMonthConverter*，适用于 MySQL 和 PostgreSQL）中，您可以看到更多示例，包括使用 `YEARMONTH` 数据类型进行强制转换和类型转换的操作。

当转换器在不同位置/类中偶尔使用时，编写具有其自己类的转换器是有用的。如果您知道转换器只在一个类中使用，那么您可以通过 `Converter.of()/ofNullable()` 在该类中本地定义它，如下所示（它们之间的区别在于 `Converter.ofNullable()` 对于 `null` 输入始终返回 `null`）：

```java
Converter<Integer, YearMonth> converter = 
```

```java
  Converter.ofNullable(Integer.class, YearMonth.class,
```

```java
    (Integer t) -> {
```

```java
      return YearMonth.of(1970, 1)
```

```java
             .with(ChronoField.PROLEPTIC_MONTH, t);
```

```java
    },
```

```java
    (YearMonth u) -> {
```

```java
      return (int) u.getLong(ChronoField.PROLEPTIC_MONTH);
```

```java
    }
```

```java
);
```

此外，从 jOOQ 3.15+ 开始，我们可以使用所谓的 *ad hoc 转换器*。这种类型的转换器对于将转换器附加到特定列以供单个查询或几个局部查询使用非常有用。例如，有一个转换器（`INTEGER_YEARMONTH_CONVERTER`），我们可以用它来处理单个列，如下所示：

```java
ctx.insertInto(CUSTOMER, CUSTOMER.CUSTOMER_NAME, ...,  
```

```java
 CUSTOMER.FIRST_BUY_DATE.convert(INTEGER_YEARMONTH_CONVERTER))
```

```java
   .values("Atelier One", ..., YearMonth.of(2020, 10))
```

```java
   .execute();
```

为了方便起见，jOOQ 除了提供特定的`convert()`函数（允许你将`Field<T>`类型转换为`Field<U>`类型，反之亦然）之外，还提供了`convertTo()`（允许你将`Field<U>`类型转换为`Field<T>`类型）和`convertFrom()`（允许你将`Field<T>`类型转换为`Field<U>`类型）的特定风味。由于我们的`INSERT`语句无法利用转换器的两个方向，我们可以回退到`convertTo()`，如下所示：

```java
ctx.insertInto(CUSTOMER, CUSTOMER.CUSTOMER_NAME, ...,
```

```java
 CUSTOMER.FIRST_BUY_DATE.convertTo(YearMonth.class, 
```

```java
            u -> INTEGER_YEARMONTH_CONVERTER.to(u)))
```

```java
   .values("Atelier One", ..., YearMonth.of(2020, 10))
```

```java
   .execute();
```

或者，在`SELECT`语句的情况下，你可能希望使用`converterFrom()`，如下所示：

```java
List<YearMonth> result = ctx.select(
```

```java
  CUSTOMER.FIRST_BUY_DATE.convertFrom(
```

```java
       t -> INTEGER_YEARMONTH_CONVERTER.from(t)))
```

```java
  .from(CUSTOMER)
```

```java
  .where(CUSTOMER.CUSTOMER_NAME.eq("Atelier One"))
```

```java
  .fetchInto(YearMonth.class);
```

当然，你甚至不需要在单独的类中定义转换器的负载。你可以简单地内联它，就像我们在这里所做的那样：

```java
ctx.insertInto(CUSTOMER, ...,
```

```java
    CUSTOMER.FIRST_BUY_DATE.convertTo(YearMonth.class, 
```

```java
      u -> (int) u.getLong(ChronoField.PROLEPTIC_MONTH)))
```

```java
   .values(..., YearMonth.of(2020, 10)) ...;
```

```java
List<YearMonth> result = ctx.select(
```

```java
    CUSTOMER.FIRST_BUY_DATE.convertFrom(
```

```java
      t -> YearMonth.of(1970, 1)
```

```java
       .with(ChronoField.PROLEPTIC_MONTH, t)))
```

```java
   .from(CUSTOMER) ...;
```

你可以查看*YearMonthAdHocConverter*中的示例，了解 MySQL 和 PostgreSQL 的特定转换器。

进一步来说，可以通过嵌套`to()`/`from()`方法的调用来嵌套转换器，可以通过`<X> Converter<T,X> andThen(Converter<? super U, X> converter)`方法进行链式调用。嵌套和链式调用都在捆绑代码（*YearMonthConverter*）中通过使用第二个转换器进行了示例，该转换器在`YearMonth`和`Date`之间进行转换，命名为`YEARMONTH_DATE_CONVERTER`。

此外，如果你想从`<T, U>`逆转换到`<U, T>`，那么就依靠`Converter.inverse()`方法。这在嵌套/链式调用转换器时可能很有用，可能需要你逆转换`T`与`U`以获得数据类型之间的适当匹配。这也在捆绑代码中得到了示例。

新的数据类型可以根据`converter`进行定义，如下所示：

```java
DataType<YearMonth> YEARMONTH 
```

```java
   = INTEGER.asConvertedDataType(converter);
```

也可以不使用显式的`Converter`来定义新的数据类型。只需使用`public default <U> DataType<U> asConvertedDataType(Class<U> toType, Function<? super T,? extends U> from, Function<? super U,? extends T> to)`这种风味，就像捆绑代码中那样，jOOQ 将在幕后使用`Converter.of(Class, Class, Function, Function)`。

另一方面，如果一个转换器被大量使用，那么最好允许 jOOQ 在没有显式调用的情况下自动应用它，就像之前的示例中那样。为了实现这一点，我们需要对 jOOQ 代码生成器进行适当的配置。

## 为转换器强制类型

通过使用所谓的*强制类型* (`<forcedTypes/>`)，我们可以指示 jOOQ 代码生成器覆盖列的数据类型。实现这一目标的一种方法是通过`org.jooq.Converter`将列数据类型映射到用户定义的数据类型。

此配置步骤依赖于使用 `<forcedTypes/>` 标签，它是 `<database/>` 标签的子标签。在 `<forcedTypes/>` 标签下，我们可以有一个或多个 `<forcedType/>` 标签，并且每个这样的标签都封装了覆盖列数据类型的特定情况。每种情况都通过几个标签定义。首先，我们有 `<userType/>` 和 `<converter/>` 标签，用于将 UDT 和适当的 `Converter` 链接起来。其次，我们有几个标签用于通过名称和/或类型识别某个特定列（或多个列）。虽然你可以在 jOOQ 手册中找到所有这些标签的描述（[`www.jooq.org/doc/latest/manual/code-generation/codegen-advanced/codegen-config-database/codegen-database-forced-types/`](https://www.jooq.org/doc/latest/manual/code-generation/codegen-advanced/codegen-config-database/codegen-database-forced-types/))，但在这里我们提到两个最常用的标签：`<includeExpression/>` 和 `<includeTypes/>`。`<includeExpression/>` 包含一个 Java `<includeTypes/>` 包含一个 Java 正则表达式，匹配应该强制具有此类型的数据类型（`<userType/>` 类型）。在存在多个正则表达式的情况下，使用管道操作符（`|`）分隔它们，并且如果 `<includeExpression/>` 和 `<includeTypes/>` 在同一个 `<forcedType/>` 标签中存在，那么请记住它们必须匹配。

例如，`YearMonthConverter` 的 `<forcedType/>` 类型看起来像这样：

```java
<forcedTypes>
```

```java
  <forcedType>
```

```java
    <!-- The Java type of the custom data type. 
```

```java
         This corresponds to the Converter's <U> type. -->
```

```java
    <userType>java.time.YearMonth</userType>
```

```java
    <!-- Associate that custom type with our converter. -->
```

```java
    <converter>
```

```java
      com.classicmodels.converter.YearMonthConverter
```

```java
    </converter>
```

```java
    <!-- Match the fully-qualified column. -->
```

```java
    <includeExpression>
```

```java
      classicmodels\.customer\.first_buy_date
```

```java
    </includeExpression>
```

```java
    <!-- Match the data type to be forced. -->
```

```java
    <includeTypes>INT</includeTypes>
```

```java
  </forcedType>
```

```java
</forcedTypes>
```

注意我们是如何通过包含模式、表和列名称的表达式来识别 `first_buy_date` 列的。在其他情况下，你可能希望使用更宽松的表达式；因此，这里有一些流行的例子：

```java
<!-- All 'first_buy_date' fields in any 'customer' table, 
```

```java
     no matter the schema -->
```

```java
.*\.customer\.first_buy_date
```

```java
<!-- All 'first_buy_date' fields, 
```

```java
     no matter the schema and the table -->
```

```java
.*\.first_buy_date
```

```java
<!-- All fields containing 'first_buy_' -->
```

```java
.*\.first_buy_.*
```

```java
<!-- Case-insensitive expressions -->
```

```java
(?i:.*\.customer\.first_buy_date)
```

```java
(?i:classicmodels\.customer\.first_buy_date)
```

重要提示

注意，jOOQ 代码生成器中的所有正则表达式都匹配以下内容之一：

1) **完全限定对象名称**（**FQONs**）

2) 部分限定对象名称

3) 未限定对象名称

因此，除了 `.*\.customer\.first_buy_date`，你也可以直接写 `customer\.first_buy_date`。

此外，请记住，默认情况下，正则表达式是区分大小写的。当你使用多个方言时，这一点很重要（例如，Oracle **标识符**（**IDs**）是 *UPPER_CASE*，在 PostgreSQL 中，它们是 *lower_case*，而在 SQL Server 中，它们是 *PascalCase*）。

此外，匹配任何类型是通过 `<includeTypes>.*</includeTypes>` 完成的，而匹配特定类型如 `NVARCHAR(4000)` 是通过 `NVARCHAR\(4000\)` 完成的，而类型如 `NUMBER(1, 0)` 是通过 `NUMBER\(1,\s*0\)` 完成的。有关此示例的更详细注释的完整版本可在捆绑的代码中找到。

这次，`FIRST_BUY_DATE` 字段没有映射到 `java.lang.Integer`。如果我们检查生成的与 `CUSTOMER` 表对应的表类（`jooq.generated.tables.Customer`），那么我们会看到以下声明：

```java
public final TableField<CustomerRecord, YearMonth> 
```

```java
 FIRST_BUY_DATE = createField(DSL.name("first_buy_date"),   
```

```java
  SQLDataType.INTEGER, this, "", new YearMonthConverter());
```

因此，`FIRST_BUY_DATE` 映射到 `YearMonth`，因此我们之前的 `INSERT` 和 `SELECT` 语句现在看起来是这样的：

```java
ctx.insertInto(CUSTOMER, CUSTOMER.CUSTOMER_NAME, ...,  
```

```java
               CUSTOMER.FIRST_BUY_DATE)
```

```java
   .values("Atelier One", ..., YearMonth.of(2020, 10))
```

```java
   .execute();
```

然后，`SELECT` 语句将看起来像这样：

```java
List<YearMonth> ymList = ctx.select(CUSTOMER.FIRST_BUY_DATE)
```

```java
   .from(CUSTOMER)
```

```java
   .where(CUSTOMER.CUSTOMER_NAME.eq("Atelier One"))
```

```java
   .fetch(CUSTOMER.FIRST_BUY_DATE);
```

jOOQ 会自动应用我们的转换器，因此不需要显式调用它。它甚至在我们执行 `ResultQuery<R1>` 到 `ResultQuery<R2>` 的强制操作时也能工作，如下所示：

```java
Result<Record2<String, YearMonth>> result = ctx.resultQuery(
```

```java
  "SELECT customer_name, first_buy_date FROM customer")
```

```java
  .coerce(CUSTOMER.CUSTOMER_NAME, CUSTOMER.FIRST_BUY_DATE)
```

```java
.fetch();
```

换句话说，jOOQ 会自动使用我们的转换器来绑定变量和从 `java.util.ResultSet` 中获取数据。在查询中，我们只需将 `FIRST_BUY_DATE` 视为 `YEARMONTH` 类型。代码命名为 *YearMonthConverterForcedTypes* 并适用于 MySQL。

### 通过 Converter.of() 或 Converter.ofNullable() 定义内联转换器

在前面的章节中，我们的转换器被编写为一个 Java 类，并在 jOOQ 代码生成器的配置中引用了这个类。但我们可以将自定义数据类型与一个 *内联转换器* 关联起来，这是一个直接写入配置的转换器。为此，我们使用 `<converter/>` 标签，如下所示：

```java
<forcedTypes>
```

```java
 <forcedType>    
```

```java
  <userType>java.time.YearMonth</userType>
```

```java
  ...
```

```java
  <converter>
```

```java
   <![CDATA[
```

```java
    org.jooq.Converter.ofNullable(
```

```java
       Integer.class, YearMonth.class, 
```

```java
     (Integer t) -> { return YearMonth.of(1970, 1).with(
```

```java
      java.time.temporal.ChronoField.PROLEPTIC_MONTH, t); },
```

```java
     (YearMonth u) -> { return (int) u.getLong(
```

```java
      java.time.temporal.ChronoField.PROLEPTIC_MONTH); }
```

```java
    )
```

```java
   ]]>
```

```java
  </converter>    
```

```java
  ...
```

```java
 </forcedType>
```

```java
</forcedTypes>
```

这个转换器的使用部分保持不变。完整的代码命名为 *InlineYearMonthConverter*，程序化版本命名为 *ProgrammaticInlineYearMonthConverter*。这两种应用都适用于 MySQL。

### 通过 lambda 表达式定义内联转换器

可以通过 `<lambdaExpression/>` 编写更简洁的内联转换器。这个标签使我们免去了显式使用 `Converter.of()`/`Converter.ofNullable()` 的麻烦，并允许我们简单地指定一个通过 `<from/>` 标签从数据库类型转换的 lambda 表达式，以及一个通过 `<to/>` 标签转换到数据库类型的 lambda 表达式。让我们在我们的转换器中举例说明，如下所示：

```java
<forcedTypes>
```

```java
 <forcedType>                                         
```

```java
  <userType>java.time.YearMonth</userType>
```

```java
  ...
```

```java
  <lambdaConverter>
```

```java
   <from>
```

```java
    <![CDATA[(Integer t) -> { return YearMonth.of(1970, 1)
```

```java
    .with(java.time.temporal.ChronoField.PROLEPTIC_MONTH, t);    
```

```java
    }]]>
```

```java
   </from>
```

```java
   <to>
```

```java
    <![CDATA[(YearMonth u) -> { return (int) 
```

```java
    u.getLong(java.time.temporal.ChronoField.PROLEPTIC_MONTH);    
```

```java
    }]]>
```

```java
   </to>                                            
```

```java
  </lambdaConverter>
```

```java
  ...
```

```java
  </forcedType>
```

```java
</forcedTypes>
```

再次强调，这个转换器的使用部分保持不变。完整的代码命名为 *LambdaYearMonthConverter*，程序化版本命名为 *ProgrammaticLambdaYearMonthConverter*。这两种应用都适用于 MySQL。

### 通过 SQL 匹配强制类型

在前面的章节中，我们通过 `<includeExpression/>` 和 `<includeTypes/>` 标签使用正则表达式匹配列名。每当我们需要更复杂的匹配列名的标准时，我们都可以依赖 `<sql/>` 标签。这个标签的正文是一个针对我们数据库字典视图执行的 SQL 查询。例如，匹配来自我们的 MySQL `classicmodels` 数据库的所有 `TIMESTAMP` 类型的列可以这样实现：

```java
<sql>
```

```java
 SELECT concat('classicmodels.', TABLE_NAME, '.', COLUMN_NAME)
```

```java
 FROM INFORMATION_SCHEMA.COLUMNS
```

```java
 WHERE TABLE_SCHEMA = 'classicmodels'
```

```java
  AND TABLE_NAME != 'flyway_schema_history'
```

```java
  AND DATA_TYPE = 'timestamp'
```

```java
</sql>
```

这应该返回几个列，其中包含来自 `PAYMENT` 表的两个列和来自 `BANK_TRANSACTION` 表的一个列：`PAYMENT.PAYMENT_DATE`、`PAYMENT.CACHING_DATE` 和 `BANK_TRANSACTION.CACHING_DATE`。对于这些列，jOOQ 将应用在捆绑代码中开发的 `Converter<LocalDateTime, JsonNode>`。但我们的查询返回的不仅仅是这些列，jOOQ 还会将此转换器应用于 `PAYMENT.MODIFIED` 和 `TOKEN.UPDATED_ON`，它们也是 `TIMESTAMP` 类型。现在，我们有两种避免这种情况的方法——我们可以相应地调整我们的查询谓词，或者我们可以快速添加 `<excludeExpression/>`，如下所示：

```java
<excludeExpression>
```

```java
    classicmodels\.payment\.modified
```

```java
  | classicmodels\.token\.updated_on
```

```java
</excludeExpression>
```

您可以在名为 *SqlMatchForcedTypes* 的名称下找到 MySQL 的示例。

我很确信你已经明白了这个概念，并且知道如何为你的数据库编写这样的查询。

## JSON 转换器

当 jOOQ 检测到数据库使用`JSON`类型时，它将数据库类型映射到`org.jooq.JSON`类。这是一个非常方便的类，它代表了一个从数据库中获取的整洁 JSON 包装类型。它的 API 包括`JSON.data()`方法，该方法返回`org.jooq.JSON`的字符串表示形式，以及`JSON.valueOf(String data)`方法，该方法从字符串表示形式返回`org.jooq.JSON`。通常，`org.jooq.JSON`就是你所需要的，但如果你想要通过专用 API（Jackson、Gson、**JSON Binary**（**JSONB**）等）来操作获取的 JSON，那么你需要一个转换器。

因此，为了练习更多示例，本书附带的自带代码包含一个 JSON 转换器（`JsonConverter`），如以下更详细地解释。

对于具有`JSON`数据类型的 MySQL 和 PostgreSQL，转换器在`org.jooq.JSON`和`com.fasterxml.jackson.databind.JsonNode`之间进行转换，因此它实现了`Converter<JSON, JsonNode>`接口。当然，你可以将 Jackson 的`JsonNode`替换为`com.google.gson.Gson`、`javax/jakarta.json.bind.Jsonb`等。MySQL 和 PostgreSQL 可用的代码命名为*JsonConverterForcedTypes*。此应用程序的程序化版本仅适用于 MySQL（但你很容易将其适应其他方言），并命名为*ProgrammaticJsonConverter*。

对于 Oracle 18c，它没有专门的 JSON 类型（然而，从 Oracle 21c 开始，这种类型是可用的；参见[`oracle-base.com/articles/21c/json-data-type-21c`](https://oracle-base.com/articles/21c/json-data-type-21c)），对于相对较小的 JSON 数据，通常使用`VARCHAR2(4000)`，而对于大型 JSON 数据，则使用`BLOB`。在这两种情况下，我们都可以添加一个`CHECK ISJSON()`约束来确保 JSON 数据的有效性。令人惊讶的是，jOOQ 能够检测到 JSON 数据的存在，并将此类列映射到`org.jooq.JSON`类型。我们的转换器在`org.jooq.JSON`和`com.fasterxml.jackson.databind.JsonNode`之间进行转换。请考虑名为*ConverterJSONToJsonNodeForcedTypes*的应用程序。

对于没有专门 JSON 类型的 SQL Server，通常使用带有`CHECK ISJSON()`约束的`NVARCHAR`。jOOQ 没有检测 JSON 数据使用情况的支持（如 Oracle 的情况），并将此类型映射到`String`。在这种情况下，我们有一个名为*JsonConverterVarcharToJSONForcedTypes*的转换器，它在`NVARCHAR`和`org.jooq.JSON`之间进行转换，还有一个名为*JsonConverterVarcharToJsonNodeForcedTypes*的转换器，它在`NVARCHAR`和`JsonNode`之间进行转换。

请花时间练习这些示例，以便熟悉 jOOQ 转换器。接下来，让我们来处理 UDT 转换器。

## UDT 转换器

如您从 *第三章*，*jOOQ 核心概念* 中所知，Oracle 和 PostgreSQL 支持用户定义类型（UDTs），并且在我们的模式中有一个名为 `EVALUATION_CRITERIA` 的 UDT。这个 UDT 是 `MANAGER.MANAGER_EVALUATION` 字段的类型，在 Oracle 中看起来像这样：

```java
CREATE OR REPLACE TYPE "EVALUATION_CRITERIA" AS OBJECT (
```

```java
  "communication_ability" NUMBER(7), 
```

```java
  "ethics" NUMBER(7), 
```

```java
  "performance" NUMBER(7), 
```

```java
  "employee_input" NUMBER(7),
```

```java
  // the irrelevant part was skipped
```

```java
);
```

我们已经知道 jOOQ 代码生成器自动通过 `jooq.generated.udt.EvaluationCriteria` 和 `jooq...pojos.EvaluationCriteria` `jooq...udt.EvaluationCriteria.EvaluationCriteriaRecord` 记录映射 `evaluation_criteria` UDT 的字段。

但如果我们假设我们的应用程序需要将此类型作为 JSON 进行操作，那么我们需要一个在 `EvaluationCriteriaRecord` 和 JSON 类型（例如，Jackson `JsonNode`）之间进行转换的转换器。`JsonConverter` 模板如下所示：

```java
public class JsonConverter implements 
```

```java
        Converter<EvaluationCriteriaRecord, JsonNode> {
```

```java
   @Override
```

```java
   public JsonNode from(EvaluationCriteriaRecord t) { ... }
```

```java
   @Override
```

```java
   public EvaluationCriteriaRecord to(JsonNode u) { ... }
```

```java
   ...
```

```java
}
```

接下来，我们配置此转换器，如下所示：

```java
<forcedTypes>
```

```java
 <forcedType>
```

```java
  <userType>com.fasterxml.jackson.databind.JsonNode</userType>
```

```java
  <converter>com...converter.JsonConverter</converter>
```

```java
  <includeExpression>
```

```java
   CLASSICMODELS\.MANAGER\.MANAGER_EVALUATION
```

```java
  </includeExpression>
```

```java
  <includeTypes>EVALUATION_CRITERIA</includeTypes>
```

```java
 </forcedType>
```

```java
</forcedTypes>  
```

拥有这个集合后，我们可以表达一个 `INSERT` 语句，如下所示：

```java
JsonNode managerEvaluation = "{...}";
```

```java
ctx.insertInto(MANAGER, 
```

```java
          MANAGER.MANAGER_NAME, MANAGER.MANAGER_EVALUATION)
```

```java
   .values("Mark Joy", managerEvaluation)
```

```java
   .execute();
```

我们可以表达一个 `SELECT` 语句，如下所示：

```java
List<JsonNode> managerEvaluation = ctx.select(
```

```java
      MANAGER.MANAGER_EVALUATION)
```

```java
   .from(MANAGER)
```

```java
   .fetch(MANAGER.MANAGER_EVALUATION);
```

包含的代码命名为 *ConverterUDTToJsonNodeForcedTypes*，适用于 Oracle 和 PostgreSQL。

# 自定义数据类型和类型绑定

大概来说，当我们想要将一个类型映射到非标准的 JDBC 类型（一个不在 `org.jooq.impl.SQLDataType` 中的类型）时，我们需要关注 `org.jooq.Binding` API，如下面的代码片段所示：

```java
public interface Binding<T, U> extends Serializable { ... }
```

例如，将非标准的特定供应商的 PostgreSQL `HSTORE` 数据类型绑定到某些 Java 数据类型（例如，`HSTORE` 可以非常方便地映射到 Java 的 `Map<String, String>`）需要利用 `Binding` API，该 API 包含以下方法（请参阅注释）：

```java
// A converter that does the conversion between 
```

```java
// the database type T and the user type U
```

```java
Converter<T, U> converter();
```

```java
// A callback that generates the SQL string for bind values of // this binding type. Typically, just ?, but also ?::json, ...
```

```java
void sql(BindingSQLContext<U> ctx) throws SQLException;
```

```java
// Register a type for JDBC CallableStatement OUT parameters
```

```java
ResultSet void register(BindingRegisterContext<U> ctx) 
```

```java
          throws SQLException;
```

```java
// Convert U to a type and set in on a JDBC PreparedStatement
```

```java
void set(BindingSetStatementContext<U> ctx) 
```

```java
          throws SQLException;
```

```java
// Get a type from JDBC ResultSet and convert it to U
```

```java
void get(BindingGetResultSetContext<U> ctx) 
```

```java
          throws SQLException;
```

```java
// Get a type from JDBC CallableStatement and convert it to U
```

```java
void get(BindingGetStatementContext<U> ctx) 
```

```java
          throws SQLException;
```

```java
// Get a value from JDBC SQLInput (useful for Oracle OBJECT)
```

```java
void get(BindingGetSQLInputContext<U> ctx) 
```

```java
          throws SQLException;
```

```java
// Get a value from JDBC SQLOutput (useful for Oracle OBJECT)
```

```java
void set(BindingSetSQLOutputContext<U> ctx) 
```

```java
          throws SQLException;
```

例如，让我们考虑我们已经有一个在 `Map<String, String>` 和 `HSTORE` 之间名为 `HstoreConverter` 的 `org.jooq.Converter` 实现，并且我们继续添加一个名为 `HstoreBinding` 的 `org.jooq.Binding` 实现，如下所示：

```java
public class HstoreBinding implements 
```

```java
          Binding<Object, Map<String, String>> {
```

```java
   private final HstoreConverter converter 
```

```java
      = new HstoreConverter();
```

```java
   @Override
```

```java
   public final Converter<Object, Map<String, String>> 
```

```java
                                              converter() {
```

```java
      return converter;
```

```java
   }
```

```java
   ...
```

```java
}
```

另一方面，对于 MySQL 的特定供应商 `POINT` 类型，我们可能有一个名为 `PointConverter` 的转换器，并且需要一个如下所示的 `PointBinding` 类——`POINT` 类型很好地映射到 Java 的 `Point2D.Double` 类型：

```java
public class PointBinding implements Binding<Object,Point2D> {
```

```java
   private final PointConverter converter 
```

```java
      = new PointConverter();
```

```java
   @Override
```

```java
   public final Converter<Object, Point2D> converter() {
```

```java
      return converter;
```

```java
   }
```

```java
   ...
```

```java
}
```

接下来，我们专注于实现 PostgreSQL 的 `HSTORE` 和 MySQL 的 `POINT` 的 `Binding` SPI。这一过程的一个重要方面是将绑定上下文值转换为 `HSTORE` 类型。这是在 `sql()` 方法中完成的，如下所示：

```java
@Override    
```

```java
public void sql(BindingSQLContext<Map<String, String>> ctx) 
```

```java
                                       throws SQLException {
```

```java
   if (ctx.render().paramType() == ParamType.INLINED) {
```

```java
      ctx.render().visit(inline(
```

```java
         ctx.convert(converter()).value())).sql("::hstore");
```

```java
   } else {
```

```java
      ctx.render().sql("?::hstore");
```

```java
   }
```

```java
}
```

注意，对于 jOOQ 内联参数（详细信息请参阅 *第三章*，*jOOQ 核心概念*），我们不需要渲染占位符（`?`）；因此，我们只渲染 PostgreSQL 特定的语法，`::hstore`。根据数据库特定的语法，您必须渲染预期的 SQL。例如，对于 PostgreSQL 的 `INET` 数据类型，您将渲染 `?::inet`（或，`::inet`），而对于 MySQL 的 `POINT` 类型，您将渲染 `ST_PointFromText(?)` 如下所示（`Point2D` 是 `java.awt.geom.Point2D`）：

```java
@Override
```

```java
public void sql(BindingSQLContext<Point2D> ctx) 
```

```java
        throws SQLException {
```

```java
  if (ctx.render().paramType() == ParamType.INLINED) {
```

```java
      ctx.render().sql("ST_PointFromText(")
```

```java
         .visit(inline(ctx.convert(converter()).value()))
```

```java
         .sql(")");
```

```java
  } else {
```

```java
      ctx.render().sql("ST_PointFromText(?)");
```

```java
  }
```

```java
}
```

接下来，我们关注为 JDBC `CallableStatement` 的 OUT 参数注册一个兼容的/合适的类型。通常，`VARCHAR` 是一个合适的选择（例如，对于 `HSTORE`、`INET` 或 `JSON` 类型，`VARCHAR` 是一个好的选择）。下面的代码片段展示了代码：

```java
@Override
```

```java
public void register(BindingRegisterContext
```

```java
          <Map<String, String>> ctx) throws SQLException {
```

```java
   ctx.statement().registerOutParameter(
```

```java
      ctx.index(), Types.VARCHAR);
```

```java
}
```

但是，由于默认情况下 MySQL 以二进制数据形式返回 `POINT`（只要我们不使用任何 MySQL 函数，例如 `ST_AsText(g)` 或 `ST_AsWKT(g)` 将几何值从内部几何格式转换为 `java.sql.Blob`，如下面的代码片段所示）：

```java
@Override
```

```java
public void register(BindingRegisterContext<Point2D> ctx)  
```

```java
                                       throws SQLException {
```

```java
   ctx.statement().registerOutParameter(
```

```java
      ctx.index(), Types.BLOB);
```

```java
}
```

接下来，我们将 `Map<String, String>` 转换为 `String` 值，并将其设置在 JDBC `PreparedStatement` 上（对于 MySQL 的 `POINT` 类型，我们将 `Point2D` 转换为 `String`），如下所示：

```java
@Override
```

```java
public void set(BindingSetStatementContext
```

```java
            <Map<String, String>> ctx) throws SQLException {
```

```java
   ctx.statement().setString(ctx.index(), Objects.toString(
```

```java
     ctx.convert(converter()).value(), null));
```

```java
}  
```

此外，对于 PostgreSQL 的 `HSTORE`，我们从 JDBC `ResultSet` 中获取一个 `String` 值，并将其转换为 `Map<String, String>`，如下所示：

```java
@Override
```

```java
public void get(BindingGetResultSetContext
```

```java
           <Map<String, String>> ctx) throws SQLException {
```

```java
   ctx.convert(converter()).value(
```

```java
     ctx.resultSet().getString(ctx.index()));
```

```java
}
```

当我们处理 MySQL 的 `POINT` 类型时，我们会从 JDBC `ResultSet` 中获取一个 `Blob`（或一个 `InputStream`），并将其转换为 `Point2D`，如下所示：

```java
@Override
```

```java
public void get(BindingGetResultSetContext<Point2D> ctx) 
```

```java
                                      throws SQLException {
```

```java
   ctx.convert(converter()).value(ctx.resultSet()
```

```java
      .getBlob(ctx.index())); // or, getBinaryStream()
```

```java
}
```

接下来，我们对 JDBC `CallableStatement` 做同样的事情。对于 `HSTORE` 类型，我们有以下内容：

```java
@Override
```

```java
public void get(BindingGetStatementContext
```

```java
           <Map<String, String>> ctx) throws SQLException {
```

```java
   ctx.convert(converter()).value(
```

```java
      ctx.statement().getString(ctx.index()));
```

```java
}
```

对于 `POINT` 类型，我们有以下内容：

```java
@Override
```

```java
public void get(BindingGetStatementContext<Point2D> ctx) 
```

```java
                                       throws SQLException {
```

```java
   ctx.convert(converter()).value(
```

```java
      ctx.statement().getBlob(ctx.index()));
```

```java
}
```

最后，我们重写了 `get(BindingGetSQLInputContext<?> bgsqlc)` 和 `set(BindingSetSQLOutputContext<?> bsqlc)` 方法。由于我们不需要 `HSTORE`/`POINT`，我们只是抛出一个 `SQLFeatureNotSupportedException` 异常。为了简洁，我们省略了这段代码。

一旦 `Binding` 准备就绪，我们必须在 jOOQ 代码生成器中对其进行配置。这相当类似于 `Converter` 的配置，只是我们使用 `<binding/>` 标签而不是 `<converter/>` 标签，如下所示——在这里，我们配置 `HstoreBinding`（`PointBinding` 的配置可以在捆绑的代码中找到）：

```java
<forcedTypes>
```

```java
  <forcedType>
```

```java
    <userType>java.util.Map&lt;String, String&gt;</userType>   
```

```java
    <binding>com.classicmodels.binding.HstoreBinding</binding>
```

```java
    <includeExpression>
```

```java
      public\.product\.specs
```

```java
    </includeExpression>
```

```java
    <includeTypes>HSTORE</includeTypes>
```

```java
  </forcedType>
```

```java
</forcedTypes>
```

现在，我们可以测试 `HstoreBinding`。例如，`PRODUCT` 表有一个名为 `SPECS` 的字段，其类型为 `HSTORE`。下面的代码插入了一个带有一些规格的新产品：

```java
ctx.insertInto(PRODUCT, PRODUCT.PRODUCT_NAME, 
```

```java
               PRODUCT.PRODUCT_LINE, PRODUCT.SPECS)
```

```java
   .values("2002 Masserati Levante", "Classic Cars",
```

```java
      Map.of("Length (in)", "197", "Width (in)", "77.5", 
```

```java
             "Height (in)", "66.1", "Engine", "Twin Turbo 
```

```java
             Premium Unleaded V-6"))
```

```java
   .execute();  
```

下面是渲染后的 SQL 看起来的样子：

```java
INSERT INTO "public"."product" (
```

```java
            "product_name", "product_line", "specs")
```

```java
VALUES (?, ?, ?::hstore)
```

在解决 `?` 占位符之后，SQL 看起来是这样的：

```java
INSERT INTO "public"."product" (
```

```java
            "product_name", "product_line", "specs")
```

```java
VALUES ('2002 Masserati Levante', 'Classic Cars', 
```

```java
        '"Width (in)"=>"77.5", "Length (in)"=>"197", 
```

```java
         "Height (in)"=>"66.1", 
```

```java
         "Engine"=>"Twin Turbo Premium Unleaded V-6"'::hstore)
```

在 `INSERT`（`UPDATE`、`DELETE` 等等）时，`HstoreConverter` 将 Java `Map<String, String>` 转换为 `HSTORE` 类型。在 `SELECT` 时，相同的转换器将 `HSTORE` 转换为 `Map<String, String>`。因此，我们的 `SELECT` 语句可能看起来像这样：

```java
List<Map<String, String>> specs = ctx.select(PRODUCT.SPECS)
```

```java
   .from(PRODUCT)
```

```java
   .where(PRODUCT.PRODUCT_NAME.eq("2002 Masserati Levante"))
```

```java
   .fetch(PRODUCT.SPECS);
```

注意，我们没有显式地使用任何 `Binding` 或 `Converter`，也没有**触及**`HSTORE` 类型。对我们来说，在应用程序中，`SPECS` 的类型是 `Map<String, String>`。

注意，从 jOOQ 3.15 版本开始，我们可以访问 `jOOQ-postgres-extensions` 模块（[`github.com/jOOQ/jOOQ/issues/5507`](https://github.com/jOOQ/jOOQ/issues/5507)），它也支持 `HSTORE`。

可以使用 `Binding` 和 `Converter` 来编写不同的辅助方法。例如，以下方法可以用来将任何 `Param` 转换为其数据库数据类型：

```java
static <T> Object convertToDatabaseType(Param<T> param) {
```

```java
   return param.getBinding().converter().to(param.getValue());
```

```java
}
```

但是，没有 `Binding` 会发生什么？是不是一切都丢失了？

## 不使用 `Binding` 理解所发生的事情

当 jOOQ 检测到一个没有关联`Binding`的非标准 JDBC 类型时，它将相应的字段标记为`@deprecated Unknown data type`，并附带消息，*请定义一个明确的{@link org.jooq.Binding}来指定如何处理此类型。可以通过在代码生成器配置中使用{@literal <deprecationOnUnknownTypes/>}来关闭弃用。*

根据经验法则，依赖`Binding`是最佳做法，但作为权宜之计，我们也可以为`SELECT`语句和`public static <T> Field<T> field(String sql, Class<T> type, Object... bindings)`或更适合的`field()`变体使用显式映射，对于`INSERT`、`UPDATE`等。

然而，在`INSERT`语句（`UPDATE`语句等）中使用这样的非标准 JDBC 类型会导致 jOOQ 的`SQLDialectNotSupportedException`异常，*类型 Foo 在方言 Buzz 中不受支持，并且在 SELECT 语句中，会导致 jOOQ 的`DataTypeException`，没有找到类型 Foo 和 Buzz 的转换器。*

您可以检查名为*HstoreBinding*的应用程序中的此节中的`HSTORE`示例，以及名为*PointGeometryBinding*的应用程序中的`POINT`示例。

此外，捆绑的代码包含用于 PostgreSQL `INET`类型的`InetBinding`，用于 PostgreSQL `JSON`类型的`JsonBinding`，以及代表 PostgreSQL `INET`类型`Binding`程序配置的`ProgrammaticInetBinding`。接下来，让我们讨论枚举以及如何转换这些枚举。

# 操作枚举

jOOQ 通过名为`org.jooq.EnumType`的接口表示 SQL 枚举类型（例如，通过`CREATE TYPE`创建的 MySQL `enum`或 PostgreSQL `enum`数据类型）。每当 jOOQ Java 代码生成器检测到 SQL 枚举类型的用法时，它会自动生成一个实现`EnumType`的 Java 枚举。例如，`SALE`表的 MySQL 模式包含以下`enum`数据类型：

```java
'vat' ENUM ('NONE', 'MIN', 'MAX') DEFAULT NULL
```

对于`vat`，jOOQ 生成器渲染了`jooq.generated.enums.VatType`枚举，如下所示：

```java
public enum VatType implements EnumType {
```

```java
    NONE("NONE"), MIN("MIN"), MAX("MAX");
```

```java
    private final String literal;
```

```java
    private VatType(String literal) {
```

```java
        this.literal = literal;
```

```java
    }
```

```java
    @Override
```

```java
    public Catalog getCatalog() {
```

```java
        return null;
```

```java
    }
```

```java
    @Override
```

```java
    public Schema getSchema() {
```

```java
        return null;
```

```java
    }
```

```java
    @Override
```

```java
    public String getName() {
```

```java
        return "sale_vat";
```

```java
    }
```

```java
    @Override
```

```java
    public String getLiteral() {
```

```java
        return literal;
```

```java
    }
```

```java
    public static VatType lookupLiteral(String literal) {
```

```java
        return EnumType.lookupLiteral(VatType.class, literal);
```

```java
    }
```

```java
}
```

默认情况下，此类名称由表名和列名以*PascalCase*形式组成，这意味着前一个类的名称应该是`SaleVat`。但每当我们要修改默认名称时，我们可以依靠 jOOQ *生成策略*和正则表达式，就像我们在*第二章*，*自定义 jOOQ 参与级别*中所做的那样。例如，我们已经通过以下策略将前一个类名称自定义为`VatType`：

```java
<strategy>
```

```java
  <matchers>                                              
```

```java
    <enums>                                           
```

```java
      <enum>         
```

```java
        <expression>sale_vat</expression>                         
```

```java
        <enumClass>                                                    
```

```java
          <expression>VatType</expression>
```

```java
          <transform>AS_IS</transform>                          
```

```java
        </enumClass>                             
```

```java
      </enum>
```

```java
    </enums>
```

```java
  </matchers>
```

```java
</strategy>
```

拥有这些知识就足以开始编写基于 jOOQ 生成枚举的查询了——例如，向`SALE`表插入的`INSERT`语句和从中选择的`SELECT`语句，如下面的代码片段所示：

```java
import jooq.generated.enums.VatType;
```

```java
...
```

```java
ctx.insertInto(SALE, SALE.FISCAL_YEAR, ..., SALE.VAT)
```

```java
   .values(2005, ..., VatType.MAX)
```

```java
   .execute();
```

```java
List<VatType> vats = ctx.select(SALE.VAT).from(SALE)
```

```java
   .where(SALE.VAT.isNotNull())
```

```java
   .fetch(SALE.VAT);
```

当然，`Sale`生成的 POJO（或用户定义的 POJO）和`SaleRecord`利用了`VatType`，就像任何其他类型一样。

## 编写枚举转换器

当 jOOQ 生成的枚举不足以使用时，我们关注枚举转换器。以下是一些可能需要某种枚举转换的场景列表：

+   使用自己的 Java 枚举作为数据库枚举类型

+   使用自己的 Java 枚举作为数据库非枚举类型（或枚举类似类型）

+   使用 Java 非枚举类型作为数据库枚举

+   总是转换为 Java 枚举，偶尔转换为另一个 Java 枚举

为了简化枚举转换任务，jOOQ 提供了一个内置的默认转换器，名为`org.jooq.impl.EnumConverter`。此转换器可以将`VARCHAR`值转换为枚举字面量（反之亦然），或将`NUMBER`值转换为枚举序号（反之亦然）。您也可以显式实例化它，如下所示：

```java
enum Size { S, M, XL, XXL; }
```

```java
Converter<String, Size> converter 
```

```java
   = new EnumConverter<>(String.class, Size.class);
```

接下来，让我们解决之前列出的枚举场景。

### 使用自己的 Java 枚举作为数据库枚举类型

在我们的四个数据库中，只有 MySQL 和 PostgreSQL 有针对枚举的专用类型。MySQL 有`enum`类型，PostgreSQL 有`CREATE TYPE foo AS enum( ...)`语法。在这两种情况下，jOOQ 都会为我们生成枚举类，但假设我们更愿意使用自己的 Java 枚举。例如，让我们关注`SALE`表的 MySQL 模式，其中包含这两个枚举：

```java
`rate` ENUM ('SILVER', 'GOLD', 'PLATINUM') DEFAULT NULL
```

```java
`vat` ENUM ('NONE', 'MIN', 'MAX') DEFAULT NULL
```

在 PostgreSQL 中，相同的枚举声明如下：

```java
CREATE TYPE rate_type AS enum('SILVER', 'GOLD', 'PLATINUM');
```

```java
CREATE TYPE vat_type AS enum('NONE', 'MIN', 'MAX');
```

```java
…
```

```java
rate rate_type DEFAULT NULL,
```

```java
vat vat_type DEFAULT NULL,
```

假设对于`vat`，我们仍然依赖于 jOOQ 生成的 Java 枚举类（如前文所述，`VatType`），而对于`rate`，我们已经编写了以下 Java 枚举：

```java
public enum RateType { SILVER, GOLD, PLATINUM }
```

为了自动将`rate`列映射到`RateType`枚举，我们依赖于`<forcedType/>`和`<enumConverter/>`标志标签，如下所示：

```java
<forcedTypes>
```

```java
  <forcedType>
```

```java
    <userType>com.classicmodels.enums.RateType</userType>
```

```java
    <enumConverter>true</enumConverter>
```

```java
    <includeExpression>
```

```java
      classicmodels\.sale\.rate # MySQL
```

```java
      public\.sale\.rate        # PostgreSQL
```

```java
    </includeExpression>
```

```java
    <includeTypes>
```

```java
       ENUM      # MySQL
```

```java
       rate_type # PostgreSQL
```

```java
    </includeTypes>
```

```java
  </forcedType>
```

```java
</forcedTypes>
```

通过启用`<enumConverter/>`，我们指示 jOOQ 在`SALE.RATE`被使用时自动应用内置的`org.jooq.impl.EnumConverter`转换器。完成！从现在起，我们可以将`SALE.RATE`字段视为`RateType`类型，jOOQ 将处理映射字段的转换方面（如下列 MySQL 所示），如下所示：

```java
public final TableField<SaleRecord, RateType> RATE 
```

```java
  = createField(DSL.name("rate"), SQLDataType.VARCHAR(8),    
```

```java
    this, "", new EnumConverter<String, RateType>   
```

```java
                             (String.class, RateType.class));
```

命名为*SimpleBuiltInEnumConverter*的应用程序包含了 MySQL 和 PostgreSQL 的完整示例。

这是一个非常方便的方法，在 MySQL 和 PostgreSQL 中效果相同，但如果我们不使用这种自动转换，我们仍然可以使用我们的`RateType` Java 枚举手动或显式地使用。让我们看看如何！

首先，我们配置 jOOQ 代码生成器以排除`sale_rate`（MySQL）/`rate_type`（PostgreSQL）类型的枚举生成；否则，`SALE.RATE`字段将自动映射到生成的 Java 枚举。代码如下所示：

```java
<database>
```

```java
   <excludes>
```

```java
     sale_rate (MySQL) / rate_type (PostgreSQL)
```

```java
   </excludes>
```

```java
</database>
```

在这种情况下，jOOQ 将 MySQL 中的`SALE.RATE`映射为`String`，在 PostgreSQL 中映射为`Object`。在 PostgreSQL 中，该字段被注释为`@deprecated Unknown data type`，但我们通过`<deprecationOnUnknownTypes/>`配置关闭了这种弃用，如下所示：

```java
<generate>
```

```java
  <deprecationOnUnknownTypes>false</deprecationOnUnknownTypes>
```

```java
</generate>
```

接下来，在 MySQL 中，我们可以编写如下`INSERT`语句（`SALE.RATE`为`String`类型）：

```java
ctx.insertInto(SALE, SALE.FISCAL_YEAR, ..., SALE.RATE)
```

```java
   .values(2005, ..., RateType.PLATINUM.name())
```

```java
   .execute();
```

我们可以编写如下`SELECT`语句：

```java
List<RateType> rates = ctx.select(SALE.RATE)
```

```java
   .from(SALE)
```

```java
   .where(SALE.RATE.isNotNull())
```

```java
   .fetch(SALE.RATE, RateType.class);
```

对于 MySQL 来说，这相当顺畅，但对于 PostgreSQL 来说，则有点棘手。PostgreSQL 的语法要求我们在`INSERT`时渲染类似`?::"public"."rate_type"`的内容，如下面的代码片段所示：

```java
ctx.insertInto(SALE, SALE.FISCAL_YEAR, ..., SALE.RATE)
```

```java
   .values(2005, ..., field("?::\"public\".\"rate_type\"", 
```

```java
           RateType.PLATINUM.name()))
```

```java
   .execute();
```

在`SELECT`时，我们需要将`Object`显式转换为`String`，如下面的代码片段所示：

```java
List<RateType> rates = ctx.select(SALE.RATE)
```

```java
   .from(SALE)
```

```java
   .where(SALE.RATE.isNotNull())
```

```java
   .fetch(SALE.RATE.coerce(String.class), RateType.class);
```

应用程序名为*MyEnumBuiltInEnumConverter*，其中包含 MySQL 和 PostgreSQL 的完整示例。如果我们不抑制 jOOQ 枚举生成，那么另一种方法是通过扩展 jOOQ 内置的`org.jooq.impl.EnumConverter`转换器，在 jOOQ 生成的枚举和我们的枚举之间编写一个显式的转换器。当然，这个转换器必须在你的查询中显式调用。你可以在前面提到的应用程序中找到`vat`枚举的此类示例。

### 使用您自己的 Java 枚举类型来表示数据库的非枚举类型（或类似枚举的类型）

让我们考虑一个包含只接受某些值的列的遗留数据库，但该列被声明为`VARCHAR`（或`NUMBER`）——例如，`SALE`表有一个`TREND`字段，其类型为`VARCHAR`，只接受*UP*、*DOWN*和*CONSTANT*的值。在这种情况下，通过枚举强制使用此字段会更实用，如下所示：

```java
public enum TrendType { UP, DOWN, CONSTANT }
```

但现在，我们必须处理`TrendType`和`VARCHAR`之间的转换。如果我们添加以下`<forcedType/>`标签（这里针对 Oracle），jOOQ 可以自动完成此操作：

```java
<forcedType>    
```

```java
  <userType>com.classicmodels.enums.TrendType</userType>     
```

```java
  <enumConverter>true</enumConverter>                                              
```

```java
  <includeExpression>
```

```java
    CLASSICMODELS\.SALE\.TREND
```

```java
  </includeExpression>                                            
```

```java
  <includeTypes>VARCHAR2\(10\)</includeTypes>
```

```java
</forcedType>
```

在*SimpleBuiltInEnumConverter*应用程序中，你可以看到与其他四个数据库的示例并排的完整示例。

由于 SQL Server 和 Oracle 没有枚举类型，我们使用了替代方案。其中之一是，一个常见的替代方案依赖于`CHECK`约束来实现类似枚举的行为。这些类似枚举的类型可以像之前展示的那样利用`<enumConverter/>`。这里，它是 Oracle 中的`SALE.VAT`字段：

```java
vat VARCHAR2(10) DEFAULT NULL 
```

```java
  CHECK (vat IN('NONE', 'MIN', 'MAX'))
```

这里是`<forcedType/>`标签：

```java
<forcedType>                                                                                                                                        
```

```java
  <userType>com.classicmodels.enums.VatType</userType>                                                                                                                                                                                 
```

```java
  <enumConverter>true</enumConverter>                                            
```

```java
  <includeExpression>    
```

```java
    CLASSICMODELS\.SALE\.VAT
```

```java
  </includeExpression>                                            
```

```java
  <includeTypes>VARCHAR2\(10\)</includeTypes>
```

```java
</forcedType>
```

如果我们不希望依赖自动转换，那么我们可以使用一个显式的转换器，如下所示：

```java
public class SaleStrTrendConverter 
```

```java
           extends EnumConverter<String, TrendType> {   
```

```java
  public SaleStrTrendConverter() {
```

```java
     super(String.class, TrendType.class);
```

```java
  }        
```

```java
}
```

在*BuiltInEnumConverter*应用程序中，你可以找到与其他四个数据库的示例并排的完整示例。

### 使用 Java 非枚举类型表示数据库枚举

有时，我们需要一个非枚举类型来表示数据库枚举。例如，假设我们想用一些整数代替`VatType`枚举（*0*代表*NONE*，*5*代表*MIN*，*19*代表*MAX*），因为我们可能在不同的计算中需要这些整数。也许最好的办法是编写一个像这样的转换器：

```java
public class SaleVatIntConverter 
```

```java
   extends EnumConverter<VatType, Integer> { … }
```

但这不起作用，因为`EnumConverter`的签名实际上是`EnumConverter<T, U extends Enum<U>>`类型。显然，`Integer`不满足这个签名，因为它没有扩展`java.lang.Enum`，因此我们可以依赖一个常规的转换器（正如你在上一节中看到的），如下所示：

```java
public class SaleVatIntConverter 
```

```java
   implements Converter<VatType, Integer> { … }
```

*BuiltInEnumConverter*应用包含此示例，以及其他示例。当然，您也可以尝试通过`Converter.of()`/`ofNullable()`或 lambda 表达式将此转换器作为内联转换器编写。

### 总是转换为 Java 枚举，偶尔也转换为另一个 Java 枚举

总是转换为 Java 枚举和偶尔转换为另一个 Java 枚举可能不是一项流行的任务，但让我们用它作为前提来总结我们迄今为止关于枚举转换所学到的东西。

让我们考虑 MySQL 中众所周知的`SALE.RATE`枚举字段。首先，我们希望始终/自动将`SALE.RATE`转换为我们的`RateType` Java 枚举，如下所示：

```java
public enum RateType { SILVER, GOLD, PLATINUM }
```

为了此目的，我们编写以下`<forcedType/>`标签：

```java
<forcedType>
```

```java
  <userType>com.classicmodels.enums.RateType</userType>
```

```java
  <enumConverter>true</enumConverter>
```

```java
  <includeExpression>
```

```java
    classicmodels\.sale\.rate
```

```java
  </includeExpression>
```

```java
  <includeTypes>ENUM</includeTypes>
```

```java
</forcedType>
```

到目前为止，我们可以在查询中引用`SALE.RATE`为`RateType`枚举，但让我们假设我们还有以下`StarType`枚举：

```java
public enum StarType { THREE_STARS, FOUR_STARS, FIVE_STARS }
```

基本上，`StarType`是`RateType`的替代品（*THREE_STARS*对应于*SILVER*，*FOUR_STARS*对应于*GOLD*，*FIVE_STARS*对应于*PLATINUM*）。现在，我们可能偶尔想在查询中使用`StarType`而不是`RateType`，因此我们需要一个转换器，如下所示：

```java
public class SaleRateStarConverter extends 
```

```java
                    EnumConverter<RateType, StarType> {
```

```java
   public final static SaleRateStarConverter 
```

```java
      SALE_RATE_STAR_CONVERTER = new SaleRateStarConverter();
```

```java
   public SaleRateStarConverter() {
```

```java
      super(RateType.class, StarType.class);
```

```java
   }
```

```java
   @Override
```

```java
   public RateType to(StarType u) {
```

```java
      if (u != null) {
```

```java
         return switch (u) {
```

```java
                case THREE_STARS -> RateType.SILVER;
```

```java
                case FOUR_STARS -> RateType.GOLD;
```

```java
                case FIVE_STARS -> RateType.PLATINUM;
```

```java
         };
```

```java
      }
```

```java
      return null;
```

```java
   }
```

```java
}
```

由于`RateType`和`StarType`不包含相同的字面量，我们必须重写`to()`方法并定义预期的匹配。完成！

使用`RateType`的`INSERT`语句表达如下：

```java
// rely on <forcedType/> 
```

```java
ctx.insertInto(SALE, SALE.FISCAL_YEAR, ,..., SALE.RATE)
```

```java
   .values(2005, ..., RateType.PLATINUM)
```

```java
   .execute();
```

并且每当我们要使用`StarType`而不是`RateType`时，我们就依赖于静态的`SALE_RATE_STAR_CONVERTER`转换器，如下所示：

```java
// rely on SALE_RATE_STAR_CONVERTER
```

```java
ctx.insertInto(SALE, SALE.FISCAL_YEAR, ..., SALE.RATE)
```

```java
   .values(2005, ...,   
```

```java
           SALE_RATE_STAR_CONVERTER.to(StarType.FIVE_STARS))
```

```java
   .execute();
```

*BuiltInEnumConverter*应用包含此示例，以及其他示例。

通过`classicmodels\.sale\.rate`，我们指定了一个特定的列（`CLASSICMODELS.SALE.RATE`），但我们可能希望获取此枚举类型的所有列。在这种情况下，SQL 查询比正则表达式更合适。以下是一个 Oracle 的查询示例：

```java
SELECT 'CLASSICMODELS.' || tab.table_name || '.' 
```

```java
                        || cols.column_name
```

```java
FROM sys.all_tables tab
```

```java
JOIN sys.all_constraints con ON tab.owner = con.owner
```

```java
   AND tab.table_name = con.table_name
```

```java
JOIN sys.all_cons_columns cols ON cols.owner = con.owner
```

```java
   AND cols.constraint_name = con.constraint_name
```

```java
   AND cols.table_name = con.table_name
```

```java
WHERE constraint_type = 'C'
```

```java
   AND tab.owner in ('CLASSICMODELS')
```

```java
   AND search_condition_vc 
```

```java
      = q'[rate IN('SILVER', 'GOLD', 'PLATINUM')]'
```

您可以在 MySQL 和 Oracle 中找到此示例，作为*BuiltInEnumSqlConverter*。

在捆绑的代码中，有更多应用，例如*EnumConverter*，它提供了枚举的`org.jooq.Converter`类型的示例；*EnumConverterForceTypes*，它包含`<forcedType/>`和枚举示例；以及`I`*nsertEnumPlainSql*，当不使用 jOOQ 代码生成器时，它包含`INSERT`和枚举示例。

### 获取给定枚举数据类型的`DataType<T>`标签

获取给定枚举数据类型的`DataType<T>`标签可以像以下三个自解释的示例那样完成：

```java
DataType<RateType> RATETYPE = SALE.RATE.getDataType();
```

```java
DataType<VatType> VATTYPE 
```

```java
  = VARCHAR.asEnumDataType(VatType.class);
```

```java
DataType<com.classicmodels.enums.VatType> VATTYPE
```

```java
  = VARCHAR.asEnumDataType(jooq.generated.enums.VatType.class)
```

```java
    .asConvertedDataType(VAT_CONVERTER);
```

现在，您可以使用此数据类型作为任何其他数据类型。接下来，让我们讨论数据类型重写的话题。

# 数据类型重写

`<forcedTypes/>`的另一个实用功能是数据类型重写。这允许我们显式选择在 Java 中应使用的 SQL 数据类型（由数据库支持，或不支持但在`org.jooq.impl.SQLDataType`中存在）。

例如，在 Oracle 中，一个常见的用例是将缺失的`BOOLEAN`类型映射为`NUMBER(1,0)`或`CHAR(1)`，如下所示：

```java
CREATE TABLE sale (
```

```java
  ...
```

```java
  hot NUMBER(1,0) DEFAULT 0
```

```java
  hot CHAR(1) DEFAULT '1' CHECK (hot IN('1', '0'))
```

```java
  ...
```

```java
}
```

但这意味着 jOOQ 代码生成器将字段类型`NUMBER(1, 0)`映射到`SQLDataType.TINYINT` SQL 数据类型和`java.lang.Byte`类型，并将类型为`CHAR(1)`的字段映射到`SQLDataType.CHAR` SQL 数据类型和`String` Java 类型。

但 Java 的`String`类型通常与文本数据处理相关联，而`Byte`类型通常与二进制数据处理相关联（例如，读取/写入二进制文件），Java 的`Boolean`类型则清楚地表达了使用标志类型数据的意图。此外，Java 的`Boolean`类型有一个与`SQLDataType.BOOLEAN`同源的 SQL 类型（标准 JDBC 类型）。

jOOQ 允许我们强制指定列的类型，因此我们可以强制将`SALE.HOT`的类型设置为`BOOLEAN`，如下所示：

```java
<forcedType>
```

```java
  <name>BOOLEAN</name>
```

```java
  <includeExpression>CLASSICMODELS\.SALE\.HOT</includeExpression>
```

```java
  <includeTypes>NUMBER\(1,\s*0\)</includeTypes>
```

```java
  <includeTypes>CHAR\(1\)</includeTypes>
```

```java
</forcedType>
```

完成！现在，我们可以将`SALE.HOT`视为 Java `Boolean`类型。以下是一个`INSERT`示例：

```java
ctx.insertInto(SALE, ..., SALE.HOT)
```

```java
   .values(2005,..., Boolean.FALSE)
```

```java
   .execute();
```

根据`NUMBER`的精度，jOOQ 将此数据类型映射到`BigInteger`、`Short`，甚至`Byte`（正如您刚才看到的）。如果您觉得使用此类 Java 类型很麻烦，并且知道您的数据更适合`Long`或`Integer`类型，那么您有两个选择：相应地调整`NUMBER`精度，或者依赖 jOOQ 类型重写。当然，您可以将此技术应用于任何其他类型和方言。

一个完整的示例可以在*DataTypeRewriting*中找到。这个示例的程序版本被称为*ProgrammaticDataTypeRewriting*。接下来，让我们了解如何处理 jOOQ 的可嵌入类型。

# 处理可嵌入类型

可嵌入类型是 jOOQ 3.14 版本引入的一个强大功能。大致来说，这个功能通过合成 UDT（用户定义类型）来实现，这些 UDT 可以与 jOOQ 支持的所有数据库一起使用。虽然 PostgreSQL 和 Oracle 支持 UDT（我们可以在**数据定义语言**（**DDL**）中直接使用 UDT），但包括 MySQL 和 SQL Server 在内的其他数据库不支持 UDT。但是，通过 jOOQ 的可嵌入类型，我们可以在应用级别使用合成 UDT 与任何数据库一起工作，jOOQ 将负责将这些类型映射到数据库的底层方面。

可嵌入类型通过在生成的`org.jooq.EmbeddableRecord`中合成地包装一个（通常是多个）数据库列来模拟 UDT。例如，我们可以通过以下 jOOQ 代码生成器中的配置将`OFFICE.CITY`、`OFFICE.STATE`、`OFFICE.COUNTRY`、`OFFICE.TERRITORY`和`OFFICE.ADDRESS_LINE_FIRST`包装在名为`OFFICE_FULL_ADDRESS`的可嵌入类型下（这里针对 MySQL）：

```java
<embeddable>
```

```java
 <!-- The optional catalog of the embeddable type -->
```

```java
 <catalog/>
```

```java
 <!-- The optional schema of the embeddable type -->
```

```java
 <schema>classicmodels</schema>
```

```java
 <!-- The name of the embeddable type -->
```

```java
 <name>OFFICE_FULL_ADDRESS</name>
```

```java
 <!-- An optional, defining comment of an embeddable -->
```

```java
 <comment>The full address of an office</comment>
```

```java
 <!-- The name of the reference to the embeddable type -->
```

```java
 <referencingName/>
```

```java
 <!-- An optional, referencing comment of an embeddable -->
```

```java
 <referencingComment/>
```

我们继续设置匹配表和字段，如下所示：

```java
 <!-- A regular expression matching qualified/unqualified 
```

```java
      table names to which to apply this embeddable. If left 
```

```java
      blank, this will apply to all tables -->
```

```java
 <tables>.*\.office</tables>
```

```java
 <!-- A list of fields to match to an embeddable. Each field   
```

```java
      must match exactly one column in each matched table. A 
```

```java
      mandatory regular expression matches field names, and  
```

```java
      an optional name can be provided to define the 
```

```java
      embeddable attribute name. If no name is provided, then 
```

```java
      the first matched field's name will be taken -->
```

```java
 <fields>
```

```java
  <field><expression>CITY</expression></field>                                              
```

```java
  <field><expression>ADDRESS_LINE_FIRST</expression></field>
```

```java
  <field><expression>STATE</expression></field>
```

```java
  <field><expression>COUNTRY</expression></field>                                                
```

```java
  <field><expression>TERRITORY</expression></field>
```

```java
 </fields>                                                                             </embeddable>                                        
```

接下来，jOOQ 生成`jooq...records.OfficeFullAddressRecord`，它扩展了`EmbeddableRecordImpl`和`jooq...pojos.OfficeFullAddress`。此外，在生成的`Office`表中，我们观察到一个新的`OFFICE_FULL_ADDRESS`字段，它可以在以下`INSERT`语句中使用：

```java
ctx.insertInto(OFFICE, ... ,
```

```java
               OFFICE.ADDRESS_LINE_SECOND, ...)
```

```java
   .values(...,
```

```java
           new OfficeFullAddressRecord("Naples", "Giuseppe 
```

```java
              Mazzini", "Campania", "Italy", "N/A"),
```

```java
           ...)
```

```java
   .execute();
```

显然，`OFFICE_FULL_ADDRESS`列可以在所有类型的语句中使用，包括`INSERT`、`UPDATE`、`DELETE`和`SELECT`。在这里，它被用在`SELECT`语句中：

```java
Result<Record1<OfficeFullAddressRecord>> result 
```

```java
       = ctx.select(OFFICE.OFFICE_FULL_ADDRESS).from(OFFICE)
```

```java
   .fetch();
```

或者，它也可以被检索到`OfficeFullAddress` POJO 中，如下所示：

```java
List<OfficeFullAddress> result 
```

```java
   = ctx.select(OFFICE.OFFICE_FULL_ADDRESS).from(OFFICE)
```

```java
        .fetchInto(OfficeFullAddress.class);
```

在捆绑的代码中，对于 MySQL，我们有*EmbeddableType*，它包含前面的示例，而对于 PostgreSQL，我们有*ProgrammaticEmbeddableType*，它是前面示例的程序化版本。

## 替换字段

到目前为止，我们可以访问（可以使用）可嵌入类型，但我们仍然可以直接访问这个可嵌入类型包装的字段。例如，这些字段可以用在`INSERT`语句、`SELECT`语句等中，并且它们出现在**集成开发环境**（**IDE**）的自动完成列表中。

*替换字段*功能意味着向 jOOQ 发出信号，不允许直接访问可嵌入类型的一部分的字段。这些字段将不再出现在 IDE 的自动完成列表中，并且`SELECT`语句的结果集将不包含这些字段。可以通过`<replacesFields/>`标志启用此功能，如下所示：

```java
<embeddable>
```

```java
 ...
```

```java
 <replacesFields>true</replacesFields>
```

```java
</embeddable>
```

*EmbeddableTypeReplaceFields*应用程序包含 Oracle 的此示例，而*ProgrammaticEmbeddableTypeReplaceFields*包含 SQL Server 的此示例的程序化版本。

## 转换可嵌入类型

转换可嵌入类型可以通过`org.jooq.Converter`完成，就像对任何其他类型一样。例如，在`JsonNode`和`OFFICE_FULL_ADDRESS`之间进行转换可以通过一个以如下方式开始的`Converter`完成：

```java
public class JsonConverter implements   
```

```java
           Converter<OfficeFullAddressRecord, JsonNode> {
```

```java
   public static final JsonConverter JSON_CONVERTER 
```

```java
      = new JsonConverter();
```

```java
   @Override
```

```java
   public JsonNode from(OfficeFullAddressRecord t) { ... }
```

```java
   @Override 
```

```java
   public OfficeFullAddressRecord to(JsonNode u) { ... }
```

```java
   ...
```

```java
}
```

在这里，这是一个`SELECT`语句，通过`JSON_CONVERTER`获取`OFFICE.OFFICE_FULL_ADDRESS`作为`JsonNode`：

```java
List<JsonNode> result = ctx.select(OFFICE.OFFICE_FULL_ADDRESS)
```

```java
   .from(OFFICE)
```

```java
   .fetch(OFFICE.OFFICE_FULL_ADDRESS, JSON_CONVERTER);
```

*ConvertEmbeddableType*应用程序包含 MySQL 的此示例。

## 嵌入式域

在 PostgreSQL 中非常流行，*域*类型是在其他类型之上构建的 UDT，并包含可选约束。例如，在我们的 PostgreSQL 模式中，我们有以下域：

```java
CREATE DOMAIN postal_code AS varchar(15)
```

```java
CHECK(
```

```java
   VALUE ~ '^\d{5}$'
```

```java
OR VALUE ~ '^[A-Z]{2}[0-9]{3}[A-Z]{2}$'
```

```java
);
```

它在`office`表中使用，如下所示：

```java
CREATE TABLE office (
```

```java
  ... 
```

```java
  "postal_code" postal_code NOT NULL,
```

```java
  ... 
```

```java
);
```

如果我们启用此功能，jOOQ 可以为每个域类型生成一个 Java 类型，如下所示：

```java
// Maven and standalone
```

```java
<database>                               
```

```java
  ...
```

```java
  <embeddableDomains>.*</embeddableDomains>
```

```java
</database>
```

```java
// Gradle
```

```java
database {
```

```java
  embeddableDomains = '.*'
```

```java
}
```

```java
// programmatic
```

```java
withEmbeddableDomains(".*")
```

虽然`.*`匹配所有域类型，但你可以使用更严格的正则表达式来精确匹配将被可嵌入类型替换的域。

jOOQ 代码生成器生成一个默认名为`PostalCodeRecord`（在`jooq.generated.embeddables.records`中）的可嵌入类型。我们可以用它来创建语义上类型安全的查询，如下例所示：

```java
ctx.select(OFFICE.CITY, OFFICE.COUNTRY)
```

```java
   .from(OFFICE)
```

```java
   .where(OFFICE.POSTAL_CODE.in(
```

```java
        new PostalCodeRecord("AZ934VB"),
```

```java
        new PostalCodeRecord("DT975HH")))
```

```java
   .fetch();
```

```java
ctx.insertInto(OFFICE, ..., OFFICE.POSTAL_CODE, ...)
```

```java
   .values(..., new PostalCodeRecord("OP909DD"), ...)
```

```java
   .execute();
```

PostgreSQL 的完整代码命名为*Domain*。

好吧，我们已经到达了本节的结尾和本章的结尾。请注意，我们故意跳过了可嵌入类型和可嵌入键（包括复合键）的主题，因为这个主题将在*第十一章*，*jOOQ 键*中稍后讨论。

# 摘要

这一章是你在 jOOQ 工具箱中不可或缺的一部分。掌握这里涵盖的主题——例如自定义数据类型、转换器、绑定、数据库厂商特定的数据类型、枚举、可嵌入类型等——将帮助你调整 Java 和数据库数据类型之间的交互，以适应你的复杂场景。在下一章中，我们将介绍数据检索和映射的主题。
