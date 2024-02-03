# Deployment of 3-Tier Web Application Architecture With AWS
Deployment of three-tier web application on AWS involves setting up three layers: presentation (frontend), logic tier (backend), and data tier (database).
This guide to deploying such an application on AWS using commonly used services like EC2,VPC,RDS,ELB, and more.

# Architecture of the Project
![3tier-architecture1](https://github.com/Shimanshushinde/3-Tier-Web-Application-Architecture-with-AWS/assets/137445826/956b9025-fbba-4631-bd83-cdd8a36cae1e)

## Prerequisites

Before proceeding, ensure the following:

- AWS account with necessary permissions.
- Familiarity with the AWS Management Console.
- Familiarity with VPC network structures, EC2 instances, Auto scaling groups, and security groups.
- Familiarity with Linux commands, scripting, and SSH.
- Access to a command line tool.

## Steps to Deploy
### Step 1: Set Up Underlying Network Architecture
This base network consists of:
1. **Create a VPC**
   - In the VPC Console, create a new VPC. select the "VPC and more" option.
2. **Create Subnets**
   - 2 public subnets spread across two availability zones (Web Tier).
   - 2 private subnets spread across two availability zones (Application Tier).
   - 2 private subnets spread across two availability zones (Database Tier).
   - 1 public route table that connects the public subnets to an internet gateway.
   - 1 private route table that will connect the Application Tier private subnets and a NAT gateway.
   - Make sure "Enable auto-assign public IPv4 address" for BOTH public subnets.
3. **Create an Internet Gateway**
   - Attach to VPC     
3. **Create a NAT Gateway**
   - Navigate to "NAT Gateways" section and click on "create NAT gateway".
   - Select one of the public subnets, allocate an elastic IP, and create.
4. **Configure route tables**
   - Associate the public route table with "Internet Gateway".
   - Associate the private route table with "NAT Gateway".
### Step 2: Tier 1: Web tier (Frontend)
1. **Create a web server Launch Template**
   - In the EC2 console, create "Launch templates".
     - AMI: Amazon 2 Linux
     - Instance type: t2.micro 
     - A new or existing key pair
   - create a new security group with inbound SSH, HTTP, and HTTPS rules.
   - Under ‘Advanced details >[User Data](https://github.com/Shimanshushinde/3-Tier-Web-Application-Architecture-with-AWS/blob/e687d4a2beda1a7585e19ee079ef285c3787610e/User%20Data)
     ```sh
     #!/bin/bash

     #install apache
     sudo yum -y install httpd

     #enable and start apache
     sudo systemctl enable httpd
     sudo systemctl start httpd
     ```


2. **Create an Auto scaling group (ASG)**
   - Navigate to the ASG console, select "Launch Template" and create a new group.
   - Select the "VPC" along with the two public subnets.

3. **Application load balancer (ALB)**
   - ALB to be "Internet-facing," needs to 'listen’ over HTTP on port 80 and a target group that routes to ec2 instance.
   - Add a scaling policy > Target Tracking Policy (Average CPU Utilization at 50%).
   - Group size:- Desired capacity: 2
   - Review the ASG settings and create the group
  
### Step 3: Tier 2: Application tier (Backend)
1. **Create an application server Launch Template**
   - In the EC2 console, Create a "Launch templates".
      - (Amazon 2 Linux, t2.micro-1GB, same key pair).
   - create a new security group to allow "All ICMP–IPv4" & "SSH" from the WebServer-SG, and "MYSQL" from Database-SG.
   - Under ‘Advanced details, User Data.
       ```
       #!/bin/bash
       sudo yum install mysql -y
       ```
   - Review and create the template.

2. **Create an Auto Scaling Group (ASG)**
   - Navigate to the ASG console, select "Launch Template" and create a new group.
   - Select the "VPC" along with the two private subnets.

3. **Application Load Balancer (ALB)**
   - This time, we want the ALB to be "Internal," since we’re routing traffic from our Web Tier, not the Internet.
   - Group size:- Desired capacity: 2
   - Review the ASG settings and create the group

### Step 4: Tier 3: Database tier (Data storage & retrieval)
1. **Create a DB subnet group**
   - Select VPC, Select at least 2 availability zones and 2 subnets for Database and Click on create.
    
2. **Create an RDS database**
   - Under the RDS console and Choose a database creation "Standard Create" with a MySQL engine.
   - Choose Template "Production or Dev/Test".
   - Deployment options "Multi-AZ DB instance".
   - Type the database name and password
   - Select instance type and Storage.
    
3. **Create a database security group**
   - Create a new security group called, "Database-SG." Make sure the VPC is selected.
   - Add inbound AND outbound rules that allow MySQL requests to and from the application servers on port 3306.
   - Do the same for the AppServer-SG.

### Step 5: Test Your Application
1. **Use SSH Connection To Connect To The Web Server**
   - SSH into our EC2 server, with public ip and keypair.
  
     ```sh
     ssh -i <KEY_PAIR_NAME> ec2-user@<PUBLIC_IPV4_ADDRESS>
     ```
   - Upload keypair to server and change permission.
     ```
     vi <KEY_PAIR_NAME>
     ```
     ```
     Chmod 400 <KEY_PAIR_NAME>
     ```
   - Ping the private IP address of one of the app server EC2s.
     ```
     ping <PRIVATE_IPV4_ADDRESS>
     ```
2. **Connect To Application Server In A Private Subnet From Web Server**
   
     ```
     ssh -i <KEY_PAIR_NAME> ec2-user@<PRIVATE_IPV4_ADDRESS>
     ```
3. **Connect to the Database From App Server**

   ```
   mysql -h <YOUR_DB_ENDPOINT> -P 3306 -u <YOUR_DB_USERNAME> -p
   ```
   **successfully connected to our database from our application server!**

### Step 6: Documentation

1. **Architecture documentation:**

    #### Architecture Overview:

    Our deployment of 3tier Web Application architecture utilizes the following components:

    - **Frontend Service:** Web/Presentation Tier: Houses the user-facing elements of the application, such as web servers and the interface/frontend.
    - **Backend Service:** Application Tier: Houses the backend and application source code needed to process data and run functions.
    - **Database service:** Data Tier: Houses and manages the application data. Often where the databases are stored.
    - **AWS Services:** Used services like EC2,VPC,RDS,ELB, and more.
   
     #### Infrastructure Setup:

    - **AWS Environment:** Configured with VPC, subnets, security groups.


## Additional Notes

- This deployment process is designed for educational purposes and can be modified for production environments.
- Refer to individual sections for detailed instructions on each step.

