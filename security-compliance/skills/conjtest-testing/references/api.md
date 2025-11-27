# Conjtest Clojure API Reference

## Overview

Conjtest provides a Clojure API for programmatic policy validation of infrastructure configuration files. This enables seamless integration with Clojure test frameworks and build pipelines.

## Core Namespace

```clojure
(require '[conjtest.core :as conjtest])
```

## Main Functions

### `test-file`

Primary function for validating a configuration file against policies.

**Signature:**
```clojure
(test-file file-path options)
```

**Parameters:**
- `file-path` (String): Absolute or relative path to the configuration file to test
- `options` (Map): Configuration options for the test execution

**Options Map:**
- `:policy` (String): Path to policy directory or file
- `:namespace` (String, optional): Specific policy namespace to evaluate
- `:output` (String, optional): Output format - "json", "table", or "tap"
- `:trace` (Boolean, optional): Enable trace output for debugging

**Return Value:**

Returns a map containing test results:
```clojure
{:failures [vector of violation messages]
 :warnings [vector of warning messages]
 :successes integer-count
 :file "path/to/tested/file"}
```

**Example Usage:**
```clojure
(let [result (conjtest/test-file
               "infrastructure/terraform/tfplan.json"
               {:policy "infrastructure/policies/terraform"
                :namespace "terraform.encryption"
                :output "json"})]
  (when (seq (:failures result))
    (println "Policy violations found:")
    (doseq [failure (:failures result)]
      (println "  -" failure))))
```

## Policy Definition

### Pure Clojure Functions

Define policies as regular Clojure functions that evaluate configuration data:

```clojure
(ns policy)

(defn deny-privileged-containers [input]
  "Deny containers running in privileged mode"
  (when (get-in input [:spec :containers 0 :securityContext :privileged])
    "Container must not run in privileged mode"))

(defn deny-root-user [input]
  "Deny containers running as root (UID 0)"
  (when (= 0 (get-in input [:spec :containers 0 :securityContext :runAsUser]))
    "Container must not run as root user"))

(defn require-resource-limits [input]
  "Require resource limits on all containers"
  (let [containers (get-in input [:spec :containers])]
    (when-not (every? #(contains? % :resources) containers)
      "All containers must define resource limits")))
```

### Schema-Based Validation with Malli

Define policies using malli schemas for declarative validation:

```clojure
(ns policy
  (:require [malli.core :as m]))

(def kubernetes-pod-schema
  [:map
   [:apiVersion :string]
   [:kind [:enum "Pod" "Deployment" "StatefulSet"]]
   [:metadata
    [:map
     [:name :string]
     [:labels [:map-of :keyword :string]]]]
   [:spec
    [:map
     [:containers
      [:vector
       [:map
        [:name :string]
        [:image [:re #"^[a-z0-9-]+\.[a-z0-9.-]+/.*:[^:latest]$"]]
        [:securityContext
         [:map
          [:privileged [:= false]]
          [:runAsNonRoot [:= true]]
          [:readOnlyRootFilesystem [:= true]]]]
        [:resources
         [:map
          [:limits [:map [:cpu :string] [:memory :string]]]
          [:requests [:map [:cpu :string] [:memory :string]]]]]]]]]])

(defn validate-pod-schema [input]
  "Validate pod configuration against security schema"
  (when-not (m/validate kubernetes-pod-schema input)
    (m/explain kubernetes-pod-schema input)))
```

## Configuration File Parsing

Conjtest automatically detects and parses multiple configuration formats:

### Supported Formats

| Format | Extensions | Use Case |
|--------|-----------|----------|
| YAML | `.yaml`, `.yml` | Kubernetes, Ansible, Docker Compose |
| JSON | `.json` | Terraform plans, API configs |
| HCL1/HCL2 | `.tf` | Terraform infrastructure |
| TOML | `.toml` | Configuration files |
| EDN | `.edn` | Clojure data |
| Dockerfile | `Dockerfile*` | Container images |
| HOCON | `.conf` | Application configs |
| XML | `.xml` | Legacy configs |
| Jsonnet | `.jsonnet` | Templated configs |

### Format-Specific Considerations

**Terraform (HCL):**
```clojure
;; Test Terraform plan JSON output (recommended)
(conjtest/test-file "tfplan.json" {:policy "policies/terraform"})

;; Or test raw .tf files
(conjtest/test-file "main.tf" {:policy "policies/terraform"})
```

