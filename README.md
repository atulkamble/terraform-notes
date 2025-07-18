# 🗞 Terraform Notes, Commands, Tutorials & Scenarios BY @atulkamble

## 🎯 Terraform Interview Key Points to Remember

- **Terraform is declarative**: It defines *what* the infrastructure should look like, not *how* to achieve it.
- **HCL (HashiCorp Configuration Language)** is the primary language used.
- **State file (`terraform.tfstate`)** is critical — stores the current state of resources.
- **`.terraform.lock.hcl`** locks provider versions for consistent runs.
- **Terraform is idempotent**: Multiple `apply` runs result in the same state.
- **Use `terraform fmt`, `validate`, and `plan`** before every apply.
- **Always version control `.tf` files**, but **never commit `terraform.tfstate` or secrets**.
- **Remote state** can be stored in S3 with locking using DynamoDB.
- **Workspaces** allow managing multiple environments (dev, prod).
- **Modules** enable reuse and organization of infrastructure code.
- **Lifecycle blocks** (`create_before_destroy`, `prevent_destroy`) control behavior.
- **Use `count`, `for_each`, and `dynamic`** for scalable infrastructure.
- **`data` blocks** are used to fetch existing cloud resources.
- **Terraform Cloud/Enterprise** provides collaboration, runs, version control integration.
- **Useful Interview Tip**: Be ready to explain a real project or scenario and how Terraform was used.

## 📖 What is Terraform?
Terraform is an open-source infrastructure as code (IaC) tool developed by HashiCorp that allows you to define, provision, and manage cloud infrastructure using a declarative configuration language (HCL - HashiCorp Configuration Language). 

### 🌐 Key Features:
- **Cloud Agnostic:** Supports AWS, Azure, GCP, and many others via providers.
- **Declarative Syntax:** Describe the desired end state of your infrastructure.
- **State Management:** Keeps track of infrastructure changes over time.
- **Modular & Scalable:** Use reusable modules and variables.

Terraform is widely used in DevOps workflows to automate infrastructure deployment, improve consistency, and reduce human error.

## 🧱 Basics
```hcl
# provider.tf
provider "aws" {
  region = "us-east-1"
}
```

## 🛠 Terraform Commands List

### 📂 Terraform Ignore Files (.terraformignore and .gitignore)

To prevent Terraform from processing or uploading unnecessary files (especially with remote backends like S3), use `.terraformignore`. For Git, use `.gitignore`.

#### Example `.terraformignore`:
```
*.log
*.bak
*.zip
*.tar.gz
*.tmp
.DS_Store
*.pem
``` 

This helps keep remote state clean and prevents uploading local artifacts.

#### Example `.gitignore` for Terraform:
```
.terraform/
*.tfstate
*.tfstate.backup
.terraform.lock.hcl
*.tfvars
*.pem
*.key
*.zip
```
```bash
terraform init                   # Initialize working directory
terraform plan                   # Show execution plan
terraform apply                  # Apply changes (type 'yes' when prompted)
terraform destroy                # Destroy resources
terraform validate               # Validate config files
terraform fmt                    # Format code
terraform output                 # Display output values
terraform import RESOURCE ID     # Import existing resource to Terraform state
terraform taint RESOURCE         # Mark a resource for recreation
terraform state list             # List all resources in state file
terraform state show RESOURCE    # Show detailed info about resource in state
terraform state rm RESOURCE      # Remove resource from state file
terraform graph                  # Visualize resource graph
terraform providers              # Show providers required by config
terraform show                   # Show current state or a plan file
terraform workspace list         # List available workspaces
terraform workspace new dev      # Create a new workspace 'dev'
terraform workspace select dev   # Switch to 'dev' workspace
terraform output -json           # Show outputs in JSON format
terraform plan -out=tfplan       # Save plan to file
terraform apply tfplan           # Apply saved plan
terraform refresh                # Refresh state with real infrastructure
```


---

## ✅ Beginner Tutorial: EC2 Instance
### Files:
- `main.tf`, `variables.tf`, `output.tf`

```hcl
# main.tf
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  key_name      = "my-key"
  tags = {
    Name = "WebServer"
  }
}
```

```hcl
# variables.tf
variable "region" {
  default = "us-east-1"
}
```

```hcl
# output.tf
output "instance_ip" {
  value = aws_instance.web.public_ip
}
```

