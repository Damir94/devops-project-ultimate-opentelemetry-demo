### Docker Installation Guide (Ubuntu)
This README provides a clear, step-by-step guide to installing Docker Engine and related components on an Ubuntu system using the official Docker repository.

Prerequisites
- Ubuntu-based system (ec2 t2.large instance)
- A user with sudo privileges
- Internet connectivity

Step 1: Update the Package Index
```bash
sudo apt update
```
Step 2: Install Required Dependencies
```bash
sudo apt install -y ca-certificates curl
```
Step 3: Add Docker’s Official GPG Key
```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
Step 4: Add the Docker Repository
```bash
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```
```bash
sudo apt update
```
Step 5: Install Docker Engine
```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
Step 6: Verify Docker Installation
```bash
sudo systemctl start docker
sudo systemctl status docker
sudo docker run hello-world
```
<img width="1032" height="278" alt="Screenshot 2026-01-13 at 11 30 45 AM" src="https://github.com/user-attachments/assets/822e2b3d-909e-43a9-8668-6d4af00024d4" />

Step 7: Run Docker Without sudo (Optional but Recommended)
```bash
sudo usermod -aG docker ubuntu
```
Important: Log out and log back in for the group changes to take effect. After re-login, you should be able to run Docker commands without sudo.

### Kubernetes CLI (kubectl) Installation
Step 1: Download kubectl Binary and Checksum
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
```
Step 2: Install kubectl
```bash
kubectl version --client
```
<img width="585" height="98" alt="Screenshot 2026-01-13 at 11 38 09 AM" src="https://github.com/user-attachments/assets/b4035a28-99db-48fa-aabc-9e3e890c9354" />

### Terraform Installation
Step 1: Install Required Packages
```bash
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
```
Step 2: Add HashiCorp GPG Key
```bash
wget -O- https://apt.releases.hashicorp.com/gpg | \
  gpg --dearmor | \
  sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null
```
Verify the key fingerprint:
```bash
gpg --no-default-keyring \
  --keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg \
  --fingerprint
```
Step 3: Add HashiCorp Repository
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
```
Update package index and install Terraform:
```bash
sudo apt update
sudo apt-get install -y terraform
```
Verify Terraform installation:
```bash
terraform --version
```
<img width="525" height="96" alt="Screenshot 2026-01-13 at 11 40 58 AM" src="https://github.com/user-attachments/assets/d678be0b-77b6-4f15-b640-a12dab3d5840" />

### Run the Project Locally
Clone the project repository and start the application using Docker Compose.
```bash
git clone https://github.com/iam-veeramalla/ultimate-devops-project-demo.git
cd ultimate-devops-project-demo
ls
docker compose up
```
Check disk usage (useful before and after resizing volumes):
```bash
df -h
```
### Resize an AWS EBS Volume (Ubuntu EC2)
This section explains how to resize an EBS root volume and extend the filesystem on an EC2 instance.

Step 1: Modify the Volume in AWS Console
  - Open EC2 → Volumes
  - Select the volume attached to your instance
  - Click Actions → Modify volume
  - Change the size to 30 GB and click Modify

Step 2: Verify the New Volume Size
```bash
lsblk
```
You will notice:
  - /dev/xvda → resized to 30 GB
  - /dev/xvda1 → still showing the old size (e.g., 7 GB)

Step 3: Install Cloud Guest Utilities
```bash
sudo apt install -y cloud-guest-utils
```
Step 4: Extend the Partition
```bash
sudo growpart /dev/xvda 1
lsblk
df -h
```
Step 5: Resize the Filesystem
```bash
sudo resize2fs /dev/xvda1
df -h
```
At this point, the root filesystem should reflect the new 30 GB size.

Step 6: Restart the Application
```bash
docker compose up -d
```
### AWS Security Groups and Application Access
To access the application running on the EC2 instance:
  - Open the Security Group attached to the EC2 instance
  - Go to Inbound rules → Edit inbound rules
  - Allow All Traffic (for demo/testing purposes only)
  - Save the rules

### Access the Application
```bash
http://<EC2-PUBLIC-IP>:8080
```
You should now be able to access the application successfully.

### How Terraform Authenticates with AWS
  - You create AWS access keys (Access Key + Secret Key).
  - You configure these keys using the AWS CLI.
  - AWS CLI stores the credentials locally.
  - Terraform reads these credentials and uses them to make AWS API calls.

### Configure AWS CLI
- Run the following command:
```bash
aws configure
```
You will be prompted to enter:
  - AWS Access Key ID
  - AWS Secret Access Key
  - Default region (e.g., us-east-1)
  - Default output format (press Enter to keep default)

### VPC Module for EKS Cluster (Terraform)
In this project, we create a reusable Terraform VPC module and use it to deploy an EKS cluster in AWS.

