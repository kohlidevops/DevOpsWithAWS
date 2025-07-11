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


