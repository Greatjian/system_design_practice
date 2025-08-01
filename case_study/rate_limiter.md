# Rate Limiter

## Requirements

* Accuracy
* Low latency
* Less memory usage
* Exception handling
* High fault tolerance (distributed)

Clarify: server side or middleware (e.g. API gateway)

* Server side: make sure the language and service support, take effort but can be customized (e.g.full control of algorithm)
* Gateway: third party, simple to build. Or already have one

Algorithm (burst of traffic, memory usage):

* Token bucket: bucket size, refill rate (different buckets for each service/ip address)
* Leaking bucket: bucket size, process rate (stable outflow rate)
* Fixed window: window duration, threshold
* Sliding window: window duration, threshold (more accurate, but more memory usage)

## High Level Design

client -> rate limiter -> API server
                       -> Redis (in-memory cache, store counter, buckets info)

## Deep Dive

* Rules: xml/proto files, store on disks
* Exceed rate limit: store in queue for retry / drop message
* Client aware: store info in HTTP response header
* Logging, monitoring, alert

## Detailed Design

client -> rate limiter (rule cache) -> workers periodically pull from rule files
                       -> API server
                       -> Redis (in-memory cache, store counter, buckets info)
                       -> message queue

## Distributed environment

* Race condition: multiple concurrent requests sent to one rate limiter. Lock increases latency -> distribute the rate limiter
* Syncronization issue: how to syncronize if there are multiple rate limiters -> eventual consistency model