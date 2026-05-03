# Day 16 – Shell Scripting Basics

## Task Overview

# In this task, I started learning the basics of shell scripting.

I learned:
- Shebang line `#!/bin/bash`
- Variables
- `echo`
- `read`
- Basic if-else conditions
- File checking using `-f`
- Service status checking using `systemctl`

## Folder Structure
2026/
└── day-16/
    ├── hello.sh
    ├── variables.sh
    ├── greet.sh
    ├── check_number.sh
    ├── file_check.sh
    ├── server_check.sh
    └── day-16-shell-scripting.md
    
# Task 1: Your First Script
File: hello.sh
#!/bin/bash

echo "Hello, DevOps!"
Commands Used
chmod +x hello.sh
./hello.sh
Output
Hello, DevOps!

# What happens if the shebang line is removed?

# If the shebang line #!/bin/bash is removed, the script may still run in some cases because the current shell may execute it.

# However, it is not reliable.

# The shebang tells the system which interpreter should run the script.

For example:

#!/bin/bash

means the script should be executed using Bash.

Without the shebang, Bash-specific commands may fail if another shell executes the script.

# Task 2: Variables

File: variables.sh

#!/bin/bash

NAME="Hardik"
ROLE="DevOps Engineer"

echo "Hello, I am $NAME and I am a $ROLE"

echo "Using double quotes:"
echo "Hello, I am $NAME and I am a $ROLE"

echo "Using single quotes:"
echo 'Hello, I am $NAME and I am a $ROLE'
Commands Used
chmod +x variables.sh
./variables.sh

# Output
Hello, I am Hardik and I am a DevOps Engineer
Using double quotes:
Hello, I am Hardik and I am a DevOps Engineer
Using single quotes:
Hello, I am $NAME and I am a $ROLE

# Single Quotes vs Double Quotes
Double quotes allow variable expansion.
# Example:
echo "Hello $NAME"
Output:
Hello Hardik

# Single quotes print the text exactly as written.
Example:
echo 'Hello $NAME'
Output:
Hello $NAME

# Task 3: User Input with read
File: greet.sh

#!/bin/bash

read -p "Enter your name: " NAME
read -p "Enter your favourite tool: " TOOL

echo "Hello $NAME, your favourite tool is $TOOL"
# Commands Used
chmod +x greet.sh
./greet.sh

# Output
Enter your name: Hardik
Enter your favourite tool: Docker
Hello Hardik, your favourite tool is Docker

# Task 4: If-Else Conditions

File: check_number.sh

#!/bin/bash
read -p "Enter a number: " NUMBER
if [ "$NUMBER" -gt 0 ]; then
    echo "$NUMBER is positive"
elif [ "$NUMBER" -lt 0 ]; then
    echo "$NUMBER is negative"
else
    echo "The number is zero"
fi

# Commands Used
chmod +x check_number.sh
./check_number.sh

Output 1
Enter a number: 10
10 is positive
Output 2
Enter a number: -5
-5 is negative
Output 3
Enter a number: 0
The number is zero

# File: file_check.sh

#!/bin/bash
read -p "Enter filename: " FILENAME
if [ -f "$FILENAME" ]; then
    echo "File '$FILENAME' exists."
else
    echo "File '$FILENAME' does not exist."
fi

# Commands Used
chmod +x file_check.sh
./file_check.sh

# Output 1
Enter filename: hello.sh
File 'hello.sh' exists.
# Output 2
Enter filename: test.txt
File 'test.txt' does not exist.

# Task 5: Combine It All
File: server_check.sh
#!/bin/bash
SERVICE="nginx"

read -p "Do you want to check the status? (y/n): " ANSWER

if [ "$ANSWER" = "y" ]; then
    systemctl status "$SERVICE"

    if systemctl is-active --quiet "$SERVICE"; then
        echo "$SERVICE is active."
    else
        echo "$SERVICE is not active."
    fi

elif [ "$ANSWER" = "n" ]; then
    echo "Skipped."
else
    echo "Invalid input. Please enter y or n."
fi

# Commands Used
chmod +x server_check.sh
./server_check.sh

# Output 1
Do you want to check the status? (y/n): y
nginx is active.
# Output 2
Do you want to check the status? (y/n): y
nginx is not active.
# Output 3
Do you want to check the status? (y/n): n
Skipped.

# What I Learned
The shebang line #!/bin/bash tells the system which interpreter should run the script.
Variables are used to store values, and we can access them using $VARIABLE_NAME.
If-else conditions help shell scripts make decisions based on input, numbers, files, or service status.
