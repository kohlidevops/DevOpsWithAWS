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


### Adding an Amazon ECS Standard Deploy Action to Your Pipelibne

**What we Build?**


<img width="1027" height="449" alt="image" src="https://github.com/user-attachments/assets/6eb72026-20b9-43db-bbb1-136ffa18dfcd" />


**🔄 Pipeline Flow Breakdown**

**1. 👩‍💻 Developer Pushes Code**

```
A developer commits and pushes code to AWS CodeCommit (Git repository).
This triggers AWS CodePipeline.
```

**2. 📦 CodePipeline - Source Stage**

```
CodePipeline fetches the source artifact from CodeCommit.
This artifact is passed to the next stage for build.
```

**3. 🔧 CodeBuild - Build Stage**

```
AWS CodeBuild compiles the application and builds a Docker image.
It creates a build artifact (e.g., updated task definition).
```

**4. 📤 Push to Amazon ECR**

```
The Docker image is pushed to Amazon Elastic Container Registry (ECR) for storage and future deployment.
```

**5. 📄 Task Definition Update**

```
The new image version is referenced in a new ECS task definition revision (e.g., Rev. 2).
```

**6. 🚀 ECS Deployment**

```
Amazon ECS pulls the updated image from ECR.
A rolling update strategy ensures minimal downtime.
The ECS service is updated to use the new task definition.
```

**How do ECS standard deployments work?**


<img width="488" height="212" alt="image" src="https://github.com/user-attachments/assets/3c497e5d-ebbc-4d46-ad88-a9f013275bde" />


**🧩 Component Breakdown**

**1. 🟧 ECS Service**

```
Manages the deployment and ensures the desired number of tasks are running.
Handles deployments by replacing old tasks with new ones (rolling update).
```

**2. 📦 Task Definition**

```
A blueprint describing how to run containers (e.g., Docker image, port, CPU, memory).
Each task runs a specific revision (e.g., rev 1, rev 2).
```

**3. 🛠️ Deployment 1: rev 1**

```
The current active deployment.
All tasks here use the old version of the container image/config.
```

**4. 🆕 Deployment 2: rev 2**

```
The new version being rolled out.
ECS launches new tasks with updated settings.
```

**5. 🔄 Rolling Deployment**

```
ECS replaces old tasks (rev 1) with new ones (rev 2) gradually.
As tasks become healthy, ECS deregisters old ones and registers new ones with the ALB.
```

**6. 🌐 Application Load Balancer (ALB)**

```
Automatically routes traffic only to healthy tasks.
Ensures users always reach a functional backend, avoiding downtime or broken sessions.
```

### To add the Deploy ECS stage into the Pipeline

AWS > Pipeline > Choose your Pipeline > Edit > Add the Stage after the Build stage

Add the Stage > Stage name > DeployToECS > Add stage > Then Add Action Group

```
Action name > DeployToECS
Action provider > Amazon ECS
Region > Choose your region
Input artifacts > BuildArtifact
Cluster name > Choose your cluster
Service name > Choose your service
Image definitions file > imagedefinitions.json
Done > Done > Save the Pipeline
```
  
So CodeBuild Artifacts doesnot provide the imagedefinitions file needed by our ECS. So we have to update the buildspec file.

To add the ECS_FullAccess policy to the Code Pipeline service role

To update the buildspec file

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

      # Generate image definitions file for ECS
      - printf '[{"name":"myweb","imageUri":"%s"}]' ${ECR_IMAGE_URI} > imagedefinitions.json

artifacts:
  files:
    - imagedefinitions.json
```

To update the version 2.0 in src/app/app.component.html

Then Pipeline should trigger automatically and deployment has been succeeded!


<img width="806" height="290" alt="image" src="https://github.com/user-attachments/assets/c4eed8ba-a5b9-4583-9a87-96175a61cccf" />


If you check with the ALB URL, the new version is updated


<img width="740" height="267" alt="image" src="https://github.com/user-attachments/assets/f5e8e0ba-1964-4aca-b339-37264dfe12a9" />



What if the deployment fails and our ECS service scale-out to more tasks after that. In that case, our new tasks will try to run the new failed Docker image. Because the latest tag start pointing to the failed docker image after the build stage is completed in our pipeline even before the ECS deployment starts. So is there anyway to tag your Docker images differently to prevent this from happening?


To update the buildspec file

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

      - ECR_IMAGE_URI="${ECR_MAIN_URI}/${ECR_REPO_NAME}:${CODEBUILD_RESOLVED_SOURCE_VERSION:0:8}"
  build:
    commands:
      - docker build -t my-angular-app:latest .
  post_build:
    commands:
      - docker tag my-angular-app:latest ${ECR_IMAGE_URI}
      - docker push ${ECR_IMAGE_URI}

      # Generate image definitions file for ECS
      - printf '[{"name":"myweb","imageUri":"%s"}]' ${ECR_IMAGE_URI} > imagedefinitions.json

artifacts:
  files:
    - imagedefinitions.json
```

