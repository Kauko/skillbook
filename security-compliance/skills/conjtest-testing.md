---
description: Use when validating infrastructure configs against security policies. Uses Conjtest (Clojure Conftest wrapper).
---

# Conftest Testing with Conjtest

## Overview

Conjtest is a Clojure wrapper for Conftest (OPA policy testing tool), enabling seamless integration of security policy validation into Clojure/Clojurescript projects.

Reference: https://github.com/ilmoraunio/conjtest

## Prerequisites Check

Verify Conjtest is available in project dependencies:

1. Check `deps.edn` for Conjtest:
   ```bash
   grep -A 2 "ilmoraunio/conjtest" deps.edn
   ```

2. If not found, guide addition to `deps.edn`:
   ```clojure
   {:deps {ilmoraunio/conjtest {:mvn/version "0.1.0"}}}
   ```

   Or as an alias for testing:
   ```clojure
   {:aliases
    {:test {:extra-deps {ilmoraunio/conjtest {:mvn/version "0.1.0"}}}}}
   ```

3. Verify Conftest is installed (Conjtest requires it):
   ```bash
   which conftest
   ```

4. If Conftest not found, guide installation:
   ```bash
   # macOS
   brew install conftest

   # Linux
   wget https://github.com/open-policy-agent/conftest/releases/latest/download/conftest_Linux_x86_64.tar.gz
   tar xzf conftest_Linux_x86_64.tar.gz
   sudo mv conftest /usr/local/bin/

   # Windows
   # Download from: https://github.com/open-policy-agent/conftest/releases/latest
   ```

5. Verify installations:
   ```bash
   conftest --version
   clj -M:test -e "(require '[conjtest.core :as conjtest]) (println \"Conjtest ready\")"
   ```

## Configure Conjtest

### Policy Directory Configuration

Set up Conjtest to use policies from `infrastructure/policies/`:

Create or update `conftest.toml` in project root:

```toml
# Conftest Configuration
# Policies are organized by infrastructure domain

[[namespaces]]
name = "terraform"
policy = "infrastructure/policies/terraform"

[[namespaces]]
name = "kubernetes"
policy = "infrastructure/policies/kubernetes"

[[namespaces]]
name = "docker"
policy = "infrastructure/policies/docker"

# Additional settings
[general]
output = "json"
trace = false
```

### Conjtest Test Configuration

Create test namespace: `test/security/policy_test.clj`:

```clojure
(ns security.policy-test
  (:require [clojure.test :refer [deftest is testing]]
            [conjtest.core :as conjtest]
            [clojure.java.io :as io]
            [clojure.data.json :as json]))

(defn policy-dir []
  "infrastructure/policies")

(defn test-config []
  {:policy-dir (policy-dir)
   :output "json"
   :trace false})
```

## Test Infrastructure Configurations

### Test Terraform Plans

Create comprehensive test suite for Terraform:

```clojure
(ns security.terraform-policy-test
  (:require [clojure.test :refer [deftest is testing]]
            [conjtest.core :as conjtest]
            [clojure.java.io :as io]
            [clojure.data.json :as json]
            [clojure.string :as str]))

(defn terraform-plan-json []
  "Path to Terraform plan JSON output"
  "infrastructure/terraform/tfplan.json")

(defn generate-terraform-plan []
  "Generate Terraform plan for testing"
  (let [tf-dir "infrastructure/terraform"]
    (println "Generating Terraform plan...")
    (shell/sh "terraform" "init" :dir tf-dir)
    (shell/sh "terraform" "plan" "-out=tfplan.binary" :dir tf-dir)
    (shell/sh "terraform" "show" "-json" "tfplan.binary" :dir tf-dir
              :out (io/file (terraform-plan-json)))))

(deftest test-terraform-encryption-policies
  (testing "Terraform encryption policies"
    (when-not (.exists (io/file (terraform-plan-json)))
      (generate-terraform-plan))

    (let [result (conjtest/test-file
                   (terraform-plan-json)
                   {:policy "infrastructure/policies/terraform"
                    :namespace "terraform.encryption"
                    :output "json"})
          violations (:failures result)]

      (testing "S3 buckets have encryption enabled"
        (is (empty? (filter #(str/includes? % "S3 bucket") violations))
            "All S3 buckets must have encryption enabled"))

      (testing "RDS instances have encryption enabled"
        (is (empty? (filter #(str/includes? % "RDS") violations))
            "All RDS instances must have storage encryption"))

      (testing "No weak encryption algorithms"
        (is (empty? (filter #(str/includes? % "weak encryption") violations))
            "Only strong encryption algorithms (AES256, aws:kms) allowed"))

      (when (seq violations)
        (println "\n❌ Encryption Policy Violations:")
        (doseq [v violations]
          (println "  -" v))))))

(deftest test-terraform-networking-policies
  (testing "Terraform networking policies"
    (let [result (conjtest/test-file
                   (terraform-plan-json)
                   {:policy "infrastructure/policies/terraform"
                    :namespace "terraform.networking"
                    :output "json"})
          violations (:failures result)]

      (testing "No unrestricted security group rules"
        (is (empty? (filter #(str/includes? % "0.0.0.0/0") violations))
            "Security groups must not allow unrestricted access"))

      (testing "Sensitive ports are protected"
        (is (empty? (filter #(re-find #"port (22|3389|3306|5432)" %) violations))
            "Sensitive ports (SSH, RDP, databases) must not be exposed"))

      (testing "VPC flow logs are enabled"
        (is (empty? (filter #(str/includes? % "flow logs") violations))
            "All VPCs must have flow logging enabled"))

      (when (seq violations)
        (println "\n❌ Networking Policy Violations:")
        (doseq [v violations]
          (println "  -" v))))))
```

