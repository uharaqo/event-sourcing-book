# Design Requirements

## Overview

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

The system receives a command and looks up the command handler for that. It may read events to generate zero or more summaries. The command handler generates zero or more events, or rejects the command. The command handler may call external systems. The emitted events are persisted in a database. Projections can read these events to make side effects such as updating databases or sending events to external systems.

### Event

An event is an immutable record persisted in a database. Events contain information selected to be persisted. Each event has a summary group, summary ID, monotonously increasing version number, and event content.

A new event must be generated based on the latest summary associated with the event.

Example: group: `cart`, id: `cartId-123` , version: `5`, content: `{"_name": "ItemAdded", "itemId": 41, "count": 2}`

### Summary

Events are grouped by a summary. For instance, `RegisterUser`, `UpdateEmail`, and `ChangeAddress` events can be grouped by a summary group: `user`, summary ID: `alice`.

A summary is an interpretation of events. It is generated by aggregating events by the `foldLeft` function with an event handler. Summaries are only used for command handlers.

### Event Handler

Generates the next summary by consuming the next event and the previous summary.

`summary_n = foldLeft( summary_0 )( (previousSummary, event) => eventHandler(previousSummary, event) )`

The initial summary is predefined by developers. The initial summary and an event v1 generate a summary v1, and the summary v1 and an event v2 generate summary v2, and so on.&#x20;

Event handlers should only depend on given state and event. This constraint ensures that summaries are safely reproduced at any time. Since events are immutable and never lost, the summary from the events never change unless there's a change to the event handler.

### Summary Group

A summary group is literary a group of summaries. It's used for a projection to discover new events without knowing their IDs.

### Projection

A projection is an event consumer. Projections can be used to run side effects based on persisted events (e.g. updating a table, sending an email). Since each event is immutable and has a unique version, they can be safely consumed concurrently as many times as required. Typically, a projection runs in a separate process for scalability and is designed under eventual consistency and at-least-once semantics.

### Command

An input from outside the system. It's a request to change the system's internal state and/or to make side effects. The concept is mostly the same as a typical HTTP POST / PUT request body.

### Command Handler

Business rules are implemented as a command handler. It receives a command and generate events or rejects the command. It may consume summaries and call external systems.