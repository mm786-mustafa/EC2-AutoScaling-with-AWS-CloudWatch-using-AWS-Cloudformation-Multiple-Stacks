# EC2-AutoScaling-with-AWS-CloudWatch-using-AWS-Cloudformation-Multiple-Stacks
📦 WordPress Deployment on AWS Using CloudFormation
This repository provides an automated solution for deploying a WordPress website on AWS infrastructure using CloudFormation.
The deployment is broken into three separate stacks for modularity and easy management:

1. Networking Stack (VPC, Subnets, IGW, NAT, Route Tables, etc.)

2. Database Stack (RDS MySQL Database)

3. Application Stack (EC2 instances, Auto Scaling Group, Load Balancer)

🏗️ Stack Overview

Stack	Description
Networking Stack	Creates VPC, Public/Private Subnets, Internet Gateway, NAT Gateway, Route Tables, Security Groups
Database Stack	Deploys Amazon RDS (MySQL) Database inside private subnets
Application Stack	Launches EC2 instances using Auto Scaling and Load Balancer for WordPress hosting
📜 Prerequisites
AWS Account with permissions to create EC2, RDS, VPC, Auto Scaling, Load Balancer resources.

Existing EC2 Key Pair.

AWS CLI configured or use directly through the AWS Management Console.

Basic knowledge of CloudFormation and AWS Networking concepts.

🛠️ Deployment Instructions
Step 1: Deploy Networking Stack
bash
Copy
Edit
aws cloudformation create-stack \
  --stack-name wordpress-networking-stack \
  --template-body file://networking-stack.yaml \
  --capabilities CAPABILITY_IAM
This stack will create the network infrastructure (VPC, subnets, NAT gateway, route tables, security group, etc.)

Step 2: Deploy Database Stack
bash
Copy
Edit
aws cloudformation create-stack \
  --stack-name wordpress-database-stack \
  --template-body file://database-stack.yaml \
  --parameters ParameterKey=VpcId,ParameterValue=<VpcId_from_Networking_Stack> \
               ParameterKey=PrivateSubnetIds,ParameterValue=<Comma_separated_private_subnet_IDs> \
               ParameterKey=SecurityGroupId,ParameterValue=<SecurityGroupID_from_Networking_Stack> \
  --capabilities CAPABILITY_IAM
This stack will provision an RDS MySQL database inside the private subnets.

Step 3: Deploy Application Stack
bash
Copy
Edit
aws cloudformation create-stack \
  --stack-name wordpress-application-stack \
  --template-body file://application-stack.yaml \
  --parameters ParameterKey=VpcId,ParameterValue=<VpcId_from_Networking_Stack> \
               ParameterKey=PublicSubnetIds,ParameterValue=<Comma_separated_public_subnet_IDs> \
               ParameterKey=SecurityGroupId,ParameterValue=<SecurityGroupID_from_Networking_Stack> \
               ParameterKey=RdsEndpoint,ParameterValue=<RDSEndpoint_from_Database_Stack> \
               ParameterKey=DBName,ParameterValue=wordpress \
               ParameterKey=DBUsername,ParameterValue=<DBUsername> \
               ParameterKey=DBPassword,ParameterValue=<DBPassword> \
               ParameterKey=KeyName,ParameterValue=<EC2_Keypair_name> \
  --capabilities CAPABILITY_IAM
This stack will launch EC2 instances with a WordPress installation, connect them to the database, and attach them to an Application Load Balancer.

📂 Repository Structure
bash
Copy
Edit
.
├── networking-stack.yaml   # Defines all VPC and networking components
├── database-stack.yaml     # Defines RDS database resources
├── application-stack.yaml  # Defines EC2 launch template, ASG, ALB and WordPress install
└── README.md               # This file
⚙️ Parameters Explained

Parameter	Description
Environment	Deployment environment (dev, testing, production)
VpcCIDR	CIDR range for the custom VPC
HttpAccessCidr	CIDR range allowed for HTTP access
KeyName	Name of existing EC2 keypair for SSH access
DatabaseUsername	Username for RDS
DatabaseUserPassword	Password for RDS
DBInstanceClass	RDS instance type
AvailabilityZones	List of AZs to deploy subnets
TargetGroupName	Name for Target Group (ALB)
LoadBalancerName	Name for Application Load Balancer
🛡️ Security Considerations
The RDS database is only accessible from the EC2 instances via a private security group.

Only HTTP (Port 80) is exposed to the internet through the Load Balancer.

SSH access is only available if you manually open the security group after deployment.

📈 Auto Scaling and Monitoring
Auto Scaling Group dynamically adjusts the number of EC2 instances based on CPU utilization:

Scale Up when CPU > 80%

Scale Down when CPU < 40%

CloudWatch Alarms trigger scaling actions automatically.

🌐 Outputs

Output	Description
ALBDNSName	Public DNS of the Application Load Balancer
RDSEndpoint	Endpoint of the RDS database
📌 Notes
WordPress configuration (wp-config.php) is automatically set up using environment variables.

Default WordPress files are deployed to /var/www/html.

Database settings are populated dynamically from CloudFormation parameters.

🤝 Contributing
If you'd like to improve the templates or fix bugs, feel free to open a pull request!
