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

# --------------> EXPLANATION HERE ------------------
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
# -------------> END OF EXPLANATION --------------

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
-	# -----------> EXPLANATION ------------
- **Soft limit:** On this memory limit, we are saying that, for this container above we need 128 MB of compute capacity to be reserved for this container. Since it is just a small computer.
- **Hard limit:** When do you put hard limit at 256 for example, you are saying that at any given point in time, this container above should not exceed 256 compute memory size of 256MB.
- So, in our case, We remove hard limit and maintain soft limit because we don't want to block our container to any hard limit.
- Port mappings: **Host port:** [**0**] ***{This "zero" is the indication that we are asking AWS to create a dynamic port. Although AWS will create some host somewhere that we dont know and will map it to the 80 port of the Container. If it was a static EC2 Instance, then we could use classical Load Balancer}***
- Container port: [**80**] ***{ Note that, this mapping is for dynamic Port mapping since our ECS Cluster has just one single EC2 carrying different Containers in it. So it has an ENI that has just 1 port called "Dynamic Port Mapping". which the ALB is very helpful and used here because it will distribute traffic to all the Containers in the EC2 Instance. [So classical Load balancer cannot map dynamic ports]. }***
- **There are 3 types of Load Balancers.--- Classical LB; Here, the Load Balancerjust map directly to the Port number of the Instance e.g 8080, although it has been suppressed. --- Application LB; Application Load Balancer is used when there are dynamically created ports. Although there are some host ports somewhere in AWS, since there are internally managed by AWS, whenever ALB is used, then the ALB will then map to those ports dynamically created inside that EC2 Cluster which is carrying diff diff containers. So that it will automatically grap the dynamically created ports inside the cluster and will route traffic to those containers ports inside the cluster.***
- ***In most cases, you will be using Application Load balancer. But when you need a faster network or faster data pocket e.g Netflix.com (for movies) which requires such a fast letency, you should use the "Network Load Balancer***
- ***Note that, for EC2 to get exposed, it does so in an Elastic Network Interface (Internet). Whenever you see IP Addresses within this range 49152-65535, know that it will requires a Dynamic Load Balancer***
- # ----------- End of Explanation ---------------
- 
-	Click now on “advanced configuration” 
- Essential: Thick on this box beside [ ] Essential
-	Then click on “add"
-	click now on “create”
-	  
- To see the version task definitions, so that you can quickly grab the old version,
-	Click on "task definition" it will show you the various tasks that you have
-	then click on the Name of the task you want and then you will see the different versions of it, so then you can then pick the old version that you want.
- We are done with task definition. Let's now run the task definition by creating the **Service.**
- 
# Running task definition   
- Here, we are going to run task definition and create the containers as mentioned in task definistion.
- So, to that, go to your ECS console.
-	Click on “Task Definition” on the left
-	click on the specific task definition that you want to run. In this case, click on the task definition that we have created which is **ecs-demo-ec2-task-def**
•	Then, select the Revision that we want to run. (we want to run the latest version) so select the "latest revision".
•	Then click on “Action” at the top
•	Then you click on “Run Task”
-	launch type: select [EC2]
-	Cluster: **ecs-demo-ec2** {Here, you can use thesame Task to run in Dev, Prod and maybe Acceptance. You do this by selecting it here under Cluster}
-	Number of tasks: **2** ***{lets just say 2 tasks. the intention is to run 2 tasks in this cluster so that we shall see how the ports are created dynamically}***
-	Placement templates: **AZ Balance Spread**
-	Click to check the box on [Enable ECS managed tags]
•	Click now on “run task”
-
# -----------> Explanation ------------
- Now, you will see that the 2 tasks that we created are now running.
•	Now click on the 1st task ID to see all the details.
- If you go under container and click on the small drop down arrow, you will see the container details as well. ***{You will see under that, although we provided just the Container Port, AWS has dynamically created the Host Port 49153 and under the External Link, it has mapped this Host Port to the IP Address of this Container. Although we put the Host Port as "0", AWS has understood that we want dynamic port and it has created the host port for us}***
- ***{The IP Address of this Container is thesame public IP of that EC2 Instance, so that, if somebody want to access the container (which is a running image), they will access it through the Public IP of the EC2 Instance*** {That is why rather than putting some hard coded port, we just put the dynamic port}.
- 
- If you copy the IP address with the Host Port under External Link and take it to a New Browser, Our simple Application is now exposed on top of our EC2 instance as it is now expose ontop of our EC2 Instance, as it is now visibly exposed on the web. And it is exposed through that dynamically created port.
- 
-	If you click on the 2nd Task and do thesame, you will still see that, it has been exposed to another Host Port 49154. And if you copy the IP Address plus the Port under External Link and take it to a Browser, you will see again our Application still exposed on top of our EC2 cluster in the web being exposed through a  dynamically created Port.
-	Now, the created Containers can be verified by logging into the EC2 machine and verify Docker images and Docker Containers on the instance by using the below commands.
-	So, with Task Definition, we can launch as many tasks as we want.
-	In thesame way, with the Image in the ECR, WE CAN CREATE AS MANY IMAGES AS WE WANT.
- ***To expose our Application on the web Browser, We shall map this External Link or we shall take this External Link and map it with the Domain Name, Which will then be mapped to the ALB that tagged together those EC2 and is routing traffic into the containers found inside those EC2. So that whenever somebody type facebook.com for example it will come to the DNS and then to the  ALB and get into those EC2 then to the container or task or ports.***
- External Link--------> DNS ------------> ALB ----------------> 1st EC2, 2nd EC2, 3rd EC2
- ***{so, to manage all those Docker Containers is so challenging, that is the more reason why, we employ this EKS, or ECS to manage these numerous Containers. So that, we would be able to bundle them and tag them and expose them to the outside world}***
- 
•	Like we said, the container can be verified by logging into the EC2 machine and verifying the docker images and docker containers on the instance.
- So navigate to the EC2 instance that got created and ssh into it.
  - Then do `Sudo su -`
  - Then run this command `docker images`
  - you will see that our Docker image has been pushed to the cluster or on top of the ECS cluster
  - you will also see the image of the Agent.
  - Now do `docker ps -a`  ***{So as to see the running containers.}***
  - Here, you will see the 3 running containers which are part of our task. You will see their respective Ports in which they are running on.
  - **Flexibility of ECS-EC2 Cluster**. (Now you see the flexibility that we are all talking about when it comes to ECS-EC2 Cluster. As you can easily ssh into the instance and see what is going on there. As well you can add the level of security if you want to do it)
- Once we have understood the task definition and task working behavior, Lets now stop the task by clicking on the stop option in the task.
- So go to **Amazon EKS-cluster-tasks** where you see the 3 task running.
-	Select the 3 task
-	Then click on “stop” to stop those Tasks
-	Are you sure you want to stop the following tasks…..
•	Click on “stop” to stop all the task.
- If we now reload it, we will not see any running task.
***So, when we stop the task, we can see that is not automatically creating any replica task or not blocking any termination of task; so in order to manage such actions and to keep a certain number of tasks running at any given points of time, and to export tasks, we use a concept called **Service** in the ECS.***

- ***So if it was a running Application and something happens wherein this task got stopped, then our users will not be able to access the Application. That will be frustrated. That is why in order to avoid that scenario we use **Service** which is the next ECS component.
- This shows how our Task and Containers correlate with each other because the Container is inside the task.
- This is the fundamental xtics of Containers. If it stops, that is all about that Container. Reason why we have to always think about a possible version of that Container - So that if the existing one stops, we either rollback or we use an alternative. That is why we have to use **Service** here as our rollback.
- So, **Service** will make sure that at any given moment of time, the desired number of tasks prescribed are up and running. At thesame time **service** will help us to expose these tasks to the end-word on the LB and we can still put some Auto-scaling group along this service as well.
# -------------> End Of Explanation -------------

# Service: or ECS-Service
- Now, after seeing the importance of service, let's now go ahead and create the ECS-Service by accessing the "Services" and click on create.
- ***{Since it is a Daemon, only one Task will be deployed on 1 EC2 Instance, just as 1 Container per Pod or EC2. But its thesame Task which can be replicated to another EC2}***
- So, still in the ECS console,
-	click on "cluster" and
-	Select the cluster that we created **ecs-demo-ec2**
-	Then you click on “Services”
-	then click on “create” which is just directly under service
- Configure service
  - launch type:
  - select [EC2]
  - Task definition: **ecs-demo-ec2-task-def**
  - Revision: **Choose the latest revision**
  - Cluster: **ecs-demo-ec2**
  - Service name: **ecs-demo-ec2-service**
  - Service type: select **REPLICA**
  - Number of task: **2**
  - Minimum healthy person: **100**
  - Maximum percent:**200**
  - Deployement circuit breaker: **Disabled**
- Deployments
  - Deployement type: **Rolling update**
- Task Placement:
  - Placement templates: **AZ balance spread**
- Task tagging configuration
  - Click to check the box on [Enable ECS managed tags]
  - Propagate tags from: **Do not propagate.**
- Click on “next step”
- Load Balancing
  - Select [None] (For now select "none". we shall later see how to integrate LB)
- click on “next step”
- set auto scaling ( optional)
  - select: [do not adjust the services desired count] ***{Here, although auto-scaling will be explianed later, the desired # put under Service will always be maintained (2) as what we prescribed. So, with SERVICE already defined, it really doesnt matter if Auto-scaling is Configured or not}***. {So, Service will always make sure to keep our desired number as "2", nomatter whatever the situation}
- click on “next step”
- Review
  - click on “Create Service”
  - click on "view service" to see the tasks running automatically. (This is because we have created the task definition while creating the services on the service we have just created. That is why task is not running automatically).
 - Now, we can access the task by clicking on the task. We can see the task details and the External Link of the Container which is used to access the Applications
 - 
# --------> Just for Practice --------------
- Now, let's go back to the cluster's main screen and click on the clusters name **ecs-demo-ec2** and then stop the running tasks. You will see that service will automatically create a new task.
-	Cluster. -----> Click on their name **ecs-demo-ec2**, and then stop the running task.
-	You will see that Service will automatically create a new Task.
-	Cluster -------> click on the name **ecs-demo-ec2**,-------> Task -----------> Stop
-	Click on refresh again and again (you will see that another task is just getting created to replace a stopped task.)
-	
# Importance of service 
1)	Service will always maintain the number of tasks and ensure that, there are up and running even if they are killed
2)	Service will also perform the **ALB** function with those task
3)	Service will also perform the ASG of the container within the task.

