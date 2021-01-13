# GitHub Actions `workflow_run` event

This repository contains a code example to accompany my answer to the question [_"How to use the GitHub Actions `workflow_run`  event?"_](stackoverflow.com/questions/63343937/how-to-use-the-new-event-workflow-run-of-gtihub-action-added-recently/), asked on StackOverflow

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
    runs-on: ubuntu-18.04
    steps:
      - name: Debug
        shell: bash
        run: 'echo -e "GITHUB_WORKFLOW: ${GITHUB_WORKFLOW}\\nGITHUB_EVENT_NAME: ${GITHUB_EVENT_NAME}"'
```

The second file states that it should only trigger `on` the `workflow_run` event for any `workflows` with the name `Preflight`:

```yml
---
name: Test

on:
  workflow_run:
    workflows:
      - Preflight

jobs:
  test-job:
    name: Test Step
    runs-on: ubuntu-18.04
    steps:
      - name: Debug
        shell: bash
        run: 'echo -e "GITHUB_WORKFLOW: ${GITHUB_WORKFLOW}\\nGITHUB_EVENT_NAME: ${GITHUB_EVENT_NAME}"'
```

This is exactly the same as [the example from the GitHub Actions manual](https://docs.github.com/en/free-pro-team@latest/actions/reference/events-that-trigger-workflows#workflow_run).


As you can see on [the actions page](https://github.com/potherca-blog/github-actions-workflow_run-event/actions) of [my example repo](https://github.com/potherca-blog/github-actions-workflow_run-event), the Preflight workflow will run first. After it has completed, the Test workflow will be triggered:

[![enter image description here][1]][1]


  [1]: https://i.stack.imgur.com/14Bbn.png
