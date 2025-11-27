# Terraform Infrastructure as Code

**Description:** Use when generating or managing infrastructure as code. Creates Terraform from architecture model, validates with security policies, and follows IaC best practices.

## When to Use This Skill

Use this skill when:
- Converting architecture diagrams to Terraform code
- Setting up new infrastructure components
- Managing cloud resources (AWS, GCP, Azure)
- Validating infrastructure against security policies
- Documenting deployment architecture
- Establishing infrastructure standards

## Prerequisites Check

Before proceeding, verify Terraform is installed:

```bash
terraform version
```

If not installed, guide the user:
- **macOS**: `brew install terraform`
- **Linux**: Download from https://www.terraform.io/downloads
- **Windows**: Use Chocolatey `choco install terraform` or download installer

Also check for optional but recommended tools:
- **Conftest** (policy validation): `brew install conftest` or https://www.conftest.dev/install/
- **tflint** (linting): `brew install tflint` or https://github.com/terraform-linters/tflint
- **terraform-docs** (documentation): `brew install terraform-docs`

## Workflow

### 1. Analyze Architecture Model

Look for existing architecture documentation:
- `vault/arc42/07-deployment-view.md` - Deployment architecture
- `workspace/model/` - Overarch model files (EDN format)
- Architecture diagrams or specifications from the user

Extract:
- Infrastructure components (compute, storage, networking)
- Cloud provider and region requirements
- Dependencies between components
- Security requirements
- Scaling requirements

### 2. Generate Terraform Configuration

Create Terraform code in `infrastructure/terraform/` with this structure:

```
infrastructure/terraform/
├── main.tf           # Main configuration
├── variables.tf      # Input variables
├── outputs.tf        # Output values
├── versions.tf       # Provider versions
├── terraform.tfvars.example  # Example values
├── modules/          # Reusable modules
│   ├── networking/
│   ├── compute/
│   └── storage/
└── environments/     # Environment-specific configs
    ├── dev/
    ├── staging/
    └── prod/
```

### 3. Module Templates

Use these starter templates for common patterns:

#### AWS Basic Infrastructure

```hcl
# modules/aws-basic/main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# VPC
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = merge(var.common_tags, {
    Name = "${var.project_name}-vpc"
  })
}

# Public Subnets
resource "aws_subnet" "public" {
  count                   = length(var.availability_zones)
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index)
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true

  tags = merge(var.common_tags, {
    Name = "${var.project_name}-public-${var.availability_zones[count.index]}"
    Type = "public"
  })
}

# Private Subnets
resource "aws_subnet" "private" {
  count             = length(var.availability_zones)
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, count.index + length(var.availability_zones))
  availability_zone = var.availability_zones[count.index]

  tags = merge(var.common_tags, {
    Name = "${var.project_name}-private-${var.availability_zones[count.index]}"
    Type = "private"
  })
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = merge(var.common_tags, {
    Name = "${var.project_name}-igw"
  })
}

# NAT Gateway
resource "aws_eip" "nat" {
  count  = length(var.availability_zones)
  domain = "vpc"

  tags = merge(var.common_tags, {
    Name = "${var.project_name}-nat-eip-${var.availability_zones[count.index]}"
  })
}

resource "aws_nat_gateway" "main" {
  count         = length(var.availability_zones)
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id

  tags = merge(var.common_tags, {
    Name = "${var.project_name}-nat-${var.availability_zones[count.index]}"
  })
}

# modules/aws-basic/variables.tf
variable "project_name" {
  description = "Project name for resource naming"
  type        = string
}

variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
}

variable "common_tags" {
  description = "Common tags for all resources"
  type        = map(string)
  default     = {}
}

# modules/aws-basic/outputs.tf
output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "Public subnet IDs"
  value       = aws_subnet.public[*].id
}

output "private_subnet_ids" {
  description = "Private subnet IDs"
  value       = aws_subnet.private[*].id
}
```

