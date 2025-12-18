# Project Standards Checker Workflow Guide

## Overview

The Project Standards Checker Workflow analyzes your codebase for standards violations and best practice gaps, generating **executable improvement tasks** with specific fixes. This workflow is designed as a companion to `/proj-gen-standards`, checking compliance against your established standards while also identifying opportunities based on industry best practices.

## Key Differences from Other Workflows

| Workflow | Purpose | Output | Best For |
|----------|---------|--------|----------|
| `/dev` | Immediate implementation | Working code | Quick feature development |
| `/proj-gen-standards` | Standards extraction | CODESTYLE.md + migration plan | Documenting team conventions |
| **`/proj-check-standards`** | **Compliance checking** | **Actionable improvement tasks** | **Identifying violations, code audits** |
| `/proj-gen-task-prompts` | Prompt generation | AI-executable prompts | Distributed execution |

## When to Use

### Ideal Scenarios
- **After standards generation**: Check compliance following `/proj-gen-standards`
- **Pre-release audits**: Verify code quality before major releases
- **Quarterly reviews**: Regular compliance checks to prevent drift
- **Team onboarding**: Show new members current improvement areas
- **Refactoring planning**: Identify technical debt before refactoring
- **CI/CD integration**: Automated compliance checking in pipelines

### Not Recommended For
- **No standards documentation**: Run `/proj-gen-standards` first to establish standards
- **Brand new projects**: No code to check
- **Quick bug fixes**: Use `/debug` or `/code` for immediate fixes
- **Feature development**: Use `/dev` or `/proj-gen-task-prompts` instead

## Command Usage

```bash
# Basic usage - check against auto-detected standards
/proj-check-standards

# Specify standards document location
/proj-check-standards --standards-file=./CODESTYLE.md

# Focus on critical issues only
/proj-check-standards --severity p0

# Quick mode (skip detailed examples, faster)
/proj-check-standards --quick

# Custom output directory
/proj-check-standards --output=./improvements/

# Output location (default)
.claude/improvements/{project_name}/
```

## Workflow Steps

### Step 1: Environment Setup & Standards Discovery (2-3 minutes)

**Objective**: Locate and verify project standards documentation

**Actions**:
1. **Auto-detect standards documents** (search order):
   ```
   - ./.claude/standards/{project}/CODESTYLE.md
   - ./CODESTYLE.md
   - ./docs/CODESTYLE.md
   - ./.github/CODESTYLE.md
   ```

2. **Detect project context**:
   - Primary language (file extension count)
   - Framework (package.json, requirements.txt, go.mod)
   - Project name (for output directory naming)

3. **Create output directory**: `./.claude/improvements/{project_name}/`

4. **Validation**:
   - If no standards found, prompt user to run `/proj-gen-standards` first
   - Verify standards document is valid and parseable

**Output**: Project metadata + standards document location

---

### Step 2: Standards Document Parsing (5-10 minutes)

**Objective**: Extract all rules, severity levels, exceptions, and scope

**Actions**: Invokes `standards-parser` agent

The agent will:
- Read all standards documents (CODESTYLE.md, CODE_REVIEW_CHECKLIST.md, etc.)
- Extract rules with:
  - Rule ID, severity (Required/Recommended/Optional)
  - Description and rationale
  - Good/bad code examples
  - Enforcement method (automated via ESLint, manual, etc.)
  - Exceptions and scope (file patterns, legacy code exemptions)
- Parse tool configurations (.eslintrc, .prettierrc, tsconfig)
- Build searchable rule index by category
- Flag ambiguous rules requiring human judgment

**Deliverables**:
- Complete rule inventory with severity classification
- Exception patterns with justifications
- Tool-enforceable vs manual-review rules
- Ambiguous cases flagged for clarification

**Output**: `./.claude/improvements/{project}/01-standards-index.yaml`

