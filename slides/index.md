- title : Orleankka F# intro
- description : Intro to statefull distributed systems via Orleankka/Orleans
- author : Alexander Prooks
- theme : night
- transition : default

***

## Orleankka + F#

<br />
<br />

### Statefull distributed architecture via MSFT Orleans

<br />
<br />
Alexander Prooks - [@aprooks](http://www.twitter.com/aprooks)

***

## Stateless

<img src="images/stateless.png" style="background: transparent; border-style: none; height: 500px"  />

---

### Pros

<br/>

* Easy?
* Scalable? application servers

<br/>
<br/>

### Cons
<br/>

* State = database
* Scale via Queues, BUSes etc.
* Cache invalidation
* External scheduling:
    * lock user if was not active for 24 hours?

---

### Objects in memory?

* Concurrency (locks, mutexes etc.)
* Object location
* System Resilience
* Memory management

=> Just don't! 

***

# Actor model

* Function or object
* Isolated state
* Syncronous execution
* Messages (requests) are queued

---

<img src="images/actors.svg" style="background: transparent; border-style: none;"  />

---

## Implementations

* Erlang ( Since 1986! )
* Akka: Jvm and .Net
* Orleans: .Net and Orbit Jvm
* ProtoActors: Go, .Net (cross platform)

---

### Akka/Erlang
``` C#
var game = activate(“game-1”, “tcp://10.0.0.1/”)
game.invoke(“foo()”) 
```

### Orleans

``` C#
var game = getGrain(“game-1”)
game.invoke(“foo()”) 
```
---

<img src="images/runtime.png" style="background: transparent; border-style: none;"  />


---
## Orleans vs others

* Virtual actors = grains
* Infrastructure manages lifecycle
* In-built cluster 
* Automatic actors distribution
* At-least-once delivery by default
* Caller awaits remote execution
* Garbase collection

<br />
<br />
<br />

### Simplicity => Profit!

***

## Programming model

    [lang=c#]
    public interface IPinger{
        Task<Pong> Ping();
    }

    public class Pinger: Grain, IPinger
    {
        public Task<Pong> Ping(){
            return Task.FromResult("Pong");
        }
    }

---

## Orleankka function interface

    [lang=c#]
    [Serializable]
    public class Ping : ActorMessage<Pinger>
    {        
    }

    public class Pinger: Actor{
        public Task<string> Handle(Ping request){
            return Task.FromResult("Pong");
        }
    }

---

## Calling another actor

    [lang=c#]
    var pinger = System.ActorOf<Pinger>("someId");
    await pinger.Tell(new Ping());

***

### Orleankka F#

    module Pingers =
        type Message = 
        | Ping
        
        type Pinger() = 
            inherit Actor<Message>()

            override this.Receive msg = task {
                match msg with
                | Ping -> return response("Pong")
            }
---

### Client

    let job() = task {
        let pinger =  ActorSystem.actorOf<Pinger>(actorSystem,"myId")
        let! res = pinger <? Ping
        printfn "%s" res //Pong
    } 

    Task.run (job)
    |> ignore

***

## Grain core components

* OnReceive  => Handle all
* Activate   => load  state
* Reminder   => Persistent scheduling
* Timers     => Non-persisten scheduling
* Streams    => Pub/Sub
* Reentrancy => Concurrent execution (Queries!)
* Workers    => Stateless parallelism 

***

# DEMO


***

# Marketing

* Halo with 14 ml users <br>
    * "25 servers with 90% utilization without any instability"
    * linear scaleability

    https://www.infoq.com/presentations/halo-4-orleans

* EA implemented Orbit in JVM

***

### Thank you!