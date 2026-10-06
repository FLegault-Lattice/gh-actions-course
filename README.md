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
