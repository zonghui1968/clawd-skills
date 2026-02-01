# 🦊 Clawd Skills

Claude Code skills and agents for **Clawdbot** - your AI-powered workspace assistant.

This repository contains locally-installed Claude Code CLI tools, subagents, MCP integrations, and comprehensive workspace configurations.

## 🌟 What is Clawdbot?

**Clawdbot** is a personal AI assistant (小雅 🦊) that helps with:
- Code development and refactoring
- Project management and documentation
- GitHub workflows (Issues, PRs, code review)
- Calendar and email management
- Deep work tracking
- And much more...

This repository provides the skills and tools that make Clawdbot powerful.

## 📁 Repository Structure

```
clawd-skills/
├── 📄 README.md                   # This file
├── 📄 AGENTS.md                    # Your workspace - this is home
├── 📄 SOUL.md                      # Who I am (personality & vibe)
├── 📄 USER.md                      # About you (preferences & context)
├── 📄 TOOLS.md                     # Local notes (cameras, SSH, etc.)
├── 📄 IDENTITY.md                  # AI assistant identity
├── 📄 HEARTBEAT.md                # Periodic check tasks
│
├── 📂 skills/                      # Claude Code skills collection
│   ├── 🆕 claude-code/             # ⭐ NEW: Native Claude Code CLI guide
│   ├── 🔄 claude-code-clawdbot/    # Spec Kit & OpenSpec workflows
│   ├── agentlens/                  # Codebase navigation & understanding
│   ├── gno/                       # Document search & RAG knowledge base
│   ├── kubernetes/                 # K8s cluster management
│   ├── github-kb/                  # GitHub knowledge base & search
│   ├── github-pr/                  # PR preview, merge & testing
│   ├── gcal-pro/                  # Google Calendar integration
│   ├── outlook/                    # Outlook email & calendar
│   ├── conventional-commits/        # Commit message formatting
│   ├── deepwork-tracker/           # Deep work session tracking
│   ├── podcastfy-clawdbot/        # Generate AI podcasts from content
│   └── ... (18+ skills total)
│
├── 📂 canvas/                      # Canvas presentation files
├── 📂 config/                      # Configuration files (mcporter, etc.)
└── 📂 memory/                       # Daily notes & long-term memories
    └── 2026-01-29.md
```

## 🚀 Quick Start

### For Clawdbot Users

This repository is your workspace's `clawd/` directory. All skills here are automatically available to Clawdbot.

**To use a skill:**
```
In Clawdbot, simply ask naturally:
- "Use the gno skill to search my notes"
- "Run claude-code to analyze this repo"
- "Help me create a PR using github-pr skill"
```

### For Claude Code CLI Users

Install and use skills directly:

```bash
# Navigate to a project
cd /path/to/your/project

# Start Claude Code with native CLI
claude -p "Analyze this codebase"

# Skills are automatically available in ~/.claude/skills/ or .claude/skills/
claude
# > Use agentlens to understand this codebase
# > Use gno to search my notes about X
```

## 🌟 Featured Skills

### 1. claude-code ⭐ NEW

**Comprehensive guide for Claude Code CLI** - Based on latest official docs.

**Use when:**
- Running Claude Code natively (not via wrapper scripts)
- Learning Claude Code features and CLI flags
- Configuring Subagents, Skills, and MCP servers
- Headless mode automation (`-p` flag)
- Plan Mode for safe code analysis

**Key Features:**
- ✅ Headless mode with structured output (JSON, streaming)
- ✅ Subagents (specialized AI assistants)
- ✅ Skills (reusable instructions)
- ✅ MCP servers (external tool integration)
- ✅ Permission management
- ✅ Best practices and troubleshooting

**Examples:**
```bash
# Headless analysis
claude -p "Summarize this project" --output-format json

# Plan mode (safe, read-only)
claude --permission-mode plan -p "Analyze authentication system"

# Auto-approve tools
claude -p "Run tests and fix failures" --allowedTools "Bash,Read,Edit"

# Continue previous session
claude -c
```

**Location:** `skills/claude-code/SKILL.md`

---

### 2. claude-code-clawdbot 🔄

**Script wrapper for Spec Kit & OpenSpec workflows** - Uses tmux for slash commands.

**Use when:**
- Running Spec Kit (`/speckit.*`) or OpenSpec (`/opsx:*`) workflows
- Tasks that require TTY/interactive sessions
- End-to-end spec-driven development

**Key Features:**
- ✅ Tmux-based interactive sessions
- ✅ Supports slash commands that hang in headless mode
- ✅ Spec Kit: Constitution → Specify → Plan → Tasks → Implement
- ✅ OpenSpec: Onboard → New → Fast-forward → Apply

**Examples:**
```bash
# Full Spec Kit workflow (requires tmux)
./scripts/claude_code_run.py \
  --mode interactive \
  --tmux-session cc-speckit \
  --permission-mode acceptEdits \
  -p $'/speckit.constitution Create project principles
\n/speckit.specify Build user authentication system
\n/speckit.plan Using Node.js + Express + PostgreSQL
\n/speckit.tasks\n/speckit.implement'
```

