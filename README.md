# aws-site-deployment
_This repository will walk you through the steps of setting up a cloud enviornment using the tools provided by AWS._ 

---
</br>

# Assigment 1: `Cloud Web Hosting`

## Cloud Static Site Setup
In this setup we are going to be going over utililzing the <b>AWS S3 buckets</b> to deploy the files of our site to our EC2 server, and the AWS <b>CloudFront</b> tools to provide us a <b>Content Delivery Network (CDN)</b> for delivering content to our end users.

## Configuring S3 Buckets
We create a aws bucket with a unique name without with all of the objects within being set to private. We upload the files that are contained in this repo as our Objects that we would like to use for our static site. This include pages, images, videos any other files that may help run and maintain our site. 

## Connect our Cloudfront to S3 bucket
Firstly we must create the cloud front bucket by choosing the S3 bucket as our origin domain. 
Next is setting the Origin Access Identity (OAI). This was set to link with the S3 access identity to link the allow cloudfront bucket access. Once this stage deployed we now had a working static website for people around the world to see. 

## EC2 
Providing access to S3 buckets can allow us to maintain and support our site with backups incase of system failures. This keeps up with fast recoverys and continuing site reliability. In our case we can also deploy files to our S3 bucket through our instance EC2. 

We first need to create an instance. Our EC2 instance consits of a Amazon Linux 2 AMI (HVM) - Kernel 5.10, SSD Volume Type hosted by apache. To setup the instance we created an instance type of t2.micro and configured our instance by utlizing this script to install and start an appache hosted server. 

```
#!/bin/bash
# Use this for your user data (script from top to bottom)
# install httpd (Linux 2 version)
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello World from $(hostname -f)</h1>" > /var/www/html/index.html
```

We added some basic tags to associate with our instance. After we add a Security group rule to allow HTTP access to our site. We also created a pem key so that we can access the virtual machine given the key information. 

## Setup EC2 with S3 bucket

At this stage we made a IAM Group that allows our EC2 to read and write to the S3 buckets we created. The permissions policies for this user role should include `AmazonS3FullAccess` as it will allow us full control of the S3 bucket through our instance. We now have the ability to deploy our site from EC2 instance -> S3 Bucket -> CloudFront (CDN)

</br>

---
--- 
</br>

# Assignment 2: `CloudFormation`

This assignment goes through the steps on creating a CloudFormation script and the files used to execute them in order to produce a robust Cloud Network to host a website with built in firewalls and private secure shell accessing. 

---

## Servers: `server-security.yml`
This file creates 2 discint Availability Zones that are accessed using a Virtual Private Cloud (VPC) in order to determine the routing of clients to the website. The Infastructure contains 2 NAT Gateways pointed by an Elastic IP address within a public subnet. The Gateways allow access, with the proper permissions, to the private subnets which host the database and web applications. 

Since the resources is the most important peice to the configuration of a Cloud Stack we will focus on those variables, how we set them up, and what they mean.

### ___Scaling Groups___
Scaling groups add or remove servers based off the Scaling policy set

```yml
Resources:
    WebServerSecGroup:      #
    WebAppLaunchConfig:     #
    WebAppGroup:            #
    ...
```

### ___Load Balancers___

Automatically distribute incoming application traffic across multiple servers. This can all be configured under a given VPC. We utilized it within the private subnets

```yml
Resources:
    ...
    LBSecGroup:             #
    WebAppLB:               #
    Listener:               #
    ALBListenerRule:        #
    WebAppTargetGroup:      #
```

---

</br>

## Network: `network.yml`

### ___VPC___
A Virtual Private Cloud is the networking layer for Amazon EC2 dedicated to your AWS account. The user is able to configure the networking enviornment in any way they see fit such as; network placement, connectivity, and security. 

### ___Internet Gateway___
A gateway that is attached to a VPC and allows the communication between resources and the internet.

### ___Public Subnet___
A Public Subnet uses a specified ip address to route to the internet from it's VPC. It can send information to the internet using a gateway to send and recieve information. 

### ___Private Subnet___
A Private Subnet uses the specified ip addresses to communicate within it's own VPC to send and share information within the server. These IP Addresses cannot be accessed from the internet. 

### ___Elastic IP Addresses (EIP)___
Elastic IP is used for large scale sites, when there can be multiple zones in which a site is deployed upon. An EIP will allow us to mask the failures of a zone by remapping the public ip address to another public IP Address.  

### ____Network Address Translation Gateways (NAT)____
A NAT will allow private networks access to the internet. For example in this configuation the NAT address will allow us to access our VPC. In other instances we can also directly access a private subnet. 

### __Route Table___ 
A set of rules called routes, that are used to determine where network traffic is directed. 

This file goes over each key value and it's importance to this cloud formation stack that was cerated. 
``` yml
Resources:
    VPC:                                    # Deploy VPC with CIDR option
    InternetGateway:                        # Connect resources within the VPC
    InternetGatewayAttachment:              # Attach the Internet Gateway to the VPC
    PublicSubnet1:                          # IP connecting to internet from given AVZ 1
    PublicSubnet2:                          # IP connecting to internet from given AVZ 2
    PrivateSubnet1:                         # IP connecting to VPC can only be used in AVZ 1
    PrivateSubnet2:                         # IP connecting to VPC can only be used in AVZ 1
    NatGateway1EIP:                         # Create a Public Elastic IP Address 1
    NatGateway2EIP:                         # Create a Public Elastic IP Address 2
    NatGateway1:                            # Connect NAT EIP 1 with PublicSubnet 1
    NatGateway2:                            # Connect NAT EIP 1 with PublicSubnet 2
    PublicRouteTable:                       # Create an EC2 RouteTable
    DefaultPublicRoute:                     # Connect RouteTable to InternetGateway
    PublicSubnet1RouteTableAssociation:     # Attach PublicSubnet 1 to RouteTable
    PublicSubnet2RouteTableAssociation:     # Attach PublicSubnet 2 to RouteTable
    PrivateRouteTable1:                     # Create private route table in AZ1
    DefaultPrivateRoute1:                   # Attach private route table 1 to NatGateway 1
    PrivateSubnet1RouteTableAssociation:    # Attach private subnet 1 to private route table 1
    PrivateRouteTable2:                     # Create private route table in AZ2
    DefaultPrivateRoute2:                   # Attach private route table 2 to NatGateway 2
    PrivateSubnet2RouteTableAssociation:    # Attach private subnet 2 to private route table 2
```


