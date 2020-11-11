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

According to [Confluent page](https://www.confluent.io/product/ksql/), KSQL is
a streaming mechanism that enables real-time processing against Kafka.
It provides an intuitive SQL-like interface to manage stream processing, and
this approach avoids the burden of write Kafka Streams code (in Python of Java).
It allow us to perform a very broad range of operations, such as data filtering,
transformations, aggregations, joins, windows and session handling.

Kafka KSQL is not part of Apache Kafka distribution. You have to download
Confluent dist in order to have it.
{:.note}

## Push and Pull Queries

Kafka KSQL works with queries. In that regard, there are two types of queries
to consider and to understand:

- **Push Queries**: Query the state of the system in motion (real-time) and
continue to output results until they meet a LIMIT condition or are terminated
by the user.
  - These are the most common queries (and default from KSQL 5.3 onwards)
  - `EMIT CHANGES` clause is required for this type of query since KSQL 5.4
  onwards.
- **Pull Queries**: Query the current state of the system, return a result,
and terminate.
  - These are materialized aggregate tables, which are those created by a
  `CREATE TABLE AS SELECT <fields>, <agg functions> FROM <sources> GROUP BY <key>`
  kind of statement.
  - It is required to performs queries on it with rowkey

## How to Run

- Start Zookeeper and Kafka in Apache Kafka distribution, as usual
- Download [Confluent Kafka](https://www.confluent.io/download/) (required
  to run KSQL)
  - Please note that `confluent/etc/ksqldb/ksql-server.properties` file is
  configured to listen to port `8088` and to connect at Kafka Cluster in
  `0.0.0.0:9092`. Change this values right now if they are not suitable to
  your Kafka installation
- Execute the following commands in Confluent distribution directory:

```bash
$ cd ~/confluent  

# starting server
[confluent]$ ./bin/ksql-server-start ./etc/ksqldb/ksql-server.properties

# running CLI - inform http protocol or it will not work
[confluent]$ ./bin/ksql http://localhost:8088
```

## KSQL Commands

Somes things to consider regarding KSQL commands in CLI:

- KSQL commands and queries are case-insensitive
- All statements end with semicolon (;)

### Showing Topics and their Data

```ksql
# list existing topics
ksql> list topics;
ksql> show topics;

# print topic data
ksql> print <topicname>;
ksql> print <topicname> from beginning;
ksql> print <topicname> from beginning limit 2;
ksql> print <topicname> from beginning interval 2 limit 2;
```

### Creating a Stream

When creating a Stream, the `ROWTIME (bigint)` and `ROWKEY (varchar)` fields
will be implicitly created.

In order for the streams to be properly created, the related topics must
exist prior to the stream creation.
{:.note}

```ksql
ksql> create stream users_stream (name VARCHAR, countrycode VARCHAR)
  with (KAFKA_TOPIC='users', VALUE_FORMAT='DELIMITED');

ksql> create stream userprofile_stream (userid INT, firstname VARCHAR, lastname
  VARCHAR, countrycode VARCHAR, rating DOUBLE)
  with (KAFKA_TOPIC='userprofile', VALUE_FORMAT='JSON');
```

- "DELIMITED" means CSV data (comma-delimited values)
- Another supported value="JSON": in that case, JSON object fields are mapped
to stream fields
- Keyworks may be written in lowercase as well

### Showing the Details of new Streams

```ksql
ksql> list streams;
or
ksql> show streams;

ksql> describe 'users';           # to see stream fields and data types
ksql> describe extended 'users';  # to see stream in detail
```

### Select Query Stream

By default, print of data in topics and select queries adopt __latest__
consumption of data (so, only new data are shown). But this behavior can be
changed by using SET/UNSET directives, as below:

```ksql
ksql> set 'auto.offset.reset'='earliest';

ksql> select name, countrycode from users_stream
  emit changes;   

ksql> unset 'auto.offset.reset';
```

### Query Streams with Limits

```ksql
ksql> select * from users_stream emit changes limit 2;
```

Using limit, query is finalized when amount or records are reached. Without
it, this is a unbonded stream that only ends with CTRL+C.
- **auto.offset.reset='latest' + limit**: it only ends when new data arrives
until the defined limit; in this meantime, CLI remains waiting/blocking  
- **auto.offset.reset='earliest' + limit**: start reading messages and leaves
as soon as the limit is reached

### Aggregate Function: COUNT

```ksql
ksql> select countrycode, count(*) as countries_count from users_stream
  group by countrycode
  emit changes;
```

### Deleting a Stream

```ksql
ksql> drop stream if exists users_stream delete topic;
```

### Generating Synthetic Data

Usually, it's useful to generate synthetic data for testing purposes. KSQL has
a tool to do it, called `ksql-datagen`.

The basic syntax is the following:

```bash
[confluent]$ ./bin/ksql-datagen schema=path/to/.avro/file format=JSON \
  topic=topic_name key=topic_key_name iterations=10
```

`Iterations` refer to the maximum number of records to be generated.

### KSQL Build-In Functions

There are two type of KSQL built-in functions:

- **Scalar Functions**: get one or more values and returns one or more values
(map)
- **Aggregation Functions**: aggregate stream list values

### Creating a new Stream (from file) Based on Existing Stream

Let's consider the file `user-profile-pretty.sql`:

```sql
set 'auto.offset.reset'= 'earliest';

create stream user_profile_pretty as
  select firstname + ' ' + ucase(lastname) + ' from ' + countrycode + ' has
    a rating of ' + cast(rating as varchar) + ' stars. ' +
    case
      when rating < 2.5 then 'Poor'
      when rating between 2.5 and 4.2 then 'Good'
      else 'Excellent'
      end
    as description   
  from user_profile_stream emit changes;

unset 'auto.offset.reset';
```

This second stream (user_profile_pretty) remains bounded to the first
(user_profile_stream), with a running query; if the second stream is stopped,
the first stop to work, as expected.
{:.note}

To run this file from KSQL CLI, we can just run:

```ksql
ksql> run script '/path/to/file/user-profile-pretty.sql'
```

### Delete a Stream with a Running Query

The `user_profile_pretty` stream is running and we want to delete it. The
steps are below described:

```ksql
ksql> terminate <query ID>
ksql> drop stream if exists user_profile_pretty delete topic;
```

As pointed, query must be deleted before the stream.


## Tables in KSQL

Streams are a continuum of data in time. Tables, in the other hand, represent
the state of data right now.

Records relate with table two different ways:

- **add new records**: when there is no records with the same key
- **update existing records**: when new data has the same key as previous data

The `offset earliest` concept do not apply for tables, because each query
always brings the complete and final state for each key in general, regardless
of offset position.
{:.note}

## Joins

The Join mechanism is similiar from database, but with differences because of
the type of data (streams and tables).

Streams and Tables can be joined in the following way:

- Stream + Stream => Stream
- Table + Table => Table
- Stream + Table = Stream

### Join Requirements

In order to be joined, two objects (Stream or Table) must have the following
characteristics:

- Co-partitioned data:
  - Input records must have the same number of partitions at both sides
- Key must exists at both sides
- For Tables only:
  - Key must be varchar/string
  - Message key must have the same content as defined in Key column

### First Join

Joining a Stream and a Table (Stream + Table => Stream):

```ksql
select * from userprofile up
left join contrytable ct
on ct.countrycode = up.countrycode;
```

The command `describe extended` can be used to check this new stream details.



## References

- [Kafka: The Definitive Guide](https://www.confluent.io/resources/kafka-the-definitive-guide/)
- [Stephane Maarek's Kafka Courses @ Udemy](https://www.udemy.com/courses/search/?courseLabel=4556&q=stephane+maarek&sort=relevance&src=sac)
