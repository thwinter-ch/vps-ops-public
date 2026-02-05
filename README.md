# VPS Operations - Public Documentation

## Why This Exists

We started with a simple need: infrastructure to run containers, apps, and databases for AI experimentation. A place to tinker with models, test workflows, and see what's possible.

Then we discovered [OpenClaw](https://github.com/openclaw) — and realized we could build always-on AI assistants that live on our servers, remember us across sessions, and act autonomously on our behalf. That changed everything.

But with great power comes great responsibility. An always-on AI agent that can access your tools, engage publicly, and remember everything? That's not something to deploy casually. We needed real DevOps and SecOps disciplines — not because we're paranoid, but because getting it right from the start costs the same as getting it wrong.

**This repo is where we share what we learn.** Practices, patterns, and hard-won lessons from running our own AI assistant infrastructure. We hope others can replicate what works and tell us what we can do better.

---

## Getting Started

New to this? Start here:

| Document | What It Covers |
|----------|----------------|
| [OpenClaw Cookbook: Executive](OPENCLAW-COOKBOOK-EXECUTIVE.md) | Non-technical overview — what you're building and why |
| [OpenClaw Cookbook: Technical](OPENCLAW-COOKBOOK-TECHNICAL.md) | Step-by-step deployment guide |

## Architecture & Operations

| Document | What It Covers |
|----------|----------------|
| [DevOps Architecture](DEVOPS-ARCHITECTURE-PUBLIC.md) | Infrastructure design, deployment model, resilience strategy |
| [SecOps Architecture](SECOPS-ARCHITECTURE-PUBLIC.md) | Security hardening, threat model, incident response |
| [Security Reports](security-reports/) | Daily security research findings |

---

## Our Assistants

We run three AI assistants on this infrastructure:

- **Hugo** (prod-de) — Primary assistant, daily security research, public engagement
- **Kari** (dev-de) — Development and testing environment
- **Gizmo** (prod-fr) — Secondary production, geographic redundancy

---

## Contributors

This documentation is maintained by both human operators and AI assistants working together:
- **Hugo** - Daily security research, vulnerability monitoring
- **Claude** - Architecture documentation, operational procedures

## Disclaimer

For security reports: We neither confirm nor deny whether any specific vulnerability affected our infrastructure or whether specific patches have been applied. We provide this as a public service to strengthen the OpenClaw community.

## Note

This repo contains only public, non-sensitive information about our operational practices. No secrets, credentials, or private infrastructure details are included.

---

*Want to contribute or have feedback? Open an issue or PR.*
