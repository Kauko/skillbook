# Common Policy Patterns

This document provides proven patterns for writing effective OPA/Rego policies across different domains and use cases.

## Foundational Patterns

### Deny Rules Pattern

The most common pattern for validation policies. Collect all violations and report them.

```rego
package security.validation

# Deny pattern: Generate set of violation messages
deny[msg] {
    condition_that_should_not_exist
    msg := "Description of what went wrong"
}

# Multiple deny rules accumulate violations
deny[msg] {
    another_bad_condition
    msg := "Another violation message"
}
```

**Example: Require encryption**

```rego
package terraform.security

deny[msg] {
    resource := input.resource.aws_s3_bucket[name]
    not resource.server_side_encryption_configuration
    msg := sprintf("S3 bucket '%s' must have encryption enabled", [name])
}

deny[msg] {
    resource := input.resource.aws_rds_instance[name]
    not resource.storage_encrypted
    msg := sprintf("RDS instance '%s' must have storage encryption", [name])
}
```

### Allow Rules Pattern

Positive authorization pattern. Only explicitly allowed actions succeed.

```rego
package authz

# Default deny
default allow := false

# Allow specific conditions
allow if {
    input.user.role == "admin"
}

allow if {
    input.user.role == "editor"
    input.action in {"read", "write"}
}

allow if {
    input.resource.public == true
    input.action == "read"
}
```

### Warning Rules Pattern

Non-blocking recommendations that don't fail validation.

```rego
package security.recommendations

warn[msg] {
    resource := input.resource.aws_s3_bucket[name]
    not resource.versioning[_].enabled
    msg := sprintf("Consider enabling versioning on bucket '%s'", [name])
}

warn[msg] {
    resource := input.resource.aws_s3_bucket[name]
    resource.tags.sensitivity == "high"
    encryption := resource.server_side_encryption_configuration[_]
    encryption.rule[_].apply_server_side_encryption_by_default[_].sse_algorithm == "AES256"
    msg := sprintf("Bucket '%s' is sensitive but uses AES256. Consider aws:kms for better audit trails", [name])
}
```

### Default Values Pattern

Establish baseline behavior with overridable defaults.

```rego
package network.policy

# Default deny all traffic
default allow_traffic := false

# Allow specific traffic patterns
allow_traffic if {
    input.source.zone == "trusted"
    input.destination.zone == "dmz"
    input.protocol in {"https", "ssh"}
}

# Default risk level
default risk_level := "medium"

# Override based on conditions
risk_level := "critical" if {
    input.violations.count > 10
}

risk_level := "high" if {
    input.violations.count > 5
}

risk_level := "low" if {
    input.violations.count == 0
}
```

## RBAC Patterns

### Basic Role-Based Access Control

```rego
package rbac.authz

import future.keywords.if
import future.keywords.in

# Role definitions
roles := {
    "admin": ["read", "write", "delete"],
    "editor": ["read", "write"],
    "viewer": ["read"]
}

# Check if user has permission
allow if {
    user_permissions := roles[input.user.role]
    input.action in user_permissions
}
```

### Hierarchical RBAC

```rego
package rbac.hierarchical

# Role hierarchy: each role inherits from roles below it
role_hierarchy := {
    "admin": ["admin", "manager", "user", "guest"],
    "manager": ["manager", "user", "guest"],
    "user": ["user", "guest"],
    "guest": ["guest"]
}

# Permissions per role
permissions := {
    "admin": ["create", "read", "update", "delete", "configure"],
    "manager": ["create", "read", "update", "delete"],
    "user": ["create", "read", "update"],
    "guest": ["read"]
}

# User has permission if any inherited role grants it
allow if {
    user_role := input.user.role
    inherited_roles := role_hierarchy[user_role]
    some role in inherited_roles
    role_permissions := permissions[role]
    input.action in role_permissions
}
```

### Attribute-Based Access Control (ABAC)

```rego
package abac.authz

# Combine role, resource attributes, and context
allow if {
    # Role check
    input.user.role in {"admin", "manager"}

    # Resource ownership
    input.resource.owner == input.user.id

    # Department match
    input.user.department == input.resource.department

    # Time-based access
    is_business_hours
}

is_business_hours if {
    hour := time.clock(time.now_ns())[0]
    hour >= 9
    hour < 17
}
```

### Resource-Specific RBAC

```rego
package rbac.resources

# Different permissions for different resource types
allow if {
    resource_type := input.resource.type
    user_role := input.user.role
    action := input.action

    has_permission(resource_type, user_role, action)
}

# Document permissions
has_permission("document", "admin", _)  # Admin can do anything
has_permission("document", "owner", action) if {
    action in {"read", "write", "delete"}
}
has_permission("document", "collaborator", action) if {
    action in {"read", "write"}
}
has_permission("document", "viewer", "read")

# Database permissions
has_permission("database", "admin", _)
has_permission("database", "dba", action) if {
    action in {"read", "write", "backup", "restore"}
}
has_permission("database", "developer", action) if {
    action in {"read", "write"}
}
has_permission("database", "analyst", "read")
```

