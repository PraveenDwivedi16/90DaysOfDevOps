# Shell Scripting Cheat Sheet
A quick reference guide for Shell scripting basics, conditions, loops, functions, text processing, error handling, and useful DevOps one-liners.
# Quick Reference Table

| Topic | Key Syntax | Example |
|---|---|---|
| Shebang | `#!/bin/bash` | `#!/bin/bash` |
| Run Script | `chmod +x script.sh` | `./script.sh` |
| Variable | `VAR="value"` | `NAME="DevOps"` |
| Use Variable | `$VAR` | `echo "$NAME"` |
| Read Input | `read -p "msg" VAR` | `read -p "Name: " NAME` |
| Argument | `$1`, `$2` | `./script.sh arg1` |
| Argument Count | `$#` | `echo "$#"` |
| All Arguments | `$@` | `echo "$@"` |
| Exit Code | `$?` | `echo "$?"` |
| If | `if [ condition ]; then` | `if [ -f file.txt ]; then` |
| For Loop | `for i in list; do` | `for i in 1 2 3; do echo "$i"; done` |
| While Loop | `while [ condition ]; do` | `while [ "$n" -gt 0 ]; do` |
| Function | `name() { ... }` | `greet() { echo "Hi"; }` |
| Local Variable | `local VAR="value"` | `local NAME="DevOps"` |
| Grep | `grep pattern file` | `grep -i "error" log.txt` |
| Awk | `awk '{print $1}' file` | `awk -F: '{print $1}' /etc/passwd` |
| Sed | `sed 's/old/new/g' file` | `sed -i 's/foo/bar/g' config.txt` |
| Find Old Files | `find path -mtime +N` | `find /tmp -mtime +7` |
| Strict Mode | `set -euo pipefail` | Use after shebang |
# Task 1: Basics
## Shebang
The shebang tells Linux which interpreter should run the script.
#!/bin/bash
Example:
#!/bin/bash
echo "Hello, DevOps!"
Why it matters:
Without shebang, the system may not know which shell should execute the script.
## Running a Script
Make script executable:
chmod +x script.sh
Run script directly:
./script.sh
Run script using Bash:
bash script.sh
Difference:
./script.sh uses the shebang.
bash script.sh runs the file directly with Bash.
## Comments
Single-line comment:
# This is a comment
echo "Hello"
Inline comment:
echo "Starting backup" # Print backup message
Comments help explain code but are ignored during execution.
## Variables
Declare variable:
NAME="DevOps"
Use variable:
echo "$NAME"
Correct:
ROLE="DevOps Engineer"
echo "I am a $ROLE"
Wrong:
ROLE = "DevOps Engineer"
There should be no spaces around `=`.
## Quoting Variables
Double quotes allow variable expansion:
NAME="Hardik"
echo "Hello $NAME"
Output:
Hello Hardik
Single quotes print text exactly:
echo 'Hello $NAME'
Output:
Hello $NAME
Best practice:
echo "$NAME"
Always quote variables to avoid issues with spaces.
## Reading User Input

Use `read` to take input from the user.
read -p "Enter your name: " NAME
echo "Hello $NAME"
Example output:
Enter your name: Hardik
Hello Hardik
## Command-Line Arguments

Arguments are values passed while running the script.

Run:
./script.sh Hardik DevOps
Use inside script:
echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "Total arguments: $#"
echo "All arguments: $@"
echo "Last command exit code: $?"

Meaning:

| Syntax | Meaning |
|---|---|
| `$0` | Script name |
| `$1` | First argument |
| `$2` | Second argument |
| `$#` | Total number of arguments |
| `$@` | All arguments |
| `$?` | Exit code of last command |

# Task 2: Operators and Conditionals

## String Comparisons

| Operator | Meaning | Example |
|---|---|---|
| `=` | Equal | `[ "$NAME" = "DevOps" ]` |
| `!=` | Not equal | `[ "$NAME" != "Admin" ]` |
| `-z` | Empty string | `[ -z "$NAME" ]` |
| `-n` | Non-empty string | `[ -n "$NAME" ]` |

Example:
NAME="DevOps"

if [ "$NAME" = "DevOps" ]; then
    echo "Name matched"
fi
Check empty string:
NAME=""

if [ -z "$NAME" ]; then
    echo "Name is empty"
fi
Check non-empty string:
NAME="Hardik"

