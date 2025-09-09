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
- This is different with the VPC based networking and bridges based networking. As the containers using Bridge, Networking ia using Dynamic Port mapping running internally on Port 80:80. If you need to hit the IP Addresss in a Browser and it takes you to the Application running internally on Port 80 of the Task (ENI) and Port 80 of the Container(all internal). But with VPC based networking, you will need to enter the IP Address:Host Port number in a Browser to be able to access the Application.
- Now, when you go down to the **"Container"** and click on the small arrow to see the Container's information, down closer to Key ------> Value, you will see **"View logs in CloudWatch"**. If you click on that hyperlink, you should be able to access the log group.
- Try to generate some logs again by
   - clicking on the task
   - click on the small arrow, then click on the public IP to access the Application and generate some logs
   - then you click on "View logs in CloudWatch".
   - You will see details about the logs starting from the details of the Container entry point, etc.
     
              ----> Just a small Excercise here ---->
     - Go to Google and type "MyIP"
     - open the first link that says "My Ip address.com", it will show you your IP Address.
     - Then, you go to the logs, you will see that your IP Address access the Application,the date, etc
     - This is how you should be able to see and interprete logs groups
            ------> End of small Excercise -------->
       
  - When you are working and something, the log will highlight it. But they will be so many logs. So, for you to filter the logs, **copy the key words or line in that hightlight log, then go up to the box up that you see [filter], paste it inside this box inside the double quote, then go beside on the small calender and click on "Absolute", to enter the time frame and date. Then go to "Relative" whichis maybe and click on "10" to mean "last 10 minutes" or "last 2 hrs". Then you click on "Apply"
  - So that, CloudWatch will filter those particular lines of logs of that event.
  - If it is a "404 Not Found", for example, you may still copy it but in the box like [Q: "404 Not Found"], then you can click on the "Create Metrix Filter" to generate the Filter and send to your team mates.
  - **Once we have understood the flow and the differences between the EC2-based on Task definition an Fargate-based Task definition, click now on the **stop** to delete the running task and to create task through Service in the next session.**

- So, select the stack(s) you have and go to the top and click on “stop”
-	Then click on “stop” on the poped up page.
-	You will notice after some few minutes that all tasks are stopped. And ECS is not creating or running any task  again. Everything is stopped.
-	So, at time maybe something happen, Maybe the container is unable to pull the image etc, and something happened and task get stop, and Nothing is working anymore. So users will not be able to access this Application anymore.
- ***So, in order to maintain this desired number of task running always, even if some are killed  or get stopped, we are going to use a specific component call “**Service**”
- ***{So, you need to understand that there is no mandatory requirement that you must use "Service". Where task is running, that is fine. But what if something happens. The whole application stops running. So that is why we now go ahead to configure "Service"}***
# ------------> End of Explanation --------------

# Service definition for Fargate Cluster.
- So, now let's create service by clicking on Service category
- so still under "Cluster" in the ECS console, Click on “Service” just beside “task”
-	Click on “create” just below it
-	Configure service
  - launch type: select "**Fargate**"
  - Operating system: **linux**
  - Task definition: Family: **ecs-demo-fargate-task-def**
  - Revision: **select the latest version (latest)**
  - Platform version: **LATEST**
  - Cluster: **ecs-demo-fargate**
  - Service name: **ecs-demo-fargate-service**
  - Number of task: **2**
  - Service type: **REPLICA**
  - Deployments:
    - Deployment type: **Rolling update**
  - Task tagging Configuration:
   - Chcek this box on [Enable ECS managed tags]
  - Propagate tags from: [Do not propagate]
-	Click on “next step”
- Configure network:
  - Cluster VPC: {***get the VPC ID from the CloudFormation stack, then go to output you will see the VPC ID there. Grap it or note it. Then, you select it here.***}
- Subnet: **select both subnets**
- Keep the other remaining settings as they are, nothing to change.
-	Click on “Next step”
-	Set Auto-Scaling (optional)
-	Click on “Next step”
-	Review
-	Click now on “Create Service’
-	You can see that service has been created successfully.
-	
- Now click on “view service”.
- You will see the task running  and another one pending.
- To access the task, Once you are In the service page, click on the "task", grap its public IP and paste in a Browser, you will see our Application.
# -------> Hello or Hi there.-------->