- With the above steps, we have successfully created the ECS-EC2 based cluster, service ,task definition and container definitions.
- **The question is "Pods are just as Tasks in ECS", then why are people still using Kubernetes?**
- Its all because many people are still using K8s and because K8s has evolved where there are not yet ECS and EKS; And also because in K8s, we can do more complex things than in ECS and EKS Which is still the advantage that K8s has over Docker Swam
- # Note that although EKS in AWS, it is build ontop of Kubernetes.
We can now delete the cluster if we want
a) To do that, go to services. Select the service **ec2-demo-ec2-service**
-	Then click on “delete”
-	this will automatically delete the task connected to it.
b)	Now, go to CloudFormation console
-	Select our cluster stack **ecs-demo-ec2-cluster**
-	Then click on “delete” to delete it.


# -----------> ECS fargate based cluster via AWS console -------------

- Before going to create the fargate based cluster let's first start by creating the Network stack if you had already deleted your stack previously.
- Since the stack need some rule we shall equally create this IAM  role.
•	But since Vamsi keeps updating the repo and adding new things clone, this repo at this stage for usage: https://github.com/Kenneth-lekeanyi/ecs-demo

Now let's proceed to create the stacks.
 - Network stack
 - IAM stack
- finally we shall create ECS-EC2 stack, where we shall launch the Fargate just as we launch the EC2 ontop of the ECS Cluster too.
- So, go to CloudFormation and click on “stack”
- click on “create stack”
- click on "with new resources (standard)"
- then click "upload a template file"
- click now on [choose file].
  - Choose “**ecs-demo**” in the cloned repo
  - Go to “infra”
  - Select “VPC”
  - Then click on “open”
  - Click now on "Next”
  - stack name: **ecs-demo-network**
  - click on “Next”
  - click again on “Next”
  - Review
  - click on “create stack”
  - 
