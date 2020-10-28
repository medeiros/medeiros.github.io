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
application is just a standard java application (jar) that runs in a JVM as an
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

Kafka Streams supports exactly once capabilities. This means it can guarantee
that each message will be processed only once and Kafka will process a message
only one time.

It can be guaranteed because both input and output systems are Kafka.

#### Common Problems Without Exacly Once

Lets consider the following flow of events:

- 1) Kafka Streams app get message from Broker (input topic)
- 2) Kafka Streams app send processed message back to Broker (output topic)
- 3) Kafka Streams app get ack from the Broker (output topic)
- 4) Kafka Streams app commit offsets related to the output topic

If the step 3 do not occur (messages are being processed but not confirmed to
the clients), the retry mechanism will make clients to send the messages
again. Kafka Broker will get duplicated messages.

If the step 4 do not occur (data is saved in the output topic and ack by
the clients, but the offset was not commited), other clients will consume
the same messages again (because they will read from the offset, which was
not updated).
- That could happen because offsets are commited from time to time - this is
not an atomic operation - and, in the meanwhile, network issues (for instance)
can prevent this commit to happen

There are cases where is acceptable to have duplicates (like page stats, or
logs). But in scenarios where precision is a requirements (financial
business, for instance), the Exacly Once semantics is sometimes mandatory.

#### Solving those problems with Exactly Once

Kafka consumers and producers can use the concept of idempotency. So,
duplicated messages are always identified and discarded.

In an advanced Kafka Streams API, it is possible to write messages in
different topics as a parte of a big transaction and only commit it if data
is saved in all topics.
{:.note}

In order to adopt exactly semantics in Kafka Streams, one must add the
following property:

```java
import org.apache.kafka.streams.StreamsConfig;

// StreamsConfig.PROCESSING_GUARANTEE_CONFIG="processing.guarantee"
// StreamsConfig.EXACTLY_ONCE="exactly_once"
properties.put(StreamsConfig.PROCESSING_GUARANTEE_CONFIG,
  StreamsConfig.EXACTLY_ONCE);
```

#### More Information

More details related to Exactly Once capabilities can be found
[on this Kafka article](https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/).

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

### Streams Repartition

Each operation that changes the key (`Map`, `FlatMap` and `SelectKey`)
triggers repartition. Once a Stream is marked to be repartitioned, it
only actually applies it when there is a real need (for instance, when a
`.count()` method is called).

Repartition is a Kafka Streams internal operation that performs some
read/write to Kafka Cluster, so there is a performance penalty.

- A topic with _"_repartition"_ suffix is created in Kafka to handle
repartitions

It is a good idea to only use the APIs that trigger repartition if there is
a actual need for key changing; and, otherwise, to adopt their value conterparts
(`MapValues` and `FlatMapValues`).
{:.note}

### KTables and Log Compaction

Log Compaction is topic attribute that improves performance (meaning less
I/O and less reads to get to the final state of a keyed data).

It can be used in cases where only the latest value for a given key is
required. Since this is the exactly situation of a KTable, it is right to
assume that each topic that uses KTable would be set as log-compacted.

KTable insert, update or delete records based on key value, so old records
with the same key will be discarded.



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


## Handling Messages: KStreams and KTables

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
compacted topics. UPDATE is performed in non-null values, and DELETE occurs
in messages with null values.

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

As we can see, data with a new key was inserted, data with the same key was
updated, and data with null value was deleted.

### KStream x KTable Usage

KStream should be used when:

- data is read from non-compacted topic
- if a new data is parcial and transactional

KTable should be used when:

- data is read from log compacted topic
- if a database-like structure is required (each update is self-sufficient)

### Read/Write Operations

#### Reading Data

Data can be read from a Topic in Kafka Streams using KTable, KStream or
GlobalKTable. The API is very similar, as presented below:

- `KStream<String, Long> wordCounts = builder.stream(Serdes.String(),
Serder.Long(), "some-topic");`
- `KTable<String, Long> wordCounts = builder.table(Serdes.String(),
Serder.Long(), "some-topic");`
- `GlobalKTable<String, Long> wordCounts = builder.globalTable(Serdes.String(),
Serder.Long(), "some-topic");`

