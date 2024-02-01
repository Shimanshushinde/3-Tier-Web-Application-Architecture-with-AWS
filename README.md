# Building a 3-tier web application architecture with AWS
Deploying a three-tier web application on AWS involves setting up three layers: presentation (frontend), logic tier (backend), and data tier (database).
This guide to deploying such an application on AWS using commonly used services like EC2, RDS,ELB, and more.

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
   - public-rtb to serve as the main table, so select the public-rtb from the "Route tables".
3. **Create a NAT Gateway**
   - Navigate to "NAT Gateways" section and click on "create NAT gateway".
   - Select one of the public subnets, allocate an elastic IP, and create.
4. **Configure private route tables**
   - Select any one of the private route tables, associate this table with all four private subnets.
   - add a new route to NAT gateway and save it.
  
### Step 2: Tier 1: Web tier (Frontend)
1. **Create a web server launch template**
   - In the EC2 console, navigate to "Launch templates" under the "Instances" sidebar menu.
     - AMI: Amazon 2 Linux
     - Instance type: t2.micro (1GB – Free Tier)
     - A new or existing key pair
   - create a new security group with inbound SSH, HTTP, and HTTPS rules.
   - Under ‘Advanced details >[User Data](https://github.com/Shimanshushinde/3-Tier-Web-Application-Architecture-with-AWS/blob/e687d4a2beda1a7585e19ee079ef285c3787610e/User%20Data)

2. **Create an Auto scaling group (ASG)**
   - Navigate to the ASG console from the sidebar menu and create a new group.
   - Select the "3tier-webapp-vpc" along with the two public subnets.

3. **Application load balancer (ALB)**
   - ALB to be "Internet-facing," needs to 'listen’ over HTTP on port 80 and a target group that routes to ec2 instance.
   - add a scaling policy > Target Tracking Policy (Average CPU Utilization at 50%).
   - Group size:
      - Desired capacity: 2
      - Minimum capacity: 2
      - Maximum capacity: 5
   - Review the ASG settings and create the group
   - To see ALB is properly routing traffic, let’s go to its public IP address and past in Browser.

4. **Use SSH Connection To Connect To The Web Server**
   - SSH into our EC2 server, use putty as ssh client.(Remember, this is the ‘Presentation’ layer, where our users will directly interact with our app.)
  
### Step 3: Tier 2: Application tier (Backend)
1. **Create an application server launch template**
   - In the EC2 console, navigate to "Launch templates" under the "Instances" sidebar menu.
      - (Amazon 2 Linux, t2.micro-1GB, same key pair).
   - create a new security group to allow all ICMP–IPv4 from the 3tier-webServer-SG, which allows us to ping the application server from our web server.
   - In the ‘User data’ field under ‘Advanced details
       ```
       #!/bin/bash
       sudo yum install mysql -y
       ```
   - Review and create the template.

2.**Create an Auto Scaling Group (ASG)**
   - Navigate to the ASG console from the sidebar menu and create a new group.
   - Select the "3tier-webapp-vpc" along with the two private subnets.

3. **Application Load Balancer (ALB)**
   - This time, we want the ALB to be "Internal," since we’re routing traffic from our Web Tier, not the Internet.
   - Confirm connectivity from the Web Tier.
     - SSH into the web server EC2
        ```sh
        ssh -i "webServer_key.ppk" ec2-user@ec2-54-209-250-120.compute-1.amazonaws.com
        ```
     - Ping the private IP address of one of the app server EC2s.
         ```
         ping PRIVATE_IPV4_ADDRESS
         ```
4.**Create a Bastion host**
  - BH



