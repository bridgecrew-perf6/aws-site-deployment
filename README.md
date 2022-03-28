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
This file creates 2 discint Availability Zones that are accessed using a Virtual Private Connection (VPC) in order to determine the routing of clients to the website. The Infastructure contains 2 NAT Gateways pointed by an Elastic IP address within a public subnet. The Gateways allow access, with the proper permissions, to the private subnets which host the database and web applications. 

Since the resources is the most important peice to the configuration of a Cloud Stack we will focus on those variables, how we set them up, and what they mean.

### ___Scaling Groups___
Scaling groups add or remove servers based off the Scaling policy set

```yml
Resources:
    WebServerSecGroup:
    WebAppLaunchConfig:
    WebAppGroup:
    ...
```

### ___Load Balancers___

Automatically distribute incoming application traffic across multiple servers. This can all be configured under a given VPC. We utilized it within the private subnets

```yml
Resources:
    ...
    LBSecGroup:
    WebAppLB:
    Listener:
    ALBListenerRule:
    WebAppTargetGroup:
```

---

</br>

## Network: `network.yml`


