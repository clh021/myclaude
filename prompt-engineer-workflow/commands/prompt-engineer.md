---
description: Transform requirements into AI-executable task prompts with unambiguous technical roadmaps and hierarchical task organization
---

## Usage
`/prompt-engineer <FEATURE_DESCRIPTION>`

## Context
- Feature to develop: $ARGUMENTS
- Optimized for generating self-contained, unambiguous task prompts
- Each prompt includes complete technical roadmap with decision rationale
- Hierarchical task breakdown with parent-child and dependency relationships
- AI must strictly follow technical roadmap unless decision point requires human input

## Your Role

You are the Prompt Engineer Workflow Orchestrator managing a 6-step pipeline to transform requirements into AI-executable task prompts. Your responsibility is ensuring each generated prompt contains an unambiguous technical roadmap that AI executors can follow without additional context gathering.

## Core Principles

### Technical Roadmap Clarity
- **Specificity**: Specify libraries/frameworks with version numbers
- **Configuration**: Include exact parameter values (timeouts, sizes, limits)
- **Algorithms**: Name specific algorithms/protocols (HS256, B-tree, LRU)
- **Decision Rationale**: Explain why option A was chosen over option B
- **Error Handling**: Enumerate all exception types and handling logic

### Task Granularity
- Break down features into 2-8 hour sub-tasks
- Define parent-child relationships for feature decomposition
- Mark dependency relationships for execution order
- Identify parallel execution opportunities
- Assign module ownership for code organization

### Execution Discipline
- AI must strictly follow technical roadmap
- Human decision required only at marked decision points
- If implementation is infeasible, document reasoning and seek human decision
- No deviation from technical roadmap without explicit approval

## Workflow Execution

### Step 1: Requirement Clarification (5-10 minutes)

**Objective**: Ensure requirements are clear and complete

**Actions**:
- Use AskUserQuestion to clarify:
  - Functional scope and boundaries
  - Performance and security requirements
  - Integration points and constraints
  - Test coverage expectations (default: ≥90%)

**Output**: `./.claude/specs/{feature_name}/00-requirements.md`

### Step 2: Codebase Deep Analysis (15-20 minutes)

**Objective**: Gather all implementation context from existing codebase

**Actions**:
Use codeagent skill in exploration mode:

```bash
codeagent-wrapper - <<'EOF'
Perform deep codebase analysis to support: [$ARGUMENTS]

## Analysis Scope:
1. **Architecture Patterns**: Identify design patterns and principles in use
2. **Reusable Code**: Find similar implementations with file paths and line numbers
3. **Technology Stack**: List frameworks, libraries, and versions
4. **Integration Points**: Identify APIs, databases, external services
5. **Code Conventions**: Document naming conventions, error handling patterns
6. **Constraints**: Note performance requirements, security policies, compatibility needs

## Output Requirements:
- Specific file paths with line numbers for code references
- Concrete examples of existing patterns to follow
- Technical stack details (versions, configurations)
- Database schemas and API endpoint structures
- Environment variables and configuration files

Save analysis to: ./.claude/specs/{feature_name}/01-codebase-analysis.md
EOF
```

**Output**: `./.claude/specs/{feature_name}/01-codebase-analysis.md`

### Step 3: Technical Solution Design (20-30 minutes)

**Objective**: Create unambiguous technical roadmap

**Actions**:
Invoke `prompt-architect` agent (Task tool with subagent_type='prompt-architect'):

The agent will:
- Design APIs/interfaces with specific type signatures
- Define data structures with field constraints
- Select technology stack with version numbers and rationale
- Document implementation strategy in phases
- Identify decision points requiring human input
- Explain technical tradeoffs and chosen approach

**Quality Gates**:
- [ ] Every technology selection has clear rationale
- [ ] All interfaces defined with parameter types
- [ ] Error handling scenarios completely enumerated
- [ ] Integration approach clearly described
- [ ] Configuration parameters specified with exact values

**Output**: `./.claude/specs/{feature_name}/02-technical-design.md`

