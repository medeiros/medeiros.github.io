---
layout: post
title: "Kafka Connect Concepts"
description: >
  Review of the core concepts in Apache Kafka.
categories: distributedarchitecture
tags: [core, kafka]
comments: true
---
> Kafka is a distributed, resilient, fault tolerant streaming platform that works with high data throughput.
In this page, the main concepts of Kafka Connect technology will be covered.
{:.lead}

- Table of Contents
{:toc}

## What are Kafka Connectors

Kafka Connectors are components that make it easier to programmers to get and
send data to/from external sources or sink to/from Kafka Cluster.

Those are bridges between Kafka Cluster and the external world.

## Why to Use Kafka Connect

- Simplifies common actions that programmers usually do (reading data to send
  to Kafka, get data from Kafka and save it elsewhere)
- Promote reuse of common problems (since it's a component)
- Some issues are hard to solve in programming (ditributed ordering, exacly
  once, etc), and it is usually better to use a common connector
  that already solve these problems, instead of solve those from scratch

## Sources and Sinks

A connector can be a `Sink`, a `Source`, or both.

- `Source`: acting as a producer, bringing data to Kafka from an external
source
  - for instance: to read data from REST API (source), parse its JSON response  
and send the parsed data to Kafka Cluster
- `Sink`: acting as a consumer, reading data from Kafka and saving them in an
external sink
  - for instance: get data from Kafka Cluster and send it to Tweeter (sink)
as a tweet

## Common Use Cases

|--|--|--|
Source -> Kafka|Producer API|Kafka Connect Source
Kafka <-> Kafka|Consumer API, Producer API|Kafka Streams
Kafka <- Sink|Consumer API|Kafka Connect Sink
Kafka <- App|Consumer API|-

## Kafka Connect High-Level Overview

In a high overview, Kafka Connect interacts with Kafka in the following way:

![](/assets/img/blog/kafka/kafka-connect-overview.png)

Figure: Kafka Connnect Overview.
{:.figcaption}

As described in the previous diagram:

- step 1. Kafka Connector Source (running into a Cluster) go get data from
source (external)
- step 2. Kafka Connector Source sends collected data to Kafka Cluster (acting
  like a producer in that regard)
- step 3a. Kafka client application (acting as consumer) read data from Kafka
Cluster
- step 3b. Kafka client application (now acting as producer) send processed
data to Kafka Cluster
- step 4. Kafka Connector Sink (running into a Cluster) read data from Kafka
Cluster (acting like a consumer in that regard)
- step 5. Kafka Connector Sink save read data into a sink (external)

## Workers

Connectors in execution are called `Workers`. In those workers, scalability is
also supported.

- Each worker is an isolated, simple Java Process (JVM)
- Workers run Connectors (_each connector is a `jar` file_)
- Worker runs Connectors' `Tasks` to perform its actions
  - A Task is linked with a connector configuration
  - A job configuration can be composed of several tasks
- A Worker can run in standalone mode or distributed mode

|--|--|
Standalone Worker|Distributed Worker
|--|--|
A single process run both connectors and tasks|Multiple workers run connectors and tasks
Configuration is set into process|Configuration is done by a REST API
Very easy to use; good for dev and test|Useful for production deployment
No fail tolerance, no scalability, hard to monitor|Easy to scale (only add new Workers), and fail tolerant (automatic rebalance in case of a inactive worker)

Distributed workers do not necessary have to run in a cluster environment.
For testing purposes, one may run several Workers in the same machine,
just starting JVMs using different properties files.
{:.note}

## Cluster and Distributed Architecture

A Kafka Connect Cluster can handle multiple Connectors.

![](/assets/img/blog/kafka/kafka-connect-workers-cluster.png)

Figure: Kafka Connnect: Workers in a Cluster.
{:.figcaption}

In the previous diagram:
- Each conector will have its tasks spread in the cluster workers'
  - A single worker can have more than one task from the same connector
- When a Worker dies, its tasks are rebalanced to other live brokers

## Kafka Connect: Landoop

Landoop is a tool to make it easy to manage Kafka Connect workers.
- There is a docker image to make the usage simple (landoop/fast-data-dev)
- default URL: http://localhost:3030

## Running a Standalone Worker to Read a File and Send it to Kafka

Let's run a simple worker in standalone mode to read a text file (source) and
send it to the Kafka Broker. The Source Connector for a File Stream is native
to Kafka, so we don't need to import any jars for this example.

[FileStreamSourceConnector Source code @ Github](https://github.com/apache/kafka/blob/trunk/connect/file/src/main/java/org/apache/kafka/connect/file/FileStreamSourceConnector.java)
{:.note}

First of all, it is necessary to create a properties file to configure this
connector.

Let's call it `file-standalone.properties`, with the following content:

`/home/daniel/data/kafka/connectors/file-standalone.properties`
```properties
# These are standard kafka connect parameters, needed for ALL connectors
name=file-standalone
connector.class=org.apache.kafka.connect.file.FileStreamSourceConnector
tasks.max=1
# FileStremSourceConnector's properties
file=/home/daniel/data/kafka/connectors/file-to-read.txt
topic=connect-source-file-topic
```

Next, let's create this file to be read (_file-to-read.txt_):

`/home/daniel/data/kafka/connectors/file-to-read.txt`
```
hello
using 'FileStreamSourceConnector' Kafka Connector Source to produce some data
another line produced
one more line
and thats ok for now
bye
```

Next step is to start Kafka in the local environment:

```bash
kafka $ ./bin/zookeeper-server-start.sh -daemon ./config/zookeeper.properties
kafka $ ./bin/kafka-server-start.sh -daemon ./config/server.properties
```

Now, we can execute the worker:
```bash
kafka $ ./bin/connect-standalone.sh ./config/connect-standalone.properties
  /home/daniel/data/kafka/connectors/file-standalone.properties
```

- The first parameter of shell is a property file that define brokers
characteristics. In that case, we're using Kafka Default file for Kafka
Connect in standalone mode
- The second parameter is the property file to our specific configuration of
Kafka Connect File Stream Source Connector. You can pass as many files after
the first one (the broker definition), and each file will be related to a
different Sink/Source Connector.

After that, we can consume this topic and see the results:

```bash

kafka $ ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092
  --topic connect-source-file-topic --from-beginning

{"schema":{"type":"string","optional":false},"payload":"using
  'FileStreamSourceConnector' Kafka Connector Source to produce some data"}
{"schema":{"type":"string","optional":false},"payload":"another line produced"}
{"schema":{"type":"string","optional":false},"payload":"one more line"}
{"schema":{"type":"string","optional":false},"payload":"and thats ok for now"}
{"schema":{"type":"string","optional":false},"payload":"bye"}
{"schema":{"type":"string","optional":false},"payload":"hello"}
```

As we can notice, all file information was written as JSON in Kafka Broker (
but our file is not a JSON file). That is because the default Kafka file for
worker properties was used (that uses JSON for conversion).

Considering again the call to Kafka Connect in standalone mode:
```bash
kafka $ ./bin/connect-standalone.sh ./config/connect-standalone.properties
  /home/daniel/data/kafka/connectors/file-standalone.properties
```
- `file-standalone.properties`: this is the file that was created to configure
the connect source to read our particular file
- `./config/connect-standalone.properties`: this is the default Kafka file,
that assumes the following properties:
```properties
bootstrap.servers=localhost:9092
key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
key.converter.schemas.enable=true
value.converter.schemas.enable=true
offset.storage.file.filename=/tmp/connect.offsets
offset.flush.interval.ms=10000
```

Since `key.converter` and `value.converter` pointed to a `JsonConverter` class,
that explains why our data is written as JSON. Naturally, this can be changed.

If we try to change the value of both `key.converter` and `value.converter`
to a String converter (`org.apache.kafka.connect.storage.StringConverter`) and
run again, it was expected that data would be generated as string instead of
JSON. However, no data is even published.

That is because ...
curl localhost:8083/connectors/file-standalone/tasks/0/restart -X POST
(improve here)


## References

- [Kafka: The Definitive Guide](https://www.confluent.io/resources/kafka-the-definitive-guide/)
- [Stephane Maarek's Kafka Courses @ Udemy](https://www.udemy.com/courses/search/?courseLabel=4556&q=stephane+maarek&sort=relevance&src=sac)
