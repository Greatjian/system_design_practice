# DDIA Notes

## Catalog

**Part 1 Foundations of Data Systems**

- [CP1 Reliable, Scalable, and Maintainable Applications](#CP1)
- [CP2 Data Models and Query Languages](#CP2)

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
