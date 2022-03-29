+++
categories = ["recipes"]
tags = ["persistence","data-repository", "mongodb","spring boot"]
summary = "Accessing data in MongoDB"
title = "5. Access Data in MongoDB"
date = 2020-12-09T14:02:27-05:00
weight = 1
+++

## CONTEXT
This guide walks you through the process of using Spring Data MongoDB to build an
application that stores data in and retrieves it from [MongoDB](https://www.mongodb.org/), a
document-based database.

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

Spring Data MongoDB focuses on storing data in MongoDB. It also inherits functionality
from the Spring Data Commons project, such as the ability to derive queries.

 ```java
   import org.springframework.data.mongodb.repository.MongoRepository;

   public interface AccountRepository extends MongoRepository<Account, String> {
   
      Account findByAccountOwner(String accountOwner);
   
      @Query("{ 'accountNumber': ?0, 'routingNumber': ?1}")
      Account findByAccountNumberAndRoutingNumber(Long accountNumber, Long routingNumber);
   }
 ```

1. **AccountRepository** extends the **MongoRepository** interface and plugs in the type of
values and ID that it works with: **Account** and **String**, respectively. MongoRepository 
in turn extends **PagingAndSortingRepository** interface defined in Spring Data Commons.

1. You can define other queries by declaring their method signatures. In this case, add
`findByAccountOwner`, which essentially seeks documents of type **Account** and finds the
documents that match on `accountOwner`.

1. You also have `findByAccountNumberAndRoutingNumber`, which finds account by **accountnumber** and **routingnumber**.

## Testing

Using Junit5 and Spring's TestContext framework to create MongoDB repository Integration tests

```java
    @ExtendWith(SpringExtension.class)
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT, classes = GreenfieldReactiveApplication.class)
    public class AccountRepositoryTest {
    
        @Autowired
        private AccountRepository accountRepository;


        @Test
        public void testSave() {
            //when
            Account account = repository.save(new Account(null, 918345L, 234518L, "alex"));
            
            //then
            assertThat(account.getId()).isNotNull();
        }
    
        @Test
        public void testFindById() {
            //given
            Account account = repository.save(new Account(null, 918345L, 234518L, "alex"));
    
            //when
            Account createdCustomerAccount = repository.findById(account.getId());
    
            //then
            assertThat("alex").isEqualTo(createdCustomerAccount.getAccountOwner());
            assertThat(234518L).isEqualTo(createdCustomerAccount.getRoutingNumber());
            assertThat(createdCustomerAccount.getId()).isNotNull();
        }

         @Test
         public void testFindByAccountNumberAndRoutingNumber() {
            //given
            Account account = repository.save(new Account(null, 918345L, 234518L, "alex"));
      
            //when
            Account createdCustomerAccount = repository.findByAccountNumberAndRoutingNumber(account.getAccountNumber(), account.getRoutingNumber);
      
            //then
            assertThat("alex").isEqualTo(createdCustomerAccount.getAccountOwner());
            assertThat(234518L).isEqualTo(createdCustomerAccount.getRoutingNumber());
            assertThat(createdCustomerAccount.getId()).isNotNull();
         }
    }
```
---
## NOTE
