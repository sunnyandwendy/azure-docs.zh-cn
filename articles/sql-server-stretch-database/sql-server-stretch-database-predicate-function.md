---
title: "使用筛选器函数来选择要迁移的行 (Stretch Database) | Microsoft 文档"
description: "了解如何使用筛选器函数来选择要迁移的行。"
services: sql-server-stretch-database
documentationcenter: 
author: douglaslMS
manager: jhubbard
editor: 
ms.assetid: f5ef79d9-68ef-4394-a057-d7aac5706b72
ms.service: sql-server-stretch-database
ms.workload: data-management
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/28/2016
ms.author: douglasl
translationtype: Human Translation
ms.sourcegitcommit: 219dcbfdca145bedb570eb9ef747ee00cc0342eb
ms.openlocfilehash: 76af756316523935cf04e19f12a3a1380d0f3a42


---
# <a name="select-rows-to-migrate-by-using-a-filter-function-stretch-database"></a>使用筛选器函数来选择要迁移的行 (Stretch Database)
如果在单独的某个表中存储了冷数据，则你可以将 Stretch Database 配置为迁移整个表。 另一方面，如果表同时包含热数据和冷数据，则可以指定筛选器函数来选择要迁移的行。 筛选器谓词是一个内联表值函数。\- 本主题描述如何编写内联表值函数以选择要迁移的行。\-

> [!NOTE]
> 如果提供的筛选器函数性能不佳，则数据迁移的性能也会不佳。 Stretch Database 使用 CROSS APPLY 运算符对表应用筛选器函数。
> 
> 

如果未指定筛选器函数，则会迁移整个表。

运行“启用数据库延伸”向导时，可以迁移整个表，也可以在向导中指定简单的筛选器函数。 如果想使用不同类型的筛选器函数来选择要迁移的行，请执行下列操作之一。

* 退出向导并运行 ALTER TABLE 语句，以便为表启用延伸，并指定筛选器函数。
* 退出向导后，运行 ALTER TABLE 语句，以便指定筛选器函数。

本主题稍后将介绍用于添加函数的 ALTER TABLE 语法。

## <a name="basic-requirements-for-the-filter-function"></a>筛选器函数的基本要求
Stretch Database 筛选器谓词所需的内联表值函数类似于以下示例。\-

```tsql
CREATE FUNCTION dbo.fn_stretchpredicate(@column1 datatype1, @column2 datatype2 [, ...n])
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN    SELECT 1 AS is_eligible
        WHERE <predicate>
```
该函数的参数必须是表中列的标识符。

需要使用架构绑定来防止删除或更改筛选器函数所使用的列。

### <a name="return-value"></a>返回值
如果该函数返回非空结果，则表示该行符合迁移条件。\- 否则（\-即，如果该函数未返回结果\-），则表示该行不符合迁移条件。

### <a name="conditions"></a>条件
&lt;*谓词*&gt; 可以包含一个条件，或使用 AND 逻辑运算符联接的多个条件。

```
<predicate> ::= <condition> [ AND <condition> ] [ ...n ]
```
而每个条件又包含一个基元条件或者使用 OR 逻辑运算符联接的多个基元条件。

```
<condition> ::= <primitive_condition> [ OR <primitive_condition> ] [ ...n ]
```

### <a name="primitive-conditions"></a>基元条件
基元条件可以执行以下比较之一。

```
<primitive_condition> ::=
{
<function_parameter> <comparison_operator> constant
| <function_parameter> { IS NULL | IS NOT NULL }
| <function_parameter> IN ( constant [ ,...n ] )
}
```

* 将函数参数与常量表达式进行比较。 例如，`@column1 < 1000`。
  
  以下示例将检查 *date* 列的值是否 &lt; 1/1/2016。
  
  ```tsql
  CREATE FUNCTION dbo.fn_stretchpredicate(@column1 datetime)
  RETURNS TABLE
  WITH SCHEMABINDING
  AS
  RETURN    SELECT 1 AS is_eligible
          WHERE @column1 < CONVERT(datetime, '1/1/2016', 101)
  GO
  
  ALTER TABLE stretch_table_name SET ( REMOTE_DATA_ARCHIVE = ON (
      FILTER_PREDICATE = dbo.fn_stretchpredicate(date),
      MIGRATION_STATE = OUTBOUND
  ) )
  ```
