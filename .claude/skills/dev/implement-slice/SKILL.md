---
name: implement-slice
description: Implements a single slice with Signal validation and unit tests. Use when a user wants to implement one specific slice, mentions "implement slice X.Y", or is working incrementally through slices.
---

# Implement Slice

Executes a single slice with Signal validation and unit tests.

## Workflow

1. **Read slice** - Load the entire slice file into context
2. **Create TODO list** - Three items:
   - `{slice-name} - Implement`
   - `{slice-name} - Signal Validation`
   - `{slice-name} - Unit Tests`
3. **Implement** - Mark in_progress, implement all code specified in the slice
4. **Signal validation** - Check the Signal section:
   - If Signal Skill specified: invoke `skill: "[signal-name]"`, wait for output
   - Compare against Expected Behavior
   - Fix issues and re-invoke until validated
   - If Signal Skill is "None": skip to unit tests
5. **Unit tests** - Create/update unit tests for the implemented functionality
6. **Complete** - Mark all TODOs complete, stop

## Signal Processing

Each slice includes a Signal section after the Objective:

```markdown
## Signal

**Signal Skill:** [signal-skill-name | None]

**Expected Behavior:**
- What should succeed when correctly implemented
```

### Signal Workflow

1. After implementing slice code, check the Signal section
2. If Signal Skill is specified:
   - Invoke the signal: `skill: "[signal-name]"`
   - Wait for signal output
   - Follow the guidance from Signal
3. If signal indicates success: Continue to unit tests
4. If signal indicates failure:
   - Review signal output to identify specific issue
   - Fix the implementation
   - Re-invoke signal until success
5. If Signal Skill is "None": Skip to unit tests

## TODO Structure

```
[ ] 1.3-user-auth - Implement
[ ] 1.3-user-auth - Signal Validation
[ ] 1.3-user-auth - Unit Tests
```

## Guidelines

**DON'T:**
- Implement beyond the slice scope
- Proceed with failing signal validation
- Skip signal validation if specified
- Implement dependent slices (that's for implement-mainspec)