- So if you go to "status", you will see that it is "Active" and the desired task is **2**  then running task is **1**.
- If you now go to "Task", you will notice that as the pending task get killed, Service will try to provision a new task. So Service will automatically invooke a new task to replace the old or killed task.
- {**Now, you will see only one Task running and another one is running and stopping. That is quite OK. This is because Vamsi's DockerHub that we pulled the image from has a pull request limit configured by him. So, with so many people calling that image at that very time, it starts limiting the pull request. But dont bother because when we start the Pipeline, we shall call the image from ECR and not from DockerHub.**}
- So, at work, if you face that problem, go to "Edit Container", after the image, click on "private repository authentication" and log in, so that you will not face that kind of issue anymore.

# -----> Exercise.-----> 
- If you go to task, select all the running task and stop them, within few minutes Service will provision new ones and get them up and running automatically.
- So, after understanding the service and task, let's terminate the task completely.
- And in order to do that, still under Service click on the "Service" to select it.
-	Then you click on “Delete” in the middle of update and action
-	Then type “delete me” and click on delete.
-	This will delete the service and also go and delete the task that are running.
- Now, go to CloudFormation, click on **ecs-demo-fargate-stack**
-	then click on "Delete”
-	then click on "Delete”
-	
- Now, keep the **ecs-demo-iam** as it is.
- Go below that and click on **ecs-demo-network** and then you click on “delete”
- Then click on “delete stack”
- We shall later create all these things with CloudFormation as our focus now is in ELB and ASG.
- So for now keep your **ecs-demo-iam**
- 
- With the above, we have completed the ECS basics (i.e  ECS clusters, ECS cluster types (EC2 fargate), service, task Definition and container definition.
- Let's not proceed to section 2, which has to do with Load Balancing and Auto Scaling.
- 
- When you cannot delete VPC, it means that task was being created VPC too was created and SG too got created and mapped to that VPC.
Reason why you are unable to singly delete the stack because the VPC has some dependencies. Dependencies file like SG will prevent it from being deleted.
- So, to delete it, first go to the VPC console and try to delete the SG first before you come to CloudFormation and try to delete the stack.
- **At Work, first create a security group, name it before you attach it to the network.**

# ------> Security of our Application -------->
- In our task Security Group ID, We allow that Task Security Group ID as our Source. Then, we open a Saved port. And then on our ALB, WE ALLOW ITS SOURCE FROM 0.0.0.0/0, So that all end users can access the load Balancer.
# -------> End of Security of Our Application ---->

# Section 2: AWS ECS Load Balancing and Auto Scaling Load Balancing with ALB.

- Before we proceed clone this git repo to get the latest code as we are going should be working on the new section: https://github.com/Kenneth-lekeanyi/ecs-demo
# -------> (https://github.com/cvamsikrishna11/ecs-demo)--------->
- ***{You can go to your GitHub and delete the old repo before cloning this new repo.}***
- Now let's try to understand the Load Balancing concepts by creating a stack with CloudFormation scripts first. Then we can discuss the flow of the load balancer.
So:
# create a VPC, ECS an ALB with cloud formation stack
so,
- Go to CloudFormation console and click on “stack”
-	then click on “create stack” 
  - [with new resources (standard)]
-	Prepare template:
  - click as **template is ready**
-	Template source:
  - click on [upload template file]
-	Upload a template file
  - [choose file] (click on choose file)
  - **ecs-demo**
  - click on **infra**
  - click to select **vpc-alb-ecs**
  - then click on "open"
  - then click on “next”
- Stack details:
-	Stack name: **ecs-demo-vpc-ecs-alb**
-	Parameter:
- Keep environmentName as it is; **ecs-demo**
-	click on next”
-	click again on next”
-	Review
-	click on “create stack” (this will take time to create)
- once the stack gets created, we can see the created VPC, ECS cluster and ALB on their console respectively.
- Under CloudFormation,
•	You can click on the “Resource” section to see all the resources that are being created
•	Go to the ECS console, you will see the cluster that is being created.
•	Click on the output section of CloudFormation to see the resources that are outputed.
- In the next section, we shall see how the ELB is routing traffic to the Tasks inside a Cluster
- So, the LB will when configured be detached to the public subnet, which will then route Traffic to the Task or Port where containers are seated.
# ---------> BUT ---------->
- When we use a Target Group, the ELB will be mapped to the target Group and then from the Target Group before the traffic would be routed to the public subnet containers from the target group.
  # Users -----> ELB ----> TG ------> (EC2), (EC2).
  
# Now, create Fargate-Service Task and map the Task with a ELB.
- Here, we are going to create a Fargate Service. We create a Fargate Task and then we shall then map the created Tasks to the ELB.
The github repo is: https://github.com/Kenneth-lekeanyi/ecs-demo/blob/master/infra/fargate-service-task.yml

- To do this, we have to create a new stack.
- So go again to the CloudFormation console
-	click on stack and click on “create stack”
  - [With new resources (standard)]
-	prepare templates:
  - [template is ready]
-	specify template:
  - [upload a template file]
-	upload a template file:
  - [choose file] click here
- "infra"
  - select **fargate-service-task**
-	Click on "open"
-	then click on “next”
-	specify stack details:
   - stack name: **ecs-demo-fargate**
-	parameter {Thes parameters values can be altered depending on the requirements of the Application in question}
  - Container CPU: **256**
  - Container memory: **512**
  - Container port: **80**
  - Desired count: **2** ***{You can put 4 or 3}***
- EnvironmentName: **ecs-demo**
- ImageURI: **Vamsichunduru/ecs-demo:v1**
- Path: *  ***{bECAUSE WE ARE PUTTING A load Balancer, that LB will be able to have a particular path. by default, we are saying that, whatever traffic you have (*) rout it to the Load Balancer that we are creating}***
- Prority: **1** **{Here, we are saying that, execute this particular function "*" that we have in our path as the first priority, and whatever traffic that it has from the LB, it will route the traffic to Task that we have put in "Desired count" above.}***
- Service name: **ecs-demo-service**  (Since we are creating a service, This is just the Service Name}
- Click on “next”
-	Configure stack
  - click on “next”
-	Review
  - click on “create stack”
# -----> successful------------->

- Once the stack has been created successfully, we can now see the created ECS-cluster service Task and can access the Application by assessing the ALB, DNS URL.
To do this
-	Go to EC2 console
-	Go to Load Balancer
-	select the newly created ALB and copy its DNS name.
-	Paste it in the Browser to see the deployed Application with the LB DNS.
-	You can also access the logs to see the logs that have been generated
-	Note that: These logs are coming from the Tasks. So that whatever is happening inside the Task is being pushed to CloudWatch. So, these logs are reporting whatever is coming from the ALB and getting into task.

# -------> Some Explanation ------>
- **Service:*** This ensures that the desired number of required Tasks are up and running. And also register the Task to the ALB, ASG to connect to external world.
- **Task Definition:** Roles networking, Compute power such as VCPU, Memory etc Revision in Task;
- when Service creates a new Task to replace the Task that get killed, such a newly created Task comes in as Revision as a revised copy of the old Task that got killed after executing its function. So basically, Revisions are different versions of Tasks.
- **Container definition:*** This will have the Name of the Container, URI OF THE container, Image name, Volume.
- **ECS Cluster:*** This is just a logical grouping of all these resources (Service, Task, Contianer).
- 
- **Dynamic Port Mapping:** This happens in an EC2 Instance wherein Containers or different Containers found within that single EC2 Instance can be able to communicate using thesame Host Port without being limited to the choice of using that single Host Port which we put as "0".
- Whenever any of the Port is using this dynamic Port, we should expect it IP Address to fall within the range of 49152-6556
- So, whenever AWS sees this "0", it will automatically know that the intention is to use dynamic Port number and the IP address or any IP Address that it attribute in this situation will fall between 49152-6556 which always available for your Task, and not restricting to say "already in-use", so that, whenever you see any IP address falling between this range, know it is using dynamic port mapping.
- 
- Q & A:
- **Question:** What is the difference between the Networking of EC2 Cluster and the Networking of Fargate Cluster?
- **Answer:** EC2 Cluster Networking uses the Bridge Networking, which is the Docker default networking with only 1 AMI attached to Containers which is static port. But the Fargate Cluster uses the AWS VPC Networking of ENI which is dynamic port.
-
- **Question:** If a company wants to build an application like this one that we have been dealing with, what will be required from us the DevOps engineers?
- **Answer:*** Go through this with the Architectural diagram illustration
- **First idea:**
# 1)	First thing to do, is to manually prepare it as we did, so as to understand (So first thing to do is a manual hands-on as we did)
  - Task definition
  - Service
  - Cluster
- So that if you are creating a container you will know that you need an image or container image, (that will be placed in an ECR or DockerHub.
# 2)	Then think about how you will automate it. Whether you will use CloudFormation or terraform or Boto3 or CDK or whatever tool you are comforted with.
- **First job**
# 1a) Then using all these things we shall start to develop the infrastructure first.
# And what are the infrastructure we need?
1)	We need a VPC
2)	we need some IAM Roles
3)	we need an ECS cluster stack
- All these if you are deploying for ECS.
- If you want to use but EKS, it is pretty much the same thing.
  - You need a VPC
  - you need some IAM roles
  - you need an EKS cluster
# if you want to use but EC2 to do the deployement
- you need a VPC
-	you need some IAM roles
-	you need an EC2
# if you want to deploy using lambda, then
-	You need a VPC
-	you need some IAM roles
-	you need Lambda.
-	
- So, first creat your VPC so that the same VPC can be used by all methods like ECS, EKS,EC2 etc.
- Then create your IAM roles too, so that the same IAM Roles can be used by both methods as well if possible.
- 
1)	So with a VPC; first group the cidr ranage, subnet cidr etc. So as to go and prepare your CloudFormation script
2)	then do the same thing for IAM role
3)	then go and develop or create an empty ECS cluster
- you then prepare all these 1,2,3, initial phase and keep
- So that whenever the developers gives you the application code then we just proceed to put up layer (4)
4)	We will then come up with another CloudFormation to create
  - Sservice
  - Task definition
- (with the Application Code, taking the requirements from the application developer).
- This is because, unless we have the infrastructure even if the developer has the Application, we cannot deploy everything. We need the infrastructure to deploy this Application Code that has been build.
- So, as he is building his Application Code, you too the DevOps engineer you are preparing your infrastructure as well. Such that immediately he gives the Application Code to you, you just start with stage 4 to prepare the CloudFormation on
  - Service
  - Task
