# What is Event Sourcing?

## This is not a book about THE Event Sourcing

There is no single definition or design for Event Sourcing.&#x20;

Many people say Event Sourcing needs to be Eventually Consistent. Many say a state is an DDD Aggregate. Some say Event Sourcing is too complex to be applied to typical systems. Some insist that the Event Sourcing is their event driven architecture. I got a lot of insights from Greg Young's articles, presentations, and lessons. I also joined Event Sourcing course. But I felt that all of these are designs created for their use cases and Event Sourcing has more capability that can be applied to even a small CRUD application.

In this series of short articles, I'm going to design an Event Sourcing system from scratch. It would look different from the designs known as "Event Sourcing" because I removed some assumptions that existing designs have.

Let's start from a simple design and see what can be deduced from that.

## Core Concept: foldLeft

The idea behind Event Sourcing is quite simple. The `foldLeft` function is the key technical concept.

`state_n = fold( state_0 )( (previousState, event) => eventHandler(previousState, event) )`

* An event represents a state transition
* An event handler generates the next state based on the previous state and an event

As you can easily see, once we have events, we can recover the state at any point in time.

### Basic Event Requirements

A typical application stores its states in a DB and uses DB functionalities to manage state transitions. Event Sourcing stores events (state transitions) insetad of states themselves. Of course, we shouldn't lose states just because of this design change. States need to be safely recovered from events at any time.

* Events can be identified in the inserted order for recovering a specific state
* A state at any point in time can be reliably recovered

To satisfy these,

* Events are immutable
* Each event has an ID of the associated state. Let's call this a `State ID`.
* Events with the same state ID are sorted in the insertion order. Each event has a version for that
* An event should be generated based on the latest state generated from all the previous events
* Event handlers should only depend on the given state and event

Anyone who uses Event Sourcing would agree on these requirements.

Each event can be safely replicated and cached. So are the states generated from these events. This is because of the immutability of the events and side-effect free event handlers. The only thing to be considered is the staleness of each state (i.e. event consumption delay).

### Event Insertion

In general, concurrent updates of a state needs to be prevented. Only one process that has the latest state can insert a new event with a newer version.

* Detect a duplicate on insertion. This is supported by most databases.
* Use a pessimistic lock in the database such as a transaction
* Use a coordination service that selects a process that can write events for each state ID

Apparently, the first solution is the easIest, and the last one is the hardest. Though usage of transactions limit the choice of a database, it provides additional functionalities to the system. For now, let's assume that we use the first solution and consider transactions later.&#x20;

### Event Group

Let's assume that we only have a huge single state for the entire system. We'll quickly face performance issues due to insertion conflicts. Also, all the events need to be read and the huge state needs to be reconstructed on memory. We need to split the state.

* Events are grouped by an `Event Group` which represents a transaction boundary
* Events in each event group with the same state ID are sorted in the insertion order

Event groups detect conflicts on insertion and identify events on query. As the silly example shows, having more than one event group is not for functionality but for performance optimization.

### Command Handler, State, and Event

DDD practitioners would call an event group as aggregate. An aggregate is a concrete entity that has multiple entities inside it. If we think a state to be an aggregate, we would use the Event Sourcing system as a way to backup and recover the aggregate. The system updates the aggregate, and events are differences between each version of the aggregate. The aggregate would contain all relevant information in the previous events.

We have not seen any reason to put the assumption and constraints. A state does not need to keep information in the past events. Therefore, events might not be the differences between versions of a state.

A state is only used by command handlers. A command handler receives a command to mainly generate events. States only need to hold information required to execute command handlers.

If a state is not necessarily an aggregate, what actually is an event? That's a piece of information we want to persist. We keep recording events so as not to lose information we received from external systems. We can recover any state that was generated from the persisted events, and process a command again like a time machine except that side effects to external systems cannot be repeated. This unique functionality is what we cannot easily get from a traditional state-based systems.

Also note that we didn't put other constraints for command handlers. A command handler might call external systems, consume multiple states, and generate multiple events for the states. It might use a transaction for that.

Even at this point, the design we discussed is quite different from typical Event Sourcing designs. Eventual consistency and aggregates are not necessary requirements. I summarized what we have discussed so far in the next page.
