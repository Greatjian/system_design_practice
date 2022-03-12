# DDIA Notes

## Catalog

**Part 1 Foundations of Data Systems**

- [CP1 Reliable, Scalable, and Maintainable Applications](#CP1)
- [CP2 Data Models and Query Languages](#CP2)
- [CP3 Storage and Retrieval](#CP3)
- [CP4 Encoding and Evolution](#CP4)

**Part 2 Distributed Data**

- [CP5 Replication](#CP5)
- [CP6 Partitioning](#CP6)
- [CP7 Transactions](#CP7)
- [CP8 The Trouble with Distributed Systems](#CP8)
- [CP9 Consistency and Consensus](#CP9)

**Part 3 Derived Data**

- [CP10 Batch Processing](#CP10)
- [CP11 Stream Processing](#CP11)
- [CP12 The Future of Data Systems](#CP12)

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
    - Searching for occurrance of a particular value

- Store values within the index

    - Clustered index: store value within the index

        - The table is stored in the sort order specified by the primary key
        - Can be either heap or index-organized storage
        - Table in the same/sequential pages -> avoid disk random IO

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

|       Property       |                        OLTP                       |                    OLAP                   |
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

## CP6

Partitioning

Better scalability, larger query throughput, less latency

- Partitioning Key-Value Data

    - Spread the data and query load evenly across nodes (skew otherwise)
    - Random, but not way knowing which node the record is in
    - Partitioning by Key Range

        - Easy to fetch for range scans
        - But certain access patterns can lead to hot spots
        - Prefix other fields in front of the key
        - Dynamic partitioning

    - Partitioning by Hash of Key

        - Uniformly distribute the skewed data
        - No need to be cryptographically strong
        - Partitioning by key hash ranges
        - Not good for ranged queries: adjacent keys in different partitions
        - Fixed number of partitions/(per node)/Dynamic partitioning

    - Compromise between the two above

        - Compound primary key for several columns
        - Only the first column is hashed to determine the partition
        - Other columns are used as a cancatenated index for sorting the data
        - Can perform ranged queries over the rest columns given a fixed value of the first column

    - Skewed Workloads and Relieving Hot Spots

        - Worst case: all reads and writes on the same key
        - Add a random number to the start/end of the key
        - But need to read data to all the keys and combine it
        - Extra bookkeeping of the keys

- Partitioning and Secondary Indexes

    - Partitioning Secondary Indexes by Document

        - Each partition maintains its own secondary indexes
        - Add document ID (in this partition) to the secondary index
        - Easy for writes to add new document under one partition
        - Need to send the query to all partitions and combine the results
        - Read queries are expensive, prone to latency amplification

    - Partitioning Secondary Indexes by Term

        - A global index that covers all partitions of documents
        - The index term is partitioned
        - Partitioning by the term: useful for range scans
        - Partitioning by the hash of the term: even load distribution
        - Reads are more efficient
        - Writes are slower and more complicated, affect multiple partitions
        - Write updates could be asynchronous, meaning an instant read may not be updated

- Rebalancing Partitions

    - More/less machines, more/less CPU/RAM (thus more/less partitions) in a node over time
    - Moving load from one node in the cluster to another
    - Requirements

        - Evenly distributed load
        - Continue accepting reads and writes while rebalancing
        - No more data than necessary should be moved (faster, less network/disk IO)

    - Strategies

        - Hash(key) mod N. Bad since most of the keys need to move
        - Fixed number of partitions

            - Operationally simpler
            - More data -> larger partition size
            - High enough to acommodate future growth
            - Hard if total size of the dataset is varaible

        - Dynamic partitioning

            - Limited partition size
            - More data -> more number of partitions per node
            - When a partition grows to exceed a configured size, it splits into two partitions
            - But it starts with one single partion in a single node to hanle all writes while other nodes are idle
            - Could configure to start with an intial set of partitions

        - Partitioning proportionally to nodes

            - Fixed number of partitions per node
            - More data -> more nodes, but same amount of partition in a node
            - Limited partition size
            - Repartition needs to pick new boundaries: require hash-based partitioning

    - Operations: Automatic/manual rebalancing

        - Fully automated rebalancing: less operational work, but more unpredictable
        - Only changes when the administrator explicitly configures it
        - Fully manual

- Request Routing

    - How does the client know which node to connect?
    - Allow clients to contact any node (handle/forward request)

        - Gossip protocol among the nodes
        - More complexity in the nodes, but no more coordnation service dependency

    - Send the client request to a routing tier first, which determines the node, then forward the request

        - Coordination service: ZooKeeper
        - Maintain mappings of partition to nodes
        - Notify routing tier for any mapping update

    - Requires the client to be aware of the partition assignment to nodes first

## CP7

Transactions

- Intro

    - Group several reads and writes together into a logical unit
    - Simplify the programming model for applications accessing a database
    - Higher transactional guarantee, but lower performance and availability
    - Isolation levels: read committed, snopshot isolation, serializability

- The Meaning of ACID (safety guarantees)

    - Atomicity: something cannot be broken down into smaller parts
    - Consistency: certain statements about data invariants must always be true (also rely on applications, e.g. balanced accounts)
    - Isolation: concurrent executing transactions are isolated from each other (serializability)
    - Durability: once a transaction is committed, its data will never be forgotten

- Single-Object and Multi-Object Operations

    - BEGIN TRANSACTION ... COMMIT
    - No way for nonrelational databases to group operations together
    - Single-object writes: log for crash recovery + lock on each object
    - Need for multi-object transactions

        - Writes to several different objects at once
        - e.g. Foreign key reference in relational data model
        - e.g. Updating denormalized information
        - e.g. Database with secondary indexes
        - Hard for error handling without transactions

    - Handling errors and aborts

        - Retry an aborted transaction, but not perfect ...
        - e.g. Transaction succeeded, but fail to commit to client, so performed twice
        - e.g. Make it worse if the error is due to overload
        - e.g. Only worth trying transient errors
        - e.g. More side effects during the retry

- Weak Isolation levels

    - Weaker levels of isolation, better performance
    - Read committed

        - No dirty reads

            - Only see data has been committed when reading
            - Locks on objects, works but bad performance since need to wait for long-running write transactions
            - Let database remember both the old and new values (based on whether the transaction changing the value has been committed or not)

        - No dirty writes

            - Only overwrite data has been committed when writing
            - Row level locks on object: hold until committed / aborted

    - Snapshot Isolation

        - Read committed is vulnerable to nonrepeatable reads, which can be solved by snapshot isolation
        - Each transaction reads from a consistent snapshot of the database
        - Database keeps several different committed version of an object (multi-version concurrency control, MVCC)
        - Read committed uses a separate snapshot for each query, while snapshot isolation uses the same snapshot for an entire transaction
        - Implementation: add (TxId, created_by, deleted_by) for each object value change
        - Visibility rules for a consistent snapshot

            - a: The transaction created the object had been committed
            - b: The object is not marked for deleted, or the transaction requested the deletion has not been committed
            - If both a and b apply, then the object is visible
            - Summary: earlier transactions (with smaller TxId) cannot see later transaction changes, even if it happens early on some objects

        - Indexes on snapshot isolation

            - Have index point to all versions of an object and filter out the invisible ones to the current transaction
            - Or, Create a new copy of each modified page, and have the parent pages till the root pages copied (Each write creates a new B-tree root)

    - Preventing Lost Updates

        - Discussed what a read-only transaction can see among the concurrent writes
        - Now discuss two transactions writing concurrently (e.g. read-modify-write cycle, one modification can be lost)
        - Solutions

            - Atomic write operations: database provides atomic update operations by taking an exclusive lock on the object
            - Explicit locking: up to the application to lock the object from the query
            - Database automatically detecting lost updates: let them execute in parallel, and have transaction manager detected it (the check is more efficient with snapshot isolation), the abort one and force it to retry
            - Compare-and-set: allowing an update only if the value has not changed since last read
            - Conflict resolution and replication: allow replicated databases to have concurrent writes to create several conflicting versions of a value, and have application code or special data structures to resolve and merge the versions / Or last write wins (LWW), which is faster but prone to lost updates

    - Write Skew and Phantoms

        - More potential race conditions on concurrent writes
        - Two check(read)-decide-write cycles, with write changes the decide
        - Lock may not help since the check may not return any row
        - Solution

            - Materializing conflicts: take a phantom and turn it into a lock conflict on a concrete set of rows in the database (e.g. create a new table), but hard and error-prone to do
            - Serializable isolation is preferrable

- Serializability

    - Though transactions may execute in parallel, the end result is the same as if they had executed one at a time, serially
    - Possible techniques

        - Actual Serial Execution

            - Literally execute transactions in a serial order
            - RAM becomes cheap, it's possible to have the entire dataset in memory
            - OLTP workloads are short and involve a small number of reads and writes (which fits here)
            - OLAP workloads are typically read only, and can run on snapshot isolation level
            - Avoid the coordination overhead of locking
            - Encapsulate transactions in stored procedures for less network IO (and less disk IO if the dataset can fit into memory), but...

                - Each database has its own language for stored procedures, lack the library ecosystem
                - Code running in a database (stored procedures) is difficult to manage (debug, deploy, test, monitoring)
                - Database is more performance sensitive than application server, prone to badly written procedures

            - Partitioning

                - Single CPU core works for low write throughtput applications
                - For high write throughtput, can partition the data and have its own processing thread and CPU core
                - But have addtional coordination overhead for cross-partition transactions

        - Two-Phase Locking (2PL)

            - Snapshot isolation: readers never block writers, and writers never block readers
            - 2PL: writers block readers, and vice versa
            - Provides erializability, no race conditions and prevent lost updates
            - Implementation

                - Shared locks for reads, exclusive locks for writes
                - Transaction holds the lock till committed / aborted
                - Database automatically detects deadlocks, aborts one for retry

            - Performance

                - Worse throughtput and latency compared to weak isolation
                - Due to overhead of acquiring and releasing locks
                - But more on reduced concurrency for waiting for the locks
                - Unstable lantencies and slow at higher percentiles
                - Deadlock happens more frequently than lock-based read committed isolation

            - Predicate lock to prevent phantoms

                - Belongs to all objects matching the predicate (instead of a particular object)
                - Applies even to objects that do not yet exist in the database, but might add in the future (phantom)
                - Could be time consuming for checking matches
                - Use index-range locking instead

                    - Simplified approximation by matching a greater set of objects
                    - Have the search condition attached to the index, and lock the index
                    - More locked objects, but less overhead

        - Serializable Snapshot Isolation (SSI)

            - Pessimistic concurrency control (e.g. 2PL): wait until it's safe and continue
            - Optimistic concurrency control (e.g. SSI): continue anyway and check whether to abort and retry during commit
            - Bad performance for high contention (multiple transactions try to access the same object, thus many retries, contention can be reduced for commutative atomic operations (e.g. counter))
            - Bad performance if it's closing to the maximum throughtput (retries adds more throughput)
            - Otherwise tend to perform better than pessimistic concurrency control: no need to block wating for locks, thus predictable and less variable latency
            - Desicion based on an outdated premises (Write Skew and Phantoms, two check(read)-decide-write cycles, with write changes the decide)

                - Detecting stale MVCC reads (first uncommitted write before the second read): durng the second transaction commit checking, abort it if there are any ignored writes that have been committed
                - Detecting writes that affect prior reads (first uncommitted write after the second read): during the first write, database notifies the transactions that have its prior read. During the notified transaction commit checking, abort it if the write transaction has been committed

            - Performance tradeoff: the database keeps track of the detail of transactions vs booking keeping overhead
            - Favor transactions with short time read-write queries, thus less abort rates

## CP8

The Trouble with Distributed Systems

- Faults and Partial Failures

    - Single computer is deterministic -- either fully functional or entirely broken
    - Distributed system may have parts that are broken in some unpredictable way -- nondeterministic partial failure
    - Fault tolerant: build reliable system from unreliable components

- Unreliable Networks

    - Shared-nothing systems: machines connected by a network
    - Cheap, make use of commoditized cloud computing services, achieve high reliability through redundancy
    - Asynchronous packet networks: no guarantee when or whether it will arrive
    - Need to define and test network error handling
    - Usual way of handling network errors is a timeout: give up wating after some time

        - Too long: long wait till declare a node is dead
        - Too short: higher risk of incorrect declaration (e.g. temporary slow down due to spike, make it worse if it's due to high load)
        - Most systems don't have gurantee: unbounded delays
        - Choose timeout experimentally, or automatically adjust the timeout according to the reponse time

    - Packet delays on network due to queueing

        - Network switch queues the packets on a busy network link
        - Operating system queues the packets if all the CPU cores are busy
        - Virtual machine monitor queues the packets when operating system is paused
        - TCP flow control queues the packets at the sender before the data enters the network

    - Synchronous vs asynchronous networks

        - Telephone requires reliable network: low latency and enough bandwidth
        - Synchronous network: establish a circuit for a fixed, guaranteed amount of bandwidth allocated
        - But not good for TCP connection, which is optimized for bursy traffic and no particular bandwidth requirement
        - It wastes network capacity, causing slow transfer
        - TCP dynamically adapts the rate of data transfer to the available network capacity
        - Tradeoff of dynamic resource partitioning: latency guarantee at the cost of reduced utilization

- Unreliable Clocks

    - Monotonic vs Time-of-Day Clocks (physical clocks)

        - Time-of-Day clock

            - Synchronized with network time protocol (NTP)
            - Ideally the same timestamp among machines
            - But may be forcibly reset and jump back to previous time point (vary by e.g. temperature)
            - Unsuitable for measuring elapsed time
            - Can achieve high accuracy with GPS receivers, but requires significant effort and expertise

        - Monotonic clock

            - Guaranteed to always move forward, suitable for measuring a duration
            - Absolute value is meaningless, make no sense to compare between computers

    - Replying on Synchronized Clocks

        - Pitfalls

            - A day may not have exactly 86400s
            - Time-of-day clocks may move backward in time
            - Time on one node may be quite different from time on another node

        - Robust software needs to be prepated to deal with incorrect clocks
        - Logical clock based on incrementing counters is a safer alternative for ordering events
        - Clock reading has a confidence interval
        - Snapshot isolation needs a monotonically increasing transaction ID: uncertainty of clock accuracy if using timestamp

    - Process Pauses

        - How does the leader know it's still the leader: obtain a lease from other nodes and periodically renew
        - But lease may expire during unexpected program execution pause and the leader doesn't know

            - e.g. waiting for garbage collector which stops all running threads
            - e.g. virtual machine gets suspected and resumed later
            - e.g. operating system context-switches to another thread / virtual machine
            - e.g. synchronous disk IO wait

        - Response time guarantee

            - Deadline by which the software must respond
            - Sum all the worst-case execution times
            - Enormous testing and measurement must be done
            - Simply not economical and appropriate

- Knowledge, Truth, and Lies

    - The Truth is Defined by the Majority

        - A node cannot necessarily trust its own judgement of a situation (e.g. declaring nodes dead)
        - Many distributed algorithms rely on a quorum (voting among the nodes)
        - The leader and the lock (can only hold by one node): a node believes it's the chosen one doesn't mean the quorum agrees
        - Protect with feasing tokens: returned by the system and will be checked later during execution

    - Byzantine Faults

        - A node may be malfunctioning and not obeying the protocal
        - Safely assume the system doesn't have such fault
        - Most Byzantine fault-tolerant algorithm require a supermajority of over 2/3 of the nodes to be functioning correctly
        - Weak forms of lying: invalid message due to hardware issues, software bugs and misconfiguration

    - System Model and Reality

        - System model is an abstraction describes what things an algorithm may assume

            - Synchronous model: bounded network delay, process pauses, clock error (not realistic)
            - Partially synchronized model: behave like a synchronous model most of the time, but sometimes exceed (realistic)
            - Asynchronous model: No timing consumption (no clocks)

        - System models for node failures

            - Crash-stop faults: node can only fail by crashing (no response -> dead)
            - Crash-recovery faults: node may crash, or starting responding after some time
            - Byzanetine (arbitrary) faults: node may do anything, even try to trick and deceive other nodes

        - Real system: partially synchronized model with crash-recovery faults
        - Safety: nothing bad happens (uniqueness, monotonic sequence)
        - Liveness: Something good eventually happens (availability)

## CP9

Consistency and Consensus

The best way to build fault-tolerant systems is to find some general-purpose
abstractions with useful guarantees

The most important abstractions for distributed system is consensus:
getting all of the nodes to agree on something

- Consistency Guarantees

    - Weak guarantee: eventually conssitency
    - Stronger consistency models

        - Worse performance, less fault-tolerant
        - But easier to use correctly

    - Transaction isolation levels: avoiding race conditions due to concurrently executing transactions
    - Distributed consistency: coordinating the state of replicas in the face of delays and faults

- Linearizability

    - Make a system appear as if there were only one copy of the data
    - In a linearizable system we imagine there must be some point in time where the write happens, and all subsequent reads must return the new value
    - Linearizability vs serializability

        - Serializability is an isolation property of transactions
        - It guarantees that the transactions behave like they are executed in some serial order
        - Linearizability is a recency guarantee (新鲜度保证) on reads and writes of a register / object
        - Implementations of serializability based on 2PL or actual serial execution are typically linearizable
        - Serializable snapshot isolation is not linearizable since it reads from a consistent snapshot, which doesn't include more recent writes

    - Relying on Linearizability (as an important requirement for system to work correctly)

        - Locking and leader election in coordnation service (ZooKeeper, etcd)
        - Uniqueness constraints (primary key)
        - Cross-channel timing dependencies (could potentially detect the Linearizability violation)

    - Implementing Linearizable Systems

        - Really only use a single copy of the data (but not fault-tolerant)
        - Single-leader replication with synchronously updated followers are linearizable
        - Consensus algorithms are linearizable (see below)
        - Multi-leader / leaderless replication is not linearizable

    - The Cost of Linearizability

        - CAP theorem: applications that don't require linearizability can be more tolerant of network problems (either consistent or available when partitioned)
        - i.e. disconnected replicas cannot process requests if the application requires linearizability

- Ordering Guarantees

    - Operations are executed in some well-defined order
    - Ordering and Causality

        - Orders help preserve causality (因果关系): causes come before effect
        - Snapshot isolation provides casual consistency
        - Linearizable system has a total order of operations (no concurrent operations)
        - In a causally consistent system, two events are ordered if they are casually related. Concurrent events are incomparable (in different branches)
        - Linearizability implies causality
        - Casual consistency is the strongest possible consistency model that doesn't slow down due to network delays
        - Ordered (casually related) operations must be processed in that order on every replica
        - The system needs to track casual dependencies across the entire database (not a single key), can be achieved by generalized version vectors

    - Sequence Number Ordering

        - Tracking all casual dependencies has a large overhead
        - Instead, can use sequence numbers / timestamps (e.g. a logical clock) to order events
        - The generated sequence numbers must be consistent with causality: lamport timestamp
        - A pair of (counter, node ID)
        - Every node / client keeps tracks of the maximum counter it has seen so far. It increases that maximum counter by 1 during its new request / response
        - Casual dependency results in an increased timestamp
        - But timestamp ordering is not sufficient in practice: you have to check all other nodes for the operations to determine the order

    - Total Order Broadcast

        - A protocol for exchanging messages between nodes
        - Two safety properties

            - Reliable delivery: no message is lost. A message is delivered to all nodes if it's delivered to one node
            - Total ordered delivery: Messages are delivered to every node in the same order

        - Using total order broadcast: database replication, implement serializable transactions, creating logs (replication / transaction log)
        - Implementing linearizable storage using total order broadcast

            - Write: append a message to the log, read the log and wait for your appended message, check if the first occurrence of the request is yours
            - Read (could be stale if the store is asynchronously updated from the log) options:

                - Append the read message to the log, read the log and perform the actual read when the message is delivered to you
                - Or, fetch the position of the latest log message, perform the read when that message is delivered to you
                - Or, read from a replica that is synchronously updated on writes

        - Implementing total order broadcast using linearizable storage

            - Assume the storage has an atomic increment-and-get operation
            - Use the operation for a linearizable integer and attach it to message as the sequence number
            - Every node reads the messages with sequence numbers increasing 1 at a time

        - Linearizable compare-and-set / increment-and-get operation and total order broadcast are both equivalent to consensus

- Distributed Transactions and Consensus

    - Consensus: getting several nodes to agree on something (e.g. leader election, atomic commit)
    - Atomic Commit and Two-Phase Commit (2PC)

        - Atomic commit: all nodes agree on either commit or abort
        - Single node transaction commit: decide at the moment disk finishing writing the commit record
        - Multi-node transaction commit

            - A node must only commit once it's certain that all other nodes in the transaction are going to commit
            - A transaction commit is irrevocable
            - Achieved by two-phase commit (2PC)

        - Two-phase commit

            - Coordinator / transaction manager: a library within the same process that is requesting the transaction
            - two phases: prepare and commit

                - When application is ready to commit, the coordinator sends a prepare request to all nodes, asking them whether they want to commit
                - The coordinator sends the commit request iff all nodes reply yes, otherwise abort request is sent
                - The coordinator must send the decision (commit / abort) to all nodes (retry forever until it succeeds)

            - Commit point: coordinator writes the desicion (commit / abort) to the transaction log after receiving all the node replies, and before sending the decision to all the nodes
            - Coordinator failure: must wait for it to recover to complete the 2PC
            - Three-phase commit

                - Nonblocking atomic commit protocal that don't get stuck if the coordinator crashes
                - It assumes a network with bounded delay and nodes with bounded response time
                - Thus have a perfect failure detector (to tell whether a node has crashed)

    - Distributed Transactions in Practice

        - Tradeoff: provide an important safety guarantee vs bad performance and cause operational problems
        - Work well for database-internal distributed transactions (all nodes run the same database software)
        - Could be challenging for heterogeneous distributed transactions (allow diverse systems to be integrated)
        - Holding locks while in doubt, but could lead to lock being held forever (e.g. if the coordinator crashes)
        - Recover from coordinator failure

            - Orphaned in-doubt transactions cannot be resolved automatically
            - Only for an administrator to manually decide whether to commit or rollback

    - Fault-Tolerant Consensus

        - Everyone decides on the same outcome, and once you have decided, you cannot change your mind
        - Even if some nodes fail, the other nodes must still reach a decision
        - Requires a majority of the nodes are functioning correctly to safely form a quorum
        - Consensus algorithms

            - Decide on a sequence of values
            - Equivalent to total order broadcast (decide on each round the next message to deliver)

        - Limitations of consensus

            - Nodes voting uses synchronous replication, causing bad performance
            - Consensus system requires a strict majority to operate. It's blocked when network failure happens
            - Most consensus algorithm assumes a fixed set of nodes to participate the voting. Hard to add or remove nodes in the cluster
            - Consensus systems generally replies on timeouts to detect failed nodes. The could be wrong guesses and cause bad performance
            - Consensus algorithm is sensitive to network problems

    - Membership and Coordination Services

        - ZooKeeper and etcd: distributed key-value stores
        - Designed to hold small data that can fit entirely into memory
        - Data is replicated to all nodes using a fault-tolerant total order broadcast algorithm
        - Useful use cases

            - Allocating work to nodes: choose leader, assign resource partition to nodes
            - Service discovery: find out IP address to reach a particular service (as a service registry)
            - Membership service: determine which nodes are active and live members of a cluster

# Part 3 Derived Data

Systems taht store and process data

- System of record

    - Source of truth,hold authoritative version of data
    - Data is typically normalized

- Derived data systems

    - Take existing data and tranform / process it in some ways
    - Data is denormalized (values, indexes, materialized views)
    - Redundancy / deplication can have better performance

## CP10

Batch Processing

- Three Types of Systems

    - Service (online systems)

        - Response time is usually the promary measure of performance of a service
        - Availability is important

    - Batch processing system (offline system)

        - Take a large amount of data, run a job to process it and produce some output data
        - Job takes a while to run, often scheduled periodically
        - Throughput is the primary performance measure

    - Stream processing system

        - Between online and offline system
        - Stream job operates on events shortly after they happen (whereas a batch job operates on fixed-sized input)
        - Have lower latency than equivalent batch systems

- Batch Processing with Unix Tools

    - Simple Log Analysis

        - Small data: in-memory hash table
        - Large data: sort (make efficient use of disk)

    - The Unix Philosopy

        - A uniform interface

            - All programs use the same input / output interface (e.g. file)
            - The output of one program becomes the input of another progress

        - Separation of logic and wiring (loose coupling of programs)
        - Transparency and experimentation: make it easy to see what's going on (e.g. immutatble inputs)
        - Biggest limitation: only run on a single machine

- MapReduce and Distributed Filesystems

    - MapReduce jobs read and write files on a distributed filesystem (e.g. HDFS for Hadoop)
    - HDFS consists of a daemon process running on each machine, exposing a network service that allow other nodes to access its stored files
    - Replicated file blocks on multiple machines to tolerate machine and disk failure
    - MapReduce Job Execution

        - Mapper: extract key and values from input records
        - Reducer: takes the key-value pairs from the mapper, and produce output records grouping the same key
        - Distributed: both on mapper (on each database) and reducer (same key ends up on the same partitioned reducer)

    - MapReduce workflow

        - Chain MapReduces jobs together into a workflow
        - Output of a job becomes the input of the next job
        - Prior job must finish before the next job starts
        - Need workflow scheduler to handle job execution dependencies

    - Reduce-Side Joins and Grouping

        - Sort-merge joins

            - The mapper output is sorted by key (and database of records, secondary sort)
            - The reducer can directly merge records together

        - Group By

            - Used in aggregations
            - Set up the mapper so that the key in the key-value pair is the grouping key

        - Handling skew (hot keys / spots)

            - Spreading the work over several reducers (and another reducer in the next stage to group them all)
            - The cost is to replicate the other join inputs to multiple reducers
            - Or, user species the hot keys, which will be handled by mapper-side joins

    - Map-Side joins

        - Broadcast hash join

            - A large dataset joins a smaller one, which can fit into memory
            - Build in-memory hash table
            - Or store it in a read-only index on local disk, and the index can be cached in the os pages (no need to fit in memory in this case)

        - Partitioned hash join

            - All the inputs are partitioned in the same way
            - Records with the same key end up in the same partition
            - Sufficient for each mapper to read one partition of the data (not all)

        - Map-side merge joins

            - All the inputs and partitioned in the same way, and also sorted base on the same key
            - Mapper can do the merging part by reading files sequentially and matching records with the same key

        - MapReduce workflows with map-side joins

            - Map-side joins make more assumptions about the size, sorting, partitioning of the input datasets
            - Knowing the physical layout of the datasets is important for optimizing join strategies

    - The Output of Batch Workflows

        - Neither OLTP (scans a larger portion of data), nor OLAP (output is a data structure not a report)
        - Use cases of batch processing

            - Building search indexes
            - Key value stores (machine learning systems, e.g. classifier)

        - Philosophy of batch process outputs

            - Input is immutable to avoid side effects
            - Easy to rollback, backwards compatible, better feature development and maintainance
            - Automatic rescheduling in case of failure
            - Separation of wiring (decoupling on each stage)

    - Comparing Hadoop to Distributed Databases

        - Diversity of storage

            - Hadoop indiscriminates dumping data into HDFS, later figuring out how to process it
            - This shifts the burden of intepreting data to the consumer
            - By contrast, massive parallel processing (MPP) database requires careful up-front modeling of data and query pattern
            - This slows down the centralized data collection process

        - Design of processing models

            - MPP databases are monolithic, tightly integrated pieces of software that handles storage layout on disk, query planning, scheduling and execution
            - Hadoop can be built as a SQL query execution engine, or other types (e.g. machine learning system)

        - Designing for frequent faults

            - MPP databases abort the entire query if a node crash
            - Hadoop can tolerate the failure of MapReduce job failure by retrying the failed task
            - Hadoop also eager to write data to disk

                - This may good for fault tolerance, but incurs significant overheads
                - In Google, MapReduce workflows are more likely to retry if getting preempted by higher priority resources
                - This model achieves better resource utilization and greater efficiency at the cost of failure rate

- Beyond MapReduce

    - Intro of MapReduce

        - MapReduce is a fairly clear and simple abstraction on top of a distributed filesystem
        - Easy to understand but not easy to use -> higher-level programming models are built on top of it
        - Poor performance for some kinds of processing
        - Alternatives exist

    - Materialization of Intermediate State

        - Intermediate state: a means of passing data from one job to next
        - Materialization: write out the intermediate state to files
        - Downsides of MapReduce fully materializing intermediate state

            - A new job can start only if all its preceding jobs have finished (not pre-consume the input as soon as it's produced)
            - Mappers are often redundant
            - The state is replicated across several nodes

        - Dataflow engines (e.g. Spark, Tez, Flink)

            - Handle the entire workflow as one job, consisting of operators
            - Operators are more flexible than mappers and reducers

                - Sorting is performed only if actually required
                - No unnecessary map tasks
                - The scheduler has an overview of the pipeline so that it can make locality optimizations
                - May only need to keep the intermediate state in memory / local disk
                - Operators can start as soon as its input is ready

            - Some avoid writing intemediate state to HDFS

                - Recompute the data from others that are still available
                - The computation needs to be deterministic
                - May not be a good idea if the intermediate data is small or the recomputation is very CPU-intensive

    - Graphs and Iterative Processing

        - Goal is to have an offline processing / analysis on an entire graph
        - MapReduce uses an inefficient iterative style

            - The scheduler checks the finish condition on every run
            - It will always reads the entire dataset on every run, even if it only changes a small portion

        - The Pregel processing model

            - Bulk synchronous parallel (BSP) model
            - A vertex remembers its state in memory from one iteration to next
            - Function only needs to process new incoming messages (causing the state change)
            - The only waiting is between iterations (for all messages to be delivered)
            - Acheving fault tolerance: periodically check the states of all vertices at the end of an iteration (e.g. write to disk so that it can rollback in case of a crash)
            - Parallel execution: cross-machine communication is unavoidable since it's hard to partition the vertices, leading to more overhead than a single machine

    - High-Level APIs and Languages

        - Pig, Hive, Cascading, Crunch
        - Less laborious
        - Able to move the new dataflow engine without the need to rewrite job code
        - Allow interactive use (write analysis code to observe)
        - More productive for humans and improve job execution efficiency
        - Incorporating declarative aspects to high-level APIs

            - e.g. specifies joins in a declarative way (relational operator)
            - The query optimizer decides how they can best be executed
            - Or use column-oriented storage layout for filters
            - Built into a large ecosystem of libraries (e.g. parsing, data analysis, numerical algorithm)

## CP11

Stream Processing

- Intro

    - In reality, a lot of data is unbounded because it arrives gradually over time
    - Daily batch process could be too slow for impatient users
    - Stream refers to data that is incrementally made available over time

- Transmitting Event Streams

    - Intro

        - The input and output in batch are files
        - Stream: a sequence of records / events
        - An event is generated by a producer / sender / publisher, and then potentially processed by multiple consumers / recipients / subscribers
        - Related events are usually grouped together into a topic / stream
        - Consumers get notified when new events appear

    - Messaging System

        - Publish / subscribe model: allow multiple producers to send messages to the same topic, also multiple consumers to rereceive messages in a topic

        - Producer send messages faster than the consumers

            - Drop messages
            - Buffer messages in a queue (crash, or write to disk if queue is too large)
            - Apply backpressure (flow control, blocking the producer)

        - Node crashes or temporarily goes offline

            - Replication / write to disk
            - Or lose few messages (higher throughput and lower latency)

        - Direct messaging from producers to consumers

            - Direct network communication between producers and consumers
            - Requires the application code to be aware of the possibility of message loss
            - Reliable UDP: low lantency, made reliable at application level
            - Unreliable UDP: collect mertics and monitor
            - HTTP / RPC request: if the consumer exposes a service on the network

        - Message brokers (widely-used alternative)

            - Run as a server, with producers and consumers connecting to it as clients
            - System can tolerate client nodes come and go (connect, disconnect, crash)
            - Durability is handled by the broker (keep message in memory / write to disk)
            - Generally allow unbounded queueing
            - Consumers are generally asynchronous: producer only wait for the broker to confirm it has buffered the message, not wait for the broker to process the message

        - Message brokers compared to the databases

            - Databases ususally keeps the data; brokers delete a message when it's been delivered to the consumers
            - Most brokers assume their working set is fairly small (queues are short)
            - Databases often support secondary indexes and various ways of searching for data, while brokers often support ways of subscribing to topics
            - The result of querying a database is based on point-in-time snapshots. Brokers don't support queries, it only notify clients when data changes

        - Multiple consumers

            - Multiple consumers read messages in the same topic
            - Load balancing: each message is delivered to one of the consumers, and consumers can share the work of processing the messages in the topic
            - Fan-out: each message is delivered to all of the consumers
            - Two patterns can be combined

        - Acknowledgements and redelivery

            - To ensure the message is not lost, a client must eplicitly tell the broker when it has finished processing a message so that the broker can remove it from the queue
            - The broker will deliver the message again if not receiving the acknowledgement
            - The combination of load balancing with redelivery could lead to messages being reordered

    - Partitioned Logs

        - Traditional messaging

            - Asynchronous form of RPC
            - The exact order of message processing is not important
            - No need to go back and read messages again after they have been processed

        - Using logs for message storage

            - A producer sends a message by appending it to the end of the log, and a consumer receives messages by reading the log sequentially
            - Logs can be partitioned to scale to higher throughput
            - A topic can be defined as a group of partitions that carry messages of the same type
            - Messages within a partition are totally ordered. There is no ordering guarantee across different partitions

        - Logs compared to traditional messaging

            - Downsides

                - num_consumers in a topic can be at most num_partitions in that topic, csince messages within the same parititon are delivered to the same consumer
                - If a single message is slow to process, it holds up the processing of subsequent messages in that partition

            - Not good for ...

                - Message is expensive to process
                - Parallelize processing on a message by message basis
                - Message ordering is not so important

            - Good for ...

                - High message throughput and each message is fast to process
                - Message ordering is important

            - All messages that need to be ordered consistently need to be routed to the same partition

        - Consumer offsets

            - Less bookkeeping overhead: broker only periodically records the consumer offsets
            - No need to track acknowledgements for every single message
            - Similar to log sequence number

        - Disk space usage

            - Circular buffer to archive storage
            - Slow consumer may potentially points to a deleted segment (rarely happen)
            - The throughput is more ot less constant (all messages write to disk)
            - In contrast to systems that keep message in memory by default and only write to disk if the queue grows too large

        - When consumer cannot keep up with producers

            - Log-based approach is a form of buffering with a large, fixed-size buffer (disk space)
            - Can monitor how far a consumer is behind the log, and raise the alert if it's too much
            - Only that slowed consumer is affected

        - Replaying old messages

            - Start a copy of log and record the consumer offset for replay
            - Like batch process
            - Allow more experimentation and easier recovery from errors and bugs
            - Good tool for integrating dataflows

- Databases and Streams

    - Keeping Systems in Sync

        - Same or related data appearing in several places need to be in sync
        - Periodic full database dumps are too slow
        - Alternative is for application code writes to systems when data changes (dual writes)

            - One problem is race condition, need concurrency detection mechanism (e.g. version vectors)
            - Another one is that one of the writes may fail while others succeed, need atomic commit

        - Can have one leader (e.g. database) and others (e.g. search index) to be followers

    - Change Data Capture (CDC)

        - The process of observing all data changes written to a database and extracting them in a form in which they can be replicated to other systems
        - Changes are made as a stream, immidiately as they are written
        - Implementing data capture

            - Log-based message broker: preserve message ordering
            - Database triggers: fragile and have significant performance overheads
            - Parsing the replication log: robust but with challenge (e.g. handling schema change)
            - Usually asynchronous: adding a slow consumer won't affect the system, but may face replication lag

        - Initial snapshot

            - Reconstruct the state from start of the log: require too much disk space and take too long
            - Or, start with a consistent snapshot (state) and the corresponding know position / offset in the log

        - Log compaction

            - Keep a limited amount of history of the log
            - The system periodically looks for log records of the same key, and only keep themost recent update
            - Run in the background

        - API support for change streams: database support change streams in the API

    - Event Sourcing

        - Involve storing all changes to the application state as a log of change events
        - Different from CDC in the level of abstraction

            - CDC extracts log in low level (e.g. parsing) that the application is not aware of (record change)
            - Event sourcing happends on application level, on the basis of immutable events written to the log (event)

        - Deriving current state from the event log

            - Application need to take the log of events and transform it into applicaiton state
            - The transformation process must be deterministic
            - Log compaction needs to be handled differently: full event history is needed
            - Application needs some mechanism for storing snapshots of the state from log of events

        - Commands and events

            - Command is when user request arrives
            - The system validates the command and accepts it, which turns it into durable and immutable event

    - State, Streams, and Immutability

        - The log of all changes, the changelog, represents the evolution of state over time
        - Advantage of immutable events: append-only log of immutable events make it easier to debug and revocer from the problem
        - Deriving several views from the same event log

            - Derive several read-oriented representations from the same log events
            - Have an explicit translation step from an event log makes it easier to evolve the application over time

        - Concurrency control

            - The consumers of the event log are usually asynchronous (reading own writes?)
            - One way is to make it synchronous: same storage system
            - Or use total order broadcast to implement linearizable storage
            - But, derive state from the event log is easy if the user action can be made atomic

        - Limitations of immutability

            - Support for point-in-time snapshots (e.g. version control system)
            - Bad performance for workloads that have a high rate of updates / deletes on a small datasets
            - Or some data needs to be deleted for administrative or legal reasons (may rewrite history / lazy deletion)
            - Deletion -> make it harder to retrieve

  - Processing Streams

      - What to do with streams once you have it

          - Take the event data and write it into database / cache / search index ... to be queried by other clients
          - Push the event to users (e.g. email, notification)
          - Process one or more input streams to produce one or more output streams

      - Stream never ends

          - Sorting does not make sense (e.g. sort-merge join)
          - Fault-tolerance mechanism needs to change

      - Uses of Stream Processing

          - Monitoring purpose

              - Fraud detection system
              - Trading system
              - Manufactoring system
              - Military and intelligence system

          - Complex event processing: analyzing event streams for certain event patterns
          - Stream analytics: aggregations and statistical metrics over a large number of events
          - Maintaining materialized views: derive an alternative view on to some datasets, and update the view whenever the underlying data changes
          - Search on streams: search for individual events based on complex criteria (e.g. full text search)
          - Message passing and RPC: message passing systems base on events and messages

      - Reasoning About Time

          - Stream processor deals with time for analytic purposes
          - Batch process looks at the timestamp embedded in each event
          - Stream processing frameworks use local system clock

              - Simple
              - Reasonable if the delay between event creation and processing is short
              - But break down if there is any significant processing lag

          - Event time vs processing time

              - Processing may be delayed (e.g. queueing, network faults, performance issue, consumer restart...)
              - Message delay leads to unpredictable ordering of messages
              - Confusing event time and processing time leads to bad data

          - Knowing when you are ready

              - One problem of defining windows is that you can never know when you have received all the events
              - Handling straggler events

                  - Ignore. Track the number of dropped events as a metric, and alert if there is too much
                  - Publish a correction

          - Type of windows

              - Tumbling window: fixed length with every event belonging to exactly one window
              - Hopping windoe: fixed length that allows windows to overlap
              - Sliding window: contain all the events that occur within some interval of each other (random start and end point)
              - Session window: no fixed length, group all events for the same user that occcur closely together

      - Stream Joins

          - Stream-stream join (window join)

              - e.g. user search vs user click, where click rate = num_click / num_search within a time frame
              - Can choose a suitable window for the join

          - Stream-table join(stream enrichment)

              - e.g. user activity vs user profile, enriching the activity events with information from the database
              - May look up from a remote database, but could be slow and overload the database
              - Can load a local copy of the database into the stream processor instead
              - The local copy needs to be up to date, which can be solved by change data capture

          - Table-table join

              - e.g. tweet event vs tweet followers, to get the user timeline
              - Also is a materialized view for a query that joins two tables
              - The timeline is a cache of the result of the query, updated every time the underlying tables change

          - Time-dependence of joins

              - What if the order of the event matters (e.g. join becomes undeterministic)
              - Partitoned log: event orders within a single partition is preserved
              - Slowly changing dimension (SCD): use a unique identifier for a particular version of the join record
              - It makes the join deterministic, but log compaction is not possible

      - Fault tolerance

          - Microbatching and checkpointing

              - Microbatching: break the stream into small blocks and treat each block like a mini-batch process
              - Implicitly provide a tumbling window equals to the batch size
              - Checkpointing: periodically generate rolling checkpoints of the state and write them to durable storage
              - No forcing of particular window size
              - Both approach cannot be able to discard the output of a failed batch (e.g. write to database, send message, email the user)

          - Atomic commit revisited

              - Ensure all outputs and side effects of processing an event take effect iff the processing is successful
              - These things need to all happen atomically, or none of them happen

          - Idempotence

              - Perform multiple times, and it has the same effect as if it's performed once
              - Even if an operation is not natually idempotence, it can often be made idempotence with a bit of extra metadata
              - The system can retry the failed idempotence event without worrying being processed multiple times

          - Rebuilding state after a failure

              - Any stream process that requires state must ensure that the state can be recovered after a failure
              - Can keep the state as remote database
              - Or, keep the state local to the stream processor, and replicate it periodically
              - Or some cases, it can be rebuilt from the input stream / log-compacted change stream

## CP12

The Future of Data Systems

- Data Integration

    - The most appropriate choice of software tools (database) depends on circumstances (usage pattern)
    - Also, data is used in several different ways
    - Combining Specialized Tools by Deriving Data

        - Reasoning about dataflows: clear about the inputs and outputs
        - Derived data vs distributed transaction

            - Distributed transaction: decide ordering of writes with mutex locks; ensure changes take effect exactly once with atomic commit; provide linearizable guarantee (e.g. reading own writes)
            - Derived data: CDC and event sourcing use logs for ordering; ensure changes take effect exactly once with deterministic retry and idempotence; no same guarantee with asynchronous updates

        - The limits of total ordering: total order of events == total order broadcast == consensus (most algorithms are designed for single node, multiple nodes under open research)

        - Ordering events to capture causality: events order matters when they are causally related

    - Batch and Stream Processing

        - Goal of data integration: make sure that data ends up in the right form in all the right places
        - Batch and stream processors are the tolls for achieving this goal
        - Maintaining derived state

            - Batch processing: encourage deterministic, pure functions, treating inputs as immutable and outputs as append-only
            - Stream processing: extend operators to allow managed, fault tolerant state
            - Asynchrony makes the system based on log robust: allow a fault in one part to be contained locally. Distributed transaction fails all if one participant fails
            - Cross-partition communication (e.g. secondary index) is more reliable and scalable if the index is maintained asynchronously

        - Reprocessing data for application evolution

            - Provide a good mechanism for maintaining a system, evolving it to support new features and changed requirements
            - Restructure a dataset into a completely different model in order to better serve new requirements
            - Derived views allow gradual evolution, which is easily reversible if something goes wrong

        - The lambda architecture

            - Combine the batch and stream processing (fast approximate vs slow exact)
            - The stream processor comsumes the events and quickly produces an approximate update to the view
            - The batch process later consumes the same set of events and produces a corrected version of the derived view
            - Practical problems

                - Significant additional effort
                - Merging two results could be hard if the data / number of operators is large
                - Doing so frequently is expensive on large datasets

        - Unifying batch and stream processing

            - Require the following features

                - The ability to replay historical events through the same processing engine that handles the stream of recent events
                - Exactly-once semantics for stream processors
                - Tools for windowing by event time, not processing time

- Unbundling Databases

    - Abstract level: database / operating system stores data, and allows you to process and query that data
    - Treat information management differently

        - Unix: logical but fairly low-level hardware abstraction
        - Relational database: high level obstraction that hide the complexities of data structure on disk, concurrency, crash recovery ...

    - Composing Data Storage Technologies

        - Parallels between the features built into databases and the derived data systems
        - Creating an index

            - Setting up a new follower replica / bootstrapping change data capture in a streaming system
            - Reprocess the existing dataset and derives the index as a new view onto the existing data

        - The meta-database of everything

            - Dataflow looks like one huge database (e.g. batch / stream / ETL keeps the indexes or materialized views up to date)
            - Two avenues by which different storage and processing tools composed into a cohesive system

                - Federated databases: unifying reads (easy to address read-only querying across different systems)
                - Unbundled databases: unifying writes (synchronize writes across disparate technologies)

        - Make unbundling work

            - Federated read-only querying requires mapping one data model into another (manageable)
            - Synchronizing writes requires distributed transaction across heterogeneous storage systems (wrong solution, lack of standardized transaction protocal)
            - Asynchronous event log with idempotence writes is a much more robust and practical approach (loose coupling between the various components with log-based integration)

                - System level: robust to outage or performance degradation of individual components
                - Human level: allow different software components and services to be developed, improved, and maintained independently by different teams. Event logs provide an interface to capture consistency and are applicable to almost any data

        - Unbundled vs integrated system

            - The goal of unbundling is not to compete with individual databases on performance for particular workloads
            - The goal is to allow you to combine several different databases in order to achieve good performance for a much wider range of workloads than is possible with a single piece of software
            - It's about breadth, not depth

        - What is missing

            - Unbundled-database equivalent of the Unix shell
            - High level language for composing storage and processing systems in a simple and declarative way

    - Designing Applications Around Dataflow

        - When a record in a database changes, we want any index / materialized view related to that record also updated
        - Application code as a derivation function (triggers, stored procedures, UDFs)
        - Separation of application code and state

            - It make sense to have some parts of a system that specialize in durable data storage, and other parts that specialize in running application code
            - The two can interact while still remaining independent
            - Cannot subscribe to changes in a mutable variable in a database, can only read it periodically

        - Dataflow: interplay between state changes and application code

            - Unbundling the database: take the derived datasets outside the primary database
            - e.g. caches, full-text search indexes, machine learning and analytic systems
            - Use stream processing and message systems for this purpose
            - Maintaining the derived data is not the same as asynchronous job execution

                - The order of the state changes is often important (many message brokers don't have this property)
                - Fault tolerance is essential for derived data (message delivery and status updates must be reliable)

            - Modern stream processors can provide these ordering and reliable guarantees at scale

        - Stream processors and services

            - Application development: break down the functionality into a set of services that communicate via synchronous network requests: organizational scalability with loose coupling
            - Similar to composing stream operators into dataflow systems: one-directional, asynchronous message streams
            - Better fault tolerance
            - Better performance: replace synchronous network request to another service with aquery to a local database

    - Observing Derived State

        - Dataflow system gives you a process for creating derived datasets and keeping them up-tp-date
        - This is the write path
        - Read path: query it again at a later time
        - The derived state is the place that the write path and the read path meet
        - It represents a trade-off between the amount of work that needs to be done at write time vs read time
        - Materialized views, indexes and caching: they shift the boundary between the read path and the write path (more work on the write path by precomputing result to save efforts on the read path)
        - Stateful, offline-capable clients

            - Do as much as possible using local database without requiring network connection
            - Sync with remote servers on the background when network connection is available
            - Think of the device state as a cache of the server state

        - Pushing state changes to clients

            - The browser is a stale cache of the web page unless you poll for changes
            - Or, the server can actively push messages to the browser as long as it remains connected
            - Or, read for initial states, and rely on a stream of state changes thereafter
            - If offline, it can catch up with consumer offsets from log-based message brokers later

        - End-to-end event streams

            - Recent tools (e.g. React) already manage internal client-side state by subscribing to a stram of events from server responses
            - It's natural to allow servers to push state-change events to client-side pipeline
            - This would form the end-to-end write path for state changes
            - The challenge is the current model of stateless client + request/response interaction with databases
            - Databases support read/write operations but not the ability to subscribe to changes
            - Rethink the way to build the system: request/response interaction -> publish/subscribe dataflow

        - Reads are events too

            - Writes go through the event log, reads directly go through the datastore being queried
            - But it's also possible to represent read requests as a stream of events
            - Recording a log of read events

                - Good to track casual dependency and data provenance across a system
                - But incur additional storage and I/O cost

        - Multi-partition data processing

            - No need to collection a stream for a single partition queries
            - It's possible for distributed execution of complex quries that need to combine data from several partitions
            - It takes the advantage of message routing, partitioning, and joining that is already supported by stream processors

- Aiming for Correctness

    - We want to build applications that are reliable and correct
    - Transactional properties of atomicity, isolation, and durability
    - Simpler if the system can tolerate occational corrupting or losing data
    - Stronger assurance of correctness (serializability and atomic commit)
    - The cost is that they only work in a single datacenter (limit the scale and fault-tolerance properties)
    - The End-to-End Argument for Databases

        - Exactly-once execution of an operation

            - Exactly-once / effectively-once semantic
            - Processing twice is a form of data corruption
            - Make the operation idempotent: need to maintain additional metadata
            - Ensure fensing when failing over from one node to another

        - Duplicate suppression: only work within the context of a single TCP connection
        - Uniquely identifying requests

            - Not sufficient to rely just on a transaction mechanism from the database
            - Need to consider the end-to-end flow of the request
            - A unique identifier passed all the way from the end-user client to the database

        - The end-to-end argument

            - Also apply to checking the integrity of the data
            - Cannot detect corruption due to bugs in the software
            - The lower level features (e.f. TCP duplicate suppression) are still useful, since they reduce the probability of problems at higher levels

        - Applying end-to-end thinking in data systems

            - Explore fault-tolerance abstractions that make it easy to provide application-specific end-to-end correctness properties
            - Also maintain good performance and good operational characteristics in a large-scale distributed environment

    - Enforcing Constraints

        - Uniqueness constraints require consensus

            - Make a single node the leader, and put it in charge of making all the decisions
            - Can be scaled out by partitioning based on the value needs to be unique (all requests with the same id are routed to the same partition)
            - Not good for multi-master replication: different masters concurrently accept conflicting writes

        - Uniqueness in log-based messaging

            - Log ensures all consumers see messages in the same order
            - A stream processor consumes all the messages in a log partition sequentially, and the log is partitioned based on the unique value
            - Any writes that may conflict are routed to the same partition and processed sequentially

        - Multi-partition request processing

            - Ensure an operation is executed atomically, while satisfying constraints when several partitions (tables) are involved
            - Atomic commit across all partitions in required in the tranditional database approach, throughput is likely suffer
            - Equicalent for partitioned logs (Log the request in a single message, as single-object writes are atomic)

                - Client sends the request with an unique id
                - A stream processor reads the log of requests, and include the request id in all its output streams
                - Further processors comsume the stream and apply the change, deduplicate by the request id

    - Timeliness and Integrity

        - The correctness of the uniqueness check does not depend on whether is sender of the message waits for the outcome (sync/async)
        - The waiting only has the purpose of synchronously informing the sender whether the uniqueness check succeeded
        - Consistency

            - Timeliness: ensuring the users observe the system in an up-to-date state (temporary inconsistency, eventual conssitency)
            - Integrity: the absence of corruption (e.g. no data loss, no contradictory or false data), permanent inconsistency, explicit checking and repair is needed

        - In most applications, integrity is much more important than timeliness
        - Correctness of dataflow system

            - ACID transactions provide both timeliness (linearizability) and integrity (atomic commit) guarantees
            - Event-based dataflow system decouple timeliness and integrity
            - Exactly-once / effectively-once semantics (fault-tolerant message delivery and deplicate suppression (e.g. idempotent operations) mechanism for oreserving integrity)
            - Last example

                - Represent write operation as a single message to be atomic
                - Derive all other state updates from that single message using deterministic derivation functions
                - Pass a client-generated id through all levels of processing for end-to-end duplicate suppression
                - Make messages immutable and allow derived data to be reprocessed from time to time, for it easier to recover from bugs

        - Loosely interpreted constraints

            - Enforcing a uniqueness constraint requires consensus, typically implemented by funneling all events in a particular partition through a single node
            - Many application can get away with much weak notion of uniqueness

                - Compensating transaction: change to correct a mistake
                - Making appology
                - Expected as part of the business (e.g. airline overbook)
                - Other higher level, easy to achieve constraint (e.g. bank withdrawal limit per day)

            - The tranditional model of checking all constraints before writing the data is unneccessarily restrictive
            - A reasonable choice is to write optimistically, and to check the constraint after the fact
            - These applications do require integrity, but don't require timeliness on the enforcement of the constraints

        - Coordination-avoiding data systems

            - Two observations

                - Dataflow systems can maintain integrity guarantees on derived data without atomic commit, linearizability, or synchronous cross-partition coordination
                - Although strict uniqueness constraints require timeliness and coordination, many applications are actually fine with loose constraints that may be temporarily violated and fixed up later, as long as integrity is preserved throughout

            - Dataflow system can provide data management service without requiring coordination, while still giving strong integrity guarantees
            - Such coordination-avoiding data systems can achieve better performance and fault tolerance than systems that need to perform synchronous coordination
            - Tradeoff between coordination and constriants: inconsistencies (appologies to make) vs availability problems (performance)

    - Trust, but Verify

        - Previous assumption: certain things might go wrong, but other things won't
        - Process can crash, machines can suddenly lose power, and network can arbitrarily delay or drop messages
        - Data written to disk won't lost, data in memory won't corrupt, multiplication instruction of CPU always returns the correct result (hardware issues)
        - Maintaining integrity in the face of software bugs
        - Don't just blindly trust what they promise

            - Data corruption is sooner or later
            - Auditing: checking the integrity of data
            - Mature system tend to consider the possibility of unlikely things going wrong, and manage that risk

        - A culture of verification: seeing more self-validating / auditing systems that continually check their own integrity, rather than relying on blind trust
        - Designing for auditability

            - Transaction can mutate several values in a databse, hard to audit
            - Event-based system provides better auditability
            - Resulting state updates are derived from single immutable events, which can be made deterministic and repeatable
            - Make data provenance clearer, easy to rerun and debug

        - The end-to-end argument again: checking the integrity of data systems is best done in an end-to-end fashion
        - Tools for auditable data systems

            - Like cryptocurrency and blockchain system
            - The replica continually check each other's integrity and use a consensus protocal to agree on the transactions that should be executed

- Doing the Right Thing

    - Aware of the consequences: the ethical responsibility
    - Predictive Analytics

        - Bias and discrimination

            - The patterns learnt by the system could be opaque
            - If there is a systematic bias in the input data, the system could learn and amplify the bias in its output

        - Responsibility and accountability

            - Automated decision making opens the question of responsibility and accountability
            - It's harder to understand how a particular decision is made and whether it's been treated in an unfair or discriminated way
            - Much data is statistical in nature, individual cases could be wrong even if the whole probability distribution is correct

        - Feedback loops

            - Recommendation system: people end up seeing opinions they already agree with
            - Breed stereotypes, misinformation and polarization
            - Things get worse in self-reinforcing feedback loops

    - Privacy and Tracking

        - Ethical problems in data collection itself
        - For application systems, sometimes the advertisers are the actual customers, and users' interest take the second place (marketing purpose)
        - Surveillance: examine the data collection whether it's surveillance to help understand the relationship with the data collector
        - Consent and freedom of choice

            - Without understanding what happens to the data, users cannot give meaningful consent
            - Data is extracted from users through a one way process, not a relationship with true reciprocity

        - Privacy and use of data

            - Having privacy does not mean keeping everything secret
            - It means having the freedom to choose which things to reveal to whom, what to make public, and what to keep secret

        - Data assets and power: if targeted adcertising is what pays for a service, then behavioral data about people is the service's core asset
        - Remembering the Industrial Revolution

            - The Industrial Revolution came about through major technological and agricultural advances: economical growth and improvements of living standard
            - It also come with major problems: air and water pollution, establishment of social classes causing powerty of some groups
            - It takes long time before safeguards were established (e.g. environmental protection regulations, safety protocals for workplace, outlawing child labors, health inspection for food)
            - For the information age, the collection and use of data is one of the problems

        - Legislation and self-regulation

            - Data protection laws might be able to help preserve individuals' rights
            - We need a cultural shift in the tech industry with regard to personal data
            - We should allow each individual to maintain their privacy (i.e. their control over own data)
