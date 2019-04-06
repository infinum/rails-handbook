We use the [continuous delivery](https://en.wikipedia.org/wiki/Continuous_delivery)
approach, sometimes also called [continuous integration](https://en.wikipedia.org/wiki/Continuous_integration), to deploy new versions of our websites.

This approach includes an automated build service that runs your automated test
suite, which can usually also deploy the latest changes to the server if the
build passes.

## Why?

* By having an extensive test suite that runs automatically after pushing to the
  remote repository, you can be certain that your new changes didn't break any
  old functionality.

* The build server can deploy the changes automatically if it's set up to do so
  and the build passes successfully, which removes the need to deploy changes
  manually after a push to the remote repository.

* The build server can also send build status notifications to Slack, emails, and
  other communication channels. This is useful since all members of your
  team will be notified if a build or deployment fails.

## Mina

[Mina](https://github.com/mina-deploy/mina) is the deploy tool we're currently
using. It's being maintained by our very own [@d4be4st](https://github.com/d4be4st/), so if you run into any
issues with it, you can always ping him.

Since Mina shouldn't be used directly for deployment, we won't go into
extreme lengths about it in this article. You can read more about the deployment process with
Mina in the README.md of the Mina repository.

## Semaphore

[Semaphore](http://www.semaphoreci.com) is the build service we're currently
using. Semaphore supports notifications through instant messaging software,
emails, and other methods. It also supports automated deployments, which have to
be configured separately.

Semaphore uses Mina for deployment in the background, and you can check other
projects for details on how to set up deployment.

New projects on Semaphore can only be created by admin users, so if you need to
create a new project, be sure to contact your team lead.

## Git and continuous delivery

Using continuous delivery for deploying our applications requires some care when
handling Git branches.

* The master branch is the production branch, while the develop branch is the
  staging branch
* Changes should never be committed or pushed to the master branch directly—
  they should always be merged into master from the develop branch instead
* All changes should be made in a separate branch, created from the develop
  branch
* When changes are completed and reviewed, they should be merged into the
  develop branch and deployed to the staging server
* Once the changes have passed any additional checks on the staging server, the
  develop branch can be merged into the master branch and deployed to production.

## Sources
* [A Ruby on Rails Continuous Integration process using Github, Semaphore,
CodeClimate and
HipChat](https://infinum.co/the-capsized-eight/articles/a-ruby-on-rails-continous-integration-process-using-semaphore-github-codeclimate-and-hipchat), by Jan Varljen
