# Day 11 – File Ownership (chown & chgrp) 
# PART 1 – CORE CONCEPT (MUST UNDERSTAND FIRST)

# What is File Ownership?
Every file in Linux has:
Owner (User) + Group
Example:
ls -l file.txt
Output:
-rw-r--r-- 1 tokyo developers 100 Oct 10 file.txt

# Breakdown:
Field	Meaning
tokyo	Owner (who created/owns file)
developers	Group (who shares access)

# Difference Between Owner & Group
Owner	Group
Single user	Multiple users
Full control	Shared access
Can change permissions	Limited access

# DevOps use:
Owner → application user
Group → team access

# Why Ownership Matters
Real-world:
Problem	Reason
App can't write logs	wrong owner
User can't access file	wrong group
Deployment fails	permission mismatch

# PART 2 – HANDS-ON TASKS
# 🔹 TASK 1 – Understand Ownership
Step 1
ls -l
Example:
-rw-r--r-- 1 ubuntu ubuntu devops.txt

Owner = ubuntu
Group = ubuntu
Answer:
Owner = who controls file
Group = who shares access

# 🔹 TASK 2 – chown (Change Owner)
# Step 1: Create file
touch devops-file.txt
ls -l devops-file.txt

# Step 2: Change owner to tokyo
sudo chown tokyo devops-file.txt

# Step 3: Change owner to berlin
sudo chown berlin devops-file.txt
Verify:
ls -l devops-file.txt

# 🔹 TASK 3 – chgrp (Change Group)
# Step 1: Create file
touch team-notes.txt
ls -l team-notes.txt

# Step 2: Create group
sudo groupadd heist-team

# Step 3: Change group
sudo chgrp heist-team team-notes.txt
Verify:
ls -l team-notes.txt

# 🔹 TASK 4 – Change Owner + Group Together
# Step 1: Create file
touch project-config.yaml
# Step 2: Change both
sudo chown professor:heist-team project-config.yaml

# Step 3: Directory
mkdir app-logs
sudo chown berlin:heist-team app-logs

# 🔹 TASK 5 – Recursive Ownership
# Step 1: Create structure
mkdir -p heist-project/vault
mkdir -p heist-project/plans
touch heist-project/vault/gold.txt
touch heist-project/plans/strategy.conf

# Step 2: Create group
sudo groupadd planners

# Step 3: Apply recursive ownership
sudo chown -R professor:planners heist-project/

# Step 4: Verify
ls -lR heist-project/

# 🔹 TASK 6 – Practice Challenge
Step 1: Create users
sudo useradd -m tokyo
sudo useradd -m berlin
sudo useradd -m nairobi

# Step 2: Create groups
sudo groupadd vault-team
sudo groupadd tech-team

# Step 3: Create directory
mkdir bank-heist

# Step 4: Create files
touch bank-heist/access-codes.txt
touch bank-heist/blueprints.pdf
touch bank-heist/escape-plan.txt

# Step 5: Set ownership
sudo chown tokyo:vault-team bank-heist/access-codes.txt
sudo chown berlin:tech-team bank-heist/blueprints.pdf
sudo chown nairobi:vault-team bank-heist/escape-plan.txt

# Step 6: Verify
ls -l bank-heist/

# PART 3 – REAL PRODUCTION SCENARIOS
# Scenario 1 – App cannot write logs
Problem:
Wrong owner
Fix:
sudo chown appuser:appgroup /var/log/app.log
# Scenario 2 – User cannot access file
Fix:
chgrp developers file
chmod 775 file

# Scenario 3 – Docker volume permission error
Fix:
chown -R 1000:1000 /data

# Scenario 4 – Deployment failure
Fix:
chown -R www-data:www-data /var/www/html

# PART 4 – INTERVIEW QUESTIONS
# Q1: Difference between chown and chgrp?
Command	Purpose
chown	change owner
chgrp	change group
# Q2: How to change both owner and group?
chown user:group file
# Q3: What is recursive ownership?
chown -R user:group folder
# Q4: Why ownership is important?
Controls file access for applications and users

Q5: What happens if user doesn't exist?

👉 Command fails
