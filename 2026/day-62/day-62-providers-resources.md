# Day 62 - Providers, Resources and Dependencies

> **#90DaysOfDevOps - Terraform Day 62**

# Introduction

On **Day 62**, we moved to real-world infrastructure where every resource depends on another resource. Instead of creating individual resources, we built a complete AWS networking environment using Terraform.

In this practical, we learned:

* AWS Provider
* Provider Version Constraints
* Terraform Initialization
* Provider Lock File
* AWS VPC
* Public Subnet
* Internet Gateway
* Route Table
* Route Table Association
* Security Group
* EC2 Instance
* S3 Bucket
* Implicit Dependencies
* Explicit Dependencies (`depends_on`)
* Terraform Graph
* Lifecycle Rules
* Terraform Destroy

This lab demonstrates how Terraform automatically determines the correct order for creating and destroying infrastructure resources.

---

# Learning Objectives

After completing this lab, you will be able to:

* Understand Terraform Providers.
* Configure AWS Provider.
* Understand Provider Version Constraints.
* Initialize a Terraform Project.
* Build a complete AWS Networking Infrastructure.
* Understand Terraform Dependencies.
* Create an EC2 Instance inside a VPC.
* Use Security Groups.
* Create an S3 Bucket.
* Use Explicit Dependencies.
* Generate a Dependency Graph.
* Understand Lifecycle Rules.
* Safely Destroy Infrastructure.

---

# Prerequisites

Before starting this lab, ensure you have:

* Terraform Installed
* AWS Account
* AWS CLI Configured
* IAM User with Programmatic Access
* Valid AWS Credentials
* Basic Terraform Knowledge (Day 61)

---

# Project Directory Structure

```
terraform-aws-infra/
│
├── providers.tf
├── main.tf
├── .terraform.lock.hcl
├── .terraform/
└── terraform.tfstate
```

---

# Terraform Workflow

Terraform follows a simple workflow.

```
Write Code
      │
      ▼
terraform fmt
      │
      ▼
terraform validate
      │
      ▼
terraform init
      │
      ▼
terraform plan
      │
      ▼
terraform apply
      │
      ▼
Verify Resources
      │
      ▼
terraform destroy
```

---

# What is a Provider?

A **Provider** is a plugin that allows Terraform to communicate with cloud platforms and services.

Terraform itself does not know how to create AWS resources.

The AWS Provider translates Terraform configuration into AWS API calls.

Example:

```
Terraform

      │

AWS Provider

      │

AWS API

      │

AWS Cloud
```

Without a provider, Terraform cannot create any infrastructure.

---

# Why Do We Need a Provider?

Suppose Terraform wants to create an EC2 instance.

Instead of directly communicating with AWS, Terraform sends the request to the AWS Provider.

The provider then communicates with AWS APIs.

```
terraform apply

↓

AWS Provider

↓

Create EC2 API

↓

AWS Creates EC2
```

---

# Provider Configuration

Create **providers.tf**

```hcl
terraform {
  required_providers {

    aws = {

      source  = "hashicorp/aws"

      version = "~> 5.0"

    }

  }
}

provider "aws" {

  region = "us-east-1"

}
```

---

# Code Explanation

## terraform Block

The `terraform` block defines Terraform-specific configuration.

It is used to specify:

* Required Providers
* Required Terraform Version
* Backend Configuration

---

## required_providers

This block tells Terraform which providers are required.

Example:

```
required_providers {

aws

}
```

Terraform downloads the provider automatically during initialization.

---

## source

```
source = "hashicorp/aws"
```

This tells Terraform to download the AWS Provider from the official HashiCorp Registry.

---

## version

```
version = "~> 5.0"
```

This restricts Terraform to use only compatible versions within the 5.x release.

---

## provider Block

```
provider "aws" {

region="us-east-1"

}
```

This configures the AWS Provider.

In our lab, all resources are created inside the **us-east-1** region.

