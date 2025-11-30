# Day 5: Shell Scripting - Automating Tasks with Bash

## Introduction

Shell scripting allows you to automate repetitive tasks, create deployment pipelines, and build powerful tools. A well-written script can save hours of manual work and reduce human error. Today, you'll learn the fundamentals of Bash scripting to boost your productivity as a developer.

## Learning Objectives

By the end of this lesson, you will be able to:
- Write and execute Bash scripts
- Use variables, conditionals, and loops
- Handle command-line arguments
- Create functions and modular scripts
- Build practical automation scripts

---

## Getting Started

### Your First Script

```bash
#!/bin/bash
# This is a comment
# hello.sh - My first script

echo "Hello, World!"
echo "Today is $(date)"
echo "You are logged in as: $USER"
```

### Making Scripts Executable

```bash
# Create script
nano hello.sh

# Make executable
chmod +x hello.sh

# Run script
./hello.sh

# Or run with bash directly
bash hello.sh
```

### The Shebang Line

```bash
#!/bin/bash          # Use Bash
#!/bin/sh            # Use POSIX shell
#!/usr/bin/env bash  # Find bash in PATH (portable)
#!/usr/bin/env python3  # For Python scripts

# The shebang must be the first line!
```

---

## Variables

### Declaring Variables

```bash
#!/bin/bash

# Simple variables (no spaces around =)
name="John"
age=25
is_active=true

# Using variables
echo "Name: $name"
echo "Age: $age"
echo "Is active: $is_active"

# Curly braces for clarity
echo "Hello, ${name}!"
echo "${name}s files"  # Johns files

# Command substitution
current_date=$(date)
file_count=$(ls | wc -l)
echo "Today: $current_date"
echo "Files in directory: $file_count"

# Arithmetic
result=$((5 + 3))
count=$((count + 1))
echo "Result: $result"
```

### Special Variables

```bash
#!/bin/bash

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "All arguments: $@"
echo "Number of arguments: $#"
echo "Exit status of last command: $?"
echo "Process ID: $$"
echo "Last background PID: $!"
```

### Environment Variables

```bash
#!/bin/bash

# Access environment variables
echo "Home: $HOME"
echo "User: $USER"
echo "Path: $PATH"
echo "Shell: $SHELL"
echo "Current directory: $PWD"

# Set environment variable (for child processes)
export MY_VAR="value"

# Set for current script only
MY_LOCAL="local value"
```

### Reading User Input

```bash
#!/bin/bash

# Simple input
echo "What is your name?"
read name
echo "Hello, $name!"

# Input with prompt
read -p "Enter your age: " age
echo "You are $age years old"

# Silent input (passwords)
read -sp "Enter password: " password
echo  # New line after hidden input
echo "Password received"

# Input with timeout
read -t 5 -p "Quick! Enter value (5 sec): " value

# Input with default
read -p "Enter name [John]: " name
name=${name:-John}
echo "Name: $name"
```

---

## Conditionals

### If Statements

```bash
#!/bin/bash

# Basic if
if [ "$1" = "hello" ]; then
    echo "Hello to you too!"
fi

# If-else
if [ -f "config.txt" ]; then
    echo "Config file exists"
else
    echo "Config file not found"
fi

# If-elif-else
if [ "$1" = "start" ]; then
    echo "Starting..."
elif [ "$1" = "stop" ]; then
    echo "Stopping..."
elif [ "$1" = "restart" ]; then
    echo "Restarting..."
else
    echo "Usage: $0 {start|stop|restart}"
fi
```

### Test Operators

```bash
# String comparisons
[ "$a" = "$b" ]   # Equal
[ "$a" != "$b" ]  # Not equal
[ -z "$a" ]       # Empty string
[ -n "$a" ]       # Not empty

# Numeric comparisons
[ "$a" -eq "$b" ] # Equal
[ "$a" -ne "$b" ] # Not equal
[ "$a" -lt "$b" ] # Less than
[ "$a" -le "$b" ] # Less or equal
[ "$a" -gt "$b" ] # Greater than
[ "$a" -ge "$b" ] # Greater or equal

# File tests
[ -f "$file" ]    # Is regular file
[ -d "$dir" ]     # Is directory
[ -e "$path" ]    # Exists
[ -r "$file" ]    # Is readable
[ -w "$file" ]    # Is writable
[ -x "$file" ]    # Is executable
[ -s "$file" ]    # File size > 0
[ -L "$file" ]    # Is symbolic link

# Logical operators
[ "$a" = "1" ] && [ "$b" = "2" ]  # AND
[ "$a" = "1" ] || [ "$b" = "2" ]  # OR
[ ! -f "$file" ]                   # NOT
```

### Modern Test Syntax [[ ]]

