# Fiserv NOW - Application Modernization Cookbook

This Cookbook contains recipes to help developers modernize applications using cloud native development practices.

This Cookbook was created by VMware Tanzu Labs - Solution Architects and Fiserv developers during the Fiserv NOW Modernization consulting engagement, and, is deployed to PCF as a static-web application.

## How to add a new recipe to this cookbook ?

## How to update recipes in this cookbook ?

1. Clone this cookbook project from GitHub
2. Create a project in IntelliJ IDE by importing the cookbook project directory
3. To Update an existing recipe:
   - `cd content\recipes-non-reactive` or `\recipes-reactive` or `\common` or `\best-practices`
   - select/edit the appropriate recipe markdown file (.md extension) 
4. cd to the root directory i.e. cookbook project directory
5. Start Hugo locally: `localserver` (Mac or Linux) or `localserver.bat` (Windows)
6. Review your latest updates to the recipe(s) in the browser http://localhost:1313
7. Publish your changes: `publish` or `publish.bat` (windows)
8. Deploy the changes to PCF
   - `cd public`
   - `cf push`
9. Verify the latest recipes changes in cookbook hosted in PCF    
