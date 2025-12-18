# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a multi-agent workflow system for Claude Code that implements dual-agent architecture:
- **Orchestrator**: Claude Code handles planning, context gathering, verification, and user interaction
- **Executor**: codeagent-wrapper delegates code editing/testing to pluggable AI backends (Codex/Claude/Gemini)

The system provides modular workflows (dev, bmad-agile, requirements-driven, essentials) installable via `install.py`.

## Core Architecture

### 1. Modular Installation System

The repository uses a JSON-driven installer (`install.py`) with configuration in `config.json`:

**Module Structure:**
- `dev`: Core dev workflow + codeagent skill (default enabled)
- `bmad`: Full agile workflow with 6 specialized agents (default disabled)
- `requirements`: Requirements-driven workflow (default disabled)
- `essentials`: Core development commands (default enabled)

**Installation:**
```bash
# Install default modules (dev + essentials)
python3 install.py --install-dir ~/.claude

# Install specific module
python3 install.py --module bmad

# Force overwrite
python3 install.py --force
```

**What Gets Installed:**
```
~/.claude/
├── CLAUDE.md              # Role definition (from memorys/CLAUDE.md)
├── commands/              # Slash commands merged from modules
├── agents/                # Agent definitions merged from modules
├── skills/codeagent/      # Codeagent integration
└── installed_modules.json # Installation tracking
```

### 2. Codeagent-Wrapper Architecture

**Location**: `/codeagent-wrapper/` (Go implementation)

**Purpose**: Multi-backend AI code execution wrapper that parses JSON streaming output from backend CLIs.

**Key Components:**
- `main.go`: CLI entry point, argument parsing, mode routing
- `executor.go`: Task execution with timeout/signal handling
- `parser.go`: JSON stream parsing for backend output
- `logger.go`: PID-aware logging for parallel execution
- `backend.go`: Backend command construction (codex/claude/gemini)
- `config.go`: Permission and concurrency control

**Supported Modes:**
1. **Standard**: `codeagent-wrapper [--backend <backend>] <task> [workdir]`
2. **HEREDOC**: `codeagent-wrapper [--backend <backend>] - [workdir] <<'EOF'`
3. **Resume**: `codeagent-wrapper [--backend <backend>] resume <session_id> - <<'EOF'`
4. **Parallel**: `codeagent-wrapper --parallel [--backend <backend>] <<'EOF'`

**Backend Selection:**
- `codex` (default): Complex code analysis, refactoring, algorithm optimization
- `claude`: Quick features, documentation, prompt engineering
- `gemini`: UI prototyping, design system implementation

**Parallel Execution Format:**
```
---TASK---
id: task_id
backend: <backend>  # Optional per-task override
workdir: /path
dependencies: dep1, dep2
---CONTENT---
task content
```

**Build/Test:**
```bash
cd codeagent-wrapper
go build -o codeagent-wrapper
go test ./...
```

### 3. Workflow System

**Primary Workflow: /dev**

Location: `dev-workflow/commands/dev.md`

6-step lightweight workflow:
1. **Requirement Clarification**: AskUserQuestion for functional boundaries, constraints, test coverage
2. **Codex Deep Analysis**: Plan-mode style exploration with UI detection (checks for .css/.scss/styled-components AND .tsx/.jsx/.vue)
3. **Dev Plan Generation**: Invoke `dev-plan-generator` agent → creates `.claude/specs/{feature}/dev-plan.md`
4. **Parallel Execution**: Execute tasks via codeagent skill (codex for backend, gemini for UI)
5. **Coverage Validation**: Enforce ≥90% test coverage, retry up to 2 rounds if insufficient
6. **Completion Summary**: Report tasks, coverage, file changes

**Key Decision Points:**
- Skip deep analysis for simple/single-file changes
- UI detection criteria: style assets (CSS/SCSS/styled-components/CSS modules/tailwindcss) + frontend components (tsx/jsx/vue)
- Backend selection: codex (default), gemini (if UI detected)

**Other Workflows:**
- `/bmad-pilot`: Full agile (PO → Architect → Tech Lead → Dev → Review → QA)
- `/requirements-pilot`: Requirements → Plan → Code → Review → Test
- `/project-task-prompts`: Requirements → Unambiguous task prompts (for distributed AI execution)
- `/project-standards`: Extract code patterns → Generate CODESTYLE.md → Review best practices → Migration roadmap
- Direct commands: `/code`, `/debug`, `/test`, `/review`, `/optimize`, `/refactor`, `/docs`

## Development Commands

### Building/Testing

**Codeagent-Wrapper:**
```bash
# Build
cd codeagent-wrapper
go build -o codeagent-wrapper

# Test
go test ./...                           # All tests
go test -v -run TestParserBasic         # Specific test
go test -race ./...                     # Race detection
go test -bench=. ./...                  # Benchmarks

# Coverage
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

**Installation System:**
```bash
# Test installer
python3 install.py --list-modules       # List available modules
python3 install.py --verbose            # Verbose output
python3 install.py --force              # Force overwrite

