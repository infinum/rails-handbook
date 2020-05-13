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
  other communication channels. This is useful because all members of your
  team get notified if a build or deployment fails.

## Mina

[Mina](https://github.com/mina-deploy/mina) is the deploy tool we're currently
using. It's being maintained by our very own [@d4be4st](https://github.com/d4be4st/), so if you run into any issues with it, you can always ping him.

Since Mina shouldn't be used directly for deployment, we won't go into extreme lengths about it in this article. You can read more about the deployment process with Mina in the README.md of the Mina repository.

## Semaphore

[Semaphore](http://www.semaphoreci.com) is the build service we're currently
using. Semaphore supports notifications through instant messaging software,
emails, and other methods. It also supports automated deployments, which have to
be configured separately.

Semaphore uses Mina for deployment in the background, and you can check other
projects for details on how to set up deployment.

New projects on Semaphore can be created by admin users only so, if you need to
create a new project, be sure to contact your team lead.

## Git and continuous delivery

Using continuous delivery for deploying our applications requires some care when
handling Git branches.

* The master branch is the production branch, while the develop branch is the
  staging branch.
* Changes should never be committed or pushed to the master branch directly—they should always be merged into master from the develop branch instead.
* All changes should be made in a separate branch, created from the develop
  branch.
* When changes are completed and reviewed, they should be merged into the
  develop branch and deployed to the staging server.
* Once the changes have passed any additional checks on the staging server, the
  develop branch can be merged into the master branch and deployed to production.

## Sources
* [A Ruby on Rails Continuous Integration Process Using Github, Semaphore,
CodeClimate and
HipChat](https://infinum.co/the-capsized-eight/articles/a-ruby-on-rails-continous-integration-process-using-semaphore-github-codeclimate-and-hipchat), by Jan Varljen




# Mina Workshop

## 1. Mina is just a wrapper around rake

```ruby
task :hello do
  puts 'Hello World'
end

task :question do
  puts 'How are you?'
end

task answer: :question do
  puts 'I am fine, thanks.'
end

namespace :db do
  task :migrate do
    puts 'I am migrating database'
  end
end
```

```bash
rake hello          # invocation
rake hello question # chainging invocation
rake answer         # dependencies
rake db:migrate     # namespaces
```
## 2. Installing mina

```bash
gem install mina
```

or add it to your Gemfile with require: false

## 3. Using mina

### 3.1 initializing

```bash
mina init # creates config/deploy.rb -- Also works with ['Minafile', '.deploy.rb', 'deploy.rb', 'config/deploy.rb', minafile]
```

Least code for mina to work:
```ruby
# config/deploy.rb
require 'mina/default'

task :deploy do
  puts 'deploying'
end
```

### 3.2 variables

```ruby
require 'mina/default'

set :hello, 'world'                        # setting a variable
set :lambda, -> { "#{fetch(:hello)} world" } # setting as lambda

task :write do
  puts fetch(:hello)                       # using variables
  puts fetch(:world)                       # default is nil
  puts fetch(:world, 'Heja')               # can assign a defualt if nonexisting
  puts fetch(:lambda)
end
```

Can override varibles with ENV

```bash
hello=buja mina write
```

### 3.3 commands

```ruby
require 'mina/default'

set :execution_mode, :pretty  # modes: [:pretty, :system, :exec, :printer](https://github.com/mina-deploy/mina/blob/master/docs/how_mina_works.md#execution-modes-runners)

task :deploy do
  run :local do               # backend: [:local, :remote]
    invoke :predeploy         # invoking other tasks
    command 'echo $PATH'      # running shell commands
  end
end

task :predeploy do
  command 'pwd'
end

```
### 3.4 options

```bash
mina -AT # list tasks
mina -d  # display configuration
mina -v  # verbose mode
mina -s  # simulate
```

### 3.5 running on remote

```ruby
require 'mina/default'

set :execution_mode, :pretty
set :domain, 'localhost'

task :deploy do
  run :remote do
    invoke :predeploy
    command 'echo $PATH'
  end
end

task :predeploy do
  command 'pwd'
end
```

## 4. Deploying with mina

```ruby
# config/deploy.rb
require 'mina/deploy'
require 'mina/git'

set :application_name, 'mina_test'
set :domain, 'localhost'
set :deploy_to, '/Users/stef/dev/misc/mina_workshop/examples/4_www'
set :repository, '/Users/stef/dev/misc/mina_workshop/examples/4_deploy'
set :branch, 'master'

task :deploy do
  deploy do
    invoke :'git:clone'

    on :launch do
      command 'pwd'
    end
  end
end
```

```bash
mina setup
mina deploy
tree ../4_www
../4_www
├── current -> /Users/stef/dev/misc/mina_workshop/examples/4_www/releases/2
├── releases
│   ├── 1
│   └── 2
│       └── config
│           └── deploy.rb
├── scm
│   ├── HEAD
│   ├── config
│   ├── description
│   ├── hooks
│   │   ├── applypatch-msg.sample
│   │   ├── commit-msg.sample
│   │   ├── post-update.sample
│   │   ├── pre-applypatch.sample
│   │   ├── pre-commit.sample
│   │   ├── pre-push.sample
│   │   ├── pre-rebase.sample
│   │   ├── pre-receive.sample
│   │   ├── prepare-commit-msg.sample
│   │   └── update.sample
│   ├── info
│   │   └── exclude
│   ├── objects
│   │   ├── 03
│   │   │   └── 2ebb0ede1ede2ef44ca2a0ed2182e554e22548
│   │   ├── 55
│   │   │   └── 3506a9a535dbdd64f3e8016a50a4f2bee39bba
│   │   ├── ab
│   │   │   └── 87456cb4f6725a6b39e64e22bfd84650d353c5
│   │   ├── c8
│   │   │   └── d683fbe9cc6ede0e82a360ab01b85f78f00f2c
│   │   ├── info
│   │   └── pack
│   ├── packed-refs
│   └── refs
│       ├── heads
│       └── tags
├── shared
└── tmp
```

more info at https://github.com/mina-deploy/mina/blob/master/docs/deploying.md
and https://github.com/mina-deploy/mina/blob/master/data/deploy.sh.erb

## 5. Avaliable plugins

* deploy
* git
* bundler
* rails
* managers (rbenv, ry, rvm, chruby)

