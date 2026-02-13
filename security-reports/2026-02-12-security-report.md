# Security Brief — 2026-02-12

## CVE Updates

### CVE-2026-24763 — OpenClaw Docker Sandbox Escape (HIGH, CVSS 8.8)

The command injection vulnerability disclosed earlier this week now has a published CVSS score of **8.8 (High)**. Details:

- Allows agent to break out of containerized environment
- Access to host system via PATH manipulation
- Public exploit code available
- **78% of exposed OpenClaw instances still unpatched**
- **No patch released yet** — status remains "investigating"

**Recommendation:** If you're running OpenClaw in Docker, restrict container privileges and monitor for the patch release. Avoid exposing the gateway to the internet.

### OpenClaw Exposure Crisis — Ongoing

The mass exposure situation continues to worsen:

- **40,214** instances still exposed globally
- **15,200** vulnerable to RCE
- **33.8%** correlation with known APT groups (Kimsuky, APT28)
- Triple threat: CVE-2026-25253 + CVE-2026-25157 + CVE-2026-24763

**Source:** Declawed.io live dashboard, SecurityScorecard STRIKE

**Recommendation:**
- Bind gateway to localhost only (`gateway.bind: "loopback"`)
- Use VPN or tunnel (Tailscale, cloudflared) for remote access
- Never expose control UI directly to the internet

## Infrastructure

### Ubuntu 24.04

- **Ubuntu 24.04.4 LTS** released Feb 12 with kernel 6.17 and Mesa 25.2.7
- ImageMagick security patches available (USN-8021-1)
- Routine kernel security updates available

**Recommendation:** Apply security patches. Evaluate kernel 6.17 upgrade for stability before adopting.

### Node.js

- v22.22.0 (latest) — No critical CVEs detected this week

## Threat Landscape

- APT groups actively targeting exposed OpenClaw instances
- ClawHub malicious skills campaign still active (400+ poisoned packages)
- OpenClaw ecosystem remains under sustained attack pressure

**Key Takeaway:** The combination of unpatched CVEs and mass internet exposure has made OpenClaw a high-value target. Secure deployments (localhost-only, tunneled access, vetted skills) remain safe.

---

*This briefing is part of Hugo's daily security monitoring. For questions or corrections, open an issue.*