#### GCP Basic Infrastructure

```hcl
# modules/gcp-basic/main.tf
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }
}

# VPC Network
resource "google_compute_network" "main" {
  name                    = "${var.project_name}-network"
  auto_create_subnetworks = false
  project                 = var.project_id
}

# Subnets
resource "google_compute_subnetwork" "subnets" {
  count         = length(var.regions)
  name          = "${var.project_name}-subnet-${var.regions[count.index]}"
  ip_cidr_range = cidrsubnet(var.network_cidr, 8, count.index)
  region        = var.regions[count.index]
  network       = google_compute_network.main.id
  project       = var.project_id

  secondary_ip_range {
    range_name    = "pods"
    ip_cidr_range = cidrsubnet(var.network_cidr, 4, count.index + 1)
  }

  secondary_ip_range {
    range_name    = "services"
    ip_cidr_range = cidrsubnet(var.network_cidr, 6, count.index + 16)
  }
}

# Firewall Rules
resource "google_compute_firewall" "allow_internal" {
  name    = "${var.project_name}-allow-internal"
  network = google_compute_network.main.name
  project = var.project_id

  allow {
    protocol = "tcp"
    ports    = ["0-65535"]
  }

  allow {
    protocol = "udp"
    ports    = ["0-65535"]
  }

  allow {
    protocol = "icmp"
  }

  source_ranges = [var.network_cidr]
}

# modules/gcp-basic/variables.tf
variable "project_id" {
  description = "GCP Project ID"
  type        = string
}

variable "project_name" {
  description = "Project name for resource naming"
  type        = string
}

variable "network_cidr" {
  description = "CIDR block for network"
  type        = string
  default     = "10.0.0.0/16"
}

variable "regions" {
  description = "List of GCP regions"
  type        = list(string)
}

# modules/gcp-basic/outputs.tf
output "network_id" {
  description = "Network ID"
  value       = google_compute_network.main.id
}

output "subnet_ids" {
  description = "Subnet IDs"
  value       = google_compute_subnetwork.subnets[*].id
}
```

#### Kubernetes Cluster (EKS)

```hcl
# modules/eks-cluster/main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# EKS Cluster IAM Role
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

# EKS Cluster
resource "aws_eks_cluster" "main" {
  name     = var.cluster_name
  role_arn = aws_iam_role.cluster.arn
  version  = var.kubernetes_version

  vpc_config {
    subnet_ids              = var.subnet_ids
    endpoint_private_access = true
    endpoint_public_access  = var.enable_public_access
    public_access_cidrs     = var.public_access_cidrs
  }

  enabled_cluster_log_types = [
    "api",
    "audit",
    "authenticator",
    "controllerManager",
    "scheduler"
  ]

  tags = var.common_tags

  depends_on = [
    aws_iam_role_policy_attachment.cluster_policy
  ]
}

# Node Group IAM Role
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

# EKS Node Group
resource "aws_eks_node_group" "main" {
  cluster_name    = aws_eks_cluster.main.name
  node_group_name = "${var.cluster_name}-node-group"
  node_role_arn   = aws_iam_role.node.arn
  subnet_ids      = var.private_subnet_ids

  instance_types = var.node_instance_types

  scaling_config {
    desired_size = var.node_desired_size
    max_size     = var.node_max_size
    min_size     = var.node_min_size
  }

  update_config {
    max_unavailable = 1
  }

  tags = var.common_tags

  depends_on = [
    aws_iam_role_policy_attachment.node_policy
  ]
}

# modules/eks-cluster/variables.tf
variable "cluster_name" {
  description = "EKS cluster name"
  type        = string
}

variable "kubernetes_version" {
  description = "Kubernetes version"
  type        = string
  default     = "1.28"
}

variable "subnet_ids" {
  description = "Subnet IDs for cluster"
  type        = list(string)
}

variable "private_subnet_ids" {
  description = "Private subnet IDs for nodes"
  type        = list(string)
}

variable "enable_public_access" {
  description = "Enable public API access"
  type        = bool
  default     = false
}

variable "public_access_cidrs" {
  description = "CIDR blocks for public access"
  type        = list(string)
  default     = ["0.0.0.0/0"]
}

variable "node_instance_types" {
  description = "EC2 instance types for nodes"
  type        = list(string)
  default     = ["t3.medium"]
}

variable "node_desired_size" {
  description = "Desired number of nodes"
  type        = number
  default     = 2
}

variable "node_min_size" {
  description = "Minimum number of nodes"
  type        = number
  default     = 1
}

variable "node_max_size" {
  description = "Maximum number of nodes"
  type        = number
  default     = 4
}

variable "common_tags" {
  description = "Common tags"
  type        = map(string)
  default     = {}
}

# modules/eks-cluster/outputs.tf
output "cluster_id" {
  description = "EKS cluster ID"
  value       = aws_eks_cluster.main.id
}

output "cluster_endpoint" {
  description = "EKS cluster endpoint"
  value       = aws_eks_cluster.main.endpoint
}

output "cluster_security_group_id" {
  description = "Security group ID attached to the cluster"
  value       = aws_eks_cluster.main.vpc_config[0].cluster_security_group_id
}

output "cluster_certificate_authority_data" {
  description = "Certificate authority data"
  value       = aws_eks_cluster.main.certificate_authority[0].data
}
```

