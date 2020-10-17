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

## Kafka Connect High-Level Overview

In a high overview, Kafka Connect interacts with Kafka in the following way:

![](/assets/img/blog/kafka/kafka-connect-overview.png)

Figure: Kafka Connnect Overview.
{:.figcaption}

As described in the previous diagram:

- step 1. Kafka Connector Source (running into a Cluster) get data from
source (external)
- step 2. Kafka Connector Source sends collected data to Kafka Cluster (acting
  like a producer in that regard)
- step 3a. Kafka client application (acting as consumer) read data from Kafka
Cluster
- step 3b. Kafka client application (now acting as producer) send processed
data to Kafka Cluster
- step 4. Kafka Connector Sink (running into a Cluster) read data from Kafka
Cluster (acting like a consumer in that regard)
- step 5. Kafka Connector Sink send read data into a sink (external)

## Workers

Connectors run inside processes called `Workers`. In those workers,
scalability is also supported.

- Each worker is an isolated, simple Java Process (JVM)
- Workers run Connectors (_each connector is class inside a `jar` file_)
- Worker runs Connectors' `Tasks` to perform its actions
  - A Task is linked with a connector configuration
  - A job configuration can be composed of several tasks
- A Worker can run in standalone mode or distributed mode

|--|--|
Standalone Worker|Distributed Worker
|--|--|
A single process run both connectors and tasks|Multiple workers run connectors and tasks
Configuration use `.properties` files|Configuration is performed by a REST API
Very easy to use; good for dev and test|Useful for production deployment
No fail tolerance, no scalability, hard to monitor|Easy to scale (only add new Workers), and fail tolerant (automatic rebalance in case of an inactive worker)

Distributed workers do not necessary need to run in a cluster environment.
For testing purposes, one may run several Workers in the same machine,
just starting different JVMs using different properties files.
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

## Running a Standalone Worker to Read a File and Send it to Kafka

Let's run a simple worker in standalone mode to read a text file (source) and
send it to the Kafka Broker. The Source Connector for a File Stream is native
to Kafka, so we don't need to import any jars for this example.

