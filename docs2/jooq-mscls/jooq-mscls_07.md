# *第五章*：处理不同类型的 SELECT、INSERT、UPDATE、DELETE 和 MERGE

jOOQ 初学者常见的场景来自于有一个应该通过 jOOQ DSL API 表达的标准有效的 SQL。虽然 jOOQ DSL API 非常直观且易于学习，但缺乏实践仍然可能导致我们无法找到或直观地找到应该链式调用的适当 DSL 方法来表达特定的 SQL。

本章通过一系列流行的查询来解决这个问题，这给了你基于 Java 模式练习 jOOQ DSL 语法的机遇。更确切地说，我们的目标是使用 jOOQ DSL 语法表达、收集我们日常工作中使用的`SELECT`、`INSERT`、`UPDATE`、`DELETE`和`MERGE`语句的精心挑选列表。

这样，到本章结束时，你应该已经通过 jOOQ DSL 语法过滤了大量的 SQL 语句，并在基于 Maven 和 Gradle 的 Java 应用程序中针对 MySQL、PostgreSQL、SQL Server 和 Oracle 数据库进行了尝试。由于 jOOQ DSL 是无方言的，它擅长通过模拟有效语法来处理大量的方言特定问题，因此这也是尝试这四个流行数据库这一方面的好机会。

注意，即使你看到了一些性能提示，我们的重点并不是寻找最佳 SQL 或针对特定用例的最优 SQL。这并不是我们的目标！我们的目标是学习 jOOQ DSL 语法，达到一个能够以高效方式编写几乎所有`SELECT`、`INSERT`、`UPDATE`、`DELETE`和`MERGE`语句的水平。

在这种情况下，我们的议程包括以下内容：

+   表达`SELECT`语句

+   表达`INSERT`语句

+   表达`UPDATE`语句

+   表达`DELETE`语句

+   表达`MERGE`语句

让我们开始吧！

# 技术要求

本章的代码可以在 GitHub 上找到，链接为[`github.com/PacktPublishing/jOOQ-Masterclass/tree/master/Chapter05`](https://github.com/PacktPublishing/jOOQ-Masterclass/tree/master/Chapter05)。

# 表达 SELECT 语句

在本节中，我们将通过 jOOQ DSL 语法表达/编写一系列`SELECT`语句，包括常见的投影、流行的子查询、标量相关子查询、并集和行值表达式。我们将从常用的投影开始。

## 表达常用投影

通过**常用投影**，我们理解的是针对众所周知的虚拟表`DUAL`的投影。正如你可能知道的，`DUAL`表是 Oracle 特有的；在 MySQL 中（尽管 jOOQ 仍然为了 MySQL 5.7 的兼容性而生成它），它通常是不必要的，而在 PostgreSQL 和 SQL Server 中则不存在。

在此上下文中，即使 SQL 标准要求一个`FROM`子句，jOOQ 也从不要求这样的子句，并在需要/支持时渲染`DUAL`表。例如，选择`*0*`和`*1*`可以通过`selectZero()`和`selectOne()`方法完成（这些静态方法在`org.jooq.impl.DSL`中可用）。下面将举例说明后者（`selectOne()`），包括一些替代方案：

```java
MySQL 8.x                          : select 1 as `one`
```

```java
PostgreSQL (no dummy-table concept): select 1 as "one"
```

```java
ctx.selectOne().fetch();
```

```java
ctx.select(val(1).as("one")).fetch();
```

```java
ctx.fetchValue((val(1).as("one")));
```

作为旁注，DSL 类还公开了三个用于表达常用`0`字面量（`DSL.zero()`）、`1`字面量（`DSL.one()`）和`2`字面量（`DSL.two()`）的辅助器。因此，虽然`selectZero()`会导致一个新的 DSL 子查询，用于常量`0`字面量，但`zero()`本身代表`0`字面量。选择特定值可以如下进行（由于我们无法在`select()`中使用普通值，我们依赖于在*第三章*，*jOOQ 核心概念*中引入的`val()`方法来获取适当的参数）：

```java
Oracle: select 1 "A", 'John' "B", 4333 "C", 0 "D" from dual
```

```java
ctx.select(val(1).as("A"), val("John").as("B"), 
```

```java
           val(4333).as("C"), val(false).as("D")).fetch();
```

或者，可以通过`values()`表构造函数来完成，这允许我们表达内存中的临时表。在 jOOQ 中，`values()`表构造函数可以用来创建可以在`SELECT`语句的`FROM`子句中使用的表。注意我们如何为`values()`构造函数指定列别名（“派生列列表”）以及表别名（`"t"`）：

```java
MySQL: 
```

```java
select `t`.`A`, ..., `t`.`D`        
```

```java
  from (select null as `A`, ..., null as `D`            
```

```java
          where false 
```

```java
          union all
```

```java
            select * from 
```

```java
              (values row ('A', 'John', 4333, false)) as `t`
```

```java
       ) as `t`
```

```java
ctx.select().from(values(row("A", "John", 4333, false))
```

```java
   .as("t", "A", "B", "C", "D")).fetch();
```

这里是`selectOne()`的另一个替代方案：

```java
PostgreSQL (no dummy-table concept): 
```

```java
select "t"."one" from (values (1)) as "t" ("one")
```

```java
ctx.select().from(values(row(1)).as("t", "one")).fetch();
```

我们还可以指定一个显式的`FROM`子句来指出一些特定的表。以下是一个示例：

```java
SQL Server: 
```

```java
select 1 [one] from [classicmodels].[dbo].[customer], 
```

```java
                    [classicmodels].[dbo].[customerdetail]
```

```java
ctx.selectOne().from(CUSTOMER, CUSTOMERDETAIL).fetch();
```

当然，提供`selectOne()`及其类似功能的目的并不是真正为了允许查询`ctx.selectOne().fetch()`，而是在投影无关的查询中使用，如下例所示：

```java
ctx.deleteFrom(SALE)
```

```java
   .where(exists(selectOne().from(EMPLOYEE) 
```

```java
// .whereExists(selectOne().from(EMPLOYEE)
```

```java
      .where(SALE.EMPLOYEE_NUMBER.eq(EMPLOYEE.EMPLOYEE_NUMBER)
```

```java
        .and(EMPLOYEE.JOB_TITLE.ne("Sales Rep")))))
```

```java
   .execute();
```

在本书附带代码中，你可以找到更多未在此列出的示例。花些时间探索`CommonlyUsedProjections`应用程序。接下来，让我们解决`SELECT`子查询或子选择。

## 表达`SELECT`以获取所需的数据

从 jOOQ DSL 开始，可能的目标是简单的`SELECT`查询，例如`SELECT all_columns FROM table`或`SELECT * FROM table`类型。这种查询可以用 jOOQ 写成如下形式：

```java
ctx.select().from(ORDER)
```

```java
   .where(ORDER.ORDER_ID.eq(10101L)).fetch();
```

```java
ctx.selectFrom(ORDER)
```

```java
   .where(ORDER.ORDER_ID.eq(10101L)).fetch();
```

```java
ctx.select(ORDER.fields()).from(ORDER)
```

```java
   .where(ORDER.ORDER_ID.eq(10101L)).fetch();
```

由于我们依赖于生成的基于 Java 的模式（如你在*第二章*，*自定义 jOOQ 参与级别*)中看到的，jOOQ 可以推断出`ORDER`表的字段（列），并在生成的查询中明确地渲染它们。但是，如果你需要渲染`*`本身而不是字段列表，则可以使用方便的`asterisk()`方法，如下面的查询所示：

```java
ctx.select(asterisk()).from(ORDER)
```

```java
   .where(ORDER.ORDER_ID.eq(10101L))
```

```java
   .fetch();
```

如 Lukas Eder 所提到的："*也许在这里更加强调一下，星号 (*`*`*) 并不等于查询所有列的其他三种方式。星号 (*`*`*) 会从实时数据库模式中投影所有列，包括 jOOQ 不知道的列。其他三种方法会投影 jOOQ 知道的列，但这些列可能不再存在于实时数据库模式中。可能存在不匹配，这在映射到记录（例如，使用* `selectFrom()`* 或 *`into(recordtype)`*）时尤其重要。即使如此，当使用 *`*`* 并且在 *`from()`* 中的所有表都为 jOOQ 所知时，jOOQ 将尝试展开星号以访问所有转换器和数据类型绑定，以及可嵌入的记录和其他事物*。"

