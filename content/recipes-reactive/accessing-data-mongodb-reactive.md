+++
categories = ["recipes"]
tags = ["persistence","data-repository", "mongodb","spring boot"]
summary = "Accessing mongodb data reactively"
title = "5. Access Data in MongoDB"
date = 2020-12-09T14:02:27-05:00
weight = 1
+++

## CONTEXT
This guide walks you through the process of how to configure and implement
database operations using Reactive Programming through Spring data Reactive Repositories and Template with MongoDB.

## SOLUTION
## Define Entity
MongoDB is a NoSQL document store. In this example, you store `Customer` objects. 

```java
  @Builder
  @AllArgsConstructor
  @Getter
  @Setter
  @ToString
  public class Account {

    @Id
    private String id;
    private String accountNumber;
    private String routingNumber;
    private String accountOwner;
}
```

1. **Account** class has four attributes: **id**, **accountNumber**, **routingNumber** and **accountOwner**. 
   The **id** is mostly for internal use by MongoDB. You also have a single constructor to 
   populate the entities when creating a new instance.

1. **id** fits the standard name for a MongoDB ID, so it does not require any special 
   annotation to tag it for Spring Data MongoDB.

1. The other three properties, **accountNumber**, **routingNumber** and **accountOwner** are left unannotated. 
   It is assumed that they are mapped to fields that share the same name as the properties themselves.

The convenient **@ToString()** method prints out the details about a customer.

---
 **NOTE**

1. MongoDB stores data in collections. Spring Data MongoDB maps the **Customer** class
into a collection called **customer**. If you want to change the name of the collection, you
can use Spring Data MongoDB's
[@Document](https://docs.spring.io/spring-data/data-mongodb/docs/current/api/org/springframework/data/mongodb/core/mapping/Document.html)
annotation on the class.

---

## Create Queries

### ReactiveMongoRepository

Spring Data MongoDB focuses on storing data in MongoDB. _ReactiveMongoRepository_ interface inherits
from _ReactiveCrudRepository_ and adds new query methods:

 ```java
   import org.springframework.data.mongodb.repository.ReactiveMongoRepository;

   public interface AccountRepository extends ReactiveMongoRepository<Account, String> {
   
      Mono<Account> findByAccountOwner(String accountOwner);
   
      @Query("{ 'accountNumber': ?0, 'routingNumber': ?1}")
      Mono<Account> findByAccountNumberAndRoutingNumber(Long accountNumber, Long routingNumber);
   }
 ```
Using the ReactiveMongoRepository, we can query by example.

### ReactiveMongoTemplate

Besides the repositories approach, there is also ReactiveMongoTemplate:

```java
      @Service
      @AllArgsConstructor
      public class AccountTemplateOperations {
   
      private final ReactiveMongoTemplate template;
   
      public Mono<Account> findById(String id) {
         return template.findById(id, Account.class);
      }
   
      public Flux<Account> findAll() {
         return template.findAll(Account.class);
      }
   
      public Mono<Account> save(Mono<Account> account) {
         return template.save(account);
      }
   
      public Flux<Account> findByFirstNameAndLastName(String firstName, String lastName) {
         Query query = new Query();
         query.addCriteria(Criteria.where("firstName").is(firstName).and("lastName").is(lastName));
         return template.find(query, Account.class);
      }
   }
```

## Testing

1. Using **Junit5** and Spring Test Framework we can write Integration Tests for MongoDB
   
1. Project reactor provides library `io.projectreactor:reactor-test` which is used to test reactive streams. One of the key
   elements in reactor test library is: **StepVerifier**.
   
1. StepVerifier provides a declarative way of creating verifiable steps for async publisher sequence by expressing expectations
   about the set of events that will eventually happen upon subscription. Example below

```java
   @ExtendWith(SpringExtension.class)
   @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT, classes = GreenfieldReactiveApplication.class)
   public class AccountRepositoryTest {
   
      @Autowired
      private AccountMongoRepository repository;
   
      @Test
      public void testFindById() {
         //given
         Account account = repository.save(new Account(null, 918345L, 234518L, "alex"))
                 .block();
   
         //when
         Mono<Account> createdAccount = repository.findById(account.getId());
   
         //then
         StepVerifier
                 .create(createdAccount)
                 .assertNext(acc -> {
                    assertThat("alex").isEqualTo(acc.getAccountOwner());
                    assertThat(234518L).isEqualTo(acc.getRoutingNumber());
                    assertThat(acc.getId()).isNotNull();
                 })
                 .expectComplete()
                 .verify();
   
      }
   
      @Test
      public void testSave() {
         Mono<Account> accountMono = repository.save(new Account(null, 918345L, 234518L, "alex"));
   
         StepVerifier
                 .create(accountMono)
                 .assertNext(account -> assertThat(account.getId()).isNotNull())
                 .expectComplete()
                 .verify();
      }
   
      @Test
      public void testFindByAccountOwner() {
         //given
         Account account = repository.save(new Account(null, 918345L, 234518L, "ron"))
                 .block();
   
         //when
         Mono<Account> createdAccount = repository.findByAccountOwner(account.getAccountOwner());
   
         //then
         StepVerifier
                 .create(createdAccount)
                 .assertNext(acc -> {
                    assertThat("ron").isEqualTo(acc.getAccountOwner());
                    assertThat(234518L).isEqualTo(acc.getRoutingNumber());
                    assertThat(acc.getId()).isNotNull();
                 })
                 .expectComplete()
                 .verify();
      }
   
      @Test
      public void testFindByAccountNumberAndRoutingNumber() {
         //given
         Account account = repository.save(new Account(null, 918345L, 235189L, "john"))
                 .block();
   
         //when
         Mono<Account> createdAccount = repository.findByAccountNumberAndRoutingNumber(account.getAccountNumber(), account.getRoutingNumber());
   
         //then
         StepVerifier
                 .create(createdAccount)
                 .assertNext(acc -> {
                    assertThat("john").isEqualTo(acc.getAccountOwner());
                    assertThat(235189L).isEqualTo(acc.getRoutingNumber());
                    assertThat(acc.getId()).isNotNull();
                 })
                 .expectComplete()
                 .verify();
      }
   
      @Test
      public void testFindAll() {
         //given
         repository.save(new Account(null, 918345L, 234518L, "mike"))
                 .block();
         ExampleMatcher matcher = ExampleMatcher.matching().withMatcher("accountOwner", startsWith());
         Example<Account> example = Example.of(new Account(null, 918345L, 234518L, "mike"), matcher);
   
         //when
         Flux<Account> accountFlux = repository.findAll(example);
   
         //then
         StepVerifier
                 .create(accountFlux)
                 .assertNext(account -> assertThat("mike").isEqualTo(account.getAccountOwner()))
                 .expectComplete()
                 .verify();
      }
   }
```
    The source code is available in: [Wells Fargo GitHub](https://)   
