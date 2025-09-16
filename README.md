# skipper Installation Guide

skipper is a free and open-source HTTP router. Skipper provides HTTP router and reverse proxy for service composition

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
  - Storage: 1GB for config
  - Network: HTTP/HTTPS
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9090 (default skipper port)
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

# Install skipper
sudo dnf install -y skipper

# Enable and start service
sudo systemctl enable --now skipper

# Configure firewall
sudo firewall-cmd --permanent --add-port=9090/tcp
sudo firewall-cmd --reload

# Verify installation
skipper --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install skipper
sudo apt install -y skipper

# Enable and start service
sudo systemctl enable --now skipper

# Configure firewall
sudo ufw allow 9090

# Verify installation
skipper --version
```

### Arch Linux

```bash
# Install skipper
sudo pacman -S skipper

# Enable and start service
sudo systemctl enable --now skipper

# Verify installation
skipper --version
```

### Alpine Linux

```bash
# Install skipper
apk add --no-cache skipper

# Enable and start service
rc-update add skipper default
rc-service skipper start

# Verify installation
skipper --version
```

### openSUSE/SLES

```bash
# Install skipper
sudo zypper install -y skipper

# Enable and start service
sudo systemctl enable --now skipper

# Configure firewall
sudo firewall-cmd --permanent --add-port=9090/tcp
sudo firewall-cmd --reload

# Verify installation
skipper --version
```

### macOS

```bash
# Using Homebrew
brew install skipper

# Start service
brew services start skipper

# Verify installation
skipper --version
```

### FreeBSD

```bash
# Using pkg
pkg install skipper

# Enable in rc.conf
echo 'skipper_enable="YES"' >> /etc/rc.conf

# Start service
service skipper start

# Verify installation
skipper --version
```

### Windows

```bash
# Using Chocolatey
choco install skipper

# Or using Scoop
scoop install skipper

# Verify installation
skipper --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/skipper

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
skipper --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable skipper

# Start service
sudo systemctl start skipper

# Stop service
sudo systemctl stop skipper

# Restart service
sudo systemctl restart skipper

# Check status
sudo systemctl status skipper

# View logs
sudo journalctl -u skipper -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add skipper default

# Start service
rc-service skipper start

# Stop service
rc-service skipper stop

# Restart service
rc-service skipper restart

# Check status
rc-service skipper status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'skipper_enable="YES"' >> /etc/rc.conf

# Start service
service skipper start

# Stop service
service skipper stop

# Restart service
service skipper restart

# Check status
service skipper status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start skipper
brew services stop skipper
brew services restart skipper

# Check status
brew services list | grep skipper
```

### Windows Service Manager

```powershell
# Start service
net start skipper

# Stop service
net stop skipper

# Using PowerShell
Start-Service skipper
Stop-Service skipper
Restart-Service skipper

# Check status
Get-Service skipper
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream skipper_backend {
    server 127.0.0.1:9090;
}

server {
    listen 80;
    server_name skipper.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name skipper.example.com;

    ssl_certificate /etc/ssl/certs/skipper.example.com.crt;
    ssl_certificate_key /etc/ssl/private/skipper.example.com.key;

    location / {
        proxy_pass http://skipper_backend;
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
    ServerName skipper.example.com
    Redirect permanent / https://skipper.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName skipper.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/skipper.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/skipper.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9090/
    ProxyPassReverse / http://127.0.0.1:9090/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend skipper_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/skipper.pem
    redirect scheme https if !{ ssl_fc }
    default_backend skipper_backend

backend skipper_backend
    balance roundrobin
    server skipper1 127.0.0.1:9090 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R skipper:skipper /etc/skipper
sudo chmod 750 /etc/skipper

# Configure firewall
sudo firewall-cmd --permanent --add-port=9090/tcp
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
sudo systemctl status skipper

# View logs
sudo journalctl -u skipper -f

# Monitor resource usage
top -p $(pgrep skipper)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/skipper"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/skipper-backup-$DATE.tar.gz" /etc/skipper /var/lib/skipper

echo "Backup completed: $BACKUP_DIR/skipper-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop skipper

# Restore from backup
tar -xzf /backup/skipper/skipper-backup-*.tar.gz -C /

# Start service
sudo systemctl start skipper
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u skipper -n 100
sudo tail -f /var/log/skipper/skipper.log

# Check configuration
skipper --version

# Check permissions
ls -la /etc/skipper
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9090

# Test connectivity
telnet localhost 9090

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep skipper)

# Check disk I/O
iotop -p $(pgrep skipper)

# Check connections
ss -an | grep 9090
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  skipper:
    image: skipper:latest
    ports:
      - "9090:9090"
    volumes:
      - ./config:/etc/skipper
      - ./data:/var/lib/skipper
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update skipper

# Debian/Ubuntu
sudo apt update && sudo apt upgrade skipper

# Arch Linux
sudo pacman -Syu skipper

# Alpine Linux
apk update && apk upgrade skipper

# openSUSE
sudo zypper update skipper

# FreeBSD
pkg update && pkg upgrade skipper

# Always backup before updates
tar -czf /backup/skipper-pre-update-$(date +%Y%m%d).tar.gz /etc/skipper

# Restart after updates
sudo systemctl restart skipper
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/skipper

# Clean old logs
find /var/log/skipper -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/skipper
```

## Additional Resources

- Official Documentation: https://docs.skipper.org/
- GitHub Repository: https://github.com/skipper/skipper
- Community Forum: https://forum.skipper.org/
- Best Practices Guide: https://docs.skipper.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
