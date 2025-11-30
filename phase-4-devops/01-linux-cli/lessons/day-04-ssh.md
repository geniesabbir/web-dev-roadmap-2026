# Day 4: SSH - Secure Remote Access

## Introduction

SSH (Secure Shell) is the standard protocol for securely accessing remote Linux servers. Whether you're deploying applications, managing cloud infrastructure, or connecting to development servers, SSH is an essential skill. Today, you'll learn how to connect securely, manage keys, and configure SSH for productivity.

## Learning Objectives

By the end of this lesson, you will be able to:
- Connect to remote servers using SSH
- Generate and manage SSH key pairs
- Configure SSH for convenience and security
- Transfer files securely with SCP and SFTP
- Use SSH tunneling for secure connections

---

## SSH Basics

### How SSH Works

```
┌──────────────┐         Encrypted Connection        ┌──────────────┐
│   Client     │ ◄────────────────────────────────► │   Server     │
│  (Your PC)   │                                     │ (Remote)     │
│              │     1. Client initiates             │              │
│ ~/.ssh/      │     2. Server sends public key      │ /etc/ssh/    │
│   id_rsa     │     3. Key exchange                 │   sshd_config│
│   id_rsa.pub │     4. Encrypted session            │              │
└──────────────┘                                     └──────────────┘
```

### Basic Connection

```bash
# Connect to server
ssh username@hostname
ssh username@192.168.1.100
ssh deploy@server.example.com

# Specify port (default is 22)
ssh -p 2222 user@hostname

# First connection - verify fingerprint
ssh user@newserver
# The authenticity of host 'newserver' can't be established.
# ECDSA key fingerprint is SHA256:abc123...
# Are you sure you want to continue connecting (yes/no)? yes

# Execute single command
ssh user@server "ls -la /var/www"
ssh user@server "df -h && free -m"

# Verbose mode (debugging)
ssh -v user@server
ssh -vvv user@server    # More verbose
```

---

## SSH Key Authentication

### Why Use Keys?

```
Password Authentication:
❌ Can be brute-forced
❌ Must remember/type password
❌ Harder to automate

Key Authentication:
✅ Virtually impossible to brute-force
✅ No password needed (optional passphrase)
✅ Perfect for automation
✅ Can be restricted per-key
```

### Generating SSH Keys

```bash
# Generate key pair (recommended: Ed25519)
ssh-keygen -t ed25519 -C "your_email@example.com"
# Generating public/private ed25519 key pair.
# Enter file in which to save the key (/home/user/.ssh/id_ed25519):
# Enter passphrase (empty for no passphrase):
# Your identification has been saved in /home/user/.ssh/id_ed25519
# Your public key has been saved in /home/user/.ssh/id_ed25519.pub

# Generate RSA key (older, widely compatible)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Generate with specific name
ssh-keygen -t ed25519 -f ~/.ssh/work_key -C "work@company.com"

# View your public key
cat ~/.ssh/id_ed25519.pub
# ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... your_email@example.com
```

### Key Types Comparison

| Type | Security | Speed | Compatibility |
|------|----------|-------|---------------|
| Ed25519 | Excellent | Fast | Modern systems |
| RSA 4096 | Excellent | Slower | Universal |
| ECDSA | Good | Fast | Most systems |
| DSA | Deprecated | - | Avoid |

### Copying Keys to Server

```bash
# Easiest method: ssh-copy-id
ssh-copy-id user@server
ssh-copy-id -i ~/.ssh/work_key.pub user@server

# Manual method
cat ~/.ssh/id_ed25519.pub | ssh user@server "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"

# Or copy and paste manually
cat ~/.ssh/id_ed25519.pub
# Then on server:
echo "ssh-ed25519 AAAAC3..." >> ~/.ssh/authorized_keys

# Verify key-based login works
ssh user@server
# Should connect without password prompt
```

### Managing Keys

```bash
# List keys in SSH agent
ssh-add -l

# Add key to agent
ssh-add ~/.ssh/id_ed25519
ssh-add ~/.ssh/work_key

# Add with timeout (12 hours)
ssh-add -t 43200 ~/.ssh/id_ed25519

# Remove key from agent
ssh-add -d ~/.ssh/id_ed25519

# Remove all keys
ssh-add -D

# Start SSH agent (if not running)
eval $(ssh-agent -s)
```

---

## SSH Configuration

### Client Configuration (~/.ssh/config)