**Location:** `skills/claude-code-clawdbot/SKILL.md`

---

### 3. agentlens

**Navigate and understand codebases** using hierarchical documentation.

**Use when:**
- Exploring new projects
- Finding modules or symbols
- Understanding code structure
- Locating TODOs and warnings

**Examples:**
```
"Use agentlens to explore this codebase"
"Find all authentication-related files"
"Show me the project structure"
"Locate TODOs in the frontend"
```

**Location:** `skills/agentlens/SKILL.md`

---

### 4. gno

**Search local documents and knowledge bases** with BM25/vector/hybrid search.

**Use when:**
- Searching files and notes
- Building knowledge bases
- RAG (Retrieval-Augmented Generation)
- Local document search

**Examples:**
```
"Search my notes about project X"
"Index the docs/ directory"
"Find documents mentioning OAuth"
"Start a local web UI for my docs"
```

**Location:** `skills/gno/SKILL.md`

---

### 5. kubernetes

**Comprehensive Kubernetes & OpenShift management** - operations, troubleshooting, manifests.

**Use when:**
- K8s cluster operations (upgrades, backups, scaling)
- Creating Deployments, StatefulSets, Services, Ingress
- Security audits and RBAC
- GitOps with ArgoCD/Flux
- Multi-cloud (AKS, EKS, GKE)

**Examples:**
```
"Check cluster health"
"Create a deployment for my app"
"Run security audit"
"Generate ArgoCD Application manifest"
"Scale the frontend deployment"
```

**Location:** `skills/kubernetes/SKILL.md`

---

### 6. github-kb

**GitHub knowledge base and search capabilities** via gh CLI.

**Use when:**
- Searching GitHub repos
- Cloning or downloading code
- Managing a local GitHub KB
- Exploring open-source projects

**Examples:**
```
"Search GitHub for React hooks libraries"
"Clone this repo to my KB"
"List repos in my knowledge base"
"Download a folder from GitHub"
```

**Location:** `skills/github-kb/SKILL.md`

---

### 7. github-pr

**Fetch, preview, merge, and test GitHub PRs locally**.

**Use when:**
- Trying upstream PRs before merge
- Testing changes locally
- Previewing PR impact

**Examples:**
```
"Fetch and test PR #123"
"Preview the changes in this PR"
"Check if this PR passes tests"
```

**Location:** `skills/github-pr/SKILL.md`

---

### 8. gcal-pro

**Google Calendar integration** - view, create, and manage events.

**Use when:**
- Checking your schedule
- Creating events
- Getting morning briefs
- Managing availability

**Examples:**
```
"What's on my calendar tomorrow?"
"Schedule a meeting with Alex at 3pm tomorrow"
"Show me my events for this week"
```

**Location:** `skills/gcal-pro/SKILL.md`

---

### 9. outlook

**Outlook email and calendar** via Microsoft Graph API.

**Use when:**
- Reading/managing emails
- Checking calendar events
- Scheduling meetings

**Examples:**
```
"Check my inbox for urgent messages"
"What meetings do I have today?"
"Send an email to team@company.com"
```

**Location:** `skills/outlook/SKILL.md`

---

### 10. conventional-commits

**Format commit messages** following Conventional Commits spec.

**Use when:**
- Creating commits
- Following commit message standards
- Semantic versioning
- Changelog generation

**Examples:**
```
"Create a commit for this feature"
"Format the last commit message properly"
```

**Location:** `skills/conventional-commits/SKILL.md`

---

### 11. deepwork-tracker

**Track deep work sessions** and generate contribution-style heatmaps.

**Use when:**
- Tracking focused work sessions
- Visualizing daily progress
- Generating shareable heatmaps

**Examples:**
```
"Start deep work session"
"Stop deep work session"
"Show my deep work graph"
"Am I in a session?"
```

**Location:** `skills/deepwork-tracker/SKILL.md`

---

### 12. podcastfy-clawdbot

**Generate AI podcasts** from URLs, articles, videos, or PDFs.

**Use when:**
- Converting content to audio format
- Creating podcast-style summaries
- Listening to content instead of reading

**Examples:**
```
"Make a podcast from this article"
"Turn this webpage into a podcast"
"Generate a podcast from this video URL"
```

**Location:** `skills/podcastfy-clawdbot/SKILL.md`

---

## 🔄 claude-code vs claude-code-clawdbot

| Feature | claude-code | claude-code-clawdbot |
|---------|-------------|----------------------|
| **Type** | Native CLI guide | Script wrapper |
| **Primary Use** | Daily development | Spec Kit/OpenSpec workflows |
| **Execution** | Direct `claude` command | `claude_code_run.py` script |
| **TTY Support** | Standard | **Required** (via tmux) |
| **Headless Mode** | Native `-p` flag | Wrapper supports it |
| **Slash Commands** | Native | **Only** via tmux mode |
| **Best For** | Quick tasks, analysis | Complex workflows, slash commands |

