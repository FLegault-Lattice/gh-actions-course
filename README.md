# GitHub Actions Course

Repository containing all examples and notes for the GitHub Actions course.

## Table of Contents

- [GitHub Actions Course](#github-actions-course)
  - [Table of Contents](#table-of-contents)
- [Learn the building blocks of GitHub Actions](#learn-the-building-blocks-of-github-actions)
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
- [Controlling the Execution Flow](#controlling-the-execution-flow)
  - [Sequential job execution via the `needs` key](#sequential-job-execution-via-the-needs-key)
- [Inputs](#inputs)
  - [GitHub Action](#github-action)
  - [Caller](#caller)
- [Outputs](#outputs)
- [Caching](#caching)
  - [Use-Case](#use-case)
  - [Example](#example)
- [Artifacts](#artifacts)
  - [Difference between Artifacts \& Caching](#difference-between-artifacts--caching)
- [Matrices](#matrices)
  - [Matrix example](#matrix-example)
    - [Resulting jobs](#resulting-jobs)

---

# Learn the building blocks of GitHub Actions

| Building block | Defined at | Responsibilities | Default behavior |
| --- | --- | --- | --- |
| **Workflows** | Repository level | Define which triggers start the workflow; composed of one or more jobs | N/A |
| **Jobs** | Workflow level | Define the execution environment they run in; composed of one or more steps | Run in parallel |
| **Steps** | Job level | Define the actual script or **GitHub Action** to execute | Run sequentially |

---

# Triggering workflows in multiple ways

There are many ways to trigger GitHub workflows:

- **Repository events**
  - `push`: Triggered when someone pushes to the repository
  - `issues`: Triggered by a variety of events related to issues
  - `pull_request`: Triggered by a variety of events related to PRs
  - `pull_request_review`: Triggered by a variety of events related to PR reviews (submitting, editing, deleting)
  - `fork`: Triggered when the repository is forked
  - ...
- **Manual trigger**
  - **Via the UI:** Triggered from the **Actions** tab in GitHub
  - **Via an API call:** Triggered through GitHub's REST API
  - **From another workflow:** Triggered from within another workflow
- **Schedule**
  - Runs as a cron job

> [!NOTE]
> This is just a sample. The full documentation can be found [here](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows).

---

# Workflow Runners

- **GitHub-hosted (standard)**
  - Managed service
  - A VM is scoped to a job: steps share the VM, but jobs do not (by default, each job receives a clean VM instance)
- **Self-hosted**
  - Run workflows on (almost) any infrastructure of your choice
  - Full control over the VM infrastructure
  - Not managed, meaning we need to take care of OS patching, software updates, among other ops tasks
  - Can be added at the repository, organization, or enterprise level
  - Jobs do not necessarily have to run on clean instances

## Pro-tips

> [!TIP]
> Keep the VM resources in mind, especially when running commands that rely on parallel execution (for example, running parallel jest tests).

## Warning

> [!WARNING]
> **Do not** use self-hosted runners in public repositories.

---

# Workflow Contexts

Access information about runs, variables, jobs, and much more.

GitHub provides multiple sources of data in different contexts so that we can easily provide all the necessary information to our workloads.

## Example of Contexts

| Context | Description |
| --- | --- |
| **github** | Commit SHA, event name, ref of the branch or tag triggering the workflow |
| **env** | Contains variables that have been defined in a workflow, job, or step. Changes based on which part of the workflow is executing |
| **inputs** | Contains input properties passed via the keyword `with` to an action, to a reusable workflow, or to a manually triggered workflow |
| **vars** | Contains custom configuration variables set at the organization, repository, and environment level |

> [!NOTE]
> This is just a sample. The full documentation can be found [here](https://docs.github.com/en/actions/reference/workflows-and-actions/contexts).

---

# Expressions

Use dynamic values and expressions in your workflows.

- Can be used to reference information from multiple sources
- Must use the `${{ <expression> }}` syntax
- Can be any combination of
  - **Literal values**
    - Strings, numbers, booleans, null
  - **Context values**
    - Values passed via the many workflow contexts
  - **Functions**
    - Built-in functions provided by GitHub Actions
- Support the use of functions and operators such as `!`, `<`, `>`, `!=`, `&&`, `||`, and many others

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

---

# Functions

Out-of-the-box functions to model complex behavior.

## General purpose functions

Provides a set of utility functions to interact with data from multiple contexts and model more complex behavior, such as more advanced control of the workflow and job execution.

### Examples

- `contains()`
- `startsWith()`
- `endsWith()`
- `fromJSON()`
  - Can be used to convert variables from `string` to different data types
- `toJSON()`

## Status check functions

Provides a set of functions that allow using the status of the workflow, previous jobs, or steps to define whether a certain job or step should be executed.

### Examples

- `success()`
- `failure()`
- `always()`
- `cancelled()`
  - `!cancelled()` can be used to execute jobs or steps even if previous jobs or steps failed, but prevent execution in case the workflow is cancelled

---

# Controlling the Execution Flow

Execute jobs and steps conditionally, and set dependencies between jobs.

## Sequential job execution via the `needs` key

Jobs wait until their dependencies successfully execute.

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

---

# Inputs

Provide information to customize workflows and actions.

Inputs enable us to request specific information from the workflow or action called and use this information at runtime.

## GitHub Action

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

---

# Outputs

Output data from jobs for later usage.

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

---

# Caching

Speed up workflow runs by caching stable files.

Caching allows us to store files and later retrieve them based on a key. Workflows can access the cache from another branch or from the default branch.

## Use-Case

Cache dependencies to speed up execution by avoiding always downloading and installing them.

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

---

# Artifacts

Share data between jobs and store data after workflows have completed

## Difference between Artifacts & Caching

**Artifacts**

- Stored for up to **90 days**
- Managed via two actions
  - `upload-artifacts`
  - `download-artifacts`
- Recomended when the stored files are likely to be accessed outside the workflow, for example
  - Build outputs
  - Test results
  - Logs

**Caching**

- Stored for up to **7 days**
- Managed via a single action
  - `cache`
- Recommended when the stored files are likely to be accessed only within the workflow, for example
  - Build dependencies

---

# Matrices

Run several variations of the same job.

- Defined via the `strategy.matrix` key of a job
- A job is created for every combination of the values provided
- Values can be accessed via the `matrix` context

> [!NOTE]
> **Use-case:** Run test suits in parallel in multiple Node version before publising an NPM package to ensure backward compatibility.

> [!WARNING]
> Each executed jobs counts toward billing purposes. A matrix generating 9 jobs of 3 minutes each will lead to 27 minutes of billed time.

## Matrix example

```yaml
name: My NPM package workflow
on: push

jobs:
  backwards-compatibility:
    name: ${{ matrix.os }}-${{ matrix.node }}
    strategy:
      matrix:
        node: [14, 16, 18]
        os:
          - ubuntu-latest
          - macos-latest
          - windows-latest
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node }}
```

### Resulting jobs

The workflow above creates nine jobs, named after the `name` key:

- ubuntu-latest-14
- ubuntu-latest-16
- ubuntu-latest-18
- macos-latest-14
- macos-latest-16
- macos-latest-18
- windows-latest-14
- windows-latest-16
- windows-latest-18