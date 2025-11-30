# Day 1: Linux Filesystem - Navigating and Managing Files

## Introduction

Understanding the Linux filesystem is essential for every developer. Whether you're deploying applications, managing servers, or working in containers, you'll constantly interact with the command line. Today, you'll learn how to navigate, create, manipulate, and organize files and directories in Linux.

## Learning Objectives

By the end of this lesson, you will be able to:
- Navigate the Linux directory structure
- Create, copy, move, and delete files and directories
- View and search file contents
- Understand file paths and wildcards
- Use essential file management commands

---

## The Linux Directory Structure

```
/                           # Root directory
├── bin/                    # Essential user binaries (ls, cp, mv)
├── boot/                   # Boot loader files
├── dev/                    # Device files
├── etc/                    # System configuration files
├── home/                   # User home directories
│   └── username/           # Your home directory (~)
├── lib/                    # Shared libraries
├── media/                  # Removable media mount points
├── mnt/                    # Temporary mount points
├── opt/                    # Optional software packages
├── proc/                   # Process information (virtual)
├── root/                   # Root user's home directory
├── sbin/                   # System binaries
├── srv/                    # Service data
├── sys/                    # System information (virtual)
├── tmp/                    # Temporary files
├── usr/                    # User programs and data
│   ├── bin/                # User binaries
│   ├── lib/                # Libraries
│   ├── local/              # Locally installed software
│   └── share/              # Shared data
└── var/                    # Variable data (logs, databases)
    ├── log/                # Log files
    ├── www/                # Web server files
    └── lib/                # Variable state data
```

### Key Directories for Developers

| Directory | Purpose |
|-----------|---------|
| `/home/user` | Your files, projects, configurations |
| `/etc` | System and application configs |
| `/var/log` | Application and system logs |
| `/tmp` | Temporary files (cleared on reboot) |
| `/opt` | Third-party applications |
| `/usr/local` | Locally compiled software |

---

## Navigation Commands

### pwd - Print Working Directory

```bash
# Show current directory
pwd
# Output: /home/username/projects

# Use in scripts
current_dir=$(pwd)
echo "You are in: $current_dir"
```

### cd - Change Directory

```bash
# Go to home directory
cd
cd ~
cd $HOME

# Go to specific directory
cd /var/log
cd ~/projects

# Go to parent directory
cd ..

# Go to previous directory
cd -

# Go up multiple levels
cd ../..
cd ../../other-folder

# Using tab completion
cd /ho[TAB]     # Completes to /home/
cd ~/pro[TAB]   # Completes to ~/projects/
```

### ls - List Directory Contents

```bash
# Basic listing
ls

# Long format (detailed)
ls -l
# drwxr-xr-x  5 user group  4096 Jan 15 10:30 projects
# -rw-r--r--  1 user group  1234 Jan 15 10:25 file.txt

# Show hidden files (starting with .)
ls -a
ls -la    # Long format + hidden

# Human-readable file sizes
ls -lh
# -rw-r--r--  1 user group  1.2K Jan 15 10:25 file.txt
# -rw-r--r--  1 user group  45M  Jan 14 09:00 large-file.zip

# Sort by time (newest first)
ls -lt

# Sort by size (largest first)
ls -lS

# Reverse order
ls -lr

# Recursive (show subdirectories)
ls -R

# Show directory itself (not contents)
ls -ld /var/log

# One file per line
ls -1

# Colorized output
ls --color=auto
```

---

## File and Directory Operations

### Creating Files and Directories

```bash
# Create empty file
touch newfile.txt
touch file1.txt file2.txt file3.txt

# Create file with content
echo "Hello World" > hello.txt
cat > notes.txt << EOF
Line 1
Line 2
Line 3
EOF

# Create directory
mkdir projects
mkdir -p projects/web/frontend  # Create parent directories

# Create multiple directories
mkdir dir1 dir2 dir3
mkdir -p project/{src,tests,docs}
```

### Copying Files and Directories

