## PreCreation Checkup (Team Lead)

Before a developer creates a new application, a couple of 3rd party services need to be setup.

* **Bitbucket repository** - Create a repository for the project with template: 'web_{app_name}'
* **Bugsnag** - Create a new project on bugsnag and add developers to it
* **Semaphore** - Create a new project, link it to bitbucket, integrate it with slack
* **Codeclimate** - Create a new project, link it to bitbucket, integrate it with semaphore and slack
* **Vault** - Create a new github team, create new policies for that team, server the application will run and a app_id for semaphore

# Generating New Rails Application

99% of our work is based on the Rails framework. We have some standard configurations and gems that are basically in every Rails application and to make our life easier we made a gem called [Initiate](https://github.com/infinum/initiate) to generate new Rails applications with those standards.

Please check Initiate's [README](https://github.com/infinum/initiate#initiate-) for more information.

**Use Initiate after you've done the Architecture Review.**

# PostCreation Checkup (Devops)

* RDS databases for staging and production
* Server nginx conf
* Add DNS records
