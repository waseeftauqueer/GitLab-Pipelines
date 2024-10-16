# GitLab CI/CD Pipeline for Web Application

This repository contains a GitLab CI/CD pipeline for building, testing, and deploying a web application using Node.js and AWS S3. The pipeline consists of multiple stages to ensure a smooth deployment process.

## Pipeline Stages

1. **Build**: Installs dependencies, lints the code, runs tests, builds the application, and generates a version file.
2. **Test**: Serves the built application locally and verifies that the application is running correctly.
3. **Deploy Staging**: Deploys the application to the staging environment using AWS S3.
4. **Deploy Production**: Manually deploys the application to the production environment using AWS S3.

## Prerequisites

- Ensure you have a GitLab account and access to GitLab CI/CD features.
- Set up an AWS account and create an S3 bucket to host your application.
- Configure your GitLab repository with the following environment variables:
  - `AWS_S3_BUCKET`: Your S3 bucket name.
  - `CI_ENVIRONMENT_URL`: The URL of your deployed application environment.

## Pipeline Configuration

The pipeline is defined in a `.gitlab-ci.yml` file. Below are the key sections of the configuration:

### Stages

The pipeline consists of the following stages:

```yaml
stages:
  - build
  - test
  - deploy
  - deploy_staging
  - deploy_production
```

### Build Stage

In the `build_website` job, the application dependencies are installed, the code is linted, tests are run, and the application is built. The app version is recorded in a `version.html` file.

```yaml
build_website:
  image: node:16-alpine
  stage: build
  script:
    - |
      yarn install
      yarn lint
      yarn test
      yarn build
      echo "App Version: $APP_VERSION" > build/version.html
  artifacts:
    paths:
      - build
```

### Test Stage

The `test_website` job serves the built application and checks if it is accessible.

```yaml
test_website:
  image: node:16-alpine
  stage: test
  script:
    - yarn global add serve
    - apk add curl
    - serve -s build &
    - sleep 10
    - curl http://localhost:3000 | grep "React App"
```

### Deployment Stages

The deployment jobs extend the `.deploy` template, which handles the AWS S3 synchronization and version verification.

#### Pre-Production (Staging)

Deploys the application to the staging environment.

```yaml
pre_production:
  stage: deploy_staging
  environment:
    name: Staging
  extends: .deploy
```

#### Production Deployment

Manually deploys the application to the production environment.

```yaml
deploy_to_s3:
  stage: deploy_production
  when: manual
  environment:
    name: Production
  extends: .deploy
```

## Usage

1. **Push Changes**: Push your changes to the default branch (usually `main` or `master`) to trigger the pipeline.
2. **Monitor Pipeline**: Go to your GitLab repository's CI/CD section to monitor the pipeline's progress.
3. **Manual Deployment**: After the staging deployment, manually trigger the production deployment by clicking the "Play" button in the GitLab CI/CD interface.

## Conclusion

This CI/CD pipeline automates the process of building, testing, and deploying your web application to AWS S3. By following the steps outlined in this README, you can efficiently manage your deployment workflow.

---
