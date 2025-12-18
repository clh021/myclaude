# Project Standards Workflow

Extract and standardize code style from existing codebase with best practices review and improvement recommendations.

## Overview

The Project Standards Workflow analyzes your codebase to extract coding patterns, generates comprehensive standards documentation, reviews against industry best practices, and provides prioritized improvement recommendations with migration roadmaps.

## Key Features

- **Data-Driven Analysis**: Statistical analysis of existing code patterns (≥80% consistency auto-adopted)
- **Industry Benchmarking**: Compare against Airbnb, Google, PEP 8, and other recognized standards
- **Two-Stage Confirmation**: Pattern validation + improvement approval with user control
- **Balanced Strictness**: Identifies over-strict rules (blocking flexibility) and under-specified rules (allowing inconsistency)
- **Automation First**: Generates ESLint, Prettier, TSConfig with automated enforcement
- **Incremental Migration**: Week-by-week roadmap, not big-bang refactoring

## When to Use

### Ideal Scenarios
- **New team onboarding**: Document implicit standards for new developers
- **Code review friction**: Inconsistent standards causing debate in PRs
- **Technical debt**: Legacy code with mixed styles needing standardization
- **Pre-refactoring**: Establish baseline before major architectural changes
- **Compliance needs**: Enterprise clients requiring documented standards

### Not Recommended For
- Brand new projects with no existing code (use industry templates instead)
- Projects with <1000 lines of code (overhead not justified)
- One-person projects (standards less critical)

## Usage

```bash
# Basic usage - analyze current directory
/proj-gen-standards

# Specify project name
/proj-gen-standards --project-name="My Project"

# Skip best practices review (faster, only extract and document)
/proj-gen-standards --skip-review

# Quick mode (skip detailed examples, faster execution)
/proj-gen-standards --quick
```

## Workflow Steps

### Step 1: Project Context & Setup (2-3 minutes)
- Auto-detect project name, primary language, framework
- Create output directory: `./.claude/standards/{project_name}/`

### Step 2: Codebase Deep Scan (15-20 minutes)
- Statistical analysis across 8 dimensions:
  - Naming conventions (variables, files, classes)
  - Code structure (organization, file size, complexity)
  - Formatting (indentation, line length, whitespace)
  - Comment patterns (JSDoc coverage, quality)
  - Error handling (try-catch coverage, custom errors)
  - Testing conventions (file naming, coverage, quality)
  - Type system (annotations, `any` usage, strictness)
  - Dependency management (import patterns, circular deps)
- Output: `./.claude/standards/{project}/01-codebase-scan.md`

### Step 3: Pattern Extraction (10-15 minutes)
- Classify patterns by consistency:
  - **High confidence** (≥80%): Auto-adopt as official standard
  - **Medium confidence** (50-80%): Mark for user confirmation
  - **Low confidence/Conflict** (<50%): Flag as needing decision
- Identify quick wins vs complex migrations
- Output: `./.claude/standards/{project}/02-extracted-patterns.md`

### Step 4: User Confirmation - Round 1 (Interactive)
- Validate high-confidence patterns (approve/customize)
- Resolve conflicting patterns (choose A vs B vs Mixed)
- Add custom standards not detected automatically

### Step 5: Standards Documentation (5-10 minutes)
- Generate comprehensive CODESTYLE.md:
  - Each rule has: severity, good/bad examples, rationale, enforcement
  - 15 sections covering all coding aspects
- Generate tool configs: `.eslintrc.js`, `.prettierrc`, `tsconfig.strict.json`
- Generate CODE_REVIEW_CHECKLIST.md
- Output: `./.claude/standards/{project}/CODESTYLE.md` + configs

### Step 6: Best Practices Review (15-20 minutes) *Optional*
- Compare against industry standards (Airbnb, Google, PEP 8, etc.)
- Identify over-strict rules (blocking flexibility)
- Identify under-specified rules (allowing inconsistency)
- Check for conflicts with modern practices (ES6+, async/await)
- Evaluate maintainability risks
- Generate strictness balance matrix
- Output: `./.claude/standards/{project}/03-standards-review.md`

