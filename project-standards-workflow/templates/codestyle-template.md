# {Project Name} Code Style Guide

**Version**: 1.0
**Last Updated**: {date}
**Status**: Official
**Maintained By**: {team}

---

## Table of Contents

1. [Introduction](#introduction)
2. [Naming Conventions](#naming-conventions)
3. [File Organization](#file-organization)
4. [Code Formatting](#code-formatting)
5. [Import/Export Patterns](#import-export-patterns)
6. [Comment Standards](#comment-standards)
7. [Error Handling](#error-handling)
8. [Testing Requirements](#testing-requirements)
9. [Type System Guidelines](#type-system-guidelines) *(TypeScript projects)*
10. [Security Standards](#security-standards)
11. [Performance Guidelines](#performance-guidelines) *(Optional)*
12. [Accessibility Standards](#accessibility-standards) *(Frontend projects)*
13. [Tool Configurations](#tool-configurations)
14. [Exceptions & Special Cases](#exceptions-special-cases)
15. [Enforcement Strategy](#enforcement-strategy)

---

## 1. Introduction

### Purpose
This document defines the official coding standards for {Project Name}. These standards ensure consistency, maintainability, and quality across the codebase.

### Scope
- **Applies to**: All new code and modified existing code
- **Languages**: {Primary languages}
- **Frameworks**: {Frameworks in use}
- **Enforcement**: Automated (ESLint/Prettier/etc.) + Manual (code review)

### Severity Levels
- **Required** (❗): Must follow - build fails if violated
- **Recommended** (💡): Should follow - warnings in PR
- **Optional** (ℹ️): Nice to have - team discretion

### How to Use This Guide
1. Read the relevant sections for your work
2. Follow "Good" examples, avoid "Bad" examples
3. Use automated tools to catch violations
4. Consult code review checklist before submitting PRs

---

## 2. Naming Conventions

### 2.1 Variables & Functions (❗ Required)

**Standard**: {Naming convention - e.g., camelCase}

✅ **Good**:
```{language}
{Good example with real code}
```

❌ **Bad**:
```{language}
{Bad example showing anti-pattern}
```

**Rationale**:
- {Why this standard - industry convention, readability, etc.}

**Enforcement**:
- Tool: {ESLint rule / Linter config}
- Auto-fix: {Yes/No}
- CI check: {Yes/No}

**Examples from Codebase**:
- `@{file_path}:{line}` - {Description}

---

### 2.2 Constants (❗/💡 Required/Recommended)

**Standard**: {Convention - e.g., UPPER_SNAKE_CASE}

✅ **Good**:
```{language}
{Good example}
```

❌ **Bad**:
```{language}
{Bad example}
```

**Rationale**: {Why}

**Enforcement**: {Tool and method}

---

### 2.3 Classes & Interfaces (❗ Required)

**Standard**: {Convention - e.g., PascalCase}

✅ **Good**:
```{language}
{Good example}
```

❌ **Bad**:
```{language}
{Bad example}
```

**Rationale**: {Why}

**Enforcement**: {Tool and method}

---

### 2.4 Files & Directories (❗ Required)

**Standard**: {Convention - e.g., kebab-case}

✅ **Good**:
```
{Good directory structure}
```

❌ **Bad**:
```
{Bad directory structure}
```

**Rationale**: {Why}

**Enforcement**: {Manual review / Pre-commit hook}

---

## 3. File Organization

### 3.1 Directory Structure (❗ Required)

**Standard**: {Feature-based / Type-based organization}

```
{Project directory structure}
```

**Rationale**: {Why this organization}

**Migration**: {How to transition existing code}

---

### 3.2 File Length (💡 Recommended)

**Standard**: Max {N} lines per file (excluding tests)

**Rationale**: {Readability, maintainability}

**Exceptions**:
- Complex algorithms (document why)
- Generated code
- Configuration files

**Enforcement**: {ESLint rule}

---

## 4. Code Formatting

### 4.1 Indentation (❗ Required)

**Standard**: {N spaces / tabs}

✅ **Good**:
```{language}
{Good indentation example}
```

❌ **Bad**:
```{language}
{Bad indentation example}
```

**Enforcement**:
- Prettier config: `{config}`
- EditorConfig: `{config}`
- Auto-fix: Yes

---

### 4.2 Line Length (❗ Required)

**Standard**: Max {N} characters per line

**Exceptions**:
- URLs and long strings
- Import statements
- JSDoc comments

✅ **Good**:
```{language}
{Good line wrapping example}
```

**Enforcement**:
- Prettier config: `"printWidth": {N}`
- Auto-fix: Yes

---

### 4.3 Semicolons (❗ Required)

**Standard**: {Always use / Never use} semicolons

✅ **Good**:
```{language}
{Good example}
```

❌ **Bad**:
```{language}
{Bad example}
```

**Rationale**: {Why}

**Enforcement**:
- ESLint rule: `{rule}`
- Prettier config: `{config}`
- Auto-fix: Yes

---

### 4.4 Quotes (❗ Required)

**Standard**: {Single / Double} quotes for strings

✅ **Good**:
```{language}
{Good example}
```

❌ **Bad**:
```{language}
{Bad example}
```

**Exception**: Use backticks for template literals

**Enforcement**:
- Prettier config: `{config}`
- Auto-fix: Yes

---

### 4.5 Trailing Commas (❗ Required)

**Standard**: {Use / Omit} trailing commas in multi-line structures

✅ **Good**:
```{language}
{Good example}
```

**Rationale**: {Cleaner git diffs, etc.}

**Enforcement**:
- Prettier config: `{config}`
- Auto-fix: Yes

---

## 5. Import/Export Patterns

### 5.1 Named vs Default Exports (❗/💡 Required/Recommended)

**Standard**: {Prefer named / Allow both with guidelines}

✅ **Good**:
```{language}
{Good export pattern}
```

❌ **Bad**:
```{language}
{Bad export pattern}
```

**Exception**: {When default exports allowed}

**Rationale**: {Tree-shaking, clarity, refactoring}

**Enforcement**: {ESLint rule}

---

### 5.2 Import Paths (❗ Required)

**Standard**: {Relative / Absolute with alias}

✅ **Good**:
```{language}
{Good import paths}
```

❌ **Bad**:
```{language}
{Bad import paths}
```

**Configuration**: {tsconfig.json or equivalent}

**Rationale**: {Consistency, refactoring}

**Enforcement**: {ESLint rule / Custom rule}

---

### 5.3 Import Ordering (💡 Recommended)

**Standard**: Group imports by source

```{language}
// 1. Node built-ins
// 2. External dependencies
// 3. Internal absolute imports
// 4. Relative imports
```

**Enforcement**:
- ESLint plugin: `{plugin}`
- Auto-fix: Yes

---

## 6. Comment Standards

### 6.1 JSDoc/Docstrings for Public APIs (❗ Required)

**Standard**: {When JSDoc required}

✅ **Good**:
```{language}
{Good JSDoc example}
```

❌ **Bad**:
```{language}
{Bad or missing JSDoc example}
```

**Rationale**: {IDE support, documentation generation}

**Enforcement**: {ESLint plugin}

---

### 6.2 Inline Comments (💡 Recommended)

**Standard**: Explain "why", not "what"

✅ **Good**:
```{language}
{Good inline comment}
```

❌ **Bad**:
```{language}
{Bad inline comment}
```

**Rationale**: {Code should be self-documenting}

---

### 6.3 TODO/FIXME Format (❗ Required)

**Standard**: Include owner and date

```{language}
// TODO(username, YYYY-MM-DD): Description
// FIXME(username, YYYY-MM-DD): Description
```

**Rationale**: {Tracking technical debt}

---

## 7. Error Handling

### 7.1 Async Error Handling (❗ Required)

**Standard**: All async functions must handle errors

✅ **Good**:
```{language}
{Good async error handling}
```

❌ **Bad**:
```{language}
{Missing error handling}
```

**Rationale**: {Prevent unhandled rejections}

**Enforcement**: {ESLint plugin}

---

### 7.2 Custom Error Classes (❗ Required)

**Standard**: Extend Error for domain-specific errors

✅ **Good**:
```{language}
{Custom error class example}
```

**Rationale**: {Type-safe error handling, clarity}

---

### 7.3 Error Logging (❗ Required)

**Standard**: Structured logging with context

✅ **Good**:
```{language}
{Good error logging}
```

**Required Fields**:
- Error message
- Error type/name
- Stack trace
- Context (function args, user ID, etc.)

---

## 8. Testing Requirements

### 8.1 Test File Naming (❗ Required)

**Standard**: {Co-located / Separate directory}

✅ **Good**:
```
{Good test file structure}
```

❌ **Bad**:
```
{Bad test file structure}
```

**Rationale**: {Easy to locate tests}

---

### 8.2 Test Coverage (❗ Required)

**Standard**:
- **Minimum**: {N}% overall line coverage
- **Critical paths**: {M}% coverage (auth, payment, data access)

**Enforcement**:
- Jest/Coverage tool configuration
- CI fails if coverage drops below threshold

---

### 8.3 Test Scenarios (❗ Required)

**Standard**: Every function must test

- ✅ Happy path: Normal inputs → expected outputs
- ✅ Edge cases: Boundary values, empty inputs, max limits
- ✅ Error handling: Invalid inputs, exceptions, failures
- ✅ State transitions: If stateful, all valid state changes

**Example**:
```{language}
{Good test coverage example}
```

---

## 9. Type System Guidelines *(TypeScript/Flow)*

### 9.1 Type Annotations (❗ Required)

**Standard**: Annotate parameters and return types

✅ **Good**:
```typescript
{Good type annotations}
```

❌ **Bad**:
```typescript
{Missing type annotations}
```

**Enforcement**:
- TSConfig: `{config}`
- ESLint rule: `{rule}`

---

### 9.2 Avoid any Type (❗ Required)

**Standard**: Avoid `any` unless absolutely necessary

✅ **Good**:
```typescript
{Using unknown or specific types}
```

❌ **Bad**:
```typescript
{Using any type}
```

**Exception**: {When any is acceptable - mark with TODO}

**Enforcement**: {ESLint rule}

---

## 10. Security Standards

### 10.1 Secret Management (❗ Required)

**Standard**: Never hardcode secrets

✅ **Good**:
```{language}
{Environment variable usage}
```

❌ **Bad**:
```{language}
{Hardcoded secret}
```

**Enforcement**:
- Secretlint / Pre-commit hook
- Manual code review

---

### 10.2 Input Validation (❗ Required)

**Standard**: Validate all external inputs

**Required for**:
- HTTP request parameters, query strings, body
- Database query inputs
- File uploads

**Use validation library**: {Zod / Joi / class-validator}

---

### 10.3 SQL Injection Prevention (❗ Required)

**Standard**: Always use parameterized queries

✅ **Good**:
```{language}
{Parameterized query}
```

❌ **Bad**:
```{language}
{String interpolation in query}
```

---

### 10.4 XSS Prevention (❗ Required)

**Standard**: Sanitize user-generated content before rendering

**Use**: {DOMPurify / Framework auto-escaping}

---

## 11. Performance Guidelines *(Optional)*

### 11.1 Database Queries (💡 Recommended)

**Standard**: Optimize N+1 queries

**Problem**: {N+1 query example}

**Solution**: {Join or batching example}

---

### 11.2 Bundle Size (💡 Recommended)

**Standard**: Keep bundles < {N}KB (gzipped)

**Strategies**:
- Code splitting
- Tree shaking
- Lazy loading

---

## 12. Accessibility Standards *(Frontend)*

### 12.1 Semantic HTML (❗ Required)

**Standard**: Use semantic HTML elements

✅ **Good**:
```html
{Semantic HTML}
```

❌ **Bad**:
```html
{Non-semantic HTML}
```

---

### 12.2 ARIA Labels (❗ Required)

**Standard**: Add ARIA labels for non-text elements

```html
{ARIA label example}
```

---

### 12.3 Keyboard Navigation (❗ Required)

**Standard**: All interactive elements must be keyboard accessible

**Test**: Tab through page, Enter/Space to activate

---

## 13. Tool Configurations

### 13.1 ESLint

See `.eslintrc.js` in config directory.

**Key rules**:
- {List key ESLint rules}

**Usage**:
```bash
npm run lint              # Check for violations
npm run lint:fix          # Auto-fix violations
```

---

### 13.2 Prettier

See `.prettierrc` in config directory.

**Key settings**:
- {List key Prettier settings}

**Usage**:
```bash
npm run format            # Format all files
npm run format:check      # Check formatting
```

---

### 13.3 Pre-commit Hooks

**Recommended**: Use Husky + lint-staged

```bash
npm install --save-dev husky lint-staged
```

Configuration in `package.json`:
```json
{
  "lint-staged": {
    "*.{ts,tsx,js,jsx}": [
      "eslint --fix",
      "prettier --write"
    ]
  }
}
```

---

## 14. Exceptions & Special Cases

### 14.1 Legacy Code

**Rule**: Pre-existing code not required to follow new standards immediately

**Migration Strategy**:
- **Boy Scout Rule**: Clean up code you touch
- **Incremental**: Refactor during feature work
- **Planned**: Dedicated refactoring sprints for critical areas

---

### 14.2 Third-Party Integrations

**Rule**: Follow library conventions when interfacing with external APIs

**Example**: {Framework-specific patterns}

---

### 14.3 Generated Code

**Rule**: Auto-generated code exempt from style rules

**How to handle**:
- Add to `.eslintignore`
- Don't manually edit generated files

---

## 15. Enforcement Strategy

### 15.1 Automated Enforcement (Primary)

**Tools**:
- ESLint: Syntax and style violations
- Prettier: Code formatting
- TypeScript: Type safety
- Jest: Test coverage thresholds

**CI Integration**:
```yaml
{CI workflow example}
```

**Result**: PR cannot merge if checks fail

---

### 15.2 Manual Review (Secondary)

**Code Review Checklist**: See `CODE_REVIEW_CHECKLIST.md`

**Focus areas**:
- Logic correctness
- Error handling completeness
- Test quality (not just coverage)
- Security concerns
- Performance implications

---

## Appendix

### A. Migration Guide

For existing code not following these standards, see `STYLE-MIGRATION.md`

### B. FAQs

**Q: What if Prettier and ESLint conflict?**
A: Prettier takes precedence for formatting. Use `eslint-config-prettier` to disable conflicting ESLint rules.

**Q: Can I disable a rule in special cases?**
A: Yes, with inline comments and justification:
```{language}
{ESLint disable example with comment}
```

**Q: How to propose changes to this guide?**
A: Open a PR to this document with rationale and team discussion.

---

**Document Version**: 1.0
**Last Updated**: {date}
**Next Review**: Quarterly
**Contact**: {team}@{company}.com
