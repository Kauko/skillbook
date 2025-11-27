---
description: Use when generating security policies from Threagile findings. Creates OPA/Rego policies for infrastructure validation.
---

# Policy as Code with OPA/Rego

## Prerequisites Check

Verify Open Policy Agent (OPA) is available:

1. Check for OPA CLI:
   ```bash
   which opa
   ```

2. If not found, guide installation:
   ```bash
   # macOS
   brew install opa

   # Linux
   curl -L -o opa https://openpolicyagent.org/downloads/latest/opa_linux_amd64
   chmod +x opa
   sudo mv opa /usr/local/bin/

   # Windows
   # Download from: https://openpolicyagent.org/downloads/latest/opa_windows_amd64.exe
   ```

3. Verify installation:
   ```bash
   opa version
   ```

## Read Threat Model and Security Requirements

Before generating policies, understand the security context:

1. Read Threagile output:
   ```bash
   cat vault/security/threat-analysis/risks.json
   ```

2. Read threat model documentation:
   - `vault/security/threat-model.md` - Human-readable findings
   - `vault/security/threagile.yaml` - Architecture model

3. Identify enforceable mitigations:
   - Which risks can be prevented by policy enforcement?
   - What infrastructure configurations need validation?
   - Which security controls require automated checks?

## Policy Generation Strategy

Map threats to policy domains:

### Infrastructure Policies
- **Encryption in Transit**: Enforce TLS versions, certificate validation
- **Encryption at Rest**: Require encrypted storage, key management
- **Network Isolation**: Validate network segmentation, firewall rules
- **Access Control**: Check IAM policies, RBAC configurations

### Container Policies
- **Base Images**: Prohibit untrusted registries, require scanning
- **Privileges**: Block privileged containers, root users
- **Resources**: Enforce limits, prevent resource exhaustion
- **Secrets**: Disallow hardcoded secrets, require secret management

### Kubernetes Policies
- **Pod Security**: Enforce security contexts, admission controls
- **Network Policies**: Require network policies for namespaces
- **Service Mesh**: Enforce mTLS, authorization policies
- **Compliance**: RBAC, audit logging, security standards

### Terraform Policies
- **Cloud Resources**: Validate security groups, bucket policies
- **Encryption**: Require encryption settings on storage/databases
- **Monitoring**: Ensure logging and monitoring enabled
- **Compliance**: Tag enforcement, region restrictions

## Generate Rego Policies

Create policies in `infrastructure/policies/` organized by domain:

### Directory Structure
```
infrastructure/policies/
├── terraform/
│   ├── encryption.rego
│   ├── encryption_test.rego
│   ├── networking.rego
│   └── networking_test.rego
├── kubernetes/
│   ├── pod-security.rego
│   ├── pod-security_test.rego
│   ├── network-policy.rego
│   └── network-policy_test.rego
├── docker/
│   ├── dockerfile-security.rego
│   └── dockerfile-security_test.rego
└── common/
    ├── helpers.rego
    └── helpers_test.rego
```

### Example Policy: Terraform S3 Encryption

Create `infrastructure/policies/terraform/encryption.rego`:

```rego
package terraform.encryption

# METADATA
# title: S3 Bucket Encryption
# description: Ensures S3 buckets have encryption enabled
# related_risks:
#   - RISK-002: Unencrypted Data at Rest
#   - RISK-007: Data Breach via Storage Access
# severity: high
# frameworks:
#   - CWE-311
#   - ASVS-V9.1.1
#   - NIST-CSF-PR.DS-1

import future.keywords.contains
import future.keywords.if
import future.keywords.in

# Deny S3 buckets without server-side encryption
deny[msg] {
    resource := input.resource.aws_s3_bucket[name]
    not has_encryption(resource)

    msg := sprintf(
        "S3 bucket '%s' does not have server-side encryption enabled. " +
        "This violates security policy for data at rest. " +
        "Add server_side_encryption_configuration block. " +
        "Related: vault/security/threat-model.md#risk-002",
        [name]
    )
}

# Check if bucket has encryption configuration
has_encryption(bucket) if {
    bucket.server_side_encryption_configuration
}

# Deny weak encryption algorithms
deny[msg] {
    resource := input.resource.aws_s3_bucket[name]
    encryption := resource.server_side_encryption_configuration[_]
    rule := encryption.rule[_]
    algorithm := rule.apply_server_side_encryption_by_default[_].sse_algorithm

    not is_strong_encryption(algorithm)

    msg := sprintf(
        "S3 bucket '%s' uses weak encryption algorithm '%s'. " +
        "Use AES256 or aws:kms instead.",
        [name, algorithm]
    )
}

# Approved encryption algorithms
is_strong_encryption(algo) if {
    algo == "AES256"
}

is_strong_encryption(algo) if {
    algo == "aws:kms"
}

# Recommend KMS for sensitive data
warn[msg] {
    resource := input.resource.aws_s3_bucket[name]
    encryption := resource.server_side_encryption_configuration[_]
    rule := encryption.rule[_]
    algorithm := rule.apply_server_side_encryption_by_default[_].sse_algorithm

    algorithm == "AES256"
    bucket_has_sensitive_tag(resource)

    msg := sprintf(
        "S3 bucket '%s' is tagged as sensitive but uses AES256. " +
        "Consider using aws:kms for better key management and audit trails.",
        [name]
    )
}

bucket_has_sensitive_tag(bucket) if {
    bucket.tags.sensitivity == "high"
}

bucket_has_sensitive_tag(bucket) if {
    bucket.tags.compliance == "pci"
}
```

### Example Test: Terraform S3 Encryption Test

Create `infrastructure/policies/terraform/encryption_test.rego`:

```rego
package terraform.encryption

# Test: Bucket without encryption should be denied
test_s3_bucket_no_encryption {
    result := deny with input as {
        "resource": {
            "aws_s3_bucket": {
                "test_bucket": {
                    "bucket": "my-bucket"
                }
            }
        }
    }

    count(result) > 0
    contains(result[_], "does not have server-side encryption enabled")
}

# Test: Bucket with AES256 encryption should pass
test_s3_bucket_with_aes256 {
    result := deny with input as {
        "resource": {
            "aws_s3_bucket": {
                "test_bucket": {
                    "bucket": "my-bucket",
                    "server_side_encryption_configuration": {
                        "rule": {
                            "apply_server_side_encryption_by_default": {
                                "sse_algorithm": "AES256"
                            }
                        }
                    }
                }
            }
        }
    }

    count(result) == 0
}

# Test: Bucket with KMS encryption should pass
test_s3_bucket_with_kms {
    result := deny with input as {
        "resource": {
            "aws_s3_bucket": {
                "test_bucket": {
                    "bucket": "my-bucket",
                    "server_side_encryption_configuration": {
                        "rule": {
                            "apply_server_side_encryption_by_default": {
                                "sse_algorithm": "aws:kms",
                                "kms_master_key_id": "arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012"
                            }
                        }
                    }
                }
            }
        }
    }

    count(result) == 0
}

# Test: Sensitive bucket with AES256 should warn
test_sensitive_bucket_should_use_kms {
    result := warn with input as {
        "resource": {
            "aws_s3_bucket": {
                "test_bucket": {
                    "bucket": "my-bucket",
                    "tags": {
                        "sensitivity": "high"
                    },
                    "server_side_encryption_configuration": {
                        "rule": {
                            "apply_server_side_encryption_by_default": {
                                "sse_algorithm": "AES256"
                            }
                        }
                    }
                }
            }
        }
    }

    count(result) > 0
    contains(result[_], "Consider using aws:kms")
}
```

## Common Rego Patterns Library

Create reusable patterns in `infrastructure/policies/common/helpers.rego`:

