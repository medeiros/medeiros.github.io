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
the Broker. It just don't care. There is a risk of losing messages. It is
ideal in cases in which data loss is not critical (metrics and logs).
- `acks=1`: leader acknowledgment. This is the default. Client expects that
only the partition leader to send confirmation, and not cares about the
ISRs. It is more secure than `acks=0` in terms of data consistency, but
still can lose messages (if leader was down before message can be replicated)
- `acks=all`: all brokers' replicas of a topic (leader and ISRs) must confirm
that the message was received. This is the slowest method, but much more
secure in terms of data consistency, since no data is lost. This property
must be used together with `min.insync.replicas`.
  - `min.insync.replicas`: this is the minimal number of ISR (Leader and
  replicas) that must acknowledge the message in order to consider `all`.
  In fact, "all" is not the case (but this value instead). This property may
  be set in broker or topic level (topic overrides broker definition).
  A `NotEnoughReplicas` exception will be thrown if the number of replicas
  that acks the message is less than this parameter value.

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

## Retrying mechanism (to prevent messages to be lost)

In case of failures (no acks returned), a message need to be resend or will
be lost. Kafka Producers do this automatically - and the `retries` property is
important to define the behavior (the number of re-attempts to perform).

- Kafka <= 2.0: retries = 0 per default
- Kafka >= 2.1: retries = Integer.MAX_VALUE per default

Other properties in this regard are also important to consider:

- `retry.backoff.ms=100` (default): The time (in milisseconds) in which Kafka
Producer will wait til resend the message to Kafka Broker again. Since the
retry property if very big, it is also important to define a timeout.
- `delivery.timeout.ms=120000` (default): Time time (in milisseconds) until
timeout occurs and message is discarded. Default is two minutes. So, according
to the defaults, a message will be resend at each 100ms for two minutes
until Kafka Producer give it up. An exception will be thrown for the Producer
to catch, that will indicate that a message acknowledge could not be get
during the timeout period.
- `max.in.fligh.requests.per.connection`: This is the number of messages that
a `producer.send()` method sends in parallel. A default value is 5, but in the
case of retries, this may generate messages out of order. To ensure order,
one must set this property as 1, but throughput will then be compromised.
The best approach is to use `enable.idempotence=true`, that allows the default
value of 5 for this property and ensure ordering at the same time, by using
metadata information in the message header.

All this retrying mechanism is better handled if using the concept of
`Idempotent Producers`, previously described. If using `Idempotent Producer`,
your producer can be considered `safe`.

## Compression of messages

Is this concept, a bunch of messages is grouped together (as a batch)
in a producer before be send to the broker.
- Since each message has a body and a header, the approach to compress
messages in a batch of messages offers benefits in size, because now several
messages share the same header (instead of duplicate a header for each message).
The compression also helps when the body of a message is a big text (such as
JSON payloads, for instance).
- In terms of throughput, it also offers improvement, since reduces the number
of requests that have to be made to a bunch of messages reach the broker.
- The compressed message (a message with a bunch of messages) is saved
as-it-is in the broker, and only de-compressed in the consumer. The broker do
not know if a message is compressed or not.

Compression is more effective according to the number of messages being sent.

The property that defines compression is `compression.type` and can assume
the following values:

- `none`: default
- `gzip`: strong compression, but at a cost of more time and cpu cycles
- `lz4`:  less compression than gzip, but very fast
- `snappy`: offers a balance between gzip and lz4

### Compression Advantages

- `latency`: faster to transfer data in network
- `throughtput`: increased number of messages sent together per request
- `disk`: better utilization of storage

### Compression Disadvantages

- `CPU at clients`: producers and consumers will have to use cpu cycles in
order to compress and decompress data.

### Compression Overall

- Since distributed applications' number 1 issue is network, it is important
to compress messages, even if CPU usage will increase at client-side.
- Brokers do not suffer anything, with ou without compression, so that is
nothing to worry about in terms of any effect in the cluster.
- Adopt `lz4` or `snappy` compression types to the best balance between
compression ratio and speed

## Batch of messages

When using `max.in.flight.requests.per.connection=5`, Kafka producer will use
5 threads to send 5 different messages at a time. While the acks for those
message do not return, Kafka Producer will start batching the next messages.

This kind of "background batching" improves latency (messages reduced in size)
and throughtput (more messages per request are being sent).

The batch mechanism do not need any action in the broker side, and only need
to be configured on the producer side.

- `linger.ms` (default: 0): this is the number of milisseconds that Kafka will
wait until send data. During this time, messages are being grouped as a batch.
  - by adding a little delay (for instance, `linger.ms=5`), we can improve
  throughtput (more messages being sent per request), compression (since data
    is better compressed with more messages) and efficiency of producers

If `batch.size` is reached before `linger.ms` time has passed, the batch is
sent immediately.

![](/assets/img/blog/kafka/kafka-producer-batch.png)

Figure: Kafka Batch and Compression of messages
{:.figcaption}


## References

- [Kafka: The Definitive Guide](https://www.confluent.io/resources/kafka-the-definitive-guide/)
- [Stephane Maarek's Kafka Courses @ Udemy](https://www.udemy.com/courses/search/?courseLabel=4556&q=stephane+maarek&sort=relevance&src=sac)
- [Exploit Apache Kafka’s Message Format to Save Storage and Bandwidth](https://medium.com/swlh/exploit-apache-kafkas-message-format-to-save-storage-and-bandwidth-7e0c533edf26)
