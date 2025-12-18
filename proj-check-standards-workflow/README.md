# Project Standards Checker Workflow

Check codebase against project standards and best practices, generate actionable improvement tasks.

## Overview

The Project Standards Checker Workflow is a companion to `/proj-gen-standards` that analyzes your codebase for standards violations and best practice gaps. Unlike the standards generation workflow, this workflow **checks compliance** and generates **executable improvement tasks** with specific fixes.

## Key Features

### 1. Dual-Source Checking
- **Project Standards**: Enforces rules from CODESTYLE.md (Required/Recommended/Optional)
- **Best Practices**: Checks against industry standards (filtered to avoid conflicts)
- **Intelligent Conflict Resolution**: Automatically skips best practice suggestions that contradict project standards

### 2. Task-Level Specificity
Every improvement includes:
- Exact file paths and line numbers
- Current code snippet showing violation
- Specific code fix (not just "improve X")
- Verification steps with commands
- Realistic effort estimates

### 3. Comprehensive Coverage
Scans for:
- **Security**: SQL injection, XSS, hardcoded secrets, CVEs, insecure crypto
- **Reliability**: Missing error handling, race conditions, resource leaks
- **Performance**: N+1 queries, missing indexes, inefficient algorithms
- **Maintainability**: High complexity, duplication, large files
- **Testing**: Coverage gaps, missing edge cases, brittle tests

### 4. Prioritized Improvements
- **P0 (Critical)**: Security vulnerabilities, crash risks, Required rule violations
- **P1 (Important)**: Recommended rules, performance issues, maintainability risks
- **P2 (Nice-to-have)**: Optional rules, code quality polish

### 5. Actionable Output
- Individual task files with complete implementation details
- Batch fix opportunities (ESLint auto-fix, rename refactoring, etc.)
- Quick wins (high impact, low effort)
- Phased roadmap with realistic timelines

## When to Use

**Ideal For**:
- After generating standards with `/proj-gen-standards`
- Before major releases (compliance audit)
- Quarterly code quality reviews
- Onboarding new team members (show current issues)
- Pre-refactoring assessment

**Not Ideal For**:
- Projects without standards documentation (run `/proj-gen-standards` first)
- Brand new projects with no code
- Quick fixes (use `/debug` or `/code` instead)

## Workflow Steps

1. **Environment Setup** (2-3 min)
   - Auto-detect standards documents
   - Detect project context
   - Create output directory

2. **Standards Parsing** (5-10 min)
   - Extract all rules from CODESTYLE.md
   - Identify exceptions and scope
   - Build rule index

3. **Comprehensive Scan** (15-20 min)
   - Check standards compliance
   - Check best practices (filtered)
   - Identify all violations with context

4. **Violation Classification** (10-15 min)
   - Assign severity (P0/P1/P2)
   - Calculate statistics
   - Identify hotspots and quick wins

5. **Task Generation** (15-20 min)
   - Generate detailed task files
   - Group similar violations
   - Create batch fix opportunities

6. **Requirements Document** (10-15 min)
   - Aggregate all tasks
   - Generate executive summary
   - Create implementation roadmap

## Output Structure

```
.claude/improvements/{project_name}/
├── 01-standards-index.yaml          # Parsed project standards
├── 02-scan-results.yaml             # Raw violations from scan
├── 03-prioritized-violations.yaml   # Classified and prioritized
├── IMPROVEMENTS.md                  # ✅ Master requirements document
└── tasks/
    ├── TASK-001-P0-security.md      # Detailed improvement tasks
    ├── TASK-002-P0-reliability.md
    ├── TASK-015-P1-performance.md
    ├── TASK-042-P2-maintainability-batch.md  # Batch tasks
    └── ...
```

## Usage Examples

### Example 1: Basic Standards Check

```bash
# After generating standards
/proj-gen-standards

# Check compliance
/proj-check-standards

# Output location
ls .claude/improvements/{project}/
```

