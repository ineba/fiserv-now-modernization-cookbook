+++
categories = ["recipes"]
tags = ["application development", "Spring", "Spring Boot", "actuator", "endpoint", "health", "health check"]
summary = "Configure Actuators in microservice"
title = "3. Configure Actuators"
date = 2020-12-09T14:02:27-05:00
weight = 1
+++

## CONTEXT

The Spring Boot Actuator provides production-ready features for our Spring Boot application.
Actuators can be useful for providing diagnostics, information, metrics, and controls on your Spring Boot application running in dev,
production, on-prem, or the cloud. It is highly recommended including and enabling Actuators if you are writing a Spring Boot application. 

## SOLUTION
## Actuator Basics
## Actuators

In essence, Actuator brings production-ready features to our application. It monitors app, gather metrics, determines the state of Database and much more.
The main benefit of this library is that we can get production-grade tools without having to actually implement these features ourselves.
Actuator is mainly used to expose **operational information** about the application: **health**, **metrics**, **info**, **dump**, **env**, etc.
It uses **Http** endpoints or **JMX** beans to expose information.

## Setup:

1. Add below dependency to build.gradle.
 
     ```groovy
        implementation: 'org.springframework.boot:spring-boot-starter-actuator'
     ```

## Configuring Actuators

1. Spring Boot 2.x onwards **Actuator comes with most endpoints disabled**. The only two endpoints available 
    by default are _/health_ and _/info_.
1. By Default, All Actuator endpoints are now placed under _/actuator_ path

## Predefined Endpoints

 1. Please find below details of actuator endpoints.

    | Endpoint        | Details  |
    | :---            |    :----   | 
    | /health | Shows application health information.
    | /info | Shows general information. It might be custom data, build information or details about the latest commit |
    | /env | Shows current environment properties.
    | /configprops | Allows to fetch all @ConfigurationProperties beans.
    | /heapdump | Builds and Returns a heap dump from the JVM used by the application.
    | /metrics | Shows ‘metrics’ information for the current application..
    | /threaddump | Performs a thread dump.
    | /conditions | Shows the conditions that were evaluated on configuration and auto-configuration classes and the reason why they did or did not match.
    | /beans   | Displays a complete list of all the Spring beans in the application.
    | /auditevents | Exposes audit events information for the current application. Requires an **AuditEventRepository** bean.

## Hypermedia for Actuator Endpoints

1. Spring boot adds a discovery endpoint that returns links to all available actuator endpoints. This facilitates
   discovering actuator endpoints and their corresponding URLs.

1. By default, this discovery endpoint is accessible through the _/actuator_ endpoint.

## Enable Http Endpoints

All **web** actuator endpoints can be enabled by adding property in **application.yml** or **application.properties**: 

```yaml
    management:
       endpoints:
         web:
           exposure:
             include: *
```

If you want to enable specific **web** endpoints you can enable them in **application.yml** or **application.properties**:

```yaml
    management:
       endpoints:
         web:
           exposure:
             include: health,info,env,metrics
```

By default, _/health_ endpoint only shows below: 
 ```json
  {
  "status": "UP"
  }   
 ```
you can enable _show-details_ management health property in **application.yml** to provide detailed status of the application:

```yaml
    management:
        endpoint:
          health:
            show-details: always
```

you can also disable all **web** endpoints in **application.yml**:

```yaml
    management:
      endpoints:
        enabled-by-default: false
```
## Enable All JMX Endpoints

Java Management Extensions (JMX) provide a standard mechanism to monitor and manage applications. 
By default, Spring Boot exposes management endpoints as JMX MBeans under the org.springframework.boot domain.
you can customize the **JMX** domain under which endpoints are exposed using below
   
```yaml
    management:
      endpoints:
        jmx:
          domain: com.wellsfargo.cto.eai 
```

All **JMX** actuator endpoints can be enabled by adding property in **application.yml** or **application.properties**:

```yaml
    management:
       endpoints:
         jmx:
           exposure:
             include: *
```

If you want to enable specific **JMX** endpoints you can enable them in **application.yml** or **application.properties**:

```yaml
    management:
       endpoints:
         jmx:
           exposure:
             include: health,info,env,metrics
```

All **JMX** endpoints can be disabled by:

```yaml
   endpoints:
     default:
       jmx:
         enabled: false
```
## Enable Actuators based on Profiles:

1. Spring profiles provide a way to segregate parts of your application configuration and make it only available in
   certain environments. We can utilize spring profiles to enable to disable endpoints based on environments.

1. By using `spring.profiles.active=local` Spring will look for a file **application-local.yml** and will try to load
   configurations. We can specify the management endpoints that we need for development/production by following this approach.  


