+++
categories = ["recipes"]
tags = ["practices", "spring boot", "microservice", "custom spring bean validation"]
title = "4.Custom Spring Bean Validation"
description = "A guide to use custom validation"
date = 2020-12-09
weight = 20
draft = false
authors = ["Rohan Mukesh"]
+++

## Context

While implementing Spring REST endpoints for Spring boot applications, adding validations (inbuilt/custom) becomes inevitable. For most cases the inbuilt validators provided by JSR 380, also known as Bean Validation 2.0 framework would suffice. Some of the inbuilt validators provided are: @NotNull, @NotEmpty, @NotBlank, @Min, @Max, @Size to name a few. There are still instances where the validation need can’t be taken care of by the inbuilt validators provided by JSR 380 and in such cases we need to write custom validators which takes care of providing custom validation logic to the bean attributes.
    
## Use Case:

Let’s assume a use case wherein we need to validate customer location details with custom validation of fields locationId, countryCode and postCode. These three fields should accept *only numeric strings* (ex: “123")

   ```java
    @Getter
    @Setter
    public class CustomerLocation {
        @NumericString(message = "locationId should be numeric")
        private String locationId;
        @NotBlank(message = "city cannot not be empty")
        private String city;
        @NumericString(message = "countryCode should be numeric")
        private String countryCode;
        @NumericString(message = "postCode should be numeric")
        private String postCode;
     }
   ```
In the above example both inbuilt (@NotBlank) and custom (@NumericString) validators are being used. @NotBlank would ensure that the value passed to city attribute is not blank however @NumericString validator would ensure that the value passed to locationId, countryCode & postCode is *a numeric string*

## Setup:
 
Add below dependency to build.gradle. The latest dependency can be checked [here](https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.hibernate%22%20AND%20a%3A%22hibernate-validator%22)
 ```
    compile group: 'org.hibernate', name: 'hibernate-validator', version: '4.2.0.Final'
 ```
If we're using Spring Boot, then we need below dependency to be added, which will bring in the hibernate-validator dependency also.
 ```
    implementation: 'org.springframework.boot:spring-boot-starter-validation:2.4.1'
 ```

## **Controller:**

Lets see the REST endpoint which validates the incoming request:

   ```java
      @RestController
      @Validated  
      public class ValidatorController {
          @PostMapping(value="/v1/validate", produces = "application/json")
          public ResponseEntity<String> validateCustomerLocation(@ValidCustomerLocation @RequestBody CustomerLocation customerLocation){
              return new ResponseEntity<>(HttpStatus.ACCEPTED);
          }
      }
   ```
Note that we have to add Spring’s @Validated annotation to the controller at class level to tell Spring to evaluate the constraint annotations on method parameters.
The @Validated annotation is only evaluated on class level in this case, even though it’s allowed to be used on methods.

## New Custom Annotation:

Creating a custom validator entails us rolling out our own annotation and using it in our model to enforce the validation rules.
ValidCustomerLocation is a custom validator annotation for which the constraints would be validated by CustomerLocationValidator class as shown below:

   ```java
      @Target({ElementType.FIELD, ElementType.PARAMETER})
      @Retention(RetentionPolicy.RUNTIME)
      @Constraint(validatedBy = {CustomerLocationValidator.class})
      @Documented
      public @interface ValidCustomerLocation {
      String message() default "Invalid customer location";
          Class<?>[] groups() default {};
          Class<? extends Payload>[] payload() default {};
      }
   ```
The @Constraint annotation defined in the class is going to validate our field and message() is the error message that is returned to the client.
The additional code is boilerplate code that conforms to Spring standards.

## Custom Validator:

The validation class (CustomerLocationValidator) implements the _ConstraintValidator_ interface and must implement the isValid() method.
It's in this method we will define our validation rules.

   ```java
      public class CustomerLocationValidator implements ConstraintValidator<CustomerLocationConstraint, CustomerLocation> {
    
        @Autowired
        Validator validator;
    
        @Override
        public boolean isValid(com.wellsfargo.cto.eai.customvalidator.model.CustomerLocation customerLocation, ConstraintValidatorContext constraintValidatorContext) {
            Set<ConstraintViolation<com.wellsfargo.cto.eai.customvalidator.model.CustomerLocation>> constraintViolations = validator.validate(customerLocation);
            if (!CollectionUtils.isEmpty(constraintViolations)) {
                constraintValidatorContext.disableDefaultConstraintViolation();
                constraintViolations.forEach(customerLocationConstraintViolation -> constraintValidatorContext
                        .buildConstraintViolationWithTemplate(
                                customerLocationConstraintViolation.getMessageTemplate())
                        .addConstraintViolation());
                return false;
            }
            return true;
        }
    }
   ```
Attributes with @NumericString annotation would be validated by NumericStringValidator class as shown below:

 ```java
    @Target({ElementType.FIELD, ElementType.PARAMETER})
    @Retention(RetentionPolicy.RUNTIME)
    @Constraint(validatedBy = {NumericStringValidator.class})
    @Documented
    public @interface NumericString {
        String message() default "String should be numeric";
    
        Class<?>[] groups() default {};
    
        Class<? extends Payload>[] payload() default {};
    }
 ```
The implementation of NumericStringValidator would override the isValid() method and check if the attributes annotated with @NumericString contains only numerals using the regular expression check:

 ```java
    public class NumericStringValidator implements ConstraintValidator<NumericString, String> {
        @Override
        public boolean isValid(String str, ConstraintValidatorContext constraintValidatorContext) {
            if (str.matches("[0-9]+")) return true;
            return false;
        }
    }
```
## Unit Testing:



## Test:

Clone the codebase, build and run the CustomvalidatorApplication class. The application runs on default port 8080. Once it is up and running perform the below two tests:

### 1. Valid request:

Endpoint URL: localhost:8080/v1/validate
  ```json    
      {
        "locationId":"aa",
        "countryCode":"sns",
        "postCode":"ss"
      }
 ```
**Response**:

Status: HTTP response code 202 Accepted

### 2. Invalid request:

Endpoint URL: localhost:8080/v1/validate

 ```json
     {
      "locationId":"locationId",
      "countryCode":"countryCode",
      "postCode":"postCode"
     }
 ```   

### Response:

Status: HTTP response code 400 Bad Request
 ```json
    {
      "errorCode": "400 BAD_REQUEST",
      "errorMessage": "Validation Errors",
      "errors": [
        {
          "rejectedValue": {
            "locationId": "aa",
            "city": null,
            "countryCode": "sns",
            "postCode": "ss"
          },
          "message": "postCode should be numeric"
        },
        {
          "rejectedValue": {
            "locationId": "aa",
            "city": null,
            "countryCode": "sns",
            "postCode": "ss"
          },
          "message": "city cannot not be empty"
        },
        {
          "rejectedValue": {
            "locationId": "aa",
            "city": null,
            "countryCode": "sns",
            "postCode": "ss"
          },
          "message": "locationId should be numeric"
        },
        {
          "rejectedValue": {
            "locationId": "aa",
            "city": null,
            "countryCode": "sns",
            "postCode": "ss"
          },
          "message": "countryCode should be numeric"
        }
      ]
    }
```
The source code is available at [wf github](https://)