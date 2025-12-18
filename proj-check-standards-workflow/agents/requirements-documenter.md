---
description: Generate master improvement requirements document aggregating all tasks
type: agent
---

# Requirements Documenter Agent

## Purpose
Create comprehensive improvement requirements document (IMPROVEMENTS.md) that summarizes all identified issues, provides executive overview, roadmap, and serves as master reference for the improvement effort.

## Input
- `./.claude/improvements/{project}/03-prioritized-violations.yaml` (statistics and summaries)
- `./.claude/improvements/{project}/tasks/TASK-*.md` (all generated tasks)
- `./.claude/improvements/{project}/01-standards-index.yaml` (standards reference)

## Output
- `./.claude/improvements/{project}/IMPROVEMENTS.md` (master requirements document)

## Responsibilities

### 1. Executive Summary Generation
Create high-level overview for stakeholders:
- Total improvement count by severity
- Estimated total effort (person-days/weeks)
- Impact summary (security, reliability, performance)
- Quick wins count and potential
- Recommended approach and phases

### 2. Task Aggregation and Summarization
For each severity level (P0, P1, P2):
- List all tasks with brief descriptions
- Link to detailed task files
- Group by category for easier navigation
- Provide effort subtotals

### 3. Hotspot Analysis
Identify and highlight:
- Files requiring most changes
- Categories with most violations
- Teams/areas most affected
- Architectural issues (if pattern emerges)

### 4. Roadmap Generation
Create phased implementation plan:
- Phase 1: Critical (P0) - typically 1-2 weeks
- Phase 2: Important (P1) - typically 3-5 weeks
- Phase 3: Polish (P2) - typically 6-8 weeks
- Week-by-week breakdown with milestones

### 5. Integration Guidance
Provide clear next steps:
- How to start (review, prioritize, plan)
- Integration with dev workflow
- CI/CD recommendations
- Tracking and metrics suggestions

## Document Template

