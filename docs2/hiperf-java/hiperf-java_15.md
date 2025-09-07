# 15

# 优化您的数据库和 SQL 查询

数据库是大型软件系统的一个关键组件。这些系统不断使用数据库连接和查询检索和更新数据。现代系统中的数据规模极其庞大，导致需要查询、更新和显示的数据更多。这种增加的规模可能导致我们的 Java 应用程序出现性能问题，强调了确保我们的数据库和查询得到优化的重要性。

本章探讨了关键的数据库设计概念，包括数据库模式、索引策略和数据分区技术。数据库查询也被检查，特别关注**结构化查询语言**（**SQL**）查询。我们对查询优化的覆盖包括编写高效查询的最佳实践、查询执行计划和高级 SQL 技术。

本章还涵盖了高级 SQL 技术，如数据库配置、性能监控和数据库维护。本章以几个现实世界的案例研究结束，这些案例研究有助于展示如何识别和解决现有系统中的数据库相关性能问题。

在本章结束时，你应该对优化数据库和数据库查询的策略有基础的了解。有了这种理解，并利用从实际练习中获得的经验，你应该能够提高包含数据库和数据库查询的 Java 应用程序的性能。

本章涵盖了以下主要主题：

+   数据库设计

+   SQL 查询优化

+   额外的策略

+   案例研究

# 数据库设计

对于新系统，我们有在设计数据库时考虑性能的便利。我们的数据库设计可以显著影响 SQL 查询的效率。在本节中，我们将检查数据库设计的关键原则，包括模式、索引、分区和分片。

## 模式原则

数据库模式

数据库模式是数据库的设计，作为创建它的蓝图。

在我们创建数据库之前，我们应该创建一个**模式**来记录我们的数据如何组织以及它们是如何相互关联的。我们的目标是设计一个使查询数据库更高效的模式。

需要做的早期决定之一是确定我们的数据库将是**规范化**还是**非规范化**。非规范化数据库涉及减少表的数量以降低查询的复杂性。相反，规范化涉及创建单独的表以消除重复数据。让我们来看一个例子。

下表显示了作者和出版社字段中的重复数据。

| **BookID** | **Author** | **Title** | **Publisher** | **Price ($)** |
| --- | --- | --- | --- | --- |
| 1 | N. Anderson | *Zion 的介绍* | Packt | 65.99 |
| 2 | N. Anderson | *Zion 的插图历史* | Packt | 123.99 |
| 3 | W. Rabbit | *天体力学* | Packt | 89.99 |
| 4 | W. Rabbit | *Gyro Machinery* | Forest Press | 79.99 |

表 15.1 – 一个非归一化表

如您在前面的表中可以看到，有两个条目对应两位不同的作者，一个出版社被列出了不止一次。这是一个未归一化的表。为了归一化这个表，我们将创建三个表，每个表分别对应书籍、作者和出版社。

| **BookID** | **Title** | **AuthorID** | **PublisherID** | **Price ($)** |
| --- | --- | --- | --- | --- |
| 1 | *Zion* 的 *介绍* | 1 | 1 | 65.99 |
| 2 | *Zion* 的 *插图历史* | 1 | 1 | 123.99 |
| 3 | *Astro Mechanics* | 2 | 1 | 89.99 |
| 4 | *Gyro Machinery* | 2 | 2 | 79.99 |

表 15.2 – 书籍表

`Books` 表引用了 `AuthorID` 和 `PublisherID` 字段。这些字段在以下表中建立。以下是 `Authors` 表：

| **AuthorID** | **Author** |
| --- | --- |
| 1 | N. Anderson |
| 2 | W. Rabbit |

表 15.3 – 作者表

我们的最后一个表是**出版社**的表：

| **PublisherID** | **Publisher** |
| --- | --- |
| 1 | Packt |
| 2 | Forest Press |

表 15.4 – 出版社表

实施归一化或非归一化数据库的决定涉及考虑查询的复杂性、您数据库的大小以及数据库的读写负载。

另一个重要的数据库设计考虑因素是每列的数据类型。例如，使用 SQL 创建具有适当数据类型的 `Authors` 表是合适的：

```java
CREATE TABLE Authors {
  Author_ID INT,
  Author VARCHAR(80)
);
```

在我们设计好数据库表并确定数据类型后，我们需要实现索引。让我们在下一节中看看这一点。

## 索引

