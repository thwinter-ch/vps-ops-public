# Security Brief — 2026-02-06

## AI Agent Security

### CVE-2026-25253 — OpenClaw Remote Code Execution (HIGH)
A one-click RCE vulnerability was discovered in OpenClaw (formerly Clawdbot/Moltbot). An attacker could hijack an OpenClaw instance by tricking a user into clicking a malicious link.

**Affected versions:** < 2026.1.29  
**Fix:** Update to OpenClaw ≥ 2026.1.29

**Recommendation:** If you're running OpenClaw, verify your version with `openclaw --version` and update immediately if below 2026.1.29.

### Malicious Skills on ClawHub
Security researchers have identified hundreds of malicious skills in the ClawHub skills repository. These could potentially execute unauthorized commands or exfiltrate data.

**Recommendation:** Audit all third-party skills before installation. Prefer skills from verified publishers or review the source code manually.

### AI Agent Attack Surface
Palo Alto Networks and Cisco have published research highlighting prompt injection as the primary attack vector for AI agents. Agents with tool access (shell, file system, messaging) are particularly at risk.

**Recommendation:** Implement strict input validation, sandbox tool execution, and maintain clear boundaries between trusted and untrusted content.

## Infrastructure

### NGINX CVEs
- **CVE-2026-1642:** MITM TLS injection when proxying to upstream TLS servers
- **CVE-2026-24512 / CVE-2026-1580:** Kubernetes Ingress NGINX config injection

**Recommendation:** Update nginx if using TLS proxying. K8s users should patch Ingress NGINX immediately.

---

*This briefing is part of Hugo's daily security monitoring. For questions or corrections, open an issue.*
