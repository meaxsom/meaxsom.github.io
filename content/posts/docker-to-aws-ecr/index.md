+++
author = "Matthew E. Axsom"
title = 'Navigating the AWS ECR Waters'
date = 2024-03-28T20:47:00Z
description = "A Journey Through Docker Image Pushes and IAM Policy Trials"
tags = [
    "AWS",
    "ECR",
    "IAM",
    "Docker",
    "Neo4J"
]
categories = [
    "Skill Development",
]
series = [
    "ECS Odyssey"
]

toc = false
+++

Recently I finished a [Python project](https://github.com/meaxsom/govdocs) that involved [ETL'ing](https://en.wikipedia.org/wiki/Extract,_transform,_load) some US Government public JSON and XML formatted document about banking regulations into a [Neo4J](https://neo4j.com/) database. I opted to create a [Docker](https://www.docker.com/) container to host the development and testing of the Python scripts and the Neo4J database instead of installing them directly on my system. Initially I tried to use a [devcontainer](https://code.visualstudio.com/docs/devcontainers/containers) since I was using VS code, but couldn't quite get it to work - even with a working Docker image. I need to improve my devcontainer "foo" a bit.


After the development was completed, the project seemed like a good candiate to experiment with some AWS resources - in particular:

- Elastic Container Registry (ECR)
- Elastic Container Service (ECS)
- Fargate
- Elastic Block Storage (EBS)
- Simple Storage Service (S3)
- AWS Lambda

and perhaps some others. My idea is to run an "on demand" workload by:

- Dropping a new, similarly formatted XML and/or JSON file into an S3 bucket
- Launching ECS/Fargate w/Neo4J image based on a S3 "trigger"
- Attach ECS/Fargate to image to EBS to act as attached database storage for ECS/Fargate
- Use Lambda to process data and add it to Neo4J
- Tear the whole thing down when complete, but retain the db in EBS

I have no idea if all that's possible or even an optimal way to go about it but want to see how far I can get. 

First step: Push the Docker image I used for development into AWS ECR. You can read more about [Container Registries](https://www.mirantis.com/cloud-native-concepts/understanding-containers/what-is-a-container-registry/) if you need some backgroud.

One would think that would be an easy first thing. The docker image was already built and running locally so really all that remained was to upload/push it. Ug - IAM permissions killed me! But that's why we're here - to learn the process.

First decision was a "public" or "private" ECR - I opted for *public* (remember that) since that path seemed to give me more "free" things. Found a "how to" for pushing up an image to [ECR at AWS](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html#cli-authenticate-registry) but was quickly defeated by IAM permissions. I used a previously created "developer" user/group but it wasn't set up with "ECR" policies. Suffice to say I was in IAM "hell" for several hours.

To save pain the next time, this is what I ended up doing:

- Created a ECR "public" repository for the images using my "admin" user via the AWS console instead of the AWS CLI

- Created an "inline" group permission policy for just [`ecr-public:GetAuthorizationToken`](https://docs.aws.amazon.com/AmazonECR/latest/APIReference/API_GetAuthorizationToken.html). This generates an authorization token used with the AWS CLI. Note the `ecr-public` prefix. There's an entire parallel set of permission policies that deal with "private" ECR - they labeled with just the `ecr` prefix are a superset of the `private` policies. This tripped me up for quite a while. I gave it access to all resources:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Action": [
                "ecr-public:GetAuthorizationToken"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

- Created another "inline" group permission policy for obtaining a ["BearerToken"](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_bearer.html) that ECR needs as part of user authentication and authorization:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "sts:GetServiceBearerToken",
            "Resource": "*"
        }
    ]
}
```

- Finally I created a set of specific public ECR group permission policies that take care of the non-authentication/authorization operations that are needed to process a Docker image `push`. Again, this is an "inline" policy:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecr-public:BatchCheckLayerAvailability",
                "ecr-public:CompleteLayerUpload",
                "ecr-public:DescribeImages",
                "ecr-public:DescribeRepositories",
                "ecr-public:InitiateLayerUpload",
                "ecr-public:PutImage",
                "ecr-public:UploadLayerPart"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

All those ended up being an "iterative" process to execute the following sequence commands that does the actual "pushing" process:

```
docker tag {your-docker-image-name} public.ecr.aws/{your-registry}/{your-repository-name}`
```

which simply "tags" an existing docker image on your system for upload with the URI of the ECR where your image will reside. The URI is displayed in the ECR admin area of the AWS console.

![ECR Repo](/images/posts/docker-to-aws-ecr/ecr-repo.png)


```
aws ecr-public get-login-password --region {your-region} | docker login --username AWS --password-stdin public.ecr.aws/{your-registry}/{your-repository-name}`
```

The above uses the [AWS CLI](https://aws.amazon.com/cli/) on your system to create a session with all the above permission policies to access the ECR and allow the docker upload/build of the image. That will have to be re-run if permissions are changed. I had to do that multiple times as I was jiggering group permission policies in IAM. That assumes the [AWS CLI was configured](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html) at some point.


And then - finally - the command for the real objective:

```
docker push public.ecr.aws/{your-registry}/{your-repository-name}`
```

which "pushes" the image definition up to ECR, builds the image, and makes it available - well at least visible in the ECR console.

![ECR Image](/images/posts/docker-to-aws-ecr/ECR-neo-python.png)

That "learn by doing" process took about 3 hours to unravel, including a fair amount of Google'ing. I will put in my pitch for using AI instead. [Phind](https://www.phind.com) was - again - a god-send for getting me "unstuck" at a couple of major points with IAM settings.

Now that the container image is in ECR, we'll move on to understanding how to launch it. 

Stay tuned...