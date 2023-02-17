# Projections

## Motivation

Projections are event consumers that run side effects based on stored events. There is no constraint on projections. Variety of processes and systems can be integrated. To track the progress of a projection, event version or timestamp can be used.

The only thing to be considered is how to run the projection on each event write.

### Push-based (pub/sub) or pull-based (scheduled job)?

Let's consider a push-based approach. After an event gets stored, the event gets published to an external messaging queue or pub/sub system. But there are some problems. If this is done inside the transaction, even when the transaction gets rolled back, the published event cannot be removed. If the event is published without a transaction, delivery is not guaranteed. Basically, event publish from the command side is at-most-once. Also, ordering of events are not be guaranteed.

How about pull-based approach? A projection process periodically scan through events for new events. Once an event is processed, the ID and version of the event can be recorded in a durable storage. Even if the process goes down, a new process can easily resume from the previous version.

Though pull-based approach is easier to maintain, it's not easy to choose the interval of event queries. If it's too short, it increases the load on the database side. If it's too long, it takes longer for changes to be applied.

I typically use both approaches:

* Create a projection that checks events based on a given ID and version. On success, update the version.
* Pull: Create a scheduler that periodically invokes the projection based on the stored ID and version.
* Push: Send a change notification on successful event write from a command processor. This is "at-most-once" delivery. When a push notification is received, invoke the projection with the stored ID and version. Information in the notification should be only used to lookup the version since ordering is not guaranteed.

In most cases, a projection gets invoked by a push notification. Even if these notifications are received in a wrong order, the projection works as expected since it doesn't depend on the information in the notification. Even when the notification gets lost, the scheduler can detect new events. The interval of the scheduler becomes the maximum time for the data to be stale.

### Eventual Consistency

There is some delay for events to be applied by projections, which means that the data updated by projections is eventually consistent with the original events. Most people who heard about Event Sourcing get skeptical by the eventual consistency.

But by using the push + pull mechanism, the lag between event lag and invocation of projection would be tens of milliseconds. In most cases, this lag would not be noticeable.

If strong consistency is required, we can ask the caller to send the latest version they got from the command handler and block the response.

Though there are some more failure scenarios, most problems also happen in typical RESTful services and not so many people are even aware of the problems. For instance, what if a user updates something on a browser while another user is seeing that? The latter user would just gets an error on update attempt and just refreshes the page. If that's not allowed, we would implement real-time data sync such as websocket but that is another topic and we can also support that with Event Sourcing model.