我们索引数据库，以便快速找到数据。我们的索引策略直接影响到我们的查询性能，因此需要尽职尽责。有两种索引类型。第一种是**平衡树**（**B-tree**），这是大多数数据库中实现的方式。这种索引保持数据排序，并允许顺序访问、插入和删除。

数据库索引的第二种类型是**哈希索引**。当需要等值比较时，这种索引类型是理想的，但它不适合范围查询。

索引创建很简单，以下 SQL 语句展示了这一点：

```java
CREATE INDEX idx_authors_authorid ON Authors (AuthorID);
```

如果您有一个现有的数据库，并且经常使用 `WHERE`、`JOIN`、`ORDER BY` 或 `GROUP BY`，您可能可以从索引中受益。让我们看看一个例子。

当我们没有索引时，我们可以使用以下 SQL 语句：

```java
SELECT * FROM Authors WHERE Author = 'N. Anderson';
```

以下 SQL 语句搜索相同的作者，但使用了索引：

```java
CREATE INDEX idx_authors_authorid ON Authors (AuthorID);
SELECT * FROM Authors WHERE Author = 'N. Anderson';
```

使用第二个例子将比非索引方法提供更快的结果。请注意，索引确实会占用额外的存储空间，并在使用 `INSERT`、`DELETE` 和 `UPDATE` 操作时增加额外的处理开销。

在下一节中，我们将检查分区和分片作为提高数据库和数据库查询效率的方法。

## 分区与分片

分区和分片是用于通过将大型数据集划分为更小的组件来提高大型数据库及其查询性能的策略。

### 分区

有两种分区类型，**水平分区**和**垂直分区**。水平分区是通过将表拆分为行来实现的，每个水平分区包含这些行的一个子集。这种方法的典型用例是基于日期范围创建分区。以下示例创建了三个表，每个表包含特定年份的订单信息：

```java
CREATE TABLE Book_Orders_2021 (
    CHECK (OrderDate >= '2021-01-01' AND OrderDate < '2022-01-01')
) INHERITS (Book_Orders);
CREATE TABLE Book_Orders_2022 (
    CHECK (OrderDate >= '2022-01-01' AND OrderDate < '2023-01-01')
) INHERITS (Book_Orders);
CREATE TABLE Book_Orders_2023 (
    CHECK (OrderDate >= '2023-01-01' AND OrderDate < '2024-01-01')
) INHERITS (Book_Orders);
```

垂直分区将数据库表拆分为列，每个分区包含列的子集。为了演示垂直分区，让我们看看一个尚未分区的`Books`表：

```java
CREATE TABLE Books (
    BookID INT PRIMARY KEY,
    Title VARCHAR(100),
    AuthorID INT,
    Pages INT,
    Genre VARCHAR(50),
    PublisherID INT,
    PublishedDate DATE,
    OriginalPrice DECIMAL(10, 2),
    DiscountPrice DECIMAL(10, 2)
);
```

现在，使用垂直分区，让我们创建两个表，每个表包含列的子集：

```java
CREATE TABLE Books_CatalogData (
    BookID INT PRIMARY KEY,
    Title VARCHAR(100),
    AuthorID INT,
    Pages INT,
    Genre VARCHAR(50),
    PublisherID INT,
    PublishedDate DATE,
);
CREATE TABLE Books_SalesData (
    BookID INT PRIMARY KEY,
    OriginalPrice DECIMAL(10, 2),
    DiscountPrice DECIMAL(10, 2),
    FOREIGN KEY (BookID) REFERENCES Books_CatalogData(BookID)
);
```

通过将我们的表拆分为两个分区，我们可以更有效地搜索，因为我们不需要处理任何无关数据（即，在搜索目录数据时，我们不关心销售数据）。

现在我们来看另一种提高数据库效率的策略，称为**分片**。

### 分片

**分片**是将我们的数据分布到多个服务器上的过程，将数据移动到用户附近。这种策略有两个主要好处——将数据移动到靠近用户的服务器可以减少网络延迟和单个服务器的负载。一个常见的用例是基于地理位置进行分片。以下是一个示例，说明我们如何实现这一点：

```java
CREATE TABLE Users_US (
    UserID INT PRIMARY KEY,
    UserName VARCHAR(100),
    Region CHAR(2) DEFAULT 'US'
);
CREATE TABLE Users_EU (
    UserID INT PRIMARY KEY,
    UserName VARCHAR(100),
    Region CHAR(2) DEFAULT 'EU'
);
```

