# Building Your Code with AWS CodeBuild and Deploy the app to S3 Bucket using AWS CodePipeline

## Preparing a Repository in GitHub

To create a new private repository in GitHub and clone the below link to make the project available in the repository.

```
https://github.com/kohlidevops/my-angular-repo
```

To clone the repo in the local machine and push the code to the GitHub repo using below commands

```
echo "# my-angular-repo" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/kohlidevops/my-angular-repo.git
git push -u origin main
```

## To create a S3 bucket for static website hosting

AWS > S3 > Create a new bucket named "my-deployment-testing-s3-bucket" with public access enabled


<img width="638" alt="image" src="https://github.com/user-attachments/assets/a1fa5cd4-903f-47ae-9148-609653707f0e" />


To update the bucket policy with below json file

```
https://github.com/kohlidevops/DevOpsWithAWS/blob/main/angular-app-bucket-policy.json
```

To enable the static website hosting in S3 bucket

S3 > Properties > static website hosting > enable

Hosting type > Static Website Hosting

```
index document - index.html
error document - index.html
```

Then save changes

## Creating a CodePipeline with Build Stage

AWS > CodePipeline > Build a Custom Pipeline

Pipeline name > MyDeploymentPipeline

Execution mode > Queued

Service role > New Role

Source provider > GitHub (via GitHub App)

Connections > Connect to a GitHub and create a connection for specific repo

Repository name > kohlidevops/my-angular-repo

Default branch > main

Output artifact format > CodePipeline default

Next > Add build stage > other build provider > AWS CodeBuild > Create a new build project

Project name > MyTestBuild

Environment > On-demand

Environment image > Managed image

Compute > EC2

Running mode > Container

Operating system > Ubuntu

Run times > Standard

Image > aws/codebuild/standard:7.0

Image version > Always use the latest image for this runtime version

Service role > New service role

Build spec > use a buildspec file

Logs > Cloudwatch logs > Continue to Pipeline

Build type > Single build

Input artifact > SourceArtifact

Next > Skip Test Stage

Next > Deployment stage > Amazon S3

Input artifact > BuildArtifact

Bucket > my-deployment-testing-s3-bucket

Choose > Extract file before deploy

Next > Review > Create a Pipeline

```
Pipeline automatically started and it should be failed due to missing buildspec yaml file - It is expected one
```


<img width="854" alt="image" src="https://github.com/user-attachments/assets/96e1a208-24b4-427a-a20e-f2b573b63d5a" />


## Creating a Buildspec file for Builds


