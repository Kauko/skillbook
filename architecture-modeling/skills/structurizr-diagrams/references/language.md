# Structurizr DSL Language Reference

Complete reference for the Structurizr DSL (Domain-Specific Language) for defining software architecture models using the C4 model framework.

## Overview

The Structurizr DSL is a **model-based** approach to diagrams as code, making it possible to generate multiple diagrams at different levels of abstraction from a single DSL file. It runs on the Java Virtual Machine, allowing execution of Java, Groovy, Kotlin, and JRuby code during workspace generation.

## Core Structure

### Workspace

The top-level language construct and wrapper for the model and views:

```dsl
workspace [name] [description] {
    model { ... }
    views { ... }
}
```

**Properties:**
- `name` (optional): Workspace name
- `description` (optional): Workspace description
- Can extend another workspace using a local DSL/JSON file or remote HTTPS URL

**Permitted children:**
- `properties` - Custom metadata
- `!identifiers` - Identifier configuration
- `!docs` - Documentation
- `!adrs` - Architecture decision records
- `model` - Element definitions
- `views` - Diagram definitions
- `configuration` - Workspace configuration

## Model Elements

### Person

Represents users, actors, roles, or personas:

```dsl
person <name> [description] [tags] {
    [properties]
}
```

**Example:**
```dsl
user = person "User" "A system user" {
    url "https://example.com/user"
    properties {
        role "End User"
    }
}
```

**Default tags:** `Element`, `Person`

### Software System

Top-level architectural component representing a software system:

```dsl
softwareSystem <name> [description] [tags] {
    [containers]
}
```

**Example:**
```dsl
mainSystem = softwareSystem "Main System" "Primary application" {
    webapp = container "Web Application" "User interface" "React"
    api = container "API" "Business logic" "Node.js"
    database = container "Database" "Data store" "PostgreSQL"
}
```

**Default tags:** `Element`, `Software System`

### Container

Deployable unit within a software system (application, service, database):

```dsl
container <name> [description] [technology] [tags] {
    [components]
}
```

**Example:**
```dsl
backend = container "Backend API" "REST API" "Java, Spring Boot" {
    controller = component "Controller" "Request handler" "Spring MVC"
    service = component "Service" "Business logic" "Spring"
    repository = component "Repository" "Data access" "Spring Data"
}
```

**Default tags:** `Element`, `Container`

**Technology:** Comma-separated list of technologies used

### Component

Building block within a container:

```dsl
component <name> [description] [technology] [tags] {
    [properties]
}
```

**Default tags:** `Element`, `Component`

**Technology:** Comma-separated list of technologies used

## Deployment Elements

### Deployment Environment

Defines deployment environments like development, staging, or production:

```dsl
deploymentEnvironment <name> {
    [deployment nodes]
}
```

**Example:**
```dsl
production = deploymentEnvironment "Production" {
    deploymentNode "AWS" {
        deploymentNode "ECS Cluster" {
            containerInstance backend
        }
    }
}
```

### Deployment Node

Infrastructure hosting element (servers, containers, clusters):

```dsl
deploymentNode <name> [description] [technology] [tags] [instances] {
    [child nodes or instances]
}
```

**Example:**
```dsl
deploymentNode "Load Balancer" "AWS ALB" "AWS Application Load Balancer" "Infrastructure" {
    deploymentNode "ECS Service" "Container service" "AWS ECS" 1..4 {
        containerInstance api
    }
}
```

**Default tags:** `Element`, `Deployment Node`

**Instances:** Can be a static number or range (e.g., `1..N`, `1..4`)

**Supports nesting** for hierarchical infrastructure

### Infrastructure Node

Non-application infrastructure components (load balancers, firewalls, DNS):

```dsl
infrastructureNode <name> [description] [technology] [tags] {
    [properties]
}
```

**Example:**
```dsl
infrastructureNode "Load Balancer" "Distributes traffic" "Nginx" "Infrastructure"
```

**Default tags:** `Element`, `Infrastructure Node`

### Software System Instance / Container Instance

Deployed instances of systems or containers:

```dsl
softwareSystemInstance <identifier> [deploymentGroups] [tags]
containerInstance <identifier> [deploymentGroups] [tags]
```

**Example:**
```dsl
deploymentNode "Web Server" {
    containerInstance webapp "Production" "Live"
}
```

## Relationships

Uni-directional connections between elements using the `->` operator.

### Explicit Relationships

```dsl
<source> -> <destination> [description] [technology] [tags]
```

