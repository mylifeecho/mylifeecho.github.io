---
layout: post
title: "Writing your own elasticsearch plugin in Scala assembled by Gradle step-by-step"
description: "Elasticsearch is quite popular search engine, Scala is quite popular JVM language and Gradle is quite popular build tool. Let's try all of them at once." 
category: dev
tags: [ Scala, elasticsearch, Gradle ]
---
{% include JB/setup %}

According to wiki
> Elasticsearch is a search server based on Lucene. It provides a distributed, multitenant-capable full-text search engine with a RESTful web interface and schema-free JSON documents.

After more then one year of experience with elasticsearch, I can say it does work very well. Elasticsearch developers made very good job not only to develop high quality search database but also to keep it out of new features which are not necessary and be focused on critical and generic functionality. At the same time they provide very good plugin system to provide the way to extend elasticsearch for you needs when basic functionality is not enough for you. You can find a lot of different plugins on Github which monitor elasticsearch cluster, collect data from different sources like Twitter, RabbitMQ, MongoDB (so called "rivers"), Forsquare [published on github][4sq-score] plugin for custom geo-based scoring written in Scala. 


