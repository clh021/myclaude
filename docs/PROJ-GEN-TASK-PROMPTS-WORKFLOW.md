# Project Task Prompts Workflow Guide

## Overview

The Project Task Prompts Workflow transforms requirements into AI-executable task prompts with unambiguous technical roadmaps. This workflow is optimized for creating self-contained documentation that can be distributed to multiple AI agents, stored as reusable templates, or used as technical specifications.

## Key Differences from Other Workflows

| Workflow | Purpose | Output | Best For |
|----------|---------|--------|----------|
| `/dev` | Immediate implementation | Working code | Quick feature development |
| `/bmad-pilot` | Project management | Sprint plans | Large team coordination |
| `/requirements-pilot` | Spec-to-code | Technical specs → Code | Prototyping |
| **`/proj-gen-task-prompts`** | **Prompt generation** | **AI-executable prompts** | **Distributed execution, templates** |

## When to Use

### Ideal Scenarios
- **Multi-agent coordination**: Need to distribute work across multiple AI agents
- **Template library**: Building reusable implementation patterns
- **Detailed planning**: Want comprehensive technical documentation before coding
- **Asynchronous execution**: Tasks will be executed at different times

### Not Recommended For
- Quick fixes or hotfixes (use `/code` or `/debug`)
- Immediate implementation needs (use `/dev`)
- Full sprint planning (use `/bmad-pilot`)

## Command Usage

```bash
# Basic usage
/proj-gen-task-prompts "Implement JWT authentication with refresh tokens"

# Output location
.claude/specs/jwt-authentication/tasks/
```

## Workflow Steps

### Step 1: Requirement Clarification (5-10 minutes)
**Action**: Uses `AskUserQuestion` to resolve ambiguities

**Topics**:
- Functional scope and boundaries
- Performance/security requirements
- Integration points and constraints
- Test coverage expectations (default ≥90%)

**Output**: `.claude/specs/{feature}/00-requirements.md`

### Step 2: Codebase Deep Analysis (15-20 minutes)
**Action**: Invokes `codeagent` skill in exploration mode

**Analysis Scope**:
- Architecture patterns and design principles
- Similar implementations with file paths/line numbers
- Technology stack (frameworks, libraries, versions)
- Integration points (APIs, databases, services)
- Code conventions and error handling patterns

**Output**: `.claude/specs/{feature}/01-codebase-analysis.md`

### Step 3: Technical Solution Design (20-30 minutes)
**Action**: Invokes `prompt-architect` agent

**Deliverables**:
- Technology selections with version numbers and rationale
- API/interface definitions with type signatures
- Data structure specifications with constraints
- Implementation strategy in 3 phases
- Error handling blueprints
- Human decision points marked

**Output**: `.claude/specs/{feature}/02-technical-design.md`

### Step 4: Task Breakdown (15-20 minutes)
**Action**: Invokes `task-organizer` agent

**Deliverables**:
- Hierarchical task tree (Epic → Feature → Sub-task)
- Dependency graph (execution constraints)
- Parallel execution opportunities
- Module ownership assignments
- Critical path analysis
- Execution phases with timing

**Output**: `.claude/specs/{feature}/03-task-hierarchy.yaml`

### Step 5: Prompt Generation (10-15 min per task)
**Action**: Invokes `prompt-generator` agent

**Per-Task Output**:
- 10-module structured prompt (see below)
- Unambiguous technical roadmap
- Code references with file:line format
- Complete testing strategy
- Human decision points marked

**Output**: `.claude/specs/{feature}/tasks/TASK-*.md`

### Step 6: Validation & Delivery (10 minutes)
**Action**: Automated validation and summary generation

**Checks**:
- All tasks covered in hierarchy
- No circular dependencies
- Prompt structure complete
- Technical roadmap unambiguous

**Output**: `.claude/specs/{feature}/tasks/execution-plan.md`

## Prompt Structure (10 Modules)

Each generated task prompt contains:

1. **Metadata**: ID, type, parent, module, priority, complexity
2. **Task Relations**: Dependencies, blocking tasks, parallel tasks
3. **Objective**: Core functionality, business value, acceptance criteria
4. **Context**: Existing code, tech stack, integration points
5. **Technical Roadmap**: Technology selections, phased strategy, data structures
6. **Implementation Details**: File organization, pseudocode, code snippets
7. **Testing Strategy**: Test scenarios, commands, ≥90% coverage requirement
8. **Risk Mitigation**: Identified risks with mitigation strategies
9. **Human Decision Points**: Marked decisions with options and recommendations
10. **References**: Code refs (file:line), external docs, ADRs

## Technical Roadmap Standards

### Unambiguous Specification Requirements

✓ **Must Include**:
- Tool/library with version number (`jsonwebtoken@9.0.0`)
- Configuration parameters with exact values (`TTL: 604800 seconds`)
- Algorithm/protocol names (`HS256`, `B-tree`, `LRU`)
- Data structure types and fields (`interface JwtPayload { user_id: string; ... }`)
- All exception types enumerated
- Decision rationale (why A over B)

✓ **Good Example**:
```
Use jsonwebtoken@9.0.0 with HS256 algorithm. Sign with 256-bit secret from
process.env.JWT_SECRET. Access token expires in 900 seconds (15 minutes).
Store refresh tokens in Redis Hash with key format: refresh_token:{userId}:{tokenId},
TTL 604800 seconds. Rationale: HS256 sufficient for single-app deployment,
faster than RS256, simpler key management.
```

✗ **Bad Example**:
```
Use JWT library to generate tokens. Store refresh tokens appropriately.
Handle errors properly.
```

