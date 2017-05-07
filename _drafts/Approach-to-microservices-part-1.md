---
layout: post
title: "An evolutionary, MVP approach example to Microservices - part 1: Initial Concepts"
categories: journal
tags: [microservices,spring boot]
comments: true
#image:
#  feature: mountains.jpg
#  teaser: mountains-teaser.jpg
#  credit: Death to Stock Photo
#  creditlink: ""
---

In this series of articles, my main intent is to exercise the usage of microservice technologies and related concepts. Guided by the MVP values (Maximum value product), the idea is to develop the less usable feature and start using as soon as possible - and then keep increasing it until it incorporates microservices architectural style as a whole.

It is a evolutionary design, although, which starts with a simple Spring Boot CRUD application and covers many technologies, from MongoDB to Spring Cloud.

## Requirements
This series of articles consider that you may have the following environment:
- Linux distribution (preferably Arch Linux)
- Java 8
- Gradle
- MongoDB

These are the application dependencies for development.

## The 'Personal finance investment' application

The idea is to implement an application to manage personal financial investments.

### Creating a Web Application

Let's start by creating a simple Spring Boot Web Application. in order to to so, we could use [Spring Initializr](https://start.spring.io/), which provides a web interface to create a quick application bootstrap. However, since I prefer the CLI, I'm goint to use Spring Boot CLI.

#### Installing Spring Boot CLI

