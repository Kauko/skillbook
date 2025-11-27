# Conjtest Integration Guide

## Overview

This guide covers integrating Conjtest policy validation into CI/CD pipelines, build tools, Git hooks, and deployment workflows.

## Build Tool Integration

### Clojure CLI (deps.edn)

#### Project Configuration

Add Conjtest to your `deps.edn`:

```clojure
{:deps {org.clojure/clojure {:mvn/version "1.11.1"}}

 :aliases
 {:test
  {:extra-paths ["test"]
   :extra-deps {ilmoraunio/conjtest {:mvn/version "0.1.0"}
                org.clojure/data.json {:mvn/version "2.4.0"}}}

  :policy-check
  {:main-opts ["-m" "security.compliance-report"]
   :extra-deps {ilmoraunio/conjtest {:mvn/version "0.1.0"}}}

  :dev
  {:extra-paths ["dev"]
   :extra-deps {ilmoraunio/conjtest {:mvn/version "0.1.0"}}}}}
```

#### Running Tests

```bash
# Run all tests including policy tests
clj -M:test -m cognitect.test-runner

# Run only policy compliance tests
clj -M:policy-check

# Run with kaocha
clj -M:test -m kaocha.runner
```

### Leiningen

#### project.clj Configuration

```clojure
(defproject myapp "0.1.0"
  :dependencies [[org.clojure/clojure "1.11.1"]]

  :profiles
  {:test
   {:dependencies [[ilmoraunio/conjtest "0.1.0"]
                   [org.clojure/data.json "2.4.0"]]
    :test-paths ["test"]}

   :policy
   {:dependencies [[ilmoraunio/conjtest "0.1.0"]]
    :main security.compliance-report}}

  :aliases
  {"policy-check" ["with-profile" "policy" "run" "-m" "security.compliance-report"]})
```

#### Running Tests

```bash
# Run all tests
lein test

# Run policy compliance check
lein policy-check

# Run with auto-test
lein test-refresh
```

### Boot

#### build.boot Configuration

```clojure
(set-env!
  :dependencies '[[ilmoraunio/conjtest "0.1.0" :scope "test"]
                  [org.clojure/data.json "2.4.0" :scope "test"]]
  :source-paths #{"src"}
  :test-paths #{"test"})

(deftask policy-check []
  "Run policy compliance checks"
  (comp
    (testing)
    (clojure "-m" "security.compliance-report")))
```

#### Running Tests

```bash
# Run policy checks
boot policy-check

# Run in watch mode
boot watch policy-check
```

## CI/CD Integration

### GitHub Actions

#### Basic Policy Check Workflow

Create `.github/workflows/policy-check.yml`:

```yaml
name: Policy Compliance

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]
  schedule:
    # Run daily at 2 AM UTC
    - cron: '0 2 * * *'

jobs:
  policy-validation:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Clojure
        uses: DeLaGuardo/setup-clojure@12.5
        with:
          cli: latest

      - name: Install Conftest
        run: |
          wget -q https://github.com/open-policy-agent/conftest/releases/latest/download/conftest_Linux_x86_64.tar.gz
          tar xzf conftest_Linux_x86_64.tar.gz
          sudo mv conftest /usr/local/bin/
          conftest --version

      - name: Cache Clojure dependencies
        uses: actions/cache@v3
        with:
          path: |
            ~/.m2/repository
            ~/.gitlibs
          key: ${{ runner.os }}-clojure-${{ hashFiles('**/deps.edn') }}
          restore-keys: |
            ${{ runner.os }}-clojure-

      - name: Run policy compliance tests
        run: clj -M:test -m security.compliance-report

      - name: Upload compliance report
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: policy-compliance-report
          path: vault/security/policy-compliance.md
          retention-days: 30

      - name: Fail on critical violations
        run: |
          if grep -q "Critical:" vault/security/policy-compliance.md; then
            echo "Critical policy violations found!"
            exit 1
          fi
```

#### Advanced Workflow with Terraform

