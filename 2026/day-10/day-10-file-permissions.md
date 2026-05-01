# Day 10 – File Permissions & File Operations 

# PART 1 – DEEP CONCEPT (MUST UNDERSTAND FIRST)

# What are File Permissions?
Every file in Linux has access control.
-rwxrwxrwx
Breakdown:
# Section	Meaning
rwx	Owner
rwx	Group
rwx	Others

# Permission Values (VERY IMPORTANT)
Permission	Value
r (read)	= 4
w (write)	= 2
x (execute)	= 1

# Example:
rwx = 7
rw- = 6
r-- = 4

# Real DevOps Use Case
File Type	Permission
Script	755
Config	644
Sensitive file	600

# PART 2 – HANDS-ON TASK (STEP-BY-STEP)
# 🔹 TASK 1 – Create Files
# Step 1: Create empty file
touch devops.txt

# Step 2: Create file with content
echo "This is DevOps practice" > notes.txt

# Step 3: Create script file
vim script.sh
Add:
echo "Hello DevOps"
Save:
ESC → :wq

# Step 4: Verify
ls -l
Example output:

-rw-r--r-- devops.txt
-rw-r--r-- notes.txt
-rw-r--r-- script.sh

# 🔹 TASK 2 – Read Files
Read full file
cat notes.txt
Read script in read-only
vim -R script.sh
First 5 lines
head -n 5 /etc/passwd
Last 5 lines
tail -n 5 /etc/passwd

# 🔹 TASK 3 – Understand Permissions
Check permissions
ls -l devops.txt notes.txt script.sh
Example:
-rw-r--r--
Meaning:
Role	Permission
Owner	read + write
Group	read
Others	read
Answer:
Owner → can read/write
Group → can read
Others → can read
No one can execute

# 🔹 TASK 4 – Modify Permissions
1. Make script executable
chmod +x script.sh
Run:
./script.sh
2. Make devops.txt read-only
chmod a-w devops.txt

# Removes write from all
4. Set notes.txt to 640
chmod 640 notes.txt
Meaning:
Role	Permission
Owner	rw
Group	r
Others	none

# 4. Create directory
mkdir project
Set permission:
chmod 755 project
Verify everything
ls -l
ls -ld project

# 🔹 TASK 5 – Test Permissions
# Test 1 – Write to read-only file
echo "test" >> devops.txt
Error:
Permission denied
# Test 2 – Run file without execute
Remove execute:
chmod -x script.sh
./script.sh
Error:
Permission denied
# PART 3 – REAL-WORLD SCENARIOS
# Scenario 1 – Script not running
Fix:
chmod +x script.sh

# Scenario 2 – Config file accidentally modified
Fix:
chmod 644 config.conf

# Scenario 3 – Sensitive file exposed
Fix:
chmod 600 secrets.txt

# Scenario 4 – App can't write logs
Fix:
chmod 775 /var/log/app

# PART 4 – INTERVIEW QUESTIONS
Q1: What is chmod?
Command to change file permissions

Q2: What is 755?
Owner: rwx
Group: r-x
Others: r-x

Q3: Difference between 644 and 600?
Permission	Meaning
644	Public read
600	Private

Q4: Why script not executing?
Missing execute permission

Q5: How to remove write permission?
chmod -w file
