# Day 13 – Linux Volume Management (LVM)
#  PART 1 – CORE CONCEPT 

# What Problem Does LVM Solve.?
Without LVM 
Fixed disk size
Cannot easily extend storage
Downtime required
With LVM 
Disk → Physical Volume → Volume Group → Logical Volume → Filesystem

# You can:
Extend disk size without downtime
Combine multiple disks
Resize storage dynamically

# LVM COMPONENTS 

# 🔹 1. Physical Volume (PV)
Real disk or virtual disk
Example:
/dev/sdb
/dev/loop0

# 🔹 2. Volume Group (VG)
Pool of storage created from PV
PV → combined → VG
Example:
devops-vg

# 🔹 3. Logical Volume (LV)
Virtual partition created from VG
Example:
app-data

# Simple Analogy
Hard Disk = Land
Volume Group = City
Logical Volume = House

# PART 2 – HANDS-ON TASK (STEP-BY-STEP)

# STEP 0 – Setup Virtual Disk (if no spare disk)
dd if=/dev/zero of=/tmp/disk1.img bs=1M count=1024
losetup -fP /tmp/disk1.img
losetup -a

# Output example:
/dev/loop0

# 🔹 TASK 1 – Check Current Storage
lsblk
pvs
vgs
lvs
df -h

# What you observe:
Existing disks
No LVM yet

# 🔹 TASK 2 – Create Physical Volume
pvcreate /dev/loop0
pvs

# What happens:
Disk becomes LVM-ready

# TASK 3 – Create Volume Group
vgcreate devops-vg /dev/loop0
vgs

# What happens:
# Storage pool created

# TASK 4 – Create Logical Volume
lvcreate -L 500M -n app-data devops-vg
lvs

# What happens:
Virtual partition created

# 🔹 TASK 5 – Format and Mount
Step 1: Format
mkfs.ext4 /dev/devops-vg/app-data
Step 2: Mount
mkdir -p /mnt/app-data
mount /dev/devops-vg/app-data /mnt/app-data
Step 3: Verify
df -h /mnt/app-data

# What happens:
Storage becomes usable

# 🔹 TASK 6 – Extend Volume
Step 1: Extend LV
lvextend -L +200M /dev/devops-vg/app-data
Step 2: Resize filesystem
resize2fs /dev/devops-vg/app-data
Step 3: Verify
df -h /mnt/app-data

#  What happens:

# Disk size increased without data loss

# PART 3 – REAL PRODUCTION SCENARIOS

# Scenario 1 – Disk Full (Common Issue)
Problem:
No space left on device
Fix using LVM:
lvextend -L +5G /dev/devops-vg/app-data
resize2fs /dev/devops-vg/app-data

# Scenario 2 – Multiple Disks
# Combine:

vgextend devops-vg /dev/sdc

# Scenario 3 – Application Crash due to storage

#  Fix:
Extend volume
Clear logs

#  Scenario 4 – Wrong mount
# Fix:
mount | grep app-data

#  PART 4 – INTERVIEW QUESTIONS (IMPORTANT)

Q1: What is LVM?

Logical Volume Manager
Provides flexible disk management

Q2: Difference between PV, VG, LV?
Component	Meaning
PV	physical disk
VG	storage pool
LV	usable partition

Q3: Can we extend disk without downtime?
 Yes, using LVM

Q4: What is lvextend?
Increase logical volume size

Q5: Why resize2fs?
To resize filesystem after LV extend

Q6: What happens if you skip resize2fs?
Disk increases but OS cannot use space
