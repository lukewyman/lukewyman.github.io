---
title: 'Basic CI / CD Pipeline'
subtitle: 'Continuous Integration using GitHub Actions and Terraform'
date: 2023-07-01 00:01:00
description: "With a serverless application as the subject of deployment, this project demonstrates a basic continuous integraton pipeline, and features:"
features:
  - a Serverless application implemented with Terraform
  - a GitHub Actions CI/CD pipeline
  - integration tests
  - pipeline output written to the pull request
featured_image: '/images/demo/demo-square.jpg'
---


## Basic Continuous Integraton Pipeline

[source code](https://github.com/lukewyman/dynamodb-to-opensearch)

With a serverless application as the subject of deployment, this project demonstrates a basic continuous integraton pipeline using GitHub Actions and Terraform. Features include automation of Terraform deployment, integration tests, and output written to the pull request.

### Tech Stack
- GitHub Actions
- Terraform
- Python / pytest
- Amazon API Gateway with OpenAPI and API Gateway extentions
- Amazon DynamoDB

### The Application Under Deployment

The application itself in this project is simple, so that the CI pipeline is the focus. To that end, the application is a serverless application composed of API Gateway and DynamoDB which are integrated with OpenAPI and API Gateway extensions. This provides the bare minimum of CRUD endpoints for integration tests to demonstrate a GitHub Actions workflow.

### Overview of Pipeline Steps

- check out the code using `actions/checkout@v3`.
- configure AWS credentials with `aws-actions/configure-aws-credentials@v2`.
- setup Terraform with `hashicorp/setup-terraform@v2`.
- check the Terraform formatting.
- initialize the Terraform workspace in a `test` environment.
- validate the Terraform code.
- deploy to the `test` environment with `terraform apply`.
- extract the Terraform outputs to a json file so that they are available for the integration tests to reference.
- setup Python on the GitHub runner.
- install Python dependencies.
- run a Python script to seed the database with test data.
- run the integration tests with pytest.
- tear down the test environment with `terraform destroy`.
- run `terraform plan` in the UAT environment.
- update the pull request with the terraform validation and integration test results, as well as the UAT Terraform plan.

### Security Hardening with AWS IAM and OpenID Connect

Rather than implement security the quick-and-easy way by adding the AWS access keys as secrets in GitHub, this project demonstrates the more robust approach of configuring an OIDC approach with AWS IAM:

1. Configure an OIDC provider in AWS IAM.
2. Configure a Role in AWS IAM and add a trust relationship policy that allows access for the GitHub project:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::<account number>:oidc-provider/token.actions.githubusercontent.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
                },
                "StringLike": {
                    "token.actions.githubusercontent.com:sub": [
                        "repo:lukewyman/actions-and-terraform-pipeline:*"
                    ]
                }
            }
        }
    ]
}
```
3. Add permissions to the GitHub Action workflow to access the OIDC token:
```
jobs:
  test_and_deploy:
    name: "Test & Deploy to UAT"    
    runs-on: ubuntu-22.04
    defaults:
      run:
        working-directory: app
    permissions:
      pull-requests: write 
      id-token: write
      contents: read
```
4. Add a step to the job that uses the `aws-credentials/configure-aws-credentials` action and configure the step to use the role:
```
- name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v2 
        with:
          role-to-assume: arn:aws:iam::<account number>:role/github-actions
          aws-region: us-west-2
```

### Integration Testing