**Results**:
- 47 violations found (5 P0, 18 P1, 24 P2)
- Estimated effort: 3 weeks
- Quick wins: 8 tasks (<4 hours each)
- Batch fixable: 28 tasks (ESLint auto-fix)

---

### Example 2: Focus on Critical Issues Only

```bash
# Check only P0 violations
/proj-check-standards --severity p0

# Review critical tasks
cat .claude/improvements/{project}/IMPROVEMENTS.md | grep -A 50 "## Critical"
```

**Results**:
- 5 critical security vulnerabilities
- 3 reliability risks in payment processing
- Estimated effort: 3 days

---

### Example 3: Quick Mode for CI/CD

```bash
# Fast scan for automated checks
/proj-check-standards --quick --severity p0

# Exit code 0 if no P0 issues, 1 if found
echo $?
```

**Use Case**: Block PRs with critical violations

---

## Integration with Other Workflows

### With `/proj-gen-standards`
```bash
# Step 1: Generate standards
/proj-gen-standards

# Step 2: Check compliance
/proj-check-standards

# Step 3: Implement improvements
# ... work through tasks ...

# Step 4: Re-check to verify
/proj-check-standards --quick
```

### With `/dev` Workflow
```bash
# Execute individual improvement tasks
/dev "Implement TASK-001: $(cat .claude/improvements/{project}/tasks/TASK-001-P0-security.md)"
```

### With `/proj-gen-task-prompts`
```bash
# Convert improvements to detailed execution prompts
/proj-gen-task-prompts "Implement all P0 fixes from .claude/improvements/{project}/IMPROVEMENTS.md"
```

### With CI/CD
```yaml
# .github/workflows/standards-check.yml
name: Standards Check
on: [pull_request]
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: /proj-check-standards --severity p0 --quick
```

## Agents

### standards-parser
- **Purpose**: Extract rules from CODESTYLE.md into structured format
- **Input**: Standards document(s)
- **Output**: `01-standards-index.yaml`
- **Responsibilities**: Rule extraction, exception patterns, tool config parsing

### violation-classifier
- **Purpose**: Prioritize violations by severity and impact
- **Input**: Scan results
- **Output**: `03-prioritized-violations.yaml`
- **Responsibilities**: Severity assignment, statistics, hotspot analysis, quick wins

### task-generator
- **Purpose**: Generate executable improvement tasks
- **Input**: Prioritized violations
- **Output**: `tasks/TASK-*.md` files
- **Responsibilities**: Task structuring, grouping, effort estimation, verification steps

### requirements-documenter
- **Purpose**: Create master IMPROVEMENTS.md document
- **Input**: All tasks and statistics
- **Output**: `IMPROVEMENTS.md`
- **Responsibilities**: Executive summary, roadmap, integration guidance

## Quality Standards

### Scan Quality
- [ ] All standards rules checked
- [ ] Best practices filtered for conflicts
- [ ] File paths and line numbers accurate
- [ ] No false positives (manual verification of samples)

### Task Quality
- [ ] Every task has specific file locations
- [ ] Fixed code provided (not just descriptions)
- [ ] Verification steps are actionable
- [ ] Effort estimates realistic (±50%)
- [ ] No ambiguity requiring research

### Document Quality
- [ ] Executive summary fits on 2 pages
- [ ] Roadmap is achievable
- [ ] Statistics accurate
- [ ] Quick wins genuinely quick
- [ ] Integration guidance specific

## Troubleshooting

### Issue: Standards document not found

**Solution**:
```bash
# Generate standards first
/proj-gen-standards

# Then run check
/proj-check-standards
```

---

### Issue: Too many violations (overwhelming)

**Solution**:
```bash
# Start with critical issues only
/proj-check-standards --severity p0

# After fixing P0, move to P1
/proj-check-standards --severity p1
```

---

### Issue: Scan missed violations

**Cause**: Code patterns not covered by scan logic

