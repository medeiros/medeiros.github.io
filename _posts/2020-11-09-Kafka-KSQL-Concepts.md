---
layout: post
title: "Kafka KSQL Concepts"
description: >
  Review of KSQL concepts in Apache Kafka.
categories: distributedarchitecture
tags: [core, kafka]
comments: true
image: /assets/img/blog/kafka/kafka-ksql-basics.png
---
> Kafka is a distributed, resilient, fault tolerant streaming platform that
works with high data throughput.
In this page, the main concepts of Kafka KSQL technology will be covered.
{:.lead}

- Table of Contents
{:toc}

## KSQL Overview

![](/assets/img/blog/kafka/kafka-ksql-basics.png)

Figure: KSQL Overview.
{:.figcaption}

## KSQL Commands to Show Topics and their Data

```ksql
# list existing topics
list topics;
show topics;

# print topic data
print <topicname>;
print <topicname> from beginning;
print <topicname> from beginning limit 2;
print <topicname> from beginning interval 2 limit 2;
```

## Creating a Stream

```ksql
create stream users_stream (name VARCHAR, countrycode VARCHAR)
with (KAFKA_TOPIC='users', VALUE_FORMAT='DELIMITED');
```

- "DELIMITED" means CSV data (comma-delimited values)
- Keyworks may be written in lowercase as well

## Showing the Details of new Streams

```ksql
list streams;
show streams;
describe 'users';  # to see stream fields
```

## Query Stream

By default, select queries adopt __latest__ consumption of data (so, only new
data are shown). But this behavior can be changed by using SET/UNSET directives,
as below:

```ksql
set 'auto.offset.reset'='earliest'

select name, countrycode from users_stream
  emit changes;   

unset 'auto.offset.reset'
```

The 'emit changes' clause is required in KSQL 5.4 onwards (for push queries).
{:.note}

## Query Streams with Limits

```ksql
select * from users_stream emit changes limit 2;
```

Using limit, query is finalized when amount or records are reached. Without
it, this is a unbonded stream that only ends with CTRL+C.
- **auto.offset.reset='latest' + limit**: it only ends when new data arrives
until the defined limit; in this meantime, CLI remains waiting/blocking  
- **auto.offset.reset='earliest' + limit**: start reading messages and leaves
as soon as the limit is reached

## Aggregate Function: COUNT

```ksql
select countrycode, count(*) from users_stream
group by countrycode
emit changes;
```

## Deleting a Stream

```ksql
drop stream if exists users_stream delete topic;
```


## References

- [Kafka: The Definitive Guide](https://www.confluent.io/resources/kafka-the-definitive-guide/)
- [Stephane Maarek's Kafka Courses @ Udemy](https://www.udemy.com/courses/search/?courseLabel=4556&q=stephane+maarek&sort=relevance&src=sac)
