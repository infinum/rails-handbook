## PreCreation checklist (Devops)

Before a developer creates a new application, a couple of third-party services need to be set up.

* **Bitbucket repository**—Create a repository for the project with template "web-{app-name}"
* **Bugsnag**—Create a new project on Bugsnag and add developers to it
* **Semaphore**
  - Create a new project, link it to Bitbucket, integrate it with Slack
  - Create new servers for staging and production to be automatically deployed
* **Codeclimate**—Create a new project, link it to Bitbucket, integrate it with Semaphore and Slack
* **Vault**—Create a new GitHub team, create new policies for that team, a server the application will run on, and an app_id for Semaphore

* RDS databases for staging and production
* Server nginx conf
* Add DNS records

## Generating new Rails application

99% of our work is based on the Rails framework. We have some standard configurations and gems that are basically in every Rails application. To make our lives easier, we've made a [default template](https://github.com/infinum/default_rails_template/) to generate new Rails applications that meet those standards.

## First deploy

Immediately after the application is created, your friendly devops will give you all the information you need to set up for the first deploy.
Commit the changes, push them to Git, and make sure Semaphore successfully deploys the application.
