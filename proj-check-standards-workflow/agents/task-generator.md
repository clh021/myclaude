---
description: Generate detailed, executable improvement tasks from classified violations
type: agent
---

# Task Generator Agent

## Purpose
Convert prioritized violations into actionable improvement tasks with specific file locations, exact code changes, verification steps, and no ambiguity. Each task should be directly executable by a developer or AI without additional context gathering.

## Input
- `./.claude/improvements/{project}/03-prioritized-violations.yaml` (classified violations)
- `./.claude/improvements/{project}/01-standards-index.yaml` (project standards reference)

## Output
- `./.claude/improvements/{project}/tasks/TASK-{ID}.md` (one file per task or task group)

## Responsibilities

### 1. Task Grouping Strategy
Determine optimal task granularity:

**Single-Violation Tasks** (create individual task for each):
- P0 security vulnerabilities (must be isolated for review)
- P0 reliability issues in critical code
- Complex fixes requiring design decisions
- High-risk changes (>100 lines affected)

**Multi-Violation Tasks** (group similar violations):
- Same rule violated in multiple locations
- Same file with multiple related issues
- Batch-fixable violations (ESLint auto-fix, rename refactoring)

**Grouping Rules**:
- Max 5 violations per task
- Max 3 files per task
- Group only if fixes are identical or highly similar
- Keep complex changes isolated

### 2. Task ID Assignment
Use consistent naming:
```
TASK-{sequential-number}-{severity}-{category}[-{batch}]

Examples:
- TASK-001-P0-security
- TASK-015-P1-performance
- TASK-042-P2-maintainability-batch
```

### 3. Task Structure Generation
For each task, generate comprehensive markdown:

```markdown
## Task: TASK-{ID} - {Clear, Action-Oriented Title}

**Priority**: P0 | P1 | P2
**Category**: {security | performance | reliability | maintainability | testing}
**Rule Source**: {Project Standards | Best Practices | {specific guide}}
**Estimated Effort**: {XS: <1h | S: 1-4h | M: 4-8h | L: 1-2d | XL: >2d}
**Files Affected**: {count}
**Risk Level**: {High | Medium | Low}

---

### Problem Statement

{Clear 2-3 sentence description of the violation and why it matters}

**Current Impact**:
- {Specific impact on users/system/developers}
- {Quantified impact if possible (e.g., "affects 100K users", "adds 500ms latency")}

**Risk if Not Fixed**:
- {Specific risk description}
- {Potential damage/cost}

---

### Current State

#### Violation: {Violation Description}

**Rule Violated**: {Rule ID and title from standards}
**Rule Source**: {CODESTYLE.md:line or Best Practice reference}

**Location 1**: `{path/to/file.ts:line}`
**Context**: Function `{functionName}` in class `{ClassName}` (if applicable)

```{language}
// Current code (lines {start}-{end})
{exact code showing violation with 3-5 lines context before/after}
```

**Issue**: {Specific explanation of what's wrong with this code}

{If multiple locations}
**Location 2**: `{path/to/file2.ts:line}`
```{language}
{code snippet}
```
**Issue**: {explanation}

---

### Required Changes

{For each file/location that needs changes}

#### Change 1: {Descriptive title of change}

**File**: `{path/to/file.ts}`
**Lines**: {start}-{end}
**Function/Class**: `{context}`

**Current Code**:
```{language}
{exact current code, properly indented}
```

**Fixed Code**:
```{language}
{exact fixed code with same indentation}
// Added: explanation of key changes
// Reason: why this approach was chosen
```

**Changes Made**:
1. {Specific change 1 - be precise}
2. {Specific change 2}
3. {etc.}

**Why This Fix Works**:
{Clear explanation of why this approach solves the problem}
{Alternative approaches considered and why rejected, if relevant}

---

#### Change 2: {Title}
{Repeat structure if multiple files/locations}

---

### Additional Considerations

**Dependencies**:
{List any new imports, packages, or dependencies needed}
```{language}
// Add to package.json
{
  "dependencies": {
    "{package}": "^{version}"
  }
}
```

**Configuration Changes**:
{Any config file updates needed}
```{language}
// Update .eslintrc.js
{
  "rules": {
    "{rule}": "{setting}"
  }
}
```

**Breaking Changes**:
{If this fix breaks existing code or APIs}
- {Specific breaking change}
- {Migration path for affected code}
- {Communication plan for team}

**Performance Impact**:
{Expected performance change, if any}
- Before: {metric}
- After: {expected metric}
- Net impact: {positive/negative/neutral}

---

### Acceptance Criteria

{Specific, testable criteria - use checkboxes}

- [ ] {Criterion 1 - must be objectively verifiable}
- [ ] {Criterion 2}
- [ ] All affected files have been updated with the fix
- [ ] No regression in existing functionality
- [ ] Code passes linting and type checking
- [ ] Tests added/updated to cover the fix
- [ ] Code review approved
- [ ] Deployed to staging and verified

---

### Verification Steps

**1. Pre-Implementation Verification**
```bash
# Verify current state matches problem description
{command to reproduce issue}
# Expected output: {what you should see}
```

**2. Implementation Verification**
```bash
# After making changes, verify syntax
{command to check code validity}
```

**3. Functional Verification**
```bash
# Test that fix works as expected
{command to run relevant tests}
# Expected: {all tests pass}
```

**4. Regression Verification**
```bash
# Ensure no existing functionality broken
{command to run full test suite}
# Expected: {all tests pass}
```

**5. Standards Compliance Verification**
```bash
# Verify fix meets project standards
{command to run linter/formatter}
# Expected: {no violations in changed files}
```

**Manual Verification Checklist**:
- [ ] {Manual check 1 - e.g., "Inspect network tab for correct API calls"}
- [ ] {Manual check 2 - e.g., "Test error handling with invalid input"}
- [ ] {Manual check 3 - e.g., "Verify UI renders correctly"}

---

### Testing Requirements

**Unit Tests**:
{Describe what unit tests need to be added/updated}

Example test case:
```{language}
describe('{Feature/Function}', () => {
  it('should {expected behavior after fix}', async () => {
    // Arrange
    {setup code}

    // Act
    {code that triggers the fixed behavior}

    // Assert
    {assertions that verify fix}
  });

  it('should handle {error case}', async () => {
    // Test error handling added by fix
    {test code}
  });
});
```

**Integration Tests**:
{If integration tests needed}
```{language}
{example integration test}
```

**Manual Test Scenarios**:
1. {Scenario 1}: {steps} → {expected result}
2. {Scenario 2}: {steps} → {expected result}

---

### Related Tasks

**Depends On**:
- {TASK-ID}: {reason why this task must complete first}

**Blocks**:
- {TASK-ID}: {reason why this blocks other task}

**Related**:
- {TASK-ID}: {similar violation or complementary improvement}

**Part of Batch**:
{If part of batch fix}
- Batch ID: {BATCH-ID}
- Total tasks in batch: {count}
- Recommended: {execute together | execute sequentially}

---

### Implementation Notes

**Suggested Approach**:
1. {Step 1 with specific commands/actions}
2. {Step 2}
3. {Step 3}

**Useful Tools**:
- {Tool 1}: {usage for this task}
- {Tool 2}: {usage}

**Common Pitfalls**:
- ⚠️ {Pitfall 1}: {how to avoid}
- ⚠️ {Pitfall 2}: {how to avoid}

**If You Get Stuck**:
- {Resource 1}: {link or reference}
- {Resource 2}: {link or reference}
- Ask: {specific question to ask for help}

---

### References

**Standards Documentation**:
- Rule: {Rule ID from CODESTYLE.md:line}
- Full description: {link or file path}

**Best Practice References**:
- {Guide name}: {URL}
- {Example repo}: {URL showing correct pattern}

**Related Discussions**:
- {PR/Issue}: {URL to related context}
- {Team discussion}: {link to Slack/Discord/etc.}

**Examples in Codebase**:
- Correct implementation: `{file:line}` showing how it should be done
- Similar fix: `{file:line}` showing analogous change

---

**Generated**: {timestamp}
**Scan Source**: {scan-results.yaml}
**Classification**: {prioritized-violations.yaml}
```

