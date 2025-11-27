# Overarch Views Reference

This document provides comprehensive details on all view types available in Overarch for visualizing architecture models.

## View Fundamentals

Views specify what to render from the model and how. They are separate from model elements, enabling:
- Multiple views of the same model
- Different abstraction levels
- Focused views for specific audiences
- Reusable view patterns

## Common View Properties

All views share these common properties:

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `:el` | keyword | Yes | View type (e.g., `:context-view`, `:container-view`) |
| `:id` | namespaced keyword | Yes | Unique identifier |
| `:title` | string | No | View title shown in rendered diagram |
| `:spec` | map | No | View specification (selection, includes, layout) |
| `:ct` | vector | No | Explicit content list (ordered references) |
| `:layout` | keyword | No | Layout direction (`:top-down`, `:left-right`) |
| `:styles` | set | No | Custom styling definitions |
| `:themes` | vector | No | Theme references |

## View Specification (`:spec`)

The `:spec` map controls what elements appear in the view:

```clojure
{:spec
 {:selection {}              ; Element selection criteria
  :include [:relations       ; Automatic inclusions
            :related]
  :layout :top-down          ; Layout direction
  :expand-external false     ; Show internals of external systems
  :node-separation 50        ; PlantUML layout parameter
  :rank-separation 100       ; PlantUML vertical spacing
  :sprite-libs [:awslib14    ; Technology icon libraries
                :azure]
  :skinparams {}}}           ; PlantUML skin parameters
```

## C4 Architecture Views

### System Context View

Shows the system in its environment with external actors and systems.

**Element Type:** `:context-view`

**Purpose:** Highest level of abstraction showing who uses the system and what it integrates with.

**Shows:**
- Systems (your system and external systems)
- Persons (users and external actors)
- Relationships between them

**Example:**
```clojure
{:el :context-view
 :id :views/system-context
 :title "System Context - Internet Banking"
 :spec {:include [:end-user
                  :banking.internet-banking/internet-banking-system
                  :banking.mainframe/mainframe-banking-system
                  :email/email-system]
        :layout :top-down}}
```

**Best Practices:**
- Keep simple (5-10 elements)
- Show only direct interactions
- Use for executive/stakeholder communication
- One per system

### System Landscape View

Shows multiple systems and their relationships.

**Element Type:** `:system-landscape-view`

**Purpose:** Enterprise-level view showing all systems in an organization.

**Shows:**
- Multiple systems
- High-level system interactions
- Organizational boundaries

**Example:**
```clojure
{:el :system-landscape-view
 :id :views/enterprise-landscape
 :title "Enterprise System Landscape"
 :spec {:selection {:el :system}
        :include [:relations]
        :layout :left-right}}
```

**Best Practices:**
- Use for enterprise architecture
- Group by business domain
- Show strategic system relationships
- Include only key systems

### Container View

Shows containers (deployable units) within a system.

**Element Type:** `:container-view`

**Purpose:** Shows the high-level technology choices and how containers communicate.

**Shows:**
- Containers (applications, services, databases)
- External systems and actors they interact with
- Technology stack for each container
- Communication patterns

**Example:**
```clojure
{:el :container-view
 :id :views/internet-banking-containers
 :title "Container View - Internet Banking System"
 :spec {:include [:banking.internet-banking/web-app
                  :banking.internet-banking/api-server
                  :banking.internet-banking/database
                  :end-user
                  :banking.mainframe/mainframe-banking-system]
        :include [:relations :related]
        :layout :top-down
        :sprite-libs [:awslib14]}}
```

**Best Practices:**
- Include 5-15 containers
- Show technology choices
- Include external dependencies
- One per system or subsystem

### Component View

Shows components within a container.

**Element Type:** `:component-view`

**Purpose:** Shows internal structure and organization of a container.

**Shows:**
- Components within a container
- Other containers they interact with
- Component responsibilities
- Internal architecture patterns

**Example:**
```clojure
{:el :component-view
 :id :views/api-server-components
 :title "Component View - API Server"
 :spec {:include [:banking.api/authentication-component
                  :banking.api/accounts-component
                  :banking.api/transactions-component
                  :banking.internet-banking/database
                  :banking.mainframe/mainframe-banking-system]
        :include [:relations :related]
        :layout :top-down}}
```

