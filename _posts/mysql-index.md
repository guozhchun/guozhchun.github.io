---
title: mysql 索引介绍
date: 2019-09-13 14:41:12
tags: mysql
categories: mysql
---

# 简介

MySQL 索引是为了帮助 MySQL 更快的查找数据而建立的一种数据结构，大部分的索引都是通过 B+ 树来实现的。由于 B+ 树内部维护着有序的数据结构，因此当查找某些具体的值时，通过索引能够快读定位到对应的行数据，从而避免顺序读取或全表扫描。通过这种方式，可以减少磁盘 IO 的次数，从而提高查询效率。

<!-- more -->

# 索引的优缺点

* 优点

  索引是结合了排序和查找两大功能的数据结构（针对 B+ 树），通过索引，可以快速提高检索效率，减少服务器需要扫描的数据量，减少了磁盘 IO。另一方面，由于索引内部是有序的结构，可以利用排好序的索引避免一些重复的排序操作，降低数据排序的成本，减低了 CPU 的消耗。

* 缺点

  由于索引内部需要维护有序的结构（针对 B+ 树），所以插入、更新或删除操作都需要同步维护索引数，这会降低这些操作的执行效率。另一方面，索引文件需要保存到磁盘中，因此如果建立的索引过大或过多，会占用比较多的磁盘空间。

# 索引分类

## 按数据结构进行分类

按索引实现的数据结构进行分类，可以将索引分为：B+ 树索引，哈希索引、R 树索引、倒排索引（反向索引）。

- B+ 树索引

  这是最常见的索引类型，基于 B 树数据结构实现的，也是 InnoDB 存储引擎默认采用的的索引类型。B+ 树是一种排好序的数据结构，因此可以用来对某个值进行精确查找或范围查找。

  在 InnoDB 存储引擎中，B+ 树索引还可以细分为聚簇索引和非聚餐索引。

  - 聚簇索引

    树的叶子节点的内容存放的是整行的数据，也就是使用聚簇索引查找到叶子节点后，可以直接拿出需要的其他列的数据。常用于主键索引。如果一张表中没有指定主键，MySQL 会自动为每一行生成一个唯一的标识符。

  - 非聚簇索引

    树的叶子节点的内容存放的是主键 ID（如果表没有设置主键，则存的是 MySQL 自动生成的类似主键的标识符）。使用这种索引进行查找，当查找到叶子节点后，如果需要查询的数据不是主键列，则需要拿到主键 ID 后回到主键索引数上再次进行查找以获取其他的数据，此时就需要两次查找，这种查询方式也称为「回表扫描」。当然，如果查找的列刚好就包含在索引中，就可以直接取出索引的值而不用回到主键树上再进行扫描了，这种也称为「覆盖索引」。

- 哈希索引

  基于哈希表实现，当用于精确查找时，其速度由于 B+ 索引树，但是由于哈希表是无序的，因此哈希索引也只能用于精确查找，不支持范围查找，也就是说如果使用哈希索引，则使用 `order by` 或 `group by` 时需要在 server 层进行排序，而无法利用索引在存储引擎中将数据准备好。

- R 树索引

  也称为空间（spatial）索引，基于 R 树实现的索引方式，常用于表示空间数据结构的类型。

- 倒排索引

  主要用于全文（FULLTEXT）索引的构建，是一种快速匹配文档的方式。

## 按创建方式进行分类

根据创建时的不同做法，可以将索引分成：单值索引和复合索引。

- 单值索引

  一个索引只包含数据表中的一个列，一个表可以存在多个不同的单值索引。如果在创建索引时加上 `unique` ，则成为唯一索引，要求索引值必须唯一不能重复，但允许有空值。

- 复合索引

  一个索引包含多个列。与单值索引相同，也可以在创建索引时加上 `unique` ，使其成为唯一索引。

# 索引基本用法

* 查看表上的索引

  使用 `show index from ${tableName}` 命令可以查看 `${tableName}` 表上有哪些索引，以及索引的类型等

* 为表的字段建索引

  创建索引有两种方式，一种是 `create` 命令，一种是 `alter` 命令

  * `create` 方式

    使用命令 `create [unique|fulltext|spatial] index ${indexName} on ${tableName} (${colName1}, ${colName2})` 可以为表 `${tableName}` 的 `${colName1}, ${colName2}` 字段建立名称为 `${indexName}` 的索引。其中 `unique` 表示建立的是唯一索引，`fulltext` 表示建立的是全文索引，`spatial` 表示建立的是空间索引。

  * `alter` 方式

    使用命令 `alter table ${tableName} add [unique|fulltext|spatial] index ${indexName} (${colName1}, ${colName2})` 可以为表 `${tableName}` 的 `${colName1}, ${colName2}` 字段建立名称为 `${indexName}` 的索引。其中 `unique` 表示建立的是唯一索引，`fulltext` 表示建立的是全文索引，`spatial` 表示建立的是空间索引。

