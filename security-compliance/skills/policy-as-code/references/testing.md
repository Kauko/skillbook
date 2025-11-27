# Testing Rego Policies

This document provides comprehensive guidance on testing OPA/Rego policies using the built-in testing framework.

## Overview

Open Policy Agent provides a built-in testing framework for Rego policies that helps developers verify correctness and speed up development cycles. The framework supports:

- Unit testing individual rules
- Integration testing policy modules
- Data-driven/parameterized testing
- Mocking data and functions
- Coverage analysis
- Multiple output formats

## Test Structure

### Basic Test Format

Tests follow a naming convention with a "test_" prefix and are typically placed in files suffixed with "_test.rego":

```rego
package myapp.authz

# Production rule
allow if {
    input.user.role == "admin"
}

# Test rule
test_admin_allowed {
    allow with input as {"user": {"role": "admin"}}
}

test_non_admin_denied {
    not allow with input as {"user": {"role": "user"}}
}
```

### Test File Organization

```
policies/
├── authz.rego           # Production policy
├── authz_test.rego      # Tests for authz.rego
├── encryption.rego
└── encryption_test.rego
```

**Convention:** Place tests in the same package as the code under test, in separate `*_test.rego` files.

## Running Tests

### Basic Test Execution

```bash
# Run all tests in current directory
opa test .

# Run with verbose output
opa test . -v

# Run specific test files
opa test policies/authz_test.rego -v

# Run tests in multiple directories
opa test policies/ common/ -v
```

### Filtering Tests

Use regex patterns to run specific tests:

```bash
# Run only tests matching pattern (re2 syntax)
opa test . -v --run test_admin

# Run tests containing "encryption" or "network"
opa test . -v --run "encryption|network"

# Run all tests except performance tests
opa test . -v --run "^(?!.*performance).*$"
```

### Output Formats

```bash
# Human-readable output (default)
opa test . -v

# JSON output for CI/CD integration
opa test . --format=json

# Pretty JSON
opa test . --format=json | jq
```

### Fail on Empty

Ensure tests actually execute:

```bash
# Exit with error if no tests found
opa test . --fail-on-empty
```

## Test Results

The framework categorizes outcomes as:

| Result | Condition | Exit Code |
|--------|-----------|-----------|
| **PASS** | Rule evaluates to true | 0 |
| **FAIL** | Rule undefined or returns non-true value | 1 |
| **ERROR** | Runtime error encountered | 1 |
| **SKIPPED** | Rules prefixed with "todo_" | 0 |

### Example Test Output

```
policies/authz_test.rego:
data.myapp.authz.test_admin_allowed: PASS (2.5ms)
data.myapp.authz.test_non_admin_denied: PASS (1.8ms)
data.myapp.authz.test_invalid_role: FAIL (1.2ms)
data.myapp.authz.test_missing_field: ERROR (0.9ms)
--------------------------------------------------------------------------------
PASS: 2/4
FAIL: 1/4
ERROR: 1/4
```

## Writing Effective Tests

### Test Positive Cases (Should Pass)

```rego
package terraform.encryption

# Test: Bucket with encryption should pass
test_s3_bucket_with_encryption {
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

    # Should produce no violations
    count(result) == 0
}
```

### Test Negative Cases (Should Fail)

```rego
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

    # Should produce violation
    count(result) > 0
}
```

### Test Specific Error Messages

```rego
# Test: Verify exact error message
test_encryption_error_message {
    result := deny with input as {
        "resource": {
            "aws_s3_bucket": {
                "test_bucket": {
                    "bucket": "my-bucket"
                }
            }
        }
    }

    # Check specific message content
    count(result) == 1
    contains(result[_], "test_bucket")
    contains(result[_], "encryption")
}
```

### Test Edge Cases

```rego
# Test: Empty input
test_empty_input {
    result := deny with input as {}
    count(result) == 0
}

# Test: Null values
test_null_encryption_config {
    result := deny with input as {
        "resource": {
            "aws_s3_bucket": {
                "test_bucket": {
                    "bucket": "my-bucket",
                    "server_side_encryption_configuration": null
                }
            }
        }
    }
    count(result) > 0
}

# Test: Missing fields
test_missing_resource_field {
    result := deny with input as {
        "resource": {}
    }
    count(result) == 0  # Should handle gracefully
}
```

### Test Boundary Conditions

