# Linux Commands Complete Mastery Tutorial
## From Beginner to Advanced - The Ultimate Guide

---

## Table of Contents

### Part 1: Foundation (Beginner Level)
1. [Introduction to Linux Command Line](#1-introduction-to-linux-command-line)
2. [Terminal Basics](#2-terminal-basics)
3. [File System Navigation](#3-file-system-navigation)
4. [Basic File Operations](#4-basic-file-operations)
5. [Viewing and Reading Files](#5-viewing-and-reading-files)
6. [Basic Text Editing](#6-basic-text-editing)
7. [Getting Help](#7-getting-help)

### Part 2: Essential Skills (Intermediate Level)
8. [File Permissions and Ownership](#8-file-permissions-and-ownership)
9. [Text Processing Commands](#9-text-processing-commands)
10. [Search and Find Commands](#10-search-and-find-commands)
11. [Process Management](#11-process-management)
12. [System Information](#12-system-information)
13. [User Management](#13-user-management)
14. [Package Management](#14-package-management)
15. [Archiving and Compression](#15-archiving-and-compression)

### Part 3: Advanced Operations (Advanced Level)
16. [Disk and Storage Management](#16-disk-and-storage-management)
17. [Networking Commands](#17-networking-commands)
18. [Network Diagnostics and Troubleshooting](#18-network-diagnostics-and-troubleshooting)
19. [System Monitoring and Performance](#19-system-monitoring-and-performance)
20. [Shell Scripting Essentials](#20-shell-scripting-essentials)
21. [Advanced Text Processing](#21-advanced-text-processing)
22. [System Administration](#22-system-administration)
23. [Security and Firewall](#23-security-and-firewall)
24. [Kernel and System Tuning](#24-kernel-and-system-tuning)
25. [Advanced Networking](#25-advanced-networking)

---

# Part 1: Foundation (Beginner Level)

## 1. Introduction to Linux Command Line

### What is the Command Line?
The command line interface (CLI) is a text-based interface for interacting with your computer's operating system. Unlike graphical user interfaces (GUIs), the CLI allows you to execute commands by typing text.

### Why Use the Command Line?
- **Efficiency**: Many tasks are faster via command line
- **Power**: Access to advanced features not available in GUI
- **Automation**: Scripts can automate repetitive tasks
- **Remote Management**: Essential for server administration
- **Resource Light**: Uses minimal system resources

### Shell Types
- **Bash** (Bourne Again Shell): Most common Linux shell
- **Zsh** (Z Shell): Modern shell with advanced features
- **Fish** (Friendly Interactive Shell): User-friendly shell
- **Sh** (Bourne Shell): Original Unix shell

### Command Structure
```
command [options] [arguments]
```
- **command**: The program to execute
- **options**: Modify command behavior (usually start with - or --)
- **arguments**: Data for the command to process

---

## 2. Terminal Basics

### `pwd` - Print Working Directory
**Description**: Displays the current directory path.

**Syntax**:
```bash
pwd [OPTIONS]
```

**Options**:
- `-L`: Display logical path (default, follows symlinks)
- `-P`: Display physical path (resolves symlinks)

**Examples**:
```bash
# Show current directory
pwd
# Output: /home/username/documents

# Show physical path (resolve symlinks)
pwd -P
```

**Use Cases**:
- Verify your current location in the filesystem
- Use in scripts to determine execution context
- Debug navigation issues

---

### `echo` - Display Text
**Description**: Prints text or variables to standard output.

**Syntax**:
```bash
echo [OPTIONS] [STRING]
```

**Options**:
- `-n`: Don't output trailing newline
- `-e`: Enable interpretation of backslash escapes
- `-E`: Disable interpretation of backslash escapes (default)

**Escape Sequences** (with `-e`):
- `\n`: Newline
- `\t`: Tab
- `\\`: Backslash
- `\a`: Alert (bell)
- `\b`: Backspace
- `\c`: Suppress trailing newline

**Examples**:
```bash
# Simple text output
echo "Hello, World!"
# Output: Hello, World!

# Print variable
NAME="John"
echo "Hello, $NAME"
# Output: Hello, John

# Without newline
echo -n "Loading..."
# Output: Loading... (cursor stays on same line)

# With escape sequences
echo -e "Line 1\nLine 2\tTabbed"
# Output:
# Line 1
# Line 2    Tabbed

# Print multiple lines
echo -e "Name: John\nAge: 30\nCity: NYC"
```

**Use Cases**:
- Display messages in scripts
- Debug by printing variable values
- Create simple output formatting

---

### `clear` - Clear Terminal Screen
**Description**: Clears the terminal screen.

**Syntax**:
```bash
clear
```

**Keyboard Shortcut**:
- `Ctrl + L`: Clear screen (keeps scrollback)

**Examples**:
```bash
# Clear the screen
clear

# Clear and reset terminal
reset
```

---

### `history` - Command History
**Description**: Displays previously executed commands.

**Syntax**:
```bash
history [OPTIONS] [N]
```

**Options**:
- `-c`: Clear history list
- `-d offset`: Delete history entry at offset
- `-a`: Append new history entries to history file
- `-w`: Write current history to history file
- `N`: Display last N commands

**Examples**:
```bash
# Show all command history
history

# Show last 10 commands
history 10

# Clear history
history -c

# Search history (Ctrl+R in terminal)
# Type Ctrl+R, then start typing command

# Execute command from history
!100        # Execute command number 100
!!          # Execute last command
!-2         # Execute command 2 positions back
!ssh        # Execute last command starting with 'ssh'
```

**History File**: `~/.bash_history` (for Bash)

**Use Cases**:
- Review previously executed commands
- Reuse complex commands
- Audit command execution
- Learn from past commands

---

### `exit` - Exit Shell
**Description**: Terminates the current shell session.

**Syntax**:
```bash
exit [N]
```

**Parameters**:
- `N`: Exit status code (0-255, default is 0)

**Examples**:
```bash
# Exit with success status
exit

# Exit with specific code
exit 1      # Indicates error
exit 0      # Indicates success
```

**Keyboard Shortcut**:
- `Ctrl + D`: Exit shell (EOF signal)

---

### `date` - Display/Set Date and Time
**Description**: Shows or sets system date and time.

**Syntax**:
```bash
date [OPTIONS] [+FORMAT]
```

**Common Format Codes**:
- `%Y`: Year (4 digits)
- `%m`: Month (01-12)
- `%d`: Day (01-31)
- `%H`: Hour (00-23)
- `%M`: Minute (00-59)
- `%S`: Second (00-59)
- `%A`: Full weekday name
- `%B`: Full month name
- `%Z`: Time zone
- `%s`: Seconds since epoch (1970-01-01 00:00:00 UTC)

**Examples**:
```bash
# Current date and time
date
# Output: Thu Dec  4 12:30:45 PST 2025

# Custom format
date "+%Y-%m-%d"
# Output: 2025-12-04

# Full date format
date "+%A, %B %d, %Y"
# Output: Thursday, December 04, 2025

# Date and time
date "+%Y-%m-%d %H:%M:%S"
# Output: 2025-12-04 12:30:45

# Unix timestamp
date +%s
# Output: 1733338245

# Convert timestamp to date
date -d @1733338245
# Output: Thu Dec  4 12:30:45 PST 2025

# Date arithmetic
date -d "tomorrow"
date -d "next Monday"
date -d "2 weeks ago"
date -d "+5 days"
```

**Use Cases**:
- Timestamp log files
- Calculate date differences
- Schedule tasks
- Generate date-based filenames

---

### `cal` - Display Calendar
**Description**: Shows calendar for specified month/year.

**Syntax**:
```bash
cal [OPTIONS] [[month] year]
```

**Options**:
- `-1`: Display single month (default)
- `-3`: Display previous, current, and next month
- `-y`: Display entire year
- `-j`: Display Julian dates (day of year)

**Examples**:
```bash
# Current month
cal

# Specific month and year
cal 12 2025

# Entire year
cal 2025

# Three months
cal -3

# Year with Julian dates
cal -j 2025
```

---

## 3. File System Navigation

### Linux File System Hierarchy
```
/                   # Root directory
├── /bin            # Essential command binaries
├── /boot           # Boot loader files
├── /dev            # Device files
├── /etc            # System configuration files
├── /home           # User home directories
├── /lib            # System libraries
├── /media          # Mount points for removable media
├── /mnt            # Temporary mount points
├── /opt            # Optional software packages
├── /proc           # Process information (virtual filesystem)
├── /root           # Root user's home directory
├── /sbin           # System administration binaries
├── /srv            # Service data
├── /sys            # System information (virtual filesystem)
├── /tmp            # Temporary files
├── /usr            # User programs and data
└── /var            # Variable data (logs, caches)
```

### Path Types
- **Absolute Path**: Complete path from root (`/home/user/file.txt`)
- **Relative Path**: Path from current directory (`../documents/file.txt`)

### Special Directory Symbols
- `.`: Current directory
- `..`: Parent directory
- `~`: Home directory
- `-`: Previous directory

---

### `cd` - Change Directory
**Description**: Changes the current working directory.

**Syntax**:
```bash
cd [DIRECTORY]
```

**Examples**:
```bash
# Go to home directory
cd
cd ~

# Go to specific directory (absolute path)
cd /var/log

# Go to specific directory (relative path)
cd documents
cd ./documents

# Go to parent directory
cd ..

# Go up two levels
cd ../..

# Go to previous directory
cd -

# Go to root directory
cd /

# Go to another user's home
cd ~username
```

**Use Cases**:
- Navigate filesystem
- Access different directories for file operations
- Return to home directory quickly

---

### `ls` - List Directory Contents
**Description**: Lists files and directories.

**Syntax**:
```bash
ls [OPTIONS] [FILE/DIRECTORY]
```

**Common Options**:
- `-l`: Long format (detailed information)
- `-a`: Show hidden files (starting with .)
- `-h`: Human-readable file sizes
- `-R`: Recursive listing
- `-t`: Sort by modification time
- `-r`: Reverse order
- `-S`: Sort by file size
- `-1`: One file per line
- `-d`: List directories themselves, not their contents
- `-i`: Show inode numbers
- `-F`: Append indicator (/ for directories, * for executables)

**Examples**:
```bash
# Simple listing
ls

# Long format
ls -l
# Output: -rw-r--r-- 1 user group 1234 Dec 4 12:00 file.txt
#         │││││││││  │ │    │     │    │        │
#         │││││││││  │ │    │     │    │        └─ filename
#         │││││││││  │ │    │     │    └────────── modification time
#         │││││││││  │ │    │     └─────────────── size in bytes
#         │││││││││  │ │    └───────────────────── group
#         │││││││││  │ └────────────────────────── owner
#         │││││││││  └──────────────────────────── number of hard links
#         └┴┴┴┴┴┴┴┴───────────────────────────── permissions

# Show all files including hidden
ls -la

# Human-readable sizes
ls -lh
# Output: -rw-r--r-- 1 user group 1.2K Dec 4 12:00 file.txt

# Sort by modification time (newest first)
ls -lt

# Sort by size (largest first)
ls -lS

# Recursive listing
ls -R

# List only directories
ls -d */

# Show with file type indicators
ls -F
# Output: directory/ executable* symlink@ file

# Specific directory
ls /var/log

# Multiple directories
ls /home /var
```

**Permission String Explanation**:
```
-rw-r--r--
│││││││││
││││││││└─ Others execute
│││││││└── Others write
││││││└─── Others read
│││││└──── Group execute
││││└───── Group write
│││└────── Group read
││└─────── Owner execute
│└──────── Owner write
└───────── Owner read

First character:
- : Regular file
d : Directory
l : Symbolic link
b : Block device
c : Character device
p : Named pipe
s : Socket
```

---

### `tree` - Display Directory Tree
**Description**: Shows directory structure in tree format.

**Syntax**:
```bash
tree [OPTIONS] [DIRECTORY]
```

**Common Options**:
- `-L level`: Limit depth of tree
- `-d`: Show only directories
- `-a`: Show hidden files
- `-h`: Human-readable sizes
- `-p`: Show permissions
- `-u`: Show owner
- `-g`: Show group
- `-D`: Show last modification date
- `-f`: Show full path
- `--du`: Show disk usage

**Installation** (if not present):
```bash
# Ubuntu/Debian
sudo apt install tree

# CentOS/RHEL
sudo yum install tree

# macOS
brew install tree
```

**Examples**:
```bash
# Basic tree
tree

# Limit depth to 2 levels
tree -L 2

# Show only directories
tree -d

# Show hidden files
tree -a

# Show with sizes and permissions
tree -phD

# Show disk usage
tree --du -h
```

**Output Example**:
```
.
├── directory1
│   ├── file1.txt
│   └── file2.txt
├── directory2
│   ├── subdirectory
│   │   └── file3.txt
│   └── file4.txt
└── file5.txt

3 directories, 5 files
```

---

## 4. Basic File Operations

### `touch` - Create/Update Files
**Description**: Creates empty files or updates timestamps.

**Syntax**:
```bash
touch [OPTIONS] FILE...
```

**Options**:
- `-a`: Change only access time
- `-m`: Change only modification time
- `-c`: Don't create file if it doesn't exist
- `-d STRING`: Use specified date/time
- `-t STAMP`: Use [[CC]YY]MMDDhhmm[.ss] format
- `-r FILE`: Use timestamps from reference file

**Examples**:
```bash
# Create empty file
touch newfile.txt

# Create multiple files
touch file1.txt file2.txt file3.txt

# Update modification time to current
touch existingfile.txt

# Set specific time
touch -d "2025-01-01 12:00:00" file.txt

# Use timestamp format
touch -t 202501011200 file.txt

# Copy timestamp from another file
touch -r reference.txt newfile.txt

# Don't create if doesn't exist
touch -c maynotexist.txt
```

**Use Cases**:
- Create placeholder files
- Reset file timestamps
- Test file operations
- Trigger time-based processes

---

### `mkdir` - Make Directories
**Description**: Creates new directories.

**Syntax**:
```bash
mkdir [OPTIONS] DIRECTORY...
```

**Options**:
- `-p`: Create parent directories as needed
- `-m MODE`: Set permissions (octal)
- `-v`: Verbose output

**Examples**:
```bash
# Create single directory
mkdir newdir

# Create multiple directories
mkdir dir1 dir2 dir3

# Create nested directories
mkdir -p project/src/components

# Create with specific permissions
mkdir -m 755 publicdir
mkdir -m 700 privatedir

# Verbose output
mkdir -pv parent/child/grandchild
# Output: mkdir: created directory 'parent'
#         mkdir: created directory 'parent/child'
#         mkdir: created directory 'parent/child/grandchild'
```

**Permission Modes**:
- `755`: rwxr-xr-x (owner full, others read+execute)
- `700`: rwx------ (owner only)
- `775`: rwxrwxr-x (owner+group full, others read+execute)

---

### `cp` - Copy Files and Directories
**Description**: Copies files and directories.

**Syntax**:
```bash
cp [OPTIONS] SOURCE DESTINATION
cp [OPTIONS] SOURCE... DIRECTORY
```

**Options**:
- `-r`, `-R`: Copy directories recursively
- `-i`: Interactive (prompt before overwrite)
- `-f`: Force (overwrite without prompting)
- `-v`: Verbose output
- `-p`: Preserve attributes (permissions, timestamps)
- `-a`: Archive mode (-dR --preserve=all)
- `-u`: Copy only when source is newer
- `-n`: No clobber (don't overwrite)
- `-l`: Hard link instead of copy
- `-s`: Symbolic link instead of copy
- `-b`: Backup before overwrite

**Examples**:
```bash
# Copy file
cp source.txt destination.txt

# Copy to directory
cp file.txt /path/to/directory/

# Copy multiple files to directory
cp file1.txt file2.txt file3.txt /dest/dir/

# Copy directory recursively
cp -r sourcedir/ destdir/

# Copy with confirmation
cp -i file.txt existing.txt
# Output: cp: overwrite 'existing.txt'? y/n

# Preserve attributes
cp -p original.txt copy.txt

# Archive copy (preserve everything)
cp -a sourcedir/ backup/

# Copy only if newer
cp -u newer.txt existing.txt

# Verbose copy
cp -rv directory/ backup/

# Create backup of existing file
cp -b file.txt existing.txt
# Creates: existing.txt~ (backup)

# Copy with pattern
cp *.txt /destination/
```

**Use Cases**:
- Backup files
- Duplicate directory structures
- Copy configuration files
- Distribute files across directories

---

### `mv` - Move/Rename Files
**Description**: Moves or renames files and directories.

**Syntax**:
```bash
mv [OPTIONS] SOURCE DESTINATION
mv [OPTIONS] SOURCE... DIRECTORY
```

**Options**:
- `-i`: Interactive (prompt before overwrite)
- `-f`: Force (overwrite without prompting)
- `-v`: Verbose output
- `-n`: No clobber (don't overwrite)
- `-u`: Move only when source is newer
- `-b`: Backup before overwrite

**Examples**:
```bash
# Rename file
mv oldname.txt newname.txt

# Move file to directory
mv file.txt /path/to/directory/

# Move multiple files
mv file1.txt file2.txt file3.txt /dest/dir/

# Rename directory
mv olddir newdir

# Move with confirmation
mv -i file.txt existing.txt

# Verbose move
mv -v *.txt /documents/

# Move only if newer
mv -u newfile.txt existing.txt

# Create backup
mv -b file.txt existing.txt

# Move and rename
mv /source/oldname.txt /dest/newname.txt
```

**Difference between `cp` and `mv`**:
- `cp`: Creates duplicate, source remains
- `mv`: Transfers file, source is removed

---

### `rm` - Remove Files and Directories
**Description**: Deletes files and directories.

**Syntax**:
```bash
rm [OPTIONS] FILE...
```

**Options**:
- `-r`, `-R`: Remove directories recursively
- `-f`: Force (ignore nonexistent files, no prompts)
- `-i`: Interactive (prompt for each file)
- `-v`: Verbose output
- `-d`: Remove empty directories
- `-I`: Prompt once for more than 3 files
- `--no-preserve-root`: Allow deletion of root directory (DANGEROUS!)

**Examples**:
```bash
# Remove file
rm file.txt

# Remove multiple files
rm file1.txt file2.txt file3.txt

# Remove with confirmation
rm -i important.txt
# Output: rm: remove regular file 'important.txt'? y/n

# Remove directory and contents
rm -r directory/

# Force remove without prompts
rm -rf directory/

# Verbose removal
rm -v *.log

# Remove with pattern
rm *.tmp
rm backup_*

# Safe removal of many files (prompts once)
rm -I *.txt
```

**⚠️ WARNING - DANGEROUS COMMANDS**:
```bash
# NEVER RUN THESE:
rm -rf /          # Deletes entire system
rm -rf /*         # Deletes everything in root
rm -rf ~          # Deletes entire home directory

# BE CAREFUL WITH:
rm -rf */         # Deletes all subdirectories in current location
rm -rf .*         # Deletes all hidden files/directories
```

**Safe Deletion Practices**:
```bash
# 1. Use -i for important operations
rm -i important.txt

# 2. Check what you're deleting first
ls *.log          # See what matches
rm *.log          # Then remove

# 3. Use trash utilities instead
trash file.txt    # Moves to trash (if trash-cli installed)

# 4. Double-check paths
pwd               # Verify location
ls -la            # See what's present
rm -rf directory/ # Then remove
```

---

### `rmdir` - Remove Empty Directories
**Description**: Removes empty directories only.

**Syntax**:
```bash
rmdir [OPTIONS] DIRECTORY...
```

**Options**:
- `-p`: Remove parent directories if empty
- `-v`: Verbose output
- `--ignore-fail-on-non-empty`: Don't error if directory not empty

**Examples**:
```bash
# Remove empty directory
rmdir emptydir

# Remove nested empty directories
rmdir -p parent/child/grandchild

# Verbose removal
rmdir -v olddir
```

**Note**: Use `rm -r` to remove directories with content.

---

### `ln` - Create Links
**Description**: Creates hard or symbolic links to files.

**Syntax**:
```bash
ln [OPTIONS] TARGET LINK_NAME
```

**Options**:
- `-s`: Create symbolic (soft) link
- `-f`: Force (remove existing destination)
- `-i`: Interactive (prompt before overwrite)
- `-v`: Verbose output
- `-b`: Backup existing files
- `-n`: Treat link destination as normal file if it's a symlink to a directory

**Hard Link vs Symbolic Link**:

**Hard Link**:
- References same inode as original file
- Cannot link directories
- Cannot cross filesystem boundaries
- Survives if original is deleted
- Same file, multiple names

**Symbolic Link (Symlink)**:
- Pointer to file path
- Can link directories
- Can cross filesystem boundaries
- Breaks if original is deleted
- Separate file pointing to original

**Examples**:
```bash
# Create hard link
ln original.txt hardlink.txt

# Create symbolic link
ln -s original.txt symlink.txt

# Create symlink to directory
ln -s /path/to/directory linkname

# Force create (overwrite if exists)
ln -sf target.txt link.txt

# Verbose linking
ln -sv /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/

# Create multiple hard links
ln file.txt link1.txt link2.txt

# Relative symlink
ln -s ../config.ini config_link.ini

# Absolute symlink
ln -s /usr/share/data /home/user/data
```

**Checking Links**:
```bash
# List with link information
ls -l
# Output: lrwxrwxrwx 1 user group 12 Dec 4 12:00 symlink.txt -> original.txt
#         ^                                                    └──────────────── target
#         └─ 'l' indicates symbolic link

# Find all links to a file
ls -li file.txt hardlink.txt
# Same inode number = hard links

# Show symlink target
readlink symlink.txt

# Show full path of symlink target
readlink -f symlink.txt
```

**Use Cases**:
- Create shortcuts to frequently accessed files/directories
- Maintain multiple versions/locations of same file
- Enable/disable configuration files (common in web servers)
- Save disk space with hard links

---

## 5. Viewing and Reading Files

### `cat` - Concatenate and Display Files
**Description**: Displays file contents, concatenates multiple files.

**Syntax**:
```bash
cat [OPTIONS] [FILE...]
```

**Options**:
- `-n`: Number all output lines
- `-b`: Number non-empty output lines
- `-s`: Squeeze multiple blank lines into one
- `-E`: Display $ at end of each line
- `-T`: Display TAB characters as ^I
- `-A`: Equivalent to -vET
- `-v`: Show non-printing characters

**Examples**:
```bash
# Display file contents
cat file.txt

# Display multiple files
cat file1.txt file2.txt

# Number all lines
cat -n file.txt
# Output:
#      1  Line one
#      2  Line two
#      3  Line three

# Number non-blank lines
cat -b file.txt

# Squeeze blank lines
cat -s file.txt

# Show end of lines
cat -E file.txt
# Output: This is a line$

# Concatenate files into new file
cat file1.txt file2.txt > combined.txt

# Append to existing file
cat file3.txt >> combined.txt

# Create file with here-document
cat > newfile.txt << EOF
Line 1
Line 2
Line 3
EOF

# Copy file using cat
cat source.txt > destination.txt
```

**Use Cases**:
- Quick view of file contents
- Combine multiple files
- Create simple text files
- Pipe content to other commands

---

### `less` - View Files Page by Page
**Description**: Allows backward and forward navigation through files.

**Syntax**:
```bash
less [OPTIONS] FILE
```

**Options**:
- `-N`: Show line numbers
- `-S`: Chop long lines (don't wrap)
- `-i`: Ignore case in searches
- `-I`: Ignore case in searches (smarter)
- `+F`: Follow mode (like tail -f)
- `-X`: Don't clear screen on exit

**Navigation Keys**:
- `Space` / `PageDown`: Forward one page
- `b` / `PageUp`: Backward one page
- `Enter` / `Down Arrow`: Forward one line
- `Up Arrow`: Backward one line
- `g`: Go to first line
- `G`: Go to last line
- `#g`: Go to line number #
- `/pattern`: Search forward
- `?pattern`: Search backward
- `n`: Next search match
- `N`: Previous search match
- `q`: Quit
- `h`: Help

**Examples**:
```bash
# View file
less file.txt

# View with line numbers
less -N file.txt

# View without line wrapping
less -S file.txt

# Case-insensitive search
less -i file.txt

# Follow file (like tail -f)
less +F /var/log/syslog

# View multiple files
less file1.txt file2.txt
# :n - next file
# :p - previous file

# Search in less
# Press / then type search term
/error
# Press n for next match
# Press N for previous match
```

**Use Cases**:
- Read large files efficiently
- Navigate log files
- Search through documentation
- Monitor growing files

---

### `more` - View Files Page by Page (Simple)
**Description**: Simpler pager, forward navigation only.

**Syntax**:
```bash
more [OPTIONS] FILE
```

**Options**:
- `-d`: Display help instead of beeping
- `-f`: Count logical lines
- `+#`: Start at line number #
- `+/pattern`: Start at first occurrence of pattern

**Navigation**:
- `Space`: Next page
- `Enter`: Next line
- `q`: Quit
- `/pattern`: Search
- `n`: Next search match
- `b`: Back one page (limited support)

**Examples**:
```bash
# View file
more file.txt

# Start at line 50
more +50 file.txt

# Start at first occurrence of "error"
more +/error logfile.txt
```

**Note**: `less` is more feature-rich than `more`.

---

### `head` - Display First Lines
**Description**: Shows the first N lines of files.

**Syntax**:
```bash
head [OPTIONS] [FILE...]
```

**Options**:
- `-n NUM`: Display first NUM lines (default: 10)
- `-c NUM`: Display first NUM bytes
- `-q`: Quiet (don't print filenames)
- `-v`: Verbose (always print filenames)

**Examples**:
```bash
# Show first 10 lines (default)
head file.txt

# Show first 20 lines
head -n 20 file.txt
head -20 file.txt      # Shorthand

# Show first 100 bytes
head -c 100 file.txt

# Multiple files
head file1.txt file2.txt
# Output:
# ==> file1.txt <==
# [first 10 lines]
#
# ==> file2.txt <==
# [first 10 lines]

# Pipe usage
cat largefile.txt | head -n 5

# Show all but last 5 lines
head -n -5 file.txt
```

**Use Cases**:
- Preview file contents
- Sample data files
- Check file format/headers
- Quick file inspection

---

### `tail` - Display Last Lines
**Description**: Shows the last N lines of files.

**Syntax**:
```bash
tail [OPTIONS] [FILE...]
```

**Options**:
- `-n NUM`: Display last NUM lines (default: 10)
- `-c NUM`: Display last NUM bytes
- `-f`: Follow (monitor file for new content)
- `-F`: Follow with retry (if file is renamed/rotated)
- `-q`: Quiet (don't print filenames)
- `-v`: Verbose (always print filenames)
- `--pid=PID`: With -f, terminate after process PID dies
- `-s SECONDS`: Sleep interval for -f (default: 1.0)

**Examples**:
```bash
# Show last 10 lines (default)
tail file.txt

# Show last 20 lines
tail -n 20 file.txt
tail -20 file.txt       # Shorthand

# Show last 100 bytes
tail -c 100 file.txt

# Follow file (real-time monitoring)
tail -f /var/log/syslog
# Press Ctrl+C to stop

# Follow with retry (handles log rotation)
tail -F /var/log/application.log

# Show all but first 10 lines
tail -n +10 file.txt

# Follow multiple files
tail -f /var/log/nginx/access.log /var/log/nginx/error.log

# Follow with specific interval
tail -f -s 0.5 fastlog.log

# Follow until process ends
tail -f --pid=12345 process.log
```

**Use Cases**:
- Monitor log files in real-time
- Debug applications
- Check recent entries
- Tail logs during deployments

---

### `wc` - Word Count
**Description**: Counts lines, words, bytes, and characters.

**Syntax**:
```bash
wc [OPTIONS] [FILE...]
```

**Options**:
- `-l`: Count lines only
- `-w`: Count words only
- `-c`: Count bytes only
- `-m`: Count characters only
- `-L`: Length of longest line

**Examples**:
```bash
# Count everything (lines, words, bytes)
wc file.txt
# Output: 42 156 892 file.txt
#         │   │   │   └─ filename
#         │   │   └───── bytes
#         │   └───────── words
#         └───────────── lines

# Count lines only
wc -l file.txt
# Output: 42 file.txt

# Count words only
wc -w file.txt

# Count bytes only
wc -c file.txt

# Count characters (different from bytes for UTF-8)
wc -m file.txt

# Multiple files (shows total)
wc -l file1.txt file2.txt file3.txt
# Output:
# 10 file1.txt
# 20 file2.txt
# 15 file3.txt
# 45 total

# Pipe usage
cat file.txt | wc -l

# Count files in directory
ls | wc -l

# Count running processes
ps aux | wc -l
```

**Use Cases**:
- Verify file line counts
- Check document length
- Count log entries
- Monitor process counts

---

## 6. Basic Text Editing

### `nano` - Simple Text Editor
**Description**: User-friendly terminal text editor.

**Syntax**:
```bash
nano [OPTIONS] [FILE]
```

**Options**:
- `-l`: Enable line numbers
- `-m`: Enable mouse support
- `-i`: Auto-indent new lines
- `-E`: Convert typed tabs to spaces
- `-w`: Disable line wrapping
- `+#`: Start at line number #
- `-B`: Backup file before saving

**Common Keyboard Shortcuts** (^ = Ctrl):
- `Ctrl+O`: Save (WriteOut)
- `Ctrl+X`: Exit
- `Ctrl+K`: Cut line
- `Ctrl+U`: Paste (UnCut)
- `Ctrl+W`: Search (Where Is)
- `Ctrl+\`: Search and replace
- `Ctrl+G`: Help
- `Ctrl+C`: Show cursor position
- `Ctrl+_`: Go to line number
- `Alt+U`: Undo
- `Alt+E`: Redo

**Examples**:
```bash
# Create/edit file
nano newfile.txt

# Edit with line numbers
nano -l file.txt

# Edit at specific line
nano +25 file.txt

# Edit with mouse support
nano -m file.txt

# Read-only mode
nano -v file.txt
```

**Basic Workflow**:
1. Open file: `nano file.txt`
2. Edit text normally
3. Save: Press `Ctrl+O`, confirm filename, press Enter
4. Exit: Press `Ctrl+X`

---

### `vim` / `vi` - Advanced Text Editor
**Description**: Powerful modal text editor (vi improved).

**Syntax**:
```bash
vim [OPTIONS] [FILE]
vi [OPTIONS] [FILE]
```

**Modes**:
1. **Normal Mode**: Command mode (default)
2. **Insert Mode**: Text editing
3. **Visual Mode**: Text selection
4. **Command-Line Mode**: Execute commands

**Entering Modes**:
- `i`: Insert mode (before cursor)
- `a`: Insert mode (after cursor)
- `I`: Insert mode (beginning of line)
- `A`: Insert mode (end of line)
- `o`: Insert mode (new line below)
- `O`: Insert mode (new line above)
- `Esc`: Return to normal mode
- `v`: Visual mode (character)
- `V`: Visual mode (line)
- `:`: Command-line mode

**Basic Commands (Normal Mode)**:
```
Navigation:
  h, j, k, l    - left, down, up, right
  w             - next word
  b             - previous word
  0             - beginning of line
  $             - end of line
  gg            - first line of file
  G             - last line of file
  #G            - go to line #

Editing:
  x             - delete character
  dd            - delete line
  yy            - copy line
  p             - paste after cursor
  P             - paste before cursor
  u             - undo
  Ctrl+r        - redo
  .             - repeat last command

Search:
  /pattern      - search forward
  ?pattern      - search backward
  n             - next match
  N             - previous match
  :%s/old/new/g - replace all

Save/Quit:
  :w            - save
  :q            - quit
  :wq           - save and quit
  :q!           - quit without saving
  :x            - save and quit (if modified)
  ZZ            - save and quit
  ZQ            - quit without saving
```

**Examples**:
```bash
# Edit file
vim file.txt

# Edit multiple files
vim file1.txt file2.txt file3.txt
# :n - next file
# :prev - previous file

# Open file at line 50
vim +50 file.txt

# Open file at first occurrence of "function"
vim +/function file.txt

# Open file in read-only mode
vim -R file.txt
view file.txt

# Diff two files
vim -d file1.txt file2.txt
vimdiff file1.txt file2.txt
```

**Quick Start**:
1. Open: `vim file.txt`
2. Press `i` to enter insert mode
3. Type your text
4. Press `Esc` to return to normal mode
5. Type `:wq` and press Enter to save and quit

**Configuration**: `~/.vimrc`

---

## 7. Getting Help

### `man` - Manual Pages
**Description**: Displays command documentation.

**Syntax**:
```bash
man [SECTION] COMMAND
```

**Manual Sections**:
- `1`: User commands
- `2`: System calls
- `3`: Library functions
- `4`: Device files
- `5`: File formats
- `6`: Games
- `7`: Miscellaneous
- `8`: System administration commands

**Navigation** (same as `less`):
- `Space`: Next page
- `b`: Previous page
- `/pattern`: Search
- `n`: Next search result
- `q`: Quit

**Examples**:
```bash
# View command manual
man ls

# View specific section
man 5 passwd      # passwd file format
man 1 passwd      # passwd command

# Search all manual pages
man -k keyword
apropos keyword   # Same as man -k

# Search in descriptions
man -K "search text"

# Show short description
whatis ls

# Show all available sections
man -wa ls
```

**Manual Page Structure**:
- NAME: Command name and brief description
- SYNOPSIS: Command syntax
- DESCRIPTION: Detailed explanation
- OPTIONS: Available options
- EXAMPLES: Usage examples
- FILES: Related files
- SEE ALSO: Related commands
- BUGS: Known issues
- AUTHOR: Author information

---

### `help` - Built-in Command Help
**Description**: Shows help for shell built-in commands.

**Syntax**:
```bash
help [COMMAND]
```

**Examples**:
```bash
# List all built-in commands
help

# Help for specific built-in
help cd
help echo
help export

# Bash help
bash --help
```

**Note**: Use `man` for external commands, `help` for shell built-ins.

---

### `--help` / `-h` Option
**Description**: Most commands support help options.

**Examples**:
```bash
# Long form
ls --help
grep --help
tar --help

# Short form
ls -h
grep -h
```

---

### `info` - Info Pages
**Description**: GNU info documentation system (more detailed than man).

**Syntax**:
```bash
info [COMMAND]
```

**Navigation**:
- `Space`: Next page
- `Backspace`: Previous page
- `n`: Next node
- `p`: Previous node
- `u`: Up one level
- `q`: Quit
- `Tab`: Next hyperlink
- `Enter`: Follow hyperlink

**Examples**:
```bash
# View info page
info ls
info coreutils

# Direct to specific node
info '(coreutils) ls invocation'
```

---

### `which` - Locate Command
**Description**: Shows full path of command executable.

**Syntax**:
```bash
which [OPTIONS] COMMAND...
```

**Options**:
- `-a`: Show all matching executables in PATH

**Examples**:
```bash
# Find command location
which ls
# Output: /usr/bin/ls

# Find multiple commands
which python python3 node

# Show all occurrences
which -a python
# Output: /usr/bin/python
#         /usr/local/bin/python
```

---

### `whereis` - Locate Binary, Source, Manual
**Description**: Finds binary, source, and manual page files.

**Syntax**:
```bash
whereis [OPTIONS] COMMAND
```

**Options**:
- `-b`: Search only for binaries
- `-m`: Search only for manual pages
- `-s`: Search only for sources

**Examples**:
```bash
# Find all locations
whereis ls
# Output: ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz

# Binary only
whereis -b python

# Manual pages only
whereis -m gcc
```

---

### `type` - Display Command Type
**Description**: Shows how a command would be interpreted.

**Syntax**:
```bash
type [OPTIONS] COMMAND...
```

**Options**:
- `-a`: Show all occurrences
- `-t`: Show type only
- `-p`: Show path of executable

**Examples**:
```bash
# Check command type
type cd
# Output: cd is a shell builtin

type ls
# Output: ls is aliased to `ls --color=auto'

type python
# Output: python is /usr/bin/python

# Show type only
type -t cd
# Output: builtin

# Show all occurrences
type -a python
```

**Command Types**:
- `alias`: Shell alias
- `keyword`: Shell keyword
- `function`: Shell function
- `builtin`: Shell built-in
- `file`: Disk file (executable)

---

# Part 2: Essential Skills (Intermediate Level)

## 8. File Permissions and Ownership

### Understanding Linux Permissions

**Permission Types**:
- **r** (read): View file contents / List directory contents
- **w** (write): Modify file / Create/delete files in directory
- **x** (execute): Run file as program / Access directory

**Permission Groups**:
- **User (u)**: File owner
- **Group (g)**: Group members
- **Others (o)**: Everyone else
- **All (a)**: All above

**Permission Representation**:
```
Symbolic:  rwxrwxrwx
Position:  uuugggooo
           ││││││└┴┴─ others
           │││└┴┴──── group
           └┴┴──────── user/owner

Octal:     421421421
           ││││││└┴┴─ others: 4(r) + 2(w) + 1(x)
           │││└┴┴──── group: 4(r) + 2(w) + 1(x)
           └┴┴──────── user: 4(r) + 2(w) + 1(x)
```

**Common Permission Values**:
```
0 (---) = No permissions
1 (--x) = Execute only
2 (-w-) = Write only
3 (-wx) = Write and execute
4 (r--) = Read only
5 (r-x) = Read and execute
6 (rw-) = Read and write
7 (rwx) = Read, write, and execute
```

**Common Permission Combinations**:
```
644 (rw-r--r--) = Owner can read/write, others can read (typical files)
755 (rwxr-xr-x) = Owner can do everything, others can read/execute (typical dirs/scripts)
777 (rwxrwxrwx) = Everyone can do everything (AVOID - security risk)
700 (rwx------) = Only owner has access (private files)
600 (rw-------) = Only owner can read/write (sensitive files)
```

---

### `chmod` - Change File Permissions
**Description**: Modifies file and directory permissions.

**Syntax**:
```bash
chmod [OPTIONS] MODE FILE...
```

**Options**:
- `-R`: Recursive (apply to directory and contents)
- `-v`: Verbose (show what's being changed)
- `-c`: Report only when changes are made
- `--reference=FILE`: Use permissions from reference file

**Symbolic Mode Syntax**:
```
chmod [ugoa][+-=][rwx] file

[ugoa]:
  u = user/owner
  g = group
  o = others
  a = all

[+-=]:
  + = add permission
  - = remove permission
  = = set exact permission

[rwx]:
  r = read
  w = write
  x = execute
```

**Examples**:
```bash
# Octal mode - set exact permissions
chmod 644 file.txt         # rw-r--r--
chmod 755 script.sh        # rwxr-xr-x
chmod 700 private.txt      # rwx------
chmod 600 secret.key       # rw-------

# Symbolic mode - add permissions
chmod u+x script.sh        # Add execute for user
chmod g+w file.txt         # Add write for group
chmod o+r document.txt     # Add read for others
chmod a+x program          # Add execute for all

# Symbolic mode - remove permissions
chmod u-x script.sh        # Remove execute from user
chmod g-w file.txt         # Remove write from group
chmod o-r secret.txt       # Remove read from others
chmod a-w readonly.txt     # Remove write from all

# Symbolic mode - set exact permissions
chmod u=rwx,g=rx,o=r file.txt  # rwxr-xr--
chmod u=rw,go=r file.txt       # rw-r--r--

# Multiple operations
chmod u+x,g-w,o-r file.txt

# Recursive
chmod -R 755 directory/

# Make script executable
chmod +x script.sh         # Shorthand for a+x

# Remove all permissions for others
chmod o-rwx file.txt
chmod o= file.txt          # Same result

# Copy permissions from another file
chmod --reference=file1.txt file2.txt

# Verbose output
chmod -v 644 *.txt
```

**Special Permissions**:
```bash
# Setuid (4000): Run as owner
chmod u+s executable
chmod 4755 executable       # rwsr-xr-x

# Setgid (2000): Inherit group, run as group
chmod g+s executable
chmod 2755 executable       # rwxr-sr-x
chmod g+s directory         # New files inherit group

# Sticky bit (1000): Only owner can delete (common for /tmp)
chmod +t directory
chmod 1755 directory        # rwxr-xr-t

# Combined special permissions
chmod 6755 file             # Setuid + Setgid
chmod 7755 directory        # All special permissions
```

**Use Cases**:
- Make scripts executable
- Secure sensitive files
- Set directory permissions
- Configure web server file permissions

---

### `chown` - Change File Owner
**Description**: Changes file owner and group.

**Syntax**:
```bash
chown [OPTIONS] [OWNER][:GROUP] FILE...
```

**Options**:
- `-R`: Recursive
- `-v`: Verbose
- `-c`: Report only when changes are made
- `--from=CURRENT_OWNER`: Change only if current owner matches
- `--reference=FILE`: Use owner/group from reference file

**Examples**:
```bash
# Change owner only
chown newowner file.txt

# Change owner and group
chown newowner:newgroup file.txt

# Change group only (colon required)
chown :newgroup file.txt

# Recursive change
chown -R username:groupname directory/

# Change owner, keep current group
chown username: file.txt

# Multiple files
chown user:group file1.txt file2.txt file3.txt

# Change only if current owner matches
chown --from=olduser newuser file.txt

# Copy ownership from another file
chown --reference=file1.txt file2.txt

# Verbose output
chown -v user:group *.txt

# Practical examples
sudo chown www-data:www-data /var/www/html/*
sudo chown -R $USER:$USER ~/myproject
```

**Note**: Usually requires root privileges (`sudo`).

---

### `chgrp` - Change File Group
**Description**: Changes file group ownership.

**Syntax**:
```bash
chgrp [OPTIONS] GROUP FILE...
```

**Options**:
- `-R`: Recursive
- `-v`: Verbose
- `-c`: Report only when changes are made
- `--reference=FILE`: Use group from reference file

**Examples**:
```bash
# Change group
chgrp developers file.txt

# Recursive change
chgrp -R webadmin /var/www/html

# Multiple files
chgrp staff *.txt

# Copy group from another file
chgrp --reference=file1.txt file2.txt

# Verbose output
chgrp -v developers project/
```

---

### `umask` - Default Permission Mask
**Description**: Sets default permissions for newly created files.

**Syntax**:
```bash
umask [OPTIONS] [MODE]
```

**How umask Works**:
```
Default permissions:
  Files: 666 (rw-rw-rw-)
  Directories: 777 (rwxrwxrwx)

umask value is SUBTRACTED:
  umask 022:
    Files: 666 - 022 = 644 (rw-r--r--)
    Directories: 777 - 022 = 755 (rwxr-xr-x)

  umask 077:
    Files: 666 - 077 = 600 (rw-------)
    Directories: 777 - 077 = 700 (rwx------)
```

**Examples**:
```bash
# View current umask
umask
# Output: 0022

# View in symbolic mode
umask -S
# Output: u=rwx,g=rx,o=rx

# Set umask (octal)
umask 022           # Standard: 644 files, 755 dirs
umask 077           # Restrictive: 600 files, 700 dirs
umask 002           # Group writable: 664 files, 775 dirs

# Set umask (symbolic)
umask u=rwx,g=rx,o=

# Make permanent (add to ~/.bashrc)
echo "umask 022" >> ~/.bashrc
```

**Common umask Values**:
```
022 = Files: 644, Dirs: 755 (standard)
027 = Files: 640, Dirs: 750 (group readable)
077 = Files: 600, Dirs: 700 (user only)
002 = Files: 664, Dirs: 775 (group writable)
```

---

### `stat` - Display File Status
**Description**: Shows detailed file information.

**Syntax**:
```bash
stat [OPTIONS] FILE...
```

**Options**:
- `-f`: Display filesystem status
- `-c FORMAT`: Custom format
- `-L`: Follow symbolic links
- `-t`: Terse mode

**Examples**:
```bash
# Full file information
stat file.txt
# Output:
#   File: file.txt
#   Size: 1234            Blocks: 8          IO Block: 4096   regular file
# Device: 801h/2049d      Inode: 123456      Links: 1
# Access: (0644/-rw-r--r--)  Uid: ( 1000/ user)   Gid: ( 1000/ user)
# Access: 2025-12-04 12:00:00.000000000 -0800
# Modify: 2025-12-04 12:00:00.000000000 -0800
# Change: 2025-12-04 12:00:00.000000000 -0800
#  Birth: -

# Filesystem information
stat -f /

# Custom format
stat -c "%n %s %a" file.txt
# Output: file.txt 1234 644

# Common format codes:
#   %n = filename
#   %s = size in bytes
#   %a = permissions (octal)
#   %A = permissions (human-readable)
#   %U = owner username
#   %G = group name
#   %y = modification time
```

---

### `getfacl` / `setfacl` - ACL Permissions
**Description**: Access Control Lists for fine-grained permissions.

**Syntax**:
```bash
getfacl [OPTIONS] FILE
setfacl [OPTIONS] MODE FILE
```

**Examples**:
```bash
# View ACL
getfacl file.txt

# Grant user specific permissions
setfacl -m u:username:rwx file.txt

# Grant group specific permissions
setfacl -m g:groupname:rx file.txt

# Remove ACL entry
setfacl -x u:username file.txt

# Remove all ACL
setfacl -b file.txt

# Recursive ACL
setfacl -R -m u:username:rwx directory/

# Copy ACL from another file
getfacl file1.txt | setfacl --set-file=- file2.txt
```

**Note**: ACLs require filesystem support (ext4, XFS, etc.).

---

## 9. Text Processing Commands

### `grep` - Search Text Patterns
**Description**: Searches for patterns in files using regular expressions.

**Syntax**:
```bash
grep [OPTIONS] PATTERN [FILE...]
```

**Common Options**:
- `-i`: Ignore case
- `-v`: Invert match (show non-matching lines)
- `-n`: Show line numbers
- `-c`: Count matches
- `-l`: Show filenames with matches
- `-L`: Show filenames without matches
- `-r`, `-R`: Recursive search
- `-w`: Match whole words
- `-x`: Match whole lines
- `-A NUM`: Show NUM lines after match
- `-B NUM`: Show NUM lines before match
- `-C NUM`: Show NUM lines context (before and after)
- `-o`: Show only matched parts
- `-q`: Quiet (no output, just exit status)
- `-E`: Extended regex (same as egrep)
- `-F`: Fixed strings (same as fgrep)
- `-P`: Perl regex
- `--color`: Colorize matches

**Examples**:
```bash
# Basic search
grep "error" logfile.txt

# Case-insensitive
grep -i "error" logfile.txt

# Show line numbers
grep -n "error" logfile.txt

# Count matches
grep -c "error" logfile.txt

# Invert match (lines without "error")
grep -v "error" logfile.txt

# Multiple files
grep "error" *.log

# Recursive search
grep -r "TODO" project/

# Recursive with line numbers
grep -rn "function" src/

# Whole word match
grep -w "log" file.txt       # Matches "log" but not "login"

# Multiple patterns
grep -e "error" -e "warning" logfile.txt
grep "error\|warning" logfile.txt    # Using regex OR

# Context lines
grep -A 3 "error" log.txt     # Show 3 lines after match
grep -B 2 "error" log.txt     # Show 2 lines before match
grep -C 2 "error" log.txt     # Show 2 lines before and after

# Files with matches
grep -l "error" *.log

# Files without matches
grep -L "error" *.log

# Show only matched text
grep -o "http://[^[:space:]]*" file.txt

# Extended regex
grep -E "error|warning|critical" log.txt

# Beginning of line
grep "^Error" log.txt

# End of line
grep "failed$" log.txt

# Pipe usage
ps aux | grep "nginx"
cat file.txt | grep -i "search term"

# Exclude directories
grep -r --exclude-dir=node_modules "pattern" .

# Exclude file patterns
grep -r --exclude="*.log" "pattern" .

# Binary files
grep -a "text" binaryfile     # Treat as text
grep -I "text" *              # Skip binary files

# Count non-empty lines
grep -c "." file.txt

# Lines with numbers
grep "[0-9]" file.txt

# Email addresses
grep -E "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" file.txt

# IP addresses
grep -E "([0-9]{1,3}\.){3}[0-9]{1,3}" file.txt
```

**Regular Expression Basics**:
```
.           # Any single character
*           # Zero or more of previous
+           # One or more of previous (extended regex)
?           # Zero or one of previous (extended regex)
^           # Beginning of line
$           # End of line
[abc]       # Any character in brackets
[^abc]      # Any character not in brackets
[a-z]       # Range of characters
\           # Escape special character
|           # OR (extended regex)
()          # Grouping (extended regex)
\b          # Word boundary
\d          # Digit (Perl regex)
\w          # Word character (Perl regex)
\s          # Whitespace (Perl regex)
```

**Use Cases**:
- Search log files for errors
- Find code patterns
- Filter command output
- Validate file contents

---

### `sed` - Stream Editor
**Description**: Performs text transformations on streams.

**Syntax**:
```bash
sed [OPTIONS] 'COMMAND' [FILE...]
```

**Common Options**:
- `-i`: Edit files in place
- `-i.bak`: Edit in place with backup
- `-e`: Multiple commands
- `-n`: Suppress default output
- `-r`, `-E`: Extended regex

**Common Commands**:
- `s/pattern/replacement/`: Substitute
- `d`: Delete line
- `p`: Print line
- `a\text`: Append text
- `i\text`: Insert text
- `c\text`: Replace line

**Examples**:
```bash
# Basic substitution (first occurrence per line)
sed 's/old/new/' file.txt

# Global substitution (all occurrences)
sed 's/old/new/g' file.txt

# Case-insensitive substitution
sed 's/old/new/gi' file.txt

# Edit file in place
sed -i 's/old/new/g' file.txt

# Edit with backup
sed -i.bak 's/old/new/g' file.txt

# Delete lines containing pattern
sed '/pattern/d' file.txt

# Delete empty lines
sed '/^$/d' file.txt

# Delete lines 5-10
sed '5,10d' file.txt

# Print only matching lines
sed -n '/pattern/p' file.txt

# Print specific lines
sed -n '5p' file.txt          # Line 5
sed -n '5,10p' file.txt       # Lines 5-10
sed -n '5,$p' file.txt        # Line 5 to end

# Replace on specific line
sed '3s/old/new/' file.txt

# Replace in line range
sed '1,5s/old/new/g' file.txt

# Multiple commands
sed -e 's/one/1/g' -e 's/two/2/g' file.txt
sed 's/one/1/g; s/two/2/g' file.txt

# Append text after line
sed '/pattern/a\New line here' file.txt

# Insert text before line
sed '/pattern/i\New line here' file.txt

# Replace entire line
sed '/pattern/c\Replacement line' file.txt

# Add line numbers
sed = file.txt | sed 'N;s/\n/\t/'

# Remove comments (lines starting with #)
sed '/^#/d' file.txt

# Remove trailing whitespace
sed 's/[[:space:]]*$//' file.txt

# Remove leading whitespace
sed 's/^[[:space:]]*//' file.txt

# Double-space file
sed G file.txt

# Convert DOS to Unix line endings
sed 's/\r$//' dosfile.txt > unixfile.txt

# Extract IP addresses
sed -n 's/.*\([0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\).*/\1/p' file.txt

# Replace between specific patterns
sed '/START/,/END/s/old/new/g' file.txt

# Use different delimiter (when pattern contains /)
sed 's|/old/path|/new/path|g' file.txt

# Backreferences
sed 's/\([0-9]*\)-\([0-9]*\)/\2-\1/' file.txt  # Swap numbers around dash

# Multiple files
sed -i 's/old/new/g' file1.txt file2.txt file3.txt
```

**Use Cases**:
- Find and replace in files
- Filter and transform log files
- Process configuration files
- Batch file modifications

---

### `awk` - Pattern Scanning and Processing
**Description**: Powerful text processing and data extraction tool.

**Syntax**:
```bash
awk [OPTIONS] 'PROGRAM' [FILE...]
```

**Built-in Variables**:
- `$0`: Entire line
- `$1, $2, ...`: First field, second field, etc.
- `NF`: Number of fields in current line
- `NR`: Current line number
- `FS`: Field separator (default: whitespace)
- `OFS`: Output field separator (default: space)
- `RS`: Record separator (default: newline)
- `ORS`: Output record separator (default: newline)
- `FILENAME`: Current filename

**Examples**:
```bash
# Print entire file
awk '{print}' file.txt
awk '{print $0}' file.txt

# Print specific fields
awk '{print $1}' file.txt           # First field
awk '{print $1, $3}' file.txt       # First and third fields
awk '{print $NF}' file.txt          # Last field
awk '{print $(NF-1)}' file.txt      # Second to last field

# Custom separator
awk -F: '{print $1}' /etc/passwd    # Print usernames
awk -F',' '{print $1, $2}' data.csv # CSV data

# Multiple separators
awk -F'[,:]' '{print $1}' file.txt

# Output field separator
awk -F: 'OFS="\t" {print $1, $3}' /etc/passwd

# Pattern matching
awk '/pattern/' file.txt                    # Lines containing pattern
awk '/pattern/ {print $1}' file.txt         # First field of matching lines
awk '!/pattern/' file.txt                   # Lines NOT containing pattern

# Conditional
awk '$3 > 100' file.txt                     # Lines where field 3 > 100
awk '$1 == "root"' file.txt                 # Lines where field 1 equals "root"
awk '$2 ~ /pattern/' file.txt               # Lines where field 2 matches pattern

# Line numbers
awk '{print NR, $0}' file.txt               # Add line numbers
awk 'NR==5' file.txt                        # Print line 5
awk 'NR>=5 && NR<=10' file.txt              # Print lines 5-10

# BEGIN and END
awk 'BEGIN {print "Start"} {print} END {print "Done"}' file.txt

# Calculations
awk '{sum += $1} END {print sum}' numbers.txt           # Sum first column
awk '{sum += $1; count++} END {print sum/count}' file.txt   # Average

# Count lines
awk 'END {print NR}' file.txt

# Count fields
awk '{print NF}' file.txt

# Print lines with more than 5 fields
awk 'NF > 5' file.txt

# Format output
awk '{printf "%-10s %-10s\n", $1, $2}' file.txt

# Remove duplicate lines (preserving order)
awk '!seen[$0]++' file.txt

# Combine lines
awk 'ORS=NR%3?",":"\n"' file.txt    # Join every 3 lines with comma

# Process multiple files
awk '{print FILENAME, $0}' file1.txt file2.txt

# Field manipulation
awk '{$2 = $2 * 2; print}' file.txt    # Multiply field 2 by 2

# Length of line
awk 'length($0) > 80' file.txt         # Lines longer than 80 characters

# String functions
awk '{print toupper($1)}' file.txt     # Convert to uppercase
awk '{print tolower($1)}' file.txt     # Convert to lowercase
awk '{print substr($1, 1, 5)}' file.txt # First 5 characters

# Process log files
awk '$9 == 404 {print $1, $7}' access.log       # Find 404 errors
awk '{bytes += $10} END {print bytes}' access.log   # Total bytes

# CSV processing
awk -F',' '{print $1, $3}' data.csv
awk -F',' 'NR>1 {sum+=$2} END {print sum}' data.csv  # Skip header

# Complex conditions
awk '$3 > 100 && $4 < 200 {print $0}' file.txt
awk '$1 == "ERROR" || $1 == "CRITICAL" {print}' log.txt
```

**Use Cases**:
- Extract columns from structured data
- Process CSV/TSV files
- Generate reports from logs
- Calculate statistics

---

### `cut` - Extract Columns
**Description**: Cuts out sections from each line of files.

**Syntax**:
```bash
cut [OPTIONS] [FILE...]
```

**Options**:
- `-f LIST`: Select fields
- `-d DELIM`: Field delimiter (default: TAB)
- `-c LIST`: Select characters
- `-b LIST`: Select bytes
- `--complement`: Invert selection
- `-s`: Don't print lines without delimiter

**List Format**:
- `N`: Field/character N
- `N-M`: Range from N to M
- `N-`: From N to end
- `-M`: From beginning to M
- `N,M`: Fields N and M

**Examples**:
```bash
# Extract first field (default TAB delimiter)
cut -f1 file.txt

# Extract fields 1 and 3
cut -f1,3 file.txt

# Extract field range
cut -f1-3 file.txt         # Fields 1 to 3
cut -f3- file.txt          # Field 3 to end
cut -f-3 file.txt          # Beginning to field 3

# Custom delimiter
cut -d':' -f1 /etc/passwd  # Extract usernames
cut -d',' -f2,4 data.csv   # CSV fields

# Extract characters
cut -c1-10 file.txt        # First 10 characters
cut -c1,5,10 file.txt      # Characters 1, 5, and 10

# Complement (everything except)
cut -d':' -f1 --complement /etc/passwd

# Skip lines without delimiter
cut -d':' -f1 -s file.txt

# Pipe usage
echo "John:Doe:30:Engineer" | cut -d':' -f1,3
# Output: John:30

# Process command output
ls -l | cut -c1-10         # First 10 chars of ls output
df -h | cut -d' ' -f1,5    # Disk usage columns
```

**Use Cases**:
- Extract columns from delimited files
- Parse structured text output
- Process CSV/TSV data
- Quick data extraction

---

### `sort` - Sort Lines
**Description**: Sorts lines of text files.

**Syntax**:
```bash
sort [OPTIONS] [FILE...]
```

**Options**:
- `-r`: Reverse order
- `-n`: Numeric sort
- `-h`: Human-numeric sort (1K, 1M, 1G)
- `-k FIELD`: Sort by field number
- `-t DELIM`: Field delimiter
- `-u`: Unique (remove duplicates)
- `-f`: Ignore case
- `-b`: Ignore leading blanks
- `-V`: Version sort
- `-R`: Random sort
- `-c`: Check if sorted
- `-m`: Merge already sorted files
- `-o FILE`: Output to file

**Examples**:
```bash
# Simple alphabetic sort
sort file.txt

# Reverse sort
sort -r file.txt

# Numeric sort
sort -n numbers.txt

# Human-readable numeric sort
sort -h sizes.txt
# Input: 1K, 500, 1M, 2G
# Output: 500, 1K, 1M, 2G

# Remove duplicates while sorting
sort -u file.txt

# Case-insensitive sort
sort -f file.txt

# Sort by specific field
sort -k2 file.txt          # Sort by 2nd field
sort -k2,2 file.txt        # Sort by 2nd field only
sort -k2n file.txt         # Sort by 2nd field numerically

# Custom delimiter and field
sort -t':' -k3n /etc/passwd    # Sort by UID

# Multiple sort keys
sort -k1,1 -k2n file.txt   # Sort by field 1, then by field 2 numerically

# Sort by field range
sort -k2,4 file.txt        # Sort by fields 2-4

# Version sort (for version numbers)
sort -V versions.txt
# Input: v1.10, v1.2, v1.1
# Output: v1.1, v1.2, v1.10

# Random sort (shuffle)
sort -R file.txt

# Check if file is sorted
sort -c file.txt
# Output: sort: file.txt:3: disorder: line content

# Output to file
sort file.txt -o sorted.txt

# Sort in place (overwrites original)
sort file.txt -o file.txt

# Merge sorted files
sort -m sorted1.txt sorted2.txt

# Sort by size (from ls -lh)
ls -lh | sort -k5h

# Sort by multiple criteria
ps aux | sort -k3nr | head -10    # Top 10 CPU users
```

**Use Cases**:
- Organize data alphabetically or numerically
- Remove duplicates from files
- Prepare data for join or comm commands
- Sort log files by timestamp

---

### `uniq` - Remove Duplicate Lines
**Description**: Filters out repeated adjacent lines.

**Syntax**:
```bash
uniq [OPTIONS] [INPUT [OUTPUT]]
```

**Options**:
- `-c`: Count occurrences
- `-d`: Only show duplicate lines
- `-u`: Only show unique lines
- `-i`: Ignore case
- `-f N`: Skip first N fields
- `-s N`: Skip first N characters
- `-w N`: Compare only first N characters

**Important**: `uniq` only removes **adjacent** duplicates. Usually used with `sort`.

**Examples**:
```bash
# Remove duplicate lines (must be adjacent)
uniq file.txt

# Remove duplicates from anywhere in file (sort first)
sort file.txt | uniq

# Count occurrences
uniq -c file.txt
# Output:
#   3 apple
#   2 banana
#   1 orange

# Show only duplicates
uniq -d file.txt

# Show only unique lines
uniq -u file.txt

# Case-insensitive
uniq -i file.txt

# Sort and remove duplicates with count
sort file.txt | uniq -c

# Find most common lines
sort file.txt | uniq -c | sort -rn | head

# Skip first field when comparing
uniq -f1 file.txt

# Skip first 5 characters
uniq -s5 file.txt

# Compare only first 10 characters
uniq -w10 file.txt

# Remove duplicates and save
sort file.txt | uniq > unique.txt

# Find duplicate entries in log
sort access.log | uniq -d

# Count unique IPs
cut -d' ' -f1 access.log | sort | uniq -c | sort -rn
```

**Use Cases**:
- Remove duplicate entries
- Count occurrences
- Find unique/duplicate records
- Analyze log files

---

### `tr` - Translate Characters
**Description**: Translates or deletes characters.

**Syntax**:
```bash
tr [OPTIONS] SET1 [SET2]
```

**Options**:
- `-d`: Delete characters in SET1
- `-s`: Squeeze repeated characters
- `-c`: Complement SET1 (use all characters except SET1)
- `-t`: Truncate SET1 to length of SET2

**Character Sets**:
- `[:alnum:]`: Alphanumeric
- `[:alpha:]`: Alphabetic
- `[:digit:]`: Digits
- `[:lower:]`: Lowercase
- `[:upper:]`: Uppercase
- `[:space:]`: Whitespace
- `[:punct:]`: Punctuation

**Examples**:
```bash
# Convert lowercase to uppercase
echo "hello world" | tr 'a-z' 'A-Z'
# Output: HELLO WORLD

tr '[:lower:]' '[:upper:]' < file.txt

# Convert uppercase to lowercase
echo "HELLO" | tr 'A-Z' 'a-z'

# Delete specific characters
echo "Hello, World!" | tr -d ',!'
# Output: Hello World

# Delete digits
echo "Room 123" | tr -d '[:digit:]'
# Output: Room

# Delete all non-alphanumeric
echo "Hello@#$World123" | tr -cd '[:alnum:]'
# Output: HelloWorld123

# Squeeze repeated characters
echo "Hello    World" | tr -s ' '
# Output: Hello World

# Replace spaces with newlines
echo "one two three" | tr ' ' '\n'
# Output:
# one
# two
# three

# Remove all whitespace
echo "  hello  world  " | tr -d '[:space:]'
# Output: helloworld

# Replace multiple characters
echo "hello" | tr 'helo' 'HELO'
# Output: HELLO

# ROT13 encoding
echo "hello" | tr 'A-Za-z' 'N-ZA-Mn-za-m'

# Convert DOS to Unix line endings
tr -d '\r' < dosfile.txt > unixfile.txt

# Remove all except printable ASCII
tr -cd '[:print:]' < file.txt

# Extract numbers from text
echo "Price: $123.45" | tr -cd '[:digit:].'
# Output: 123.45
```

**Use Cases**:
- Case conversion
- Character replacement
- Remove unwanted characters
- Format text data

---

### `wc` - Word, Line, Character Count
**Description**: Already covered in Part 1, but here are advanced uses.

**Advanced Examples**:
```bash
# Count files in directory
ls | wc -l

# Count code lines (excluding blanks)
grep -v '^$' *.js | wc -l

# Count unique lines
sort file.txt | uniq | wc -l

# Count words in all files
wc -w *.txt

# Find longest line
wc -L file.txt

# Statistics for multiple files
wc *.txt
# Shows totals at the end
```

---

### `diff` - Compare Files
**Description**: Compares files line by line.

**Syntax**:
```bash
diff [OPTIONS] FILE1 FILE2
```

**Options**:
- `-u`: Unified format
- `-c`: Context format
- `-y`: Side-by-side
- `-q`: Brief (only report if different)
- `-r`: Recursive (directories)
- `-i`: Ignore case
- `-w`: Ignore whitespace
- `-b`: Ignore changes in whitespace amount
- `-B`: Ignore blank lines
- `--color`: Colorize output

**Examples**:
```bash
# Basic diff
diff file1.txt file2.txt
# Output:
# 2c2
# < old line
# ---
# > new line

# Unified format (common in patches)
diff -u file1.txt file2.txt
# Output:
# --- file1.txt
# +++ file2.txt
# @@ -1,3 +1,3 @@
#  line 1
# -old line
# +new line
#  line 3

# Context format
diff -c file1.txt file2.txt

# Side-by-side comparison
diff -y file1.txt file2.txt

# Brief (just report difference)
diff -q file1.txt file2.txt
# Output: Files file1.txt and file2.txt differ

# Ignore case
diff -i file1.txt file2.txt

# Ignore whitespace
diff -w file1.txt file2.txt

# Ignore blank lines
diff -B file1.txt file2.txt

# Compare directories
diff -r dir1/ dir2/

# Create patch file
diff -u original.txt modified.txt > changes.patch

# Colorized output
diff --color file1.txt file2.txt

# Exclude files from directory comparison
diff -r --exclude='*.log' dir1/ dir2/
```

**Understanding Output**:
```
Symbols:
  a = add
  c = change
  d = delete
  < = from first file
  > = from second file

"5c5" means:
  Line 5 in file1 differs from line 5 in file2

"5,7c5,9" means:
  Lines 5-7 in file1 differ from lines 5-9 in file2
```

---

### `comm` - Compare Sorted Files
**Description**: Compares two sorted files line by line.

**Syntax**:
```bash
comm [OPTIONS] FILE1 FILE2
```

**Output Columns**:
1. Lines unique to FILE1
2. Lines unique to FILE2
3. Lines common to both

**Options**:
- `-1`: Suppress column 1
- `-2`: Suppress column 2
- `-3`: Suppress column 3
- `-12`: Show only common lines
- `-23`: Show only lines unique to FILE1
- `-13`: Show only lines unique to FILE2

**Examples**:
```bash
# Basic comparison (files must be sorted)
comm file1.txt file2.txt

# Show only common lines
comm -12 file1.txt file2.txt

# Show only lines unique to first file
comm -23 file1.txt file2.txt

# Show only lines unique to second file
comm -13 file1.txt file2.txt

# Compare after sorting
comm <(sort file1.txt) <(sort file2.txt)

# Find common users
comm -12 <(cut -d: -f1 /etc/passwd | sort) <(cut -d: -f1 backup/passwd | sort)
```

**Use Cases**:
- Find common/unique elements
- Compare lists
- Set operations on text files

---

## 10. Search and Find Commands

### `find` - Search for Files
**Description**: Searches for files in directory hierarchy.

**Syntax**:
```bash
find [PATH...] [EXPRESSION]
```

**Common Tests**:
- `-name PATTERN`: Match filename
- `-iname PATTERN`: Case-insensitive name match
- `-type TYPE`: File type (f=file, d=directory, l=symlink)
- `-size N`: File size
- `-mtime N`: Modified N days ago
- `-atime N`: Accessed N days ago
- `-ctime N`: Changed N days ago
- `-mmin N`: Modified N minutes ago
- `-perm MODE`: Permission match
- `-user USER`: Owned by user
- `-group GROUP`: Owned by group
- `-empty`: Empty files/directories
- `-executable`: Executable files

**Common Actions**:
- `-print`: Print filename (default)
- `-print0`: Print with null separator
- `-ls`: List in ls -l format
- `-delete`: Delete matched files
- `-exec COMMAND {} \;`: Execute command
- `-execdir COMMAND {} \;`: Execute in file's directory
- `-ok COMMAND {} \;`: Execute with confirmation

**Examples**:
```bash
# Find by name
find /path -name "filename.txt"
find . -name "*.js"
find /home -iname "*.JPG"    # Case-insensitive

# Find by type
find . -type f               # Files only
find . -type d               # Directories only
find . -type l               # Symbolic links

# Find by size
find . -size +100M           # Larger than 100MB
find . -size -1k             # Smaller than 1KB
find . -size 50M             # Exactly 50MB
find . -size +1G -size -2G   # Between 1GB and 2GB

# Size units: c=bytes, k=KB, M=MB, G=GB

# Find by modification time
find . -mtime -7             # Modified in last 7 days
find . -mtime +30            # Modified more than 30 days ago
find . -mtime 0              # Modified today
find . -mmin -60             # Modified in last 60 minutes

# Find by access time
find . -atime -1             # Accessed in last day

# Find by permissions
find . -perm 644             # Exactly 644
find . -perm -644            # At least 644
find . -perm /u+w            # User writable

# Find by owner
find . -user username
find . -group groupname
find / -user root -type f

# Find empty files/directories
find . -empty
find . -type f -empty        # Empty files
find . -type d -empty        # Empty directories

# Find executable files
find . -type f -executable

# Combine conditions (AND)
find . -type f -name "*.log" -size +10M

# OR conditions
find . -name "*.txt" -o -name "*.md"
find . \( -name "*.jpg" -o -name "*.png" \)

# NOT condition
find . -not -name "*.txt"
find . ! -name "*.log"

# Execute commands on found files
find . -name "*.log" -exec rm {} \;
find . -type f -exec chmod 644 {} \;
find . -name "*.jpg" -exec cp {} /backup/ \;

# Execute with confirmation
find . -name "*.tmp" -ok rm {} \;

# Execute for multiple files at once (faster)
find . -name "*.log" -exec rm {} +

# Execute in file's directory
find . -type f -execdir echo "Found: {}" \;

# Delete found files
find . -name "*.tmp" -delete
find . -type f -empty -delete

# Find and list details
find . -name "*.conf" -ls

# Find and count
find . -type f | wc -l

# Find recently modified files
find . -type f -mtime -1 -ls

# Find large files
find / -type f -size +500M -exec ls -lh {} \; 2>/dev/null

# Find by multiple extensions
find . -type f \( -name "*.js" -o -name "*.ts" -o -name "*.jsx" \)

# Find and archive
find . -name "*.log" -print0 | xargs -0 tar czf logs.tar.gz

# Find files modified more recently than another file
find . -newer reference.txt

# Find broken symbolic links
find . -type l ! -exec test -e {} \; -print

# Exclude directories
find . -path ./node_modules -prune -o -name "*.js" -print
find . -type f -not -path "*/node_modules/*" -name "*.js"

# Find files with specific permission issues
find . -type f -perm /o+w     # World-writable files (security risk)

# Find and show file age
find . -type f -printf "%T+ %p\n" | sort

# Complex example: Find and process
find /var/log -name "*.log" -type f -mtime +30 -exec gzip {} \;
```

**Use Cases**:
- Locate files by various criteria
- Clean up old or large files
- Bulk file operations
- System maintenance

---

### `locate` - Find Files by Name (Fast)
**Description**: Quickly finds files using pre-built database.

**Syntax**:
```bash
locate [OPTIONS] PATTERN
```

**Options**:
- `-i`: Ignore case
- `-c`: Count matches
- `-l NUM`: Limit output to NUM entries
- `-r`: Use regex
- `-b`: Match only basename
- `-e`: Only show existing files

**Examples**:
```bash
# Basic search
locate filename

# Case-insensitive
locate -i readme.txt

# Count matches
locate -c "*.pdf"

# Limit results
locate -l 10 *.jpg

# Basename only
locate -b '\*.conf'

# Use regex
locate -r '/etc/.*\.conf$'

# Update database (run as root)
sudo updatedb

# Only existing files
locate -e filename
```

**Advantages**:
- Much faster than find
- Searches entire filesystem

**Disadvantages**:
- Requires updated database
- Less flexible than find
- Won't find very recent files

**Database Update**:
```bash
# Manual update (as root)
sudo updatedb

# Database location: /var/lib/mlocate/mlocate.db
# Update frequency: Usually daily via cron
```

---

### `which` - Locate Command
**Description**: Already covered in Part 1. Shows executable location in PATH.

---

### `whereis` - Locate Binary, Source, Manual
**Description**: Already covered in Part 1.

---

## 11. Process Management

### Understanding Processes

**Process States**:
- `R`: Running
- `S`: Sleeping (interruptible)
- `D`: Sleeping (uninterruptible, usually I/O)
- `T`: Stopped
- `Z`: Zombie (terminated but not reaped)

**Process Hierarchy**:
- **PID**: Process ID (unique identifier)
- **PPID**: Parent Process ID
- **Init/Systemd**: PID 1, ancestor of all processes

---

### `ps` - Process Status
**Description**: Shows information about running processes.

**Syntax**:
```bash
ps [OPTIONS]
```

**Common Option Sets**:
- `ps`: Show processes for current shell
- `ps aux`: Show all processes (BSD style)
- `ps -ef`: Show all processes (Unix style)
- `ps -u USER`: Show processes for specific user

**Options**:
- `a`: All users
- `u`: User-oriented format
- `x`: Include processes without controlling terminal
- `-e`, `-A`: All processes
- `-f`: Full format
- `-l`: Long format
- `--forest`: ASCII art process tree
- `-p PID`: Specific process
- `-C COMMAND`: Processes by command name

**Examples**:
```bash
# Current shell processes
ps

# All processes (BSD style)
ps aux
# Output columns:
# USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND

# All processes (Unix style)
ps -ef
# Output columns:
# UID PID PPID C STIME TTY TIME CMD

# User's processes
ps -u username

# Long format
ps -l

# Process tree
ps -ef --forest
ps auxf

# Specific process by PID
ps -p 1234
ps -p 1234 -f

# Processes by command name
ps -C nginx
ps -C "node"

# Custom output format
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu
ps -eo user,pid,stat,command

# Top CPU consumers
ps aux --sort=-%cpu | head -10

# Top memory consumers
ps aux --sort=-%mem | head -10

# Threads
ps -eLf
ps auxm

# Wide output (don't truncate)
ps auxww

# Watch processes
watch -n 1 'ps aux --sort=-%cpu | head -20'

# Filter output
ps aux | grep "nginx"
ps aux | grep -v grep | grep "nginx"
ps aux | awk '$3 > 50'    # CPU > 50%

# Count processes
ps aux | wc -l

# Process by TTY
ps -t tty1

# Zombie processes
ps aux | grep 'Z'
```

**Understanding ps aux Output**:
```
USER       = Owner of the process
PID        = Process ID
%CPU       = CPU usage percentage
%MEM       = Memory usage percentage
VSZ        = Virtual memory size (KB)
RSS        = Resident set size (physical memory, KB)
TTY        = Controlling terminal (? = no terminal)
STAT       = Process state
  R = Running
  S = Sleeping
  D = Uninterruptible sleep
  T = Stopped
  Z = Zombie
  < = High priority
  N = Low priority
  L = Has pages locked in memory
  s = Session leader
  + = Foreground process group
START      = Start time
TIME       = Cumulative CPU time
COMMAND    = Command with arguments
```

---

### `top` - Dynamic Process Viewer
**Description**: Real-time process monitoring.

**Syntax**:
```bash
top [OPTIONS]
```

**Common Options**:
- `-d SECONDS`: Update delay
- `-u USER`: Show user's processes
- `-p PID`: Monitor specific PIDs
- `-H`: Show threads
- `-b`: Batch mode (for scripts)
- `-n NUM`: Number of iterations

**Interactive Commands** (while running):
```
h or ?     = Help
q          = Quit
Space      = Refresh
k          = Kill process (prompts for PID)
r          = Renice process
M          = Sort by memory
P          = Sort by CPU (default)
T          = Sort by time
c          = Show full command path
V          = Forest view (process tree)
u          = Filter by user
1          = Show individual CPUs
f          = Choose display fields
W          = Save configuration
```

**Examples**:
```bash
# Basic top
top

# Update every 2 seconds
top -d 2

# Show specific user
top -u username

# Monitor specific process
top -p 1234
top -p 1234,5678,9012    # Multiple PIDs

# Show threads
top -H
top -H -p 1234

# Batch mode (one iteration)
top -b -n 1

# Batch mode to file
top -b -n 1 > top-output.txt

# Monitor and sort by memory
top -o %MEM

# Color output
top -c
```

**Top Display Sections**:
```
Header:
  Uptime, load average, tasks, CPU states, memory, swap

Columns:
  PID     = Process ID
  USER    = Owner
  PR      = Priority
  NI      = Nice value
  VIRT    = Virtual memory
  RES     = Resident memory
  SHR     = Shared memory
  S       = Status
  %CPU    = CPU usage
  %MEM    = Memory usage
  TIME+   = CPU time
  COMMAND = Command name
```

---

### `htop` - Enhanced Process Viewer
**Description**: User-friendly alternative to top with better interface.

**Installation**:
```bash
# Ubuntu/Debian
sudo apt install htop

# CentOS/RHEL
sudo yum install htop

# macOS
brew install htop
```

**Features**:
- Color-coded display
- Mouse support
- Tree view
- Easy process management

**Interactive Commands**:
```
F1/h       = Help
F2         = Setup (configuration)
F3//       = Search
F4/\       = Filter
F5/t       = Tree view
F6/>       = Sort by column
F9/k       = Kill process
F10/q      = Quit
Space      = Tag process
U          = Untag all
c          = Tag process and children
Shift+M    = Sort by memory
Shift+P    = Sort by CPU
Shift+T    = Sort by time
```

**Examples**:
```bash
# Launch htop
htop

# Show specific user
htop -u username

# Show process tree
htop -t

# Show specific PIDs
htop -p 1234,5678
```

---

### `pgrep` - Find Process by Name
**Description**: Searches for processes by name or attributes.

**Syntax**:
```bash
pgrep [OPTIONS] PATTERN
```

**Options**:
- `-l`: List process name with PID
- `-a`: List full command line
- `-u USER`: Match user
- `-x`: Exact match
- `-f`: Match full command line
- `-n`: Newest process only
- `-o`: Oldest process only
- `-c`: Count matches

**Examples**:
```bash
# Find process by name
pgrep nginx
# Output: 1234\n5678

# Show name and PID
pgrep -l nginx
# Output: 1234 nginx\n5678 nginx

# Show full command
pgrep -a nginx
# Output: 1234 nginx: master process

# User's processes
pgrep -u username

# Exact match
pgrep -x nginx

# Count processes
pgrep -c nginx

# Newest process
pgrep -n firefox

# Full command line match
pgrep -f "node server.js"

# Combine with other commands
kill $(pgrep processname)
kill -9 $(pgrep -u username firefox)
```

---

### `pkill` - Kill Process by Name
**Description**: Sends signals to processes by name.

**Syntax**:
```bash
pkill [OPTIONS] PATTERN
```

**Options**: Similar to pgrep
- `-signal`: Specify signal (default: TERM)
- `-u USER`: Match user
- `-x`: Exact match
- `-f`: Match full command line
- `-n`: Newest only
- `-o`: Oldest only

**Examples**:
```bash
# Kill process by name
pkill nginx

# Force kill
pkill -9 nginx
pkill -KILL nginx

# Graceful kill
pkill -15 nginx
pkill -TERM nginx

# Kill user's processes
pkill -u username

# Kill by full command
pkill -f "node server.js"

# Kill oldest process
pkill -o firefox

# Kill with confirmation (use -e for echo)
pkill -e processname
```

---

### `kill` - Send Signal to Process
**Description**: Sends signals to processes by PID.

**Syntax**:
```bash
kill [SIGNAL] PID...
```

**Common Signals**:
- `1` / `HUP`: Hangup (reload configuration)
- `2` / `INT`: Interrupt (Ctrl+C)
- `3` / `QUIT`: Quit
- `9` / `KILL`: Force kill (cannot be caught)
- `15` / `TERM`: Terminate gracefully (default)
- `18` / `CONT`: Continue if stopped
- `19` / `STOP`: Stop process (cannot be caught)
- `20` / `TSTP`: Stop/pause (Ctrl+Z)

**Examples**:
```bash
# List all signals
kill -l

# Graceful termination (default)
kill 1234
kill -15 1234
kill -TERM 1234

# Force kill
kill -9 1234
kill -KILL 1234

# Reload configuration
kill -HUP 1234
kill -1 1234

# Multiple processes
kill 1234 5678 9012

# Kill process group
kill -TERM -1234    # Negative PID = process group

# Check if process exists
kill -0 1234
echo $?    # 0 = exists, 1 = doesn't exist

# Practical examples
# Reload nginx
sudo kill -HUP $(cat /var/run/nginx.pid)

# Gracefully stop service
kill -TERM $(pgrep myapp)

# Force kill if not responding
kill -KILL $(pgrep -f "stuck_process")
```

**Signal Choice**:
1. Try `TERM` (15) first - allows cleanup
2. Wait a few seconds
3. Use `KILL` (9) if needed - immediate termination

---

### `killall` - Kill by Process Name
**Description**: Kills all processes with specified name.

**Syntax**:
```bash
killall [OPTIONS] NAME...
```

**Options**:
- `-signal`: Specify signal
- `-i`: Interactive (ask before killing)
- `-u USER`: Kill user's processes
- `-w`: Wait for processes to die
- `-e`: Exact match
- `-r`: Regex match

**Examples**:
```bash
# Kill all by name
killall firefox

# Force kill
killall -9 chrome

# Interactive
killall -i nginx

# Kill user's processes
killall -u username

# Wait for processes to die
killall -w processname

# Exact match
killall -e exact_name

# Regex match
killall -r "fire.*"
```

⚠️ **Warning**: Be careful with killall - it kills ALL matching processes!

---

## Continuation

**This tutorial continues in Part 2:**

See [Linux_Commands_Complete_Mastery_Tutorial_Part2.md](file:///d:/projects/dsa/Linux_Commands_Complete_Mastery_Tutorial_Part2.md) for:

- **Disk and Storage Management** (mount, umount, fdisk, mkfs, rsync)
- **Advanced Networking Commands** (ip, ifconfig, ping, traceroute, netstat, ss, nslookup, dig, wget, curl)
- **Network Diagnostics** (tcpdump, nmap, netcat, mtr)
- **System Monitoring** (vmstat, iostat, iotop, sar, lsof, strace)
- **Shell Scripting Essentials** (variables, conditionals, loops, functions, arrays)
- **System Administration** (systemctl, journalctl, cron, at)
- **Security and Firewall** (ufw, iptables, fail2ban)
- **Kernel Tuning** (sysctl)
- **Advanced Networking Topics**

---

## Summary - Part 1

In this first part, you've learned:

### Beginner Level (✅ Completed)
- **Terminal Basics**: Navigation, file operations, text viewing
- **File System**: Understanding Linux directory structure
- **Basic Commands**: Essential daily-use commands
- **Getting Help**: man, help, --help, info

### Intermediate Level (✅ Completed)
- **Permissions & Ownership**: chmod, chown, umask, ACLs
- **Text Processing**: grep, sed, awk, cut, sort, uniq, tr, diff
- **Search & Find**: find, locate powerful file searching
- **Process Management**: ps, top, htop, kill, killall, nice, renice, nohup, bg, fg
- **System Information**: uname, hostname, uptime, df, du, free, lsblk
- **User Management**: useradd, usermod, userdel, passwd, groups, sudo
- **Package Management**: apt, yum, dnf, rpm, dpkg
- **Archiving**: tar, compression tools

### Key Concepts Covered
1. **File Operations**: Create, read, update, delete
2. **Permissions**: Understanding and managing access control
3. **Text Processing**: Powerful command-line data manipulation
4. **Process Control**: Managing system processes effectively
5. **User Administration**: System user management
6. **Package Management**: Software installation and updates

---

## Quick Command Reference - Part 1

### Navigation & Files
```bash
pwd, cd, ls, tree, touch, mkdir, cp, mv, rm, ln
```

### Viewing Files
```bash
cat, less, more, head, tail, wc
```

### Text Processing
```bash
grep, sed, awk, cut, sort, uniq, tr, diff, comm
```

### Search
```bash
find, locate, which, whereis, type
```

### Permissions
```bash
chmod, chown, chgrp, umask, stat, getfacl, setfacl
```

### Process Management
```bash
ps, top, htop, pgrep, pkill, kill, killall, nice, renice
```

### System Info
```bash
uname, hostname, uptime, whoami, who, w, last, df, du, free
```

### User Management
```bash
useradd, usermod, userdel, passwd, groupadd, groupmod, groupdel, id, groups, su, sudo
```

### Package Management
```bash
# Debian/Ubuntu
apt update, apt install, apt remove, apt search, dpkg

# Red Hat/CentOS/Fedora
yum install, yum update, yum remove, dnf, rpm
```

---

## Best Practices Learned

1. **Always use `sudo` carefully** - It grants root privileges
2. **Read man pages first** - `man command` is your friend
3. **Test with dry-run** - Many commands support `--dry-run` or `-n`
4. **Backup before modifying** - Use `cp -b` or manually backup
5. **Use tab completion** - Saves time and prevents typos
6. **Understand file permissions** - Critical for security
7. **Check command exit status** - `echo $?` shows last command status
8. **Use version control for scripts** - Git for shell scripts
9. **Comment your commands** - Especially in scripts
10. **Stay organized** - Use meaningful directory and file names

---

## Common Pitfalls to Avoid

1. **Running `rm -rf /` or similar destructive commands**
2. **Not checking current directory before `rm -rf *`**
3. **Forgetting to use sudo when needed**
4. **Not understanding symbolic vs hard links**
5. **Modifying files without backups**
6. **Using `kill -9` as first resort (use `kill -15` first)**
7. **Not reading command output/errors**
8. **Ignoring file permissions issues**
9. **Running fsck on mounted filesystems**
10. **Not testing regex patterns before applying**

---

## Next Steps

1. **Continue to Part 2** for advanced topics
2. **Practice regularly** - Use Linux for daily tasks
3. **Set up a test environment** - VM or container
4. **Read system logs** - Learn from `/var/log`
5. **Write shell scripts** - Automate repetitive tasks
6. **Explore open source projects** - Learn from others
7. **Join Linux communities** - Forums, Reddit, Stack Exchange
8. **Take online courses** - Deepen specific areas
9. **Get certified** - LPIC, RHCSA, CompTIA Linux+
10. **Build projects** - Web servers, automation, etc.

---

## Additional Resources

### Documentation
- Linux man pages: `man command`
- GNU Coreutils Manual: https://www.gnu.org/software/coreutils/manual/
- The Linux Documentation Project: https://www.tldp.org/
- Arch Linux Wiki: https://wiki.archlinux.org/

### Interactive Learning
- Linux Journey: https://linuxjourney.com/
- OverTheWire Wargames: https://overthewire.org/wargames/
- Explainshell: https://explainshell.com/

### Books (Recommended)
- "The Linux Command Line" by William Shotts
- "UNIX and Linux System Administration Handbook"
- "How Linux Works" by Brian Ward

### Communities
- r/linux, r/linuxquestions (Reddit)
- Unix & Linux Stack Exchange
- LinuxQuestions.org
- Linux.org Forums

---

## Conclusion - Part 1

Congratulations! You've completed Part 1 of the Linux Commands Complete Mastery Tutorial. You now have a solid foundation in:

- Basic terminal operations
- File and directory management  
- Text processing and manipulation
- Process management
- User administration
- Package management
- System information gathering

These skills form the core of Linux command-line proficiency. With practice, these commands will become second nature.

**Remember**: The best way to master Linux commands is through consistent practice. Set up a test environment and experiment freely!

---

🚀 **Ready for Advanced Topics?**

**Continue your journey with Part 2:**
[Linux_Commands_Complete_Mastery_Tutorial_Part2.md](file:///d:/projects/dsa/Linux_Commands_Complete_Mastery_Tutorial_Part2.md)

---

**Created with ❤️ for Linux enthusiasts at all levels**

**Last Updated**: December 2025