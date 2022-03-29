+++
categories = ["recipes"]
tags = ["resilience","resilience4j", "circuitbreaker","reactor"]
summary = "Configure Circuit Breaker"
title = "1. Configure Circuit Breaker"
date = 2020-12-09T14:02:27-05:00
weight = 1
+++

## CONTEXT
This recipe walks you through the process of how to configure and implement
circuit breaker using Reactive Programming.

## How a Circuit Breaker works ?
The circuit breaker wraps a function call inside the circuit breaker object, which continuously monitors for failures. Once failure reaches the defined threshold, the circuit breaker trips,
and all further calls to the circuit breaker return with an error, without the wrapped function getting called.
![](/images/circuit-breaker-states.png)

Circuit Breaker has three states:

* CLOSED : It will be in the close state and if failures exceed the threshold value configured at any point in time, the circuit will trip
  and will move to open state.
* OPEN : It will be in the open state when the failures are recorded and the threshold is reached i.e calls will start to fail fast without even executing the function calls.
* HALF_OPEN: The circuit breaker does not make the wrapped function calls when it is OPEN. After a wait time duration has elapsed, it makes state transition from OPEN to HALF_OPEN and allows only a configurable number of calls.
  Further calls are rejected until all permitted calls have been completed. If the failure rate or slow call rate is greater than or equal
  to the threshold, the state changes back to OPEN. if the failure rate or slow call rate is below the threshold, the state changes
  back to CLOSED.
  
## Setup

1. Add below dependency to build.gradle.
   
```groovy
   implementation "io.github.resilience4j:resilience4j-spring-boot2:${resilience4jVersion}"
   implementation "io.github.resilience4j:resilience4j-reactor:${resilience4jVersion}"
```

## Solution

1. Determine and record the following **circuit breaker configuration**

   | Property        | Default Value  | Details
      | :---            |    :----   |  :---
   | failureRateThreshold | 50 | Configures the failure rate threshold in percentage. When the failure rate is equal or greater than the threshold the CircuitBreaker transitions to open and starts short-circuiting calls.
   | slidingWindowSize | 100 | Configures a threshold in percentage. The CircuitBreaker considers a call as slow when the call duration is greater than slowCallDurationThreshold When the percentage of slow calls is equal or greater the threshold, the CircuitBreaker transitions to open and starts short-circuiting calls.
   | minimumNumberOfCalls  | 100 | Configures the minimum number of calls which are required (per sliding window period) before the CircuitBreaker can calculate the error rate or slow call rate. For example, if minimumNumberOfCalls is 10, then at least 10 calls must be recorded, before the failure rate can be calculated. If only 9 calls have been recorded the CircuitBreaker will not transition to open even if all 9 calls have failed.
   | permittedNumberOfCallsInHalfOpenState | 10 | Configures the number of permitted calls when the CircuitBreaker is half open.
   | automaticTransitionFromOpenToHalfOpenEnabled | false | 	If set to true it means that the CircuitBreaker will automatically transition from open to half-open state and no call is needed to trigger the transition. A thread is created to monitor all the instances of CircuitBreakers to transition them to HALF_OPEN once waitDurationInOpenState passes. Whereas, if set to false the transition to HALF_OPEN only happens if a call is made, even after waitDurationInOpenState is passed. The advantage here is no thread monitors the state of all CircuitBreakers.
   | waitDurationInOpenState | 60000 [ms] | The time that the CircuitBreaker should wait before transitioning from open to half-open.
   | recordExceptions | empty | A list of exceptions that are recorded as a failure and thus increase the failure rate. Any exception matching or inheriting from one of the list counts as a failure, unless explicitly ignored via ignoreExceptions. If you specify a list of exceptions, all other exceptions count as a success, unless they are explicitly ignored by ignoreExceptions.
   | ignoreExceptions | empty | A list of exceptions that are ignored and neither count as a failure nor success. Any exception matching or inheriting from one of the list will not count as a failure nor success, even if the exceptions is part of recordExceptions.

2. Add _circuit_breaker_ configuration in `src/main/resources/application.yml`
   ```yaml
      resilience4j:
      circuitbreaker:
         configs:
            default:
               registerHealthIndicator: true
               slidingWindowSize: 10
               minimumNumberOfCalls: 6
               permittedNumberOfCallsInHalfOpenState: 3
               automaticTransitionFromOpenToHalfOpenEnabled: true
               waitDurationInOpenState: 5s
               failureRateThreshold: 50
               eventConsumerBufferSize: 10
               recordExceptions:
                  - org.springframework.web.client.HttpServerErrorException
                  - java.util.concurrent.TimeoutException
                  - java.io.IOException
                  - org.springframework.web.server.ResponseStatusException
               ignoreExceptions:
                  - com.wellsfargo.reactive.starter.greenfieldreactiveapplicationstarter.error.BusinessException
         instances:
            customerService:
               baseConfig: default
   ```
3. Add `@CircuitBreaker` annotation to service mentioned in `src/main/resources/application.yml`

```java
    @CircuitBreaker(name = CUSTOMER_SERVICE)
    public Flux<Customer> getAllCustomers() {
        return customerRepository.findAll()
                .switchIfEmpty(Flux.error(Mono.error(new ResponseStatusException(HttpStatus.NO_CONTENT, "Customer Not Found"))));
    }
```

**@CircuitBreaker** annotation is the annotation that will invoke the circuit breaker when anything goes wrong in the application. 
This annotation takes two parameters, first being the **service name** which is also mentioned in the configuration, the second being the **fallback** method name which is optional.
To check, if the circuit breaker actually goes into the open state or other for that matter, you need to have the said(as mentioned in the application.yaml file) failed requests.
