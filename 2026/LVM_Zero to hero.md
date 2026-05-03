# Quick Command Cheat Sheet

# Check disks
lsblk

# Install LVM If not install 
sudo apt update
sudo apt install lvm2 -y

# Create Physical Volume
sudo pvcreate /dev/xvdf

# Check PV
sudo pvs

# Create Volume Group
sudo vgcreate vg_test /dev/xvdf

# Check VG
sudo vgs

# Create Logical Volumes
sudo lvcreate -L 2G -n lv_demo vg_test
sudo lvcreate -L 2G -n lv_demo-1 vg_test
sudo lvcreate -L 3G -n lv_demo-2 vg_test

# Check LV
sudo lvs

# Format LVs
sudo mkfs.ext4 /dev/vg_test/lv_demo
sudo mkfs.ext4 /dev/vg_test/lv_demo-1
sudo mkfs.ext4 /dev/vg_test/lv_demo-2

# Create mount points
sudo mkdir -p /mnt/lvm
sudo mkdir -p /mnt/lvm-1
sudo mkdir -p /mnt/lvm-2

# Mount LVs
sudo mount /dev/vg_test/lv_demo /mnt/lvm
sudo mount /dev/vg_test/lv_demo-1 /mnt/lvm-1
sudo mount /dev/vg_test/lv_demo-2 /mnt/lvm-2

# Verify
lsblk
df -h
sudo pvs
sudo vgs
sudo lvs

# Create test files
echo "LV 1 working" | sudo tee /mnt/lvm/test.txt
echo "LV 2 working" | sudo tee /mnt/lvm-1/test.txt
echo "LV 3 working" | sudo tee /mnt/lvm-2/test.txt

# Read test files
cat /mnt/lvm/test.txt
cat /mnt/lvm-1/test.txt
cat /mnt/lvm-2/test.txt

# Extend LV by 1 GB
sudo lvextend -r -L +1G /dev/vg_test/lv_demo

# Verify after extend
df -h /mnt/lvm
sudo lvs

# LVM Practice Revision Documentation
# Ubuntu EC2 + AWS EBS Volume

1. My Practice Setup

In this practice, I used an Ubuntu EC2 instance in AWS and attached one extra EBS volume for LVM practice.

My extra disk was:

/dev/xvdf

Disk size:

10 GB

I created one Volume Group:

vg_test

Inside this Volume Group, I created three Logical Volumes:

lv_demo      = 2 GB
lv_demo-1    = 2 GB
lv_demo-2    = 3 GB

I mounted them on these folders:

/mnt/lvm
/mnt/lvm-1
/mnt/lvm-2
2. My lsblk Output
xvdf                 202:80   0   10G  0 disk
├─vg_test-lv_demo    252:0    0    2G  0 lvm  /mnt/lvm
│                                             /mnt/lvm
├─vg_test-lv_demo--1 252:1    0    2G  0 lvm  /mnt/lvm-1
│                                             /mnt/lvm
└─vg_test-lv_demo--2 252:2    0    3G  0 lvm  /mnt/lvm-2
                                              /mnt/lvm
3. What This Output Means

This output shows that I successfully created LVM storage.

The main disk is:

xvdf

This is my extra AWS EBS volume.

It is 10 GB in size.

From this 10 GB disk, I created LVM Logical Volumes:

vg_test-lv_demo
vg_test-lv_demo--1
vg_test-lv_demo--2

These are mounted at:

/mnt/lvm
/mnt/lvm-1
/mnt/lvm-2

The structure is:

AWS EBS Volume
     ↓
Linux Disk: /dev/xvdf
     ↓
Physical Volume
     ↓
Volume Group: vg_test
     ↓
Logical Volumes:
     - lv_demo
     - lv_demo-1
     - lv_demo-2
     ↓
Mount Points:
     - /mnt/lvm
     - /mnt/lvm-1
     - /mnt/lvm-2
4. Simple Explanation of LVM

