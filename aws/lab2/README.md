# Lab 12: Launching an EC2 Instance
## Steps

### 1. Create a VPC

1. Navigate to the AWS Management Console.
2. Search for "VPC" in the search bar and select it.
3. Click on "Create VPC" and provide the following details:
   - **Name tag**: `MyVPC`
   - **IPv4 CIDR block**: `10.0.0.0/16`
4. Click "Create VPC".

### 2. Create Subnets

1. In the VPC Dashboard, go to "Subnets" and click on "Create Subnet".
2. Provide the following details:
   - **VPC**: Select `MyVPC`
   - **Subnet Name**: `PublicSubnet`
   - **Availability Zone**: Choose an AZ (e.g., `us-east-1a`)
   - **IPv4 CIDR block**: `10.0.1.0/24`
3. Repeat the steps to create a private subnet:
   - **Subnet Name**: `PrivateSubnet`
   - **IPv4 CIDR block**: `10.0.2.0/24`

### 3. Create an Internet Gateway

1. In the VPC Dashboard, go to "Internet Gateways" and click "Create Internet Gateway".
2. Provide a name (e.g., `MyIGW`).
3. Attach the Internet Gateway to `MyVPC`:
   - Select the Internet Gateway, click "Actions", and choose "Attach to VPC".

### 4. Create a Route Table and Configure Routes

1. In the VPC Dashboard, go to "Route Tables" and click "Create Route Table".
2. Provide a name (e.g., `PublicRouteTable`) and select `MyVPC`.
3. Add a route to the route table:
   - **Destination**: `0.0.0.0/0`
   - **Target**: Select `MyIGW`
4. Associate the `PublicSubnet` with the route table:
   - Go to "Subnet Associations", click "Edit Subnet Associations", and select `PublicSubnet`.

### 5. Launch EC2 Instances

#### Public Instance (Bastion Host)

1. In the EC2 Dashboard, click "Launch Instance".
2. Select an AMI (e.g., Amazon Linux 2).
3. Choose an instance type (e.g., `t2.micro`).
4. In the Network settings:
   - Select `MyVPC` and `PublicSubnet`.
   - Enable "Auto-assign Public IP".
5. Create or select a security group that allows SSH from your IP (e.g., `0.0.0.0/0`).
6. Launch the instance.

#### Private Instance

1. Repeat the steps for the public instance, but:
   - Select `PrivateSubnet`.
   - Do **not** enable "Auto-assign Public IP".
2. Create a security group that allows SSH only from the public instance's private IP.

### 6. Configure Security Group for the Private Instance

1. Go to "Security Groups" in the EC2 Dashboard.
2. Select the security group for the private instance and click "Edit Inbound Rules".
3. Add a rule:
   - **Type**: SSH
   - **Source**: The private IP of the public instance.

### 7. Access the Private Instance via the Public Instance (Bastion Host)

1. Open a terminal on your local machine.
2. SSH into the public instance using the following command:
   ```bash
   ssh -i my-key.pem ec2-user@3.235.158.7
   ```
3. From the public instance, SSH into the private instance:
   ```bash
   ssh -i my-key.pem ec2-user@10.0.2.233
   ```

