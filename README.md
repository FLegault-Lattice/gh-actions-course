Repository containing all examples and notes for the Github Actions course.

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