```bash
# ~/.ssh/config

# Default settings for all hosts
Host *
    AddKeysToAgent yes
    IdentitiesOnly yes
    ServerAliveInterval 60
    ServerAliveCountMax 3

# Production server
Host prod
    HostName production.example.com
    User deploy
    Port 22
    IdentityFile ~/.ssh/production_key

# Staging server
Host staging
    HostName staging.example.com
    User deploy
    IdentityFile ~/.ssh/staging_key

# Development server
Host dev
    HostName 192.168.1.100
    User developer
    Port 2222

# Jump through bastion host
Host internal
    HostName 10.0.1.50
    User admin
    ProxyJump bastion

Host bastion
    HostName bastion.example.com
    User jump
    IdentityFile ~/.ssh/bastion_key

# GitHub
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_key

# GitLab with different key
Host gitlab.com
    HostName gitlab.com
    User git
    IdentityFile ~/.ssh/gitlab_key
```

### Using SSH Config

```bash
# Instead of:
ssh -i ~/.ssh/production_key -p 22 deploy@production.example.com

# Simply use:
ssh prod

# Works with scp too
scp file.txt prod:/path/

# And git
git clone git@github.com:user/repo.git
```

### Server Configuration (/etc/ssh/sshd_config)

```bash
# Important server settings
# Edit with: sudo nano /etc/ssh/sshd_config

# Change default port (security through obscurity)
Port 2222

# Disable root login
PermitRootLogin no

# Disable password authentication (after setting up keys!)
PasswordAuthentication no
PubkeyAuthentication yes

# Limit users
AllowUsers deploy admin
# or
AllowGroups ssh-users

# Limit authentication attempts
MaxAuthTries 3

# Disconnect idle sessions
ClientAliveInterval 300
ClientAliveCountMax 2

# After changes, restart SSH
sudo systemctl restart sshd

# Test BEFORE disconnecting!
# Open new terminal and verify you can connect
```

---

## File Transfer

### SCP - Secure Copy

```bash
# Copy local file to remote
scp file.txt user@server:/path/to/destination/
scp file.txt prod:/home/deploy/

# Copy remote file to local
scp user@server:/path/to/file.txt ./
scp prod:/var/log/app.log ./

# Copy directory (recursive)
scp -r ./project/ user@server:/path/

# Preserve permissions and times
scp -rp ./project/ user@server:/path/

# With specific port
scp -P 2222 file.txt user@server:/path/

# With bandwidth limit (KB/s)
scp -l 1000 large-file.zip user@server:/path/

# Between two remote servers
scp user1@server1:/path/file user2@server2:/path/
```

### SFTP - Secure FTP

```bash
# Start SFTP session
sftp user@server

# SFTP commands
sftp> pwd              # Remote working directory
sftp> lpwd             # Local working directory
sftp> ls               # List remote files
sftp> lls              # List local files
sftp> cd /path         # Change remote directory
sftp> lcd /local/path  # Change local directory
sftp> get file.txt     # Download file
sftp> put file.txt     # Upload file
sftp> get -r directory # Download directory
sftp> put -r directory # Upload directory
sftp> rm file.txt      # Delete remote file
sftp> mkdir newdir     # Create remote directory
sftp> exit             # Close session

# Non-interactive commands
sftp user@server:/path/file.txt ./
echo "get file.txt" | sftp user@server
```

### rsync - Efficient Sync

```bash
# Sync directory to remote
rsync -avz ./project/ user@server:/path/project/

# Options explained:
# -a  Archive mode (preserves permissions, timestamps, etc.)
# -v  Verbose
# -z  Compress during transfer

# Sync with delete (mirror)
rsync -avz --delete ./project/ user@server:/path/project/

# Exclude files
rsync -avz --exclude 'node_modules' --exclude '.git' ./project/ user@server:/path/

# Dry run (see what would happen)
rsync -avzn ./project/ user@server:/path/project/

# With progress
rsync -avz --progress large-file.zip user@server:/path/

# Resume interrupted transfer
rsync -avz --partial large-file.zip user@server:/path/

# Using SSH with specific port
rsync -avz -e "ssh -p 2222" ./project/ user@server:/path/
```

---

## SSH Tunneling

### Local Port Forwarding

```bash
# Access remote service through local port
# Syntax: ssh -L local_port:remote_host:remote_port user@server

# Access remote database (port 5432) via localhost:5432
ssh -L 5432:localhost:5432 user@server

# Access internal service through jump host
ssh -L 8080:internal-server:80 user@bastion

# Keep tunnel in background
ssh -fNL 5432:localhost:5432 user@server
# -f  Background
# -N  No command execution
# -L  Local forwarding

# Example: Access remote PostgreSQL
ssh -L 5432:localhost:5432 user@server
# Now connect locally: psql -h localhost -p 5432
```

### Remote Port Forwarding

```bash
# Make local service available on remote
# Syntax: ssh -R remote_port:local_host:local_port user@server

# Expose local dev server on remote port 8080
ssh -R 8080:localhost:3000 user@server
# Now server:8080 points to your localhost:3000

# Keep in background
ssh -fNR 8080:localhost:3000 user@server
```