LVM means:

Logical Volume Manager

It is used to manage Linux storage in a flexible way.

Without LVM, storage usually works like this:

Disk → Partition → Filesystem → Mount Point

With LVM, storage works like this:

Disk → Physical Volume → Volume Group → Logical Volume → Filesystem → Mount Point

LVM gives more flexibility than normal disk partitioning.

For example, if a normal partition becomes full, increasing it can be difficult.

But with LVM, we can easily increase the size of a Logical Volume if free space is available in the Volume Group.

5. Why LVM Is Useful in DevOps

LVM is useful in real DevOps work because servers often need flexible storage.

Common real-world examples:

Application logs are increasing
Database storage is becoming full
Docker storage needs more space
Backup storage needs separate space
New cloud disk is added to a server
A filesystem needs to be extended without rebuilding the server

In AWS EC2, we can attach a new EBS volume and add it to LVM.

This makes storage management easier.

Example:

Attach new EBS disk
Create PV
Extend VG
Extend LV
Resize filesystem
6. Important LVM Terms
6.1 Disk

A disk is physical or virtual storage.

In AWS EC2, the disk is usually an EBS volume.

Example from my practice:

/dev/xvdf

This was my extra 10 GB disk.

Important:

Do not use the OS disk for practice.

OS disk is usually:

/dev/xvda

or:

/dev/nvme0n1
6.2 Physical Volume

Physical Volume is also called:

PV

A Physical Volume is a disk that is prepared for LVM.

Command used:

sudo pvcreate /dev/xvdf

Meaning:

Prepare /dev/xvdf for LVM use

After this command, /dev/xvdf becomes part of LVM.

6.3 Volume Group

Volume Group is also called:

VG

A Volume Group is a storage pool.

In my practice, I created:

vg_test

Command:

sudo vgcreate vg_test /dev/xvdf

Meaning:

Create a storage pool named vg_test using /dev/xvdf
6.4 Logical Volume

Logical Volume is also called:

LV

A Logical Volume is like a flexible partition created from a Volume Group.

In my practice, I created:

lv_demo
lv_demo-1
lv_demo-2

Example command:

sudo lvcreate -L 2G -n lv_demo vg_test

Meaning:

Create a 2 GB logical volume named lv_demo from vg_test
6.5 Filesystem

A filesystem allows Linux to store files and folders.

Common filesystems:

ext4
xfs

In beginner practice, ext4 is commonly used.

Example command:

sudo mkfs.ext4 /dev/vg_test/lv_demo

Meaning:

Format lv_demo with ext4 filesystem

Warning:

Formatting removes data from the selected volume.

6.6 Mount Point

A mount point is a folder where we attach storage.

Example:

/mnt/lvm

After mounting, we can store files inside that folder.

Example command:

sudo mount /dev/vg_test/lv_demo /mnt/lvm

Meaning:

Attach lv_demo storage to /mnt/lvm folder
7. My Complete LVM Flow

The full flow I practiced was:

/dev/xvdf
   ↓
pvcreate
   ↓
Physical Volume
   ↓
vgcreate vg_test
   ↓
Volume Group
   ↓
lvcreate
   ↓
Logical Volumes
   ↓
mkfs.ext4
   ↓
Filesystem
   ↓
mount
   ↓
Use folders:
   /mnt/lvm
   /mnt/lvm-1
   /mnt/lvm-2
8. Step-by-Step Practice Revision
Step 1: Check Available Disks
What I did

I checked available disks on Ubuntu.

Why I did it

Before using LVM, I must know which disk is my extra practice disk.

Command
lsblk
Expected output
xvda    8G   disk
xvdf   10G   disk

or:

nvme0n1   OS disk
nvme1n1   Extra disk
Important understanding

The disk mounted on / is the OS disk.

Do not use the OS disk.

Use only the extra EBS disk.

In my practice, the extra disk was:

/dev/xvdf
Common mistake