# Verify installation
cat ~/.claude/installed_modules.json
ls -la ~/.claude/commands/
```

**Makefile Targets:**
```bash
make build          # Build codeagent-wrapper
make test           # Run tests
make coverage       # Generate coverage report
make install        # Install to ~/.claude/bin
make clean          # Clean build artifacts
```

### Linting/Formatting

```bash
# Go code
cd codeagent-wrapper
go fmt ./...
go vet ./...
golangci-lint run   # If installed

# Python installer
cd /
python3 -m pylint install.py
python3 -m black install.py --check
```

## Key Files and Patterns

**Installation Flow:**
1. `config.json` defines modules and operations (merge_dir, copy_file, copy_dir, run_command)
2. `install.py` validates against `config.schema.json`, executes operations, logs to `install.log`
3. Each module's operations copy files to `~/.claude/` and run setup commands (e.g., `bash install.sh`)

**Workflow Orchestration:**
- Commands (`.md` files in `*/commands/`) define workflow steps as prompts
- Agents (`.md` files in `*/agents/`) are invoked via Task tool with `subagent_type`
- Skills (`.md` files in `skills/*/SKILL.md`) integrate external tools (codeagent-wrapper)

**Backend Integration:**
- All code edits/tests MUST go through codeagent skill (NEVER directly via Edit/Write tools per memorys/CLAUDE.md)
- Use HEREDOC format for complex tasks: `codeagent-wrapper - <<'EOF'`
- Reference files with `@` syntax: `@src/auth.ts`

**Logging/Debugging:**
- `install.log`: Installation operations, command output, errors
- codeagent-wrapper: Logs include PID for parallel execution correlation
- Session IDs: Appended to output as `SESSION_ID: <uuid>` for resumption

## Environment Variables

**Codeagent-Wrapper:**
- `CODEX_TIMEOUT`: Timeout in milliseconds (default: 7200000 = 2 hours)
- `CODEAGENT_SKIP_PERMISSIONS`: Permission control
  - Claude backend: Set to `true`/`1` to **disable** `--dangerously-skip-permissions` (default: enabled)
  - Codex/Gemini: Set to `true`/`1` to enable permission skipping (default: disabled)
- `CODEAGENT_MAX_PARALLEL_WORKERS`: Limit concurrent tasks (default: unlimited, recommended: 8)

## Testing Conventions

**Codeagent-Wrapper:**
- Unit tests: `*_test.go` files alongside implementation
- Integration tests: `main_integration_test.go`
- Stress tests: `concurrent_stress_test.go`, `executor_concurrent_test.go`
- Test fixtures: Use `t.TempDir()` for temporary directories
- Mock backends: `testCmdEcho`, `testCmdSleep` for timeout/signal testing

**Test Coverage Requirements:**
- Target: ≥90% per the /dev workflow mandate
- Run coverage before declaring tasks complete
- If <90%, add tests for uncovered scenarios (max 2 retry rounds)

## Important Constraints

1. **Workflow Contract** (from memorys/CLAUDE.md):
   - Claude Code: intake, context gathering, planning, verification ONLY
   - Every edit/test: delegated to codeagent skill
   - NEVER use Edit/Write tools directly for code changes

2. **Backend Selection Logic**:
   - UI work (CSS/SCSS/styled-components + tsx/jsx/vue): Use gemini backend
   - Complex analysis/refactoring: Use codex backend (default)
   - Documentation/prompts: Use claude backend

3. **Parallel Execution Safety**:
   - Topological sort ensures dependency order
   - Independent tasks run concurrently (unlimited by default)
   - Errors isolated (failures don't stop other tasks)
   - Set `CODEAGENT_MAX_PARALLEL_WORKERS` in production

4. **Installation Behavior**:
   - `merge_dir` merges subdirs (commands/, agents/) into `~/.claude/`
   - `copy_file` preserves source structure relative to target path
   - Existing files skipped unless `--force` specified
   - Rollback on failure unless `--force` used

5. **Communication**:
   - Think in English, respond in Chinese (per memorys/CLAUDE.md role definition)
   - Keep output concise (≤6 bullet points for medium changes)
   - Include file paths with line numbers when referencing code

## Common Tasks

**Add a New Workflow Module:**
1. Create directory: `<module-name>-workflow/{commands,agents}/`
2. Add module config to `config.json`:
   ```json
   "module-name": {
     "enabled": false,
     "description": "...",
     "operations": [
       {"type": "merge_dir", "source": "<module-name>-workflow"}
     ]
   }
   ```
3. Test: `python3 install.py --module module-name`

**Add a New Backend:**
1. Update `codeagent-wrapper/backend.go`: Add case for new backend
2. Update `codeagent-wrapper/config.go`: Add permission defaults
3. Update `skills/codeagent/SKILL.md`: Document backend usage
4. Rebuild: `cd codeagent-wrapper && go build`

**Debug Codeagent-Wrapper:**
1. Check binary exists: `which codeagent-wrapper`
2. Test directly: `codeagent-wrapper "simple task"`
3. Check logs for PID correlation in parallel mode
4. Verify PATH: `echo $PATH | grep "$HOME/bin"`

**Extend /dev Workflow:**
1. Modify `dev-workflow/commands/dev.md` for orchestration logic
2. Add agents in `dev-workflow/agents/` if new specialized tasks needed
3. Ensure coverage validation remains ≥90%
4. Test: `/dev "test feature"`
