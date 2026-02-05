# Hugo Infrastructure: DevOps Architecture

*How we built a resilient, self-managing AI assistant infrastructure*

---

## Overview

Hugo is a personal AI assistant running on dedicated infrastructure with full operational autonomy. This document describes the architecture decisions, deployment model, and resilience strategy that keeps Hugo running 24/7.

## Architecture Philosophy

**Principle: The AI should be able to manage itself.**

Rather than treating the AI as a stateless API consumer, Hugo operates as a persistent entity with:
- Its own workspace and file system
- Direct access to infrastructure tooling
- Memory that survives across sessions
- The ability to diagnose and (partially) heal itself

## Infrastructure Layout

### Multi-Server Model

| Role | Purpose | Provider |
|------|---------|----------|
| **Production (DE)** | Primary runtime, all user-facing traffic | VPS (EU region) |
| **Development (DE)** | Testing, staging, backup standby | VPS (EU region) |
| **Production (FR)** | Secondary production, additional agents | VPS (EU region) |

All servers run identical software stacks, allowing rapid failover if needed.

### Software Stack

```
┌─────────────────────────────────────────┐
│           User Channels                 │
│ (Telegram, WhatsApp, Discord, Web)      │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│          OpenClaw Gateway               │
│   (Message routing, session mgmt)       │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│          Claude API (Anthropic)         │
│   (Language model inference)            │
└─────────────────────────────────────────┘
```

**OpenClaw** handles:
- Multi-channel message ingestion (Telegram, WhatsApp, webchat)
- Session isolation per user/channel
- Tool execution (shell, browser, APIs)
- Memory and context management

### Workspace Structure

Hugo's "brain" lives in a Git-managed workspace:

```
/workspace/
├── AGENTS.md      # Behavioral rules, access policies
├── SOUL.md        # Personality, communication style
├── USER.md        # Owner preferences
├── TOOLS.md       # API credentials references, device notes
├── MEMORY.md      # Long-term curated memory
├── memory/        # Daily session logs
│   └── YYYY-MM-DD.md
├── assets/        # Images, media
└── references/    # Documentation, style guides
```

Everything is version-controlled. Hugo can read, write, and commit changes to its own configuration.

### Memory Architecture

Hugo wakes up fresh each session but maintains continuity through a file-based memory system:

**Long-term Memory (`MEMORY.md`)**
- Curated knowledge: key facts, preferences, lessons learned
- Owner-only access (never loaded in guest sessions)
- Updated periodically with distilled insights from daily logs

**Daily Logs (`memory/YYYY-MM-DD.md`)**
- Raw operational notes from each day
- Decisions made, tasks completed, context captured
- Searchable via semantic memory_search tool

**Session Context**
- Workspace files (AGENTS.md, SOUL.md, USER.md, TOOLS.md) injected automatically
- Provides behavioral rules, personality, and owner preferences every session
- No manual loading required

**Recall**
- Semantic search across all memory files via embedding-based memory_search
- Targeted snippet retrieval with memory_get
- Enables answering questions about past work, decisions, and preferences

## Operational Model

### Self-Management Capabilities

Hugo can:
- ✅ Read and update its own configuration files
- ✅ Execute shell commands for diagnostics
- ✅ Check service status and logs
- ✅ Commit and push changes to its repository
- ✅ Manage cron jobs for scheduled tasks
- ✅ Install and update its own dependencies (with approval)

Hugo cannot (by policy):
- ❌ Restart core services without owner confirmation
- ❌ Modify authentication credentials directly
- ❌ Access production secrets in plaintext
- ❌ Execute destructive commands without confirmation

### Heartbeat System

A periodic heartbeat allows Hugo to:
1. Check for pending tasks
2. Monitor external services (email, calendar)
3. Perform maintenance (memory cleanup, log rotation)
4. Proactively reach out if something needs attention

The heartbeat runs via cron, triggering a lightweight prompt that Hugo can act on or dismiss.

## Resilience & Recovery

### What We're Protecting Against

| Failure Mode | Mitigation |
|--------------|------------|
| Server crash | Systemd auto-restart, secondary server standby |
| Corrupted memory | Git history allows rollback to any point |
| Lost credentials | Secrets stored in external vault, not on disk |
| Network partition | VPN mesh allows out-of-band access |
| Provider outage | Can migrate to dev server within minutes |

### Disaster Recovery Procedure

**If the production server dies completely:**

1. **Secrets** — Stored in 1Password vault, not on the server. New server authenticates via service account token.

2. **Configuration** — Clone workspace from private GitHub repo. All behavioral rules, memory, and settings restored instantly.

3. **State** — Session transcripts stored locally but non-critical. Hugo can resume without them (just loses recent conversation context).

4. **Channels** — Re-authenticate messaging channels (Telegram bot token from vault, WhatsApp re-link via QR).