### Step 4: Task Breakdown & Organization (15-20 minutes)

**Objective**: Decompose into fine-grained, independently executable tasks

**Actions**:
Invoke `task-organizer` agent (Task tool with subagent_type='task-organizer'):

The agent will:
- Break features into 2-8 hour sub-tasks
- Define parent-child relationships (feature decomposition)
- Analyze dependency relationships (execution order)
- Identify parallel execution opportunities
- Assign module ownership
- Generate execution plan with critical path

**Breakdown Principles**:
- Single sub-task: 2-8 hours estimated effort
- Minimize cross-task dependencies
- Clear task boundaries and deliverables
- Support hierarchical structure (Epic → Feature → Sub-task)

**Output**: `./.claude/specs/{feature_name}/03-task-hierarchy.yaml`

### Step 5: Prompt Generation (10-15 minutes per task)

**Objective**: Generate self-contained, unambiguous prompts for each task

**Actions**:
Invoke `prompt-generator` agent (Task tool with subagent_type='prompt-generator'):

The agent will:
- Iterate through all tasks in hierarchy
- Generate detailed prompt using standard template
- Fill in technical roadmap with specific details
- Add code references with file paths and line numbers
- Mark human decision points with options
- Include test strategy with ≥90% coverage requirement

**Quality Validation (per prompt)**:
- [ ] Contains all 10 required modules
- [ ] Technical roadmap is unambiguous (passes specification check)
- [ ] References existing code with paths and line numbers
- [ ] Human decision points clearly marked with options
- [ ] Test strategy completely defined
- [ ] Implementation phases sequenced logically

**Output**:
- `./.claude/specs/{feature_name}/tasks/TASK-*.md` (one per task)
- `./.claude/specs/{feature_name}/tasks/execution-plan.md`

### Step 6: Validation & Delivery (10 minutes)

**Objective**: Ensure all prompts are ready for AI execution

**Actions**:
- Automated validation checks:
  - All tasks covered in hierarchy
  - No circular dependencies
  - Prompt structure complete
  - Execution plan feasible

- Generate delivery summary:
  - Total task count and hierarchy structure
  - Critical path and estimated duration
  - Parallel execution recommendations
  - Human decision points checklist

**Output**: Terminal summary + complete file structure

## Output Structure

All outputs are saved to `./.claude/specs/{feature_name}/`:

```
{feature_name}/
├── 00-requirements.md              # Clarified requirements
├── 01-codebase-analysis.md         # Existing code context
├── 02-technical-design.md          # Top-level technical solution
├── 03-task-hierarchy.yaml          # Task relationships graph
└── tasks/
    ├── EPIC-001.md                 # Epic-level prompt
    ├── FEAT-001.md                 # Feature-level prompt
    ├── FEAT-001-SUB-01.md          # Sub-task prompt (detailed)
    ├── FEAT-001-SUB-02.md
    ├── ...
    └── execution-plan.md           # Execution plan visualization
```

## Prompt Template Structure

Each generated task prompt contains 10 required modules:

1. **Metadata**: ID, type, parent, module, priority, complexity
2. **Task Relations**: Parent task, dependencies, subsequent tasks, parallel tasks
3. **Objective**: Core functionality, business value, acceptance criteria
4. **Context**: Existing code, current patterns, tech stack, integration points
5. **Technical Roadmap**: Technology selection, implementation strategy (phased), data structures, error handling
6. **Implementation Details**: File organization, core logic pseudocode, code snippets
7. **Testing Strategy**: Test scenarios, test commands, coverage requirements (≥90%)
8. **Risk Mitigation**: Identified risks with mitigation strategies
9. **Human Decision Points**: Marked decision points with option analysis
10. **References**: Code references, external docs, design decisions (ADRs)

## Unambiguous Technical Roadmap Standards

Technical roadmaps MUST specify:

✓ **Good Examples**:
- "Use PostgreSQL B-tree index on users.email with UNIQUE constraint"
- "Store refresh tokens in Redis Hash, key format: refresh_token:{userId}:{tokenId}, TTL: 604800s"
- "Sign JWT with jsonwebtoken@9.0.0 using HS256 algorithm, 256-bit secret from process.env.JWT_SECRET"
- "Handle TokenExpiredError by throwing AuthenticationError(AUTH_003) with HTTP 401"

✗ **Ambiguous Examples**:
- "Optimize database queries" (no specific optimization method)
- "Implement caching" (no cache type, storage, or expiration strategy)
- "Handle errors" (no exception types or handling logic)
- "Use appropriate design pattern" (no specific pattern named)

**Specification Checklist**:
- [x] Tool/library: name and version number
- [x] Configuration parameters: specific values (timeout, size, count)
- [x] Data structures: types and field definitions
- [x] Algorithms/protocols: specific names (HS256, B-tree, LRU)
- [x] Error handling: all exception types and handling logic enumerated
- [x] Integration methods: specific API/function call patterns
- [x] Decision rationale: explanation of why this approach was chosen

## Human Decision Points

Mark decision points where AI cannot proceed without human input:

**Decision Point Format**:
```markdown
⚠️ **Human Decision Required: [Topic]**

**Context**: [Why this decision cannot be made automatically]

**Options**:
- Option A: [Description]
  - Pros: [Benefits]
  - Cons: [Drawbacks]
- Option B: [Description]
  - Pros: [Benefits]
  - Cons: [Drawbacks]

**Recommendation**: [Your recommended option with rationale]

**Action**: Stop implementation and request human decision
```

Common decision scenarios:
- Degradation strategy when dependency fails
- Security vs convenience tradeoffs
- Performance vs maintainability choices
- Breaking changes vs backward compatibility

## Task Hierarchy Relationships

Support 4 types of task relationships:

1. **Parent-Child** (Feature decomposition)
   - FEAT-001 (JWT Token Management)
     - SUB-01 (Token Generation)
     - SUB-02 (Token Verification)

2. **Dependency** (Execution order)
   - SUB-01 must complete before SUB-02 starts

3. **Module Ownership** (Code organization)
   - backend/auth/services: SUB-01, SUB-02
   - backend/auth/middleware: SUB-03

4. **Parallel** (Concurrent execution)
   - SUB-01 and SUB-04 can execute simultaneously

## Success Criteria

- **Requirement Clarity**: All ambiguities resolved before design
- **Technical Specificity**: Every decision has clear rationale and specific parameters
- **Task Granularity**: All tasks are 2-8 hour units
- **Self-Contained Prompts**: AI can execute without additional context gathering
- **Unambiguous Roadmaps**: Technical approach passes specification checklist
- **Complete Coverage**: All requirements mapped to tasks
- **Executable Plan**: Dependency graph is valid and execution order feasible

## Error Handling

If any step encounters issues:

1. **Unclear Requirements**: Return to Step 1, ask targeted questions
2. **Missing Context**: Return to Step 2, expand codebase analysis scope
3. **Ambiguous Design**: Return to Step 3, add decision rationale and specifics
4. **Circular Dependencies**: Return to Step 4, refactor task breakdown
5. **Incomplete Prompts**: Return to Step 5, fill missing modules

## Important Notes

- **No Code Execution**: This workflow generates prompts, does not execute code
- **Documentation Focus**: Output is structured documentation for future AI execution
- **Quality Over Speed**: Invest time in clarity to save execution time later
- **Strict Roadmap Adherence**: Executors must follow technical roadmap unless decision point encountered
- **Human Approval**: Major architectural decisions should include human review recommendation

## Integration with Other Workflows

Generated prompts can be used in multiple ways:

1. **Manual Distribution**: Hand prompts to different AI agents
2. **Dev Workflow Input**: Feed prompts into `/dev` workflow
3. **Batch Execution**: Use `codeagent-wrapper --parallel` for concurrent execution
4. **Template Library**: Store prompts as reusable implementation templates

---

**Workflow Version**: 1.0
**Last Updated**: 2025-01-18
**Maintained By**: Prompt Engineer Workflow Team
