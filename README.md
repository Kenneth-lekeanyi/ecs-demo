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
# 1)	IAM Stack creation.
- Let's proceed to create all the above created Roles, but this time around with CloudFormation, so as to understand how the flow works with automation.
# a)	Start by creating the required IAM Roles by using the CloudFormation script. ***{So that these Roles will be used during the creation of the Cluster}***
- So, go to the CloudFormation console
-	click on “stack”
-	 click on “create stack”
- Click on “with new resources” (standard)
-	Prepare template:
  - Click to select [Template is ready]
- Template Source:
  - Click to select [Upload a template file]
-	Upload a template file:
  - click here on [choose file] {It will take you to your folders. Go to Downloads where you have the "ecs-demo" Folder and choose "Infra" then select "iam.yml".
- Then click on “open”
-	click on "next"
-	stack name: **ecs-demo-iam**
-	environment name: **ecs-demo**
-	click on “next”
-	configure stack options
- Click on “next”
- click to check the box on "Acknowledge"
-	click on “create stack”
- Under the stack, you will see all the Roles that has been created. Under [output], you will see the values of the resources that we exported in the script.

- # 2)	Creating the ECS-EC2-based Cluster using CloudFormation.
- Go to your CloudFormation console
-	then click on “stack”
-	click on "create stack”
- select "with new resources (standard)"
-	Prepared templates:
  - Click to select [template is ready]
-	Specify templates:
  - Click to select [upload a template file]
-	Upload a template file:
  - click here on [choose file] ***{it will take you to the Folder. Go to your Downloads where you have the **ecs-demo**. click on "Infra", then click on "ecs-ec2.yml". then click on "open"
-	click on “next”
-	stack name: **ecs-demo-ec2-stack**
-	Desired capacity: **1**
-	ECSAMI: **/aws/service/ecs/optimized-ami/amazon-linux-2/recommended/image-id**. ***{This AMI is because the cluster is carrying some EC2 wherein, that EC2 has containers in it. So the cluster has Docker, Docker engine and EC2 Agent which helps it to interact with the cluster. That is why it has to generate this AMI}***
-	Environment name: **ecs-demo**
-	Instance type: **t2.small**
-	Max size: **3**
- Click now on “next”
- Configure option:
  - click on “next”
-	Check the box on [Acknowledge]
-	click now on “create stack”
  - It will take some time to create
    
-Go now to CloudFormation and see the resources under creation.
-	Click on [output] to see the cluster that we exported
-	***{in the Demo, if you see Rollback, go to your resource and click on "Event". it will show you why you are having such a Rollback}***
-	
**[Deletion]**
-	Go to CloudFormation. Select the **ecs-demo-ec2-stack** then click on "delete". ***{If you say you will only delete the EC2 Instance, because it has an autoscaling group, it will still create another EC2 instance. So, its better to just delete the stack and everything will be deleted}***
-	
Now let's understand the differences between EC2-based cluster and Fargate-based cluster by creating fargate cluster using the console and CloudFormation script.
Before that let's first see Task definition and Service in ECS-EC2 mode.

- # Task definition and service in ECS-EC2  mode
- **Task definition:** Your containers are defined in a task definition With properties like memory, network mode etc.
- **Service:** Service will orchestrate the required containers as part of the tasks.
- 
# 1)	Creating the task definition first: 
- Now, we are going to create task definition and launch tasks in the above created clusters.
- So, go to your AWS console and search for ECS console
- But, if you had deleted the stack, then we have to create it back as we need to use it so as to create the cluster first
- 
# So let's create the stack again
- So, log into your AWS console
-	click on CloudFormation
-	click on “create stack”
- New resources (standard) click on this.
-	Select or upload a template file:
  - Click on [choose file]
  - elect "infra"
  - Select “ecs-ec2.yml”
  - Click on “next”
 - Stack name: **ecs-demo-ec2-stack**
- Desired capity: **1**
- ECSAMI: **same as in behind a back page**
- EnvironmentName: **ecs-demo**
- Everything same as behind
- Click on “create stack”
- Now, if you go to ECS console you will see the **ECS-demo-EC2** cluster that we have just created. But you will see everything there 0 0 0 as nothing is running. We haven't created any resources here yet.
- Click on the cluster itself, and when you click on EC2 instance between Tasks and Metrics, you will see that there is no EC2-instance that got registered here.
- But if you go to instances under EC2 console, You will see that one EC2 instance is in the process of being created.
- Once it gets created, it will automatically feature in the cluster or in the **EC2-demo-ec2** cluster.
- It would equally install an **ECS agent**, and **Docker**
- And then, it will register this EC2 instance in the ECS cluster.
- Note that, if this EC2 instance doesn't have an IAM rule attached to it, It will not be able to attach to the ECS cluster. That is the reason why we went to our CloudFormation script and included the IAM Role which we had earlier set it up and use it as part of the **ECS-ec2.yml** file.
- Now, let's now go to our task definition and service.
- 
- # Tasks definition and service in ECS-EC2 mode.
- # Task definition:
- Your containers are defined in a task definition with properties like memory, network mode etc.
- # Services:
- Service will  orchastsrate the required containers as part of the tasks.
- # Creating the task definition first
- Go to your ECS console
-	Now click on “Task Definitions” on the left below cluster
-	Then click on "create new task definition"
-	Select "launch type compatitibility"
- Select “EC2” or "EC2 based cluster"
- **{FOR EXTERNAL---- If you have some on-prem servers that you want to intergrate them to the cluster, you will need to install an ECS-Agent on your cluster. Then you configure AWS CLI Secret Key and ID, So that you will be able to use it to launch the On-prem server on the ECS Cluster. That is why we have "External". In that way, we can enable the communication from On-prem to the cloud}***
- 
-	Click now on "Next step" after selecting EC2
-	Configure Task and container definition
•	Task definiriton name: **ecs-demo-ec2-task-def**
•	Task role: **ecs-demo-task-execution-role** **{ If Task wants to communicate with other services like S3 for eaxample to get some invoices for the Application code like python, boto3, etc into S3, it needs such Task Role to be able to do that. So this one id for the Application to interact with those AWS resources like S3, DB etc. If your Application is not using any of those AWS Services, you can just put "None" here. But if your application is using any of the aws services, then we will put the Task role, so that Task will push the services to S3 and DB etc}***
•	Network mode: default
•	Task execution IAM Role ***{ This task role will help Task to grap any Docker images form the ECR. It will also help to push any log to CloudWatch}***
- Task execution role: **ecs-demo-task-execution-role**
•	Task execution role: **ecs-demo-task-execution-role**
•	Task size
  - Task memory (miB) ***{leave it blank}. 'we are going to assign these very soon as we shall add them in the container'***
  - Task CPU (unit) ***{Also leave it blank}. 'we are going to assign these very soon as we shall add them in the container'***
•	Click on “ Add container”
- The portal that pops up will prompt us to input container information
- [add Container]
-	Container name: **ecs-demo-container**
-	Image: **vamsichunduru/ecs-demo:v1**
-	Private repository authentication: 'Leave this box blank or unchecked. If you are drawing the image from a private repository, then you can check it and then you pass the ARN of the Repo or the Secret Manager information under'.
-	Memory limits: [soft limit]: [128]  ***{Make sure you type the "128" out}***
-	
-	# [EXPLANATION]
**Soft limit:** On this memory limit, we are saying that for this container above we need 128 MB before compute capacity to be reserved for this container. Since it is just a small computer.
Hard limit: When do you put hard limit at 256 for example, you are saying that at any given point in time, this container above should not exceed 256 compute memory size of 256MB
So in our case, We recommend hard limit and maintain soft limit because we don't want to block our container to any hard limit.
Port mappings: Host port.                              Container port: 80
-	Click now on “advanced configuration” 
Essential
-	click on “add"
-	click now on “create”    
to see the version task definitions so that you can quickly grab the old version .
-	Click on task definition it will show you the various tasks that you have
-	then click on the name of the tasks you want and then you will see the different versions of it so then you can then pack the old version that you want.
We are done with task definition. Let's now run the task definition by creating the service.
Running task definition   
Here we are going to run task definition and create the containers as mentioned in task definistion.
So go to that, go to your ECS console.
-	Click on “task definition” on the left
-	click on the task definition that you want to run. In this case click on the tax definition that we have created which is ecs-demo-ec2-task-def
•	Then select the revision that we want to run (we want to run the latest version) so select the latest revision.
•	Then click on “action” at the top
•	then you click on “run task”
-	launch type: 0 select EC2
-	Cluster: ecs-demo-ec2
-	Number of tasks: 2
-	Placement templates: AZ balance spread
-	Enable ECS managed tags.
•	Click now on “run task”
Then you will see that the two tasks we created and now running.
•	Now click on the first task ID to see all the details. If you go under container and click on the small arrow you will see the container details as well
If you copy the IP address with the host port under external link and take it to a new browser, Our simple application is not exposed on top of our EC2 instance as it is well visible or expose on the web. And it is exposed through that dynamically created port.
-	If you click on the second task and do the same you will see that it has been exposed to another host port 49154. An if you copy the IP address plus the pot under external link and take it to a browser, you will see again our application still exposed on top of our EC2 cluster In the web being exposed through a  dynamically created port.
-	Now the created containers can be verified by logging into the EC2 machine and verify docker images and docker containers on the instance by using the below commands.
To expose our application on the web browser, We shall map this external link or we shall take this external link and map it with the domain name Which will then be mapped to the ALB that tagged together those EC2 and is routing traffic into the container Find within those EC2 so that whenever somebody type facebook.com for example it will come to the DNS and then to the  ALB and get into those EC2 then to the container or task or ports.
External link-DNS-ALB-EC2,EC2.EC2.
•	Like we said the container can be verified by logging into the EC2 machine and verifying the docker images and docker containers on the instance.
So navigate to the EC2 instance that got created and ssh into it.
-	Then do Sudo su-
-	Then run this command “docker images”
you will see that our docker images have been pushed to the cluster or on top of the ECS cluster
you will also see the image of the agent.
-	Now do Docker ps-a
So as to see the running containers. here you will see the 3 running containers which are part of our task. You will see their respective ports in which they are running on. (Now you see the flexibility we are all talking about when it comes to ECS EC2 cluster. As you can easily ssh into the instance and see what is going on there. As well you can add the level of security if you want to do it)
Once we have understood the task definition and task working behavior, Lets now stop the task by clicking on the stop option in the task.
So go to Amazon EKS-cluster-tasks where you see the 3 task running.
-	Select the 3 task
-	Then click on “stop” to stop those task
-	Are you sure you want to stop the following tasks…..
•	Click on “stop” to stop all the task
if we now reload it we will not see any running task.
So when we stop the task we can see that is not automatically creating any replicate task or not blocking any termination of task; so in order to manage such actions and to keep a certain number of tasks running at any given points of time and to export task we use a concept called service in the ECS
So if it was a running application and something happens where in this task got stopped, then our users will not be able to access the application. That will be frustrated. That is why in order to avoid that scenario we use service which is the next ECS component.
So service will make sure that at any given moment of time the desired number of tasks prescribed are up and running. All the same time service will help us to expose these tasks through the end word on the LB and we can still put some auto scaling group along this service as well.

Service or ECS service
now after seeing the importance of service, let's now go ahead and create the ECS service by accessing the services and click on create.
So still in the ECS console,
-	click on cluster and then you click on
-	Select the cluster that we created ecs-demo-ec2
-	Then you click on “services”
-	then click on “create” which is just directly under service
•	Configure service
-	launch type
-	select EC2
-	task definition: ecs-demo-ec2-task-def
-	revision: Choose the latest revision
-	cluster: ecs-demo-ec2
-	service name: ecs-demo-ec2-service
-	Service type, select REPLICA
-	number of task: 2
-	minimum healthy person: 100
-	maximum percent:200
-	deployement circuit breaker: disabled
•	deployments
-	Deployement type: rolling update
•	Task placement:
-	Placement templates: AZ balance spread
•	task tagging configuration
-	enable ECS managed tags
-	propagate tags from: Do not propagate.
•	Click on “next step”
•	Load balancing
-	Select none (For now select none we shall later see how to integrate LB)
•	click on “next step”
•	set auto scaling ( optional)
-	select: do not adjust the services desired count
•	click on “next step”
•	review
-	click on “create service”
-	click on view service to see the tasks running automatically.( This is because we have created the task definition while creating the services on the service we have just created. That is why task is not running automatically).
Now we can access the task by clicking on the task. We can see the task details and the external link of the container which is used to access the applications
Just for practice
now let's go back to the cluster main screen and click on the clusters name ecs-demo-ec2 and then stop the running tasks. You will see that service will automatically create a new tax.
-	Cluster. Click on their name ecs-demo-ec2, task, stop, stop
-	Click on refresh again and again (you will see that another task is just getting created to replace a stopped task.)
Importance of service 
1)	 service will always maintain the number of tasks and ensure there are up and running even if they are killed
2)	Service will also perform the ALB formation with those task
3)	service will also perform the ASG of the container within the tax.
With the above steps, we have successfully created the ECS, EC2 based cluster, service ,task definition and container definitions.
The question is
Pods are just from a task
So the question is only kubenetes?
•	Because many people are still using K85 and because K85 has evolved where there are not yet ECS and EKS And also because in K85 we can do more complex things than in ECS and EKS Which is still the advantage that K85 has over Docker Swam
•	Note that although EKS in AWS, it is build ontop of Kubernetes.
We can now delete the cluster if we want
a)	go to services. Select the service ec2-demo-ec2-service
-	Then click on “delete”
-	this will automatically delete the task connected to it.
b)	Now go to cloud formation console
-	Select our cluster stack ecs-demo-ec2-cluster
-	Then click on “delete” to delete it.
Always make sure that you copy and paste without spaces so that it should work without any issue

- ECS fargate based cluster via AWS console



