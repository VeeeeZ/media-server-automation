https://github.com/VeeeeZ/media-server-automation/releases

[![Releases](https://img.shields.io/github/v/release/VeeeeZ/media-server-automation?label=Releases&style=for-the-badge)](https://github.com/VeeeeZ/media-server-automation/releases)

# Automated Media Server Stack — VPN-Protected, One-Command Setup

![Media Server Banner](https://images.unsplash.com/photo-1518709268805-4e9042af9f23?auto=format&fit=crop&w=1350&q=80)

A complete infrastructure-as-code solution to deploy a secure, self-hosted media stack. This project installs Sonarr, Radarr, Prowlarr, Bazarr, Transmission, NZBGet and the VPN gateway. The setup runs under Docker and uses Ansible for automation. Run a single script from Releases to provision a production-grade lab with strong isolation and download protection.

- Topics: ansible, automation, docker, homelab, infrastructure-as-code, media-server, radarr, self-hosted, servarr, sonarr, transmission, vpn

Table of Contents
- About
- Key features
- Architecture
- Components
- Requirements
- Quick start (one command)
- Installation details
- Configuration
- Network, ports and firewall
- Backup and updates
- Troubleshooting
- FAQ
- Contributing
- License

About
This repo provides code and playbooks to deploy a full media server environment. It integrates download managers with ServArr tools and routes all downloads through a VPN container. It aims for reproducible deployments on homelab hardware and cloud VMs. Use the Releases page to get the installer script.

Key features
- One-command installer delivered via Releases
- Ansible playbooks to manage lifecycle
- Docker Compose stacks per service
- VPN gateway for all download traffic
- Network-level isolation using bridge networks
- Centralized config via group_vars and host_vars
- Service health checks and unit tests
- TLS support for web UIs (optional)

Architecture
![Architecture Diagram](https://raw.githubusercontent.com/mayeaux/diagrams/master/media-server-architecture.png)

- Host runs Docker Engine and systemd.
- Ansible controls the host using SSH.
- A VPN container (WireGuard/OpenVPN) acts as a router for download containers.
- Download tools run inside containers that use the VPN container network.
- ServArr apps run in their own containers and talk to download tools via HTTP/RPC on internal networks.
- Volumes map media, config and downloads to persistent storage.

Components
- Sonarr — TV show automation and management.
- Radarr — Movie automation and management.
- Prowlarr — Indexer manager for NZB/Torrent indexers.
- Bazarr — Subtitle fetcher integrated with ServArr tools.
- Transmission — Lightweight bittorrent client (containerized).
- NZBGet — Usenet download client.
- VPN Gateway — WireGuard or OpenVPN container to route download traffic.
- Watchtower (optional) — Automatic container updates.

Requirements
- A Linux host (Debian/Ubuntu/CentOS) or VM.
- Docker Engine 20.10+ and Docker Compose plugin.
- Ansible 2.9+ on the controller machine (can be the host).
- A user with sudo privileges and SSH access to the target host.
- Static local storage for media and downloads (recommended: separate HDD/SSD).
- A public IP or port-forwarding for service access (reverse proxy recommended).

Supported platforms
- x86_64 Linux
- ARM64 (Raspberry Pi / RockPro64) with adjusted images for ARM

Quick start (one command)
1) Visit Releases and download the installer. The installer must be downloaded and executed from the Releases page:
https://github.com/VeeeeZ/media-server-automation/releases

2) Example download command. Replace <TAG> with the release tag you want to use:
curl -Lo install.sh https://github.com/VeeeeZ/media-server-automation/releases/download/<TAG>/install.sh && chmod +x install.sh && sudo ./install.sh

3) The installer runs an Ansible playbook that sets up Docker, pulls images and configures services. After it finishes, open your browser to the configured service ports.

Installation details
Directory layout (repo)
- ansible/
  - playbooks/
    - site.yml
    - vpn.yml
    - servarr.yml
  - roles/
    - docker
    - vpn
    - sonarr
    - radarr
    - prowlarr
    - bazarr
    - transmission
    - nzbget
  - group_vars/
  - host_vars/
- docker/
  - compose/
    - sonarr.yml
    - radarr.yml
    - prowlarr.yml
    - bazarr.yml
    - transmission.yml
    - nzbget.yml
- scripts/
  - helpers.sh
  - healthcheck.sh
- docs/
  - network.md
  - tls.md
  - storage.md

Ansible playbook flow
- docker role installs Docker Engine and Compose plugin.
- vpn role deploys WireGuard/OpenVPN container and sets up routing.
- Each app role deploys a Docker Compose file.
- Post-deploy tasks create users, set permissions, and start healthchecks.

Service configuration
- Config files live in /srv/media/{service}/config by default.
- Media files go to /srv/media/media/{movies|tv|music}.
- Downloads go to /srv/media/downloads/{torrents|usenet}.
- Change paths in group_vars/all.yml before deployment.

Network and VPN
- The VPN container uses a dedicated Docker network (vpn-net).
- Download containers connect to vpn-net and set the VPN container as their gateway.
- ServArr apps use an internal network (servarr-net) and do not route web UI traffic through the VPN.
- Example: Transmission runs with network_mode: service:vpn to force traffic through the VPN.

Reverse proxy and TLS
- Use a reverse proxy (Traefik, Nginx, Caddy) to expose web UIs.
- TLS termination occurs at the proxy.
- The repo contains sample Traefik labels for each service.
- You can enable Let’s Encrypt via the proxy. See docs/tls.md.

Configuration variables
Key vars in group_vars/all.yml:
- docker_compose_path: /srv/media/compose
- vpn_provider: wireguard | openvpn
- vpn_config_path: /srv/media/config/vpn
- media_path: /srv/media/media
- downloads_path: /srv/media/downloads
- user: mediaadmin
- timezone: UTC

Customizing services
- To add indexers to Sonarr/Radarr, configure Prowlarr first and link the apps to Prowlarr via their UI.
- To switch download client, adjust host_vars for that host and run ansible-playbook with --tags download-clients.
- The repo stores sample docker-compose files. Copy them to /srv/media/compose and change volume mounts as needed.

Backup and updates
- Back up config folders in /srv/media/config regularly.
- Use the provided backup role or your own rsync/cron job.
- For updates, run the update script from Releases or use Ansible to pull new image tags and restart services.

Troubleshooting
- Container logs: docker-compose -f <service>.yml logs -f
- Check VPN connectivity inside container: docker exec -it <vpn-container> ping -c 3 8.8.8.8
- Verify routing for download containers: docker exec -it <download> ip route
- If an app fails to start, check file permissions on config and media volumes.

FAQ
Q: Which VPN does the installer use?
A: The repo supports WireGuard and OpenVPN. Choose vpn_provider in group_vars/all.yml.

Q: Can I run this on a single-board computer?
A: Yes. Use ARM-compatible images and ensure you have enough RAM and storage.

Q: How do I update a service image?
A: Update the image tag in the compose file, then run docker-compose pull && docker-compose up -d. Or use the included Ansible update role.

Q: Does download traffic leak outside the VPN?
A: Download containers route through the VPN container by design. Check network_mode and ip rules if you suspect leaks.

Security best practices
- Keep the host OS and Docker updated.
- Run services as non-root where possible.
- Limit exposed ports to the reverse proxy.
- Use strong credentials for app UIs and change defaults.
- Isolate storage volumes and use filesystem permissions.

CI and tests
- The repo contains simple linting for Ansible roles.
- Run ansible-lint and yamllint before PRs.
- Containers include healthcheck scripts that the playbook registers.

Contributing
- Fork the repo and open a branch for your change.
- Run tests: ansible-lint, yamllint.
- Send a pull request with a clear description.
- Add tests for new roles and update docs.

Releases and installer
- The installer script and release assets live on the Releases page. Download the installer from the release you choose and run it:
https://github.com/VeeeeZ/media-server-automation/releases

- The release asset named install.sh downloads the playbook and runs the standard playbook. Use a release tag when downloading to keep installs reproducible.

Example advanced commands
- Dry run with Ansible:
ansible-playbook ansible/playbooks/site.yml -i inventory/hosts --check

- Run a single role:
ansible-playbook ansible/playbooks/site.yml -i inventory/hosts --tags vpn

- Pull latest images for all services:
ansible-playbook ansible/playbooks/site.yml -i inventory/hosts --tags update-images

Useful links
- Sonarr: https://sonarr.tv
- Radarr: https://radarr.video
- Prowlarr: https://prowlarr.com
- Bazarr: https://www.bazarr.media
- Transmission: https://transmissionbt.com
- NZBGet: https://nzbget.net

License
This repo uses the MIT License. See LICENSE for the full text.