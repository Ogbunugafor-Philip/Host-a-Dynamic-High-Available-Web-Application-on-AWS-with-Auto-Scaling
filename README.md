# Host-a-Dynamic-High-Available-Web-Application-on-AWS-with-Auto-Scaling

## Introduction
In today's fast-paced digital environment, ensuring that web applications are scalable, resilient, and highly available is essential. AWS provides a robust suite of tools, such as Auto Scaling, Elastic Load Balancing (ELB), and multi-AZ deployments, to help you build and maintain such infrastructure.

This project focuses on deploying a dynamic web application capable of handling variable traffic loads, maintaining performance, and ensuring fault tolerance using AWS services.

## Project Objectives
- Deploy a dynamic web application capable of handling traffic spikes.
- Implement fault-tolerant architecture using AWS multi-AZ capabilities.
- Dynamically manage resources with AWS Auto Scaling.
- Ensure high availability and data consistency with Amazon RDS or other managed database services.

## Architecture Overview
The project involves creating a three-tier VPC, setting up secure networking, deploying a LAMP stack application on EC2, and integrating it with other AWS services like RDS, S3, Route 53, and Auto Scaling.

---

## Project Steps

### Step 1: Build a 3-Tier VPC
A Virtual Private Cloud (VPC) is a secure, isolated section of a cloud provider's network where you can run and manage your applications, servers, and other resources. Think of it as your own private data center within the cloud, but without the hassle of setting up and maintaining physical hardware.

**Procedure:**
1. Create a VPC with a CIDR block of `10.0.0.0/16`.
2. Enable DNS hostnames for the VPC.
3. Create two public subnets in two different Availability Zones.
4. Create four private subnets (two for application and two for data).
5. Create and attach an Internet Gateway to the VPC.
6. Configure route tables for public and private subnets.

**Code:**
```bash
# Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Enable DNS Hostnames
aws ec2 modify-vpc-attribute --vpc-id <vpc-id> --enable-dns-hostnames

# Create Subnets
aws ec2 create-subnet --vpc-id <vpc-id> --cidr-block 10.0.0.0/24 --availability-zone us-east-1a
aws ec2 create-subnet --vpc-id <vpc-id> --cidr-block 10.0.1.0/24 --availability-zone us-east-1b
aws ec2 create-subnet --vpc-id <vpc-id> --cidr-block 10.0.2.0/24 --availability-zone us-east-1a
aws ec2 create-subnet --vpc-id <vpc-id> --cidr-block 10.0.3.0/24 --availability-zone us-east-1b

# Create and Attach Internet Gateway
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --vpc-id <vpc-id> --internet-gateway-id <igw-id>

# Create Route Table
aws ec2 create-route-table --vpc-id <vpc-id>
aws ec2 create-route --route-table-id <route-table-id> --destination-cidr-block 0.0.0.0/0 --gateway-id <igw-id>
```

### Step 2: Create a NAT Gateway
A NAT Gateway allows instances in private subnets to access the internet while preventing inbound traffic from reaching them.

**Procedure:**
1. Allocate an Elastic IP.
2. Create a NAT Gateway in a public subnet.
3. Create private route tables and associate them with private subnets.

**Code:**
```bash
# Allocate Elastic IP
aws ec2 allocate-address --domain vpc

# Create NAT Gateway
aws ec2 create-nat-gateway --subnet-id <subnet-id> --allocation-id <eip-allocation-id>

# Create Private Route Table
aws ec2 create-route-table --vpc-id <vpc-id>
aws ec2 create-route --route-table-id <route-table-id> --destination-cidr-block 0.0.0.0/0 --nat-gateway-id <nat-gateway-id>
```

### Step 3: Create Security Groups
Security groups act as virtual firewalls to control inbound and outbound traffic.

**Procedure:**
1. Create security groups for the Application Load Balancer, EC2 instances, and RDS database.
2. Define inbound and outbound rules for each security group.

**Code:**
```bash
# Create Security Groups
aws ec2 create-security-group --group-name ALB-SG --description "ALB Security Group" --vpc-id <vpc-id>
aws ec2 create-security-group --group-name SSH-SG --description "SSH Security Group" --vpc-id <vpc-id>
aws ec2 create-security-group --group-name WebServer-SG --description "Web Server Security Group" --vpc-id <vpc-id>
aws ec2 create-security-group --group-name DB-SG --description "Database Security Group" --vpc-id <vpc-id>

# Add Rules to Security Groups
aws ec2 authorize-security-group-ingress --group-id <sg-id> --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id <sg-id> --protocol tcp --port 443 --cidr 0.0.0.0/0
```