**Best Practices:**
- Focus on one container
- Show 5-15 components
- Group by responsibility
- Document component patterns

### Deployment View

Shows infrastructure and how containers are deployed.

**Element Type:** `:deployment-view`

**Purpose:** Shows deployment topology and infrastructure.

**Shows:**
- Infrastructure nodes (servers, clusters, etc.)
- Container instances deployed to nodes
- Network boundaries
- Infrastructure relationships

**Example:**
```clojure
{:el :deployment-view
 :id :views/production-deployment
 :title "Production Deployment"
 :spec {:selection {:el :node}
        :include [:deployed-to :link]
        :layout :top-down}}
```

**Best Practices:**
- One per environment (dev, staging, prod)
- Show redundancy and scaling
- Include network zones
- Document infrastructure dependencies

### Deployment Architecture View

Combined view showing both deployment topology and container architecture.

**Element Type:** `:deployment-architecture-view`

**Purpose:** Integrated view of containers and their deployment infrastructure.

**Example:**
```clojure
{:el :deployment-architecture-view
 :id :views/prod-deployment-architecture
 :title "Production Deployment Architecture"
 :spec {:selection {:namespace :infrastructure.production}
        :include [:deployed-to :link :relations]
        :layout :top-down}}
```

### Dynamic View

Shows runtime interaction sequences.

**Element Type:** `:dynamic-view`

**Purpose:** Shows step-by-step interactions for a specific scenario.

**Shows:**
- Numbered sequence of interactions
- Runtime behavior
- Message flow
- Interaction order

**Example:**
```clojure
{:el :dynamic-view
 :id :views/login-flow
 :title "User Login Flow"
 :ct [{:el :ref
       :id :banking/web-to-api
       :order 1}
      {:el :ref
       :id :banking/api-to-auth
       :order 2}
      {:el :ref
       :id :banking/auth-to-db
       :order 3}]}
```

**Best Practices:**
- One scenario per view
- Number steps clearly
- Keep sequences focused (5-15 steps)
- Document key workflows

## UML Design Views

### Code View

UML class diagram showing code structure.

**Element Type:** `:code-view`

**Purpose:** Shows classes, interfaces, and their relationships.

**Shows:**
- Classes, interfaces, protocols
- Class members (fields, methods)
- Inheritance, implementation
- Associations, dependencies

**Example:**
```clojure
{:el :code-view
 :id :views/domain-model
 :title "Domain Model"
 :spec {:selection {:namespace :banking.domain}
        :include [:inheritance :implementation :association]
        :layout :top-down}}
```

### State Machine View

Shows state machines and transitions.

**Element Type:** `:state-machine-view`

**Purpose:** Documents state-dependent behavior.

**Shows:**
- States and transitions
- Events triggering transitions
- Entry/exit actions
- State hierarchies

**Example:**
```clojure
{:el :state-machine-view
 :id :views/order-state-machine
 :title "Order Processing States"
 :spec {:include [:order/order-lifecycle]
        :layout :left-right}}
```

### Use Case View

Shows actors and their interactions with use cases.

**Element Type:** `:use-case-view`

**Purpose:** Documents functional requirements and user goals.

**Shows:**
- Actors and use cases
- Use case relationships (includes, extends)
- System boundary
- Actor interactions

**Example:**
```clojure
{:el :use-case-view
 :id :views/banking-use-cases
 :title "Internet Banking Use Cases"
 :spec {:selection {:namespace :banking.use-cases}
        :include [:uses :includes :extends]
        :layout :left-right}}
```

## Documentation Views

### Concept View

GraphViz concept map showing domain concepts.

**Element Type:** `:concept-view`

**Purpose:** Visualizes domain model and concept relationships.

**Shows:**
- Domain concepts
- Conceptual relationships
- Hierarchies and associations
- Knowledge structure

**Example:**
```clojure
{:el :concept-view
 :id :views/banking-concepts
 :title "Banking Domain Concepts"
 :spec {:selection {:el :concept}
        :include [:is-a :has :rel]
        :engine :dot}}
```

**Rendering:** Uses GraphViz instead of PlantUML.

**Layout Engines:**
- `:dot` - Hierarchical (default)
- `:neato` - Spring model
- `:fdp` - Force-directed
- `:sfdp` - Scalable force-directed
- `:circo` - Circular
- `:twopi` - Radial

