---
layout: post
title: "Kafka Schema Registry & REST Proxy Concepts"
description: >
  Review of Kafka Schema Registry & REST Proxy concepts in Apache Kafka.
categories: distributedarchitecture
tags: [core, kafka]
comments: true
image: /assets/img/blog/kafka/schema-registry-pattern.png
---
> Kafka is a distributed, resilient, fault tolerant streaming platform that
works with high data throughput.
In this page, the main concepts of Kafka Schema Registry & REST Proxy
technologies will be covered.
{:.lead}

- Table of Contents
{:toc}

## The Problem with Data Change and Duplication

Kafka receives and delivers **bytes**. It does not know what are in those
bytes - and it's important that it do not know it, since this is a guarantee of
performance improvement (by _zero copy optimization_).

But there are drawbacks in this approach. If the producer send some
**"bad data"** (for instance, rename some required field), consumer will break.
In order to avoid such scenario, it is necessary to adopt a **schema**, that is
verified over data in order to avoid things to break.

A Schema can be applied to message key, message value, or both.
{:.note}

In addition to adopt a schema, it is also important that this approach consider
data optimization, in a way that data could be separated from schema to avoid
duplication (and increase in payload size). Schema Registry can be be adopted
to solve this problem.

## Schema Registry Pattern

Schema Registry is a pattern that separates schema from data. In Kafka, it
means that a separate component stores the Schema, and only data is sent to
Kafka. So, both producers and consumers must be able to connect and talk to
both Kafka Cluster and Schema Registry.

![](/assets/img/blog/kafka/schema-registry-pattern.png)

Figure: Schema Registry Pattern.
{:.figcaption}

In this pattern, the payload size to the Kafka Cluster is optimized, because
the Schema is sent only to the Schema Registry (and not to Kafka), and only
the data (without schema) is sent to Kafka. The consumers and producers are
the ones who have to relate Schema with data (and validate/reject bad data
based on the schema, ensuring that bad data never reaches Kafka cluster).

