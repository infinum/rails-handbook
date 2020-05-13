## Incentive

All apps use some secret keys, access keys, passwords and other configuration that should be hidden from the world. Until now we put those secrets directly in git repositories because all of our repositories were private and no one had access to them. This is a *huge* security issue and something that needs to be rectified. Here is good [read](https://www.humankode.com/security/how-a-bug-in-visual-studio-2015-exposed-my-source-code-on-github-and-cost-me-6500-in-a-few-hours) of what might happen if we don't.

To battle that issue we are storing those secrets in [vault](https://vaultproject.io). Vault, among other things, is a secure secrets storage which stores encrypted data in the backend of your choosing. So even if someone brakes the database, he would have access to only encrypted data.  

To make things easier we will write all our secrets to one file and then store the whole file to vault. For that job we are using [figaro](https://github.com/laserlemon/figaro).

For easier assignment of policies to different users, name of secret key in vault will be `rails/#{git_repo_name}/#{environment}`.


TL;DR:  

* Move all sensitive data to a file that is used by figaro, add that file to `.gitignore`.  
* Store the content of that file in vault to `rails/#{git_repo_name}/#{environment}`  
* New developer gets the development secrets from vault  
* Servers get staging/production secrets from vault  

## Installation

First install `secrets_cli`

    $ gem install secrets_cli

The gem has a dependency on `vault-binaries` and it will install `vault` as well.

## Configuration

Create a new token on [https://github.com/settings/tokens](https://github.com/settings/tokens) with `read:org` permissions.

Create 3 new environment variables:

    export VAULT_ADDR=https://vault.infinum.co:8200
    export VAULT_AUTH_METHOD=github
    export VAULT_AUTH_TOKEN={your_github_token}

Don't forget to restart your `exec $SHELL`

## Policies
**Or how to test your configuration is correct?**

    $ secrets policies
    [rails, team-leaders, web-team, default]

This command will return all policies associated with your auth method and token. Policies are permissions your token has with vault. If you do not have the needed policies please contact your friendly devops or team lead.

## Moving secrets to figaro's config/application.yml file

Add figaro gem to gemfile.  

    gem 'figaro'

Move all secrets from secret.yml, database.yml, and other places to config/application.yml file. Please read [figaro documentation](https://github.com/laserlemon/figaro) on usage.

Check that your app is up and running with new configuration before continuing.

Separate you staging and production secrets into separate files. [See examples](https://gist.github.com/d4be4st/688cf928d5571cd34a24d1e8c7771466)

## Initializing secrets

    $ secrets init
    Which secrets gem are you using? Figaro (config/application.yml)
    What will the secrets storage key be? rails/web_tvornica_snova/
    Written in .secrets:
    ---
    :secrets_file: config/application.yml
    :secrets_storage_key: rails/web_tvornica_snova/

Command will guide you though setting up `.secrets` config file.

**Secrets Repo Naming Convention**

Value of projects `secrets_storage_key` will be `rails/#{git_repo_name}/`.  
Example for tvornica: Bitbucket repo is `https://bitbucket.org/infinum_hr/web_tvornica_snova` so `secrets_storage_key` will be `rails/web_tvornica_snova`.  
Full value for a projects storage key is `rails/#{git_repo_name}/#{environment}` so we can write different configurations for different environments.  
This is an agreed upon naming convention so we do not lose our secrets as vault does not have a way to list all stored secrets.

## Reading, Pushing and Pulling

Now you are all set up to push and pull secrets.

    $ secrets read
    $ secrets push
    $ secrets pull

Read will only read the secrets repo.  
Push will write the contents of your `secrets_file` file to secrets repo.  
Pull will read the secrets repo and overwrite `secrets_file`.

Default environment for these 3 commands is `development`. If you need to change environment use `-e` or `--environment` parameter.

If you need to push a staging/production secrets, create a new environment file (ie `config/application.staging.yml`), fill it up with secrets and push with:

    $ secrets push -e staging -f config/application.staging.yml

## Deploying with secrets

Add `mina-secrets` to your gemfile.  
Add `require 'mina-secrets'` to the top of `deploy.rb` file.  
Add `config/application.yml` to `shared_paths`:

   set :shared_paths, ['log', 'config/application.yml']


Add `invoke :'secrets:pull'` to your deploy task after `git:clone`:

    task :deploy => :environment do
      deploy do
        invoke :'git:clone'
        invoke :'secrets:pull'
        invoke :'deploy:link_shared_paths'
        ...

## Access

**For developers**

For each project a github team needs to be created, and developers working on a project assigned to that team.
Vault policy needs to be created to allow that team to read/write `rails/{git_repo_name}/*`

**For servers**

Each server will need to have its own vault token with policies permitting it to read on all `rails/{git_repo_name}/*` for all projects on that server.

## Configuring the servers

Install and configure as would a developer.  
Only difference is an env variable:

    VAULT_AUTH_METHOD=app_id
    VAULT_AUTH_APP_ID={vault_app_id}
    VAULT_AUTH_USER_ID={vault_user_id}

## Configuring semaphore

Add these two lines to your project build:

    gem install secrets_cli --no-document
    secrets pull