* 删除表上的索引

  创建索引有两种方式，一种是 `drop` 命令，一种是 `alter` 命令

  * `drop` 方式

    使用 `drop index ${indexName} on ${tableName}` 命令可以将  `${tableName}` 表上的 `${indexName}` 的索引删除

  * `alter` 方式

    使用命令 `alter table ${tableName} drop index ${indexName}` 可以将  `${tableName}` 表上的 `${indexName}` 的索引删除

# 索引优化

## explain 使用

explain 是查看 MySQL 执行计划的命令，通过该命令可以查看表的执行顺序和索引的使用情况，常用于分析 SQL 语句执行效率。使用 explain 只需要在 SQL 语句前加上关键字 `explain` 即可。如下图所示

![mysql explain](/images/mysql-index-1.png)

### id

执行计划中，第一列是 id，可以是一个数字也可以是 `NULL`。如果是数字，则数字越大优先级越高（先被执行），如果是相同数字，则按顺序从上到下执行。如果是 `NULL` 表示该查询的结果来自于其他其他几个查询结果的并集。此时 `table` 列显示 `<union M,N>` 表示此结果来自于 `id` 是 `M` 和 `N` 的查询结果的并集。如下所示，表示 `NULL` 列是最后的结果集，表示其结果来自 `id` 为 `1` (tb1 表查询结果) 和 `2` （tb2 表查询结果）的融合。

![mysql explain id](/images/mysql-index-2.png)

### select_type

表明查询的类型，主要用于区别普通查询、子查询和联合查询，常见的有：SIMPLE、PRIMARY、UNION、UNION RESULT、SUBQUERY、DEPENDENT SUBQUERY、DERIVED。更多的请参考 [MySQL官方文档](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_select_type)

* SIMPLE：表明这是一个简单查询，不包含子查询或 UNION
* PRIMARY：查询中如果包含子查询或 UNION 等复杂查询，最外层的查询标记为 PRIMARY
* UNION：UNION 关键字连接的后面的查询语句
* UNION RESULT：从 UNION 结果集中获取查询结果的语句
* SUBQUERY：子查询列表中的第一个 select 语句
* DEPENDENT SUBQUERY：字查询列表中的第一个 select 语句，且使用外部的字段进行过滤操作
* DERIVED：在 FROM 列表中保护的子查询被标记为 DERIVED，MySQL 会递归执行这些子查询，把结果放在临时表中

### table

表明该查询使用的是哪张表的数据。一般情况下显示表名，有时也显示 <union**M**,**N**>、<derived**N**>、<subquery**N**>。其中 M 和 N 都表示 id 列的数字，表示这是来自于 id  为 M 或 N 的查询结果集。如 <union**M**,**N**> 表示这个查询的数据来自于 id 为 M 和 N 的查询结果的并集，<derived**N**> 表示这个查询的数据来自于 id 为 N 的查询结果集。

### partitions

表明该查询语句使用了哪个分区进行查询。如果表没有建立分区，则值是 NULL

### type

表示 MySQL 在表中找到所需行的方式，也称 “访问类型”。值有：system、const、eq_ref、ref、fulltext、ref_or_null、index_merge、unique_subquery、index_subquery、range、index、ALL。这些值从左到右的查询性能越来越差。

常见的有以下几种

* system：表中只有一条数据（基本上是系统表），是 const 的一种特殊类型。
* const：只需要通过一次查找就能获取相应的记录（最多只有一行）。一般是根据主键或唯一索引进行的查找。
* eq_ref：唯一性索引扫描，每次组合查询时，对该表只查询一次，只有一条记录与之匹配。与 const 的区别是，eq_ref 常出现在多表的关联查询中，而 const 常出现的单表的查询中。
* ref：非唯一性索引扫描，返回匹配某个单独值的所有行。一般是使用部分索引（最左前缀）进行查找，或者使用非唯一性索引进行匹配返回多行数据。
* range：使用索引进行范围匹配返回符合结果的所有行。此时 key 列表明使用了哪个索引进行范围查找。常见的范围查找的运算符有：`<>、>、>=、<、<=、is null、between、like、in`。与 ref 的区别是，ref 使用的是具体的值进行查找，而 range 使用的范围值进行查找。
* index：使用索引进行全表扫描，只扫描索引树。与 range 的区别在于 range 是使用索引进行范围查找（部分查找），index 需要遍历索引树（全部查找）
* ALL：没有使用索引进行全表扫描。

### possible_keys

指出 MySQL 能使用哪个索引在表中找到记录，查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询使用

### key

显示 MySQL 实际决定使用的索引，如果没有选择索引，值是 NULL

### key_len

表示使用的索引的长度，通过这个值可以算出有多少个索引被使用。如果 key 列是 NULL，此列的值也是 NULL

### ref

表示表的连接匹配条件，即哪些列或常量被用于查找索引列上的值

### rows