### Glossary

Markdown table of all elements with descriptions.

**Element Type:** `:glossary`

**Purpose:** Generated documentation listing all model elements.

**Shows:**
- Element names and descriptions
- Technology stacks
- Element types
- Sorted alphabetically

**Example:**
```clojure
{:el :glossary
 :id :views/architecture-glossary
 :title "Architecture Glossary"
 :spec {:selection {}}}  ; Empty selection = all elements
```

**Output:** Markdown table format.

### Organization Structure View

Shows organizational hierarchy.

**Element Type:** `:organization-structure-view`

**Purpose:** Documents organizational structure and responsibilities.

**Shows:**
- Organization units
- Team hierarchies
- Responsibilities
- Reporting structure

**Example:**
```clojure
{:el :organization-structure-view
 :id :views/engineering-org
 :title "Engineering Organization"
 :spec {:selection {:namespace :company.engineering}
        :include [:contained-in]}}
```

### Deployment Structure View

Shows infrastructure hierarchy.

**Element Type:** `:deployment-structure-view`

**Purpose:** Documents infrastructure organization.

**Shows:**
- Node hierarchies
- Infrastructure layers
- Nested deployments

**Example:**
```clojure
{:el :deployment-structure-view
 :id :views/infrastructure-structure
 :title "Infrastructure Structure"
 :spec {:selection {:el :node}
        :layout :top-down}}
```

## Element Selection

The `:selection` map uses criteria to filter model elements:

### Selection by Element Type

```clojure
{:selection {:el :system}}           ; Single type
{:selection {:els [:system :person]}} ; Multiple types
```

### Selection by Namespace

```clojure
{:selection {:namespace :banking.internet-banking}}        ; Exact namespace
{:selection {:namespaces [:banking.api :banking.web]}}    ; Multiple namespaces
```

### Selection by ID

```clojure
{:selection {:id :banking.internet-banking/web-app}}      ; Single ID
{:selection {:ids [:banking/web-app :banking/api-server]}} ; Multiple IDs
```

### Selection by Technology

```clojure
{:selection {:tech "Java"}}                     ; Contains technology
{:selection {:techs ["Java" "Spring Boot"]}}    ; Contains any
{:selection {:all-techs ["Java" "PostgreSQL"]}} ; Contains all
```

### Selection by Tags

```clojure
{:selection {:tag :frontend}}                   ; Has tag
{:selection {:tags #{:frontend :public-api}}}   ; Has any tag
{:selection {:all-tags #{:security :audited}}}  ; Has all tags
```

### Selection by External Status

```clojure
{:selection {:external? true}}   ; External systems only
{:selection {:external? false}}  ; Internal systems only
```

### Selection by Maturity

```clojure
{:selection {:maturity :proposed}}              ; Single maturity
{:selection {:maturities [:proposed :deprecated]}} ; Multiple
```

### Selection by Relationship

```clojure
{:selection {:from :banking/web-app}}           ; Relations from element
{:selection {:to :banking/database}}            ; Relations to element
{:selection {:refers-to :banking/api-server}}   ; Elements referring to
{:selection {:referred-by :banking/web-app}}    ; Elements referred by
```

### Selection by Hierarchy

```clojure
{:selection {:child-of :banking/main-system}}       ; Direct children
{:selection {:parent-of :banking/web-app}}          ; Direct parent
{:selection {:descendant-of :banking/main-system}}  ; All descendants
{:selection {:ancestor-of :banking/component}}      ; All ancestors
```

### Text Pattern Matching

```clojure
{:selection {:name ".*Banking.*"}}    ; Regex in name
{:selection {:desc ".*API.*"}}        ; Regex in description
{:selection {:doc ".*security.*"}}    ; Regex in documentation
```

### Negation

Prefix any key with `!` to negate:

```clojure
{:selection {:el :system
             :!external? true}}       ; Internal systems only

{:selection {:namespace :banking
             :!tags #{:deprecated}}}  ; Not deprecated
```

### Conjunctions and Disjunctions

**Single map = AND (conjunction):**
```clojure
{:selection {:el :container
             :tech "Java"
             :tags #{:backend}}}  ; Containers AND Java AND backend tag
```

