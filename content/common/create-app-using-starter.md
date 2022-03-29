+++
categories = ["recipes"]
tags = ["starter","microservice", "barebone microservice"]
summary = "How to create a microservice using the greenfield-app-starter"
title = "1. Create barebone microservice using the starter"
date = 2020-12-09
weight = 10
draft = false
+++

## CONTEXT
This recipe is part of a cookbook for developing a modern _cloud ready_ microservice using the **greenfield-app-starter**.  
Please refer - recipe - [How to Structure - Spring Boot Application!](/best-practices/spring-boot-structure/).  
Upon completion of this recipe, you'll have a _working_  [Spring Boot](https://medium.com/stuff-about-cloud-native-development/my-spring-boot-101-journal-getting-started-5183f68606cb) microservice with enabled actuator (_info,health and metrics_) endpoints.

### Prerequisite

- JDK 8 or higher
- IntelliJ or Eclipse IDE
- GIT client
- Access to **greenfield-app-starter** Github repo

## SOLUTION

1. Collect or record the following details for the new microservice

   | Property        | Description | Example |
   | :---          |    :----   |  :----   |
   | lob  | line of business | **consumer** |
   | business-group | business group within the lob | **lending** |
   | application-group  | application category or grouping | **loan** |
   | microservice-name      | camelCased name  | **AutoLoanCalculator**
   | microservice-version    | in `major.minor.patch` format; [Versioning Basics](https://medium.com/fiverr-engineering/major-minor-patch-a5298e2e1798) | **0.1.0**
   | description    | short phrase describing the purpose of the microservice | **consumer auto loan calculator for period less than 36 months**
   | JDK-version  |Java 8 or above; one of `1.8`, `1.11`, `1.12`, `1.13` or `1.14`| **1.12**
   | project-group  | `com.wellsfargo.<lob>.<business-group>.<application-group>` |  **`com.wellsfargo.consumer.lending.loan`**

1. Clone the **greenfield-app-starter** from Gitlab repo `git clone <repo url>`
![starter app folder structure](/images/resized.jpg)

1. Rename folder: `greenfield-app-starter` to `<microservice-name>`  
   (example: **AutoLoanCalculator**)

1. Update the microservice name in `settings.gradle`

   ```gradle
   rootProject.name = "<microservice-name>"
   ```

   ```gradle
   # EXAMPLE
   
   rootProject.name = "AutoLoanCalculator"
   ```   


1. Update project details in `build.gradle`
   
   ```gradle
   description = "<description>"
   group = "<project-group>"
   version = "<microservice-version>"
   sourceCompatibility = "<JDK-version>"
   ```

    ```gradle
   # EXAMPLE
   
   description = "consumer auto loan calculator for period less than 36 months"
   group = "com.wellsfargo.consumer.lending.loan"
   version = "0.1.0"
   sourceCompatibility = "1.12"
   ```
   
1. Rename _package_ from `com.wellsfargo.cto.eai.starter` to `<project-group>`  
   (example: **`com.wellsfargo.consumer.lending.loan`**)

1. Rename _main application classname_ from `GreenfieldMicroservice` to `<microservice-name>`  
   (example: **`AutoLoanCalculator`**)

1. As an example, the _barebone_ microservice will have the following:
   
   * folder: `AutoLoanCalculator` containing
      * `com.wellsfargo.consumer.lending.loan.AutoLoanCalculator.java`
      * `src/main/resources/application.yml`
      * `build.gradle`
      * `settings.gradle`
   

1. Create the codebase folder/package structure based on the recipe in ***Best Practices***

### Validation

1. Open a command window in the `<microservice-name>` directory

1. Validate the new microservice
   - can be built locally: `gradlew bootJar`
   - runs locally: `gradlew bootRun`

1. Verify microservice health in the browser
   - `http://localhost:8080/actuator/info`
   - `http://localhost:8080/actuator/health`


### Next Step
Follow the **Reactive** or **Non-Reactive** path depending on your microservice needs. 

## NOTES
* [Why use Spring Boot ?](https://medium.com/stuff-about-cloud-native-development/my-spring-boot-101-journal-getting-started-5183f68606cb)
* [How to version software ?](https://semver.org/)