```bash
# Copy file
cp source.txt destination.txt
cp file.txt /path/to/destination/

# Copy with new name
cp original.txt backup.txt

# Copy multiple files to directory
cp file1.txt file2.txt /destination/

# Copy directory (recursive)
cp -r source-dir/ destination-dir/

# Interactive (prompt before overwrite)
cp -i file.txt destination/

# Preserve attributes (permissions, timestamps)
cp -p file.txt backup.txt

# Verbose (show progress)
cp -v large-file.zip /backup/
```

### Moving and Renaming

```bash
# Move file
mv file.txt /new/location/

# Rename file
mv oldname.txt newname.txt

# Move and rename
mv file.txt /path/to/newname.txt

# Move multiple files
mv *.txt /text-files/

# Move directory
mv old-dir/ new-location/

# Interactive (prompt before overwrite)
mv -i file.txt destination/

# Don't overwrite existing files
mv -n file.txt destination/
```

### Deleting Files and Directories

```bash
# Remove file
rm file.txt
rm file1.txt file2.txt

# Remove with confirmation
rm -i file.txt

# Force remove (no confirmation)
rm -f file.txt

# Remove directory (must be empty)
rmdir empty-directory/

# Remove directory and contents (recursive)
rm -r directory/
rm -rf directory/  # Force, no confirmation

# Remove all files in directory
rm directory/*

# Safe removal - move to trash (install trash-cli)
trash file.txt
trash-restore  # Restore from trash
```

---

## Viewing File Contents

### cat - Concatenate and Display

```bash
# Display file contents
cat file.txt

# Display with line numbers
cat -n file.txt

# Display multiple files
cat file1.txt file2.txt

# Concatenate files into new file
cat file1.txt file2.txt > combined.txt
```

### less and more - Paged Viewing

```bash
# View file with paging
less file.txt
more file.txt

# Navigation in less:
# Space/Page Down - next page
# b/Page Up - previous page
# g - go to beginning
# G - go to end
# /pattern - search forward
# ?pattern - search backward
# n - next search result
# q - quit
```

### head and tail - View Start/End

```bash
# First 10 lines
head file.txt

# First N lines
head -n 20 file.txt
head -20 file.txt

# Last 10 lines
tail file.txt

# Last N lines
tail -n 20 file.txt

# Follow file (watch for new content)
tail -f /var/log/syslog
tail -f app.log

# Follow multiple files
tail -f file1.log file2.log
```

### wc - Word Count

```bash
# Lines, words, characters
wc file.txt
# 100  500  3000 file.txt

# Lines only
wc -l file.txt

# Words only
wc -w file.txt

# Characters only
wc -c file.txt

# Count files in directory
ls | wc -l
```

---

## Searching and Finding

### find - Find Files

```bash
# Find by name
find /path -name "filename.txt"
find . -name "*.js"

# Case-insensitive
find . -iname "readme*"

# Find directories
find . -type d -name "node_modules"

# Find files
find . -type f -name "*.log"

# Find by size
find . -size +100M      # Larger than 100MB
find . -size -1k        # Smaller than 1KB

# Find by modification time
find . -mtime -7        # Modified in last 7 days
find . -mtime +30       # Modified more than 30 days ago

# Find and execute command
find . -name "*.tmp" -delete
find . -name "*.js" -exec wc -l {} \;
find . -type f -name "*.log" -exec rm {} +

# Find empty files/directories
find . -empty

# Exclude directories
find . -name "*.js" -not -path "./node_modules/*"
```

### grep - Search File Contents

```bash
# Search for pattern in file
grep "error" logfile.txt

# Case-insensitive
grep -i "error" logfile.txt

# Show line numbers
grep -n "error" logfile.txt

# Recursive search in directory
grep -r "TODO" ./src/

# Show only filenames
grep -l "pattern" *.txt

# Invert match (lines NOT containing)
grep -v "debug" logfile.txt

# Count matches
grep -c "error" logfile.txt

# Context (lines before/after)
grep -B 3 -A 3 "error" logfile.txt  # 3 before, 3 after
grep -C 3 "error" logfile.txt        # 3 before and after

# Extended regex
grep -E "error|warning|fatal" logfile.txt

# Search for whole word
grep -w "error" logfile.txt

# Multiple patterns
grep -e "error" -e "warning" logfile.txt
```