### Test Kubernetes Manifests

```clojure
(ns security.kubernetes-policy-test
  (:require [clojure.test :refer [deftest is testing]]
            [conjtest.core :as conjtest]
            [clojure.java.io :as io]
            [clojure.string :as str]))

(defn k8s-manifests-dir []
  "infrastructure/kubernetes/manifests")

(defn find-k8s-manifests []
  "Find all Kubernetes YAML manifests"
  (->> (file-seq (io/file (k8s-manifests-dir)))
       (filter #(.isFile %))
       (filter #(or (.endsWith (.getName %) ".yaml")
                    (.endsWith (.getName %) ".yml")))
       (map #(.getPath %))))

(deftest test-kubernetes-pod-security-policies
  (testing "Kubernetes pod security policies"
    (doseq [manifest (find-k8s-manifests)]
      (testing (str "Validating " manifest)
        (let [result (conjtest/test-file
                       manifest
                       {:policy "infrastructure/policies/kubernetes"
                        :namespace "kubernetes.pod-security"
                        :output "json"})
              violations (:failures result)]

          (testing "No privileged containers"
            (is (empty? (filter #(str/includes? % "privileged") violations))
                "Containers must not run in privileged mode"))

          (testing "Containers do not run as root"
            (is (empty? (filter #(str/includes? % "root") violations))
                "Containers must not run as UID 0"))

          (testing "Read-only root filesystem"
            (is (empty? (filter #(str/includes? % "writable") violations))
                "Root filesystem should be read-only"))

          (testing "Resource limits defined"
            (is (empty? (filter #(str/includes? % "resource limits") violations))
                "All containers must have resource limits"))

          (when (seq violations)
            (println (str "\n❌ Policy Violations in " manifest ":"))
            (doseq [v violations]
              (println "  -" v))))))))

(deftest test-kubernetes-network-policies
  (testing "Kubernetes network policies"
    (let [namespaces-dir "infrastructure/kubernetes/namespaces"
          namespaces (->> (file-seq (io/file namespaces-dir))
                          (filter #(.isFile %))
                          (map #(.getPath %)))]

      (doseq [ns-file namespaces]
        (testing (str "Validating namespace " ns-file)
          (let [result (conjtest/test-file
                         ns-file
                         {:policy "infrastructure/policies/kubernetes"
                          :namespace "kubernetes.network-policy"
                          :output "json"})
                violations (:failures result)]

            (testing "Network policies defined"
              (is (empty? violations)
                  "All namespaces must have network policies"))

            (when (seq violations)
              (println (str "\n❌ Network Policy Violations in " ns-file ":"))
              (doseq [v violations]
                (println "  -" v)))))))))
```

### Test Dockerfiles

