# Unique ID Generator

## Requirements

* Unique
* Numerical values fits into 64 bit
* Increase with time, not necessarily increment by 1 each time
* Generate 10k per second

## High Level Design

* Database auto_increment feature: hard to scale
* UUID: non-numeric, 128 bit long, don't go up with time
* Divide and conquer: generate under different sections

## Detailed Design

Timestamp + data center id + machine id + sequence number

To be discussed:

* Clock synchronization across machines/data centers