---

## Wildcards and Patterns

### Glob Patterns

```bash
# * - Match any characters
ls *.txt           # All .txt files
ls file*           # Files starting with "file"
ls *test*          # Files containing "test"

# ? - Match single character
ls file?.txt       # file1.txt, file2.txt, but not file10.txt
ls ???.txt         # Three-character names

# [] - Match character set
ls file[123].txt   # file1.txt, file2.txt, file3.txt
ls file[1-5].txt   # file1.txt through file5.txt
ls [Rr]eadme.txt   # Readme.txt or readme.txt
ls file[!0-9].txt  # Not followed by number

# {} - Brace expansion
mkdir {src,tests,docs}
cp file.{txt,bak}  # Copy file.txt to file.bak
touch file{1..5}.txt  # file1.txt through file5.txt
echo {a..z}        # a b c ... z
```

---

## File Paths

### Absolute vs Relative Paths

```bash
# Absolute path (starts from root)
/home/user/projects/app/src/index.js

# Relative path (from current directory)
./src/index.js
../other-project/file.txt

# Special path references
.     # Current directory
..    # Parent directory
~     # Home directory
-     # Previous directory (cd only)
```

### Path Operations

```bash
# Get filename from path
basename /path/to/file.txt
# Output: file.txt

# Get directory from path
dirname /path/to/file.txt
# Output: /path/to

# Get absolute path
realpath ./relative/path
readlink -f ./symlink

# Combine paths
path="/home/user"
file="document.txt"
full_path="$path/$file"
```

---

## Useful Shortcuts

### Keyboard Shortcuts

```
Ctrl + A    Move cursor to beginning of line
Ctrl + E    Move cursor to end of line
Ctrl + U    Clear line before cursor
Ctrl + K    Clear line after cursor
Ctrl + W    Delete word before cursor
Ctrl + L    Clear screen
Ctrl + R    Reverse search history
Ctrl + C    Cancel current command
Ctrl + D    Exit shell / End of input
Tab         Auto-complete
Tab Tab     Show all completions
```

### Command History

```bash
# View history
history
history 20    # Last 20 commands

# Run previous command
!!

# Run command by number
!123

# Run last command starting with...
!cd
!git

# Search history
Ctrl + R, then type

# Clear history
history -c
```

---

## Practice Exercises

### Exercise 1: Navigation

```bash
# 1. Go to your home directory
# 2. Create a directory structure: ~/practice/web/src
# 3. Navigate to src
# 4. Go back to practice using relative path
# 5. Go to /var/log, then back to previous directory
```

### Exercise 2: File Operations

```bash
# 1. Create files: index.html, style.css, app.js
# 2. Create a backup directory
# 3. Copy all files to backup
# 4. Rename index.html to home.html
# 5. Delete app.js
```

### Exercise 3: Searching

```bash
# 1. Find all .js files in your home directory
# 2. Search for "TODO" in all source files
# 3. Find files larger than 10MB
# 4. Find files modified in the last 24 hours
```

---

## Quick Reference

| Command | Description |
|---------|-------------|
| `pwd` | Print working directory |
| `cd` | Change directory |
| `ls` | List directory contents |
| `mkdir` | Create directory |
| `touch` | Create empty file |
| `cp` | Copy files/directories |
| `mv` | Move/rename files |
| `rm` | Remove files/directories |
| `cat` | Display file contents |
| `less` | Page through file |
| `head` | View first lines |
| `tail` | View last lines |
| `find` | Find files |
| `grep` | Search file contents |
| `wc` | Count lines/words |

---

## Key Takeaways

1. **Everything is a file** - Including directories and devices
2. **Paths matter** - Absolute vs relative
3. **Tab completion** - Your best friend
4. **Wildcards save time** - `*`, `?`, `[]`
5. **`-r` for recursive** - Works with most commands
6. **Be careful with `rm -rf`** - No undo!

---

## What's Next?

Tomorrow, we'll learn about **File Permissions** - controlling who can read, write, and execute files on Linux systems.
