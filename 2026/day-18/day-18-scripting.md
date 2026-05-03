# Day 18 – Shell Scripting: Functions & Intermediate Concepts

## Task Overview

In Day 18, I learned how to write cleaner and reusable shell scripts using functions and safer scripting patterns.

This task focuses on:

- Writing and calling functions
- Passing arguments to functions
- Using return values and exit codes
- Using local variables
- Using strict mode with `set -euo pipefail`
- Creating a real-world system information reporter script

## Folder Structure
2026/
└── day-18/
    ├── functions.sh
    ├── disk_check.sh
    ├── strict_demo.sh
    ├── local_demo.sh
    ├── system_info.sh
    └── day-18-scripting.md

# Task 1: Basic Functions

## What is a Function?

A function is a reusable block of code.

Instead of writing the same commands again and again, we can place those commands inside a function and call the function whenever needed.

Function syntax:

function_name() {
    commands
}

Example:

greet() {
    echo "Hello"
}

Call the function:

greet

## Script: functions.sh

## Requirement

Create `functions.sh` with:

- A function `greet` that takes a name as an argument and prints `Hello, <name>!`
- A function `add` that takes two numbers and prints their sum
- Call both functions from the script

#!/bin/bash

greet() {
    echo "Hello, $1!"
}

add() {
    local NUM1=$1
    local NUM2=$2
    local SUM=$((NUM1 + NUM2))

    echo "Sum of $NUM1 and $NUM2 is $SUM"
}

greet "Hardik"
add 10 20

## Explanation

greet() {
    echo "Hello, $1!"
}

This creates a function named `greet`.

Inside the function:

$1

means the first argument passed to the function.

Example:

greet "Hardik"

Here, `Hardik` becomes `$1`.

add() {
    local NUM1=$1
    local NUM2=$2
    local SUM=$((NUM1 + NUM2))

    echo "Sum of $NUM1 and $NUM2 is $SUM"
}

This function accepts two numbers.

Here:

- `$1` is the first number
- `$2` is the second number
- `$((NUM1 + NUM2))` performs arithmetic addition

## Commands Used

nano functions.sh
chmod +x functions.sh
./functions.sh

## Output

Hello, Hardik!
Sum of 10 and 20 is 30

# Task 2: Functions with Return Values

## Requirement

Create `disk_check.sh` with:

- A function `check_disk` that checks disk usage of `/` using `df -h`
- A function `check_memory` that checks free memory using `free -h`
- A main section that calls both and prints the results

## Script: disk_check.sh

## Code

#!/bin/bash

check_disk() {
    echo "Disk usage for root partition:"
    df -h /
}

check_memory() {
    echo "Memory usage:"
    free -h
}

main() {
    echo "===== System Resource Check ====="
    echo

    check_disk
    echo

    check_memory
    echo

    echo "Resource check completed."
}

main

## Explanation

check_disk() {
    df -h /
}

This function checks disk usage of the root `/` partition.

df -h /

Meaning:

- `df` shows disk usage
- `-h` shows output in human-readable format
- `/` means root partition

check_memory() {
    free -h
}

This function checks memory usage.

free -h

Meaning:

- `free` shows memory information
- `-h` shows output in human-readable format

main() {
    check_disk
    check_memory
}

The `main` function is used to organize the script flow.

At the end:

main

calls the main function.

## Commands Used

vim disk_check.sh
chmod +x disk_check.sh
./disk_check.sh

## Output Example

===== System Resource Check =====

Disk usage for root partition:
Filesystem      Size  Used Avail Use% Mounted on
/dev/root        30G   12G   18G  40% /

Memory usage:
               total        used        free      shared  buff/cache   available
Mem:           7.7Gi       2.0Gi       3.5Gi       120Mi       2.2Gi       5.3Gi
Swap:          2.0Gi          0B       2.0Gi

Resource check completed.

Note: Your output may be different depending on your system.


# Task 3: Strict Mode — set -euo pipefail

## What is Strict Mode?

Strict mode makes shell scripts safer.

It helps detect errors early and prevents scripts from continuing silently after failures.

Strict mode:

set -euo pipefail

Usually placed after shebang:

#!/bin/bash
set -euo pipefail

## Meaning of Each Flag

## set -e

set -e means exit immediately if any command fails.

Example:

wrongcommand
echo "This will not run"

If `wrongcommand` fails, the script exits immediately.

## set -u

set -u means treat undefined variables as errors.

Example:

echo "$USERNAME"

If `USERNAME` is not defined, the script stops with an error.

## set -o pipefail

Normally, in a pipeline, Bash checks only the last command.

Example:

wrongcommand | grep test

Without `pipefail`, the script may not properly fail if the first command fails.

With `pipefail`, if any command in the pipeline fails, the full pipeline fails.

## Script: strict_demo.sh

## Requirement

Create `strict_demo.sh` with:

- `set -euo pipefail`
- Try using an undefined variable
- Try a command that fails
- Try a piped command where one part fails
- Document what each flag does

## Important Note

Because `set -euo pipefail` exits on errors, run each test one by one by uncommenting only one test at a time.

#!/bin/bash
set -euo pipefail

echo "Strict mode demo started."

# Test 1: set -u
# Uncomment below line to test undefined variable error.
# echo "Undefined variable value is: $UNDEFINED_VARIABLE"

# Test 2: set -e
# Uncomment below line to test command failure.
# wrongcommand

# Test 3: set -o pipefail
# Uncomment below line to test pipeline failure.
# wrongcommand | grep "test"

echo "Strict mode demo completed."

vim  strict_demo.sh
chmod +x strict_demo.sh
./strict_demo.sh

## Output When No Error Test Is Enabled

Strict mode demo started.
Strict mode demo completed.

## Output for set -u

If this line is uncommented:

echo "Undefined variable value is: $UNDEFINED_VARIABLE"

Output:

Strict mode demo started.
./strict_demo.sh: line 8: UNDEFINED_VARIABLE: unbound variable

Meaning:

set -u stops the script when an undefined variable is used.

## Output for `set -e`

If this line is uncommented:

wrongcommand

Output:

Strict mode demo started.
./strict_demo.sh: line 12: wrongcommand: command not found

Meaning:

`set -e` stops the script when a command fails.

## Output for `set -o pipefail`

If this line is uncommented:

wrongcommand | grep "test"
Output:

Strict mode demo started.
./strict_demo.sh: line 16: wrongcommand: command not found


Meaning:

`set -o pipefail` makes the full pipeline fail if any command inside the pipeline fails.

# Task 4: Local Variables

## What is a Local Variable?

A local variable is available only inside the function where it is created.

Syntax:

local VARIABLE_NAME="value"

Local variables help avoid accidental changes to variables outside the function.

## Requirement

Create `local_demo.sh` with:

- A function that uses `local` keyword for variables
- Show that local variables do not leak outside the function
- Compare with a function that uses regular variables

## Script: local_demo.sh


#!/bin/bash

local_variable_demo() {
    local MESSAGE="I am a local variable"
    echo "Inside local_variable_demo: $MESSAGE"
}

regular_variable_demo() {
    MESSAGE="I am a regular variable"
    echo "Inside regular_variable_demo: $MESSAGE"
}

echo "Calling local variable function:"
local_variable_demo

echo
echo "Trying to access local variable outside function:"
echo "Outside function MESSAGE is: ${MESSAGE:-Not available}"

echo
echo "Calling regular variable function:"
regular_variable_demo

echo
echo "Accessing regular variable outside function:"
echo "Outside function MESSAGE is: $MESSAGE"

## Explanation

local MESSAGE="I am a local variable"

This variable exists only inside the function.

Outside the function, it is not available.

MESSAGE="I am a regular variable"

This is a normal/global variable.

It can be accessed outside the function after the function runs.

${MESSAGE:-Not available}

This means:

If `MESSAGE` is not set, print:

Not available

This prevents an error if the variable is empty or undefined.

## Commands Used

vim local_demo.sh
chmod +x local_demo.sh
./local_demo.sh

## Output

Calling local variable function:
Inside local_variable_demo: I am a local variable

Trying to access local variable outside function:
Outside function MESSAGE is: Not available

Calling regular variable function:
Inside regular_variable_demo: I am a regular variable

