> Every dependency in your application has the potential to bloat your app, to destabilize your app, to inject odd behaviour via monkeypatching or buggy native code.
>
> When you are considering adding a dependency to your Rails app, it’s a good idea to do a quick sanity check, in order of preference:
>
> Do I really need this at all? Kill it.
>
> Can I implement the required minimal functionality myself? Own it.
>
> If you need a gem:
>
> Does the gem have a native extension? Look for pure ruby alternatives.
>
> Does the gem transitively pull in a lot of other gems? Look for simpler alternatives.
>
> Gems with native extensions can destabilize your system; they can be the source of mysterious bugs and crashes. Avoid gems which pull in more dependencies than their value warrants.
>
> -- <cite>[Kill your dependencies](https://www.mikeperham.com/2016/02/09/kill-your-dependencies/)</cite>

### Why should you bother with updates?
Convincing clients that we need to spend time regularly updating dependencies is sometimes challenging. The application code already works and it doesn't need changing to keep functioning. Properly updating a dependency takes not only development time (research and execution) but also warrants testing of affected features on all environments. From the client's naive point of view there's no added benefit. It is our job to communicate the following points clearly and take care of overall project health.

- All dependencies have a certain **support life cycle**. If you're using an older version and a new crucial security update is announced you may be forced to do a rapid upgrade of multiple versions to secure your application. Doing that is risky, especially in a time crunch.
-  **Regular smaller dependency upgrades** are more predictable, safer and easier to manage in the context of a sprint. Failing to update dependencies regularly can turn the upgrade process from several bite size chunks which can be done independently to a massive undertaking that takes days and blocks all development. Estimating these big upgrades is always hard and we usually underestimate it by a factor of 3 or 4.
- Dependency upgrades usually introduce **new features, bug fixes and performance enchancements**. With widely used dependencies you're usually not the first person to run into a problem or see room for improvement. In almost all cases somebody has already reported the problem you're having, notified the maintainer and if you're lucky even fixed the issue. The maintainers and community are doing the hard work for free, all you need to do is perform the upgrade and reap the benefits.

## Identifying outdated dependencies
The Ruby ecosystem is quite large and active, so being up to date on all your dependency changes can be a challenge. Manual options like checking the [Ruby security mailing lists](https://groups.google.com/g/ruby-security-ann) and Github repositiories have their place, but they require developers to put in extra time and effort. Luckily the community has developed some great automated tools to help you ease the load.
### Bundle outdated
  Bundler [already includes](https://bundler.io/man/bundle-outdated.1.html) a handy tool which can be used to get a list of outdated gems in your project.

  ```
    Outdated gems included in the bundle:
      * aws-partitions (newest 1.367.0, installed 1.338.0)
      * blueprinter (newest 0.25.1, installed 0.25.0) in groups "default"
      * brakeman (newest 4.9.1, installed 4.9.0) in groups "development"
  ```

  You should have a periodic reminder in your calendar to run `bundle outdated` and triage the result. You can't always be on the edge, but you should at least be aware how far behind you are. This will help you estimate the time needed to update your dependencies and fit them in your schedule.
### [Bundler-audit](https://github.com/rubysec/bundler-audit)
Running `bundle-audit` give you a list of all the dependencies in your project that contain publicly known security vulnerabilities. We mandate this check to be executed before each commit (using [overcommit](https://github.com/sds/overcommit) or [lefthook](https://github.com/Arkweid/lefthook)). This gives developers a clear message -
> Triage and deal with the problem immediately, then continue your work.

Once a vulnerability is made public and added to the [Ruby Advisory Database](https://github.com/rubysec/ruby-advisory-db) you don't have much time to spare. By the time that you get the warning the vulnerability is probably already being exploited in the wild. Arrange with your project manager to get this sorted **immediately**.

### [Dependabot](https://docs.github.com/en/github/administering-a-repository/about-github-dependabot-version-updates)
Can you imagine having a co-worker who would check your dependencies for updates, summarise the changelogs and open a PR for each new one each week? You're in luck, Dependabot does exactly that.

To trigger Dependabot you will have to add a file named `dependabot.yml` to the `.github` directory, located in the root of the project. The configuration should loosely follow the following format

```
version: 2
updates:
  - package-ecosystem: "bundler"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 2

  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 2
```

We recommend that you adjust the limit of weekly pull requests based on the circumstance of your project. Dependencies get updated rapidly, and you won't always have the time sift through all the updates in the same week.

## Dependency upgrades

Dependency upgrades should be treated no differently than a feature request. They should be planned, estimated, implemented, reviewed, deployed to staging, checked by the QA team, and then finally deployed to production and monitored.

You will seldom understand all the inner workings of your downstream dependencies, so you should in fact take even more care than usual when upgrading dependencies.

The key to reducing dependency upgrade risk is making small steps. You need to understand what's being changed and how it's going to affect your project.
>  It may take a bit more planning to pull off, but by finding a way to avoid disrupting the rest of the business, housekeeping activities like keeping your dependencies up-to-date can be done over the course of everyday development—hopefully helping the team avoid ever again falling several years behind on a major dependency like Rails.
>
> -- <cite>[3 keys to upgrading Rails](https://blog.testdouble.com/posts/2019-09-03-3-keys-to-upgrading-rails/)</cite>

The following steps can serve as a guide to help you on your path:

1. Check the dependency policy regarding versioning. Is the dependency using [Semantic versioning](https://semver.org)? What version number changes include breaking API changes?
2. Survey the changelog for breaking changes. It is usually located in the `CHANGELOG.md` file, placed at the root level of the source code. **Always** go through all the intermediate steps between the current and future dependency version tag. You probably won't be familiar with all the phrases and terminology, so take your time. Be especially careful when the major version is bumped.
3. When in doubt, look at the source code diffs. Commit messages and associated PRs usually tell most of the story, but you may still have to dig into the commit content sometimes.
4. Check for any open issues related to the new version.
5. Make an overview of all the places in the codebase that are affected by the dependency. Note the testing steps in the associated task.
6. Apply the required code changes and bump the dependency version (make sure to use the **--conservative** bundler flag to only update the necessary dependencies)
7. Run tests and verify the behaviour locally.
8. Identify & remedy [deprecation warnings](https://www.fastruby.io/blog/rails/upgrades/deprecation-warnings-rails-guide.html).
9. Deploy to a live environment and have one of your colleagues check the behaviour. In most cases it would be too expensive to run a full project smoke test, so focus on the areas that are most likely to be affected by the dependency.
10. If all goes well then deploy to production and **monitor** for regressions.