Now let's proceed to the next step which is for IAM creation so let's now go to IAM stcak creation
- So, go to CloudFormation again and
-	click on "stack"
-	click on "create stack”
- select "with new resources (standard)"
-	Click on "upload a template file”
-	upload a template file
  - choose file
  - go to "ecs-demo"
  - click on "Infra"
  - click on "IAM"
  - CLICK NOW ON "Open"
-	click on "Next"
-	stack name; **ecs-demo-iam**
-	click on "Next"
-	click on "next"
-	Review
-	acknowledge the box
-	click on "create stack"
-	
# Now, let's proceed to create ECS-Fargate cluster
And to do that,
1)	we have to navigate to the ECS console
-	then click on "clusters"
-	click on "create cluster"
-	select [cluster template]
- click or select “Networking only” for AWS Fargate
-	click on "Next step"
  - Configure Cluster
-	cluster name: **ecs-demo-fargate-cluster**
-	click on "CloudWatch container insight"
  - Check the box on [Enable Container insights]
  - click now on the create"
- we have successfully created ECS-Fargate-based cluster. If you click on “view cluster", we shall see the cluster details. ***{So we see that with Fargate, it is pretty simple and we dont have all the Configurations that we had with the EC2 based cluster wehere we had to do a lot of instance configurations. This is purposely because ECS-Fargate is serverless and then AWS will manage all those EC2 Configurations at the backend. So we are just only creating and managing our stack ontop of this cluster.}***
- you can now proceed to delete this fargate cluster.
- 
-	so still on the ECS console
-	click on cluster
-	then on the "ecs-demo-fargate- cluster"
-	Then click on “delete cluster”
-	then type “delete me”
-	delete
-	
# Now, we are going to create the same ECS-Fargate cluster based using CloudFormation Automation.
- So to create the ECS-Fargate cluster using Cloudformation
- go to CloudFormation Console
-	click on “create stack”
- Click on "with new resources (standard)"
-	specify templates
  - select or click on “upload a template file”
