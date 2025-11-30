# Day 2: File Permissions - Controlling Access in Linux

## Introduction

Linux is a multi-user operating system where permissions control who can read, write, or execute files. Understanding permissions is crucial for server security, deployment, and troubleshooting access issues. Today, you'll learn how to read, modify, and troubleshoot file permissions.

## Learning Objectives

By the end of this lesson, you will be able to:
- Read and interpret file permissions
- Modify permissions using chmod
- Change file ownership with chown and chgrp
- Understand special permissions (setuid, setgid, sticky bit)
- Apply appropriate permissions for common scenarios

---

## Understanding Permission Basics

### Permission Structure

```
-rwxr-xr-x  1  user  group  4096  Jan 15 10:30  script.sh
│└─┬─┘└─┬─┘└─┬─┘
│  │    │    │
│  │    │    └── Others (everyone else)
│  │    └─────── Group (members of file's group)
│  └──────────── Owner (file's owner)
└─────────────── File type (- = file, d = directory, l = link)
```

### Permission Types

| Symbol | Meaning | For Files | For Directories |
|--------|---------|-----------|-----------------|
| `r` | Read | View contents | List contents (ls) |
| `w` | Write | Modify contents | Create/delete files |
| `x` | Execute | Run as program | Enter directory (cd) |
| `-` | No permission | Denied | Denied |

### Reading Permissions

```bash
# List with permissions
ls -l

# Example output
drwxr-xr-x  5 user group  4096 Jan 15 10:30 projects
-rw-r--r--  1 user group  1234 Jan 15 10:25 readme.txt
-rwx------  1 user group  5678 Jan 15 10:20 secret.sh
lrwxrwxrwx  1 user group    11 Jan 15 10:15 link -> target.txt

# Breaking down: drwxr-xr-x
# d     = directory
# rwx   = owner can read, write, execute
# r-x   = group can read and execute (not write)
# r-x   = others can read and execute (not write)
```

---

## Numeric (Octal) Permissions

### Permission Values

```
┌───────────────────────────────────────────┐
│        Permission Number Values           │
├─────────────┬─────────────┬──────────────┤
│   Read (r)  │  Write (w)  │ Execute (x)  │
│      4      │      2      │      1       │
└─────────────┴─────────────┴──────────────┘

Combined values:
7 = rwx (4+2+1)
6 = rw- (4+2)
5 = r-x (4+1)
4 = r-- (4)
3 = -wx (2+1)
2 = -w- (2)
1 = --x (1)
0 = --- (0)
```

### Common Permission Sets

```bash
# 755 - Standard for directories and executables
# Owner: rwx (7), Group: r-x (5), Others: r-x (5)
chmod 755 script.sh

# 644 - Standard for regular files
# Owner: rw- (6), Group: r-- (4), Others: r-- (4)
chmod 644 document.txt

# 600 - Private files (only owner)
# Owner: rw- (6), Group: --- (0), Others: --- (0)
chmod 600 private.key

# 700 - Private executable/directory
chmod 700 secret-folder/

# 777 - Full access (avoid in production!)
chmod 777 public.txt
```

---

## Changing Permissions with chmod

### Symbolic Mode

```bash
# Syntax: chmod [who][operator][permission] file
# Who: u (user/owner), g (group), o (others), a (all)
# Operator: + (add), - (remove), = (set exactly)

# Add execute for owner
chmod u+x script.sh

# Remove write for group and others
chmod go-w file.txt

# Set exact permissions for group
chmod g=rx file.txt

# Add read for all
chmod a+r document.txt

# Multiple changes at once
chmod u+x,g-w,o-rwx file.txt

# Remove all permissions for others
chmod o= file.txt
```

### Numeric Mode

```bash
# Set specific permissions
chmod 755 script.sh       # rwxr-xr-x
chmod 644 config.txt      # rw-r--r--
chmod 600 credentials.env # rw-------
chmod 700 private-dir/    # rwx------

# Common patterns
chmod 755 *.sh            # All shell scripts
chmod 644 *.html          # All HTML files
```