if [ -n "$NAME" ]; then
    echo "Name is not empty"
fi

## Integer Comparisons

| Operator | Meaning | Example |
|---|---|---|
| `-eq` | Equal | `[ "$A" -eq "$B" ]` |
| `-ne` | Not equal | `[ "$A" -ne "$B" ]` |
| `-lt` | Less than | `[ "$A" -lt "$B" ]` |
| `-gt` | Greater than | `[ "$A" -gt "$B" ]` |
| `-le` | Less than or equal | `[ "$A" -le "$B" ]` |
| `-ge` | Greater than or equal | `[ "$A" -ge "$B" ]` |

Example:
NUMBER=10

if [ "$NUMBER" -gt 0 ]; then
    echo "Positive number"
fi
## File Test Operators

| Operator | Meaning | Example |
|---|---|---|
| `-f` | File exists and is regular file | `[ -f file.txt ]` |
| `-d` | Directory exists | `[ -d /tmp ]` |
| `-e` | File or directory exists | `[ -e path ]` |
| `-r` | File is readable | `[ -r file.txt ]` |
| `-w` | File is writable | `[ -w file.txt ]` |
| `-x` | File is executable | `[ -x script.sh ]` |
| `-s` | File exists and is not empty | `[ -s file.txt ]` |

Example:
if [ -f "app.log" ]; then
    echo "File exists"
else
    echo "File not found"
fi
Directory check:
if [ -d "/var/log" ]; then
    echo "Directory exists"
fi

Executable check:
if [ -x "deploy.sh" ]; then
    echo "Script is executable"
fi

## if, elif, else Syntax
if [ condition ]; then
    command
elif [ another_condition ]; then
    command
else
    command
fi
Example:
read -p "Enter number: " NUMBER

if [ "$NUMBER" -gt 0 ]; then
    echo "Positive"
elif [ "$NUMBER" -lt 0 ]; then
    echo "Negative"
else
    echo "Zero"
fi
## Logical Operators

| Operator | Meaning | Example |
|---|---|---|
| `&&` | AND / run next only if previous succeeds | `mkdir test && cd test` |
| `||` | OR / run next only if previous fails | `mkdir test || echo "Failed"` |
| `!` | NOT | `[ ! -f file.txt ]` |

Example using `&&`:
[ -f app.log ] && echo "Log file exists"
Example using `||`:
[ -f app.log ] || echo "Log file missing"

Example using `!`:
if [ ! -d "/backup" ]; then
    mkdir /backup
fi
## Case Statements

Use `case` when checking multiple possible values.

Syntax:
case "$VAR" in
    value1)
        command
        ;;
    value2)
        command
        ;;
    *)
        default_command
        ;;
esac
Example:
read -p "Enter environment: " ENV

case "$ENV" in
    dev)
        echo "Development environment"
        ;;
    staging)
        echo "Staging environment"
        ;;
    prod)
        echo "Production environment"
        ;;
    *)
        echo "Unknown environment"
        ;;
esac

# Task 3: Loops

## for Loop – List Based
Use when looping over a known list.
for fruit in apple banana mango
do
    echo "$fruit"
done

Output:
apple
banana
mango
## for Loop – Range Based
for number in {1..5}
do
    echo "$number"
done
Output:
1
2
3
4
5
## for Loop – C Style
for (( i=1; i<=5; i++ ))
do
    echo "$i"
done
Output:
1
2
3
4
5
## while Loop
Runs while the condition is true.
COUNT=5

while [ "$COUNT" -gt 0 ]
do
    echo "$COUNT"
    COUNT=$((COUNT - 1))
done
## until Loop
Runs until the condition becomes true.
COUNT=1

until [ "$COUNT" -gt 5 ]
do
    echo "$COUNT"
    COUNT=$((COUNT + 1))
done

## Loop Control – break
break stops the loop.
for i in 1 2 3 4 5
do
    if [ "$i" -eq 3 ]; then
        break
    fi
    echo "$i"
done
Output:
1
2
## Loop Control – continue

continue skips the current iteration.
for i in 1 2 3 4 5
do
    if [ "$i" -eq 3 ]; then
        continue
    fi
    echo "$i"
done
Output:
1
2
4
5

## Looping Over Files
for file in *.log
do
    echo "Processing $file"