[Kafka's FileStreamSourceConnector Source code @ Github](https://github.com/apache/kafka/blob/trunk/connect/file/src/main/java/org/apache/kafka/connect/file/FileStreamSourceConnector.java)
{:.note}

First of all, it is necessary to create a properties file to configure this
connector.

Let's call it `file-standalone.properties`, with the following content:

```properties
# /home/daniel/data/kafka/connectors/file-standalone.properties

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

- The first parameter of shell call is a property file that define workers
characteristics. In that case, we're using Kafka Default file for Kafka
Connect in standalone mode
- The second parameter is the property file to our specific configuration of
Kafka Connect File Stream Source Connector. You can pass as many files after
the first one (the worker definition), and each file will be related to a
different Sink/Source Connector.

After that, we can consume this topic and see the results:

```bash
kafka $ ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 \
  --topic connect-source-file-topic --from-beginning

{"schema":{"type":"string","optional":false},"payload":"using
  'FileStreamSourceConnector' Kafka Connector Source to produce some data"}
{"schema":{"type":"string","optional":false},"payload":"another line produced"}
{"schema":{"type":"string","optional":false},"payload":"one more line"}
{"schema":{"type":"string","optional":false},"payload":"and thats ok for now"}
{"schema":{"type":"string","optional":false},"payload":"bye"}
{"schema":{"type":"string","optional":false},"payload":"hello"}
```

As we can notice, all file information from source file was written in Kafka
Broker as JSON (but our file is not formatted as JSON file). That is because
the default Kafka file for worker properties (that define JSON as converter
method) was used.

### About Key and Value Converters

So, our data is being written as JSON. To understand why this is happening,
let's consider again the call to Kafka Connect in standalone mode:
```bash
kafka $ ./bin/connect-standalone.sh ./config/connect-standalone.properties
  /home/daniel/data/kafka/connectors/file-standalone.properties
```
- `file-standalone.properties`: this is the file that was created to explain
to the worker that to do: a Kafka Source Connector must read our particular
text file and send it to a topic
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
that explains why our data is being written as JSON. Naturally, this can
be changed.

### Changing Conversion Method

If we want to change the data format from JSON to String, for instance, the
only required action is to change the value of both `key.converter` and
`value.converter` properties to a String converter
(`org.apache.kafka.connect.storage.StringConverter`).
- This properties can be changed in worker properties file, or defined in
the file-standalone properties (in this case, it will override worker
definition)
- After that, it is necessary to restart the broker or task. New data in the
file will start to be written as Strings (instead of JSON) in a topic

```bash
# /home/daniel/data/kafka/connectors/file-standalone.properties

# These are standard kafka connect parameters, need for ALL connectors
name=file-standalone
connector.class=org.apache.kafka.connect.file.FileStreamSourceConnector
tasks.max=1
# optional: override kafka worker definition
key.converter=org.apache.kafka.connect.storage.StringConverter
value.converter=org.apache.kafka.connect.storage.StringConverter
# FileStremSourceConnector's properties
file=/home/daniel/data/kafka/connectors/file-to-read.txt
topic=connect-source-file-topic
```

```bash
kafka$ ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 \
  --topic connect-source-file-topic --from-beginning

{"schema":{"type":"string","optional":false},"payload":"hello"}
{"schema":{"type":"string","optional":false},"payload":"using
  'FileStreamSourceConnector' to produce some data"}
{"schema":{"type":"string","optional":false},"payload":"another line produced"}
{"schema":{"type":"string","optional":false},"payload":"one more line"}
{"schema":{"type":"string","optional":false},"payload":"and thats ok for now"}
{"schema":{"type":"string","optional":false},"payload":"bye"}
{"schema":{"type":"string","optional":false},"payload":"and back"}
{"schema":{"type":"string","optional":false},"payload":"and forth"}
{"schema":{"type":"string","optional":false},"payload":"and again?"}
{"schema":{"type":"string","optional":false},"payload":"and again!"}
xaxaxa xamps    ## new data arrived as plain String
```

### When and how the file is reprocessed?

While the file do not change, it will not be processed again. Only new lines
will be processed in this connector. After addition, there are two ways to
make the new lines to be processed:

- stop and start the worker
- only restart the specific task related to that connector, using REST API:
```bash
$ curl -X POST localhost:8083/connectors/file-standalone/tasks/0/restart
```

## Available REST APIs for Kafka Connect

The following commands use `curl` for http requests and `jq` in order to parse
json for human-readable format.

```bash
# To get cluster information:
$ curl localhost:8083 | jq    # GET verb is default

# To get a list of components already installed into the cluster:
$ curl localhost:8083/connector-plugins | jq

# To list all connectors:
$ curl localhost:8083/connectors | jq  

# To get data about a single connector:
$ curl localhost:8083/connectors/<connector-name>  | jq

# To check properties of a single connector
$ curl localhost:8083/connectors/<connector-name>/config | jq

# To get data of a single connector tasks:
$ curl localhost:8083/connectors/<connector-name>/tasks | jq

# To get status of a single connector:
$ curl localhost:8083/connectors/<connector-name>/status  | jq

# To control the execution of a connector
$ curl -X PUT localhost:8083/connectors/<connector-name>/pause
$ curl -X PUT localhost:8083/connectors/<connector-name>/resume

# To delete a connector
$ curl -X DELETE localhost:8083/connectors/<connector-name>

# To create a new connector using inline json config
$ curl -X POST -H "Content-type: application/json" --data \
   '{"name": "connector-name", "config": { "connector.class":"FileStreamSourceConnector", "tasks.max":"1", "file":"somefile.txt", "topic":"sometopic" }}' \
    localhost:8083/connectors | jq

# To create a new connector using json file config
$ echo '{"name": "connector-name", "config": { "connector.class":"FileStreamSourceConnector", "tasks.max":"1", "file":"somefile.txt", "topic":"sometopic" }}' \
    | jq > config.json
$ curl -X POST -H "Content-type: application/json" --data @config.json \
    localhost:8083/connectors | jq

# To update a connector configuration
$ curl -X PUT -H "Content-type: application/json" --data "<config as json>" \
    localhost:8083/connectors/<connector-name>/config

# To control the execution of specific task:
$ curl -X POST localhost:8083/connectors/<connector-name>/tasks/<task-id>/pause
$ curl -X POST localhost:8083/connectors/<connector-name>/tasks/<task-id>/restart
```

## Kafka Connect: Landoop

Landoop is a tool to make it easy to manage Kafka Connect workers.
- There is a docker image to make the usage simple (landoop/fast-data-dev)
- default URL: http://localhost:3030


## References

- [Kafka: The Definitive Guide](https://www.confluent.io/resources/kafka-the-definitive-guide/)
- [Stephane Maarek's Kafka Courses @ Udemy](https://www.udemy.com/courses/search/?courseLabel=4556&q=stephane+maarek&sort=relevance&src=sac)