## Output Directory Structure

```
.claude/specs/{feature_name}/
├── 00-requirements.md              # Clarified requirements
├── 01-codebase-analysis.md         # Existing code context
├── 02-technical-design.md          # Technical solution design
├── 03-task-hierarchy.yaml          # Task relationships graph
└── tasks/
    ├── EPIC-001.md                 # Epic-level overview
    ├── FEAT-001.md                 # Feature-level prompt
    ├── FEAT-001-SUB-01.md          # Detailed sub-task prompts
    ├── FEAT-001-SUB-02.md
    ├── FEAT-002-SUB-01.md
    └── execution-plan.md           # Dependency graph + critical path
```

## Usage Examples

### Example 1: Distribute to Multiple AI Agents

```bash
# Generate prompts
/proj-gen-task-prompts "Build payment processing system"

# Distribute to agents
# Agent A gets:
cat .claude/specs/payment-processing/tasks/FEAT-001-SUB-01.md

# Agent B gets:
cat .claude/specs/payment-processing/tasks/FEAT-002-SUB-01.md
```

### Example 2: Feed into /dev Workflow

```bash
# Generate detailed prompt first
/proj-gen-task-prompts "User authentication"

# Use prompt as input for /dev
/dev @.claude/specs/user-authentication/tasks/FEAT-001-SUB-01.md
```

### Example 3: Batch Execution with codeagent-wrapper

```bash
# Execute all tasks in parallel
codeagent-wrapper --parallel <<'EOF'
---TASK---
id: FEAT-001-SUB-01
---CONTENT---
$(cat .claude/specs/feature/tasks/FEAT-001-SUB-01.md)

---TASK---
id: FEAT-001-SUB-02
dependencies: FEAT-001-SUB-01
---CONTENT---
$(cat .claude/specs/feature/tasks/FEAT-001-SUB-02.md)
EOF
```

## Task Hierarchy & Dependencies

### Hierarchy Levels

```
EPIC-001: High-level business objective
  └─ FEAT-001: Functional component
       ├─ FEAT-001-SUB-01: 2-8h implementation unit
       ├─ FEAT-001-SUB-02: 2-8h implementation unit
       └─ FEAT-001-SUB-03: 2-8h implementation unit
```

### Dependency Types

1. **Technical Dependency**: Code B needs code A
2. **Data Dependency**: Task B needs data from task A
3. **Knowledge Dependency**: Understanding from A helps B

### Parallel Execution

Tasks can run in parallel if:
- No dependency relationship
- No shared file modifications
- Independent test suites
- No race conditions

## Human Decision Points

Marked when AI cannot proceed without human input:

**Format**:
```markdown
⚠️ **Human Decision Required: {Topic}**

**Context**: {Why this cannot be automated}

**Options**:
- Option A: {Description}
  - Pros: {Benefits}
  - Cons: {Drawbacks}
- Option B: {Description}
  ...

**Recommendation**: {Suggested option with rationale}

**Action**: Stop implementation and request decision
```

**Common Scenarios**:
- Degradation strategy when dependency fails
- Security vs convenience tradeoffs
- Breaking changes vs backward compatibility

## Quality Assurance

### Validation Checklist (Per Prompt)
- [ ] All 10 modules present and complete
- [ ] Technical roadmap passes specificity standards
- [ ] Code references include file paths with line numbers
- [ ] Testing strategy defines ≥5 scenarios with ≥90% coverage
- [ ] Human decision points marked with options
- [ ] Implementation phases sequenced logically

### Success Metrics
- **Generation Speed**: <15 minutes per prompt
- **Quality Score**: ≥95% on validation checklist
- **AI Success Rate**: ≥85% of prompts result in working code
- **Revision Rate**: <10% require human adjustment

## Troubleshooting

### Issue: Prompts Too Generic

**Symptoms**: AI asks for clarification when executing prompt

**Solution**:
1. Enhance Step 2 (Codebase Analysis)
2. Request more specific code references
3. Add concrete examples to technical roadmap

### Issue: Missing Human Decision Points

**Symptoms**: AI makes arbitrary choices during execution

**Solution**:
1. Review Step 3 (Technical Design)
2. Identify areas with multiple valid approaches
3. Mark as decision points with options

### Issue: Circular Dependencies

**Symptoms**: Tasks cannot be ordered for execution

**Solution**:
1. Review Step 4 (Task Breakdown)
2. Extract shared component into separate task
3. Introduce interface/contract to break cycle

## Integration Points

### With `/dev` Workflow
- Use prompts as detailed input for immediate implementation
- Skip /dev's analysis phase (already done)

### With `codeagent-wrapper`
- Batch execution of multiple tasks
- Support for parallel execution with dependency management

### With Template Library
- Store prompts for future similar features
- Reuse technical roadmaps and implementation patterns

## Best Practices

1. **Invest in Analysis**: Spend time in Steps 2-3 gathering context
2. **Be Specific**: Every decision needs rationale and exact parameters
3. **Mark Unknowns**: When uncertain, create human decision point
4. **Test Execution**: Validate prompts by executing with AI
5. **Iterate**: Improve templates based on execution feedback

## See Also

- [Project Task Prompts Module README](../proj-gen-task-prompts-workflow/README.md)
- [Agent Specifications](../proj-gen-task-prompts-workflow/agents/)
- [Templates](../proj-gen-task-prompts-workflow/templates/)
- [Codeagent-Wrapper Guide](./CODEAGENT-WRAPPER.md)

---

**Version**: 1.0
**Last Updated**: 2025-01-18
**Status**: Production Ready
