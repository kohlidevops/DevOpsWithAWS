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


