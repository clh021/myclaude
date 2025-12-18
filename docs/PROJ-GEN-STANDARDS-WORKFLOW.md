# Project Standards Workflow Guide

## Overview

The Project Standards Workflow extracts coding patterns from your existing codebase, generates comprehensive standards documentation, reviews against industry best practices, and provides prioritized improvement recommendations with actionable migration roadmaps.

## Key Differences from Other Workflows

| Workflow | Purpose | Output | Best For |
|----------|---------|--------|-------------|
| `/dev` | Immediate implementation | Working code | Quick feature development |
| `/proj-gen-task-prompts` | Prompt generation | AI-executable prompts | Distributed execution |
| **`/proj-gen-standards`** | **Standards extraction** | **CODESTYLE.md + migration plan** | **Team consistency, onboarding** |

## When to Use

### Ideal Scenarios
- **New team members**: Need documented standards for onboarding
- **Code review friction**: Debates over style in PRs consume time
- **Technical debt**: Mixed coding styles across legacy code
- **Pre-refactoring**: Establish baseline before major changes
- **Compliance**: Enterprise clients requiring formal standards

### Not Recommended For
- Brand new projects (no existing code to analyze)
- Tiny projects (<1000 LOC, overhead not justified)
- Solo projects (less critical for one person)

## Command Usage

```bash
# Basic usage - analyze current directory
/proj-gen-standards

# Specify project name
/proj-gen-standards --project-name="Authentication Service"

# Skip best practices review (faster, extract and document only)
/proj-gen-standards --skip-review

# Quick mode (skip detailed examples)
/proj-gen-standards --quick

# Output location
.claude/standards/{project_name}/
```

## Workflow Steps

### Step 1: Project Context & Setup (2-3 minutes)
**Action**: Auto-detect project information

**Detection Logic**:
- **Project name**: package.json `name` → git remote → directory name
- **Language**: File extension count (.ts > .js → TypeScript)
- **Framework**: package.json dependencies (react, vue, express, etc.)

**Output**: Project context metadata + output directory created

---

### Step 2: Codebase Deep Scan (15-20 minutes)
**Action**: Invokes `codeagent` skill in exploration mode

**Analysis Dimensions**:

1. **Naming Conventions**: Variables (camelCase/snake_case), files (kebab-case/PascalCase), classes (PascalCase), constants (UPPER_SNAKE_CASE) - statistical breakdown with percentages

2. **Code Structure**: Directory organization (feature-based vs type-based), file size distribution, function length/complexity

3. **Formatting & Style**: Indentation (tabs/spaces, 2/4-space), line length distribution, semicolons (usage %), quotes (single/double), trailing commas

4. **Comment Patterns**: JSDoc/docstring coverage %, comment density, TODO/FIXME count, file headers

5. **Error Handling**: Try-catch coverage for async functions, custom error classes usage, error logging patterns

6. **Testing Conventions**: Test file naming (.test/.spec), co-location vs separate, coverage %, test quality

7. **Type System** *(TypeScript)*: Type annotation coverage, `any` usage %, strictness settings

8. **Dependency Management**: Import paths (relative vs absolute), unused imports, circular dependencies

**Output**: `./.claude/standards/{project}/01-codebase-scan.md` with exact percentages and code examples (file:line format)

---

### Step 3: Pattern Extraction (10-15 minutes)
**Action**: Invokes `style-extractor` agent

**Classification Logic**:
- **High Confidence** (≥80% consistency): Auto-adopt as official standard
  - Example: "92% of variables use camelCase" → Adopt camelCase as required
- **Medium Confidence** (50-80%): User confirmation needed
  - Example: "65% single quotes, 35% double quotes" → Ask user to choose
- **Low Confidence/Conflict** (<50%): Decision required
  - Example: "52% named exports, 48% default exports" → Present options with pros/cons

**Deliverables**:
- High-confidence patterns (auto-adopt candidates)
- Medium-confidence patterns (with recommendations)
- Conflicting patterns (with industry standard comparison)
- Quick wins (≥90% consistent, just needs documentation)
- Problem areas (<50% consistent, needs standardization)
- Pattern summary table

**Output**: `./.claude/standards/{project}/02-extracted-patterns.md`

---

### Step 4: User Confirmation - Round 1 (Interactive)
**Action**: Uses `AskUserQuestion` for three confirmation types

