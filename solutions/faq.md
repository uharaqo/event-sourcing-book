# FAQ

Most people who started learning Evnet Sourcing say that the design is reasonable. Most of them are already using some of similar design patterns. Whenever I explain Event Sourcing, I get many similar questions. I was one of them. Though some of them are valid doubts, most of them are not inherent in Event Sourcing and they are not aware that even the traditional design, REST with RDBMS, has the same problems.

### States can be too big. Can they fit into memory?

A state only needs to keep information required to generate events or reject a command. For instance, in a `User` state, it might be useful to keep a `dateOfBirth` to check user's age for permissions, but a `name` would not need to be kept since it won't be used for decision making.

Also, if a state is too complex or big, that functionality can be externalized as a DB record or a service. No need to use Event Sourcing for everything.

### Read model might be stale

What if the response to a query is stale?

### Learning Curve

### Concurrent Requests

How this event group is modeled depends on use cases. Transaction boundaries based on write patterns and query patterns for read patterns would be the key consideration points. For example, we can have `User` and `Email` states separately and share a single `User` event group. By doing this, we can recover each state separately.

### Writing command handlers without a transaction is too complex for a task

An answer from Event Sourcing community would be&#x20;

### Benefits of using Event Sourcing

#### Append Only Logs

Once events get persisted, they are never updated or lost. This is important not only for auditing but also for debugging and testing applications. Events are stored into a single simple schema. It's easy to replicate a database in a local machine and reproduce what happened at any point in time.

#### &#x20;CQRS: Command Query Responsibility Segregation

The command handler process is logically separated from query processes. Typically, SLAs and requirements for commands and queries are quite different. This separation of concern enables us to flexibly choose appropriate designs for each system.

#### Event Modeling

The design is quite simple and even non-tech people can easily understand the concept. This design fits well with the concepts introduced by Domain Driven Design, which encourages collaboration between domain experts and developers. This can have significant impact on design process and model based development.

#### State Management

State management is often one of the most time-consuming process in development. Event Sourcing offers a reasonable design, which could simplify state management.

### Downside of using Event Sourcing

#### Eventual Consistency

A traditional approach (REST and RDBMS) uses ACID transactions, which means that results to queries are always consistent at any time. Typically, an Event Sourcing system is designed like an application using a key-value store. Most performance characteristics come from this choice.

By choosing this design, you need to find alternative design that can be easily solved by relational databases with ACID transactions and foreign key and unique constraints.

For instance, let's think about a `RegisterUser` command that can generate a `UserRegistered` event for a `User` state. What happens when the caller sends duplicated commands? The command handler would generate an ID, which means that both commands might successfully store `UserRegistered` events. What if we need to put a unique constraint to the email address in the event? A typical answer would be registering an email and handle the command only when it succeeds. But what if the process goes down immediately after registering an email? The orphan email record prevents user registration without a manual operation.
