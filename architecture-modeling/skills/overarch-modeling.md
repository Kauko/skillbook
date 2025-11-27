---
description: Use when creating or updating the architecture model. Creates/maintains architecture in Overarch EDN format.
---

# Overarch Modeling Skill

Use this skill when creating or updating the architecture model in Overarch EDN format.

## Prerequisites Check

Before proceeding, verify Overarch CLI is installed:

```bash
overarch --version
```

If not installed, guide the user to install:

**macOS (Homebrew):**
```bash
brew install overarch/brew/overarch
```

**Linux/macOS (manual):**
```bash
# Download latest release from https://github.com/soulspace-org/overarch/releases
# Extract and add to PATH
```

**Verify installation:**
```bash
overarch --version
```

## Core Workflow

### 1. Initialize or Update Architecture Model

Create or update the central architecture model at `vault/architecture/model.edn`.

### 2. Model Structure

The Overarch EDN format follows this structure:

```clojure
{:model
 {:nodes
  [{:el :system
    :id :my-system
    :name "My System"
    :desc "Description of the system"
    :tech ["Technology Stack"]}

   {:el :container
    :id :web-app
    :name "Web Application"
    :desc "Frontend web application"
    :tech ["React" "TypeScript"]
    :ct "SPA"}

   {:el :container
    :id :api-server
    :name "API Server"
    :desc "Backend REST API"
    :tech ["Node.js" "Express"]
    :ct "Service"}

   {:el :component
    :id :auth-component
    :name "Authentication Component"
    :desc "Handles user authentication"
    :tech ["JWT" "OAuth2"]
    :ct :authentication}

   {:el :person
    :id :end-user
    :name "End User"
    :desc "System user"}

   {:el :database
    :id :main-db
    :name "Main Database"
    :desc "Primary data store"
    :tech ["PostgreSQL"]
    :ct "RDBMS"}]

  :relations
  [{:el :rel
    :id :user-web-rel
    :from :end-user
    :to :web-app
    :name "Uses"
    :desc "Interacts with the application"}

   {:el :rel
    :id :web-api-rel
    :from :web-app
    :to :api-server
    :name "Calls"
    :tech ["REST" "HTTPS"]}

   {:el :rel
    :id :api-db-rel
    :from :api-server
    :to :main-db
    :name "Reads/Writes"
    :tech ["SQL"]}]}

 :views
 {:context-views
  [{:el :context-view
    :id :system-context
    :title "System Context"
    :spec {:include [:end-user :my-system :main-db]}}]

  :container-views
  [{:el :container-view
    :id :container-view
    :title "Container View"
    :spec {:include [:web-app :api-server :main-db]}}]

  :component-views
  [{:el :component-view
    :id :component-view
    :title "Component View"
    :spec {:include [:auth-component]}}]}}
```

### 3. Element Types Reference

**Structural Elements:**
- `:system` - Software system
- `:container` - Deployable/executable unit (app, service, database)
- `:component` - Grouping of related functionality
- `:person` - Human user or actor
- `:enterprise-boundary` - Organizational boundary
- `:context-boundary` - System boundary

**Technical Elements:**
- `:database` - Data store
- `:queue` - Message queue
- `:cache` - Caching layer

**Relationships:**
- `:rel` - General relationship
- `:request` - Synchronous request
- `:response` - Response to request
- `:publish` - Async message publish
- `:subscribe` - Async message subscription
- `:dataflow` - Data flow

**Security Elements:**
- `:trust-boundary` - Security/trust boundary
- `:threat` - Security threat
- `:control` - Security control

### 4. Common Patterns

**Microservices System:**
```clojure
{:el :system
 :id :microservices-system
 :name "Microservices Platform"
 :containers
 [{:el :container :id :service-a :name "Service A" :tech ["Java" "Spring Boot"]}
  {:el :container :id :service-b :name "Service B" :tech ["Go"]}
  {:el :container :id :api-gateway :name "API Gateway" :tech ["Kong"]}
  {:el :container :id :service-mesh :name "Service Mesh" :tech ["Istio"]}]}
```

**Data Architecture:**
```clojure
{:el :container
 :id :data-pipeline
 :name "Data Pipeline"
 :tech ["Apache Kafka" "Apache Spark"]
 :data-assets
 [{:el :data-asset
   :id :user-data
   :name "User Data"
   :classification "PII"
   :sensitivity "high"}]}
```

**Security Boundaries:**
```clojure
{:el :trust-boundary
 :id :internal-network
 :name "Internal Network"
 :desc "Trusted internal zone"
 :contains [:api-server :main-db]}

{:el :trust-boundary
 :id :dmz
 :name "DMZ"
 :desc "Demilitarized zone"
 :contains [:web-app :api-gateway]}
```

### 5. Starter Template

When creating a new model, use this starter template:

