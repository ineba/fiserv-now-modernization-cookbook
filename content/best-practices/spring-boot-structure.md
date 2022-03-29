+++
categories = ["recipes"]
tags = ["practices", "spring boot", "microservice", "application development"]
title = "1.Organize Spring Boot App"
description = "A simple guide to organize codebase in a microservice"
date = 2020-12-09
weight = 10
draft = false
authors = ["Rohan Mukesh"]
+++

## CONTEXT
This recipe provides details for organizing codebase in a typical spring boot microservice.

## SOLUTION

### Layers

#### Why even have layers today? What year is this?
> *Most of the examples I see have one Spring bean which does configuration, endpoints, data, security, validation and pancakes...*

An argument can be made that trivial services (especially focused examples) have no need for the overhead of multiple layers, packages, separation of duties, abstraction, and even basic organization.  

After all, we aren't building monoliths anymore, we're building microsevices right?

Yes, but we are looking for a balance here between overhead and chaos.

**We know at least the following:**

- We need just enough abstraction for things to change easily (layers & interfaces)
- Other developers have to make sense of our code (convention & organization)
- Our code has to be easily testable (isolation & separation of duties).

- Another big one: Annotation based frameworks commonly utilize [dynamic proxies](https://docs.oracle.com/javase/8/docs/technotes/guides/reflection/proxy.html) which only works on public method calls to spring-injected beans (i.e. hystrix, JPA, transactions etc.)

> *But even the Spring Initializr only gives me dependencies, and a single Application class*

The rest is left to you simply because there are so many options. 

Still here? Let's look at a possible strategy.  

### API Based Services

Most http/api Spring Boot applications consist of 3 primary layer types.

```
       Controller (API Front End - Integration with external consumers)
       
                                   |
                                   v
       
       Services (POJO Capabilities i.e. the valuable stuff)
       
                                   |
                                   v
      
      Integration (Integration with external producers)
```

Integration classes are injected into services, services are injected into controllers or other services for composition.

Calls through layers go down, never up (no integration classes calling Services).

### Controller Layer
This is the API exposure layer. It accepts and returns http request/response and defines how external clients will interact with the service.

You can organize parts of the API into separate controller classes, though *sometimes* this is an indication that the service should be split into smaller microservices.

This should be the only layer that "knows" it is a webservice at all (don't pass raw http request/response through service layer).

> **IMPORTANT:** Do not make database calls or calls to other services in the controller.   
> As much as possible, place only http related mapping/translating/versioning code here (and potentially authorization code depending on your design).

It is simply a http adapter to the capabilities of your service.

The controller layer has the final exception handling responsibilities as well: [Controller Advice](https://dzone.com/articles/global-exception-handling-with-controlleradvice)

### Service Layer

This layer contains one or more classes representing the internal composable capabilities of the app.

The name "service" used here can sometimes confuse developers new to Spring vernacular. It refers to classes which are the composable, plain Java, "middle bit" of the application which do not deal with protocols or other integration concerns.  

Services should contain things like business logic, data transformation, user authorization and calls to the DAOs.

Services can aggregate/compose/orchestrate across other services

### Integration Layer

Contains one or more classes which perform integrations with other webservices, databases, message queues, etc.

 **Common naming conventions for integration classes depending on their purpose:**  
- DAO (A bit too generic, but widely used)  
- Repository (Mostly CRUD data operations)  
- Sender (Message sender over Kafka, JMS, AMQP etc.)  

> UNIMPORTANT: For an integration which is more than just data operations (think tax calculation), some folks use a different suffix such as "Engine".

**Example:**

```
           AccountController
                   |
            AccountService
              /          \
AccountDataService    CustomerService
        /                      \
 *AccountRepo*        *CustomerRepo*
```

Integration classes should follow an interface/implementation pattern.  
`AccountRepo ` would be the interface, `JDBCAccountRepo` would be a potential implementation of the interface (I know naming things is hard, Please don't call it an "Impl")

### Message Services

Message services are structured nearly the same as an API service, but instead of a controller they have a message listener.

**Example:**

```
          AccountMessageListener
                   |
            AccountService
                   |
             AccountRepo
```

## Non-layer Classes

### Models

Contains plain java classes which represent data passed between the layers of an application.

Each layer *may* have its own models depending on the abstraction needed (one model or "translate everywhere").

Many difficult trade-offs are made here between simplicity and abstraction.

### Application

A single class which declares your app as Spring Boot-able.

### Config

Any spring bean config which is not embedded in the bean itself (annotations) will go here. This is usually limited to security config and declaration of spring beans that you did not write.

## Input Validation

Where to do input validation for attribute x is often a subject of debate.  
Since there is no one right answer for all input types, we'll just list some things to consider when making the decision.

*Wherever you choose to validate, it's usually best to move this to its own class.*

**Considerations:**

1. Some input validation (format or required elements) can change depending on the API version of the service
2. Input validation error responses should include the attribute value and location which caused the error (if safe to do so)
3. Some input validation is consistent across API versions and duplication of code may not be desirable
4. If you use Spring Bean Validation, you have less code to write but also less control over where/when things happen

## API Versioning

> **NOTE:** This section is not meant to be a full API versioning strategy (that's a different recipe).
> It should only recommend where to implement a versioning strategy within the layers described above.
> If it accidentally oversteps, refer to the main versioning recipe.

API major-versioning should be done as a last resort because it adds a hefty maintenance cost.
Favor backward compatible API changes when possible.

If you choose to maintain multiple major versions of an API (within the same app) this versioning should be done in the controller. Any data translation which is version specific should be done here.

Each major version should be represented by a separate controller class

Example: `AccountControllerV1` & `AccountControllerV2`



#### To illustrate, consider the `account-service` which supports 3 versions concurrently:

**Option #1 - Chaining Translation**

```
   AccountControllerV1  -> AccountControllerV2  ->  AccountControllerV3
                                                       |
                                                  AccountService
                                                       |
                                                    AccountRepo
```

**Option #2 - Direct Translation**

```
   AccountControllerV1   AccountControllerV2    AccountControllerV3
                   \           |             /
                          AccountService
                               |
                            AccountRepo
```

The `AccountService` (business logic and orchestration) does not know anything about versioning.
It implicitly represents the highest version.

The highest version controller (`AccountControllerV3`) does not do any versioning, it just calls the `AccountService `.

Lower version controllers either translate up in a chain (v1->v2->v3) or translate and directly call the `AccountService ` (v1->v3).

#### Does this mean I have a different set of API models for each controller/version?

If the major version change is non-trivial and involves structural changes to the API model classes, yes. Another reason to avoid this.

## Package Organization

Again, there are many ways to do this just keep in mind that consistency across projects makes it easier for new developers (though keeping services extremely small makes this slightly less important).

#### Horizontal or vertical? Package by layer or package by feature?

[https://dzone.com/articles/project-package-organization](https://dzone.com/articles/project-package-organization)

### Package by Layer Example

```
com.wellsfargo.cto.eai/
                   TrackingApplication.java
                   
com.wellsfargo.cto.eai.config/
                   SecurityConfig.java
                   RedisConfig.java
                   
com.wellsfargo.cto.eai.controller/
                   AccountController.java
                   AccountControllerAdvice.java
                   AccountInputValidator.java
                   
com.wellsfargo.cto.eai.controller.model/
                   AccountRequest.java
                   AccountResponse.java
                   ...
                   
com.wellsfargo.cto.eai.model/
                   AccountStatus.java
                   Customer.java
                   ...
                   
com.wellsfargo.cto.eai.service/
                   AccountService.java
                   CustomerService.java
                   
com.wellsfargo.cto.eai.repo/
                   AccountRepo.java
                   JDBCAcountRepo.java
                   CustomerRepo.java
                   CDSCustomerRepo.java
                   
com.wellsfargo.cto.eai.repo.model/
                   Customer.java							
```

### Package by Feature Example
