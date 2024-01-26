
# Web App Deployment on AWS VPC

This project demonstrates the creation of a Virtual Private Cloud (VPC) on AWS for deploying a web application in a production environment. The AWS Management Console is utilized for all tasks in this project. 






## VPC Workflow

As depicted in the architecture diagram below, the VPC comprises public and private subnets in two Availability Zones.
 
![vpc-example-private-subnets](https://github.com/harshilp156/Two-tier-web-app-deployment-on-VPC/assets/67538347/764a426d-cca0-4c3c-9e24-508604257c72)


Public subnets include a NAT gateway and an Application Load Balancer node each.

Servers operate in private subnets, automatically launched and terminated by an Auto Scaling group.

Servers receive traffic from the Application Load Balancer.

Servers can connect to the internet via the NAT gateway.

Servers can connect to Amazon S3 using a Gateway VPC Endpoint.

High Availability : To ensure availability, the application is deployed in two Availability Zones (us-east-1, us-east-2).

Bastion hosts or Jump server : As the application is deployed in private subnets, direct SSH access is not possible. Bastion hosts are deployed in each public subnet to securely log into private subnets.

NAT gateway : If applications in private subnets need internet access, NAT gateway provides secure internet usage without exposing private IP addresses. It acts as a mask for the application over its private IP address.

Auto scaling group : To ensure reliability, Auto Scaling groups are used in each private subnet. These groups can dynamically scale up or down based on traffic, ensuring optimal resource utilization. 

Load balancer : The load balancer evenly distributes traffic among multiple servers, continuously monitoring server health and adjusting traffic distribution accordingly.



## Step: 1 Create A VPC

Here I have started with creating VPC.

Select VPC and more, the best thing is aws will automatically create public subnets and private subnet in two availability zones.

I live in Ontario, that is the reason I have selected us-east-1 as my region.

Along with subnets the Route Tables are also created by AWS.

Route tables: Route table is the one which is responsible to define how to route traffic within the subnet. Both public subnets are attached to the route table, which has the destination to the internet gateway so that the traffic flows into the public subnets.

Also, both the private subnets are attached to each route tables, and these route tables have destination to VPC endpoint so that these private subnets can connect to Amazon S3.

Here my IP Address Would be: 10.0.0.0/16. 
 
It means 65,536IPS are allocated towards my VPC.
In simple words I can allocate this IP address 10.0.0.0/16 to 65,535 instances.

I have selected IPV4 CIDR block for this project.

Number of availability zones: 2

Number of public subnets: 2

Number of private subnets: 2

NAT gateways: 1 per availability zones


![App Screenshot](screenshots/Screenshot (461).png?raw=true "ss")






## Step: 2 Create Auto Scaling Group

Here I have created Auto scaling groups to launch and terminated the Ec2 instances (servers) as per the needs automatically.

In aws auto scaling groups cannot be created directly, first we have to create launch template which can be used across multiple auto scaling groups.

I have used Ubuntu server as my AMI.

My instance type is t2.micro which is free tier eligible.

Key pair to SSH in my instance.

Created new security groups with below inbound security group rules.

1. SSH Source 

Type: Anywhere 

Port range: 22

To securely login to my instance.

2. Custom TCP 

source Type: Anywhere

Port range: 8000

Here I ran my application on python server on port range 8000.

Now created auto scaling group with this launch template and selected the vpc which I have created.

Availability Zone and subnets: selected both private subnets in which the auto scaling group will be deployed.

No load balancer is attached here.

Desired capacity: 2

Minimum Desired capacity: 1 

Maximum Desired capacity: 4

Here I have selected 1 instances as my Minimum capacity and 4 instance as my maximum capacity, the auto scaling group will launch and terminate instances according to the demand.


![Screenshot (466)](https://github.com/harshilp156/Two-tier-web-app-deployment-on-VPC/assets/67538347/8b3e6010-73d6-4159-8d9f-e71d5e077ddd)
![Screenshot (467)](https://github.com/harshilp156/Two-tier-web-app-deployment-on-VPC/assets/67538347/3befa930-9e9a-4d78-8c81-e29f8472485c)
![Screenshot (468)](https://github.com/harshilp156/Two-tier-web-app-deployment-on-VPC/assets/67538347/7ab7c63f-e9d3-42ac-9d06-7896cbb4c4af)
![Screenshot (469)](https://github.com/harshilp156/Two-tier-web-app-deployment-on-VPC/assets/67538347/4dde3e3c-c92d-4d9c-9343-856fa072e2bb)
![Screenshot (470)](https://github.com/harshilp156/Two-tier-web-app-deployment-on-VPC/assets/67538347/84ff30a8-145a-448c-a73d-30dda10a909a)
![Screenshot (471)](https://github.com/harshilp156/Two-tier-web-app-deployment-on-VPC/assets/67538347/b49b6188-9dd3-4fdf-b3fd-bda123e7febb)



## Step: 3 Create Bastion Host / Jump server

Both instances do not have public IP addresses as these are in private subnet.

So, in order to securely log into these instances, I have created bastion hosts in each public subnet.

Host 1: Configuration

AMI: Ubuntu Server

Instance Type: t2.micro

Selected the VPC which I have created.

Subnet: Public us-east-1a 

Auto assign Public IP: Enable

Created security Group. 

Inbound security Group Rules 

Type: SSH

Source Type: Anywhere

Host 2: Configuration

AMI: Ubuntu Server

Instance Type: t2.micro

Selected the VPC which I have created.

Subnet: Public us-east-1b 

Auto assign Public IP: Enable

Created security Group. 

Inbound security Group Rules 

Type: SSH

Source Type: Anywhere

## Step: 4 Install Application in subnets.

Here to securely login and install application in each private subnet, I have used bastion hosts which are deployed in each public subnet and through these bastion hosts or jump server I have logged in to the private subnets and installed my application.

To be able to log into private subnet I need to have a key pair inside my each bastion hosts.




## Copy command

So first I copy my pem file in my bastion host using the command below.


```bash
  scp -i my-key.pem ubuntu@"public IP address":/home/ubuntu
```
Copied my pem file for both bastion hosts.

Now SSH into these bastion hosts and install the aaplication.

## SSH command 

```bash
  ssh -i my-key.pem ubuntu@"public IP address of bastion host"
```

## Now log into private subnet.

```bash
  ssh -i my-key.pem ubuntu@"private IP address of private subnet"
```
Now I have created html file using vim edior.

```bash
  vim index.html
```
## Index.html Code for private subnets in us-east-1a
 - [index1a.html](https://github.com/harshilp156/Two-tier-web-app-deployment-on-VPC/blob/main/index1a.html)

## Index.html Code for private subnets in us-east-1b
 - [index1b.html](https://github.com/harshilp156/Two-tier-web-app-deployment-on-VPC/blob/main/index1b.html)


## Now run the html file of both private subnets on python server

```bash
  python3 -m http.server 8000
```


## Step: 5 Creating Application Load Balancer


Application Load Balancer should be internet facing (public subnet) with IPv4 IP address type.

Selected the vpc which I have created for this project and selected public subnets of us-east-1a and us-east-1b.

Security Group: My vpc Security Group

Also, we need a target group as well for the load balancer and add that target group with this load balancer.

Add below inbound rule http, port range 80 , traffic in my vpc Security group to access the site.

Type: http

Port range: 80

Source: Anywhere IPv4

As per the below Screenshots the application load balancer equally manages the load and using both availability zones.


![Screenshot (482)](https://github.com/harshilp156/Two-tier-web-app-deployment-on-VPC/assets/67538347/12d6f879-49a2-4478-803a-dfafda4d37df)

![Screenshot (483)](https://github.com/harshilp156/Two-tier-web-app-deployment-on-VPC/assets/67538347/e6a44be9-0117-41e6-bcbf-c0ba9ee7e4a8)

![Screenshot (486)](https://github.com/harshilp156/Two-tier-web-app-deployment-on-VPC/assets/67538347/1cc4326c-15fe-46f7-b44a-f060d75de43a)


## Tech Stack

**AWS** : VPC , Ec2 , S3, Load Balancer , Auto Scaling Group , IAM , Security Groups, NACL

**Linux** 

**HTML** 

**CSS** 

**VIM Editor** 
