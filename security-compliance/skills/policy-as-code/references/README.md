# Policy-as-Code Reference Documentation

This directory contains comprehensive reference material for OPA/Rego policy development, fetched from official Open Policy Agent documentation sources.

## Contents

### 1. Rego Language Reference ([rego-language.md](./rego-language.md))

**Purpose:** Complete language reference for the Rego policy language

**Topics Covered:**
- Data types (scalars, arrays, objects, sets)
- Variables and references
- Control flow (negation, quantification, conditionals)
- Operators (assignment, comparison, unification, `in`)
- Comprehensions (array, object, set)
- Functions (user-defined and built-in)
- Modules and organization
- Metadata and schema annotations
- Language semantics and best practices

**When to Use:**
- Learning Rego syntax
- Understanding data types and operators
- Writing complex rules and functions
- Troubleshooting language-specific issues

**Size:** ~824 lines

---

### 2. Policy Patterns ([policy-patterns.md](./policy-patterns.md))

**Purpose:** Proven patterns for writing effective policies across different domains

**Topics Covered:**
- Foundational patterns (deny, allow, warn, defaults)
- RBAC and ABAC patterns
- Kubernetes admission control patterns
- Infrastructure policies (Terraform, Docker)
- Helper function patterns
- Data validation patterns
- Performance optimization patterns
- Integration patterns

**When to Use:**
- Starting a new policy domain
- Implementing authorization logic
- Writing Kubernetes admission controllers
- Validating infrastructure-as-code
- Optimizing policy performance
- Looking for reusable patterns

**Size:** ~842 lines

---

### 3. Testing Guide ([testing.md](./testing.md))

**Purpose:** Comprehensive guide to testing Rego policies

**Topics Covered:**
- Test structure and organization
- Running tests and filtering
- Writing effective tests (positive, negative, edge cases)
- Data-driven and parameterized testing
- Mocking with the `with` keyword
- Coverage analysis
- Advanced testing techniques
- CI/CD integration
- Testing best practices and anti-patterns

**When to Use:**
- Writing tests for policies
- Setting up CI/CD for policy validation
- Improving test coverage
- Debugging failed tests
- Learning testing best practices

**Size:** ~909 lines

---

## Quick Start

### For New Users

1. Start with **[rego-language.md](./rego-language.md)** sections:
   - "Purpose and Philosophy"
   - "Data Types"
   - "Rules and Documents"
   - "Basic Syntax" examples

2. Review common patterns in **[policy-patterns.md](./policy-patterns.md)**:
   - "Deny Rules Pattern"
   - "Allow Rules Pattern"
   - "Helper Function Patterns"

3. Set up testing with **[testing.md](./testing.md)**:
   - "Basic Test Format"
   - "Running Tests"
   - "Writing Effective Tests"

### For Specific Use Cases

**Writing Kubernetes Policies:**
- [policy-patterns.md](./policy-patterns.md) → "Kubernetes Admission Control Patterns"

**Writing Terraform Policies:**
- [policy-patterns.md](./policy-patterns.md) → "Terraform/Infrastructure Patterns"

**Implementing RBAC:**
- [policy-patterns.md](./policy-patterns.md) → "RBAC Patterns"

**Optimizing Performance:**
- [policy-patterns.md](./policy-patterns.md) → "Performance Optimization Patterns"
- [rego-language.md](./rego-language.md) → "Best Practices"

**Setting Up CI/CD:**
- [testing.md](./testing.md) → "CI/CD Integration"

---

## Documentation Sources

All content in this directory is derived from official Open Policy Agent documentation:

- **Primary Source:** [https://www.openpolicyagent.org/docs/latest/](https://www.openpolicyagent.org/docs/latest/)
- **Policy Language:** [https://www.openpolicyagent.org/docs/latest/policy-language/](https://www.openpolicyagent.org/docs/latest/policy-language/)
- **Testing:** [https://www.openpolicyagent.org/docs/latest/policy-testing/](https://www.openpolicyagent.org/docs/latest/policy-testing/)
- **Performance:** [https://www.openpolicyagent.org/docs/latest/policy-performance/](https://www.openpolicyagent.org/docs/latest/policy-performance/)

**Last Updated:** 2025-11-27

---

## Additional Resources

- [OPA Playground](https://play.openpolicyagent.org/) - Interactive Rego testing
- [OPA Policy Library](https://github.com/open-policy-agent/library) - Community examples
- [OPA Gatekeeper Library](https://github.com/open-policy-agent/gatekeeper-library) - Kubernetes policies
- [Rego Style Guide](https://github.com/StyraInc/rego-style-guide) - Code style best practices
- [Regal Linter](https://github.com/StyraInc/regal) - Linting for Rego

---

## How to Use These References

These documents are designed to complement the main [policy-as-code.md](../policy-as-code.md) skill:

1. **Main Skill File** - Workflow-focused guide for generating policies from threat models
2. **Reference Documentation** (this directory) - Deep technical reference for Rego development

The main skill provides:
- Prerequisites and installation
- Threat model integration
- Policy generation strategy
- Directory structure
- Documentation templates
- Integration guides

The reference documentation provides:
- Comprehensive language reference
- Detailed pattern library
- Testing framework guide
- Performance optimization
- Advanced techniques

Use them together for complete policy-as-code implementation!
