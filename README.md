# github-actions-module-template

## How to use module
  Create a new module using this template file. To call the templated module in a new repo add it in your github actions workflow as follows:

    steps:
    - name: Run Composite Module
      uses: <ORG>/<REPO_NAME>@main

### action.yaml

  This file is where your module will call the actions that other repos will use. Any needed changes are outlined as comments underneath the action.yaml