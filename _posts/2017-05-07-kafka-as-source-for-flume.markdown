---
layout: post
title: Kafka as source for Flume
date: '2017-05-07 06:47:49'
---

We are re-platforming a [monolithic application](//en.wikipedia.org/wiki/Monolithic_application) into [microservices](//martinfowler.com/articles/microservices.html), on the transition phase the services will be added incrementally and the monolithic app will be retired once all services are in place. To achieve the goal, we had to set up a real-time event processing model for syncing the data store. 

[Kafka](//kafka.apache.org/intro) and Flume were the options proposed on architecture council discussions. Kafka is a general purpose pub-sub messaging system which offers strong durability, horizontal scalability and fault-tolerance. While [Flume](//flume.apache.org/) is a distributed system for collecting, aggregating and transfer large data from different sources to a centralized data store. Flume is more tightly integrated with the [Hadoop](//hadoop.apache.org/) ecosystem whereas Kafka is not, so the obvious choice was Kafka.

While doing [POC](//en.wikipedia.org/wiki/Proof_of_concept) on Kafka I set out to try Flume as both have overlapping features. Wanted something simple to get hands dirty on Flume, so decided to use Kafka as source of Flume, the sink will be a logger.



#### 1. Create a Flume configuration file that uses Kafka source to send data to logger sink


{% highlight bash  %}
flume1.sources=kafka-source-1
flume1.channels=memory-channel-1
flume1.sinks=log-sink-1

flume1.sources.kafka-source-1.type = org.apache.flume.source.kafka.KafkaSource
flume1.sources.kafka-source-1.zookeeperConnect = localhost:2181
flume1.sources.kafka-source-1.topic = kafkalog
flume1.sources.kafka-source-1.batchSize = 100

flume1.channels.memory-channel-1.type = memory

flume1.sinks.log-sink-1.type = logger

flume1.sources.kafka-source-1.channels = memory-channel-1
flume1.sinks.log-sink-1.channels = memory-channel-1
{% endhighlight %}

#### 2. Create a topic to stream data to logger

{% highlight bash  %}
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic kafkalog
{% endhighlight %}


#### 3. Start Kafka server

{% highlight bash  %}
bin/kafka-server-start.sh config/server.properties
{% endhighlight %}

#### 4. Run the Flume agent

{% highlight bash  %}
flume-ng agent --conf /usr/local/flume/conf --conf-file /usr/local/flume/conf/kafkalog.properties --name flume1 -Dflume.root.logger=INFO,console
{% endhighlight %}

#### 5. Push message to Kafka topic using producer

{% highlight bash  %}
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic kafkalog
{% endhighlight %}