Using the wrong disk.

Dangerous example:

sudo pvcreate /dev/xvda

This can damage the OS disk.

Correct for my practice:

sudo pvcreate /dev/xvdf
Step 2: Install LVM Tools
What I did

I installed the LVM package.

Why I did it

Without the LVM package, commands like pvcreate, vgcreate, and lvcreate may not work.

Command
sudo apt update
sudo apt install lvm2 -y
Verify
lvm version
Expected result

LVM version information should be displayed.

Common mistake

Running LVM commands before installing lvm2.

If command is not found, install:

sudo apt install lvm2 -y
Step 3: Create Physical Volume
What I did

I converted /dev/xvdf into a Physical Volume.

Why I did it

LVM cannot directly use the disk until it is initialized as a Physical Volume.

Command
sudo pvcreate /dev/xvdf
Expected output
Physical volume "/dev/xvdf" successfully created.
Verify
sudo pvs
Expected output
PV         VG       Fmt   Attr   PSize   PFree
/dev/xvdf           lvm2         10.00g  10.00g
Explanation

At this point:

/dev/xvdf

is prepared for LVM.

But it is not yet a storage pool.

For storage pool, we need a Volume Group.

Common mistakes

Mistake 1:

Using the OS disk.

Mistake 2:

Running pvcreate on a disk that already has important data.

Mistake 3:

Not checking lsblk before running the command.

Step 4: Create Volume Group
What I did

I created a Volume Group named:

vg_test
Why I did it

A Volume Group is a storage pool.

Logical Volumes are created from this pool.

Command
sudo vgcreate vg_test /dev/xvdf
Expected output
Volume group "vg_test" successfully created
Verify
sudo vgs
Expected output
VG       #PV  #LV  Attr    VSize    VFree
vg_test   1    0   wz--n-  10.00g   10.00g
Explanation

Now the structure is:

/dev/xvdf
   ↓
Physical Volume
   ↓
vg_test

vg_test is now my storage pool.

Common mistakes

Mistake 1:

Wrong VG name.

Mistake 2:

Trying to create the same VG again.

Mistake 3:

Running vgcreate before pvcreate.

Step 5: Create First Logical Volume
What I did

I created my first Logical Volume:

lv_demo

Size:

2 GB
Why I did it

Logical Volume works like a flexible partition.

We can format it and mount it.

Command
sudo lvcreate -L 2G -n lv_demo vg_test
Expected output
Logical volume "lv_demo" created.
Verify
sudo lvs
Expected output
LV       VG       LSize
lv_demo  vg_test  2.00g
Explanation

Now the structure is:

/dev/xvdf
   ↓
PV
   ↓
vg_test
   ↓
lv_demo
Common mistakes

Mistake 1:

Creating LV bigger than available VG space.

Mistake 2:

Wrong VG name.

Mistake 3:

Forgetting -n for name.

Step 6: Format First Logical Volume
What I did

I formatted the Logical Volume with ext4.

Why I did it

Linux cannot store files on a raw Logical Volume.

It needs a filesystem.

Command
sudo mkfs.ext4 /dev/vg_test/lv_demo
Expected output
Creating filesystem...
Writing superblocks and filesystem accounting information: done
Verify
sudo blkid /dev/vg_test/lv_demo
Expected output
/dev/vg_test/lv_demo: UUID="..." TYPE="ext4"
Important warning

mkfs.ext4 formats the volume.

Formatting removes existing data.

Use it only on a new empty LV.

Common mistakes

Mistake 1:

Formatting the wrong device.

Mistake 2:

Formatting /dev/xvdf directly after creating LV.

Mistake 3:

Forgetting to format before mounting.

Step 7: Create Mount Folder for First LV
What I did

I created a folder:

/mnt/lvm
Why I did it

A mount point is required to access the storage.

Command
sudo mkdir -p /mnt/lvm
Verify
ls -ld /mnt/lvm
Expected output
drwxr-xr-x root root /mnt/lvm
Common mistakes