#### Database (RDS PostgreSQL)

```hcl
# modules/rds-postgres/main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# DB Subnet Group
resource "aws_db_subnet_group" "main" {
  name       = "${var.db_name}-subnet-group"
  subnet_ids = var.subnet_ids

  tags = merge(var.common_tags, {
    Name = "${var.db_name}-subnet-group"
  })
}

# Security Group
resource "aws_security_group" "db" {
  name        = "${var.db_name}-sg"
  description = "Security group for ${var.db_name} database"
  vpc_id      = var.vpc_id

  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = var.allowed_security_groups
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(var.common_tags, {
    Name = "${var.db_name}-sg"
  })
}

# RDS Instance
resource "aws_db_instance" "main" {
  identifier     = var.db_name
  engine         = "postgres"
  engine_version = var.postgres_version

  instance_class    = var.instance_class
  allocated_storage = var.allocated_storage
  storage_type      = "gp3"
  storage_encrypted = true

  db_name  = var.database_name
  username = var.master_username
  password = var.master_password

  db_subnet_group_name   = aws_db_subnet_group.main.name
  vpc_security_group_ids = [aws_security_group.db.id]

  backup_retention_period = var.backup_retention_days
  backup_window           = var.backup_window
  maintenance_window      = var.maintenance_window

  multi_az               = var.multi_az
  publicly_accessible    = false
  skip_final_snapshot    = var.skip_final_snapshot
  final_snapshot_identifier = var.skip_final_snapshot ? null : "${var.db_name}-final-snapshot-${formatdate("YYYY-MM-DD-hhmm", timestamp())}"

  enabled_cloudwatch_logs_exports = ["postgresql", "upgrade"]

  tags = merge(var.common_tags, {
    Name = var.db_name
  })
}

# modules/rds-postgres/variables.tf
variable "db_name" {
  description = "Database identifier"
  type        = string
}

variable "database_name" {
  description = "Initial database name"
  type        = string
}

variable "master_username" {
  description = "Master username"
  type        = string
  default     = "postgres"
}

variable "master_password" {
  description = "Master password"
  type        = string
  sensitive   = true
}

variable "postgres_version" {
  description = "PostgreSQL version"
  type        = string
  default     = "15.4"
}

variable "instance_class" {
  description = "DB instance class"
  type        = string
  default     = "db.t4g.micro"
}

variable "allocated_storage" {
  description = "Allocated storage in GB"
  type        = number
  default     = 20
}

variable "vpc_id" {
  description = "VPC ID"
  type        = string
}

variable "subnet_ids" {
  description = "Subnet IDs for DB subnet group"
  type        = list(string)
}

variable "allowed_security_groups" {
  description = "Security groups allowed to access DB"
  type        = list(string)
}

variable "multi_az" {
  description = "Enable Multi-AZ deployment"
  type        = bool
  default     = false
}

variable "backup_retention_days" {
  description = "Backup retention period in days"
  type        = number
  default     = 7
}

variable "backup_window" {
  description = "Backup window"
  type        = string
  default     = "03:00-04:00"
}

variable "maintenance_window" {
  description = "Maintenance window"
  type        = string
  default     = "mon:04:00-mon:05:00"
}

variable "skip_final_snapshot" {
  description = "Skip final snapshot on destroy"
  type        = bool
  default     = false
}

variable "common_tags" {
  description = "Common tags"
  type        = map(string)
  default     = {}
}

# modules/rds-postgres/outputs.tf
output "db_instance_id" {
  description = "Database instance ID"
  value       = aws_db_instance.main.id
}

output "db_endpoint" {
  description = "Database endpoint"
  value       = aws_db_instance.main.endpoint
}

output "db_address" {
  description = "Database address"
  value       = aws_db_instance.main.address
}

output "db_port" {
  description = "Database port"
  value       = aws_db_instance.main.port
}

output "security_group_id" {
  description = "Security group ID"
  value       = aws_security_group.db.id
}
```

