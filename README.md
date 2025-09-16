# bookstack Installation Guide

bookstack is a free and open-source documentation platform. BookStack provides simple, self-hosted platform for organizing documentation

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
  - RAM: 512MB minimum
  - Storage: 1GB for data
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default bookstack port)
  - None
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

# Install bookstack
sudo dnf install -y bookstack

# Enable and start service
sudo systemctl enable --now bookstack

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
bookstack --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install bookstack
sudo apt install -y bookstack

# Enable and start service
sudo systemctl enable --now bookstack

# Configure firewall
sudo ufw allow 80

# Verify installation
bookstack --version
```

### Arch Linux

```bash
# Install bookstack
sudo pacman -S bookstack

# Enable and start service
sudo systemctl enable --now bookstack

# Verify installation
bookstack --version
```

### Alpine Linux

```bash
# Install bookstack
apk add --no-cache bookstack

# Enable and start service
rc-update add bookstack default
rc-service bookstack start

# Verify installation
bookstack --version
```

### openSUSE/SLES

```bash
# Install bookstack
sudo zypper install -y bookstack

# Enable and start service
sudo systemctl enable --now bookstack

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
bookstack --version
```

### macOS

```bash
# Using Homebrew
brew install bookstack

# Start service
brew services start bookstack

# Verify installation
bookstack --version
```

### FreeBSD

```bash
# Using pkg
pkg install bookstack

# Enable in rc.conf
echo 'bookstack_enable="YES"' >> /etc/rc.conf

# Start service
service bookstack start

# Verify installation
bookstack --version
```

### Windows

```bash
# Using Chocolatey
choco install bookstack

# Or using Scoop
scoop install bookstack

# Verify installation
bookstack --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/bookstack

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
bookstack --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable bookstack

# Start service
sudo systemctl start bookstack

# Stop service
sudo systemctl stop bookstack

# Restart service
sudo systemctl restart bookstack

# Check status
sudo systemctl status bookstack

# View logs
sudo journalctl -u bookstack -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add bookstack default

# Start service
rc-service bookstack start

# Stop service
rc-service bookstack stop

# Restart service
rc-service bookstack restart

# Check status
rc-service bookstack status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'bookstack_enable="YES"' >> /etc/rc.conf

# Start service
service bookstack start

# Stop service
service bookstack stop

# Restart service
service bookstack restart

# Check status
service bookstack status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start bookstack
brew services stop bookstack
brew services restart bookstack

# Check status
brew services list | grep bookstack
```

### Windows Service Manager

```powershell
# Start service
net start bookstack

# Stop service
net stop bookstack

# Using PowerShell
Start-Service bookstack
Stop-Service bookstack
Restart-Service bookstack

# Check status
Get-Service bookstack
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream bookstack_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name bookstack.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name bookstack.example.com;

    ssl_certificate /etc/ssl/certs/bookstack.example.com.crt;
    ssl_certificate_key /etc/ssl/private/bookstack.example.com.key;

    location / {
        proxy_pass http://bookstack_backend;
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
    ServerName bookstack.example.com
    Redirect permanent / https://bookstack.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName bookstack.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/bookstack.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/bookstack.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend bookstack_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/bookstack.pem
    redirect scheme https if !{ ssl_fc }
    default_backend bookstack_backend

backend bookstack_backend
    balance roundrobin
    server bookstack1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R bookstack:bookstack /etc/bookstack
sudo chmod 750 /etc/bookstack

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
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
sudo systemctl status bookstack

# View logs
sudo journalctl -u bookstack -f

# Monitor resource usage
top -p $(pgrep bookstack)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/bookstack"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/bookstack-backup-$DATE.tar.gz" /etc/bookstack /var/lib/bookstack

echo "Backup completed: $BACKUP_DIR/bookstack-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop bookstack

# Restore from backup
tar -xzf /backup/bookstack/bookstack-backup-*.tar.gz -C /

# Start service
sudo systemctl start bookstack
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u bookstack -n 100
sudo tail -f /var/log/bookstack/bookstack.log

# Check configuration
bookstack --version

# Check permissions
ls -la /etc/bookstack
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80

# Test connectivity
telnet localhost 80

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep bookstack)

# Check disk I/O
iotop -p $(pgrep bookstack)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  bookstack:
    image: bookstack:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/bookstack
      - ./data:/var/lib/bookstack
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update bookstack

# Debian/Ubuntu
sudo apt update && sudo apt upgrade bookstack

# Arch Linux
sudo pacman -Syu bookstack

# Alpine Linux
apk update && apk upgrade bookstack

# openSUSE
sudo zypper update bookstack

# FreeBSD
pkg update && pkg upgrade bookstack

# Always backup before updates
tar -czf /backup/bookstack-pre-update-$(date +%Y%m%d).tar.gz /etc/bookstack

# Restart after updates
sudo systemctl restart bookstack
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/bookstack

# Clean old logs
find /var/log/bookstack -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/bookstack
```

## Additional Resources

- Official Documentation: https://docs.bookstack.org/
- GitHub Repository: https://github.com/bookstack/bookstack
- Community Forum: https://forum.bookstack.org/
- Best Practices Guide: https://docs.bookstack.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
