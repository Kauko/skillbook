---
name: error-recovery
description: Use IMMEDIATELY when any verification tool fails - TLC counterexample, overarch --check error, threagile analysis failure, linting errors, or test failures. Do not attempt fixes without this skill.
requires:
  tools: []
  skills: []
skip_when:
  - Tool succeeded with no errors
  - User explicitly asks to ignore the failure
  - Failure is in a dependency outside project control
---

# Error Recovery

Systematic approach to recovering from tool failures. **Do not guess at fixes.** Understand first, then act.

## Trigger Conditions

Use this skill when ANY of these occur:
- TLC/Recife reports invariant violation or counterexample
- `overarch --check` fails validation
- `threagile analyze` reports errors (not risks - risks are expected)
- `clj-kondo`, `splint`, or `cljfmt` report violations
- Tests fail
- Build/compilation errors

## Recovery Framework

### Phase 1: Understand the Failure

```
┌─────────────────────────────────────────────────────────┐
│ STOP. Do not attempt fixes until you complete Phase 1. │
└─────────────────────────────────────────────────────────┘
```

**1.1 Capture the exact error:**
```bash
# Re-run the failing command, capture full output
[failing-command] 2>&1 | tee /tmp/claude/error-output.txt
```

**1.2 Classify the failure type:**

| Type | Characteristics | Recovery Path |
|------|-----------------|---------------|
| **Syntax** | Parse error, invalid format | Fix syntax at reported location |
| **Semantic** | Type mismatch, missing reference | Trace dependency chain |
| **Invariant** | Property violation, counterexample | Analyze state trace |
| **Configuration** | Missing file, wrong path | Check prerequisites |
| **Environment** | Tool not found, permission denied | Install/configure tool |

**1.3 Locate the root cause:**

For **counterexample traces** (TLC/Recife):
```
State 0 (Initial): [variables]
State 1: [action taken] → [new state]
State 2: [action taken] → [VIOLATION]
```
- Read each state transition
- Identify which action led to violation
- Check if the action's precondition was too weak

For **validation errors** (overarch, threagile):
```bash
# Get structured error if available
[tool] --format json 2>&1 | jq '.errors'
```

For **linting errors**:
```bash
# Get machine-readable output
clj-kondo --lint src --config '{:output {:format :edn}}'
splint --format edn src
```

### Phase 2: Categorize and Plan

**2.1 Is this a real problem or a false positive?**

| Signal | Likely Real Problem | Likely False Positive |
|--------|--------------------|-----------------------|
| Multiple related errors | Yes | No |
| Error in business logic | Yes | No |
| Error only in generated code | No | Yes |
| Tool version mismatch mentioned | No | Yes |

**2.2 Determine fix scope:**

```
Single file → Fix directly
Multiple files, same pattern → Fix one, verify, then fix rest
Architectural issue → May need design change, consult user
```

**2.3 Create fix plan:**

Before making ANY changes, write out:
1. What the error says
2. What you believe the root cause is
3. What change you will make
4. How you will verify the fix worked

### Phase 3: Apply Fix

**3.1 Make minimal change:**
- Fix ONLY what's broken
- Don't refactor while fixing
- Don't "improve" adjacent code

**3.2 For counterexample fixes:**

```clojure
;; If invariant violated, either:

;; Option A: Strengthen precondition (prevent bad state)
(r/defaction transfer [amount]
  (r/and (>= balance-a amount)  ; existing
         (> amount 0)            ; ADD: prevent zero/negative
         ...))

;; Option B: Weaken invariant (allow state that's actually OK)
(r/definvariant money-conserved
  (= (+ balance-a balance-b) initial-total))  ; was: hardcoded 100
```

**3.3 For validation errors:**

```bash
# Fix the specific element mentioned
# Example: overarch complains about missing :from in relationship
# → Add the missing :from field, don't restructure the model
```

### Phase 4: Verify Fix

**4.1 Re-run the exact command that failed:**
```bash
[same-failing-command]
```

**4.2 Check for regression:**
```bash
# Run broader verification
[tool] --check [full-project-scope]
```

**4.3 Verification checklist:**

```bash
# Machine-readable verification script
verify_fix() {
  local tool="$1"
  local expected_exit=0

  $tool
  local actual_exit=$?

  if [ $actual_exit -eq $expected_exit ]; then
    echo "PASS: $tool"
    return 0
  else
    echo "FAIL: $tool (exit $actual_exit, expected $expected_exit)"
    return 1
  fi
}
```

## Tool-Specific Recovery

### TLC/Recife Counterexamples

```
Error: Invariant money-conserved violated.
State 3: balance-a = -10, balance-b = 110
```

**Recovery steps:**
1. Print the full trace: identify which action caused the negative balance
2. Check action guard: was `(>= balance-a amount)` present?
3. Check for race condition: could two actions run concurrently?
4. Fix: strengthen guard or add mutex

### Overarch Validation

```
Error: Relationship :user-webapp references unknown element :webapp
```

**Recovery steps:**
1. Check if `:webapp` exists in model.edn
2. Check spelling (`:webapp` vs `:web-app`)
3. Check if element is in correct file (model.edn vs views.edn)
4. Fix: add missing element or correct reference

### Threagile Analysis

```
Error: Technical asset 'api-server' has no trust boundary
```

**Recovery steps:**
1. Review trust_boundaries section
2. Add asset to appropriate boundary's `technical_assets_inside`
3. If no boundary fits, create new boundary

### Linting Errors

```
src/app/core.clj:42: error: Unresolved symbol: proces-data
```

**Recovery steps:**
1. Check if typo (`proces` vs `process`)
2. Check if namespace required
3. Check if function exists in target namespace
4. Fix: correct typo or add require

## Anti-Patterns

**DO NOT:**
- Disable the check that's failing
- Add blanket exceptions to suppress errors
- "Fix" by deleting the failing code
- Make multiple unrelated changes while fixing
- Assume you know the fix without reading the error

**DO:**
- Read the full error message
- Understand why the check exists
- Make minimal targeted fix
- Verify the fix resolved the issue
- Document if the fix reveals a design problem

## Success Criteria

```bash
#!/bin/bash
# Machine-readable success verification

TOOL="$1"
ORIGINAL_ERROR="$2"

# Run tool
OUTPUT=$($TOOL 2>&1)
EXIT_CODE=$?

# Verify
if [ $EXIT_CODE -eq 0 ]; then
  echo "exit_code:0"
  echo "status:success"
  echo "original_error_resolved:true"
else
  echo "exit_code:$EXIT_CODE"
  echo "status:failure"
  echo "remaining_errors:$(echo "$OUTPUT" | grep -c 'error')"
fi
```

- [ ] `$FAILING_TOOL` now exits with code 0
- [ ] Original error message no longer appears in output
- [ ] No new errors introduced
- [ ] Related tests still pass

## Related Skills

- `recife-modeling` - TLA+ formal verification
- `overarch-modeling` - Architecture validation
- `threagile-analysis` - Threat modeling
- `quality-check` - Linting and formatting