### Step 7: Improvement Recommendations (10-15 minutes)
- Prioritize improvements: P0 (Critical) / P1 (Important) / P2 (Nice-to-have)
- For each improvement:
  - Impact assessment (reliability, maintainability, velocity)
  - Effort estimation (days, files affected)
  - ROI calculation (impact / cost)
  - Migration path (week-by-week)
- Generate migration roadmap with timeline
- Output: `./.claude/standards/{project}/04-improvement-recommendations.md`

### Step 8: User Confirmation - Round 2 & Final Output (Interactive)
- Review improvement summary (P0/P1/P2 breakdown)
- Per-improvement confirmation (adopt / defer / reject / modify)
- Generate final artifacts:
  - Updated CODESTYLE.md with adopted improvements
  - STYLE-MIGRATION.md with week-by-week roadmap
  - CODE_REVIEW_CHECKLIST.md for reviewers
  - Tool configs with strict rules

## Output Structure

```
./.claude/standards/{project_name}/
├── 01-codebase-scan.md              # Raw analysis data
├── 02-extracted-patterns.md         # Extracted patterns
├── 03-standards-review.md           # Best practices review
├── 04-improvement-recommendations.md # Prioritized improvements
├── CODESTYLE.md                     # ✅ Official standards document
├── STYLE-MIGRATION.md               # ✅ Migration roadmap
├── CODE_REVIEW_CHECKLIST.md        # ✅ Review checklist
└── config/
    ├── .eslintrc.js                 # ✅ ESLint configuration
    ├── .prettierrc                  # ✅ Prettier configuration
    ├── tsconfig.strict.json         # ✅ Strict TypeScript config
    └── .editorconfig                # ✅ Editor configuration
```

## Agents

### style-extractor
- **Purpose**: Extract patterns from codebase scan and classify by consistency
- **Input**: `01-codebase-scan.md`
- **Output**: `02-extracted-patterns.md`
- **Classification**:
  - High confidence (≥80%): Auto-adopt candidates
  - Medium confidence (50-80%): User confirmation needed
  - Conflicts (<50%): Decision required

### standards-documenter
- **Purpose**: Transform patterns into comprehensive CODESTYLE.md
- **Input**: `02-extracted-patterns.md` + user confirmations
- **Output**: `CODESTYLE.md`, `.eslintrc.js`, `.prettierrc`, `tsconfig.strict.json`
- **Structure**: 15 sections with rules, examples, rationale, enforcement

### best-practices-reviewer
- **Purpose**: Compare standards against industry best practices
- **Input**: `CODESTYLE.md`, `02-extracted-patterns.md`
- **Output**: `03-standards-review.md`
- **Analysis**:
  - Industry alignment score (compare to Airbnb, Google, etc.)
  - Over-strict rules identification
  - Under-specified rules identification
  - Maintainability risk assessment
  - Strictness balance matrix

### standards-improver
- **Purpose**: Generate prioritized improvement recommendations
- **Input**: `03-standards-review.md`, `02-extracted-patterns.md`, `CODESTYLE.md`
- **Output**: `04-improvement-recommendations.md`, `STYLE-MIGRATION.md`
- **Deliverables**:
  - P0/P1/P2 prioritized improvements
  - ROI analysis (benefits / cost)
  - Week-by-week migration roadmap
  - Success metrics and monitoring

## Examples

### Example 1: JavaScript/TypeScript Project

```bash
/proj-gen-standards
```

**Analysis Results**:
- Variable naming: 92% camelCase (✅ auto-adopt)
- Indentation: 88% 2-space (✅ auto-adopt)
- String quotes: 65% single quotes (⚠️ confirm with user)
- Export pattern: 52% named vs 48% default (⚠️ user decision)

**Review Findings**:
- Named exports rule too strict (blocks React components)
- JSDoc 100% coverage over-strict (reduce to complex functions)
- Missing security standards (add secret management, input validation)

**Improvements**:
- P0: Add error handling (43% → 100% coverage, ROI 9:1)
- P0: Add security standards (0 → comprehensive, ROI ∞)
- P1: Relax named exports (allow default for React)
- P1: Reduce JSDoc scope (100% → complex functions only)

**Migration Timeline**: 8 weeks, 15 days effort, ROI 5.6:1

---

### Example 2: Python Project

