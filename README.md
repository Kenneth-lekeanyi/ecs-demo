# Elastic Container Service (ECS) Pipeline.

<img width="1439" height="711" alt="Screenshot 2025-08-19 at 11 25 56 PM" src="https://github.com/user-attachments/assets/f77d5a38-8f43-4647-a255-ad7f07bc9085" />

- For the Source Code or Application Code that the Developers send to us through the GitHub Repository, Link here; ***{https://github.com/Kenneth-lekeanyi/ecs-demo/tree/master}***.
- Make sure you have these Files
  - app
  - Requirements.txt
  - Dockerfile
  - Buildspec.yml
  - Infra.yml ***{in here, we have "ecs-vpc-ALB", "ecs-Fargate", "ecs-iam"}***
  - The image in the Dockerfile is also passed in there from DockerHub and pass in to Fargate infra or Ec2 infra. So that when we create the Fargate stack, that image will appear under parameter in stack under CloudFormation as
  - **Key:** image URI     **-value:** vamsichunduru/ecs-demo:v1
- We graped this image from his private DockerHub Account or Repository. That is why when we launch the stack, it graped the image from DockerHub and it will create the image which you will see in your Application as {Hi there}
- Now, the whole Application image will be sent to the ECR Repository after the Build in the CodeBuild.
- Remember that when we are doing the deployment, we shall then grap the image from the ECR Repository and it will then deploy the latest version.

- # AWS ELASTIC CONTAINER SERVICE DEEP DRIVE.

- The aim of this studies is to understand the concept how AWS ECS works and dive deeper to understand AWS-ECS basics such as
- 1) a) Cluster
     b) ECS with EC2 Instance type
     c) ECS with Fargate
     d) Task definition
     e) Container definition
     f) Service
     g) Cluster setup manual and automated

- 2) ECS LoadBalancing and Auto-scaling
  3) Setup CICD Pipeline for AWS-ECS deployments for both Rollout and Blue/Green deployment.

# Section One: AWS ECS Basics
