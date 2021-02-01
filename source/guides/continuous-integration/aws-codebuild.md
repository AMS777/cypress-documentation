---
title: AWS CodeBuild
---

{% note info %}
# {% fa fa-graduation-cap %} What you'll learn

- How to use AWS CodeBuild for Cypress Tests

{% endnote %}

With {% url "AWS CodeBuild" https://aws.amazon.com/codebuild/ %} Amazon Web Services (AWS) offers developers "...a fully managed build service that compiles your source code, runs unit tests, and produces artifacts that are ready to deploy" allowing teams to "pay only for the build time you use".

Detailed documentation is available in the {% url "AWS CodeBuild Documentation" https://docs.aws.amazon.com/codebuild/ %}.

# Basic Setup

The example below shows a basic setup and job to use {% url "AWS CodeBuild" https://aws.amazon.com/codebuild/ %} to run end-to-end tests with Cypress and Electron.

{% note info %}
Clone the {% url "Cypress Kitchen Sink" https://github.com/cypress-io/cypress-example-kitchensink %} example and place the following config in `buildspec.yml` in the root of the project.
{% endnote %}

How this buildspec works:

- On push to this repository, this job will run an Amazon Linux container.
- Our code from our GitHub repository.
- Our action runs as follows:
  - Install dependencies (npm/yarn)
  - Start the web server (`npm start:ci`)
  - Run the tests against Electron

```yaml
# buildspec.yml
version: 0.2

phases:
  install:
    runtime-versions:
      nodejs: latest
    commands:
      - npm ci
  pre_build:
    commands:
      - npm run cy:verify
      - npm run cy:info
  build:
    commands:
        - npm run start:ci &
        - npx cypress run --record
```

# Chrome and Firefox with Cypress Docker Images

{% note info %}
As of version 0.2, CodeBuild does not provide a way to specify a custom image for single build configurations.  One way to solve this is using an {% url "AWS CodeBuild build-list strategy" https://docs.aws.amazon.com/codebuild/latest/userguide/batch-build-buildspec.html#build-spec.batch.build-list %}.
{% endnote %}

AWS CodeBuild offers a {% url "build-list strategy" https://docs.aws.amazon.com/codebuild/latest/userguide/batch-build-buildspec.html#build-spec.batch.build-list %} of different job configurations for a single job definition.

The {% url "build-list strategy" https://docs.aws.amazon.com/codebuild/latest/userguide/batch-build-buildspec.html#build-spec.batch.build-list %} offers a way to specify an image hosted on DockerHub or the {% url "Amazon Elastic Container Registry (ECR)" https://aws.amazon.com/ecr/ %}.

Cypress offers various {% url "Docker Images" https://github.com/cypress-io/cypress-docker-images %} for running Cypress locally and in CI, which are built with Google Chrome and Firefox. This allows us to run the tests in Firefox by passing the `--browser firefox` attribute to `cypress run`.

```yaml
# buildspec.yml
version: 0.2

# AWS CodeBuild Batch configuration
# https://docs.aws.amazon.com/codebuild/latest/userguide/batch-build-buildspec.html
# Define build to run using the "cypress/browsers:node12.14.1-chrome85-ff81" image on DockerHub
batch:
  fast-fail: false
  build-list:
    - identifier: cypress-e2e-tests
      env:
        variables:
          IMAGE: cypress/browsers:node12.14.1-chrome85-ff81

phases:
  install:
    runtime-versions:
      nodejs: latest
    commands:
      - npm ci
  pre_build:
    commands:
      - npm run cy:verify
      - npm run cy:info
  build:
    commands:
        - npm run start:ci &
        - npx cypress run --record --browser firefox

```

# Caching Dependencies and Build Artifacts


# Parallelization

The {% url "Cypress Dashboard" 'dashboard' %} offers the ability to {% url 'parallelize and group test runs' parallelization %} along with additional insights and {% url "analytics" analytics %} for Cypress tests.

AWS CodeBuild offers a {% url "build-matrix strategy" https://docs.aws.amazon.com/codebuild/latest/userguide/batch-build-buildspec.html#build-spec.batch.build-matrix  %} of different job configurations for a single job definition.  The {% url "build-matrix strategy" https://docs.aws.amazon.com/codebuild/latest/userguide/batch-build-buildspec.html#build-spec.batch.build-matrix  %} provides an option to specify a container image for the job.

Cypress offers various {% url "Docker Images" https://github.com/cypress-io/cypress-docker-images %} for running Cypress locally and in CI, which are built with Google Chrome and Firefox. This allows us to run the tests in Firefox by passing the `browser: firefox` attribute to `cypress run`.

{% note info %}
The following configuration with `--parallel` and `--record` options to Cypress requires a subscription to the {% url "Cypress Dashboard" https://on.cypress.io/dashboard %}.
{% endnote %}

Below we define a job for "UI Chrome Tests" that will run tests under the `cypress/tests/ui/` path against Google Chrome.

It has a {% url "build-matrix strategy" %} for 5 workers.

Our command records results to the {% url "Cypress Dashboard" https://on.cypress.io/dashboard %} in parallel, using the `CYPRESS_RECORD_KEY` environment variable.

Jobs can be organized by groups and in this job we specify a `group: "UI - Chrome"` to consolidate all runs for these workers in a central location in the {% url "Cypress Dashboard" https://on.cypress.io/dashboard %}.

```yaml
name: Cypress Tests with Install Job and UI Chrome Job x 5
```


{% note info %}
#### {% fa fa-graduation-cap %} Real World Example

A complete CI workflow against multiple browsers, viewports and operating systems is available in the {% url "Real World App (RWA)" https://github.com/cypress-io/cypress-realworld-app %}.

Clone the {% fa fa-github %} {% url "Real World App (RWA)" https://github.com/cypress-io/cypress-realworld-app %} and refer to the {% url ".buildspec.yml" https://github.com/cypress-io/cypress-realworld-app/blob/develop/buildspec.yml %} file.
{% endnote %}

# Debugging with the Cypress Dashboard

Talk about debugging failures with the Cypress Dashboard in a general way.