```clojure
{:model
 {:nodes
  ;; External Actors
  [{:el :person
    :id :end-user
    :name "End User"
    :desc "System user"}

   ;; Systems
   {:el :system
    :id :main-system
    :name "Main System"
    :desc "Primary software system"}

   ;; Containers (within main-system)
   {:el :container
    :id :frontend
    :name "Frontend Application"
    :desc "User interface"
    :tech ["Technology"]
    :ct "Web Application"}

   {:el :container
    :id :backend
    :name "Backend Service"
    :desc "Business logic and API"
    :tech ["Technology"]
    :ct "Service"}

   {:el :database
    :id :database
    :name "Database"
    :desc "Data persistence"
    :tech ["Database Technology"]
    :ct "Database"}]

  ;; Relationships
  :relations
  [{:el :rel
    :id :user-frontend-rel
    :from :end-user
    :to :frontend
    :name "Uses"
    :desc "Accesses via web browser"}

   {:el :rel
    :id :frontend-backend-rel
    :from :frontend
    :to :backend
    :name "Calls"
    :tech ["REST API" "HTTPS"]}

   {:el :rel
    :id :backend-database-rel
    :from :backend
    :to :database
    :name "Reads/Writes"
    :tech ["SQL"]}]}

 ;; Views define what to visualize
 :views
 {:context-views
  [{:el :context-view
    :id :system-context
    :title "System Context Diagram"
    :spec {:include [:end-user :main-system :database]}}]

  :container-views
  [{:el :container-view
    :id :container-view
    :title "Container Diagram"
    :spec {:include [:frontend :backend :database]}}]}}
```

## Validation Steps

### 1. EDN Syntax Validation

After editing, validate EDN syntax:

```bash
# Using Clojure CLI (if available)
clojure -M -e "(require '[clojure.edn :as edn]) (edn/read-string (slurp \"vault/architecture/model.edn\"))"
```

Or check manually:
- Balanced brackets: `{`, `[`, `(`
- Keywords start with `:`
- Strings in quotes
- Commas are whitespace (optional)

### 2. Model Completeness Check

Verify:
- All `:id` values are unique
- All `:from` and `:to` in relations reference existing `:id`
- Required fields present: `:el`, `:id`, `:name`
- Technology stacks specified where relevant

### 3. Export Validation

Test model can be processed:

```bash
overarch -m vault/architecture/model.edn -r export -f json
```

## Educational Guidance

### C4 Model Levels

**Level 1 - System Context:**
- Shows the system in its environment
- Includes users and external systems
- Highest level of abstraction

**Level 2 - Container:**
- Shows major containers (apps, services, databases)
- Each container is separately deployable/executable
- Shows technology choices

**Level 3 - Component:**
- Shows components within a container
- Groups of related functionality
- Internal structure

**Level 4 - Code:**
- UML class diagrams, ER diagrams
- Usually not maintained in Overarch

### Best Practices

1. **Start Simple**: Begin with context diagram, add detail incrementally
2. **Consistent Naming**: Use kebab-case for IDs, Title Case for names
3. **Meaningful Relationships**: Name relationships clearly ("Uses", "Calls", "Reads from")
4. **Technology Tags**: Always specify technology stack
5. **Documentation**: Add `:desc` to explain purpose and responsibilities
6. **Boundaries**: Use trust boundaries to show security zones
7. **Data Flow**: Mark sensitive data flows explicitly

## Warning System

After model updates, check if derived artifacts need regeneration:

```bash
# Check if model.edn is newer than derived files
if [ vault/architecture/model.edn -nt vault/architecture/diagrams.md ]; then
  echo "WARNING: Model has changed but diagrams not regenerated!"
  echo "Run sync-architecture skill to update derived artifacts."
fi
```

## Integration Points

This skill integrates with:
- **structurizr-diagrams** skill: Generates visual diagrams from model
- **sync-architecture** skill: Regenerates all derived formats

## Common Operations

### Add New System

```clojure
{:el :system
 :id :new-system
 :name "New System"
 :desc "Description"
 :tech ["Tech Stack"]}
```

### Add New Container

```clojure
{:el :container
 :id :new-service
 :name "New Service"
 :desc "Service description"
 :tech ["Language" "Framework"]
 :ct "Service"
 :external false}
```

### Add Relationship

```clojure
{:el :rel
 :id :service-a-to-b
 :from :service-a
 :to :service-b
 :name "Synchronizes data with"
 :tech ["gRPC" "Protocol Buffers"]
 :desc "Real-time data synchronization"}
```

### Define Trust Boundary

```clojure
{:el :trust-boundary
 :id :secure-zone
 :name "Secure Zone"
 :desc "High security internal network"
 :contains [:sensitive-service :secure-db]}
```

### Add Data Asset

```clojure
{:el :data-asset
 :id :customer-data
 :name "Customer Data"
 :classification "PII"
 :sensitivity "high"
 :stored-in [:main-db :backup-db]}
```

## Output

After creating or updating the model:

1. Confirm model saved at `vault/architecture/model.edn`
2. Validate EDN syntax
3. Suggest next steps:
   - Run **structurizr-diagrams** skill to generate visualizations
   - Run **sync-architecture** skill to regenerate all derived formats
   - Review and commit changes to version control

## Example Usage

**User Request:** "Create an architecture model for a web application with React frontend, Node.js API, and PostgreSQL database"

**Assistant Actions:**
1. Check Overarch CLI installation
2. Create `vault/architecture/model.edn` with appropriate structure
3. Define person (user), system, containers (frontend, backend, database)
4. Define relationships between components
5. Create context and container views
6. Validate EDN syntax
7. Suggest running structurizr-diagrams to visualize

---

Remember: The Overarch model is the single source of truth. All diagrams and documentation are derived from this central model.