#### 4.1 High Confidence Patterns
```
Question: "Detected 6 high-consistency patterns (≥80% usage). Adopt as official standards?"

Patterns preview:
- Variable naming: camelCase (92%)
- Indentation: 2 spaces (88%)
- File naming: kebab-case (96%)
- Test naming: .test.ts co-located (94%)
- Semicolons: Always required (85%)
- Async/await: Preferred over Promises (89%)

Options:
- Adopt all (Recommended)
- Review individually
- Customize
```

#### 4.2 Conflicting Patterns
For each conflict (e.g., named vs default exports):
```
Question: "Export pattern conflict - Named: 52% vs Default: 48%. Choose standard?"

Context:
- Named exports: Better tree-shaking, explicit imports, easier refactoring
- Default exports: Simpler for single exports, shorter syntax
- Industry: Airbnb and Google prefer named exports

Options:
- Named exports only (Recommended)
- Default exports only
- Mixed (context-dependent - allow both with guidelines)
```

#### 4.3 Custom Standards
```
Question: "Add standards not detected automatically?"

Options:
- Security rules (no eval, no hardcoded secrets)
- Performance rules (optimize N+1 queries, bundle size limits)
- Team conventions (Git commit format, PR templates)
- Skip
```

---

### Step 5: Standards Documentation (5-10 minutes)
**Action**: Invokes `standards-documenter` agent

**Generated Documents**:

1. **CODESTYLE.md** - Main style guide with:
   - 15 sections covering all coding aspects
   - Each rule has: severity (❗ Required / 💡 Recommended / ℹ️ Optional), good/bad examples, rationale, enforcement method
   - Code examples from actual codebase (file:line references)
   - Tool configurations referenced

2. **config/.eslintrc.js** - ESLint configuration enforcing standards

3. **config/.prettierrc** - Prettier configuration for formatting

4. **config/tsconfig.strict.json** - TypeScript strict configuration *(if TypeScript)*

5. **config/.editorconfig** - Editor configuration (indentation, line endings)

6. **CODE_REVIEW_CHECKLIST.md** - Manual review guidelines for things tools can't check

**Output**: `./.claude/standards/{project}/CODESTYLE.md` + configs

---

### Step 6: Best Practices Review (15-20 minutes) *Optional*
**Action**: Invokes `best-practices-reviewer` agent

**Skipped if**: `--skip-review` flag provided

**Industry Comparison**:
- **JavaScript/TypeScript**: Airbnb, Google, StandardJS, TypeScript Handbook
- **Python**: PEP 8, Black, Google Python Guide
- **Java**: Google Java Style, Oracle Code Conventions
- **Go**: Effective Go, Uber Go Style Guide

**Analysis**:
1. **Alignment Score**: Calculate alignment with industry standards (0-10 scale)
2. **Over-Strict Rules**: Identify rules that block flexibility
   - Example: "Named exports only" blocks React component idioms
3. **Under-Specified Rules**: Identify vague rules allowing inconsistency
   - Example: "Handle errors appropriately" needs concrete error classes and logging format
4. **Modern Practice Conflicts**: Check for conflicts with ES6+, async/await, optional chaining
5. **Maintainability Risks**: Flag standards that hinder future development
6. **Strictness Balance Matrix**: Evaluate appropriate enforcement level per standard

**Deliverables**:
- Industry alignment score (e.g., "7.8/10 - Grade B+")
- Over-strict rules with impact assessment and relaxation recommendations
- Under-specified rules with concrete improvement suggestions
- Missing critical standards (security, performance, accessibility)
- Strictness balance matrix (Required/Recommended/Optional per standard)

**Output**: `./.claude/standards/{project}/03-standards-review.md`

---

### Step 7: Improvement Recommendations (10-15 minutes)
**Action**: Invokes `standards-improver` agent

**Prioritization**:
- **P0 (Critical)**: Must fix - Security, reliability risks
  - Example: Add error handling (43% → 100% coverage)
- **P1 (Important)**: Should fix - Consistency, maintainability
  - Example: Relax named exports rule (too strict)
- **P2 (Nice-to-have)**: Could fix - Quality of life
  - Example: Add performance guidelines

