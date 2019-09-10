---
layout:     post
title:      "SQL Server concurrent transactions"
subtitle:   "Isolation levels and concurrent problems"
date:       2017-05-20 12:00:00
author:     "Shengwen"
header-img: "img/database-bg.jpg"
tags:
    - SQL Server
    - Multi-threading
---

## Catalog

1. [Isolation Levels](#isolation-levels)
2. [Concurrent Problems](#concurrent-problems)
    1. [Dirty Read](#dirty-read)
    2. [Lost Update](#lost-update)
    3. [Nonrepeatable Read](#nonrepeatable-read)
    4. [Phantom Read](#phantom-read)
3. [Difference between repeatable read and serializable](#difference-between-repeatable-read-and-serializable)
4. [Difference between serializable and snapshot isolation levels](#difference-between-serializable-and-snapshot-isolation-levels)
5. [Read committed snapshot isolation level](#read-committed-snapshot-isolation-level)
6. [Difference between snapshot isolation and read committed](#difference-between-snapshot-isolation-and-read-committed)

## Isolation Levels

|Isolation Level|Dirty Reads|Lost Update|Nonrepeatable Reads|Phantom Reads|
|-|-|-|-|-|
|Read Uncommitted|Yes|Yes|Yes|Yes|
|Read Committed|No|Yes|Yes|Yes|
|Repeatable Read|No|No|No|Yes|
|Snapshot|No|No|No|No|
|Serializable|No|No|No|No|

## Concurrent Problems

#### Dirty Read
A dirty read happens when one transaction is permitted to read data that has been modified by another transaction that has not yet been committed. In most cases this would not cause a problem. However, if the first transaction is rolled back after the second reads the data, the second transaction has dirty data that does not exist anymore.
![](/static/in-post/post-concurrent/dirtyread.png)

#### Lost Update
Lost update problem happens when 2 transactions read and update the same data.
![](/img/in-post/post-concurrent/lostupdate.png)

#### Nonrepeatable Read
Non repeatable read problem happens when one transaction reads the same data twice and another transaction updates that data in between the first and second read of transaction one. 
![](/img/in-post/post-concurrent/nonrepeatableread.png)

#### Phantom Read
Phantom read happens when one transaction executes a query twice and it gets a different number of rows in the result set each time. This happens when a second transaction inserts a new row that matches the WHERE clause of the query executed by the first transaction.
![](/img/in-post/post-concurrent/phantomread.png)

## Difference between repeatable read and serializable
Repeatable read prevents only non-repeatable read. Repeatable read isolation level ensures that the data that one transaction has read, will be prevented from being updated or deleted by any other transaction, but it doe not prevent new rows from being inserted by other transactions resulting in phantom read concurrency problem. 

Serializable prevents both non-repeatable read and phantom read problems. Serializable isolation level ensures that the data that one transaction has read, will be prevented from being updated or deleted by any other transaction. It also prevents new rows from being inserted by other transactions, so this isolation level prevents both non-repeatable read and phantom read problems. 

## Difference between serializable and snapshot isolation levels
Serializable isolation is implemented by acquiring locks which means the resources are locked for the duration of the current transaction. This isolation level does not have any concurrency side effects but at the cost of significant reduction in concurrency. 

Snapshot isolation doesn't acquire locks, it maintains versioning in Tempdb. Since, snapshot isolation does not lock resources, it can significantly increase the number of concurrent transactions while providing the same level of data consistency as serializable isolation does.

## Read committed snapshot isolation level
Read committed snapshot isolation level is not a different isolation level. It is a different way of implementing Read committed isolation level. One problem we have with Read Committed isloation level is that, it blocks the transaction if it is trying to read the data, that another transaction is updating at the same time. 

We can make the transaction that reads data to use row versioning technique instead of locks by enabling Read committed snapshot isolation at the database level. Use the following command to enable READ_COMMITTED_SNAPSHOT isolation
```sql
Alter database SampleDB SET READ_COMMITTED_SNAPSHOT ON
```
## Difference between snapshot isolation and read committed

|Read Committed Snapshot Isolation|Snapshot Isolation|
|-|-|
|No update conflicts|Vulnerable to update conflicts|
|Works with existing applications without requiring any change to the application|Application change may be required to use with an existing application|
|Can be used with distributed transactions|Cannot be used with distributed transactions|
|Provides statement-level read consistency|Provides transaction-level read consistency|

Read Committed Snapshot Isolation returns the last committed data before the select statement began and not the last committed data before the transaction began. 

Snapshot Isolation returns the last committed data before the transaction began and not the last committed data before the select statement began. 


Reference: [kudvenkat's YouTube channel](https://www.youtube.com/user/kudvenkat)
