# Rego Language Reference

This document provides comprehensive reference material for the Rego policy language used by Open Policy Agent (OPA).

## Purpose and Philosophy

Rego is OPA's declarative policy language designed for evaluating structured data like API requests and configuration files. It extends Datalog to support JSON document models, enabling policy authors to express desired rules as code focusing on "what" rather than "how."

**Key Design Principles:**
- Declarative syntax for expressing policy logic
- Purpose-built for complex hierarchical data structures
- Domain-agnostic language suitable for diverse use cases
- Optimized for JSON document models

## Data Types

### Scalar Values

**Strings:**
```rego
standard_string := "Hello, World!"
raw_string := `C:\path\to\file`  # No escape sequences
multiline := `Line 1
Line 2
Line 3`
```

**Numbers:**
```rego
integer := 42
negative := -100
decimal := 3.14159
scientific := 1.23e-4
```

**Booleans and Null:**
```rego
is_valid := true
is_invalid := false
missing := null
```

### Composite Values

**Arrays** (ordered, zero-indexed, allows duplicates):
```rego
servers := ["web1", "web2", "db1"]
first_server := servers[0]  # "web1"
mixed_array := [1, "two", true, null]
```

**Objects** (unordered key-value pairs):
```rego
config := {
    "host": "localhost",
    "port": 8080,
    "ssl": true
}
port := config.port  # 8080
```

**Sets** (unordered, unique values):
```rego
allowed_protocols := {"https", "ssh", "tls"}
```

Note: Sets serialize as arrays in JSON but maintain uniqueness semantics.

## Rules and Documents

Rules generate virtual document content. The basic structure follows:

```rego
<name> <key>? <value>? <body>?
```

### Complete Definitions

Complete rules produce single values:

```rego
# Simple constant
max_connections := 100

# Conditional rule
allow if {
    input.user == "admin"
}

# Rule with computed value
risk_score := score if {
    violations := count(input.violations)
    score := violations * 10
}
```

### Partial Definitions

Partial rules create sets or objects through union semantics:

```rego
# Partial set (collects all violations)
violations[msg] {
    resource := input.resources[_]
    not resource.encrypted
    msg := sprintf("Resource %s not encrypted", [resource.name])
}

# Partial object (builds mapping)
resource_status[name] := status {
    resource := input.resources[_]
    name := resource.name
    status := resource.state
}
```

### Default Values

The `default` keyword provides fallback values when all rule definitions fail:

```rego
default allow := false

allow if {
    input.role == "admin"
}

allow if {
    input.permissions[_] == "write"
}
```

## Variables and References

Variables appear in both rule heads and bodies, functioning simultaneously as inputs and outputs. They must be locally scoped and appear in non-negated equality expressions within their rule.

### Basic Variable Usage

```rego
# Variable in rule body
is_admin if {
    user := input.user
    user.role == "admin"
}

# Multiple variables
has_permission if {
    user := input.user
    permission := input.required_permission
    permission == user.permissions[_]
}
```

### References

Access nested documents using dot-notation or bracket-notation:

```rego
# Dot notation
hostname := sites[0].servers[1].hostname

# Bracket notation
port := config["database"]["port"]

# Mixed notation
value := data.policies.security["encryption-rules"].level
```

### Variable Keys in References

Enable collection iteration:

```rego
# Iterate over array indices
deny[msg] {
    server := input.servers[i]
    not server.encrypted
    msg := sprintf("Server %d not encrypted", [i])
}

# Iterate over object keys
deny[msg] {
    resource := input.resources[name]
    not resource.compliant
    msg := sprintf("Resource '%s' not compliant", [name])
}
```

### Anonymous Iterators

Use underscores (`_`) when you don't need the index/key value:

```rego
has_admin {
    user := input.users[_]  # Don't care about index
    user.role == "admin"
}
```

## Control Flow

### Negation

Express prohibited states using `not`:

```rego
deny[msg] {
    resource := input.resource
    not resource.encrypted
    msg := "Encryption required"
}

# Multiple negations
allow if {
    not input.user.suspended
    not input.user.expired
    input.user.active
}
```