表示 MySQL 根据表统计信息及索引选用情况，估算的找到所需的记录所需要扫描的行数，这是一个估算值，并不是真正查询时扫描的行数。一般情况下，这个值越小，查询性能越好

### Extra

查询的额外信息（也很重要）。常见的有以下几个值，详情请看[MySQL官方文档](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain-extra-information)

* Using filesort：表示 MySQL 会对数据使用一次外部的排序（没有使用索引树的排序结构，也就是说，要么没有索引，要么索引的顺序和排序的顺序不对）。当出现这个值时，就需要引起注意了，如果数据量大时，查询性能会大大降低，需要调整索引进行优化。
* Using temporary：表明查询使用了临时表来存储临时的数据。常见于包含 group by 或 order by 的查询。当出现这个值时，更需要引起注意了，如果数据量大时，查询性能会大大降低，需要调整索引进行优化。
* Using index：表明使用了覆盖索引，可以直接从索引树中取出相关的数据，不需要回表扫描。当出现这个值时，一般表明这是一个高效率的查询语句。
* Using where：表明使用了 where 进行条件过滤。
* Using join buffer(Block Nested Loop)、Using join buffer(Batched Key Access)：表明使用了连接缓存，Block Nested Loop 表明使用 Block Nested-Loop 算法，Batched Key Access 表明使用了 Batched Key Access 算法。

## 优化步骤

* 在环境上单独运行 sql，看是否真的“慢”，排除其他因素（CPU、IO等）的影响。
* 执行 explain 查看执行计划，重点关注 type、key、rows、Extra 几个字段的值，看是否使用了索引，以及是否选择了正确的索引执行该查询语句。
* 参考[建索引的几个注意事项](#建索引的几个注意事项)，新建或调整索引。
* 重新运行 sql 语句观察执行结果，不符合预期则重复执行以上操作。

## 建索引的几个注意事项

* 时刻铭记并用好最左前缀匹配原则，去除多余的索引。比如同一个表有两个索引：a 和 (a,b)，则根据最左前缀匹配原则，当单独使用 a 列进行查找时，是能够用上 (a,b) 这个复合索引，此时 a 的索引就没有存在的必要，可以删除，这样也能减少维护索引的成本。

  最左前缀匹配原则：MySQL 会一直对索引列进行向右精确查找，直到遇到了范围查找后，范围查找列之后的索引列将无法使用。比如：有索引（a,b,c），如果使用条件 a = 1 and b > 2 and c = 3，则只能使用 a、b两个索引列，c 这一列是无法使用索引的。

* `%like%` 和 `%like` 无法使用索引，`like%` 可以使用索引（相当于范围查找）。

  如果一定要使用 `%like%`，并且要使用索引，可以使用覆盖索引避免其发生全表扫描。如下图所示，对 `col_2` 进行 `like %a%` 查找，没有使用覆盖索引时，走全表扫描，使用覆盖索引查询后，用上了索引。

  ![mysql index like](/images/mysql-index-3.png)

  ![mysql index like](/images/mysql-index-4.png)

  ![mysql index like](/images/mysql-index-5.png)

* 不要对索引列进行运算或隐式转换，否则索引失效无法使用，从而导致全表扫描。如存在索引 a，如果使用 `select * from tableName where concat(a, 'b') = 'ab'`，则 a 列上的索引无法使用。如果 a 字段是纯数字的字符型，则使用  `select * from tableName where a = 5` 会导致 a 字段隐式转换，也无法使用索引。

  ![mysql index implict cast](/images/mysql-index-9.png)

* 对于 `in` 条件查询，如果 `or` 连接的列上都有索引，则可以用上索引，否则只有其中部分列有索引的话，会导致索引失效。如果 `or` 连接的条件列只有一部分有索引，但是又想使用索引，可以使用 `union all` 代替 `or` 的写法。

  ![mysql index implict cast](/images/mysql-index-10.png)

  ![mysql index implict cast](/images/mysql-index-11.png)

* 选择区分度大的列建立索引。如果列的值区分度不大，则使用索引过滤掉的记录很少，效果也不大。

* 查询中经常使用到的字段以及与其他表进行关联的字段，考虑建立索引。

* 对 order by 和 group by（隐藏操作：分组前必排序） 中的字段，考虑建立索引，避免发生 `Using filesort`、`Using temporary` 操作。同时需要注意排序和查询使用相同的索引（包括排序列的升序或降序，要么全使用升序，要么全使用降序，不能部分使用升序部分使用降序），否则也容易产生  `Using filesort`。对于复合索引的情况，允许部分前缀以常量的方式出现在 where 条件中同时 order by不用以该列作为第一个排序字段。如下图所示

  ![mysql index order by](/images/mysql-index-6.png)

  ![mysql index order by](/images/mysql-index-7.png)

  ![mysql index order by](/images/mysql-index-8.png)

# 参考资料

MySQL官方文档：https://dev.mysql.com/doc/refman/8.0/en/