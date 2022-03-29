+++
categories = ["recipes"]
tags = ["reactive","spring", "reactor","spring webflux", "mdc", "logging"]
summary = "Spring WebFlux MDC Logging"
title = "9. Spring WebFlux MDC Logging"
date = 2021-01-19T14:02:27-05:00
weight = 1
+++

## CONTEXT
In the [spring-webflux-webfilter](/recipes-reactive/spring-webflux-webfilter) we saw how to implement `WebFilter` to add request headers
to the context and how to add context to response headers. 

This recipe will walk you through logging along with context in Spring WebFlux.

## SOLUTION

1. To add [context](https://projectreactor.io/docs/core/release/reference/#context) information to the Context, 
   We will use `subscriberContext` method present on all Mono and Flux instances. 
   It accepts a `Function<Context, Context>` which transforms the existing immutable Context into a new Context.
    ```java
    @GetMapping("/{customerId}")
    public Mono<Customer> getCustomerById(@PathVariable String customerId){
            return customerService.findById(customerId)
            .doOnEach(logOnNext(customer -> log.info("Customer: {}", customer)))
            .subscriberContext(LogHelper.put("CUSTOMER-ID", customerId));
    }    
    ```
1. Create a new class `LoggingHelper` class and add method `put` like below:

    ```java
    public static Function<Context, Context> put(String key, String value) {
         return context -> {
             Optional<Map<String, String>> contextMap = context.getOrEmpty(CONTEXT_MAP);
             if (contextMap.isPresent()) {
                 contextMap.get().put(key, value);
                 return context;
             } else {
                 Map<String, String> ctxMap = new HashMap<>();
                 ctxMap.put(key, value);
                 return ctx.put(CONTEXT_MAP, ctxMap);
             }
         };
    }
    ```
    The helper function `put` provides the required `Function<Context, Context>` that adds the given key and value to the context or creates a new context if none exists.
    `subscriberContext` is always added as a last operation in the chain of calls. As per the Reactor documentation:

     {{% note  %}}
        Even though subscriberContext is the last piece of the chain, it is the one that gets executed first (due to its subscription time nature, and the fact that the subscription signal flows from bottom to top). 
     {{% /note  %}}
 
    With the approach suggested by **Simon Basle** in this [post](https://simonbasle.github.io/2018/02/contextual-logging-with-reactor-context-and-mdc/), We will utilize
    `doOnEach` method present on all Mono and Flux instances.

1. Create helper method `logOnNext` as below:

    ```java
    public static <T> Consumer<Signal<T>> logOnNext(Consumer<T> log) {
          return signal -> {
              if (signal.getType() != SignalType.ON_NEXT) return;
    
              Optional<Map<String, String>> maybeContextMap = signal.getContext().getOrEmpty(CONTEXT_MAP);
    
              if (maybeContextMap.isEmpty()) {
                  log.accept(signal.get());
              } else {
                    MDC.setContextMap(maybeContextMap.get());
                    try {
                        log.accept(signal.get());
                  } finally {
                        MDC.clear();
                  }
              }
          };
    }    
    ```
   Both invokes a lambda that accepts the current result or the exception respectively. The helper methods logOnNext are best placed in a helper class. 
   Both methods extract's the context information from the `doOnEach` Signal and set it as the **MDC** before calling the provided lambda.

1. Update `CONSOLE_LOGGING_PATTERN` in `logback-spring.xml` or update `logging.pattern.console` in `src/main/resources/application.yml` file
 
    ```yaml
       logging:
           pattern:
              console: %d{dd-MM-yyyy HH:mm:ss.SSS} %magenta([%thread]) %highlight(%-5level) %logger.%M - %mdc%msg%n
    ```
    The source code is available in: [Wells Fargo GitHub](https://)   