- From the App, you can build another CICD as well. And knowing that the base infrastructure setup is now good and guaranteed enough, we can deploy our CICD application gotten from the Application Code on top of this base infrastructure.
-
# - What most companies now do is that, they use master method wherein;
- 1) They start by building the **Infrastructure Pipeline** using **Terraform Infrastructure Pipeline**
  2) They develop a Terraform Code that provision ALL the infrastructure resources into a Repository
  3) They configure AWS CLI in their local
  4) They then push the Infrastructure code into the Terraform Infrastructure pipeline and all the Infrastructure get provisioned in AWS Concole automatically called cluster stack. So that you are not doing anything manually.
# 5) They then take the application code, build a CICD Pipeline and deploy to the cluster that is already existingat the Infra level.
- So, everything is automated. Nothing is manually done.
# -------> End of Explanation ------>

# Let's delete the stuff to prevent some changes
1)	go to Load Balancer
-	under the load balancer click on “listener”
-	then click on the minus sign - at the top
-	then click on “delete”
- this role was added by him manually that is why we are deleting it single.
2)	Now go back to CloudFormation
-	click on stack
-	then under the stack name click on the **ecs-demo-fargate** stack.
-	Then click on "delete". When it is completed, we proceed to delete the "ecs-demo-vpc-ecs-ALB". This is because it has a NAT gateway attached to it that will change us as well.
3)	Click on the **ecs-demo-vpc-ecs-alb** under CloudFormation then click on “delete”
 	
# --------> Auto-scaling.--------->

- Auto-scaling will help to scale-out and scale-in the number of tasks within a service to help the heavy incoming traffic flow and to save the cost during less user access by scaling-in.
# Steps to enable autoscaling
1)	Update the service to enable the Auto-scaling
2)	Configure the scaling policy with metrics
   a) Target scaling policy
   b) Step scaling policy
- So, if you did not delete the stuff that's fine but if you deleted them, then let's create them back.
- To create the VPC stack, go to CloudFormation Console again
-	click on “create stack”
  - [with new resources (standard)]
- prepare template:
  - template is ready
- specify template:
  - upload a template file
- Upload a template file
  - [choose file] click here
  - "ecs-demo"
  - click on "infra"
  - choose “VPC-alb-ecs”
  - then click on “open”
- Click now on “next”
- stack name: **ecs-demo-vpc-ecs-alb**
- next
- configure stack options
- click on “next”
- Review
- click on “create stack”
- 
# Let's now proceed to create the Fargate Stack
- so still go to the CloudFormation console
-	click on “create stack”
- [with new resources (standard)]
-	prepare template:
  - Template is ready
-	template source:
  - upload a template file
-	Upload a template file:
  - [choose file] click here
  - "ecs-demo"
  - click on "infra"
  - then click on "fargate service task"
  - then click on “open”
-	then click on “next”
-	stack name: **ecs-demo-fargate**
-	next
-	Review
- Click on “create stack”
-
- # Now let's focus on Auto-Scaling
- we have seen that: **ECS Service---> help us to maintain -----> [desired number of tasks = 2], increase in traffic or decrease in traffic ----> ALB ----> ASG Increase & Decrease.**
- The role of ASG here is that, **Service** will help to always maintain the desired number of task that are up and running. But what if the traffic suddenly rises to say 2 million trying to access only 2 desired number of tasks?
- So, ASG will come in and edit the desired number of tasks to either scale up to 5 or 6 depending on the amount of traffic coming in,. And it will also scale down if traffic gradually reduces to say 1 task by editing the desired number of tasks prescribed to respond to the decrease number of traffic.

# Auto-Scaling Group for Task
- For us to create an Auto-scaling Group for our tasks, we go to Service, because one of the copabilities of Service is to create the ALB, ASG And maintain the desired number of Tasks
--------> Service------> desired number of task
--------> Service------> ALB; security
--------> Service------> ASG; traffic
  
- So, let's go to our service. And to access service go to ECS console
-	Click on "Clusters"
-	Click on our cluster **ecs-demo-cluster** that we have created
-	this will then open the cluster when you will be able to see services
- So, click on our “**ecs-demo-service**”
- once we are on the service page, now click on "Auto-sealing section" to see or check if any auto scaling is already enabled on the service.
- As there is no Auto-scaling enabled as of now, click on "update" to edit the service and add the Auto-scaling.
-	So click now on "Auto-scaling"
-	then you click on "update” (so that we can update the service). This is when we shall configure the Auto scaling task.
-	Configure service;
-	Here, you will see that we have already configured this before. So proceed to the next step
-	click on “next step”
-	configure network
- click on “next step”
-	set Auto scaling (optional): ***{this is the place where we are going to add auto scaling settings.}***
1)	**Target Taking scaling:** T1, T2, T3; When it gets to 70% CPU Utilization, it will stop scaling most task
2)	**Schedule Scaling:** e.g Black Friday, email checking in the mornings etc, scale up within these days and times in the mornings
3)	**Step scaling policy:** 70% whenever CPU utilization gets to 70%, +add 1 step and whenever CPU Utilization gets to 30%, subtract 1 step (Most companies uses this Step Scaling policy, because this is where you have most of the control)
so,
-	set auto scaling(optional)
- now on this page,
-	click on select “**configure service Auto-scaling to adjust your services desire courd**” and then fill in the details here below.
  a) Minimum number of task: **1**
  b) Design number of task: **2**
  c) Maximum number of taxsks: **4**
  d) IAM role for service auto scaling: **ecs-demo-asg-role**
-	Automatic task scaling policies
  - click on “Add scaling policy”
- A page will pop up for you to add the policy 
-	Scaling policy type
  - select “**step scaling**”
  - policy name: **ecs-demo-scale-out-policy**,
  - Execute policy when: **create new alarm**
  - Alarm name: **scale-out-alarm**
  - ECS service metric: **CPU Utilization**
  - Alarm threshold: [**Average**] of CPU utilization >- [2] for [70] consecutive period of [5 minutes]
  - Then click on “ save”
-	Scaling action: [**Add88] [**1**] [task] when [**2**]
- # Interpretations: i.e whenever CPU utilization is greater than 2, add 1 task.
-	Cool down period. [**300**] Seconds between sealing actions.
-	# So we are saying that for every time that you add a task, when CPU utilization get greater than 2, take a break to colddown.
-	 Click now on “ save”
- Then simply proceed to the next step
- Click again on “Add Scaling policy”
- Scaling policy type:
  - **Step scaling**
-	Policy name: **ecs-demo-scale-in-policy**
- Execute policy when: click to select "create new alarm"
- Alarm name: **scale-in-alarm**
- ECS service metric (this means taking alarm of): **CPU utilization**
- Alarm threshold [Average] of CPU utilization [<-][1] "For demo purposes" for [1] consecutive period of [5 minutes]
# Interprelation: So, whenever CPU utilization is les than 1, go ahead and remove 1 task. So whenever this is done, that will trigger the alarm that will go ahead and remove the task.
- Then click on “save”
- Add policy
  - Scaling action: [**Remove**] [**1**] task when (1)
  - Cooldown period [**300**]
- Click now on “save”
- Click now on “next step”
- Review
- Click on “Update service”
- Click here on “view service”
- Click on Auto scaling between [event] and [deployment] to see your AS.
- Our Auto scaling has been created.

