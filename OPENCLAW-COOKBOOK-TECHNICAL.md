# Building Your Own AI Assistant: Technical Cookbook

*Step-by-step guide to deploying a secure, self-hosted OpenClaw infrastructure.*

---

## Prerequisites

Before you begin, you'll need:

| Requirement | Purpose |
|-------------|---------|
| VPS account | Hostinger, Hetzner, DigitalOcean, or similar |
| Domain name | For clean URLs (assistant.yourdomain.com) |
| Cloudflare account | Free tier is sufficient |
| 1Password account | Or another secrets manager with CLI support |
| Anthropic API key | For Claude access |
| Messaging platform bot | Telegram Bot Token, WhatsApp Business, etc. |

**Estimated time:** 2-4 hours for initial setup.

---

## Step 1: VPS Setup and Hardening

### 1.1 Provision the Server

Choose a provider and create a VPS:
- **OS:** Ubuntu 22.04 LTS or Debian 12
- **Memory:** 4GB minimum (8GB recommended)
- **Storage:** 40GB SSD minimum
- **Location:** Closest to your users

### 1.2 Initial Access

```bash
# SSH to your new server
ssh root@<server-ip>

# Update system
apt update && apt upgrade -y

# Set hostname
hostnamectl set-hostname prod-de  # or whatever you prefer
```

### 1.3 Create Non-Root User (Optional but Recommended)

```bash
adduser deploy
usermod -aG sudo deploy

# Copy SSH keys
mkdir -p /home/deploy/.ssh
cp ~/.ssh/authorized_keys /home/deploy/.ssh/
chown -R deploy:deploy /home/deploy/.ssh
```

### 1.4 Harden SSH

Edit `/etc/ssh/sshd_config`:

```bash
# Disable password authentication
PasswordAuthentication no

# Disable root login (if using non-root user)
PermitRootLogin no

# Only allow specific users
AllowUsers deploy
```

Restart SSH:
```bash
systemctl restart sshd
```

### 1.5 Configure Firewall (UFW)

```bash
# Install UFW
apt install ufw -y

# Default policies
ufw default deny incoming
ufw default allow outgoing

# Allow SSH (will be restricted to Tailscale later)
ufw allow 22/tcp

# Enable firewall
ufw enable
```

### 1.6 Remove Unnecessary Services

```bash
# Check what's listening
ss -tlnp

# Remove CUPS if present (print server)
apt purge cups cups-browsed -y

# Remove any other unnecessary services
```

---

## Step 2: Install Tailscale (VPN)

Tailscale provides secure admin access without exposing SSH to the internet.

### 2.1 Install

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

### 2.2 Authenticate

```bash
tailscale up
```

Follow the URL to authenticate with your Tailscale account.

### 2.3 Restrict SSH to Tailscale

Once Tailscale is working, lock SSH to the VPN interface:

Edit `/etc/ssh/sshd_config`:
```bash
ListenAddress <tailscale-ip>  # e.g., 100.x.y.z
```

Update firewall:
```bash
ufw delete allow 22/tcp
ufw allow in on tailscale0 to any port 22
systemctl restart sshd
```

**Test:** Verify you can still SSH via Tailscale IP before disconnecting.

---

## Step 3: Install Docker

### 3.1 Install Docker Engine

```bash
# Install prerequisites
apt install ca-certificates curl gnupg -y

# Add Docker's GPG key
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg

# Add repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install
apt update
apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
```

### 3.2 Verify Installation

```bash
docker run hello-world
```

---

## Step 4: Configure Docker Securely

### 4.1 The Critical Rule: Bind to Localhost

**Docker bypasses UFW.** When you expose a port, it modifies iptables directly, ignoring your firewall rules.

**WRONG — Exposes port to internet:**
```yaml
ports:
  - "8443:8443"
```

**CORRECT — Only accessible locally:**
```yaml
ports:
  - "127.0.0.1:8443:8443"
```

### 4.2 Example docker-compose.yml

```yaml
version: '3.8'
services:
  openclaw:
    image: openclaw/gateway:latest
    restart: unless-stopped
    ports:
      - "127.0.0.1:8443:8443"  # CRITICAL: localhost only
    volumes:
      - ./workspace:/workspace
      - ./config:/config
    env_file:
      - .env
```

### 4.3 Verify Ports Are Not Exposed

