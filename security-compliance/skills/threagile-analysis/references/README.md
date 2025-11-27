# Threagile Analysis Reference Documentation

This directory contains comprehensive reference documentation for the Threagile threat modeling tool.

## Contents

### [Model Schema Reference](model-schema.md)
Complete YAML schema documentation for creating Threagile threat models.

**Covers:**
- Model file structure and metadata
- Data assets: definitions, CIA ratings, quantities
- Technical assets: types, technologies, machine types, encryption
- Trust boundaries: network zones and security domains
- Shared runtimes: execution environment grouping
- Communication links: protocols, authentication, data flows
- Individual risk categories: custom risk definitions
- Risk tracking: status management and mitigation tracking
- All field types and enumeration values
- Best practices and examples

**Use when:** Creating or modifying threat models, understanding model structure, validating model completeness.

---

### [Risk Rules Reference](risk-rules.md)
Built-in risk detection rules organized by security category.

**Covers:**
- 40+ built-in risk rules
- STRIDE threat model mapping
- Authentication & Identity Management risks
- Injection attack detection (SQL, LDAP, XSS, XXE, etc.)
- Web application security (CSRF, SSRF, etc.)
- Infrastructure & Deployment security
- Data protection (encryption, validation)
- Access control & Network security
- Secrets & Vault management
- Cloud security
- Risk severity calculation methodology
- Detection logic for each rule
- Mitigation recommendations
- CWE and ASVS mappings

**Use when:** Understanding identified risks, planning mitigations, customizing risk detection, performing security reviews.

---

### [Outputs Reference](outputs.md)
All output formats and their uses.

**Covers:**
- PDF reports for management and stakeholders
- JSON files for tool integration and automation
- Excel files for risk tracking and collaboration
- Diagrams (data flow, data assets) for visualization
- Graphviz DOT files for custom rendering
- Configuration options for each output
- CI/CD integration examples
- Dashboard integration patterns
- JIRA ticket automation
- Output customization

**Use when:** Integrating Threagile with other tools, automating workflows, creating dashboards, distributing reports.

---

## Quick Reference

### Essential Model Fields

**Data Asset:**
```yaml
data-asset-id:
  id: "unique-id"
  description: "What this data represents"
  quantity: "many"  # very-few|few|many|very-many
  confidentiality: "confidential"  # public|internal|restricted|confidential|strictly-confidential
  integrity: "important"  # archive|operational|important|critical|mission-critical
  availability: "important"
  justification_cia_rating: "Why these ratings"
```

**Technical Asset:**
```yaml
asset-id:
  id: "unique-id"
  type: "process"  # external-entity|process|datastore
  technology: "web-service-rest"
  machine: "container"  # physical|virtual|container|serverless
  encryption: "data-with-symmetric-shared-key"
  custom_developed_parts: true
  data_assets_processed: ["data-asset-id"]
  communication_links:
    link-name:
      target: "target-asset-id"
      protocol: "https"
      authentication: "token"
      data_assets_sent: ["data-asset-id"]
```

### Top Risk Categories

1. **SQL/NoSQL Injection** - Unsanitized database queries
2. **Unencrypted Communication** - Sensitive data over plaintext
3. **Missing Authentication** - Unauthenticated access to sensitive assets
4. **Cross-Site Scripting (XSS)** - Unescaped user input in web pages
5. **Untrusted Deserialization** - Deserializing untrusted data

### Key Output Files

- `report.pdf` - Management report
- `risks.json` - Machine-readable risks
- `risks.xlsx` - Tracking spreadsheet
- `data-flow-diagram.png` - Architecture visualization
- `stats.json` - Metrics and statistics

## External Resources

- [Threagile GitHub](https://github.com/Threagile/threagile)
- [Threagile Website](https://threagile.io)
- [STRIDE Threat Model](https://en.wikipedia.org/wiki/STRIDE_(security))
- [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/)
- [CWE Database](https://cwe.mitre.org/)

## Updates

This reference documentation is based on Threagile 1.0.0 and was compiled from official documentation and source code analysis. For the latest information, consult the [official Threagile documentation](https://threagile.io).

Last updated: 2024-01-15
