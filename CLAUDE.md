# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a **multi-agent agile team starter template** for Claude Code projects. It provides a specialized team of AI agents that collaborate on software development through systematic code review, testing, security analysis, UX evaluation, and parallel development workflows.

**IMPORTANT: This is a shell repository designed to be reused across multiple future projects.** The agent definitions in `.claude/agents/` are pre-configured and ready to use. When starting new projects, you do not need to reconfigure these agents - simply copy this repository and begin delegating work.

## CRITICAL: Agent Delegation Philosophy

**YOU MUST PROACTIVELY DELEGATE TO AGENTS TO CONSERVE CONTEXT.**

As the main Claude instance, your primary role is to orchestrate and delegate work to specialized subagents. This architecture is designed to:

1. **Preserve your context window** - Offload implementation, testing, security reviews, and code cleanup to agents
2. **Enable parallel execution** - Launch multiple independent agents simultaneously for faster delivery
3. **Provide specialized expertise** - Each agent has deep domain knowledge and follows best practices
4. **Ensure comprehensive coverage** - code-simplifier and security-reviewer systematically scan the entire codebase over multiple iterations

### When to Delegate (Almost Always)

**Delegate to agents for:**
- Any feature implementation (tech-lead-developer)
- All testing work (qc-test-maintainer)
- Security reviews after any code changes (security-reviewer)
- Code cleanup and optimization (code-simplifier)
- UI/UX evaluation (ux-reviewer)

**Handle directly only for:**
- Simple file reads or searches (use Read/Grep tools directly)
- Answering quick questions about the codebase
- Coordinating between agents

### Parallel Agent Execution

**ALWAYS launch agents in parallel when tasks are independent:**
```
# CORRECT - Single message with multiple Task calls
<Task tool call for endpoint A>
<Task tool call for endpoint B>
<Task tool call for endpoint C>

# INCORRECT - Sequential calls that could be parallel
<Task tool call for endpoint A>
[wait for response]
<Task tool call for endpoint B>
[wait for response]
```

Multiple tech-lead-developer instances can work simultaneously on non-conflicting features, endpoints, or modules.

### Iterative Agent Pattern for Full Codebase Coverage

**code-simplifier and security-reviewer are designed for ITERATIVE execution:**

These agents maintain tracking files that enable them to:
- Scan the entire codebase over multiple invocations
- Automatically resume from where they left off
- Track when files were last reviewed
- Detect file changes and trigger re-reviews
- Prioritize sections that need attention

**DO NOT try to scan the entire codebase in a single agent call.** Instead:
1. Call the agent multiple times
2. Each call processes a manageable chunk
3. The agent reads its tracking file to determine what to scan next
4. The agent updates the tracking file after each scan

This pattern prevents context overflow while ensuring comprehensive coverage.

## Agent Architecture

This repository defines five specialized agents in `.claude/agents/`:

### tech-lead-developer (Blue)
Primary development agent for implementing features and modules. Designed for parallel execution of independent tasks.

**Key Use Cases:**
- Creating independent API endpoints or services
- Implementing utility functions and modules
- Building isolated frontend components
- Developing features without shared dependencies

**Parallel Development Pattern:**
When you have multiple independent tasks (e.g., three separate API endpoints), launch multiple instances of this agent simultaneously using parallel Task tool calls in a single message.

### qc-test-maintainer (Yellow)
Quality assurance specialist for comprehensive testing.

**Key Responsibilities:**
- Create unit, integration, and end-to-end tests in `tests/` directory
- Maintain and update existing test suites
- Run test validation after changes
- Build mock data and fixtures in `tests/fixtures/`
- Use pytest as the testing framework
- Ensure tests are deterministic, independent, and fast

**Testing Standards:**
- ALL test files go in `tests/` directory (NEVER in project root)
- Use `conftest.py` for shared fixtures
- Store test outputs in `tests/outputs/` (gitignored)
- Run full suite with `pytest tests/`

### security-reviewer (Red)
Security architect focused on threat modeling and vulnerability assessment.

**Key Focus Areas:**
- OWASP Top 10 vulnerabilities
- Authentication and authorization implementations
- File upload validation and data handling security
- API endpoint security
- Environment variable and secrets management
- Docker and deployment security configurations

