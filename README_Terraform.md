# EC2 Nginx Deployment with Terraform

## Overview

This project provisions an AWS EC2 instance using Terraform and installs an Nginx web server on Ubuntu. The Nginx default page is replaced with a custom HTML page displaying:

```text
Welcome to the Terraform-managed Nginx Server on Ubuntu
```

This assignment demonstrates Infrastructure as Code (IaC) using Terraform to create, manage, and destroy AWS resources.

---

## Project Structure

```text
terraform-nginx-ubuntu/
├── main.tf
├── variables.tf
├── outputs.tf
├── README.md
└── .gitignore
```

---

## Resources Created

The Terraform configuration creates the following AWS resources:

- AWS provider configuration
- Default VPC lookup
- Ubuntu 20.04 LTS AMI lookup
- EC2 instance running Ubuntu
- Security group allowing:
  - HTTP access on port 80
  - SSH access on port 22
- Nginx web server installed using `user_data`
- Custom Nginx index page
- Terraform outputs for public IP and URL

---

## Technical Requirements Covered

- Uses AWS provider and selected AWS region
- Launches an Ubuntu 20.04 LTS EC2 instance
- Uses default VPC only
- Does not create a custom VPC, subnet, or internet gateway
- Configures security group for HTTP and SSH
- Installs Nginx using EC2 `user_data`
- Replaces default Nginx page with a custom HTML page
- Outputs the EC2 public IP address
- Supports complete cleanup using `terraform destroy`

---

## Prerequisites

Before running this project, make sure the following tools are installed:

- Terraform CLI
- AWS CLI
- Git
- AWS account or AWS lab account
- AWS access key and secret access key configured

Verify the tools:

```bash
terraform version
```

```bash
aws --version
```

```bash
git --version
```

---

## AWS CLI Configuration

Configure AWS credentials using:

```bash
aws configure
```

Enter the required values:

```text
AWS Access Key ID: <your-access-key-id>
AWS Secret Access Key: <your-secret-access-key>
Default region name: us-east-1
Default output format: json
```

Verify AWS access:

```bash
aws sts get-caller-identity
```

If the command returns your AWS account ID and ARN, the AWS CLI is configured correctly.

---

## Terraform Files

### main.tf

```hcl
terraform {
  required_version = ">= 1.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

data "aws_vpc" "default" {
  default = true
}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

resource "aws_security_group" "nginx_sg" {
  name        = "terraform-nginx-sg"
  description = "Allow HTTP and SSH access"
  vpc_id      = data.aws_vpc.default.id

  ingress {
    description = "Allow HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "Allow SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    description = "Allow all outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name        = "terraform-nginx-sg"
    Environment = "Assignment"
  }
}

resource "aws_instance" "nginx_server" {
  ami                    = data.aws_ami.ubuntu.id
  instance_type          = var.instance_type
  vpc_security_group_ids = [aws_security_group.nginx_sg.id]

  user_data = <<-EOF
              #!/bin/bash
              apt update -y
              apt install nginx -y

              cat > /var/www/html/index.html <<HTML
              <html>
              <head>
                <title>Terraform Nginx Server</title>
              </head>
              <body>
                <h1>Welcome to the Terraform-managed Nginx Server on Ubuntu</h1>
              </body>
              </html>
              HTML

              systemctl enable nginx
              systemctl restart nginx
              EOF

  tags = {
    Name        = "Terraform-Nginx-Server"
    Environment = "Assignment"
  }
}
```

### variables.tf

```hcl
variable "aws_region" {
  description = "AWS region where resources will be created"
  type        = string
  default     = "us-east-1"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}
```

> Note: The assignment mentions `t2.micro`. If your AWS lab account shows the error that `t2.micro` is not eligible for Free Tier, use `t3.micro` as shown above.

### outputs.tf

```hcl
output "instance_public_ip" {
  description = "Public IP address of the EC2 instance"
  value       = aws_instance.nginx_server.public_ip
}

output "nginx_url" {
  description = "URL to access the Nginx web server"
  value       = "http://${aws_instance.nginx_server.public_ip}"
}
```

### .gitignore

