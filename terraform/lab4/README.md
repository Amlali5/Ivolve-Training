### Terraform Variables and Loops
## Project Structure

The project consists of the following files:

- **main.tf**: Contains the main Terraform configuration.
- **variables.tf**: Declares input variables for the configuration.
- **output.tf**: Contains the output definitions for public and private IPs of the EC2 instances.
- **terraform.tfvars**: Specifies values for the variables.
  


#### main.tf
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    time = {
      source  = "hashicorp/time"
      version = "~> 0.7"
    }
  }
}

provider "aws" {
  region = var.region
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "subnet1" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.0.0/24"
  map_public_ip_on_launch = true
}

resource "aws_subnet" "subnet2" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
  map_public_ip_on_launch = true
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
}

resource "aws_route_table" "public_route_table" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
}

resource "aws_route_table_association" "public_subnet_association" {
  subnet_id      = aws_subnet.subnet1.id
  route_table_id = aws_route_table.public_route_table.id
}

resource "aws_security_group" "nginx_sg" {
  vpc_id = aws_vpc.main.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_security_group" "apache_sg" {
  vpc_id = aws_vpc.main.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_eip" "nat_eip" {
  domain = "vpc"
}

resource "aws_nat_gateway" "nat_gateway" {
  allocation_id = aws_eip.nat_eip.id
  subnet_id     = aws_subnet.subnet1.id
}

resource "aws_route_table" "private_route_table" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat_gateway.id
  }
}

resource "aws_route_table_association" "private_subnet_association" {
  subnet_id      = aws_subnet.subnet2.id
  route_table_id = aws_route_table.private_route_table.id
}

resource "aws_key_pair" "my_key" {
  key_name   = "my-key"
  public_key = file("./id_rsa.pub")
}

resource "time_sleep" "wait_for_public_ip" {
  create_duration = "2m"
}

resource "time_sleep" "wait_for_instance" {
  create_duration = "2m"
}

resource "aws_instance" "nginx" {
  ami           = var.ami_id
  instance_type = var.instance_type
  subnet_id     = aws_subnet.subnet1.id
  vpc_security_group_ids = [aws_security_group.nginx_sg.id]
  key_name      = aws_key_pair.my_key.key_name

  depends_on = [aws_internet_gateway.igw, aws_nat_gateway.nat_gateway, time_sleep.wait_for_public_ip, time_sleep.wait_for_instance]

  connection {
    type        = "ssh"
    user        = "ec2-user"  
    private_key = file("./id_rsa")
    host        = self.public_ip
    timeout     = "10m"
  }

  provisioner "remote-exec" {
    inline = [
      "sudo yum update -y",  
      "sudo yum install -y nginx"  
    ]
  }
}

resource "aws_instance" "apache" {
  ami           = var.ami_id
  instance_type = var.instance_type
  subnet_id     = aws_subnet.subnet2.id
  vpc_security_group_ids = [aws_security_group.apache_sg.id]
  key_name      = aws_key_pair.my_key.key_name

  depends_on = [aws_internet_gateway.igw, aws_nat_gateway.nat_gateway, time_sleep.wait_for_public_ip, time_sleep.wait_for_instance]

  user_data = <<-EOF
              #!/bin/bash
              sudo yum update -y
              sudo yum install -y httpd
              sudo systemctl start httpd
              sudo systemctl enable httpd
              EOF
}
```

#### outputs.tf
```hcl
output "nginx_public_ip" {
  description = "The public IP address of the Nginx instance"
  value       = aws_instance.nginx.public_ip
}

output "nginx_private_ip" {
  description = "The private IP address of the Nginx instance"
  value       = aws_instance.nginx.private_ip
}

output "apache_public_ip" {
  description = "The public IP address of the Apache instance"
  value       = aws_instance.apache.public_ip
}

output "apache_private_ip" {
  description = "The private IP address of the Apache instance"
  value       = aws_instance.apache.private_ip
}
```

#### variables.tf
```hcl
variable "region" {
  description = "The AWS region to create resources in"
  type        = string
}

variable "ami_id" {
  description = "The AMI ID for the EC2 instances"
  type        = string
}

variable "instance_type" {
  description = "The instance type for the EC2 instances"
  type        = string
}
```
### terraform.tfvars
```hcl
region             = "us-east-1"
ami_id             = "ami-05b10e08d247fb927"
instance_type      = "t2.micro"
```
## Steps to Deploy
1. Initialize Terraform:
   ```bash
   terraform init
   ```

2. Validate the configuration:
   ```bash
   terraform validate
   ```

3. Plan the infrastructure:
   ```bash
   terraform plan
   ```

4. Apply the configuration:
   ```bash
   terraform apply
   ```

5. Note the output for public and private IPs of the EC2 instances.

## Outputs
- **web_instance_public_ips**: Public IP addresses of the EC2 instances.
- **web_instance_private_ips**: Private IP addresses of the EC2 instances.








