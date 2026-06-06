# Day-61: Introduction to Terraform and Your First AWS Infrastructure

## Objective
Today I started my Terraform journey and learned how to create cloud infrastructure using code.
Instead of manually creating resources from the AWS Console, I used Terraform to create and manage AWS resources.

By the end of this task I successfully:

* Installed Terraform
* Configured AWS CLI
* Created an S3 Bucket using Terraform
* Created an EC2 Instance using Terraform
* Understood Terraform State File
* Updated Infrastructure using Terraform
* Destroyed Infrastructure using Terraform
  
# Task-1: Understand Infrastructure as Code (IaC)

## What is Infrastructure as Code (IaC)?

Infrastructure as Code (IaC) is the practice of creating and managing infrastructure using code instead of manually creating resources from cloud consoles.

Using IaC we can define servers, networks, storage, databases and cloud resources in files and deploy them whenever required.

### Real Life Example

Without IaC:

```text
Login AWS Console
Create EC2
Create S3
Create Security Group
Create VPC
```

Every time manually.

With IaC:

```text
Write Terraform Code
Run terraform apply
Infrastructure Ready
```

### Simple Defination 

Infrastructure as Code means managing infrastructure through code instead of manual clicks.

---

## Why IaC Matters in DevOps?

IaC is important because it provides:

### Automation

Infrastructure can be created automatically.

### Consistency

Same infrastructure can be created multiple times.

### Repeatability

Development, Testing and Production environments remain identical.

### Version Control

Infrastructure code can be stored in Git.

### Reduced Human Errors

Manual mistakes are minimized.

### Faster Deployment

Resources can be created in minutes.

---

## Problems with Manual AWS Console Creation

### Manual Process

Creating resources one by one takes time.

### Human Errors

Wrong configuration can be applied accidentally.

### No Version History

No tracking of changes.

### Difficult Scaling

Large environments become difficult to manage.

### Difficult Recovery

Infrastructure must be recreated manually.

---

# Terraform vs CloudFormation vs Ansible vs Pulumi

## Terraform

Terraform is an Infrastructure as Code tool developed by HashiCorp.

### Features

* Multi Cloud Support
* Declarative
* Open Source
* Large Community

### Supported Platforms

* AWS
* Azure
* GCP
* Kubernetes
* VMware

---

## AWS CloudFormation

CloudFormation is AWS's native Infrastructure as Code service.

### Features

* Works only with AWS
* Deep AWS Integration
* AWS Managed

### Limitation

Cannot easily manage Azure or GCP resources.

---

## Ansible

Ansible is mainly a Configuration Management Tool.

### Terraform Creates

```text
EC2
S3
VPC
RDS
```

### Ansible Configures

```text
Nginx
Apache
Docker
MySQL
Application Deployment
```

### Example

Terraform:

```text
Create EC2 Server
```

Ansible:

```text
Install Nginx on EC2
```

### Interview Answer

Terraform provisions infrastructure while Ansible configures servers and applications.

---

## Pulumi

Pulumi is also an Infrastructure as Code tool.

Difference:

Terraform uses:

```text
HCL
(HashiCorp Configuration Language)
```

Pulumi uses:

```text
Python
JavaScript
TypeScript
Go
C#
```

programming languages.

---

# What Does Declarative Mean?

Terraform is Declarative.

This means we tell Terraform:

```text
What we want
```

not

```text
How to create it
```

Example:

```terraform
resource "aws_instance" "web" {
  instance_type = "t2.micro"
}
```

Terraform automatically decides:

* API Calls
* Creation Order
* Resource Management

### Simple Defination 

In Declarative style we define the desired result and Terraform figures out how to achieve it.

---

# What Does Cloud Agnostic Mean?

Terraform works across multiple cloud providers.

Example:

Same Terraform knowledge can be used for:

* AWS
* Azure
* GCP

### Simple Defination 

Cloud agnostic means Terraform is not tied to a single cloud provider.

---

# Terraform Architecture

```text
Terraform Code
       |
       v
Terraform Core
       |
       v
Provider
       |
       v
Cloud API
       |
       v
Resources
```

---

# Terraform 5 Pillars

## 1. Provider

Provider is a plugin that allows Terraform to communicate with a platform.

Example:

```terraform
provider "aws" {
  region = "us-east-1"
}
```

### Simple Defination 

Provider acts as a translator between Terraform and AWS.

---

## 2. Resource

A resource represents an infrastructure component.

Examples:

```text
EC2
S3
VPC
RDS
```

Example:

```terraform
resource "aws_s3_bucket" "demo_bucket" {
  bucket = "terraform-demo"
}
```

### Simple Defination 

A resource is something Terraform creates or manages.

---

## 3. State

Terraform stores information about resources inside:

```text
terraform.tfstate
```

### Simple Defination 

State file is Terraform's database.

---

## 4. Variable

Variables help avoid hardcoded values.

Example:

```terraform
variable "region" {
  default = "us-east-1"
}
```

### Simple Defination 
Variables make code reusable and flexible.

## 5. Output

Outputs display useful information after deployment.

Example:

