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


### 🔁 AppSpec Lifecycle Hooks

**ApplicationStop**

👉 Think of this as "Clean Up Before New Code"

Stops your running app safely

Cleans the environment (e.g., stops services, removes temp files)

Happens before the new version is downloaded

✅ Used only from second deployment onward (first deployment doesn't have a previous version to stop)

**BeforeInstall**

👉 Think of this as "Get Ready for Installation"

Runs just before installing your new code

Can decrypt secrets, take backups of the current app, etc.

**AfterInstall**

👉 Think of this as the setup step after files are copied.

Use it to:

Run post-install scripts

Configure your application

Set permissions, etc.

📦 Example: Install dependencies or unzip files.

**ApplicationStart**

👉 Time to launch your app (that was stopped earlier).

Use it to:

Start your app

Restart web servers or services

📦 Example: sudo systemctl start myapp

**ValidateService**

👉 Final check – “Did everything go well?”

Use it to:

Run tests or health check scripts

Ensure the app is up and working

📦 Example: Ping an endpoint or check if service is running.

**BeforeBlockTraffic**

👉 Run this before removing the instance from the load balancer (ELB).

Use it to:

Notify monitoring tools

Finish any active requests

Gracefully prepare for downtime

**AfterBlockTraffic**

👉 Run this after the instance is removed from the ELB.

Use it to:

Stop services

Start maintenance tasks

Free up resources

**BeforeAllowTraffic**

👉 Run this before adding the instance back to the ELB.

Use it to:

Finalize app setup

Warm up services or cache

Health check readiness

**AfterAllowTraffic**

👉 Run this after the instance is registered with the ELB.

Use it to:

Notify monitoring that the app is live

Log deployment completion

Send alerts or metrics


### Creating an Appspec File for Deployments to EC2 Instances and update a Buildspec File

#### Summary of Buildspec and Appspec File

**🔨 buildspec.yml — Build Phase (CodeBuild)**

**1. Environment Setup**

Installs Node.js 20 and Angular CLI v17.

**2. Dependency Installation**

Runs npm install to install all project dependencies.

**3. Build**

- Runs Angular production build: ng build -c production. 
- Output is generated in dist/my-angular-project/.

**4. Artifacts Packaging**

Uploads the following to S3 (used by CodeDeploy):

- Built Angular app (dist/my-angular-project)
- Deployment scripts (deploy-scripts/**/*)
- Deployment configuration (appspec.yml)

**🚀 appspec.yml — Deployment Phase (CodeDeploy)**

**1. File Copy**

Copies the built Angular app from dist/my-angular-project (in S3) to /var/www/my-angular-project on the target Linux server.

**2. Set Permissions**

- Applies 0755 permissions (read-execute for all) to all files and folders.
- Sets root as the owner and group.

**3. Post-Deployment Hook**

- Runs a custom shell script: deploy-scripts/application-start-hook.sh.
- This script is used to start or restart the application (typically launching a web server like NGINX).

#### 📘 Full Flow Summary

**🟢 1. buildspec.yml — Runs First (in CodeBuild)**


✅ Runs in AWS CodeBuild.

✅ Installs Node.js and Angular CLI.

✅ Installs your app’s dependencies.

✅ Builds your Angular app → output goes to dist/my-angular-project.

✅ Packages these into an artifact:

- Your app build folder (dist/my-angular-project/**/*)
- appspec.yml
- Any deployment scripts (deploy-scripts/**/*)

✅ Uploads the artifact to S3 — automatically handled by CodeBuild.

**🟠 2. appspec.yml — Runs Next (in CodeDeploy)**

This is your deployment phase — triggered after CodeBuild completes.

✅ Runs in AWS CodeDeploy.

✅ Downloads the artifact from S3 (that CodeBuild uploaded).

✅ Copies your Angular app to the target path: /var/www/my-angular-project.

✅ Sets file permissions and ownership.

✅ Runs deploy-scripts/application-start-hook.sh to start or reload your app (e.g., restart NGINX or PM2).

#### Buildspec updated file

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
    finally:
      - echo 'This is the finally block execution!'
artifacts:
  files:
    - 'dist/my-angular-project/**/*'
    - appspec.yml
    - 'deploy-scripts/**/*'
```

#### Appspec updated file

```
version: 0.0
os: linux
files:
  - source: dist/my-angular-project
    destination: /var/www/my-angular-project
permissions:
  - object: /var/www/my-angular-project
    pattern: '**'
    mode: '0755'
    owner: root
    group: root
    type:
      - file
      - directory
hooks:
  ApplicationStart:
    - location: deploy-scripts/application-start-hook.sh
      timeout: 300
```

#### To create a deploy script

To create a folder named "deploy-scripts" in the root directory of GitHub project repository and create a file named "application-start-hook.sh" inside the deploy-script folder


_application-start-hook.sh_

```
#!/bin/bash
sudo service nginx restart
```


<img width="721" alt="image" src="https://github.com/user-attachments/assets/672edd8c-7237-42ad-8137-0feb39c95e43" />


#### To update a buildspec and appspec file

To add a buildspec and appspec file in the root directory of the project repo and do the Release change in the AWS CodePipeline


//If codepipeline faile due to insufficient permission then please add below inline policy to the CodePipeline role

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "codedeploy:*"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```
The CodePipeline has been succeeded!


<img width="884" alt="image" src="https://github.com/user-attachments/assets/a911f36d-2b40-48a7-b5a7-99a95e5f7938" />


If you access the EC2 DNS


<img width="832" alt="image" src="https://github.com/user-attachments/assets/28cb0a88-9087-4d8d-a447-fba8c1bab181" />


### Viewing Deployment Logs on Your EC2 Instance

SSH to EC2 Instance 

```
ls -lh /opt/codedeploy-agent/deployment-root/deployment-logs/
cat /opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-agent-deployments.log
```

<img width="917" alt="image" src="https://github.com/user-attachments/assets/5bfe1684-9344-4548-a208-0572bc6dbdf8" />


You can find the deployment group id, deployment archieve, and its logs using below path

```
ls -lh /opt/codedeploy-agent/deployment-root/f5f8e035-99f2-4def-912c-3e6cadc55ef5/
ls -lh /opt/codedeploy-agent/deployment-root/f5f8e035-99f2-4def-912c-3e6cadc55ef5/d-5RUS2K3BC/
```


<img width="944" alt="image" src="https://github.com/user-attachments/assets/b05670d4-75db-46cb-9445-974172318c04" />


### To Streaming Deployment Logs to CloudWatch Logs

**To add the CloudWatchAgentServer Policy to an EC2 Instance IAM Role**

AWS > Choose your EC2 Instance > Security > IAM Role > Attach the policy and Save


<img width="695" alt="image" src="https://github.com/user-attachments/assets/32ba1d2d-ff3d-4597-945f-fde075b0fcc7" />


**To Instance the CloudwatchAgent onto the EC2 Instance**

SSH to EC2 Instance

```
# Download the unified CloudWatch agent
wget https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm
# Install the package
sudo rpm -U ./amazon-cloudwatch-agent.rpm
# Create configuration file using CloudWatch agent wizard
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
# Now it will ask set of questions as below
```

On which OS are you planning to use the agent?
1. linux
2. windows
3. darwin
default choice: [1]: 1

Trying to fetch the default region based on ec2 metadata...
I! imds retry client will retry 1 timesAre you using EC2 or On-Premises hosts?
1. EC2
2. On-Premises
default choice: [1]: 1

Which user are you planning to run the agent?
1. cwagent
2. root
3. others
default choice: [1]: 1

Do you want to turn on StatsD daemon?
1. yes
2. no
default choice: [1]: 2

Do you want to monitor metrics from CollectD? WARNING: CollectD must be installed or the Agent will fail to start
1. yes
2. no
default choice: [1]: 2

Do you want to monitor any host metrics? e.g. CPU, memory, etc.
1. yes
2. no
default choice: [1]: 2

Do you have any existing CloudWatch Log Agent (http://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AgentReference.html) configuration file to import for migration?
1. yes
2. no
default choice: [2]: 2

Do you want to monitor any log files?
1. yes
2. no
default choice: [1]: 1

Log file path: /opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-agent-deployments.log

Log group name: default choice: [codedeploy-agent-deployments.log]

Log group class:
1. STANDARD
2. INFREQUENT_ACCESS
default choice: [1]: 1

Log stream name:
default choice: [{instance_id}]

Log Group Retention in days:
default choice: [1]: 2

Do you want to specify any additional log files to monitor?
1. yes
2. no
default choice: [1]: 2

Do you want the CloudWatch agent to also retrieve X-ray traces?
1. yes
2. no
default choice: [1]: 2

Existing config JSON identified and copied to:  /opt/aws/amazon-cloudwatch-agent/etc/backup-configs
Saved config file to /opt/aws/amazon-cloudwatch-agent/bin/config.json successfully.

Do you want to store the config in the SSM parameter store?
1. yes
2. no
default choice: [1]: 2

```
# Edit the config file
sudo nano /opt/aws/amazon-cloudwatch-agent/bin/config.json

# "timestamp_format": "[%Y-%m-%d %H:%M:%S.%f]"
# Ctrl+X to exit nano, press Y to save changes, press Enter to approve.
```

Your config.json would be

```
{
        "agent": {
                "run_as_user": "cwagent"
        },
        "logs": {
                "logs_collected": {
                        "files": {
                                "collect_list": [
                                        {
                                                "file_path": "/opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-agent$                                                "log_group_class": "STANDARD",
                                                "log_group_name": "codedeploy-agent-deployments.log",
                                                "log_stream_name": "{instance_id}",
                                                "retention_in_days": 1,
                                                "timestamp_format": "[%Y-%m-%d %H:%M:%S.%f]"
                                        }
                                ]
                        }
                }
        }
}
```

To start/stop the CloudWatchAgent and check the status

```
# Start CloudWatch agent 
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s
# Check CloudWatch agent status
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status
```

<img width="876" alt="image" src="https://github.com/user-attachments/assets/eadb8451-3b70-4919-9a91-7b0664b41e7b" />


If you check with the CloudWatch Loggroups


<img width="890" alt="image" src="https://github.com/user-attachments/assets/71891d15-afd1-415c-8ec6-baaadb3e8025" />


### Creating a Deployment Group with Auto Scaling & Load balancing


**To create an Image for the MyDeployment EC2 instance**

SSH to instance and clean up the webserver project using below command

```
ls -lh /var/www/my-angular-project/
sudo rm -rf /var/www/my-angular-project/*
```

Then stop the instance and create an Image


**To create a Launch Template**

AWS > EC2 > Launch Template > Create a new Launch Template

```
name > MyDeploymentLaunchTeamplate
Choose > Autoscaling guidance
AMI > Choose you AMI
Instance type > t2.micro
Keypair > Choose your keypair
Subnet > Dont include in this Launch Template
Security group > Choose your EC2 instance SG
IAM Instance profile > Choose your CodeDeployRole (Which is already associated with your EC2 instance)
Lets create a Launch Template
```


<img width="563" alt="image" src="https://github.com/user-attachments/assets/13ea37a2-e967-4885-92dc-cf983b246116" />


**To create a Security group for ALB**

To create a Security group and add the inbout rule as Http with OpentoAnywhere


**To create a Target group**

```
Target type > Instance
Name > MyDeploymentTG-80
Protocol > HTTP
Port > 80
VPC > Default
Health check protocol > HTTP
Health check path > /
Next and Create a Target Group
```

**To create an Application Loadbalancer**

```
Load balancer name > MyDeploymentALB
Scheme > Internet facing
Load balancer IP address type > IPv4
Network Mapping > Choose your VPC and all public subnets
Security group > Choose your SG
Listeners and Routing > HTTP | 80 | MyDeploymentTG-80
Then create a Load balancer
```

**To create an Auto Scaling Group**

```
ASG name > MyDeployment-ASG
Launch Template > Choose your Template
Network > Choose your VPC and all availability zones
Loadbalcing > Attach to an exisiting Load balancer > Select your Target Group
Turn on ELB health checks
Desired > 2 | Min > 2 | Max > 2
Next > Next to Create an ASG
```

**To create a Deployment Group in CodeDeploy**

AWS > CodeDeploy > Application > Choose your Application


<img width="662" alt="image" src="https://github.com/user-attachments/assets/55c199ce-c6b9-4175-b8de-a43cfca9a04f" />


```
Create a new Deployment Group
Name > MyDeploymentAutoScalingGroup
Service Role > Choose your Service Role
Deployment type > In-place
Environment configuration > EC2 AutoScaling Group > Choose your ASG
Deployment configuration > CodeDeployDefault.AllAtOnce
Enable Load balancer > ALB > Choose your Target Group
Then create a Deployment Group
```


<img width="647" alt="image" src="https://github.com/user-attachments/assets/aa6491c8-d4ea-411b-abae-e78c3cf2171e" />


To update the Deploy Action in your CodePipeline

AWS > CodePipeline > Choose your Pipeline


<img width="856" alt="image" src="https://github.com/user-attachments/assets/e5c67159-a49c-441b-ad20-0b7dcf194a16" />


```
Edit > Your Pipeline
Choose > Deploy Stage
Edit > DeployToEC2Instance (Exisiting one)
Deployment Group > MyDeploymentAutoScalingGroup (It should be new one which is created now in Deployment Group)
Done > Done > Save Changes
```

Then Release the change to deploy everything on the EC2 ASG! If you check with ALB URL,


<img width="839" alt="image" src="https://github.com/user-attachments/assets/c72a0c03-18b3-403b-9501-1afeaec8096e" />


The CodePipeline deployed the webapp on the EC2 ASG instances successfully!


### In-Place All-At-Once Deployments with Auto Scaling & Load balancing

In an In-Place All-At-Once deployment using AWS CodeDeploy, the new application version is deployed simultaneously to all instances in the deployment group.

**Key Characteristics:**

⚡ Fastest deployment method

🔄 Replaces the current version directly on existing EC2 instances

💥 All instances go out of service briefly during deployment

**Pros**:

Quick rollout of changes

Simple deployment strategy

**Cons:**

❌ Not suitable for production

⛔ Causes short outages (downtime during deployment)

❌ No rollback safety without manual intervention

❌ High risk if new version fails


I have changed the version as 2.0 in GitHub Project - src/app/app.component.html. Then Pipeline triggered automatically! Let see what will happen

The Deployment started


<img width="832" alt="image" src="https://github.com/user-attachments/assets/7e4bb634-c76f-4768-8906-d5d53f1529ad" />


Now Target group instances are started draining

<img width="727" alt="image" src="https://github.com/user-attachments/assets/44b86c1f-48cd-476b-81db-3d1a5e23981b" />


As a result, we could see the temporary outage which should not accept in production


<img width="559" alt="image" src="https://github.com/user-attachments/assets/e94e5009-356f-4798-8f22-b1b15608a124" />

Now Deployment has been done. So i can able to see the site and version has been changed as v2.0


<img width="823" alt="image" src="https://github.com/user-attachments/assets/5d5297d4-6512-40c3-9d3f-fe7a29055e51" />


### What If Your Auto Scaling Groups Scales Out?

If you use CodeDeploy with an EC2 Auto-Scaling group and if it scales out, the revision from the last known successful deployment is deployed to the new EC2 instances automatically. However please associate your auto-scaling group with only one CodeDeploy deployment group at a time. Otherwise multiple deployments from different deployment group may cause conflicts in your EC2 instances, and they become a Unhealthy. It may even cause your aut-scaling group to repeatedly launch and remove new EC2 instances

If you scale-out the ASG from 2 to 3, then new deployment has been succeeded!


<img width="645" alt="image" src="https://github.com/user-attachments/assets/908521cc-83f7-4a42-89cd-dc91c1e125e8" />


All ASG instances are healthy to receive traffic


<img width="679" alt="image" src="https://github.com/user-attachments/assets/8d2c98c2-d22a-4d74-af9c-e300f57ebb28" />



### In-Place Rolling Deployments With Auto Scaling & Load balancing

1. CodeDeploy starts the deployment on one instance at a time.

2. Before deploying, CodeDeploy: Deregisters the instance from the Load Balancer (so no traffic is sent to it).

3. Deployment runs on that instance (app stops, new version installed, app restarts).

4. After success, it is re-registered to the Load Balancer.

5. The process repeats for the next instance until all 3 are updated.

**🔁 If a Deployment Fails:**

1. CodeDeploy immediately stops the deployment.

2. It triggers a rollback to the last successful revision on already-deployed instances.

3. Unaffected instances stay on the original version.

4. ASG remains healthy, and traffic is only served by successfully running instances.

To change the Deployment configuration in CodeDeploy application

AWS > CodeDeploy > Application > Choose your Deployment groups > MyDeploymentAutoScalingGroup > Edit


<img width="553" alt="image" src="https://github.com/user-attachments/assets/3ae39a63-ff5d-4084-9f19-82cd89eac52d" />

```
Deployment configuration > CodeDeployDefault.OneAtATime
Choose > Advanced
Disable rollbacks > Deselect
Select > Roll back when a deployment fails
Save the changes
```

To change the version as 3.0 in GitHub repo - src/app/app.component.html

Then Pipeline trigger automatically, then deploy the app one by one


<img width="732" alt="image" src="https://github.com/user-attachments/assets/8054518d-922b-4fa1-8cbc-fd11ef18aa07" />


If you see the Deployment Lifecycle, then one is in-propgress others are pending


<img width="593" alt="image" src="https://github.com/user-attachments/assets/e081d8d8-5605-45ef-89da-ca133213ff32" />


Sometimes your application will serve - V3.0


<img width="827" alt="image" src="https://github.com/user-attachments/assets/d273fb43-6145-4f23-be78-553cb63cb060" />


Sometimes V2.0


<img width="845" alt="image" src="https://github.com/user-attachments/assets/6be6b38d-7a66-4d2a-825d-146042314a44" />


This is slow deployment, But safe!


<img width="559" alt="image" src="https://github.com/user-attachments/assets/2364294e-4f05-4117-ae79-d58982327039" />


The deployment lifecycle events are succeeded!

<img width="638" alt="image" src="https://github.com/user-attachments/assets/ac90ff07-e6aa-4fb0-8e68-c40e33e57e7b" />


### Automated Rollback on In-Place Rolling Deployment Failure

1. You're using CodeDeployDefault.OneAtATime strategy with 3 EC2 instances.

2. Rollback is enabled (via deployment configuration or UI).

3. CodeDeploy starts in-place deployment, updating one instance at a time.

**❌ What Happens When the First Instance Fails?**

1. CodeDeploy stops the deployment immediately.

2. It does NOT proceed to the other 2 instances.

3. It initiates rollback:

```
For the instance where the deployment failed → it will attempt to roll back to the last successful revision.
The other 2 instances remain untouched, since they were not updated yet.
```

**To create a file in repo**

deploy-scripts/simulate-failure.sh

```
#!/bin/bash

echo "This script will fail"
exit 1
```

To update the appspec file

```
version: 0.0
os: linux
files:
  - source: dist/my-angular-project
    destination: /var/www/my-angular-project
permissions:
  - object: /var/www/my-angular-project
    pattern: '**'
    mode: '0755'
    owner: root
    group: root
    type:
      - file
      - directory
hooks:
  ApplicationStart:
    - location: deploy-scripts/application-start-hook.sh
      timeout: 300
  ValidateService:
    - location: deploy-scripts/simulate-failure.sh
```

To change the v4.0 in src/app/app.component.html

Then commit the changes and Pipeline should trigger automatically! (This should get failed due to simulate-failure.sh file)

Deployment lifecycle failed and others are skipped


<img width="659" alt="image" src="https://github.com/user-attachments/assets/3dbd5107-0149-44c4-aa62-00a5f01a2ac3" />


The new deployment started to deploy the latest successful revision to the failed instance


<img width="652" alt="image" src="https://github.com/user-attachments/assets/d4f97346-777f-4256-9594-eda89f3bd061" />


Now, the last successful revision has been deployed to the failed instance


<img width="650" alt="image" src="https://github.com/user-attachments/assets/841abd03-9e3f-425c-aa4a-46a7567b3075" />