**Safety requirement:** Negated expressions must reference variables bound elsewhere in the rule.

```rego
# SAFE: user is bound before negation
deny if {
    user := input.user
    not user.verified
}

# UNSAFE: missing_field never bound
deny if {
    not input.missing_field  # Error: unsafe variable
}
```

### Universal Quantification

The `every` keyword asserts a condition holds for all collection members:

```rego
# All servers must have encryption
all_encrypted if {
    every server in input.servers {
        server.encrypted == true
    }
}

# All ports must be in allowed list
valid_ports if {
    allowed := {80, 443, 8080}
    every port in input.open_ports {
        port in allowed
    }
}
```

Logically equivalent using `some` and negation:

```rego
# Same as: every x in collection { condition }
all_valid if {
    not any_invalid
}

any_invalid if {
    some x in input.collection
    not is_valid(x)
}
```

### Conditional Evaluation with `else`

Establish rule evaluation precedence:

```rego
# Evaluated in order, stops at first match
authorization := "admin" if {
    input.user.role == "administrator"
} else := "user" if {
    input.user.authenticated
} else := "guest"
```

## Operators and Expressions

### Assignment Operator (`:=`)

Binds variables locally:

```rego
compute_score if {
    base := input.base_value
    multiplier := 1.5
    score := base * multiplier
    score > 100
}
```

### Comparison Operators

Require pre-assigned variables:

```rego
# Equality
is_production if { input.env == "production" }

# Inequality
not_test if { input.env != "test" }

# Numeric comparison
high_risk if {
    score := input.risk_score
    score > 80
}

# All comparison operators
# ==  (equal)
# != (not equal)
# <   (less than)
# <=  (less than or equal)
# >   (greater than)
# >=  (greater than or equal)
```

### Unification Operator (`=`)

Combines assignment and comparison capabilities:

```rego
# Unifies variable with value
match_pattern if {
    input.type = "security_group"
}

# Pattern matching
extract_name if {
    {"name": n, "type": "server"} = input.resource
    # n is now bound to the name value
}
```

### The `in` Operator

#### Membership Testing

```rego
is_allowed_protocol if {
    input.protocol in {"https", "ssh", "tls"}
}

# Array membership
is_primary if {
    input.server in ["web1", "web2", "db1"]
}
```

#### Collection Iteration

```rego
# Iterate array elements
find_admins[user] {
    some user in input.users
    user.role == "admin"
}

# Iterate with index
check_order[msg] {
    some i, value in input.sequence
    value != i
    msg := sprintf("Position %d has wrong value", [i])
}

# Iterate object key-value pairs
validate_config[msg] {
    some key, value in input.config
    not is_valid_setting(key, value)
    msg := sprintf("Invalid setting: %s = %v", [key, value])
}
```

## Comprehensions

Construct composite values from sub-queries.

### Array Comprehensions

```rego
# Simple transformation
server_names := [name | server := input.servers[_]; name := server.name]

# With filtering
encrypted_servers := [s |
    s := input.servers[_]
    s.encrypted == true
]

# Complex computation
risk_scores := [score |
    resource := input.resources[_]
    violations := count_violations(resource)
    score := violations * 10
]
```

### Object Comprehensions

```rego
# Map names to status
status_map := {name: status |
    server := input.servers[_]
    name := server.name
    status := server.state
}

# Computed keys and values
risk_levels := {resource.id: level |
    resource := input.resources[_]
    level := compute_risk(resource)
}
```

### Set Comprehensions

```rego
# Collect unique values
unique_owners := {owner |
    resource := input.resources[_]
    owner := resource.owner
}

# Filter and collect
high_risk_ids := {id |
    resource := input.resources[_]
    resource.risk_score > 80
    id := resource.id
}
```

### Nested Comprehensions

