---
layout: project
title: 'kafka-streams-covid19api'
caption: Kafka Stream application to summarize and rank Covid19 data for Brazil.
description: >
  The purpose of this application is to summarize a daily list of information
  regarding Covid-19 numbers across countries, ranking Brazil in that context.
  The metrics are: new cases, total cases, new deaths, total deaths, new
  recovered and total recovered. This is a Kafka Streams application. It gets
  its input JSON data from a topic that was previously loaded by ["kafka-connect-covid19api"](/projects/kafka-connect-covid19api/)
  Kafka connector, and delivers output JSON data to a topic that will be later
  sink to Twitter.
date: 14 May 2020
image: /assets/img/projects/kafka.png
links:
  - title: kafka-streams-covid19api
    url: https://github.com/medeiros/kafka-streams-covid19api
accent_color: '#4fb1ba'
accent_image:
  background: 'linear-gradient(to bottom, #0a7b81 0%, #01636e 25%, #02505b 50%, #073a4a 75%, #082e39 100%)'
  overlay:    true
sitemap: false
---