```bash
/proj-gen-standards --project-name="Python API"
```

**Analysis Results**:
- Variable naming: 87% snake_case (✅ auto-adopt)
- Indentation: 95% 4-space (✅ auto-adopt)
- Import ordering: 68% absolute imports (⚠️ confirm)
- Docstrings: 41% coverage (❌ low, needs standardization)

**Review Findings**:
- Aligns well with PEP 8 (8.5/10 score)
- Docstring requirement under-specified (add concrete examples)
- Missing type hints standards (add mypy configuration)

**Improvements**:
- P0: Increase docstring coverage (41% → 80%, public APIs)
- P1: Add type hints (optional → required for functions)
- P2: Add Black formatter (auto-format on commit)

**Migration Timeline**: 6 weeks, 12 days effort

## Quality Standards

### Pattern Extraction Quality
- ✅ All patterns have statistical support (percentages)
- ✅ High-confidence patterns have ≥80% consistency
- ✅ Conflicts identified with percentage breakdown
- ✅ Examples include file paths and line numbers
- ✅ Quick wins (already consistent) highlighted

### Documentation Quality
- ✅ Each rule has clear rationale
- ✅ Good and bad examples provided
- ✅ Enforcement method specified (automated or manual)
- ✅ Tool configurations are syntactically valid
- ✅ Exceptions and special cases documented

### Review Quality
- ✅ Compared against ≥2 industry standards
- ✅ Over-strict and under-specified rules identified
- ✅ Maintainability risks flagged
- ✅ Strictness balance evaluated

### Improvement Quality
- ✅ Prioritization based on impact and cost
- ✅ ROI calculated for each improvement
- ✅ Migration path is incremental (no big-bang)
- ✅ Tool automation opportunities identified
- ✅ Success metrics defined

## Integration with Other Workflows

Generated standards can be used in:
1. **Code reviews**: Use CODE_REVIEW_CHECKLIST.md
2. **New feature development**: Reference CODESTYLE.md
3. **Refactoring**: Follow STYLE-MIGRATION.md roadmap
4. **CI/CD**: Integrate generated ESLint/Prettier configs
5. **Onboarding**: Share CODESTYLE.md with new team members

## Best Practices

1. **Run on mature codebases**: Wait until ≥5000 LOC for meaningful patterns
2. **Involve the team**: Share extracted patterns before finalizing
3. **Balance strictness**: Use "recommended" over "required" when unsure
4. **Automate enforcement**: Use tools (ESLint, Prettier) over manual review
5. **Incremental migration**: Follow the week-by-week roadmap, don't rush
6. **Monitor metrics**: Track test coverage, error rates, code review time
7. **Review quarterly**: Standards should evolve with project maturity

## Troubleshooting

### Issue: No clear patterns detected (low consistency)

**Symptoms**: Most patterns <50% consistency

**Solution**:
1. This indicates the codebase lacks consistency (common in legacy projects)
2. Accept this as the starting point
3. Choose industry standards as baseline
4. Plan more aggressive migration (will take longer)

---

### Issue: Conflicting team preferences

**Symptoms**: Team disagrees on extracted patterns

**Solution**:
1. Use voting system (majority wins)
2. Favor industry standards as tiebreaker
3. Document dissenting opinions
4. Allow trial period (revisit in 3 months)

---

### Issue: Migration timeline too long

**Symptoms**: Estimated 12+ weeks for migration

**Solution**:
1. Focus on P0 only (defer P1/P2)
2. Reduce scope (migrate critical paths first)
3. Accept partial consistency temporarily
4. Increase team allocation

## See Also

- [Project Standards Workflow Guide](../../docs/PROJ-GEN-STANDARDS-WORKFLOW.md)
- [Style Extractor Agent](./agents/style-extractor.md)
- [Standards Documenter Agent](./agents/standards-documenter.md)
- [Best Practices Reviewer Agent](./agents/best-practices-reviewer.md)
- [Standards Improver Agent](./agents/standards-improver.md)
- [CODESTYLE Template](./templates/codestyle-template.md)
- [Migration Roadmap Template](./templates/migration-template.md)

---

**Version**: 1.0
**Last Updated**: 2025-01-18
**Status**: Production Ready