Accessing regular variable outside function:
Outside function MESSAGE is: I am a regular variable

# Task 5: Build a Script — System Info Reporter

## Requirement

Create `system_info.sh` that uses functions for everything:

- A function to print hostname and OS info
- A function to print uptime
- A function to print disk usage, top 5 by size
- A function to print memory usage
- A function to print top 5 CPU-consuming processes
- A main function that calls all functions with section headers
- Use `set -euo pipefail` at the top
- Output should look clean and readable

## Script: system_info.sh

#!/bin/bash
set -euo pipefail

print_header() {
    local TITLE=$1
    echo
    echo "========================================"
    echo "$TITLE"
    echo "========================================"
}

print_host_info() {
    print_header "Hostname and OS Information"

    echo "Hostname: $(hostname)"

    if [ -f /etc/os-release ]; then
        echo "OS Information:"
        cat /etc/os-release | grep -E "PRETTY_NAME|VERSION_ID"
    else
        echo "OS information file not found."
    fi
}

print_uptime() {
    print_header "System Uptime"
    uptime
}

print_disk_usage() {
    print_header "Top 5 Disk Usage by Size"

    df -h | sort -hr -k 2 | head -n 5
}

print_memory_usage() {
    print_header "Memory Usage"

    free -h
}

print_top_cpu_processes() {
    print_header "Top 5 CPU Consuming Processes"

    ps -eo pid,comm,%cpu,%mem --sort=-%cpu | head -n 6
}

main() {
    echo "System Information Report"
    echo "Generated on: $(date)"

    print_host_info
    print_uptime
    print_disk_usage
    print_memory_usage
    print_top_cpu_processes

    echo
    echo "System information report completed."
}

main

## Explanation

## Shebang and Strict Mode

#!/bin/bash
set -euo pipefail

The script uses Bash and strict mode for safety.

## Header Function

print_header() {
    local TITLE=$1
}

This function prints clean section headers.

Using one header function avoids repeating the same formatting code again and again.


## Host Info Function

print_host_info()

This function prints:

- Hostname
- OS information from `/etc/os-release`

## Uptime Function

print_uptime()

This function shows how long the system has been running.


## Disk Usage Function

print_disk_usage()

This function shows disk usage and displays the top 5 entries sorted by size.

## Memory Usage Function

print_memory_usage()

This function shows memory usage using:

free -h

## CPU Process Function

print_top_cpu_processes()

This function prints top CPU-consuming processes using:

ps -eo pid,comm,%cpu,%mem --sort=-%cpu | head -n 6

`head -n 6` is used because the first line is the header.


## Main Function

main() {
    print_host_info
    print_uptime
    print_disk_usage
    print_memory_usage
    print_top_cpu_processes
}

The `main` function controls the script flow.

At the end:

main

calls the main function.

## Commands Used

nano system_info.sh
chmod +x system_info.sh
./system_info.sh

## Output Example

System Information Report
Generated on: Tue Jan 21 10:30:00 UTC 2026

========================================
Hostname and OS Information
========================================
Hostname: devops-server
OS Information:
PRETTY_NAME="Ubuntu 22.04.4 LTS"
VERSION_ID="22.04"

========================================
System Uptime
========================================
 10:30:12 up 2 days,  4:15,  1 user,  load average: 0.08, 0.04, 0.01

========================================
Top 5 Disk Usage by Size
========================================
Filesystem      Size  Used Avail Use% Mounted on
/dev/root        30G   12G   18G  40% /
tmpfs           3.9G     0  3.9G   0% /dev/shm

========================================
Memory Usage
========================================
               total        used        free      shared  buff/cache   available
Mem:           7.7Gi       2.1Gi       3.4Gi       120Mi       2.2Gi       5.2Gi
Swap:          2.0Gi          0B       2.0Gi

========================================
Top 5 CPU Consuming Processes
========================================
    PID COMMAND         %CPU %MEM
   1234 chrome           5.5  2.1
    900 dockerd          2.0  1.4
    700 containerd       1.2  0.8
      1 systemd          0.5  0.2
   1400 bash             0.1  0.1

System information report completed.
