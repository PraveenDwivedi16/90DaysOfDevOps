# Linux A–Z Revision (Zero → Hero)

#  1. Linux Basics
What is Linux?
Open-source Operating System
Used in servers, cloud, DevOps

Examples:
Ubuntu
Amazon Linux
CentOS

# Important Concept
Everything in Linux is a file

# 2. Linux Architecture
User → Shell → Kernel → Hardware
Components
Component	Role
Kernel	Core OS
Shell	Command interface
User space	Apps
systemd	Service manager

# 3. Basic Commands (Must Know)
pwd        # current path
ls -l      # list files
cd         # change directory
mkdir      # create folder
rm -rf     # delete
cp         # copy
mv         # move

#  4. File Operations
Create
touch file.txt
echo "text" > file.txt
Read
cat file.txt
head -n 5 file.txt
tail -n 5 file.txt
Append
echo "new line" >> file.txt

# 5. Process Management
ps aux
top
htop
kill PID
pkill process
Process States
Running
Sleeping
Zombie
Stopped

# 6. Services (systemd)
systemctl status nginx
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
Logs
journalctl -u nginx

# 7. File System (Important Directories)
/        → root
/home    → users
/root    → admin
/etc     → config
/var/log → logs
/tmp     → temp
/bin     → commands
/usr/bin → installed apps
/opt     → third-party apps

# 8. Permissions
-rwxrwxrwx
Values
Symbol	Value
r	4
w	2
x	1
Commands
chmod 755 file
chmod +x script.sh
chmod -w file

# 9. Users & Groups
Commands
useradd username
passwd username
groupadd group
usermod -aG group user
groups user
Files
/etc/passwd
/etc/group

# 10. Ownership
chown user file
chgrp group file
chown user:group file
chown -R user:group folder

# 11. Networking Basics
Commands
ping google.com
curl -I google.com
dig google.com
ss -tulpn
Layers
Application → HTTP, DNS
Transport → TCP
Network → IP

# 12. DNS
Domain → DNS → IP
Records
Type	Meaning
A	IPv4
AAAA	IPv6
CNAME	alias
MX	mail

# 13. IP & Subnet
Private IP
10.x.x.x
172.16–31.x.x
192.168.x.x
CIDR
CIDR	Hosts
/24	254
/16	65534
/28	14

# 14. Ports
Port	Service
22	SSH
80	HTTP
443	HTTPS
53	DNS
3306	MySQL
6379	Redis
27017	MongoDB

# 15. Troubleshooting Flow
Golden Flow
1. Check service
2. Check logs
3. Check process
4. Check network
5. Check permissions
Commands
systemctl status
journalctl
ps aux
ss -tulpn
df -h

# 16. LVM (Storage)
PV → VG → LV → Mount
Commands
pvcreate
vgcreate
lvcreate
lvextend
resize2fs

# 17. Cloud Basics
EC2 Flow
Launch → SSH → Install → Run → Open port → Access

# 18. Real DevOps Debugging
Example
App down → systemctl
Error → logs
Slow → top
Network → ping
Permission → ls -l

# INTERVIEW QUICK REVISION
Top Questions
Q1: What is Linux?

OS used for servers and DevOps

Q2: What is chmod?

Change file permissions

Q3: What is chown?

Change file ownership

Q4: What is systemctl?

Manage services

Q5: What is LVM?

Flexible disk management

Q6: What is DNS?

Domain → IP

Q7: How to debug issue?
Check service → logs → process → network → permission
