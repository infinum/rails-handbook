## Starting a new project

When you're starting a new project, there are a few things you need to do:

#### 1. Open a repository

Ask your team lead or DevOps to open a repository.

#### 2. Fill out the project setup sheet

We've come up with a project setup template in cooperation with the DevOps team. Its purpose is to provide the DevOps with all the details needed for deploying your application to staging and production servers.

It contains general info about your project that your project manager can fill out, as well as more technical information like frameworks and technologies used, the build and deploy setup, third party services that need to be set up, etc.

Ask your PM for a link to the sheet template.

**Fill in the blanks on backend-related stuff.**

If you need any other services that aren't mentioned, **provide as much information as possible**

**The project setup sheet needs to be up to date** with new changes to the project setup and services used. If you're adding some new dependency or service or changing something, please update it in the project setup sheet and notify DevOps to make these changes.


#### 3. Generate a new application

Use the [**default rails template**](https://github.com/infinum/default_rails_template/) to start your new application.

Most of our work is based on the Rails framework. We have some standard configurations and gems that are basically in every Rails application. To make our lives easier, we've made a default template to generate new Rails applications that meet those standards.

Please always update the default rails template if something gets broken or out of date when you're starting a new project.


#### 4. Check for new/better gems

We usually use specific gems for some purposes, e.g. Devise for authentication, Pundit for authorization, etc. That doesn't mean we need to use those gems necessarily.

When adding a gem to your application, check if there's some new and better gem for that purpose.

When checking out a gem, always look for these things:

* is the gem maintained? when was the last commit?
* how many stars does it have?
* what is the ratio between open and closed issues?


#### 5. Fill out the readme file

After setting up a project, fill out the readme file with all the necessary details about the application:

* dependencies & prerequisites
* setup instructions
* development instructions
* deployment instructions
* test instructions
* any other project specific information that someone who takes over a project needs to know


#### 6. Create an ER diagram

Before you start coding, you always need to create an ER diagram for your database. Go through the project specification and list all the entities and their attributes and construct relationships between listed entities.

For creating an ER diagram use:

* [DB diagram](https://dbdiagram.io/)
* [sqlDBM](https://sqldbm.com/)
* [MySQL Workbench](https://dev.mysql.com/downloads/workbench/)

After creating an ER diagram, you need to go through an architecture review. You can read more about it in the [Architecture review chapter](Architecture review)

If you need to create an application or a server architecture diagram use:

* [Gliffy](https://www.gliffy.com/)

If you need to create a sequence diagram use:

* [Web Sequence Diagram](https://www.websequencediagrams.com/)
