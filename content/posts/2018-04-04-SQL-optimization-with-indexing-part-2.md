---
title: "SQL optimization with indexing - part 3 - multi-column index"
date: 2018-04-04T12:00:00-04:00
draft: false
tags: [ "sql" , "mysql", "database"]
categories: [ "programming", "data engineering" ]
---

MySQL can only use one index for each SELECT statement, it is very common to perform operations on multiple columns like:

```mysql
SELECT 
    gaAppIds,
    dimension_date,
    dimension_country,
    dimension_source,
    dimension_medium,
    SUM(metric_sessions)
FROM
    country_analytics
GROUP BY gaAppIds , dimension_date , dimension_source , dimension_medium , dimension_country;
```

By using a single-column index, we cannot take the full advantage for this query. We can add index for multiple columns (up to 16 columns). In the example above, we can add the following index with the 5 columns in the `GROUP BY` :

```mysql
CREATE INDEX compositeIndex
ON country_analytics (gaAppIds, dimension_date, dimension_source, dimension_medium, dimension_country);
```

If we are running the query above, the index would be utilized to speed up the query and any queries that use the left-most columns. 

Example:

- GROUP BY gaAppIds
- GROUP BY gaAppIds, dimension_date
- GROUP BY gaAppIds, dimension_date, dimension_source

Index is *not* used in the following queries:

- GROUP BY dimension_date
- GROUP BY gaAppIds, dimension_date, dimension_medium (skipped dimension_source)
- GROUP BY gaAppIds, dimension_source, dimension_date (wrong order)

Hence, the order of columns in the index is critical. If we cannot alter the indexes easily, we can change the way we query to make sure we are taking advantage of the indexes.

For more explaination on the limitation on multiple-column-indexes, read the [docs](https://dev.mysql.com/doc/refman/5.7/en/multiple-column-indexes.html):

> A multiple-column index can be considered a sorted array, the rows of which contain values that are created by concatenating the values of the indexed columns.)

This is similar to the reason why an [b-tree index](https://dev.mysql.com/doc/refman/5.7/en/index-btree-hash.html) can not be used to speed up search criteria starts with a wildcard: `LIKE '%text'`, but works for `LIKE 'text%`.

## **How to know if the query is using indexes or not?**

`EXPLAIN` statement is a very useful tool to find out information about how MySQL execute statements. To find out whether a `SELECT` query is using index or not, simply add `EXPLAIN` in the beginning of the query:

```mysql
EXPLAIN SELECT * FROM country_analytics; 
```



We get something like this:

| id   | select_type | table             | partitions | type | possible_keys | key  | key_len | ref  | rows    | filtered | Extra |
| ---- | ----------- | ----------------- | ---------- | ---- | ------------- | ---- | ------- | ---- | ------- | -------- | ----- |
| 1    | SIMPLE      | country_analytics | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 1870000 | 100.00   | NULL  |

possible_keys are key are both NULL, that means MySQL cannot find any useful index and hence not using any indexes in this query.

If we add a condition using the left-most column of the `compositeIndex` we added earlier:

```mysql
EXPLAIN SELECT * FROM country_analytics where  gaAppIds = 12345678;
```



We get:

| id   | select_type | table             | partitions | type | possible_keys  | key            | key_len | ref   | rows   | filtered | Extra |
| ---- | ----------- | ----------------- | ---------- | ---- | -------------- | -------------- | ------- | ----- | ------ | -------- | ----- |
| 1    | SIMPLE      | country_analytics | NULL       | ref  | compositeIndex | compositeIndex | 5       | const | 937000 | 100.00   | NULL  |

We can see compositeIndex on both possible_keys and key, that means MySQL is using the index to speed up the query, and the rows scanned reduced by about 50% in this case.

In some cases, we will see possible_keys but not in key, that means MySQL find that the index would be possibly useful but it did not use it at the end. 

ref: http://www.vertabelo.com/blog/technical-articles/an-introduction-to-mysql-indexes

ref: https://stackoverflow.com/questions/1823685/when-should-i-use-a-composite-index



## **How to check existing index info?**

Use `SHOW INDEX FROM tbl_name`, you can see information for all the indexes added in the table like index_type, index sequence for composite index etc.

ref: https://dev.mysql.com/doc/refman/5.7/en/show-index.html