```
CodeBuild sets CODEBUILD_RESOLVED_SOURCE_VERSION=abc123de90f...
The resulting Docker image tag is :abc123de
You push this to ECR and deploy via ECS or Fargate
If something breaks, you can roll back to the previously tagged image (e.g., :789def45)
```

To update the version 3.0 in src/app/app.component.html

Then Pipeline should trigger automatically and deployment has been succeeded!


<img width="687" height="257" alt="image" src="https://github.com/user-attachments/assets/d007c45d-2f23-46cf-833f-f2aea1cb2aaf" />


The version too updated!


<img width="742" height="269" alt="image" src="https://github.com/user-attachments/assets/4b405a80-2cec-44fc-836c-ebbc6af12c3d" />


If you navigate to ECS > Deployments > Service revisions (latest one) - The tag image URI of the container is same as the commit ID of the CodePipeline and Github.

Container image URI


<img width="514" height="178" alt="image" src="https://github.com/user-attachments/assets/a64c208a-7bb4-4112-aea3-560c4bb0ed2b" />


CodePipeline commit ID


<img width="710" height="227" alt="image" src="https://github.com/user-attachments/assets/1b438c63-7113-404e-8f25-e3248cd2b774" />


If our deployments rollback for some reason, this will prevent our ECS from launching the faulty image using the latest tag. Because each image tag on the task definition revisions will be unique

As of now our Dockerfile look like below

```
# syntax=docker/dockerfile:1

# STAGE 1: Build the Angular project
FROM node:20 AS builder

# Install Angular CLI
RUN npm install -g @angular/cli@17

# Change my working directory to a custom folder created for the project
WORKDIR /my-project

# Copy everything from the current folder (except the ones in .dockerignore) 
# into my working directory on the image
COPY . .

# Install dependencies and build my Angular project
RUN npm install && ng build -c production


# STAGE 2: Build the final deployable image
FROM nginx:1.25

# Allow the HTTP port needed by the Nginx server for connections
EXPOSE 80

# Copy the generated static files from the builder stage
# to the Nginx server's default folder on the image
COPY --from=builder /my-project/dist/my-angular-project /usr/share/nginx/html
```

Here we are using Docker repository to pull the image and we login to the Dockerhub and no longer to send the images to the repo - Why because? To avoid the pulling limit.

Now, we are going to use AWS ECR public repository instead of the Docker repo.


### Using ECR Public Gallery in Your Docker Builds

**To update your Dockerfile like below**

```
# syntax=docker/dockerfile:1

# STAGE 1: Build the Angular project
FROM public.ecr.aws/docker/library/node:20 AS builder

# Install Angular CLI
RUN npm install -g @angular/cli@17

# Change my working directory to a custom folder created for the project
WORKDIR /my-project

# Copy everything from the current folder (except the ones in .dockerignore) 
# into my working directory on the image
COPY . .

# Install dependencies and build my Angular project
RUN npm install && ng build -c production


# STAGE 2: Build the final deployable image
FROM public.ecr.aws/docker/library/nginx:1.25

# Allow the HTTP port needed by the Nginx server for connections
EXPOSE 80

# Copy the generated static files from the builder stage
# to the Nginx server's default folder on the image
COPY --from=builder /my-project/dist/my-angular-project /usr/share/nginx/html
```

**To update the buildspec file (We dont need Docker login and docker related environment variables in buildspec file)**

