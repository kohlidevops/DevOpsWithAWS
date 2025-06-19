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


### How do EC2 Instances work with AWS CodeDeploy?

➤ EC2 instances communicate with AWS CodeDeploy through a software package called the CodeDeploy agent.


<img width="419" alt="image" src="https://github.com/user-attachments/assets/a1e6cb7e-2686-48a3-b990-6b83a51e3433" />

➤ The CodeDeploy agent supports specific versions of Amazon Linux, Ubuntu Server, Windows Server, and Red Hat Enterprise Linux (RHEL).

➤ Once installed and enabled, the CodeDeploy agent polls AWS CodeDeploy for new deployments every second by default.

➤ The CodeDeploy agent downloads the revision containing the source code, the Appspec file, and dthe deployment scripts from Amazon S3.

➤ The EC2 instance needs permission to access this S3 bucket via an IAM role.


<img width="464" alt="image" src="https://github.com/user-attachments/assets/d309e246-caf3-42e0-96ee-9ebce7fabe97" />


➤ After downloading the revision, the CodeDeploy agent performs the deployment according to the Appspec file.


**Creating an IAM Role for EC2 instances for Deployments**

AWS > IAM Role > Create a new IAM Role > Trusted entity type > AWS Service > Use case > EC2 > Associate IAM Policy "AmazonEC2RoleforAWSCodeDeploy" and name it "MyDeploymentCodeDeployRoleForEC2"


<img width="698" alt="image" src="https://github.com/user-attachments/assets/ea4b3341-230c-4ed3-9c24-d5d637466f2e" />


### Configuring EC2 Instance for AWS CodeDeploy

To Launch an Amazon Linux2

AWS > EC2 console > Launch Amazon Linux2 > Associate an IAM Role "AmazonEC2RoleforAWSCodeDeploy" and launch Instance


<img width="792" alt="image" src="https://github.com/user-attachments/assets/986453f5-beb8-49a6-8184-b2fdb2c3c255" />


SSH to EC2 Instance

```
sudo yum update -y
sudo yum install -y ruby wget
wget https://aws-codedeploy-ap-south-1.s3.ap-south-1.amazonaws.com/latest/install
chmod +x ./install
sudo ./install auto
# Checking CodeDeploy Agent status
sudo service codedeploy-agent status
# Installing Nginx
sudo amazon-linux-extras install -y nginx1
# Checking Nginx status
sudo service nginx status
# Starting Nginx
sudo service nginx start
# Enabling Nginx to restart on system reboot
sudo chkconfig nginx on
# Creating folders for deployments
sudo mkdir -p /var/www/my-angular-project
# Changing Nginx configuration
sudo nano /etc/nginx/nginx.conf
# Only change root to /var/www/my-angular-project
//root         /var/www/my-angular-project/
//save and exit
# Restart Nginx
sudo service nginx restart
```


<img width="756" alt="image" src="https://github.com/user-attachments/assets/c6d174a8-4c96-4885-9ba6-3ccd9a1ae339" />


Its expected one as this directory is empty! So CodeDeploy raises 403 error


### Creating a CodeDeploy Deployment Group for Tagged EC2 Instances

To Create an IAM Role for CodeDeploy Service

AWS > IAM Role > Create a new IAM Role > Trusted entity > CodeDeploy > Use case > CodeDeploy > name it "MyDeploymentCodeDeployServiceRole" and create

Add below inline policy to that role

```
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "Statement1",
			"Effect": "Allow",
			"Action": ["ec2:RunInstances", "ec2:CreateTags", "iam:PassRole"],
			"Resource": "*"
		}
	]
}
```

else you can use below link to copy

```
https://github.com/kohlidevops/DevOpsWithAWS/blob/main/CodeDeployBlueGreenPolicy.json
```

Finally your CodeDeployServiceRole should belo policies


<img width="695" alt="image" src="https://github.com/user-attachments/assets/c5af5c21-d928-422f-b43f-faa14085d9d8" />


### Create a Application in CodeDeploy

AWS > CodeDeploy > Application > Create a new Application

Application name > MyDeploymentApplication

Compute platform > EC2/On-Premise


<img width="773" alt="image" src="https://github.com/user-attachments/assets/ae0aa1a7-35ba-4ef9-971c-f1f910b4a0c7" />


### Create a Deployment Group

AWS > CodeDeploy > Application > Choose your Application "MyDeploymentApplication" > Deployment Group > Create a Deployment Group


<img width="678" alt="image" src="https://github.com/user-attachments/assets/21dd04d8-890e-4ada-9c28-1ece882f6a6e" />


Deployment group name  > MyDeploymentGroup

Service role > Just select what you have created now

Deployment type > In place

Environment configuration > Amazon EC2 Instances

Tag key > Name and Tag value > MyCodeDeployment-AmazonLinux2

Deployment settings > Deployment configuration > CodeDeployDefault.AllAtOnce

Disable > Load balancer > Create a Deployment group


<img width="676" alt="image" src="https://github.com/user-attachments/assets/3b268e9e-63e0-48d8-af11-3b17c73cfdd7" />


### Adding a CodeDeploy Deploy Action to your Pipeline

Already we have a created a CodePipeline for S3 Static website hosting. So Im going to use the same Pipeline - Please have a look below link 

```
https://github.com/kohlidevops/DevOpsWithAWS/blob/main/Part-5%20Building%20Your%20Code%20with%20AWS%20CodeBuild.md
```

Existing Pipeline is like below


<img width="855" alt="image" src="https://github.com/user-attachments/assets/1644b9a0-2a09-4e01-b2c4-1107501a299c" />

As I'm going to remove the "Deploy" Action using Edit > Pipeline > Deploy > Edit stage > Remove the current Deploy Action and New Deploy Action

Action name > DeployToEC2Instance

Action provider > AWS CodeDeploy

Region > Mumbai

Artifacts > BuildArtifact

Application name > MyDeploymentApplication

Deployment group > MyDeploymentGroup

Done > save the Pipeline changes


<img width="853" alt="image" src="https://github.com/user-attachments/assets/fe4f74ad-b869-4b88-84b2-93ea518bb5e6" />











