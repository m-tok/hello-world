# Amazon AWS VPC

This Terraform configuration creates:


* Amazon AWS VPC
* Subnets in all configured availability zones
* Routing tables linking them to the Internet Gateway and NAT Gateway.

### Table of Contents 

[Amazon AWS VPC](https://github.com/m-tok/hello-world/edit/main/README.md#amazon-aws-vpc)

* [Prerequisites and dependencies](https://github.com/m-tok/hello-world/edit/main/README.md#prerequisites-and-dependencies)

* [Creating the VPC and Subnets](https://github.com/m-tok/hello-world/edit/main/README.md#creating-thevpc-and-subnets)
* [Attaching IGW and NG to Subnets](https://github.com/m-tok/hello-world/edit/main/README.md#attaching-igw-and-ng-to-subnets)
* [Route Tables](https://github.com/m-tok/hello-world/edit/main/README.md#route-tables)
* [Apply Terraform Configuration](https://github.com/m-tok/hello-world/edit/main/README.md#apply-terraform-configuration)
* [Verification of VPC ](https://github.com/m-tok/hello-world/edit/main/README.md#verification-of-vpc)
* [Deleting the VPC ](https://github.com/m-tok/hello-world/edit/main/README.md#deleting-the-vpc)


## Prerequisites and dependencies

There are no other dependencies apart from [Terraform](https://www.terraform.io/).

## Creating the VPC and Subnets

This code will create VPC , 3 private and 3 public subnets.


```
provider "aws" {
  region = var.region
}

variable "region" {
  default = "us-east-1"
}

resource "aws_vpc" "project1" {
  cidr_block = "10.0.0.0/16"
  tags       = local.common_tags
}

resource "aws_subnet" "public1" {
  vpc_id                  = aws_vpc.project1.id
  cidr_block              = "10.0.101.0/24"
  map_public_ip_on_launch = true
  availability_zone       = "us-east-1a"
  tags = {
    Name = "public1"
  }
}

resource "aws_subnet" "public2" {
  vpc_id                  = aws_vpc.project1.id
  cidr_block              = "10.0.102.0/24"
  map_public_ip_on_launch = true
  availability_zone       = "us-east-1b"
  tags = {
    Name = "public2"
  }
}

resource "aws_subnet" "public3" {
  vpc_id                  = aws_vpc.project1.id
  cidr_block              = "10.0.103.0/24"
  map_public_ip_on_launch = true
  availability_zone       = "us-east-1c"
  tags = {
    Name = "public3"
  }
}

resource "aws_subnet" "private1" {
  vpc_id                  = aws_vpc.project1.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true
  availability_zone       = "us-east-1a"
  tags = {
    Name = "private1"
  }
}

resource "aws_subnet" "private2" {
  vpc_id                  = aws_vpc.project1.id
  cidr_block              = "10.0.2.0/24"
  map_public_ip_on_launch = true
  availability_zone       = "us-east-1b"
  tags = {
    Name = "private2"
  }
}

resource "aws_subnet" "private3" {
  vpc_id                  = aws_vpc.project1.id
  cidr_block              = "10.0.3.0/24"
  map_public_ip_on_launch = true
  availability_zone       = "us-east-1c"
  tags = {
    Name = "private3"
  }
}

locals {
  common_tags = {
    Name = "project1"
    Team = "ProTeam"
    Env  = "Dev"
  }
}
```

## Attaching IGW and NG to Subnets

This code will attach Internet Getway to Public Subnets and Nat Getway to Private Subnets.

```
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.project1.id
  tags   = local.common_tags
}

resource "aws_eip" "nat" {
  vpc  = true
  tags = local.common_tags
}

resource "aws_nat_gateway" "nat_gw" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public1.id
  tags          = local.common_tags
}
```
## Route Tables

Route tables will be configured based on Gateway that is attached. It will forward traffic to Private or Public Subnets.

```
resource "aws_route_table" "public_route_table" {
  vpc_id = aws_vpc.project1.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
  tags = local.common_tags
}

resource "aws_route_table_association" "public1" {
  subnet_id      = aws_subnet.public1.id
  route_table_id = aws_route_table.public_route_table.id
}

resource "aws_route_table_association" "public2" {
  subnet_id      = aws_subnet.public2.id
  route_table_id = aws_route_table.public_route_table.id
}

resource "aws_route_table_association" "public3" {
  subnet_id      = aws_subnet.public3.id
  route_table_id = aws_route_table.public_route_table.id
}

resource "aws_route_table" "private_route_table" {
  vpc_id = aws_vpc.project1.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_nat_gateway.nat_gw.id
  }
  tags = local.common_tags
}

resource "aws_route_table_association" "private1" {
  subnet_id      = aws_subnet.private1.id
  route_table_id = aws_route_table.private_route_table.id
}

resource "aws_route_table_association" "private2" {
  subnet_id      = aws_subnet.private2.id
  route_table_id = aws_route_table.private_route_table.id
}

resource "aws_route_table_association" "private3" {
  subnet_id      = aws_subnet.private3.id
  route_table_id = aws_route_table.private_route_table.id
}
```
## Apply Terraform Configuration

To create the VPC:

```
terraform init
terraform apply 

```
## Verification of VPC 

To verify the VPC is running:

* Create ec2 instance in Amazon console manually
* Ping google.com

## Deleting the VPC

To delete the VPC,

* Destroy Terraform configuration:

```
terraform destroy 
```
