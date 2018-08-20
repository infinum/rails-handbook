Our git process is a upgraded version of [Github Flow](https://guides.github.com/introduction/flow/).

![](https://ftp.infinum.co/stjepan_hadjic/git-flow-2.jpg)

Instead of a single `master` branch, we use two branches to record the history of the project. The `master` branch stores code that's currently in production. The `staging` branch serves as an integration branch for features and fixes. It holds data that reflects what's currently on staging.

**The master branch is always deployable.**

As we use [productive](https://productive.io) to manage our tasks we use **feature branches** for fixes and new features. Names of branches are always prefixed with `feature/` together with a feature name, so a branch for adding authentication would be named `feature/authentication`.

Feature branches are **always** branched out of `master`. They are also merged first into `staging` and then into `master` if they are ready for production.

**Never branch out of `staging` branch and never merge `staging` into `master`**

We use pull requests for adding new features - pull requests initiate discussion about your commits. Because they're tightly integrated with the underlying Git repository, anyone can see exactly what changes would be merged if they accept your request. When making a pull request **always** make one for `staging` and one for `master`. The pull request workflow is defined for each project separately.

### Why do we need `staging` branch and why never merge it into `master`

We use `staging` server so our QA team can test our applications in an environment that is as close to production as possible. We can also be working on multiple features in parallel. That fixes and features need to be verified by our QA team and sometimes by the client as well. It can happen that one feature is ready for production while others aren't and are still being woked on. In that case the `staging` branch contains multiple features but only one needs to end up on `master`. That is why we do not branch out of `staging` and do not merge `staging` into `master`.

### Note on workflow during early development:

While the application is still not deployed on a production server, you can ommit the `staging` branch. Once the production server is setup and first deploy is up, create a `staging` branch.

### Other important notes on using Git:

**Commit messages are important**, especially since Git tracks your changes and then displays them as commits once they're pushed to the server. By writing clear commit messages, you can make it easier for other people to follow along and provide feedback. Read how to make proper commit messages [here](http://chris.beams.io/posts/git-commit/).

**Make small commits, following the Single Responsibility Principle in git.** This makes commits easier to review when doing pull requests and it's easier to see what's going on when something goes wrong.

**Don't use `git add .`** Review what you're adding to your repo - this is the #1 cause of adding unwanted changes.
