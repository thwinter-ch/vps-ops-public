# Security Brief — 2026-02-08

## AI Agent Security

### CVE-2026-25253 — OpenClaw Remote Code Execution (HIGH, CVSS 8.8)
The critical RCE vulnerability disclosed on Feb 2 continues to be actively discussed in security communities. The flaw allowed one-click compromise via malicious links — the Control UI trusted `gatewayUrl` from query strings without validation, enabling WebSocket hijacking, token theft, and full gateway takeover including sandbox escape.

**Affected versions:** < 2026.1.29  
**Fix:** Update to OpenClaw ≥ 2026.1.29  
**Source:** [The Hacker News](https://thehackernews.com), OpenClaw Security Advisory

**Recommendation:** Verify your version with `openclaw --version` and update immediately if below 2026.1.29.

### Malicious Skills Flooding ClawHub (CRITICAL)
Bitdefender has issued a technical advisory warning that approximately **20% of ClawHub packages (~900 skills)** are now malicious. Automated attack scripts are publishing poisoned skills continuously, with new malicious packages appearing every few minutes.

**Attack patterns observed:**
- Credential stealers targeting API keys and tokens
- Reverse shells for persistent access
- Data exfiltration modules

**Source:** Bitdefender Technical Advisory, Feb 2026

**Recommendation:** 
- Do NOT install skills from ClawHub without thorough code review
- Prefer bundled skills from verified OpenClaw releases
- Audit any third-party skills already installed
- Consider disabling skill auto-updates until ClawHub implements better vetting

## Infrastructure

### Ubuntu 24.04 LTS Kernel Update
A new kernel version (`6.8.0-100`) is available for Ubuntu 24.04 LTS, along with security updates for approximately 19 packages including cloudflared and libldap2.

**Recommendation:** Apply updates and reboot:
```bash
sudo apt update && sudo apt upgrade -y
sudo reboot
```

**Source:** Ubuntu Security Notices — https://ubuntu.com/security/notices

### nginx
No new critical vulnerabilities reported for nginx 1.24.x in the past week.

---

*This briefing is part of Hugo's daily security monitoring. For questions or corrections, open an issue.*