```
version: 0.2
env:
  variables:
    ECR_REPO_NAME: myweb
phases:
  pre_build:
    commands:
      # ECR Public Gallery login
      - aws ecr-public get-login-password --region us-east-1 | docker login -u AWS --password-stdin public.ecr.aws
      
      # ECR login
      - ECR_MAIN_URI="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
      - aws ecr get-login-password --region ${AWS_REGION} | docker login -u AWS --password-stdin ${ECR_MAIN_URI}

      - ECR_IMAGE_URI="${ECR_MAIN_URI}/${ECR_REPO_NAME}:${CODEBUILD_RESOLVED_SOURCE_VERSION:0:8}"
  build:
    commands:
      - docker build -t my-angular-app:latest .
  post_build:
    commands:
      - docker tag my-angular-app:latest ${ECR_IMAGE_URI}
      - docker push ${ECR_IMAGE_URI}

      # Generate image definitions file for ECS
      - printf '[{"name":"myweb","imageUri":"%s"}]' ${ECR_IMAGE_URI} > imagedefinitions.json

artifacts:
  files:
    - imagedefinitions.json
```

**To add the ECR Read access to the CodeBuild Service role**

CodeBuild > Service role > Attach policy > ElasticContainerRegistryPublicReadOnly > add it

Now CodeBuild can access the ECR public gallery

To update the version 4.0 in src/app/app.component.html

Now commit all the changes to check the CodePipeline - The Pipeline is succeeded


<img width="831" height="344" alt="image" src="https://github.com/user-attachments/assets/084079f5-fa90-41fb-9fef-5bd88d170175" />


As a result, the version has been changed into the application


<img width="871" height="410" alt="image" src="https://github.com/user-attachments/assets/52603ee7-0974-45aa-b98f-905c3f2b70dd" />


### What If Your ECS Rolling Deployment Fails ?

**To update the Dockerfile - I just comment the nginx and added the ubuntu in stage-2 build as it should failed to serve traffic**

```
# syntax=docker/dockerfile:1

# STAGE 1: Build the Angular project
FROM public.ecr.aws/docker/library/node:20 AS builder

# Install Angular CLI
RUN npm install -g @angular/cli@17

# Change my working directory to a custom folder created for the project
WORKDIR /my-project

# Copy everything from the current folder (except the ones in .dockerignore) 
# into my working directory on the image
COPY . .

# Install dependencies and build my Angular project
RUN npm install && ng build -c production


# STAGE 2: Build the final deployable image
# FROM public.ecr.aws/docker/library/nginx:1.25
FROM public.ecr.aws/docker/library/ubuntu

# Allow the HTTP port needed by the Nginx server for connections
EXPOSE 80

# Copy the generated static files from the builder stage
# to the Nginx server's default folder on the image
COPY --from=builder /my-project/dist/my-angular-project /usr/share/nginx/html
```

To update the version 5.0 in src/app/app.component.html

Now commit all the changes to check the CodePipeline - The Pipeline started

For ECS deployment, it is continuously register and deregister the task


<img width="711" height="294" alt="image" src="https://github.com/user-attachments/assets/669ce337-3956-4e76-97d9-f140c3042fd9" />


In ECS > Service > Deployment - we intentionally disabled the Deployment circuit breaker and Deployment rollback. As a result, new tasks couldnot pass the loadbalncer healthchecks. So our ECS rolling deployment starts and stops new tasks.- 

If we enabled the settings - These settings enable you to fail your ECS rolling deployments after reaching a failure threshold and roll them back.

To resolve this first stop and abondon the CodePipeline using Stop execution and update the Dockerfile as below - To uncomment the nginx and comment the ubuntu in stage-2

```
# syntax=docker/dockerfile:1

# STAGE 1: Build the Angular project
FROM public.ecr.aws/docker/library/node:20 AS builder

# Install Angular CLI
RUN npm install -g @angular/cli@17

# Change my working directory to a custom folder created for the project
WORKDIR /my-project

# Copy everything from the current folder (except the ones in .dockerignore) 
# into my working directory on the image
COPY . .

# Install dependencies and build my Angular project
RUN npm install && ng build -c production


# STAGE 2: Build the final deployable image
FROM public.ecr.aws/docker/library/nginx:1.25
# FROM public.ecr.aws/docker/library/ubuntu

# Allow the HTTP port needed by the Nginx server for connections
EXPOSE 80

# Copy the generated static files from the builder stage
# to the Nginx server's default folder on the image
COPY --from=builder /my-project/dist/my-angular-project /usr/share/nginx/html
```

Then commit the changes and to trigger the Pipeline - It is succeeded


<img width="818" height="322" alt="image" src="https://github.com/user-attachments/assets/52c6e8b5-f847-441e-b58f-fe49085214e2" />


