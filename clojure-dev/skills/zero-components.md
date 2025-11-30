---
name: zero-components
description: Use when user wants to create web components with Zero, document UI components, or set up component demos with hot reload.
requires:
  tools: [shadow-cljs]
  skills: [init-obsidian-vault]
  deps: [pcp/zero, thheller/shadow-cljs]
---

# Zero Web Components

Build and document web components using Zero (ClojureScript) with live demos.

## Prerequisites

```bash
command -v npx >/dev/null || { echo "Install Node.js"; exit 1; }
grep -q "shadow-cljs" package.json 2>/dev/null || echo "Add shadow-cljs - see below"
grep -q "pcp/zero" deps.edn 2>/dev/null || echo "Add Zero dependency - see below"
```

Add if missing:

```clojure
;; deps.edn
{:deps {pcp/zero {:mvn/version "0.1.0"}}}
```

```json
// package.json
{"devDependencies": {"shadow-cljs": "^2.28.0"}}
```

## Directory Structure

```
src/
  components/
    button.cljs           # Zero component definition
    card.cljs

vault/components/
  index.md                # Component catalog
  button.md               # Documentation
  card.md
  demos/
    button.html           # Live demo (imports compiled JS)
    card.html

resources/public/
  js/components.js        # shadow-cljs output
```

## Workflow

### 1. Configure shadow-cljs

`shadow-cljs.edn`:
```clojure
{:source-paths ["src"]
 :dependencies [[pcp/zero "0.1.0"]]
 :builds
 {:components
  {:target :browser
   :output-dir "resources/public/js"
   :asset-path "/js"
   :modules {:components {:entries [components.core]}}
   :devtools {:watch-dir "resources/public"}}}}
```

### 2. Create a Component

`src/components/button.cljs`:
```clojure
(ns components.button
  (:require [zero.core :as z]
            [zero.component :refer [component]]))

(component ::button
  {:props #{:variant :disabled :loading}
   :view
   (fn [{:keys [variant disabled loading]}]
     [:button
      {:class [(or variant "default")
               (when disabled "disabled")
               (when loading "loading")]
       :disabled disabled}
      (if loading
        [:span.spinner]
        [:slot])])})
```

`src/components/core.cljs`:
```clojure
(ns components.core
  (:require [components.button]))

(defn init []
  (js/console.log "Components loaded"))
```

### 3. Create Documentation

`vault/components/button.md`:
```markdown
# Button Component

**Source:** `src/components/button.cljs`
**Tag:** `<z-button>`

## Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `variant` | string | "default" | "default", "primary", "danger" |
| `disabled` | boolean | false | Disables interaction |
| `loading` | boolean | false | Shows spinner, disables click |

## Demo

Open [[demos/button.html]] in browser (requires dev server running)

## Usage

\`\`\`html
<z-button variant="primary">Submit</z-button>
<z-button disabled>Disabled</z-button>
<z-button loading>Processing...</z-button>
\`\`\`

## Related

- [[card]] - Card component for content containers
```

### 4. Create Demo Page

`vault/components/demos/button.html`:
```html
<!DOCTYPE html>
<html>
<head>
  <title>Button - Component Demo</title>
  <script src="/js/components.js"></script>
  <style>
    body { font-family: system-ui; padding: 2rem; }
    section { margin: 2rem 0; padding: 1rem; border: 1px solid #e0e0e0; border-radius: 4px; }
    h1 { border-bottom: 2px solid #333; padding-bottom: 0.5rem; }
    h2 { margin-top: 0; color: #666; }
    .demo-row { display: flex; gap: 1rem; align-items: center; }
  </style>
</head>
<body>
  <h1>Button Component</h1>
  <p><a href="../button.md">← Back to documentation</a></p>

  <section>
    <h2>Variants</h2>
    <div class="demo-row">
      <z-button variant="default">Default</z-button>
      <z-button variant="primary">Primary</z-button>
      <z-button variant="danger">Danger</z-button>
    </div>
  </section>

  <section>
    <h2>States</h2>
    <div class="demo-row">
      <z-button>Normal</z-button>
      <z-button disabled>Disabled</z-button>
      <z-button loading>Loading</z-button>
    </div>
  </section>

  <section>
    <h2>Combined</h2>
    <div class="demo-row">
      <z-button variant="primary" loading>Submitting...</z-button>
      <z-button variant="danger" disabled>Cannot Delete</z-button>
    </div>
  </section>
</body>
</html>
```

