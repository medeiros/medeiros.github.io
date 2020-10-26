---
layout: post
title: "Kafka Streams Concepts"
description: >
  Review of the streams concepts in Apache Kafka.
categories: distributedarchitecture
tags: [core, kafka]
comments: true
image: /assets/img/blog/kafka/kafka-streams-topology.png
---
> Kafka is a distributed, resilient, fault tolerant streaming platform that
works with high data throughput.
In this page, the main concepts of Kafka Streams technology will be covered.
{:.lead}

- Table of Contents
{:toc}

## Kafka Streams Concepts

A **Stream is a sequence of messages**. It is immutable, so it cannot be
changed by any means, by definition.

Kafka Topic is an example of Stream: it is replayed, fault-tolerant and can be
ordered.

Streams work in a **Topology** concept. **Stream Processor** is a **node** on
that Topology, responsible to process the stream in a specific way. It can
create new streams, but never change an existing stream - since it's immutable.
Stream Processors are connected by Streams.

![](/assets/img/blog/kafka/kafka-streams-topology.png)

Figure: Kafka Streams Topology.
{:.figcaption}

A **Source Stream Processor** is the one who connects to a topic to bring data
to the topology.

- It has no parent in the topology
- Do not transform data

A **Sink Stream Processor** is the one who send processed data back to a Kafka
topic.

- It is a leaf in the topology; has no successors
- Do not transform data

So, as we can see, Kafka Streams only iteracts to Kafka **Topics**, to get and
send data. This is different than [Kafka Connect](blog/distributedarchitecture/2020-10-15-Kafka-Connect-Concepts/),
that brings and send data to all kinds of different external systems.

## Kafka Streams Architecture: Simple and Flexible

Kafka Streams are very simple in terms of architecture. A Kafka Streams
application is just a standard java application (jar) that runs in a JVM as a
isolated process. The concept of "cluster" do not apply to it.

Since Kafka Streams inherits Kafka characteristics, it also can be considered
fault-tolerant and scalable.

Because of this simple architecture, it supports any application size - from
the smallest to the largest one.  

### Where is Kafka Streams in a Overall Kafka Architecture?

![](/assets/img/blog/kafka/kafka-streams-overview.png)

Figure: Kafka Streams Overview.
{:.figcaption}

The numbered steps are below described:

1. Kafka Source Connector gets some data
2. Kafka Source Connector ask to its Worker to send data to the Broker's topic
3. **Kafka Streams gets data from the Broker's topic, processes it and send it
back to another topic**
4. Kafka Sink Connector asks to its Worker to get data from the second topic
5. Kafka Sink Connector send processed data to the destination

### True Streaming: One Record at a Time

Kafka Streams process one record at a time. It is not a batch mechanism, like
Spark, but a stream mechanism that **process** and **transform** records, one
at a time.

### Exactly Once Capabilities

Kafka Streams supports exactly once capabilities. This means it can handle
each message in a way that for each read message, there will be a write message,
every single time, without the hassle of reattempts and concerns about acks.
A simple Kafka Producer/Consumer (and other streams platform systems) can
only handle `at least once` and `at most once` capabilities, but Kafka Streams
can handle the ideal, `exactly once` capability.

### Kafka Streams Internal Topics

Some intermediate topics are created internally by Kafka Streams:

- **Repartitioning Topics**: to handle stream key transformation
- **Changelog Topics**: to handle aggregations of messages

Messages in those topics will be saved in a compacted way. Those topics
have the prefix `application.id`, and are basically used to save/restore state
and to partition data.

These internal topics are managed exclusively by Kafka Streams, and must never
be directly manipulated.

### Scaling Kafka Streams Application

Kafka Streams uses Kafka Consumer, so the same rules apply.

- Each partition can only be read by a Kafka Stream App Process (for
the same consumer group) at a time
- If you run more Kafka Stream processes than the number of partitions,
those aditional processes will remain inactive

So, for each partition to be consumed in a topic, it could have a
Kafka Streams application process running. For instance, if a topic has
3 partitions, so three Kafka Stream Application processes could be run in
parallel to read data from the topic.

There is no need for any cluster! Just run the Java process and that's it.
{:.note}

Kafka Streams algo handle rebalancing (in a case that one of the Kafka Streams
application processes dies), just as with consumers.


## Kafka Streams Compared to Other Tools

This table is a comparison between Kafka Stream and another tools the perform
similar job (Spart Streaming, Apache NiFi and Apache Flink):