**Quality Checks**:
- [ ] All rules have clear severity (Required/Recommended/Optional)
- [ ] Exceptions documented with scope and reason
- [ ] Tool configurations parsed correctly
- [ ] Statistics accurate (total rules, by severity, by enforcement method)

---

### Step 3: Comprehensive Codebase Scan (15-20 minutes)

**Objective**: Identify all violations and improvement opportunities

**Actions**: Uses `codeagent` skill for deep scanning

**Scan Scope**:

**A. Project Standards Compliance**:
- **Required Rules**: Find ALL violations with exact locations
- **Recommended Rules**: Find violations, note context that may justify exceptions
- **Optional Rules**: Sample top violations (not exhaustive)

**B. Best Practices Compliance** (filtered by project standards):

**Security** (OWASP Top 10, CWE Top 25):
- Hardcoded secrets/credentials
- SQL injection vulnerabilities
- XSS vulnerabilities (unsanitized user input)
- Insecure dependencies (known CVEs)
- Missing authentication/authorization
- Weak cryptography (hardcoded keys, weak algorithms)

**Performance**:
- N+1 query patterns
- Missing database indexes
- Large bundle sizes (frontend)
- Memory leaks (event listeners, circular refs)
- Inefficient algorithms (O(n²) where O(n) possible)

**Reliability**:
- Missing error handling (try-catch for async)
- Race conditions (concurrent access)
- Resource leaks (unclosed files, connections)
- Missing input validation
- Improper null/undefined handling

**Maintainability**:
- High complexity (cyclomatic > 10)
- Large files (>500 lines)
- Deep nesting (>3 levels)
- Code duplication
- Missing documentation (complex public APIs)
- Dead code (unused functions, imports)

**Testing**:
- Low coverage (<80% overall, <90% critical paths)
- Missing edge case tests
- Missing error scenario tests
- Brittle tests (implementation-dependent)