```terraform
output "bucket_name" {
  value = aws_s3_bucket.demo_bucket.id
}
``
### Simple Defination 
Outputs show important values after infrastructure creation.

# Task-2: Install Terraform and Configure AWS

## Objective

Before creating infrastructure, Terraform and AWS CLI must be installed and configured.

Terraform will create resources, while AWS CLI provides authentication and access to AWS services.

---

# Terraform Installation

## Windows Installation

### Step-1

Download Terraform ZIP file from HashiCorp website.

### Step-2

Extract the ZIP file.

Example:

```text
C:\Terraform
```

### Step-3

Add Terraform folder path to Environment Variables.

```text
System Properties
→ Advanced System Settings
→ Environment Variables
→ Path
→ Add Terraform Folder Path
```

### Step-4

Open Git Bash or PowerShell.

Verify:

```bash
terraform -version
```

Output:

```text
Terraform v1.15.5
```

---

# AWS CLI Installation

AWS CLI is used to communicate with AWS services from terminal.

### Verify Installation

```bash
aws --version
```

Output:

```text
aws-cli/2.x.x
```

---

# Configure AWS CLI

Run:

```bash
aws configure
```

AWS asks:

```text
AWS Access Key ID
AWS Secret Access Key
Default Region
Output Format
```

Example:

```text
AWS Access Key ID     = ****************
AWS Secret Access Key = ****************
Default Region        = us-east-1
Output Format         = json
```

---

# Verify AWS Authentication

Run:

```bash
aws sts get-caller-identity
```

Output:

```json
{
  "UserId": "XXXXXXXX",
  "Account": "XXXXXXXX",
  "Arn": "arn:aws:iam::XXXXXXXX:user/admin"
}
```

This confirms AWS authentication is working successfully.

---

# AWS Authentication Flow

Many beginners think Terraform directly connects to AWS.

Actual flow:

```text
Terraform
    |
AWS Provider
    |
AWS Credentials
    |
AWS API
    |
AWS Resources
```

---

# Access Key and Secret Key

AWS uses two important values for authentication.

## Access Key

Works like a username.

Example:

```text
AKIAxxxxxxxxxxxx
```

---

## Secret Key

Works like a password.

Example:

```text
xxxxxxxxxxxxxxxxxxxxxxxx
```

Never share Secret Keys publicly.

---

# Where AWS Credentials Are Stored?

Windows:

```text
C:\Users\<username>\.aws\
```

Files:

```text
credentials
config
```

---

# credentials File

Example:

```ini
[default]
aws_access_key_id = AKIAxxxxxxxxxxxx
aws_secret_access_key = xxxxxxxxxxxxx
```

---

# config File

Example:

```ini
[default]
region = us-east-1
output = json
```

---

# How Terraform Finds Credentials

When Terraform starts, AWS Provider searches for credentials.

Common order:

```text
Environment Variables
↓
AWS Credentials File
↓
AWS Config File
↓
IAM Role
```

If valid credentials are found, Terraform can connect to AWS.

---

# Provider Deep Dive

Provider acts as a translator between Terraform and Cloud Platforms.

Example:

```terraform
provider "aws" {
  region = "us-east-1"
}
```

Terraform itself cannot create AWS resources.

AWS Provider converts Terraform code into AWS API requests.

---

# Terraform Project Structure

Project folder:

```text
terraform-basics/
│
├── main.tf
├── s3.tf
├── ec2.tf
├── terraform.tfstate
├── terraform.tfstate.backup
├── .terraform/
└── .terraform.lock.hcl
```

---

# Why Separate Files?

Bad Example:

```text
main.tf
1000+ lines
```

Difficult to maintain.

---

Better Example:

```text
main.tf       → Provider
s3.tf         → S3 Resources
ec2.tf        → EC2 Resources
variables.tf  → Variables
outputs.tf    → Outputs
```

This structure is easier to manage.

---

# main.tf

Contains Terraform settings and provider configuration.

Example:

```terraform
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}
```

---

# Terraform Block

Used to configure Terraform itself.

Example:

```terraform
terraform {
}
```

This block controls provider requirements and Terraform settings.

---

# required_providers

Used to tell Terraform which provider is needed.

Example:

```terraform
required_providers {
  aws = {
    source = "hashicorp/aws"
  }
}
```

Terraform downloads the provider automatically.

---

# source

Example:

```terraform
source = "hashicorp/aws"
```

Meaning:

Download AWS provider from Terraform Registry.

---

# version

Example:

```terraform
version = "~> 6.0"
```

Meaning:

Use compatible versions within major version 6.

Possible versions:

```text
6.0
6.1
6.2
6.5
```

Not:

```text
7.0
```

---

# Why Provider Version Is Important?

Without version locking:

```text
Today → Provider 6.x
Tomorrow → Provider 7.x
```

Code may break unexpectedly.

Version locking ensures consistent deployments.

---

# Terraform Registry

Terraform providers are downloaded from:

```text
https://registry.terraform.io
```

Examples:

```text
AWS Provider
Azure Provider
Google Provider
Kubernetes Provider
Docker Provider
```

---

# Important Commands Used

## terraform -version

Shows installed Terraform version.

```bash
terraform -version
```

---

## aws --version

Shows installed AWS CLI version.

```bash
aws --version
```

---

## aws sts get-caller-identity

Shows current authenticated AWS user.

```bash
aws sts get-caller-identity
```

---

# Important Points Learned

* Terraform creates infrastructure using code.
* AWS CLI provides authentication.
* Provider connects Terraform with AWS.
* Credentials are stored locally.
* Terraform downloads providers from Terraform Registry.
* Version locking prevents unexpected issues.
* Infrastructure code should be organized into separate files.

---

# Common Mistakes

## Mistake-1

Hardcoding Access Keys inside Terraform code.

Bad:

```terraform
provider "aws" {
  access_key = "xxxx"
  secret_key = "xxxx"
}
```

Use AWS CLI configuration instead.

---

## Mistake-2

Not adding Terraform to PATH.

Result:

```bash
terraform: command not found
```

---

## Mistake-3

Forgetting AWS authentication.

Result:

```bash
Unable to locate credentials
```

---

## Mistake-4

Using wrong AWS region.

Resources may be created in an unexpected location.

---

# Interview Questions

### What is AWS CLI?

AWS CLI is a command-line tool used to interact with AWS services.

---

### What is a Provider in Terraform?

A provider is a plugin that allows Terraform to communicate with cloud platforms.

---

### Why is AWS Provider required?

Without a provider, Terraform cannot communicate with AWS.

---

### What is Terraform Registry?

Terraform Registry is a repository where providers and modules are stored.

---

### Why should provider versions be specified?

Provider versions ensure stable and predictable deployments.

---

### Where are AWS credentials stored?

AWS credentials are usually stored inside:

```text
~/.aws/credentials
```

---

### What command verifies AWS authentication?

```bash
aws sts get-caller-identity
```

---

### What happens during terraform init?

Terraform downloads providers, creates the .terraform directory, and initializes the project.

# Task-3: Create Your First Terraform Configuration (S3 Bucket)

## Objective

In this task, we create our first AWS resource using Terraform.

We will create:

```text
1 S3 Bucket
```

using only Terraform code.

This is the first real Infrastructure as Code deployment.

---

# What is Amazon S3?

Amazon S3 stands for Simple Storage Service.

It is an object storage service provided by AWS.

S3 is commonly used for:

```text
Images
Videos
Backups
Application Files
Logs
Static Websites
Documents
```

---

# Real Example

Think of S3 Bucket as a folder.

Example:

```text
Laptop
│
└── Photos Folder
```

AWS:

```text
S3 Bucket
│
├── photos.jpg
├── backup.zip
└── logs.txt
```

---

# S3 Bucket Naming Rules

Bucket names must be:

```text
Globally Unique
Lowercase Only
No Spaces
No Uppercase Letters
```

Good Example:

```text
terraform-praveen-307946647949-demo
```

Bad Example:

```text
MyBucket
```

Bad Example:

```text
Terraform Bucket
```

---

# Create First Terraform Configuration

File:

```text
s3.tf
```

Code:

```terraform
resource "aws_s3_bucket" "demo_bucket" {
  bucket = "terraform-praveen-307946647949-demo"
}
```

---

# Code Explanation

## resource

```terraform
resource
```

Used to create infrastructure resources.

Terraform understands:

```text
Something must be created.
```

---

## aws_s3_bucket

```terraform
aws_s3_bucket
```

Resource Type.

Meaning:

```text
Create an AWS S3 Bucket.
```

---

## demo_bucket

```terraform
demo_bucket
```

Terraform Logical Name.

Important:

This is NOT the AWS bucket name.

Terraform uses it internally.

Example:

```terraform
aws_s3_bucket.demo_bucket
```

---

## bucket

```terraform
bucket = "terraform-praveen-307946647949-demo"
```

Actual AWS bucket name.

This is what appears in AWS Console.

---

# Terraform Workflow

Every Terraform project follows this workflow:

```text
Write Code
    ↓