Item|Kafka Streams|Other Related Tools
--|--|--
Mechanism|Stream|Micro Batch
Requires Cluster?|No|Yes
Scale just by adding Java Processes?|Yes|No
Exactly Once Semantics related to Kafka Brokers?|Yes|No (at least once)
Based on code?|Yes|Yes=Spark Streaming and Flink; No=NiFi (drag and drop)

## Kafka Streams Demo: WordCount

Kafka has a built-in demo class that allows us to test Kafka Streams.
In order to test it, the following steps must be followed:

1. Create topics for input and output with a specific name

```bash
$ ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic \
    streams-plaintext-input --partitions 1 --replication-factor 1

$ ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic \
    streams-wordcount-output --partitions 1 --replication-factor 1
```

2. Send some data to the input topic

```bash
$ ./bin/kafka-console-producer.sh --broker-list localhost:9092 \
    --topic streams-plaintext-input

> java
> is
> pretty
> yes
> you
> are
> java
```

3. Create a consumer to listen to the output topic

```bash
$ ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic \
    streams-wordcount-output --from-beginning --formatter \
    kafka.tools.DefaultMessageFormatter --property print.key=true --property \
    print.value=true --property \
    key.deserializer=org.apache.kafka.common.serialization.StringDeserializer \
    --property \
    value.deserializer=org.apache.kafka.common.serialization.LongDeserializer
```

4. And now, run the built-in demo Kafka Streams class:

```bash
$ ./bin/kafka-run-class.sh \
    org.apache.kafka.streams.examples.wordcount.WordCountDemo
```

You should see the following:

```
> java    1
> is      1  
> pretty  1
> yes     1
> you     1
> are     1
> java    2
```

Since it is a Stream, the word "java" will appear duplicated in the end,
with a count of 2.   

### What is the Topology for WordCount?

The idea here is to perform an aggregation of records, grouping them so
they can be count by the amount of duplicated words:

- **Stream** to create a stream: `<null, "Java pretty Java">`
- **MapValues** to lowercase: `<null, "java pretty java">`
- **FlatMapValue** split by space: `<null, "java">`, `<null, "pretty">`, `<null, "java">`
- **SelectKey** to apply a key: `<"java", "java">`, `<"pretty", "pretty">`, `<"java", "java">`
- **GroupByKey** before aggregation: (`<"java", "java">`), (`<"pretty", "pretty">`), (`<"java", "java">`)
- **Count** ocurrencies in each group: `<"java", 2>`, `<"pretty", 1>`
- **To** in order to write processed data back to Kafka

## How to Code Kafka Streams in Java?

There are two main ways:

- **High Level (DSL):** a more practical approach
- **Low Level Processor API:** a more detailed approach. It may be used to
write something more complex, but usually it is not necessary to adopt it.

### Kafka Streams Application Properties

Kafka Streams runs over Kafka Consumer and Kafka Producer, so it also uses
their properties. In addition, it defines your own set of properties:

- `application.id`: this property is for Kafka Streams only. It must have
the same value of group id of the consumer. It is defined by the constant
`org.apache.kafka.streams.StreamsConfig.APPLICATION_ID_CONFIG`.
- `bootstrap.server`: same as Kafka. It is defined by the constant
`org.apache.kafka.streams.StreamsConfig.BOOTSTRAP_SERVERS_CONFIG`.
- `auto.offset.reset`: This is a consumer property. use **earliest** (to
  consume topic from the beginning) or **latest**. It is defined by the
  constant
  `org.apache.kafka.clients.consumer.ConsumerConfig.AUTO_OFFSET_RESET_CONFIG`.
- `processing.guarantee`: Use **exactly_once** here, for obvious
behavior. It is defined by the constant
`org.apache.kafka.streams.StreamsConfig.StreamsConfig.PROCESSING_GUARANTEE_CONFIG`.
- `default.key.serde`: This is used for key serialization and deserialization.
It is defined by the constant
`org.apache.kafka.streams.StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG`.
For instance, it can assume the value
`org.apache.kafka.common.serialization.Serdes.StringSerde` _(default)_
- `default.value.serde`: This is used for value serialization and
deserialization. It is defined by the constant
`org.apache.kafka.streams.StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG`.
For instance, it can assume the value
`org.apache.kafka.common.serialization.Serdes.StringSerde` _(default)_

### Running as a JAR

Kafka Streams should be deployed as a `fat jar` (containing the code and its
dependencies), that can be created using a `maven assembly plugin`. The jar
articact must have a `main` method in order to run properly.


## KStreams and KTables

