# nextcloud Installation Guide

nextcloud is a free and open-source self-hosted file sync and collaboration platform. Nextcloud provides file hosting, sync, and collaboration features, serving as a privacy-focused alternative to Google Drive, Dropbox, or Microsoft OneDrive

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
  - RAM: 512MB minimum (2GB+ recommended)
  - Storage: 10GB+ for data
  - Network: HTTPS for secure access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80/443 (default nextcloud port)
  - Port 3478 for TURN server
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

# Install nextcloud
sudo dnf install -y nextcloud

# Enable and start service
sudo systemctl enable --now nextcloud

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/443/tcp
sudo firewall-cmd --reload

# Verify installation
occ --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install nextcloud
sudo apt install -y nextcloud

# Enable and start service
sudo systemctl enable --now nextcloud

# Configure firewall
sudo ufw allow 80/443

# Verify installation
occ --version
```

### Arch Linux

```bash
# Install nextcloud
sudo pacman -S nextcloud

# Enable and start service
sudo systemctl enable --now nextcloud

# Verify installation
occ --version
```

### Alpine Linux

```bash
# Install nextcloud
apk add --no-cache nextcloud

# Enable and start service
rc-update add nextcloud default
rc-service nextcloud start

# Verify installation
occ --version
```

### openSUSE/SLES

```bash
# Install nextcloud
sudo zypper install -y nextcloud

# Enable and start service
sudo systemctl enable --now nextcloud

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/443/tcp
sudo firewall-cmd --reload

# Verify installation
occ --version
```

### macOS

```bash
# Using Homebrew
brew install nextcloud

# Start service
brew services start nextcloud

# Verify installation
occ --version
```

### FreeBSD

```bash
# Using pkg
pkg install nextcloud

# Enable in rc.conf
echo 'nextcloud_enable="YES"' >> /etc/rc.conf

# Start service
service nextcloud start

# Verify installation
occ --version
```

### Windows

```bash
# Using Chocolatey
choco install nextcloud

# Or using Scoop
scoop install nextcloud

# Verify installation
occ --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/nextcloud

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
occ --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable nextcloud

# Start service
sudo systemctl start nextcloud

# Stop service
sudo systemctl stop nextcloud

# Restart service
sudo systemctl restart nextcloud

# Check status
sudo systemctl status nextcloud

# View logs
sudo journalctl -u nextcloud -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add nextcloud default

# Start service
rc-service nextcloud start

# Stop service
rc-service nextcloud stop

# Restart service
rc-service nextcloud restart

# Check status
rc-service nextcloud status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'nextcloud_enable="YES"' >> /etc/rc.conf

# Start service
service nextcloud start

# Stop service
service nextcloud stop

# Restart service
service nextcloud restart

# Check status
service nextcloud status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start nextcloud
brew services stop nextcloud
brew services restart nextcloud

# Check status
brew services list | grep nextcloud
```

### Windows Service Manager

```powershell
# Start service
net start nextcloud

# Stop service
net stop nextcloud

# Using PowerShell
Start-Service nextcloud
Stop-Service nextcloud
Restart-Service nextcloud

# Check status
Get-Service nextcloud
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream nextcloud_backend {
    server 127.0.0.1:80/443;
}

server {
    listen 80;
    server_name nextcloud.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name nextcloud.example.com;

    ssl_certificate /etc/ssl/certs/nextcloud.example.com.crt;
    ssl_certificate_key /etc/ssl/private/nextcloud.example.com.key;

    location / {
        proxy_pass http://nextcloud_backend;
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
    ServerName nextcloud.example.com
    Redirect permanent / https://nextcloud.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName nextcloud.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/nextcloud.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/nextcloud.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/443/
    ProxyPassReverse / http://127.0.0.1:80/443/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend nextcloud_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/nextcloud.pem
    redirect scheme https if !{ ssl_fc }
    default_backend nextcloud_backend

backend nextcloud_backend
    balance roundrobin
    server nextcloud1 127.0.0.1:80/443 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R nextcloud:nextcloud /etc/nextcloud
sudo chmod 750 /etc/nextcloud

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
sudo systemctl status nextcloud

# View logs
sudo journalctl -u nextcloud -f

# Monitor resource usage
top -p $(pgrep nextcloud)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/nextcloud"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/nextcloud-backup-$DATE.tar.gz" /etc/nextcloud /var/lib/nextcloud

echo "Backup completed: $BACKUP_DIR/nextcloud-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop nextcloud

# Restore from backup
tar -xzf /backup/nextcloud/nextcloud-backup-*.tar.gz -C /

# Start service
sudo systemctl start nextcloud
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u nextcloud -n 100
sudo tail -f /var/log/nextcloud/nextcloud.log

# Check configuration
occ --version

# Check permissions
ls -la /etc/nextcloud
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
top -p $(pgrep nextcloud)

# Check disk I/O
iotop -p $(pgrep nextcloud)

# Check connections
ss -an | grep 80/443
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  nextcloud:
    image: nextcloud:latest
    ports:
      - "80/443:80/443"
    volumes:
      - ./config:/etc/nextcloud
      - ./data:/var/lib/nextcloud
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update nextcloud

# Debian/Ubuntu
sudo apt update && sudo apt upgrade nextcloud

# Arch Linux
sudo pacman -Syu nextcloud

# Alpine Linux
apk update && apk upgrade nextcloud

# openSUSE
sudo zypper update nextcloud

# FreeBSD
pkg update && pkg upgrade nextcloud

# Always backup before updates
tar -czf /backup/nextcloud-pre-update-$(date +%Y%m%d).tar.gz /etc/nextcloud

# Restart after updates
sudo systemctl restart nextcloud
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/nextcloud

# Clean old logs
find /var/log/nextcloud -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/nextcloud
```

## Additional Resources

- Official Documentation: https://docs.nextcloud.org/
- GitHub Repository: https://github.com/nextcloud/nextcloud
- Community Forum: https://forum.nextcloud.org/
- Best Practices Guide: https://docs.nextcloud.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
