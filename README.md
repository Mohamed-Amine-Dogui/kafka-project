# Kafka Quickstart Guide

This guide will walk you through the steps to get a Kafka cluster up and running, starting with Zookeeper and Kafka, followed by common Kafka topic operations and producer examples.

## Prerequisites

- Ensure you have Kafka installed. This guide assumes Kafka version 2.13-3.8.0.

## Starting the Services

### 1. Start Zookeeper

Zookeeper is required for managing Kafka brokers. Start the Zookeeper service by running the following command:

```bash
zookeeper-server-start.sh ~/kafka_2.13-3.8.0/config/zookeeper.properties
```

### 2. Start Kafka

Once Zookeeper is running, open a new terminal tab or window, and start the Kafka broker:

```bash
kafka-server-start.sh ~/kafka_2.13-3.8.0/config/server.properties
```

## Kafka Cluster is Running

After successfully starting Zookeeper and Kafka:

- Zookeeper has established a connection.
- Kafka is now listening for incoming connections on port `9092`.

Your Kafka cluster should now be up and running. You can proceed with creating topics, and producing and consuming messages.

---

## Kafka Topic Operations

Here are some common Kafka topic operations:

### 1. Create a Topic

To create a new topic called `first_topic` with 3 partitions and a replication factor of 1, use:

```bash
kafka-topics.sh --bootstrap-server localhost:9092 --topic first_topic --create --partitions 3 --replication-factor 1
```

### 2. List All Topics

To list all available topics in the Kafka cluster:

```bash
kafka-topics.sh --bootstrap-server localhost:9092 --list
```

### 3. Describe a Topic

To view details about a specific topic, such as partition count and replication factor:

```bash
kafka-topics.sh --bootstrap-server localhost:9092 --topic first_topic --describe
```

### 4. Delete a Topic

To delete a topic (this only works if `delete.topic.enable=true` is set in the Kafka server properties):

```bash
kafka-topics.sh --bootstrap-server localhost:9092 --topic first_topic --delete
```

---

## Kafka Producer Operations

Below are some examples of producing messages to Kafka topics.

### 1. Producing to a Topic

To start producing messages to the `first_topic`, use the following command:

```bash
kafka-console-producer.sh --bootstrap-server localhost:9092 --topic first_topic
```

Type your messages after the prompt (`>`), and use `Ctrl + C` to exit:

```plaintext
> Hello World
> My name is Amine
> I'm learning Kafka
> ^C
```

### 2. Producing with Acknowledgement Properties

You can configure producer properties such as `acks` to ensure message delivery. For example:

```bash
kafka-console-producer.sh --bootstrap-server localhost:9092 --topic first_topic --producer-property acks=all
```

Produce some messages:

```plaintext
> some message that is acked
> just for fun
> fun learning!
```

### 3. Producing to a Non-Existing Topic

If you produce to a topic that doesn't exist, Kafka will auto-create the topic if the configuration allows:

```bash
kafka-console-producer.sh --bootstrap-server localhost:9092 --topic new_topic
```

You may see some errors initially, but the topic `new_topic` will be created. By default, this topic will have only 1 partition:

```bash
kafka-topics.sh --bootstrap-server localhost:9092 --list
kafka-topics.sh --bootstrap-server localhost:9092 --topic new_topic --describe
```

To configure auto-creation with more partitions, edit `config/server.properties` or `config/kraft/server.properties`:

```plaintext
num.partitions=3
```

Now, producing to a new topic will create it with 3 partitions:

```bash
kafka-console-producer.sh --bootstrap-server localhost:9092 --topic new_topic_2
hello again!
```

Check the topic details:

```bash
kafka-topics.sh --bootstrap-server localhost:9092 --list
kafka-topics.sh --bootstrap-server localhost:9092 --topic new_topic_2 --describe
```

**Best Practice:** It's recommended to disable auto-creation of topics and manually create topics with the appropriate number of partitions before producing to them.

### 4. Producing with Keys

You can produce messages with keys, which allows you to route messages to specific partitions:

```bash
kafka-console-producer.sh --bootstrap-server localhost:9092 --topic first_topic --property parse.key=true --property key.separator=:
```

Example of producing messages with keys:

```plaintext
>name:Amine
>Age:33
```
---

## Kafka Consumer Operations

Here are some common Kafka consumer operations:

### 1. Create a Topic with Multiple Partitions

Before consuming, let’s create a topic called `second_topic` with 3 partitions:

```bash
kafka-topics.sh --bootstrap-server localhost:9092 --topic second_topic --create --partitions 3
```

### 2. Consuming Messages

To start consuming messages from `first_topic`:

```bash
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic
```

**Note:** This will not return any results even though we previously sent data to `first_topic`. This happens because the consumer is only retrieving data from the point it starts running, not from the beginning.

### 3. Consuming from the Beginning

To consume all messages from the beginning of the `first_topic`:

```bash
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --from-beginning
```

### 4. Producing with Round-Robin Partitioner

In the following example, we will produce messages to the `second_topic` (the topic we just created). We'll use a producer property to specify the `partitioner.class`, which in this case is the "RoundRobinPartitioner":

```bash
kafka-console-producer.sh --bootstrap-server localhost:9092 --producer-property partitioner.class=org.apache.kafka.clients.producer.RoundRobinPartitioner --topic second_topic
```

**Why Round-Robin?** The RoundRobinPartitioner is used here to send messages to one partition at a time, cycling through each partition. Without this partitioner, Kafka's optimizations typically send messages to the same partition until about 16KB of data has been sent, after which it switches partitions. However, it's important to note that using the RoundRobinPartitioner in production is generally inefficient and not recommended due to its poor performance compared to Kafka’s default partitioning logic.

### 5. Displaying Key, Value, and Timestamp

To display keys, values, and timestamps while consuming from `second_topic`:

```bash
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic second_topic --formatter kafka.tools.DefaultMessageFormatter --property print.timestamp=true --property print.key=true --property print.value=true --property print.partition=true --from-beginning
```

**Understanding Ordering:** Remember, Kafka ensures message ordering within each partition, not across the entire topic. This means that while you do get ordering within a single partition, full ordering across multiple partitions is not guaranteed or expected.