A Kafka Stream can be handled as a KStream or KTable.

### KStream

In a KStream, all data is INSERTED. It is similar to the log concept (data are
inserted in sequence). There is no end, so it have infinite size (always wait
for new messages to arrive).

The following table describes the representation of a topic using KStream:

--|--
Topic (key, value)|KStream
--|--
(alice, 18) | **(alice, 18)**
(bob, 22)   | (alice, 18)
            | **(bob, 22)**
(alice, 19) | (alice, 18)
            | (bob, 22)
            | **(alice, 19)**

As we can see, data is always appended to the end of a KStream, and there is
no relation between old and new data.

### KTable

KTable behaves more like a table. It is similar to the concept of log
compacted topics. UPDATE is performed in non-null data, and DELETE can happen
in null data.

The following table describes the representation of a topic using KTable:

--|--
Topic (key, value)|KTable
--|--
(alice, 18) | **(alice, 18)**
(bob, 22)   | (alice, 18)
            | **(bob, 22)**
(alice, 19) | **(alice, 19)**
            | (bob, 22)
(bob, null) | (alice, 19)

As we can see, data with a new key are inserted, data with the same key are
updated, and data with null value are deleted.

### KStream x KTable Usage

KStream should be used when:

- data is read from non-compacted topic
- if a new data is parcial and transactional

KTable should be used when:

- data is read from log compacted topic
- if a database-like structure is required (each update is self-sufficient)


## Stateless and Statefull Operations

**Stateless operations** are those who do not require knowledge of previous
operations.

**Statefull operations** are those who depends on previous operations - for
instance, to count messages based on aggregation rules considering previous
data.

### Stateless Operations

#### Select Key

This operation sets a new Key for a record. Record is marked for repartition
(since key was changed).

```java
// (null, "alice") -> ("alice", "alice")
rekeyed = stream.selectKey((k,v) -> v);
```

#### Map / MapValues

Gets one record and returns one record (1 to 1 transformation).

--|--
Map|MapValues
--|--
Applied to Keys and Values|Applied only to Values
Alter keys (trigger repartition)|Do not alter keys
Only for KStreams|For KStreams and KTables

```java
// (alice, "Java") -> ("alice", "JAVA")
uppercase = stream.mapValues(value -> value.toUpperCase());
```

#### Filter / FilterNot

Gets one record and returns zero or one record. A FilterNot is just the
logical inverse of a Filter.

Filter characteristics:

- Do not change keys and values
- Do not trigger repartition
- It can be used in KStream and KTable

```java
// ("alice", 1) -> ("alice", 1)
// ("alice", -1) -> record is discarded
KStream<String, Long> positives = stream.filter((k, v) -> v >= 0);
```

#### FlatMap / FlatMapValues

Gets one record and returns zero, one or more than one records (an iterable).

--|--
FlatMap|FlatMapValues
--|--
Change Keys|Do not change keys
Trigger repartition|Do not trigger repartition
Only for KStreams|Only for KStreams

```java
// ("alice", "I like coffee") -> ("alice", "I"), ("alice", "like"), ("alice", "coffee")
aliceWords = aliceSentence.flatMapValues(v -> Arrays.asList(v.split("\\s+")));
```

#### Branch

Branch is a very interesting concept. It splits a KStream based on
particular predicates.
- Predicates are evaluated in order. The first to match gets the record
- Records only goes to single, unique branch  
- If a record makes no match with any of the predicates, it is discarded

Consider the following branch structure:
```java
KStream<String, Long>[] branches = stream.branch(
  (k, v) -> v > 100,  // first predicate
  (k, v) -> v > 10,   // second predicate
  (k, v) -> v > 0     // third predicate
);
```

--|--|--|--
Data Stream|branches[0] (v>100)|branches[1] (v>10)|branches[2] (v>0)
--|--|--|--
(alice, 1)    | (alice, 300)  |  (alice, 30)  | (alice, 1)  
(alice, 30)   |               |  (alice, 60)  | (alice, 8)
(alice, 300)  |               |               |
(alice, 8)    |               |               |
(alice, -100) |               |               |
(alice, 60)   |               |               |

The record `(alice, -100)` was discarded from the Stream, since it do not match
any of the three predicates.


## References

- [Kafka: The Definitive Guide](https://www.confluent.io/resources/kafka-the-definitive-guide/)
- [Stephane Maarek's Kafka Courses @ Udemy](https://www.udemy.com/courses/search/?courseLabel=4556&q=stephane+maarek&sort=relevance&src=sac)