---

# Provider Version Constraints

Terraform supports multiple version constraints.

## Exact Version

```
= 5.0.0
```

Only version **5.0.0** is allowed.

---

## Greater Than

```
>=5.0
```

Allows any version greater than or equal to 5.0.

---

## Less Than

```
<=5.0
```

Allows version 5.0 or below.

---

## Compatible Version

```
~>5.0
```

Allows:

* 5.0
* 5.10
* 5.25
* 5.100

Does NOT allow:

* 6.0

This is the most commonly used version constraint in production.

---

# Terraform Initialization

Run:

```bash
terraform init
```

Terraform performs the following operations:

1. Reads Terraform Configuration
2. Downloads Required Providers
3. Creates the `.terraform` Directory
4. Creates `.terraform.lock.hcl`
5. Initializes Backend

Example Output

```
Initializing the backend...

Initializing provider plugins...

Finding hashicorp/aws...

Installing hashicorp/aws...

Terraform has been successfully initialized.
```

---

# .terraform Directory

After initialization, Terraform creates a hidden folder.

```
.terraform/
```

This directory contains downloaded provider plugins.

This folder should **NOT** be committed to Git.

Example:

```
.terraform/

└── providers

    └── registry.terraform.io

        └── hashicorp

            └── aws
```

---

# .terraform.lock.hcl

Terraform automatically creates this file.

Example:

```hcl
provider "registry.terraform.io/hashicorp/aws" {

version = "5.100.0"

constraints = "~> 5.0"

}
```

This file stores:

* Exact Provider Version
* Version Constraints
* Security Hashes

Purpose:

* Ensures every developer installs the same provider version.
* Prevents unexpected version upgrades.
* Verifies provider authenticity using hashes.

Unlike `.terraform`, this file **should be committed to Git**.

---

# Common Terraform Commands

| Command              | Purpose                                         |
| -------------------- | ----------------------------------------------- |
| `terraform fmt`      | Formats Terraform code                          |
| `terraform validate` | Validates configuration                         |
| `terraform init`     | Downloads providers and initializes the project |
| `terraform plan`     | Shows execution plan                            |
| `terraform apply`    | Creates infrastructure                          |
| `terraform destroy`  | Deletes infrastructure                          |


# Best Practices
* Pin provider versions.
* Commit `.terraform.lock.hcl`.
* Do not commit `.terraform/`.
* Never hardcode AWS credentials.
* Always run `terraform fmt` before committing code.
* Review `terraform plan` before applying changes.
* Destroy unused infrastructure to avoid unnecessary AWS charges.

## Summary

In this section, we learned:
* What a Terraform Provider is.
* Why Providers are required.
* AWS Provider Configuration.
* Version Constraints.
* Terraform Initialization.
* `.terraform` Directory.
* `.terraform.lock.hcl`.
* Basic Terraform Workflow.
* Essential Terraform Commands.

**Next:** Part 2 - Building AWS Networking Infrastructure (VPC, Subnet, Internet Gateway, Route Table, and Route Table Association).
# Day 62 - Providers, Resources and Dependencies

# Part 2 - Building AWS Networking Infrastructure

---

# Introduction

Before launching an EC2 instance, we first need a network.

In AWS, an EC2 instance cannot exist without a VPC and a subnet.

The networking components created in this lab are:

* VPC
* Public Subnet
* Internet Gateway
* Route Table
* Route Table Association

These resources form the foundation of AWS networking.

---

# AWS Network Architecture

```text
                     Internet
                         │
                         │
                Internet Gateway
                         │
                         │
                  Route Table
                         │
                         │
                  Public Subnet
                         │
                         │
                    EC2 Instance
                         │
                         │
                        VPC
```

Every component is connected to another component.

Terraform automatically creates these resources in the correct order.

---

# What is a VPC?

**VPC (Virtual Private Cloud)** is a logically isolated virtual network inside AWS.

