# Linux & CLI

**Duration:** 1 week

## Learning Objectives

By the end of this section, you will:
- Navigate the Linux filesystem confidently
- Manage files, permissions, and processes
- Connect to remote servers via SSH
- Write basic shell scripts

---

## Day 1: Filesystem Navigation

### Essential Commands
```bash
# Navigation
pwd                 # Print working directory
ls                  # List files
ls -la              # List all files with details
cd /path/to/dir     # Change directory
cd ..               # Go up one level
cd ~                # Go to home directory
cd -                # Go to previous directory

# File operations
touch file.txt      # Create empty file
mkdir dirname       # Create directory
mkdir -p a/b/c      # Create nested directories
cp source dest      # Copy file
cp -r src dest      # Copy directory recursively
mv source dest      # Move/rename
rm file.txt         # Remove file
rm -rf dirname      # Remove directory (careful!)

# Viewing files
cat file.txt        # Display entire file
less file.txt       # Paginated view
head -n 20 file     # First 20 lines
tail -n 20 file     # Last 20 lines
tail -f file        # Follow file (logs)
```

### Exercises
1. Navigate to your home directory
2. Create a project folder structure
3. Create, copy, move, and delete files
4. View contents of system files (/etc/hosts)

---

## Day 2: File Permissions

### Understanding Permissions
```bash
# ls -la output:
# -rw-r--r--  1 user group  1234 Jan 1 12:00 file.txt
# drwxr-xr-x  2 user group  4096 Jan 1 12:00 dirname

# Permission format: type|owner|group|others
# r = read (4)
# w = write (2)
# x = execute (1)

# Changing permissions
chmod 755 file      # rwxr-xr-x
chmod 644 file      # rw-r--r--
chmod +x script.sh  # Add execute permission
chmod -w file       # Remove write permission

# Changing ownership
chown user:group file
chown -R user:group directory

# Common permission sets
chmod 700 private_script.sh    # Owner only
chmod 755 public_script.sh     # Owner full, others read/execute
chmod 644 config.txt           # Owner read/write, others read
chmod 600 secrets.env          # Owner only, sensitive files
```

### Exercises
1. Create a script and make it executable
2. Create a private directory only you can access
3. Understand permission denied errors

---

## Day 3: Process Management

### Process Commands
```bash
# View processes
ps aux              # All processes
ps aux | grep node  # Filter by name
top                 # Interactive process viewer
htop                # Better interactive viewer

# Process control
./script.sh &       # Run in background
nohup ./script.sh & # Run and persist after logout
jobs                # List background jobs
fg %1               # Bring job to foreground
bg %1               # Send job to background

# Killing processes
kill PID            # Graceful terminate
kill -9 PID         # Force kill
killall node        # Kill by name
pkill -f "pattern"  # Kill by pattern

# System resources
free -h             # Memory usage
df -h               # Disk usage
du -sh dirname      # Directory size
```

### Exercises
1. Run a Node.js server in the background
2. Monitor its resource usage
3. Stop it gracefully

---

## Day 4: SSH & Remote Access

### SSH Basics
```bash
# Connect to remote server
ssh user@hostname
ssh -p 2222 user@hostname    # Custom port
ssh -i key.pem user@hostname # With key file

# SSH key management
ssh-keygen -t ed25519 -C "your@email.com"
# Keys stored in ~/.ssh/id_ed25519 and ~/.ssh/id_ed25519.pub

# Copy key to server
ssh-copy-id user@hostname

# SSH config (~/.ssh/config)
Host myserver
    HostName 192.168.1.100
    User deploy
    Port 22
    IdentityFile ~/.ssh/id_ed25519

# Now connect with just:
ssh myserver

# File transfer
scp file.txt user@host:/path/     # Copy to remote
scp user@host:/path/file.txt .    # Copy from remote
scp -r dirname user@host:/path/   # Copy directory

# rsync (better for large transfers)
rsync -avz --progress source/ user@host:/dest/
```

### Exercises
1. Generate an SSH key pair
2. Connect to a server (use a VM or cloud instance)
3. Transfer files to/from the server

---

## Day 5: Shell Scripting

### Basic Script Structure
```bash
#!/bin/bash
# This is a comment

# Variables
NAME="World"
echo "Hello, $NAME!"

# User input
read -p "Enter your name: " USERNAME
echo "Hello, $USERNAME!"

# Conditionals
if [ "$USERNAME" == "admin" ]; then
    echo "Welcome, admin!"
elif [ -z "$USERNAME" ]; then
    echo "No name provided"
else
    echo "Hello, $USERNAME!"
fi

# Loops
for i in 1 2 3 4 5; do
    echo "Number: $i"
done

for file in *.txt; do
    echo "Processing $file"
done

# Functions
greet() {
    local name=$1
    echo "Hello, $name!"
}

greet "Developer"
```

### Useful Scripts

**Deployment Script:**
```bash
#!/bin/bash
set -e  # Exit on error

echo "Starting deployment..."

# Pull latest code
git pull origin main

# Install dependencies
npm ci

# Build
npm run build

# Restart server
pm2 restart app

echo "Deployment complete!"
```

**Backup Script:**
```bash
#!/bin/bash

BACKUP_DIR="/backups"
DATE=$(date +%Y-%m-%d_%H-%M-%S)
DB_NAME="myapp"

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup database
pg_dump $DB_NAME > "$BACKUP_DIR/db_$DATE.sql"

# Backup uploads
tar -czf "$BACKUP_DIR/uploads_$DATE.tar.gz" /app/uploads

# Keep only last 7 days
find $BACKUP_DIR -mtime +7 -delete

echo "Backup completed: $DATE"
```

### Exercises
1. Write a script to set up a new project
2. Write a deployment script for your app
3. Create a backup script

---

## Quick Reference

### Common Operations
```bash
# Finding files
find /path -name "*.js"           # Find by name
find /path -type f -mtime -7      # Modified in last 7 days
which node                        # Find executable location

# Text processing
grep "pattern" file               # Search in file
grep -r "pattern" dir/            # Recursive search
grep -i "pattern" file            # Case insensitive
sed 's/old/new/g' file            # Replace text
awk '{print $1}' file             # Print first column

# Compression
tar -czf archive.tar.gz dirname   # Create archive
tar -xzf archive.tar.gz           # Extract archive
zip -r archive.zip dirname        # Create zip
unzip archive.zip                 # Extract zip

# Network
curl https://api.example.com      # HTTP request
wget https://example.com/file     # Download file
netstat -tulpn                    # Show ports in use
lsof -i :3000                     # What's using port 3000

# System info
uname -a                          # System information
hostname                          # Server hostname
whoami                            # Current user
env                               # Environment variables
```

---

## Resources

- [Linux Command Line Basics](https://ubuntu.com/tutorials/command-line-for-beginners)
- [Bash Scripting Tutorial](https://linuxconfig.org/bash-scripting-tutorial)
- [explainshell.com](https://explainshell.com/) - Explains commands

---

## Checklist Before Moving On

- [ ] Navigate filesystem confidently
- [ ] Understand file permissions
- [ ] Can manage processes
- [ ] Can connect via SSH
- [ ] Can write basic shell scripts

---

**Next:** [Docker](../02-docker/README.md)
