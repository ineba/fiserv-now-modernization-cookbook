+++
categories = ["recipes"]
tags = ["reactive","spring", "functional endpoints", "reactor","spring webflux"]
summary = "Spring WebFlux Functional Endpoints"
title = "6. Spring WebFlux Functional Endpoints"
date = 2021-01-06T14:02:27-05:00
weight = 1
+++

## CONTEXT
This recipe walks you through creating functional endpoints.

## Solution

Spring WebFlux includes WebFlux.fn, a lightweight functional programming model in which functions are used to route and handle requests and contracts are designed for immutability. 
It is an alternative to the annotation-based programming model but otherwise runs on the same *reactor-core* foundation.

1. In **WebFlux.fn** an HTTP request is handled with a **HandlerFunction:** a function that takes `ServerRequest` and returns a delayed `ServerResponse` (i.e. `Mono<ServerResponse>`). 
Both the request and the response object have immutable contracts that offer JDK 8-friendly access to the HTTP request and response. 
HandlerFunction is the equivalent of the body of a `@RequestMapping` method in the annotation-based programming model.
Incoming requests are routed to a handler function with a **RouterFunction:** a function that takes ServerRequest and returns a delayed HandlerFunction (i.e. Mono<HandlerFunction>). When the router function matches, a handler function is returned; otherwise an empty Mono. RouterFunction is the equivalent of a @RequestMapping annotation, but with the major difference that router functions provide not just data, but also behavior.
`RouterFunctions.route()` provides a router builder that facilitates the creation of routers, as the following example shows:

    ```java
        @Bean
        public RouterFunction<ServerResponse> accountsRoute(AccountHandler accountHandler) {
            return route(GET("/accounts").and(accept(MediaType.APPLICATION_JSON)), accountHandler::getAllAccounts)
    
                  .and(route(GET("/accounts/{id}").and(accept(MediaType.APPLICATION_JSON)), accountHandler::findById))
    
                  .and(route(POST("/accounts").and(accept(MediaType.APPLICATION_JSON)), accountHandler::save));
        }
    ```

1. For each route the first argument defines the request that will return the corresponding handler. In this example routes have been created to a handler function that returns a **Account** resource.
   In a typical application it is useful to group handler functions together. The following is a handler class example exposes a reactive `AccountRepository`.
    ```java
        public Mono<ServerResponse> getAllAccounts(ServerRequest request) {
            return ServerResponse.ok()
                    .contentType(MediaType.APPLICATION_JSON)
                    .body(accountRepository.findAll(), Account.class));
        }
    
        public Mono<ServerResponse> findById(ServerRequest request) {
            return ServerResponse.ok()
                    .contentType(MediaType.APPLICATION_JSON)
                    .body(accountRepository.findById(request.pathVariable("id"), Account.class));
        }
    ```
    `ServerResponse` provides access to the HTTP response and can be created using the build method. 00The builder can set the response code, response headers or a body.

1. Unit Test Case To test Route:
The **spring-test** module provides mock implementations of `ServerHttpRequest`, `ServerHttpResponse`, and `ServerWebExchange`. 
The `WebTestClient` is built on these mock request and response objects to provide support for testing WebFlux applications without an HTTP server. 
You can use the WebTestClient for end-to-end integration tests, too.

   The source code is available in: [Wells Fargo GitHub](https://)   