The version too updated


<img width="839" height="399" alt="image" src="https://github.com/user-attachments/assets/df1bcc3d-3c8d-4a74-8bb5-14e5eef2c5a8" />


### Enabling Automated Rollbacks on ECS Rolling Deployment

To enable the Deployment failure detection by updating Service in ECS

To ensure both option enable under the Deployment failure detection > Update


<img width="640" height="194" alt="image" src="https://github.com/user-attachments/assets/2ca9d9d7-fefd-4511-ba4d-15e8b792e693" />


**To update the Dockerfile - I just comment the nginx and added the ubuntu in stage-2 build as it should failed to serve traffic**

```
# syntax=docker/dockerfile:1

# STAGE 1: Build the Angular project
FROM public.ecr.aws/docker/library/node:20 AS builder

# Install Angular CLI
RUN npm install -g @angular/cli@17

# Change my working directory to a custom folder created for the project
WORKDIR /my-project

# Copy everything from the current folder (except the ones in .dockerignore) 
# into my working directory on the image
COPY . .

# Install dependencies and build my Angular project
RUN npm install && ng build -c production


# STAGE 2: Build the final deployable image
# FROM public.ecr.aws/docker/library/nginx:1.25
FROM public.ecr.aws/docker/library/ubuntu

# Allow the HTTP port needed by the Nginx server for connections
EXPOSE 80

# Copy the generated static files from the builder stage
# to the Nginx server's default folder on the image
COPY --from=builder /my-project/dist/my-angular-project /usr/share/nginx/html
```

To update the version 6.0 in src/app/app.component.html

Now commit all the changes to check the CodePipeline - The Pipeline started

The Pipeline was failed after some number of attempt to register the task.


<img width="829" height="344" alt="image" src="https://github.com/user-attachments/assets/da9b1d4f-acba-46ee-9515-2107378adc91" />


As a result, the traffice will serve to version 5.0


<img width="853" height="416" alt="image" src="https://github.com/user-attachments/assets/59973abb-6cf3-4b72-a0c6-4e49f153062b" />


**To update the Dockerfile - I just uncomment the nginx and comment the ubuntu in stage-2 build as it should failed to serve traffic**

The pipeline should trigger automatically - The deployment succeeded and version has been updated!


<img width="826" height="404" alt="image" src="https://github.com/user-attachments/assets/fc75b458-dce5-4d68-a63f-998abb14d414" />


### Min & Max Running Tasks in ECS Rolling

AWS > ECS > Service > update

```
Desired tasks > from 2 to 3
Min running tasks % > 50
Max running tasks % > 150
```

3 tasks started

<img width="623" height="256" alt="image" src="https://github.com/user-attachments/assets/569f3f0c-ee68-41ce-bad3-75c7efe3d5d7" />

To update the version 7.0 in src/app/app.component.html

Now commit all the changes to check the CodePipeline - The Pipeline started


<img width="849" height="338" alt="image" src="https://github.com/user-attachments/assets/5a64a5a5-3822-4062-bfdd-f7359071048d" />


### To delete the ECS Service

From ECS console if you delete the Service then it may delete the resources that by created manually. The best way to delete the ECS - Just navigate to the CFT and delete the ECS which description should have "The template used to create an ECS Service from the ECS Console" > Select > Delete it


### Creating ECS Service on ASG Capacity Providers for Rolling Deployments

ECS > Cluster > Service > Create a new Service

```
Task definition family > myweb
Task definition revision > 10 (as per your revision)
Service name > my-rolling-service-2
Compute options > Capacity provider strategy
Capacity provider strategy > Use Custom (Advanced)
Capacity provider > Infra-ECS-Cluster-my-cluster | Base > 0 | Weight > 1
Service type > Replica > Desired task > 1
Deployment strategy > Rolling update
Min running tasks > 100 %
Max running tasks > 200 %
Deployment failure detection > Choose Use the Amazon ECS deployment circuit breaker and Rollback on failures
Networking > Choose your VPC and public subnets and the SG
Loadbalancing > Use ALB
Create a new ALB > name > my-rolling-service-alb-2
Create a new Listener > HTTP with Port 80
Create a new TG > name > my-rolling-service-target-group-2 > HTTP with Port 80
Deregistration delay > 30 sec
Healthcheck protocol > HTTP
Healtcheck path > /
Create a Service
```