### Recursive Permissions

```bash
# Change permissions recursively
chmod -R 755 directory/

# Different permissions for files vs directories
# Files: 644, Directories: 755
find /path -type f -exec chmod 644 {} \;
find /path -type d -exec chmod 755 {} \;

# Using chmod with find (more efficient)
find /path -type f -exec chmod 644 {} +
find /path -type d -exec chmod 755 {} +
```

---

## Changing Ownership

### chown - Change Owner

```bash
# Change owner
chown newowner file.txt

# Change owner and group
chown newowner:newgroup file.txt

# Change only group
chown :newgroup file.txt

# Recursive
chown -R user:group directory/

# Common scenarios
sudo chown www-data:www-data /var/www/html/
sudo chown -R deploy:deploy /app/
```

### chgrp - Change Group

```bash
# Change group only
chgrp developers file.txt

# Recursive
chgrp -R developers project/

# Verify changes
ls -l file.txt
```

---

## Special Permissions

### Setuid (Set User ID)

```bash
# When executed, runs as file owner (not executing user)
# Represented as 's' in owner execute position

# View setuid files
ls -l /usr/bin/passwd
# -rwsr-xr-x 1 root root ... /usr/bin/passwd

# Set setuid (use with caution!)
chmod u+s executable
chmod 4755 executable

# Remove setuid
chmod u-s executable
```

### Setgid (Set Group ID)

```bash
# Files: Run as file's group
# Directories: New files inherit directory's group

# Set setgid on directory
chmod g+s shared-folder/
chmod 2755 shared-folder/

# New files in shared-folder inherit its group
mkdir shared-folder
chgrp developers shared-folder
chmod g+s shared-folder
# Now all new files belong to 'developers' group
```

### Sticky Bit

```bash
# Only owner can delete their files in directory
# Common on /tmp

# View sticky bit
ls -ld /tmp
# drwxrwxrwt 15 root root ... /tmp

# Set sticky bit
chmod +t shared-directory/
chmod 1777 shared-directory/

# Remove sticky bit
chmod -t shared-directory/
```

### Special Permission Numbers

```
4000 = setuid
2000 = setgid
1000 = sticky bit

# Examples
chmod 4755 file  # setuid + rwxr-xr-x
chmod 2755 dir   # setgid + rwxr-xr-x
chmod 1777 dir   # sticky + rwxrwxrwx
```

---

## Default Permissions (umask)

### Understanding umask

```bash
# umask determines default permissions for new files/directories
# Subtracts from maximum permissions (777 for dirs, 666 for files)

# View current umask
umask
# 0022

# Default directory: 777 - 022 = 755 (rwxr-xr-x)
# Default file: 666 - 022 = 644 (rw-r--r--)
```

### Setting umask

```bash
# More restrictive (recommended)
umask 027
# Directories: 750 (rwxr-x---)
# Files: 640 (rw-r-----)

# Most restrictive
umask 077
# Directories: 700 (rwx------)
# Files: 600 (rw-------)

# Set permanently in ~/.bashrc
echo "umask 027" >> ~/.bashrc
```

---

## Common Permission Scenarios

### Web Server Files

```bash
# Web root ownership
sudo chown -R www-data:www-data /var/www/html/

# Directories: 755, Files: 644
sudo find /var/www/html -type d -exec chmod 755 {} \;
sudo find /var/www/html -type f -exec chmod 644 {} \;

# Upload directory (writable)
sudo chmod 775 /var/www/html/uploads/
```

### SSH Keys

```bash
# SSH directory
chmod 700 ~/.ssh/

# Private keys (very important!)
chmod 600 ~/.ssh/id_rsa
chmod 600 ~/.ssh/id_ed25519

# Public keys
chmod 644 ~/.ssh/id_rsa.pub

# authorized_keys
chmod 600 ~/.ssh/authorized_keys

# SSH config
chmod 600 ~/.ssh/config
```

### Application Deployment