**Recovery time objective: < 30 minutes** for full restoration on a fresh VPS.

### Backup Strategy

**Three layers of protection:**

#### 1. Weekly System Backup
- Full server snapshot via hosting provider
- Captures entire system state: OS, packages, configs, data
- Disaster recovery baseline — can restore entire server from scratch

#### 2. Daily Prod → Dev Sync
- Runs at **4 AM UTC** via cron (`/scripts/backup.sh`)
- Syncs workspace to dev server via rsync over Tailscale
- Syncs OpenClaw config and session metadata
- Auto-commits and pushes workspace changes to Git
- Log: `/var/log/backup.log`

#### 3. Real-time Version Control
- All workspace files in private Git repo
- Significant changes committed automatically during backup
- Manual commits for important updates
- Full history enables rollback to any point

| Component | Backup Method | Frequency |
|-----------|---------------|-----------|
| Full server | Provider snapshot | Weekly |
| Workspace files | rsync to dev + Git | Daily (4 AM) |
| OpenClaw config | rsync to dev | Daily (4 AM) |
| Secrets | 1Password vault | Real-time sync |
| Session transcripts | rsync to dev (recent) | Daily (4 AM) |

## Development Workflow

### Testing Changes

1. Deploy changes to **dev** server first
2. Run integration tests (channel connectivity, tool access)
3. Promote to **prod** after validation

### Updating Hugo

Hugo can update itself:
```
# Hugo runs this when asked to update
openclaw update
```

The update process:
1. Pulls latest from npm registry
2. Validates configuration compatibility
3. Restarts gateway with new version
4. Pings owner to confirm success

### Rolling Back

If an update breaks things:
1. Git revert workspace changes
2. Reinstall previous OpenClaw version
3. Restart gateway

Hugo can perform steps 1 and 3 autonomously; step 2 requires owner confirmation.

## Monitoring

### What Hugo Watches

- Gateway process health (systemd status)
- Channel connectivity (message round-trip)
- Disk space and memory usage
- API quota consumption

### Alerting

Hugo proactively notifies the owner via Telegram when:
- A channel disconnects
- Disk space runs low
- An API returns repeated errors
- A scheduled task fails

### Weekly Security Scan

A recurring security audit runs weekly to verify infrastructure posture:

**Port Exposure Check:**
- Verify Docker ports are bound to localhost only
- Test that internal services are not reachable from internet
- Confirm firewall rules are active

**Service Health:**
- OpenClaw gateway running
- Cloudflare tunnels active
- Docker containers healthy

**Credential Audit:**
- GitHub tokens are fine-grained PATs (not OAuth)
- No plaintext secrets in config files
- 1Password integration functioning

The scan checklist lives in the private ops repo and can be executed by Claude on demand.

## Repository Organization

### Topic-Based Classification

All infrastructure repositories use GitHub topics for discoverability:

| Topic | Purpose |
|-------|---------|
| `vps-ops` | Core infrastructure: servers, agents, skills |
| `contains-secrets` | Warning: repos with committed .env files |
| `mcp` | MCP server integrations |
| `trading` | Trading bot infrastructure |
| `experiment` | Experimental/prototype projects |

**Searching by topic (GitHub website):**
1. Go to Repositories tab
2. Type `topic:vps-ops` in search
3. Or click any topic tag to filter

### Archiving Policy

Repos inactive for 12+ months are archived:
- Read-only, still visible
- Marked with "Archived" badge
- Can be unarchived if needed later

## Lessons Learned

1. **Git-managed workspace is essential** — Being able to rollback any change to Hugo's "personality" or rules has saved us multiple times.

2. **Secrets must live elsewhere** — The AI has broad file access. Secrets on disk = secrets exposed. External vault is mandatory.

3. **Two servers beats one** — Even if you rarely use the second server, having it ready cuts recovery time dramatically.

4. **Let the AI manage itself (within limits)** — Hugo fixing its own config issues is faster than waiting for human intervention, as long as guardrails exist.

5. **Stateless is a myth** — Persistent memory and context make the AI dramatically more useful. Design for state from day one.

6. **Explicit behavioral rules beat implicit understanding** — After a January 2026 incident where Hugo leaked infrastructure details and confused session routing, we learned that "don't share secrets" isn't enough. Rules must enumerate specific prohibited behaviors: no IP disclosure, no message relaying, explicit session routing via tools.

7. **Multi-user means multi-threat** — Family members aren't automatically trusted. Each user session must be isolated, and the AI must verify identity by session key, not by claims.

---

*Last updated: February 5, 2026*

*This architecture has been running in production since January 2026, handling hundreds of messages daily across multiple channels.*

*Key updates:*
- *January 29, 2026: Security hardening after session confusion incident*
- *January 30, 2026: nginx → Caddy migration*
- *February 5, 2026: Third server (prod-fr), fine-grained GitHub PATs, weekly security scans, repository organization*
