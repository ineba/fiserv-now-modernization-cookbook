# Fiserv NOW - Application Modernization Cookbook

This Cookbook contains recipes to help developers modernize applications using cloud native development practices.

This Cookbook was created by Solution Architects from VMware Tanzu (Pivotal) Labs and developers from Fiserv during the **Fiserv NOW Modernization** consulting engagement, and deployed to Cloud (PCF) as a static-web application.


## How to add new recipe to this cookbook

## How to update recipe in this cookbook

1. Clone this cookbook project from GitHub
2. Create a project in IntelliJ IDE by importing the cookbook project directory
3. To Update an existing recipe:
   -  `cd content\recipes`  or `cd content\best-practices`
   - select/edit the appropriate recipe markdown file (.md extension) 
4. cd to the root directory i.e. cookbook project directory
5. Start Hugo locally: `localserver` (Mac or Linux) or `localserver.bat` (Windows)
6. Verify the recipe changes in cookbook at http://localhost:1313


## How to deploy this cookbook to cloud

1. clone this cookbook project from GitHub
2. `cd app-mod-cookbook-starter`
3. Publish cookbook: `publish`  (Mac or Linux) or `publish.bat` (windows)
4. Using `cf cli` login to the appropriate Cloud Foundry org/space
5. Deploy the changes to Cloud Foundry
   - `cd public`
   - `cf push`
6. Verify the latest recipes changes appear in the cookbook deployed in Cloud Foundry 