```gitignore
# Terraform directories and files
.terraform/
*.tfstate
*.tfstate.*
.terraform.lock.hcl

# Terraform plan files
*.tfplan

# Logs
*.log

# IDE files
.vscode/
.idea/
```

---

## Step-by-Step Deployment Instructions

### Step 1: Clone or Create the Project Directory

```bash
mkdir terraform-nginx-ubuntu
cd terraform-nginx-ubuntu
```

Create the required files:

```bash
touch main.tf variables.tf outputs.tf README.md .gitignore
```

---

### Step 2: Initialize Terraform

```bash
terraform init
```

Expected result:

```text
Terraform has been successfully initialized!
```

---

### Step 3: Format Terraform Files

```bash
terraform fmt
```

---

### Step 4: Validate Terraform Configuration

```bash
terraform validate
```

Expected result:

```text
Success! The configuration is valid.
```

---

### Step 5: Review Terraform Plan

```bash
terraform plan
```

Expected result:

```text
Plan: 2 to add, 0 to change, 0 to destroy.
```

Terraform should create:

- One security group
- One EC2 instance

---

### Step 6: Apply Terraform Configuration

```bash
terraform apply
```

When prompted, type:

```text
yes
```

After successful deployment, Terraform displays outputs similar to:

```text
instance_public_ip = "54.xxx.xxx.xxx"
nginx_url = "http://54.xxx.xxx.xxx"
```

---

### Step 7: Verify Nginx Deployment

Open the URL displayed in Terraform output:

```text
http://<public-ip-address>
```

Expected webpage content:

```text
Welcome to the Terraform-managed Nginx Server on Ubuntu
```

You can also verify from CLI:

```bash
curl http://<public-ip-address>
```

---

## Screenshots to Include

Add screenshots in your course submission or repository showing:

1. Successful `terraform init`
2. Successful `terraform plan`
3. Successful `terraform apply`
4. Terraform output showing the public IP
5. Browser showing the custom Nginx page
6. Successful `terraform destroy`

---

## Destroying Resources

To avoid unnecessary AWS charges, destroy all resources after validation:

```bash
terraform destroy
```

When prompted, type:

```text
yes
```

Expected result:

```text
Destroy complete!
```

---

## GitHub Submission Steps

Initialize Git:

```bash
git init
```

Add only required files:

```bash
git add main.tf variables.tf outputs.tf README.md .gitignore
```

Commit the files:

```bash
git commit -m "Initial Terraform Nginx assignment"
```

Add GitHub remote repository:

```bash
git remote add origin https://github.com/<your-username>/terraform-nginx-ubuntu.git
```

Rename branch to main:

```bash
git branch -M main
```

Push to GitHub:

```bash
git push -u origin main
```

---

## Important GitHub Note

Do not commit the `.terraform` directory because it contains large provider binaries. GitHub rejects files larger than 100 MB.

If accidentally committed, remove it using:

```bash
git rm -r --cached .terraform
```

Then commit again:

```bash
git add .gitignore
git commit -m "Remove Terraform generated files"
git push -u origin main --force
```

---

## Troubleshooting

### Error: Instance type is not eligible for Free Tier

If you see:

```text
The specified instance type is not eligible for Free Tier
```

Update `variables.tf`:

```hcl
variable "instance_type" {
  default = "t3.micro"
}
```

Then run:

```bash
terraform plan
terraform apply
```

---

### Error: GitHub Large File Detected

If GitHub rejects your push due to Terraform provider files, ensure `.gitignore` includes:

```gitignore
.terraform/
*.tfstate
*.tfstate.*
.terraform.lock.hcl
```

Then remove `.terraform` from Git tracking:

```bash
git rm -r --cached .terraform
git add .
git commit -m "Remove Terraform provider files"
git push -u origin main --force
```

---

## Final Submission Checklist

- [x] `main.tf` created
- [x] `variables.tf` created
- [x] `outputs.tf` created
- [x] `.gitignore` created
- [x] README.md created
- [x] Terraform initialized successfully
- [x] Terraform plan completed successfully
- [x] EC2 instance deployed
- [x] Nginx installed successfully
- [x] Custom webpage verified in browser
- [x] Public IP output generated
- [x] Screenshots captured
- [x] Terraform destroy completed
- [x] GitHub repository submitted

---

## Author

Sirajuddin Mohammed