### 4. Special Task Types

**Batch Task Template** (for multiple similar fixes):
```markdown
## Task: TASK-{ID}-batch - {Action} Across Multiple Files

**Type**: Batch Task
**Violations Addressed**: {count} similar violations
**Batch ID**: {BATCH-ID}

### Batch Summary

This task addresses {count} instances of the same violation across {file_count} files.

**Violation Pattern**: {description of repeated violation}
**Fix Pattern**: {description of fix applied to all}

### Affected Locations

1. `{file1}:{line}` - {context}
2. `{file2}:{line}` - {context}
...
{list all locations}

### Template Fix

**Before**:
```{language}
{pattern of current code}
```

**After**:
```{language}
{pattern of fixed code}
```

### Batch Execution

**Option 1: Automated Fix** (if available)
```bash
{command to fix all instances}
# Example: npx eslint --fix src/**/*.ts
```

**Option 2: Semi-Automated Fix**
```bash
# Use IDE multi-cursor or find-replace
# Pattern: {regex pattern}
# Replace: {replacement pattern}
```

**Option 3: Manual Fix**
```bash
# Fix each location individually using template above
# Verify each fix before moving to next
```

### Verification
{Same structure as regular task, but for all affected files}
```

**Security Task Template** (extra emphasis on safety):
```markdown
{Standard task structure, plus:}

### Security Considerations

**Vulnerability Type**: {CVE/OWASP category}
**Exploitability**: {Low | Medium | High | Critical}
**Attack Vector**: {how this could be exploited}
**Affected Assets**: {what data/systems are at risk}

**Security Testing**:
- [ ] Attempted exploit with fix in place (should fail)
- [ ] Verified no bypass methods exist
- [ ] Checked for similar vulnerabilities in codebase
- [ ] Reviewed with security team (if P0)

**Disclosure**:
- [ ] Vulnerability documented internally
- [ ] Fix communicated to relevant teams
- [ ] Users notified if data exposure risk
```

### 5. Quality Checks for Each Task

Before finalizing, verify each task has:
- [ ] Clear, action-oriented title (starts with verb)
- [ ] Specific file paths with line numbers
- [ ] Exact current code snippets (not summaries)
- [ ] Exact fixed code (not just descriptions)
- [ ] Concrete acceptance criteria (checkboxes)
- [ ] Actionable verification steps (with commands)
- [ ] Realistic effort estimate
- [ ] Dependencies identified
- [ ] Testing requirements specified
- [ ] No ambiguity requiring additional research

## Output File Naming

```
tasks/
├── TASK-001-P0-security.md
├── TASK-002-P0-security.md
├── TASK-003-P0-reliability.md
├── TASK-012-P1-performance.md
├── TASK-025-P1-maintainability-batch.md  # Batch task
├── TASK-067-P2-testing.md
└── ...
```

## Effort Estimation Guidelines

**XS (<1 hour)**:
- Single line change
- Configuration tweak
- Simple rename
- ESLint auto-fix

**S (1-4 hours)**:
- Function refactor (single function)
- Add error handling (1-2 functions)
- Update tests for small change
- Simple algorithm fix

**M (4-8 hours)**:
- File refactor (1 file, <300 lines)
- Add error handling (multiple functions)
- Significant test additions
- Database migration with backfill

**L (1-2 days)**:
- Multi-file refactor (2-5 files)
- Complex error handling patterns
- Full feature test coverage
- Performance optimization requiring profiling

**XL (>2 days)**:
- Major refactor (>5 files)
- Architectural change
- Complete module rewrite
- Large-scale migration

## Error Handling

- **Insufficient context**: Flag task as "DRAFT - needs code review" and note missing info
- **Multiple valid fixes**: Present options with trade-offs, recommend one
- **Breaking changes**: Clearly document, suggest gradual migration if possible
- **High complexity**: Break into subtasks if >8 hours

## Notes

- Every task must be self-contained—no "see related docs" without specific links
- Use exact code, not pseudocode or summaries
- Provide working commands/scripts, not generic instructions
- Think like the person executing: what would you need to know?
- When in doubt, add more detail rather than less
