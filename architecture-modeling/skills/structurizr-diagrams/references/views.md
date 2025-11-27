# Structurizr DSL Views Reference

Complete guide to all view types and configurations in Structurizr DSL.

## Overview

Structurizr DSL supports nine distinct view types for visualizing software architecture at different levels of abstraction. All views support common features like filtering, auto-layout, animations, and styling.

## View Types

### System Landscape View

Shows all people and software systems in a single diagram - the broadest architectural perspective.

**Syntax:**
```dsl
systemLandscape [key] [description] {
    include <*|element identifier|expression>
    exclude <element identifier|expression>
    autoLayout [tb|bt|lr|rl] [rankSeparation] [nodeSeparation]
    default
    animation { ... }
    title "Custom Title"
    description "Custom Description"
    properties { ... }
}
```

**Example:**
```dsl
views {
    systemLandscape "SystemLandscape" "Overview of all systems" {
        include *
        autoLayout
        title "Enterprise Architecture - System Landscape"
    }
}
```

**Scope:** All people and software systems

**Notes:**
- Optional auto-generated key if not specified
- "Automatically generated view keys are not guaranteed to be stable over time"

### System Context View

Depicts a specific software system with its external dependencies and users.

**Syntax:**
```dsl
systemContext <software system identifier> [key] [description] {
    include <*|element identifier|expression>
    exclude <element identifier|expression>
    autoLayout [tb|bt|lr|rl] [rankSeparation] [nodeSeparation]
    default
    animation { ... }
    title "Custom Title"
    description "Custom Description"
    properties { ... }
}
```

**Example:**
```dsl
views {
    systemContext mainSystem "MainSystemContext" "Context view for main system" {
        include *
        exclude "element.tag==Internal"
        autoLayout lr
        title "Main System - System Context"
    }
}
```

**Scope:** People, other software systems, and external containers connected to the specified system

**Use case:** Understanding external dependencies and users of a system

### Container View

Illustrates internal container structure within a software system.

**Syntax:**
```dsl
container <software system identifier> [key] [description] {
    include <*|element identifier|expression>
    exclude <element identifier|expression>
    autoLayout [tb|bt|lr|rl] [rankSeparation] [nodeSeparation]
    default
    animation { ... }
    title "Custom Title"
    description "Custom Description"
    properties { ... }
}
```

**Example:**
```dsl
views {
    container mainSystem "Containers" "Container view" {
        include *
        autoLayout tb
    }

    # Focused view showing only backend components
    container mainSystem "BackendView" "Backend containers only" {
        include backend database cache messageQueue
        autoLayout lr
        title "Backend Infrastructure"
    }
}
```

**Scope:** All containers within the specified system, plus connected external systems and people

**Use case:** Revealing application architecture and major technology choices

### Component View

Details internal component structure within a container.

**Syntax:**
```dsl
component <container identifier> [key] [description] {
    include <*|element identifier|expression>
    exclude <element identifier|expression>
    autoLayout [tb|bt|lr|rl] [rankSeparation] [nodeSeparation]
    default
    animation { ... }
    title "Custom Title"
    description "Custom Description"
    properties { ... }
}
```

**Example:**
```dsl
views {
    component backend "BackendComponents" "Backend internal structure" {
        include *
        autoLayout tb 150 150
        title "Backend API - Component Diagram"
    }
}
```

**Scope:** Components within the container, plus connected external elements (containers, systems, people)

**Use case:** Lowest-level structural detail for a specific deployable unit

### Filtered View

Creates a derived view from an existing diagram by filtering elements based on tags.

**Syntax:**
```dsl
filtered <baseKey> <include|exclude> <tags> [key] [description] {
    # Additional overrides
}
```

**Example:**
```dsl
views {
    # Base view
    container mainSystem "AllContainers" {
        include *
        autoLayout
    }

    # Filtered view showing only critical components
    filtered "AllContainers" include "Critical" "CriticalOnly" "Critical components" {
        title "Critical Infrastructure Only"
    }

    # Filtered view excluding external systems
    filtered "AllContainers" exclude "External" "InternalOnly" "Internal systems"
}
```

**Base view types:** System Landscape, System Context, Container, or Component

**Mode:**
- `include`: Show only elements with specified tags
- `exclude`: Hide elements with specified tags

**Use case:** Creating role-specific or concern-specific perspectives from a master view

### Dynamic View

Demonstrates runtime behavior and sequences through numbered interaction steps.

