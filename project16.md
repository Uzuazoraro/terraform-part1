## AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM PART 1
   ==================================================================

![set-up](image-1.png)

Prerequisites before you begin writing Terraform code
•	You must have completed Terraform course from the Learning dashboard
•	Create an IAM user, name it terraform (ensure that the user has only programatic access to your AWS account) and grant this user AdministratorAccess permissions.

Steps:
Create AWS Organization > Create User > Set permission > Grant access by clicking aws account, select mgt acct and assign users or groups > click user and select the user > next.......

Create user
============

Click IAM > user and give it a name > click on user group to create a user group > give administrative access and attach the policy > create user > click on the created user name > click security credentials > click create access key.

•	Copy the secret access key and access key ID. Save them in a notepad temporarily.
•	Configure programmatic access from your workstation to connect to AWS using the access keys (type: $ "aws configure" on your terminal) copied above and a Python SDK (boto3). You must have Python 3.6 or higher on your workstation.
If you are on Windows, use gitbash, if you are on a Mac, you can simply open a terminal. Read here to configure the Python SDK properly.
For easier authentication configuration – use AWS CLI with aws configure command.

## Make sure boto3 is installed
## Create this file file: 'test.py' and paste:

import boto3
s3 = boto3.resource('s3')
for bucket in s3.buckets.all():
    print(bucket.name)

Type 'py test.py' and the name of the bucket you created will come up.  

•	Create an S3 bucket (https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html) to store Terraform state file. You can name it something like <yourname>-dev-terraform-bucket (Note: S3 bucket names must be unique unique within a region partition, you can read about S3 bucket naming in this article "https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html").

## VPC | SUBNETS | SECURITY GROUPS
========================================================================

## Create a folder called PBL Create a file in the folder, name it main.tf

Provider and VPC resource section
Set up Terraform CLI as per this instruction.

Add AWS as a provider, and a resource to create a VPC in the main.tf file. Provider block informs Terraform that we intend to build infrastructure within AWS. Resource block will create a VPC. provider "aws" { region = "us-east-2" }

provider "aws" {
  region = "us-east-2"
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = "172.16.0.0/16"
  enable_dns_support             = "true"
  enable_dns_hostnames           = "true"
  enable_classiclink             = "false"
  enable_classiclink_dns_support = "false"
}

Note: You can change the configuration above to create your VPC in other region that is closer to you. The same applies to all configuration snippets that will follow.
•	The next thing we need to do, is to download necessary plugins for Terraform to work. These plugins are used by providers and provisioners. At this stage, we only have provider in our main.tf file. So, Terraform will just download plugin for AWS provider.
•	Lets accomplish this with terraform init command as seen in the below demonstration.
+ Run 'terraform plan'
+ Run 'terraform apply'

Observations:
1.	A new file is created terraform.tfstate This is how Terraform keeps itself up to date with the exact state of the infrastructure. It reads this file to know what already exists, what should be added, or destroyed based on the entire terraform code that is being developed.
2.	If you also observed closely, you would realize that another file gets created during planning and apply. But this file gets deleted immediately. terraform.tfstate.lock.info This is what Terraform uses to track, who is running its code against the infrastructure at any point in time. This is very important for teams working on the same Terraform repository at the same time. The lock prevents a user from executing Terraform configuration against the same infrastructure when another user is doing the same – it allows to avoid duplicates and conflicts.

## Provider and VPC resource section
===================================================

## Set up Terraform CLI as per this instruction (https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)

The secrets of writing quality Terraform code
The secret recipe of a successful Terraform projects consists of:
•	Your understanding of your goal (desired AWS infrastructure end state)
•	Your knowledge of the IaC technology used (in this case – Terraform)
•	Your ability to effectively use up to date Terraform documentation here
As you go along completing this project, you will get familiar with Terraform-specific terminology, such as:
•	Attribute
•	Resource
•	Interpolations
•	Argument
•	Providers
•	Provisioners
•	Input Variables
•	Output Variables
•	Module
•	Data Source
•	Local Values
•	Backend
Make sure you understand them and know when to use each of them.

## Best practices
==============================

•	Ensure that every resource is tagged using multiple key-value pairs. You will see this in action as we go along.
•	Try to write reusable code, avoid hard coding values wherever possible. (For learning purpose, we will start by hard coding, but gradually refactor our work to follow best practices).
VPC | SUBNETS | SECURITY GROUPS
VPC | Subnets | Security Groups

## Subnets resource section
===============================

According to our architectural design, we require 6 subnets:
•	2 public
•	2 private for webservers
•	2 private for data layer
Let us create the first 2 public subnets.
Add below configuration to the main.tf file:

## Create public subnets1
    resource "aws_subnet" "public1" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.0.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "us-east-2a"
}

# Create public subnet2
    resource "aws_subnet" "public2" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.1.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "us-east-2b"
}
