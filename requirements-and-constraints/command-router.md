# Command Router

The command processors are just functions. State is just simple immutable data. We can handle these as a part of an application process, or as an independent server application.

Since the commands are also just simple data, we can use HTTP, gRPC, message queues, or whatever protocol that suits well with the environment.

What needs to be considered is performance of state recovery and conflicts on insertion. If a command is always routed to the same process, the impact of both problems can be significantly reduced.

A state can be safely cached. If a command can be routed to the process that already has a state cache, it significantly reduce the number of events to be read. Also, a snapshot on the database side has the same effect.

If commands for the same event group are queued for the same process, it will significantly reduce the chance of event insertion conflicts.

There are many solutions for these. Message queue, sticky routing, transactions, distributed coordination, and so on.

It would be more productive not to expect 100% accuracy for this because this is just a performance optimization. Even under an unstable network condition, the system should keep running consistently.