### Step 4: Launch MySQL RDS Instance
MySQL RDS is a managed database service that simplifies setup and scaling.

**Procedure:**
1. Create an RDS subnet group.
2. Launch a MySQL RDS instance.

**Code:**
```bash
# Create RDS Subnet Group
aws rds create-db-subnet-group --db-subnet-group-name <name> --db-subnet-group-description "RDS Subnet Group" --subnet-ids <subnet-id-1> <subnet-id-2>

# Launch RDS Instance
aws rds create-db-instance --db-instance-identifier <db-name> --db-instance-class db.t2.micro --engine mysql --allocated-storage 20 --master-username <username> --master-user-password <password> --vpc-security-group-ids <sg-id>
```

### Step 5: Create S3 Bucket and IAM Role
S3 is used for storing static web files, and IAM roles grant secure access.

**Procedure:**
1. Create an S3 bucket and upload web files.
2. Create an IAM role with an S3 policy.

**Code:**
```bash
# Create S3 Bucket
aws s3api create-bucket --bucket <bucket-name> --region us-east-1 --create-bucket-configuration LocationConstraint=us-east-1

# Upload Files
aws s3 cp /path/to/web-files s3://<bucket-name>/ --recursive

# Create IAM Role
aws iam create-role --role-name S3AccessRole --assume-role-policy-document file://trust-policy.json
aws iam attach-role-policy --role-name S3AccessRole --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
```

### Step 6: Launch and Configure EC2 Instance
Launch an EC2 instance, configure LAMP stack, and deploy the web application.

**Code:**
```bash
# Create Key Pair
aws ec2 create-key-pair --key-name <key-name> --query "KeyMaterial" --output text > <key-name>.pem

# Launch EC2 Instance
aws ec2 run-instances --image-id ami-0abcdef1234567890 --count 1 --instance-type t2.micro --key-name <key-name> --security-group-ids <sg-id> --subnet-id <subnet-id>

# Configure LAMP Stack (SSH into EC2 instance)
sudo yum update -y
sudo yum install -y httpd php mysql php-mysqlnd
sudo systemctl enable httpd
sudo systemctl start httpd
```

### Step 7: Create an AMI
Create an AMI of the configured EC2 instance for consistency.

**Code:**
```bash
aws ec2 create-image --instance-id <instance-id> --name "WebApp-AMI" --description "AMI for Web App"
```

### Step 8: Configure Application Load Balancer (ALB)
Distribute traffic across multiple EC2 instances using ALB.

**Code:**
```bash
# Create Target Group
aws elbv2 create-target-group --name <target-group-name> --protocol HTTP --port 80 --vpc-id <vpc-id>

# Create ALB
aws elbv2 create-load-balancer --name <alb-name> --subnets <subnet-1> <subnet-2> --security-groups <sg-id>
```

### Step 9: Register a Domain with Route 53
Map a domain name to your ALB.

**Code:**
```bash
# Create Route 53 Hosted Zone
aws route53 create-hosted-zone --name <domain-name> --caller-reference <unique-string>

# Add Record Set
aws route53 change-resource-record-sets --hosted-zone-id <hosted-zone-id> --change-batch file://record-set.json
```

### Step 10: Create Auto Scaling Group
Automatically scale EC2 instances to handle traffic spikes.

**Code:**
```bash
# Create Launch Template
aws ec2 create-launch-template --launch-template-name <template-name> --version-description "Version1" --launch-template-data file://launch-template-data.json

# Create Auto Scaling Group
aws autoscaling create-auto-scaling-group --auto-scaling-group-name <asg-name> --launch-template LaunchTemplateId=<template-id>,Version=1 --min-size 1 --max-size 4 --desired-capacity 2 --vpc-zone-identifier <subnet-ids>
```

---

## Conclusion
This project demonstrates how to design, deploy, and manage a dynamic web application on AWS with high availability and scalability. By leveraging AWS services like Auto Scaling, ALB, RDS, and Route 53, you can ensure seamless user experiences, robust security, and efficient resource management.

---

