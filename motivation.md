# Motivation

## State is the biggest cause of complexity

We create a system by integrating sub-systems. We start from small components and compose a bigger component from them. We need to make sure that each component works as we expect. Even one line of wrong code could break the entire system.

We can only understand a small and simple component. We can also understand a component composed of multiple simple components. But we cannot fully understand a complex component. A composite of multiple complex components gets exponentially complex.

Side effects are a major cause of complexity. State management is especially important. If a component reads or mutates a state, we need to understand all the states and the behavior for each state. If we overlook some cases, we have unknown behaviors in the component. A component that contains unknown components also have unknown behaviors. This is how a system gets complicated. [We need to keep each component simple](https://youtu.be/LKtk3HCgTa8).

In other words, separating out state management from other code makes the code much easier to be understood. If a component does not depend on any internal or external state, the only thing it does is a conversion from each input to the output. A component that uses those stateless components can be also stateless.

In summary, we can build a predictable component by composing smaller components without side effects. By composing a system based on the reliable components, we can build a high quality system and maximize productivity. The design for state management is the key for that. The benefit of having such a design will quickly overcome the cost of designing the system since we can eliminate the complexity that increases over time.

<figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

## Database is a huge state

An application changes its behaviors based on its internal state kept in memory and external states fetched from external systems. Typically, an application does not keep all the states in memory and delegate the state management to a database.

RDBMS is the most common database choice. By design, tables in a RDBMS are often tightly coupled. The code that manages those tables can easily become entangled. Furthermore, most applications keep data as a directed graph which requires mapping to and from relational schemas.

Without having a design to solve these problems, the database can easily become a huge shared state. Increasing number of queries and unmanaged state transitions make it hard to understand the patterns of state and behaviors.

A widely used approach is to use the RDBMS like an object storage. For instance, the Aggregate pattern logically divides tables as aggregates. A single transaction manages only one aggregate root. However, this pattern limits the database's capabilities, and aggregates and data schemas get tightly coupled, resulting in design inflexibility on both application and database sides. Although a NoSQL database can simplify some problems, it introduces other design challenges.

Another impedance mismatch is in the service's APIs. APIs, domain objects, and tables an easily get tightly coupled. Typically, we need to maintain similar data models in several layers.&#x20;

Testability is a good indicator of a system's complexity. Setting up test data is one of the hardest problem in development. There are too many possible scenarios to be tested. Even if we list up all test cases, maintaining the test data on each code and database change is quite hard. Integration testing is a time-consuming and error-prone task. Developers tend to give up testing most of possible scenarios.

Maintainability is another indicator of the complexity. When there is a change to requirements, many modules need to be updated: APIs, domain objects, data accessors, data models in each layer, data schemas, ... Ideally, components that change together should be in a single module.

The complexity comes from the complex state in a database and the integration of inconsistent designs. Although the code and data schemas might seemingly look simple, the number of possible combinations of requests and states is too enormous to be fully comprehend. Developers make many mistakes due to the complexity and spend non-trivial amount of time for debugging.

We should consider if these conventional design choices are worth the cost for each application. Is there any design such as:

* Single consistent design for APIs, domain logic, data accessors, and data schemas
* Design that reduces the complexity of the states in a database
* Design that is flexible and generic enough to be applied to various use cases
* Data schemas that can be easily set up for testing
* Clear separation of concerns for maintainability

## Is Event Sourcing an overengineering?

Event Sourcing seemed a great solution. It's designed for clear separation of concerns and strong cohesion / loose coupling.

* Queries are separated and they can be flexibly designed on demand
* Simple immutable data models: commands, events, and states on the write side
* Commands represent the intent of callers
* Command handlers generate events based on a command and state
* Immutable events that are easy to replicate

With this design, we can focus on essential workflows. For example, `RegisterUser` -> `UserRegistered`, `UpdateEmail` -> `EmailUpdated` , `UpgradePlan` -> `PlanUpgraded`, ...  It is simple enough to be understood and developers can focus on writing declarative code. Code becomes declarative and straight forward. It solves many problems mentioned above.

However, many proponents of Event Sourcing claim that it should be used for limited use cases because too many factors need to be taken into account and it sometimes requires a tailored solution even for a simple feature (e.g. Saga, Process Manager, compensations, schema evolution, ...). Although I researched numerous articles, books, and videos, I could not find a generic solution to the problems.

## Proposal: Consistent Event Sourcing

After spending many months, I discovered that there are many variations of Event Sourcing designs and each of them has implicit assumptions. For example, Domain Driven Design, high-throughput writes, complex workflow management, a distributed system with key-value stores, actor systems, functional programming, ... Although these assumptions might make sense for their respective environment, none of them was a generic solution that is applicable across a variety of domains, from simple CRUD applications to complex workflow management tools.

After examining the designs from various angles, I discovered that two assumptions commonly associated with Event Sourcing are not actually necessary: Event Sourcing as a state recovery mechanism, and the lack of transactions.

Event Sourcing can be designed not as a design to recover a state from events but as a design to store immutable events. From this perspective, events are not merely differences between state versions, but rather, essential information to be preserved. On the other hand, states are just intermediate data to generate new events. Although the difference may seem trivial, it can eliminate unnecessary assumptions, which unlocks valuable functionalities.

In addition, I noticed that many problems associated with Event Sourcing look similar to those of  key-value stores. While the use of relational schemas is not required, it does not mean that we cannot use ACID transactions. Transactions provide an easy way to manage concurrent processing of commands and access to multiple states.

Of course, the use of transactions limits the selection of a database and may lead to some performance bottlenecks. This approach may not be favored by other Event Sourcing users. However, given that many organizations use RDBMS, use of transactions would not be considered as a unbearable constraint.

I described the design process in this series of short articles. I also implemented a simple library in Scala 3, Postgres, and ZIO. It can be easily transported to other tech stacks. The library is small enough to be embedded in a standard server. It does not need to be used for everything; in case there are some problems hard to be solved by the design, we can choose another solution for them while using Event Sourcing for others.

I view that this design solves many problems discussed above. The design is reasonable and simple enough. It would significantly reduce complexity, which exponentially increases developers' productivity. Developers can focus on essential design decisions and reflect the design to declarative code. It can be easily understood without prior knowledge. No need to learn many conventions. Even a newbie can start writing code in a few days.
