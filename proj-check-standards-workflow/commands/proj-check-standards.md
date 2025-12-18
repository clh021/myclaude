---
description: Check codebase against project standards and best practices, generate actionable improvement tasks
---

## Usage
`/proj-check-standards [OPTIONS]`

### Options
- `--standards-file`: Path to standards document (default: auto-detect CODESTYLE.md)
- `--severity`: Minimum severity to report (p0, p1, p2, all) (default: all)
- `--quick`: Quick mode (skip detailed code examples, faster execution)
- `--output`: Output directory (default: ./.claude/improvements/)

## Context
- Analyzes codebase against existing project standards documentation
- Checks against industry best practices (filtered to avoid conflicts)
- Generates executable improvement tasks with specific file paths and fixes
- Output: $ARGUMENTS

## Your Role

You are the Project Standards Checker Workflow Orchestrator managing a 6-step pipeline to identify standards violations and best practice gaps. Your responsibility is generating **actionable, task-level improvement requirements** that can be directly converted to implementation tasks without ambiguity.

## Core Principles

### Dual-Source Checking
- **Primary Source**: Project standards documentation (CODESTYLE.md, etc.)
  - Enforce all Required rules strictly
  - Check Recommended rules with context
  - Note Optional rules for completeness
- **Secondary Source**: Industry best practices
  - Language-specific guides (ESLint, PEP 8, Go conventions)
  - Security standards (OWASP, CWE)
  - Performance patterns
  - **Conflict Resolution**: Skip suggestions that contradict project standards

### Task-Level Specificity
Every improvement must include:
- **Exact Location**: File path, line numbers, function/class name
- **Current State**: Code snippet showing the violation
- **Violation Details**: Which rule is violated and why
- **Specific Fix**: Exact code change required (not just "improve X")
- **Expected Result**: What the code should look like after fix
- **Verification**: How to verify the fix is correct

### Prioritization
- **P0 (Critical)**: Security vulnerabilities, reliability risks, Required rule violations
- **P1 (Important)**: Recommended rule violations, performance issues, maintainability risks
- **P2 (Nice-to-have)**: Optional rule violations, code quality improvements

### Conflict Avoidance
- **Ignore** best practices that contradict project standards
- **Flag** ambiguous cases where standards are unclear
- **Respect** explicit exceptions documented in standards

## Workflow Execution

### Step 1: Environment Setup & Standards Discovery (2-3 minutes)

**Objective**: Locate and parse project standards documentation

**Actions**:
1. **Auto-detect standards documents**:
   ```bash
   # Search order
   - ./.claude/standards/{project}/CODESTYLE.md
   - ./CODESTYLE.md
   - ./docs/CODESTYLE.md
   - ./.github/CODESTYLE.md

   # If not found, prompt user for location
   ```

2. **Detect project information**:
   - Primary language (file extension count)
   - Framework (package.json, requirements.txt, go.mod)
   - Project name (for output directory)

3. **Create output directory**: `./.claude/improvements/{project_name}/`

**Output**: Project context + standards document location

---

### Step 2: Standards Document Parsing (5-10 minutes)

**Objective**: Extract all rules, exceptions, and severity levels

**Actions**:
Invoke `standards-parser` agent (Task tool with subagent_type='standards-parser'):

The agent will:
- Read standards document(s) (CODESTYLE.md, CODE_REVIEW_CHECKLIST.md, etc.)
- Extract all rules with:
  - Rule ID (if available)
  - Severity (Required/Recommended/Optional)
  - Description and rationale
  - Good/bad code examples
  - Enforcement method (automated/manual)
  - Exceptions and special cases
- Build rule index by category (naming, formatting, error handling, etc.)
- Identify ambiguous rules requiring human judgment
- Extract explicit ignore patterns (e.g., "legacy code exempt")

**Quality Checks**:
- [ ] All rules have clear severity classification
- [ ] Exceptions documented with scope (files/patterns)
- [ ] Tool-enforceable rules identified (ESLint, Prettier, etc.)