```rego
# Test: Exactly at threshold
test_risk_score_at_threshold {
    risk := compute_risk_score with input as {
        "violations": 5  # Exactly at critical threshold
    }
    risk == "high"
}

# Test: Just below threshold
test_risk_score_below_threshold {
    risk := compute_risk_score with input as {
        "violations": 4
    }
    risk == "medium"
}

# Test: Just above threshold
test_risk_score_above_threshold {
    risk := compute_risk_score with input as {
        "violations": 6
    }
    risk == "critical"
}
```

## Data-Driven Testing

### Parameterized Tests with Inline Data

```rego
package kubernetes.security

# Test data
test_cases := [
    {
        "name": "privileged_container",
        "input": {
            "containers": [{
                "name": "app",
                "securityContext": {"privileged": true}
            }]
        },
        "expect_violation": true
    },
    {
        "name": "secure_container",
        "input": {
            "containers": [{
                "name": "app",
                "securityContext": {"privileged": false}
            }]
        },
        "expect_violation": false
    },
    {
        "name": "no_security_context",
        "input": {
            "containers": [{
                "name": "app"
            }]
        },
        "expect_violation": false
    }
]

# Parameterized test
test_privileged_containers {
    some case in test_cases
    result := deny with input as case.input

    case.expect_violation == (count(result) > 0)
}
```

### Test Data from External Files

**test_data/valid_buckets.json:**
```json
[
    {
        "name": "encrypted-bucket",
        "config": {
            "server_side_encryption_configuration": {
                "rule": {
                    "apply_server_side_encryption_by_default": {
                        "sse_algorithm": "AES256"
                    }
                }
            }
        }
    }
]
```

**encryption_test.rego:**
```rego
package terraform.encryption

import data.test_data.valid_buckets

test_valid_buckets_pass {
    some bucket in valid_buckets
    result := deny with input.resource.aws_s3_bucket as {
        bucket.name: bucket.config
    }
    count(result) == 0
}
```

### Table-Driven Tests

```rego
package network.validation

# Define test table
port_tests := {
    "http": {"port": 80, "secure": false},
    "https": {"port": 443, "secure": true},
    "ssh": {"port": 22, "secure": true},
    "telnet": {"port": 23, "secure": false},
    "custom_secure": {"port": 8443, "secure": true}
}

# Test all cases
test_port_security {
    some name, test_case in port_tests
    result := is_secure_port(test_case.port)
    result == test_case.secure
}
```

## Mocking with `with` Keyword

### Mock Input Data

```rego
# Original rule uses input.user
allow if {
    input.user.role == "admin"
}

# Test with mocked input
test_admin_access {
    allow with input.user as {"role": "admin"}
}

# Mock nested paths
test_nested_mock {
    allow with input.user.role as "admin"
         with input.user.department as "security"
}
```

### Mock Data Documents

```rego
# Policy depends on data.roles
allow if {
    user_role := input.user.role
    permissions := data.roles[user_role]
    input.action in permissions
}

# Test with mocked data
test_with_mock_data {
    allow with input as {"user": {"role": "admin"}, "action": "write"}
         with data.roles as {
            "admin": ["read", "write", "delete"],
            "user": ["read"]
         }
}
```

### Mock Functions

```rego
# Original rule
allow if {
    current_time := time.now_ns()
    is_business_hours(current_time)
}

# Test with mocked time
test_business_hours {
    # Mock time.now_ns() to return 2pm (14:00)
    business_hours_time := 1640008800000000000  # Unix nano timestamp for 2pm

    result := allow with time.now_ns as business_hours_time
    result == true
}

test_after_hours {
    # Mock time.now_ns() to return 8pm (20:00)
    after_hours_time := 1640030400000000000

    result := allow with time.now_ns as after_hours_time
    result == false
}
```

### Mock Multiple Dependencies

```rego
test_complex_scenario {
    result := allow
        with input.user as {"id": "user123", "role": "editor"}
        with input.resource as {"id": "doc456", "owner": "user123"}
        with data.permissions as {"editor": ["read", "write"]}
        with data.resources as {"doc456": {"sensitivity": "low"}}

    result == true
}
```

## Coverage Analysis

### Generate Coverage Report

```bash
# Basic coverage report
opa test --coverage .

# Coverage with verbose output
opa test --coverage --format=json . | jq

# HTML coverage report
opa test --coverage --format=json . > coverage.json
# Use external tool to visualize
```

### Example Coverage Output

