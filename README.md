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
  - **CLUSTER ----------- - nodegroup-------------- - Pods--------- - containers/images**

# Section One: THE CORE CONCEPT OF ECS
-
---------------------------------------------------------------------------------------------------------------------------------------------------------
- **SOME EXPLANATION HERE**
- 
There are 4 major core concepts of ECS which are:
1)	container drfinition
2)	task definition
3)	service
4)	cluster (which we are discuseed more on it)
Let's get started with a quick demo on ECS so as to understand the flow of ECS and also understand these major core concepts above.
•	So go to your AWS console.
-	In the search bar type is ECS
-	then click on “get started”
you will realize that we are starting with the internal which is container definition and the second which is task definition.
1)	Under container definition
-	choose the image for your container
•	so choose nginx and 
•	then click on “edit” so see what we have in the container definition.
So in the container we will have the following
-	the container name:nginx
-	the image: nginx:latest
-	If it is a private repository (it is a private doctor Bob), we will check this box and then provide the secret manager ARN for the credentials or the name
-	Check the box if it is not a private repository
-	memory limit (MIB) soft limit s12
-	port mappings
-	story or volume
-	log groups
-	click on advanced container configuration to enter all the information when you are creating and defining a container.
•	When we input all the information to define and create a Container, this container will then become part of task.
2)	Under tax definition
•	go under task definition and click on “edit”
-	Task definition name
-	network mode: aws VPC
-	task execution role:
-	Capabilities: FARGATE (we already know about ECS fargate cluster)
-	fask size
•	fask or container memory size: 0.5GB
•	Container or task CPU utilization 0.25GB of CPU
•	We can then add some more definition like auto scaling when doing the actual demo
•	Click on “Next”
3)	Define your service:
-	service name: nginx-service
-	number of desired tasks
-	load balancer type: application load balancer
click on edit to define more resources under service like 5G
click on “next”
4)	configure your cluster
-	click on “next”
-	click again on “next”
-	then click on “create”
When you now go to cloud formation you will see ECS container service creation in progress.
Under this ECS now, you will see resources that are being created.
Click on “view service” to see the details
since we are done with this cluster go ahead and delete it
to delete it:
under the Amazon ECS console click on cluster
-	locate default and click on it
-	then click on “delete cluster”
-	type “delete me” define lutting delete
---------------------------------------------------------------------------------------------------------------------------------------------
- **END OF EXPLANATION**
-	
a)	Set up required IAM roles