```rego
# All IP addresses from all servers
all_ips := {ip |
    server := input.servers[_]
    ip := server.ip_addresses[_]
}

# Complex nested logic
violation_summary := {resource_id: violations |
    resource := input.resources[_]
    resource_id := resource.id
    violations := [v |
        check := resource.checks[_]
        not check.passed
        v := check.name
    ]
}
```

## Functions

User-defined functions accept arbitrary term arguments and produce exactly one output.

### Basic Function Definition

```rego
# Simple function
min(a, b) := a if a < b
min(a, b) := b if a >= b

# Use the function
smallest := min(10, 20)  # 10
```

### Functions with Multiple Parameters

```rego
# Calculate risk score
risk_score(violations, severity) := score if {
    base := count(violations)
    multiplier := severity_multiplier(severity)
    score := base * multiplier
}

severity_multiplier("critical") := 10
severity_multiplier("high") := 5
severity_multiplier("medium") := 2
severity_multiplier("low") := 1
severity_multiplier(_) := 0  # Default case
```

### Incremental Function Definitions

Multiple definitions with conditional matching:

```rego
# Pattern matching on argument values
classify_port(port) := "http" if port == 80
classify_port(port) := "https" if port == 443
classify_port(port) := "ssh" if port == 22
classify_port(port) := "unknown"  # Catch-all

# Complex conditions
is_compliant(resource) if {
    resource.encrypted
    resource.monitored
    has_valid_tags(resource)
}

is_compliant(resource) if {
    resource.exempted
    has_approval(resource)
}
```

## The `with` Keyword

Substitutes values into expressions, useful for testing:

```rego
# Original rule
allow if {
    input.user.role == "admin"
    input.resource.public == false
}

# Test with mocked input
test_admin_access {
    allow with input as {
        "user": {"role": "admin"},
        "resource": {"public": false}
    }
}

# Mock specific data path
test_with_mock_data {
    result := check_permissions with data.roles as {
        "admin": ["read", "write", "delete"]
    }
    "write" in result
}
```

## Modules and Organization

### Package Declaration

Organize policies through packages (namespaced rule groups):

```rego
package terraform.security.encryption

# All rules in this file belong to:
# data.terraform.security.encryption
```

### Imports

Declare dependencies on other packages:

```rego
package myapp.policies

# Import entire package
import data.common.helpers

# Import with alias
import data.common.helpers as utils

# Import specific rule
import data.common.helpers.is_production

# Use future language features
import future.keywords.contains
import future.keywords.if
import future.keywords.in
```

### The `some` Keyword

Explicitly declare local variables in rules:

```rego
# Required for variable-key references
deny[msg] {
    some i
    server := input.servers[i]
    not server.encrypted
    msg := sprintf("Server %d not encrypted", [i])
}

# Multiple variable declarations
check_access {
    some user, permission
    user := input.users[_]
    permission := user.permissions[_]
    permission == "admin"
}

# With `in` operator
find_violations {
    some resource in input.resources
    not resource.compliant
}
```

## Built-in Functions

OPA provides extensive built-in functions organized by namespace.

### String Functions

```rego
# Formatting
msg := sprintf("Error: %s at line %d", [error.type, error.line])

# Manipulation
lower_name := lower(input.name)
upper_env := upper(input.environment)

# Pattern matching
matches := regex.match(`^[a-z]+$`, input.username)
```

### Array Functions

```rego
# Aggregation
total := sum([1, 2, 3, 4])  # 10
maximum := max([5, 2, 8, 1])  # 8

# Collection operations
items := count(input.resources)  # Number of elements
combined := array.concat([1, 2], [3, 4])  # [1, 2, 3, 4]
```

### Object Functions

```rego
# Extract keys
keys := object.keys({"a": 1, "b": 2})  # ["a", "b"]

# Merge objects
merged := object.union(
    {"a": 1, "b": 2},
    {"b": 3, "c": 4}
)  # {"a": 1, "b": 3, "c": 4}

# Filter
filtered := object.filter(
    input.config,
    ["allowed_key1", "allowed_key2"]
)
```

### Set Operations

