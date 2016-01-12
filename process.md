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

# 1.3 Message-passing semantics

The actor model is is very strictly connected with message­passing semantics. Message-passing programming paradigm assumes many instances of process work in sequential mode and they can communicate by sending messages to each other. In pure message-­passing these computing units don’t share any data between them. However, programs written using this semantics can be run on architecture that supports a different computational model. For more information about message­-passing semantics, please see S.S. Kadam’s presentation [3].

One of the most important distinctions among message passing systems is whether they use synchronous or asynchronous message passing: synchronous message passing is what typical object­oriented programming languages such as Java and Smalltalk use. Asynchronous message passing requires additional capabilities for storing and retransmitting data for systems that may not run concurrently.

In Figure 2 you can see messages passing between actors A and B, every of them is working in sequential mode and don’t share anything with external actors:

![Messages passing between actors A and B (source: [4])](https://raw.githubusercontent.com/akka-tracing-tool/akka-tracing-docs/master/images/proc/fig2.png "Messages passing between actors A and B (source: [4])")

**Figure 2.** Messages passing between actors A and B (source: [4])

Message­-passing semantics is easy to reason about and simpler than another existing computational models used to create complex, distributed systems.