done
Useful for log processing, backups, and file cleanup.
## Looping Over Command Output

Use `while read` for line-by-line processing.
cat users.txt | while read line
do
    echo "User: $line"
done
Better version:
while read line
do
    echo "User: $line"
done < users.txt
# Task 4: Functions

## Defining a Function

```bash
greet() {
    echo "Hello"
}

## Calling a Function
greet

Full example:
#!/bin/bash

greet() {
    echo "Hello, DevOps!"
}

greet
## Passing Arguments to Functions

Inside functions, `$1`, `$2`, etc. represent function arguments.
greet() {
    echo "Hello, $1"
}

greet "Hardik"
Output:
Hello, Hardik

## Function with Multiple Arguments
add() {
    SUM=$(($1 + $2))
    echo "$SUM"
}
add 10 20
Output:
30
## Return Values – return vs echo
Use `return` for exit status.
check_file() {
    if [ -f "$1" ]; then
        return 0
    else
        return 1
    fi
}

check_file "app.log"

if [ $? -eq 0 ]; then
    echo "File exists"
else
    echo "File missing"
fi

Use `echo` to return actual data.
get_hostname() {
    echo "$(hostname)"
}

HOST=$(get_hostname)
echo "Hostname is $HOST"
## Local Variables
Use `local` inside functions to avoid variable conflicts.
demo() {
    local MESSAGE="Inside function"
    echo "$MESSAGE"
}

demo
Example showing local variable:
NAME="Global"

show_name() {
    local NAME="Local"
    echo "Inside function: $NAME"
}

show_name
echo "Outside function: $NAME"
Output:
Inside function: Local
Outside function: Global
# Task 5: Text Processing Commands
## grep
Used to search patterns in files.
Basic search:
grep "ERROR" app.log
Case-insensitive search:
grep -i "error" app.log
Recursive search:
grep -r "TODO" .
Count matches:
grep -c "ERROR" app.log
Show line numbers:
grep -n "CRITICAL" app.log
Invert match:
grep -v "INFO" app.log
Extended regex:
grep -E "ERROR|Failed" app.log
## awk
Used for column extraction and text processing.
Print first column:
awk '{print $1}' file.txt
Print first and third column:
awk '{print $1, $3}' file.txt
Use custom field separator:
awk -F: '{print $1}' /etc/passwd
Pattern matching:
awk '/ERROR/ {print}' app.log
BEGIN block:
awk 'BEGIN {print "Start"} {print $1}' file.txt
END block:
awk '{count++} END {print count}' file.txt
Sum column:
awk '{sum += $1} END {print sum}' numbers.txt
## sed
Used to edit or transform text.
Substitute first match per line:
sed 's/old/new/' file.txt
Substitute all matches:
sed 's/old/new/g' file.txt
In-place edit:
sed -i 's/foo/bar/g' config.txt
Delete lines containing pattern:
sed '/DEBUG/d' app.log
Print specific line:
sed -n '5p' file.txt
Print line range:
sed -n '10,20p' file.txt
## cut
Used to extract columns.
Extract first column by comma:
cut -d',' -f1 data.csv
Extract first and third column:
cut -d',' -f1,3 data.csv
Extract characters:
cut -c1-5 file.txt
Example with `/etc/passwd`:
cut -d':' -f1 /etc/passwd
## sort
Sort alphabetically:
sort names.txt
Sort numerically:
sort -n numbers.txt
Reverse sort:
sort -r names.txt
Numerical reverse sort:
sort -rn numbers.txt
Sort and remove duplicates:
sort -u names.txt
## uniq
Remove duplicate adjacent lines:
uniq file.txt
Count duplicates:
uniq -c file.txt
Sort before uniq:
sort file.txt | uniq -c
Find duplicate lines only:
sort file.txt | uniq -d
## tr
Translate lowercase to uppercase:
echo "devops" | tr 'a-z' 'A-Z'
Output:
DEVOPS
Delete characters:
echo "hello123" | tr -d '0-9'
Output:
hello
Replace spaces with new lines:
echo "one two three" | tr ' ' '\n'
## wc
Count lines:
wc -l file.txt
Count words:
wc -w file.txt
Count characters:
wc -c file.txt
Count lines only:
wc -l < file.txt

## head
Show first 10 lines:
head file.txt
Show first 5 lines:
head -n 5 file.txt

## tail

Show last 10 lines:
tail file.txt
Show last 20 lines:
tail -n 20 file.txt
Follow live log:
tail -f app.log
Follow last 100 lines:

tail -n 100 -f app.log

# Task 6: Useful Patterns and One-Liners

## 1. Find and Delete Files Older Than N Days

Delete `.log` files older than 7 days:

find /var/log/myapp -type f -name "*.log" -mtime +7 -delete

Safer version, print first:

find /var/log/myapp -type f -name "*.log" -mtime +7


## 2. Compress Logs Older Than 7 Days

find /var/log/myapp -type f -name "*.log" -mtime +7 -exec gzip {} \;


## 3. Count Lines in All `.log` Files

wc -l *.log

Recursive:

find . -type f -name "*.log" -exec wc -l {} \;

## 4. Replace a String Across Multiple Files

Replace `old_url` with `new_url` in all `.conf` files:

sed -i 's/old_url/new_url/g' *.conf

Recursive version:

find . -type f -name "*.conf" -exec sed -i 's/old_url/new_url/g' {} \;

## 5. Check If a Service Is Running

systemctl is-active --quiet nginx && echo "nginx is running" || echo "nginx is stopped"

## 6. Monitor Disk Usage with Alert

df -h / | awk 'NR==2 {print $5}' | tr -d '%'

Script version:

USAGE=$(df / | awk 'NR==2 {print $5}' | tr -d '%')

if [ "$USAGE" -gt 80 ]; then
    echo "Disk usage is above 80%"
fi

## 7. Tail a Log and Filter Errors in Real Time

tail -f app.log | grep --line-buffered "ERROR"

## 8. Find Top 5 ERROR Messages
grep "ERROR" app.log | awk '{$1=$2=$3=""; print}' | sort | uniq -c | sort -rn | head -5

## 9. Parse CSV Column

Print first column from CSV:
awk -F',' '{print $1}' data.csv
Skip header:
awk -F',' 'NR>1 {print $1}' data.csv

## 10. Parse JSON from Command Line
Using `jq`:

cat response.json | jq '.name'
Example:
curl -s https://api.example.com/users | jq '.data'

## 11. Find Large Files
Find files larger than 100 MB:
find /var/log -type f -size +100M
## 12. Show Top 5 CPU Processes

ps -eo pid,comm,%cpu,%mem --sort=-%cpu | head -n 6

## 13. Show Top 5 Memory Processes

ps -eo pid,comm,%cpu,%mem --sort=-%mem | head -n 6

## 14. Backup a Directory

tar -czf backup-$(date +%Y-%m-%d).tar.gz /path/to/source

## 15. Count ERROR and Failed Lines

grep -Ei "ERROR|Failed" app.log | wc -l


# Task 7: Error Handling and Debugging

## Exit Codes

Every command returns an exit code.

echo $?

Common exit codes:

| Code | Meaning |
|---|---|
| `0` | Success |
| `1` | General failure |
| `2` | Misuse of command |

Example:

ls /tmp
echo "$?"

## exit 0

Use `exit 0` for successful script completion.

echo "Success"
exit 0

## exit 1

Use `exit 1` when the script fails.

echo "Error occurred"
exit 1

## Check Last Command Status

mkdir backup

if [ $? -eq 0 ]; then
    echo "Directory created"
else
    echo "Failed to create directory"
fi

Better version:

mkdir backup && echo "Directory created" || echo "Failed"

## set -e

Exit script immediately when a command fails.

#!/bin/bash
set -e

mkdir test
cd test
wrongcommand
echo "This will not run"
## set -u

Treat unset variables as errors.

#!/bin/bash
set -u

echo "$USERNAME"
If `USERNAME` is not defined, the script exits.

Safe default:

echo "${USERNAME:-Guest}"

## set -o pipefail

Fail the full pipeline if any command in the pipe fails.

#!/bin/bash
set -o pipefail

wrongcommand | grep "test"

Without `pipefail`, Bash may only check the last command.

## Strict Mode

Recommended for safer scripts:

#!/bin/bash
set -euo pipefail

Meaning:

| Flag | Meaning |
|---|---|
| `-e` | Exit on command failure |
| `-u` | Error on unset variable |
| `-o pipefail` | Fail pipeline if any command fails |

## set -x

Debug mode. Shows commands before execution.
#!/bin/bash
set -x

NAME="DevOps"
echo "$NAME"
Run debug without editing script:
bash -x script.sh

## Trap

trap runs cleanup commands when the script exits.

Example:
#!/bin/bash
set -euo pipefail

cleanup() {
    echo "Cleaning up temporary files"
    rm -f /tmp/demo-file
}

trap 'cleanup' EXIT

touch /tmp/demo-file
echo "Doing work..."
Useful for:
temporary files
lock files
cleanup before script exits

## Trap on Error
#!/bin/bash
set -euo pipefail

handle_error() {
    echo "Error occurred on line $1"
}

trap 'handle_error $LINENO' ERR

echo "Starting"
wrongcommand
echo "Done"

# Extra Practical Examples

## Validate Required Argument

if [ $# -ne 1 ]; then
    echo "Usage: ./script.sh <file>"
    exit 1
fi

## Check File Exists
FILE=$1

if [ ! -f "$FILE" ]; then
    echo "File does not exist: $FILE"
    exit 1
fi
## Check Directory Exists

DIR=$1

if [ ! -d "$DIR" ]; then
    echo "Directory does not exist: $DIR"
    exit 1
fi

## Check Root User

if [ "$EUID" -ne 0 ]; then
    echo "Please run as root"
    exit 1
fi

## Create Timestamped File
DATE=$(date +%Y-%m-%d)
FILE="report-$DATE.txt"

touch "$FILE"

## Write Logs with Timestamp

echo "$(date '+%Y-%m-%d %H:%M:%S') : Backup completed" >> app.log

## Read File Line by Line

while read line
do
    echo "$line"
done < file.txt

## Check Command Exists

if command -v docker &> /dev/null; then
    echo "Docker is installed"
else
    echo "Docker is not installed"
fi

## Install Package If Missing

Ubuntu/Debian:

PACKAGE="nginx"

if dpkg -s "$PACKAGE" &> /dev/null; then
    echo "$PACKAGE is already installed"
else
    sudo apt install "$PACKAGE" -y
fi

RHEL/CentOS/Amazon Linux:

PACKAGE="nginx"

if rpm -q "$PACKAGE" &> /dev/null; then
    echo "$PACKAGE is already installed"
else
    sudo yum install "$PACKAGE" -y
fi

# Mini Templates

## Basic Script Template

#!/bin/bash

echo "Script started"

echo "Script completed"

## Strict Script Template

#!/bin/bash
set -euo pipefail

echo "Script started"

echo "Script completed"

## Argument-Based Script Template

```bash
#!/bin/bash
set -euo pipefail

