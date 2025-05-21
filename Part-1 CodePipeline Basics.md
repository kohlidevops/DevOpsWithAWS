# AWS CodePipeline basics

## What is CI/CD?

### Continuous Integration

In Conctinuous Integration (CI), developers commit the code changes frequently, several times a day, to the code repository.

In every code submission, automated builds and unit tests are run.

So Integration errors are detected earlier and can be eliminated before the deployment.


![image](https://github.com/user-attachments/assets/a468efb1-e726-4164-b519-fad74f0515d1)



### Continuous Delivery

In Continuous Delivery, each code submission is built, tested and deployed to a non-production environment. Multiple tests can be run before production.

The deployment to production required a manual approval.


![image](https://github.com/user-attachments/assets/67f7da9a-1758-46fb-a539-71bba053105e)


### Continuous Deployment

In Continuous Deployment, each code change is automatically built, tested and deployed to a staging environment as in Continuous Delivery.

But there is no manual approval process. The code is deployed to the production environment automatically.


![image](https://github.com/user-attachments/assets/8db037c4-31fd-411a-81d3-b89b5b6d0ad5)


## What is AWS CodePipeline?

AWS CodePipeline is a continuous delivery service that helps you model, visualize, and automate your software release process.

You can model continuous deployment or continuous delivery to build, test, and deploy to pre-production and production environments based on your needs.

You can integrate third-party tools like Jenkins, BlazeMeter, or add custom actions using AWS Lambda.

AWS CodePipeline acts as the orchestrator of your CI/CD workflow.


![image](https://github.com/user-attachments/assets/bd16510f-49a1-4fa9-b843-526d6402d2f4)


### AWS CodePipeline Concepts (Simplified)

Pipeline: A step-by-step workflow that shows how your code moves from writing to deployment. It helps you automate how your software is built, tested, and released.

Source Revision: - This is the specific version of your code that triggers the pipeline to start. You can have more than one version flowing through the pipeline at once. If you push multiple commits quickly, each one is treated as a separate source revision. CodePipeline can process multiple revisions at the same time, even if one is already being built, tested, or deployed.


### AWS CodePipeline Concepts

Stage: A group of one or more actions in a pipeline. Only one source revision can be processed in a stage at any given time.

Action: A task that is performed on a source revision. Actions within a stage can run sequentially or in parallel.


![image](https://github.com/user-attachments/assets/a93ec130-f3f0-487e-9777-17df91aaa77c)


Transition: A connection between two stages in a pipeline. It can be enabled or disabled to control the flow between stages.

Artifact: A file or set of files produced as output from an action. These are used as input for subsequent actions in later stages.


![image](https://github.com/user-attachments/assets/d0f67a23-7a67-4388-a200-1919dcde05ef)


Pipeline Execution: A unique release of changes processed through the pipeline. Each pipeline execution has its own ID to track the specific run.

An execution can be triggered by: A source code commit, or manual release of the latest commit.


### How your Pipeline works?


You push your changes to the source location of your pipeline — this creates a source revision.

Your pipeline is triggered, and the source revision flows through the pipeline stages.

Finally, the source revision is deployed to the production environment.


![image](https://github.com/user-attachments/assets/219a61e4-b82b-4464-9336-56b8f094663c)
