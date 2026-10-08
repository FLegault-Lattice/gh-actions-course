Repository containing all examples and notes for the Github Actions course.

- [Learn the building blocks of Github Actions.](#learn-the-building-blocks-of-github-actions)
- [Triggering workflows in mutliple ways](#triggering-workflows-in-mutliple-ways)
- [Workflow Runners](#workflow-runners)
  - [Pro-tips](#pro-tips)
  - [Warning](#warning)
- [Workflow Contexts](#workflow-contexts)
  - [Example of Contexts](#example-of-contexts)


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

# Triggering workflows in mutliple ways
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
  - It's not managed, meaning we need to take care of OS patching, software updates, amoung other ops tasks
  - Can be added at the repository, organization, or enterprise level
  - Jobs do not necessarily have to run on clean instances

## Pro-tips

Keep the VM resources in mind, especially when running commands that rely on parallel execution (for example, running parallel jest tests).

## Warning

**Do not** use self-hosted runners in public repositories.

# Workflow Contexts

Access information about runs, variables, jobs, and much more

Github providfes multiple sources of data in different contexts so that we can easily provide all the necessary information to our workfloads

## Example of Contexts

| Context | Description |
| --- | --- |
| **github** | Commit SHA, Event name, Ref of branch or tag triggering the workflow |
| **env** | Contains variable that have been defined in a workflow, job, or step. Cchanges based on which part of the workflow is executing |
| **inputs** | Contains input properties passed via the keyword `with` to an action, to a reusable workflow, or to a manually triggered workflow | 
| **vars** | Contains custom configuration variables set at the organization, repository, and environment level

***This is just a sample the full documentation can be found [here](https://docs.github.com/en/actions/reference/workflows-and-actions/contexts)***