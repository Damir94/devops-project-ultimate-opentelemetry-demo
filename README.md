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
Step 2: Step 2: Install kubectl
```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```
Step 3: Verify Installation
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

<img width="1882" height="963" alt="Screenshot 2026-01-15 at 11 44 31 AM" src="https://github.com/user-attachments/assets/1c9ac7f5-527d-4450-a454-bc7855af7c65" />

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
### Terraform Backend Setup (S3 + DynamoDB)
This guide explains how to create a Terraform backend using AWS S3 for state storage and DynamoDB for state locking.
Sample Terraform Code (main.tf)
```hcl
provider "aws" {
  region = "us-east-1"
}
resource "aws_s3_bucket" "terraform_state_bucket" {
  bucket = "terraform-demo-x-state-s3-bucket-damir"

  lifecycle {
    prevent_destroy = false
  }
}
resource "aws_dynamodb_table" "terraform_state_lock" {
  name         = "terraform-eks-state-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  attribute {
    name = "LockID"
    type = "S"
  }
}
```
### Terraform Commands to Run
From inside the backend folder:
```bash
terraform init
terraform plan
terraform apply
```
<img width="1010" height="303" alt="Screenshot 2026-01-17 at 1 58 50 PM" src="https://github.com/user-attachments/assets/0eb2e015-bee5-4649-8585-453e4e9de19a" />

<img width="1523" height="242" alt="Screenshot 2026-01-17 at 1 59 07 PM" src="https://github.com/user-attachments/assets/0018a958-cb06-4c4f-a4c2-9636d325f435" />

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
<img width="1568" height="650" alt="Screenshot 2026-01-17 at 2 10 48 PM" src="https://github.com/user-attachments/assets/69824ec3-7a56-4c59-b3db-4ab32663e000" />

### One-Line Summary
This Terraform code creates a complete EKS cluster by setting up IAM roles, attaching required policies, creating the control plane, and launching worker nodes with proper networking and permissions.

### In this lecture we will learn how to create VPC and EKS using the modules that we wrote in the previous lecture. Because we already wrote the modules.

### main.tf
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    bucket         = "demo-terraform-eks-state-s3-bucket"
    key            = "terraform.tfstate"
    region         = "us-west-2"
    dynamodb_table = "terraform-eks-state-locks"
    encrypt        = true
  }
}

provider "aws" {
  region = var.region
}

module "vpc" {
  source = "./modules/vpc"

  vpc_cidr             = var.vpc_cidr
  availability_zones   = var.availability_zones
  private_subnet_cidrs = var.private_subnet_cidrs
  public_subnet_cidrs  = var.public_subnet_cidrs
  cluster_name         = var.cluster_name
}

module "eks" {
  source = "./modules/eks"

  cluster_name    = var.cluster_name
  cluster_version = var.cluster_version
  vpc_id          = module.vpc.vpc_id
  subnet_ids      = module.vpc.private_subnet_ids
  node_groups     = var.node_groups
}
```

### variables.tf
```hcl
variable "region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "Availability zones"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

variable "private_subnet_cidrs" {
  description = "CIDR blocks for private subnets"
  type        = list(string)
  default     = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
}

variable "public_subnet_cidrs" {
  description = "CIDR blocks for public subnets"
  type        = list(string)
  default     = ["10.0.4.0/24", "10.0.5.0/24", "10.0.6.0/24"]
}

variable "cluster_name" {
  description = "Name of the EKS cluster"
  type        = string
  default     = "my-eks-cluster"
}

variable "cluster_version" {
  description = "Kubernetes version"
  type        = string
  default     = "1.30"
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
  default = {
    general = {
      instance_types = ["t3.medium"]
      capacity_type  = "ON_DEMAND"
      scaling_config = {
        desired_size = 2
        max_size     = 4
        min_size     = 1
      }
    }
  }
}
```
### outputs.tf
```hcl
output "cluster_endpoint" {
  description = "EKS cluster endpoint"
  value       = module.eks.cluster_endpoint
}

output "cluster_name" {
  description = "EKS cluster name"
  value       = module.eks.cluster_name
}

