---
title: SQL学习心得1
date: 2020-07-24 16:26:15
tags:
categories:
- 数据库
- MySQL
---

**本文章总结了SQL中最常用、最基础的语法。SQL的其他细节知识例如数据类型、操作符等不在此总结**

# DDL（Data Definition Languages）

**主要用于操作数据对象的定义，例如数据库、表、列、索引的定义**

<!-- more -->

## 数据库

### 使用/切换数据库

```sql
USE dataBaseName;
```

### 创建数据库

```sql
CREATE DATABASE databaseName;
```

### 查看数据库内所有表

```;sql
SHOW TABLES;
```

### 删除数据库

```sql
DROP DATABASE databaseName;
```

## 表

### 创建表

+ 其中column_name 是列名，column_type是列类型，constraints是定义列的约束。

```sql
CREATE TABLE tablename (column_name_1 column_type_1 constraints，
column_name_2 column_type_2 constraints ，... column_name_n column_type_n
constraints);
```

### 查看表的详细信息

```sql
DESC tableName;
```

### 删除表

```sql
DROP TABLE tableName;
```

### 修改列

+ 1-4依次是修改、添加、删除表中列、修改列名的命令。
+ `FIRST`表示把列置为第一位。`AFTER`则是指定列的前一列。
+ 注意MODIFY和CHANGE是互补的，前者使用更方便只需要写一次列名但是不能修改列名，而后者可以修改列名

```sql
1. ALTER TABLE tablename MODIFY [COLUMN] column_definition [FIRST | AFTER col_name];
2. ALTER TABLE tablename ADD [COLUMN] column_definition [FIRST | AFTER col_name];
3. ALTER TABLE tablename DROP [COLUMN] col_name;
4. ALTER TABLE tablename CHANGE [COLUMN] old_col_name column_definition [FIRST|AFTER col_name];
```

### 修改表名

```sql
ALTER TABLE tablename RENAME [TO] new_tablename;
```

# DML（Data Manipulation Languages）

**用于更新、查询、删除等操作具体的数据**

与DDL对应的操作不同。

+ DDL的删除：DROP，修改：ALTER，MODIFY，等，增加(插入)：ADD，查询：SHOW。
+ 而在DML中，删除：DELETE，FROM，修改：UPDATE，SET，增加：INSERT，查询：SELECT，FROM，WHERE。

## 插入行

+ 单行插入记录
  + 注意：filed可以不写但是不写就必须让values中的值与表的列一一对应。

```sql
INSERT INTO tablename (field1,field2,……fieldn) VALUES(value1,value2,……valuesn);
```

+ 多行一次性插入记录

```sql
INSERT INTO tablename (field1, field2,……fieldn)
VALUES
(record1_value1, record1_value2,……record1_valuesn),
(record2_value1, record2_value2,……record2_valuesn),
……
(recordn_value1, recordn_value2,……recordn_valuesn)
;
```

## 更新多表的记录

+ $t_1,t_2$等是表名，显然我们也可以把本命令改为只改一个表的更新命令，那时无需指定属性列对应的表名了。

```sql
UPDATE t1,t2…tn set t1.field1=expr1,tn.fieldn=exprn [WHERE CONDITION]
```

## 删除记录

```sql
DELETE FROM tablename [WHERE CONDITION]
```

## 查询记录

这是DML最重要、最复杂的内容了，此处仅介绍基础必备语法。

+ ### 基础SELECT语法

  + WHERE条件可以非常复杂。

```sql
SELECT * FROM tablename [WHERE CONDITION]
SELECT colName1, colName2,... FROM tableName [WHERE CONDITION]
```

+ ### 排序SELECT语法

  + 不指定DESC：降序或者ASC：升序时，默认升序
  + 可有多个排序关键字。例如依次为A，B，则仅当A处相等时再按B排序。
  + 每个关键字均可执行升降序
  + 若有记录使用了所有关键字仍无规定顺序，最后结果是无序的。

```sql
SELECT * FROM tablename [WHERE CONDITION] [ORDER BY field1 [DESC|ASC] ， field2
[DESC|ASC]，……fieldn [DESC|ASC]]
```

+ ### 限制显示数量SELECT语法

  + offset_start是显示起点，row_count是显示条数
  + limit属于MySQL拓展语法，其他数据库可能无法使用

```sql
SELECT ……[LIMIT offset_start,row_count]
```

+ ### 聚合查询

  + 聚合指：我们查询中用到的可能是多条记录，但是最后结果却只有一条。
  + 聚合函数例如SUM，COUNT，MIN，MAX
  + having 和where 的区别在于having 是对聚合后的结果进行条件的过滤，例如筛选出COUNT结果符合要求的。而where 是在聚合前就对记录进行过滤，如果逻辑允许，我们尽可能用where 先过滤记录。
  + 有了GroupBy以后我们的聚类结果是分散的，WITH ROLLUP指最后再全部聚类查询一次，例如查询总人数而不仅是各部门总人数。

```sql
SELECT [field1,field2,……fieldn] fun_name
FROM tablename
[WHERE where_contition]
[GROUP BY field1,field2,……fieldn
[WITH ROLLUP]]
[HAVING where_contition]
```

+ ### 连接查询

  + 连接查询包括了常提到的内、左、右、全连接查询
  + 如果我们使用逻辑运算来表示这几种连接查询，假设连接查询$t_1,t_2$那么有：内连接：$t_1\bigcap t_2$，左连接：$t_1\bigcup(t_1\bigcap t_1)$，右连接：$t_2\bigcup(t_1\bigcap t_2)$，全连接：$t_1\bigcup t_2$。

  内连接(示例)：

  ```sql
  select ename,deptname from emp,dept where emp.deptno=dept.deptno;
  ```

  左连接示例：

  ```sql
  select ename,deptname from emp left join dept on emp.deptno=dept.deptno;
  ```

+ ### 子SELECT查询

  + 也即使用一个SELECT子句的结果作为父级SELECT的输入
  + 子查询的关键字不仅有`in`，还有not in、=、!=、exists、not exists等等。

```sql
select * from emp where deptno in (select deptno from dept);
```

+ ### 多个SELECT结果组合在一起

  + 这里的组合不修改行内部的内容，仅仅只是把结果行组合到一个结果中。
  + UNION会对结果去重，相当于使用了DISTINCT，但是UNIONALL则不管重复全部组合。

```sql
SELECT * FROM t1
UNION|UNION ALL
SELECT * FROM t2
……
UNION|UNION ALL
SELECT * FROM tn;
```



# DCL（Data Control Languages）

 **用于设置数据库的各种访问权限、用户、用户组等**