---
title: "SQL optimization with indexing - part 2 - index types in MySQL"
date: 2018-04-03T12:00:00-04:00
draft: false
tags: [ "sql" , "mysql", "database"]
categories: [ "programming", "data engineering" ]
---

There are 2 main data structures for index in MySQL, they are B-tree and hash table. They are also the main index structures for other RMDBS. 

## **B-tree**

B-tree is a self-balancing sorted data structure similar to a [binary search tree](https://en.wikipedia.org/wiki/Binary_search_tree). The only difference is that a node in B-tree can have more than 2 children. For me, it looks like a binary tree with a range of numbers in each node (vs only 1 number in a node). As we know, updating a binary tree is costly, B-tree is a more efficient way to enable search, sequential access, insertion and deletion faster. 

![img](https://upload.wikimedia.org/wikipedia/commons/thumb/6/65/B-tree.svg/400px-B-tree.svg.png)

When we create an index on column(s), the column(s) value is first sorted, then it would be stored in each node along with a pointer to the row.

## **Hash table**

Hash table is making use of the hash function to generate the "address" where the index is stored. For each row in a column, the value will pass through a hash function h(input) = index, where the index is the actual bucket that is storing the key and value. To look up a row, we can always pass the input in the hash function and it will always return the index. Then the data is retrieved. 

![Image result for hash table mysql](http://www.csci.csusb.edu/tongyu/courses/cs330/images/hash/HASHTBL.png)

Hash table is very efficient to look up a single record.

## **Difference between B-tree vs Hash indexes**

Generally by default, B-tree is used for the following reasons:

1. It is more flexible as it can be used for column comparison that use =, >, >=, `BETWEEN`  and even `LIKE` operators while hash table can be used only for equality comparisons like = or <=>.
2. Hash table is not as scalable as B-tree, as the table size grows, the index might have to be rehashed.
3. Hash table cannot be use to optimize `ORDER BY` operations.

However, in term of performance, hash table is very fast, it is O(1) vs tree algorithm is O(log n). However, in some case if we need to get a wide range of rows that is a significant portion of the table, then using hash table index might even be slower than not using any index at all. Usually mySQL is smart enough to decide whether to use index.

reference:

https://dev.mysql.com/doc/refman/5.5/en/index-btree-hash.html

https://stackoverflow.com/questions/7306316/b-tree-vs-hash-table

http://www.vertabelo.com/blog/technical-articles/all-about-indexes-part-2-mysql-index-structure-and-performance