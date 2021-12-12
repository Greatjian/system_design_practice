# DDIA Notes

## Catalog

**Part 1 Foundations of Data Systems**

- [CP1 Reliable, Scalable, and Maintainable Applications](#CP1)
- [CP2 Data Models and Query Languages](#CP2)
- [CP3 Storage and Retrieval](#CP3)
- [CP4 Encoding and Evolution](#CP4)

**Part 2 Distributed Data**

- [CP5 Replication](#CP5)

## CP1

Reliable, Scalable, and Maintainable Applications

Three concerns that are important in most software systems:

- Reliability: make system work correctly, even when faults occur

   - Correct result
   - Fault-tolerant

       - Hardware faults (e.g. hard disk crash): add redundancy to individual hardware component, software fault-tolerant techniques
       - Software errors: no quick solution to software systematic faults
       - Human errors (leading cause): better design to minimize the opportunities for error, decouple components, test thoroughly at all levels, quick and easy discovery for failure, detailed and clear monitoring (metrics), good management and training practice

- Scalability: system's ability to cope with increased load

   - Describe load: load parameters (e.g. requests/sec, reads/writes, num of active users, hit rates of s cache)
   - Describe performance: throughput, response time, latency, P50/95/99
   - Common approaches

      - Scale up: vertical scaling, move to more powerful machines
      - Scale out: horizontal scaling, distribute the load across multiple machines
      - Scale out may introduce extra complexity, prefer scale up until the scaling out/distributed requirement prevails

   - No generic, one-size-fits-all scalable architectures: base on the assumption of the load parameters (which operation is common and which is rare)

- Maintainability: make life better for engineer and operation teams

   - Operability: make the operation teams easy to keep the system running smoothly
   - Simplicity: make it easy for new engineers to understand the system (e.g. abstraction)
   - Evolvability: make it easy for engineers to make changes to the system (e.g. Agile pattern)

## CP2

Data Models and Query Languages

- Software development layers

   - Applications: objects, data structures, APIs
   - Data model (relational, document, graph, CP2)
   - Data storage (bytes, disks/memory/network, CP3)
   - Hardware

- History of relational models

   - use cases: transaction processing (business: banking/booking/stock-keeping), batch processing (invoice/reporting)
   - Competitors:

      - 1970-1980: network model, hierarchical model
      - 1980-1990: Object database
      - 2000: XML database
      - 2010: NoSQL/Not Only SQL

- NoSQL's advantages

   - Greater scalability, large datasets with high throughput
   - Specialized query operations
   - Dynamic and expressive data model

- Object-Relational Mismatch

   - One-to-one: both works well
   - One-to-Many: document works well. Relational needs separate tables for normalization
   - Many-to-One, Many-to-Many: relational works well with table joins. Document is more expensive, leaving the join logic in the application codes

- Relational vs docuement database today

   - Relational:

      - Better support for joins
      - Many-to-One/Many relationships
   - Document:

      - Schema flexibility

          - Not enforced by the database (not schema-on-write), but schema-on-read. (not static/compile-time type checking, but dynamic/runtime type checking)
          - Q: Do all records have the same structure (homogeneous)?
      - Better performance due to locality

          - Q: Do you need large parts of the document at the same time?
          - Keep documents small and avoid writes to increase the writes of a document
      - Closer to application data structures
   - Convergence

      - XML and JSON support for relational databases
      - Join queries support in document databases

- Query languages for data

    - Imperactive

       - Tell the computer to perform certain operations in a certain order
    - Declarative

       - Specify the pattern of data you want, but nit how to achieve the goal
       - Concise and easier to work with
       - Hide implementation details of database engine, allow performance improvements without requiring changes to queries
       - Allow parallel execution since no ordering is guaranteed
    - Declarative queries on the web: CSS/XSL vs JavaScript for UI

- Graph data models

   - Good for common many-to-many relationships
   - Components: vertices (nodes, entities), edges (relationships, arcs)
   - Use cases: social graph, web graph, road/rail networks
   - Models and query languages

      - Property graph, Cypher (key-value pairs)
      - Triple-store, SPARQL (subject, predicate, object)

## CP3

Storage and Retrieval

- Storage engines

   - Optimized for transactional / analytics workloads
   - Log-structured (Hash Indexes, SSTables and LSM-Trees)
   - Page-oriented (B-Trees)

- Index

   - An additional structure to speed up read queries
   - Incur overhead on writes

- Hash Indexes

   - In-memory hash map
   - Key - value pair, value: byte offsets
   - Append-only map

      - Sequential write operations are faster than random writes
      - Concurrency control and crash recovery is simpler

   - Read: find the last key appearance
   - Write: Insert new records
   - Break log into segments and perform compaction on one/several segments
   - Delete: append deletion records that run on compaction
   - Crash recovery: store a snapshot of each segment's hash map on disk
   - Concurrency control: only one writer thread
   - Limitations

       - Hash table must fit in memory
       - Range queries are not efficient

- SSTables and LSM-Trees (no more above limitations)

   - SSTables

       - Sorted String Table: key value pairs sorted by keys
       - Merging segments is simple and efficient: mergesort algorithm
       - Each segment no longer keeps all the keys in memory (every few KB)
       - Apply compaction in read requests and then write to disks

   - LSM-Trees

       - Log-Structured Merge-Tree
       - Write: add to memtable (in-memory balanced red-black / AVL tree)
       - When memtable gets bigger, write to disk as a SSTable file as the most recent segment
       - Read: first find key in memtable, then in disk segments
       - Run SSTable segment compaction and merging over time
       - Crash recovery: keep a separate log for writes
       - Optimization

           - Bloom filters for looking up keys (non-existence)
           - SSTable segment compaction and merging (size-tiered compaction, leveled compaction)

- B-Trees

   - Key value pairs sorted by keys
   - Fixed-size blocks / pages on disk
   - Key appears only once in pages, update-in-place
   - Branching factor: number of references to child pages in one page
   - Optimization

       - Crash recovery: write-ahead log (WAL) / copy-on-write scheme
       - Save page space by storing abbreviated keys
       - Add sibling page references

- Compare B-Trees and LSM-Trees

   - LSM-Trees are typically faster for writes, B-Trees are thought to be faster on reads
   - Write amplification:

       - B-Trees: WAL, page, split pages
       - LSM-Trees: SSTable compaction and merging

   - LSM-Trees advantages

       - Lower wite amplification
       - Sustain higher write throughput (sequential writes)
       - Lower storage overhead (better compaction vs unused pages)

   - LSM-Trees disadvantages

       - Compaction interferes with ongoing read and writes
       - High write throughput: compaction cannot keep up with incoming writes
       - B-Trees: each key exists once in the index, better for locks on ranges of keys

- Secondary index

    - Index values are not necessarily unique
    - Map value to a list of ids / Make each value unique by appending the id

- Store values within the index

    - Clustered index: store value within the index
    - Covering index: store some of a table's columns within the index
    - Non-clustered index: store references of the data within the index

       - Heap file: append only, avoid data duplication

- Multi-column Indexes

   - Concatenated index
   - Multi-dimentional index

       - Geospatial data
       - Regular data with more search filters

- Full-text Search and Fuzzy Indexes

   - Within a certain edit distance
   - In-memory SSTable index of a finite state automation of the key, similar to a trie
   - Document classification and machine learning

- Keep Everything in Memory

   - Small datasets
   - In-memory database: Memcached
   - Restart: load the state from disk / network from a replica
   - Better performance: avoid the overhead of encoding in-memory data structures in a form that can be written to disk

- OLTP vs OLAP (online transaction / analytic processing)

|       Property       |                        OLAP                       |                    OLTP                   |
|:--------------------:|:-------------------------------------------------:|:-----------------------------------------:|
| Main read pattern    | Small number of records per query, fetched by key | Aggregate over large number of records    |
| Main write pattern   | Random-access, low-latency writes from user input | Bulk import (ETL) or event stream         |
| Primarily used by    | End user/customer, via web application            | Internal analyst, for decision support    |
| What data represents | Latest state of data (current point in time)      | History of events that happened over time |
| Dataset size         | GB to TB                                          | TB to PB                                  |

- Data Warehousing

   - OLTP systems are expected to be highly available and process transactions with low latency
   - A separate database that analytics can query to, without affecting OLTP operations
   - A separate database that can be optimized for analytic access patterns
   - Use Extract-Transform-Load (ETL) process to get data from various OLTP systems
   - Handle low volume but demanding queries
   - Disk bandwidth (not seek time) is the bottleneck

- Analytic Schemas

   - Star schema

       - Fact table: individual events, extremely large
       - Columns: attributes, foreign key references to dimension tables
       - Dimension table: who, what, where, when, how, why of the event

   - Snowflake schema

       - Dimensions are further broken down into sub-dimensions
       - More normalized, but not simpler

- Column-Oriented Storage

   - Store all the values from each column together
   - Each column file contains the rows in the same order

- Column Compression

   - Repetitive column values
   - Bitmap encoding
   - Run-length encoding
   - Advantages

       - Reduce the volume of data loaded from disk
       - Make efficient use of CPU cycles (bit-wise operations for vectorized processing vs function calls and conditions)

- Sorted Orders in Column Storage

    - Sort the entire row by each column
    - Faster queries based on the sorted column
    - Help with column compression (less distinct value together)
    - Data replication based on different sorted orders
    - Write:

        - B-Trees needs to update all column files on an insertion
        - LSM-Trees is a good solution

- Aggregation: Data Cubes and Materialized Views

    - Materialized View: a copy of the query result written to disk
    - Optimized for reads, but need to update for writes
    - Common special case: data cube
    - Data Cube: a grid of aggregates grouped by different dimensions
    - Certain queries are fast since they are precomputed, but less flexibility

## CP4

Encoding and Evolution

Change of data format / schema + rolling upgrade / staged rollout = old/new code version and schema coexist in the system at the same time

Backward compatibility: new code can read old data format

Forward compatibility: old code can read new data format

- Formats for Encoding Data

    - Two representations of data

        - Objects in memory in data structures for efficient access and manipulation
        - A self-contained sequence of bytes to write to a file / send over the network
        - Encoding / serialization vs decoding / deserialization

    - Language Specific Formats

        - Language built-in support
        - Encoding is tied to a particular programming language
        - Decoding needs to instantiate arbitrary classes - security problem
        - Data versioning for forward / backward compatibility
        - Efficiency problem (CPU time, size)

    - JSON, XML, CSV Textual Formats

        - Ambiguity in encoding numbers (precision, large numbers)
        - JSON and XML don't support binary strings
        - Complicated optional schema support for JSON and XML
        - No schema support for CSV
        - but...
        - Human readable, good as data interchange formats where no efficiency requires
        - Binary encoding for JSON and XML: use a lot of space of object field names since there is no schema

    - Thrift (fb) and Protocol Buffers (Google)

        - Binary encoding, require a schema
        - Field tags in schema definition to replace field names
        - Compatibility for schema evolution

            - Give new field a new tag
            - New field cannot be required
            - Can only remove optional fields
            - Never use the same tag number again

        - Why schema

            - Much more detailed validation rules
            - More compact in binary formats
            - Valuable for documentation
            - Check forward / backward compatibility
            - Enable type checking at compile time for statically typed programming languages

    - Apache Avro

        - Binary encoding, no tag numbers in the schema
        - Writer's schema (write a file, send over the network) compatible with reader's schema (read a file, receive from the network)
        - Schema resolution matches fields by field name, no order required
        - Only add / remove a field with a default value
        - Can change the field data type provided that they are convertible
        - Reader accesses write's schema

            - Large file with lots of records (Writer only needs to include once at the beginning of the file)
            - Database with individually written records (reader can fetch a record, extract the version number, fetch the schema)
            - Sending records over a network connection (bidiretion RPC protocal)

        - No need to pay extra attention for tag numbers in terms of a schema change (friendlier to dynamically typed schemas)
        - No code generation (No need for dynamically typed programming languages)

- Modes of Data Flow

    - Dataflow Through Databases

        - Write & read: sending a message to your future self
        - Preserve both forward and backward compatibility
        - Extra caution: e.g. support preservation of unknown field
        - Data outlives code, try to avoid rewriting / migrating data into a new schema

    - Dataflow through Services: REST and RPC

        - Client and server, API, microservice
        - Web service: using HTTP protocal

            - REST: a design philosophy

                - Simple data format (e.g. JSON)
                - URL for identifying resourses
                - Use HTTP features as much as we can (cache control, authentication, content type negotiation)

            - SOAP

                - XML based protocal
                - Enables code generation
                - Access remote service via local classes and method calls (not useful in dynamically typed programming languages)
                - An example of RPC

        - Problems with RPC (vs local method calls)

            - Unpredictable due to network problems
            - No result due to timeout
            - Retry mutiple times due to lost response
            - Wilder range for latency
            - Serialization vs reference / pointer
            - Translate datatypes across different languages

    - Message - Passing Dataflow

        - Asynchronous message - passing system
        - Intermediary message broker (message queue / middleware)

            - Act as a buffer
            - Automatically redeliver messages
            - Avoid the sender to know IP address and port number of the recipient (useful in cloud deployment)
            - Allow one message to be sent to several recipients
            - Logically decouples the sender from the recipient
            - But ... communication is usually one way

# Part 2 Distributed Data

Distribute a database across multiple machines

- Scalability / throughtput: spread the load
- Fault tolerance: redundancy to handle crash / network issue
- Latency: serve users on geographically-closed data centers to save network time

Vertical scaling: RAM/CPU cost grows faster than linearly, limited fault tolerance

Horizontal scaling: good for price, latency and fault tolerance, but add extra complexity

## CP5

Replication

- Leaders and followers

   - One replica as the leader, others as the followers
   - Leader handles writes, followers handle reads
   - Leader sends data change log (replication log) to followers
   - Single-leader replication: no conflict resolution
   - Multi-leader / leaderless replication: less latency, higher availability, better fault tolerance, but weaker consistency guarantee

- Sync vs async replication

   - Sync: leader waits for the follower(s) to receive the update then confirm sunccessful writes to the client
   - Followers is guaranteed to be up-to-date, but may take a long time if any follower doesn't respond
   - Often only one replica is configured synchronous
   - Async: leader doesn't wait for follower's response before returning write result to the client
   - Leader may fail and writes will lose, but faster

- Set up new followers

   - Take a consistent snapshot from the leader, and copy it to the follower
   - The snapshot is associated with the exact position in the replication log
   - The follower proceeds with the rest of log actions

- Handling node outages

   - Follower failure

      - Find the position in the replication log from its last transaction
      - Request for the rest of data changes

   - Leader failure (failover)

      - Determine that the leader has failed: sending frequently bounced messages
      back and forth between replicas (e.g. 30s)

      - Choosing a new leader (replica with the most up-to-date data changes)
      - Reconfigure the system to use the new leader
      - Possible bad senario

         - Not-updated writes in the old leader in async replication: discarding
         writes violates client durability expectation

         - Two nodes both believe they are the leader
         - Right timeout to determine leader is dead. Longer -> longer recovery time;
         shorter -> unnecessary failover due to temporary load spike

- Replication logs

   - Statement-based replication

      - SQL statements
      - nondeterministic function may cause difference in each replica
      - Must follow the same order, which limits multiple conccurent executing transactions
      - statements have side effects (triggers, UDF) to be nondeterministic

   - Write-ahead log (WAL)

      - Append-only sequence of bytes containing all writes to the database
      - Good: follower can process the log to catch up with the leader
      - Bad: Bytes are low level data that couples closely to the storage engine (e.g. disk blocks)

   - Logical (row-based) log replication

      - Row-level records, decouple from storage engine internals
      - Record all column value for a new row
      - Identifiers for a deleted/updated row

   - Trigger-based replication

      - Trigger lets you to register custom application code that is automatically
      executed during data change

      - Flexible, but prone to bugs and have more overheads

- Replication lag

   - Application reads from async follower that has fallen behind
   - Possible problems

      - Reading your own writes

         - Read-after-write consistency

              - Read from leader for something that user may have modified (e.g. own user profile)
              - Track the last modified update and only serve reads from the leader/followers that have a more recent update
              - Client sends the timestamp (logical: log sequence number; physical: clock timestamp)

         - Cross-device read-after-write consistency

              - Centralized user timestamp metadata for each replica knows the last user update on other replicas
              - Route requests from all of a user's devices to the same datacenter to be handled by the same leader

      - Monotonic reads

         - See things backward in time
         - Strong consistency > monotomic reads > eventual consistency
         - Each user always reads from the same replica (chosen from hash of user id)

      - Consistent prefix reads

         - Reading the writes must follow the same order of the writes
         - Different partition operates differently. No global ordering of writes
         - Any writes that are casually related to each other are written to the same partition

- Multi-leader replication

   - Situations

       - Multi-datacenter operation

           - One leader per datacenter
           - Less network delay, better latency
           - Each datacenter works independently, better fault tolerance
           - Local network within a datacenter, better network tolerance

       - Clients with offline operation

           - Application needs to work when it's offline
           - One device per/is a datacenter

       - Collaborative editing

           - Accept multiple writes at the same time
           - Too slow with only one replica accepts writes that uses locks

   - Handling Write Conflicts

       - Synchronous vs Asynchronous conflict handling

           - Sync: wait for the writes to be replicated to all replicas before telling the writes are successful
           - Single-leader replication is better

       - Conflict avoidance

           - Ensure all writes for a particular record to go through the same leader
           - Sometimes not possible: one datacenter is down / user has moved a new location served by another datacenter

       - Converging to a consistent state

           - No ordering of writes, may be different to each datacenter
           - All datacenters need to converge to the same final value
           - Ways

               - Give each write a unique ID, last write (highest ID) wins (LWW), prone to data loss
               - Give each replica a unique ID, prone to data loss
               - Merge the values together. e.g. concatenation
               - Record the conflict within a data structure and let application code to resolve

       - Custom conflict resolution logic

           - User writes conflict resolution logic using application code
           - On write: don't prompt the user, run in the backgroud
           - On read: store the conflict at write, prompt the user on the read at the conflict

   - Multi-leader replication topologies

       - Communication paths along which writes are propagated from one node to another
       - All-to-all topology

           - Network delay may cause writes (write + update on the same record) to arrive at wrong orders in some replicas
           - Version vectors: always apply the writes with the latest version

       - Circular/Star topology

           - One write needs to pass through several nodes to reach all replicas
           - Tag the node identifier all the path to prevent infinite loops
           - Less fault tolerance for node failure

- Leaderless replication

   - All replicas receive reads and writes
   - Writing to the Database When a Node Is Down

       - A read and write send to all replicas, version vectors to determine the latest value
       - Read repair: update replicas with staled values at a read. Works well for frequently read values
       - Anti-entropy process: background process constantly looking for and updating staled values

   - Quorums for reading and writing

       - At least `w + r > n` to expect a up-to-date read
       - Node unavailable due to crash / full disk / network interruption ...

   - Limitations of Quorum Consistency

       - Set `w + r <= n`, more likely to read stale values, but lower latency and higher availability
       - Edges cases for reading stale values even `w + r > n`

           - Sloppy quorum is used
           - Two writes occurred concurrently
           - A write happens concurrently with a read
           - A write overall fails, but not rolled back on the successful replicas
           - A node carrying new values fails, and restored from replica with old values. This could break the quorum

       - Better for use cases that can tolerate eventual consistency
       - Monitoring staleness

           - Metrics for replication lag
           - Leader-based replication: writes are applied in the same order from the replication log
           - Hard to tell from leaderless replication since no order for the writes (and no log)

   - Sloppy quorum and Hinted Handoff

       - When quorum doesn't meet, should we fail the write or write it to some other nodes that aren't among the n nodes
       - When the original nodes are back, other nodes send the writes back
       - Useful to inscrease write availability, but bring no guarantee to the reads until the hinted handoff
       - Multi-datacenter operation

           - `n` includes nodes in all datacenters, user can specify from the config file
           - The client usually waits for ack from a quorum of nodes with the local datacenters
           - Unaffected by the dalays and interruptions of the network

           - Or, `n` describes the number of replicas within one datacenter
           - Cross-datacenter replication (anti-entropy process) happens async in the background

   - Detecting concurrent writes

       - Last writes wins

           - Give each write / replica a unique id. Largest ID wins
           - Reach eventual consistency, but at the cost of data loss
           - Some writes are reported successful to the user, but are getting replaced

       - Two operations are concurrent if neither happens before the other (i.e. neither knows about the other)
       - Version vector

           - A version number per replica per key attached with the value
           - Version vector is the collection of veriosn numbers from all the replicas