### 4. Best Practices for Terraform Organization

Apply these principles:

#### State Management

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "project/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

**Key Points:**
- Use remote state (S3, GCS, Azure Blob)
- Enable state locking (DynamoDB, GCS locking, Azure Blob leases)
- Never commit state files to version control
- Use separate state files per environment
- Implement state encryption

#### Variable Organization

```hcl
# variables.tf - Define all variables
variable "environment" {
  description = "Environment name"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

# terraform.tfvars.example - Example values (committed)
environment = "dev"
vpc_cidr    = "10.0.0.0/16"

# terraform.tfvars - Actual values (NOT committed, in .gitignore)
# Users copy from .example and customize
```

#### Resource Naming Convention

```hcl
locals {
  name_prefix = "${var.project_name}-${var.environment}"

  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "terraform"
    Repository  = var.repository_url
  }
}

resource "aws_instance" "app" {
  # Use consistent naming
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-app-server"
  })
}
```

#### Module Versioning

```hcl
# Reference modules by version
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.1.2"  # Pin to specific version

  name = "${local.name_prefix}-vpc"
  cidr = var.vpc_cidr
}
```

#### Documentation

```hcl
# Use terraform-docs to auto-generate
# Add this to each module's README.md

<!-- BEGIN_TF_DOCS -->
## Requirements

| Name | Version |
|------|---------|
| terraform | >= 1.5.0 |
| aws | ~> 5.0 |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| vpc_cidr | CIDR block for VPC | `string` | n/a | yes |

<!-- END_TF_DOCS -->
```

Generate with:
```bash
terraform-docs markdown table . > README.md
```

### 5. Policy Validation with Conftest

Create OPA policies in `infrastructure/policies/`:

```rego
# infrastructure/policies/terraform.rego
package terraform

deny[msg] {
  resource := input.resource_changes[_]
  resource.type == "aws_s3_bucket"
  not resource.change.after.server_side_encryption_configuration
  msg := sprintf("S3 bucket '%s' must have encryption enabled", [resource.address])
}

deny[msg] {
  resource := input.resource_changes[_]
  resource.type == "aws_db_instance"
  not resource.change.after.storage_encrypted
  msg := sprintf("RDS instance '%s' must have storage encryption enabled", [resource.address])
}

deny[msg] {
  resource := input.resource_changes[_]
  resource.type == "aws_security_group"
  rule := resource.change.after.ingress[_]
  rule.cidr_blocks[_] == "0.0.0.0/0"
  msg := sprintf("Security group '%s' allows unrestricted access (0.0.0.0/0)", [resource.address])
}
```

