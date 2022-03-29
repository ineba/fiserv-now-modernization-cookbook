+++
categories = ["recipes"]
tags = ["reactive","spring", "reactor","spring webflux", "webclient"]
summary = "Spring WebClient"
title = "3.Spring WebClient"
date = 2021-01-06T14:02:27-05:00
weight = 100
+++

## CONTEXT
This recipe walks you through how to use `WebClient` and it's best practices.

## Solution
WebClient is a non-blocking, reactive client to perform HTTP requests.
`RestTemplate` class is currently in maintenance mode and as per official [Documentation](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html).

>As of 5.0 this class is in maintenance mode, with only minor requests for changes and bugs to be accepted going forward.
> Please, consider using the `org.springframework.web.reactive.client.WebClient` which has a more modern API and supports sync, async, and streaming scenarios.

### Difference between WebClient and RestTemplate
The main difference between the two is RestTemplate works _synchronously_ (blocking) and WebClient works _asynchronously_ (non-blocking).
**RestTemplate** is a *synchronous* client to perform HTTP requests, exposing a simple, template method API over underlying HTTP client libraries such as the JDK HttpURLConnection, Apache HttpComponents, and others.
**WebClient** is an asynchronous, reactive client to perform HTTP requests, a part of Spring WebFlux framework.

## Setup

There are several ways to configure the WebClient.
1. Create `WebClient` with default settings:
   
   ```java
      WebClient webClient = WebClient.create();
   ```
  
1. Create `WebClient` instance using URL.

    ```java
       WebClient webClient = WebClient.create("${REST_URL}");
    ```

1. Create `WebClient` using `DefaultWebClientBuilder` which allows customization **URL** and **TimeOuts**
    ```java
       WebClient webClient = WebClient
            .builder()
            .baseUrl("${REST_URL}")
            .defaultCookie("cookieKey", "cookieValue")
            .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
            .build();
    ```

1. Create `WebClient` instance with Timeouts:
   The default HTTP timeout is **30** seconds. If you want to change this setting and configure WebClient you 
   can do so using `TcpClient` class. In this class we can set the connection timeout using _ChannelOption.connect_TIMEOUT_MILLIS_
   value. We can also set **read** and **write** timeouts using a _ReadTimeoutHandler_ and a _WriteTimeoutHandler_ respectively
   
    ```java
       TcpClient tcpClient = TcpClient
                               .create()
                               .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 5000)
                               .doOnConnected(connection -> {
                                connection.addHandlerLast(new ReadTimeoutHandler(5000, TimeUnit.MILLISECONDS));
                                connection.addHandlerLast(new WriteTimeoutHandler(5000, TimeUnit.MILLISECONDS));
                               });

       WebClient client = WebClient.builder()
                                   .clientConnector(new ReactorClientHttpConnector(HttpClient.from(tcpClient)))
                                   .build();    
    ```

## WebClient Request

`WebClient` supports following methods:
* get()
* post()
* put()
* patch()
* delete()

You can optionally also specify following options:

* Path Variables or Query Parameters with `uri()` method.
* RequestHeaders with `headers()` method.
* Custom cookies with `cookies()` method.

The call to Http Endpoint is made using `retrieve()` and `exchange()`

> Please do not use `exchange()` as it has been attributed to memory leaks in production code.

### Asynchronous Request

1. Create Asynchronous request to retrieve Customer:
    
    ```java
       public Mono<Customer> getCustomerById() {
       
           return client
                   .get()
                   .uri("customer/123")
                   .retrieve()
                   .bodyToMono(Customer.class);
   
       }
    ```

1. In the above example we specified that we want to make a **GET** request using `get()` method following by adding the URI.
The `retrieve()` method retrieves the response body and then it is extracted to a `Mono`. In reactive applications 
nothing happens until you `subscribe` to a `Mono` or `Flux` like below:

    ```java
         customerService
              .getCustomerById("1")
              .subscribe(customer -> log.info("Customer: {}", customer));  
    ```
 
   > Note: Even if server takes time to respond, the execution immediately continues without blocking subscribe() call.

### Synchronous Request:

`WebClient` can be also be used to replace `RestTemplate` in Spring Boot application. This can be done by 
using `block()` method. The `block()` method blocks the thread.

1. Create Synchronous request to retrieve Customer:
    ```java
       public User getCustomerByIdSync(final String id) {
          return webClient
                        .get()
                        .uri(String.join("", "/customer/", id))
                        .retrieve()
                        .bodyToMono(Customer.class)
                        .block();
       }
    ```

## Retries on Failure

1. `Webclient` provides `retryWhen` function which takes in `reactor.util.retry.Retry` class as an argument and can be used
    to retry failed call over a network::

    ```java
       public Customer getCustomerWithRetry(final String id) {
          return webClient
                      .get()
                      .uri(String.join("", "/customer-broken/", id))
                      .retrieve()
                      .bodyToMono(Customer.class)
                      .retryWhen(Retry.fixedDelay(3, Duration.ofMillis(100)));
       }

    ```
   
1. You can also configure `retryWhen` for [**Exponential Backoff**](https://projectreactor.io/docs/core/release/reference/#faq.exponentialBackoff)
   
## Error Handling

1. In case of an error when the retry does not help, we can still control the situation with a fallback.
The `doOnError()` triggers when `Mono` completes with an error. `onErrorResume()` as described in 
[spring-webflux-error-handling-recipe](/recipes-reactive/spring-webflux-error-handling) subscribes to a **fallback**
publisher when any error occurs, using a function to choose the fallback depending on the error:

    ```java
       public Customer getCustomerWithFallBack(final String id) {
          return webClient
                          .get()
                          .uri(String.join("", "/customer-broken/", id))
                          .retrieve()
                          .bodyToMono(Customer.class)
                          .doOnError(error -> log.error("An error has occurred {}", error.getMessage()))
                          .onErrorResume(error -> Mono.just(Customer.builder.build()));
       }
    ```
1. There are situations where you may want to react to a _specific error code_. We can use `onStatus`
   method:
    ```java
       public User getCustomerWithErrorHandling(final String id) {
            return webClient
                        .get()
                        .uri(String.join("", "/customer-broken/", id))
                        .retrieve()
                        .onStatus(HttpStatus::is4xxClientError,
                                  error -> Mono.error(new RuntimeException("API not found")))
                        .onStatus(HttpStatus::is5xxServerError,
                                  error -> Mono.error(new RuntimeException("Server is not responding")))
                        .bodyToMono(Customer.class);
        } 
    ```
   
The code snippets can be found in [Wells Fargo GitHub](https://)