# Fiserv NOW - Application Modernization Cookbook

Cookbook with recipes for developing cloud-ready services using _modern development practices_ and _cloud native principles_.


This cookbook was a collaborative effort between **VMWare Tanzu (Pivotal) Labs** and **Fiserv NOW Team** during the **Fiserv NOW Modernization** engagement in 2022, and can be deployed in cloud as a static-web application.

## How to add new recipe to this cookbook
1. `cd app-mod-cookbook-starter`
2. `hugo new recipes/my-new-recipe.md` 
3. `cd content/recipes`
4. add content to the markdown file using your favorite text editor
5. Don't forget to update the _preamble_ at top of the markdown file using the existing markdown files as reference.

```toml
+++
title = "Configure Actuators"
summary = "Configure Actuators in microservice"
categories = ["recipes"]
tags = ["application development", "Spring", "Spring Boot", "actuator", "endpoint", "health", "health check"]
date = 2020-12-09T14:02:27-05:00
weight = 1
+++
```
-  set `title`,`summary` appropriate to the recipe content and context 
- `tags`: remove unnecessary tags and add tags specific to the recipe content and context. Remember _less is more_ for the tags!
6. `cd app-mod-cookbook-starter`
7. Start Hugo locally: `localserver` (Mac or Linux) or `localserver.bat` (Windows)
6. Verify the recipe appears in cookbook at http://localhost:1313


## How to update recipe in this cookbook

1. Clone this cookbook project from GitHub
2. Create a project in IntelliJ IDE by importing the cookbook project directory
3. To Update an existing recipe:
   -  `cd content/recipes`  or `cd content/best-practices`
   - select/edit the appropriate recipe markdown file (.md extension) 
4. cd to the root directory i.e. cookbook project directory
5. Start Hugo locally: `localserver` (Mac or Linux) or `localserver.bat` (Windows)
6. Verify the recipe changes in cookbook at http://localhost:1313


## How to deploy this cookbook to cloud (_Cloud Foundry_)

1. clone this cookbook project from GitHub
2. `cd app-mod-cookbook-starter`
3. Publish cookbook: `publish`  (Mac or Linux) or `publish.bat` (windows)
4. Using `cf cli` login to the appropriate Cloud Foundry org/space
5. Deploy the changes to Cloud Foundry
   - `cd public`
   - `cf push`
6. Verify the latest recipes changes appear in the cookbook deployed in Cloud Foundry 

### Author
**Prashanth **'PB'** Belathur**    
Program Technical Advisor, VMWare Tanzu (Pivotal) Labs.  
pbelathur@vmware.com