```rego
package common.helpers

import future.keywords.contains
import future.keywords.if
import future.keywords.in

# Check if resource has required tags
has_required_tags(resource, required) if {
    tags := resource.tags
    required_set := {tag | tag := required[_]}
    actual_set := {tag | tags[tag]}
    required_set - actual_set == set()
}

# Check if TLS version is secure
is_secure_tls(version) if {
    version == "TLSv1.2"
}

is_secure_tls(version) if {
    version == "TLSv1.3"
}

# Check if port is in allowed list
is_allowed_port(port, allowed_ports) if {
    port == allowed_ports[_]
}

# Check if CIDR block is too permissive
is_too_permissive(cidr) if {
    cidr == "0.0.0.0/0"
}

is_too_permissive(cidr) if {
    cidr == "::/0"
}

# Extract sensitivity level from tags
sensitivity_level(resource) := level if {
    level := resource.tags.sensitivity
} else := "standard"

# Check if resource is in production environment
is_production(resource) if {
    resource.tags.environment == "production"
}

is_production(resource) if {
    resource.tags.env == "prod"
}

# Build standardized error message
build_error(resource_type, resource_name, violation, recommendation, risk_id) := msg if {
    msg := sprintf(
        "%s '%s' %s. Recommendation: %s. Related Risk: %s",
        [resource_type, resource_name, violation, recommendation, risk_id]
    )
}
```

## Policy Documentation

Generate comprehensive documentation at `vault/security/policies.md`:

```markdown
# Security Policies

Generated: [TIMESTAMP]
Policy Engine: Open Policy Agent (OPA)
Policy Language: Rego

## Overview

This document catalogs all security policies enforced through Policy as Code.
Each policy is mapped to threat model risks and compliance frameworks.

## Policy Enforcement

Policies are enforced at multiple stages:

1. **Development**: Pre-commit hooks validate configurations
2. **CI/CD**: Pipeline checks block non-compliant deployments
3. **Runtime**: Admission controllers enforce in Kubernetes
4. **Audit**: Periodic scans verify compliance

## Policy Catalog

### Terraform Policies

#### Encryption at Rest (encryption.rego)

**Purpose**: Ensure all data storage resources have encryption enabled

**Threats Addressed**:
- [[threat-model#risk-002]]: Unencrypted Data at Rest
- [[threat-model#risk-007]]: Data Breach via Storage Access

**Compliance**:
- CWE-311: Missing Encryption of Sensitive Data
- ASVS V9.1.1: Verify that data at rest is encrypted
- NIST CSF PR.DS-1: Data-at-rest is protected

**Rules**:
1. **DENY**: S3 buckets without `server_side_encryption_configuration`
2. **DENY**: Encryption algorithms other than AES256 or aws:kms
3. **WARN**: Sensitive buckets using AES256 instead of KMS
4. **DENY**: RDS databases without `storage_encrypted = true`
5. **DENY**: EBS volumes without encryption

**Test Coverage**: `encryption_test.rego` - 12 test cases

**Usage**:
```bash
# Test policy
opa test infrastructure/policies/terraform/

# Evaluate against Terraform plan
terraform plan -out=tfplan.binary
terraform show -json tfplan.binary > tfplan.json
opa eval -i tfplan.json -d infrastructure/policies/terraform/ \
    "data.terraform.encryption.deny"
```

**Example Violation**:
```hcl
# ❌ FAILS - No encryption
resource "aws_s3_bucket" "data" {
  bucket = "my-data-bucket"
}

# ✅ PASSES - Encryption enabled
resource "aws_s3_bucket" "data" {
  bucket = "my-data-bucket"

  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "aws:kms"
        kms_master_key_id = aws_kms_key.s3.id
      }
    }
  }
}
```

---

#### Network Security (networking.rego)

**Purpose**: Enforce network segmentation and access controls

**Threats Addressed**:
- [[threat-model#risk-003]]: Unrestricted Network Access
- [[threat-model#risk-005]]: Lateral Movement

**Compliance**:
- CWE-284: Improper Access Control
- ASVS V1.4.1: Verify that trusted enforcement points exist
- NIST CSF PR.AC-5: Network integrity is protected

**Rules**:
1. **DENY**: Security groups with 0.0.0.0/0 on sensitive ports (22, 3389, 3306, 5432)
2. **DENY**: Security groups allowing all traffic from internet
3. **WARN**: Security groups with overly broad CIDR ranges (>/16)
4. **DENY**: VPC without flow logs enabled
5. **DENY**: Public RDS instances

**Test Coverage**: `networking_test.rego` - 15 test cases

---

### Kubernetes Policies

#### Pod Security (pod-security.rego)

**Purpose**: Enforce secure pod configurations and prevent privilege escalation

**Threats Addressed**:
- [[threat-model#risk-004]]: Container Escape
- [[threat-model#risk-006]]: Privilege Escalation

**Compliance**:
- CWE-250: Execution with Unnecessary Privileges
- ASVS V14.2.1: Verify application deployment uses least privilege
- Kubernetes Pod Security Standards: Restricted

**Rules**:
1. **DENY**: Privileged containers (`privileged: true`)
2. **DENY**: Containers running as root (UID 0)
3. **DENY**: Host network, PID, or IPC namespace sharing
4. **DENY**: Writable root filesystem
5. **DENY**: Dangerous capabilities (SYS_ADMIN, NET_ADMIN)
6. **WARN**: Missing resource limits
7. **DENY**: hostPath volumes

**Test Coverage**: `pod-security_test.rego` - 20 test cases

**Example Violation**:
```yaml
# ❌ FAILS - Privileged container
apiVersion: v1
kind: Pod
metadata:
  name: unsafe-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    securityContext:
      privileged: true