# ---------> EXPLANATION ------->
# Now, we can check the individual alarm details by clicking on the scale-out-alarm and scale-in-alarms individually.
- If you click as the “ecs-demo-cluster” and click on "auto-scaling", you will see that the desired count has been turned to “**1**” instead of “**2**”. This is because auto-scaling has gone into effect immediately and it has seen that they is no optimum utiliation of the CPU, so it has triggered an alarm and **1** task has been removed.
- This shows that auto-scaling group is working perfectly.
- 
- Now, to see how the alarm is working, still under “auto-scaling” click on any of the alarms (either the “scale-out-alarm” or on the “scale-on-alarm”). So as to see the state of the alarm in the cloudwatch.  **OR**
- You can go to the CloudWatch console, then you click on “All alarms” you will see all our 2 alarms there. You will see that the **in-alarms** is showing in **Red**, meaning that this in-alarm has been triggered. Whatever condition that we put there has been reached, so the alarm has been triggered.
- Click on “history” to read what is going on there”.
- If you select or click on "Event” you will see what has happened.
- Read the message there in the time frame that it happened
- **Now, as both the Load Balancer and Auto-scaling are enabled, let's simulate some traffic on the Load Balancer endpoint so that it should automatically increase the CPU utilization of the ECS service.***
- 
- ***The objective of this exercice is that, we want to inject some traffic on the ALB, And see how the CPU Utilization will scale up due to the increased load and then Auto-scaling will scale up the number of Tasks.***
- Here is the doc used to simulate the load: https://humansreadcode.com/post/2017-05-29-ab-testing-docker/

- So execute the below commands in order to download Apache-ab and to run the load test using it. For this we are using docker, so it does not matter whether your machine is a Marc or window, as long as you have docker installed and configured, you are good.
- Use the below GitHub link to access the Commands. where we can grap those commands to use: https://github.com/Kenneth-lekeanyi/ecs-demo/blob/master/help/apache-bench-load-testing.txt

- so, to start
1)	Let's download the http-ab image and to run the container through this command.
- `docker run -dit --name httpd-ab -v /var/www/html:/usr/local/apache2/htdocs/ httpd`
- Run that command in your terminal since docker is already installed in your computer.
-	But before running that command, ensure your ducker is up and running. So do
-	`docker ps -a`, to see that the container is up and running
2)	Going to the next command, let's access the bash in the running container through the command.
- `docker exec -it httpd-ab bash`
3)	Let's now generate the load to the Load Balancer DNS. But you have to replace the LB DNS you find in the command with the DNS of your own ALB.
- So, first copy the entire command and take it to a notepad or on a text editor and modify it there by the URL with that of your own Load Balancer. To see the URL of your Load balancer, go to your EC2 Console, locate and click on "load Balancers", Then click to select the newly created ALB and copy its DNS.
- `ab -r -c 500 -n 5000000 http://ecs-d-publi-h9d4ufps3s74-2106267235.us-east-1.elb.amazonaws.com/`  ***{"ab" means apache workbench, "-r" means recalling, "c" means continouous request, "-n" means number of intotals}***

- Go now and group the DNS of your ALB
-	So go to your EC2 console
- click on load balancers
- select the newly created ALB
- Then copy it DNS by clicking on the little copy
- then go to where you pasted the command and place your LB and DNS within the red line.
- Then, copy the whole command and run it in your terminal.
- Ideally instead of putting the DNS of the load balancer you should put your domain name such as **facebook.com** or **netflix.com**
- Now, in a few minutes, we shall see that this scale out alarm would trigger immediately and will reach an alarm state as shown in the CloudWatch alarm section
-	Go now to Auto-Scaling and go to the "Events" section in our cluster.
-	Then, go to task and see what is going on there. Task is getting created way task
-	
-	Then go to service and click on metrics to see how matrix is going up.
-	We can see that the task number got changed from 1 to 2 due to Auto-scaling-out event and vice versa on scale-up or scale-down events.
- **Now, if you have many instances it will be pretty difficult to be taking each instance and then monitoring it alarm. That is why we want to create a dashboard and add the created alarms to the dashboard so that we can easily verify them on the dashboard level for monitoring.**
- 
- In order to do that, access **CloudWatch Console**
-	Then you go to "all alarms"
1)	- Then select any alarm you want (scale-in-alarm)
-	Then click on "Actions" on the top right
-	Now, select by clicking on “Add to dashboard” that populate after you clicked on actions.
-	
- Since we don't have a dashboard yet, on the page the poped up after you had clicked on "add to dashboard” on the left you will see “**create new dashboard**”
-	Name of the dashboard **ecs-demo-asg-alarms-dashboard**
-	then you click on "create"
-	widget type [line….]
-	customized widget title
  - [scale-in-alarm]
-	Click now on add to dashboard”
2)	Now, let's go to **scale-out-alarm** and click on it too.
-	click on "Action" after selecting the scale-out-alarm
-	select a dashboard: **ecs-demo-asg-alarms-dashboard**
-	widget type: **line**
-	customized widget title: **scale-out-alarm**
-	click now on “Add to Dashboard”
-	
- Now go to the top left and under "Dashboard” and click on ”**ecs-demo-asg-alarms-dashboard**”
- 
- As we are done with the Auto-scaling demo, we can stop the Apache bench process by pressing **control + C** or **command + C** when your terminal is up.
- Now do "exit" to come out of that particular container. 
- But if you do `docker ps-a`, you will see that the container is still running with the name **httpd-ab**
-	To effectively kill the container do `docker kill httpd-ab`
-	now verify if the containers are still running by doing this command “ docker ps-a”
- Now as we are done deleting the container, let's go to the CloudFormation console and delete both the **ecs-demo-fargate cluster** stack and the **ecs-demo-vpc-ecs-alb** stack to save cost.
- 
1)	# Delete the ECS- demo-fargate first
-	Cloudformation console
-	Stacks
-	Select "ecs-demo-fargate"
-	Delete
2) **Delete the ecs-demo-vpc-ecs-alb**
-	Cloudformation console
-	Stacks
-	Select "ecs-demo-vpc-ecs-alb"
-	Click on ‘delete”
-	Confirm delete.
- So we have successfully completed Section 2 which has to do with Load Balancing and Auto-Scaling with ECS.
- 
- # Load Balancer ------> Listerner --------> Target Group (T1, T2, T3).
- **Listerner** is used to connect the Load Balancer to the Target Group.
- **How do you integrate the Load Balancer to the Target Group, in such a way that, whenever there is traffic coming into the Load Balancer, that traffic will be routed to the Target Group?**
- You will use a Listerner for that. In the Listerner, you will mention the details e.g on what port? example port 80, you will direct it to the TG. While port 443 for example; if some traffic hit port 443, you should direct it to the TG. So, in the Listerner, you will mention the Port Number. **OR**
- If somebody hit this path, your LB will send itt request to that particular Target Group and some other Task or Containers or pods or EKS.


# Set up CICID pipeline  for AWS ECS Deployment for both Rollout and Blue/Green 
-
# -------> Explanation Here ---------->
- The final Github link that we should clone is here below. so, clone this final repo: https://github.com/Kenneth-lekeanyi/ecs-demo
- Delete the other repos and clone this final Repo.
- 
- **With this repo consider it coming from the Developers. So,  you then clone it into your local as a Devops Engineer**.
- And immediately you (the Devops Engineer) clone the code into your local, you will start to add your own features there such as;
  1) **Dockerfile**,
  2) **Buildspec.yml**,
  3) **infra.yml**
  4) **appsspec.yml**
  5) **taskdef.json** etc.
- Inorder to add it to the base code that was sent to you by the Developer.
- Once you have done so, you will now do the "git add" in order to add the combination of what the Developer sent to you plus whatever you have build in your local staging area to the cetral repository.
- So, to push it to the central repository, you will do 
  - `git status`
  - `git add .`
  - `git commit -m “commit message”`
  - `git push` ***{This will finally push the changes and the configure files to the central repository which is CodeCommit}***.