#### Writing Data

Each KStream or KTable can be write back to Kafka topics.

When using KTable, consider log compacted mode in your related topic, so it
can reduce disk usage
{:.note}

Some operations related to data writing:

- `TO:` is a terminal, sink operation, used to write data in a topic.
  - `stream.to("output-topic");`
  - `table.to("output-topic");`
- `THROUGH:` it writes data to a topic and gets a strem/table of the same
topic. So, it is not a terminal operation.
  - `KStream<String, Long> newStream = stream.through("output-topic"); `
  - `KTable<String, Long> newTable = table.through("output-topic"); `

### The KStream/KTable Duality

Streams and tables are the same, but represented differently. So, they
can be related in particular ways.

#### Stream as Table

A KStream can be seen as a KTable changelog, where each record in a stream
captures a state change in a table.

#### Table as Stream

A table can be considered a snapshot, in a given period of time, of the latest
value for each key

#### Transformation: KTable to KStream

```java
KTable<String, Long> table = ...
KStream<String, Long> stream = table.toStream();

```

#### Transformation: KStream to KTable

First option: call an aggregated operation in the KStream; the result will
always be a KTable.

```java
KStream<String, Long> stream = ...
KStream<String, Long> table = stream.groupByKey().count();
```

Second option: Write a KStream in Kafka and read it later as a KTable

```java
KStream<String, Long> stream = ...
stream.to("some-topic");
KStream<String, Long> table = builder.table("some-topic");
```

In this case, it will read each record and perform based on the key and value
characteristics:
- if the key do not exists, the record will be inserted into the KTable
- if the key already exists and the value is non-null, the record will be
updated
- if the key already exists and the value is null, the record will be deleted


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
any of the three predicates. The other records were related to specific branches
based on the branch predicates. The order of predicates change the result, so
define the proper order of branch predicates is very important.

#### Peek

The `peek` method allow us to run an action that do not generates collateral
side effects in the stream (such as print data or collect stats) and then
to return the same stream for further processing.

```java
KStream<byte[], String> unmodifiedStream = stream.peek(
  (k, v) -> System.out.println("Key=" + key + "; value=" + value));
```


### Stateful Operations

#### GroupBy (KTable)

This is required to work with aggregations. It triggers repartition, because
key changes as a result of an aggregation.

```java
// KTable groupBy method returns KGroupedTable
KGroupedTable<String, Integer> groupedTable = table.groupBy(
  (k, v) -> KeyValue.pair(v, v.length()), Serdes.String(), Serdes.Integer());

// KStream groupBy method returns KGroupedStream
KGroupedStream<String, Integer> groupedStream = stream.groupBy(
  (k, v) -> KeyValue.pair(v, v.length()), Serdes.String(), Serdes.Integer());
```

#### KGroupedTable / KGroupedStream objects

The aggregation methods (`count`, `reduce` and `aggregate`) can only be
called in `KGroupedTable` or `KGroupedStream` objects. And these type of
objects are obtained only by the return of `groupBy` and `groupByKey` methods
(that can be called in both KStream and KTable, as above described).

So, ideally:
- `KGroupedStream gs = stream.groupBy(); gs.count() => X`
- `KGroupedTable gt = table.groupByKey(); gt.count() => N`

All of the aggregation methods (`count`, `reduce` and `aggregate`),
called in both `KGroupedTable` or `KGroupedStream` objects, always return
a `KTable`.
{:.note}

Considering group streams related to null data:

- `KGroupStream`:
  - Records with null data (for both keys and values) are ignored
- `KGroupTable`:
  - Records with null keys are ignored
  - Records with null values are treated like "delete"

##### Count

It just count the amount of records in the group, according to the null rules
previously explained.

##### Aggregate

In a `KGroupStream`:
- It requires a `initializer`, an `adder` (which has a key, a new value and
  an aggregated value), a `serde` and a `state-store name`.

