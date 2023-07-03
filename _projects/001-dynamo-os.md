---
title: 'DynamoDB to Opensearch'
subtitle: 'Real-time updates to Opensearch indices'
date: 2023-07-01 00:02:00
description: How does one maintain an Opensearch domain when the live, source-of-truth database is receiving CRUD operations? This project showcases how to use a DynamoDB change stream and AWS Lambda to update the indices of an Opensearch domain.
featured_image: '/images/projects/dynamo-os-landscape.jpeg'
---

![](/images/projects/dynamo-os-landscape.jpeg)

## DynamoDB to Opensearch

[source code](https://github.com/lukewyman/dynamodb-to-opensearch)

This project demonstrates how to automatically update indices in an Opensearch domain in response to CRUD operations against the source-of-truth database, in this case, DynamoDB. This keeps the Opensearch indices in sync with the DynamoDB table in near real-time.  

### Tech Stack
- Terraform / Terragrunt
- AWS Lambda, written in Python, deployed with Docker
- Amazon API Gateway
- Amazon DynamoDB
- Amazon Opensearch Service

