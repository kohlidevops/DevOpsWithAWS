# Deploying to EC2 Instances with AWS CodeDeploy


![image](https://github.com/user-attachments/assets/877c9c83-1e7a-4470-81ff-3e50ad147edb)


## What is CodeDeploy?

AWS service to automate your application deployments.

A member of AWS Developer Tools, and a managed AWS service.

Scalable — you can deploy to one instance or thousands of instances.

CodeDeploy supports deployments to these platforms:


<img width="502" alt="image" src="https://github.com/user-attachments/assets/3fed3f33-da2a-445f-95e1-5a77014d22f6" />


## Benefits of AWS CodeDeploy

You automate and standardize your deployment process, prevent human errors resulting from manual deployments, and deploy faster.

You can minimize downtime during application deployments to EC2 instances by configuring rolling updates.

You can configure blue-green deployments to avoid downtime.

For Lambda and ECS, you can manage how to shift traffic using one of canary, linear, or all-at-once methods.

You can also configure automated rollbacks in case of errors.


## How to Use AWS CodeDeploy

You can create deployments manually using AWS CLI.

You can integrate CodeDeploy with AWS CodePipeline.


![image](https://github.com/user-attachments/assets/95583c18-61a3-4d5b-aae0-564f2c2173c0)


### 📦 AWS CI/CD Workflow (Using CodePipeline)

This diagram shows how AWS services work together to automate the build and deployment of your application:

**1. AWS CodePipeline:**

The main orchestrator. It manages the flow of code from source to deployment.

**2. AWS CodeCommit (Source):**

Your code is stored in this Git-based repository.
✅ Output: Source Artifact

**3. AWS CodeBuild (Build):**

Takes the source artifact and builds it (compiles, runs tests, packages it).
✅ Output: Build Artifact

**4. AWS CodeDeploy (Deploy):**

Takes the build artifact and deploys it to your servers. In this example, it’s targeting Amazon EC2.


### AWS CodeDeploy Concepts


**Application:** Main component of CodeDeploy. It represents your application.

**Compute Platform:** The platform CodeDeploy application deploys on — EC2/On-premises, Lambda, or ECS.

**Deployment Configuration:** A set of rules and success/failure conditions for the deployment.

**Deployment Group:** A set of instances where the application is deployed to.

**Revision:** The version of your application containing your source code, deployment scripts, and deployment specification file (appspec).

**Deployment:** Execution of the deployment to the deployment group for a specific revision and deployment configuration.

### AppSpec (Application Specification File)

A YAML or JSON file that defines the specifications of the CodeDeploy deployment.

The contents vary by compute platform:

For EC2, it includes the folder path on the instance, and scripts to execute.

The AppSpec file must be in the root directory of your source code.

**File naming requirements:**

_For EC2:_ use appspec.yml (YAML) or appspec.json (JSON).

_For ECS and Lambda:_ use appspec.yaml (YAML) or appspec.json (JSON).



