+++
toc = true
title = "SQL"
weight = 1
+++

由于SQL语法灵活复杂，分布式数据库和单机数据库的查询场景又不完全相同，难免有和单机数据库不兼容的SQL出现。

本文详细罗列出已明确可支持的SQL种类以及已明确不支持的SQL种类，尽量让使用者避免踩坑。

其中必然有未涉及到的SQL欢迎补充，未支持的SQL也尽量会在未来的版本中支持。

## 支持项

全面支持DQL、DML和DDL。支持分页、去重、排序、分组、聚合、关联查询（不支持跨库关联）。以下用最为复杂的DQL举例：

- SELECT主语句

```sql
SELECT select_expr [, select_expr ...] FROM table_reference [, table_reference ...]
[WHERE where_condition] 
[GROUP BY {col_name | position} [ASC | DESC]] 
[ORDER BY {col_name | position} [ASC | DESC], ...] 
[LIMIT {[offset,] row_count | row_count OFFSET offset}]
```

- select_expr

```sql
* | 
[DISTINCT] COLUMN_NAME [AS] [alias] | 
(MAX | MIN | SUM | AVG)(COLUMN_NAME | alias) [AS] [alias] | 
COUNT(* | COLUMN_NAME | alias) [AS] [alias]
```

- table_reference

```sql
tbl_name [AS] alias] [index_hint_list] | 
table_reference ([INNER] | {LEFT|RIGHT} [OUTER]) JOIN table_factor [JOIN ON conditional_expr | USING (column_list)] | 
```

## 不支持项

不支持冗余括号、CASE WHEN、HAVING、UNION (ALL)，有限支持子查询。

除了分页子查询的支持之外(详情请参考[分页](/cn/features/sharding/use-norms/pagination))，也支持同等模式的子查询。无论嵌套多少层，ShardingSphere都可以解析至第一个包含数据表的子查询，一旦在下层嵌套中再次找到包含数据表的子查询将直接抛出解析异常。

例如，以下子查询可以支持：

```sql
SELECT COUNT(*) FROM (SELECT * FROM t_order o)
```

以下子查询不支持：

```sql
SELECT COUNT(*) FROM (SELECT * FROM t_order o WHERE o.id IN (SELECT id FROM t_order WHERE status = ?))
```

简单来说，通过子查询进行非功能需求，在大部分情况下是可以支持的。比如分页、统计总数等；而通过子查询实现业务查询当前并不能支持。

由于归并的限制，子查询中包含聚合函数目前无法支持。

不支持包含schema的SQL。因为ShardingSphere的理念是像使用一个数据源一样使用多数据源，因此对SQL的访问都是在同一个逻辑schema之上。

## 示例

### 支持的SQL

| SQL                                                                                         | 必要条件                    |
| ------------------------------------------------------------------------------------------- | ---------------------------|
| SELECT * FROM tbl_name                                                                      |                            |
| SELECT * FROM tbl_name WHERE (col1 = ? or col2 = ?) and col3 = ?                            |                            |
| SELECT * FROM tbl_name WHERE col1 = ? ORDER BY col2 DESC LIMIT ?                            |                            |
| SELECT COUNT(*), SUM(col1), MIN(col1), MAX(col1), AVG(col1) FROM tbl_name WHERE col1 = ?    |                            |
| SELECT COUNT(col1) FROM tbl_name WHERE col2 = ? GROUP BY col1 ORDER BY col3 DESC LIMIT ?, ? |                            |
| INSERT INTO tbl_name (col1, col2,...) VALUES (?, ?, ....)                                   |                            |
| INSERT INTO tbl_name VALUES (?, ?,....)                                                     |                            |
| INSERT INTO tbl_name (col1, col2, ...) VALUES (?, ?, ....), (?, ?, ....)                    |                            |
| UPDATE tbl_name SET col1 = ? WHERE col2 = ?                                                 |                            |
| DELETE FROM tbl_name WHERE col1 = ?                                                         |                            |
| CREATE TABLE tbl_name (col1 int, ...)                                                       |                            |
| ALTER TABLE tbl_name ADD col1 varchar(10)                                                   |                            |
| DROP TABLE tbl_name                                                                         |                            |
| TRUNCATE TABLE tbl_name                                                                     |                            |
| CREATE INDEX idx_name ON tbl_name                                                           |                            |
| DROP INDEX idx_name ON tbl_name                                                             |                            |
| DROP INDEX idx_name                                                                         |TableRule中配置logic-index    |
| SELECT DISTINCT * FROM tbl_name WHERE col1 = ?                                              |在3.1.0支持，目前还未发布正式版本|
| SELECT COUNT(DISTINCT col1) FROM tbl_name                                                   |在3.1.0支持，目前还未发布正式版本|


