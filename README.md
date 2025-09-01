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
•	EnvironmentName: **ecs-demo**
•	ImageURI: **Vamsichunduru/ecs-demo:v1**
•	Path: *  ***{bECAUSE WE ARE PUTTING A load Balancer, that LB will be able to have a particular path. by default, we are saying that, whatever traffic you have (*) rout it to the Load Balancer that we are creating}***
•	Prority: **1** **{Here, we are saying that, execute this particular function "*" that we have in our path as the first priority, and whatever traffic that it has from the LB, it will route the traffic to Task that we have put in "Desired count" above.}***
•	Service name: **ecs-demo-service**  (Since we are creating a service, This is just the Service Name}
•	Click on “next”
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
-	Paste it in the browser to see the deployed Application with the LB DNS.
-	You can also access the logs to see the logs that have been generated
-	Note that: These logs are coming from the Tasks. So that whatever is happening inside the Task is being pushed to CloudWatch. So, these logs are reporting whatever is coming from the ALB and getting into task.

-
Q $ A
If a company wants to build an application like this one that we have been dealing with what will be required from us the DevOps engineers?
Go through this with this diagram illustration
first idea
1)	First thing to do is to manually prepare it as we did so as to understand (So first thing to do is a manual hands on as we did)
Task definition
Service
Cluster
So that if you are creating a container you will know that you need an image or container image.
2)	Then think about how you will automate it. Whether you will use cloud formation or terraform or Boto3 or CDK or whatever tool you are comforted with.
First job
1a) Then using all these things we shall start to develop the infrastructure first.
On what are the infrastructure we need?
1)	We need a VPC
2)	we need some IAM roles
3)	we need an ECS cluster stack
all these if you are deploying for ECS CPU.
If you want to use but EKS, it is pretty much the same
-	You need a VPC
-	you need some IAM roles
-	you need an EKS cluster
if you want to use but EC2 to do the deployement
-	you need a VPC
-	you need some IAM roles
-	you need an EC2
if you want to deploy using lambda, then
-	You need a VPC
-	you need some IAM roles
-	you need Lambda.
•	So first creat your VPC so that the same VPC can be used by all methods like ECS, EKS,EC2 etc.
•	Then create your IAM roles too, so that the same IAM can be use by both methods as well if possible.
1)	So with a VPC; first group the cidr ranage, subnet cidr etc. So as to go and prepare your cloud formation script
2)	then do the same thing for IAM role
3)	then go and develop or create an empty ECS cluster
you then prepare all these 1,2,3, initial phase and keep
So that whenever the developers gives you the application code then we just proceed to put up layer (4)
4)	We will then come up with another cloud formation to create
-service
-task definition
(with the application code taking the requirements from the application developer).
This is because unless we have the infrastructure even if the developer has the application, we cannot deploy everything. We need the infrastructure to deploy this application code that has been build.
So as he is building his application code, you too the DevOps engineer you are preparing your infrastructure as well. Search that immediately he gives the application code to you you just start with stage 4 to prepare the CF on
-	Service
-	Task
From the app you can build another CICD as well. And knowing that the base infrastructure setup is now Good and guaranteed enough, we can deploy our CICD application gotten from the application code on top of this base infrastructure.
What most of the companies now do is that they use mastered method therein




So then with this task you are not doing anything manually.
As a DevOps engineer you are pushing the infrastructure code to the repository and immediately the infrastructure is getting triggered and it is creating your infrastructure.
So everything is automated, nothing is manual here.
So, so far we launched our
-	VPC
-	ECS
-	IAM
fargate services
task
ALB (target group)
Path mapping
Let's delete the stuff to prevent some changes
1)	go to load balancer
-	under the load balancer click on “listener”
-	then click on the - at the top
-	then click on “delete”
this rule was added by him manually that is why we are deleting it single.
2)	Now go back to cloud formation
-	click on stack
-	then under the stock name click on the “ecs-demo-vpc-ecs-ALB” This is because it has a NAT gateway attached to it that will change it as well.
3)	Click on the ecs-demo-vpc-ecs-alb under cloud formation then click on “delete”
Auto-scaling
auto scaling with help to scale out/scale in the number of tasks within a service to help the heavy incoming traffic flow and to save the cost during less user access by scaling in.
Steps to enable autoscaling
1)	update the service to enable the auto scaling
2)	configure the scaling policy with metrics
a)	target scaling policy
b)	step scaling policy
so if you did not delete the stuff that's fine but if you deleted them, then let's create them back.
To create the VPC stack
•	go to create the stack, go to your cloud formation console again
-	click on “create stack”
with new resources (standard)
•	prepare template
-	template is ready
•	specify template
-	upload a template file
•	upload a template file
choose file click here
•	ecs-demo
-	click on infra
-	choose “VPC-alb-ecs”
-	then click on “open”
•	Click now on “next”
•	stack name: ecs-demo-vpc-ecs-alb
•	next
•	configure stack options
click on “next”
•	review
click on “create stack”
Let's now proceed to create the fargate stack
so still go to the cloud formation console
-	click on “create stack”
with new resources (standard)
-	prepare template
Template is ready
-	template source
upload a template file
-	Upload a template file
(choose file) click here
Ecs-demo
Click on infra
then click on fargate service tax
then click on “open”
-	then click on “next”
-	stack name: ecs-demo-fargate
-	next
-	review
Click on “create stack”
now let's focus on auto scaling
we have seen that