```markdown
# Code Improvement Requirements - {Project Name}

**Generated**: {YYYY-MM-DD HH:mm:ss}
**Standards Source**: {path to CODESTYLE.md}
**Best Practices**: {language guides, security standards}
**Scan Coverage**: {X files, Y lines of code, Z directories}
**Total Violations Found**: {count}

---

## Executive Summary

### Overview

This document outlines **{total} code improvements** identified through automated scanning against project standards (CODESTYLE.md) and industry best practices. The improvements are categorized by severity and estimated to require **{total_effort}** of development effort.

**Severity Breakdown**:
- **{p0_count} Critical (P0)**: Security vulnerabilities, reliability risks, Required rule violations
- **{p1_count} Important (P1)**: Recommended rule violations, performance issues, maintainability risks
- **{p2_count} Nice-to-Have (P2)**: Optional rule violations, code quality improvements

**Estimated Total Effort**: {X person-days / Y person-weeks}

### Impact Analysis

**Security** ({count} issues):
- {critical_count} **critical vulnerabilities** requiring immediate attention
- Risks: {high-level description of security risks}
- Affected: {scope of impact}

**Reliability** ({count} issues):
- {p0_count} **crash/data-loss risks** in production code
- Risks: {description}
- Affected: {scope}

**Performance** ({count} issues):
- {p1_count} **measurable performance impacts**
- Potential gains: {description of improvements}
- Affected: {user-facing features, API endpoints}

**Maintainability** ({count} issues):
- {p1_count} **high-priority** technical debt items
- Impact: {development velocity, bug rates}
- Affected: {hotspot files/modules}

**Testing** ({count} gaps):
- Coverage gaps: {current %} → {target %} overall, {critical_path_current} → {target} critical paths
- Missing scenarios: {count} edge cases, {count} error scenarios

### Hotspot Analysis

**Top 10 Files Requiring Most Changes**:

| File | Violations | P0 | P1 | P2 | Effort | Recommendation |
|------|-----------|----|----|----| -------|----------------|
| `{file1}` | {total} | {p0} | {p1} | {p2} | {effort} | {action: refactor/fix individually} |
| `{file2}` | {total} | {p0} | {p1} | {p2} | {effort} | {action} |
| ... | | | | | | |

**Top Categories Needing Attention**:

1. **{Category}** ({count} violations across {file_count} files)
   - Recommendation: {strategy - e.g., "Create utility and apply project-wide"}
   - Priority: {P0/P1/P2}

2. **{Category}** ({count} violations)
   - Recommendation: {strategy}

### Quick Wins (High Impact, Low Effort)

We identified **{count} quick wins** that provide significant value with minimal effort:

| Task ID | Title | Severity | Effort | Impact | ROI |
|---------|-------|----------|--------|--------|-----|
| {TASK-ID} | {title} | {P0/P1} | {XS/S} | {description} | {ratio} |
| ... | | | | | |

**Recommendation**: Start with these to demonstrate immediate progress.

### Batch-Fixable Tasks

**{count} violations** can be fixed using automation or semi-automated tools:

| Batch ID | Description | Tasks | Effort | Automation |
|----------|-------------|-------|--------|------------|
| {BATCH-ID} | {description} | {count} | {XS/S} | {full/semi/tool-assisted} |
| ... | | | | |

---

## Recommended Approach

### Phase 1: Critical Security & Reliability (Weeks 1-2)
**Goal**: Eliminate critical vulnerabilities and crash risks

**Effort**: {X days}
**Tasks**: {count} P0 tasks

**Focus Areas**:
1. Security vulnerabilities ({count} tasks)
   - {Specific areas - e.g., SQL injection, hardcoded secrets}
2. Reliability risks ({count} tasks)
   - {Specific areas - e.g., unhandled exceptions in payment flow}

**Success Criteria**:
- Zero critical security vulnerabilities
- Zero unhandled exceptions in critical paths
- All P0 tasks completed and verified

### Phase 2: Standards Compliance & Performance (Weeks 3-5)
**Goal**: Achieve standards compliance and address performance issues

**Effort**: {X days}
**Tasks**: {count} P1 tasks

**Focus Areas**:
1. Required/Recommended standards violations ({count} tasks)
2. Performance optimizations ({count} tasks)
3. High-complexity/duplication issues ({count} tasks)

**Success Criteria**:
- 95% standards compliance
- Key performance metrics improved by {X}%
- Technical debt score reduced by {Y}%

### Phase 3: Code Quality Polish (Weeks 6-8)
**Goal**: Improve code quality and maintainability

**Effort**: {X days}
**Tasks**: {count} P2 tasks (prioritized subset)

**Focus Areas**:
1. Optional standards adoption
2. Documentation improvements
3. Test coverage expansion

**Success Criteria**:
- Test coverage {current}% → {target}%
- Code quality metrics at target levels
- Documentation complete for all public APIs

---

## Critical Improvements (P0) - {count} Tasks

{For each P0 task, provide summary}

### Security Vulnerabilities ({count} tasks, {effort})

#### TASK-001-P0-security: {Title}
**File**: `{path}:{line}`
**Category**: {OWASP category or vulnerability type}
**Effort**: {estimate}
**Impact**: {description of security risk}

**Problem**: {1-2 sentence summary of vulnerability}

**Fix**: {1-2 sentence summary of solution}

**Action**: See detailed task at [`tasks/TASK-001-P0-security.md`](tasks/TASK-001-P0-security.md)

---

#### TASK-002-P0-security: {Title}
{Same structure}

---

### Reliability Risks ({count} tasks, {effort})

#### TASK-010-P0-reliability: {Title}
{Same structure as above}

---

## Important Improvements (P1) - {count} Tasks

{Group by category for easier navigation}

### Standards Compliance ({count} tasks, {effort})

{List of P1 tasks related to standards}

### Performance Optimizations ({count} tasks, {effort})

{List of P1 performance tasks}

### Maintainability Improvements ({count} tasks, {effort})

{List of P1 maintainability tasks}

---

## Nice-to-Have Improvements (P2) - {count} Tasks

{Can be more condensed, focus on categories rather than individual tasks}

### By Category

**Testing Improvements** ({count} tasks, {effort}):
- Test coverage expansion: {description}
- Edge case testing: {description}
- See tasks: {TASK-IDs}

**Code Quality** ({count} tasks, {effort}):
- Documentation: {count} tasks
- Minor refactoring: {count} tasks
- See tasks: {TASK-IDs}

**Optional Standards** ({count} tasks, {effort}):
- {Description of optional improvements}
- See tasks: {TASK-IDs}

---

## Implementation Roadmap

### Week 1: Critical Security

**Sprint Goal**: Eliminate all critical security vulnerabilities

**Tasks** ({count} tasks, {effort}):
- [ ] TASK-001: {title} ({effort}) - {assignee}
- [ ] TASK-002: {title} ({effort}) - {assignee}
- [ ] TASK-005: {title} ({effort}) - {assignee}

**Daily Breakdown**:
- **Monday**: TASK-001, TASK-002 start
- **Tuesday**: TASK-001 review, TASK-002 complete, TASK-005 start
- **Wednesday**: TASK-005 complete, security testing
- **Thursday**: Fix review feedback, regression testing
- **Friday**: Deploy to staging, verify, document

**Deliverables**:
- [ ] All security vulnerabilities patched
- [ ] Security scan shows zero critical issues
- [ ] Penetration test passed (if applicable)
- [ ] Post-mortem: how vulnerabilities were introduced, prevention strategy

**Success Metrics**:
- Critical vulnerabilities: {current} → 0
- Security score: {current} → {target}

---

### Week 2: Critical Reliability

**Sprint Goal**: Eliminate crash and data-loss risks

{Same detailed structure as Week 1}

---

### Weeks 3-5: Standards & Performance

{Can be less granular than Week 1-2, focus on weekly milestones}

**Week 3**:
- Tasks: {TASK-IDs}
- Focus: {area}
- Deliverable: {milestone}

**Week 4**:
- Tasks: {TASK-IDs}
- Focus: {area}
- Deliverable: {milestone}

**Week 5**:
- Tasks: {TASK-IDs}
- Focus: {area}
- Deliverable: {milestone}

---

### Weeks 6-8: Code Quality

{High-level overview, P2 tasks are lower priority}

---

## Quick Start Guide

### Step 1: Review and Prioritize

```bash
# Navigate to improvements directory
cd .claude/improvements/{project}

