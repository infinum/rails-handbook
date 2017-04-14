## PreCreation checklist (Devops)

Before a developer creates a new application, a couple of 3rd party services need to be setup.

* **Bitbucket repository** - Create a repository for the project with template: [web-{app_name}](https://github.com/infinum/internal-guides/blob/master/How%20to%20name%20your%20repository.md)
* **Bugsnag** - Create a new project on bugsnag and add developers to it
* **Semaphore**
  - Create a new project, link it to bitbucket, integrate it with slack
  - Create new servers for staging and production to be automatically deployed
* **Codeclimate** - Create a new project, link it to bitbucket, integrate it with semaphore and slack
* **Vault** - Create a new github team, create new policies for that team, server the application will run and a app_id for semaphore

* RDS databases for staging and production
* Server nginx conf
* Add DNS records

## Generating New Rails Application

99% of our work is based on the Rails framework. We have some standard configurations and gems that are basically in every Rails application and to make our life easier we made a [default template](https://github.com/infinum/default_rails_template/) to generate new Rails applications with those standards.

Use `rails new myapp --database=postgresql -T -B -m https://raw.githubusercontent.com/infinum/default_rails_template/master/template.rb` to create a new application.

## First deploy

Immediately after the application is created, your friendly devops will give you all the information you need to set up for the first deploy.
Commit the changes, push them to git and make sure semaphore successfully deploys the application.
