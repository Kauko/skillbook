# Overarch Modeling References

This directory contains comprehensive technical reference documentation for Overarch modeling, extracted from official Overarch sources including the GitHub repository and usage documentation.

## Documentation Structure

### [model-elements.md](model-elements.md)
**Complete element type catalog** (456 lines)

Covers all model element types available in Overarch:
- **Architecture Elements**: person, system, container, component, node
- **Use Case Elements**: actor, use-case
- **Code Model Elements**: package, namespace, class, interface, protocol, method, function
- **State Machine Elements**: state-machine, state, transition
- **Concept Model Elements**: concept
- **Organization Elements**: organization, org-unit
- **Process Elements**: capability, process, artifact, requirement

Includes:
- Common properties for all elements
- Element-specific attributes
- Subtypes (database, queue)
- Comprehensive examples
- Naming conventions
- Best practices
- Model composition patterns

### [relationships.md](relationships.md)
**Complete relationship type catalog** (754 lines)

Covers all relationship types for connecting model elements:
- **Architecture Relations**: request, response, send, publish, subscribe, dataflow
- **Hierarchical Relations**: contained-in
- **Traceability Relations**: implements, ref
- **Use Case Relations**: uses, include, extends, generalizes
- **Code Relations**: association, aggregation, composition, inheritance, implementation, dependency
- **Deployment Relations**: link, deployed-to
- **Concept Relations**: is-a, has
- **Organization Relations**: responsible-for, collaborates-with, role-in
- **Process Relations**: required-for, input-of, output-of

Includes:
- Relationship properties and semantics
- Common communication patterns (sync, async, event-driven)
- Architecture patterns (layered, microservices)
- Layout control with `:direction`
- Naming best practices
- Troubleshooting guide
- Advanced features (conditional relationships, security annotations)

### [views.md](views.md)
**Complete view type and configuration reference** (947 lines)

Covers all view types and their configuration:
- **C4 Architecture Views**: context-view, container-view, component-view, deployment-view, system-landscape-view, deployment-architecture-view, dynamic-view
- **UML Design Views**: code-view, state-machine-view, use-case-view
- **Documentation Views**: concept-view, glossary, organization-structure-view, deployment-structure-view

Includes:
- View fundamentals and common properties
- Element selection criteria (by type, namespace, ID, technology, tags, relationships, hierarchy)
- Include patterns (:relations, :related)
- Content lists (:ct) for precise control
- Layout control (direction, spacing, engines)
- Styling and themes (sprite libraries, custom styles, PlantUML skinparams)
- View organization best practices
- Advanced techniques (multi-level views, filtered views, environment-specific views)
- Troubleshooting guide
- Use case examples

### [cli-usage.md](cli-usage.md)
**Complete CLI command reference** (651 lines)

Covers all command-line interface usage:
- **Core Options**: model directory, rendering, export
- **Rendering**: PlantUML, GraphViz, Markdown formats
- **Export**: JSON, Structurizr formats
- **Query**: Element selection and querying
- **Template Generation**: Configuration and template processing
- **Watch Mode**: Auto-regeneration on file changes
- **Debug**: Logging and troubleshooting

Includes:
- All CLI options with examples
- Common command patterns (initial rendering, development workflow, production export)
- PlantUML integration
- GraphViz integration
- Workflow examples (first-time setup, iterative development, CI/CD)
- Project structure recommendations
- Makefile example
- Troubleshooting guide
- Performance tips
- Advanced usage (programmatic access, custom validation, tool integration)

## How to Use These References

### For Workflow Guidance
Refer to the main skill file (`../overarch-modeling.md`) for:
- Step-by-step workflow
- Quick start templates
- Common operations
- Integration with other skills

### For Technical Details
Refer to these reference files when you need:
- Specific element properties
- Relationship semantics
- View configuration options
- CLI command syntax
- Advanced features

### For Learning
Read in this order:
1. Start with main skill file for overview
2. Read `model-elements.md` to understand what you can model
3. Read `relationships.md` to understand how to connect elements
4. Read `views.md` to understand how to visualize your model
5. Read `cli-usage.md` to understand how to generate outputs

## Source Information

This documentation is synthesized from:
- **Overarch GitHub Repository**: https://github.com/soulspace-org/overarch
- **Usage Documentation**: Official Overarch usage guide
- **Design Documentation**: Overarch design philosophy and architecture
- **README**: Project overview and quick start

All documentation reflects Overarch capabilities as of the latest available documentation (2024).

## Maintenance

These references should be updated when:
- New Overarch versions add element types
- New relationship types are introduced
- New view types become available
- CLI options change
- Best practices evolve

To update, fetch latest documentation from:
- https://github.com/soulspace-org/overarch/blob/main/README.md
- https://github.com/soulspace-org/overarch/blob/main/doc/usage.md
- https://github.com/soulspace-org/overarch/blob/main/doc/design.md
