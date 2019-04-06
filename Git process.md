Our Git process is an upgraded version of [Github Flow](https://guides.github.com/introduction/flow/).

![](https://ftp.infinum.co/stjepan_hadjic/git-flow-2.jpg)

Instead of a single `master` branch, we use two branches to record the history of the project. The `master` branch stores code that's currently in production. The `staging` branch serves as an integration branch for features and fixes. It holds data that reflects what's currently on staging.

**The master branch is always deployable.**

As we use [Productive](https://productive.io) to manage our tasks, we use **feature branches** for fixes and new features. Names of branches are always prefixed with `feature/` together with a feature name. For example, a branch for adding authentication would be named `feature/authentication`.

Feature branches are **always** branched out of `master`. They are also merged first into `staging`, and then into `master` if they are ready for production.

**Never branch out of the `staging` branch and never merge `staging` into `master`**

We use pull requests to add new features—pull requests initiate discussion about your commits. As they're tightly integrated with the underlying Git repository, everyone can see exactly what changes will be merged if they accept your request. When making a pull request, **always** make one for `staging` and one for `master`. The pull request workflow is defined separately for each project.

### Why do you need the `staging` branch and why you should never merge it into `master`

We use the `staging` server so that our QA team could test our applications in an environment that is as close to production as possible. We can also work on multiple features in parallel. Those fixes and features need to be verified by our QA team and sometimes by the client as well. Sometimes, a feature may be ready for production while others aren't and are still being worked on. In that case, the `staging` branch contains multiple features, but only one needs to end up on `master`. That is why we do not branch out of `staging` and do not merge `staging` into `master`.

### Note on workflow during early development:

While the application is still not deployed on a production server, you can omit the `staging` branch. Once the production server has been set up, and the first deploy has been up, create a `staging` branch.

### Other important notes on using Git:

**Commit messages are important**, especially since Git tracks your changes and then displays them as commits once they've been pushed to the server. By writing clear commit messages, you can make it easier for other people to follow and provide feedback. Read how to write proper commit messages [here](http://chris.beams.io/posts/git-commit/).

**Make small commits, following the Single Responsibility Principle in Git.** This makes commits easier to review when making pull requests, and it's easier to notice what's going on when something goes wrong.

**Don't use `git add .`** Review what you're adding to your repo—this is the #1 cause of making unwanted changes.