# ✅ PASSES - Secure configuration
apiVersion: v1
kind: Pod
metadata:
  name: safe-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: myapp:latest
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
    resources:
      limits:
        memory: "512Mi"
        cpu: "500m"
```

---

### Docker Policies

#### Dockerfile Security (dockerfile-security.rego)

**Purpose**: Ensure Docker images are built securely

**Threats Addressed**:
- [[threat-model#risk-008]]: Supply Chain Attack
- [[threat-model#risk-009]]: Vulnerable Dependencies

**Compliance**:
- CWE-1104: Use of Unmaintained Third Party Components
- ASVS V14.2.2: Verify components are from trusted sources

**Rules**:
1. **DENY**: Base images from untrusted registries
2. **DENY**: Using `latest` tag for base images
3. **DENY**: Running as root (missing USER directive)
4. **WARN**: Missing HEALTHCHECK
5. **DENY**: Hardcoded secrets or credentials
6. **WARN**: Excessive RUN commands (consider multi-stage builds)

**Test Coverage**: `dockerfile-security_test.rego` - 10 test cases

---

## Policy Traceability Matrix

| Policy | Risk ID | Severity | CWE | ASVS | NIST CSF |
|--------|---------|----------|-----|------|----------|
| S3 Encryption | RISK-002 | High | CWE-311 | V9.1.1 | PR.DS-1 |
| RDS Encryption | RISK-002 | High | CWE-311 | V9.1.1 | PR.DS-1 |
| Security Group Rules | RISK-003 | High | CWE-284 | V1.4.1 | PR.AC-5 |
| Privileged Containers | RISK-004 | Critical | CWE-250 | V14.2.1 | PR.AC-4 |
| Container Root User | RISK-006 | High | CWE-250 | V14.2.1 | PR.AC-4 |
| Base Image Trust | RISK-008 | High | CWE-1104 | V14.2.2 | PR.DS-6 |

## Testing Policies

For comprehensive testing documentation including test structure, data-driven testing, mocking, coverage analysis, and CI/CD integration, see:

**[references/testing.md](./policy-as-code/references/testing.md)**

### Quick Testing Guide

All policies include comprehensive test suites using OPA's testing framework:

```bash
# Run all policy tests
opa test infrastructure/policies/ -v

# Run specific policy tests
opa test infrastructure/policies/terraform/encryption_test.rego -v

# Run with coverage report
opa test --coverage infrastructure/policies/

# Filter tests by pattern
opa test infrastructure/policies/ -v --run "encryption|network"
```

**Coverage Requirements**:
- Minimum 80% statement coverage
- Test both positive (should pass) and negative (should fail) cases
- Test edge cases and boundary conditions

## Integrating Policies

### Pre-commit Hook

Add to `.git/hooks/pre-commit`:
```bash
#!/bin/bash
# Validate Terraform files
terraform fmt -check -recursive || exit 1
opa test infrastructure/policies/ || exit 1
```

### CI/CD Pipeline

Add to `.github/workflows/security.yml`:
```yaml
- name: Run OPA Policy Tests
  run: opa test infrastructure/policies/ -v

- name: Validate Infrastructure
  run: |
    terraform plan -out=tfplan.binary
    terraform show -json tfplan.binary > tfplan.json
    opa eval -i tfplan.json -d infrastructure/policies/ \
      "data.terraform.deny" --fail-defined