```bash
#!/bin/bash

# Double brackets - more features
if [[ "$name" == "John" ]]; then
    echo "Hello John"
fi

# Pattern matching
if [[ "$file" == *.txt ]]; then
    echo "Text file"
fi

# Regex matching
if [[ "$email" =~ ^[a-z]+@[a-z]+\.[a-z]+$ ]]; then
    echo "Valid email format"
fi

# Compound conditions
if [[ "$age" -ge 18 && "$country" == "US" ]]; then
    echo "Adult in US"
fi
```

### Case Statements

```bash
#!/bin/bash

case "$1" in
    start)
        echo "Starting service..."
        ;;
    stop)
        echo "Stopping service..."
        ;;
    restart)
        echo "Restarting service..."
        ;;
    status)
        echo "Service status: running"
        ;;
    *)
        echo "Usage: $0 {start|stop|restart|status}"
        exit 1
        ;;
esac

# Pattern matching in case
case "$file" in
    *.txt)
        echo "Text file"
        ;;
    *.jpg|*.png|*.gif)
        echo "Image file"
        ;;
    *.sh)
        echo "Shell script"
        ;;
    *)
        echo "Unknown file type"
        ;;
esac
```

---

## Loops

### For Loops

```bash
#!/bin/bash

# Loop over list
for name in Alice Bob Charlie; do
    echo "Hello, $name"
done

# Loop over range
for i in {1..5}; do
    echo "Number: $i"
done

# Loop with step
for i in {0..10..2}; do
    echo "Even: $i"
done

# C-style for loop
for ((i=0; i<5; i++)); do
    echo "Index: $i"
done

# Loop over files
for file in *.txt; do
    echo "Processing: $file"
done

# Loop over command output
for user in $(cat users.txt); do
    echo "User: $user"
done

# Loop over array
fruits=("apple" "banana" "cherry")
for fruit in "${fruits[@]}"; do
    echo "Fruit: $fruit"
done
```

### While Loops

```bash
#!/bin/bash

# Basic while
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    ((count++))
done

# Read file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < input.txt

# Infinite loop with break
while true; do
    read -p "Enter command (quit to exit): " cmd
    if [ "$cmd" = "quit" ]; then
        break
    fi
    echo "You entered: $cmd"
done

# Continue to skip iteration
for i in {1..10}; do
    if [ $((i % 2)) -eq 0 ]; then
        continue  # Skip even numbers
    fi
    echo "Odd: $i"
done
```

### Until Loops

```bash
#!/bin/bash

# Until loop (opposite of while)
count=1
until [ $count -gt 5 ]; do
    echo "Count: $count"
    ((count++))
done

# Wait for file
until [ -f "/tmp/ready.flag" ]; do
    echo "Waiting for ready flag..."
    sleep 1
done
echo "Ready!"
```

---

## Functions

### Defining Functions

```bash
#!/bin/bash

# Simple function
greet() {
    echo "Hello, World!"
}

# Function with parameters
greet_user() {
    local name=$1
    local time=$2
    echo "Good $time, $name!"
}

# Function with return value
is_even() {
    local num=$1
    if [ $((num % 2)) -eq 0 ]; then
        return 0  # true
    else
        return 1  # false
    fi
}

# Call functions
greet
greet_user "John" "morning"

if is_even 4; then
    echo "4 is even"
fi
```

### Returning Values

```bash
#!/bin/bash

# Return via echo (capture output)
get_date() {
    echo $(date +%Y-%m-%d)
}

today=$(get_date)
echo "Today is: $today"

# Return via global variable
calculate() {
    local a=$1
    local b=$2
    RESULT=$((a + b))
}

calculate 5 3
echo "Result: $RESULT"

# Return array
get_files() {
    local dir=$1
    FILES=($(ls "$dir"))
}

get_files /home
echo "Files: ${FILES[@]}"
```

### Local Variables

```bash
#!/bin/bash

# Global variable
count=10

increment() {
    local count=0  # Local variable shadows global
    ((count++))
    echo "Local count: $count"
}

increment
echo "Global count: $count"  # Still 10
```

---

## Arrays

### Working with Arrays

```bash
#!/bin/bash

# Declare array
fruits=("apple" "banana" "cherry")

# Access elements
echo "${fruits[0]}"      # apple
echo "${fruits[1]}"      # banana
echo "${fruits[-1]}"     # cherry (last element)

# All elements
echo "${fruits[@]}"      # apple banana cherry
echo "${fruits[*]}"      # apple banana cherry

# Array length
echo "${#fruits[@]}"     # 3

# Add element
fruits+=("date")

# Modify element
fruits[1]="blueberry"

# Delete element
unset fruits[2]

# Slice
echo "${fruits[@]:1:2}"  # Elements 1-2

# Loop array
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done

# Loop with index
for i in "${!fruits[@]}"; do
    echo "$i: ${fruits[$i]}"
done
```

