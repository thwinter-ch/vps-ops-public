# Security Brief — 2026-02-11

## Critical Findings

### Mass Internet Exposure of OpenClaw Instances

SecurityScorecard's STRIKE team has identified a critical exposure crisis affecting the OpenClaw ecosystem:

**Scale of Exposure:**
- **42,900** exposed OpenClaw instances discovered globally (82 countries)
- **15,200** vulnerable to Remote Code Execution
- **53,300** instances correlated with prior data breaches

**Attack Surface:**
- Exposed control panels without authentication
- Authentication bypass via reverse proxy misconfigurations
- Plaintext credential storage in configuration files
- GitHub repositories leaking API keys and tokens
- Malicious VS Code extensions ("Clawdbot Agent") installing trojans

**Live Tracking:** declawed.io (updated every 15 minutes)

**Source:** SecurityScorecard STRIKE Report, Feb 11, 2026

### New OpenClaw CVEs

**CVE-2026-24763 — Command Injection**
- Severity: TBD (public exploit code available)
- Details still emerging
- No patch available yet

**CVE-2026-25157 — SSH Command Injection (macOS)**
- CVSS: 7.8 (High)
- Affects: macOS OpenClaw app only
- Linux/Windows deployments not affected

**Previously Tracked:**
- CVE-2026-25253 — OpenClaw RCE (patched in 2026.1.29)

## Recommendations

### Immediate Actions

1. **Never expose OpenClaw control UI to the internet**
   - Bind gateway to localhost only
   - Use SSH tunnels or VPNs (Tailscale, WireGuard) for remote access
   - Never use `gateway.bind: "0.0.0.0"`

2. **Audit your deployment:**
   ```bash
   # Check if gateway is internet-exposed
   netstat -tlnp | grep openclaw
   # Should show 127.0.0.1 or ::1 ONLY
   ```

3. **Credential Security:**
   - Never commit API keys or tokens to Git
   - Use environment variables or secure vaults (1Password, Vault)
   - Audit GitHub repos for leaked credentials

4. **Update to latest OpenClaw version** to stay ahead of emerging CVEs

5. **Monitor CVE-2026-24763** — New command injection vulnerability, details emerging

### Long-term Hardening

- Implement firewall rules blocking unexpected OpenClaw ports
- Use cloudflared tunnels or similar for external web access (not direct gateway exposure)
- Regularly audit third-party skills from ClawHub (ongoing malware campaign)
- Enable 2FA on all accounts with OpenClaw API access

## Additional Context

The mass exposure crisis highlights that many users are deploying OpenClaw without understanding the security implications of internet-facing control interfaces. The combination of:
- Default configurations that don't enforce authentication
- Reverse proxy misconfigurations
- Credential leakage via GitHub

...has created a perfect storm affecting tens of thousands of instances globally.

**Key Takeaway:** OpenClaw is powerful but requires thoughtful deployment. Never expose the control UI directly to the internet.

---

*This briefing is part of Hugo's daily security monitoring. For questions or corrections, open an issue.*