**Output**: `./.claude/improvements/{project}/01-standards-index.yaml`

---

### Step 3: Comprehensive Codebase Scan (15-20 minutes)

**Objective**: Identify all violations and improvement opportunities

**Actions**:
Use codeagent skill for deep scanning:

```bash
codeagent-wrapper - <<'EOF'
Perform comprehensive standards compliance scan for project: {project_name}

## Scan Scope

### A. Project Standards Compliance
Based on standards document at: {standards_file_path}

For each rule in standards index:
1. **Required Rules**: Find ALL violations
   - Report file path, line number, code snippet
   - Count total violations per rule
   - Identify hotspot files (most violations)

2. **Recommended Rules**: Find violations with context
   - Same as Required, but note context that may justify exception
   - Flag cases where exception might be reasonable

3. **Optional Rules**: Sample violations (top 10 files)
   - Don't exhaustively scan, just show examples

### B. Best Practices Compliance
Check against industry standards (filtered by project standards):

**Security** (OWASP Top 10):
- Hardcoded secrets/credentials
- SQL injection vulnerabilities (unparameterized queries)
- XSS vulnerabilities (unsanitized user input)
- Insecure dependencies (known CVEs)
- Missing authentication/authorization checks
- Insecure cryptography (weak algorithms, hardcoded keys)

**Performance**:
- N+1 query patterns
- Missing database indexes (slow queries)
- Large bundle sizes (frontend)
- Memory leaks (event listeners, circular refs)
- Inefficient algorithms (O(n²) where O(n) possible)

**Reliability**:
- Missing error handling (try-catch for async, error returns)
- Race conditions (concurrent access, missing locks)
- Resource leaks (unclosed files, connections)
- Missing input validation
- Improper null/undefined handling

**Maintainability**:
- High complexity functions (cyclomatic > 10)
- Large files (>500 lines)
- Deep nesting (>3 levels)
- Code duplication (repeated logic)
- Missing documentation (complex public APIs)
- Dead code (unused functions, imports)

**Testing**:
- Missing test coverage (<80% overall, <90% critical paths)
- Missing edge case tests
- Missing error scenario tests
- Brittle tests (implementation-dependent)

## Conflict Resolution Rules
**CRITICAL**: Before reporting a best practice violation:
1. Check if project standards explicitly allow it
2. Check if it's in documented exceptions
3. If conflict detected, **SKIP** the violation (don't report)

## Output Format
For each violation, report:

```yaml
- id: VIOLATION-{category}-{number}
  severity: P0 | P1 | P2
  category: security | performance | reliability | maintainability | testing
  rule_source: project_standards | best_practices
  rule_id: "{rule identifier from standards or practice name}"
  rule_description: "{what rule is violated}"

  location:
    file: "path/to/file.ts"
    line: 123
    function: "functionName" # if applicable
    class: "ClassName" # if applicable

  current_code: |
    {exact code showing violation, 3-5 lines context}

  violation_details: |
    {clear explanation of why this is a violation}

  impact:
    severity_rationale: "{why P0/P1/P2}"
    risk: "{security/reliability/performance risk description}"
    affected_users: "{who is impacted}"

  specific_fix:
    description: "{what needs to change}"
    fixed_code: |
      {exact code after fix, matching context}
    rationale: "{why this fix is correct}"

  verification:
    manual_check: "{how to manually verify fix}"
    automated_check: "{test command or tool to verify}"

  estimated_effort: "{XS/S/M/L/XL - time estimate}"
  files_affected: 1 # how many files need changing
```

Save complete scan results to: ./.claude/improvements/{project}/02-scan-results.yaml
EOF
```

**Output**: `./.claude/improvements/{project}/02-scan-results.yaml`

---

### Step 4: Violation Categorization & Prioritization (10-15 minutes)

**Objective**: Group violations and assign priority

**Actions**:
Invoke `violation-classifier` agent (Task tool with subagent_type='violation-classifier'):

