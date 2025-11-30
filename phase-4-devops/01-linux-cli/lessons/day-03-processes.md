# Day 3: Process Management - Controlling Running Programs

## Introduction

Every program running on Linux is a process. Understanding how to view, manage, and control processes is essential for server administration, debugging applications, and optimizing system performance. Today, you'll learn how to monitor and manage processes effectively.

## Learning Objectives

By the end of this lesson, you will be able to:
- View and understand process information
- Monitor system resources in real-time
- Start, stop, and manage processes
- Work with background and foreground processes
- Use signals to control process behavior

---

## Understanding Processes

### What is a Process?

```
┌─────────────────────────────────────────────────────────┐
│                      Process                             │
├─────────────────────────────────────────────────────────┤
│  PID: 1234              │  Unique identifier             │
│  PPID: 1                │  Parent process ID             │
│  User: deploy           │  Owner of the process          │
│  State: Running         │  Current status                │
│  CPU: 2.5%              │  CPU usage                     │
│  Memory: 128MB          │  RAM usage                     │
│  Command: node app.js   │  What's running                │
└─────────────────────────────────────────────────────────┘
```

### Process States

| State | Symbol | Description |
|-------|--------|-------------|
| Running | R | Currently executing |
| Sleeping | S | Waiting for event |
| Stopped | T | Suspended (Ctrl+Z) |
| Zombie | Z | Terminated but not cleaned up |
| Disk Sleep | D | Waiting for I/O |

---

## Viewing Processes

### ps - Process Status

```bash
# Basic: Current terminal's processes
ps
#   PID TTY          TIME CMD
#  1234 pts/0    00:00:00 bash
#  5678 pts/0    00:00:00 ps

# All processes (BSD style)
ps aux
# USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
# root         1  0.0  0.1  16940  4520 ?        Ss   Jan15   0:03 /sbin/init
# deploy    1234  2.5  1.2 245632 50124 ?        Sl   10:30   1:25 node app.js

# All processes (Unix style)
ps -ef
# UID        PID  PPID  C STIME TTY          TIME CMD
# root         1     0  0 Jan15 ?        00:00:03 /sbin/init

# Process tree (show hierarchy)
ps auxf
ps -ejH

# Specific user's processes
ps -u deploy

# Specific process by name
ps aux | grep node
ps -C node

# Show threads
ps -eLf

# Custom format
ps -eo pid,ppid,user,%cpu,%mem,stat,cmd
ps -eo pid,ppid,user,%cpu,%mem,cmd --sort=-%cpu | head -10
```

### Understanding ps aux Output

```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
deploy    1234  2.5  1.2 245632 50124 ?        Ssl  10:30   1:25 node app.js
│         │    │    │    │      │     │        │││  │       │    │
│         │    │    │    │      │     │        │││  │       │    └─ Command
│         │    │    │    │      │     │        │││  │       └────── CPU time used
│         │    │    │    │      │     │        │││  └────────────── Start time
│         │    │    │    │      │     │        ││└────────────────── Multi-threaded
│         │    │    │    │      │     │        │└─────────────────── Session leader
│         │    │    │    │      │     │        └──────────────────── Sleeping
│         │    │    │    │      │     └───────────────────────────── Terminal (? = none)
│         │    │    │    │      └─────────────────────────────────── Resident memory (KB)
│         │    │    │    └────────────────────────────────────────── Virtual memory (KB)
│         │    │    └─────────────────────────────────────────────── Memory percentage
│         │    └──────────────────────────────────────────────────── CPU percentage
│         └───────────────────────────────────────────────────────── Process ID
└─────────────────────────────────────────────────────────────────── User
```

### pgrep - Find Processes

```bash
# Find PID by name
pgrep node
pgrep -l node    # With process name
pgrep -a node    # With full command

# Find by user
pgrep -u deploy

# Find by exact name
pgrep -x node

# Newest/oldest process
pgrep -n node    # Newest
pgrep -o node    # Oldest
```

---

## Real-Time Monitoring

### top - Interactive Process Viewer

```bash
# Start top
top

# Keyboard shortcuts in top:
# h         - Help
# q         - Quit
# M         - Sort by memory
# P         - Sort by CPU
# T         - Sort by time
# k         - Kill process (enter PID)
# r         - Renice process
# u         - Filter by user
# 1         - Show individual CPUs
# c         - Show full command
# Space     - Refresh

# Start with specific options
top -u deploy      # Specific user
top -p 1234        # Specific PID
top -d 1           # Update every 1 second
top -n 5           # Run 5 iterations then exit
```

