+++
categories = ["recipes"]
tags = ["persistence","hikari","database connection pool","anti patterns"]
summary = "Configure Oracle datasource in microservice"
title = "3. Configure Oracle"
date = 2020-12-09T14:02:27-05:00
weight = 2
+++

## CONTEXT
This recipe is part of a cookbook for developing a modern _cloud ready_ microservice using the **greenfield-app-starter**.   
This recipe deals with configuring Oracle to fulfill persistence needs in the microservice.  
A _Hikari_ datasource bean with connection pooling is created after completing this recipe. 

### Prerequisite

- **STEP 1: CREATE APPLICATION USING THE STARTER** is completed.

## SOLUTION

1. Define **environment variables** for **oracle database connection properties**,
   these are usually defined in uDeploy and **never**   hardcoded in`application.yml`

   | Property      | Details  |
   | :---          |    :----   | 
   | `ORACLE_DB_URL`  |  `jdbc:oracle:thin:@<host>:<port>:<schema>` [thin] OR `jdbc:oracle:oci:@<host>:<port>:<schema>` [oci] |
   | `ORACLE_SCHEMA`     | database schema name  | 
   | `ORACLE_USERNAME` | username  | 
   | `ORACLE_PASSWORD` | encrypted password|
   
1. Tweak (_if necessary_) the default **database connection pool settings** after reviewing the  [**anti-patterns**](https://github.com/pbelathur/spring-boot-performance-analysis).

   | Property        | Description | Starter Default  |
   | :---          |    :----   |  :----   | 
   | `max-pool-size`  | maximum size that the pool is allowed to reach, includes both idle and in-use connections | 10  |
   | `min-idle` | minimum number of idle connections, for efficient performance keep this value at 50% of max-pool-size or less |5 | 
   | `connection-timeout` | the maximum number of milliseconds that a client will wait for a connection from the pool. If this time is exceeded without a connection becoming available, a SQLException will be thrown. Minimum values is 250ms.   | 1000  |
   | `idle-timeout`  | the maximum amount of time a connection is allowed to sit idle in the pool. This setting only applies when `min-idle` < `max-pool-size.` Minimum is 10000ms (10s). | 10000  | 
   | `max-lifetime`    |the maximum lifetime of a connection in the pool. Minimum values allowed is 30000ms (30s) | 1800000 |

   **NOTE**
   - refrain from increasing the `max-pool-size` beyond 15
   - the `connection-timeout`can be increased by small increments for networks with higher than normal latencies. 

1. Navigate to the `<microservice>` directory

1. Review the _database connection_ and _connection pool_ properties in `src/main/resources/application.yml`
   
   ```yml
   application:
     id: <wells fargo distributed-id>
     persistence:
      oracle:
        name: oracle-datasource
        url: ${oracle.db.url}
        username: ${oracle.username}
        password: ${oracle.password}
        schema: ${oracle.schema}
        driver: oracle.jdbc.OracleDriver
        connection-pool:
          max-pool-size: 10
          idle-pool-size: 2
          connection-timeout: 500
          idle-timeout: 800
          max-lifetime: 3000
   ```
- replace `<wells fargo distributed-id>` with the **distributed ID**   
- the `placeholder` property is defined as an **environment variable** in uDeploy.
   - e.g.  environment variable: `ORACLE_DB_URL` corresponds to `${oracle.db.url}`
   - the `${placeholder}` is used for _automatic variable expansion_ during microservice startup.
   
   
  
### Validation
1. Open a command window in the `<microservice-name>` directory

2. Set all the required **environment variables** for the *property placeholder*s in`application.yml`

   ```shell
   set ORACLE_DB_URL=jdbc:oracle:thin:@<host>:<port>:<schema>
   set ORACLE_SCHEMA=schema-name
   set ORACLE_USERNAME=username
   set ORACLE_PASSWORD=encrypted password
   ```

3. Validate the new microservice ...
   
   - can be built locally: `gradlew clean bootJar`
   - runs locally: `java -jar build/libs/<app-name>-<version>.jar`
 
  
4. Verify microservice health in the browser

   - `http://localhost:8080/actuator/info`

   - `http://localhost:8080/actuator/health`  
      the _datasource details_ and _uptime status_ should be displayed     in the browser.
     
   - `http://localhost:8080/actuator/beans`  
     search for _HikariDataSource_ in the browser.
     
## NOTES
- [database connection pool **anti-patterns**](https://github.com/pbelathur/spring-boot-performance-analysis)
- We are using spring data jdbc *NOT* spring data JPA for Oracle;
  use `spring-data-jpa-starter`