### 5. Run Development

**Terminal 1:** Start your Clojure backend (serves static files including demos)
```bash
clj -M:dev
```

**Terminal 2:** Start shadow-cljs watch
```bash
npx shadow-cljs watch components
```

Changes to `.cljs` files hot-reload automatically in the browser.

### 6. Update Component Catalog

`vault/components/index.md`:
```markdown
# Component Catalog

UI components built with Zero (ClojureScript web components).

## Development

1. Start backend: `clj -M:dev`
2. Start shadow-cljs: `npx shadow-cljs watch components`
3. Open demo pages in browser

## Components

| Component | Tag | Description |
|-----------|-----|-------------|
| [[button]] | `<z-button>` | Interactive button with variants |
| [[card]] | `<z-card>` | Content container with header/body |

## Adding a Component

1. Create `src/components/newcomponent.cljs`
2. Add require to `src/components/core.cljs`
3. Create `vault/components/newcomponent.md` (documentation)
4. Create `vault/components/demos/newcomponent.html` (live demo)
```

## Component Template

When creating a new component:

**1. Component file** (`src/components/{name}.cljs`):
```clojure
(ns components.{name}
  (:require [zero.core :as z]
            [zero.component :refer [component]]))

(component ::{name}
  {:props #{:prop1 :prop2}
   :view
   (fn [{:keys [prop1 prop2]}]
     [:div.{name}
      [:slot]])})
```

**2. Documentation** (`vault/components/{name}.md`):
```markdown
# {Name} Component

**Source:** `src/components/{name}.cljs`
**Tag:** `<z-{name}>`

## Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|

## Demo

Open [[demos/{name}.html]] in browser

## Usage

\`\`\`html
<z-{name}>Content</z-{name}>
\`\`\`
```

**3. Demo page** (`vault/components/demos/{name}.html`):
```html
<!DOCTYPE html>
<html>
<head>
  <title>{Name} - Component Demo</title>
  <script src="/js/components.js"></script>
</head>
<body>
  <h1>{Name} Component</h1>

  <section>
    <h2>Default</h2>
    <z-{name}>Content</z-{name}>
  </section>
</body>
</html>
```

## Component Library Workflow

Components MUST be created in the library before integration:

### 1. Mockup First

Before implementing, create and get approval for mockup:
- Use `ui-mockups` skill
- Create `mockups/<component>.html`
- Get user approval

### 2. Create in Library

Create component in `vault/components/`:
- Component code
- Documentation
- Demo in `vault/components/demos/`

### 3. Verify with Playwright

Run Playwright tests on the isolated component:
- All states render correctly
- Interactions work
- No console errors

### 4. Then Integrate

Only after Playwright verification:
- Import into feature
- Connect to application state
- Test in context

## Linking Requirements

When documenting a component:
- Link to approved mockup: `Design: [[mockups/component-name]]`
- Link to related components: `See also [[components/related]]`
- Link to features using this component: `Used in [[features/feature-name]]`

Example component doc header:

```markdown
# Button Component

Design: [[mockups/button]]
See also: [[components/icon]], [[components/spinner]]
Used in: [[features/login]], [[features/checkout]]
```

## Success Criteria

- [ ] `npx shadow-cljs compile components` succeeds
- [ ] Component renders in demo HTML
- [ ] Hot reload updates component without page refresh
- [ ] Documentation in vault links to demo

## Related Skills

- `clojure-repl` - Interactive development
- `init-obsidian-vault` - Vault structure setup
