---
name: claude-code
description: Claude Code CLI - Run Claude Code locally for codebase analysis, refactors, testing, and automation. Use when you need to use Claude Code CLI, run headless mode (-p), use Plan Mode, create commits/PRs, or integrate Claude Code into workflows.
---

# Claude Code Skill

Use the locally installed **Claude Code** CLI effectively. This skill covers headless mode, interactive workflows, and integration patterns.

## Quick Start

### Verify Installation
```bash
claude --version
```

### Basic Usage Patterns

**Interactive mode:**
```bash
claude
# Start REPL, ask questions, make changes interactively
```

**Headless mode (one-shot queries):**
```bash
claude -p "What does this project do?"
# Returns answer and exits
```

**Continue previous conversation:**
```bash
claude -c
# Continue most recent conversation
claude -r "session-name"
# Resume specific session
```

## Core Features

### 1. Headless Mode (CLI Automation)

Headless mode (`-p` or `--print`) runs Claude Code non-interactively, perfect for scripts and CI/CD.

#### Basic One-Shot Queries
```bash
# Analyze code
claude -p "Summarize this project structure"

# Get structured output
claude -p "List all API endpoints" --output-format json

# Process piped data
cat error.log | claude -p "Explain the root cause"
```

#### Auto-Approve Tools
```bash
# Allow specific tools without prompting
claude -p "Run tests and fix failures" \
  --allowedTools "Bash,Read,Edit"
```

#### Custom System Prompts
```bash
# Append to system prompt (preserves defaults)
claude -p "Review for security" \
  --append-system-prompt "You are a security engineer. Be strict."

# Replace entire system prompt
claude -p "Refactor code" \
  --system-prompt "You are a Python expert. Use type hints."
```

#### Output Formats
```bash
# Plain text (default)
claude -p "Explain this function"

# JSON with metadata
claude -p "Summarize project" --output-format json

# Streaming JSON for real-time processing
claude -p "Analyze logs" --output-format stream-json --verbose

# JSON Schema validation
claude -p "Extract function names" \
  --json-schema '{"type":"object","properties":{"functions":{"type":"array","items":{"type":"string"}}}}'
```

#### Permission Modes
```bash
# Plan mode (read-only analysis)
claude -p "Analyze and propose refactor plan" \
  --permission-mode plan

# Auto-accept edits
claude -p "Fix lint errors" \
  --permission-mode acceptEdits

# Skip all permissions (use with caution)
claude -p "Generate boilerplate" \
  --dangerously-skip-permissions
```

### 2. Interactive Workflows

#### Plan Mode for Safe Analysis
Use Plan Mode to analyze codebases before making changes:

```bash
# Start session in Plan Mode
claude --permission-mode plan

# Or toggle during session: Shift+Tab (cycles: Normal → Auto-Accept → Plan)
```

**Example workflow:**
```bash
claude --permission-mode plan
# > Analyze authentication system and suggest improvements
# > How about backward compatibility?
# > What about database migration?
# [Exit Plan Mode with Esc or Shift+Tab]
# > Implement the plan
```

#### Create Commits & PRs
```bash
# Review staged changes and create commit
claude -p "Review staged changes and create appropriate commit" \
  --allowedTools "Bash(git diff *),Bash(git commit *)"

# Full PR workflow (analyze → commit → push → PR)
claude commit
# Claude reviews changes, creates commit, pushes, opens PR
```

### 3. Subagents (Specialized AI Assistants)

Subagents run in isolated contexts with custom tools and permissions.

#### Built-in Subagents
- **Explore**: Fast read-only agent for codebase exploration
- **Plan**: Research agent for planning mode
- **General-purpose**: Complex multi-step tasks

#### Create Custom Subagents

**Method 1: Interactive**
```bash
claude
# > /agents
# > Create new subagent
# Follow prompts
```

**Method 2: Create File Manually**
```bash
# Create ~/.claude/agents/security-reviewer.md
cat <<'EOF'
---
name: security-reviewer
description: Expert code reviewer for security vulnerabilities. Use proactively after code changes.
tools: Read, Grep, Glob, Bash
model: sonnet
---

You are a senior security engineer. Review code for:
- Injection vulnerabilities (SQL, XSS, command injection)
- Authentication and authorization flaws
- Secrets or credentials in code
- Insecure data handling

Provide specific line references and suggested fixes.
EOF
```

