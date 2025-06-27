After a while you'll get notifications about comments on your pull requests from your reviewers. Oftentimes you'll need to change something before the branch is ready.

## Solving change requests
Change requests should be applied in separate _fixup_ or _squash_ commits. Rebasing the branch during an ongoing review is not appreciated unless there is a good reason for it, like pulling in some new and necessary changes from the `main` branch, because it makes it harder for the reviewers to know what the new changes are and what they have already reviewed.

These commits should be integrated into `staging` as well when they are done.

## Integration Methodology

### Staging
Once the PR is opened, the branch can be integrated into `staging`, unless it contains considerable
logic in the migrations, in which case the reviewers should prioritize reviewing the migrations,
and giving a thumbs up before integration.

We use cherry picking methodology on staging:

```bash
git switch staging
git fetch origin staging && git pull --rebase
git cherry-pick -n {BASE-OF-BRANCH}..{branch-name}
```

_Note_: BASE-OF-BRANCH is one commit prior to the first commit of the branch.

The commit message should be in the following format:

```
Merge-squash {branch-name}

(pull request {pull-request-link})
(cherry picked from {commit-sha})
```

Including the pull request link in the message results with a reference in the pull request feed, which is helpful because we'll know when the branch was deployed to `staging`.

You can use this [script](https://app.productive.io/1-infinum/docs/doc/113800?filter=MjQ3Nzgw&page=148751) to generate commit message.

Make sure that everything is OK by running the specs, then push.

#### Keep it clean
After a while the `staging` branch history can become quite different than that of the `main` branch, because of the different branch integrations. You'll sometimes even find yourself pushing a couple of _fixup_ commits to `staging` if you're fixing something and then applying the changes to the `fix/branch`.

The `staging` branch is reset to the `main` branch after each sprint, or more frequently as we see fit:

```bash
git switch staging
git reset --hard origin/master
git push origin staging --force
```

### Main branch
Once the PR has at least one approval, the branch has been successfully deployed to `staging` and tested, and
there are no failing specs, it can be integrated into the `main` branch.

We use one of these two methods to integrate branches to the main branch:

1. Squash and merge
2. Merge without squashing

Before merging we also rebase the feature branch onto the latest `main` branch, so that the git
history is nice and clean, and that the latest code can be run on CI before actually merging into
the `main` branch.

While on feature branch:

```bash
git fetch origin master
git rebase -i origin/master
```

This will start interactive rebase. You can edit it with your editor of choice.
To set the default editor you'll need to edit the git configuration:

```bash
git config core.editor "{editor-name} --wait"
```

Examples for visual studio code, atom and vim:

```bash
git config core.editor "code --wait"
git config core.editor "atom --wait"
git config core.editor "vim --wait"
```

Refer to [this](https://git-scm.com/book/en/v2/Appendix-C%3A-Git-Commands-Setup-and-Config) guide
for more information.

Once the interactive rebase has started, you will see feature branch's commits listed. Each commit will consist of command, SHA (commit hash) and commit's text. Each commit will, initially, have `pick` command. You'll see something similar to this (may vary depending on the editor):

```bash
pick 07c5abd First commit
pick de9b1eb Add controller
pick 3e7ee36 Remove controller
pick fa20af3 Finish up

# Rebase 8db7e8b..fa20af3 onto 8db7e8b
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

Save and close the editor to finish the rebase and then force push to update remote's history:

```bash
git push --force-with-lease
```

#### Squash and merge

After there are no conflicts, select the Squash and merge option on the pull request.
This will combine all commits into a single one and merge it into the master branch.

#### Merge without squashing

If you want to keep all commits from your pull request on the main branch, use the merge without squashing method:

```bash
git fetch origin master
git switch master && git pull
git merge --no-ff --no-edit {branch-name}
```

We perform non-fast-forward merges to group commits of a single feature together in a meaningful way.

Make sure the history graph is nice and clean by entering the following command or similar. No lines should "cross over".

```bash
git log --oneline --graph
```

Bad:

```bash
*   1b82b9f (HEAD -> master) Merge branch 'feature/add-git-process-to-readme'
|\
| * a25b3dc (origin/feature/add-git-process-to-readme, feature/add-git-process-to-readme) Add git process to readme
* |   bfe1152 (origin/master) Merge branch 'feature/xx-some-other-feature'
|\ \
| |/
|/|
| * 3345dbb Some other feature subject
|/
*   7eade95 Merge branch 'feature/xx-another-other-feature'
|\
| * 0a80385 Another feature subject
|/
*
```

Good:

```bash
*   1a164b4 (HEAD -> master) Merge branch 'feature/add-git-process-to-readme'
|\
| * 497dcd7 (origin/feature/add-git-process-to-readme, feature/add-git-process-to-readme) Add git process to readme
|/
*   bfe1152 (origin/master) Merge branch 'feature/xx-some-other-feature'
|\
| * 3345dbb Some other feature subject
|/
*   7eade95 Merge branch 'feature/xx-another-other-feature'
|\
| * 0a80385 Another feature subject
|/
*
```

Make sure that everything is OK by running the specs, then push.

If you're having trouble understanding the differences between squashing, rebasing or any other topics covered in these chapters, you can learn more from the [Git book](https://git-scm.com/book/en/v2).