```java
KTable<String, Long> aggregatedStream = groupedStream.aggregate(
  () -> 0L,
  (aggKey, newValue, aggValue) -> aggValue + newValue,
  Serdes.Long(),
  "aggregated-stream-store");
```
In the snippet above, `aggKey` type must match `aggregatedStream` key type
(`String`), `newValue` must match `aggregatedStream` value (`Long`) and
`aggValue` must match the `initializer` type (`Long`). The initializer and
the new value can be different (translation will happen in the adder).

In a `KGroupTable`:
- It requires a `initializer`, an `adder` (which has a key, a new value and
  an aggregated value), a `subtractor` (analogous behavior to the adder),
  a `serde` and a `state-store name`.

```java
KTable<String, Long> aggregatedStream = groupedStream.aggregate(
  () -> 0L,
  (aggKey, newValue, aggValue) -> aggValue + newValue,
  (aggKey, newValue, aggValue) -> aggValue - newValue,
  Serdes.Long(),
  "aggregated-stream-store");
```
The `subtractor` is used to treat the `delete` cases, previously explained.

##### Reduce

This is similar to aggregate, but the new and aggregated values must be of the
same type (`aggregate` can deal with different types). It is much simpler:
do not have both initializer and Serde.

It can work with just an `adder`:

```java
KTable<String, Long> aggregatedStream = groupedStream.reduce(
  (aggValue, newValue) -> aggValue + newValue,    // adder
  "aggregated-stream-store");
```

Or can also be used with both `adder` and `subtractor` (to treat delete, as
  previously explained):

```java
KTable<String, Long> aggregatedStream = groupedStream.reduce(
  (aggValue, newValue) -> aggValue + newValue,    // adder
  (aggValue, newValue) -> aggValue - newValue,    // subtractor
  "aggregated-stream-store");
```

##### KStream Transform/TransformValues

These applies a transformation in each record. It uses ProcessorAPI, which
is less common.
- Transform: triggers repartition, since transformation occurs with both
keys and values
- TrasnformValues: do not trigger repartition, since just transform values

## Use Cases and Examples

### WordCount Demo and Topology

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

In terms of topology, the idea here is to perform an aggregation of records
and grouping them, so they can be counted by the amount of duplicated words:

- **Stream** to create a stream: `<null, "Java pretty Java">`
- **MapValues** to lowercase: `<null, "java pretty java">`
- **FlatMapValue** split by space: `<null, "java">`, `<null, "pretty">`, `<null, "java">`
- **SelectKey** to apply a key: `<"java", "java">`, `<"pretty", "pretty">`, `<"java", "java">`
- **GroupByKey** before aggregation: (`<"java", "java">`), (`<"pretty", "pretty">`), (`<"java", "java">`)
- **Count** ocurrencies in each group: `<"java", 2>`, `<"pretty", 1>`
- **To** in order to write processed data back to Kafka

### Topology for 'Favorite Color' Problem

Let's consider the following requirements to a hipotetical problem:

- Data must be received in a format: (userId, color)
- It must accept only red, green and blue colors
- It must count the total of favorite colors from all users at the end
- The user's favorite color can change

What is the proper topology for this problem?

- **Stream** data
- **SelectKey** defined as userId
- **MapValue** per color (lowercase)
- **Filter** for acceptable colors only ("red", "green" and "blue")
- **To** topic with log compaction
  - that is because we want to count the latest color for all users, so we
  want to remove duplicates for ech user, maintaining only the last state.
  By sending data to a log compacted topic, Kafka will make sure that this
  will occurr as expected
- **KTable** read log compacted data
- **GroupBy** colors
  - the userId is not relevant anymore; grouping is being performed over
  the last color choose by each user; only colors should be counted, not users.
- **Count** grouped colors
- **To** final topic

## References

- [Kafka: The Definitive Guide](https://www.confluent.io/resources/kafka-the-definitive-guide/)
- [Stephane Maarek's Kafka Courses @ Udemy](https://www.udemy.com/courses/search/?courseLabel=4556&q=stephane+maarek&sort=relevance&src=sac)
- [Exactly-Once Semantics Are Possible: Here’s How Kafka Does It](https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/)
