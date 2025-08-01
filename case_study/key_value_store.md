# Key Value Store

## Requirements

* Pair size is small (~10kb)
* Num pairs could be large
* High availability (distributed)
* Automatic scaling
* Low latency

## High Level Design

Distributed hash table

client < --- > nodes (with consistant hashing)

write path: commit log, memory cache, sstable

read path: memory cache, bloom filter, sstable

## Deep Dive

* CAP theorem: partition tolerance is always chosen. Need to discuss choosing between consistency vs availability

* Data partition: consistent hashing

    * Distribute data to servers evenly
    * Minimize data movement when nodes are added/removed
    * Support automatic scaling and heterogeneity

* Data replication

    * Ensure high availability and reliability
    * Place in several data centers globally

* Consistency

    * Leaderless replication: quorum W + R > N for strong consistency
    * Eventual consistency: ask client to reconcile conflicts using version vectors