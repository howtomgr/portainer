# portainer Installation Guide

portainer is a free and open-source container management platform. Portainer simplifies container management across Docker, Swarm, and Kubernetes environments, providing a GUI alternative to command-line tools

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 1 core minimum
  - RAM: 256MB minimum
  - Storage: 100MB for installation
  - Network: Docker API access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9000 (default portainer port)
  - Port 8000 for edge agent
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install portainer
sudo dnf install -y portainer

# Enable and start service
sudo systemctl enable --now portainer

# Configure firewall
sudo firewall-cmd --permanent --add-port=9000/tcp
sudo firewall-cmd --reload

# Verify installation
portainer --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install portainer
sudo apt install -y portainer

# Enable and start service
sudo systemctl enable --now portainer

# Configure firewall
sudo ufw allow 9000

# Verify installation
portainer --version
```

### Arch Linux

```bash
# Install portainer
sudo pacman -S portainer

# Enable and start service
sudo systemctl enable --now portainer

# Verify installation
portainer --version
```

### Alpine Linux

```bash
# Install portainer
apk add --no-cache portainer

# Enable and start service
rc-update add portainer default
rc-service portainer start

# Verify installation
portainer --version
```

### openSUSE/SLES

```bash
# Install portainer
sudo zypper install -y portainer

# Enable and start service
sudo systemctl enable --now portainer

# Configure firewall
sudo firewall-cmd --permanent --add-port=9000/tcp
sudo firewall-cmd --reload

# Verify installation
portainer --version
```

### macOS

```bash
# Using Homebrew
brew install portainer

# Start service
brew services start portainer

# Verify installation
portainer --version
```

### FreeBSD

```bash
# Using pkg
pkg install portainer

# Enable in rc.conf
echo 'portainer_enable="YES"' >> /etc/rc.conf

# Start service
service portainer start

# Verify installation
portainer --version
```

### Windows

```bash
# Using Chocolatey
choco install portainer

# Or using Scoop
scoop install portainer

# Verify installation
portainer --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/portainer

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
portainer --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable portainer

# Start service
sudo systemctl start portainer

# Stop service
sudo systemctl stop portainer

# Restart service
sudo systemctl restart portainer

# Check status
sudo systemctl status portainer

# View logs
sudo journalctl -u portainer -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add portainer default

# Start service
rc-service portainer start

# Stop service
rc-service portainer stop

# Restart service
rc-service portainer restart

# Check status
rc-service portainer status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'portainer_enable="YES"' >> /etc/rc.conf

# Start service
service portainer start

# Stop service
service portainer stop

# Restart service
service portainer restart

# Check status
service portainer status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start portainer
brew services stop portainer
brew services restart portainer

# Check status
brew services list | grep portainer
```

### Windows Service Manager

```powershell
# Start service
net start portainer

# Stop service
net stop portainer

# Using PowerShell
Start-Service portainer
Stop-Service portainer
Restart-Service portainer

# Check status
Get-Service portainer
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream portainer_backend {
    server 127.0.0.1:9000;
}

server {
    listen 80;
    server_name portainer.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name portainer.example.com;

    ssl_certificate /etc/ssl/certs/portainer.example.com.crt;
    ssl_certificate_key /etc/ssl/private/portainer.example.com.key;

    location / {
        proxy_pass http://portainer_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName portainer.example.com
    Redirect permanent / https://portainer.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName portainer.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/portainer.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/portainer.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9000/
    ProxyPassReverse / http://127.0.0.1:9000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend portainer_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/portainer.pem
    redirect scheme https if !{ ssl_fc }
    default_backend portainer_backend

backend portainer_backend
    balance roundrobin
    server portainer1 127.0.0.1:9000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R portainer:portainer /etc/portainer
sudo chmod 750 /etc/portainer

# Configure firewall
sudo firewall-cmd --permanent --add-port=9000/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status portainer

# View logs
sudo journalctl -u portainer -f

# Monitor resource usage
top -p $(pgrep portainer)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/portainer"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/portainer-backup-$DATE.tar.gz" /etc/portainer /var/lib/portainer

echo "Backup completed: $BACKUP_DIR/portainer-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop portainer

# Restore from backup
tar -xzf /backup/portainer/portainer-backup-*.tar.gz -C /

# Start service
sudo systemctl start portainer
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u portainer -n 100
sudo tail -f /var/log/portainer/portainer.log

# Check configuration
portainer --version

# Check permissions
ls -la /etc/portainer
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9000

# Test connectivity
telnet localhost 9000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep portainer)

# Check disk I/O
iotop -p $(pgrep portainer)

# Check connections
ss -an | grep 9000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  portainer:
    image: portainer:latest
    ports:
      - "9000:9000"
    volumes:
      - ./config:/etc/portainer
      - ./data:/var/lib/portainer
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update portainer

# Debian/Ubuntu
sudo apt update && sudo apt upgrade portainer

# Arch Linux
sudo pacman -Syu portainer

# Alpine Linux
apk update && apk upgrade portainer

# openSUSE
sudo zypper update portainer

# FreeBSD
pkg update && pkg upgrade portainer

# Always backup before updates
tar -czf /backup/portainer-pre-update-$(date +%Y%m%d).tar.gz /etc/portainer

# Restart after updates
sudo systemctl restart portainer
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/portainer

# Clean old logs
find /var/log/portainer -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/portainer
```

## Additional Resources

- Official Documentation: https://docs.portainer.org/
- GitHub Repository: https://github.com/portainer/portainer
- Community Forum: https://forum.portainer.org/
- Best Practices Guide: https://docs.portainer.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
