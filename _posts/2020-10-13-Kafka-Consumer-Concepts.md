---
layout: post
title: "Kafka Consumer Concepts"
description: >
  Review of the consumer concepts in Apache Kafka.
categories: distributedarchitecture
tags: [core, kafka]
comments: true
---
> Kafka is a distributed, resilient, fault tolerant streaming platform that
works with high data throughput. In this page, the main concepts of Kafka
Consumer technology will be covered.
{:.lead}

- Table of Contents
{:toc}

## What is a Consumer?

- Consumers are Kafka clients, responsible to read data from Kafka Cluster
- They know automatically which Broker to connect to get the data they need,
because of the [Broker Discovery mechanism](../2020-10-09-Kafka-Core-Concepts#broker-discovery),
also used in Producers (other type of Kafka clients).
- Consumer know how to recover in case of Cluster failure: [Broker Discovery](../2020-10-09-Kafka-Core-Concepts#broker-discovery) will be applied after the
[cluster recovers itself](../2020-10-09-Kafka-Core-Concepts/#what-happens-if-a-broker-fails-rebalance-and-replication-of-data).
- Data is read in the order that was written in the partition.
- Consumer read data from different partitions in parallel, without any order
between partitions (the concept of order only exists in the messages from
the same partition)

## Consumer Group

Kafka group topic consumers in "consumer groups". This way, each consumer
collaborate to read data from a specific partition of a particular topic,
improving performance in reading of topic's data in parallel.

![](/assets/img/blog/kafka/kafka-consumer-groups.png)

Figure: Kafka Consumer Group mechanism.
{:.figcaption}

- Each consumer group can be seen as a distinct application, responsible to
read data from a particular topic (or more than one topic as well)
- If there are more consumers than partitions, some consumers will remain
inactive
- This is a internal Kafka Consumer mechanism, and nothing need to be done by
programmers in this regard

Kafka defines the relationship between Consumers and Consumer Groups x
Partitions using coordinator classes (GroupCoordinator and ConsumerCoordinator):

- When a consumer wants to join a group, it sends a `JoinGroup` request to
to group coordinator. If it is the first to do so, it becames a `leader` of
that group. A group leader then receives a list of group members (those who
send heartbeats to the server and are considered active) and became
responsible to assign a subset of partitions to those members (making sure
that no partition is assigned to more than one member, while it is still
possible for a member to handle more than one partition).
  - Only the leader has the list of members; each regular member knows nothing
  about other members in that regard
  - This process repeats every time a rebalance happens

In general, the **number of consumers in a group must be <= number of partitions**.
Otherwise, you will remain with inactive consumers - a waste of resources.
{:.note}

In other perspective, when creating partitions for a new topic, one must
consider the number of available consumers for that topic. If you only have
five available consumers, it should be better to create 5 partitions or less
in yout topic in terms of data reading.
{:.note}

## What if my consumers are not in a group?

Internally, each consumer is associate to a group in Kafka (even if not
directly explicited). If not defined to be part of a group, then your consumers
will be part of individual, internal and independent groups.

In this situation, each consumer will consume data independent from the others.

![](/assets/img/blog/kafka/kafka-consumer-groups-individual.png)

Figure: Kafka Consumer mechanism with implicit, individual groups for each
client.
{:.figcaption}

Each consumer read data from all partitions, independent from the others, in
duplication - behaving like separate applications.

## Consumer Offsets

Kafka consumers read data, and then they have to mark which data was already
read (advancing the offset). In order to do it, this index of already read data
is saved in a particular topic called `__consumer_offsets`.
- Actually, when a consumer from a consumer group reads data, it also writes
(commit) the offset of read data in the `__consumer_offsets` topic.

### Changing offsets of a group

Offsets position can be changed for a group, but all the consumers for that
group must be inactive when this happens. To do this offset manual changing,
use the `kafka-consumer-groups.sh` shell (more on that in the next sections).

### What about '--from-beginning' parameter?

The parameter `--from-beginning` is used to consume messages from a particular
position in a topic. If used, the data will be read from the oldest offset. If
not, data consumption starts from the latest offset.

However, the behavior is different with consumer groups.

If you are using kafka-console-consumer without any explicit group
- with `--from-beginning`: it will read data from the beginning of topic
- without `--from-beginning`: it will read only new data

But, if you're adopting `--group` parameter, data will always be read since
the last commit offset (and `--from-beginning` seems not to have the expected
behavior).

## Delivery Semantics for Consumers

- **at most once**: offsets are commited when the message is received by the
consumer. If processing of that message goes wrong, the message will be lost
(it won't be read again, since it is already marked as read). Not very adopted.

- **at least once**: offsets are commited as soon as data is processed. This
is the preferred delivery approach. If there is any issue during processing,
the message will be read again. In order to prevent duplicates when reading,
the process must be idempotent (read again a message should not affect the
system).

- **exactly once**: only achieved from Kafka to Kafka workflows (using Kafka
  Streams, for instance).

## Consumer Poll Behavior

Kafka consumer adopts a "poll" model (instead of a push model, which is adopted
by many messaging systems). In a poll model, one have better control over
consumption, speed and even replay of events.

![](/assets/img/blog/kafka/kafka-consumer-poll-broker.png)

Figure: Kafka Consumer Poll and Broker.
{:.figcaption}

### Importart throughput properties

Some importart properties that can improve you throughput is below listed. If
there is no need to improve throughput, those can remain as defaults:

- `fetch.min.bytes` (default: 1): control how many data can be send in a
request. Help improve throughput and reduce number of requests, but decrease
latency
- `max.poll.records` (default: 500): control how many messages can be received
by poll request. You can increase this number if your messages are small and
there is a lot of available RAM. It is also good to monitor how many records
are actually being polled by request (if the number is always 500), this
property value must be increase to improve throughput
- `max.partitions.fetch.bytes` (default 1MB): maximum data returned per
partition. If there is a lot of partition to be read, you'll have a lot of RAM
- `fetch.max.bytes` (default: 50MB): maximum data returned per each fetch
request (between multiple partitions). Consumer can perform multiple fetches
in parallel for different partitions

## Consumer Offset: Commit Strategies

There are two main ways to commit offsets in a consumer:

- `enable.auto.commit=true` (default) and processing batches synchronously
  - by using this property, offsets will be automatically commited in a
  period of time defined by `auto.commit.interval.ms` (default: 5000, or 5s)),
  every time `poll()` method is called
  - note that if rebalance happens between commit intervals, data will be
  processed again and may generate duplicates

```java
while (true) {
  List<Records> batch = consumer.poll(Duration.ofMillis(100));
  doSomethingSynchronouslyWith(batch);
}
```

- `enable.auto.commit=false` and manual, synchronous commit of offsets
  - by using this property, you control how and when commits are made
  - for instance: accumulate messages in a buffer and then save those messages
  in a database and commit offset.

```java
while (true) {
  batch += consumer.poll(Duration.ofMillis(100));
  if (isReady(batch)) {
    doSomethingSynchronouslyWith(batch);
    consumer.commitSync();
  }
}
```

## Consumer Offset: Reset Behaviors and Data Retention Period

The property `auto.offset.reset` defines the behavior of consumer when
start reading data from the topic:
- `earliest`: start to read data from the beginning of topic
- `latest`: start to read data from the end of topic

Offset data may also be lost even if `earliest` is set, because of the
`log.rentention.hours` property. If this property is set for 1 week (default)
and the consumer did not run in that period (for instance, it only runs after
10 days), data are no longer there to be consumed.   

## Replaying Data for Consumers

If is required to replay data, the approach is to reset the offsets for
thar particular consumer group.

All consumers must be inactive and the following command must be executed:

```bash
$ ./kafka-consumer-group.sh --bootstrap-server kafka1:9092 --group sometopic-g1
--reset-offsets --to-earliest --topic sometopic[:topicNumber] --execute
```

## Consumer Internal Threads

![](/assets/img/blog/kafka/kafka-consumer-poll-heartbeat.png)

Figure: Kafka Consumer Poll and Heartbeat.
{:.figcaption}

- All consumers in a group talk to a consumer coordinator
- The heartbeat mechanism will detect inactive consumers
  - If Kafka do not get a heartbeat signal during a specified period of time,
  it will consider that consumer inactive and will perform rebalance of
  partitions to the remainder consumers
- Consumers are encouraged to execute frequent polls and process data rapidly,
in order to avoid issues

The main properties in that regard are the following:

- `session.timeout.ms` (default: 10000, or 10s): this is the maximum period of
time that the broker will wait for heartbeats from consumer before consider
it inactive
- `heartbeat.interval.ms` (default: 30000, or 3s): this is the period of time
in which Kafka consumers will send heartbeats to the broker. The general rule
of thumb is to set this value as 1/3 of `session.timeout.ms`
- `max.poll.interval.ms` (defalt: 30000, or 5 minutes): This is the maximum
interval of time between two `poll()` calls, before to declare the consumer
as inactive.
  - This is particularly important to big data applications iteractions, like
  Spark, where processing could take time
  - This property is useful to detect some locked processing in the consumer
  side. In the case of Big Data, it may be necessary to increase this value
  or to improve big data processing time.

## CLI: important commands to know

### Consumer commands
```bash
# read new data from a particular topic
$ ./kafka/bin/kafka-console-consumer.sh --bootstrap-server kafka2:9092
--topic sometopic

# read data from a particular topic (from the beginning)
$ ./kafka/bin/kafka-console-consumer.sh --bootstrap-server kafka2:9092
--topic sometopic --from-beginning

# read new data from a particular topic (with keys)
$ ./kafka/bin/kafka-console-consumer.sh --bootstrap-server kafka2:9092
--topic sometopic --property print.key=true --property key.separator=,

# read new data from a particular topic (and saving it in a file)
$ ./kafka/bin/kafka-console-consumer.sh --bootstrap-server kafka2:9092
> sometopic_output.txt
```

### Consumer group commands
```bash
# read data from a topic using consumer groups
$ ./kafka/bin/kafka-console-consumer.sh --bootstrap-server kafka2:9092
--topic sometopic --group sometopic-g1

# list available consumer groups
$ ./kafka/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list

# describe a consumer group for all topics
# (here you can see offsets size, current position and lags)
$ ./kafka/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092
--group sometopic-g1 --describe

# reset the offset (to earliest) for a particular consumer group's topic
$ ./kafka/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092
--group sometopic-g1 --topic sometopic --reset-offsets --to-earliest --execute

# reset the offset (get back two positions) for a particular consumer group's
# topic
$ ./kafka/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092
--group sometopic-g1 --topic sometopic --reset-offsets --shift-by -2 --execute

# reset the offset (get back one position) for a particular consumer group's
# topic and partition
$ ./kafka/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092
--group sometopic-g1 --topic sometopic:0 --reset-offsets --shift-by -1 --execute

```

## References

- [Kafka: The Definitive Guide](https://www.confluent.io/resources/kafka-the-definitive-guide/)
- [Stephane Maarek's Kafka Courses @ Udemy](https://www.udemy.com/courses/search/?courseLabel=4556&q=stephane+maarek&sort=relevance&src=sac)
- [Exploit Apache Kafka’s Message Format to Save Storage and Bandwidth](https://medium.com/swlh/exploit-apache-kafkas-message-format-to-save-storage-and-bandwidth-7e0c533edf26)
