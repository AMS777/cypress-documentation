---
title: GitHub Actions
---

{% note info %}
# {% fa fa-graduation-cap %} What you'll learn

- How to GitHub Actions for Cypress Tests

{% endnote %}

# What is GitHub Actions

{% url "GitHub Actions" https://github.com/features/actions %} provide a way to "automate, customize, and execute your software development workflows" within your GitHub repository.  Detailed documentation is available in the {% url "GitHub Action Documentation" https://docs.github.com/en/actions %}.

# What is the Cypress GitHub Action

GitHub Actions can be packaged and shared through GitHub itself.  GitHub maintains many, such as the {% url "checkout" https://github.com/marketplace/actions/checkout %} and {% url "cache" https://github.com/marketplace/actions/cache %} actions used below.

The Cypress team maintains a {% url "Cypress GitHub Action" https://github.com/marketplace/actions/cypress-io %} for running Cypress end-to-end tests. This action provides npm installation, custom caching, additional configuration options and simplifies setup of advanced workflows with Cypress in the GitHub Actions platform.

# Simple Setup

The example below shows the simplest setup and job using the Cypress GitHub Action to run end-to-end tests with Cypress and Electron.

On push to this repository, this job will run a GitHub-hosted runner for Ubuntu Linux.

It uses the checkout action - https://github.com/marketplace/actions/checkout

Finally, our Cypress GitHub Action will be used to install dependencies and run the tests against Electron.

```md
name: Cypress Tests
on: [push]
jobs:
  cypress-run:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      # Install NPM dependencies, cache them correctly
      # and run all Cypress tests
      - name: Cypress run
        uses: cypress-io/github-action@v2
```

# Testing against Chrome and Firefox with Cypress Docker Images

GitHub Actions provides the option to specify a container image for the job.

Cypress offers various {% url "Docker Images" https://github.com/cypress-io/cypress-docker-images %} for running Cypress locally and in CI.

Below we add the `container` attribute for a container built with Google Chrome and Firefox and in our

```md
name: Cypress Tests using Cypress Docker Image

on: [push]

jobs:
  cypress-run:
    runs-on: ubuntu-latest
    container: cypress/browsers:node12.18.3-chrome87-ff82
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      # Install NPM dependencies, cache them correctly
      # and run all Cypress tests
      - name: Cypress run
        uses: cypress-io/github-action@v2
        with:
          # Specify Browser since container image is compile with Firefox
          browser: firefox
```

# Caching Dependencies and Build Artifacts

Dependencies and build artifacts maybe cache between jobs using the {% url "cache" https://github.com/marketplace/actions/cache %} GitHub Action.

The job below includes a cache of `node_modules`, the Cypress binary in `~/.cache/Cypress` and the `build` directory.  In addition, the `build` attribute is added to the Cypress GitHub Action to generate the build artifacts prior to the test run.

```md
name: Cypress Tests with Dependency and Artifact Caching

on: [push]

jobs:
  cypress-run:
    runs-on: ubuntu-latest
    container: cypress/browsers:node12.18.3-chrome87-ff82
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      
      - uses: actions/cache@v2
        id: yarn-build-cache
        with:
          path: |
            node_modules
            ~/.cache/Cypress
            build
          key: ${{ runner.os }}-node_modules-build-${{ hashFiles('**/yarn.lock') }}
          restore-keys: |
            ${{ runner.os }}-node_modules-build-

      # Install NPM dependencies, cache them correctly
      # and run all Cypress tests
      - name: Cypress run
        uses: cypress-io/github-action@v2
        with:
          # Specify Browser since container image is compile with Firefox
          browser: firefox
          build: yarn build
      
```


{% note info %}
#### {% fa fa-graduation-cap %} Real World Example

A complete CI workflow against multiple browsers, viewports and operating systems is available in the {% url "Real World App (RWA)" https://github.com/cypress-io/cypress-realworld-app %}.

Clone the {% fa fa-github %} {% url "Real World App (RWA)" https://github.com/cypress-io/cypress-realworld-app %} and refer to the {% url ".github/workflows/main.yml" https://github.com/cypress-io/cypress-realworld-app/blob/develop/github/workflows/main.yml %} file.
{% endnote %}


# Parallelization

The {% url "Cypress Dashboard" 'dashboard' %} offers the ability to {% url 'parallelize and group test runs' parallelization %} along with additional insights and {% url "analytics" analytics %} for Cypress tests.

GitHub Actions offers a {% url "matrix strategy" https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idstrategymatrix %} of different job configurations for a single job definition.

## Installation Job

The separation of installation from run is necessary when running parallel jobs.  It allows for reuse of various build steps aided by caching.

First, we'll define the `install` step that will be used by the worker jobs defined in the matrix strategy.

Notice that we pass `runTests: false` to the Cypress GitHub Action to instruct it to only install dependencies without running the tests.

The {% url "cache" https://github.com/marketplace/actions/cache %} GitHub Action is included and will save the state of the `node_modules`, `~/.cache/Cypress` and `build` directories for the worker jobs.

```md
name: Cypress Tests with installation job

on: [push]

jobs:
  install:
    runs-on: ubuntu-latest
    container: cypress/browsers:node12.18.3-chrome87-ff82
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - uses: actions/cache@v2
        id: yarn-and-build-cache
        with:
          path: |
            ~/.cache/Cypress
            build
            node_modules
          key: ${{ runner.os }}-node_modules-build-${{ hashFiles('**/yarn.lock') }}
          restore-keys: |
            ${{ runner.os }}-node_modules-build-

      - name: Cypress install
        uses: cypress-io/github-action@v2
        with:
          runTests: false
          build: yarn build
```

## Caching for Parallel Jobs

## Organizing by Jobs with Grouping

Cypress Groups can be used with the Cypress Dashboard to
help and jobs vary by their config

# Debug End-to-End Test Runs with the Cypress Dashboard

Talk about debugging failures with the Cypress Dashboard in a general way.