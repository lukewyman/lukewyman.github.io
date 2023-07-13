---
title: 'DynamoDB to Opensearch'
subtitle: 'Real-time updates to Opensearch indices'
date: 2023-07-01 00:02:00
description: "How to maintain an OpenSearch domain when the live, source-of-truth database is receiving CRUD operations using:"
features:
  - a DynamoDB table as the source of truth database
  - a DynamoDB change stream as the event source
  - an AWS Lambda to process the record changes
  - an OpenSearch domain
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

### The REST API and CRUD Operations
The scenario of a transactional application performing CRUD (Create, Read, Update and Delete) operations against a source of truth database is realized with a simple, Serverless application compose of a REST API for the CRUD functionality and a DynamoDB table as the database. 

The REST API is implemented with Amazon API Gateway and OpenAPI. There is no "app" as such. The Rest API is directly integrated with the DynamoDB table using API Gateway exensions. This is reflects the agile way of starting out simple, only bringing in Lambdas in the case that there is more complex processing logic required between the API and database.

For example, the POST method integrated directly with the DynamoDB table using Velocity Template Language and API Gateway extensions in the OpenAPI spec as follows:

```yaml
/books:
    post: 
      x-amazon-apigateway-integration:
        type: aws
        uri: "arn:aws:apigateway:${aws_region}:dynamodb:action/PutItem"
        httpMethod: POST 
        passthroughBehavior: when_no_match
        credentials: ${apigw_role_arn}
        requestTemplates:
          application/json: |
            #set($inputRoot = $input.path('$'))
            {
              "TableName": "${books_table_name}",
              "Item": {
                "book_id": { "S": "$context.requestId" },
                "title": { "S": "$inputRoot.title" },
                "author": { "S": "$inputRoot.author" }
              }
            }
        responses:
          default:
            statusCode: 200 
            responseTemplates:
              application/json: |
                {
                  "message": "New book created."
                  "book_id": "$context.requestId"
                }
```

### Processing the Record Changes in DyanamoDB