- Immediately as the code drops in CodeCommit, the build get started immediately, (as cloudwaqtch triggers the event to provock CodePipeline which triggers CodeBuild to start the building process immediately).
- So, you as the Devops Engineer must have prepared the groundwork by building your **Dockerfile** and your **Buildspec**, so much so that, when you push it to CodeCommit from your staging area the building processes are initiated and goes into effect immediately.
- In the CodeBuild, whatever you specified in the **buildspec.yml** file will be build here. Any specification such as doing some sonarquebe checking, lambda function, building the image etc is done here at the level of codebuild.
- When it is done an image of that code will be sent to the ECR  for version storage while the latest version will be sent to the deployment stage taking place in the ECS cluster.
- But, before it gets to the cluster, we can configure it to go through the rollout process as well as also the Blue-green process, taking place between CodeBuild and the ECS cluster.
- Codebuild is using a file called **Buildspec.yml** that you have build to do the action one-after-the other that you have mentioned or prescribed in that **buildspec.yml** command. So this buildspec.yml file is nothing but a group of share commands grouped and arranged together 1-by-1
- In term of prices, CodeBuild charges us depending on the resources that it is running.

# Read more about blue/green deployment here: (https://www.proud2becloud.com/ecs-deployment-strategies-reduce-downtime-and-risk-with-blue-green-deployment/)
- Concerning deployment from codebuild to the ECS, as compared to the EKS where we ran some commands such as `Kubectle apply -f kubect/manifest`. ***This ECS has native intergration between CodeBuild and the ECS cluster Wherein we initiated it in the console and we can further configure the roll-out and a blue/green stage after CodeBuild define the deployement gets to the ECS cluster.***
- 
- So, in the deployment stage we have 2 types of deployments
1)	**Rollout deployment**
2)	**Blue/Green deployment**
- Wherein all this build from codeBuild ,CodePipeline will either use one of them to deploy it either of the individual services found in the ECS cluster which is sealed within a VPC.
- For all these to happen, we need some IAM Roles wherein it will provide enough flexibility for all the CodeCommit and CodeBuild to interact with each other. Basically, CodeBuild needs some roles attached to it for it to be able to push it created or generated images to the ECR repository.
- Finally, whenever these services are runing inside the cluster, task will be constandly pushing some logs to cloudwatch, so that if there is any issue, such logs will indicate in the cloudwatch Dashboard coming from Task within the cluster at the tail-end.
- The ECS cluster carrying those 2 services are found or are embedded within a vpc for the simple reason that the cluster is the core part of our architexture or project. It carries our Application it carries our task, container definition where our application is deployed and its residing in there.
- So, it is better and best for us to protect it inside our VPC, that is why we have our VPC configure all with it subnet, NATGW etc in our buildspec.yml file. For security and best practices purposes.
# --------> End of Explanation --------->
-
***That set: Open this Github link of final Repository here: **https://github.com/Kenneth-lekeanyi/ecs-demo** and 
- clone or download the Github project and put it in your local.
- 
- # Now let start:
# Step one:
# 1)	Let create our IAM user CodeCommit Git Credentials: create IAM credential to your IAM user which can be use to push the Application Code for AWS CodeCommit
- So go to AWS IAM console
-	Click on “**Users**”
-	Either you create a new user by clicking on “Add user” or you click on the IAM user that you are using all through.
- Make sure that your user has "administration Access"
-	If you are creating a new user, after clicking on “add user”
-	Name of the user:
-	Give him some priviledges and “next”
-	Then under permission click on "Aattach existing policies directly”.
- Then, select “Administrator Access”
-	After clicking on the user and ensuring that your user have administrator access;
-	then click on "security credentials”
-	Scroll down to locate “https Git credentials for CodeCommit”
-	then under this click on "generate credentials”
-	***then you make sure you download the credentials and save somewhere you can locate it***
- if your user already have the credentials that you created some time back, still click on “generate credentials” so that those credentials will pop up.
- You can then click on download credentials to download and save them somewhere.

# Note that, if someone wants to contribute to the pipeline or building or doing anything in the pipeline which you recognize, you have to share this credentials with the person for him or her to be able to contribute to the pipeline.
-	Then click on "close”
- So, we have successfully created our IAM User credentials. Now as we have successfully had our credentials we move to the next step.
- 
# Step two: Set up CodeCommit repo and push the code to CodeCommit repository: ***{so that, we can put our CodeBase in the CodeCommit Repository}***.
- So, go to the search engine and search for "CodeCommit"
  - click on "Codecommit"
  - click on "repositories"
  - click now on “create repository”
  - repository name: **ecs-cicd-demo-repo**
  - Descriptions: **repo to store the Application code for the ECS cicd demo.**
  - Click now on "create"
# -------> Successful ------->

-	After successfully creating this CodeCommit repository, the next thing you have to do is to clone this repository and take it to your local.
-	To clone this CodeCommit repo, while still in the repository where you have successfully created, do the following;
-	Locate Step 3: "clone the repository"
  - click on the "copy" button to copy this repo
  - then you take it to your terminal (for mac)
  - so open your terminal
  - paste the cloned repo that you just copied from CodeCommit and hit enter
  - It will prompt you to enter the CodeCommit credentials. So go to where you stored the credential and copy them. 
  - Pass the Username:
  - Pass the password:
  - It will show you that you just cloned an empty repository
- Now, go to your location **Finder**--->**Owner** you will see it there
- At this time, we are going to push the Application Based Code and the files that we the DevOps engineer have prepared or build into the empty just created CodeCommit repository.
- So, let's copy the files from our GitHub repo folder (**ecs-demo**) to the newly created CodeCommit repo.
- Note here that as a DevOps engineer, we will have to create our own code files and add to what the developer sent to us to form 1 repo.
- But for this demo we are using the already created files to save time. That is a reason why we are copying from our local GitHub folder to the new CodeCommit folder
- the ECS demo file that I clone from this repository: https://github.com/Kenneth-lekeanyi/ecs-demo
- It is seated inside the Downloads Directory. Open it and copy the files in there. Copy all the files except the file titled as .git (Some people will have this git while others will not have it)
- Now, open the newly created empty CodeCommit folder (**ecs-cicd-demo-repo**) That we clone to our local and paste it there. As you can see a file .git already exist in there, as this is a git repository.
- Then open the **ecs-cicd-demo-repo** in your VS code.
- Now open the "Dockerfile"
  -------> Explanation here ----->
- **Dockerfile**
# FROM nginx
FROM: public.ecr.aws/nginx/nginx:/latest
Copy: app/user/share/nginx/html/
***{The reason we use this method is that Docker recently come up with some limitations that once you have pulled a certain limit of Docker images, you cannot exceed that limit. Reason why we go to the public ECR of AWS and pull this latest version of nginx. So, under Docker, if you are not logged into DockerHub to become a premier user, you cannot pull images after a certain limit has reached}***

- When you open this Application code in your v.s code, save it first. ***(control+s)**
_ When you open the Application code in your V.S code locate “terminal” up at the taskbar and click on it to open it.
-	Then click on “New terminal”
- On the terminal that you just open on the V.S code, You will see your cursor blinking on the path of **ecs-cicd-demo repo**
- Now, it is time to push the code to the CodeCommit repository. And to do that, we have to execute the git commands;
1)	Execute the first command to check the status and to check this current status of the changes do
- `git status`. It will show you in red the files that you are to be added
2)	Add all these new files or changes to the git staging area or environment. Do this command
- `git add .`
3)	Now, add the commit message and connect it to your staging area through this command in order to completely save it.
- `git commit -m “adding codebase to CodeCommit repo"`
4)	Now, push the changes to the Central CodeCommit repository in CodeCommit using the command
- `git push --set-upstream origin master`
- Note, the push action will prompt you to provide the CodeCommit credentials which we have created earlier. If so then enter the CodeCommit credentials;
  - **Username:**
  - ***Password:**