Create `.github/workflows/terraform-policy.yml`:

```yaml
name: Terraform Policy Validation

on:
  pull_request:
    paths:
      - 'infrastructure/terraform/**'
      - 'infrastructure/policies/terraform/**'

jobs:
  terraform-policy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.6.0

      - name: Setup Clojure
        uses: DeLaGuardo/setup-clojure@12.5
        with:
          cli: latest

      - name: Install Conftest
        run: |
          wget -q https://github.com/open-policy-agent/conftest/releases/latest/download/conftest_Linux_x86_64.tar.gz
          tar xzf conftest_Linux_x86_64.tar.gz
          sudo mv conftest /usr/local/bin/

      - name: Terraform Init
        run: |
          cd infrastructure/terraform
          terraform init -backend=false

      - name: Generate Terraform Plan
        run: |
          cd infrastructure/terraform
          terraform plan -out=tfplan.binary
          terraform show -json tfplan.binary > tfplan.json

      - name: Validate Terraform Plan
        run: clj -M:test -m security.terraform-policy-test

      - name: Comment PR with violations
        if: failure() && github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const violations = fs.readFileSync('vault/security/policy-compliance.md', 'utf8');
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## Terraform Policy Violations\n\n${violations}`
            });
```

### GitLab CI

Create `.gitlab-ci.yml`:

```yaml
variables:
  CONFTEST_VERSION: "latest"

stages:
  - setup
  - validate
  - report

cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - .m2/repository
    - .gitlibs

setup:conftest:
  stage: setup
  image: alpine:latest
  script:
    - apk add --no-cache wget tar
    - wget https://github.com/open-policy-agent/conftest/releases/latest/download/conftest_Linux_x86_64.tar.gz
    - tar xzf conftest_Linux_x86_64.tar.gz
    - mv conftest /usr/local/bin/
  artifacts:
    paths:
      - /usr/local/bin/conftest
    expire_in: 1 day

policy:validation:
  stage: validate
  image: clojure:temurin-21-tools-deps
  dependencies:
    - setup:conftest
  script:
    - clj -M:test -m security.compliance-report
  artifacts:
    when: always
    paths:
      - vault/security/policy-compliance.md
    reports:
      junit: test-results.xml
  only:
    - merge_requests
    - main
    - develop

policy:report:
  stage: report
  image: alpine:latest
  dependencies:
    - policy:validation
  script:
    - cat vault/security/policy-compliance.md
  only:
    - merge_requests
    - main
  when: always
```

### CircleCI

Create `.circleci/config.yml`:

```yaml
version: 2.1

orbs:
  clojure: circleci/clojure@1.0

jobs:
  policy-check:
    docker:
      - image: cimg/clojure:1.11
    steps:
      - checkout

      - restore_cache:
          keys:
            - v1-deps-{{ checksum "deps.edn" }}
            - v1-deps-

      - run:
          name: Install Conftest
          command: |
            wget https://github.com/open-policy-agent/conftest/releases/latest/download/conftest_Linux_x86_64.tar.gz
            tar xzf conftest_Linux_x86_64.tar.gz
            sudo mv conftest /usr/local/bin/

      - run:
          name: Download dependencies
          command: clj -P -M:test

      - save_cache:
          key: v1-deps-{{ checksum "deps.edn" }}
          paths:
            - ~/.m2
            - ~/.gitlibs

      - run:
          name: Run policy compliance tests
          command: clj -M:test -m security.compliance-report

      - store_artifacts:
          path: vault/security/policy-compliance.md
          destination: policy-compliance-report

      - store_test_results:
          path: test-results

workflows:
  version: 2
  policy-validation:
    jobs:
      - policy-check:
          filters:
            branches:
              only:
                - main
                - develop
                - /^feature\/.*/
