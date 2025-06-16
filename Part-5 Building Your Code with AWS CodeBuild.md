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

### AWS CodeBuild – Buildspec Phases

**➤ install:**

Commands run during installation.

Use for installing packages in the build environment.

Runtime versions are also defined in this phase. If not specified, default runtimes in the build image are used.

**➤ pre_build:**

Commands run before the build.

Use for installing application dependencies, signing in to ECR or Docker Hub, etc.

**➤ build:**

Commands run during the build.

Use for running builds and tests.

**➤ post_build:**

Commands run after the build.

Use for your post-build actions, such as pushing Docker images to ECR, sending notifications via SNS, etc.

buildspec.yml file

```
version: 0.2

phases:
  install:
    runtime-versions:
      nodejs: 20
    commands:
      - npm install -g @angular/cli@17

  pre_build:
    commands:
      - npm install

  build:
    commands:
      - ng build -c production

artifacts:
  base-directory: dist/my-angular-project
  files:
    - '**/*'
```

else you can download the file from below link

```
https://github.com/kohlidevops/DevOpsWithAWS/blob/main/buildspec.yml
```

This buildspec.yml should be placed in the project root directory in GitHub repository


<img width="685" alt="image" src="https://github.com/user-attachments/assets/06a8e2d3-29a9-43aa-a33d-bc9f69ae41b8" />


After updated in the repository, The CodePipeline has been triggered and it is completed successfully!


<img width="868" alt="image" src="https://github.com/user-attachments/assets/2f41737b-e9c6-481e-87f5-ee0eace0ccff" />


If you check with the S3 static website hosting,


<img width="874" alt="image" src="https://github.com/user-attachments/assets/fab43f71-84bb-4f98-b14e-54cc5aa5fa53" />


### Build Life Cycle

✅ SUBMITTED - The build request is submitted to AWS CodeBuild.

✅ QUEUED - The build is waiting in a queue for resources to become available.

✅ PROVISIONING - CodeBuild is setting up the build environment (provisioning the container or VM).

✅ DOWNLOAD_SOURCE - Your source code is downloaded from the source location (like GitHub, CodeCommit, or S3).

✅ INSTALL - This is where tools and runtime environments are installed. 🛠 Example: Install Node.js, Angular CLI, Python, etc.

✅ PRE_BUILD - Any commands that should run before the build process itself. 🧩 Example: npm install, logging in to Docker or ECR, setting up environment variables.

✅ BUILD - The actual build process runs here. 🏗️ Example: ng build, npm run build, mvn package, docker build, etc.

✅ POST_BUILD - Commands to run after the build is done. 📤 Example: Push Docker image, invalidate CloudFront cache, send notifications.

✅ UPLOAD_ARTIFACTS - The built output (e.g., compiled code, zip files, build folders) is uploaded to the specified destination like S3.

✅ FINALIZING - CodeBuild is doing cleanup tasks like closing connections and finalizing logs.

✅ COMPLETED - The entire build lifecycle is complete—success or failure is now reported.


