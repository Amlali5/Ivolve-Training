
# Terraform Lab: Creating a VPC and EC2 Instances with Reusable Modules

This project demonstrates how to use Terraform to create a VPC and deploy EC2 instances in AWS using reusable modules. The setup includes creating a VPC, subnets, and two EC2 instances with Nginx installed.

---

## **Project Structure**

```
.
├── main.tf
├── variables.tf
├── outputs.tf
└── modules
    ├── network
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    └── server
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

---

## **File Descriptions**

1. **`main.tf`**: The main configuration file that calls the `network` and `server` modules to create AWS resources.
2. **`variables.tf`**: Defines the input variables for the project.
3. **`outputs.tf`**: Displays the outputs after running Terraform (e.g., public IPs of EC2 instances).
4. **`modules/network`**: Contains code for creating the VPC, subnets, Internet Gateway, and routing table.
5. **`modules/server`**: Contains code for creating EC2 instances and configuring them with Nginx.

---

## **Project Details**

### **Root Files**

#### **`main.tf`**

This file orchestrates the creation of the infrastructure by calling the `network` and `server` modules.

```hcl
provider "aws" {
  region = "us-east-1"
}

module "network" {
  source              = "./modules/network"
  vpc_cidr            = "10.0.0.0/16"
  public_subnet_cidrs = ["10.0.1.0/24", "10.0.2.0/24"]
  availability_zones  = ["us-east-1a", "us-east-1b"]
}

module "server1" {
  source        = "./modules/server"
  ami_id        = "ami-05b10e08d247fb927"  
  instance_type = "t2.micro"
  subnet_id     = module.network.public_subnet_ids[0]
  vpc_id        = module.network.vpc_id
  key_name      = "my-key"
}

module "server2" {
  source        = "./modules/server"
  ami_id        = "ami-05b10e08d247fb927"  
  instance_type = "t2.micro"
  subnet_id     = module.network.public_subnet_ids[1]
  vpc_id        = module.network.vpc_id
  key_name      = "my-key"
}
```

#### **`variables.tf`**

Defines input variables for the root configuration.

```hcl
variable "vpc_cidr" {
  description = "CIDR block for the VPC"
  type        = string
}

variable "public_subnet_cidrs" {
  description = "List of CIDR blocks for public subnets"
  type        = list(string)
}

variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
}
```

#### **`outputs.tf`**

Displays outputs from the project.

```hcl
output "vpc_id" {
  value = module.network.vpc_id
}

output "public_subnet_ids" {
  value = module.network.public_subnet_ids
}

output "server1_public_ip" {
  value = module.server1.instance_public_ip
}

output "server2_public_ip" {
  value = module.server2.instance_public_ip
}
```

### **Modules**

#### **1. Network Module**

This module creates the VPC, subnets, Internet Gateway, and routing table.

- **`main.tf`**:

```hcl
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "main-vpc"
  }
}

resource "aws_internet_gateway" "gw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "main-igw"
  }
}

resource "aws_subnet" "public" {
  count = length(var.public_subnet_cidrs)

  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidrs[count.index]
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "public-subnet-${count.index + 1}"
  }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.gw.id
  }

  tags = {
    Name = "public-route-table"
  }
}

resource "aws_route_table_association" "public" {
  count = length(aws_subnet.public)

  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}
```
- **`variables.tf`**:

```hcl
variable "vpc_cidr" {
  description = "CIDR block for the VPC"
  type        = string
}

variable "public_subnet_cidrs" {
  description = "List of CIDR blocks for public subnets"
  type        = list(string)
}

variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
}
```

- **`outputs.tf`**:

```hcl
output "vpc_id" {
  value = aws_vpc.main.id
}

output "public_subnet_ids" {
  value = aws_subnet.public[*].id
}
```

---

#### **2. Server Module**

This module creates the EC2 instances with Nginx installed.

- **`main.tf`**:

```hcl
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type
  subnet_id     = var.subnet_id
  key_name      = var.key_name
  user_data     = <<-EOF
                  #!/bin/bash
                  sudo yum update -y
                  sudo yum install -y nginx
                  sudo systemctl start nginx
                  EOF

  tags = {
    Name = "web-server"
  }
}

resource "aws_security_group" "web" {
  vpc_id = var.vpc_id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "web-sg"
  }
}
```
- **`variables.tf`**:

```hcl
variable "ami_id" {
  description = "AMI ID for the EC2 instance"
  type        = string
}

variable "instance_type" {
  description = "Instance type for the EC2 instance"
  type        = string
}

variable "subnet_id" {
  description = "Subnet ID for the EC2 instance"
  type        = string
}

variable "vpc_id" {
  description = "VPC ID for the security group"
  type        = string
}

variable "key_name" {
  description = "Key pair name for SSH access"
  type        = string
}
```

- **`outputs.tf`**:

```hcl
output "instance_public_ip" {
  value = aws_instance.web.public_ip
}
```


---

## How to Run the Project

1. **Initialize Terraform**:
   ```bash
   terraform init
   ```

2. **Validate Configuration**:
   ```bash
   terraform validate
   ```

3. **View the Execution Plan**:
   ```bash
   terraform plan
   ```

4. **Apply the Configuration**:
   ```bash
   terraform apply
   ```

---

## Expected Outputs

Once you run `terraform apply`, you should see outputs similar to the following:

```plaintext
Outputs:

server1_public_ip = "54.161.193.189"
server2_public_ip = "18.209.18.72"
public_subnet_ids = [
  "subnet-08e4c629d9149d4f9",
  "subnet-034d1c1e3376eaa0b"
]
```

Use the public IPs to access the EC2 instances via SSH or a web browser to see the Nginx welcome page.

---

## Results

- **VPC**: A VPC with CIDR block `10.0.0.0/16` is created.
- **Subnets**: Two public subnets in different availability zones.
- **Internet Gateway**: An Internet Gateway attached to the VPC.
- **EC2 Instances**: Two instances with Nginx installed, accessible over HTTP.


