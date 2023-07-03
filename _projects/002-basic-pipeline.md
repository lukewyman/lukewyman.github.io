---
title: 'Basic CI Pipeline'
subtitle: 'Continuous Integration using GitHub Actions and Terraform'
date: 2023-07-01 00:01:00
description: With a serverless application as the subject of deployment, this project demonstrates a basic continuous integraton pipeline using GitHub Actions and Terraform. Features include automation of Terraform deployment, integration tests, and output written to the pull request.
featured_image: '/images/demo/demo-square.jpg'
---


## Basic Continuous Integraton Pipeline

[source code](https://github.com/lukewyman/dynamodb-to-opensearch)

With a serverless application as the subject of deployment, this project demonstrates a basic continuous integraton pipeline using GitHub Actions and Terraform. Features include automation of Terraform deployment, integration tests, and output written to the pull request.

### Tech Stack
- GitHub Actions
- AW
- Terraform
- Python / pytest
- Amazon API Gateway with OpenAPI and API Gateway extentions
- Amazon DynamoDB

### The Application Under Deployment

The application itself in this project is simple, so that the CI pipeline is the focus. To that end, the application is a serverless application composed of API Gateway and DynamoDB which are integrated with OpenAPI and API Gateway extensions. 

### The Pipeline Steps

- check out the code using `actions/checkout@v3`
- configure AWS credentials with `aws-actions/configure-aws-credentials@v2` so that 

### Security Hardening with AWS IAM and OpenID Connect

### Integration Testing

### Writing Pipeline Results to the Pull Request