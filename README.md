This is a exercise used to assess your approach to development and use of infrastructure as code.

In this example we have a simple VPC network. To run this configuration you can use the commands below:

```bash
terraform init
terraform plan -var-file=default.tfvars
terraform apply -var-file=default.tfvars
```

This configuration needs to be adapted to become a more robust network architecture.

1. The AWS us-east-1 region has more than 2 availability zones, and we'd like to make use of one more for our network configuration. Modify the configuration to span 3 AZs.

2. To improve security and network separation we'd like to have private subnets that are not accessible to the internet. Modify the configuration for subnet b to make it private. Note that private subnets should still be able to egress to the internet.

3. To further improve security we'd like to have a database subnet that is not addressable from the internet, and also has no access to egress to the internet. Modify the configuration for the subnet you created in step 1 to make it only accessible from the local internal network.


## default.tfvars
```t
region="us-east-1"
vpc_cidr = "10.10.10.0/24"
subnet_cidr_a = "10.10.10.0/27"
subnet_cidr_b = "10.10.10.32/27"
```

## main.tf
```t
provider "aws" {
  region = var.region
}

resource "aws_vpc" "vpc" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
}

resource "aws_subnet" "subnet_a" {
  vpc_id            = aws_vpc.vpc.id
  cidr_block        = var.subnet_cidr_a
  availability_zone = "${var.region}a"
}

resource "aws_subnet" "subnet_b" {
  vpc_id            = aws_vpc.vpc.id
  cidr_block        = var.subnet_cidr_b
  availability_zone = "${var.region}b"
}

resource "aws_route_table" "subnet_route_table" {
  vpc_id = aws_vpc.vpc.id
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.vpc.id
}

resource "aws_route" "subnet_route" {
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.igw.id
  route_table_id         = aws_route_table.subnet_route_table.id
}

resource "aws_route_table_association" "subnet_a_route_table_association" {
  subnet_id      = aws_subnet.subnet_a.id
  route_table_id = aws_route_table.subnet_route_table.id
}

resource "aws_route_table_association" "subnet_b_route_table_association" {
  subnet_id      = aws_subnet.subnet_b.id
  route_table_id = aws_route_table.subnet_route_table.id
}
```

## variables.tf
```t
variable "region" {}

variable "vpc_cidr" {}

variable "subnet_cidr_a" {}

variable "subnet_cidr_b" {}
```