output "vpc_id" {
  description = "VPC ID"
  value       = module.vpc.vpc_id
}
```
### Terraform Commands to Run
```bash
terraform init
terraform plan
terraform apply
```
<img width="1237" height="274" alt="Screenshot 2026-01-17 at 2 11 19 PM" src="https://github.com/user-attachments/assets/752dea95-4a24-4a29-8fcc-37ad94084d6b" />

### Connecting kubectl to an Amazon EKS Cluster
By default, kubectl does not know which Kubernetes cluster to connect to.
To communicate with a cluster (like EKS), kubectl relies on a configuration file called kubeconfig.

### aws cli is installed
Update your system
```bash
sudo apt update
```
Download AWS CLI
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```
Unzip the installer
```bash
unzip awscliv2.zip
```
Install AWS CLI
```bash
sudo ./aws/install
```
Verify Installation
```bash
aws --version
```
### Update kubeconfig with EKS Cluster Info
Run the following command:
```bash
 aws eks update-kubeconfig --name <EKS_CLUSTER_NAME> --region <AWS_REGION>
```
### Verify the Connection
Check cluster details: 
```bash
kubectl config view
kubectl config current-context
kubectl get nodes
```
If nodes are listed, kubectl is successfully connected to your EKS cluster.

<img width="770" height="108" alt="Screenshot 2026-01-17 at 2 19 34 PM" src="https://github.com/user-attachments/assets/57254485-68a9-4781-9cdf-2eff9773ee38" />

### Deploying the Project to an EKS Cluster
This guide explains how to deploy the project’s microservices to an EKS (Kubernetes) cluster, verify the deployment, and understand how services communicate with each other.
1. Verify Cluster Context
```bash
kubectl config current-context
```
2. Verify a Clean Namespace
```bash
kubectl get all
```
3. Navigate to the Kubernetes manifests directory:
```bash
cd ultimate-devops-project/kubernetes
```

### Step 1: Create the Service Account
All microservices in this project use a custom service account instead of the default one
Apply Service Account
```bash
kubectl apply -f serviceaccount.yaml
```
Verify
```bash
kubectl get sa
```
You should see: opentelemetry-demo
<img width="764" height="102" alt="Screenshot 2026-01-17 at 2 24 04 PM" src="https://github.com/user-attachments/assets/0c7e0f5c-1ca0-46db-8dcf-e81cee7368d2" />

### Step 2: Deploy All Microservices
  -  Option 1: Apply Individual Manifests
     You can apply manifests folder by folder, but this is time-consuming.
  - Option 2: Apply Combined Deployment (Recommended)
     A single merged YAML file is provided that contains:
       - All services
       - All deployments
       - ~20 microservices combined
```bash
kubectl apply -f complete-deploy.yaml
```
This file:
  - Creates services first
  - Then creates deployments
### Step 3: Verify Deployment and Services Status
```bash
kubectl get pods
kubectl get svc
```
<img width="959" height="413" alt="Screenshot 2026-01-17 at 2 26 06 PM" src="https://github.com/user-attachments/assets/9f087407-18d2-499d-a4b4-c368e903c5d7" />

<img width="1033" height="377" alt="Screenshot 2026-01-17 at 2 26 41 PM" src="https://github.com/user-attachments/assets/bb5dc2dd-609b-4138-84ba-b4ce9d939ea7" />

## Accessing the Application
### Why the App Is Not Accessible Yet
If you check the frontend service:
```bash
kubectl get svc | grep opentelemetry-demo-frontend
```
You will see: Private IPs (e.g., 172.x.x.x)
These IPs are only accessible inside the VPC

That means:
  - You cannot access the application from your browser yet
  - The EKS cluster runs inside a private VPC network

### Exposing the Application Using LoadBalancer Service
By default, Kubernetes services are created as ClusterIP, which means they are accessible only inside the cluster.
To access the application from the internet, we must expose the frontend using a LoadBalancer service.

### Step 1: Identify the Frontend Service. First, find the frontend proxy service:
```bash
kubectl get svc | grep opentelemetry-demo-frontendproxy 
```
### Step 2: Edit the Service Type
```bash
kubectl edit svc opentelemetry-demo-frontendproxy 
```
At the bottom of the file, change: type: ClusterIP to type: LoadBalancer
<img width="571" height="143" alt="Screenshot 2026-01-17 at 2 29 56 PM" src="https://github.com/user-attachments/assets/b5cbca31-6ab6-4a74-aa78-9831b7a4c745" />

