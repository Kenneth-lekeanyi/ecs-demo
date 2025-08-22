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
# a)	Set up required IAM roles.
- We need to set up an EC2 Role. The reason is for the ECS agents on the EC2 instance to interact with the ECS and ECR. So,
- go to "IAM"
-	click on “Roles”
-	click on “create new role”
-	"Trusted entity type"
  - click to select [AWS service]
  - Then click on [EC2] 
-	Click now on “next”
-	Permissions: ***{Since AWS already has an EC2 container service EC2-Role, we are going to use it. So search and select
 - Click to check the box on **[AmazonEC2ContainerServiceforEC2Role]**  {This policy was created by default by AWS for Amazon EC2 Container Service}
-	Then click on “Next”
-	Rolename: **ecs-demo-ec2-role**
-	Description: **Allow EC2 instance to call AWS service on your behalf**
-	Click now on “Create role” at the bottom
-	
For the roles, we have to create 4 Roles. As we have already created the first Role, we are going to use CloudFormation to create the remaining Roles which are
1)	**ecs-demo-ecs-role**.  ***(This role is for ECS cluster to manage AWS resources)***
2)	**ecs-demo-task-execution-role**. ***(This role is for task execution to pull images from ECR and push logs to cloudmatch as well)***
3)	**ecs-demo-asg-role**. ***(This auto scaling role is to verify the instance frequently and setup auto scaling as required). This role will give the flexibility from instances to interact with autoscaling group***.
- "Challenge: if you create a Role, and dont attach a permission to it at creation, EC2 WILL BE CREATED BUT CREATION PROCESS WILL JUST BE IDLING AND NOT GO THROUGH"
- 
- So download the codebase from this repository; https://github.com/Kenneth-lekeanyi/ecs-demo

- When you are in this GitHub repository in the due GitHub Account,
- clone the repo by clicking on the green botton “Code”
  - Either you clone it under HTTPS or you click on “Download zip”. When you click on "Download zip", it will take the **ecs-demo-master** to your Download folder in your local. You can now see the same folder in your local machine under your download having 
  - app
  - infra
  - Dockerfile
  - README.md
-	Now, open your text editor ( V.S code) and find this **ecs-demo-master** in V.S code to see the
  - app
  - infra
  - Dockerfile
  - README.md
- Now, inside this our "ecs-demo folder", we have three folders: app,infra and Dockerfile
  - **app:** ***{This is basically the Application code folder that is coming from Developers and we are going to deploy it. This Application Folder consist of both '**Contactus.html***' and '**index.html**'
  - **infra:** ***{This is where we have our CloudFormation.yml scripts such as **"ecs-ec2.yml"**, **"ecs-fargate.yml"**, **"iam.yml"**, and **"vpc.yml"**. These are the things that we are going to use for our infrastructure}***
  - **Dockerfile:** ***{This consist of our image that will be used in future.}***

# b)	Now set up the VPC. (We shall see cloud formation scripts) to set up the VPC.
- We need to set up the VPC so that ECS cluster can use the VPC network for the following demos. This because we need to launch our cluster in a VPC.
- In order to create the VPC and subnets flow, let's use the predefined CloudFormation template which we have already created.
- So, in order to do this,
  - Go to AWS account
  - go to the CloudFormation console
  - click on “create stack"
  - click to select [with new resources (standard)]
  - prepare templates; click on [Template is ready]
  - Specify template:
    - click on [upload a template file] ***{This is because we have our template file in our local folder}***
    - then click on “choose file” ***{It will take you to either your Downloads or your Documents where you have your ecs-demo-master folder. There, select "Infra", then select "vpc", then click on "open"}***
- Then click on “next”
-	Stack name: **ecs-demo-network**
-	EnvironmentName: **ecs-demo**
- Then click on “next”
- Click again on "Next" (we dont have to do anything on Configuration)
- Then click on “create stack”
- Now our VPC is set up, and its up and running.
- 
- Let's proceed to the next step, which is to setup the ECS cluster for both **EC2-based** and **fargate-based** clusters.
- Note: For Windows users, if you are using Gitbash, use **.pem** But if you are using putty, select **.ppk** for the keypair.

# c)	Set up the ECS cluster for both EC2-based and Fargate-based cluster.
 ***{Just Showing us how to Set up both ECS, EC2 and fargate clusters using the AWS console in order to understand the flow.}***
1)	ECS, EC2-based cluster setup
- Create 1 keypair in the AWS EC2 console with the name as **ecs-demo**, so that we can use it in future. If required, when launching the EC2 based ECS cluster;
  - Go to EC2 console
  - locate and click on keypair. ***{This keypair will also help you to ssh into the instance if there is an issue}***. (We can use an existing keypair, but since we already hardcoded the name of the ecs-demo keypair in the ecs-ec2.yml file, we should just maintain the name as of our keypair as "ecs-demo", or you can go to the ecs-ec2.yml file and change the name of the key under keyname)
  - Create keypair
  - keypair name: **ecs-demo**