* 向函数参数应用 IS NULL 或 IS NOT NULL 运算符。
* 使用 IN 运算符将函数参数与常量值列表进行比较。
  
  以下示例将检查 *shipment\_status* 列的值是否为 `IN (N'Completed', N'Returned', N'Cancelled')`。
  
  ```tsql
  CREATE FUNCTION dbo.fn_stretchpredicate(@column1 nvarchar(15))
  RETURNS TABLE
  WITH SCHEMABINDING
  AS
  RETURN    SELECT 1 AS is_eligible
          WHERE @column1 IN (N'Completed', N'Returned', N'Cancelled')
  GO
  
  ALTER TABLE table1 SET ( REMOTE_DATA_ARCHIVE = ON (
      FILTER_PREDICATE = dbo.fn_stretchpredicate(shipment_status),
      MIGRATION_STATE = OUTBOUND
  ) )
  ```

### <a name="comparison-operators"></a>比较运算符
支持以下比较运算符。

`<, <=, >, >=, =, <>, !=, !<, !>`

```
<comparison_operator> ::= { < | <= | > | >= | = | <> | != | !< | !> }
```

### <a name="constant-expressions"></a>常量表达式
在筛选器函数中使用的常量可以是定义函数时可求值的任何确定性表达式。 常量表达式可以包含以下内容。

* 文本。 例如，`N’abc’, 123`。
* 代数表达式。 例如，`123 + 456`。
* 确定性函数。 例如，`SQRT(900)`。
* 使用 CAST 或 CONVERT 的确定性转换。 例如，`CONVERT(datetime, '1/1/2016', 101)`。

### <a name="other-expressions"></a>其他表达式
如果在将 BETWEEN 和 NOT BETWEEN 运算符替换为等效的 AND 和 OR 表达式后，生成的函数符合本文所述的规则，则你可以使用这些 BETWEEN 和 NOT BETWEEN 运算符。

不能使用子查询或者类似于 RAND() 或 GETDATE() 的不确定性函数。

## <a name="add-a-filter-function-to-a-table"></a>向表中添加筛选器函数
通过运行 **ALTER TABLE** 语句，并将现有的内联表值函数指定为 **FILTER\_PREDICATE** 参数的值，向表中添加筛选器函数。 例如：

```tsql
ALTER TABLE stretch_table_name SET ( REMOTE_DATA_ARCHIVE = ON (
    FILTER_PREDICATE = dbo.fn_stretchpredicate(column1, column2),
    MIGRATION_STATE = <desired_migration_state>
) )
```
将函数作为谓词绑定到表后，将发生以下情况。

* 下一次执行数据迁移时，只会迁移该函数对其返回了非空值的行。\-
* 该函数使用的列已绑定到架构。 只要表使用函数作为筛选器谓词，你就不能更改这些列。

只要表使用函数作为筛选器谓词，就不能删除内联表值函数。\-

> [!NOTE]
> 若要提高筛选器函数的性能，请在该函数使用的列上创建索引。
> 
> 

### <a name="passing-column-names-to-the-filter-function"></a>将列名传递给筛选器函数
向表分配筛选器函数时，使用一部分名称指定传递给筛选器函数的列名。 如果传递列名时指定三部分名称，则针对已启用延伸的表进行的后续查询将失败。\-

例如，如果你指定下面的示例中所示的三部分列名，该语句将成功运行，但对该表的后续查询将失败。

```tsql
ALTER TABLE SensorTelemetry
  SET ( REMOTE_DATA_ARCHIVE = ON (
    FILTER_PREDICATE=dbo.fn_stretchpredicate(dbo.SensorTelemetry.ScanDate),
    MIGRATION_STATE = OUTBOUND )
  )
```

请改为使用一部分列名指定筛选器函数，如下面的示例中所示。

```tsql
ALTER TABLE SensorTelemetry
  SET ( REMOTE_DATA_ARCHIVE = ON  (
    FILTER_PREDICATE=dbo.fn_stretchpredicate(ScanDate),
    MIGRATION_STATE = OUTBOUND )
  )
```

