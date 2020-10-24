---
layout: post
title: "Non-Blocking IO and Tomcat"
description: >
  How Tomcat handles concurrent connections.
categories: distributedarchitecture
tags: [tomcat, web, nio]
comments: true
#image: /assets/img/blog/kafka/kafka-connect-overview.png
---
> This page discuss a little bit about blocking/non-blocking IO and Tomcat
in that context.
{:.lead}

- Table of Contents
{:toc}

## What is Blocking and Non-Blocking IO

Let's consider the relationship between request HTTP connections and server
threads allocated to process those connections.

In the first scenario, each request connection blocks a server thread, and
only releases it when the server returns processing. This is called a
**Blocking I/O** (since the thread is blocked to that connection until return).

The problem here is that we have a limited amount of threads to share with
our HTTP clients. So, let's say that we have 200 max threads to be shared. If
we have a low latency (< 10ms), it is probably not a issue (since each
thread is blocked for a very small period of time). But if you latency is about
two seconds and yout throughput is 500 requests/second, you have a problem.

In that scenario, in the first second, 200 clients get connection and are
waiting for the server to return. The other 300 clients get a
_"Connection Refused"_ and _"Server Unavailable" (503)_ messages, and never
get the connection.

So, let's consider **Non-Blocking I/O**. In this scenario, the server gets a
connection from the client, but do not block a thread. Instead, it holds the
connection but alternate between threads to process the request, until return.
In this scenario, more requests can be served.  

## How Tomcat Handles It

The default installation of Tomcat supports 200 simultaneous connections.
This number can be increased in the `maxThreads` attribute of a `Connector`
tag in the `server.xml` file.

However, Tomcat also supports Non-Blocking I/O. It is said that it supports
near 13,000 simultaneous requests in NIO mode.

Still, there is a limit. The `acceptCount` attribute defines the maximum
number of HTTP connections that will be put in a queue while there is no free
connection available to serve.


### Example

This tag is placed in `config/server.xml` file:

```html
<Connector URIEncoding="UTF-8" acceptCount="100" connectionTimeout="60000"
  blankmaxHttpHeaderSize="8192" maxKeepAliveRequests="5000" maxThreads="200"
  blankport="80" protocol="HTTP/1.1" redirectPort="8443"/>
```

In the example above, Tomcat is able to accept a maximum of `300` simultaneous
requests before start refusing connections, returning `HTTP Status Code 503`
messages. That is because, based on parameters, it can handle `200` requests
and put `100` requests in a queue while wait for others to complete. These
are default Tomcat values.