上述示例创建了两个表。下一步是将每个表存储在不同的服务器上。

有目的的分区和分片方法可以导致一个性能就绪的数据库设计。它可以提高你的 SQL 查询效率，从而提高 Java 应用程序的整体性能。

# 查询优化

现在我们已经对如何以性能为导向设计数据库有了基本的了解，我们可以开始探讨编写高效查询的最佳实践。我们还将探讨查询执行计划和一些高级 SQL 技术。为了使本节中的示例 SQL 语句更具相关性，我们将使用一个图书库存和订单处理数据库。

## 查询执行

理解我们的数据库如何处理查询是能够对其进行优化的关键。

查询执行计划

查询执行计划提供了数据库引擎执行查询的详细信息

查询执行计划包括数据库查询操作（如连接和排序）的详细信息。让我们看看一个简单的查询，它给出了特定书籍的总销售额：

```java
FROM Orders o
JOIN Books b ON o.BookID = b.BookID
WHERE b.Title = 'High Performance with Java';
```

现在，让我们向相同的查询添加一个`EXPLAIN`命令，以揭示数据库引擎执行我们的查询的步骤：

```java
EXPLAIN SELECT SUM(o.Qty * o.Price) AS TotalSales
FROM Orders o
JOIN Books b ON o.BookID = b.BookID
WHERE b.Title = 'High Performance with Java';
```

通过查看查询执行计划，我们可以识别潜在的瓶颈，这为我们进一步优化数据库和查询提供了机会。

接下来，让我们看看编写高效查询的一些最佳实践。

## 最佳实践

编写 SQL 查询时的目标是尽量减少资源使用并减少执行时间。为了实现这些目标，我们应该遵循最佳实践，包括以下针对 `SELECT` 语句、`JOIN` 操作、**子查询**和**公用表表达式**（**CTEs**）的详细说明。

### `SELECT` 语句

使用 `SELECT` 语句有三个最佳实践。首先，我们应该避免使用 `SELECT *`，而只指定我们需要的列。例如，不要使用 `SELECT * FROM Books;`，而应使用 `SELECT Title, AuthorID, Genre FROM Books;`。

另一个最佳实践是使用 `WHERE` 子句尽可能缩小我们的结果集。以下是一个示例：

```java
SELECT Title, AuthorID, Genre FROM Books WHERE Genre = 'Non-Fiction';
```

使用 `SELECT` 语句的第三个最佳实践是限制查询返回的行数。我们可以使用 `LIMIT` 子句，如下所示：

```java
SELECT Title, AuthorID, Genre FROM Books WHERE Genre = 'Non-Fiction' LIMIT 10;
```

使用 `SELECT` 语句的三个最佳实践对于提高查询效率至关重要。接下来，让我们看看使用 `JOIN` 操作的最佳实践。

### JOIN 操作

使用 `JOIN` 操作有两个最佳实践。首先，我们应该确保在 `JOIN` 条件中使用的所有列都已建立索引。这将提高这些操作的效率。

另一个最佳实践是使用以下表格中指示的适当 `JOIN` 类型。

| **类型** | **用途** |
| --- | --- |
| `INNER JOIN` | 用于匹配行 |
| `LEFT JOIN` | 包括左表中的所有行 |
| `RIGHT JOIN` | 包括右表中的所有行 |

表 15.5：JOIN 类型及其用途

接下来，让我们探讨子查询的概念及其相关最佳实践。

### 子查询

正如标题所示，**子查询**用于将复杂查询分解成多个更简单的查询。以下是一个示例：

```java
SELECT b.Title, b.Genre
FROM Books b
WHERE b.BookID IN (SELECT o.BookID FROM Orders o WHERE o.Qty > 100);
```

接下来，让我们看看 CTEs。

### CTEs

CTEs 可以用来使复杂查询更易于阅读。这增加了它们的可重用性并简化了它们的可维护性。以下是一个示例：

```java
WITH HighSales AS (
    SELECT BookID, SUM(Qty) AS TotalQty
    FROM Orders
    GROUP BY BookID
    HAVING SUM(Qty) > 100
)
SELECT b.Title, b.Genre
FROM Books b
JOIN HighSales hs ON b.BookID = hs.BookID;
```

现在我们已经审查了编写查询的几个最佳实践，让我们看看一些高级 SQL 技术。

