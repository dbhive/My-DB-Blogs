---
layout: post
title: Various types of File Formats in Hadoop
excerpt: "Various types of File Formats in Hadoop"
modified: 2015-10-30
categories: articles
author: DB_Hive
tags: [Hadoop]
comments: true
share: true
---


# Various File Formats supported in Hadoop:

### 1.Text/CSV Files
CSV files are still quite common and often used for exchanging data between Hadoop and external systems. 
They are readable and ubiquitously parsable. They come in handy when doing a dump from a database or bulk loading data from 
Hadoop into an analytic database. However, CSV files <b>do not</b> support block compression, thus compressing a CSV file in Hadoop 
often comes at a significant read performance cost.
When working with Text/CSV files in Hadoop, never include header or footer lines.
Each line of the file should contain a record. This, of course, means that there is no metadata stored with 
the CSV file. You must know how the file was written in order to make use of it. Also, since the file structure is 
dependent on field order, new fields can only be appended at the end of records while existing fields can never be deleted. 
As such, CSV files have limited support for schema evolution.
 
### 2. JSON Records

JSON records are different from JSON Files in that each line is its own JSON datum making the files splittable. 
Unlike CSV files, JSON stores metadata with the data, fully enabling schema evolution.
However, like CSV files, JSON files do not support block compression. 
Additionally, JSON support was a relative late comer to the Hadoop toolset and many of the native serdes contain significant bugs. 
Fortunately, third party serdes are frequently available and often solve these challenges. 
You may have to do a little experimentation and research for your use cases.

### 3. Avro Files

Avro files are quickly becoming the best multi-purpose storage format within Hadoop. 
Avro files store metadata with the data but also allow specification of an independent schema for reading the file. 
This makes Avro the epitome of schema evolution support since you can rename, add, delete and change the data types of fields by 
defining new independent schema. Additionally, Avro files are splittable, support block compression and enjoy broad, 
relatively mature, tool support within the Hadoop ecosystem.

### 4. Sequence Files

Sequence files store data in a binary format with a similar structure to CSV. Like CSV, sequence files do not store metadata 
with the data so the only schema evolution option is appending new fields. However, unlike CSV, sequence files do support
block compression. Due to the complexity of reading sequence files, they are often only used for in flight data such as 
intermediate data storage used within a sequence of MapReduce jobs.

### 5. RC Files

RC Files or Record Columnar Files were the first columnar file format adopted in Hadoop. Like columnar databases, 
the RC file enjoys significant compression and query performance benefits. However, the current serdes for RC files in Hive 
and other tools do not support schema evolution. In order to add a column to your data you must rewrite every pre-existing RC 
file. Also, although RC files are good for query, writing an RC file requires more memory and computation than non-columnar file formats. 
They are generally slower to write.

### 6. ORC Files

ORC Files or Optimized RC Files were invented to optimize performance in Hive and are primarily backed by HortonWorks. 
ORC files enjoy the same benefits and limitations as RC files just done better for Hadoop. This means ORC files compress better than 
RC files, enabling faster queries. However, they still do not support schema evolution. 
Some benchmarks indicate that ORC files compress to be the smallest of all file formats in Hadoop. It is worthwhile to note that,
at the time of this writing, Cloudera Impala does not support ORC files.

### 7. Parquet Files

Parquet Files are yet another columnar file format that originated from Hadoop creator Doug Cuttings Trevni project. 
Like RC and ORC, Parquet enjoys compression and query performance benefits, and is generally slower to write than non-columnar file
formats. However, unlike RC and ORC files Parquet serdes support limited schema evolution. In Parquet, new columns can be added at 
the end of the structure. At present, Hive and Impala are able to query newly added columns, but other tools in the ecosystem such 
as Hadoop Pig may face challenges. Parquet is supported by Cloudera and optimized for Cloudera Impala. Native Parquet support is 
rapidly being added for the rest of the Hadoop ecosystem.
One note on Parquet file support with Hive... It is very important that Parquet column names are lowercase. If your Parquet file 
contains mixed case column names, Hive will not be able to read the column and will return queries on the column with null 
values and not log any errors. Unlike Hive, Impala handles mixed case column names. A truly perplexing problem when you encounter it.\


#### How to choose a file format?
There are three types of performance to consider: 
* Write performance -- how fast can the data be written.
* Partial read performance -- how fast can you read individual columns within a file.
* Full read performance -- how fast can you read every data element in a file.


A columnar, compressed file format like Parquet or ORC may optimize partial and full read performance, but they do so at the expense
of write performance. Conversely, uncompressed CSV files are fast to write but due to the lack of compression and column-orientation 
are slow for reads. You may end up with multiple copies of your data each formatted for a different performance profile. 

As discussed, each file format is optimized by purpose. Your choice of format is driven by your use case and environment. 
Here are the key factors to consider:
* Hadoop Distribution- Cloudera and Hortonworks support/favor different formats
* Schema Evolution- Will the structure of your data evolve.
* Processing Requirements- Will you be crunching the data and with what tools.
* Read/Query Requirements- Will you be using SQL on Hadoop? Which engine.
* Extract Requirements- Will you be extracting the data from Hadoop for import into an external database engine or other platform.
* Storage Requirements- Is data volume a significant factor? Will you get significantly more bang for your storage buck through
compression.


So, with all the options and considerations are there any obvious choices? If you are storing intermediate data between MapReduce jobs, then Sequence files are preferred. 
If query performance against the data is most important, ORC (HortonWorks/Hive) or Parquet (Cloudera/Impala) are optimal 
but these files will take longer to write. 
(We have also seen order of magnitude query performance improvements when using Parquet with Spark SQL.)
Avro is great if your schema is going to change over time, but query performance will be slower than ORC or Parquet.
CSV files are excellent if you are going to extract data from Hadoop to bulk load into a database.