# Review executive summary
cat IMPROVEMENTS.md | less

# See all critical tasks
ls tasks/TASK-*-P0-*.md
```

**Action Items**:
- [ ] Read executive summary with team lead
- [ ] Review all P0 tasks in detail
- [ ] Adjust priorities based on business context (if needed)
- [ ] Assign tasks to team members

---

### Step 2: Set Up Tracking

**Option A: GitHub Issues**
```bash
# Create issues from tasks (script provided)
./scripts/create-github-issues.sh
```

**Option B: Jira/Linear**
```bash
# Import tasks into project management tool
# CSV export available at: tasks/tasks-export.csv
```

**Option C: Simple Spreadsheet**
```
Task ID | Title | Priority | Effort | Assignee | Status | PR Link | Completed
--------|-------|----------|--------|----------|--------|---------|----------
TASK-001 | ... | P0 | M | @alice | In Progress | #123 |
```

---

### Step 3: Start with Quick Wins

```bash
# Review quick wins
cat IMPROVEMENTS.md | grep -A 20 "## Quick Wins"

# Start with first quick win
cat tasks/TASK-{first-quick-win}.md
```

**Rationale**: Quick wins provide immediate value and build momentum.

---

### Step 4: Apply Batch Fixes

```bash
# Run automated fixes (if available)
npx eslint --fix src/**/*.ts
npx prettier --write src/**/*.{ts,tsx}

# Verify changes
git diff

# Commit batch fixes
git add .
git commit -m "Apply automated code standards fixes

- ESLint auto-fix: 28 violations
- Prettier formatting: consistent style
- See: BATCH-001 in IMPROVEMENTS.md"
```

---

### Step 5: Execute Critical Tasks

{For each P0 task}
```bash
# Read task details
cat tasks/TASK-001-P0-security.md

# Create branch
git checkout -b fix/task-001-sql-injection

# Implement fix (follow task instructions)
# ... make changes ...

# Verify fix
npm test
npm run lint

# Create PR
gh pr create --title "Fix: SQL injection in auth endpoint (TASK-001)" \
  --body "Closes TASK-001. See tasks/TASK-001-P0-security.md for details."
```

---

## Integration with Development Workflow

### With `/dev` Workflow
```bash
# Execute tasks using /dev command
/dev "Implement $(cat tasks/TASK-001-P0-security.md)"
```

### With `/proj-gen-task-prompts` Workflow
```bash
# Convert improvement requirements to detailed execution prompts
/proj-gen-task-prompts "Implement all P0 security fixes from IMPROVEMENTS.md"
```

### With CI/CD Pipeline

**Prevent Regressions**:
```yaml
# .github/workflows/standards-check.yml
name: Standards Check
on: [pull_request]
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm run lint
      - run: npm run type-check
      - run: npm run test:coverage
      # Block PR if critical rules violated
      - run: ./scripts/check-critical-rules.sh
```

**Track Progress**:
```yaml
# Weekly scheduled job to track improvement progress
name: Improvement Metrics
on:
  schedule:
    - cron: '0 0 * * 1'  # Every Monday
jobs:
  metrics:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: ./scripts/calculate-improvement-metrics.sh
      - run: ./scripts/post-to-slack.sh
