---
layout: post
title: "An evolutionary, MVP approach example to Microservices - part 2: Front-end construction"
categories: journal
tags: [microservices,react]
comments: true
#image:
#  feature: mountains.jpg
#  teaser: mountains-teaser.jpg
#  credit: Death to Stock Photo
#  creditlink: ""
---

In this series of articles, my main intent is to exercise the usage of microservice technologies and related concepts. Guided by the MVP values (Maximum value product), the idea is to develop the less usable feature and start using as soon as possible - and then keep increasing it until it incorporates microservices architectural style as a whole.

It is a evolutionary design, although, which starts with a simple Spring Boot CRUD application and covers many technologies, from MongoDB to Spring Cloud.

## Moving on...
In the first part of this series, we created a CRUD REST back-end over a well-documented API. In this article, we are going to create the front-end to consume this back-end.

## Creating a new UI project

We could create the front-end code in the same project as the back-end project. But is my intent to keep these physically separated. So, let's create a new UI project. We also will adopt webpack and react tools. For webpack, you need to have Node.js installed.

### What is webpack?

answer that

### What is React?

answer that

### What about Node.js?

answer that

## Ok, let's move on...

To start project creation, execute the following line in linux bash shell:

```bash
$ mkdir financial-investment-ui && cd $_
```

This will create a project directory and get into the new directory.

create package.json

use npm install to download dependencies no packages.json on node_modules/.bin

and then use `Node.js` to install `Webpack`. The `node_modules` directory will be created, containing all required dpendencies.

use npm start to execute dev server

localhost:7070