if [ $# -ne 1 ]; then
    echo "Usage: ./script.sh <argument>"
    exit 1
fi

INPUT=$1

echo "Input is: $INPUT"

## Function-Based Script Template

#!/bin/bash
set -euo pipefail

print_message() {
    local MESSAGE=$1
    echo "$MESSAGE"
}

main() {
    print_message "Script started"
    print_message "Script completed"
}

main

## Backup Script Template

```bash
#!/bin/bash
set -euo pipefail

SOURCE_DIR=$1
BACKUP_DIR=$2
DATE=$(date +%Y-%m-%d)
BACKUP_FILE="$BACKUP_DIR/backup-$DATE.tar.gz"

if [ ! -d "$SOURCE_DIR" ]; then
    echo "Source directory does not exist"
    exit 1
fi

mkdir -p "$BACKUP_DIR"

tar -czf "$BACKUP_FILE" "$SOURCE_DIR"

echo "Backup created: $BACKUP_FILE"

## Log Analyzer Template

#!/bin/bash
set -euo pipefail

LOG_FILE=$1

if [ ! -f "$LOG_FILE" ]; then
    echo "Log file not found"
    exit 1
fi

echo "Total lines:"
wc -l < "$LOG_FILE"

echo "Errors:"
grep -Ei "ERROR|Failed" "$LOG_FILE" | wc -l

echo "Critical events:"
grep -n "CRITICAL" "$LOG_FILE" || true

# Final Notes

This cheat sheet covers the shell scripting concepts learned from Day 16 to Day 20.

It includes:

- Basics
- Variables
- Inputs
- Arguments
- Conditions
- Loops
- Functions
- Text processing
- Error handling
- Debugging
- Real-world one-liners

This file can be used as a quick reference during DevOps practice and real project work.