**For Each Improvement**:
1. **Current vs Target State**: Metrics (before → after)
2. **Impact Analysis**: Reliability, maintainability, velocity improvement
3. **Effort Estimation**: Days, files affected, complexity, risk
4. **ROI Calculation**: Benefits / Cost (e.g., "9:1 ROI, payback 1.3 months")
5. **Implementation Strategy**: Day-by-day breakdown for complex changes
6. **Success Metrics**: Measurable indicators (test coverage %, error rate, etc.)

**Migration Roadmap**:
- **Phase 1** (Weeks 1-2): P0 improvements
- **Phase 2** (Weeks 3-5): P1 improvements
- **Phase 3** (Weeks 6-8): P2 improvements (optional)
- Week-by-week timeline with milestones

**Output**: `./.claude/standards/{project}/04-improvement-recommendations.md`

---

### Step 8: User Confirmation - Round 2 & Final Output (Interactive)
**Action**: Review improvements and generate final artifacts

#### 8.1 Present Improvement Summary
```
Improvement Summary:
- Critical (P0): 2 items (error handling, security)
- Important (P1): 3 items (relax named exports, reduce JSDoc scope, file size guideline)
- Nice-to-have (P2): 2 items (performance guidelines, accessibility)

Estimated effort: 15 days over 8 weeks
Expected ROI: 5.6:1
Payback: 2 months

View detailed recommendations?
```

#### 8.2 Per-Improvement Confirmation
For each P0 and P1:
```
Question: "Improvement: Standardize Error Handling"

Current: 43% of async functions lack try-catch
Target: 100% error handling coverage
Impact: High (prevents production crashes)
Effort: Medium (5 days, ~150 functions)
ROI: 9:1

Options:
- Adopt and add to standards (Recommended)
- Adopt but defer implementation
- Reject
- Modify
```

#### 8.3 Generate Final Artifacts

Based on user selections:

1. **CODESTYLE.md** - Updated with adopted improvements
2. **STYLE-MIGRATION.md** - Week-by-week migration roadmap with:
   - Day-by-day tasks for each improvement
   - File-level migration checklist
   - Success metrics and validation
   - Risk mitigation strategies
   - Rollback procedures
3. **CODE_REVIEW_CHECKLIST.md** - Reviewer guidelines
4. **Tool configs** - ESLint, Prettier, TSConfig with strict rules

#### 8.4 Display Completion Summary
```
✅ Project Standards Generated

Output Location: ./.claude/standards/{project_name}/

Key Artifacts:
- CODESTYLE.md: Official coding standards (47 rules)
- STYLE-MIGRATION.md: 8-week migration plan (12 improvements)
- CODE_REVIEW_CHECKLIST.md: Reviewer guidelines
- Tool configs: ESLint, Prettier, TSConfig (ready to use)

Next Steps:
1. Review CODESTYLE.md with team
2. Copy config files to project root:
   cp ./.claude/standards/{project}/config/.eslintrc.js .
   cp ./.claude/standards/{project}/config/.prettierrc .
3. Integrate ESLint/Prettier into CI/CD
4. Start migration following STYLE-MIGRATION.md

Quick Start:
  npm install --save-dev eslint prettier
  npm run lint
  npm run format
```

---

## Output Directory Structure

```
.claude/standards/{project_name}/
├── 01-codebase-scan.md              # Statistical analysis
├── 02-extracted-patterns.md         # Classified patterns
├── 03-standards-review.md           # Industry comparison
├── 04-improvement-recommendations.md # Prioritized improvements
├── CODESTYLE.md                     # ✅ Official standards
├── STYLE-MIGRATION.md               # ✅ Migration roadmap
├── CODE_REVIEW_CHECKLIST.md        # ✅ Review guidelines
└── config/
    ├── .eslintrc.js                 # ✅ ESLint config
    ├── .prettierrc                  # ✅ Prettier config
    ├── tsconfig.strict.json         # ✅ TypeScript config
    └── .editorconfig                # ✅ Editor config
```

## Usage Examples

### Example 1: JavaScript/TypeScript Project

```bash
/proj-gen-standards
```

**Scan Results**:
- 2,847 variables analyzed: 92% camelCase, 8% snake_case/PascalCase
- 1,245 files analyzed: 88% 2-space indent, 12% 4-space or tabs
- 8,423 strings: 65% single quotes, 35% double quotes
- 423 modules: 52% named exports, 48% default exports

