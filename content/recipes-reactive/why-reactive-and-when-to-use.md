+++
categories = ["recipes"]
tags = ["reactive","spring", "reactor","spring webflux"]
summary = "Why Reactive"
title = "2. Why Reactive"
date = 2021-01-06T14:02:27-05:00
weight = 1
+++

## CONTEXT
This recipe provides the context of Why Reactive Programming and Use cases

## What is Reactive Programming

Reactive Programming is a new paradigm in which you use declarative code (in a manner that is similar to functional programming)
in order to build asynchronous processing pipelines. It is an event-based model where data is pushed to the consumer, as it becomes available: we deal with asynchronous sequences of events

Reactive code is assembled around asynchronous functional chains, where inputs are streamed (propagated) through these chains from the producer to the subscribers.

## Why Reactive 

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


## When: Use Cases

#### When you may not want to Use Reactive

* An app requires a significant amount of rework from Traditional MVC.
* Your app is a traditional CRUD-style app.
* You have a relational DB, but no reactive driver or abstraction .
* App has low concurrency (< 1,000 concurrent users).
* Using an older version of Servlet (3.1+ is required).
* Your data sources are **blocking** on I/O and do not scale.
* You depend on ORM features like caching, lazy loading, write-behind.

#### When you may want to use Reactive

* Relational Database with reactive driver or R2DBC abstraction (MS SQL, Postgres).
* Non-relational reactive DB (Mongo, Cassandra, Redis).
* Application with more than 1000 concurrent users.
    * Web/Mobile apps.
    * Remote apps requiring data refresh(ex- backpressure).
* A large number of transaction-processing services.
* Notification services.

  > A banking example of application around Reactive principles could be online car shopping and financing.
  > Customers can browse million cars from over thousand's of dealers and pre-qualify for financing in seconds, without impacting credit scores.


## Message-Driven Communication
One of the widely asked question is how to connect components in the distributed system and preserve decoupling, isolation, and scalability at the same time.
Let's deep dive into below code

```java
   @RequestMapping("/resource")                                       // (1)
   public Object processRequest() {
       RestTemplate template = new RestTemplate();                    // (2)
   
       ExamplesCollection result = template.getForObject(             // (3) 
          "http://abc.com/api/resource2",                              
          ExamplesCollection.class                                    
       );                                                             
       processResultFurther(result);                                  // (4)
   }
```

In the preceding example, we have defined the request handler which will be invoked on users' requests.
In turn, each invocation of the handler produces an additional HTTP call to an external service and then subsequently executes another processing stage. 
Despite the fact that the preceding code may look familiar and transparent in terms of logic, it has some flaws. 
To understand what is wrong in this example, let's take an overview of the following request's timeline:
 ![](/images/blocking-thread.PNG)

only a small part of the processing time is allocated for effective CPU usage whereas the rest of the time thread is being blocked by the I/O and cannot be used for handling other requests.
Java world, we have thread pools, which may allocate additional threads to increase parallel processing.
However, under a high load, such a technique may be extremely inefficient to process the new I/O task simultaneously. So,
So to achieve better resource utilization in I/O cases, we should use an **asynchronous** and **non-blocking** interaction model.
In real life, this kind of communication is **messaging**.
![](/images/NonBlocking.png)

## Project Reactor

Project Reactor is a fully non-blocking foundation with back-pressure support included. 
It’s the foundation of the reactive stack in the Spring ecosystem and is featured in projects such as Spring WebFlux, Spring Data, and Spring Cloud Gateway.

## Reactive Microservices with Spring Boot

The Spring portfolio provides two parallel stacks. One is based on a Servlet API with Spring MVC and Spring Data constructs.
The other is a fully reactive stack that takes advantage of Spring WebFlux and Spring Data’s reactive repositories.
In both cases, Spring Security has you covered with native support for both stacks.
![](/images/Reactive-Spring.png)

## MVC vs WebFlux
|         | MVC | WebFlux |
   | :---          |    :----   |  :----   |
| **Scaling**  | Multithreading | Event Loop |
| **Default Server** | Tomcat | Netty |
| **Responses**  | List<T>, Object, ResponseEntity | Flux<T>, Mono<T> |
| **Routing**     | **Annotational**<br/> @Controller, @RequestMapping, @Controller, @RequestMapping, @GetMapping, etc.  | **Annotational**<br/>  @Controller, @RequestMapping, @GetMapping, etc.<br/> **Functional** Router+Handler
                                                      
## Performance

The performance statistics below is between spring boot 2 webflux vs spring boot 1. The applications used along with scripts are available [here](https://github.com/Tanzu-Solutions-Engineering/pace-cnd-java/tree/master/boot2-load-demo/applications)

### Performance: 300 concurrent users

|         | Synchronous | Asynchronous |
   | :---          |    :----   |  :----   |
| **Minimum Response Time**  | 302ms | 301ms |
| **Maximum Response Time Server** | 317ms | 323ms |
| **Mean Response Time**  | 306ms | 305ms |
| **Mean Requests/Second**     | 102.273 | 103.448

### Performance: 3000 concurrent users

|         | Synchronous | Asynchronous |
   | :---          |    :----   |  :----   |
| **Minimum Response Time**  | 302ms | 301ms |
| **Maximum Response Time Server** | 3734ms | 769ms |
| **Mean Response Time**  | 2506ms | 313ms |
| **Mean Requests/Second**     | 596.026 | 1011.236

### Resource Utilization

Based on the above performance numbers specially when having 3000 concurrent users you see the 
reactive application numbers are really good and the classic way of doing things (**blocking threads**)
starts to show its problems when we require performance and horizontal scalability.

One of the reasons why the Reactive Model is able to serve requests that fast is because it’s offloading the task of
_checking if an operation has completed_ to the Operating System, which has an optimized way of doing that (kqueue/epoll on Linux for example) 
and thus avoiding the overhead of syscalls and userspace/kernel switching.

Resource contention is a real problem in the blocking model and it’s one of the reasons why even with more threads the throughput is significantly lower than the reactive model.
The non-blocking I/O used in the Reactive Model allows the CPU threads to keep running (so there’s no context switch for the paused->running or running->paused transition) 
and simply ask the Operating System to be notified when a given task has been completed. This way of operating allows the CPU to keep a low number of worker threads always busy but with different units of work, in this case with different requests.

Reactive Model, is performant and suited for a high-throughput application that wants to keep its resource
consumption low(CPU, RAM).