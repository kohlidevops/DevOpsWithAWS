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

