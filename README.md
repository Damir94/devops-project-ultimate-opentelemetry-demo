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

### Terraform Backend Setup (S3 + DynamoDB)
This guide explains how to create a Terraform backend using AWS S3 for state storage and DynamoDB for state locking in the backend folder.

Sample Terraform Code (main.tf)
```hcl
provider "aws" {
  region = "us-east-1"
}
resource "aws_s3_bucket" "terraform_state_bucket" {
  bucket = "terraform-demo-x-state-s3-bucket"

  lifecycle {
    prevent_destroy = false
  }
}
resource "aws_dynamodb_table" "terraform_state_lock" {
  name         = "terraform-ecs-state"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "lock_id"
  attribute {
    name = "lock_id"
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
