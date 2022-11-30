# Running tests inside GitHub actions
Goal of this chapter is to run the angular tests when a pull request is created or updated.
For this you are going to create a new workflow.

## Steps of the workflow
For running angular tests inside the workflow, you need to follow these steps:
1. Run a container
2. Checkout the code
3. Install Node.js
4. Install dependencies
5. Run the tests

## Create the workflow
Create a file named `test-on-pullrequest.yml`.

## Run a container
Add the following code to your `test-on-pullrequest.yml`.
It will run the workflow on (re-)opening and synchronizing a pull request.
It will then create a job called `Run Angular tests` running inside an ubuntu container.
```yml
name: Run Angular tests

on:
  pull_request_target:
    types: [opened, synchronize, reopened]

jobs:
  docs:
    name: 'Run Angular tests'
    runs-on: ubuntu-latest
```

## Checkout the code
To checkout the repository you use the action `actions/checkout@v3` ([Find the repository on GitHub](https://github.com/actions/checkout)).
It uses a parameter `with` to define which commit should be checked out.
`${{ github.event.pull_request.head.sha }}` gets the head of the current pull request.
```yml
    steps:
      - name: '☁️ Checkout repository'
        uses: actions/checkout@v3
        with:
          ref: ${{ github.event.pull_request.head.sha }}
```

## Install Node.js
By using `actions/setup-node@v3` ([Find the repository on GitHub](https://github.com/actions/setup-node)),
you can easily set up Node.js, so you will be able to run Angular tests.
```yml
      - name: '⚙️ Use Node.js'
        uses: actions/setup-node@v3
        with:
          check-latest: true
          cache: 'npm'
```

## Install dependencies
It is also possible to run custom code for example `npm install`.
By adding the following you will be able to install dependencies.
```yml
      - name: '⛓️ Install dependencies'
        run: npm ci --no-optional --no-audit --prefer-offline --progress=false
```

## Run the tests
To run the tests you need to call `npm test`. You will also do this by defining a new step with the `run` property.
```yml
      - name: 'Test'
        run: npm test
```

## Trigger the workflow
To trigger the workflow you now need to create a new pull request.
To do so, create a new branch, do some changes and push it. Now create a pull request.
This should start a new build which you can find in the tab `Actions`.  

![Running GitHub Actions](assets/running-actions.png)

## Branch protection / PR rules
Without any further settings it will be possible to merge the pull request with or without a successful workflow run.
To make the successful run a condition to being able to merge a pull request you need to set up branch protection.  
Go to your repository &rarr; Settings &rarr; Branches &rarr; Branch protection rules &rarr; Add rule  

![](assets/branch-protection.png)  

You can protect all your branches by simply adding `*` (or another pattern) as branch name pattern.
Then check the option `Require status checks to pass before merging`.

![](assets/add-branch-protection.png)

After this you need to select which are the required status checks. You can add your workflow in the textbox below.
It will no longer be possible to merge pull requests
without a successful workflow run.  

![](assets/define-required-status-checks.png)

## The workflow
```yml
name: Run Angular tests

on:
  pull_request_target:
    types: [opened, synchronize, reopened]

jobs:
  docs:
    name: 'Run Angular tests'
    runs-on: ubuntu-latest

    steps:
      - name: '☁️ Checkout repository'
        uses: actions/checkout@v3
        with:
          ref: ${{ github.event.pull_request.head.sha }}

      - name: '⚙️ Use Node.js'
        uses: actions/setup-node@v3
        with:
          check-latest: true
          cache: 'npm'

      - name: '⛓️ Install dependencies'
        run: npm ci --no-optional --no-audit --prefer-offline --progress=false

      - name: 'Test'
        run: npm test
```

## Next step (Deployment to GitHub pages)
[Continue with the tutorial](deployment-to-github-pages.md)