```rego
# Union
all := {"a", "b"} | {"b", "c"}  # {"a", "b", "c"}

# Intersection
common := {"a", "b", "c"} & {"b", "c", "d"}  # {"b", "c"}

# Difference
missing := {"a", "b", "c"} - {"b"}  # {"a", "c"}
```

### Type Checking

```rego
is_string(input.name)
is_number(input.port)
is_boolean(input.enabled)
is_array(input.servers)
is_object(input.config)
is_set(input.protocols)
is_null(input.optional_field)
```

## Metadata Annotations

Rules and packages support rich YAML annotations:

```rego
# METADATA
# title: S3 Bucket Encryption
# description: |
#   Ensures all S3 buckets have encryption enabled.
#   This is required for compliance with security policy.
# authors:
#   - Security Team <security@example.com>
# related_resources:
#   - https://docs.aws.amazon.com/s3/encryption.html
# custom:
#   severity: high
#   frameworks:
#     - NIST-CSF-PR.DS-1
#     - CWE-311
deny[msg] {
    bucket := input.resource.aws_s3_bucket[name]
    not bucket.server_side_encryption_configuration
    msg := sprintf("S3 bucket '%s' lacks encryption", [name])
}
```

## Schema Annotations

Enhance static type checking using JSON Schema:

```rego
# METADATA
# schemas:
#   - input: schema.input
package myapp.authz

# Schema definition in separate file or inline
# METADATA
# schemas:
#   - input:
#       type: object
#       properties:
#         user:
#           type: object
#           properties:
#             role:
#               type: string
#         resource:
#           type: object
deny[msg] {
    input.user.role != "admin"
    msg := "Admin access required"
}
```

## Key Language Semantics

**Conjunction (AND):**
Rule bodies express conjunction—all expressions must evaluate true:

```rego
# All three conditions must be true
allow if {
    input.user.authenticated     # AND
    input.user.role == "admin"   # AND
    not input.resource.locked    # AND
}
```

**Disjunction (OR):**
Incrementally defined rules express disjunction:

```rego
# Any of these rules can make allow true
allow if {
    input.user.role == "admin"
}

allow if {
    input.user.role == "owner"
    input.resource.owner == input.user.id
}

allow if {
    input.resource.public == true
}
```

**Existential Quantification:**
Variables are existentially quantified by default—queries succeed if *some* binding exists satisfying all conditions:

```rego
# Succeeds if ANY server is encrypted
has_encrypted_server if {
    server := input.servers[_]
    server.encrypted == true
}
```

**Expression Reordering:**
OPA automatically reorders expressions for optimization and safety compliance. Variables must be bound before use:

```rego
# OPA will evaluate in safe order
safe_rule if {
    x > 10              # Will be reordered after x := ...
    x := input.value    # Binds x first
}
```

## Error Handling

By default, runtime errors yield `undefined`. Use `--strict-builtin-errors` flag to convert errors to exceptions.

```rego
# Safe navigation
value := object.get(input, "nested.path", "default")

# Explicit error handling
result := parse_result if {
    parsed := json.unmarshal(input.json_string)
    result := parsed
} else := {"error": "Invalid JSON"}
```

## Best Practices

1. **Use future keywords:** Import `future.keywords.*` for modern syntax
2. **Name rules clearly:** Use descriptive names that indicate purpose
3. **Add metadata:** Document rules with title, description, and related info
4. **Prefer comprehensions:** More readable than complex rule chains
5. **Factor out helpers:** Reuse logic in functions and helper rules
6. **Use `some` explicitly:** Make variable declarations clear
7. **Avoid deep nesting:** Flatten logic with helper rules
8. **Test edge cases:** Include tests for boundary conditions
9. **Use schemas:** Enable type checking for early error detection
10. **Profile performance:** Use `opa eval --profile` for optimization

## Additional Resources

- [OPA Documentation](https://www.openpolicyagent.org/docs/)
- [Rego Playground](https://play.openpolicyagent.org/) - Interactive testing
- [Rego Style Guide](https://github.com/StyraInc/rego-style-guide)
- [OPA Policy Library](https://github.com/open-policy-agent/library)
