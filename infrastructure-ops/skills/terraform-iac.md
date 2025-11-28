---
name: terraform-iac
description: Use when user wants to create infrastructure as code, provision cloud resources, or generate Terraform from architecture documentation.
requires:
  tools: [terraform]
  skills: [policy-as-code, threagile-analysis]
---

# Terraform Infrastructure as Code

## Prerequisites

```bash
command -v terraform >/dev/null || { echo "Install: brew install terraform"; exit 1; }
command -v tflint >/dev/null || echo "Optional: brew install tflint"
command -v conftest >/dev/null || echo "Optional: brew install conftest"
```

## Directory Structure

```
infrastructure/terraform/
├── main.tf              # Main configuration
├── variables.tf         # Input variables
├── outputs.tf           # Output values
├── versions.tf          # Provider versions
├── terraform.tfvars.example
├── modules/             # Reusable modules
└── environments/
    ├── dev/
    ├── staging/
    └── prod/
```

## Workflow

### 1. Analyze Architecture

Check existing docs:
- `vault/arc42/07-deployment.md`
- Overarch models in `models/`

Extract: components, cloud provider, dependencies, security requirements.

### 2. Generate Terraform

**versions.tf:**
```hcl
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = { source = "hashicorp/aws", version = "~> 5.0" }
  }
}
```

**variables.tf:**
```hcl
variable "environment" {
  type        = string
  description = "Deployment environment"
}

variable "region" {
  type    = string
  default = "eu-north-1"
}
```

### 3. Validate

```bash
terraform fmt -check
terraform validate
tflint
conftest test . -p ../policies/  # If policies exist
```

### 4. Plan and Apply

```bash
terraform plan -out=tfplan
terraform apply tfplan
```

## Key Patterns

**Module structure:**
```hcl
module "api" {
  source      = "./modules/ecs-service"
  name        = "api"
  environment = var.environment
  # ...
}
```

**Secure defaults:**
```hcl
resource "aws_s3_bucket" "data" {
  bucket = "${var.project}-${var.environment}-data"
}

resource "aws_s3_bucket_public_access_block" "data" {
  bucket                  = aws_s3_bucket.data.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

## Success Criteria

- [ ] `terraform validate` passes
- [ ] `terraform fmt -check` shows no changes needed
- [ ] `terraform plan` shows expected resources
- [ ] Security policies pass (if conftest configured)

## Related Skills

- `policy-as-code` - OPA policies for validation
- `conftest-testing` - Policy testing
- `threagile-analysis` - Security requirements source
