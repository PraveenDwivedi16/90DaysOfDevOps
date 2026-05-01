# Day 07 – Linux File System + Scenario Practice (Full Training)

#  PART 1 – FIRST UNDERSTAND THE TOPIC 

Before doing commands, understand this clearly 

# What is Linux File System?
Linux file system is how Linux organizes everything.

# Important concept:
Everything in Linux is a file
Examples:
Logs = files
Config = files
Commands = files
Devices = files

# Structure (Tree Style)
/        = starting point (/root)
/home    = normal user files
/root    = root/admin user files
/etc     = configuration
/var/log = logs
/tmp     = temporary files
/bin     = basic commands
/usr/bin = installed commands
/opt     = optional apps

 / = ROOT (starting point of everything)

#  DevOps Mindset
When issue happens, you think:
Problem	Where to check
Service issue	/etc + systemctl
Logs issue	/var/log
User issue	/home
Permission issue	file location
App installed	/opt

# PART 2 – HANDS-ON TASK (DO THIS ON YOUR SYSTEM)
Run each command yourself 
# 🔹 1. ROOT DIRECTORY /
Command:
ls -l /
What you see:
Folders like:
home, etc, var, tmp
Understanding:
Root is the top-level directory
Use case:
I would use this when:
"Exploring system structure or debugging unknown paths"

# 🔹 2. HOME DIRECTORY /home
ls -l /home
Understanding:
Contains user folders
Example:
/home/ubuntu
Use:
"Checking user files or scripts"

# 🔹 3. ROOT USER /root
ls -l /root
Understanding:
Home of admin user
Use:
"Admin-level scripts or configs"

# 🔹 4. CONFIG DIRECTORY /etc
ls -l /etc
Example files:
hostname
passwd
ssh
Check one file:
cat /etc/hostname
 Use:
"Fixing service/config issues"

# 🔹 5. LOG DIRECTORY /var/log
ls -l /var/log
Find largest logs:
du -sh /var/log/* 2>/dev/null | sort -h | tail -5
Use:
"Debugging errors and failures"

# 🔹 6. TEMP DIRECTORY /tmp
ls -l /tmp
Understanding:
Temporary files
Use:
"Testing or temporary storage"

#🔹 7. BIN DIRECTORY /bin
ls -l /bin
Contains:
Basic commands like ls, cp
 Use:
"Understanding system commands"

# 🔹 8. /usr/bin
ls -l /usr/bin
Contains:
Advanced commands
Use:
"Installed programs"

# 🔹 9. /opt
ls -l /opt
Contains:
Third-party apps
Use:
"Custom application deployment"

# 🔹 10. HOME CHECK
ls -la ~
Shows:
Hidden files (.bashrc)
#  IMPORTANT UNDERSTANDING
If app fails → check /var/log
If config wrong → check /etc
If user issue → check /home

#  PART 3 – SCENARIO TRAINING (MOST IMPORTANT)
Now we think like DevOps 

# Scenario 1 – Service Not Starting
Step 1
systemctl status myapp
Why: Check if service failed or stopped

# Step 2
journalctl -u myapp -n 50
Why: Check logs for error

# Step 3
systemctl is-enabled myapp
Why: Check if auto-start enabled

# Step 4
systemctl restart myapp
Why: Try fixing quickly

# Scenario 2 – High CPU
# Step 1
top
Why: Live CPU usage

# Step 2
ps aux --sort=-%cpu | head -10
Why: Find top process

# Step 3
pgrep <process>
Why: Get PID

# Scenario 3 – Find Logs
# Step 1
systemctl status docker
Why: Confirm service

# Step 2
journalctl -u docker -n 50
Why: Recent logs

# Step 3
journalctl -u docker -f
Why: Live logs

# Scenario 4 – Permission Issue
# Step 1
ls -l /home/user/backup.sh
Why: Check permissions

# Step 2
chmod +x /home/user/backup.sh
Why: Add execute permission

# Step 3
./backup.sh
Why: Run script
