# 5 - Custom actions
In this lab you will create and use custom actions.
> Duration: 15-20 minutes

References:
- [Creating actions](https://docs.github.com/en/actions/creating-actions)
- [Creating a composite action](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action)
- [Creating a JavaScript action](https://docs.github.com/en/actions/creating-actions/creating-a-javascript-action)
- [GitHub Actions Toolkit](https://github.com/actions/toolkit)
- [actions/github-script](https://github.com/actions/github-script)

## 5.1 Use the github-script action to apply a label to an issue

1. Open the workflow file [github-script.yml](/.github/workflows/github-script.yml)
2. Edit the file and copy the following YAML content at the end of the file:
```YAML
  apply-label:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.addLabels({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              labels: ['Training']
            })
```
3. Commit the changes into the `main` branch
4. Open a new issue or edit an exiting one to trigger the workflow. If the `Issues` tab is not visible, open your repository settings and enable it.
5. Go to `Actions` and see the details of your running workflow
6. After the workflow completes, a new label should be applied to your issue

## 5.2 Create a new branch 
1. Create a new branch named `module-05` based on `main`
2. Switch to the `module-05` branch in the `code` view
   
## 5.3 Use a composite action

1. Open the composite action file [/.github/actions/hello-world-composite-action/action.yml](/.github/actions/hello-world-composite-action/action.yml)
2. Edit the file and copy the following YAML content at the end of the file:
```YAML
    - name: Hello world
      uses: actions/hello-world-javascript-action@main
      with:
        who-to-greet: "${{ inputs.who-to-greet }}"
      id: hello
    - name: Echo the greeting's time
      run: echo 'The time was ${{ steps.hello.outputs.time }}.'
      shell: bash
```
3. Commit the changes to `module-05` branch
4. Open the workflow file [hello-world-composite.yml](/.github/workflows/hello-world-composite.yml)
5. Edit the file and copy the following YAML content at the end of the file:
```YAML
  hello_world_job2:
    runs-on: ubuntu-latest
    name: A job2 to say hello
    steps:
      - uses: actions/checkout@v7
      - id: hello-world
        uses: ./.github/actions/hello-world-composite-action
        with:
          who-to-greet: 'Mona the Octocat from composite action'
      - run: echo random-number from composite action ${{ steps.hello-world.outputs.random-number }}
        shell: bash
```
6. Update the workflow to run on push events
```YAML
on:
  push:
     branches: module-05
  workflow_dispatch:    
```
7. Commit the changes into the same `module-05` branch
8. Go to `Actions` and see the details of your running workflow

## 5.4 Custom JS and Docker actions
1. Study the implementation of the custom action from the folder: [/.github/actions/](/.github/actions/)
2. Open the workflow file [use-custom-actions.yml](/.github/workflows/use-custom-actions.yml)
3. Edit the file and copy the following YAML content to update the issue title:
```YAML
         issue-title: "A joke for you from custom actions workflow" 
```
4. Commit the changes into the `main` branch
5. Go to `Actions` and manually trigger the workflow by clicking on `Run Workflow` button
6. See the details of your running workflow

## 5.5 (Optional) Create a JavaScript action
1. Follow the guide to create a JavaScript action
    - [Creating a JavaScript action](https://docs.github.com/en/actions/creating-actions/creating-a-javascript-action)
2. Use your action in a workflow
```YAML
      - name: Hello world action step
        id: hello
        uses: /.github/actions/hello-world-javascript-action@v1.1
        with:
            who-to-greet: 'Mona the Octocat'
```

## 5.6 Final
<details>
  <summary>github-script.yml</summary>
  
```YAML
name: 05-1. GitHub Script - Thank you
on:
  issues: 
    types: [opened, edited, reopened, labeled]

# Limit the permissions of the GITHUB_TOKEN
permissions:
  contents: read
  issues: write

jobs:
  comment:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/github-script@v7
        with:
          github-token: ${{secrets.GITHUB_TOKEN}}
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '👋 Thank you! We appreciate your contribution to this project.'
            })
  apply-label:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.addLabels({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              labels: ['Training']
            })
```
</details>

<details>
  <summary>hello-world-composite-action/action.yml</summary>
  
```YAML
name: 'Hello World Composite Action'
description: 'Greet someone'
inputs:
  who-to-greet:  # id of input
    description: 'Who to greet'
    required: true
    default: 'World'
outputs:
  random-number:
    description: "Random number"
    value: ${{ steps.random-number-generator.outputs.random-id }}
runs:
  using: "composite"
  steps:
    - run: echo Hello from composite action ${{ inputs.who-to-greet }}.
      shell: bash
    - id: random-number-generator
      run: echo "random-id=$(echo $RANDOM)" >> $GITHUB_OUTPUT
      shell: bash
    - run: echo "${{ github.action_path }}" >> $GITHUB_PATH
      shell: bash    
    - name: Hello world
      uses: actions/hello-world-javascript-action@main
      with:
        who-to-greet: "${{ inputs.who-to-greet }}"
      id: hello
    - name: Echo the greeting's time
      run: echo 'The time was ${{ steps.hello.outputs.time }}.'
      shell: bash      

```
</details>

<details>
  <summary>hello-world-composite.yml</summary>
  
```YAML
name: 05-2. Hello World Composite

on:
  push:
     branches: [module-05]
  workflow_dispatch:  

jobs:
  hello_world_job1:
    runs-on: ubuntu-latest
    name: A job1 to say hello
    steps:
      - id: hello-world
        uses: githubabcs/hello-world-composite-action@main
        with:
          who-to-greet: 'Hello from GH ABCs'
      - run: echo random-number ${{ steps.hello-world.outputs.random-number }}
        shell: bash
  hello_world_job2:
    runs-on: ubuntu-latest
    name: A job2 to say hello
    steps:
      - uses: actions/checkout@v4
      - id: hello-world
        uses: ./.github/actions/hello-world-composite-action
        with:
          who-to-greet: 'Mona the Octocat from composite action'
      - run: echo random-number from composite action ${{ steps.hello-world.outputs.random-number }}
        shell: bash
```
</details>

<details>
  <summary>use-custom-actions.yml</summary>
  
```YAML
name: 05-3. Use Custom Actions (JS & Doker)

on:
  pull_request:
    types: [labeled]
  workflow_dispatch:

# Limit the permissions of the GITHUB_TOKEN
permissions:
  contents: read
  issues: write

jobs:
  
  js-custom-actions:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7

      - run: echo "🎉 Running the JS actions"

      - name: hello-action
        uses: ./.github/actions/hello-world-js
        if: ${{ success() }}

      - name: ha-ha
        uses: ./.github/actions/joke-action
        id: jokes

      - name: create-issue
        uses: ./.github/actions/issue-maker-js
        with:
          repo-token: ${{secrets.GITHUB_TOKEN}}
          joke: ${{steps.jokes.outputs.joke-output}}
          issue-title: "A joke for you from custom actions workflow"       

  docker-custom-actions:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v7

      - run: echo "🎉 Running the Docker actions"

      - name: hello-action
        uses: ./.github/actions/hello-world-docker

      - name: meow
        uses: ./.github/actions/cat-facts
        id: cat

      - name: create-issue
        uses: ./.github/actions/issue-maker-docker
        with:
          repoToken: ${{secrets.GITHUB_TOKEN}}
          catFact: ${{steps.cat.outputs.fact}}
          issueTitle: "A cat fact for you from ${{ github.repository_owner }}"

```
</details>
