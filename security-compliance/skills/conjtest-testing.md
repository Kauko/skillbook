---
name: conftest-testing
description: Use when user wants to validate Terraform, Kubernetes, or Docker configs against security policies using Conftest.
requires:
  tools: [conftest]
  skills: [policy-as-code]
---

# Conftest Policy Testing

## Prerequisites

```bash
command -v conftest >/dev/null || { echo "Install: brew install conftest"; exit 1; }
```

For Clojure wrapper (optional):
```clojure
{:deps {ilmoraunio/conjtest {:mvn/version "0.1.0"}}}
```

## Usage

### Test Terraform

```bash
conftest test infrastructure/terraform/ -p policies/
```

### Test Kubernetes

```bash
conftest test k8s/*.yaml -p policies/kubernetes/
```

### Test Dockerfile

```bash
conftest test Dockerfile -p policies/docker/
```

## Writing Tests

`policies/terraform/encryption_test.rego`:
```rego
package terraform.encryption

test_s3_encrypted {
    deny with input as {"resource": {"type": "aws_s3_bucket", "server_side_encryption_configuration": {}}}
    count(deny) == 0
}

test_s3_unencrypted {
    deny with input as {"resource": {"type": "aws_s3_bucket"}}
    count(deny) > 0
}
```

Run tests:
```bash
conftest verify -p policies/
```

## Clojure Integration

```clojure
(ns policy-test
  (:require [conjtest.core :as ct]
            [clojure.test :refer [deftest is]]))

(deftest terraform-policies
  (let [results (ct/test "infrastructure/terraform/" {:policy "policies/"})]
    (is (empty? (:failures results)))))
```

## CI Integration

```yaml
- name: Policy check
  run: |
    conftest test infrastructure/terraform/ -p policies/ --no-fail
    conftest verify -p policies/
```

## Output Formats

```bash
conftest test . -p policies/ -o json    # JSON
conftest test . -p policies/ -o table   # Table
conftest test . -p policies/ -o tap     # TAP
```

## Reference Documentation

- `references/api.md` - Conjtest API
- `references/integration.md` - CI/CD setup

## Success Criteria

- [ ] `conftest test` runs against target configs
- [ ] `conftest verify` passes policy unit tests
- [ ] Violations reported with actionable messages

## Related Skills

- `policy-as-code` - Writing OPA policies
- `terraform-iac` - Infrastructure to test
