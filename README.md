CI/CD Pipeline
using AWS and GitHub Actions
Overview
The following report is the documentation on building a CI/CD pipeline that can help automate the workflow of developers and speed up deployment timeframes. The project is an implementation of current practices for teams working on the same project, but with the developers set in different locations around the city/country/globe. It seeks to simplify and speed-up the delivery of approved changes in the project and deploy them to the desired server in the company VPC.
While free options to build the infrastructure and a CI/CD pipeline are available, I chose this solution because it takes advantage of two major players in the web and cloud application development (Amazon and Microsoft). It also makes great use of the automating capabilities of AWS CloudFormation in building the initial infrastructure and GitHub Actions’ ease of connecting resources across different cloud providers.
Basic familiarity with AWS is recommended in building this project. Both around billing details and the options provided by AWS in its service offering. A basic level of knowledge is recommended for the use of Git and GitHub.
Overall, the project is easy to follow and implement, and it is an adapted version of an AWS tutorial in creating this type of pipeline. I did start out looking at different technology stacks and different approaches to build this project. I tried a Jenkins and AWS pipeline, I built an AWS CodePipeline CI/CD, but in the end, I chose to present the current pipeline as a final version of the project, because it felt to be the most elegant execution. CloudFormation’s automatic build of the resources required for the infrastructure helped me deliver the project faster as a one-person approach.
Purpose of the document
The project purpose is to build a fully functional CI/CD pipeline for a full-stack development team. It is broken up into two main goals. Building the infrastructure for the application within AWS and building a continuous integration pipeline using GitHub Actions.
For the first part, the project makes use of AWS CloudFormation for the initial build of the infrastructure. The application will be deployed on one of the instances built by CloudFormation within a private subnet in the newly built VPC.
The second part will connect GitHub to the infrastructure using GitHub Actions and AWS CodeDeploy. This way, any code changes made on a local machine (that are then pushed onto a specific repository within GitHub) will trigger an automatic build within GitHub Actions. The application will be deployed onto the desired instance within the VPC previously built by the Cloud Formation.
Scope of the document
This document serves as a guide to those interested in creating a pipeline using Git Actions and AWS. It also shows the pipeline working through the presentation given. 
The project does not show how to build an application that is deployed on the pipeline, nor does it show the benefits of having an autoscaling group and a load balancer, though these are built by the CloudFormation template as default.
The deliverables of this project are:
•	The application infrastructure - an AWS VPC with all the required resources for a secure deployment.
•	A working CI/CD pipeline - that can be used to deploy code from a local machine to a server in AWS.
•	A database layer with a database server to connect to
The tasks are as follows:
1.	Create a GitHub repository
2.	Clone the sample repo 
3.	Deploy the infrastructure and sample application using CloudFormation 
4.	Configure and connect GitHub
5.	Integrate AWS CodeDeploy with GitHub
6.	Trigger the GitHub Action to build and deploy the code. 
7.	Verify the GitHub Action trigger and the working pipeline.
8.	Automate the deployment to deploy the code on every push to the main branch.
Note: The project is not Free Tier available, it incurs costs due to usage of a NAT Gateway and Load Balancers. Estimated NAT Gateway cost per month is approximately €35. The Load Balancer costs will vary depending on traffic.
Environment Information
	An internet connection is required for the code to be pushed between the on-premises machine and GitHub repository, and to view the deployment and information on AWS.
	The local environment:
This depends on individual developer setup regarding IDE, but Git needs to be installed on the local machine and the AWS CLI can also be installed and used to interact with AWS. 
	Personal setup:
•	A Windows machine
•	PowerShell to work with the Git commands
•	Notepad++ to adapt the template from AWS CloudFormation and to keep documentation and notes
•	MySQL Workbench to connect to the database
•	Microsoft Word to write the final project report
•	Draw.io to build the diagram
	The online environment:
A GitHub repository needs to be used to receive the code from the developer/s and to deploy it further using Git Actions and AWS CodeDeploy.
A registered AWS account to make use of AWS services.
Architecture

