+++
categories = ["recipes"]
tags = ["reactive","spring", "reactor","spring webflux"]
summary = "Spring WebFlux Exception Handling"
title = "8. Spring WebFlux Exception Handling"
date = 2021-01-06T14:02:27-05:00
weight = 1
+++

## CONTEXT
This recipe walks you though how to do error handling in a spring webflux application.

## SOLUTION
In Reactive Streams, errors are terminal events. As soon as an error occurs, it stops the sequence and gets propagated down the chain of operators to the last step, 
the defined **Subscriber** and its **onError** method.

Such errors should still be dealt with at the application level.
For instance, you might display an error notification in a UI or send a meaningful error payload in a REST endpoint. 
For this reason, the subscriber’s **onError** method should always be defined.

> If not defined, onError throws an **UnsupportedOperationException**.

One of the most important things to note is _any error in a reactive sequence is a terminal event_. Even if error-handling operator is used, it does
not let the original sequence to continue. Rather, it converts the **onError* signal into the start of new sequence (fallback)

### Error Handling Operators
You may be familiar with several ways of dealing with exceptions in a **try-catch** block. Most notably, these include the following:

* Catch and return a static default value.
* Catch and execute an alternative path with a fallback method.
* Catch and dynamically compute a fallback value.
* Catch, wrap to a BusinessException, and re-throw.
* Catch, log an error-specific message, and re-throw. 
  
All of these have equivalents in Reactor, in the form of error-handling operators

* **Static Fallback Value:**
  The equivalent of “Catch and return a static default value” is **onErrorReturn**. The following example shows how to use it:

    ```java
    public Mono<ServerResponse> handleRequest(ServerRequest request) {
        return findById(request.pathVariable("id"))
          .onErrorReturn("No Customer Found")
          .flatMap(s -> ServerResponse.ok()
          .contentType(MediaType.APPLICATION_JSON)
          .syncBody(s));
    }
    ```
* **Fallback Method:**
If you want more than a single default value and you have an alternative (safer) way of processing your data, you can use onErrorResume. 
This would be the equivalent of “Catch and execute an alternative path with a fallback method”. The following example shows how to use it:

    ```java
    public Mono<ServerResponse> handleRequest(ServerRequest request) {
        return findById(request.pathVariable("id"))
          .flatMap(s -> ServerResponse.ok()
          .contentType(MediaType.TEXT_PLAIN)
          .syncBody(s))
          .onErrorResume(e -> findByIdFallback()
                              .flatMap(s ->; ServerResponse.ok()
                              .contentType(MediaType.APPLICATION_JSON)
                              .syncBody(s)));
    }
    ```

* **Dynamic Fallback Value:**
If you do not have an alternative way of processing your data, you might want to compute a fallback value
out of the exception you have received. This would be equivalent of "Catch and dynamically compute a fallback value".
The following example shows how to use it:

    ```java
    public Mono<ServerResponse> handleRequest(ServerRequest request) {
        return findById(request.pathVariable("id"))
              .flatMap(s -> ServerResponse.ok()
              .contentType(MediaType.APPLICATION_JSON)
              .syncBody(s))
              .onErrorResume(e -> Mono.just("Error " + e.getMessage())
                                    .flatMap(s -> ServerResponse.ok()
                                .contentType(MediaType.APPLICATION_JSON)
                                .syncBody(s)));
    }
    ``` 

* **Catch and Rethrow:**
The final option using _onErrorResume()_ is to catch, wrap, and re-throw an error e.g. InvalidAccountException:

    ```java
    public Mono<ServerResponse> handleRequest(ServerRequest request) {
        return ServerResponse.ok()
          .body(findById(request.pathVariable("id"))
          .onErrorResume(e -> Mono.error(new InvalidAccountException(
                                        HttpStatus.BAD_REQUEST, 
                                        "Account is invalid", e))), Account.class);
    }
    ```

## Handling Errors at a Global Level

The above examples provide ways to handle errors at functional level.
Spring Boot provides a `WebExceptionHandler` that handles all errors in a sensible way. 
Its position in the processing order is immediately before the handlers provided by WebFlux, which are considered last. 
For machine clients, it produces a JSON response with details of the error, the HTTP status, and the exception message. 
For browser clients, there is a “whitelabel” error handler that renders the same data in HTML format. 

1. The first step to customizing this feature often involves using the existing mechanism but replacing or augmenting 
the error contents. For that, you can add a bean of type `ErrorAttributes`.
   
    ```java
    @Component
    public class ErrorAttributes extends DefaultErrorAttributes {
        
       @Override
       public Map<String, Object> getErrorAttributes(ServerRequest request, ErrorAttributeOptions options) {
         Map<String, Object> errorAttributesMap = super.getErrorAttributes(request, options);
         Throwable throwable = getError(request);
         if (throwable instanceof  ResponseStatusException) {
           ResponseStatusException responseStatusException = (ResponseStatusException) throwable;
           errorAttributesMap.put("message", responseStatusException.getMessage());
         }
         return errorAttributesMap;
       }
    }
    ```

1. To change the error handling behavior, you can implement `ErrorWebExceptionHandler` and register a bean definition of that type. 
Because a WebExceptionHandler is quite low-level, Spring Boot also provides a convenient `AbstractErrorWebExceptionHandler` 
to let you handle errors in a WebFlux functional way, as shown in the following:
   
   
    ```java
    @Component
    @Order(-2)
    public class WebExceptionHandler extends AbstractErrorWebExceptionHandler {
        
       public WebExceptionHandler(ErrorAttributes errorAttributes, ResourceProperties resourceProperties,
                                  ApplicationContext applicationContext, ServerCodecConfigurer codeConfigurer) {
         super(errorAttributes, resourceProperties, applicationContext);
         this.setMessageWriters(codeConfigurer.getWriters());
       }
       
       @Override
       protected RouterFunction<ServerResponse> getRoutingFunction(ErrorAttributes errorAttributes) {
         return RouterFunctions.route(RequestPredicates.all(),this::formatErrorResponse);
       }
        
       private Mono<ServerResponse> formatErrorResponse(ServerRequest request) {
         Map<String, Object> errorAttributesMap = getErrorAttributes(request, ErrorAttributeOptions.of(ErrorAttributeOptions.Include.STACK_TRACE));
         int status = (int) Optional.ofNullable(errorAttributesMap.get("status")).orElse(500);
        
         return ServerResponse
                 .status(status)
                 .contentType(MediaType.APPLICATION_JSON)
                 .body((BodyInserters.fromValue(errorAttributesMap)));
       }
    }
    ```
We are setting the order of our error handler to -2. This is to give it a higher priority than the `DefaultErrorWebExceptionHandler` which is registered at `@Order(-1)`

    The source code is available in: [Wells Fargo GitHub](https://)   