---
title: "MSSQL temporary table"
date: 2021-06-24T08:06:25+06:00
description: "introduction of basic use for MSSQL temorary table"
menu:
  sidebar:
    name: MSSQL temporary table
    identifier: MSSQL-directory
    parent : SQL-directory
    weight: 10
hero: images/sql.jpg
---

Temporary tables are used to store data temporarily and they can perform CRUD (Create, Read, Update, and Delete), join, and some other operations like the persistent database tables, which gives us more flexibility to get the infos we want from database. 
SQL Server supports a few types of temp tables by using `#`, `##` or `@` before the table name, they are exist for different session.

## 1.Using #TempTable
a single hashtag (#) sign before the table name indicates a local tamporary table, it's available ONLY to the session that created it and are dropped when the session is closed, use it when you need the table to be visible only in the current session.
```sql
SELECT * INTO #TableName FROM TableName where ...

SELECT * FROM #TableName
```

Another way to create it is to use create table command then use insert and select command to make a copy of the table we want.
```sql
CREATE TABLE #TableName (
    Column1 int PRIMARY KEY IDENTITY(1,1),
    Column2 varchar(255),
    Column3 varchar(255),
)

INSERT INTO #TableName (Column1,Column2)
SELECT Column1,Column1
FROM  OtherTable
```
This way, the temptable is stored in tempdb, you can check that by using
```sql
select * from tempdb.dbo.#TableName
```

It's better to drop the table once we finish the process rather than waiting for the system to drop it out.
```sql
DROP TABLE #TableName
```
or use
```sql
DROP TABLE IF EXISTS #TableName
```
While we can't create a temp table if it already exsist, this command is safe to use before creating one, as it DROP TABLE if it exsist, and ignored if not, executing this command will do both for you in one line.

## 2.Using ##TempTable
With double hashtag (##) before the table name means it's a global tamp table, which is available to ALL sessions, but are still dropped when the session that created it is closed and all other references to them are closed. Ways of creating it is the same.
```sql
SELECT * INTO #TableName FROM TableName where ...
```
or
```sql
CREATE TABLE ##TableName (
    Column1 int PRIMARY KEY IDENTITY(1,1),
    Column2 varchar(255),
    Column3 varchar(255),
)

INSERT INTO ##TableName (Column1,Column2)
SELECT Column1,Column2
FROM OtherTable
```
Also be careful to drop it after use.

## 3.Using @TempTable
A @ sign berfore table name is called Table Variable, it resides in the tempdb database as well but is accessible only within the session that created it. Meaning we have to do the crud in one session, once the command is executed, this Table Variable will be dropped automatically, which means there's no need to drop it at all. Table variables are better to use with small amounts of data.
```sql 
DECLARE @TableName AS TABLE 
(Column1 INT NOT NULL PRIMARY KEY,
 Column2 INT NOT NULL)
 
 INSERT INTO @TableName( Column1, Column2 )
 SELECT
  Column1,
  Column2 
FROM dbo.OtherTable 
```

### References
[When to Use SQL Temp Tables vs. Table Variables](https://www.sqlshack.com/when-to-use-temporary-tables-vs-table-variables/)

[Overview and Performance Tips of Temp Tables in SQL Server](https://www.sqlshack.com/overview-and-performance-tips-of-temp-tables-in-sql-server/)
