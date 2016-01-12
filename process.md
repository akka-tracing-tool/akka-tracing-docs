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

Debugging in one actor is relatively easy (because it’s sequential), but what happens when we want to move to another actors that will be receiving that message? Well-known in standard debugger button “Next line/Step into” is not really useful in such systems. One proposal for a convenient tool was suggested by Iulian Dragos: an async debugger [6]. He pointed out that debugging that kind of applications is like being a detective: you need to go from effects back to causes, attempt a fix, and check whether everything works fine. Threads and locks are replaced by higher­level abstractions like futures, actors and parallel collections which use a thread­pool for executing their computations. He also suggested that instead of ‘Step into’ we should add ability to ‘step-­with-­message’ ­ this can happen in a different thread or even these two actors can exchanged different messages in the meantime! It’s not important for us: we want to move to the next logical step which is  
receiving the message.

As you can see below, in Figure 4, there is an implementation of that behaviour. Currently, it is available as a feature in Scala IDE.

![A screenshot from Scala IDE - in debug frame you can see green ‘bang’ icon that allows using *step-with-message* ability (source: [7])](https://raw.githubusercontent.com/akka-tracing-tool/akka-tracing-docs/master/images/proc/fig4.png "A screenshot from Scala IDE - in debug frame you can see green ‘bang’ icon that allows using *step-with-message* ability (source: [7])")

**Figure 4.** A screenshot from Scala IDE - in debug frame you can see green ‘bang’ icon that allows using *step-with-message* ability (source: [7])

## 1.5 The need for tracing

Very often during debugging it is helpful to have information about the whole message path. Tracing allows developers to inspect what causes unwanted behaviour. Sometimes with tracing developers can gather statistics and test performance of application. Full tracing is very expensive process so it’s reasonable only for development time. However, tracing is possible in production environment combined with sampling.

Information gathered through tracing developers can use in many ways: 

* espy unwanted message and check its sender, 
* check message correctness, 
* filter by actors, message types or contents, 
* gather different aggregate statistics. 

Main idea for the product that is the subject of this Project was a tool that enables developers to ease debugging of application with the usage of traces and message flow. Below you can see an example of situation where such tool could give us sensible information about what happened. 

## 1.6 Illustrative example

In Figure 5, you can see that problem depicted in a simple diagram: An actor indicates the existence of the problem by e.g. throwing an exception. The cause of this error was in Actor 4 processing a faulty message originated by Actor 2 which is 2 interactions before. The stack trace of Actor 4 shows only information about current actor’s context. However, we would like to know what really caused this problem. 

![Messages flow that causes an unwanted behaviour](https://raw.githubusercontent.com/akka-tracing-tool/akka-tracing-docs/master/images/proc/fig5.png "Messages flow that causes an unwanted behaviour")

**Figure 5.** Messages flow that causes an unwanted behaviour

## 1.7 Product goals

We would like to have a tool that:

* enables the user to collect and see the messages passed between actors in a user’s actor system, 
* provides a way easily enable/disable tracing, 
* requires minimum workload for programmers to use it, 
* does not require drastic changes in the actor system that user wants to trace, 
* should not have large impact on the performance of the traced system, 
* provides a way to visualize collected traces either through some external tool or built during the development process. 

## 1.8 Discussion of problems and their solutions

In the scope of the work we encountered several problems that we needed to solve: 

* **integration with existing collectors and visualization tools** - we knew about very good, production tested systems for distributed tracing, but it could turned out that integration would be very hard or even impossible due to differences between data formats or gathered informations. We also didn’t want to run against these systems - ­we wanted to create something a little bit different than existing implementation. We appreciated that we had to create our own version of collector.
* **client requirements** - they could be too strong, which means that gathered tracing data may be not helpful at all. We had to figure it out and check frequently whether this product introduces real value into developing.
* **more research topic than some practical project**, so it demanded from us more prototyping than coding in the initial phase. We had to very carefully create schedule and stick to it, because it was so easy to not meet client requirements.
* **simplicity of usage** - it is very important for developers: they don’t want to spend a lot of time configuring and doing manual hand­work to getting things done. We’ve struggled to gain best results in this field. Compared to Akka­tracing we try to collect message flows in generic way without specific hints and instrumentation in codebase, so-called *zero application specific hints instrumentation*. Although we know that in specific case it’s very useful to manually define additional data to log but we haven’t considered it much - it can be done in future works.

# 2 Feasibility study

The feasibility study was the first phase of our Project. The process of creating a tool that is the subject of the Project was associated very strongly with potential problems, which could cause delays and even lead to situations where it was impossible to end the work with existing requirements. We knew that this phase of our Project is very important and we’ve tried to predict most of the problems and minimize the possibility of failure and prepare best foundation for future work. 