**Kubernetes YAML:**
```clojure
;; Single manifest
(conjtest/test-file "deployment.yaml" {:policy "policies/k8s"})

;; Multi-document YAML (separated by ---)
;; Conjtest parses each document separately
(conjtest/test-file "all-resources.yaml" {:policy "policies/k8s"})
```

**Dockerfile:**
```clojure
;; Standard Dockerfile
(conjtest/test-file "Dockerfile" {:policy "policies/docker"})

;; Named Dockerfiles
(conjtest/test-file "Dockerfile.production" {:policy "policies/docker"})
```

## Policy Return Values

### Denial Rules

Return a string message to indicate a violation:

```clojure
(defn deny-http-ingress [input]
  (when (= "http" (get-in input [:spec :rules 0 :http]))
    "Ingress must use HTTPS, not HTTP"))
```

### Warning Rules

Use specific keywords to indicate warnings vs failures:

```clojure
(defn warn-missing-monitoring [input]
  (when-not (get-in input [:metadata :annotations :prometheus.io/scrape])
    {:warning "Consider adding Prometheus monitoring annotations"}))
```

### Conditional Logic

Combine conditions for complex policies:

```clojure
(defn deny-permissive-cors [input]
  (let [cors-origin (get-in input [:metadata :annotations
                                   :nginx.ingress.kubernetes.io/cors-allow-origin])]
    (when (and cors-origin (= "*" cors-origin))
      "CORS allow-origin must not be '*' in production")))
```

## Integration with clojure.test

### Basic Test Structure

```clojure
(ns security.policy-test
  (:require [clojure.test :refer [deftest is testing]]
            [conjtest.core :as conjtest]))

(deftest test-terraform-security
  (testing "Terraform security policies"
    (let [result (conjtest/test-file
                   "infrastructure/terraform/tfplan.json"
                   {:policy "infrastructure/policies/terraform"
                    :output "json"})]
      (is (empty? (:failures result))
          "No policy violations allowed")
      (is (pos? (:successes result))
          "At least one policy check must pass"))))
```

### Testing Multiple Files

```clojure
(defn test-directory [dir policy-dir]
  "Test all configuration files in a directory"
  (let [files (->> (file-seq (io/file dir))
                   (filter #(.isFile %))
                   (filter #(or (.endsWith (.getName %) ".yaml")
                               (.endsWith (.getName %) ".yml")))
                   (map #(.getPath %)))]
    (reduce
      (fn [acc file]
        (let [result (conjtest/test-file file {:policy policy-dir})]
          (update acc :failures concat (:failures result))))
      {:failures []}
      files)))

(deftest test-all-kubernetes-manifests
  (testing "All Kubernetes manifests comply with policies"
    (let [results (test-directory
                    "infrastructure/kubernetes"
                    "infrastructure/policies/kubernetes")]
      (is (empty? (:failures results))
          (str "Policy violations: " (pr-str (:failures results)))))))
```

### Parameterized Tests

```clojure
(deftest test-multiple-environments
  (doseq [env ["dev" "staging" "production"]]
    (testing (str "Testing " env " environment")
      (let [result (conjtest/test-file
                     (str "infrastructure/terraform/" env "/tfplan.json")
                     {:policy "infrastructure/policies/terraform"
                      :namespace (str "terraform." env)})]
        (is (empty? (:failures result))
            (str env " environment has policy violations"))))))
```

## Advanced Usage

### Custom Output Handling

```clojure
(defn format-violations [result]
  "Format violations for human-readable output"
  (when (seq (:failures result))
    (str "Policy Violations Found:\n"
         (str/join "\n"
           (map-indexed
             #(str (inc %1) ". " %2)
             (:failures result))))))

(let [result (conjtest/test-file "config.yaml" {:policy "policies"})]
  (when-let [formatted (format-violations result)]
    (println formatted)))
```

### Filtering Results by Severity

```clojure
(defn classify-violation [msg]
  "Classify violation by severity based on keywords"
  (cond
    (re-find #"(?i)critical|privileged|root|0\.0\.0\.0" msg) :critical
    (re-find #"(?i)high|encryption|secret" msg) :high
    (re-find #"(?i)medium|resource|limit" msg) :medium
    :else :low))

(defn filter-critical-violations [result]
  "Extract only critical violations"
  (filter
    #(= :critical (classify-violation %))
    (:failures result)))
```

