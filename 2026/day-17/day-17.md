# Day 17 – Shell Scripting: Loops, Arguments & Error Handling

## Task Overview

In Day 17, I learned advanced basics of shell scripting.

This task focuses on:
- `for` loops
- `while` loops
- Command-line arguments
- Installing packages using a shell script
- Basic error handling
- Root user validation


## Folder Structure

```text
2026/
└── day-17/
    ├── for_loop.sh
    ├── count.sh
    ├── countdown.sh
    ├── greet.sh
    ├── args_demo.sh
    ├── install_packages.sh
    ├── safe_script.sh
    └── day-17-scripting.md

# Task 1: For Loop

## What is a For Loop?

A `for` loop is used when we want to repeat a task multiple times.

For example, if we want to print multiple fruits one by one, we can use a `for` loop instead of writing many `echo` commands.


## Script 1: for_loop.sh

### Requirement

Create `for_loop.sh` that loops through a list of 5 fruits and prints each one.

#!/bin/bash

for fruit in apple banana mango orange grapes
do
    echo "Fruit: $fruit"
done

## Explanation
for fruit in apple banana mango orange grapes
This line starts the loop.
Here:

- `fruit` is a variable.
- `apple banana mango orange grapes` is the list.
- The loop runs once for every item in the list.
do
    echo "Fruit: $fruit"
done
The commands between `do` and `done` run repeatedly.

## Commands Used
vim  for_loop.sh
chmod +x for_loop.sh
./for_loop.sh

## Output
Fruit: apple
Fruit: banana
Fruit: mango
Fruit: orange
Fruit: grapes

## Script 2: count.sh

### Requirement
Create `count.sh` that prints numbers 1 to 10 using a `for` loop.

#!/bin/bash

for number in {1..10}
do
    echo "$number"
done

## Explanation

for number in {1..10}

This loop starts from `1` and ends at `10`.

Each number is stored in the `number` variable one by one.


## Commands Used
vim  count.sh
chmod +x count.sh
./count.sh

## Output
1
2
3
4
5
6
7
8
9
10

# Task 2: While Loop

## What is a While Loop?

A while loop runs as long as the condition is true.

It is useful when we do not know exactly how many times the loop should run.

## Script: countdown.sh

### Requirement

Create countdown.sh that:

- Takes a number from the user
- Counts down to `0` using a `while` loop
- Prints `"Done!"` at the end

#!/bin/bash

read -p "Enter a number: " NUMBER

while [ "$NUMBER" -ge 0 ]
do
    echo "$NUMBER"
    NUMBER=$((NUMBER - 1))
done

echo "Done!"

## Explanation

read -p "Enter a number: " NUMBER
This takes input from the user and stores it in the `NUMBER` variable.
while [ "$NUMBER" -ge 0 ]
This means the loop will run while `NUMBER` is greater than or equal to `0`.
NUMBER=$((NUMBER - 1))
This decreases the number by `1` after every loop.

## Commands Used
vim  countdown.sh
chmod +x countdown.sh
./countdown.sh

## Output

Example:
Enter a number: 5
5
4
3
2
1
0
Done!

# Task 3: Command-Line Arguments

## What are Command-Line Arguments?

Command-line arguments are values passed to a script while running it.

## Script 1: greet.sh
### Requirement
Create greet.sh that:
- Accepts a name as `$1`
- Prints `Hello, <name>!`
- If no argument is passed, prints usage message

#!/bin/bash

if [ $# -eq 0 ]; then
    echo "Usage: ./greet.sh <name>"
    exit 1
fi

NAME=$1

echo "Hello, $NAME!"

## Explanation

if [ $# -eq 0 ]; then

This checks if no argument is passed.

`$#` means total number of arguments.

NAME=$1

This stores the first argument in the `NAME` variable.

## Commands Used
vim  greet.sh
chmod +x greet.sh
./greet.sh Hardik

## Output
Hello, Hardik!

## Output When No Argument Is Passed

Command:

./greet.sh

# Output:
Usage: ./greet.sh <name>

## Script 2: args_demo.sh

### Requirement

Create `args_demo.sh` that:

- Prints total number of arguments using `$#`
- Prints all arguments using `$@`
- Prints the script name using `$0`

#!/bin/bash

echo "Script name: $0"
echo "Total number of arguments: $#"
echo "All arguments: $@"

## Explanation
$0
Prints the script name.
$#
Prints total number of arguments.
$@
Prints all arguments.

## Commands Used
vim args_demo.sh
chmod +x args_demo.sh
./args_demo.sh Linux Docker Kubernetes

## Output
Script name: ./args_demo.sh
Total number of arguments: 3
All arguments: Linux Docker Kubernetes

# Task 4: Install Packages via Script

## Requirement

Create `install_packages.sh` that:

- Defines a list of packages: `nginx`, `curl`, `wget`
- Loops through the list
- Checks if each package is installed
- Installs it if missing
- Skips if already installed
- Prints status for each package
- Checks if script is running as root
- Exits with a message if not root

## Important Note

This script should be run as root.

Use:
sudo ./install_packages.sh
or switch to root:
sudo -i

#!/bin/bash

if [ "$EUID" -ne 0 ]; then
    echo "Please run this script as root."
    echo "Example: sudo ./install_packages.sh"
    exit 1
fi

PACKAGES="nginx curl wget"

for package in $PACKAGES
do
    echo "Checking package: $package"

    if dpkg -s "$package" &> /dev/null; then
        echo "$package is already installed."
    else
        echo "$package is not installed. Installing now..."
        apt install "$package" -y
        echo "$package installation completed."
    fi

    echo "-----------------------------"
done
## Explanation

if [ "$EUID" -ne 0 ]; then

This checks if the script is running as root.

`EUID` means effective user ID.

Root user ID is always:

0

So if `EUID` is not equal to `0`, the script exits.

PACKAGES="nginx curl wget"


This stores package names in a variable.

for package in $PACKAGES

This loops through every package.

dpkg -s "$package" &> /dev/null

This checks if the package is installed.

If package is installed, it skips installation.

If package is missing, it installs the package using:

apt install "$package" -y

## Commands Used

vim install_packages.sh
chmod +x install_packages.sh
sudo ./install_packages.sh

## Output Example

If package is already installed:

Checking package: nginx
nginx is already installed.

If package is missing:

Checking package: wget
wget is not installed. Installing now...
wget installation completed.

## Output If Not Running as Root

Command:
./install_packages.sh

# Output:

Please run this script as root.
Example: sudo ./install_packages.sh

# Task 5: Error Handling

## What is Error Handling?

Error handling means handling failures properly in a script.

A script should not fail silently.

It should print a clear message when something goes wrong.

## Important Error Handling Concepts

| Concept | Meaning |
|---|---|
| `set -e` | Exit immediately if any command fails |
| `||` | Run next command only if previous command fails |
| `exit 1` | Exit script with failure status |
| `exit 0` | Exit script with success status |

## Script: safe_script.sh

### Requirement

Create `safe_script.sh` that:

- Uses `set -e`
- Tries to create directory `/tmp/devops-test`
- Tries to navigate into it
- Creates a file inside
- Uses `||` operator to print an error if any step fails

#!/bin/bash

set -e

mkdir /tmp/devops-test || echo "Directory already exists"

cd /tmp/devops-test || {
    echo "Failed to enter directory"
    exit 1
}

touch test-file.txt || {
    echo "Failed to create file"
    exit 1
}

echo "File created successfully inside /tmp/devops-test"

## Explanation

set -e


This makes the script exit immediately if a command fails.

mkdir /tmp/devops-test || echo "Directory already exists"

If directory creation fails, the message is printed.

If the directory already exists, it prints:

Directory already exists

cd /tmp/devops-test || {
    echo "Failed to enter directory"
    exit 1
}

If the script cannot enter the directory, it prints an error and exits.

touch test-file.txt || {
    echo "Failed to create file"
    exit 1
}

If file creation fails, it prints an error and exits.

## Commands Used

vim safe_script.sh
chmod +x safe_script.sh
./safe_script.sh

## Output First Time

File created successfully inside /tmp/devops-test

## Output If Directory Already Exists
Directory already exists
File created successfully inside /tmp/devops-test
