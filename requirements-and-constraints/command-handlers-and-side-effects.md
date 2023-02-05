# Command Handlers and Side Effects

## Simple case: no side effects

We have simple mechanisms that can store and consume the events. Now let's design the biggest component, command handlers. A command handler is responsible for generating events or rejecting commands. It may consume events to recover states or call external systems.

Let's suppose that a command handler does not depend on data from external systems.

These are success cases:

* Emit no event (no state change)
* Emit one event
* Emit more than one events for a single event group
* Emit more than one events for multiple event groups
* Reject the command

Failure cases:

* Network error (events read / write)
* Unexpected error (a bug)

All of these can be easily understood. On success, events are written if any; otherwise, nothing changes.

## Getting external states

There are many use cases that depend on external states. A command handler sometimes needs to read data by remote calls.

An external system is a state: value that change over time. A response from external systems is a command: input to the system.

Suppose that we update a command handler 1 year later. If we don't preserve the original responses from external systems, we need to infer the state from stored events since the state is already lost. This is exactly the problem we are trying to resolve by replacing an state-based database. Basically, those external states in the past should be stored as events.

1. Simple Approach: Command = (Original Command, Response)

The easiest way to manage this is just to treat a response as a command. A pair of command and response becomes the command for the command handler.

2. Chain command handlers

What if we have a function like this?

`fetch = if state != null then state else fetchAndStoreExternalState(request)`

This can be an independent command handler. It accepts an external call request as a command, emits the response as an event, and reconstructs the state based on the events.

Apparently this approach is more complex, but we can store the intermediate response as an event. We can easily model a multi-step workflow by chaining command handlers.

## Mutating external states

In the previous section, we saw the benefit of keeping an external state as an event.

When updating external states, we need to check if it's idempotent or not. Non-idempotent update depends on its previous state and idempotent update does not. Idempotent update is repeatable. This is not a topic specific to Event Sourcing.

Non-idempotent updates cannot be repeated in general. But even if it is a non-idempotent update, we might be able to make it effectively idempotent. For example, even when `POST /users` fails due to a duplicated request, we might be able to get the same result by calling `GET /users?email=...` . In that case, seen from outside, the external state and the response are exactly same as what are expected.

Once a mutation succeeds, we can keep the result just like what we saw in [#getting-external-states](command-handlers-and-side-effects.md#getting-external-states "mention").

&#x20;

### Errors

* `Retryable` or `Unrecoverable` errors

