Rails is probably one of your biggest project dependencies. While it may look like a big blob of magic, Rails is in essence a composition of multiple opinionated ([omakase](https://dhh.dk/2012/rails-is-omakase.html)) frameworks. Each of the individual functionalities (object-relation mapping, view templating, email sending, ...) is handled by its own gem:

`actioncable, actionmailbox, actionmailer, actionpack, actiontext, actionview, activejob, activemodel, activerecord, activestorage, activesupport`

Upgrading Rails is thus much more [challenging](https://engineering.shopify.com/blogs/engineering/upgrading-shopify-to-rails-5-0) and [tends to hurt](https://blog.testdouble.com/posts/2019-09-03-3-keys-to-upgrading-rails) because of the big surface area. You're essentially upgrading 11 gems at one time, which are themselves dependencies to a lot of your project's dependencies.


### Upgrade strategy
Planning a Rails upgrade strategy is trivial on well maintained projects where we're only one minor/major Rails version behind, but there are projects where we're severly behind and need to set a precise plan of action. The steps below can guide you in both of those cases and should be formally noted as part of the upgrade task.

#### Assessing the current state of the project
The prerequisite for creating a roadmap is knowing your current location. You should make note of these properties before commencing a Rails upgrade cycle:

- Ruby version
  - how many minor/major versions away from the latest version is it
  - is it [EOL](https://endoflife.date/ruby)
- Rails version
  - how many minor/major versions away from the latest version is it
  - is it [EOL](https://endoflife.date/rails)
- test suite confidence
  - number of tests
  - code coverage
  - types of tests
  - do tests pass locally
- deployment pipeline
  - is it functional
  - do we have access to it
- dependency age
  - how many dependencies does the project have
  - when were they last updated
  - how far away from the latest version are they
- context
  - do we have anyone from the client side available to answer basic questions
  - do we have anyone with in-depth knowledge of the project available internally
  - does the QA team have a set of test cases or someone with in-depth knowledge of the project to perform testing
  - do we have established deployment slots (e.g. scheduled, manually approved)

#### Determining upgrade steps
You should **always** update one minor version of Rails at a time. Each new minor version increment can introduce **important deprecation warnings** to warn you of changes in the next minor/major release. Skipping minor versions can lead to unexpected, hard to diagnose errors. It may seem like you're going slower one minor version at a time, but you'll be faster (and safer) in the long run. Minimizing the complexity and risk of code changes is always the better option.

Our goal is to follow the [Ruby](https://www.ruby-lang.org/en/downloads/releases/) and [Rails](https://rubyonrails.org/category/releases) release cycles closely and be as aligned with new releases as we can be.

Writing out the individual upgrade steps will enable you to better visualize the scope of work and determine the order of execution. The upgrades generally follow a predetermined cycle for each Rails upgrade

- upgrade Ruby [if the next version of Rails requires it](https://www.fastruby.io/blog/ruby/rails/versions/compatibility-table.html)
- [upgrade gems so they support the next version of Rails](#1-upgrading-gems-for-rails-x-support)
- [upgrade Rails to the next minor version](#2-rails-x-upgrade)
- [enable new Rails defaults](#3-enabling-rails-x-defaults)

Here is an example of a roadmap for a very outdated project where we want to upgrade Ruby and Rails from 5.1.7 to 7.0.4 and Ruby from 2.5.5 to 3.1.3.

```
- Rails 5.1.7 -> Rails 5.2.8.1
  - lots of point releases with CVEs
- Upgrading gems for Rails 6 support
- Rails 5.2.8.1 -> Rails 6.0.6
  - add zeitwerk autoloader support
- Rails 6.0.6 -> Rails 6.1.7
- Rails 6.1 enable defaults
- Ruby 2.5.5 -> Ruby 2.6.9
- Ruby 2.6.9 -> Ruby 2.7.7
  - 2.7.6 introduces 3.0 deprecation warnings which are disabled by default
  - 2.7.x required by Rails 7
- Upgrading gems for Rails 7 support
- Rails 6.1.7 -> Rails 7.0.4
- Rails 7.0 enable defaults
- Fixing Ruby 3 deprecation warnings
  - updating gems
  - updating application code
- Ruby 2.7.7 -> 3.0.5
  - possible Passenger issues due to bundled gems
- Ruby 3.0.5 -> 3.1.3
  - possible Passenger issues due to bundled gems
```

Each top level item should have **its own task** (with specific acceptance criteria, estimate and test steps for QA) and **should be deployed individually**.

##### Priorities
If a project is in bad shape, you may be required to combine upgrade efforts with patching security vulnerabilities. The budget and technical constraints might vary by project, but in general you should prioritize the high-priority low-hanging security patches first.

1. High priority CVEs for dependencies not related to Rails
2. High priority Brakeman warnings not related to Rails
3. Rails upgrades
4. Ruby upgrades
5. CVE leftovers
5. Brakeman leftovers

#### Estimations
Estimating upgrades is tough work due to the Rails gravitational pull. Auxiliary gems are tightly coupled to it and so is our application. We can rely on our gut in some cases, but that can be easily thrown off by a seemingly simple one line changelog notice or an outdated auxiliary dependency that will require major application level changes before we can upgrade it.

Splitting the work into phases might help us visualize the scope of work a bit better, but we should also perform a check of potential challenges **in the estimation phase**.

- assess the [current state of the project](#assessing-the-current-state-of-the-project)
- prepare a [roadmap](#upgrade-strategy) and estimate each phase individually
- find exactly which gems [explicitly block the Rails upgrade](#explicit-dependency-rails-support) and estimate their upgrades individually
- find exactly which gems [implicitly block the Rails upgrade](#implicit-dependency-rails-support) and estimate their upgrades individually
- check the [Rails changelogs](#2-rails-x-upgrade) ahead of time and triage which breaking changes are likely to affect the estimate (e.g. the introduction of Zeitwerk autoloading might require you to rename modules and classes)
- check in with the rest of the Rails team and ask very specific questions
  - has anyone performed **this exact** upgrade?
  - what was the initial estimation and actual time spent on the upgrade?
  - was the work deployed in phases?
  - who performed the testing and to what extent?
  - was the project similar in size or requirements (big/small, internal/client)?
  - did we discover any gotchas?
- do a trial upgrade of the Gemfile if possible and note the count of deprecation warnings or application errors when running specs
- check the dependency project health, projects which haven't been updated for a long time will likely require more work
- sync with the project team to determine the amount of manual testing you want to do in each specific phase (e.g. testing yourself, small/complete smoke tests by the QA team). Communicate separate development and QA testing estimates
- sync with the devops team to provide a Ruby installation estimate if applicable

Projects in bad health will require extra care in the estimation phase. The following factors may severly (e.g. 3x) impact the estimate
- low test coverage
- low internal application functionality knowledge
- being more than 1 major Rails or Ruby version behind
- incomplete or proprietary deployment pipeline
- poor codebase health
- lack of functional specification
- forked dependencies
- unmaintained dependencies (e.g. paperclip)
- the app is not bootable locally
- there is no staging environment

**Please perform a sanity check with a TL/LE for all major Rails upgrades.**

### Upgrade steps
The upgrade process can be split into multiple smaller phases. Splitting the work, reviewing and deploying it individually enables us to:

- introduce smaller code changes,
- observe and review a single change at a time,
- perform smaller, more targeted smoke tests of the application,
- more easily plan multiple smaller amounts of work (usually a a day or so per phase) as opposed to a big (weeks) chunk of work,
- estimate the work with a higher degree of confidence.

The phases noted below should each contain their own task, acceptance criteria and verification steps (e.g. partial or full smoke test). They should be deployed to the production environment **individually** and there should be some grace period between them when required.

All rules have exceptions so please check in with a fellow TL/LE if you won't be following these guidelines due to project specific constraints. Bundling the upgrade steps together does introduce extra risk so we should be explicit and communicate the tradeoffs to stakeholders.

#### 1. Upgrading gems for Rails X support
Rails has a strong gravitational pull towards the general gem ecosystem, so it is very likely that most Rails upgrades will require some auxiliary dependencies to be upgraded as well. Compiling a list of gems that need to be upgraded to support the next Rails version will allow you to split the upgrade into multiple parts and deploy smaller, safer changes.

Gems declare their dependency version constraints in the `gemspec` file, but that is not a solid guarantee that a dependency supports the next version of Rails.

##### Explicit dependency Rails support
All project dependencies and their dependency requirements are listed in the `Gemfile.lock` file. Some gems constrict their dependency requirements quite conservatively while others keep an open mind. Relaxing these dependencies (usually by upgrading them) is the first step towards enabling you to upgrade to the next Rails version.

In the following case we present a situation where a project is using Rails 6.1, but is blocked by one of the dependencies (`delayed_job`) to versions betwen 3.0 and 6.0.x.

```
delayed_job (4.1.8)
  activesupport (>= 3.0, < 6.1)
```

An upgrade of the Rails gem (`bundle update rails --conservative`) will fail in this case.

Finding all of the gems that explicitly require upgrades will likely require a repetitive process and some of your time.
- running `bundle update rails --conservative`
- checking the output `Bundler attempted to update rails but its version stayed the same`
- checking `Gemfile.lock` and bundler output for dependencies which might be blocking the upgrade

```
delayed_job (4.1.8)
  activesupport (>= 3.0, < 6.1)
delayed_job_active_record (4.1.4)
  activerecord (>= 3.0, < 6.1)
  delayed_job (>= 3.0, < 5)
audited (4.9.0)
  activerecord (>= 4.2, < 6.1)
```

- adding the dependency to the upgrade list and retrying the upgrade `bundle update rails --conservative delayed_job delayed_job_active_record audited`

Once the `bundle update` command passes successfully you'll have a list of the exact **explicit** dependencies which are blocking the Rails upgrades.

##### Implicit dependency Rails support
Some dependencies might have very open constraints in terms of supported Rails versions.

```
bullet (6.1.5)
  activesupport (>= 3.0.0)
  uniform_notifier (~> 1.11)
```

These requirements would allow you to upgrade the Rails gem from version 6 to 7, but that does not guarantee that the selected gem and its functionalities are compatible with the new Rails version.

In these types of cases the [gem changelog](https://github.com/flyerhzm/bullet/blob/master/CHANGELOG.md) is your best source of information as it usually indicates exactly when support for a new Rails version was added.

>#### 7.0.0 (12/18/2021)
> * Support **rails 7**
> * Fix Mongoid 7 view iteration
>* Move CI from Travis to Github Actions
>#### 6.1.5 (08/16/2021)
>* Rename whitelist to safelist
>* Fix onload called twice
>* Support Rack::Files::Iterator responses
>* Ensure HABTM associations are not incorrectly labeled n+1

In case the changelog is empty, you still have some options:

- check the gem readme for mentions of compatibility
- chech the gem (open and closed) issues for mentions of compatiblity
- check with the Rails team if any projects with that dependency have already upgraded Rails
- check the commit messages and diffs.

Going through changelogs of all dependencies is painstaking work so focus on the core set of your dependencies (e.g. authentication, email sending, background jobs) first. The test suite and QA process is likely to suss out any remaining incompatibilities, but we should at least do our due diligence on the most important dependencies.

##### Gem upgrades
Once you've identified the suite of gems that are incompatible with the Rails upgrade, you should update them and deploy them to production **before** upgrading Rails.

In some cases dependency upgrades can be grouped together to reduce the amount of QA effort and overhead. We advocate for this with tightly coupled gems (e.g. devise and devise_invitable) since they support the same functionality and can be tested together or development dependencies (e.g. puma, rspec, factory_bot) since they don't require a formal QA step.

#### 2. Rails X upgrade
Your primary source of information for minor and major upgrades should be the [Rails upgrade docs](https://guides.rubyonrails.org/upgrading_ruby_on_rails.html). **Read about the relevant changes once, then read again.** An ounce of prevention is worth a pound of cure. Don't start playing error whack-a-mole without fully understanding the framework changes introduced by a new version.

Minor version upgrades have their changelogs linked on the [Rails release page](https://rubyonrails.org/2022/9/9/Rails-7-0-4-6-1-7-6-0-6-have-been-released).

There are two levels of changelogs to go through:

- changelogs for each individual Rails gem (e.g. [ActiveRecord](https://github.com/rails/rails/blob/v6.0.6/activerecord/CHANGELOG.md))
- [condensed changelog summary](https://github.com/rails/rails/releases/tag/v6.0.6) for that specific Rails version.

If you need to upgrade through multiple patch versions (e.g. Rails 6.0.1 to Rails 6.0.6) you should [compare the tags](https://github.com/rails/rails/compare/v6.0.1...v6.0.6) to get a look at all the changelogs. Sometimes even patch versions might introduce breaking changes to [patch security vulnerabilities](https://github.com/advisories/GHSA-3hhc-qp5v-9p2j).

##### Gemfile changes
The Gemfile contains your Rails version requirement (e.g. `gem rails, '~> 6.0.3'`), but the exact version of you're using is noted in `Gemfile.lock`. If the requirement is too restrictive, then you will need to relax it (e.g. `gem rails, '~> 6.0'`) before running the bundle update command

```
bundler update rails --conservative
```

#####  Configuration file changes
Updating Rails configuration files is sometimes tricky due to the amount of configuration file changes. The interactive update tool `rails app:update` is the best resource to guide you through the process with well established options like overwrite, show diff and run merge tools. While this process will run smoothly for vanilla apps, you will run into merge conflicts with apps that have a lot of custom edits in the configuration files. Approach each file change separately and make sure you do this step right. You can use [Rails diff](http://railsdiff.org) to get a better (cleaner) view of the before and after state of the configuration files. If you wish to make your next upgrade easier, then make sure to add custom configuration options at the end of the environment file (e.g. `config/environments/production.rb`) to reduce the chance of merge conflicts.

The update command may also generate migrations (e.g. `activestorage`) and a new framework defaults file (`config/initializers/new_framework_defaults_x_y.rb`) which you will tackle [later](#3-enabling-rails-x-defaults).

##### Deprecation warnings
Running the test suite at this point is likely to bring some new deprecation warnings to the surface. Addressing these deprecation warnings is crucial since they will start raising runtime errors in the next release cycle.

#### 3. Enabling Rails X defaults
Most Rails upgrades introduce some amount of breaking changes. The purpose of the `new_framework_defaults_x_y.rb` file is to allow you to **opt-in** to these changes **at your own pace**, separating the upgrade of Rails and introduction of breaking behaviour.

The defaults file should be carefully triaged. You should be able to answer the following set of question before enabling each of the options:

- how does the behavior change look in practice?
- which parts of the application code will this affect?
- how would we verify if this change will cause us an issue?
- what is the estimated effort of changing the application level code to comply with this option?

If the risk or effort of an individual option is deemed too high, then you should tackle it separately from the rest of the options. Most of these low-hanging-fruit defaults can be packaged and deployed to production together, with risky ones being tackled in separate tasks since they usually require some application level changes and a more focused QA effort.

Once **all** of the defaults have been uncommented in the `new_framework_defaults_x_y.rb` file, you can change the `config.load_defaults` directive in `application.rb` to match your Rails version. At this point the framework defaults file is redundant and can be deleted.

##### Application level backwards compatibility with previous versions of Rails
In the following example Rails 7 changed the default digest class and introduced a breaking change.

```
# Change the digest class for the key generators to `OpenSSL::Digest::SHA256`.
# Changing this default means invalidate all encrypted messages generated by
# your application and, all the encrypted cookies. Only change this after you
# rotated all the messages using the key rotator.
#
# See upgrading guide for more information on how to build a rotator.
# https://guides.rubyonrails.org/v7.0/upgrading_ruby_on_rails.html
# Rails.application.config.active_support.key_generator_hash_digest_class = OpenSSL::Digest::SHA256
```

Encrypted cookies created by earlier (e.g. 6.1) Rails versions would immediately become invalid once the upgrade is be deployed if the option was enabled by default. In this case the grace period between deploying Rails 7 and enabling this setting allows us to implement the cookie rotator in application code safely. We can turn this option on after some time (e.g. a few days) when we're sure that all the users' cookies have been safely migrated on the production environment.

##### Framework level backwards compatibility with previous versions of Rails

In the following example Rails 6.1 introduced a change in how CSRF tokens are encoded.

```
 # Generate CSRF tokens that are encoded in URL-safe Base64.
 #
 # This change is not backwards compatible with earlier Rails versions.
 # It's best enabled when your entire app is migrated and stable on 6.1.
 # Rails.application.config.action_controller.urlsafe_csrf_tokens = true
```
The defaults file explicitly states that previous versions of Rails (e.g. 6.0) are not compatible with this change. If you enabled this option, deployed the change to the production environment, let user traffic to it (generating new CRSF tokens) and for any reason had to revert to the previous version of Rails, you would end up with a lot of errors. The new CSRF tokens generated by Rails 6.1 and still stored on the client side would not be valid when checked by Rails 6.0 so users would probably be unable to save any existing forms they might have had open, potentially even losing data.

You will not hit any such issues if you follow the explicit recomendation of only enabling this option once the new version of Rails has been stable in production for a while.