```

### Jenkins

Create `Jenkinsfile`:

```groovy
pipeline {
    agent any

    environment {
        CONFTEST_VERSION = 'latest'
    }

    stages {
        stage('Setup') {
            steps {
                sh '''
                    wget https://github.com/open-policy-agent/conftest/releases/latest/download/conftest_Linux_x86_64.tar.gz
                    tar xzf conftest_Linux_x86_64.tar.gz
                    sudo mv conftest /usr/local/bin/
                '''
            }
        }

        stage('Dependency Cache') {
            steps {
                cache(maxCacheSize: 1000, caches: [
                    arbitraryFileCache(
                        path: '.m2/repository',
                        cacheValidityDecidingFile: 'deps.edn'
                    )
                ]) {
                    sh 'clj -P -M:test'
                }
            }
        }

        stage('Policy Validation') {
            steps {
                sh 'clj -M:test -m security.compliance-report'
            }
        }

        stage('Publish Report') {
            steps {
                publishHTML([
                    reportDir: 'vault/security',
                    reportFiles: 'policy-compliance.md',
                    reportName: 'Policy Compliance Report'
                ])
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'vault/security/policy-compliance.md', fingerprint: true
        }
        failure {
            emailext (
                subject: "Policy Violations Detected: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
                body: '''${FILE,path="vault/security/policy-compliance.md"}''',
                to: 'security-team@example.com'
            )
        }
    }
}
```

## Git Hooks Integration

### Pre-commit Hook

Create `.git/hooks/pre-commit`:

```bash
#!/bin/bash
set -e

echo "Running policy compliance checks..."

# Check if Conjtest is available
if ! command -v conftest &> /dev/null; then
    echo "Error: Conftest not installed"
    echo "Install with: brew install conftest"
    exit 1
fi

# Get list of changed configuration files
CHANGED_FILES=$(git diff --cached --name-only --diff-filter=ACM | \
    grep -E '\.(yaml|yml|json|tf|toml)$' || true)

if [ -z "$CHANGED_FILES" ]; then
    echo "No configuration files changed, skipping policy checks"
    exit 0
fi

# Run policy checks on changed files
VIOLATIONS=0
for file in $CHANGED_FILES; do
    echo "Checking $file..."

    # Determine policy directory based on file path
    if [[ $file == infrastructure/terraform/* ]]; then
        POLICY_DIR="infrastructure/policies/terraform"
    elif [[ $file == infrastructure/kubernetes/* ]]; then
        POLICY_DIR="infrastructure/policies/kubernetes"
    elif [[ $file == *Dockerfile* ]]; then
        POLICY_DIR="infrastructure/policies/docker"
    else
        continue
    fi

    # Run Conjtest validation
    if ! clj -M:test -e "(require '[conjtest.core :as c]) \
        (let [r (c/test-file \"$file\" {:policy \"$POLICY_DIR\"})] \
        (when (seq (:failures r)) \
        (println \"Violations in $file:\") \
        (doseq [f (:failures r)] (println \"  -\" f)) \
        (System/exit 1)))"; then
        VIOLATIONS=$((VIOLATIONS + 1))
    fi
done

if [ $VIOLATIONS -gt 0 ]; then
    echo ""
    echo "Policy violations detected in $VIOLATIONS file(s)"
    echo "Fix violations or use 'git commit --no-verify' to bypass"
    exit 1
fi

echo "All policy checks passed!"
exit 0
```

Make it executable:
```bash
chmod +x .git/hooks/pre-commit
```

### Pre-push Hook

Create `.git/hooks/pre-push`:

```bash
#!/bin/bash
set -e

echo "Running comprehensive policy compliance check before push..."

# Run full compliance report
if ! clj -M:test -m security.compliance-report; then
    echo ""
    echo "Policy compliance check failed!"
    echo "Review violations in vault/security/policy-compliance.md"
    echo "Use 'git push --no-verify' to bypass (not recommended)"
    exit 1
fi

echo "Policy compliance check passed!"
exit 0
```

Make it executable:
```bash
chmod +x .git/hooks/pre-push
```

### Commit Message Hook

Create `.git/hooks/commit-msg`:

```bash
#!/bin/bash

COMMIT_MSG_FILE=$1
COMMIT_MSG=$(cat "$COMMIT_MSG_FILE")

# Check if commit includes policy bypass flag
if echo "$COMMIT_MSG" | grep -q "\[skip-policy\]"; then
    echo "Warning: Policy checks skipped by commit message"
    exit 0
fi

# Run quick policy validation on staged files
if ! clj -M:test -e "(println \"Policy check passed\")"; then
    echo "Policy validation failed"
    echo "Add [skip-policy] to commit message to bypass"
    exit 1
fi

exit 0
```

Make it executable:
```bash
chmod +x .git/hooks/commit-msg
```

## Container Integration

### Docker

#### Dockerfile for Policy Validation

```dockerfile
FROM clojure:temurin-21-tools-deps-alpine

# Install Conftest
RUN wget https://github.com/open-policy-agent/conftest/releases/latest/download/conftest_Linux_x86_64.tar.gz && \
    tar xzf conftest_Linux_x86_64.tar.gz && \
    mv conftest /usr/local/bin/ && \
    rm conftest_Linux_x86_64.tar.gz

WORKDIR /app

# Copy project files
COPY deps.edn .
COPY src/ src/
COPY test/ test/
COPY infrastructure/ infrastructure/

# Download dependencies
RUN clj -P -M:test

# Run policy checks
CMD ["clj", "-M:test", "-m", "security.compliance-report"]
```

#### Docker Compose

```yaml
version: '3.8'

services:
  policy-validator:
    build:
      context: .
      dockerfile: Dockerfile.policy
    volumes:
      - ./infrastructure:/app/infrastructure
      - ./vault:/app/vault
    environment:
      - POLICY_DIR=/app/infrastructure/policies
    command: clj -M:test -m security.compliance-report

  policy-watcher:
    build:
      context: .
      dockerfile: Dockerfile.policy
    volumes:
      - ./infrastructure:/app/infrastructure
      - ./vault:/app/vault
    command: clj -M:test -m kaocha.runner --watch
```

Run with:
```bash
docker-compose run policy-validator
docker-compose up policy-watcher
```

## Kubernetes Integration

### Admission Controller

Deploy Conftest as a Kubernetes admission controller:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: conjtest-policies
  namespace: policy-system
data:
  policy.clj: |
    (ns policy)

    (defn deny-privileged-containers [input]
      (when (get-in input [:spec :containers 0 :securityContext :privileged])
        "Privileged containers not allowed"))

    (defn deny-host-network [input]
      (when (get-in input [:spec :hostNetwork])
        "Host network access not allowed"))
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: policy-validator
  namespace: policy-system
spec:
  replicas: 2
  selector:
    matchLabels:
      app: policy-validator
  template:
    metadata:
      labels:
        app: policy-validator
    spec:
      containers:
      - name: validator
        image: my-registry/policy-validator:latest
        ports:
        - containerPort: 8080
        volumeMounts:
        - name: policies
          mountPath: /policies
      volumes:
      - name: policies
        configMap:
          name: conjtest-policies
---
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: policy-validator
webhooks:
- name: validate.policy.example.com
  clientConfig:
    service:
      name: policy-validator
      namespace: policy-system
      path: "/validate"
  rules:
  - operations: ["CREATE", "UPDATE"]
    apiGroups: [""]
    apiVersions: ["v1"]
    resources: ["pods"]
  admissionReviewVersions: ["v1"]
  sideEffects: None
```

## IDE Integration

### VS Code

Create `.vscode/tasks.json`:

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Run Policy Checks",
      "type": "shell",
      "command": "clj -M:test -m security.compliance-report",
      "group": {
        "kind": "test",
        "isDefault": true
      },
      "presentation": {
        "reveal": "always",
        "panel": "new"
      },
      "problemMatcher": []
    },
    {
      "label": "Watch Policy Checks",
      "type": "shell",
      "command": "clj -M:test -m kaocha.runner --watch",
      "isBackground": true,
      "problemMatcher": {
        "pattern": {
          "regexp": "."
        },
        "background": {
          "activeOnStart": true,
          "beginsPattern": "^.*Running tests.*$",
          "endsPattern": "^.*tests complete.*$"
        }
      }
    }
  ]
}
```

### Emacs/CIDER

Add to `.dir-locals.el`:

```elisp
((clojure-mode . ((cider-default-cljs-repl . node)
                  (cider-test-default-include-selectors . "[:policy]")
                  (cider-test-run-on-save . t))))
```

### IntelliJ/Cursive

Add run configuration in `.idea/runConfigurations/Policy_Check.xml`:

```xml
<component name="ProjectRunConfigurationManager">
  <configuration default="false" name="Policy Check" type="ClojureREPL" factoryName="REPL">
    <module name="myproject" />
    <option name="execution" value="LEININGEN" />
    <option name="REPL_TYPE" value="NREPL" />
    <option name="REPL_OPTIONS" value="-M:test -m security.compliance-report" />
    <method v="2" />
  </configuration>
</component>
```

## Monitoring and Alerting

### Prometheus Integration

Export metrics from policy checks:

```clojure
(ns security.metrics
  (:require [iapetos.core :as prometheus]
            [iapetos.collector.ring :as ring-metrics]))

(def registry
  (-> (prometheus/collector-registry)
      (ring-metrics/initialize)))

(def policy-violations-counter
  (prometheus/counter
    registry
    :policy/violations
    "Total policy violations detected"
    {:labels [:domain :severity]}))

(defn record-violation [domain severity]
  (prometheus/inc policy-violations-counter
                  {:domain domain :severity severity}))

(defn export-metrics []
  (prometheus/serialize registry))
```

### Grafana Dashboard

Example Grafana dashboard JSON:

```json
{
  "dashboard": {
    "title": "Policy Compliance",
    "panels": [
      {
        "title": "Violations by Severity",
        "targets": [
          {
            "expr": "sum(policy_violations_total) by (severity)"
          }
        ],
        "type": "graph"
      },
      {
        "title": "Compliance Rate",
        "targets": [
          {
            "expr": "100 * (1 - (sum(policy_violations_total) / sum(policy_checks_total)))"
          }
        ],
        "type": "singlestat"
      }
    ]
  }
}
```

## Best Practices

### 1. Fast Feedback Loops

- Run lightweight checks in pre-commit hooks
- Run comprehensive checks in CI/CD
- Use watch mode during development

### 2. Fail Fast

- Validate on commit, not just on deploy
- Block merges with policy violations
- Provide clear remediation guidance

### 3. Progressive Enforcement

- Start with warnings for new policies
- Gradually increase strictness
- Exempt legacy code with documented exceptions

### 4. Clear Communication

- Include policy documentation in violation messages
- Link to remediation guides
- Provide severity levels for prioritization

### 5. Performance Optimization

- Cache dependencies in CI/CD
- Run only relevant policies for changed files
- Use parallel execution for multiple files

## Troubleshooting

### Common Issues

**Conftest Not Found:**
```bash
# Verify installation
which conftest
conftest --version

# Add to PATH if needed
export PATH=$PATH:/usr/local/bin
```

**Slow CI/CD Builds:**
```bash
# Enable dependency caching
# GitHub Actions example
- uses: actions/cache@v3
  with:
    path: ~/.m2/repository
    key: ${{ runner.os }}-clojure-${{ hashFiles('**/deps.edn') }}
```

**Policy Not Applied:**
```bash
# Verify policy directory
ls -la infrastructure/policies/

# Test policy directly
conftest test config.yaml -p infrastructure/policies/
```

**Permission Issues:**
```bash
# Make hooks executable
chmod +x .git/hooks/pre-commit
chmod +x .git/hooks/pre-push
```

## References

- Conjtest: https://github.com/ilmoraunio/conjtest
- Conftest: https://www.conftest.dev
- Open Policy Agent: https://www.openpolicyagent.org
- Clojure CLI: https://clojure.org/guides/deps_and_cli