- Once the above steps are completely successful verify it by accessing the CodeCommit console
-	Go to CodeCommit
-	click on repositories”
-	You will see the **ecs-cicd-demo-repo**” Code.
-	And when you click on it you will see all what has been added.
- So this is called a **Central repository** where all the developers and DevOps engineers can see all the files that has been added here.
- 
- **With this, we have successfully completed the Code setup part of our pipeline.**
- The next step will be to first of all create an ECR repository. As you remember, the ECR repository is where the code that is built in is CodeBuild, is happening. The images will be pushed into this ECR repository.
- 
# Step three: setup the CodeBuild project.
- Here, we are going to setup a CodeBuild project to build the Application, create docker image and store it in the ECR repository.
- But before that, we have to first create an **ECR repository**. 
- So access the "ECR console".
-	Then click on "repositories"
-	then click on "create repository"
- still on this create repository page keep all the default details as it is and fill
  - Repo name: 
  - general settings:
  - visibility settings:
   - private: Click to check the box on [private]
-	repository name: **ecs-demo-cicd-repo**
-	Click on "create repository" at the bottom of the page.
-	With this we have successfully created the ECR repo, which we will use during our cicd setup to push docker images to it for storage of these images.

# Step four: Setup IAM role for CodeBuild **{So, CodeBuild is the guy doing the interaction, that is why, it needs the IAM Role policy, so as to make it have due permissions or flexibility to interact with other Services.}***
- Here, we are going to create a policy that will be used by CodeBuild to interact with other AWS services as part of the CICD Pipeline.
- So, go to IAM console.
- And before we go to create an IAM role 1st we need to create a policy.
-	Therefore under IAM , click on “Policies”
-	Then click on “create policy”
-	Now, on the create policy page, you will see two petit sections: [**visual editor**] and [**Json.**]
-	Click to select “**Json**” on clear all the existing script in there
-	Now, replace our custom policy in this editor under Json with the policy you find in this link.: https://github.com/Kenneth-lekeanyi/ecs-demo/blob/master/help/iam-policy-ecs-demo-codebuild.json
- So, copy this policy here in this link and take it to the create policy page under Json.
- paste the policy there.
-	Click now on “next: Tags”
-	Then click on "Review"
-	Review policy
  - Name: **iam-policy-ecs-demo-codebuild**
  - Description: **IAM policy used for CodeBuild to interact with other AWS services as part of the ECS-demo**
  - Now, click on "create policy"
- Policy IAM has been created
-As we are done with the policy, the next step is to create a role, so that we can attach this policy to a rule

- # Now, create an IAM role and attaches policy for that role, so that ColdBuild can utilize the attach role and policy permission.
- So, go again to IAM console.
-	Click on "Roles"
-	click again on "create role”
-	select [trusted entity type]
  - select [AWS services]
  - use cases for other AWS services; In the box, type [**CodeBuild**]
  - Click to check the box on "CodeBuild"
  - Then, click on “next”
-	Add permission
  - Search on the search bar for the policy that we created with the name as **iam-policy-ecs-demo-codebuild**
-	click on “next”
-	Review
-	Role details
  - Role name: **iam-role-ecs-cicd-demo-codebuild**
  - Description: **Allows CodeBuild to call AWS services on your behalf**
-	click now on “create role”
- With the above steps we're done setting up the required role for the code build service.
- The next step will be to set up CodeBuild

# Step five: Setup CodeBuild project
- Now, we are going to set up the CodeBuild project to build, create Docker image and push the image to the ECR repo.
- So, log into or go to your CodeBuild console
-	click on "Build projects”
-	then you click on "create build project”
-	project configuration
  - project name: **ecs-cicd-demo-codebuild**
  - Descripcion: **Codebuild project to build the ECS demo Application and push image to the ECR repo.**
-	Source:
 - Source provider:
  - **AWS Codecommit**
  - **ECS-cicd-demo-repo**
-	Reference type:
  - Select [Branch]
-	Branch:
  - Use the drop down to select [**Master**]
-	Environment
  - Environment image
   - Select this section on [managed image]
-	Operatinf system:
  - Select [Amazon linux2]
-	Runtime:
  - Standard
-	Image:
  - Select [Aws/codebuild/amazonlinux2-x86_64-standard: 3.0]
-	Image version:
  - Select [Always use the latest image for this runtime version]
-	Environment type:
  - Select [Linux]
-	Priviledge:
  - Check the box on [Enable this flag if you want to build Docker image or want your builds to get elevated priviledges.]
-	Service role:
  - Click to select “[existing service role]”
-	Role ARN: Here search for the full name of the IAM CodeBuild Role that we created "**iam-role-ecs-cicd-demo-codebuild**". When you search and enter it, its ARN will appear in the box as shown below.  OR you can go to the ECR Console and copy the URL of this ecs-demo-cicd-repo directly and paste it in this box.
  - in this [arn:aws;iam::46599248654:role-ecs-demo-codebuild]
  - ***{As you can see here, by default CodeBuild has a default role which you met inside here. Just as thesame as CodePipeline and CodeCommit, they all have default Roles. But as compared to CodePipeline and codeCommit, their default Role is sufficient, thats why we did not create any additional role there. But concerning CodeBuild, its own default Role is not sufficient simply because it is doing so much work. That is the reason why we are removing it default Role and selecting or attaching our own created custom role to this CodeBuild}.***
- Check this box on [Allow AWS CodeBuild to modify this service role so it can be used with this build project].
- If you want to choose a VPC that you want your CodeBuild to access it, you can do that under additional configuration
- And in the same light if you want your compute power to have a certain **GB of memory** and a certain **VCPU**, You can also choose that under additional configuration.
- But this demo is not doing anything under additional configuration so go to Buildspec. But let's See environment variable under add C.
- Click on "Additional Configuration”
- then you Scroll down to locate Environment Variable
-	Environment Variables:
  - Name:**REPOSITORY_URI**     - value: **464599248654.dkr.ecr.us-east-1.amazon….**       - Type: **Plaintext** ***{you get this value from the ECR Repo URI. Go to the ECR Console, locate the ecs-demo-cicd-repo, you will see this value as it URI. copy it and paste it here.}***                                                           
- **Buildspec:**
-	Build specifications:
   - Select [**Use a buildspec file**]. This as opposed to "insert build" command, so that we can keep a version of the buildspec file. If we use "Insert build" command, you will not be able to keep a version of the command you use.
-	buildspec name:
  - buildspec.yml
-	CloudWatch
  - Check the box on [CloudWatch logs]
  - click on "create build project”