**Solution**:
1. Check `02-scan-results.yaml` for excluded patterns
2. Extend codeagent scan scope in Step 3
3. Add manual review for complex patterns
4. Report false negatives to improve workflow

---

### Issue: False positives (code is actually correct)

**Solution**:
1. Review violation details in task file
2. If legitimately correct, add exception to CODESTYLE.md
3. Re-run check to verify exception respected
4. Document why this pattern is allowed

---

## Best Practices

1. **Start with P0**: Always address critical issues first
2. **Batch similar fixes**: Use automated tools where possible (ESLint, Prettier)
3. **Track progress**: Use project management tool to monitor completion
4. **Re-check regularly**: Run quarterly to prevent regression
5. **Integrate into CI/CD**: Prevent new violations from being introduced
6. **Celebrate wins**: Share metrics improvements with team

## Example: Security Vulnerability Task

```markdown
## Task: TASK-001-P0-security - Fix SQL Injection in Login Endpoint

**Priority**: P0
**Category**: security
**Estimated Effort**: M (4-8h)
**Files Affected**: 1
**Risk Level**: High

### Problem Statement
User-controlled input is directly interpolated into SQL query, allowing
SQL injection attacks that could compromise the entire database.

**Current Impact**:
- Affects all 100K users attempting login
- Attacker can bypass authentication
- Attacker can dump entire user database

**Risk if Not Fixed**:
- Complete database compromise
- User data breach (PII, passwords)
- Legal/compliance violations (GDPR, etc.)

### Current State

**File**: `src/api/auth.ts:45`
**Function**: `loginUser`

```typescript
// Current code (lines 43-47)
async function loginUser(email: string, password: string) {
  const query = `SELECT * FROM users WHERE email = '${email}'`;
  const user = await db.query(query);
  return user;
}
```

**Issue**: String interpolation allows injection like:
```
email = "admin@example.com' OR '1'='1"
```

### Required Changes

**File**: `src/api/auth.ts`
**Lines**: 45-46

**Current**:
```typescript
const query = `SELECT * FROM users WHERE email = '${email}'`;
const user = await db.query(query);
```

**Fixed**:
```typescript
const query = `SELECT * FROM users WHERE email = $1`;
const user = await db.query(query, [email]);
// Changed: Use parameterized query ($1 placeholder)
// Reason: Database driver escapes parameters, preventing injection
```

**Changes Made**:
1. Replaced string interpolation with placeholder ($1)
2. Passed email as parameter array to db.query()
3. Database driver now handles escaping automatically

**Why This Fix Works**:
Parameterized queries separate SQL logic from data. The database driver
treats parameters as literal values, not executable SQL, making injection
impossible.

### Verification Steps

**1. Pre-Implementation Verification**:
```bash
# Verify vulnerability exists
curl -X POST http://localhost:3000/api/login \
  -d '{"email":"admin@example.com'\'' OR '\''1'\''='\''1","password":"any"}'
# Expected: Should succeed (vulnerability confirmed)
```

**2. Functional Verification**:
```bash
# After fix, injection should fail
curl -X POST http://localhost:3000/api/login \
  -d '{"email":"admin@example.com'\'' OR '\''1'\''='\''1","password":"any"}'
# Expected: 401 Unauthorized
```

**3. Security Test**:
```bash
# Run security scan
npm run security:scan
# Expected: No SQL injection vulnerabilities
```

**Manual Verification Checklist**:
- [ ] Normal login still works
- [ ] Injection attempts fail (tried 5 common payloads)
- [ ] Error messages don't leak information
```

## See Also

- [Project Standards Checker Guide](../../docs/PROJ-CHECK-STANDARDS-WORKFLOW.md)
- [Standards Parser Agent](./agents/standards-parser.md)
- [Violation Classifier Agent](./agents/violation-classifier.md)
- [Task Generator Agent](./agents/task-generator.md)
- [Requirements Documenter Agent](./agents/requirements-documenter.md)

---

**Module Version**: 1.0
**Last Updated**: 2025-12-18
**Status**: Ready for Production Use