**Syntax:**
```dsl
dynamic <*|software system identifier|container identifier> [key] [description] {
    <element identifier> -> <element identifier> [description] [technology]
    [order:] <relationship identifier> [description]
    autoLayout [tb|bt|lr|rl] [rankSeparation] [nodeSeparation]
    default
    title "Custom Title"
    description "Custom Description"
    properties { ... }
}
```

**Example:**
```dsl
views {
    dynamic mainSystem "UserLogin" "User login flow" {
        user -> webapp "1. Enters credentials"
        webapp -> api "2. POST /auth/login"
        api -> database "3. Verify credentials"
        database -> api "4. Return user data"
        api -> cache "5. Store session"
        api -> webapp "6. Return JWT token"
        webapp -> user "7. Display dashboard"
        autoLayout lr
        title "Authentication Flow"
    }

    dynamic backend "DataProcessing" "Data processing sequence" {
        1: controller -> validator "Validate input"
        2: validator -> service "Process request"
        3: service -> repository "Fetch data"
        4: repository -> service "Return data"
        5: service -> controller "Return response"
        autoLayout tb
    }
}
```

**Scope options:**
- `*`: People and software systems only
- Software system identifier: Containers and related elements
- Container identifier: Components and related elements

**Numbering:** Steps are automatically numbered in sequence, or use explicit `order:` prefix

**Use case:** Showing how elements interact during specific scenarios or workflows

### Deployment View

Visualizes infrastructure topology and runtime artifacts.

**Syntax:**
```dsl
deployment <*|software system identifier> <environment> [key] [description] {
    include <*|element identifier|expression>
    exclude <element identifier|expression>
    autoLayout [tb|bt|lr|rl] [rankSeparation] [nodeSeparation]
    default
    animation { ... }
    title "Custom Title"
    description "Custom Description"
    properties { ... }
}
```

**Example:**
```dsl
views {
    deployment mainSystem "Production" "ProductionDeployment" "Production infrastructure" {
        include *
        autoLayout tb
        title "Production Deployment"
    }

    deployment * "Staging" "StagingDeployment" "Staging environment" {
        include *
        autoLayout lr
    }
}
```

**Scope options:**
- `*`: All deployment nodes and instances in the environment
- Software system identifier: Specific system instances and hosting infrastructure

**Use case:** Showing deployment nodes, instances, and infrastructure components

### Custom View

Accommodates non-standard diagram types using "custom elements" outside the C4 model hierarchy.

**Syntax:**
```dsl
custom [key] [title] [description] {
    include <*|element identifier|expression>
    exclude <element identifier|expression>
    autoLayout [tb|bt|lr|rl] [rankSeparation] [nodeSeparation]
    default
    animation { ... }
    title "Custom Title"
    description "Custom Description"
    properties { ... }
}
```

**Example:**
```dsl
model {
    group "Network Topology" {
        router = element "Router" {
            tags "NetworkDevice"
        }
        switch = element "Switch" {
            tags "NetworkDevice"
        }
        firewall = element "Firewall" {
            tags "NetworkDevice"
        }
    }
}

views {
    custom "NetworkDiagram" "Network Topology" {
        include element.tag==NetworkDevice
        autoLayout lr
    }
}
```

**Restriction:** Only custom elements permitted on this view type

**Use case:** Diagrams that don't fit C4 model structure (network topology, data flows, etc.)

### Image View

Embeds external diagram formats within workspaces.

**Syntax:**
```dsl
image <*|element identifier> [key] {
    plantuml <file|url|viewKey>
    mermaid <file|url|viewKey>
    kroki <format> <file|url>
    image <file|url>
    title "Custom Title"
    contentType "image/png|image/jpeg|image/svg+xml|..."
}
```

**Example:**
```dsl
views {
    # Embed PlantUML diagram
    image * "SequenceDiagram" {
        plantuml diagrams/sequence.puml
        title "User Authentication Sequence"
    }

    # Embed Mermaid diagram
    image mainSystem "Flowchart" {
        mermaid diagrams/workflow.mmd
        title "Request Processing Flow"
    }

    # Embed Kroki diagram
    image * "StateMachine" {
        kroki graphviz diagrams/states.dot
        title "Order State Machine"
    }

    # Embed static image
    image * "Screenshot" {
        image screenshots/ui-mockup.png
        contentType "image/png"
        title "UI Mockup"
    }
}
```

**Supported sources:**
- **PlantUML:** `.puml` files or URLs
- **Mermaid:** `.mmd` files or URLs
- **Kroki:** Multiple formats (graphviz, ditaa, blockdiag, etc.)
- **Static images:** PNG, JPEG, SVG, etc.

