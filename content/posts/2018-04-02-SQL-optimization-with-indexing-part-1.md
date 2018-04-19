---
title: "SQL optimization with indexing - part 1"
date: 2018-04-02T12:00:00-04:00
draft: false
tags: [ "sql" , "mysql", "database"]
categories: [ "programming", "data engineering" ]
---

Knowing how to write a query is not enough, when our tables grow larger it is common to run into query performance issues. If we know in advance how we are querying the table, one very handful technique to "magically" speed up the query is to add appropriate indexes in tables.

## **What is index?**

From MySQL [documentation](https://dev.mysql.com/doc/refman/5.7/en/mysql-indexes.html):

> Indexes are used to find rows with specific column values quickly. Without an index, MySQL must begin with the first row and then read through the entire table to find the relevant rows. The larger the table, the more this costs. If the table has an index for the columns in question, MySQL can quickly determine the position to seek to in the middle of the data file without having to look at all the data. This is much faster than reading every row sequentially.

I think of index on a SQL table similar to the index on the side of a dictionary (yes, a physical book). By creating an index on column(s), we are mapping the column(s) to an new data structure (usually a tree structure/ hash table) holding the column value and a pointer to the original record. The index structure is then sorted logically, enabling fast lookup so we can look at the index and point directly to the corresponding rows without scanning through each rows one by one (aka [full table scan](https://dev.mysql.com/doc/refman/5.5/en/glossary.html#glos_full_table_scan)).

## **How to create index?**

There are 2 ways to create a single column index, one is to run a [`CREATE INDEX` statement](https://dev.mysql.com/doc/refman/5.7/en/create-index.html):

```mysql
CREATE INDEX [index name] ON [table name] ( [column name] )
```



The other way that does the exact same is an `ALTER TABLE` statement:

```mysql
ALTER TABLE [table name] ADD INDEX [index name] ( [column name])
```

They are mapped to the exact same implementation on the server-side, `ALTER TABEL` is a more general command that provide more functionality than `CREATE INDEX`. 

## **Toy example**

We have a huge table containing the first name, last name, and phone number, this is the first 4 rows in the table:

| id   | first_name | last_name | phone      |
| ---- | ---------- | --------- | ---------- |
| 1    | James      | Charles   | 9999999999 |
| 2    | David      | Smith     | 8888888888 |
| 3    | Amy        | Lee       | 7777777777 |
| 4    | James      | Charles   | 9999999998 |

A common task would be looking up the phone number for a specific user first name, we run a query like this to phone number from everyone with the first name "David"

```mysql
SELECT first_name, phone FROM tbl WHERE first_name = 'David'
```



If the table is not indexed, mySQL would scan from the beginning to the end of the first_name column to find all matches with 'David'. If the table is huge, it will take a long time. To speed this up, we can create an index on the column `first_name` by

```mysql
CREATE INDEX by_first_name ON students (`first_name`);
```

This statement basically create a small "table" looks like: *(This is not exactly how it works under the hood but this is a simplified version that help with my understanding. For more, read the next section)*

| value  | pointer |
| ------ | ------- |
| Amy    | 3       |
| Billy  | 23      |
| ...    | ...     |
| David  | 2       |
| David  | 200     |
| Esther | 15      |

Since the value is sorted logically (in this case aphetically or if the column data type is datetime then it will be sorted chronically and so on), every time we query on the first_name column, the search is speeded up by optimized search algorithm (e.g. binary search) and quickly locate all the rows satisfying the condition in the base table.

Other than filtering rows satisfying the `SELECT` statement with `WHERE` condition, we can also leverage the power of indexing in other operations like `JOIN`, `ORDER BY`, `GROUP BY`, `MAX()`, `MIN()` given the query satisfy certain requirements. Read more: https://dev.mysql.com/doc/refman/5.5/en/mysql-indexes.html



## **What is the disadvantage of using index?**

1. Disk space
   By creating index, we are basically duplicating part of the data (column value) on the disk and hence additional disk space is needed. By having many indexes that are not utilized, we are just wasting space.

2. `UPDATE` , `INSERT` ,`DELETE`

   When we update, add or remove records in the table, the same operation has to be performed on on index as well. That means the more indexes you have, the more it will take to perform these operations.

3. Adding an index to a table locks the table for reads and writes. Adding one for a large table takes time (in hours), so plan ahead when adding index in production. 

To conclude, if we know in advance how we are going to query the table frequently, it is incredibly beneficial to add indexes.

The more you learn about how a RMDB works, the more you are amazed by how much thoughts and work is put into optimizing all kinds of operations. Imagine the time you saved by using a free and open source db if you need to build you own data storage solution starting from basic data structures.





