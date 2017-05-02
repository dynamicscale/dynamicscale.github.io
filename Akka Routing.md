An Akka router is used to route messages efficiently to destination actors. We can use different routing strategies which come in-built with Akka and create our own routing.

Routing can be done in Akka in two ways:

* Create a router which manages routees itself
* Use a self-contained router actor with configurable routing capabilities

# How to use a simple router?

A simple router manages routees itself and has to do the following:

* Create a set of routees
* Define routing strategy
* Route messages to routees
* Optionally watch the routees and replace when any of the routees die

```
import akka.routing.{ ActorRefRoutee,RoundRobinRoutingLogic, Router }

class Master extends Actor {
  var router = {
    val routees = Vector.fill(5) {
      val r = context.actorOf(Props[Worker])
      context watch r
      ActorRefRoutee(r)
    }
    Router(RoundRobinRoutingLogic(), routees)
  }

  def receive = {
    case w: Work =>
      router.route(w, sender())
    case Terminated(a) =>
      router = router.removeRoutee(a)
      val r = context.actorOf(Props[Worker])
      context watch r
      router = router.addRoutee(r)
    }
}
```

In this example, `Master` actor creates 5 routees (`Worker` actors) and wraps them in `ActorRefRoutee` and then creates a router `akka.routing.Router` with `RoundRobinRoutingLogic`. It also watches the routees so that it can take appropriates action when any of the them fails. 

Then it just sends the incoming messages of type `Work` to routees as per the routing logic (round robin here) using `route` method.

In case any of the routees is terminated, it will just create a new one and add it to routee set.

# Which routing strategies are available in Akka
Akka provides the following routing strategies:

###### RoundRobinRoutingLogic
Messages are routed in a round robin fashion.

###### RandomRoutingLogic
Each time a messages arrives, the router picks a routee at random and forwards the meesages.

###### SmallestMailBoxRoutingLogic
Messages are sent to actors with fewest messages in their mailbox.

###### BroadcastRoutingLogic
Messages are sent to all the routees.

###### ScatterGatherFirstCompletedRoutingLogic
Messages are sent to all the routees and the response from the first one to reply is used. Responses from all other routees are discarded.

###### TailChoppingRoutingLogic
It first sends a messages to one of the routees (randomly picked). Then after a small delay, it will send the message to another (again randomly picked) and so on. Response from the first one to reply is used. Responses from all other routees are discarded.

###### ConsistentHashingRoutingLogic
Messages are sent in consistent hashing fashion.



