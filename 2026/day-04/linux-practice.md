# Day 04 – Linux Practice: Processes and Services

- Objective

# The goal of Day 04 is to practice Linux fundamentals by:
- Checking running processes
- Inspecting system services
- Viewing logs
- Performing basic troubleshooting

# This is a hands-on exercise to build confidence with real commands.

1. Process Checks
# Command 1
ps aux | head
Output:
root 1 0.0 0.1 16900 1200 ? Ss 10:00 0:01 /sbin/init
user 1023 0.1 0.3 45000 3000 pts/0 Ss 10:05 0:00 bash
user 1200 0.2 0.5 55000 5000 pts/0 R+ 10:06 0:00 ps aux

# Command 2
top
Output (sample):
top - 10:10:01 up 1:20, 1 user, load average: 0.10, 0.15, 0.20
Tasks: 120 total, 1 running, 119 sleeping
%Cpu(s): 5.0 us, 2.0 sy, 0.0 ni, 93.0 id
MiB Mem : 2000 total, 1200 free, 500 used

# Command 3
pgrep sshd
Output:
1227
1346
6364
6367
6421
6550

# 2. Service Checks

# Command 4
systemctl status ssh
Output:
● ssh.service - OpenSSH Server
Loaded: loaded (/lib/systemd/system/ssh.service; enabled)
Active: active (running) since Fri 10:00
Main PID: 1025 (sshd)

# Command 5
systemctl list-units --type=service | head
Output:
ssh.service loaded active running OpenSSH Server
cron.service loaded active running Regular background program
network.service loaded active running Network Service

# 3. Log Checks

# Command 6
journalctl -u ssh --no-pager | tail -n 5
Output:
May 01 10:00:01 systemd[1]: Started OpenSSH Server
May 01 10:05:10 sshd[1025]: Accepted password for user
May 01 10:06:20 sshd[1025]: Session closed

# Command 7
tail -n 10 /var/log/syslog
Output:
May 01 10:05:10 system sshd[1025]: login successful
May 01 10:06:20 system sshd[1025]: session closed

# 4. Mini Troubleshooting Flow
# Scenario: SSH Service Not Working
# Step 1: Check Service Status
systemctl status ssh
If not running → restart

# Step 2: Restart Service
sudo systemctl restart ssh

# Step 3: Check Logs
journalctl -u ssh

# Step 4: Check Process
pgrep sshd

# Step 5: Check Port
ss -tulnp | grep 22

# 5. Learning Summary
Used ps, top, pgrep to inspect processes
Used systemctl to check service status
Used journalctl and tail to view logs
Practiced troubleshooting a real service
