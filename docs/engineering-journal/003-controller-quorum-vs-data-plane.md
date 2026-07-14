# Engineering Journal 003

## Question

What happens to an existing Kafka topic when one controller in a two-node KRaft cluster becomes unavailable?

## Initial Observation

After stopping Broker 2:

- Broker 1 became leader for all partitions.
- ISR contained only Broker 1.
- Broker logs repeatedly showed:

In the new epoch 3803, the leader is (none).


indicating the controller quorum had no elected leader.

## Initial Hypothesis

I expected message production to fail because the controller quorum was unavailable.

## Experiment

Produced several messages while Broker 2 remained offline.

Initially, the new records did not appear in the topic offsets.

After waiting and rechecking, the records became visible and were successfully consumed.

## Final Observation

Existing-topic reads continued.

Existing-topic writes also continued, although delivery appeared delayed.

The broker logs continued showing repeated controller election attempts.

## What I Learned

This experiment highlighted the distinction between:

- Partition leadership
- Controller quorum
- Data plane
- Control plane

A degraded controller quorum did not immediately prevent existing-topic reads and writes.

Further experiments are needed to understand how metadata operations behave while the controller quorum is unavailable.

Future Work: Investigate whether creating a new topic succeeds while the controller quorum has no elected leader.
## Metadata Operation Test

To test whether the control plane remained available, I attempted to create a new topic named `payments` while Broker 2 was offline.

### Command

```bash
kafka-topics \
  --bootstrap-server kafka-1:29092 \
  --create \
  --topic payments \
  --partitions 3 \
  --replication-factor 1

### Result
TimeoutException: Call(callName=createTopics...) timed out
Cancelled createTopics request due to node 1 being disconnected


## Recovery Test

Broker 2 was restarted to restore the two-node controller quorum.

After both Kafka nodes were running again, I retried the topic creation request.

### Result

The `payments` topic was created successfully after controller quorum was restored.

### Final Conclusion

The experiment demonstrated that a two-node combined broker/controller KRaft cluster can continue serving some existing-topic traffic after one node fails.

However, without a controller majority:

- controller elections repeatedly fail
- metadata operations such as creating new topics time out
- the cluster operates without redundancy
- another broker failure would make the topic unavailable

Restoring Broker 2 re-established controller quorum and allowed metadata operations to resume.

## Production Connection

This behavior was similar to a production ZooKeeper incident I previously supported, where existing application traffic continued while metadata operations failed.

The experiment helped connect my ZooKeeper-based operational experience with KRaft controller quorum behavior.
