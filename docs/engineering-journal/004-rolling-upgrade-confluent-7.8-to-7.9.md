# Engineering Journal 004
# Rolling Upgrade: Confluent Platform 7.8.1 to 7.9.8 (Docker KRaft Cluster)

**Date:** July 15, 2026
#Architecture

        Before Upgrade

        Kafka 1 (7.8.1)
             │
──────── Controller Quorum ────────
             │
        Kafka 2 (7.8.1)


             ↓ Rolling Upgrade


        Kafka 1 (7.9.8)
             │
──────── Controller Quorum ────────
             │
        Kafka 2 (7.9.8)
---

## Objective

Upgrade an existing two-node Apache Kafka KRaft cluster from Confluent Platform 7.8.1 to 7.9.8 using Docker Compose while preserving:

- Kafka metadata
- Existing topics
- Existing messages
- Named Docker volumes

The objective was to determine whether a one-node-at-a-time upgrade was possible in a two-node controller quorum.

---

## Environment

- Confluent Platform 7.8.1
- Apache Kafka (KRaft Mode)
- Docker Compose
- 2 Combined Broker/Controller Nodes
- Kafka UI

Topology:

Broker 1 (Controller)
Broker 2 (Controller)

Replication Factor: 2

---

## Upgrade Strategy

Instead of shutting down the entire cluster:

1. Pull Confluent Platform 7.9.8 image
2. Upgrade Kafka 1 only
3. Validate cluster
4. Upgrade Kafka 2
5. Validate cluster again

The Docker volumes were intentionally preserved to retain cluster metadata.

---

## Kafka 1 Upgrade

Updated docker-compose.yml

```
image: confluentinc/cp-kafka:7.9.8
```

Recreated only Kafka 1.

```
docker compose up -d \
--no-deps \
--force-recreate kafka-1
```

---

## Observations

Kafka 1 required approximately **2–3 minutes** before reaching the RUNNING state.

During this time:

- Kafka 2 remained reachable
- Existing topics remained accessible
- Existing messages were preserved

After Kafka 1 rejoined:

- Producer succeeded
- Consumer succeeded
- Topic creation succeeded
- Metadata quorum recovered successfully

The cluster successfully operated in a temporary mixed-version state:

| Broker | Version |
|---------|----------|
| Kafka 1 | 7.9.8 |
| Kafka 2 | 7.8.1 |

---

## Kafka 2 Upgrade

Kafka 2 was upgraded using the same procedure.

During Kafka 2 startup:

- Kafka 1 remained operational
- Producer commands completed successfully
- Topic creation completed successfully

After Kafka 2 became RUNNING:

Both brokers operated on Confluent Platform 7.9.8.

---

## Validation

Validated:

✓ Existing topics preserved

✓ Existing messages preserved

✓ New messages successfully produced

✓ New messages successfully consumed

✓ New topics created

✓ Metadata quorum recovered

---

## Findings

The rolling Docker upgrade completed successfully without deleting cluster data.

However, because this cluster consists of only **two controller nodes**, metadata quorum is unavailable while one controller is offline.

The experiment demonstrated that successful client operations immediately after broker restart do not necessarily imply continuous metadata availability. Some client operations may wait until quorum has been restored.

---

## Lessons Learned

A two-node controller quorum can tolerate temporary broker restarts for certain client workloads.

However, it does not provide controller quorum fault tolerance.

A three-node controller quorum is recommended for production environments to maintain metadata availability during maintenance.

---

## Next Steps

Investigate Kafka ISR behavior during broker failures.

Explore how:

- replication factor
- acks
- min.insync.replicas

affect write availability and durability.