```clojure
(ns security.docker-policy-test
  (:require [clojure.test :refer [deftest is testing]]
            [conjtest.core :as conjtest]
            [clojure.java.io :as io]
            [clojure.string :as str]))

(defn find-dockerfiles []
  "Find all Dockerfiles in the project"
  (->> (file-seq (io/file "."))
       (filter #(.isFile %))
       (filter #(or (= (.getName %) "Dockerfile")
                    (.startsWith (.getName %) "Dockerfile.")))
       (map #(.getPath %))))

(deftest test-dockerfile-security-policies
  (testing "Dockerfile security policies"
    (doseq [dockerfile (find-dockerfiles)]
      (testing (str "Validating " dockerfile)
        (let [result (conjtest/test-file
                       dockerfile
                       {:policy "infrastructure/policies/docker"
                        :namespace "docker.dockerfile-security"
                        :output "json"})
              violations (:failures result)]

          (testing "Base images from trusted registries"
            (is (empty? (filter #(str/includes? % "untrusted") violations))
                "Base images must be from approved registries"))

          (testing "No 'latest' tags"
            (is (empty? (filter #(str/includes? % "latest") violations))
                "Base images must have specific version tags"))

          (testing "Non-root user"
            (is (empty? (filter #(str/includes? % "root") violations))
                "Dockerfile must include USER directive"))

          (testing "No hardcoded secrets"
            (is (empty? (filter #(str/includes? % "secret") violations))
                "No hardcoded credentials allowed"))

          (when (seq violations)
            (println (str "\n❌ Dockerfile Violations in " dockerfile ":"))
            (doseq [v violations]
              (println "  -" v))))))))
```

## Report Policy Violations

### Generate Compliance Report

Create reporting namespace: `src/security/compliance_report.clj`:

```clojure
(ns security.compliance-report
  (:require [conjtest.core :as conjtest]
            [clojure.java.io :as io]
            [clojure.string :as str]
            [clojure.data.json :as json]))

(defn test-all-infrastructure []
  "Run all policy tests and aggregate results"
  (let [terraform-result (conjtest/test-file
                           "infrastructure/terraform/tfplan.json"
                           {:policy "infrastructure/policies/terraform"})
        k8s-results (map #(conjtest/test-file
                            %
                            {:policy "infrastructure/policies/kubernetes"})
                         (find-k8s-manifests))
        docker-results (map #(conjtest/test-file
                               %
                               {:policy "infrastructure/policies/docker"})
                            (find-dockerfiles))]

    {:terraform terraform-result
     :kubernetes k8s-results
     :docker docker-results
     :timestamp (java.time.Instant/now)}))

(defn violation-severity [violation]
  "Determine severity based on violation message"
  (cond
    (or (str/includes? violation "privileged")
        (str/includes? violation "root")
        (str/includes? violation "0.0.0.0/0"))
    :critical

    (or (str/includes? violation "encryption")
        (str/includes? violation "authentication")
        (str/includes? violation "secret"))
    :high

    (or (str/includes? violation "resource limits")
        (str/includes? violation "network policy"))
    :medium

    :else :low))

(defn extract-risk-id [violation]
  "Extract risk ID from violation message"
  (when-let [match (re-find #"RISK-\d+" violation)]
    match))

(defn link-to-threat-model [violation]
  "Create link to threat model documentation"
  (if-let [risk-id (extract-risk-id violation)]
    (str "vault/security/threat-model.md#" (str/lower-case risk-id))
    "vault/security/threat-model.md"))

(defn link-to-policy [violation domain]
  "Create link to policy documentation"
  (str "vault/security/policies.md#" domain "-policies"))

(defn format-violation [violation domain]
  "Format violation with links and metadata"
  {:violation violation
   :severity (violation-severity violation)
   :risk-id (extract-risk-id violation)
   :threat-link (link-to-threat-model violation)
   :policy-link (link-to-policy violation domain)
   :domain domain})

(defn generate-markdown-report [results output-file]
  "Generate markdown compliance report"
  (let [all-violations (flatten
                         (concat
                           (map #(format-violation % "terraform")
                                (:failures (:terraform results)))
                           (mapcat #(map (fn [v] (format-violation v "kubernetes"))
                                         (:failures %))
                                   (:kubernetes results))
                           (mapcat #(map (fn [v] (format-violation v "docker"))
                                         (:failures %))
                                   (:docker results))))
        by-severity (group-by :severity all-violations)
        critical (get by-severity :critical [])
        high (get by-severity :high [])
        medium (get by-severity :medium [])
        low (get by-severity :low [])]

    (spit output-file
          (str
            "# Security Policy Compliance Report\n\n"
            "Generated: " (:timestamp results) "\n"
            "Policy Engine: Conftest + OPA\n"
            "Test Framework: Conjtest\n\n"

            "## Executive Summary\n\n"
            "Total Violations: " (count all-violations) "\n"
            "- Critical: " (count critical) "\n"
            "- High: " (count high) "\n"
            "- Medium: " (count medium) "\n"
            "- Low: " (count low) "\n\n"

            (when (empty? all-violations)
              "✅ **All infrastructure configurations comply with security policies!**\n\n")

            "## Compliance Status\n\n"
            "| Domain | Violations | Status |\n"
            "|--------|------------|--------|\n"
            "| Terraform | "
            (count (filter #(= "terraform" (:domain %)) all-violations))
            " | " (if (empty? (filter #(= "terraform" (:domain %)) all-violations))
                    "✅ Compliant" "❌ Non-compliant") " |\n"
            "| Kubernetes | "
            (count (filter #(= "kubernetes" (:domain %)) all-violations))
            " | " (if (empty? (filter #(= "kubernetes" (:domain %)) all-violations))
                    "✅ Compliant" "❌ Non-compliant") " |\n"
            "| Docker | "
            (count (filter #(= "docker" (:domain %)) all-violations))
            " | " (if (empty? (filter #(= "docker" (:domain %)) all-violations))
                    "✅ Compliant" "❌ Non-compliant") " |\n\n"

            (when (seq critical)
              (str "## Critical Violations (Immediate Action Required)\n\n"
                   (str/join "\n"
                             (map-indexed
                               (fn [idx v]
                                 (str (inc idx) ". **" (:violation v) "**\n"
                                      "   - Domain: " (:domain v) "\n"
                                      "   - Risk: [" (or (:risk-id v) "General") "]("
                                      (:threat-link v) ")\n"
                                      "   - Policy: [" (str/capitalize (:domain v))
                                      " Policies](" (:policy-link v) ")\n"))
                               critical))
                   "\n\n"))

            (when (seq high)
              (str "## High Severity Violations\n\n"
                   (str/join "\n"
                             (map-indexed
                               (fn [idx v]
                                 (str (inc idx) ". " (:violation v) "\n"
                                      "   - Domain: " (:domain v) "\n"
                                      "   - Risk: [" (or (:risk-id v) "General") "]("
                                      (:threat-link v) ")\n"
                                      "   - Policy: [" (str/capitalize (:domain v))
                                      " Policies](" (:policy-link v) ")\n"))
                               high))
                   "\n\n"))

            (when (seq medium)
              (str "## Medium Severity Violations\n\n"
                   (str/join "\n"
                             (map-indexed
                               (fn [idx v]
                                 (str (inc idx) ". " (:violation v) "\n"
                                      "   - Domain: " (:domain v) "\n"))
                               medium))
                   "\n\n"))

            (when (seq low)
              (str "## Low Severity Violations\n\n"
                   (str/join "\n"
                             (map-indexed
                               (fn [idx v]
                                 (str (inc idx) ". " (:violation v) "\n"))
                               low))
                   "\n\n"))

            "## Remediation Priority\n\n"
            "### Phase 1: Critical (This Sprint)\n"
            (str/join "\n" (map #(str "- [ ] " (:violation %)) critical))
            "\n\n"
            "### Phase 2: High (Next Sprint)\n"
            (str/join "\n" (map #(str "- [ ] " (:violation %)) high))
            "\n\n"
            "### Phase 3: Medium (Next Quarter)\n"
            (str/join "\n" (map #(str "- [ ] " (:violation %)) medium))
            "\n\n"

            "## Continuous Compliance\n\n"
            "This report is generated automatically by running:\n"
            "```bash\n"
            "clj -M:test -m security.compliance-report\n"
            "```\n\n"
            "Policies are enforced in:\n"
            "1. Pre-commit hooks\n"
            "2. CI/CD pipeline\n"
            "3. Kubernetes admission controllers\n"
            "4. Periodic audits\n\n"

            "## Related Documentation\n\n"
            "- [[threat-model|Threat Model]]\n"
            "- [[policies|Security Policies]]\n"
            "- [[decisions|Security ADRs]]\n"))))

(defn -main []
  "Generate compliance report"
  (println "Running security policy compliance checks...")
  (let [results (test-all-infrastructure)
        output-file "vault/security/policy-compliance.md"]
    (generate-markdown-report results output-file)
    (println (str "✓ Compliance report generated: " output-file))
    (System/exit (if (empty? (flatten
                               (map :failures
                                    (concat [(:terraform results)]
                                            (:kubernetes results)
                                            (:docker results)))))
                   0
                   1))))