A VPC allows you to create your own private network where you can deploy AWS resources securely.

Think of a VPC as your own private data center inside AWS.

---

# Why Do We Need a VPC?

Without a VPC:

* Resources would not have a private network.
* Network isolation would not exist.
* Security and routing would not be possible.

A VPC provides:

* Private Networking
* IP Address Management
* Routing
* Security
* Internet Connectivity

---

# Create a VPC

```hcl
resource "aws_vpc" "main" {

  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "TerraWeek-VPC"
  }

}
```

---

# Code Explanation

## resource

Creates a new AWS resource.

---

## aws_vpc

Specifies that Terraform should create a Virtual Private Cloud.

---

## main

Logical name used by Terraform.

Terraform references this resource as:

```hcl
aws_vpc.main
```

---

## cidr_block

```hcl
cidr_block = "10.0.0.0/16"
```

Defines the IP address range of the VPC.

CIDR:

```
10.0.0.0/16
```

Provides approximately **65,536 IP addresses**.

Formula:

```
2^(32 - Prefix)

2^(32-16)

2^16

65536
```

---

## tags

```hcl
tags = {

Name = "TerraWeek-VPC"

}
```

Tags help identify resources easily in the AWS Console.

---

# What is a Subnet?

A **Subnet** is a smaller network inside a VPC.

Every EC2 instance is launched inside a subnet.

A VPC can contain multiple subnets.

Example:

```
VPC

├── Public Subnet

├── Private Subnet

└── Database Subnet
```

---

# Public vs Private Subnet

| Public Subnet       | Private Subnet            |
| ------------------- | ------------------------- |
| Has Internet Access | No Direct Internet Access |
| Hosts Web Servers   | Hosts Databases           |
| Public IP Allowed   | Public IP Not Required    |

In this lab, we create a **Public Subnet**.

---

# Create Public Subnet

```hcl
resource "aws_subnet" "public" {

  vpc_id = aws_vpc.main.id

  cidr_block = "10.0.1.0/24"

  map_public_ip_on_launch = true

  tags = {

    Name = "TerraWeek-Public-Subnet"

  }

}
```

---

# Code Explanation

## vpc_id

```hcl
vpc_id = aws_vpc.main.id
```

Associates the subnet with the VPC.

Terraform automatically understands that the VPC must be created first.

---

## cidr_block

```hcl
10.0.1.0/24
```

Provides approximately **256 IP addresses**.

Formula:

```
2^(32-24)

2^8

256
```

---

## map_public_ip_on_launch

```hcl
true
```

Automatically assigns a public IP address to EC2 instances launched inside this subnet.

---

# What is an Internet Gateway?

An **Internet Gateway (IGW)** connects a VPC to the public Internet.

Without an Internet Gateway:

* No Internet Access
* No Public Connectivity

---

# Create Internet Gateway

```hcl
resource "aws_internet_gateway" "main" {

  vpc_id = aws_vpc.main.id

  tags = {

    Name = "TerraWeek-IGW"

  }

}
```

---

# Code Explanation

## vpc_id

Attaches the Internet Gateway to the VPC.

Without attaching the IGW to a VPC, Internet connectivity will not work.

---

# What is a Route Table?

A **Route Table** controls where network traffic should go.

It acts like a routing map.

Example:

Destination:

```
0.0.0.0/0
```

Gateway:

```
Internet Gateway
```

Meaning:

Send all Internet traffic through the Internet Gateway.

---

# Create Route Table

```hcl
resource "aws_route_table" "public" {

  vpc_id = aws_vpc.main.id

  route {

    cidr_block = "0.0.0.0/0"

    gateway_id = aws_internet_gateway.main.id

  }

  tags = {

    Name = "TerraWeek-Public-RT"

  }

}
```

---

# Code Explanation

## route

Defines a routing rule.

---

## 0.0.0.0/0

Represents all IPv4 addresses on the Internet.

