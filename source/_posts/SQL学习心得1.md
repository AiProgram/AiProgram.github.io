---
title: SQL学习心得1
date: 2020-07-24 16:26:15
tags:
categories:
- 数据库
- MySQL
---

# DDL（Data Definition Languages）

**主要用于操作数据对象的定义，例如数据库、表、列、索引的定义**

<!-- more -->

## 数据库

### 使用/切换数据库

```sql
USE dataBaseName
```

### 创建数据库

```sql
CREATE DATABASE databaseName
```

### 查看数据库内所有表

```sql
SHOW TABLES
```

### 删除数据库

```sql
DROP DATABASE databaseName
```

## 表

### 创建表

+ 其中column_name 是列名，column_type是列类型，constraints是定义列的约束。

```sql
CREATE TABLE tablename (column_name_1 column_type_1 constraints，
column_name_2 column_type_2 constraints ，... column_name_n column_type_n
constraints)
```

### 查看表的详细信息

```sql
DESC tableName
```

### 删除表

```sql
DROP TABLE tableName
```

### 修改列

+ 1-4依次是修改、添加、删除表中列、修改列名的命令。
+ `FIRST`表示把列置为第一位。`AFTER`则是指定列的前一列。
+ 注意MODIFY和CHANGE是互补的，前者使用更方便只需要写一次列名但是不能修改列名，而后者可以修改列名

```sql
1. ALTER TABLE tablename MODIFY [COLUMN] column_definition [FIRST | AFTER col_name]
2. ALTER TABLE tablename ADD [COLUMN] column_definition [FIRST | AFTER col_name]
3. ALTER TABLE tablename DROP [COLUMN] col_name
4. ALTER TABLE tablename CHANGE [COLUMN] old_col_name column_definition [FIRST|AFTER col_name]
```



# DML（Data Manipulation Languages）

**用于更新、查询、删除等操作具体的数据**

# DCL（Data Control Languages）

 **用于设置数据库的各种访问权限、用户、用户组等**