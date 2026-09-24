# 7 - CI/CD
In this lab you will use CI/CD workflows.
> Duration: 15-20 minutes

References:
- [About continuous integration](https://docs.github.com/en/actions/automating-builds-and-tests/about-continuous-integration)
- [Using a build matrix for your jobs](https://docs.github.com/en/actions/using-jobs/using-a-build-matrix-for-your-jobs)
- [Storing workflow data as artifacts](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts)
- [About continuous deployment](https://docs.github.com/en/actions/deployment/about-deployments/about-continuous-deployment)
- [Using concurrency](https://docs.github.com/en/actions/using-jobs/using-concurrency)

## 7.1 Create a new branch 
1. Create a new branch named `module-07` based on `main`
2. Switch to the `module-07` branch in the `code` view
   
## 7.2 Update the CI workflow
1. Open the workflow file [ci-workflow.yml](/.github/workflows/ci-workflow.yml)
2. Edit the file and change the branch name in the `push` trigger to use the `module-07` branch
```YAML
on:
  push:
    branches: [module-07]
  pull_request:
    branches: [main]
```
3. Edit the file and copy the following YAML content to replace the `strategy` of the `ci` job:
```YAML
    strategy:
      # Cancel all matrix jobs if one of them fails
      fail-fast: true
      matrix:
        # our matrix for testing across node versions and OSs
        node-version: [12, 14, 16]
        os: [macos-latest, windows-latest, ubuntu-latest]
```
4. In the `ci` job, before the `deploy-test` job, copy the following YAML content to use the `upload-artifact` action:
```YAML
      - shell: bash
        run: |
          echo 'Test upload artifact' > output.log
      - name: Upload output file
        uses: actions/upload-artifact@v7
        with:
          name: output-log-file
          path: output.log
```
5. In the `deploy-test` job, after the `checkout` action, copyt the following YAML content to use the `download-artifact` action
```YAML
      - name: Download a single artifact
        uses: actions/download-artifact@v8
        with:
          name: output-log-file
```
6. Commit the changes into `module-07` branch
7. Go to `Actions` and see the details of your running workflow

## 7.3 Update the CD workflow

1. Open the workflow file [cd-workflow.yml](/.github/workflows/cd-workflow.yml) (in the `module-07`branch)
2. Edit the file and copy the following YAML content before the `Deploy to production` step:
```YAML
    - name: Download artifact from build job
      uses: actions/download-artifact@v8
      with:
        name: node-app
```
3. Update the workflow to run on push events
```YAML
on:
  push:
     branches: [module-07]
```
4. Commit the changes into the `module-07` branch
5. Go to `Actions` and see the details of your running workflow

## 7.4 Final
<details>
  <summary>ci-workflow.yml</summary>
  
```YAML
name: 07-1. CI Workflow

# Trigger CI for every PR event, when PR has target branch = main
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  # The first job lints the code base
  lint:
    uses: githubabcs/gh-abcs-actions/.github/workflows/super-linter.yml@main

  # CI job to run a test suite on the code base
  ci:
    name: CI
    # We want to test across mutiple OSs, defined by our matrix
    runs-on: ${{ matrix.os }}
    needs: lint
    strategy:
      # Cancel all matrix jobs if one of them fails
      fail-fast: true
      matrix:
        # our matrix for testing across node versions and OSs
        node-version: [16, 18, 20]
        os: [macos-latest, windows-latest, ubuntu-latest]
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      # Configure our node environment according to matrix
      - name: Setup node ${{ matrix.node-version }} on ${{ matrix.os }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}

      - name: Run test suite
        run: |
          echo npm ci
          echo npm run build --if-present
          echo npm test

      # Add here the upload-artifact action
      - shell: bash
        run: |
          echo 'Test upload artifact' > output.log
      - name: Upload output file
        uses: actions/upload-artifact@v8
        with:
          name: output-log-file
          path: output.log

  # If both linting and CI succeeds we want to deploy the code to a test environment
  deploy-test:
    name: Deploy to test env
    runs-on: ubuntu-latest
    needs: ci
    environment:
      name: TEST
      url: https://test.company.com
    steps:
      - name: Checkout
        uses: actions/checkout@v8

      # Add here the download-artifact step
      - name: Download a single artifact
        uses: actions/download-artifact@v4
        with:
          name: output-log-file

      # Placeholder - this step would be some action or run commands that deploys the code
      - name: Deploy to test env
        if: ${{ success() }}
        run: |
          echo "Deploying to test environment"

```
</details>

<details>
  <summary>cd-workflow.yml</summary>
  
```YAML
name: 07-2. CD Workflow 

on:
  push:
     branches: [main]

env:
  AZURE_WEBAPP_NAME: your-app-name    # set this to your application's name
  AZURE_WEBAPP_PACKAGE_PATH: '.'      # set this to the path to your web app project, defaults to the repository root
  NODE_VERSION: '14.x'                # set this to the node version to use

# We only want to allow one deploy-to-prod workflow running at any point in time
concurrency: 
  group: cd-${{ github.ref }}
  cancel-in-progress: true

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: Set up Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ env.NODE_VERSION }}

    - name: npm install, build, and test
      run: |
        echo npm install
        echo npm run build --if-present
        echo npm run test --if-present

    - name: Upload artifact for deployment job
      uses: actions/upload-artifact@v4
      with:
        name: node-app
        path: .

  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    needs: build

    environment:
      name: PROD
      url: ${{ steps.deploy-to-webapp.outputs.webapp-url }}

    steps:
    
    # Add here the download-artifact step
    - name: Download artifact from build job
      uses: actions/download-artifact@v4
      with:
        name: node-app

    - name: Deploy to Prod
      if: ${{ success() }}
      run: echo "Specific deploy steps..."

    - name: 'Deploy to Azure WebApp'
      id: deploy-to-webapp 
      uses: azure/webapps-deploy@v3
      continue-on-error: true
      with:
        app-name: ${{ env.AZURE_WEBAPP_NAME }}
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        package: ${{ env.AZURE_WEBAPP_PACKAGE_PATH }}

```
</details>