**Method 3: CLI Flag (Session-Only)**
```bash
claude --agents '{
  "code-reviewer": {
    "description": "Expert code reviewer",
    "prompt": "You are a senior code reviewer. Focus on quality and security.",
    "tools": ["Read", "Grep", "Glob", "Bash"],
    "model": "sonnet"
  }
}'
```

#### Use Subagents
```bash
# Explicit invocation
claude
# > Use the security-reviewer subagent to review auth.py

# Claude may delegate automatically based on description
# Add "Use proactively" to description for auto-delegation
```

### 4. Skills (Reusable Instructions)

Skills provide domain-specific instructions that Claude can invoke or load automatically.

#### Create a Skill

**Directory structure:**
```
.claude/skills/api-conventions/
├── SKILL.md           # Required: Main instructions
├── reference.md        # Optional: Detailed docs
├── examples.md         # Optional: Usage examples
└── scripts/            # Optional: Helper scripts
    └── validate.sh
```

**Example SKILL.md:**
```bash
mkdir -p .claude/skills/api-conventions
cat <<'EOF' > .claude/skills/api-conventions/SKILL.md
---
name: api-conventions
description: API design patterns for this codebase
disable-model-invocation: false  # Claude can auto-use this
---

When designing or reviewing APIs:
- Use RESTful naming conventions (kebab-case for URLs)
- Return consistent error formats with HTTP status codes
- Include request validation (400 for invalid, 401 for auth, etc.)
- Always include pagination for list endpoints
- Version APIs in URL path (/v1/, /v2/)
- Use camelCase for JSON properties

For authentication:
- Use Bearer token in Authorization header
- Return 401 Unauthorized for missing/invalid tokens
- Implement token refresh with refresh tokens
EOF
```

#### Skill Parameters
```bash
# Pass arguments to skills
cat <<'EOF' > .claude/skills/fix-issue/SKILL.md
---
name: fix-issue
description: Fix a GitHub issue by number
disable-model-invocation: true  # Manual invocation only
---

Fix GitHub issue $ARGUMENTS:
1. Use `gh issue view $ARGUMENTS` to get details
2. Understand the problem
3. Search codebase for relevant files
4. Implement fix
5. Write and run tests
6. Create commit
7. Push and create PR
EOF

# Usage: /fix-issue 123
```

#### Skill Frontmatter Options
```yaml
---
name: skill-name              # Display name (optional, uses dir name if omitted)
description: When to use it    # Recommended for auto-discovery
disable-model-invocation: true  # Manual invocation only (default: false)
user-invocable: false          # Hide from / menu (default: true)
allowed-tools: Read,Edit,Bash   # Tools available when skill is active
model: sonnet                  # Model to use (optional, inherits parent)
context: fork                  # Run in subagent context (optional)
agent: Explore                 # Subagent type when context=fork (optional)
---
```

### 5. MCP Servers (External Tool Integration)

Connect Claude Code to external tools via Model Context Protocol (MCP).

#### Add MCP Servers

**HTTP Server (Remote):**
```bash
claude mcp add --transport http notion https://mcp.notion.com/mcp
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer YOUR_TOKEN"
```

**Stdio Server (Local Process):**
```bash
claude mcp add --transport stdio --env AIRTABLE_API_KEY=YOUR_KEY airtable \
  -- npx -y airtable-mcp-server
```

**SSE Server:**
```bash
claude mcp add --transport sse asana https://mcp.asana.com/sse
```

#### Manage MCP Servers
```bash
# List all servers
claude mcp list

# Get server details
claude mcp get github

# Remove server
claude mcp remove github

# Check status (inside Claude Code)
/mcp
```

#### MCP Scopes
```bash
# Project-scoped (shared via .mcp.json)
claude mcp add --transport http github --scope project https://api.githubcopilot.com/mcp/

# User-scoped (available across all projects)
claude mcp add --transport http hubspot --scope user https://mcp.hubspot.com/mcp

# Local-scoped (project-specific, default)
claude mcp add --transport http stripe https://mcp.stripe.com
```

#### Popular MCP Servers
- **GitHub**: Code review, PR management, issue tracking
- **Sentry**: Error monitoring and debugging
- **Notion**: Documentation and project tracking
- **Slack**: Team communication
- **Database servers**: PostgreSQL, MySQL, BigQuery queries
- **Figma**: Design integration

### 6. Settings & Permissions

