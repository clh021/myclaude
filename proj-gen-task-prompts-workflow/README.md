# Project Task Prompts Workflow

Transform requirements into AI-executable task prompts with unambiguous technical roadmaps.

## Overview

The Project Task Prompts Workflow is a specialized workflow module that generates self-contained, detailed task prompts optimized for AI execution. Unlike other workflows that focus on immediate implementation, this workflow creates comprehensive documentation that can be:

- Distributed to multiple AI agents for parallel execution
- Stored as reusable implementation templates
- Used as technical specifications for future development
- Fed into other workflows (e.g., `/dev`) as input

## Key Features

### 1. Unambiguous Technical Roadmaps
Every task prompt includes:
- Specific technology selections with version numbers and rationale
- Detailed implementation strategy broken into phases
- Concrete code references with file paths and line numbers
- Complete error handling scenarios
- Clear human decision points with options analysis

### 2. Hierarchical Task Organization
Tasks are organized in three levels:
- **Epic**: High-level business objective (e.g., "User Authentication System")
- **Feature**: Functional component (e.g., "JWT Token Management")
- **Sub-task**: 2-8 hour implementation unit (e.g., "Token Generation Logic")

### 3. Dependency Management
- Explicit dependency relationships for execution ordering
- Parallel execution opportunities identified
- Critical path analysis for timeline estimation
- Module ownership for code organization

### 4. Quality Assurance
- 10-module prompt structure ensures completeness
- Technical roadmap specificity standards
- ≥90% test coverage requirement
- Validation checklist for every prompt

## When to Use

**Ideal For**:
- Complex features requiring coordination across multiple developers/AIs
- Creating reusable implementation patterns for common tasks
- Documenting detailed technical approaches before implementation
- Preparing work packages that can be executed independently

**Not Ideal For**:
- Quick fixes or small changes (use `/code` or `/debug`)
- Immediate implementation needs (use `/dev`)
- Full project management (use `/bmad-pilot`)

## Workflow Steps

1. **Requirement Clarification** (5-10 min)
   - Interactive Q&A to resolve ambiguities
   - Output: `00-requirements.md`

2. **Codebase Deep Analysis** (15-20 min)
   - Use codeagent to explore existing patterns
   - Output: `01-codebase-analysis.md`

3. **Technical Solution Design** (20-30 min)
   - Invoke `prompt-architect` agent
   - Output: `02-technical-design.md`

4. **Task Breakdown** (15-20 min)
   - Invoke `task-organizer` agent
   - Output: `03-task-hierarchy.yaml`

5. **Prompt Generation** (10-15 min per task)
   - Invoke `prompt-generator` agent
   - Output: `tasks/TASK-*.md`

6. **Validation & Delivery** (10 min)
   - Automated quality checks
   - Generate execution plan
   - Output: `tasks/execution-plan.md`

## Output Structure

```
.claude/specs/{feature_name}/
├── 00-requirements.md              # Clarified requirements
├── 01-codebase-analysis.md         # Existing code context
├── 02-technical-design.md          # Technical solution
├── 03-task-hierarchy.yaml          # Task relationships
└── tasks/
    ├── EPIC-001.md                 # Epic-level prompt
    ├── FEAT-001.md                 # Feature-level prompt
    ├── FEAT-001-SUB-01.md          # Sub-task prompts (detailed)
    ├── FEAT-001-SUB-02.md
    └── execution-plan.md           # Execution plan visualization
```

## Prompt Structure

Each generated task prompt contains 10 required modules:

1. **Metadata**: ID, type, parent, module, priority, complexity
2. **Task Relations**: Dependencies, blocks, parallel tasks
3. **Objective**: Core functionality, business value, acceptance criteria
4. **Context**: Existing code, tech stack, integration points
5. **Technical Roadmap**: Technology selection with rationale, phased strategy
6. **Implementation Details**: File organization, pseudocode, code references
7. **Testing Strategy**: Test scenarios, commands, coverage requirements
8. **Risk Mitigation**: Identified risks with mitigation strategies
9. **Human Decision Points**: Marked decisions with option analysis
10. **References**: Code references, external docs, ADRs