**Vector of maps = OR (disjunction):**
```clojure
{:selection [{:el :system}
             {:el :person}]}  ; Systems OR persons
```

### Empty Selection

```clojure
{:selection {}}  ; Selects ALL elements
```

Use with caution; typically too broad for useful views.

## Include Patterns

The `:include` key adds related elements automatically:

### Include Relations

```clojure
{:include [:relations]}  ; Include all relations between selected elements
```

### Include Related Elements

```clojure
{:include [:related]}  ; Include elements connected to selected elements
```

### Combined

```clojure
{:include [:relations :related]}  ; Both
```

### Explicit Element List

Instead of selection criteria, explicitly list elements:

```clojure
{:include [:banking/web-app
           :banking/api-server
           :banking/database]}
```

## Content List (`:ct`)

For precise control, use `:ct` to specify ordered content:

```clojure
{:ct [{:el :ref :id :banking/web-app}
      {:el :ref :id :banking/api-server}
      {:el :ref :id :banking/database}
      {:el :ref :id :banking/web-to-api}
      {:el :ref :id :banking/api-to-db}]}
```

**Use Cases:**
- Dynamic views (numbered sequences)
- Specific element ordering
- View-specific element customization

**Reference Overrides:**

```clojure
{:ct [{:el :ref
       :id :banking/web-app
       :name "Custom Name"       ; Override display name
       :desc "Custom description"
       :tech ["Updated Tech"]}]} ; Override tech stack
```

## Layout Control

### Layout Direction

```clojure
{:layout :top-down}   ; Vertical layout (default for most views)
{:layout :left-right} ; Horizontal layout
```

**Recommendations:**
- Context views: `:top-down`
- Container views: `:top-down`
- Sequence flows: `:left-right`
- Concept maps: `:left-right`

### PlantUML Layout Parameters

Fine-tune PlantUML rendering:

```clojure
{:spec {:node-separation 50    ; Horizontal spacing (pixels)
        :rank-separation 100   ; Vertical spacing (pixels)}}
```

### GraphViz Layout Engines

For concept views:

```clojure
{:spec {:engine :dot}}     ; Hierarchical (default)
{:spec {:engine :neato}}   ; Spring model
{:spec {:engine :fdp}}     ; Force-directed placement
{:spec {:engine :sfdp}}    ; Scalable force-directed
{:spec {:engine :circo}}   ; Circular layout
{:spec {:engine :twopi}}   ; Radial layout
```

## Styling and Themes

### Technology Sprites

Add technology icons from sprite libraries:

```clojure
{:spec {:sprite-libs [:awslib14      ; AWS icons
                      :azure         ; Azure icons
                      :gcp           ; Google Cloud icons
                      :cloudinsight  ; Cloud icons
                      :cloudogu      ; Cloud icons
                      :devicons      ; Development icons
                      :devicons2     ; More dev icons
                      :font-awesome-5 ; Font Awesome
                      :logos]}}      ; Tech logos
```

**Sprite Matching:**
- Overarch matches `:tech` values against sprite names
- Explicit `:sprite` on element references overrides

**Example:**
```clojure
{:el :container
 :id :app/database
 :tech ["PostgreSQL"]  ; Automatically finds PostgreSQL sprite
 ...}
```

### Custom Styles

Define custom styling:

```clojure
{:styles
 #{{:el :element-style
    :id :style/highlight
    :background-color "#FF0000"
    :font-color "#FFFFFF"
    :border-color "#000000"
    :border-width 2
    :shape :rounded-box}}}
```

Apply to elements via reference:

```clojure
{:ct [{:el :ref
       :id :banking/critical-service
       :style :style/highlight}]}
```

### PlantUML Skinparams

Low-level PlantUML styling:

```clojure
{:spec {:skinparams
        {:backgroundColor "#FEFEFE"
         :shadowing false
         :roundCorner 10}}}
```

## View Organization Best Practices

### Naming Conventions

**View IDs:**
- `:views/system-context`
- `:views/container-{system-name}`
- `:views/component-{container-name}`
- `:views/deployment-{environment}`

**Titles:**
- "System Context - {System Name}"
- "Container View - {System Name}"
- "Component View - {Container Name}"
- "{Environment} Deployment"

### View Composition Patterns

**Progressive Disclosure:**
1. Start with context view
2. Drill into container view
3. Detail with component views
4. Show deployment separately