### Step 3: Verify the Load Balancer Check the service status: kubectl get svc
```bash
kubectl get svc
```
Copy the EXTERNAL-IP / FQDN and access it in the browser: 

<img width="1354" height="972" alt="Screenshot 2026-01-17 at 2 32 21 PM" src="https://github.com/user-attachments/assets/a675b502-356f-4e8f-9842-f47837068ff7" />

### Installing AWS ALB Ingress Controller on EKS
This guide explains how to install the AWS ALB (AWS Load Balancer) Ingress Controller on an EKS cluster, along with the required IAM setup.
#### Prerequisites
Before starting, make sure:

### eksctl is installed: eksctl version
Install
```bash
sudo apt update && sudo apt install -y curl
```
Download the latest eksctl binary
```bash
curl -sLO https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz
```
Extract the binary
```bash
tar -xzf eksctl_Linux_amd64.tar.gz
```
Move it to a system path
```bash
sudo mv eksctl /usr/local/bin
```
Verify installation
```bash
eksctl version
```
### helm is installed
```bash
sudo apt update
```
Download and install Helm
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```
Verify installation
```bash
helm version
```
Set the cluster name:
```bash
export cluster_name=demo-cluster
```
1. Setup OIDC Connector
```bash
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
```
2. Check if OIDC provider already exists
```bash
aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4
```
If OIDC is NOT configured, run this
```bash
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```
3. Download IAM Policy for ALB Controller
```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
```
4. Create the IAM Policy in AWS
```bash
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```
### If you get this error: An error occurred (EntityAlreadyExists) when calling the CreatePolicy operation: A policy called AWSLoadBalancerControllerIAMPolicy already exists. Duplicate names are not allowed.
### What you should do instead
#### Get the policy ARN
You need the ARN, not to recreate the policy.
```bash
aws iam list-policies \
  --query "Policies[?PolicyName=='AWSLoadBalancerControllerIAMPolicy'].Arn" \
  --output text
```
#### Attach the policy to your IAM role
(Usually the role used by the AWS Load Balancer Controller)
```bash
aws iam attach-role-policy \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --policy-arn arn:aws:iam::123456789012:policy/AWSLoadBalancerControllerIAMPolicy
```
Replace:  
  - AmazonEKSLoadBalancerControllerRole with your actual role name
  - Account ID if different
5. Create IAM Role + Kubernetes Service Account
```bash
eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
--attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```
6. Deploy the ALB Controller using Helm
```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
```
7. Install the ALB Controller
```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \            
  -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>
```
8. Verify the Controller is Running
```bash
kubectl get pods -n kube-system
```
<img width="774" height="212" alt="Screenshot 2026-01-18 at 2 54 37 PM" src="https://github.com/user-attachments/assets/8297076a-7042-4ea7-a3d8-3790ba15aa7a" />

9. Check logs of the controller
```bash
kubectl logs pod-name -n kube-system
```
10. Remove Existing LoadBalancer Service
Change service type back to NodePort or ClusterIP
```bash
kubectl edit svc opentelemetry-demo-frontendproxy
```
type: NodePort (or ClusterIP)
This automatically deletes the AWS Load Balancer created earlier.
11. Create Ingress Resource
```bash
cd ultimate-devops-project-demo/kubernetes/frontendproxy
```
12. Create a new file: ingress.yaml
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: frontend-proxy
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  ingressClassName: alb
  rules:
    - host: example.com
      http:
        paths:
          - path: "/"
            pathType: Prefix
            backend:
              service:
                name: opentelemetry-demo-frontendproxy
                port:
                  number: 8080
```
13. Apply the Ingress
```bash
kubectl apply -f ingress.yaml
kubectl get ingress
```
<img width="1076" height="295" alt="Screenshot 2026-01-18 at 2 56 40 PM" src="https://github.com/user-attachments/assets/804849a5-3624-4228-84e8-b5ad3da57654" />

14. Verify ALB Creation
  - Go to AWS Console → EC2 → Load Balancers
  - Wait until the load balancer status becomes Active
