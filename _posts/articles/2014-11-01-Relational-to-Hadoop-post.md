---
layout: post
title: Journey from relational DB to hadoop
excerpt: "Journey from relational DB to hadoop."
modified: 2014-11-01
categories: articles
tags: [sample-post]
comments: true
share: true
---

Let me start this blog by telling you a story. There was data architect named Tom.
He had to build a data warehouse for a financial services corporation to capture settlement data. This financial corporation was a small company. Tom selected relational database IBM DB2 for storing this data. He selected relational database because it is a predominant choice in storing data. DB2 is a mature database. Lot of documentation and tools was available to support and administrator it. There was no one in the firm who was challenging this solution. There were entities and relations defined. Nice 3rd normalization was put in place to minimize redundancy. There were smaller tables with well-defined relationship between them. 
SQL was supported which is very flexible and can query data pretty fast. Most of the developers and analysists in the firm knew SQL so there was very less learning curve.
 It was a decent design. Query performance was very good. It was supporting ACID properties. Everyone was happy.

Few months down the line, business was doing good which started bringing lot of data into the system. Queries started taking more and more time.. Complex reports started taking minutes. Everyone started contacting Tom again. After few days of analysis, Tom created few indexes. Queries again started performing well. He even created few MQTs (Materialized Query Tables) which made complex reports very fast! Everyone was happy. 

Few months down the line, again system started giving performance issues, backup was taking lot of time.. Downtime increased, Queries were slow. Indexes were huge.
Tom advised to get a new hardware. Company spend good amount of money to buy new hardware. Migration was easy and fast. Downtime of few hours did the migration to new hardware. Query performance was again very fast. and again everyone was happy.

Few months down the line business was expanding very fast.. and so was data. and the settlement system was again performing bad. Queries were slow. Now Tom took a hard decision to scale vertically. He started exploring massive parallel processing and introduces sharding. They again bought a fancy hardware cluster. Data was partitioned across nodes . Migration was tough and lengthy this time. and after few days Tom had a new system . This was a real game changer. It was a multi node system. Query performance was good. There were separate nodes for ETL and reporting. Everyone was happy.. Frankly this system worked fine for a long time. Whenever there were performance issues, Tom used to order more hardware, add mode nodes to the database , go through a painful migration and boom.. performance was back in acceptable limits.
In the meanwhile, Tom started de-normalizing the data. This helped in reducing # of joins in queries but introduced redundancy . This had its own problems . It was difficult to maintain consistency.

Tom, introduced one more server which was exclusive for reporting. This made queries faster but further introduced consistency issues. 
By Now Tom has reached a dead end.

Here comes Hadoop . 

The Apacheª Hadoop¨ project develops open-source software for reliable, scalable, distributed computing.
The Apache Hadoop software library is a framework that allows for the distributed processing of large data sets across clusters of computers using simple programming models. It is designed to scale up from single servers to thousands of machines, each offering local computation and storage. Rather than rely on hardware to deliver high-availability, the library itself is designed to detect and handle failures at the application layer, so delivering a highly-available service on top of a cluster of computers, each of which may be prone to failures.

