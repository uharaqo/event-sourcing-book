# Command Handlers and Side Effects

## a

Now we have simple mechanisms that can store events, recover states, and consume the events.&#x20;

The last component of the Event Sourcing system is command handlers. A command handler is responsible for generating events or rejecting commands. It may consume events to recover states or call external systems.







If these side effects in command handlers are abstracted out as a model, we can design a system to manage the complexity.

## Constructs

### Operation Types

* `CREATE` : Fails if an event is already stored for the ID. HTTP POST.
* `UPDATE` : State transition depends on the previous state. HTTP POST.
* `OVERWRITE` : Produces the same state transition even when executed more than once. HTTP PUT. Idempotent operation.
* `APPEND` : No dependency to states. It might be useful to have a feature to load events more efficiently.

### Side Effects

* `Nullpotent` : query that has no effect to external systems.
* `Idempotent` : command that produces the same effect even when executed more than once.
* `NonIdempotent` : command that cannot be retried
* `EffectivelyIdempotent` :  idempotent command that can be repeated (e.g. get result if command fails)

### Errors

* `Retryable` or `Unrecoverable` errors

## Problems

*