### Batch Testing

```clojure
(defn test-all-infrastructure []
  "Run all infrastructure policy tests"
  {:terraform (conjtest/test-file
                "infrastructure/terraform/tfplan.json"
                {:policy "infrastructure/policies/terraform"})
   :kubernetes (map #(conjtest/test-file
                       %
                       {:policy "infrastructure/policies/kubernetes"})
                    (find-k8s-manifests))
   :docker (map #(conjtest/test-file
                   %
                   {:policy "infrastructure/policies/docker"})
                (find-dockerfiles))})
```

## Error Handling

### Common Errors

**File Not Found:**
```clojure
(try
  (conjtest/test-file "nonexistent.yaml" {:policy "policies"})
  (catch java.io.FileNotFoundException e
    (println "Configuration file not found:" (.getMessage e))))
```

**Invalid Policy:**
```clojure
(try
  (conjtest/test-file "config.yaml" {:policy "invalid/path"})
  (catch Exception e
    (println "Policy error:" (.getMessage e))))
```

**Parse Error:**
```clojure
(try
  (conjtest/test-file "malformed.yaml" {:policy "policies"})
  (catch Exception e
    (println "Failed to parse configuration:" (.getMessage e))))
```

## Best Practices

### 1. Organize Policies by Domain

```clojure
infrastructure/policies/
├── terraform/
│   ├── policy.clj          ; Main policies
│   ├── encryption.clj      ; Encryption-specific
│   └── networking.clj      ; Network-specific
├── kubernetes/
│   ├── policy.clj
│   ├── pod-security.clj
│   └── network-policy.clj
└── docker/
    ├── policy.clj
    └── dockerfile-security.clj
```

### 2. Use Descriptive Function Names

```clojure
;; Good - describes what is being checked
(defn deny-public-s3-buckets [input] ...)
(defn require-encryption-at-rest [input] ...)

;; Bad - unclear purpose
(defn check-s3 [input] ...)
(defn validate [input] ...)
```

### 3. Provide Clear Error Messages

```clojure
;; Good - actionable message
(defn deny-missing-backup [input]
  (when-not (get-in input [:spec :backupRetentionPeriod])
    "RDS instance must have backup retention period >= 7 days (set backupRetentionPeriod)"))

;; Bad - unclear message
(defn deny-missing-backup [input]
  (when-not (get-in input [:spec :backupRetentionPeriod])
    "Invalid configuration"))
```

### 4. Test Your Policies

```clojure
(ns policy-test
  (:require [clojure.test :refer [deftest is testing]]
            [policy :as p]))

(deftest test-deny-privileged-containers
  (testing "Denies privileged containers"
    (is (some? (p/deny-privileged-containers
                 {:spec {:containers [{:securityContext {:privileged true}}]}})))
    (is (nil? (p/deny-privileged-containers
                {:spec {:containers [{:securityContext {:privileged false}}]}})))))
```

## Performance Considerations

### Caching Results

```clojure
(def test-cache (atom {}))

(defn cached-test-file [file-path options]
  "Cache test results to avoid redundant checks"
  (let [cache-key [file-path options]]
    (or (@test-cache cache-key)
        (let [result (conjtest/test-file file-path options)]
          (swap! test-cache assoc cache-key result)
          result))))
```

### Parallel Testing

```clojure
(require '[clojure.core.async :as async])

(defn test-files-parallel [files policy-dir]
  "Test multiple files in parallel"
  (let [results-chan (async/chan (count files))]
    (doseq [file files]
      (async/go
        (async/>! results-chan
          (conjtest/test-file file {:policy policy-dir}))))
    (async/<!! (async/into [] results-chan))))
```

## Debugging

### Enable Trace Output

```clojure
(conjtest/test-file
  "config.yaml"
  {:policy "policies"
   :trace true})  ; Shows detailed evaluation trace
```

### Inspect Policy Evaluation

```clojure
(defn debug-policy [input]
  "Policy with debug output"
  (let [privileged (get-in input [:spec :containers 0 :securityContext :privileged])]
    (println "DEBUG: privileged =" privileged)
    (when privileged
      "Container is privileged")))
```

## References

- Conjtest Repository: https://github.com/ilmoraunio/conjtest
- Malli Schema Library: https://github.com/metosin/malli
- Conftest Documentation: https://www.conftest.dev
- Open Policy Agent: https://www.openpolicyagent.org