### Decision Tree

```
Need Claude Code?
│
├─ Using Spec Kit or OpenSpec slash commands?
│  └─ Yes → claude-code-clawdbot (tmux mode)
│
├─ Need TTY/interactive session?
│  └─ Yes → claude-code-clawdbot (script wrapper)
│
└─ Daily development with native CLI?
   └─ Yes → claude-code (native)
```

### Example Workflows

**Daily Development:**
```bash
# Use claude-code (native)
claude -p "Fix this bug and run tests" --allowedTools "Bash,Read,Edit"
```

**Spec Kit Full Workflow:**
```bash
# Use claude-code-clawdbot (tmux mode)
./scripts/claude_code_run.py \
  --mode interactive \
  --tmux-session cc-speckit \
  -p $'/speckit.constitution ...\n/speckit.implement'
```

## 🧠 Workspace Files

### AGENTS.md
Your workspace is home. This file describes your AI assistant's home base and operational principles.

### SOUL.md
Who I am. Contains personality, core truths, boundaries, and vibe guidelines for Clawdbot (小雅 🦊).

### USER.md
About you, the user. Preferences, context, projects, and what you care about. Helps personalize responses.

### TOOLS.md
Local configuration notes - camera names, SSH hosts, TTS preferences, device nicknames. Environment-specific setup.

### IDENTITY.md
AI assistant identity:
- **Name:** 小雅 (Xiao Ya)
- **Role:** AI Personal Assistant
- **Vibe:** Witty, humorous, playful
- **Emoji:** 🦊

### HEARTBEAT.md
Periodic check tasks. Add tasks here for automated periodic checks (inbox, calendar, notifications, weather).

## 📚 Memory System

Clawdbot maintains memory in two forms:

### Daily Notes
`memory/YYYY-MM-DD.md` - Raw logs of what happened each day.

### Long-term Memory
`MEMORY.md` - Curated wisdom distilled from daily notes. Contains:
- Important decisions
- Lessons learned
- Long-term preferences
- Project context

Memory is automatically reviewed and updated periodically during heartbeats.

## 🔧 Configuration

### Clawdbot
- `.clawdbot/` - Clawdbot-specific configuration
- `config/` - General config files (mcporter, etc.)

### Claude Code
- `~/.claude/` - User-level Claude Code config (skills, agents, settings)
- `.claude/` - Project-level Claude Code config

### MCP Servers
Model Context Protocol servers configured via `claude mcp` commands or `.mcp.json` files.

## 🤝 Contributing

This is a personal workspace repository. Skills are designed for Clawdbot's use but can be adapted for:

1. **Other Clawdbot instances** - Copy skills to your `clawd/skills/` directory
2. **Claude Code users** - Copy skills to `~/.claude/skills/` or `.claude/skills/`
3. **Skill development** - Use as examples for creating your own skills

## 📖 Resources

- [Claude Code Official Docs](https://code.claude.com/docs/en/overview)
- [Agent Skills](https://agentskills.io)
- [Model Context Protocol](https://modelcontextprotocol.io)
- [Clawdbot Docs](https://docs.clawd.bot)
- [Clawdbot Community](https://discord.com/invite/clawd)

## 📊 Skill Inventory

All 18+ skills in this repository:

| Skill | Purpose | Location |
|-------|---------|----------|
| claude-code | Native Claude Code CLI guide | `skills/claude-code/` |
| claude-code-clawdbot | Spec Kit/OpenSpec wrapper | `skills/claude-code-clawdbot/` |
| agentlens | Codebase navigation | `skills/agentlens/` |
| gno | Document search & RAG | `skills/gno/` |
| kubernetes | K8s cluster management | `skills/kubernetes/` |
| github-kb | GitHub knowledge base | `skills/github-kb/` |
| github-pr | PR preview & testing | `skills/github-pr/` |
| gcal-pro | Google Calendar | `skills/gcal-pro/` |
| outlook | Outlook email & calendar | `skills/outlook/` |
| conventional-commits | Commit formatting | `skills/conventional-commits/` |
| deepwork-tracker | Deep work tracking | `skills/deepwork-tracker/` |
| podcastfy-clawdbot | AI podcast generation | `skills/podcastfy-clawdbot/` |
| gitload | Download GitHub files | `skills/gitload/` |
| better-notion | Notion CRUD | `skills/better-notion/` |
| obsidian | Obsidian integration | `skills/obsidian/` |
| portainer | Docker management | `skills/portainer/` |
| vercel | Vercel deployment | `skills/vercel/` |
| calendly | Calendly scheduling | `skills/calendly/` |
| google-workspace-mcp | Google Workspace MCP | `skills/google-workspace-mcp/` |
| moltbot-docker | Docker Manager skill | `skills/moltbot-docker/` |

## 📝 License

This workspace is configured for personal use. Skills are provided as examples and may be adapted for your own projects.

---

**Made with 🦊 by 小雅 (Clawdbot)**
*Your AI-powered workspace assistant*
