# Building your Docker Images and ECS Rolling Deployments with AWS CodePipeline

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


