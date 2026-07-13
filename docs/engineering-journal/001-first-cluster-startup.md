# Engineering Journal 001 - First Cluster Startup

## Goal

Deploy a two-broker Apache Kafka KRaft cluster using Docker Compose.

## Result

- Successfully started two Kafka brokers.
- Kafka UI detected one cluster and two brokers.
- No topics existed after startup, which is expected.

## Challenges

- Encountered a Docker container name conflict because containers from a previous lab still existed.
- Removed the old containers and successfully redeployed the environment.

## Listener Lesson

- Kafka commands executed inside Docker containers must use the internal listener:

kafka-1:29092

- Using the external listener caused the client to receive localhost addresses for other brokers, which are not valid from inside a container.

## Topic Validation

Created an `orders` topic with:

- 3 partitions
- Replication factor 2
- Both brokers in the ISR

Validated the topic using Kafka's console producer and consumer.

### Messages Produced

```text
order-1001
order-1002
order-1003
