+++
categories = ["recipes"]
tags = ["reactive","spring", "reactor","spring webflux", "webfilter"]
summary = "Spring WebFlux WebFilter"
title = "7. Spring WebFlux WebFilter"
date = 2021-01-19T14:02:27-05:00
weight = 1
+++

## CONTEXT
Slf4j provides MDC (Mapped Diagnostic Contexts) feature that allows you to enrich logs with contextual data.
Example of contextual data might include but is not limited to the following:

* Request Id,
* The ID of the client initiating the request,
* DNS name of the hardware involved in processing the request.

In a microservices environment, this information is then typically passed to all services involved in handling a specific client request.
This recipe will walk you through how to best pass context information between services as HTTP non-standard headers.

## SOLUTION

`WebFilter`s function much like their [servlet counterparts](https://www.oracle.com/java/technologies/filters.html), but using Spring
and Reactor APIs. Spring WebFlux provides a `WebFilter` interface that can be implemented to filter HTTP request-response exchanges. WebFilter beans found in the application context will be automatically used to filter each exchange.
Where the order of the filters is important they can implement Ordered or be annotated with `@Order`

1. Create a new class `MdcHeaderFilter` which implements `WebFilter` interface.

    ```java
        @Component
        public class MDCHeaderFilter implements WebFilter {
        private static final String MDC_HEADER_PREFIX = "MDC-";
        public static final String CONTEXT_MAP = "context-map";
    
        @Override
        public Mono<Void> filter(ServerWebExchange serverWebExchange, WebFilterChain chain) {
            serverWebExchange.getResponse()
                    .beforeCommit(() -> addContextToHttpResponseHeaders(serverWebExchange.getResponse()));
    
            return chain.filter(serverWebExchange)
                    .subscriberContext(ctx -> addRequestHeadersToContext(serverWebExchange.getRequest(), ctx));
        }
    ```
1. Add method `addContextToHttpResponseHeaders`. The method copies the MDCs passed as request headers to the [context](https://projectreactor.io/docs/core/release/reference/#context).
   It gets invoked first (before any `@Controller`S)

    ```java
        private Context addRequestHeadersToContext(final ServerHttpRequest request, final Context context) {
        final Map<String, String> contextMap = request
            .getHeaders().toSingleValueMap().entrySet()
            .stream()
            .filter(x -> x.getKey().startsWith(MDC_HEADER_PREFIX))
            .collect(
            toMap(v -> v.getKey().substring(MDC_HEADER_PREFIX.length()), Map.Entry::getValue));
        return context.put(CONTEXT_MAP, contextMap);
       }
    ``` 

1. Add method `addRequestHeadersToContext`. The method copies the [context](https://projectreactor.io/docs/core/release/reference/#context) to the response. It 
   gets invoked after the request has been processed by the controllers. It will inspect all headers of every request passed through it and 
   extract all headers starting with **MDC-** and populate a map with them. 
     
    ```java
    private Mono<Void> addContextToHttpResponseHeaders(final ServerHttpResponse res) {
    
       return Mono.subscriberContext()
                 .doOnNext(context -> {
       if (!context.hasKey(CONTEXT_MAP)) return;
    
       final HttpHeaders headers = res.getHeaders();
       context.<Map<String, String>>get(CONTEXT_MAP)
              .forEach((key, value) -> headers.add(MDC_HEADER_PREFIX + key, value));})
              .then();
    } 
    ``` 

    {{% note  %}}
      The headers starting with prefix **X-** is deprecated and is discouraged. [RFC](https://tools.ietf.org/html/rfc6648)
    {{% /note  %}}

1. We can test `WebFilter` using `WebTestClient` by sending a **POST** request and check whether we are able to retrieve
   **MDC** headers or not:
   
    ```java
    @Test
    public void testHeader() {
    
            Customer customer = Customer.builder().firstName("john")
            .lastName("smith")
            .phoneNumber("7589756789")
            .build();
    
            webTestClient.post()
            .uri("/customer")
            .contentType(MediaType.valueOf(MediaType.APPLICATION_JSON_VALUE))
            .header("MDC-CUSTOMER-ID", "123")
            .body(Mono.just(customer),Customer.class)
            .exchange()
            .expectStatus().isCreated()
            .expectHeader()
            .exists("MDC-CUSTOMER-ID")
            .expectBody()
            .jsonPath("$.customerId").isNotEmpty()
            .jsonPath("$.firstName").isEqualTo("john");
    }       
    ```
   The source code is available in: [Wells Fargo GitHub](https://)    