The agent will:
- Read scan results (02-scan-results.yaml)
- Apply prioritization logic:
  - **P0 (Critical)**:
    - Security vulnerabilities with exploit potential
    - Reliability risks causing crashes/data loss
    - Required rule violations in production code
  - **P1 (Important)**:
    - Recommended rule violations (consistency)
    - Performance issues with measurable impact
    - Maintainability risks (complexity, duplication)
  - **P2 (Nice-to-have)**:
    - Optional rule violations
    - Code quality improvements
    - Documentation gaps (non-critical)
- Calculate statistics:
  - Total violations by severity
  - Violations by category
  - Violations by file (hotspot analysis)
  - Estimated total effort (sum of individual estimates)
- Identify quick wins (high impact, low effort)
- Flag batch-fixable violations (can fix multiple with script/tool)

**Deliverables**:
- Prioritized violation list
- Hotspot files (top 10 files with most violations)
- Quick win opportunities
- Batch fix candidates
- Statistics dashboard

**Output**: `./.claude/improvements/{project}/03-prioritized-violations.yaml`

---

### Step 5: Improvement Task Generation (15-20 minutes)

**Objective**: Convert violations into executable improvement tasks

**Actions**:
Invoke `task-generator` agent (Task tool with subagent_type='task-generator'):

The agent will:
- Read prioritized violations (03-prioritized-violations.yaml)
- For each violation, generate detailed task:

**Task Structure** (following requirements-driven format):
```markdown
## Task: {TASK-ID} - {Brief Description}

**Priority**: P0 | P1 | P2
**Category**: {security | performance | reliability | maintainability | testing}
**Estimated Effort**: {XS: <1h | S: 1-4h | M: 4-8h | L: 1-2d | XL: >2d}
**Files Affected**: {count}

### Problem Statement
{Clear description of the violation and its impact}

### Current State
**File**: `{path/to/file.ts:line}`
**Function/Class**: `{context}`

```{language}
{current code with violation, 5-10 lines context}
```

**Violation**: {which rule/practice is violated}
**Impact**: {security/reliability/performance impact description}

### Required Changes

#### Change 1: {Description}
**File**: `{path/to/file.ts}`
**Lines**: {start}-{end}

**Current**:
```{language}
{current code}
```

**Fixed**:
```{language}
{fixed code with explanation comments}
```

**Rationale**: {why this fix is correct and safe}

#### Change 2: {Description}
{repeat if multiple changes needed}

### Acceptance Criteria
- [ ] {Specific testable criterion 1}
- [ ] {Specific testable criterion 2}
- [ ] {Verification method passed}

### Verification Steps
1. {Manual check step}
2. {Automated test command}
3. {Expected output/behavior}

### Related Tasks
- Depends on: {TASK-ID} (if applicable)
- Blocks: {TASK-ID} (if applicable)
- Related: {TASK-ID} (similar violations)

### Notes
- {Any special considerations}
- {Potential side effects}
- {Alternative approaches (if applicable)}
```

**Grouping Strategy**:
- Group similar violations across files (e.g., "Add error handling to all async functions")
- Create batch tasks where appropriate (e.g., "Fix all ESLint violations in src/components/")
- Keep complex tasks focused (1 file or 1 logical unit)

**Quality Gates**:
- [ ] Every task has specific file paths and line numbers
- [ ] Fixed code is provided (not just "fix X")
- [ ] Verification steps are concrete and actionable
- [ ] Effort estimation is realistic
- [ ] Dependencies are identified

**Output**: `./.claude/improvements/{project}/tasks/TASK-{ID}.md` (one per task)

---

### Step 6: Improvement Requirements Document (10-15 minutes)

**Objective**: Generate master document summarizing all improvements

**Actions**:
Invoke `requirements-documenter` agent (Task tool with subagent_type='requirements-documenter'):

The agent will:
- Aggregate all tasks into master document
- Generate executive summary
- Create roadmap with phases
- Provide quick-start guide

**Document Structure**:

```markdown
# Code Improvement Requirements - {Project Name}

**Generated**: {date}
**Standards Source**: {path to CODESTYLE.md}
**Scan Coverage**: {X files, Y lines of code}

## Executive Summary

### Overview
- Total Improvements: {count} ({P0} critical, {P1} important, {P2} nice-to-have)
- Estimated Total Effort: {days/weeks}
- Quick Wins: {count} tasks (high impact, low effort)
- Batch-Fixable: {count} tasks (can automate)

### Impact Analysis
- **Security**: {count} vulnerabilities ({critical} critical)
- **Reliability**: {count} risks
- **Performance**: {count} issues
- **Maintainability**: {count} improvements
- **Testing**: {count} coverage gaps

### Hotspot Files
Top 10 files requiring most changes:
1. `{file}` - {count} violations ({P0}/{P1}/{P2})
2. ...

### Recommended Approach
1. **Phase 1** (Week 1-2): Critical fixes (P0) - {count} tasks, {effort}
2. **Phase 2** (Week 3-5): Important improvements (P1) - {count} tasks, {effort}
3. **Phase 3** (Week 6-8): Quality polish (P2) - {count} tasks, {effort}

---

## Critical Improvements (P0)

{For each P0 task}
### TASK-{ID}: {Title}
**File**: `{path}:{line}`
**Category**: {category}
**Effort**: {estimate}
**Impact**: {description}

**Quick Summary**: {1-2 sentence description}

**Action**: See detailed task at `tasks/TASK-{ID}.md`

---

## Important Improvements (P1)

{Same structure as P0}

---

## Nice-to-Have Improvements (P2)

{Same structure as P0, may summarize instead of full details}

---

## Quick Wins (High Impact, Low Effort)

{List of tasks that provide significant benefit with minimal effort}

1. **TASK-{ID}**: {title} - {impact description} (Effort: {XS/S})
2. ...

---

## Batch-Fixable Tasks

{Groups of tasks that can be fixed with automation}

### Batch 1: ESLint Auto-Fix
**Command**: `npx eslint --fix src/**/*.ts`
**Tasks**: TASK-001, TASK-003, TASK-015 (total {count})
**Effort**: XS (automated)

### Batch 2: Add Error Handling Template
**Script**: Use code generation tool
**Tasks**: TASK-020, TASK-022, TASK-025 (total {count})
**Effort**: M (semi-automated)

---

## Implementation Roadmap

### Phase 1: Critical Security & Reliability (Weeks 1-2)
**Goal**: Eliminate critical vulnerabilities and crash risks

#### Week 1
- [ ] TASK-001: {title} (P0, {effort})
- [ ] TASK-005: {title} (P0, {effort})
- **Deliverable**: All P0 security issues resolved

#### Week 2
- [ ] TASK-010: {title} (P0, {effort})
- **Deliverable**: All P0 reliability issues resolved

**Success Metrics**:
- Zero critical security vulnerabilities
- Zero production crashes in monitored period

### Phase 2: Standards Compliance & Performance (Weeks 3-5)
{Similar structure}

### Phase 3: Code Quality Polish (Weeks 6-8)
{Similar structure}

---

## Statistics

### Violations by Severity
- P0 (Critical): {count} ({percentage}%)
- P1 (Important): {count} ({percentage}%)
- P2 (Nice-to-have): {count} ({percentage}%)

### Violations by Category
- Security: {count}
- Performance: {count}
- Reliability: {count}
- Maintainability: {count}
- Testing: {count}

### Violations by Rule Source
- Project Standards: {count} ({percentage}%)
- Best Practices: {count} ({percentage}%)

### Effort Distribution
- XS (<1h): {count} tasks
- S (1-4h): {count} tasks
- M (4-8h): {count} tasks
- L (1-2d): {count} tasks
- XL (>2d): {count} tasks

---

## Quick Start Guide

### 1. Review Critical Tasks
```bash
# See all P0 tasks
ls tasks/TASK-*-P0-*.md

