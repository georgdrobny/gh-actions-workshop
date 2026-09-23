# 4 - Workflow Templates
In this lab you will reuse workflow templates.
> Duration: 10-15 minutes

References:
- [Reusing workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [Sharing workflows with your organization](https://docs.github.com/en/actions/using-workflows/sharing-workflows-secrets-and-runners-with-your-organization)
- [Sharing actions and workflows with your enterprise](https://docs.github.com/en/enterprise-cloud@latest/actions/creating-actions/sharing-actions-and-workflows-with-your-enterprise)
- [Using starter workflows](https://docs.github.com/en/actions/using-workflows/advanced-workflow-features#using-starter-workflows)

## 4.1 Create a new branch 
1. Create a new branch named `module-04` based on `main`
2. Switch to the `module-04` branch in the `code` view
   
## 4.2 Create a reusable workflow
1. For a workflow to be reusable, the `on` must include the `workflow_call` event
2. Open the workflow file [job-dependencies.yml](/.github/workflows/job-dependencies.yml)
3. Edit the file and add three hyphens on line 1 before the name of the workflow
```YAML
---
name: 02-2. Dependencies
```
4. Edit the file and update the workflow to run on workflow call event
```YAML
on:
  workflow_call:
```
5. Commit the changes into `module-04` branch
6. Go to `Code` and select the `module-04` from the branches drop down list
7. Open the workflow file [reusable-workflow-template.yml](/.github/workflows/reusable-workflow-template.yml)
8. Edit the file and copy the following YAML content at the end of the file:
```YAML
  call_dependencies_workflow_job:
    needs: call_reusable_workflow_job
    uses: ./.github/workflows/job-dependencies.yml
```
9. Update the workflow to run on push events
```YAML
on:
  push:
     branches: [module-04]
  workflow_dispatch:    
```
10. Commit the changes into the same `module-04` branch
11. Go to `Actions` and see the details of your running workflow

## 4.2 Final
<details>
  <summary>job-dependencies.yml</summary>
  
```YAML
---
name: 02-2. Dependencies 

on:
  workflow_call:
    
jobs:
  initial:
    runs-on: ubuntu-latest
    steps:
      - run: echo "This job will be run first."
  fanout1:
    runs-on: ubuntu-latest
    needs: initial
    steps:
      - run: echo "This job will run after the initial job, in parallel with fanout2."
  fanout2:
    runs-on: ubuntu-latest
    needs: initial
    steps:
      - run: echo "This job will run after the initial job, in parallel with fanout1."
  fanout3:
    runs-on: ubuntu-latest
    needs: fanout1
    steps:
      - run: echo "This job will run after the initial job, in parallel with fanout2."
  fanin:
    runs-on: ubuntu-latest
    needs: [fanout1, fanout2]
    steps:
      - run: echo "This job will run after fanout1 and fanout2 have finished."
  build:
    runs-on: ubuntu-latest  
    strategy:
      matrix:
        configuration: [debug, release]
    steps:
    - run: echo "This job builds the cofiguration ${{ matrix.configuration }}."
  test:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - run: echo "This job will be run after the build job."
  ring01:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - run: echo "This job will be run after the test job."
  ring02:
    runs-on: macos-latest
    needs: test
    steps:
      - run: echo "This job will be run after the test job."
  ring03:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - run: echo "This job will be run after the test job."
  ring04:
    runs-on: ubuntu-latest
    needs: [ring01,ring02,ring03]
    steps:
      - run: echo "This job will be run after the ring01,ring02,ring03 jobs."
  prod:
    runs-on: ubuntu-latest
    needs: [ring04]
    steps:
      - run: echo "This job will be run after the ring04 job."
```
  </details>

<details>
<summary>reusable-workflow-template.yml</summary>
  
```YAML
name: 04-1. Call Reusable Workflow Templates

on:
  push:
     branches: [main]
  workflow_dispatch:    

jobs:
  call_greet_everyone_workflow_job:
    uses: ./.github/workflows/greet-everyone.yml@main
    with:
      name: 'Reusable Workflow Templates'
    
  call_reusable_workflow_job:
    uses: ./.github/workflows/super-linter.yml@main

  call_demo_workflow_job:
    needs: call_greet_everyone_workflow_job
    uses: ./gh-abcs-actions/.github/workflows/github-actions-demo.yml@main
  
  call_dependencies_workflow_job:
    needs: call_reusable_workflow_job
    uses: ./.github/workflows/job-dependencies.yml@main

```
</details>

