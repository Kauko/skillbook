---
description: Use when generating C4 architecture diagrams from the Overarch model.
---

# Structurizr Diagrams Skill

Use this skill when generating C4 architecture diagrams from the Overarch EDN model.

## Reference Documentation

Comprehensive Structurizr DSL documentation is available in the `references/` directory:

- **[Language Reference](references/language.md)** - Complete DSL syntax, elements, relationships, and properties
- **[Views Reference](references/views.md)** - All view types (context, container, component, dynamic, deployment, etc.)
- **[Styling Reference](references/styling.md)** - Themes, colors, shapes, element/relationship styles, and branding

Consult these references for detailed syntax, advanced features, and examples.

## Prerequisites Check

Before proceeding, verify Structurizr CLI is installed:

```bash
structurizr-cli --version
```

If not installed, guide the user to install:

**macOS (Homebrew):**
```bash
brew install structurizr-cli
```

**Manual Installation:**
```bash
# Download from https://github.com/structurizr/cli/releases
# Extract and add to PATH
curl -L https://github.com/structurizr/cli/releases/latest/download/structurizr-cli.zip -o structurizr-cli.zip
unzip structurizr-cli.zip
# Add to PATH or move to /usr/local/bin
```

**Verify installation:**
```bash
structurizr-cli --version
```

## Core Workflow

### 1. Read Overarch Model

Read the source model from `vault/architecture/model.edn`:

```bash
cat vault/architecture/model.edn
```

Check file modification time to detect staleness later.

### 2. Translate EDN to Structurizr DSL

Parse the Overarch EDN and translate to Structurizr DSL format.

**Mapping Rules:**

**Elements:**
- `:person` → `person "Name" "Description"`
- `:system` → `softwareSystem "Name" "Description"`
- `:container` → `container "Name" "Description" "Technology"`
- `:component` → `component "Name" "Description" "Technology"`
- `:database` → `container "Name" "Description" "Technology" "Database"`

**Relationships:**
- `:rel` → `-> "Destination" "Description" "Technology"`

**Views:**
- `:context-view` → `systemContext`
- `:container-view` → `container`
- `:component-view` → `component`

### 3. Generate Structurizr DSL Template

Create DSL at `vault/architecture/workspace.dsl`:

```dsl
workspace "System Architecture" "Architecture description" {

    model {
        # People
        user = person "User" "System user"

        # Software Systems
        mainSystem = softwareSystem "Main System" "Primary software system" {
            # Containers
            frontend = container "Frontend" "User interface" "React, TypeScript" "Web Browser"
            backend = container "Backend API" "Business logic" "Node.js, Express" "Service"
            database = container "Database" "Data store" "PostgreSQL" "Database"

            # Relationships within system
            frontend -> backend "Makes API calls to" "REST/HTTPS"
            backend -> database "Reads from and writes to" "SQL/TCP"
        }

        # External relationships
        user -> frontend "Uses" "HTTPS"

        # External systems (if any)
        # externalSystem = softwareSystem "External System" "Description" "External"
        # backend -> externalSystem "Integrates with" "REST API"
    }

    views {
        systemContext mainSystem "SystemContext" {
            include *
            autoLayout
        }

        container mainSystem "Containers" {
            include *
            autoLayout
        }

        # Uncomment to add component view
        # component backend "BackendComponents" {
        #     include *
        #     autoLayout
        # }

        styles {
            element "Person" {
                shape person
                background #08427B
                color #ffffff
            }
            element "Software System" {
                background #1168BD
                color #ffffff
            }
            element "Container" {
                background #438DD5
                color #ffffff
            }
            element "Component" {
                background #85BBF0
                color #000000
            }
            element "Database" {
                shape cylinder
                background #438DD5
                color #ffffff
            }
            element "External" {
                background #999999
                color #ffffff
            }
        }

        theme default
    }
}
```

### 4. Generate Diagrams

Create output directory and generate diagrams:

```bash
mkdir -p vault/architecture/diagrams
cd vault/architecture
structurizr-cli export -workspace workspace.dsl -format png -output diagrams/
```