Meaning:

All Internet traffic should follow this route.

---

## gateway_id

Specifies that Internet traffic should use the Internet Gateway.

---

# What is Route Table Association?

Creating a Route Table is not enough.

It must be attached to a subnet.

This attachment is called **Route Table Association**.

---

# Create Route Table Association

```hcl
resource "aws_route_table_association" "public" {

  subnet_id = aws_subnet.public.id

  route_table_id = aws_route_table.public.id

}
```

---

# Code Explanation

## subnet_id

Specifies which subnet will use this Route Table.

---

## route_table_id

Specifies which Route Table should be attached.

---

# Complete Networking Flow

```
Internet
    │
    ▼
Internet Gateway
    │
    ▼
Route Table
    │
    ▼
Public Subnet
    │
    ▼
EC2
```

---

# Terraform Commands

Format Configuration

```bash
terraform fmt
```

---

Validate Configuration

```bash
terraform validate
```

Expected Output

```
Success! The configuration is valid.
```

---

Execution Plan

```bash
terraform plan
```

Expected Output

```
Plan: 5 to add, 0 to change, 0 to destroy.
```

---

Apply Configuration

```bash
terraform apply
```

Expected Output

```
Apply complete!

Resources: 5 added, 0 changed, 0 destroyed.
```

---

# AWS Resources Created

After applying the configuration, the following resources are created:

* VPC
* Public Subnet
* Internet Gateway
* Route Table
* Route Table Association

# Summary

In this section, we learned:

* Virtual Private Cloud (VPC)
* CIDR Blocks
* Public Subnet
* Internet Gateway
* Route Table
* Route Table Association
* AWS Networking Flow
* Terraform Networking Resources
* Planning and Applying Infrastructure


**Next:** Part 3 - Security Groups, EC2 Instance, S3 Bucket, Implicit & Explicit Dependencies.
# Day 62 - Providers, Resources and Dependencies

# Part 3 - Security Group, EC2, S3 Bucket and Terraform Dependencies

---

# Introduction

After creating the AWS networking infrastructure, the next step is to launch an EC2 instance.

However, an EC2 instance cannot be accessed unless we configure firewall rules.

In AWS, firewall rules are managed using **Security Groups**.

In this section, we will create:

* Security Group
* EC2 Instance
* S3 Bucket
* Implicit Dependencies
* Explicit Dependencies (`depends_on`)
* Terraform Dependency Graph

---

# Infrastructure Architecture

```text
                  Internet
                      │
                      ▼
             Internet Gateway
                      │
                      ▼
                Route Table
                      │
                      ▼
               Public Subnet
                      │
                      ▼
             Security Group
                      │
                      ▼
                 EC2 Instance
                      │
                      ▼
                  S3 Bucket
```

---

# What is a Security Group?

A **Security Group** is a virtual firewall that controls traffic to and from an AWS resource.

It controls:

* Incoming Traffic (Ingress)
* Outgoing Traffic (Egress)

Unlike traditional firewalls, Security Groups are **stateful**, meaning if incoming traffic is allowed, the response traffic is automatically allowed.

---

# Ingress vs Egress

| Ingress          | Egress                        |
| ---------------- | ----------------------------- |
| Incoming Traffic | Outgoing Traffic              |
| Internet → EC2   | EC2 → Internet                |
| SSH, HTTP, HTTPS | Updates, API Calls, Downloads |

---

# Create Security Group

```hcl
resource "aws_security_group" "main" {

  name        = "TerraWeek-SG"
  description = "Allow SSH and HTTP"

  vpc_id = aws_vpc.main.id

  ingress {
    description = "SSH Access"

    from_port = 22
    to_port   = 22

    protocol = "tcp"

    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {

    description = "HTTP Access"

    from_port = 80
    to_port   = 80

    protocol = "tcp"

    cidr_blocks = ["0.0.0.0/0"]

  }

  egress {

    from_port = 0
    to_port   = 0

    protocol = "-1"

    cidr_blocks = ["0.0.0.0/0"]

  }

  tags = {

    Name = "TerraWeek-SG"

  }

}
```

