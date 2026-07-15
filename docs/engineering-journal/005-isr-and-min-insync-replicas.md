# Engineering Journal 005
# Understanding ISR, min.insync.replicas and Write Durability

**Date:** July 15, 2026

---
##Architecture
Broker 2 Down

        Kafka 1 (Leader)
             │
      ISR = {1}

Broker 2 (Offline)

RF = 2
minISR = 1  → Writes succeed

RF = 2
minISR = 2  → Writes rejected
## Objective

Investigate how Kafka behaves during broker failures when different durability settings are used.

Specifically:

- In Sync Replicas (ISR)
- Replication Factor
- acks=all
- min.insync.replicas

---

## Environment

Confluent Platform 7.9.8

Two-node KRaft Cluster

Replication Factor = 2

Topic:

rolling-upgrade-test

---

## Test 1

### Broker Failure

Broker 2 was stopped.

Immediately after Broker 2 shutdown:

Metadata quorum request:

```
kafka-metadata-quorum describe --status
```

Result:

```
TimeoutException
DisconnectException
```

Topic creation:

```
timeout 20s kafka-topics --create ...
```

Result:

```
Exit Code 124
```

The metadata operation timed out.

---

## Existing Topic Operations

Producer

```
acks=all
```

Result:

Success

Consumer

Result:

Success

The produced message was successfully consumed.

---

## Kafka UI

Kafka UI reported:

Replication Factor:

2

Under Replicated Partitions:

3

In Sync Replicas:

3 of 6

This indicated that each partition had only one replica remaining in the ISR.

---

## Interpretation

Although the controller quorum became unavailable, the leader broker continued accepting writes for existing topics.

At this point the topic was still using the default minimum ISR configuration.

The leader acknowledged writes because only one ISR member was required.

---

# Test 2

Changed topic configuration:

```
min.insync.replicas=2
```

Broker 2 remained offline.

Producer:

```
acks=all
```

Result:

```
NOT_ENOUGH_REPLICAS
```

Producer retries:

```
NOT_ENOUGH_REPLICAS
```

Final error:

```
NotEnoughReplicasException
```

---

## Why It Failed

Topic configuration:

Replication Factor:

2

Required ISR:

2

Available ISR:

1

Kafka rejected the write because the minimum number of required in-sync replicas was not available.

---

## Comparison

| Configuration | Broker 2 Offline | Result |
|---------------|------------------|--------|
| RF=2, minISR=1, acks=all | Yes | Produce succeeded |
| RF=2, minISR=2, acks=all | Yes | Produce rejected |

---

## Key Learning

Replication Factor determines how many copies Kafka attempts to maintain.

min.insync.replicas determines how many replicas must acknowledge writes before Kafka accepts the record.

acks=all does not guarantee that every configured replica stores the message.

It guarantees acknowledgements only from the replicas currently required by the ISR policy.

---

## Operational Trade-off

Higher Availability

```
Replication Factor = 2
min.insync.replicas = 1
acks=all
```

Advantages

- Writes continue during broker failure

Disadvantages

- Lower durability
- Recent writes could be lost if the remaining broker fails before replication recovers

---

Higher Durability

```
Replication Factor = 2
min.insync.replicas = 2
acks=all
```

Advantages

- Prevents acknowledged writes from existing on only one broker

Disadvantages

- Writes become unavailable when one broker fails

---

## Conclusion

This experiment demonstrated the distinction between Kafka's control plane and data plane.

Loss of controller quorum prevented metadata operations.

Existing topic reads continued.

Existing topic writes depended on ISR configuration.

By modifying only min.insync.replicas, Kafka shifted from prioritizing write availability to prioritizing durability.

Understanding this trade-off is critical when designing highly available Kafka platforms.
