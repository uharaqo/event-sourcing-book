# Projections

## Motivation

Projections are event consumers that make side effects based on stored events. Event version or timestamp can be used to store the position of the last event read. Also, some problems might be easily resolved by sending a command from a projection. Other than that, projection is just a typical event consumer. Many solutions in this area can be integrated.

### Push-based (pub/sub) or pull-based (cron job)?

Let's consider a push-based approach. After an event gets stored, it's published to an external messaging queue or pub/sub system. But there are some problems. If this is done inside the transaction, when the transaction gets rolled back, the published event cannot be removed. If the event is published without a transaction, delivery is not guaranteed. Basically, event publish from the command side is at-most-once. Also, ordering of events might not be guaranteed.

How about pull-based approach? A projection process reads some events periodically and record the ID and version in a durable storage once it's done. Even if the process goes down, a new process can easily resume from the previous version. Since the process is separated from command handlers, it can spend as much time as allowed. For instance, transactions and distributed coordination can be used if required. There are many options to tune up concurrency and consistency. They can be chosen for each use case.

Though pull-based approach is easier to maintain, the interval of event queries can be a problem. If it's too short, it increases the load on the database side. If it's too long, it takes a longer time for changes to be applied.

I typically create a projection based on pull-based design. And then pushes a "change notification" from the command side so that the projection can quickly pick up the change. Though the delivery of the notifications is not guaranteed, the next periodical fetch would find out the changes.&#x20;

### Eventual Consistency

The results of projections are eventually consistent. This is the point most people who heard about Event Sourcing becomes skeptical. In a regular CRUD system, an update is immediately gets reflected. What if a user gets a stale data even though he has just updated that successfully?

What if the delay is tens of milliseconds? It would get reflected before the response of the command arrives at the user's device. And if showing the latest information is required, the process can wait for the event to be applied. It can be easily achieved by checking the projection's position of event consumption.

What if it also times out? It is exactly the same behavior as other regular CRUD systems. A user makes a POST request and the request times out. Did it succeed or not? The user needs to check the latest status or makes the request again.