## <a name="a-nameaddafterwizaadd-a-filter-function-after-running-the-wizard"></a><a name="addafterwiz"></a>运行向导后添加筛选器函数
如果你想使用在“**启用数据库延伸**”向导中无法创建的函数，则可以在退出该向导后运行 ALTER TABLE 语句来指定函数。 但是，在应用函数之前，你必须停止已在进行的数据迁移并取回已迁移的数据。 （有关必须这样做的原因的详细信息，请参阅[替换现有的筛选器函数](#replacePredicate)）。  

1. 反向迁移，并取回已迁移的数据。 启动此操作后将无法取消此操作。 并且，针对出站数据传输（\(传出\)）还会在 Azure 上产生费用。 有关详细信息，请参阅 [Azure 如何定价](https://azure.microsoft.com/pricing/details/data-transfers/)。  
   
    ```tsql  
    ALTER TABLE <table name>  
         SET ( REMOTE_DATA_ARCHIVE ( MIGRATION_STATE = INBOUND ) ) ;   
    ```  
2. 等待迁移完成。 你可以从 SQL Server Management Studio 检查 **Stretch Database 监视器**的状态，或者可以查询 **sys.dm_db_rda_migration_status** 视图。 有关详细信息，请参阅[数据迁移的监视和故障排除](sql-server-stretch-database-monitor.md)或 [sys.dm_db_rda_migration_status](https://msdn.microsoft.com/library/dn935017.aspx)。  
3. 创建要应用于表的筛选器函数。  
4. 将该函数添加到表，并重新启动到 Azure 的数据迁移。  
   
    ```tsql  
    ALTER TABLE <table name>  
        SET ( REMOTE_DATA_ARCHIVE  
            (           
                FILTER_PREDICATE = <predicate>,  
                MIGRATION_STATE = OUTBOUND  
            )  
        );   
    ```  

## <a name="filter-rows-by-date"></a>按日期筛选行
下面的示例将迁移**日期**列中包含值早于 2016 年 1 月 1 日的行。

```tsql
-- Filter by date
--
CREATE FUNCTION dbo.fn_stretch_by_date(@date datetime2)
RETURNS TABLE
WITH SCHEMABINDING
AS
       RETURN SELECT 1 AS is_eligible WHERE @date < CONVERT(datetime2, '1/1/2016', 101)
GO
```

## <a name="filter-rows-by-the-value-in-a-status-column"></a>按状态列中的值筛选行
下面的示例将迁移**状态**列中包含指定值之一的行。

```tsql
-- Filter by status column
--
CREATE FUNCTION dbo.fn_stretch_by_status(@status nvarchar(128))
RETURNS TABLE
WITH SCHEMABINDING
AS
       RETURN SELECT 1 AS is_eligible WHERE @status IN (N'Completed', N'Returned', N'Cancelled')
GO
```

## <a name="filter-rows-by-using-a-sliding-window"></a>使用滑动窗口筛选行
若要使用滑动窗口筛选行，请记住筛选器函数的下列要求。

* 该函数必须是确定性函数。 因此，不能创建随着时间的推移自动重新计算滑动窗口的函数。
* 该函数使用架构绑定。 因此，你不能只通过调用 **ALTER FUNCTION** 移动滑动窗口来每天“就地”更新该函数。

如以下示例，先使用筛选器函数，迁移 **systemEndTime** 列中包含的值早于 2016 年 1 月 1 日的行。

```tsql
CREATE FUNCTION dbo.fn_StretchBySystemEndTime20160101(@systemEndTime datetime2)
RETURNS TABLE
WITH SCHEMABINDING  
AS  
RETURN SELECT 1 AS is_eligible
  WHERE @systemEndTime < CONVERT(datetime2, '2016-01-01T00:00:00', 101) ;
```

对表应用筛选器函数。

```tsql
ALTER TABLE <table name>
SET (
        REMOTE_DATA_ARCHIVE = ON
                (
                        FILTER_PREDICATE = dbo.fn_StretchBySystemEndTime20160101 (SysEndTime)
                                , MIGRATION_STATE = OUTBOUND
                )
        )
;
```

如果要更新滑动窗口，请执行以下操作。

1. 创建一个新函数，以指定新的滑动窗口。 下面的示例选择的日期早于 2016 年 1 月 2 日，而不是 2016 年 1 月 1 日。
2. 通过调用 **ALTER TABLE** 将以前的筛选器函数替换为新的筛选器函数，如下面的示例中所示。
3. 或者，通过调用 **DROP FUNCTION** 删除不再使用的前一个筛选器函数。 （本示例中未说明此步骤。）

```tsql
BEGIN TRAN
GO
        /*(1) Create new predicate function definition */
        CREATE FUNCTION dbo.fn_StretchBySystemEndTime20160102(@systemEndTime datetime2)
        RETURNS TABLE
        WITH SCHEMABINDING
        AS
        RETURN SELECT 1 AS is_eligible
               WHERE @systemEndTime < CONVERT(datetime2,'2016-01-02T00:00:00', 101)
        GO

        /*(2) Set the new function as the filter predicate */
        ALTER TABLE <table name>
        SET
        (
               REMOTE_DATA_ARCHIVE = ON
               (
                       FILTER_PREDICATE = dbo.fn_StretchBySystemEndTime20160102(SysEndTime),
                       MIGRATION_STATE = OUTBOUND
               )
        )
COMMIT ;
```

## <a name="more-examples-of-valid-filter-functions"></a>有效的筛选器函数的更多示例
* 以下示例使用 AND 逻辑运算符组合两个基元条件。
  
  ```tsql
  CREATE FUNCTION dbo.fn_stretchpredicate((@column1 datetime, @column2 nvarchar(15))
  RETURNS TABLE
  WITH SCHEMABINDING
  AS
  RETURN    SELECT 1 AS is_eligible
    WHERE @column1 < N'20150101' AND @column2 IN (N'Completed', N'Returned', N'Cancelled')
  GO
  
  ALTER TABLE table1 SET ( REMOTE_DATA_ARCHIVE = ON (
      FILTER_PREDICATE = dbo.fn_stretchpredicate(date, shipment_status),
      MIGRATION_STATE = OUTBOUND
  ) )
  ```
* 以下示例使用了多个条件，并使用 CONVERT 执行确定性转换。
  
  ```tsql
  CREATE FUNCTION dbo.fn_stretchpredicate_example1(@column1 datetime, @column2 int, @column3 nvarchar)
  RETURNS TABLE
  WITH SCHEMABINDING
  AS
  RETURN    SELECT 1 AS is_eligible
      WHERE @column1 < CONVERT(datetime, '1/1/2015', 101)AND (@column2 < -100 OR @column2 > 100 OR @column2 IS NULL)AND @column3 IN (N'Completed', N'Returned', N'Cancelled')
  GO
  ```
* 以下示例使用数学运算符和函数。
  
  ```tsql
  CREATE FUNCTION dbo.fn_stretchpredicate_example2(@column1 float)
  RETURNS TABLE
  WITH SCHEMABINDING
  AS
  RETURN    SELECT 1 AS is_eligible
          WHERE @column1 < SQRT(400) + 10
  GO
  ```
* 以下示例使用 BETWEEN 和 NOT BETWEEN 运算符。 这种用法是有效的，因为在将 BETWEEN 和 NOT BETWEEN 运算符替换为等效的 AND 和 OR 表达式之后，生成的函数符合此处所述的规则。
  
  ```tsql
  CREATE FUNCTION dbo.fn_stretchpredicate_example3(@column1 int, @column2 int)
  RETURNS TABLE
  WITH SCHEMABINDING
  AS
  RETURN    SELECT 1 AS is_eligible
          WHERE @column1 BETWEEN 0 AND 100
              AND (@column2 NOT BETWEEN 200 AND 300 OR @column1 = 50)
  GO
  ```
  在将 BETWEEN 和 NOT BETWEEN 运算符替换为等效的 AND 和 OR 表达式后，上面的函数等效于下面的函数。
  
  ```tsql
  CREATE FUNCTION dbo.fn_stretchpredicate_example4(@column1 int, @column2 int)
  RETURNS TABLE
  WITH SCHEMABINDING
  AS
  RETURN    SELECT 1 AS is_eligible
          WHERE @column1 >= 0 AND @column1 <= 100AND (@column2 < 200 OR @column2 > 300 OR @column1 = 50)
  GO
  ```

## <a name="examples-of-filter-functions-that-arent-valid"></a>无效的筛选器函数的示例
* 下面的函数无效，因为它包含非确定性转换。\-
  
  ```tsql
  CREATE FUNCTION dbo.fn_example5(@column1 datetime)
  RETURNS TABLE
  WITH SCHEMABINDING
  AS
  RETURN    SELECT 1 AS is_eligible
          WHERE @column1 < CONVERT(datetime, '1/1/2016')
  GO
  ```
* 下面的函数无效，因为它包含非确定性函数调用。\-
  
  ```tsql
  CREATE FUNCTION dbo.fn_example6(@column1 datetime)
  RETURNS TABLE
  WITH SCHEMABINDING
  AS
  RETURN    SELECT 1 AS is_eligible
          WHERE @column1 < DATEADD(day, -60, GETDATE())
  GO
  ```
* 以下函数无效，因为它包含子查询。
  
  ```tsql
  CREATE FUNCTION dbo.fn_example7(@column1 int)
  RETURNS TABLE
  WITH SCHEMABINDING
  AS
  RETURN    SELECT 1 AS is_eligible
          WHERE @column1 IN (SELECT SupplierID FROM Supplier WHERE Status = 'Defunct'))
  GO
  ```
* 下面的函数无效，因为当定义函数时使用代数运算符或内置函数的表达式的计算结果必须为常量。\- 不能在代数表达式或函数调用中包含列引用。
  
  ```tsql
  CREATE FUNCTION dbo.fn_example8(@column1 int)
  RETURNS TABLE
  WITH SCHEMABINDING
  AS
  RETURN    SELECT 1 AS is_eligible
          WHERE @column1 % 2 =  0
  GO
  
  CREATE FUNCTION dbo.fn_example9(@column1 int)
  RETURNS TABLE
  WITH SCHEMABINDING
  AS
  RETURN    SELECT 1 AS is_eligible
          WHERE SQRT(@column1) = 30
  GO
  ```
* 下面的函数无效，因为在将 BETWEEN 运算符替换为等效的 AND 表达式之后，它违反了此处所述的规则。
  
  ```tsql
  CREATE FUNCTION dbo.fn_example10(@column1 int, @column2 int)
  RETURNS TABLE
  WITH SCHEMABINDING
  AS
  RETURN    SELECT 1 AS is_eligible
          WHERE (@column1 BETWEEN 1 AND 200 OR @column1 = 300) AND @column2 > 1000
  GO
  ```
  在将 BETWEEN 运算符替换为等效的 AND 表达式后，上面的函数等效于下面的函数。 此函数无效，因为基元条件只能使用 OR 逻辑运算符。
  
  ```tsql
  CREATE FUNCTION dbo.fn_example11(@column1 int, @column2 int)
  RETURNS TABLE
  WITH SCHEMABINDING
  AS
  RETURN    SELECT 1 AS is_eligible
          WHERE (@column1 >= 1 AND @column1 <= 200 OR @column1 = 300) AND @column2 > 1000
  GO
  ```

## <a name="how-stretch-database-applies-the-filter-function"></a>Stretch Database 如何应用筛选器函数
Stretch Database 使用 CROSS APPLY 运算符对表应用筛选器函数并确定符合条件的行。 例如：

```tsql
SELECT * FROM stretch_table_name CROSS APPLY fn_stretchpredicate(column1, column2)
```
如果该函数返回行的非空结果，则行符合迁移条件。\-

## <a name="a-namereplacepredicateareplace-an-existing-filter-function"></a><a name="replacePredicate"></a>替换现有的筛选器函数
你可以通过再次运行 **ALTER TABLE** 语句并为 **FILTER\_PREDICATE** 参数指定新值来替换以前指定的筛选器函数。 例如：

```tsql
ALTER TABLE stretch_table_name SET ( REMOTE_DATA_ARCHIVE = ON (
    FILTER_PREDICATE = dbo.fn_stretchpredicate2(column1, column2),
    MIGRATION_STATE = <desired_migration_state>
```
新的内联表值函数具有以下要求。\-

* 新函数的限制强度必须低于以前的函数。
* 旧函数中的所有运算符必须存在于新函数中。
* 新函数不能包含旧函数中不存在的运算符。
* 运算符参数的顺序不能更改。
* 只有属于 `<, <=, >, >=` 比较的常量值才可以使用使函数限制更少的方式来进行更改。

### <a name="example-of-a-valid-replacement"></a>有效替换的示例
假设以下函数是当前的筛选器谓词。

```tsql
CREATE FUNCTION dbo.fn_stretchpredicate_old (@column1 datetime, @column2 int)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN    SELECT 1 AS is_eligible
        WHERE @column1 < CONVERT(datetime, '1/1/2016', 101)
            AND (@column2 < -100 OR @column2 > 100)
GO
```
下面的函数是有效的替换，因为新的日期常量（它指定更晚的截止日期）使函数限制更少。

```tsql
CREATE FUNCTION dbo.fn_stretchpredicate_new (@column1 datetime, @column2 int)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN    SELECT 1 AS is_eligible
        WHERE @column1 < CONVERT(datetime, '2/1/2016', 101)
            AND (@column2 < -50 OR @column2 > 50)
GO
```

### <a name="examples-of-replacements-that-arent-valid"></a>无效替换的示例
下面的函数不是有效的替换，因为新的日期常量（它指定更早的截止日期）不使函数限制更少。

```tsql
CREATE FUNCTION dbo.fn_notvalidreplacement_1 (@column1 datetime, @column2 int)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN    SELECT 1 AS is_eligible
        WHERE @column1 < CONVERT(datetime, '1/1/2015', 101)
            AND (@column2 < -100 OR @column2 > 100)
GO
```
以下函数不是有效的替换，因为删除了某个比较运算符。

```tsql
CREATE FUNCTION dbo.fn_notvalidreplacement_2 (@column1 datetime, @column2 int)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN    SELECT 1 AS is_eligible
        WHERE @column1 < CONVERT(datetime, '1/1/2016', 101)
            AND (@column2 < -50)
GO
```
以下函数不是有效的替换，因为使用 AND 逻辑运算符添加了新的条件。

```tsql
CREATE FUNCTION dbo.fn_notvalidreplacement_3 (@column1 datetime, @column2 int)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN    SELECT 1 AS is_eligible
        WHERE @column1 < CONVERT(datetime, '1/1/2016', 101)
            AND (@column2 < -100 OR @column2 > 100)
            AND (@column2 <> 0)
GO
```

## <a name="remove-a-filter-function-from-a-table"></a>从表中删除筛选器函数
若要迁移整个表而不是所选的行，可通过将 **FILTER\_PREDICATE** 设置为 null 删除现有的函数。 例如：

```tsql
ALTER TABLE stretch_table_name SET ( REMOTE_DATA_ARCHIVE = ON (
    FILTER_PREDICATE = NULL,
    MIGRATION_STATE = <desired_migration_state>
) )
```
删除筛选器函数后，表中的所有行都符合迁移条件。 因此，稍后不能为同一个表指定筛选器函数，除非你先从 Azure 取回该表的所有远程数据。 存在此限制是为了避免这样的情况，即当你提供新的筛选器函数时不符合迁移条件的行已迁移到 Azure。

## <a name="check-the-filter-function-applied-to-a-table"></a>检查应用于表的筛选器函数
若要检查应用于表的筛选器函数，可打开目录视图 **sys.remote\_data\_archive\_tables**，并检查 **filter\_predicate** 列的值。 如果值为 null，则整个表符合存档条件。 有关详细信息，请参阅 [sys.remote_data_archive_tables (Transact SQL)](https://msdn.microsoft.com/library/dn935003.aspx)。

## <a name="security-notes-for-filter-functions"></a>筛选器函数的安全注意事项
具有 db_owner 特权的盗用帐户可以执行以下操作。  

* 创建并应用这样的表值函数：占用大量的服务器资源，或者等待较长时间从而导致拒绝服务。  
* 创建并应用这样的表值函数：可以推导出该用户已被显式拒绝读取访问的表的内容。  

## <a name="see-also"></a>另请参阅
[ALTER TABLE (Transact-SQL)](https://msdn.microsoft.com/library/ms190273.aspx)




<!--HONumber=Nov16_HO3-->

