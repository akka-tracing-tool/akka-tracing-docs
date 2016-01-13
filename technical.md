Technical documentation
=======================

# 1 Project domain
## 1.1 The actor model in Akka
### 1.1.1 How does an actor look like?

Actors in Akka toolkit are classes which extend *Actor* trait and implement the *receive* method. The *receive* method should define a series of case statements, that defines which messages actor can handle, using pattern matching available in Scala language. Every case should be provided with the implementation of behaviour when the message is received. Below is an example of simple actor:

```scala
import akka.actor.Actor

class ExampleActor extends Actor {
  def receive = {
    case "test" => println("received test")
    case _      => println("received unknown message")
  }
}
```

The *receive* method has the type *PartialFunction[Any, Unit]*, which means that you don’t have to provide a pattern match for all messages. However, if you want to handle unknown messages then you need to have a default case as in the example above. The return type of the behaviour is *Unit*: if the actor shall reply to the received message then this must be done explicitly by sending a new message.

For more information about actors in Akka, please see the official Akka library documentation [1].

### 1.1.2 More complex example

It’s a good practice to provide factory methods on the companion object of each actor which helps keeping the creation of Props very close to the actor definition (Props is a configuration object used in creating an Actor). Another advantage is that this method can perform type check on compilation, which is very useful from the developer point of view:

```scala
object DemoActor {
  def props(magicNumber: Int): Props = Props(new DemoActor(magicNumber))
}

class DemoActor(magicNumber: Int) extends Actor {
  def receive = {
     case x: Int => sender() ! (x + magicNumber)
  }
}

class SomeOtherActor extends Actor {
  // Props(new DemoActor(42)) would not be safe
  val actor = context.actorOf(DemoActor.props(42), "demo")
  // ...
}
```

Another good practice is to declare messages an actor can receive in companion object. In this approach, the developer can see which messages he needs to handle. In other words, the companion object defines the contract for actors or their protocol just like an interface defines the contract for the classes that implements it in object-oriented programming.

```scala
object ExampleActor {
  case class Greeting(from: String)
  case object Goodbye
}

class ExampleActor extends Actor with ActorLogging {
  import ExampleActor._

  def receive = {
    case Greeting(greeter) => log.info(s"I was greeted by $greeter.")
    case Goodbye           => log.info("Someone said goodbye to me.")
  }
}
```

### 1.1.3 Messages

In Akka messages can be any kind of object - there’s only one constraint: they have to be immutable. Scala can’t enforce immutability so you need to take care of this by yourself - this is to avoid the shared mutable state trap, as described in the official Akka documentation [2]. Primitives like String or Boolean are always immutable. The recommended approach is to use Scala case classes which work very well with pattern matching and greatly improve readability of code.

Here is an example:

```scala
//the case class definition
case class Message(from: String)

//creating a new case class message
val message = Message("Bob")
```

### 1.1.4 Sending messages

Sending messages in Akka toolkit is possible through one of the following methods:

! - ‘fire-and-forget’, send a message asynchronously and return immediately. Also known as tell.
? - sends a message asynchronously and returns a *Future*. Also known as ask.

#### Tell - ‘fire-and-forget’

Sending messages using this method is very simple and looks exactly like in Erlang language:

```scala
actorRef ! message
```

It’s the preferred way of passing messages. Also if invoked from within an actor, the sending actor reference will be implicitly passed along with the message and receiving actor can get it from *sender()* method - available in *ActorRef* class. It’s very handful - developers don’t need to explicitly pass sending actor reference in messages which increase readability of code.

#### Ask - ‘send-and-receive future’

The *ask* pattern allows actor to ask another of information and get the return value through the Future object. To send a message this way, you need to provide a `Timeout` object, which specifies how long actor will be waiting before throwing `AskTimeoutException`. Below is an example of simple usage of that pattern:

```scala
case object AskMessage

class TestActor extends Actor {
  def receive = {
    case AskMessage =>
              sender ! "Returned value"
    case _ => println("Unexpected message")
  }
}

implicit val timeout = Timeout(5 seconds)
val future = myActor ? AskMessage
val result = Await.result(future, timeout.duration).asInstanceOf[String]
println(result)
```

For a complete example using the ask sending messages pattern, please see Alvin Alexander’s blog post [3].

### 1.1.5 Akka Remote

