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