### Run Commands:
```bash
terraform init                   # Initialize the working directory
terraform validate               # Validate the syntax of Terraform files
terraform fmt                    # Format code for readability
terraform plan                   # Preview infrastructure changes
terraform apply                  # Apply the planned infrastructure changes
terraform output                 # Show output values (e.g., IP address)
terraform show                   # View full state info of deployed infrastructure
terraform providers              # Check provider versions and requirements
terraform state list             # See all tracked resources
```

---

## 📦 Intermediate Tutorial: Use Modules
```hcl
module "ec2_instance" {
  source        = "terraform-aws-modules/ec2-instance/aws"
  name          = "my-ec2"
  instance_type = "t2.micro"
  ami           = "ami-0c55b159cbfafe1f0"
}
```

### Run Commands:
```bash
terraform init                   # Setup the environment and download providers
terraform validate               # Check for syntax correctness
terraform fmt                    # Format all files properly
terraform plan                   # See what will be created or changed
terraform apply                  # Confirm and deploy infrastructure
terraform state list             # View all resources in state
terraform show                   # Display current infrastructure state
```

---

## 📘 Advanced Scenario: EC2 + Apache + S3 Image
### `user_data.sh`
```bash
#!/bin/bash
sudo yum update -y
sudo yum install -y httpd
aws s3 cp s3://my-bucket/image.jpg /var/www/html/
systemctl start httpd
systemctl enable httpd
```

### Terraform:
```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  user_data     = file("user_data.sh")
  tags = {
    Name = "ApacheS3"
  }
}
```

### Run Commands:
```bash
terraform init
terraform apply
```

---

## 🔒 Scenario: VPC Peering with EC2 in each VPC
```hcl
resource "aws_vpc" "vpc1" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_vpc" "vpc2" {
  cidr_block = "10.1.0.0/16"
}

resource "aws_vpc_peering_connection" "peer" {
  vpc_id        = aws_vpc.vpc1.id
  peer_vpc_id   = aws_vpc.vpc2.id
  auto_accept   = true
}

resource "aws_route" "route1" {
  route_table_id         = aws_vpc.vpc1.main_route_table_id
  destination_cidr_block = "10.1.0.0/16"
  vpc_peering_connection_id = aws_vpc_peering_connection.peer.id
}
```

### Run Commands:
```bash
terraform init
terraform apply
```

---

## 🔁 Blue-Green Deployment Example (ALB + ASG)
```hcl
# Create 2 Launch Templates or ASGs
# Use lifecycle block to prevent in-place update
# Weighted target group config for routing traffic
```

### Run Commands:
```bash
terraform init
terraform validate
terraform plan
terraform apply
terraform show
```

---

## 📚 Additional Tutorials

## 🧠 Additional Things To Do (Enhancements & Explorations)

### 🔬 1. Real-World Scenario Codes

#### ✅ Multi-AZ High Availability (VPC + 2 Subnets + EC2)
```hcl
resource "aws_subnet" "public_1" {
  availability_zone = "us-east-1a"
  ...
}

resource "aws_subnet" "public_2" {
  availability_zone = "us-east-1b"
  ...
}

resource "aws_instance" "app" {
  count = 2
  subnet_id = aws_subnet.public_[count.index].id
  ...
}
```

#### ✅ Bastion Host with Public/Private Subnets
```hcl
resource "aws_instance" "bastion" {
  subnet_id = aws_subnet.public.id
  associate_public_ip_address = true
}

resource "aws_instance" "private" {
  subnet_id = aws_subnet.private.id
  associate_public_ip_address = false
}
```

#### ✅ RDS + EC2 Integration
```hcl
resource "aws_db_instance" "db" {
  allocated_storage = 20
  engine            = "mysql"
  ...
}

resource "aws_security_group_rule" "db_allow" {
  from_port   = 3306
  to_port     = 3306
  protocol    = "tcp"
  ...
}
```