**Extracted Patterns**:
- ✅ Auto-adopt: camelCase variables, 2-space indent, kebab-case files
- ⚠️ Confirm: Single quotes (65% vs 35%)
- ❌ Decide: Named exports (52% vs 48%)

**Review Findings**:
- Over-strict: Named exports only (blocks React components)
- Under-specified: Error handling (no structured logging format)
- Missing: Security standards (hardcoded secrets in 8 files)

**Improvements**:
- P0-1: Add error handling (43% → 100%, ROI 9:1, 5 days)
- P0-2: Add security standards (0 → comprehensive, ROI ∞, 3 days)
- P0-3: Improve test coverage (58% → 80%, ROI 9:1, 8 days)
- P1-1: Relax named exports (allow default for React, 0.5 days)
- P1-2: Reduce JSDoc scope (100% → complex only, 1 day)

**Migration**: 8 weeks, 15 days effort, ROI 5.6:1

---

### Example 2: Python Project

```bash
/proj-gen-standards --project-name="Analytics API"
```

**Scan Results**:
- 1,523 variables: 87% snake_case (PEP 8 aligned)
- 456 files: 95% 4-space indent (PEP 8 aligned)
- Docstrings: Only 41% of functions documented

**Review Findings**:
- ✅ Strong PEP 8 alignment (8.5/10 score)
- Under-specified: Docstring requirements vague
- Missing: Type hints standards (no mypy config)

**Improvements**:
- P0: Increase docstring coverage (41% → 80%, public APIs)
- P1: Add type hints (optional → required for functions)
- P2: Add Black formatter (auto-format on commit)

**Migration**: 6 weeks, 12 days effort

---

## CODESTYLE.md Structure

Each standard includes:

### 1. Naming Conventions
- Variables & functions (camelCase vs snake_case)
- Constants (UPPER_SNAKE_CASE)
- Classes & interfaces (PascalCase)
- Files & directories (kebab-case)

### 2. File Organization
- Directory structure (feature-based vs type-based)
- File length guidelines (300 lines)
- Module organization

### 3. Code Formatting
- Indentation (tabs/spaces, 2/4-space)
- Line length (80/100 chars)
- Semicolons (always/never)
- Quotes (single/double)
- Trailing commas

### 4. Import/Export Patterns
- Named vs default exports
- Import paths (relative vs absolute with @/ alias)
- Import ordering

### 5. Comment Standards
- JSDoc/docstrings for public APIs
- Inline comments (explain "why" not "what")
- TODO/FIXME format

### 6. Error Handling
- Async error handling (try-catch coverage)
- Custom error classes
- Error logging format

### 7. Testing Requirements
- Test file naming (.test.ts co-located)
- Coverage requirements (80% overall, 90% critical paths)
- Test scenarios (happy path, edge cases, errors)

### 8. Type System Guidelines *(TypeScript)*
- Type annotations (parameters, return types)
- Avoid `any` type
- Strictness settings

### 9. Security Standards
- Secret management (no hardcoded secrets)
- Input validation (all external inputs)
- SQL injection prevention (parameterized queries)
- XSS prevention (sanitize user content)

### 10. Performance Guidelines *(Optional)*
- Database query optimization (N+1 queries)
- Bundle size limits
- Caching strategies

### 11. Accessibility Standards *(Frontend)*
- Semantic HTML
- ARIA labels
- Keyboard navigation

### 12-15. Tool Configurations, Exceptions, Enforcement

---

## STYLE-MIGRATION.md Structure

### Phase 1: Critical Fixes (Weeks 1-2)
- P0 improvements with day-by-day breakdown
- File-level migration checklist
- Validation steps per day
- Rollback procedures

### Phase 2: Quality Improvements (Weeks 3-5)
- P1 improvements with timeline
- Incremental rollout strategy

### Phase 3: Polish (Weeks 6-8)
- P2 improvements (optional)
- Final review and team training

### Tool Automation
- Automated migrations (Prettier, ESLint --fix)
- Semi-automated (IDE refactoring, TypeScript quick fixes)
- Manual migrations (error handling patterns, validation)

### Risk Management
- Risks identified with mitigation strategies
- Monitoring metrics (test coverage, error rate)
- Contingency plans (rollback, pause migration)

### Success Metrics
- Quantitative: Test coverage, error rate, code review time
- Qualitative: Team feedback surveys

---

## Quality Assurance

