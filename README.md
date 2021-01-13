# GitHub Actions `workflow_run` event

This repository contains a code example to accompany my answer to the question [_"How to use the GitHub Actions `workflow_run`  event?"_](stackoverflow.com/questions/63343937/how-to-use-the-new-event-workflow-run-of-gtihub-action-added-recently/), asked on StackOverflow

## The Question

> Could anybody tell me how to implement the **example** proposed using the new event `workflow_run`? The documentation only provide a very simple example:
> 
> ```yaml
> on:
>   workflow_run:
>     workflows: ["Run Tests"]
>     branches: [main]
>     types: 
>       - completed
>       - requested
> ```

## My Answer

To get the example to work (i.e. to have one workflow wait for another to complete) you need two files. Both files live in the `.github/workflows` folder of a repository.

The first file would be set up as usual. This file will be triggered by whatever event(s) are set in the `on` section:

```yml
---
name: Preflight

on:
  - pull_request
  - push

jobs:
  preflight-job:
    name: Preflight Step
    runs-on: ubuntu-latest
    steps:
      - run: env
```

The second file states that it should only trigger `on` the `workflow_run` event for any `workflows` with the name `Preflight`:

```yml
---
name: Test

on:
  workflow_run:
    workflows:
      - Preflight
    types:
      - completed

jobs:
  test-job:
    name: Test Step
    runs-on: ubuntu-latest
    steps:
      - run: env
```

This more-or-less the same as [the example from the GitHub Actions manual][github-docs-workflow_run].

As you can see on [the actions page][potherca-example-repo-actions]
of [my example repo][potherca-example-repo], the Preflight workflow will run first. After it has completed, the Test workflow will be triggered:

![Screenshot of workflows screen][screenshot-1]

As you can _also_ see, the **branch** does not appear for the "Test" workflow.

This is because, (quoting from the manual):

![Screenshot of caveat in the manual][screenshot-2]

> This event will only trigger a workflow run if the workflow file is on the default branch.

This means that the "Test" workflow will run on/with the code from the default branch (usually `main` or `master`).

There is a workaround for this...

Every actions is run with a set of [contexts][github-docs-contexts]. The `github` context holds information about the event that triggered the workflow. This includes the branch that the event was originally triggered from/for: `github.event.workflow_run.head_branch`.

This can be used to check out the origination branch in the action, using the [`actions/checkout`][github-checkout-action] action provided by GitHub.

To do this, the Yaml would be:

```yml
---
name: Test

on:
  workflow_run:
    workflows:
      - Preflight
    types:
      - completed

jobs:
  test-job:
    name: Test Step
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          ref: ${{ github.event.workflow_run.head_branch }}
      - run: git branch
      - run: env
```
[github-checkout-action]: https://github.com/actions/checkout
[github-docs-contexts]: https://docs.github.com/en/free-pro-team@latest/actions/reference/context-and-expression-syntax-for-github-actions
[github-docs-workflow_run]: https://docs.github.com/en/free-pro-team@latest/actions/reference/events-that-trigger-workflows#workflow_run
[potherca-example-repo-actions]: https://github.com/potherca-blog/github-actions-workflow_run-event/actions
[potherca-example-repo]: https://github.com/potherca-blog/github-actions-workflow_run-event
[screenshot-1]: https://i.stack.imgur.com/14Bbn.png
[screenshot-2]: https://i.stack.imgur.com/RpEBC.png
