Background/Technical documentation
==================================

# 1 Project goals and vision

## 1.1 Main Project goal

The goal of the Project is to provide a tool for monitoring of distributed applications in actor model in Akka framework. We begin with the short introduction to the basics, e.g. the actor model, message­passing semantics and practical problems with debugging distributed systems. Based on this introduction we define our problem precisely and describe in more details the tool that we will be creating. 

## 1.2 The actor model

The actor model in computer science is a mathematical model of concurrent computation that treats ‘actors’ as the universal primitives of these computations. In response to a message that it receives an actor can:

* make a local decision, 
* create more actors, 
* send messages to other actors, 
* determine how to respond to the next message received.

In Figure 1 you can see a simple diagram of actor system and how actors communicate with each other.

![Simple diagram of actor system model (source: [1])](https://raw.githubusercontent.com/akka-tracing-tool/akka-tracing-docs/master/images/proc/fig1.png "Simple diagram of actor system model (source: [1])")

**Figure 1.** Simple diagram of actor system model (source: [1])

The actor model originated in 1973 and was proposed i.a. by Carl Hewitt [2]. Its development was motivated by creating highly parallel computing machines consisting of huge amount of independent processors that use them all. 
 
The actor model has been used both as a framework for a theoretical understanding of computations and also for practical implementation of concurrent systems.

## 1.3 Message-passing semantics

The actor model is is very strictly connected with message­passing semantics. Message-passing programming paradigm assumes many instances of process work in sequential mode and they can communicate by sending messages to each other. In pure message-­passing these computing units don’t share any data between them. However, programs written using this semantics can be run on architecture that supports a different computational model. For more information about message­-passing semantics, please see S.S. Kadam’s presentation [3].

One of the most important distinctions among message passing systems is whether they use synchronous or asynchronous message passing: synchronous message passing is what typical object­oriented programming languages such as Java and Smalltalk use. Asynchronous message passing requires additional capabilities for storing and retransmitting data for systems that may not run concurrently.

In Figure 2 you can see messages passing between actors A and B, every of them is working in sequential mode and don’t share anything with external actors:

![Messages passing between actors A and B (source: [4])](https://raw.githubusercontent.com/akka-tracing-tool/akka-tracing-docs/master/images/proc/fig2.png "Messages passing between actors A and B (source: [4])")

**Figure 2.** Messages passing between actors A and B (source: [4])

Message­-passing semantics is easy to reason about and simpler than another existing computational models used to create complex, distributed systems.

## 1.4 Debugging and tracing

Developing applications is a very complex process and consists among other things of testing. It is very important, because of that you can increase the quality of codebase and ease future maintenance. Various types of metrics were established to measure tests coverage. But even 100% of coverage can’t make you sure that your application doesn’t contain any bugs and flaws. That’s why we need debugging ­ sometimes you don’t cover every case in the system and you have to check what causes a bug and find it.

Developing a complex system requires usage of different debugging techniques: from manually instrumenting code by logging information on *standard output* up to usage of sophisticated tools that allow you to stop, check variable states and even evaluate expressions.

In a sequential environment using and reasoning about results of such a tool are obvious and intuitive. Problems mount up when we meet multiple threads: it’s not possible to predict interlacing of different processes. There are debuggers for multi­threaded applications but complexity of using them is much greater. Ok, so what will happen when we move to the actor model? Actors are independent: they process messages sequentially in separation of any others. Everything is asynchronous and you can’t easily connect sent message with it actual processing in receiver. This is also connected with a dispatcher every message can be dispatched in different threads so it’s hard to follow that. It’s getting even more complicated when you run your application on multiple JVMs.

In Figure 3 you can see how actors’ code are dispatched and prepared for execution on CPU processors:

![Diagram of dispatching and executing actors’ code on threads (source: [5])](https://raw.githubusercontent.com/akka-tracing-tool/akka-tracing-docs/master/images/proc/fig3.png "Diagram of dispatching and executing actors’ code on threads (source: [5])")

**Figure 3.** Diagram of dispatching and executing actors’ code on threads (source: [5])

Debugging in one actor is relatively easy (because it’s sequential), but what happens when we want to move to another actors that will be receiving that message? Well­known in standard debugger button “Next line/Step into” is not really useful in such systems. One proposal for a convenient tool was suggested by Iulian Dragos: an async debugger [6]. He pointed out that debugging that kind of applications is like being a detective: you need to go from effects back to causes, attempt a fix, and check whether everything works fine. Threads and locks are replaced by higher­level abstractions like futures, actors and parallel collections which use a thread­pool for executing their computations. He also suggested that instead of ‘Step into’ we should add ability to ‘step-­with-­message’ ­ this can happen in a different thread or even these two actors can exchanged different messages in the meantime! It’s not important for us: we want to move to the next logical step which is  
receiving the message.

As you can see below, in Figure 4, there is an implementation of that behaviour. Currently, it is available as a feature in Scala IDE.

![A screenshot from Scala IDE - in debug frame you can see green ‘bang’ icon that allows using *step-with-message* ability (source: [7])](https://raw.githubusercontent.com/akka-tracing-tool/akka-tracing-docs/master/images/proc/fig4.png "A screenshot from Scala IDE - in debug frame you can see green ‘bang’ icon that allows using *step-with-message* ability (source: [7])")

**Figure 4.** A screenshot from Scala IDE - in debug frame you can see green ‘bang’ icon that allows using *step-with-message* ability (source: [7])