---

# Code Explanation

## name

Security Group name shown in the AWS Console.

---

## description

Provides information about the purpose of the Security Group.

---

## vpc_id

Associates the Security Group with the VPC.

```hcl
vpc_id = aws_vpc.main.id
```

---

## ingress

Defines incoming traffic rules.

### SSH Rule

```text
Port : 22

Protocol : TCP

Source : 0.0.0.0/0
```

Allows SSH from anywhere.

---

### HTTP Rule

```text
Port : 80

Protocol : TCP

Source : 0.0.0.0/0
```

Allows HTTP traffic from anywhere.

---

## egress

Allows outbound traffic.

```text
Protocol : All

Destination : Anywhere
```

---

# Note

For learning purposes, SSH is open to everyone (`0.0.0.0/0`).

In production, SSH should be restricted to trusted IP addresses or VPN networks.

---

# What is an AMI?

AMI stands for **Amazon Machine Image**.

An AMI contains:

* Operating System
* Default Software
* Boot Configuration

Every EC2 instance must use an AMI.

---

# Create EC2 Instance

```hcl
resource "aws_instance" "main" {

  ami = "ami-0b6d9d3d33ba97d99"

  instance_type = "t2.micro"

  subnet_id = aws_subnet.public.id

  vpc_security_group_ids = [

    aws_security_group.main.id

  ]

  associate_public_ip_address = true

  tags = {

    Name = "TerraWeek-Server"

  }

}
```

---

# Code Explanation

## ami

Specifies the operating system image.

---

## instance_type

```text
t2.micro
```

Free Tier eligible instance.

---

## subnet_id

Launches the EC2 instance inside the Public Subnet.

---

## vpc_security_group_ids

Attaches the Security Group to the EC2 instance.

---

## associate_public_ip_address

```hcl
true
```

Automatically assigns a Public IPv4 address.

---

# Result

After applying:

* EC2 Instance Created
* Public IP Assigned
* SSH Enabled
* HTTP Enabled

---

# Create S3 Bucket

```hcl
resource "aws_s3_bucket" "logs" {

  bucket = "terraweek-praveen-logs-2026"

  depends_on = [

    aws_instance.main

  ]

  tags = {

    Name = "TerraWeek-Logs"

  }

}
```

---

# Code Explanation

## bucket

Defines the globally unique bucket name.

---

## depends_on

```hcl
depends_on = [

aws_instance.main

]
```

Forces Terraform to create the EC2 instance before creating the S3 bucket.

---

# Implicit Dependencies

Terraform automatically creates dependencies whenever one resource references another resource.

Example:

```hcl
vpc_id = aws_vpc.main.id
```

Terraform automatically understands:

```text
VPC

↓

Subnet
```

No manual configuration is required.

---

# Implicit Dependencies Used in This Lab

| Resource                | Depends On             |
| ----------------------- | ---------------------- |
| Public Subnet           | VPC                    |
| Internet Gateway        | VPC                    |
| Route Table             | VPC, Internet Gateway  |
| Security Group          | VPC                    |
| EC2 Instance            | Subnet, Security Group |
| Route Table Association | Route Table, Subnet    |

Terraform detects all of these automatically.

---

# Explicit Dependencies

Sometimes Terraform cannot determine the dependency automatically.

Example:

```hcl
resource "aws_s3_bucket" "logs" {

depends_on = [

aws_instance.main

]

}
```

There is no direct reference between the EC2 instance and the S3 bucket.

Using `depends_on`, we explicitly instruct Terraform to create the EC2 instance first.

---

# Implicit vs Explicit Dependencies