15. Troubleshooting (If ADDRESS Is Missing)
```bash
kubectl get pods -n kube-system
kubectl logs -n kube-system <alb-controller-pod-name>
```
### Access Behavior (Important Concept)
  - Because this ingress uses host-based routing:
  - Accessing the ALB IP address will NOT work
  - Accessing the ALB DNS name directly will NOT work
  - Accessing via the configured host (example.com) WILL work

16. First run nslookup to identify loadbalancer IP address
```bash
nslookup ALB-DNS-name
``` 
18. Configure Local DNS (For Testing Only) on local computer.
```bash
sudo vi /etc/hosts
```
```bash
<ALB-IP-ADDRESS>   example.com
```
<img width="1817" height="893" alt="Screenshot 2026-01-18 at 10 51 36 AM" src="https://github.com/user-attachments/assets/0a4927e2-5c18-496b-b8e5-c7b1813dbeea" />
    
### Product Catalog Service – CI Pipeline
This repository uses GitHub Actions to run a Continuous Integration (CI) pipeline for the Product Catalog Service (a Go application).
The pipeline automatically:
  - Builds the application
  - Runs unit tests
  - Checks code quality
  - Builds and pushes a Docker image
  - Updates the Kubernetes deployment with the new image

### What is this workflow?
This is a CI (Continuous Integration) pipeline for the Product Catalog Service, which is a Go application.

It automatically:
  - Builds the Go app
  - Runs unit tests
  - Checks code quality
  - Builds and pushes a Docker image
  - Updates a Kubernetes deployment file with the new image

### ci.yaml
```yaml
# CI for Product Catalog Service

name: product-catalog-ci

on: 
    pull_request:
        branches:
        - main

jobs:
    build:
        runs-on: ubuntu-latest

        steps:
        - name: checkout code
          uses: actions/checkout@v4

        - name: Setup Go 1.22
          uses: actions/setup-go@v2
          with:
            go-version: 1.22
        
        - name: Build
          run: |
            cd src/product-catalog
            go mod download
            go build -o product-catalog-service main.go

        - name: unit tests
          run: |
            cd src/product-catalog
            go test ./...
    
    code-quality:
        runs-on: ubuntu-latest

        steps:
        - name: checkout code
          uses: actions/checkout@v4
        
        - name: Setup Go 1.22
          uses: actions/setup-go@v2
          with:
           go-version: 1.22
        
        - name: Run golangci-lint
          uses: golangci/golangci-lint-action@v6
          with:
            version: v1.55.2
            run: golangci-lint run
            working-directory: src/product-catalog

    docker:
        runs-on: ubuntu-latest

        needs: build

        steps:
        - name: checkout code
          uses: actions/checkout@v4

        - name: Install Docker
          uses: docker/setup-buildx-action@v1
        
        - name: Login to Docker
          uses: docker/login-action@v3
          with:
            username: ${{ secrets.DOCKER_USERNAME }}
            password: ${{ secrets.DOCKER_TOKEN }}

        - name: Docker Push
          uses: docker/build-push-action@v6
          with:
            context: src/product-catalog
            file: src/product-catalog/Dockerfile
            push: true
            tags: ${{ secrets.DOCKER_USERNAME }}/product-catalog:${{github.run_id}}

    
    updatek8s:
        runs-on: ubuntu-latest

        needs: docker

        steps:
        - name: checkout code
          uses: actions/checkout@v4
          with:
            token: ${{ secrets.GITHUB_TOKEN }}

        - name: Update tag in kubernetes deployment manifest
          run: | 
               sed -i "s|image: .*|image: ${{ secrets.DOCKER_USERNAME }}/product-catalog:${{github.run_id}}|" kubernetes/productcatalog/deploy.yaml
        
        - name: Commit and push changes
          run: |
            git config --global user.email "abdurakhimov.da@gmail.com"
            git config --global user.name "Damir Abdurakhimov"
            git add kubernetes/productcatalog/deploy.yaml
            git commit -m "[CI]: Update product catalog image tag"
            git push origin HEAD:main -f
```
### Triggering the CI Pipeline
To verify the CI workflow:
  - Create a new feature branch (for example: github-ci-check)
  - Make a small dummy change to the application code
  - Commit and push the changes
  - Open a Pull Request (PR) to the main branch