**Conflict Resolution**:
- **Critical**: Before reporting best practice violation, check if project standards explicitly allow it
- If conflict detected, **SKIP** the violation (don't report)
- Flag ambiguous cases for manual review

**Output Format** (per violation):
```yaml
- id: VIOLATION-{category}-{number}
  severity: P0 | P1 | P2  # Assigned in Step 4
  category: security | performance | reliability | maintainability | testing
  rule_source: project_standards | best_practices
  rule_id: "{rule identifier}"

  location:
    file: "path/to/file.ts"
    line: 123
    function: "functionName"

  current_code: |
    {exact code showing violation, 3-5 lines context}

  violation_details: |
    {clear explanation why this is a violation}

  specific_fix:
    description: "{what needs to change}"
    fixed_code: |
      {exact code after fix}
    rationale: "{why this fix is correct}"

  estimated_effort: "{XS/S/M/L/XL}"
  files_affected: 1
```

**Output**: `./.claude/improvements/{project}/02-scan-results.yaml`

---

### Step 4: Violation Categorization & Prioritization (10-15 minutes)

**Objective**: Classify violations by severity and identify patterns

**Actions**: Invokes `violation-classifier` agent

The agent will:

**Severity Assignment Logic**:

**P0 (Critical)** - Must fix immediately:
- Security vulnerabilities with exploit potential (SQL injection, XSS, secrets)
- Reliability risks causing crashes/data loss (unhandled exceptions in critical paths)
- Required rule violations in production code

**P1 (Important)** - Should fix soon:
- Recommended rule violations (consistency issues)
- Performance issues with measurable impact (N+1 queries, missing indexes)
- Maintainability risks (high complexity, significant duplication)

**P2 (Nice-to-have)** - Can defer:
- Optional rule violations
- Code quality improvements (moderate complexity, minor duplication)
- Non-critical documentation gaps

**Statistics Calculation**:
- Violations by severity (P0/P1/P2 counts and percentages)
- Violations by category (security, reliability, performance, etc.)
- Violations by rule source (project standards vs best practices)
- Effort distribution (XS/S/M/L/XL task counts)

**Hotspot Analysis**:
- Top 10 files with most violations
- Top categories requiring attention
- Files/modules recommended for refactoring
- Pattern-based issues (e.g., "error handling missing project-wide")

**Quick Win Identification**:
- High impact, low effort tasks (P0/P1 severity, XS/S effort)
- ROI calculation: (impact score / effort score)
- Criteria: ROI >= 5:1

**Batch Fix Opportunities**:
- ESLint auto-fixable violations
- Rename refactorings (IDE-assisted)
- Template-based fixes (e.g., add try-catch pattern to multiple functions)
- Automation level: full | semi | tool-assisted

**Deliverables**:
- Prioritized violation list with rationale
- Comprehensive statistics dashboard
- Hotspot files and categories
- Quick wins list
- Batch fix groups

**Output**: `./.claude/improvements/{project}/03-prioritized-violations.yaml`

---

### Step 5: Improvement Task Generation (15-20 minutes)

**Objective**: Convert violations into executable tasks with zero ambiguity

**Actions**: Invokes `task-generator` agent

The agent will generate detailed task files using this structure:

```markdown
## Task: TASK-{ID} - {Action-Oriented Title}

**Priority**: P0 | P1 | P2
**Category**: {security | performance | reliability | maintainability | testing}
**Rule Source**: {Project Standards | Best Practices}
**Estimated Effort**: {XS: <1h | S: 1-4h | M: 4-8h | L: 1-2d | XL: >2d}
**Files Affected**: {count}
**Risk Level**: {High | Medium | Low}

---

### Problem Statement

{Clear 2-3 sentence description of violation and why it matters}

**Current Impact**:
- {Specific impact on users/system/developers}
- {Quantified impact: "affects 100K users", "adds 500ms latency"}

**Risk if Not Fixed**:
- {Specific risk description}
- {Potential damage/cost}

---

### Current State

**File**: `{path/to/file.ts:line}`
**Function/Class**: `{context}`

```{language}
{exact current code with 3-5 lines context}
```

**Violation**: {which rule/practice is violated}
**Rule Reference**: {CODESTYLE.md:line or best practice URL}

---

### Required Changes

#### Change 1: {Description}

**File**: `{path/to/file.ts}`
**Lines**: {start}-{end}

**Current**:
```{language}
{exact current code}
```

**Fixed**:
```{language}
{exact fixed code with comments explaining key changes}
// Reason: why this approach solves the problem
```

**Changes Made**:
1. {Specific change 1}
2. {Specific change 2}

**Why This Fix Works**:
{Clear explanation + alternatives considered if relevant}

---

### Acceptance Criteria

- [ ] {Specific testable criterion 1}
- [ ] {Specific testable criterion 2}
- [ ] Code passes linting and type checking
- [ ] Tests added/updated to cover fix
- [ ] No regression in existing functionality

---

### Verification Steps

**1. Pre-Implementation**:
```bash
{command to reproduce current issue}
# Expected: {what you should see}
```

**2. Implementation Verification**:
```bash
{command to check syntax/validity}
```

**3. Functional Verification**:
```bash
{command to test fix works}
# Expected: {all tests pass}
```

**4. Regression Verification**:
```bash
{command to run full test suite}
```

**Manual Checks**:
- [ ] {Manual verification step 1}
- [ ] {Manual verification step 2}

---

### Testing Requirements

**Unit Tests**:
```{language}
describe('{Feature}', () => {
  it('should {expected behavior after fix}', () => {
    // Test code
  });
});
```

**Manual Test Scenarios**:
1. {Scenario}: {steps} → {expected result}

---

### Related Tasks

**Depends On**: {TASK-ID} (if any)
**Blocks**: {TASK-ID} (if any)
**Related**: {TASK-ID} (similar violations)
**Part of Batch**: {BATCH-ID} (if applicable)

---

### Implementation Notes

**Suggested Approach**:
1. {Step 1 with commands}
2. {Step 2}

**Common Pitfalls**:
- ⚠️ {Pitfall 1}: {how to avoid}

**References**:
- {Guide/Documentation URL}
```

**Task Grouping Strategy**:
- **Single-violation tasks**: P0 security, P0 reliability, complex fixes (>100 lines)
- **Multi-violation tasks**: Same rule violated in multiple locations (max 5 violations, max 3 files)
- **Batch tasks**: ESLint auto-fix, rename refactoring, template-based fixes

**Quality Gates** (per task):
- [ ] Exact file paths and line numbers
- [ ] Fixed code provided (not descriptions)
- [ ] Verification steps concrete and actionable
- [ ] Effort estimate realistic
- [ ] Dependencies identified
- [ ] Testing requirements specified

**Output**: `./.claude/improvements/{project}/tasks/TASK-{ID}.md` (one per task)

---

### Step 6: Improvement Requirements Document (10-15 minutes)

**Objective**: Generate master document summarizing all improvements

**Actions**: Invokes `requirements-documenter` agent

The agent will generate `IMPROVEMENTS.md` with:

**Executive Summary**:
- Total improvements by severity (P0/P1/P2)
- Estimated total effort (person-days/weeks)
- Impact analysis (security, reliability, performance, maintainability, testing)
- Quick wins count
- Batch-fixable count
- Recommended approach (phased plan)

**Hotspot Analysis**:
- Top 10 files requiring most changes
- Top categories needing attention
- Refactoring recommendations

**Critical Improvements (P0)**:
- List all P0 tasks with summaries
- Group by category (security, reliability)
- Link to detailed task files

**Important Improvements (P1)**:
- List all P1 tasks grouped by category
- Standards compliance, performance, maintainability

**Nice-to-Have Improvements (P2)**:
- Condensed list, focus on categories
- Optional improvements

**Quick Wins Section**:
- High impact, low effort tasks
- ROI ratios
- Recommendation to start here

**Batch-Fixable Tasks**:
- Groups of tasks fixable with automation
- Commands/tools to use
- Effort estimates

**Implementation Roadmap**:
- **Phase 1** (Weeks 1-2): Critical fixes (P0)
- **Phase 2** (Weeks 3-5): Standards compliance & performance (P1)
- **Phase 3** (Weeks 6-8): Code quality polish (P2)
- Week-by-week breakdown with milestones

**Quick Start Guide**:
- Step-by-step instructions
- Commands to run
- Integration with other workflows

**Statistics Dashboard**:
- Violations by severity (table)
- Violations by category (table)
- Violations by rule source (table)
- Effort distribution (table)

**Appendices**:
- Standards documentation references
- Best practice references
- Excluded patterns
- Ambiguous cases requiring manual review

**Output**: `./.claude/improvements/{project}/IMPROVEMENTS.md`

---

## Output Directory Structure

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

### Example 1: First-Time Standards Check

```bash
# Step 1: Generate standards (if not already done)
/proj-gen-standards

# Step 2: Check compliance
/proj-check-standards

# Review results
cat .claude/improvements/{project}/IMPROVEMENTS.md | less
```

**Expected Results**:
- 47 violations found (5 P0, 18 P1, 24 P2)
- Estimated effort: 3 weeks
- Quick wins: 8 tasks (<4 hours each, high impact)
- Batch fixable: 28 violations (ESLint auto-fix)

**Next Steps**:
1. Review executive summary
2. Fix all P0 tasks (week 1)
3. Apply batch fixes (automated)
4. Address P1 tasks (weeks 2-3)

---

### Example 2: Focus on Security Vulnerabilities Only

```bash
# Check for critical security issues only
/proj-check-standards --severity p0 --quick

# Review critical tasks
ls .claude/improvements/{project}/tasks/TASK-*-P0-security.md

# Implement first task
cat .claude/improvements/{project}/tasks/TASK-001-P0-security.md
```

**Results**:
- 5 critical security vulnerabilities
- SQL injection in auth endpoint
- Hardcoded API key in config
- XSS vulnerability in user input
- Insecure session management
- Missing CSRF protection

**Estimated Effort**: 3 days
**Priority**: Fix immediately before next release

---

### Example 3: Quarterly Code Quality Review

```bash
# Full compliance check
/proj-check-standards

# Compare with previous quarter
diff .claude/improvements/{project}/IMPROVEMENTS.md \
     .claude/improvements/{project}/IMPROVEMENTS-2024-Q3.md

# Generate metrics report
cat .claude/improvements/{project}/03-prioritized-violations.yaml | \
  grep 'total_violations:' # Track trend over time
```

**Metrics to Track**:
- Total violations: 47 (Q4) vs 62 (Q3) - ✅ **24% reduction**
- P0 violations: 5 (Q4) vs 8 (Q3) - ✅ **37% reduction**
- Test coverage: 82% (Q4) vs 75% (Q3) - ✅ **7% improvement**

---

### Example 4: CI/CD Integration

```yaml
# .github/workflows/standards-check.yml
name: Standards Compliance Check
on: [pull_request]

jobs:
  check-standards:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Check critical violations
        run: |
          /proj-check-standards --severity p0 --quick

      - name: Fail if P0 violations found
        run: |
          P0_COUNT=$(cat .claude/improvements/{project}/03-prioritized-violations.yaml | \
            grep 'p0:' | awk '{print $2}')
          if [ "$P0_COUNT" -gt 0 ]; then
            echo "❌ Found $P0_COUNT critical violations"
            exit 1
          fi

      - name: Comment on PR
        if: failure()
        run: |
          gh pr comment ${{ github.event.pull_request.number }} \
            --body "⚠️ Standards check failed. See artifacts for details."

      - uses: actions/upload-artifact@v3
        if: failure()
        with:
          name: standards-violations
          path: .claude/improvements/{project}/
```

---

## Task Structure Deep Dive

### Example: Complete Security Task

Here's what a real task file looks like:

```markdown
## Task: TASK-001-P0-security - Fix SQL Injection in Login Endpoint

**Priority**: P0
**Category**: security
**Rule Source**: Best Practices (OWASP A1)
**Estimated Effort**: M (4-8h)
**Files Affected**: 1
**Risk Level**: High

---

### Problem Statement

User-controlled input is directly interpolated into SQL query, allowing
SQL injection attacks that could compromise the entire database.

**Current Impact**:
- Affects all 100K users attempting login
- Attacker can bypass authentication
- Attacker can dump entire user database

**Risk if Not Fixed**:
- Complete database compromise
- User data breach (PII, passwords, hashes)
- Legal/compliance violations (GDPR, HIPAA)
- Reputational damage

---

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

**Violation**: String interpolation in SQL query
**Issue**: Input like `admin@example.com' OR '1'='1` bypasses authentication

---

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
2. Passed email as parameter array
3. Database driver now handles escaping

**Why This Fix Works**:
Parameterized queries separate SQL logic from data. The database driver
treats parameters as literal values, not executable SQL, making injection
impossible.

**Alternative Approaches Considered**:
- Manual escaping: ❌ Error-prone, misses edge cases
- ORM (e.g., TypeORM): ✅ Also good, but bigger change
- Input validation only: ❌ Insufficient, can be bypassed

---

### Acceptance Criteria

- [ ] SQL query uses parameterized placeholders
- [ ] Email parameter passed as array to db.query()
- [ ] Injection attempts fail (tested with 5+ payloads)
- [ ] Normal login still works
- [ ] Unit tests added covering injection attempts
- [ ] Security scan shows zero SQL injection vulnerabilities

---

### Verification Steps

**1. Pre-Implementation** (verify vulnerability exists):
```bash
curl -X POST http://localhost:3000/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@example.com'"'"' OR '"'"'1'"'"'='"'"'1","password":"any"}'
# Expected: 200 OK (vulnerability confirmed)
```

**2. Implementation Verification**:
```bash
npm run lint
npm run type-check
# Expected: No errors
```

**3. Functional Verification**:
```bash
# Normal login should work
curl -X POST http://localhost:3000/api/login \
  -d '{"email":"valid@example.com","password":"correctpass"}'
# Expected: 200 OK with valid token

# Injection should fail
curl -X POST http://localhost:3000/api/login \
  -d '{"email":"admin@example.com'"'"' OR '"'"'1'"'"'='"'"'1","password":"any"}'
# Expected: 401 Unauthorized
```

**4. Security Testing**:
```bash
npm run security:scan
# Expected: No SQL injection vulnerabilities
```

**Manual Verification**:
- [ ] Tested 5 common SQL injection payloads (all failed)
- [ ] Error messages don't leak DB structure
- [ ] Logging captures failed injection attempts

---

### Testing Requirements

**Unit Tests** (add to `src/api/auth.test.ts`):
```typescript
describe('loginUser', () => {
  it('should prevent SQL injection via email parameter', async () => {
    const maliciousEmail = "admin@example.com' OR '1'='1";
    const result = await loginUser(maliciousEmail, 'any');

    expect(result).toBeNull(); // No user found
    // Verify query was parameterized (mock assertion)
    expect(db.query).toHaveBeenCalledWith(
      expect.stringContaining('$1'),
      [maliciousEmail]
    );
  });

  it('should still authenticate valid users', async () => {
    const result = await loginUser('valid@example.com', 'correctpass');
    expect(result).toBeTruthy();
    expect(result.email).toBe('valid@example.com');
  });
});
```

**Manual Test Scenarios**:
1. **SQL Injection Attempts**: Try 5 payloads from OWASP list → All fail
2. **Normal Login**: Valid credentials → Success
3. **Invalid Login**: Wrong password → Fail with proper error

---

### Related Tasks

**Related**: TASK-003-P0-security (XSS in user profile)
**Part of Batch**: BATCH-002 (Security vulnerability remediation)

---

### Implementation Notes

**Suggested Approach**:
1. Update auth.ts line 45-46 with parameterized query
2. Run unit tests: `npm test src/api/auth.test.ts`
3. Run security scan: `npm run security:scan`
4. Manual injection testing (see verification steps)
5. Create PR with title: "Security: Fix SQL injection in login endpoint (TASK-001)"

**Common Pitfalls**:
- ⚠️ Don't use template literals for parameterized queries
  - Wrong: `` query($1) ``, `[${email}]` `` (still vulnerable)
  - Right: `query, [email]`
- ⚠️ Ensure ALL user input uses parameterization, not just email

**References**:
- OWASP SQL Injection: https://owasp.org/www-community/attacks/SQL_Injection
- PostgreSQL Parameterized Queries: https://node-postgres.com/features/queries
- Similar fix in codebase: `src/api/users.ts:89`
```

---

## Quality Assurance

### Scan Quality
- [ ] All standards rules checked
- [ ] Best practices filtered to avoid conflicts with project standards
- [ ] File paths and line numbers accurate (spot-check 10%)
- [ ] Violation counts match actual issues
- [ ] No false positives (manual verification of samples)

### Task Quality
- [ ] Every task has specific file locations with line numbers
- [ ] Fixed code provided (not just "improve error handling")
- [ ] Verification steps are actionable (copy-paste commands)
- [ ] Effort estimates realistic (±50% accuracy target)
- [ ] Dependencies identified correctly
- [ ] Testing requirements complete
- [ ] No ambiguity requiring additional research

### Document Quality
- [ ] Executive summary clear and fits on 2 pages
- [ ] Roadmap achievable (validated with team capacity)
- [ ] Statistics accurate (cross-check with source data)
- [ ] Quick wins genuinely quick (<4 hours each)
- [ ] Integration guidance specific and actionable
- [ ] All task file links correct

---

## Troubleshooting

### Issue: Standards Document Not Found

**Symptoms**: "Error: Could not locate CODESTYLE.md"

**Solution**:
```bash
# Generate standards first
/proj-gen-standards

# Then run check
/proj-check-standards
```

---

### Issue: Too Many Violations (Overwhelming)

**Symptoms**: 200+ violations, team feels overwhelmed

**Solution**:
```bash
# Focus on critical issues only
/proj-check-standards --severity p0

# After fixing P0, move to P1
/proj-check-standards --severity p1

# Use quick wins for morale boost
cat .claude/improvements/{project}/IMPROVEMENTS.md | grep -A 20 "Quick Wins"
```

**Strategy**:
1. Week 1: Fix all P0 (typically 5-10 tasks)
2. Week 2: Apply batch fixes (ESLint auto-fix, etc.)
3. Week 3+: Address P1 tasks incrementally

---

### Issue: Scan Missed Known Violations

**Symptoms**: Manual code review found issues not in report

**Cause**: Scan logic doesn't cover specific patterns

**Solution**:
1. Check `02-scan-results.yaml` for excluded patterns
2. Review project standards for missing rules
3. Add rule to CODESTYLE.md
4. Re-run check: `/proj-check-standards`
5. Report false negative for workflow improvement

---

### Issue: False Positives (Code is Actually Correct)

**Symptoms**: Task claims violation, but code is intentionally written that way

**Solution**:
1. Review task details and rationale
2. If legitimately correct:
   - Add exception to CODESTYLE.md
   - Document why this pattern is allowed
   - Re-run check to verify exception respected
3. If rule is too strict:
   - Update CODESTYLE.md to relax rule
   - Add context/examples for when exception applies

**Example Exception**:
```markdown
## Error Handling

**Rule**: All async functions must have try-catch blocks
**Severity**: Required
**Exceptions**:
- Test files (`**/__tests__/**`): Can omit for brevity
- Reason: Test framework handles errors automatically
```

---

### Issue: Verification Steps Fail

**Symptoms**: Following task verification steps, but tests fail unexpectedly

**Solution**:
1. Check task dependencies: Are prerequisite tasks completed?
2. Verify environment setup (packages installed, DB running, etc.)
3. Check for outdated task (codebase changed since scan)
4. Re-run scan if major changes occurred
5. Consult task "Common Pitfalls" section

---

## Integration Points

### With `/proj-gen-standards` Workflow

**Typical Sequence**:
```bash
# 1. Generate standards from existing codebase
/proj-gen-standards

# 2. Review and approve CODESTYLE.md
cat .claude/standards/{project}/CODESTYLE.md

# 3. Check compliance against established standards
/proj-check-standards

# 4. Implement improvements
# ... work through tasks ...

# 5. Re-check to verify compliance
/proj-check-standards --quick
```

**Integration Benefits**:
- Standards extraction (gen) → Compliance checking (check)
- Closed feedback loop: improve code, re-check, repeat
- Metrics tracking over time

---

### With `/dev` Workflow

**Execute Individual Tasks**:
```bash
# Read task details
cat .claude/improvements/{project}/tasks/TASK-001-P0-security.md

# Execute with /dev
/dev "Implement TASK-001: Fix SQL injection in auth endpoint

$(cat .claude/improvements/{project}/tasks/TASK-001-P0-security.md)"
```

**Benefits**:
- Task provides complete context (no additional research needed)
- Verification steps included
- Testing requirements specified

---

### With `/proj-gen-task-prompts` Workflow

**Generate Detailed Execution Prompts**:
```bash
# Convert improvement requirements to AI-executable prompts
/proj-gen-task-prompts "Implement all P0 security fixes from:
@.claude/improvements/{project}/IMPROVEMENTS.md

Focus on tasks: TASK-001, TASK-002, TASK-005"
```

**Output**: Detailed prompts with:
- Unambiguous technical roadmap
- Step-by-step implementation strategy
- Complete test coverage requirements
- Ready for distributed AI execution

---

### With CI/CD Pipeline

**Prevent Regressions**:
```yaml
# .github/workflows/standards-check.yml
name: Standards Compliance Check
on: [pull_request]

jobs:
  check-standards:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Check for critical violations
        run: /proj-check-standards --severity p0 --quick

      - name: Fail if P0 violations exist
        run: |
          P0_COUNT=$(cat .claude/improvements/*/03-prioritized-violations.yaml | \
            grep 'p0:' | awk '{print $2}')
          if [ "$P0_COUNT" -gt 0 ]; then
            echo "❌ Found $P0_COUNT critical violations - blocking PR"
            exit 1
          fi