### main.tf
```hcl
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name                                           = "${var.cluster_name}-vpc"
    "kubernetes.io/cluster/${var.cluster_name}"    = "shared"
  }
}

resource "aws_subnet" "private" {
  count             = length(var.private_subnet_cidrs)
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_subnet_cidrs[count.index]
  availability_zone = var.availability_zones[count.index]

  tags = {
    Name                                           = "${var.cluster_name}-private-${count.index + 1}"
    "kubernetes.io/cluster/${var.cluster_name}"    = "shared"
    "kubernetes.io/role/internal-elb"              = "1"
  }
}

resource "aws_subnet" "public" {
  count             = length(var.public_subnet_cidrs)
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.public_subnet_cidrs[count.index]
  availability_zone = var.availability_zones[count.index]

  map_public_ip_on_launch = true

  tags = {
    Name                                           = "${var.cluster_name}-public-${count.index + 1}"
    "kubernetes.io/cluster/${var.cluster_name}"    = "shared"
    "kubernetes.io/role/elb"                       = "1"
  }
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "${var.cluster_name}-igw"
  }
}

resource "aws_eip" "nat" {
  count = length(var.public_subnet_cidrs)
  domain = "vpc"

  tags = {
    Name = "${var.cluster_name}-nat-${count.index + 1}"
  }
}

resource "aws_nat_gateway" "main" {
  count         = length(var.public_subnet_cidrs)
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id

  tags = {
    Name = "${var.cluster_name}-nat-${count.index + 1}"
  }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = {
    Name = "${var.cluster_name}-public"
  }
}

resource "aws_route_table" "private" {
  count  = length(var.private_subnet_cidrs)
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main[count.index].id
  }

  tags = {
    Name = "${var.cluster_name}-private-${count.index + 1}"
  }
}

resource "aws_route_table_association" "private" {
  count          = length(var.private_subnet_cidrs)
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private[count.index].id
}

resource "aws_route_table_association" "public" {
  count          = length(var.public_subnet_cidrs)
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}
```
###Big Picture (One-Line Summary)
  - VPC → Network
  - Public Subnets → Internet-facing resource
  - Private Subnets → EKS tasks / backend
  - Internet Gateway → Incoming/outgoing internet
  - NAT Gateway → Secure outbound internet
  - Route Tables → Control traffic flow

### variables.tf
```hcl
variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
}

variable "availability_zones" {
  description = "Availability zones"
  type        = list(string)
}

variable "private_subnet_cidrs" {
  description = "CIDR blocks for private subnets"
  type        = list(string)
}

variable "public_subnet_cidrs" {
  description = "CIDR blocks for public subnets"
  type        = list(string)
}

variable "cluster_name" {
  description = "Name of the EKS cluster"
  type        = string
}
```
### outputs.tf
```hcl
output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "private_subnet_ids" {
  description = "Private subnet IDs"
  value       = aws_subnet.private[*].id
}

output "public_subnet_ids" {
  description = "Public subnet IDs"
  value       = aws_subnet.public[*].id
}
```

### EKS Module Using Terraform
In this module, we create an EKS (EKS-style Kubernetes) cluster using Terraform.

### main.tf
```hcl
resource "aws_iam_role" "cluster" {
  name = "${var.cluster_name}-cluster-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "eks.amazonaws.com"
      }
    }]
  })
}

resource "aws_iam_role_policy_attachment" "cluster_policy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"
  role       = aws_iam_role.cluster.name
}

resource "aws_eks_cluster" "main" {
  name     = var.cluster_name
  version  = var.cluster_version
  role_arn = aws_iam_role.cluster.arn

  vpc_config {
    subnet_ids = var.subnet_ids
  }

  depends_on = [
    aws_iam_role_policy_attachment.cluster_policy
  ]
}

resource "aws_iam_role" "node" {
  name = "${var.cluster_name}-node-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "ec2.amazonaws.com"
      }
    }]
  })
}

resource "aws_iam_role_policy_attachment" "node_policy" {
  for_each = toset([
    "arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy",
    "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy",
    "arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly"
  ])

  policy_arn = each.value
  role       = aws_iam_role.node.name
}

resource "aws_eks_node_group" "main" {
  for_each = var.node_groups

  cluster_name    = aws_eks_cluster.main.name
  node_group_name = each.key
  node_role_arn   = aws_iam_role.node.arn
  subnet_ids      = var.subnet_ids

  instance_types = each.value.instance_types
  capacity_type  = each.value.capacity_type

  scaling_config {
    desired_size = each.value.scaling_config.desired_size
    max_size     = each.value.scaling_config.max_size
    min_size     = each.value.scaling_config.min_size
  }

  depends_on = [
    aws_iam_role_policy_attachment.node_policy
  ]
}
```
### variables.tf
```hcl
variable "cluster_name" {
  description = "Name of the EKS cluster"
  type        = string
}

variable "cluster_version" {
  description = "Kubernetes version"
  type        = string
}

variable "vpc_id" {
  description = "VPC ID"
  type        = string
}

variable "subnet_ids" {
  description = "Subnet IDs"
  type        = list(string)
}

variable "node_groups" {
  description = "EKS node group configuration"
  type = map(object({
    instance_types = list(string)
    capacity_type  = string
    scaling_config = object({
      desired_size = number
      max_size     = number
      min_size     = number
    })
  }))
}
```
### outputs.tf
```hcl
output "cluster_endpoint" {
  description = "EKS cluster endpoint"
  value       = aws_eks_cluster.main.endpoint
}

output "cluster_name" {
  description = "EKS cluster name"
  value       = aws_eks_cluster.main.name
}
```

### Final Flow (Very Important)
  - Create cluster IAM role
  - Attach cluster policy
  - Create EKS cluster (control plane)
  - Create node IAM role
  - Attach node policies
  - Create node groups (worker nodes)

### One-Line Summary
This Terraform code creates a complete EKS cluster by setting up IAM roles, attaching required policies, creating the control plane, and launching worker nodes with proper networking and permissions.