```

### Kubernetes Admission Controller

Deploy OPA Gatekeeper:
```bash
kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/release-3.14/deploy/gatekeeper.yaml
```

Convert Rego policies to Gatekeeper ConstraintTemplates.

## Rego Language Guide

For comprehensive Rego language documentation including syntax, data types, operators, functions, and advanced features, see:

**[references/rego-language.md](./policy-as-code/references/rego-language.md)**

### Quick Reference: Basic Syntax

```rego
# Package declaration
package myapp.policy

# Import statements
import future.keywords.if
import future.keywords.in

# Rules (deny, allow, warn)
deny[msg] {
    # Conditions
    condition1
    condition2

    # Message
    msg := "Violation message"
}

# Helper functions
is_valid(x) if {
    x == "expected_value"
}

# Comprehensions
violations := [msg |
    resource := input.resources[_]
    not is_valid(resource)
    msg := sprintf("Invalid: %s", [resource.name])
]
```

### Common Patterns Quick Reference

For detailed policy patterns including RBAC, Kubernetes admission control, infrastructure policies, and optimization techniques, see:

**[references/policy-patterns.md](./policy-as-code/references/policy-patterns.md)**

**Checking for missing fields**:
```rego
deny[msg] {
    resource := input.resource[name]
    not resource.required_field
    msg := sprintf("Missing required_field in %s", [name])
}
```

**Iterating over arrays**:
```rego
deny[msg] {
    rule := input.rules[_]  # _ means any index
    rule.protocol == "http"
    msg := "Insecure protocol"
}
```

**Set operations**:
```rego
required := {"tag1", "tag2", "tag3"}
actual := {tag | input.tags[tag]}
missing := required - actual

deny[msg] {
    count(missing) > 0
    msg := sprintf("Missing tags: %v", [missing])
}
```

## Reference Documentation

This skill includes comprehensive reference documentation:

1. **[Rego Language Reference](./policy-as-code/references/rego-language.md)**
   - Complete language syntax and semantics
   - Data types, operators, and expressions
   - Functions, comprehensions, and modules
   - Built-in functions and metadata
   - Best practices and performance tips

2. **[Policy Patterns](./policy-as-code/references/policy-patterns.md)**
   - Foundational patterns (deny, allow, warn, defaults)
   - RBAC and ABAC patterns
   - Kubernetes admission control patterns
   - Infrastructure policy patterns (Terraform, Docker)
   - Helper functions and optimization techniques

3. **[Testing Guide](./policy-as-code/references/testing.md)**
   - Test structure and organization
   - Data-driven and parameterized testing
   - Mocking with the `with` keyword
   - Coverage analysis
   - CI/CD integration
   - Debugging and best practices

## Educational Resources

- [OPA Documentation](https://www.openpolicyagent.org/docs/) - Official documentation
- [Rego Playground](https://play.openpolicyagent.org/) - Interactive testing environment
- [OPA Policy Library](https://github.com/open-policy-agent/library) - Community examples
- [Rego Style Guide](https://github.com/StyraInc/rego-style-guide) - Best practices
- [OPA Gatekeeper Library](https://github.com/open-policy-agent/gatekeeper-library) - Kubernetes policies

## Maintenance

### Adding New Policies

1. Identify threat from threat model
2. Determine enforceable control
3. Write policy in appropriate domain directory
4. Write comprehensive tests
5. Document in policies.md
6. Update traceability matrix
7. Integrate into CI/CD

### Updating Existing Policies

1. Review threat model changes
2. Update policy rules
3. Update tests (ensure they fail first!)
4. Update documentation
5. Communicate changes to team

### Policy Review Schedule

- **Weekly**: Review policy violations in CI/CD
- **Monthly**: Review policy effectiveness
- **Quarterly**: Align policies with updated threat model
- **Annually**: Comprehensive policy audit

## Next Steps

After generating policies:

1. **Test Policies**: Run `opa test` to verify all tests pass
2. **Integrate CI/CD**: Add policy checks to pipeline
3. **Run Conftest**: Use `conjtest-testing` skill to validate infrastructure
4. **Train Team**: Share policies.md with development team
5. **Monitor**: Track policy violations and remediation
