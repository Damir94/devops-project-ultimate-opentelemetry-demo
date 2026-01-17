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
### Big Picture (One-Line Summary)
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

In this lecture we will learn how to create VPC and EKS using the modules that we wrote in the previous lecture. Because we already wrote the modules.

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
  default     = "us-west-2"
}

variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "Availability zones"
  type        = list(string)
  default     = ["us-west-2a", "us-west-2b", "us-west-2c"]
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
### Connecting kubectl to an Amazon EKS Cluster
By default, kubectl does not know which Kubernetes cluster to connect to.
To communicate with a cluster (like EKS), kubectl relies on a configuration file called kubeconfig.

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
kubectl apply -f service-account.yaml
```
Verify
```bash
kubectl get sa
```
You should see: opentelemetry-demo
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

## Accessing the Application
### Why the App Is Not Accessible Yet
If you check the frontend service:
```bash
kubectl get svc | grep frontend
```
You will see: Private IPs (e.g., 172.x.x.x)
These IPs are only accessible inside the VPC

That means:
  - You cannot access the application from your browser yet
  - The EKS cluster runs inside a private VPC network
### Summary
  - Always verify cluster context before deployment
  - Ensure namespace is clean
  - Create the service account first
  - Deploy all services and deployments
  - Wait for all pods to be running
  - Microservices communicate via service names
  - Application is not publicly accessible yet (by design)

### Exposing the Application Using LoadBalancer Service
By default, Kubernetes services are created as ClusterIP, which means they are accessible only inside the cluster.
To access the application from the internet, we must expose the frontend using a LoadBalancer service.

### Step 1: Identify the Frontend Service. First, find the frontend proxy service:
```bash
kubectl get svc | grep frontend-proxy
```
### Step 2: Edit the Service Type
```bash
kubectl edit svc frontend-proxy
```
At the bottom of the file, change: type: ClusterIP to type: LoadBalancer

### Step 3: Verify the Load Balancer Check the service status: kubectl get svc
```bash
kubectl get svc
```
Copy the EXTERNAL-IP / FQDN and access it in the browser: 

### Installing AWS ALB Ingress Controller on EKS
This guide explains how to install the AWS ALB (AWS Load Balancer) Ingress Controller on an EKS cluster, along with the required IAM setup.
#### Prerequisites
Before starting, make sure:
  - You are connected to the correct EC2 instance
  - eksctl is installed: eksctl version
  - kubectl is connected to the correct EKS cluster: kubectl config current-context
  - helm is installed
You know:
  - EKS cluster name
  - AWS region
  - AWS account ID
  - VPC ID used by the EKS cluster

1. Setup OIDC Connector
Without this, the controller cannot create or manage ALBs.
Set the cluster name:
```bash
export cluster_name=demo-cluster
```
### Get the OIDC ID for the cluster
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
Add the Helm repository
```bash
helm repo add eks https://aws.github.io/eks-charts
```
Install the ALB Controller
```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \            
  -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>
```
7. Verify the Controller is Running
```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```
8. Common Issue: No LoadBalancer Address Problem
```bash
 kubectl get ingress -n robot-shop
```
You might see: No ADDRESS assigned
### Why this happens
The IAM policy is missing a required permission: elasticloadbalancing:DescribeListenerAttributes
Without it, the controller cannot fully read ALB configuration.

9. Check if the Permission Exists
```bash
aws iam get-policy-version \
    --policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
    --version-id $(aws iam get-policy --policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy --query 'Policy.DefaultVersionId' --output text)
```
10. Fix Missing Permission (If Needed)
Download the current policy
```bash
aws iam get-policy-version \
    --policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
    --version-id $(aws iam get-policy --policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy --query 'Policy.DefaultVersionId' --output text) \
    --query 'PolicyVersion.Document' --output json > policy.json
```
Add this permission to policy.json
```bash
{
  "Effect": "Allow",
  "Action": "elasticloadbalancing:DescribeListenerAttributes",
  "Resource": "*"
}
```
Create a new policy version
```bash
aws iam create-policy-version \
    --policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://policy.json \
    --set-as-default
```
### Access Frontend Application Using Ingress (ALB)
This guide explains how to expose the frontend proxy of the application to external users using Kubernetes Ingress with the AWS Load Balancer Controller (ALB).
Prerequisites
  - AWS Load Balancer Controller installed and running
  - Frontend proxy Deployment and Service already created
  - kubectl access to the cluster
  - Service should NOT be of type LoadBalancer

Step 1: Remove Existing LoadBalancer Service
Change service type back to NodePort or ClusterIP
```bash
kubectl edit svc opentelemetry-demo-frontend-proxy
```
type: NodePort (or ClusterIP)

Step 2: Create Ingress Resource
```bash
cd ultimate-devops-project-demo/kubernetes/frontend-proxy
```
Step 3: Create a new file: ingress.yaml
Key points:
  - Use host-based routing
  - Use a dummy domain (example.com) for testing
  - Route traffic to the frontend proxy service
  - Use ALB-specific annotations
  - Specify IngressClassName as alb
Sample ingress.yaml
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
          - path: /
            pathType: Prefix
            backend:
              service:
                name: opentelemetry-demo-frontend-proxy
                port:
                  number: 8080
```
Step 4: Apply the Ingress
```bash
kubectl apply -f ingress.yaml
kubectl get ingress
```

Step 5: Verify ALB Creation
Go to AWS Console → EC2 → Load Balancers
Wait until the load balancer status becomes Active

Step 6: Troubleshooting (If ADDRESS Is Missing)
If you do not see an ADDRESS for the ingress: 
```bash
kubectl get pods -n kube-system
```
Check logs of the ALB controller pod: 
```bash
kubectl logs -n kube-system <alb-controller-pod-name>
```
Step 8: Configure Local DNS (For Testing Only)
Since example.com is not a real domain you own, map it locally.
On macOS / Linux
Edit hosts file: 
```bash
sudo vi /etc/hosts
```
Add: <ALB-IP-ADDRESS> example.com
Step 9: Access the Application
```bash
http://example.com
```
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
4. Check Services:
```bash
kubectl get svc -n argocd
```
5. Expose Argo CD UI
```bash
kubectl edit svc argocd-server -n argocd
```
Change: type: ClusterIP to type: LoadBalancer
6. Then verify: 
```bash
kubectl get svc -n argocd
```
7. Login to Argo CD
```bash
kubectl get secrets -n argocd
```
Look for: argocd-initial-admin-secret
8. Decode the Password
```bash
kubectl get secret argocd-initial-admin-secret \
  -n argocd \
  -o jsonpath="{.data.password}" | base64 --decode
```
