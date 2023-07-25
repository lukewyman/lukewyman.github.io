---
title: 'The Karaoke Suite'
subtitle: 'A non-trivial, real-world use case architected across a suite of projects.'
date: 2023-07-01 00:03:00
description: "To showcase my DevOps and Application Development skills, this suite of projects features:"
features:
  - "Application Development with FastAPI and Python"
  - "Databases: Postgresql, DocumentDB/MongoDB, DynamoDB and OpenSearch"
  - "Infrastructure as Code with Terraform, Terragrunt and AWS services"
  - "CI/CD Pipelines using GitHub Actions"
featured_image: '/images/singermic_square.jpg'
  
---

![](/images/singermic_landscape.jpg)


## The Karaoke Suite

In 2019, I started taking singing lessons. I also frequented karaoke events on a regular basis as an outlet. Karaoke today is still in the dark ages. The song database is a set of large, black binders, and song requests are written in pencil on little pieces of paper and turned in to the DJ. After a year of singing, I realized that I had been accumulating user requirements for a karaoke app in the back of my mind. 

Since I was already using AWS Services at work, and developing an interest in DevOps, I arrived at this suite of projects as a way to create a hands-on, learning amusement park for myself. As such, I've moved around between Infrastructure as Code, application development and databases as I've worked on this effort. This is very much a work in progress. Where you find me is where I am now. I will update this site as more work gets completed.

Working on this project has provided me with the opportunity to showcase what I can do:

- Demonstrate my ability to architect a large, non-trivial application across a suite of projects as a way of managing complexity.
- Show my product skills by analyzing a real-world problem and arriving at a software solution.
- There's nothing like hands-on learning. I have enjoyed bouncing from one skill to another, and can now show some versatility.

### Tech Stack

| DevOps         | App Dev                     | Databases            | AWS Services   |
|--------------------------------------------------------------------------------------|
| Terraform      | Python                      | MongoDB              | VPC            |
| Terragrunt     | FastAPI                     | PostgreSQL           | EC2            |
| GitHub Actions | Pytest / TDD                | DynamoDB             | Load Balancing |
| Docker         | motor / mongomock-motor     |                      | ECS / Fargate  |
| Docker Compose | sqlalchemy                  |                      | RDS            |
|                | pynamodb                    |                      | DocumentDB     |

### The Karaoke App 
[source code](https://github.com/lukewyman/karaoke-app)

Since my primary interest of late has been in DevOps, this application was initially intended to be a simple application around which I could build the DevOps to describe and deploy the infrastructure on which the application would run. As of this writing, my first stab at the application is still embedded in the Infrastructure project as an after thought. 

It has occurred to me, that since I know how to build a straightforward application, I should demonstrate that ability with a stand-alone project that does just that. As such, I am in the middle of performing a "surgery" of sorts to separate that out into a pure application-only project (this project, the Karaoke App), and then have the Karaoke App Infrastructure project pull the resulting Docker images, so that it is Infrastructure as Code and deployment only.

#### The Microservices
![](/images/projects/microservices.jpeg)

The Karaoke App, at this point, is composed of three microservices:

- A **song-library microservice** that allows for the storage and retrieval of karaoke song tracks and their metadata, as well as a search feature to search for tracks by song title and artist.
  - **Current functionality:** 
    - A FastAPI / Python application backed by a MongoDB / DocumentDB database for the track metadata. 
    - A Docker Compose setup that allows the application to be run locally for smoke testing.
    - Robust unit tests that test both the data layer and REST api layer with a mongomock-motor mock database.
  - **Planned functionality:**
    - A DocumentDB change stream and AWS Lambda setup that maintains search indices in an OpenSearch domain.
    - The ability to upload the karaoke track media files to an S3 bucket.
<br /><br />
- A **singers microservice** that allows for storage and retreival of singers data. Each user of the app would have a Singer record.
  - **Current functionality:**
    - A FastAPI / Phython application backed by a PostgreSQL database.
    - A Docker compose setup that allows the application to be run locally for smoke testing.
  - **Planned functionality:**
    - Unit tests! Currently, this microservice only has a simple unit test on the health check endpoint so that the microservice can participate in the GitHub Actions workflow matrix when the unit tests are run. I still need to wrap my head around a mock PostgreSQL database strategy that will serve that end.
<br /><br />
- A **rotations microservice**. This microservice is still in the conception stages. I will serve as the work horse microservice when a karaoke event is underway, allowing singers to add themselves to the queue, along with their song choice for their upcoming performance. Here are my thoughts on what this microservice will look like:
  - Rotations will be a FastAPI / Python application backed by a DynamoDB database. 
  - The core design aspect that I'm currently debating, is how to represent the data in DynamoDB database. I'm wrestling with the following two approaches:
    - The first approach would be to store the Queue and Enqueued Singer objects directly with a standard CRUD approach. The upside to this, is it's the fasted way to get to a completed microservice. The downsides are that 1) that I've already demonstrated more than enough CRUD as a skill in the first two microservices, and 2) the CRUD approach doesn't allow for much flexibility for an evolving model, particularly since Rotations is the heart of the application.
    - The second approach would be to store the data as an event log in DynamoDB, as the command side of a CQRS solution with events such as `StartQueue`, `EnqueueSinger`, `RemoveSinger`, etc. The `queue_id` would be the partition key, and the `event_time` would be the sort key. I could then set up the read side of CQRS with an AWS Lambda listening the the DynamoDB change stream and updating the read side views in a PostgreSQL database. The downside is that this would be a lot of work and a longer timeline to a finished product. The upsides are that 1) I would have a blast building this, 2) showing off CQRS as a skill is a good thing, and 3) I have a flexible model whereby I can modify read-side views on the fly. (It might be obvious which way I'm leaning...)


#### The Continuous Integration Pipeline
![](/images/projects/karaoke-app-pipeline.jpeg)

The Karaoke App has a simple continuous integration pipeline that I have mostly completed as of this writing. The building of the three microservices runs as three parallel jobs via a matrix. Each job will checkout the code, run the unit tests, and build the Docker image and push it to the ECR repo. This part is working well at the moment.

The second part, which introduces a bit of a challenge, is for the workflow to write the test and build results to the pull request. The problem with this, is that if the original three jobs write the results in parallel, the steps for each of the jobs will likely be interleaved as they try to write at the same time. The reason this is tricky to solve, is because it's always the last matrix job to finish that gets to write its results for the entire job. 

What I'm working on at the moment, is the use of a pair of GitHub action by cloudposse that address this issue:
- `cloudposse/github-action-matrix-outputs-write` writes the results of the job steps to the job output at the end of each build job.
- `cloudposee/github-action-matrix-outputs-read` will be used by the update pull request job so that it can gather the outputs from the three parallel build jobs and write them to the pull request in an orderly way. 

### Karaoke App Infrastructure
[source code](https://github.com/lukewyman/karaoke-with-infra-and-pipeline)

As mentioned in the Karaoke App description, Karaoke App Infrastructure is the DevOps half of this effort. The other half of the "surgery" that I am working on here, is to remove the application code currently embedded in this project, and have it simply pull the Docker images during deploy time. 

![](/images/projects/karaoke-microservice-infra.jpeg)

### The Base Infrastructure Project
[source code](https://github.com/lukewyman/karaoke-base-infrastructure)

![](/images/projects/base-infrastructure.jpeg)

### Future Projects
- Karaoke Media:
- Karaoke OIDC: 

