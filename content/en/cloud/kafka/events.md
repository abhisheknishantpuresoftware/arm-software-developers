---
processors : ["Neoverse-N1"]
software : ["linux"]
title: "Verify the Kafka Cluster"
type: docs
weight: 5
hide_summary: true
description: >
    Learn how to create a topic, store events, an read them back.
---

After successfully setting up a 3 node Kafka cluster, we can verify it works by creating a topic and storing the events. Follow the steps below to create a topic, write some events into the topic, and then read the events.

## Create a topic:

Open terminal on the client machine.

```console

./bin/kafka-topics.sh --create --topic test-topic --bootstrap-server kf_1_ip:9092,kf_2_ip:9092,kf_3_ip:9092 --replication-factor 3 --partitions 64

```

## Describe the topic created:

Run in the same client terminal where the topic was created.

```console

./bin/kafka-topics.sh --topic test-topic --bootstrap-server kf_1_ip:9092,kf_2_ip:9092,kf_3_ip:9092 --describe

```
![describe_command (3)](https://user-images.githubusercontent.com/66300308/196909519-20eb720c-e43a-4eb7-abfa-405d4f931417.png)

## Run the producer client to write events into the created topic:

Run in the same client terminal where the topic was created.

```console

./bin/kafka-console-producer.sh --topic test-topic --bootstrap-server kf_1_ip:9092,kf_2_ip:9092,kf_3_ip:9092

```

![producer_message](https://user-images.githubusercontent.com/66300308/196901191-bc9944cb-8bd9-4d0c-8096-00b1a5a23452.png)

## Run the consumer client to read all the events created:

Open a new terminal on the client machine to run the consumer client.

```console

./bin/kafka-console-consumer.sh --topic test-topic --bootstrap-server kf_1_ip:9092,kf_2_ip:9092,kf_3_ip:9092

```
![consumer_message](https://user-images.githubusercontent.com/66300308/196900853-e8ca972c-6154-4a5c-92ac-3af92410ba95.png)

Write a message into the producer client terminal and press enter. You should see the same message appear on consumer client terminal. 

[<-- Return to Learning Path](/content/en/cloud/kafka/#sections)