- With the above steps completed we can now see that the new CodeBuild project has been created.
- 
- Note that, you're file name to be used in bluildspec could be anything. But with a yml extension. In these our Application we are using buildspec.yml
- study this buildspec.yml file at this link
https://github.com/cvamsikrishna11/ecs-demo/blob/master/buildspec.yml
- After successfully creating the build or ColdBuild project, let's proceed at this time to start the build.
- 
-	So click on “**start build**”
- you will then see that the build has been initiated immediately
- Once the build has started and completed successfully, we can see the build status and success message
- then we can now go to the ECR repo and you will see the created Docker image there having the latest version.
-	You will also see a black bar saying ‘**showing the last 1000 lines of the build log. [View entire log]**
- if you click on this [view entire log], it would directly take you to the CloudWatch log by group. if you take open link in a new tab”
- (it is that role that facilitate the logs to be pushed to CloudWatch)
- From the image we are seeing in the ECR repo, we notice that we just only created the repository and everything is being pushed to this repository automatically.
- Note that, CodeBuild does not really charge that much. It only vharge us depending on the **number of innovations and the number of runtime.

- 
# Step six: Setup CodePipeline to orchestrate the stages
# Note: 
1)	This section **(CodePipeline)** has a dependency to create the VPC stack.
- We have already setup our VPC using IAC in this link: https://github.com/Kenneth-lekeanyi/ecs-demo/blob/master/infra/vpc-alb-ecs.yml
2)	This section **(CodePipeline)** also has a descendancy to create an ECS cluster, Service and Task stack. We have already also setup these dependencies using IAC in this link below: https://github.com/Kenneth-lekeanyi/ecs-demo/blob/master/infra/fargate-service-task.yml
- In this section, the goal is to create the CodePipeline to integrate all the other AWS services that are required for this **ECS-DEMO-CICD PIPELINE.**
- Let's start by going to the AWS CodePipeline console
1)	  Go to CloudFormation and ensure that you still have our stack and to create our VPC stack;
-	Go to the CloudFormation console and click on "stacks"
-	click on "create stack”
   - "with new resources (standard)"
-	prepare templates:
 - select [template is ready]
-	Template source:
 - Select [upload a template file]
-	Upload a template file:
 -   Click to select [choose file]
-	**ECS-demo** or **ecs-cicd-demo** repo, select any of these especially the **ecs-cicd-demo-repo** that we already created
  - Click on "Infra"
  - Then click on "VPC-alb-ecs"
  - click now on "open"
  - click on "Next”
  - stack name: **ecs-demo-vpc-ecs-alb**
  - click on "next”
  - click again on "next”
  - Review
  - click on “create stack”
- As you can see our stack has been created.
- 
2)	Now, let's proceed to create another stack for our Fargate Services
- So, still go to CloudFormation console
-	click on "stack"
-	Click now on "create stack"
  - {with new resources (standard)}
-	prepare template:
  - Select [Template is ready]
-	Template source:
  - Click to select [Upload a template file]
-	Upload a template file:
  - Click to select [choose file]
  - select "ecs-cicd-demo-repo"
  - Click on "infra"
  - Then click on "Fargate-service-task.yml" {we are using this template because it has the code to create Fargate and services}.
-	click on "open"
- click now on "Next"
- stock name: ecs-demo-fargate
- click on next”
- click again on "Next”
- Review
- click on "create stack”
- you can verify to see the Load Balancer by going to EC2 console.
- 
# Let's Now Start With Our Actual Work By Setup The CodePipeline:
- So, go to the CodePipeline console and click on "create pipeline”
-	name of the pipeline: **ecs-demo-cicd-CodePipeline**
- service role
  - Click to select "News Service role"
-	Advanced Settings: Click to check the box on [Allow AWS codepipeline to create role so it can...]
- Artifacts store:
  - Click to select [Default location]
-	Encryption key:
  - Click to select [Default AWS managed key]
-	click on "Next"
-	Add source stage
- source provider:
  - Select [AWS CodePipeline]
- Repository name:
  - **ecs-cicd-demo-repo**
- Branch:
  - [master]
- Change detection options:
  - Select [Amazon CloudWatch Events]
- Output artifact format
  - Select [CodePipeline default]
- click on "Next"
- Add build stage:
  - Click to select [Build provider]
   - Click on [**AWS CodeBuild**]  **{Now, we are trying to integrate the CodeBuild with this CodePipeline}**
-	Region:
   - US-east (N. Virginia)
-	Protection name:
   - **ecs-cicd-demo-codebuild**
-	Build type:
   - Select [single build]
-	click on "Next”
-	Add deploy stage:
   - Deploy provider:
     - Select [Amazon ECS]
-	Region:
  - US-east (N Virginia)
-	cluster name:
  - **ecs-demo-cluster**
- Service Name:
   - **ecs-demo-service**
-	image definition file:
   - **imagedefinitions.json** ***{Basically, we are calling this imagedefinitions.json file that was created in the CodeBuild and put some files inside as seen in the Buildspec file. We are now bringing the file in the deployment stage. This file carries image details. So, at thew CodeBuild, while the image is pushed to the ECR, other files or Artifacts is carried and moved forward to CodeDeploy.}***
-	click on "Next”
-	Review
-	click then on "create pipeline”
- with the above action, we can see that CodePipeline will be created and will start the first build automatically.
- 
a)	And we can access the current build details by creating on the "details” icon in the build stage directly below “succeeded”
b)	Go to "Deploy", and under "in progress”, you will see details”. Click on such "details"
- Once you click on this "Detail” you will be directed to the concerned **ECS service** and **Deployment section**, where we can verify the deployement process.
- So, click on **ecs-demo-service**
-	then click on the **Deployments** between ASC and Metrics.
-	If you now reload the Load balancer so many times, it will be showing the Application UI. This is because the Task are up and running fully.
-	
c)	If you click on **Task**, you should be able to see two extra tasks that has been added to the existing 2 Tasks to make the new deployment without any issues.
d)	After a few minutes, if everything is smooth and Tasks are up and running, ECS Service will delete the old Task that are running and keep the newly deployed Task as it is. We can see it by refreshing the same Task screen after a few minutes.
e)	Now, also verify if the newly created Task definition revision has the Docker Container image that was created as part of the CI/CD pipeline and push it to the ECR, so then let's click on the “**Task definition**”.
f)	On the Task definition page, go to the "New Revision” and go to the container section, and under the container image section, we should be able to see the container image URI
- So, go to the ECR console, then click on **ecs-demo-cicd-repo** and compare the latest image to see if the container tags are matching.
We can see that both the container definition and the ECR image have the same tags, by which we can understand the CI/CD pipeline is automatically updating the ECS Container definition to place the latest docker image.
g)	And lastly, if we access the **ecs-demo-cluster** and go to **ecs-demo-service** and then go to “Events”, we can see all the events that are happening in that cluster related to deployements: scale-in, scale-out etc
- 
# Now,Lets Inject Some Changes Into The Application Code And See What Happens At The Level Of The Application UI After Pasting Our LB URL On The Browser.
- So, we are going to inject some changes and then we push it to the pipeline.
- To introduce this change,
- Open the VS code (which we are using to interact with the CodeCommite). Then on your VS code, open the application base code (**ECS-CICD-DEMO-REPO**) Where you have all your files in there.
-	Now open the “app” folder
- inside this app folder, we have the **index.html**. (This indexhtml is whatever that you are seeing in the user interface UI of this Application when you paste the LB on the browser).
- Inside this index.html, inject the following
- <h3> App version-1.2.0<h3>
- <h3>deployment from the codepipeline demo and we are gonna discuss rollout deployments</h3>
- Now, bring up your VS code terminal
-	Check the status by those changes we have just made, by doing
- `git status`
-	Now, add the changes to your staging area by doing
- `git add .`**OR** `git add app/index.html`
-	Now, committe the changes to your local staging area through
- `git commit -m “diploying version 1.2.0”`
- If you now do `git status`, you will see that one file has changed and it has gotten to the master branch.
-	Now push our staging area to the central repository that is "CodCommit"
- `git posh`
- if it prompt that you enter the CodeCommit credentials, then just go to the CodeCommit credentials that you downloaded and grab them to pass them here in
-	**Username:**
-	**Password:**
- Now, go to the CodePipeline console. You will see that the CodePipeline has been triggered (to deploying version 1.2.0)
- If you go to CodeBuild console and click on "build project", you will see the build status saying “**in progress**”
- if you go to CodeCommit console and you click on commit, you will see the changes we just made.
- Now, go to CodeBuild console to see the "build log"
- Now, go to the CodePipeline console
  - click on "Pipeline"
  - Scroll down to the "deploy"
  - Then click on "details". This will direct you to the details part of the deployment
  - Then, under "Deployement", you will see that 
  - minimum healthy percent is **100**
  - maximum healthy percent is **200**
