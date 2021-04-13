After a while you'll get notifications about comments on your pull requests from your reviewers. Oftentimes you'll need to change something before the branch is ready.

## Solving change requests
Change requests should be applied in separate _fixup_ or _squash_ commits. Rebasing the branch during an ongoing review is not appreciated unless there is a good reason for it, like pulling in some new and necessary changes from `master`, because it makes it harder for the reviewers to know what the new changes are and what they have already reviewed.

These commits should be merged into `staging` as well when they are done.

## Merging

### Staging
Once the PR is opened, the branch can be merged into `staging`, unless it contains considerable
logic in the migrations, in which case the reviewers should prioritize reviewing the migrations,
and giving a thumbs up before merging.

There are two ways of merging branches to `staging`:

1. Merge & squash
2. Cherry-picking a commit range

#### Merge squash

```bash
git switch staging
git fetch origin staging && git pull --rebase
git merge --squash {branch-name}
```

#### Cherry-picking

```bash
git switch staging
git fetch origin staging && git pull --rebase
git cherry-pick -n {BASE-OF-BRANCH}..{branch-name}
```

_Note_: BASE-OF-BRANCH is one commit prior to the first commit of the branch.

Cherry picking usually produces less merge conflicts once `master` and `staging` diverge.

The commit message should be in the following format.

```
Merge-squash {branch-name}

(pull request {pull-request-link})
[optionally](cherry picked from {commit-sha})
```

Including the pull request link in the message results with a reference in the pull request feed, which is helpful because we'll know when the branch was deployed to `staging`.

Make sure that everything is OK by running the specs, then push.

#### Keep it clean
After a while the `staging` branch history can become quite different than that of the `master` branch, because of the different branch merges. You'll sometimes even find yourself pushing a couple of _fixup_ commits to `staging` if you're fixing something and then applying the changes to the `fix/branch`.

The `staging` branch is reset to `master` after each sprint, or more frequently as we see fit:

```bash
git switch staging
git reset --hard origin/master
git push origin staging --force
```

### Master
Once the PR has at least one approval, the branch has been successfully deployed to `staging` and tested, and
there are no failing specs, it can be merged into `master`.

We perform non-fast-forward merges to group commits of a single feature together in a meaningful
way. Before merging we also rebase the feature branch onto the latest `master`, so that the git
history is nice and clean, and that the latest code can be run on CI before actually merging into
`master`. We also squash the review comments corrections here, so make sure the `--autosquash`
option is on by default or add that flag to the rebase command.

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

At this point you'll want to change all commands to `squash` or even `fixup`. The difference between `squash` and `fixup` is that `fixup` will discard log for that commit. This is how the commit list looks after the change:

```bash
pick 07c5abd First commit
squash de9b1eb Add controller
squash 3e7ee36 Remove controller
squash fa20af3 Finish up
```

Or you can even use shorthand flag for the command, for instance `s` instead of `squash`

```bash
pick 07c5abd First commit
s de9b1eb Add controller
s 3e7ee36 Remove controller
s fa20af3 Finish up
```

You should rename your squashed commit message into something more meaningful, so pick `reword` for the first commit. Edit commit message, save and close editor to finish the rebase.

Next, force push to update remote's history:

```bash
git push --force-with-lease
```

Then, check once again that everything is squashed correctly and continue on to the `master` branch:

```bash
git fetch origin master
git switch master && git pull
git merge --no-ff --no-edit {branch-name}
```

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


Testing new things