```
--------------------------------------------------------------
policies/authz.rego:
  Lines covered: 45/50 (90.0%)
  Lines not covered: 6, 12, 23, 31, 42

policies/encryption.rego:
  Lines covered: 38/40 (95.0%)
  Lines not covered: 15, 28

--------------------------------------------------------------
Total coverage: 92.2%
```

### Improve Coverage

Identify untested code paths:

```rego
# Original policy
deny[msg] {
    resource := input.resource[name]
    not resource.encrypted
    msg := sprintf("Resource %s not encrypted", [name])
}

deny[msg] {
    resource := input.resource[name]
    resource.encryption_algorithm == "DES"  # Not covered by tests!
    msg := sprintf("Resource %s uses weak encryption", [name])
}

# Add test for uncovered path
test_weak_encryption_algorithm {
    result := deny with input.resource as {
        "test": {
            "encrypted": true,
            "encryption_algorithm": "DES"
        }
    }
    count(result) > 0
    contains(result[_], "weak encryption")
}
```

## Advanced Testing Techniques

### Testing Helper Functions

```rego
package common.helpers

# Helper function
is_secure_tls(version) if {
    version in {"TLSv1.2", "TLSv1.3"}
}

# Test helper directly
test_secure_tls_versions {
    is_secure_tls("TLSv1.2")
    is_secure_tls("TLSv1.3")
}

test_insecure_tls_versions {
    not is_secure_tls("TLSv1.0")
    not is_secure_tls("TLSv1.1")
    not is_secure_tls("SSLv3")
}
```

### Testing Comprehensions

```rego
# Rule with comprehension
violation_count := count([resource |
    resource := input.resources[_]
    not resource.compliant
])

# Test comprehension
test_violation_count {
    result := violation_count with input.resources as [
        {"name": "r1", "compliant": true},
        {"name": "r2", "compliant": false},
        {"name": "r3", "compliant": false}
    ]
    result == 2
}
```

### Testing with Tracing

Enable detailed execution traces:

```bash
# Run with explanation trace
opa test . -v --explain=full

# Save trace to file
opa test . --explain=full > trace.txt
```

### Performance Testing

```bash
# Run benchmarks
opa test --bench .

# Multiple iterations for statistical analysis
opa test --bench --count=10 .

# Export results
opa test --bench --format=gobench . > bench.txt
```

## Testing Anti-Patterns

### Don't Test Implementation Details

```rego
# ❌ BAD: Testing internal helper behavior
test_internal_helper {
    _internal_helper_function(input) == expected
}

# ✅ GOOD: Test public API behavior
test_policy_behavior {
    allow with input as test_input
}
```

### Don't Mock Everything

```rego
# ❌ BAD: Over-mocking makes test meaningless
test_overmocked {
    allow with input as mock_input
         with data as mock_data
         with is_valid as true  # Mocking the logic itself!
         with check_permission as true
}

# ✅ GOOD: Mock only external dependencies
test_reasonable_mocking {
    allow with input as realistic_input
         with data.external_service as mock_external_data
}
```

### Don't Skip Error Cases

```rego
# ❌ BAD: Only testing happy path
test_valid_input {
    result := process(valid_input)
    result == expected
}

# ✅ GOOD: Test error handling
test_invalid_input {
    result := process(invalid_input)
    result == error_response
}

test_missing_fields {
    result := process({})
    result == validation_error
}
```

### Write Focused Tests

```rego
# ❌ BAD: Testing multiple concerns
test_everything {
    allow  # Tests authorization
    count(deny) == 0  # Tests validation
    audit_log  # Tests logging
}

# ✅ GOOD: One test, one concern
test_admin_authorization {
    allow with input.user.role as "admin"
}

test_validation_passes {
    result := deny with input as valid_resource
    count(result) == 0
}

test_audit_logging {
    logs := audit_log with input as test_input
    count(logs) > 0
}
```

## CI/CD Integration

### GitHub Actions Example

```yaml
name: OPA Policy Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup OPA
        uses: open-policy-agent/setup-opa@v2
        with:
          version: latest

      - name: Run Policy Tests
        run: opa test policies/ -v --fail-on-empty

      - name: Check Coverage
        run: |
          opa test policies/ --coverage --format=json > coverage.json
          COVERAGE=$(jq '.coverage' coverage.json)
          echo "Coverage: $COVERAGE%"
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "Coverage below 80% threshold"
            exit 1
          fi
```

### GitLab CI Example