Creating the PR automatically triggers the CI pipeline.
<img width="1623" height="321" alt="Screenshot 2026-01-18 at 3 12 56 PM" src="https://github.com/user-attachments/assets/587bdf1f-2199-4ff3-8104-d5de045dfa33" />

<img width="1614" height="886" alt="Screenshot 2026-01-18 at 3 13 15 PM" src="https://github.com/user-attachments/assets/a1954736-2dd0-4860-87a7-1b7a0f90706f" />

<img width="1888" height="735" alt="Screenshot 2026-01-18 at 3 28 18 PM" src="https://github.com/user-attachments/assets/98f000f8-7d6a-4948-9e4e-0a282b80ea2a" />

### Static Code Analysis Results
  - The Code Quality stage failed, which is expected and useful:
  - Detected usage of deprecated Go functions
  - Found unused functions in the code
  - This shows that static analysis is working correctly and catching real issues before deployment.


### For this setup, Argo CD is installed using plain Kubernetes manifests, as recommended for getting started.

Install Argo CD
1. Create the Argo CD Namespace
```bash
kubectl create namespace argocd
```
2. Apply the Argo CD Manifests
```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
3. Verify Installation
```bash
kubectl get pods -n argocd
```
<img width="906" height="207" alt="Screenshot 2026-01-18 at 3 33 20 PM" src="https://github.com/user-attachments/assets/bf1a3ca1-caf6-4efd-8294-1da05abdbac1" />

4. Check Services:
```bash
kubectl get svc -n argocd
```
<img width="1065" height="208" alt="Screenshot 2026-01-18 at 3 33 38 PM" src="https://github.com/user-attachments/assets/21d23a9f-897b-40b3-b50a-f7d3b0aa1818" />

5. Expose Argo CD UI
```bash
kubectl edit svc argocd-server -n argocd
```
Change: type: ClusterIP to type: LoadBalancer

<img width="420" height="146" alt="Screenshot 2026-01-18 at 3 34 35 PM" src="https://github.com/user-attachments/assets/37105a2f-b248-4cc8-a73e-6e8753e032ea" />

6. Then verify and loadbalancer dns name: 
```bash
kubectl get svc -n argocd
```
<img width="1370" height="954" alt="Screenshot 2026-01-18 at 3 37 37 PM" src="https://github.com/user-attachments/assets/a2705dae-ff0f-447b-a286-23a6d7084b51" />

7. Login to Argo CD
```bash
kubectl get secrets -n argocd
```
<img width="942" height="167" alt="Screenshot 2026-01-18 at 3 38 13 PM" src="https://github.com/user-attachments/assets/54105a99-b178-42f6-92d8-4ba8ed94a150" />
Look for: argocd-initial-admin-secret

8. Decode the Password
```bash
kubectl get secret argocd-initial-admin-secret \
  -n argocd \
  -o jsonpath="{.data.password}" | base64 --decode
```
### Configuring Argo CD Application (GitOps Deployment)
This section explains how to connect Argo CD to a Git repository so that it can automatically deploy updated Kubernetes manifests to the cluster.

### Create an Argo CD Application
1. Open the Argo CD UI
2. Click Create Application

### Application Settings
- Application Name
   - Use a meaningful name such as:
      - product-catalog
      - product-catalog-service
- Project
    - Keep it as default
- Sync Policy
    - Argo CD supports two sync modes:
- Manual
    - Deployment happens only when triggered manually
- Automatic (Recommended)
    - Argo CD automatically detects changes in Git
    - Deploys updates to the cluster without manual action
- For this setup, Automatic sync is enabled.
- Repository Configuration
    - Provide the Git repository URL that contains Kubernetes manifests
- Revision
    - Use HEAD to always track the latest commit
- Path
  - Provide the path where Kubernetes manifests are stored: kubernetes/productcatalog
- Destination Configuration
    - Cluster: https://kubernetes.default.svc
- Namespace
    - Use: default
- Deploy the Application
    - Click Create
  
<img width="1398" height="748" alt="Screenshot 2026-01-18 at 3 39 21 PM" src="https://github.com/user-attachments/assets/7cd32dce-ab33-4a58-98b8-f9755afe6af9" />