```

---

## Monitoring and Metrics

### Progress Tracking

**Completion Rate**:
- P0 tasks: {completed} / {total} ({percentage}%)
- P1 tasks: {completed} / {total} ({percentage}%)
- P2 tasks: {completed} / {total} ({percentage}%)
- Overall: {completed} / {total} ({percentage}%)

**Velocity**:
- Tasks completed per week: {average}
- Estimated completion: Week {X} ({date})

### Quality Metrics

**Before Improvements**:
- Security vulnerabilities: {count}
- Test coverage: {percentage}%
- Cyclomatic complexity (avg): {value}
- Code duplication: {percentage}%
- Technical debt ratio: {value}

**After Improvements** (target):
- Security vulnerabilities: 0
- Test coverage: {target}%
- Cyclomatic complexity (avg): <{target}
- Code duplication: <{target}%
- Technical debt ratio: <{target}

**Track Progress Weekly**:
```bash
# Run metrics calculation
npm run metrics

# Compare with baseline
./scripts/compare-metrics.sh baseline.json current.json
```

---

## Statistics

### Violations by Severity
| Severity | Count | Percentage | Estimated Effort |
|----------|-------|------------|------------------|
| P0 (Critical) | {count} | {percent}% | {effort} |
| P1 (Important) | {count} | {percent}% | {effort} |
| P2 (Nice-to-have) | {count} | {percent}% | {effort} |
| **Total** | **{count}** | **100%** | **{effort}** |

### Violations by Category
| Category | Total | P0 | P1 | P2 | Effort |
|----------|-------|----|----|----| -------|
| Security | {count} | {p0} | {p1} | {p2} | {effort} |
| Reliability | {count} | {p0} | {p1} | {p2} | {effort} |
| Performance | {count} | {p0} | {p1} | {p2} | {effort} |
| Maintainability | {count} | {p0} | {p1} | {p2} | {effort} |
| Testing | {count} | {p0} | {p1} | {p2} | {effort} |
| **Total** | **{count}** | **{p0}** | **{p1}** | **{p2}** | **{effort}** |

### Violations by Rule Source
| Source | Count | Percentage |
|--------|-------|------------|
| Project Standards | {count} | {percent}% |
| Best Practices | {count} | {percent}% |
| **Total** | **{count}** | **100%** |

### Effort Distribution
| Effort | Count | Percentage | Total Time |
|--------|-------|------------|------------|
| XS (<1h) | {count} | {percent}% | {hours}h |
| S (1-4h) | {count} | {percent}% | {hours}h |
| M (4-8h) | {count} | {percent}% | {hours}h |
| L (1-2d) | {count} | {percent}% | {days}d |
| XL (>2d) | {count} | {percent}% | {days}d |
| **Total** | **{count}** | **100%** | **{effort}** |

---

## Appendix

### A. Standards Documentation
- Project Standards: [`{path to CODESTYLE.md}`]({path})
- Code Review Checklist: [`{path}`]({path})
- Tool Configurations: [`.eslintrc.js`](.eslintrc.js), [`.prettierrc`](.prettierrc)

### B. Best Practice References
- **{Language}**: [{Guide name}]({URL})
- **Security**: [OWASP Top 10]({URL}), [CWE Top 25]({URL})
- **Performance**: [{Framework} Best Practices]({URL})
- **Testing**: [{Testing Guide}]({URL})

### C. Excluded Patterns
The following patterns were intentionally excluded from violations:

1. **Legacy Code** (`src/legacy/**`):
   - Reason: {explanation}
   - Migration plan: {timeline}

2. **Test Files** (`**/__tests__/**`):
   - Reason: {explanation}
   - Exceptions: {specific rules relaxed}

3. **{Other exceptions}**:
   - Reason: {explanation}

### D. Ambiguous Cases Requiring Manual Review
{count} violations were flagged as requiring human judgment:

1. **{Description}**: See `{task file}` for details
2. **{Description}**: See `{task file}` for details

### E. Scan Metadata
- **Scan date**: {timestamp}
- **Files scanned**: {count}
- **Lines of code**: {count}
- **Directories**: {list}
- **Excluded patterns**: {patterns}
- **Tool versions**: ESLint {version}, Prettier {version}, {other tools}

---

**Document Version**: 1.0
**Last Updated**: {timestamp}
**Maintainers**: {team/person responsible}
**Feedback**: {how to provide feedback on this document}
```

## Quality Checks

Before finalizing document, verify:
- [ ] Executive summary is clear and actionable
- [ ] All P0 tasks are listed with sufficient detail
- [ ] Roadmap is realistic and achievable
- [ ] Statistics are accurate and add up correctly
- [ ] Quick wins are genuinely high-impact, low-effort
- [ ] Integration guidance is specific and actionable
- [ ] All links to task files are correct
- [ ] Monitoring metrics defined clearly

## Notes

- Keep executive summary under 2 pages for stakeholder review
- Provide both high-level view (execs) and detailed lists (developers)
- Include clear next steps—readers should know exactly what to do
- Link generously to task files for details
- Make roadmap realistic—overpromising hurts credibility
- Celebrate progress—tracking metrics motivates teams
