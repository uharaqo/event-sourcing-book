# Motivation

## Typical development organization

Imagine that a new grad developer, Alex, just started his career. He did not have much knowledge on development.  How long does it take for him to master server development? Maybe 5 - 10 years. For instance, he might need to learn Java, Object Oriented Programming, Spring, Hibernate, Postgres, AWS, Kubernetes, REST APIs, JUnit, ... Even after reading all the documents and books for these, he still cannot write a reliable application. He needs to learn system design.

Alex was told to "follow the conventions". He would learn how to write some procedures, extract out some procedures to avoid copy & paste. He would learn how to efficiently do these tasks by copying conventions chosen by senior engineers. After 5 years, he learned the development conventions in his organization.

He decided to be a manager. He started mentoring younger engineers. He told them "follow the conventions". The engineers copied some code written by him and learn how to write an application.

Now he got promoted to a manager. He started hiring more engineers. He listed up the technologies used in the organization as qualifications such as 3+ years of experience: Java, Spring, Postgres, ...

He encouraged his subordinates by saying "We are not rocket scientists. We are hired to deliver the product. It's much more important to get feedback from customers than to build a perfect solution. Let's be productive and keep improving the code little by little. Done is better than perfect."

This is what typically happens in many teams. Their system would keep getting complicated, which causes more and more bugs, and at some point they would need to rewrite their applications. Nobody is doing anything wrong, but they spent a lot of time on fixing problems that could have been used for other valuable tasks.

## State is the biggest cause of complexity

We create a system by composing sub-systems. We start from small reliable components such as simple data and functions, and compose a bigger component from the small components. Even one line of code could break the entire system. We need to make sure that each component works as we expect. Even one line of wrong code could break the entire system.

We can only understand a small and simple component. We can also understand a component composed of multiple simple components. But we cannot fully understand a complex component. A composite of multiple complex components gets exponentially complex. I believe that [a simple design makes things easier](https://youtu.be/LKtk3HCgTa8).&#x20;

A major cause of complexity is side effects. State management is especially important. If a component reads or mutates a state, we need to understand all the states and the behavior for each state. If we overlook some cases, we have unknown behaviors in the component. A component that uses unknown components also have unknown behaviors. This is how a system gets complicated.

The biggest state we typically have is a database. Typically, we use a RDBMS to keep consistency of data. The state is managed both by the application and database. Unless it's properly managed, it can easily become a huge state that any component can read and mutate. It becomes harder to understand how a record gets updated and which components get affected if a data schema or SQL gets modified.

Testing is also difficult. To write integration tests for an application with RDBMS, you need to setup patterns of requests and states. This is one of the hardest part of development because if you prepare lots of data, when something is changed you also need to update those data without breaking consistency. This is not a trivial task and most developers give up testing most cases.

In other words, separating out state management from other parts of the code makes the code much easier to be understood. If a component does not depend on any internal or external state, the only thing it does is a conversion from each input to the output. A component that uses those stateless components can be also stateless.

If the productivity is truly important, we should have a design for state management. There is a learning curve, but the benefit of managing complexity would quickly overcome the cost since the complexity increases exponentially over time.

![](<../.gitbook/assets/image (3).png>)

## Where is the solution for that?

There are some design patterns for state management. For instance, the repository pattern and aggregate that represents a transaction boundary is widely used. But that's a convention and not a design applicable to variety of problems. We still see discussions after more than tens of years like "Should this be in a repository or in a service? Should this be an aggregate or entity?"

You might know that Functional Programming works best for this problem since it clearly separates out side effects. You might be able to learn it and manage the complexity. But would you introduce that to the Alex's team? None of them know Functional Programming and they have a lot of code written in Java. Alex would say "Done is better than perfect. It's working. Why would you dare to take a risk? How do we educate and hire engineers?"

The solution should be easy enough for junior engineers to understand and use as a convention and generic enough to be applied to real world problems.

## Is Event Sourcing overkill?

When I discovered Event Sourcing, I was developing an ETL job. While writing the data pipeline, I was thinking that most web applications can be written much better as an ETL job. But most applications depend on database features and they don't always fit well with a design like Map Reduce. For instance, how do we support transactions, unique constraint, foreign keys, and JOINs?

Event Sourcing looked like a great solution. It does not store latest states but stores state transitions. We can focus on essential workflows. Commands and events can be represented as JSON. For example, `RegisterUser` -> `UserRegistered`, `UploadPicture` -> `PictureUploaded` , `RegisterAddress` -> `AddressRegistered`, `RegisterCard` -> `CardRegistered`, ...  It is simple enough to be understood and even junior engineers can focus on writing declarative code. It solves many problems such as state management, testing, debugging, flexible read models, and so on.

I started introducing the design to my colleagues. Though they showed interest, none of them looked impressed. I also noticed that I'm also struggling to justify the use of Event Sourcing.

Even those who use Event Sourcing say that Event Sourcing should be only used for complex problems because it's not a solution for everything. You need to accept eventual consistency, need a solution without a transaction such as the Saga Pattern or Process Manager, compensation to revert an event, sometimes manual intervention is required, and so on.

## Proposal: Consistent Event Sourcing

I researched many articles, books, videos, and took some courses but none of them provided easy solution for the problems. I found that most people put many assumptions to the problems that can be solved by Event Sourcing. For instance, DDD aggregates, high-throughput writes, complex workflow management, a distributed system with key-value stores, actor system, functional programming, ... All of them would be valid designs for their environment, but I was looking for a solution that fits variety of problems from simple CRUD applications to complex workflow management tools.

I decided to design Event Sourcing design from scratch. I checked various problems from different angles and found out that two assumptions are not actually required: event as a state transition and lack of transaction.

We can design Event Sourcing not as a design to recover a state from events but as a design to store immutable events. The difference might sound trivial, but we can discover new design patterns from that. By using a transaction, we can easily implement problems that are not so easy to be solved by other designs. For instance, Constraints such as unique keys and foreign keys can be easily supported.

Since the use of transaction limits the choices of a database, it might not be a design other Event Sourcing users want. But the number of people who use RDBMS considered, I don't think it is a strong constraint.

I described how I reached to the conclusion in this series of short articles. I also implemented a simple framework in Scala 3, Postgres, and ZIO. It can be easily transported to other tech stacks.

I view that this design is what I was looking for. The design is simple and easily understood even by junior developers. The coding convention is intuitive enough and developers can focus on essential problems. Developers don't need to learn lots of conventions. They just need to write data models and business rules in a declarative way.&#x20;
