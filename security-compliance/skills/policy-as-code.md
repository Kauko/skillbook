---
name: policy-as-code
description: Use when user wants to write OPA/Rego policies, enforce security rules programmatically, or automate compliance checks.
requires:
  tools: [opa]
  skills: [threagile-analysis]
---

# Policy as Code with OPA/Rego

## Prerequisites

```bash
command -v opa >/dev/null || { echo "Install: brew install opa"; exit 1; }
```

## Workflow

### 1. Read Threat Model

```bash
cat vault/security/risks.json
```

Identify enforceable mitigations.

### 2. Create Policy

`policies/security/encryption.rego`:
```rego
package security.encryption

default allow = false

# Deny unencrypted storage
deny[msg] {
    input.resource.type == "aws_s3_bucket"
    not input.resource.server_side_encryption_configuration
    msg := sprintf("S3 bucket %s must have encryption enabled", [input.resource.name])
}

# Deny unencrypted databases
deny[msg] {
    input.resource.type == "aws_db_instance"
    input.resource.storage_encrypted == false
    msg := sprintf("RDS %s must have storage encryption", [input.resource.name])
}

# Require TLS 1.2+
deny[msg] {
    input.resource.type == "aws_lb_listener"
    input.resource.ssl_policy
    not regex.match("TLS-1-[2-3]", input.resource.ssl_policy)
    msg := "Load balancer must use TLS 1.2 or higher"
}
```

### 3. Test Policy

```bash
# Unit test
opa test policies/

# Against sample data
opa eval -d policies/ -i test/fixtures/s3.json "data.security.encryption.deny"
```

### 4. Integrate with CI

```bash
# With Conftest
conftest test infrastructure/terraform/ -p policies/
```

## Policy Patterns

**Require encryption:**
```rego
deny[msg] {
    resource_needs_encryption(input.resource)
    not has_encryption(input.resource)
    msg := "Encryption required"
}
```

**Restrict network access:**
```rego
deny[msg] {
    input.resource.type == "aws_security_group_rule"
    input.resource.cidr_blocks[_] == "0.0.0.0/0"
    input.resource.from_port <= 22
    input.resource.to_port >= 22
    msg := "SSH must not be open to internet"
}
```

**Require tags:**
```rego
required_tags := {"Environment", "Owner", "Project"}

deny[msg] {
    missing := required_tags - {tag | input.resource.tags[tag]}
    count(missing) > 0
    msg := sprintf("Missing tags: %v", [missing])
}
```

## File Organization

```
policies/
├── security/
│   ├── encryption.rego
│   ├── network.rego
│   └── iam.rego
├── compliance/
│   └── tagging.rego
└── tests/
    └── *_test.rego
```

## Reference Documentation

- `references/rego-syntax.md` - Rego language
- `references/policy-patterns.md` - Common patterns

## Success Criteria

- [ ] Rego policies created in `policies/` directory
- [ ] `opa test policies/` passes
- [ ] Policies enforce identified risks from threat model

## Related Skills

- `conftest-testing` - Policy testing
- `threagile-analysis` - Risk source