### Validation Checklist
- [ ] All patterns have exact percentages (not estimates)
- [ ] Code examples include file paths and line numbers
- [ ] Industry standards cited for major decisions
- [ ] Migration effort assessed realistically
- [ ] User decision points clearly marked
- [ ] Quick wins identified and prioritized
- [ ] Problem areas flagged with impact analysis
- [ ] ROI calculated for improvements

### Success Metrics
- **Generation Speed**: <60 minutes total (excluding user interaction)
- **Quality Score**: ≥95% on validation checklist
- **Team Adoption**: ≥80% of team agrees standards are clear and helpful
- **Revision Rate**: <10% of standards require post-adoption adjustment

---

## Troubleshooting

### Issue: No Clear Patterns (Low Consistency)

**Symptoms**: Most patterns <50% consistency

**Cause**: Legacy codebase with mixed styles over time

**Solution**:
1. Accept low consistency as baseline
2. Choose industry standards as target
3. Plan longer migration (12+ weeks)
4. Focus on critical paths first (auth, payment)
5. Accept partial consistency temporarily

---

### Issue: Conflicting Team Preferences

**Symptoms**: Team debates extracted patterns

**Cause**: Different prior experiences or preferences

**Solution**:
1. Vote (majority wins)
2. Use industry standards as tiebreaker
3. Trial period (revisit in 3 months)
4. Document dissenting opinions in exceptions section
5. Allow escape hatch (eslint-disable with justification)

---

### Issue: Migration Timeline Too Long

**Symptoms**: Estimated 12+ weeks

**Cause**: Large codebase or many P0 improvements

**Solution**:
1. Reduce scope: Focus on P0 only (defer P1/P2)
2. Partial migration: Critical paths first, defer non-critical
3. Increase team allocation: Dedicate 20% time to standards
4. Accept longer timeline: Standards work is investment

---

### Issue: Tools Can't Auto-Fix

**Symptoms**: Many manual changes required

**Cause**: Complex patterns (error handling, validation)

**Solution**:
1. Create helper utilities (error handler wrapper, validation decorators)
2. Share migration examples (show don't tell)
3. Pair programming sessions (experienced devs help others)
4. Code review focus (emphasize pattern compliance)

---

## Integration Points

### With `/dev` Workflow
- Use CODESTYLE.md as reference during /dev implementation
- Follow CODE_REVIEW_CHECKLIST.md before marking tasks complete

### With `/proj-gen-task-prompts` Workflow
- Include CODESTYLE.md link in generated prompts
- AI follows project standards during code generation

### With CI/CD
- Integrate generated ESLint/Prettier configs
- Add pre-commit hooks (Husky + lint-staged)
- Block PRs failing lint/format checks

### With Onboarding
- Share CODESTYLE.md with new team members
- Onboarding checklist includes "Read and understand CODESTYLE.md"
- Tool setup includes ESLint/Prettier installation

---

## Best Practices

1. **Run on mature codebases**: Wait until ≥5000 LOC for meaningful analysis
2. **Involve team early**: Share extracted patterns before finalizing
3. **Balance strictness**: Use "Recommended" over "Required" when unsure
4. **Automate enforcement**: Prefer tools over manual review
5. **Incremental migration**: Follow week-by-week roadmap, don't rush
6. **Monitor metrics**: Track test coverage, error rates, code review time
7. **Review quarterly**: Standards evolve with project maturity
8. **Escape hatch**: Allow eslint-disable with justification for exceptions
9. **Document rationale**: Every standard explains "why", not just "what"
10. **Celebrate progress**: Acknowledge team effort in improving standards

---

## See Also

- [Project Standards Workflow Module README](../proj-gen-standards-workflow/README.md)
- [Style Extractor Agent](../proj-gen-standards-workflow/agents/style-extractor.md)
- [Standards Documenter Agent](../proj-gen-standards-workflow/agents/standards-documenter.md)
- [Best Practices Reviewer Agent](../proj-gen-standards-workflow/agents/best-practices-reviewer.md)
- [Standards Improver Agent](../proj-gen-standards-workflow/agents/standards-improver.md)
- [CODESTYLE Template](../proj-gen-standards-workflow/templates/codestyle-template.md)
- [Migration Roadmap Template](../proj-gen-standards-workflow/templates/migration-template.md)

---

**Version**: 1.0
**Last Updated**: 2025-01-18
**Status**: Production Ready
