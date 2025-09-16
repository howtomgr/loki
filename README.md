# grafana-loki Installation Guide

grafana-loki is a free and open-source horizontally-scalable log aggregation system. Developed by Grafana Labs, Loki provides efficient log aggregation inspired by Prometheus, serving as an alternative to Elasticsearch/ELK or Splunk

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
  - RAM: 1GB minimum (4GB+ recommended)
  - Storage: 10GB+ for logs
  - Network: HTTP/gRPC connectivity
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 3100 (default grafana-loki port)
  - Port 9095 for gRPC
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

# Install grafana-loki
sudo dnf install -y loki

# Enable and start service
sudo systemctl enable --now loki

# Configure firewall
sudo firewall-cmd --permanent --add-port=3100/tcp
sudo firewall-cmd --reload

# Verify installation
loki --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install grafana-loki
sudo apt install -y loki

# Enable and start service
sudo systemctl enable --now loki

# Configure firewall
sudo ufw allow 3100

# Verify installation
loki --version
```

### Arch Linux

```bash
# Install grafana-loki
sudo pacman -S loki

# Enable and start service
sudo systemctl enable --now loki

# Verify installation
loki --version
```

### Alpine Linux

```bash
# Install grafana-loki
apk add --no-cache loki

# Enable and start service
rc-update add loki default
rc-service loki start

# Verify installation
loki --version
```

### openSUSE/SLES

```bash
# Install grafana-loki
sudo zypper install -y loki

# Enable and start service
sudo systemctl enable --now loki

# Configure firewall
sudo firewall-cmd --permanent --add-port=3100/tcp
sudo firewall-cmd --reload

# Verify installation
loki --version
```

### macOS

```bash
# Using Homebrew
brew install loki

# Start service
brew services start loki

# Verify installation
loki --version
```

### FreeBSD

```bash
# Using pkg
pkg install loki

# Enable in rc.conf
echo 'loki_enable="YES"' >> /etc/rc.conf

# Start service
service loki start

# Verify installation
loki --version
```

### Windows

```bash
# Using Chocolatey
choco install loki

# Or using Scoop
scoop install loki

# Verify installation
loki --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/loki

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
loki --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable loki

# Start service
sudo systemctl start loki

# Stop service
sudo systemctl stop loki

# Restart service
sudo systemctl restart loki

# Check status
sudo systemctl status loki

# View logs
sudo journalctl -u loki -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add loki default

# Start service
rc-service loki start

# Stop service
rc-service loki stop

# Restart service
rc-service loki restart

# Check status
rc-service loki status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'loki_enable="YES"' >> /etc/rc.conf

# Start service
service loki start

# Stop service
service loki stop

# Restart service
service loki restart

# Check status
service loki status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start loki
brew services stop loki
brew services restart loki

# Check status
brew services list | grep loki
```

### Windows Service Manager

```powershell
# Start service
net start loki

# Stop service
net stop loki

# Using PowerShell
Start-Service loki
Stop-Service loki
Restart-Service loki

# Check status
Get-Service loki
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream loki_backend {
    server 127.0.0.1:3100;
}

server {
    listen 80;
    server_name loki.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name loki.example.com;

    ssl_certificate /etc/ssl/certs/loki.example.com.crt;
    ssl_certificate_key /etc/ssl/private/loki.example.com.key;

    location / {
        proxy_pass http://loki_backend;
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
    ServerName loki.example.com
    Redirect permanent / https://loki.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName loki.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/loki.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/loki.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:3100/
    ProxyPassReverse / http://127.0.0.1:3100/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend loki_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/loki.pem
    redirect scheme https if !{ ssl_fc }
    default_backend loki_backend

backend loki_backend
    balance roundrobin
    server loki1 127.0.0.1:3100 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R loki:loki /etc/loki
sudo chmod 750 /etc/loki

# Configure firewall
sudo firewall-cmd --permanent --add-port=3100/tcp
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
sudo systemctl status loki

# View logs
sudo journalctl -u loki -f

# Monitor resource usage
top -p $(pgrep loki)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/loki"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/loki-backup-$DATE.tar.gz" /etc/loki /var/lib/loki

echo "Backup completed: $BACKUP_DIR/loki-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop loki

# Restore from backup
tar -xzf /backup/loki/loki-backup-*.tar.gz -C /

# Start service
sudo systemctl start loki
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u loki -n 100
sudo tail -f /var/log/loki/loki.log

# Check configuration
loki --version

# Check permissions
ls -la /etc/loki
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 3100

# Test connectivity
telnet localhost 3100

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep loki)

# Check disk I/O
iotop -p $(pgrep loki)

# Check connections
ss -an | grep 3100
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  loki:
    image: loki:latest
    ports:
      - "3100:3100"
    volumes:
      - ./config:/etc/loki
      - ./data:/var/lib/loki
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update loki

# Debian/Ubuntu
sudo apt update && sudo apt upgrade loki

# Arch Linux
sudo pacman -Syu loki

# Alpine Linux
apk update && apk upgrade loki

# openSUSE
sudo zypper update loki

# FreeBSD
pkg update && pkg upgrade loki

# Always backup before updates
tar -czf /backup/loki-pre-update-$(date +%Y%m%d).tar.gz /etc/loki

# Restart after updates
sudo systemctl restart loki
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/loki

# Clean old logs
find /var/log/loki -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/loki
```

## Additional Resources

- Official Documentation: https://docs.loki.org/
- GitHub Repository: https://github.com/loki/loki
- Community Forum: https://forum.loki.org/
- Best Practices Guide: https://docs.loki.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
