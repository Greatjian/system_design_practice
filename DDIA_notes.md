# DDIA Notes

## Catalog

**Part 1 Foundations of Data Systems**

- [CP1 Reliable, Scalable, and Maintainable Applications](#CP1)
- [CP2 Data Models and Query Languages](#CP2)
- [CP3 Storage and Retrieval](#CP3)

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