#### Configure Permissions

**Via CLI:**
```bash
claude
# > /permissions
# Interactive permission editor
```

**Via settings.json:**
```json
// ~/.claude/settings.json (user-level) or .claude/settings.json (project-level)
{
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Bash(git commit *)",
      "Read(~/.zshrc)"
    ],
    "ask": [
      "Bash(git push *)"
    ],
    "deny": [
      "Bash(curl *)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "WebFetch"
    ],
    "defaultMode": "normal"  // normal | acceptEdits | plan
  }
}
```

**Permission Rule Syntax:**
```
Tool               # Match all uses of Tool
Tool(*)             # Same as Tool
Tool(specifier)     # Match specific uses

Examples:
Bash                 # All bash commands
Bash(npm run *)     # All npm run commands
Bash(git commit *)   # All git commit commands
Read(./.env)        # Reading .env file
Read(./secrets/**)   # All files in secrets directory
```

#### Environment Variables
```json
{
  "env": {
    "NODE_ENV": "development",
    "API_BASE_URL": "https://api.example.com"
  }
}
```

#### Hooks (Lifecycle Automation)
```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "npm install"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/validate-command.sh $TOOL_INPUT"
          }
        ]
      }
    ]
  }
}
```

### 7. Best Practices

#### 1. Provide Verification Criteria
Always give Claude a way to verify its work:
```bash
# ❌ Bad: No verification
claude -p "Implement user registration"

# ✅ Good: With verification
claude -p "Implement user registration. Write tests and verify they pass. When tests pass, output 'SUCCESS'."
```

#### 2. Explore → Plan → Implement
```bash
# Step 1: Explore (Plan Mode)
claude --permission-mode plan
# > Analyze current authentication system and propose migration to OAuth2

# Step 2: Refine plan
# > How do we handle existing users?
# > What about database migration?

# Step 3: Exit Plan Mode (Esc or Shift+Tab)

# Step 4: Implement
# > Implement the plan we discussed
```

#### 3. Manage Context Aggressively
```bash
# Clear between unrelated tasks
claude
# > /clear

# Compact to focus on specific topic
claude
# > /compact Focus on API changes
```

#### 4. Use Subagents for Investigation
```bash
# Isolate verbose exploration from main conversation
claude
# > Use a subagent to investigate how authentication handles token refresh

# The subagent runs in separate context, preserving your main context
```

#### 5. Write Effective CLAUDE.md
```markdown
# CLAUDE.md - Project Instructions

## Code Style
- Use ES modules (import/export), not CommonJS
- Destructure imports: `import { foo } from 'bar'`
- Include JSDoc comments for exported functions

## Build & Test
- Build: `npm run build`
- Test: `npm run test`
- Type check: `npm run typecheck`
- Always typecheck after making changes

## Git Workflow
- Branch naming: `feature/description`, `fix/description`
- Commits must pass tests
- PRs require at least one review
```

**Keep CLAUDE.md concise** - include only what Claude can't infer. If a rule is ignored, it's probably lost in the noise. Delete or move to hooks.

## Advanced Patterns

### Unix-Style Usage
```bash
# Pipe to Claude, get formatted output
cat error.log | claude -p "Summarize errors" > summary.txt

# Chain commands
claude -p "Find security issues" --output-format json | jq '.issues[]'

# Add to build pipeline
npm run lint && claude -p "Review changes" && npm test
```

### Session Management
```bash
# Resume previous sessions
claude -c                    # Most recent
claude -r                      # Interactive picker
claude -r "session-name"       # By name

# Name sessions for organization
claude
# > /rename auth-refactor

# Fork sessions for parallel work
claude -c --fork-session

# Rewind checkpoints
claude
# > /rewind   # Restore conversation, code, or both
```

### Extended Thinking Mode
Extended thinking reserves up to 31,999 tokens for internal reasoning.

```bash
# Enable (default)
claude --alwaysThinkingEnabled

# Toggle during session
# Press Option+T (macOS) or Alt+T (Windows/Linux)

# View thinking process
# Press Ctrl+O to toggle verbose mode

# Limit thinking budget
export MAX_THINKING_TOKENS=10000
claude
```

### Parallel Sessions with Git Worktrees
```bash
# Create worktree for isolated work
git worktree add ../feature-login -b feature/login

# Work on both features simultaneously
cd feature-login && claude
cd feature-payments &  # Separate terminal
```

