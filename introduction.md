---
description: Is the
---

# Introduction

## Motivation

Most developers focus on adding features. Many people follow conventions of each language and framework. Though it makes development easier, it does not necessarily make a system simpler.

I rewrote many production applications. I often heard managers and developers saying "why would you dare to change applications that are working?", but I tend to find tens of bugs in each application. No error and no test failure don't mean that applications are working as expected. Most developers do not fully understand what the application they wrote is doing. Typically, it's because state of the application is too complex to be fully managed.

To rewrite an application, state transitions and pairs of input and outputs are required. As long as these contracts don't change, nothing gets broken. If we cannot list up these contracts, basically nobody can precisely explain what the application is doing even if the code looks clean. As you can easily imagine, the more complex state transitions are, the harder it is to list up inputs and outputs. In general, what matters is not how neatly code is written but how well state transitions are managed.

One major cause of the complexity is database management. RDBMS is often chosen as a data storage. Most developers use it as a shared state protected by transactions. Typically, data access is abstracted out by a framework for easiness of development. Developers can just call a method in an interface and complex transactions are managed by the framework. Developers can focus on the application's domain by separating out complex database manipulation. Though that is important to produce business value, from the technical viewpoint database management is much more important.

## Is there a better Event Sourcing design?

We store all the state transitions in a system as events. Once we have events, any state in the past can be recovered. Commands are inputs from the external world. Outcomes of handling a command under each state can be easily understood and verified. Developers can still focus on writing domain code without worrying too much about database management.

If we can design a simple architecture like this, the complexity mentioned above will be significantly simplified and anyone can easily understand what the system is doing.

Event Sourcing is often introduced as a way to recover application's state. That is totally valid, but I found out that the potential of Event Sourcing is not fully leveraged because these solutions put too much assumptions such as "aggregates" and "eventual consistency" at the early phase of the design process.

In this series of short articles, I'll defer these design decisions on the application layer and try to understand the requirements and constraints the Event Sourcing design has. Based on what are discovered, I'll design a working application that supports management of both state transitions and domain logic.