### 不支持的SQL

| SQL                                                                                           | 不支持原因                                         |
| -------------------------------------------------------------------------------------         | ------------------------------ |
| INSERT INTO tbl_name (col1, col2, ...) SELECT col1, col2, ... FROM tbl_name WHERE col3 = ?    | INSERT .. SEL                  |
| INSERT INTO tbl_name SET col1 = ?                                                             | INSERT .. SET                  |
| SELECT COUNT(col1) as count_alias FROM tbl_name GROUP BY col1 HAVING count_alias > ?          | HAVING                         |
| SELECT * FROM tbl_name1 UNION SELECT * FROM tbl_name2                                         | UNION                          |
| SELECT * FROM tbl_name1 UNION ALL SELECT * FROM tbl_name2                                     | UNION ALL                      |
| SELECT * FROM tbl_name1 WHERE (val1=?) AND (val1=?)                                           | 冗余括号                        |
| SELECT * FROM ds.tbl_name1                                                                    | 包含schema                     |
| SELECT SUM(DISTINCT col1), SUM(col1) FROM tbl_name                                            | 详见`DISTINCT`支持情况详细说明    |



## `DISTINCT`支持情况详细说明（在3.1.0支持，目前还未发布正式版本）

### 支持的SQL

| SQL                                                                                         | 所需条件                             |
| ------------------------------------------------------------------------------------------- | ----------------------------------- |
| SELECT DISTINCT * FROM tbl_name WHERE col1 = ?                                              |                                     |
| SELECT DISTINCT col1 FROM tbl_name                                                          |                                     |
| SELECT DISTINCT col1, col2, col3 FROM tbl_name                                              |                                     |
| SELECT COUNT(DISTINCT col1) FROM tbl_name                                                   |                                     |
| SELECT SUM(DISTINCT col1) FROM tbl_name                                                     |                                     |
| SELECT DISTINCT col1 FROM tbl_name ORDER BY col1                                            |                                     |
| SELECT DISTINCT col1 FROM tbl_name ORDER BY col2                                            |                                     |
| SELECT COUNT(DISTINCT col1) FROM tbl_name GROUP BY col1                                     |                                     |
| SELECT COUNT(DISTINCT col1), col1 FROM tbl_name GROUP BY col1                               |                                     |

### 不支持的SQL

| SQL                                                                                         | 不支持原因                        |
| ------------------------------------------------------------------------------------------- |--------------------------------- |
| SELECT DISTINCT(col1) FROM tbl_name                                                         | DISTINCT()                       |
| SELECT COUNT(DISTINCT col1 + col2) FROM tbl_name                                            | PLUS FUNCTION + DISTINCT         |
| SELECT AVG(DISTINCT col1) FROM tbl_name                                                     | AVG(DISTINCT)                    |
| SELECT COUNT(DISTINCT col1), SUM(DISTINCT col1) FROM tbl_name                               | 同时使用2种DISTINCT的聚合函数       |
| SELECT SUM(DISTINCT col1), SUM(col1) FROM tbl_name                                          | 同时使用普通聚合函数和DISTINCT聚合函数|
| SELECT col1, COUNT(DISTINCT col1) FROM tbl_name GROUP BY col1                               | COUNT(DISTINCT)和普通查询项的顺序问题|
