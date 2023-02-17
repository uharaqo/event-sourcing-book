# Proposed Development Process

## 1. Define Command

A command is input from the external world. It is almost the same as typical HTTP POST / PUT requests. Example:

```
{
  "_type": "RegisterUser",
  "name": "Alice",
  "email": "alice@..."
}
```

## 2. Define Event

An event contains information to be preserved. It's worth including information in the command and responses from external services so as not to lose these information.

```
{
  "_type": "UserRegistered",
  "name": "Alice",
  "email": "alice@...",
  "serial": 194832048431
}
```

## 3. Implement Command Handler

Now it's time to write the command handler.&#x20;

```
val registerUser =
  commandHandlerFor[RegisterUser, UserRegistered, User] { (c, s, ctx) =>
    s match {
      case User.EMPTY =>
        val serial = UserService.newSerial(c.email)
        ctx.save( UserRegistered.from(c, serial) )
      case _ =>
        ctx.failure( InvalidState.create("Already Registered") )
  }
```

## 4. Implement Summary and Event Handler

To implement the command handler, the `User` summary needs to be defined. At this point, we only need to know if the user is registered or not. We can add more fields to this summary if required. For instance, we might want to keep `email` for other commands but we wouldn't need `name` since it's rare for a command handler to change its behaviors based on a user's name.

```
class User()

object User:
  val EMPTY = new User()
```

```
val eventHandler =
  eventHandlerFor[UserRegistered, User] { (e, s) =>
    s match
      case User.Empty => new User()
      case _ => s
  }
```

## 5. Testing

Testing is easy and straight forward; send a command and validate the output events. Additionally, the summary can also be validated.

```
tester
  .command( RegisterUser("Alice", "alice@...") )
  .events( UserRegistered("Alice", "alice@...", 194832048431) )
  .summary( new User() )
```

## 6. Implement Projections

Projections are event consumers that are executed as separate processes. There is no constraints on projections.

```
val userProjection =
  projectionFor[UserEvent] { e =>
    e match {
      case RegisterUser => db.update(e)
    }
  }
```

## Atomic Writes

Let's assume that we need to ensure that an email address is unique. We can emit an `EmailRegistered` event with the email as a key along with the `RegisterUser` event. In the command handler, we can lookup the `RegisteredEmail` summary that is generated from 0 or 1 `EmailRegistered` event.

## Transactions



## Event Storming

