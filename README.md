# github-actions-module-template

## How to use module
  Create a new module using this template file. To call the templated module in a new repo add it in your github actions workflow as follows:

    steps:
    - name: Run Composite Module
      uses: <ORG>/<REPO_NAME>@main

## Contents of the module

### .github/workflows/main.yaml
  This file sets the workflow for using pre-commit and will test the workflow. Feel free to move the pre-commit checks into 

  This file has to change two areas: 
  `run-name:`

  `runs-on: codebuild-<RUNNER_NAME>`

  This file will test the workflow. It will test it by calling the workflow on the test branch. Ideally when using this file, you would want to push changes to the test branch and then test the pipeline by creating a pull request.

  This file has to change three areas: 
  `run-name:`

  `runs-on: codebuild-<RUNNER_NAME>`

    steps:
      - name: Run Composite Module
        uses: <ORG>/<REPO_NAME>@test



### action.yaml

  This file is where your module will call the actions that other repos will use. Any changes are outlined as comments underneath the action.yaml