```bash
# Application directory
sudo chown -R deploy:deploy /app/
sudo chmod 755 /app/

# Environment files (sensitive)
chmod 600 /app/.env

# Log directory
sudo chown deploy:deploy /app/logs/
chmod 755 /app/logs/

# Executable scripts
chmod 755 /app/scripts/*.sh
```

### Shared Project Directory

```bash
# Create shared directory
sudo mkdir /projects/shared
sudo chgrp developers /projects/shared
sudo chmod 2775 /projects/shared
# - 2 = setgid (files inherit group)
# - 775 = rwxrwxr-x

# All developers can now collaborate
# New files belong to 'developers' group
```

---

## Troubleshooting Permissions

### Common Permission Errors

```bash
# Permission denied
$ cat secret.txt
cat: secret.txt: Permission denied

# Solution: Check permissions
ls -l secret.txt
# -rw------- 1 root root ... secret.txt

# Fix: Add read permission or use sudo
sudo cat secret.txt
# or
sudo chmod a+r secret.txt
```

### Debugging Permission Issues

```bash
# Check file permissions
ls -la file.txt

# Check directory permissions (need x to enter)
ls -ld /path/to/directory

# Check effective permissions
namei -l /path/to/file

# Check your groups
groups
id

# Check if user can access
sudo -u username test -r /path/to/file && echo "Can read"
```

### Common Mistakes

```bash
# Mistake 1: Directory without execute
chmod 644 directory/
cd directory/  # Permission denied!
# Fix:
chmod 755 directory/

# Mistake 2: Script without execute
./script.sh  # Permission denied
# Fix:
chmod +x script.sh

# Mistake 3: Using 777 in production
chmod 777 /var/www/  # Security risk!
# Fix:
chmod 755 /var/www/
chown -R www-data:www-data /var/www/
```

---

## Access Control Lists (ACLs)

### Extended Permissions

```bash
# View ACLs
getfacl file.txt

# Set ACL for specific user
setfacl -m u:username:rwx file.txt

# Set ACL for specific group
setfacl -m g:groupname:rx file.txt

# Remove ACL
setfacl -x u:username file.txt

# Remove all ACLs
setfacl -b file.txt

# Default ACLs for directory (inherited by new files)
setfacl -d -m u:username:rwx directory/
```

---

## Practice Exercises

### Exercise 1: Basic Permissions

```bash
# 1. Create a file and check its permissions
# 2. Make it executable only by owner
# 3. Make it readable by everyone
# 4. Remove all permissions for others
```

### Exercise 2: Shared Directory

```bash
# 1. Create a directory /tmp/shared
# 2. Set group to 'users'
# 3. Enable setgid so new files inherit the group
# 4. Enable sticky bit so only owners can delete
```

### Exercise 3: Web Application

```bash
# 1. Create /var/www/myapp/
# 2. Set ownership to www-data:www-data
# 3. Set directories to 755, files to 644
# 4. Create an uploads folder with write access
```

---

## Quick Reference

### Permission Commands

| Command | Description |
|---------|-------------|
| `chmod 755 file` | Set permissions (numeric) |
| `chmod u+x file` | Add permission (symbolic) |
| `chown user file` | Change owner |
| `chown user:group file` | Change owner and group |
| `chgrp group file` | Change group |
| `umask 022` | Set default permissions |

### Common Permission Sets

| Numeric | Symbolic | Use Case |
|---------|----------|----------|
| 755 | rwxr-xr-x | Directories, executables |
| 644 | rw-r--r-- | Regular files |
| 600 | rw------- | Private files |
| 700 | rwx------ | Private directories |
| 775 | rwxrwxr-x | Shared directories |

---

## Key Takeaways

1. **Three permission types** - Read (r), Write (w), Execute (x)
2. **Three permission levels** - Owner, Group, Others
3. **Numeric is faster** - 755, 644, 600 are most common
4. **Directories need x** - To enter/list contents
5. **Least privilege principle** - Give minimum necessary permissions
6. **Avoid 777** - Never use in production

---

## What's Next?

Tomorrow, we'll learn about **Process Management** - understanding how to view, control, and manage running processes on Linux systems.