**Iterative Scanning Pattern:**
The security-reviewer maintains a `.security-review-tracking.json` file to systematically scan the entire codebase over multiple iterations. This enables:
- Comprehensive security coverage without overwhelming context windows
- Tracking which files/modules have been reviewed and when
- Automatic re-scanning when files are modified
- Prioritization of high-risk areas (auth, file uploads, API endpoints)

**When to Use:**
1. **Proactively after implementing features** involving authentication, data handling, API endpoints, file uploads, external integrations, or deployment configurations
2. **Periodically for full codebase scans** - The agent will check its tracking file and scan sections that haven't been reviewed recently or have changed since last review
3. **Before deployments** - Full security audit by calling the agent multiple times to cover all modules

**Usage Pattern:**
Call the security-reviewer multiple times to scan the entire codebase in chunks. The agent will automatically track progress and resume from where it left off. Alternatively, the agent will query its tracking file to determine if rescanning is needed based on file changes.

### code-simplifier (Cyan)
Software architect for systematic codebase cleanup and optimization.

**Core Responsibilities:**
- Remove code duplication and orphaned code
- Simplify unnecessary complexity
- Maintain `.code-simplifier-tracking.json` to rotate through codebase
- Run tests before and after changes to ensure nothing breaks
- Work on feature branches, never directly on main

**Iterative Scanning Pattern:**
The code-simplifier maintains a `.code-simplifier-tracking.json` file that tracks:
- When each file/module was last reviewed
- What issues were found and fixed
- What changes are pending developer approval
- A rotation schedule to ensure comprehensive coverage over time

This enables systematic cleanup of the entire codebase without context overload.

**Operational Pattern:**
1. Check `.code-simplifier-tracking.json` for sections due for review or recently modified code
2. Analyze code for duplication, orphaned code, and complexity
3. Make safe changes immediately (unused imports, dead code, simple refactoring)
4. Consult developer for complex refactoring requiring architectural changes
5. Always run relevant tests before committing
6. Update tracking file with review timestamps and findings

**When to Use:**
1. **After completing features** - Clean up newly written code before moving on
2. **Periodically for rotation** - The agent will query its tracking file to find sections that haven't been reviewed recently
3. **Before releases** - Call multiple times to systematically clean the entire codebase
4. **When files change** - Agent checks tracking file against actual file modification times to identify code needing re-review

**Usage Pattern:**
Call the code-simplifier multiple times to systematically rotate through the entire codebase. The agent will automatically track which sections need attention and resume from the appropriate point. Each invocation processes a manageable chunk of the codebase.

### ux-reviewer (Pink)
User experience specialist for visual appeal and cross-platform usability.

**Review Focus:**
- Visual design assessment (color, typography, spacing, hierarchy)
- Usability evaluation (task flow, affordances, feedback, error handling)
- Cross-platform compatibility (mobile, tablet, desktop)
- Code efficiency (avoid UI bloat, leverage existing framework features)
- Accessibility (WCAG 2.1 AA compliance)

**When to Use:**
After implementing or updating UI components, making layout changes, or adding interactive elements.

## Recommended Workflows

### Feature Development Workflow
1. **tech-lead-developer**: Implement the feature
2. **qc-test-maintainer**: Create comprehensive tests
3. **security-reviewer**: Audit for vulnerabilities
4. **code-simplifier**: Optimize and refactor
5. **ux-reviewer**: Review if UI components involved

### Parallel Development Pattern
For independent tasks that don't share dependencies:
```
"I need three new endpoints: /export, /validate, and /stats.
Use tech-lead-developer agents in parallel for each."
```

Launch multiple agents simultaneously by making multiple Task tool calls in a single message.

### Maintenance Workflow (Iterative Pattern)

**IMPORTANT:** code-simplifier and security-reviewer are designed for iterative execution. Call them multiple times to systematically cover the entire codebase.

- **Weekly code cleanup**: Call code-simplifier multiple times - it will query `.code-simplifier-tracking.json` to review modules not checked recently or that have changed
- **Weekly security scans**: Call security-reviewer multiple times - it will query `.security-review-tracking.json` to scan sections due for review
- **After major changes**: Use qc-test-maintainer to run full test suite and update tests
- **Pre-deployment**:
  - Call security-reviewer multiple times to complete full security audit
  - Call code-simplifier multiple times to clean entire codebase
  - Call qc-test-maintainer to verify all tests pass