**Audience-Specific Views:**
- Executive: Context and landscape views
- Developers: Container and component views
- Operations: Deployment and infrastructure views
- Security: Views filtered by `:security` tag

### File Organization

Organize views in separate files:

```
models/
  views/
    context-views.edn
    container-views.edn
    component-views.edn
    deployment-views.edn
    uml-views.edn
```

## View Generation Workflow

### CLI Rendering

```bash
# Render all views
overarch -m models/ -r all

# Render specific format
overarch -r plantuml

# Watch mode (auto-regenerate on file changes)
overarch -m models/ -r all --watch

# Render to specific directory
overarch -m models/ -r all -R output/diagrams/
```

### Output Formats

**PlantUML:**
- `.puml` files generated
- Convert to PNG/SVG/PDF with PlantUML

**GraphViz:**
- `.dot` files generated
- Convert with `dot` command

**Markdown:**
- `.md` files generated
- Ready for documentation systems

## Advanced View Techniques

### Multi-Level Views

Create views at different abstraction levels for the same area:

```clojure
[{:el :context-view
  :id :views/payment-context
  :title "Payment System - Context"}

 {:el :container-view
  :id :views/payment-containers
  :title "Payment System - Containers"}

 {:el :component-view
  :id :views/payment-processor-components
  :title "Payment Processor - Components"}]
```

### Filtered Views by Tag

Use tags to create focused views:

```clojure
[{:el :container-view
  :id :views/security-view
  :title "Security Architecture"
  :spec {:selection {:tags #{:security}}
         :include [:relations :related]}}

 {:el :container-view
  :id :views/data-flow
  :title "Data Processing Flow"
  :spec {:selection {:tags #{:data-processing}}
         :include [:relations :related]}}]
```

### Environment-Specific Deployment Views

```clojure
[{:el :deployment-view
  :id :views/dev-deployment
  :title "Development Environment"
  :spec {:selection {:namespace :infrastructure.dev}}}

 {:el :deployment-view
  :id :views/prod-deployment
  :title "Production Environment"
  :spec {:selection {:namespace :infrastructure.prod}}}]
```

### Subset Views for Documentation

Create focused views for documentation sections:

```clojure
{:el :container-view
 :id :views/authentication-flow
 :title "Authentication Flow"
 :spec {:include [:banking/web-app
                  :banking/api-server
                  :banking/auth-service
                  :banking/identity-provider]
        :include [:relations]}}
```

## Troubleshooting Views

### View Shows No Elements

**Causes:**
- Selection criteria too restrictive
- Namespace mismatch
- Element IDs incorrect

**Solutions:**
- Simplify selection: `{:selection {}}`
- Check element namespaces
- Verify IDs exist in model

### Too Many Elements

**Solutions:**
- Add selection criteria
- Use tags to filter
- Split into multiple focused views
- Use namespace filtering

### Layout Issues

**Solutions:**
- Adjust `:layout` direction
- Set `:node-separation` and `:rank-separation`
- Use `:direction` on relationships
- Try different GraphViz engines for concept views

### Missing Relationships

**Causes:**
- Relationship endpoints not in view
- Relationship type not included

**Solutions:**
- Add `:include [:relations]`
- Include related elements: `:include [:related]`
- Check relationship `:from` and `:to` IDs

## View Examples by Use Case

### Executive Overview

```clojure
{:el :system-landscape-view
 :id :views/enterprise-overview
 :title "Enterprise System Landscape"
 :spec {:selection {:el :system}
        :include [:relations]
        :layout :left-right}}
```

### Developer Onboarding

```clojure
[{:el :context-view
  :id :views/onboarding-context
  :title "System Overview"}

 {:el :container-view
  :id :views/onboarding-containers
  :title "Main Components"}

 {:el :component-view
  :id :views/onboarding-backend
  :title "Backend Structure"}]
```

### Security Review

```clojure
{:el :container-view
 :id :views/security-architecture
 :title "Security Architecture"
 :spec {:selection {:tags #{:security :authentication :encryption}}
        :include [:relations :related]}}
```

### API Documentation

```clojure
{:el :component-view
 :id :views/api-architecture
 :title "API Architecture"
 :spec {:selection {:namespace :api
                    :tags #{:public-api}}
        :include [:relations :related]}}
```
