# Command Handlers and Side Effects

## Simple case: no side effects

We designed simple mechanisms to store and consume events. Let's explore what we can do with the event store design.

A command handler is responsible for generating events or rejecting commands. It may consume events to recover summaries or call external systems.

Let's start from a simple scenario: a command handler that does not have external calls. Possible scenarios are:

* Emit no event (no state change)
* Reject the command (invalid request)
* Emit a single event
* Write conflict (events write)
* Network error (events read)
* Network error (events write)
* Unexpected error (a bug)

These can be easily supported by the event store design. If there was another process that already wrote an event for the summary, the write request fails due to the conflict.

* Emit more than one events for a single summary
* Emit more than one events for multiple summaries

These can be supported by an atomic write. If any of the events conflicts with existing records, none of them gets written and the command handler fails.

## Getting external states

A command handler may need to retrieve data from external systems. A response from an external system can be considered like a command, an input to the system. It can be helpful to capture the response at the time as an event.

Suppose that we need to modify an existing command handler that calls an external service. If we don't preserve the original responses from the external system, we need to infer the responses from stored events. This is exactly the problem we are trying to resolve by replacing a state-based database. If we keep the response as a part of the event, we can modify the handler by using the past events.

### A. Simple Approach: Command = (Original Command, Response)

The easiest way to manage this is just to treat a response as a command. A pair of command and response becomes the command for the command handler.

### B. Chain command handlers

What if we have a command handler like this?

```
val fetcher = (s, c) => if s != null then s else fetchExternalState(request)
```

It accepts an external call request as a command, emits the response as an event, and memoise the event as a summary. Apparently this approach is more complex, but we can store the intermediate response as an event. We can easily model a multi-step workflow by chaining command handlers. See [#command-chain](command-handlers-and-side-effects.md#command-chain "mention")

## Mutating external states

When updating external states, it's important to understand whether the update is idempotent or not. Non-idempotent updates depend on the previous state of the system, while idempotent updates do not. In general, non-idempotent updates cannot be repeated, while idempotent updates can be repeated without causing any unintended side effects. There are a few things to be considered, especially for non-idempotent updates.

### Concurrent updates

In general, commands to the same summary can be processed concurrently. The easiest way to prevent an unexpected consequence is to block other command handlers for the same summary by using a transaction. In addition, the possibility of concurrent command executions for the same summary can be reduced by [command-router.md](command-router.md "mention").

### Timeout errors on non-idempotent updates

Although a transaction can prevent concurrent updates, a change made to the external system cannot get reverted on errors. For example, if an update request times out, we don't know if the request was successfully processed or not. Although an idempotent update can be just retried, a repeated non-idempotent update call might fail.

```
val updater = (s, c) => if s != null then s else updateExternalState(request)
```

If we can have a command handler like this just like [#b.-chain-command-handlers](command-handlers-and-side-effects.md#b.-chain-command-handlers "mention"), we can treat it as an "effectively" idempotent update since we can't distinguish the difference from the outside.

In some cases, we might be able to make a call "effectively" idempotent simply by calling another API. For example, when `POST /users` fails due to a duplicate, we might be able to get the same result by calling `GET /users?email=...`&#x20;

Once a mutation succeeds, we can keep the result just like  [#getting-external-states](command-handlers-and-side-effects.md#getting-external-states "mention").

## Command Chain

<figure><img src=".gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

By using a transaction, we can compose a multi-phase command handler from simpler command handlers.