**Example - Full Codebase Security Audit:**
```
Call security-reviewer (iteration 1 - scans auth modules)
Call security-reviewer (iteration 2 - scans API endpoints)
Call security-reviewer (iteration 3 - scans data handling)
Call security-reviewer (iteration 4 - scans deployment configs)
```

Each call automatically picks up where the previous scan left off by reading the tracking file.

## Git Workflow Integration

Standard branching pattern:
```bash
git checkout -b feature/new-feature-name

# Use agents for development, testing, security review, cleanup
# Agents can commit work to feature branches if significant work is done

git add .
git commit -m "feat: description of feature with tests and security review"
git push origin feature/new-feature-name
```

Main branch: `main`

## Project Structure

```
.
├── .claude/
│   └── agents/              # Agent definition files (DO NOT modify without understanding impact)
│       ├── tech-lead-developer.md
│       ├── qc-test-maintainer.md
│       ├── security-reviewer.md
│       ├── code-simplifier.md
│       └── ux-reviewer.md
├── .gitignore               # Python-standard gitignore
├── CLAUDE.md                # This file (customize for your project)
├── README.md                # User-facing documentation
├── requirements.txt         # Python dependencies
└── [Agent tracking files - created at runtime, add to .gitignore]
    ├── .code-simplifier-tracking.json
    └── .security-review-tracking.json
```

## Agent Tracking Files

The code-simplifier and security-reviewer agents maintain tracking files to systematically scan the entire codebase over multiple iterations:

**`.code-simplifier-tracking.json`**
- Tracks when each file/module was last reviewed
- Records issues found, fixed, and pending
- Maintains rotation schedule for comprehensive coverage
- Updates automatically on each agent invocation

**`.security-review-tracking.json`**
- Tracks security review status of all files/modules
- Records vulnerability findings and remediation status
- Prioritizes high-risk areas (auth, file uploads, APIs)
- Tracks when files change to trigger re-scanning

**IMPORTANT:** Add these tracking files to your `.gitignore` as they are runtime artifacts specific to each developer's workflow and should not be committed to version control.

## Customization for Your Project

When adapting this template:

1. **Update CLAUDE.md** with project-specific:
   - Technology stack details
   - Build/test/lint commands
   - Architecture patterns unique to your project
   - Domain-specific requirements

2. **Optionally customize agents** in `.claude/agents/`:
   - Add project-specific context sections
   - Adjust coding standards and conventions
   - Include industry-specific requirements
   - Modify operational workflows

3. **Maintain agent collaboration patterns**:
   - Sequential: One agent completes before next begins (safest)
   - Parallel: Multiple agents work simultaneously on independent modules (fastest)
   - Iterative: Agent makes changes, another reviews, first refines (highest quality)

## Agent Invocation Best Practices

**Context Conservation (Primary Goal):**
- **ALWAYS delegate to agents** for any non-trivial work to preserve main context window
- Offload implementation, testing, security, and cleanup work to subagents
- Your role as main Claude is to orchestrate, not to implement

**Parallel Execution:**
- **Launch multiple agents in single message** when tasks are independent
- Multiple tech-lead-developer instances can work simultaneously on different features
- Never wait for one agent to finish if another can start immediately

**Iterative Patterns:**
- **Call code-simplifier multiple times** to systematically rotate through entire codebase
- **Call security-reviewer multiple times** for comprehensive security coverage
- Agents automatically track progress via `.code-simplifier-tracking.json` and `.security-review-tracking.json`
- Each invocation picks up where the last one left off

**Agent Communication:**
- Provide clear, complete instructions for what agent should accomplish
- Specify what information agent should return in its final report
- Each agent invocation is stateless - provide full context needed
- For parallel work, ensure tasks are truly independent with no shared dependencies

## Important Notes

- This is a starter template with no actual application code yet
- No build/test/lint commands exist until you add your project code
- The agent definitions are production-ready but expect customization
- Focus README.md on user-facing documentation; keep CLAUDE.md for AI context