In Arch Linux, we can use the [AUR files](https://aur.archlinux.org/cgit/aur.git/tree/?h=spring-boot-cli) to install Spring Boot CLI. We just have to download the plain files and install. The following bash script does all this job nicely:

```bash
$ TMP_INSTALL_DIR=~/tmp/spring-boot-cli-install
$ URLS="https://aur.archlinux.org/cgit/aur.git/plain/.SRCINFO?h=spring-boot-cli \
https://aur.archlinux.org/cgit/aur.git/plain/PKGBUILD?h=spring-boot-cli"

# create temp directory to keep installation files
$ mkdir -p $TMP_INSTALL_DIR && cd $_

# get AUR script files to generate package to Spring Boot CLI
$ for u in $URLS; do wget -O $(echo $u | awk -F'/' '{print $NF}' | $ awk -F'?' '{print $NR}') $u; done

# build application package to installation
$ makepkg -s

# install package generated that contain the 'any.pkg' in the same (such as spring-boot-cli-1.5.2-1-any.pkg.tar.xz)
$ sudo pacman --noconfirm -U $(ls -la | grep -i any.pkg | awk '{print $9}')

# remove temp installation
$ cd ~ && rm -r $TMP_INSTALL_DIR
$ URLS=
$ TMP_INSTALL_DIR=
```

After all that, the ``spring`` command is available for use.

#### Creating webapp by CLI

It is very easy to start a new project. There a a lot of possibilities, that can be checked with the ``spring init --list`` command.

In our case, let's create a gradle project, using java version 8 with the dependencies ``spring-boot-starter-web`` (web application), ``spring-boot-starter-thymeleaf`` (front-end templating engine),  ``spring-boot-starter-data-mongodb`` (MongoDB NoSQL database) and ``spring-data-rest-webmvc`` (REST capabilities).

```bash
$ spring init --build=gradle --java-version=1.8 --dependencies=web,thymeleaf,data-mongodb,data-rest --packaging=war --artifactId=variable-accounts --groupId=com.arneam.finandeiros --package-name=com.arneam.finandeiros.financialinvestment --name="Personal Financial Investment" --description="Web application to handle personal finance investments" financial-investment
```

That's it. The bootstrap application is now set and ready. You can load the project into IntelliJ and then you'll be ready to start coding.

### Creating the domain entities

Let's create a set of domain entities that represent the financial investment concept. In order to simplify our job, we're going to use ``Lombok`` library, to generate getters, setters and stuff like that. We also are going to use JSR-310 jackson library, to serialize JDK8 dates in JSON format (since our domain will have these kind of dates).

The fist action is to add the following lines to ```build.gradle``` file, in the ```dependencies``` section:

```groovy
compile('org.projectlombok:lombok:1.16.16')
compile("com.fasterxml.jackson.datatype:jackson-datatype-jsr310")
```

After that, we are free to create the following classes. Create a domain subpackage and then make sure your classes seems like the following:

```java
package com.arneam.finandeiros.financialinvestment.domain;

import lombok.Value;

@Value
public class Portfolio {
    private String name;
    private String description;
}
```

```java
package com.arneam.finandeiros.financialinvestment.domain;

import lombok.Value;

@Value
public class Fund {
    private String name;
}
```
The ```@Value``` annotation defines an immutable object: there are no setters and all attributes are initialized from the constructor. That's the idea with ```Portfolio``` and ```Fund``` classes, entities that do not have identity, and can therefore be treated as [Value Objects](https://martinfowler.com/bliki/ValueObject.html).

```java
package com.arneam.finandeiros.financialinvestment.domain;

import com.fasterxml.jackson.databind.annotation.JsonDeserialize;
import com.fasterxml.jackson.databind.annotation.JsonSerialize;
import com.fasterxml.jackson.datatype.jsr310.deser.LocalDateDeserializer;
import com.fasterxml.jackson.datatype.jsr310.ser.LocalDateSerializer;
import lombok.AccessLevel;
import lombok.Data;
import lombok.Getter;
import lombok.NonNull;
import lombok.Setter;
import org.springframework.data.annotation.Id;

import java.math.BigDecimal;
import java.time.LocalDate;

@Data
public class Movement {
    @Id
    @Getter(AccessLevel.NONE)
    @Setter(AccessLevel.NONE)
    private String id;

    @JsonDeserialize(using = LocalDateDeserializer.class)
    @JsonSerialize(using = LocalDateSerializer.class)
    @NonNull
    private LocalDate date;

    @NonNull private Portfolio portfolio;
    @NonNull private Fund fund;
    @NonNull private BigDecimal value;
    private String description;
}
```

The ```Movement'``` class has nullity validation in almost all its attributes (by setters and constructors), throwing NullPointerException to any attempt of set null (description attribute is not required). This is a good thing because antecipates any issue regarding data inconsistency, avoiding NullPointer exceptions to grow under control (classic problem). The class is annotated with ```@Data``` in order to be trated as a traditional [Entity](https://martinfowler.com/bliki/EvansClassification.html), whose internal data can be changed. The annotations in the id attribute define that it is an Id (unique identifier of database record) and that this attribute cannot have getter and setter (@Data annotation add getters and setters for all attributes, and we must override it to Id attribute). The @Json annotations define specific Jackson converters for JDK8 date format.

### Creating a CRUD functionality

Since the agregated root entity and its value objects are already created, the next step is to implement the CRUD functionality regarding this domain. These actions (create, read, update and delete) must be exposed through REST services.

This requirement is very simple to achieve with Spring Data Rest. We can have default implementations out-of-the-box for CRUD action, which is what we need right now.

The first step is to change the class ```PersonalFinancialInvestmentApplication```, adding the following content:

```java
package com.arneam.finandeiros.financialinvestment;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Import;
import org.springframework.data.mongodb.repository.config.EnableMongoRepositories;
import org.springframework.data.rest.core.config.RepositoryRestConfiguration;
import org.springframework.data.rest.webmvc.config.RepositoryRestConfigurerAdapter;
import org.springframework.data.rest.webmvc.config.RepositoryRestConfigurer;
import org.springframework.data.rest.webmvc.config.RepositoryRestMvcConfiguration;

@SpringBootApplication
@Import({RepositoryRestMvcConfiguration.class})
@EnableMongoRepositories
public class PersonalFinancialInvestmentApplication {

	public static void main(String[] args) {
		SpringApplication.run(PersonalFinancialInvestmentApplication.class, args);
	}

	@Bean
	public RepositoryRestConfigurer repositoryRestConfigurer() {
		return new RepositoryRestConfigurerAdapter() {
			@Override
			public void configureRepositoryRestConfiguration(RepositoryRestConfiguration config) {
				config.setBasePath("/api");
			}
		};
	}

}
```

The ```@Import``` annotation brings configuration classes to Spring Data and REST. The ```@EnableMongoRepositories``` is self-explanatory. The ```repositoryRestConfigurer``` method defines an entrypoint (```"/api"```) to those services.

Now that the application is properly configured, we need to create a repository interface for Spring Boot to construct the CRUD implementation. We have to create the ```MovementRepository``` interface, which respective code follows:

```java
package com.arneam.finandeiros.financialinvestment.domain;

import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.data.rest.core.annotation.RepositoryRestResource;

@RepositoryRestResource(path = "movement", collectionResourceRel="movement")
public interface MovementRepository extends MongoRepository<Movement, String>{

}
```

That's it. Now we have a CRUD REST API to Finance Investment domain, that must be validated.


### Testing CRUD with cURL

Let's start our testing. First of all, the Spring Boot application and MongoDB database need to be started.

```bash
#start mongodb
$ systemctl start mongodb

#start webapp
gradle clean
gradle bootRun
```

Spring Boot will inform that Tomcat is started on port 8080. Let's then check if the application is responding, executing the cURL command in linux bash:

```bash
curl -X GET --header 'Accept: application/json' 'http://localhost:8080/api/'
```

If the application is running properly, the following JSON will be returned:

```json
{
  "_links" : {
    "variable-account" : {
      "href" : "http://localhost:8080/api/movement{?page,size,sort}",
      "templated" : true
    },
    "profile" : {
      "href" : "http://localhost:8080/api/profile"
    }
  }
}
```

#### It would be nice to have this API documented

We know that the application is running, but we don't actually know the details of the API. To provide this, let's add Swagger to document this to us.

We need to add the following dependencies to build.gradle:

```groovy
compile('io.springfox:springfox-swagger2:2.6.1')
compile('io.springfox:springfox-swagger-ui:2.6.1')
compile('io.springfox:springfox-data-rest:2.6.1')
```

Then, we need to change the ```PersonalFinancialInvestmentApplication``` class to enable Swagger into the application:

```java
// ...
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.data.rest.configuration.SpringDataRestConfiguration;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;
// ...
@Import({RepositoryRestMvcConfiguration.class, SpringDataRestConfiguration.class})
// ...
@EnableSwagger2
public class PersonalFinancialInvestmentApplication {
// ...
  @Bean
  public Docket swaggerAPIDocumentation() {
    return new Docket(DocumentationType.SWAGGER_2)
      .select()
      .apis(RequestHandlerSelectors.any())
      .paths(PathSelectors.any()).build();
  }
// ...
```

After all that, restart the application. Access the url http://localhost:8080/swagger-ui.html#/Movement_Entity to check the API documentation. Now we have enough information to test the API.

#### Getting back to API testing ...

So lets explore the API according to CRUD actions and HTTP verbs. First, lets test the Create action, sending a POST request to the application, and then check the data load, sending a GET request to retrieve all data. We can use Swagger page to perform all these actions, but I'll use cURL to keep the CLI approach.

```bash
curl -X POST --header 'Content-Type: application/json' --header 'Accept: application/json' -d '{
   "date": "2017-05-07",
   "description": "",
   "fund": {
     "name": "TD NTNB 2045"
   },
   "portfolio": {
     "description": "My retirement savings",
     "name": "Retirement"
   },
   "value": 1200.00
 }' 'http://localhost:8080/api/movement'
```

If everything goes OK, the server will respond with data of this new resource (including its ID).

We can now test a GET to a specific element, by its ID.
Lets try to call the API, passing the recovered ID from previous step.

```bash
curl -X GET --header 'Accept: application/json' 'http://localhost:8080/api/movement/590e8d981cc93227062cd71c'
```

The server respond with the resource data, in JSON format.

Lets change the investment value:

```bash
curl -X PATCH --header 'Content-Type: application/json' --header 'Accept: application/json' -d '{
   "value": 1350.00
 }' 'http://localhost:8080/api/movement/590e8d981cc93227062cd71c'
```

We can see in the response that the value was correctly changed.

Let's now remove (with a DELETE action) and check if it was really removed (another GET request to get all data must be empty).

```bash
$ curl -X DELETE --header 'Accept: application/json' 'http://localhost:8080/api/movement/590e8d981cc93227062cd71c'
$ curl -X GET --header 'Accept: application/json' 'http://localhost:8080/api/movement/590e8d981cc93227062cd71c'
```

We can see that the resource no longer exists.

[This could also be tested automatically with Spring test. Lets consider this approach also (or before?). -- we can also print the output here, with http status codes for each situation]
