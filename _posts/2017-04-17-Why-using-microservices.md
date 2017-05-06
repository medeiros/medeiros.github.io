---
layout: post
title: "Why using microservices?"
categories: journal
tags: [microservices]
comments: true
#image:
  #feature: mountains.jpg
  #teaser: mountains-teaser.jpg
  #credit: Death to Stock Photo
  #creditlink: ""
---

Today, I watched the Sam Neil's video ["Why would you use microservices?"](https://www.oreilly.com/ideas/why-would-you-use-microservices). It's a very nice summarization of what microservices are and their value to adoption in organizations. Key points that I found particularly interesting are the following:

- Small teams, responsible for its own codebase (restricted to the team domain(s)), perform better and are happier;
- People are talking a lot about the [Conway Law](https://en.wikipedia.org/wiki/Conway%27s_law) in recent times. The idea that the architecture evolves with the organization, however, seems a little bit romantic in my opinion;

> organizations which design systems ... are constrained to produce designs which are copies of the communication structures of these organizations — M. Conway

- The good and old scenario of having to change a single line in a monolith and then stop and redeploy the entire application seems more and more absurd;
- The suggested book ["The Art of Scalability"](https://www.amazon.com.br/Art-Scalability-Architecture-Organizations-Enterprise/dp/0134032802) seems pretty much interesting, since is a very important topic in the regards of microservices archutectural style;
- The three axis presented (*"Horizontal Duplication"*, *"Data Partitioning"* and *"Functional Decomposition - Microservices"*) are a perfect concept. Those three items are recurrent problems when we continuosly have to deal when handling with modularization of different domains in a physical level.
  - In the regards of *"Functional Decomposition"*, it is nice to figure that DDD patterns (thinking of code in terms of business domain) *finally* finds its place. **Bounded Context** are one of the patterns to adopt in order to achieve functional decomposition. Hope people value **Eric Evans** work now.
- The idea that each part of the application can scale independently gives a lot of flexibility to infrastructure people (DevOps, if you can do it). Add *Docker* to all this and you have a lot of exciting and interesting material to study and work for several years!
  > It's a nice time to work in applications' infrastructure

- Adopt different technologies easier: this is also great. Thanks to API, there is flexibility to develop microservices using your preferred tools and technologies. For instance, it would be nice to get a bunch of enthusiastic, self-taught people to develop a specific Clojure/Datomic microservice to gather some banking account data without the hassle of hear something like: "this is not the official language of the company; we're not able to train all developers of the company to code in Clojure!"
  - It was nice and surprising that Sam talked in the video about Clojure in the same viewpoint I also consider (*"the next best thing"*) :D

## More information

This video is great. Besides it, there are also a lot of informative content regarding this subject that I have been gathering and studying. I liked a lot the very instructive Udemy course ["Microservices with Spring Cloud"](https://www.udemy.com/microservices-with-spring-cloud/learn/v4/), along with the readings: ["Building Microservices: Designing Fine-Grained Systems"](https://www.amazon.com/Building-Microservices-Designing-Fine-Grained-Systems/dp/1491950358) (from the same author of this video) (Amazon), ["Microservices in Production"](http://www.oreilly.com/programming/free/microservices-in-production.csp) from Susan Fowler (Free) and ["Migrating and ESB to a Cloud-Native Platform"](https://content.pivotal.io/white-papers/migrating-an-esb-to-a-cloud-native-platform) from Pivotal (Free White paper).

They are all very nice materials to start learning microservices.
