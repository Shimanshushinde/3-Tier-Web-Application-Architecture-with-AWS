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
   - Navigate to "NAT Gateways" and create a new gateway called public-NAT-1.
   - Select one of the public subnets, allocate an elastic IP, and create.
4. **Configure private route tables**
   - Select any one of the private route tables, associate this table with all four private subnets.
   - add a new route to NAT gateway.