```yaml
opa-tests:
  image: openpolicyagent/opa:latest
  script:
    - opa test policies/ -v --fail-on-empty
    - opa test policies/ --coverage --format=json > coverage.json
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage.json
  coverage: '/Total coverage: (\d+\.\d+)%/'
```

### Makefile Integration

```makefile
.PHONY: test test-verbose test-coverage

test:
	opa test policies/ --fail-on-empty

test-verbose:
	opa test policies/ -v

test-coverage:
	opa test policies/ --coverage --format=json | \
		jq -r '.files[] | "\(.path): \(.covered_lines)/\(.not_covered_lines + .covered_lines)"'

test-ci:
	opa test policies/ -v --fail-on-empty --format=json > test-results.json
```

## Test Organization Best Practices

### 1. Group Related Tests

```rego
package terraform.encryption

# S3 encryption tests
test_s3_no_encryption { ... }
test_s3_with_aes256 { ... }
test_s3_with_kms { ... }

# RDS encryption tests
test_rds_no_encryption { ... }
test_rds_with_encryption { ... }

# EBS encryption tests
test_ebs_no_encryption { ... }
test_ebs_with_encryption { ... }
```

### 2. Use Descriptive Test Names

```rego
# ✅ GOOD: Clear intent
test_admin_can_delete_any_resource { ... }
test_user_cannot_delete_others_resources { ... }
test_guest_cannot_delete_any_resources { ... }

# ❌ BAD: Unclear intent
test_case1 { ... }
test_case2 { ... }
test_case3 { ... }
```

### 3. Test Naming Convention

```
test_<rule_name>_<scenario>_<expected_result>

Examples:
- test_allow_admin_with_valid_token_returns_true
- test_deny_missing_encryption_generates_violation
- test_risk_score_high_violations_returns_critical
```

### 4. Shared Test Fixtures

```rego
package test.fixtures

# Shared test data
valid_admin_user := {
    "id": "admin123",
    "role": "admin",
    "permissions": ["read", "write", "delete"]
}

valid_user := {
    "id": "user456",
    "role": "user",
    "permissions": ["read", "write"]
}

encrypted_s3_bucket := {
    "bucket": "test-bucket",
    "server_side_encryption_configuration": {
        "rule": {
            "apply_server_side_encryption_by_default": {
                "sse_algorithm": "AES256"
            }
        }
    }
}
```

```rego
package terraform.encryption

import data.test.fixtures

test_encrypted_bucket_passes {
    result := deny with input.resource.aws_s3_bucket as {
        "test": fixtures.encrypted_s3_bucket
    }
    count(result) == 0
}
```

## Debugging Failed Tests

### Use Enriched Reports

```bash
# Show variable values that caused failures
opa test . -v --var-values
```

### Use Print Statements

```rego
test_debug_values {
    result := deny with input as test_input

    # Debug output
    print("Input:", test_input)
    print("Result:", result)
    print("Count:", count(result))

    count(result) == 0
}
```

Run with verbose flag to see print output:
```bash
opa test . -v
```

### Use Trace for Deep Debugging

```bash
# Full execution trace
opa test . --explain=full

# Notes trace (high-level)
opa test . --explain=notes
```

## Skipping Tests

Mark tests as TODO/skipped:

```rego
# Test will be skipped
todo_test_future_feature {
    # Not implemented yet
    false
}

# Regular test runs
test_current_feature {
    true
}
```

Output shows skipped tests:
```
data.myapp.todo_test_future_feature: SKIPPED
data.myapp.test_current_feature: PASS
```

## Testing Checklist

When writing tests for a policy, ensure you cover:

- [ ] Happy path (valid input passes)
- [ ] Invalid input is rejected
- [ ] Edge cases (empty, null, missing fields)
- [ ] Boundary conditions (thresholds, limits)
- [ ] Error handling
- [ ] Each branch of conditional logic
- [ ] Each deny/allow/warn rule
- [ ] Helper functions independently
- [ ] Integration with real-world-like data
- [ ] Performance with large datasets (if applicable)

## Additional Resources

- [OPA Testing Documentation](https://www.openpolicyagent.org/docs/latest/policy-testing/)
- [Regal Linter](https://github.com/StyraInc/regal) - Linting and code quality
- [OPA Playground](https://play.openpolicyagent.org/) - Interactive testing
- [Conftest](https://www.conftest.dev/) - Policy testing for configuration files