terraform fmt
    ↓
terraform validate
    ↓
terraform init
    ↓
terraform plan
    ↓
terraform apply
```

---

# terraform fmt

Command:

```bash
terraform fmt
```

Purpose:

Automatically formats Terraform code.

Before:

```terraform
resource "aws_s3_bucket" "demo_bucket"{
bucket="demo"
}
```

After:

```terraform
resource "aws_s3_bucket" "demo_bucket" {
  bucket = "demo"
}
```

---

# Why Use terraform fmt?

Benefits:

```text
Readable Code
Consistent Formatting
Professional Style
```

---

# terraform validate

Command:

```bash
terraform validate
```

Purpose:

Checks Terraform syntax.

Example:

```terraform
resource "aws_s3_bucket" "demo_bucket" {
```

Missing closing bracket:

```terraform
}
```

Validation will fail.

---

# What terraform validate Does

Checks:

```text
Syntax
Terraform Structure
Required Blocks
```

Does NOT:

```text
Create Resources
Modify AWS
Delete Resources
```

---

# terraform init

Command:

```bash
terraform init
```

Purpose:

Initializes Terraform project.

---

# What Happens During terraform init?

Terraform performs:

```text
Provider Download
Plugin Installation
Working Directory Initialization
Lock File Creation
```

---

# Provider Download Process

Terraform reads:

```terraform
required_providers {
  aws = {
    source = "hashicorp/aws"
  }
}
```

Then downloads AWS Provider.

Flow:

```text
Terraform
     ↓
Terraform Registry
     ↓
AWS Provider
     ↓
Local Machine
```

---

# .terraform Directory

Created automatically after:

```bash
terraform init
```

Example:

```text
terraform-basics/
│
├── .terraform/
```

Contains:

```text
Downloaded Providers
Metadata
Plugin Files
```

---

# .terraform.lock.hcl

Also created after:

```bash
terraform init
```

Purpose:

Locks provider versions.

Example:

```text
AWS Provider Version
Checksums
Signatures
```

---

# Why Lock File Matters?

Without lock file:

```text
Today → AWS Provider 6.x
Tomorrow → AWS Provider 7.x
```

Code behavior may change.

Lock file ensures consistent versions.

---

# terraform plan

Command:

```bash
terraform plan
```

Purpose:

Shows what Terraform will do before making changes.

---

# Plan Output

Output:

```text
+ create
```

Meaning:

Terraform will create a resource.

---

Example:

```text
# aws_s3_bucket.demo_bucket will be created
```

Meaning:

Terraform plans to create an S3 Bucket.

---

# Plan Summary

Output:

```text
Plan: 1 to add, 0 to change, 0 to destroy
```

Meaning:

```text
1 Resource Create
0 Resource Update
0 Resource Delete
```

---

# Important Terraform Symbols

## +

```text
Create
```

Example:

```text
+ aws_s3_bucket.demo_bucket
```

---

## ~

```text
Update
```

Example:

```text
~ tags
```

---

## -

```text
Destroy
```

Example:

```text
- aws_instance.web
```

---

## -/+

```text
Destroy and Recreate
```

Used when Terraform cannot update a resource in-place.

---

# known after apply

Plan Output:

```text
arn = (known after apply)
```

Meaning:

Terraform will know the value only after resource creation.

Example:

```text
ARN
ID
Public IP
```

are generated by AWS.

---

# terraform apply

Command:

```bash
terraform apply
```

Purpose:

Creates infrastructure in AWS.

---

# Confirmation

Terraform asks:

```text
Do you want to perform these actions?
```

Type:

```text
yes
```

---

# Successful Output

Example:

```text
aws_s3_bucket.demo_bucket: Creation complete

