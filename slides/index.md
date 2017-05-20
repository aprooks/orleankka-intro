- title : Orleankka F# intro
- description : Intro to statefull distributed systems via Orleankka/Orleans
- author : Alexander Prooks
- theme : night
- transition : default

***

## Orleankka F#

<br />
<br />

### Statefull distributed architecture

<br />
<br />
Alexander Prooks - [@aprooks](http://www.twitter.com/aprooks)

***

## Stateless request handling

    //Infrastructure: Cache, RequestValidation
    let Put (request:SetNewPassword) =
        
        let user  = Users.Load request.Id
        user.Password = request.NewPassword;
        Users.Save user

---

## Real life

    let Put req = 

        let a = A.Load req.SomeId
        let b = B.Load req.AnotherId
        let c = C.Load req.Id

        if a.Data > b.Data && c.List.Contains 42 then
            failwith "Q?"
        else
            D.save 42

---

### Pros

<br/>

* Easy
* Scalable application servers

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

=> Just don't! :trollface:

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

## Orleans vs others

* Virtual actors = grains
* Infrastructure manages lifecycle
* In-built cluster 
* Automatic actors distribution
* At-least-once delivery by default
* Caller awaits remote execution
<br />
<br />
<br />
### Simplicity => Profit!

---

## Orleans

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

## Orleankka aka functional Orleans

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

            override this.Receive msg = task{
                match msg with
                | Ping -> return "Pong"
            }
---

### Client

    let system = [|Assembly.GetExecutingAssembly()|]
                 |> ActorSystem.createPlayground
                 |> ActorSystem.start   

    task {
        let pinger =  ActorSystem.actorOf<Pinger>(system,"myId")
        let! res = pinger <? Pong
        printfn "%s" res //Pong
    } 
    |> Task.run 
    |> ignore

***

# DEMO

***

---

## Grain core components

* OnActivate => loads state
* OnReceive  => Handle
* Reminders   => Persistent scheduling
* Timers     => Non-persisten scheduling
* Streams    => Pub/Sub
* Reentrancy => Concurrent execution (Queries!)
* Workers    => Stateless parallelism 

***

### Thank you!

* https://github.com/fable-compiler/fable-elmish
* https://ionide.io
* https://facebook.github.io/react-native/
