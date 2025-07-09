# Building your Docker Images and ECS Rolling Deployments with AWS CodePipeline


![image](https://github.com/user-attachments/assets/2197a019-3b18-48e3-b856-bc403d60daa9)



## What is Docker?

A software tool to package & deploy your applications into standardized units called containers.

Containers contain everything your application needs to run, including application code & dependencies, operating system libraries & runtime.


![image](https://github.com/user-attachments/assets/2819362a-581b-4d98-b5df-4cb4156254b6)


Containers are isolated from each other, independently deployable & scalable.

Docker suits well to microservice architectures as it allows decoupling.


![image](https://github.com/user-attachments/assets/70c2d7d3-f25f-4727-aed2-1cd977c4a471)


Containers are created from lightweight, standalone software executables called Docker images.

Docker images contain everything your application needs and become containers at runtime.


<img width="377" alt="image" src="https://github.com/user-attachments/assets/3061d189-3caa-48fc-bb30-66300575237e" />


Docker builds images from an instruction file called a Dockerfile.

A Dockerfile contains all the commands to build an image step by step.


<img width="427" alt="image" src="https://github.com/user-attachments/assets/69135a1e-bda0-433c-9b22-e50af9ad1881" />


You can run containers from your image on your computer to test your applications locally.

You can push your Docker images to a central image repository to deploy it on other environments like staging and production.


<img width="422" alt="image" src="https://github.com/user-attachments/assets/4ff8f9a2-cd89-4a0f-bd36-7a7184a8b194" />


Docker tools for container management & networking may be enough for a single host setup.

Container orchestration tools help you when you need more servers to host your Docker containers for scalability & reliability.


![image](https://github.com/user-attachments/assets/b4b88715-8580-41cf-8384-56e01fa3f55c)


## What is Amazon ECS ?

Highly scalable, native container orchestration tool of AWS

It has direct integrations with the AWS ecosystem.


![image](https://github.com/user-attachments/assets/06ef23c9-5bb0-44e4-a181-2a27091730c7)


**Task Definition (ECS)**

A JSON text file describing the properties of your containers that will run on ECS, such as:

```
Docker image
Ports
CPU
Memory
IAM role
Logging, etc.
```

It can contain up to 10 containers, but you should define multiple containers only if they must be deployed together.

You create ECS tasks from your task definitions to deploy your Docker containers on Amazon ECS.


![image](https://github.com/user-attachments/assets/a4602048-5b16-4fe9-8619-294424d030a0)


You can run standalone task manually using AWS tools.

Standalone tasks are run until they stop for any reason.

You can run tasks at specific interval or in response to an event with AWS Eventbridge.

**Service**

It allows you to maintain a  desired number of running tasks.

If any of the tasks fail, the ECS service scheduler creates a replacement task to keep the number

You can automatically scale the number of your ECS tasks in your service with Application Auto-scaling.


![image](https://github.com/user-attachments/assets/6304d974-9021-486d-93ef-199ff103c205)


You can attach an Application Loadbalancer to your ECS service and distribute traffic to your ECS tasks.


![image](https://github.com/user-attachments/assets/e4936008-4c0d-4d17-823e-94370bdcbd90)


**Cluster**

A logical grouping of your ECS tasks services and other resources in an AWS region.


![image](https://github.com/user-attachments/assets/d19e4059-bba6-4276-94bf-eba3a5f8f54a)


**EC2 Launch Type**

You run your tasks on the EC2 instances you manage that are associated with your ECS cluster. They are called container instances.

Container instances must be running the ECS agent and Docker.

AWS provides ECS optomized AMI's for container instances.

You need to manage the scaling of your container instances to ensure there is enough capacity for new tasks.

All management tasks such as patching, ECS agent version upgrades, and security of the container instances are your responsibility.


![image](https://github.com/user-attachments/assets/7367e391-f858-4024-a799-243b72312b34)


**AWS Fargate Launch Type**

You run your tasks on the infrastructure managed by AWS.

The infrastructure's security, scalability, and reliability are AWS's responsibility. 

Serverless way to deploy containers on AWS.

You focus on building and deploying your containerized applications.


![image](https://github.com/user-attachments/assets/0a7b801d-af0a-40f8-b976-183dbd673f03)


**Capacity Providers**

A newer and enhanced way to manage the infrastructure your tasks run on.

Capacity provider strategies with one or more capacity providers determine how the tasks are run.

An ECS cluster can have one or more capacity providers and an optional default capacity provider strategy.

For Amazon ECS workloads on AWS Fargate, FARGATE and FARGATE_SPOT capacity providers are added to your ECS cluster on creation.

FARGATE works like Fargate Launch Type whereas FARGATE_SPOT allows you to use spare AWS Fargate capacity upto a 70 % discount. But your task can be stopped anytime if the price rises.

For Amazon ECS workloads on Amazon EC2, you create a new capacity provider for the EC2 autoscaling group of your container instances.

On Autoscaling Group capacity providers, ECS can manage the scaling of the container instance capacity. But you still manage the rest.


![image](https://github.com/user-attachments/assets/7d0e2e5d-a541-433a-b5e5-cce73c9fe778)


You can include one or more cpacity providers in a strategy and attach it to your ECS service or standalone tasks.


![image](https://github.com/user-attachments/assets/20ed4fea-75a9-4763-b30f-7d158a3dd831)


