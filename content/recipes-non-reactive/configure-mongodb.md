+++
categories = ["recipes"]
tags = ["persistence","mongodb","spring boot"]
summary = "Configure MongoDB datasource in microservice"
title = "4. Configure MongoDB"
date = 2020-12-09T14:02:27-05:00
weight = 3
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
   | uri | mongodb://\<host>:\<port>/\<database name> 
   | database-name | database  name  |
   | username | username (plain text)
   | password | password (encrypted)
 
1. Navigate to the `<microservice>` directory
   
1. Update the _database connection_ and _connection pool requirements_ in `src/main/resources/application.yml`

   ```yml
   application:
           name:
           description:
           id: <wells fargo distributed-id>
           persistence:
              mongodb:
                  host:
                  port:
                  database-name:
                  username:
                  password:
    ```

### Validation

1. Open a command window in the `<microservice-name>` directory

1. Validate the new microservice
   - can be built locally: `gradlew bootJar`
   - runs locally: `gradlew bootRun`

1. Verify microservice health in the browser

   - `http://localhost:8080/actuator/info`
     
   - `http://localhost:8080/actuator/health`  
      the _datasource details_ and _uptime status_ should be displayed in the browser.
   
   - `http://localhost:8080/actuator/beans`  
     search for _mongo_ in the browser and you should see the following
     ```json
      "mongo": {
         "status": "UP",
         "details": {
         "version": "4.2.0"
         }
      },    
     ```

## NOTES
  you can also use properties provided by **Spring** to configure mongodb
  ```yaml
     spring:
        mongodb:
          uri:    
          database:   
          username:
          password:
  ```