Mistake 1:

Mounting to a folder that does not exist.

Mistake 2:

Using confusing folder names.

Mistake 3:

Mounting multiple LVs on the same folder by mistake.

Step 8: Mount First Logical Volume
What I did

I mounted lv_demo on /mnt/lvm.

Why I did it

After mounting, /mnt/lvm becomes usable storage.

Command
sudo mount /dev/vg_test/lv_demo /mnt/lvm
Verify
df -h /mnt/lvm
Expected output
Filesystem                    Size  Used  Avail  Use%  Mounted on
/dev/mapper/vg_test-lv_demo   2.0G   ...  ...    ...   /mnt/lvm
Common mistakes

Mistake 1:

Mounting before formatting.

Mistake 2:

Using wrong LV path.

Mistake 3:

Mounting different LVs on same folder.

Step 9: Create Second Logical Volume
What I did

I created another LV:

lv_demo-1

Size:

2 GB
Why I did it

This shows that one Volume Group can contain multiple Logical Volumes.

Command
sudo lvcreate -L 2G -n lv_demo-1 vg_test
Expected output
Logical volume "lv_demo-1" created.
Verify
sudo lvs
Expected output
LV         VG       LSize
lv_demo    vg_test  2.00g
lv_demo-1  vg_test  2.00g
Explanation

Now vg_test has two LVs:

vg_test
   ├── lv_demo
   └── lv_demo-1
Step 10: Format Second Logical Volume
Command
sudo mkfs.ext4 /dev/vg_test/lv_demo-1
Verify
sudo blkid /dev/vg_test/lv_demo-1
Expected output
TYPE="ext4"
Step 11: Create Mount Folder for Second LV
Command
sudo mkdir -p /mnt/lvm-1
Verify
ls -ld /mnt/lvm-1
Step 12: Mount Second Logical Volume
Command
sudo mount /dev/vg_test/lv_demo-1 /mnt/lvm-1
Verify
df -h /mnt/lvm-1
Expected output
/dev/mapper/vg_test-lv_demo--1   2.0G   /mnt/lvm-1
Why name shows lv_demo--1

In LVM device mapper names, a hyphen - inside LV name is shown as double hyphen --.

So:

lv_demo-1

appears as:

vg_test-lv_demo--1

This is normal.

It is not an error.

Step 13: Create Third Logical Volume
What I did

I created third LV:

lv_demo-2

Size:

3 GB
Command
sudo lvcreate -L 3G -n lv_demo-2 vg_test
Verify
sudo lvs
Expected output
LV         VG       LSize
lv_demo    vg_test  2.00g
lv_demo-1  vg_test  2.00g
lv_demo-2  vg_test  3.00g
Explanation

Now the 10 GB disk has:

2 GB + 2 GB + 3 GB = 7 GB used

Remaining space in VG should be around:

3 GB free

Check using:

sudo vgs
Step 14: Format Third Logical Volume
Command
sudo mkfs.ext4 /dev/vg_test/lv_demo-2
Verify
sudo blkid /dev/vg_test/lv_demo-2
Step 15: Create Mount Folder for Third LV
Command
sudo mkdir -p /mnt/lvm-2
Verify
ls -ld /mnt/lvm-2
Step 16: Mount Third Logical Volume
Command
sudo mount /dev/vg_test/lv_demo-2 /mnt/lvm-2
Verify
df -h /mnt/lvm-2
Expected output
/dev/mapper/vg_test-lv_demo--2   3.0G   /mnt/lvm-2
9. Final Verification Commands

After creating all three LVs, I should verify using these commands.

9.1 Check Disk Structure
lsblk

Expected structure:

xvdf
├─vg_test-lv_demo       2G   /mnt/lvm
├─vg_test-lv_demo--1    2G   /mnt/lvm-1
└─vg_test-lv_demo--2    3G   /mnt/lvm-2
9.2 Check Mounted Filesystems
df -h