```

**Track Progress Over Time**:
```yaml
# .github/workflows/metrics.yml
name: Code Quality Metrics
on:
  schedule:
    - cron: '0 0 * * 1'  # Weekly on Monday

jobs:
  metrics:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Run standards check
        run: /proj-check-standards --quick

      - name: Extract metrics
        run: |
          TOTAL=$(grep 'total_violations:' .claude/improvements/*/03-prioritized-violations.yaml | awk '{print $2}')
          P0=$(grep 'p0:' .claude/improvements/*/03-prioritized-violations.yaml | awk '{print $2}')
          echo "Total violations: $TOTAL"
          echo "Critical (P0): $P0"

      - name: Post to Slack
        run: |
          curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
            -H 'Content-Type: application/json' \
            -d '{"text":"📊 Weekly Code Quality: '$TOTAL' violations ('$P0' critical)"}'
```

---

### With Code Review Process

**Use in PR Reviews**:
```bash
# Check branch for violations
git checkout feature/new-feature
/proj-check-standards --quick

# Compare with main branch
git checkout main
/proj-check-standards --quick --output=./baseline/
git checkout feature/new-feature
/proj-check-standards --quick --output=./current/

# Report new violations introduced
diff baseline/03-prioritized-violations.yaml current/03-prioritized-violations.yaml
```

**PR Checklist Integration**:
```markdown
## PR Checklist