## 高级 SQL 技术

本节演示了三种高级 SQL 技术——窗口函数、递归查询和临时表及视图。

### 窗口函数

**窗口函数**用于计算与当前行相关的一组行的值。以下是一个示例：

```java
SELECT BookID, Title, Genre,
      SUM(Quantity) OVER (PARTITION BY Genre) AS TotalSalesByGenre
FROM Books b
JOIN Orders o ON b.BookID = o.BookID;
```

### 递归查询

递归查询是复杂的，但在处理具有层次结构的数据（如书籍类别和子类别）时非常有用。以下是一个示例：

```java
WITH RECURSIVE CategoryHierarchy AS (
    SELECT CategoryID, CategoryName, ParentCategoryID
    FROM Categories
    WHERE ParentCategoryID IS NULL
    UNION ALL
    SELECT c.CategoryID, c.CategoryName, c.ParentCategoryID
    FROM Categories c
    JOIN CategoryHierarchy ch ON c.ParentCategoryID = ch.CategoryID
)
SELECT * FROM CategoryHierarchy;
```

### 临时表和视图

另一种高级技术是使用临时表和视图来提高性能并帮助管理复杂查询。以下是一个临时表的示例：

```java
CREATE TEMPORARY TABLE TempHighSales AS
SELECT BookID, SUM(Qty) AS TotalQty
FROM Orders
GROUP BY BookID
HAVING SUM(Qty) > 100;
SELECT b.Title, b.Genre
FROM Books b
JOIN TempHighSales ths ON b.BookID = ths.BookID;
```

以下 SQL 语句是一个临时视图的示例：

```java
CREATE VIEW HighSalesView AS
SELECT BookID, SUM(Qty) AS TotalQty
FROM Orders
GROUP BY BookID
HAVING SUM(Qty) > 100;
SELECT b.Title, b.Genre
FROM Books b
JOIN HighSalesView hsv ON b.BookID = hsv.BookID;
```

在本节中尝试使用所介绍的高级技术可以提高编写高效查询的能力，从而有助于提高 Java 应用程序的整体性能。

# 额外策略

到目前为止，本章已经涵盖了为效率设计数据库模式以及如何编写高效的 SQL 查询。我们还可以采用一些其他策略，包括微调、监控和维护。本节将探讨这些策略，并使用上一节中相同的图书库存和订购示例。

## 微调

我们可以微调数据库服务器的配置参数，以确保我们的查询能够高效地使用资源。这种微调可以归类如下：

+   **数据库** **服务器参数**

+   使用 MySQL 中的`innodb_buffer_pool_size`参数和`SET shared_buffers = '3GB';` SQL 语句。

+   **连接池**：如*第十章*中详细介绍的*连接池*，我们可以将数据库连接进行池化，以减少开销并提高整体应用性能。

+   `SET query_cache_size = 256MB';`将启用查询缓存。

+   **内存管理**

+   **数据库缓存**：我们可以缓存数据库以加快对频繁访问数据的读取操作。可以使用**Redis**等工具来辅助这项技术。

接下来，让我们探讨如何监控和评估数据库的性能。

## 数据库性能监控

一旦我们的数据库启动并运行，并且所有查询都已建立，我们就可以准备监控数据库的性能。监控可以帮助我们识别潜在的瓶颈，并允许我们对性能进行优化。

识别瓶颈的一个有效方法是启用**慢查询日志**。这可以帮助我们识别哪些查询的执行时间超过了我们的期望。以下是启用方法：

```java
SET long_query_time = 1;
SET slow_query_log = 'ON';
```

使用查询分析工具可以帮助我们分析和优化慢查询。根据您的数据库类型和服务，有各种可用的工具。

监控和评估可以帮助识别优化机会。在下一节中，我们将探讨数据库维护。

## 数据库维护

数据库是动态的，需要通过定期维护来维护。这是一种主动而非被动的方法来维护数据库。以下是一些建议：

+   定期运行`VACUUM`以回收存储空间。

+   在每次运行`VACUUM`之后，运行`ANALYZE`以便查询规划器更新统计信息。

+   使用`REINDEX`定期重新索引数据库。这将提高查询性能。

+   存档不再需要的老数据。您可以将这些数据分区到历史数据库中。

+   清除不再需要的数据。这将释放存储空间并应提高查询性能。

本节中提出的策略可以帮助您进一步优化查询性能、数据库性能以及 Java 应用程序的整体性能。这些策略对于大型数据库和高交易率数据库尤为重要。