#### ✅ EKS Cluster with Workers
Use module: [terraform-aws-eks](https://github.com/terraform-aws-modules/terraform-aws-eks)
```hcl
module "eks" {
  source          = "terraform-aws-modules/eks/aws"
  cluster_name    = "demo"
  subnet_ids      = [subnet-1, subnet-2]
  ...
}
```

#### ✅ IAM Role + Policy
```hcl
resource "aws_iam_role" "example" {
  name = "example"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = { Service = "ec2.amazonaws.com" }
    }]
  })
}

resource "aws_iam_role_policy_attachment" "policy" {
  role       = aws_iam_role.example.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonS3FullAccess"
}
```

### ⚙️ 2. Advanced Terraform Concepts
```hcl
resource "aws_instance" "web" {
  count = 2
  lifecycle {
    prevent_destroy = true
  }
  provisioner "local-exec" {
    command = "echo 'Provisioned!'"
  }
  depends_on = [aws_s3_bucket.logs]
}
```

#### Backend Config (S3 + DynamoDB)
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "dev/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-lock"
  }
}
```

#### terraform console
```bash
terraform console
> var.project_name
```

### 📦 3. Advanced State Management
```bash
terraform workspace new dev
terraform workspace select prod
terraform state list
terraform state mv old.new
terraform state pull
terraform state push tfstate.json
```

### 🔄 4. CI/CD & Security
- GitHub Actions example:
```yaml
- name: Terraform Apply
  run: terraform apply -auto-approve
```
- Run Scanners:
```bash
tflint .
tfsec .
checkov -d .
```

### 🧪 5. Infrastructure Testing
```go
package test
func TestInfra(t *testing.T) {
  terraformOptions := &terraform.Options{
    TerraformDir: "../stage",
  }
  terraform.InitAndApply(t, terraformOptions)
}
```

### 📊 6. Visualizations
```bash
terraform graph | dot -Tpng > graph.png
```

### 📚 7. Module Example
```hcl
module "ec2" {
  source   = "./modules/ec2"
  ami_id   = var.ami_id
  key_name = var.key_name
}
```

### 📂 8. Organize Codebase
```
terraform-project/
├── main.tf
├── variables.tf
├── output.tf
├── modules/
│   ├── ec2/
│   └── vpc/
├── environments/
│   ├── dev/
│   └── prod/
```

### 🛠 10. Practice Projects
- **Multi-tier App**: VPC + Web + DB
- **EKS Fargate**: Serverless workloads
- **Lambda + API Gateway**
- **Blue-Green ALB + ASG**
- **S3 Website + CloudFront + Route53**

### ⚙️ Scenario-Based Terraform Commands

#### 1. Validate Before Plan
```bash
terraform validate       # Ensure HCL syntax is correct
terraform plan -out=tfplan
terraform show tfplan    # Review the plan before applying
```

#### 2. Apply with Auto-Approve (CI/CD)
```bash
terraform apply -auto-approve   # Skip confirmation
```

#### 3. Refresh State from Real Infra
```bash
terraform refresh                # Re-sync state file with actual infrastructure
```

#### 4. Destroy Specific Resource
```bash
terraform destroy -target=aws_instance.web
```

#### 5. Remove Resource from State (e.g., unmanaged manually)
```bash
terraform state rm aws_instance.web
```

#### 6. Import Existing Resource
```bash
terraform import aws_instance.web i-0abcdef1234567890
terraform state show aws_instance.web
```

#### 7. Manage Workspaces
```bash
terraform workspace new dev
terraform workspace select dev
terraform workspace list
```

### 🔁 Looping with count and for_each
```hcl
resource "aws_instance" "web" {
  count         = 3
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  tags = {
    Name = "Web-${count.index}"
  }
}
```

### Run:
```bash
terraform init
terraform plan
terraform apply
```

### 🔧 Dynamic Block Example (Security Group Rules)
```hcl
variable "ports" {
  default = [22, 80, 443]
}

resource "aws_security_group" "web_sg" {
  name   = "web-sg"
  vpc_id = aws_vpc.main.id

  dynamic "ingress" {
    for_each = var.ports
    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}
```

### Run:
```bash
terraform init
terraform validate
terraform apply
```

### 🧪 Use Data Sources (e.g., latest AMI)
```hcl
data "aws_ami" "latest_amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

resource "aws_instance" "example" {
  ami           = data.aws_ami.latest_amazon_linux.id
  instance_type = "t2.micro"
}
```

### Run:
```bash
terraform init
terraform apply
```

---
## 👨‍💻 Author

**Atul Kamble**

- 💼 [LinkedIn](https://www.linkedin.com/in/atuljkamble)
- 🐙 [GitHub](https://github.com/atulkamble)
- 🐦 [X](https://x.com/Atul_Kamble)
- 📷 [Instagram](https://www.instagram.com/atuljkamble)
- 🌐 [Website](https://www.atulkamble.in)