Apply complete!
Resources: 1 added, 0 changed, 0 destroyed.
```

Meaning:

Bucket created successfully.

---

# Verify S3 Bucket

Using AWS CLI:

```bash
aws s3 ls
```

Output:

```text
terraform-praveen-307946647949-demo
```

---

# Verify Using AWS Console

Open:

```text
AWS Console
→ S3
→ Buckets
```

Bucket should be visible.

---

# Terraform Resource Address

Terraform identifies resources using:

```text
resource_type.resource_name
```

Example:

```text
aws_s3_bucket.demo_bucket
```

Breakdown:

```text
aws_s3_bucket → Resource Type
demo_bucket   → Logical Name
```

---

# Important Learning

Terraform does not track resources using file names.

Terraform tracks resources using:

```text
Resource Address
+
State File
```

Example:

```text
aws_s3_bucket.demo_bucket
```

Even if resource moves from:

```text
main.tf
```

to

```text
s3.tf
```

Terraform still recognizes it.

---

# Common Mistakes

## Mistake-1

Running validate before init.

Result:

```text
Missing required provider
```

Fix:

```bash
terraform init
```

---

## Mistake-2

Using duplicate bucket name.

Result:

```text
BucketAlreadyExists
```

Fix:

Use unique bucket name.

---

## Mistake-3

Using uppercase bucket names.

Result:

```text
Invalid Bucket Name
```

---

## Mistake-4

Forgetting terraform fmt.

Result:

Messy code formatting.

---

# Important Points Learned

* S3 Bucket created successfully using Terraform.
* Terraform Provider downloaded successfully.
* terraform init initializes the project.
* terraform validate checks syntax.
* terraform fmt formats code.
* terraform plan shows proposed changes.
* terraform apply creates infrastructure.
* Terraform uses resource addresses to track resources.

---

# Interview Questions

### What is Amazon S3?

Amazon S3 is an object storage service used to store files and data.

---

### What is a Terraform Resource?

A resource is an infrastructure object managed by Terraform.

---

### What does terraform fmt do?

It automatically formats Terraform files.

---

### What does terraform validate do?

It checks Terraform configuration syntax and structure.

---

### What does terraform init do?

It initializes the project and downloads required providers.

---

### What does terraform plan do?

It shows what Terraform will create, update, or destroy.

---

### What does terraform apply do?

It creates or modifies infrastructure resources.

---

### What does the + symbol mean?

Terraform will create a new resource.

---

### What does known after apply mean?

The value becomes available only after resource creation.

---

### How does Terraform identify resources?

Terraform identifies resources using resource addresses and state information.

# Task-4: Add an EC2 Instance Using Terraform

## Objective

In this task, we create our first EC2 instance using Terraform.

Previously we created:

```text
S3 Bucket
```

Now we will create:

```text
EC2 Instance
```

using Terraform code.

---

# What is EC2?

EC2 stands for:

```text
Elastic Compute Cloud
```

EC2 is a virtual server provided by AWS.

It allows us to run applications, websites, databases, and services in the cloud.

---

# Real Example

Think of EC2 as a cloud computer.

Physical Laptop:

```text
CPU
RAM
Disk
Operating System
```

EC2 Instance:

```text
vCPU
Memory
Storage
Operating System
```

Both work similarly.

Difference:

```text
Laptop → Physical Machine
EC2 → Virtual Machine
```

---

# Common Uses of EC2

Examples:

```text
Web Servers
Application Servers
Jenkins Server
GitLab Server
Docker Host
Kubernetes Node
Monitoring Server
```

---

# EC2 Resource Configuration

File:

```text
ec2.tf
```

Code:

```terraform
resource "aws_instance" "web" {
  ami           = "ami-091138d0f0d41ff90"
  instance_type = "t2.micro"

  tags = {
    Name = "TerraWeek-Day1"
  }
}
```

---

# Code Explanation

## Resource Block

```terraform
resource "aws_instance" "web"
```

Meaning:

Terraform will create an EC2 instance.

---

## aws_instance

Resource Type.

Meaning:

```text
Create AWS EC2 Instance
```

---

## web

Terraform Logical Name.

Used internally by Terraform.

Example:

```terraform
aws_instance.web
```

---

# AMI

Code:

```terraform
ami = "ami-091138d0f0d41ff90"
```

AMI means:

```text
Amazon Machine Image
```

---

# What is an AMI?

AMI is a template used to launch EC2 instances.

Think of it like:

```text
Windows ISO
Ubuntu ISO
Linux Installation Image
```

AWS version of those templates is called AMI.

---

# Examples of AMIs

```text
Amazon Linux
Ubuntu
Red Hat
Windows Server
Debian
```

---

# Why AMI is Required?

Without AMI:

```text
No Operating System
No Server
```

AWS would not know which OS to install.

---

# Easy Understanding

AMI decides:

```text
Which Operating System
Which Preinstalled Software
Which Initial Configuration
```

the EC2 instance will have.

---

# Instance Type

Code:

```terraform
instance_type = "t2.micro"
```

---

# What is Instance Type?

Instance Type defines:

```text
CPU
Memory
Network Capacity
Storage Performance
```

---

# Real Example

Laptop Choices:

```text
4GB RAM
8GB RAM
16GB RAM
```

AWS Choices:

```text
t2.micro
t2.small
t3.medium
m5.large
```

---

# Why t2.micro?

Task requirement:

```text
t2.micro
```

Small machine suitable for learning.

Approximate configuration:

```text
1 vCPU
1 GB RAM
```

---

# Tags

Code:

```terraform
tags = {
  Name = "TerraWeek-Day1"
}
```

---

# What are Tags?

Tags are labels attached to AWS resources.

---

# Real Example

Think of a name sticker on a laptop.

Example:

```text
Owner = Praveen
Environment = Dev
Project = Terraform
```

---

# Why Tags Are Important?

Benefits:

```text
Easy Identification
Cost Tracking
Resource Organization
Automation
```

---

# Example Tags

```terraform
tags = {
  Name        = "WebServer"
  Environment = "Dev"
  Team        = "DevOps"
}
```

---

# Terraform Plan After Adding EC2

Command:

```bash
terraform plan
```

Output:

```text
aws_s3_bucket.demo_bucket: Refreshing state...
```

---

# What Does Refreshing State Mean?

Terraform checks:

```text
State File
```

and compares it with:

```text
Actual AWS Infrastructure
```

---

# State Comparison

Current State:

```text
S3 Bucket Exists
```

Desired State:

```text
S3 Bucket Exists
EC2 Instance Required
```

Difference:

```text
EC2 Missing
```

Result:

```text
Create EC2
```

---

# Most Important Learning

Terraform did NOT create another S3 Bucket.

Reason:

Terraform already knew:

```text
Bucket Exists
```

through:

```text
terraform.tfstate
```

---

# Interview Question

### How does Terraform know the S3 bucket already exists?

Terraform reads the state file and compares it with the current configuration.

---

# Plan Output

Example:

```text
Plan: 1 to add, 0 to change, 0 to destroy.
```

Meaning:

```text
1 Resource Create
0 Resource Update
0 Resource Delete
```

---

# Known After Apply

Plan Output:

```text
public_ip = (known after apply)
```

Example:

```text
id
arn
public_ip
private_ip
public_dns
```

AWS generates these values after instance creation.

---

# terraform apply

Command:

```bash
terraform apply
```

Confirm:

```text
yes
```

---

# Successful Output

Example:

```text
Apply complete!
Resources: 1 added, 0 changed, 0 destroyed.
```

Meaning:

EC2 instance created successfully.

---

# Verify Using AWS Console

Open:

```text
AWS Console
→ EC2
→ Instances
```

Check:

```text
Instance State = Running
Name Tag = TerraWeek-Day1
```

---

# Verify Using terraform show

Command:

```bash
terraform show
```

Important values:

```text
Instance ID
Public IP
Private IP
Availability Zone
Instance State
```

---

# Public IP

Example:

```text
52.x.x.x
```

Used for internet communication.

---

# Private IP

Example:

```text
172.x.x.x
```

Used for internal AWS communication.

---

# Availability Zone

Example:

```text
us-east-1d
```

Represents a specific AWS data center inside a region.

---

# Important EC2 Values Found

During practical we saw:

```text
Instance ID
Public IP
Private IP
Security Group
Subnet
Volume ID
```

---

# Security Group

Output:

```text
default
```

Acts as a firewall.

Controls:

```text
Inbound Traffic
Outbound Traffic
```

---

# Subnet

Output:

```text
subnet-xxxxxxxx
```

Subnet defines the network location where the instance runs.

---

# Root Volume

Output:

```text
Volume Type = gp3
Volume Size = 8 GB
```

This is the main disk attached to the server.

---

# Important Learning

Creating EC2 automatically created:

```text
Network Interface
Storage Volume
Private IP
Public IP
```

along with the instance.

---

# Common Mistakes

## Mistake-1

Using wrong AMI.

Result:

```text
Instance Launch Failure
```

---

## Mistake-2

Using unavailable instance type.

Result:

```text
Instance creation fails
```

---

## Mistake-3

Using AMI from another region.

Example:

```text
AMI from ap-south-1
Used in us-east-1
```

Result:

```text
AMI Not Found
```

---

## Mistake-4

Forgetting tags.

Result:

Difficult resource management.

---

# Important Points Learned

* EC2 is a virtual server.
* AMI defines the operating system.
* Instance Type defines machine size.
* Tags help identify resources.
* Terraform uses state comparison before creating resources.
* terraform show displays resource details.
* Public IP and Private IP serve different purposes.
* Security Groups work as firewalls.

---

# Interview Questions

### What is EC2?

Amazon EC2 is a virtual server provided by AWS.

---

### What is an AMI?

AMI is a template used to launch EC2 instances.

---

### Why is AMI required?

AMI provides the operating system and initial configuration for the instance.

---

### What is an Instance Type?

Instance Type defines CPU, memory, and networking capacity.

---

### What is a Tag?

A tag is a key-value label attached to AWS resources.

---

### What is the difference between Public IP and Private IP?

Public IP is used for internet communication, while Private IP is used inside AWS networks.

---

### What is an Availability Zone?

An Availability Zone is an isolated AWS data center inside a region.

---

### What is the purpose of Security Groups?

Security Groups control inbound and outbound traffic for EC2 instances.

---

### Why did Terraform create only EC2 and not another S3 Bucket?

Because the S3 Bucket already existed in Terraform state and Terraform detected no change was required.

---

### What command shows EC2 details managed by Terraform?
terraform show

# Task-5: Understand Terraform State File

## Objective

Terraform must remember which resources it created.

For this purpose Terraform uses a special file called:

```text
terraform.tfstate
```

This file is one of the most important parts of Terraform.

Without state file, Terraform cannot properly manage infrastructure.

---

# What is Terraform State?

Terraform State is a file that stores information about resources created by Terraform.

File:

```text
terraform.tfstate
```

Terraform uses this file to track infrastructure.

---

# Real Example

Suppose a builder creates 100 houses.

Builder maintains a register:

```text
House-1
House-2
House-3
```

Without register:

```text
No tracking
No ownership information
No update history
```

Terraform State works exactly like that register.

---

# Why Terraform Needs State?

Suppose Terraform created:

```text
S3 Bucket
```

Tomorrow you add:

```text
EC2 Instance
```

Terraform must know:

```text
Bucket Already Exists
```

Otherwise Terraform would try to create it again.

---

# State Comparison Process

Terraform always compares:

```text
Current State
```

with

```text
Desired State
```

Flow:

```text
terraform.tfstate
        ↓