Validate with:
```bash
terraform plan -out=tfplan.binary
terraform show -json tfplan.binary > tfplan.json
conftest test tfplan.json -p infrastructure/policies/
```

### 6. Run Terraform Plan and Analyze

```bash
cd infrastructure/terraform
terraform init
terraform validate
terraform plan -out=tfplan

# Review the plan carefully
terraform show tfplan

# If using Conftest
terraform show -json tfplan > tfplan.json
conftest test tfplan.json -p ../policies/
```

Analyze the plan output for:
- Resource counts (added, changed, destroyed)
- Cost implications (use Infracost if available)
- Security considerations
- Dependency order
- Potential risks

### 7. Document in arc42

Update `vault/arc42/07-deployment-view.md`:

```markdown
## Infrastructure Overview

### Cloud Provider
- **Provider**: [AWS/GCP/Azure]
- **Region**: [Primary region]
- **Availability Zones**: [List AZs]

### Network Architecture
- **VPC CIDR**: 10.0.0.0/16
- **Public Subnets**: [CIDR blocks]
- **Private Subnets**: [CIDR blocks]

### Compute Resources
- **EKS Cluster**: [cluster-name]
  - Kubernetes Version: 1.28
  - Node Groups: t3.medium (2-4 nodes)

### Data Storage
- **RDS PostgreSQL**: [instance-id]
  - Version: 15.4
  - Instance Class: db.t4g.micro
  - Multi-AZ: Enabled/Disabled
  - Backup Retention: 7 days

### Terraform Structure
```
infrastructure/terraform/
├── environments/
│   ├── dev/
│   ├── staging/
│   └── prod/
└── modules/
    ├── networking/
    ├── compute/
    └── storage/
```

### State Management
- **Backend**: S3
- **Bucket**: my-terraform-state
- **Locking**: DynamoDB table terraform-locks

### Security Policies
- Encryption enforced for S3 and RDS
- No unrestricted security group rules (0.0.0.0/0)
- Private subnets for application workloads
```

### 8. Testing and Validation

```bash
# Linting
tflint

# Security scanning
tfsec .

# Cost estimation (if Infracost installed)
infracost breakdown --path .

# Format code
terraform fmt -recursive

# Validate syntax
terraform validate
```

## Common Patterns and Anti-Patterns

### DO:
- Use modules for reusable components
- Pin provider and module versions
- Use remote state with locking
- Implement encryption by default
- Tag all resources consistently
- Use variables for environment-specific values
- Document expected inputs/outputs
- Run plan before apply

### DON'T:
- Hardcode secrets in Terraform files
- Allow unrestricted security group rules
- Skip state locking
- Use default encryption keys
- Mix environments in same state file
- Make manual changes to Terraform-managed resources
- Ignore plan output warnings

## Integration with Other Skills

This skill works well with:
- **policy-as-code**: Use OPA policies for validation
- **iso25010-quality**: Align infrastructure with quality requirements
- **slo-sli-observability**: Infrastructure supports observability needs
- **architecture-modeling**: Generate Terraform from architecture models

## Troubleshooting

### Common Issues

**State Lock Errors:**
```bash
# Force unlock (use with caution)
terraform force-unlock <lock-id>
```

**Provider Plugin Issues:**
```bash
# Clear plugin cache
rm -rf .terraform
terraform init -upgrade
```

**Drift Detection:**
```bash
# Detect manual changes
terraform plan -refresh-only
```

## Next Steps

After generating infrastructure:
1. Review plan output carefully
2. Validate with Conftest policies
3. Test in dev environment first
4. Document in arc42
5. Commit Terraform code to version control
6. Set up CI/CD for Terraform applies
7. Implement drift detection monitoring
