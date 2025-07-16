# Building Docker Images and ECS Rolling Deployment with AWS CodePipeline

## To create a CodeCommit repository

AWS > CodeCommit > To create a new repo and upload the below files

```
https://github.com/kohlidevops/DevOpsWithAWS/blob/main/my-angular-project_final.zip
```

To update the code file into the repo using local machine

```
git init
git add .
git commit -am "my first commit"
git remote add origin https://git-codecommit.ap-south-1.amazonaws.com/v1/repos/mywebrepo
git push origin main
```

## To generate the personal access token in Docker Hub

Login > Docker Hub > Generate the Personal Access Token with Read and Write access

## To create a CodeBuild Project

AWS > CodeBuild > Create a new Project

```
Project name > myweb-project
Project type > Default
Source > Provider > CodeCommit
Repository > mywebrepo
Branch > main
Provisioned model > On-demand
Environment image > Managed image
Compute > EC2
Running mode > Container
Operating system > Amaozon Linux
Runtimes > Standard
Image > aws/codebuild/amazonlinux-x86_64-standard:5.0
Image version > Always use the latest image
Service role > New
Buildspec > Use a buildspec file
Create a build project
```

## To create a CodePipeline

AWS > CodePipeline > Create a new Pipeline

```
Pipeline name > myweb-pipeline
Execution mode > Queued
Service role > New
Source > Provider > CodeCommit
Repository name > mywebrepo
Branch > main
Build > Other build providers > AWS CodeBuild
Project name > myweb-project (select your project)
Environment variable > Add > DOCKERHUB_TOKEN and Value
Environment variable > Add > DOCKERHUB_USER and Value
Build type > Single build
Input artifacts > SourceArtifact
As of now Skip Test and Deploy stage
Then create a Pipeline
```

The Pipeline has been succeeded!


<img width="728" height="283" alt="image" src="https://github.com/user-attachments/assets/fb62392c-dd55-45d9-ace7-c3f1eedf6a53" />


The Image pushed into the Docker Hub 


<img width="653" height="310" alt="image" src="https://github.com/user-attachments/assets/435ceb4b-d07f-4c8b-8168-f42e5d66e76d" />


## Environment Variables on AWS CodeBuild

1. Plaintext
2. AWS Systems Manager Parameter Store
   Supports encrypted & unencrypted parameters.
   Free for standard (max. 4 KB) parameters.

3. AWS Secrets Manager
   Your secrets are automatically encrypted.
   Supports automatic secret rotation with integrated DBs like Amazon RDS.
   Not free.

## How to set? (Higher to lower precedence)

1. While triggering your build project. CodePipeline action configurations fall into this category.
2. On your CodeBuild project's configuration.
3. In your CodeBuild buildspec files.
4. If you define an environment variable on your buildspec but also provide it on your build project configuration, the build project configuration wins. This is the precedence.
5. You can use the same buildpsec file in multiple build projects, so the build project has more precedence
6. You can also trigger the same build project from different pipelines. So the pipeline has more precedence.

### To create a DockerHub credentials in AWS Secrets Manager

AWS > Secrets Manager > Secret type > Other type of secret

```
key > user and value > latchudevops
key > token and value > ********
secret name > dockerhub
create it
```

### To remove the environment variables from CodeBuild project

AWS > CodePipeline > Choose your pipeline > Edit > CodeBuild Stage > Edit > Remove the environment variables called DOCKERHUB_TOKEN and DOCKERHUB_USER

Save the stage > save the Pipeline

### To add the policy to the CodeBuild project role

Navigate to your CodeBuild project IAM role > Edit > Add in-line policy

```
Service > Secrets manager
Action allowed > Read > GetSecretValue
Resource > Specific > Add ARN > Copy/paste the secrets manager arn
Next > Name it and create a Policy
```

### To update the Buildspec yaml

I have added the docker credentials into the buildspec.yml

