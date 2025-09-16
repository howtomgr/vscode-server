# VS Code Server Installation Guide

VS Code Server (code-server) is a free and open-source implementation of Visual Studio Code running on a remote server, accessible through a web browser. Developed by Coder, it allows developers to use VS Code on any device with a consistent development environment. As a FOSS alternative to Microsoft's proprietary Visual Studio Code Remote Development, code-server provides full IDE functionality without requiring local installations

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
  - CPU: 2+ cores (4+ recommended)
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 10GB for workspace and extensions
- **Operating System**: Linux, macOS, FreeBSD, or Windows
- **Network Requirements**:
  - Port 8080 (default code-server port)
  - HTTPS recommended for production use
- **Dependencies**:
  - Node.js 16+ (for source installation)
  - Git
  - Build tools (for compiling native extensions)
- **System Access**: root or sudo privileges required for system-wide installation


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

# Install using official installer script
curl -fsSL https://code-server.dev/install.sh | sh

# Or install from release tarball
wget https://github.com/coder/code-server/releases/download/v4.19.0/code-server-4.19.0-linux-amd64.tar.gz
tar -xzf code-server-4.19.0-linux-amd64.tar.gz
sudo mv code-server-4.19.0-linux-amd64 /opt/code-server
sudo ln -s /opt/code-server/bin/code-server /usr/local/bin/code-server

# Create systemd service
sudo tee /etc/systemd/system/code-server@.service > /dev/null <<EOF
[Unit]
Description=code-server
After=network.target

[Service]
Type=exec
ExecStart=/usr/local/bin/code-server --bind-addr 0.0.0.0:8080
Restart=always
User=%i

[Install]
WantedBy=default.target
EOF

# Enable and start service
sudo systemctl enable --now code-server@$USER

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
code-server --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install using official installer script
curl -fsSL https://code-server.dev/install.sh | sh

# Or install using deb package
wget https://github.com/coder/code-server/releases/download/v4.19.0/code-server_4.19.0_amd64.deb
sudo dpkg -i code-server_4.19.0_amd64.deb

# Enable and start service
sudo systemctl enable --now code-server@$USER

# Configure firewall
sudo ufw allow 8080/tcp

# Verify installation
code-server --version
```

### Arch Linux

```bash
# Install from AUR
yay -S code-server

# Or install from AUR (alternative)
paru -S code-server

# Enable and start service
sudo systemctl enable --now code-server@$USER

# Verify installation
code-server --version
```

### Alpine Linux

```bash
# Install dependencies
apk add --no-cache nodejs npm python3 make g++

# Install using npm
npm install -g code-server

# Create OpenRC service
sudo tee /etc/init.d/code-server > /dev/null <<'EOF'
#!/sbin/openrc-run

name="code-server"
description="VS Code Server"

command="/usr/bin/code-server"
command_args="--bind-addr 0.0.0.0:8080"
command_user="${USER:-root}"
pidfile="/run/${RC_SVCNAME}.pid"
command_background="yes"

depend() {
    need net
    after firewall
}
EOF

sudo chmod +x /etc/init.d/code-server

# Enable and start service
rc-update add code-server default
rc-service code-server start

# Verify installation
code-server --version
```

### openSUSE/SLES

```bash
# Install dependencies
sudo zypper install -y nodejs npm python3 make gcc-c++

# Install using npm
sudo npm install -g code-server

# Create systemd service
sudo tee /etc/systemd/system/code-server@.service > /dev/null <<EOF
[Unit]
Description=code-server
After=network.target

[Service]
Type=exec
ExecStart=/usr/bin/code-server --bind-addr 0.0.0.0:8080
Restart=always
User=%i

[Install]
WantedBy=default.target
EOF

# Enable and start service
sudo systemctl enable --now code-server@$USER

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
code-server --version
```

### macOS

```bash
# Using Homebrew
brew install code-server

# Start service
brew services start code-server

# Or run manually
code-server --bind-addr 0.0.0.0:8080

# Verify installation
code-server --version
```

### FreeBSD

```bash
# Install dependencies
pkg install -y node npm python

# Install using npm
npm install -g code-server

# Create rc.d script
sudo tee /usr/local/etc/rc.d/code-server > /dev/null <<'EOF'
#!/bin/sh

# PROVIDE: code-server
# REQUIRE: NETWORKING
# KEYWORD: shutdown

. /etc/rc.subr

name="code_server"
rcvar="code_server_enable"

load_rc_config $name

: ${code_server_enable:="NO"}
: ${code_server_user:="www"}
: ${code_server_bind:="0.0.0.0:8080"}

command="/usr/local/bin/code-server"
command_args="--bind-addr ${code_server_bind}"

run_rc_command "$1"
EOF

sudo chmod +x /usr/local/etc/rc.d/code-server

# Enable in rc.conf
echo 'code_server_enable="YES"' | sudo tee -a /etc/rc.conf

# Start service
service code-server start

# Verify installation
code-server --version
```

### Windows

```bash
# Using npm (with Node.js installed)
npm install -g code-server

