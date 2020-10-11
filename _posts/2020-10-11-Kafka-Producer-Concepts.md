---
layout: post
title: "Kafka Producer Concepts"
description: >
  Review of the producer concepts in Apache Kafka.
categories: distributedarchitecture
tags: [core, kafka]
comments: true
---
> Kafka is a distributed, resilient, fault tolerant streaming platform that
works with high data throughput. In this page, the main concepts of Kafka
Producer technology will be covered.
{:.lead}

- Table of Contents
{:toc}

## What is a Producer?

Producer is a specific type of Kafka client, responsible to send data to Kafka
Broker for writing.  

- Producers know to which partition and broker to send messages, because of the
metadata provided by the brokers.
- Producer is able to recovery automatically if broker fails, because of
the re-election of a leader, performed by Kafka Cluster. Producer will receive
metadata with the new leader definition for a topic - and this leader is the
one that will now receive the data.

In order to understand much of the basic concepts here described, it is
mandatory to understand Kafka Core concepts. Those concepts can be found
in details in the [Kafka Core Concepts'](../2020-10-09-Kafka-Core-Concepts)
page.

## Acknowledgments

When Kafka Producer send data to the broker, it must receive a confirmation
that the data was properly received. This is called _acknowledgment_.

The acknowledgement can be of the the following types:

- `acks=0`: no acknowledgment. Client do not expect any confirmation from
the Broker. It just don't care. There is a risk of losing messages.
- `acks=1`: leader acknowledgment. This is the default. Client expects that
only the partition leader to send confirmation, and not cares about the
ISRs. It is more secure than `acks=0` in terms of data consistency, but
still can lose messages (if leader was down before message can be replicated)
- `acks=all`: all brokers' replicas of a topic (leader and ISRs) must confirm
that the message was received. This is the slowest method, but much more
secure in terms of data consistency.

## Message Keys

- A message key can be string, number, object, etc - anything.
- If a message key is a `null` value, Producer's Round Robin will decide to
which partition to send data. If it was decided that message must go to
`Partition1`, Kafka Producer now check in metadata for who is the broker that
is the Leader of `Partition1`. Then, the data is directly sent to that broker'
partition.
- If a message key is a `non-null` value, then a Producer's `MurmurHash3`
algorithm will define to which partition to send that message, based on the
number of partitions of a topic and the key. After that, every message
that have the same key will go to the same partition (since the hash is
always the same for these constant attributes).

Kafka message keys is how Kafka ensures the order of messages in a topic, and
this is why is important to keep the same number of partitions for a topic
once defined.  

Since Kafka relates a particular key to a particular partition using hashing
algorithm (based on the number of partitions and message value), it is not
possible to link a key directly to a partition. If this is truly necessary,
it will be necessary to overwrite a `TopicPartition` class (to change the
internal behavior of Kafka in this matter).
{:.note}

## Idempotent Producer

In distributed applications, it's always possible that data will be duplicated
because of network issues. In Kafka, message duplication may happen like this:

![](/assets/img/blog/kafka/kafka-producer-idempotent.png)

Figure: Kafka messages' duplication scenario
{:.figcaption}

In order to prevent that, the concept of `Idempotent Producer` was introduced.
A `Idempotent Producer` ensures that only one sent message will be received
and accepted by the broker.

The concept of `Idempotent Producer` was implemented in Kafka 0.11. Now,
Kafka Broker will detect if a message is duplicated and do not commit it. For
this to work, both producer and broker must be at `version >= 0.11`, and the
property `enable.idempotence=true` must be set at client-side (producer).
Nothing is required at broker level, in this regard.

![](/assets/img/blog/kafka/kafka-producer-idempotent-2.png)

Figure: Kafka Idempotent Producer avoiding message duplication
{:.figcaption}

The definition of `enable.idempotence=true` only works if three other properties
are set accordingly:

- `retries > 0`: the maximum number of re-attempts
- `max.in.flight.requests.per.connection <= 5` (Kafka >= 1.0): the number of
requests a client will send in a single connection before blocking. Kafka can
ensures high performance and still keep the order even if in parallel.
More information can be found in [KIP-5494](https://issues.apache.org/jira/browse/KAFKA-5494)
- `acks=all`: both Leader and ISR must ensure that the message is received.

If any of the previous properties are not explicitly set, the
`enable.idempotence=true` property will ensure suitable values for them, as
follows:
- `retries=<Integer.MAX_VALUE>`
- `max.in.flight.requests.per.connection=5`
- `acks=all`


## References

- [Kafka: The Definitive Guide](https://www.confluent.io/resources/kafka-the-definitive-guide/)
- [Stephane Maarek's Kafka Courses @ Udemy](https://www.udemy.com/courses/search/?courseLabel=4556&q=stephane+maarek&sort=relevance&src=sac)
