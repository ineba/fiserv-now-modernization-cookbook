+++
categories = ["recipes"]
tags = ["persistence","mongodb","spring boot"]
summary = "Configure mongodb datasource in microservice"
title = "4. Configure MongoDB"
date = 2020-12-09T14:02:27-05:00
weight = 1

+++

## CONTEXT
This is the second recipe in the series, for developing a modern _cloud ready_ microservice using the **greenfield-app-starter**.  
This recipe deals with configuring persistence in the microservice.  

### Prerequisite

- **STEP 1: CREATE APPLICATION USING THE STARTER** is completed.

## SOLUTION

1. Determine and record the following **mongodb database connection details** 

   | Property        | Details  |
      | :---            |    :----   | 
   | uri | \<host>:\<port>
   | database-name | database  name  |
 
1. Navigate to the `<microservice>` directory
   
1. Update the _database connection_ in `src/main/resources/application.yml`

   ```yml
      mongodb:
        host:
        database-name:
    ```
1. Create a new class `MongoProperties` . This class will load the MongoDB properties from `src/main/resources/application.yml`

 ```java
   @ConfigurationProperties("mongo")
   @Component
   @Data
   public class MongoProperties {
      private List<String> hosts;
      private String database;
 }
 ```

1. Create new class `MongoDBConfiguration`. This class will extend `AbstractReactiveMongoConfiguration`.
   The purpose of this class is to load MongoDB connection url and database and configure `ReactiveMongoDBTemplate`
   
 ```java
@Configuration
@AllArgsConstructor
public class MongoDBConfiguration extends AbstractReactiveMongoConfiguration {

    private static final String CONNECTION_URL = "mongodb://%s/%s";

    private final MongoProperties mongoProperties;

    @Override
    protected String getDatabaseName() {
        return mongoProperties.getDatabase();
    }

    @Override
    public MongoClient reactiveMongoClient() {
        return MongoClients.create(getMongoDBConnectionUrl());
    }

    @Override
    public Collection getMappingBasePackages() {
        return Collections.singleton("com.wellsfargo.reactive.starter.greenfieldreactiveapplicationstarter");
    }

    private String getMongoDBConnectionUrl() {
        return String.format(CONNECTION_URL, mongoProperties.getHosts().get(0), mongoProperties.getDatabase());
    }

    @Bean
    public ReactiveMongoTemplate reactiveMongoTemplate() {
        return new ReactiveMongoTemplate(reactiveMongoClient(), getDatabaseName());
    }
}
```

### Validation

1. Open a command window in the `<microservice-name>` directory


1. Validate the new microservice can be built locally: `gradlew bootJar`


1. Validate the new microservice runs locally: `gradlew bootRun`


1. Verify microservice health and info in the browser

   - `http://localhost:8080/actuator/info`
     
   - `http://localhost:8080/actuator/health`  
      the _datasource details_ and _uptime status_ should be displayed in the browser.
   
   - `http://localhost:8080/actuator/beans`  
     search for _mongo_ in the browser and you should see below
     ```json
      "mongo": {
         "status": "UP",
         "details": {
         "version": "4.2.0"
         }
      },    
     ```

The code snippets can be found in [Wells Fargo GitHub](https://)