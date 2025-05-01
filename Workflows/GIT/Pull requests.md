Pull requests initiate discussion about your commits. Since they're tightly integrated with the underlying Git repository, everyone can see exactly what changes will be merged if they accept your request.

## New pull request
Once a feature or fix is done, a PR for it should be opened.

It's essential to write a good pull request description. Reviewers will usually read it before
looking at the diff, so make sure it gives them enough context to know what they are looking at.

Another purpose of a good description is to provide the documentation for a future reference.
A new developer might join the project in the future. The description will help him understand
the motivation behind the implementation of a specific feature. Please note that any important information
from the description should also be added to the main documentation in the project as a central reference point
where the latest state of the project is documented.

A new Rails project created with the [Infinum rails template](https://github.com/infinum/default_rails_template)
includes a [pull request description template](https://help.github.com/en/articles/creating-a-pull-request-template-for-your-repository).
The template is defined in the `rails_project/.github/PULL_REQUEST_TEMPLATE.md` file.

```
Task: [#__TASK_NUMBER__](__ADD_URL_TO_PRODUCTIVE_TASK__)

#### Aim


#### Solution

```

### Link to the task
Replace `__TASK_NUMBER__` with a number of the task from Productive (eg. `149`) and
`__ADD_URL_TO_PRODUCTIVE_TASK__` with the url to the task (eg. `https://app.productive.io/1-infinum/m/task/487456`).

### Aim
In the Aim section, provide enough information for reviewers to have context for your changes.

- Why are we introducing this change?
- Why now?
- What problems is the code solving now?
- How does it work on a business level?
- Why have we picked this solution historically, etc.?

### Solution
Explain why things are done the way they are in this PR. Highlight the most important and/or controversial design decisions you have taken.

- Why is it implemented the way it is?
- What alternative implementations have you considered and why have you chosen this one, etc.?

The description is a good place to include questions that came up during development.

- Is this class/method name the best one?
- Could approach B be more applicable in this case, etc.?

This is also a good place to talk about the performance and security considerations if there are any.

## Default pull request reviewers
It's a bit tedious to add the same reviewers to pull requests over and over again. GitHub allows us to set a
list of default PR reviewers.

A new rails project created with the [Infinum rails template](https://github.com/infinum/default_rails_template)
includes a list of default pull request reviewers known as [Code Owners](https://help.github.com/en/articles/about-code-owners)).
The list is defined in the `rails_project/.github/CODEOWNERS` file.

When `rails new` command is run, a developer is prompted to enter a list of GitHub username handles
that are automatically added to the file as code owners.