or:

df -h /mnt/lvm
df -h /mnt/lvm-1
df -h /mnt/lvm-2

Expected:

/mnt/lvm     mounted
/mnt/lvm-1   mounted
/mnt/lvm-2   mounted
9.3 Check Physical Volume
sudo pvs

Expected:

/dev/xvdf belongs to vg_test
9.4 Check Volume Group
sudo vgs

Expected:

VG       VSize    VFree
vg_test  10.00g   around 3.00g
9.5 Check Logical Volumes
sudo lvs

Expected:

lv_demo    2G
lv_demo-1  2G
lv_demo-2  3G
10. Why /mnt/lvm Appears Multiple Times in Output

Your output shows something like:

/mnt/lvm
/mnt/lvm-1
/mnt/lvm-2
/mnt/lvm

This can happen when:

1. A device was mounted more than once
2. Same mount point was used accidentally
3. Previous mounts were not unmounted properly
4. lsblk shows multiple mount references

To check exact mounts, use:

findmnt

or:

findmnt | grep lvm

Also check:

df -h | grep lvm

Expected clean result should be:

lv_demo      mounted on /mnt/lvm
lv_demo-1    mounted on /mnt/lvm-1
lv_demo-2    mounted on /mnt/lvm-2

If one LV is mounted on the wrong folder, unmount and mount correctly.

Example:

sudo umount /mnt/lvm
sudo umount /mnt/lvm-1
sudo umount /mnt/lvm-2

Then mount again:

sudo mount /dev/vg_test/lv_demo /mnt/lvm
sudo mount /dev/vg_test/lv_demo-1 /mnt/lvm-1
sudo mount /dev/vg_test/lv_demo-2 /mnt/lvm-2

Verify:

df -h | grep lvm
11. Create Test Files in Each Mount

This is important for practice.

It proves that all three LVs are working.

Test LV 1
echo "This is lv_demo mounted on /mnt/lvm" | sudo tee /mnt/lvm/test.txt
cat /mnt/lvm/test.txt

Expected output:

This is lv_demo mounted on /mnt/lvm
Test LV 2
echo "This is lv_demo-1 mounted on /mnt/lvm-1" | sudo tee /mnt/lvm-1/test.txt
cat /mnt/lvm-1/test.txt

Expected output:

This is lv_demo-1 mounted on /mnt/lvm-1
Test LV 3
echo "This is lv_demo-2 mounted on /mnt/lvm-2" | sudo tee /mnt/lvm-2/test.txt
cat /mnt/lvm-2/test.txt

Expected output:

This is lv_demo-2 mounted on /mnt/lvm-2
12. Important Command Revision
Check disks
lsblk

Used to check disk, partition, LVM, and mount structure.

Check disk usage
df -h

Used to check mounted filesystem usage.

Check Physical Volumes
sudo pvs

Used to check PV information.

Check Volume Groups
sudo vgs

Used to check VG size and free space.

Check Logical Volumes
sudo lvs

Used to check LV names and sizes.

Create Physical Volume
sudo pvcreate /dev/xvdf

Used to prepare disk for LVM.

Dangerous if wrong disk is selected.

Create Volume Group
sudo vgcreate vg_test /dev/xvdf

Used to create a storage pool.

Create Logical Volume
sudo lvcreate -L 2G -n lv_demo vg_test

Used to create a flexible partition from VG.

Format Logical Volume
sudo mkfs.ext4 /dev/vg_test/lv_demo

Used to create filesystem.

Dangerous because it erases existing data.

Create Mount Directory
sudo mkdir -p /mnt/lvm

Used to create mount point folder.

Mount Logical Volume
sudo mount /dev/vg_test/lv_demo /mnt/lvm

Used to attach LV to folder.

13. Important Understanding from My Practice

I created 3 Logical Volumes from 1 EBS disk.

This means one physical disk can be divided into multiple flexible logical volumes.

My disk:

/dev/xvdf = 10 GB

My Volume Group:

vg_test = storage pool

My Logical Volumes:

lv_demo    = 2 GB
lv_demo-1  = 2 GB
lv_demo-2  = 3 GB

Total used:

7 GB

Approximate remaining free space:

3 GB

This remaining space can be used later to:

Create another LV
Extend existing LV
Practice resizing
14. Next Practice: Extend Existing Logical Volume

Now that I have free space in vg_test, I can increase one LV.

Example:

Increase lv_demo by 1 GB.

Step 1: Check current size
df -h /mnt/lvm
sudo lvs
sudo vgs
Step 2: Extend LV and filesystem together
sudo lvextend -r -L +1G /dev/vg_test/lv_demo

Explanation:

lvextend = increase logical volume size
-r       = resize filesystem automatically
-L +1G   = add 1 GB
Step 3: Verify
df -h /mnt/lvm
sudo lvs

Expected:

lv_demo size should increase from 2 GB to 3 GB
15. Next Practice: Permanent Mount Using /etc/fstab

Right now, mounts are temporary.

After reboot, they may disappear.

To mount automatically after reboot, we use:

/etc/fstab
Step 1: Get UUIDs
sudo blkid /dev/vg_test/lv_demo
sudo blkid /dev/vg_test/lv_demo-1
sudo blkid /dev/vg_test/lv_demo-2

Example output:

UUID="abc-111" TYPE="ext4"
UUID="abc-222" TYPE="ext4"
UUID="abc-333" TYPE="ext4"
Step 2: Backup fstab
sudo cp /etc/fstab /etc/fstab.backup

Important:

Always backup before editing /etc/fstab.

Wrong fstab can create boot problems.

Step 3: Edit fstab
sudo nano /etc/fstab

Add lines like this:

UUID=abc-111 /mnt/lvm ext4 defaults,nofail 0 2
UUID=abc-222 /mnt/lvm-1 ext4 defaults,nofail 0 2
UUID=abc-333 /mnt/lvm-2 ext4 defaults,nofail 0 2

Replace UUID values with your real UUIDs.

Step 4: Test fstab
sudo mount -a

Expected output:

No output

No output usually means success.

Step 5: Verify
df -h | grep lvm
Important fstab Warning

Do not reboot before testing:

sudo mount -a

If /etc/fstab has wrong entries, server may boot with errors.

Use nofail for AWS practice disks so server can still boot if disk is missing.

16. Common Mistakes and Fixes
Mistake 1: Using OS Disk

Wrong:

sudo pvcreate /dev/xvda

Why wrong:

/dev/xvda is usually the OS disk.

Fix:

Always check:

lsblk

Use extra disk:

/dev/xvdf
Mistake 2: Formatting Wrong Device

Wrong:

sudo mkfs.ext4 /dev/xvdf

If you already created logical volumes, format the LV, not the full disk.

Correct:

sudo mkfs.ext4 /dev/vg_test/lv_demo
Mistake 3: Mounting Multiple LVs on Same Folder

Wrong:

sudo mount /dev/vg_test/lv_demo /mnt/lvm
sudo mount /dev/vg_test/lv_demo-1 /mnt/lvm
sudo mount /dev/vg_test/lv_demo-2 /mnt/lvm

This is confusing.

Correct:

sudo mount /dev/vg_test/lv_demo /mnt/lvm
sudo mount /dev/vg_test/lv_demo-1 /mnt/lvm-1
sudo mount /dev/vg_test/lv_demo-2 /mnt/lvm-2
Mistake 4: Forgetting Filesystem

If you try to mount without formatting, mount may fail.

Fix:

sudo mkfs.ext4 /dev/vg_test/lv_demo

Then mount.

Mistake 5: Not Checking Free Space

Before creating new LV, check:

sudo vgs

If VFree is low, LV creation may fail.

Mistake 6: Confusing LV Name with Mapper Name

LV name:

lv_demo-1

May appear in lsblk as:

vg_test-lv_demo--1

This is normal.

Hyphen becomes double hyphen in device mapper output.

17. Troubleshooting Commands
Check exact mounts
findmnt

or:

findmnt | grep lvm
Check disk usage
df -h
Check LVM status
sudo pvs
sudo vgs
sudo lvs
Check filesystem UUID
sudo blkid
Check if mount folder exists
ls -ld /mnt/lvm
ls -ld /mnt/lvm-1
ls -ld /mnt/lvm-2
Unmount if mounted wrongly
sudo umount /mnt/lvm
sudo umount /mnt/lvm-1
sudo umount /mnt/lvm-2

If target is busy:

sudo lsof +f -- /mnt/lvm

or:

sudo fuser -vm /mnt/lvm
18. Safe Cleanup Commands

Use these only if you want to delete your practice.

Danger:

These commands remove data.

Step 1: Unmount
sudo umount /mnt/lvm
sudo umount /mnt/lvm-1
sudo umount /mnt/lvm-2
Step 2: Remove fstab entries
sudo nano /etc/fstab

Remove or comment LVM mount lines.

Example:

# UUID=abc-111 /mnt/lvm ext4 defaults,nofail 0 2
Step 3: Remove Logical Volumes
sudo lvremove /dev/vg_test/lv_demo
sudo lvremove /dev/vg_test/lv_demo-1
sudo lvremove /dev/vg_test/lv_demo-2

It will ask confirmation.

Type:

y
Step 4: Remove Volume Group
sudo vgremove vg_test
Step 5: Remove Physical Volume
sudo pvremove /dev/xvdf
Step 6: Remove directories
sudo rmdir /mnt/lvm
sudo rmdir /mnt/lvm-1
sudo rmdir /mnt/lvm-2
19. Senior-Level Best Practices
1. Always use lsblk before storage commands

Before dangerous commands, run:

lsblk

Dangerous commands include:

pvcreate
mkfs.ext4
lvremove
vgremove
pvremove
2. Use clear names

Good names:

vg_test
vg_data
vg_logs
vg_database
lv_demo
lv_app
lv_logs
lv_mysql

Avoid confusing names:

test1
abc
lv1
new
3. Keep separate mount points

Good:

/mnt/lvm
/mnt/lvm-1
/mnt/lvm-2

Better real-world naming:

/data/app
/data/logs
/data/backup
4. Use UUID in /etc/fstab

Device names can change.

UUID is safer.

Use:

sudo blkid

Then add UUID in fstab.

5. Always test fstab before reboot
sudo mount -a

Never reboot before testing this.

6. Use nofail for AWS practice disks

In AWS, if the EBS disk is detached and fstab requires it, boot can have issues.

Use:

defaults,nofail

Example:

UUID=abc-111 /mnt/lvm ext4 defaults,nofail 0 2
7. Document storage changes

For real production, document:

Server name
Disk name
Disk size
VG name
LV name
Mount point
Filesystem
Date
Reason
Rollback plan
20. Interview Questions and Answers
Q1. What is LVM?

LVM means Logical Volume Manager.

It is used in Linux to manage storage flexibly.

It allows us to create Physical Volumes, Volume Groups, and Logical Volumes.

Q2. What is PV?

PV means Physical Volume.

It is a disk or partition prepared for LVM.

Example:

sudo pvcreate /dev/xvdf
Q3. What is VG?

VG means Volume Group.

It is a storage pool created from one or more Physical Volumes.

Example:

sudo vgcreate vg_test /dev/xvdf
Q4. What is LV?

LV means Logical Volume.

It is a flexible partition created from a Volume Group.

Example:

sudo lvcreate -L 2G -n lv_demo vg_test
Q5. What is the LVM flow?
Disk → PV → VG → LV → Filesystem → Mount Point
Q6. How do you check LVM details?
sudo pvs
sudo vgs
sudo lvs
Q7. How do you check mounted storage?
df -h
Q8. How do you check disk structure?
lsblk
Q9. How do you create filesystem on LV?
sudo mkfs.ext4 /dev/vg_test/lv_demo
Q10. How do you mount LV?
sudo mount /dev/vg_test/lv_demo /mnt/lvm
Q11. How do you extend LV by 1 GB?
sudo lvextend -r -L +1G /dev/vg_test/lv_demo
Q12. What is the difference between -L 3G and -L +3G?

-L 3G means set total LV size to 3 GB.

-L +3G means add 3 GB to current LV size.

Q13. Why does lv_demo-1 show as lv_demo--1?

Because LVM device mapper escapes hyphens.

One hyphen in LV name appears as double hyphen in mapper output.

So:

lv_demo-1

appears as:

lv_demo--1

This is normal.

Q14. What is /etc/fstab?

/etc/fstab is a Linux file used to mount filesystems automatically after reboot.

Q15. Why use UUID in fstab?

UUID is stable.

Device names can change after reboot or disk attach/detach.

21. Scenario-Based Interview Questions
Scenario 1: /mnt/lvm is full. What will you do?

I will first check current usage:

df -h /mnt/lvm

Then check VG free space:

sudo vgs

If free space is available, extend LV:

sudo lvextend -r -L +1G /dev/vg_test/lv_demo

Then verify:

df -h /mnt/lvm
Scenario 2: VG has no free space. What will you do?

I will attach a new EBS volume to EC2.

Then check disk:

lsblk

Create PV:

sudo pvcreate /dev/xvdg

Extend VG:

sudo vgextend vg_test /dev/xvdg

Then extend LV:

sudo lvextend -r -L +5G /dev/vg_test/lv_demo
Scenario 3: Mount is not visible after reboot. Why?

Because it was mounted manually but not added to /etc/fstab.

Fix:

Get UUID:

sudo blkid

Edit fstab:

sudo nano /etc/fstab

Add mount entry and test:

sudo mount -a
Scenario 4: lvextend completed but df -h still shows old size. Why?

Possible reason:

Filesystem was not resized.

Fix for ext4:

sudo resize2fs /dev/vg_test/lv_demo

Better command:

sudo lvextend -r -L +1G /dev/vg_test/lv_demo
Scenario 5: Server boot problem after fstab change. What will you do?

I will restore fstab backup.

sudo cp /etc/fstab.backup /etc/fstab

Then test:

sudo mount -a

Prevention:

Always run:

sudo mount -a

before reboot.

22. My Final LVM Revision Summary

In this practice, I successfully created 3 LVM Logical Volumes on Ubuntu EC2.

My disk was:

/dev/xvdf

It was a 10 GB AWS EBS volume.

I created a Physical Volume using:

sudo pvcreate /dev/xvdf

I created a Volume Group using:

sudo vgcreate vg_test /dev/xvdf

I created three Logical Volumes:

sudo lvcreate -L 2G -n lv_demo vg_test
sudo lvcreate -L 2G -n lv_demo-1 vg_test
sudo lvcreate -L 3G -n lv_demo-2 vg_test

I formatted them using:

sudo mkfs.ext4 /dev/vg_test/lv_demo
sudo mkfs.ext4 /dev/vg_test/lv_demo-1
sudo mkfs.ext4 /dev/vg_test/lv_demo-2

I mounted them using:

sudo mount /dev/vg_test/lv_demo /mnt/lvm
sudo mount /dev/vg_test/lv_demo-1 /mnt/lvm-1
sudo mount /dev/vg_test/lv_demo-2 /mnt/lvm-2

I verified using:

lsblk
df -h
sudo pvs
sudo vgs
sudo lvs

The most important LVM flow is:

Disk → PV → VG → LV → Filesystem → Mount Point

The most important safety rule is:

Always check lsblk before using pvcreate, mkfs, lvremove, vgremove, or pvremove.
