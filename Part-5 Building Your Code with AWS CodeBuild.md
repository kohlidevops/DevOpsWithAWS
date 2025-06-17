# Building Your Code with AWS CodeBuild and Deploy the app to S3 Bucket using AWS CodePipeline


<img width="538" alt="image" src="https://github.com/user-attachments/assets/3feac7ed-1809-44b3-9637-26adea789866" />



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


### Adding a Test Action with CodeBuild

AWS > your Pipeline > Choose > Edit

Select > Add Action group in Build Stage


<img width="693" alt="image" src="https://github.com/user-attachments/assets/5a880b53-0de9-430a-a023-770e066dd844" />


Action name > UnitTest

Action provider > Test > CodeBuild

Input artifact > SourceArtifact

Project name > Create a project

Project name > MyUnitTest

Provisioned model > On-demand

Environment image > Managed image

Compute > EC2

Running mode > Container

Operating system > Ubuntu

Image > aws/codebuild/standard:7.0

Image version > Always use the latest

Service role > New role

Use a buildspec file > unit-test-buildspec.yml

Then > Continue to Pipeline


<img width="861" alt="image" src="https://github.com/user-attachments/assets/cee8e1db-8958-4eb3-9ab9-3e44318f74ce" />


Save & Done


<img width="780" alt="image" src="https://github.com/user-attachments/assets/3ba9a89d-4117-4b5d-957c-1f783f25d38d" />


Finally save the Pipeline


<img width="875" alt="image" src="https://github.com/user-attachments/assets/ece4d61f-7832-4075-a44c-0004450251b0" />


### To add the Unit Test buildspec in project root directory in GitHub repository

```
version: 0.2
phases:
  install:
    runtime-versions:
      nodejs: 20
    commands:
      - npm install -g @angular/cli@17

      # Chrome installation for Angular's unit tests
      - wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | gpg --dearmor | tee /etc/apt/keyrings/chrome.gpg
      - echo "deb [signed-by=/etc/apt/keyrings/chrome.gpg] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list
      - apt-get -y update
      - apt-get -y install google-chrome-stable
  pre_build:
    commands:
      - npm install
  build:
    commands:
      - ng test --no-watch --no-progress --browsers=ChromeHeadlessCI
```


<img width="889" alt="image" src="https://github.com/user-attachments/assets/0d09ff28-fa73-4952-a0f9-8a5a60d2f656" />


After adding the unit-test-buildspec.yml in to the repository, then Pipeline should triggered. (If it is failed due to permission, then add the below policy into the CodePipeline IAM Role)

```
https://github.com/kohlidevops/DevOpsWithAWS/blob/main/codebuild-startbuild-policy.json
```

The pipeline has been succeeded!


<img width="860" alt="image" src="https://github.com/user-attachments/assets/41eb2b01-3d01-4787-8911-ed419390b079" />


If you do change in addition function in repository - my-angular-repo/src/app/calculator/calculator.component.ts

```
switch ( operator ) {
      case '+': {

        this.result =  firstInput / secondInput;
        break;

      }
```

//I have updated symbol "/" instead of "+" and do the commit - Now the test case should failed means Pipeline should get failed

The build has been failed due to test cases


<img width="854" alt="image" src="https://github.com/user-attachments/assets/c5681a9f-58d2-457f-8953-ba4b97184071" />


If you update the symbol "+" instead of "/" and do the commit - Now the test case should succeeded means Pipeline should passed


<img width="847" alt="image" src="https://github.com/user-attachments/assets/569d9f36-c9c1-48d0-a4e3-326f259f740e" />