Project diagram:
 
The project is built as a 3-tier architecture in an AWS VPC containing 3 subnets, replicated across 2 Availability Zones.
The user can access the application through the internet facing ALB. The business layer resides in 2 private subnets and the data layer can be found in the next 2 private subnets. In this project the sample application is not built to work with the database or connect to it, so the connection is made using a separate bastion host that is deployed in the public subnet of AZ1 (us-east-1a).
Project Resources
Resources built by AWS CloudFormation:
	VPC with a CIDR of 10.192.0.0/16.
	2 public subnets in 2 different Availability Zones. us-east-1a and us-east-1b.
	2 private subnets for the development/production servers
	NAT Gateway – only one is created. In a real-life scenario, for high availability, there should be 1 NAT Gateway within each Availability Zone.
	Elastic IP
	Public Route table
o	Associations: Public Subnet 1 and Public Subnet 2
	Private Route Tables – one for each private subnet (2 in total)
o	Each private route table is connected to the NAT Gateway.
	Application IAM role is created, together with the role policy.
	OIDC (Open ID Connect) Provider and GitHub IAM Role – allow the connection with GitHub
	CodeDeploy Application and Deployment Group
	ALB Security Group
	Application Security Group
	S3 bucket to hold a copy of the application. The bucket has versioning enabled.
	The CodeDeploy Role
	The Application instance type
	The Application Launch configuration 
	Autoscaling Group
	The application load balancer and listener (on port 8080)
For security reasons, the application is deployed in instances in the private instances of the VPC. That way, the servers are not directly accessible from the internet. The internet facing ALB helps the user reach the application, on the default HTTP port 8080.

Resources built post CloudFormation build
	A database layer with:
o	2 private subnets
o	an RDS MySQL server
o	a bastion host in the public subnet of us-east-1a, that helps connect to the database

Prerequisites for Pipeline Integration
to build the pipeline:
1.	Create an AWS account.
2.	Create a GitHub repository.
3.	Clone the project at https://github.com/aws-samples/aws-codedeploy-github-actions-deployment to a local directory
4.	Push the sample code to the empty repository created in the first task.
Execute the following commands to make sure that the local repository points to the GitHub repository:
	git remote remove origin
	git remote add origin <your repository URL>
	git branch -M main
	git push -u origin main
Once the above steps are complete, the infrastructure and pipeline can be built.
Infrastructure and Pipeline build
Steps:
1.	Use AWS CloudFormation to deploy the template and create all the resources. 
2.	Change the template parameters to suit your needs.
3.	Setup GitHub workflow and secrets.
4.	Integrate CodeDeploy with GitHub.
5.	Trigger the GitHub Action to build and deploy the code. 
6.	Verify the GitHub Action trigger.
7.	Automate the deployment to deploy the code on every push to the main branch.

1.	To deploy the CloudFormation template, complete the following steps:
	Open AWS CloudFormation console
	Select Create or select Create Stack, which ever option is available
	Select Template is Ready
	Select Upload a template file
	Select Choose File. Navigate to “template.yml” file in your cloned repository or local directory.
	Select the “template.yml” file and select next.
	In Specify Stack Details, add or modify the values as needed.

2.	Change the template parameters
	Stack name = CodeDeployStack
	VPC and Subnets = these are pre-populated but can be changed
	GitHubThumbprintList = 6938fd4d98bab03faadb97b34396831e3780aea1
	GitHubRepoName – Name of your GitHub personal repository which you created.

3.	Setup GitHub workflow and secrets
	From the project root directory navigate to /.github/workflows/deploy.yml and update:
o	AWS_REGION: us-east-1a
o	S3BUCKET: <bucket ARN from CloudFormation Output tab>
	Update aws/scripts/after-install.sh. (This script would copy the deployment artifact from the Amazon S3 bucket to the tomcat webapps folder.)
	In the GitHub repository, select Settings > New Repository Secret > Secrets > Actions
