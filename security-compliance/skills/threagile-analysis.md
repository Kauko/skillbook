---
name: threagile-analysis
description: Use when architecture changes or BEFORE deploying new components. Run threat analysis to identify STRIDE risks. Required before any production deployment.
requires:
  tools: [threagile]
  skills: []
skip_when:
  - No architectural changes since last analysis
  - Working on internal tooling with no external exposure
  - Pure refactoring with no new data flows or trust boundaries
---

# Threagile Threat Modeling

## Prerequisites

```bash
command -v threagile >/dev/null || { echo "Install: brew install threagile"; exit 1; }
```

## Workflow

### 1. Check for Model

```bash
ls vault/security/threagile.yaml
```

### 2. Create/Update Model

`vault/security/threagile.yaml`:
```yaml
threagile_version: 1.0.0

title: "System Threat Model"
author:
  name: "Team"

business_criticality: important  # archive, operational, important, critical, mission-critical

technical_assets:
  api-server:
    title: "API Server"
    type: process
    usage: business
    size: application
    technology: web-service-rest
    machine: container
    encryption: none
    data_assets_processed: [user-credentials, customer-data]
    data_assets_stored: []
    communication_links:
      database-connection:
        target: database
        protocol: sql
        authentication: credentials
        encryption: transparent

  database:
    title: "Database"
    type: datastore
    usage: business
    size: component
    technology: database
    machine: container
    encryption: transparent
    data_assets_stored: [customer-data]

data_assets:
  customer-data:
    title: "Customer Data"
    usage: business
    quantity: many
    confidentiality: confidential
    integrity: critical
    availability: operational

trust_boundaries:
  dmz:
    title: "DMZ"
    type: network-cloud-security-group
    technical_assets_inside: [api-server]
  internal:
    title: "Internal Network"
    type: network-on-prem
    technical_assets_inside: [database]
```

### 3. Run Analysis

```bash
threagile analyze -model vault/security/threagile.yaml -output vault/security/
```

### 4. Review Outputs

Generated files:
- `risks.json` - Machine-readable findings
- `report.pdf` - Human-readable report
- `data-flow-diagram.png` - Visual model

### 5. Address Findings

For each risk:
1. Accept risk (document rationale)
2. Mitigate (implement control)
3. Transfer (insurance/contract)

## Risk Categories (STRIDE)

| Category | Threats |
|----------|---------|
| Spoofing | Authentication bypass |
| Tampering | Data modification |
| Repudiation | Action denial |
| Information Disclosure | Data leaks |
| Denial of Service | Availability attacks |
| Elevation of Privilege | Unauthorized access |

## Reference Documentation

- `references/model-schema.md` - YAML schema
- `references/risk-rules.md` - Built-in rules
- `references/outputs.md` - Output formats

## Success Criteria

```bash
#!/bin/bash
# verify-threagile.sh - Machine-readable threat analysis verification

verify_threagile() {
  local model_file="${1:-vault/security/threagile.yaml}"
  local output_dir="${2:-vault/security}"
  local exit_code=0

  echo "threagile_verification:start"

  # Check model exists
  if [ -f "$model_file" ]; then
    echo "model_exists:true"
  else
    echo "model_exists:false"
    echo "threagile_verification:exit_code:1"
    return 1
  fi

  # Check threagile installed
  if ! command -v threagile &>/dev/null; then
    echo "threagile:not_installed"
    echo "threagile_verification:exit_code:1"
    return 1
  fi

  # Run analysis
  if threagile analyze -model "$model_file" -output "$output_dir" >/dev/null 2>&1; then
    echo "analysis:pass"
  else
    echo "analysis:fail"
    exit_code=1
  fi

  # Check outputs generated
  if [ -f "$output_dir/risks.json" ]; then
    echo "risks_json:exists"
    local risk_count
    risk_count=$(jq 'length' "$output_dir/risks.json" 2>/dev/null || echo "0")
    echo "risks_identified:$risk_count"

    # Check for unaddressed risks (no mitigation status)
    local unaddressed
    unaddressed=$(jq '[.[] | select(.status == null or .status == "")] | length' "$output_dir/risks.json" 2>/dev/null || echo "0")
    echo "risks_unaddressed:$unaddressed"
    [ "$unaddressed" -gt 0 ] && exit_code=1
  else
    echo "risks_json:missing"
    exit_code=1
  fi

  echo "threagile_verification:exit_code:$exit_code"
  return $exit_code
}

verify_threagile "$@"
```

**Success = all checks pass:**
- [ ] `model_exists:true` - Threat model YAML exists
- [ ] `analysis:pass` - Threagile completes without errors
- [ ] `risks_json:exists` - Risk report generated
- [ ] `risks_unaddressed:0` - All risks have accept/mitigate/transfer decision

## Related Skills

- `policy-as-code` - Policy enforcement
- `iso25010-quality` - Security requirements
