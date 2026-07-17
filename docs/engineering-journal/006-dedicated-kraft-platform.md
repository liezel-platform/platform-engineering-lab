# Engineering Journal 006
# Dedicated KRaft Platform: Three Controllers and Three Brokers

**Date:** July 16, 2026

## Objective

Build a production-inspired Apache Kafka KRaft cluster using dedicated controller and broker roles.

The goal was to separate the Kafka control plane from the data plane and create a topology capable of maintaining controller quorum and broker availability during single-node failures.

## Environment

- Confluent Platform 7.9.8
- Docker Compose
- Three dedicated KRaft controllers
- Three dedicated Kafka brokers
- Kafka UI
- KRaft mode
- Static controller quorum

## Architecture

```text
              KRaft Controller Quorum

        controller-1
        controller-2
        controller-3

                  |
                  |
        -----------------------
        |          |          |
     kafka-1    kafka-2    kafka-3

                  |
               Kafka UI
## Node Configuration
| Node         | Role       | Node ID |
| ------------ | ---------- | ------: |
| controller-1 | Controller |       1 |
| controller-2 | Controller |       2 |
| controller-3 | Controller |       3 |
| kafka-1      | Broker     |     101 |
| kafka-2      | Broker     |     102 |
| kafka-3      | Broker     |     103 |


## Controller Quorum
1@controller-1:29093
2@controller-2:29093
3@controller-3:29093

The controllers were started before the brokers.

Validation confirmed:

One active controller leader
Three registered controller voters
Metadata quorum available
Metadata high watermark advancing

##Broker Validation

All three Kafka brokers successfully registered with the controller quorum.

Broker API validation confirmed the presence of:

101
102
103

##Topic Validation

Topic description confirmed that every partition had replicas distributed across all three brokers.

Expected replica and ISR membership:

Replicas: 101,102,103
ISR: 101,102,103

##Findings

The dedicated topology successfully separated controller responsibilities from broker responsibilities.

Compared with the earlier two-node combined-role cluster:

Broker traffic is isolated from controller quorum duties.
Controller nodes can be tested independently from brokers.
Broker maintenance does not automatically remove a controller voter.
The three-controller quorum can tolerate the loss of one controller while preserving a majority.
Topics using replication factor 3 and min.insync.replicas=2 can tolerate one broker failure while preserving stronger write durability

##Lessons Learned

A production-inspired KRaft design benefits from independent controller and broker roles.

Three controllers provide quorum fault tolerance because two controllers can maintain a majority after one controller fails.

Three brokers allow partitions to be replicated across independent broker nodes.