This generates:
- `SystemContext.png` - System context diagram
- `Containers.png` - Container diagram
- `BackendComponents.png` - Component diagram (if defined)

### 5. Create Markdown Documentation

Generate `vault/architecture/diagrams.md` with embedded images:

```markdown
# Architecture Diagrams

Generated from Overarch model on [DATE]

## System Context Diagram

Shows the system in its environment with users and external dependencies.

![System Context](diagrams/SystemContext.png)

## Container Diagram

Shows the major containers (applications, services, databases) within the system.

![Containers](diagrams/Containers.png)

## Component Diagrams

### Backend Components

Shows the internal structure of the Backend API service.

![Backend Components](diagrams/BackendComponents.png)

---

**Source:** `vault/architecture/model.edn`
**Generated:** [TIMESTAMP]
**Tool:** Structurizr CLI

To regenerate: Run the `sync-architecture` skill or `structurizr-diagrams` skill.
```

## Translation Examples

### Example 1: Simple System

**Overarch EDN:**
```clojure
{:model
 {:nodes
  [{:el :person :id :user :name "User" :desc "End user"}
   {:el :system :id :app :name "Application" :desc "Main app"}
   {:el :database :id :db :name "Database" :desc "Data store" :tech ["PostgreSQL"]}]

  :relations
  [{:el :rel :from :user :to :app :name "Uses"}
   {:el :rel :from :app :to :db :name "Reads/Writes" :tech ["SQL"]}]}}
```

**Structurizr DSL:**
```dsl
workspace "System Architecture" {
    model {
        user = person "User" "End user"
        app = softwareSystem "Application" "Main app"
        db = softwareSystem "Database" "Data store" "External"

        user -> app "Uses"
        app -> db "Reads/Writes" "SQL"
    }

    views {
        systemContext app "SystemContext" {
            include *
            autoLayout
        }
    }
}
```

### Example 2: Containers with Technology

**Overarch EDN:**
```clojure
{:model
 {:nodes
  [{:el :container :id :web :name "Web App" :tech ["React" "TypeScript"] :ct "SPA"}
   {:el :container :id :api :name "API" :tech ["Go" "Gin"] :ct "Service"}
   {:el :database :id :cache :name "Cache" :tech ["Redis"] :ct "Cache"}]

  :relations
  [{:el :rel :from :web :to :api :name "Calls" :tech ["REST" "HTTPS"]}
   {:el :rel :from :api :to :cache :name "Caches in" :tech ["Redis Protocol"]}]}}
```

**Structurizr DSL:**
```dsl
model {
    system = softwareSystem "System" {
        web = container "Web App" "SPA" "React, TypeScript" "Web Browser"
        api = container "API" "Service" "Go, Gin" "Service"
        cache = container "Cache" "Cache" "Redis" "Database"

        web -> api "Calls" "REST/HTTPS"
        api -> cache "Caches in" "Redis Protocol"
    }
}

views {
    container system "Containers" {
        include *
        autoLayout
    }
}
```

## Staleness Detection

Check if DSL/diagrams are stale:

```bash
# Check model vs DSL
if [ vault/architecture/model.edn -nt vault/architecture/workspace.dsl ]; then
    echo "WARNING: model.edn is newer than workspace.dsl"
    echo "DSL needs regeneration"
fi

# Check DSL vs diagrams
if [ vault/architecture/workspace.dsl -nt vault/architecture/diagrams/SystemContext.png ]; then
    echo "WARNING: workspace.dsl is newer than diagrams"
    echo "Diagrams need regeneration"
fi
```

## Diagram Customization

### Custom Styles

Add custom styling to DSL using element and relationship styles:

```dsl
styles {
    element "Person" {
        shape person
        background #08427B
        color #ffffff
        fontSize 22
    }
    element "Microservice" {
        background #91C4F2
        color #000000
        shape hexagon
    }
    element "External" {
        background #999999
        color #ffffff
        opacity 70
        border dashed
    }
    element "Critical" {
        stroke #ff0000
        strokeWidth 4
    }
    element "Database" {
        shape cylinder
        background #438DD5
        color #ffffff
    }

    relationship "Synchronous" {
        style solid
        thickness 2
        color #000000
    }
    relationship "Asynchronous" {
        style dashed
        thickness 2
        color #666666
    }
}
```