### htop - Enhanced Process Viewer

```bash
# Install htop (if not available)
sudo apt install htop    # Debian/Ubuntu
sudo yum install htop    # CentOS/RHEL

# Run htop
htop

# Features:
# - Color-coded output
# - Mouse support
# - Horizontal/vertical scrolling
# - Tree view (F5)
# - Search (F3)
# - Filter (F4)
# - Kill process (F9)
# - Nice value (F7/F8)
```

### Other Monitoring Tools

```bash
# Memory usage
free -h
#               total        used        free      shared  buff/cache   available
# Mem:           16Gi       4.2Gi       8.1Gi       256Mi       3.7Gi       11Gi
# Swap:          2.0Gi          0B       2.0Gi

# Disk I/O
iostat
iotop     # Requires root

# Network connections
netstat -tuln
ss -tuln

# System summary
vmstat 1    # Every 1 second
```

---

## Managing Processes

### Starting Processes

```bash
# Run in foreground (blocks terminal)
node app.js

# Run in background (append &)
node app.js &
# [1] 1234    <- Job number and PID

# Run and detach from terminal
nohup node app.js &
nohup node app.js > output.log 2>&1 &

# Disown a running process
node app.js &
disown
```

### Foreground and Background

```bash
# Start process
node app.js

# Suspend to background (Ctrl+Z)
^Z
# [1]+  Stopped                 node app.js

# View background jobs
jobs
# [1]+  Stopped                 node app.js
# [2]-  Running                 npm run watch &

# Resume in background
bg %1

# Bring to foreground
fg %1

# Kill background job
kill %1
```

### Stopping Processes

```bash
# Kill by PID
kill 1234

# Kill by name
pkill node
killall node

# Force kill (SIGKILL)
kill -9 1234
kill -KILL 1234

# Kill all user's processes
pkill -u username

# Kill process group
kill -9 -1234    # Negative PID = process group
```

---

## Signals

### Common Signals

| Signal | Number | Description |
|--------|--------|-------------|
| SIGHUP | 1 | Hangup (reload config) |
| SIGINT | 2 | Interrupt (Ctrl+C) |
| SIGQUIT | 3 | Quit (Ctrl+\) |
| SIGKILL | 9 | Force kill (cannot be caught) |
| SIGTERM | 15 | Graceful termination (default) |
| SIGSTOP | 19 | Stop process (Ctrl+Z) |
| SIGCONT | 18 | Continue stopped process |

### Sending Signals

```bash
# List all signals
kill -l

# Send SIGTERM (graceful, default)
kill 1234
kill -15 1234
kill -TERM 1234

# Send SIGKILL (force, last resort)
kill -9 1234
kill -KILL 1234

# Send SIGHUP (reload configuration)
kill -1 1234
kill -HUP 1234

# Send signal by name
pkill -TERM node
killall -HUP nginx
```

### Graceful vs Force Kill

```bash
# Always try graceful first
kill 1234

# Wait a few seconds
sleep 5

# Check if still running
ps -p 1234

# Force kill only if necessary
kill -9 1234

# One-liner pattern
kill 1234 && sleep 5 && kill -9 1234 2>/dev/null
```

---

## Process Priority

### Nice Values

```
Priority Range: -20 (highest) to 19 (lowest)
Default: 0

Lower nice = Higher priority = More CPU time
Higher nice = Lower priority = Less CPU time
```

### Setting Priority

```bash
# Start with specific priority
nice -n 10 ./heavy-task.sh     # Low priority
nice -n -10 ./important.sh     # High priority (needs root)

# Change priority of running process
renice 10 -p 1234              # Set to 10
renice -5 -p 1234              # Set to -5 (needs root)
renice 15 -u deploy            # All user's processes

# View nice values
ps -eo pid,ni,cmd
top   # NI column
```

---

## Process Information

### /proc Filesystem

```bash
# Process directory
ls /proc/1234/

# Process status
cat /proc/1234/status
# Name:   node
# State:  S (sleeping)
# Pid:    1234
# PPid:   1
# Threads: 8

# Command line
cat /proc/1234/cmdline

# Environment variables
cat /proc/1234/environ | tr '\0' '\n'

# File descriptors
ls -l /proc/1234/fd/

# Memory maps
cat /proc/1234/maps

# Current working directory
ls -l /proc/1234/cwd
```

