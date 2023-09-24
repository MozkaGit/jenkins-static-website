# Jenkins Static Website

This repository contains the necessary configuration and code to deploy a static website on AWS Elastic Beanstalk using Jenkins pipeline.

[![Build Status](https://237d-82-66-124-175.ngrok-free.app/buildStatus/icon?job=Mini+projet)](http://192.168.56.14:8080/job/Mini%20projet/)

## Prerequisites

1. Git installed on your local machine.
2. An AWS account with access keys and permissions to deploy resources.
3. Jenkins Setup:
    - Environment variables
    - Secrets
    - Plugins
4. GitHub Setup:
    - GitHub repository created
    - Webhook configured to your Jenkins server

## Usage

1. Triggering the Pipeline:

- The pipeline will be triggered through your webhook.
- For pull requests, the `Push image in review and deploy it` stage will be executed.

2. Pipeline Stages:

- `Build Image`: The Docker image is built using the provided Dockerfile.
- `Run Container`: A Docker container is spun up based on the built image.
- `Test Container`: The running container is tested using curl.
- `Artifact`: Previous containers are removed to clean up.
- `Push Image to Docker Hub`: The built image is tagged and pushed to Docker Hub.
- `Push Image in Review and Deploy`: This stage is specifically for change requests like pull.
- `Push Image in Staging and Deploy`: This stage push the image in staging environment.
- `Push Image in Production and Deploy`: This stage push the image in production environment.
- `Post`: A notification is sent to Slack indicating if the pipeline succeed or not.
