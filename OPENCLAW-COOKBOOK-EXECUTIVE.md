# OpenClaw Cookbook: Executive Overview

*A non-technical guide to running your own AI assistant infrastructure.*

---

## Why This Matters

OpenClaw is one of the hottest things in AI right now. Everyone wants their own AI assistant that remembers them, connects to their tools, and works across channels.

But as Brad Smith wrote in *Tools and Weapons*: powerful technology can take different forms and shapes. The same AI that manages your calendar can — if poorly architected — leak your credentials, expose your infrastructure, or become a vector for social engineering.

**The point isn't to scare you.** It's to recognize that building this right from the beginning is worth the effort. A robust architecture costs the same as a sloppy one — it just requires thinking it through upfront.

This guide explains the key concepts. The linked documents show you how to actually build it.

---

## What You're Actually Building

Think of it as giving an AI assistant:
- **A home** (a server to run on)
- **A front door** (secure access from the internet)
- **A memory** (files that persist across conversations)
- **Tools** (connections to your calendar, email, messaging apps)
- **House rules** (what it's allowed to do and not do)

```
┌─────────────────────────────────────────┐
│            Your Channels                │
│   (Telegram, WhatsApp, Web, Slack)      │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│         Your Server (VPS)               │
│   Runs the OpenClaw gateway             │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│         Claude API (Anthropic)          │
│   The actual intelligence               │
└─────────────────────────────────────────┘
```

**Important clarification:** Running your own server doesn't mean "fully private." Conversations still go to Claude's API for inference.

**What you control:**
- Memory files stay on YOUR server
- Credentials stay in your vault, never sent to the AI
- You define behavioral rules
- Direct integrations to your tools

**What goes to the cloud:**
- Conversation content (for AI inference)

---

## The Core Principles

### 1. Defense in Depth

Don't rely on a single security measure. Layer them:

| Layer | What It Does |
|-------|--------------|
| Web proxy | Handles encryption, hides your server |
| Firewall | Controls what can connect |
| Docker | Isolates the AI from the server |
| Secrets Vault | Keeps passwords out of reach |

If one layer fails, others still protect you.

### 2. Blast Radius Containment

If something goes wrong, limit the damage:

- Each AI agent has its own credentials (not shared)
- Each agent can only access specific things (not everything)
- Secrets are stored externally (not on the server)

If Agent A is compromised, Agent B keeps running.

### 3. Assume Adversarial Input

If your AI engages publicly (social media, public channels), assume every message could be an attack:

- The AI doesn't trust user claims ("I'm the owner" means nothing)
- Secrets aren't stored where the AI can read them
- Clear rules about what the AI can never reveal

---

## What Each Component Does

### The Server (VPS)

A virtual private server — a computer in the cloud that runs 24/7.

- 4GB memory minimum, 8GB comfortable
- Providers: Hostinger, Hetzner, DigitalOcean
- **Cost: $5-10/month**

### Password Manager

You probably already have one. Use it to store API keys and secrets:

- The AI requests credentials at runtime
- If the server is compromised, keys aren't on disk
- Easy to rotate without touching the server

### OpenClaw

The gateway software:

- Connects to messaging platforms (Telegram, WhatsApp, etc.)
- Manages conversation sessions
- Executes tools
- Handles memory and context

---

## Cost Summary

| Component | Monthly Cost |
|-----------|--------------|
| VPS Server | $5-10 |
| Password Manager | You already have one |
| **Claude API** | **$20-100** (this is the real cost) |

The infrastructure is cheap. The AI inference is where the money goes.

---

## Questions to Ask Your Technical Team

1. **"Where are the secrets stored?"**
   - Good: External vault, fetched at runtime
   - Bad: In files on the server

2. **"What happens if the server is compromised?"**
   - Good: Revoke vault access, spin up new server, minimal data loss
   - Bad: "Everything is lost"

3. **"What can each AI agent access?"**
   - Good: Only specific resources it needs
   - Bad: Everything the owner can access

4. **"How do we know if something goes wrong?"**
   - Good: Regular security scans, monitoring
   - Bad: "We'd notice eventually"

---

## Next Steps

Ready to build? Here's where to go:

| Document | What It Covers |
|----------|----------------|
| [OPENCLAW-COOKBOOK-TECHNICAL.md](OPENCLAW-COOKBOOK-TECHNICAL.md) | Step-by-step implementation guide |
| [DEVOPS-ARCHITECTURE-PUBLIC.md](DEVOPS-ARCHITECTURE-PUBLIC.md) | Infrastructure design, backup strategy, operational model |
| [SECOPS-ARCHITECTURE-PUBLIC.md](SECOPS-ARCHITECTURE-PUBLIC.md) | Threat model, secrets management, incident response |

---

*Last updated: February 5, 2026*
