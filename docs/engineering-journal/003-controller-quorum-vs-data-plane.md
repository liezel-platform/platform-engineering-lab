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
