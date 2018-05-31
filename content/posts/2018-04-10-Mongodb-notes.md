---
title: "MongoDB for Developer M101P notes"
date: 2018-04-10T12:00:00-04:00
draft: false
tags: [ "mongodb" , "pymongo", "database" ]
categories: [ "programming", "data engineering" ]
---



1. insertMany(), default ordered: true, once it encounters as error, insert stop. If ordered set to false, it will insert the rest of documents despite any errors.

2. ObjectId: 12-Byte Hex string  
   _ _ _ _ , _ _ _ , _ _ , _ _ _  
   Date (time stamp), Mac addr, PID (process ID), Counter

3. Exact match on array field example: db.movieDetails.find({"writers": ["A", "B"]}).count(), ordering matters. To find a match within array: db.movieDetails.find({"writers":"A" }), same with scalar value match. If order matters, db.movieDetails.find({"writers.0":"A"}) return only the array start with "A".

4. Select field to return db.movieDetails.find({"writers":"A"}, {title:1, _id:0}). By default, _id is returned. 

5. One reason to have an $and operation is to set multiple constraint to the same field.

6. Use mongorestore to restore the dump into your running mongod. Do this by opening a terminal window (mac) or cmd window (windows) and navigating to the directory so that you are in the parent directory of the dump directory (if you used the default extraction method, it should be hw1/). Now type: `mongorestore dump`. Note you will need to have your path setup correctly to find mongorestore.

7. Update, `$push`, `$each` to add the entire array as one field within an array, `$slice` keep first nth record, `$position` push to the front if 0.

8. Upsert: update if exist, create if not exist  
   ![mongo upsert](/img/mongo upsert.png)

9. On cursor: Sort first, then skip, then limit, sort example: `cursor.sort([("a": pymongo.ASCENDING)])`

10. Difference between Pymongo vs Mongo syntax, json preserves order, dictionary from Python doesn't preserve order. Multiple sorting condition on Pymongo use tuple.

11. Insert_many, if no order flag specific, by default = True, so once there is an error, any future insert would not be executed.

12. Replace_one, update_one and update_many all call the server's update command behind the scene.

13. Update need to pass `$set` or `$unset` operator while Replace no need

14. Update one only send the set/ unset info to the database while Replace send the entire document back. (Pymongo: Updating Data using Replace Chapter 2)

15. find_one_and_delete/ _replace/ _update all use find_and_modify under the hood

16. atomic operation: indivisible, unchangeable, whole and irreducible. Either return original state or complete state, never intermediate state. Must be performed entirely or not performed at all

    - 3 ways to replace transactions in RDB
      1. restructure, all date is stored within single document
      2. implement in SW (api?)
      3. tolerate (some inconsistency)

17. Goals of normalization:

    1. free the database of modification anomalies
    2. minimize redesign when extending
    3. avoid bias toward any particular access pattern

18. Normalized tables in third normal form: every non-key attribute in the table must provide a fact about the whole key and nothing but the key. (courtroom: telling the truth, the whole truth, nothing but the truth.)

19. One-to-many relations: True linking with 2 documents
    One-to-few relations: Store within 1 documents
    many-to-many relations: embed or store Id of the other document

20. Default storage engine: MMAPv1

    1. Collection level locking (multiple reader/ single writer)
    2. In place updates
    3. Power of Two size (allocate 2^n space)

21. WeirdTiger storage engine   
     `mongod --storageEngine wiredTiger`  

     1. Document level concurency
     2. Compression of data/ indexes

22. To check index, use explain(), in the "winningPlan", if stage = "COLLSCAN" = collection scan, if using index, stage = "FETCH" with "indexName". Find how many documents it scan from "executionStats": "docExamined"

23. Multikey Indexes: can add indexes on array, can create compound indexes that includes array. If you have an index that is multikey and at least one document with a value of array, you cant have multiple values of index both be array.

24. Sparse index cannot be used for sorting if using it would omit any document in final result

25. Foreground index creation by default, it is faster but read and write would be block on db level. Background is slower but not blocked.

26. Covered query: nRetured > 0, totalKeysExamined > 0, totalDocsExamined = 0

27. Text index search = OR, e.x. `db.movies.find({$text: {$search: "dog cat"}})` return all documents with "dog" or "cat".

28. query to look in the system profile collection for all queries that took longer than one second, ordered by timestamp descending. `db.system.profile.find({millis:{$gt:1}}).sort({ts:1})`

29. Mongotop ~ unix top, mongostat ~ iostat

30. Stages of aggregation pipeline:

    - $project - reshape - 1:1
    - $match - filter - n:1
    - $group - aggregate - n:1
    - $sort - sort - 1:1
    - $skip - skips - n:1
    - $limit - limits - n:1
    - $unwind -normalize - 1:n
    - $out -output - 1:1

31. $addtoset VS $push: addtoset results is a set which is distinct, push result is a list which there are duplicated

32. $out: similar with `select into new table` , will overwrite existing collection, careful with duplicate key error

33. aggregation option, use following array aggregate([stage1,stage2,stage3],{option:1})

    - explain - query plan
    - allowDiskUse - 100MB in memory limit, if exceed, use this

34. Pymongo by default return one big document from aggregation result, Mongo Shell by default returns a cursor. To return cursor on Pymongo, add cursor=true after the aggregation array.

35. Tips on aggregation framework: modular -> add one step at a time, for debugging and confirming you are working towards the expected result

36. W is value to represent whether we are going to wait for the write to be acknowledged by the server is W, by default W = 1, J stands for journal, represent whether we wait for jornal to be written to disk before we continue, by default, J = 0. Journal to the disk is where data become persistent on disk.
    Write concern:

    - W = 1, J = 0: fast, small window of vulnerability 
    - W = 1, J = 1: slow, no vulnerability  
    - W = 0: not recommended

37. Network errors, change update into insert 

38. Types of replica set nodes:

    1. Regular
    2. Arbiter for voting
    3. delayed (Priority = 0)
    4. Hidden (not primary, priority = 0)

39. rs.status() to see the config of a replica set, rs.slaveOK() to enable read in a slave node. "Optime" is the time data is up to date until

40. db.local.oplog.rs.find() to read oplog in the node. Capped collection, will be roll off so adjust the size according to uses

41. To write robustly, need to catch AutoReconnect error, wait exponentially, and retry, catch duplicated key error as well.

42. To read robustly, need to catch AutoReconnect error only as duplicated key error is not applicable to read operations

43. To update robustly, it is possible to run the update twice using autoReconnect exception. One solution is to turn non-idempotent updates (e.x. \$push/ \$inc) into idempotent updates (e.x. $set). Idempotent means you can do the same operation multiple times with the same result.

44. Setting write concern (w/ j/ wtimeout) can be done in 3 places: 1. connection, 2. collection, 3. replica set

45. Read preference in a replica set: Primary/ PrimaryPreferred/ Secondary/ SecondaryPreferred/ Nearest

46. MongoS is the router for sharding, no longer need to connect to mongoD

47. 2 ways to do sharding: range-based (e.g. id = 1-100 in shard1, 101-200 in shard2) and hashed based

48. sh.status() gives status of a sharded system.

49. Implication of sharding:

    - every doc includes the shard key
    - shard key is immutable
    - index that starts with the shard key
    - No shard key means scatter gather (expensive)
    - no unique key unless its part of shard key
    - multi-key as shard key is illegal

50. Inserting doesn't use index, removing index can speed up inserting.

    ​