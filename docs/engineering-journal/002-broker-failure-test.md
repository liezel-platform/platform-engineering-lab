# Engineering Journal 002 - Broker Failure Test

## Hypothesis

If Broker 2 stops, I expect message production to fail because the topic has a replication factor of two and one replica will become unavailable.

I also expect partitions led by Broker 2 to become unavailable unless Kafka can elect Broker 1 as the new leader.

## Current State

- Broker 1: running
- Broker 2: running
- Topic: `orders`
- Partitions: 3
- Replication factor: 2
- ISR for all partitions: both brokers

## Actual Result

After stopping Broker 2, Broker 1 became leader for all three partitions.

The ISR changed from both brokers to only Broker 1.

The topic remained available, but the cluster was operating without redundancy.

[appuser@kafka-1 ~]$ kafka-topics --bootstrap-server kafka-1:29092 --describe --topic orders
Topic: orders   TopicId: PitfYzGGSZSU-mVpDMdoBg PartitionCount: 3       ReplicationFactor: 2    Configs:
        Topic: orders   Partition: 0    Leader: 1       Replicas: 1,2   Isr: 1  Elr:    LastKnownElr:
        Topic: orders   Partition: 1    Leader: 1       Replicas: 2,1   Isr: 1  Elr:    LastKnownElr:
        Topic: orders   Partition: 2    Leader: 1       Replicas: 2,1   Isr: 1  Elr:    LastKnownElr:
[appuser@kafka-1 ~]$ kafka-console-producer --bootstrap-server kafka-1:29092 --topic orders
>order-after-broker2-stop
[appuser@kafka-1 ~]$ kafka-console-consumer --bootstrap-server kafka-1:29092 --topic orders --from-beginning
order-1001
order-1002
order-1003
order-after-broker2-stop
^CProcessed a total of 4 messages
