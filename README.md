# harbor Installation Guide

harbor is a free and open-source cloud native registry. Harbor provides container image registry with security scanning, signing, and replication, serving as an enterprise alternative to Docker Hub

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
  - CPU: 2+ cores recommended
  - RAM: 4GB minimum (8GB+ recommended)
  - Storage: 40GB+ for images
  - Network: HTTPS for registry access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80/443 (default harbor port)
  - Port 4443 for notary
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

# Install harbor
sudo dnf install -y harbor

# Enable and start service
sudo systemctl enable --now harbor

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/443/tcp
sudo firewall-cmd --reload

# Verify installation
harbor --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install harbor
sudo apt install -y harbor

# Enable and start service
sudo systemctl enable --now harbor

# Configure firewall
sudo ufw allow 80/443

# Verify installation
harbor --version
```

### Arch Linux

```bash
# Install harbor
sudo pacman -S harbor

# Enable and start service
sudo systemctl enable --now harbor

# Verify installation
harbor --version
```

### Alpine Linux

```bash
# Install harbor
apk add --no-cache harbor

# Enable and start service
rc-update add harbor default
rc-service harbor start

# Verify installation
harbor --version
```

### openSUSE/SLES

```bash
# Install harbor
sudo zypper install -y harbor

# Enable and start service
sudo systemctl enable --now harbor

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/443/tcp
sudo firewall-cmd --reload

# Verify installation
harbor --version
```

### macOS

```bash
# Using Homebrew
brew install harbor

# Start service
brew services start harbor

# Verify installation
harbor --version
```

### FreeBSD

```bash
# Using pkg
pkg install harbor

# Enable in rc.conf
echo 'harbor_enable="YES"' >> /etc/rc.conf

# Start service
service harbor start

# Verify installation
harbor --version
```

### Windows

```bash
# Using Chocolatey
choco install harbor

# Or using Scoop
scoop install harbor

# Verify installation
harbor --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/harbor

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
harbor --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable harbor

# Start service
sudo systemctl start harbor

# Stop service
sudo systemctl stop harbor

# Restart service
sudo systemctl restart harbor

# Check status
sudo systemctl status harbor

# View logs
sudo journalctl -u harbor -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add harbor default

# Start service
rc-service harbor start

# Stop service
rc-service harbor stop

# Restart service
rc-service harbor restart

# Check status
rc-service harbor status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'harbor_enable="YES"' >> /etc/rc.conf

# Start service
service harbor start

# Stop service
service harbor stop

# Restart service
service harbor restart

# Check status
service harbor status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start harbor
brew services stop harbor
brew services restart harbor

# Check status
brew services list | grep harbor
```

### Windows Service Manager

```powershell
# Start service
net start harbor

# Stop service
net stop harbor

# Using PowerShell
Start-Service harbor
Stop-Service harbor
Restart-Service harbor

# Check status
Get-Service harbor
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream harbor_backend {
    server 127.0.0.1:80/443;
}

server {
    listen 80;
    server_name harbor.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name harbor.example.com;

    ssl_certificate /etc/ssl/certs/harbor.example.com.crt;
    ssl_certificate_key /etc/ssl/private/harbor.example.com.key;

    location / {
        proxy_pass http://harbor_backend;
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
    ServerName harbor.example.com
    Redirect permanent / https://harbor.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName harbor.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/harbor.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/harbor.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/443/
    ProxyPassReverse / http://127.0.0.1:80/443/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend harbor_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/harbor.pem
    redirect scheme https if !{ ssl_fc }
    default_backend harbor_backend

backend harbor_backend
    balance roundrobin
    server harbor1 127.0.0.1:80/443 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R harbor:harbor /etc/harbor
sudo chmod 750 /etc/harbor

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/443/tcp
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
sudo systemctl status harbor

# View logs
sudo journalctl -u harbor -f

# Monitor resource usage
top -p $(pgrep harbor)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/harbor"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/harbor-backup-$DATE.tar.gz" /etc/harbor /var/lib/harbor

echo "Backup completed: $BACKUP_DIR/harbor-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop harbor

# Restore from backup
tar -xzf /backup/harbor/harbor-backup-*.tar.gz -C /

# Start service
sudo systemctl start harbor
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u harbor -n 100
sudo tail -f /var/log/harbor/harbor.log

# Check configuration
harbor --version

# Check permissions
ls -la /etc/harbor
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80/443

# Test connectivity
telnet localhost 80/443

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep harbor)

# Check disk I/O
iotop -p $(pgrep harbor)

# Check connections
ss -an | grep 80/443
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  harbor:
    image: harbor:latest
    ports:
      - "80/443:80/443"
    volumes:
      - ./config:/etc/harbor
      - ./data:/var/lib/harbor
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update harbor

# Debian/Ubuntu
sudo apt update && sudo apt upgrade harbor

# Arch Linux
sudo pacman -Syu harbor

# Alpine Linux
apk update && apk upgrade harbor

# openSUSE
sudo zypper update harbor

# FreeBSD
pkg update && pkg upgrade harbor

# Always backup before updates
tar -czf /backup/harbor-pre-update-$(date +%Y%m%d).tar.gz /etc/harbor

# Restart after updates
sudo systemctl restart harbor
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/harbor

# Clean old logs
find /var/log/harbor -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/harbor
```

## Additional Resources

- Official Documentation: https://docs.harbor.org/
- GitHub Repository: https://github.com/harbor/harbor
- Community Forum: https://forum.harbor.org/
- Best Practices Guide: https://docs.harbor.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
