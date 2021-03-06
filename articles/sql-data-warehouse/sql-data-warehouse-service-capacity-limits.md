---
title: "SQL 数据仓库容量限制 | Microsoft 文档"
description: "SQL 数据仓库的连接、数据库、表和查询的最大值。"
services: sql-data-warehouse
documentationcenter: NA
author: kevinvngo
manager: jhubbard
editor: 
ms.assetid: e1eac122-baee-4200-a2ed-f38bfa0f67ce
ms.service: sql-data-warehouse
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: data-services
ms.custom: reference
ms.date: 12/14/2017
ms.author: kevin;barbkess
ms.openlocfilehash: 3a8edb3806f981ebb6f8c1ca6c994ae198df2ec2
ms.sourcegitcommit: 821b6306aab244d2feacbd722f60d99881e9d2a4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/16/2017
---
# <a name="sql-data-warehouse-capacity-limits"></a>SQL 数据仓库容量限制
下表包含 Azure SQL 数据仓库的各个组件允许的最大值。

## <a name="workload-management"></a>工作负荷管理
| 类别 | 说明 | 最大值 |
|:--- |:--- |:--- |
| [数据仓库单位 (DWU)][Data Warehouse Units (DWU)] |单个 SQL 数据仓库的最大 DWU | 弹性优化[性能层](performance-tiers.md)：DW6000<br></br>计算优化[性能层](performance-tiers.md)：DW30000c |
| [数据仓库单位 (DWU)][Data Warehouse Units (DWU)] |每个服务器的默认 DTU |54,000<br></br>默认情况下，每个 SQL Server（例如 myserver.database.windows.net）的 DTU 配额为 54,000，最多可以允许 DW6000c。 此配额仅仅只是安全限制。 可以通过[创建支持票证][creating a support ticket]并选择“配额”作为请求类型来增加配额。  要计算 DTU 需求，请将所需的 DWU 总数乘以 7.5 或将所需的 cDWU 总数乘以 9.0。 例如：<br></br>DW6000 x 7.5 = 45,000 DTU<br></br>DW600c x 9.0 = 54,000 DTU。<br></br>可以在门户中的 SQL Server 选项中查看当前 DTU 消耗量。 已暂停和未暂停的数据库都计入 DTU 配额。 |
| 数据库连接 |并发打开的会话 |1024<br/><br/>1024 个活动会话每一个都能同时向 SQL 数据仓库数据库提交请求。 请注意，可并发执行的查询数量是有限制的。 当超出并发限制时，请求将进入内部队列等待处理。 |
| 数据库连接 |预处理语句的最大内存 |20 MB |
| [工作负荷管理][Workload management] |并发查询数上限 |32<br/><br/> 默认情况下，SQL 数据仓库可以执行最多 32 个并发查询并将剩余查询排列起来。<br/><br/>当用户被分配到较高资源类或者 SQL 数据仓库的[服务级别](performance-tiers.md#service-levels)较低时，可减少并发查询的数量。 某些查询（如 DMV 查询）始终可以运行。 |
| [tempdb][Tempdb] |最大 GB |每 DW100 399 GB。 因此，在 DWU1000 的情况下，tempdb 的大小为 3.99 TB |

## <a name="database-objects"></a>数据库对象
| 类别 | 说明 | 最大值 |
|:--- |:--- |:--- |
| 数据库 |最大大小 |磁盘上压缩后 240 TB<br/><br/>此空间与 tempdb 或日志空间无关，因此，此空间专用于永久表。  聚集列存储压缩率估计为 5 倍。  此压缩率允许数据库在所有表都为聚集列存储（默认表类型）的情况下增长到大约 1 PB。 |
| 表 |最大大小 |磁盘上压缩后 60 TB |
| 表 |每个数据库的表数 |20 亿 |
| 表 |每个表的列数 |1024 个列 |
| 表 |每个列的字节数 |取决于列[数据类型][data type]。  char 数据类型的限制为 8000，nvarchar 数据类型的限制为 4000，MAX 数据类型的限制为 2 GB。 |
| 表 |每行的字节数，定义的大小 |8060 字节<br/><br/>每行字节数的计算方式同于使用页面压缩的 SQL Server。 与 SQL Server 一样，SQL 数据仓库支持行溢出存储，使可变长度列能够脱行推送。 对可变长度行进行拖行推送时，只将 24 字节的根存储在主记录中。 有关详细信息，请参阅：[超过 8-KB 的行溢出数据][Row-Overflow Data Exceeding 8 KB]。 |
| 表 |每个表的分区数 |15,000<br/><br/>为了实现高性能，建议在满足业务需求的情况下尽量减少所需的分区数。 随着分区数目的增长，数据定义语言 (DDL) 和数据操作语言 (DML) 操作的开销也会增长，导致性能下降。 |
| 表 |每个分区边界值的字符数。 |4000 |
| 索引 |每个表的非聚集索引数。 |999<br/><br/>仅适用于行存储表。 |
| 索引 |每个表的聚集索引数。 |1<br><br/>适用于行存储表和列存储表。 |
| 索引 |索引键大小。 |900 字节。<br/><br/>仅适用于行存储索引。<br/><br/>如果创建索引时列中的现有数据未超过 900 字节，那么可以创建最大大小超过 900 字节的 varchar 列上的索引。 但是，以后导致总大小超过 900 字节的对列的 INSERT 或 UPDATE 操作会失败。 |
| 索引 |每个索引的键列数。 |16<br/><br/>仅适用于行存储索引。 聚集列存储索引包括所有列。 |
| 统计信息 |组合的列值的大小。 |900 字节。 |
| 统计信息 |每个统计对象的列数。 |32 |
| 统计信息 |每个表的列上创建的统计信息条数。 |30,000 |
| 存储过程 |最大嵌套级数。 |8 |
| 查看 |每个视图的列数 |1,024 |

## <a name="loads"></a>加载
| 类别 | 说明 | 最大值 |
|:--- |:--- |:--- |
| Polybase 加载 |每行 MB 数 |1<br/><br/>Polybase 仅加载到小于 1 MB 的行，并且无法加载到 VARCHAR(MAX)、NVARCHAR(MAX) 或 VARBINARY(MAX)。<br/><br/> |

## <a name="queries"></a>查询
| 类别 | 说明 | 最大值 |
|:--- |:--- |:--- |
| 查询 |用户表的排队查询数。 |1000 |
| 查询 |系统视图的并发查询数。 |100 |
| 查询 |系统视图的排队查询数。 |1000 |
| 查询 |最大值参数 |2098 |
| Batch |最大大小 |65,536\*4096 |
| SELECT 结果 |每个行的列数 |4096<br/><br/>在 SELECT 结果中每行的列数始终不得超过 4096。 无法保证最大值始终为 4096。 如果查询计划需要一个临时表，那么将应用每个表最多 1024 列的最大值。 |
| SELECT |嵌套子查询 |32<br/><br/>在 SELECT 语句中的嵌套子查询数始终不得超过 32 个。 无法保证最大值始终为 32 个。 例如，JOIN 可以将子查询引入查询计划。 还可以通过可用内存来限制子查询的数量。 |
| SELECT |每个 JOIN 的列数 |1024 个列<br/><br/>JOIN 中的列数始终不得超过 1024。 无法保证最大值始终为 1024。 如果 JOIN 计划需要列数多于 JOIN 结果的临时表，那么将 1024 限制应用于此临时表。 |
| SELECT |每个 GROUP BY 列的字节数。 |8060<br/><br/>GROUP BY 子句中的列的字节数最大为 8060 字节。 |
| SELECT |每个 ORDER BY 列的字节数 |8060 字节。<br/><br/>ORDER BY 子句中的列的字节数最大为 8060 字节。 |
| 每个语句的标识符和常量数 |被引用的标识符和常量的数量。 |65,535<br/><br/>SQL 数据仓库限制一条查询的单个表达式中可包含的标识符和常量数。 此限制为 65,535。 超过此数字将导致 SQL Server 错误 8632。 有关详细信息，请参阅 [Internal error: An expression services limit has been reached][Internal error: An expression services limit has been reached]（内部错误：已达到表达式服务限制）。 |

## <a name="metadata"></a>元数据
| 系统视图 | 最大行数 |
|:--- |:--- |
| sys.dm_pdw_component_health_alerts |10,000 |
| sys.dm_pdw_dms_cores |100 |
| sys.dm_pdw_dms_workers |最近 1000 个 SQL 请求的 DMS 辅助角色的总数。 |
| sys.dm_pdw_errors |10,000 |
| sys.dm_pdw_exec_requests |10,000 |
| sys.dm_pdw_exec_sessions |10,000 |
| sys.dm_pdw_request_steps |sys.dm_pdw_exec_requests 中存储的最近 1000 个 SQL 请求的步骤总数。 |
| sys.dm_pdw_os_event_logs |10,000 |
| sys.dm_pdw_sql_requests |sys.dm_pdw_exec_requests 中存储的最近 1000 个 SQL 请求。 |

## <a name="next-steps"></a>后续步骤
有关更多参考信息，请参阅 [SQL 数据仓库参考概述][SQL Data Warehouse reference overview]。

<!--Image references-->

<!--Article references-->
[Data Warehouse Units (DWU)]: ./sql-data-warehouse-overview-what-is.md
[SQL Data Warehouse reference overview]: ./sql-data-warehouse-overview-reference.md
[Workload management]: ./sql-data-warehouse-develop-concurrency.md
[Tempdb]: ./sql-data-warehouse-tables-temporary.md
[data type]: ./sql-data-warehouse-tables-data-types.md
[creating a support ticket]: /sql-data-warehouse-get-started-create-support-ticket.md

<!--MSDN references-->
[Row-Overflow Data Exceeding 8 KB]: https://msdn.microsoft.com/library/ms186981.aspx
[Internal error: An expression services limit has been reached]: https://support.microsoft.com/kb/913050