Everything in Akka is designed to work in distributed manner: all interactions between actors use message passing and are asynchronous. As a result all functions are equally available when running your application on single JVM or cluster consisting of more machines. Success of this implementation is based on design: developers started from remote solution as a base environment for running applications and then go to local using optimization. Different approach - generalization from local to remote - is described in documentation as “bound to fail”. For more arguments and detailed discussion see paper “A Note on Distributed Computing” [4].

For more information about Akka Remote, please see official Akka documentation [5].

#### Usage in project

The Akka remote package is available as a separate jar file. Developer needs to provide following dependency:

```scala
"com.typesafe.akka" %% "akka-remote" % "2.3.9"
```

Also minimal configuration is necessary. It allows us to enable remote capabilities along with setting basic values, like hostname and port.

```scala
akka {
  actor {
    provider = "akka.remote.RemoteActorRefProvider"
  }
  remote {
    enabled-transports = ["akka.remote.netty.tcp"]
    netty.tcp {
      hostname = "127.0.0.1"
      port = 2552
    }
  }
}
```

This package in simple words allows communication between actors in different actor systems. In our solution it is necessary because our collector exists outside of the application actor system and we want to gather data using message passing semantics.

For more information regarding Akka please see the official Akka documentation [6].

## 1.2 Aspect-oriented programming

Aspect-oriented programming is a paradigm that aims to increase modularity of code by separating different concerns. It’s done by providing additional behavior into existing code using *an advice* and specify exactly which *pointcuts* will be affected. This mechanism allows to add behaviors that “cut across” different abstractions in the application, choose what parts of system we want to modify and encapsulate them in “unit of code/entity”. Very good example for cross-cutting concern is logging functionality: it isn’t central to the business logic but every part of the application need to log some information, e.g. ‘log all functions that set fields in object - method name starts with *“set”*’. Using aspect-oriented programming you can easily specify which methods you want to instrument - a pointcut - and how - an advice. An example of instrumenting the code can be seen in Figure 1.

![Figure 1](https://raw.githubusercontent.com/akka-tracing-tool/akka-tracing-docs/master/images/tech/fig1.png "Figure 1")

**Figure 1.** The main idea of aspect-oriented programming

For more information about aspect-oriented programming please see the Wikipedia article [7].

### 1.2.1 AspectJ

AspectJ is an aspect-oriented programming extension created for Java language. It’s available both stand-alone version and integrated into Eclipse IDE. AspectJ has become widely known because of simplicity and ease of use for the developers.
For more information about AspectJ, please see the Wikipedia article [8].

#### AspectJ syntax

AspectJ allows to write programs using Java language syntax or specifically prepared constructions called aspects. Below are presented basic structures:

* **pointcuts** - helpers that allow programmers to specify which join-points want to instrument. (A join-point is well defined moments in the execution of program, e.g. function call or variable access.)

```
pointcut setPointcut(): execution (void set*(..));
```

Every method which name starts with *‘set’* and return type is void can be weaved using this *setPointcut()*.

* **advice** - encapsulate code to run and specify a joint-point. Advice can be weaved before, after or around pointcuts.

```
before (): setPointcut(): {
Logger.debug("before 'set' method");
}
```

#### Annotation based syntax

Another method to create aspect is to use annotations. It’s very similar to earlier mention AspectJ flavoured syntax and it’s done by placing pointcut specifiers expressions in annotations, which are available in Java API from version 1.5.

It’s very useful when you don’t want to use AspectJ syntax because of requirements and introducing unnecessary complexity into your project.

The pointcut:

```
pointcut setPointcut(): execution (void set*(..));
```

can be written as:

```java
@Pointcut("execution(void *.set*(..))")
void setPointcut() {}
```

And the advice:

```
before (): setPointcut(): {
    Logger.debug("before 'set' method");
}
```

can be written as:

```java
@Before("setPointcut()")
public void beforeSetPointcut() {
    Logger.debug("before 'set' method");
}
```

#### Implementation

Instrumentation using aspect-oriented programming can be implemented in multiple ways, e.g. source-weaving, bytecode-weaving or runtime-weaving. It depends on the user requirements and accessibility to source code. Source-weaving is good way to provide best performance but it’s necessary to have access to code you want to instrument. Bytecode-weaving allows to instrument compiled class. Runtime-weaving is very similar to the previous one - it enables to instrument class during loading into JVM using service API provided in *java.lang.instrument* package. In our product we’re using this type of weaving and we have to provide additional library which is responsible for weaving our code into existing actor’s code.

For more information about AspectJ weaving types, please see Denis Zhdanov’s blog post [9].