| Implicit Dependency               | Explicit Dependency        |
| --------------------------------- | -------------------------- |
| Automatic                         | Manual                     |
| Created using resource references | Created using `depends_on` |
| Recommended whenever possible     | Used only when necessary   |

---

# Terraform Graph

Terraform can visualize all dependencies.

Run:

```bash
terraform graph
```

Sample Output:

```text
aws_instance.main

↓

aws_security_group.main

↓

aws_vpc.main
```

Your graph also showed:

```text
aws_instance.main -> aws_security_group.main

aws_instance.main -> aws_subnet.public

aws_security_group.main -> aws_vpc.main

aws_route_table.public -> aws_internet_gateway.main

aws_s3_bucket.logs -> aws_instance.main
```

This confirms that Terraform correctly built the dependency graph.

---

# Generate Graph Image

If Graphviz is installed:

```bash
terraform graph | dot -Tpng > graph.png
```

This creates a PNG image of the dependency graph.

---

# Terraform Commands

```bash
terraform fmt
```

Formats Terraform code.

---

```bash
terraform validate
```

Validates Terraform configuration.

---

```bash
terraform plan
```

Shows the execution plan.

Expected:

```text
Plan: 2 to add, 0 to change, 0 to destroy.
```

---

```bash
terraform apply
```

Creates:

* Security Group
* EC2 Instance

Expected:

```text
Apply complete!

Resources: 2 added, 0 changed, 0 destroyed.

Later:

```bash
terraform apply
```

Creates:

* S3 Bucket

Expected:

```text
Apply complete!

Resources: 1 added, 0 changed, 0 destroyed.

# Summary

In this section, we learned:

* Security Groups
* Ingress Rules
* Egress Rules
* EC2 Instance
* Amazon Machine Image (AMI)
* Public IP Assignment
* S3 Bucket
* Implicit Dependencies
* Explicit Dependencies (`depends_on`)
* Terraform Graph
* Dependency Visualization


**Next:** Part 4 - Lifecycle Rules, Terraform Destroy, Best Practices, Common Errors, Cheat Sheet, Revision Notes and 40+ Interview Questions & Answers.
# Day 62 - Providers, Resources and Dependencies

# Part 4 - Lifecycle Rules, Destroy, Best Practices, Cheat Sheet & Interview Questions

---

# Terraform Lifecycle

Terraform provides a **lifecycle** block to control how resources are created, updated, and destroyed.

Syntax:

```hcl
lifecycle {

}
```

Lifecycle rules help prevent accidental downtime and resource deletion.

---

# Lifecycle Rule 1 - create_before_destroy

This rule tells Terraform to create the new resource before deleting the old resource.

Example:

```hcl
resource "aws_instance" "main" {

  ami           = "ami-0b6d9d3d33ba97d99"
  instance_type = "t2.micro"

  lifecycle {

    create_before_destroy = true

  }

}
```

---

## Why Use create_before_destroy?

Suppose you change the AMI.

Without lifecycle:

```text
Old EC2
   │
Destroy
   │
   ▼
New EC2
```

This causes downtime.

---

With lifecycle:

```text
New EC2
   │
Create
   │
   ▼
Old EC2
Destroy
```

The new server is created first, reducing downtime.

---

# Terraform Plan After Changing AMI

When the AMI changes, Terraform shows:

```text
+/- create replacement and then destroy
```

Meaning:

* Create a new EC2 instance.
* Destroy the old EC2 instance.

---

# Lifecycle Rule 2 - prevent_destroy

Protects critical resources from accidental deletion.

Example:

```hcl
lifecycle {

  prevent_destroy = true

}
```

If someone runs:

```bash
terraform destroy
```

Terraform will stop with an error instead of deleting the resource.

### Common Use Cases

* Production Database
* Production S3 Bucket
* Critical EC2 Instance

---

# Lifecycle Rule 3 - ignore_changes

Sometimes changes are made manually in AWS.

Terraform normally tries to revert those changes.