-	Upload a template file:
  - [choose file] (click here)
   - click on "Ecs-demo"
   - Click on "infra"
   - hen you click on “ecs-fargate”
   - then click on "open
   - Click now on "next"
   - Stack name: **ecs-demo-fargate-stack**
   - Click on "Next”
   - Configure:
   - Click on next”
   - Review
   - click on "create stack"
- We are now done creating a ECS-Fargate using CloudFormation scripts. We shall now proceed to see how to use the cluster. i.e in task service.
- 
# Now, let's start by creating a Task definition for Fargate Cluster
- Note that, this is quite different from the Task definition that we saw under ECS-EC2 based cluster.
# Tasks definition for Fargate Cluster
- So, in order to create these Tasks definition, go to ECS console
-	click on "Task Definition”
-	click on “create a new task definition”
-	select launch type compatibility:
   - select or click on “FARGATE”
   - --------> External Explanation ----------
   - This is when we face the situation on ("On-premis" server), To connect those On-Premis server to the ECS Cluster, we can use Hybrid Activation under SSM; But eith that, this method is quite simple, where we go to the server on-prem and install AWS CLI. Then we configure AWS COnfigure rule where we generate the Secrete Key and new uswer ID. So that these ECS Agent will use the Secrete Key and secured ID to discover the ECS Agent. Hence, AWS has provided the flexibility to interact with on-prem services.
   - ---------> End of Explanation ----------
   - 
