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