在下一节中，我们将回顾几个现实世界的案例研究，以帮助您将本章中提出的概念置于上下文中。

# 案例研究

本节展示了三个使用本章中提到的书籍库存和订单处理数据库的真实案例研究。对案例研究的回顾将展示本章中提出的策略和技术如何用于解决常见的数据库性能问题。

每个案例研究都按照以下格式展示：

+   场景

+   初始 SQL 查询

+   问题

+   优化步骤

+   结果

## 案例研究 1

**场景**：每次书店管理员运行销售报告时，都需要几分钟时间——比应有的时间要长得多。报告只是简单地按标题总结总销售额。数据库模式与本章前面展示的相同。

**初始** **SQL 查询**：

```java
SELECT b.Title, SUM(o.Qty * o.Price) AS TotalSales
FROM Orders o
JOIN Books b ON o.BookID = b.BookID
GROUP BY b.Title;
```

`Books` 和 `Orders` 表。这导致性能缓慢。

**优化步骤**：

1.  数据库管理员在 `Books` 和 `Orders` 表的 `BookID` 列上添加了索引：

    ```java
    CREATE INDEX idx_books_bookid ON Books (BookID);
    CREATE INDEX idx_orders_bookid ON Orders (BookID);
    ```

1.  查询被优化，仅包括检索所需数据所需的列：

    ```java
    EXPLAIN ANALYZE
    SELECT b.Title, SUM(o.Qty * o.Price) AS TotalSales
    FROM Orders o
    JOIN Books b ON o.BookID = b.BookID
    GROUP BY b.Title;
    ```

`EXPLAIN ANALYZE` 命令显示查询时间显著减少。销售报告现在在一分钟内完成。

## 案例研究 2

`Orders` 表现在包含数百万条记录。对特定年份的查询运行速度极慢。

**初始** **SQL 查询**：

```java
SELECT * FROM Orders WHERE OrderDate BETWEEN '2023-01-01' AND '2023-12-31';
```

**问题**：查询执行了全表扫描，导致性能缓慢。

**优化步骤**：

1.  数据库管理员执行了水平分区，为每年的数据创建表：

    ```java
    CREATE TABLE Orders_2023 (
        CHECK (OrderDate >= '2023-01-01' AND OrderDate < '2024-01-
        01')
    ) INHERITS (Orders);
    CREATE TABLE Orders_2024 (
        CHECK (OrderDate >= '2024-01-01' AND OrderDate < '2025-01-
        01')
    ) INHERITS (Orders);
    ```

1.  在数据分区后，管理员更新了查询以针对特定的分区：

    ```java
    SELECT * FROM Orders_2023 WHERE OrderDate BETWEEN '2023-01-01' AND '2023-12-31';
    ```

**结果**：查询性能显著提高，运行时间仅为之前的一小部分。

## 案例研究 3

`Books` 表导致数据库产生显著且不必要的负载。

**初始** **SQL 查询**：

```java
SELECT * FROM Books WHERE BookID = 1;
```

**问题**：数据库被反复发送相同的查询，导致高负载和缓慢的响应时间。

**优化步骤**：数据库管理员使用 **Redis** 缓存书籍详情。

**结果**：数据库负载显著减少，响应时间大幅缩短。

# 总结

本章探讨了优化数据库和 SQL 查询的必要策略和技术。本章的总体目标是介绍与数据库相关的增强和最佳实践，以提高您数据驱动应用程序的性能。我们从数据库设计的根本原则开始，包括模式规范化、适当的索引和分区策略。然后我们探讨了如何编写高效的 SQL 查询。我们的覆盖范围还包括查询执行计划和利用高级 SQL 技术，如窗口函数和递归查询。本章还讨论了包括数据库配置、监控、分析和定期维护在内的其他策略。本章以实际案例研究结束，以展示本章中涵盖的策略和技术的实际应用。现在，您应该有信心实施这些最佳实践，并确保您的数据库系统可以轻松处理大型数据集和复杂查询。

在下一章中，我们将探讨代码监控和代码维护的概念，始终保持对我们的 Java 应用程序高性能的警觉。我们的代码监控和维护方法将包括在问题出现之前进行代码审查，以识别潜在的性能问题。具体来说，我们将探讨**应用性能管理**（**APM**）工具、代码审查、日志分析和持续改进。
