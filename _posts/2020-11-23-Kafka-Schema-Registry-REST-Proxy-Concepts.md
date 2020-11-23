---
layout: post
title: "Kafka Schema Registry & REST Proxy Concepts"
description: >
  Review of Kafka Schema Registry & REST Proxy concepts in Apache Kafka.
categories: distributedarchitecture
tags: [core, kafka]
comments: true
#image: /assets/img/blog/kafka/kafka-ksql-basics.png
---
> Kafka is a distributed, resilient, fault tolerant streaming platform that
works with high data throughput.
In this page, the main concepts of Kafka Schema Registry & REST Proxy
technologies will be covered.
{:.lead}

- Table of Contents
{:toc}

## The Problem with Data Change

Kafka receives and delivers **bytes**. It does not know what are in those
bytes - and it's important that it do not know it, since this is a guarantee of
performance improvement (by _zero copy optimization_).

But there are drawbacks in this approach. If the producer send some
**"bad data"** (for instance, rename some required field), consumer will break.
In order to avoid such scenario, it is necessary to adopt a **schema**, that is
verified over data in order to avoid things to break.

So, Schema Registry was created to solve this problem.

## Schema Registry

Schema Registry is a separated component, responsible for validation of schema
between producers and consumers. So, both producers and consumers must be able
to connect and talk to it.

Schema Registry is also able to reject bad data (which means data that do not
comply with the desired schema).

Also, a common data format must be adopted. This data format must support
schemas and evolutions of data, and must be lightweight. Schema Registry
uses [Apache Avro](https://avro.apache.org/) as its offficial data format.

## Apache Avro and Data Format Comparison

The most basic data format is CSV. These is a are very simple and flexible
format, but it is fragile, since data types must be inferred.

So, data format evolved to tables. Those have a clear data structure for data
types, hence are better than CSV in that regard. However, tables are hard to
access (you have to adopt some sort of driver, and each database adopts a
different technology), and data must be flat.

JSON data format is much easier to access than tables, since it is a textual
format. It also support some data structures, such as arrays, integers, etc,
and data do not need to be flat (actually, complex data types must be modeled
in that format). It is also a very common data format nowadays. However, it is
verbose (there is a lot of duplication) and it does not have a schema.

Apache Avro is a schema written in JSON, and it have a payload (body) of data.
Advantages:
- **Data Type**: It is strongly typed
- **Less CPU usage**: data is compressed automatically
- **Schema**: schema goes along with the data
- **Documented**: documentation goes along with the schema
- **Binary Protocol**: data can be read between different languages
- **Schema evolution**: it supports evolution of schemas

Disadvantages:
- It is not supported by all languages
- Data is compressed and serialized, so it cannot be seen without specific tools

## Avro Characteristics

### Primitive Types

data type|description
--|--
null|no value
boolean|binary value (true/false)
int|32-bit signed integer
long|64 byte signed integer
float|single precision (32 bit) [IEEE 754](https://en.wikipedia.org/wiki/IEEE_754) floating point number
double|double precision (64 bit) [IEEE 754](https://en.wikipedia.org/wiki/IEEE_754) floating point number
bytes|sequence of 8 bit unsigned bytes
string|sequence of unicode characters

"Signed data" means that data has a "signal" (must be positive[+] or
negative[-]). Integer and float and signed data, and bytes area unsigned
data (since byte has no signal).
{:.note}

### Record Schemas

In Avro, record schemas are defined as JSON.
The most common **Schema data** are listed below:

Schema Data|Meaning
--|--
name|schema name
namespace|equivalent to Java `package`
doc|documentation that explain the schema
aliases|another (optional) names for the schema

There is also a **Fields** section, with information related to the field data:

Field Data|Meaning
--|--
name|field name
doc|documentation of the field (optional)
type|a primitive/complex type for data
default|a (optional) default value to the field

### Complex Types

Avro complex types are: records, enums, arrays, maps, union and fixed.

#### Records

Record is just a definition of a new schema inside a field.

schema:
```json
{
  "type": "record",
  "name": "LongList",                 
  "fields" : [
    {"name": "value", "type": "long"},
    {"name": "next", "type": ["null", "LongList"]}
  ]
}
```

#### Enums

Enum is a closed value domain, such as found in languages like `Java`. A
`enum` type is followed by a `symbols` attribute, that contains a array of
closed domain data.

schema:
```json
{"type": "enum", "name": "Medal", "symbols": ["Bronze", "Silver", "Gold"]}
```

example data:
```json
{"Medal": "Bronze"}
```

After defined, enum data cannot change, in order to maintain compatibility
with older and newer schema versions (more on that later on this page).
{:.note}

#### Arrays

These are a list of undefined items that share a same schema. A `array` type
is followed by a `items` attribute, that defines the data type for its items.

schema:
```json
{"name": "emails", "type": "array", "items": "string", "default": []}

example data:
```json
{"emails": ["someemail@gmail.com", "anotheremail@gmail.com"]}
```

#### Maps

Maps are a list of keys/values, with keys always defined as **String**. A `map`
type is followed by a `values` attribute, that defined the data type of values
in the map.

schema:
```json
{"name": "scores", "type": "map", "values": "long", "default": {}}

example data:
```json
{"scores": {"John": 9.5, "Mary": 9.8}}
```

#### Unions

It is not a complex data type per se, but a support for different data types
in the same field. If `default` attribute is being adopted, its value must be
of the type defined in the first item of the array.

The most common use case for unions is to defined a optional value, as below:

schema:
```json
{"name": "middle_name", "type": ["null", "string"], "default": null}

example data:
```json
{"first_name": "Joao", "middle_name": "Silva"}
{"first_name": "Maria"}
```

#### Fixed

A fixed number of 8 bit unsigned bytes. Types must be "fixed" and is supports
two attributes: `name` and `size`.

schema:
```json
{"type": "fixed", "name": "md5", "size": 16}

example data:
```json
{"md5": "1122334455667788"}
```

## References

- [Kafka: The Definitive Guide](https://www.confluent.io/resources/kafka-the-definitive-guide/)
- [Stephane Maarek's Kafka Courses @ Udemy](https://www.udemy.com/courses/search/?courseLabel=4556&q=stephane+maarek&sort=relevance&src=sac)