From an external machine:
```bash
# Should timeout or refuse connection
curl --connect-timeout 3 http://<server-public-ip>:8443/
```

From the server itself:
```bash
# Should work
curl http://127.0.0.1:8443/
```

---

## Step 5: Set Up Cloudflare Tunnel

### 5.1 Install cloudflared

```bash
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | gpg --dearmor -o /usr/share/keyrings/cloudflare-main.gpg
echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared any main' | tee /etc/apt/sources.list.d/cloudflared.list
apt update
apt install cloudflared -y
```

### 5.2 Authenticate

```bash
cloudflared tunnel login
```

This opens a browser. Select your domain in Cloudflare.

### 5.3 Create Tunnel

```bash
cloudflared tunnel create prod-tunnel
```

Note the tunnel ID and credentials file path.

### 5.4 Configure Tunnel

Create `/etc/cloudflared/config.yml`:

```yaml
tunnel: <tunnel-id>
credentials-file: /root/.cloudflared/<tunnel-id>.json

ingress:
  # Hugo web UI
  - hostname: hugo.yourdomain.com
    service: http://localhost:8443

  # Intake webhook (public for Telegram)
  - hostname: intake.yourdomain.com
    service: http://localhost:8080

  # Catch-all
  - service: http_status:404
```

### 5.5 Create DNS Routes

```bash
cloudflared tunnel route dns prod-tunnel hugo.yourdomain.com
cloudflared tunnel route dns prod-tunnel intake.yourdomain.com
```

### 5.6 Install as Service

```bash
cloudflared service install
systemctl enable cloudflared
systemctl start cloudflared
```

### 5.7 Set Up Cloudflare Access (Optional but Recommended)

For services that need authentication:

1. Go to Cloudflare Dashboard → Zero Trust → Access → Applications
2. Add Application → Self-hosted
3. Set Application domain: `hugo.yourdomain.com`
4. Add policy: Allow emails matching `you@yourdomain.com`
5. Users will authenticate via email OTP before reaching the app

---

## Step 6: Secrets Management with 1Password

### 6.1 Install 1Password CLI

```bash
curl -sS https://downloads.1password.com/linux/keys/1password.asc | gpg --dearmor -o /usr/share/keyrings/1password-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/1password-archive-keyring.gpg] https://downloads.1password.com/linux/debian/amd64 stable main" | tee /etc/apt/sources.list.d/1password.list
apt update
apt install 1password-cli -y
```

### 6.2 Create Service Account

1. Go to 1Password → Developer → Service Accounts
2. Create new service account for this server
3. Grant access to specific vault only (e.g., "Hugo" vault)
4. Copy the token

### 6.3 Store Service Account Token

Create `/etc/openclaw/op-token`:
```bash
echo "YOUR_SERVICE_ACCOUNT_TOKEN" > /etc/openclaw/op-token
chmod 600 /etc/openclaw/op-token
```

### 6.4 Create Secrets Loader Script

Create `/opt/openclaw/load-secrets.sh`:

```bash
#!/bin/bash
export OP_SERVICE_ACCOUNT_TOKEN=$(cat /etc/openclaw/op-token)

# Load secrets from 1Password
export ANTHROPIC_API_KEY=$(op read 'op://Hugo/Anthropic API/credential')
export TELEGRAM_BOT_TOKEN=$(op read 'op://Hugo/Telegram Bot/credential')
# Add more as needed

# Start the application
exec docker compose up
```

```bash
chmod +x /opt/openclaw/load-secrets.sh
```

---

## Step 7: Deploy OpenClaw

### 7.1 Create Directory Structure

```bash
mkdir -p /opt/openclaw
cd /opt/openclaw
```

### 7.2 Create docker-compose.yml

```yaml
version: '3.8'
services:
  openclaw:
    image: openclaw/gateway:latest
    restart: unless-stopped
    ports:
      - "127.0.0.1:8443:8443"
    volumes:
      - ./workspace:/workspace
      - ./config:/config
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - TELEGRAM_BOT_TOKEN=${TELEGRAM_BOT_TOKEN}
```

### 7.3 Create Workspace

```bash
mkdir -p workspace
cd workspace

# Create initial files
cat > AGENTS.md << 'EOF'
# Agent Configuration
Your behavioral rules go here.
EOF

cat > SOUL.md << 'EOF'
# Personality
Your AI's personality definition.
EOF
```

