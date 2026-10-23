Repository containing all examples and notes for the Github Actions course.

- [Learn the building blocks of Github Actions.](#learn-the-building-blocks-of-github-actions)
- [Triggering workflows in multiple ways](#triggering-workflows-in-multiple-ways)
- [Workflow Runners](#workflow-runners)
  - [Pro-tips](#pro-tips)
  - [Warning](#warning)
- [Workflow Contexts](#workflow-contexts)
  - [Example of Contexts](#example-of-contexts)
- [Expressions](#expressions)
  - [Expressions example](#expressions-example)
- [Functions](#functions)
  - [General purpose functions](#general-purpose-functions)
    - [Examples](#examples)
  - [Status check functions](#status-check-functions)
    - [Examples](#examples-1)
- [Controlling the Execution Flow](#controlloing-the-execution-flow)
  - [Sequential job execution via the `needs` key](#sequential-job-execution-via-the-needs-key)
- [Inputs](#inputs)
  - [Github Action](#github-action)
  - [Caller](#caller)
- [Outputs](#outputs)
- [Caching](#caching)
  - [Use-Case](#use-case)
  - [Example](#example)


# Learn the building blocks of Github Actions.
- **Workflows:**
  - Are defined at the repository level
  - Define which triggers actually start the workflow
  - Are composed of one or more jobs
- **Jobs:**
  - Are defined at the workflow level
  - Define in which execution environment they are run
  - Are composed of one or more steps
  - Run in parallel by default
- **Steps:**
  - Are defined at the job level
  - Define the actual script or **Github Action** that will be executed
  - Run sequentially by default

# Triggering workflows in multiple ways
There are many ways we can trigger Github workflows:

- **Repository events:**
  - **push:** Triggered when someone pushes to the repo
  - **issues:** Triggered by a variety of events related to issues
  - **pull_request:** Triggered by a variety of events related to PRs
  - **pull_request_review:** Triggered by a variety of events related to PR reviews (submitting, editing, deleting)
  - **fork:** Triggered when your repository is forked
  - ...
- **Manual trigger:**
  - **Triggered via the UI:** Triggered from the **Actions** tab in Github
  - **Triggered via an API call:** Triggered via Github's REST API
  - **Triggered from another workflow:** Triggered from within another workflow
- **Schedule:**
  - Run as a cron job

***This is just a sample the full documentation can be found [here](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows)***

# Workflow Runners
- **Github-hosted (standard)**
  - Managed service
  - A VM is scoped to a job: steps share the VM, but jobs don't (by default, each job receives a clean VM instance)
- **Self-Hosted**
  - Run workflows on (almost) any infrastructure of your choice
  - Full control over the VM infrastructure
  - It's not managed, meaning we need to take care of OS patching, software updates, among other ops tasks
  - Can be added at the repository, organization, or enterprise level
  - Jobs do not necessarily have to run on clean instances

## Pro-tips

Keep the VM resources in mind, especially when running commands that rely on parallel execution (for example, running parallel jest tests).

## Warning

**Do not** use self-hosted runners in public repositories.

# Workflow Contexts

Access information about runs, variables, jobs, and much more

Github provides multiple sources of data in different contexts so that we can easily provide all the necessary information to our workloads

## Example of Contexts

| Context | Description |
| --- | --- |
| **github** | Commit SHA, Event name, Ref of branch or tag triggering the workflow |
| **env** | Contains variable that have been defined in a workflow, job, or step. Changes based on which part of the workflow is executing |
| **inputs** | Contains input properties passed via the keyword `with` to an action, to a reusable workflow, or to a manually triggered workflow | 
| **vars** | Contains custom configuration variables set at the organization, repository, and environment level

***This is just a sample the full documentation can be found [here](https://docs.github.com/en/actions/reference/workflows-and-actions/contexts)***

# Expressions

Use dynamic values and expressions in your workflows

- Can be used to reference information from multiple sources
- Must use the `${{ <expression> }}` syntax
- Can be any combination of
  - **Literal values**
    - Strings, numbers, booleans, null
  - **Context values**
    - Values passed via the many workflow contexts.
  - **Functions**
    - Built-in functions provided by Github Actions.
- Support the use of functions and operators such as `!`, `<`, `>`, `!=`, `&&`, `||`, and many others.

## Expressions example
```yaml
name: My NPM workflow
on: [push, pull_request]

jobs:
  node-16-test:
    runs-on: ubuntu-latest
    steps:
      - if: github.event_name == 'push'
        run: |
          echo "Triggered by a push event"
          echo "Running on ${{ github.ref_name }}"
      - uses: actions/setup-node@v3
        with:
          node-version: 16
      - run: npm ci
      - run: npm test
```

# Functions

Out-of-the-box functions to model complex behavior

## General purpose functions

Provides a set of utility functions to interact with data from multiple contexts and model more complex behavior, such as more advanced control of the workflow and job execution

### Examples
- `contains()`
- `startsWith()`
- `endsWith()`
- `fromJSON()`
  - Can be used to convert variables from `string` to different data types
- `toJSON()`

## Status check functions

Provides a set of functions that allow using the status of the workflow, previous jobs or steps to define whether a certain job or step should be executed

### Examples
- `success()`
- `failure()`
- `always()`
- `cancelled()`
  - `!cancelled()` can be used to execute jobs or steps even if previous jobs or steps failed, but prevent execution in case the workflow is cancelled

# Controlling the Execution Flow

Execute jobs and steps conditionally, and set dependencies between jobs

## Sequential job execution via the `needs` key

Jobs wait until their dependencies successfully execute

```yaml
jobs:
  job1:
  job2:
  job3:
    needs:
      - job1
      - job2
  job4:
    needs: job1
```

# Inputs

Provide information to customize workflows and actions

inputs enable us to request specific information from the workflow or action called and use this information at runtime.

## Github Action
```yaml
inputs:
  url:
    description: '...'
    required: true
  max-trials:
    description: '...'
    required: false
    default: '60'
```

## Caller
```yaml
jobs:
  job1:
    steps:
      - name: Ping URL
        uses: ping-url-example@v1
        with:
          url: https://www.google.com
          max-trials: 10
```

# Outputs

Output data from jobs for later usage

1. Define a step ID for the steps that produce outputs
2. Echo key-value pairs to the `$GITHUB_OUTPUT` variable
3. Mention the outputs in the `outputs` section of the job
4. List the job as a dependency of jobs that need the output
5. Access the output via the `needs` context

```yaml
jobs:
  welcome:
    runs-on: ubuntu-latest
    outputs:
      name: ${{ steps.step1.outputs.NAME }}
    steps:
      - id: step1
        run: echo "NAME=Lauro" >> "$GITHUB_OUTPUT"
  goodbye:
    runs-on: ubuntu-latest
    needs: welcome
    steps:
      - run: echo "Bye, ${{ needs.welcome.outputs.name }}"
```

# Caching

Speed up workflow runs by caching stable files

Caching allows us to store files and later retrieve them based on a key. Workflows can access the cache from another branch or from the default branch.

## Use-Case

Cache dependencies to speed up execution by avoiding always downloading and installing them

## Example

```yaml
steps:
  - uses: actions/cache@v3
    id: cache
    with:
      path: node_modules
      key: ${{ hashFiles('**/package-lock.json') }}
  - name: Install dependencies
    if: steps.cache.outputs.cache-hit != 'true'
    run: npm ci
  - name: Lint & test
    run: |
      npm run lint
      npm run test
```