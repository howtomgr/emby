# emby Installation Guide

emby is a free and open-source media server system. Emby provides media streaming and management, serving as an alternative to Plex with more open components

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
  - CPU: 2+ cores
  - RAM: 2GB minimum
  - Storage: 100GB+ for media
  - Network: Streaming bandwidth
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8096 (default emby port)
  - Port 8920 for HTTPS
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

# Install emby
sudo dnf install -y emby

# Enable and start service
sudo systemctl enable --now emby

# Configure firewall
sudo firewall-cmd --permanent --add-port=8096/tcp
sudo firewall-cmd --reload

# Verify installation
emby --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install emby
sudo apt install -y emby

# Enable and start service
sudo systemctl enable --now emby

# Configure firewall
sudo ufw allow 8096

# Verify installation
emby --version
```

### Arch Linux

```bash
# Install emby
sudo pacman -S emby

# Enable and start service
sudo systemctl enable --now emby

# Verify installation
emby --version
```

### Alpine Linux

```bash
# Install emby
apk add --no-cache emby

# Enable and start service
rc-update add emby default
rc-service emby start

# Verify installation
emby --version
```

### openSUSE/SLES

```bash
# Install emby
sudo zypper install -y emby

# Enable and start service
sudo systemctl enable --now emby

# Configure firewall
sudo firewall-cmd --permanent --add-port=8096/tcp
sudo firewall-cmd --reload

# Verify installation
emby --version
```

### macOS

```bash
# Using Homebrew
brew install emby

# Start service
brew services start emby

# Verify installation
emby --version
```

### FreeBSD

```bash
# Using pkg
pkg install emby

# Enable in rc.conf
echo 'emby_enable="YES"' >> /etc/rc.conf

# Start service
service emby start

# Verify installation
emby --version
```

### Windows

```bash
# Using Chocolatey
choco install emby

# Or using Scoop
scoop install emby

# Verify installation
emby --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/emby

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
emby --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable emby

# Start service
sudo systemctl start emby

# Stop service
sudo systemctl stop emby

# Restart service
sudo systemctl restart emby

# Check status
sudo systemctl status emby

# View logs
sudo journalctl -u emby -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add emby default

# Start service
rc-service emby start

# Stop service
rc-service emby stop

# Restart service
rc-service emby restart

# Check status
rc-service emby status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'emby_enable="YES"' >> /etc/rc.conf

# Start service
service emby start

# Stop service
service emby stop

# Restart service
service emby restart

# Check status
service emby status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start emby
brew services stop emby
brew services restart emby

# Check status
brew services list | grep emby
```

### Windows Service Manager

```powershell
# Start service
net start emby

# Stop service
net stop emby

# Using PowerShell
Start-Service emby
Stop-Service emby
Restart-Service emby

# Check status
Get-Service emby
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream emby_backend {
    server 127.0.0.1:8096;
}

server {
    listen 80;
    server_name emby.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name emby.example.com;

    ssl_certificate /etc/ssl/certs/emby.example.com.crt;
    ssl_certificate_key /etc/ssl/private/emby.example.com.key;

    location / {
        proxy_pass http://emby_backend;
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
    ServerName emby.example.com
    Redirect permanent / https://emby.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName emby.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/emby.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/emby.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8096/
    ProxyPassReverse / http://127.0.0.1:8096/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend emby_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/emby.pem
    redirect scheme https if !{ ssl_fc }
    default_backend emby_backend

backend emby_backend
    balance roundrobin
    server emby1 127.0.0.1:8096 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R emby:emby /etc/emby
sudo chmod 750 /etc/emby

# Configure firewall
sudo firewall-cmd --permanent --add-port=8096/tcp
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
sudo systemctl status emby

# View logs
sudo journalctl -u emby -f

# Monitor resource usage
top -p $(pgrep emby)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/emby"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/emby-backup-$DATE.tar.gz" /etc/emby /var/lib/emby

echo "Backup completed: $BACKUP_DIR/emby-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop emby

# Restore from backup
tar -xzf /backup/emby/emby-backup-*.tar.gz -C /

# Start service
sudo systemctl start emby
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u emby -n 100
sudo tail -f /var/log/emby/emby.log

# Check configuration
emby --version

# Check permissions
ls -la /etc/emby
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8096

# Test connectivity
telnet localhost 8096

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep emby)

# Check disk I/O
iotop -p $(pgrep emby)

# Check connections
ss -an | grep 8096
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  emby:
    image: emby:latest
    ports:
      - "8096:8096"
    volumes:
      - ./config:/etc/emby
      - ./data:/var/lib/emby
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update emby

# Debian/Ubuntu
sudo apt update && sudo apt upgrade emby

# Arch Linux
sudo pacman -Syu emby

# Alpine Linux
apk update && apk upgrade emby

# openSUSE
sudo zypper update emby

# FreeBSD
pkg update && pkg upgrade emby

# Always backup before updates
tar -czf /backup/emby-pre-update-$(date +%Y%m%d).tar.gz /etc/emby

# Restart after updates
sudo systemctl restart emby
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/emby

# Clean old logs
find /var/log/emby -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/emby
```

## Additional Resources

- Official Documentation: https://docs.emby.org/
- GitHub Repository: https://github.com/emby/emby
- Community Forum: https://forum.emby.org/
- Best Practices Guide: https://docs.emby.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