- then click on "Next step”
-	configure task and container definition
- Task definition name: **ecs-demo-fargate-task-def**
- Task role: **ecs-demo-task-execution-role**
- Network mode: **awsvpc**
- Operating system family: **linux**
- Task execution rule: **ecs-demo-task-execution-role**
- Task memory (GB) : **0.5 GB** {If the Application is big, we should select a bigger GB like 28. But since our Application is small, we just select 0.5 GB}
- Task CPU (VCPU): **0.25 VCPU**  {Here also, if the Application is big, then as we selected a big GB of memory, we should also select a big VCPU}
  {With Fargate, these are the only part that we are paying for. (All jointly called "Compute Power"}
- As you can see, the container is just attached to Task. That is why immediately after finishing with Task, we just go to container on the same page.
-
- So click on ”Add Container” which is nothing but container definition.
- So, a page will pop up, which is container definition page.
   - Container name: **ecs-demo-fargate-container**
   - Image: **vamsichunduru/ecs-demo:v1** {Here, we are just graping the container name that he has created and saved in his DockerHub. (Go to hub.docker.com)
   - Private repository authentication: [Live it blank]. We are leaving it blank because we are using a public repository. But if the image is from a private repository, we will have to check this box and provide the secret manager ARN or name of that Repo.
- Memory limits (MIB): [soft limit] [125]. (Inshort, we leave it blank because we have already defined this under Task Definition.)
- Port mappings: **80** ***{Thisis because thesame container will be put as Host port since Fargate is running dynamic port mapping. Under ECS-ec2, we deliberately put "0" and "80" because we were imposing that it should use dynamic port mapping}***
- Essential: [ ] check this box
- Log Configuration: Check the box on [auto-configuration cloudwatch logs]
- Click on “Add”
- Now click on “create”
- We see that the Task definition with the name, **ecs-demo-fargate-task-def** has been created successfully.
-	If you click on “view task’ you will see it
- If you click on the "version:1" and then click on "Json", you will see the template that creates such Task definistion. You can also see that we provided Port 80 and it automatically create the host port as well.
- Now, we are done with this task definition and in order to run this task definition;
- 
-	Click on "Action”
-	then you click on "Run task”
-	launch type; select **Fargate**
-	Operating System: select **linux**
-	Platform version; **latest**
-	Cluster: **ecs-demo-fargate** (This drop down here is because you can use this task for another name like if you use it for Dev, you can still come and use this tasks fro Prod etc. So that this Task will be launched in the cluster. So, for now, our intention is to launch this Task in our ecs-demo-Fargate cluster. So keep it as it is.)
-	number of task: **2**
-	cluster VPC: ***{Go under CloudFormation, select 'ecs-demo-network' stack. Then click on "output". It will show you the VPC ID that this ecs-demo is using. So, you not this ID. then select that particular VPC ID here under Cluster ID.}***
-	subnets: Select both subnets (Select all the 2 subnets)
-	click then on “Run Task”
- "created task accessfully"
- you can see the 2 task
- refresh to see that they're all running
- click on the task and click on "open link" in a new web. Scroll down to locate it public IP , select it and click on it to take “go to public IP” It will take you to the
**Hi there!!!*** **Welcome to ECS Devops Demo** (Which is our Application). We can access the Application by just using the IP Address and hitting "Enter" because of dynamic Port mapping runing internally on Port 80:80.

# ----------> Explanation -----------
- This is different with the VPC based networking and bridges based networking



Once we have understood the clone and the differences between the EC2 based on this definition an fargate based tax definition click now on the step to delete the running task and to create task through service in the next session.
So select the stacks you have and go to the top and click on “stop”
-	Then click on “stop” on the pop up page. You see while after some minutes that all tasks are stopped. And ECS is not creating or running at task  again. Everything is stopped. so at time maybe something happen Maybe the container is unable to pull the image etc and something happened and task get stop and Nothing is making anymore. So users will not be able to access this application anymore.
So in order to maintain this desired number of task running always even if some are killed  or get stopped, we are going to use a specific component call “service”
So you need to understand that there is no mandatory requirement that you must use service. Where task is running that is fine. But what if something happens.
The whole application stops running.
So that is why we now go ahead to configure service
service definition for Fragate cluster
So now let's create service by clicking on service category
so stay under cluster in the ECS console, Click on “service” just beside “task”
-	Click on “create” just below it
-	configure service
•	launch type 0 select fargate
•	operating system: linux
•	Task definition;
Family: ecs-demo-fargate-task-def
Revision; select the latest version (latest)
•	Platform version: LATEST
•	Cluster: ecs-demo-fargate
•	Service name: ecs-demo-fargate-service
•	Number of task:2
•	service type: REPLICA
-	deployments
deployment type. Rolling update
-	Task tagging configuration
Enable ECS managed tags
-	Propagate tax form: (Do not propagate)
-	Click on “next step”
•	configure network
•	cluster VPC: get the VPC ID from the cloud formation stack then go to output you received a VPC ID there. Group it or not it then you select it here.
•	Subnet: select both subnets
Keep the other remaining settings as they are, nothing to change.
-	Click on “next step”
-	Set auto-scaling (optional)
-	Click on “next step”
-	Review
-	Click on “create service’
You can see that service has been created successfully.
Now click on “view service”. You see the task running  and another one pending. To access the task, On ie you are In the service page, click on the task, group it public IP and paste in a Browser you will see our application.
Hello or Hi there.
So if you go to status you will see that it is active and the desired task is 2  then running task is 1.
If you now Go to task, you will notice that On the pending task get killed, service will try to provision a new task. So service will automatically invooke a new task to replace the old or killed task.
Exercise. If you go to task,select all the running task and stop them, within few minutes service will provision new ones and get them up and running automatically.
So after understanding the service and task, let's terminate the tax completely.
An in order to do that see under service click on the service to select it.
-	Then you click on “Delete” in the middle of update and action
-	Then type “delete me” and click on delete. This will delete the service and also go and delete the task that are running.
Now go to cloud formation, click on ecs-demo-fargate-stack
-	then click on delete”
-	then click on delete”
•	now keep the ecs-demo-iam as it is.
•	Go below that and click on ecs-demo-network and then ypu click on “delete”
Then click on “delete stack”
We shall later create all these things with cloud formation as our focus now is in ELB and A5Co.
So for now keep your ecs-demo-iam.
•	With the above we have completed the ECS basics (i.e  ECS cluster, ECS cluster types (EC2 fargate), service, task Definition and container definition.
Let's not proceed to section 2
which has to do with load balancing and auto scaling.
•	When you cannot delete VPC, it means that task was being created VPC too was created and 5G too go created and mapped to that VPC.
Reason why you are unable to sync Lee delete the stack because the VPC has some dependencies. Dependencies file 5G will permit it from being deleted.
So to delete it first go to the VPC console and try to delete the 5G first before you come to cloud formation and try to delete the stack.
•	At Work first create a security group, name it before you attach it to the network .

Section 2: AWS ECS load balancing and Auto scaling load balancing with ALB.

