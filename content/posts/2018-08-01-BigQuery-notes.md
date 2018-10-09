---
title: "BigQuery notes"
date: 2018-08-01T12:00:00-04:00
draft: false
tags: [ "BigQuery" , "MySQL", "database", "notes" ]
categories: [ "programming", "data engineering" ]
---

There is a new project at work, we store snowplow events data on Google BigQuery (vs on AWS S3 conventionally). The query language is very similar with SQL however, there is some differences in term of best practice and cost. Here is some notes.

# BigQuery vs MySQL

BigQuery: fully-managed cloud-based enterprise data warehouse. Good for OLAP task (i.e. low volume of transactions, complex query with aggregation). Distributed storage and parallel computation for query big scale data very fast.

MySQL: self-hosted (can be managed) relational database. Good for OLTP tasks (i.e. large number of short online transactions like INSERT, DELETE, UPDATE). Scalability and performance limited to an extend.

# BigQuery Query cost

### Query pricing

BigQuery charges by using one metric: **the number of bytes processed**.  

First 1TB per month is free, then $5 per TB.

BigQuery uses a [columnar data structure](https://en.wikipedia.org/wiki/Column-oriented_DBMS). According to the total data processed in the columns you select, even if you set an explicit `LIMIT` on the results and the total data per column is calculated based on the types of data in the column. 

source: https://cloud.google.com/bigquery/pricing#on_demand_pricing

### Storage pricing

based on uncompressed data size, and based on data types of individual columns.

Active storage: first 10 GB per month is free, then $0.02 per GB

Long-term storage: first 10 per month is free, then $0.01 per GB

*Long-term storage means a table that is not edited for 90 consecutive days. Once the table is edited, the price reverts back to regular storage and the 90-day timer reset. For more details: https://cloud.google.com/bigquery/pricing#long-term-storage

# Best practice using BigQuery

1. Avoid SELETE *

   - full scan of every column
   - Use data preview for explorative queries

2. Using query validator to estimate cost ![Query validator](https://cloud.google.com/bigquery/images/query-validator.png)

3. Materialize query results in stages

   - cost of storing materialized results < cost of processing large amounts of data
   - Write interim query results by writing them to a destination table

4. Denormalize data > relational schema

   - Avoid joining tables, take advantage of nested and repeated fields

   - Increase in storage costs < performance gains 
   - Joins require data coordination (communication bandwidth), denormalization localizes data and execution can be done in parallel

5. Nested and repeated fields > completely flat

   - it might requires extra grouping and hence network communication / shuffling)


# Views at BigQuery

- Similar with MySQL
- Logical views, not materialized view. Each time we run a view, it runs the underlying query and bill according to the the total amount of data in all tables from the query.