Current Infrastructure
        ↓
main.tf / *.tf
        ↓
Difference Detection
        ↓
Plan Output
```

---

# State File Created Automatically

After:

```bash
terraform apply
```

Terraform creates:

```text
terraform.tfstate
```

automatically.

---

# State File Structure

Example:

```json
{
  "resources": [
    {
      "type": "aws_s3_bucket",
      "name": "demo_bucket"
    }
  ]
}
```

Actual file contains much more information.

---

# Information Stored in State File

Example:

```text
Resource Type
Resource Name
Resource ID
ARN
Region
Tags
Attributes
```

---

# During Practical We Saw

Example:

```text
Bucket Name
Bucket ARN
Account ID
Region
Resource ID
```

stored inside state.

---

# Resource Type

Example:

```json
"type": "aws_s3_bucket"
```

Meaning:

```text
Terraform is managing an S3 Bucket.
```

---

# Resource Name

Example:

```json
"name": "demo_bucket"
```

Terraform Logical Name.

---

# Resource ID

Example:

```json
"id": "terraform-praveen-307946647949-demo"
```

AWS Resource Identifier.

---

# ARN

Example:

```json
"arn": "arn:aws:s3:::terraform-praveen-demo"
```

AWS Unique Resource Name.

---

# Identity Block

Example:

```json
"identity": {
  "account_id": "307946647949"
}
```

Terraform knows:

```text
Which AWS Account
Which Region
Which Resource
```

belongs to the infrastructure.

---

# terraform show

Command:

```bash
terraform show
```

Purpose:

Shows Terraform state in human readable format.

---

# Example Output

```text
aws_s3_bucket.demo_bucket
```

or

```text
aws_instance.web
```

with all attributes.

---

# Why Use terraform show?

Benefits:

```text
Easy Reading
Resource Details
Current Infrastructure View
```

---

# terraform state list

Command:

```bash
terraform state list
```

Purpose:

Displays all resources currently managed by Terraform.

---

# Example Output

```text
aws_instance.web
aws_s3_bucket.demo_bucket
```

Meaning:

Terraform manages:

```text
1 EC2 Instance
1 S3 Bucket
```

---

# Why state list is Useful?

Useful when:

```text
Large Projects
Hundreds of Resources
State Troubleshooting
```

---

# terraform state show

Command:

```bash
terraform state show aws_instance.web
```

Purpose:

Shows detailed information for one specific resource.

---

# Example

```bash
terraform state show aws_s3_bucket.demo_bucket
```

Displays:

```text
Bucket Name
ARN
Region
Encryption
Versioning
```

---

# Difference Between Commands

## terraform show

Shows:

```text
Entire State
```

---

## terraform state list

Shows:

```text
Resource Names Only
```

---

## terraform state show

Shows:

```text
Single Resource Details
```

---

# Why Terraform Did Not Recreate S3 Bucket?

Current State:

```text
S3 Bucket Exists
```

Desired State:

```text
S3 Bucket Exists
EC2 Instance Needed
```

Difference:

```text
Only EC2 Missing
```

Result:

```text
Create EC2 Only
```

---

# Important Interview Question

### How does Terraform know what already exists?

Terraform reads the state file and compares it with the current configuration.

---

# Why Should We Never Edit State File Manually?

Suppose:

State File:

```text
EC2 Exists
```

AWS:

```text
EC2 Exists
```

Everything is correct.

---

Now manually remove EC2 from state file.

Terraform thinks:

```text
EC2 Does Not Exist
```

while AWS still has the EC2.

Result:

```text
State Corruption
Duplicate Resources
Unexpected Changes
```

---

# Important Rule

Never manually edit:

```text
terraform.tfstate
```

unless absolutely necessary.

---

# Why State File Should Not Be Committed to Git?

State file may contain:

```text
Resource IDs
ARNs
IP Addresses
Account Information
Sensitive Values
```

---

# Problems

## Security Risk

Sensitive information may leak.

---

## Team Conflicts

Multiple users changing state can create merge conflicts.

---

# Recommended .gitignore

```gitignore
.terraform/
*.tfstate
*.tfstate.backup
```

---

# Production Approach

Instead of storing state locally:

```text
Local Machine
```

Production teams store state remotely.

Example:

```text
AWS S3 Backend
```

Benefits:

```text
Shared Access
Backup
Versioning
Collaboration
```

---

# Task-6: Modify Infrastructure

## Objective

Update an existing resource.

Current Tag:

```terraform
Name = "TerraWeek-Day1"
```

New Tag:

```terraform
Name = "TerraWeek-Modified"
```

---

# Why Modify Infrastructure?

Infrastructure changes happen regularly.

Examples:

```text
Tag Updates
Security Group Updates
Instance Type Updates
Storage Changes
```

Terraform manages all these updates.

---

# Terraform Plan After Change

Command:

```bash
terraform plan
```

Output:

```text
~ update in-place
```

---

# What Does ~ Mean?

Symbol:

```text
~
```

Meaning:

```text
Modify Existing Resource
```

without recreating it.

---

# Example Output

```text
Name:
TerraWeek-Day1
      ↓
