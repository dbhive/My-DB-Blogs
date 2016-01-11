---
layout: post
title: How to optimize HIVE queries
excerpt: "How to optimize HIVE queries."
modified: 2014-11-01
categories: articles
author: DB_Hive
tags: [Hadoop]
comments: true
share: true
---

Until recently, optimizing Hive queries focused mostly on data layout techniques such as partitioning and bucketing or using custom 
file formats.
In the last couple of years, driven largely by the innovation of the Hive community around the Stinger initiative,
Hive query time has improved dramatically, enabling Hive to support both batch and interactive workloads at speed and at scale.
However, many data scientists remain unfamiliar with basic techniques and best practices for running their Hive queries at maximum speed.
In this blog post, I will highlight a few simple tricks I use most often to improve performance of my HIVE queries.

### 1: Use Tez
Hive can use the Apache Tez execution engine instead of the venerable Map-reduce engine. 
I won’t go into details about the many benefits of using Tez which are mentioned here; 
instead, I want to make a simple recommendation: if it’s not turned on by default in your environment, 
use Tez by setting to ‘true’ the following in the beginning of your Hive query:
1. set hive.execution.engine=tez;

With the above setting, every HIVE query you execute will take advantage of Tez.

### 2: Use ORCFile
Hive supports ORCfile, a new table storage format that sports fantastic speed improvements through techniques like predicate push-down,
compression and more.
Using ORCFile for every HIVE table should really be a no-brainer and extremely beneficial to get fast response times 
for your HIVE queries.
As an example, consider two large tables A and B 
(stored as text files, with some columns not all specified here), and a simple query like:

SELECT  A.customerID, A.name, A.age, A.address join
 B.role, B.department, B.salary 
 ON A.customerID=B.customerID;
This query may take a long time to execute since tables A and B are both stored as TEXT. 
Converting these tables to ORCFile format will usually reduce query time significantly:


CREATE TABLE A_ORC (
 customerID int, name string, age int, address string
) STORED AS ORC tblproperties (“orc.compress" = “SNAPPY”);


INSERT INTO TABLE A_ORC SELECT * FROM A;
 
CREATE TABLE B_ORC (
 customerID int, role string, salary float, department string
) STORED AS ORC tblproperties (“orc.compress" = “SNAPPY”);
 
INSERT INTO TABLE B_ORC SELECT * FROM B;
 
SELECT  A_ORC.customerID, A_ORC.name, 
 A_ORC.age, A_ORC.address join
 B_ORC.role, B_ORC.department, B_ORC.salary 
 ON A_ORC.customerID=B_ORC.customerID;
 
ORC supports compressed storage (with ZLIB or as shown above with SNAPPY) but also uncompressed storage.
Converting base tables to ORC is often the responsibility of your ingest team, and it may take them some time to change the complete 
ingestion process due to other priorities. The benefits of ORCFile are so tangible that I often recommend a do-it-yourself approach as
demonstrated above – convert A into A_ORC and B into B_ORC and do the join that way, so that you benefit from faster queries immediately,
with no dependencies on other teams

### 3: Use Vectorization
Vectorized query execution improves performance of operations like scans, aggregations, filters and joins, by performing them in batches of 1024 rows at once instead of single row each time.
Introduced in Hive 0.13, this feature significantly improves query execution time, and is easily enabled with two parameters settings:

set hive.vectorized.execution.enabled = true;
set hive.vectorized.execution.reduce.enabled = true;

### 4: cost based query optimization 
Hive optimizes each query’s logical and physical execution plan before submitting for final execution. 
These optimizations are not based on the cost of the query – that is, until now.
A recent addition to Hive, Cost-based optimization, performs further optimizations based on query cost, resulting in potentially 
different decisions: how to order joins, which type of join to perform, degree of parallelism and others.
To use cost-based optimization (also known as CBO), set the following parameters at the beginning of your query:
 
set hive.cbo.enable=true;
set hive.compute.query.using.stats=true;
set hive.stats.fetch.column.stats=true;
set hive.stats.fetch.partition.stats=true;

Then, prepare the data for CBO by running Hive’s “analyze” command to collect various statistics on the tables for which we want to use CBO.
For example, in a table tweets we want to collect statistics about the table and about 2 columns: “sender” and “topic”:
 
analyze table tweets compute statistics;
analyze table tweets compute statistics for columns sender, topic;

With HIVE 0.14 (on HDP 2.2) the analyze command works much faster, and you don’t need to specify each column, so you can just issue:

analyze table tweets compute statistics for columns;

That’s it. Now executing a query using this table should result in a different execution plan that is faster because 
of the cost calculation and different execution plan created by Hive.

### 5 Use map-side joins 


Another technique is to use map-side joins – by setting the following params:

set hive.auto.convert.join=true;
set hive.auto.convert.join.noconditionaltask=true;
set hive.auto.convert.join.noconditionaltask.size=30000000;

You’ll know it’s being used when you see something like the following in the logs:

SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
Execution log at: /tmp/psgetl/psgetl_20150828051515_08972512-beb2-4da9-8da5-338092cdf940.log
2015-08-28 05:16:24     Starting to launch local task to process map join;      maximum memory = 4261937152
2015-08-28 05:16:39     Dump the side-table into file: file:/tmp/psgetl/hive_2015-08-28_05-15-32_360_4897376719178160971-1/-local-10008/HashTable-Stage-3/MapJoin-mapfile21--.hashtable
2015-08-28 05:16:41     Uploaded 1 File to: file:/tmp/psgetl/hive_2015-08-28_05-15-32_360_4897376719178160971-1/-local-10008/HashTable-Stage-3/MapJoin-mapfile21--.hashtable (390642 bytes)
2015-08-28 05:16:42     Dump the side-table into file: file:/tmp/psgetl/hive_2015-08-28_05-15-32_360_4897376719178160971-1/-local-10008/HashTable-Stage-3/MapJoin-mapfile11--.hashtable
2015-08-28 05:16:42     Uploaded 1 File to: file:/tmp/psgetl/hive_2015-08-28_05-15-32_360_4897376719178160971-1/-local-10008/HashTable-Stage-3/MapJoin-mapfile11--.hashtable (527 bytes)
2015-08-28 05:16:42     End of local task; Time Taken: 18.664 sec.
Execution completed successfully
 


