# 1. Kafka Quickstart Guide

This guide will walk you through the steps to get a Kafka cluster up and running, starting with Zookeeper and Kafka, followed by common Kafka topic operations and producer examples.

## Table of Contents

1. [Kafka Quickstart Guide](#1-kafka-quickstart-guide)
    1. [Prerequisites](#11-prerequisites)
    2. [Starting the Services](#12-starting-the-services)
        1. [Start Zookeeper](#121-start-zookeeper)
        2. [Start Kafka](#122-start-kafka)
    3. [Kafka Cluster is Running](#13-kafka-cluster-is-running)
2. [Kafka Topic Operations](#2-kafka-topic-operations)
    1. [Create a Topic](#21-create-a-topic)
    2. [List All Topics](#22-list-all-topics)
    3. [Describe a Topic](#23-describe-a-topic)
    4. [Delete a Topic](#24-delete-a-topic)
3. [Kafka Producer Operations](#3-kafka-producer-operations)
    1. [Producing to a Topic](#31-producing-to-a-topic)
    2. [Producing with Acknowledgement Properties](#32-producing-with-acknowledgement-properties)
    3. [Producing to a Non-Existing Topic](#33-producing-to-a-non-existing-topic)
    4. [Producing with Keys](#34-producing-with-keys)
4. [Kafka Consumer Operations](#4-kafka-consumer-operations)
    1. [Create a Topic with Multiple Partitions](#41-create-a-topic-with-multiple-partitions)
    2. [Consuming Messages](#42-consuming-messages)
    3. [Consuming from the Beginning](#43-consuming-from-the-beginning)
    4. [Producing with Round-Robin Partitioner](#44-producing-with-round-robin-partitioner)
    5. [Displaying Key, Value, and Timestamp](#45-displaying-key-value-and-timestamp)
5. [Kafka Consumer in Group](#5-kafka-consumer-in-group)
    1. [Create a Topic with 3 Partitions](#51-create-a-topic-with-3-partitions)
    2. [Start a Consumer in a Group](#52-start-a-consumer-in-a-group)
    3. [Start a Producer and Begin Producing Messages](#53-start-a-producer-and-begin-producing-messages)
    4. [Start Another Consumer in the Same Group](#54-start-another-consumer-in-the-same-group)
    5. [Start Another Consumer in a Different Group, Reading from the Beginning](#55-start-another-consumer-in-a-different-group-reading-from-the-beginning)
    6. [Observing Consumer Behavior](#56-observing-consumer-behavior)
6. [Reset Offsets](#6-reset-offsets)
    1. [Describe the Consumer Group](#61-describe-the-consumer-group)
    2. [Dry Run: Reset the Offsets to the Beginning of Each Partition](#62-dry-run-reset-the-offsets-to-the-beginning-of-each-partition)
    3. [Execute the Offset Reset](#63-execute-the-offset-reset)
    4. [Describe the Consumer Group Again](#64-describe-the-consumer-group-again)
    5. [Consume from Where the Offsets Have Been Reset](#65-consume-from-where-the-offsets-have-been-reset)
    6. [Describe the Group Again](#66-describe-the-group-again)
    7. [Key Points to Remember](#67-key-points-to-remember)

---

## 1.1. Prerequisites

- Ensure you have Kafka installed. This guide assumes Kafka version 2.13-3.8.0.

## 1.2. Starting the Services

### 1.2.1. Start Zookeeper

Zookeeper is required for managing Kafka brokers. Start the Zookeeper service by running the following command:

```bash
zookeeper-server-start.sh ~/kafka_2.13-3.8.0/config/zookeeper.properties
```

### 1.2.2. Start Kafka

Once Zookeeper is running, open a new terminal tab or window, and start the Kafka broker:

```bash
kafka-server-start.sh ~/kafka_2.13-3.8.0/config/server.properties
```

## 1.3. Kafka Cluster is Running

After successfully starting Zookeeper and Kafka:

- Zookeeper has established a connection.
- Kafka is now listening for incoming connections on port `9092`.

Your Kafka cluster should now be up and running. You can proceed with creating topics, and producing and consuming messages.

---

# 2. Kafka Topic Operations

Here are some common Kafka topic operations:

## 2.1. Create a Topic

To create a new topic called `first_topic` with 3 partitions and a replication factor of 1, use:

```bash
kafka-topics.sh --bootstrap-server localhost:9092 --topic first_topic --create --partitions 3 --replication-factor 1
```

## 2.2. List All Topics

To list all available topics in the Kafka cluster:

```bash
kafka-topics.sh --bootstrap-server localhost:9092 --list
```

## 2.3. Describe a Topic

To view details about a specific topic, such as partition count and replication factor:

```bash
kafka-topics.sh --bootstrap-server localhost:9092 --topic first_topic --describe
```

## 2.4. Delete a Topic

To delete a topic (this only works if `delete.topic.enable=true` is set in the Kafka server properties):

```bash
kafka-topics.sh --bootstrap-server localhost:9092 --topic first_topic --delete
```

---

# 3. Kafka Producer Operations

Below are some examples of producing messages to Kafka topics.

## 3.1. Producing to a Topic

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

## 3.2. Producing with Acknowledgement Properties

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

## 3.3. Producing to a Non-Existing Topic

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

## 3.4. Producing with Keys

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

# 4. Kafka Consumer Operations

Here are some common Kafka consumer operations:

## 4.1. Create a Topic with Multiple Partitions

Before consuming, let’s create a topic called `second_topic` with 3 partitions:

```bash
kafka-topics.sh --bootstrap-server localhost:9092 --topic second_topic --create --partitions 3
```

## 4.2. Consuming Messages

To start consuming messages from `first_topic`:

```bash
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic
```

**Note:** This will not return any results even though we previously sent data to `first_topic`. This happens because the consumer is only retrieving data from the point it starts running, not from the beginning.

## 4.3. Consuming from the Beginning

To consume all messages from the beginning of the `first_topic

`:

```bash
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --from-beginning
```

## 4.4. Producing with Round-Robin Partitioner

In the following example, we will produce messages to the `second_topic` (the topic we just created). We'll use a producer property to specify the `partitioner.class`, which in this case is the "RoundRobinPartitioner":

```bash
kafka-console-producer.sh --bootstrap-server localhost:9092 --producer-property partitioner.class=org.apache.kafka.clients.producer.RoundRobinPartitioner --topic second_topic
```

**Why Round-Robin?** The RoundRobinPartitioner is used here to send messages to one partition at a time, cycling through each partition. Without this partitioner, Kafka's optimizations typically send messages to the same partition until about 16KB of data has been sent, after which it switches partitions. However, it's important to note that using the RoundRobinPartitioner in production is generally inefficient and not recommended due to its poor performance compared to Kafka’s default partitioning logic.

## 4.5. Displaying Key, Value, and Timestamp

To display keys, values, and timestamps while consuming from `second_topic`:

```bash
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic second_topic --formatter kafka.tools.DefaultMessageFormatter --property print.timestamp=true --property print.key=true --property print.value=true --property print.partition=true --from-beginning
```

**Understanding Ordering:** Remember, Kafka ensures message ordering within each partition, not across the entire topic. This means that while you do get ordering within a single partition, full ordering across multiple partitions is not guaranteed or expected.

---

# 5. Kafka Consumer in Group

This section will demonstrate how Kafka consumers operate within consumer groups and how messages are distributed across multiple consumers.

## 5.1. Create a Topic with 3 Partitions

First, we create a topic named `third_topic` with 3 partitions to distribute the load across multiple consumers:

```bash
kafka-topics.sh --bootstrap-server localhost:9092 --topic third_topic --create --partitions 3
```

## 5.2. Start a Consumer in a Group

Next, start a consumer that will be part of a consumer group. Here, we specify the group ID as `my-first-application`:

```bash
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic third_topic --group my-first-application
```

**Explanation:** At this point, the consumer joins the `my-first-application` group. However, since no messages have been produced yet, the consumer will be idle until messages are available in the topic.

## 5.3. Start a Producer and Begin Producing Messages

Now, start a producer to send messages to the `third_topic`. We'll use the `RoundRobinPartitioner` to evenly distribute messages across the 3 partitions:

```bash
kafka-console-producer.sh --bootstrap-server localhost:9092 --producer-property partitioner.class=org.apache.kafka.clients.producer.RoundRobinPartitioner --topic third_topic
```

**Explanation:** The `RoundRobinPartitioner` is used here for learning purposes to observe how messages are distributed across partitions. In a real production environment, Kafka's default partitioning logic, which optimizes based on the data sent, is typically more efficient.

## 5.4. Start Another Consumer in the Same Group

Now, start a second consumer as part of the same `my-first-application` group:

```bash
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic third_topic --group my-first-application
```

**Explanation:** With two consumers in the same group, Kafka automatically distributes the partitions between them. If you send messages now, you'll notice that the messages are spread across the two consumers. Each consumer will handle different partitions, ensuring that the load is balanced.

## 5.5. Start Another Consumer in a Different Group, Reading from the Beginning

Finally, start a third consumer, but this time as part of a different group (`my-second-application`) and configure it to read from the beginning of the topic:

```bash
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic third_topic --group my-second-application --from-beginning
```

**Explanation:** Since this consumer belongs to a different group (`my-second-application`), it will start reading from the beginning of the topic (`--from-beginning`). This group will have its own offset, independent of the `my-first-application` group. This allows you to see how messages can be processed differently depending on the group.

## 5.6. Observing Consumer Behavior

- **Multiple Consumers in the Same Group:** When multiple consumers are part of the same group, Kafka divides the partitions among them. If there are more consumers than partitions, some consumers may not receive any data because they have no partitions assigned.

- **Rebalancing:** If a consumer in a group is stopped or starts, Kafka automatically rebalances the partitions among the remaining consumers. For example, if one consumer is stopped, the remaining consumers will take over its partition(s).

- **Lag Handling:** If a consumer in a group is stopped and then restarted, it will resume from the last committed offset in the group. If new messages were produced while the consumer was offline, it will "catch up" on those messages.

---

# 6. Reset Offsets

In this section, we’ll explore how to reset offsets for a consumer group using the Kafka command line tools. Resetting offsets can be useful when you want to reprocess messages from a specific point in the topic, such as from the beginning or a certain timestamp.

## 6.1. Describe the Consumer Group

First, let’s describe the current state of the consumer group `my-first-application` to see its offset positions and lag:

```bash
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-first-application
```

**Explanation:** This command provides details about the consumer group, such as the current offset for each partition, the committed offset, and the lag. The lag represents how many messages are yet to be consumed.

## 6.2. Dry Run: Reset the Offsets to the Beginning of Each Partition

Next, perform a dry run to see what would happen if we reset the offsets to the beginning (the earliest available messages) for the topic `third_topic`:

```bash
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group my-first-application --reset-offsets --to-earliest --topic third_topic --dry-run
```

**Explanation:** The `--dry-run` flag allows us to preview the effect of the offset reset without actually executing it. This is helpful to confirm that the offsets will be set as intended. In this case, the offsets would be reset to `0` for each partition, meaning the consumer group would reprocess all messages from the start.

## 6.3. Execute the Offset Reset

Once confirmed, we can execute the offset reset by removing the `--dry-run` flag and adding the `--execute` flag:

```bash
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group my-first-application --reset-offsets --to-earliest --topic third_topic --execute
```

**Explanation:** This command resets the offsets to the beginning of each partition, allowing the consumer group to reprocess all messages from the `third_topic`. It’s important to note that the consumer group must be inactive (i.e., no active consumers) for the reset to be executed.

## 6.4. Describe the Consumer Group Again

After resetting the offsets, describe the consumer group again to verify the changes:

```bash
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-first-application
```

**Explanation:** This will show the updated offsets. The offsets should now be set to `0` for each partition if the reset was successful, and the lag should reflect the total number of messages in each partition.

## 6.5. Consume from Where the Offsets Have Been Reset

Now, start consuming messages from the `third_topic` using the `my-first-application` group:

```bash
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic third_topic --group my-first-application
```

**Explanation:** Since the offsets have been reset to the beginning, the consumer will start reading from the very first message in each partition.

## 6.6. Describe the Group Again

Finally, describe the consumer group once more to observe the lag and offset positions after consuming some messages:

```bash
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-first-application
```

**Explanation:** After consuming messages, this command will show updated offsets and the new lag, which should be reduced or even zero if all messages have been consumed.

### 6.7. Key Points to Remember:

- **Offsets and Lag:** Offsets track the position of the last consumed message in each partition. Lag indicates how many messages are yet to be consumed.
- **Dry Run:** Always perform a dry run before executing an offset reset to ensure you understand the impact.
- **Execution:** The consumer group must be inactive (no consumers running) to reset offsets successfully.
- **Flexibility:** Kafka provides various options for resetting offsets, such as resetting to a specific timestamp, to the latest message, or by shifting the current offset by a certain number.