To ignore specific changes:

```hcl
lifecycle {

  ignore_changes = [

    tags

  ]

}
```

Terraform will ignore updates to resource tags.

### Common Use Cases

* Auto Scaling updates
* Resource Tags
* Monitoring Configurations

---

# Terraform Destroy

To remove all infrastructure:

```bash
terraform destroy
```

Terraform displays a destroy plan before deleting resources.

Confirm by typing:

```text
yes
```

---

# Destroy Order

Terraform destroys resources in **reverse dependency order**.

Example:

```text
S3 Bucket
    │
    ▼
EC2
    │
    ▼
Security Group
    │
    ▼
Route Table Association
    │
    ▼
Route Table
    │
    ▼
Internet Gateway
    │
    ▼
Subnet
    │
    ▼
VPC
```

This prevents dependency errors during deletion.

---

# Terraform Commands Used

| Command              | Description              |
| -------------------- | ------------------------ |
| `terraform fmt`      | Format Terraform code    |
| `terraform validate` | Validate configuration   |
| `terraform init`     | Initialize project       |
| `terraform plan`     | Preview changes          |
| `terraform apply`    | Create infrastructure    |
| `terraform graph`    | Display dependency graph |
| `terraform destroy`  | Delete infrastructure    |

---

# Common Errors and Fixes

## Provider Version Conflict

**Error**

```text
Failed to query available provider packages
```

**Solution**

Run:

```bash
terraform init -upgrade
```

---

## Invalid AWS Credentials

**Error**

```text
InvalidClientTokenId
```

**Solution**

* Check AWS Access Key
* Check Secret Key
* Verify IAM permissions

---

## Bucket Already Exists

**Error**

```text
BucketAlreadyExists
```

**Solution**

Choose a globally unique bucket name.

---

## AMI Not Found

**Error**

```text
InvalidAMIID.NotFound
```

**Solution**

Use the correct AMI for your AWS region.

---

## SSH Not Working

Check:

* Security Group
* Public IP
* Route Table
* Internet Gateway
* Key Pair

---

# Best Practices

* Always run `terraform fmt` before committing code.
* Validate configuration before planning.
* Review every execution plan.
* Pin provider versions.
* Commit `.terraform.lock.hcl`.
* Never commit `.terraform/`.
* Never hardcode AWS credentials.
* Use remote state for production.
* Destroy unused resources to avoid AWS charges.

---

# Revision Notes

## Provider

Allows Terraform to communicate with AWS.

---

## Resource

Represents an AWS service such as:

* EC2
* S3
* VPC

---

## VPC

Private virtual network inside AWS.

---

## Subnet

Smaller network inside a VPC.

---

## Internet Gateway

Connects a VPC to the Internet.

---

## Route Table

Controls network routing.

---

## Security Group

Acts as a virtual firewall.

---

## EC2

Virtual Machine.

---

## AMI

Operating system template.

---

## S3

Object Storage Service.

---

## Implicit Dependency

Automatically detected by Terraform.

Example:

```hcl
vpc_id = aws_vpc.main.id
```

---

## Explicit Dependency

Manually defined using:

```hcl
depends_on = [

aws_instance.main

]


## Lifecycle

Controls resource creation and deletion behavior.

---

# Terraform Cheat Sheet

```bash
terraform fmt

terraform validate

terraform init

terraform plan

terraform apply

terraform graph

terraform destroy


# Architecture Overview

                Internet
                    │
                    ▼
          Internet Gateway
                    │
                    ▼
             Route Table
                    │
                    ▼
             Public Subnet
                    │
                    ▼
           Security Group
                    │
                    ▼
               EC2 Instance
                    │
                    ▼
               S3 Bucket


# Interview Questions & Answers

## Beginner Level

### 1. What is Terraform?

**Answer:** Terraform is an Infrastructure as Code (IaC) tool used to provision and manage infrastructure using code.

---

