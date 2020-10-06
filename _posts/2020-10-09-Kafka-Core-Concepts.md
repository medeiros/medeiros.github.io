---
layout: post
title: "Kafka Core Concepts"
description: >
  Review of the core concepts in Apache Kafka.
categories: distributedarchitecture
tags: [core, kafka]
comments: true
---
> Kafka is a distributed, resilient, fault tolerant streaming platform that works with high data throughput.
In this page, the main concepts of Kafka technology will be covered.
{:.lead}

- Table of Contents
{:toc}

## How it starts

This section is heavily based on the O'Reilly free book 'Kafka: The Definitive
Guide', that can be downloaded at [https://www.confluent.io/resources/kafka-the-definitive-guide/](https://www.confluent.io/resources/kafka-the-definitive-guide/).
{:.note}

At first, let's consider a simple messaging system for metrics gathering.

![](/assets/img/blog/kafka/kafkadefguide-messaging-system.png)

Figure 1: Simple Publisher Messaging System (from 'Kafka: The Definitive
Guide' Book)
{:.figcaption}

This is a simple solution for generic metrics. However, over time, this solution
increases as new requirements related to metrics arrive. New kinds of metrics
arise, along with new systems to consume those messages.

![](/assets/img/blog/kafka/kafkadefguide-messaging-system-2.png)

Figure 2: The Metrics Messaging System evolves (from 'Kafka: The Definitive
Guide' Book)
{:.figcaption}

Some technical debt is created now, because of all of those integrations out
of control. In order to organize all this, you can consider to centralize these
metrics in one central application.

![](/assets/img/blog/kafka/kafkadefguide-messaging-system-3.png)

Figure 3: Central System for Metrics Management (from 'Kafka: The Definitive
Guide' Book)
{:.figcaption}

Great. The architecture design is much better now. But imagine that, at the
same time, different initiatives are arising in your company, for different
kinds of messaging integration needs (instead of metrics, there is also demand
  for logging, tracking, and so on).

![](/assets/img/blog/kafka/kafkadefguide-messaging-system-4.png)

Figure 4: Multiple Publish/Subscribe Systems (from 'Kafka: The Definitive Guide'
Book)
{:.figcaption}

There is a lot a duplication in this architecture evolution, since this three
pub/sub systems have a lot of characteristics and features in common.

## The ideia behind Kafka

Apache Kafka is a system designed to solve this problem. Instead of having
different systems that handle messaging problems in isolation (with their own
bugs, scopes, schedules and so on), it would be better to have a single
centralized system that allows you to publish generic data, that can be used
by any sorts of clients. In that way, your team do not have to worry about
maintain a pub/sub messaging system for themselves - they can just use a
generic system, and focus on the specific problems of the team.

In Kafka model, different systems can produce data and consume data using
different kinds of technologies:

- Protocols: TCP, FTP, JDBC, REST, HTTP, etc
- Data Format: JSON, XML, CSV, Binary
- Data Schema

![](/assets/img/blog/kafka/kafkadefguide-messaging-system-5.png)

Figure 5: System can produce and consume data to and from Kafka.
{:.figcaption}

### Basic Terminology: Brokers, Producers and Consumers

As presented in the previous figure, Kafka have some particular names
for its main architecture components:

- **Broker:** is the same of a Kafka Server. A Kafka Cluster is a group
of brokers;
- **Producer:** is the client that produces data to Kafka Broker;
- **Consumer:** is the client that consumes data from Kafka Broker;

## Kafka main characteristics

- **Distributed:** Kafka works with several servers (or "brokers"), that form a
cluster
- **Resilient and Fault Tolerant:** If one broker fails, others can detected
and divide the extra load
- **Scale:** Kafka can scale horizontally to 100+ brokers
- **High Throughtput:** can reach mllions of messages per second
- **Low Latency:** can handle data traffic in less than 10 ms (realtime)

### Throughtput and Latency

It is important to define the concepts of throughtput and latency. These are
complimentary concepts, and they are key in Kafka:

- **Throughtput:** Define how many messages can be processed in a specific
amount of time. For instance: the broker can deliver 10,000 messages per
second
- **Latency:** Define how much time does it take for a single message to be
processed. For instance: a message can be produced and acknowledged in 5
milisseconds.

## Topics and Partitions in a Single Broker

A `topic` is defined as a stream of data. Internally, each topic is divided in
N `partitions`.

A `partition` is a ordered log of `messages`. It is called 'ordered' because
messages are stored in partitions in the order they arrive. Each `message` in a
`partition` have an ID, which is called `offset`.

This is how this concept looks like in a single broker:

![](/assets/img/blog/kafka/kafkadefguide-messaging-system-6.png)

Figure 6: Topics, Partitions and Offsets in a Single Broker.
{:.figcaption}

Each `topic` represents a concept of data. It can be something like metrics
of some sort, website activity tracking, event sourcing, log aggregation, etc.
There are a lot of [use cases](https://kafka.apache.org/uses) that can be
represented in a topic.

### Number of Partitions for Performance Improvement

In the previous example, a topic have three partitions. But it can be one
single partition, or dozens of partitions. So, it is important to understand
how many partitions a topic must have - and why.

The number of partitions in a topic is important for performance. With only
one partition, you cannot use multiple threads to read the data - if you do,
you'll read repetitions. But if you split your 10,000 messages in three
partitions, for instance, then you could read data in parallel (in three
threads), and this will improve your throughput.

This approach *(performance enhancement when reading data "splitted" in
different partitions)* will not work if your data must be read in the same
particular order that was written. Check the next section below
in this page (*'Distributions of Messages in a Topic'*) for further info.
{:.note}

### Distributions of Messages in a Topic

When data arrives to a topic that have more than one partition, Kafka must
decide in which partition to store it. The data can be write randomly in a
partition of the topic (what is called "*round robin*") or the same data can
always go to the same partition.

Each message in Kafka can have the basic structure:

- `value`
or
- `key` and `value`

If the `message` only have `value`, it will be written to any of the partitions
of the topic, according to round robin algorithm. But, if a `key` is also
informed, the algorithm [MurmurHash3](https://en.wikipedia.org/wiki/MurmurHash)
will be applied on the key to decide to which partition the message must be
stored.

So, if 10 different messages arrive in Kafka:
- **with the same key**: they will all be stored in the same topic, in the
order that they arrived. This is important if the requirement of the topic is
to store messages that must be later consumed in the same order that they
arrived (because the consumer of data can just read all data from a particular
partition, in order). The reading is performed the same way, by using message
key and [MurmurHash3](https://en.wikipedia.org/wiki/MurmurHash) algorithm.
- **without any key**: Kafka will use round robin to distribute these 10
messages through all of the partitions of the topic. In a topic with three
partitions, maybe 3 messages go to Partition0, another 3 Message to Partition1
and 4 messages go to Partition2.
- **with a mix (3 messages with same key, 7 messages without key)**: the 7
messages without key will be distributed randomly, and the remaining 3 messages
with the same key will go to the same partition. Each message is independent
from the others.
  - **This approach is not a good idea:** if your topic receives messages with
  and without key, you can have side effects - for instance, you may expect
  that only messages with key "truckID123" will be write to a particular
  partition ("all geolocation messages from truck 123"), but when you read that
  partition, some messages different that of this particular truck (for any
  arbitrary truck) also shows up. That is because Kafka will distribute
  messages without key to any of the topics of the partition - including those
  that are also being used to store data of particular key.
  - **The best approach** is to decide, at **topic level**, if
  **message key will be adopted or not**, and be consistent with it. Just do,
  or do not, as Yoda teached us.

  ![](/assets/img/blog/kafka/kafkacore-yoda-do-or-donot.png)

  Figure 7: In a topic, use message keys or do not use message keys.
  {:.figcaption}


Some notes regarding the logic of message distribution in partitions:
- The [MurmurHash3](https://en.wikipedia.org/wiki/MurmurHash) algorithm is
based on the number of partitions of a topic. So, if it's important to maintain
order (and, hence, you're using Message Keys for that), make sure that the
number of partitions of your topic do not change.
- It is also possible to overwrite the algorithm behavior (change
[MurmurHash3](https://en.wikipedia.org/wiki/MurmurHash) for
something else), by overwriting the Kafka `TopicPartition` class.
- You can also to explicit define which data goes to which partition of a topic,
when writing the producer client. You have to use the API for that.


## Topics and Partitions in a Cluster

If you have a single broken, all your data and availability relies on this
particular broker. This is not a good idea, because you have no scalability
and no fault tolerance. If this server goes down, you application is gone.
If the disk corrupts, it is also gone. A single broker could be nice for
development purposes, but a Kafka architecture must always be deployed as a
cluster.

### Advantages of Kafka Cluster

- **No data is lost**: The data is replicated through brokers per design
- **Fault tolerance and availability**: All brokers serve data as one. If one
goes down, the others will organize themselves to respond in its behalf. Kafka
can rebalance and redistribute the load with no harm.



### Replication of Partitions

In a Kafka Cluster, data can be replicated between brokers. This is one
of the most powerful concepts in Kafka.  

Let's consider the previous mentioned Trucks Positioning Topic:

![](/assets/img/blog/kafka/kafkadefguide-messaging-system-6.png)

Figure 8: Topics, Partitions and Offsets In a Single Broker - revisited.
{:.figcaption}

In a single broker, if it goes down or the disk is corrupted, this topic will
be gone. But let's consider this same topic using **replication factor of two**
with **three brokers**:

![](/assets/img/blog/kafka/kafka-cluster-broker-partitions.png)

Figure 9: Replication of Data of Topic between three Brokers in a Cluster.
{:.figcaption}

There are a lot of information in this figure.

#### The Same Topic goes to Multiple Brokers

Because of the Replication Factor of `2`, each partition of the topic
"Trucks Positioning" exists in `2` different Brokers of `3` available in
the cluster (ids: 0, 1 and 2).

So, it is correct to say that each topic can exist in different brokers, but
the data of a topic is not entirely in a single broker. For instance,
the partition `P2` is available only on Brokers `0` and `2` - but not in
Broker `1`.

If you want all the partitions of a topic to exist in all of the brokers of
a cluster, this can be achieve by setting the replication factor number equal
to the number of partitions - in this case, `3`. But this is not a good rule to
follow, because, if you cluster have 100 brokers, you do not necessarity wanna
have 100 partitions for a single topic. More on that in the next sections.
{:.note}

It is not possible to have a Replication Factor number higher than the number
of brokers - and there will always be a single replication factor, as a minimum.
For instance, consider **four brokers**: the replication factor must
be `>= 1 and <= 4`. And consider a **single broker**: the same rule applies
(replication factor `>= 1 and <= 1` => must be always `1`). If you try to
create a topic with the wrong number of replication factor, Kafka will throw
a `org.apache.kafka.common.errors.InvalidReplicationFactorException:
Replication factor: ? larger than available brokers: ?.`.
{:.note}


#### Leader Partition, Replicas and ISR

Kafka defines each partition in a topic as a **Leader Partition**, **Replica
Partition** or **ISR (In-Sync Replica) Partition**.

A **Leader Partition** is the entry point for all the clients. When some client
have to read data from partition `P1`, for instance, the following happens, in
a nutshell:
- Kafka Client asks for any kafka Broker for the metadata related to
the partitions of a topic
- Kafka Broker will get metadata from Zookeeper (the metadata from
partitions in a topic is stored in Zookeeper)
- Kafka Broker will deliver the updated metadata info to the client
- Kafka Client will search in metadata for the Broker who is the Leader of
Partition `P1` - in this case, it is Broker 1
- Kafka Client will ask to Broker `1` for data in the partition `P1`

When some client send data to be written to Kafka, a Partition Leader gets
data and is responsible to deliver to one of the Replicas (itself or other
replicas). When that particular data is in-sync, the partition is called a
`In-Sync Replica`.

A **Replica Partition** is just a replica of data. This is a copy of a Leader
Partition data, and this partition will not be directly called by any clients -
that is a characteristic only for the Leader. A Replica partition may or may
not be syncronized (or in-sync) with the Leader Partition.
- In Figure 9, the Partition `P1` in Broker `2` is a Replica which is not
in sync. The message with `offsetId 4` are still not synchronized.

A **In-Sync Replica** is a Replica Partition that is definitely in sync with
the Leader Partition. In the Figure 9, the Partitions `P2` in Broker `0` and
`P0` in Broker `1` are in sync.

This information (partition, leaders, replicas and isr) can be seen when we
describe a topic, as below:

```bash
ubuntu@ip-x:~$ ./kafka/bin/kafka-topics.sh --zookeeper zookeeper1:2181/kafka
  --topic trucks-positioning --describe
Topic: trucks-positioning    PartitionCount: 3       ReplicationFactor: 2    Configs:
        Topic: teste    Partition: 0    Leader: 0       Replicas: 0,1   Isr: 0,1
        Topic: teste    Partition: 1    Leader: 1       Replicas: 1,2   Isr: 2,1
        Topic: teste    Partition: 2    Leader: 2       Replicas: 0,2   Isr: 0,2
```

Some notes on this regard:
- A Partition Leader is also a Replica, but with a more important function. So,
in a topic of three partitions, for instance, there will be three Replicas,
but one of those replicas will be also a Leader.
- A particular partition can be Leader in a Broker and a Replica in another
Broker. That is OK by design.
- The sync mechanism takes some time, but is something that Kafka does
internally - a Replica Partition will always became in-sync, but may take some
time. More on that on Producers and Consumers section.

#### What Happens if a Broker Fails? Rebalance and Replication of Data

Kafka is able to detect that a Broker in a Cluster is no longer available,
because Kafka brokers exchange health check messages between each other.

When this happens, Kafka will perform a Leader Election considering the remainer
brokers. For each partition that was a Leader in the missing broker, Kafka
will elect a new Leader of that partition in a different (live) broker.
That way, partitions are still available to the clients.

For instance, considering the Figure 9, let's assume that Broker `1` is down.
The result can be seen in Figure 10 (*with side effects in green*):

![](/assets/img/blog/kafka/kafka-cluster-broker-partitions-goes-down.png)

Figure 10: Replication of data when a broker goes down.
{:.figcaption}

In summary, what happens is the following:  
- After some time, Kafka Brokers `0` and `2` realize that Broker `1` is
down and, therefore, Partition `P1` no longer a has a Leader
- A new Leader Election occur for `P1`. Since `P1` only exists in Broker `2`
(and not in Broker `0`), the elected Broker Leader for `P1` is now Broker `2`.
- In this scenario, the Partition `P2` still maintain in-sync replica of two
(because exists in Broker `0` and Broker `2`), but Partitions `P0` and `P1`
only exists in a single Broker (since Broker `1` is dead), and for those, the
in-sync replicas are now only one. Kafka will not rebalance data in order to
"force" anything - it will wait for Broker `1` to recover and will remain
with `2` replicas and `1` in sync replica for these.

This can be seen when describing the same topic again:

```bash
ubuntu@ip-x:~$ ./kafka/bin/kafka-topics.sh --zookeeper zookeeper1:2181/kafka
  --topic trucks-positioning --describe
Topic: trucks-positioning    PartitionCount: 3       ReplicationFactor: 2    Configs:
        Topic: teste    Partition: 0    Leader: 0       Replicas: 0,1   Isr: 0
        Topic: teste    Partition: 1    Leader: 2       Replicas: 1,2   Isr: 2
        Topic: teste    Partition: 2    Leader: 2       Replicas: 0,2   Isr: 0,2
```

The Leaders have changed to reference only alive brokers (`0` and `2`), the
replicas remain the same, and the ISRs references only alive brokers as well.

But this still works - meaning that Kafka can respond to the clients properly,
because we have a Leader for each partition of a topic.
But what if both Brokers `1` and `2` both die?

![](/assets/img/blog/kafka/kafka-cluster-broker-partitions-two-goes-down.png)

Figure 11: Two goes down - only one broker left.
{:.figcaption}

In that case, the Partition `P1` is not available anymore, anywhere.
Since there are no Leader for Partition `P1` anymore, producers and consumers
will crash when trying to send and get data to the topic.

When creating a topic, the number of active brokers must be equal or higher
than the replication factor number (for instance, if replication factor is
2, then you must have at least two brokers). But if one server goes down
after a topic creation, it will still be working if there is a Leader for each
partition (even if the number of brokers is less than replication factor).
The replicas will remain the same - the ISRs are the ones that will be updated
to the new situation.
{:.note}

```bash
ubuntu@ip-x:~$ ./kafka/bin/kafka-topics.sh --zookeeper zookeeper1:2181/kafka
  --topic trucks-positioning --describe
Topic: trucks-positioning    PartitionCount: 3       ReplicationFactor: 2    Configs:
        Topic: teste    Partition: 0    Leader: 0       Replicas: 0,1   Isr: 0
        Topic: teste    Partition: 1    Leader: none    Replicas: 1,2   Isr: 2
        Topic: teste    Partition: 2    Leader: 0       Replicas: 0,2   Isr: 0,2
```

There is no Leader for one partition, so this topic is not available to clients.

#### The Ideal Number of Replication Factor for a Topic

### How many servers to adopt?

## References

- [Kafka: The Definitive Guide](https://www.confluent.io/resources/kafka-the-definitive-guide/)
- [Stephane Maarek's Kafka Courses @ Udemy](https://www.udemy.com/courses/search/?courseLabel=4556&q=stephane+maarek&sort=relevance&src=sac)
- [MurmurHash3 Algorithm](https://en.wikipedia.org/wiki/MurmurHash)