**Use case:** Integrating diagrams from other tools into Structurizr workspaces

## Common View Features

All views support these features:

### Include/Exclude

Filter elements and relationships in the view:

```dsl
# Include everything
include *

# Include specific elements
include user webapp api database

# Include by tag expression
include "element.tag==Critical"

# Include relationships
include "relationship.tag==Sync"

# Exclude elements
exclude externalSystem legacySystem

# Exclude by tag
exclude "element.tag==Deprecated"
```

**Expression syntax:**
- `element.tag==TagName`: Elements with specific tag
- `relationship.tag==TagName`: Relationships with specific tag
- `element.type==Person`: Elements of specific type
- Supports `==`, `!=`, `||`, `&&`

### AutoLayout

Enable automatic diagram layout with optional direction and spacing:

```dsl
# Default top-to-bottom layout
autoLayout

# Specific direction
autoLayout lr  # left-to-right
autoLayout rl  # right-to-left
autoLayout tb  # top-to-bottom
autoLayout bt  # bottom-to-top

# With custom spacing
autoLayout tb 150 150  # rankSeparation nodeSeparation in pixels
```

**Layout directions:**
- `tb`: Top-to-bottom (default)
- `bt`: Bottom-to-top
- `lr`: Left-to-right
- `rl`: Right-to-left

**Spacing parameters:**
- `rankSeparation`: Space between ranks/layers (default: 100)
- `nodeSeparation`: Space between nodes in same rank (default: 100)

### Animation

Define step-by-step element visibility:

```dsl
systemContext mainSystem "Animated" {
    include *
    animation {
        user
        webapp
        api
        database
    }
    autoLayout
}
```

Elements appear in the defined sequence, creating an animated progression through the diagram.

### Title and Description

Override the view title and description:

```dsl
container mainSystem "Containers" {
    include *
    title "Application Architecture - Container View"
    description "Shows all containers within the main system and their relationships"
    autoLayout
}
```

### Properties

Attach custom metadata to views:

```dsl
systemContext mainSystem "Context" {
    include *
    properties {
        audience "Technical Architects"
        status "Draft"
        version "1.2"
    }
    autoLayout
}
```

### Default View

Mark a view as the default (shown first in tools):

```dsl
systemContext mainSystem "Context" {
    include *
    default
    autoLayout
}
```

## View Keys

Each view has a unique key for reference:

```dsl
# Explicit key
systemContext mainSystem "MyContextKey" "Description"

# Auto-generated key (not recommended for production)
systemContext mainSystem
```

**Important:** "Automatically generated view keys are not guaranteed to be stable over time." Always specify explicit keys for production workspaces.

## Multiple Views

Create multiple perspectives of the same system:

```dsl
views {
    # High-level context
    systemContext mainSystem "FullContext" "Complete system context" {
        include *
        autoLayout
    }

    # All containers
    container mainSystem "AllContainers" "Complete container view" {
        include *
        autoLayout tb
    }

    # Frontend focused
    container mainSystem "FrontendView" "Frontend architecture" {
        include webapp spa mobileApp apiGateway
        autoLayout lr
    }

    # Backend focused
    container mainSystem "BackendView" "Backend services" {
        include api database cache messageQueue
        autoLayout tb
    }

    # Data layer
    container mainSystem "DataView" "Data architecture" {
        include database cache dataWarehouse
        autoLayout lr
    }
}
```

## Default Views

If a workspace doesn't have a views block, Structurizr automatically generates a default set of views. Explicit view definitions override this default behavior.

## Best Practices

1. **Always specify explicit keys** for production workspaces
2. **Use meaningful view keys** that reflect the view's purpose
3. **Create multiple perspectives** for different audiences
4. **Use filtered views** to show role-specific or concern-specific views
5. **Leverage animations** to guide viewers through complex diagrams
6. **Apply consistent titles** following a naming convention
7. **Use auto-layout initially**, then switch to manual if needed
8. **Tag elements strategically** to enable powerful filtering
9. **Document with properties** for view metadata
10. **Start with context, progress to detail** (landscape → context → container → component)

## View Hierarchy

Recommended progression through view types:

1. **System Landscape** - Enterprise-wide overview
2. **System Context** - Individual system boundaries
3. **Container** - Application architecture
4. **Component** - Detailed internal structure
5. **Dynamic** - Runtime behavior
6. **Deployment** - Infrastructure topology

This hierarchy follows the C4 model's zoom-in approach to architecture documentation.