此外，请注意，此类查询可能会获取比所需更多的数据，而依赖 `*` 而不是列的列表可能会带来性能惩罚，这在本文中有所讨论：[`tanelpoder.com/posts/reasons-why-select-star-is-bad-for-sql-performance/`](https://tanelpoder.com/posts/reasons-why-select-star-is-bad-for-sql-performance/)。当我提到 "*可能获取比所需更多的数据*" 时，我指的是仅处理获取结果集的子集的场景，而其余的则被简单地丢弃。获取数据可能是一项昂贵的操作（尤其是耗时操作），因此，只是为了丢弃而获取数据是一种资源浪费，并且可能导致长时间运行的交易，影响应用程序的可扩展性。这在基于 JPA 的应用程序中是一个常见的场景（例如，在 Spring Boot 中，`spring.jpa.open-in-view=true`可能会导致加载比所需更多的数据）。

在其他方面，Tanel Poder 的文章提到一个很多初学者容易忽视的问题。通过强制数据库执行 "*无用的强制工作*"（你一定会喜欢这篇文章：[`blog.jooq.org/2017/03/08/many-sql-performance-problems-stem-from-unnecessary-mandatory-work/`](https://blog.jooq.org/2017/03/08/many-sql-performance-problems-stem-from-unnecessary-mandatory-work/)) 通过 `*` 投影，它将无法应用一些查询优化，例如，连接消除，这对于复杂查询是至关重要的 ([`tanelpoder.com/posts/reasons-why-select-star-is-bad-for-sql-performance/#some-query-plan-optimizations-not-possible`](https://tanelpoder.com/posts/reasons-why-select-star-is-bad-for-sql-performance/#some-query-plan-optimizations-not-possible))。

### 获取列的子集

一般来说，获取比所需更多的数据是持久层性能惩罚的常见原因。因此，如果你只需要 `ORDER` 表的子集列，那么只需在 `SELECT` 中明确列出它们，例如 `select(ORDER.ORDER_ID, ORDER.ORDER_DATE, ORDER.REQUIRED_DATE, ORDER.SHIPPED_DATE, ORDER.CUSTOMER_NUMBER)`。有时，所需的子集列几乎等于（但不等于）字段/列的总数。在这种情况下，与其像之前那样列出子集列，不如通过 `except()` 方法指出应排除的字段/列。以下是一个从 `ORDER` 中获取所有字段/列，除了 `ORDER.COMMENTS` 和 `ORDER.STATUS` 的示例：

```java
ctx.select(asterisk().except(ORDER.COMMENTS, ORDER.STATUS))
```

```java
   .from(ORDER)
```

```java
   .where(ORDER.ORDER_ID.eq(10101L)).fetch();
```

这里有一个例子，它将 SQL `nvl()` 函数应用于 `OFFICE.CITY` 字段。每当 `OFFICE.CITY` 为 `null` 时，我们获取 `N/A` 字符串：

```java
ctx.select(nvl(OFFICE.CITY, "N/A"),
```

```java
               OFFICE.asterisk().except(OFFICE.CITY))
```

```java
   .from(OFFICE).fetch();
```

如果你需要给一个条件附加别名，那么我们首先需要通过 `field()` 方法将这个条件包装在一个字段中。以下是一个示例：

```java
ctx.select(field(SALE.SALE_.gt(5000.0)).as("saleGt5000"),
```

```java
                 SALE.asterisk().except(SALE.SALE_))
```

```java
   .from(SALE).fetch();
```

此外，这个查询的结果集表头看起来如下：

![](img/B16833_Figure_5.1.jpg)

注意，`* EXCEPT (...)` 语法灵感来源于 BigQuery，它也计划实现一个 `* REPLACE (...)` 语法。你可以在这里跟踪其进度：[`github.com/jOOQ/jOOQ/issues/11198`](https://github.com/jOOQ/jOOQ/issues/11198)。

在本书附带代码（`SelectOnlyNeededData`）中，你可以看到更多使用 `asterisk()` 和 `except()` 方法进行操作以实现涉及一个或多个表的场景的示例。

### 获取行子集

除了使用谓词外，通常通过 `LIMIT ... OFFSET` 子句来获取行子集。不幸的是，这个子句不是 SQL 标准的一部分，并且只有有限数量的数据库供应商（如 MySQL 和 PostgreSQL）理解它。尽管如此，jOOQ 允许我们通过 `limit()` 和 `offset()` 方法使用 `LIMIT ... OFFSET`，并将处理所有用于使用方言的兼容语法方面。以下是一个渲染 `LIMIT 10 OFFSET 5` 的示例：

```java
ctx.select(EMPLOYEE.FIRST_NAME, 
```

```java
           EMPLOYEE.LAST_NAME, EMPLOYEE.SALARY)
```

```java
   .from(EMPLOYEE)
```

```java
   .orderBy(EMPLOYEE.SALARY)
```

```java
   .limit(10)
```

```java
   .offset(5)
```

```java
   .fetch();
```

这里是相同的事情（相同的查询和结果），但通过 `limit(Number offset, Number numberOfRows)` 风味表达（请注意，偏移量是第一个参数——从 MySQL 继承的参数顺序）：

```java
ctx.select(EMPLOYEE.FIRST_NAME, 
```

```java
           EMPLOYEE.LAST_NAME,EMPLOYEE.SALARY)
```

```java
   .from(EMPLOYEE)
```

```java
   .orderBy(EMPLOYEE.SALARY)
```

```java
   .limit(5, 10)
```

```java
   .fetch();
```

此外，jOOQ 将根据数据库供应商渲染以下 SQL：

```java
MySQL and PostgreSQL (jOOQ 3.14):
```

```java
select ... from ...limit 10 offset 5
```

```java
PostgreSQL (jOOQ 3.15+):
```

```java
select ... from ...offset 5 rows fetch first 10 rows only
```

```java
SQL Server:
```

```java
select ... from ...offset 5 rows fetch next 10 rows only
```

```java
Oracle:
```

```java
select ... from ...offset 5 rows fetch next 10 rows only
```

通常，`LIMIT` 和 `OFFSET` 的参数是一些硬编码的整数。但是，jOOQ 允许我们使用 `Field`。例如，在这里我们使用一个标量子查询在 `LIMIT` 中（同样可以在 `OFFSET` 中做同样的事情）：

```java
ctx.select(ORDERDETAIL.ORDER_ID, ORDERDETAIL.PRODUCT_ID,            ORDERDETAIL.QUANTITY_ORDERED)
```

```java
   .from(ORDERDETAIL)
```

```java
   .orderBy(ORDERDETAIL.QUANTITY_ORDERED)                        
```

```java
   .limit(field(select(min(ORDERDETAIL.QUANTITY_ORDERED)).       from(ORDERDETAIL)))                        
```

```java
   .fetch();
```

作为 Lukas Eder 的笔记，“*从版本 3.15 开始，jOOQ 为 PostgreSQL 中的`OFFSET … FETCH`生成标准 SQL，而不是供应商特定的*`LIMIT … OFFSET`*。这是为了提供对*`FETCH NEXT … ROWS WITH TIES`*的原生支持。也许，未来的 jOOQ 也会提供 SQL 标准 2011 中 Oracle 的/SQL Server 的语法：*`OFFSET n ROW[S] FETCH { FIRST | NEXT } m [ PERCENT ] ROW[S] { ONLY | WITH TIES }`。”跟踪这里：[`github.com/jOOQ/jOOQ/issues/2803`](https://github.com/jOOQ/jOOQ/issues/2803)。

注意，之前的查询使用了显式的`ORDER BY`来避免不可预测的结果。如果我们省略`ORDER BY`，那么 jOOQ 将在需要时代表我们模拟它。例如，`OFFSET`（与`TOP`不同）在 SQL Server 中需要`ORDER BY`，如果我们省略`ORDER BY`，那么 jOOQ 将代表我们生成以下 SQL：

```java
SQL Server:
```

```java
select ... from ...
```

```java
   order by (select 0)
```

```java
   offset n rows fetch next m rows only
```

由于我们提到了模拟这个话题，让我们提醒一下你应该注意的事项。

重要提示

jOOQ 最酷的特性之一是能够在用户数据库缺少特定功能时模拟有效的和最优的 SQL 语法（建议阅读这篇文章：[`blog.jooq.org/2018/03/13/top-10-sql-dialect-emulations-implemented-in-jooq/`](https://blog.jooq.org/2018/03/13/top-10-sql-dialect-emulations-implemented-in-jooq/))。jOOQ 文档提到，“*未使用`org.jooq.Support`注解（*`@Support`*）的 jOOQ API 方法，或者使用`@Support`注解但没有指定任何 SQL 方言的方法，可以在所有 SQL 方言中安全使用。上述*`@Support`*注解不仅指定了哪些数据库原生支持某个功能。它还表明，对于缺少此功能的某些数据库，jOOQ 会模拟该功能*。”此外，每当 jOOQ 不支持某些供应商特定的功能/语法时，解决方案是使用纯 SQL 模板。这一点将在本书的后续章节中详细说明。

更多示例，请考虑本书附带代码。你将找到 15+ 个示例，包括几个边缘情况。例如，在 `EXAMPLE 10.1` 和 `10.2` 中，你可以看到通过 jOOQ 的 `sortAsc()` 方法按特定顺序检索行的示例（如果你处于这种位置，我建议你也阅读这篇文章：[`blog.jooq.org/2014/05/07/how-to-implement-sort-indirection-in-sql/`](https://blog.jooq.org/2014/05/07/how-to-implement-sort-indirection-in-sql/))。或者，在 `EXAMPLE 11` 中，你可以看到如何通过 jOOQ 的 `org.jooq.Comparator` API 和一个布尔变量在运行时选择 `WHERE ... IN` 和 `WHERE ... NOT IN` 语句。此外，在 `EXAMPLE 15` 和 `16` 中，你可以看到使用 `SelectQuery` API 从不同表中检索列的用法。花时间练习这些示例。我非常确信，你会从中学到很多技巧。应用程序名为 `SelectOnlyNeededData`。现在，让我们来谈谈如何在 jOOQ 中表达子查询。

## 表达 SELECT 子查询（子选择）

大概来说，一个 `SELECT` 子查询（或子选择）是由嵌套在另一个 `SELECT` 语句中的 `SELECT` 语句表示的。通常，它们出现在 `WHERE` 或 `FROM` 子句中，但它们出现在 `HAVING` 子句或与数据库视图结合使用也不足为奇。

例如，让我们以以下包含子查询的 `WHERE` 子句的普通 SQL 语句为例，作为谓语的一部分：

```java
SELECT first_name, last_name FROM employee
```

```java
WHERE office_code IN
```

```java
    (SELECT office_code FROM office WHERE city LIKE "S%")
```

在 jOOQ 中，我们可以直接表达这个查询：

```java
ctx.select(EMPLOYEE.FIRST_NAME, EMPLOYEE.LAST_NAME)
```

```java
   .from(EMPLOYEE)
```

```java
   .where(EMPLOYEE.OFFICE_CODE.in(
```

```java
       select(OFFICE.OFFICE_CODE).from(OFFICE)
```

```java
          .where(OFFICE.CITY.like("S%")))).fetch()
```

注意我们是如何通过 jOOQ 的 `in()` 方法使用 `IN` 语句的。同样地，你可以使用 jOOQ 支持的其他语句，例如 `NOT IN` (`notIn()`), `BETWEEN` (`between()`), `LIKE` (`like()`)，以及许多其他语句。始终注意使用 `NOT IN`，并且要注意它来自子查询的关于 `NULL` 的特殊行为（[`www.jooq.org/doc/latest/manual/sql-building/conditional-expressions/in-predicate/`](https://www.jooq.org/doc/latest/manual/sql-building/conditional-expressions/in-predicate/))。

几乎任何 SQL 语句都有一个 jOOQ 等价实现，因此，花时间扫描 jOOQ API 尽可能多地覆盖它。这是成为 jOOQ 高级用户的正确方向。

重要提示

在类型为 `SELECT foo...` 的子查询中，`(SELECT buzz)` 是一个常见的案例，可以使用 `DSLContext.select()` 和 `DSL.select()` 作为 `ctx.select(foo) ... (select(buzz))`。`DSLContext.select()` 方法用于外部的 `SELECT` 以获取数据库配置连接的引用，而对于内部或嵌套的 `SELECT`，我们可以使用 `DSL.select()` 或 `DSLContext.select()`。然而，对于内部 `SELECT` 使用 `DSL.select()` 更方便，因为它可以静态导入并简单地作为 `select()` 引用。请注意，使用 `DSL.select()` 对两种类型的 `SELECT` 或仅对内部 `SELECT` 使用会导致类型为 *Cannot execute query. No Connection configured* 的异常。但当然，您仍然可以通过 `DSLContext.fetch(ResultQuery)` 等方式执行 `DSL.select()`。

接下来，让我们看看另一个简单的 SQL，它在 `FROM` 子句中有一个子查询，如下所示：

```java
SELECT sale_id, sale
```

```java
FROM sale,
```

```java
  (SELECT avg(sale) AS avgs,employee_number AS sen
```

```java
     FROM sale
```

```java
     GROUP BY employee_number) AS saleTable
```

```java
WHERE (employee_number = saleTable.sen
```

```java
  AND sale < saleTable.avgs);
```

这次，让我们通过派生表以下列方式表达子查询：

```java
// Table<Record2<BigDecimal, Long>>
```

```java
var saleTable = select(avg(SALE.SALE_).as("avgs"),  
```

```java
                       SALE.EMPLOYEE_NUMBER.as("sen"))
```

```java
   .from(SALE)
```

```java
   .groupBy(SALE.EMPLOYEE_NUMBER)
```

```java
   .asTable("saleTable"); // derived table
```

另一种方法依赖于 `table(select(…))`。实际上，`table(Select<R>)` 是 `asTable()` 的同义词。选择您认为更流畅的一个：

```java
var saleTable = table(select(...)).as("saleTable");
```

接下来，我们可以使用这个派生表来表示外部的 `SELECT`：

```java
ctx.select(SALE.SALE_ID, SALE.SALE_)
```

```java
   .from(SALE, saleTable)
```

```java
   .where(SALE.EMPLOYEE_NUMBER
```

```java
     .eq(saleTable.field("sen", Long.class))
```

```java
       .and(SALE.SALE_
```

```java
         .lt(saleTable.field("avgs", Double.class))))
```

```java
   .fetch();
```

由于 jOOQ 无法推断用户定义的派生表的字段类型，我们可以依靠强制转换来填充 `sen` 和 `avgs` 字段的预期类型（为了快速回顾 *强制转换* 的目标，请重新阅读 *第三章*，*jOOQ 核心概念*），或 `saleTable.field(name, type)`，就像这里一样。

jOOQ API 非常灵活和丰富，它允许我们以多种方式表达相同的 SQL。这取决于我们在特定场景中选择最方便的方法。例如，如果我们认为 SQL 部分 `WHERE (employee_number = saleTable.sen AND sale < saleTable.avgs)` 可以写成 `WHERE (employee_number = sen AND sale < avgs)`，那么我们可以提取以下字段作为局部变量：

```java
Field<Double> avgs = avg(SALE.SALE_)
```

```java
  .coerce(Double.class).as("avgs");
```

```java
Field<Long> sen = SALE.EMPLOYEE_NUMBER.as("sen");
```

然后，我们可以在派生表和外部的 `SELECT` 中使用它们：

```java
var saleTable = select(avgs, sen)
```

```java
   .from(SALE)
```

```java
   .groupBy(SALE.EMPLOYEE_NUMBER)
```

```java
   .asTable("saleTable"); // derived table
```

```java
ctx.select(SALE.SALE_ID, SALE.SALE_)
```

```java
   .from(SALE, saleTable)
```

```java
   .where(SALE.EMPLOYEE_NUMBER.eq(sen)
```

```java
      .and(SALE.SALE_.lt(avgs)))
```

```java
   .fetch();
```

或者，我们可以消除显式导出的表，并以流畅的方式嵌入子查询：

```java
ctx.select(SALE.SALE_ID, SALE.SALE_)
```

```java
   .from(SALE, select(avgs, sen)
```

```java
               .from(SALE)
```

```java
               .groupBy(SALE.EMPLOYEE_NUMBER))
```

```java
  .where(SALE.EMPLOYEE_NUMBER.eq(sen)
```

```java
    .and(SALE.SALE_.lt(avgs)))
```

```java
  .fetch();
```

注意，我们还移除了显式的 `saleTable` 别名。因为 MySQL 抱怨（并且不仅仅是），每个派生表都需要一个别名，但我们不必担心。jOOQ 会代表我们生成别名（类似于 `alias_25088691`）。但是，如果您的基准测试表明别名生成不可忽略，那么提供显式别名会更好。正如 Lukas Eder 所说，“*然而，生成的别名是基于 SQL 字符串的确定性，为了稳定，这对于执行计划缓存（例如，Oracle、SQL Server，在 MySQL 和 PostgreSQL 中无关紧要）很重要。””

在捆绑的代码中考虑更多示例。例如，如果您对在 `INSERT`、`UPDATE` 和 `DELETE` 中嵌套 `SELECT` 感兴趣，那么您可以在捆绑的代码中找到示例。应用程序名为 *SampleSubqueries*。接下来，让我们谈谈标量子查询。

## 表达标量子查询

标量子查询只选择一个列/表达式，并返回一行。它可以在 SQL 查询的任何位置使用，只要可以使用列/表达式的地方。例如，让我们假设一个简单的 SQL 查询，该查询选择薪水大于或等于平均薪水加 *25,000* 的员工：

```java
SELECT first_name, last_name FROM employee
```

```java
WHERE salary >= (SELECT (AVG(salary) + 25000) FROM employee);
```

在 jOOQ 中，这个查询是通过以下代码生成的：

```java
ctx.select(EMPLOYEE.FIRST_NAME, EMPLOYEE.LAST_NAME)
```

```java
   .from(EMPLOYEE)
```

```java
   .where(EMPLOYEE.SALARY.coerce(BigDecimal.class)
```

```java
      .ge(select(avg(EMPLOYEE.SALARY).plus(25000))
```

```java
   .from(EMPLOYEE))).fetch();
```

你注意到 `...ge(select(...))` 结构了吗？在 `ge()` 的左边我们有一个 `Field`，但在右边我们有一个 `Select`。这是由于 `ge(Select<? extends Record1<T>> select)` 的存在，这是一个非常方便的快捷方式，可以节省我们显式使用 `field()` 作为 `ge(field(select(...)))`，或者使用 `asField()` 作为 `ge(select(...).asField())` 的麻烦。我们也可以写出像 `select(...).ge(select(...))` 这样的条件。但是，`field()` 和 `asField()` 之间有什么区别？

让我们再举一个例子，这个例子中插入了一个新的 `PRODUCT`（我只列出了相关部分）：

```java
ctx.insertInto(PRODUCT, 
```

```java
               PRODUCT.PRODUCT_ID, ..., PRODUCT.MSRP)
```

```java
   .values(...,
```

```java
     field(select(avg(PRODUCT.MSRP)).from(PRODUCT)))
```

```java
   .execute();
```

将 `PRODUCT.MSRP` 的值作为平均 `Field` 插入。这可以通过我们之前通过 `field(Select<? extends Record1<T>> s)` 或通过 `asField()` 做到。如果我们选择 `asField()`，那么我们会写出类似以下的内容：

```java
...select(avg(PRODUCT.MSRP)).from(PRODUCT).asField())
```

但是，`asField()` 方法返回的结果提供者是一个 `Field<?/Object>` 对象。换句话说，`asField()` 丢失了类型信息，因此不是类型安全的。通过丢失类型信息，`asField()` 允许我们意外地引入类型安全相关的错误，这些错误直到运行时才能检测到，甚至更糟糕的是，会产生意外的结果。在这里，我们使用了 `array(PRODUCT.MSRP)` 而不是 `avg(PRODUCT.MSRP)`，但我们直到运行时都没有任何抱怨：

```java
...select(array(PRODUCT.MSRP)).from(PRODUCT).asField())
```

当然，你不会写出这样的废话，但想法是，在这样的上下文中使用 `asField()` 容易出现其他数据类型不兼容的问题，这些问题可能很难发现，并可能产生意外的结果。所以，让我们保留 `asField()` 用于查询，如 `SELECT b.*, (SELECT foo FROM a) FROM b`，并专注于 `field()`：

```java
... field(select(array(PRODUCT.MSRP)).from(PRODUCT)))
```

你认为这段代码会编译吗？正确的答案是不会！你的 IDE 会立即发出数据类型不兼容的信号。虽然 `PRODUCT.MSRP` 是 `BigDecimal` 类型，但 `(array(PRODUCT.MSRP))` 是 `Field<BigDecimal[]>` 类型，所以 `INSERT` 是错误的。将 `array()` 替换为 `avg()`。问题解决！

在捆绑的代码（`ScalarSubqueries`）中，你会有更多示例，包括在 `INSERT`、`UPDATE` 和 `DELETE` 中使用嵌套的标量查询。接下来，让我们谈谈相关子查询。

## 表达相关子查询

相关子查询（或重复子查询）使用外部查询的值来计算其值。由于它依赖于外部查询，相关子查询不能作为一个独立的子查询独立执行。正如 Lukas Eder 提到的，“*在任何情况下，没有 RDBMS 被迫对每个由外部查询评估的行天真地执行一次相关子查询（显然，如果这种情况发生，那么这可能会带来性能开销，如果相关子查询必须执行多次）。许多 RDBMS 会通过将转换应用于连接或半连接来优化相关子查询。其他，如 Oracle 11g 及以后版本，通过标量子查询缓存来优化相关子查询。” ([`blogs.oracle.com/oraclemagazine/on-caching-and-evangelizing-sql`](https://blogs.oracle.com/oraclemagazine/on-caching-and-evangelizing-sql))

让我们考虑以下表示相关标量子查询的简单 SQL：

```java
SELECT s1.sale, s1.fiscal_year, s1.employee_number
```

```java
FROM sale AS s1
```

```java
WHERE s1.sale =
```

```java
    (SELECT max(s2.sale)
```

```java
     FROM sale AS s2
```

```java
     WHERE (s2.employee_number = s1.employee_number
```

```java
            AND s2.fiscal_year = s1.fiscal_year))
```

```java
ORDER BY s1.fiscal_year
```

在 jOOQ 中表达这个查询可以这样进行（注意我们希望在渲染的 SQL 中保留`s1`和`s2`别名）：

```java
Sale s1 = SALE.as("s1");
```

```java
Sale s2 = SALE.as("s2");
```

```java
ctx.select(s1.SALE_, s1.FISCAL_YEAR, s1.EMPLOYEE_NUMBER)
```

```java
   .from(s1)
```

```java
   .where(s1.SALE_.eq(select(max(s2.SALE_))
```

```java
      .from(s2)
```

```java
      .where(s2.EMPLOYEE_NUMBER.eq(s1.EMPLOYEE_NUMBER)
```

```java
         .and(s2.FISCAL_YEAR.eq(s1.FISCAL_YEAR)))))
```

```java
   .orderBy(s1.FISCAL_YEAR)
```

```java
   .fetch();
```

你是否注意到了这个：`...where(s1.SALE_.eq(select(max(s2.SALE_))`？不需要`asField()` /`field()`！查询不需要调用`asField()`/`field()`，因为 jOOQ 提供了一个便利的重载，即`Field<T>.eq(Select<? extends Record1<T>>)`。是的，我知道我之前已经告诉过你，但我只是想再次强调。

然而，正如你可能直觉到的，这个相关子查询，它依赖于重复自连接的表，可以通过以下`GROUP BY`更有效地表达（这次，我们不保留别名）：

```java
ctx.select(SALE.FISCAL_YEAR, 
```

```java
           SALE.EMPLOYEE_NUMBER, max(SALE.SALE_))
```

```java
   .from(SALE)
```

```java
   .groupBy(SALE.FISCAL_YEAR, SALE.EMPLOYEE_NUMBER)
```

```java
   .orderBy(SALE.FISCAL_YEAR)
```

```java
   .fetch();
```

在这种情况下，`GROUP BY`要好得多，因为它消除了自连接，将*O(n*2*)*转换为*O(n)*。正如 Lukas Eder 分享的，“*随着现代 SQL 的发展，自连接几乎不再真正需要。初学者可能会认为这些自连接是可行的，但它们可能会带来很大的损害，*[`twitter.com/MarkusWinand/status/1118147557828583424`](https://twitter.com/MarkusWinand/status/1118147557828583424)*，最坏情况下为 O(n*2)*。”所以，在跳入编写这样的相关子查询之前，尝试评估一些替代方案并比较执行计划。

让我们看看以下简单 SQL 的另一个例子：

```java
SELECT product_id, product_name, buy_price
```

```java
FROM product
```

```java
WHERE
```

```java
  (SELECT avg(buy_price)
```

```java
   FROM product) > ANY
```

```java
     (SELECT price_each
```

```java
      FROM orderdetail
```

```java
      WHERE product.product_id = orderdetail.product_id);
```

因此，这个查询比较了所有产品的平均购买价格（或标价）与每个产品的所有销售价格。如果这个平均值大于某个产品的任何销售价格，那么该产品将被检索到结果集中。在 jOOQ 中，可以这样表达：

```java
ctx.select(PRODUCT.PRODUCT_ID, 
```

```java
           PRODUCT.PRODUCT_NAME, PRODUCT.BUY_PRICE)
```

```java
   .from(PRODUCT)
```

```java
   .where(select(avg(PRODUCT.BUY_PRICE))
```

```java
   .from(PRODUCT).gt(any(
```

```java
        select(ORDERDETAIL.PRICE_EACH)
```

```java
           .from(ORDERDETAIL)
```

```java
              .where(PRODUCT.PRODUCT_ID
```

```java
                 .eq(ORDERDETAIL.PRODUCT_ID)))))
```

```java
   .fetch();
```

在捆绑的代码中，你有一个包含 `WHERE (NOT) EXISTS`，`ALL` 和 `ANY` 等示例的完整列表。正如 Lukas Eder 所说，“*值得指出的是，ALL 和 ANY 的名字是 '量词'，比较称为量词比较谓词。这些比与 *`MIN()`* 或 *`MAX()`* 比较或使用 *`ORDER BY .. LIMIT 1`* 更优雅，尤其是在使用行值表达式时*。” 此外，你还可以查看使用嵌套在 `INSERT`，`UPDATE` 和 `DELETE` 中的相关子查询的示例。该应用程序名为 *CorrelatedSubqueries*。接下来，让我们来谈谈在 jOOQ 中编写行表达式。

## 表达行表达式

行值表达式对于编写优雅的多行谓词非常有用。jOOQ 通过 `org.jooq.Row` 接口表示行值表达式。其用法简单，如下面的普通 SQL 所示：

```java
SELECT customer_number, address_line_first,   
```

```java
       address_line_second, city, state, postal_code, country
```

```java
FROM customerdetail
```

```java
WHERE (city, country) IN(SELECT city, country FROM office)
```

在 jOOQ 中，这可以通过 `row()` 表达，如下所示：

```java
ctx.selectFrom(CUSTOMERDETAIL)
```

```java
   .where(row(CUSTOMERDETAIL.CITY, CUSTOMERDETAIL.COUNTRY)
```

```java
         .in(select(OFFICE.CITY, OFFICE.COUNTRY).from(OFFICE)))
```

```java
.fetch();
```

在捆绑的代码（`RawValueExpression`）中，你可以练习使用行值表达式与比较谓词、`BETWEEN` 和 `OVERLAPS` 谓词（jOOQ 支持重叠日期和任意行值表达式的 2 度 – 这有多酷？！）以及 `NULL` 的示例。接下来，让我们来处理 `UNION` 和 `UNION ALL` 操作符。

## 表达 UNION 和 UNION ALL 操作符

`UNION` 和 `UNION ALL` 操作符用于将来自不同 `SELECT` 语句或 `SELECT` 的两个或多个结果集合并为一个结果集。`UNION` 从 `SELECT` 语句的结果中消除重复行，而 `UNION ALL` 不做此操作。要正常工作，两个查询中的列数和顺序必须对应，并且数据类型必须相同或至少兼容。让我们考虑以下 SQL：

```java
SELECT concat(first_name, ' ', last_name) AS full_name,
```

```java
  'Employee' AS contactType
```

```java
FROM employee
```

```java
UNION
```

```java
SELECT concat(contact_first_name, ' ', contact_last_name),
```

```java
  'Customer' AS contactType
```

```java
FROM customer;
```

jOOQ 通过 `union()` 方法渲染 `UNION`，通过 `unionAll()` 方法渲染 `UNION ALL`。前面的 SQL 通过 `union()` 渲染如下：

```java
ctx.select(
```

```java
      concat(EMPLOYEE.FIRST_NAME, inline(" "),
```

```java
      EMPLOYEE.LAST_NAME).as("full_name"),
```

```java
      inline("Employee").as("contactType"))
```

```java
   .from(EMPLOYEE)
```

```java
   .union(select(
```

```java
          concat(CUSTOMER.CONTACT_FIRST_NAME, inline(" "),
```

```java
          CUSTOMER.CONTACT_LAST_NAME),
```

```java
         inline("Customer").as("contactType"))
```

```java
   .from(CUSTOMER))
```

```java
   .fetch();
```

Lukas Eder 指出，“*`UNION` (*`ALL`*) 在处理 `NULL` 方面与其他操作符不同，这意味着两个 `NULL` 值是 '不区分的'。因此，`SELECT NULL UNION SELECT NULL` 只产生一行，就像 `SELECT NULL INTERSECT SELECT NULL`*。”

在捆绑的代码中，你可以练习更多示例，包括 `UNION` 和 `ORDER BY`，`UNION` 和 `LIMIT`，`UNION` 和 `HAVING`，`UNION` 和 `SELECT INTO`（MySQL 和 PostgreSQL），`UNION ALL` 等。不幸的是，这里没有足够的空间来列出并分析这些示例，因此，请考虑名为 *SelectUnions* 的应用程序。接下来，让我们来探讨 `INTERSECT` 和 `EXCEPT` 操作符。

## 表达 INTERSECT (ALL) 和 EXCEPT (ALL) 操作符

`INTERSECT`运算符仅产生由（或共同于）两个子查询返回的值（行）。`EXCEPT`运算符（或 Oracle 中的`MINUS`）仅产生出现在第一个（或左侧）子查询中且不出现在第二个（或右侧）子查询中的值。虽然`INTERSECT`和`EXCEPT`会从其结果中删除重复项，但`INTERSECT ALL`和`EXCEPT ALL`不会这样做。与`UNION`的情况一样，要正常工作，两个查询中的列数和顺序必须对应，并且数据类型必须相同或至少兼容。

让我们考虑以下纯 SQL：

```java
SELECT buy_price FROM product
```

```java
INTERSECT
```

```java
SELECT price_each FROM orderdetail
```

在 jOOQ 中，这可以通过以下`intersect()`方法表达（对于渲染`INTERSECT ALL`，使用`intersectAll()`方法）：

```java
ctx.select(PRODUCT.BUY_PRICE)
```

```java
   .from(PRODUCT)
```

```java
   .intersect(select(ORDERDETAIL.PRICE_EACH)
```

```java
      .from(ORDERDETAIL))
```

```java
   .fetch();
```

通过将 SQL 中的`INTERSECT`替换为`EXCEPT`，并在 jOOQ 中将`intersect()`方法替换为`except()`，我们可以获得一个`EXCEPT`用例（对于`EXCEPT ALL`，使用`exceptAll()`方法）。以下是纯 SQL（这次，让我们添加一个`ORDER BY`子句）：

```java
SELECT buy_price FROM product
```

```java
EXCEPT
```

```java
SELECT price_each FROM orderdetail
```

```java
ORDER BY buy_price
```

jOOQ 的代码如下：

```java
ctx.select(PRODUCT.BUY_PRICE)
```

```java
   .from(PRODUCT)
```

```java
   .except(select(ORDERDETAIL.PRICE_EACH).from(ORDERDETAIL))
```

```java
   .orderBy(PRODUCT.BUY_PRICE)
```

```java
   .fetch();
```

然而，如果你的数据库不支持这些运算符（例如，MySQL），那么你必须模拟它们。有几种方法可以实现这一点，在名为*IntersectAndExcept*的应用程序中（用于 MySQL），你可以看到基于`IN`（当没有重复项或`NULL`时很有用）和`WHERE EXISTS`模拟`INTERSECT`（`ALL`）的非详尽解决方案列表，并基于`LEFT OUTER JOIN`和`WHERE NOT EXISTS`模拟`EXCEPT`（`ALL`）。当然，你也可以查看*IntersectAndExcept*的 MySQL、SQL Server 和 Oracle 的示例。注意，Oracle 18c（用于我们的应用程序）仅支持`INTERSECT`和`EXCEPT`，而 Oracle20c 支持所有这四个运算符。接下来，让我们解决众所周知的`SELECT DISTINCT`和更多内容。

## 表达独特性

jOOQ 提供了一套方法来表达查询中的独特性。我们有以下方法：

+   `SELECT DISTINCT`通过`selectDistinct()`

+   `IS` (`NOT`) `DISTINCT FROM`通过`isDistinctFrom()`和`isNotDistinctFrom()`

+   `COUNT (DISTINCT...)`通过`countDistinct()`

+   `AVG`/`SUM`/`MIN`/`MAX (DISTINCT ...)`通过`avg`/`sum`/`min`/`maxDistinct()`

+   PostgreSQL `DISTINCT ON`通过`selectDistinct().on()`或`distinctOn()`

让我们看看以下`IS DISTINCT FROM`的示例（在 MySQL 中，`IS DISTINCT FROM`由`<=>`运算符表示）：

```java
SELECT office.office_code, ...
```

```java
FROM office
```

```java
JOIN customerdetail
```

```java
  ON office.postal_code = customerdetail.postal_code
```

```java
WHERE (not((office.city, office.country) <=>
```

```java
      (customerdetail.city, customerdetail.country)))
```

jOOQ 通过以下代码片段呈现此查询：

```java
ctx.select()
```

```java
   .from(OFFICE)
```

```java
   .innerJoin(CUSTOMERDETAIL)
```

```java
     .on(OFFICE.POSTAL_CODE.eq(CUSTOMERDETAIL.POSTAL_CODE))
```

```java
   .where(row(OFFICE.CITY, OFFICE.COUNTRY).isDistinctFrom(
```

```java
          row(CUSTOMERDETAIL.CITY, CUSTOMERDETAIL.COUNTRY)))
```

```java
   .fetch();
```

根据使用的方言，jOOQ 模拟正确的语法。对于 MySQL，jOOQ 呈现`<=>`运算符；对于 Oracle，它依赖于`DECODE`和`INTERSECT`；对于 SQL Server，它依赖于`INTERSECT`。PostgreSQL 支持`IS DISTINCT FROM`。

接下来，让我们看看 PostgreSQL 的`DISTINCT ON`的一个示例：

```java
SELECT DISTINCT ON (product_vendor, product_scale) 
```

```java
    product_id, product_name, ...
```

```java
FROM product
```

```java
ORDER BY product_vendor, product_scale
```

jOOQ 的代码如下：

```java
ctx.selectDistinct()
```

```java
   .on(PRODUCT.PRODUCT_VENDOR, PRODUCT.PRODUCT_SCALE)
```

```java
   .from(PRODUCT)
```

```java
   .orderBy(PRODUCT.PRODUCT_VENDOR, PRODUCT.PRODUCT_SCALE)
```

```java
   .fetch();
```

肯定值得在这里提到的是，jOOQ 通过`row_number()`窗口函数模拟了针对 MySQL、SQL Server、Oracle 等特定于 PostgreSQL 的`DISTINCT ON`。

当然，我们也可以编写模拟 `DISTINCT ON` 的 jOOQ 查询。例如，以下示例通过 jOOQ 的 `rowNumber()` 和 `qualify()` 获取每个财政年度最大销售额的员工编号：

```java
ctx.selectDistinct(SALE.EMPLOYEE_NUMBER, 
```

```java
                   SALE.FISCAL_YEAR, SALE.SALE_)
```

```java
   .from(SALE)
```

```java
   .qualify(rowNumber().over(
```

```java
      partitionBy(SALE.FISCAL_YEAR)
```

```java
         .orderBy(SALE.FISCAL_YEAR, SALE.SALE_.desc())).eq(1))
```

```java
   .orderBy(SALE.FISCAL_YEAR)
```

```java
   .fetch();
```

通过 `DISTINCT ON` 解决的经典场景依赖于选择一些不同的列（s）同时按其他列排序。例如，以下查询依赖于 PostgreSQL 的 `DISTINCT ON` 来获取按最小销售额排序的不同员工编号：

```java
ctx.select(field(name("t", "employee_number")))
```

```java
   .from(select(SALE.EMPLOYEE_NUMBER, SALE.SALE_)
```

```java
      .distinctOn(SALE.EMPLOYEE_NUMBER)
```

```java
      .from(SALE)
```

```java
      .orderBy(SALE.EMPLOYEE_NUMBER, SALE.SALE_).asTable("t"))
```

```java
   .orderBy(field(name("t", "sale")))
```

```java
   .fetch();
```

当然，也可以在不使用 `DISTINCT ON` 的情况下进行模拟。这里是一个替代方案：

```java
ctx.select(field(name("t", "employee_number")))
```

```java
   .from(select(SALE.EMPLOYEE_NUMBER,
```

```java
       min(SALE.SALE_).as("sale"))
```

```java
     .from(SALE)
```

```java
     .groupBy(SALE.EMPLOYEE_NUMBER).asTable("t"))
```

```java
   .orderBy(field(name("t", "sale")))
```

```java
   .fetch();
```

在捆绑的代码（`SelectDistinctOn`）中，你可以找到涵盖之前列表中所有条目的示例。花些时间练习它们，熟悉 jOOQ 语法。此外，不要止步于这些示例；请尽可能多地用 `SELECT` 进行实验。

关于 `SELECT` 的内容就到这里。接下来，让我们开始讨论插入操作。

# 表达 INSERT 语句

在本节中，我们将通过 jOOQ DSL 语法表达不同类型的插入，包括 `INSERT ... VALUES`、`INSERT ... SET`、`INSERT ... RETURNING` 和 `INSERT ...DEFAULT VALUES`。让我们从众所周知的 `INSERT ... VALUES` 插入开始，这是大多数数据库供应商支持的。

### 表达 INSERT ... VALUES

jOOQ 通过 `insertInto()` 和 `values()` 方法支持 `INSERT ... VALUES`。可选地，我们可以使用 `columns()` 方法来区分我们插入的表名和插入的字段/列列表。为了触发实际的 `INSERT` 语句，我们必须显式调用 `execute()`；请注意这个方面，因为 jOOQ 新手往往会在插入/更新/删除表达式的末尾忘记这个调用。此方法返回受此 `INSERT` 语句影响的行数作为整数值（`0`），这意味着没有发生任何操作。

例如，以下 jOOQ 类型安全的表达式将生成一个可以成功执行至少 MySQL、PostgreSQL、SQL Server 和 Oracle 的 `INSERT` 语句（`ORDER` 表的主键 `ORDER.ORDER_ID` 是自动生成的，因此可以省略）：

```java
ctx.insertInto(ORDER,
```

```java
      ORDER.COMMENTS, ORDER.ORDER_DATE, ORDER.REQUIRED_DATE,
```

```java
      ORDER.SHIPPED_DATE, ORDER.STATUS, ORDER.CUSTOMER_NUMBER,
```

```java
      ORDER.AMOUNT)
```

```java
   .values("New order inserted...", LocalDate.of(2003, 2, 12),
```

```java
          LocalDate.of(2003, 3, 1), LocalDate.of(2003, 2, 27),
```

```java
          "Shipped", 363L, BigDecimal.valueOf(314.44))
```

```java
   .execute();
```

或者，使用 `columns()`，可以这样表达：

```java
ctx.insertInto(ORDER)
```

```java
   .columns(ORDER.COMMENTS, ORDER.ORDER_DATE,
```

```java
           ORDER.REQUIRED_DATE, ORDER.SHIPPED_DATE,
```

```java
           ORDER.STATUS, ORDER.CUSTOMER_NUMBER, ORDER.AMOUNT)
```

```java
   .values("New order inserted...", LocalDate.of(2003, 2, 12),
```

```java
          LocalDate.of(2003, 3, 1), LocalDate.of(2003, 2, 27),
```

```java
          "Shipped", 363L, BigDecimal.valueOf(314.44))
```

```java
   .execute();
```

如果省略了整个列/字段列表（例如，出于简洁性的原因），那么 jOOQ 表达式是非类型安全的，你必须为表中的每个字段/列显式指定一个值，包括代表自动生成主键的字段/列以及具有默认值的字段/列，否则你会得到一个异常，因为“值的数量必须与字段的数量相匹配”。此外，你必须注意值的顺序；只有当你遵循为我们在其中插入的表生成的 `Record` 类构造函数中定义的参数顺序时，jOOQ 才会将值与字段匹配（例如，在这个例子中，从 `OrderRecord` 构造函数的参数顺序）。正如卢卡斯·埃德补充说，“*`Record` 构造函数参数的顺序也像其他所有东西一样，是从 DDL 中声明的列的顺序中派生出来的，这始终是真理的来源*。”在这种情况下，为自动生成的主键（或其他字段）指定显式虚拟值可以依赖于几乎普遍（和标准 SQL）的方式，SQL `DEFAULT`，或者在 jOOQ 中使用 `DSL.default_()`/`DSL.defaultValue()`（尝试使用 `NULL` 代替 SQL `DEFAULT` 会导致特定实现的行为）：

```java
ctx.insertInto(ORDER)
```

```java
   .values(default_(), // Oracle, MySQL, PostgreSQL
```

```java
           LocalDate.of(2003, 2, 12), LocalDate.of(2003, 3, 1),
```

```java
           LocalDate.of(2003, 2, 27), "Shipped",
```

```java
           "New order inserted ...", 363L, 314.44)
```

```java
   .execute();
```

在 PostgreSQL 中，我们还可以使用 `ORDER_SEQ.nextval()` 调用；`ORDER_SEQ` 是与 `ORDER` 表关联的显式序列：

```java
ctx.insertInto(ORDER)
```

```java
   .values(ORDER_SEQ.nextval(), 
```

```java
           LocalDate.of(2003, 2, 12), LocalDate.of(2003, 3, 1),
```

```java
           LocalDate.of(2003, 2, 27), "Shipped",
```

```java
           "New order inserted ...", 363L, 314.44)
```

```java
    .execute();
```

一般而言，我们可以使用与表关联的显式或自动分配的序列（如果主键是 (`BIG`)`SERIAL` 类型），并调用 `nextval()` 方法。jOOQ 还定义了一个 `currval()` 方法，表示序列的当前值。

SQL Server 非常具有挑战性，因为它在将 `IDENTITY_INSERT` 设置为 `OFF` 时无法为标识列插入显式值 ([`github.com/jOOQ/jOOQ/issues/1818`](https://github.com/jOOQ/jOOQ/issues/1818))。在 jOOQ 提供出优雅的解决方案之前，你可以依赖三个查询的批次：一个查询将 `IDENTITY_INSERT` 设置为 `ON`，一个查询是 `INSERT`，最后一个查询将 `IDENTITY_INSERT` 设置为 `OFF`。但是，即便如此，这仅适用于指定显式有效的主键。不允许使用 SQL `DEFAULT` 或 `NULL` 值，或任何其他虚拟值作为显式标识值。SQL Server 将简单地尝试使用虚拟值作为主键，并最终引发错误。正如卢卡斯·埃德所说，“*在其他一些 RDBMS 中，如果你尝试插入一个显式值，即使自动生成的值是 *`GENERATED ALWAYS AS IDENTITY`*（与 *`GENERATED BY DEFAULT AS IDENTITY`* 相反），你仍然会得到一个异常*。”

无论主键是否为自动生成类型，如果你显式（手动）指定它为有效值（不是虚拟值也不是重复键），那么在这四个数据库中（当然，在 SQL Server 中，在 `IDENTITY_INSERT` 设置为 `ON` 的上下文中）`INSERT` 操作都会成功。

重要提示

省略列列表对解释`DEFAULT`和标识符/序列很有趣，但真的不建议在`INSERT`中省略列列表。所以，你最好努力使用列列表。

对于插入多行，你可以简单地为每行添加一个`values()`调用，以流畅的方式或使用循环遍历行列表（通常是记录列表）并重复使用相同的`values()`调用，但使用不同的值。最后，别忘了调用`execute()`。这种方法（以及更多）在捆绑代码中可用。但是，从 jOOQ 3.15.0 版本开始，这可以通过`valuesOfRecords()`或`valuesOfRows()`来完成。例如，考虑一个记录列表：

```java
List<SaleRecord> listOfRecord = List.of( ... );
```

我们可以通过`valuesOfRecords()`方法将此列表插入数据库，如下所示：

```java
ctx.insertInto(SALE, SALE.fields())
```

```java
   .valuesOfRecords(listOfRecord)
```

```java
   .execute();
```

这里有一个包含行列表的示例：

```java
var listOfRows
```

```java
  = List.of(row(2003, 3443.22, 1370L,
```

```java
            SaleRate.SILVER, SaleVat.MAX, 3, 14.55), 
```

```java
            row(...), ...);
```

这次，我们可以使用`valuesOfRows()`：

```java
ctx.insertInto(SALE,
```

```java
               SALE.FISCAL_YEAR, SALE.SALE_,
```

```java
               SALE.EMPLOYEE_NUMBER, SALE.RATE, SALE.VAT,
```

```java
               SALE.FISCAL_MONTH, SALE.REVENUE_GROWTH)
```

```java
   .valuesOfRows(listOfRows)
```

```java
   .execute();
```

当你需要将数据（例如，POJOs）收集到`RowN`的列表或数组中时，你可以使用内置的`toRowList()`或`toRowArray()`收集器。你可以在捆绑的代码中找到示例。

在其他想法中，插入`Record`可以通过几种方式完成；为了快速回顾 jOOQ 记录，请重新阅读*第三章*，*jOOQ 核心概念*。现在，让我们插入以下与`SALE`表对应的`SaleRecord`：

```java
SaleRecord sr = new SaleRecord();
```

```java
sr.setFiscalYear(2003); // or, sr.set(SALE.FISCAL_YEAR, 2003);
```

```java
sr.setSale(3443.22);    // or, sr.set(SALE.SALE_, 3443.22); 
```

```java
sr.setEmployeeNumber(1370L); 
```

```java
sr.setFiscalMonth(3);
```

```java
sr.setRevenueGrowth(14.55);
```

要插入`sr`，我们执行以下操作：

```java
ctx.insertInto(SALE)
```

```java
   .values(sr.getSaleId(), sr.getFiscalYear(), sr.getSale(), 
```

```java
       sr.getEmployeeNumber(), default_(), SaleRate.SILVER,  
```

```java
       SaleVat.MAX, sr.getFiscalMonth(), sr.getFiscalYear(), 
```

```java
       default_())
```

```java
   .execute()
```

或者，我们可以这样做：

```java
ctx.insertInto(SALE)
```

```java
   .values(sr.valuesRow().fields())
```

```java
   .execute();
```

或者，我们甚至可以这样做：

```java
ctx.executeInsert(sr);
```

或者，我们可以通过`attach()`方法将记录附加到当前配置：

```java
sr.attach(ctx.configuration());
```

```java
sr.insert();
```

尝试插入一个 POJO 需要我们首先将其包装在相应的`Record`中。这可以通过`newRecord()`方法完成，该方法可以从你的 POJO 或 jOOQ 生成的 POJO 中加载 jOOQ 生成的记录。以下是对 jOOQ 生成的`Sale` POJO 的示例：

```java
// jOOQ Sale POJO
```

```java
Sale sale = new Sale(null, 2005, 343.22, 1504L,
```

```java
     null, SaleRate.SILVER, SaleVat.MAX, 4, 15.55, null);
```

```java
ctx.newRecord(SALE, sale).insert();
```

另一种方法依赖于方便的`Record.from(POJO)`方法，如下所示（基本上，这次你使用`SaleRecord`的显式实例）：

```java
SaleRecord sr = new SaleRecord();
```

```java
sr.from(sale); // sale is the previous POJO instance
```

```java
ctx.executeInsert(sr);
```

`Record.from()`有几种风味，允许我们从数组或甚至值`Map`中填充`Record`。

当你需要重置`Record`主键（或其他字段）时，可以像以下场景中那样调用`reset()`方法，这会重置手动分配的主键，并允许数据库代为生成一个：

```java
Sale sale = new Sale(1L, 2005, 343.22, 1504L, null,    
```

```java
                SaleRate.SILVER, SaleVat.MAX, 6, 23.99, null);
```

```java
SaleRecord sr = new SaleRecord();
```

```java
sr.from(sale);
```

```java
// reset the current ID and allow DB to generate one
```

```java
sr.reset(SALE.SALE_ID);       
```

```java
ctx.executeInsert(sr);
```

然而，`reset()`方法会重置*已更改*标志（跟踪记录更改）和*值*（在这种情况下，主键）。如果你想只重置值（主键或其他字段），那么你可以依赖`changed``(Field<?> field, boolean changed)`方法，如下所示：

```java
record.changed(SALE.SALE_ID, false);
```

好吧，这些只是一些示例。更多示例，包括在 PostgreSQL 和 Oracle 中使用 UDTs（用户定义类型）以及在`INSERT`中使用用户定义函数，可以在名为*InsertValues*的应用程序捆绑代码中找到。接下来，让我们谈谈`INSERT ... SET`。

### 表达 INSERT ... SET

`INSERT ... SET`是`INSERT ... VALUES`的替代方案，具有类似`UPDATE`的语法，并且在 MySQL（但不仅限于此）中常用。实际上，在`INSERT ... SET`中，我们通过`set(field, value)`方法写入字段值对，这使得代码更易读，因为我们能轻松地识别每个字段的值。让我们看看插入两行数据的示例：

```java
ctx.insertInto(SALE)
```

```java
   .set(SALE.FISCAL_YEAR, 2005)      // first row
```

```java
   .set(SALE.SALE_, 4523.33)
```

```java
   .set(SALE.EMPLOYEE_NUMBER, 1504L)
```

```java
   .set(SALE.FISCAL_MONTH, 3)
```

```java
   .set(SALE.REVENUE_GROWTH, 12.22)
```

```java
   .newRecord()
```

```java
   .set(SALE.FISCAL_YEAR, 2005)      // second row
```

```java
   .set(SALE.SALE_, 4523.33)
```

```java
   .set(SALE.EMPLOYEE_NUMBER, 1504L)
```

```java
   .set(SALE.FISCAL_MONTH, 4)
```

```java
   .set(SALE.REVENUE_GROWTH, 22.12)
```

```java
   .execute();
```

此语法也适用于`Record`：

```java
SaleRecord sr = new SaleRecord(...);
```

```java
ctx.insertInto(SALE).set(sr).execute();
```

由于`INSERT … SET`和`INSERT … VALUES`是等效的，jOOQ 将所有由 jOOQ 支持的数据库中的`INSERT … SET`模拟为`INSERT … VALUES`。完整的应用程序命名为*InsertSet*。接下来，让我们来处理`INSERT ... RETURNING`语法。

### 表达 INSERT ... RETURNING

`INSERT ... RETURNING`的特殊之处在于它可以返回所插入的内容（检索我们需要进一步处理的内容）。这可能包括返回插入行的主键或其他字段（例如，其他序列、默认生成的值和触发器结果）。PostgreSQL 原生支持`INSERT ... RETURNING`。Oracle 也支持`INSERT ... RETURNING`，jOOQ 会为其生成 PL/SQL 匿名块（不一定总是）。SQL Server 支持`OUTPUT`，这与它几乎相同（除了触发器生成的值如何受影响）。其他数据库支持较差，jOOQ 必须代表我们模拟它。在这种情况下，jOOQ 依赖于 JDBC 的`getGeneratedKeys()`方法来检索插入的主键。此外，如果生成的主键（或其他列）不能直接检索，jOOQ 可能需要执行额外的`SELECT`来达到这个目的，这可能导致竞争条件（例如，在 MySQL 中可能需要这样的`SELECT`）。

`INSERT ... RETURNING`的 jOOQ API 包含`returningResult()`方法，它有多种形式。它带有不同的参数列表，允许我们指定应返回哪些字段。如果应返回所有字段，则无需参数直接使用它。如果只返回主键（这是一个流行的用例，例如 MySQL 的`AUTO_INCREMENT`或 PostgreSQL 的(`BIG`)`SERIAL`，这些会自动生成序列），则只需指定为`returningResult(pk_field)`。如果主键有多个字段（复合主键），则列出所有字段，字段之间用逗号分隔。

这里是一个返回单个插入操作主键的示例：

```java
// Record1<Long>
```

```java
var insertedId = ctx.insertInto(SALE, 
```

```java
      SALE.FISCAL_YEAR, SALE.SALE_, SALE.EMPLOYEE_NUMBER, 
```

```java
      SALE.REVENUE_GROWTH, SALE.FISCAL_MONTH)
```

```java
   .values(2004, 2311.42, 1370L, 10.12, 1)
```

```java
   .returningResult(SALE.SALE_ID)
```

```java
   .fetchOne();
```

由于只有一个结果，我们通过`fetchOne()`方法获取它。可以通过`fetch()`方法获取多个主键，如下所示：

```java
// Result<Record1<Long>>
```

```java
var insertedIds = ctx.insertInto(SALE, 
```

```java
      SALE.FISCAL_YEAR,SALE.SALE_, SALE.EMPLOYEE_NUMBER, 
```

```java
      SALE.REVENUE_GROWTH, SALE.FISCAL_MONTH)
```

```java
   .values(2004, 2311.42, 1370L, 12.50, 1)
```

```java
   .values(2003, 900.21, 1504L, 23.99, 2)
```

```java
   .values(2005, 1232.2, 1166L, 14.65, 3)
```

```java
   .returningResult(SALE.SALE_ID)
```

```java
   .fetch();
```

这次，返回的结果包含三个主键。返回更多/其他字段可以按以下方式完成（结果看起来像一个*n-列 x n-行*的表）：

```java
// Result<Record2<String, LocalDate>>
```

```java
var inserted = ctx.insertInto(PRODUCTLINE,  
```

```java
      PRODUCTLINE.PRODUCT_LINE, PRODUCTLINE.TEXT_DESCRIPTION, 
```

```java
      PRODUCTLINE.CODE)
```

```java
    .values(..., "This new line of electric vans ...", 983423L)
```

```java
    .values(..., "This new line of turbo N cars ...", 193384L)
```

```java
    .returningResult(PRODUCTLINE.PRODUCT_LINE, 
```

```java
                     PRODUCTLINE.CREATED_ON)
```

```java
   .fetch();
```

现在，让我们看看一个更有趣的例子。在我们的数据库模式中，`CUSTOMER` 和 `CUSTOMERDETAIL` 表是单向一对一关系，并共享主键值。换句话说，`CUSTOMER` 主键同时是 `CUSTOMERDETAIL` 中的主键和外键；这样，就没有必要维护单独的外键。因此，我们必须使用 `CUSTOMER` 返回的主键来插入 `CUSTOMERDETAIL` 表中相应的行：

```java
ctx.insertInto(CUSTOMERDETAIL)
```

```java
   .values(ctx.insertInto(CUSTOMER)
```

```java
      .values(default_(), ..., "Kyle", "Doyle", 
```

```java
              "+ 44 321 321", default_(), default_(), default_())
```

```java
      .returningResult(CUSTOMER.CUSTOMER_NUMBER).fetchOne()
```

```java
         .value1(), ..., default_(), "Los Angeles", 
```

```java
           default_(), default_(), "USA")
```

```java
   .execute();
```

如果你熟悉 JPA，那么你在这里可以认出一个优雅的 `@MapsId` 的替代方案。

第一个 `INSERT`（内部 `INSERT`）将在 `CUSTOMER` 表中插入一行，并通过 `returningResult()` 返回生成的主键。接下来，第二个 `INSERT`（外部 `INSERT`）将使用此返回的主键作为 `CUSTOMERDETAIL.CUSTOMER_NUMBER` 主键的值，在 `CUSTOMERDETAIL` 表中插入一行。

此外，`returningResult()` 方法还可以返回表达式，例如 `returningResult(A.concat(B).as("C"))`。

如往常一样，花些时间检查附带代码，其中包含更多示例。应用程序命名为 *InsertReturning*。接下来，让我们谈谈 `INSERT ... DEFAULT VALUES`。

### 表达 INSERT ... DEFAULT VALUES

插入默认值的直接方法是省略 `INSERT` 中具有默认值的字段。例如，看看以下代码：

```java
ctx.insertInto(PRODUCT)
```

```java
   .columns(PRODUCT.PRODUCT_NAME, PRODUCT.PRODUCT_LINE,   
```

```java
            PRODUCT.CODE, PRODUCT.PRODUCT_SCALE,
```

```java
            PRODUCT.PRODUCT_VENDOR, PRODUCT.BUY_PRICE, 
```

```java
            PRODUCT.MSRP)
```

```java
   .values("Ultra Jet X1", "Planes", 433823L, "1:18", 
```

```java
           "Motor City Art Classics",
```

```java
            BigDecimal.valueOf(45.9), BigDecimal.valueOf(67.9))
```

```java
   .execute();
```

未列出的 `PRODUCT` 字段（`PRODUCT_DESCRIPTION`、`PRODUCT_UID`、`SPECS` 和 `QUANTITY_IN_STOCK`）将利用隐式默认值。

jOOQ API 提供了 `defaultValues()`、`defaultValue()` 和 `default_()` 方法，用于显式指出应该依赖默认值的字段。前者用于插入只有默认值的单行；如果你检查数据库模式，你会注意到 `MANAGER` 表的每一列都有一个默认值：

```java
ctx.insertInto(MANAGER).defaultValues().execute();
```

另一方面，`defaultValue()` 方法（或 `default_()`）允许我们指向应该依赖默认值的字段：

```java
ctx.insertInto(PRODUCT)
```

```java
   .columns(PRODUCT.PRODUCT_NAME, PRODUCT.PRODUCT_LINE,
```

```java
     PRODUCT.CODE, PRODUCT.PRODUCT_SCALE,   
```

```java
     PRODUCT.PRODUCT_VENDOR, PRODUCT.PRODUCT_DESCRIPTION,  
```

```java
     PRODUCT.QUANTITY_IN_STOCK, PRODUCT.BUY_PRICE, 
```

```java
     PRODUCT.MSRP, PRODUCT.SPECS, PRODUCT.PRODUCT_UID)
```

```java
   .values(val("Ultra Jet X1"), val("Planes"), 
```

```java
           val(433823L),val("1:18"), 
```

```java
           val("Motor City Art Classics"),
```

```java
           defaultValue(PRODUCT.PRODUCT_DESCRIPTION),
```

```java
           defaultValue(PRODUCT.QUANTITY_IN_STOCK),
```

```java
           val(BigDecimal.valueOf(45.99)),  
```

```java
           val(BigDecimal.valueOf(67.99)),
```

```java
           defaultValue(PRODUCT.SPECS),  
```

```java
           defaultValue(PRODUCT.PRODUCT_UID))
```

```java
   .execute();
```

此示例的非类型安全版本如下：

```java
ctx.insertInto(PRODUCT)
```

```java
   .values(defaultValue(), "Ultra Jet X1", "Planes", 433823L,
```

```java
     defaultValue(), "Motor City Art Classics",
```

```java
     defaultValue(), defaultValue(), 45.99, 67.99,  
```

```java
     defaultValue(), defaultValue())
```

```java
   .execute();
```

```java
ctx.insertInto(PRODUCT)
```

```java
   .values(defaultValue(PRODUCT.PRODUCT_ID), 
```

```java
           "Ultra JetX1", "Planes", 433823L, 
```

```java
           defaultValue(PRODUCT.PRODUCT_SCALE),
```

```java
           "Motor City Art Classics",
```

```java
           defaultValue(PRODUCT.PRODUCT_DESCRIPTION),
```

```java
           defaultValue(PRODUCT.QUANTITY_IN_STOCK),
```

```java
           45.99, 67.99, defaultValue(PRODUCT.SPECS),
```

```java
           defaultValue(PRODUCT.PRODUCT_UID))
```

```java
   .execute()
```

通过指定列的类型也可以得到相同的结果。例如，之前的 `defaultValue(PRODUCT.QUANTITY_IN_STOCK)` 调用可以写成以下形式：

```java
defaultValue(INTEGER) // or, defaultValue(Integer.class)
```

使用默认值插入 `Record` 可以非常简单，如下面的示例所示：

```java
ctx.newRecord(MANAGER).insert();
```

```java
ManagerRecord mr = new ManagerRecord();
```

```java
ctx.newRecord(MANAGER, mr).insert();
```

使用默认值对于填充那些稍后将被更新（例如，通过后续更新、触发器生成的值等）的字段或如果我们根本没有值的情况非常有用。

完整的应用程序命名为 *InsertDefaultValues*。接下来，让我们谈谈 jOOQ 和 `UPDATE` 语句。

# 表达 UPDATE 语句

在本节中，我们将表达不同类型的更新，包括 `UPDATE ... SET`、`UPDATE ... FROM` 和 `UPDATE ... RETURNING`，并通过 jOOQ DSL 语法使用行值表达式进行更新。在撰写本文时，jOOQ 支持对单个表的更新，而对多个表的更新则是一个正在进行中的任务。

### 表达 UPDATE ... SET

直接的 `UPDATE ... SET` 语句可以通过 `set(field, value)` 方法在 jOOQ 中表达，如下面的例子所示（不要忘记调用 `execute()` 来触发更新）：

```java
ctx.update(OFFICE)
```

```java
   .set(OFFICE.CITY, "Banesti")
```

```java
   .set(OFFICE.COUNTRY, "Romania")
```

```java
   .where(OFFICE.OFFICE_CODE.eq("1"))
```

```java
   .execute();
```

生成的针对 MySQL 语言的 SQL 将如下所示：

```java
UPDATE `classicmodels`.`office`
```

```java
SET `classicmodels`.`office'.`city` = ?,
```

```java
    `classicmodels`.`office`.`country` = ?
```

```java
WHERE `classicmodels`.`office`.`office_code` = ?
```

看起来像是一个经典的 `UPDATE`，对吧？注意 jOOQ 自动只渲染更新的列。如果你来自 JPA，那么你知道 Hibernate JPA 默认渲染所有列，而我们必须依赖于 `@DynamicUpdate` 来获得与 jOOQ 相同的效果。

查看一个基于员工销售额计算的增加员工工资的示例：

```java
ctx.update(EMPLOYEE)
```

```java
   .set(EMPLOYEE.SALARY, EMPLOYEE.SALARY.plus(
```

```java
      field(select(count(SALE.SALE_).multiply(5.75)).from(SALE)
```

```java
   .where(EMPLOYEE.EMPLOYEE_NUMBER
```

```java
      .eq(SALE.EMPLOYEE_NUMBER)))))
```

```java
   .execute();
```

这里是针对 SQL Server 方言生成的 SQL：

```java
UPDATE [classicmodels].[dbo].[employee]
```

```java
SET [classicmodels].[dbo].[employee].[salary] = 
```

```java
  ([classicmodels].[dbo].[employee].[salary] +
```

```java
   (SELECT (count([classicmodels].[dbo].[sale].[sale]) * ?)
```

```java
    FROM [classicmodels].[dbo].[sale]
```

```java
    WHERE [classicmodels].[dbo].[employee].[employee_number] 
```

```java
        = [classicmodels].[dbo].[sale].[employee_number]))
```

注意，这是一个没有 `WHERE` 子句的 `UPDATE`，jOOQ 会记录一条消息，例如 `A statement is executed without WHERE clause`。这只是友好的信息，如果你故意省略了 `WHERE` 子句，你可以忽略它。但是，如果你知道这不是故意为之，那么你可能想通过依赖 jOOQ 的 `withExecuteUpdateWithoutWhere()` 设置来避免这种情况。你可以从几种行为中选择，包括抛出异常，就像这个例子中那样：

```java
ctx.configuration().derive(new Settings()
```

```java
   .withExecuteUpdateWithoutWhere(ExecuteWithoutWhere.THROW))
```

```java
   .dsl()
```

```java
   .update(OFFICE)
```

```java
   .set(OFFICE.CITY, "Banesti")
```

```java
   .set(OFFICE.COUNTRY, "Romania")                  
```

```java
   .execute();
```

注意，我们使用 `Configuration.derive()`，而不是 `Configuration.set()`，因为如果 `DSLContext` 被注入，`Configuration` 是全局的和共享的。使用 `Configuration.set()` 将会影响全局设置。如果这是期望的行为，那么最好依赖于一个单独的 `@Bean`，就像你在本书中已经看到的那样。

这次，无论何时我们尝试执行没有 `WHERE` 子句的 `UPDATE`，`UPDATE` 都不会采取任何行动，并且 jOOQ 会抛出一个 `org.jooq.exception.DataAccessException` 类型的异常。

更新 `Record` 也很简单。看看这个例子：

```java
OfficeRecord or = new OfficeRecord();
```

```java
or.setCity("Constanta");
```

```java
or.setCountry("Romania");
```

```java
ctx.update(OFFICE)
```

```java
   .set(or)
```

```java
   .where(OFFICE.OFFICE_CODE.eq("1")).execute();
```

```java
// or, like this
```

```java
ctx.executeUpdate(or, OFFICE.OFFICE_CODE.eq("1"));
```

正如你在捆绑的代码中所看到的，使用 `DSLContext.newRecord()` 也是一个选项。

### 使用行值表达式表达 UPDATE

使用行值表达式更新是一个非常实用的工具，jOOQ 以非常干净和直观的方式表达这样的更新。看看这个例子：

```java
ctx.update(OFFICE)
```

```java
    .set(row(OFFICE.ADDRESS_LINE_FIRST, 
```

```java
             OFFICE.ADDRESS_LINE_SECOND, OFFICE.PHONE),
```

```java
         select(EMPLOYEE.FIRST_NAME, EMPLOYEE.LAST_NAME, 
```

```java
             val("+40 0721 456 322"))
```

```java
           .from(EMPLOYEE)
```

```java
           .where(EMPLOYEE.JOB_TITLE.eq("President")))
```

```java
   .execute();
```

生成的针对 PostgreSQL 的 SQL 如下所示：

```java
UPDATE "public"."office"
```

```java
SET ("address_line_first", "address_line_second", "phone") =
```

```java
  (SELECT "public"."employee"."first_name",
```

```java
          "public"."employee"."last_name", ?
```

```java
   FROM "public"."employee"
```

```java
   WHERE "public"."employee"."job_title" = ?)
```

即使行值表达式特别适用于编写子查询，就像前面的例子中那样，这并不意味着你不能编写以下内容：

```java
ctx.update(OFFICE)
```

```java
   .set(row(OFFICE.CITY, OFFICE.COUNTRY),
```

```java
        row("Hamburg", "Germany"))
```

```java
   .where(OFFICE.OFFICE_CODE.eq("1"))
```

```java
   .execute();
```

这对于重用具有最小冗余的字段很有用：

```java
Row2<String, String> r1 = row(OFFICE.CITY, OFFICE.COUNTRY);
```

```java
Row2<String, String> r2 = row("Hamburg", "Germany");
```

```java
ctx.update(OFFICE).set(r1, r2).where(r1.isNull())
```

```java
   .execute();
```

接下来，让我们来处理 `UPDATE ... FROM` 语法。

### 表达 UPDATE ... FROM

使用`UPDATE ... FROM`语法，我们可以将额外的表连接到`UPDATE`语句。请注意，这个`FROM`子句在 PostgreSQL 和 SQL Server 中是供应商特定的支持，但在 MySQL 和 Oracle 中不受支持（然而，当你阅读这本书时，jOOQ 可能已经模拟了这个语法，所以请查看）。以下是一个示例：

```java
ctx.update(PRODUCT)
```

```java
   .set(PRODUCT.BUY_PRICE, ORDERDETAIL.PRICE_EACH)
```

```java
   .from(ORDERDETAIL)
```

```java
   .where(PRODUCT.PRODUCT_ID.eq(ORDERDETAIL.PRODUCT_ID))
```

```java
   .execute();
```

下面的示例展示了为 PostgreSQL 渲染的 SQL：

```java
UPDATE "public"."product"
```

```java
SET "buy_price" = "public"."orderdetail"."price_each"
```

```java
FROM "public"."orderdetail"
```

```java
WHERE "public"."product"."product_id" 
```

```java
    = "public"."orderdetail"."product_id"
```

最后，让我们来处理`UPDATE ... RETURNING`语法。

### 表达 UPDATE ... RETURNING

`UPDATE ... RETURNING`类似于`INSERT ... RETURNING`，但用于`UPDATE`。这是 PostgreSQL 原生支持的，并由 jOOQ 在 SQL Server 和 Oracle 中模拟。在 jOOQ DSL 中，我们通过`returningResult()`来表示`UPDATE ... RETURNING`，如下面的示例所示：

```java
ctx.update(OFFICE)
```

```java
   .set(OFFICE.CITY, "Paris")
```

```java
   .set(OFFICE.COUNTRY, "France")
```

```java
   .where(OFFICE.OFFICE_CODE.eq("1"))
```

```java
   .returningResult(OFFICE.CITY, OFFICE.COUNTRY)
```

```java
   .fetchOne();
```

下面的示例展示了为 PostgreSQL 渲染的 SQL：

```java
UPDATE "public"."office"
```

```java
SET "city" = ?,
```

```java
    "country" = ?
```

```java
WHERE "public"."office"."office_code" = ? 
```

```java
RETURNING "public"."office"."city",
```

```java
          "public"."office"."country"
```

我们可以使用`UPDATE ... RETURNING`来逻辑上链接多个更新。例如，假设我们想根据员工的平均销售额和客户的信用额度增加员工的薪水，并将返回的薪水乘以两倍。我们可以通过以下方式流畅地表达这两个`UPDATE`语句：

```java
ctx.update(CUSTOMER)
```

```java
   .set(CUSTOMER.CREDIT_LIMIT, CUSTOMER.CREDIT_LIMIT.plus(
```

```java
      ctx.update(EMPLOYEE)
```

```java
         .set(EMPLOYEE.SALARY, EMPLOYEE.SALARY.plus(
```

```java
            field(select(avg(SALE.SALE_)).from(SALE)
```

```java
                  .where(SALE.EMPLOYEE_NUMBER
```

```java
                     .eq(EMPLOYEE.EMPLOYEE_NUMBER)))))
```

```java
         .where(EMPLOYEE.EMPLOYEE_NUMBER.eq(1504L))
```

```java
         .returningResult(EMPLOYEE.SALARY
```

```java
            .coerce(BigDecimal.class))
```

```java
         .fetchOne().value1()
```

```java
            .multiply(BigDecimal.valueOf(2))))
```

```java
   .where(CUSTOMER.SALES_REP_EMPLOYEE_NUMBER.eq(1504L))
```

```java
   .execute();
```

然而，请注意潜在的竞争条件，因为在看似单个查询中隐藏了两次往返。

通过名为*UpdateSamples*的应用程序练习这个示例和其他许多示例。接下来，让我们来处理`DELETE`语句。

# 表达 DELETE 语句

在 jOOQ 中表达`DELETE`语句可以通过`DSLContext.delete()`和`DSLContext.deleteFrom()` API，或者通过`DSLContext.deleteQuery()`和`DSLContext.executeDelete()`来实现。虽然前三种方法接收`Table<R>`类型的参数，但`executeDelete()`方法对于将记录作为`TableRecord<?>`或`UpdatableRecord<?>`删除非常有用。正如以下示例所示，`delete()`和`deleteFrom()`的工作方式完全相同：

```java
ctx.delete(SALE)
```

```java
   .where(SALE.FISCAL_YEAR.eq(2003))
```

```java
   .execute();
```

```java
ctx.deleteFrom(SALE)
```

```java
   .where(SALE.FISCAL_YEAR.eq(2003))
```

```java
   .execute();
```

这两个表达式都渲染出以下 SQL：

```java
DELETE FROM `classicmodels`.`sale`
```

```java
WHERE `classicmodels`.`sale`.`fiscal_year` = ?
```

将`DELETE`和行值表达式结合使用对于通过子查询删除非常有用，如下面的示例所示：

```java
ctx.deleteFrom(CUSTOMERDETAIL)
```

```java
   .where(row(CUSTOMERDETAIL.POSTAL_CODE,
```

```java
              CUSTOMERDETAIL.STATE).in(
```

```java
                 select(OFFICE.POSTAL_CODE, OFFICE.STATE)
```

```java
                   .from(OFFICE)
```

```java
                   .where(OFFICE.COUNTRY.eq("USA"))))
```

```java
   .execute();
```

`DELETE`的一个重要方面是从父表到子表的级联删除。在可能的情况下，依靠数据库支持来完成`DELETE`级联任务是一个好主意。例如，你可以使用`ON DELETE CASCADE`或实现级联删除逻辑的存储过程。正如 Lukas Eder 强调的，“一般来说，应该*级联*组合（UML 术语）和*限制*或*无操作*，或者*设置为 NULL*（如果支持）聚合。换句话说，如果子表不能没有父表（组合）而存在，那么就与父表一起删除它。否则，抛出异常（*限制*、*无操作*），或将引用设置为*NULL*。jOOQ 可能在将来支持*DELETE ... CASCADE*：[`github.com/jOOQ/jOOQ/issues/7367`](https://github.com/jOOQ/jOOQ/issues/7367)。

然而，如果上述任何方法都不可行，那么您也可以通过 jOOQ 来完成。您可以编写一系列单独的 `DELETE` 语句，或者依赖于 `DELETE ... RETURNING`，如下面的示例所示，这些示例通过级联（`PRODUCTLINE` – `PRODUCTLINEDETAIL` – `PRODUCT` – `ORDERDETAIL`）删除 `PRODUCTLINE`。为了删除 `PRODUCTLINE`，我们必须从 `PRODUCT` 中删除所有其产品，并从 `PRODUCTLINEDETAIL` 中删除相应的记录。为了从 `PRODUCT` 中删除 `PRODUCTLINE` 的所有产品，我们必须删除这些产品的所有引用从 `ORDERDETAIL` 中。因此，我们从 `ORDERDETAIL` 开始删除，如下所示：

```java
ctx.delete(PRODUCTLINE)
```

```java
 .where(PRODUCTLINE.PRODUCT_LINE.in(
```

```java
   ctx.delete(PRODUCTLINEDETAIL)
```

```java
    .where(PRODUCTLINEDETAIL.PRODUCT_LINE.in(
```

```java
      ctx.delete(PRODUCT)
```

```java
       .where(PRODUCT.PRODUCT_ID.in(
```

```java
         ctx.delete(ORDERDETAIL)
```

```java
          .where(ORDERDETAIL.PRODUCT_ID.in(
```

```java
            select(PRODUCT.PRODUCT_ID).from(PRODUCT)
```

```java
             .where(PRODUCT.PRODUCT_LINE.eq("Motorcycles")
```

```java
               .or(PRODUCT.PRODUCT_LINE
```

```java
                 .eq("Trucks and Buses")))))
```

```java
         .returningResult(ORDERDETAIL.PRODUCT_ID).fetch()))
```

```java
       .returningResult(PRODUCT.PRODUCT_LINE).fetch()))
```

```java
     .returningResult(PRODUCTLINEDETAIL.PRODUCT_LINE).fetch()))
```

```java
 .execute();
```

这个 jOOQ 流畅表达式生成了四个 `DELETE` 语句，您可以在捆绑的代码中检查。这里的挑战在于确保在出错时能够保证回滚功能。但是，在 Spring Boot `@Transactional` 方法中拥有 jOOQ 表达式，回滚功能是现成的。这比通过 `CascadeType.REMOVE` 或 `orphanRemoval=true` 的 JPA 级联要好得多，这些方法非常容易引发 *N + 1* 问题。jOOQ 允许我们控制要删除的内容以及如何进行删除。

在其他方面，删除 `Record` (`TableRecord` 或 `UpdatableRecord`) 可以通过 `executeDelete()` 来完成，如下面的示例所示：

```java
PaymentRecord pr = new PaymentRecord();
```

```java
pr.setCustomerNumber(114L);    
```

```java
pr.setCheckNumber("GG31455");
```

```java
...
```

```java
// jOOQ render a WHERE clause based on the record PK
```

```java
ctx.executeDelete(pr);
```

```java
// jOOQ render our explicit Condition
```

```java
ctx.executeDelete(pr, 
```

```java
   PAYMENT.INVOICE_AMOUNT.eq(BigDecimal.ZERO));
```

正如在 `UPDATE` 的情况下，如果我们尝试在没有 `WHERE` 子句的情况下执行 `DELETE`，那么 jOOQ 将通过一条友好的消息通知我们。我们可以通过 `withExecuteDeleteWithoutWhere()` 设置来控制在这种情况下应该发生什么。

在捆绑的代码中，您可以看到 `withExecuteDeleteWithoutWhere()` 在许多其他未在此列出的示例旁边。整个应用程序被命名为 *DeleteSamples*。接下来，让我们来谈谈 `MERGE` 语句。

# 表达 MERGE 语句

`MERGE` 语句是一个非常强大的工具；它允许我们从称为 *目标表* 的表执行 `INSERT`/`UPDATE` 以及甚至 `DELETE` 操作，这个表是从称为 *源表* 的表中来的。我强烈建议您阅读这篇文章，尤其是如果您需要快速回顾 `MERGE` 语句的话：[`blog.jooq.org/2020/04/10/the-many-flavours-of-the-arcane-sql-merge-statement/`](https://blog.jooq.org/2020/04/10/the-many-flavours-of-the-arcane-sql-merge-statement/).

MySQL 和 PostgreSQL 通过 `ON DUPLICATE KEY UPDATE`（MySQL）和 `ON CONFLICT DO UPDATE`（PostgreSQL）子句分别支持一个名为 `UPSERT` 的 `MERGE` 风味（`INSERT` 或 `UPDATE`）。您可以在本书捆绑的代码中找到这些语句的示例，这些示例位于众所周知的 `INSERT IGNORE INTO`（MySQL）和 `ON CONFLICT DO NOTHING`（PostgreSQL）子句旁边。顺便说一句，我们可以互换使用所有这些语句（例如，我们可以使用 `onConflictDoNothing()` 与 MySQL，使用 `onDuplicateKeyIgnore()` 与 PostgreSQL），因为 jOOQ 总是会模拟正确的语法。我们甚至可以使用 SQL Server 和 Oracle，因为 jOOQ 会通过 `MERGE INTO` 语法来模拟它们。

SQL Server 和 Oracle 支持 `MERGE INTO` 与不同的附加子句。以下是一个利用 `WHEN MATCHED THEN UPDATE`（jOOQ `whenMatchedThenUpdate()`）和 `WHEN NOT MATCHED THEN INSERT`（jOOQ `whenNotMatchedThenInsert()`）子句的示例：

```java
ctx.mergeInto(PRODUCT)
```

```java
   .usingDual() // or, (ctx.selectOne())
```

```java
   .on(PRODUCT.PRODUCT_NAME.eq("1952 Alpine Renault 1300"))
```

```java
   .whenMatchedThenUpdate()
```

```java
   .set(PRODUCT.PRODUCT_NAME, "1952 Alpine Renault 1600")
```

```java
   .whenNotMatchedThenInsert(
```

```java
     PRODUCT.PRODUCT_NAME, PRODUCT.CODE)
```

```java
   .values("1952 Alpine Renault 1600", 599302L)
```

```java
   .execute();
```

SQL Server 方言的渲染 SQL 如下所示：

```java
MERGE INTO [classicmodels].[dbo].[product] USING
```

```java
  (SELECT 1 [one]) t ON  
```

```java
    [classicmodels].[dbo].[product].[product_name] = ? 
```

```java
WHEN MATCHED THEN
```

```java
  UPDATE
```

```java
  SET [classicmodels].[dbo].[product].[product_name] = ? 
```

```java
WHEN NOT MATCHED THEN
```

```java
  INSERT ([product_name], [code])
```

```java
  VALUES (?, ?);
```

现在，让我们看看另一个使用 `WHEN MATCHED THEN DELETE`（jOOQ `whenMatchedThenDelete()`）和 `WHEN NOT MATCHED THEN INSERT`（jOOQ `whenNotMatchedThenInsert()`）子句的示例：

```java
ctx.mergeInto(SALE)
```

```java
   .using(EMPLOYEE)   
```

```java
   .on(EMPLOYEE.EMPLOYEE_NUMBER.eq(SALE.EMPLOYEE_NUMBER))
```

```java
   .whenMatchedThenDelete()
```

```java
   .whenNotMatchedThenInsert(SALE.EMPLOYEE_NUMBER,   
```

```java
      SALE.FISCAL_YEAR, SALE.SALE_,
```

```java
      SALE.FISCAL_MONTH, SALE.REVENUE_GROWTH)
```

```java
   .values(EMPLOYEE.EMPLOYEE_NUMBER, val(2015),
```

```java
      coalesce(val(-1.0).mul(EMPLOYEE.COMMISSION), val(0.0)), 
```

```java
      val(1), val(0.0))
```

```java
   .execute();
```

这在 SQL Server 中工作得非常完美，但在 Oracle 中不起作用，因为 Oracle 不支持 `WHEN MATCHED THEN DELETE` 子句。但是，我们可以通过将 `WHEN MATCHED THEN UPDATE` 与 `DELETE WHERE`（通过 jOOQ 的 `thenDelete()` 获得）子句结合起来轻松地获得相同的结果。这是因为，在 Oracle 中，您可以添加 `DELETE WHERE` 子句，但只能与 `UPDATE` 一起使用：

```java
ctx.mergeInto(SALE)
```

```java
   .using(EMPLOYEE)
```

```java
   .on(EMPLOYEE.EMPLOYEE_NUMBER.eq(SALE.EMPLOYEE_NUMBER))
```

```java
// .whenMatchedThenDelete() - not supported by Oracle
```

```java
   .whenMatchedAnd(selectOne().asField().eq(1))
```

```java
   .thenDelete()
```

```java
   .whenNotMatchedThenInsert(SALE.EMPLOYEE_NUMBER,  
```

```java
      SALE.FISCAL_YEAR, SALE.SALE_,
```

```java
      SALE.FISCAL_MONTH, SALE.REVENUE_GROWTH)
```

```java
   .values(EMPLOYEE.EMPLOYEE_NUMBER, val(2015),
```

```java
      coalesce(val(-1.0).mul(EMPLOYEE.COMMISSION), val(0.0)),  
```

```java
      val(1), val(0.0))
```

```java
   .execute();
```

`WHEN MATCHED THEN UPDATE` 是通过 jOOQ 的 `whenMatchedAnd()` 获得的；这是 jOOQ 对 `WHEN MATCHED AND <some predicate> THEN` 子句的实现，但在此情况下，它被表示为 `WHEN MATCHED THEN UPDATE`。

在 SQL Server 和 Oracle 中使用 `DELETE WHERE` 子句的效果相同。使用 `DELETE WHERE` 子句的一个重要方面是它引用的是哪个表。此子句可以针对更新前或更新后的行。以下 `MERGE` 示例更新了目标表中所有在源表中具有匹配行的行。`DELETE WHERE` 子句仅删除那些由 `UPDATE` 匹配的行（这是 `UPDATE` 之后的 `DELETE`）：

```java
ctx.mergeInto(SALE)
```

```java
   .using(EMPLOYEE)
```

```java
   .on(SALE.EMPLOYEE_NUMBER.eq(EMPLOYEE.EMPLOYEE_NUMBER))
```

```java
   .whenMatchedThenUpdate()
```

```java
   .set(SALE.SALE_, coalesce(SALE.SALE_
```

```java
        .minus(EMPLOYEE.COMMISSION), SALE.SALE_))
```

```java
   .deleteWhere(SALE.SALE_.lt(1000.0))
```

```java
   .execute();
```

以下示例显示 `DELETE WHERE` 可以匹配更新之前的行的值。这次，`DELETE WHERE` 引用了 *源表*，因此状态是针对源而不是针对 `UPDATE` 的结果进行检查（这是 `UPDATE` 之前的 `DELETE`）：

```java
ctx.mergeInto(SALE)
```

```java
   .using(EMPLOYEE)
```

```java
   .on(SALE.EMPLOYEE_NUMBER.eq(EMPLOYEE.EMPLOYEE_NUMBER))
```

```java
   .whenMatchedThenUpdate()
```

```java
   .set(SALE.SALE_, coalesce(SALE.SALE_
```

```java
       .minus(EMPLOYEE.COMMISSION), SALE.SALE_))
```

```java
   .deleteWhere(EMPLOYEE.COMMISSION.lt(1000))
```

```java
   .execute();
```

在捆绑的代码中，您可以练习更多示例。应用程序命名为 *MergeSamples*。

# 摘要

本章是表达 jOOQ DSL 语法中流行的 `SELECT`、`INSERT`、`UPDATE`、`DELETE` 和 `MERGE` 语句的全面资源，依赖于基于 Java 的模式。

为了简洁，我们无法在此列出所有示例，但我强烈建议您针对您最喜欢的数据库对每个应用程序进行练习。主要目标是让您熟悉 jOOQ 语法，并能够在短时间内通过 jOOQ API 表达任何 plain SQL。

在下一章中，我们将继续这个冒险，探讨一个非常激动人心的主题：在 jOOQ 中表达 `JOIN`。
