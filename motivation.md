# Motivation

## State is the biggest cause of complexity

We create a system by integrating sub-systems. We start from small components and compose a bigger component from them. We need to make sure that each component works as we expect. Even one line of wrong code could break the entire system.

We can only understand a small and simple component. We can also understand a component composed of multiple simple components. But we cannot fully understand a complex component. A composite of multiple complex components gets exponentially complex.

Side effects are major cause of complexity. State management is especially important. If a component reads or mutates a state, we need to understand all the states and the behavior for each state. If we overlook some cases, we have unknown behaviors in the component. A component that contains unknown components also have unknown behaviors. This is how a system gets complicated. [We need to keep each component simple](https://youtu.be/LKtk3HCgTa8).

In other words, separating out state management from other parts of the code makes the code much easier to be understood. If a component does not depend on any internal or external state, the only thing it does is a conversion from each input to the output. A component that uses those stateless components can be also stateless.

If the productivity is truly important, we should have a design for state management. Though there is a learning curve, the benefit of managing simple components would quickly overcome the cost since we can eliminate the complexity that increases over time.

<figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

## Database is a huge state

An application changes its behaviors based on its internal state and external states integrated by remote calls. Typically, applications are designed to be stateless since the processes are volatile. They fetch states from external storage or service.

Relational schemas are designed based on the relations and relationships of data, which may not always match the structure of objects which is represented as a directed graph. This impedance mismatch can lead to complexity and unnecessary constraints. Many developers use the Repository pattern to abstract out data access layer and use the Aggregate pattern to represent a transaction boundary as an object. These abstraction layers make the code complex and put unnecessary constraints. Other developers use the Transaction Script pattern but as the number of scripts increases it becomes harder to maintain them. Instead of maintaining mapping between objects and relations, we can choose a key-value store to simply store an object. It works well for a simple application, but features supported by relational schemas, SQL, and transactions need to be implemented on the application side. It requires design skills.

Another impedance mismatch is in the service's APIs. Unless the application is a simple CRUD that simply provides operations to each table, relationship between an APIs, objects, and tables are not always straight forward. For instance, maintaining test data set is typically hard. One way to do that is to call update APIs, but all the requirements in the APIs need to be satisfied. Another way is to directly insert records by scripts, but the constraints in the database need to be satisfied. Either way, when there is a change to the application or database, the setup code and scripts need to be updated by satisfying constraints in each layer. It's an error-prone and time-consuming task and most developers only check limited amount of test cases.

We should consider if the use of relational schemas and the conventions are worth paying the cost of maintenance overhead. In most cases, these vertical layers (APIs, objects, and data persistence) are tightly coupled. For instance, when you add a column to a table, you would need to change all the layers. A design that spans across these three layers would reduce the cognitive overhead and maintenance cost.

## Is Event Sourcing an overengineering?

Event Sourcing seemed a great solution. The design has common characteristics of good designs: clear separation of concerns and strong cohesion / loose coupling.

* Queries are separated and they can be flexibly designed on demand
* Simple immutable data models: commands, events, and states on the write side
* Commands represent the intent of callers
* Command handlers generate events based on a command and state
* Immutable events that are easy to replicate

With this design, we can focus on essential workflows. For example, `RegisterUser` -> `UserRegistered`, `UpdateEmail` -> `EmailUpdated` , `UpgradePlan` -> `PlanUpgraded`, ...  It is simple enough to be understood and developers can focus on writing declarative code. Code becomes declarative and straight forward. It solves many problems mentioned above.

However, when I presented this design to my colleagues, I struggled to justify the use of Event Sourcing. Some proponents of Event Sourcing claim that it is overly complex and should only be used for complicated problems. This is due to the additional considerations that come with the design's constraints, such as eventual consistency, the Saga pattern, Process Manager, compensations, ... Too many factors need to be taken into account.

## Proposal: Consistent Event Sourcing

Though I researched numerous articles, books, and videos, as well as taking courses, I could not find an easy solution to the problems. I discovered that each design approach contains several implicit assumptions. For example, DDD aggregates, high-throughput writes, complex workflow management, a distributed system with key-value stores, actor systems, or functional programming. They might be necessary assumptions for their respective environments. However, I was seeking a generic solution that is applicable across a variety of domains, from simple CRUD applications to complex workflow management tools.

I decided to design Event Sourcing from scratch. After examining the design from various angles, I discovered that two assumptions commonly associated with Event Sourcing are not actually necessary: event as a state transition, and the lack of transactions.

Event Sourcing can be designed not as a design to recover a state from events but as a design to store immutable events. From this perspective, events are the essential and states are just summaries of the events. Events are not merely differences between state versions, but rather, essential information to be preserved. Although the difference may seem trivial, it can unlock valuable functionalities by eliminating the unnecessary assumption about states.

In addition, I noticed that the problems associated with Event Sourcing look similar to those of  key-value stores. While the use of relational schemas is not required, it does not mean that we cannot use transactions. Transactions provide an easy way to manage concurrent updates and access to multiple states.

Of course, the use of transactions limits the selection of a database and may lead to some performance bottlenecks. This approach may not be favored by the proponents of Event Sourcing. However, given that many organizations use RDBMS, use of transactions would not be considered as a unbearable constraint.

I described the design process in this series of short articles. I also implemented a simple library in Scala 3, Postgres, and ZIO. It can be easily transported to other tech stacks.

The library is not a framework. In case there is a feature that is hard to be implemented with Event Sourcing, we can just add it next to the Event Sourcing module or implement the feature as a separate service.

I view that this design is what I was looking for. The design is reasonable and clearly separate concerns. It can be easily understood without prior knowledge. The coding convention is intuitive. No need to learn many conventions. Even a newbie can start writing code in a few days. Developers can focus on design decisions and reflect the design to declarative code. The simple design would significantly reduce complexity, which would exponentially increase developers' productivity.