-	create keypair
- We are done creating the keypair. Let's now proceed to create the cluster (i.e the ECS, EC2-based cluster.
- 
2)	Now let's go ahead and create an ECS, EC2 type cluster via AWS console.
- So,
-	Go to **Elastic Container Service(ECS console)**
-	Locate “cluster” on the left and click on it 
-	Now click on “create cluster”
   - As you can see here, we have 3 clusters;
     - Fargate
     - EC2 Linux
     - EC2 Windows
     - So, lets start with EC2 Linux, because our Agenda is to create an EC2 Linux cluster first. ***{Ask your Developer: what is the minimum RAM, Size of EBS, size of CPU, and Instance Type that is needed for this Application}***
-	So select "EC2 Linux + networking"
-	Click now on “Next step”
-	Under "Configure cluster";
- Cluster name: **ecs-demo-ec2-cluster**
-	Under Instance configuration
- Click to select [on-Demand instance]
-	Under EC2 instance type;
- click to select [t2.small]
-	Number of instances: **1**
-	EC2 AMI ID: **Amazon linux2 Ami** ***{Here, AWS must have combined the EC2 Agent with Docker, that is why you have this AMI ID here. So this EC2 machine is going to run the Docker image which makes it mandatory to have Docker installed. And since this EC2 machine will be having communication back and forth with the ECS Cluster, that is why it has an Agent installed in it too. That is why, it has created an AMI and thesame AMI is used in the EC2 Instance}***
-	Root EBS volume size: **30**
-	Keypairs: **ecs-demo**
- Networking: ***{Since we have created our VPS stark and exported the VPC and two public subnets, you can see this by going to CloudFormation, Click on "stack details", then click on “outputs” to see the VPC ID, so that you will note it and be able to select it under networking.
-	Now VPC: **select the VPC that you have noted it ID down**
-	Subnets: **select bioth of the subnets created and imported through output. (i.e 2 subnets)**
-	Auto assign public IP: **Use subnet setting**
-	Security Group: **create a new SG**
-	Security group inbound rules: **CIDR block**: 0.0.0.0./0, ------- **port range:** 80
-	Cortainer instance IAM role: **ecs-demo-ec2-role**
-	Click now on "create" to create our clusters.
- Once the clusters have been created verify the cluster creation by accessing CloudFormation, ECS console and EC2 console.
- Once all the three are created successfully click on “view cluster” to see the cluster.

- If you click on ECS Instances, you will not see any instance. But if you go to EC2 Console and Instances, you will see that there is a new instance there just created with the name "ECS Instance".
- Still under ECS Instance in the ECR Console, you will see the Container Instance. So, with this, you know the EC2 Instance that is hosting your containers. As we can at times have multiple containers in one EC2 Instance.
- On the CloudFormation Console, you will see that, beside the cluster that has been created, there are other resources being created as well such as Auto-scaling group, LB, and SG that are created and attached to the cluster as well.

-Let's now delete the cluster as we are now going to create a new cluster with a fully automated CloudFormation template or script.
# Deleting the cluster 
1)	Delete EC2 based ECS cluster By deleting the "ECS-demo-cluster stack" in the CloudFormation console and confirmed the deletion
-	So go to your ECS console
-	then click on “delete cluster”
- this cluster is meant to understand all inputs and what they do manually. Then we can think about automating the very process using CloudFormation, CDK or Trraform.
- 
2)	Let's now delete the "ecs-demo-ec2-role" That we created
-	go to IAM console
-	select Role
- search for "ecs-demo-ec2-role"
- select it
- then click on "delete".

- 
# Creation of ECS cluster using Cloud formation Script.
1)	IAM Stack creation.
Let's proceed to create all the above created rules but this time around with cloud formation, so as to understand how the flow works with automation.
a)	Start by creating a required IAM roles by using the cloud formation script.
So go to cloud formation console
-	click on “stack”
-	 click on “create stack”
Click on “with new resources” (standard)
-	Prepare template
Upload a template file
-	Upload a template file
click here on choose file
then click on “open”
-	click on next
-	stack name: ecs-demo-iam
-	environment name: ecs-demo
-	click on “next”
-	configure stack options
Click on “next”
-	click on “create stack”
Under the stack you will see all the rules that has been created. On the output you will see the values of the resources that were exported in the script.


