Our Git process is an upgraded version of [Github Flow](https://guides.github.com/introduction/flow/).

![](https://ftp.infinum.co/stjepan_hadjic/git-flow-2.jpg)

Instead of a single `main` branch, we use two branches to record the history of the project. The `main` branch stores code that's currently in production. The `staging` branch serves as an integration branch for features and fixes. It holds data that reflects what's currently on staging.

**The main branch must always be deployable.**

As we use [Productive](https://productive.io) to manage our tasks, we use **feature branches** for fixes and new features. Our branch naming convention is {type}/{task-number}{descriptive-task-name}. For example, a branch for adding authentication would be named `feature/123-authentication`.

*Types*

* _feature_ - new feature or an improvement to an existing feature. Goes through the code review process.
* _fix_ - non-critical bugfix, improvement, paying the technical debt. Goes through the code review process.
* _hotfix_ - time sensitive critical bugfix, that should be deployed to production as soon as possible. It is not necessary for it to go through a code review, but it should be revisited at a later stage, and properly fixed or improved.

Feature branches should always be branched out of the `main` branch. They are also merged first into `staging`, and then into the `main` branch if they are ready for production.

**Never branch out of the `staging` branch and never merge `staging` into the `main` branch**

## Why you need the `staging` branch and why you should never merge it into the `main` branch
We use the `staging` server so that our QA team could test our applications in an environment that is as close to production as possible. We can also work on multiple features in parallel. Those fixes and features need to be verified by our QA team and sometimes by the client as well. Sometimes, a feature may be ready for production while others aren't and are still being worked on. In that case, the `staging` branch contains multiple features, but only one needs to end up on the `main` branch. That is why we do not branch out of `staging` and do not merge `staging` into the `main` branch.

## Note on the workflow during early development:
While the application has not been deployed to a production server yet, you can omit the `staging` branch. Once the production server has been set up, and the first deploy is up, create a `staging` branch.

## Other important notes on using Git:
**Commit messages are important**, especially since Git tracks your changes and then displays them as commits once they've been pushed to the server. By writing clear commit messages, you can make it easier for other people to follow and provide feedback.
Commits should have a descriptive subject as well as a quick explanation of the reason for the change in the commit body. This makes it easier to check changes in the code editor as you do not have to find the pull request and open it on GitHub.
Read more about writing proper commit messages [here](https://chris.beams.io/posts/git-commit/).

**Follow the Single Resposibility Principle in git commits.** This makes commits easier to review when making pull requests, and it's easier to notice what's going on when something's wrong.

**Don't use `git add .`** Review what you're adding to your repo — this is the #1 cause of making unwanted changes.