### Associative Arrays

```bash
#!/bin/bash

# Declare associative array
declare -A user

# Set values
user[name]="John"
user[age]=25
user[city]="New York"

# Access values
echo "${user[name]}"

# All keys
echo "${!user[@]}"

# All values
echo "${user[@]}"

# Loop
for key in "${!user[@]}"; do
    echo "$key: ${user[$key]}"
done
```

---

## Error Handling

### Exit Codes

```bash
#!/bin/bash

# Exit with code
exit 0   # Success
exit 1   # General error
exit 2   # Misuse of command

# Check exit code
command
if [ $? -eq 0 ]; then
    echo "Command succeeded"
else
    echo "Command failed"
fi

# Or use && and ||
command && echo "Success" || echo "Failed"
```

### Error Handling Patterns

```bash
#!/bin/bash

# Exit on error
set -e

# Exit on undefined variable
set -u

# Exit on pipe failure
set -o pipefail

# Combine (recommended for scripts)
set -euo pipefail

# Trap errors
trap 'echo "Error on line $LINENO"; exit 1' ERR

# Cleanup on exit
cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/tempfile
}
trap cleanup EXIT

# Check command exists
if ! command -v docker &> /dev/null; then
    echo "Docker is not installed"
    exit 1
fi
```

### Custom Error Handling

```bash
#!/bin/bash

# Error function
error() {
    echo "ERROR: $1" >&2
    exit "${2:-1}"
}

# Warning function
warn() {
    echo "WARNING: $1" >&2
}

# Usage
[ -f "config.txt" ] || error "Config file not found" 2
[ -n "$API_KEY" ] || error "API_KEY is required"
```

---

## Practical Scripts

### Deployment Script

```bash
#!/bin/bash
set -euo pipefail

# deploy.sh - Deploy application

APP_DIR="/var/www/app"
BACKUP_DIR="/var/backups"
REPO="git@github.com:user/app.git"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1"
}

error() {
    log "ERROR: $1" >&2
    exit 1
}

# Backup current version
backup() {
    log "Creating backup..."
    tar -czf "$BACKUP_DIR/app-$(date +%Y%m%d%H%M%S).tar.gz" "$APP_DIR"
}

# Pull latest code
pull_code() {
    log "Pulling latest code..."
    cd "$APP_DIR"
    git fetch origin
    git reset --hard origin/main
}

# Install dependencies
install_deps() {
    log "Installing dependencies..."
    cd "$APP_DIR"
    npm ci --production
}

# Restart service
restart_service() {
    log "Restarting service..."
    sudo systemctl restart app
}

# Main
main() {
    log "Starting deployment..."
    backup
    pull_code
    install_deps
    restart_service
    log "Deployment complete!"
}

main "$@"
```

### Log Analyzer

```bash
#!/bin/bash

# analyze-logs.sh - Analyze application logs

LOG_FILE="${1:-/var/log/app.log}"

if [ ! -f "$LOG_FILE" ]; then
    echo "Log file not found: $LOG_FILE"
    exit 1
fi

echo "=== Log Analysis: $LOG_FILE ==="
echo

echo "Total lines: $(wc -l < "$LOG_FILE")"
echo "Error count: $(grep -c "ERROR" "$LOG_FILE" || echo 0)"
echo "Warning count: $(grep -c "WARN" "$LOG_FILE" || echo 0)"
echo

echo "=== Top 10 Error Messages ==="
grep "ERROR" "$LOG_FILE" | \
    sed 's/.*ERROR//' | \
    sort | uniq -c | sort -rn | head -10

echo
echo "=== Errors by Hour ==="
grep "ERROR" "$LOG_FILE" | \
    awk '{print substr($2,1,2)}' | \
    sort | uniq -c | sort -k2
```

### Backup Script

```bash
#!/bin/bash
set -euo pipefail

# backup.sh - Backup important directories

BACKUP_DIR="/backups"
SOURCES=("/home" "/etc" "/var/www")
DATE=$(date +%Y%m%d)
RETENTION_DAYS=7

log() {
    echo "[$(date +'%H:%M:%S')] $1"
}

# Create backup
create_backup() {
    local source=$1
    local name=$(basename "$source")
    local backup_file="$BACKUP_DIR/${name}-${DATE}.tar.gz"

    log "Backing up $source..."
    tar -czf "$backup_file" "$source" 2>/dev/null
    log "Created: $backup_file ($(du -h "$backup_file" | cut -f1))"
}

# Remove old backups
cleanup_old() {
    log "Removing backups older than $RETENTION_DAYS days..."
    find "$BACKUP_DIR" -name "*.tar.gz" -mtime +$RETENTION_DAYS -delete
}

# Main
main() {
    mkdir -p "$BACKUP_DIR"

    log "Starting backup..."
    for source in "${SOURCES[@]}"; do
        if [ -d "$source" ]; then
            create_backup "$source"
        else
            log "Skipping $source (not found)"
        fi
    done

    cleanup_old
    log "Backup complete!"
}

main
```