In addition, a common data format for the messages must be adopted. This data
format must support schemas and evolutions of data, and must be lightweight.
[Apache Avro](https://avro.apache.org/) is a most used data format.

An Avro file has both Schema and Data. But in the Schema Registry adoption,
what happens is the following:

1. Avro serializer (in producer) splits schema and data as separate units
2. Producer send the schema to the Schema Registry and receives an ID (if it
  is a new schema)
3. Producer send data to Kafka. This ID goes in the metadata message
4. Consumer gets the message from Kafka.
5. Consumer uses the message ID to retrieve the schema from Schema Registry
6. Avro deserializer (in consumer) uses retrieved Schema to deserialize and
read the message

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
```

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
```

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
```

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
```

example data:
```json
{"md5": "1122334455667788"}
```

### Logical Types

Logical types are complements from primitive or complex types that aggregate
more specialization to the data. These kind of data is serialized according to
its "natural" type.

The use of logical types in JSON do not exclude the need for primitive/complex
type declaration (their "natural" types). Both must be declared, as below
described.
{:.note}

The most common logical types are the following:

type|complemented from|description
--|--|--
decimal|bytes|-
date|int|days since Unix Epoch Time (1/1/1970)
time-millis|long|milisseconds since midnight
timestamp-millis|long|millisseconds since Unix Epoch Time (most commonly used)

Example usage:

```json
{"name": "signup_ts", "type": "long", "logicalType": "timestamp-millis"}
```

### Avro and the Complex Case of Decimals

As known, float and double are binary data (for instance, `10001.10010001`).
So, they are _approximations_ of the real number, that cannot be precisely
represented.

In order to represent a exact number (in case of finances, for instance), one
must consider the usage of **decimal** datatype.

Decimals are adopted for precise result (such as financial data), and
floats/doubles are naturally used for scientific computations (less precise
than decimal, but much faster).
{:.note}

The decimal datatype in Avro is a `logical type`, whose natural type is
`byte`. This kind of value cannot be property represented in JSON. So, for
decimal datatypes, the best approach so far is to treat it as a String.

### Creating Avro Records in Java

There are three types of Avro Records in Java: `Generic`, `Reflection`, and
`Specific`.

#### Avro Generic Record

In a Generic Record, Schema is defined on-the-fly and a generic object is
created based on it, using a `GenericRecordBuilder`. It is a simple approach
that do not use any POJO for object modeling, so the lack of typing is present.  

```java
import java.io.File;
import java.io.IOException;
import org.apache.avro.Schema;
import org.apache.avro.Schema.Parser;
import org.apache.avro.file.DataFileReader;
import org.apache.avro.file.DataFileWriter;
import org.apache.avro.generic.GenericData.Record;
import org.apache.avro.generic.GenericDatumReader;
import org.apache.avro.generic.GenericDatumWriter;
import org.apache.avro.generic.GenericRecordBuilder;

public class GenericRecordExample {

  public static void main(String[] args) {
    // 0: define schema
    Schema schema = new Parser().parse("{\n"
        + "     \"type\": \"record\",\n"
        + "     \"namespace\": \"com.example\",\n"
        + "     \"name\": \"Customer\",\n"
        + "     \"doc\": \"Avro Schema for our Customer\",     \n"
        + "     \"fields\": [\n"
        + "       { \"name\": \"first_name\", \"type\": \"string\", \"doc\": \"First Name of Customer\" },\n"
        + "       { \"name\": \"last_name\", \"type\": \"string\", \"doc\": \"Last Name of Customer\" },\n"
        + "       { \"name\": \"age\", \"type\": \"int\", \"doc\": \"Age at the time of registration\" },\n"
        + "       { \"name\": \"height\", \"type\": \"float\", \"doc\": \"Height at the time of registration in cm\" },\n"
        + "       { \"name\": \"weight\", \"type\": \"float\", \"doc\": \"Weight at the time of registration in kg\" },\n"
        + "       { \"name\": \"automated_email\", \"type\": \"boolean\", \"default\": true, \"doc\": \"Field indicating if the user is enrolled in marketing emails\" }\n"
        + "     ]\n"
        + "}");

    // 1: create generic record
    GenericRecordBuilder customerBuilder = new GenericRecordBuilder(schema);
    customerBuilder.set("first_name", "John");
    customerBuilder.set("last_name", "Doe");
    customerBuilder.set("age", 25);
    customerBuilder.set("height", 170f);
    customerBuilder.set("weight", 80.5f);
    customerBuilder.set("automated_email", false);
    Record customer = customerBuilder.build();
    System.out.println(customer);

    GenericRecordBuilder customerWithDefaultsBuilder = new GenericRecordBuilder(schema);
    customerWithDefaultsBuilder.set("first_name", "John");
    customerWithDefaultsBuilder.set("last_name", "Doe");
    customerWithDefaultsBuilder.set("age", 25);
    customerWithDefaultsBuilder.set("height", 170f);
    customerWithDefaultsBuilder.set("weight", 80.5f);
    Record customerWithDefault = customerWithDefaultsBuilder.build();
    System.out.println(customerWithDefault);

    // 2: write generic record to a file
    GenericDatumWriter<Record> datumWriter = new GenericDatumWriter<>(schema);
    try (DataFileWriter<Record> dataFileWriter = new DataFileWriter<>(datumWriter)) {
      dataFileWriter.create(schema, new File("customer-generic.avro"));
      dataFileWriter.append(customer);
      dataFileWriter.append(customerWithDefault);
    } catch (IOException e) {
      e.printStackTrace();
    }

    // 3: read generic record from a file
    File file = new File("customer-generic.avro");
    GenericDatumReader<Record> datumReader = new GenericDatumReader<>(schema);
    try (DataFileReader<Record> dataFileReader = new DataFileReader<>(file, datumReader)) {
      // 4: interprete as a generic record
      while (dataFileReader.hasNext()) {
        Record next = dataFileReader.next();
        System.out.println("Full record: " + next.toString());
        System.out.println("First name: " + next.get("first_name"));
        System.out.println("Non existing field: " + next.get("not-here!"));
      }
    } catch (IOException e) {
      e.printStackTrace();
    }
  }
}
```

#### Avro Reflection Record

In this approach, a Schema can be derived from a POJO. This POJO must have a
public constructor with no parameters. It uses a `ReflectData` class in order
to "extract" the Schema.

Some POJO file could described as the following:

```java
import org.apache.avro.reflect.Nullable;

public class ReflectedCustomer {

    private String firstName;
    private String lastName;
    @Nullable
    private String nickName;

    // needed by the reflection
    public ReflectedCustomer(){}

    public ReflectedCustomer(String firstName, String lastName, String nickName) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.nickName = nickName;
    }

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String fullName(){
        return this.firstName + " " + this.lastName + " " + this.nickName;
    }

    public String getNickName() {
        return nickName;
    }

    public void setNickName(String nickName) {
        this.nickName = nickName;
    }
}
```

Below, there is the code that generates Avro file based on reflection of the
previously defined POJO:

```java
import java.io.File;
import java.io.IOException;
import org.apache.avro.Schema;
import org.apache.avro.file.CodecFactory;
import org.apache.avro.file.DataFileReader;
import org.apache.avro.file.DataFileWriter;
import org.apache.avro.io.DatumReader;
import org.apache.avro.io.DatumWriter;
import org.apache.avro.reflect.ReflectData;
import org.apache.avro.reflect.ReflectDatumReader;
import org.apache.avro.reflect.ReflectDatumWriter;

public class ReflectRecordExamples {

  public static void main(String[] args) {

    // here we use reflection to determine the schema
    Schema schema = ReflectData.get().getSchema(ReflectedCustomer.class);
    System.out.println("schema = " + schema.toString(true));

    // create a file of ReflectedCustomers
    try {
      System.out.println("Writing customer-reflected.avro");
      File file = new File("customer-reflected.avro");
      DatumWriter<ReflectedCustomer> writer = new ReflectDatumWriter<>(ReflectedCustomer.class);
      DataFileWriter<ReflectedCustomer> out = new DataFileWriter<>(writer)
          .setCodec(CodecFactory.deflateCodec(9))
          .create(schema, file);

      out.append(new ReflectedCustomer("Bill", "Clark", "The Rocket"));
      out.close();
    } catch (IOException e) {
      e.printStackTrace();
    }

    // read from an avro into our Reflected class
    // open a file of ReflectedCustomers
    try {
      System.out.println("Reading customer-reflected.avro");
      File file = new File("customer-reflected.avro");
      DatumReader<ReflectedCustomer> reader = new ReflectDatumReader<>(ReflectedCustomer.class);
      DataFileReader<ReflectedCustomer> in = new DataFileReader<>(file, reader);

      // read ReflectedCustomers from the file & print them as JSON
      for (ReflectedCustomer reflectedCustomer : in) {
        System.out.println(reflectedCustomer.fullName());
      }
      // close the input file
      in.close();
    } catch (IOException e) {
      e.printStackTrace();
    }

  }
}
```

#### Avro Specific Record

This is a more elegant and sofisticated approach, in which a record Schema and
its Builder are both automatically created from a `.avsc` (Avro Schema) file.
This is different than previous methods, that are responsible for Schema
generation in code, either by parsing String (GenericRecord) or by using
Reflection ("ReflectRecord").

> A .avsc file is a common JSON file (text file), that describes the
schema of data according to [Avro Schema rules](https://avro.apache.org/docs/current/spec.html).

In order for this to work, Maven plugins must be configured (in the
_generate-sources_ Maven phase). They will read this `.avsc` resource file
and generate the related source code automatically in the `target` dir.

So, the resulting Java code (which is a representation of Avro Schema and a
Builder implementation of related POJO) can be directly referenced by our
production code in order to create a Avro Record without the need to manually
handle Schema generation.

In our `pom.xml` file, add the following plugins configuration:

```xml
<plugin>
  <groupId>org.apache.avro</groupId>
  <artifactId>avro-maven-plugin</artifactId>
  <version>${avro.version}</version>
  <executions>
    <execution>
      <phase>generate-sources</phase>
      <goals>
        <goal>schema</goal>
        <goal>protocol</goal>
        <goal>idl-protocol</goal>
      </goals>
      <configuration>
        <sourceDirectory>${project.basedir}/src/main/resources/avro</sourceDirectory>
        <stringType>String</stringType>
        <createSetters>false</createSetters>
        <enableDecimalLogicalType>true</enableDecimalLogicalType>
        <fieldVisibility>private</fieldVisibility>
      </configuration>
    </execution>
  </executions>
</plugin>
<!--force discovery of generated classes-->
<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>build-helper-maven-plugin</artifactId>
  <version>3.0.0</version>
  <executions>
    <execution>
      <id>add-source</id>
      <phase>generate-sources</phase>
      <goals>
        <goal>add-source</goal>
      </goals>
      <configuration>
        <sources>
          <source>target/generated-sources/avro</source>
        </sources>
      </configuration>
    </execution>
  </executions>
</plugin>
```

There should be a Avro file in `${project.basedir}/src/main/resources/avro`.
Let's call it `customer.avsc`, and its content could be as below:

```json
{
     "type": "record",
     "namespace": "com.example",
     "name": "Customer",
     "doc": "Avro Schema for our Customer",
     "fields": [
       { "name": "first_name", "type": "string", "doc": "First Name of Customer" },
       { "name": "last_name", "type": "string", "doc": "Last Name of Customer" },
       { "name": "age", "type": "int", "doc": "Age at the time of registration" },
       { "name": "height", "type": "float", "doc": "Height at the time of registration in cm" },
       { "name": "weight", "type": "float", "doc": "Weight at the time of registration in kg" },
       { "name": "automated_email", "type": "boolean", "default": true,
         "doc": "Field indicating if the user is enrolled in marketing emails" }
     ]
}
```

Finally, the following Java code demonstrates the creation of Avro file using
Specific Record approach:  

```java
import com.example.Customer;
import java.io.File;
import java.io.IOException;
import org.apache.avro.file.DataFileReader;
import org.apache.avro.file.DataFileWriter;
import org.apache.avro.io.DatumReader;
import org.apache.avro.io.DatumWriter;
import org.apache.avro.specific.SpecificDatumReader;
import org.apache.avro.specific.SpecificDatumWriter;

public class SpecificRecordExample {

  public static void main(String[] args) {
    // 1: create specific record
    Customer.Builder customerBuilder = Customer.newBuilder();
    customerBuilder.setFirstName("John");
    customerBuilder.setLastName("Doe");
    customerBuilder.setAge(25);
    customerBuilder.setHeight(170f);
    customerBuilder.setWeight(80.5f);
    customerBuilder.setAutomatedEmail(false);
    Customer customer = customerBuilder.build();
    System.out.println(customer);

    File file = new File("customer-specific.avro");

    // 2: write to file
    DatumWriter<Customer> datumWriter = new SpecificDatumWriter<>(customer.getSchema());
    try (DataFileWriter<Customer> dataFileWriter = new DataFileWriter<>(datumWriter)) {
      dataFileWriter.create(customer.getSchema(), file);
      dataFileWriter.append(customer);
    } catch (IOException e) {
      e.printStackTrace();
    }

    // 3: read from file
    DatumReader<Customer> datumReader = new SpecificDatumReader<>(customer.getSchema());
    try (DataFileReader<Customer> dataFileReader = new DataFileReader<>(file, datumReader)) {
      // 4: interpret
      while (dataFileReader.hasNext()) {
        Customer next = dataFileReader.next();
        System.out.println(next);
        System.out.println("Name: " + next.getFirstName());
      }
    } catch (IOException e) {
      e.printStackTrace();
    }

  }
}
```

### Reading Data from Avro file

Avro file have its format as binary by definition. So, a particular tool is
required in order to read this binary content.  

This tool is called `Avro-Tools`, and is presented as a JAR file that can be
directly downloaded from Maven Repo, as presented below:

```bash
$ wget https://repo1.maven.org/maven2/org/apache/avro/avro-tools/1.9.2/avro-tools-1.9.2.jar
```

With this Jar at hand, it's now possible to read content from an Avro file.

#### Reading Schema

```bash
$ java -jar avro-tools-1.9.2.jar getSchema <file>.avro
```

#### Reading Data (as JSON)

```bash
$ java -jar avro-tools-1.9.2.jar tojson --pretty <file>.avro
```

## Schema Evolution

Schema Evolution is the ability to support changes in the schema. For instance,
the following is a representation of a schema evolution `(v1 -> v2)`:

- **Schema (v1)**: firstName, lastName
- **Schema (v2)**: firstName, lastName, **email**

### Types of Schema Evolution

There are four types of Schema Evolution:

type|description|in depth
--|--|--
**Backward**|when a schema can be used to **read** data from `older schema`|is possible thanks to the `default` (new schema tries to read old data that has no value for a field, and then assume its default value for that field bacause it was not found)
**Forward**|when a schema can be used to **read** data from `newer schema`|is natural because new data read from an old schema will get no effect (new fields will be ignored). Delete fields in the new schema without `default` in the old schema is not forward compatible.
**Full**|both Forward and Backward|The best approach is to adopt this type; it is necessary to add fields with `default` values and only remove field (in newes schema) who have `default` values.
**Breaking**|neither Forward or Backward|Must avoid: **a)** Add/remove elements of a `Enum`; **b)** change the field type (i.e.: from string to int); **c)** rename a required, non-default field