```

## Integration with CI/CD

### GitHub Actions

Create `.github/workflows/security-compliance.yml`:

```yaml
name: Security Policy Compliance

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  policy-compliance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Install Conftest
        run: |
          wget https://github.com/open-policy-agent/conftest/releases/latest/download/conftest_Linux_x86_64.tar.gz
          tar xzf conftest_Linux_x86_64.tar.gz
          sudo mv conftest /usr/local/bin/

      - name: Install Clojure
        uses: DeLaGuardo/setup-clojure@master
        with:
          cli: latest

      - name: Generate Terraform Plan
        run: |
          cd infrastructure/terraform
          terraform init
          terraform plan -out=tfplan.binary
          terraform show -json tfplan.binary > tfplan.json

      - name: Run Policy Compliance Tests
        run: clj -M:test -m security.compliance-report

      - name: Upload Compliance Report
        uses: actions/upload-artifact@v3
        with:
          name: compliance-report
          path: vault/security/policy-compliance.md

      - name: Comment PR with Results
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v6
        with:
          script: |
            const fs = require('fs');
            const report = fs.readFileSync('vault/security/policy-compliance.md', 'utf8');
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: report
            });
```

## Document Compliance Results

The generated `vault/security/policy-compliance.md` provides:

1. **Executive Summary**: Overview of compliance status
2. **Violation Details**: Categorized by severity with links
3. **Traceability**: Each violation linked to:
   - Threat model risk
   - Policy documentation
   - Compliance framework
4. **Remediation Plan**: Prioritized action items
5. **Continuous Monitoring**: Instructions for ongoing compliance

## Best Practices

### Test Organization
- Group tests by infrastructure domain (Terraform, K8s, Docker)
- Use descriptive test names that explain what is being validated
- Include both positive and negative test cases
- Run tests in CI/CD on every commit

### Policy Maintenance
- Keep policies synchronized with threat model
- Update tests when policies change
- Document policy rationale in comments
- Version policies alongside infrastructure code

### Reporting
- Generate compliance reports regularly (daily/weekly)
- Track violation trends over time
- Share reports with security and development teams
- Use reports to prioritize remediation work

### Integration
- Enforce policies in pre-commit hooks for fast feedback
- Block deployments that violate critical policies
- Use warnings for medium/low severity violations
- Provide clear remediation guidance in violation messages

## Troubleshooting

### Conjtest Not Found
```bash
# Verify deps.edn includes conjtest
clj -Spath | grep conjtest

# Update dependencies
clj -P
```

### Conftest Execution Errors
```bash
# Test policies directly with conftest
conftest test infrastructure/terraform/tfplan.json \
  --policy infrastructure/policies/terraform

# Verify policy syntax
conftest verify --policy infrastructure/policies/
```

### Policy Not Applied
```bash
# Check conftest.toml configuration
cat conftest.toml

# Verify policy directory exists
ls -la infrastructure/policies/
```

### Test Failures
```bash
# Run tests with verbose output
clj -M:test -m security.policy-test --verbose

# Check individual policy
opa test infrastructure/policies/terraform/encryption_test.rego -v
```

## Next Steps

After validating infrastructure:

1. **Review Violations**: Analyze compliance report with team
2. **Prioritize Fixes**: Address critical violations immediately
3. **Update Infrastructure**: Apply fixes to configurations
4. **Retest**: Run compliance tests to verify fixes
5. **Monitor**: Schedule regular compliance scans
6. **Improve Policies**: Refine policies based on findings
7. **Educate Team**: Share results and best practices

## Summary Output

Always provide a summary after running compliance tests:

```
✓ Security Policy Compliance Tests Complete

Results:
- Terraform: 3 violations (2 high, 1 medium)
- Kubernetes: 0 violations
- Docker: 1 violation (1 low)

Report Generated: vault/security/policy-compliance.md

Critical Actions Required:
1. Enable S3 bucket encryption (RISK-002)
2. Restrict security group 0.0.0.0/0 rules (RISK-003)

Next Steps:
- Review vault/security/policy-compliance.md
- Address critical violations in current sprint
- Re-run tests after fixes: clj -M:test -m security.compliance-report
```
