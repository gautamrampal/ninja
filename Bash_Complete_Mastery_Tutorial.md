# Bash Complete Mastery Tutorial
## From Beginner to Ninja - The Ultimate Guide

---

## Table of Contents

### Part 1: Foundation (Beginner Level)
1. [Introduction to Bash](#1-introduction-to-bash)
2. [Shell Basics and Getting Started](#2-shell-basics-and-getting-started)
3. [Variables and Data Types](#3-variables-and-data-types)
4. [Input and Output](#4-input-and-output)
5. [Command Line Arguments](#5-command-line-arguments)
6. [Basic Operators](#6-basic-operators)
7. [Conditional Statements](#7-conditional-statements)
8. [Loops and Iterations](#8-loops-and-iterations)

### Part 2: Intermediate Skills
9. [Functions and Script Organization](#9-functions-and-script-organization)
10. [Arrays and Indexed Data](#10-arrays-and-indexed-data)
11. [String Manipulation](#11-string-manipulation)
12. [File Operations and Testing](#12-file-operations-and-testing)
13. [Pattern Matching and Globbing](#13-pattern-matching-and-globbing)
14. [Regular Expressions](#14-regular-expressions)
15. [Process Management and Job Control](#15-process-management-and-job-control)
16. [Pipes and Redirection](#16-pipes-and-redirection)

### Part 3: Advanced Techniques (Ninja Level)
17. [Advanced Parameter Expansion](#17-advanced-parameter-expansion)
18. [Command Substitution and Subshells](#18-command-substitution-and-subshells)
19. [Error Handling and Debugging](#19-error-handling-and-debugging)
20. [Signals and Traps](#20-signals-and-traps)
21. [Advanced Text Processing](#21-advanced-text-processing)
22. [Working with JSON and APIs](#22-working-with-json-and-apis)
23. [Performance Optimization](#23-performance-optimization)
24. [Security Best Practices](#24-security-best-practices)
25. [Advanced Scripting Patterns](#25-advanced-scripting-patterns)
26. [Testing and Quality Assurance](#26-testing-and-quality-assurance)
27. [Bash for DevOps and Automation](#27-bash-for-devops-and-automation)

---

# Part 1: Foundation (Beginner Level)

## 1. Introduction to Bash

### What is Bash?
**Bash** (Bourne Again Shell) is a Unix shell and command language written by Brian Fox for the GNU Project as a free software replacement for the Bourne shell. It is the default shell on most Linux distributions and macOS.

### Why Learn Bash?
- **Automation**: Automate repetitive tasks and workflows
- **System Administration**: Manage servers and systems efficiently
- **DevOps**: Essential for CI/CD pipelines and infrastructure automation
- **Portability**: Available on virtually all Unix-like systems
- **Integration**: Glue together different tools and commands
- **Scripting Power**: Create powerful automation scripts

### Bash vs Other Shells
| Shell | Description | Key Features |
|-------|-------------|--------------|
| **Bash** | Bourne Again Shell | Most popular, rich features, backward compatible |
| **Zsh** | Z Shell | Modern, powerful auto-completion, themes |
| **Fish** | Friendly Interactive Shell | User-friendly, syntax highlighting |
| **Sh** | Bourne Shell | POSIX compliant, minimal, universal |
| **Ksh** | Korn Shell | Performance, scripting features |

### Bash Script Structure
```bash
#!/bin/bash
# Shebang: Specifies the interpreter to use

# Comments start with #
# This is a basic script structure

# Variable declaration
NAME="World"

# Command execution
echo "Hello, $NAME!"

# Exit with status code
exit 0
```

### Running Bash Scripts

**Method 1: Make executable**
```bash
# Create script
cat > hello.sh << 'EOF'
#!/bin/bash
echo "Hello, World!"
EOF

# Make executable
chmod +x hello.sh

# Run script
./hello.sh
```

**Method 2: Direct interpretation**
```bash
# Run with bash command
bash hello.sh

# Or
sh hello.sh
```

**Method 3: Source/dot command**
```bash
# Execute in current shell (preserves variables)
source hello.sh
# Or
. hello.sh
```

---

## 2. Shell Basics and Getting Started

### Shebang Line
**Description**: The shebang (`#!`) tells the system which interpreter to use.

**Common Shebangs**:
```bash
#!/bin/bash          # Use bash
#!/bin/sh            # Use sh (POSIX)
#!/usr/bin/env bash  # Find bash in PATH (portable)
#!/bin/bash -e       # Exit on error
#!/bin/bash -x       # Debug mode (print commands)
```

**Best Practice**:
```bash
#!/usr/bin/env bash
# More portable as it searches for bash in PATH
```

### Comments
```bash
# Single line comment

: '
Multi-line comment
This is useful for
longer explanations
'

# Inline comment
echo "Hello"  # This prints Hello
```

### Script Template
```bash
#!/usr/bin/env bash

#######################################
# Script Name: example.sh
# Description: Does something useful
# Author: Your Name
# Date: 2025-12-09
# Version: 1.0
#######################################

# Exit on error
set -e

# Exit on undefined variable
set -u

# Exit on pipe failure
set -o pipefail

# Enable debug mode (uncomment for debugging)
# set -x

#######################################
# Main script logic
#######################################

main() {
    echo "Script started"
    # Your code here
    echo "Script completed"
}

# Execute main function
main "$@"
```

### Set Options (Shell Behavior)
```bash
# Exit immediately if command exits with non-zero status
set -e

# Treat unset variables as error
set -u

# Prevent errors in pipeline from being masked
set -o pipefail

# Print commands before executing (debugging)
set -x

# Disable pathname expansion (globbing)
set -f

# Enable options (alternative syntax)
set -euo pipefail

# Disable options
set +x  # Turn off debug mode
```

**Practical Example**:
```bash
#!/bin/bash
set -euo pipefail  # Strict mode

# This will exit if directory doesn't exist
cd /nonexistent  # Script stops here

# This won't execute due to set -e
echo "This won't print"
```

---

## 3. Variables and Data Types

### Variable Declaration and Assignment
```bash
# Simple assignment (no spaces around =)
NAME="John"
AGE=30
CITY="New York"

# Using variables
echo "Name: $NAME"
echo "Name: ${NAME}"  # Recommended (clear boundaries)

# Read-only variable (constant)
readonly PI=3.14159
declare -r CONSTANT="Cannot change"

# Unset variable
unset NAME
```

### Variable Naming Rules
```bash
# Valid names
my_var="value"
myVar="value"
MY_VAR="value"
_var="value"
var123="value"

# Invalid names
# 123var="value"      # Cannot start with number
# my-var="value"      # Hyphen not allowed
# my var="value"      # Space not allowed
```

### Variable Types
```bash
# String
STRING="Hello, World!"
EMPTY=""
MULTI_LINE="Line 1
Line 2
Line 3"

# Integer
declare -i NUMBER=42
NUMBER=10+5  # Arithmetic evaluation: 15

# Array
declare -a ARRAY=("apple" "banana" "cherry")

# Associative array (hash/dictionary)
declare -A DICT=(
    [name]="John"
    [age]="30"
    [city]="NYC"
)

# Environment variable (exported)
export PATH="/usr/local/bin:$PATH"

# Local variable (function scope)
function my_func() {
    local LOCAL_VAR="only in function"
    echo "$LOCAL_VAR"
}
```

### Special Variables
```bash
#!/bin/bash

# Script name
echo "Script: $0"

# All arguments as separate words
echo "All args: $@"

# All arguments as single string
echo "All args: $*"

# Number of arguments
echo "Arg count: $#"

# Exit status of last command
ls /tmp
echo "Exit status: $?"

# Current shell PID
echo "PID: $$"

# PID of last background command
sleep 10 &
echo "Background PID: $!"

# Positional parameters
echo "First arg: $1"
echo "Second arg: $2"
echo "Third arg: $3"
```

**Example Script**:
```bash
#!/bin/bash
# demo_args.sh - Demonstrates special variables

echo "Script name: $0"
echo "Number of arguments: $#"
echo "All arguments (\$@): $@"
echo "All arguments (\$*): $*"
echo "First argument: $1"
echo "Second argument: $2"
echo "Process ID: $$"

# Run: bash demo_args.sh hello world
```

### Variable Expansion
```bash
# Basic expansion
NAME="Alice"
echo "Hello, $NAME"        # Hello, Alice
echo "Hello, ${NAME}"      # Hello, Alice (preferred)

# Default value
echo "${VAR:-default}"     # Use "default" if VAR is unset
echo "${VAR:=default}"     # Assign and use "default" if unset
echo "${VAR:?Error msg}"   # Error if VAR is unset
echo "${VAR:+alternate}"   # Use "alternate" if VAR is set

# String length
NAME="Alice"
echo "${#NAME}"            # 5

# Substring
TEXT="Hello, World!"
echo "${TEXT:0:5}"         # Hello
echo "${TEXT:7}"           # World!

# Pattern removal
FILE="document.txt.backup"
echo "${FILE%.backup}"     # document.txt (remove shortest match)
echo "${FILE%.*}"          # document.txt (remove shortest)
echo "${FILE%%.*}"         # document (remove longest match)

# Pattern replacement
PATH_VAR="/usr/local/bin"
echo "${PATH_VAR#/}"       # usr/local/bin (remove shortest from start)
echo "${PATH_VAR##*/}"     # bin (remove longest from start)

# Case modification
TEXT="Hello World"
echo "${TEXT,,}"           # hello world (lowercase)
echo "${TEXT^^}"           # HELLO WORLD (uppercase)
echo "${TEXT~}"            # hello World (toggle first char)
echo "${TEXT~~}"           # hELLO wORLD (toggle all)
```

### Environment Variables
```bash
# Common environment variables
echo "Home: $HOME"
echo "User: $USER"
echo "Shell: $SHELL"
echo "Path: $PATH"
echo "PWD: $PWD"
echo "Hostname: $HOSTNAME"

# Set environment variable for current session
export MY_VAR="value"

# Set for single command
MY_VAR="value" command

# Permanent environment variable (add to ~/.bashrc or ~/.bash_profile)
echo 'export MY_VAR="value"' >> ~/.bashrc
source ~/.bashrc
```

---

## 4. Input and Output

### `echo` Command
```bash
# Basic output
echo "Hello, World!"

# Without newline
echo -n "Loading..."

# Escape sequences (need -e flag)
echo -e "Line 1\nLine 2"      # Newline
echo -e "Col1\tCol2"          # Tab
echo -e "\033[31mRed\033[0m"  # ANSI colors

# Suppress escape interpretation
echo -E "This \n won't create newline"  # Default behavior
```

### `printf` Command (More Control)
```bash
# Basic printf
printf "Hello, %s!\n" "World"

# Format specifiers
printf "Name: %s, Age: %d\n" "Alice" 30

# Padding and alignment
printf "%-10s %10s\n" "Left" "Right"
printf "%05d\n" 42  # 00042 (zero padding)

# Floating point
printf "Pi: %.2f\n" 3.14159  # Pi: 3.14

# Multiple values
printf "%s: %d\n" "apples" 5 "oranges" 3
# Output:
# apples: 5
# oranges: 3

# Hex and octal
printf "Hex: %x, Octal: %o\n" 255 255
# Output: Hex: ff, Octal: 377
```

### Reading User Input
```bash
# Simple read
read NAME
echo "Hello, $NAME"

# With prompt
read -p "Enter your name: " NAME
echo "Hello, $NAME"

# Silent input (for passwords)
read -sp "Enter password: " PASSWORD
echo  # New line after password
echo "Password received"

# Read with timeout
read -t 5 -p "Enter (5 sec timeout): " INPUT

# Read single character
read -n 1 -p "Press any key to continue..."
echo

# Read into array
read -a WORDS <<< "apple banana cherry"
echo "${WORDS[0]}"  # apple
echo "${WORDS[1]}"  # banana

# Read from file
while read LINE; do
    echo "Line: $LINE"
done < input.txt

# Read with default value
read -p "Enter name [John]: " NAME
NAME=${NAME:-John}
echo "Hello, $NAME"
```

### Here Documents (Heredoc)
```bash
# Basic heredoc
cat << EOF
This is a multi-line
text block that can
span multiple lines.
EOF

# With variable expansion
NAME="Alice"
cat << EOF
Hello, $NAME!
Welcome to Bash scripting.
EOF

# Without variable expansion (quoted delimiter)
cat << 'EOF'
$NAME will not expand
This is literal text
EOF

# Indented heredoc (remove leading tabs)
cat <<- EOF
	This line is indented
	But tabs will be removed
EOF

# Heredoc to variable
MESSAGE=$(cat << EOF
Multi-line
message stored
in variable
EOF
)
echo "$MESSAGE"

# Heredoc to file
cat << EOF > output.txt
This content
goes to file
EOF
```

### Here Strings
```bash
# Pass string as input
grep "pattern" <<< "text to search"

# With variables
NAME="Alice"
read INPUT <<< "$NAME"
echo "$INPUT"  # Alice

# Multiple lines (use \n)
cat <<< "Line 1\nLine 2\nLine 3"
```

### File Redirection
```bash
# Redirect stdout to file (overwrite)
echo "Hello" > output.txt

# Redirect stdout to file (append)
echo "World" >> output.txt

# Redirect stderr to file
command 2> error.log

# Redirect both stdout and stderr
command > output.log 2>&1
command &> output.log  # Shorthand

# Redirect stderr to stdout
command 2>&1

# Discard output
command > /dev/null 2>&1

# Redirect input from file
while read LINE; do
    echo "$LINE"
done < input.txt

# Multiple redirections
command < input.txt > output.txt 2> error.log

# File descriptor manipulation
exec 3< input.txt   # Open file for reading on FD 3
read LINE <&3       # Read from FD 3
exec 3<&-           # Close FD 3

exec 4> output.txt  # Open file for writing on FD 4
echo "Data" >&4     # Write to FD 4
exec 4>&-           # Close FD 4
```

---

## 5. Command Line Arguments

### Accessing Arguments
```bash
#!/bin/bash
# script.sh - Demonstrate command line arguments

# Individual arguments
echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "Third argument: $3"

# All arguments
echo "All arguments: $@"
echo "Argument count: $#"

# Run: bash script.sh arg1 arg2 arg3
```

### Processing Arguments
```bash
#!/bin/bash

# Check argument count
if [ $# -eq 0 ]; then
    echo "Usage: $0 <arg1> <arg2>"
    exit 1
fi

# Loop through arguments
echo "Processing $# arguments:"
for arg in "$@"; do
    echo "  - $arg"
done

# Access specific positions
FIRST=$1
SECOND=$2
echo "First: $FIRST, Second: $SECOND"
```

### Shift Command
```bash
#!/bin/bash
# Shift removes first argument and shifts others down

echo "Initial arguments: $@"
echo "First: $1"

shift  # Remove first argument
echo "After shift: $@"
echo "New first: $1"

shift 2  # Remove next two arguments
echo "After shift 2: $@"
```

**Example**:
```bash
#!/bin/bash
# Process all arguments

while [ $# -gt 0 ]; do
    echo "Processing: $1"
    shift
done
```

### Option Parsing (Basic)
```bash
#!/bin/bash
# Simple option parsing

while [ $# -gt 0 ]; do
    case $1 in
        -h|--help)
            echo "Usage: $0 [-h] [-v] [-f file]"
            exit 0
            ;;
        -v|--verbose)
            VERBOSE=1
            ;;
        -f|--file)
            FILE="$2"
            shift  # Extra shift for value
            ;;
        *)
            echo "Unknown option: $1"
            exit 1
            ;;
    esac
    shift
done

echo "Verbose: ${VERBOSE:-0}"
echo "File: ${FILE:-none}"
```

### `getopts` (Built-in Parser)
```bash
#!/bin/bash
# Using getopts for option parsing

USAGE="Usage: $0 [-h] [-v] [-f file] [-o output]"

while getopts "hvf:o:" opt; do
    case $opt in
        h)
            echo "$USAGE"
            exit 0
            ;;
        v)
            VERBOSE=1
            ;;
        f)
            INPUT_FILE="$OPTARG"
            ;;
        o)
            OUTPUT_FILE="$OPTARG"
            ;;
        \?)
            echo "Invalid option: -$OPTARG" >&2
            echo "$USAGE" >&2
            exit 1
            ;;
        :)
            echo "Option -$OPTARG requires an argument" >&2
            exit 1
            ;;
    esac
done

# Shift past processed options
shift $((OPTIND-1))

# Remaining arguments are in $@
echo "Verbose: ${VERBOSE:-0}"
echo "Input: ${INPUT_FILE:-none}"
echo "Output: ${OUTPUT_FILE:-none}"
echo "Remaining args: $@"
```

---

## 6. Basic Operators

### Arithmetic Operators
```bash
# Using (( )) - Arithmetic evaluation
A=10
B=5

# Basic operations
SUM=$((A + B))           # 15
DIFF=$((A - B))          # 5
PROD=$((A * B))          # 50
QUOT=$((A / B))          # 2
MOD=$((A % B))           # 0

echo "Sum: $SUM"
echo "Difference: $DIFF"
echo "Product: $PROD"
echo "Quotient: $QUOT"
echo "Modulo: $MOD"

# Increment/Decrement
COUNT=0
((COUNT++))              # Post-increment: COUNT = 1
((++COUNT))              # Pre-increment: COUNT = 2
((COUNT--))              # Post-decrement: COUNT = 1
((--COUNT))              # Pre-decrement: COUNT = 0

# Compound assignment
NUM=10
((NUM += 5))             # NUM = 15
((NUM -= 3))             # NUM = 12
((NUM *= 2))             # NUM = 24
((NUM /= 4))             # NUM = 6
((NUM %= 4))             # NUM = 2

# Complex expressions
RESULT=$(( (A + B) * 2 ))
echo "Result: $RESULT"   # 30
```

### Using `expr` (Old Style)
```bash
# expr is older, less preferred
A=10
B=5

SUM=$(expr $A + $B)      # 15
PROD=$(expr $A \* $B)    # 50 (asterisk must be escaped)

echo "Sum: $SUM"
```

### Using `let` Command
```bash
A=10
B=5

let SUM=A+B              # No $ needed for variables
let "RESULT = A * B + 10"

echo "Sum: $SUM"         # 15
echo "Result: $RESULT"   # 60
```

### Using `bc` for Floating Point
```bash
# Basic calculator
echo "5.5 + 3.2" | bc    # 8.7
echo "10 / 3" | bc       # 3 (integer division)
echo "scale=2; 10 / 3" | bc  # 3.33 (2 decimal places)

# More complex
RESULT=$(echo "scale=4; 22/7" | bc)
echo "Pi approximation: $RESULT"  # 3.1428

# Using here string
bc <<< "scale=2; (5 + 3) * 2.5"  # 20.00

# Mathematical functions (using -l for math library)
echo "s(3.14159/2)" | bc -l  # sin(π/2) = 1
echo "e(1)" | bc -l          # e^1 = 2.71828...
echo "l(2.71828)" | bc -l    # ln(e) = 1
```

### Comparison Operators (Arithmetic)
```bash
A=10
B=5

# Using (( ))
if (( A > B )); then
    echo "$A is greater than $B"
fi

if (( A >= B )); then
    echo "$A is greater than or equal to $B"
fi

if (( A < B )); then
    echo "$A is less than $B"
fi

if (( A <= B )); then
    echo "$A is less than or equal to $B"
fi

if (( A == B )); then
    echo "$A equals $B"
fi

if (( A != B )); then
    echo "$A does not equal $B"
fi
```

### String Operators
```bash
STR1="hello"
STR2="world"
EMPTY=""

# Equality
if [ "$STR1" = "$STR2" ]; then
    echo "Strings are equal"
fi

if [ "$STR1" != "$STR2" ]; then
    echo "Strings are not equal"
fi

# Empty check
if [ -z "$EMPTY" ]; then
    echo "String is empty"
fi

if [ -n "$STR1" ]; then
    echo "String is not empty"
fi

# Lexicographic comparison
if [[ "$STR1" < "$STR2" ]]; then
    echo "$STR1 comes before $STR2"
fi

if [[ "$STR1" > "$STR2" ]]; then
    echo "$STR1 comes after $STR2"
fi

# Pattern matching (only with [[  ]])
if [[ "$STR1" == h* ]]; then
    echo "Starts with h"
fi

if [[ "$STR1" =~ ^[a-z]+$ ]]; then
    echo "Contains only lowercase letters"
fi
```

### Logical Operators
```bash
A=10
B=5
C=15

# AND operator
if (( A > B )) && (( A < C )); then
    echo "$A is between $B and $C"
fi

# OR operator
if (( A == B )) || (( A == 10 )); then
    echo "A is either $B or 10"
fi

# NOT operator
if ! (( A == B )); then
    echo "A is not equal to B"
fi

# Combining operators
if (( A > B )) && (( B > 0 )) || (( C < 20 )); then
    echo "Complex condition is true"
fi

# Using -a (AND) and -o (OR) with [ ]
if [ $A -gt $B -a $A -lt $C ]; then
    echo "$A is between $B and $C"
fi

# Better: use separate [ ] with && ||
if [ $A -gt $B ] && [ $A -lt $C ]; then
    echo "$A is between $B and $C"
fi
```

---

## 7. Conditional Statements

### `if` Statement
```bash
# Basic if
if [ condition ]; then
    # commands
fi

# if-else
if [ condition ]; then
    # commands if true
else
    # commands if false
fi

# if-elif-else
if [ condition1 ]; then
    # commands for condition1
elif [ condition2 ]; then
    # commands for condition2
elif [ condition3 ]; then
    # commands for condition3
else
    # commands if all false
fi
```

**Example**:
```bash
#!/bin/bash
# Grade checker

SCORE=85

if [ $SCORE -ge 90 ]; then
    GRADE="A"
elif [ $SCORE -ge 80 ]; then
    GRADE="B"
elif [ $SCORE -ge 70 ]; then
    GRADE="C"
elif [ $SCORE -ge 60 ]; then
    GRADE="D"
else
    GRADE="F"
fi

echo "Score: $SCORE, Grade: $GRADE"
```

### Test Operators `[ ]` vs `[[ ]]`
```bash
# Single bracket [ ] - POSIX compatible, older
if [ $A -eq $B ]; then
    echo "Equal"
fi

# Double bracket [[ ]] - Bash extension, more features
if [[ $A -eq $B ]]; then
    echo "Equal"
fi

# Benefits of [[ ]]:
# 1. Pattern matching
if [[ $NAME == J* ]]; then
    echo "Name starts with J"
fi

# 2. Regular expressions
if [[ $EMAIL =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
    echo "Valid email"
fi

# 3. No word splitting
FILE="my file.txt"
if [[ -f $FILE ]]; then  # Works even with spaces (no quotes needed)
    echo "File exists"
fi

# 4. Logical operators
if [[ $A -gt 5 && $B -lt 10 ]]; then
    echo "Both conditions true"
fi
```

### File Test Operators
```bash
FILE="/path/to/file"
DIR="/path/to/directory"

# Existence
[ -e $FILE ]      # File exists (any type)
[ -f $FILE ]      # Regular file exists
[ -d $DIR ]       # Directory exists
[ -L $FILE ]      # Symbolic link exists
[ -S $FILE ]      # Socket exists
[ -p $FILE ]      # Named pipe exists
[ -b $FILE ]      # Block device exists
[ -c $FILE ]      # Character device exists

# Permissions
[ -r $FILE ]      # Readable
[ -w $FILE ]      # Writable
[ -x $FILE ]      # Executable
[ -u $FILE ]      # SUID bit set
[ -g $FILE ]      # SGID bit set
[ -k $FILE ]      # Sticky bit set

# Properties
[ -s $FILE ]      # File size > 0
[ -t FD ]         # File descriptor is terminal

# Comparisons
[ $FILE1 -nt $FILE2 ]  # FILE1 newer than FILE2
[ $FILE1 -ot $FILE2 ]  # FILE1 older than FILE2
[ $FILE1 -ef $FILE2 ]  # Same file (hard links)
```

**Example**:
```bash
#!/bin/bash
# File checker

FILE="$1"

if [ -z "$FILE" ]; then
    echo "Usage: $0 <file>"
    exit 1
fi

if [ ! -e "$FILE" ]; then
    echo "File does not exist"
    exit 1
fi

echo "File: $FILE"

if [ -f "$FILE" ]; then
    echo "  Type: Regular file"
elif [ -d "$FILE" ]; then
    echo "  Type: Directory"
elif [ -L "$FILE" ]; then
    echo "  Type: Symbolic link"
fi

if [ -r "$FILE" ]; then
    echo "  Readable: Yes"
else
    echo "  Readable: No"
fi

if [ -w "$FILE" ]; then
    echo "  Writable: Yes"
else
    echo "  Writable: No"
fi

if [ -x "$FILE" ]; then
    echo "  Executable: Yes"
else
    echo "  Executable: No"
fi

if [ -s "$FILE" ]; then
    SIZE=$(stat -f%z "$FILE" 2>/dev/null || stat -c%s "$FILE" 2>/dev/null)
    echo "  Size: $SIZE bytes"
else
    echo "  Size: 0 bytes (empty)"
fi
```

### Integer Comparison Operators
```bash
A=10
B=5

[ $A -eq $B ]     # Equal
[ $A -ne $B ]     # Not equal
[ $A -gt $B ]     # Greater than
[ $A -ge $B ]     # Greater than or equal
[ $A -lt $B ]     # Less than
[ $A -le $B ]     # Less than or equal
```

### `case` Statement
```bash
# Basic case structure
case $VARIABLE in
    pattern1)
        # commands
        ;;
    pattern2|pattern3)
        # commands for pattern2 or pattern3
        ;;
    pattern4)
        # commands
        ;;
    *)
        # default case
        ;;
esac
```

**Example: Menu**
```bash
#!/bin/bash
# Simple menu

echo "Select an option:"
echo "1) List files"
echo "2) Show date"
echo "3) Show current directory"
echo "4) Exit"

read -p "Enter choice [1-4]: " CHOICE

case $CHOICE in
    1)
        echo "Files:"
        ls -l
        ;;
    2)
        echo "Current date:"
        date
        ;;
    3)
        echo "Current directory:"
        pwd
        ;;
    4)
        echo "Goodbye!"
        exit 0
        ;;
    *)
        echo "Invalid choice"
        exit 1
        ;;
esac
```

**Example: Pattern Matching**
```bash
#!/bin/bash
# File type checker

FILE="$1"

case $FILE in
    *.txt)
        echo "Text file"
        ;;
    *.jpg|*.jpeg|*.png|*.gif)
        echo "Image file"
        ;;
    *.sh)
        echo "Shell script"
        ;;
    *.tar.gz|*.tgz)
        echo "Compressed archive"
        ;;
    *)
        echo "Unknown file type"
        ;;
esac
```

**Example: Multi-pattern**
```bash
#!/bin/bash
# Command router

COMMAND="$1"

case $COMMAND in
    start|run|execute)
        echo "Starting..."
        # start logic
        ;;
    stop|halt|kill)
        echo "Stopping..."
        # stop logic
        ;;
    restart|reload)
        echo "Restarting..."
        # restart logic
        ;;
    status|info|check)
        echo "Checking status..."
        # status logic
        ;;
    help|--help|-h)
        echo "Usage: $0 {start|stop|restart|status|help}"
        ;;
    *)
        echo "Unknown command: $COMMAND"
        exit 1
        ;;
esac
```

### Ternary-like Operations
```bash
# Using && and ||
[ condition ] && true_command || false_command

# Example
[ $A -gt $B ] && MAX=$A || MAX=$B
echo "Max: $MAX"

# Using (( )) for arithmetic
MAX=$(( A > B ? A : B ))
echo "Max: $MAX"

# Using if in subshell
RESULT=$(if [ $A -gt $B ]; then echo $A; else echo $B; fi)
echo "Result: $RESULT"
```

---

## 8. Loops and Iterations

### `for` Loop (C-style)
```bash
# C-style for loop
for (( i=0; i<10; i++ )); do
    echo "Number: $i"
done

# With step
for (( i=0; i<=20; i+=2 )); do
    echo "Even number: $i"
done

# Countdown
for (( i=10; i>=0; i-- )); do
    echo "Countdown: $i"
done

# Multiple variables
for (( i=0, j=10; i<10; i++, j-- )); do
    echo "i=$i, j=$j"
done
```

### `for` Loop (List iteration)
```bash
# Iterate over list
for item in apple banana cherry; do
    echo "Fruit: $item"
done

# Iterate over command output
for file in $(ls *.txt); do
    echo "Processing: $file"
done

# Better: use glob pattern (handles spaces)
for file in *.txt; do
    echo "Processing: $file"
done

# Iterate over array
FRUITS=("apple" "banana" "cherry")
for fruit in "${FRUITS[@]}"; do
    echo "Fruit: $fruit"
done

# Iterate over range (using brace expansion)
for num in {1..10}; do
    echo "Number: $num"
done

# Step in range
for num in {0..100..10}; do
    echo "Number: $num"
done

# Iterate over command substitution
for user in $(cat users.txt); do
    echo "User: $user"
done
```

### `while` Loop
```bash
# Basic while loop
COUNT=0
while [ $COUNT -lt 5 ]; do
    echo "Count: $COUNT"
    ((COUNT++))
done

# Infinite loop
while true; do
    echo "Running..."
    sleep 1
    # Break condition needed
done

# Reading file line by line
while read LINE; do
    echo "Line: $LINE"
done < input.txt

# Reading with IFS (Internal Field Separator)
while IFS=: read user pass uid gid comment home shell; do
    echo "User: $user, UID: $uid, Shell: $shell"
done < /etc/passwd

# While with condition check
while [ -f /tmp/running ]; do
    echo "Process running..."
    sleep 5
done
```

### `until` Loop
```bash
# Run until condition is true (opposite of while)
COUNT=0
until [ $COUNT -ge 5 ]; do
    echo "Count: $COUNT"
    ((COUNT++))
done

# Wait for file
until [ -f /tmp/done ]; do
    echo "Waiting for file..."
    sleep 1
done
echo "File found!"

# Wait for service
until ping -c 1 example.com &>/dev/null; do
    echo "Waiting for network..."
    sleep 2
done
echo "Network is up!"
```

### Loop Control: `break` and `continue`
```bash
# break - Exit loop
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        echo "Breaking at $i"
        break
    fi
    echo "Number: $i"
done

# continue - Skip to next iteration
for i in {1..10}; do
    if [ $((i % 2)) -eq 0 ]; then
        continue  # Skip even numbers
    fi
    echo "Odd number: $i"
done

# break with nested loops
for i in {1..3}; do
    for j in {1..3}; do
        if [ $j -eq 2 ]; then
            break  # Only breaks inner loop
        fi
        echo "i=$i, j=$j"
    done
done

# break with level (break outer loop)
for i in {1..3}; do
    for j in {1..3}; do
        if [ $j -eq 2 ]; then
            break 2  # Break both loops
        fi
        echo "i=$i, j=$j"
    done
done
```

### `select` Loop (Menu)
```bash
# Create interactive menu
PS3="Select option: "  # Prompt string

select option in "List files" "Show date" "Show PWD" "Quit"; do
    case $option in
        "List files")
            ls -l
            ;;
        "Show date")
            date
            ;;
        "Show PWD")
            pwd
            ;;
        "Quit")
            echo "Goodbye!"
            break
            ;;
        *)
            echo "Invalid option"
            ;;
    esac
done
```

**Example: Processing files**
```bash
#!/bin/bash
# Process all .txt files

COUNT=0
SUCCESS=0
FAILED=0

for file in *.txt; do
    # Check if glob matched any files
    [ -f "$file" ] || continue
    
    ((COUNT++))
    echo "Processing file $COUNT: $file"
    
    if process_file "$file"; then
        ((SUCCESS++))
        echo "  ✓ Success"
    else
        ((FAILED++))
        echo "  ✗ Failed"
    fi
done

echo "Summary:"
echo "  Total: $COUNT"
echo "  Success: $SUCCESS"
echo "  Failed: $FAILED"
```

---

# Part 2: Intermediate Skills

## 9. Functions and Script Organization

### Function Declaration
```bash
# Method 1: function keyword
function my_function {
    echo "Hello from function"
}

# Method 2: () syntax (POSIX)
my_function() {
    echo "Hello from function"
}

# Method 3: function keyword with ()
function my_function() {
    echo "Hello from function"
}

# Calling function
my_function
```

### Function Parameters
```bash
# Functions use positional parameters like scripts
greet() {
    local NAME=$1    # First parameter
    local AGE=$2     # Second parameter
    echo "Hello, $NAME! You are $AGE years old."
}

# Call with arguments
greet "Alice" 30
# Output: Hello, Alice! You are 30 years old.

# Access all parameters
print_all() {
    echo "Function: $FUNCNAME"
    echo "Number of args: $#"
    echo "All args: $@"
    
    local i=1
    for arg in "$@"; do
        echo "  Arg $i: $arg"
        ((i++))
    done
}

print_all apple banana cherry
```

### Return Values
```bash
# Return status code (0-255)
is_even() {
    local NUM=$1
    if [ $((NUM % 2)) -eq 0 ]; then
        return 0  # Success (true)
    else
        return 1  # Failure (false)
    fi
}

# Use in condition
if is_even 4; then
    echo "4 is even"
fi

# Check return code
is_even 7
if [ $? -eq 0 ]; then
    echo "Even"
else
    echo "Odd"
fi

# Return data via echo
get_max() {
    local A=$1
    local B=$2
    if [ $A -gt $B ]; then
        echo $A
    else
        echo $B
    fi
}

# Capture output
MAX=$(get_max 10 20)
echo "Max: $MAX"  # Max: 20
```

### Local Variables
```bash
# Global variable
GLOBAL_VAR="I am global"

my_function() {
    # Local variable (only exists in function)
    local LOCAL_VAR="I am local"
    
    # Modify global
    GLOBAL_VAR="Modified by function"
    
    echo "Inside function:"
    echo "  Local: $LOCAL_VAR"
    echo "  Global: $GLOBAL_VAR"
}

echo "Before function:"
echo "Global: $GLOBAL_VAR"
# echo "Local: $LOCAL_VAR"  # Would be empty

my_function

echo "After function:"
echo "Global: $GLOBAL_VAR"  # Modified
# echo "Local: $LOCAL_VAR"  # Still empty (out of scope)
```

### Function Libraries
```bash
# lib.sh - Library file
#!/bin/bash

# String functions
string_length() {
    echo "${#1}"
}

string_upper() {
    echo "${1^^}"
}

string_lower() {
    echo "${1,,}"
}

# Math functions
add() {
    echo $(($1 + $2))
}

multiply() {
    echo $(($1 * $2))
}

# File functions
file_exists() {
    [ -f "$1" ]
}

# Export functions (make available to subprocesses)
export -f string_length
export -f string_upper
export -f add
```

```bash
# main.sh - Main script
#!/bin/bash

# Source library
source ./lib.sh
# Or
. ./lib.sh

# Use library functions
TEXT="hello world"
echo "Original: $TEXT"
echo "Length: $(string_length "$TEXT")"
echo "Upper: $(string_upper "$TEXT")"
echo "Lower: $(string_lower "$TEXT")"

RESULT=$(add 10 20)
echo "10 + 20 = $RESULT"
```

### Recursive Functions
```bash
# Factorial
factorial() {
    local N=$1
    if [ $N -le 1 ]; then
        echo 1
    else
        local PREV=$(factorial $((N - 1)))
        echo $((N * PREV))
    fi
}

echo "5! = $(factorial 5)"  # 120

# Fibonacci
fibonacci() {
    local N=$1
    if [ $N -le 1 ]; then
        echo $N
    else
        local A=$(fibonacci $((N - 1)))
        local B=$(fibonacci $((N - 2)))
        echo $((A + B))
    fi
}

echo "Fibonacci(10) = $(fibonacci 10)"  # 55

# Directory tree traversal
list_files() {
    local DIR=$1
    local INDENT=$2
    
    for item in "$DIR"/*; do
        [ -e "$item" ] || continue
        echo "${INDENT}$(basename "$item")"
        
        if [ -d "$item" ]; then
            list_files "$item" "$INDENT  "
        fi
    done
}

list_files "/path/to/dir" ""
```

### Function Best Practices
```bash
#!/bin/bash
# Best practices for functions

# 1. Use local variables
calculate_sum() {
    local TOTAL=0  # Always use local for function variables
    for num in "$@"; do
        ((TOTAL += num))
    done
    echo $TOTAL
}

# 2. Validate parameters
divide() {
    # Check parameter count
    if [ $# -ne 2 ]; then
        echo "Error: divide requires 2 parameters" >&2
        return 1
    fi
    
    local NUM=$1
    local DEN=$2
    
    # Validate denominator
    if [ $DEN -eq 0 ]; then
        echo "Error: Division by zero" >&2
        return 1
    fi
    
    echo $((NUM / DEN))
}

# 3. Document functions
#######################################
# Sends email notification
# Globals:
#   SMTP_SERVER
# Arguments:
#   $1 - Recipient email
#   $2 - Subject
#   $3 - Message body
# Returns:
#   0 on success, 1 on failure
#######################################
send_email() {
    local RECIPIENT=$1
    local SUBJECT=$2
    local BODY=$3
    
    # Implementation...
}

# 4. Use meaningful names
# Bad
f1() { echo "$1"; }

# Good
print_message() { echo "$1"; }

# 5. Single Responsibility
# Bad - does too much
process_user() {
    validate_user "$1"
    create_user "$1"
    send_welcome_email "$1"
    log_user_creation "$1"
}

# Good - separate functions
validate_user() { :; }
create_user() { :; }
send_welcome_email() { :; }
log_user_creation() { :; }
```

---

## 10. Arrays and Indexed Data

### Indexed Arrays
```bash
# Declaration
ARRAY=()                          # Empty array
ARRAY=(apple banana cherry)       # Initialize with values
ARRAY=([0]="apple" [1]="banana")  # Explicit indices

# Adding elements
ARRAY[3]="date"                   # Add at index 3
ARRAY+=(elderberry)               # Append element
ARRAY+=(fig grape)                # Append multiple

# Accessing elements
echo "${ARRAY[0]}"                # First element
echo "${ARRAY[2]}"                # Third element
echo "${ARRAY[-1]}"               # Last element (Bash 4.3+)
echo "${ARRAY[-2]}"               # Second to last

# Array properties
echo "${#ARRAY[@]}"               # Number of elements
echo "${#ARRAY[*]}"               # Number of elements (alternative)
echo "${!ARRAY[@]}"               # All indices
echo "${#ARRAY[2]}"               # Length of element at index 2

# All elements
echo "${ARRAY[@]}"                # All elements (word splitting)
echo "${ARRAY[*]}"                # All elements (single string)

# Iterate over array
for item in "${ARRAY[@]}"; do
    echo "Item: $item"
done

# Iterate with indices
for i in "${!ARRAY[@]}"; do
    echo "Index $i: ${ARRAY[$i]}"
done
```

### Array Slicing
```bash
ARRAY=(a b c d e f g h)

# Slice: ${ARRAY[@]:start:length}
echo "${ARRAY[@]:2:3}"    # c d e (start at 2, take 3)
echo "${ARRAY[@]:3}"      # d e f g h (from index 3 to end)
echo "${ARRAY[@]: -3}"    # f g h (last 3 elements)
echo "${ARRAY[@]:1:4}"    # b c d e

# Copy array
COPY=("${ARRAY[@]}")

# Concatenate arrays
ARRAY1=(a b c)
ARRAY2=(d e f)
COMBINED=("${ARRAY1[@]}" "${ARRAY2[@]}")
echo "${COMBINED[@]}"     # a b c d e f
```

### Array Manipulation
```bash
ARRAY=(apple banana cherry date elderberry)

# Replace element
ARRAY[1]="blueberry"

# Delete element
unset ARRAY[2]           # Deletes cherry, leaves gap

# Delete entire array
unset ARRAY

# Remove gaps (re-index)
ARRAY=(apple banana cherry)
unset ARRAY[1]
ARRAY=("${ARRAY[@]}")    # Re-index: (apple cherry)

# Find and replace in array
ARRAY=(cat dog cat bird cat)
ARRAY=("${ARRAY[@]/cat/mouse}")  # Replace all cats with mouse

# Filter array
NUMBERS=(1 2 3 4 5 6 7 8 9 10)
EVENS=()
for num in "${NUMBERS[@]}"; do
    if [ $((num % 2)) -eq 0 ]; then
        EVENS+=($num)
    fi
done
echo "${EVENS[@]}"       # 2 4 6 8 10
```

### Associative Arrays (Hash/Dictionary)
```bash
# Declare associative array
declare -A PERSON

# Add key-value pairs
PERSON[name]="Alice"
PERSON[age]=30
PERSON[city]="New York"
PERSON[country]="USA"

# Access values
echo "${PERSON[name]}"   # Alice
echo "${PERSON[age]}"    # 30

# All keys
echo "${!PERSON[@]}"     # name age city country

# All values
echo "${PERSON[@]}"      # Alice 30 New York USA

# Number of entries
echo "${#PERSON[@]}"     # 4

# Check if key exists
if [ -v PERSON[name] ]; then
    echo "Name key exists"
fi

# Iterate over associative array
for key in "${!PERSON[@]}"; do
    echo "$key: ${PERSON[$key]}"
done

# Delete key
unset PERSON[city]

# Initialize with values
declare -A CONFIG=(
    [host]="localhost"
    [port]=8080
    [ssl]="true"
)
```

### Array from Command Output
```bash
# Read into array (word splitting)
FILES=($(ls))

# Better: use mapfile/readarray
mapfile -t LINES < file.txt
# Or
readarray -t LINES < file.txt

# From command output
mapfile -t USERS < <(cut -d: -f1 /etc/passwd)

# IFS manipulation for custom delimiters
IFS=':' read -ra FIELDS <<< "apple:banana:cherry"
echo "${FIELDS[0]}"      # apple
echo "${FIELDS[1]}"      # banana

# Array from string
STRING="apple banana cherry"
ARRAY=($STRING)          # Simple word splitting
echo "${ARRAY[@]}"       # apple banana cherry

# Preserve spaces in elements
IFS=',' read -ra ARRAY <<< "New York,Los Angeles,Chicago"
echo "${ARRAY[0]}"       # New York
```

### Practical Array Examples
```bash
# Example 1: Processing log files
LOG_FILES=($(find /var/log -name "*.log"))
for log in "${LOG_FILES[@]}"; do
    echo "Processing: $log"
    # grep "ERROR" "$log" >> errors.txt
done

# Example 2: Menu system
declare -A MENU=(
    [1]="View logs"
    [2]="Check disk space"
    [3]="Show processes"
    [4]="Exit"
)

for key in $(echo "${!MENU[@]}" | tr ' ' '\n' | sort -n); do
    echo "$key) ${MENU[$key]}"
done

# Example 3: Configuration management
declare -A DB_CONFIG=(
    [host]="localhost"
    [port]=5432
    [database]="mydb"
    [user]="admin"
    [password]="secret"
)

# Build connection string
CONN_STRING="postgresql://${DB_CONFIG[user]}:${DB_CONFIG[password]}@${DB_CONFIG[host]}:${DB_CONFIG[port]}/${DB_CONFIG[database]}"
echo "$CONN_STRING"

# Example 4: Data aggregation
declare -A WORD_COUNT

while read -r word; do
    ((WORD_COUNT[$word]++))
done < words.txt

for word in "${!WORD_COUNT[@]}"; do
    echo "$word: ${WORD_COUNT[$word]}"
done | sort -t: -k2 -rn  # Sort by count
```

---

## 11. String Manipulation

### String Length
```bash
STRING="Hello, World!"
echo "${#STRING}"        # 13

# Empty string
EMPTY=""
echo "${#EMPTY}"         # 0
```

### Substring Extraction
```bash
STRING="Hello, World!"

# ${STRING:position:length}
echo "${STRING:0:5}"     # Hello
echo "${STRING:7:5}"     # World
echo "${STRING:7}"       # World! (from position to end)
echo "${STRING: -6}"     # World! (last 6 chars, note the space)
echo "${STRING: -6:5}"   # World

# Negative length (remove from end)
echo "${STRING:0:-1}"    # Hello, World (remove last char)
```

### String Replacement
```bash
STRING="the quick brown fox jumps over the lazy dog"

# Replace first occurrence
echo "${STRING/the/a}"   # a quick brown fox jumps over the lazy dog

# Replace all occurrences
echo "${STRING//the/a}"  # a quick brown fox jumps over a lazy dog

# Replace at beginning
echo "${STRING/#the/a}"  # a quick brown fox jumps over the lazy dog

# Replace at end
echo "${STRING/%dog/cat}" # the quick brown fox jumps over the lazy cat

# Delete pattern (replace with nothing)
echo "${STRING/the/}"    # quick brown fox jumps over the lazy dog
echo "${STRING//the/}"   # quick brown fox jumps over lazy dog
```

### Pattern Removal
```bash
FILENAME="document.txt.backup"

# Remove shortest match from end
echo "${FILENAME%.backup}"       # document.txt

# Remove longest match from end
echo "${FILENAME%%.*}"           # document

# Remove shortest match from beginning
PATH="/usr/local/bin/script"
echo "${PATH#/}"                 # usr/local/bin/script
echo "${PATH#*/}"                # usr/local/bin/script

# Remove longest match from beginning
echo "${PATH##*/}"               # script (basename)
echo "${PATH%/*}"                # /usr/local/bin (dirname)
```

### Case Conversion
```bash
STRING="Hello World"

# Lowercase
echo "${STRING,,}"       # hello world
echo "${STRING,}"        # hello World (first char only)

# Uppercase
echo "${STRING^^}"       # HELLO WORLD
echo "${STRING^}"        # Hello World (first char only)

# Toggle case
echo "${STRING~~}"       # hELLO wORLD
echo "${STRING~}"        # hello World (first char only)

# Using tr command
echo "$STRING" | tr '[:upper:]' '[:lower:]'  # hello world
echo "$STRING" | tr '[:lower:]' '[:upper:]'  # HELLO WORLD
```

### String Concatenation
```bash
# Simple concatenation
FIRST="Hello"
LAST="World"
FULL="$FIRST $LAST"
echo "$FULL"             # Hello World

# Append to string
STRING="Hello"
STRING="$STRING, World"
STRING+=", How are you?"
echo "$STRING"           # Hello, World, How are you?

# Join array elements
ARRAY=(apple banana cherry)
IFS=,
JOINED="${ARRAY[*]}"
echo "$JOINED"           # apple,banana,cherry
unset IFS
```

### String Splitting
```bash
# Split by delimiter
STRING="apple:banana:cherry:date"
IFS=: read -ra PARTS <<< "$STRING"
echo "${PARTS[0]}"       # apple
echo "${PARTS[2]}"       # cherry

# Split into array
DATA="John,30,New York"
IFS=, read -ra FIELDS <<< "$DATA"
NAME="${FIELDS[0]}"
AGE="${FIELDS[1]}"
CITY="${FIELDS[2]}"

# Split by whitespace
TEXT="one two three four"
WORDS=($TEXT)
echo "${WORDS[1]}"       # two
```

### String Trimming
```bash
# Trim whitespace
STRING="  Hello, World!  "

# Trim leading whitespace
TRIMMED="${STRING#"${STRING%%[![:space:]]*}"}"

# Trim trailing whitespace
TRIMMED="${TRIMMED%"${TRIMMED##*[![:space:]]}"}"

# Trim both (function)
trim() {
    local var="$1"
    # Remove leading whitespace
    var="${var#"${var%%[![:space:]]*}"}"
    # Remove trailing whitespace
    var="${var%"${var##*[![:space:]]}"}"
    echo "$var"
}

RESULT=$(trim "$STRING")
echo "[$RESULT]"         # [Hello, World!]

# Using sed
echo "$STRING" | sed 's/^[[:space:]]*//;s/[[:space:]]*$//'

# Using xargs (simple)
echo "$STRING" | xargs
```

### String Contains
```bash
STRING="The quick brown fox"

# Check if contains substring
if [[ $STRING == *"quick"* ]]; then
    echo "Contains 'quick'"
fi

# Using grep
if echo "$STRING" | grep -q "quick"; then
    echo "Contains 'quick'"
fi

# Using case
case $STRING in
    *quick*)
        echo "Contains 'quick'"
        ;;
esac
```

### String Starts/Ends With
```bash
STRING="Hello, World!"

# Starts with
if [[ $STRING == Hello* ]]; then
    echo "Starts with 'Hello'"
fi

# Ends with
if [[ $STRING == *! ]]; then
    echo "Ends with '!'"
fi

# Using pattern matching
case $STRING in
    Hello*)
        echo "Starts with Hello"
        ;;
esac
```

### String Comparison
```bash
STR1="apple"
STR2="banana"

# Equality
if [ "$STR1" = "$STR2" ]; then
    echo "Equal"
fi

# Inequality
if [ "$STR1" != "$STR2" ]; then
    echo "Not equal"
fi

# Lexicographic comparison
if [[ $STR1 < $STR2 ]]; then
    echo "$STR1 comes before $STR2"
fi

if [[ $STR1 > $STR2 ]]; then
    echo "$STR1 comes after $STR2"
fi

# Case-insensitive comparison
STR1_LOWER="${STR1,,}"
STR2_LOWER="${STR2,,}"
if [ "$STR1_LOWER" = "$STR2_LOWER" ]; then
    echo "Equal (case-insensitive)"
fi
```

### Practical String Examples
```bash
# Example 1: Parse URL
URL="https://user:pass@example.com:8080/path/to/resource?query=1"

# Extract protocol
PROTOCOL="${URL%%://*}"          # https

# Remove protocol
TEMP="${URL#*://}"               # user:pass@example.com:8080/path/to/resource?query=1

# Extract credentials
CREDENTIALS="${TEMP%%@*}"        # user:pass
USERNAME="${CREDENTIALS%%:*}"    # user
PASSWORD="${CREDENTIALS#*:}"     # pass

# Remove credentials
TEMP="${TEMP#*@}"                # example.com:8080/path/to/resource?query=1

# Extract host and port
HOSTPORT="${TEMP%%/*}"           # example.com:8080
HOST="${HOSTPORT%%:*}"           # example.com
PORT="${HOSTPORT#*:}"            # 8080

# Extract path
TEMP="${TEMP#*/}"                # path/to/resource?query=1
PATH="${TEMP%%\?*}"              # path/to/resource

# Extract query
QUERY="${TEMP#*\?}"              # query=1

echo "Protocol: $PROTOCOL"
echo "Username: $USERNAME"
echo "Password: $PASSWORD"
echo "Host: $HOST"
echo "Port: $PORT"
echo "Path: $PATH"
echo "Query: $QUERY"

# Example 2: File path manipulation
FILEPATH="/home/user/documents/report.txt"

FILENAME="${FILEPATH##*/}"       # report.txt
DIRNAME="${FILEPATH%/*}"         # /home/user/documents
BASENAME="${FILENAME%.*}"        # report
EXTENSION="${FILENAME##*.}"      # txt

echo "Full path: $FILEPATH"
echo "Directory: $DIRNAME"
echo "Filename: $FILENAME"
echo "Basename: $BASENAME"
echo "Extension: $EXTENSION"

# Example 3: CSV parsing
process_csv() {
    while IFS=, read -r name age city; do
        # Trim whitespace
        name=$(echo "$name" | xargs)
        age=$(echo "$age" | xargs)
        city=$(echo "$city" | xargs)
        
        echo "Name: $name, Age: $age, City: $city"
    done < "$1"
}

# Example 4: Template replacement
TEMPLATE="Hello, {{name}}! You are {{age}} years old."
NAME="Alice"
AGE=30

OUTPUT="${TEMPLATE//\{\{name\}\}/$NAME}"
OUTPUT="${OUTPUT//\{\{age\}\}/$AGE}"
echo "$OUTPUT"  # Hello, Alice! You are 30 years old.
```

---

## 12. File Operations and Testing

### File Test Operators (Comprehensive)
```bash
FILE="/path/to/file"

# Existence tests
[ -e $FILE ]    # Exists (any type)
[ -f $FILE ]    # Is regular file
[ -d $FILE ]    # Is directory
[ -L $FILE ]    # Is symbolic link
[ -S $FILE ]    # Is socket
[ -p $FILE ]    # Is named pipe (FIFO)
[ -b $FILE ]    # Is block device
[ -c $FILE ]    # Is character device

# Permission tests
[ -r $FILE ]    # Is readable
[ -w $FILE ]    # Is writable
[ -x $FILE ]    # Is executable
[ -u $FILE ]    # Has SUID bit set
[ -g $FILE ]    # Has SGID bit set
[ -k $FILE ]    # Has sticky bit set

# Property tests
[ -s $FILE ]    # Size > 0 (not empty)
[ -t FD ]       # File descriptor is terminal
[ -O $FILE ]    # Owned by effective UID
[ -G $FILE ]    # Owned by effective GID

# Comparison tests
[ $FILE1 -nt $FILE2 ]  # FILE1 is newer than FILE2
[ $FILE1 -ot $FILE2 ]  # FILE1 is older than FILE2
[ $FILE1 -ef $FILE2 ]  # FILE1 and FILE2 are same file (hard links)
```

### Reading Files
```bash
# Read entire file into variable
CONTENT=$(cat file.txt)
# Or
CONTENT=$(<file.txt)

# Read file line by line
while read -r LINE; do
    echo "Line: $LINE"
done < file.txt

# Read with line numbers
LINE_NUM=1
while read -r LINE; do
    echo "$LINE_NUM: $LINE"
    ((LINE_NUM++))
done < file.txt

# Read specific fields
while IFS=: read -r user pass uid gid comment home shell; do
    echo "User: $user, Shell: $shell"
done < /etc/passwd

# Read into array
mapfile -t LINES < file.txt
# Or
readarray -t LINES < file.txt

# Process each line
for line in "${LINES[@]}"; do
    echo "$line"
done

# Skip header/footer lines
mapfile -t -s 1 LINES < file.txt  # Skip first line
```

### Writing Files
```bash
# Overwrite file
echo "Hello, World!" > file.txt

# Append to file
echo "New line" >> file.txt

# Write multiple lines
cat > file.txt << EOF
Line 1
Line 2
Line 3
EOF

# Write with variables
cat > config.txt << EOF
Host: $HOSTNAME
User: $USER
Date: $(date)
EOF

# Write array to file
ARRAY=(apple banana cherry)
printf "%s\n" "${ARRAY[@]}" > fruits.txt

# Atomic write (safer for critical files)
NEW_DATA="important data"
echo "$NEW_DATA" > file.txt.tmp && mv file.txt.tmp file.txt
```

### File Information
```bash
FILE="example.txt"

# Using stat command (Linux)
stat "$FILE"

# File size
SIZE=$(stat -c%s "$FILE" 2>/dev/null)      # Linux
SIZE=$(stat -f%z "$FILE" 2>/dev/null)      # macOS
# Or use wc
SIZE=$(wc -c < "$FILE")

# Last modified time
MTIME=$(stat -c%Y "$FILE" 2>/dev/null)     # Linux (timestamp)
MTIME=$(stat -f%m "$FILE" 2>/dev/null)     # macOS (timestamp)

# File permissions (octal)
PERMS=$(stat -c%a "$FILE" 2>/dev/null)     # Linux
PERMS=$(stat -f%Lp "$FILE" 2>/dev/null)   # macOS
```

---

**Tutorial Complete!** You now have a comprehensive Bash mastery guide from beginner to ninja level, covering all essential topics for professional Bash scripting and DevOps automation.