Forward Compatibility change is the most common in Kafka. It is a common
scenario to start using a Producer with a newer Schema version, while the
consumers keep working with the same (previous) version (_a schema continues
to work by reading data from newer schemas_), and these consumers upgrade their
schema versions later on.

Backward Compatibility change is less common. In that case, one must wait for
all consumers to upgrade their Schema versions and, after that, new messages
can start to be produced using the newer schema (_a schema can be used to
read data from old schemas_).

> It is better, in general, to adopt a full compatibility approach, since it is
easy to do and brings a lot of benefits.


### General Rules to Write an Avro Schema

1. Make primary key `required`
2. Define `default` values for all field that must change somehow in the future
3. Try to avoid to use `Enums`, since enum evolution are not supported
4. Do not rename fields. Use `alias` instead
5. When evolving a schema, ALWAYS set `default` values
5. When evolving a schema, NEVER remove a `required` field


## Implementing a Schema Registry Pattern

[Confluent Schema Registry](https://docs.confluent.io/platform/current/schema-registry/index.html) is a implementation of a Schema Registry Pattern.
It is a [Confluent product](https://www.confluent.io/), a separated component from Kafka distribution.


### Producers and Consumers With Avro and Schema Registry

Using Confluent CLI tools, it is possible to produce and consume messages in
Avro format:

```bash
$ kafka-avro-console-producer --bootstrap-server localhost:9092 --topic \
  test-avro --property schema.registry.url=http://localhost:8081 --property \
  value.schema=\
  '{"type": "record", "name": "myrecord", "fields": [{"name": "f1", "type":"string"}]}'

$ kafka-avro-console-consumer --bootstrap-server localhost:9092 --topic \
  test-avro --property schema.registry.url=http://localhost:8081 \
  --from-beginning
```

The value of `schema.registry.url` property requires a http protocol to be
explicitly defined. If this property is not explicitly set here, Kafka will
look for it in the default file property (
`/etc/schema-registry/schema-registry.properties`).  

### Confluent Schema Registry Operations

Using REST API (_Confluent REST Proxy - more on that below on this page_), it
is possible to perform the following actions against schemas:

- Add Schemas
- Get Schemas
- Update Schemas
- Delete Schemas

## Confluent Rest Proxy

todo: fill it

## References

- [Kafka: The Definitive Guide](https://www.confluent.io/resources/kafka-the-definitive-guide/)
- [Stephane Maarek's Kafka Courses @ Udemy](https://www.udemy.com/courses/search/?courseLabel=4556&q=stephane+maarek&sort=relevance&src=sac)
- [Avro Specification](https://avro.apache.org/docs/current/spec.html)
