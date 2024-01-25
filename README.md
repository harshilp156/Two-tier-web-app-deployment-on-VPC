
# 2-Tier Web app deployment on vpc 

This project demonstated how to create VPC (virtual private cloud) that you can use for server to deploy two tier web application in production environment.In this project I have used aws management console to complete all tasks. 






## VPC work flow

As per the below architecture diagram,the vpc has public and private subnet in two Avaibility zones.
 
![App Screenshot](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/vpc-example-private-subnets.png)


Each public subnet has NAT gateway and an Application load balancer node.

The server run in private subnet and are launched and terminated automaticall by Auto scaling group.

The server receive trafic from the Application load balancer.

The server can connect to internet using the NAT gateway.

The servers can connect to Amazon S3 by using a gateway VPC endpoint.

High Avaibility : To insure the Avaibility we have deployed application in two Avaibility zones(us-east-1,us-east-2).

Bastion hosts or Jump server : The application sare deployed in private subnets ,so we can not directly ssh into these servers. To securely log into private subnets I have used bastion hosts ,which are deployed in the each public subnets.

NAT gateway : If the application (inside private subnet) wants to access the internet for any reasons (e.g, to download any packages) then it can you use NAT gateway to securely use the internet (without exposing private ip address).
It acts as a mask for application over it's private ip address.

Auto scaling group : To insure the reliability I have used Auto scaling groups(in each private subnet).If the trafic rises for any occasion then the auto scaling group can immediately take the decision and automatically scale up the server to meet the demand and also scale down if the trafic is low.   

Load balancer : The load balancer equally balance the load between multiple server.It continuously monitors the health of servers and destribute the trafic accordingly.



## Step : 1 Create A VPC

Here I have started with creating VPC.

Select VPC and more, the best thing is aws will atutomatically create public subnets and private subnet in two avaibility zones.

I live in ontario that is the reason I have selected us-east-1 as my region.

Along with subnets the Route Tables are also created by AWS.

Route tables : Route table is the one which is responsible to define how to route trafic with in the subnet.Both public subnets are attached to the route table ,which has the destination to the internet gateway so that the trafic flows into the public subnets.

Also both the private subnets are attached to their each route tables , and these route tables have destination to VPC endpoint so that these private subnets can connect to Amazon S3.

Here my IP Address Would be : 10.0.0.0/16 
 
It means 65,536IPS are allocted towads my VPC.
In simple words I can allocate this IP address 10.0.0.0/16 to 65,535 instances.

I have selected IPV4 CIDR block for this priject.

Number of avaibility zones : 2

Number of public subnets : 2

Number of private subnets : 2

NAT gateways : 1 per avaibility zones






## Step : 2 Create Auto Scaling Group

Here I have created Auto scaling groups to launch and terminated the Ec2 instances (servers) as per the needs automatically.

In aws auto scaling groups can not be created directly, first we have to create launch template which can be used across multiple auto scaling groups.

I have used Ubuntu server as my AMI.

My instance type is t2.micro which is free tier eligible.

Key pair to SSH into my instance.

Created new security groups wihh below inbound security group rules.

1 . SSH Source 

Type : Anywhere 

Port range : 22

To securely loginto my instance.

2 . Custome TCP 

source Type : Anywhere

Port range : 8000

Here I ran my application on python server on port range 8000.

Now created auto scaling group with this launch template and selected the vpc which I have created.

Availability Zone and subnets : selected both private subnets in which the auto scaling group will be deployed.

No load balancer is attahced here.

Desired capacity : 2

Minimum Desired capacity : 1 

Maximum Desired capacity : 4

Here I have selected 1 instances as my Minimum capacity and 4 instance as my maximum capacity,the auto scaling group will launch and terminate instances according to the demand.



## Step : 3 Create Bastion Host / Jump server

Both the instance does not have public IP address as these are in private subnet.

So in order to securely log into these instances I have created bastion hosts in each public subnet.

Host 1: Configuration

AMI : Ubuntu Server

Instance Type : t2.micro

Selected the VPC which I have created.

Subnet : Public us-east-1a 

Auto assign Public IP : Enable

Created security Group 

Inbound security Group Rules 

Type : SSH

Source Type : Anywhere

Host 2: Configuration

AMI : Ubuntu Server

Instance Type : t2.micro

Selected the VPC which I have created.

Subnet : Public us-east-1b 

Auto assign Public IP : Enable

Created security Group 

Inbound security Group Rules 

Type : SSH

Source Type : Anywhere

## Step : 4 Install Application in subnets.

Here to securely login and install application in each private subnet ,I have used bastion hosts which are deployed in each public subnets and through these bastion hosts or jump server I have logged in to the private subnets and installed my application.

To be able to log into private subnet I need to have a key pair inside my each bastion host.




## Copy command

So first I copy my pem file in my bastion host using below command.


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
 - [index.html]()

## Index.html Code for private subnets in us-east-1b
 - [index.html]()


## Now run the html file of both private subnets on python server

```bash
  python3 -m http.server 8000
```


## Step : 5 Creating Application Load Balancer


Application Load Balancer should be internet facing (public subnet) with IPv4 IP address type.

Selected the vpc which I have created for this project and also selected public subnets of us-east-1a and us-east-1b.

Security Group : Myvpc Security Group

Also we need a target group as well for the load balancer and add that target group with this load balancer.

Add below inbound rule http , port range 80 , trafic in myvpc Security group to access the site.

Type : http

Port range : 80

Source : Anywhere IPv4

As per the below Screenshots the application load balancer is equally manages the load and using both availability zones.


## Tech Stack

**AWS** : VPC , Ec2 , S3, Load Balancer , Auto Scaling Group , IAM , Security Groups, NACL

**Linux** 

**HTML** 

**CSS** 

**VIM Editor** 