### Process Limits

```bash
# View limits
cat /proc/1234/limits
ulimit -a

# Set limits in shell
ulimit -n 65535    # Max open files
ulimit -u 4096     # Max processes

# Permanent limits (/etc/security/limits.conf)
deploy  soft  nofile  65535
deploy  hard  nofile  65535
deploy  soft  nproc   4096
deploy  hard  nproc   4096
```

---

## Daemon Processes

### Understanding Daemons

```bash
# Daemons are background services
# Usually started at boot
# Run without terminal

# Common daemons
systemctl list-units --type=service

# View daemon status
systemctl status nginx
systemctl status postgresql
```

### Managing Services (systemd)

```bash
# Start service
sudo systemctl start nginx

# Stop service
sudo systemctl stop nginx

# Restart service
sudo systemctl restart nginx

# Reload configuration
sudo systemctl reload nginx

# Enable at boot
sudo systemctl enable nginx

# Disable at boot
sudo systemctl disable nginx

# Check status
systemctl status nginx
systemctl is-active nginx
systemctl is-enabled nginx

# View logs
journalctl -u nginx
journalctl -u nginx -f        # Follow
journalctl -u nginx --since today
```

---

## Practical Examples

### Find Resource-Heavy Processes

```bash
# Top CPU consumers
ps aux --sort=-%cpu | head -10

# Top memory consumers
ps aux --sort=-%mem | head -10

# Using top (one-shot)
top -bn1 | head -20
```

### Monitor Application

```bash
# Watch Node.js app
watch -n 1 "ps aux | grep node"

# Monitor with continuous output
while true; do
  ps -p 1234 -o %cpu,%mem,cmd
  sleep 1
done
```

### Kill Stuck Processes

```bash
# Find zombie processes
ps aux | awk '$8 == "Z"'

# Kill all zombie processes (kill parent)
ps -eo ppid,stat | awk '$2 == "Z" {print $1}' | xargs kill -9

# Kill process by port
fuser -k 3000/tcp
lsof -ti:3000 | xargs kill -9
```

### Clean Up

```bash
# Kill all processes by name
pkill -f "node.*app.js"

# Kill old processes
pkill -9 -u deploy --older 1h

# Kill processes using too much memory
ps aux | awk '$4 > 50 {print $2}' | xargs kill
```

---

## Practice Exercises

### Exercise 1: Process Exploration

```bash
# 1. List all processes and count them
# 2. Find all processes owned by root
# 3. Find the process using the most CPU
# 4. Find the process using the most memory
```

### Exercise 2: Process Control

```bash
# 1. Start a long-running process (e.g., sleep 300)
# 2. Send it to the background
# 3. List all jobs
# 4. Bring it to foreground
# 5. Kill it gracefully
```

### Exercise 3: Monitoring

```bash
# 1. Use top to find processes using > 10% CPU
# 2. Create a script that logs CPU/memory usage every 5 seconds
# 3. Find which process is listening on port 80
```

---

## Quick Reference

### Process Commands

| Command | Description |
|---------|-------------|
| `ps aux` | List all processes |
| `top` / `htop` | Interactive process viewer |
| `pgrep name` | Find PID by name |
| `kill PID` | Terminate process |
| `kill -9 PID` | Force kill |
| `pkill name` | Kill by name |
| `jobs` | List background jobs |
| `bg` / `fg` | Background/foreground |
| `nohup cmd &` | Run immune to hangups |
| `nice -n N` | Start with priority |
| `renice N -p PID` | Change priority |

### Signal Numbers

| Signal | Number | Meaning |
|--------|--------|---------|
| HUP | 1 | Reload |
| INT | 2 | Ctrl+C |
| KILL | 9 | Force kill |
| TERM | 15 | Graceful stop |
| STOP | 19 | Pause |
| CONT | 18 | Resume |

---

## Key Takeaways

1. **Every program is a process** with unique PID
2. **ps and top** are essential monitoring tools
3. **SIGTERM before SIGKILL** - always try graceful first
4. **Background processes** with & or nohup
5. **Nice values** control CPU priority
6. **systemd** manages services on modern Linux

---

## What's Next?

Tomorrow, we'll learn about **SSH** - secure remote access to Linux servers, including key authentication and configuration.