## Kubernetes Admission Control Patterns

### Pod Security Standards

```rego
package kubernetes.admission

import future.keywords.if
import future.keywords.in

# Deny privileged containers
deny[msg] {
    some container in input.request.object.spec.containers
    container.securityContext.privileged
    msg := sprintf("Container '%s' uses privileged mode", [container.name])
}

# Deny running as root
deny[msg] {
    not input.request.object.spec.securityContext.runAsNonRoot
    msg := "Pods must set runAsNonRoot: true"
}

deny[msg] {
    some container in input.request.object.spec.containers
    container.securityContext.runAsUser == 0
    msg := sprintf("Container '%s' runs as root (UID 0)", [container.name])
}

# Deny dangerous capabilities
dangerous_capabilities := {"SYS_ADMIN", "NET_ADMIN", "SYS_MODULE"}

deny[msg] {
    some container in input.request.object.spec.containers
    some cap in container.securityContext.capabilities.add
    cap in dangerous_capabilities
    msg := sprintf("Container '%s' adds dangerous capability '%s'", [container.name, cap])
}

# Deny writable root filesystem
deny[msg] {
    some container in input.request.object.spec.containers
    not container.securityContext.readOnlyRootFilesystem
    msg := sprintf("Container '%s' must use readOnlyRootFilesystem", [container.name])
}

# Deny host namespace sharing
deny[msg] {
    input.request.object.spec.hostNetwork
    msg := "Pods cannot use host network"
}

deny[msg] {
    input.request.object.spec.hostPID
    msg := "Pods cannot use host PID namespace"
}

deny[msg] {
    input.request.object.spec.hostIPC
    msg := "Pods cannot use host IPC namespace"
}
```

### Resource Requirements

```rego
package kubernetes.resources

deny[msg] {
    some container in input.request.object.spec.containers
    not container.resources.limits.memory
    msg := sprintf("Container '%s' must specify memory limits", [container.name])
}

deny[msg] {
    some container in input.request.object.spec.containers
    not container.resources.limits.cpu
    msg := sprintf("Container '%s' must specify CPU limits", [container.name])
}

deny[msg] {
    some container in input.request.object.spec.containers
    not container.resources.requests.memory
    msg := sprintf("Container '%s' must specify memory requests", [container.name])
}

deny[msg] {
    some container in input.request.object.spec.containers
    not container.resources.requests.cpu
    msg := sprintf("Container '%s' must specify CPU requests", [container.name])
}
```

### Image Registry Restrictions

```rego
package kubernetes.images

# Allowed container registries
allowed_registries := {"docker.io/mycompany", "gcr.io/mycompany", "internal.registry.com"}

deny[msg] {
    some container in input.request.object.spec.containers
    image := container.image
    not image_from_allowed_registry(image)
    msg := sprintf("Container '%s' uses untrusted registry: %s", [container.name, image])
}

image_from_allowed_registry(image) if {
    some registry in allowed_registries
    startswith(image, registry)
}

# Deny 'latest' tag
deny[msg] {
    some container in input.request.object.spec.containers
    image := container.image
    endswith(image, ":latest")
    msg := sprintf("Container '%s' uses 'latest' tag: %s", [container.name, image])
}

deny[msg] {
    some container in input.request.object.spec.containers
    image := container.image
    not contains(image, ":")
    msg := sprintf("Container '%s' uses implicit 'latest' tag: %s", [container.name, image])
}
```

### Required Labels

```rego
package kubernetes.labels

required_labels := ["app", "version", "owner", "environment"]

deny[msg] {
    provided := object.keys(input.request.object.metadata.labels)
    missing := required_labels - provided
    count(missing) > 0
    msg := sprintf("Missing required labels: %v", [missing])
}
```

## Terraform/Infrastructure Patterns

### Encryption at Rest

```rego
package terraform.encryption

deny[msg] {
    resource := input.resource.aws_s3_bucket[name]
    not has_encryption(resource)
    msg := sprintf("S3 bucket '%s' must enable server-side encryption", [name])
}

has_encryption(bucket) if {
    bucket.server_side_encryption_configuration
}

# Ensure strong encryption algorithms
deny[msg] {
    resource := input.resource.aws_s3_bucket[name]
    encryption := resource.server_side_encryption_configuration[_]
    rule := encryption.rule[_]
    algorithm := rule.apply_server_side_encryption_by_default[_].sse_algorithm
    not algorithm in {"AES256", "aws:kms"}
    msg := sprintf("S3 bucket '%s' uses weak encryption: %s", [name, algorithm])
}

# RDS encryption
deny[msg] {
    resource := input.resource.aws_rds_instance[name]
    not resource.storage_encrypted
    msg := sprintf("RDS instance '%s' must have storage_encrypted = true", [name])
}

# EBS encryption
deny[msg] {
    resource := input.resource.aws_ebs_volume[name]
    not resource.encrypted
    msg := sprintf("EBS volume '%s' must be encrypted", [name])
}
```