TerraWeek-Modified
```

Only tag changed.

Server remained the same.

---

# In-Place Update

Terraform updates the existing resource directly.

No deletion.

No recreation.

---

# Real Example

Changing:

```text
Laptop Sticker
```

does not require buying a new laptop.

Same concept.

---

# Plan Summary

Output:

```text
Plan: 0 to add, 1 to change, 0 to destroy
```

Meaning:

```text
0 New Resources
1 Update
0 Delete
```

---

# Terraform Symbols Revision

## +

```text
Create
```

---

## ~

```text
Modify
```

---

## -

```text
Destroy
```

---

## -/+

```text
Destroy and Recreate
```

---

# Task-6: Destroy Infrastructure

After learning we removed all resources.

Command:

```bash
terraform destroy
```

Purpose:

Deletes all resources managed by Terraform.

---

# Why Destroy Resources?

Reasons:

```text
Avoid Cost
Clean Environment
Testing Complete
```

---

# Destroy Plan

Output:

```text
Plan: 0 to add, 0 to change, 2 to destroy
```

Meaning:

```text
EC2 Delete
S3 Bucket Delete
```

---

# Destroy Confirmation

Terraform asks:

```text
Do you really want to destroy all resources?
```

Type:

```text
yes
```

---

# Successful Output

```text
Destroy complete!
Resources: 2 destroyed.
```

Meaning:

```text
EC2 Deleted
S3 Deleted
```

Infrastructure cleaned successfully.

---

# Important Learning from Task-6

Terraform can:

```text
Create Infrastructure
Update Infrastructure
Delete Infrastructure
```

using the same code.

---

# Common Mistakes

## Mistake-1

Deleting resources manually from AWS Console.

Result:

```text
State Drift
```

Terraform state and AWS become different.

---

## Mistake-2

Editing terraform.tfstate manually.

Result:

```text
Broken State
```

---

## Mistake-3

Forgetting terraform destroy after learning.

Result:

```text
Unnecessary AWS Charges
```

---

## Mistake-4

Committing state file to GitHub.

Result:

```text
Security Risk
```

---

# Important Points Learned

* State file tracks infrastructure.
* terraform show displays state details.
* terraform state list displays managed resources.
* terraform state show displays a single resource.
* State file should not be edited manually.
* State file should not be committed to Git.
* Terraform uses state comparison before making changes.
* terraform destroy removes infrastructure safely.
* Terraform supports create, update, and destroy operations.

---

# Interview Questions

### What is Terraform State?

Terraform State stores information about resources managed by Terraform.

---

### What does terraform show do?

Displays the current state in human readable format.

---

### What does terraform state list do?

Displays all resources managed by Terraform.

---

### What does terraform state show do?

Displays detailed information about a specific resource.

---

### Why is Terraform State important?

Terraform uses it to track resources and calculate changes.

---

### Why should terraform.tfstate not be committed to Git?

It may contain sensitive information and create collaboration issues.

---

### What does ~ mean in Terraform Plan?

It means an existing resource will be updated.

---

### What does - mean in Terraform Plan?

It means a resource will be deleted.

---

### What does terraform destroy do?

Deletes all resources managed by Terraform.

---

### What is State Drift?
State drift occurs when infrastructure is changed outside Terraform and the state no longer matches reality.

# Part-6: Production Notes, Best Practices, Revision Notes, Interview Questions

---

# Terraform Architecture Revision

Terraform works in the following flow:

```text
Terraform Code
        │
        ▼
