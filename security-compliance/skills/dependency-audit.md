---
name: dependency-audit
description: Use when adding new dependencies, before releases, or on a regular schedule. Scans deps.edn for known CVEs in direct and transitive dependencies.
requires:
  tools: [clojure]
  skills: []
skip_when:
  - No dependency changes since last audit
  - Working on code-only changes (no deps.edn modifications)
  - Project uses Leiningen (use nvd-clojure instead)
---

# Dependency Vulnerability Audit

Scan Clojure dependencies for known security vulnerabilities using [clj-watson](https://github.com/clj-holmes/clj-watson).

## Prerequisites

Add clj-watson alias to deps.edn:

```clojure
{:aliases
 {:watson
  {:deps {io.github.clj-holmes/clj-watson {:mvn/version "6.0.1"}}
   :main-opts ["-m" "clj-watson.cli" "scan"]}}}
```

**Important:** clj-watson v6+ requires a NIST NVD API key for reliable scanning.

### Get NVD API Key (Free)

1. Go to https://nvd.nist.gov/developers/request-an-api-key
2. Fill out the form (instant approval)
3. Set environment variable:
   ```bash
   export NVD_API_KEY="your-key-here"
   ```

## Workflow

### 1. Run Scan

```bash
# With NVD API key (recommended)
clojure -M:watson -p deps.edn

# Using GitHub Advisory Database (alternative, needs GitHub PAT)
clojure -M:watson -p deps.edn -s github-advisory
```

### 2. Review Output

clj-watson reports vulnerabilities with severity levels:

```
Dependency: org.apache.commons/commons-text 1.9
  CVE-2022-42889 - CRITICAL
  Description: Apache Commons Text interpolation vulnerability
  CVSS: 9.8

Suggested remediation: Upgrade to 1.10.0 or later
```

### 3. Remediate

For each vulnerability:

| Action | When |
|--------|------|
| **Upgrade** | Newer version fixes CVE |
| **Exclude** | Transitive dep, can exclude and use patched version |
| **Accept risk** | No fix available, document in ADR |
| **Replace** | Library abandoned, find alternative |

#### Upgrade Direct Dependency

```clojure
;; deps.edn - update version
{:deps {org.apache.commons/commons-text {:mvn/version "1.10.0"}}}
```

#### Exclude Vulnerable Transitive

```clojure
{:deps
 {some-lib {:mvn/version "1.0.0"
            :exclusions [org.apache.commons/commons-text]}
  ;; Add patched version directly
  org.apache.commons/commons-text {:mvn/version "1.10.0"}}}
```

### 4. Verify Fix

```bash
# Re-run scan
clojure -M:watson -p deps.edn

# Should report no vulnerabilities (or only accepted ones)
```

## CI Integration

### GitHub Actions

```yaml
name: Security Scan
on: [push, pull_request]

jobs:
  dependency-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: DeLaGuardo/setup-clojure@12.5
        with:
          cli: latest
      - name: Scan dependencies
        env:
          NVD_API_KEY: ${{ secrets.NVD_API_KEY }}
        run: clojure -M:watson -p deps.edn --fail-on-result
```

The `--fail-on-result` flag causes non-zero exit when vulnerabilities found.

### Output Formats

```bash
# JSON for programmatic processing
clojure -M:watson -p deps.edn -o json > vulnerabilities.json

# EDN
clojure -M:watson -p deps.edn -o edn

# SARIF (for GitHub Security tab)
clojure -M:watson -p deps.edn -o sarif
```

## Scanning Strategies

### NVD (Default)

- Downloads vulnerability database from NIST
- Most comprehensive coverage
- Requires API key for reliable rate limits
- Matches by package coordinates (can have false positives)

### GitHub Advisory

```bash
clojure -M:watson -p deps.edn -s github-advisory
```

- Uses GitHub's curated advisory database
- Requires GitHub PAT with `read:packages` scope
- More precise matching (fewer false positives)
- May have less coverage than NVD

## Suppressing False Positives

Create `.clj-watson/suppressions.edn`:

```clojure
;; Suppress specific CVE for specific dependency
[{:cve "CVE-2021-12345"
  :dependency "org.example/lib"
  :reason "Not exploitable in our usage - see ADR-0042"
  :expires "2025-06-01"}]
```

Then run with suppressions:

```bash
clojure -M:watson -p deps.edn --suppressions .clj-watson/suppressions.edn
```

## Success Criteria

```bash
#!/bin/bash
# verify-dependency-audit.sh

verify_dependency_audit() {
  echo "dependency_audit:start"

  # Check watson alias exists
  if ! grep -q "clj-watson" deps.edn 2>/dev/null; then
    echo "watson_alias:missing"
    echo "hint:Add :watson alias to deps.edn"
    echo "dependency_audit:exit_code:1"
    return 1
  fi
  echo "watson_alias:present"

  # Check for NVD API key
  if [ -z "$NVD_API_KEY" ]; then
    echo "nvd_api_key:missing"
    echo "hint:Set NVD_API_KEY environment variable"
    # Not a hard failure - can use GitHub Advisory instead
  else
    echo "nvd_api_key:present"
  fi

  # Run scan
  local scan_output
  scan_output=$(clojure -M:watson -p deps.edn -o edn 2>&1)
  local scan_exit=$?

  if [ $scan_exit -ne 0 ]; then
    echo "scan_run:error"
    echo "dependency_audit:exit_code:1"
    return 1
  fi

  # Count vulnerabilities by severity
  local critical high medium low
  critical=$(echo "$scan_output" | grep -c ":severity :critical" || echo "0")
  high=$(echo "$scan_output" | grep -c ":severity :high" || echo "0")
  medium=$(echo "$scan_output" | grep -c ":severity :medium" || echo "0")
  low=$(echo "$scan_output" | grep -c ":severity :low" || echo "0")

  echo "vulnerabilities_critical:$critical"
  echo "vulnerabilities_high:$high"
  echo "vulnerabilities_medium:$medium"
  echo "vulnerabilities_low:$low"

  # Fail on critical or high
  if [ "$critical" -gt 0 ] || [ "$high" -gt 0 ]; then
    echo "scan_result:vulnerabilities_found"
    echo "dependency_audit:exit_code:1"
    return 1
  fi

  echo "scan_result:pass"
  echo "dependency_audit:exit_code:0"
  return 0
}

verify_dependency_audit
```

**Success = all checks pass:**
- [ ] `watson_alias:present` - clj-watson configured in deps.edn
- [ ] `scan_run:success` - Scan completes without errors
- [ ] `vulnerabilities_critical:0` - No critical CVEs
- [ ] `vulnerabilities_high:0` - No high-severity CVEs

## Reference

- [clj-watson GitHub](https://github.com/clj-holmes/clj-watson)
- [clj-watson GitHub Action](https://github.com/marketplace/actions/clj-watson-clojure)
- [NVD API Key Registration](https://nvd.nist.gov/developers/request-an-api-key)
- [nvd-clojure](https://github.com/rm-hull/nvd-clojure) - Alternative for Leiningen projects

## Related Skills

- `threagile-analysis` - Threat modeling (architectural security)
- `policy-as-code` - Enforce security policies
- `adr-management` - Document accepted risks
