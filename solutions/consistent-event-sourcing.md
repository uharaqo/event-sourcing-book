---
description: Eventual consistency is the key to Event Sourcing... Is that true?
---

# Consistent Event Sourcing

Many articles on Event Sourcing emphasize high-throughput writes, scalability and flexibility of read models. But these are characteristics of key-value store and CQRS. I view that these are not requirements of Event Sourcing and the design is just one of the possible designs that leverages Event Sourcing.

The most important design decision is to store state transitions as immutable records. The simple event record schema is also practically important. It provides schema-less data models (there's no restriction to the event data model).

There used to be only a file system. 3NF relational data models and SQL enabled us to write high quality applications based on ACID transactions. Web frameworks increased productivity and standardized database management patterns. NoSQL databases proved that consistency level is tunable for each use case. NewSQL proved that a distributed ACID transaction can be supported even in a distributed environment.

Of course, there are use cases that can leverage the strength of NoSQL-like design such as high-throughput writes, low latency, availability, and scalability. But there are many use cases in which strong consistency is more important, especially on the command side. What if we choose consistency over availability just like the choice of NewSQL over NoSQL?

In the worst case such as a network partition, the system might need to halt. Transactions would add non-trivial latency and throughput would decrease. Then, do the applications that are using RDBMS need to migrate to a key-value store?

The most important functionality of many server applications, such as a CRUD system, is to manage state. Typically, consistency is much more important than availability during rare network issues and maintenance periods. And now we have a solution to easily manage distributed ACID transactions.

Though the performance characteristic of choosing the NoSQL-like design will be lost, there are many benefits of using Event Sourcing.

* Persisted events are never lost
* State transitions based on pure functions
* CQRS: flexible and scalable read models and event driven design
* Immutable and intuitive data models of commands, events, and states
* The design enforces consistent model driven design and development
* Reliable system built on top of the simple and reasonable design
* Easy to debug and test by replaying events locally
* The simple data schema enables us to easily run end-to-end tests

When we create a typical CRUD application with REST and RDBMS, though it looks simple at the beginning, database schemas and data access code get more and more complicated. That complexity is mainly caused because of the first design choice, CRUD operations on a resource which is typically a representation of a table in a DB. We typically spend a lot of time on managing DB schemas, data management, data accessors, SQLs, glue code between APIs and the database. Management of test data is one of the most difficult problem in development. Integration and end-to-end testing requires some skills and not so many people invest enough time on them.

I believe that [a simple design makes things easier](https://youtu.be/LKtk3HCgTa8). REST is just a convention and APIs are not always well organized hierarchically, especially for commands. RDBMS is a powerful tool but it requires lots of effort to manage data, code, and tests. Though most people think it's easy to write CRUD applications with the approach, not so many people are aware that they are building quite complicated system that nobody can understand. Complexity leads to unpredictability, which results in bugs. Debugging and bug fixes take much more time than adding a feature, and it typically causes another problem and make the design more complex. Of course, you might be able to manage that. But can your team still keep the design quality without you?

By choosing Event Sourcing with ACID transactions, I'm viewing that we can eliminate these pain points and we will be able to focus on essential parts of the design and implementation. And even a new grad engineer who doesn't have much experience can quickly start contributing to the application.