Terraform Core
        │
        ▼
Provider
        │
        ▼
Cloud API
        │
        ▼
AWS Resources
```

Example:

```text
main.tf
      ↓
AWS Provider
      ↓
AWS API
      ↓
EC2 / S3 / VPC
```

---

# AWS Authentication Flow Revision

During this task we used:

```bash
aws configure
```

Terraform used the same credentials.

Flow:

```text
Terraform
    ↓
AWS Provider
    ↓
AWS Credentials
    ↓
AWS API
    ↓
AWS Resources
```

---

# Where AWS Credentials Are Stored?

Windows:

```text
C:\Users\<username>\.aws\
```

Files:

```text
credentials
config
```

---

# Terraform Complete Workflow

Every Terraform project follows this workflow:

```text
Write Code
    ↓
terraform fmt
    ↓
terraform validate
    ↓
terraform init
    ↓
terraform plan
    ↓
terraform apply
    ↓
terraform show
    ↓
terraform destroy
```

Remember this sequence.

Interviewers often ask this question.

---

# Terraform Commands Quick Revision

## terraform fmt

Formats Terraform files.

```bash
terraform fmt
```

---

## terraform validate

Checks configuration syntax.

```bash
terraform validate
```

---

## terraform init

Initializes project and downloads providers.

```bash
terraform init
```

---

## terraform plan

Shows upcoming changes.

```bash
terraform plan
```

---

## terraform apply

Creates or updates resources.

```bash
terraform apply
```

---

## terraform show

Displays current infrastructure state.

```bash
terraform show
```

---

## terraform state list

Displays managed resources.

```bash
terraform state list
```

---

## terraform state show

Displays one specific resource.

```bash
terraform state show aws_instance.web
```

---

## terraform destroy

Deletes infrastructure.

```bash
terraform destroy
```

---

# Production Best Practices

## Use Separate Files

Bad:

```text
main.tf
(1000+ lines)
```

Good:

```text
main.tf
ec2.tf
s3.tf
variables.tf
outputs.tf
```

---

## Always Use Version Locking

Example:

```terraform
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}
```

Without version locking, provider updates may break code.

---

## Use Meaningful Resource Names

Bad:

```terraform
resource "aws_instance" "a" {
}
```

Good:

```terraform
resource "aws_instance" "web_server" {
}
```

---

## Always Tag Resources

Example:

```terraform
tags = {
  Name        = "WebServer"
  Environment = "Dev"
  Owner       = "Praveen"
}
```

Benefits:

```text
Easy Identification
Cost Tracking
Automation
```

---

## Never Hardcode Credentials

Bad:

```terraform
provider "aws" {
  access_key = "xxxx"
  secret_key = "xxxx"
}
```

Good:

```text
aws configure
IAM Role
Environment Variables
```

---

## Keep State File Safe

Never edit manually.

Never upload to GitHub.

---

# Recommended .gitignore

```gitignore
.terraform/
*.tfstate
*.tfstate.backup
```

---

# Common Errors Faced During Day-61

## Error-1

```text
terraform: command not found
```

Cause:

Terraform not added to PATH.

Fix:

Add Terraform folder to Environment Variables.

---

## Error-2

```text
Missing required provider
```

Cause:

terraform init not executed.

Fix:

```bash
terraform init
```

---

## Error-3

```text
Unable to locate credentials
```

Cause:

AWS CLI not configured.

Fix:

```bash
aws configure
```

---

## Error-4

```text
BucketAlreadyExists
```

Cause:

S3 bucket name already used globally.

Fix:

Use unique bucket name.

---

## Error-5

```text
Unsupported argument
```

Cause:

Wrong Terraform syntax.

Fix:

Check resource block structure.

---

# Important Concepts Learned in Day-61

## Infrastructure as Code (IaC)

Managing infrastructure using code instead of manual clicks.

---

## Provider

Connects Terraform to AWS.

---

## Resource

Infrastructure object managed by Terraform.

Examples:

```text
EC2
S3
VPC
RDS
```

---

## State

Terraform database.

File:

```text
terraform.tfstate
```

---

## AMI

Template used to launch EC2.

---

## Instance Type

Defines CPU and memory size.

---

## Tags

Labels attached to resources.

---

# Beginner Mistakes to Avoid

### Creating resources manually and expecting Terraform to know

Terraform only tracks resources present in state.

---

### Editing state file manually

Can corrupt infrastructure tracking.

---

### Forgetting terraform plan

Always review changes before apply.

---

### Forgetting terraform destroy during practice

May cause AWS charges.

---

### Committing credentials to GitHub

Major security risk.

---

# 30 Important Interview Questions

## 1. What is Terraform?

Terraform is an Infrastructure as Code tool used to create and manage infrastructure.

---

## 2. What is Infrastructure as Code?

Managing infrastructure using code instead of manual configuration.

---

## 3. What is a Provider?

A plugin that allows Terraform to communicate with cloud platforms.

---

## 4. What is a Resource?

An infrastructure component managed by Terraform.

---

## 5. What is Terraform State?

A file that stores infrastructure information managed by Terraform.

---

## 6. Why is State Important?

Terraform uses state to track resources and calculate changes.

---

## 7. What is terraform init?

Initializes a Terraform project.

---

## 8. What is terraform plan?

Shows proposed infrastructure changes.

---

## 9. What is terraform apply?

Creates or updates infrastructure.

---

## 10. What is terraform destroy?

Deletes Terraform-managed infrastructure.

---

## 11. What is terraform validate?

Checks Terraform syntax.

---

## 12. What is terraform fmt?

Formats Terraform files.

---

## 13. What is an AMI?

Template used to launch EC2 instances.

---

## 14. What is an Instance Type?

Defines CPU, memory, and networking capacity.

---

## 15. What are Tags?

Labels attached to AWS resources.

---

## 16. What does + mean in plan?

Create resource.

---

## 17. What does ~ mean in plan?

Update resource.

---

## 18. What does - mean in plan?

Delete resource.

---

## 19. What does -/+ mean?

Destroy and recreate resource.

---

## 20. What is AWS CLI?

Command line tool for AWS services.

---

## 21. What is Terraform Registry?

Repository containing providers and modules.

---

## 22. Why should state not be committed to Git?

May contain sensitive information.

---

## 23. What is a Bucket?

Container used to store objects in S3.

---

## 24. What is EC2?

Virtual server provided by AWS.

---

## 25. What is Public IP?

IP address used for internet communication.

---

## 26. What is Private IP?

IP address used within internal networks.

---

## 27. What is Availability Zone?

Physical data center location inside a region.

---

## 28. How does Terraform know what exists?

By reading the state file.

---

## 29. Difference between Terraform and Ansible?

Terraform provisions infrastructure.

Ansible configures servers and applications.

---

## 30. Difference between Terraform and CloudFormation?

Terraform supports multiple clouds.

CloudFormation is AWS-specific.

---

# Scenario-Based Interview Questions

## Scenario 1

You created an EC2 manually from AWS Console. Will Terraform manage it automatically?

Answer:

No. Terraform only manages resources tracked in state.

---

## Scenario 2

State file is deleted accidentally. What happens?

Answer:

Terraform loses tracking information and may try to recreate resources.

---

## Scenario 3

Developer changed EC2 manually in AWS Console.

What problem can occur?

Answer:

State Drift.

---

## Scenario 4

terraform plan shows:

```text
~ update in-place
```

What does it mean?

Answer:

Existing resource will be modified without recreation.

---

## Scenario 5

terraform plan shows:

```text
-/+
```

What does it mean?

Answer:

Resource will be destroyed and recreated.

---

## Scenario 6

S3 bucket already exists but Terraform wants to create it again.

Possible reason?

Answer:

Resource is missing from state file.

---

## Scenario 7

AWS credentials expired.

What happens?

Answer:

Terraform cannot authenticate with AWS.

---

## Scenario 8

You forgot terraform init.

What error may appear?

Answer:

Missing required provider.

---

## Scenario 9

Bucket name is already taken globally.

What happens?

Answer:

Bucket creation fails.

---

## Scenario 10

You forgot terraform destroy after practice.

What risk exists?

Answer:

Unnecessary AWS charges.

---

# Quick Revision Notes

Remember:

Provider = Connection to AWS

Resource = Infrastructure Object

State = Terraform Database

AMI = Operating System Template

Instance Type = Machine Size

Tag = Resource Label