### 2. What is a Provider?

**Answer:** A Provider is a plugin that enables Terraform to interact with cloud platforms like AWS.

---

### 3. What is a Resource?

**Answer:** A Resource represents infrastructure components such as EC2, S3, VPC, or IAM.

---

### 4. What does `terraform init` do?

**Answer:** It initializes the working directory, downloads providers, and creates the lock file.

---

### 5. What does `terraform plan` do?

**Answer:** It shows what changes Terraform will make without applying them.

---

### 6. What does `terraform apply` do?

**Answer:** It creates or updates infrastructure based on the Terraform configuration.

---

### 7. What does `terraform destroy` do?

**Answer:** It deletes all resources managed by the current Terraform configuration.

---

### 8. What is a VPC?

**Answer:** A VPC is a logically isolated virtual network in AWS.

---

### 9. What is a Subnet?

**Answer:** A Subnet is a smaller network segment inside a VPC.

---

### 10. What is an Internet Gateway?

**Answer:** It connects a VPC to the public Internet.

---

### 11. What is a Route Table?

**Answer:** A Route Table defines how network traffic is routed.

---

### 12. What is a Security Group?

**Answer:** A Security Group is a stateful virtual firewall for AWS resources.

---

### 13. What is an AMI?

**Answer:** An Amazon Machine Image is a template used to launch EC2 instances.

---

### 14. What is an EC2 Instance?

**Answer:** An EC2 Instance is a virtual server running inside AWS.

---

### 15. What is S3?

**Answer:** Amazon S3 is an object storage service.

---

## Intermediate Level

### 16. What is an Implicit Dependency?

**Answer:** A dependency automatically detected by Terraform through resource references.

---

### 17. What is an Explicit Dependency?

**Answer:** A dependency manually defined using `depends_on`.

---

### 18. What is `.terraform.lock.hcl`?

**Answer:** It stores provider versions and security hashes.

---

### 19. Why should `.terraform.lock.hcl` be committed?

**Answer:** To ensure all team members use the same provider version.

---

### 20. Why should `.terraform/` not be committed?

**Answer:** It contains downloaded provider binaries that can be recreated using `terraform init`.

---

### 21. What does `create_before_destroy` do?

**Answer:** It creates a replacement resource before deleting the old one.

---

### 22. What does `prevent_destroy` do?

**Answer:** It prevents accidental deletion of critical resources.

---

### 23. What does `ignore_changes` do?

**Answer:** It instructs Terraform to ignore updates to selected resource attributes.

---

### 24. Why do we use `terraform graph`?

**Answer:** To visualize resource dependencies.

---

### 25. Why should we review `terraform plan`?

**Answer:** To verify infrastructure changes before applying them.

---

## Advanced Level

### 26. Difference between Implicit and Explicit Dependency?

**Answer:** Implicit dependencies are detected automatically through references, while explicit dependencies are manually defined using `depends_on`.

---

### 27. Why is dependency management important?

**Answer:** It ensures resources are created and destroyed in the correct order.

---

### 28. Why does changing an AMI replace an EC2 instance?

**Answer:** The AMI is an immutable property, so AWS requires a new instance.

---

### 29. Why is `create_before_destroy` useful?

**Answer:** It minimizes downtime during resource replacement.

---

### 30. Why should unused infrastructure be destroyed?

**Answer:** To avoid unnecessary AWS charges.

---

# Conclusion

Day 62 introduced Terraform's dependency management by building a complete AWS networking environment from scratch.

During this lab, we successfully:

* Configured the AWS Provider.
* Built a VPC and networking components.
* Created a Security Group.
* Launched an EC2 Instance.
* Created an S3 Bucket.
* Explored Implicit and Explicit Dependencies.
* Generated a Dependency Graph.
* Learned Terraform Lifecycle Rules.
* Safely destroyed AWS resources.

This completes **Day 62 - Providers, Resources and Dependencies**.


