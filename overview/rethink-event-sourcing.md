# Rethink Event Sourcing

## Core Concept: foldLeft

The idea behind Event Sourcing is quite simple. The `foldLeft` function is the key technical concept.

`summary_n = foldLeft( summary_0 )( (previousSummary, event) => eventHandler(previousSummary, event) )`

The event handler generates the next version of summary based on the previous summary and an event.

By using Event Sourcing, we can recover the application's state at any point in time.

## Why not use Command Sourcing?

Our system reacts to each command. If we store the commands, we can replay whatever happened in the past. This is called Command Sourcing. Can we implement this easily? As you can easily imagine, side effects in command handlers are the major problem. What if a command handler calls external systems to get and update external states? We can't replay these external calls.

This is why Event Sourcing is chosen. An event can contain information of inputs from the outside world (command and responses from external systems), and results of computation. It's easy to preserve these events since disks are cheap. We can keep track of all the inputs that caused state transitions.

### Basic Event Requirements

A typical application uses database functionalities to manage state transitions. The database keeps the application's latest state. On the other hand, Event Sourcing stores events (state transitions). Since events contain all the information required for the application, a summary of events contain required information for each part of the application process.

We should ensure that a summary can be safely generated from given events.

* A summary of events at any point in time can be reliably recovered

To achieve this,

* Events are immutable
* Events to generate a specific summary can be identified in the inserted order
* The latest summary generated from all the past events should be used to generate the next event
* Event handlers should only depend on given summary and events so that we can recover the summary any time from anywhere

Events can be safely replicated and cached. So are the summaries generated from the events. This is because of the immutability of the events and side-effect free event handlers.

### Event Insertion

In general, concurrent updates for a single summary must be prevented since it makes the summary generation unpredictable. There are several solutions:

* Use a conditional insertion (CAS). This is supported by most databases.
* Use a pessimistic lock such as a transaction.
* Route a command to the command handler in a specific process

Apparently, the first solution is the easiest, and the last one is the hardest. Though the use of transactions limit the choices of a database, it provides additional functionalities to the system. Let's assume that we use the first solution and consider pessimistic locks later.&#x20;

### Grouping of Events

Let's assume that we only have a single huge summary for the entire system. We wouldn't lose any functionalities because of this, but we can easily imagine problems.

Query efficiency: all the events need to be read and the huge summary needs to be constructed on memory. We want smaller summaries.

* A summary has a `summary ID` (e.g. 'user123')
* Each event has a summary ID
* Events with the same summary ID are sorted in the inserted order

Write performance: write conflicts will become a performance bottleneck. We want to control how to prevent conflicts.

* Summaries are grouped by a `Summary Group` which represents a transaction boundary
* Events in with the same summary group and summary ID are sorted in the insertion order

Summary IDs are for reads and summary groups are for writes. By using a summary group, we can create a transaction boundary for multiple summaries.

For instance, by assigning 'user' summary group to 'user' and 'email' summaries, we can ensure that 'user' and 'email' are updated consistently and we can query each summary independently.

Other Event Sourcing systems use a concept of a DDD aggregate which has an ID and represents a transactional boundary. With this design, you cannot control how to query events and how to control write conflicts separately.

### "State" is not so important

Other Event Sourcing systems put much more emphasis on a state (summary). Most of those systems use Event Sourcing to backup and recover application's state. These are the assumptions those systems have in common:

* Each command handler has its own state
* An event is a state transition (i.e. information required to update the current state to the next state)
* A command handler can only generate events for the state

But we have not seen any reason to put these assumptions and constraints. A state (summary) is just information on memory required for command handlers. They are literary summaries of events to skip reading the past events. I would put more emphasis on events and less emphasis on state.

* A command handler can consume multiple summaries
* An event is information we want to preserve
* A summary is only used by command handlers. It only needs to have information required for a command handler
* A command handler can generate events for multiple summaries

This is the reason why I called a state as a summary. Application's state is preserved as events in a database. A summary is literary a summary of events.

We receive a command that has information from the external world. The command handler for the command lookup some summaries and generate events. Events are persisted for the next command and for projections.

With this generalization of Event Sourcing concept, we can design events, summaries, and command handlers by using variety of powerful features.

Even at this point, the design we discussed is quite different from typical Event Sourcing designs. I summarized what we have discussed so far in the next page.
