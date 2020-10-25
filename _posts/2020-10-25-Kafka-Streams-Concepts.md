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

So, as we can see, Kafka Streams only iteracts to Kafka topics to get and send
data. This is different than [Kafka Connect](blog/distributedarchitecture/2020-10-15-Kafka-Connect-Concepts/),
that brings and send data to all kinds of different external systems.

## Kafka Streams Architecture: Simple and Flexible

Kafka Streams are very simple in terms of architecture. A Kafka Streams
application is just a standard java application (jar) that runs in a JVM as a
isolated process. The concept of "cluster" do not apply to it.

Since Kafka Streams inherits Kafka characteristics, it also can be considered
fault-tolerant and scalable.

Because of this simple architecture, it supports any application size - from
the smallest to the largest one.  

## Exactly Once Capabilities

Kafka Streams supports exactly once capabilities. This means it can handle
each message in a way that for each read message, there will be a write message,
every single time, without the hassle of reattempts and concerns about acks.
A simple Kafka Producer/Consumer (and other streams platform systems) can
only handle `at least once` and `at most once` capabilities, but Kafka Streams
can handle the ideal, `exactly once` capability.

## True Streaming: One Record at a Time

Kafka Streams process one record at a time. It is not a batch mechanism, like
Spark, but a stream mechanism that **process** and **transform** records, one
at a time.

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


## Where is Kafka Streams in a Overall Kafka Architecture?

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

You may see the following:

```
> java    1
> is      1  
> pretty  1
> yes     1
> you     1
> are     1
> java    2
```

Since it is a pure Stream, the word "java" will appear duplicated in the end,
with a count of 2.   

## References

- [Kafka: The Definitive Guide](https://www.confluent.io/resources/kafka-the-definitive-guide/)
- [Stephane Maarek's Kafka Courses @ Udemy](https://www.udemy.com/courses/search/?courseLabel=4556&q=stephane+maarek&sort=relevance&src=sac)
