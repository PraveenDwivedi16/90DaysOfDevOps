# Day 09 – Linux User & Group Management 
# PART 1 – CONCEPT (UNDERSTAND FIRST)

Before commands, understand how Linux users & permissions work.

# 1. What is a User?
A user = someone who can log into Linux.
Types:
Normal user → works in /home
Root user → full access (/root)

# 2. What is a Group?
A group = collection of users
Why needed?
To share access to files
To manage permissions easily

# 3. File Permission Structure
-rwxrwxr-x
Breakdown:
Part	Meaning
rwx	Owner
rwx	Group
r-x	Others

# 4. Important Files
File	Purpose
/etc/passwd	All users
/etc/group	All groups

# 5. DevOps Use Case
Real example:
Developers need access → group: developers
Admin team → group: admins
Shared folder → controlled by group
 
 # PART 2 – HANDS-ON TASKS 
# 🔹TASK 1 – Create Users
Step 1: Create users
sudo useradd -m tokyo
sudo useradd -m berlin
sudo useradd -m professor

# ( -m → creates home directory )

# Step 2: Set passwords
sudo passwd tokyo
sudo passwd berlin
sudo passwd professor

# Step 3: Verify users
cat /etc/passwd | grep tokyo
ls -l /home

# 🔹 TASK 2 – Create Groups
# Step 1: Create groups
sudo groupadd developers
sudo groupadd admins

# Step 2: Verify groups
cat /etc/group | grep developers
cat /etc/group | grep admins

# 🔹 TASK 3 – Assign Users to Groups
Assign users:
sudo usermod -aG developers tokyo
sudo usermod -aG developers berlin
sudo usermod -aG admins berlin
sudo usermod -aG admins professor

# ( -aG = append group )
Verify:
groups tokyo
groups berlin
groups professor

# 🔹 TASK 4 – Shared Directory
# Step 1: Create directory
sudo mkdir -p /opt/dev-project
# Step 2: Set group
sudo chgrp developers /opt/dev-project
# Step 3: Set permission
sudo chmod 775 /opt/dev-project
# Step 4: Verify
ls -ld /opt/dev-project
Expected:
drwxrwxr-x
# Step 5: Test access
sudo -u tokyo touch /opt/dev-project/file1.txt
sudo -u berlin touch /opt/dev-project/file2.txt
# Step 6: Verify
ls -l /opt/dev-project

# 🔹 TASK 5 – Team Workspace
# Step 1: Create user
sudo useradd -m nairobi
sudo passwd nairobi
# Step 2: Create group
sudo groupadd project-team
# Step 3: Add users
sudo usermod -aG project-team nairobi
sudo usermod -aG project-team tokyo
# Step 4: Create directory
sudo mkdir -p /opt/team-workspace
# Step 5: Set group + permission
sudo chgrp project-team /opt/team-workspace
sudo chmod 775 /opt/team-workspace
# Step 6: Test
sudo -u nairobi touch /opt/team-workspace/test.txt
# Step 7: Verify
ls -l /opt/team-workspace

# PART 3 – REAL-WORLD SCENARIOS
Scenario 1 – Permission Denied
Problem:
User can't access folder
Fix:
groups username
ls -ld /folder
Check:
User in group?
Permission correct?

# Scenario 2 – User Can't Create File
Fix:
chmod 775 folder
chgrp group folder

# Scenario 3 – Group Not Working
Fix:
usermod -aG group user
Then:
su - user

# Scenario 4 – Changes Not Applied
Reason:
User needs re-login

# PART 4 – INTERVIEW QUESTIONS

# Q1: Difference between useradd and adduser?
useradd → low-level
adduser → user-friendly

# Q2: What is -aG?
Adds user to group without removing existing groups

# Q3: What is 775?

 Owner = rwx
 Group = rwx
 Others = r-x

# Q4: How to check user groups?
groups username

# Q5: Why groups are used?
To manage permissions for multiple users easily