**See:** [Styling Reference](references/styling.md) for complete styling guide including shapes, colors, themes, and branding.

### Layout Control

Control diagram layout with auto-layout directions and spacing:

```dsl
systemContext app "SystemContext" {
    include *
    autoLayout tb  # top-to-bottom
}

container app "Containers" {
    include *
    autoLayout lr 150 150  # left-to-right with custom spacing
}

# Available directions: tb (top-bottom), bt (bottom-top), lr (left-right), rl (right-left)
```

**See:** [Views Reference](references/views.md#autolayout) for layout options.

### Multiple Views

Generate multiple perspectives for different audiences:

```dsl
views {
    # High-level context
    systemContext mainSystem "FullContext" "Complete system context" {
        include *
        autoLayout
    }

    # All containers
    container mainSystem "AllContainers" {
        include *
        autoLayout tb
    }

    # Backend-focused view
    container mainSystem "BackendView" "Backend services" {
        include api database cache messageQueue
        autoLayout lr
        title "Backend Architecture"
    }

    # Deployment view
    deployment mainSystem "Production" "ProductionDeployment" "Production infrastructure" {
        include *
        autoLayout tb
    }
}
```

**See:** [Views Reference](references/views.md) for all view types and options.

## Advanced Features

For detailed documentation on all features, see the [Reference Documentation](#reference-documentation) above.

### Deployment Diagrams

Add deployment information to show infrastructure topology:

```dsl
model {
    production = deploymentEnvironment "Production" {
        deploymentNode "AWS" {
            deploymentNode "ECS Cluster" "Container orchestration" "AWS ECS" {
                containerInstance backend
            }
            deploymentNode "RDS" "Database hosting" "AWS RDS" {
                containerInstance database
            }
        }
    }
}

views {
    deployment mainSystem "Production" "ProductionDeployment" {
        include *
        autoLayout tb
        title "Production Infrastructure"
    }
}
```

**See:** [Views Reference](references/views.md#deployment-view) for complete deployment view syntax.

### Dynamic Diagrams

Show interaction flows and sequences:

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
}
```

**See:** [Views Reference](references/views.md#dynamic-view) for dynamic view details.

### Tags and Filtering

Use tags to organize elements and create filtered views:

```dsl
model {
    backend = container "Backend" "API" "Node.js" {
        tags "Critical" "Internal" "Backend"
    }

    external = softwareSystem "External API" "Third party" {
        tags "External" "ThirdParty"
    }
}

views {
    # Base view with all containers
    container mainSystem "AllContainers" {
        include *
        autoLayout
    }

    # Filtered view showing only internal components
    filtered "AllContainers" include "Internal" "InternalOnly" "Internal systems only"

    # Filtered view excluding external systems
    filtered "AllContainers" exclude "External" "NoExternal" "Without external dependencies"
}
```

**See:** [Views Reference](references/views.md#filtered-view) for filtering expressions and patterns.

## Output Formats

Structurizr CLI supports multiple formats:

```bash
# PNG (default)
structurizr-cli export -workspace workspace.dsl -format png -output diagrams/

# SVG (vector graphics)
structurizr-cli export -workspace workspace.dsl -format svg -output diagrams/

# PlantUML (for further customization)
structurizr-cli export -workspace workspace.dsl -format plantuml -output diagrams/

# Mermaid (for Markdown integration)
structurizr-cli export -workspace workspace.dsl -format mermaid -output diagrams/
```

## Error Handling

Common issues and solutions:

**Error: "Workspace file not found"**
```bash
# Check file exists
ls -la vault/architecture/workspace.dsl
# Check current directory
pwd
```

**Error: "Invalid DSL syntax"**
```bash
# Validate DSL
structurizr-cli validate -workspace workspace.dsl
```

**Error: "Permission denied"**
```bash
# Check permissions
chmod 644 vault/architecture/workspace.dsl
mkdir -p vault/architecture/diagrams
chmod 755 vault/architecture/diagrams
```

**Error: "Command not found: structurizr-cli"**
```bash
# Check installation
which structurizr-cli
# Check PATH
echo $PATH
# Reinstall if needed
brew reinstall structurizr-cli
```

## Integration with Overarch

### Full Pipeline

1. **Edit** → `vault/architecture/model.edn` (Overarch format)
2. **Translate** → `vault/architecture/workspace.dsl` (Structurizr DSL)
3. **Generate** → `vault/architecture/diagrams/*.png` (Visual diagrams)
4. **Document** → `vault/architecture/diagrams.md` (Markdown with images)

### Automation

Create a script for regeneration:

```bash
#!/bin/bash
# vault/architecture/regenerate.sh

set -e

echo "Regenerating architecture diagrams..."

# Translate Overarch to Structurizr
echo "1. Translating model.edn to workspace.dsl..."
# (This would be done by Claude using this skill)

# Generate diagrams
echo "2. Generating diagrams..."
cd vault/architecture
structurizr-cli export -workspace workspace.dsl -format png -output diagrams/

# Update documentation
echo "3. Updating diagrams.md..."
# (This would be done by Claude using this skill)

echo "Done! Diagrams regenerated."
```

## Best Practices

1. **Single Source of Truth**: Always edit `model.edn`, never edit DSL directly
2. **Version Control**: Commit both EDN model and generated DSL/diagrams
3. **Consistent Naming**: Use same IDs/names between Overarch and Structurizr
4. **Auto Layout**: Use `autoLayout` for initial diagrams, refine later if needed
5. **View Hierarchy**: Start with context, then container, then component
6. **Tags**: Use tags to organize and filter complex models
7. **Documentation**: Always generate `diagrams.md` with timestamps
8. **Validation**: Validate DSL before generating diagrams
9. **Explicit Keys**: Always specify view keys explicitly (not auto-generated)
10. **Consult References**: Use the detailed [reference documentation](#reference-documentation) for advanced features

## Output Checklist

After generating diagrams, confirm:

- [ ] `workspace.dsl` created/updated at `vault/architecture/workspace.dsl`
- [ ] Diagrams generated in `vault/architecture/diagrams/`
- [ ] `diagrams.md` created with embedded images
- [ ] All views from Overarch model represented
- [ ] Timestamps added to documentation
- [ ] No errors during generation
- [ ] Diagrams visually correct and readable

## Example Usage

**User Request:** "Generate C4 diagrams from my Overarch model"

**Assistant Actions:**
1. Check Structurizr CLI installation
2. Read `vault/architecture/model.edn`
3. Parse EDN structure
4. Generate Structurizr DSL at `vault/architecture/workspace.dsl`
5. Create diagrams directory
6. Run `structurizr-cli export` to generate PNGs
7. Create `vault/architecture/diagrams.md` with embedded images
8. Verify all diagrams generated successfully
9. Report what was created

**User Request:** "Update the container diagram with the new services I added"

**Assistant Actions:**
1. Read updated `vault/architecture/model.edn`
2. Check if model is newer than DSL (staleness check)
3. Regenerate `workspace.dsl` from updated model
4. Regenerate diagrams
5. Update `diagrams.md` with new timestamp
6. Show what changed (diff summary)

---

## Additional Resources

This skill focuses on the workflow for generating C4 diagrams from Overarch models. For comprehensive Structurizr DSL documentation:

- **[Language Reference](references/language.md)** - Complete syntax, elements, relationships, properties, and advanced features like !include, !script, and !elements
- **[Views Reference](references/views.md)** - All view types with detailed syntax and options
- **[Styling Reference](references/styling.md)** - Complete styling guide with shapes, colors, themes, branding, and visual design patterns

These references contain:
- Full DSL syntax specifications
- All element and relationship properties
- Complete view type documentation (9 types)
- Comprehensive styling options (14 shapes, colors, themes)
- Advanced features (!include, !script, !elements, filtering expressions)
- Deployment and dynamic diagram patterns
- Animation and filtering capabilities
- Export format compatibility notes

---

Remember: Diagrams are derived artifacts. Always edit the Overarch model first, then regenerate diagrams.
