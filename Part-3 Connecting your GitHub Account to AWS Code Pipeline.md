# How to Connect GitHub Account to AWS Code Pipeline?

To create a private repository named my-website-repo in GitHub

**create a new repository on the command line**

```
echo "# my-website-repo" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/kohlidevops/my-website-repo.git
git push -u origin main
```

**or clone the repo to make project available**

```
https://github.com/kohlidevops/my-website-repo/tree/main
```

## To create a new connection in AWS Code Pipeline

AWS > Code Pipeline > Settings > Connections > Create a new connection


<img width="848" alt="image" src="https://github.com/user-attachments/assets/f15d3d24-6a7f-464b-8e7e-643f76e8fee7" />


Connect to a GitHub


<img width="729" alt="image" src="https://github.com/user-attachments/assets/a259bec3-65f2-4d30-9225-2888c6c10940" />

Choose > Install a new App to install AWS connector in your GitHub account

Authorize and Select your repository to access > Save


<img width="692" alt="image" src="https://github.com/user-attachments/assets/b0420b8d-b078-4d91-be50-99bc8c709d9f" />


Then connect > Now your GitHub connection is active


<img width="532" alt="image" src="https://github.com/user-attachments/assets/3a88386d-1803-4684-9d47-a7e362af5ef7" />


## What will we build?


<img width="514" alt="image" src="https://github.com/user-attachments/assets/bcb896b3-2d42-4b2e-8d97-2c419196d2cc" />


A CI/CD pipeline that deploys a static website to an Amazon S3 bucket using AWS CodePipeline and GitHub as the source.

### 🔧 High-Level Workflow:

**👨‍💻 Developer pushes code to GitHub**

Files like index.html and error.html are committed

Command: git push origin master

**🔄 AWS CodePipeline is triggered**

GitHub is the source stage

Pipeline automatically pulls changes on each commit

**🚚 Deployment to Amazon S3**

Pipeline deploys the HTML files to a static website–enabled S3 bucket

**🌐 Website goes live**

Users can access the site hosted in the S3 bucket

**💡 Summary:**

A simple, automated CI/CD setup for static website hosting using GitHub, CodePipeline, and Amazon S3.


### Create a Production bucket

To create a production bucket with public access enabled - This is because anyone can access it from anywhere as it is website hosting


![image](https://github.com/user-attachments/assets/f508b2e4-113b-4990-aaf2-8a2d0f6ba4c3)


To make this bucket host static website. First we will add a bucket policy - Bucket > Permission > Edit > Bucket Policy

You can find the bucket-policy.json - https://github.com/kohlidevops/DevOpsWithAWS/blob/main/bucket-policy.json


![image](https://github.com/user-attachments/assets/1da1827a-6774-4c31-82dc-574bfe87d7ee)


To make this bucket static website hosting - Properties > Static Website hosting > Enable > Hosting type > Host a static website

```
index document - index.html
error document - error.html
```

Then > Save changes


### To create a Code Pipeline

AWS > Code Pipeline > Create a new Pipeline

Category > Build a Custom Pipeline > Next

Pipeline name > MyGitHubPipeline

Execution mode > Queued

Service role > New service role > Next


<img width="759" alt="image" src="https://github.com/user-attachments/assets/03de8130-187d-4cbe-8da9-5d772b114a3c" />


Source provider > GitHub (via GitHub App)

Connection > Choose your connection 

Repository name > Choose your repo


<img width="758" alt="image" src="https://github.com/user-attachments/assets/967fcbc1-366b-4794-9482-872a5adbb2f0" />


Default branch > main

Skip build and test stage > Next

Deploy provider > Amazon S3

Region > Choose your region 

Input Artifacts > SourceArtifact

Bucket > Choose your S3 bucket

Choose > Extract file before deploy


<img width="763" alt="image" src="https://github.com/user-attachments/assets/13b004a3-e81e-4f02-b8a1-954f41c6e663" />


Next > Review and Create a Pipeline > The Pipeline has been started and completed


<img width="824" alt="image" src="https://github.com/user-attachments/assets/cd9d7ca9-e0d4-4c5b-b057-4b20e926a458" />


If you access the S3 static website hosting


<img width="690" alt="image" src="https://github.com/user-attachments/assets/c09d8888-13c7-4b6f-9f60-a7f3846b8675" />






