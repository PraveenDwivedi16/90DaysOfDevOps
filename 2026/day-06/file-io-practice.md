# Day 06 – Linux Fundamentals: Read and Write Text Files
- Objective
# The goal of this task is to understand how Linux handles text files using basic commands.

# You will practice:
Creating files
Writing data into files
Appending data
Reading files (full and partial)
Using tee for simultaneous output and write

# Why This Matters (DevOps Perspective)
In DevOps, almost everything is a text file:
Logs (/var/log/...)
Configuration files (.env, .conf)
Scripts (.sh)
Application data

# If you can read and write files efficiently, you can:
Debug faster
Modify configs safely
Automate workflows
Analyze logs quickly
Core Concepts 

# 1. What is a File in Linux?
A file in Linux is a collection of data stored on disk.
Everything in Linux is treated as a file:
Regular files
Directories
Devices
Logs

# 2. File Creation
Command:
touch notes.txt
Explanation:
Creates an empty file
If file exists → updates timestamp

# 3. Writing to a File (>)
Command:
echo "Line 1" > notes.txt
Explanation:
Writes content to file
Overwrites existing content

# Important:
If file has data → it will be replaced

# 4. Appending to a File (>>)
Command:
echo "Line 2" >> notes.txt
Explanation:
Adds new content to existing file
Does NOT remove old data

# 5. Using tee Command
Command:
echo "Line 3" | tee -a notes.txt
Explanation:
Writes to file AND displays output
-a means append
# Without -a, it will overwrite
6. Reading Files
a) Full file
cat notes.txt
Displays entire content
b) First lines
head -n 2 notes.txt
Shows first 2 lines
c) Last lines
tail -n 2 notes.txt
Shows last 2 lines

# Hands-on Practice (Commands + Output)
- Step 1: Create File
touch notes.txt

- Step 2: Write Data
echo "Line 1 - Learning Linux file handling" > notes.txt
echo "Line 2 - Practicing append operation" >> notes.txt

- Step 3: Append Using tee
echo "Line 3 - Using tee command" | tee -a notes.txt
Output:
Line 3 - Using tee command

- Step 4: Add More Lines
echo "Line 4 - File operations are important" >> notes.txt
echo "Line 5 - DevOps uses logs heavily" >> notes.txt

- Step 5: Read Full File
cat notes.txt
Output:
Line 1 - Learning Linux file handling
Line 2 - Practicing append operation
Line 3 - Using tee command
Line 4 - File operations are important
Line 5 - DevOps uses logs heavily

- Step 6: Read First Lines
head -n 2 notes.txt
Output:
Line 1 - Learning Linux file handling
Line 2 - Practicing append operation

- Step 7: Read Last Lines
tail -n 2 notes.txt
Output:
Line 4 - File operations are important
Line 5 - DevOps uses logs heavily

# Key Differences (Important for Interview)
# Command	Purpose
>	Overwrite file
>>	Append to file
tee	Write + display output
cat	Show full file
head	Show top lines
tail	Show bottom lines

# Mini DevOps Use Cases

# 1. Update config file
echo "PORT=3000" >> .env

# 2. Check logs
tail -n 50 app.log

# 3. Monitor logs live
tail -f app.log

# 4. Debug output and save
echo "Debug message" | tee debug.log

# Interview Questions & Answers
# Q1: Difference between > and >>?
> overwrites file
>> appends data

# Q2: What does tee do?
Writes output to file AND displays it on terminal.

# Q3: How to read large files?
Use less, head, or tail.

# Q4: How to view last logs?
tail -n 50 file.log

# Q5: How to create file quickly?
touch filename

# Scenario-Based Questions
# Scenario 1: You accidentally overwrote a file
Cause: Used > instead of >>
Fix:
Restore from backup (if available)
Always use >> for logs/config

# Scenario 2: Logs are too large
Use:
head -n 50 log.txt
tail -n 50 log.txt

# Scenario 3: Need to see logs in real-time
Use:
tail -f log.txt

# Scenario 4: Want to save command output and see it
Use:
echo "Test" | tee file.txt