**Example:**
```dsl
user -> webapp "Uses" "HTTPS"
webapp -> api "Makes API calls" "REST/HTTPS"
api -> database "Reads/writes" "SQL/TCP"
```

### Implicit Relationships

When defining relationships within an element's scope, the source identifier can be omitted:

```dsl
person user {
    -> webapp "Uses" "HTTPS"
}
```

This is equivalent to using the `this` identifier:

```dsl
person user {
    this -> webapp "Uses" "HTTPS"
}
```

### Valid Relationship Types

- **Person** → Person, Software System, Container, Component
- **Software System** → Person, Software System, Container, Component
- **Container** → Person, Software System, Container, Component
- **Component** → Person, Software System, Container, Component
- **Deployment nodes** → Deployment nodes
- **Infrastructure nodes** → Deployment nodes, infrastructure nodes, system/container instances

### Relationship Properties

```dsl
source -> destination "Description" "Technology" {
    tags "Tag1" "Tag2"
    url "https://example.com/api-docs"
    properties {
        protocol "HTTPS"
        port "443"
    }
}
```

**Default tag:** `Relationship`

## Element Properties

All elements support these common properties:

### Tags

Label elements for styling and filtering:

```dsl
element "Name" "Description" "Tag1,Tag2,Tag3"
# OR
element "Name" "Description" {
    tags "Tag1"
    tags "Tag2" "Tag3"
}
```

Tags can be specified comma-separated or individually.

### URL

Associate a web address with an element:

```dsl
element "Name" {
    url "https://example.com/docs"
}
```

### Properties

Custom name/value pairs for metadata:

```dsl
element "Name" {
    properties {
        owner "Team A"
        criticality "High"
        version "2.1.0"
    }
}
```

### Perspectives

Named viewpoints with descriptions:

```dsl
element "Name" {
    perspectives {
        "Security" "Uses OAuth 2.0 for authentication"
        "Performance" "Can handle 10k requests/second"
    }
}
```

### Description

Text describing the element:

```dsl
element "Name" {
    description "Detailed description of the element"
}
```

### Technology

Implementation details (for containers, components, nodes):

```dsl
container "API" "REST API" "Java, Spring Boot, PostgreSQL"
```

## Advanced Features

### !element and !elements

Query and bulk-modify existing elements:

```dsl
# Modify a single element
!element <identifier> {
    tags "NewTag"
    properties { key "value" }
}

# Bulk operations using expressions
!elements <expression> {
    tags "NewTag"
}
```

### !relationship and !relationships

Locate and modify existing relationships:

```dsl
# Modify a single relationship
!relationship <identifier> {
    tags "Async"
}

# Bulk operations
!relationships <expression> {
    tags "Critical"
}
```

### !include

Modularize DSL across multiple files:

```dsl
workspace {
    model {
        !include model/people.dsl
        !include model/systems.dsl
    }

    views {
        !include views/context-views.dsl
        !include views/container-views.dsl
    }
}
```

### !docs and !adrs

Attach documentation and architecture decision records:

```dsl
workspace {
    !docs docs/
    !adrs adrs/
}
```

### !script

Execute inline JVM scripts for dynamic model generation:

```dsl
workspace {
    model {
        !script groovy {
            // Groovy code here
            workspace.model.addPerson("Dynamic User", "Created by script")
        }
    }
}
```

Supports Java, Groovy, Kotlin, and JRuby.

## Identifiers

Elements can be assigned identifiers for reference:

```dsl
user = person "User" "End user"
system = softwareSystem "System" "Main system"

user -> system "Uses"
```

**Important:** "Automatically generated view keys are not guaranteed to be stable over time." Always specify explicit keys for production workspaces.

## Default Behavior

If a workspace doesn't have a views block, a default set of views will be defined automatically. Explicit view definitions override this default behavior.

## Best Practices

1. **Use explicit identifiers** for elements that will be referenced
2. **Organize with !include** for large models
3. **Leverage implicit relationships** within element scopes for cleaner syntax
4. **Apply tags consistently** for styling and filtering
5. **Document with perspectives** for different stakeholder views
6. **Version control your DSL** as source code
7. **Use !script cautiously** - prefer declarative DSL where possible

## File Format

DSL files typically use the `.dsl` extension and should be UTF-8 encoded.

## Compatibility Notes

Element and relationship styles are designed to work with Structurizr cloud service, on-premises installation, and Lite. They may not be fully supported by PlantUML, Mermaid, and other export formats.
