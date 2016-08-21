---
layout: post
title: "Kivas and Storm"
excerpt: "Kivas and Storm"
modified: 2016-08-01
categories: articles
author: DB_Hive
tags: [Hadoop]
comments: true
share: true
---

## Kivas :

Kivas is a distributed system designed for streaming message consumption, processing, and storage.

A variety of entry and exit points have been created allowing products to easily interface with the system with very limited code changes. 
Messages running through Kivas have the option to pass through the Storm complex event processor for aggregation, filtering, or other computation tasks before these message are stored or further processed.

Messages can enter Kivas through a number of entry points depending on application requirements and what is most convenient to integrate into your existing application. 
From the entry point, messages are synchronously persisted into Kafka, guaranteeing that your message has been received and persisted to the queue for your application. 
From this point, a set of exit points retrieve your messages from Kafka and deliver them to a configured location such as Storm, HDFS, or SQL Server.



In KIVAs you can create STORM topologies

## Storm:

Storm is a real-time, distributed, fault-tolerant, computation system. Like Hadoop, it can process huge amounts of data—but does so in real time — with guaranteed reliability; that is, every message will be processed.

Data moves through Storm within a "Topology" in the form of messages called "Tuples".

### Storm Terminology

* Topology: 
A grouping of Spouts and Bolts working together on a particular task or goal.
* Spout: 
Entry points into a Topology. Typically connect to data sources such as Kafka.
* Bolt: 
A processing step in a Topology. Can perform transformations, aggregations, write to data sources, and more.
* Tuple: 
A grouping of data to send between Spouts and Bolts.
* Nimbus: 
The "master node" in a Storm cluster.
* Supervisor: 
A node which coordinates several Workers within an individual machine .
* Worker: 
A process which can execute a Spout or Bolt.