### System Health Check

```bash
#!/bin/bash

# health-check.sh - Check system health

check_disk() {
    echo "=== Disk Usage ==="
    df -h | grep -E '^/dev/'

    # Alert if disk > 80%
    df -h | awk '$5 ~ /[0-9]/ && int($5) > 80 {print "WARNING: " $6 " is " $5 " full"}'
}

check_memory() {
    echo
    echo "=== Memory Usage ==="
    free -h

    # Get percentage used
    local used=$(free | awk '/Mem:/ {printf "%.0f", $3/$2 * 100}')
    if [ "$used" -gt 80 ]; then
        echo "WARNING: Memory usage is ${used}%"
    fi
}

check_cpu() {
    echo
    echo "=== CPU Load ==="
    uptime

    local load=$(uptime | awk -F'load average:' '{print $2}' | cut -d, -f1 | xargs)
    local cpus=$(nproc)

    if (( $(echo "$load > $cpus" | bc -l) )); then
        echo "WARNING: High CPU load: $load (CPUs: $cpus)"
    fi
}

check_services() {
    echo
    echo "=== Service Status ==="
    for service in nginx postgresql redis; do
        if systemctl is-active --quiet "$service" 2>/dev/null; then
            echo "✓ $service is running"
        else
            echo "✗ $service is NOT running"
        fi
    done
}

main() {
    echo "=== System Health Check ==="
    echo "Date: $(date)"
    echo "Hostname: $(hostname)"
    echo

    check_disk
    check_memory
    check_cpu
    check_services
}

main
```

---

## Best Practices

### Script Template

```bash
#!/bin/bash
#
# script-name.sh - Brief description
#
# Usage: ./script-name.sh [options] <arguments>
#
# Options:
#   -h, --help     Show this help message
#   -v, --verbose  Enable verbose output
#

set -euo pipefail

# Constants
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "$0")"

# Variables
VERBOSE=false

# Functions
usage() {
    grep '^#' "$0" | grep -v '#!/' | cut -c3-
    exit 0
}

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1"
}

debug() {
    if $VERBOSE; then
        echo "DEBUG: $1" >&2
    fi
}

error() {
    echo "ERROR: $1" >&2
    exit "${2:-1}"
}

# Parse arguments
while [[ $# -gt 0 ]]; do
    case $1 in
        -h|--help)
            usage
            ;;
        -v|--verbose)
            VERBOSE=true
            shift
            ;;
        *)
            break
            ;;
    esac
done

# Main function
main() {
    debug "Starting script..."

    # Your code here

    debug "Script complete"
}

main "$@"
```

### Guidelines

```bash
# 1. Always quote variables
echo "$variable"      # Good
echo $variable        # Bad - word splitting issues

# 2. Use [[ ]] instead of [ ]
[[ "$a" == "$b" ]]   # Good - more features
[ "$a" = "$b" ]      # OK but limited

# 3. Use local variables in functions
my_func() {
    local var="value"  # Local scope
}

# 4. Check return values
if ! command; then
    echo "Command failed"
fi

# 5. Use meaningful names
user_count=10          # Good
x=10                   # Bad

# 6. Add comments
# Calculate total disk usage
du -sh /home/* | sort -h
```

---

## Quick Reference

### Operators

| Operator | Description |
|----------|-------------|
| `-eq` | Equal (numeric) |
| `-ne` | Not equal (numeric) |
| `-lt` | Less than |
| `-gt` | Greater than |
| `=` / `==` | String equal |
| `!=` | String not equal |
| `-f` | Is file |
| `-d` | Is directory |
| `-z` | Is empty string |
| `-n` | Is not empty string |

### Common Patterns

```bash
# Default value
${var:-default}

# Error if unset
${var:?error message}

# Substring
${var:0:5}

# Replace
${var/old/new}

# Length
${#var}
```

---

## Key Takeaways

1. **Start with shebang** - `#!/bin/bash`
2. **Use set -euo pipefail** - Catch errors early
3. **Quote variables** - Prevent word splitting
4. **Use functions** - Organize and reuse code
5. **Handle errors** - Check return codes
6. **Test scripts** - Debug with -x flag

---

## Congratulations!

You've completed the Linux & CLI section! You now know how to:
- Navigate the filesystem
- Manage file permissions
- Monitor and control processes
- Connect securely with SSH
- Automate tasks with shell scripts

Next up is **Docker** - containerizing applications for consistent deployment!