## Usage Example

```bash
# Start workflow
/proj-gen-task-prompts "Implement JWT authentication with refresh tokens"

# Output is generated in:
# .claude/specs/jwt-authentication/tasks/

# Use generated prompts:
# Option A: Manual distribution
cat .claude/specs/jwt-authentication/tasks/FEAT-001-SUB-01.md

# Option B: Batch execution
codeagent-wrapper --parallel < .claude/specs/jwt-authentication/tasks/*.md

# Option C: Feed into /dev workflow
/dev @.claude/specs/jwt-authentication/tasks/FEAT-001-SUB-01.md
```

## Technical Roadmap Standards

### ✓ Good Example (Unambiguous)
```
Use PostgreSQL B-tree index on users.email with UNIQUE constraint.
Command: CREATE UNIQUE INDEX idx_users_email ON users(email);
Rationale: O(log n) lookup, enforces uniqueness at DB level, prevents race conditions.
```

### ✗ Bad Example (Ambiguous)
```
Optimize database queries.
```

### Requirements for Unambiguous Roadmaps
- [ ] Technology/library: name and version (e.g., `jsonwebtoken@9.0.0`)
- [ ] Configuration: exact values (e.g., `TTL: 604800 seconds`)
- [ ] Algorithms: specific names (e.g., `HS256`, not "appropriate algorithm")
- [ ] Data structures: types and fields (e.g., `interface JwtPayload { user_id: string; ... }`)
- [ ] Error handling: all exception types enumerated
- [ ] Rationale: why this approach was chosen over alternatives

## Integration with Other Workflows

### With `/dev` Workflow
```bash
# Generate prompts first
/proj-gen-task-prompts "user authentication"

# Then execute with /dev
/dev @.claude/specs/user-authentication/tasks/FEAT-001-SUB-01.md
```

### With `codeagent-wrapper`
```bash
# Parallel execution of all sub-tasks
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

## Installation

```bash
# Install proj-gen-task-prompts module
python3 install.py --module proj-gen-task-prompts

# Verify installation
ls ~/.claude/commands/proj-gen-task-prompts.md
ls ~/.claude/agents/prompt-*.md
```

## Configuration

No additional configuration required. The workflow uses:
- Existing codeagent skill for codebase analysis
- Standard Task tool for agent invocation
- Standard file paths (`.claude/specs/`)

## Troubleshooting

### Issue: Generated prompts too generic
**Solution**: Enhance codebase analysis by running:
```bash
codeagent-wrapper - <<'EOF'
Deep dive analysis for {feature}:
- Find ALL similar implementations
- Extract EXACT code patterns
- List ALL integration points with line numbers
EOF
```

### Issue: Missing human decision points
**Solution**: Review technical design for unclear areas:
- Multiple valid approaches? Mark decision point
- External dependency failure? Mark fallback strategy decision
- Configuration missing? Mark default value decision

### Issue: Task breakdown too coarse
**Solution**: Apply 2-8 hour rule:
- >8 hours? Break into smaller sub-tasks
- <2 hours? Combine with related work

## Best Practices

1. **Invest in Analysis**: Spend time in Steps 2-3 to gather context
2. **Be Specific**: Every decision needs rationale and specifics
3. **Mark Unknowns**: When unsure, mark as human decision point
4. **Test Execution**: Run generated prompts through AI to validate quality
5. **Iterate Templates**: Improve templates based on execution feedback

## Contributing

To improve the workflow:
1. Update agent definitions in `agents/`
2. Enhance templates in `templates/`
3. Add examples in `docs/PROJ-GEN-TASK-PROMPTS-WORKFLOW.md`
4. Test with real features before committing

## See Also

- [Full Workflow Documentation](../../docs/PROJ-GEN-TASK-PROMPTS-WORKFLOW.md)
- [Agent Specifications](./agents/)
- [Templates](./templates/)
- [codeagent-wrapper Guide](../../docs/CODEAGENT-WRAPPER.md)

---

**Module Version**: 1.0
**Last Updated**: 2025-01-18
**Status**: Ready for Production Use
