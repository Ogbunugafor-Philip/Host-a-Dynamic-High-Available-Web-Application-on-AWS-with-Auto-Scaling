# Host a Dynamic High Available Web Application on AWS with Auto Scaling

![image](https://github.com/user-attachments/assets/d7134a1b-f402-439f-bebc-79c731080ac2)


## Introduction
In the fast-paced digital landscape, a slow or unreliable web application can be the difference between winning and losing customers. Imagine your application thriving under heavy traffic spikes, seamlessly adapting to demand, and never missing a beat—even during peak hours. That’s the power of hosting a dynamic, highly available web application on Amazon Web Services (AWS).
AWS offers an unparalleled suite of tools to help you create a cloud infrastructure that is not only scalable but also resilient and cost-efficient. With features like Auto Scaling, Elastic Load Balancing (ELB), and multi-AZ deployments, AWS empowers you to deliver an exceptional user experience, no matter the circumstances.

## Project Objectives
- To deploy a dynamic web application that can handle variable traffic loads without compromising performance
- To implement a fault-tolerant architecture using AWS's multi-AZ capabilities
- To leverage AWS Auto Scaling to dynamically manage application resources
- To ensure data consistency and high availability using Amazon RDS or other managed database services
  
## Project Steps
i.	Build a 3 tier VPC
ii.	Create a NAT Gateway
iii.	Create Security groups
iv.	Launch MySQL RDS Instance
v.	Create S3 Bucket, upload web files and create IAM role with S3 policy
vi.	Create Key Pairs and set up EC2 Instance
vii.	Install MySQL Workbench; import data into RDS Database with the Workbench
viii.	Install a dynamic website on an EC2 (LAMP Stack)
ix.	Create an AMI
x.	Create an Application Load Balancer
xi.	Register a new domain and create a record set in route 53
xii.	Secure the web application with AWS ACM Certificate
xiii.	SSH into the EC2 instance in the private subnet to update our Web App configuration
xiv.	Create an AMI
xv.	Create Auto Scaling Group



### Step 1: Build a 3 tier VPC
A Virtual Private Cloud (VPC) is a secure, isolated section of a cloud provider's network where you can run and manage your applications, servers, and other resources. Think of it as your own private data center within the cloud, but without the hassle of setting up and maintaining physical hardware.
We would create a VPC with Public and Private Subnet on 2 Availability Zones for High Availability and Fault Tolerant 

- Create a VPC in the AWS Console with IP 4 CIDR 10.0.0.0/16
 ![image](https://github.com/user-attachments/assets/3a40f720-b044-4478-8835-97aa0d3d8319)

- Enable DNS Host name in the VPC. Go to Action -> Edit VPC setting then check Enable DNS hostnames and click save
![image](https://github.com/user-attachments/assets/25ebfd01-2cd0-4ca0-898c-f058f92f4985)


- Create Internet Gateway. Go to Internet Gateway ->Create IGW -> Give it a name and click Create Internet Gateway
![image](https://github.com/user-attachments/assets/4c7eae23-89f3-4130-9410-4500c3b9bf24)

 
- Attach the Internet Gateway to the created VPC. on the Internet Gateway Page, go to Actions -> Attach to VPC -> Select VPC then click Attach internet gateway
![image](https://github.com/user-attachments/assets/a2e7be87-9a6b-4621-b2c6-47b96f4cc812)

 

- Create two Public subnets in two Availability Zones
A public subnet is a subnet that is configured to allow direct access to the internet.
  - Name – Public Subnet AZ 1
    CIDR – 10.0.0.0/24
    Availability Zone – us-east-1a
   - Name – Public Subnet AZ 2
     CIDR – 10.0.1.0/24
      Availability Zone – us-east-1b
- Go to subnets -> Create Subnets ->Select VPC to create subnet fill the details and Create Subnet
  ![image](https://github.com/user-attachments/assets/5da077d1-8f97-44e6-87bf-bdc8a77005d1)

- Do same to create subnet in the second Availability Zone
  ![image](https://github.com/user-attachments/assets/110fcf47-92f3-40df-9740-d00c30812ac6)

  - Enable the auto assign IP settings for the two created Public Subnets. Click the subnet -> Actions -> Edit subnet setting -> Check Enable auto-assign public IPv4 address and click save
![image](https://github.com/user-attachments/assets/260e42a2-379e-4a76-b374-0f1b4c9b119e)

Do same for the second public subnet created

- Create a Route Table
A route table is like a map that tells your network where to send data. It has rules that guide traffic to the right destination, such as another device, network, or the internet.
Go to Route Table -> Create Route Table -> Give it a name and select the VPC to use for the route table then click Create Route Table
![image](https://github.com/user-attachments/assets/12c9a410-1521-477a-9c51-cb7585c3541f)

 
- Add Public route to the route table. On the route page, click edit route -> add route. Use 0.0.0.0/0 for anywhere in the internet and select Internet Gateway then save changes
  ![image](https://github.com/user-attachments/assets/531251df-3b3e-4bea-a118-dc116f137605)

 - Associate the route table with the public subnet. Associating a route table with a public subnet is done to define the network traffic rules for that subnet, specifically to enable communication between the resources in the subnet and the internet.
On the route table page, click Subnet association -> Edit Subnet association -> select the Public subnets created and click Save association
![image](https://github.com/user-attachments/assets/b8acf67e-f41b-4441-86e5-46c41e86f1de)

 
- Create Private Subnet.
A private subnet is a network segment within a Virtual Private Cloud (VPC) that is not directly accessible from the internet. It is used to host resources like databases or internal services that require security and should not be publicly exposed. Access to these resources is typically managed through a NAT gateway
We would create four private subnets with the details below;
  - Name – Private App Subnet AZ 1
    CIDR – 10.0.2.0/24
    Availability Zone – us-east-1a
  - Name – Private App Subnet AZ 2
    CIDR – 10.0.3.0/24
    Availability Zone – us-east-1b
  - Name – Private Data Subnet AZ 1
    CIDR – 10.0.4.0/24  
    Availability Zone – us-east-1a
  - Name – Private Data Subnet AZ 2
    CIDR – 10.0.5.0/24
    Availability Zone – us-east-1b

### Step 2: Create a NAT Gateway
A NAT Gateway (Network Address Translation Gateway) is a service that allows instances in a private subnet to access the internet while preventing inbound internet traffic from directly reaching those instances. It acts as an intermediary between the private subnet and the public internet.

- To create a NAT Gateway, go to the VPC dashboard -> Nat gateways ->Create NAT Gateway.
Give it a name, associate it with the Public Subnet AZ 1 created, allocate elastic IP and click create NAT Gateway
Allocating an Elastic IP  to a NAT Gateway in AWS allows you to provide a static, public IP address for outbound traffic from instances in a private subnet.
![image](https://github.com/user-attachments/assets/02b405e3-12e1-45bc-a635-2d1dcace6034)
 

- Create Private Route Table AZ 1. Go to route table -> create route table -> Give it the name Private Route Table AZ , associate it with our created VPC and click create route table
![image](https://github.com/user-attachments/assets/0f7c8468-b59a-43de-a2ac-88afcd4e4b84)
 
- Add route to the created route table. Edit Route. Use 0.0.0.0/0 for anywhere in the internet and select NAT Gateway (the one we just created in AZ1) then save changes
 ![image](https://github.com/user-attachments/assets/4cb77286-f7f1-44cd-9fb7-4168e8483dc5)

Associate the route table to the private subnets in AZ 1. Click Subnet Association -> Edit Subnet Association -> check all private subnets in AZ 1, then click save association
 ![image](https://github.com/user-attachments/assets/628c87d1-c531-4c03-aef8-e8bc17cdac45)

We have completed our NAT Gateway configuration for AZ 1, do same for AZ 2



### Step 3: Create Security groups
Security Groups are a fundamental component of network security. They act as virtual firewalls for your resources to control inbound and outbound traffic at the instance level.
We are going to create 4 security Groups

| Security Group Name          | Port Name          | Port  | Source                   |
|-----------------------------|-------------------|------|--------------------------|
| **ALB Security Group**       | HTTP and HTTPS    | 80, 443 | 0.0.0.0/0              |
| **SSH Security Group**       | SSH               | 22     | My IP                   |
| **Web Server Security Group**| HTTP and HTTPS    | 80, 443 | ALB Security Group      |
|                              | SSH               | 22     | SSH Security Group      |
| **Database Security Group**  | MySQL/Aurora      | 3306   | Web Server Security Group |

- ALB Security Group: Go to VPC -> Create Security Groups
![image](https://github.com/user-attachments/assets/8d4585fc-46c0-4f21-b7e8-5df014cc66f4)

- SSH Security Group: Go to VPC -> Create Security Groups
![image](https://github.com/user-attachments/assets/c5b7202e-6c33-4700-a0c5-8d131ef26f2c)

 - Webserver Security Group: Go to VPC -> Create Security Groups
 ![image](https://github.com/user-attachments/assets/9e35b7cc-c0a2-4205-b29c-a89521480512)

- Database Security Group: Go to VPC -> Create Security Groups
 ![image](https://github.com/user-attachments/assets/cb93b988-1bb2-43dc-aabb-2a8f11b470e4)

### Step 4: Launch MySQL RDS Instance
MySQL RDS (Relational Database Service) is a managed database service provided by Amazon Web Services (AWS). It allows you to deploy, manage, and scale MySQL databases in the cloud without worrying about the underlying infrastructure.
Before we create our RDS, we need to create subnet group. An Amazon RDS Subnet Group defines where the database instance will be launched within your Virtual Private Cloud (VPC).
- Search for RDS in the console -> Subnet Groups -> Create DB subnet groups
Below are the details to fill:
  - Name
  - Description
  - VPC: Choose the VPC we created
  - Availability Zone: Choose AZ 1 and AZ 2
  - Subnets: Select the IP of our 2 Database subnets
Then click create
![image](https://github.com/user-attachments/assets/338bc9fb-5206-4c9c-b223-52e6776fa4b4)

Next is to create a Data Base
- Search for RDS in the console -> Databases -> Create Database
- Below are the details to fill:
    - Database Creation Method: Standard Create
    - Engine Option: MySQL
    - Engine Version: 5.7.44
    - Template: Dev/Test
    - Deployment Options: Single Database instance. If you want to have a master and a standby database, use Multi AZ DB-instance
    - DB Instance Identifier: dynamicdatabase
    - Security: Click self-managed and choose your admin name and password
    - Instance Configuration: On include previous generation classes and select Burstable class
    - Connectivity (Virtual Private Cloud): Select our VPC
    - Subnet Group: Select the subnet we created
    - Security Group: Choose the existing Database Security group we created
    - Availability Zone: Choose 1b where our Master database is
    - Additional Configuration (Initial Database Name): Dynamicdb
Create Database
![image](https://github.com/user-attachments/assets/86e877ec-d4ad-407b-8d49-7f440bedb56b)
 
### Step 5: Create S3 Bucket, upload web files and create IAM role with S3 policy
An S3 bucket is a container for storing data in Amazon S3. It serves as the foundational structure for organizing and managing objects (files) in S3. Buckets can store unlimited amounts of data and are designed for flexibility, durability, and security.
An IAM role with an S3 policy allows applications, services, or users to securely access an Amazon S3 bucket with specific permissions. This IAM role would allow our EC2 instance to download the web-files on our S3 bucket.

Firstly, let us create S3 bucket and upload our web files into it
- Search and click S3 in the AWS search bar
 ![image](https://github.com/user-attachments/assets/a282c301-a1f7-42b5-9812-7f706aae59a6)

- Click Create bucket and fill the following details
    - Bucket Name: Philip-rentzone-webapp
    - Select Region:
Click create bucket
![image](https://github.com/user-attachments/assets/73dd06a3-1aa8-4f77-aa14-5b68ab6e02ff)


- To upload our web-file in the S3 bucket. Select the bucket we just created and click upload. On your local computer, go to where you saved the web-file, drag it into the S3 bucket and click upload
![image](https://github.com/user-attachments/assets/9888b866-646c-449a-9862-0453fa9aa4c9)

To create an IAM role with S3 policy ( this policy would allow our ec2 instance to access the web-files on our S3 bucket)
- Type IAM and select it
 ![image](https://github.com/user-attachments/assets/ab53b27b-28a6-4e9d-baf4-b3897eb42a90)

- Select Roles and click create roles
![image](https://github.com/user-attachments/assets/4db36bef-ad0b-43ac-b1ea-e5b7da356f09)

- Under Trusted entity type choose AWS Services and under Service or use case, select EC2 and click next
 ![image](https://github.com/user-attachments/assets/659a529b-83a9-4e24-bb51-5152e0673b92)

- Search and click AmazonS3FullAccess and click next
  ![image](https://github.com/user-attachments/assets/9671c923-6f54-439d-8eb5-caf502b64229)

 - Give the role a name and click create role
 ![image](https://github.com/user-attachments/assets/95fc070c-fb3a-4e36-9c79-0901cd0ad56b)


### Step 6: Create Key Pairs and set up EC2 Instance
A key pair consists of a public key and a private key that are used for securely connecting to instances, such as EC2 (Elastic Compute Cloud) instances.

To create a key pair,
- Search and select EC2 in the search bar
![image](https://github.com/user-attachments/assets/5e0543cc-e25d-4c15-8ef3-c02778b98194)

- On the ec2 dashboard, under network and security, click key pairs and click again create key pairs
 ![image](https://github.com/user-attachments/assets/e37084a8-3bb4-469b-bdbf-183cee9e307c)

- Give the key pair a name, select RSA in the key pair type, select pem in the private key file format and click create key pair
 ![image](https://github.com/user-attachments/assets/8592dbe5-7a64-445d-a840-26374cef4846)

Next, we would need to set up an EC2 instance

An EC2 instance (Elastic Compute Cloud instance) is a virtual server in Amazon Web Services (AWS) that allows you to run applications and services in the cloud. EC2 instances are scalable and can be configured based on your requirements for CPU, memory, storage, and networking capabilities.
To create an EC2 instance,

- Search and select EC2 in the search bar
 ![image](https://github.com/user-attachments/assets/b04410de-bc8a-4747-8a4f-a2ce1572b72b)

- Click instance and click on launch instance
 ![image](https://github.com/user-attachments/assets/666d2068-257a-4bfe-aa77-c6bc60a4ebdd)

For the instance set up, these are the details to fill;
- Name: Setup Server
- Application and OS Image: Amazon Linux
- Instance type: t2.micro
- Key Pair: Select the one we just created
- Network Settings: Click Edit
      - VPC: Select our created VPC
      - Subnet: Select Public Subnet AZ 1
      - Security Group: Select existing ones (ALB, SSH and Webserver security group)
- Advanced Details (IAM Instance Profile): Select S3 Role
Click Launch Instance
 ![image](https://github.com/user-attachments/assets/4809a0a3-1359-4a6e-94e5-907eb9d2da43)


### Step 7: Install MySQL Workbench; import data into RDS Database with the Workbench
MySQL Workbench is a powerful and versatile integrated development environment (IDE) for working with MySQL databases. It provides a graphical user interface (GUI) for database design, SQL development, and server administration.

- Open google and search MySQL workbench and choose the one with www.mysql.com
 
![image](https://github.com/user-attachments/assets/263abe69-9f31-4b1e-8dfb-28b96dde2f80)

- Click on download now -> go to download page -> Download the first one
 ![image](https://github.com/user-attachments/assets/ab21cfd2-1806-4867-8000-5868172fab83)

- After downloading, click on the package and click custom
 ![image](https://github.com/user-attachments/assets/f7a3d559-6783-48c5-9532-7f4cd8aa4711)

Follow the prompt and complete the installation
![image](https://github.com/user-attachments/assets/01c12d80-17de-4a04-9e37-5b89fe2f6db2)
 
We would now use MySQL work bench and the set up server to import the SQL for our application
- Make sure you have the SQL file for the application
- On the management console search ec2 and click instances
![image](https://github.com/user-attachments/assets/f90061f9-c505-4803-9d0d-b4a0c02cba30)
 
- Go to the AWS logo on the top left, right click and select open link in new tab. This is done because we want to have our EC2 and RDS server open hand in hand
 
![image](https://github.com/user-attachments/assets/903a2866-f975-45a2-85bc-af0f39a5f3ed)

- On the second page, let us open our RDS instance
 ![image](https://github.com/user-attachments/assets/7580bd4b-8e2a-4ccc-a604-df59b82f0208)

- Open MySQL work bench ->Database -> connect to database
 ![image](https://github.com/user-attachments/assets/a80f3463-273c-4d42-b022-524e04a628d7)

These are the detains we would need to fill in the MySQL work bench to connect to our database
- Connection Method: Standard TCP/IP over SSH
- SSH Host name: This is our EC2 setup server public dns
- SSH Username: Our setup server username (ec2-user)
- SSH Key File: This is our private key pair. Click store in vault and import the key from where it is saved in your local system
- MySQL Hostname: This is our endpoint in our RDS server
- Username: RDS username
- Password: RDS Password

Then click ok… Our Workbench has successfully connected to our RDS Database
 ![image](https://github.com/user-attachments/assets/34ac39a0-745e-4048-8e39-d7db2e83f28f)

- Click data import on the MySQL work bench
![image](https://github.com/user-attachments/assets/c5efc646-68f2-41b3-ac41-d903627a7248)

- Select the import from the self-contained file
 ![image](https://github.com/user-attachments/assets/b8a704d9-7830-43d2-9790-11d75de70492)

- Click the three dots and browse to the location where you have your SQL file
 ![image](https://github.com/user-attachments/assets/2e90616f-2b26-43d2-a299-b1bda02ecba3)

- In default target schema to be imported, select your db name
![image](https://github.com/user-attachments/assets/ab2e02fe-4b3d-4d78-b3d1-4d77ffb9badb)
 
- Click start import
 ![image](https://github.com/user-attachments/assets/4d3efc74-2f7a-48dc-ad99-b4688de4f36b)

- Import Completed
 ![image](https://github.com/user-attachments/assets/177f33b3-3d9d-46c7-82c6-63d2f3ccbf70)


### Step 8: Install a dynamic website on an EC2 (LAMP Stack)
The LAMP stack is a popular open-source software stack used to develop and deploy dynamic websites and web applications. The acronym LAMP stands for:
- L - Linux: The operating system that forms the foundation of the stack.
- A - Apache: The web server software that handles requests and serves web content to users.
- M - MySQL: The database management system used to store and retrieve data.
- P - PHP, Perl, or Python: The programming language used to create dynamic content and interact with the database.

LAMP is widely used to build dynamic web applications, from small personal projects to large-scale enterprise solutions.

Now that we’ve understood LAMP Stack, let us install it on our set up server
SSH into our set up server from our local host. To achieve this,
- Open PowerShell
 ![image](https://github.com/user-attachments/assets/aa730160-666e-48e5-8fda-6d5ea11c5dae)


- In your PowerShell, go to the folder where you have your key pair (downloads)
```sh
cd Downloads
``` 

- Go to your ec2 instance and select it, at the top, click connect -> Click  SSH client and copy the connection example
```sh
ssh -i "ec2-keypair.pem" ec2-user@ec2-52-91-75-153.compute-1.amazonaws.com
```
Copy and paste the above command on your PowerShell and click ok

Now we have successfully SSH to our set up server from our local computer
 ![image](https://github.com/user-attachments/assets/c67a05bc-09da-4bef-b22d-2bb3bce97ce2)

We would run the following commands to install our LAMP Stack
- update ec2 instance
```sh
sudo yum update -y
```
install apache 
```sh
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd
```
- install php 7.4
```sh
sudo amazon-linux-extras enable php7.4
sudo yum clean metadata
sudo yum install php php-common php-pear -y
sudo yum install php-{cgi,curl,mbstring,gd,mysqlnd,gettext,json,xml,fpm,intl,zip} -y
```
install mysql5.7
```sh
sudo rpm -Uvh https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
sudo yum install mysql-community-server -y
sudo systemctl enable mysqld
sudo systemctl start mysqld
```
- download the rentzone zip from s3 to the html directory on the ec2 instance
```sh
sudo aws s3 sync s3://philip-rentzone-webapp /var/www/html
```
- unzip the rentzone zip folder
```sh
cd /var/www/html
sudo unzip rentzone.zip
```
- move all the files and folders from the rentzone directory to the html directory
```sh
sudo mv rentzone/* /var/www/html
```
- move all the hidden files from the rentzone directory to the html directory
```sh
sudo mv rentzone/.well-known /var/www/html
sudo mv rentzone/.env /var/www/html
sudo mv rentzone/.htaccess /var/www/html
```
- delete the rentzone and rentzone.zip folder
```sh
sudo rm -rf rentzone rentzone.zip
```
- enable mod_rewrite on ec2 linux
```sh
sudo sed -i '/<Directory "\/var\/www\/html">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf
```
- set permissions
```sh
sudo chmod -R 777 /var/www/html
sudo chmod -R 777 storage/
```
- add database credentials
```sh
sudo vi .env
```
![image](https://github.com/user-attachments/assets/911a8146-be42-4040-bcbf-c53f16e3f1fb)
 
- restart server
```sh
sudo service httpd restart
```
After the configuration, copy your instance Public IP address and paste in your browser; your website would display
 
![image](https://github.com/user-attachments/assets/c84d5381-78b0-4df7-ae99-8542ae8ff169)


### Step 9: Create an AMI
An Amazon Machine Image (AMI) is a pre-configured virtual machine template provided by Amazon Web Services . It includes the operating system, application server, and applications necessary to launch and run instances (virtual servers) in the AWS environment. Each AMI is like a snapshot that contains all the essential information to create and run an EC2 instance.
Creating an Amazon Machine Image (AMI) is useful for several reasons, especially in the context of scalability, consistency, and disaster recovery in AWS environments. One of the major purposes of creating AMI is to ensure that all EC2 instances have the same operating system, configurations, and installed applications.

To create and AMI,

- Search and select EC2 in the search bar
 ![image](https://github.com/user-attachments/assets/e253b9b8-d3aa-4420-9be0-eb48bb03cfbe)

- Click instances -> instances running
 ![image](https://github.com/user-attachments/assets/3407ab81-bb28-41eb-b218-57456e974766)

- Select your running instance -> Actions -> Image and Template -> Create image
 ![image](https://github.com/user-attachments/assets/5c26671c-a4a6-499b-8a52-aae8558e012c)

- Enter the details
    - Image Name: Rentzone AMI
    - Description: Rentzone AMI
    - Tags: Click tag image and snapshot together
    - Click add tag: Under key “Name” and under Value “Rentzone AMI”
Then click create image
 ![image](https://github.com/user-attachments/assets/f28266cb-74de-45fa-8dd9-fd9c07adb0b9)

As a best practice, create an AMI and snapshots of your EC2 instance to capture the entire configuration, including the operating system, application, and data. This approach ensures consistent backups, simplifies disaster recovery, and allows for seamless scaling by launching additional instances with the same setup whenever needed.

### Step 10: Create an Application Load Balancer
Application Load Balancer (ALB) intelligently distributes incoming application traffic across multiple targets (e.g., EC2 instances, containers, or IP addresses) based on advanced routing rules, ensuring scalability, availability, and efficient request handling.

We would use the AMI we created to launch and ec2 in instance in a private subnet (The AMI already has our website). We would next, create a target group and add the instance into the target group, then we would create an application load balancer to route traffic to the target group. This is how users on the internet would be able to access our website.

- We would need to stop our set up server. Go to ec2 -> Instance -> Select set up server -> Instance -> Click stop
 ![image](https://github.com/user-attachments/assets/0b8bd75b-c567-42b6-bb9e-ea2d56606948)


- Launch Instance in the private subnet. Click Launch instance
 ![image](https://github.com/user-attachments/assets/8434793b-d6c1-461b-bef3-49cc73c2ea00)


- For the instance set up, these are the details to fill;
    - Name: Webserver AZ 1
    - Application and OS Image: Go to My AMI and select the one we created
    - Instance type: t2.micro
    - Key Pair: Select the one we just created
    - Network Settings: Click Edit
        - VPC: Select our created VPC
        - Subnet: Select Private App Subnet AZ 1
        - Security Group: Select existing ones (Webserver security group)
Click launch instance
![image](https://github.com/user-attachments/assets/e85af5b6-fa79-4e76-ba22-e1594434cf2f)
 

- Let us create our Target Group. Go to instance, under load balancing, click target group
 ![image](https://github.com/user-attachments/assets/a65200c9-f83c-4b17-a525-512a0d735328)


- Click create target group
![image](https://github.com/user-attachments/assets/6f4d2b22-e2a0-456f-a02f-c986bfcd8503)
 
Target Group Configurations
- Choose a target type: Instance
- Target Group name: Dynamic-TG
- Protocol, Ports (HTTP): 80
- VPC: Select our VPC
- Protocol Version: HTTP Version 1
- Advanced Health Check Setting (Status Code): 200, 301, 302
Then click next
![image](https://github.com/user-attachments/assets/9b4b7123-37a4-4db9-b10f-88b69f2665d5)
 
- We would need to attach our Instance to the Target Group. Select the instance, click include as pending below then click create target group
 ![image](https://github.com/user-attachments/assets/523281a9-2aae-418e-b3eb-1931d2e713b3)

- Next is to Create Application Load Balancer to route traffic to the target group
Select Load Balancing and click create load balancer
![image](https://github.com/user-attachments/assets/c1814064-65fc-492b-9c44-c8cb6f55f687)
 
For the Load Balancer configuration
- Load Balancer Type: Application Load Balancer
- Load Balancer Name: Dynamic-ALB
- Schema: Internet Facing
- IP Address Type: IPv4
- VPC: Choose our VPC
- Mapping: Select us-east-1a (Public Subnet AZ 1) 
                  us-east-1b (Public Subnet AZ 2)
- Security Group: Select ALB Security Group
- Listeners and Routing
    - Protocol: HTTP
    - Port: 80
    - Default Action: Select our Target Group
Click create load balancer
![image](https://github.com/user-attachments/assets/e317e251-6697-4844-93b7-4f357da5cbb9)
 
If we have copy the dns name of the load balancer and paste in our browser, we would see our web app
 ![image](https://github.com/user-attachments/assets/ab04f458-753f-4e76-bb2f-75fc7b386cfb)

- We would need to terminate our set up server as we do not have any need for it again
Go to ec2 -> Instance -> Select set up server -> Instance -> Click Terminate
 ![image](https://github.com/user-attachments/assets/e5650a23-7109-4492-95c7-6902dc7be53d)

### Step 11: Register a new domain and create a record set in route 53
A domain name is the unique, human-readable address used to access a website on the internet. It serves as a way for users to navigate to your website without needing to remember the numerical IP address that identifies the server where your site is hosted.

We would register a domain so that our users can access our web application with it instead of using the Load balancer dns. To register a domain;

- Go to route 53 -> Register domain
 ![image](https://github.com/user-attachments/assets/5132ae68-f786-4a77-a33b-6eb45e703ede)

Enter the domain name you want and your contact details then click create. Domain name creation costa $12.
Next, we would need to create a record set in route 53

- Search and select route 53. On the route 53 dashboard, select hosted zones
![image](https://github.com/user-attachments/assets/4e2dd5ac-64de-4901-98f3-2623867847d2)
 
- Select your domain name and click create records
 ![image](https://github.com/user-attachments/assets/11bc88f9-592c-4127-be9b-e2a69e2cca98)


- Fill the following
    - Record name: www
    - Record Type: A
    - Turn on Alias (Alias is a pointer record that routes traffic to specific AWS resources, such as CloudFront distributions, S3 buckets, or ELB instances, without incurring additional DNS query charges)
    - Under Route traffic to: Select Alias to Application and Classic Load Balancer
    - Region: us-east-1
    - Choose your created load balancer
Click create record
 ![image](https://github.com/user-attachments/assets/0f131b3a-505c-4f65-9011-669aa17b331b)

When you paste your domain name in the browser, you’ll see your website
 

### Step 12: Secure the web application with AWS ACM Certificate
Securing a web application with an AWS ACM Certificate ensures encrypted data transmission, enhances trust, and meets compliance standards.

We would first create our certificate

- Search and click Certificate Manager, then click request
 ![image](https://github.com/user-attachments/assets/4d9e0fe6-188d-4d9e-9898-7fd58b0cb145)


- Enter your domain name, click add another name to this certificate. In the new name, add the wild card *.philipogbunugafor
 ![image](https://github.com/user-attachments/assets/621cac23-fda5-4d03-a5db-0cbcae66d5b2)


- Click request
![image](https://github.com/user-attachments/assets/ae95aeb7-1e11-4e02-a42e-5043dae43a3f)
 
- To validate our certificate, click create record set in route 53
 ![image](https://github.com/user-attachments/assets/484993d7-b713-4716-b136-cbcc9c61a149)


- Check the two domains and click create record
 ![image](https://github.com/user-attachments/assets/90f2c51a-022d-4e28-811f-2d54a70c24dc)

Next we would need to secure the web application with AWS ACM Certificate. To do this, we would add a new listener to our Application Load Balancer that listens to traffic on port 443
- Search and click on ec2
- Under Load balancing ->Load Balancer and select your Application Load Balancer
- Under the listener tab, click on add listener
 ![image](https://github.com/user-attachments/assets/eefc0bd3-063a-4ca8-9333-80e564d0166b)


- Under Protocol, write HTTPS; Routing Action, Forward to target groups and select your target groups
![image](https://github.com/user-attachments/assets/a121417b-b897-484d-868c-653d79cd0003)
 

- Under default SSL/TLS select the Certificate we created
 ![image](https://github.com/user-attachments/assets/5350684c-02d4-4e6f-a141-447de73bc5a0)

- Click Add
 ![image](https://github.com/user-attachments/assets/4c1a74bc-cd5c-43a6-b058-c5aa7b3082eb)

Next we need to edit our port 80 traffic to redirect traffic to port 443. To do that;

- Under listeners and rules, select port 80 -> Manage listener -> Edit listener
![image](https://github.com/user-attachments/assets/1d928f6f-b559-43e8-b61b-daeb91424678)
 

- Select redirect to url and select HTTPS and port 443
![image](https://github.com/user-attachments/assets/c25d69bd-3801-4688-8fe0-d8876d6da6fd)
 

- Click save changes
 ![image](https://github.com/user-attachments/assets/2a342a87-b66e-4267-9681-6dfdbab330fe)


### Step 13: SSH into the EC2 instance in the private subnet to update our Web App configuration
After we uploaded our certificate to secure our website, our web page stopped displaying properly. Majorly it does not display the CSS of the web file we uploaded. We would need to ssh into our instance and make some configurations

First we would lunch a Bastion host in the public subnet and from the bastion host, we can SSH into our instance in the private subnet

A bastion host is a secure server used to provide controlled access to a private network, typically serving as a jump box for administrators to connect to resources in a restricted or isolated environment.

- To create a Bastion Host, in the management console type and select ec2. Then select instance running
 ![image](https://github.com/user-attachments/assets/a2651938-a8e6-4ba4-a898-d4d8f4ca8f56)

- Click on Launch Instances
 ![image](https://github.com/user-attachments/assets/bb912d56-ca1b-45b0-a26d-5746be16e467)


- For the instance set up, these are the details to fill;
    - Name: Bastion Host Application and OS Image: Amazon Linux 2 AMI
    - Instance type: t2.micro
    - Key Pair: Select the one we just created
    - Network Settings: Click Edit
        - VPC: Select our created VPC
        - Subnet: Select Public Subnet AZ 1
        - Security Group: Select existing ones (SSH security group)
Click launch instance

Use PowerShell to SSH into our ec2 instance in the private subnet
- Open PowerShell as an administrator
![image](https://github.com/user-attachments/assets/d64842bf-a2ef-403b-b1c8-ecf879879dc4)
 
- Run the below command to confirm that the SSH agent is running. An SSH agent is a tool that stores your SSH keys in memory so you don’t have to enter your password every time you connect to a remote server. It keeps your private keys secure and makes logging in faster and easier.
- Check if ssh-agent is running
```sh
Get-Service ssh-agent
```
- Start the service
```sh
Start-Service ssh-agent
```
- This should return the status of Running
```sh
Get-Service ssh-agent
```
- Now load your key files into the ssh-agent
```sh
ssh-add C:\WINDOWS\system32\ec2-keypair.pem
```

- Next we ssh into the Bastion Host. Go to your ec2 instance and select the Bastion Host instance, at the top, click connect -> Click  SSH client and copy the connection example
```sh
ssh -i "ec2-keypair.pem" ec2-user@ec2-54-81-86-144.compute-1.amazonaws.com
```
Copy and paste the above command on your PowerShell and click ok
![image](https://github.com/user-attachments/assets/a6c13b01-ef51-440f-b82d-59e95aa75952)
 

We have successfully ssh into our bastion host

We would use the bastion host to ssh into our private server. To do that,
- Go to our web server and copy the private IP
![image](https://github.com/user-attachments/assets/8e29d3fc-afe5-4c38-ad52-f26e31541d70)
 

- Run this command,
```sh
ssh -i “ec2-keypair.pem” ec2-user@10.0.2.122
```
 ![image](https://github.com/user-attachments/assets/f081662d-d5e8-4c34-8d28-f5d90e65ca04)

We have successfully ssh into our private webserver

- Change directory into the html directory
```sh
cd /var/www/html
``` 

- We need to edit the .env file
```sh
sudo vi .env
```
- The below things are to be updated
APP_ENV: Replace local with production
APP_URL: our domain name
![image](https://github.com/user-attachments/assets/03aece33-ad51-4860-8275-a18ad170b05d)
 
- Next we need to update the APPSERVICE.PHP FILE
```sh
cd app
cd Providers
sudo vi AppServiceProvider.php
```
Copy and paste the below code under public function boot
```sh
if (env('APP_ENV') === 'production') {\Illuminate\Support\Facades\URL::forceScheme('https');}
```
![image](https://github.com/user-attachments/assets/002f50b6-1819-49d1-ad9a-582fa92ea9a4)

- Restart apache with the command
```sh
sudo service httpd restart
```
Our website is now displaying correctly and secured
![image](https://github.com/user-attachments/assets/a4635961-571e-43eb-90af-26bc9d66cbda)
 

### Step 14: Create an AMI
We will create a new AMI to reflect the changes we have made on our web server

To create and AMI,

- Search and select EC2 in the search bar
![image](https://github.com/user-attachments/assets/f8e36fd8-6e88-4498-960c-96df037d3218)
 
- Click instances -> instances running
![image](https://github.com/user-attachments/assets/096d436e-b68f-49bb-a030-55042592f16f)
 

- Select your running instance -> Actions -> Image and Template -> Create image
![image](https://github.com/user-attachments/assets/fc0bb171-b299-45f4-8369-6cb55a640e47)
 
- Enter the details
    - Image Name: Rentzone AMI Version 2
    - Description: Rentzone AMI Version 2
    - Tags: Click tag image and snapshot together
    - Click add tag: Under key “Name” and under Value “Rentzone AMI Version 2”
Then click create image
![image](https://github.com/user-attachments/assets/1263f465-e967-459d-80a5-2554e87227e4)
 

### Step 15: Create Auto Scaling Group
An Auto Scaling Group (ASG) in AWS is a feature that automatically manages a group of Amazon EC2 instances to maintain application availability and optimize performance. It adjusts the number of instances dynamically based on conditions you define, ensuring your application can handle varying loads efficiently.

- We would need to terminate our instances running. Go to ec2 -> instance running -> check the 2 instance -> click instance state and select terminate
![image](https://github.com/user-attachments/assets/3972aefc-00cc-4acb-b958-7cfbb5969749)
 
- We would create a lunch template. The launch template would contain all our configurations that the ASG would use to create instances in the private subnet
Under instance ->Launch template -> Create launch template
 ![image](https://github.com/user-attachments/assets/f6ecba2e-6612-4d7d-8567-63039d397ccd)


- Fill the following in the launch template
    - Launch template name: Dynamic_Launch_Template
    - Launch template description: Launch template for ASG
    - Auto Scaling Guidance: Check the box
    - Application and OS Image: My AMIs (Choose the latest version, version 2 we created)
    - Key Pair: Select our Key Pair
    - Instance Type: t.2micro
    - Firewall: Choose existing and choose Webserver security group
Click create launch template
![image](https://github.com/user-attachments/assets/a5eb3235-d818-4e83-9775-83f0b9df18a5)
 
Next we will create our Auto Scaling Group
- Go to EC2 -> Auto Scaling -> Auto Scaling Groups -> Click Create Auto Scaling group
 ![image](https://github.com/user-attachments/assets/164531ef-496a-48ef-bf01-a9357fa19e3e)

- Fill the following details;
    - Auto Scaling group Name: Dynamic ASG
    - Launch Template: From the drop down, select our launch template
Click Next
 ![image](https://github.com/user-attachments/assets/fd3ce46e-c3c0-468d-a52e-7d5d134ea00e)


    - Network (VPC): From the dropdown, select our VPC
    - Availability Zone and Subnet: Choose Private App Subnet AZ1 and AZ2
 ![image](https://github.com/user-attachments/assets/00c7fb85-f0e1-44d2-9c16-a903d537faff)


    - Load balancing: Click attach to an existing load balancer
    - Attach to an existing Load Balancer: From the drop down, choose our load balancer
    - Health Checks: Check ELB
    - Monitoring: Check enable group metrics collection within cloud watch
Click next
 ![image](https://github.com/user-attachments/assets/5216cc1f-ce7b-41a9-bd65-627bdb4d18c2)

    - Desired Capacity: 2
    - Minimum Capacity: 1
    - Maximum Capacity: 4
Click Next
![image](https://github.com/user-attachments/assets/b64f12a8-5c3d-4e91-be50-9bcb2381ff36)
 

- Under Notifications, click add notifications
    - Click create topic. Give the topic a name and select the email to get the notification
Click next
 ![image](https://github.com/user-attachments/assets/2e6d7a8d-2267-4b90-9a87-1bf226a47754)

- Under tags click add tags. Key: Name and Value: ASG Webserver
 ![image](https://github.com/user-attachments/assets/51cfc453-65ad-4004-bf09-5a8252b5eac0)

Click next, scroll down and click create auto scaling group
![image](https://github.com/user-attachments/assets/05e76ccb-2fc4-4f17-8a3b-7da0680ec8c9)
 
Now, let us access our completed website. Type our domain in the browser
 ![image](https://github.com/user-attachments/assets/a1589405-e6db-4674-b0a7-7ac88bf3c24b)

Our dynamic website is perfectly working

## Conclusion
In conclusion, hosting a dynamic, highly available web application on AWS using Auto Scaling ensures a robust, scalable, and secure infrastructure. By implementing a three-tier VPC, leveraging services like RDS for database management, S3 for storage, and Route 53 for DNS management, this architecture optimizes performance and availability. The integration of an Application Load Balancer and Auto Scaling Group dynamically manages traffic and resource utilization, providing seamless user experiences even under fluctuating loads. Securing the application with an AWS ACM Certificate further enhances trust and data protection. This project not only demonstrates best practices in cloud architecture but also showcases the flexibility and power of AWS in supporting dynamic web applications.

