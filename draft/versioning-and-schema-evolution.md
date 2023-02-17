---
description: >-
  Requirements change over time. How can we update an Event Sourced system
  without breaking it?
---

# Versioning and Schema Evolution

[Versioning in an Event Sourced System Gregory Young](https://leanpub.com/esversioning/read)

## How do we design events?

Events contain all the information that need to be preserved. Information that are not in the events cannot be recovered later. If we preserve all inputs to the system (commnads, responses from external systems), we can replay the behaviors of command handlers at any point in time without causing side effects to external systems.

We can't change facts in the past. But we can change the interpretation of the facts and change our behaviors in future. Likewise, we can update the definitions of summaries and command handlers as long as the changes don't conflict with past events. In case there needs to be a breaking change, we can define a new events, summaries, or command handlers.
