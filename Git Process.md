# Git process

Our git process is a mix of [Gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow) & [Github Flow](https://guides.github.com/introduction/flow/).


Instead of a single `master` branch, we use two branches to record the history of the project. The `master` branch stores code that's currently in production, and the `develop` branch serves as an integration branch for features and holds data that reflects what's currently on staging.

**The master branch is always deployable.**

We use **feature branches** for adding new features - names of feature branches are always prefixed with `feature/`, so a feature branch for adding authentication would be named `feature/authentication`.

Feature branches are always merged first into develop and then into master if they are ready for production.

We use pull requests for adding new features - pull requests initiate discussion about your commits. Because they're tightly integrated with the underlying Git repository, anyone can see exactly what changes would be merged if they accept your request. Pull requests are done for feature branched when they need to be merged to develop. The pull request workflow is defined for each project separately. 

We use **bugfix branches** to quickly patch production releases. This is the only branch that should fork directly off of master. As soon as the fix is complete, it should be merged into both master and develop. Prefix bugfix branches with `bugfix/`.



Other important notes on using Git:

**Commit messages are important**, especially since Git tracks your changes and then displays them as commits once they're pushed to the server. By writing clear commit messages, you can make it easier for other people to follow along and provide feedback. Read how to make proper commit messages [here](http://chris.beams.io/posts/git-commit/).

**Make small commits, following the Single Responsibility Principle in git.** This makes commits easier to review when doing pull requests and it's easier to see what's going on when something goes wrong.

**Don't use `git add .`** Review what you're adding to your repo - this is the #1 cause of adding unwanted changes.
