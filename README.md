# 🎬 Media Server Automation

Complete Infrastructure as Code solution for deploying a production-ready media server with VPN-protected download clients in 30 minutes.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-20.04%2B-orange)](https://ubuntu.com/)
[![Debian](https://img.shields.io/badge/Debian-11%2B-red)](https://www.debian.org/)
[![Docker](https://img.shields.io/badge/Docker-24.0%2B-blue)](https://www.docker.com/)

## ✨ Features

### 🏗️ **One-Command Deployment**
- Deploy complete media server from bare OS in ~30 minutes
- Idempotent automation - safe to re-run
- VM testing environment included

### 🔒 **VPN-Protected Downloads**  
- Transmission & NZBGet routed through VPN
- Kill switch prevents IP leaks
- Support for ProtonVPN, Surfshark, NordVPN & 60+ providers

### 🎯 **Complete Media Stack**
- **Sonarr** - TV show management
- **Radarr** - Movie management  
- **Prowlarr** - Indexer management
- **Bazarr** - Subtitle management
- **Overseerr** - Request management
- **Transmission** - BitTorrent client (VPN)
- **NZBGet** - Usenet client (VPN)

### 🛡️ **Enterprise Security**
- UFW firewall auto-configuration
- Fail2ban brute force protection
- Tailscale secure remote access
- Non-root container execution
- Automatic security updates

### 📊 **Built-in Monitoring**
- Health checks for all services
- VPN connectivity monitoring
- Automated backups to NAS
- Performance metrics collection

## 🚀 Quick Start

### One-Command Deploy

```bash
# Clone repository
git clone https://github.com/your-username/media-server-automation.git
cd media-server-automation

# Deploy everything
sudo ./scripts/bootstrap.sh
```

**That's it!** Visit `http://your-server-ip:5055` (Overseerr) to start managing your media.

### Test in VM First

```bash
cd vm-testing
vagrant up
vagrant ssh
cd media-server-automation
sudo ./scripts/bootstrap.sh --environment development
```

## 📋 Requirements

### System Requirements
- **OS:** Ubuntu 20.04+ or Debian 11+
- **RAM:** 4GB minimum, 8GB recommended
- **Storage:** 20GB free disk space
- **Network:** Internet connection for deployment

### Optional Components
- **NAS:** Network storage for media files (NFS/SMB)
- **VPN:** ProtonVPN account (or other supported provider)
- **Domain:** For SSL certificates (optional)

## 🏛️ Architecture

### Network Flow
```
Internet
    ↓
[VPN Provider] ←── Gluetun Container
    ↓
[Transmission] [NZBGet]  ←── VPN Protected
    ↓
[Downloads] → [NAS Storage]
    ↓
[Sonarr] [Radarr] [Prowlarr] [Bazarr]  ←── Local Network
    ↓
[Overseerr] ←── User Requests
```

### Security Layers
```
[UFW Firewall] → [Docker Network] → [Service Isolation]
        ↓                ↓                    ↓
[Fail2ban] → [VPN Kill Switch] → [Non-root Users]
```

## 🔧 Configuration

### Environment Variables

Create `.env` file from template:

```bash
cp docker/.env.template docker/.env
```

Key settings:

```env
# VPN Configuration
PROTON_USER=your_vpn_username
PROTON_PASS=your_vpn_password

# Download Client Credentials  
TRANSMISSION_USER=admin
TRANSMISSION_PASS=secure_password

# Network Settings
LOCAL_NETWORK=192.168.1.0/24
NAS_HOST=your-nas.local
```

### Advanced Configuration

Custom deployment with config file:

```yaml
# production.yml
nas:
  host: nas.example.com
  path: /volume1/media
  
vpn:
  provider: protonvpn
  country: Switzerland
  
services:
  transmission:
    download_limit: 0  # Unlimited
    upload_limit: 100   # 100 KB/s
    
monitoring:
  grafana: true
  prometheus: true
```

Deploy with custom config:

```bash
sudo ./scripts/bootstrap.sh --config production.yml
```

## 📚 Documentation

| Document | Description |
|----------|-------------|
| [🚀 DEPLOYMENT.md](docs/DEPLOYMENT.md) | Complete deployment guide |
| [🔧 TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) | Common issues and solutions |
| [📁 Directory Structure](#-project-structure) | Project organization |

## 📁 Project Structure

```
media-server-automation/
├── ansible/                    # Infrastructure automation
│   ├── playbooks/             # Deployment playbooks
│   ├── inventory/             # Server inventory
│   └── group_vars/            # Configuration variables
├── docker/                    # Container orchestration
│   ├── docker-compose.yml     # Service definitions
│   └── configs/               # Service configurations
├── scripts/                   # Automation scripts
│   ├── bootstrap.sh           # Main deployment script
│   ├── backup-current.sh      # Backup existing system
│   └── restore-data.sh        # Restore from backup
├── vm-testing/                # Local testing environment
│   └── Vagrantfile           # VM configuration
└── docs/                     # Documentation
    ├── DEPLOYMENT.md         # Deployment guide
    └── TROUBLESHOOTING.md    # Issue resolution
```

## 🎯 Service Access

After deployment, access services at:

| Service | URL | Default Login |
|---------|-----|--------------|
| 🎬 **Overseerr** | http://server:5055 | Setup wizard |
| 📺 **Sonarr** | http://server:8989 | No auth required |
| 🎭 **Radarr** | http://server:7878 | No auth required |
| 🔍 **Prowlarr** | http://server:9696 | No auth required |
| 💬 **Bazarr** | http://server:6767 | No auth required |
| ⬇️ **Transmission** | http://server:9091 | admin/password |
| 📡 **NZBGet** | http://server:6789 | admin/password |

## 🔄 Migration from Existing Setup

### 1. Backup Current System

```bash
./scripts/backup-current.sh --nas-backup --compress
```

### 2. Deploy New System

```bash
sudo ./scripts/bootstrap.sh
```

### 3. Restore Configuration

```bash
./scripts/restore-data.sh /path/to/backup
```

All your settings, databases, and API keys are automatically restored!

## 🛠️ Management Commands

### Service Control

```bash
cd /opt/media-server

# View status
docker-compose ps

# View logs  
docker-compose logs [service]

# Restart service
docker-compose restart [service]

# Update all services
docker-compose pull && docker-compose up -d
```

### System Control

```bash
# Control entire media server
sudo systemctl start media-server
sudo systemctl stop media-server
sudo systemctl restart media-server

# Check system health
/opt/media-server/status.sh

# Run backups manually
/opt/media-server/backup-configs.sh
```

### Monitoring Commands

```bash
# Check VPN status
docker exec gluetun curl -s ifconfig.me

# Monitor downloads
docker logs transmission -f
docker logs nzbget -f

# Check storage usage
df -h /mnt/artie
```

## 🔒 Security Features

### Automated Security

- **Firewall:** UFW configured with minimal required ports
- **Fail2ban:** Protection against brute force attacks
- **Updates:** Automatic security updates enabled
- **VPN Kill Switch:** Download clients can't leak IP
- **User Isolation:** Services run as non-root user

### Network Security

- **VPN Routing:** Download traffic always encrypted
- **Local Access:** Management interfaces on local network only
- **Tailscale:** Secure remote access when traveling
- **DNS Protection:** Secure DNS servers configured

## 📊 Monitoring & Maintenance

### Automated Monitoring

- **Health Checks:** Every 15 minutes
- **VPN Monitoring:** Continuous IP leak detection  
- **Storage Monitoring:** Disk usage alerts
- **Service Monitoring:** Container restart on failure

### Automated Maintenance

- **Backups:** Daily configuration backups
- **Updates:** Automatic container updates (4 AM daily)
- **Cleanup:** Log rotation and disk cleanup
- **Security:** Daily security update checks

## 🆘 Common Issues

### Quick Fixes

```bash
# Services won't start
docker-compose down && docker-compose up -d

# VPN not working
docker-compose restart gluetun transmission nzbget

# Permission errors
sudo chown -R 1001:1001 /opt/media-server/configs

# Disk full
/opt/media-server/cleanup-disk.sh
```

### Get Help

1. 📖 Check [Troubleshooting Guide](docs/TROUBLESHOOTING.md)
2. 🔍 Search [existing issues](https://github.com/your-username/media-server-automation/issues)
3. 📝 Create [new issue](https://github.com/your-username/media-server-automation/issues/new) with logs
4. 💬 Ask in [Discussions](https://github.com/your-username/media-server-automation/discussions)

## 🤝 Contributing

Contributions welcome! Areas where help is needed:

- 🧪 **Testing:** More VPN providers and OS distributions
- 📚 **Documentation:** Setup guides for specific scenarios  
- 🔧 **Features:** Additional monitoring, backup options
- 🐛 **Bug Fixes:** Issue resolution and improvements

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

Built with these amazing open-source projects:

- **[LinuxServer.io](https://www.linuxserver.io/)** - Docker images for all *arr services
- **[Gluetun](https://github.com/qdm12/gluetun)** - VPN client container
- **[Ansible](https://www.ansible.com/)** - Infrastructure automation
- **[Docker](https://www.docker.com/)** - Container platform
- **[Overseerr](https://overseerr.dev/)** - Request management

Special thanks to the selfhosted community for inspiration and feedback!

## ⭐ Star History

If this project helped you, please consider giving it a star! ⭐

[![Star History Chart](https://api.star-history.com/svg?repos=your-username/media-server-automation&type=Date)](https://star-history.com/#your-username/media-server-automation&Date)

---

**🎉 Ready to automate your media server? Get started with one command:**

```bash
curl -fsSL https://raw.githubusercontent.com/your-username/media-server-automation/main/install.sh | sudo bash
```

*Happy streaming! 🍿*