# Or download Windows release
# Download from: https://github.com/coder/code-server/releases
# Extract and add to PATH

# Run code-server
code-server --bind-addr 0.0.0.0:8080

# Install as Windows service using NSSM
# Download NSSM from: https://nssm.cc/download
nssm install code-server "C:\Program Files\code-server\bin\code-server.exe" "--bind-addr 0.0.0.0:8080"
nssm start code-server

# Verify installation
code-server --version
```

## 4. Configuration

### Basic Configuration

```bash
# Create configuration directory
mkdir -p ~/.config/code-server

# Create configuration file
cat > ~/.config/code-server/config.yaml <<EOF
bind-addr: 0.0.0.0:8080
auth: password
password: your-secure-password-here
cert: false
EOF

# For HTTPS (recommended for production)
cat > ~/.config/code-server/config.yaml <<EOF
bind-addr: 0.0.0.0:8443
auth: password
password: your-secure-password-here
cert: true
cert-key: /path/to/privkey.pem
cert-file: /path/to/fullchain.pem
EOF

# Test configuration
code-server --config ~/.config/code-server/config.yaml
```

### Environment Variables

```bash
# Set environment variables
export PASSWORD="your-secure-password"
export HASHED_PASSWORD='$argon2i$v=19$m=4096,t=3,p=1$...' # Use --hashed-password
export CODE_SERVER_CONFIG="~/.config/code-server/config.yaml"

# Generate hashed password
echo -n "your-password" | npx argon2-cli -e
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable code-server

# Start service
sudo systemctl start code-server

# Stop service
sudo systemctl stop code-server

# Restart service
sudo systemctl restart code-server

# Check status
sudo systemctl status code-server

# View logs
sudo journalctl -u code-server -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add code-server default

# Start service
rc-service code-server start

# Stop service
rc-service code-server stop

# Restart service
rc-service code-server restart

# Check status
rc-service code-server status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'code-server_enable="YES"' >> /etc/rc.conf

# Start service
service code-server start

# Stop service
service code-server stop

# Restart service
service code-server restart

# Check status
service code-server status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start code_server
brew services stop code_server
brew services restart code_server

# Check status
brew services list | grep code_server
```

### Windows Service Manager

```powershell
# Start service
net start code-server

# Stop service
net stop code-server

# Using PowerShell
Start-Service code-server
Stop-Service code-server
Restart-Service code-server

# Check status
Get-Service code-server
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream code_server_backend {
    server 127.0.0.1:8080;
}

server {
    listen 80;
    server_name code_server.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name code_server.example.com;

    ssl_certificate /etc/ssl/certs/code_server.example.com.crt;
    ssl_certificate_key /etc/ssl/private/code_server.example.com.key;

    location / {
        proxy_pass http://code_server_backend;
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
    ServerName code_server.example.com
    Redirect permanent / https://code_server.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName code_server.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/code_server.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/code_server.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend code_server_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/code_server.pem
    redirect scheme https if !{ ssl_fc }
    default_backend code_server_backend

backend code_server_backend
    balance roundrobin
    server code_server1 127.0.0.1:8080 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R code_server:code_server /etc/code_server
sudo chmod 750 /etc/code_server

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
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
sudo systemctl status code-server

# View logs
sudo journalctl -u code-server -f

# Monitor resource usage
top -p $(pgrep code_server)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/code_server"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/code_server-backup-$DATE.tar.gz" /etc/code_server /var/lib/code_server

echo "Backup completed: $BACKUP_DIR/code_server-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop code-server

# Restore from backup
tar -xzf /backup/code_server/code_server-backup-*.tar.gz -C /

# Start service
sudo systemctl start code-server
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u code-server -n 100
sudo tail -f /var/log/code_server/code_server.log

# Check configuration
code-server --version

# Check permissions
ls -la /etc/code_server
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8080

# Test connectivity
telnet localhost 8080

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep code_server)

# Check disk I/O
iotop -p $(pgrep code_server)

# Check connections
ss -an | grep 8080
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  code_server:
    image: code_server:latest
    ports:
      - "8080:8080"
    volumes:
      - ./config:/etc/code_server
      - ./data:/var/lib/code_server
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update code_server

# Debian/Ubuntu
sudo apt update && sudo apt upgrade code_server

# Arch Linux
sudo pacman -Syu code_server

# Alpine Linux
apk update && apk upgrade code_server

# openSUSE
sudo zypper update code_server

# FreeBSD
pkg update && pkg upgrade code_server

# Always backup before updates
tar -czf /backup/code_server-pre-update-$(date +%Y%m%d).tar.gz /etc/code_server

# Restart after updates
sudo systemctl restart code-server
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/code_server

# Clean old logs
find /var/log/code_server -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/code_server
```

## Additional Resources

- Official Documentation: https://docs.code_server.org/
- GitHub Repository: https://github.com/code_server/code_server
- Community Forum: https://forum.code_server.org/
- Best Practices Guide: https://docs.code_server.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