### Network Security

```rego
package terraform.network

# Sensitive ports that should not be public
sensitive_ports := {22, 3389, 3306, 5432, 5984, 6379, 8086, 9200, 27017}

deny[msg] {
    resource := input.resource.aws_security_group[name]
    rule := resource.ingress[_]
    rule.cidr_blocks[_] == "0.0.0.0/0"
    port := rule.from_port
    port in sensitive_ports
    msg := sprintf(
        "Security group '%s' allows public access to sensitive port %d",
        [name, port]
    )
}

# Deny public RDS instances
deny[msg] {
    resource := input.resource.aws_rds_instance[name]
    resource.publicly_accessible
    msg := sprintf("RDS instance '%s' must not be publicly accessible", [name])
}

# Require VPC flow logs
deny[msg] {
    resource := input.resource.aws_vpc[name]
    not has_flow_logs(name)
    msg := sprintf("VPC '%s' must have flow logs enabled", [name])
}

has_flow_logs(vpc_name) if {
    some log in input.resource.aws_flow_log
    log.vpc_id == sprintf("${aws_vpc.%s.id}", [vpc_name])
}
```

### Tagging Requirements

```rego
package terraform.tagging

required_tags := ["Environment", "Owner", "CostCenter", "Project"]

deny[msg] {
    resource := input.resource[resource_type][name]
    is_taggable(resource_type)
    provided_tags := object.keys(resource.tags)
    missing := required_tags - provided_tags
    count(missing) > 0
    msg := sprintf(
        "%s '%s' missing required tags: %v",
        [resource_type, name, missing]
    )
}

# Resources that support tagging
is_taggable(type) if {
    type in {
        "aws_s3_bucket",
        "aws_rds_instance",
        "aws_ebs_volume",
        "aws_instance",
        "aws_vpc"
    }
}
```

## Docker/Dockerfile Patterns

### Base Image Security

```rego
package docker.images

# Trusted registries
trusted_registries := {
    "docker.io/library",
    "gcr.io/distroless",
    "company.registry.io"
}

deny[msg] {
    input.stages[_].from.image
    image := input.stages[_].from.image
    not from_trusted_registry(image)
    msg := sprintf("Untrusted base image: %s", [image])
}

from_trusted_registry(image) if {
    some registry in trusted_registries
    startswith(image, registry)
}

# Deny 'latest' tag
deny[msg] {
    image := input.stages[_].from.image
    endswith(image, ":latest")
    msg := sprintf("Base image uses 'latest' tag: %s", [image])
}

deny[msg] {
    image := input.stages[_].from.image
    not contains(image, ":")
    msg := sprintf("Base image uses implicit 'latest' tag: %s", [image])
}
```

### Dockerfile Best Practices

```rego
package docker.practices

# Require USER directive
deny[msg] {
    not has_user_directive
    msg := "Dockerfile must include USER directive to avoid running as root"
}

has_user_directive if {
    some cmd in input.stages[_].commands
    cmd.cmd == "user"
}

# Check for hardcoded secrets
secret_patterns := [
    "password",
    "secret",
    "api_key",
    "apikey",
    "token",
    "credential"
]

deny[msg] {
    some stage in input.stages
    some cmd in stage.commands
    cmd.cmd in {"env", "arg"}
    some pattern in secret_patterns
    contains(lower(cmd.value[0]), pattern)
    msg := sprintf("Potential hardcoded secret in %s: %s", [upper(cmd.cmd), cmd.value[0]])
}

# Recommend multi-stage builds
warn[msg] {
    count(input.stages) == 1
    count([cmd | cmd := input.stages[0].commands[_]; cmd.cmd == "run"]) > 5
    msg := "Consider using multi-stage build to reduce image size"
}
```

## Helper Function Patterns

### Reusable Validation Helpers

