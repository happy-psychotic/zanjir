# Zanjir - Matrix Server

**Self-hosted, fast, lightweight Matrix server powered by Conduit (Rust)**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Docker](https://img.shields.io/badge/docker-ready-blue.svg)](https://www.docker.com/)
[![Matrix](https://img.shields.io/badge/matrix-conduit-green.svg)](https://conduit.rs/)

---

[![zanjir-screens-copy.webp](https://i.postimg.cc/yYjVzZzN/zanjir-screens-copy.webp)](https://postimg.cc/9r43dz03)

## Video Tutorial

[![Zanjir Project Introduction](https://i.postimg.cc/0Qq8qTFm/zanjir-copy.webp)](https://youtu.be/ZKTOs9y6rpw)

**📖 [Persian Version (نسخه فارسی)](README-FA.md)**

---

> [!IMPORTANT]
> **Upgrading from Dendrite?** If you previously installed Zanjir with Dendrite, you need to migrate to the new Conduit-based version. **This will require a fresh installation and all data will be lost.** See the [Migration Guide](MIGRATE.md) for step-by-step instructions.

---
## 📋 Table of Contents

- [Features](#features)
- [Quick Start](#quick-start)
- [Installation](#installation)
- [User Guide](#user-guide)
- [Admin Panel](#admin-panel)
- [Voice/Video Calls](#voicevideo-calls)
- [Custom Port](#custom-port)
- [Troubleshooting](#troubleshooting)
- [FAQ](#faq)
- [Contributing](#contributing)
- [License](#license)

---

## ✨ Features

| Feature | Description |
|---------|-------------|
| 🔐 **Open Registration** | Users can self-register via web interface |
| 👑 **Admin Panel** | Web-based admin dashboard with audit logging |
| 📞 **Voice/Video Calls** | TURN server for reliable NAT traversal |
| 🔧 **Deployment Profiles** | Isolated mode or internal-domain federation mode |
| 📱 **Element Web** | Modern, responsive Matrix client |
| 🇮🇷 **Persian UI** | Fully translated interface |
| 🐳 **Docker Powered** | One-command installation |
| 🔒 **Certificate Modes** | Public CA, private/internal CA, or isolated IP self-signed mode |

---

## 🚀 Quick Start

**Requirements:**
- Ubuntu 20.04+ or Debian 11+
- 2GB RAM minimum
- Internal domain name for federation mode, or IP address for isolated/test mode
- Ports: 80, 443 (or custom), 3478, 5349 (UDP)

**One-line installation:**

```bash
git clone https://github.com/MatinSenPai/zanjir.git ~/zanjir
cd ~/zanjir
sudo bash install.sh
```

**Access:**
- **Web App**: `https://your-domain.com`
- **Admin Panel**: `https://your-domain.com/admin`

---

## 📦 Installation

### Step 1: Clone Repository

```bash
git clone https://github.com/MatinSenPai/zanjir.git
cd zanjir
```

### Step 2: Run Installer

```bash
sudo bash install.sh
```

**Installation prompts:**

1. **Server address** - Internal domain or IP (for example `matrix-a.example.internal` or `192.168.1.50`)
2. **Deployment mode** - `isolated` or `federated`
3. **Certificate mode** - `public` or `private` for domain installs
4. **Admin email** - Only when public CA mode is selected
5. **TLS certificate/key paths** - Only when private CA mode is selected
6. **HTTPS port** - Default: 443 (press Enter), or custom (for example `8443`)

### Supported Modes

- `isolated + IP`: Caddy internal CA, federation disabled, intended for isolated or test deployments only
- `isolated + domain + public CA`: single-server deployment with publicly trusted TLS
- `isolated + domain + private CA`: single-server deployment with operator-provided PEM files
- `federated + domain + public CA`: preferred when public validation actually works for the internal deployment
- `federated + domain + private CA`: real fallback for internal networks that cannot use a public CA

Not supported for full federation:

- plain HTTP between homeservers
- IP-only mode as a normal federation path

### Step 3: Create Your First Account

After installation completes, open the web client and register normally:

1. Visit `https://your-domain.com`
2. Click `Create account`
3. Choose username and password
4. Log in through Element Web

Current note:

- the repository no longer uses the old Dendrite `create-account` command path
- the current stack is Conduit-based and open registration is enabled by default
- federated installs now enable Matrix federation through `FEDERATION_ENABLED=true`
- private/internal CA mode mounts operator-provided PEM files from `./certs`

---

## 🎛️ Management

After installation, use the `zanjir` command to manage your server:

```bash
zanjir
```

### Available Commands

```bash
zanjir              # Interactive menu
zanjir status       # Show service status
zanjir logs         # View logs (all services)
zanjir logs conduit # View specific service logs
zanjir restart      # Restart all services
zanjir start        # Start all services
zanjir stop         # Stop all services
zanjir update       # Update to latest version
zanjir backup       # Backup all data
zanjir uninstall    # Completely remove Zanjir
```

### Interactive Menu

When you run `zanjir` without arguments, you'll see a beautiful interactive menu:

```
╔═══════════════════════════╗
║                           ║
║      Z A N J I R ⛓️       ║
║                           ║
╚═══════════════════════════╝

Matrix Server Management Tool
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Main Menu:

  1. Show Status
  2. View Logs
  3. Restart Services
  4. Start Services
  5. Stop Services
  6. Update Zanjir
  7. Backup Data
  8. Uninstall
  9. Exit
```

---

## 👤 User Guide

### Registration

1. Visit `https://your-domain.com`
2. Click **"Create account"**
3. Fill in username and password
4. Start messaging!

### Using Element Web

- **Create Room**: Click `+` button → New Room
- **Invite Users**: Room settings → Invite users → Enter `@username:your-domain.com`
- **Send Messages**: Type and press Enter
- **Voice Call**: Click phone icon in room header
- **Video Call**: Click video icon in room header

### Mobile Apps

**Android:**
- [Element (Play Store)](https://play.google.com/store/apps/details?id=im.vector.app)
- [Element (F-Droid)](https://f-droid.org/packages/im.vector.app/)

**iOS:**
- [Element (App Store)](https://apps.apple.com/app/element-messenger/id1083446067)

**Configuration:**
- Homeserver URL: `https://your-domain.com`
- Identity server: Leave blank

---

## 🛡️ Admin Panel

### Access

Visit: `https://your-domain.com/admin`

Login with your account credentials.

Current limitation:

- the admin panel is still a thin convenience UI with placeholder-grade authorization logic in `admin/app.py`
- do not treat it as the authoritative security boundary for operator actions yet

### Features

#### Dashboard
- Total users count
- Active users
- Total rooms

#### User Management
- View all users
- Disable user accounts
- Delete users
- View user status

#### Audit Logs
- Track all admin actions
- Timestamps and IP addresses
- Target user tracking
- Detailed action logs

### Admin Actions

**Disable a user:**
1. Go to `Users` page
2. Find user
3. Click **"Disable"**

**Delete a user:**
1. Go to `Users` page
2. Find user
3. Click **"Delete"** → Confirm

All actions are automatically logged and visible in the **Logs** page.

---

## 📞 Voice/Video Calls

Zanjir includes a TURN server (coturn) for reliable voice/video calls.

### How It Works

TURN server helps users behind NAT/firewalls connect:
- Direct P2P when possible
- TURN relay when necessary
- Automatic fallback

### Firewall Configuration

**Required ports:**

| Port | Protocol | Purpose |
|------|----------|---------|
| 3478 | UDP | STUN/TURN |
| 5349 | UDP | TURN-TLS |

**UFW example:**

```bash
sudo ufw allow 3478/udp
sudo ufw allow 5349/udp
```

### Testing Calls

1. Create two accounts
2. Create a room, invite both users
3. Click phone/video icon
4. Accept call on other end

---

## 🔧 Custom Port

If port 443 is already in use, you can use a custom port.

### Installation with Custom Port

During `install.sh`, enter your desired port:

```
HTTPS port (default: 443): 8443
```

This will:
- Use port 8443 for HTTPS
- Use port 8080 for HTTP (auto-calculated)
- Update all configurations

### Accessing with Custom Port

```
https://your-domain.com:8443
```

### Changing Port After Installation

1. Edit `.env`:
   ```bash
   nano .env
   ```
2. Change `HTTPS_PORT` and `HTTP_PORT`
3. Restart services:
   ```bash
   docker compose down
   docker compose up -d
   ```

---

## 🔍 Troubleshooting

### Common Issues

#### Docker Installation fails (Iranian VPS)

**Error:** `connection refused` to `download.docker.com`

**Solution:** Script automatically uses Iranian mirrors:
- `docker.arvancloud.ir`
- `registry.docker.ir`

#### Registration not working

**Check:**
1. Verify registration is enabled in docker-compose.yml: `CONDUIT_ALLOW_REGISTRATION: "true"`
2. Check `element-config.json`: `"UIFeature.registration": true`
3. Restart containers:
   ```bash
   docker compose restart
   ```

#### Voice calls failing

**Solutions:**
1. Check TURN server is running:
   ```bash
   docker ps | grep coturn
   ```
2. Verify firewall allows UDP 3478, 5349
3. Check TURN secret in `.env` matches `TURN_SECRET` environment variable

#### Admin panel login fails

**Solutions:**
1. Verify the container is healthy and the homeserver is reachable:
   ```bash
   zanjir status
   ```
2. Check admin container logs:
   ```bash
   docker logs zanjir-admin
   ```
3. Treat the current admin panel as experimental; direct homeserver or compose-level checks are more trustworthy today.

#### Federation fails between servers

**Check:**
1. Both servers were installed in `federated` mode
2. Both servers use stable internal domains, not raw IPs
3. Both peers trust the same private/internal CA, or both have publicly trusted certificates
4. `/.well-known/matrix/server` returns the expected `m.server` value:
   ```bash
   curl -k https://your-domain/.well-known/matrix/server
   ```
5. `docker compose logs caddy conduit` shows successful HTTPS requests instead of certificate failures

#### Port already in use

**Solution:** Use custom port during installation, or change port in `.env`

---

## ❓ FAQ

### General

**Q: Can I use an IP address instead of domain?**  
A: Yes, but only for isolated or test deployments. IP mode uses Caddy's internal CA and is not a supported full-federation profile.

**Q: Is federation enabled?**  
A: Only when you choose `federated` mode during installation. Isolated mode keeps federation disabled.

**Q: Can users video call externally?**  
A: Calls work for users who can reach the deployment and TURN service. Cross-server behavior depends on correct federation and certificate setup on every participating server.

### Security

**Q: How are passwords stored?**  
A: Password verification is handled by the homeserver. This repository no longer uses the old PostgreSQL-based Dendrite runtime path.

**Q: Is end-to-end encryption supported?**  
A: Yes! Element/Matrix supports E2EE by default.

**Q: What about audit logs?**  
A: Admin actions logged in SQLite (`admin/audit_log.db`).

### Performance

**Q: How many users can it handle?**  
A: Conduit is extremely lightweight. A 1GB VPS can handle ~100-500 users with ease. Conduit uses ~50MB RAM vs Dendrite's 200-500MB.

**Q: What about backups?**  
A: Back up the active Docker volumes, especially `zanjir-conduit-data`, `zanjir-caddy-data`, and `zanjir-admin-data`. The `zanjir backup` command is the current project helper for this.

---

## 🤝 Contributing

Contributions welcome! Please:
1. Fork the repository
2. Create feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open Pull Request

---

## 📄 License

MIT License - see [LICENSE](LICENSE) file

---

## 🙏 Acknowledgments

- [Matrix.org](https://matrix.org/) - Open protocol
- [Conduit](https://conduit.rs/) - Fast, lightweight Rust homeserver
- [Element](https://element.io/) - Web client
- [Coturn](https://github.com/coturn/coturn) - TURN server
- [Caddy](https://caddyserver.com/) - Reverse proxy

---

## 📞 Support

- **GitHub Issues**: [Report bugs](https://github.com/MatinSenPai/zanjir/issues)
- **Discussions**: [Ask questions](https://github.com/MatinSenPai/zanjir/discussions)

---

**Made with ❤️ for secure, private communication**