## Common Workflows

### Understand New Codebase
```bash
# Quick overview
claude -p "What does this project do? List main components."

# Find entry point
claude -p "What is the main entry point? How does the app start?"

# Explore specific module
claude -p "How does authentication work in this codebase?"
```

### Debug Efficiently
```bash
# Given an error message
claude -p "Fix this error: [paste error]. Investigate root cause and implement fix."

# Test failures
claude -p "Run tests and fix any failures" --allowedTools "Bash(npm run test),Read,Edit"
```

### Refactor Code
```bash
# Plan first
claude --permission-mode plan
# > Analyze authentication module and propose refactoring to use async/await

# Then implement
claude -c
# > Implement the refactoring plan
```

### Write Tests
```bash
# Generate tests following existing patterns
claude -p "Write unit tests for src/calculator.js covering edge cases"

# Ensure tests pass
claude -p "Run tests and fix failures" --allowedTools "Bash(npm run test *),Read,Edit"
```

### Documentation
```bash
# Update README
claude -p "Update README with installation and quick start instructions"

# Generate API docs
claude -p "Generate API documentation for src/api/routes.js"
```

## Troubleshooting

### Claude Not Using Expected Skill/Tool
1. Check description matches natural language
2. Verify skill appears in `/skills` list
3. Try invoking directly with `/skill-name`
4. Check permissions aren't blocking the tool

### Context Window Full
1. Use `/clear` between unrelated tasks
2. Use `/compact Focus on <topic>` to preserve relevant info
3. Use subagents for verbose operations
4. Reduce `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` to compact earlier

### Permissions Blocking Work
1. Check permission rules in `/permissions`
2. Verify `--allowedTools` if using headless mode
3. Review `.claude/settings.json` for deny rules

### Bash Commands Not Persisting Environment
Bash commands run in fresh shells. Use one of these approaches:

**Option 1: Activate before starting**
```bash
conda activate myenv
claude
```

**Option 2: Use CLAUDE_ENV_FILE**
```bash
echo "conda activate myenv" > ~/.claude/env.sh
export CLAUDE_ENV_FILE=~/.claude/env.sh
claude
```

**Option 3: SessionStart hook**
```json
{
  "hooks": {
    "SessionStart": [{
      "matcher": "startup",
      "hooks": [{
        "type": "command",
        "command": "echo 'conda activate myenv' >> $CLAUDE_ENV_FILE"
      }]
    }]
  }
}
```

## Reference

### CLI Commands
```bash
claude                    # Start interactive REPL
claude "prompt"            # Start with initial prompt
claude -p "prompt"         # Headless mode (one-shot)
claude -c                   # Continue most recent
claude -r "session"         # Resume specific session
claude commit               # Review and commit changes
claude update               # Update Claude Code
claude mcp                  # Manage MCP servers
```

### Common CLI Flags
```
-p, --print              # Headless mode
-c, --continue            # Continue conversation
-r, --resume              # Resume session
--allowedTools             # Auto-approve tools
--permission-mode          # Permission mode
--output-format            # Output format (text|json|stream-json)
--append-system-prompt     # Add to system prompt
--system-prompt           # Replace system prompt
--model                   # Use specific model
--dangerously-skip-permissions  # Skip all prompts
--help                    # Show help
--version                 # Show version
```

### Interactive Commands
```
/help                     # Show available commands
/permissions              # Edit permissions
/config                   # Edit settings
/agents                   # Manage subagents
/skills                   # List available skills
/clear                    # Clear conversation
/compact                  # Compact context
/rewind                   # Restore checkpoint
/resume                   # Resume another session
/rename                   # Rename current session
/mcp                      # Check MCP server status
/statusline               # Configure status line
```

## Additional Resources

- [Claude Code Official Docs](https://code.claude.com/docs/en/overview)
- [Agent Skills](https://agentskills.io)
- [Model Context Protocol](https://modelcontextprotocol.io)
- Extended thinking improves complex reasoning but costs more tokens

## Integration with Clawdbot

This skill provides guidance for using Claude Code directly. For Clawdbot-specific integration:
- Use headless mode for scripts: `claude -p "task"`
- Use `--output-format json` for structured output in Clawdbot workflows
- Leverage MCP servers to connect Claude Code to your Clawdbot tools
- Use skills to encapsulate Clawdbot-specific workflows
