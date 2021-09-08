# DDIA Notes

## Catalog

**Part 1 Foundations of Data Systems**

- [CP1 Reliable, Scalable, and Maintainable Applications](#CP1)

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