### 7.4 Create systemd Service

Create `/etc/systemd/system/openclaw.service`:

```ini
[Unit]
Description=OpenClaw Gateway
After=docker.service
Requires=docker.service

[Service]
Type=simple
WorkingDirectory=/opt/openclaw
ExecStart=/opt/openclaw/load-secrets.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl enable openclaw
systemctl start openclaw
```

---

## Step 8: GitHub Integration (Fine-Grained PAT)

### 8.1 Create Fine-Grained PAT

1. GitHub → Settings → Developer Settings → Personal Access Tokens → Fine-grained tokens
2. Generate new token
3. **Repository access:** Only select repositories
4. **Permissions:** Contents (Read/Write), Metadata (Read-only)
5. **Expiration:** 90 days
6. Copy and store in 1Password

### 8.2 Configure Git on Server

```bash
git config --global user.name "Hugo"
git config --global user.email "hugo@yourdomain.com"
```

### 8.3 Set Up Authentication

Option A — gh CLI:
```bash
# Install gh
apt install gh -y

# Login with token
export GITHUB_TOKEN=$(op read 'op://Hugo/GitHub PAT/credential')
echo "$GITHUB_TOKEN" | gh auth login --with-token
```

Option B — Credential store:
```bash
git config --global credential.helper store
export GITHUB_TOKEN=$(op read 'op://Hugo/GitHub PAT/credential')
echo "https://hugo:${GITHUB_TOKEN}@github.com" > ~/.git-credentials
chmod 600 ~/.git-credentials
```

---

## Step 9: Weekly Security Scan

### 9.1 Create Scan Script

Create `/opt/openclaw/security-scan.sh`:

```bash
#!/bin/bash
echo "=== Weekly Security Scan ==="
echo "Date: $(date)"
echo ""

echo "=== Port Exposure Check ==="
echo "Testing if Docker ports are reachable from internet..."
# Run these from an external machine or use a service like check-host.net

echo ""
echo "=== SSH Configuration ==="
ss -tlnp | grep ':22'

echo ""
echo "=== Firewall Status ==="
ufw status

echo ""
echo "=== Service Status ==="
systemctl is-active openclaw cloudflared

echo ""
echo "=== Docker Port Bindings ==="
docker ps --format "{{.Names}}: {{.Ports}}"

echo ""
echo "=== GitHub Auth Status ==="
gh auth status 2>&1

echo ""
echo "=== Scan Complete ==="
```

### 9.2 Run Weekly via Cron

```bash
crontab -e
```

Add:
```
0 9 * * 1 /opt/openclaw/security-scan.sh >> /var/log/security-scan.log 2>&1
```

---

## Step 10: Verification Checklist

After setup, verify:

- [ ] **SSH:** Only accessible via Tailscale
- [ ] **Docker ports:** Bound to 127.0.0.1, not 0.0.0.0
- [ ] **Cloudflare Tunnel:** Working, services accessible via domain
- [ ] **UFW:** Active, default deny incoming
- [ ] **1Password:** Service account working, secrets loading
- [ ] **OpenClaw:** Running, responding to messages
- [ ] **GitHub:** Fine-grained PAT, correct repos only
- [ ] **No CUPS:** Or other unnecessary services

---

## Troubleshooting

### Can't SSH after restricting to Tailscale

Boot into recovery mode via provider console, fix `/etc/ssh/sshd_config`.

### Docker container can't reach internet

Check UFW outgoing rules, Docker network configuration.

### Cloudflare Tunnel not connecting

```bash
journalctl -u cloudflared -f
```

Check credentials file path in config.yml.

### OpenClaw not starting

```bash
journalctl -u openclaw -f
docker compose logs -f
```

### 1Password CLI errors

Verify service account token, check vault access permissions.

---

## Maintenance

### Updating OpenClaw

```bash
cd /opt/openclaw
docker compose pull
systemctl restart openclaw
```

### Rotating GitHub PAT

1. Create new token in GitHub
2. Update in 1Password
3. Restart OpenClaw (will fetch new token)

### Adding a New Server

1. Provision VPS
2. Follow Steps 1-9
3. Add to Tailscale network
4. Create tunnel for new services
5. Add to monitoring

---

*This cookbook accompanies the DevOps and SecOps architecture documents. For executive overview, see COOKBOOK-EXECUTIVE.md.*

*Last updated: February 5, 2026*