- Desired counts: **2** under primary are running while
- **2** under **Active** are running
- The old task have older revision while the new task have current revision
- 
# what is happening in the Roll-out is that when the minimum percent is 100 ,
- maximum healthy percent is **200**
- then they desired count has **2**
- while Active running is **2**
- 
- very soon the older Task will get killed and ruled out, while the current number of Task continue to run When the deployment has reached its steady state.
- That is the reason why you see "Desired counts as **2**, while running-count stands at **4**. So that in few minutes it would dismantle the old tasks and maintain the current tasks. 
- **So that when there is any change, it will not encounter any downtime. Simply because as the old task gets rolled-out, the current tasks are already running perfectly.**
- ***{So, when the changes are deployed on the new Task, it will stay a while and ensures that the Deployment is steady succesfully, then it will terminate the old Tasks, while the new Tasks continue to run. Because during the Deployment, the desired count was 2, But after that, the running Task is 4, so as to rollout the older Tasks when the Deployment get steady and maintain the current 2 Tasks}***
-	if our service or Desire-counts is = 2, wherein our minimum healthy percentage is 100%, then how many task should we always have? = **2 task**
- which means that, at any given point in time, we should and will always have at least **2** Tasks running succesfully.
- Now, it also says **maximum percentage of healthy tax is 200%.**
- Then it means how many tasks should we always have? = **4 Tasks**
- which now means that, at any given point in time, It will always have a maximum of **4 Tasks running**.
- 
- Number of Service's desired-counts is **2**
- So, using the minimum percentage at 100% will always keep the task running at **2 Task** and
- Using the maximum percentage of 200% will always keep the task running at **4 Tasks**
- service ---->desired count = 4
- Minimum health percentage = 50%
- Maximum health percentage 100%
- This is how Tasks are terminated, so that at any given point in time, we must have a desired number of Tasks up and running.
- 
- Following this therefore, anytime that there is a deployement, A new image is being created and while that new image is being pushed to be stored in the ECR repo, a version of that new images is being deployed into the ECS cluster. While a version of that new version will be stored in the ECR repository.
- 
# Question: What if the Tasks that keep created or generated by Service is constantly  encountering some bugs?
# What if the Tasks that are being generated by Service are constantly reporting having bugs that once Tasks are created, the bug continue to obstruct the perfect functionality of the deployment?
***Answer (Hot Fix);***
- In this situation you will have 2 options:
1)	Either you roll-back to your changes in your Git staging area or you get to the CodeCommit Repository and grab the version of this Application Code base and do a new pipeline or
2)	You get to the ECR Repo, take the image URI (maybe the latest version)or (the older version), you take it and run the pipeline. So that this version will do the deployement in the cluster as a new version. That is the reason why we keep the version of the images stored in the ECR, so as to grab it whenever you want and deploy it.
- # How do you do this?
You simply go to CodePipeline console and
-	click on "Pipeline"
-	click on "create pipeline”
-	pipeline name: input the name
-	click on next”
-	source
  - Here, it will ask you the source of the image.
  i) if you want to introduce the older version of the image, then select Amazon ECR
  ii) and if you want to introduce the Application CodeBase as your roll-back to eliminate the bug from CodeCommit Repository, then select AWS CodeCommit.
  iii) If you want to only use the older version of Tasks to do the deployement, first copy the Tasks of that particular Task version from Task (which looks something like this **2022-04-13.00.43.26.16339c43**
  - So that when you get to the "source” page here, you just paste the Task tag there and proceed to deploy it. ***{This is quick, when customers are really on your neck}***
- When the deployement reach a steady state, the number stated in the minimum will get terminated while the maximum number in maximum health takes effect.
- This is how the Roll-out (Rolling Update) deployment strategy works. And it is very advantageous because it's generate new Task whenever there is a new deployment without any downtime.
- That is the reason why when it is running, you will see the RUNNING COUNT under (**ecs-cluster-Tasks**) at 4, while the DESIRED COUNT stands at 2.
  - Desired count: 2
  - Pending count: 0
  - Running count: 4
- This is because under the Deployement we specified "**Rolling Update**"
- This is because by default, AWS ECS deployment runs on a Rollout which is Rolling update.
- So Rolling update deployment is the default deployment in the ECS Service.
- With this, we have completed the CI/CD demo flow with Rollout update type of deployment, and we understand what is rollout type, how to use AWS CodePipeline native integration with ECS deployments.
- -------------> Explanation Here ------->
- So, Rollout in simple terms is a kind of Deployment wherein, you dont deploy on all the existing Instances or on all the existing Task sets. But you will select a set of 1 or 2Tasks or instances at a given time and deploy on them one after the other. So that  whenever the first ones get at a steady stage, the previous ones will get killed, while the newly created tasks or instances continous to run.
- Such that, if under Service, we put
   - Desired Task at = 2,
   - Minimum Health put at 100% ***{minimum % of Task will be maintained at 2. Meaning that, at any given point in time, we should and MUST have a minimum number of 2 Tasks or instances running.}***
   - Maximum health put at 200% ***{This means that at any given point in time of traffic increased, we should and MUST have a maximum number of healthy task or instances of 4 running. But, while the older 2 Tasks forms the older version, the newer Tasks created will form the newer version, wherein, when the newer versions get stabilized at its steady stage, the older version will get killed, while the newer versions remain running. All happening without any downtime noticed.}***
   - Main person behind the success of all this is thanks to the image that is stored in the ECR, that, it is using to deploy from one Task to another. So, every Task when created is grabing the new version of the Code from the ECR and its running them.
   - 
 
   - ***If it is a Web Application that we are dealing with, to attach it to a Load Balancer, we would have to open Port 80. Because Web Applications are HTTP***
   - ***If it is a Custom Application like Backend Applications such as Nodejs, Java, Python, we shall open the port based on the requirments.txt which may be 5000, 8080 etc. But it will be mentioned in the requirements.txt file*** 
- We shall now go to see how to set up the Blue/green deployment.
- ----------------> Explanation Ends Here -------->

# BLUE/GREEN DEPLOYMENT STRATEGY
- Note that, you have to select your Deployment Strategy at the level of "Service creation".
- SO, when creating Service, (you MUST have discussed and concluded on which Deployment strategy that you will use. Either Rollout or Blue/Green.
- So, you have to integrate the Deployment Strategy at the level of "Service".
- However, both of them have dependencies which all relies on "Service".
- So, you can only set up Roolout if the Rollout strategy was selected at the level of Service Creation ALSO,
- You can only set up Blue/Green if the Blue/Green strategy was selected at the level of Service Creation.
- 
- **THE BLUE/GREEN DEPLOYMENT**

- 