- [ ] All P0 violations resolved (run `/proj-check-standards --severity p0`)
- [ ] No new P1 violations introduced
- [ ] Batch-fixable issues addressed (ESLint auto-fix applied)
- [ ] Code review checklist followed (from CODESTYLE.md)
```

---

## Best Practices

### 1. Start with Critical Issues (P0)
Always address P0 security and reliability issues before anything else. They pose immediate risk.

### 2. Apply Batch Fixes Early
Use automated tools (ESLint --fix, Prettier) to quickly resolve many violations at once.

### 3. Track Progress with Metrics
Run weekly checks and track trends:
- Total violations over time
- P0 count (should trend to zero)
- Test coverage percentage
- Technical debt ratio

### 4. Integrate into CI/CD
Block PRs with new P0 violations. This prevents regression and maintains code quality.

### 5. Celebrate Quick Wins
Start with quick win tasks to build momentum and demonstrate value to the team.

### 6. Re-Check Regularly
Run quarterly to prevent standards drift and catch issues early.

### 7. Update Standards as Needed
If many false positives occur, standards may be too strict. Update CODESTYLE.md and re-check.

### 8. Use Task Files Directly
Task files are designed to be self-contained. Developers can execute them without additional context.

### 9. Involve the Team
Share IMPROVEMENTS.md executive summary. Discuss priorities and assign tasks collaboratively.

### 10. Automate Where Possible
Prefer tool-enforced rules (ESLint, Prettier) over manual review. Reduces burden and ensures consistency.

---

## See Also

- [Project Standards Checker Module README](../proj-check-standards-workflow/README.md)
- [Standards Parser Agent](../proj-check-standards-workflow/agents/standards-parser.md)
- [Violation Classifier Agent](../proj-check-standards-workflow/agents/violation-classifier.md)
- [Task Generator Agent](../proj-check-standards-workflow/agents/task-generator.md)
- [Requirements Documenter Agent](../proj-check-standards-workflow/agents/requirements-documenter.md)

---

**Version**: 1.0
**Last Updated**: 2025-12-18
**Status**: Production Ready
