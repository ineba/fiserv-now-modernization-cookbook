+++
categories = ["recipes"]
tags = ["start"]
summary = "Let's get started creating a new microservice"
title = "Let's get started"
date = 2020-12-09
weight = 10
draft = false
authors = ["Prashanth 'PB' Belathur"]
pre ="<i class='fa fa-spinner fa-pulse fa-1x fa-fw'></i>&nbsp;&nbsp;"
+++

Cookbook with recipes for using the greenfield application starters to develop modern cloud native microservice in Wells Fargo.  
The cookbook and starters were created by VMware Tanzu Labs during the WellsFargo Enterprise Architecture Consulting engagement in January 2021.


## CONTEXT

The following spring boot starters in Github will help Wells Fargo developers to build _non-reactive_ or _reactive_ microservices based on the business need:
- **greenfield-app-starter** can be used for migrating a _legacy application_ to a cloud-ready microservice OR to create a non-reactive cloud-ready microservice with traditional _blocking behavior_.
  

- **greenfield-reactive-app-starter** for _greenfield applications_, to take advantage of the _non-blocking behavior_ which improves application performance and resiliency.

### What is Reactive Programming ?

Reactive Programming is a new paradigm in which you use declarative code (in a manner that is similar to functional programming)
in order to build asynchronous processing pipelines. It is an event-based model where data is pushed to the consumer, as it becomes available: we deal with asynchronous sequences of events

Reactive code is assembled around asynchronous functional chains, where inputs are streamed (propagated) through these chains from the producer to the subscribers.

#### Why Reactive

Before we plunge into technical aspects of Reactive Systems and architectures, we should know "Why build Reactive Systems"?
Let's say that our system should:

* Responsive to interactions with its users.
* Handle failure and remain available during outages.
* Strive under varying load conditions.
* Be able to send, receive, and route messages in varying network conditions.

  These answers actually convey the core reactive traits as defined in the [reactive-manifesto](https://www.reactivemanifesto.org/)
  ![](/images/reactivemanifesto.PNG)

So how should an application be responsive? One of the first ways to achieve responsiveness is through **elasticity**. This
describes the ability to stay responsive under a varying workload, meaning the throughput of the system should increase
automatically when more users start using it and it should decrease automatically when the demand goes down. From the
application perspective, this feature enables system responsiveness because any point in time the system can be expanded
without affecting average **latency**. The acceptance criteria for the system are the ability to stay responsive under failures, or, in other words, to be resilient.
This may be achieved by applying isolation between functional components of the system, thereby isolating all internal failures and enabling independence

Another point to emphasize is that elasticity and resilience are tightly coupled, and we achieve a truly responsive system only by enabling both. With scalability, we can have multiple replicas of the component so that, if one fails, we can detect this, minimize its impact on the rest of the system, and switch to another replica.
* [Elasticity](https://www.reactivemanifesto.org/glossary#Elasticity)
* [Failure](https://www.reactivemanifesto.org/glossary#Failure)
* [Isolation](https://www.reactivemanifesto.org/glossary#Isolation)
* [Component](https://www.reactivemanifesto.org/glossary#Component)

## Application Stack

Both the application starters share a common **Application Tech Stack**, comprising of components approved for use within Wells Fargo.

![Application Tech Stack](/images/tech-stack.png)

## SOLUTION
### [Build Non-Reactive microservice](#non-reactive-path)

Complete the recipes in following order:
- **Create barebone microservice using the starter**
- **Configure Actuators**
- **Configure Oracle**
- **Configure MongoDB**
- **Configure Kafka**
- **Configure Resilience**

### [Build Reactive microservice](#reactive-path)

Complete the recipes in following order:
- **Create barebone microservice using the starter**
- **Configure Actuators**
- **Configure MongoDB**
- **Configure Kafka**
- **Configure Resilience**

## NOTES
- [What is Reactive Programming ?](https://blog.redelastic.com/what-is-reactive-programming-bc9fa7f4a7fc)
- [Essence of Reactive Programming](https://www.scnsoft.com/blog/java-reactive-programming)