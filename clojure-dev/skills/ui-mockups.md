---
name: ui-mockups
description: Use before implementing any UI component. Create HTML/CSS mockup, get human approval, then proceed to implementation.
requires:
  tools: []
  skills: []
---

# UI Mockups

Create visual mockups before writing component code. Human approves design before implementation begins.

## When to Use

Before implementing ANY UI component:
- New component
- Significant visual change to existing component
- New page or view

## Workflow

### 1. Create Mockup Directory

```bash
mkdir -p mockups
```

### 2. Create HTML/CSS Mockup

Create `mockups/<component-name>.html`:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Mockup: Component Name</title>
  <style>
    /* Component styles */
    .component {
      /* styles here */
    }
  </style>
</head>
<body>
  <h1>Component: Name</h1>

  <h2>Default State</h2>
  <div class="component">
    <!-- default state -->
  </div>

  <h2>Hover State</h2>
  <div class="component hover">
    <!-- hover state -->
  </div>

  <h2>Active State</h2>
  <div class="component active">
    <!-- active state -->
  </div>

  <h2>Disabled State</h2>
  <div class="component disabled">
    <!-- disabled state -->
  </div>
</body>
</html>
```

### 3. Show All States

Include mockups for:
- Default state
- Hover state
- Active/pressed state
- Disabled state
- Error state (if applicable)
- Loading state (if applicable)
- Empty state (if applicable)

### 4. Ask for Approval

Ask user to review:

"Please review the mockup at `mockups/<component-name>.html`. Does the visual design look correct?"

### 5. Wait for Approval

Do NOT proceed to implementation until user approves.

If user requests changes:
1. Update the mockup
2. Ask for review again
3. Repeat until approved

### 6. Proceed to Implementation

After approval:
1. Write failing test (TDD)
2. Implement component (zero-components)
3. Link component doc to mockup: `Design: [[mockups/component-name]]`

## Mockup Conventions

### File naming
- `mockups/button.html`
- `mockups/card.html`
- `mockups/nav-header.html`

### Include context
- Show component in realistic context
- Use real-ish content, not lorem ipsum
- Show responsive behavior if relevant

### Document decisions
Add comments explaining design choices:

```html
<!-- Using 8px padding for consistency with design system -->
<!-- Color #3B82F6 from brand palette -->
```

## Linking Requirements

After component is implemented:
- Component doc links to mockup: `Design: [[mockups/button]]`
- Mockup can link to component: `Implementation: [[components/button]]`

## Success Criteria

- [ ] Mockup created with all relevant states
- [ ] User reviewed and approved design
- [ ] Mockup linked from component documentation