ECS service---help us to maintain---[desired # of task=2]
                                                                       ---ALB
                                                                      ---ASG
The role of ASG service will help to always maintain the desired number of tax that are up and running. Then what if the traffic suddenly rises to set 2 million trying to access out only 2 desired number of tasks?
So ASG will come in and edit the desired number of tasks to either scale up to 5 or 6 depending on the amount of traffic coming in. And it will also scale down if traffic gradually reduces to say 1 task by editing the desired number of tasks prescribed to respond to the decrease number of traffic.
Auto scaling group of task
For us to create an autoscaling group for our tasks, we go to service because one of the copabilities of services to create the ALB, ASG And maintain the desired number of
Service-desired number of task
-	ALB; security
-	ASG; traffic
So let's go to our service and to access service go to ECS console
-	Click on clusters
-	Click on our cluster” ecs-demo-cluster” that we have created
-	this will then open the cluster when you will be able to see services
so click on our “ecs-demo-service”
once we have on the service page, now click on auto sealing section to see or check if any auto scaling is already enabled on the service.
As there is no autoscaling enable as of now, click on update to edit the service and add the auto scaling.
-	So click now on auto scaling
-	then you click on” update” (so that we can update the service) this is when we shall configure the auto scaling tax.
-	Configure service; here you will see that we have already configured this before. So proceed to the next step
-	click on “next step”
-	configure network
click on “next step”
-	set auto scaling (optional)
this is the place where we are going to add auto scaling settings.
1)	Target taking scaling: 
2)	schedule scheduling
3)	step scaling policy
so,
-	set auto scaling(optional)
now on this page
-	click on select “configure service autoscaling to adjust your services desire courd” and then feel the details here.
. Minimum number of task: 1
. Design number of task: 2
. Maximum number of taxsks: 4
. IAM role for service auto scaling: ecs-demo-asg-role
-	Automatic task scaling policies
-	click on “add scaling policy”
a page will pop up for you to add the policy 
-	scaling policy type
-	select “ step scaling”
-	policy name: ecs-demo-scale-out-policy,
-	Execute policy when: create new alarm
•	Alarm name: scale-out-alarm
•	ECS service metric: CPU utilization
•	Alarm threshold: Average of CPU utilization >- 2 for 1 consecutive period of 5 minutes
-	Then click on “ save”
-	Scaling action: Add 1 task when 2
Interpretations: i.e whenever CPU utilization is greater than 2, add 1 task.
-	 cool down period. 300 Seconds between sealing actions. So we are saying that for every time that you add a task for CPU utilization get greater than 2, take a break to colddown.
-	 Click now on “ save”
Then simply proceed to the next step
•	Click again on “ add scaling policy”
Scaling policy type
-	Step scaling
-	Policy name: ecs-demo-scale-in-policy
•	Execute policy when: create new alarm
•	Alarm name: scale-in-alarm
ECS service metric ( taking alarm of this repair): CPU utilization
•	Alarm threshold ( average) of CPU utilization < 1 for 1 consecutive period of 5 mounts
Interprelation so whenever CPU utilization is les than 1, go ahead and rename 1 task. So when ever this is done that will trigger the alarm that will afread and reuse the task.
•	Then click on “save”
•	Add policy
-	Scaling action (remove 1 task when 1)
-	Cooldown period (300)
•	Click now on “save”
•	Click now on “next step”
•	Review
Click on “update service”
•	Click here on “view service”
Click on auto scaling between event and deployment to see your AS our auto scaling has been created.
*Now, we can check the individual alarm details by clicking on the scale-out-alarm and scale-in-alarms individually.
If you click as the “ecs-demo-cluster” and click on auto-scaling, you will see that the desired count has been turned to “1” instead of “2”. This is because auto-scaling has gone into effect immediately and it has seen that they is no optimum utiliation of the CPU, so it has triggered and an alarm and 1 task has been removed.
This shows that auto-scaling group is working perfectly.
•	Now to see how the alarm is working, still under “auto-scaling” click on any of the alarms ( either the “scale-out-alarm” or on the “ scale-on-alarm”). So as to see the state of the alarm in the cloudwatch.  OR
•	You can go to the Cloud watch console, then you click on “ All alarms” you will see all our 2 alarms there. You will see that the in-alarms is showing in Bed, meaning that this in-alarm has been triggered. Whatever condition that we put there has been reached, so the alarm has been triggered.
•	Click on “ history” to read what is going on there”.
•	If you select or click on event” you will see what has happened.
Read the message there in the time frame that it happened
now as both the load balancer an auto scaling and enabled, let's simulate some traffic on the load balancer endpoint so that it should automatically increase the CPU utilization of the ECS service.
Here is the doc used to simulate the load
Hppts://www.humansreadcode.com/ab-performance-test-docker/
So executor the below commands in order to download Apache-ab and to run the load test using it. For this we are using docker so it does not matter whether you devop machine is Marc or window, as long as you have docker installed and configured.
Use the below GitHub link to access the GitHub
where we can group those commands to use
github.com/cvamsikrishna11/ecs-demo/blob/master/help/apache-bench-load-testing.txt.
so to start
1)	Let's download the http-ab image and to run the container through this command.
Docker run-dit-name httpd-ab-v/var/www/html:/user/local/apache2/htdocs/httpd
Run this command in your terminal since docker is already installed in your computer.
-	But before that common ensure your ducker is up and running.
Do: docker ps-a to see that the container is up and running
2)	Going to the next command let's access the bash in the running container through the command.
Docker exec-it httpd-ab bash
3)	Let's now generate the load to the load balancer DNS. But you have to replace the LB DNS you find in the command with the DNS of your own ALB.
So first copy the entire command and take it to a notepad or on a text editor
Ab-r-c 500-n 5000000 http://ecs-d-publi-h9d4utps3s74-2106267235.us-east-1.elb.amazonaws.com
Go now and group the DNS of your ALB
-	So go to your EC2 console
•	click on load balancers
•	select the newly created ALB
•	Then copy it DNS by clicking on the little copy
then go to where you pasted the command and place your LB and DNS within the red line.
Then copy the whole command and run it in your terminal.
Ideally instead of putting the DNS of the load balancer you should put your domain name such as facebook.com or netflix.com
now in a few minutes, we shall see that this scale out alarm would trigger immediately and will reach an alarm state as shown in the cloud match alarm section
-	go now to auto scaling and go to the events section in our cluster
-	then go to task and see what is going on there. Tast is getting created way task
-	Then go to service and click on metrics to see how matrix is going up
-	We can see the task number got changed from 1 to 2 Due to auto scaling out event and vice versa on scale up or scale down events.
Now if you have many instances it will be pretty difficult to be taking each instance and then monitoring it alarm. That is why we want to create a dashboard and add the created alarms to the dashboard so that we can easily verify them on the dashboard level for monitoring.
In order to do that, access cloudwatch console
-	Then you go to all alarms
1)	- Then select any alarm you want (scale-in-alarm)
-	Then click on actions on the top right
-	now select click on “add to dashboard” that populate after you clicked on actions
Since we don't have a dashboard yet, on the page the pops up after you had clicked on add to dashboard” on the left you will see “create new dashboard”
-	name of the dashboard ecs-demo-asg-alarms-dashboard
-	then you click on create
-	widget type (line….)
-	customized widget title
(scale-in-alarm)
-	Click now on add to dashboard”
2)	now let's go to scale out alarm and click on it to
-	click on action after selecting the scale out alarm
-	select a dashboard: ecs-demo-asg-alarms-dashboard
-	widget type: line
-	customized widget title: scale-out-alarm
-	click now on “ add to dashboard”
Now go to the top left and under dashboard” and click on ”ecs-demo-asg-alarms-dashboard”
As we are done with the auto scaling demo, we can stop the Apache bench process by pressing control C or command C when your terminal is up.
Now do exit 2 come out of that particular container. But if you do docker ps-a you will see that the container is still running with the name httpd-ab
-	to effectively kill the container do docker kill httpd-ab
-	now verify if the containers are still running by doing this command “ docker ps-a”
Now as we are done deleting the container, let's go to the cloud formation console and delete both the
Ecs-demo-fargate cluster stack and the
Ecs-demo-vpc-ecs-alb stack to save cost.
1)	Delete the ECS- demo-fargate first
-	Cloudformation console
-	Stacks
-	Select ecs-demo-fargate
-	Delete
2)	Delete the ecs-demo-vpc-ecs-alb
-	Cloudformation console
-	Stacks
-	Select ecs-demo-vpc-ecs-alb
-	Click on ‘delete”
-	Confirm delete.
So we have successfully completed Section 2 which has to do with load balancing and auto scaling with ECS.

Set up CICID pipeline  for AWS ECS Deployment for both rollout and blue/green 