### Dynamic Port Forwarding (SOCKS Proxy)

```bash
# Create SOCKS proxy
ssh -D 1080 user@server

# Background mode
ssh -fND 1080 user@server

# Configure browser/application to use:
# SOCKS Host: localhost
# SOCKS Port: 1080

# All traffic through proxy goes via the SSH server
```

### Jump Hosts / Bastion

```bash
# Jump through bastion to internal server
ssh -J user@bastion user@internal-server

# Multiple jumps
ssh -J user@jump1,user@jump2 user@final-server

# In config file (preferred)
Host internal
    HostName 10.0.1.50
    User admin
    ProxyJump bastion
```

---

## Security Best Practices

### Key Security

```bash
# Correct permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/config

# Use passphrase for keys
ssh-keygen -t ed25519 -C "email@example.com"
# Enter passphrase: ********

# Change key passphrase
ssh-keygen -p -f ~/.ssh/id_ed25519
```

### Server Hardening

```bash
# /etc/ssh/sshd_config recommended settings

# Disable password auth
PasswordAuthentication no
ChallengeResponseAuthentication no

# Disable root login
PermitRootLogin no

# Use key authentication only
PubkeyAuthentication yes

# Limit users
AllowUsers deploy admin

# Use strong ciphers only
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group16-sha512

# Limit login attempts
MaxAuthTries 3
MaxSessions 3

# Disable unused features
X11Forwarding no
AllowAgentForwarding no
AllowTcpForwarding no
PermitTunnel no
```

### Monitoring and Logging

```bash
# View SSH logs
sudo tail -f /var/log/auth.log     # Debian/Ubuntu
sudo tail -f /var/log/secure       # CentOS/RHEL

# View failed attempts
sudo grep "Failed password" /var/log/auth.log
sudo grep "Invalid user" /var/log/auth.log

# View successful logins
last
who
w
```

---

## Troubleshooting

### Common Issues

```bash
# Permission denied (publickey)
# 1. Check key is added to agent
ssh-add -l

# 2. Check permissions
ls -la ~/.ssh/

# 3. Check authorized_keys on server
cat ~/.ssh/authorized_keys

# 4. Verbose mode for details
ssh -vvv user@server

# Connection refused
# 1. Check SSH service is running
sudo systemctl status sshd

# 2. Check port is open
sudo netstat -tuln | grep 22

# 3. Check firewall
sudo ufw status
sudo iptables -L

# Host key verification failed
# Server key changed (reinstall, different server)
ssh-keygen -R hostname

# Too many authentication failures
# Specify exact key
ssh -i ~/.ssh/specific_key user@server

# Or in config
Host server
    IdentityFile ~/.ssh/specific_key
    IdentitiesOnly yes
```

### Debugging

```bash
# Client-side debug
ssh -v user@server    # Verbose
ssh -vv user@server   # More verbose
ssh -vvv user@server  # Maximum verbosity

# Server-side debug (temporary)
sudo /usr/sbin/sshd -d

# Test configuration
sudo sshd -t
```

---

## Practice Exercises

### Exercise 1: Key Setup

```bash
# 1. Generate Ed25519 key pair
# 2. Copy public key to a server (or GitHub)
# 3. Test key-based login
# 4. Disable password authentication
```

### Exercise 2: SSH Config

```bash
# 1. Create ~/.ssh/config
# 2. Add entries for multiple servers
# 3. Test connection using short names
```

### Exercise 3: File Transfer

```bash
# 1. Copy a file to remote server with scp
# 2. Sync a directory with rsync
# 3. Use SFTP to browse and download files
```

### Exercise 4: Tunneling

```bash
# 1. Create local tunnel to access remote database
# 2. Set up SOCKS proxy for secure browsing
```

---

## Quick Reference

### Common Commands

| Command | Description |
|---------|-------------|
| `ssh user@host` | Connect to server |
| `ssh-keygen -t ed25519` | Generate key pair |
| `ssh-copy-id user@host` | Copy public key |
| `ssh-add key` | Add key to agent |
| `scp file user@host:/path` | Copy file to remote |
| `rsync -avz src/ user@host:/dst/` | Sync directory |
| `ssh -L port:host:port user@host` | Local tunnel |

### Config Template

```bash
Host alias
    HostName server.example.com
    User username
    Port 22
    IdentityFile ~/.ssh/keyname
```

---

## Key Takeaways

1. **Always use key authentication** - Disable passwords
2. **Ed25519 keys** are preferred over RSA
3. **SSH config** saves time and reduces errors
4. **Correct permissions** are critical (700, 600)
5. **Tunneling** provides secure access to internal services
6. **rsync** is better than scp for large/repeated transfers

---

## What's Next?

Tomorrow, we'll learn about **Shell Scripting** - automating tasks with Bash scripts to increase productivity and create reusable automation.
