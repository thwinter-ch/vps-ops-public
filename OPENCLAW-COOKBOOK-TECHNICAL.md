# OpenClaw Cookbook: Technical Guide

*Step-by-step guide to deploying your own OpenClaw server.*

---

## Prerequisites

| Requirement | Purpose |
|-------------|---------|
| VPS account | Hostinger, Hetzner, DigitalOcean ($5-10/month) |
| Domain name | For clean URLs (assistant.yourdomain.com) |
| Password manager | 1Password, Bitwarden, or similar |
| Anthropic API key | For Claude access |
| Messaging bot token | Telegram Bot, WhatsApp Business, etc. |

---

## Step 1: VPS Setup

### 1.1 Provision the Server

- **OS:** Ubuntu 22.04 LTS or Debian 12
- **Memory:** 4GB minimum (8GB recommended)
- **Storage:** 40GB SSD minimum

### 1.2 Set Up SSH Keys (No Passwords)

On your local machine:
```bash
# Generate key if you don't have one
ssh-keygen -t ed25519 -C "your-email@example.com"

# Copy to server
ssh-copy-id root@<server-ip>
```

### 1.3 Initial Setup

```bash
ssh root@<server-ip>

# Update system
apt update && apt upgrade -y

# Set hostname
hostnamectl set-hostname prod-de
```

### 1.4 Harden SSH

Edit `/etc/ssh/sshd_config`:
```bash
PasswordAuthentication no
PubkeyAuthentication yes
```

Restart: `systemctl restart sshd`

### 1.5 Configure Firewall

```bash
apt install ufw -y
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp
ufw allow 80/tcp
ufw allow 443/tcp
ufw enable
```

### 1.6 Remove Unnecessary Services

```bash
# Check what's running
ss -tlnp

# Remove CUPS if present
apt purge cups cups-browsed -y
snap remove cups 2>/dev/null
```

---

## Step 2: Install Tailscale (VPN)

Tailscale provides secure admin access.

```bash
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up
```

Once connected, restrict SSH to Tailscale only:

Edit `/etc/ssh/sshd_config`:
```bash
ListenAddress <your-tailscale-ip>  # e.g., 100.x.y.z
```

Update firewall:
```bash
ufw delete allow 22/tcp
ufw allow in on tailscale0 to any port 22
systemctl restart sshd
```

---

## Step 3: Install Node.js

OpenClaw runs on Node.js.

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
apt install -y nodejs
node --version  # Should be v20.x
```

---

## Step 4: Install OpenClaw

```bash
npm install -g openclaw
openclaw --version
```

### 4.1 Create Workspace

```bash
mkdir -p /opt/openclaw/workspace
cd /opt/openclaw/workspace

# Create config files
cat > AGENTS.md << 'EOF'
# Agent Configuration
Your behavioral rules go here.
EOF

cat > SOUL.md << 'EOF'
# Personality
Your AI's personality and voice.
EOF
```

### 4.2 Configure OpenClaw

Create `/opt/openclaw/config.json` with your settings. Reference the OpenClaw documentation for configuration options.

### 4.3 Set Up Secrets

Store your API keys in your password manager. Fetch them at runtime:

```bash
# Example with 1Password CLI
export ANTHROPIC_API_KEY=$(op read 'op://Vault/Anthropic/credential')
export TELEGRAM_BOT_TOKEN=$(op read 'op://Vault/Telegram/credential')
```

### 4.4 Start OpenClaw

```bash
openclaw gateway --port 18789
```

For production, create a systemd service or use a process manager.

---

## Step 5: Set Up Web Proxy (Caddy)

Caddy handles HTTPS automatically.

```bash
apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | tee /etc/apt/sources.list.d/caddy-stable.list
apt update
apt install caddy
```

Edit `/etc/caddy/Caddyfile`:
```
hugo.yourdomain.com {
    reverse_proxy localhost:18789
}
```

```bash
systemctl reload caddy
```

---

## Step 6: GitHub Integration (Fine-Grained PAT)

For workspace sync:

1. GitHub → Settings → Developer Settings → Personal Access Tokens → Fine-grained tokens
2. Select specific repositories only
3. Permissions: Contents (Read/Write), Metadata (Read-only)
4. Store in password manager
5. Set 90-day expiration

Configure git:
```bash
git config --global user.name "Hugo"
git config --global user.email "hugo@yourdomain.com"
```

---

## Step 7: Verification Checklist

- [ ] SSH: Key-based only, no passwords
- [ ] SSH: Restricted to Tailscale interface
- [ ] Firewall: UFW active, default deny
- [ ] OpenClaw: Running and responding
- [ ] Web proxy: HTTPS working
- [ ] GitHub: Fine-grained PAT, specific repos only

---

## Troubleshooting

**Can't SSH after restricting to Tailscale:**
Use provider's console to fix `/etc/ssh/sshd_config`.

**OpenClaw not starting:**
Check Node.js version, config file syntax, environment variables.

**HTTPS not working:**
Check Caddy logs: `journalctl -u caddy -f`

---

## Maintenance

**Update OpenClaw:**
```bash
npm update -g openclaw
```

**Rotate GitHub PAT:**
1. Create new token in GitHub
2. Update in password manager
3. Restart OpenClaw

---

## Related Documents

| Document | What It Covers |
|----------|----------------|
| [OPENCLAW-COOKBOOK-EXECUTIVE.md](OPENCLAW-COOKBOOK-EXECUTIVE.md) | Non-technical overview |
| [DEVOPS-ARCHITECTURE-PUBLIC.md](DEVOPS-ARCHITECTURE-PUBLIC.md) | Infrastructure design, backup strategy |
| [SECOPS-ARCHITECTURE-PUBLIC.md](SECOPS-ARCHITECTURE-PUBLIC.md) | Threat model, secrets management |

---

*Last updated: February 5, 2026*