```
version: 0.2
env:
  secrets-manager:
    DOCKERHUB_TOKEN: dockerhub:token
    DOCKERHUB_USER: dockerhub:user
phases:
  pre_build:
    commands:
      - echo ${DOCKERHUB_TOKEN} | docker login -u ${DOCKERHUB_USER} --password-stdin
  build:
    commands:
      - docker build -t my-angular-app:latest .
  post_build:
    commands:
      - docker tag my-angular-app:latest ${DOCKERHUB_USER}/my-angular-app:latest
      - docker push ${DOCKERHUB_USER}/my-angular-app:latest
```

The pipeline will trigger automatically once the buildspec updated in codecommit repository.


<img width="620" height="267" alt="image" src="https://github.com/user-attachments/assets/4cc74f96-a827-46ec-869f-dc1b79b5c861" />


### To create a Parameter store as environment variable for CodeBuild project

AWS > SSM > Parameter store > Create parameter

```
Name > /dockerhub/token
Tier > Standard
Type > SecureString
Value > **************
Create parameter

Name > /dockerhub/user
Tier > Standard
Type > SecureString
Value > latchudevops
Create parameter
```


<img width="795" height="263" alt="image" src="https://github.com/user-attachments/assets/ea1a9a76-cc24-4f61-8f30-510a313a294f" />


### To add an IAM policy to CodeBuild project role

Navigate to your CodeBuild project IAM role > Edit > Add in-line policy

```
Service > Systems Manager
Action > Read > GetParameters
Resource > Add ARN > arn:aws:ssm:ap-south-1:1234567890:parameter/dockerhub/*
Name it and create a policy
```

### To update the buildspec file for parameter store

```
version: 0.2
env:
  parameter-store:
    DOCKERHUB_TOKEN: /dockerhub/token
    DOCKERHUB_USER: /dockerhub/user
phases:
  pre_build:
    commands:
      - echo ${DOCKERHUB_TOKEN} | docker login -u ${DOCKERHUB_USER} --password-stdin
  build:
    commands:
      - docker build -t my-angular-app:latest .
  post_build:
    commands:
      - docker tag my-angular-app:latest ${DOCKERHUB_USER}/my-angular-app:latest
      - docker push ${DOCKERHUB_USER}/my-angular-app:latest
```

The pipeline will trigger automatically once the buildspec updated in codecommit repository.


<img width="665" height="281" alt="image" src="https://github.com/user-attachments/assets/770dc7c0-3f8a-4efa-8864-7f6b2c36adb2" />

## Pushing the Docker Image to the AWS ECR


<img width="1089" height="439" alt="image" src="https://github.com/user-attachments/assets/bd27541a-ccca-40e7-baa6-1e1b1e90e9ce" />


### To update the IAM policy to the CodeBuild project role

Navigate to your CodeBuild project IAM role > Edit > Add in-line policy > Choose Json > place the below policy > Name it and create a policy

```
https://github.com/kohlidevops/DevOpsWithAWS/blob/main/codebuild-ecr-policy.json
```

### To add the environment variable into the CodeBuild environment

AWS > CodeBuild > Choose your project > Edit

Add Environment variables > Name > AWS_ACCOUNT_ID and Value > 1234567890

Then update the project


### To create a ECR repo

To create a private repository in ECR


<img width="643" height="162" alt="image" src="https://github.com/user-attachments/assets/f4a6e552-2268-407c-9586-4da088f51a9f" />


### To update buildspec file to pushing image to the ECR

```
version: 0.2
env:
  variables:
    ECR_REPO_NAME: myweb
  parameter-store:
    DOCKERHUB_TOKEN: /dockerhub/token
    DOCKERHUB_USER: /dockerhub/user
phases:
  pre_build:
    commands:
      # Docker Hub login
      - echo ${DOCKERHUB_TOKEN} | docker login -u ${DOCKERHUB_USER} --password-stdin 
      
      # ECR login
      - ECR_MAIN_URI="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
      - aws ecr get-login-password --region ${AWS_REGION} | docker login -u AWS --password-stdin ${ECR_MAIN_URI}

      - ECR_IMAGE_URI="${ECR_MAIN_URI}/${ECR_REPO_NAME}:latest"
  build:
    commands:
      - docker build -t my-angular-app:latest .
  post_build:
    commands:
      - docker tag my-angular-app:latest ${ECR_IMAGE_URI}
      - docker push ${ECR_IMAGE_URI}
```