o	secret name = ‘IAMROLE_GITHUB’.
o	value = ARN of GitHubIAMRole, found in the CloudFormation Output tab.

4.	Integrate CodeDeploy with GitHub
The build performed by CloudFormation deploys the application and deployment group. They need to be connected to GitHub.
From CodeDeploy:
	Deploy > Applications > Choose: CodeDeployAppNameWithASG
	Deployments > Create deployment 
o	Deployment group: CodeDeployGroupName
o	 My application is stored in GitHub
o	Sign out of GitHub in another browser tab
o	Name the ‘GitHub token name’
o	Click ‘Connect to GitHub’ and sign into GitHub. Confirm
o	Select repository name and commit ID
o	Create. It will fail. It’s ok

5.	Trigger the GitHub Action
	From the GitHub repo, go to Actions
	Select Build and Deploy
	Select Run workflow
This is the manual way to trigger the pipeline.
6.	Verify the GitHub Action trigger 
The process can be seen in GitHub Actions and the deployment can be verified in CodeDeploy, by selecting the application and going to the Deployment Group tab.
To see the application running you can get the link from the Output tab in CloudFormation, or you can navigate to EC2 Load Balancers, copy the load balancer DNS, paste it in a new tab in the browser and add ‘:8080/SpringBootHelloWorldExampleApplication’.

7.	Automate the deployment
To automate the deployment on every push to the main branch, navigate to github/workflows/deploy.yml and comment out the ‘workflow_dispatch: {}’ trigger. Then add a push trigger:
push:
    branches: [ main ]

Database connection
Database Creation
•	Create a bastion host:
o	Launch an EC2 instance in a public subnet
Important parameters:
	Create a security group for the EC2 to allow SSH connection from your local machine.
	Create a key pair with a .pem extension to connect to the EC2-instance
•	Create a database subnet group to choose as a destination for the database server
•	Create a database using AWS RDS
o	Creation method: Standard create
o	Engine: MySQL
o	Template: Free tier -> disables multi-AZ availability
o	Settings:
	DB instance identifier: database-demo (can be changed)
	Expand credentials settings:
	Master username: admin
	Master password and confirm password: ******** (min. 8-character password required)
o	Instance configuration:
	DB instance class: db.t3.micro
	Storage: leave as default but deselect autoscaling
o	Connectivity:
	VPC: select required VPC
	Public access: No
	VPC security group:
•	Create a new security group (or select existing)
o	Add name
o	Add availability zone
! Note on Security Group inbound rule requirement:
	Add inbound rule to give TCP access from the bastion host IP on port 3306
	Database authentication: Password authentication
	Database authentication: Password authentication
	Additional configuration:
•	Initial database name: Add name (this will be the database schema name)
•	Backup:	Disable automated backups
•	Maintenance:
o	Disable auto minor version upgrade
		Create database
Once available, take note of the following database parameters:
	username: admin
	password: ********
	endpoint: database-demo.cymd2li6cffa.us-east-1.rds.amazonaws.com
	port: 3306
initial name (schema): mysqlDbInitialName

Database Access Procedure
Download and install MySQL Workbench. Once installed, from the top menu:
Database  >  Manage Connections  >  Connection Methods  >  Standard TCP/IP over SSH
Connection name: Enter a name
Connection parameters:
	SSH Hostname: Public DNS of the bastion host
	SSH username: ec2-user
	SSH key file: location of key file from local machine
	MySQL Hostname: Database endpoint
	MySQL server port: 3306
	Username: admin
	Password: Store in Vault (add the DB password)
Test connection. Close.
You can then connect to the database either from the home page or by pressing Database in the top menu and choosing the connection previously created. Inside the connection you should see the schema name (initial name) and you can start working on the database.

Conclusion
	This project is a good intro for beginners looking to learn more about DevOps and CI/CD pipelines. It gives the option of a fast deployment of resource with CloudFormation and by using Git Actions it makes it much easier to get a pipeline working without having to setup extra servers and configure them for the job.