# Review executive summary
cat IMPROVEMENTS.md | grep "## Critical"
```

### 2. Start with Quick Wins
```bash
# Quick wins provide immediate value
cat IMPROVEMENTS.md | grep "## Quick Wins"
```

### 3. Apply Batch Fixes
```bash
# Run automated fixes first
npx eslint --fix src/**/*.ts
npx prettier --write src/**/*.ts
```

### 4. Track Progress
```bash
# Create tracking spreadsheet or use project management tool
# Mark tasks as: Not Started | In Progress | In Review | Completed
```

---

## Integration with Other Workflows

### With `/dev` Workflow
```bash
# Execute individual improvement tasks
/dev "Implement TASK-001: $(cat tasks/TASK-001.md)"
```

### With `/proj-gen-task-prompts` Workflow
```bash
# Convert improvement requirements to detailed execution prompts
/proj-gen-task-prompts @.claude/improvements/{project}/IMPROVEMENTS.md
```

### With CI/CD
- Add automated checks for critical rules (ESLint, security scanners)
- Block PRs that introduce new P0 violations
- Track improvement metrics over time

---

## Appendix

### A. Standards Document Reference
{Link to standards document used for checking}

### B. Best Practices References
- {Language Style Guide URL}
- {Security Guidelines URL}
- {Framework Best Practices URL}

### C. Excluded Patterns
{List of intentionally ignored patterns from standards exceptions}

### D. Manual Review Required
{List of ambiguous cases requiring human judgment}
```

**Output**: `./.claude/improvements/{project}/IMPROVEMENTS.md`

---

## Output Structure

All outputs saved to `./.claude/improvements/{project_name}/`:

```
{project_name}/
├── 01-standards-index.yaml          # Parsed project standards
├── 02-scan-results.yaml             # Raw scan results
├── 03-prioritized-violations.yaml   # Categorized violations
├── IMPROVEMENTS.md                  # ✅ Master requirements document
└── tasks/
    ├── TASK-001-P0-security.md      # Individual improvement tasks
    ├── TASK-002-P0-reliability.md
    ├── TASK-015-P1-performance.md
    └── ...
```

## Quality Standards

### Scan Quality
- [ ] All standards rules checked
- [ ] Best practices filtered for conflicts
- [ ] File paths and line numbers accurate
- [ ] Violation counts match actual issues

### Task Quality
- [ ] Every task has specific file locations
- [ ] Fixed code provided (not just descriptions)
- [ ] Verification steps are actionable
- [ ] Effort estimates realistic
- [ ] Dependencies identified correctly

### Document Quality
- [ ] Executive summary clear and concise
- [ ] Roadmap is realistic and achievable
- [ ] Statistics accurate
- [ ] Quick wins identified correctly
- [ ] Integration guidance provided

## Error Handling

If any step encounters issues:

1. **Standards document not found**: Prompt user for location or offer to generate one using `/proj-gen-standards`
2. **Scan incomplete**: Retry with broader patterns, note incomplete coverage
3. **Ambiguous violations**: Flag for manual review, don't guess intent
4. **Conflicting best practices**: Always defer to project standards, document conflict
5. **Tool failures**: Fall back to manual analysis, note limitations

## Important Notes

- **Non-invasive**: This workflow generates documentation, does not modify code
- **Context-aware**: Respects project standards over generic best practices
- **Actionable**: Every improvement is task-ready with specific fixes
- **Prioritized**: Focus on high-impact issues first
- **Realistic**: Effort estimates and roadmap are achievable

## Integration with Other Workflows

Generated improvements can be used in:

1. **Sprint planning**: Convert tasks to user stories/tickets
2. **Code reviews**: Reference specific tasks in PR feedback
3. **Refactoring**: Follow roadmap for systematic improvement
4. **CI/CD**: Integrate automated checks for critical rules
5. **Team education**: Use examples for training on standards

---

**Workflow Version**: 1.0
**Last Updated**: 2025-12-18
**Maintained By**: Project Standards Workflow Team
