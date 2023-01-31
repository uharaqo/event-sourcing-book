---
description: >-
  It's hard to infer information from a stored state. It's easy to understand
  what happened in the past if we keep state transitions. And reconstruction of
  a state is easy.
---

# What is Event Sourcing?

## What is Event Sourcing?

There is no single definition or design for Event Sourcing. I saw many articles that puts too much unnecessary constraints as the characteristics of Event Sourcing. Let's start from a simple design and see what can be deduced from that.

### Concept

The idea behind Event Sourcing is quite simple. The `foldLeft` function is the key technical concept.

`state_n = fold( state_0 )( (previousState, event) => eventHandler(previousState, event) )`

* An event represents a state transition
* An event handler generates the next state based on the previous state and an event

As you can easily understand, once we have events, we can recover the state at any point in time.

### Requirements

To reliably recover a state,

* Events should be immutable
* Each event has an ID to query a sequence of events
* Each event has a unique version that is used to sort events with the same ID in the order of inserted time
* The next event should be generated based on the latest state generated from all the previous events
* Event handlers should only depend on the given state and event.

Anyone who uses Event Sourcing would agree on these requirements.

### What can be deduced from this design

* Each event can be safely cached. So are the states generated from these events. This is because of the immutability of the events and side-effect free event handlers. The only thing to be considered is delay of event propagation which implies the staleness of the state.
* Event write conflicts need to be prevented. This is because a concurrent update of a state produces uncertainty in general. For instance, by choosing an incremental number as a version, an insertion gets rejected if an event with the version is already registered without pessimistic locks. Multiple events can be written if the database supports an atomic batch insertion. If transactions can be used, more complex use cases can be supported. We'll discuss these later.
* What if we have a single huge state for the entire application? It should satisfy all the requirements stated above, but we'll face performance bottlenecks soon because only one event insertion can be executed at a time. We would want to group events to split this transaction boundary for concurrency. Let's call the boundary as an "Event Group". It's guaranteed that only one process can insert the next event for each state and events inside the group are uniquely versioned. Also to recover a state, all the events need to be read even when only one event needs to be looked up. An event group is also important for the query performance, though the impact can be mitigated by caching the generated state.
* Most Domain Driven Design practitioners use an aggregate as this state and event group. An aggregate is an object that has entities inside that. It has a unique identifier and represents a transactional boundary. Though from the DDD modeling viewpoint this totally makes sense, from the technical viewpoint this model is not necessary. For instance, not all the information in the past events needs to be recovered as an aggregate. As an another example, we can have a `User` and `Email` states separately and share a single `User` event group. By doing this, we can recover each state separately.
* How this event group is modeled depends on use cases. Transaction boundaries based on write patterns and query patterns for read patterns would be the key consideration points.
* A command handler generates events based on a command received from outside the system and generate events based on states. I'll discuss the details later because there are many topics to be covered. Please note that I have not put any constraints at this point. For instance, a command handler might consume multiple states, generate multiple events, make side effects, or use a transaction.
* Note that a state does not need to contain all the information included in the past events. State needs to hold only information required for the command handler.  Command handlers have many topics to discuss. See [command-handlers-and-side-effects.md](requirements-and-constraints/command-handlers-and-side-effects.md "mention")
* Other processes can safely read events and make side effects such as updating a database or relaying events to an external queues / brokers. This process is called an projection and typically data that gets updated by projections is called read models. We'll discuss that in [projections.md](requirements-and-constraints/projections.md "mention")

Even at this point, this design is different from typical Event Sourcing designs. Eventual consistency and aggregates are not necessary requirements. I summarized what we have discussed so far in the next page.