```rego
package common.helpers

import future.keywords.if
import future.keywords.in

# Check if resource has all required tags
has_required_tags(resource, required) if {
    provided := object.keys(resource.tags)
    missing := required - provided
    count(missing) == 0
}

# Check TLS version
is_secure_tls(version) if {
    version in {"TLSv1.2", "TLSv1.3"}
}

# Check CIDR is not too permissive
is_too_permissive(cidr) if {
    cidr in {"0.0.0.0/0", "::/0"}
}

# Extract sensitivity from tags
sensitivity_level(resource) := level if {
    level := resource.tags.sensitivity
} else := "standard"

# Check production environment
is_production(resource) if {
    lower(resource.tags.environment) in {"production", "prod"}
}

# Build standardized error messages
build_error(resource_type, resource_name, violation, recommendation) := msg if {
    msg := sprintf(
        "%s '%s': %s. Recommendation: %s",
        [resource_type, resource_name, violation, recommendation]
    )
}

# Parse CIDR notation
cidr_subnet_size(cidr) := size if {
    parts := split(cidr, "/")
    size := to_number(parts[1])
}

# Check if port is in allowed list
is_allowed_port(port, allowed_ports) if {
    port in allowed_ports
}
```

## Data Validation Patterns

### Schema Validation

```rego
package validation.schema

# Validate required fields exist
deny[msg] {
    required_fields := ["name", "type", "owner"]
    missing := [field |
        field := required_fields[_]
        not object.get(input, field, null)
    ]
    count(missing) > 0
    msg := sprintf("Missing required fields: %v", [missing])
}

# Validate field types
deny[msg] {
    not is_string(input.name)
    msg := "Field 'name' must be a string"
}

deny[msg] {
    not is_number(input.port)
    msg := "Field 'port' must be a number"
}

# Validate value ranges
deny[msg] {
    input.port < 1
    msg := "Port must be >= 1"
}

deny[msg] {
    input.port > 65535
    msg := "Port must be <= 65535"
}

# Validate enums
deny[msg] {
    valid_types := {"server", "client", "proxy"}
    not input.type in valid_types
    msg := sprintf("Invalid type '%s'. Must be one of: %v", [input.type, valid_types])
}
```

### List Operations

```rego
package validation.lists

# All items must satisfy condition
deny[msg] {
    some item in input.items
    not is_valid(item)
    msg := sprintf("Invalid item: %v", [item])
}

# At least one item must satisfy condition
deny[msg] {
    not any_valid
    msg := "At least one item must be valid"
}

any_valid if {
    some item in input.items
    is_valid(item)
}

# Count violations
violation_count := count([item |
    item := input.items[_]
    not is_valid(item)
])

deny[msg] {
    violation_count > 0
    msg := sprintf("%d invalid items found", [violation_count])
}
```

## Performance Optimization Patterns

### Indexed Lookups

```rego
package optimized.lookup

# ❌ SLOW: Linear search through array
is_authorized_slow if {
    some user in data.users  # Array iteration
    user.id == input.user_id
    user.role == "admin"
}

# ✅ FAST: Direct object lookup
is_authorized_fast if {
    user := data.users_by_id[input.user_id]  # O(1) lookup
    user.role == "admin"
}
```

### Early Exit Pattern

```rego
package optimized.early_exit

# ✅ GOOD: Returns as soon as condition is met
allow if {
    input.user.role == "admin"  # Single ground value
}

allow if {
    check_permissions  # More complex check only if not admin
}
```

### Comprehension Caching

```rego
package optimized.caching

# ❌ SLOW: Repeated computation
deny[msg] {
    resource := input.resources[_]
    violations := compute_violations(resource)
    count(violations) > 0
    msg := sprintf("%s has violations", [resource.name])
}

# ✅ FAST: Compute once
violations_by_resource := {name: violations |
    resource := input.resources[_]
    name := resource.name
    violations := compute_violations(resource)
}

deny[msg] {
    some name, violations in violations_by_resource
    count(violations) > 0
    msg := sprintf("%s has violations", [name])
}
```

## Testing Patterns

See `references/testing.md` for comprehensive testing patterns and examples.

## Integration Patterns

### CI/CD Pipeline Integration

```rego
package cicd.validation

# Gate deployment on policy compliance
deployment_allowed if {
    count(deny) == 0
    count(critical_warnings) == 0
}

critical_warnings[msg] {
    warn[msg]
    is_critical_warning(msg)
}

is_critical_warning(msg) if {
    contains(msg, "security")
}

is_critical_warning(msg) if {
    contains(msg, "compliance")
}
```

### Audit Logging

```rego
package audit.logging

# Generate audit log entries
audit_log[entry] {
    decision := allow
    entry := {
        "timestamp": time.now_ns(),
        "user": input.user,
        "action": input.action,
        "resource": input.resource,
        "decision": decision,
        "reason": reason
    }
}
```

## Additional Resources

- [OPA Policy Library](https://github.com/open-policy-agent/library) - Community policy examples
- [Rego Style Guide](https://github.com/StyraInc/rego-style-guide) - Best practices
- [OPA Gatekeeper Library](https://github.com/open-policy-agent/gatekeeper-library) - Kubernetes policies
- [Conftest Examples](https://github.com/open-policy-agent/conftest/tree/master/examples) - Infrastructure testing
