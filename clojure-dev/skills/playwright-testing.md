---
name: playwright-testing
description: Use for UI testing with Playwright MCP. Verify components in isolation and features end-to-end.
requires:
  tools: []
  mcps: [playwright]
  skills: [zero-components]
skip_when:
  - No UI components in the feature
  - Playwright MCP not installed
---

# Playwright Testing

UI testing using Playwright MCP for component and feature verification.

## Prerequisites

Playwright MCP must be installed. If not available, notify user:

"Playwright MCP is required for UI testing but not installed. Please install it to enable automated UI verification."

## When to Use

### Component Verification
After implementing a component in the library (`vault/components/`), verify it works:
- Renders correctly
- States work (hover, active, disabled)
- Interactions work (click, input)

### Feature Verification
After integrating components into a feature:
- Full user flow works
- Components interact correctly
- No visual regressions

## Component Testing

### 1. Start the component demo server

Ensure the component demo is accessible (e.g., `vault/components/demos/button.html`).

### 2. Write Playwright test

Test each state and interaction:

```javascript
// Test default rendering
await page.goto('/demos/button.html');
await expect(page.locator('.button')).toBeVisible();

// Test hover state
await page.locator('.button').hover();
await expect(page.locator('.button')).toHaveClass(/hover/);

// Test click
await page.locator('.button').click();
await expect(page.locator('.button')).toHaveClass(/active/);

// Test disabled
await expect(page.locator('.button.disabled')).toBeDisabled();
```

### 3. Run and verify

Run tests via Playwright MCP. All should pass before proceeding.

## Feature Testing

### 1. Identify user flows

List the key user journeys:
- User can log in
- User can create item
- User can edit item
- User can delete item

### 2. Write end-to-end tests

```javascript
// Login flow
await page.goto('/login');
await page.fill('[name="email"]', 'test@example.com');
await page.fill('[name="password"]', 'password');
await page.click('button[type="submit"]');
await expect(page).toHaveURL('/dashboard');
```

### 3. Run full suite

All feature tests must pass before asking user for review.

## Checkpoints

| When | What to Test | Human Review? |
|------|--------------|---------------|
| Component done | Component in isolation | No - automated |
| Feature done | End-to-end flows | Yes - after tests pass |

## Troubleshooting

### MCP not available
Notify user and suggest installation.

### Tests failing
1. Check component renders correctly
2. Check selectors are correct
3. Check for timing issues (add waits)
4. Check demo server is running

### Visual differences
1. Update mockup if design changed
2. Get approval for visual changes
3. Update tests to match

## Success Criteria

- [ ] Playwright MCP available
- [ ] Component tests passing
- [ ] Feature tests passing
- [ ] User reviewed completed feature