The pipeline will trigger automatically once the buildspec updated in codecommit repository.


<img width="743" height="257" alt="image" src="https://github.com/user-attachments/assets/15bde7b6-9b92-4441-8b42-c1b4006788ba" />


The Docker image successfully pushed into the AWS ECR repo


<img width="643" height="163" alt="image" src="https://github.com/user-attachments/assets/9bb84798-005a-404d-b06b-2d5dd017b05a" />


## To create a Task definition

AWS > ECS > Task definition > Create a new Task definition

```
Task definition family > name > myweb
Infrastructure requirements > Launch type > AWS Fargate and Amazon EC2 instances
Operating system/Architecture > Linux/X86_64
CPU > .5 vCPU and Memory > 1 GB
Container > Name > myweb
Image URI > 1234567890.dkr.ecr.ap-south-1.amazonaws.com/myweb:latest
Container port > 80
Protocol > TCP
Then create a Task definition
```

<img width="629" height="160" alt="image" src="https://github.com/user-attachments/assets/04ca84d7-f5cc-44d1-a541-1067d17e1087" />


## To create a ECS Cluster

AWS > ECS > Cluster > Create a new Cluster

```
Cluster Name > my-cluster
Infrastructure > AWS Fargate Serverless
Choose > Amazon EC2 instances too
Autoscaling > Create a new ASG
Provisioning model > On-demand
EC2 instance type > t3.micro
EC2 instance role > Create a new role
Min > 0 and Max > 5
SSH > None
Networking > Choose your VPC and its public subnets
Security group > Use an existing SG
Create a Cluster
```

The cluster has been created


<img width="765" height="208" alt="image" src="https://github.com/user-attachments/assets/d5079ab1-6571-4f4f-a8fb-e2203b4b9609" />

If you check the Infrastructure option inside your cluster, then there are 3 capacity providers added such as FARGATE, FARGATE_SPOT and EC2 ASG group.

## To Cretate an ECS Service on Fargate for Rolling Deployment

AWS > ECS > Choose your Cluster > Select > Service > Create a Service

```
Task definition family > myweb
Task definition revision > 1 $LATEST
Service name > my-rolling-service
Compute configuration > Compute option > Capacity provider strategy > Use custom (Advanced)
Capacity provider > FARGATE
Platform version > LATEST
Deployment configuration > Service type > Replica
Desired task > 2
Health check grace period > 0
Deployment option > Deployment type > Rolling update
Min running tasks > 100
Max running tasks > 200
Deployment failure detection > Unselect > Use the Amazon ECS deployment circuit breaker and Rollback on failure
Networking > Choose your VPC and subnets
Secutiy group > Use an exisiting SG
Public IP > Turned on
Loadbalancing > Use Load balancing
VPC > Choose your VPC
Type > ALB
my container > myweb 80 : 80
Application Load Balancer > create a new ALB
Load balancer name > my-rolling-service-alb
Listener > Create new listener > Port 80 | Protocol HTTP
Target group > Create a new TG
TG name > my-rolling-service-target-group
Protocol > HTTP
Port > 80
Degregistration delay > 300
Heaṭch check protocol > HTTP
Health check path > /
Create a Service
```

The Service is created with what we have configured


<img width="656" height="239" alt="image" src="https://github.com/user-attachments/assets/9d1a0354-7524-4787-9492-8f9e6f2d04d9" />

To create a new security group for ALB with All traffic allow anywhere and associate with ALB

To allow All traffic anywhere to ASG associated security group

To change the Deregistration delay from 300 to 30 seconds - Because our target group will wait at least 5 mins to drain the existing connection on the ECS tasks before deregistering them.

If you hit the ALB URL then you should can access the site


<img width="835" height="386" alt="image" src="https://github.com/user-attachments/assets/2c1879a5-157f-4439-8413-8d511184a6fc" />









